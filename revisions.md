## Version 2.5.1 - December 21st, 2015

- Enable I2C auto-restart by setting [bit 6 of byte 3](https://github.com/firmata/protocol/commit/22cc239b5a527556d82707fe0c540b16ed42f0bc#diff-ab7ff1e563b418ee1e557d6ece901dc7R17) of the `I2C_REQUEST` message.
通过设置I2C_REQUEST消息的字节3的第6位I2C启用自动重启。
## Version 2.5.0 - November 7th, 2015

- Added [Serial feature](https://github.com/firmata/protocol/blob/master/serial.md) for interfacing with serial devices via hardware or software serial.
- 新增【串口功能（https://github.com/firmata/protocol/blob/master/serial.md），用于通过硬件或软件序列串行设备接口。
- Added ability to set the value of a pin by sending a single pin value instead of a port value. See 'set digital pin value' in [protocol.md](https://github.com/firmata/protocol/blob/master/protocol.md#message-types) for details.
- 新增通过发送一个引脚值，而不是一个端口值来设置引脚的价值的能力。更多详细信息参见“设定数字引脚值'[protocol.md]（https://github.com/firmata/protocol/blob/master/protocol.md#message-types）。

## Version 2.4.0 - December 2014

- Changed `report digital port` and `report analog pin` definition to return the port (digital) or pin (analog) value upon toggling to `enable`.
- 改变`报告数字port`和`报告模拟pin`定义在切换到`enable`返回端口（数字）或销（模拟）值。

- Added [OneWire feature](https://github.com/firmata/protocol/blob/master/onewire.md) to interface with 1-Wire devices.
新增[OneWire功能（https://github.com/firmata/protocol/blob/master/onewire.md）与1-Wire器件接口。

- Added [Encoder feature](https://github.com/firmata/protocol/blob/master/encoder.md) to interface with linear and rotary encoders.
- 新增[编码器功能（https://github.com/firmata/protocol/blob/master/encoder.md）线性和旋转编码器接口。

- Added [Scheduler feature](https://github.com/firmata/protocol/blob/master/scheduler.md) to enable scheduling Firmata tasks. Useful when you need to send more data than the 64 byte serial buffer can handle.
新增[调度功能（https://github.com/firmata/protocol/blob/master/scheduler.md），使调度Firmata任务。有用的，当你需要比64字节串行缓冲器可处理发送更多数据。

- Added [Stepper feature](https://github.com/firmata/protocol/blob/master/stepper.md) to enable interfacing with 2 wire and 4 wire stepper motor drivers and step + direction drivers.
新增【步进功能（https://github.com/firmata/protocol/blob/master/stepper.md），以实现与2线和4线步进电机驱动器和步+方向驱动程序接口。


*Note: The above 4 features were initially added for `ConfigurableFirmata` which had a different version number. They have been moved under the `v2.4.0` release here to get things back on track for the protocol version.*
*注：以上4功能最初增加了`ConfigurableFirmata`其中有一个不同的版本号。他们已经在`v2.4.0`下释放搬到这里把事情回到正轨的协议版本。*
## Version 2.3.0 - February 2013

- Angle was removed from the `SERVO_CONFIG` message.

## Version 2.2.0 - January 2011

- Added [Extended Analog](https://github.com/firmata/protocol/blob/master/protocol.md#extended-analog) to allow addressing beyond pin 15 and support analog values with any number of bits.
- 加入[扩展模拟]（https://github.com/firmata/protocol/blob/master/protocol.md#extended-analog），以允许处理以外销15和支持模拟值与任何数目的位。
- Added [Capability Query](https://github.com/firmata/protocol/blob/master/protocol.md#capability-query) to query the capabilities supported by each pin.
- 新增【能力查询（https://github.com/firmata/protocol/blob/master/protocol.md#capability-query）来查询每个引脚所支持的功能。
- Added [Analog Mapping Query](https://github.com/firmata/protocol/blob/master/protocol.md#analog-mapping-query) to map analog pin numbers to their digital pin number equivalent.
- 新增[模拟测绘查询】（https://github.com/firmata/protocol/blob/master/protocol.md#analog-mapping-query）转换为模拟引脚数映射到其数字引脚数相同。
- Added [Pin State Query](https://github.com/firmata/protocol/blob/master/protocol.md#pin-state-query) to query the state of pin (output value or if input pullup enabled).新增【端子状态查询（https://github.com/firmata/protocol/blob/master/protocol.md#pin-state-query）查询引脚的状态（输出值，或者如果启用输入上拉）。
- 

## Version 2.1.0 - March 2010

- Added [I2C feature](https://github.com/firmata/protocol/blob/master/i2c.md) to interface with I2C devices.
- 新增[I2C功能（https://github.com/firmata/protocol/blob/master/i2c.md）与I2C设备的接口。
- Added [Servo feature](https://github.com/firmata/protocol/blob/master/servos.md) to interface with servo motors.
- 新增[伺服功能（https://github.com/firmata/protocol/blob/master/servos.md）与伺服电机的接口。
- Added ability to change the [sampling interval](https://github.com/firmata/protocol/blob/master/protocol.md#sampling-interval).
-新增更改[采样间隔]的能力（https://github.com/firmata/protocol/blob/master/protocol.md#sampling-interval）。
## Version 2.0.0 - September 2008

- Changed to 8-bit port-based digital messages (a collection of 8 pins) to mirror ports from previous 14-bit ports (a collection of 14 pins) modeled after the standard (ATmega8/168/328) Arduino boards.
- 改为8位基于端口的数字信息（8个管脚的集合），以反映从以前的14位端口的标准（ATmega8的/ 168/328）的Arduino板仿照端口（14针的集合）。
- Added ability to [query firmware name and version](https://github.com/firmata/protocol/blob/master/protocol.md#query-firmware-name-and-version).
-新增能力[查询固件名称和version](https://github.com/firmata/protocol/blob/master/protocol.md#query-firmware-name-and-version).
## Version 1.0.0

- Switched to MIDI-compatible packet format (though the message interpretation differs).
- 切换到MIDI兼容包格式（虽然消息解释不同）。
