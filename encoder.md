#Encoder Feature

Provide incremental encoders support, for both [linear](http://en.wikipedia.org/wiki/Linear_encoder) and [rotary](http://en.wikipedia.org/wiki/Rotary_encoder#Incremental_rotary_encoder) encoders.

This feature is based on based on [PJRC's implementation](http://www.pjrc.com/teensy/td_libs_Encoder.html). See [this article](http://www.pjrc.com/teensy/td_libs_Encoder.html) for more informations about how it works (well explained!).

Current implementation supports 5 encoders at the same time (#[0-4]) and you can activate automatic position reports every (SAMPLING_INTERVAL)ms. Reports are disabled by default.

For best Performances, connect only interrupt pins.

Added in version 2.4.0.

Example files : 
 * EncoderFeature is a contributed feature for [ConfigurableFirmata.ino](https://github.com/firmata/arduino/blob/configurable/examples/ConfigurableFirmata/ConfigurableFirmata.ino). 
 * A dedicated example is available. See [SimpleEncoderFirmata.ino](https://github.com/firmata/FirmataEncoder/tree/master/examples/SimpleFirmataEncoder). 

## Compatible client librairies
TODO : Update this

## Tested boards
This feature has been tested on :
 * Arduino Uno
 * Arduino Mega
 * Arduino Leonardo
 * Arduino Due

## Protocol details
The protocol below use exclusively SysEx queries and SysEx responses.

### Attach encoder query
Query :
```c
 /* -----------------------------------------------------
 * 0 START_SYSEX                (0xF0)
 * 1 ENCODER_DATA               (0x61)
 * 2 ENCODER_ATTACH             (0x00)
 * 3 encoder #                  ([0 - MAX_ENCODERS-1])
 * 4 pin A #                    (first pin) 
 * 5 pin B #                    (second pin)
 * 6 END_SYSEX                  (0xF7)
 * -----------------------------------------------------
 */
```
No response.

### Report encoder's position
Query
```c
 /* -----------------------------------------------------
 * 0 START_SYSEX                (0xF0)
 * 1 ENCODER_DATA               (0x61)
 * 2 ENCODER_REPORT_POSITION    (0x01)
 * 3 Encoder #                  ([0 - MAX_ENCODERS-1])
 * 4 END_SYSEX                  (0xF7)
 * -----------------------------------------------------
 */
```
 
Response 
```c
 /* -----------------------------------------------------
 * 0 START_SYSEX                (0xF0)
 * 1 ENCODER_DATA               (0x61)
 * 2 Encoder #  &  DIRECTION    [= (direction << 6) | (#)]
 * 3 current position, bits 0-6
 * 4 current position, bits 7-13
 * 5 current position, bits 14-20
 * 6 current position, bits 21-27
 * 7 END_SYSEX                  (0xF7)
 * -----------------------------------------------------
 */
```
Note : 
Byte #2 contains both encoder's number (i.e. channel) and encoder's direction.
Direction is stored on the seventh bit,  0 (LOW) for positive and 1 (HIGH) for negative.
```c
directionMask = 0x40; // B01000000
channelMask   = 0x3F; // B00111111 

//ex direction is negative and encoder is on index 2
direction = 1;
channel = 2;
bytes[2] =  (direction << 6) | (channel);
```

### Report all encoders positions
Query
```c
 /* -----------------------------------------------------
 * 0 START_SYSEX                (0xF0)
 * 1 ENCODER_DATA               (0x61)
 * 2 ENCODER_REPORT_POSITIONS   (0x02)
 * 3 END_SYSEX                  (0xF7)
 * -----------------------------------------------------
 */
```
 
Response 
```c
 /* -----------------------------------------------------
 * 0 START_SYSEX                (0xF0)
 * 1 ENCODER_DATA               (0x61)
 * 2 first enc. #  & first enc. dir. 
 * 4 first enc. position, bits 0-6
 * 5 first enc. position, bits 7-13
 * 6 first enc. position, bits 14-20
 * 7 first enc. position, bits 21-27
 * 8 second enc. #  & second enc. dir. 
 * ...
 * N END_SYSEX                  (0xF7)
 * -----------------------------------------------------
 */
```
Note : `Report encoder's position` response is a special case of `Report all encoders positions` response.


### Reset encoder position to zero 
Query
```c
 /* -----------------------------------------------------
 * 0 START_SYSEX                (0xF0)
 * 1 ENCODER_DATA               (0x61)
 * 2 ENCODER_RESET_POSITION     (0x03)
 * 3 encoder #                  ([0 - MAX_ENCODERS-1])
 * 4 END_SYSEX                  (0xF7)
 * -----------------------------------------------------
 */
```
 
No response.

### Enable/disable reporting 
Query
```c
 /* -----------------------------------------------------
 * 0 START_SYSEX                (0xF0)
 * 1 ENCODER_DATA               (0x61)
 * 2 ENCODER_REPORT_AUTO        (0x04)
 * 3 enable                     (0x00 => false, true otherwise)
 * 4 END_SYSEX                  (0xF7)
 * -----------------------------------------------------
 */
```
 
No response.

Note : when reports are enabled, EncoderFirmata feature send the message below at every SAMPLING interval (19ms by default) :
```c
 /* -----------------------------------------------------
 * 0 START_SYSEX                (0xF0)
 * 1 ENCODER_DATA               (0x61)
 * 2 first enc. #  & first enc. dir.   [= (direction << 6) | (#)] 
 * 4 first enc. position, bits 0-6
 * 5 first enc. position, bits 7-13
 * 6 first enc. position, bits 14-20
 * 7 first enc. position, bits 21-27
 * 8 second enc. #  & second enc. dir. [= (direction << 6) | (#)]
 * ...
 * N END_SYSEX                  (0xF7)
 * -----------------------------------------------------
 */
```

### Detach encoder
Query
```c
 /* -----------------------------------------------------
 * 0 START_SYSEX                (0xF0)
 * 1 ENCODER_DATA               (0x61)
 * 2 ENCODER_DETACH             (0x05)
 * 3 encoder #                  ([0 - MAX_ENCODERS-1])
 * 4 END_SYSEX                  (0xF7)
 * -----------------------------------------------------
 */
```
 
No response.

