#Serial proposal

A draft proposal for adding Software and Hardware Serial support to Firmata.

Sample implementation code for Arduino is available [here](https://github.com/firmata/arduino/pull/205).

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
RES_TX4 = 0x07

```

### Serial pin mode

```
MODE_SERIAL = 0x0A
```

## Commands

### Serial Config

Creates new software Serial or hardware serial instance and calls begin(baud).

```
0  START_SYSEX      (0xF0)
1  SERIAL_DATA      (0x60)  // command byte
2  CONFIG           (0x10)  // OR with port (0x11 = SERIAL_CONFIG | HW_SERIAL1)
3  baud             (bits 0 - 6)
4  baud             (bits 7 - 13)
5  baud             (bits 14 - 20) // need to send 3 bytes for baud even if value is < 14 bits
6  bytesToRead      (bits 0 - 6)   // if 0, read all available bytes per iteration of the main loop
                                   // else read the number of bytes specified per iteration
7  bytesToRead      (bits 7 - 13)
8  rxPin            (0-127) [softserial only] // restrictions apply per board
9  txPin            (0-127) [softserial only] // restrictions apply per board
10 END_SYSEX        (0xF7)
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

```
0  START_SYSEX      (0xF0)
1  SERIAL_DATA      (0x60)
2  SERIAL_READ      (0x30) // OR with port (0x31 = SERIAL_READ | HW_SERIAL1)
3  SERIAL_READ_MODE (0x00) // 0x00 => read continuously, 0x01 => stop reading
4  END_SYSEX        (0xF7)
```

### Serial Reply

Board -> Firmata client

Sent in response to a SERIAL_READ event or on each iteration of the reporting loop if `SERIAL_READ_CONTINUOUSLY` is set.

```
0  START_SYSEX      (0xF0)
1  SERIAL_DATA      (0x60)
2  SERIAL_REPLY     (0x40) // OR with port (0x41 = SERIAL_REPLY | HW_SERIAL1)
3  data 0           (LSB)
4  data 0           (MSB)
3  data 1           (LSB)
4  data 1           (MSB)
...                 // up to max buffer - 5
n  END_SYSEX        (0xF7)
```

### Serial Close

Optional

```
0  START_SYSEX      (0xF0)
1  SERIAL_DATA      (0x60)
2  SERIAL_CLOSE     (0x50) // OR with port (0x51 = SERIAL_CLOSE | HW_SERIAL1)
3  END_SYSEX        (0xF7)
```

### Serial Flush

Optional

```
0  START_SYSEX      (0xF0)
1  SERIAL_DATA      (0x60)
2  SERIAL_FLUSH     (0x60) // OR with port (0x61 = SERIAL_FLUSH | HW_SERIAL1)
3  END_SYSEX        (0xF7)
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
