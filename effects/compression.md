---
title: "Dynamic Range Compression: Theory and Application"
category: "Effects"
tags: [compression, dynamics, attack, release, ratio, threshold, sidechain, limiter]
---

# Dynamic Range Compression: Theory and Application

Dynamic range compression reduces the dynamic range of an audio signal by attenuating levels above a threshold. It is one of the most fundamental and widely-used tools in mixing, mastering, and sound design.

---

## Signal Flow: Gain Computer Block Diagram

```
Input → Level Detector → Gain Computer → Gain Smoother (ballistics) → VCA → Output
                                                                          ↑
                                                               Makeup Gain added here
```

1. **Level Detector** — measures input level (peak or RMS)
2. **Gain Computer** — computes required gain reduction based on threshold, ratio, and knee
3. **Gain Smoother** — applies attack and release time constants to the gain reduction control signal
4. **VCA (Voltage Controlled Amplifier)** — applies the smoothed gain to the audio signal

---

## Core Formula

When `input_dB > threshold`:

```
gain_reduction_dB = (1 - 1/ratio) * (threshold - input_dB)
```

This is negative (attenuation). The output level is:

```
output_dB = input_dB + gain_reduction_dB
           = input_dB + (1 - 1/ratio) * (threshold - input_dB)
```

For a 4:1 ratio with threshold at -10 dBFS and input at -6 dBFS:
```
gain_reduction = (1 - 1/4) * (-10 - (-6))
               = 0.75 * (-4)
               = -3 dB
output = -6 + (-3) = -9 dBFS
```

---

## Level Detection: Peak vs RMS

### Peak Detection
- Captures instantaneous peak values of the waveform
- Responds to transients immediately (within one sample in ideal implementation)
- More accurate for brick-wall limiting and transient control
- Can over-compress steady signals with high crest factor

### RMS Detection
- Averages the squared signal over a time window (typically 10–300ms)
- Reflects perceived loudness more closely than peak
- Smoother gain reduction, less pumping artefacts on complex material
- Standard window: `rms = sqrt( (1/N) * sum(x[n]^2) )`
- RMS compression sounds more "musical" on program material

### Practical Choice
- Use **peak** for limiting, transient control, brick-wall ceilings
- Use **RMS** for glue compression, bus processing, general dynamics control

---

## Attack and Release

### Attack Time
- Time constant for gain reduction to reach ~63% of its final value (1 − 1/e ≈ 63%)
- Some definitions: time to reach −10 dB of total attenuation
- Short attack (0.1–3ms): catches transients, reduces punch/click
- Long attack (10–50ms): lets transient through before clamping — preserves punch
- Attack affects the onset of compression

### Release Time
- Time constant for gain to recover after signal drops below threshold
- Short release (<50ms): gain recovers quickly, can cause pumping on rhythmic material
- Long release (200ms–1s): smoother, more transparent, can cause gain riding artefacts
- Release affects the feel and groove of compressed material

### Ballistics: Program-Dependent Release
- Adaptive release that tracks the signal envelope
- Fast short-duration peaks get fast release; sustained loud passages get slow release
- Prevents pumping artifacts from short transients triggering long release tails
- Implemented in hardware (LA-2A opto) and modern plugins
- Algorithm: release time constant shortens when signal is consistently above threshold

---

## Ratio Settings and Their Uses

| Ratio   | Character              | Typical Use Case                          |
|---------|------------------------|-------------------------------------------|
| 1.5:1   | Gentle glue            | Bus compression, subtle dynamic control   |
| 2:1     | Subtle dynamics        | Drums room mic, mix bus, 2–3 dB GR        |
| 3:1     | Moderate               | Vocals, bass guitar, acoustic instruments |
| 4:1     | Moderate-heavy         | Kick drum, snare, parallel compression    |
| 6:1     | Heavy                  | Attack shaping, aggressive vocals         |
| 8:1     | Hard compression       | Parallel processing, drum bus             |
| 10:1    | Near-limiting          | Broadcast, DJ limiters                    |
| 20:1+   | Limiting               | Peak limiting, master bus ceiling         |
| ∞:1     | Hard limit/brickwall   | Brick-wall limiter, clipping prevention   |

---

## Knee: Hard vs Soft

### Hard Knee
- Gain reduction begins instantly when signal crosses threshold
- Sharp transition at threshold point
- More aggressive, audible onset
- Better for transparent limiting at high ratios

### Soft Knee
- Gain reduction begins gradually ±(knee_width/2) dB around the threshold
- Within the knee region: `GR = (1 - 1/ratio) * (input - threshold + knee/2)² / (2 * knee)`
- Typical knee width: 3–6 dB (some compressors 0–10 dB variable)
- More transparent, harder to hear onset of compression
- Preferred for musical material, vocals, bus compression

---

## Makeup Gain

- Compensates for the overall level reduction caused by compression
- Typical formula: `makeup_gain_dB ≈ |average_gain_reduction_dB|`
- Auto makeup gain: `makeup = (threshold - threshold/ratio)` — crude but useful estimate
- Should be set by ear after compression is dialled in
- Important: makeup gain raises the level of the compressed signal including noise floor

---

## Parallel Compression (New York Compression)

Blend dry (uncompressed) signal with a heavily compressed version.

```
output = (1 - blend) * dry_signal + blend * compressed_signal
```

### Settings for NY Compression:
- Ratio: 4:1 to 8:1 (heavily squashed)
- Threshold: −20 to −30 dBFS (compressor working hard almost constantly)
- Attack: 1–5ms (fast, catches everything)
- Release: 50–150ms
- Blend: 20–50% wet
- No makeup gain on compressed bus (the parallel blend handles apparent level)

### Why It Works
- Dry signal preserves transients and dynamics
- Compressed signal fills in sustain and body
- Result: punchy transients with enhanced sustain — best of both worlds
- Excellent for drums, bass, and mix bus

---

## Sidechain Compression

The gain computer responds to an **external** or **filtered** signal rather than the direct input.

### External Sidechain
- Use: kick drum sidechains the bass — bass ducks every time kick hits
- Creates the classic EDM "pumping" effect
- Setup: route kick to sidechain input of bass compressor
- Ratio: 4:1–8:1, attack 1–5ms, release 50–200ms tuned to kick transient decay

### Internal Sidechain with High-Pass Filter
- HPF on sidechain removes low frequencies from detection signal
- Prevents kick drum or sub bass from triggering compressor
- Standard on all pro bus compressors (SSL G-Bus style)
- HPF typically at 100–150Hz

### De-essing (Frequency-Selective Sidechain)
- Band-pass or high-pass filter on sidechain targeting sibilant frequencies (5–10kHz)
- Compressor only triggers when sibilance is present
- Ratio: 3:1–6:1, attack <1ms, release 50–100ms

---

## Multiband Compression

Split audio signal into frequency bands, compress each independently.

### Common Band Splits:
- Sub/Low: below 250 Hz
- Low-mids: 250 Hz – 2 kHz
- High-mids: 2 kHz – 8 kHz
- Air: above 8 kHz

### Implementation:
1. Split signal with crossover filters (Linkwitz-Riley 4th order recommended — zero phase shift at crossovers)
2. Apply independent compression to each band
3. Sum bands back to stereo output

### Use Cases:
- Tame boomy low-mids without affecting highs
- Control harsh 2–5kHz without pumping the whole signal
- Mastering: gentle broadband control without coloration
- Drum bus: independently tighten low end (kick/bass interaction), control mid harshness

---

## Use Case Settings Reference

### Kick Drum
- Ratio: 4:1
- Threshold: −6 dBFS
- Attack: 1ms (shape transient, or 5ms to preserve click)
- Release: 50ms
- Goal: control sustain/body, preserve or accentuate initial transient

### Snare
- Ratio: 3:1
- Threshold: −8 dBFS
- Attack: 5ms (let crack through)
- Release: 100ms
- Goal: control snare body and room, maintain crack

### Bass Guitar / 808
- Ratio: 3:1
- Threshold: −12 dBFS
- Attack: 8ms (preserve initial pluck/transient)
- Release: 150ms
- Goal: consistent level across notes, reduce dynamic range between played notes

### Vocals
- Ratio: 2.5:1
- Threshold: −18 dBFS
- Attack: 3ms
- Release: 200ms (program-dependent preferred)
- Goal: even out performance dynamics, 3–6 dB GR average
- Often use two stages: gentle utility compressor + character compressor

### Mix Bus
- Ratio: 2:1
- Threshold: −6 dBFS (2–3 dB GR at peaks)
- Attack: 30ms (let transients pass, preserve punch)
- Release: 250ms (or auto/program)
- Makeup: +2–3 dB
- Goal: glue elements together, minor density increase, not heavy-handed

---

## Compressor Topology Characteristics

### VCA (Voltage Controlled Amplifier)
- Examples: SSL G-Bus, dbx 160, API 2500
- Fast, accurate, clean
- Precise attack and release times
- Surgical or punchy depending on settings
- Most common in modern hardware and software

### Optical (Opto)
- Examples: LA-2A, LA-3A, Tube-Tech CL1B
- Photoresistor responds to light from LED/lamp
- Inherently program-dependent — opto response is non-linear
- Natural, smooth compression with musical release
- Slower attack makes it ideal for vocals, bus, acoustic guitars
- Characteristic "hold" before release begins

### FET (Field Effect Transistor)
- Examples: Urei 1176, Empirical Labs Distressor
- Very fast attack (20µs on 1176)
- Aggressive, punchy character
- "All buttons in" mode on 1176 → distorted, heavily compressed, 12:1 parallel blend
- Excellent for drums, aggressive vocals

### Tube / Variable-Mu
- Examples: Fairchild 670, Vari-Mu
- Slowest attack of all topologies
- Very musical, harmonic saturation from tubes
- Self-adjusting ratio based on gain reduction amount
- Mastering favourite for program material

---

## Metering

### Peak Meters
- Show instantaneous peak level (dBFS)
- Fast ballistics (typically 0ms attack, 300ms–∞ hold)
- Essential for preventing clipping

### RMS Meters
- Show average power level
- Slower ballistics (300ms average window typical)
- Better indication of perceived loudness

### LUFS (Loudness Units relative to Full Scale)
- ITU-R BS.1770 standard
- Integrates loudness over time using K-weighting filter (perceptual weighting)
- Streaming targets: Spotify −14 LUFS, Apple Music −16 LUFS, YouTube −14 LUFS
- Short-term LUFS: 3-second window; Integrated LUFS: entire program

### Gain Reduction Meters
- Show how much compression is being applied in real time
- Read in negative dB (e.g., −6 dB GR = 6 dB of compression happening)
- Target: 2–6 dB GR average for transparent compression, 8–20 dB for heavy effects

---

## Limiting vs Compression

| Feature       | Compressor         | Limiter              |
|---------------|--------------------|----------------------|
| Ratio         | 2:1 – 20:1         | 20:1 to ∞:1          |
| Attack        | 0.1ms – 100ms      | <1ms, often 0.1ms    |
| Release        | 50ms – 1s+         | 10ms – 500ms         |
| Lookahead     | Rarely             | Often 1–5ms          |
| Gain reduction| 2–12 dB typical    | Any, hard ceiling     |
| Knee          | Hard or soft       | Hard                 |

### True Peak Limiting
- Standard peak meters can miss inter-sample peaks (ISPs) during D/A conversion
- True peak measurement oversamples by 4x to detect ISPs
- Maximum true peak for streaming: −1 dBTP
- Necessary for broadcast compliance (EBU R128)

---

## Practical DSP Implementation

### Gain Computer (digital)

```python
def gain_computer(input_dB, threshold, ratio, knee_width):
    x = input_dB - threshold
    half_knee = knee_width / 2.0
    if x < -half_knee:
        return input_dB  # no compression
    elif x > half_knee:
        return threshold + x / ratio  # full compression
    else:
        # soft knee zone
        return input_dB + (1/ratio - 1) * (x + half_knee)**2 / (2 * knee_width)
```

### Attack / Release Smoother

```python
def smooth_gain(gain_reduction, prev_gain, attack_coeff, release_coeff):
    if gain_reduction < prev_gain:  # gain going down (more compression)
        coeff = attack_coeff
    else:                            # gain going up (releasing)
        coeff = release_coeff
    return coeff * prev_gain + (1 - coeff) * gain_reduction

# Coefficient from time constant:
# coeff = exp(-1 / (time_sec * sample_rate))
# e.g., 10ms attack at 44100 Hz: coeff = exp(-1/(0.01 * 44100)) ≈ 0.9977
```

### Peak Level Detector

```python
def peak_detect(x, prev_peak, attack_coeff, release_coeff):
    abs_x = abs(x)
    if abs_x > prev_peak:
        return attack_coeff * prev_peak + (1 - attack_coeff) * abs_x
    else:
        return release_coeff * prev_peak
```

### RMS Level Detector

```python
# Recursive RMS with time constant T (seconds)
# coeff = exp(-2.2 / (T * sample_rate))
def rms_detect(x, prev_rms_sq, coeff):
    rms_sq = coeff * prev_rms_sq + (1 - coeff) * x * x
    return math.sqrt(rms_sq)
```

---

## Common Mistakes

- **Too-fast attack on mix bus**: kills transients, sounds flat and lifeless
- **Too-slow release on kick/snare**: gain doesn't recover in time for next hit, sounds pumping
- **Over-compression**: >10 dB GR on bus compression destroys dynamics and fatigues listeners
- **Comparing with makeup gain**: always match levels before A/B comparing — louder sounds better
- **Sidechain without HPF**: sub bass triggers heavy GR on every bass note on mix bus
