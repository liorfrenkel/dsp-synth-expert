---
title: "Subtractive Synthesis Architecture"
category: "Synth Architecture"
tags: [subtractive, VCO, VCF, VCA, ADSR, Minimoog, Juno, signal-flow]
sources:
  - https://en.wikipedia.org/wiki/Subtractive_synthesis
  - https://www.soundonsound.com/techniques/synth-secrets
  - https://www.attackmagazine.com/technique/synth-secrets/
---

## Overview

Subtractive synthesis starts with a harmonically rich source waveform and removes (subtracts) frequencies using filters to sculpt the final timbre. It is the dominant paradigm in analog and analog-modeled synthesizers: Minimoog, Moog Model D, ARP 2600, Roland Juno-106, Sequential Prophet-5, Korg MS-20, and virtually every classic hardware synth from the 1970s–1990s.

The core principle: **oscillator generates harmonics → filter removes some → amplifier shapes volume over time**.

---

## Signal Flow

```
VCO(s)
  ├─ Waveform select (saw / square / triangle / sine / pulse)
  ├─ Tuning: coarse (semitones), fine (cents), octave
  └─ Optional: hard sync, ring mod, cross-mod

Noise Generator (white / pink)

Mixer
  ├─ OSC 1 level
  ├─ OSC 2 level
  ├─ OSC 3 level (if available)
  ├─ Noise level
  └─ External audio input level

VCF (Voltage-Controlled Filter)
  ├─ Cutoff frequency (Fc): 20Hz – 20kHz
  ├─ Resonance (Q / Emphasis): 0–100%
  ├─ Filter type: LP / HP / BP / Notch
  ├─ Slope: 2-pole (−12dB/oct), 4-pole (−24dB/oct)
  ├─ Envelope Amount: filter env → Fc
  ├─ Key tracking: note pitch → Fc (0–100%)
  └─ Modulation: LFO → Fc, velocity → Fc

VCA (Voltage-Controlled Amplifier)
  ├─ Amplitude envelope (ADSR) → gain
  ├─ Velocity sensitivity
  └─ LFO → tremolo (AM)

Output → Effects → DAW / Mixer
```

---

## VCO Waveforms and Harmonic Content

### Sawtooth Wave (Ramp Down)
- Contains **all harmonics** (fundamental + 2nd + 3rd + 4th + ...)
- Amplitude of nth harmonic: `An = A1 / n`
- Explicit series: 1.000, 0.500, 0.333, 0.250, 0.200, 0.167, 0.143, 0.125...
- Bright, buzzy, string/brass-like
- Most common starting point for subtractive patches

```
f(t) = (A/2) - (A/π) * Σ[n=1..∞] sin(2π·n·f·t) / n
```

### Square Wave (50% Pulse Width)
- Contains **only odd harmonics**: 1st, 3rd, 5th, 7th...
- Amplitude: `An = A1 / n` for odd n only
- Explicit: 1.000, 0, 0.333, 0, 0.200, 0, 0.143, 0, 0.111...
- Hollow, woody, clarinet-like timbre
- Classic lead and bass tone on Roland SH series, Minimoog

```
f(t) = (4A/π) * Σ[n=1,3,5...] sin(2π·n·f·t) / n
```

### Triangle Wave
- Contains **only odd harmonics** with faster rolloff
- Amplitude: `An = A1 / n²` for odd n only
- Explicit: 1.000, 0, 0.111, 0, 0.040, 0, 0.020, 0, 0.012...
- Soft, flute-like, fewer harmonics than square
- Often used as LFO source or gentle sub-oscillator

```
f(t) = (8A/π²) * Σ[n=1,3,5...] (−1)^((n−1)/2) · sin(2π·n·f·t) / n²
```

### Sine Wave
- **Only fundamental**, no harmonics
- Used as sub-oscillator (one octave down for bass fattening)
- Starting point for FM synthesis (see fm_synthesis.md)
- Very clean, can only be shaped by the envelope itself

### Pulse Wave and PWM
- Variable duty cycle D (0–100%)
- At D=50%: square wave (odd harmonics)
- At D=25%: pulse—harmonics 1,2,3 present but 4th absent, then 5,6,7 present but 8th absent
- Pattern: all multiples of (1/D) are absent
- Pulse width modulation (PWM): LFO or envelope sweeps duty cycle → chorus-like phase shifting
- Classic PWM pads: Roland Juno-106 DCO, Oberheim OB-Xa

**PWM Formula:**  
For pulse width D (0 to 1):
```
An = (2A / nπ) * sin(n·π·D)
```
At n·D = integer: partial cancels out entirely.

---

## Classic Synthesizer Architectures

### Minimoog (1970)
- **3 VCOs** (Osc 1, 2, 3) — Osc 3 can be used as LFO
- Waveforms per oscillator: triangle, triangle-sawtooth, reverse sawtooth, sawtooth, square, wide pulse, narrow pulse
- **Mixer**: 5 sources (OSC1, OSC2, OSC3, Noise, External)
- **Moog Ladder Filter** (4-pole, −24dB/oct): classic warm character; self-oscillates at full resonance
- Filter envelope: dedicated ADSR → Fc amount
- **Loudness envelope**: ADSR
- Keyboard: 44 notes, lowest priority triggering
- **No patch memory**, no MIDI (original), fully hardwired signal path
- Monophonic; later re-issued as Minimoog Voyager (2002) and Model D reissue (2016)

### ARP 2600 (1970)
- **Semi-modular**: pre-patched but all connections overridable via 3.5mm jacks
- 3 VCOs with hard sync capability
- **Dual filter topology**: 4-pole lowpass + 2-pole highpass in series, or used independently
- Sample & Hold with white noise source → random CV
- Voltage-controlled LFO with audio-rate capability
- Spring reverb built in
- Ext signal processor: envelope follower + ring mod + preamp
- Used by: Herbie Hancock, Edgar Winter, Pete Townshend

### Roland Juno-106 (1984)
- **DCO** (Digitally Controlled Oscillator) — stable tuning vs. analog VCOs
- One DCO per voice + sub-oscillator (one octave below, square wave)
- Waveforms: sawtooth, square, pulse
- **PWM**: square wave with LFO-controlled width → lush pads
- 6-voice polyphonic
- **IR3109 VCF chip** (Roland proprietary): 2-pole (−12dB/oct) analog filter (Sallen-Key topology)
- Simple single ADSR for both filter and amp (shared envelope)
- **Built-in chorus** (2 modes): signature sound of the Juno series
- DCB and MIDI connectivity
- Famous for: house music chords, 80s pop pads

### Sequential Circuits Prophet-5 (1978)
- **5-voice polyphony** — first fully programmable polyphonic analog synth
- Two VCOs per voice: CEM3340 VCO chips (Curtis Electromusic)
- OSC B can hard sync to OSC A
- **CEM3320 VCF chip**: 4-pole lowpass, −24dB/oct, Moog-like character
- **CEM3310 envelope**: ADSR per voice, VCF + VCA envelopes
- Poly mod: filter envelope → OSC B pitch, OSC A → filter cutoff (proto-FM)
- 120 patch memory locations
- Rev 1 (SSM chips), Rev 2, Rev 3.3 (CEM chips) — each has distinct character

### Korg MS-20 (1978)
- Semi-modular with external patch bay
- **Dual filter topology** (unique): highpass then lowpass in series
  - HP filter: 2-pole Sallen-Key variant, self-oscillating
  - LP filter: 2-pole Sallen-Key variant, self-oscillating
- Both filters self-oscillate simultaneously → dual sine wave output
- External signal processor: envelope follower (audio → CV) + frequency-to-voltage converter
- 2 VCOs, 2 envelopes, 1 LFO + 1 env-like modulator
- Monophonic
- Famous for: harsh industrial textures, self-oscillation drones

### Oberheim OB-Xa (1980)
- 4/6/8-voice polyphony (customer configurable)
- Two CEM3340 VCOs per voice
- **SEM-inspired 2-pole state-variable filter (SVF)**: simultaneous LP/HP/BP/Notch outputs
- Filter slope: −12dB/oct (gentler, more open sound than Moog)
- Master transpose, unison mode (all voices play same note with detuning)
- Used on: Michael Jackson "Thriller", classic Van Halen

---

## VCF Filter Types

### 4-Pole Moog Ladder Filter (−24dB/oct)
- 4 one-pole RC stages in series with resonance feedback
- Each stage contributes −6dB/oct → total −24dB/oct
- Resonance feeds back from output to input
- **Self-oscillation** at max resonance: becomes a sine wave oscillator
- Self-oscillation frequency ≈ cutoff frequency
- Character: warm, round, musical saturation at high input levels
- Transistor ladder: Minimoog, Moog Voyager, Moog One
- Diode ladder variant: Roland TB-303 (sharper, more resonant peak)

### 2-Pole Sallen-Key (−12dB/oct)
- Active filter topology using two RC stages + op-amp
- Gentler slope, more harmonics survive at cutoff
- Softer, more transparent sound
- Found in: Roland Juno series (IR3109), Oberheim SEM (modified)
- Less "filter wobble" characteristic, works well for pads

### State-Variable Filter (SVF)
- Simultaneously outputs lowpass, highpass, bandpass, notch
- 2-pole (−12dB/oct) for each output
- Resonance/Q adjustable without changing other parameters
- Can morph continuously between filter types via CV
- Found in: Oberheim SEM, Korg Mono/Poly, Arturia Minibrute

### Steiner-Parker (Synthacon Filter)
- Multi-mode, but different topology from SVF
- One input, three separate filter outputs (LP/HP/BP)
- Pronounced resonance character
- Found in: Synthacon, Moog Minitaur (modified)

### MS-20 Dual Filter
- HP → LP cascade: can produce bandpass effect
- Each filter self-oscillates independently
- Tuned separately: two pitchable sine oscillators "for free"
- Notch-like behavior when both resonances overlap in frequency

---

## Resonance Behavior

- **Q factor** (quality factor): ratio of center frequency to bandwidth
  - `Q = Fc / BW` where BW = 3dB bandwidth
  - Low Q (0.5–1): wide, gentle peak
  - Medium Q (2–5): pronounced resonance peak
  - High Q (10+): sharp, narrow boost; approaching self-oscillation
- **Self-oscillation threshold**: resonance high enough that filter sustains sine oscillation
  - At self-oscillation: filter adds a pitched tone at Fc even with no input signal
  - Sweep cutoff while self-oscillating → portamento sine wave
- **Resonance amplitude boost**: at resonance peak, up to +20dB above the cutoff region
- **Input level interaction**: overdriving filter input (pre-filter saturation) modifies resonance character
  - Moog ladder: soft clipping in stages, musical saturation
  - MS-20: distinctive harsh clipping from FET stages
- **Keyboard tracking at resonance**: self-oscillation pitch follows played note → musical intervals

---

## ADSR Envelope Mathematics

```
Attack:  linear ramp from 0 to peak level
         duration: 0.1ms – 10s
         output(t) = peak * (t / T_attack)  [linear]
         or: output(t) = peak * (1 - e^(-t/τ_a))  [RC charge, more common in analog]

Decay:   exponential fall from peak to sustain level
         duration: 1ms – 10s
         output(t) = sustain + (peak - sustain) * e^(-t/τ_d)
         τ_d = T_decay / (−ln(target_fraction))  [target ≈ 0.001 for "complete" decay]

Sustain: held level while key is pressed
         range: 0–100% of peak
         NOT a time parameter—it is a level

Release: exponential fall from sustain level to 0
         starts when key is released
         duration: 5ms – 30s
         output(t) = sustain * e^(-t/τ_r)
```

**Typical parameter ranges:**
| Parameter | Min | Max | Unit |
|-----------|-----|-----|------|
| Attack    | 0.1ms | 10s | time |
| Decay     | 1ms | 10s | time |
| Sustain   | 0% | 100% | level |
| Release   | 5ms | 30s | time |

**Envelope applications:**
- **VCF Envelope**: positive amount → filter opens on note attack, closes over decay; negative amount → filter closes on attack (dark start, then opens)
- **VCA Envelope**: shapes volume over time; slow attack = pad, fast attack = pluck/stab
- **Pitch Envelope**: small positive amount → pitch starts high, glides down → "pew" laser sound

**Velocity to envelope depth**: MIDI velocity (0–127) scales envelope peak or depth

---

## LFO (Low Frequency Oscillator) Parameters

LFOs modulate parameters cyclically at sub-audio rates.

| Parameter | Range | Notes |
|-----------|-------|-------|
| Rate      | 0.01Hz – 20Hz | Below ~0.1Hz: slow sweep; above ~10Hz: vibrato/trill |
| Depth     | 0–100% | Amount of destination parameter modulated |
| Delay     | 0ms – 5s | Time before LFO kicks in after note trigger |
| Fade-in   | 0ms – 5s | Ramp-up time of LFO depth after trigger |
| Phase     | 0–360° | Start phase of LFO cycle |

**LFO Waveforms:**
- **Sine**: smooth, even vibrato or tremolo
- **Triangle**: linear up-down sweep; slightly more "mechanical" than sine
- **Square**: abrupt two-state switching; trill effect on pitch, gated tremolo on amp
- **Sawtooth up (ramp)**: slow rise, instant drop; rising filter sweep
- **Sawtooth down (reverse ramp)**: instant rise, slow fall; descending sweep
- **Sample & Hold (S&H)**: new random value held for each cycle; random stepping
- **Random smooth (S&H with interpolation)**: random but gliding between values

**Common LFO Destinations:**
- OSC pitch → vibrato (rate 4–7Hz, depth ±50 cents for subtle)
- OSC pulse width → PWM chorus effect (rate 0.1–2Hz, depth 30–80%)
- VCF cutoff → filter wobble (rate 0.5–4Hz, depth varies)
- VCA amplitude → tremolo (rate 4–8Hz, depth 20–60%)
- Pan → auto-pan (rate 0.2–2Hz)

**LFO sync**: to MIDI clock or host tempo; rate expressed as note values (1/4, 1/8, 1/16, dotted, triplet)

---

## Filter FM (Audio-Rate Filter Modulation)

Modulating the filter cutoff at audio rate (using oscillator output as mod source) produces sidebands and tonal complexity.

- **Source**: OSC 2 or 3 audio output → VCF cutoff CV input
- **Effect**: each harmonic of the modulator creates sidebands around the carrier harmonics
- **Metallic / bell tones**: non-integer C:M ratios (e.g., mod freq = 1.41× carrier)
- **Growl / aggression**: moderate mod depth at sub-audio rates (10–50Hz)
- **FM-like complexity**: at high mod depths, spectrum becomes densely inharmonic

This is distinct from true FM synthesis: filter FM modulates the filter cutoff, not the oscillator frequency directly.

---

## Voice Architecture

### Monophonic (Mono)
- 1 voice: only one note at a time
- Classic mono synths: Minimoog, TB-303, Korg MS-20
- Last note priority / lowest note priority / highest note priority options
- Legato / retrigger: in legato mode, envelope does not retrigger on slurred notes

### Paraphonic
- Multiple oscillators but a single shared VCF + VCA
- All oscillators run through the same filter simultaneously
- Chords can be played but all notes share same filter articulation
- Examples: Korg Mono/Poly, ARP Quadra, some Juno modes

### Polyphonic (Poly)
- **4-voice**: Oberheim Two-Voice, Sequential Six-Trak
- **6-voice**: Roland Juno-106, Sequential Prophet-6
- **8-voice**: Oberheim OB-8, Prophet-08
- **16-voice**: Korg Polysix (6 voice), Korg DW-8000, modern: Prophet-16

**Voice allocation modes:**
- Last note: oldest voice stolen when all voices occupied
- Round robin: voices cycled sequentially
- Reset: all voices reset to same state (phase lock)

### Unison Mode
- All available voices play same note simultaneously
- Detune amount: each voice offset in cents (±0 to ±50 cents typical)
- Creates dense, wide, fat sound
- Spreading across stereo field: alternate voices panned left/right
- Chord mode: voices spread across chord intervals instead of unison

---

## Key Tracking (Key Follow / Keyboard Follow)

Filter cutoff tracks played note pitch:
- **0%**: no tracking; cutoff fixed regardless of pitch
- **50%**: cutoff rises half a semitone per semitone played
- **100%**: cutoff rises one semitone per semitone; filter maintains tonal consistency across keyboard
- **>100%** (e.g., 200%): filter opens more than proportional to pitch

**Why 100% tracking matters:**  
At 100%, if cutoff is set to 2× fundamental, ratio is preserved across all octaves → consistent timbre from low to high.

**Negative tracking**: cutoff decreases as pitch rises → unusual, experimental.

---

## Modulation Routing (Simplified Matrix)

```
Sources              Destinations
──────────           ────────────
LFO (1-4)     →     OSC pitch
Envelope (1-4) →    OSC level
Velocity       →    Filter cutoff
Aftertouch     →    Filter resonance
Mod wheel      →    LFO depth
Pitch bend     →    OSC pitch (±2 or ±12 semitones typical)
Key tracking   →    Filter cutoff
```

---

## Patch Design Examples

### Classic Analog Pad (Juno-style)
```
OSC: DCO sawtooth + sub octave square
PWM: LFO → pulse width, rate 0.3Hz, depth 60%
Filter: LP 2-pole, Fc=3kHz, Resonance=15%, Key track=80%
Filter Env: A=20ms, D=800ms, S=70%, R=1.2s, Amt=+30%
Amp Env: A=80ms, D=0, S=100%, R=1.5s
Chorus: ON (mode 2 for widest spread)
```

### Funky Bass (Minimoog-style)
```
OSC1: Sawtooth, 8' (concert pitch)
OSC2: Sawtooth, 8' + 3 cents detune
OSC3: Square, 16' (one octave down)
Filter: LP 4-pole, Fc=500Hz, Resonance=50%, Key track=50%
Filter Env: A=1ms, D=150ms, S=20%, R=50ms, Amt=+80%
Amp Env: A=2ms, D=200ms, S=80%, R=80ms
Velocity → Filter Env Amount: 50%
```

### Lead (Prophet-5 style)
```
OSC A: Sawtooth
OSC B: Square, hard synced to OSC A, slightly above unison
Filter: LP 4-pole, Fc=2kHz, Resonance=40%, Key track=100%
Filter Env: A=5ms, D=300ms, S=50%, R=400ms, Amt=+60%
Amp Env: A=3ms, D=100ms, S=80%, R=300ms
LFO → pitch: sine, 5.5Hz, depth=15 cents (vibrato, delayed 0.5s)
```

---

## Classic Analog Synth Component ICs

| Chip | Function | Used in |
|------|----------|---------|
| CEM3340 | VCO | Prophet-5 Rev 3, OB-Xa, SH-101 |
| CEM3320 | VCF | Prophet-5 Rev 3 |
| CEM3310 | ADSR envelope | Prophet-5 Rev 3 |
| SSM2044 | VCF | Sequential Pro-One, Korg Polysix |
| SSM2056 | VCA | Multiple |
| IR3109 | VCF | Roland Juno series |
| BA662 | VCA | Roland TB-303, SH-101 |
| LM13700 | OTA (VCF/VCA) | Generic; Moog Slim Phatty |
| AS3394 | Complete voice (VCO+VCF+VCA) | Budget analogs |

---

## Spectral Characteristics Reference

| Waveform | Fundamental | 2nd | 3rd | 4th | 5th | Character |
|----------|-------------|-----|-----|-----|-----|-----------|
| Sawtooth | 1.000 | 0.500 | 0.333 | 0.250 | 0.200 | Bright, rich |
| Square   | 1.000 | 0.000 | 0.333 | 0.000 | 0.200 | Hollow, woody |
| Triangle | 1.000 | 0.000 | 0.111 | 0.000 | 0.040 | Soft, sine-like |
| Pulse 25%| 1.000 | 0.900 | 0.707 | 0.000 | 0.588 | Nasal, thin |
| Sine     | 1.000 | 0.000 | 0.000 | 0.000 | 0.000 | Pure tone |

---

## Detuning and Supersaw

**Multiple oscillator detune**: 2+ oscillators tuned slightly apart
- Beat frequency = |f1 - f2| Hz → perceived as amplitude modulation / chorus when <15Hz
- 7 detuned sawtooth oscillators = "supersaw" (Roland JP-8000 SuperSAW)
- Spread: ±0 to ±50 cents per oscillator relative to center pitch
- Chorusing effect: analog "width" without external chorus

**Unison detune** (mono mode):
- All voices (if poly synth switched to mono unison) detuned
- ±2 to ±25 cents spread produces massive, dense tone

---

## Further Reference

- Gordon Reid's "Synth Secrets" (Sound on Sound, 63 parts, 1999–2004): definitive series on subtractive synthesis theory
- "Analog Days" (Pinch & Trocco): social history of Moog synthesizer
- Yamaha DX7 Service Manual: comparison baseline for digital implementations
