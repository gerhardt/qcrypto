To Do : links to other local wiki pages, references
# Introduction #
There is tons of good explanations out in web space what quantum key distribution is about, what is promises, what the physics behind that is and all of that. You will find nothing of that in these pages.

This is the repository for the source code used in quantum key distribution in a particular implementation, from key generation in from time-stamped photodetector data to error correction and privacy amplification. This code is applicable for cases where measruements are carried out on photon pairs (measurement-based QKD schemes) as opposed to prepare-and-send schemes (like the original BB84 and its derivatives).

If you want to implement the complete scheme yourself, you would need to build (or buy) a entangled photon pair source (not too much effort anymore), and a single photon sensitive detector system capable to measure polarization of photons in two orthogonal bases (e.g. one basis: horizontal/vertical,  the other basis left/right circular, or +/- 45degree linear).

The code was designed to work with entanglement-based pair detection schemes, either implementing a BB84-type protocol, or an Ekert-91 protocol, which we (Centre for Quantum Technologies at the National University of Singapore) used for our implementations.

The code is mostly ANSI C, typically compiled with current (2007) gcc versions.

Discussions on implementation details/problems/leaks are highly welcome, and should ultimately lead to an openly available implementation of a QKD scheme which hopefully has less open security problems than implementations using proprietary code.

At the moment, we have deposited code which has been running on vsrious linux systems in our labs, and definitions of file formats (needs to be transferred into wiki) and hardware. We have graphical meterial in flowcharts, but I need to find out how to include that here.

Also probably helpful: A set of measurement data files to be able to play with the code, try attack strategies etc. This is quite space consuming, we need to host that off-site.

(to come: hardware description, circuits, flow diagrams)

# Communication structure #
Explanation what data goes where, how the information flow is in our qkd scenario. Reference to file format wiki page.

There is a vast amount of communication necessary to identify those corresponding photon pairs out of a sea of noise which eventually lead to a useful key. The layout of the data flow tries to generate as little communication as possible, and encode it in a reasonably efficient way both in terms of size and packing/unpacking effort.

All communication takes place asynchronously to avoid problems with communication latency, and to eventually optimize packaged processing optimization by processor cache utilization. Once a photdetection event has entered the primary fifo in the hardware, the system should not rely on a certain reaction time.

All communication is tied to a time structure of photodetection events, which allows to identify when a certain measurement leading to key data was generated. See also the summary on timing conventions used (to be written).

# Code structure #
Various communication and processing tasks are distributed to a various pieces of code, which can be run and modified independently. Communication between different idependent program snippets is established either via pipes, or files in directories with are written, read or deleted by various programs.  This allows a segmented testing of various components, and a simple way of documenting the whole data flow. It is lean on storage and disk read/write requirements if a reasonably modern cache system on the temporary storage space is in place. Hardly any file makes it onto a physical medium in 'normal' operation.

The different pieces of code are:
  * **readevents** : The front end program interfacing the time stamp unit. Generates a stream of detection events, each encoding what detector or detector combination (in a 4 bit space) fired at which time. Each event has a timing resolution of nominal 125ps to match currently (2000-2008) available standard silicon avalanche photodiodes with a timing uncertainty around 300-700ps. Time is reported with 49 bit resolution, a rollover takes place after about 19 hours. This program ties at the moment to a USB device driver, and used ot be connected to a Nudaq-PCI7200 parallel input card.
  * **chopper**: Segments the raw timing/detector data on ones side into information which is transmitted (timing, measurement basis), and stores the private part of the information locally (into a **t3 file**). The timing information is brought to an "absolute" notation, and extended to a 64 bit time, leading to a rollover after about 76 years. Partitions the timing information into chunks of epochs (2<sup>29</sup> nanoseconds or 2<sup>32</sup> elementary time steps), and performs transport compression into **t1 files**.
  * **chopper2**: Segments the raw timing/detector data on the other side, but does no compression since the main data reduction process in the coincidence identification will be processed on this side.
  * **pfind**: This piece of code is used to find an initial time difference netween the reference clocks or timing units on both sides. It performs a cross-correlation on the detector timing events for that purpose on data collected for a few seconds. This piece of code currently requires local clocks with a relative frequency difference on the order of 10<sup>-9</sup>. Returns a time difference within a range of 2<sup>29</sup>ns, and a confidence estimator to  evaluate the success of the correlation finding on a short and long time scale.
  * **costream**: the piece of code which in a steady-state operation handles the coincidence identification to reject photoevents which do not correspond to matched pairs,  servoes the long-term relative clock drift between the two sides due to frequency differences, and generates message packets (**t4 files**) to the other side about successful photon pair identifications, possibly a matching measurement base, depending on the implemented protocol. Leaves behind locally a **t3 type** raw key file, and generates various monitoring output streams (coincidence rates, accidental coincidences, strange pair events etc).
  * **splicer**: Code on the side which was transmitting its timing information (by running **chopper**) which joins the response from the coincidence identification in form of **t4 files**) and the locally kept private information about the initial measurement results, stored in a **t3 file**. Generates the raw key on this side into a corresponding **t3 file**.
  * **transferd**: All classical communication gets currently transported through a single TCP port. To coordinte this bidirectional communication, which includes transfer of files, error correction messages and shrot plaintext ASCII messages between the controlling framework, an identical copy of **transferd** runs in the background on each side, and gets instructed to transfer files via command pipes from various other programs. This would be the place to put a classical communication authentication in.
  * **ecd2**: This is the error correction code, running on each side and digesting correspondig groups of raw key packets for error correction and privacy amplification. This code implements more or less a CASCADE/Binary error correction scheme and some continuous sevoing/error estimation to optimize error correction comunication. It keeps track of the revealed bits in that communication, and performs the compression of the erro r-free key into a finally, hopefully secure key by estimating the knowledge of an eavesdropper in ways depending on the implenented protocol.
  * **crgui\_ec**: An insanely messy glue script with a graphical surface to tie all components together and to allow a halfway structured parameter choice, and easy alignment of the communication channel.
  * Various helper programs

# Code availability #
Code is available as a tarball in a historical (but working!) version, and a svn containing reasonably consolidated code in use in our labs right now.

# References #
(1) General introduction to quantum key distribution

(2) CASCADE scheme, sugimoto implement