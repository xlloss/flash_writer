# 1. Overview
-------------
## 1.1. Overview
This document explains about R-Car H3/M3 eMMC writer sample software.
The eMMC writer is downloaded from the Host PC via SCIF or USB by boot ROM.
And the eMMC writer downloads the some of the raw images from Host PC via SCIF or USB, and writes the raw images to the eMMC.<BR>
The eMMC writer supports High Speed SDR(i.e. 50MHz) and x8 bus width mode, and supports only MMC0 channel.<BR>

[Chapter 2](#2-operating-environment) describes the operating environment.<BR>
[Chapter 3](#3-software) describes the software.<BR>
[Chapter 4](#4-how-to-build-the-emmc-writer) explains example of how to build the eMMC writer.<BR>
[Chapter 5](#5-how-to-run-emmc-writer) explains example of how to perform the eMMC writer.<BR>
[Chapter 6](#-6-usb-download-api) explains specifications of USB download API.<BR>
[Chapter 7](#7-error-case-to-handle)  explains how to handle error case.<BR>

*Note) This sample software does not support the file system. Therefore, can be write the raw image to the eMMC.*

## 1.2. References
The following table shows the document related to this function.

##### Related Document
| Number | Issue | Title                                                        | Edition      |
|--------|-------|--------------------------------------------------------------|--------------|
| 1      | JEDEC | Embedded Multi-Media Card (eMMC) Electrical Standard  (5.01) | JESD84-B50.1 |

## 1.3. Restrictions
There is no restriction in this revision.

# 2. Operating Environment
--------------------------
## 2.1. Hardware Environment
The following table lists the hardware needed to use this function.

##### Hardware environment (R-Car H3/M3)
| Name                          | Note                                      |
|-------------------------------|-------------------------------------------|
| R-Car H3-SiP System Evaluation Board Salvator-X<BR>R-Car M3-SiP System Evaluation Board Salvator-X<BR>R-Car H3-SiP System Evaluation Board Salvator-XS<BR>R-Car M3-SiP System Evaluation Board Salvator-XS|RTP0RC7795SIPB0010S / RTP0RC7795SIPB0011S<BR>RTP0RC7796SIPB0010S / RTP0RC7796SIPB0011S<BR>RTP0RC7795SIPB0011S<BR>RTP0RC7795SIPB0012S |
| Host PC                       | Ubuntu Desktop 14.04(64bit) or later          |
| USB cable (type A to micro B) | Connect to CN25 when using UART connection. (SCIF2)<BR>Connect to CN9 when using USB connection. (HS-USB) |

*Note) RTP0RC7795SIPB0010S needs to update to the latest PMIC-EEPROM settings.*<BR>
*Note) After starting the eMMC writer, CN9 and CN25 can not be used in parallel. Please use only either one.*

The following table shows eMMC support for each SoC.

##### eMMC support status of each SoC
| SoC              | Read/Write the eMMC | Boot from the eMMC |
|------------------|---------------------|--------------------|
| R-Car M3 Ver.1.1 | Support             | Support            |
| R-Car M3 Ver.1.0 | Support             | Support            |
| R-Car H3 Ver.2.0 | Support             | Support            |
| R-Car H3 Ver.1.1 | Support             | Not support        |
| R-Car H3 Ver.1.0 | Support             | Not support        |

The following table shows USB support for each SoC.

##### USB download support status of each SoC
| SoC              | Image download by USB | Boot from the USB download mode |
|------------------|-----------------------|---------------------------------|
| R-Car M3 Ver.1.1 | Support               | Not Support                     |
| R-Car M3 Ver.1.0 | Support               | Not Support                     |
| R-Car H3 Ver.2.0 | Support               | Support                         |
| R-Car H3 Ver.1.1 | Support               | Not support                     |
| R-Car H3 Ver.1.0 | Support               | Not support                     |

USB 2.0 High-speed is supported. USB device class is CDC ACM  compliant.<BR>
Host PC's USB driver uses OS standard in-box driver.<BR>

The following table shows USB Vendor ID and Product ID.

##### List of USB Vendor ID and Product ID
| SoC      | Vendor ID (Renesas) | Product ID |
|----------|---------------------|------------|
| R-Car M3 | 0x45B               | 0x23D      |
| R-Car H3 | 0x45B               | 0x23C      |

#### Recommended Environment
![Recommended Environment](images/recommended_environment.png)

# 2.2. Software Environment
The following table lists the software needed to use this function.

##### Software environment
| Name             | Note                                                                      |
|------------------|---------------------------------------------------------------------------|
| Linaro Toolchain | Linaro Stable Binary Toolchain Release GCC 5.2-2015.11-2 for aarch64-elf. |
|                  | Linaro Stable Binary Toolchain Release GCC 5.2-2015.11-2 for arm-eabi.    |


# 3. Software
-------------
## 3.1. Function
This package has the following functions.
- Display the CID/CSD/EXT_CSD registers of eMMC.
- Modify the EXT_CSD registers of eMMC.
- Write to the images to the boot partition of eMMC.
- Write to the images to the user data area of eMMC.
- Erase the boot partition of eMMC.
- Erase the user data area of eMMC.
- Change to the SCIF baud rate setting.
- Display the command help.

## 3.2. Module structure
This module structure is shown below.
#### Module structure
```text
emmc_writer                     : root directory of eMMC writer
|-- AArch32_boot                : boot code for AArch32
|-- AArch64_boot                : boot code for AArch64
|-- AArch32_obj                 : object file output directory for AArch32
|-- AArch64_obj                 : object file output directory for AArch64
|-- AArch32_output              : output directory for AArch32
|-- AArch64_output              : output directory for AArch64
|-- AArch32_lib                 : USB library directory for AArch32
|-- AArch64_lib                 : USB library directory for AArch64
|-- ddr                         : DRAM initialize code directory
|-- include                     : header files directory
|-- b_boarddrv.c                : identify the board type
|-- boot_init_gpio.c            : GPIO initialize code
|-- boot_init_lbsc.c            : Local bus state controller initialize code
|-- boot_init_port_M3.c         : Pin function initialize code
|-- cert_param.c                : image header for SCIF/USB download image
|-- common.c                    : miscellaneous code
|-- cpudrv.c                    : miscellaneous code
|-- devdrv.c                    : log output code
|-- dg_emmc_access.c            : eMMC writer code
|-- dg_emmc_config.c            : eMMC device configuration code
|-- dginit.c                    : initialize code
|-- dgmodul1.c                  : miscellaneous code
|-- dgtable.c                   : command table
|-- emmc_cmd.c                  : eMMC driver files
|-- emmc_erase.c                : eMMC driver files
|-- emmc_init.c                 : eMMC driver files
|-- emmc_interrupt.c            : eMMC driver files
|-- emmc_mount.c                : eMMC driver files
|-- emmc_utility.c              : eMMC driver files
|-- emmc_write.c                : eMMC driver files
|-- main.c                      : main program
|-- Message.c                   : Help message
|-- micro_wait.c                : miscellaneous code
|-- ramckmdl.c                  : memory clear code(i.e. memset)
|-- scifdrv.c                   : SCIF driver
|-- timer_api.c                 : timer API for eMMC driver
|-- memory_area0.def            : Linker script
|-- memory_writer.def           : Linker script
|-- memory_writer_with_cert.def : Linker script
`-- makefile                    : makefile
```

## 3.3. Option setting
The eMMC writer support the following build options.

### 3.3.1. AArch
Select from the following table according to the target CPU architecture.<BR>
If this option is not selected, the default value is 32.<BR>

##### Association table for the AArch value and valid CPU architecture

| AArch | CPU architecture setting                                                                                                                            |
|-------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| 32    | Generate binary that works on AArch32. (default)<BR>It works on both the AP-system Core(i.e. Cortex-A57) and the ARM Realtime Core(i.e. Cortex-R7). |
| 64    | Generate binary that works on AArch64.<BR>It works on only the AP-System Core(i.e. Cortex-A57).                                                     |



### 3.3.2. BOOT<BR>
Select from the following table according to the image header settings.<BR>
If this option is not selected, the default value is WRITER_WITH_CERT.<BR>

##### Association table for the BOOT value and valid image header settings

| BOOT             | Image header setting                                                                                                                                             |
|------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| WRITER           | Generate binary for download mode without the load information header.                                                                                           |
| WRITER_WITH_CERT |Generate binary for download mode with the load information header. (default)<BR>This image can be used in both the SCIF download mode and the USB download mode. |

Describe the details of the SCIF/USB download image as follows:

##### Detail of binary image
![Detail of binary image](images/detail_of_binary_image.png)

### 3.3.3. SCIF_CLK<BR>
Select from the following table according to the clock to be supplied to SCIF.<BR>
If this option is not selected, the default value is EXTERNAL.<BR>

##### Association table for the SCIF_CLK value and valid SCIF clock source settings
| SCIF_CLK | SCIF clock source setting                                                                                                                                 |
|----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------|
| INTERNAL | The SCIF clock source is S3D4&phi; (i.e. 66.6666MHz) in the SoC.<BR>H3 Ver.1.0: Maximum baud rate is 115200bps.<BR>Other: Maximum baud rate is 230400bps. |
| EXTERNAL | The SCIF clock source is SCK2 pin. (default)<BR>Maximum baud rate is 921600bps.<BR>Please supply the 14.7456MHz to the SCK2 pin.                          |

*Note) On the Salvator-X/XS board, 14.7456MHz is supplied to the SCK2 pin.*

## 3.4. Command specification
The following table shows the command list.

##### Command list
| Command  | Description                                                                                        |
|----------|----------------------------------------------------------------------------------------------------|
| EM_DCID  | Display the CID registers of eMMC.                                                                 |
| EM_DCSD  | Display the CSD registers of eMMC.                                                                 |
| EM_DECSD | Display the EXT_CSD registers of eMMC.                                                             |
| EM_SECSD | Modify the EXT_CSD registers of eMMC.                                                              |
| EM_W     | Write to the S-record format images to the user data area of eMMC, and the boot partition of eMMC. |
| EM_WB    | Write to the raw binary images to the user data area of eMMC, and the boot partition of eMMC.      |
| EM_E     | Erase the user data area of eMMC, and the boot partition of eMMC.                                  |
| SUP      | Change the SCIF baud rate setting.                                                                 |
| H        | Display the command help.                                                                          |

### 3.4.1. Display the CID registers command
This command displays the contents of the CID registers of the eMMC.<BR>
The following shows the procedure of this command.
```text
>EM_DCID

[CID Field Data]
[127:120]  MID  0x89
[113:112]  CBX  0x01
[111:104]  OID  0x0A
[103: 56]  PNM  0x654D4D432020
[ 55: 48]  PRV  0x01
[ 47: 16]  PSN  0x261400E9
[ 15:  8]  MDT  0xB2
[  7:  1]  CRC  0x00
```
### 3.4.2. Display the CSD registers command
This command displays the contents of the CSD registers of eMMC.<BR>
The following shows the procedure of this command.
```text
>EM_DCSD

[CSD Field Data]
[127:126]  CSD_STRUCTURE       0x03
[125:122]  SPEC_VERS           0x04
[119:112]  TAAC                0x0F
...
[ 11: 10]  FILE_FORMAT         0x00
[  9:  8]  ECC                 0x00
[  7:  1]  CRC                 0x00
```
### 3.4.3. Display the EXT_CSD registers command
This command displays the contents of the EXT_CSD registers of the eMMC.<BR>
The following shows the procedure of this command.
```text
>EM_DECSD

[EXT_CSD Field Data]
[505:505]  EXT_SECURITY_ERR                           0x00
[504:504]  S_CMD_SET                                  0x01
[503:503]  HPI_FEATURES                               0x01
…
[142:140]  ENH_SIZE_MULT                              0x000000
[139:136]  ENH_START_ADDR                             0x00000000
[134:134]  SEC_BAD_BLK_MGMNT                          0x00
```
### 3.4.4. Modify the EXT_CSD registers of eMMC command
This command modifies the contents of the registers of EXT_CSD of the eMMC.<BR>
The following shows the procedure of this command.<BR>
```text
>EM_SECSD
  Please Input EXT_CSD Index(H'00 - H'1FF) :
```
Enter the address of the EXT_CSD register in hexadecimal.
```text
>EM_SECSD
  Please Input EXT_CSD Index(H'00 - H'1FF) :b1
  EXT_CSD[B1] = 0x00
  Please Input Value(H'00 - H'FF) :
```
Enter the settings of EXT_CSD register in hexadecimal.
```text
>EM_SECSD
  Please Input EXT_CSD Index(H'00 - H'1FF) :b1
  EXT_CSD[B1] = 0x00
  Please Input Value(H'00 - H'FF) :a
  EXT_CSD[B1] = 0x0A
```
The EXT_CSD register has been modified.

### 3.4.5. Write to the S-record format images to the eMMC
This command writes the S-record format image to any partition of the eMMC.<BR>

##### Example of writing data for the eMMC boot
| Filename                 | Program Top Address | eMMC Save Partition | eMMC Save Sectors | Description            |
|--------------------------|---------------------|---------------------|-------------------|------------------------|
| bootparam_sa0.srec       | H'E6320000          | boot partition1     | H'000000          | Loader(Boot parameter) |
| bl2-`<board_name>`.srec  | H'E6304000          | boot partition1     | H'00001E          | Loader                 |
| cert_header_sa6.srec     | H'E6320000          | boot partition1     | H'000180          | Loader(Certification)  |
| bl31-`<board_name>`.srec | H'44000000          | boot partition1     | H'000200          | ARM Trusted Firmware   |
| tee-`<board_name>`.srec  | H'44100000          | boot partition1     | H'001000          | OP-TEE                 |
| u-boot-elf.srec          | H'50000000          | boot partition2     | H'000000          | U-boot                 |

The following shows the procedure of this command.<BR>
```text
>EM_W
Start --------------
---------------------------------------------------------
Please select,eMMC Partition Area.
 0:User Partition Area   : 30212096 KBytes
  eMMC Sector Cnt : H'0 - H'0399FFFF
 1:Boot Partition 1      : 4096 KBytes
  eMMC Sector Cnt : H'0 - H'00001FFF
 2:Boot Partition 2      : 4096 KBytes
  eMMC Sector Cnt : H'0 - H'00001FFF
---------------------------------------------------------
  Select area(0-2)>
```
Please enter the partition number.
```text
>EM_W
Start --------------
---------------------------------------------------------
Please select,eMMC Partition Area.
 0:User Partition Area   : 30212096 KBytes
  eMMC Sector Cnt : H'0 - H'0399FFFF
 1:Boot Partition 1      : 4096 KBytes
  eMMC Sector Cnt : H'0 - H'00001FFF
 2:Boot Partition 2      : 4096 KBytes
  eMMC Sector Cnt : H'0 - H'00001FFF
---------------------------------------------------------
  Select area(0-2)>1
-- Boot Partition 1 Program -----------------------------
Please Input Start Address in sector :
```
Please enter the start sector number of the write image in hexadecimal. Sector size is 512 bytes.
```text
>EM_W
Start --------------
---------------------------------------------------------
Please select,eMMC Partition Area.
 0:User Partition Area   : 30212096 KBytes
  eMMC Sector Cnt : H'0 - H'0399FFFF
 1:Boot Partition 1      : 4096 KBytes
  eMMC Sector Cnt : H'0 - H'00001FFF
 2:Boot Partition 2      : 4096 KBytes
  eMMC Sector Cnt : H'0 - H'00001FFF
---------------------------------------------------------
  Select area(0-2)>1
-- Boot Partition 1 Program -----------------------------
Please Input Start Address in sector :1e
Please Input Program Start Address :
```
Please enter the program top address of the write image in hexadecimal.
```text
>EM_W
Start --------------
---------------------------------------------------------
Please select,eMMC Partition Area.
 0:User Partition Area   : 30212096 KBytes
  eMMC Sector Cnt : H'0 - H'0399FFFF
 1:Boot Partition 1      : 4096 KBytes
  eMMC Sector Cnt : H'0 - H'00001FFF
 2:Boot Partition 2      : 4096 KBytes
  eMMC Sector Cnt : H'0 - H'00001FFF
---------------------------------------------------------
  Select area(0-2)>1
-- Boot Partition 1 Program -----------------------------
Please Input Start Address in sector :1e
Please Input Program Start Address : e6304000
Work RAM(H'50000000-H'50FFFFFF) Clear....
please send ! ('.' & CR stop load)
```
Please download the write image in S-record format.
```text
>EM_W
Start --------------
---------------------------------------------------------
Please select,eMMC Partition Area.
 0:User Partition Area   : 30212096 KBytes
  eMMC Sector Cnt : H'0 - H'0399FFFF
 1:Boot Partition 1      : 4096 KBytes
  eMMC Sector Cnt : H'0 - H'00001FFF
 2:Boot Partition 2      : 4096 KBytes
  eMMC Sector Cnt : H'0 - H'00001FFF
---------------------------------------------------------
  Select area(0-2)>1
-- Boot Partition 1 Program -----------------------------
Please Input Start Address in sector :1e
Please Input Program Start Address : e6304000
Work RAM(H'50000000-H'50FFFFFF) Clear....
please send ! ('.' & CR stop load)
SAVE -FLASH.......
EM_W Complete!
```
Image writing has been completed.

### 3.4.6. Write to the raw binary images to the eMMC
This command writes the raw binary image to any partition of the eMMC.<BR>
The following shows the procedure of this command.
```text
>EM_WB
EM_WB Start --------------
---------------------------------------------------------
Please select,eMMC Partition Area.
 0:User Partition Area   : 30212096 KBytes
  eMMC Sector Cnt : H'0 - H'0399FFFF
 1:Boot Partition 1      : 4096 KBytes
  eMMC Sector Cnt : H'0 - H'00001FFF
 2:Boot Partition 2      : 4096 KBytes
  eMMC Sector Cnt : H'0 - H'00001FFF
---------------------------------------------------------
  Select area(0-2)>
```
Please enter the partition number.
```text
>EM_WB
EM_WB Start --------------
---------------------------------------------------------
Please select,eMMC Partition Area.
 0:User Partition Area   : 30212096 KBytes
  eMMC Sector Cnt : H'0 - H'0399FFFF
 1:Boot Partition 1      : 4096 KBytes
  eMMC Sector Cnt : H'0 - H'00001FFF
 2:Boot Partition 2      : 4096 KBytes
  eMMC Sector Cnt : H'0 - H'00001FFF
---------------------------------------------------------
  Select area(0-2)>1
-- Boot Partition 1 Program -----------------------------
Please Input Start Address in sector :
```
Please enter the start sector number of the write image in hexadecimal. Sector size is 512 bytes.
```text
>EM_WB
EM_WB Start --------------
---------------------------------------------------------
Please select,eMMC Partition Area.
 0:User Partition Area   : 30535680 KBytes
  eMMC Sector Cnt : H'0 - H'0399FFFF
 1:Boot Partition 1      : 4096 KBytes
  eMMC Sector Cnt : H'0 - H'00001FFF
 2:Boot Partition 2      : 4096 KBytes
  eMMC Sector Cnt : H'0 - H'00001FFF
---------------------------------------------------------
  Select area(0-2)>1
-- Boot Partition 1 Program -----------------------------
Please Input Start Address in sector :1e
Work RAM(H'50000000-H'50FFFFFF) Clear....
Please Input File size(byte) :
```
Please enter the write image size in hexadecimal.
```text
>EM_WB
EM_WB Start --------------
---------------------------------------------------------
Please select,eMMC Partition Area.
 0:User Partition Area   : 30212096 KBytes
  eMMC Sector Cnt : H'0 - H'0399FFFF
 1:Boot Partition 1      : 4096 KBytes
  eMMC Sector Cnt : H'0 - H'00001FFF
 2:Boot Partition 2      : 4096 KBytes
  eMMC Sector Cnt : H'0 - H'00001FFF
---------------------------------------------------------
  Select area(0-2)>1
-- Boot Partition 1 Program -----------------------------
Please Input Start Address in sector :1e
Work RAM(H'50000000-H'50FFFFFF) Clear....
Please Input File size(byte) : 1b03c
please send binary file!
```
Please download the raw binary write image.
```text
>EM_WB
EM_WB Start --------------
---------------------------------------------------------
Please select,eMMC Partition Area.
 0:User Partition Area   : 30212096 KBytes
  eMMC Sector Cnt : H'0 - H'0399FFFF
 1:Boot Partition 1      : 4096 KBytes
  eMMC Sector Cnt : H'0 - H'00001FFF
 2:Boot Partition 2      : 4096 KBytes
  eMMC Sector Cnt : H'0 - H'00001FFF
---------------------------------------------------------
  Select area(0-2)>1
-- Boot Partition 1 Program -----------------------------
Please Input Start Address in sector :1e
Work RAM(H'50000000-H'50FFFFFF) Clear....
Please Input File size(byte) : 1b03c
please send binary file!
SAVE -FLASH.......
EM_WB Complete!
```
Image writing has been completed.

### 3.4.7. Erase the eMMC
This command erases any partition of the eMMC.<BR>
The following shows the procedure of this command.<BR>
```text
>EM_E
Start --------------
---------------------------------------------------------
Please select,eMMC Partition Area.
 0:User Partition Area   : 30212096 KBytes
  eMMC Sector Cnt : H'0 - H'0399FFFF
 1:Boot Partition 1      : 4096 KBytes
  eMMC Sector Cnt : H'0 - H'00001FFF
 2:Boot Partition 2      : 4096 KBytes
  eMMC Sector Cnt : H'0 - H'00001FFF
---------------------------------------------------------
```
Please enter the partition number.
```text
>EM_E
Start --------------
---------------------------------------------------------
Please select,eMMC Partition Area.
 0:User Partition Area   : 30212096 KBytes
  eMMC Sector Cnt : H'0 - H'0399FFFF
 1:Boot Partition 1      : 4096 KBytes
  eMMC Sector Cnt : H'0 - H'00001FFF
 2:Boot Partition 2      : 4096 KBytes
  eMMC Sector Cnt : H'0 - H'00001FFF
---------------------------------------------------------
  Select area(0-2)>0
-- User Partition Area Program --------------------------
EM_E Complete!
```
Selected partition has been erased.

### 3.4.8. Change the SCIF baud rate setting
This command will change the baud rate of the SCIF.<BR>
Baud rate is dependent on the SoC and the SCIF clock.<BR>

##### Baud rate settings after command execution
| SoC        | SCIF clock settings<BR>(Build option) | Baud rate at startup | Baud rate at After command execution |
|------------|---------------------------------------|---------------------:|-------------------------------------:|
| M3 Ver.1.1 | External clock                        | 115200bps            |                            921600bps |
|            | Internal clock                        | 115200bps            |                            230400bps |
| M3 Ver.1.0 | External clock                        | 115200bps            |                            921600bps |
|            | Internal clock                        | 115200bps            |                            230400bps |
| H3 Ver.2.0 | External clock                        | 115200bps            |                            921600bps |
|            | Internal clock                        | 115200bps            |                            230400bps |
| H3 Ver.1.1 | External clock                        | 115200bps            |                            921600bps |
|            | Internal clock                        | 115200bps            |                            230400bps |
| H3 Ver.1.0 | External clock                        | 57600bps             |                            460800bps |
|            | Internal clock                        | 57600bps             |                            115200bps |

*Note) The baud rate that has been changed in this command cannot be undone until the power is turned off.*<BR>
*Note) In the case of USB connection by CN9, this command has no effect.*

The following shows the procedure of this command.
```text
>SUP
Scif speed UP
Please change to 921.6Kbps baud rate setting of the terminal.
```

### 3.4.9. Display the command help
Displays a description of the commands.<BR>
The following shows the procedure of this command.<BR>
```text
>H
        eMMC write command
 EM_DCID        display register CID
 EM_DCSD        display register CSD
 EM_DECSD       display register EXT_CSD
 EM_SECSD       change register EXT_CSD byte
 EM_W           write program to eMMC
 EM_WB          write program to eMMC (Binary)
 EM_E           erase program to eMMC
 SUP            Scif speed UP (Change to speed up baud rate setting)
 H              help
```
# 4. How to build the eMMC writer
---------------------------------
This chapter is described how to build the eMMC writer of 32bit version and 64bit version.
Command is executed in the user's home directory (~ /).
## 4.1. Prepare the compiler
Gets cross compiler. To decompress it. Command is the following.<BR>
32 bit compiler:
```shell
$ cd ~/
$ wget https://releases.linaro.org/components/toolchain/binaries/5.2-2015.11-2/arm-eabi/gcc-linaro-5.2-2015.11-2-x86_64_arm-eabi.tar.xz
$ tar xvf gcc-linaro-5.2-2015.11-2-x86_64_arm-eabi.tar.xz
```

64 bit compiler:
```shell
$ cd ~/
$ wget https://releases.linaro.org/components/toolchain/binaries/5.2-2015.11-2/aarch64-elf/gcc-linaro-5.2-2015.11-2-x86_64_aarch64-elf.tar.xz
$ tar xvf gcc-linaro-5.2-2015.11-2-x86_64_aarch64-elf.tar.xz
```

## 4.2. Prepare the source code
Source code of eMMC writer is decompressed by the following command.<BR>
```shell
$ cd ~/
$ git clone https://github.com/renesas-rcar/emmc_writer.git
$ cd emmc_writer
$ git checkout rcar_gen3
```

## 4.3. Build the eMMC writer
S-record file is built by the following command.<BR>
32 bit compiler:
```shell
$ make AArch=32 clean
$ CROSS_COMPILE=~/gcc-linaro-5.2-2015.11-2-x86_64_arm-eabi/bin/arm-eabi- make AArch=32
```
Output the following image.<BR>
* ./AArch32_output/AArch32_eMMC_writer_SCIF_DUMMY_CERT_E6300400.mot

64 bit compiler:
```shell
$ make AArch=64 clean
$ CROSS_COMPILE=~/gcc-linaro-5.2-2015.11-2-x86_64_aarch64-elf/bin/aarch64-elf- make AArch=64
```
Output the following image.<BR>
* ./AArch64_output/AArch64_eMMC_writer_SCIF_DUMMY_CERT_E6300400.mot

The target file name changes depending on the build options.<BR>
The following table lists the relationship between build option and target files.

##### Description of build options and target files
| Build options           || Target directory | Target filename                                                                                      |
|-------|------------------|------------------|------------------------------------------------------------------------------------------------------|
| AArch | BOOT             |                  |                                                                                                      |
| 32    | WRITER           | AArch32_output   | AArch32_eMMC_writer_SCIF_E6304000.mot<BR>AArch32_eMMC_writer_SCIF_E6304000.bin                       |
|       | WRITER_WITH_CERT |                  | AArch32_eMMC_writer_SCIF_DUMMY_CERT_E6300400.mot<BR>AArch32_eMMC_writer_SCIF_DUMMY_CERT_E6300400.bin |
| 64    | WRITER           | AArch64_output   | AArch64_eMMC_writer_SCIF_E6304000.mot<BR>AArch64_eMMC_writer_SCIF_E6304000.bin                       |
|       | WRITER_WITH_CERT |                  | AArch64_eMMC_writer_SCIF_DUMMY_CERT_E6300400.mot<BR>AArch64_eMMC_writer_SCIF_DUMMY_CERT_E6300400.bin |

# 5. How to run eMMC writer
---------------------------------
## 5.1. Prepare for write to the eMMC
Start the target in the SCIF download mode and run eMMC writer sample code.<BR>
The following table shows the Dip-Switch Setting for SCIF download mode.<BR>

##### Dip switch configuration for write to the eMMC (SCIF download mode)
| SoC                                           | Boot CPU | Switch Number | Switch Name | Pin1 | Pin2 | Pin3 | Pin4 | Pin5 | Pin6 | Pin7 | Pin8 |
|--------------------------------------------------------|--------------------|------|----------|-----|-----|-----|-----|-----|-----|-----|-----|
| R-Car M3 Ver.1.1 / R-Car M3 Ver.1.0 / R-Car H3 Ver.2.0 | Cortex-A57 AArch64 | SW10 | MODESW-A | ON  | ON  | ON  | ON  | OFF | OFF | OFF | OFF |
|                                                        |                    | SW12 | MODESW-C | OFF | ON  | ON  | ON  | ON  | ON  | ON  | ON  |
|                                                        | Cortex-A57 AArch32 | SW10 | MODESW-A | ON  | ON  | ON  | ON  | OFF | OFF | OFF | OFF |
|                                                        |                    | SW12 | MODESW-C | ON  | ON  | ON  | ON  | ON  | ON  | ON  | ON  |
|                                                        | Cortex-R7          | SW10 | MODESW-A | OFF | OFF | ON  | ON  | OFF | OFF | OFF | OFF |
|                                                        |                    | SW12 | MODESW-C | -*1 | ON  | ON  | ON  | ON  | ON  | ON  | ON  |
| R-Car H3 Ver.1.1                                       | Cortex-A57 AArch64 | SW10 | MODESW-A | ON  | ON  | OFF | ON  | OFF | OFF | OFF | OFF |
|                                                        |                    | SW12 | MODESW-C | OFF | ON  | ON  | ON  | ON  | ON  | ON  | ON  |
|                                                        | Cortex-A57 AArch32 | SW10 | MODESW-A | ON  | ON  | OFF | ON  | OFF | OFF | OFF | OFF |
|                                                        |                    | SW12 | MODESW-C | ON  | ON  | ON  | ON  | ON  | ON  | ON  | ON  |
|                                                        | Cortex-R7          | SW10 | MODESW-A | OFF | OFF | OFF | ON  | OFF | OFF | OFF | OFF |
|                                                        |                    | SW12 | MODESW-C | -*1 | ON  | ON  | ON  | ON  | ON  | ON  | ON  |
| R-Car H3 Ver.1.0                                       | Cortex-A57 AArch64 | SW10 | MODESW-A | ON  | ON  | OFF | OFF | OFF | OFF | OFF | OFF |
|                                                        |                    | SW12 | MODESW-C | OFF | ON  | OFF | OFF | ON  | ON  | ON  | ON  |
|                                                        | Cortex-A57 AArch32 | SW10 | MODESW-A | ON  | ON  | OFF | OFF | OFF | OFF | OFF | OFF |
|                                                        |                    | SW12 | MODESW-C | ON  | ON  | OFF | OFF | ON  | ON  | ON  | ON  |
|                                                        | Cortex-R7          | SW10 | MODESW-A | OFF | OFF | OFF | OFF | OFF | OFF | OFF | OFF |
|                                                        |                    | SW12 | MODESW-C | -*1 | ON  | OFF | OFF | ON  | ON  | ON  | ON  |

\*1: Don't care this setting for Cortex-R7 boot mode.<BR>

The following table shows the Dip-Switch Setting for USB download mode.

##### Dip switch configuration for write to the eMMC (USB download mode)
| SoC                 | Boot CPU | Switch Number | Switch Name | Pin1 | Pin2 | Pin3 | Pin4 | Pin5 | Pin6 | Pin7 | Pin8 |
|----------------------------------------|--------------------|------|----------|-----|-----|-----|-----|-----|-----|-----|-----|
| R-Car M3 Ver.1.1 / R-Car M3 Ver.1.0 *2 | -                  | -    | -        | -   | -   | -   | -   | -   | -   | -   | -   |
| R-Car H3 Ver.2.0                       | Cortex-A57 AArch64 | SW10 | MODESW-A | ON  | ON  | ON  | ON  | OFF | OFF | OFF | ON  |
|                                        |                    | SW12 | MODESW-C | OFF | ON  | ON  | ON  | ON  | ON  | ON  | ON  |
|                                        | Cortex-A57 AArch32 | SW10 | MODESW-A | ON  | ON  | ON  | ON  | OFF | OFF | OFF | ON  |
|                                        |                    | SW12 | MODESW-C | ON  | ON  | ON  | ON  | ON  | ON  | ON  | ON  |
|                                        | Cortex-R7          | SW10 | MODESW-A | OFF | OFF | ON  | ON  | OFF | OFF | OFF | ON  |
|                                        |                    | SW12 | MODESW-C | -*1 | ON  | ON  | ON  | ON  | ON  | ON  | ON  |
| R-Car H3 Ver.1.1 / R-Car H3 Ver.1.0 *3 | -                  | -    | -        | -   | -   | -   | -   | -   | -   | -   | -   |

\*1: Don't care this setting for Cortex-R7 boot mode.<BR>
\*2: M3 Ver.1.0 and M3 Ver.1.1 cannot be boot from the USB download mode.<BR>
\*3: H3 Ver.1.0 and H3 Ver.1.1 cannot be boot from the USB download mode.<BR>

Connect the Host PC to CN25 or CN9 connector using the USB cable.<BR>
The following table shows the setting of terminal software.<BR>

##### Terminal software configuration
| SoC        | Baud rate  | Data bit length | Parity check | Stop bits | Flow control |
|------------|-----------:|-----------------|--------------|-----------|--------------|
| M3 Ver.1.1 | 115200bps  | 8bits           | none         | 1bit      | none         |
| M3 Ver.1.0 | 115200bps  | 8bits           | none         | 1bit      | none         |
| H3 Ver.2.0 | 115200bps  | 8bits           | none         | 1bit      | none         |
| H3 Ver.1.1 | 115200bps  | 8bits           | none         | 1bit      | none         |
| H3 Ver.1.0 |  57600bps  | 8bits           | none         | 1bit      | none         |

Terminal software outputs the following log at power ON the target.
```text
SCIF Download mode (w/o verification)
(C) Renesas Electronics Corp.

-- Load Program to SystemRAM ---------------
Work RAM(H'E6300000-H'E632E800) Clear....
please send !
```
Transfer S-record file after the log output.<BR>
S-record file for Cortex-A57 AArch64:<BR>
- AArch64_output/AArch64_eMMC_writer_SCIF_DUMMY_CERT_E6300400.mot

S-record file for Cortex-A57 AArch32 or Cortex-R7:
- AArch32_output/AArch32_eMMC_writer_SCIF_DUMMY_CERT_E6300400.mot

When the transfer is successful, the following log is output.
```text
eMMC Writer for R-Car H3/M3 Series V1.02 Apr.28,2017
 Work Memory SystemRAM (H'E6328000-H'E632FFFF)
>
```
For details on how to write to the eMMC, please refer to [Section 3.4](#34-command-specification).

## 5.2. Prepare for boot from the eMMC
To boot from the eMMC, need to change the Dip switch setting.<BR>
The following table shows the Dip-Switch Setting.<BR>

#### Dip switch configuration for boot from the eMMC (50MHz x8 bus width mode)
| SoC                                       | Boot CPU | Switch Number | Switch Name | Pin1 | Pin2 | Pin3 | Pin4 | Pin5 | Pin6 | Pin7 | Pin8 |
|--------------------------------------------------------|--------------------|------|----------|-----|-----|----|----|-----|-----|----|-----|
| R-Car M3 Ver.1.1 / R-Car M3 Ver.1.0 / R-Car H3 Ver.2.0 | Cortex-A57 AArch64 | SW10 | MODESW-A | ON  | ON  | ON | ON | OFF | OFF | ON | OFF |
|                                                        |                    | SW12 | MODESW-C | OFF | ON  | ON | ON | ON  | ON  | ON | ON  |
|                                                        | Cortex-A57 AArch32 | SW10 | MODESW-A | ON  | ON  | ON | ON | OFF | OFF | ON | OFF |
|                                                        |                    | SW12 | MODESW-C | ON  | ON  | ON | ON | ON  | ON  | ON | ON  |
|                                                        | Cortex-R7          | SW10 | MODESW-A | OFF | OFF | ON | ON | OFF | OFF | ON | OFF |
|                                                        |                    | SW12 | MODESW-C | -*1 | ON  | ON | ON | ON  | ON  | ON | ON  |
| R-Car H3 Ver.1.1 *2                                    | -                  | -    | -        | -   | -   | -  | -  | -   | -   | -  | -   |
| R-Car H3 Ver.1.0 *2                                    | -                  | -    | -        | -   | -   | -  | -  | -   | -   | -  | -   |

\*1: Don't care this setting for Coretex-R7 boot mode.<BR>
\*2: H3 Ver.1.0 and H3 Ver.1.1 cannot be boot from the eMMC.<BR>

# 6. USB download API

By using USB library, USB download function can be easily use.<BR>
To use the USB library, include or link the following files.

##### List of USB library files
| Path        | Filename  | Explanation              |
|-------------|-----------|--------------------------|
| include     | usb_lib.h | USB library header file.<BR>Include this file from the source file. |
| AArch32_lib | libusb.a  | USB library for AArch32.<BR>Please link when building with AArch32. |
| AArch64_lib | libusb.a  | USB library for AArch64.<BR>Please link when building with AArch64. |

The USB library provides the following seven APIs for image download function.

##### List of USB download APIs
| No. | Function Name          | Explanation                                      |
|----:|------------------------|--------------------------------------------------|
| 1   | USB_Init               | Initialize the USB Library.                      |
| 2   | USB_TerminalInputCheck | Check the input from terminal.                   |
| 3   | USB_IntCheck           | Update the internal state of the USB library.    |
| 4   | USB_ReadCount          | Number of bytes readable from the USB host.      |
| 5   | USB_ReadData           | Read data from the USB host.                     |
| 6   | USB_ReadDataWithDMA    | Read data from the USB host with DMA controller. |
| 7   | USB_WriteData          | Write data to the USB host.                      |

## 6.1 USB download API specifications

The USB download API specifications are shown below.

### 6.1.1 Initialize the USB Library
<table>
  <tr>
    <th style=text-align:left colspan=4>USB_Init</th>
  </tr>
    <td>Prototype</th>
    <td colspan=3>int32_t USB_Init(void)</td>
  <tr>
    <td>Argument</td>
    <td colspan=3>-</td>
  </tr>
  <tr>
    <td rowspan=2>Return value</td>
    <td rowspan=2>int32_t</td>
    <td>0</td>
    <td>Initialization success.</td>
  </tr>
  <tr>
    <td>other</td>
    <td>Initialization failure.</td>
  </tr>
  <tr>
    <td>Outline</td>
    <td colspan=3>Initialize the USB library.</td>
  </tr>
  <tr>
  <td>Note</td>
  <td colspan=3>Please call this function before using other USB download APIs.</td>
  </tr>
</table>

### 6.1.2 Check the input from terminal
<table>
  <tr>
    <th style=text-align:left colspan=4>USB_TerminalInputCheck</th>
  </tr>
    <td>Prototype</th>
    <td colspan=3>int32_t USB_TerminalInputCheck(uint8_t *command_area)</td>
  <tr>
    <td>Argument</td>
    <td>uint8_t *</td>
    <td>command_area</td>
    <td>Buffer for command input.</td>
  </tr>
  <tr>
    <td rowspan=2>Return value</td>
    <td rowspan=2>int32_t</td>
    <td>0</td>
    <td>No input character.</td>
  </tr>
  <tr>
    <td>1 or more</td>
    <td>Number of characters entered.</td>
  </tr>
  <tr>
    <td>Outline</td>
    <td colspan=3>This API checks the input of characters from the USB host.<BR>If no character is entered, 0 is returned.<BR>If a character is input, it returns the number of characters.</td>
  </tr>
  <tr>
  <td>Note</td>
  <td colspan=3></td>
  </tr>
</table>

### 6.1.3 Update the internal state of the USB library
<table>
  <tr>
    <th style=text-align:left colspan=4>USB_IntCheck</th>
  </tr>
    <td>Prototype</th>
    <td colspan=3>void USB_IntCheck(void)</td>
  <tr>
    <td>Argument</td>
    <td colspan=3>-</td>
  </tr>
  <tr>
    <td>Return value</td>
    <td colspan=3>-</td>
  </tr>
  <tr>
    <td>Outline</td>
    <td colspan=3>This API checks the interrupt source and updates internal status.</td>
  </tr>
  <tr>
  <td>Note</td>
  <td colspan=3>This API polls interrupt sources.<BR>Therefore, periodic calls are required.</td>
  </tr>
</table>

### 6.1.4 Number of bytes readable from the USB host
<table>
  <tr>
    <th style=text-align:left colspan=4>USB_ReadCount</th>
  </tr>
    <td>Prototype</th>
    <td colspan=3>int USB_ReadCount(void)</td>
  <tr>
    <td>Argument</td>
    <td colspan=3>-</td>
  </tr>
  <tr>
    <td rowspan=2>Return value</td>
    <td rowspan=2>int</td>
    <td>0</td>
    <td>There is no data that can be read from the USB host.</td>
  </tr>
  <tr>
    <td>other</td>
    <td>Number of bytes that can be read from the USB host.</td>
  </tr>
  <tr>
    <td>Outline</td>
    <td colspan=3>This API returns the number of bytes that can be read from the USB host.</td>
  </tr>
  <tr>
  <td>Note</td>
  <td colspan=3></td>
  </tr>
</table>

### 6.1.5 Read data from the USB host
<table>
  <tr>
    <th style=text-align:left colspan=4>USB_ReadData</th>
  </tr>
    <td>Prototype</th>
    <td colspan=3>int USB_ReadData(uint8_t *pBuff, int iDataSize)</td>
  <tr>
    <td rowspan=2>Argument</td>
    <td>uint8_t *</td>
    <td>pBuff</td>
    <td>Write destination pointer of data read from USB host.</td>
  </tr>
  <tr>
    <td>int</td>
    <td>iDataSize</td>
    <td>Size of pBuff.</td>
  </tr>
  <tr>
    <td>Return value</td>
    <td>int</td>
    <td>1 or more</td>
    <td>Number of bytes read from the USB host.</td>
  </tr>
  <tr>
    <td>Outline</td>
    <td colspan=3>This API reads the number of bytes specified by argument from USB host, read data is written to pBuff.<BR>
    If there is no data to be read, wait within the API until it can be read.</td>
  </tr>
  <tr>
  <td>Note</td>
  <td colspan=3></td>
  </tr>
</table>

### 6.1.6 Write data to the USB host
<table>
  <tr>
    <th style=text-align:left colspan=4>USB_WriteData</th>
  </tr>
    <td>Prototype</th>
    <td colspan=3>int USB_WriteData(uint8_t *pBuff, int iDataSize)</td>
  <tr>
    <td rowspan=2>Argument</td>
    <td>uint8_t *</td>
    <td>pBuff</td>
    <td>Read source pointer of data to be written to the USB host.</td>
  </tr>
  <tr>
    <td>int</td>
    <td>iDataSize</td>
    <td>Size of pBuff.</td>
  </tr>
  <tr>
    <td>Return value</td>
    <td>int</td>
    <td>1 or more</td>
    <td>Number of bytes written to the USB host.</td>
  </tr>
  <tr>
    <td>Outline</td>
    <td colspan=3>This API writes the number of bytes specified by the argument to the USB host.<BR>
    Wait in the API until the writing is completed.</td>
  </tr>
  <tr>
    <td>Note</td>
    <td colspan=3></td>
  </tr>
</table>

### 6.1.7 Read data from the USB host by DMA controller
<table>
  <tr>
    <th style=text-align:left colspan=4>USB_ReadDataWithDMA</th>
  </tr>
    <td>Prototype</th>
    <td colspan=3>void USB_ReadDataWithDMA(unsigned long bufferAddress, uint32_t totalDownloadSize)</td>
  <tr>
    <td rowspan=2>Argument</td>
    <td>long</td>
    <td>bufferAddress</td>
    <td>Write destination pointer of data read from USB host.</td>
  </tr>
  <tr>
    <td>uint32_t</td>
    <td>totalDownloadSize</td>
    <td>Size of buffer.</td>
  </tr>
  <tr>
    <td>Return value</td>
    <td colspan=3>-</td>
  </tr>
  <tr>
    <td>Outline</td>
    <td colspan=3>This API reads the number of bytes specified by argument from USB host using DMA, read data is written to bufferAddress.<BR>
    If there is no data to be read, wait within the API until it can be read.</td>
  </tr>
  <tr>
    <td>Note</td>
    <td colspan=3>In order to successfully complete this API, it must match the number of bytes sent from the USB host.</td>
  </tr>
</table>

# 7. Error case to handle
If error of eMMC command is occurred, please check the following description and restart.<BR>
 - Please Check the correct setting of EXT_CSD. If the wrong setting is present, to set the correct setting using EM_SECSD command.
 - Program start address error of S-record file.

## 7.1. EXT_CSD incorrect setting case
The following shows the setting of 50MHz x8 bus width mode, Boot partition 1 enable.

| Address      | Register Name       | Filed name             | Bit filed | Settings |
|--------------|---------------------|------------------------|-----------|----------|
| EXT_CSD[179] | PARTITION_CONFIG    | BOOT_PARTITION_ENABLE  | [5:3]     | 0x1      |
| EXT_CSD[177] | BOOT_BUS_CONDITIONS | BOOT_MODE              | [4:3]     | 0x1      |
|              |                     | BOOT_BUS_WIDTH         | [1:0]     | 0x2      |

For details of EXT_CSD, please refer to [Related Document](#related-document) No.1.

## 7.2. Program start address error of S-record format file
After the message "Please Input User Program Start Address" has been displayed, input a start address of the S-record format file to be loaded (smallest value) as the start address of the program. (This address is treated as the start address and branch address of the data transfer destination from the eMMC device in the program.)<BR>

Please check the program start address, and write again program using EM_W command.