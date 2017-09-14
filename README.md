# Documentation for the Monarco HAT

Monarco HAT is an add-on board which provides input-output interfaces designed according to industrial automation standards for the Raspberry Pi (B+ and newer) minicomputer (https://www.raspberrypi.org/) and others with compatible IO header like the UP Board (http://www.up-board.org/).

It is designed according to the HAT (Hardware Attached on Top) specification (https://github.com/raspberrypi/hats).

This repository contains various documentation for the Monarco HAT:

* **[Monarco HAT Hardware Reference Manual](Monarco_HAT_Hardware_Reference_Manual.pdf)**
* [Monarco HAT Integration with Raspberry Pi and Linux](Monarco_HAT_Integration_Raspberry-Pi_Linux.md)
* [Monarco HAT SPI Protocol Reference Manual](Monarco_HAT_SPI_Protocol.md)
* [Monarco HAT Firmware Development Roadmap](Monarco_HAT_Firmware_Roadmap.md)
* [Monarco HAT Board Schematics](Monarco_HAT_Schematics_1-4-RELEASE.pdf)

## News

### 2017/09/14

**For Raspbian users** - Since 2017-06 release, Raspbian Jessie upgraded kernel from 4.4 to 4.9 series, which partially breaks compatibility with device-tree overlay in Monarco HAT EEPROM. `/dev/ttyAMA0` UART does not work automatically anymore with Monarco HAT. This also breaks firmware upgrades. Please see [Monarco HAT Integration with Raspberry Pi and Linux](Monarco_HAT_Integration_Raspberry-Pi_Linux.md#note-for-raspbian-users) for more instructions to fix this issue.