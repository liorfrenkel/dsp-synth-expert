---
title: "FM Synthesis: Theory and Programming"
category: "Synth Architecture"
tags: [FM, DX7, operator, modulation-index, Yamaha, algorithms, sideband]
sources:
  - https://en.wikipedia.org/wiki/Frequency_modulation_synthesis
  - https://www.soundonsound.com/techniques/synth-secrets
  - Chowning, J. (1973). "The Synthesis of Complex Audio Spectra by Means of Frequency Modulation"
---

## Core Theory

FM synthesis modulates the **frequency** (or phase) of one oscillator (carrier) with the output of another oscillator (modulator). The result is a spectrum with sidebands that can produce rich, complex timbres from just two oscillators.

**Developed**: John Chowning at Stanford University, 1967–1973  
**First commercial implementation**: Yamaha GS-1 (1980), then DX7 (1983)  
**Patent**: Stanford University US Patent 4,018,121 (expired 1995)

---

## Fundamental Formula

```
output(t) = A · sin(2π·fc·t + β · sin(2π·fm·t))
```

Where:
- `A` = carrier amplitude
- `fc` = carrier frequency (Hz)
- `fm` = modulator frequency (Hz)
- `β` = modulation index (dimensionless)
- `t` = time (seconds)

### Modulation Index β

```
β = Δf / fm = peak_frequency_deviation / modulator_frequency
```

- `Δf` = peak instantaneous frequency deviation from carrier
- Higher β → more sidebands → brighter, more complex spectrum
- β = 0: pure sine wave at carrier frequency
- β = 1: modest harmonic content
- β = 3: clearly bright with many partials
- β = 7: very bright, approaching noise-like density

**DX7 equivalent**: Output Level of modulator controls Δf. β is not set directly; instead, the modulator's output level (0–99) × key velocity scaling × envelope level produces an effective β.

---

## Sideband Theory (Bessel Functions)

When modulation is sinusoidal, the spectrum of the carrier+modulator is:

```
output(t) = A · Σ[n=−∞..∞] Jn(β) · sin((ωc + n·ωm)·t)
```

Where:
- `Jn(β)` = Bessel function of the first kind, order n, argument β
- Sideband frequencies: `fc ± n·fm` for n = 1, 2, 3, ...
- n=0 term: carrier frequency `fc`

**Sideband pairs**: upper sideband at `fc + n·fm`, lower sideband at `fc − n·fm`  
When lower sideband goes negative (fc − n·fm < 0): it "folds back" with a 180° phase inversion, potentially canceling upper sidebands.

### Bessel Function Values Jn(β)

| β \ n | J0 | J1 | J2 | J3 | J4 | J5 |
|-------|-----|-----|-----|-----|-----|-----|
| 0.0   | 1.000 | 0.000 | 0.000 | 0.000 | 0.000 | 0.000 |
| 0.25  | 0.984 | 0.124 | 0.008 | 0.000 | 0.000 | 0.000 |
| 0.5   | 0.938 | 0.242 | 0.031 | 0.003 | 0.000 | 0.000 |
| 1.0   | 0.765 | 0.440 | 0.115 | 0.020 | 0.002 | 0.000 |
| 2.0   | 0.224 | 0.577 | 0.353 | 0.129 | 0.034 | 0.007 |
| 3.0   | −0.260 | 0.339 | 0.487 | 0.309 | 0.132 | 0.043 |
| 5.0   | −0.178 | −0.328 | 0.047 | 0.365 | 0.391 | 0.261 |
| 7.0   | 0.300 | 0.301 | −0.301 | −0.168 | 0.158 | 0.348 |

---

## C:M Ratio Effects

The **carrier-to-modulator ratio (C:M)** determines whether the resulting spectrum is harmonic (musical) or inharmonic (metallic/percussive).

### Harmonic C:M Ratios

| C:M Ratio | Resulting Harmonics | Character |
|-----------|--------------------|-----------| 
| 1:1 | 1, 2, 3, 4, 5, 6... (all) | Rich, like sawtooth |
| 1:2 | 1, 3, 5, 7... (odd only) | Clarinet/woodwind character |
| 1:3 | 1, 4, 7, 10... (every 3rd) | Hollow, specific harmonic gaps |
| 1:4 | 1, 5, 9, 13... | Thin, specific resonance |
| 2:1 | Sub-fundamental + harmonics | Sub-octave content added |
| 3:1 | Carrier 3× modulator | Strong sub harmonics |
| 3:2 | 3/2 ratio → 5th harmonic series | Brass-like, 5th interval |
| 2:3 | 3rd harmonic series | Metallic tones |
| N:1 | Complex harmonic | Rich inharmonics |

### Inharmonic C:M Ratios (Bell/Metallic)

| C:M Ratio | Character |
|-----------|-----------|
| 1:1.41 (≈1:√2) | Bell, metallic percussion |
| 1:2.756 | Cowbell-like |
| 1:1.5 (3:2) | 5th interval + harmonics |
| 3.5:1 | Gong-like |
| 7:11 | Complex inharmonic |

**Rule**: integer ratios → harmonic/musical; non-integer ratios → inharmonic/metallic/percussive

---

## DX7 Operator System

The Yamaha DX7 uses 6 operators arranged in one of 32 fixed **algorithms**.

### What is an Operator?
An operator is a sine wave oscillator combined with an envelope generator. Each operator has:
- A **frequency** (ratio or fixed Hz)
- An **amplitude envelope** (4 rates + 4 levels)
- Various scaling and sensitivity parameters

### Carrier vs. Modulator
- **Carrier**: operator whose output goes to the audio output
- **Modulator**: operator whose output modulates the frequency of a carrier (or another modulator)
- In an algorithm diagram, arrows show modulator → carrier connections

---

## All 32 DX7 Algorithms

Notation: [Op] = operator number, arrows show modulation direction, → audio = outputs to audio

```
Algo 1:  [2]→[1]→out, [4]→[3]→out, [6]→[5]→out  (3 stacks of 2)
Algo 2:  [2,3]→[1]→out, [5]→[4]→out, [6]→out     (mixed)
Algo 3:  [3]→[2]→[1]→out, [5]→[4]→out, [6]→out   (1 stack of 3, 1 of 2, 1 solo)
Algo 4:  [2]→[1]→out, [4,5,6]→[3]→out             (1 chain of 2, 1 with 3 mods)
Algo 5:  [2]→[1]→out, [4]→[3]→out, [6]→[5]→out   (3 chains with feedback on 1 or 2)
Algo 6:  [2,3,4,5,6]→[1]→out                      (5 mods into 1 carrier)
Algo 7:  [3]→[1,2]→out, [5]→[4]→out, [6]→out      (1 mod into 2 carriers)
Algo 8:  [3]→[1,2]→out, [5,6]→[4]→out             (branching)
Algo 9:  [3]→[1,2]→out, [6]→[4,5]→out             (two branching pairs)
Algo 10: [4]→[1,2,3]→out, [6]→[5]→out             (1 mod into 3 carriers)
Algo 11: [3]→[1,2]→out, [4,5,6]→out               (branch + 3 direct carriers)
Algo 12: [4,5,6]→[1,2,3]→out                      (3 mods into 3 carriers)
Algo 13: [3,4,5,6]→[1,2]→out                      (4 mods into 2 carriers)
Algo 14: [3,4,5]→[1]→out, [6]→[2]→out             (complex)
Algo 15: [2]→[1]→out, [4,5,6]→[3]→out             (2-chain + triple mod into carrier)
Algo 16: [5]→[4,3,2,1]→out, [6]→out               (serial 4 + 1 direct carrier mod)
Algo 17: [5]→[4]→[3]→[2]→[1]→out, [6]→out        (chain of 5 + 1 solo carrier)
Algo 18: [6]→[5]→[4]→[3]→[2]→[1]→out             (chain of 6: deepest serial FM)
Algo 19: [2]→[1]→out, [3,4,5,6]→out               (1 chain + 4 direct carriers)
Algo 20: [2,3,4,5]→[1]→out, [6]→out               (4 mods into 1 + 1 direct carrier)
Algo 21: [2,3,4,5,6]→[1]→out                      (5 mods into 1 carrier — max modulation)
Algo 22: [2]→[1]→out, [3,4,5,6]→out               (1 chain of 2 + 4 direct carriers)
Algo 23: [2,3]→[1]→out, [4,5,6]→out               (2 mods into 1 + 3 direct carriers)
Algo 24: [2,3,4]→[1]→out, [5,6]→out               (3 mods into 1 + 2 direct carriers)
Algo 25: [2,3,4,5]→[1]→out, [6]→out               (4 mods into 1 + 1 direct)
Algo 26: [2]→[1]→out, [4]→[3]→out, [5,6]→out     (2 chains + 2 direct carriers)
Algo 27: [3]→[1,2]→out, [5]→[4]→out, [6]→out     (branching + chain + direct)
Algo 28: [3]→[2]→[1]→out, [6]→[5]→[4]→out        (two chains of 3: brass algorithm)
Algo 29: [4]→[3]→[2]→[1]→out, [5,6]→out          (chain of 4 + 2 direct carriers)
Algo 30: [5]→[4]→[3]→[2]→[1]→out, [6]→out        (chain of 5 + 1 direct carrier)
Algo 31: [2]→[1]→out, [3,4,5,6]→out               (same as 22/19 variant)
Algo 32: [2,3,4,5,6]→[1]→out                      (5 modulators, 1 carrier — additive with mod)
```

**Practical algorithm selection guide:**
- Algorithms 1–8: Complex timbre, multiple stacks → bells, pianos, electric pianos
- Algorithms 15–20: Brightness control, single carrier → leads, organs
- Algorithms 21–32: Additive-like (many carriers with fewer modulators) → complex evolving pads
- Algorithm 5: Classic E-Piano (Yamaha "Rhodes" preset)
- Algorithm 28: Brass, winds (two independent 3-operator stacks)
- Algorithm 18: Maximum timbral complexity (single chain of 6)

---

## DX7 Operator Parameters

### Envelope Generator (EG) — Per Operator
Each operator has a 4-stage envelope with 4 rates and 4 levels (not standard ADSR):

```
Rate 1 (R1):   0–99   Attack rate (higher = faster)
Level 1 (L1):  0–99   Level after attack (usually 99 = max)
Rate 2 (R2):   0–99   Decay 1 rate
Level 2 (L2):  0–99   Level after decay 1
Rate 3 (R3):   0–99   Decay 2 rate (sustain approach)
Level 3 (L3):  0–99   Sustain level (held while key down)
Rate 4 (R4):   0–99   Release rate
Level 4 (L4):  0–99   Release target level (usually 0)
```

Note: EG starts at Level 4, then transitions L4→L1 at R1, L1→L2 at R2, L2→L3 at R3, then on note-off L3→L4 at R4.

### Oscillator Parameters — Per Operator
```
Output Level:        0–99    Amplitude of this operator
Osc Frequency Mode:  Ratio | Fixed
  Ratio mode:  Coarse (0–31) × Fine (0–99) → frequency as multiple of key pitch
  Fixed mode:  Specific Hz regardless of key (useful for kick drums, effects)
Osc Detune:          −7 to +7   Fine pitch adjustment in ~1 cent steps
Feedback:            0–7        Only operator 1 (or algorithm-specific) feeds back on itself
```

### Scaling Parameters — Per Operator
```
Break Point (BP):    MIDI note A0 to C8   Keyboard pivot note for scaling
Left Curve:          -Lin | -Exp | +Exp | +Lin   Scaling shape below BP
Left Depth:          0–99   How much scaling to apply below BP
Right Curve:         -Lin | -Exp | +Exp | +Lin   Scaling shape above BP
Right Depth:         0–99   How much scaling to apply above BP
Rate Scaling:        0–7    Higher note → faster envelope rates (shorter sounds)
```

### Sensitivity Parameters — Per Operator
```
Amp Mod Sensitivity: 0–3    Response to amplitude modulation (LFO modulating volume)
Key Velocity:        0–7    How much velocity affects operator output level
```

---

## Feedback Operator

- Only **one operator** per voice can self-feedback (on DX7: Operator 1)
- **Feedback level 0**: pure sine
- **Feedback 1–3**: soft saturation, slightly richer sine
- **Feedback 4–5**: mild harmonics added
- **Feedback 6**: moderate harmonic content
- **Feedback 7**: maximum feedback → produces **all harmonics** (sawtooth-like waveform)
- Self-feedback creates a stable nonlinear oscillation when β is large enough
- Use feedback operator as a carrier + modulated by other operators = complex timbres

---

## β vs. Timbre Reference Table

| β Value | Spectral Character | Sounds Like |
|---------|-------------------|-------------|
| 0 | Pure sine | Flute fundamental, test tone |
| 0.25 | Slight 1st sideband | Soft tone with hint of harmonics |
| 0.5 | 2 sidebands audible | Mellow, slightly brighter sine |
| 1.0 | Rich, 3+ sidebands | Gentle string, distant bell |
| 2.0 | Moderate harmonics | Bright, formant-like |
| 3.0 | Many harmonics | Brass-like, vibrant |
| 5.0 | Very dense spectrum | Thick, metallic, noisy timbre |
| 7.0 | Near-noise density | Gong, cymbal shimmer |
| 10+ | Noise-like | Metallic noise, percussive attacks |

**Dynamic β via modulator envelope**: high β at attack → bright transient, β falls over decay → timbre softens. This is the essential FM programming trick for piano/E-piano sounds.

---

## Classic DX7 Patches

### E-Piano (Algorithm 5)
```
Algorithm: 5 (three 2-op stacks: [2→1], [4→3], [6→5])
Op 1 (carrier A): Level=99, Freq ratio 1.00, EG: fast attack, moderate decay
Op 2 (mod A):     Level=75, Freq ratio 14.00, EG: very fast attack, quick decay
                  → creates inharmonic bell attack transient
Op 3 (carrier B): Level=85, Freq ratio 1.00
Op 4 (mod B):     Level=50, Freq ratio 1.00, Feedback=0
Op 5 (carrier C): Level=70, Freq ratio 1.00
Op 6 (mod C):     Level=30, Freq ratio 1.00
Key velocity → Op 2 level: creates brighter attack when played harder
Rate scaling: high (short sustain at high notes)
```

### DX7 Bass (Algorithm 1)
```
Algorithm: 1 (two 3-op stacks: [3→2→1] and [6→5→4])
Stack 1: Ops 3→2→1
  Op 1 (carrier): Freq 1.00, Level=99
  Op 2 (mid mod): Freq 1.00, Level=60, EG fast decay
  Op 3 (deep mod): Freq 0.50, Level=80, EG medium attack/decay
Stack 2: Ops 6→5→4
  Op 4 (carrier): Freq 1.00, Level=70
  Op 5 (mid mod): Freq 2.00, Level=55
  Op 6 (deep mod): Freq 1.00, Level=45, Feedback=5
```

### Brass (Algorithm 28)
```
Algorithm: 28 (two 3-op serial stacks: [3→2→1] and [6→5→4])
Stack 1 (tone): 
  Op 1 (carrier): Freq 1.00, Level=99, Feedback=3
  Op 2: Freq 1.00, Level=70, EG: shaped to match brass attack
  Op 3: Freq 1.00, Level=60
Stack 2 (growl):
  Op 4 (carrier): Freq 1.00, Level=85
  Op 5: Freq 1.00, Level=65
  Op 6: Freq 1.00, Level=55, Feedback=0
Modulator EGs: sharp attack, decay-only (mirrors real brass envelope inversion)
```

---

## 4-Operator FM (OPL2/OPL3, TX81Z)

### Yamaha OPL2 (YM3812) — AdLib Sound Card
- 4-operator FM but only 2 operators per voice (2-op mode), 9 voices
- Used in: AdLib sound card (1987), Sound Blaster OPL2 mode
- Algorithm: single modulator → single carrier (no choice)
- Waveforms: sine, half-sine, absolute sine, pulse-sine

### Yamaha OPL3 (YMF262) — Sound Blaster 16
- 4-operator FM, 18 two-op voices or 6 four-op + 6 two-op voices
- Algorithms: can configure 4-op as [M→C→C] or [M→C + M→C]
- Waveforms: 8 shapes per operator
- Stereo output with per-channel panning
- Used in: Sound Blaster 16, Gravis Ultrasound

### Yamaha TX81Z (DX21 module)
- 4-operator FM with DX7-compatible algorithm subset (8 algorithms vs 32)
- **8 additional waveforms** per operator (not just sine):
  - Sine, half-sine, absolute sine, pulse-quarter sine, sine-even, absolute sine-even, square sine, derived square
- More timbral variety than pure sine-only DX7 at 4-op
- Famous for: E-bass preset ("Lately Bass"), which became a hip-hop and R&B staple

### DX7 vs TX81Z vs OPL2 Comparison

| Feature | DX7 | TX81Z | OPL2 |
|---------|-----|-------|------|
| Operators/voice | 6 | 4 | 2 |
| Algorithms | 32 | 8 | 1 |
| Waveforms/op | 1 (sine) | 8 | 4 |
| Polyphony | 16 | 8 | 9 |
| Velocity layers | Yes | Yes | No |
| Pitch EG | Yes | Yes | No |

---

## Korg FM Synthesizers

### Korg Volca FM (2016)
- 6-operator, 3-voice, DX7 algorithm-compatible
- Loads DX7 SysEx patches
- 32 algorithms matching DX7
- Built-in sequencer, MIDI, sync
- Operators use sine only (same as DX7)
- Parameter access improved vs DX7's cryptic interface

### Korg opsix (2020)
- 6-operator FM with 40 algorithms (8 new beyond DX7's 32)
- Hybrid: operators can be subtractive, waveshaping, filter, ring mod
- Each operator: choice of 11 operator types
- Multi-mode filters per operator
- Allows FM + subtractive in same patch

---

## Phase Modulation vs. Frequency Modulation

Yamaha's DX7 implementation uses **phase modulation (PM)**, not strict FM:

```
FM: output(t) = A · sin(ωc·t + β · sin(ωm·t))
PM: output(t) = A · sin(ωc·t + φ · sin(ωm·t))
```

Where φ is the phase deviation (radians) rather than β (frequency deviation / modulator frequency).

Mathematical equivalence: PM with sinusoidal modulator = FM with sinusoidal modulator when modulation is applied at audio rate. The DX7 computed PM because it is phase-coherent and easier to implement digitally without integration errors.

**Practical difference**: In FM, β = Δf/fm (scales with modulator amplitude divided by modulator frequency). In PM (DX7 style), the index is independent of modulator frequency — the same modulator level produces the same phase deviation regardless of modulator pitch. This means DX7 FM patches with non-1:1 C:M ratios behave differently than "true" FM theory predicts.

---

## Through-Zero FM (TZFM)

Standard VCO FM cannot go below 0 Hz — the oscillator stalls or exhibits non-linear behavior at the zero crossing.

**TZFM**: oscillator passes through 0 Hz and reverses phase direction.
- Output at negative frequencies: time-reversed waveform
- Creates symmetric sidebands and richer spectra
- Eliminates "dead spots" where carrier phase locks to 0
- Found in: Buchla 259e, Verbos Complex Osc, many modern Eurorack VCOs
- Software: Arturia Pigments, u-he Diva (TZ mode)

---

## FM Synthesis Algorithms in Code

### Basic 2-Operator FM
```cpp
// Two-operator FM synthesis
float sampleRate = 44100.0f;
float carrierFreq = 440.0f;   // Hz
float modFreq = 440.0f;        // Hz (C:M = 1:1)
float beta = 3.0f;             // modulation index

float carrierPhase = 0.0f;
float modPhase = 0.0f;

float nextSample() {
    float modOut = sinf(modPhase * 2.0f * M_PI);
    float freqDev = beta * modFreq;  // peak frequency deviation
    float output = sinf(carrierPhase * 2.0f * M_PI + beta * sinf(modPhase * 2.0f * M_PI));
    
    carrierPhase += carrierFreq / sampleRate;
    modPhase     += modFreq / sampleRate;
    
    if (carrierPhase >= 1.0f) carrierPhase -= 1.0f;
    if (modPhase >= 1.0f)     modPhase -= 1.0f;
    
    return output;
}
```

### Operator with Envelope
```cpp
struct Operator {
    float freq;        // ratio × pitch or fixed Hz
    float level;       // output amplitude 0–1
    float phase;
    float envOut;      // envelope output 0–1
    
    // 4-stage EG (DX7 style)
    int   egStage;     // 0=attack, 1=decay1, 2=decay2/sustain, 3=release
    float egValue;
    
    float process(float pitchHz, float modulatorIn, float sampleRate) {
        float freq = this->freq * pitchHz;  // ratio mode
        phase += freq / sampleRate;
        if (phase >= 1.0f) phase -= 1.0f;
        return level * envOut * sinf(2.0f * M_PI * phase + modulatorIn);
    }
};
```

---

## Historical Timeline

| Year | Event |
|------|-------|
| 1967 | John Chowning discovers FM synthesis at Stanford |
| 1973 | Chowning licenses algorithm to Yamaha |
| 1974 | Yamaha builds first FM digital synthesizer prototype |
| 1977 | US Patent 4,018,121 granted to Stanford (Chowning) |
| 1980 | Yamaha GS-1 released — first commercial FM synth ($16,000) |
| 1981 | Yamaha CE20/CE25 (consumer FM instruments) |
| 1983 | Yamaha DX7 released ($1,995) — becomes best-selling synth |
| 1984 | Yamaha TX816 (8× DX7 modules), DX9 (4-op budget version) |
| 1985 | Yamaha DX7II / DX7S (microtuning, dual FM) |
| 1987 | AdLib sound card (OPL2 FM) popularizes FM in gaming |
| 1994 | Yamaha VL1 (physical modeling) marks shift away from FM |
| 1995 | FM patent expires — FM available to all manufacturers |
| 1999 | Yamaha FS1R (16 operators, formant shaping + FM) |
| 2003 | Native Instruments FM7 software release |
| 2016 | Korg Volca FM; Yamaha Montage (FM-X, 8 operators) |
| 2018 | Elektron Digitone (4-op FM with sequencer) |
| 2020 | Korg opsix (hybrid 6-op FM) |
| 2023 | Yamaha Montage M (8-op FM-X + analog AN-X engine) |
