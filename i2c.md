I2C
===

Added in Firmata 2.3.

I2C read/write request
```
0  START_SYSEX (0xF0)
1  I2C_REQUEST (0x76)
2  slave address (LSB)
3  slave address (MSB) + read/write and address mode bits
   {7: always 0} + {6: reserved} + {5: address mode, 1 means 10-bit mode} +
   {4-3: read/write, 00 => write, 01 => read once, 10 => read continuously, 11 => stop reading} +
   {2-0: slave address MSB in 10-bit mode, not used in 7-bit mode}
4  data 0 (LSB)
5  data 0 (MSB)
6  data 1 (LSB)
7  data 1 (MSB)
...
n  END_SYSEX (0xF7)
```

A note about read/write modes (above). The ```read continuously``` mode indicates that
the firmware should continuously read the device at the rate specified by the
[sampling interval](https://github.com/firmata/protocol/blob/master/protocol.md). A firmware implementation should support read continuous mode
for several I2C devices simultaneously. Sending the ```stop reading``` command will
end read continuous mode for that particular device.


I2C reply
```
0  START_SYSEX (0xF0)
1  I2C_REPLY (0x77)
2  slave address (LSB)
3  slave address (MSB)
4  register (LSB)
5  register (MSB)
6  data 0 (LSB)
7  data 0 (MSB)
...
n  END_SYSEX (0XF7)
```

I2C config
```
0  START_SYSEX (0xF0)
1  I2C_CONFIG (0x78)
2  Delay in microseconds (LSB) [optional]
3  Delay in microseconds (MSB) [optional]
... user defined for special cases, etc
n  END_SYSEX (0xF7)
```

The optional ```Delay``` is for I2C devices that require a delay between when the
register is written to and the data in that register can be read.


Proposal to ammend: add auto restart support
---

Some I2C hardware needs to support an auto-restart mode (MMA8452Q accelerometer for example). There are a couple of options to add support:

1. Make bit 6 of the I2C read/write request a boolean that indicates whether auto-restart mode should be used for the particular device.

2. Add as byte 4 of the I2C config message.
