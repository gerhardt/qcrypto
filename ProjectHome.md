(This repository will not be maintained anymore - it is in a state of migration to GitHub. At the moment, look for https://github.com/kurtsiefer/qcrypto )

This is the repository for the source code used in a series of quantum key distribution experiments at the National University of Singapore, from key generation in form time-stamped detector data to error correction and privacy amplification.

The code was designed to work with entanglement-based pair detection schemes, either implementing a BB84-type/BBM92 protocol, or an Ekert-91 protocol, which we [(Centre for Quantum Technologies)](http://www.quantumlah.org) used for our implementations (1,2) of quantum key distribution systems.

The code is mostly ANSI C, typically compiled with gcc (4.2 or 4.3, but earlier versions also work). We have run the suite mostly on Linux machines running various distros, haven't tested out anything else because our USB hardware has at the moment only a linux driver.

Discussions on implementation details/problems/leaks are welcome, and should ultimately lead to an openly available implementation of a QKD scheme which hopefully has less open security problems than implementations using proprietary code. We believe that one of the main attack scheme for a quantum key distribution system lies in the software and communication implementation, and would encourage people to try to break it.

We eventually will open hardware specs and detailled designs as well, but please bear with us since this publicization is sort of a side project. Our internal documentation is at the moment somewhat chaotic. But if you have any specific questions, do not hesitate to get in touch with us directly.

In an initial phase, have deposited code which has been running on several linux systems in our labs, and definitions of file formats and hardware.

The currently recommended way of installing it is to draw the complete source tree from [the source repository](http://code.google.com/p/qcrypto/source/checkout) and compile the different elements following the readme instructions.
The code as used in (1) and for a [demo](http://events.ccc.de/congress/2007/Fahrplan/events/2275.en.html) at the 24C3 conference is currently available as a tarball in the download area here as qcrypto\_20071229.tgz. Consolidation for our internal code branches is in progress; the inclusion of all the gui and other components for an implementation of an Ekert91 protocol  is not yet complete.

If you are interested in joining the project, please send an email to the project owner at gmail.com.

# Attacks #
We recently had a visitor, [Vadim Makarov](http://www.vad1.com/) from Norway in our group who had an excellent idea how to break the QKD system. Due to issues we have with the detector hardware, we are a bit in trouble with his attack. We could demonstrate that the quantum key distribution could be completely compromised (5). We demonstrated that at DEFCON 17, and at HAR in 2009.

The attack opens issues way beyond the security of a specific protocol, we even could fake the results of a Bell inequality test (6), which forms the basis of the Ekert-91 protocol or its more quantitative version of device-independent protocols (2). At this point, we feel that either the assumptions going into the security claims of quantum key distribution need to be reconsidered in the sense that it is only given under certain assumptions about devices and eavesdropper abilities (7), or new protocols are developed.

# News #
There are a number of other communication primitives that can be implemented with this scheme - like classical bit commitment under the assumption of limited and noisy quantum storage (8).

# References #
Our published work can be found under

(1) I. Marckic, A. Lamas-Linares, and C. Kurtsiefer, Appl.
Phys. Lett. **89**, 101122 (2006), earlier version at [arXiv:quant-ph/0606072v2](http://arxiv.org/abs/quant-ph/0606072).

(2) Alexander Ling, Matthew P. Peloso, Ivan Marcikic, Valerio Scarani, Antia Lamas-Linares, Christian Kurtsiefer: _Experimental quantum key distribution based on a Bell test_, Phys. Rev. A. **78**, 020301 (R) (2008) [http://arxiv.org/abs/0805.3629](http://arxiv.org/abs/0805.3629).

(3) Matthew P. Peloso, Ilja Gerhardt, Caleb Ho, A. Lamas-Linares, C. Kurtsiefer: _Daylight operation of a free space, entanglement-based quantum key distribution system_.  [New Journal of Physics \*11\*, 045007 (2009)](http://www.iop.org/EJ/abstract/1367-2630/11/4/045007).

(4) Caleb Ho, A. Lamas-Linares, and C. Kurtsiefer: _Clock synchronization by remote detection of correlated photon pairs_.  [New Journal of Physics \*11\*, 045011 (2009)](http://www.iop.org/EJ/abstract/1367-2630/11/4/045011).

(5) I. Gerhardt, Q. Liu, A. Lamas-Linares, J. Skaar, C. Kurtsiefer, and V. Makarov: _Full-field implementation of a perfect eavesdropper on a quantum cryptography system_. Nature Communications, **2**, 349 (2011)

(6) I. Gerhardt, Q. Liu, A. Lamas-Linares, J. Skaar, V. Scarani, V. Makarov, and C. Kurtsiefer: _Experimentally faking the violation of Bell's inequalities_. Phys. Rev. Lett. **107**, 170404 (2011)

(7) V. Scarani, C. Kurtsiefer: _The black paper of quantum cryptography: real implementation problems_. [arXiv:0906:4547](http://arxiv.org/abs/0906.4547). Supposed to be published in a Festschrift for the 25th anniversary of BB84, but that itself may only show up for the 125th anniversary of BB84.

(8) H.Y.N. Ng, S.K. Joshi, C. M. Chia, C. Kurtsiefer, S. Wehner: _Experimental implementation of bit commitment in the noisy-storage model_. Nature Communications **3**, 1326 (2013)

A notoriously outdated web page of the QKD activities in the SG group can be found under http://www.qolah.org/research/crypto/crypto.html