# Digital Pin Groups (Proposal)

Provides support for the situation where you want to set or get the values of 
an arbitrary set of digital IO pins that may not necessarily align to a port and 
do all of this in one operation.

Examples of this behaviour would include:

* Analog Multiplexer / Demultimplexer, where you need to set the bit value of
three pins in order to determine which analog line is being used on the multiplexer.
* Keypads where the value of the key presses are expressed using a combination of
states across a set of digital lines (eg: [https://www.sparkfun.com/products/8653](https://www.sparkfun.com/products/8653) )

When you want to issue an equivalent of a digitalWrite to a group of pins,
you'll then issue a sequence of 7-bit bytes that provides the states of the pins collectively. 
This will save several calls to digital write and allow them to be done in one group.

Reads will work the same way but return a byte with the states of all of the pins.
This is particularly important in a scenario like a keypad where independent
async reads can make it extremely challenging to get the state of the keypress properly.

## Requirements

* Currently unimplemented (PoC to come shortly)
* An array of pin groups (suggest 8 groups so it can be identifed with 3 bits 
each with up to 14 pins defined in the group)
* Modifications to firmata to accept the new protocol.

## Protocol

### Digital Pin Group commands

In order to save space in the protocol, the Digital Pin Group command comprises
both protocol commands as well as the id address space for the groups as below:

LSB
0 - 2:  3 bits to determine which Pin Group command is being issued
3    :  Reserved for future use / address space increases
4 - 6:  3 bits for Pin Group ID address space - providing up to 8 distinct digital pin groups

Command values are provided below
```
CONFIG              (0x00)
CLEAR               (0x01)
PIN_STATE_SET       (0x02)
PIN_STATE_REQUEST   (0x03)
PIN_STATE_REPLY     (0x04)
future reserved     (0x05 - 0x07)
```

### Configuration

Specifies which pins should be grouped together and in which order. A maximum
of 14 pins can be grouped together in one pin group. When specified in the config
message, the pins will be provided in little endian style so the first pin will
then be configured to mapped to the Least Significant Bit in subsequent write
and read messages.

```
0:  START_SYSEX         (0xF0)
1:  pin group command   (0x60)
2:  pin group id (0 - 7) << 4 | CONFIG
3:  lowest bit set for pinMode (0=READ, 1=WRITE) top 6 bits reserved
4:  first pin in pin group (0 - 127)
5:  second pin in pin group (0 - 127)
... up to maximum of 14
N:  END_SYSEX           (0xF7)
```

### Clear

Removes any pin entries associated to a pin group. This should free up any
memory that has been allocted and remove any references to the pins that were
configured. This is to ensure no side effects occur if a pin group is recycled.

```
0:  START_SYSEX         (0xF0)
1:  pin group command   (0x60)
2:  pin group id (0 -7) << 4 | CLEAR
3:  END_SYSEX           (0xF7)
```

### State set

Sets the states of the pins in the group. As noted above, the first pin that
is supplied in the config will be considered the least significant bit in this
message. Any values provided that don't match the config definition should simply
be ignored (ie a value comes through for the 5th pin in the group but none was
defined so it should be ignored).

```
0:  START_SYSEX         (0xF0)
1:  pin group command   (0x60)
2:  pin group id (0 - 7) << 4 | PIN_STATE_SET
3:  packed 7 bit array as single byte providing values for the pin group
... optional second packed 7 bit array providing values for the pin group
N:  END_SYSEX           (0xF7)
```

### State request and reply

Getting the states of the group of pins (essentially a group digital read)
comprises a request to the board and a reply back.

Make a request for getting the states of the pin group.

```
0:  START_SYSEX         (0xF0)
1:  pin group command   (0x60)
2:  pin group id (0 - 7) << 4 | PIN_STATE_REQUEST
3:  END_SYSEX
```

Reply with the pin states.

```
0:  START_SYSEX         (0xF0)
1:  pin group command   (0x60)
2:  pin group id (0 - 7) << 4 | PIN_STATE_REPLY
3:  packed 7 bit array representing pin states, LSB is first pin defined in config
... optional second 7 bit array representing pin states for additional pins in group
N:  END_SYSEX
```

