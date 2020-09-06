This is a page describing the 1AUD filter, a modification of the 1EURO filter

The 1EURO filter increases the cutoff frequency of the filter during rapid input movements.

This filter uses the same principle but has some modifications that appear to improve performance in certain applications, such as tracking a gimbal.

In principle, the user determines a minimum cutoff frequency for the smoothing function, and a gain factor for how quickly the cutoff should rise during fast movement.  The differential factor then shifts the cutoff upwards during faster movements.  

In more detail, the code:

- (optionally) applies a slew limiter to remove extreme solitary spikes
- applies a second order lowpass filter to smooth the data / suppress high frequency noise
- calculates the derivative of that smoothed input signal
- shifts the minCutoff value upwards as derivative increases
- calculates the lowpass constant k for the final smoothing function
- constrains k to limit the maximum cutoff frequency
- applies a (first order) lowpass with this variable k factor over the (slew limited) input data

The max frequency can be set using an appropriate maximum k value. 

In pseudocode:

```
  ## Calculate lowpass k value
kInput = 2*PI*inputCutoff/sampleRate
 ## (optional) slew limit input steps to 20 max
slewedInput = slew(input)
 ## First order PT1 that slewed input
smoothedSlewedInput = prevSlewedInput + kInput * (slewedInput - prevSlewedInput)
prevSlewedInput = smoothedSlewedInput
 ## Repeat to get second order smoothing
verySmoothedSlewedInput = prevVerySmoothedSlewedInput + kInput * (smoothedSlewedInput - prevVerySmoothedSlewedInput)
prevVerySmoothedSlewedInput = verySmoothedSlewedInput
## 1Euro code:
  ## get differential of very smoothed slewed input
dx = (verySmoothedSlewedInput-prevVerySmoothedSlewedInput) * sample rate
  ## adjust cutoff upwards using dx and Beta
cutoff = minCutoff + Beta * ABS(dx)
  ## get the k value for the cutoff 
kCutoff = 2*PI*cutoff/sampleRate
  ## limit kCutoff to a value that sets the upper limit of the cutoff
  ## kCutoff must not exceed 1
kCutoff = MAX (kCutoff, 1)
  ## use kCutoff to filter the slewed input 
oneEuroOut = prevOneEuroOut + kCutoff * (slewedInput - prevOneEuroOut)
prevOneEuroOut = oneEuroOut
  ## for second order / smoother filtering
oneAUD = prevOneAUD + kCutoff * (oneEuroOut - prevOneAUD)
output = one AUD
```