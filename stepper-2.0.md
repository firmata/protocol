Stepper Motor 2.0
===

Provides support for full 2 wire, full 3 wire, full 4 wire, half 3 wire, and half 4 wire stepper motor drivers (H-bridge, darlington array, etc) as well as step + direction drivers such as the [EasyDriver](http://www.schmalzhaus.com/EasyDriver/).
Current implementation supports 10 stepper motors at the same time (#[0-9]).

Includes optional support for acceleration and deceleration of the motor.

Also includes multiStepper support which allows groups of steppers to be simultaneously passed different ```to``` values and the duration of their movements will match. Up to five multiStepper groups can be created. The total number of steppers is still limited to 10.

Example files:
 * Version 2.0 of the stepper protocol has not yet been implemented.

Protocol
---

**Stepper configuration**
```
0  START_SYSEX                                (0xF0)
1  Stepper Command                            (0x62)
2  config command                             (0x00 = config)
3  device number                              (0-9) (Supports up to 10 motors)

4  interface                                  (upper 3 bits = wire count:
                                                000XXXX = driver
                                                010XXXX = two wire
                                                011XXXX = three wire
                                                100XXXX = four wire)

                                              (4th - 6th bits = step type
                                                XXX000X = whole step
                                                XXX001X = half step
                                                XXX010X = micro step)

                                              (lower 1 bit = has enable pin:
                                                XXXXXX0 = no enable pin
                                                XXXXXX1 = has enable pin)

5  minPulseWidth                              (0-127 measured in us)
6  num steps-per-revolution LSB
7  num steps-per-revolution MSB
8  maxSpeed LSB
9  maxSpeed MSB
10 motorPin1 or directionPin number           (0-127)
11 motorPin2 or stepPin number                (0-127)
12 [when interface >= 0x0110000] motorPin3    (0-127)
13 [when interface >= 0x1000000] motorPin4    (0-127)
14 [when interface && 0x0000001] enablePin    (0-127)
15 [optional] pins to invert                  (lower 5 bits = pins:
                                                XXXXXX1 = invert motorPin1
                                                XXXXX1X = invert motorPin2
                                                XXXX1XX = invert motorPin3
                                                XXX1XXX = invert motorPin4
                                                XX1XXXX = invert enablePin)
16 END_SYSEX                                  (0xF7)
```

**Stepper zero**
```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  zero command                            (0x01)
3  device number                           (0-9)
4  END_SYSEX                               (0xF7)
```

**Stepper step (relative move)**

Position is specified as a 32-bit unsigned long. Speed is specified as a
16-bit unsigned int.

```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  step command                            (0x02)
3  device number                           (0-9)
4  direction                               (0-1) (0x00 = CW, 0x01 = CCW)
5  num steps, bits 0-6
6  num steps, bits 7-13
7  num steps, bits 14-20
8  num steps, bits 21-27
9  num steps, bits 28-32
10 speed, bits 0-6                         (steps per second * 1000)
11 speed, bits 7-13
12 speed, bits 14-16
12 END_SYSEX                               (0xF7)
```

**Stepper to (absolute move)**

Sets a stepper to a desired position based on the number of steps from the zero position.
Position is specified as a 32-bit signed long.

```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  to command                              (0x03)
3  device number                           (0-9)
4  num steps, bits 0-6
5  num steps, bits 7-13
6  num steps, bits 14-20
7  num steps, bits 21-27
8  num steps, bits 28-32
9  speed, bits 0-6                         (steps per second * 1000)
10 speed, bits 7-13
11 speed, bits 14-16
12 [optional] accel, bits 0-6              (acceleration in steps/sec^2 * 1000)
13 [optional] accel, bits 7-13
14 [optional] accel, bits 14-16
15 [optional] decel, bits 0-6              (deceleration in steps/sec^2 * 1000)
16 [optional] decel, bits 7-13
17 [optional] decel, bits 14-16
18 END_SYSEX                               (0xF7)
```

**Stepper enable**
```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  enable command                          (0x04)
3  device number                           (0-9)
4  device state                            (HIGH : enabled | LOW : disabled)
4  END_SYSEX                               (0xF7)
```

**Stepper stop**
```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  stop command                            (0x05)
3  device number                           (0-9)
4  END_SYSEX                               (0xF7)
```

**Stepper report position**

Sent when a move completes or stop is called. Position is reported as a 32-bit signed long.

```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  stop reply command                      (0x06)
3  device number                           (0-9)
4  position, bits 0-6
5  position, bits 7-13
6  position, bits 14-20
7  position, bits 21-27
8  position, bits 28-32
9  END_SYSEX                               (0xF7)
```

**Stepper limit**

When a limit pin (digital) is set to its limit state, movement in that direction is disabled.

```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  stop limit command                      (0x07)
3  device number                           (0-9)
4  lower limit pin number                  (0-127)
5  lower limit state                       (0x00 | 0x01)
6  upper limit pin number                  (0-127)
7  upper limit state                       (0x00 | 0x01)
8  END_SYSEX                               (0xF7)
```

**Stepper set acceleration**

Sets the acceleration/deceleration in steps/sec^2. Value is specified as step/sec^2 * 1000 since Firmata doesn't support sending and receiving floats.

```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  set acceleration command                (0x08)
3  device number                           (0-9) (Supports up to 10 motors)
4  accel, bits 0-6                         (acceleration in steps/sec^2 * 1000)
5  accel, bits 7-13
6  accel, bits 14-16
7  END_SYSEX                               (0xF7)
```

**MultiStepper configuration**

Stepper instances that have been created with the stepper configuration command above can be added to a multiStepper group. Groups can be sent a list of devices/positions in a single command and their movements will be coordinated to begin and end simultaneously. Note that multiStepper does not support acceleration or deceleration.

```
0  START_SYSEX                              (0xF0)
1  Stepper Command                          (0x62)
2  config command                           (0x20)
3  group number                             (0-127)
4  member 0x00 device number                (0-9)
5  member 0x01 device number                (0-9)
6  [optional] member 0x02 device number     (0-9)
7  [optional] member 0x03 device number     (0-9)
8  [optional] member 0x04 device number     (0-9)
9  [optional] member 0x05 device number     (0-9)
10 [optional] member 0x06 device number     (0-9)
11 [optional] member 0x07 device number     (0-9)
12 [optional] member 0x08 device number     (0-9)
13 [optional] member 0x09 device number     (0-9)
14 END_SYSEX                                (0xF7)
```

**MultiStepper to**

```
0  START_SYSEX                              (0xF0)
1  Stepper Command                          (0x62)
2  multi to command                         (0x21)
3  group number                             (0-127)
4  member number                            (0-9)
5  num steps, bits 0-6
6  num steps, bits 7-13
7  num steps, bits 14-20
8  num steps, bits 21-27
9  num steps, bits 28-32

*Optionally repeat 4 through 9 for each device in group*

63 END_SYSEX                                (0xF7)
```

**MultiStepper stop**
```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  multi stop command                      (0x23)
3  group number                            (0-127)
4  END_SYSEX                               (0xF7)
```

**MultiStepper report positions**

Sent when a move completes or stop is called.

```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  multi stop reply command                (0x24)
3  group  number                           (0-127)
4  member number                           (0-9)
5  position, bits 0-6
6  position, bits 7-13
7  position, bits 14-20
8  position, bits 21-27
9  position, bits 28-32

*Optionally repeat 4 through 9 for each device in group*

63 END_SYSEX                               (0xF7)
```
