---
title: "Oscillator Mathematics and Bandlimited Waveforms"
category: "DSP"
tags: [oscillator, bandlimited, BLEP, PolyBLEP, wavetable, FM, phase-accumulator]
---

# Oscillator Mathematics and Bandlimited Waveforms

## Overview

A digital oscillator generates a periodic waveform at a desired frequency within a digital audio system. The central challenge is **aliasing**: naive digital waveforms (except sine) contain harmonics that exceed the Nyquist limit (Fs/2) and fold back as inharmonic artifacts. The solutions are bandlimited synthesis methods: PolyBLEP, BLIT, wavetable mipmaps, and oversampling.

---

## Phase Accumulator: The Core of Every Oscillator

All oscillators are built on a **phase accumulator** — a number that increases monotonically, representing the current phase of the waveform cycle.

```c
// Phase increment per sample:
phaseInc = freq / sampleRate;    // range: [0, 0.5]

// Phase update (called every sample):
phase += phaseInc;
if (phase >= 1.0) phase -= 1.0;  // wrap to [0, 1)

// Equivalent to:
phase = fmod(phase + phaseInc, 1.0);
```

- `phase` ranges over `[0.0, 1.0)`, representing one full cycle
- `phaseInc = 0.5` → Nyquist frequency (a sample alternating +1/-1)
- `phaseInc = 1/N` → period of N samples
- For MIDI note to frequency: `freq = 440.0 * pow(2.0, (note - 69) / 12.0)`

---

## Naive Waveform Generation

These are the mathematically simple waveforms, **without** aliasing correction. Acceptable only for:
- Subaudible LFOs (<20 Hz)
- Waveforms that will be heavily low-pass filtered immediately
- Situations where aliasing is acceptable or desired

### Sine Wave

```c
output = sin(2.0 * M_PI * phase);
// or: output = sin(2.0 * M_PI * freq * t);  // t in seconds
```

Sine contains only the fundamental. **No aliasing** regardless of frequency.

### Sawtooth Wave

```c
// Rising sawtooth (ramp up):
output = 2.0 * phase - 1.0;   // maps [0,1) → [-1, +1)

// Falling sawtooth (ramp down):
output = 1.0 - 2.0 * phase;
```

Harmonic content: `amplitude_n = 2/π * (1/n) * (-1)^(n+1)` for n = 1, 2, 3, ...
- Fundamental, all harmonics (odd and even), each falling at 6 dB/octave
- Discontinuity at `phase = 0/1.0` (the "flyback") causes aliasing

### Square Wave

```c
output = (phase < 0.5) ? 1.0 : -1.0;
// With variable pulse width (PW):
output = (phase < pulseWidth) ? 1.0 : -1.0;  // PW ∈ [0, 1]
```

Harmonic content: `amplitude_n = 4/(nπ)` for n = 1, 3, 5, 7, ... (odd harmonics only)
- Contains only odd harmonics at 6 dB/octave
- Two discontinuities per cycle — two aliasing sources

### Triangle Wave

```c
if (phase < 0.5)
    output = 4.0 * phase - 1.0;        // rising portion: [-1, +1]
else
    output = 3.0 - 4.0 * phase;        // falling portion: [+1, -1]
```

Harmonic content: `amplitude_n = 8/(n²π²) * (-1)^((n-1)/2)` for n = 1, 3, 5, ...
- Odd harmonics only, falling at 12 dB/octave (amplitude ∝ 1/n²)
- **No discontinuity**, only slope discontinuities → weaker aliasing than saw/square
- Quasi-bandlimited at higher frequencies due to rapid harmonic decay

---

## Why Naive Waveforms Alias: The Gibbs Phenomenon

### Nyquist-Shannon Theorem

A discrete-time system at sample rate `Fs` can only faithfully represent frequencies below `Fs/2` (the Nyquist frequency).

A 440 Hz sawtooth contains harmonics at 440, 880, 1320, 1760, ... Hz. At 44100 Hz sample rate (Nyquist = 22050 Hz), harmonics above 22050 Hz cannot be represented — they **fold back** (alias) to:

```
f_alias = |f_harmonic - round(f_harmonic / Fs) * Fs|
```

Example: A 440 Hz saw played at 44100 Hz sample rate:
- Harmonic 50: 22000 Hz (just under Nyquist, OK)
- Harmonic 51: 22440 Hz → aliases to |22440 - 44100| = 21660 Hz
- These alias frequencies are **inharmonic** and pitch-dependent → audible as digital harshness

### Gibbs Phenomenon

When a signal with a discontinuity (like a sawtooth jump) is reconstructed by a finite number of sinusoidal components, you get overshoot/undershoot near the discontinuity. The digital reconstruction of a naive saw does not cleanly reproduce the transition — it rings. The ringing manifests as pre-ringing and post-ringing artifacts proportional to the omitted harmonics.

### Alias Audibility by Register

- **Bass (< 200 Hz)**: aliases are far above the fundamental, usually above hearing or masked
- **Mid (200–2000 Hz)**: aliases start becoming audible, especially during filter sweeps
- **High (> 2000 Hz)**: extremely audible; naively generated saw at 5 kHz only has 4–5 clean harmonics

---

## PolyBLEP: Polynomial Bandlimited Step

PolyBLEP (Polynomial Band-Limited Step) corrects the discontinuities of naive waveforms by adding a small polynomial correction function at each discontinuity point. Fast, simple, and effective.

### Core Concept

At each discontinuity, add/subtract a residual `polyblep(t, dt)` where:
- `t` = phase at the discontinuity, normalized to `[0, 1)` locally
- `dt` = phase increment (= `freq/sampleRate`)

### PolyBLEP Residual Function

```c
double polyblep(double t, double dt) {
    // t is normalized position in [0, 1)
    // Apply correction in range [-dt, +dt] around the discontinuity
    if (t < dt) {
        t /= dt;
        return t + t - t*t - 1.0;       // rising edge correction
    } else if (t > 1.0 - dt) {
        t = (t - 1.0) / dt;
        return t*t + t + t + 1.0;       // falling edge correction
    }
    return 0.0;
}
```

### PolyBLEP Sawtooth

```c
double saw_polyblep(double phase, double phaseInc) {
    double output = 2.0 * phase - 1.0;   // naive saw
    output -= polyblep(phase, phaseInc); // correct at phase = 0 (wrap)
    return output;
}
```

### PolyBLEP Square Wave

```c
double square_polyblep(double phase, double phaseInc, double pw) {
    double output = (phase < pw) ? 1.0 : -1.0;   // naive square
    output += polyblep(phase, phaseInc);           // correct rising edge (phase=0)
    output -= polyblep(fmod(phase - pw + 1.0, 1.0), phaseInc); // correct falling edge (phase=pw)
    return output;
}
```

### PolyBLEP Quality

- Effective up to ~8 kHz with single correction (2nd-order polynomial)
- Higher-order PolyBLEP (4th-order, 6th-order) extends useful range
- At very high notes (>8 kHz), wavetable with mipmap is cleaner
- CPU cost: minimal — a few branches and multiplies per sample
- Latency: zero samples (no lookahead needed for 2nd-order)

---

## BLIT: Band-Limited Impulse Train

The Stilson/Smith BLIT method synthesizes bandlimited waveforms by summing a finite number of sinc functions (a truncated impulse train), then integrating to get the desired waveform.

### BLIT Formula

For M harmonics at frequency f₀:

```
         M-1
BLIT(t) = Σ  sinc(n * f₀ * t)  scaled appropriately
         n=0
```

In practice, the formula is:
```
BLIT(t) = (f₀/Fs) * sin(M * π * f₀ * t / Fs) / (M * sin(π * f₀ * t / Fs))
```

Where `M = floor(Fs / (2 * f₀))` = maximum number of harmonics below Nyquist.

### Waveform Integration from BLIT

- **Sawtooth**: integrate a single BLIT, add DC-blocking filter
- **Square**: subtract two BLITs offset by half-cycle
- **Triangle**: integrate a BLIT twice

```c
// BLIT sawtooth via leaky integration:
double blit = compute_blit(phase, phaseInc);
output = output * (1.0 - 0.001) + blit;  // leaky integrator
```

### BLIT vs. PolyBLEP

| Feature         | BLIT              | PolyBLEP         |
|-----------------|-------------------|------------------|
| Accuracy        | Theoretically exact| Approximate (polynomial) |
| CPU cost        | Higher (sinc sum) | Very low (polynomial) |
| Implementation  | More complex      | Simple           |
| Aliasing        | Nearly zero       | Low (worse at high freq) |
| Practical use   | Less common       | Very common      |

---

## MINBLEP: Minimum Phase BLEP Tables

MinBLEP (minimum phase band-limited step) uses a precomputed table of the minimum-phase version of the BLEP residual. This allows zero-latency correction with better high-frequency accuracy than PolyBLEP.

```c
// Typical MINBLEP table: 2048 samples, 64x oversampled, Hann-windowed
// Lookup at discontinuity, add fractional portion of table:
for (int i = 0; i < MINBLEP_SIZE; i++) {
    output_buffer[pos + i] += minblep_table[i] * step_size;
}
```

- MINBLEP tables are computed once and stored
- Higher quality than PolyBLEP for complex waveforms
- Used in high-quality soft synths (e.g., Surge synthesizer)

---

## Hardsync

Hard sync resets the slave oscillator's phase to 0 when the master oscillator completes a cycle. This creates complex, buzzy timbres characteristic of classic synthesizer sounds (e.g., Minimoog solos).

### Math

```c
// Master oscillator phase:
masterPhase += masterPhaseInc;
if (masterPhase >= 1.0) {
    masterPhase -= 1.0;
    slavePhase = 0.0;   // HARD SYNC: reset slave phase
}

// Slave generates output from its own phase:
slavePhase += slavePhaseInc;
if (slavePhase >= 1.0) slavePhase -= 1.0;  // ignored at sync point
output = saw(slavePhase);  // or any waveform
```

### Timbral Effect

- When slave frequency > master: creates multiple "restarts" per master cycle
- Produces a spectrum with components at `f_master ± n * f_slave`
- Ratio control (slave/master ratio) sweeps through very different timbres
- Classic use: set master to note pitch, sweep slave pitch for lead synth textures

### Hardsync PolyBLEP Correction

The sync reset introduces a discontinuity that must be corrected:
```c
// At the sync reset point, compute exact sub-sample position:
double t = (masterPhase - 1.0) / masterPhaseInc;  // fraction [0,1) into current sample
// Apply blep residual to correct the discontinuity in slave output
applyBlepAtPosition(slaveOutput, t, slaveValue - 0.0);  // step = from current value to reset value
```

---

## Softsync

Softsync resets the slave oscillator to half-phase (instead of zero), creating gentler timbral effects. Less commonly implemented:

```c
if (masterPhase >= 1.0) {
    masterPhase -= 1.0;
    slavePhase = 0.5;   // SOFT SYNC: reset to midpoint
}
```

---

## FM Synthesis (Frequency Modulation)

### Basic FM Equation

```
output(t) = A * sin(2π * fc * t + β * sin(2π * fm * t))
```

Where:
- `fc` = carrier frequency
- `fm` = modulator frequency
- `β` = modulation index = `Δf / fm` (peak frequency deviation divided by modulator frequency)
- `A` = carrier amplitude

### Modulation Index and Sidebands

Sidebands appear at `fc ± n*fm` for n = 1, 2, 3, ...

Amplitude of sideband n is given by Bessel function `Jn(β)`:
- `β = 0`: pure carrier, no sidebands
- `β = 1`: 1–2 significant sideband pairs, mellow
- `β = 5`: rich, bright spectrum with many sidebands
- `β = 10+`: extremely bright, inharmonic potential

### C/Ratio Relationship

The harmonic relationship between carrier and modulator:
```
C:M ratio determines spectral character:
1:1  → harmonic series, complex tone
2:1  → every other harmonic (like square wave character)
1:2  → sub-harmonic emphasis
3:2  → non-integer, metallic/bell tones
1:√2 → strongly inharmonic, metallic
```

### DX7-Style FM Implementation (Phase Accumulator)

```c
// Modulator:
modPhase += modFreq / sampleRate;
if (modPhase >= 1.0) modPhase -= 1.0;
modOutput = sin(2.0 * M_PI * modPhase) * modulationDepth;

// Carrier (modulated by modulator):
carPhase += carFreq / sampleRate;
if (carPhase >= 1.0) carPhase -= 1.0;
output = sin(2.0 * M_PI * carPhase + modOutput * M_PI * 2.0);
```

### FM Feedback

Self-modulation of an operator:
```c
// Operator with feedback:
feedback = (prevOutput1 + prevOutput2) * 0.5 * feedbackLevel;
output   = sin(2.0 * M_PI * phase + feedback);
prevOutput2 = prevOutput1;
prevOutput1 = output;
```
Feedback produces sawtooth-like spectra as level increases.

### Envelope-Controlled Modulation Index

The classic DX7 "bark" envelope: high beta at attack, decaying to low beta:
```
β(t) = β_start * env(t) + β_end * (1 - env(t))
```

---

## Oscillator Reset (Phase Reset)

### Use Cases

- **Note-on sync**: reset phase to 0 at note start for consistent attack transients
- **Hard sync**: described above
- **Retrigger LFOs**: reset LFO phase on note on

```c
void reset(double newPhase = 0.0) {
    phase = newPhase;
}
```

Phase reset mid-cycle introduces a discontinuity. Apply PolyBLEP or cross-fade to avoid click:
```c
// Crossfade over N samples for soft reset:
for (int i = 0; i < fadeLen; i++) {
    double alpha = (double)i / fadeLen;
    buf[pos + i] = alpha * newOsc(i) + (1.0 - alpha) * oldOsc(i);
}
```

---

## Wavetable Phase Increment

```c
// Phase increment for wavetable playback:
phaseInc = tableSize * freq / sampleRate;

// Phase update:
phase += phaseInc;
if (phase >= tableSize) phase -= tableSize;

// Linear interpolation (from table):
int   i    = (int)phase;
double frac = phase - i;
int   j    = (i + 1) % tableSize;   // wrap
output     = table[i] + frac * (table[j] - table[i]);
```

---

## Cubic (4-Point Hermite) Interpolation

Higher quality than linear interpolation, especially at low table sizes:

```c
double hermite4(double frac, double y0, double y1, double y2, double y3) {
    // y0 = sample before, y1/y2 = bracketing samples, y3 = sample after
    double c0 = y1;
    double c1 = 0.5 * (y2 - y0);
    double c2 = y0 - 2.5*y1 + 2.0*y2 - 0.5*y3;
    double c3 = 0.5 * (y3 - y0) + 1.5 * (y1 - y2);
    return ((c3 * frac + c2) * frac + c1) * frac + c0;
}
```

Usage in wavetable:
```c
int   i  = (int)phase;
double t  = phase - i;
double y0 = table[(i - 1 + tableSize) % tableSize];
double y1 = table[i % tableSize];
double y2 = table[(i + 1) % tableSize];
double y3 = table[(i + 2) % tableSize];
output = hermite4(t, y0, y1, y2, y3);
```

Hermite interpolation has flat frequency response up to about 0.35 * Nyquist, good enough for most wavetable applications.

---

## Classic Waveform Harmonic Series

### Sawtooth Wave Harmonics

```
saw(t) = 2/π * Σ (-1)^(n+1) / n * sin(2π*n*f₀*t)
                n=1..∞

Harmonic:   1    2    3    4    5    ...
Amplitude: 1.0  0.5  0.33  0.25  0.20  ...
Phase:      0    π    0    π    0    ...
```

### Square Wave Harmonics

```
square(t) = 4/π * Σ 1/n * sin(2π*n*f₀*t)
                  n=1,3,5,...

Harmonic:   1    3    5    7    9    ...
Amplitude: 1.27  0.42  0.25  0.18  0.14  ...
```

### Triangle Wave Harmonics

```
tri(t) = 8/π² * Σ (-1)^((n-1)/2) / n² * sin(2π*n*f₀*t)
                n=1,3,5,...

Harmonic:   1    3    5    7    9    ...
Amplitude: 0.81  0.09  0.03  0.016  0.010  ...
```

Note the 1/n² amplitude decay — much faster than saw or square, explaining the softer sound.

---

## Pulse Width Modulation (PWM)

```c
// PWM via two detuned sawtooth oscillators:
// osc1 and osc2 are identical saws, phase offset by pulseWidth

output = saw(phase) - saw(phase - pulseWidth + (pulseWidth > phase ? 1.0 : 0.0));

// Or equivalently (from phase accumulator with offset):
output1 = 2.0 * phase - 1.0;
output2 = 2.0 * fmod(phase - pulseWidth + 1.0, 1.0) - 1.0;
output  = (output1 - output2) * 0.5;  // normalize
```

PWM creates a time-varying duty cycle, producing chorus-like animation when the pulse width is slowly modulated by an LFO.

---

## Anti-Aliasing Method Quick Reference

| Method         | CPU Cost | Quality | Best Use              |
|----------------|----------|---------|------------------------|
| Naive          | Minimal  | Bad     | Sub-audio LFOs only   |
| PolyBLEP       | Low      | Good    | General purpose synths|
| MinBLEP        | Low-Med  | Better  | High-quality synths   |
| BLIT + integrate| Medium  | Good    | Saw/square            |
| Wavetable mipmap| Low     | Excellent| All waveform types   |
| Oversampling 4x | High    | Excellent| Simple shapes        |
| Oversampling 8x | Very High| Best   | Complex/FM synthesis  |
