# Required packages #
Various elements of the code, in particular the GUI, require various packages to be loaded which may not be part of a standard distribution.  The necessary or recommended packages are:
  * **fftw3 library:** This is necessary to implement the initial tiing difference finding.
  * **fftw3-devel:** This package is required for compiling the time difference finding code
  * **ntpd:** The two sides require a system time on the two machines which is not further apart than half an epoch or about 250msec.
  * **kernel-source:** Necessary for compiling a kernel module for talking to the timestamp card.
  * **gcc and make:** developing environment
  * **tcl:** required for running the GUI
  * **tk:** tcl extension required for running the GUI
  * **gnuplot:** required for running the GUI
  * **svn:** necessary for acessing the source tree on this server.


# Installation on our common systems #
On most of our machines, we currently use a flavour of SuSE Linux to run the code, currently  openSUSE 10.3 and 11.0 on laptops (lenovo x60s). By now, the software suite works well up to openSUSE 12.2. both on i586 and x66\_64 hardware.

  * At the moment, the recommended installation procedure starts with generation of a common program directory _somename_ somewhere in user space. We use _programs_ for _somename_, so if there are problems with some other names be prepared for residual hardcoded namings.
  * The code tree then is installed into this dirctory with svn using the command....
  * Prepare the timestamp card driver. Therefore, cd into the _usbtimetagdriver_ directory and follow the steps in the README file
  * Prepare the timestamp card user space program. For that, cd into the _timestamp3_ directory. A simple _make_ should do the job.
  * Prepare the error correction deamon; for that, cd into the _errorcorrection_ directory and issue a _make_.
  * Prepare the rest of the code. For that, cd into the _remotecrypto_ directory and issue a _make_ command.
  * Make sure you open a TCP/IP port in a firewall (so existent) on both machines for the classical communication part. By default, this is port 4852.
  * Edit the localparams file for the local machine name, and the remote IP address. Here, also various detector delay constants are defined.

A successful implementation was also performed on OLPCs running under its native OS/interface combination Debian/Sugar. A homogenous hardware infrastructure is not necessaru per se, although most of the time we do that.