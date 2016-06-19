# I2C

Enables communication with I2C devices. Currently only supports one I2C port per board.
实现与I2C器件进行通信。目前只支持每板1个I2C端口。

Added in version 2.1.0.

### I2C read/write request
###I2C 读/写要求

Updated in Firmata 2.5.1 to enable setting auto-restart by setting bit 6 of byte 3.
在Firmata2.5.1更新，从而支持通过设置3个字节的第6位设置自动重启。
```
0  START_SYSEX (0xF0)
1  I2C_REQUEST (0x76)
2  slave address (LSB)
3  slave address (MSB) + read/write and address mode bits
     {bit 7: always 0}
     {bit 6: auto restart transmission, 0 = stop (default), 1 = restart}
     {bit 5: address mode, 1 = 10-bit mode}
     {bits 4-3: read/write, 00 = write, 01 = read once, 10 = read continuously, 11 = stop reading}
     {bits 2-0: slave address MSB in 10-bit mode, not used in 7-bit mode}
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
关于读/写模式（上图）的说明。该```读continuously```模式指示
固件应该连续地由指定的速率读取设备
[采样间隔（https://github.com/firmata/protocol/blob/master/protocol.md）。固件实现应支持读连续模式
同时为几个I2C器件。发送```停止reading```命令
结束该特定装置读出连续模式。

*auto-restart (byte 3, bit 6) is needed by some devices such as the MMA8452Q accelerometer and the MPL3115As altimeter*
*自动重启（字节3，第6位）则是有些设备需要这样的MMA8452Q加速计和高度计MPL3115As*

### I2C reply

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

### I2C config

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
可选的```Delay```是为需要的时候之间的延迟I2C器件
寄存器被写入，并在该寄存器中的数据可以被读出。
