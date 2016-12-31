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

Each SPI device is initialized via a config message. This message sets the bit
order data mode clock speed and optional CS pin for the device. Arrays of bytes
can be transferred to and received from the slave device. If a CS pin is
specified, it is pulled low at the beginning of a transfer and released after
the last byte sent. However, the user can change the active CS pin state (HIGH
to LOW rather than the default LOW to HIGH behavior) or even specify if the
pin should be toggled between every byte vs the beginning and end of every
sequence of bytes.

A **sequence** byte is provide to allow breaking up a transfer into multiple
packets or even enabling a continuous data stream (provided HW doesn't have
tight timing requirements).

There are 3 ways to send and receive data from the SPI slave device:

1. Transfer - for each byte sent a byte is received
2. Write - only write data (ignore any data retuned by the slave device)
3. Read - only read data, writing 0 for each byte to be read


## Protocol details

### Config

Send once for each SPI device, specifying a new deviceId for each device
(beginning from 0).

Bit order: LSBFIRST = 0, MSBFIRST = 1

Data modes:

| mode    | clock polarity (CPOL) | clock phase (CPHA) |
| --------|-----------------------|--------------------|
| 0       | 0                     | 0                  |
| 1       | 0                     | 1                  |
| 2       | 1                     | 0                  |
| 3       | 1                     | 1                  |


```
0:  START_SYSEX
1:  SPI_DATA             (0x68)
2:  SPI_CONFIG_DEVICE    (0x00)
3:  deviceId | channel   (bits 2-6: deviceId, bits 0-1: channel)
4:  bitOrder | dataMode  (bit 0: bitOrder, bits 1-2: dataMode (0-3))
                          see mode column in table above
5:  clockSpeed           (bits 0 - 6)
6:  clockSpeed           (bits 7 - 14)
7:  clockSpeed           (bits 15 - 21)
8:  clockSpeed           (bits 22 - 28)
9:  clockSpeed           (bits 29 - 32)
10: csPin                (0-127) [optional]
11: csOptions [used only if csPin is set]:
      csActive (bit 1) default = 0 (0 = LOW -> HIGH, 1 = HIGH -> LOW)
      csToggle (bit 2) default = 0 (0 = toggle after last byte,
                          1 = toggle between bytes)
12: END_SYSEX
```

### SPI Transfer

**Read a bybe for each byte written.**

Send an array of data to transfer to the SPI slave device, expecting a byte
returned for each byte written. Respond with a `SPI_REPLY` message.

**sequence**

Use the sequence byte to specify if this is a one time transfer or if it is part
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

### SPI Read only

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

### SPI Write only

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
N: END_SYSEX
```

### Reply

A byte array of data received from the SPI slave device in response to the
transfer. Used for both `SPI_TRANSFER_REQUEST` AND `SPI_READ_REQUEST`.

**sequence**

The sequence byte specifies which part of a sequence the reply corresponds to.
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
N: END_SYSEX
```

### Begin

Call once to initialize SPI hardware. Can optionally be set internally upon
receiving first `SPI_CONFIG_DEVICE` message.

```
0:  START_SYSEX
1:  SPI_DATA             (0x68)
2:  SPI_BEGIN            (0x05)
3:  channel              (HW supports multiple SPI ports. range = 0-3, default = 0)
4:  END_SYSEX
```

### End

Call once to release SPI hardware send before quitting a Firmata client
application.

```
0:  START_SYSEX
1:  SPI_DATA             (0x68)
2:  SPI_END              (0x06)
3:  channel              (HW supports multiple SPI ports. range = 0-3, default = 0)
4:  END_SYSEX
```

# Addendum

The following commands are more tightly coupled to the Arduino SPI library
implemenation and could be considered to add finer control at the expense
of having to send and receive a lot more messages. There are a couple of
different ways to structure it, one that requres the use of SPI_CONFIG_DEVICE
(**Option A**) and one that does not (**Option B**).

## Option A: Config required

### SPI_BEGIN_TRANSACTION

Wraps `SPI.beginTransaction(settings)` and uses the `SPISettings` object that
would be created from the `SPI_CONFIG_DEVICE` command (that is stored and looked
up by the deviceId).

In Option A, `SPI_CONFIG_DEVICE` needs to be sent before sending
`SPI_BEGIN_TRANSACTION`.

```
0:  START_SYSEX
1:  SPI_DATA              (0x68)
2:  SPI_BEGIN_TRANSACTION (0x07)
3:  deviceId | channel    (bits 2-6: deviceId, bits 0-1: channel)
4:  END_SYSEX
```

### SPI_SINGLE_BYTE_TRANSFER

Write, read or read and write a single byte at a time. CS pin (or any other pins
if required) must be toggled separately using a `DIGITAL_MESSAGE`. This provides
the most flexibility but requires a lot more traffic on the transport since
separate messages must be sent to start the transaction, clear the CS pin, send
the data, set the CS pin and end the transaction. That's 5 separate messages
that can be accomplished in a single message using `SPI_TRANSFER_REQUEST`,
`SPI_READ_REQUEST` or `SPI_WRITE`.

The user can continue sending as many transfer messages as neceesary, one byte
at a time, handling each return byte individually (if applicable).

SPI_REPLY will have the sequence byte set to 0 and numBytes set to 1.

```
0:  START_SYSEX
1:  SPI_DATA                 (0x68)
2:  SPI_SINGLE_BYTE_TRANSFER (0x08)
3:  deviceId | channel       (bits 2-6: deviceId, bits 0-1: channel)
4:  transferOptions          (0 = read & write, 1 = read only, 2 = write only)
                             if read only, a value of 0x00 will be written.
                             if read & write or read only, respond with SPI_REPLY
5:  data 0                   (bits 0-6)
6:  data 1                   (bit 7) // or inverted depending on bitOrder setting
7:  END_SYSEX
```

### SPI_END_TRANSACTION

Wraps SPI.endTransaction().

```
0:  START_SYSEX
1:  SPI_DATA              (0x68)
2:  SPI_END_TRANSACTION   (0x09)
3:  channel               (bits 0-1: channel)
4:  END_SYSEX
```


## Option B: No config required

### SPI_BEGIN_TRANSACTION

Wraps `SPI.beginTransaction(SPISettings(clockSpeed, dataOrder, dataMode))`.

```
0:  START_SYSEX
1:  SPI_DATA              (0x68)
2:  SPI_BEGIN_TRANSACTION (0x07)
3:  channel               (HW supports multiple SPI ports. range = 0-3, default = 0)
4:  bitOrder | dataMode   (bit 0: bitOrder, bits 1-2: dataMode (0-3))
5:  clockSpeed            (bits 0 - 6)
6:  clockSpeed            (bits 7 - 14)
7:  clockSpeed            (bits 15 - 21)
8:  clockSpeed            (bits 22 - 28)
9:  clockSpeed            (bits 29 - 32)
10: END_SYSEX
```

### SPI_SINGLE_BYTE_TRANSFER

Write, read or read and write a single byte at a time. CS pin (if required)
must be toggled using DIGITAL_MESSAGE. This provides the most flexibility but
requires a lot more traffic on the transport since separate messages must be
sent to start the transaction, clear the CS pin, send the data, set the
CS pin and end the transaction. That's 5 separate messages that can be
accomplished in a single message using SPI_TRANSFER_REQUEST, SPI_READ_REQUEST
or SPI_WRITE.

The user can continue sending as many transfer messages as neceesary, one byte
at a time, handling each return byte (if applicable).

SPI_REPLY will have the sequence byte set to 0 and numBytes set to 1.

```
0:  START_SYSEX
1:  SPI_DATA                 (0x68)
2:  SPI_SINGLE_BYTE_TRANSFER (0x08)
3:  channel                  (HW supports multiple SPI ports. range = 0-3, default = 0)
4:  transferOptions          (0 = read & write, 1 = read only, 2 = write only)
                             if read only, a value of 0x00 will be written.
                             if read & write or read only, respond with SPI_REPLY
5:  data 0                   (bits 0-6)
6:  data 1                   (bit 7) // or inverted depending on bitOrder setting
7: END_SYSEX
```

### SPI_END_TRANSACTION

Wraps SPI.endTransaction().

```
0:  START_SYSEX
1:  SPI_DATA              (0x68)
2:  SPI_END_TRANSACTION   (0x09)
3:  channel               (HW supports multiple SPI ports. range = 0-3, default = 0)
4:  END_SYSEX
```
