---
title: "Modulation Systems and Signal Routing"
category: "Synth Architecture"
tags: [modulation, LFO, envelope, CV, MIDI, matrix, Eurorack, through-zero-FM]
sources:
  - https://en.wikipedia.org/wiki/Modular_synthesizer
  - Surge synthesizer manual (surge-synthesizer.github.io/manual)
  - MIDI 1.0 specification
  - Doepfer Eurorack standard documentation
---

## Overview

Modulation systems define how parameters change over time and in response to user input. A modulation system has **sources** (things that produce varying signals) and **destinations** (parameters that are modified). The connection between them — including amount and polarity — is the modulation **routing**.

Modern synthesizers offer modulation matrices, CV patching, MIDI mapping, and audio-rate modulation. Understanding modulation systems is the key to making a synthesizer sound "alive" and expressive rather than static.

---

## Modulation Matrix

The modulation matrix is a grid of sources vs. destinations. Any source can be connected to any destination with an arbitrary amount (positive or negative).

### Matrix Structure

```
Sources (rows):     LFO1  LFO2  Env1  Env2  Vel  AT   MW   Rand  ...
Destinations (cols): OscPitch  Filter  Amp  PW   Pan  ...
                   ┌─────────────────────────────────────────┐
LFO1               │  +50   0    0    0    0    0    0    0  │
LFO2               │   0   0    0   +80   0    0    0    0  │
Env1               │   0   0   +70   0    0    0    0    0  │
Env2               │   0  +60   0    0    0    0    0    0  │
Velocity           │   0  +40   0    0   +30   0    0    0  │
Aftertouch         │  +20  0    0    0    0    0    0    0  │
ModWheel           │  +30  0    0    0    0   +40   0    0  │
                   └─────────────────────────────────────────┘
Each cell: modulation amount (−100% to +100%)
```

### Amount Conventions
- **+100%**: full positive modulation (max effect)
- **0%**: no connection
- **−100%**: full negative modulation (inverted effect)
- **Unipolar source** (envelope: 0 to +1): amount 50% means destination moved 0–50% of its range
- **Bipolar source** (LFO: −1 to +1): amount 50% means destination moves ±50% around its center

---

## Modulation Sources

### LFO (Low-Frequency Oscillator)

**Rate**: 0.01Hz – 20Hz (some synths allow audio-rate: up to 20kHz)  
**Key sync**: LFO resets phase on each MIDI note-on  
**Tempo sync**: rate quantized to note values (1/4, 1/8, 1/16, triplet, dotted)

**Waveforms:**

| Shape | Description | Primary Use |
|-------|-------------|-------------|
| Sine | Smooth, symmetrical oscillation | Vibrato, tremolo |
| Triangle | Linear up-down | Filter sweep, vibrato |
| Square | Two-state switching | Trill, gated effects |
| Sawtooth up | Slow rise, instant drop | Rising sweeps |
| Sawtooth down (ramp) | Instant rise, slow fall | Falling sweeps |
| Sample & Hold | Random new value each cycle | Random stepping, glitchy |
| Smooth random | S&H with interpolation | Organic random motion |

**LFO envelope (fade-in/delay)**:
```
Delay:   LFO output = 0 until delay time has elapsed
Fade-in: LFO output ramps from 0 → full depth over fade-in time
→ natural vibrato onset after note attack (like a singer's natural vibrato)
```

**LFO count**: modern synths typically offer 4–8 LFOs. Classic analog synths: 1.

### Envelopes (ADSR, Multi-Stage)

**ADSR**: See subtractive_synthesis.md for math details.

**Multi-stage envelope (e.g., Surge: 6-segment)**:
```
Pre-delay → Attack → Hold → Decay → Sustain → Release
```
- Pre-delay: time before attack begins (trigger-to-start delay)
- Hold: time at peak level before decay begins (useful for punchy pads)

**Envelope loop modes**:
- One-shot: plays once and stays at sustain
- Loop (attack-sustain): loops A→D→A→D while key held
- Loop (full): continuous looping, envelope becomes an LFO-like source

**Envelope polarity**:
- Unipolar: output 0.0 to 1.0 (always non-negative)
- Bipolar: output −1.0 to +1.0 (useful for pitch with negative depth = pitch starts below and rises)

### Velocity

- MIDI velocity: 0–127 (how hard a key is pressed)
- Normalized to 0.0–1.0 for modulation
- **Default uses**: scale VCA amplitude (louder when played harder), scale filter envelope amount (brighter when played harder), scale modulator levels in FM

### Aftertouch (Channel Pressure)

- Pressure applied after key is fully pressed down
- Channel aftertouch: one value for whole keyboard (most common)
- Polyphonic aftertouch (MPE): per-key pressure (Roli Seaboard, Haken Continuum)
- **Typical use**: aftertouch → vibrato depth (performer adds vibrato by pressing harder)
- MIDI CC: not a CC, but a dedicated MIDI message (channel pressure = D0h)

### Mod Wheel (CC#1)

- Standard MIDI controller, CC number 1
- Range 0–127
- Historically connected to: LFO depth → vibrato, filter modulation
- Fully re-assignable in modulation matrix

### Pitch Bend

- Dedicated MIDI message (not a CC)
- 14-bit resolution (−8192 to +8191)
- Range typically: ±2 semitones (standard) or ±12, ±24 semitones (synth-specific)
- **Not** in modulation matrix in most hardware (hardwired to pitch)
- In software synths: can be routed to any destination

### Random / Noise Sources

| Source | Description |
|--------|-------------|
| White noise | New random value every sample: flat spectrum |
| Per-voice random | Random value assigned once per note trigger |
| S&H LFO | Random value at regular interval |
| Chaos (logistic map, etc.) | Deterministic but chaotic behavior |

### Step Sequencer as Modulation Source

- CV sequence output (in CV/gate systems) or internal sequencer
- Each step: a different CV value → modulates pitch, filter, etc.
- Creates repeating melodic or timbral patterns
- Sync to tempo: each step = 1 note value

### Audio-Rate Modulation Sources

- OSC output → mod destination (FM via filter, ring mod, waveshaping)
- Noise → pitch or filter at audio rate → complex textures
- Any audio signal → envelope follower → CV source

---

## Modulation Destinations

### Oscillator Destinations

| Destination | Unit | Notes |
|-------------|------|-------|
| Pitch (coarse) | Semitones | ±12, ±24, ±48 typical range |
| Pitch (fine) | Cents | ±100 cents typical |
| Pitch (FM) | Hz deviation | For FM effects via LFO |
| Level/Amplitude | 0–100% | Ring mod-like if audio rate |
| Pulse Width | 0–100% | PWM effect |
| Wavetable Position | 0–100% | For wavetable OSCs |
| Phase | 0–360° | Phase mod |

### Filter Destinations

| Destination | Unit | Notes |
|-------------|------|-------|
| Cutoff frequency | Hz or semitones | Main filter sweep target |
| Resonance | 0–100% | Modulated resonance; risky at high values |
| Filter type morph | 0–1 | LP→BP→HP morphing in SVF |
| Drive/Saturation | 0–100% | Pre-filter overdrive amount |

### Amplifier Destinations

| Destination | Unit | Notes |
|-------------|------|-------|
| Volume | 0–100% or dB | Overall level modulation = tremolo |
| Pan | −100% to +100% | Auto-pan with LFO |
| Stereo width | 0–200% | MS processing |

### Effects Destinations

| Destination | Example use |
|-------------|-------------|
| Reverb mix | Swell reverb on long notes |
| Delay time | Vibrato on delay signal |
| Chorus rate | Animated chorus |
| Distortion drive | Dynamic distortion |
| Pitch shift amount | Vibrato via effect |

---

## CV/Gate Standard (Eurorack)

### Physical Standard
- **Connector**: 3.5mm TS (mono) jack
- **Audio signals**: typically ±5V (10V peak-to-peak)
- **CV range**: 0–10V or −5V to +5V depending on source
- **Gate**: 0V (off) / 5V (on); some sources use 0V/10V
- **Trigger**: short pulse (1–10ms), typically 5V or 10V

### Pitch CV (1V/Octave Standard)
```
1V per octave = 1/12 V per semitone = 83.3mV per semitone
0V = some reference pitch (typically C0 or A0, synth-dependent)
+1V = one octave up
+2V = two octaves up
−1V = one octave down

Frequency from CV:
f = f_ref * 2^(CV_volts)
Example: f_ref = 8.176Hz (C0), CV = 3V → f = 8.176 * 2³ = 65.4Hz (C2)
```

**Limitations**: analog circuitry must be thermally stable; temperature drift causes pitch drift (major reason for DCOs and digital oscillators)

### Gate and Trigger

```
Gate: HIGH while key pressed, LOW when released
  → Used for: envelope ADSR gate (holds sustain while high)
  
Trigger: brief HIGH pulse regardless of key hold duration
  → Used for: single-shot events (drum triggers, envelope one-shot)
  
V-trig (Voltage trigger): uses high voltage pulse
S-trig (Shorting trigger): Moog/legacy standard, uses short-circuit (inverted logic)
  → Many older Moog modules use S-trig; adapter cables needed for modern systems
```

### Eurorack Module Sizing
- **1 HP** (horizontal pitch) = 5.08mm (1/5 inch)
- Standard rack height: 3U (133.35mm / 5.24 inches)
- Common module widths: 2HP, 4HP, 6HP, 8HP, 10HP, 12HP, 14HP, 16HP, 20HP, 28HP, 42HP
- Power headers: 10-pin or 16-pin IDC; +12V, −12V, +5V (optional), GND

### CV Utilities

| Module Type | Function |
|-------------|----------|
| Attenuverter | Scale CV by ±1 (attenuate and/or invert) |
| Offset | Add fixed DC offset to CV |
| Mixer | Sum multiple CVs |
| Mult | Split one CV to multiple destinations |
| Quantizer | Round CV to semitones / scale degrees |
| S&H | Sample-and-hold: capture CV value on trigger |
| Slew limiter | Smooth sudden CV changes (portamento effect) |
| Logic (AND/OR) | Combine gate signals |
| Comparator | Output gate when CV exceeds threshold |

---

## MIDI CC Reference

Standard MIDI Controller Change (CC) assignments most relevant to synthesis:

| CC# | Function | Range |
|-----|----------|-------|
| 1 | Modulation Wheel | 0–127 |
| 2 | Breath Controller | 0–127 |
| 4 | Foot Controller | 0–127 |
| 5 | Portamento Time | 0–127 |
| 7 | Channel Volume | 0–127 |
| 10 | Pan | 0=left, 64=center, 127=right |
| 11 | Expression | 0–127 (sub-fader for channel volume) |
| 16 | General Purpose 1 | 0–127 |
| 17 | General Purpose 2 | 0–127 |
| 64 | Sustain Pedal | 0=off, 64–127=on |
| 65 | Portamento On/Off | 0=off, 64–127=on |
| 66 | Sostenuto Pedal | 0–127 |
| 67 | Soft Pedal | 0–127 |
| 71 | Resonance (Timbre) | 0–127 |
| 72 | Release Time | 0–127 |
| 73 | Attack Time | 0–127 |
| 74 | Brightness (Filter Cutoff) | 0–127 |
| 75 | Decay Time | 0–127 |
| 76 | Vibrato Rate | 0–127 |
| 77 | Vibrato Depth | 0–127 |
| 78 | Vibrato Delay | 0–127 |
| 91 | Reverb Send Level | 0–127 |
| 93 | Chorus Send Level | 0–127 |
| 120 | All Sound Off | 0 |
| 121 | Reset All Controllers | 0 |
| 123 | All Notes Off | 0 |

**High-resolution (14-bit) CCs**: CCs 0–31 paired with CCs 32–63 (MSB/LSB) for 16384 values. Rarely used in practice.

**MIDI Polyphonic Expression (MPE)**: each note on separate MIDI channel; per-note pitch bend, pressure, slide.

---

## Through-Zero FM (TZFM)

### Problem with Standard FM
Standard VCO FM: as carrier frequency is modulated down, it approaches 0Hz and "stalls" — the phase freezes momentarily. This creates:
- Phase discontinuities
- Missing sidebands
- "Wavy" instability at deep modulation

### Through-Zero Solution
TZFM allows the oscillator to pass through 0Hz and **reverse direction**:

```
Phase increment normally: Δφ = (fc + mod_signal * depth) / sampleRate

Standard FM: Δφ = max(0, ...) → phase can't go negative
TZFM:        Δφ = (fc + mod_signal * depth) / sampleRate  → can be negative
When Δφ < 0: phase decrements → output is time-reversed
```

**Effect**: symmetric upper and lower sidebands at all modulation depths. Eliminates dead spots and produces richer, more symmetric FM spectra.

**Analog implementation**: requires oscillator core that can run at negative "frequency" (phase reversed sine). Buchla 259 and similar VCOs achieve this via core design that allows negative current flow.

**Comparison:**
```
Standard FM (fc=100Hz, fm=50Hz, β=5):
  Lower sidebands: 100-50, 100-100, 100-150 → 50, 0(dead!), -50(folded)
  
TZFM (same parameters):
  Lower sidebands: 50, 0(phase reversal), -50(TZFM treats as +50 with phase flip)
  Result: fully populated symmetric spectrum
```

---

## Ring Modulation

**Definition**: multiply two audio signals together, suppress both originals:

```
output(t) = A(t) × B(t)
```

For two sine waves:
```
A(t) = a · sin(2π·fA·t)
B(t) = b · sin(2π·fB·t)

output(t) = (ab/2) · [cos(2π·(fA−fB)·t) − cos(2π·(fA+fB)·t)]
```

**Output frequencies**: `fA + fB` and `|fA − fB|` (sum and difference only, no originals)

**Uses:**
- Metallic, bell-like tones (inharmonic fA/fB ratios)
- Robot voice (speech × sub-audio oscillator)
- Stereo imager (channel cross-multiplication)
- Classic IDM and industrial textures

**vs. AM (Amplitude Modulation)**:
```
AM: output(t) = (1 + m · B(t)) · A(t)
    = A(t) + m · A(t) · B(t)
    → carrier A + sidebands (fA ± fB)
    → carrier is PRESENT in output (unlike ring mod)
    m = modulation depth (0–1 for classic AM)
```

---

## Amplitude Modulation (AM) Synthesis

```
output(t) = (1 + m · Amod(t)) · Acarrier(t)
```

Where:
- `m` = modulation depth (0 = no mod, 1 = full AM)
- `Amod` = modulator signal (normalized −1 to +1)
- `Acarrier` = carrier signal

**Sidebands created at**: `fc ± fm`, `fc ± 2fm`, `fc ± 3fm`... (same as FM but with different amplitudes)

**At m = 0**: just the carrier  
**At m = 1**: 100% AM, carrier amplitude goes from 0 to 2× (full modulation)  
**At m > 1**: overmodulation — carrier inverts phase → harsh, distorted AM

**Key difference from FM**: in AM, the carrier frequency itself does not change; only its amplitude is modulated. FM modulates the frequency (or phase). For sinusoidal modulator, both produce sidebands at `fc ± n*fm`, but with very different amplitudes at those sidebands.

---

## Waveshaping

Apply a nonlinear transfer function `f(x)` to a waveform. The nonlinearity introduces new harmonics.

### Transfer Function Examples

```cpp
float hardClip(float x, float threshold) {
    return fmaxf(-threshold, fminf(threshold, x));
}

float softClip(float x) {
    return tanhf(x);  // smooth saturation (tanh)
}

float bitcrush(float x, int bits) {
    float scale = pow(2, bits - 1);
    return round(x * scale) / scale;
}

float foldback(float x, float threshold) {
    // Buchla-style wavefolding
    while (fabsf(x) > threshold) {
        if (x > threshold) x = 2*threshold - x;
        else               x = -2*threshold - x;
    }
    return x;
}
```

### Chebyshev Polynomial Waveshaping

For input `x = sin(θ)`, Chebyshev polynomials Tₙ(x) produce pure harmonic n:

```
T1(x) = x                      → fundamental only
T2(x) = 2x² - 1                → 2nd harmonic + DC
T3(x) = 4x³ - 3x              → 3rd harmonic + fundamental
T4(x) = 8x⁴ - 8x² + 1        → 4th harmonic + 2nd + DC
T5(x) = 16x⁵ - 20x³ + 5x    → 5th harmonic + 3rd + fundamental
```

By constructing the transfer function as a linear combination of Chebyshev polynomials, you can target specific harmonic ratios:

```cpp
// Add 3rd harmonic at 50% strength:
float output = T1(x) + 0.5f * T3(x);
```

---

## Wavefolding

Buchla-style wavefolding: when the signal exceeds a threshold, it is "folded back" on itself.

```
Threshold = 0.5 (normalized)

Signal:  0.0 → 0.5 → 1.0 → 1.5 → 2.0 → (continues rising in original)
Folded:  0.0 → 0.5 → 0.0 → -0.5 → 0.0 → (reflects back each time it hits threshold)
```

```cpp
float wavefold(float x, float threshold) {
    float t = threshold;
    while (true) {
        if      (x > t)  x = 2*t - x;   // fold down
        else if (x < -t) x = -2*t - x;  // fold up
        else             break;
    }
    return x;
}
```

**Character**: creates complex symmetric waveforms. Number of folds = floor(peak/threshold). More folds → more harmonics, brighter/more complex timbre.

**Buchla Complex Oscillator (Buchla 259)**: classic wavefolding instrument. Also includes TZFM.

---

## Crossfading Oscillators (Equal-Power)

When morphing between two oscillator waveforms or layers, naive linear crossfade causes a loudness dip at 50%:

**Linear crossfade** (incorrect for equal loudness):
```
output = alpha * A + (1 - alpha) * B
at alpha=0.5: amplitude = 0.5*A + 0.5*B
if A and B are uncorrelated (different phases): power = 0.5² + 0.5² = 0.5 → −3dB dip
```

**Equal-power crossfade** (constant loudness):
```cpp
float alpha = position;  // 0.0 to 1.0
float gainA = cosf(alpha * M_PI_2);   // cos(0→90°) = 1→0
float gainB = sinf(alpha * M_PI_2);   // sin(0→90°) = 0→1

output = gainA * A + gainB * B;
// Power: gainA² + gainB² = cos²(θ) + sin²(θ) = 1.0 (constant!)
```

**Applications**: wavetable position scanning, layer crossfade, macro parameter morph

---

## Surge Synthesizer Modulation System

Surge is a free/open-source software synth with an advanced modulation matrix.

### Surge Modulation Sources
- **LFOs** (6 per patch): rate, shape, trigger mode, deform parameter
- **Scene-level vs. voice-level**: LFO can be per-note or per-patch
- **Envelopes**: 6-stage (pre-delay, attack, hold, decay, sustain, release)
- **Special**: stepsequencer (LFO shape), MSEGS (multi-segment envelope), formula (custom scripts)

### Surge Modulation Routing
- Right-click any parameter → "Add Modulation" → choose source
- Amount: drag after adding, or set numerically
- Multiple sources per destination: additive
- Bipolar amounts: can invert any source

### Surge Extended Modulation Sources
- Velocity, Aftertouch, Polyphonic Aftertouch (MPE)
- MIDI CC: any CC number
- Pitchbend range: separate up and down amounts
- Macro knobs: user-assignable macros (8 per patch) → assign to any parameter → one knob controls many parameters
- Random: per-voice (different random each note) or per-patch (stable per patch)

---

## Routing Topologies Summary

### Series (Chain)
```
Source → Amount × → Add to destination
Multiple sources in series: destination += Σ(sourceᵢ × amountᵢ)
```

### Parallel
```
Source 1 ──→ Destination A
Source 1 ──→ Destination B (same source, two destinations)
```

### Nested / Modulation of Modulation
```
LFO1 depth ← controlled by Envelope 2
(meta-modulation: amount of one modulator controlled by another)
Available in: Surge, Patcher, VCV Rack, etc.
```

### Audio-Rate Modulation
When a modulation source runs at audio rate (above ~20Hz):
- **Pitch mod at audio rate** = FM synthesis (see fm_synthesis.md)
- **Amplitude mod at audio rate** = AM/Ring modulation
- **Filter cutoff mod at audio rate** = filter FM, creates sidebands and non-linear spectrum
