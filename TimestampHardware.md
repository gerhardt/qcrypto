# Introduction #

This is still a temporary, incomplete, unpolished version, but it should supply some basic information on what the timestamp card does before this unit is documented properly.

The timestamp card forms the interface between the physical measurement device and the software by generating a stream of detector event timing data structures, each 64 bit wide and containing timing and detector combination information in a raw format.

Basic elements of the card are a free-running counter clocked with a 500 MHz base frequency, a phase interpolation stage, and an event register able to capture the status of four fast signal pulse input lines following a NIM standard (unimportant here).

Once any of the four input lines (or any combination thereof) changes form logical 0 to logical 1, indicating the advent of a photodetection event, the state of the counter, the state of a phase interpolation stage of the base clock signal, and after some adjustable time the state of the four input lines is sampled, and stored as a 64 bit word in a FIFO in the device.

Once the USB transfer engine is started, the content of that FIFO is handed over to the host computer in the raw sampling format.

Various commands can be sent to the card on a low bandwidth channel, most of them are of the flavour of initializing counters, adjusting input threshold values and adjusting a few internal delays and skew times, and clock selection. There is also built-in internal calibration hardware to characterize the phase interpolation stage. More details on various functionalities can be found in the USB engine firmware.

While the counter has a time resolution of 2ns, effective 4 additional bits are extracted using a phase measurement stage to arrive at nominally 125ps time resolution. The decoding of a 9-bit phase interpolation pattern, sampled with each event, into a high resolution time is currently performed by host software (specifically, in the _readevents_ program).

# Inputs to the timestamp card #
Besides the four signal input lines for the four detectors, the card has an optional input for a 10 MHz reference clock, from which the internal 500 MHz system clock is derived via a PLL. This reference clock input essentially defines the time base for all timing measurements.

# Output data format #
Format of a 64 bit unit for the high bandwidth data for each detection event is, motivated by the original 32 bit design, formulated as the content of two sequential uint32 elements (although it does not show as such in the code):

```
struct dataelement {
    unsigned int cv; /* coarse value */
    unsigned int dv; /* fine value / detector info / phase pattern */
};
```

The internal counter is segmented in a 5 bit wide fast counter running at 500 MHz, and a 40 bit wide slow counter, running at 500 MHz / 2<sup>5</sup> = 15.625 MHz.

## Bit values: ##
| cv[31:0] | most significant bits c44..c13 of the sampled counter state|
|:---------|:-----------------------------------------------------------|
| dv[31:24] | bits c12..c5 of the sampled counter state|
| dv[23:18] | not used, read status undefined |
| dv[17 ](.md) | sampled state of detector line C |
| dv[16 ](.md) | sampled state of detector line D |
| dv[15 ](.md) | sampled state of detector line A |
| dv[14 ](.md) | sampled state of detector line B |
| dv[13:9] | bits c4..c0 of the sampled counter state|
| dv[8:0] | phase interpolation sample state |

.

# Known issues #
Due to some polarity errors in clock handover form the fast counter to the slow counter, there is a correction necessary to see a monotoneous increase in the reported time with real time of events.

The phase interpolation stage does currently not lead to an equidistant distribution of time intervals spaced at 125ps. The effective time resolution is about 320-300ps, with some intervals on some cards not being resolved better than 400ps.

For one of the phase patterns, there is an ambiguity between the the main counter having made the transition from one state to the other. This effectively leads to time readings which can be 2 ns off for this particulatr phase pattern. Cure to this problem is clear, but not yet implemented in the hardware we have. Consequence of this problem is a small fraction of satellite timing peaks 2ns apart in correlation functions.

# Hardware availability #
We think of opensourcing the whole hardware description, but the bottleneck for that is the time it takes me to tidy up documentation for it. We currently provide circuit diagrams [in pdf form](http://code.google.com/p/qcrypto/source/browse/trunk/hardware/electronics), and the [complete firmware](http://code.google.com/p/qcrypto/source/browse/trunk/hardware/timestampfirmware/) in the USB interface of our timestamp card. While mostly ready for an outsourced manufacturing process, we are still not there yet (yet improving), and it still takes me about a day to build one of those, assuming the parts are ready. If you are interested in making/obtaining one, feel free to contact me. Absence of documentation is not a question of not wanting to publish it, but lack of time in documenting stuff.