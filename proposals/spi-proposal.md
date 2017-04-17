SPI (Proposal)
===

A proposal for a SPI protocol for Firmata.

SPI is tricky to add to Firmata in a generic way since it is a fairly loose
standard. There are variations in number of bits written and read, how the CS
pin is used (if at all), use of other pins, etc. This proposal attempts to
accommodate uses cases beyond the common:

1. set cs pin LOW
2. write/read 1 or more words
3. set cs Pin HIGH
4. return data read

### Overview

A `SPI_CONFIG` command is used to initialize the SPI bus. Up to 4 SPI ports
are supported using the `channel` parameter.

In the same spirit as the Arduino SPI library, a `SPI_BEGIN_TRANSACTION` command
and a `SPI_END_TRANSACTON` command are used to frame data transfers related to a
specific SPI device. This is useful in reinforcing that arbitrary reads and writes
can't occur between 2 or more SPI devices without being part of a transaction
(since each device may have a different clock speed, data mode, bit order, csPin,
etc).

There are 3 ways to send and receive data from the SPI slave device:

1. `SPI_TRANSFER` For each word written a word is read simultaneously.
2. `SPI_WRITE` Only write data (ignore any data returned by the slave device).
3. `SPI_READ` Only read data, writing `0` for each word to be read.

A `SPI_REPLY` is used to send requested data back to the client application when
either a `SPI_TRANSFER` mode or `SPI_READ` message is sent.

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
1:  SPI_DATA              (0x68)
2:  SPI_CONFIG            (0x00)
3:  channel               (HW supports multiple SPI ports. range = 0-3, default = 0)
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
controlled per the rules specified in `csPinOptions`. For uncommon use cases
of CS or other required HW pins per certain SPI devices, it is better to control
them separately by not specifying a CS pin and instead using Firmata
`DIGITAL_MESSAGE` to control the CS pin state. If a CS pin is specified, the
`csPinOptions` for that pin must also be specified.


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
10: wordSize              (0 = DEFAULT = 8-bits, 1 = 1-bit, 2 = 2 bits, etc)
11: csPin                 [optional] (0-127) The chip select pin number
12: csPinOptions          (required if csPin specified)
                          bit 0: CS_ACTIVE_STATE (0 = Active LOW (default)
                                                  1 = Active HIGH)
                          bit 1: CS_TOGGLE (0 = toggle at end of transfer (default)
                                            1 = toggle between words)
11|13: END_SYSEX
```

#### bitOrder
```
LSBFIRST = 0 (default)
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

#### wordSize

The size of a `word` in bits. Typically 8-bits (default). 0 = DEFAULT = 8-bits, 1 = 1 bit,
2 = 2 bits, etc (limit is TBD).

#### csPin / csPinOptions

Specify a CS (chip select) pin to automatically toggle the pin rather than having to
send a separate `DIGITAL_MESSAGE` command with every transfer.

The default behavior is to pull the CS pin LOW at the beginning of a transfer and toggle
it at the end of the transfer. Setting bit 0 (`CS_ACTIVE_STATE`) will pull the CS pin HIGH
at the start of the transfer. Set bit 1 (`CS_TOGGLE`) to handle the rare use case of
toggling the CS pin between every word sent rather than at the end of the transaction.

The CS pin behavior can be further controlled by using the `csPinControl` byte of the
`SPI_TRANSFER`, `SPI_WRITE` or `SPI_READ` messages.


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

Full-duplex write/read transfer. This is the normal SPI transfer mode, a word must be
written for every word read. Reply is sent via `SPI_REPLY`.

The user can continue sending as many transfer messages as necessary, even
alternating between `SPI_TRANSFER`, `SPI_WRITE` and `SPI_READ` within a single SPI
transaction.

```
0:  START_SYSEX
1:  SPI_DATA              (0x68)
2:  SPI_TRANSFER          (0x03)
3:  channel               (HW supports multiple SPI ports. range = 0-3, default = 0)
4:  csPinControl          (see details in csPinControl section below)
                            0x00: default behavior (as specified by csPinOptions)
                            0x01: CS_DISABLE
                            0x02: CS_START_ONLY
                            0x03: CS_END_ONLY
5.  numWords              (number of words to transfer)
6:  data 0                (bits 0-6)
7:  data 0                (bits 7-14 if word size if word size > 7 && < 15)
8:  data 0                (if word size > 14)
...                       up to numWords * (wordSize / 7)
N:  END_SYSEX
```

#### csPinControl

If a chip select (CS) pin is specified in the `SPI_BEGIN_TRANSACTION` message, the
default behavior is to set the CS active state at the beginnnig of the transfer and
toggle the CS pin at the end of the transfer. Use the `csPinControl` to override the
default behavior. The primay use case for `csPinControl` is breaking up a large amount
of data over multipe transfers (for example due to buffer size limitiations). Use the
options `CS_DISABLE`, `CS_START_ONLY` and `CS_END_ONLY` as describe below.

`0x00: DEFAULT` Set `CS_ACTIVE_STATE` at beginning of transfer and toggle CS pin at end
unless `CS_TOGGLE` is also set (in that case toggle between every word).

`0x01: CS_DISABLE` Disable CS pin control (even if a CS pin was specified in the
`SPI_BEGIN_TRANSACTION` command). Useful to avoid toggling at the end of middle transfers
in when a large amount of data requires multiple transfers.

`0x02: CS_START_ONLY` Set active state only at beginning of transfer. Useful if breaking
up a transfer across multiple packets.

`0x03: CS_END_ONLY` Toggle active state only at end of transfer. Useful if breaking up a
transfer across multiple packets.


### SPI_WRITE

Only write data, ignoring any data returned by the slave device. No `SPI_REPLY` message
is sent.

Provided as a convenience. The same can be accomplished using `SPI_TRANSFRER` and ignoring
the `SPI_REPLY` message.

```
0:  START_SYSEX
1:  SPI_DATA              (0x68)
2:  SPI_WRITE             (0x04)
3:  channel               (HW supports multiple SPI ports. range = 0-3, default = 0)
4:  csPinControl          (see details in SPI_TRANSFER csPinControl description)
                            0x00: default behavior (as specified by csPinOptions)
                            0x01: CS_DISABLE
                            0x02: CS_START_ONLY
                            0x03: CS_END_ONLY
5.  numWords              (number of words to write)
6:  data 0                (bits 0-6)
7:  data 0                (bits 7-14 if word size if word size > 7 && < 15)
8:  data 0                (if word size > 14)
...                       up to numWords * (wordSize / 7)
N:  END_SYSEX
```

### SPI_READ

Only read data, writing `0` for each word to be read. Reply is sent via `SPI_REPLY`.

Provided as a convenience. The same can be accomplished using `SPI_TRANSFRER` and sending
a `0` for each byte to be read.

```
0:  START_SYSEX
1:  SPI_DATA              (0x68)
2:  SPI_WRITE             (0x05)
3:  channel               (HW supports multiple SPI ports. range = 0-3, default = 0)
4:  csPinControl          (see details in SPI_TRANSFER csPinControl description)
                            0x00: default behavior (as specified by csPinOptions)
                            0x01: CS_DISABLE
                            0x02: CS_START_ONLY
                            0x03: CS_END_ONLY
5.  numWords              (number of words to read)
6:  END_SYSEX
```

### SPI_REPLY

An array of data received from the SPI slave device in response to a
`SPI_TRANSFER` read/write or read-only message.

```
0:  START_SYSEX
1:  SPI_DATA              (0x68)
2:  SPI_REPLY             (0x06)
3:  deviceId | channel    (bits 2-6: deviceId, bits 0-1: channel)
4:  numWords
5:  data 0                (bits 0-6)
6:  data 0                (bits 7-14 if word size if word size > 7 && < 15)
7:  data 0                (if word size > 14)
...                       up to numWords * (wordSize / 7)
N:  END_SYSEX
```

### SPI_END

Call once to release SPI hardware send before quitting a Firmata client
application.

```
0:  START_SYSEX
1:  SPI_DATA              (0x68)
2:  SPI_END               (0x07)
3:  channel               (HW supports multiple SPI ports. range = 0-3, default = 0)
4:  END_SYSEX
```