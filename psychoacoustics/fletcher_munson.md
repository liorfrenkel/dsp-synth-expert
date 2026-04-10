---
title: "Fletcher-Munson Curves and Equal-Loudness Contours"
category: "Psychoacoustics"
tags: [Fletcher-Munson, equal-loudness, loudness, phon, sone, hearing, frequency-response]
---

# Fletcher-Munson Curves and Equal-Loudness Contours

The Fletcher-Munson curves describe the frequency-dependent sensitivity of human hearing. They are fundamental to understanding why audio sounds different at different playback levels, and why producers must reference mixes at consistent volumes.

---

## History and Standards

### Original Research
- **Fletcher & Munson (1933)**: Harvey Fletcher and Wilden A. Munson at Bell Labs conducted the first rigorous measurement of equal-loudness contours
- Method: listeners adjusted sine tone level at each frequency until it sounded equally loud to a 1 kHz reference tone
- Result: a family of curves showing the SPL needed at each frequency to produce equal perceived loudness

### ISO 226:2003 Revision
- Updated equal-loudness contours based on improved measurement methodology
- Revised data showed significant differences from Fletcher-Munson at low frequencies and extreme levels
- ISO 226:2003 is now the international standard
- Key difference: revised curves show the ear is even more sensitive around 3–4 kHz than the original measurement, and low-frequency corrections differ at extreme levels

---

## Human Hearing Sensitivity: Absolute Threshold

- The **threshold of hearing** is defined as 0 dB SPL at 1 kHz (by convention)
- Most sensitive frequency range: **1–5 kHz** (due to ear canal resonance at ~3.4 kHz and ossicle mechanical response)
- Ear canal resonance at ~3.4 kHz adds approximately +10–15 dB of sensitivity (explains the "dip" in equal-loudness curves in this region)
- Less sensitive at:
  - Low frequencies (<200 Hz): requires significantly more SPL to be perceived as equally loud
  - Very high frequencies (>8 kHz): rapid sensitivity decrease, especially above 12 kHz

### Audible Frequency Range
- Typical human: 20 Hz – 20 kHz
- Infrasound (<20 Hz): felt but rarely heard; still psychoacoustically relevant (vertigo, pressure sensation)
- Ultrasound (>20 kHz): inaudible, but can produce intermodulation products with audible frequencies via nonlinear cochlear response
- Range narrows significantly with age: by age 50, typical upper limit is 14–16 kHz

---

## Equal-Loudness Contours: Key Data Points

### Phon Definition
The **phon** is the unit of loudness level.
- By definition: a tone at frequency `f` has loudness `L phons` if it sounds equally loud as a 1 kHz tone at `L` dB SPL
- Therefore: 1 kHz tone at 60 dB SPL = 60 phons (by definition, at any level)

### SPL Required at Different Frequencies for Constant Loudness

**40 phons (quiet listening, background music):**

| Frequency | Required SPL | Difference from 1 kHz |
|-----------|-------------|----------------------|
| 20 Hz     | ~97 dB      | +57 dB               |
| 40 Hz     | ~73 dB      | +33 dB               |
| 100 Hz    | ~52 dB      | +12 dB               |
| 500 Hz    | ~41 dB      | +1 dB                |
| 1 kHz     | 40 dB       | 0 dB (reference)     |
| 3.15 kHz  | ~34 dB      | −6 dB (most sensitive) |
| 10 kHz    | ~45 dB      | +5 dB                |
| 16 kHz    | ~65 dB      | +25 dB               |

**60 phons (moderate listening, typical mixing reference):**

| Frequency | Required SPL | Difference from 1 kHz |
|-----------|-------------|----------------------|
| 40 Hz     | ~88 dB      | +28 dB               |
| 100 Hz    | ~72 dB      | +12 dB               |
| 500 Hz    | ~61 dB      | +1 dB                |
| 1 kHz     | 60 dB       | 0 dB                 |
| 3.15 kHz  | ~54 dB      | −6 dB                |
| 10 kHz    | ~66 dB      | +6 dB                |
| 16 kHz    | ~80 dB      | +20 dB               |

**80 phons (loud listening):**

| Frequency | Required SPL | Difference from 1 kHz |
|-----------|-------------|----------------------|
| 40 Hz     | ~97 dB      | +17 dB               |
| 100 Hz    | ~85 dB      | +5 dB                |
| 500 Hz    | ~80 dB      | 0 dB                 |
| 1 kHz     | 80 dB       | 0 dB                 |
| 3.15 kHz  | ~75 dB      | −5 dB                |
| 10 kHz    | ~83 dB      | +3 dB                |

### Critical Observation: Bass Relativisation
At 80 phons vs 40 phons at 100 Hz:
- 40 phons: need 52 dB SPL at 100 Hz (12 dB more than 1 kHz)
- 80 phons: need 85 dB SPL at 100 Hz (only 5 dB more than 1 kHz)

**Conclusion**: bass and sub-bass are perceived as **relatively louder** at high playback levels than at low levels.

This explains:
- Why club music sounds bass-heavy at high SPL but thin at home
- Why consumer electronics add "loudness" bass boost (compensates for quiet listening)
- Why broadcast and streaming masters have more restrained low end than club mixes

---

## Loudness Units

### Phon
- Unit of loudness **level** (not perceptual loudness itself)
- Defined relative to 1 kHz reference
- Logarithmic scale

### Sone
- Unit of **perceived loudness** (psychoacoustic magnitude)
- 1 sone = 40 phons (by definition)
- Doubles every 10 phons increase:

```
sones = 2^((phons - 40) / 10)

phons = 10 × log2(sones) + 40
```

| Phons | Sones | Perceived loudness |
|-------|-------|-------------------|
| 10    | 0.025 | Barely audible    |
| 20    | 0.063 | Very quiet        |
| 30    | 0.125 | Quiet             |
| 40    | 1.0   | Reference         |
| 50    | 2.0   | Twice as loud     |
| 60    | 4.0   | Four times louder |
| 70    | 8.0   | Eight times       |
| 80    | 16.0  | Sixteen times     |
| 90    | 32.0  | Very loud         |
| 100   | 64.0  | Uncomfortably loud |
| 120   | 256.0 | Threshold of pain  |

---

## Frequency Weighting Filters

Derived from equal-loudness contours to model perceptual sensitivity in measurement.

### A-Weighting (dBA)
- Most common weighting for measuring sound levels
- Based on 40-phon equal-loudness curve (inverted and normalised)
- De-emphasises:
  - Sub bass (<100 Hz): −20 to −50 dB relative attenuation
  - Very high frequencies (>10 kHz): −10 dB relative
- Emphasises: 1–5 kHz range (peak sensitivity)
- Standard for: occupational noise, environmental noise regulations, consumer product specs
- **Limitation**: designed for moderate levels; not accurate at very high (80+ phons) or very low (20 phons) levels

### B-Weighting (dBB)
- Based on 70-phon curve
- Rarely used in practice

### C-Weighting (dBC)
- Near-flat response; slight rolloff below 100 Hz and above 4 kHz
- Used for high-level noise measurements, peak measurement in pro audio
- Standard for concert noise ordinances and hearing damage risk

### K-Weighting (LUFS)
- Used in EBU R128 and ITU-R BS.1770 loudness measurement
- Pre-filter: high-shelf +4 dB above 1.6 kHz (accounts for head/ear acoustics)
- RLB filter: HPF at 38 Hz (removes inaudible low-frequency contribution)
- More accurate than A-weighting for perceptual loudness of full-bandwidth audio content

```
K-weighting = Pre-filter × RLB (high-pass) filter
```

### Comparison of Weightings at Key Frequencies

| Frequency | dBA   | dBC   | K-weight (approx) |
|-----------|-------|-------|-------------------|
| 31.5 Hz   | −39.4 | −3.0  | −∞ (below HPF)    |
| 63 Hz     | −26.2 | −0.8  | −∞                |
| 125 Hz    | −16.1 | −0.2  | 0                 |
| 250 Hz    | −8.6  | 0     | 0                 |
| 500 Hz    | −3.2  | 0     | 0                 |
| 1 kHz     | 0     | 0     | 0                 |
| 2 kHz     | +1.2  | −0.2  | +0.5              |
| 4 kHz     | +1.0  | −0.8  | +3                |
| 8 kHz     | −1.1  | −3.0  | +3.5              |
| 16 kHz    | −6.6  | −8.5  | +2                |

---

## Critical Bands: Bark Scale

The cochlea resolves frequency differences within **critical bands** — frequency regions within which masking, loudness, and pitch perception operate differently.

### Critical Band Width
- Below 500 Hz: approximately **100 Hz wide**
- Above 500 Hz: approximately **20% of center frequency**

```
CB_Hz(f) = 25 + 75 × (1 + 1.4 × (f/1000)²)^0.69   (Zwicker approximation)
```

### Bark Scale (24 critical bands from 0–15.5 kHz)

| Bark | Center (Hz) | Width (Hz) |
|------|-------------|------------|
| 1    | 50          | 100        |
| 2    | 150         | 100        |
| 3    | 250         | 100        |
| 4    | 350         | 100        |
| 5    | 450         | 110        |
| 6    | 570         | 120        |
| 7    | 700         | 140        |
| 8    | 840         | 150        |
| 9    | 1000        | 160        |
| 10   | 1170        | 190        |
| 11   | 1370        | 210        |
| 12   | 1600        | 240        |
| 13   | 1850        | 280        |
| 14   | 2150        | 320        |
| 15   | 2500        | 380        |
| 16   | 2900        | 450        |
| 17   | 3400        | 550        |
| 18   | 4000        | 700        |
| 19   | 4800        | 900        |
| 20   | 5800        | 1100       |
| 21   | 7000        | 1300       |
| 22   | 8500        | 1800       |
| 23   | 10500       | 2500       |
| 24   | 13500       | 3500       |

**Key insight**: instruments and sounds in the same critical band mask each other most strongly. Instruments in widely separated Bark bands coexist more easily.

---

## Practical Implications for Mixing and Sound Design

### 1. Monitor at a Consistent Reference Level
- Recommended: **79–83 dB SPL** at the mix position (approximately where the curve is most linear)
- At this level, the equal-loudness curve is relatively flat across 200 Hz – 8 kHz
- Mixes made at this level translate better across playback systems
- **Check at quiet levels (~60 dB)**: sub bass will seem to disappear — this is normal; if mix sounds thin, bass may be over-represented at loud monitoring levels

### 2. The "Loudness Wars" Context
- Heavily limited/compressed masters maintain high average loudness
- This keeps the signal near the 80+ phon range where bass and treble are relatively louder
- Streaming normalization (−14 LUFS) partially negates this: all tracks play back at similar LUFS → dynamic masters sound natural, squashed masters sound lifeless

### 3. Consumer "Loudness" / "Bass Boost" Buttons
- Classic loudness contour circuits add low-shelf boost (~+6–12 dB at 100 Hz) and high-shelf boost at low volume
- Compensates for ears becoming less sensitive to bass/treble at low listening levels
- Based directly on the difference between equal-loudness curves at quiet vs. moderate levels

### 4. Low-Frequency Mixing Decisions
- Sub bass (20–60 Hz) is largely inaudible at typical home listening levels (60–65 dB)
- 808s and sub bass should carry their weight via **harmonics** (60–200 Hz range) for small-speaker translation
- "Presence" frequencies (1–5 kHz) are most easily perceived at all listening levels — use for intelligibility cues

### 5. High-Frequency Fatigue
- Prolonged exposure to 2–5 kHz (the most sensitive region) causes listening fatigue fastest
- EQ choices in this range should be checked after ear rest
- Recording studios: diffuse-field monitoring (far-field) helps reduce ear fatigue vs near-field

---

## Loudness Metering Standards

### LUFS / LKFS
- **LUFS**: Loudness Units relative to Full Scale
- **LKFS**: Loudness, K-weighted, relative to Full Scale (identical measurement, ITU designation)
- Measurement includes:
  - **Momentary**: 400ms sliding window
  - **Short-term**: 3-second window
  - **Integrated**: entire program, gated (ignores silence)
  - **LRA (Loudness Range)**: statistical spread of short-term loudness (dB)

### Platform Targets (2024)

| Platform         | Integrated LUFS | True Peak |
|------------------|-----------------|-----------|
| Spotify          | −14 LUFS        | −1 dBTP   |
| Apple Music      | −16 LUFS        | −1 dBTP   |
| YouTube          | −14 LUFS        | −1 dBTP   |
| Tidal            | −14 LUFS        | −1 dBTP   |
| SoundCloud       | −14 LUFS        | −1 dBTP   |
| Broadcast (EBU R128) | −23 LUFS    | −1 dBTP   |
| Film/TV          | −24 LUFS        | −2 dBTP   |

### True Peak vs Sample Peak
- Intersample peaks (ISPs): digital waveform reconstructed by D/A converter can produce peaks between samples that exceed 0 dBFS
- Standard peak meter: measures samples only — misses ISPs
- **True peak meter**: oversamples by 4× to detect ISPs
- Standard requirement: master to −1 dBTP (true peak) to allow headroom for D/A reconstruction

---

## Summary: Equal-Loudness for Practical Audio Work

1. Human hearing is **not flat** — most sensitive at 1–5 kHz, least sensitive at sub bass and extreme highs
2. Sensitivity profile **changes with level**: at high SPL, bass sounds relatively louder
3. Mix at **consistent moderate level** (79–83 dB SPL) for accurate perception
4. **Check mixes at low volume**: bass disappearing is expected; loss of presence indicates mid-range problem
5. A-weighting approximates perceptual sensitivity for noise measurement at moderate levels
6. LUFS with K-weighting is the standard for perceptual loudness measurement in production
7. Critical bands group frequencies perceptually — sounds in the same band mask each other most strongly
