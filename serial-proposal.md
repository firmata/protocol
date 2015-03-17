Serial proposal
===

A draft proposal for adding Software and Hardware Serial support to Firmata.

A discussion and sample implementation code for Arduino is available here: https://github.com/firmata/arduino/issues/97

```
// SW_SERIAL  = 0x00
// HW_SERIAL1 = 0x01
// HW_SERIAL2 = 0x02
// HW_SERIAL3 = 0x03

// config: creates new software Serial or hardware serial instance and calls begin(baud)
0  START_SYSEX      (0xF0)
1  SERIAL_DATA      (0x60)  // command byte
2  CONFIG           (0x11)  // subcommand byte (SERIAL_CONFIG | HW_SERIAL1)
3  baud             (bits 0 - 6)
4  baud             (bits 7 - 13)
5  baud             (bits 14 - 20) // need to send 3 bytes for baud even if value is < 14 bits
6  bufferLen        (bits 0 - 6)
7  bufferLen        (bits 7 - 13)
8  txPin            (0-127) [softserial only] // restrictions apply per board
9  rxPin            (0-127) [softserial only] // restrictions apply per board
10 END_SYSEX        (0XF7)
```

```
// Board -> Firmata client
// read contents of serial buffer and send to Firmata client
0  START_SYSEX      (0xF0)
1  SERIAL_DATA      (0x60)
2  SERIAL_TX        (0x21) // subcommand byte (SERIAL_TX | HW_SERIAL1)
3  data 0           (LSB)  // each byte is split
4  data 0           (MSB)
5  data 1           (LSB)
6  data 1           (MSB)
...                 // up to max buffer - 5
n  END_SYSEX        (0XF7)
```

```
// Firmata client -> Board
// receive serial data from Firmata client, reassemble and
// write for each byte received
0  START_SYSEX      (0xF0)
1  SERIAL_DATA      (0x60)
2  SERIAL_RX        (0x31) // subcommand byte (SERIAL_RX | HW_SERIAL1)
3  data 0           (LSB)
4  data 0           (MSB)
5  data 1           (LSB)
6  data 1           (MSB)
...                 // up to max buffer - 5
n  END_SYSEX        (0XF7)
```

```
// [optional]
0  START_SYSEX      (0xF0)
1  SERIAL_DATA      (0x60)
2  SERIAL_CLOSE     (0x41) // subcommand byte (SERIAL_CLOSE | HW_SERIAL1)
3  END_SYSEX        (0XF7)
```

```
// [optional]
0  START_SYSEX      (0xF0)
1  SERIAL_DATA      (0x60)
2  SERIAL_FLUSH     (0x51) // subcommand byte (SERIAL_FLUSH | HW_SERIAL1)
3  END_SYSEX        (0XF7)
```
