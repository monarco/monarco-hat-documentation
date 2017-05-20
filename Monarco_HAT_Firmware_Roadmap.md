# Monarco HAT Firmware Development Roadmap

Monarco HAT firmware public repository: <https://github.com/monarco/monarco-hat-firmware-bin>


## Features implemented in v2.006 Release

* Analog inputs fatory calibration - supported since HW batch C (S/N 1701C...)
* Persistent parameters in flash memory - now for manufacturer use only


## Features implemented in v2.005 Release

* Ability to disable host side UART by setting its baudrate to 0 (in register `0x012`)
* Fixed PWM behaviour on frequency change, fixed setting PWM2 frequency at all


## Features implemented in v2.004 Release

* Digital inputs. COUNTER function with up-count on rising/falling/both edge mode and quadrature encoder mode.
* Digital outputs. PWM function with 1 Hz to 100 kHz frequency range.
* Analog inputs with 0 to 10 V / 0 to 20 mA switching.
* Analog outputs.
* RS-485 forwarding with full featured parameters configuration (speed, parity, data bits).
* Service data channel (SDC) communication with basic set of registers: firmware revision, CPU ID, HW configuration, RS485 configuration and diagnostics counters, COUNTER mode configuration.
* Process data watchdog which switches outputs to safe states in case of SPI Master failure.


## Features under development

* Configurable deboounce filters on digital inputs
* Configurable low pass filters on analog inputs
* Continuous input sampling mode (not triggered by SPI transfer)
* Counter direction controlled by external signal
* Counter CAPTURE function
* Persistent counter values
* Watchdog for Rapberry Pi
* Pulse/Dir profile generation for positioning (Motion Control)
* Persistent configuration parameters


## Known issues

* Slow update rate (10 Hz) of LED1-LED8 in default input/output state indication mode. 
