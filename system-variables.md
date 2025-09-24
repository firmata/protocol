# SYSTEM_VARIABLES message group

With more and more new boards available and in wide use, and particularly also really powerful ones like ESP32, certain features of the boards should be queryable by the firmata host. Sometimes, new features for these boards are required for certain special use cases. Some of the features I'm thinking of involve:

- Setting the ADC bit depth
- Setting the PWM frequency
- Query the input buffer size
- Setting power states
- etc.

It seems ugly to create new components and hence new main SYSEX commands for such simple tasks. This would not only unnecessarily use up the remaining numbers, it would also add complexity to the firmware. Instead, a single command should be used for setting/retrieving such "System Variables". 

Set/Query System Variable command (used both directions)
----
```
0  START_SYSEX       (0xF0)
1  SYSTEM_VARIABLES (0x66)
2  Operation (0 = query value, 1 = set value)
3  Data type (see below)
4  Status (0 = success, 1 = value is read-only, 2 = value is write-only, see below)
5..6 14 Bit variable ID
7 Pin number (if applicable, 0x7f otherwise)
8...n-1 Value, compressed (e.g. 5 bytes for an int32 field)
n  END_SYSEX         (0xF7)
```
To keep the implementation in the firmware simple, only few different data types will probably be supported.


Data type
----
```
0 Undefined (do not use)
1 A single Int32 value
```

Status
The status field is filled by the firmware to report the result of the operation. 

| Value | Name  | Meaning when operation was "query" | Meaning when operation was "set"
| ------------- | ------------- | --- | --- |
| 0 | Success | The value contains the queried data | The value contains the new value
| 1 | Field is read-only | (never) | The value contains the existing value
| 2 | Field is write-only | The operation failed, the value field is undefined | (never)
| 3 | The data type is unknown |  |  |
| 4 | The Variable ID is unknown | | |
| 5 | Some other error (check log messages) | | |

Each message from the host is answered, either with a success message or an error, except if the message is syntactically invalid (e.g. to few bytes)

Variable IDs
----
| Value | Data Type | R/W | Description |
| ---- |----|----|----|
| 0 | Int | R | Function support check. Always returns 1 |
| 1 | Int | R | Query maximum length of SYSEX message)|
| 2 | Int | R | Query input buffer size. When using network communication, systems with enough memory are typically able to handle multiple messages at once, but to avoid data loss, sending more data than indicated by this value will cause significant extra delay. Example: On the ESP32, the input buffer is typically 4kb, and SPI messages can be sent in chunks of 4kb. But after that, either a message requiring an ACK or a delay is required. Otherwise, a network retransmission is necessary, causing a significant performance drop.
| 3-99 | Reserved | |
| 100 | Int | R/(W) | Set / Query ADC bit depth, per pin|
| 101 | Int | R/(W) | Set / Query PWM frequency, per pin|
| 102 | Int | (R)/W | Enter sleep mode after X seconds|
| 103 | Int | R/W | Set Sleep wakeup pin. The argument defines whether the wakeup shall be on High (1) or low (0)|

Note: Support for all commands except 0 is optional. Some values might be read-write on some boards, but readonly on others. 

