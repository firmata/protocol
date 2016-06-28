#Serial

Enables control of up to 4 software and 4 hardware (UART) serial ports. Multiple ports can be
used simultaneously (depending on restrictions of a given microcontroller board's capabilities).   

允许最多4软件和4个硬件（UART）串行端口控制。多个端口可同时使用（根据给定的微控制器板的能力的限制）。

Sample implementation code for Arduino is available   
对于Arduino的示例实现代码可用   
[here](https://github.com/firmata/arduino/blob/master/examples/StandardFirmataPlus/StandardFirmataPlus.ino).

A client implementation can be found   
一个客户实现   

[here](https://github.com/jgautier/firmata/blob/master/lib/firmata.js).

Added in version 2.5.0   
新增的2.5.0版本

## Constants

### Port IDs

Use these constants to identify the hardware or software serial port to address for each command.   
使用这些常数来识别硬件或软件的串行端口，以解决每个命令。  

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

### Serial pin capability response  Serial pin能力响应

Use these constants to identify the pin type in a [capability query response](https://github.com/firmata/protocol/blob/master/protocol.md#capability-query).
使用这些常量标识在pin类型

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
配置指定的硬件或软件串行端口。RX和TX引脚是可选的，如果平台要求它们应被设置仅指定。

```
0  START_SYSEX      (0xF0)
1  SERIAL_DATA      (0x60)  // command byte
2  SERIAL_CONFIG    (0x10)  // OR with port (0x11 = SERIAL_CONFIG | HW_SERIAL1)
3  baud             (bits 0 - 6)
4  baud             (bits 7 - 13)
5  baud             (bits 14 - 20) // need to send 3 bytes for baud even if value is < 14 bits
6  rxPin            (0-127) [optional] // only set if platform requires RX pin number
7  txPin            (0-127) [optional] // only set if platform requires TX pin number
6|8 END_SYSEX      (0xF7)
```

### Serial Write

Firmata client -> Board

Receive serial data from Firmata client, reassemble and write for each byte received.  
从Firmata客户端接收串行数据，重组和写入每个字节接收。

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
读串口缓冲区的内容并传送到客户端Firmata

`maxBytesToRead` optionally specifies how many bytes to read for each iteration. Set to 0 (or do not
define) to read all available bytes. If there are less bytes in the buffer than the number of bytes
specified by `maxBytesToRead` then the lesser number of bytes will be returned.    
`maxBytesToRead`可选择指定多少字节，为每个迭代读取。设置为0（或不定义）来读取所有可用的字节。如果在缓冲区比字节数较少字节    
通过指定`maxBytesToRead`那么小的字节数将被退回。


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
如果`SERIAL_READ_CONTINUOUSLY`设置，发送响应SERIAL_READ事件或报告的每个迭代循环。

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
关闭串行端口。 如果你关闭端口，您需要一个 `27SERIAL_CONFIG` 消息发送到重新打开它。

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
刷新串行端口。 exact的确切行为取决于基础平台。例如，用Arduino`flush`硬件上的串行端口将释放TX输出缓冲区，`flush`上的串行端口接收缓冲区将重置到开始放弃缓冲区中的任何数据。其他平台可以定义`flush`不同，所以看到flush的文件为您正在使用的平台了解它是如何运作。

Generally `flush` is rarely needed so this functionality is primarily provided for advanced use
cases.     
一般很少需要`flush`所以这个功能主要是提供先进的应用
案例。

```
0  START_SYSEX        (0xF0)
1  SERIAL_DATA        (0x60)
2  SERIAL_FLUSH       (0x60) // OR with port (0x61 = SERIAL_FLUSH | HW_SERIAL1)
3  END_SYSEX          (0xF7)
```

### Serial Listen

Enable switching serial ports. Necessary for Arduino SoftwareSerial but may not be applicable to
other platforms.   
启用切换串行端口。
必要的Arduino SoftwareSerial，但可能并不适用于
其他平台。

```
0  START_SYSEX        (0xF0)
1  SERIAL_DATA        (0x60)
2  SERIAL_LISTEN      (0x70) // OR with port to switch to (0x79 = switch to SW_SERIAL1)
3  END_SYSEX          (0xF7)
```
