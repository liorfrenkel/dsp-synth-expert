---
title: "Additive Synthesis"
category: "Synth Architecture"
tags: [additive, harmonics, Fourier, partial, Hammond, drawbar, resynthesis]
sources:
  - https://en.wikipedia.org/wiki/Additive_synthesis
  - https://en.wikipedia.org/wiki/Fourier_series
  - Hammond organ documentation
---

## Overview

Additive synthesis constructs sounds by summing individual sine wave oscillators (partials), each with independent frequency, amplitude, and phase control. It is the inverse of subtractive synthesis: instead of removing harmonics with a filter, you explicitly specify which harmonics to include and at what level.

**Foundation**: Joseph Fourier proved (1822) that any periodic waveform can be expressed as a sum of sinusoids. Additive synthesis applies this in reverse: build any desired spectrum by summing sinusoids.

**Historical instruments**: Hammond organ (1935), Telharmonium (1906), Novachord (1938)  
**Modern implementations**: Kawai K5000, Additive software synths (Cameleon 5000, Harmor, VirSyn Cantor), IRCAM tools

---

## Fourier Series: The Mathematical Foundation

Any periodic signal `f(t)` with period `T = 1/f₀` can be written as:

```
f(t) = a₀/2 + Σ[n=1..∞] (aₙ · cos(2π·n·f₀·t) + bₙ · sin(2π·n·f₀·t))
```

Or in amplitude-phase form:

```
f(t) = A₀ + Σ[n=1..∞] Aₙ · sin(2π·n·f₀·t + φₙ)
```

Where:
- `f₀` = fundamental frequency (Hz)
- `n` = harmonic number
- `Aₙ` = amplitude of nth harmonic
- `φₙ` = phase of nth harmonic
- `A₀` = DC offset (usually 0 for audio)

**Key insight for synthesis**: The phase `φₙ` has minimal perceptual effect on timbre for steady-state sounds (though it matters for transients and certain synthesis artifacts). Practical additive synthesis focuses on controlling amplitudes `Aₙ`.

---

## Harmonic Amplitudes for Standard Waveforms

### Sawtooth Wave (Ramp Down)

Formula: `Aₙ = A₁ / n`  (amplitude inversely proportional to harmonic number)

| Harmonic n | Amplitude Aₙ | Relative level (dB) |
|------------|-------------|---------------------|
| 1 (fundamental) | 1.000 | 0.0 dB |
| 2 | 0.500 | −6.0 dB |
| 3 | 0.333 | −9.5 dB |
| 4 | 0.250 | −12.0 dB |
| 5 | 0.200 | −14.0 dB |
| 6 | 0.167 | −15.6 dB |
| 7 | 0.143 | −16.9 dB |
| 8 | 0.125 | −18.1 dB |
| 9 | 0.111 | −19.1 dB |
| 10 | 0.100 | −20.0 dB |
| 12 | 0.083 | −21.6 dB |
| 16 | 0.063 | −24.1 dB |

All harmonics present. Spectral rolloff: −6dB per octave (1/f spectrum).

### Square Wave (50% Pulse Width)

Formula: `Aₙ = A₁ / n` for **odd** n only; even harmonics = 0

| Harmonic n | Amplitude Aₙ | Notes |
|------------|-------------|-------|
| 1 | 1.000 | Fundamental |
| 2 | 0.000 | Even — absent |
| 3 | 0.333 | 1st odd overtone |
| 4 | 0.000 | Even — absent |
| 5 | 0.200 | 2nd odd overtone |
| 6 | 0.000 | Even — absent |
| 7 | 0.143 | 3rd odd overtone |
| 8 | 0.000 | Even — absent |
| 9 | 0.111 | 4th odd overtone |
| 11 | 0.091 | — |
| 13 | 0.077 | — |

Only odd harmonics. Spectral rolloff: −6dB per octave for present harmonics.

### Triangle Wave

Formula: `Aₙ = A₁ / n²` for **odd** n only; even harmonics = 0; alternating phase signs

| Harmonic n | Amplitude Aₙ | Notes |
|------------|-------------|-------|
| 1 | 1.000 | Fundamental |
| 2 | 0.000 | Even — absent |
| 3 | 0.111 | 1st odd overtone (−19.1 dB) |
| 4 | 0.000 | Even — absent |
| 5 | 0.040 | 2nd odd overtone (−28.0 dB) |
| 7 | 0.020 | (−33.9 dB) |
| 9 | 0.012 | (−38.4 dB) |
| 11 | 0.008 | (−42.0 dB) |

Odd harmonics only, rolloff −12dB/octave. Much softer than square wave. Alternating phase: `(−1)^((n−1)/2)` for harmonic n.

### Pulse Wave (25% Duty Cycle)

Formula: `Aₙ = (2/nπ) · sin(n·π·D)` where D = 0.25

| Harmonic n | Amplitude | Notes |
|------------|-----------|-------|
| 1 | 0.900 | Fundamental |
| 2 | 0.636 | Strong 2nd harmonic |
| 3 | 0.300 | Present |
| 4 | 0.000 | Absent (4×0.25 = integer) |
| 5 | 0.180 | Present |
| 6 | 0.212 | Present |
| 7 | 0.128 | Present |
| 8 | 0.000 | Absent (8×0.25 = integer) |
| 9 | 0.100 | Present |

Every 4th harmonic absent (multiples of 1/D = 4). Nasal, oboe-like timbre.

### Typical Real Instrument Harmonic Structure

**Cello (bowed, A3 = 220Hz, mezzo forte):**
| Harmonic | Relative Level (dB) |
|----------|---------------------|
| 1 | 0 (reference) |
| 2 | −3.5 |
| 3 | −6.0 |
| 4 | −8.0 |
| 5 | −12.0 |
| 6 | −14.0 |
| 7 | −18.0 |

**Clarinet (A4 = 440Hz, mezzo forte):**
| Harmonic | Relative Level (dB) |
|----------|---------------------|
| 1 | 0 |
| 2 | −20 |
| 3 | −5 |
| 4 | −25 |
| 5 | −12 |
| 6 | −28 |
| 7 | −17 |

Strong odd-harmonic dominance (cylindrical bore instrument, closed at one end).

---

## Hammond Organ Model

The Hammond organ (1935, Laurens Hammond) is the canonical additive synthesis instrument, using 91 tonewheels (spinning metal discs near electromagnetic pickups) generating sine waves.

### Drawbar System

9 drawbars per manual, each controlling a different harmonic (called **footages**, related to pipe organ lengths):

| Drawbar | Footage | Harmonic (vs 8') | Interval | Color |
|---------|---------|-----------------|----------|-------|
| 1 | 16' | Sub-octave (0.5×) | 1 octave below | Brown (left) |
| 2 | 5⅓' | Sub-quint (1.5×) | Octave+5th below | Brown |
| 3 | 8' | Fundamental (1×) | Concert pitch | White |
| 4 | 4' | Octave (2×) | 1 octave above | White |
| 5 | 2⅔' | Nazard (3×) | 2 octaves+5th | Black |
| 6 | 2' | Super octave (4×) | 2 octaves | White |
| 7 | 1⅗' | Tierce (5×) | 2 octaves+3rd | Black |
| 8 | 1⅓' | Larigot (6×) | 2 octaves+5th' | Black |
| 9 | 1' | Sifflöte (8×) | 3 octaves | White |

Each drawbar: 0 (silent) to 8 (full level, +0dB for that partial)  
9 drawbars → 9 digits form a "registration" (e.g., "888000000")

### Classic Hammond Registrations

```
Full organ (all out):    888888888   — maximum harmonics, very bright
Full organ (bass-heavy): 876543210   — traditional cathedral
Jazz (jazz organ):       888800000   — loud fundamentals, clean low-end
Rock/Gospel:             888888508   — all on + emphasized octave
Gospel/R&B:              808000000   — sub + fundamental only, dark
"Jimmy Smith" jazz:      886000040   — warm with added fifth
Gospel (bright):         888888088   — percussive high end
Boogie (gospel runs):    888800088   — rich lows + sparkle highs
Soft lead:               008080000   — subtle 4' octave melody
Flute solo:              004600000   — 4' + 2⅔' = breathy flute color
```

### Hammond Percussion
- Adds a fast-decaying click at start of each note (2nd or 3rd harmonic)
- Percussion decay: Fast (~0.24s) or Slow (~1.0s)
- Percussion volume: Soft or Normal
- Percussion only triggers on legato notes (no retrigger while keys held)
- Creates the characteristic "pop" of classic Hammond recordings

### Hammond Vibrato/Chorus Scanner
- Mechanical vibrato/chorus via rotating scanning drum
- C1–C3: 3 chorus/vibrato settings each
- Depth: ±±25 cents vibrato, or amplitude variation (chorus)
- All-digital replications: Leslie simulator plugins, hardware Leslies

---

## Additive Oscillator Bank Design

An N-partial additive synthesizer maintains an oscillator bank:

```cpp
struct Partial {
    float frequency;    // Hz
    float amplitude;    // 0–1
    float phase;        // radians, 0–2π
    float freqRate;     // phase increment per sample = freq / sampleRate
};

struct AdditiveSynth {
    int numPartials;
    Partial partials[MAX_PARTIALS];
    float sampleRate;
    
    float nextSample() {
        float output = 0.0f;
        for (int i = 0; i < numPartials; i++) {
            output += partials[i].amplitude * sinf(partials[i].phase);
            partials[i].phase += partials[i].freqRate;
            if (partials[i].phase >= 2.0f * M_PI)
                partials[i].phase -= 2.0f * M_PI;
        }
        return output;
    }
    
    void setFundamental(float f0) {
        for (int i = 0; i < numPartials; i++) {
            partials[i].frequency = f0 * (i + 1);
            partials[i].freqRate = (2.0f * M_PI * partials[i].frequency) / sampleRate;
        }
    }
    
    void setHarmonicProfile(float* amplitudes) {
        for (int i = 0; i < numPartials; i++)
            partials[i].amplitude = amplitudes[i];
    }
};
```

### Anti-Aliasing in Additive Synthesis
- Only include partials below Nyquist: `partial_freq < sampleRate / 2`
- For sawtooth: at f0 = 1kHz, sr = 44100Hz → max 22 partials
- Dynamically mute upper partials as fundamental rises
- Alternatively: use bandlimited oscillators (PolyBLEP, BLIT) for standard waveforms

---

## Computational Cost

**Naive additive synthesis**: each partial requires:
- 1 phase increment (multiply + add)
- 1 sine computation (most expensive operation)
- 1 amplitude scale (multiply)
- 1 accumulation (add)

**Cost estimate:**  
- 64 partials × 44100 samples/sec = 2.8 million sine evaluations/second
- Modern CPU: ~100–500 million flops/second → feasible but not trivial
- Lookup table sine: much faster (~10× speedup)
- SIMD (AVX/SSE): process 8 partials in parallel

**IFFT-based additive (more efficient for dense spectra):**
```
1. Fill frequency-domain buffer: bin[n] = amplitude[n] * e^(j*phase[n])
2. Run IFFT: output_block = IFFT(freq_buffer)
3. Output each sample of the block
```
IFFT of N points costs O(N log N) vs. O(N×samples) for direct synthesis.  
For 4096-sample buffer, 2048 partials: FFT approach is ~12× faster.

---

## Partial Tracking (Analysis-Synthesis)

Partial tracking decomposes a recorded sound into sinusoidal partials + residual noise:

```
Input audio
    ↓
STFT (Short-Time Fourier Transform)
    ↓
Peak picking per frame (find local spectral maxima)
    ↓
Partial tracking (connect peaks across frames by frequency proximity)
    ↓
Sinusoidal partials: each with frequency(t), amplitude(t), phase(t)
    +
Residual (stochastic noise component)
    ↓
Re-synthesis: sum sinusoids + add filtered noise
```

**Applications:**
- Time-stretching without pitch change (stretch time axis of partials)
- Pitch-shifting without timing change (multiply all frequencies by ratio)
- Spectral morphing: crossfade between two partial sets
- Sound "cloning": extract spectrum, modify, resynthesize
- Tools: SPEAR (Sinusoidal Partial Editing and Resynthesis), AudioSculpt, Loris, IRCAM SuperVP

---

## Spectral Morphing

Crossfading between two harmonic spectra A and B:

```
For each harmonic n:
    amplitude_out[n] = (1 - alpha) * amplitude_A[n] + alpha * amplitude_B[n]

Where alpha = 0 → pure A, alpha = 1 → pure B
```

**Frequency morphing** (also interpolate partial frequencies):
```
freq_out[n] = (1 - alpha) * freq_A[n] + alpha * freq_B[n]
```

This transitions between the harmonic series of A and B, passing through intermediate (sometimes inharmonic) states.

**Equal-power morphing** (avoids loudness dip at midpoint):
```
amplitude_out[n] = sqrt((1-alpha)² * A[n]² + alpha² * B[n]²)
```

---

## IFFT-Based Additive Synthesis

Set magnitude spectrum bins, run IFFT for each output buffer:

```python
import numpy as np

def synth_block(block_size, partials, phases, sample_rate):
    """
    partials: list of (harmonic_number, amplitude) tuples
    phases: current phase for each partial
    """
    freq_domain = np.zeros(block_size // 2 + 1, dtype=complex)
    
    for harm_n, amp in partials:
        bin_index = round(harm_n * fundamental_hz * block_size / sample_rate)
        if 0 < bin_index < len(freq_domain):
            freq_domain[bin_index] = amp * np.exp(1j * phases[harm_n])
        # Update phase for next block
        phases[harm_n] += 2 * np.pi * harm_n * fundamental_hz * block_size / sample_rate
    
    # IFFT gives time-domain samples
    time_domain = np.fft.irfft(freq_domain)
    return time_domain
```

Advantages: efficient for many partials; inherently anti-aliased (no partials above Nyquist).  
Disadvantages: block-based latency; phase continuity requires careful tracking.

---

## Resynthesis Applications

**Time-stretching without pitch change:**
```
Original: partial at 440Hz, amplitude varies 0.1–0.8 over 1 second
Stretched (2×): same 440Hz, but amplitude varies over 2 seconds
Compress time: amplitude varies over 0.5 seconds
```

**Pitch-shifting without time change:**
```
All partial frequencies multiplied by shift ratio
Shift ratio = 2^(semitones/12)
E.g., +5 semitones: ratio = 2^(5/12) ≈ 1.3348
```

**Cross-synthesis:** apply amplitude envelope of sound A to spectrum of sound B
- "Speak through a string": use speech amplitude modulation on violin partials

---

## Kawai K5000 Additive Synthesizer

Commercial additive synthesizer using 128 partials per voice:

- 128 sine oscillators per voice (64 in some modes)
- Each partial: independent amplitude envelope (multiple breakpoints)
- "Source": combination of static additive + dynamic additive + PCM samples
- Additive "formant" sweeps: animate spectral shape over time
- 6-part multitimbral, 32-voice polyphony
- Spectral morphing between user-defined spectra
- Parameter count: enormous; deep but complex to program

---

## Comparison: Additive vs. Other Synthesis

| Feature | Additive | Subtractive | FM | Wavetable |
|---------|----------|-------------|-----|-----------|
| CPU cost | High (many oscillators) | Low | Low-Medium | Low |
| Spectrum control | Exact (per-partial) | Imprecise | Via β | Table-dependent |
| Natural sounds | Excellent (resynthesis) | Limited | Moderate | Moderate |
| Edit complexity | Very high | Low | Moderate | Low |
| Time-varying | Yes (partial envelopes) | Via filter sweep | Via op envelopes | Via wavetable scan |
| Hardware historical | Hammond (1935) | Minimoog (1970) | DX7 (1983) | PPG Wave (1978) |
