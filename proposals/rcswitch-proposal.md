# RCSwitchFirmata Feature

[RCSwitchFirmata](https://github.com/git-developer/RCSwitchFirmata) is an adapter between [ConfigurableFirmata](https://github.com/firmata/ConfigurableFirmata) and the [RCSwitch](https://github.com/sui77/rc-switch) library.

RCSwitch is a library to send and receive messages to/from radio controlled devices. Sender and receiver are referred to as *devices* within the context of this document. Multiple devices may be used at the same time; the only requirement is a pin per device. All devices may be used and configured independently. Thus, this document separates the main functions *send* and *receive*. RCSwitchFirmata is subdivided into *RCOutputFirmata* implementing the send function and *RCInputFirmata* implementing the receive function.

## Protocol details

A common pattern of all queries is that they echo the query message as response. This pattern allows for detection of unsupported or wrong messages.

### Tristate bits
RCSwitch supports - besides the types `long` and `char[]` - so-called *tristate* bits. A tristate bit has one of the values 0, 1, or F. Each tristate bit is coded as 2 bits as follows:
```
TRISTATE_0              0x00
TRISTATE_F              0x01
TRISTATE_RESERVED       0x02
TRISTATE_1              0x03
```
Thus, 1 byte consisting of 8 bits ABCDEFGH may hold up to 4 tristate bits AB, CD, EF and GH. The leftmost 2 bits represent the first tristate bit, the rightmost 2 bits represent the fourth tristate bit. If less than 4 tristate bits are used, the byte is filled with the reserved value `0x02`.

### Send
#### Attach sender
Query:
```c
 /*
  * -----------------------------------------------------
  * 0 START_SYSEX                (0xF0)
  * 1 RCOUTPUT_DATA              (0x5C)
  * 2 RCOUTPUT_ATTACH            (0x01)
  * 3 pin                        (pin number)
  * 4 END_SYSEX                  (0xF7)
  * -----------------------------------------------------
  */
```

Response: the query message

#### Detach sender
Query:
```c
 /*
  * -----------------------------------------------------
  * 0 START_SYSEX                (0xF0)
  * 1 RCOUTPUT_DATA              (0x5C)
  * 2 RCOUTPUT_DETACH            (0x02)
  * 3 pin                        (pin number)
  * 4 END_SYSEX                  (0xF7)
  * -----------------------------------------------------
  */
```

Response: the query message

#### Configuration of rcswitch parameter `protocol`
Query:
```c
 /*
  * -----------------------------------------------------
  * 0 START_SYSEX                (0xF0)
  * 1 RCOUTPUT_DATA              (0x5C)
  * 2 RCOUTPUT_PROTOCOL          (0x11)
  * 3 pin                        (pin number)
  * 4 protocol (int), bits 0-6
  * 5 protocol (int), bits 7-13
  * 6 protocol (int), bits 14-15
  * 7 END_SYSEX                  (0xF7)
  * -----------------------------------------------------
  */
```

Response: the query message

#### Configuration of rcswitch parameter `pulse length`
Query:
```c
 /*
  * -----------------------------------------------------
  * 0 START_SYSEX                (0xF0)
  * 1 RCOUTPUT_DATA              (0x5C)
  * 2 RCOUTPUT_PULSE_LENGTH      (0x12)
  * 3 pin                        (pin number)
  * 4 pulse length (int), bits 0-6
  * 5 pulse length (int), bits 7-13
  * 6 pulse length (int), bits 14-15
  * 7 END_SYSEX                  (0xF7)
  * -----------------------------------------------------
  */
```

Response: the query message

#### Configuration of rcswitch parameter `repeat transmit`
Query:
```c
 /*
  * -----------------------------------------------------
  * 0 START_SYSEX                (0xF0)
  * 1 RCOUTPUT_DATA              (0x5C)
  * 2 RCOUTPUT_PULSE_LENGTH      (0x14)
  * 3 pin                        (pin number)
  * 4 repeat transmit (int), bits 0-6
  * 5 repeat transmit (int), bits 7-13
  * 6 repeat transmit (int), bits 14-15
  * 7 END_SYSEX                  (0xF7)
  * -----------------------------------------------------
  */
```

Response: the query message

#### Send tristate code as char array
Query:
```c
 /*
  * -----------------------------------------------------
  * 0 START_SYSEX                (0xF0)
  * 1 RCOUTPUT_DATA              (0x5C)
  * 2 RCOUTPUT_TRISTATE          (0x21)
  * 3 pin                        (pin number)
  * 4..n-1 RC data (packed as 7-bit): char array with tristate bits ('0', '1', 'F')
  * n END_SYSEX                  (0xF7)
  * -----------------------------------------------------
  */
```

Response: the query message

#### Send long code
Query:
```c
 /*
  * -----------------------------------------------------
  * 0 START_SYSEX                (0xF0)
  * 1 RCOUTPUT_DATA              (0x5C)
  * 2 RCOUTPUT_CODE_LONG         (0x22)
  * 3 pin                        (pin number)
  * 4..n-1 RC data (packed as 7-bit): 2 bytes (int) with the number of bits to send, 4 bytes (long) data bits
  * n END_SYSEX                  (0xF7)
  * -----------------------------------------------------
  */
```

Response: the query message

#### Send char array
Query:
```c
 /*
  * -----------------------------------------------------
  * 0 START_SYSEX                (0xF0)
  * 1 RCOUTPUT_DATA              (0x5C)
  * 2 RCOUTPUT_CODE_CHAR         (0x24)
  * 3 pin                        (pin number)
  * 4..n-1 RC data (packed as 7-bit): char array with characters to send
  * n END_SYSEX                  (0xF7)
  * -----------------------------------------------------
  */
```

Response: the query message

#### Send tristate code as packed tristate bits
Query:
```c
 /*
  * -----------------------------------------------------
  * 0 START_SYSEX                   (0xF0)
  * 1 RCOUTPUT_DATA                 (0x5C)
  * 2 RCOUTPUT_CODE_TRISTATE_PACKED (0x28)
  * 3 pin                           (pin number)
  * 4..n-1 RC data (packed as 7-bit): byte array with 4 tristate bits per byte
  * n END_SYSEX                     (0xF7)
  * -----------------------------------------------------
  */
```

Response: the query message

### Receive
#### Attach receiver
Query:
```c
 /*
  * -----------------------------------------------------
  * 0 START_SYSEX                (0xF0)
  * 1 RCINPUT_DATA               (0x5D)
  * 2 RCINPUT_ATTACH             (0x01)
  * 3 pin                        (pin number)
  * 4 END_SYSEX                  (0xF7)
  * -----------------------------------------------------
  */
```

Response: the query message

#### Detach receiver
Query:
```c
 /*
  * -----------------------------------------------------
  * 0 START_SYSEX                (0xF0)
  * 1 RCINPUT_DATA               (0x5D)
  * 2 RCINPUT_DETACH             (0x02)
  * 3 pin                        (pin number)
  * 4 END_SYSEX                  (0xF7)
  * -----------------------------------------------------
  */
```

Response: the query message

#### Configuration of rcswitch parameter `receive tolerance` (in percent)
Query:
```c
 /*
  * -----------------------------------------------------
  * 0 START_SYSEX                (0xF0)
  * 1 RCINPUT_DATA               (0x5D)
  * 2 RCINPUT_TOLERANCE          (0x31)
  * 3 pin                        (pin number)
  * 4 tolerance                  (percent)
  * 5 END_SYSEX                  (0xF7)
  * -----------------------------------------------------
  */
```

Response: the query message

#### Configuration parameter `enable raw data`
Query:
```c
 /*
  * -----------------------------------------------------
  * 0 START_SYSEX                (0xF0)
  * 1 RCINPUT_DATA               (0x5D)
  * 2 RCINPUT_ENABLE_RAW_DATA    (0x32)
  * 3 pin                        (pin number)
  * 4 rawdataEnabled             (0 for disabled, 1 for enabled)
  * 5 END_SYSEX                  (0xF7)
  * -----------------------------------------------------
  */
```

#### Receive a RF message
Query: none

Response:
```c
 /*
  * -----------------------------------------------------
  *  0 START_SYSEX                (0xF0)
  *  1 RCINPUT_DATA               (0x5D)
  *  2 RCINPUT_MESSAGE            (0x41)
  *  3 pin                        (pin number)
  *  4 message value (long), bits 0-6
  *  5 message value (long), bits 7-13
  *  6 message value (long), bits 14-20
  *  7 message value (long), bits 21-27
  *  8 message value (long), bits 28-32
  *  9 bitlength (int), bits 0-6
  * 10 bitlength (int), bits 7-13
  * 11 bitlength (int), bits 14-15
  * 12 delay (int), bits 0-6
  * 13 delay (int), bits 7-13
  * 14 delay (int), bits 14-15
  * 15 protocol (int), bits 0-6
  * 16 protocol (int), bits 7-13
  * 17 protocol (int), bits 14-15
  * 18..n-1 raw data (int[]); optional: only if rawdata was enabled
  * n END_SYSEX                  (0xF7)
  * -----------------------------------------------------
  */
```
