Servo
===

Send a Servo config message to configure a pin a servo. Then use the `SET_PIN_MODE`
message to attach/detach Servo support to a pin. This saves space in the protocol
by reusing the `SET_PIN_MODE` message, but the host software implementation
could have a different interface, e.g. Arduino's `attach()` and `detach()`.   
发送一个伺服配置信息来配置一个伺服系统。然后用set_pin_mode消息附加/分离伺服支持pin。这节省了通过复用set_pin_mode消息协议中的空间，但主机软件实现可能有不同的接口，如Arduino的attach()和detach()。

The `SERVO_CONFIG` message can be sent at any time to chang the settings.   
`SERVO_CONFIG`消息可以在任何时候昌的设置被发送。

Added in version 2.1.0.   
新增的2.1.0版本。

Servo config   
伺服配置
```
// minPulse and maxPulse are 14-bit unsigned integers
0  START_SYSEX          (0xF0)
1  SERVO_CONFIG         (0x70)
2  pin number           (0-127)
3  minPulse LSB         (0-6)
4  minPulse MSB         (0-13)
5  maxPulse LSB         (0-6)
6  maxPulse MSB         (7-13)
7  END_SYSEX            (0xF7)
```

*This is just the standard `SET_PIN_MODE` message:*
Set digital pin mode   
集数字引脚模式
```
0  set digital pin mode (0xF4) (MIDI Undefined)
1  pin number           (0-127)
2  state                (SERVO, 4)
```

Write to servo, servo write is performed if the pin mode is SERVO    
写伺服，如果引脚模式是SERVO 进行伺服写
```
0  ANALOG_MESSAGE       (0xE0-0xEF)
1  value LSB
2  value MSB
```

If the pin number is higher than 15, or if the value to write to the servo is
greater than 14 bits, then the Extended Analog message can be used in place
of the standard `ANALOG_MESSAGE`:    
如果引脚数比15以上，或如果该值写入伺服大于14位，则可以代替使用的扩展模拟消息
标准的`ANALOG_MESSAGE`

```
0  START_SYSEX              (0xF0)
1  extended analog message  (0x6F)
2  pin                      (0-127)
3  bits 0-6                 (least significant byte)
4  bits 7-13                (most significant byte)
... additionaly bytes may be sent if more bits are needed
N  END_SYSEX                (0xF7)
```
