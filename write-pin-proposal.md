Write single digital pin proposal
===

Currently, setting HIGH (1) or LOW (0) values to digital pins requires writing a full port range of 8 pins at a time using 4 bytes.  While this can be efficient in some use cases, it also causes a number of problems.

Writing to a full port uncessarily makes the firmata protocol stateful.  If for example, I want to write to pin 5, I have to keep track of the current values of pins 0 through 7 and send a message to write the wanted values of all 8 pins.

Writing to a full port also makes the assumptions that the firmata-capable device has a multiple of 8 pins. Some smaller devices may have only a few addressable pins.


Proposal: Use the `0xF5` 3 byte command to set the digital value of a single digital pin.
---


Set digital pin value
```
0  set digital pin value (0xF5) (MIDI Undefined)
1  set digital pin number (0-127)
2  value LOW or HIGH (0/1)
```

