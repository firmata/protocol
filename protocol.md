protocol
========

Current version: 2.5.1

The intention of this protocol is to allow as much of the microcontroller to be controlled as possible from the host computer. This protocol was designed for the direct communication between a microcontroller and a software object on a host computer. The host software object should then provide an interface that makes sense in that environment.

The data communication format uses MIDI messages. It is not necessarily a MIDI device, first it uses a faster serial speed, and second, the messages don't always map the same.


Message Types
===

This protocol uses the [MIDI message format](http://www.midi.org/techspecs/midimessages.php), but does not use the whole protocol.
Most of the command mappings here will not be directly usable in terms of MIDI
controllers and synths. It should co-exist with MIDI without trouble and can be
parsed by standard MIDI interpreters. Just some of the message data is used
differently.


| type                  | command | MIDI channel | first byte          | second byte     |
| --------------------- | ------- | ------------ | ------------------- | --------------- |
| analog I/O message    | 0xE0    | pin #        | LSB(bits 0-6)       | MSB(bits 7-13)  |
| digital I/O message   | 0x90    | port         | LSB(bits 0-6)       | MSB(bits 7-13)  |
| report analog pin     | 0xC0    | pin #        | disable/enable(0/1) | - n/a -         |
| report digital port   | 0xD0    | port         | disable/enable(0/1) | - n/a -         |
|                       |         |              |                     |                 |
| start sysex           | 0xF0    |              |                     |                 |
| set pin mode(I/O)     | 0xF4    |              | pin # (0-127)       | pin state(0=in) |
| set digital pin value | 0xF5    |              | pin # (0-127)       | pin value(0/1)  |
| sysex end             | 0xF7    |              |                     |                 |
| protocol version      | 0xF9    |              | major version       | minor version   |


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

The idea for SysEx is to have a second command space using the first byte after
the SysEx start byte. The key difference is that data can be of any size, rather
than just one or two bytes for standard MIDI messages.

Generic SysEx Message
```
0   START_SYSEX (0xF0) (MIDI System Exclusive)
1   sysex command (0x00-0x7F)
... between 0 and MAX_DATA_BYTES 7-bit bytes of arbitrary data
N   END_SYSEX (0xF7) (MIDI End of SysEx - EOX)
```

Following are SysEx commands used in this version of the protocol:
```
RESERVED               0x00-0x0F // The first 16 bytes are reserved for custom commands
SERIAL_MESSAGE              0x60 // communicate with serial devices, including other boards
ENCODER_DATA                0x61 // reply with encoders current positions
REPORT_FEATURES             0x62 // report the features supported by this firmware (Servo, I2C, etc)
ANALOG_MAPPING_QUERY        0x69 // ask for mapping of analog to pin numbers
ANALOG_MAPPING_RESPONSE     0x6A // reply with mapping info
CAPABILITY_QUERY            0x6B // ask for supported modes and resolution of all pins
CAPABILITY_RESPONSE         0x6C // reply with supported modes and resolution
PIN_STATE_QUERY             0x6D // ask for a pin's current mode and state (different than value)
PIN_STATE_RESPONSE          0x6E // reply with a pin's current mode and state (different than value)
EXTENDED_ANALOG             0x6F // analog write (PWM, Servo, etc) to any pin
SERVO_CONFIG                0x70 // pin number and min and max pulse
STRING_DATA                 0x71 // a string message with 14-bits per char
STEPPER_DATA                0x72 // control a stepper motor
ONEWIRE_DATA                0x73 // send an OneWire read/write/reset/select/skip/search request
SHIFT_DATA                  0x75 // shiftOut config/data message (reserved - not yet implemented)
I2C_REQUEST                 0x76 // I2C request messages from a host to an I/O board
I2C_REPLY                   0x77 // I2C reply messages from an I/O board to a host
I2C_CONFIG                  0X78 // Enable I2C and provide any configuration settings
REPORT_FIRMWARE             0x79 // report name and version of the firmware
SAMPLEING_INTERVAL          0x7A // the interval at which analog input is sampled (default = 19ms)
SCHEDULER_DATA              0x7B // send a createtask/deletetask/addtotask/schedule/querytasks/querytask request to the scheduler
SYSEX_NON_REALTIME          0x7E // MIDI Reserved for non-realtime messages
SYSEX_REALTIME              0X7F // MIDI Reserved for realtime messages
```

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

Report features
---

Report standard and contributed features supported by the firmware and the version of each feature.

As of Firmata version 2.6.0 there are now 3 types of features:
1. Core features (those defined in this protocol.md file)
2. Standard features (I2C, Serial)
3. Contributed features (Encoder, OneWire, Stepper, Servo, Scheduler)

*Core features* are composed of both *Core Direct* messages such as `SystemReset(0xFF)`,
`ReportVersion(0xF9)`, `ReportDigital(0xD0)`, `ReportAnalog(0xE0)`, etc and *Core Sysex* messages
such as `ReportFirmware(0x79)`, `ExtendedAnalog(0x6F)`, `StringData(0x71)`, `CapabilityQuery(0x68)`,
etc.

*Standard features* are typically pin-level features of the microcontroller board such as I2C, SPI,
UART, CAN, etc.

*Contributed features* define generic (Servo, Stepper, OneWire) or device-specific (DHT11, NeoPixel)
interfaces or other services (Scheduler) that are not integral to the functionality of the
microcontroller itself or part of the core logic.

The *core* protocol version is defined at the top of this file and covers the features specified
in this document. The core version covers the Core Direct and Core Sysex messages. If any change is
made to those message, the core version needs to be incremented appropriately.

As of Firmata v2.6.0, standard and contributed features have their own version, beginning at v1.0.0
for each feature that was in use prior to v2.6.0 and beginning at v0.1.0 for any new features added
since core v2.6.0. The feature bugfix version is not reported but it should be maintained.

This enables changes to be made to individual features without needing to change the core version.
It also enables multiple variations of a feature to coexist simultaneously, for example a new I2C
feature could be added that enables the ability to use multiple I2C ports rather than a single
version. That enables Firmata client libraries to adapt the new I2C interface while deprecating the
old one without breaking client libraries that don't update to the new version. Both versions would
use the same pin mode but have different command identifiers.

**Query supported standard and contributed features.**
```
0  START_SYSEX                (0xF0)
1  REPORT_FEATURES            (0x62)
2  REPORT_FEATURES_QUERY      (0x00)
3  END_SYSEX                  (0xF7)
```

**Supported standard and contributes features query response.**

The `FEATURE_ID` is 2 bytes for scalability (composed as a pair of two 7-bit bytes rather than a
single number). Features existing prior core protocol v2.6.0 should assign their existing 1 byte
subcommand to the LSB of the ID and set the MSB to 0. For existing features that don't have a single
subcommand byte (I2C), the CONFIG subcommand should be the identifying subcommand, so for I2C the
ID would be: `0x0078`. This also means the the LSB in the range `0x50 - 0x7F` is reserved so new new
IDs cannot be allocated within this range: `0xXX50 - 0xXX7F`.
```
0  START_SYSEX                (0xF0)
1  REPORT_FEATURES            (0x62)
2  REPORT_FEATURES_RESPONSE   (0x01)
3  1st FEATURE_ID LSB         The subcommand byte allocated to the feature
4  1st FEATURE_ID MSB         use 2 bytes to ensure scalability
5  1st FEATURE_MAJOR_VERSION  (0-127)
6  1st FEATURE_MINOR_VERSION  (0-127)
7  2nd FEATURE_ID LSB
8  2nd FEATURE_ID MSB
9  2nd FEATURE_MAJOR_VERSION  (0-127)
10 2nd FEATURE_MINOR_VERSION  (0-127)
... for all supported features
n  END_SYSEX                  (0xF7)
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
... additionaly bytes may be sent if more bits are needed
N  END_SYSEX                (0xF7)
```

Capability Query
---

The capability query provides a list of all modes supported by each pin and
the resolution used by each pin. Each pin has 2 bytes for each supported mode
and a value of 127 to mark the end of that pin's data. The number of pins
supported is inferred by the message length. The resolution information may be
used to adapt to future implementation where PWM, analog input and others may
have different values (such as 12 or 14 bit analog instead of 10-bit analog).
*For some features such as i2c, the resolution information is less important so
a value of 1 is used.*

Capabilities query
```
0  START_SYSEX              (0xF0)
1  capabilities query       (0x6B)
2  END_SYSEX                (0xF7)
```

Capabilities response
```
0  START_SYSEX              (0xF0)
1  capabilities response    (0x6C)
2  1st mode supported of pin 0
3  1st mode's resolution of pin 0
4  2nd mode supported of pin 0
5  2nd mode's resolution of pin 0
... additional modes/resolutions, followed by a single 127 to mark the end of
    the pin's modes. Each pin follows with its mode and 127 until all pins are
    implemented.
N  END_SYSEX                 (0xF7)
```

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
the state is the status of the pullup resistor.

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

Features details
---
See specific files:

* [i2c](https://github.com/firmata/protocol/blob/master/i2c.md)
* [servos](https://github.com/firmata/protocol/blob/master/servos.md)
* [stepper](https://github.com/firmata/protocol/blob/master/stepper.md)
* [scheduler](https://github.com/firmata/protocol/blob/master/scheduler.md)
* [onewire](https://github.com/firmata/protocol/blob/master/onewire.md)
* [encoder](https://github.com/firmata/protocol/blob/master/encoder.md)
* [serial](https://github.com/firmata/protocol/blob/master/serial.md)
