SPI
===

A proposal for a SPI protocol for Firmata.

Each SPI device is initialized via a config message. This message sets the bit order
data mode clock speed and optional CS pin for the device. Arrays of bytes can be
transferred to and received from the slave device. By default the CS pin pulled low
at the beginning of a transfer and released after the last byte sent. The user can
change the active CS pin state or even opt to handle the CS pin manually.

Additional options can be set for less common cases such as changing the word length
or forcing the CS pin to toggle between every word sent.

There are 3 ways to send and receive data from the SPI slave device:

1. Transfer - for each word sent a word is received
2. Write - only write data (ignore any data retuned by the slave device)
3. Read - only read data, writing 0 for each byte to be read


## Protocol details

### Begin

```
// called once to initialize SPI hardware
// can optionally be set internally on first SPI_CONFIG_DEVICE message
0:  START_SYSEX
1:  SPI_DATA
2:  SPI_BEGIN (bits 0 - 4) | channel (bits 5 - 6) if HW supports multiple SPI ports
3:  END_SYSEX
```

### Config

Data modes:

| mode    | clock polarity (CPOL) | clock phase (CPHA) |
| --------|-----------------------|--------------------|
| 0       | 0                     | 0                  |
| 1       | 0                     | 1                  |
| 2       | 1                     | 0                  |
| 3       | 1                     | 1                  |


```
// sent once for each SPI device
0:  START_SYSEX
1:  SPI_DATA
2:  SPI_CONFIG_DEVICE (bits 0 - 4) | channel (bits 5 - 6) if HW supports multiple SPI ports
3:  bitOrder (bit 0) | dataMode (bits 1 - 2) see table above | deviceId (bits 3 - 6)
4:  csPin (0-127)  // use csPin to identify individual devices
5:  csOptions:
      handleCSPin (bit 0) default = 1
      csActive    (bit 1) default = 0
      csToggle    (bit 2) default = 0
6:  wordSize   (bits 0 - 3) add 1 to value (range: 1 - 16)
7:  clockSpeed (bits 0 - 6)
8:  clockSpeed (bits 7 - 14)
9:  clockSpeed (bits 15 - 21)
10: clockSpeed (bits 22 - 28)
11: clockSpeed (bits 29 - 32)
12: END_SYSEX
```

### Transfer

```
// send an array of data to transfer to the SPI slave device
0:  START_SYSEX
1:  SPI_DATA
2:  SPI_TRANSFER_REQUEST
3:  csPin (0 - 127)  (or deviceId: 0 - MAX_SPI_DEVICES - 1)
4:  numWords // needed because we'll reuse a single array for all transfers
5:  data 0 (bits 0 - 6)
6:  data 1 (bit 7)  or bits 7 - 14 if wordSize > 8 && <= 14
7:  data 2 (bits 15 - 16 if wordSize > 14)
    order of data bits may be flipped depending on bitOrder setting
... up to numWords * ((byte) wordSize / 7)
N:  END_SYSEX
```

### Read

```
// use if device has a ready signal or requires a delay after
// a write command
// writes zeros for each byte to be read
0:  START_SYSEX
1:  SPI_DATA
2:  SPI_READ_REQUEST
3:  csPin (0 - 127) (or deviceId: 0 - MAX_SPI_DEVICES - 1)
4:  numWords
5:  END_SYSEX
```

### Reply

```
// a byte array of data received from the SPI slave device in response to the transfer
// used for both SPI_TRANSFER_REQUEST AND SPI_READ_REQUEST
0:  START_SYSEX
1:  SPI_DATA
2:  SPI_REPLY
3:  csPin (0 - 127)  (or deviceId: 0 - MAX_SPI_DEVICES - 1)
4:  numWords
5:  data 0 (bits 0 - 6)
6:  data 1 (bit 7)  or bits 7 - 14 if wordSize > 8 && <= 14
7:  data 2 (bits 15 - 16 if wordSize > 14)
... up to numWords * ((byte) wordSize / 7)
N: END_SYSEX
```

### Write

```
// use when a transfer does not require a response
0:  START_SYSEX
1:  SPI_DATA
2:  SPI_WRITE
3:  csPin (0 - 127) (or deviceId: 0 - MAX_SPI_DEVICES - 1)
4:  numWords
5:  data 0 (bits 0 - 6)
6:  data 1 (bit 7)  or bits 7 - 14 if wordSize > 8 && <= 14
7:  data 2 (bits 15 - 16 if wordSize > 14)
... up to numWords * ((byte) wordSize / 7)
N: END_SYSEX
```

### End

```
// called once to release SPI hardware
// send before quitting a Firmata client application
0:  START_SYSEX
1:  SPI_DATA
2:  SPI_END (bits 0 - 4) | channel (bits 5 - 6) if HW supports multiple SPI ports
3:  END_SYSEX
```