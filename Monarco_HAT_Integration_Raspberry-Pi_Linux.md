# Monarco HAT Integration with Raspberry Pi and Linux

## Other Resources

* [Monarco Homepage - https://www.monarco.io/](https://www.monarco.io/)
* [Repository - Documentation for the Monarco HAT](https://github.com/monarco/monarco-hat-documentation)


## Hardware Interconnect Overview

Raspberry Pi IOs in use by Monarco HAT [physical pin numbers on 40pin header]:

* **SPI-0** with CE0 [19, 21, 23, 24]:
  * Primary data communication channel with Monarco MCU
* **UART-0** [8, 10]:
  * RS-485 forwarded through MCU (can be disabled) / MCU firmware update
* **I2C-1** (SCL, SDA) [3, 5]:
  * Real Time Clock, 1-Wire controller
* **I2C-0** (ID_SC, ID_SD) [27, 28]:
  * ID EEPROM according to the HAT standard
* **GPIO20** [38]:
  * MCU bootloader enable after reset (high active)
* **GPIO21** [40]:
  * MCU RESETn (low active), useful for initializing MCU to default state during communication development, and for bootloader activation
* **GPIO26** [37]:
  * ID EEPROM write enable (low active)
* **GPIO05** [29], **GPIO06** [31], **GPIO23** [16], **GPIO24** [18]:
  * Optional use with MCU's GPIO. These signals can be freely used, MCU keeps them floating by default. Intended for future use or custom firware functions.

**Do not use GPIOs 0 (ID_SD), 1 (ID_SC), 2 (SDA), 3 (SCL), 8 (CE0), 9 (MISO), 10 (MOSI), 11 (SCLK), 14 (TXD), 15 (RXD), 20, 21, 26 for any custom functions! You could break correct operation of the Monarco HAT.** Using colliding GPIOs can break the operation until complete power cycling is performed.    

For additional details, please see [Monarco HAT Hardware Reference Manual](Monarco_HAT_Hardware_Reference_Manual.pdf).


## HAT ID EEPROM and Device Tree Overlay

On the **I2C-0** (`/dev/i2c-0`) Raspberry Pi bus, Monarco HAT contains EEPROM with device tree overlay which is automatically loaded by Raspberry Pi bootloader. This overlay enables SPI interface and I2C-1 interface with real time clock (MCP79410) and 1-Wire controller (DS2482-100) device nodes. 

For additional details about Device Tree Overlay and notes for Raspberry Pi 3 and for `owserver` users, please see [Monarco HAT Hardware Reference Manual](Monarco_HAT_Hardware_Reference_Manual.pdf), Chapter *3.1.2 ID EEPROM*.

## NOTE FOR RASPBIAN USERS

Since 2017-06 release, Raspbian Jessie upgraded kernel from 4.4 to 4.9 searies, which partially breaks compatibility with the old device-tree overlay format in Monarco HAT EEPROM.

On older Monarco HAT boards shipped before shipped before 10/2017, switching `/dev/ttyAMA0` UART to GPIO header does not work on Raspberry Pi 3 nor Raspberry Pi 2 without EEPROM update.

**Please upgrade the HAT EEPROM on your older Monarco HATs if you are using Raspbian with kernel 4.9 or newer.** Please see: [Instructions and tools for EEPROM check and upgrade](https://github.com/monarco/monarco-hat-firmware-bin#id-eeprom-update)

Note: Former workaround with extra file `monarco-fix-4-9.dtbo` is not needed anymore with upgraded EEPROM.

## Onboard I2C Devices

On the **I2C-1** (`/dev/i2c-1`) Raspberry Pi bus, Monarco HAT contains following devices: 

* Real Time Clock MCP79410, Linux driver module `rtc_ds1307`, device `mcp7941x`, address `0x6f` (RTC/SRAM) + `0x57` (EEPROM)
* 1-Wire Interface Controller, Linux driver module `ds2482`, device `ds2482`, address `0x18`

Thanks to Device Tree Overlay in HAT ID EEPROM, no additional configuration is needed in Raspbian Linux to make these devices working.


## SPI Communication

On the **SPI-0/CE0** (`/dev/spidev0.0`) Raspberry Pi bus, Monarco HAT MCU (microcontroller) is connected as SPI Slave. This interface is dedicated for all input/output signals and Monarco HAT configuration communication.  

For communication protocol details, please see [Monarco HAT SPI Protocol Reference Manual](Monarco_HAT_SPI_Protocol.md).


## UART / RS-485 Communication

On the **UART-0** (`/dev/ttyAMA0`, `/dev/serial0`) Raspberry Pi bus, Monarco HAT MCU (microcontroller) is connected as UART device. Data on this interface is normally forwarded between Monarco HAT RS-485 and Raspberry Pi UART-0 in both directions with correct buffering for half-duplex operation. Buffer size for each direction is 256 bytes (as of firmware version v2.004).

Default communication parameters on Raspberry Pi (Host side) side are: 115200 Baud, 8 data bits, no parity, 1 stop bit. Host side baudrate is configurable by service register `0x012`, but it is usually not needed, as RS-485 side communication parameters are independed on Host side.

RS-485 side communication parameters can be fully configured by service registers `0x010` and `0x011`. Default parameters are: 9600 Baud, no parity, 1 stop bit. Please see [Monarco HAT SPI Protocol Reference Manual](Monarco_HAT_SPI_Protocol.md) for details.

UART-0 is also used for Monarco HAT MCU firmware update using bootloader embedded in MCU.

**Note for Raspbian Linux configuration:** UART-0 (`/dev/ttyAMA0`, `/dev/serial0`) is by default used by Linux system serial console. If you want to use Monarco HAT RS-485 for different purpose such as MODBUS RTU communication, you have to disable serial console by removing `console=serial0,115200` part from `/boot/cmdline.txt` file, reboot and check result in `/proc/cmdline`.
