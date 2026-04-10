---
title: "Dubstep/Neuro Growl Bass: FM and Wavetable Design"
category: "Genre Recipes"
tags: [dubstep, growl, neuro, FM, wavetable, Skrillex, bass, LFO, formant]
---

# Dubstep/Neuro Growl Bass: FM and Wavetable Design

## Historical Context

Dubstep originated in South London around 2001–2002 at venues like Plastic People. Early dubstep (Benga, Skream, Digital Mystikz) was dark, minimal, and sub-bass focused — heavy sub-bass with a half-time feel, not growl-oriented.

The **growl bass** emerged around 2008-2009 as producers began modulating complex filters and distortion with tempo-locked LFOs:

- **Rusko, Caspa** (UK, 2008) – heavier, more aggressive sound
- **Skrillex** ("Scary Monsters and Nice Sprites," 2010) – brought "brostep" to mainstream, defined the growl as a genre-defining element
- **Datsik, Excision** (2010–2012) – heavily distorted, "face-melting" growl
- **Feed Me, Doctor P** – UK wobble bass
- **Noisia, Phace** (post-2012) – "neurofunk" more complex, processing-heavy versions

Key tracks:
- "Scary Monsters and Nice Sprites" – Skrillex (2010) – archetypal brostep growl
- "Bass Head" – Bassnectar (2011)
- "Renegade" – Eekkoo (2012) – UK dubstep growl
- "Shockwave" – Noisia (2008) – early neurofunk
- "Stigma" – Noisia (2009)

---

## The Growl Mechanism

A growl bass is essentially a **vowel synthesizer** — the filter sweeps through frequency bands associated with vowel sounds (formants), creating the "wub wub wub" or "waaaah waaaah" vocal character.

```
Human vowel formant frequencies (approximately):
  "A" (as in "ah") : F1=800Hz, F2=1200Hz
  "E" (as in "ee") : F1=280Hz, F2=2250Hz  
  "I" (as in "ih") : F1=400Hz, F2=2000Hz
  "O" (as in "oh") : F1=450Hz, F2=800Hz
  "U" (as in "oo") : F1=300Hz, F2=870Hz

Growl synthesis moves a bandpass or comb filter through these formant frequencies
in rhythm, creating a vowel-like "talking" quality.
```

The LFO modulating the filter cutoff IS the "growl speed." Slower LFO = slow wobble. Faster = rapid stutter.

---

## Method 1: FM Synthesis Growl

FM synthesis naturally creates spectrally complex timbres that respond well to envelope modulation.

```
Carrier OSC:
  Frequency : 80–110 Hz (fundamental bass note)
  Waveform  : Sine

Modulator OSC:
  Frequency : same as carrier (ratio 1:1) or 2:1 for richer harmonics
  Level     : dynamically swept from 0 → 8 via LFO
              Low β → clean sine → High β → harmonically dense

Modulation index sweep:
  LFO shape  : Rising sawtooth or custom shape
  LFO rate   : 1/2 note (tempo-synced) for classic half-bar growl
               or 1/4 note for faster wobble
  β range    : 0 → 8 (sweeps from clean to heavily distorted FM)

Effect: At β=0, just a sine bass. As β rises to 8, progressively richer harmonics
        appear, then disappear, creating the characteristic "wub" movement.
```

---

## Method 2: Wavetable Growl (Most Common Modern Approach)

Wavetable synthesis allows sweeping through morphable waveforms in a table.

### Basic Wavetable Setup

```
Wavetable: "Talking Dirty" (Serum), "Formant" tables, or any aggressive
           saw-to-noise table with complex harmonic evolution

OSC A (main growl):
  Wavetable    : Aggressive/vocal wavetable
  WT Position  : Start at position 0 (sawtooth/simple)
  Unison voices: 4
  Detune       : 15 cents (stereo spread)

OSC B (sub bass body):
  Waveform     : Sine
  Semitones    : -24 (two octaves below OSC A)
  Level        : 60–70% of OSC A

LFO 1 → WT Position (OSC A):
  Rate   : 1/2 note (tempo synced)
  Shape  : Rising sawtooth (steady sweep upward through wavetable)
           or Triangle (sweeps up and back)
  Depth  : 50–70%
  
Effect: Wavetable position sweeps rhythmically, cycling through all the different
        waveforms in the table — creates rich, animated movement.
```

---

## Method 3: Formant Filter Growl

### Dual Bandpass Filter Vowel Synthesis

```
OSC: Distorted sawtooth or white noise (rich harmonic source)
     Note: The richer the harmonic content, the more pronounced the formant effect

Filter 1 (F1 — first formant):
  Type  : Bandpass (BPF 12dB/oct)
  Center: 300–800 Hz (sweeping range)
  Q/Res : 3–6 (moderate-high narrowness)
  LFO 1 : sweeps CENTER up and down

Filter 2 (F2 — second formant):
  Type  : Bandpass
  Center: 800–2500 Hz
  Q/Res : 4–8 (higher narrowness for vowel definition)
  LFO 2 : same rate as LFO 1 but offset 180° or different curve shape

Mix    : blend F1 and F2 outputs (50/50 or weighted toward F2)

Vowel sweep path examples:
  "OO" → "EE": F1 300Hz→280Hz, F2 870Hz→2250Hz  (dramatic)
  "AH" → "OH": F1 800Hz→450Hz, F2 1200Hz→800Hz
  "WOB": rapid sweep through "O" → "A" range (the "wub")
```

---

## Complete Serum Recipe: Classic Brostep Growl

### Oscillators

```
OSC A:
  Wavetable    : "Talking Dirty" (Serum built-in) or "Pulse Mod"
  Octave       : 0 (concert pitch)
  Unison       : 4 voices
  Detune       : 15 cents
  Pan spread   : 40%
  Blend        : 100% (all voices full)
  
OSC B:
  Waveform     : Basic Shapes → Sine
  Octave       : -2 (24 semitones down = 2 octaves)
  Level        : 65% (body/sub)
  
Sub Osc:
  Waveform     : Sine (additional sub support)
  Level        : 30%
```

### Filter

```
Filter 1:
  Type     : MG Low 24 (Moog-style 24dB ladder) 
  Cutoff   : 400 Hz
  Resonance: 30%
  Drive    : 15% (mild saturation in filter)
  
Filter 2 (Serial):
  Type     : Bandpass (BP 12)
  Cutoff   : 1200 Hz
  Resonance: 45%

Routing  : OSC A → Filter 1 → Filter 2 → FX
           OSC B → Filter 1 (LP only, skip BP) → FX
```

### Modulation

```
LFO 1 (main growl):
  Rate   : 1/2 (half-note, tempo synced)
  Shape  : Rising Sawtooth
  Destinations:
    WT Position OSC A : +50%
    Filter 1 Cutoff   : +35%

LFO 2 (secondary movement):
  Rate   : 1/4 (quarter note, tempo synced)
  Shape  : Sine
  Destinations:
    Filter 2 Cutoff   : +40%
    OSC A Unison Pan  : +20% (widening pulse)

ENV 2 (filter attack):
  Attack  : 0 ms
  Decay   : 200 ms
  Sustain : 50%
  Release : 150 ms
  → Filter 1 Cutoff: +20% (additional attack brightness)
```

### FX Chain in Serum

```
Distortion: Tube (type)
  Drive  : 25%
  Mix    : 100%

EQ:
  High-pass: 35 Hz
  Shelf cut: -3 dB at 300 Hz (clear mud)
  Boost    : +2 dB at 2.5 kHz (snarl/presence)
  High cut : -4 dB at 8 kHz

Reverb:
  Size   : 15% (small room — just enough air)
  Decay  : 0.4s
  Mix    : 10%
  Predelay: 5ms
```

---

## Neuro Bass (Post-2012 Evolution)

Neuro bass adds extreme processing complexity on top of the growl framework.

### Core Neuro Techniques

```
1. HEAVY WAVESHAPING
   Fold the waveform 2–4 times using a wavefolder or aggressive waveshaper
   Creates extremely dense harmonic spectrum
   
2. FORMANT PAIR (two moving bandpass filters):
   BPF 1: sweeps 200–1200 Hz
   BPF 2: sweeps 800–3000 Hz, phase-inverted movement relative to BPF 1
   When BPF 1 goes down, BPF 2 goes up → creates vowel formant pair movement
   
3. COMPLEX MODULATION:
   LFO rates at unusual rhythmic values: dotted eighth, triplet sixteenth
   Multiple LFOs with different shapes feeding same destination (additive)
   
4. FM WITHIN THE FILTER:
   Modulate filter resonance with audio-rate LFO (aliasing creates FM-like tones)
   Rate: 80–200 Hz (audio rate) → sidebands appear around filter resonance peak
   
5. COMB FILTER RESONANCE:
   Add comb filter with resonance → creates pitched metallic resonance on top of growl
   Tune comb to same root note as bass
```

### Noisia-Style Neuro Bass

```
Source   : Distorted sawtooth (heavy waveshaping pre-filter)
Filter 1 : BPF, 600Hz, Q=8, LFO 1/4 note + 1/8 note dotted (complex rhythm)
Filter 2 : BPF, 1500Hz, Q=10, LFO 1/4 note triplet (asynchronous to Filter 1)
Post     : Multiband compression (split at 200Hz, 1kHz)
           Low band    : 2:1 light compression (preserve sub punch)
           Mid band    : 4:1 (control growl dynamics)
           High band   : 6:1 (tame harsh high-frequency transients)
Exciter  : 2–5 kHz exciter (+3–4 dB harmonic enhancement)
M/S EQ   : Low-pass side channel at 150Hz (keep lows mono)
           → sub-bass is mono, growl mid-highs can be wide
```

---

## Half-Time Drum Pattern

Dubstep/brostep is characterized by a half-time feel — the kick and snare feel like they're at half-tempo.

```
At 140 BPM (standard dubstep):

Beat position : 1 . . . 2 . . . 3 . . . 4 . . . | 1 . . . 2 . . . 3 . . . 4 . . .
Kick          : X               X                 |                 X               
Snare         :                 X                 |                 X               
Hi-hat        : x   x   x   x   x   x   x   x   | x   x   x   x   x   x   x   x

Perceptually:
  Kick feels like beat 1 (bar 1) and beat 1 of bar 2
  Snare feels like beat 2 (in half-time feel)
  → 70 BPM feel within 140 BPM track

Growl bass LFO synced to 1/2 note = one full wobble per bar
```

---

## Reese Layer Under Growl

Standard production technique: layer a Reese bass underneath the growl for low-end support.

```
Growl bass:
  Frequency range: 100–3000 Hz
  Character      : Animated, harmonically complex
  Level          : Primary mix level

Reese bass (under growl):
  Frequency range: 35–150 Hz
  Character      : Detuned saws, subtle movement
  Level          : -6 to -8 dB below growl
  HPF            : Off (need full sub range)
  LPF on Reese   : 150 Hz (prevent mid-bass clash with growl)

Blend          :
  Sub-bass (Reese) provides body and weight
  Growl provides articulation and movement in the mid-bass
  Combined: full-spectrum dynamic bass system
```

---

## Processing Chain (Full)

```
Growl Bass Synth
  ↓
Pre-distortion EQ (cut extreme lows below 50Hz to prevent distortion muddiness)
  ↓
Distortion (Tube or Hyper, 20–35% drive)
  ↓
Formant/BPF filter modulated by LFO
  ↓
Multiband Compression:
  Band 1 (< 200Hz) : 2:1, medium attack
  Band 2 (200Hz–2kHz): 4:1, fast attack
  Band 3 (> 2kHz)  : 6:1, very fast
  ↓
Exciter (2–5 kHz, +3 dB harmonic enhancement)
  ↓
Stereo M/S EQ:
  Mid channel  : full bandwidth
  Side channel : LP at 150 Hz (mono sub-bass)
  ↓
Bus compressor (1.5:1, glue compression, slow attack 30ms)
  ↓
Output
```

---

## Parameter Reference Table

| Parameter | Slow/Light | Moderate | Heavy/Fast |
|-----------|-----------|----------|------------|
| LFO Rate | 1/1 (whole note) | 1/2 note | 1/4 or 1/8 note |
| Filter Q | 2–4 (smooth) | 5–8 (defined) | 9–15 (sharp) |
| Wavefold | 1x | 2x | 3–4x |
| Distortion Drive | 10–15% | 25–35% | 50%+ |
| WT Sweep Depth | 20–30% | 40–60% | 70–100% |
| Sub Level | -3 dB (subtle) | -6 dB | -10 dB (minimal sub) |
| Unison | Off | 2–3 voices | 4–6 voices |
| Detune | 5–8 cents | 12–18 cents | 20–30 cents |

---

## Genre Tempo Reference

| Subgenre | BPM | Half-time feel | 808 or Reese? |
|----------|-----|---------------|---------------|
| UK Dubstep (2006–2009) | 140 | Yes | Reese sub |
| Brostep (2010–2013) | 140 | Yes | Growl |
| Neurofunk (2012+) | 170–174 | Sometimes | Reese + Growl |
| Riddim dubstep (2015+) | 140 | Yes | Heavy distorted growl |
| Future Bass (2016+) | 150–160 | Sometimes | Wobble/chords |
| Deathstep/Doom | 140–150 | Yes | Maximum distortion |
