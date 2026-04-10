---
title: "Auditory Masking and Frequency Perception"
category: "Psychoacoustics"
tags: [masking, auditory, critical-band, simultaneous, temporal, psychoacoustics]
---

# Auditory Masking and Frequency Perception

Auditory masking describes how the presence of one sound (the masker) makes another sound (the maskee or probe) harder or impossible to hear. It is one of the most important phenomena in music production, psychoacoustics, and perceptual audio coding (MP3, AAC).

---

## Types of Masking

### 1. Simultaneous Masking (Frequency Masking)
- Masker and masked signal occur at the same time
- Loud sound raises the threshold of hearing for nearby frequencies
- Most relevant to mixing decisions: competing instruments in the same frequency range

### 2. Temporal Masking
- Masker and masked signal are separated in time
- Subdivided into:
  - **Forward masking**: masker makes subsequent signals harder to hear
  - **Backward masking**: masker makes preceding signals harder to hear

### 3. Informational Masking
- Cognitive interference rather than peripheral auditory masking
- Two simultaneous speech streams — intelligibility drops even if spectral masking is minimal
- Less relevant to music production but important in broadcast/voiceover

---

## Simultaneous Masking: Mechanism

The cochlea separates frequencies along the basilar membrane (place theory):
- High frequencies: near the base (rigid, stiff)
- Low frequencies: near the apex (flexible)

A loud tone at frequency `f_m` causes physical displacement of the basilar membrane in a region that extends asymmetrically:
- **Below `f_m`**: displacement drops steeply (steep low-frequency skirt)
- **Above `f_m`**: displacement extends further (gradual high-frequency tail)

This asymmetry creates the **upward spread of masking**: a loud low-frequency sound masks high-frequency sounds more effectively than a loud high-frequency sound masks lower frequencies.

### Masking Curve Shape (Simultaneous)

```
Masking threshold (dB above absolute threshold)

         ^
         |      Masker
         |       ↓
    80 dB|      ███
         |     █████
    60 dB|    ████████
         |  ████████████░░░
    40 dB|██████████████░░░░░░░░
         |      ████████████████░░░░
    20 dB|              ████████████░░
         |                      ████████
     0 dB+────────────────────────────────→ Frequency
              f_m                         (log scale)
```

- Steep low-frequency skirt (−100 dB/octave)
- Gradual high-frequency tail (−25 to −40 dB/octave, steepness depends on masker level)
- Width increases with masker level (louder masker → broader masking region)

---

## Critical Band and Masking

### Critical Bandwidth
The critical band is the fundamental resolution unit of the cochlea. Sounds within one critical band interact strongly (masking, roughness, beating). Sounds in separate critical bands are processed more independently.

```
CB_Hz(f) ≈ 100 Hz          for f < 500 Hz
CB_Hz(f) ≈ 0.20 × f        for f > 500 Hz (20% of center frequency)
```

More precise Zwicker formula:
```
CB = 25 + 75 × (1 + 1.4 × (f/1000)²)^0.69
```

### Masking Within vs Across Critical Bands

- **Within one critical band**: masking is strongest; a tone 15–40 dB below the masker can be inaudible
- **Across critical bands**: masking weakens rapidly with frequency distance
- Two-tone masking: masking drops approximately 40–60 dB/critical band as frequency separation increases

### Masking Threshold Formula (Approximate)

For a masker at level `L_m` (dB SPL) at frequency `f_m`, the masking threshold at frequency `f` is approximately:

```
MT(f) = L_m - SF(f, f_m)

where SF (spread function) in dB:
  If f < f_m:  SF ≈ 27 dB/Bark (steep low-frequency skirt)
  If f > f_m:  SF ≈ (24 + 0.23*(f_m/1000) - 0.2*L_m) dB/Bark
```

---

## Masking Thresholds: Practical Numbers

### Tone-in-Tone Masking

| Frequency Separation | Masking Threshold |
|---------------------|-------------------|
| Same frequency       | −40 to −60 dB below masker |
| 1 critical band apart | −30 to −40 dB below masker |
| 2 critical bands      | −50 to −60 dB below masker |
| 3+ critical bands     | Approaches absolute threshold |

### Narrowband Noise Masker
- Noise is a more effective masker than pure tones
- Noise masker at the same level masks a wider frequency region
- White noise masker: threshold elevation of ~40 dB within the noise band

### Signal-to-Masker Ratio (SMR)
Used in perceptual audio coding:

```
SMR(f) = signal_level(f) - masking_threshold(f)
```

- If SMR > 0: signal is audible above the masked threshold
- If SMR < 0: signal is below the masking threshold → can be discarded without audible effect
- MP3/AAC uses this to determine bit allocation per frequency band

---

## Temporal Masking

### Forward Masking
- A loud sound raises the hearing threshold for subsequent sounds
- Duration: up to **200 ms** after the masker ends
- Decay curve: non-linear; most effect in first 50 ms, decays gradually to 200 ms
- Threshold elevation: can be 20–40 dB above normal threshold immediately after masker

```
Masking threshold after masker ends:
t = 0 ms:    ~30–40 dB above absolute threshold
t = 50 ms:   ~15–25 dB above absolute threshold
t = 100 ms:  ~5–15 dB above absolute threshold
t = 200 ms:  ~0 dB (returns to normal)
```

### Backward Masking (Pre-masking)
- The nervous system cannot fully process temporal order at short timescales
- A subsequent loud sound can mask a preceding sound arriving up to **30 ms** earlier
- Much shorter range than forward masking
- Mechanism: neural integration window; the auditory system averages over ~30 ms
- Perceptual consequence: very short transients before a loud event may be inaudible

### Temporal Masking in Music
- Kick drum transient: forward masks snare features for ~20–50 ms after the kick
- Loud crash cymbal: backward masks preceding hi-hat detail
- Pre-ringing from linear phase EQ: audible if pre-ringing falls outside backward masking window (>30 ms)

---

## Specific Masking Situations in Mixing

### Sub Bass vs Kick Drum (40–120 Hz)
- Sub bass fundamental (40–60 Hz) and kick drum fundamental (60–100 Hz) overlap
- Both occupy the same or adjacent critical bands (Bark 1–3)
- Masking creates competition: one obscures the other depending on level
- **Solutions**:
  - Sidechain compress bass from kick (duck bass during kick transient — use forward masking window)
  - EQ split: cut bass at 80 Hz where kick peaks; boost kick at 40–50 Hz where bass peaks
  - Frequency band ownership: kick owns 80–100 Hz, bass owns 40–60 Hz

### Synth Lead vs Vocals (500 Hz – 4 kHz)
- Both occupy the 1–4 kHz range — highest ear sensitivity + maximum masking interaction
- In a dense mix, one will mask the other
- **Solutions**:
  - Automate volume of synth lead downward during vocal phrases
  - EQ: gentle notch in synth lead at 1–3 kHz (carve space for vocal formants)
  - Timing offset: synth lead plays in gaps between vocal phrases
  - M/S processing: vocal in center (M), synth lead panned to sides

### Guitar vs Synth Pad (200 Hz – 3 kHz)
- Electric guitar body (200–500 Hz) + synth pad warmth zone (200–600 Hz) compete
- **Solutions**:
  - Cut low-mids in synth (250–400 Hz) when guitar is present
  - HPF synth pad at 150–200 Hz
  - Stereo placement: guitar mono/center, pad stereo/sides

### Hi-Hat vs Cymbal Sizzle (5–12 kHz)
- Dense upper partials from hi-hats and cymbals compete for brightness
- High-frequency masking is narrower (higher Q on Bark scale)
- **Solutions**:
  - Automate ride/crash levels to maintain hi-hat clarity
  - LPF open hi-hat slightly

---

## Ear Fatigue: Masking and Sensitivity

- Prolonged exposure to sounds in the 2–5 kHz range causes temporary threshold shift (TTS)
- This is both the most sensitive and most commonly masked frequency region in dense mixes
- TTS typically recovers within minutes to hours; repeated loud exposure causes permanent threshold shift (PTS)
- After fatigued listening: perception of 2–5 kHz drops → mix decisions at 3–4 kHz become unreliable
- **Practical**: take breaks every 30–45 minutes; never mix at loud levels for extended periods

---

## Haas Effect (Precedence Effect)

When the same sound arrives from two sources at slightly different times:

### Time Delay Ranges

| Delay    | Perception |
|----------|------------|
| 0–1 ms   | Barely perceptible image shift |
| 1–30 ms  | Sound fused into single image at first-arriving speaker; adds apparent width |
| 30–50 ms | Image still fused but with pitch-like coloring (comb filtering) — "pre-echo" region |
| >50 ms   | Two distinct sounds perceived (echo) |

### Haas Effect Formula
If sound arrives from left at t=0 and right at t=Δt:
- For Δt < 30 ms: listener hears a single sound from the left source
- Even if the right (delayed) source is up to 10 dB louder, the image remains at the first source (precedence)

### Practical Applications

**Haas delay for width:**
- Delay right channel copy by 10–25 ms → listener perceives width
- Simple, mono-compatible technique
- Warning: can create comb filtering if >5–10 ms delay is used without treatment

**Slapback echo (50–150 ms):**
- Single repeat — more distinct than Haas but less than full echo
- Classic Rockabilly, Elvis-era vocal treatment
- Preserves intelligibility while adding space and "vintage" character

**Double tracking:**
- Two separate performances of same part, each played back from opposite sides
- Natural timing variations (5–30 ms at most) = Haas-like fusion, but no comb filtering
- Sounds much more natural than artificial Haas delay
- ADT (Artificial Double Tracking): tape-era technique using slight speed variation to create delay

---

## Binaural Hearing: Localisation Cues

The auditory system uses three primary cues to localise sounds in three dimensions:

### ITD (Interaural Time Difference)
- Time difference between arrival of sound at left vs right ear
- Maximum ITD: ~660–700 µs (for source at 90° to the side)
- Used for localisation: 0–1.5 kHz (low frequencies, where phase is trackable)
- The auditory system can detect ITD differences as small as 10–20 µs

```
ITD(θ) = (d / c) × sin(θ)    (simplified spherical head model)

where d = 0.215 m (inter-ear distance), c = 343 m/s (speed of sound)
Maximum: 0.215 / 343 ≈ 0.000627 s = 627 µs
```

### ILD (Interaural Level Difference)
- Level difference between the two ears due to head shadow effect
- The head acts as a diffraction barrier at high frequencies
- Significant above **1.5 kHz** (wavelengths smaller than head diameter)
- At 90° azimuth: ILD can be 15–25 dB at 4–8 kHz
- Primary localisation cue for high frequencies

### HRTF (Head-Related Transfer Function)
- Frequency-dependent filtering by the outer ear (pinna), head, and torso
- Encodes elevation information (up/down) that ITD and ILD alone cannot provide
- Also encodes front/back distinction
- Unique to each individual (pinna shape differs)
- Binaural audio: convolve monaural signal with HRTF pair (L ear, R ear) → 3D localisation on headphones
- Databases: CIPIC (UC Davis), MIT KEMAR, Listen (IRCAM)

---

## Masking in Perceptual Audio Coding

MP3, AAC, Ogg Vorbis, and similar codecs exploit masking to reduce file size.

### Psychoacoustic Model Steps
1. Split signal into frequency subbands (using MDCT — Modified Discrete Cosine Transform)
2. Compute signal level in each subband
3. Compute masking threshold for each subband based on:
   - Spreading function from neighbouring subbands
   - Temporal masking from previous frames
   - Absolute hearing threshold
4. Allocate bits: more bits to subbands where signal is above masking threshold, fewer (or zero) where signal is masked
5. Quantisation noise must stay below the masking threshold in each band

### Perceptual Transparency
- At high bitrates (192+ kbps MP3): quantisation noise is well below masking threshold → inaudible
- At low bitrates (64 kbps): noise may exceed masking threshold in some bands → pre-echo, pumping artefacts
- **Pre-echo**: temporal masking window may be exceeded by sudden transients → the quantisation noise before an attack becomes audible (classic low-bitrate artefact)

---

## Summary: Masking Rules for Mixing

| Rule | Application |
|------|-------------|
| Low frequencies mask high more than vice versa | Handle sub/bass before building midrange |
| Masking strongest within one critical band | Instruments sharing frequency range compete directly |
| Forward masking up to 200 ms | Leave space after loud transients (kick/snare) |
| Haas <30 ms = width, >50 ms = echo | Use short delays for width, not slapback |
| 2–5 kHz: highest sensitivity + highest masking | Mix at moderate levels; check with rested ears |
| ITD relevant below 1.5 kHz; ILD above | Pan differently at different frequencies for natural placement |
| LUFS integrates K-weighting | Equal-loudness contour is built into LUFS measurement |
