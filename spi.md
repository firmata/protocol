SPI (Proposal)
===

SPI protocol for Firmata.

SPI is tricky to add to Firmata in a generic way since it is a fairly loose
standard. There are variations in number of bits written and read, how the CS
pin is used (if at all), use of other pins, etc. This proposal attempts to
accommodate uses cases beyond the common sequence of:

1. set cs pin LOW
2. write/read 1 or more words
3. set cs Pin HIGH
4. return data read

[Implementation Note: ConfigurableFirmata has a basic implementation for SPI, but not all 
features are implemented]

### Overview

A `SPI_BEGIN` command is used to initialize the SPI bus. Up to 8 SPI ports
are supported using the `channel` parameter.

The `SPI_DEVICE_CONFIG` command is used to configure each attached SPI device.

There are 3 ways to send and receive data from the SPI slave device:

1. `SPI_TRANSFER` For each word written a word is read simultaneously.
2. `SPI_WRITE` Only write data (ignore any data returned by the slave device).
3. `SPI_READ` Only read data, writing `0` for each word to be read.

A `SPI_REPLY` is used to send requested data back to the client application when
either a `SPI_TRANSFER` mode or `SPI_READ` command is sent.

A `SPI_END` command disables the SPI bus for the specified channel.

The `CAPABILITY_RESPONSE` will return a value of 0x0C for SPI capable pins. Most
microcontrollers have only one set of pins for SPI. 

### SPI_BEGIN

Use `SPI_BEGIN` to initialize the SPI bus. Up to 8 SPI ports are supported,
where each port is identified by a `channel` number (0-7).

`SPI_BEGIN` must be called at least once before sending any of the other
commands.

`channel` is used to identify which SPI bus is used in the case that a board
has multiple ports (SPI, SPI1, SPI2, etc). Many boards only have one port so the
`channel` value will most often be `0`.

```
0:  START_SYSEX
1:  SPI_DATA              (0x68)
2:  SPI_BEGIN             (0x00)
3:  channel               (HW supports multiple SPI ports. range = 0-7, default = 0)
4:  END_SYSEX
```

### SPI_DEVICE_CONFIG

Send this command once for each attached SPI device to initialize it before use.
See parameter descriptions below.

```
0:  START_SYSEX
1:  SPI_DATA              (0x68)
2:  SPI_DEVICE_CONFIG     (0x01)
3:  deviceId | channel    (bits 3-6: deviceId, bits 0-2: channel)
4:  dataMode | bitOrder   (bit3: use 7 bit encoding, bits 1-2: dataMode (0-3), bit 0: bitOrder)
5:  maxSpeed              (bits 0 - 6)
6:  maxSpeed              (bits 7 - 14)
7:  maxSpeed              (bits 15 - 21)
8:  maxSpeed              (bits 22 - 28)
9:  maxSpeed              (bits 29 - 32)
10: wordSize              (0 = DEFAULT = 8-bits, 1 = 1-bit, 2 = 2 bits, etc)
11: csPinOptions          bit 0: CS_PIN_CONTROL (0 = disable
                                                 1 = enable (default))
                          bit 1: CS_ACTIVE_STATE (0 = Active LOW (default)
                                                  1 = Active HIGH)
                          bits 2-6: reserved for future options
12: csPin                 (0-127) The chip select pin number (ignored if
                          CS_PIN_CONTROL set to 0)
13: END_SYSEX
```

#### deviceId

The `deviceId` may either be used as a specific identifier (Linux) or as an
arbitrary identifier (Arduino). The `deviceId` value range is from 0 to 15 and
can be specified separately for each SPI channel. The `deviceId` will also be
returned with the channel for each `SPI_REPLY` command so it is clear which
device the data corresponds to.

#### bitOrder
```
LSBFIRST = 0
MSBFIRST = 1 (default)
```

#### dataMode

| mode    | clock polarity (CPOL) | clock phase (CPHA) |
| --------|-----------------------|--------------------|
| 0       | 0                     | 0                  |
| 1       | 0                     | 1                  |
| 2       | 1                     | 0                  |
| 3       | 1                     | 1                  |

#### use 7 bit encoding

If this bit is set, the SPI_TRANSFER command expects the data in "7-bit" mode. Instead of
using 2 bytes for bits 0-6 and 7 of each byte, 7 data bytes are packed into 8 transmitted bytes.
This mode is only supported for the default wordSize of 8 bits.

#### maxSpeed

The maximum speed of communication with the SPI device. For a SPI device rated
up to 5 MHz, use 5000000.

*For platforms that use a clock divider instead of a speed, pass the clock
divider value instead.* 

#### wordSize

The size of a `word` in bits. Typically 8-bits (default). 0 = DEFAULT = 8-bits,
1 = 1 bit, 2 = 2 bits, etc (limit is TBD).

#### csPinOptions / csPin

Use `CS_ACTIVE_STATE` to set the active state (typically LOW) for the CS pin. If
the platform's SPI implementation does not already automatically handle the CS
pin (it's handled automatically on Raspberry Pi, but not Arduino boards for
example), then set `CS_PIN_CONTROL` to `enable` and specify the `csPin` number
in byte 12. If the platform already handles the csPin then set
`CS_PIN_CONTROL` to `disable` and the `csPin` number will be ignored (set to
zero). For non-Linux platforms such as Arduino, to enable manual control of
the CS pin using `DIGITAL_MESSAGE` commands, set `CS_PIN_CONTROL` to `disable`.

### SPI_TRANSFER

Full-duplex write/read transfer. This is the normal SPI transfer mode, a word
must be written for every word read. Reply is sent via `SPI_REPLY`.

Since transport (Serial, Ethernet, Wi-Fi, etc) buffers tend to be small on
microcontroller platforms, it may be necessary to send several `SPI_TRANSFER`
commmands to complete a single SPI transaction. Use the `deselectCsPin`
parameter to ensure the CS pin is not deselected in between `SPI_TRANSFER`
commands until the transaction is complete.

`requestId` is used in the request messages `SPI_TRANSFER`, `SPI_WRITE` and
`SPI_READ` and in the reply message `SPI_REPLY`. Its purpose is to ensure that
the SPI_REPLY message matches the request. For each request message, increment
a single 7-bit requestId value, rolling it over to 0 when > 127.

`deselectCsPin` is used to control the csPin at the end of the transfer. By
default the csPin will be deselected at the end of every transfer. However, to
prevent deselection to enable back-to-back transfers for example, set
`deselectCsPin` to `0` and the pin state won't be affected at the end of the
transfer.

If `CS_PIN_CONTROL` is enabled, then the CS pin active state will be set when
the `SPI_TRANSFER` command is received. It will only be deselected at the end of
the transfer if `deselectCsPin` is set to 1.

```
0:  START_SYSEX
1:  SPI_DATA              (0x68)
2:  SPI_TRANSFER          (0x02)
3:  deviceId | channel    (bits 3-6: deviceId, bits 0-2: channel)
4:  requestId             (0-127) // increment for each call
5:  deselectCsPin         (0 = don't deselect csPin
                          1 = deselect csPin (default))
6.  numWords              (0-127: number of words to transfer)
7:  data 0                (bits 0-6)
8:  data 0                (bits 7-14 if word size if word size > 7 && < 15)
9:  data 0                (if word size > 14)
...                       up to numWords * (wordSize / 7)
N:  END_SYSEX
```

### SPI_WRITE

Only write data, ignoring any data returned by the slave device.

Provided as a convenience. The same can be accomplished using `SPI_TRANSFER`
and ignoring the `SPI_REPLY` command.

If `CS_PIN_CONTROL` is enabled, then the CS pin active state will be set when
the `SPI_WRITE` command is received. It will only be deselected at the end of
the write if `deselectCsPin` is set to 1.

A `SPI_WRITE` command does not return any data.

```
0:  START_SYSEX
1:  SPI_DATA              (0x68)
2:  SPI_WRITE             (0x03)
3:  deviceId | channel    (bits 3-6: deviceId, bits 0-2: channel)
4:  requestId             (0-127) // increment for each call
5:  deselectCsPin         (0 = don't deselect csPin
                          1 = deselect csPin (default))
6.  numWords              (0-127: number of words to write)
7:  data 0                (bits 0-6)
8:  data 0                (bits 7-14 if word size if word size > 7 && < 15)
9:  data 0                (if word size > 14)
...                       up to numWords * (wordSize / 7)
N:  END_SYSEX
```

### SPI_WRITE_ACK

`SPI_WRITE_ACK (0x07)` is almost identical to `SPI_WRITE`, except that the
command should return a `SPI_REPLY` with a data length of zero. The other
fields in the reply will be filled. This is for scenarios where no 
hardware flow control is available and writing large blocks of data would
result in a transmission buffer overflow.

### SPI_READ

Only read data, writing `0` for each word to be read. Reply is sent via
`SPI_REPLY`.

Provided as a convenience. The same can be accomplished using `SPI_TRANSFER`
and sending a `0` for each byte to be read.

If `CS_PIN_CONTROL` is enabled, then the CS pin active state will be set when
the `SPI_READ` command is received. It will only be deselected at the end of the
read if `deselectCsPin` is set to 1.

```
0:  START_SYSEX
1:  SPI_DATA              (0x68)
2:  SPI_READ              (0x04)
3:  deviceId | channel    (bits 3-6: deviceId, bits 0-2: channel)
4:  requestId             (0-127)  // increment for each call
5:  deselectCsPin         (0 = don't deselect csPin
                          1 = deselect csPin (default))
6.  numWords              (0-127: number of words to read)
7:  END_SYSEX
```

### SPI_REPLY

An array of data received from the SPI slave device in response to a
`SPI_TRANSFER` or `SPI_READ` command. The `requestId` should match the ID
from the transfer, read or write command.

```
0:  START_SYSEX
1:  SPI_DATA              (0x68)
2:  SPI_REPLY             (0x05)
3:  deviceId | channel    (bits 3-6: deviceId, bits 0-2: channel)
4:  requestId             (0-127) // must match the ID from the request
5:  numWords              (0-127: number of words in the reply)
6:  data 0                (bits 0-6)
7:  data 0                (bits 7-14 if word size if word size > 7 && < 15)
8:  data 0                (if word size > 14)
...                       up to numWords * (wordSize / 7)
N:  END_SYSEX
```

### SPI_END

Call to release SPI hardware send before quitting a Firmata client application.

```
0:  START_SYSEX
1:  SPI_DATA              (0x68)
2:  SPI_END               (0x06)
3:  channel               (HW supports multiple SPI ports. range = 0-7, default = 0)
4:  END_SYSEX
```
