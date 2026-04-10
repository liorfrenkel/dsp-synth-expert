---
title: "Fourier Analysis for Sound Designers"
category: "DSP"
tags: [FFT, DFT, spectrum, additive, convolution, window-functions]
---

# Fourier Analysis for Sound Designers

## Overview

Fourier analysis decomposes any periodic signal into a sum of sinusoids. For audio, it reveals the frequency content (the spectrum) of a sound. The Discrete Fourier Transform (DFT) and its fast algorithm (FFT) are foundational tools in synthesis, analysis, effects processing, and convolution reverb.

---

## Discrete Fourier Transform (DFT)

### Definition

The DFT transforms N time-domain samples into N frequency-domain bins:

```
X[k] = Σ x[n] * e^(-j * 2π * k * n / N)    for k = 0, 1, ..., N-1
       n=0..N-1

where:
  x[n] = input signal (time domain), n = sample index
  X[k] = output spectrum (frequency domain), k = bin index
  N    = transform size (number of samples)
  j    = imaginary unit (√-1)
  e^(-j*2π*k*n/N) = complex sinusoid at frequency k/N cycles per sample
```

Expanding via Euler's formula: `e^(-jθ) = cos(θ) - j*sin(θ)`:

```
X[k] = Σ x[n] * [cos(2π*k*n/N) - j*sin(2π*k*n/N)]
       n=0..N-1
```

### Frequency Bin Mapping

Bin `k` corresponds to frequency:
```
f_k = k * Fs / N     (in Hz)
f_k = k / N          (in cycles per sample, normalized)
```

- Bin 0: DC (0 Hz)
- Bin 1: Fs/N Hz (fundamental frequency for a period-N signal)
- Bin N/2: Fs/2 Hz (Nyquist frequency)
- Bins N/2+1 .. N-1: mirror image of bins N/2-1 .. 1 (negative frequencies)

For a 1024-point DFT at 44100 Hz:
- Frequency resolution: 44100/1024 ≈ **43.07 Hz per bin**
- Bin 1 = 43.07 Hz, Bin 10 = 430.7 Hz, Bin 512 = 22050 Hz (Nyquist)

### DFT Complexity: O(N²)

The naive DFT computes each of N output bins as a sum of N multiplications:
- Total operations: N × N = **N² complex multiplications**
- At N = 1024: ~1,048,576 operations per frame
- At N = 16384: ~268 million operations per frame
- Impractical for real-time audio at large N

---

## Fast Fourier Transform (FFT)

### Cooley-Tukey Algorithm

The FFT (Cooley-Tukey, 1965) reduces DFT complexity from O(N²) to **O(N log₂ N)** by exploiting symmetry and periodicity of the twiddle factors `e^(-j2πk/N)`.

### Butterfly Diagram

The FFT is computed as a series of "butterfly" operations, recursively dividing the transform into smaller DFTs:

```
// Decimation-in-time (DIT) radix-2 butterfly:
// Input:  A, B (two complex values)
// Twiddle: W = e^(-j * 2π * k / N)

A' = A + W * B
B' = A - W * B
```

For N = 8, the 8-point FFT requires log₂(8) = 3 stages of 4 butterflies each:
- Stage 1: 4 butterflies (N/2 = 4, twiddle factor W₈⁰, W₈², W₈⁴, W₈⁶)
- Stage 2: 4 butterflies with different twiddles
- Stage 3: 4 butterflies with different twiddles
- Total: 3 × 4 = 12 butterfly operations vs. 64 for naive DFT

### Complexity Comparison

| N      | DFT (N²)         | FFT (N log₂N)  | Speedup |
|--------|------------------|-----------------|---------|
| 64     | 4,096            | 384             | 10.7×   |
| 1024   | 1,048,576        | 10,240          | 102.4×  |
| 4096   | 16,777,216       | 49,152          | 341×    |
| 65536  | 4,294,967,296    | 1,048,576       | 4096×   |

### FFT Requirements

- Input size must be a **power of two** (radix-2): 64, 128, 256, 512, 1024, 2048, ...
- Non-power-of-two sizes: use mixed-radix FFT or zero-pad to next power of two
- Real-valued input: exploit conjugate symmetry, reducing computation by ~2×

### Simple In-Place Radix-2 FFT (Cooley-Tukey)

```c
// Complex array: interleaved real/imag, length N (power of 2)
void fft(double *real, double *imag, int N) {
    // Bit-reversal permutation:
    for (int i = 1, j = 0; i < N; i++) {
        int bit = N >> 1;
        for (; j & bit; bit >>= 1) j ^= bit;
        j ^= bit;
        if (i < j) { swap(real[i], real[j]); swap(imag[i], imag[j]); }
    }
    // Butterfly stages:
    for (int len = 2; len <= N; len <<= 1) {
        double ang = -2 * M_PI / len;
        double wr = cos(ang), wi = sin(ang);
        for (int i = 0; i < N; i += len) {
            double cr = 1.0, ci = 0.0;
            for (int j = 0; j < len/2; j++) {
                double ur = real[i+j],          ui = imag[i+j];
                double vr = real[i+j+len/2]*cr - imag[i+j+len/2]*ci;
                double vi = real[i+j+len/2]*ci + imag[i+j+len/2]*cr;
                real[i+j]         = ur + vr;    imag[i+j]         = ui + vi;
                real[i+j+len/2]   = ur - vr;    imag[i+j+len/2]   = ui - vi;
                double ncr = cr*wr - ci*wi;
                ci = cr*wi + ci*wr;
                cr = ncr;
            }
        }
    }
}
```

### Inverse FFT (IFFT)

The IFFT is identical to the FFT with conjugated twiddle factors and a 1/N normalization:

```c
void ifft(double *real, double *imag, int N) {
    // Conjugate input (negate imaginary parts):
    for (int i = 0; i < N; i++) imag[i] = -imag[i];

    // Apply forward FFT:
    fft(real, imag, N);

    // Conjugate output and normalize:
    for (int i = 0; i < N; i++) {
        real[i] =  real[i] / N;
        imag[i] = -imag[i] / N;
    }
}
```

---

## Magnitude and Phase Spectrum

From the complex DFT output `X[k] = Re[k] + j*Im[k]`:

### Magnitude Spectrum

```
|X[k]| = sqrt(Re[k]² + Im[k]²)    // linear amplitude at bin k
|X[k]|_dB = 20 * log10(|X[k]|)    // in decibels
```

- Represents **amplitude** of each frequency component
- Symmetric for real input: `|X[k]| = |X[N-k]|`
- DC component: `|X[0]|` = sum of all samples (mean × N)

### Phase Spectrum

```
∠X[k] = atan2(Im[k], Re[k])       // phase in radians [-π, +π]
```

- Represents the **phase offset** of each frequency component
- Often ignored in spectral display but critical for:
  - Phase vocoder / pitch shifting
  - Convolution (correct alignment)
  - Additive synthesis reconstruction

### Power Spectrum

```
P[k] = |X[k]|² = Re[k]² + Im[k]²
```

Used for spectral analysis when phase is not needed. Parseval's theorem:
```
Σ |x[n]|² = (1/N) * Σ |X[k]|²
 n                    k
```
Total energy is preserved between time and frequency domain.

---

## Window Functions

### The Spectral Leakage Problem

The DFT assumes the signal is **periodic within the analysis window**. When a sinusoid is not an integer number of cycles within the N-sample window, its energy leaks into neighboring bins.

Example: Analyzing a 1000 Hz sine at 44100 Hz with N=1024:
- Period = 44100/1000 = 44.1 samples
- Window captures non-integer cycles
- Energy appears in multiple bins (spectral leakage)

Solution: **multiply the signal by a window function** that smoothly tapers to zero at both ends, reducing the discontinuity at the frame edges.

### Window Functions Comparison

```c
// Apply window to N-sample buffer before FFT:
for (int n = 0; n < N; n++) {
    x[n] *= window[n];
}
```

#### Rectangular (No Window)

```
w[n] = 1.0  for all n
```

- Main lobe: 2 bins wide
- Sidelobe level: -13 dB
- Highest spectral leakage
- Use only when signal is exactly periodic in the window

#### Hann (Hanning) Window

```
w[n] = 0.5 * (1 - cos(2π * n / (N-1)))
```

- Main lobe: 4 bins wide
- Sidelobe level: -31 dB (first), -60 dB (far sidelobes)
- Good general-purpose; standard for STFT/phase vocoder
- Perfect reconstruction with 75% overlap (overlap-add)

#### Hamming Window

```
w[n] = 0.54 - 0.46 * cos(2π * n / (N-1))
```

- Main lobe: 4 bins wide
- Sidelobe level: -41 dB (first sidelobe)
- Doesn't fully reach zero at edges → slight leakage
- Better peak sidelobe than Hann; slightly worse far sidelobes

#### Blackman Window

```
w[n] = 0.42 - 0.5 * cos(2π*n/(N-1)) + 0.08 * cos(4π*n/(N-1))
```

- Main lobe: 6 bins wide
- Sidelobe level: -58 dB
- Good for high-dynamic-range spectral analysis

#### Blackman-Harris Window

```
w[n] = 0.35875 - 0.48829*cos(2π*n/(N-1)) + 0.14128*cos(4π*n/(N-1)) - 0.01168*cos(6π*n/(N-1))
```

- Main lobe: 8 bins wide
- Sidelobe level: -92 dB
- Excellent for locating weak frequency components near strong ones

#### Kaiser-Bessel Window

```
w[n] = I₀(β * sqrt(1 - (2n/(N-1) - 1)²)) / I₀(β)

where I₀ = zero-th order modified Bessel function
      β  = shape parameter (controls sidelobe level)
```

Common β values:
- β = 0: Rectangular
- β = 5: ~57 dB sidelobe attenuation
- β = 8: ~80 dB sidelobe attenuation
- β = 12: ~120 dB sidelobe attenuation

Kaiser is the most flexible window: β tunes the **sidelobe level vs. main lobe width trade-off** continuously.

#### Window Properties Summary Table

| Window          | Main Lobe (bins) | Peak Sidelobe (dB) | Best Use                  |
|-----------------|-----------------|---------------------|---------------------------|
| Rectangular     | 2               | -13                 | Exact-period signals only |
| Hann            | 4               | -31                 | STFT, phase vocoder       |
| Hamming         | 4               | -41                 | Speech analysis           |
| Blackman        | 6               | -58                 | Audio spectral analysis   |
| Blackman-Harris | 8               | -92                 | Precision spectral analysis|
| Kaiser (β=8)   | ~6              | -80                 | Flexible, controllable    |
| Kaiser (β=12)  | ~8              | -120                | Extreme dynamic range     |

**Trade-off rule**: wider main lobe = better sidelobe suppression. Choose based on whether frequency resolution (narrow lobe) or dynamic range (low sidelobes) is more important.

---

## Convolution Theorem

### Statement

**Time-domain convolution = Frequency-domain multiplication**:

```
x * h = IFFT(FFT(x) × FFT(h))

where * denotes convolution and × denotes pointwise multiplication
```

This is the foundation of **convolution reverb** and FIR filter implementation.

### Computational Advantage

- Direct convolution of N-sample signal with M-sample filter: O(N×M)
- FFT-based convolution: O((N+M) × log₂(N+M))
- For large N, M: FFT approach is orders of magnitude faster

### Overlap-Add Method (Block Convolution)

For real-time convolution with long impulse responses:

```c
// Block size L = 512 (for example)
// Impulse response length M
// FFT size N = next power of 2 ≥ L + M - 1

// For each input block of L samples:
// 1. Zero-pad input block to N samples
// 2. FFT(input block) → X[k]
// 3. Multiply X[k] * H[k] (H precomputed from impulse response)
// 4. IFFT → y block of N samples
// 5. Add first L samples to output, save tail (N-L samples) for overlap with next block
```

The "overlap" from step 5 contains the convolution tail that spills into the next block — essential for correct temporal reconstruction.

---

## Short-Time Fourier Transform (STFT)

The STFT analyzes how the spectrum changes over time by applying the DFT to successive (possibly overlapping) windows of the signal.

### STFT Definition

```
STFT{x}(m, k) = Σ x[n] * w[n - m*H] * e^(-j*2π*k*n/N)
                n

where:
  m = frame index (time position)
  H = hop size (samples between frame centers)
  w = window function
  N = FFT size (= window length or zero-padded)
```

### STFT Parameters

| Parameter | Effect |
|-----------|--------|
| N (FFT size) | Larger → better frequency resolution, worse time resolution |
| H (hop size) | Smaller → smoother time resolution, more computation |
| Overlap = N/H | Common: 4× overlap (75%), 2× overlap (50%) |
| Window | Hann typical; must satisfy COLA condition for reconstruction |

### COLA (Constant Overlap-Add) Condition

For perfect reconstruction in overlap-add:
```
sum of all window values at any time point = constant

Hann window: hop = N/2 → sum = 1.0 (perfect)
Hann window: hop = N/4 → sum = 1.5 (also valid, normalize by 1.5)
```

### Spectrogram

The spectrogram visualizes STFT magnitude over time:
- X-axis: time (frame index m)
- Y-axis: frequency (bin index k)
- Color/brightness: `|STFT(m,k)|` in dB

```c
// Compute spectrogram:
for (int frame = 0; frame < numFrames; frame++) {
    int offset = frame * hopSize;
    for (int n = 0; n < fftSize; n++) {
        int sampleIdx = offset + n - fftSize/2;  // centered window
        real[n] = (sampleIdx >= 0 && sampleIdx < signalLen)
                  ? signal[sampleIdx] * hannWindow[n]
                  : 0.0;
        imag[n] = 0.0;
    }
    fft(real, imag, fftSize);
    for (int k = 0; k < fftSize/2; k++) {
        spectrogram[frame][k] = sqrt(real[k]*real[k] + imag[k]*imag[k]);
    }
}
```

---

## Phase Vocoder: Pitch Shifting and Time Stretching

The phase vocoder modifies time and pitch independently using the STFT. Foundation of all high-quality pitch shifters (Melodyne, Autotune, etc.).

### Analysis Phase (decompose)

1. Compute STFT frames with overlap (typically 75% = 4× hop)
2. For each bin k in frame m: record magnitude `|X_m[k]|` and phase `∠X_m[k]`

### Phase Unwrapping

Raw phase wraps in [-π, +π]. Compute instantaneous frequency deviation:

```c
// Expected phase advance per hop at bin k:
double expectedPhase = 2.0 * M_PI * k * hopSize / fftSize;

// Actual phase difference:
double phaseDiff = phase[k] - prevPhase[k] - expectedPhase;

// Wrap to [-π, π]:
phaseDiff = fmod(phaseDiff + M_PI, 2.0 * M_PI) - M_PI;

// True instantaneous frequency (in bins):
double trueFreq = k + phaseDiff * fftSize / (2.0 * M_PI * hopSize);

prevPhase[k] = phase[k];
```

### Time Stretching

To stretch by factor `α` (α > 1 = longer, slower):

```c
double synthHopSize = hopSize * alpha;  // analysis hop stays same, synthesis hop changes

// Output phase accumulation:
for each bin k:
    outputPhase[k] += trueFreq[k] * 2.0 * M_PI * synthHopSize / fftSize;
    outputMag[k]    = inputMag[k];  // magnitude unchanged

// Synthesize each frame at output phase, IFFT, overlap-add at synthesis hop
```

### Pitch Shifting

Pitch shift by `semitones` (positive = up):

```c
double shiftFactor = pow(2.0, semitones / 12.0);

// Time-stretch to synthHop = hopSize * shiftFactor, then resample to original length:
// Step 1: Phase vocoder time-stretch by shiftFactor
// Step 2: Resample output by 1/shiftFactor (change speed back to normal)
// Net effect: same duration, different pitch
```

### Pitch Correction (Autotune)

1. Detect fundamental frequency using autocorrelation or cepstrum
2. Compare to target pitch (nearest scale note)
3. Shift by the difference: `semitones = target_note - detected_pitch`

---

## Additive Synthesis via IFFT

### Concept

To synthesize a sound additively, set the desired magnitudes and phases in frequency domain, then IFFT:

```c
// Clear arrays:
for (int k = 0; k < N; k++) real[k] = imag[k] = 0.0;

// Set frequency components:
// Sine: imag part; Cosine: real part; Arbitrary phase: both
real[harmonic]     =  amp * cos(phase);
imag[harmonic]     = -amp * sin(phase);
// Mirror for conjugate symmetry (real output):
real[N - harmonic] =  amp * cos(phase);
imag[N - harmonic] =  amp * sin(phase);

// IFFT to get time-domain waveform:
ifft(real, imag, N);
// real[] now contains the time-domain waveform
```

### Spectral Content of Classic Waveforms

#### Sawtooth Wave (rising)

Harmonics at all positive integers, amplitude = 1/n:
```
Harmonic n:  1     2     3     4     5     6     7     8
Amplitude:  1.000 0.500 0.333 0.250 0.200 0.167 0.143 0.125
Phase:        0     π     0     π     0     π     0     π
dB:          0  -6.02 -9.54 -12.04 -13.98 -15.56 -16.90 -18.06
```

#### Square Wave

Odd harmonics only, amplitude = 1/n:
```
Harmonic n:  1     3     5     7     9     11    13
Amplitude:  1.000 0.333 0.200 0.143 0.111 0.091 0.077
Phase:        0     0     0     0     0     0     0
dB:          0  -9.54 -13.98 -16.90 -19.08 -20.83 -22.28
```

#### Triangle Wave

Odd harmonics only, amplitude = 1/n², alternating sign:
```
Harmonic n:  1     3     5     7     9     11    13
Amplitude:  1.000 0.111 0.040 0.020 0.012 0.008 0.006
Phase:       π/2 -π/2   π/2  -π/2   π/2  -π/2   π/2
dB:          0  -19.08 -27.96 -33.81 -38.06 -41.66 -44.57
```

#### Sine Wave

Single harmonic (n=1), no overtones. DFT bins:
```
Real part: real[1] = A/2,  real[N-1] = A/2
Imag part: imag[1] = -A/2, imag[N-1] = A/2
(for amplitude A, zero initial phase)
```

---

## Cepstrum Analysis

The cepstrum is the IFFT of the log magnitude spectrum. Useful for separating the fundamental (periodicity) from the spectral envelope (timbre):

```c
// Compute cepstrum:
// 1. FFT(x) → |X[k]|
// 2. Compute log: log(|X[k]|)
// 3. IFFT(log|X[k]|) → cepstrum c[n]

// Low-time cepstrum (n near 0): spectral envelope
// High-time cepstrum (n near period): fundamental pitch info

// Pitch detection via cepstrum:
int peakIdx = argmax(cepstrum, lowTimeEnd, N/2);
double period = peakIdx;            // in samples
double pitch  = sampleRate / period; // in Hz
```

---

## Autocorrelation for Pitch Detection

Autocorrelation finds the periodicity in a signal — the lag at which the signal most resembles itself:

```c
// Autocorrelation of signal x[n] of length N:
double acf[N];
for (int lag = 0; lag < N; lag++) {
    double sum = 0.0;
    for (int n = 0; n < N - lag; n++) {
        sum += x[n] * x[n + lag];
    }
    acf[lag] = sum;
}
// acf[0] = maximum (signal matches itself with no lag)
// First peak after acf[0] = fundamental period
```

FFT-based autocorrelation: O(N log N) vs. O(N²) for direct:
```c
// 1. FFT(x) → X[k]
// 2. Compute power spectrum: P[k] = |X[k]|²
// 3. IFFT(P[k]) → autocorrelation function
```

---

## Practical DSP Block Sizes

| Block Size | Latency @ 44100 Hz | Frequency Resolution | Use Case                    |
|------------|---------------------|---------------------|-----------------------------|
| 64         | 1.45 ms             | 689 Hz              | Real-time low-latency        |
| 128        | 2.90 ms             | 345 Hz              | Low-latency plugins          |
| 256        | 5.80 ms             | 172 Hz              | Standard plugin processing   |
| 512        | 11.6 ms             | 86 Hz               | Moderate latency effects     |
| 1024       | 23.2 ms             | 43 Hz               | STFT effects, good resolution|
| 2048       | 46.4 ms             | 21 Hz               | Spectral processing          |
| 4096       | 92.9 ms             | 10.8 Hz             | High-quality spectral work   |

Note: For STFT-based processing, actual block latency = N samples of buffering. With 4× overlap and Hann window, processing latency = window_size/2 samples.

---

## Zero Padding

Padding the input with zeros before FFT increases frequency resolution of the output spectrum (interpolates between DFT bins) without adding new frequency information:

```c
// Zero-pad N real samples to 4N for 4× spectral interpolation:
for (int n = 0;       n < N;  n++) paddedX[n]   = x[n];
for (int n = N;       n < 4*N; n++) paddedX[n]  = 0.0;
fft(paddedX, paddedImag, 4*N);
// Now 4N bins, each N/(4N) = 1/4 the original bin spacing
```

**Zero-padding does NOT improve frequency resolution** in the information-theoretic sense — it reveals the DFT's implicit interpolation. To genuinely improve resolution, you must analyze more actual signal samples (longer window).
