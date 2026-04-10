---
title: "Stereo Field Perception and Spatial Audio"
category: "Psychoacoustics"
tags: [stereo, mid-side, Haas, panning, spatial, width, mono-compatibility, HRTF]
---

# Stereo Field Perception and Spatial Audio

Stereo field perception encompasses how the auditory system extracts spatial information from two-channel audio. This knowledge directly informs panning decisions, stereo processing, mono compatibility, and spatial audio design.

---

## Stereo Fundamentals

### Two-Channel Stereo Standard
- Left (L) and Right (R) channels, reproduced from speakers at ±30° from center axis
- Phantom center image: when L = R, brain localises sound to center between speakers
- Hard panning: signal in one channel only → sound perceived at that speaker
- Panning law determines level compensation as signal is panned between center and side

### Stereo vs Binaural
- **Stereo**: designed for loudspeaker reproduction; uses L/R level differences
- **Binaural**: designed for headphone reproduction; uses HRTF-filtered signals for 3D localisation
- Binaural on speakers: partially breaks down (no head shadow separation possible)
- Stereo on headphones: sounds "inside the head" — no external externalisation without HRTF

---

## Mid-Side (M/S) Processing

### Encoding: Stereo → Mid/Side

```
M = (L + R) / 2      (mono/center content — what both channels share)
S = (L - R) / 2      (side/difference content — what separates L from R)
```

Or in unit-gain form:
```
M = (L + R) × 0.707     (−3 dB)
S = (L − R) × 0.707
```

### Decoding: Mid/Side → Stereo

```
L = M + S
R = M − S
```

Verification: if S = 0 (pure mono), then L = R = M (correct).
If M = 0 (fully differential), then L = S, R = −S (anti-phase mono — unusual but valid).

### Mono Compatibility via M/S
- In mono playback: L_mono = (L + R) / 2 = M
- The **S channel cancels completely in mono**
- Therefore: any content placed exclusively in the S channel (hard panning L−R difference) disappears in mono
- **Rule**: always check that critical content (bass, kick, lead vocal) is present in the M channel

### M/S EQ

Process M and S channels independently before decoding back to L/R:

```
Input stereo → M/S encode → EQ M channel → EQ S channel → M/S decode → Output stereo
```

#### Common M/S EQ Moves

| Move | Channel | Frequency | Action | Effect |
|------|---------|-----------|--------|--------|
| Bass mono | S | <200 Hz | HPF | Forces sub bass to center; prevents phase issues |
| Width boost | S | >8 kHz | +3 dB shelf | Wider, airier stereo image |
| Tighten | S | 200–2k Hz | −2 dB | Less width in midrange; more focused |
| Center clarity | M | 1–5 kHz | +1–2 dB | Pushes lead/vocal forward |
| Remove harshness | M | 3–5 kHz | −2 dB | Tames center harsh zone |
| Width check | S | all | mute | Reveals pure mono content |

---

## Panning Laws

Panning law determines how signal level changes as a source is panned from center to full L or R.

### Equal Power (-3 dB at center, panning law most common)

```
L_level = cos(θ × π/2)    where θ = 0 (hard L) to 1 (hard R), center = 0.5
R_level = sin(θ × π/2)

At center (θ = 0.5):
L = cos(π/4) = 0.707 = −3.01 dB
R = sin(π/4) = 0.707 = −3.01 dB
Sum (mono): L + R = 1.414 = +3.01 dB above either channel
```

Why: equal power preserves consistent loudness from any pan position (physically: power = L² + R²).

### Linear (-6 dB at center)

```
L_level = 1 − θ           for θ ∈ [0, 1]
R_level = θ

At center (θ = 0.5):
L = R = 0.5 = −6.02 dB
Sum: L + R = 1.0 (unity)
```

Preserves level in mono sum. Sounds quieter at center than equal power.

### Sine/Cosine vs Linear Comparison

| Position | Equal Power L/R | Linear L/R |
|----------|-----------------|------------|
| Hard L   | 1.0 / 0.0       | 1.0 / 0.0  |
| 75% L    | 0.924 / 0.383   | 0.75 / 0.25 |
| Center   | 0.707 / 0.707   | 0.5 / 0.5  |
| 75% R    | 0.383 / 0.924   | 0.25 / 0.75 |
| Hard R   | 0.0 / 1.0       | 0.0 / 1.0  |

Most DAWs default to −3 dB equal power panning law. Some (Pro Tools) offer a choice.

### 0 dB Panning Law (No Compensation)

```
L = 1.0 (full level) at center
R = 1.0 (full level) at center
```

Mono sum is +6 dB vs full L or R. Not used in standard DAW mixing but common in some hardware summing architectures.

---

## Stereo Width Techniques

### 1. Chorus / Doubling
- Slightly time-delayed copy of signal (5–20 ms) panned to opposite side
- Natural-sounding width
- Slight comb filtering (less than 5 ms delay)
- Best for: synth pads, guitars, vocals
- Mono compatible (comb filtering reduces but signal present)

### 2. Haas Delay (Panned Delay)
- Delay one channel copy by 10–25 ms, pan to opposite side from original
- Very effective width, sounds natural within Haas fusion window
- Warning: mono sum can have comb filtering at delay frequency
  - At 15 ms delay: comb notch at 1/0.015 = 66 Hz, 133 Hz, 200 Hz...
- Safer: apply Haas at 10–20 ms (within fusion window), EQ notch in S channel at comb frequencies

```python
# Simple Haas stereo widener
delay_samples = int(0.015 * sample_rate)  # 15 ms

for n in range(num_samples):
    L_out[n] = mono_in[n]
    R_out[n] = mono_in[n - delay_samples] if n >= delay_samples else 0.0
```

### 3. Mid-Side Width Processing
- Boost or cut S channel gain to control stereo width
- Width = 0: full mono (S = 0)
- Width = 100%: normal stereo
- Width > 100%: exaggerated stereo (S boosted)
- Most transparent width adjustment method (no timing artefacts)

```
S_gain = width_factor  (1.0 = normal, 0.0 = mono, 2.0 = double width)

L_out = M + S × S_gain
R_out = M − S × S_gain
```

### 4. Unison Detune Stereo
- Multiple synthesizer voices, each slightly detuned (1–10 cents)
- Each voice panned to different position across the stereo field
- Beating between adjacent voices creates chorus-like width
- Classic technique: Roland JP-8000 "Super Saw", Virus supersaw, Serum unison

```
Voice 1: −8 cents, pan = −1.0 (hard L)
Voice 2: −4 cents, pan = −0.5
Voice 3: 0 cents,  pan = 0.0 (center)
Voice 4: +4 cents, pan = +0.5
Voice 5: +8 cents, pan = +1.0 (hard R)
```

### 5. Auto-Pan
- LFO modulates the panning position over time
- Rate: 0.1–4 Hz for musical effect; sync to tempo for rhythmic auto-pan
- Creates width through temporal distribution rather than spatial simultaneity
- Hard auto-pan (L/R switching): tremolo-like feel
- Smooth auto-pan: spacious movement

### 6. Room/Reverb Stereo Returns
- Send mono source to stereo reverb plugin
- Return in stereo — reverb halo appears in stereo field around a centered dry signal
- Effective for locating a sound in space without panning the dry signal

---

## Frequency and Panning Interaction

### Low Frequencies Are Omnidirectional
- Below ~200 Hz, wavelengths are long enough that the head provides no shadow
- ILD is negligible below 200 Hz
- Bass is perceived as coming from "everywhere" rather than a specific direction
- **Implication**: hard-panned bass is still perceived as centered on most systems
- **Rule**: keep bass and sub below 200 Hz in mono (M channel) or equal in L and R

### High Frequencies Are Directional
- Above 1–2 kHz, head shadow provides significant ILD
- Panning of high-frequency content is clearly perceived
- Use high-frequency elements (hi-hats, cymbals, synth highs) for stereo width
- Wide stereo placement of high frequencies creates openness without mono issues

### Mid-Frequency Panning
- 200 Hz – 2 kHz: moderate directional cues via ITD + ILD
- Instruments panned here will fold reasonably in mono
- EQ before panning: some engineers HPF slightly more aggressively on hard-panned signals to improve mono compatibility

---

## Mono Compatibility

### What Breaks Mono Compatibility

1. **Phase-cancelling frequencies**: if L and R are identical but 180° phase-shifted, mono sum is silence
2. **Haas delay content**: comb notches in mono sum
3. **Over-wide M/S processing**: S-channel content exceeding equal power levels
4. **Certain stereo chorus/flanger**: phase relationships that cancel in sum

### Testing Mono Compatibility
- Collapse to mono during mix and check for:
  - Level drops (signal became louder when stereo → mix-bus summing issue)
  - Spectral holes (certain frequencies cancelling)
  - Specific instruments disappearing completely
- Acceptable mono level loss: −3 dB (equal power stereo to mono sum)
- Unacceptable: any instrument disappearing or obvious frequency suck-out

### Fixing Mono Issues
- Check plugin phase: some stereo effects (chorus, pitch shift) can introduce LR phase inversion
- HPF the S channel below 200 Hz
- Reduce stereo width of extreme processors
- Phase alignment: verify no polarity inversion between L and R channels of the same source

---

## Standard Mix Panning Reference

### Drums
| Instrument      | Panning               | Reason |
|-----------------|-----------------------|--------|
| Kick drum       | Center (0°)           | Sub bass: omnidirectional |
| Snare (top)     | Center or slight right| Anchor element |
| Hi-hat (closed) | 30–50% R (drummer POV) or L (audience POV) | Preference |
| Hi-hat (open)   | Match closed          | Consistency |
| Rack tom 1      | 30% L                 | Natural drum kit position |
| Rack tom 2      | 10% L                 | |
| Floor tom       | 40% R                 | |
| Crash L         | Hard L or 70% L       | |
| Crash R         | Hard R or 70% R       | |
| Ride            | 60–80% R              | |
| Room mics       | Wide stereo           | Full room ambience |
| Overheads       | Wide stereo (matched pair) | Natural overhead capture |

### Bass and Low-Frequency
| Instrument | Panning | Reason |
|------------|---------|--------|
| Bass guitar | Center | Sub energy: mono |
| 808 bass   | Center | Sub: mono |
| Sub pad    | Center or narrow | Bass: omnidirectional |

### Harmony and Texture
| Instrument | Panning | Strategy |
|------------|---------|----------|
| Lead synth | Center | Anchor, focal point |
| Backing synth 1 | 50% L | Symmetry |
| Backing synth 2 | 50% R | Symmetry |
| Pad (stereo) | Full width (M/S or chorus) | Fill the field |
| Guitar rhythm 1 | 60–80% L | Classic double-track position |
| Guitar rhythm 2 | 60–80% R | |
| Piano (stereo) | Wide, naturalistically | Natural piano stereo |

### Vocals
| Vocal | Panning | Strategy |
|-------|---------|----------|
| Lead vocal | Center | Primary focal point |
| Backing vocal 1 | 40% L | Frame lead |
| Backing vocal 2 | 40% R | |
| Ad lib 1 | 70% L | Extra width |
| Ad lib 2 | 70% R | |
| Doubled lead | ±30% (or center wide M/S) | Thicken without shifting image |

---

## Spatial Audio Beyond Stereo

### 5.1 Surround
- L, C, R, Ls, Rs + LFE (subwoofer)
- Film standard (Dolby Digital, DTS)
- Center channel anchors dialog to screen
- Surround channels: ambient, effects, immersive elements

### Dolby Atmos
- Object-based audio: sound objects have 3D coordinates rather than fixed channel assignments
- Renderer places objects using speaker array or binaural HRTF rendering
- Up to 128 audio objects + 10 channels of bed material
- Streaming delivery: Dolby Atmos Music (Apple Music, Tidal)
- Headphone delivery: binaural rendering with Apple Spatial Audio

### Ambisonics
- Scene-based audio: encodes a 3D soundfield as spherical harmonic components
- First order (B-format): W, X, Y, Z channels (4 channels)
- Higher order: more channels, higher spatial resolution
- Flexible playback: decode to any speaker array or binaural
- Standard for VR/360 video audio

### Binaural Rendering
Convolve mono signal with HRTF pair (L, R ear measurements) for full 3D spatial audio on headphones.

```python
def apply_hrtf(signal, hrtf_L, hrtf_R):
    """
    signal: mono audio array
    hrtf_L, hrtf_R: head-related impulse responses for L and R ears
    at a specific azimuth/elevation angle
    """
    binaural_L = convolve(signal, hrtf_L)  # FFT convolution
    binaural_R = convolve(signal, hrtf_R)
    return binaural_L, binaural_R
```

For accurate externalisation (3D outside-head feel), HRTF must match the listener's individual ear/head geometry. Generic HRTFs (e.g., KEMAR) work for most listeners but externalisation varies.

---

## Psychoacoustic Width vs True Stereo

### True Stereo
- L and R channels contain genuinely different content (recorded with spaced or coincident stereo microphones)
- Phase and level differences are naturally occurring
- Sounds natural on both speakers and headphones

### Artificial Width
- Width created by processing mono or dual-mono sources
- Common methods: M/S processing, chorus, Haas, unison detune
- Can sound unnatural on headphones (exaggerated inside-the-head width)
- Mono compatibility may be reduced

### Coincident vs Spaced Stereo Microphone Techniques

| Technique | Pair Type | Width | Mono Compatibility |
|-----------|-----------|-------|-------------------|
| XY (coincident) | Cardioid, ±45–90° | Moderate | Excellent |
| ORTF | Cardioid, 17 cm / ±55° | Good | Very good |
| AB (spaced) | Omni, 0.5–2 m | Wide | Fair (phase issues) |
| Blumlein | Figure-8, 90° | Full | Excellent |
| M/S | Cardioid + Figure-8 | Variable | Excellent |

---

## Loudness and Width Perception Interaction

- Wider stereo images often appear louder than narrower ones
- Reason: wider images have lower inter-channel correlation → ear integrates them as more energetic
- When comparing mixes: match LUFS before comparing width
- Over-widened mixes can "fool" engineers into thinking they are louder or fuller than they are
- Check: collapse mix to mono, A/B with and without stereo width processing at matched levels
