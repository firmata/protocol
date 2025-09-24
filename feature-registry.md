# Firmata sysex feature registry

The feature registry defines allocated and proposed Firmata feature IDs. The feature ID is the 2nd byte in the sysex message. An extended set of IDs is also available by setting the initial ID byte to `00H` and then following with a 2 byte ID. All bytes between `START_SYSEX` and `END_SYSEX` must have the most significant bit set to 0.


| byte 0      | byte 1       | bytes 2 - N-1                             | byte N    |
| ----------- | ------------ | ----------------------------------------- | --------- |
| START_SYSEX | ID (01H-7DH) | PAYLOAD                                   | END_SYSEX |
| START_SYSEX | ID (00H)     | EXTENDED_ID (00H 00H - 7FH 7FH) + PAYLOAD | END_SYSEX |

Proposing a new feature
===

There are two different feature sets: [Core features](#core-feature-set) and [optional features](#optional-feature-set). See the descriptions for each type of feature set below. To propose a new core feature, open an issue to start a discussion. To propose a new optional feature, [open an issue](https://github.com/firmata/protocol/issues)/and or a pull request adding a markdown file for the proposed feature. Also edit the [optional feature set table](#optional-feature-set) to reserve an ID for the proposed feature and enter the status as "proposed". If the proposed feature exposes a very specific device or device driver (a NeoPixel light strip for example), assign an ID in the extended ID set (`00H nnH nnH`).


Core feature set
===

Core features are related to functionality such as digital and analog I/O, querying information about the state and capabilities of the microcontroller board and the firmware running on the board. The core features are documented in the [protocol.md](https://github.com/firmata/protocol/blob/master/protocol.md) file and the full set of core features is versioned together using [semver](http://semver.org/) notation. The current protocol version is 2.5.1.

Firmata firmware should report the current protocol version (using the [protocol version command: 0xF9](https://github.com/firmata/protocol/blob/master/protocol.md#message-types)) and implement the full set of current core features defined for that version (with the exception of very limited hardware which can implement a subset of the core feature set).

*The range 01H - 0FH is reserved for user-defined features that are not added to this registry.*


| Feature ID  | Feature name / link to documentation | Status     |
| ----------- | ------------------------------------ | ---------  |
| 00H         | EXTENDED_ID                          | proposed   |
| 01H - 0FH   | *Reserved for user features*         | n/a        |
| 63H         | [REPORT_DIGTIAL_PIN - proposal](https://github.com/firmata/protocol/issues/68#issuecomment-257105540) | proposed |
| 64H         | [EXTENDED_REPORT_ANALOG](https://github.com/firmata/protocol/issues/68#issuecomment-258748963) | current |
| 65H         | [REPORT_FEATURES - proposal](https://github.com/firmata/protocol/issues/41) | proposed |
| 66H         | [SYSTEM_VARIABLES](https://github.com/firmata/protocol/blob/master/system-variables.md) | current |
| 69H         | [ANALOG_MAPPING_QUERY](https://github.com/firmata/protocol/blob/master/protocol.md#analog-mapping-query) | current |
| 6AH         | [ANALOG_MAPPING_RESPONSE](https://github.com/firmata/protocol/blob/master/protocol.md#analog-mapping-query) | current |
| 6BH         | [CAPABILITY_QUERY](https://github.com/firmata/protocol/blob/master/protocol.md#capability-query) | current |
| 6CH         | [CAPABILITY_RESPONSE](https://github.com/firmata/protocol/blob/master/protocol.md#capability-query) | current |
| 6DH         | [PIN_STATE_QUERY](https://github.com/firmata/protocol/blob/master/protocol.md#pin-state-query) | current |
| 6EH         | [PIN_STATE_RESPONSE](https://github.com/firmata/protocol/blob/master/protocol.md#pin-state-query) | current |
| 6FH         | [EXTENDED_ANALOG](https://github.com/firmata/protocol/blob/master/protocol.md#extended-analog) | current |
| 71H         | [STRING_DATA](https://github.com/firmata/protocol/blob/master/protocol.md#string) | current |
| 79H         | [REPORT_FIRMWARE](https://github.com/firmata/protocol/blob/master/protocol.md#query-firmware-name-and-version) | current |
| 7AH         | [SAMPLING_INTERVAL](https://github.com/firmata/protocol/blob/master/protocol.md#sampling-interval) | current |
| 7CH         | [ANALOG_CONFIG - proposal](https://github.com/firmata/protocol/pull/8/files) | proposed |
| 7EH         | SYSEX_NON_REALTIME*                  | n/a        |
| 7FH         | SYSEX_REALTIME*                      | n/a        |

**7EH and 7FH are reserved because they have a special meaning to midi parsers.*


Optional feature set
===

Optional features extend the hardware capabilities beyond basic digital I/O and analog I/O (eg: I2C, Serial/UART, etc). Optional features also provide APIs to interface with general components (eg: servo, stepper, rotary encoder, etc) as well as specific components (eg: DHT11, NeoPixel, etc). The optional feature set also encompass functionality such as a general purpose scheduler API and a standardized device interface API. General features should use the single byte feature ID (allocating new IDs in descending order). However, any feature that wraps a specific driver, specific sensor, one-off custom component, etc should use the extended feature ID (`00H nnH nnH`) or should use the `DEVICE_QUERY/RESPONSE` API.

Each feature should be documented in a markdown file and versioned independently using [semver](http://semver.org/) notation. In the case where a feature spans multiple IDs (I2C for example), that entire set is documented in a single file and versioned together.


| Feature ID  | Feature name                       | Link to documentation  | Status     |
| ----------- | ---------------------------------- | ---------------------- | ---------- |
| 5CH         | RCOUTPUT_DATA                      | [rcswitch-proposal.md](https://github.com/firmata/protocol/blob/master/proposals/rcswitch-proposal.md) | proposed |
| 5DH         | RCINPUT_DATA                       | [rcswitch-proposal.md](https://github.com/firmata/protocol/blob/master/proposals/rcswitch-proposal.md) | proposed |
| 5EH         | DEVICE_QUERY                       | [proposal](https://github.com/finson-release/Luni/blob/master/extras/v0.9/v0.8-device-driver-C-firmata-messages.md) | proposed |
| 5FH         | DEVICE_RESPONSE                    | [proposal](https://github.com/finson-release/Luni/blob/master/extras/v0.9/v0.8-device-driver-C-firmata-messages.md) | proposed |
| 60H         | SERIAL_DATA (1.0)                  | [serial-1.0.md](https://github.com/firmata/protocol/blob/master/serial-1.0.md) | current |
| 61H         | ENCODER_DATA                       | [encoder.md](https://github.com/firmata/protocol/blob/master/encoder.md) | current |
| 62H         | ACCELSTEPPER_DATA                  | [accelStepperFirmata.md](https://github.com/firmata/protocol/blob/master/accelStepperFirmata.md) | current |
| 67H         | SERIAL_DATA (2.0)                  | [proposal](https://github.com/firmata/protocol/blob/master/proposals/serial-2.0-proposal.md) | proposed |
| 68H         | SPI_DATA                           | [spi.md](https://github.com/firmata/protocol/blob/master/spi.md) | beta |
| 70H         | SERVO_CONFIG                       | [servos.md](https://github.com/firmata/protocol/blob/master/servos.md) | current |
| 72H         | STEPPER_DATA                       | [stepper-legacy.md](https://github.com/firmata/protocol/blob/master/stepper-legacy.md) | deprecated |
| 73H         | ONEWIRE_DATA                       | [onewire.md](https://github.com/firmata/protocol/blob/master/onewire.md) | current |
| 74H         | DHTSENSOR_DATA                     | [dhtsensor.md](https://github.com/firmata/protocol/blob/master/dhtsensor.md) | beta |
| 75H         | SHIFT_DATA                         | [shift-proposal.md](https://github.com/firmata/protocol/blob/master/proposals/shift-proposal.md) | proposed |
| 76H         | I2C_REQUEST                        | [i2c.md](https://github.com/firmata/protocol/blob/master/i2c.md) | current |
| 77H         | I2C_REPLY                          | [i2c.md](https://github.com/firmata/protocol/blob/master/i2c.md) | current |
| 78H         | I2C_CONFIG                         | [i2c.md](https://github.com/firmata/protocol/blob/master/i2c.md) | current |
| 7BH         | SCHEDULER_DATA                     | [scheduler.md](https://github.com/firmata/protocol/blob/master/scheduler.md) | current |
| 7DH         | FREQUENCY_COMMAND                  | [frequency.md](https://github.com/firmata/protocol/blob/master/frequency.md) | beta |
|             |                                    |                        |            |
| 00H nnH nnH | (start of extended feature ID set) |                        |            |

