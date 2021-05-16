## Version 2.6.0 - September 16th, 2017

- Added AccelStepperFirmata (Stepper 2.0) for improved and more scalable stepper motor support.
- Deprecated the old Stepper protocol, now renamed to "stepper-legacy.md".

## Version 2.5.1 - December 21st, 2015

- Enable I2C auto-restart by setting [bit 6 of byte 3](https://github.com/firmata/protocol/commit/22cc239b5a527556d82707fe0c540b16ed42f0bc#diff-ab7ff1e563b418ee1e557d6ece901dc7R17) of the `I2C_REQUEST` message.

## Version 2.5.0 - November 7th, 2015

- Added [Serial feature](https://github.com/firmata/protocol/blob/master/serial-1.0.md) for interfacing with serial devices via hardware or software serial.
- Added ability to set the value of a pin by sending a single pin value instead of a port value. See 'set digital pin value' in [protocol.md](https://github.com/firmata/protocol/blob/master/protocol.md#message-types) for details.

## Version 2.4.0 - December 2014

- Changed `report digital port` and `report analog pin` definition to return the port (digital) or pin (analog) value upon toggling to `enable`.
- Added [OneWire feature](https://github.com/firmata/protocol/blob/master/onewire.md) to interface with 1-Wire devices.
- Added [Encoder feature](https://github.com/firmata/protocol/blob/master/encoder.md) to interface with linear and rotary encoders.
- Added [Scheduler feature](https://github.com/firmata/protocol/blob/master/scheduler.md) to enable scheduling Firmata tasks. Useful when you need to send more data than the 64 byte serial buffer can handle.
- Added [Stepper feature](https://github.com/firmata/protocol/blob/master/stepper.md) to enable interfacing with 2 wire and 4 wire stepper motor drivers and step + direction drivers.

*Note: The above 4 features were initially added for `ConfigurableFirmata` which had a different version number. They have been moved under the `v2.4.0` release here to get things back on track for the protocol version.*

## Version 2.3.0 - February 2013

- Angle was removed from the `SERVO_CONFIG` message.

## Version 2.2.0 - January 2011

- Added [Extended Analog](https://github.com/firmata/protocol/blob/master/protocol.md#extended-analog) to allow addressing beyond pin 15 and support analog values with any number of bits.
- Added [Capability Query](https://github.com/firmata/protocol/blob/master/protocol.md#capability-query) to query the capabilities supported by each pin.
- Added [Analog Mapping Query](https://github.com/firmata/protocol/blob/master/protocol.md#analog-mapping-query) to map analog pin numbers to their digital pin number equivalent.
- Added [Pin State Query](https://github.com/firmata/protocol/blob/master/protocol.md#pin-state-query) to query the state of pin (output value or if input pullup enabled).

## Version 2.1.0 - March 2010

- Added [I2C feature](https://github.com/firmata/protocol/blob/master/i2c.md) to interface with I2C devices.
- Added [Servo feature](https://github.com/firmata/protocol/blob/master/servos.md) to interface with servo motors.
- Added ability to change the [sampling interval](https://github.com/firmata/protocol/blob/master/protocol.md#sampling-interval).

## Version 2.0.0 - September 2008

- Changed to 8-bit port-based digital messages (a collection of 8 pins) to mirror ports from previous 14-bit ports (a collection of 14 pins) modeled after the standard (ATmega8/168/328) Arduino boards.
- Added ability to [query firmware name and version](https://github.com/firmata/protocol/blob/master/protocol.md#query-firmware-name-and-version).

## Version 1.0.0

- Switched to MIDI-compatible packet format (though the message interpretation differs).
