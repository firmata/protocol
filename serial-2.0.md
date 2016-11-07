#Serial 2.0

Enables control of up to 4 software and 4 hardware (UART) serial ports. Multiple ports can be
used simultaneously (depending on restrictions of a given microcontroller board's capabilities).

Example files:
 * Version 2.0 of the Serial feature has not yet been implemented.

Added in version 2.6.0

## Constants

### Port IDs

Use these constants to identify the hardware or software serial port to address for each command.

```
HW_SERIAL0 = 0x00 (for using Serial when another transport is used for the Firmata Stream)
HW_SERIAL1 = 0x01
HW_SERIAL2 = 0x02
HW_SERIAL3 = 0x03

SW_SERIAL0 = 0x08
SW_SERIAL1 = 0x09
SW_SERIAL2 = 0x0A
SW_SERIAL3 = 0x0B
```

### Serial pin capability response

Use these constants to identify the pin type in a [capability query response](https://github.com/firmata/protocol/blob/master/protocol.md#capability-query).

```
// Where the pin mode = "Serial" and the pin resolution = one of the following:
RES_RX0 = 0x00
RES_TX0 = 0x01
RES_RX1 = 0x02
RES_TX1 = 0x03
RES_RX2 = 0x04
RES_TX2 = 0x05
RES_RX3 = 0x06
RES_TX3 = 0x07
// extensible up to 8 HW ports

```

### Serial pin mode

```
PIN_MODE_SERIAL = 0x0A
```

## Commands

### Serial Config

Configures the specified hardware or software serial port. RX and TX pins are optional and should
only be specified if the platform requires them to be set.

```
0  START_SYSEX      (0xF0)
1  SERIAL_MESSAGE   (0x64)  // command byte
2  SERIAL_CONFIG    (0x00)
3  port             (HW_SERIALn OR SW_SERIALn)
4  baud             (bits 0 - 6)
5  baud             (bits 7 - 13)
6  baud             (bits 14 - 20) // need to send 3 bytes for baud even if value is < 14 bits
7  rxPin            (0-127) [optional] // only set if platform requires RX pin number
8  txPin            (0-127) [optional] // only set if platform requires TX pin number
7|9 END_SYSEX       (0xF7)
```

### Serial Write

Firmata client -> Board

Receive serial data from Firmata client, reassemble and write for each byte received.

```
0  START_SYSEX      (0xF0)
1  SERIAL_MESSAGE   (0x64)
2  SERIAL_WRITE     (0x01)
3  port             (HW_SERIALn OR SW_SERIALn)
4  data 0           (LSB)
5  data 0           (MSB)
6  data 1           (LSB)
7  data 1           (MSB)
...                 // up to max buffer - 5
n  END_SYSEX        (0xF7)
```

### Serial Read

Board -> Firmata client

Read contents of serial buffer and send to Firmata client (send with `SERIAL_REPLY`).

`maxBytesToRead` optionally specifies how many bytes to read for each iteration. Set to 0 (or do not
define) to read all available bytes. If there are less bytes in the buffer than the number of bytes
specified by `maxBytesToRead` then the lesser number of bytes will be returned.

```
0  START_SYSEX        (0xF0)
1  SERIAL_MESSAGE     (0x64)
2  SERIAL_READ        (0x02)
3  port               (HW_SERIALn OR SW_SERIALn)
4  SERIAL_READ_MODE   (0x00) // 0x00 => read continuously, 0x01 => stop reading
5  maxBytesToRead     (lsb) [optional]
6  maxBytesToRead     (msb) [optional]
5|7 END_SYSEX         (0xF7)
```

### Serial Reply

Board -> Firmata client

Sent in response to a SERIAL_READ event or on each iteration of the reporting loop if `SERIAL_READ_CONTINUOUSLY` is set.

```
0  START_SYSEX        (0xF0)
1  SERIAL_MESSAGE     (0x64)
2  SERIAL_REPLY       (0x03)
3  port               (HW_SERIALn OR SW_SERIALn)
4  data 0             (LSB)
5  data 0             (MSB)
6  data 1             (LSB)
7  data 1             (MSB)
...                   // up to max buffer - 5
n  END_SYSEX          (0xF7)
```

### Serial Close

Close the serial port. If you close a port, you will need to send a `SERIAL_CONFIG` message to
reopen it.

```
0  START_SYSEX        (0xF0)
1  SERIAL_MESSAGE     (0x64)
2  SERIAL_CLOSE       (0x04)
3  port               (HW_SERIALn OR SW_SERIALn)
4  END_SYSEX          (0xF7)
```

### Serial Flush

Flush the serial port. The exact behavior of flush depends on the underlying platform. For example,
with Arduino, calling `flush` on a HW serial port will drain the TX output buffer, calling `flush`
on a SW serial port will reset the RX buffer to the beginning, abandoning any data in the buffer.
Other platforms may define `flush` differently as well so see the documentation of flush for the
platform you are working with to understand exactly how it functions.

Generally `flush` is rarely needed so this functionality is primarily provided for advanced use
cases.

```
0  START_SYSEX        (0xF0)
1  SERIAL_MESSAGE     (0x64)
2  SERIAL_FLUSH       (0x05)
3  port               (HW_SERIALn OR SW_SERIALn)
4  END_SYSEX          (0xF7)
```

### Serial Listen

Enable switching serial ports. Necessary for Arduino SoftwareSerial but may not be applicable to
other platforms.

```
0  START_SYSEX        (0xF0)
1  SERIAL_MESSAGE     (0x64)
2  SERIAL_LISTEN      (0x06)
3  port               (HW_SERIALn OR SW_SERIALn)
4  END_SYSEX          (0xF7)
```

### Serial Update Baud

Update the baud rate on a configured serial port.

```
0  START_SYSEX        (0xF0)
1  SERIAL_MESSAGE     (0x64)
2  SERIAL_UPDATE_BAUD (0x07)
3  port               (HW_SERIALn OR SW_SERIALn)
4  baud               (bits 0 - 6)
5  baud               (bits 7 - 13)
6  baud               (bits 14 - 20) // need to send 3 bytes for baud even if value is < 14 bits
7  END_SYSEX          (0xF7)
```
