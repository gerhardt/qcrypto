# Introduction #

Error correction is done asynchronously with acquisition of raw key. The same code runs on both machines and receives instructions via a command socket.  More doc to follow.


# Details #
## What this does ##
We implement a more or less straightforward CASCADE algorithm (1 on a cluster of raw key blocks, and continues with a privacy amplification step. The error correction step is broken down two rounds of subsequent parity checking on a smaller and smaller number of blocks to isolate errors, a randomized parity estimation over halfs of the whole message to efficiently bring down the error probability below a given threshold, and then the privacy amplification step. If an error is found in the randomized halfs estimation, another cross checking in the original parity blocks is done to identify and remove the twin errors (1a).
In the whole process, we track exactly how many bits are revealed in the process, and make them the basis of an information leakage in the error correction process. When optimized to a rough expected error ratio QBER (an information which is provided manually, and servoed from previous runs), this implementation reaches the Shannon limit, i.e., the fraction of revealed bits is determined by h(QBER), where h is the binary entropy function.

## How the code runs ##
Due to the many messages passing between the two side in the CASCADE algorithm, we need to implement reasonable deep pipelining, i.e., work on many raw key clusters in parallel while waiting for individual message responses from the other side.

Thus, we run error correction as a demon on both sides which is triggered either by a raw key cluster being submitted for EC/PA, or by messages received from the other side upon processing messages on previous raw bit clusters.

Once the bit error threshold has been reached, the cluster is passed onto the privacy amplification stage, which essentially is a compression step down to a size determined by the original raw key length minus the publicly revealed bits minus the inferred knowledge of the eavesdropper, which may be either derived from the observed error ratio, or externally with the degree of violation of a Bell inequality. The compression is probably done in an improper way just by multiplication of the raw key vector with a rectangular 'random' hash matrix, which itself is generated from a pseudorandom number generator seeded with a fresh (but published) 32bit number derived from a 'good' random source for each raw key block.

## Inputs, Outputs ##
We consume clusters of raw key files in the t3 format, and output the final key in the same format (different header?)

# Open issues #
We use extensively random seeds for generating the parity subsets for cascade, which are pulled off the system's random number generator. Instead of trusting the entropy sources in the host machine for that, we really should really recycle the vast amount ov entropy given to us by the randomness of the arrival times of the photons and background counts.

We also stretch the random seeds with a 32 bit LFSR pesudorandom number generator, which is probably a very bad idea in the first place. This PRNG should probably be replaced by something better.

In particular, we generate the hash matrix for the privacy amplification in this way, which has the tendency to be a REALLY bad idea. Any suggestions to improve on this would be highly welcome. Recently, I came across a reference (2) from H. Krawczyk from 1990ies which seems to suggest a very similar hash algorithm; if I understand this article correctly (apologies I am not a mathematician), there may even be a proof that the LFSR-based hashing we use in the privacy amplification step is perhaps 'epsilon-otp-secure', whatever that means. Would be nice to have someone clarifying this.

# Comments #
A nice idea is the use of LDPC instead of CASCADE. As soon as I understand how this could be done, we should probably replace the error correction engine by that. Advantage is apparently vast reduction in the number of transmitted messages, disadvantage a larger number of bits revealed to the public. We were told that, while a problem from the information theoretical point of view (a syndrom reveals about half of the key in form of some parity bits), this poses no problem as long as long as the syndrome generators are secred (or based on an initially shared secret - Reference: communication with Keith Harrison at ICQIT 2009)

# References #
(1)  G. Brassard and L. Salvail, in _Advances in Cryptology - Proc.
Eurocrypt'94_, pp. 410-423 (1994)

(1a) T. Sugimoto and K. Yamazaki, _IEICE Trans. Fundamentals_ **E83-A**, 1987 (2000)

(2) Hugo Krawczyk: LFSR-based Hashing and Authentication, pp. 129-139,  Lecture Notes In Computer Science; Vol. 839. Proceedings of the 14th Annual International Cryptology Conference on Advances in Cryptology, Springer Verlag 1994 (ISBN:3-540-58333-5)