# DHT Sensor Feature

Provide support for connecting a DHT sensor like [this one](https://www.sparkfun.com/datasheets/Sensors/Temperature/DHT22.pdf) (also works with DHT11).

This feature is based on based on @niesteszeck's [interrupt driven DHT library](https://github.com/niesteszeck/idDHTLib), so the sensor has to be connected to an interrupt pin.

Current [implementation](https://github.com/mdlima/FirmataDHT) supports only one DHT sensor, but the protocol supports more.

Reporting is not done using Firmata's main reporting cycle. Instead, it's done at its own sampling interval of 2 seconds minimum, as these sensors are very slow.


Example files: 
 * See [SimpleFirmataDHT.ino](https://github.com/mdlima/FirmataDHT/blob/master/examples/SimpleFirmataDHT/SimpleFirmataDHT.ino).

## Compatible client libraries

- [firmata.js](https://github.com/mdlima/firmata.js/tree/dht-sensor)

## Tested boards

This feature has been with a DHT22 sensor on:
 * Arduino Duemilanove
 * Arduino Nano

## Protocol details

The protocol below use exclusively SysEx queries and SysEx responses.

### Attach sensor

Query :
```
0 START_SYSEX                (0xF0)
1 DHTSENSOR_DATA             (0x74)
2 DHTSENSOR_ATTACH_DHT22     (0x02) [for DHT11 sensors use DHTSENSOR_ATTACH_DHT11 (0x01)]
3 pin                        (0-127)
4 blocking reads             (0-1) [optional - enables an 18ms blocking call when starting the read **use with caution**. default: 0 (disabled)]
5 sampling interval          (lsb) [optional - sampling interval to report sensor data (in ms). minimum: 500ms. default: 500ms.]
... additional bytes may be sent if more bits are needed
N END_SYSEX                  (0xF7)
```

No response.

### Report sensor data
No Query, report is done automatically on the requested interval

Response 
```
0 START_SYSEX                (0xF0)
1 DHTSENSOR_DATA             (0x74)
2 DHTSENSOR_RESPONSE         (0x00)
3 pin                        (0-127)
4 temperature, bits 0-6      (lsb - temperature in Celsius * 10)
5 temperature, bits 7-13     (msb - temperature in Celsius * 10)
6 humidity, bits 0-6         (lsb - relative humidity in % * 10)
7 humidity, bits 7-13        (msb - relative humidity in % * 10)
N END_SYSEX                  (0xF7)
```

Note:

Temperature

- Range: -40~80 ºC
- Resolution: 0.1 ºC
- Accuracy: <0.5 ºC

Humidity

- Range: 0-100 %RH
- Resolution: 0.1 %RH
- Accuracy: +-2 %RH (Max +-5 %RH)

### Detach sensor
Query :
```
0 START_SYSEX                (0xF0)
1 DHTSENSOR_DATA             (0x74)
2 DHTSENSOR_DETACH           (0x03)
3 pin                        (0-127)
4 END_SYSEX                  (0xF7)
```

No response.
