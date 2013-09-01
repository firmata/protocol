protocol
========

Current version: 2.3.6

The intention of this protocol is allow as much of the microcontroller to be controlled as possible from the host computer. This protocol then was designed for the direct communication between a microcontroller and an software object on a host computer. The host software object should then provide an interface that makes sense in that environment.

The data communication format uses MIDI messages. It is not necessarily a MIDI device, first it uses a faster serial speed, and second, the messages don't always map the same.

TO DO: provide a better description above.


Message Types
===

This protocol uses the MIDI message format, but does not use the whole protocol.
Most of the command mappings here will not be directly usable in terms of MIDI
controllers and synths. It should co-exist with MIDI without trouble and can be
parsed by standard MIDI interpreters. Just some of the message data is used
differently.

[MIDI format](http://www.harmony-central.com/MIDI/Doc/table1.html)

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

Control Messages Expansion
===

Sysex Message Format
===

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

Features
---

links to i2c, servos, stepper, shift, scheduler, onewire, etc.