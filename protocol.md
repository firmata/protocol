protocol
========

Documentation of the Firmata protocol.

The intention of this protocol is allow as much of the microcontroller to be controlled as possible from the host computer. This protocol then was designed for the direct communication between a microcontroller and an software object on a host computer. The host software object should then provide an interface that makes sense in that environment.

The data communication format uses MIDI messages. It is not necessarily a MIDI device, first it uses a faster serial speed, and second, the messages don't always map the same.

TO DO: provide a better description above.


Message Types
===

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