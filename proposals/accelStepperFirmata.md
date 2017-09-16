accelStepperFirmata
===

Provides support for full 2 wire, full 3 wire, full 4 wire, half 3 wire, and half 4 wire stepper motor drivers (H-bridge, darlington array, etc) as well as step + direction drivers such as the [EasyDriver](http://www.schmalzhaus.com/EasyDriver/).
Current implementation supports 10 stepper motors at the same time (#[0-9]).

Includes optional support for acceleration and deceleration of the motor.

Also includes multiStepper support which allows groups of steppers to be simultaneously controlled. Up to five multiStepper groups can be created. The total number of steppers is still limited to 10.

AccelStepperFirmata sends and receives floats in a custom format described at the end
of this document.

Example files:
 * accelStepperFirmata has not yet been implemented.

Protocol
---

**Stepper configuration**

This message is required and must be sent prior to any other message. The device number is arbitrary, but must be unique.

```
0  START_SYSEX                                (0xF0)
1  accelStepper Command                       (0x62)
2  config command                             (0x00 = config)
3  device number                              (0-9) (Supports up to 10 motors)

4  interface                                  (upper 3 bits = wire count:
                                                001XXXX = driver
                                                010XXXX = two wire
                                                011XXXX = three wire
                                                100XXXX = four wire)

                                              (4th - 6th bits = step type
                                                step size = 1/2^0bXXX 
                                                Examples: 
                                                XXX000X = whole step
                                                XXX001X = half step
                                                XXX010X = quarter step 
                                                etc...)

                                              (lower 1 bit = has enable pin:
                                                XXXXXX0 = no enable pin
                                                XXXXXX1 = has enable pin)

5  motorPin1 or stepPin number                (0-127)
6  motorPin2 or directionPin number           (0-127)
7  [when interface >= 0x011] motorPin3        (0-127)
8  [when interface >= 0x100] motorPin4        (0-127)
9  [when interface && 0x0000001] enablePin    (0-127)
10 [optional] pins to invert                  (lower 5 bits = pins:
                                                XXXXXX1 = invert motorPin1
                                                XXXXX1X = invert motorPin2
                                                XXXX1XX = invert motorPin3
                                                XXX1XXX = invert motorPin4
                                                XX1XXXX = invert enablePin)
11 END_SYSEX                                  (0xF7)
```

**Stepper zero**

accelStepper will store the current absolute position of the stepper motor (in steps). Sending the zero command will reset the position value to zero without moving the stepper. 

```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  zero command                            (0x01)
3  device number                           (0-9)
4  END_SYSEX                               (0xF7)
```

**Stepper step (relative move)**
 
Steps to move is specified as a 32-bit signed long.

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
9  END_SYSEX                               (0xF7)
```

**Stepper to (absolute move)**

Moves a stepper to a desired position based on the number of steps from the zero position.
Position is specified as a 32-bit signed long.

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
9 END_SYSEX                               (0xF7)
```

**Stepper enable**

For stepper motor controllers that are configured with an enable pin, the enable command manages whether the controller passes voltage through to the motor. When a stepper motor is idle, voltage is still being consumed so if the stepper motor does not need to hold its position use enable to save power.

```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  enable command                          (0x04)
3  device number                           (0-9)
4  device state                            (HIGH : enabled | LOW : disabled)
5  END_SYSEX                               (0xF7)
```

**Stepper stop**

Stops a stepper motor. Results in ```STEPPER_MOVE_COMPLETE``` being sent to the client with the position of the motor when stop is completed note: If an acceleration is set, stop will not be immediate.

```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  stop command                            (0x05)
3  device number                           (0-9)
4  END_SYSEX                               (0xF7)
```

**Stepper report position (request)**

Request a position report.

```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  report position command                 (0x06)
3  device number                           (0-9)
4  END_SYSEX                               (0xF7)
```

**Stepper report position (reply)**

Sent when a report position is requested. Position is reported as a 32-bit signed long.

```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  report position command                 (0x06)
3  device number                           (0-9)
4  position, bits 0-6
5  position, bits 7-13
6  position, bits 14-20
7  position, bits 21-27
8  position, bits 28-32
9  END_SYSEX                               (0xF7)
```

**Stepper move complete**

Sent when a move completes. Position is reported as a 32-bit signed long.

```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  move complete command                   (0x0b)
3  device number                           (0-9)
4  position, bits 0-6
5  position, bits 7-13
6  position, bits 14-20
7  position, bits 21-27
8  position, bits 28-32
9  END_SYSEX                               (0xF7)
```

**Stepper limit**

*Not yet implemented*

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
using accelStepperFirmata's custom float format described below.


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

**Stepper set speed**

If acceleration is off (equal to zero) sets the speed in steps per second. 
If acceleration is on (non-zero) sets the maximum speed in steps per second. 
The speed value is passed using accelStepperFirmata's custom float format described below.

```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  set speed command                       (0x09)
3  device number                           (0-9) (Supports up to 10 motors)
4  maxSpeed, bits 0-6                      (maxSpeed in steps per sec)
5  maxSpeed, bits 7-13
6  maxSpeed, bits 14-20
7  maxSpeed, bits 21-28
8  END_SYSEX                               (0xF7)
```

**MultiStepper configuration**

Stepper instances that have been created with the stepper configuration command above can be added to a multiStepper group. Groups can be sent a list of devices/positions in a single command and their movements will be coordinated to begin and end simultaneously. Note that multiStepper does not support acceleration or deceleration.

```
0  START_SYSEX                              (0xF0)
1  Stepper Command                          (0x62)
2  multiConfig command                      (0x20)
3  group number                             (0-4)
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
3  group number                             (0-4)
4  position, bits 0-6
5  position, bits 7-13
6  position, bits 14-20
7  position, bits 21-27
8  position, bits 28-32

*Repeat 4 through 8 for each device in group*

53 END_SYSEX                                (0xF7)
```

**MultiStepper stop**

Immediately stops all steppers in the group.

```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  multi stop command                      (0x23)
3  group number                            (0-4)
4  END_SYSEX                               (0xF7)
```

**MultiStepper move compelte**

Sent when a multiStepper move completes.

```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  multi stepper move complete command     (0x24)
3  group  number                           (0-4)
4  END_SYSEX                               (0xF7)
```

AccelStepperFirmata's Custom Float Format
---

Floats sent and received by accelStepperFirmata are composed of a 23-bit significand (mantissa)
and a 4-bit, base 10 exponent (biased -11 with an explicit 1's bit) and a sign bit.

|0-20                  |21   |22-25   |26-27   
|----------------------|-----|--------|---------------------
|least significant bits|sign |exponent|most significant bits 
|21 bits               |1 bit|4 bits  |2 bits

These values allow a range from 8.388608*10^-11 to 83886.08. Small enough to represent one step per year and large enough to exceed our max achievable stepper speed.

**Example 1: 1 step per hour**

1 step per hour = 1 step / 60 minutes / 60 seconds = 0.000277... steps per second

The largest integer that can be represented in 23 bits is 8388608 so the
significand will be limited to 6 or 7 digits. In this case 2777777 (note the value truncates).

The exponent is 4 bits which limits the range to 0-15, but we subtract
11 from that value on the receiving end to give us a range from -11 to 4. In
this example we are passing 1 to give us a -10 value in the exponent.

|            | Decimal| Binary                 |
|------------|--------|------------------------|
|Significand | 2777777| 01010100110001010110010|
|Exponent    |       1|                    0001|
|Sign        |       0|                       0|



Values in firmata are passed in the 7 least significant bits of each message byte
so we will be passing in 4 bytes in this order:

|                                           | Binary  | Hex |
|-------------------------------------------|---------|-----|
| Least most significant bits               |  0110001| 0x31|
| Next most significant bits                |  1000101| 0x45|
| Next most significant bits                |  0101001| 0x29|
| Sign, Exponent and 2 most significant bits|  0000101| 0x05|

**Example 2: 100 steps per second**

We have to pad our significand on the right with four zeros to get our full
precision. That makes the significand 100000000 and our exponent value will be 2.
Since the value we send will be biased -11 on the receiving end, we send 13 in
the exponent.

|            | Decimal| Binary                 |
|------------|--------|------------------------|
|Significand |       1| 00000000000000000000001|
|Exponent    |      13|                    1101|
|Sign        |       0|                       0|

Values in firmata are passed in the 7 least significant bits of each message byte
so we would be passing in 4 bytes in this order:

|                                           | Binary  | Hex |
|-------------------------------------------|---------|-----|
| Least most significant bits               |  0000001| 0x01|
| Next most significant bits                |  0000000| 0x00|
| Next most significant bits                |  0000000| 0x00|
| Sign, Exponent and 2 most significant bits|  0110100| 0x34|