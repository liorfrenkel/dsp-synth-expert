---
title: "80s DX7 FM Electric Piano and Signature Sounds"
category: "Genre Recipes"
tags: [DX7, FM, 80s, electric-piano, brass, Yamaha, pop]
---

# 80s DX7 FM Electric Piano and Signature Sounds

## Historical Context

The Yamaha DX7 (1983) changed popular music overnight. Priced at $1,995, it was the first mass-market digital synthesizer and appeared on virtually every major pop and R&B recording from 1983 to 1991. Its FM (Frequency Modulation) synthesis engine used 6 sine-wave operators arranged in 32 possible algorithms to generate complex timbres with no analog circuitry.

Key records featuring DX7:
- **"Jump"** – Van Halen (1984) – DX7 piano introduction
- **"Against All Odds"** – Phil Collins (1984) – DX7 electric piano throughout
- **"Alone"** – Heart (1987) – DX7 synth textures
- **"Take On Me"** – A-ha (1985) – DX7 arpeggiated synth lines
- **"What's Love Got To Do With It"** – Tina Turner (1984) – DX7 chord stabs

---

## FM Synthesis Fundamentals

### Operators and Algorithms

Each DX7 operator is a sine oscillator with:
- **Frequency ratio** (C:M) – relative to the played note frequency
- **Output level** (0–99) – amplitude of the operator
- **Envelope** – 4 Rate/Level pairs (R1/L1, R2/L2, R3/L3, R4/L4) plus keyboard level scaling
- **Detune** – ±7 semitone fine-tuning in steps

**Operator roles:**
- **Carrier** — directly outputs audio (the sound you hear)
- **Modulator** — modulates the frequency of the operator below it in the algorithm stack; changes timbre not pitch (if level is low) or pitch (if level is very high)

```
Modulation Index β = Modulator Peak Deviation / Carrier Frequency
                   = (Modulator Level × Mod Ratio) / Carrier Ratio

β < 1  → subtle harmonic enrichment (warm, flute-like)
β 1–3  → moderately complex spectra (piano, electric piano)
β > 5  → heavily distorted, noisy, inharmonic spectra (bell, percussion)
```

### Reading DX7 Algorithms

32 algorithms define how 6 operators connect. Notation:
```
Op 2 → Op 1         Op 2 modulates Op 1 (Op 1 is carrier)
Op 4 → Op 3 → Op 1  Op 4 modulates Op 3; Op 3 modulates Op 1 (3-deep stack)
```
Brackets indicate feedback: `[Op 1]` = Op 1 has feedback (self-modulates).

---

## Patch 1: Rhodes-Style Electric Piano ("E.PIANO 1")

This is the most famous DX7 preset. It emulates the Fender Rhodes electric piano by combining a sustained fundamental tone (tine resonance) with a bright metallic attack transient.

### Algorithm

**Algorithm 5:**
```
Stack A:         Stack B:
Op 2 → Op 1     Op 5 → Op 4 → Op 3
   ↓                        ↓
Output A                Output B
```
Two independent carrier stacks. Stack A = the "tine" fundamental. Stack B = additional harmonic character.

### Operator Settings

```yaml
Op 1 (Carrier — fundamental tine resonance):
  Ratio   : 1.00
  Level   : 99
  Env R1  : 94  L1: 99   # very fast attack to full level
  Env R2  : 25  L2: 50   # slow decay to 50% (sustain decay)
  Env R3  : 20  L3: 0    # continues to decay toward 0 during hold
  Env R4  : 60  L4: 0    # medium-fast release
  Vel Sens: 4 (moderate velocity response)
  Key Scale: -2 / +3 (darkens lower notes, brightens upper)

Op 2 (Modulator — attack click and bell character):
  Ratio   : 14.0          # 14x carrier = creates inharmonic high-frequency transient
  Level   : 58            # moderate modulation index on attack
  Env R1  : 99  L1: 99   # instant attack
  Env R2  : 95  L2: 0    # extremely fast decay — click only at start
  Env R3  : 0   L3: 0
  Env R4  : 0   L4: 0
  Vel Sens: 7 (maximum — attack click scales with velocity → harder hit = brighter click)

Op 3 (Carrier — harmonic body):
  Ratio   : 1.00
  Level   : 99
  Env     : similar to Op 1, slightly different decay curve

Op 4 (Modulator — "tine" characteristic overtones):
  Ratio   : 1.00
  Level   : 78            # high modulation → complex harmonics when active
  Env R1  : 94  L1: 99
  Env R2  : 30  L2: 30
  Env R3  : 18  L3: 0
  Env R4  : 56  L4: 0

Op 5 (Second-stage modulator):
  Ratio   : 14.0
  Level   : 50
  Vel Sens: 7
  Env: fast attack, instant decay (same role as Op 2)
```

### Key Behavior

- At **low velocity** (pp): Op 2 and Op 5 contribute minimally — sound is warm, round, sustaining (soft Rhodes)
- At **high velocity** (ff): Op 2 and Op 5 open fully — strong attack transient with bright metallic click (hard Rhodes strike)
- **Pitch EG** on DX7: all 4 operators receive subtle global pitch envelope. Set a tiny negative pitch deviation on decay (-2 to -3) to mimic Rhodes tine pitch drop on hard strikes

### Master Settings

```
Algorithm  : 5
Feedback   : 0 (no feedback on this patch)
LFO        : off (no vibrato in standard preset)
Transpose  : 0
Pitch EG   : R1=50 L1=50, R2=50 L2=50, R3=50 L3=50, R4=50 L4=50 (neutral/bypassed)
```

---

## Patch 2: DX7 Brass ("BRASS 1")

The DX7 brass is another defining 80s sound — heard on countless pop records. It uses stacked operators with all ratios at 1:1 and heavy feedback to generate a buzzy, dense harmonic spectrum similar to brass instruments.

### Algorithm

**Algorithm 28:**
```
[Op 1] → Op 2 → Op 3 → Op 4 → Op 5 → Op 6
```
All 6 operators in series. Op 1 has feedback (self-modulates → controls "brightness" of buzz).

### Operator Settings

```yaml
All Operators:
  Ratio   : 1.00 (all at fundamental frequency)
  Level   : 90–99 (high levels throughout chain)
  
Op 1 (at end of chain — carrier):
  Feedback: 5 (creates harmonic-rich, buzzy timbre)
  Level   : 99
  Env R1  : 99  L1: 99  # hard, fast attack
  Env R2  : 58  L2: 66  # slight decay
  Env R3  : 40  L3: 60  # sustain area
  Env R4  : 55  L4: 0   # medium release

Op 2 (first modulator in chain):
  Level   : 95
  Env     : closely mirrors Op 1 timing
  Vel Sens: 3

Op 3–6:
  Levels  : 90–95
  Detune  : each op offset by ±1 to ±3 steps → internal chorus effect without LFO
             (Op 3: +2, Op 4: -1, Op 5: +3, Op 6: -2)
  Vel Sens: 2–3
```

### Detune for Ensemble Effect

The subtle detuning of each operator creates a gentle chorus without any external processing:

```
If all operators at R=1.00:
  Op 3 detune +2 → frequency = note_freq × (1 + 2×0.01547%) ≈ note_freq + tiny offset
```
The sum of 6 slightly detuned sines beating against each other creates ensemble richness.

### Velocity Mapping

```
Hard velocity (pp → ff):
  All operators scale level by Vel Sens setting
  At ff: fuller spectrum → brighter, more "blaring" brass
  At pp: reduced modulation → softer, rounder tone
```

---

## Patch 3: DX7 Bass

### Algorithm

**Algorithm 1:**
```
Op 2 → Op 1
Op 4 → Op 3  (all feed into Op 1 as carriers)
Op 6 → Op 5
```
Multiple modulator/carrier pairs feeding one output.

### Settings

```yaml
Op 6 (primary carrier):
  Ratio   : 1.0
  Level   : 99
  Env R1  : 99  L1: 99  # instant attack
  Env R2  : 60  L2: 50  # mid decay
  Env R3  : 30  L3: 40  # sustain
  Env R4  : 70  L4: 0   # medium-fast release

Op 5 (modulator — harmonics):
  Ratio   : 1.0
  Level   : 70           # β ≈ 2.8 → moderate harmonic content
  Env R1  : 99  L1: 99
  Env R2  : 80  L2: 20   # fast decay on modulator → attack transient only
  Env R3  : 0   L3: 0
  Env R4  : 0   L4: 0

Pitch EG:
  L1: 60  (above center = starts sharp)
  R1: 75  (moderate speed back to center)
  L2: 50  (center pitch = sustain)
  → creates subtle pitch drop on attack mimicking plucked upright bass
```

The pitch envelope slight overshoot and fall imitates the upright bass attack where string tension produces a brief pitch rise before settling.

---

## Patch 4: Marimba / Vibraphone

### Inharmonic Partial Synthesis

Marimba/vibraphone tones have a characteristic 3rd partial at a non-integer ratio. The key is C:M = 1:3.5.

```yaml
Algorithm: 3 (Op 2 → Op 1, simple pair)

Op 1 (carrier — fundamental body):
  Ratio   : 1.0
  Level   : 99
  Env R1  : 99  L1: 99  # instant
  Env R2  : 65  L2: 0   # moderate decay (~400ms on marimba)
  Env R3  : 0   L3: 0
  Env R4  : 65  L4: 0

Op 2 (modulator — inharmonic attack transient):
  Ratio   : 3.5          # creates characteristic "wooden" inharmonic partial
  Level   : 72           # β = L2 / (99/2) × ratio ≈ moderate
  Env R1  : 99  L1: 99  # instant
  Env R2  : 90  L2: 0   # VERY fast decay — click only (< 50ms)
  Env R3  : 0   L3: 0
  Env R4  : 0   L4: 0

Vibraphone variant:
  Add a second carrier pair with Op ratio 2.756 (slightly inharmonic 2nd partial)
  Op 4 (carrier): Ratio 2.756, Level 60, slower decay than Op 1
  Op 3 (modulator): Ratio 2.756, Level 40, fast decay
```

The fast modulator decay creates the "clunk" or "bonk" attack characteristic of struck percussion, while the carrier's longer decay provides the sustaining resonance.

---

## DX7 Programming Reference

### Envelope Rate/Level Guide

```
Rate values (R1–R4): 0–99
  0  = extremely slow (many seconds)
  50 = medium (hundreds of milliseconds)
  90 = fast (tens of milliseconds)
  99 = near-instant (<5 ms)

Level values (L1–L4): 0–99
  0  = silence
  50 = approximately -6 dBFS
  99 = maximum amplitude
  
Note: L4 (R4) = release segment destination level — always set to 0 for complete releases
```

### Operator Level to Modulation Index

```
β (modulation index) = Op_Level / 99 × Ratio_Factor

Approximate β ranges for timbre:
  Level 30–40 → β ≈ 0.3–0.8  → subtle, warm, flute-like
  Level 55–70 → β ≈ 1.5–3.0  → piano, electric piano, plucked
  Level 80–95 → β ≈ 4.0–8.0  → organ, brass, aggressive
  Level 99    → β ≈ 10+      → noise, percussion, extreme distortion
```

### Keyboard Level Scaling

Allows different operator levels at different keyboard positions:

```
Left Curve / Right Curve: linear, -linear, exponential, -exponential
Left Depth / Right Depth: 0–99

Example for piano (brighten high notes):
  Op 2 (attack modulator) Right Curve: +linear, Right Depth: 40
  → attack click gets more prominent toward high register (mimics harder material strike)
```

---

## Recreating DX7 Sounds on Modern Tools

### Dexed (Free Plugin)

Dexed is a free, open-source DX7 emulator that accepts original DX7 SYX bank files. Load factory presets ROM1A through ROM1B for original E.PIANO 1, BRASS 1, BASS 1, VIBES, STRINGS.

```
Download factory banks: search "DX7 ROM1A.syx" (public domain)
Load in Dexed: File → Open Cartridge
```

### FM8 (Native Instruments)

FM8 expands beyond 6 operators. Replicate DX7 patches using operators A–F (F = Op 1 in DX7). Enable only 6 operators and set algorithms to match DX7 numbering.

### Yamaha Reface DX / DX-200

Hardware options with 4-operator FM. Subset of DX7 architecture — not direct conversion but similar workflow.

---

## Signature 80s Processing Chain

### Electric Piano

```
DX7 E.Piano → Chorus (rate 0.4Hz, depth 15ms, mix 40%) → Compression (2:1, fast attack)
            → Reverb (plate, 1.5s, 25% mix) → Slight saturation (+2dB clip for warmth)
```

### Brass

```
DX7 Brass → EQ (boost 2–4kHz +3dB for presence, cut 500Hz -2dB)
          → Short reverb (room, 0.6s) → Stereo widening (chorus L+R, ±12ms offset)
```

### Bass

```
DX7 Bass → Direct out (minimal processing) → slight compression (4:1) → HPF at 40Hz
```

---

## Reference Recordings by Patch Type

| Sound | Track | Artist | Year |
|-------|-------|--------|------|
| E.Piano | Against All Odds | Phil Collins | 1984 |
| E.Piano | What's Love Got To Do | Tina Turner | 1984 |
| Brass stabs | Jump | Van Halen | 1984 |
| Brass stabs | Axel F | Harold Faltermeyer | 1984 |
| Vibes/Marimba | In the Air Tonight | Phil Collins | 1981 (pre-DX7, Simmons) |
| Bass | Thriller | Michael Jackson | 1982 |
| Strings | Take On Me | A-ha | 1985 |
