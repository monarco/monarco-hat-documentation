# Documentation for the Monarco HAT


## Introduction

Monarco HAT is an add-on board which provides input-output interfaces designed according to industrial automation standards for the Raspberry Pi (B+ and newer) minicomputer (https://www.raspberrypi.org/) and others with compatible IO header like the UP Board (http://www.up-board.org/).

Main features of the Monarco HAT:

* 4 x digital IN, configurable as two up/down counters or encoder inputs, up to 500 kHz
* 4 x digital OUT, configurable as PWM/frequency generators, 1 Hz to 100 kHz range
* 2 x analog IN (0 to 10 V / 0 to 20 mA), 2 x analog OUT (0 to 10 V)
* 1 x RS-485
* 1 x 1-Wire bus
* 9 x LED indicator
* Power supply: 10 to 30 V DC
* Battery-backed RTC chip
* Hardware watchdog for Raspberry Pi
* High quality push-in terminals, detachable connector

It is designed according to the HAT (Hardware Attached on Top) specification (https://github.com/raspberrypi/hats).


## Documents

This repository contains various documentation for the Monarco HAT:

* **[Monarco HAT Hardware Reference Manual](Monarco_HAT_Hardware_Reference_Manual.pdf)**
* [Monarco HAT Integration with Raspberry Pi and Linux](Monarco_HAT_Integration_Raspberry-Pi_Linux.md)
* [Monarco HAT SPI Protocol Reference Manual](Monarco_HAT_SPI_Protocol.md)
* [Monarco HAT Firmware Development Roadmap](Monarco_HAT_Firmware_Roadmap.md)
* [Monarco HAT Board Schematics](Monarco_HAT_Schematics_1-6-RELEASE.pdf)


## Other Resources

* [Monarco Homepage - https://www.monarco.io/](https://www.monarco.io/)
* [Repository - Monarco HAT Firmware Images and Tools](https://github.com/monarco/monarco-hat-firmware-bin)
* [Repository - C language driver library for the Monarco HAT](https://github.com/monarco/monarco-hat-driver-c)
* [Repository - Node.js driver library for Monarco HAT](https://github.com/monarco/monarco-hat-driver-nodejs)
* [Repository - Node-RED driver library for Monarco HAT](https://github.com/monarco/node-red-contrib-monarco-hat)


## News

### 2019/07/28

New firmware v2.008 with Host Watchdog feature implemented. Updated SPI protocol documentation.

### 2018/06/08

New firmware v2.007 and updated SPI protocol documentation, see [Monarco HAT Firmware History](Monarco_HAT_Firmware_Roadmap.md).

### 2018/04/03

Updated version of Hardware Reference Manual is published.

**For Raspbian users** - Monarco HAT boards shipped since 01/2018 (Series F) have updated ID EEPROM for full compatibility with Raspbian Linux kernel 4.9+ (tested also on 4.14), so the manual configuration of overlay file is not needed anymore for the new boards.

ID EEPROM on the old boards can be upgraded to be fully compatible with the latest Raspbian releases using [Monarco HAT Firmware Downloader](https://github.com/monarco/monarco-hat-firmware-bin).

### 2017/09/14

**For Raspbian users** - Since 2017-06 release, Raspbian Jessie upgraded kernel from 4.4 to 4.9 series, which partially breaks compatibility with device-tree overlay in Monarco HAT EEPROM. `/dev/ttyAMA0` UART does not work automatically anymore with Monarco HAT. This also breaks firmware upgrades. Please see [Monarco HAT Integration with Raspberry Pi and Linux](Monarco_HAT_Integration_Raspberry-Pi_Linux.md#note-for-raspbian-users) for more instructions to fix this issue.
