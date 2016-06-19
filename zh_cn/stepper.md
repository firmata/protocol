Stepper Motor
===

Provides support for 4 wire and 2 wire stepper motor drivers (H-bridge, darlington array, etc) as well as step + direction drivers such as the [EasyDriver](http://www.schmalzhaus.com/EasyDriver/).
Current implementation supports 6 stepper motors at the same time (#[0-5]).
提供4线和2线步进电机驱动器（H桥，达林顿阵列等）的支持，以及步+方向驱动器，如EasyDriver。当前实现支持在同一时间6步进电机（＃[0-5]）。

Also includes optional support for acceleration and deceleration of the motor.
还包括用于将电动机的加速和减速的可选支持。

Added in version 2.4.0.

Example files:
 * The Stepper feature is include by default in [ConfigurableFirmata.ino](https://github.com/firmata/ConfigurableFirmata/blob/master/examples/ConfigurableFirmata/ConfigurableFirmata.ino).
 * 步进功能是默认包括[ConfigurableFirmata.ino](https://github.com/firmata/ConfigurableFirmata/blob/master/examples/ConfigurableFirmata/ConfigurableFirmata.ino)
 * [Example implementation](https://github.com/firmata/ConfigurableFirmata/blob/master/src/StepperFirmata.cpp) as a configurable Firmata feature class.
 * 示例实现（https://github.com/firmata/ConfigurableFirmata/blob/master/src/StepperFirmata.cpp）作为配置Firmata要素类。
 * [Example of Stepper implementation in StandardFirmata](https://github.com/soundanalogous/AdvancedFirmata). *Note the dependency on the FirmataStepper class.*
 *[在StandardFirmata步进实施示例]（https://github.com/soundanalogous/AdvancedFirmata）。 *注意：在FirmataStepper类的依赖*

Protocol
---

Stepper configuration

*Note: `stepDelay` is the the number of microseconds between steps. The default
value is 1us. You can change the delay to 2us (useful for high current stepper
motor drivers). Additional delay values can be added in the future.*
```
0  START_SYSEX                       (0xF0)
1  Stepper Command                   (0x72)
2  config command                    (0x00 = config, 0x01 = step)
3  device number                     (0-5) (supports up to 6 motors)
4  stepDelay | interface             (upper 4 bits = step delay:
                                        0000XXX = default 1us delay [default]
                                        0001XXX = 2us delay
                                        additional bits not yet used)

                                     (lower 3 bits = interface:
                                        XXXX001 = step + direction driver
                                        XXXX010 = two wire
                                        XXXX100 = four wire)
5  steps-per-revolution LSB
6  steps-per-revolution MSB
7  motorPin1 or directionPin number  (0-127)
8  motorPin2 or stepPin number       (0-127)
9  [only when interface = 0x04] motorPin3 (0-127)
10 [only when interface = 0x04] motorPin4 (0-127)
11 END_SYSEX                         (0xF7)
```

Stepper step
```
0  START_SYSEX          (0xF0)
1  Stepper Command      (0x72)
2  config command       (0x01)
3  device number        (0-5)
4  direction            (0-1) (0x00 = CW, 0x01 = CCW)
5  num steps byte1 LSB
6  num steps byte2
7  num steps byte3 MSB  (21 bits (2,097,151 steps max))
8  speed LSB            (steps in 0.01*rad/sec  (2050 = 20.50 rad/sec))
9  speed MSB
10 [optional] accel LSB (acceleration in 0.01*rad/sec^2 (1000 = 10.0 rad/sec^2))
11 [optional] accel MSB
12 [optional] decel LSB (deceleration in 0.01*rad/sec^2)
13 [optional] decel MSB
14 END_SYSEX            (0xF7)
```
