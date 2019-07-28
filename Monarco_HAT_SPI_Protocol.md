# Monarco HAT SPI Protocol Reference Manual

**Valid for firmware version:** v2.008 (`0x2008` in register 0x001)

Note: functions labelled [FUTURE] are currently being under development, and will be part of future firmware releases.


## Other Resources

* [Monarco Homepage - https://www.monarco.io/](https://www.monarco.io/)
* [Repository - Documentation for the Monarco HAT](https://github.com/monarco/monarco-hat-documentation)


## Introduction

Primary data communication interface of Monarco HAT is an SPI (Serial Peripheral Interface). Monarco HAT act as an *SPI Slave*, host device (Raspbery Pi) act as an *SPI Master*.

SPI communication is organized into data transfers. Each data transfer is initiated by *SPI Master* which send N bytes to *SPI Slave*, and simultaneously *SPI Master* receive N bytes from *SPI Slave*.


## Data transfer basic principles

For such realtime input/output device like the Monarco HAT, it is common to define two classes of data communication:

* **Proces (cyclic) data channel - PDC:** contains input/output values - like current state of digital and analog inputs, input-based counters, and demanded state of digital and analog outpus. These values are required to be continuously refreshed by *SPI Master*, usually synchronously with each execution period of upper level control algorithm (e.g. task in realtime control system software).
* **Service (acyclic) data channel - SDC:** is used for configuration and diagnostics values - like RS-485 baudrate and mode, analog inputs type and range, input/output technology functions configuration, firmware version information, and various diagnostics counters. These values do not have to be read or written all the time or with each data transfer. So they can be communicated asynchronously using shared low bandwidth channel.

For ease of implementation of both *SPI Master* and *SPI Slave*, only two types of SPI transfer with predefined length are defined. The high level transfer structure is the same for both communication directions:

* **Standard transfer, length = 26 bytes:**
  * 4 bytes: Service data channel (SDC)
  * 20 bytes: Process data channel (PDC)
  * 2 bytes: Cyclic Redundancy Check (CRC)
* **[FUTURE] Service data only transfer, length = 6 bytes:**
  * 4 bytes: Service data channel (SDC)
  * 2 bytes: Cyclic Redundancy Check (CRC)

Length of process channel is 20 bytes by default. For future extensions, *SPI Master* can use *Service data only transfer* to reconfigure process channel contens. In such case, length of *Standard transfer* can be changed.

Service data are organized as set of 16 bit registers which can be read and written. See chapter *Service channel register description* below for details.

If valid process data is not received by Monarco HAT (*SPI Slave*) within specified interval (1500 ms by default, register 0x00F, see below for details), internal alarm is triggered and all outputs are switched to default states.


## SPI data transfer parameters

* Maximal SPI clock frequency: 4 MHz (limited by Monarco HAT's MCU SPI slave peripheral)
* SPI mode: `MODE0 -- CPOL = 0, CPHA = 0` - Clock idle low, data is valid on clock rising edge
* SPI data bits: 8


## Data transfer format

Byte order of 16/32 bit values is little-endian, e.g. least significant byte first.


## Standard transfer structure - Host (RPi) TX to Monarco HAT (SPI Master to Slave)

<pre>
offset  size  content
   0.0   4.0  Service channel request (spi_data_service_t)
   4.0   1.0  Control byte (spi_data_control_byte_t)
   5.0   1.0  User LEDs mask
   6.0   1.0  User LEDs value
   7.0   1.0  Digital outputs (bits 0..3 = DOUT1..DOUT4, bits 4..7 = RESERVED)
   8.0   2.0  PWM1 frequency
  10.0   2.0  PWM1 Channel A (DOUT1) duty cycle   
  12.0   2.0  PWM1 Channel B (DOUT2) duty cycle
  14.0   2.0  PWM1 Channel C (DOUT3) duty cycle
  16.0   2.0  PWM2 frequency
  18.0   2.0  PWM2 Channel A (DOUT4) duty cycle
  20.0   2.0  Analog output 1 (0..4095 = 0..10 V, > 4095 = RESERVED)
  22.0   2.0  Analog output 2 (0..4095 = 0..10 V, > 4095 = RESERVED)
  24.0   2.0  CRC-16
============
  26.0 ([bytes].[bits])
</pre>

### Control byte

<pre>
offset
   0.0  Status LED mask
   0.1  Status LED on
   0.2  1-Wire power shutdown
   0.3  RESERVED
   0.4  Counter 1 reset (edge sensitive)
   0.5  Counter 2 reset (edge sensitive)
   0.6  Sign of Life 0
   0.7  Sign of Life 1
</pre>

* *Status LED mask* - set `1` for control LED0 by *Control byte / Status LED on*; set `0` for default LED0 function (system status indication, see Monarco HAT Hardware Reference Manual).
* *Status LED on* - set `1` for LED0 lit on; set `0` for LED0 off; only applicable if *Control byte / Status LED mask* set `1`.
* *1-Wire power shutdown* - set `1` for 1-Wire bus power down (useful for bus reset in case of freeze); set `0` for normal operation.
* *RESERVED* - 
*  set `0`.
* *Counter 1/2 reset* - set `1` for `COUNTER1/COUNTER2 Value` reset request; set `0` for normal operation. Reset operation is edge-triggered and its completion is signalised by *Status byte / Counter 1/2 reset done* bit.
* [FUTURE] *Sign of Life 0/1* - should be incremented with each data transfer as 2bit number low/high bit, used by firmware as health check of *SPI Master*.

### PWMx frequency

f_PWM = 32 MHz / ( prescaler × TOP )

<pre>
bit  1 ..  0
  00  0 : prescaler =   1
  01  1 : prescaler =   8
  10  2 : prescaler =  64
  11  3 : prescaler = 512

bit 15 ..  2 : TOP / 4
</pre>

Example:

* `0xC352 = 50002` - prescaler = 64, TOP = 50000, f_PWM = 10 Hz

For conversion form required frequency in Hz to the most sutable value of process data register function `monarco_util_pwm_freq_to_u16` can be found in file `src/monarco_util.c` in the [C language driver Repository](https://github.com/monarco/monarco-hat-driver-c/blob/master/src/monarco_util.c)

### CRC

Master to Slave data integrity is protected by 16bit CRC (cyclic redundancy check). Monarco HAT (Slave) ignores data frames with bad CRC.

There are numerous varieties of CRC-16 in common use which differs by the polynomial and its representation. Monarco HAT use the `Modbus RTU CRC-16` / `CRC-16-IBM` type, polynomial: `x^16 + x^15 + x^2 + 1` which is `0x8005` (normal) / `0xA001` (reversed), initial value: `0xFFFF`. 


## Standard transfer structure - Host (RPi) RX from Monarco HAT (Slave to Master)

<pre>
offset  size
   0.0   4.0  Service channel response (spi_data_service_t)
   4.0   1.0  Status byte (spi_data_status_byte_t)
   5.0   2.0  RESERVED
   7.0   1.0  Digital inputs (bits 0..3 = DIN1..DIN4, bits 4..7 = RESERVED)
   8.0   4.0  COUNTER1 Value
  12.0   4.0  COUNTER2 Value
  16.0   4.0  COUNTER2 Capture
  20.0   2.0  Analog input 1 (0..4095 = 0..10 V, > 4095 = RESERVED)
  22.0   2.0  Analog input 2 (0..4095 = 0..10 V, > 4095 = RESERVED)
  24.0   2.0  CRC-16
============
  26.0 ([bytes].[bits])
</pre>

### Status byte

<pre>
offset
   0.0  RESERVED
   0.1  RESERVED
   0.2  RESERVED
   0.3  RESERVED
   0.4  Counter 1 reset done
   0.5  Counter 2 reset done
   0.6  Sign of Life 0
   0.7  Sign of Life 1
</pre>

* *Counter 1/2 reset done* - signalisation of counter reset operation completion (see *Control byte* description above).
* *Sign of life 0/1* - is incremented with each data transfer as 2bit number low/high bit, can be used by *SPI Master* as health check of the Monarco HAT.

### Analog values

Values of analog inputs are internally processed by factory-calibration constants.

For the voltage measurement mode, full range input value 10.0 V equals to process data value 4095. 

For the current-loop measurement mode, theoreretical full range input value 52.475 mA equals to process data value 4095.

Function for analog values conversion can be found in file `src/monarco_util.c` in the [C language driver Repository](https://github.com/monarco/monarco-hat-driver-c/blob/master/src/monarco_util.c) 

### CRC

Slave to Master data integrity is also protected by 16bit CRC. Master should ignore data frames received with bad CRC.

The same CRC type is used as for the RX (Master to Slave) direction.


## Service data channel principles

Service data channel (SDC) provides acyclic access to set of status and configuration registers. Register is identified by `ADDRESS` (12 bit, `0x000` to `0xFFF`) and contains 16 bit `VALUE` data. 

In current firmware version, all register values are initialised to its defaults after power on (or Monarco HAT MCU reset). Persistent storage of important configuration registers is planned for future versions.

### Protocol data structure and logic

Both service data channel Request and Response share the same 4 byte data structure (`spi_data_service_t`):

<pre>
offset  size
   0.0   2.0  VALUE - Data sent (request) / received (response). 16 bit unsigned value.
   2.0   1.4  ADDRESS - Register address to read / write.
   3.4   0.1  WRITE - Request write flag. If set, request VALUE will be written to ADDRESS.  
   3.5   0.1  ERROR - Response error flag. if set, response VALUE contains error code.
   3.6   0.2  RESERVED
</pre>

In each SPI transfer Master sends SDC Request and simultaneously receives SDC Response to previous request. Two Request types are defined:

* **Register read**
    * Request (Master > Slave): `ADDRESS` of register to be read. `VALUE = 0` (ignored). `WRITE = 0`. `ERROR = 0`.
    * Response (Slave > Master): `ADDRESS` have to be checked against the Request. `WRITE = 0`. `VALUE` contains read value if `ERROR = 0`, or Error Code if `ERROR = 1`. 

* **Register write**
    * Request (Master > Slave): `ADDRESS` of register to be written, `VALUE` to be written into the register. `WRITE = 1`. `ERROR = 0`.
    * Response (Slave > Master): `ADDRESS` and `VALUE` have to be checked against the Request if `ERROR = 0`, or `VALUE` contains Error Code if `ERROR = 1`. `WRITE = 1`.

If no register is currently required to be read or written, it is recommended to read register `0x000` - Status Word.

### Error Codes

* `0xFFFF`: Unknown Register / Write Not Allowed
* [FUTURE] separated codes for: write not allowed, invalid value, invalid state


## Service channel register description

Note: Registers are read-only [R], read-write [RW] or write-only [W].

* **0x000: Status Word [RW]**
  * Read:
    * `0xABCD` = OK
    *  TODO: "watchdog triggered"
  * Write - accept only `0xABCD` = reset status

---

* **0x001: Firmware Version - Low Word [R]**
* **0x002: Firmware Version - High Word [R]** 
* **0x003: Hardware Version - Low Word [R]**
* **0x004: Hardware Version - High Word [R]**
* **0x005: MCU Unique ID 1 [R]**
  * Monarco HAT onboard microcontroller (MCU) contains factory programmed 16 bytes unique identification number. It can be read from 4 consecutive registers.
* **0x006: MCU Unique ID 2 [R]**
* **0x007: MCU Unique ID 3 [R]**
* **0x008: MCU Unique ID 4 [R]**

---

* **0x00A: HW Config 1 [RW]**
  * `bit 00` - RS-485 termination resistor enable
  * `bit 01` - AIN1 current loop shunt resistor enable
  * `bit 02` - AIN2 current loop shunt resistor enable
  * `bit 03..15` - RESERVED (write zeros)

* **0x00F: Process Data Refresh Timeout [RW]**
  * If no valid SPI trasfer is performed during this period all outputs are switched to default states.
  * 0 = no timeout, feature disabled
  * Unit: 1 ms
  * Default = 1500 (since ver. 2.008, 100 before)

---

* **0x010: RS-485 Baudrate [RW]**
  * Unit: 100 Bd, Min: 3 (300 Bd), Max 10000 (1 MBd)
* **0x011: RS-485 Mode [RW]**
  * `bit 00..02` - parity - `0: none, 1: even, 2: odd`
  * `bit 03..04` - data bits: `0: 5, 1: 6, 2: 7, 3: 8`
  * `bit 05..06` - stop bits: `0: half, 1: one, 2: one and half, 3: two`
  * `bit 07..15` - RESERVED (write zeros)
* **0x012: Host UART Baudrate [RW]**
  * Unit: 100 Bd, Min: 3 (300 Bd), Max 10000 (1 MBd)
  * Set to 0 to disable Host UART (set Monarco pins to Hi-Z) when you are using it for another external device.
* **0x014: RS-485 Bytes Received Diagnostics Counter [R]**
* **0x015: RS-485 Bytes Transmitted Diagnostics Counter [R]**
* **0x018: RS-485 RX Framing Error Diagnostics Counter [R]**
* **0x019: RS-485 RX Parity Error Diagnostics Counter [R]**

---

* **0x024: COUNTER1 Configuration [W]**
    * `bit 00..02` - mode:
        * `0`: off
        * `1`: pulse counting
        * `2`: quadrature encoder
        * `3`: counting pulses generated by PWM1
        * `4`: counting pulses generated by PWM2
    * `bit 03..05` - direction (only for pulse counting mode):
        * `0`: up
        * `1`: [FUTURE] external control, low/high = up/down
    * `bit 06..07` - active edge (only for pulse counting mode):
        * `0`: rising
        * `1`: falling
        * `2`: both
    * `bit 08..15` - RESERVED (write zeros)
* **0x025: COUNTER2 Configuration [W]**
    * `bit 00..02` - mode:
        * `0`: off
        * `1`: pulse counting
        * `2`: quadrature encoder
    * `bit 03..05` - direction (only for pulse counting mode):
        * `0`: up
        * `1`: [FUTURE] external control, low/high = up/down
    * `bit 06..07` - active edge (only for pulse counting mode):
        * `0`: rising
        * `1`: falling
        * `2`: both
    * `bit 08..09` - CAPTURE function active edge:
        * `0`: disabled
        * `1`: [FUTURE] rising
        * `2`: [FUTURE] falling
        * `3`: [FUTURE] both  
    * `bit 10..15` - RESERVED (write zeros)

---

* **0x030: Host Watchdog Timeout [RW]** (since ver. 2.008)
  * 0 = Host Watchdog disabled (default)
  * Unit: 1 s
  * If the Monarco HAT receives no valid data via SPI within the watchdog interval, watchdog is triggered and the power supply for the host device (e.g. Raspberry Pi) is cut for 2 seconds. The interval typically allows for restarting and reloading your application.

* **0x031: Host Watchdog Retry Timeout [RW]** (since ver. 2.008)
  * 0 = Host Watchdog Retry feature disabled (default)
  * Unit: 1 s, Minimum: 60
  * The 2nd stage watchdog timer is started only when a watchdog-triggered reboot occurs. It is stopped when the first valid data arrives via SPI. If the Monarco HAT receives no valid data within the 2nd stage watchdog interval, watchdog is triggered and the power supply for the host device (e.g. Raspberry Pi) is cut for 2 seconds. The interval should allow for booting the operating system and starting your application.    

---

* **0xF00-0xF3F: Persistent parameters area (64 registers, 128 byte)**
    * Now for manufacturer use only.


## SPI data transfer example

<pre>
Master > Slave (hex): 00 00 00 00 01 FF 01 00 00 7D 00 00 00 00 00 00 80 0C 00 00 99 09 CC 04 E1 C6
</pre>

* Service channel request [4 bytes]:
  * Read register `0x000` request
* Process data [20 bytes]:
  * Control byte: `0x01`: status LED mask on, value off
  * User LEDs mask: `0xFF`
  * User LEDs value: `0x01`
  * Digital outputs: `0x00`
  * PWM1 frequency: `0x7D00` = `32000`: prescaler = 1, TOP = 32000: frequency = 1000 Hz
  * PWM1 Channel A (DOUT1) duty cycle: `0x0000`: PWM disabled
  * PWM1 Channel B (DOUT2) duty cycle: `0x0000`: PWM disabled
  * PWM1 Channel C (DOUT3) duty cycle: `0x0000`: PWM disabled
  * PWM2 frequency: `0x0C80` = `3200`: prescaler = 1, TOP = 3200: frequency = 10000 Hz
  * PWM2 Channel A (DOUT4) duty cycle: `0x0000`: PWM disabled
  * Analog output 1: `0x0999` = `2457`: 6.0 V
  * Analog output 2: `0x04CC` = `1228`: 3.0 V
* CRC-16 [2 bytes]: `0xC6E1`

<pre>
Slave > Master (hex): CD AB 00 00 80 00 00 00 FF FF 00 00 5F 03 00 00 44 33 22 11 94 09 C7 04 0F 66
</pre>

* Service channel response [4 bytes]:
  * Read register `0x000` success, value `0xABCD`
* Process data [20 bytes]: 
  * Status byte: `0x00`
  * RESERVED: `0x0000`
  * Digital inputs: `0x00`
  * COUNTER1 Value: `0x0000_FFFF`
  * COUNTER2 Value: `0x0000_035F`
  * COUNTER2 Capture: `0x1122_3344`
  * Analog input 1: `0x0994` = `2452`: 5.988 V
  * Analog input 2: `0x04C7` = `1223`: 2.987 V
* CRC-16 [2 bytes]: `0x660F`


## Timing considerations

### SPI bus timing

Do not perform multiple SPI transfers in a row withnout some delay between them. Firmware of the Monarco HAT needs some time (200us is safe) after last transfer end to prepare for next SPI transaction.

For implementation reasons, there should be some delay (10us is safe) between asserting chip select active and first clock edge. Standard Linux drivers on Raspberry Pi with access over `/dev/spidev` are safe with no need for extra tuning.

### Input sampling timing

In the current firmware version, digital, analog and counter inputs sampling is triggered by SPI data transfer completion. This means you allways read almost one cycle "old" input values.

This approach works well for short cycle times about 5 ms when sampling synchronicity is preffered.

With longer cycle times, you might prefer continuous sampling approach where you read as recent input values as possible. This option will be implemented if such a demand arises.
