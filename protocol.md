protocol
========

Current version: 2.5.1

The intention of this protocol is to allow as much of the microcontroller to be controlled as possible from the host computer. This protocol was designed for the direct communication between a microcontroller and a software object on a host computer. The host software object should then provide an interface that makes sense in that environment.

The data communication format uses MIDI messages. It is not necessarily a MIDI device, first it uses a faster serial speed, and second, the messages don't always map the same.


Message Types
===

This protocol uses the [MIDI message format](http://www.midi.org/techspecs/midimessages.php), but does not use the whole protocol.
Most of the command mappings here will not be directly usable in terms of MIDI controllers and synths. It should co-exist with MIDI without trouble and can be parsed by standard MIDI interpreters. Just some of the message data is used
differently.


| type                  | command | MIDI channel | first byte          | second byte     |
| --------------------- | ------- | ------------ | ------------------- | --------------- |
| analog I/O message    | 0xE0    | pin #        | LSB(bits 0-6)       | MSB(bits 7-13)  |
| digital I/O message   | 0x90    | port         | LSB(bits 0-6)       | MSB(bits 7-13)  |
| report analog pin     | 0xC0    | pin #        | disable/enable(0/1) | - n/a -         |
| report digital port   | 0xD0    | port         | disable/enable(0/1) | - n/a -         |
|                       |         |              |                     |                 |
| start sysex           | 0xF0    |              |                     |                 |
| set pin mode(I/O)     | 0xF4    |              | pin # (0-127)       | pin mode        |
| set digital pin value | 0xF5    |              | pin # (0-127)       | pin value(0/1)  |
| sysex end             | 0xF7    |              |                     |                 |
| protocol version      | 0xF9    |              | major version       | minor version   |
| system reset          | 0xFF    |              |                     |                 |


Sysex-based sub-commands (0x00 - 0x7F) are used for an extended command set.

| type                  | sub-command | first byte       | second byte   | ...            |
| --------------------- | -------     | ---------------  | ------------- | -------------- |
| string                | 0x71        | char *string ... |               |                |
| firmware name/version | 0x79        | major version    | minor version | char *name ... |


Data Message Expansion
===

Two byte digital data format, second nibble of byte 0 gives the port number (eg 0x92 is the third port, port 2)
```
0  digital data, 0x90-0x9F, (MIDI NoteOn, bud different data format)
1  digital pins 0-6 bitmask
2  digital pin 7 bitmask
```

Analog 14-bit data format
```
0  analog pin, 0xE0-0xEF, (MIDI Pitch Wheel)
1  analog least significant 7 bits
2  analog most significant 7 bits
```
Version report format
```
0  version report header (0xF9) (MIDI Undefined)
1  major version (0-127)
2  minor version (0-127)
```

Control Messages Expansion
===

Set pin mode
```
0  set digital pin mode (0xF4) (MIDI Undefined)
1  set pin number (0-127)
2  mode (INPUT/OUTPUT/ANALOG/PWM/SERVO/I2C/ONEWIRE/STEPPER/ENCODER/SERIAL/PULLUP, 0/1/2/3/4/6/7/8/9/10/11)
```

Set digital pin value (added in v2.5)
```
0  set digital pin value (0xF5) (MIDI Undefined)
1  set pin number (0-127)
2  value (LOW/HIGH, 0/1)
```

Toggle analogIn reporting by pin
```
0  toggle analogIn reporting (0xC0-0xCF) (MIDI Program Change)
1  disable(0) / enable(non-zero)
```
*As of Firmata 2.4.0, upon enabling an analog pin, the pin value should be reported to the client
application.*

Toggle digital port reporting by port (second nibble of byte 0), eg 0xD1 is port 1 is pins 8 to 15
```
0  toggle digital port reporting (0xD0-0xDF) (MIDI Aftertouch)
1  disable(0) / enable(non-zero)
```
*As of Firmata 2.4.0, upon enabling a digital port, the port value should be reported to the client
application.*

Request version report
```
0  request version report (0xF9) (MIDI Undefined)
```

Sysex Message Format
===

System exclusive (sysex) messages are used to define sets of core and optional firmata features. Core features are related to functionality such as digital and analog I/O, querying information about the state and capabilities of the microcontroller board and the firmware running on the board. All core features are documented in this protocol.md file. Optional features extend the hardware capabilities beyond basic digital I/O and analog I/O and also provide APIs to interface with general and specific components and system services. Optional features are individually documented in separate markdown files.


Each firmata sysex message has a feature ID composed of either a single byte or an extended ID composed of 3 bytes where the first byte is always 0 to indicate it's an extended ID. The following table illustrates the structure. The most significant bit must be set to 0 in each byte between the `START_SYSEX` and `END_SYSEX` which frame the message.

| byte 0      | byte 1       | bytes 2 - N-1                             | byte N    |
| ----------- | ------------ | ----------------------------------------- | --------- |
| START_SYSEX | ID (01H-7DH) | PAYLOAD                                   | END_SYSEX |
| START_SYSEX | ID (00H)     | EXTENDED_ID (00H 00H - 7FH 7FH) + PAYLOAD | END_SYSEX |

Following are SysEx commands used for core features defined in this version of the protocol:

```
EXTENDED_ID                 0x00 // A value of 0x00 indicates the next 2 bytes define the extended ID
RESERVED               0x01-0x0F // IDs 0x01 - 0x0F are reserved for user defined commands
ANALOG_MAPPING_QUERY        0x69 // ask for mapping of analog to pin numbers
ANALOG_MAPPING_RESPONSE     0x6A // reply with mapping info
CAPABILITY_QUERY            0x6B // ask for supported modes and resolution of all pins
CAPABILITY_RESPONSE         0x6C // reply with supported modes and resolution
PIN_STATE_QUERY             0x6D // ask for a pin's current mode and state (different than value)
PIN_STATE_RESPONSE          0x6E // reply with a pin's current mode and state (different than value)
EXTENDED_ANALOG             0x6F // analog write (PWM, Servo, etc) to any pin
STRING_DATA                 0x71 // a string message with 14-bits per char
REPORT_FIRMWARE             0x79 // report name and version of the firmware
SAMPLING_INTERVAL           0x7A // the interval at which analog input is sampled (default = 19ms)
SYSEX_NON_REALTIME          0x7E // MIDI Reserved for non-realtime messages
SYSEX_REALTIME              0X7F // MIDI Reserved for realtime messages
```

The full set of core and optional Firmata feature IDs is defined in the [firmata-registry.md](https://github.com/firmata/protocol/blob/master/feature-registry.md) file. See the registry for more info on proposing a new feature and obtaining an feature ID.

Query Firmware Name and Version
---

The firmware name to be reported should be exactly the same as the name of the
Firmata client file, minus the file extension. So for StandardFirmata.ino, the
firmware name is: StandardFirmata.

Query firmware Name and Version
```
0  START_SYSEX       (0xF0)
1  queryFirmware     (0x79)
2  END_SYSEX         (0xF7)
```

Receive Firmware Name and Version (after query)
```
0  START_SYSEX       (0xF0)
1  queryFirmware     (0x79)
2  major version     (0-127)
3  minor version     (0-127)
4  first 7-bits of firmware name
5  second 7-bits of firmware name
... for as many bytes as it needs
N  END_SYSEX         (0xF7)
```

Extended Analog
---

As an alternative to the normal analog message, this extended version allows
addressing beyond pin 15 and supports sending analog values with any number of
bits. The number of data bits is inferred by the length of the message.

```
0  START_SYSEX              (0xF0)
1  extended analog message  (0x6F)
2  pin                      (0-127)
3  bits 0-6                 (least significant byte)
4  bits 7-13                (most significant byte)
... additional bytes may be sent if more bits are needed
N  END_SYSEX                (0xF7)
```

Capability Query
---

The capability query provides a list of all modes supported by each pin. Each mode is described by 2 bytes where the first byte is the pin mode (such as digital input, digital output, PWM) and the second byte is the resolution (or sometimes the type of pin such as RX or TX for a UART pin). A value of `0x7F` is used as a separator to mark the end each pin's list of modes. The number of pins supported is inferred by the message length.

### Capabilities query
```
0  START_SYSEX              (0xF0)
1  CAPABILITY_QUERY         (0x6B)
2  END_SYSEX                (0xF7)
```

### Capabilities response
```
0  START_SYSEX              (0xF0)
1  CAPABILITY_RESPONSE      (0x6C)
2  1st supported mode of pin 0
3  1st mode's resolution of pin 0
4  2nd supported mode of pin 0
5  2nd mode's resolution of pin 0
... additional modes/resolutions, followed by `0x7F`,
    to mark the end of the pin's modes. Subsequently, each pin
    follows with its modes/resolutions and `0x7F`,
    until all pins are defined.
N  END_SYSEX                (0xF7)
```

#### Supported Modes
The modes in the following list are the modes of operation that can be returned during the capability response:
```
DIGITAL_INPUT      (0x00)
DIGITAL_OUTPUT     (0x01)
ANALOG_INPUT       (0x02)
PWM                (0x03)
SERVO              (0x04)
SHIFT              (0x05)
I2C                (0x06)
ONEWIRE            (0x07)
STEPPER            (0x08)
ENCODER            (0x09)
SERIAL             (0x0A)
INPUT_PULLUP       (0x0B)
```

*If no modes are defined for a pin, no values are returned (other than the separator value `0x7F`) and it should be assumed that pin is unsupported by Firmata.*

#### Mode Resolution
The resolution byte serves a couple of different purpose:

1. The original purpose was to define the resolution for analog input, pwm, servo and other modes that define a specific resolution (such as 10-bit for analog).
2. The resolution byte has been adapted for another purpose for Serial/UART pins, it defines if the pin is RX or TX and which UART it belongs to. [RX0](https://github.com/firmata/protocol/blob/master/serial.md#serial-pin-capability-response) is the RX pin of UART0 (Serial on an Arduino for example), TX1 if the TX pin of UART1 (Serial1 on an Arduino).

Modes utilizing the resolution byte as resolution data:
```
DIGITAL_INPUT      (0x00) // resolution is 1 (binary)
DIGITAL_OUTPUT     (0x01) // resolution is 1 (binary)
ANALOG_INPUT       (0x02) // analog input resolution in number of bits
PWM                (0x03) // pwm resolution in number of bits
SERVO              (0x04) // servo resolution in number of bits
STEPPER            (0x08) // resolution is number number of bits in max number of steps
INPUT_PULLUP       (0x0B) // resolution is 1 (binary)
```

Modes utilizing the resolution byte to define type of pin:
```
SERIAL             (0x0A) // See description in [serial.md](https://github.com/firmata/protocol/blob/master/serial.md#serial-pin-capability-response)
// also to be added to I2C in the future to define SCL and SDA pins
```

*For other features (including I2C until updated) the resolution information is less important so a value of 1 is used.*

Analog Mapping Query
---

Analog messages are numbered 0 to 15, which traditionally refer to the Arduino
pins labeled A0, A1, A2, etc. However, these pis are actually configured using
"normal" pin numbers in the pin mode message, and when those pins are used for
non-analog functions. The analog mapping query provides the information about
which pins (as used with Firmata's pin mode message) correspond to the analog
channels.

Analog mapping query
```
0  START_SYSEX              (0xF0)
1  analog mapping query     (0x69)
2  END_SYSEX                (0xF7)
```

Analog mapping response
```
0  START_SYSEX              (0xF0)
1  analog mapping response  (0x6A)
2  analog channel corresponding to pin 0, or 127 if pin 0 does not support analog
3  analog channel corresponding to pin 1, or 127 if pin 1 does not support analog
4  analog channel corresponding to pin 2, or 127 if pin 2 does not support analog
... etc, one byte for each pin
N  END_SYSEX                (0xF7)
```

*The above 2 queries provide static data (should never change for a particular
board). Because this information is fixed and should only need to be read once,
these messages are designed for a simple implementation in StandardFirmata,
rather that bandwidth savings (eg, using packed bit fields).*


Pin State Query
---

The pin **state** is any data written to the pin (*it is important to note that
pin state != pin value*). For output modes (digital output,
PWM, and Servo), the state is any value that has been previously written to the
pin. For input modes, typically the state is zero. However, for digital inputs,
the state is the status of the pull-up resistor which is 1 if enabled, 0 if disabled.

The pin state query can also be used as a verification after sending pin modes
or data messages.

Pin state query
```
0  START_SYSEX              (0xF0)
1  pin state query          (0x6D)
2  pin                      (0-127)
3  END_SYSEX                (0xF7)
```

Pin state response
```
0  START_SYSEX              (0xF0)
1  pin state response       (0x6E)
2  pin                      (0-127)
3  pin mode (the currently configured mode)
4  pin state, bits 0-6
5  (optional) pin state, bits 7-13
6  (optional) pin state, bits 14-20
... additional optional bytes, as many as needed
N  END_SYSEX                (0xF7)
```

String
---

Send short string messages between the board and the client application. String length is limited
to half the buffer size - 3 (for Arduino this limits strings to 30 chars). Commonly used to report
error messages to the client.
```
0  START_SYSEX        (0xF0)
1  STRING_DATA        (0x71)
2  first char LSB
3  first char MSB
3  second char LSB
4  second char MSB
... additional bytes up to half the buffer size - 3 (START_SYSEX, STRING_DATA, END_SYSEX)
N  END_SYSEX (0xF7)
```

Sampling Interval
---

The sampling interval sets how often analog data and i2c data is reported to the
client. The default for the arduino implementation is 19ms. This means that every
19ms analog data will be reported and any i2c devices with read continuous mode
will be read.
```
0  START_SYSEX        (0xF0)
1  SAMPLING_INTERVAL  (0x7A)
2  sampling interval on the millisecond time scale (LSB)
3  sampling interval on the millisecond time scale (MSB)
4  END_SYSEX (0xF7)
```
