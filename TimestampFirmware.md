# Introduction #

Here should go some description for the firmware in a USB engine, a Cypress 68013 (or recently: 68013A) EZ-USB microcontroller with a fast USB serialization engine.

The microcontroller is using a 16 bit wide parallel transfer from the main event FIFO on 16 input lines. As we use a 56pin version of that chip, we have to share these lines occasionally as output lines to talk to an on-board DAC and to the 500MHz synthesizer.

The code is currently residing in a serial EEPROM as part of the timestamp unit.

We shamelessly abused the Cypress Vendor ID with a bogous product ID before we will have a proper one.