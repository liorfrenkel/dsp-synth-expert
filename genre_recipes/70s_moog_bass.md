---
title: "70s Moog Bass: Classic Analogue Bass Synthesis"
category: "Genre Recipes"
tags: [Moog, bass, 70s, analogue, subtractive, funk, rock]
---

# 70s Moog Bass: Classic Analogue Bass Synthesis

## Historical Context

The Minimoog Model D (1970) redefined bass in popular music. Before it, bass parts were either played on upright or electric bass guitar. The Minimoog gave keyboardists direct access to massive, harmonically rich bass tones that no electric bass could replicate.

Key artists and records:
- **Rick Wakeman** (Yes) – thick, sustaining bass lines doubled with ensemble
- **Keith Emerson** (ELP) – aggressive, overdriven Moog bass on *Tarkus* (1971)
- **Bernie Worrell** (Parliament-Funkadelic) – rubbery, syncopated funk Moog bass on *Mothership Connection* (1975)
- **Sun Ra** – early explorations of modular Moog in free jazz contexts
- **Giorgio Moroder** – sequenced Moog bass driving Eurodisco (*I Feel Love*, 1977)

The Minimoog Model D shipped with 3 VCOs, a 24dB/oct Moog Ladder Filter (Transistor Ladder), a single contour generator shared between filter and amp, and a dedicated filter EG. Its signal chain is the template for all subtractive bass synthesis.

---

## Signal Chain

```
VCO 1 (Saw 8')  ─┐
VCO 2 (Saw 8')  ─┤─→ Mixer ─→ 24dB Ladder LPF ─→ VCA ─→ Output
VCO 3 (Sq  16') ─┘              ↑            ↑
                            Filter EG      Amp EG
```

The Moog Ladder Filter is the defining element. Its 24dB/oct roll-off (4-pole) is steeper than many competitors (12dB = 2-pole, 18dB = 3-pole). This steep slope combined with high resonance creates the characteristic resonant peak without excessive brightness above it.

---

## Patch Settings

### Oscillators

```
OSC 1:
  Waveform : Sawtooth
  Footprint: 8' (concert pitch / fundamental)
  Fine Tune: 0 cents (reference)
  Volume   : 8/10

OSC 2:
  Waveform : Sawtooth
  Footprint: 8' (unison with OSC 1)
  Fine Tune: +4 cents (beating frequency ≈ 0.25Hz at A2=110Hz → slow throb)
  Volume   : 7/10
  Note     : slight detune thickens sound without full chorus effect

OSC 3:
  Waveform : Square (50% pulse width) or Sawtooth
  Footprint: 16' (one octave below → sub-octave weight)
  Fine Tune: 0 cents
  Volume   : 4/10 (supporting low-end body, not dominating)
```

The 16' sub from OSC 3 fills the 40-60Hz register on lower notes. On the original Minimoog, running OSC 3 as a 16' adds physical weight without clouding the filter response.

### Mixer

```
OSC 1     : 8/10
OSC 2     : 7/10
OSC 3     : 4/10
External  : off
Noise     : off (can add 1-2% for vintage texture)
```

### Filter (24dB/oct Moog Ladder Low-Pass)

```
Cutoff Frequency : 200–400 Hz (depending on register — lower notes need lower cutoff)
Resonance        : 3–4 / 10 (adds presence without self-oscillation)
EG Amount        : 60–70% (the envelope opens the filter significantly on attack)
Keyboard Track   : 50–66% (partial tracking so higher notes don't get too bright)
Contour Type     : Low-Pass
```

The cutoff at rest (sustain position) sits in the 200-400Hz range — dark and thick. The EG Amount of 60-70% means the filter sweeps up substantially on each attack, creating the characteristic "pluck" before settling into the darker sustain body.

### Filter Envelope

```
Attack  :   0 ms (instant — immediate pluck opening)
Decay   :  80–150 ms (fast decay back to sustain level)
Sustain :  30% (filter closes significantly during sustain → dark body)
Release : 100 ms (short, controlled)
```

The combination of zero attack, 80-150ms decay, and 30% sustain is what creates the classic "thwack then thud" texture. The filter opens fully at each note onset then closes down within 150ms.

### Amp Envelope

```
Attack  :   0 ms (instant onset)
Decay   : 500 ms (gradual fade if note is short)
Sustain :  80% (high sustain for held bass notes)
Release : 200 ms (medium release — notes don't cut off abruptly)
```

### Glide / Portamento

```
Portamento Time: 30–80 ms
Mode          : Legato only (on original Minimoog, glide only applies when key is held)
```

Glide at 30-80ms creates smooth melodic connections between consecutive notes in legato playing. Funk basslines typically use short glide (30ms) for expressive slides; rock/progressive parts may use 60-80ms for more pronounced portamento.

---

## Processing Chain

### Overdrive / Saturation

The original Minimoog output would clip the input stages of mixing consoles, contributing to the signature warm distortion. Replicate this:

```
Saturation Type: Tube or Tape (soft clip, not hard clip)
Drive Level    : +3 to +6 dB above unity (subtle — adds harmonics, not grunge)
2nd Harmonic   : boosted by soft saturation → adds "octave" warmth
3rd Harmonic   : minimal at +3dB, present at +6dB → adds bite
```

### Compression

```
Type   : VCA or Optical (fast, smooth)
Ratio  : 4:1
Attack : 1–5 ms (fast — catches the transient, slightly blunts it)
Release: 100–150 ms
Gain   : +3–6 dB makeup
Threshold: set so 4–6 dB GR on peaks
```

Compression glues the detuned oscillators and tames the filter envelope transient. The 4:1 ratio is aggressive enough to control dynamics without killing the envelope movement.

### EQ (optional)

```
High-pass : 40 Hz (remove sub-sub rumble)
Low shelf : +2 dB at 80 Hz (fundamental weight)
Mid cut   : -2 dB at 300-500 Hz (reduce nasal boxiness)
Presence  : +1 dB at 1-2 kHz (articulation, not excessive)
```

---

## Modern Recreation

### Software Synths

**Arturia Mini V3** — most accurate software emulation of Minimoog Model D. Use the built-in oscillator settings above directly. The drive knob replicates the input stage saturation.

**Moog Model D app (iOS/macOS)** — official Moog Software recreation. Identical panel layout. Suitable for exact A/B comparison.

**Native Instruments Monark** — physically modelled. Good for nuanced expression. Map mod wheel to filter cutoff for real-time filter "wah."

**u-he Repro-1** — based on Pro-One but similar architecture. Use for variations.

### Parameter Mapping (DAW Recreation)

```
Any subtractive synth with:
- 3 independent oscillators with footprint/octave switching
- 4-pole (24dB/oct) low-pass filter
- Independent filter EG + amp EG
- Portamento (glide)
```

---

## Variations

### Funk "Wah" Effect

```
Resonance        : 6–7/10 (higher — approaching self-oscillation)
EG Amount        : 80%
Filter Env Decay : 60–100 ms (fast snap)
Cutoff at rest   : 150–200 Hz (starts very dark)
Add LFO          : Sine, 0.5–2 Hz, depth 15% on cutoff → auto-wah
```

This mimics the envelope-filter wah pedal effect used in funk rhythm bass.

### Blues Pitch Slides

```
Portamento: 100–200 ms (exaggerated for expressive slides)
Pitch Bend: ±2 semitones (manual bend at phrase ends)
Playing style: play target note, bend down 1 semitone then release — creates blues inflection
```

### Sub-Bass Variant (Disco/Electronic)

```
OSC 1: Sine, 16' (pure sub fundamental)
OSC 2: Sawtooth, 8' (harmonics)
OSC 3: off
Filter Cutoff: 300 Hz
EG Amount: 40% (less pluck)
Amp Attack: 5ms (very slight softening for sine)
```

Cleaner, with more sub-bass extension. Used for disco and early electronic dance music.

### Overdriven Rock Bass

```
OSC 3 Volume: 7/10 (match OSC 1 level → more aggressive)
Filter Resonance: 5/10 (more pronounced peak)
Post-filter: hard clip saturation at +6–9 dB
EQ: boost 800 Hz–1.5 kHz +3 dB (more midrange grind)
```

---

## Performance Tips

- **Legato playing technique**: hold previous note while pressing next for glide to engage (on Minimoog, glide only triggers when keys overlap in legato)
- **Accent notes**: increase velocity → velocity-to-VCA mapping adds dynamics
- **Detuning stability**: in live use, VCOs on hardware drift with temperature — small amounts of "organic" drift are part of the vintage character
- **Register considerations**:
  - Below C2 (65Hz): reduce filter cutoff to 150-250Hz, EG amount 40-50%
  - C2–C3 (65–130Hz): full settings as above
  - Above C3: increase cutoff to 400-600Hz, reduce OSC 3 sub weight

---

## Reference Recordings

| Track | Artist | Year | Notes |
|-------|--------|------|-------|
| Tarkus | Emerson, Lake & Palmer | 1971 | Aggressive overdriven Moog bass |
| Mothership Connection | Parliament | 1975 | Rubbery funk Moog bass (Bernie Worrell) |
| I Feel Love | Donna Summer / Moroder | 1977 | Sequenced monophonic Moog bassline |
| Superstition | Stevie Wonder | 1972 | Clavinet + Moog bass combination |
| Lucky Man | ELP | 1970 | Classic Moog solo AND bass |

---

## Technical Notes

### Why 24dB/oct Matters

The Moog ladder filter's 4-pole design provides 24dB of attenuation per octave above the cutoff. Compare:

```
Frequency above cutoff | 12dB filter | 24dB filter
+1 octave              | -12 dB      | -24 dB
+2 octaves             | -24 dB      | -48 dB
```

This steep roll-off means harmonics are dramatically attenuated above the cutoff, giving the characteristic "sealed" sound that distinguishes Moog from Oberheim (12dB) or Roland (18dB) architectures.

### Detuning and Beat Frequency

At A2 (110 Hz):
```
4 cents = 110 Hz × (2^(4/1200) - 1) ≈ 0.25 Hz beat frequency
```

This is extremely slow beating — barely perceptible as oscillation, more as a static thickening. For a more obvious chorus effect, increase to 8-12 cents (0.5–0.75 Hz beat at A2).
