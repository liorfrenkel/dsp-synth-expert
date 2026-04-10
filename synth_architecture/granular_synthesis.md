---
title: "Granular Synthesis"
category: "Synth Architecture"
tags: [granular, grain, cloud, time-stretch, pitch-shift, Clouds, texture]
sources:
  - https://en.wikipedia.org/wiki/Granular_synthesis
  - Mutable Instruments Clouds manual
  - Roads, Curtis. "Microsound" (MIT Press, 2001)
  - Truax, Barry. "Handbook for Acoustic Ecology" (1978)
---

## Overview

Granular synthesis generates sound from many tiny audio fragments ("grains") that are layered, scattered, and shaped to build complex textures. Each grain is a short burst of audio (1ms–500ms), windowed to avoid clicks, with independent pitch, timing, and position in a source audio buffer.

**Theoretical basis**: Dennis Gabor's quantum acoustics (1947) — all sounds can be described as a collection of time-frequency atoms ("acoustical quanta")  
**First computer implementation**: Curtis Roads (1974)  
**First real-time implementation**: Barry Truax, DMX-1000 Signal Processing Computer (1986)  
**Composer credited with invention**: Iannis Xenakis (expanded Gabor's theory into musical practice)

---

## Grain Parameters

### Core Parameters

| Parameter | Range | Description |
|-----------|-------|-------------|
| Grain Size | 1ms – 500ms | Duration of each grain |
| Density | 1 – 200 grains/sec | How many grains are playing simultaneously |
| Position | 0 – 100% | Playhead location in source audio buffer |
| Pitch | −24 to +24 semitones | Transposition per grain (or in cents: −2400 to +2400) |
| Amplitude | 0 – 100% | Per-grain level |
| Pan | −100% (L) to +100% (R) | Stereo placement of each grain |

### Scatter/Randomization Parameters

| Parameter | Range | Description |
|-----------|-------|-------------|
| Position Scatter | 0 – 100% | Random offset around position point |
| Pitch Scatter | 0 – 100 cents | Random detune per grain |
| Size Scatter | 0 – 100% | Random variation in grain duration |
| Pan Scatter | 0 – 100% | Random stereo spread per grain |
| Amplitude Scatter | 0 – 100% | Random level variation per grain |
| Timing Scatter | 0 – 100% | Jitter in grain trigger times (deviation from regular grid) |

**Low scatter** (0–20%): predictable, tonal, focused  
**Medium scatter** (30–60%): slight randomness, organic, lush  
**High scatter** (70–100%): washy, diffuse, cloud-like texture

---

## Grain Envelope Shapes

Each grain is multiplied by an envelope window to prevent clicks at the grain boundaries.

### Rectangular Window
```
Envelope: [1.0, 1.0, 1.0, ..., 1.0]
```
- No fade in/out
- Hard click at start and end of grain
- Suitable only for very short grains (<5ms) or intentional glitch effects
- Maximum spectral leakage

### Trapezoidal (Attack-Sustain-Release)
```
Envelope: /‾‾‾‾‾‾‾\
          A  S   R
```
- Attack: linear rise over first A% of grain
- Sustain: flat at 1.0 for middle S% 
- Release: linear fall over last R% of grain
- Parameters: attack % (5–20%), release % (5–20%)
- Common default for granular instruments

### Hann Window
```
Envelope: w(n) = 0.5 * (1 - cos(2π·n / (N-1)))
```
- Bell-shaped: rises and falls smoothly
- Zero at both endpoints (no click)
- Smooth spectral characteristics
- Good general-purpose grain window
- Half-Hann: only the rising or falling half used

### Gaussian Window
```
Envelope: w(n) = e^(−0.5 · ((n - N/2) / (σ · N/2))²)
```
- Centered bell curve, parameter σ controls width
- Smoothest of all windows for granular
- Mathematically minimizes time-frequency uncertainty (satisfies Gabor's conditions)
- Slightly more CPU intensive than Hann

### Tukey Window
```
Attack taper + flat sustain + release taper
Parameter α (0=rectangular, 1=Hann)
```

### Which Window to Use?
| Grain Size | Recommended Window |
|------------|-------------------|
| < 5ms | Rectangular or Hann |
| 5–20ms | Hann or Gaussian |
| 20–100ms | Trapezoidal (medium A/R) |
| 100ms–500ms | Trapezoidal (short A/R) or Hann |

---

## Time Stretching via Granular

Time stretch: change the duration of audio without changing pitch.

**Algorithm**:
```
source_playhead_advance = 1 / stretch_factor

At each grain trigger time:
    grain_position = source_playhead_position
    grain_pitch = 1.0 (no pitch change)
    source_playhead += grain_size * source_playhead_advance
    
if stretch_factor = 1.0:  playhead advances normally
if stretch_factor = 0.5:  playhead advances at half speed → 2× duration
if stretch_factor = 2.0:  playhead advances at double speed → 0.5× duration
```

**Overlap and cross-fading**:
- Grain density must be high enough that grains overlap
- Adjacent grains fade out as next grain fades in → smooth seamless playback
- Rule of thumb: grain_density × grain_size ≥ 2 for seamless coverage

**Artefacts**:
- Low density at slow stretch: pitch wobble (grains don't perfectly overlap)
- Phase incoherence: "watery" or "metallic" sound when grain phases don't align
- Solution: phase vocoder (analyzes and corrects phase between grains)

---

## Pitch Shifting via Granular

Pitch shift: change pitch without changing duration.

```
grain_playback_rate = pitch_shift_ratio = 2^(semitones/12)
source_playhead_advance = pitch_shift_ratio
grain_trigger_interval = grain_size (unchanged)
```

Playhead advances at same rate as grain duration advances in source → output duration unchanged.

**Combined pitch+time manipulation**:
```
# Independent control:
playhead_advance_rate = stretch_factor       # controls output duration
grain_playback_rate   = pitch_shift_ratio    # controls output pitch

# Both can be set independently because they are separate parameters
```

This is the fundamental advantage of granular over simpler pitch-shifting methods: pitch and time are fully decoupled.

---

## Mutable Instruments Clouds (Eurorack Module)

Clouds (2014, Olivier Gillet) became the iconic granular Eurorack module. Discontinued 2017, open-sourced.

### Panel Controls and CV Inputs

| Knob/CV | Function | Range |
|---------|----------|-------|
| Position | Playhead in buffer (0=start, 1=end) | 0–100% |
| Size | Grain duration | 1ms – 500ms |
| Pitch | Grain transposition | −2 octaves to +2 octaves |
| Density | Number of simultaneous grains | Sparse to continuous |
| Texture | Grain envelope shape | Smooth (Hann) to rough (near-rectangular) |
| Blend | Wet/Dry mix | 0–100% wet |
| Freeze | Halt buffer recording (loop current buffer) | Gate/Toggle |

**Internal reverb**: Blend knob alternately controls: Dry/Wet, Stereo spread, Feedback amount, Reverb amount (shifted mode)

### Clouds Modes (Hidden via Button Hold)
1. **Granular mode** (default): standard granular playback
2. **Pitch-shifting/Time-stretching**: phase vocoder granular
3. **Looping delay**: granular feedback delay
4. **Spectral mode**: FFT-based texture blending

### Freeze Function
- Stops the input audio buffer from recording new audio
- All grains loop over the frozen buffer content
- Effect: infinite sustain pad from any input audio
- CV-controllable: gate input pauses/resumes recording

### Position and Spray
- **Position**: sets the center of grain reading in the buffer
- **Spray** (Position CV): adds randomness around the center
  - Low spray: narrow grain window (focused, tonal)
  - High spray: wide random distribution (diffuse, washy texture)

### Texture Parameter Detail
- At 0: smooth Hann-windowed grains (smooth, blurred)
- At 0.5: grains with mild rectangular component
- At 1.0: very short, harsh grains with minimal windowing → grainy, crispy texture
- Also affects grain overlap behavior

---

## Granular Techniques

### Frozen Pad
```
1. Play any note or chord (or input audio)
2. Hit Freeze while buffer is full
3. Buffer loops indefinitely
4. Modulate Position slowly → morphing pad
5. Modulate Pitch → harmonize/transpose
```

### Spray Texture / Ambient Cloud
```
Settings:
  Size: 80–200ms
  Density: high (50–100 grains/sec)
  Position Scatter: 60–90%
  Pitch Scatter: 0–20 cents
  Envelope: Hann
Result: diffuse, continuous wash (similar to reverb but based on actual audio content)
```

### Granular Chopping / Glitch
```
Settings:
  Size: 5–30ms
  Density: medium (10–30 grains/sec)
  Timing Scatter: 40–80%
  Pitch: random or stepped
Result: rhythmically stuttering, glitchy texture
```

### Time-Stretch Drone
```
Settings:
  Stretch factor: 8–16× (playhead moves very slowly)
  Grain size: 80–200ms
  Density: high
  Position Scatter: low (5–10%)
Result: extremely slow, blurred version of original recording; bass notes become sub-frequency drones
```

### Pitch-Harmonized Cloud
```
Settings:
  Multiple grain voices: pitch offset +0, +7, +12, +19 semitones
  Position Scatter: 20–40%
  Size: 100–300ms
Result: harmonized layers of the source audio, all from same time position
```

---

## Corpus: Resonator per Grain

Combining granular with physical modeling: each grain excites a resonator (modal synthesis object).

**Architecture:**
```
Grain → Resonator filter (biquad or waveguide tuned to grain pitch)
```

Each grain is shaped by its own resonant body model, adding tonal character to otherwise noise-like grains. Used in:
- Mutable Instruments **Rings** (resonator bank)  
- Combining Clouds output → Rings input in a Eurorack patch
- Max/MSP patch: `grain~` + `bpf~` per voice

---

## Software Implementations

### Max/MSP
```
[granular~]: grain cloud from buffer~
  Arguments: buffer_name grain_size scatter density
[groove~]: looping playback, variable rate
[poly~]: multi-voice granular (grain per voice in sub-patcher)
```

### SuperCollider
```supercollider
// TGrains - key SuperCollider granular UGen
TGrains.ar(
    numChannels: 2,
    trigger: Impulse.ar(10),      // 10 grains/sec
    bufnum: b.bufnum,
    rate: 1.0,                     // playback rate (pitch)
    centerPos: 0.5,                // position in buffer (0–1)
    dur: 0.1,                      // grain duration in seconds
    pan: 0.0,                      // stereo position
    amp: 0.5,                      // grain amplitude
    interp: 4                      // interpolation (1=no, 2=linear, 4=cubic)
)
```

### Ableton Live
- **Sampler**: Granular mode in Oscillator section
  - Grain size: 1–300ms
  - Flux: randomization
  - Transients: preserves or ignores transients during stretch
- **Simpler** with Complex or Complex Pro warp mode

### Serum (Xfer Records)
- Noise oscillator with "Grain" mode (single-cycle grain scanning)
- Wavetable morph mode can simulate granular scanning

### Bitwig Polysynth / Phase-4
- Built-in granular oscillator module
- Position, size, spray, speed parameters

---

## Advanced: Phase Vocoder Granular

Standard granular has phase incoherence between grains. Phase vocoder analysis corrects this:

```
1. STFT analysis: extract magnitude and phase per bin per frame
2. Track phase delta between frames: Δφ = φ(t+1) - φ(t) - expected_increment
3. For time-stretch: scale time axis, maintain phase coherence
4. Resynthesize: ISTFT with corrected phases
```

**Result**: much cleaner time-stretching without the "watery" granular artefact.  
**CPU cost**: higher than naive granular (full FFT analysis per block)  
**Used in**: Ableton's Complex Pro warp mode, zplane élastique, Celemony Melodyne

---

## Grain Cloud Mathematical Model

```
output(t) = Σ[i=1..N] aᵢ(t) · w(t - tᵢ, dᵢ) · s(rᵢ · (t - tᵢ) + pᵢ)

Where:
  N     = number of active grains
  aᵢ   = amplitude of grain i
  w(t, d) = window function (duration d, centered at 0)
  s(·)  = source audio signal
  rᵢ   = playback rate of grain i (= pitch shift ratio)
  tᵢ   = start time of grain i  
  pᵢ   = position in source buffer for grain i
```

**Density and overlap:**
```
Average simultaneous grains = density × grain_size
  (density in grains/sec, grain_size in seconds)

Example: 20 grains/sec × 100ms = 2 average simultaneous grains
         50 grains/sec × 200ms = 10 average simultaneous grains
```

**High density (>50 simultaneous grains)**: quasi-continuous sound, strong beating/chorus  
**Low density (1–5 simultaneous grains)**: individual grains audible as separate events

---

## Psychoacoustic Considerations

- **Grains < 10ms**: perceptually no longer heard as separate events, but their repetition rate creates a pitch-like sensation if regular
- **Grains 10–40ms**: transition zone; may produce clicks or roughness
- **Grains > 50ms**: heard as short audio snippets; pitch and timbre more distinct
- **Grain density and masking**: at high density, early grains mask later ones (temporal masking)
- **Haas effect**: grains within 40ms of each other fuse perceptually; beyond 40ms → echo
- **Pitch from repetition rate**: regular grain triggering at rate R Hz produces a perceived pitch at R Hz (and overtones) when R is in the 20–5000 Hz range

---

## Historical Context and Key Figures

| Year | Event |
|------|-------|
| 1947 | Dennis Gabor: quantum acoustics theory |
| 1952 | Iannis Xenakis begins exploring granular composition concepts |
| 1974 | Curtis Roads: first computer granular synthesis |
| 1978 | Roads publishes first academic description of granular synthesis |
| 1986 | Barry Truax: first real-time implementation on DMX-1000 |
| 1988 | Truax: real-time granular on DSP-56001 |
| 1991 | Trevor Wishart: "On Sonic Art" — musical applications |
| 1996 | Granular synthesis in commercial plugins begins |
| 2001 | Curtis Roads: "Microsound" (MIT Press) — definitive textbook |
| 2014 | Mutable Instruments Clouds: democratizes granular in Eurorack |
| 2017 | Clouds discontinued; firmware opens to community (Parasite, Supercell, etc.) |
