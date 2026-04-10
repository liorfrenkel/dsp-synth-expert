---
title: "Chorus, Flanger, and Phaser: Modulated Delay Effects"
category: "Effects"
tags: [chorus, flanger, phaser, delay, LFO, modulation, comb-filter, allpass]
---

# Chorus, Flanger, and Phaser: Modulated Delay Effects

Chorus, flanger, and phaser all use modulated delay or phase shifts to create movement, thickness, and spatial effects. They differ in delay length, topology, and the perceptual character they produce.

---

## Common Architecture

All three effects share this core structure:

```
Input ──┬───────────────────────────────────────────────────────→ Dry
        │
        └→ Variable Delay or Allpass chain → Mix → Wet output
                   ↑
               LFO modulation
```

The LFO (Low Frequency Oscillator) sweeps the delay time or filter frequency periodically.

---

## Flanger

### Principle
A very short delay (0.1–15 ms) is mixed with the dry signal. The mixing creates **comb filtering**: constructive and destructive interference at regularly-spaced frequencies.

When delay time is modulated by an LFO, the comb filter notches sweep up and down the frequency spectrum, producing the characteristic "jet plane" whoosh.

### Comb Filter Notch Frequencies

```
f_notch(n) = (2n + 1) × (1 / (2 × delay_time_sec))

for n = 0, 1, 2, 3, ...
```

Example: delay = 5 ms = 0.005 s
```
f_notch(0) = 1 / (2 × 0.005) = 100 Hz
f_notch(1) = 3 × 100 = 300 Hz
f_notch(2) = 5 × 100 = 500 Hz
...
```

Notches are **evenly spaced** in linear frequency — at every odd multiple of `1/(2×delay)`.

When delay decreases to 1 ms:
```
f_notch(0) = 500 Hz, f_notch(1) = 1500 Hz, f_notch(2) = 2500 Hz, ...
```

The notches sweep upward as delay decreases, downward as delay increases.

### Signal Flow with Feedback

```
Input ──┬──→ Delay Line (M samples, LFO-modulated) ──→ × g_wet ──→ Sum ──→ Output
        │                                                ↑
        │                                          Feedback ──← × g_fb ←──┘
        └──→ × g_dry ──────────────────────────────────────────────────────────┘
```

### Parameters

| Parameter  | Typical Range | Effect |
|------------|---------------|--------|
| Rate       | 0.1–5 Hz      | Speed of the LFO sweep |
| Depth      | 0–15 ms       | Maximum delay modulation excursion |
| Manual / Center | 0–15 ms | Center point of the delay sweep |
| Feedback   | 0–95%         | Resonance: adds peaks between notches |
| Wet/Dry    | 0–100%        | Blend of modulated and dry signals |
| Polarity   | +/−           | Invert feedback for different character |

### Feedback Effect
- Feedback `g_fb > 0` adds resonant peaks between the notches
- Higher feedback → more pronounced metallic resonance
- Negative feedback shifts peak positions → different harmonic flavour
- Above ~95% feedback → instability / self-oscillation

### Classic Hardware Flangers
- **AMS DMX 15-80S**: studio standard; manual delay control, very musical
- **Eventide H949 Harmonizer**: features detuning for chorus-like flanging
- **MXR Flanger M117R**: analogue BBD-based, Rate/Width/Regen/Manual controls
- **Tape flanging** (origin of the name): simultaneous playback of same tape on two machines; operator manually slows one reel (touching the "flange") — causes delay that varies naturally

### Flanging at Extreme Settings
- Very short delay (<1 ms) + feedback: harsh, metallic resonance
- Low rate (0.1 Hz): slow, spacious sweep
- High rate (5+ Hz): tremolo-like shimmer
- Zero rate, manual sweep: manual phasing effect for transitions

### Flanging DSP Implementation

```python
import math

class Flanger:
    def __init__(self, sample_rate, max_delay_ms=15.0):
        self.sr = sample_rate
        self.max_delay = int(max_delay_ms * sample_rate / 1000)
        self.buffer = [0.0] * (self.max_delay + 2)
        self.write_idx = 0
        self.phase = 0.0
        self.feedback_sample = 0.0

    def process(self, x, rate_hz, depth_ms, center_ms, feedback, wet):
        # LFO
        lfo = math.sin(2 * math.pi * self.phase)
        self.phase = (self.phase + rate_hz / self.sr) % 1.0

        # Delay time in samples
        center_samp = center_ms * self.sr / 1000
        depth_samp = depth_ms * self.sr / 1000
        delay_samp = center_samp + depth_samp * lfo

        # Write to buffer with feedback
        self.buffer[self.write_idx] = x + feedback * self.feedback_sample

        # Linear interpolation read
        read_pos = (self.write_idx - delay_samp) % len(self.buffer)
        i0 = int(read_pos)
        frac = read_pos - i0
        i1 = (i0 + 1) % len(self.buffer)
        delayed = self.buffer[i0] * (1 - frac) + self.buffer[i1] * frac

        self.feedback_sample = delayed
        self.write_idx = (self.write_idx + 1) % len(self.buffer)

        return x * (1 - wet) + delayed * wet
```

---

## Chorus

### Principle
Multiple copies of the signal (2–4 voices), each slightly delayed and pitch-shifted by a slowly modulating LFO, are mixed together. The slight detuning and timing differences simulate the natural imprecision of multiple performers playing in unison.

Unlike flanging, chorus uses longer delays (10–30 ms) which creates a more spacious, "thickened" sound with less obvious comb filtering.

### Chorus vs Flanger Comparison

| Feature          | Flanger          | Chorus               |
|------------------|------------------|----------------------|
| Delay range      | 0.1–15 ms        | 10–30 ms             |
| LFO rate         | 0.1–5 Hz         | 0.1–2 Hz             |
| Voices           | 1 delayed copy   | 2–8 voices           |
| Comb filtering   | Strong, audible  | Weak, subtle         |
| Character        | Sweeping jet/whoosh | Thick, wide, lush |
| Feedback         | Often used       | Rarely (colours too much) |

### Chorus Parameters

| Parameter  | Typical Range    | Effect |
|------------|------------------|--------|
| Rate       | 0.1–2 Hz         | LFO speed; slow for lush, fast for vibrato-like |
| Depth      | 5–30 ms          | Peak LFO deviation; more = more pitch variation |
| Voices     | 2–8              | Number of delayed copies; more = denser |
| Spread     | 0–180°           | Phase offset between voices; 180° = L/R symmetry |
| Wet/dry    | 20–80%           | More wet = wider, less dry clarity |

### LFO Detuning Between Voices
Key to natural-sounding chorus: each voice gets a slightly different LFO rate and phase.

```python
# 4-voice chorus LFO assignment
rates = [0.40, 0.43, 0.37, 0.41]  # Hz — slightly different per voice
phases = [0.0, 0.25, 0.5, 0.75]   # quarter-cycle offsets
pans   = [-1.0, -0.5, 0.5, 1.0]  # stereo spread
```

If all voices use the same LFO rate, they will periodically align perfectly → audible "beat" in the modulation. Random initial phase offsets prevent this.

### Roland Juno-106 Chorus (Classic)

- BBD chips: MN3101 (clock) + MN3102 (delay line), 512 stages
- **Mode I**: single BBD, rate ≈ 0.5 Hz, depth ≈ 15 ms
- **Mode II**: two BBDs slightly different rates for stereo shimmer
- Both modes simultaneously: very thick, slightly unstable chorus
- Character: smooth, lush, slightly low-fidelity due to BBD noise
- No feedback path in original circuit
- This specific sound drove 1980s pop/synthwave aesthetics

### Roland Juno Chorus Parameters (Approximate)
- Rate: 0.513 Hz (Mode I), 0.863 Hz (Mode II)
- Depth: ~15 ms (BBD delay at nominal clock)
- Stages: 512 samples per BBD chip
- Anti-aliasing: input and output filters at ~8 kHz

### Subtle Chorus Recipe (2-voice)
```
Voice 1: rate=0.4 Hz, depth=8 ms, phase=0°, pan=L (−1.0)
Voice 2: rate=0.43 Hz, depth=8 ms, phase=180°, pan=R (+1.0)
Wet: 40–50%
```

### Thick Chorus Recipe (4-voice)
```
Voice 1: rate=0.30 Hz, depth=15 ms, pan=hard L
Voice 2: rate=0.37 Hz, depth=12 ms, pan=mid L
Voice 3: rate=0.43 Hz, depth=14 ms, pan=mid R
Voice 4: rate=0.50 Hz, depth=13 ms, pan=hard R
Wet: 60–70%
```

### BBD (Bucket Brigade Device) Technology
- Analogue shift register integrated circuit
- Samples propagate from "bucket to bucket" at a clock rate
- Delay time = `N_stages / (2 × clock_frequency)` (factor 2 for charge coupling)
- Famous chips: MN3007 (1024 stages), MN3205 (4096 stages), SAD1024
- Imperfections: clock noise, bandwidth limiting, slight signal degradation — contribute character
- Modern plugins (e.g., TAL-Chorus-LX, Arturia Juno V) model these imperfections

---

## Phaser

### Principle
An allpass filter network sweeps the phase of frequency components without changing their amplitude. When the phase-shifted signal is mixed with the dry signal, phase cancellation creates notches at frequencies where phase difference = 180°.

Unlike the evenly-spaced notches of a flanger, phaser notches are **not evenly spaced** in linear frequency — they follow the allpass filter's phase characteristics, giving a more musical, "spiralling" quality.

### Allpass Filter Stage

Each allpass stage contributes a phase sweep from 0° to −180° (or 0° to +180°) as frequency goes from 0 to Nyquist.

A first-order digital allpass filter:

```
y[n] = a * (x[n] - y[n-1]) + x[n-1]

where a = (tan(π × f_c / fs) - 1) / (tan(π × f_c / fs) + 1)
```

The notch condition: when the cumulative phase shift = ±180°, the allpass output and dry signal cancel.

### Stages and Notch Count

| Stages | Notches |
|--------|---------|
| 2      | 1       |
| 4      | 2       |
| 6      | 3       |
| 8      | 4       |
| 12     | 6       |
| 24     | 12      |

### Phaser Signal Flow

```
Input ──┬──→ AP Stage 1 → AP Stage 2 → ... → AP Stage N ──→ × g_wet ──→ Sum ──→ Output
        │                                                                    ↑
        └──────────────────────── × g_dry ──────────────────────────────────┘
                                                Feedback: out → AP input
```

The LFO sweeps the cutoff frequency of the allpass stages simultaneously, causing all notches to sweep together.

### Phaser Parameters

| Parameter      | Typical Range | Effect |
|----------------|---------------|--------|
| Rate           | 0.05–5 Hz     | LFO speed |
| Depth          | 0–100%        | Notch sweep range |
| Stages         | 2/4/6/8/12    | Number of notches in spectrum |
| Feedback       | 0–90%         | Resonance around notches |
| Center frequency | 100 Hz–5 kHz | Where notches are centered |
| Wet/dry        | 0–100%        | 50% is typical (gives maximum cancellation depth) |

### Phaser at 50% Wet
- At 50/50 mix, notches can go to −∞ dB (full cancellation)
- At other ratios, notches are shallower
- Most classic phasers run at exactly 50% wet

### Classic Phasers

**Small Stone (Electro-Harmonix)**
- 4-stage allpass
- Single Rate knob + Color switch (adds feedback)
- Classic rock sound: Smashing Pumpkins, Nirvana, Pink Floyd
- Color switch: adds feedback → resonant peaks between notches

**MXR Phase 90**
- 4-stage allpass
- Single Rate knob — famously simple
- Van Halen "Eruption" phaser sound
- Script vs block logo versions: different op-amp biasing → subtle tonal difference

**MXR Phase 45**
- 2-stage allpass (single notch)
- Very subtle, gentle phase sweep
- Less dramatic than Phase 90

**Univibe (Shin-ei)**
- Simulates the rotary speaker (Leslie cabinet) effect
- Uses 4 LDR (light-dependent resistor) / lamp stages — not true allpass
- Photocell stages have asymmetric response → unique wobble character
- Jimi Hendrix "Machine Gun", Robin Trower

**Eventide H3000 phaser mode**
- More stages, advanced modulation routing

### Phaser vs Flanger: Key Distinction
- Flanger notches: evenly spaced in **linear** frequency (harmonically ambiguous)
- Phaser notches: not evenly spaced (allpass frequency warping) → sounds more musical, less "electronic"
- Phaser at slow rate + high feedback: string/vocal ensemble shimmer
- Flanger at fast rate + high feedback: classic electronic/industrial sound

---

## Vibrato

Pure pitch modulation with no dry signal (100% wet modulated delay, no feedback).

```
y[n] = input[n - delay(n)]    where delay varies with LFO
```

No dry signal is mixed in, so there is no comb filtering — only pitch variation.

### Parameters
- Rate: 5–7 Hz (natural voice/instrument vibrato rate)
- Depth: ±25–50 cents (quarter-tone to half-semitone)
- Waveform: sine (smooth) or triangle (linear pitch ramp)

### Vibrato vs Chorus
- Chorus: vibrato + dry signal mixed = apparent pitch and timing variation without full detuning
- Vibrato: no dry signal = pure pitch modulation, more pronounced effect

---

## LFO Waveforms and Their Character

| Waveform  | Character |
|-----------|-----------|
| Sine      | Smooth, musical sweep — most common |
| Triangle  | Linear sweep, slightly harsher edges |
| Square    | Sudden jump between two delay times — choppy, gated |
| Sawtooth  | Rising or falling ramp — one-directional sweep |
| S&H (random) | Stepped random jumps — unpredictable, glitchy |

---

## Modulation Routing in Modern Synthesizers

All three effects are commonly implemented as built-in or insert effects in synthesizers:

- **Chorus insert on oscillator output**: applied before filter → filter processes chorus artefacts
- **Chorus in effect chain after filter/amp**: applied to final voice output — more standard
- **Phaser in filter self-oscillation context**: can interact with filter resonance for unusual tones
- **Flanger on percussion**: gated flanger (envelope-triggered depth) for industrial sound

### Interaction with Other Effects
- Chorus before reverb: each chorus voice gets its own reverb tail → very spacious
- Flanger before distortion: comb filter notches emphasise distortion harmonics in interesting ways
- Phaser on bass: gentle phaser adds movement without pitch variation — classic funk bass technique

---

## Summary: Choosing Between Chorus, Flanger, and Phaser

| Effect  | Best For                                      | Avoid When                          |
|---------|-----------------------------------------------|-------------------------------------|
| Chorus  | Thickening pads, synths, guitars, bass        | Already dense mixes (adds mud)      |
| Flanger | Dramatic sweeps, percussion, transitions       | Delicate harmonic material           |
| Phaser  | Rhythmic movement, electric piano, funk       | Dense polyphonic material (masking) |
| Vibrato | Solo instruments, leads, subtle animation     | Mix elements needing pitch stability |
