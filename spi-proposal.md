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

### Overview

A `SPI_CONFIG` command is used to initialize the SPI bus. Up to 4 SPI ports
are supported using the `channel` parameter.

In the same spirit as the Arduino SPI library, a `SPI_BEGIN_TRANSACTION` command
and a `SPI_END_TRANSACTON` command are used to frame data transfers related to a
specific SPI device. This is useful in reinforcing that arbitrary reads and writes
can't occur between 2 or more SPI devices without being part of a transaction
(since each device may have a different clock speed, data mode, bit order, csPin,
etc).

A `SPI_TRANSFER` command enables data to be written to and read from a SPI device.
There are 3 ways to send and receive data from the SPI slave device:

1. **Read/Write** For each byte written a byte is read simultaneously.
2. **Write** Only write data (ignore any data returned by the slave device).
3. **Read** Only read data, writing `0x00` for each byte to be read.

A `SPI_REPLY` is used to send requested data back to the client application when
either **Read/write** mode or **Read-only** mode is used for the `SPI_TRANSFER`.

A `SPI_END` command disables the SPI bus for the specified channel.


### SPI_CONFIG

Use `SPI_CONFIG` to initialize the SPI bus. Up to 4 SPI ports are supported,
where each port is identified by a `channel` number (0-3).

`SPI_CONFIG` must be called at least once before sending any of the other
commands.

`channel` is used to identify which SPI bus is used in the case that a board
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

Initialize a SPI device or switch focus to a SPI device (if more than one device
is on the SPI bus). Call once before reading from or writing to a separate
SPI device.

The `deviceId` is any value between 0 and 31 and is chosen by the user. The
value should be unique for each device used in an application. The `deviceId`
will be returned with each `SPI_REPLY` message so it is clear which device the
data corresponds to.

A chip select pin (`csPin`) can optionally be specified. This pin will be
controlled per the rules specified in `pinOptions` byte of the `SPI_TRANSACTION`
command. For uncommon use cases of CS or other required HW pins per certain SPI
devices, it is better to control them separately by not specifying a CS pin and
instead using Firmata `DIGITAL_MESSAGE` to control the CS pin state. If a CS pin
is specified, the `csActiveState` for that pin must also be specified. The most
common active state is *Active LOW* where the CS pin is pulled LOW at the start
of the transfer and then set to HIGH at the end of the transfer.


```
0:  START_SYSEX
1:  SPI_DATA              (0x68)
2:  SPI_BEGIN_TRANSACTION (0x01)
3:  deviceId | channel    (bits 2-6: deviceId, bits 0-1: channel)
4:  dataMode | bitOrder   (bits 1-2: dataMode (0-3), bit 0: bitOrder)
5:  maxSpeed              (bits 0 - 6)
6:  maxSpeed              (bits 7 - 14)
7:  maxSpeed              (bits 15 - 21)
8:  maxSpeed              (bits 22 - 28)
9:  maxSpeed              (bits 29 - 32)
10  csPin                 [optional] (0-127) The chip select pin number
11  csActiveState         (required if csPin specified) 0: Active LOW, 1: Active HIGH
10|12: END_SYSEX
```

#### bitOrder
```
LSBFIRST = 0
MSBFIRST = 1
```

#### dataMode

| mode    | clock polarity (CPOL) | clock phase (CPHA) |
| --------|-----------------------|--------------------|
| 0       | 0                     | 0                  |
| 1       | 0                     | 1                  |
| 2       | 1                     | 0                  |
| 3       | 1                     | 1                  |


#### maxSpeed

The maximum speed of communication with the SPI device. For a SPI device rated up to
5 MHz, use 5000000.


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

Write to and read from a SPI device. There are 3 ways to send and receive data
from the SPI slave device:

1. **Read/Write** For each byte written a byte is read simultaneously.
2. **Write** Only write data (ignore any data returned by the slave device).
3. **Read** Only read data, writing `0x00` for each byte to be read.

The user can continue sending as many transfer messages as necessary, even
alternating between read-only, write-only and read/write as necessary within
a single SPI transaction.

If a chip select (CS) pin is specified in the `SPI_BEGIN_TRANSACTION` message, it
will be used according to the behavior specified by `pinOptions` parameter (byte 4).
Use bits 2, 3 and 4 for reading or writing data over multiple spi transfers. Set
bit 3 (CS_START_ONLY) for the first transfer, bit 2 (CS_DISABLE) for any middle transfers
and finally bit 4 (CS_END_ONLY) for the last transfer. Another option, `CS_TOGGLE`
(bit 5) allows the user to specify whether or not the CS pin should be toggled at the
end of the message (typical SPI behavior) or between each byte sent (edge case).


```
0:  START_SYSEX
1:  SPI_DATA              (0x68)
2:  SPI_TRANSFER          (0x03)
3:  channel               (HW supports multiple SPI ports. range = 0-3, default = 0)
4:  pinOptions | transferMode
                          bits 0-1: transferMode
                            0 = read/write,
                            1 = read-only,
                            2 = write-only
                            (see details in transferMode section below)
                          bits 2-4 : pinOptions
                            bit 2: CS_DISABLE
                            bit 3: CS_START_ONLY
                            bit 4: CS_END_ONLY
                            bit 5: CS_TOGGLE
                            (see details in pinOptions section below)
5.  numBytes              (number of bytes to read, write or read & write)
// if read-only, then no bytes sent, else write numBytes
6:  data 0                (bits 0-6)
7:  data 1                (bit 7) // or inverted depending on bitOrder setting
...                       up to numBytes * (8 / 7)
6|N:  END_SYSEX
```

#### transferMode

|B6-B2       |B1-B0       |
|------------|------------|
|pinOptions  |transferMode|

|B1-B0|Function                                                                        |
|-----|--------------------------------------------------------------------------------|
|00   |**Read/Write** Read a byte for every byte written.                              |
|01   |**Read only** Read the requested number of bytes, writing `0x00` for each requested byte.|
|10   |**Write only** Write to the slave device, but don't send a reply to the client. |

*When setting `00` (read/write) as the `transferMode`, a byte must be written for
every byte received. The user must then extract data from the array of bytes returned
in the `SPI_REPLY` message.*


#### pinOptions

|B6   |B5       |B4         |B3           |B2        |B1-B0       |
|-----|---------|-----------|-------------|----------|------------|
|0    |CS_TOGGLE|CS_END_ONLY|CS_START_ONLY|CS_DISABLE|transferMode|


**CS_DISABLE bit (2)** If set, disable CS pin control (even if a CS pin was specified in
the SPI_BEGIN_TRANSACTION command).

**CS_START_ONLY bit (3)** If set, set active state only at beginning of transfer. Useful
if breaking up a transfer across multiple packets.

**CS_END_ONLY bit (4)** If set, toggle active state only at end of transfer. Useful
if breaking up a transfer across multiple packets.

**CS_TOGGLE bit (5)** If set, toggle CS pin between every byte transfered rather than at
end of transfer.

*Note: It may be useful to separate pinOptions and transferMode to better scale to
handle additional use cases as they arise.*


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
...                      up to numBytes * (8 / 7)
N:  END_SYSEX
```

### SPI_END

Call once to release SPI hardware send before quitting a Firmata client
application.

```
0:  START_SYSEX
1:  SPI_DATA             (0x68)
2:  SPI_END              (0x05)
3:  channel              (HW supports multiple SPI ports. range = 0-3, default = 0)
4:  END_SYSEX
```