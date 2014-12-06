led strip proposal
===

This proposal is to allow you to control led strip leds via firmata.

An implementation of this proposal is currently available [here](https://github.com/RussTheAerialist/arduino/compare/firmata:configurable_dev...RussTheAerialist:neopixel_strip).

```
// wrapper for led strip initialize function
//  when initialition is finished, firmata sends done response to host
//  see done response below
0  START_SYSEX                  (0xF0)
1  LEDSTRIP_DATA                (0x62)
2  LEDSTRIP_CMD_INIT            (0x01)
3  LEDSTRIP_INIT_PARAM_DATA_PIN 
4  LEDSTRIP_INIT_PARAM_COUNT    (lsb) // Number of pixels on the strand
5  LEDSTRIP_INIT_PARAM_COUNT    (msb)
... Additional Strip specific configuration information
N  END_SYSEX                 (0xF7)
```

For Neopixel, order and speed would be in the additional configuration
information
For WS2801, Clock Pin would be included in the additional configuratio information.

```
// wrapper for led strip set pixel function
0  START_SYSEX                   (0xF0)
1  LEDSTRIP_DATA                 (0x62)
2  LEDSTRIP_CMD_PIXEL            (0x02)
3  LEDSTRIP_PIXEL_PARAM_LOCATION (lsb) // pos of the pixel to change, 0-based
4  LEDSTRIP_PIXEL_PARAM_LOCATION (msb)
5  LEDSTRIP_PIXEL_PARAM_RED      (lsb) // Red value 0-255
6  LEDSTRIP_PIXEL_PARAM_RED      (msb) 
7  LEDSTRIP_PIXEL_PARAM_GREEN    (lsb) // Green value 0-255
8  LEDSTRIP_PIXEL_PARAM_GREEN    (msb) 
9  LEDSTRIP_PIXEL_PARAM_BLUE     (lsb) // Blue value 0-255
10 LEDSTRIP_PIXEL_PARAM_BLUE     (msb) 
11 END_SYSEX                     (0xF7)
```

```
// wrapper for show function (displays the next frame on the leds)
0  START_SYSEX                   (0xF0)
1  LEDSTRIP_DATA                 (0x62)
2  LEDSTRIP_CMD_SHOW             (0x05)
3  END_SYSEX                     (0xF7)
```

```
// wrapper for clear function (sets all pixels to black)
0  START_SYSEX                   (0xF0)
1  LEDSTRIP_DATA                 (0x62)
2  LEDSTRIP_CMD_CLEAR            (0x04)
3  END_SYSEX                     (0xF7)
```

```
// wrapper for set brightness function (scales the brightness of all leds)
//   this is also a response from firmata back to the host computer
//   see query brightness below
0  START_SYSEX                   (0xF0)
1  LEDSTRIP_DATA                 (0x62)
2  LEDSTRIP_CMD_BRIGHTNESS       (0x06)
3  LEDSTRIP_BRIGHTNESS_PARAM_VALUE (lsb) // 0-255
4  LEDSTRIP_BRIGHTNESS_PARAM_VALUE (msb) // 0-255
5  END_SYSEX                     (0xF7)
```

```
// wrapper for query brightness (return the current brightness value)
//   responds with above set brightness to host computer
0  START_SYSEX                   (0xF0)
1  LEDSTRIP_DATA                 (0x62)
2  LEDSTRIP_CMD_BRIGHTNESS       (0x06)
3  END_SYSEX                     (0xF7)
```

```
// done response to host computer
0  START_SYSEX                   (0xF0)
1  LEDSTRIP_DATA                 (0x62)
2  LEDSTRIP_CMD_DONE             (0x07)
3  END_SYSEX                     (0xF7)
```

```
// set frame (send values for all pixels)
//  all pixels set, and then show is called.
//  This is to reduce the number of bytes necessary for updating all of the pixels and showing them
//  useful if the host computer keeps copies of the led values
//  done response is sent after the frame is shown
//  (not currently implemented)
0  START_SYSEX (0xF0)
1  LEDSTRIP_DATA (0x62)
2  LEDSTRIP_CMD_FRAME (0x03)
3  ... (number of pixels * 6 bytes, RGB order, two bytes per channel with lsb, msb ordering)
N  END_SYSEX (0xF7)
```

Strip Specific Information
===

Adafruit Neopixel (and other WS2812 strips)
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
