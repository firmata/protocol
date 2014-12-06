neopixel proposal
===

This proposal is to allow you to control neopixel leds via firmata.

An implementation of this proposal is currently available [here](https://github.com/RussTheAerialist/arduino/compare/firmata:configurable_dev...RussTheAerialist:neopixel_strip).

```
// wrapper for neopixel initialize function
//  when initialition is finished, firmata sends done response to host
//  see done response below
0  START_SYSEX               (0xF0)
1  NEOPIXEL_DATA             (0x62)
2  NEOPIXEL_CMD_INIT         (0x01)
3  NEOPIXEL_INIT_PARAM_PIN 
4  NEOPIXEL_INIT_PARAM_COUNT (lsb) // Number of pixels on the strand
5  NEOPIXEL_INIT_PARAM_COUNT (msb)
6  NEOPIXEL_INIT_ORDER       (see below)
7  NEOPIXEL_INIT_SPEED       (see below)
8  END_SYSEX                 (0xF7)
```

```
// wrapper for neopixel set pixel function
0  START_SYSEX                   (0xF0)
1  NEOPIXEL_DATA                 (0x62)
2  NEOPIXEL_CMD_PIXEL            (0x02)
3  NEOPIXEL_PIXEL_PARAM_LOCATION (lsb) // pos of the pixel to change, 0-based
4  NEOPIXEL_PIXEL_PARAM_LOCATION (msb)
5  NEOPIXEL_PIXEL_PARAM_RED      (lsb) // Red value 0-255
6  NEOPIXEL_PIXEL_PARAM_RED      (msb) 
7  NEOPIXEL_PIXEL_PARAM_GREEN    (lsb) // Green value 0-255
8  NEOPIXEL_PIXEL_PARAM_GREEN    (msb) 
9  NEOPIXEL_PIXEL_PARAM_BLUE     (lsb) // Blue value 0-255
10 NEOPIXEL_PIXEL_PARAM_BLUE     (msb) 
11 END_SYSEX                     (0xF7)
```

```
// wrapper for show function (displays the next frame on the leds)
0  START_SYSEX                   (0xF0)
1  NEOPIXEL_DATA                 (0x62)
2  NEOPIXEL_CMD_SHOW             (0x05)
3  END_SYSEX                     (0xF7)
```

```
// wrapper for clear function (sets all pixels to black)
0  START_SYSEX                   (0xF0)
1  NEOPIXEL_DATA                 (0x62)
2  NEOPIXEL_CMD_CLEAR            (0x04)
3  END_SYSEX                     (0xF7)
```

```
// wrapper for set brightness function (scales the brightness of all leds)
//   this is also a response from firmata back to the host computer
//   see query brightness below
0  START_SYSEX                   (0xF0)
1  NEOPIXEL_DATA                 (0x62)
2  NEOPIXEL_CMD_BRIGHTNESS       (0x06)
3  NEOPIXEL_BRIGHTNESS_PARAM_VALUE (lsb) // 0-255
4  NEOPIXEL_BRIGHTNESS_PARAM_VALUE (msb) // 0-255
5  END_SYSEX                     (0xF7)
```

```
// wrapper for query brightness (return the current brightness value)
//   responds with above set brightness to host computer
0  START_SYSEX                   (0xF0)
1  NEOPIXEL_DATA                 (0x62)
2  NEOPIXEL_CMD_BRIGHTNESS       (0x06)
3  END_SYSEX                     (0xF7)
```

```
// done response to host computer
0  START_SYSEX                   (0xF0)
1  NEOPIXEL_DATA                 (0x62)
2  NEOPIXEL_CMD_DONE             (0x07)
3  END_SYSEX                     (0xF7)
```

Order and Speed
---

Order and Speed are from the Adafruit Neopixel library.

```
// Order Values for the order of the rgb bytes
NEO_RGB (0x00)
NEO_GRB (0x01)
NEO_BRG (0x04)
```

```
// Speed, either 400Khz (trinket, slower processors) or 800Khz (normal)
NEO_KHZ400 (0x00)
NEO_KHZ800 (0x02)
```
