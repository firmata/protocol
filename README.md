# Firmata Protocol Documentation

[![Join the chat at https://gitter.im/firmata/protocol](https://badges.gitter.im/firmata/protocol.svg)](https://gitter.im/firmata/protocol?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Firmata is a protocol for communicating with microcontrollers from software on a computer (or smartphone/tablet, etc). The protocol can be implemented in firmware on any microcontroller architecture as well as software on any computer software package (see list of client libraries below).

Firmata is based on the [midi message format](http://www.midi.org/techspecs/midimessages.php) in that commands bytes are 8 bits and data bytes are 7 bits. For example the midi Channel Pressure (Command: 0xD0) message is 2 bytes long, in Firmata the Command 0xD0 is used to enable reporting for a digital port (collection of 8 pins). Both the midi and Firmata versions are 2 bytes long, but the meaning is obviously different. In Firmata, the number of bytes in a message must conform with the corresponding midi message. Midi [System Exclusive](http://www.2writers.com/eddie/tutsysex.htm) (Sysex) messages however, can be any length and are therefore used most prominently throughout the Firmata protocol.

This repository contains documentation of the Firmata protocol. The core of the protocol is described in the [protocol.md file](protocol.md) file. Feature-specific documentation is described in individual markdown files ([i2c.md](i2c.md), [accelStepperFirmata.md](https://github.com/firmata/protocol/blob/master/accelStepperFirmata.md), [servos.md](servos.md), etc). Files added to the proposals directory are proposals for new features that have not yet been finalized. See [feature-registry.md](https://github.com/firmata/protocol/blob/master/feature-registry.md) for the full list of documented firmata features.

The Firmata protocol could theoretically be implemented for any microcontroller platform. Currently however, the most complete implementation is for [Arduino](http://arduino.cc) (including Arduino-compatible microcontrollers). Here are the known Firmata microcontroller platform implementations:

* [Firmata for Arduino](https://github.com/firmata/arduino)
* [Firmata for Spark.io](https://github.com/firmata/spark)


*Please note: I'm sure there are other implementations. If you know of others, please submit a pull request to update this readme file, or open an issue providing the link to be added to this document.*

## Firmata client libraries
There are several client libraries. These are libraries that implement the Firmata protocol in order to communicate (from a computer, smartphone or tablet for example) with Firmata firmware running on a microcontroller platform. The following is a list of Firmata client library implementations:

* processing
  * [https://github.com/firmata/processing]
* python
  * [https://github.com/MrYsLab/pymata4]
  * [https://github.com/MrYsLab/pymata-express]
  * [https://github.com/firmata/pyduino]
  * [https://github.com/lupeke/python-firmata]
  * [https://github.com/tino/pyFirmata]
* perl
  * [https://github.com/ntruchsess/perl-firmata]
  * [https://github.com/rcaputo/rx-firmata]
* ruby
  * [https://github.com/hardbap/firmata]
  * [https://github.com/PlasticLizard/rufinol]
* clojure
  * [https://github.com/nakkaya/clodiuno]
  * [https://github.com/peterschwarz/clj-firmata]
* javascript
  * [https://github.com/jgautier/firmata]
  * [http://breakoutjs.com]
  * [https://github.com/rwldrn/johnny-five]
* java
  * [https://github.com/4ntoine/Firmata]
  * [https://github.com/kurbatov/firmata4j]
  * [https://github.com/reapzor/FiloFirmata]
  * [https://github.com/mattjlewis/diozero/tree/main/diozero-provider-firmata]
* .NET
  * [https://github.com/SolidSoils/Arduino]
  * [https://github.com/dotnet/iot]
* PHP
  * [https://bitbucket.org/ThomasWeinert/carica-firmata]
  * [https://github.com/oasynnoum/phpmake_firmata]
* Haskell
  * [http://hackage.haskell.org/package/hArduino]
* iOS
  * [https://github.com/jacobrosenthal/iosfirmata]
* Dart
  * [https://github.com/nfrancois/firmata]
* Max/MSP
  * [http://www.maxuino.org/]
* Elixir
  * [https://github.com/kfatehi/firmata]
* Modelica
  * [https://www.wolfram.com/system-modeler/libraries/model-plug/]
* golang
  * [https://github.com/kraman/go-firmata]
* Qt/QML
  * [https://github.com/callaa/qfirmata]
* Android/Kotlin
  * [https://github.com/xujiaao/android-firmata]
* Smalltalk
  * [https://github.com/pharo-iot/Firmata] 
* LabVIEW
  * [https://github.com/KMurphs/labview-client-for-firmata] 

*Each client library may not support the most recent version of the Firmata protocol and all features described in this reposity.*

## Contributing

To submit a proposal for a new feature, create a [markdown](https://help.github.com/articles/github-flavored-markdown/) file for your proposal and append "-proposal" to the filename. Submit a pull request to add the proposal.

To make a change to an existing protocol, submit a pull request with your proposed changes. Be sure to provide any rationale in the pull request description.

Some hints for drafting a new proposal:

* See [feature-registry.md](https://github.com/firmata/protocol/blob/master/feature-registry.md) for information on proposing a new feature and requesting a feature ID.
* Use sub-commands (3rd byte) as necessary if you have more than one message. See the [accelStepperFirmata.md](https://github.com/firmata/protocol/blob/master/accelStepperFirmata.md) file for an example. Note the use of `0x62` for the feature ID and how each section has an enumerated set of subcommands (0x00 = config, 0x01 = zero, 02 = step, etc).
* It's okay to have optional values in a sysex message as long as those values are all at the end of the message. See the bytes 6 & 7 of the `SERIAL_CONFIG` message in [serial-1.0.md](https://github.com/firmata/protocol/blob/master/serial-1.0.md)
* Try to keep your sysex messages as short as possible.
* Pack bits if necessary. See the Response message for **Report encoder's position** in [encoder.md](encoder.md) for an example (also note how this was documented following the response message... please include similar documentation if you use bit packing in your proposal).
* If your proposal uses any of the available non-sysex midi messages, the number of bytes in the message must correspond to the number of bytes in the midi message. The meaning however does not need to be the same. However if the midi message uses channels (such as Note Off (0x80)) then the Firmata message must also use channels since a midi parser may expect this.
