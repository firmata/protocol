# Frequency measurement feature

Provides support for measuring the frequency of a digital signal applied to a pin. This can be used to get precise measurements from encoders.

Current [implementation](https://github.com/firmata/ConfigurableFirmata/src/Frequency.cpp) supports only one pin at a time, but the protocol supports more.

## Compatible client libraries

- [dotnet/iot](https://github.com/dotnet/iot)

## Tested boards

This feature has been tested on
 * Arduino Uno
 * Arduino Nano
 * Arduino Due

Frequency measuremnt is only possible on pins that identify as "Frequency capable" during capability query. The pin used for frequency measurement must support interrupts, which means only 2 pins are available on AVR based boards (2 and 3). The capability message returns 0x10 for pins supporting frequency measurement.

## Protocol details

The protocol below uses SysEx queries and SysEx responses.

### Attach sensor

Enable with this command (note: The current implementation only supports one pin being monitored at a time).
The pin to use must support interrupts.
The firmware will send one FREQUENCY_SUBCOMMAND_REPORT every sampling interval.
```
0 START_SYSEX                (0xF0)
1 FREQUENCY_COMMAND          (0x7D)
2 FREQUENCY_SUBCOMMAND_QUERY (0x01)
3 pin                        (0-127)
4 interrupt mode             See below
5 sampling interval          (lsb, bits 0-6)
6 sampling interval          (msb, bits 7-13) lsb and msb together give a 14 bit sampling interval, given in milliseconds.
N END_SYSEX                  (0xF7)
```

Interrupt modes are defined below. In most situations, either RISING or FALLING shall be used. When using CHANGE, the reported frequency will be twice the real frequency.
```
INTERRUPT_MODE_DISABLE 0
INTERRUPT_MODE_LOW 1
INTERRUPT_MODE_HIGH 2
INTERRUPT_MODE_RISING 3
INTERRUPT_MODE_FALLING 4
INTERRUPT_MODE_CHANGE 5
```

One immediate response is sent, unless there was an error.

### Report sensor data
The report is sent each time the sampling interval elapses.

Response 
```
0 START_SYSEX                (0xF0)
1 FREQUENCY_COMMAND          (0x7D)
2 FREQUENCY_SUBCOMMAND_REPORT (0x02)
3 pin                        (0-127)
4 Time of measurement        (32 bits as 5 bytes) in milliseconds
5 Current number of ticks    (32 bits as 5 bytes)
N END_SYSEX                  (0xF7)
```

Both 32 bit values are encoded as "packed". 5 bytes are used to send bits 0-6, 7-13, 14-20, 21-27 and 28-31 respectively. The client will typically remember the last measurment and then perform the division "Delta Ticks / Delta Time" to calculate the current frequency.

### Disable reporting
Query :
```
0 START_SYSEX                (0xF0)
1 FREQUENCY_COMMAND          (0x7D)
2 FREQUENCY_SUBCOMMAND_CLEAR (0x00)
3 pin                        (0-127) Use 0x7F to clear all pin interrupts.
N END_SYSEX                  (0xF7)
```

No response.

### Set debouncing period
This message is supported from ConfigurableFirmata Version 3.4.0 / Protocol Version 2.8.
It can be used to set a debouncing period on a frequency pin. Any events that are within the given
timespan to a previous event will be ignored. 

Query:
```
0 START_SYSEX                (0xF0)
1 FREQUENCY_COMMAND          (0x7D)
2 FREQUENCY_SUBCOMMAND_FILTER (0x03)
3 pin                        (0-127)
5 Debouncing period, in us   (32 bits as 5 bytes)
N END_SYSEX                  (0xF7)
```

No response.


