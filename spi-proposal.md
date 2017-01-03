SPI
===

A proposal for a SPI protocol for Firmata.

SPI is tricky to add to Firmata in a generic way since it is a fairly loose
standard. There are variations in number of bits written and read, how the CS
pin is used (if at all), use of other pins, etc. This proposal attempts to
accommodate uses cases beyond the common:

1. set cs pin LOW
2. write (and optionally read) 1 or more bytes
3. set cs Pin HIGH
4. return data if read

This document includes two different proposals: **Option A** and **Options B**.


# Option A

**The intent of Option A is to enable the user to send and receive the most data
using the least number of messages.**

In Option A, each SPI device is initialized via a config message. This message
sets the bit order, data mode, clock speed, and optional CS pin for the device.
Arrays of bytes can be transferred to and received from the slave device. If a
CS pin is specified, it is pulled low at the beginning of a transfer and
released after the last byte sent. However, the user can change the active CS
pin state (HIGH to LOW rather than the default LOW to HIGH behavior) or even
specify if the pin should be toggled between every byte vs the beginning and end
of every sequence of bytes.

A **sequence** byte is provide to allow breaking up a transfer into multiple
packets or even enabling a continuous data stream (provided HW doesn't have
tight timing requirements).

There are 3 ways to send and receive data from the SPI slave device:

1. Transfer - for each byte sent a byte is received
2. Write - only write data (ignore any data returned by the slave device)
3. Read - only read data, writing 0 for each byte to be read

### SPI_CONFIG_DEVICE

Send once for each SPI device, specifying a new deviceId for each device
(beginning from 0).

#### bitOrder
```
LSBFIRST = 0
MSBFIRST = 1
```

#### dataModes

| mode    | clock polarity (CPOL) | clock phase (CPHA) |
| --------|-----------------------|--------------------|
| 0       | 0                     | 0                  |
| 1       | 0                     | 1                  |
| 2       | 1                     | 0                  |
| 3       | 1                     | 1                  |


#### optionalPins

The `optionalPins` byte allows HW pins such as chip select, data ready, and
other common (or perhaps even uncommon) actions related to SPI HW that are
not directly handled by SPI libraries. Optional pins are identified as bits of
the `optionalPins` byte.

```
bit 0 = chip select
bit 1 = data ready
... additional bits TBD as needed
```
If one or more bits are set, then the pin number and an options byte are
defined for each pin (so 2 bytes per pin) in order beginning with bit 0.

The optional pins are described below:

```
Chip select (CS):
0:  csPin                (0-127)
1:  csOptions:
      csActive (bit 0)   (0 = LOW -> HIGH, 1 = HIGH -> LOW, default = 0)
      csToggle (bit 1)   (0 = toggle after last byte,
                          1 = toggle between bytes, default = 0)

Data ready:
0:  dataReadyPin         (0-127)
1:  dataReadyOptions:
      dataReadyActive (bit 0) (0 = LOW, 1 = HIGH, default = 1)
```

```
0:  START_SYSEX
1:  SPI_DATA             (0x68)
2:  SPI_CONFIG_DEVICE    (0x00)
3:  deviceId | channel   (bits 2-6: deviceId, bits 0-1: channel (0-3))
4:  bitOrder | dataMode  (bit 2: bitOrder, bits 0-1: dataMode (0-3))
                          see mode column in table above
5:  clockSpeed           (bits 0 - 6)
6:  clockSpeed           (bits 7 - 14)
7:  clockSpeed           (bits 15 - 21)
8:  clockSpeed           (bits 22 - 28)
9:  clockSpeed           (bits 29 - 32)
10: optionalPins         (see description above)
... defined optional pins (2 bytes each)
N:  END_SYSEX (N = 10 + (num optional pins * 2))
```

### SPI_TRANSFER_REQUEST

**Read a byte for each byte written.**

Send an array of data to transfer to the SPI slave device, expecting a byte
returned for each byte written. Respond with a `SPI_REPLY` message.

Use the **sequence** byte to specify if this is a one time transfer or if it is part
of a sequence of transfers, or an ongoing stream of transfers. Define as
follows:

```
0 = single transfer (no sequence)
1 = start sequence (1st part of sequence)
2 = 2nd part of sequence
3 = 3rd part of sequence, etc...
... up to 100, after 100, wrap back to 1
127 = end sequence
```

```
0:  START_SYSEX
1:  SPI_DATA             (0x68)
2:  SPI_TRANSFER_REQUEST (0x01)
3:  deviceId | channel   (bits 2-6: deviceId, bits 0-1: channel)
4:  sequence             (see note on sequence above)
5:  numBytes to read and write
6:  data 0               (bits 0-6)
7:  data 1               (bit 7) // or inverted depending on bitOrder setting
... up to numBytes * (8 / 7)
N:  END_SYSEX
```

### SPI_READ

Use if device has a ready signal or requires a delay after a write command.
Writes `0x00` for each byte to be read. Respond with a `SPI_REPLY` message.

```
0:  START_SYSEX
1:  SPI_DATA             (0x68)
2:  SPI_READ_REQUEST     (0x02)
3:  deviceId | channel   (bits 2-6: deviceId, bits 0-1: channel)
4:  sequence             (see note on sequence in Transfer section)
5:  numBytes to read
6:  END_SYSEX
```

### SPI_WRITE

Use when a transfer does not require a response.

```
0:  START_SYSEX
1:  SPI_DATA             (0x68)
2:  SPI_WRITE            (0x03)
3:  deviceId | channel   (bits 2-6: deviceId, bits 0-1: channel)
4:  sequence             (see note on sequence in Transfer section)
5:  numBytes to write
6:  data 0               (bits 0-6)
7:  data 1               (bit 7) // or inverted depending on bitOrder setting
... up to numBytes * (8 / 7)
N:  END_SYSEX
```

### SPI_REPLY

A byte array of data received from the SPI slave device in response to the
transfer. Used for both `SPI_TRANSFER_REQUEST` AND `SPI_READ_REQUEST`.

The **sequence** byte specifies which part of a sequence the reply corresponds to.
Helps ensure reply matches request.

```
0 = single transfer (no sequence)
1 = start sequence (1st part of sequence)
2 = 2nd part of sequence
3 = 3rd part of sequence, etc...
... up to 100, after 100, wrap back to 1
127 = end sequence
```

```
0:  START_SYSEX
1:  SPI_DATA             (0x68)
2:  SPI_REPLY            (0x04)
3:  deviceId | channel   (bits 2-6: deviceId, bits 0-1: channel)
4:  sequence             (see note on sequence above)
5:  numBytes in reply
6:  data 0               (bits 0-6)
7:  data 1               (bit 7) // or inverted depending on bitOrder setting
... up to numBytes * (8 / 7)
N:  END_SYSEX
```

### SPI_BEGIN

Call once to initialize SPI hardware. Can optionally be set internally upon
receiving first `SPI_CONFIG_DEVICE` message.

```
0:  START_SYSEX
1:  SPI_DATA             (0x68)
2:  SPI_BEGIN            (0x05)
3:  channel              (HW supports multiple SPI ports. range = 0-3, default = 0)
4:  END_SYSEX
```

### SPI_END

Call once to release SPI hardware send before quitting a Firmata client
application.

```
0:  START_SYSEX
1:  SPI_DATA             (0x68)
2:  SPI_END              (0x06)
3:  channel              (HW supports multiple SPI ports. range = 0-3, default = 0)
4:  END_SYSEX
```


# Option B

**The intent of Option B is to provide more flexibility at the expense of having
to send more messages.**

Option B relies on a `SPI_BEGIN_TRANSACTION` message and a `SPI_END_TRANSACTON`
message to frame data transfers related to a specific SPI device. This is useful
in reinforcing that arbitrary reads and writes can't occur between 2 or more
SPI devices without being part of a transaction (since each device may have a
different clock speed, data mode, bit order, csPin, etc).

An advantage of Option B over Option A is that it would require less memory
on the firmware side options are passed during each transaction, they don't
need to be retained across devices. Whereas in Option A an array of device
parameters would be required to track the options.


### SPI_CONFIG

This is more like SPI_BEGIN in Option A in that it initializes the SPI port
rather than a specific SPI device.

Call once to initialize SPI hardware. Can optionally be set internally upon
receiving first `SPI_CONFIG_DEVICE` message.

`channel` is used to identify which SPI port is used in the case that a board
has multiple ports (SPI, SPI1, SPI2, etc). Many only have one port so the
`channel` value will most often be `0`.

```
0:  START_SYSEX
1:  SPI_DATA             (0x68)
2:  SPI_CONFIG           (0x00)
3:  channel              (HW supports multiple SPI ports. range = 0-3, default = 0)
4:  END_SYSEX
```

### SPI_BEGIN_TRANSACTION

Initialize a SPI device. Call once before reading from or writing to a separate
SPI device.

The `deviceId` is any value between 0 and 31 and is chosen by the user. The
value should be unique for each device used in an application. The `deviceId`
will be returned with each `SPI_REPLY` message so it is clear which device the
data corresponds to.

A chip select pin (`csPin`) can optionally be specified. This pin will be
controlled per the rules specified in S`PI_TRANSACTION`. For uncommon use cases
of CS or other required HW pins per certain SPI devices, it is better to control
them separately using Firmata `DIGITAL_MESSAGE`.

```
0:  START_SYSEX
1:  SPI_DATA              (0x68)
2:  SPI_BEGIN_TRANSACTION (0x01)
3:  deviceId | channel    (bits 2-6: deviceId, bits 0-1: channel)
4:  bitOrder | dataMode   (bit 0: bitOrder, bits 1-2: dataMode (0-3))
5:  clockSpeed            (bits 0 - 6)
6:  clockSpeed            (bits 7 - 14)
7:  clockSpeed            (bits 15 - 21)
8:  clockSpeed            (bits 22 - 28)
9:  clockSpeed            (bits 29 - 32)
10  csPin                 [optional] (0-127) The chip select pin number
11: END_SYSEX
```

### SPI_END_TRANSACTION

Signals the end of a transaction. Must be sent before starting communication
with another SPI device.

```
0:  START_SYSEX
1:  SPI_DATA              (0x68)
2:  SPI_END_TRANSACTION   (0x02)
3:  channel               (HW supports multiple SPI ports. range = 0-3, default = 0)
4:  END_SYSEX
```

### SPI_TRANSFER

Write, read or read and write one or more bytes.

The user can continue sending as many transfer messages as necessary, even
alternating between read-only, write-only and read-write as necessary within
a single SPI transaction.

When setting `0` (read/write) as the `transferOption`, a byte must be written for
every byte received. Write `0x00` when you only need a byte returned. The user
must then extract data from the array of bytes returned in the `SPI_REPLY`
message.

If a CS pin is specified in the `SPI_BEGIN_TRANSACTION` message, it will be used
according to the behavior specified by `pinOptions` (byte 4). This only covers
common cases, with the default being set CS to `LOW `before sending data and set
to `HIGH` after the last byte has been sent. Using the `pinOptions` bits, it
also enables the user to set the CS pin value only on start (bit 4) or only on
end (bit 5) or ignore entirely (bit 2) in order to send data that requires
multiple `SPI_TRANSFER` messages. Another option (bit 6) allows the user to
specify whether or the CS pin should be toggled at the end of the message
(typical SPI behavior) or between each byte sent (edge case).

*Note: It may be useful to separate pinOptions and transferOptions to handling
additional use cases as they arise.*

*Note: TBD if a single SPI_TRANSFER message is sufficient or if we need
additional transfer messages such as SPI_TRANSFER_16.*


```
0:  START_SYSEX
1:  SPI_DATA              (0x68)
2:  SPI_TRANSFER          (0x03)
3:  channel               (HW supports multiple SPI ports. range = 0-3, default = 0)
4:  pinOptions | transferOptions
                          bits 0-1: transfer options
                            0 = read/write,
                            1 = read-only,
                            2 = write-only)
                            if read-only, a value of 0x00 will be written.
                            if read/write or read-only, respond with `SPI_REPLY`
                          bits 2-4 : pin options
                            bit 2: 0 = no CS, 1 use CS
                            bit 3: (if CS) 0 = start LOW -> HIGH, 1 = HIGH -> LOW
                            bit 4: (if CS) 1 = set CS at start only
                            bit 5: (if CS) 1 = set CS at end only
                            bit 6: (if CS) 0 = toggle after last byte (normal)
                                           1 = toggle between bytes
5.  numBytes              (number of bytes to read, write or read & write)
// if read-only, then no bytes sent, else write numBytes
6:  data 0                (bits 0-6)
7:  data 1                (bit 7) // or inverted depending on bitOrder setting
... up to numBytes * (8 / 7)
6|N:  END_SYSEX
```

### SPI_REPLY

A byte array of data received from the SPI slave device in response to a
`SPI_TRANSFER` read/write or read-only message.

```
0:  START_SYSEX
1:  SPI_DATA             (0x68)
2:  SPI_REPLY            (0x04)
3:  deviceId | channel   (bits 2-6: deviceId, bits 0-1: channel)
4:  numBytes
5:  data 0               (bits 0-6)
6:  data 1               (bit 7) // or inverted depending on bitOrder setting
... up to numBytes * (8 / 7)
N:  END_SYSEX
```