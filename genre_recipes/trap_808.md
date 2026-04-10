---
title: "Trap 808: Sub Bass Design and Production"
category: "Genre Recipes"
tags: [808, trap, sub-bass, Roland, hip-hop, pitch-slide, Atlanta]
---

# Trap 808: Sub Bass Design and Production

## Origin and History

The Roland TR-808 Rhythm Composer (1980) was a programmable drum machine originally used for demos and practice. Like the TB-303, it was commercially unsuccessful and discontinued in 1983. Producers discovered its bass drum output could be pitched and extended into a melodic sub-bass instrument.

### Timeline

- **1986–1990**: Miami bass / booty bass producers (2 Live Crew, Luke Records) pitch the 808 kick down to create "bass"
- **1990s**: Atlanta crunk music (Lil Jon, YING YANG TWINS) establishes the extended 808 as a genre staple
- **2007–2010**: T-Pain, Lex Luger, and Shawty Redd codify the modern trap 808
- **2012–present**: Metro Boomin, Mike Will Made-It, Southside establish the 808 as the primary melodic bass element in trap

Key tracks and producers:
- **"Hard in da Paint"** – Waka Flocka Flame (prod. Lex Luger, 2010) – explosive high 808s
- **"Versace"** – Migos (prod. Zaytoven, 2013)
- **"Love Sosa"** – Chief Keef (prod. Young Chop, 2012) – long sliding 808s
- **"Bad and Boujee"** – Migos (prod. Metro Boomin, 2016)
- **"Sicko Mode"** – Travis Scott (prod. Metro Boomin, Tay Keith, et al., 2018)

---

## TR-808 Bass Drum Circuit

The original 808 bass drum uses an analogue circuit:

```
Bridged-T network oscillator (tuned to 40–80 Hz)
  + Pitch envelope (fast decay from high frequency to low)
  + Amplitude envelope (decay time knob = how long bass sustains)
  + Noise burst (very short, creates the attack "snap")

Controls:
  Tone    : cutoff of high-frequency filter → darker/brighter attack
  Decay   : amplitude envelope release time (50ms–2000ms)
  Level   : output volume
  (Tuning : not on original 808 panel — fixed ~65Hz — but CV-pitch-able)
```

The original oscillator produces a **tuned triangle/sine wave** at approximately 65 Hz when decay is long. Short decay = punchy kick. Long decay = bass instrument.

---

## Modern 808 Synthesis

### Method 1: Pitch Envelope Synthesis (Most Common)

This method replicates the 808 circuit directly.

```
OSC:
  Waveform        : Sine (pure fundamental, no harmonics)
                    or Triangle (subtle odd harmonics → slightly more body)
  Base frequency  : Same as target note (e.g., 73.4 Hz for D2)

Pitch Envelope:
  Start pitch     : +24 to +36 semitones above note (2–3 octaves)
                    e.g., for D2 (73.4 Hz): start at D4 (293.7 Hz) or D5 (587.3 Hz)
  Decay to note   : 10–50 ms (fast — this IS the "knock" attack transient)
  Hold at note    : sustained for remainder of note length

Amplitude Envelope:
  Attack  :   0 ms (instant)
  Decay   : 500–4000 ms (this controls the "woofer" length)
  Sustain :   0% (pure decay — note dies naturally, no sustain level)
  Release :  50 ms (short release when note is lifted)
```

The **pitch envelope** at the start creates the characteristic "knock." The oscillator starts high-pitched (creating the click/attack transient) and falls to the note pitch within 10-50ms. This pitch drop IS the 808 attack character.

### Method 2: Portamento Slide (For Melodic Trap 808s)

Used for the "sliding 808" ubiquitous in modern trap:

```
OSC      : Sine, same as Method 1 but no pitch envelope start
Portamento: 200–800 ms (long — creates the "WOOM" slide between notes)
Playing  : Legato — hold previous note while pressing next
           The portamento slides the pitch smoothly

Note pitching:
  Play notes that match the chord progression
  Each note held for 1–2 bars, slid into next note legato
```

This method creates melodic movement across the song — the 808 becomes the harmonic bass voice.

### Method 3: Sample-Based 808 (Most Common in Practice)

Most producers use 808 samples from sample packs, then pitch-tune them:

```
Source   : 808 kick sample (various lengths: 1s, 2s, 3s, 4s)
Pitch    : Tune to key of track using pitch detection (e.g., Melodyne, auto-tune)
Length   : Chop or extend to match desired note length
Layering : Layer pitched 808 sample under short 808 "kick" sample for attack
```

---

## The "Knock" Anatomy

```
0–10ms    : High-frequency pitch transient → the "knock" or "click"
            (from pitch env start high + fast decay)
10–80ms   : Pitch settling to note frequency + amplitude at peak
80ms–1s   : Fundamental tone sustaining (sub-bass body)
1s–4s+    : Natural amplitude decay (longer with high Decay setting)

Each phase:
  Knock    → translates on small speakers, phone speakers, laptop speakers
  Body     → subwoofer, car audio, club system
  Decay    → fills space between notes in arrangement
```

---

## Saturation for Small Speaker Translation

A pure 808 sine wave below 80Hz is **inaudible** on:
- Phone speakers (typically roll off below 150–200Hz)
- Laptop speakers (roll off below 200Hz)
- Small Bluetooth speakers (roll off below 100–150Hz)
- TV speakers

Solution: **Add harmonic content via saturation**

```
Saturation types and their harmonic output:
  Tape saturation   : strong 2nd harmonic (octave) + 3rd harmonic
                      Character: warm, "vintage"
  Tube saturation   : strong 2nd harmonic, weaker 3rd
                      Character: smooth, musical
  Transistor/drive  : 2nd + 3rd + 5th harmonics
                      Character: brighter, more complex
  Hard clip         : odd harmonics (3rd, 5th, 7th dominant)
                      Character: harsh, buzzy — careful with this

For 808 at 73.4 Hz (D2):
  Tape sat → adds 146.8 Hz (D3) and 220.2 Hz (A3) — audible on small speakers
  
Amount: +3–6 dB of saturation (subtle — not a distortion effect)
        Enough to add 2nd/3rd harmonics, not so much as to change the fundamental character
```

---

## Sidechain Compression

```
Concept : The 808 and kick drum occupy the same frequency range (40–80Hz).
          Simultaneous playback creates a muddy, undefined low end.
          Sidechain compression makes the 808 duck when the kick hits.

Settings:
  Trigger source : Kick drum (direct output or ghost track)
  Ratio          : 4:1 to 8:1
  Attack         :  1–5 ms (fast — catches the kick transient immediately)
  Release        : 100–200 ms (medium — 808 comes back quickly after kick)
  GR             : -6 to -12 dB (808 ducks significantly on kick)

Visual check: In your DAW, view kick and 808 on spectrum analyzer simultaneously.
              With sidechain active, 808 energy at 60–80Hz should dip on kick hits.

Alternative: Use automation rather than sidechain —
             manually draw -8dB reduction on 808 track at kick hit points.
```

---

## EQ

```
High-pass filter : 30 Hz (remove sub-sonic rumble — inaudible and uses headroom)
                   Note: do not cut too high — 808 fundamental often 40–80Hz

Low shelf        : +2–3 dB at 60–80 Hz (boost fundamental, especially after sidechain)

Mid-bass notch   : -3 to -6 dB at 200–400 Hz (remove "boxy" mud)
                   The 808 naturally has energy here from the pitch envelope start
                   Cutting here clarifies the kick punch in the same range

Presence (if distorted):
                 : +1–2 dB at 1–3 kHz (adds click/presence for small speaker translation)
                   Only relevant if 808 has saturation — otherwise no energy here

High cut         : -6 dB/oct above 3–4 kHz (optional, remove any noise from saturation)
```

---

## Tuning the 808

**Critical**: The 808 must be in tune with the track. A de-tuned 808 clashes with melodic elements and chord progressions.

```
Step 1: Identify key of track (e.g., F minor)
Step 2: Determine bass note for each section (e.g., F1, C1, Ab1, Eb1)
Step 3: Pitch 808 sample/synth to match
  
Methods:
  - Manual pitch shift: slow down/speed up sample in sampler
  - Melodyne: analyze 808, correct pitch to target note
  - Pitch envelope: set end pitch of pitch env to target frequency
  
Verification:
  - Play 808 with the main chord/melody
  - If it sounds "wrong" or "tense," re-tune
  - Use a spectrum analyzer to confirm 808 fundamental matches note frequency
```

---

## Processing Chain

```
808 Source (synth or sample)
  ↓
[Optional: Pitch envelope for knock]
  ↓
Tape Saturation (+3–5 dB drive)
  ↓
Compression (4:1–8:1, attack 2ms, release 100ms)
  ↓
Sidechain from kick (GR -6 to -12 dB)
  ↓
EQ (HPF 30Hz, cut 200–400Hz, optional presence)
  ↓
Parallel Compression (blend 30% of heavily-compressed parallel signal)
  ↓
Bus Limiter (catch peaks, ensure consistent level)
```

### Parallel Compression for 808

```
Setup : Split 808 signal
  Dry path  : Main 808 with moderate compression
  Wet path  : Same 808, heavy compression (20:1, -20dB GR), boosted +6dB
  Blend     : Mix dry 70% / wet 30%

Effect: Parallel compression retains the natural dynamics of the 808's decay
        while adding sustain and punch from the heavily-compressed signal.
        The wet signal "fills in" quieter portions of the 808 decay.
```

---

## Pattern Design

### Basic Trap 808 Pattern

```
At 140 BPM, 4/4:

Bar 1:
  Kick : beat 1, beat 3 (or: beat 1, beat 2-and, beat 3, beat 4-and)
  808  : whole note F1, slides into next bar

Bar 2:
  808  : continues F1 → slides to C1 on beat 3

Bar 3:
  808  : C1 whole note → slides to Ab1

Bar 4:
  808  : Ab1 → Eb1 → Bb1 (faster slides at end of phrase)
```

### Half-time Trap Feel

```
Hi-hat: sixteenth notes (4 per beat) at 140 BPM
Kick  : beat 1 only (or beat 1 + beat 2-and)
808   : follows underlying harmony, sliding between chord roots
Snare : beat 3 only (half-time feel)
```

### Slide Amount Guide

```
Slow slide (600–800ms): Used for phrases where 808 moves across a bar boundary.
                        Creates melodic, singing quality.
                        
Medium slide (200–400ms): More punctuated transitions.
                          Between notes within same bar.

Fast slide (50–150ms): Quick ornaments, note bends. 
                       Often used for expressive emphasis.
```

---

## Common Plugins and Sample Sources

### Synthesis Plugins

```
Scaler 2      : Chord detection → helps tune 808s to key
iZotope Ozone : Mid/side EQ for 808 low-end management
Fab Filter Pro-Q3: Precise EQ including dynamic EQ on 808
Waves Submarine: Sub-bass enhancement and frequency translation tool
```

### Sample Packs and Kits

```
Splice        : "808 Warfare" by Native Instruments (industry standard 808 samples)
Producer Spot : Various 808 trap kits
Drumify       : Custom 808 generation
Sample Magic  : SM808 — multi-sampled 808 across full pitch range
```

---

## Regional Style Variations

| Region/Sub-style | 808 Character | Decay | Saturation | Slide |
|-----------------|---------------|-------|------------|-------|
| Atlanta trap | Long, deep, heavy | 1.5–3s | Medium (tape) | Long slides |
| NYC drill | Punchy, shorter | 500ms–1s | Moderate | Minimal slides |
| UK drill | Dark, filtered | 1–2s | Heavy | Some slides |
| Chicago drill | Punchy, hard | 500ms–1s | Heavy overdrive | Short |
| SoundCloud rap | Distorted, blown-out | 1–2s | Very heavy (clip) | Pitch bent |
| Melodic trap | Clean, sustained | 2–4s | Subtle | Many long slides |

---

## Advanced Techniques

### 808 Chord Stacks

```
Layer 3 separate 808 voices:
  Voice 1: Root note (e.g., F1)
  Voice 2: Perfect 5th (C2) at -6dB
  Voice 3: Minor 3rd (Ab1) at -12dB
  
High-pass Voice 2+3 at 100Hz → remove sub clash
Result: Harmonic fullness without sub-bass muddiness
```

### Granular 808 Stretching

```
Take an 808 sample → run through granular processor (Ableton Simpler, GranuLab)
Grain size: 100–200ms
Stretch: 200–300% (extend decay dramatically)
Pitch scatter: 0 (preserve pitch)
Result: extended, evolving 808 decay with texture
```
