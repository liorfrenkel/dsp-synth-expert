---
title: "Anti-Aliasing in Digital Synthesis"
category: "DSP"
tags: [anti-aliasing, Nyquist, oversampling, BLEP, PolyBLEP, aliasing]
---

# Anti-Aliasing in Digital Synthesis

## Overview

Aliasing is the most audible artifact in naive digital synthesis. When a waveform's harmonics exceed the Nyquist frequency (half the sample rate), they fold back into the audible spectrum as inharmonic tones. This document covers the theory of aliasing and every practical technique to prevent it.

---

## Nyquist-Shannon Sampling Theorem

**Theorem**: A continuous-time signal can be perfectly reconstructed from its samples if and only if it was sampled at a rate greater than twice the highest frequency component in the signal.

```
Maximum representable frequency = Fs / 2  (Nyquist frequency)
```

Corollary for synthesis: any frequency component generated above `Fs/2` cannot be represented faithfully — it will alias.

### Practical Limits

| Sample Rate | Nyquist Frequency | Last Safe Harmonic @ 440 Hz |
|-------------|-------------------|------------------------------|
| 44100 Hz    | 22050 Hz          | Harmonic 50 (21,780 Hz)      |
| 48000 Hz    | 24000 Hz          | Harmonic 54 (23,760 Hz)      |
| 88200 Hz    | 44100 Hz          | Harmonic 100 (43,956 Hz)     |
| 96000 Hz    | 48000 Hz          | Harmonic 109 (47,916 Hz)     |
| 192000 Hz   | 96000 Hz          | Harmonic 218 (95,920 Hz)     |

At 44100 Hz, a naive 440 Hz sawtooth has harmonic 50 at 21,780 Hz (just below Nyquist) and harmonic 51 at 22,440 Hz which aliases to |22,440 - 44,100| = 21,660 Hz — an inharmonic tone.

---

## The Aliasing Mechanism: Fold-Back Formula

When a signal at frequency `f` is sampled at rate `Fs`:

```
f_alias = |f_harmonic - round(f_harmonic / Fs) * Fs|

// Simplified (for f_harmonic in range [0, Fs]):
if (f_harmonic > Fs/2)
    f_alias = Fs - f_harmonic;   // reflects around Nyquist
```

### Examples (Fs = 44100 Hz)

```
f_harmonic = 23000 Hz → f_alias = 44100 - 23000 = 21100 Hz
f_harmonic = 30000 Hz → f_alias = 44100 - 30000 = 14100 Hz
f_harmonic = 50000 Hz → round(50000/44100)*44100 = 44100
                       → f_alias = |50000 - 44100| = 5900 Hz
```

### Pitch-Dependent Aliasing

Aliases shift when the pitch changes:
- At A2 (110 Hz): alias tone from harmonic 201 = |201×110 - 44100| = |22110 - 44100| = 21990 Hz
- At A4 (440 Hz): alias tone from harmonic 51 = |51×440 - 44100| = |22440 - 44100| = 21660 Hz
- At A5 (880 Hz): alias tone from harmonic 26 = |26×880 - 44100| = |22880 - 44100| = 21220 Hz

As pitch rises, aliases occur at lower harmonics and become progressively louder relative to the fundamental. Above ~2 kHz, aliasing from a naive sawtooth becomes plainly audible.

---

## Analog vs. Digital: Why Analog Has No Aliasing

Analog oscillators operate in continuous time with no concept of a Nyquist limit. The physical limitations of the circuit (component capacitances, inductor cutoffs) naturally roll off the signal well before any problematic frequency — and even if they didn't, there's no sampling process to create fold-back.

In analog:
- A 440 Hz sawtooth contains harmonics to ~100+ kHz naturally
- These are filtered by the circuit's own bandwidth (typically 20–80 kHz)
- No aliasing: inaudible frequencies simply aren't heard

This is why digital emulation requires explicit band-limiting: we must artificially recreate what analog achieves physically for free.

---

## Anti-Aliasing Technique 1: Oversampling

### Concept

Generate the signal at N times the target sample rate, then low-pass filter at 0.45 × target Fs before discarding the extra samples.

```
Generate at N*Fs → Apply LPF at 0.45*Fs → Decimate by N → Output at Fs
```

At 4x oversampling (176400 Hz internally at 44100 output):
- Nyquist is now 88200 Hz
- Even a naively-generated 10 kHz saw has clean harmonics up to 88 kHz
- The decimation LPF removes everything above 22050 Hz before downsampling

### Oversampling C Implementation

```c
const int OS_FACTOR = 4;
float oversampleBuf[OS_FACTOR];

// Generate N samples at internal rate:
for (int k = 0; k < OS_FACTOR; k++) {
    phase += phaseInc_internal;  // phaseInc / OS_FACTOR
    if (phase >= 1.0) phase -= 1.0;
    oversampleBuf[k] = 2.0 * phase - 1.0;  // naive saw
}

// Decimate (simple averaging — not ideal, use proper FIR in production):
float output = 0.0f;
for (int k = 0; k < OS_FACTOR; k++) output += oversampleBuf[k];
output /= OS_FACTOR;
```

### Proper Decimation Filter

Simple averaging is a box filter with poor stopband attenuation. Use a Parks-McClellan designed FIR:

```c
// Example: 32-tap FIR for 4x decimation, passband 0-20kHz, stopband 24.05kHz+
// (Cutoff at 0.4545 * Fs_original = 20.05 kHz, transition to 0.5454 * Fs_original)
const float decimFIR[32] = {
    -0.000197, -0.000574, -0.000892, -0.000736,  0.000418,
     0.002682,  0.005344,  0.006754,  0.004524, -0.003059,
    -0.014974, -0.026543, -0.030088, -0.017064,  0.020086,
     0.079547,  0.148064,  0.200264,  0.218100,  0.200264,
     0.148064,  0.079547,  0.020086, -0.017064, -0.030088,
    -0.026543, -0.014974, -0.003059,  0.004524,  0.006754,
     0.005344,  0.002682  // (symmetric, normalization factor omitted)
};
```

### Oversampling Cost vs. Quality

| Oversampling | CPU Factor | Alias Reduction | Use Case                |
|-------------|------------|-----------------|-------------------------|
| 1x          | 1.0×       | None            | Only for bandlimited sources |
| 2x          | ~2.2×      | Good            | Moog/SVF filter nonlinearity |
| 4x          | ~4.5×      | Very good       | Distortion, simple osc   |
| 8x          | ~9×        | Excellent       | FM, complex waveshaping  |
| 16x         | ~18×       | Near-perfect    | Extreme distortion       |

(CPU factor includes decimation filter overhead)

---

## Anti-Aliasing Technique 2: PolyBLEP

PolyBLEP corrects aliasing by adding a polynomial residual correction at each waveform discontinuity. Developed by Valimaki et al.; popularized in software synthesis.

### The BLEP Concept

A Bandlimited Step (BLEP) is the ideal bandlimited version of a unit step function. The difference between a naive step and a bandlimited step is the **BLEP residual**. Adding the residual to a naive waveform approximates a bandlimited waveform.

```
Bandlimited waveform ≈ Naive waveform + BLEP residuals at each discontinuity
```

### PolyBLEP: Polynomial Approximation of BLEP Residual

The polynomial BLEP residual function approximates the ideal BLEP residual with a simple piecewise polynomial, evaluated in a small window around each discontinuity:

```c
// PolyBLEP residual function:
// t  = phase position [0, 1)
// dt = phase increment (freq/sampleRate)
// Returns the correction to add to the naive sample at position t

double polyblep(double t, double dt) {
    if (t < dt) {
        // We're in the interval [0, dt) — just after the discontinuity:
        t /= dt;                          // normalize to [0, 1)
        return t + t - t * t - 1.0;       // = 2t - t² - 1
    } else if (t > 1.0 - dt) {
        // We're in the interval (1-dt, 1) — just before the discontinuity:
        t = (t - 1.0) / dt;               // normalize to [-1, 0)
        return t * t + t + t + 1.0;       // = t² + 2t + 1
    }
    return 0.0;
}
```

The residual is nonzero only within one `dt` sample of the discontinuity. The polynomial ensures the frequency response matches the ideal sinc-interpolated step.

### PolyBLEP Sawtooth

The sawtooth has one discontinuity per cycle at `phase = 0` (the jump from +1 back to -1):

```c
double saw_polyblep(double &phase, double phaseInc) {
    // Naive sawtooth: maps phase [0,1) → [-1, +1)
    double output = 2.0 * phase - 1.0;

    // PolyBLEP correction at phase = 0 (discontinuity location):
    // Subtract residual (the discontinuity is a falling step of magnitude 2)
    output -= polyblep(phase, phaseInc);

    // Update phase:
    phase += phaseInc;
    if (phase >= 1.0) phase -= 1.0;

    return output;
}
```

### PolyBLEP Square Wave

The square wave has two discontinuities: rising edge at `phase = 0`, falling edge at `phase = pulseWidth`:

```c
double square_polyblep(double &phase, double phaseInc, double pulseWidth = 0.5) {
    // Naive square:
    double output = (phase < pulseWidth) ? 1.0 : -1.0;

    // Correct rising edge at phase = 0 (adds +2 step):
    output += polyblep(phase, phaseInc);

    // Correct falling edge at phase = pulseWidth (subtracts -2 step):
    // Compute local phase relative to falling edge:
    double phaseShifted = phase - pulseWidth;
    if (phaseShifted < 0.0) phaseShifted += 1.0;
    output -= polyblep(phaseShifted, phaseInc);

    // Update phase:
    phase += phaseInc;
    if (phase >= 1.0) phase -= 1.0;

    return output;
}
```

### PolyBLEP Triangle

The triangle wave has slope discontinuities (not amplitude discontinuities). Integrate a PolyBLEP square, or apply the derivative of the BLEP:

```c
double triangle_polyblep(double &phase, double phaseInc) {
    // Integrate the square wave output with leaky integrator:
    double squareOut = square_polyblep(phase, phaseInc);
    static double integ = 0.0;
    integ = integ + phaseInc * squareOut;  // basic integration
    return integ;
}
// Note: DC-block the output to remove integration offset
```

### PolyBLEP Quality Analysis

- **2nd-order PolyBLEP** (as shown): effective below ~10 kHz at 44100 Hz
- **4th-order PolyBLEP**: extends quality to ~15 kHz; uses wider polynomial window
- **6th-order PolyBLEP**: near-MinBLEP quality; rarely needed
- **Phase error**: PolyBLEP introduces slight phase shift near discontinuity (typically inaudible)
- **Bandwidth**: correction degrades at high frequencies where `dt > 0.5` (avoid using above Fs/4)

---

## Anti-Aliasing Technique 3: BLIT (Band-Limited Impulse Train)

### Theory

A periodic waveform can be expressed as a Fourier series. The BLIT method directly synthesizes the bandlimited version by summing only the harmonics below Nyquist:

```
                  Fs/(2*f₀)
BLIT(t) = (2/T) * Σ cos(2π * n * f₀ * t)
                  n=1

= (f₀/Fs) * sin(M * π * f₀/Fs * t) / sin(π * f₀/Fs * t)

where M = 2 * floor(Fs / (2 * f₀)) + 1  (odd number of harmonics)
      T = 1/f₀ = period
```

### BLIT Implementation (Stilson/Smith)

```c
double blit(double phase, double phaseInc) {
    // M = number of harmonics (odd integer):
    int M = 2 * (int)(0.5 / phaseInc) + 1;

    double x   = M_PI * phase;       // phase in [0, π*M]
    double denom = sin(x);

    if (fabs(denom) < 1e-10) {
        // Near singularity: use L'Hopital's limit = M/period
        return phaseInc * M;
    }

    return phaseInc * sin(M * x) / denom;
}
```

### BLIT to Sawtooth (Integration)

```c
// Running integral with DC block:
double blitSaw(double &phase, double phaseInc, double &leaky) {
    double b = blit(phase, phaseInc);
    phase += phaseInc;
    if (phase >= 1.0) phase -= 1.0;

    // Leaky integration (prevents DC drift):
    leaky = leaky + b - phaseInc;
    return leaky;
}
```

### BLIT to Square (Two BLIT Streams)

```c
// Square from difference of two BLIT streams offset by half-period:
double blitSquare(double &phase1, double &phase2, double phaseInc, double &leaky) {
    double b1 = blit(phase1, phaseInc);
    double b2 = blit(phase2, phaseInc);
    phase1 += phaseInc;
    phase2 += phaseInc;
    if (phase1 >= 1.0) phase1 -= 1.0;
    if (phase2 >= 1.0) phase2 -= 1.0;

    leaky = leaky + (b1 - b2) - phaseInc;  // integrate difference
    return leaky;
}
// Initialize: phase2 = phase1 + 0.5 (half-period offset)
```

---

## Anti-Aliasing Technique 4: MinBLEP

MinBLEP (Minimum Phase BLEP) is a precomputed lookup table version of the BLEP residual. Created by Andy Farnell and used in Surge, U-He, and other professional synthesizers.

### Construction

```c
// 1. Create the ideal sinc kernel (windowed):
//    Length: 2 * MINBLEP_CUTOFF + 1 samples, oversampled by MINBLEP_OS

// 2. Apply window (Blackman or Kaiser):
for (int i = 0; i < kernelLen; i++) {
    double t      = (i - kernelLen/2.0);
    double sinc   = (t == 0) ? 1.0 : sin(M_PI * t / MINBLEP_OS) / (M_PI * t / MINBLEP_OS);
    double window = 0.42 - 0.5*cos(2*M_PI*i/kernelLen) + 0.08*cos(4*M_PI*i/kernelLen);
    kernel[i]     = sinc * window;
}

// 3. Convert to minimum phase via cepstrum method

// 4. Integrate to get BLEP step function

// 5. Subtract ideal step to get residual: blep_table[i] = integral - unitStep
```

### MinBLEP Usage

```c
// At each discontinuity, inject the residual into the output buffer:
void insertMinBLEP(float *outputBuf, int bufSize,
                   double fractionalOffset,  // 0..1 sub-sample position
                   float stepHeight,         // magnitude of discontinuity
                   const float *minblep, int minblepLen) {
    // Interpolate into minblep table for sub-sample accuracy:
    int intOffset   = (int)(fractionalOffset * MINBLEP_OS);
    for (int i = 0; i < minblepLen / MINBLEP_OS; i++) {
        if (bufPos + i < bufSize)
            outputBuf[bufPos + i] += minblep[i * MINBLEP_OS + intOffset] * stepHeight;
    }
}
```

### MinBLEP Characteristics

- **Quality**: effectively perfect bandlimited waveforms (SNR > 100 dB)
- **Latency**: zero (minimum phase → causal)
- **CPU**: O(N) per discontinuity, where N = minblep length / oversample ratio
- **Memory**: 2–16 KB for typical minblep tables
- **Sub-sample accuracy**: handles fractional sample positions via table interpolation

---

## Anti-Aliasing Technique 5: Wavetable with Mipmap

Already covered in `wavetable_interpolation.md`. Summary for completeness:

- Pre-compute band-limited versions of the wavetable for each octave range
- Select the appropriate table based on playback frequency
- Zero aliasing (within the band-limit of the table)
- Works for any arbitrary waveform, not just simple waveforms
- Best approach for general-purpose software synthesizers

---

## Technique Comparison Table

| Method          | Quality | CPU    | Memory | Arbitrary Waveforms | Dynamic Modulation |
|-----------------|---------|--------|--------|---------------------|-------------------|
| Naive (none)    | Poor    | Minimal| None   | Yes                 | Yes               |
| Oversampling 4x | Good    | 4–5×   | None   | Yes                 | Yes               |
| Oversampling 8x | Excellent| 9×    | None   | Yes                 | Yes               |
| PolyBLEP 2nd    | Fair    | Low    | None   | Saw/Sq/Tri only     | Yes               |
| PolyBLEP 4th    | Good    | Low    | None   | Saw/Sq/Tri only     | Yes               |
| MinBLEP         | Excellent| Medium| ~16 KB | Saw/Sq/Tri only     | Yes               |
| BLIT            | Very Good| Medium| None   | Saw/Sq/Tri          | Yes               |
| Wavetable mipmap| Excellent| Low   | ~100KB+| Any waveform        | Yes (table select)|

---

## Practical Recommendations by Synthesis Type

### Basic Subtractive Synthesis (Saw/Square/Triangle)

Recommended: **PolyBLEP** (2nd or 4th order)
- Zero-latency, negligible CPU, easy implementation
- Sufficient quality below ~8 kHz
- Above 8 kHz: minimal aliasing even without correction (harmonics too high to hear clearly)

### High-Quality Subtractive

Recommended: **Wavetable + mipmap** with Hermite interpolation
- Pre-computed tables, near-zero runtime CPU
- Perfect quality at all frequencies
- Handles arbitrary single-cycle waveforms, not just classic shapes

### FM Synthesis

Recommended: **Oversampling 4x–8x**
- FM generates sum and difference frequencies at unexpected locations
- The carrier + modulator intermodulation products require headroom above Nyquist
- 4x oversampling pushes aliasing artifacts well above audibility

### Distortion / Waveshaping

Recommended: **Oversampling 4x–8x** before and after the nonlinearity
- Apply distortion at elevated sample rate
- Decimate after nonlinearity
- Stack: [oscillator → upsample → distort → decimate → output]

### Moog-Style Ladder / SVF Filter Nonlinearity

Recommended: **2x oversampling** inside the filter loop
- The tanh saturation inside the filter feedback loop creates harmonics
- Running the filter at 2x internal rate keeps these harmonics below Nyquist
- Run the filter twice with the same input sample; discard every other output

```c
// Double-sample Moog at 2x internal rate:
float doubleSampledMoog(float input, MoogFilter &f) {
    f.process(input);  // first pass (discard output)
    return f.process(input);  // second pass (keep output)
    // This approximates 2x oversampling for free (same input both times)
}
```

---

## DC Blocking

Integration-based methods (BLIT, integrated PolyBLEP) can accumulate DC offset over time. Apply a simple DC-blocking highpass filter:

```c
// One-pole DC blocker (R = 0.995 to 0.9999 depending on lowest freq):
double dcBlock(double input, double &prev_in, double &prev_out) {
    double output = input - prev_in + 0.9995 * prev_out;
    prev_in  = input;
    prev_out = output;
    return output;
}
```

Cutoff frequency: `fc = (1 - R) * Fs / (2 * π) ≈ 3.5 Hz at R=0.9995, Fs=44100 Hz`

---

## Aliasing Audibility Thresholds

Practical observation: aliasing at -60 dB below fundamental is generally inaudible in a mix. PolyBLEP reduces aliases to approximately -40 to -60 dB; MinBLEP/wavetable reduces to < -90 dB.

| Alias Level (dB) | Audibility in Mix |
|------------------|-------------------|
| 0 dB (naive)     | Very loud, clearly audible |
| -20 dB           | Audible on isolated signal |
| -40 dB           | Audible in quiet passages |
| -60 dB           | Borderline in bright mix  |
| -80 dB           | Inaudible in virtually all contexts |
| -90 dB+          | Below noise floor of most systems |

For professional production: target -80 dB or better. PolyBLEP achieves ~-50 to -60 dB; full-oversampling with proper decimation or MinBLEP achieves > -90 dB.
