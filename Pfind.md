# Introduction #

A critical task in measurement-based quantum key distribution is to identify photodetection events from the same birth process, leading to the correlated measurement results necessary to bulid up key.

That physical reason apart, the task is to find correlated photodetection events on two remote sites, where the signal is immersed in a huge number of photodetection events which are not connected with photon pairs. We identify them by their remote temporal coincidence in the costream code, and for that process, an initial time difference between the two sides needs to be known, preferrably with an accuracy of about one nanosecond.

The two sides use normal NTP clock synchronization between the CPUs, but that protocol does not easily allow for synchronization down to that accuracy. We therefore need an algorithm to find this time difference.

# Algorithm #
The time difference finding is based on the evaluation of a cross-correlation function between the two detector timing streams, which show a pronounced peak at the clock searched difference. Noisy background events on both sides are typically uncorrelated in time, and lead to a homogenous background.

The code evaluates the cross correlation function _ccf(tau)_ via three Fourier transformations via
> _ccf(tau)=F<sup>-1</sup>[F[d<sub>1</sub>(t)] x F<sup>*</sup>[d<sub>2</sub>(t)]_ ,
and finds the peak loaction in _ccf(tau)_. The time resolution necessary is 1ns, the normal NTP/PC-based timing uncertainty was assumed to be about +/-250 ms. This would require the evaluation of _ccf(tau)_ for about 10<sup>8</sup> to 10<sup>9</sup> entries, which is an unrealistically complex task for the PCs used for QKD.

We therefore split up the difference finding in a coarse and fine timing part, so the necessary 27..30 bit of timing information gets distributed into two timing finding problems with only 14..16 bits or 10<sup>4</sup> to 10<sup>5</sup> elements in the Fourier transformations each.

Since the signal/noise ratio in the peak finding problem for a given pair and background event rate decreases with smaller Fourier ensembles, we choose two overlapping time scales with 17 to 20 bit each to obtain the difference with an acceptable confidence. Fourier transformations of that size are easily managable even on very light CPU systems.

Further segmentation in smaller and smaller problems to perhaps reflect the spirit of the Cooley-Turkey FFT idea  did not work out in practice, since the background noise renders the peak finding problem unreliable.

# Implementation of the algorithm #
The code in _pfind.c_ is written to concatenate a few packets of timing event data generated on the two sides into the respective file formats, and extract the time difference with a resolution of by default 2 nsec.

In a first step, the event timing data is cast into two discrete arrays for each side, one for obtaiing coarse timing information with a resolution of 2048 ns, and one for a fine timing resolution of 2 ns. This conversion visits each timing event only once, and increments the corresponding array element. Timing information is converted into an array index by trunking/shifting of the full-reslution time information. Before the arrays are filled, they get initialized with a negative number removing the DC bias in the fianl correlation array. This is intended to avoid any numerical saturation in the FFT process.

In a next step, the Fourier transformation is carried out using the fftw3 library. The two transformations are then conjucated/multiplied to obtain the FFT of the cross correlation function, and then transformed back to the with the according fftw3 ( http://www.fftw.org/   ) call.

The back-transformed array is then searched for its maximum. In the same loop, the mean (which should be zero due to the pre-loading of the original arrays) and the variance are determined to estimate with which statistical significance the maximum in the cross correlation function exceeds the offset due to the background level. This number serves as a good criterion for identifying the coincidences.

Finally, the time difference with its full resolution is obtained by merging the relevant bits of maximum positions of the ccf both for the fine and coarse discretization time. The time difference, together with the statistical significance of the peaks found for coarse and fine resolution are then passed on to feed the coincidence tracking code _cosream.c_.


# Confidence estimation of a given time difference #
The confidence of finding a peak is gauged by the excess _S_ of a peak above the mean of the ccf, expressed in multiples of standard deviations of the ccf. For the large number of elements in the FFT, the latter is usually determined by fluctuations in the base line of the ccf. With the assumption of a Gaussian distribution of noise on this base line, a statatistical significance _S_ above 6 ensures a very low probability of misidentifying a fluctuation as the correct ccf peak position.

If the reference clocks are slightly out of sync, and/or the acwuisition time is very long to obtain a stronger signal, the statistical significance for the fine resolution may be  worse than the one for the coarse resolution. On the contrary, if the acquired data is badly matched to the size of the discrete array, usually the statistical significance of the coarse resolution is smaller than for the fine resolution.

A high (_S_>6) statistical significance may with a consecutive lack of coincidences may indicate that the time difference is larger than what the algorithm an capture. This is usually caused by an insufficient synchronism between the host clocks.

# Future work #
Currently, this code needs a frequency matching of the two local reference oscillators within 10<sup>-9</sup>. This frequency pairing can only be accomplished using Rb reference oscillators or broadcasted copies of atomic clock references such as GPS receiving systems.

While Rb oscillators are reasonably easy available, and GPS receivers by now are almost a consumer item, it still requires a hardware complexity which is not necessitated by the problem: There is enough information content in the two timing data sets to extract not only a time difference, but also a frequency difference between two local clocks.

Caleb Ho has developed such an algorithm, which allows to use consumer-grade crystal oscillators with a long term stability and accuracy of 10-100ppm as reference clocks. A description of this algorithm, together with a more thorough analysis of the present scheme, can be found at http://arxiv.org/abs/0901.3203