tone proposal
===

Add ability to call Arduino `tone` and `noTone` functions. For non-Arduino architectures,
similar functions to `tone` and `noTone` would need to be implemented.

添加调用的Arduino`tone`和`noTone`功能的能力。对于非Arduino的架构中，
相似的功能`tone`和`noTone`将需要实施。


The duration could be extended if necessary. Duration could also be optional. If
left out, the user would need to send the NO_TONE command to stop the tone.

持续时间可能需要适当延长。持续时间也可能是可选的。 如果
离开了，用户将需要发送的NO_TONE命令停止了基调。

An implementation of this proposal is currently available

这个建议的实现是当前可用

[here](https://github.com/firmata/arduino/compare/configurable...tone).


```
// wrapper for tone function
0  START_SYSEX        (0xF0)
1  TONE_DATA          (0x5F)
2  TONE               (0x00)
3  pinNumber
4  frequency LSB      (in HZ)
5  frequency MSB      (in HZ) (audible range is 20 HZ to 15,000 HZ so 14 bits is sufficient)
6  duration bits 0-6  (in ms)
7  duration bits 7-14 (in ms) (max duration is 16,383 milliseconds)
11  END_SYSEX (0xF7)
```

```
// wrapper for noTone function
0  START_SYSEX        (0xF0)
1  TONE_DATA          (0x5F)
2  NO_TONE            (0x01)
3  pinNumber
n  END_SYSEX          (0xF7)
```
