# Serial port upgrade MCU firmware



## Version Information

The internal klipper_440x warehouse is integrated into the host computer file system, and a new fw folder is created in it to store the firmware of the small board MCU of the lower computer of each model, but only the latest version of the firmware is retained.

In order to facilitate identification, development and use, both hardware version and software version information need to be reflected in the firmware name. When configuring and compiling the lower computer firmware, the automatic renaming operation is performed according to certain rules.



### hardware version

The hardware version needs to contain three types of information: small board type (mcu, nozzle, bed, etc.), hardware version, and MCU model.

Format definition:

1. Between the above three types of information, use underscore as the separator, occupying 4, 3, and 3 characters respectively, including a total of 12 characters including the underscore, as shown in the following table;
2. If there are multiple small boards of the same type, a number is reserved after all types to distinguish them, as shown in the table below;

2. Because there are many subdivisions of MCU models, different models of MCUs in the same category may be used at the same time, so a mapping table needs to be made to reduce character occupancy. The first character represents the manufacturer's brand, such as ST represented by S, GD represented by G; the second character represents the large series, such as 4xx represented by 4, 0xx represented by 0; the third character represents the adapted model. , starting from 0 and increasing in sequence. As shown in the table below;

| Small board type | Hardware version | MCU model | Final string |
| :------: | :------: | :----------------: | :----------: |
| mcu0 | 1.0.0 | STM32F401 --> S40 | mcu0_100_S40 |
| mcu0 | 1.0.1 | GD32F303XE --> G32 | mcu0_101_G32 |
| noz0 | 1.0.1 | STM32G071 --> S06 | noz0_101_S06 |
| noz1 | 1.0.1 | STM32G071 --> S06 | noz1_101_S06 |
| bed0 | 0.0.1 | GD32E230X8 --> G21 | bed0_001_G21 |



### Software version

The software version needs to contain two types of information: board type and firmware version, and the rest is reserved.

Format definition:

1. Between the above three types of information, underscores are also used as separators, occupying 4, 3, and 3 characters respectively, including a total of 12 characters including underscores, as shown in the following table;
2. If there are multiple small boards of the same type, a number is reserved after all types to distinguish them, as shown in the table below;

| Small board type | Firmware version | Reserved | Final string |
| :------: | :------: | :--: | :----------: |
| mcu0 | 0.1.0 | 000 | mcu0_010_000 |
| noz0 | 0.1.1 | 000 | noz0_011_000 |
| noz1 | 0.1.2 | 000 | noz1_012_000 |
| bed0 | 0.2.0 | 000 | bed0_020_000 |



## Pattern management

Modify klipper's Kconfig and Makefile, see klipper_440x internal warehouse submission for details. The main features are as follows:

1. Support the configuration of new board type information (such as small board type, hardware version, software version and MCU model, etc.) and save it manually to src/configs/***_defconfig. If necessary, configure the file Name them based on the model + hardware version (for example, the nozzle board hardware schematic designs of K1 and H1 are different), or only use the hardware version (for example, the host computer motherboard has integrated main MCU). The directory tree is roughly as follows:

   ```
   src/configs/
   ├── K1_bed0_100_G21_defconfig
   ├── K1_noz0_110_S06_defconfig
   ├── Kconfig
   ├── mcu0_110_G32_defconfig
   └── mcu0_110_S40_defconfig
   ```

2. The software version that supports the board type is stored in a specific area of ​​ROM, tentatively determined to be the interrupt vector table base address + 512 bytes offset, with a size of 32 bytes; it is read by the bootloader and returned to the host computer upgrade management program.

3. Supports automatic acquisition of hardware version and software version, and copies and renames the firmware after compiling it. See the firmware management chapter below.



## Firmware Management

Create a new fw folder in the root directory of the internal klipper_440x warehouse, and create some model name folders for debugging support inside it, such as K1 (currently shared by K1 Max), which are used to store the firmware of each MCU slave computer. The directory tree looks roughly like this:

```
fw/
└── K1
    ├── bed0_100_G21-bed0_000_000.bin
    ├── mcu0_110_G32-mcu0_000_000.bin
    └── noz0_110_S06-noz0_000_000.bin
```



## Upgrade management program

In order to maintain flexibility and convenience, it can be used together with scripts. The main features are as follows:

1. Supports a single handshake operation to avoid bootloader timeout (tentatively 15 seconds) and jumping to start the APP due to long upgrade time and long waiting time in queue.
2. On the premise that the handshake has been completed, it is supported to obtain the hardware version and software version return separately, search for matches in the shell script, and determine whether an upgrade is needed. If not, jump directly to start the APP.
3. Supports direct forced upgrade of firmware without comparing software versions when matching hardware versions, that is, forced upgrade mode (to be implemented)
4. Support direct forced firmware upgrade without matching hardware versions or comparing software versions, that is, forced debugging mode (to be implemented)

Code repository: http://172.23.88.26:3333/wuhui/mcu_update



## Support list (continuously updated)

Record the board types that have been developed and supported. In addition to the regular software and hardware version information, some major configuration item details are also added.

| Machine model | Small board type | Hardware version | MCU model | bootloader offset | Communication interface | Serial port baud rate | GPIO initialization | Board configuration file |
| :--: | :------: | :----------: | :----------------: | :-- ----------: | :-------------: | :--------: | :--------- -----: | :--------------------------: |
| K1 | mcu0 | S11-->110 | GD32F303XE --> G32 | 12KB | usart1(PA2/PA3) | 230400 | PC7,PB0 | mcu0_110_G32_defconfig |
| K1 | noz0 | V1.1-->110 | STM32G071 --> S06 | 12KB | usart2(PA2/PA3) | 230400 | !PB14,!PB15,!PA8 | K1_noz0_110_S06_defconfig |
| K1 | bed0 | V1.1.0-->110 | GD32E230X8 --> G21 | 12KB | usart0(PA9/PA10) | 230400 | None | K1_bed0_110_G21_defconfig |
| K1 | noz0 | v12 -->120 | GD32F303xB --> G30 | 12KB | usart1(PA2/PA3) | 230400 | !PB5,!PB6,!PB7 | K1_noz0_120_G30_defconfig |
| | | | | | | | | |
