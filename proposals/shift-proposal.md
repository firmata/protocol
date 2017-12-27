shift in/out proposal
===

There are a few different ways to approach shift in/out support. It's complicated
since different hardware handles shift in/out in different ways. For example,
not all hardware requires a latch pin and those that use some sort of a latch
don't always use it the same way.

There has also been some discussion around supporting fractional byte devices. The proposals below do not include such functionality. I'm not sure how popular
such devices are. If someone has a proposal that includes support for shifting 
fractional bytes, please submit a pull request to add the proposal to this document.

Propoasl A: shift in/out with no config or latch support
---

In this version it the user must configure the pin (input / output) separately.
If the hardware they are using requires a latch, the latch pin must be managed
separately.

```
// shift out
0  START_SYSEX
1  SHIFT_DATA          (0x75)
2  SHIFT_OUT           (0x01)
3  dataPin
4  clockPin
5  bitOrder            (MSBFIRST or LSBFIRST)
n ...                  (shift out data)
n+1 END_SYSEX

// shift in (for client application to request shift-in data from microcontroller)
0  START_SYSEX
1  SHIFT_DATA          (0x75)
2  SHIFT_IN            (0x02)
3  dataPin
4  clockPin
5  bitOrder            (MSBFIRST or LSBFIRST)
6  numBytes            (number of bytes to shift in. Default to 1)
7 END_SYSEX

// shift in reply (for sending shift-in data to client application)
0  START_SYSEX
1  SHIFT_DATA          (0x75)
2  SHIFT_IN_REPLY      (0x03)
3  dataPin             (so you know which data pin the reply corresponds to)
n ...                  (shift in data)
n+1 END_SYSEX
```


Proposal B: shift in/out with config and latch support
---

The advantages with this version over the one above is that pin modes are handled by the implementation (in the other version you have to handle them manually). You also send fewer bytes when shifting out or in data (since only have to specify clock, bitOrder and optional latch pin once when sending the config). The disadvantage is that memory must be allocated to store pin information.

Another advantage with this version is that you can rely on the firmware to do
more of the work. For example you can shift in multiple bytes at a time and send them to the client in a single packet rather than a single byte at at time (if
your hardware requires a latch/load pin).

This version uses a SHIFT_CONFIG command to set the clock pin, bitOrder and optional latchPin (or load for some shift-in hardware). There are a few different shift types / latch configurations:

```
// shift types (specified in bits 3:5 of byte 2)
SHIFT_OUT                     // shift out with no latch
SHIFT_IN                      // shift out with no latch/load
LATCH_L_SHIFT_OUT             // set latch low then shift out
LATCH_L_SHIFT_OUT_LATCH_H     // set latch low, shift out, then set latch high
SHIFT_OUT_LATCH_H             // shift out then set latch high
TOGGLE_LOAD_SHIFT_IN          // toggle load pin low, then high and shift in
```

The protocol is as follows:
```
// shift config (send when instantiating a new shift-based hardware module)
0  START_SYSEX
1  SHIFT_DATA    (0x75)
2  SHIFT_CONFIG  (bits 0:2 shift command, bits 3:5 shift type, bit 6 unused)
3  dataPin
4  clockPin
5  bitOrder
6  latchPin      [optional]
7 END_SYSEX

// shift out
0  START_SYSEX
1  SHIFT_DATA    (0x75)
2  SHIFT_OUT     (bits 0:2 shift command, bits 3:5 shift type, bit 6 unused)
3  dataPin
n  ...           (shift out data)
n+1 END_SYSEX

// shift in
0  START_SYSEX
1  SHIFT_DATA    (0x75)
2  SHIFT_IN      (bits 0:2 shift command, bits 3:5 shift type, bit 6 unused)
3  dataPin
4  numBytes      (number of bytes to shift in. Default to 1)
5  END_SYSEX
```