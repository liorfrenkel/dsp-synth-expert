---
title: "Reverb Algorithms: From Schroeder to Convolution"
category: "Effects"
tags: [reverb, Schroeder, Freeverb, convolution, IR, plate, spring, hall, room]
---

# Reverb Algorithms: From Schroeder to Convolution

Reverberation simulates the acoustic reflections of a physical space. Understanding the underlying algorithms enables precise control of reverb character, size, density, and decay.

---

## Physical Reverb: What We Are Modelling

When a sound is produced in a room:

1. **Direct sound** arrives first (0 ms delay)
2. **Early reflections** (0–80 ms): sparse, discrete echoes from walls, floor, ceiling; convey room geometry and size
3. **Late reverberation** (>80 ms): dense, diffuse, exponentially decaying noise; modal character of the space emerges as echo density increases beyond ~1000 echoes/second

The perceptual parameters that characterise a reverb:
- **RT60**: time for reverb to decay by 60 dB (a key measure of room "size")
- **EDT (Early Decay Time)**: time for first 10 dB of decay × 6 — heard as "initial decay" — most perceptible measure
- **Clarity C80**: ratio of energy arriving before 80 ms vs after 80 ms (dB scale)
- **Definition D50**: ratio of energy in first 50 ms to total energy
- **IACC (Interaural Cross-Correlation)**: measure of spaciousness; higher IACC = more centred; lower = more diffuse

---

## Early Reflections vs Late Reverberation

### Early Reflections (0–80 ms)
- Sparse, discrete reflections — audible as individual echoes at low density
- Convey size and geometry of space (arrival direction, delay times)
- Help localisation: early reflections carry information about source position
- In algorithms: simulated with a tapped delay line (TDL), typically 8–20 taps
- Level typically: −10 to −20 dBFS relative to direct

### Late Reverberation (>80 ms)
- Dense, diffuse, statistically decorrelated noise
- Echo density: >1000 echoes/second for smooth reverb tail
- Exponentially decays: `level(t) = level(0) × 10^(-60t / RT60)`
- Should be spectrally shaped (HF decays faster due to air absorption)
- Largely determines perceived "roominess" and reverb character

### Separation in Algorithms
- Some algorithms treat early reflections and late reverb as separate modules
- Moorer (1979) added a tapped delay line for early reflections before the Schroeder network
- Pre-delay parameter artificially adds gap between direct sound and first reflection

---

## Delay Line Primitives

### Comb Filter (Feedback Delay Line, FDL)
Recursive delay with feedback coefficient `g`:

```
y[n] = x[n] + g * y[n - M]
```

Where `M` = delay length in samples, `g` = feedback gain (0 < g < 1).

- Transfer function: `H(z) = 1 / (1 - g * z^(-M))`
- Resonates at frequencies: `f_k = k * fs / M` for k = 0, 1, 2, ...
- RT60 of a comb filter: `RT60 = -3 * M / (fs * log10(g))` seconds
- Loop gain for desired RT60: `g = 10^(-3 * M / (fs * RT60))`

### Allpass Filter
```
y[n] = -g * x[n] + x[n-M] + g * y[n-M]
```

Transfer function: `H(z) = (-g + z^(-M)) / (1 - g * z^(-M))`

- Flat frequency response (`|H(f)| = 1` for all f)
- Non-flat phase response — causes dispersion
- Used to increase echo density without adding resonant coloration
- Two common forms: Schroeder allpass (above) and nested allpass

---

## Schroeder Reverberator (1962)

The first successful algorithmic reverberator, published by Manfred Schroeder at Bell Labs.

### Topology
```
                  ┌────────────── Comb 1 (M=1687, g=g1) ──────────────┐
                  │                                                       │
Input ──────────── Comb 2 (M=1601, g=g2) ───────────── Sum ─── AP1 ─── AP2 ─── Output
                  │                                       │
                  ├────────────── Comb 3 (M=2053, g=g3) ─┤
                  │                                       │
                  └────────────── Comb 4 (M=2251, g=g4) ─┘
```

4 parallel comb filters feeding into 2 series allpass filters.

### Delay Times (at 44100 Hz sample rate)

| Filter | Delay (samples) | Delay (ms) |
|--------|-----------------|------------|
| Comb 1 | 1687            | 38.25      |
| Comb 2 | 1601            | 36.30      |
| Comb 3 | 2053            | 46.55      |
| Comb 4 | 2251            | 51.04      |
| AP 1   | 225             | 5.10       |
| AP 2   | 556             | 12.61      |

### Feedback Coefficient
- `g = 0.84` for comb filters (controls RT60)
- `g_ap = 0.7` for allpass filters (standard Schroeder value)
- RT60 at g=0.84 for M=1687: `RT60 = -3 * 1687 / (44100 * log10(0.84)) ≈ 1.0 s`

### Problems with Schroeder Networks
- Audible "flutter" and metallic coloration from evenly-spaced resonances
- Insufficient echo density for smooth reverb tail
- Delay times need to be mutually prime to avoid common resonance periodicity
- Requirement: delay lengths should not share common factors

---

## Moorer Reverb (1979)

James A. Moorer extended Schroeder with:
1. **Tapped delay line** for early reflections (before the comb/allpass network)
2. **Low-pass filter inside each comb loop** for frequency-dependent damping

### Modified Comb Filter with Damping

```
y[n] = x[n] + g_c * lpf( y[n - M] )
```

Where `lpf` is a simple 1-pole IIR lowpass:

```
lpf_out[n] = (1 - d) * y[n-M] + d * lpf_out[n-1]
```

`d` = damping coefficient (0 = no damping, 0.9 = heavy HF damping)

- HF content decays faster → simulates air absorption
- Perceptually much more natural than flat comb filters

### Moorer Network Topology
```
Input ──→ Tap Delay Line (early reflections) ──→ Sum ──→ 6 Parallel Combs (w/ LPF) ──→ AP Diffuser ──→ Output
                                                   ↑
                                            direct signal
```

---

## Freeverb (Jezar at Dreampoint, 2000)

Free, open-source Schroeder-Moorer type reverb. Widely studied and ported.

### Architecture
- **8 parallel comb filters** (with LPF in feedback loop)
- **4 series allpass diffusers** after the comb network
- **Stereo**: two complete networks, right channel delay times offset by a fixed amount (23 samples)

### Freeverb Comb Delay Times (left channel, at 44100 Hz):

```
1116, 1188, 1277, 1356, 1422, 1491, 1557, 1617 samples
```

Right channel: each + 23 samples

### Freeverb Allpass Delay Times (left channel):

```
556, 441, 341, 225 samples
```

Right channel: each + 23 samples

### Freeverb Parameters

| Parameter | Range | Description |
|-----------|-------|-------------|
| roomSize  | 0–1   | Maps to comb feedback gain (g = roomSize * 0.28 + 0.7) |
| damping   | 0–1   | HF rolloff inside comb loops |
| wet       | 0–1   | Reverb output level |
| dry       | 0–1   | Dry signal level |
| width     | 0–1   | Stereo width (mix of L/R network outputs) |

### Freeverb Stereo Width Decoding

```python
# width=1: full stereo separation; width=0: mono
out_L = wet_L * (width/2 + 0.5) + wet_R * (0.5 - width/2)
out_R = wet_R * (width/2 + 0.5) + wet_L * (0.5 - width/2)
```

### Freeverb Code Sketch (comb filter with damping)

```python
class CombFilter:
    def __init__(self, delay_samples, roomsize, damping):
        self.buf = [0.0] * delay_samples
        self.idx = 0
        self.feedback = roomsize
        self.damp1 = damping
        self.damp2 = 1.0 - damping
        self.filterstore = 0.0

    def process(self, x):
        output = self.buf[self.idx]
        self.filterstore = output * self.damp2 + self.filterstore * self.damp1
        self.buf[self.idx] = x + self.filterstore * self.feedback
        self.idx = (self.idx + 1) % len(self.buf)
        return output
```

---

## Plate Reverb Simulation

Physical plate reverb (EMT 140) uses a thin steel plate excited by a transducer; pickups at other positions capture the reverberant field.

### Perceptual Character
- Very dense early reflections (no distinct echoes — plate has no "geometry")
- Bright, smooth reverb tail — classic for snare and vocals
- No low-frequency modal resonances typical of rooms
- Relatively uniform decay across frequencies

### Algorithmic Approximation
- Dense FDN (Feedback Delay Network) with many closely-spaced taps
- No early reflection model needed (reflections begin immediately)
- Less HF damping than room simulation
- Use: snare drums, vocals, solo instruments — "studio" quality reverb

---

## Spring Reverb

Physical spring reverb (Accutronics tanks) passes audio through a coiled spring; vibrations travel along the spring and reflect from the end.

### Perceptual Character
- Characteristic "boing" — dispersive delay: low frequencies travel faster than high
- This dispersion causes the reverb to "stretch" in a frequency-dependent way
- Multiple spring tanks sum outputs for density
- Pronounced metallic resonance at spring fundamental frequencies

### Dispersion Simulation
Model using an allpass chain where allpass delay is frequency-dependent:

```
H_dispersion(z) = (z^(-N1) * prod_k AP_k(z))
```

Where allpass filters have phase responses approximating the spring's physical dispersion curve. A practical simulation uses a cascade of 10–20 allpass filters.

### Parameters
- Diffusion/drip: density of the boing characteristic
- Decay: length of tail
- Tone: HF rolloff in the feedback path

---

## Feedback Delay Networks (FDN)

A generalisation of Schroeder reverb: N delay lines with feedback through an N×N matrix.

```
state = M * diag(z^(-D1), ..., z^(-DN)) * state + input
output = C * state
```

Where:
- `M` = feedback matrix (must be lossless unitary or near-unitary for stable reverb)
- `D_k` = delay lengths (prime to each other for maximum diffusion)
- `C` = output mixing matrix

### Hadamard Matrix (common choice for M)

For N=4:
```
     1  [ 1  1  1  1 ]
M = --- [ 1 -1  1 -1 ]
    2   [ 1  1 -1 -1 ]
         [ 1 -1 -1  1 ]
```

Householder matrices or random orthogonal matrices also used.

### Why FDN is Preferred
- Fully controllable RT60 per frequency band
- Easy to extend to many channels (8, 16, 32 delay lines for dense reverb)
- Can model anisotropic spaces (different decay per direction)
- Basis of most modern algorithmic reverb engines (Lexicon, TC Electronic, Valhalla)

---

## Convolution Reverb

### Concept
Capture the impulse response (IR) of a real space → convolve it with audio.

```
output[n] = sum_k ( input[k] * ir[n-k] )   # discrete convolution
```

The IR represents the complete acoustic response of the space to an ideal impulse.

### IR Measurement Methods
1. **Sine sweep** (exponential swept sine, ESS): most common; deconvolve the swept sine with the recorded response
2. **MLS (Maximum Length Sequence)**: pseudorandom sequence with flat spectrum; deconvolve
3. **Starter pistol / balloon burst**: impulsive; direct, but lower SNR

### Computational Cost
- Naïve convolution: O(N²) per sample — impractical for long IRs (>0.1s)
- FFT convolution: O(N log N) per block
- **Overlap-add** / **overlap-save**: block-based FFT, low latency possible
- **Partitioned convolution**: split IR into short blocks for low latency + long IR support
  - Head blocks (first ~2048 samples): processed with minimal-latency FFT (e.g., 64–256 samples)
  - Tail blocks (remainder): processed in larger FFT blocks (amortised)

### Perceptual Quality
- Accurate room simulation — captures every reflection, diffusion, and resonance
- Fixed — cannot modulate room size or decay without re-processing the IR
- Cannot "tune" the reverb after capture (use EQ on IR to approximate adjustment)
- Memory intensive for long IRs at high sample rates

### IR Libraries
- **Fokke van Saane's collection**: free IRs of famous reverb hardware (EMT 140, Lexicon 480L)
- **OpenAIR**: acoustics of real spaces (cathedrals, concert halls)
- **Altiverb**: commercial high-quality IRs

---

## Algorithmic Parameters and Perceptual Effects

| Parameter     | Range            | Perceptual Effect                                                        |
|---------------|------------------|--------------------------------------------------------------------------|
| Pre-delay     | 0–80 ms          | Perceived distance: 0ms = close, 80ms = far/large space                  |
| RT60          | 0.1s – 20s+      | Room size: 0.1s = closet, 0.5s = small room, 2s = hall, 8s = cathedral   |
| Diffusion     | 0–100%           | Low = distinct flutter echoes; high = smooth, washy tail                  |
| Damping / HF decay | 0.3–3s    | Simulates air absorption; shorter HF RT60 = warmer, more natural         |
| Wet/dry mix   | 0–100%           | 10–20% = subtle; 40–60% = ambient; 100% = effect/send                    |
| Modulation    | 0–100%           | Slight pitch modulation of delay lines — reduces metallic resonance, adds shimmer |
| Stereo width  | 0–100%           | Mono reverb to fully decorrelated stereo                                  |

### HF Air Absorption in Real Rooms
- High frequencies (above 2 kHz) attenuate approximately 0.01–0.05 dB/m in typical air
- At 5 kHz in a 10 m × 10 m room: several dB more damping than at 500 Hz
- Algorithmic simulation: `T_HF ≈ 0.3–0.7 × RT60` (HF decay time is fraction of mid-band RT60)

---

## Reverb Use Cases with Recommended Settings

### Snare Drum
- Type: plate reverb or short room
- RT60: 0.8–1.5 s
- Pre-delay: 15–30 ms (separates reverb from snare snap)
- Damping: moderate (50%)
- Wet/dry: 100% wet on send aux, blend 15–30% on return

### Kick Drum
- Usually minimal or no reverb on kick
- If used: very short room (RT60 < 0.3 s), HPF on reverb return at 200 Hz
- Long kick reverb muddies the low end and reduces punch

### Vocals
- Type: medium room, hall, or plate
- RT60: 1.5–2.5 s
- Pre-delay: 20–30 ms (common vocal pre-delay for intelligibility)
- HF damping: 50% (warm reverb tail — not bright/harsh)
- Wet/dry: 15–30% as send, or 100% wet send with return level control

### Synth Pad
- Type: large hall or cathedral
- RT60: 3–8 s (or infinite/"freeze" for drone pad)
- Pre-delay: 0–10 ms (tight coupling)
- Diffusion: high (80–100%) for smooth wash
- Damping: low to moderate (retain HF shimmer)
- Wet/dry: 40–80% wet

### Piano
- Type: concert hall or plate
- RT60: 1.5–3 s
- Pre-delay: 10–20 ms
- Damping: moderate

### Drums (Send / Room)
- Type: short room or hall
- RT60: 0.8–1.5 s
- Full bus send: all drums to one reverb send for cohesion ("room glue")
- HPF on reverb return at 200–400 Hz (prevent mud)

### Ambient / Atmospheric
- Type: convolution (cathedral, cave) or infinite algorithmic
- RT60: 8–30 s
- Pre-delay: 0–50 ms
- Full wet, no dry signal in reverb channel
- Use as parallel effect

---

## Shimmer Reverb

A creative extension: pitch-shift feedback inside the reverb loop.

```
signal → reverb → pitch_shift(+12 semitones or +7 semitones) → back into reverb input
```

- Each pass through the reverb gains an octave (or fifth)
- Creates evolving, ethereal harmonic clouds
- Popular: Brian Eno, Sigur Rós, ambient music
- Implementation: delay-line pitch shifting (Doppler interpolation) inside feedback path

---

## Sample Rate Considerations

All delay times above are for 44100 Hz. For other sample rates:

```
delay_samples = round( delay_ms * sample_rate / 1000 )
```

Common practice: pick delay times as prime numbers near the desired ms value to minimise common divisors between delay lines and reduce audible resonance patterns.
