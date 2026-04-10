---
title: "Wavetable Synthesis and Interpolation"
category: "DSP"
tags: [wavetable, interpolation, mipmap, anti-aliasing, cubic, sinc]
---

# Wavetable Synthesis and Interpolation

## Overview

Wavetable synthesis stores one or more single-cycle waveforms in memory (the "wavetable"), then plays them back at arbitrary frequencies by stepping through the table at a rate proportional to the desired pitch. The core challenges are:

1. **Aliasing**: harmonics above Nyquist fold back (use band-limited tables)
2. **Interpolation**: fractional phase positions must sample between stored values
3. **Morphing**: smooth transitions between different timbres

---

## Wavetable Construction

### Single-Cycle Waveforms

A wavetable holds exactly one cycle of a waveform, sampled at a fixed power-of-two length:
- **64 samples**: low quality, good for ROM-limited environments
- **256 samples**: adequate for simple waveforms
- **512 samples**: standard minimum for moderate quality
- **1024–2048 samples**: high quality, standard for software synths
- **4096 samples**: high quality with long sinc interpolation kernels

### Common Table Sizes

| Application | Typical Size |
|-------------|-------------|
| Classic hardware (PPG Wave, etc.) | 64 or 256 |
| Software synth basic quality     | 512–1024   |
| Software synth high quality      | 2048       |
| Serum / Vital (modern) per frame | 2048       |
| MinBLEP source table             | 2048–4096  |

### Constructing a Band-Limited Sawtooth Table

```c
// Build sawtooth via additive synthesis (IFFT method):
// tableSize = 2048

float ar[tableSize], ai[tableSize];  // real and imaginary FFT bins

// Clear all bins:
for (int i = 0; i < tableSize; i++) ar[i] = ai[i] = 0.0f;

// Set harmonic amplitudes: amplitude_n = 1.0/n
int maxHarmonic = tableSize / 2;  // limit to Nyquist
for (int h = 1; h <= maxHarmonic; h++) {
    float amp = 1.0f / h;
    ar[h] = amp;        // real part (cosine)
    ai[h] = 0.0f;       // imaginary part (sine) = 0 for sawtooth
}
// Mirror for negative frequencies (IFFT convention):
for (int h = 1; h < tableSize; h++) {
    ar[tableSize - h] =  ar[h];
    ai[tableSize - h] = -ai[h];
}

// Inverse FFT → time-domain waveform
ifft(tableSize, ar, ai);

// Normalize to [-1, +1]
float peak = 0.0f;
for (int i = 0; i < tableSize; i++) peak = max(peak, fabsf(ar[i]));
for (int i = 0; i < tableSize; i++) ar[i] /= peak;
```

### From Arbitrary Audio (User Waveform)

1. Load a single-cycle waveform (from WAV, recorded audio, etc.)
2. Resample to desired table size (power of two, typically 2048)
3. Take FFT to get frequency domain representation
4. Apply band-limit mask to remove harmonics above Nyquist (see Mipmap section)
5. Take IFFT back to time domain
6. Normalize
7. Store as the first mipmap level; generate additional levels by halving harmonic count

```c
// Step 4: Zero harmonics above maxHarmonic before IFFT
void bandLimitTable(float *ar, float *ai, int tableSize, int maxHarmonic) {
    for (int i = maxHarmonic + 1; i < tableSize - maxHarmonic; i++) {
        ar[i] = ai[i] = 0.0f;
    }
}
```

---

## Wavetable Playback

### Phase Accumulator for Wavetable

```c
// Phase in samples (fractional):
phase += tableSize * freq / sampleRate;   // phaseInc = tableSize * freq / sampleRate
if (phase >= tableSize) phase -= tableSize;

// Integer and fractional parts:
int    idx  = (int)phase;
double frac = phase - idx;
```

---

## Linear Interpolation

The simplest interpolation method. Takes two neighboring samples and blends between them proportionally.

### Formula

```
output = y[i] + frac * (y[i+1] - y[i])
       = y[i] * (1 - frac) + y[i+1] * frac
```

Where `frac ∈ [0.0, 1.0)` is the fractional position between `i` and `i+1`.

### C Implementation

```c
float linearInterp(float *table, int tableSize, double phase) {
    int    i    = (int)phase;
    double frac = phase - i;
    int    j    = (i + 1) % tableSize;  // wrap around
    return (float)(table[i] + frac * (table[j] - table[i]));
}
```

### Characteristics

- **CPU**: 1 multiply, 1 add, 1 lerp → very fast
- **Quality**: introduces low-pass roll-off at high frequencies (first-order hold response)
- **Frequency response**: `H(f) = sinc(f/Fs)` — -3 dB at approximately 0.26 * Nyquist
- **Recommended for**: quick prototyping, low-frequency wavetables, non-critical paths

---

## 4-Point Cubic (Hermite) Interpolation

Uses four surrounding samples to compute a smooth cubic polynomial interpolation. Significantly better high-frequency response than linear.

### Hermite Coefficients

Given 4 sample values y[-1], y[0], y[1], y[2] (where we interpolate between y[0] and y[1]):

```c
double hermite4(double frac, double ym1, double y0, double y1, double y2) {
    // Catmull-Rom / Hermite coefficients:
    double c0 = y0;
    double c1 = 0.5 * (y1 - ym1);
    double c2 = ym1 - 2.5 * y0 + 2.0 * y1 - 0.5 * y2;
    double c3 = 0.5 * (y2 - ym1) + 1.5 * (y0 - y1);

    // Horner's method for efficiency:
    return ((c3 * frac + c2) * frac + c1) * frac + c0;
}
```

- `c0 = y[0]` (value at integer position)
- `c1 = 0.5 * (y[1] - y[-1])` (slope estimate at y[0])
- `c2` = curvature term
- `c3` = cubic twist term

### Usage in Wavetable

```c
float cubicInterp(float *table, int tableSize, double phase) {
    int    i   = (int)phase;
    double frac = phase - i;

    // Wrap indices:
    int im1 = (i - 1 + tableSize) % tableSize;
    int i0  = i % tableSize;
    int i1  = (i + 1) % tableSize;
    int i2  = (i + 2) % tableSize;

    return (float)hermite4(frac,
                           table[im1],
                           table[i0],
                           table[i1],
                           table[i2]);
}
```

### Characteristics

- **CPU**: ~10 multiplies, ~6 adds (vs. 1/1 for linear)
- **Quality**: flat response to ~0.4 * Nyquist, then rolls off
- **Phase response**: near-linear (minimal smearing)
- **Overshoot**: ≤ 0 dB for inputs with smooth transitions
- **Recommended for**: standard wavetable synthesis, good trade-off of quality vs. CPU

### Waveform Hermite vs. Catmull-Rom

Both use the same coefficient structure. The difference:
- **Hermite**: explicit tangent control (here we estimate tangents from neighbors)
- **Catmull-Rom**: automatically uses neighboring points as tangent estimates (same formula)
- In practice, the above implementation **is** Catmull-Rom / Hermite-4 (same thing for audio)

---

## Sinc Interpolation (Windowed Sinc)

The theoretically ideal interpolation for bandlimited signals. Reconstructs the exact continuous-time signal using the ideal low-pass filter (sinc kernel).

### Formula

```
output = Σ x[n] * sinc(π * (t - n)) * window(n)
         n

where sinc(x) = sin(x) / x
```

In practice, use a windowed sinc to prevent ringing:

```c
float sincInterp(float *table, int tableSize, double phase,
                 float *sincTable, int sincLen) {
    int centerIdx = (int)phase;
    double frac   = phase - centerIdx;

    float output = 0.0f;
    int halfLen  = sincLen / 2;

    for (int i = -halfLen; i <= halfLen; i++) {
        int   srcIdx = (centerIdx + i + tableSize) % tableSize;
        float t      = (float)i - frac;
        // Windowed sinc lookup (precomputed table):
        float sincVal = sincTable[(int)((t + halfLen) * oversamplingFactor)];
        output += table[srcIdx] * sincVal;
    }
    return output;
}
```

### Windowed Sinc Kernel Generation

```c
// Generate Hann-windowed sinc kernel:
for (int i = 0; i < kernelLen; i++) {
    double t     = (i - kernelLen/2.0) / oversample;
    double sinc  = (t == 0.0) ? 1.0 : sin(M_PI * t) / (M_PI * t);
    double hann  = 0.5 * (1.0 - cos(2.0 * M_PI * i / kernelLen));
    kernel[i]    = sinc * hann;
}
```

### Common Window Functions for Sinc

| Window       | Stopband Attenuation | Main Lobe Width | Notes                  |
|--------------|---------------------|-----------------|------------------------|
| Rectangular  | ~13 dB              | Narrowest       | Poor; visible ringing  |
| Hann         | ~44 dB              | Moderate        | Good general purpose   |
| Hamming      | ~41 dB              | Moderate        | Slight pedestals       |
| Blackman     | ~74 dB              | Wider           | Good quality, wider    |
| Kaiser-Bessel| 60–120 dB (variable) | Adjustable     | Best control via β     |

### Sinc Interpolation Computational Cost

- Kernel length 8: ~8 multiplies/adds → practical for real-time
- Kernel length 16: ~16 multiplies/adds → good quality
- Kernel length 32: ~32 multiplies/adds → near-perfect
- Kernel length 64+: high quality, typically precomputed into oversampled table

For audio synthesis, a **polyphase filter bank** precomputes the sinc kernel at many fractional offsets, enabling O(1) table lookup per interpolation step.

### Characteristics vs. Other Methods

- **Quality**: theoretically perfect for bandlimited signals
- **CPU**: much higher (proportional to kernel length)
- **Latency**: kernel_length/2 - 1 samples of latency
- **Recommended for**: sample playback (one-shot samples), high-quality resamplers, mastering

---

## Mipmap Anti-Aliasing

A mipmap is a set of pre-filtered versions of the wavetable, each containing progressively fewer harmonics. The correct table is selected based on the playback frequency to avoid aliasing.

### The Aliasing Problem

If you play a 2048-sample table that was built with harmonics up to Nyquist at a high pitch, those high harmonics (which were fine at 100 Hz) now exceed the digital Nyquist limit at 5000 Hz playback.

Solution: use a different table for different frequency ranges, each pre-filtered to only contain harmonics that are safe at that range.

### Table Selection Formula

```c
// Number of harmonics safe for this frequency:
int maxHarmonics = (int)(sampleRate / (2.0 * freq));

// Table index (higher pitch → lower index in mipmap stack):
// Assuming tables are arranged from most harmonics to fewest:
int tableIndex = (int)floor(log2(sampleRate / (2.0 * freq)));

// Practical clamp:
tableIndex = max(0, min(tableIndex, numTables - 1));
```

Alternative formula from EarLevel engineering:
```c
// topFreq for each table = the highest frequency (as a phase increment) it handles:
// Each successive table handles one octave higher, containing half the harmonics.
// Select: find the lowest-index table whose topFreq exceeds phaseInc:
int tableIdx = 0;
while (phaseInc > waveTables[tableIdx].topFreq && tableIdx < numTables - 1)
    tableIdx++;
float *table    = waveTables[tableIdx].waveTable;
int    tableLen = waveTables[tableIdx].waveTableLen;
```

### Building Mipmap Tables

```c
// Start with full-bandwidth table (all harmonics up to tableSize/2)
// For each successive octave:
//   1. Take FFT of current table
//   2. Zero out upper half of harmonics
//   3. Inverse FFT
//   4. Store as next mipmap level

// Example: 2048-sample table, 11 mipmap levels (log2(2048) = 11):
// Level 0: harmonics 1..1024  (for lowest pitches)
// Level 1: harmonics 1..512   (one octave up)
// Level 2: harmonics 1..256   (two octaves up)
// ...
// Level 10: harmonic 1 only   (sine wave, for highest pitches)
```

### Mipmap Storage

```c
typedef struct {
    double  topFreq;        // phase increment threshold (freq/sampleRate)
    int     waveTableLen;   // can vary per level
    float  *waveTable;      // pointer to table data
} WaveTableEntry;

WaveTableEntry waveTables[MAX_NUM_TABLES];
int numWaveTables = 0;
```

### EarLevel Wavetable Oscillator Architecture

From Nigel Redmon's implementation:
- Phase accumulator: `double phasor, phaseInc, phaseOfs`
- Variable-length tables: each mipmap level can have different length
- Table selection: compare `phaseInc` against stored `topFreq` thresholds
- Interpolation: linear (sufficient when tables are pre-bandlimited)
- PWM: two oscillators sharing same tables, second with phase offset `phaseOfs`

```c
// Phase update (per sample):
void updatePhase(void) {
    phasor += phaseInc;
    if (phasor >= 1.0) phasor -= 1.0;
}

// Output retrieval:
float getOutput(void) {
    // Find appropriate table:
    int tableIdx = 0;
    while (phaseInc > waveTables[tableIdx].topFreq && tableIdx < numWaveTables - 1)
        tableIdx++;

    WaveTableEntry *wt = &waveTables[tableIdx];
    double pos   = phasor * wt->waveTableLen;
    int    i     = (int)pos;
    double frac  = pos - i;
    return wt->waveTable[i] + frac * (wt->waveTable[i + 1] - wt->waveTable[i]);
}
```

---

## Oversampling

Oversampling generates audio at 2x, 4x, or 8x the desired sample rate and then decimates (low-pass filters + downsamples) to the target rate.

### Oversampling Workflow

```
Input signal (at 1x Fs)
    ↓
Generate waveform at N*Fs (run oscillator N times per output sample)
    ↓
Apply anti-aliasing LPF at 0.45 * Fs (stopband before Nyquist of original Fs)
    ↓
Keep every Nth sample (decimation)
    ↓
Output at Fs
```

### Oversampling by Factor of 4 (Simple)

```c
// 4x oversampling inline:
float oversampledBuf[4];
for (int k = 0; k < 4; k++) {
    phase += phaseInc / 4.0;          // quarter-step
    if (phase >= 1.0) phase -= 1.0;
    oversampledBuf[k] = naiveSaw(phase);
}
// Decimate: apply FIR LPF and take every 4th sample
float output = decimate4x(oversampledBuf);
```

### Decimation Filter Requirements

For 4x oversampling, the decimation FIR must:
- Passband: 0 to 0.45 * Fs (original sample rate)
- Stopband: 0.55 * Fs to 2 * Fs (or better)
- Transition band: 0.45 to 0.55 * Fs
- Stopband attenuation: ≥ 90 dB for professional audio

A simple 4-tap FIR (linear phase):
```c
// Coefficients for half-band 4x decimation (approximate):
const float h[] = {0.0625, 0.5625, 0.5625, 0.0625};  // simple
// Higher quality: use Parks-McClellan/Remez designed FIR
```

### Recommended Oversampling Levels

| Synthesis Type          | Min Oversampling | Notes                         |
|-------------------------|------------------|-------------------------------|
| Pure sine               | 1x               | No aliasing by definition     |
| Triangle wave           | 2x               | Low harmonic energy           |
| Saw/Square (wavetable)  | 1x               | Mipmap handles it             |
| Saw/Square (naive)      | 4x               | PolyBLEP better alternative   |
| Moog ladder filter      | 2x               | Filter nonlinearities         |
| SVF with saturation     | 2x               | Prevent fold-back artifacts   |
| FM synthesis            | 4x               | Intermodulation products      |
| Distortion/waveshaping  | 4–8x             | Strong nonlinear content      |
| Complex FM stacks       | 8x               | Many sum/diff frequencies     |

---

## Wavetable Morphing

Morphing interpolates between two or more wavetables over time, creating smooth timbral evolution.

### Linear Crossfade Between Two Tables

```c
float morphOutput(WaveTable *tableA, WaveTable *tableB,
                  double phase, double morphPos) {
    float outA = interpolate(tableA, phase);
    float outB = interpolate(tableB, phase);
    return outA + morphPos * (outB - outA);  // morphPos ∈ [0,1]
}
```

### Multi-Table Morphing (N Tables)

```c
// morphPos ∈ [0, N-1]:
int   lowerIdx  = (int)morphPos;
float localMorph = morphPos - lowerIdx;
int   upperIdx   = min(lowerIdx + 1, N - 1);

float outLower = interpolate(tables[lowerIdx], phase);
float outUpper = interpolate(tables[upperIdx], phase);
return outLower + localMorph * (outUpper - outLower);
```

### Morphing Band-Limiting

When morphing between tables, each must be appropriately band-limited for the current pitch. Use the same mipmap level from both source tables:

```c
// Select mipmap level based on current pitch (same for both):
int mipmapLevel = selectMipmap(phaseInc);
float outA = interpolate(tableA_mips[mipmapLevel], phase);
float outB = interpolate(tableB_mips[mipmapLevel], phase);
```

---

## Serum-Style Wavetable Format

Xfer Serum popularized a specific wavetable format that has become an industry standard:

- **256 frames** per wavetable file
- **2048 samples** per frame
- Each frame is a single-cycle, band-limited waveform
- Total: 256 × 2048 = 524,288 samples per wavetable file
- Frames can be scanned via "wavetable position" parameter (morph control)
- Format: typically 32-bit float, stored as WAV (2048 × 256 samples = single long WAV)

### Frame Selection

```c
// Wavetable position ∈ [0.0, 255.0]:
int   frameA = (int)wtPosition;
float frac   = wtPosition - frameA;
int   frameB = min(frameA + 1, 255);

float *tableA = wavetable + frameA * 2048;
float *tableB = wavetable + frameB * 2048;
output = interpolate(tableA, phase) * (1.0f - frac)
       + interpolate(tableB, phase) * frac;
```

---

## Phase Distortion Synthesis (Casio CZ)

Used in Casio CZ series synthesizers. Instead of reading a table at a uniform phase rate, the phase is run through a nonlinear remapping function before table lookup.

### Basic Phase Distortion

```c
// distortedPhase: remaps [0,1] → [0,1] with a "kink"
double phaseDistort(double phase, double cutoffPos) {
    if (phase < cutoffPos)
        return phase * 0.5 / cutoffPos;          // slow rise to 0.5
    else
        return 0.5 + (phase - cutoffPos) * 0.5 / (1.0 - cutoffPos);  // fast rise to 1.0
}

// Apply to sine table:
double distortedPh = phaseDistort(phase, cutoff);
output = sin(2.0 * M_PI * distortedPh);
```

- `cutoffPos = 0.5`: symmetric → pure sine
- `cutoffPos < 0.5`: fast rise, slow fall → brighter (more harmonic content)
- `cutoffPos → 0.0`: approaches sawtooth character

### CZ Waveform Types

The CZ synthesizers supported 8 waveform types using combinations of phase distortion applied to half-cycles:
- Saw, Square, Pulse, Double-Sine, Sawtooth-Pulse, Resonance 1/2/3
- All generated via phase distortion of a sine table (not raw waveforms)

---

## Table Size Tradeoffs

| Table Size | Memory/Table | Harmonics (Nyquist) | Linear Interp Quality | Notes                    |
|------------|-------------|---------------------|----------------------|--------------------------|
| 64         | 256 B       | 32                  | Audible degradation  | Vintage hardware style   |
| 256        | 1 KB        | 128                 | Fair                 | Early wavetable synths   |
| 512        | 2 KB        | 256                 | Good at low freq     | Reasonable minimum       |
| 1024       | 4 KB        | 512                 | Very good            | Standard software synth  |
| 2048       | 8 KB        | 1024                | Excellent            | Modern standard (Serum)  |
| 4096       | 16 KB       | 2048                | Near-perfect         | Sinc interp at this size |

With 2048-sample tables and linear interpolation, SNR is approximately 90 dB. With cubic interpolation, SNR exceeds 100 dB.

---

## Practical Implementation Notes

1. **Always use power-of-two table sizes**: enables fast index wrapping with bitmasking
   ```c
   // Fast wrap with bitmask (tableSize must be power of 2):
   int mask = tableSize - 1;
   i = i & mask;  // instead of i % tableSize
   ```

2. **Guard point**: add one extra sample at the end equal to `table[0]` to avoid branch in linear interpolation:
   ```c
   table[tableSize] = table[0];  // guard
   // Now: output = table[i] + frac * (table[i+1] - table[i])  (no wrap check needed)
   ```

3. **Normalized frequency vs. Hz**: always work in `phaseInc = freq/sampleRate` internally; convert from Hz only at the parameter input boundary

4. **Denormalization**: ensure wavetable values don't generate denormal floats near zero; use DC offset flush or `_MM_SET_FLUSH_ZERO_MODE`

5. **CPU cache**: keep frequently-used mipmap levels (highest frequency tables are smallest) hot in L1 cache; they are accessed most often for high-pitched notes
