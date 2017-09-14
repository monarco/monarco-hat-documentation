# Monarco HAT Integration with Raspberry Pi and Linux

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
  * MCU bootloader enable (high active)
* **GPIO21** [40]:
  * MCU RESETn (low active)
* **GPIO26** [37]:
  * ID EEPROM write enable (low active)
* **GPIO05** [29], **GPIO06** [31], **GPIO23** [16], **GPIO24** [18]:
  * Optional use with MCU's GPIO. These signals can be freely used, MCU keeps them floating by default.

For additional details, please see [Monarco HAT Hardware Reference Manual](Monarco_HAT_Hardware_Reference_Manual.pdf).


## HAT ID EEPROM and Device Tree Overlay

On the **I2C-0** (`/dev/i2c-0`) Raspberry Pi bus, Monarco HAT contains EEPROM with device tree overlay which is automatically loaded by Raspberry Pi bootloader. This overlay enables SPI interface and I2C-1 interface with real time clock (MCP79410) and 1-Wire controller (DS2482-100) device nodes. 

For additional details about Device Tree Overlay and notes for Raspberry Pi 3 and for `owserver` users, please see [Monarco HAT Hardware Reference Manual](Monarco_HAT_Hardware_Reference_Manual.pdf), Chapter *3.1.2 ID EEPROM*.

## NOTE FOR RASPBIAN USERS

Since 2017-06 release, Raspbian Jessie upgraded kernel from 4.4 to 4.9 searies, which partially breaks compatibility with device-tree overlay in Monarco HAT EEPROM.

Switching `/dev/ttyAMA0` UART to GPIO header does not work anymore on Raspberry Pi 3 nor Raspberry Pi 2 with current Raspbian because of change in device-tree nodes names.

**Please add [monarco-fix-4-9.dtbo](device-tree/monarco-fix-4-9.dtbo) to your `/boot/overlays` directory, and add `dtoverlay=monarco-fix-4-9` line to the `/boot/config.txt`.**

## Onboard I2C Devices

On the **I2C-1** (`/dev/i2c-1`) Raspberry Pi bus, Monarco HAT contains following devices: 

* Real Time Clock MCP79410, address `0x6f`, Linux driver module `rtc_ds1307`, device `mcp7941x`
* 1-Wire Interface Controller, address `0x18`, Linux driver module `ds2482`, device `ds2482`

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


## GPIO Signals

Several Raspberry Pi GPIOs are connected to Monarco HAT MCU:

* **GPIO21** [40]: 
  * Low level brings MCU to RESET state, useful for initializing MCU to default state during communication development, and for bootloader activation.  
* **GPIO20** [38]:
  * High level enables MCU bootloader for firmware update after reset.
* **GPIO26** [37]:
  * Low level enabled write to ID EEPROM on I2C-0. Note: On Raspberry Pi 3, I2C-0 communication does not work well during Linux runtime, but there is almost no reason for directly reading or writing ID EEPROM.
* **GPIO05** [29], **GPIO06** [31], **GPIO23** [16], **GPIO24** [18]:
  * Freely usable signals connected to MCU. Standard firmware keep them floating. Intended for future use or custom firware functions.
