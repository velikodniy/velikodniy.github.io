Every step of pipeline affects noise. For example the Bayer filter averages the noise, and so on.

Of course, the stages are not equal in terms of noise changing, but it might be useful (and pretty interesting) to check.

Regarding the parameters of the noise, it’s interesting that [the deviation of the intensity follows the Poisson distribution](https://www.mssl.ucl.ac.uk/www_detector/ccdgroup/optheory/darkcurrent.html). We can use this fact for sampling noise.

A couple of other papers on the noise sources in CCD sensors:

- [CCD Noise Sources and Signal-to-Noise Ratio](https://hamamatsu.magnet.fsu.edu/articles/ccdsnr.html)
- [CCD Sensitivity and Noise - What is noise and what does it matter?](https://andor.oxinst.com/learning/view/article/ccd-sensitivity-and-noise)
- [A Software Platform for Manipulating the Camera Imaging Pipeline](https://karaimer.github.io/camera-pipeline/).