---
title: "Distortion and Saturation: Nonlinear Processing"
category: "Effects"
tags: [distortion, saturation, overdrive, fuzz, clipping, waveshaping, harmonics, tubes]
---

# Distortion and Saturation: Nonlinear Processing

Distortion and saturation add harmonic content to audio signals through nonlinear transfer functions. This ranges from subtle warmth (saturation) to extreme harmonic generation (fuzz). Understanding the math behind the transfer functions enables precise control of harmonic character.

---

## Why Nonlinear Processing Generates Harmonics

A linear system passes a sine wave at frequency `f` and outputs only frequency `f` — no new frequencies are created.

A nonlinear system passes a sine wave `x = A sin(ωt)` and outputs a polynomial expansion:

```
y = a₀ + a₁x + a₂x² + a₃x³ + a₄x⁴ + ...
```

Substituting `x = A sin(ωt)` and using trigonometric identities:
- `x²` generates DC + 2nd harmonic (2ω)
- `x³` generates fundamental (ω) + 3rd harmonic (3ω)
- `x⁴` generates DC + 2nd + 4th harmonic
- `xⁿ` generates harmonics up to `nω`

**Even-order terms** (x², x⁴, ...) → even harmonics (2nd, 4th, 6th...)
**Odd-order terms** (x³, x⁵, ...) → odd harmonics (3rd, 5th, 7th...)

---

## Transfer Functions

### Hard Clipping

```
y = clip(x, -T, +T)

        ⎧  T      if x > T
y(x) = ⎨  x      if |x| ≤ T
        ⎩ -T      if x < -T
```

Where `T` is the clipping threshold (typically 0 < T ≤ 1 for normalised signals).

- **Harmonic content**: strong odd harmonics (3rd, 5th, 7th, ...) dominate
- Sharper clipping → more high-order harmonics → more "bright" and "harsh"
- At full clipping: approaches square wave → near-infinite odd harmonic series (1, 1/3, 1/5, 1/7, ...)
- **Perceptual character**: aggressive, digital, transistor-like

### Soft Clipping: Hyperbolic Tangent (tanh)

```
y = tanh(drive × x)
```

Where `drive` is the input gain (1 = subtle, 5 = moderate, 20 = heavy).

- Smooth saturation curve — never hard-clips (asymptotic to ±1)
- Generates both odd and even harmonics, but **odd harmonics dominate**
- At low drive: nearly linear; at high drive: approaches square wave
- **Perceptual character**: warm, smooth, "tube-like"
- Used extensively in software saturation (compressors, tape models, tube emulations)

```python
import math

def tanh_saturation(x, drive):
    return math.tanh(drive * x) / math.tanh(drive)  # normalised to unit gain at DC
```

### Soft Clipping: Cubic Polynomial

```
y = x - x³/3    valid for |x| ≤ 1
```

- Pure 3rd-harmonic generator — adds only the 3rd harmonic
- Beyond |x| = 1: function is not monotonic — must clamp
- Lower computational cost than tanh (no transcendental function)
- **Harmonic content**: primarily 3rd harmonic; minor higher harmonics from waveshaping non-ideal range

```python
def cubic_soft_clip(x):
    x = max(-1.0, min(1.0, x))  # clamp to valid range
    return x - (x**3) / 3.0
```

### Asymmetric Clipping (Diode)

Clips differently on positive and negative half-cycles, simulating a diode circuit:

```
y = hard_clip(x, +0.3) on positive half
y = soft_clip(x, −1.0) on negative half   (or different threshold)
```

Or more precisely:

```python
def asymmetric_clip(x, thresh_pos=0.3, thresh_neg=0.7):
    if x > thresh_pos:
        return thresh_pos + (x - thresh_pos) * 0.1  # residual soft clip
    elif x < -thresh_neg:
        return -thresh_neg + (x + thresh_neg) * 0.1
    else:
        return x
```

- Asymmetry → **both even and odd harmonics** generated
- Even harmonics (2nd, 4th) add "body" and "warmth" without adding harshness
- Simulates silicon diode clipping (1N4148) used in Tube Screamer-style pedals
- **Perceptual character**: full-sounding, slightly aggressive, still musical

### Foldback Distortion (Wavefolder)

Above threshold `T`, the signal "folds back" (reflects):

```
y(x) = |sin(π × x / T)|    when |x| > T
```

Or a simpler triangle-wave fold:

```python
def wavefold(x, threshold):
    while abs(x) > threshold:
        if x > threshold:
            x = 2 * threshold - x
        elif x < -threshold:
            x = -2 * threshold - x
    return x
```

- Generates complex harmonic content (both even and odd)
- At high drive: near-chaotic harmonic spectrum
- Classic synthesis technique: Buchla 259 Wavefolder, Serge waveshaper
- **Perceptual character**: metallic, complex, inharmonic at extreme settings

---

## Harmonic Content by Circuit/Type

| Source              | Dominant Harmonics | Character                       |
|---------------------|--------------------|---------------------------------|
| Tube/valve amp      | 2nd > 3rd > 4th    | Warm, musical, musical fatigue  |
| Solid-state FET     | 3rd > 5th > 7th    | Bright, hard, aggressive        |
| Diode clipping      | 2nd + 3rd balanced | Aggressive but fuller than FET  |
| Tape saturation     | 2nd + 3rd subtle   | Transparent warmth, compression |
| Op-amp hard clip    | Strong odd         | Cold, digital-sounding          |
| Fuzz (transistor)   | Full harmonic series| Thick, dense, square wave-like |
| Wavefolder          | Complex spectrum   | Metallic, inharmonic            |

### Why Tube Amps Sound "Warm"
- Triode gain stage: transfer curve is naturally soft and compressive
- Second-harmonic predominance: 2nd harmonic is one octave up — blends musically
- As drive increases, compression effect limits peaks → softer dynamic feel
- Interstage transformers add slight LF resonance and HF rolloff

---

## THD (Total Harmonic Distortion)

Measurement of harmonic content relative to fundamental:

```
THD = sqrt(V₂² + V₃² + V₄² + ...) / V₁

THD% = THD × 100
```

Where `V₁` is fundamental amplitude, `V₂, V₃...` are harmonic amplitudes.

### THD Levels by Category

| Category         | THD%        | Description |
|------------------|-------------|-------------|
| Saturation       | 0.1–3%      | Subtle warmth, usually inaudible in isolation |
| Overdrive        | 5–20%       | Audible but musical breakup |
| Distortion       | 20–60%      | Heavy harmonic coloration, sustain |
| Fuzz             | 60–95%+     | Near-square wave, massive harmonic generation |
| Clipped square wave | ~48%     | Square wave THD (odd harmonics only) |

---

## Waveshaping with Chebyshev Polynomials

Chebyshev polynomials `Tₙ(x)` have the property:

```
Tₙ(cos θ) = cos(nθ)
```

This means if the input is a sine wave `x = cos(θ)`, then `Tₙ(x) = cos(nθ)` outputs exactly the nth harmonic.

### First Several Chebyshev Polynomials

```
T₁(x) = x                         → fundamental (pass-through)
T₂(x) = 2x² − 1                   → 2nd harmonic
T₃(x) = 4x³ − 3x                  → 3rd harmonic
T₄(x) = 8x⁴ − 8x² + 1            → 4th harmonic
T₅(x) = 16x⁵ − 20x³ + 5x        → 5th harmonic
T₆(x) = 32x⁶ − 48x⁴ + 18x² − 1 → 6th harmonic
```

### Precise Harmonic Injection Example

Add 10% 3rd harmonic and 5% 2nd harmonic to a signal:

```python
def add_harmonics(x, h2_amount=0.05, h3_amount=0.10):
    t2 = 2*x*x - 1      # 2nd harmonic
    t3 = 4*x*x*x - 3*x  # 3rd harmonic
    return x + h2_amount * t2 + h3_amount * t3
```

This is used in:
- Analog modelling (adding known tube harmonic signature)
- Sound design (deliberately adding warmth or brightness)
- Mastering saturation with controllable character

---

## Bit Crushing

Reduce the bit depth of the audio signal → quantisation distortion.

```python
def bitcrush(x, bits):
    levels = 2 ** bits
    return round(x * levels) / levels
```

### Bit Depth Character Reference

| Bits | Dynamic Range | Character |
|------|---------------|-----------|
| 24   | 144 dB        | Transparent (standard modern audio) |
| 16   | 96 dB         | CD quality, clean |
| 12   | 72 dB         | SP-1200, MPC60 — classic hip-hop grit |
| 8    | 48 dB         | Lo-fi crunch, early video game |
| 6    | 36 dB         | Heavy quantisation noise, aggressive |
| 4    | 24 dB         | Extreme, near-musical noise |
| 1    | 6 dB          | 1-bit delta modulation — square wave |

### What Bit Crushing Actually Does
- Quantisation noise = signal-correlated error → harmonic content
- At low levels: quantisation noise is dominant (noise floor is high)
- At high levels: clipping at ±1.0 (same as hard clipping at full scale)
- Error signal is `x - quantised(x)` → added back as distortion
- Odd harmonics dominate (due to symmetric quantisation around zero)

### Dithering
Before reducing bit depth, add dither (low-level noise) to decorrelate quantisation error from signal:

```python
import random

def bitcrush_with_dither(x, bits):
    lsb = 2 / (2**bits)  # 1 LSB = least significant bit value
    dither = (random.random() + random.random() - 1.0) * lsb  # TPDF dither
    return round((x + dither) * 2**(bits-1)) / 2**(bits-1)
```

Dithering converts harmonic distortion into low-level noise → psychoacoustically preferable at 16-bit and above.

---

## Sample Rate Reduction

Reduce the effective sample rate → aliasing products fold back into audible band.

```python
def sample_rate_reduce(signal, factor):
    """Process at original rate but hold samples for `factor` samples."""
    out = []
    held = 0.0
    for i, x in enumerate(signal):
        if i % factor == 0:
            held = x
        out.append(held)
    return out
```

### Effect
- High-frequency content aliases back into lower frequencies
- Alias frequencies are: `f_alias = |f_signal ± n × f_reduced|`
- Unlike true harmonic distortion, aliases are **inharmonic** unless signal is simple
- Character: metallic, gritty, sample-accurate retro sound
- Common use: sample rate reduction at 22050 Hz → 1990s sampler sound; 11025 Hz → 8-bit game audio

---

## Oversampling in Distortion Plugins

When a distortion effect is applied at standard sample rate (44100 Hz), nonlinear processing generates harmonics above Nyquist (22050 Hz) which fold back as aliasing artefacts.

### Solution: Oversample, Process, Decimate

```
Input (44100 Hz) → Upsample × N → Apply nonlinearity → Downsample / N → Output (44100 Hz)
```

Where N = 2, 4, 8, or 16 (oversampling factor).

### Why This Works
- At 4× oversample: Nyquist = 88200 Hz
- Harmonics up to ~80 kHz before any fold back to audible range
- Anti-aliasing LPF during downsampling removes harmonics above 22050 Hz
- Result: much cleaner harmonic content, no intermodulation artefacts

### Cost vs Quality
| Oversample | CPU Cost  | Alias Rejection |
|------------|-----------|-----------------|
| 1×         | Baseline  | None            |
| 2×         | ~2×       | Moderate        |
| 4×         | ~5×       | Good            |
| 8×         | ~10×      | Very good       |
| 16×        | ~22×      | Excellent       |

Most commercial distortion plugins use 4× or 8× oversampling.

---

## Drive / Gain Staging

Input drive level controls the operating point on the transfer curve.

```
y = f(drive × x)   where f is the chosen transfer function
```

Low drive: operates in the linear region → minimal harmonic generation
High drive: clips or saturates → rich harmonic content

### Gain Staging Best Practice
- Set input level so peak hits the desired point on the saturation curve
- Compensate output with matching gain reduction (makeup gain)
- Many hardware units are designed for −10 dBV (consumer) or +4 dBu (pro) nominal levels
- "Hot" input (+4 dBu or above) drives tubes/transistors into saturation naturally

---

## Practical Applications

### 808 Bass Saturation
Goal: add harmonics for audibility on small speakers (sub bass is inaudible below ~80 Hz on laptop speakers).

```
1. Input: 808 sine bass signal
2. Tape-style saturation: tanh(drive=2 × x) → adds 2nd and 3rd harmonics at 160 Hz, 240 Hz
3. Trim output to avoid level increase
4. Result: fundamental still sub bass, but harmonics audible on small speakers
```

Drive setting: push 3–6 dB into saturation region. Mild distortion only.

### Synth Lead Overdrive
```
1. HPF at 100 Hz (clean up low end before distortion)
2. Hard clip or tube saturation (drive=4–8)
3. LPF at 6–8 kHz after distortion (tame harsh high-order harmonics)
4. Blend with dry signal (parallel) for clarity
```

### Parallel Distortion
Mix heavily distorted copy with clean signal:

```python
blend = 0.25  # 25% distorted, 75% clean
output = (1 - blend) * clean + blend * distorted
```

Benefits:
- Preserves low-end clarity (distortion compresses low end heavily)
- Preserves transient dynamics
- Adds body and harmonic colour without destroying the original signal
- Blend 15–30% for subtle, 40–60% for obvious character

### Tape Saturation Model
Full tape emulation includes:
1. HF compression / treble rolloff (head bump at low frequencies)
2. Nonlinear saturation (even + odd harmonics, slight compression)
3. Slight wow/flutter modulation
4. IPS (tape speed) dependent frequency response

Simplified tape saturation:

```python
def tape_sat(x, drive=1.5, bias=0.0):
    # Asymmetric soft clipping simulating tape oxide magnetisation curve
    x_driven = x * drive + bias
    return (2.0 / math.pi) * math.atan(x_driven * math.pi / 2.0) / drive
```

---

## Distortion in Synthesis Contexts

### Waveshaping Synthesis
Using distortion as the primary tone-shaping method, not an effect:
- Sine wave input + waveshaper = rich timbre
- Modulating the waveshaper transfer curve with an envelope → evolving harmonic content
- Buchla Music Easel, 100 Series modulars used this as primary sound source

### FM + Waveshaping
FM generates complex sidebands; passing through waveshaper adds additional harmonic layering → extremely dense timbres.

### Filter Saturation
Many analogue-modelling filters include saturation before or inside the resonant loop:
- Moog ladder filter: subtle tanh saturation in transistor ladder → "creamy" character
- Without saturation: resonance can be abrasive; with saturation: smooth and musical
- Modelling the filter's exact saturation characteristic is key to accurate analogue sound

---

## Clipping Headroom Reference

| dBFS Level | Linear Amplitude | Headroom to 0 dBFS |
|------------|-----------------|---------------------|
| 0 dBFS     | 1.000           | 0 dB (clip point)   |
| −3 dBFS    | 0.707           | 3 dB                |
| −6 dBFS    | 0.500           | 6 dB                |
| −12 dBFS   | 0.250           | 12 dB               |
| −18 dBFS   | 0.125           | 18 dB               |
| −24 dBFS   | 0.063           | 24 dB               |

In a DAW with 32-bit float summing, internal signals can exceed 0 dBFS without distortion — saturation only occurs at the hardware output stage or when gain-staged plugins are hit hard.
