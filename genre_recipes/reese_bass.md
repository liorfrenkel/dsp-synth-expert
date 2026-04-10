---
title: "Reese Bass: Detuned Oscillator Technique"
category: "Genre Recipes"
tags: [Reese, bass, drum-and-bass, jungle, detuned, chorus, dark, DnB]
---

# Reese Bass: Detuned Oscillator Technique

## Origin and History

The Reese bass takes its name from **Kevin Saunderson**'s recording alias "Reese," specifically the track **"Just Want Another Chance"** (1988) released on KMS Records (Detroit). The bassline on that track used two detuned oscillators creating a rich, chorusing bass tone that became foundational to jungle and drum & bass a few years later.

Kevin Saunderson was one of the "Belleville Three" (with Juan Atkins and Derrick May), pioneers of Detroit techno. The Reese bass technique spread into:

- **Hardcore Rave / Jungle** (UK, 1991–1993) – dark, rolling bass
- **Drum and Bass** (1994–present) – signature "amen-and-Reese" combination
- **Darkcore / Darkstep** – heaviest, most distorted versions
- **Liquid DnB** – smoother, cleaner Reese with subtle movement
- **Neuro DnB** – heavily processed, distorted, formant-modulated versions

Key records:
- "Just Want Another Chance" – Reese (1988) – the original
- "Terminator" – Goldie (1992) – jungle/rave
- "Valley of the Shadows" – Origin Unknown (1993) – dark jungle Reese
- "Timeless" – Goldie (1995) – classic DnB Reese
- "Inner City Life" – Goldie feat. Diane Charlemagne (1994)

---

## Core Technique

The Reese bass is fundamentally an **oscillator beating** technique. When two oscillators are tuned close together (but not identical), their waveforms interact through **constructive and destructive interference**, creating a cyclical amplitude modulation (AM) — the beating effect.

```
Beat frequency = |f1 - f2|

At A2 (110 Hz), 10 cents detune:
  f1 = 110 Hz
  f2 = 110 × 2^(10/1200) = 110 × 1.005792 ≈ 110.637 Hz
  Beat frequency = 110.637 - 110 = 0.637 Hz
  (one full beating cycle every ≈ 1.57 seconds)

At A3 (220 Hz), same 10 cents detune:
  f2 = 220 × 1.005792 ≈ 221.274 Hz
  Beat frequency ≈ 1.27 Hz
  (doubles with each octave)
```

This beating creates the characteristic "movement" without any LFO or modulation — just physics.

---

## Exact Recipe

### Oscillator Configuration

```
OSC 1:
  Waveform  : Sawtooth
  Octave    : 8' (or 16' for deeper register)
  Fine Tune : 0 cents (reference)
  Level     : 100%

OSC 2:
  Waveform  : Sawtooth
  Octave    : 8' (same as OSC 1)
  Fine Tune : -12 to -18 cents (asymmetric negative detune)
              Note: negative detune preferred over positive — slightly darker character
  Level     : 90–100%

OSC 3 (optional — low-end weight):
  Waveform  : Square or Sine
  Octave    : 16' (sub octave, one octave below)
  Fine Tune : 0 cents
  Level     : 30–50% (supporting, not dominant)
```

**Why asymmetric detune?** Most producers set OSC 2 negative only rather than +7/-7. The slight frequency asymmetry (one slightly below concert pitch) gives a darker fundamental character.

### Detuning Amount Guide

```
Detune (cents) | Beat at A2 (110Hz) | Beat at A3 (220Hz) | Character
---------------------------------------------------------------------------
   ±3 cents    |    0.19 Hz         |    0.38 Hz         | Very subtle throb
   ±6 cents    |    0.38 Hz         |    0.76 Hz         | Slow chorus
  ±10 cents    |    0.64 Hz         |    1.27 Hz         | Classic Reese movement
  ±15 cents    |    0.95 Hz         |    1.91 Hz         | More obvious wobble
  ±20 cents    |    1.27 Hz         |    2.54 Hz         | Fast beating, chorus-like
  ±25 cents    |    1.59 Hz         |    3.18 Hz         | Vibrato territory

Classic Reese range: ±6 to ±18 cents (or asymmetric: 0 / -12 to -18)
```

### Filter Settings

```
Type        : 12dB/oct or 24dB/oct Low-Pass
Cutoff      : 200–500 Hz
Resonance   : 20–40% (subtle emphasis — not aggressive)
Key tracking: 30–50% partial tracking

Rationale: 
  - Cutoff in 200–500 Hz range retains sub-bass body while rolling off harsh highs
  - Low resonance (20–40%) adds slight emphasis without squelch
  - Too much resonance interferes with the natural beating effect
```

### Filter LFO (Subtle Movement)

```
Shape  : Sine
Rate   : 0.3–1.0 Hz
Depth  : 10–25% of cutoff range
Target : Filter cutoff

This adds slow, organic filter sweep on top of the oscillator beating.
Combined effect: beating from oscillators + slow sweep from LFO = complex, alive bass
```

### Amplitude Envelope

```
Attack  :   0 ms (instant onset — percussive attack on each note)
Decay   : 200–400 ms (moderate — glides off if note is short)
Sustain :  80–90% (high sustain for held notes)
Release : 100–300 ms (short to medium)
```

---

## Signal Flow

```
OSC 1 (Saw, 0¢)  ─┐
OSC 2 (Saw, -15¢)─┤──→ Mixer ──→ LPF (200-500Hz, low Res) ──→ VCA ──→ Output
OSC 3 (Sub, 16') ─┘        ↑                    ↑
                         [optional         Filter LFO (Sine, 0.5Hz)
                          slight overdrive
                          pre-filter]
```

---

## Processing Chain

### Compression

```
Type     : VCA (fast, transparent) or Optical (slower, musical)
Ratio    : 8:1 to 10:1 (heavy — tames the dynamic beating effect)
Attack   :  2–5 ms (fast enough to catch transient)
Release  : 80–150 ms
Threshold: Set to 8–12 dB GR on peaks
Makeup   : +6–10 dB

Reason: Heavy compression evens out the amplitude variations from beating,
        creating a more consistent, punchy bass character.
```

### Overdrive / Saturation

```
Type  : Tube saturation or tape saturation (soft clip)
Drive : +3–6 dB (subtle — adds harmonics, creates sub-bass that translates on
                 small speakers)

Note  : Pure sawtooth oscillators are already harmonically rich.
        Saturation primarily affects the sub-octave OSC3 sine,
        making it audible on non-subwoofer systems.
```

### Sidechain Compression

```
Trigger : Kick drum output
Ratio   : 4:1 to 6:1
Attack  :  1–5 ms
Release : 100–200 ms
GR      : -4 to -8 dB on kick hit

Effect  : Bass pumps away from kick — creates rhythmic breathing,
          ensures kick transient always cuts through
```

### EQ

```
High-pass : 35–45 Hz (remove infrasonic rumble)
Low shelf : +2 dB at 80 Hz (fundamental weight)
Mid cut   : -3 to -4 dB at 250–400 Hz (reduce muddiness between sub and mids)
Presence  : +1 dB at 2–3 kHz (subtle — adds edge to sat/distortion harmonics)
High cut  : -6 dB/oct above 5 kHz (remove harshness)
```

---

## DnB Context

### Tempo and Structure

```
BPM     : 168–174 (standard DnB), most commonly 170 BPM
Bar     : 8 bars standard
Pattern : Reese bass typically plays whole notes or half notes
          — contrast with rapid Amen break overhead
```

### Sub vs. Mid-Bass Layers

In DnB, the Reese bass often splits into two registered layers:

```
Layer 1 (Sub-bass): 40–80 Hz
  - Sine or light sawtooth
  - No visible beating effect — just clean sub tone
  - Heavy sidechain from kick
  - Often distorted slightly for small speaker translation

Layer 2 (Mid-bass): 80–400 Hz  
  - The actual Reese detuned sawtooth layer
  - All the beating and movement lives here
  - Filtered to not conflict with sub layer below
  
Crossover: HP at 80 Hz on mid layer, LP at 80 Hz on sub layer
```

---

## Beating Frequency vs. Detune at Different Pitches

```
Pitch  | Frequency | 10¢ detune beat | 15¢ beat | 20¢ beat
------------------------------------------------------------
C1     |  32.7 Hz  |   0.19 Hz       |  0.28 Hz |  0.38 Hz
C2     |  65.4 Hz  |   0.38 Hz       |  0.57 Hz |  0.76 Hz
A2     | 110.0 Hz  |   0.64 Hz       |  0.95 Hz |  1.27 Hz
C3     | 130.8 Hz  |   0.76 Hz       |  1.14 Hz |  1.52 Hz
A3     | 220.0 Hz  |   1.27 Hz       |  1.91 Hz |  2.54 Hz
C4     | 261.6 Hz  |   1.52 Hz       |  2.28 Hz |  3.04 Hz

Key insight: As pitch rises, beating rate increases proportionally.
A fixed cent detune creates proportionally faster beating at higher registers.
Compensate by reducing detune for high-register Reese basslines.
```

---

## Variations

### Classic DnB Reese

```
OSC 1: Saw, 0¢
OSC 2: Saw, -16¢
Filter: LPF 250Hz, Res 25%
LFO on filter: Sine, 0.5Hz, depth 20%
Compression: 10:1 heavy limiting
Processing: subtle tape saturation
```

### Liquid DnB Reese (Cleaner)

```
OSC 1: Saw, 0¢
OSC 2: Saw, -8¢  (less detune — subtler movement)
OSC 3: Sine, 16' sub
Filter: LPF 400Hz, Res 15%  (more open, brighter)
Compression: 4:1 (gentler)
Processing: Chorus instead of beating (Dimension C-style)
Reverb: subtle room (0.5s, 15% mix) — adds space for liquid character
```

### Neuro Bass (Heavy Version)

```
OSC 1: Saw, 0¢
OSC 2: Saw, -20¢  (more detune)
Pre-filter: Heavy waveshaping (hard clip or fold) — generates dense harmonic content
Filter: BPF (bandpass) 200–800Hz, Res 40% + LFO sweep 1–3Hz, depth 60%
        This creates vowel-like "talking" quality
Post-filter: Additional saturation
Formant: Two bandpass filters moving in opposite directions (formant pair)
LFO sync: LFO to 1/4 note or 1/8 note (tempo-locked movement)
```

### Minimal Techno Reese

```
OSC 1: Saw, 0¢
OSC 2: Saw, -6¢  (very subtle beating, almost subliminal)
Filter: LPF 150–200Hz (dark, closed)
No LFO
Compression: 4:1
Processing: Minimal — let the natural beating provide all movement
```

---

## Software Implementation

### Ableton Live (Operator)

```
Oscillator A: Sawtooth, volume 100%
Oscillator B: Sawtooth, fine tune -15 cents, volume 90%
Oscillator C: Sine, octave -1 (sub), volume 35%
Filter: Ladder LPF, freq 250Hz, res 25%
Filter LFO: Sine, 0.5Hz, amount 15%
Amp Env: A=0, D=400ms, S=80%, R=200ms
```

### Serum (Xfer)

```
OSC A: Sawtooth wavetable, Unison OFF (single voice)
OSC B: Sawtooth wavetable, detune -0.15 semitones (= -15 cents)
OSC C (Sub): Sine, -24 semitones (2 octave down)
Filter 1: MG Low 24 (Moog-style), Cutoff 250Hz, Res 20%
LFO 1 → Filter Cutoff: Sine, 0.5Hz, amount 20%
ENV 1 (Amp): A=0, D=350ms, S=80%, R=200ms
```

### Native Instruments Massive

```
OSC 1: Sawtooth-Saw (wavetable position 0), Amp 8.0
OSC 2: Same wavetable, Amp 7.5, Tune -0.15 semitones
OSC 3: Sine, Amp 3.5, Octave -1
Filter 1: Lowpass 4 (24dB), Cutoff 25%, Resonance 20%
LFO 1 → Filter Cutoff: Rate 0.5Hz, Depth 15%
Amp Env: attack 0, decay 300ms, sustain 80%, release 200ms
```
