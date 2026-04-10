---
title: "Modern Techno Kick and Rumble: Synthesis and Design"
category: "Genre Recipes"
tags: [techno, kick, rumble, Berlin, modular, 909, distortion, sub]
---

# Modern Techno Kick and Rumble: Synthesis and Design

## Historical Context

Techno kick design descends directly from the Roland TR-909 Rhythm Composer (1983). While the TR-808 created the sub-bass dominant hip-hop kick, the TR-909 kick had a more aggressive, punchy character suited to faster dance music.

### TR-909 vs. TR-808 Kick

```
TR-808:
  Oscillator: Triangle wave, pitch env 200Hz → 50Hz in ~25ms
  Character : Boomy, round, deep, long decay available
  BPM use   : 85–110 BPM (hip-hop, R&B, early house)
  
TR-909:
  Oscillator: Triangle/sine, pitch env 150Hz → 55Hz in ~30–60ms
  Noise burst: white noise VCA envelope (the "snap" transient)
  Character : Punchy attack, tight body, gritty when overdriven
  BPM use   : 120–145 BPM (house, techno, trance)
```

The Berlin techno scene (1989–present) took the TR-909 and subjected it to heavy processing:
- **Richie Hawtin** (Plastikman) – stripped down, minimal, processed 909
- **Adam Beyer** – hard, punching, saturated kick
- **Slam** – industrial-weight kick design
- **Robert Hood** – minimal funky techno kick
- **Jeff Mills** – fast, precise 909 patterns

---

## TR-909 Kick Circuit Architecture

```
[Noise Burst VCA]
        ↓
        ├──────────────────────────────────────────────┐
[Pitch Envelope]                                       │
  180Hz → 55Hz (20–80ms)                             VCA (Amplitude Env)
        ↓                                               ↑
[Triangle Oscillator]──────────────────────────────────┤
        ↓                                               │
   Body Tone (55–65Hz)                          Decay: 100ms–2s
        │
        └──→ Mix → Output
```

The key is the **simultaneous pitch sweep and amplitude envelope**:
- Pitch sweeps from 180Hz (click frequency) to 55Hz (body frequency) in 20–80ms
- Amplitude envelope controls overall decay
- Noise burst (separate VCA) provides the initial "snap" transient

---

## Synthesis Recreation of TR-909 Kick

### Layer 1: Noise Burst (Attack Transient)

```
Source  : White noise generator
Envelope: 
  Attack  :  0 ms (instant)
  Decay   :  8–20 ms (very short — just a snap)
  Sustain :  0%
  Release :  5 ms

Filter  : High-pass at 1 kHz (optional — brightens the snap)
Level   : Set to complement body, not overpower (usually -6 to -12 dB relative to body)

Character: The noise burst IS the initial "click" heard at the start of every 909 kick.
           It makes the kick cut through a mix even at high BPM.
```

### Layer 2: Mid Punch (100–200 Hz)

```
OSC     : Sine or Triangle
Pitch   : Fixed at 150–200 Hz (mid-range punch)
Envelope:
  Attack  :  0 ms
  Decay   : 40–80 ms (shorter than body)
  Sustain :  0%
  Release : 20 ms

Level   : -3 to -6 dB relative to body layer
Role    : Fills the "chest punch" register — what you feel in your sternum
```

### Layer 3: Body (Main Tone)

```
OSC        : Sine (purest) or Triangle (slight odd harmonics)
Pitch Env  :
  Start freq : 150–180 Hz
  End freq   : 55–65 Hz
  Sweep time : 20–80 ms (faster = tighter kick; slower = longer sweep)
  Env shape  : Exponential fall (faster initial drop, slower approach to sustain)

Amplitude Env:
  Attack  :  0 ms
  Decay   : 400–900 ms (this is the BODY decay — long for "rolling" techno kicks)
  Sustain :  0%
  Release : 50 ms

Tuning    : F1 (43.7 Hz), G1 (49.0 Hz), A1 (55.0 Hz) are common body pitches
            55 Hz (A1) = strong fundamental in 50–60Hz subwoofer range
```

### Combining All Layers

```
Signal flow:
  [Noise Layer] ──────────────────────┐
  [Mid Punch Layer] ──────────────────┤──→ Sum/Mixer ──→ Output
  [Body Layer] ───────────────────────┘

Relative levels:
  Body       : 0 dB (reference)
  Mid Punch  : -4 dB
  Noise Burst: -8 dB

Adjust to taste — less noise for cleaner kick, more for grittier character.
```

---

## Processing Chain

### Saturation / Clipping

```
Stage   : After mixing all layers
Type    : Soft clip (tape) or hard clip (transistor)
Amount  : +6–12 dB saturation drive (aggressive for Berlin techno)
Effect  : Adds harmonics to the sine/triangle tones → kick cuts through full mix
          Hard clip at +12 dB is common for techno — produces slight grit/crunch

Soft clip:
  Drive  : +6–8 dB
  Result : Warm saturation, rounded transient
  
Hard clip:
  Drive  : +10–15 dB
  Result : Aggressive, punchy, industrial character
  
Tube/tape simulation (recommended for most applications):
  Waves CLA-2A, Soundtoys Decapitator, Softube Saturation Knob
```

### Transient Shaping

```
Attack knob  : Slightly increase (+2–4 ms) to blend noise burst into body
              (sharp transient can over-emphasize the click if not controlled)
Sustain knob : +3–5 dB (extend body level slightly)
Tool examples: SPL Transient Designer, Waves Smack Attack, Eventide Physion
```

### EQ

```
High-pass : 30 Hz (remove infrasonic content below audible range)
Low shelf : +2–4 dB at 60 Hz (reinforce kick fundamental)
Mid cut   : -4 to -6 dB at 200–400 Hz (remove boxy midrange accumulation)
Presence  : +3–5 dB at 3–5 kHz (enhance the click/snap — critical for translation)
Air       : +2 dB at 8–12 kHz (adds clarity to the noise transient if desired)
```

### Compression

```
Ratio  : 1.5:1 to 2:1 (very gentle — just for glue, not gain reduction)
Attack : 25–40 ms (SLOW — let the transient through uncompressed)
Release: 40–80 ms (fast release so compressor resets between kicks at 130–145 BPM)
GR     : 1–3 dB (minimal — mainly for limiting peaks)
Note   : Over-compressing the kick kills the transient. Techno kicks need punch.
```

### Limiter

```
Threshold: -1 to -2 dBFS (catch hard peaks)
Release  : auto or 1–5 ms
Style    : Transparent limiter (FabFilter Pro-L, UAD Precision Limiter)
```

---

## Techno Rumble

The rumble is a low-frequency textural element that sits under the kick, providing sustained low-end energy between kick hits. It transforms a dry kick pattern into a moving, physical sonic mass.

### Method 1: Distorted Sine Rumble

```
OSC     : Sine or Triangle
Freq    : 50–70 Hz
Detune  : Slow pitch modulation ±2–3 semitones
          LFO: Sine, rate 0.1–0.3 Hz, depth 15–25%
          Effect: Pitch slowly drifts, creating unsettled, organic quality

Saturation: Medium-heavy (add harmonics for audibility)
  Input saturation: +6–9 dB
  This creates a constant distorted sub texture

Volume  : -8 to -12 dB relative to kick (supporting, not competing)

Sidechain: From kick (GR -4 to -6 dB on kick hits)
           Rumble ducks on kick, swells back up in between — pumping sub effect
```

### Method 2: Filtered Noise Rumble

```
Source  : White noise generator
Filter  : Low-pass, cutoff 60–80 Hz, resonance 50–70%
          High resonance creates a pitched character at the cutoff frequency

Resonance effect:
  At 60Hz LP with 70% resonance: creates a near-sine-wave "resonant" tone at 60Hz
  Slight resonance sweep creates moving, breathing quality

LFO on filter cutoff:
  Rate  : 0.2–0.5 Hz (very slow)
  Depth : 10–20%
  Shape : Sine
  Effect: Rumble "breathes" with slow filter movement

Volume  : -10 to -14 dB (low-level texture)
```

### Method 3: Physical Modelling Resonator

```
Resonator tuned to kick fundamental (typically 55–65 Hz)
Excited by kick transient signal → resonator rings after each kick hit

Resonator settings:
  Frequency : 55 Hz (A1) — match kick body tone
  Q/Decay   : medium-long (0.5–1.5 seconds of ring)
  Input     : kick transient (pre-fader kick signal)

Result    : Each kick hit "excites" the resonator, which then rings out
            Creates sympathetic vibration effect — kick and rumble are acoustically coupled

Tools: Physical modelling plugins (Mutable Instruments Elements, Rings clones),
       convolution with resonant IR, or tuned delay lines
```

---

## Sidechain Pumping Effect

The 909 "pumping" is defining for techno and house music:

```
Classic pump (4:1):
  All mid-range elements (pads, bassline, synths) ducked by kick
  
Compression settings for sidechain elements:
  Ratio   : 4:1 to 8:1
  Attack  : 5–15 ms (don't want instant ducking — brief delay before dip)
  Release : 200–400 ms (medium — element "swells back" clearly)
  GR      : -6 to -12 dB (significant reduction)
  
The rhythmic swell between kicks is the "pump":
  Beat 1: kick hits → everything ducks down -8dB
  Beat 1+: everything swells back up over 200ms
  Beat 1.5: at full level — maximum contrast before next kick
  Beat 2: kick → duck again
```

### Multi-element Sidechain

```
Elements that should pump:
  ✓ Bass/sub elements (-8 to -12 dB)
  ✓ Pads and sustained synths (-6 to -10 dB)
  ✓ Techno rumble (-4 to -6 dB)
  
Elements that should NOT pump:
  ✗ Reverb tails (they should sustain naturally)
  ✗ Transient elements (hats, percussion)
  ✗ Atmospheric elements (too much pumping sounds dated)
```

---

## Techno Template

### Standard Structure

```
Tempo : 130–145 BPM (most common: 133–138 BPM for modern techno)
Time sig: 4/4

Kick    : every quarter note (4 per bar) — the relentless 4/4
Hi-hat  : 8th notes (open 8th notes) or 16th notes with accent on 8th
Clap/Snare: beat 2 and 4 (or every 4 bars for minimal style)
Bass    : on offbeats (and-beats: 1&, 2&, 3&, 4&) or sustained
Rumble  : continuous pad-like, sidechain-ducked
```

### Pattern Example at 135 BPM

```
Beat:  1   e   &   a   2   e   &   a   3   e   &   a   4   e   &   a
Kick:  X               X               X               X
Hat:       x       x       x       x       x       x       x       x
Open HH:                   O                           O
Bass:          X               X               X               X
              (offbeat = and-beat of each beat)
Rumble: ████████████████████████████████████████████████ (continuous)
```

### Arrangement Convention

```
8-bar loop base (most techno)
  Bars 1–8  : Kick + hi-hat (intro "tool" section)
  Bars 9–16 : + Bass
  Bars 17–24: + Rumble / sub texture
  Bars 25–32: + Additional percussion
  ...
  Drop     : Remove elements, then "drop" back in for energy

Filter automation:
  Common technique: automate low-pass filter on main elements (rising or falling)
  over 32–64 bars to create tension and release without changing arrangement
```

---

## Advanced Kick Techniques

### Kick Layering Strategy

```
Sub kick  : focused on 55–70 Hz (the "thud" on subwoofers)
Mid kick  : focused on 100–200 Hz (punch)
Top kick  : focused on 2–5 kHz (click/snap, translates on small speakers)

Use EQ to ensure each layer occupies its own frequency range:
  Sub  : HPF off / LPF 150Hz
  Mid  : HPF 80Hz / LPF 800Hz
  Top  : HPF 1kHz / no LPF

Layer all three → full-spectrum kick with sub weight, punch, and click
```

### Distortion Amount and Harmonic Series

```
Starting with a 55 Hz sine wave (A1) and adding saturation:

+3 dB soft clip:
  Adds 2nd harmonic at 110 Hz (A2): +3 dB relative to fundamental
  Adds 3rd harmonic at 165 Hz (E3): +7 dB below fundamental
  
+9 dB hard clip:
  2nd harmonic at 110 Hz: prominent
  3rd harmonic at 165 Hz: prominent
  5th harmonic at 275 Hz: present
  7th harmonic at 385 Hz: present
  → Substantially changes timbre, adds "gritty" buzzy quality

Techno sweet spot: +6–9 dB soft-hard clip — adds richness without excessive grunge
```

### Tuning Considerations

```
Kick body frequency selection matters for musical compatibility:

If track is in F minor:
  F1 = 43.7 Hz — kick body tuned to root note (maximum resonance with chord)
  C2 = 65.4 Hz — fifth above (also works)
  
Common compromise: A1 (55 Hz) or B1 (61.7 Hz) — work across many keys
In practice: many techno producers use fixed 55–65 Hz regardless of key
             (subsonics below 80Hz are poorly pitch-perceived)
```

---

## Reference Recordings

| Track | Artist | Year | Kick Style |
|-------|--------|------|------------|
| Spastik | Plastikman (Richie Hawtin) | 1993 | Minimal 909 |
| Humana | Slam | 2001 | Heavy industrial kick |
| Punisher | Adam Beyer | 2013 | Saturated Berlin kick |
| The Bells | Jeff Mills | 1997 | Fast precise 909 |
| Cosmic Car (Ostgut Mix) | Surgeon | 2001 | Dark industrial kick |
| Big Fun | Robert Hood | 1994 | Funky minimal 909 |
