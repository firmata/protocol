Stepper Motor 2.0
===

Provides support for full 2 wire, full 3 wire, full 4 wire, half 3 wire, and half 4 wire stepper motor drivers (H-bridge, darlington array, etc) as well as step + direction drivers such as the [EasyDriver](http://www.schmalzhaus.com/EasyDriver/).
Current implementation supports 10 stepper motors at the same time (#[0-9]).

Includes optional support for acceleration and deceleration of the motor.

Also includes multiStepper support which allows groups of steppers to be simultaneously controlled. Up to five multiStepper groups can be created. The total number of steppers is still limited to 10.

Stepper 2.0 sends and receives floats in a custom format described at the end
of this document.

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

Steps to move is specified as a 32-bit signed long.

The speed value is a float passed using Stepper 2.0's custom float format described below.

```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  step command                            (0x02)
3  device number                           (0-9)
4  num steps, bits 0-6
5  num steps, bits 7-13
6  num steps, bits 14-20
7  num steps, bits 21-27
8  num steps, bits 28-32
9  speed, bits 0-6                         (steps per second)
10 speed, bits 7-13
11 speed, bits 14-20
12 speed, bits 21-28
13 END_SYSEX                               (0xF7)
```

**Stepper to (absolute move)**

Sets a stepper to a desired position based on the number of steps from the zero position.
Position is specified as a 32-bit signed long.

The speed value is a float passed using Stepper 2.0's custom float format described below.

```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  to command                              (0x03)
3  device number                           (0-9)
4  position, bits 0-6
5  position, bits 7-13
6  position, bits 14-20
7  position, bits 21-27
8  position, bits 28-32
9  speed, bits 0-6                         (steps per second)
10 speed, bits 7-13
11 speed, bits 14-20
12 speed, bits 21-27
13 END_SYSEX                               (0xF7)
```

**Stepper enable**
```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  enable command                          (0x04)
3  device number                           (0-9)
4  device state                            (HIGH : enabled | LOW : disabled)
5  END_SYSEX                               (0xF7)
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

Sets the acceleration/deceleration in steps/sec^2. The accel value is passed
using Stepper 2.0's custom float format described below.


```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  set acceleration command                (0x08)
3  device number                           (0-9) (Supports up to 10 motors)
4  accel, bits 0-6                         (acceleration in steps/sec^2)
5  accel, bits 7-13
6  accel, bits 14-20
7  accel, bits 21-28
8  END_SYSEX                               (0xF7)
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

Sets each stepper in a group to a desired position based on the number of
steps from its zero position. Positions are specified as a 32-bit signed long.

Stepper movements will be coordinated so that all arrive at their desired
position simultaneously. The duration of this move is based on which stepper
will take the longest given the change in position and the stepper's max speed.

```
0  START_SYSEX                              (0xF0)
1  Stepper Command                          (0x62)
2  multi to command                         (0x21)
3  group number                             (0-127)
4  position, bits 0-6
5  position, bits 7-13
6  position, bits 14-20
7  position, bits 21-27
8  position, bits 28-32

*Repeat 4 through 8 for each device in group*

53 END_SYSEX                                (0xF7)
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
4  position, bits 0-6
5  position, bits 7-13
6  position, bits 14-20
7  position, bits 21-27
8  position, bits 28-32

*Repeat 4 through 8 for each device in group*

53 END_SYSEX                               (0xF7)
```

Stepper 2 Custom Float Format
---

Floats sent and received by Stepper 2 are composed of a 23-bit significand (mantissa)
and a 4-bit, base 10 exponent (biased -17 with an explicit 1's bit) and a sign bit.

|27   |26-23   |22-0       |
|-----|--------|-----------|
|sign |exponent|significant|
|1 bit|4 bits  |23 bits    |

Those values allow a range from 8.388608*10^-11 to 83886.08.

**Example 1: 1 step per hour**

1 step per hour = 1 step / 60 minutes / 60 seconds = 0.000277... steps per second

The largest integer that can be represented in 23 bits is 8388608 so the
significand will be limited to 6 or 7 digits. In this case 2777778.

The exponent is 4 bits which limits the range to 0-15, but we subtract
17 from that value on the receiving end to give us a range from -17 to -2. In
this example we pass 7 to give us a -10 value in the exponent.

|                       | Decimal| Binary                 |
|-----------------------|--------|------------------------|
|Sign (bit 27)          |       0|                       0|
|Exponent (bits 26-23)  |       7|                    0110|
|Significand (bits 22-0)| 2777778| 01010100110001010110010|

Values in firmata are passed in the 7 least significant bits of each message byte
so we will be passing in 4 bytes:

|                                           | Binary  | Hex |
|-------------------------------------------|---------|-----|
| Sign, Exponent and 2 most significant bits| 00011001| 0x19|
| Next most significant bits                | 00101001| 0x29|
| Next most significant bits                | 01000101| 0x45|
| Least most significant bits               | 00110010| 0x32|


**Example 2: 100 steps per second**

We have to pad our significand on the right with four zeros to get our full
precision. That makes the significand 1000000 and our exponent value will be -4.
Since the value we send will be biased -17 on the receiving end, we send 13 in
the exponent.

|                       | Decimal| Binary                 |
|-----------------------|--------|------------------------|
|Sign (bit 27)          |       0|                       0|
|Exponent (bits 26-23)  |      13|                    1101|
|Significand (bits 22-0)| 1000000| 00011110100001001000000|

Values in firmata are passed in the 7 least significant bits of each message byte
so we would be passing in 4 bytes:

|                                           | Binary  | Hex |
|-------------------------------------------|---------|-----|
| Sign, Exponent and 2 most significant bits| 00110100| 0x29|
| Next most significant bits                | 00111101| 0x29|
| Next most significant bits                | 00000100| 0x45|
| Least most significant bits               | 01000000| 0x32|
