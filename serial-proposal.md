#Serial proposal

A draft proposal for adding Software and Hardware Serial support to Firmata.

Enables control of up to 4 software and 4 hardware (UART) serial ports. Multiple ports can be
used simultaneously (depending on restrictions of a given microcontroller board's capabilities).

Sample implementation code for Arduino is available [here](https://github.com/firmata/arduino/pull/205).

## Constants

### Port IDs

Use these constants to identify the hardware or software serial port to address for each command.

```
HW_SERIAL0 = 0x00 (for using Serial when another transport is used for the Firmata Stream)
HW_SERIAL1 = 0x01
HW_SERIAL2 = 0x02
HW_SERIAL3 = 0x03
// extensible up to 8 HW serial ports

SW_SERIAL0 = 0x08
SW_SERIAL1 = 0x09
SW_SERIAL2 = 0x0A
SW_SERIAL3 = 0x0B
// extensible up to 8 SW serial ports
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
MODE_SERIAL = 0x0A
```

## Commands

### Serial Config

Configures the specified hardware or software serial port. The format options (bytes 6 and 7) are
not supported by all Serial ports. See the documentation for your platform (Arduino, etc) regarding
which format (or "config") options are available per serial port per board.

```
0  START_SYSEX      (0xF0)
1  SERIAL_DATA      (0x60)  // command byte
2  SERIAL_CONFIG    (0x10)  // OR with port (0x11 = SERIAL_CONFIG | HW_SERIAL1)
3  baud             (bits 0 - 6)
4  baud             (bits 7 - 13)
5  baud             (bits 14 - 20) // need to send 3 bytes for baud even if value is < 14 bits
6  options          (bits 0 - 6)
7  options          (bits 7 - 13)
                    Set to 0 to ignore, otherwise set data, parity and stop bits as follows:
                    {13-9: reserved} +
                    {8-5: data bits, B1000 = 8, etc} +
                    {4-2: parity, B000 = None, B001 = Even, B010 = Odd, B011 = Mark, B100 = Space} +
                    {1-0: stop bits, B00 = 1, B01 = 1.5, B10 = 2}
                    // Restrictions will apply per capabilities of individual serial ports
8  rxPin            (0-127) [softserial only] // restrictions apply per board
9  txPin            (0-127) [softserial only] // restrictions apply per board
8|10 END_SYSEX      (0xF7)
```

### Serial Write

Firmata client -> Board

Receive serial data from Firmata client, reassemble and write for each byte received.

```
0  START_SYSEX      (0xF0)
1  SERIAL_DATA      (0x60)
2  SERIAL_WRITE     (0x20) // OR with port (0x21 = SERIAL_WRITE | HW_SERIAL1)
3  data 0           (LSB)
4  data 0           (MSB)
5  data 1           (LSB)
6  data 1           (MSB)
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
1  SERIAL_DATA        (0x60)
2  SERIAL_READ        (0x30) // OR with port (0x31 = SERIAL_READ | HW_SERIAL1)
3  SERIAL_READ_MODE   (0x00) // 0x00 => read continuously, 0x01 => stop reading
4  maxBytesToRead     (lsb) [optional]
5  maxBytesToRead     (msb) [optional]
4|6 END_SYSEX         (0xF7)
```

### Serial Reply

Board -> Firmata client

Sent in response to a SERIAL_READ event or on each iteration of the reporting loop if `SERIAL_READ_CONTINUOUSLY` is set.

```
0  START_SYSEX        (0xF0)
1  SERIAL_DATA        (0x60)
2  SERIAL_REPLY       (0x40) // OR with port (0x41 = SERIAL_REPLY | HW_SERIAL1)
3  data 0             (LSB)
4  data 0             (MSB)
3  data 1             (LSB)
4  data 1             (MSB)
...                   // up to max buffer - 5
n  END_SYSEX          (0xF7)
```

### Serial Close

Close the serial port. If you close a port, you will need to send a `SERIAL_CONFIG` message to
reopen it.

```
0  START_SYSEX        (0xF0)
1  SERIAL_DATA        (0x60)
2  SERIAL_CLOSE       (0x50) // OR with port (0x51 = SERIAL_CLOSE | HW_SERIAL1)
3  END_SYSEX          (0xF7)
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
1  SERIAL_DATA        (0x60)
2  SERIAL_FLUSH       (0x60) // OR with port (0x61 = SERIAL_FLUSH | HW_SERIAL1)
3  END_SYSEX          (0xF7)
```

### Serial Listen

Enable switching serial ports.

*For Arduino, `SERIAL_LISTEN` is only used for SoftwareSerial ports.*

```
0  START_SYSEX        (0xF0)
1  SERIAL_DATA        (0x60)
2  SERIAL_LISTEN      (0x70) // OR with port to switch to (0x79 = switch to SW_SERIAL1)
3  END_SYSEX          (0xF7)
```
