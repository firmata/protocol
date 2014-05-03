RCSwitch proposal
=================

[RCSwitch](http://code.google.com/p/rc-switch/) is a library to send and receive messages to/from radio controlled devices. Sender and receiver are referred to as *devices* within the context of this proposal. Multiple devices may be used at the same time; the only requirement is a pin per device. All devices may be used and configured independently. Thus, this proposal separates the main functions *send* and *receive*.

Send
----

```
PINMODE_RC_TRANSMIT     0x0A


// configuration of sender pin: set protocol
0  START_SYSEX
1  RC_DATA          (0x66)
2  senderPin        
3  CONFIG_PROTOCOL  (0x11)
4,5 protocol (int)
6 END_SYSEX

// configuration of sender pin: set pulse length
0  START_SYSEX
1  RC_DATA             (0x66)
2  senderPin        
3  CONFIG_PULSE_LENGTH (0x12)
4,5 pulseLength (int)
6 END_SYSEX

// configuration of sender pin: set repeat transmit
0  START_SYSEX
1  RC_DATA             (0x66)
2  senderPin        
3  CONFIG_PULSE_LENGTH (0x14)
4,5 repeatTransmit (int)
6 END_SYSEX

// send code as tristate 
0  START_SYSEX
1  RC_DATA          (0x66)
2  senderPin        
3  CODE_TRISTATE    (0x21)
4..n RC data in tristate format (packed as 7-bit)
n+1 END_SYSEX

// send code as long
0  START_SYSEX
1  RC_DATA          (0x66)
2  senderPin        
3  CODE_LONG        (0x22)
4..n RC data (long) (packed as 7-bit)
n+1 END_SYSEX

// send code as char[]
0  START_SYSEX
1  RC_DATA          (0x66)
2  senderPin        
3  CODE_CHAR        (0x24)
4..n RC data (char[]) (packed as 7-bit)
n+1 END_SYSEX
```

Receive
-------

```
PINMODE_RC_RECEIVE     0x0B

// configuration of receiver pin: set receive tolerance (in percent)
0 START_SYSEX
1 RC_DATA           (0x66)
2 receiverPin        
3 CONFIG_TOLERANCE  (0x31)
4 tolerance
5 END_SYSEX

// receive message
0 START_SYSEX
1 RC_DATA             (0x66)
2 receiverPin        
3 MESSAGE             (0x41)
4,5,6,7 value (long)
8,9 bitlength (int)
10,11 delay (int) 
12,13 protocol (int)
14 END_SYSEX
```

Tristate bits
-------------
RCSwitch supports - besides the types long and char[] - so-called *tristate* bits. A tristate bit has one of the values 0, 1, or F. Each tristate bit is coded as 2 bits as follows:
```
TRISTATE_0              0x00
TRISTATE_F              0x01
TRISTATE_RESERVED       0x02
TRISTATE_1              0x03
```
Thus, 1 byte consisting of 8 bits ABCDEFGH may hold up to 4 tristate bits AB, CD, EF and GH. The leftmost 2 bits represent the first tristate bit, the rightmost 2 bits represent the fourth tristate bit. If less than 4 tristate bits are used, the byte is filled with the unused value 0x02.

