protocol
========

Current version: 2.3.6

The intention of this protocol is allow as much of the microcontroller to be controlled as possible from the host computer. This protocol then was designed for the direct communication between a microcontroller and an software object on a host computer. The host software object should then provide an interface that makes sense in that environment.

The data communication format uses MIDI messages. It is not necessarily a MIDI device, first it uses a faster serial speed, and second, the messages don't always map the same.

TO DO: provide a better description above.


Message Types
===

This protocol uses the [MIDI message format](http://www.midi.org/techspecs/midimessages.php), but does not use the whole protocol.
Most of the command mappings here will not be directly usable in terms of MIDI
controllers and synths. It should co-exist with MIDI without trouble and can be
parsed by standard MIDI interpreters. Just some of the message data is used
differently.


| type                | command | MIDI channel | first byte          | second byte     |
| ------------------- | ------- | ------------ | ------------------- | --------------- |
| analog I/O message  | 0xE0    | pin #        | LSB(bits 0-6)       | MSB(bits 7-13)  |
| digital I/O message | 0x90    | port         | LSB(bits 0-6)       | MSB(bits 7-13)  |
| report analog pin   | 0xC0    | pin #        | disable/enable(0/1) | - n/a -         |
| report digital port | 0xD0    | port         | disable/enable(0/1) | - n/a -         |
|                     |         |              |                     |                 |
| start sysex         | 0xF0    |              |                     |                 |
| set pin mode(I/O)   | 0xF4    |              | pin # (0-127)       | pin state(0=in) |
| sysex end           | 0xF7    |              |                     |                 |
| protocol version    | 0xF9    |              | major version       | minor version   |


Sysex-based commands (0x00 - 0x7F) are used for an extended command set.

| type                  | command | first byte          | second byte   | ...            |
| --------------------- | ------- | ------------------- | ------------- | -------------- |
| string                | 0x71    | char *string ...    |               |                |
| firmware name/version | 0x79    | major version       | minor version | char *name ... |


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
2  state (INPUT/OUTPUT/ANALOG/PWM/SERVO, 0/1/2/3/4/)
```

Toggle analogIn reporting by pin
```
0  toggle digitalIn reporting (0xC0-0xCF) (MIDI Program Change)
1  disable(0) / enable(non-zero)
```

Toggle digital port reporting by port (second nibble of byte 0), eg 0xD1 is port 1 is pins 8 to 15
```
0  toggle digital port reporting (0xD0-0xDF) (MIDI Aftertouch)
1  disable(0) / enable(non-zero)
```

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
                                 // (provide link to section describing custom commands)
ANALOG_MAPPING_QUERY        0x69 // ask for mapping of analog to pin numbers
ANALOG_MAPPING_RESPONSE     0x6A // reply with mapping info
CAPABILITY_QUERY            0x6B // ask for supported modes and resolution of all pins
CAPABILITY_RESPONSE         0x6C // reply with supported modes and resolution
PIN_STATE_QUERY             0x6D // ask for a pin's current mode and state (different than value)
PIN_STATE_RESPONSE          0x6E // reply with a pin's current mode and state (different than value)
EXTENDED_ANALOG             0x6F // analog write (PWM, Servo, etc) to any pin
SERVO_CONFIG                0x70 // pin number and min and max pulse
STRING_DATA                 0x71 // a string message with 14-bits per char
SHIFT_DATA                  0x75 // shiftOut config/data message (reserved - not yet implemented)
I2C_REQUEST                 0x76 // I2C request messages from a host to an I/O board
I2C_REPLY                   0x77 // I2C reply messages from an I/O board to a host
I2C_CONFIG                  0X78 // Enable I2C and provide any configuration settings
REPORT_FIRMWARE             0x79 // report name and version of the firmware
SAMPLEING_INTERVAL          0x7A // the interval at which analog input is sampled (default = 19ms)
SYSEX_NON_REALTIME          0x7E // MIDI Reserved for non-realtime messages
SYSEX_REALTIME              0X7F // MIDI Reserved for realtime messages
ENCODER_DATA                0X61 // Used by encoder feature
```

Query Firmware Name and Version
---

Extended Analog
---

Capability Query
---

Analog Mapping Query
---

Pin State Query
---

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
See specific files : 
* [i2c](https://github.com/firmata/protocol/i2c.md), 
* [servos](https://github.com/firmata/protocol/servo.md), 
* [stepper](https://github.com/firmata/protocol/stepper.md), 
* [shift](https://github.com/firmata/protocol/shift-proposal.md), 
* [scheduler](https://github.com/firmata/protocol/scheduler.md), 
* [onewire](https://github.com/firmata/protocol/onewire.md), 
* [encoder](https://github.com/firmata/protocol/encoder.md).