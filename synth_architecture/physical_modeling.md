---
title: "Physical Modeling Synthesis"
category: "Synth Architecture"
tags: [physical-modeling, Karplus-Strong, waveguide, modal, string, resonator]
sources:
  - https://en.wikipedia.org/wiki/Physical_modelling_synthesis
  - Julius O. Smith III CCRMA publications
  - Karplus & Strong (1983). "Digital Synthesis of Plucked String and Drum Timbres"
  - Yamaha VL1 documentation
---

## Overview

Physical modeling synthesis simulates the physics of acoustic instruments by modeling the actual mechanical behavior of vibrating objects, resonant cavities, and excitation mechanisms. Rather than recording, filtering, or summing sine waves, it computes what a physical system would actually produce.

**Core distinction**: the model processes excitation energy through a physics simulation of a resonant body.

**Historical milestone**: Karplus-Strong algorithm (1983), then generalized by Julius O. Smith III into **Digital Waveguide Synthesis** at CCRMA/Stanford in the mid-1980s.  
**First commercial waveguide synthesizer**: Yamaha VL1 (1994)  
**Key researchers**: Kevin Karplus, Alex Strong, Julius O. Smith III, Perry Cook, James McIntyre, John Woodhouse

---

## Karplus-Strong String Algorithm

The simplest and most elegant physical modeling algorithm. Models a plucked string.

### Basic Algorithm

```cpp
// Karplus-Strong plucked string
// N = delay line length = sampleRate / frequency (rounded to integer)
// b = feedback coefficient (controls decay; 0 < b ≤ 1)
// g = lowpass averaging coefficient (0.5 = maximum averaging)

int N = (int)round(sampleRate / frequency);  // delay line length
float delayLine[N];

// Step 1: Initialize delay line with noise (the "pluck")
for (int i = 0; i < N; i++) {
    delayLine[i] = (rand() / (float)RAND_MAX) * 2.0f - 1.0f;  // random in [-1, 1]
}

int readPtr  = 0;
int writePtr = 0;

// Step 2: Main loop — each sample
float nextSample() {
    // Read current output
    float output = delayLine[readPtr];
    
    // Compute next value: average of current + next sample (lowpass filter in loop)
    int nextPtr = (readPtr + 1) % N;
    float averaged = b * 0.5f * (delayLine[readPtr] + delayLine[nextPtr]);
    
    // Write back with feedback
    delayLine[writePtr] = averaged;
    
    // Advance pointers
    readPtr  = (readPtr  + 1) % N;
    writePtr = (writePtr + 1) % N;
    
    return output;
}
```

### How It Works
1. The delay line stores one period of the waveform
2. Each iteration, the output is the current value, and the new value is the average of two adjacent samples
3. The averaging acts as a **one-pole lowpass filter** — high frequencies decay faster
4. This mimics the physics: a string's high-frequency modes lose energy (to friction, air drag, bridge coupling) faster than low-frequency modes
5. The delay loop creates the **periodicity** (pitch)

### Parameter Effects

**Frequency tuning:**
```
N = sampleRate / frequency
Exact frequency: fHz = sampleRate / N (integer N only → quantized pitch)
Fine tuning: use fractional delay line (allpass interpolation for non-integer N)
```

**Decay time:**
```
b = feedback coefficient
b = 1.0:   no decay (sustain forever — physical impossibility but useful)
b = 0.999: very slow decay (≈3–10 seconds)
b = 0.99:  moderate decay (≈0.3–1 second)
b = 0.95:  fast decay (pluck of short-sustain object)
b = 0.5:   very fast decay

Approximate decay formula: T60 = -3*N / (sampleRate * log10(b))
```

**Lowpass coefficient g (averaging blend):**
```
g = 0.5:  maximum lowpass (most realistic bright plucked string)
g = 0.4–0.49: slightly less aggressive lowpass
g > 0.5: NOT typically used (stability requires g ≤ 0.5 for basic KS)
```

### Karplus-Strong Variations

**Hard pluck** (uniform random initial conditions):
```
for (i) delayLine[i] = random(-1, 1)
```

**Soft pluck** (sine or triangle initial conditions):
```
for (i) delayLine[i] = sin(π * i / N) * random_amplitude
```

**Guitar body coupling**: convolve output with impulse response of guitar body.

**Stretched string** (inharmonic overtones for piano-like sound):  
Insert an allpass filter in the delay loop. Allpass delays high frequencies more than low frequencies, slightly stretching the harmonic series:
```cpp
// Allpass filter in feedback loop:
float allpass(float input, float coeff, float* state) {
    float output = coeff * input + *state;
    *state = input - coeff * output;
    return output;
}
// Insert between read and write in KS loop
// coeff ≈ 0.1–0.5 for slight to extreme inharmonicity
```

**Drum membrane** (Karplus-Strong + sign randomization):
```cpp
// Probability p of sign flip at each step → aperiodic → drum-like
float averaged = b * 0.5f * (delayLine[readPtr] + delayLine[nextPtr]);
if ((rand() / (float)RAND_MAX) < p) averaged = -averaged;
delayLine[writePtr] = averaged;
```

**Bowed string**: replace random noise initialization with continuous bow excitation  
Each sample: mix friction nonlinearity output into delay line instead of running free

---

## Digital Waveguide Synthesis

Generalization of Karplus-Strong by Julius O. Smith III. Models acoustic wave propagation in one-dimensional media (strings, bores).

### Bidirectional Delay Lines

A string supports both left-traveling and right-traveling waves:

```
                  ← y⁻(t, x) (left-traveling)
────────────────────────────────── string
  y⁺(t, x) → (right-traveling)
```

Each direction modeled by a delay line. At the endpoints (bridge, nut), waves reflect with boundary conditions:

```
y_bridge(t) = −y⁺(t, bridge) + y⁻(t, bridge)  [rigid boundary: y=0]
// Reflection inverts sign
```

**Lossy reflection** (physical absorption at endpoints):
```
y_bridge_new = reflection_coeff * y_bridge_old
// |reflection_coeff| < 1 → energy absorption → decay
```

### Scattering Junctions

When two waveguides meet (e.g., string joins string), energy is distributed according to impedance:

```cpp
// Two waveguides meeting, impedances Z1 and Z2
float junction(float wave_right, float wave_left, float Z1, float Z2) {
    float total_Z = Z1 + Z2;
    float reflected_1 = ((Z2 - Z1) / total_Z) * wave_right + (2*Z2 / total_Z) * wave_left;
    float reflected_2 = ((Z1 - Z2) / total_Z) * wave_left  + (2*Z1 / total_Z) * wave_right;
    // Return both reflected waves
}
```

This correctly models bow-string contact point, tone hole openings in woodwinds, bridge coupling.

### Applications of Digital Waveguide

- **Plucked string**: single loop delay line with lowpass at reflection
- **Bowed string**: nonlinear friction model at excitation point + bidirectional delays
- **Flute/recorder**: jet nonlinearity + cylindrical bore (open both ends)
- **Clarinet**: reed nonlinearity + cylindrical bore (closed-open)
- **Saxophone**: reed nonlinearity + conical bore (requires special treatment)
- **Trumpet/trombone**: lip oscillation + conical/cylindrical bore + bell radiation

---

## Modal Synthesis

Represents a resonant object as a bank of damped resonant modes (eigenfrequencies). Each mode is a biquad bandpass filter excited by an impulse.

### Theory

Any linear resonant object can be characterized by its **mode frequencies**, **mode amplitudes**, and **mode decay rates**:

```
output(t) = Σ[k=1..K] Ak * e^(-dk*t) * cos(2π*fk*t + φk)
```

Where:
- `fk` = frequency of mode k (Hz)
- `Ak` = initial amplitude of mode k (depends on excitation location)
- `dk` = decay rate of mode k (modes with higher k typically decay faster)
- `K` = total number of modes

### Implementation: Biquad Resonator Bank

```cpp
// Second-order (biquad) resonator for one mode:
// H(z) = b0 / (1 - 2*r*cos(ω₀)*z⁻¹ + r²*z⁻²)
// Where: ω₀ = 2π*fk/sr, r = e^(-dk/sr) = decay per sample

struct ModalResonator {
    float r;      // pole radius (decay): r = e^(-decay_rate / sample_rate)
    float omega;  // mode frequency: omega = 2*pi*freq / sample_rate
    float a1, a2; // IIR coefficients
    float b0;     // gain
    float z1, z2; // state
    
    void set(float freq, float decayRate, float amplitude, float sampleRate) {
        omega = 2.0f * M_PI * freq / sampleRate;
        r     = expf(-decayRate / sampleRate);
        a1    = -2.0f * r * cosf(omega);
        a2    = r * r;
        b0    = amplitude;
        z1 = z2 = 0.0f;
    }
    
    float process(float input) {
        float output = b0 * input - a1 * z1 - a2 * z2;
        z2 = z1;
        z1 = output;
        return output;
    }
};

// To process a "strike":
// Send a single impulse (1.0, followed by 0.0s) through all resonators and sum outputs
float strikeOutput(float* impulse, int numSamples) {
    float output = 0.0f;
    for (int i = 0; i < numModes; i++)
        output += resonators[i].process(impulse[currentSample]);
    return output;
}
```

### Mode Frequencies of Real Objects

**Ideal string** (harmonic):
```
fk = k * f1    (integer multiples)
f1 = sqrt(T / μ) / (2L)   where T=tension, μ=mass/length, L=string length
```

**Stiff string** (piano wire) — slightly inharmonic:
```
fk = k * f1 * sqrt(1 + B*k²)
B = inharmonicity coefficient (higher for thicker, shorter strings)
```

**Circular membrane (drum head)**:
```
Mode frequencies are NOT integer multiples (Bessel function zeros)
f(m,n) = α(m,n) * c / (2π*a)
Where: a=radius, c=wave speed, α(m,n)=mth zero of Jn (Bessel function)
First few: α(0,1)=2.405, α(1,1)=3.832, α(2,1)=5.136, α(0,2)=5.520...
Ratio ≈ 1 : 1.59 : 2.14 : 2.30 (inharmonic!)
```

**Bell/Bar** (metallophone key):
```
First four mode ratios: 1 : 2.76 : 5.40 : 8.93 (highly inharmonic)
Very different from string or membrane
```

### Commuted Synthesis

Computing the full excitation-body system is expensive. **Commuted synthesis** moves the body resonance into the excitation signal:

```
Standard:  excitation → physical_body_model → output
Commuted:  (excitation * body_IR) → resonator_only → output
```

Because convolution is commutative, this is mathematically identical but computationally cheaper when the body impulse response is pre-computed and stored. Used in: acoustic guitar synthesis (pluck with a pre-convolved excitation containing guitar body response).

---

## Vocal Tract Model

### Source-Filter Model of the Voice

The voice = excitation source × vocal tract filter:

```
Vocal cords (source):
  - Periodic buzz (voiced speech): harmonic spectrum with 1/f rolloff
  - Noise (unvoiced): white noise
  - Mixed (breathiness): both

Vocal tract (filter):
  - Resonant tube of variable cross-section
  - Resonances = formants (F1, F2, F3...)
  - F1: 300–1000Hz (tongue height: low vowels have high F1)
  - F2: 900–2800Hz (tongue position: front vowels have high F2)
  - F3: 2000–3500Hz (lip rounding, r-sounds)
```

### Tube Model (Waveguide Vocal Tract)

Vocal tract modeled as a series of cylindrical tube sections:

```
     Glottis   Pharynx    Oral tract   Lips
     ──────────────────────────────────── >>>
     [A0][A1][A2][A3]...[AN]
```

Each section characterized by cross-sectional area A(x). Reflection coefficients at each junction:

```
k_i = (A_i+1 - A_i) / (A_i+1 + A_i)    (reflection coefficient)
```

This is equivalent to a **lattice filter** (also used in LPC speech coding).

### LPC (Linear Predictive Coding)

LPC analyzes a speech signal and extracts vocal tract filter coefficients:

```python
import numpy as np

def lpc_coefficients(x, order):
    """Compute LPC coefficients of order p for signal x."""
    # Autocorrelation method (Levinson-Durbin algorithm)
    R = np.correlate(x, x, mode='full')
    R = R[len(x)-1:]  # take non-negative lags only
    
    # Levinson-Durbin recursion
    a = np.zeros(order)
    k = np.zeros(order)
    E = R[0]
    
    for i in range(order):
        k[i] = -R[i+1] / E
        for j in range(i):
            k[i] -= a[j] * R[i-j]
        k[i] /= E
        
        a_new = a.copy()
        for j in range(i):
            a_new[j] += k[i] * a[i-j-1]
        a_new[i] = k[i]
        a = a_new
        E *= (1 - k[i]**2)
    
    return a, E
```

---

## Wind Instrument Models

### Clarinet (Reed + Cylindrical Bore)

```
                  Reed (nonlinear)
                       │
Mouth pressure ─────→ ┤ ←── bore pressure
                       │
                  ┌────┴────┐
                  │ Bore    │ (cylindrical waveguide, closed-open)
                  │ delay   │
                  └────┬────┘
                       │ (open end reflection + radiation)
                     output
```

**Reed nonlinearity**: 
```cpp
// Simplified clarinet reed function
// pm = mouth pressure, p = bore pressure at mouthpiece
float deltaP = pm - p;
float Ur;  // volume flow through reed
if (deltaP >= pm_threshold) {
    Ur = 0.0f;  // reed closed
} else {
    Ur = sign(deltaP) * sqrt(abs(deltaP)) * reed_opening;
}
```

Cylindrical bore produces **odd harmonics only** (closed-open boundary conditions), which is why clarinets sound hollow/narrow.

### Flute (Jet + Open Bore)

```
Embouchure hole (jet) → bore (open both ends)
                              
Jet oscillation: vjet = velocity * sin(φ)
φ = 2π * jet_delay * f
Nonlinear: jet flips between in-bore and out-bore at threshold
```

### Brass (Lip Oscillation + Bell)

```
Lips → cylindrical tube → bell flare → radiation
  
Lip model (valve oscillator):
  Δp = pressure difference across lips
  if Δp > closing_pressure: lips open
  if Δp < closing_pressure: lips close
  Volume flow ∝ lip_opening * sqrt(Δp)
```

Bell radiation: frequency-dependent reflection. High frequencies radiate more (less reflection back into tube). Modeled as a digital filter in the waveguide feedback path.

---

## Commuted Piano (Piano Physical Model)

Full piano physical model:

```
1. Hammer model: nonlinear spring (harder strike → brighter)
   F = k * x^p  where p ≈ 2.3 for piano hammer felt
2. String model: stiff string waveguide × 3 strings per note (unison beating)
   - Inharmonicity coefficient B per string length
3. Soundboard: modeled as modal resonator bank
4. String-soundboard coupling: scattering junction
5. Room acoustics: convolution reverb (optional)
```

**Commuted synthesis shortcut**: pre-convolve piano body response into the initial noise burst (hammer hit). The waveguide string model then resonates this pre-colored excitation.

---

## Mutable Instruments Rings (Eurorack)

Hardware modal synthesis + waveguide module:

### Modes
1. **Modal resonator** (bank of tuned resonators, like struck metal object)
2. **Sympathetic strings** (3 simulated strings tuned to harmonics)
3. **Modulated string** (waveguide with modulated excitation)
4. **FM voice** (2-op FM with resonant body)
5. **Quantized string** (tuned to equal temperament / user-defined scale)
6. **Inharmonic string** (stretched inharmonicity coefficient)

### Key Parameters
| Parameter | Description |
|-----------|-------------|
| Frequency | Fundamental pitch of the resonator |
| Structure | Changes mode frequencies (harmonic → inharmonic transition) |
| Brightness | Timbre of excitation (dull→bright) |
| Damping | Decay time of resonator |
| Position | Excitation point (changes which modes are excited strongly) |
| Input | Trigger or audio excitation |

**Structure parameter detail**:
- Structure = 0.0: fully harmonic mode frequencies (string-like)
- Structure = 0.5: slightly inharmonic (guitar body-like)  
- Structure = 1.0: highly inharmonic (bell/gong-like)

---

## Yamaha VL1 (1994)

First commercial physical modeling synthesizer.

- 2-voice polyphony
- Wind instrument and string models
- Parameters: embouchure, breath, lip, mouthpiece tightness, reed type
- Breath controller input (optional) for real-time wind dynamics
- Price: $10,000+ (very expensive in 1994)
- Used for: acoustic instrument emulation, unusual hybrid timbres
- Followed by: VL70-m (1996, MIDI module version, 16-voice)

---

## Comparative Overview

| Instrument Type | Algorithm | Key Components |
|----------------|-----------|----------------|
| Plucked string (guitar/harp) | KS + lowpass | Noise burst → looping delay with LP filter |
| Piano | Stiff KS + soundboard | Allpass KS + modal body resonator |
| Bowed string | Waveguide + friction | Bidirectional DL + nonlinear bow-string model |
| Clarinet | Reed + cylindrical bore | Nonlinear reed + odd-harmonic waveguide |
| Flute | Jet + open bore | Jet oscillation + open-ended waveguide |
| Trumpet | Lip valve + conical bore | Lip model + conical waveguide + bell |
| Bell/gong | Modal bank | Bank of inharmonic damped biquad resonators |
| Drum head | 2D membrane | 2D wave equation (FDTD) or modal bank |
| Voice | Glottal source + LPC | Buzz/noise → all-pole lattice filter |
