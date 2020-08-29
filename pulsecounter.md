# Pulsecounter Feature

Provides pulse counting with a total number of four counters for each pin:
* `cnt_pulse`: Number of valid pulses detected
* `cnt_longPulse`: Number of too long pulses
* `cnt_shortPulse`: Number of too short pulses
* `cnt_shortPause`: Number of too short pauses before a pulse

A valid pulse length is defined by two times:
* `minPulseLength_us`: Minimal pulse length in micro seconds
* `maxPulseLength_us`: Maximal pulse length in micro seconds

A third time defines the minmal pause length before a pulse. This is
* `minPauseBefore_us`

A pulse is valid if it's length is within the min/max boundaries and the state of the pin was stable for at least `minPauseBefore_us` before the beginning of the pulse.

Example implementation : 
 * Pulsecounter feature is available [here](https://github.com/tedeler/FirmataPulseCounter) 
 * A dedicated example is available. See [here](https://github.com/tedeler/FirmataPulseCounter/blob/master/example/main.cpp). 

## Tested boards
This feature has been tested on :
 * Wemos with SAMD21

## Protocol details
The protocol below use exclusively SysEx queries and SysEx responses.

### Attach encoder query
Query :
```c
 /* -----------------------------------------------------
 * 0 START_SYSEX                (0xF0)
 * 1 PULSECOUNTER_DATA          (0x63)
 * 2 PULSECOUNTER_ATTACH        (0x00)
 * 3 pulsecntID                 ([0 - MAXPULSECOUNTER-1])
 * 4 pin                        (pin) 
 * 5 polarity                   (0=>Active LOW, 1=>Active HIGH)

 * 6 minPauseBefore_us          (Bit 27-21)
 * 7 minPauseBefore_us          (Bit 20-14)
 * 8 minPauseBefore_us          (Bit 13-7)
 * 9 minPauseBefore_us          (Bit 6-0)

 * 10 minPulseLength_us         (Bit 27-21)
 * 11 minPulseLength_us         (Bit 20-14)
 * 12 minPulseLength_us         (Bit 13-7)
 * 13 minPulseLength_us         (Bit 6-0)

 * 14 maxPulseLength_us         (Bit 27-21)
 * 15 maxPulseLength_us         (Bit 20-14)
 * 16 maxPulseLength_us         (Bit 13-7)
 * 17 maxPulseLength_us         (Bit 6-0)
 * 18 END_SYSEX                 (0xF7)
 * -----------------------------------------------------
 */
```
No response.

### Report of counter values 
Sent on request or if any counter changes
Format :
```c
 /* -----------------------------------------------------
 * 0 START_SYSEX                (0xF0)
 * 1 PULSECOUNTER_DATA          (0x63)
 * 2 pulsecntID                 ([0 - MAXPULSECOUNTER-1])

 * 3 cnt_shortPause             (Bit 27-21)
 * 4 cnt_shortPause             (Bit 20-14)
 * 5 cnt_shortPause             (Bit 13-7)
 * 6 cnt_shortPause             (Bit 6-0)

 * 7 cnt_shortPulse             (Bit 27-21)
 * 8 cnt_shortPulse             (Bit 20-14)
 * 9 cnt_shortPulse             (Bit 13-7)
 * 10 cnt_shortPulse            (Bit 6-0)

 * 11 cnt_longPulse             (Bit 27-21)
 * 12 cnt_longPulse             (Bit 20-14)
 * 13 cnt_longPulse             (Bit 13-7)
 * 14 cnt_longPulse             (Bit 6-0)

 * 15 cnt_pulse                 (Bit 27-21)
 * 16 cnt_pulse                 (Bit 20-14)
 * 17 cnt_pulse                 (Bit 13-7)
 * 18 cnt_pulse                 (Bit 6-0)

 * 19 END_SYSEX                 (0xF7)
 * -----------------------------------------------------
 */
```


### Reset of all counters
Query :
```c
 /* -----------------------------------------------------
 * 0 START_SYSEX                (0xF0)
 * 1 PULSECOUNTER_DATA          (0x63)
 * 2 PULSECOUNTER_RESET_COUNTER (0x02)
 * 3 pulsecntID                 ([0 - MAXPULSECOUNTER-1])
 * 4 END_SYSEX                  (0xF7)
 * -----------------------------------------------------
 */
```
Response is the counter report (PULSECOUNTER_DATA)

### Request report of counter values
Query :
```c
 /* -----------------------------------------------------
 * 0 START_SYSEX                (0xF0)
 * 1 PULSECOUNTER_DATA          (0x63)
 * 2 PULSECOUNTER_REPORT        (0x01)
 * 3 pulsecntID                 ([0 - MAXPULSECOUNTER-1])
 * 4 END_SYSEX                  (0xF7)
 * -----------------------------------------------------
 */
```
Response is the counter report (PULSECOUNTER_DATA)

### Detach pulse counter
Query :
```c
 /* -----------------------------------------------------
 * 0 START_SYSEX                (0xF0)
 * 1 PULSECOUNTER_DATA          (0x63)
 * 2 PULSECOUNTER_RESET_COUNTER (0x02)
 * 3 pulsecntID                 ([0 - MAXPULSECOUNTER-1])
 * 4 END_SYSEX                  (0xF7)
 * -----------------------------------------------------
 */
```
No response.
