shift in/out proposal
===

There are a few different ways to approach shift in/out support. It's complicated
since different hardware handles shift in/out in different ways. For example,
not all hardware requires a latch pin and those that use some sort of a latch
don't always use it the same way.

有几种不同的方式来处理输入/输出支持转变。 情况很复杂
因为不同的硬件处理输入/输出以不同的方式转变。 例如，
不是所有的硬件要求的闩锁引脚和那些使用的锁存器的某种
不要总是用同样的方式。


There has also been some discussion around supporting fractional byte devices. The proposals below do not include such functionality. I'm not sure how popular
such devices are. If someone has a proposal that includes support for shifting 
fractional bytes, please submit a pull request to add the proposal to this document.

此外，也有周边配套小数字节的设备进行一些讨论。提案下面不包括这样的功能。我不知道有多受欢迎
这种装置。如果有人有包括用于移动支持的建议
分数字节，请提交pull请求的建议添加到该文件。

Propoasl A: shift in/out with no config or latch support
建议A：移入/移出，没有配置或闭锁支持
---

In this version it the user must configure the pin (input / output) separately.
If the hardware they are using requires a latch, the latch pin must be managed
separately.
在这个版本中它的用户必须配置销（输入/输出）分别。
如果他们正在使用的硬件要求闩，锁闩引脚必须被管理
分别。
```
// shift out
0  START_SYSEX
1  SHIFT_DATA          (0x75)
2  SHIFT_OUT           (0x01)
3  dataPin
4  clockPin
5  bitOrder            (MSBFIRST or LSBFIRST)
n ...                  (shift out data)
n+1 END_SYSEX

// shift in (for client application to request shift-in data from microcontroller)
0  START_SYSEX
1  SHIFT_DATA          (0x75)
2  SHIFT_IN            (0x02)
3  dataPin
4  clockPin
5  bitOrder            (MSBFIRST or LSBFIRST)
6  numBytes            (number of bytes to shift in. Default to 1)
7 END_SYSEX

// shift in reply (for sending shift-in data to client application)
0  START_SYSEX
1  SHIFT_DATA          (0x75)
2  SHIFT_IN_REPLY      (0x03)
3  dataPin             (so you know which data pin the reply corresponds to)
n ...                  (shift in data)
n+1 END_SYSEX
```


Proposal B: shift in/out with config and latch support
建议B：移/与配置和锁存支持
---

The advantages with this version over the one above is that pin modes are handled by the implementation (in the other version you have to handle them manually). You also send fewer bytes when shifting out or in data (since only have to specify clock, bitOrder and optional latch pin once when sending the config). The disadvantage is that memory must be allocated to store pin information.
与此版本在上面的一个优点是，销模式通过实现（你必须手动处理它们的其他版本）来处理。换挡时，或在数据（因为只有发送配置时指定一次时钟，bitOrder和可选的锁销）也发送更少的字节。的缺点是，存储器必须被分配以存储销信息。

Another advantage with this version is that you can rely on the firmware to do
more of the work. For example you can shift in multiple bytes at a time and send them to the client in a single packet rather than a single byte at at time (if
your hardware requires a latch/load pin).、
与此版本的另一个好处是，你可以依靠固件做
更多的工作。例如可以在一个时间中多个字节移位，并将它们在一个单一的数据包在时刻发送到客户端，而不是一个单一的字节（如果
硬件需要一个锁存/负载引脚）。

This version uses a SHIFT_CONFIG command to set the clock pin, bitOrder and optional latchPin (or load for some shift-in hardware). There are a few different shift types / latch configurations:
此版本使用SHIFT_CONFIG命令来设置时钟引脚，bitOrder和可选latchPin（或负载一些转变的硬件）。有几种不同类型的转变/锁存器配置：
```
// shift types (specified in bits 3:5 of byte 2)
SHIFT_OUT                     // shift out with no latch
SHIFT_IN                      // shift out with no latch/load
LATCH_L_SHIFT_OUT             // set latch low then shift out
LATCH_L_SHIFT_OUT_LATCH_H     // set latch low, shift out, then set latch high
SHIFT_OUT_LATCH_H             // shift out then set latch high
TOGGLE_LOAD_SHIFT_IN          // toggle load pin low, then high and shift in
```

The protocol is as follows:
```
// shift config (send when instantiating a new shift-based hardware module)
0  START_SYSEX
1  SHIFT_DATA    (0x75)
2  SHIFT_CONFIG  (bits 0:2 shift command, bits 3:5 shift type, bit 6 unused)
3  dataPin
4  clockPin
5  bitOrder
6  latchPin      [optional]
7 END_SYSEX

// shift out
0  START_SYSEX
1  SHIFT_DATA    (0x75)
2  SHIFT_OUT     (bits 0:2 shift command, bits 3:5 shift type, bit 6 unused)
3  dataPin
n  ...           (shift out data)
n+1 END_SYSEX

// shift in
0  START_SYSEX
1  SHIFT_DATA    (0x75)
2  SHIFT_IN      (bits 0:2 shift command, bits 3:5 shift type, bit 6 unused)
3  dataPin
4  numBytes      (number of bytes to shift in. Default to 1)
5  END_SYSEX
```
