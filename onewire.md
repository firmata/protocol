OneWire
===

The idea is to configure Arduino Pins as OneWire Busmaster. The may be more than one pin configured for OneWire and there may be more than one device connected to such a pin.
这个想法是配置Arduino Pins作为OneWire BUSMASTER。可能有多个为配置OneWire的pin和可能存在连接到这样一个pins的多个设备。

Each one-wire-device has a unique identifier which is 8 bytes long and comes factory-programmed into the the device. To scan all devices connected to a pin configured for onewire a SEARCH-request message is sent. The response contains all addresses of devices found. Having the address of a device OneWire-command-messages may be sent to this device.
每线设备都有一个唯一的标识符，是8个字节长，出厂时编程到设备中。要扫描连接到配置为onewire引脚发送一个搜索请求消息的所有设备。响应包含找到的设备的所有地址。具有设备OneWire-命令消息的地址可以被发送到该设备。

The actual commands executed on the OneWire-bus are 'reset', 'skip', 'select', 'read', 'delay' and 'write' All these commands may be executed with a single OneWire-command-message. The subcommand-byte contains these commands bit-encoded. The data required to execute each bus-command must only be included in the message when the corresponding bit is set.
在OneWire总线上执行的'复位'，'跳'，'选择'的实际命令，“读”，“延迟”和“写”所有这些命令可以与单个OneWire命令消息来执行。子命令字节包含这些命令位编码。执行每个总线命令所需的数据必须只被包括在该消息中，当相应的位被置位。

The order of execution of bus commands is: 'reset'->'skip'->'select'->'write'->'read'->'delay' (remember: each of these steps is optional. Also some combinations don't make sense and in fact are mutual exclusive in terms of OneWire bus protocol, so you cannot run a 'skip' followed by a 'select') The delay is useful for OneWire-commands included into taskdata (see [Firmata-scheduler proposal](https://github.com/firmata/protocol/blob/master/scheduler.md)).
总线的执行命令的顺序是：'重置' - >'跳过' - >'选择' - >'写' - >'读' - >'延迟'（记住：每个步骤是可选的另外一些组合没有意义，实际上是相互独家OneWire总线协议的条款，所以不能运行'跳过'后'选择'）的延迟是有用OneWire的命令纳入taskdata（见[Firmata调度器建议]（https://github.com/firmata/protocol/blob/master/scheduler.md））。

Some OneWire-devices require some time to carry out e.g. a a/d-conversion after receiving the appropriate command. Including a delay into a OneWire-message saves some bytes in the taskdata (in comparism to the inclusion of a 'delay_task' scheduler message). OneWire Read- and ReadReply messages are correlated using a correlationid (16bits). The reply contains the correlationid-value that was sent with the original request.
一些OneWire的设备需要一定的时间来进行如接收到适当的命令后一个A / D转换。包括延迟到OneWire消息保存在taskdata一些字节（comparism对列入“delay_task”调度信息）。 OneWire读 - 和ReadReply消息都将使用的correlationID（16位）相关。该答复包含用原始请求发送的correlationID价值。

Added in version 2.4.0.


### Example files: 
 * OneWire is include by default in [ConfigurableFirmata.ino](https://github.com/firmata/ConfigurableFirmata/blob/master/examples/ConfigurableFirmata/ConfigurableFirmata.ino). 
 * [Example implementation](https://github.com/firmata/ConfigurableFirmata/blob/master/src/OneWireFirmata.cpp) as a configurable Firmata feature class.


### Compatible host implementations
* [ConfigurableFirmata](https://github.com/firmata/ConfigurableFirmata)


### Compatible client librairies
* [perl-firmata](https://github.com/ntruchsess/perl-firmata)
* [node-firmata](https://github.com/jgautier/firmata/blob/master/lib/firmata.js)


### Protocol details

OneWire SEARCH request
```
0  START_SYSEX      (0xF0)
1  OneWire Command  (0x73)
2  search command   (0x40|0x44) 0x40 normal search for all devices on the bus
                                0x44 SEARCH_ALARMS request to find only those
                                devices that are in alarmed state.
3  pin              (0-127)
4  END_SYSEX        (0xF7)
```

OneWire SEARCH reply
```
0  START_SYSEX      (0xF0)
1  OneWire Command  (0x73)
2  search reply command (0x42|0x45) 0x42 normal search reply
                                    0x45 reply to a SEARCH_ALARMS request
3  pin              (0-127)
4  bit 0-6   [optional] address bytes encoded using 8 times 7 bit for 7 bytes of 8 bit
5  bit 7-13  [optional] 1.address[0] = byte[0]    + byte[1]<<7 & 0x7F
6  bit 14-20 [optional] 1.address[1] = byte[1]>>1 + byte[2]<<6 & 0x7F
7  ....                 ...
11 bit 49-55            1.address[6] = byte[6]>>6 + byte[7]<<1 & 0x7F
12 bit 56-63            1.address[7] = byte[8]    + byte[9]<<7 & 0x7F
13 bit 64-69            2.address[0] = byte[9]>>1 + byte[10]<<6 &0x7F
n  ... as many bytes as needed (don't exceed MAX_DATA_BYTES though)
n+1  END_SYSEX      (0xF7)
```

OneWire CONFIG request
```
0  START_SYSEX      (0xF0)
1  OneWire Command  (0x73)
2  config command   (0x41)
3  pin              (0-127)
4  power            (0x00|0x01) 0x00 = leave pin on state high after write to support
                                parasitic power
                                0x01 = don't leave pin on high after write
5  END_SYSEX (0xF7)
```

OneWire COMMAND request
```
0  START_SYSEX      (0xF0)
1  OneWire Command  (0x73)
2  command bits     (0x00-0x2F) bit 0 = reset, bit 1 = skip, bit 2 = select,
                                bit 3 = read, bit 4 = delay, bit 5 = write
3  pin              (0-127)
4  bit 0-6   [optional] data bytes encoded using 8 times 7 bit for 7 bytes of 8 bit
5  bit 7-13  [optional] data[0] = byte[0]   + byte[1]<<7 & 0x7F
6  bit 14-20 [optional] data[1] = byte[1]>1 + byte[2]<<6 & 0x7F
7  ....                 data[2] = byte = byte[2]>2 + byte[3]<<5 & 0x7F ...
n  ... as many bytes as needed (don't exceed MAX_DATA_BYTES though)
n+1  END_SYSEX      (0xF7)

// data bytes within OneWire Request Command message
0  address[0]                    [optional, if bit 2 set]
1  address[1]                              "
2  address[2]                              "
3  address[3]                              "
4  address[4]                              "
5  address[5]                              "
6  address[6]                              "
7  address[7]                              "
8  number of bytes to read (LSB) [optional, if bit 3 set]
9  number of bytes to read (MSB)           "
10 request correlationid byte 0            "
11 request correlationid byte 1            "
10 delay in ms      (bits 0-7)   [optional, if bit 4 set]
11 delay in ms      (bits 8-15)            "
12 delay in ms      (bits 16-23)           "
13 delay in ms      (bits 24-31)           "
14 data to write    (bits 0-7)   [optional, if bit 5 set]
15 data to write    (bits 8-15)            "
16 data to write    (bits 16-23)           "
n  ... as many bytes as needed (don't exceed MAX_DATA_BYTES though)
```

OneWire READ reply
```
0  START_SYSEX          (0xF0)
1  OneWire Command      (0x73)
2  read reply command   (0x43)
3  pin                  (0-127)
4  bit 0-6   [optional] data bytes encoded using 8 times 7 bit for 7 bytes of 8 bit
5  bit 7-13  [optional] correlationid[0] = byte[0]   + byte[1]<<7 & 0x7F
6  bit 14-20 [optional] correlationid[1] = byte[1]>1 + byte[2]<<6 & 0x7F
7  bit 21-27 [optional] data[0] = byte[2]>2 + byte[3]<<5 & 0x7F
8  ....                 data[1] = byte[3]>3 + byte[4]<<4 & 0x7F
n  ... as many bytes as needed (don't exceed MAX_DATA_BYTES though)
n+1  END_SYSEX          (0xF7)
```
