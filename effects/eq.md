---
title: "Equalisation: Theory, Types, and Application"
category: "Effects"
tags: [EQ, filter, parametric, graphic, shelving, highpass, lowpass, frequency, resonance]
---

# Equalisation: Theory, Types, and Application

Equalisation (EQ) shapes the frequency content of a signal by boosting or cutting specific frequency bands. It is the foundational tool of audio mixing, sound design, and mastering.

---

## EQ Filter Types

### High-Pass Filter (HPF)
- Removes frequencies **below** the cutoff frequency
- Common slopes: 6, 12, 18, 24 dB/octave (1st through 4th order Butterworth/Linkwitz-Riley)
- 6 dB/oct: gentle, gradual rolloff (1st order RC filter)
- 12 dB/oct: clean rolloff, slight resonance possible at cutoff with high Q
- 24 dB/oct: steep, surgical — used in crossovers
- **Typical use**: remove sub rumble from non-bass instruments; recommended cutoff 80–120 Hz
- High Q HPF adds resonant emphasis at cutoff — useful for character, dangerous on bus

### Low-Pass Filter (LPF)
- Removes frequencies **above** the cutoff frequency
- Same slope options: 6/12/18/24 dB/oct
- **Typical use**: darken a sound, remove harshness; tame digital brightness; crossover networks
- High-pass and low-pass together form a band-pass filter

### Peak / Bell Filter
- Boosts or cuts at a center frequency `f0` with a bandwidth controlled by Q
- Symmetric boost/cut around `f0` on a logarithmic frequency scale
- Transfer function gain at `f0`: `H(f0) = A` where A is the linear gain (10^(dB/20))
- Q = f0 / BW where BW is the -3 dB bandwidth in Hz
- **Bandwidth in octaves**: `BW_oct = 2 * log2( (Q + sqrt(Q^2 + 1)) / Q )` ≈ 1.4427/Q for high Q
- Narrow Q (4–10): surgical notch or boost; wide Q (0.5–1.5): broad tonal shaping

### Low Shelf Filter
- Boosts or cuts **all frequencies below** the shelf frequency
- Gradual transition band around shelf frequency
- Q/slope controls transition steepness
- Use: add or remove body/warmth; classic mixing tool for bass and low-mid control

### High Shelf Filter
- Boosts or cuts **all frequencies above** the shelf frequency
- Use: add air/brightness (boost above 8–12 kHz); tame harsh highs (cut above 10 kHz)
- Gentle high shelf boost is the classic "mastering air" move

### Notch / Band-Reject Filter
- Very narrow, very deep cut at a specific frequency
- Used to remove tonal interference: 50 Hz mains hum (Europe), 60 Hz (US), HVAC noise, resonant room modes
- Q typically 10–50 for notch applications
- Can also be used musically to thin out a frequency before a boost nearby

### Band-Pass Filter
- Passes only a narrow band around the center frequency
- HPF + LPF in series, or a resonant peak near ∞ gain
- Use: telephone effect, CB radio character, resonant sound design

---

## Parametric EQ Parameters

| Parameter | Unit | Description |
|-----------|------|-------------|
| Frequency | Hz   | Center frequency of the filter |
| Gain      | dB   | Amount of boost (+) or cut (−) |
| Q         | —    | Quality factor; controls bandwidth |
| Type      | —    | HPF, LPF, Peak, Shelf, Notch |

### Q and Bandwidth Relationship

```
Q = f0 / BW_Hz

BW_Hz = f0 / Q

BW_octaves ≈ 1.4427 / Q    (approximation for Q > 2)

Exact: BW_oct = log2( (2*Q^2 + 1 + sqrt((2*Q^2 + 1)^2 - 4*Q^4)) / (2*Q^2) )
```

Common Q values and their character:
- Q = 0.5: very wide, about 2–3 octaves
- Q = 0.7: Butterworth (maximally flat, used for shelves)
- Q = 1.0: about 1.5 octaves wide
- Q = 2.0: roughly 0.72 octaves
- Q = 4.0: narrow, about 0.36 octaves
- Q = 10.0: surgical, about 0.14 octaves

---

## Graphic EQ

Fixed-frequency bands with variable gain (typically ±12 dB or ±15 dB per band).

### 31-Band (1/3 Octave) ISO Centre Frequencies (Hz):
```
20, 25, 31.5, 40, 50, 63, 80, 100, 125, 160, 200, 250, 315, 400, 500, 630,
800, 1k, 1.25k, 1.6k, 2k, 2.5k, 3.15k, 4k, 5k, 6.3k, 8k, 10k, 12.5k, 16k, 20k
```

### 10-Band (1 Octave) Bands (Hz):
```
31, 63, 125, 250, 500, 1k, 2k, 4k, 8k, 16k
```

- Bands are **constant-Q or proportional-Q** depending on implementation
- Proportional-Q: bandwidth widens at large gains — common in analogue hardware
- Constant-Q: bandwidth stays fixed regardless of gain amount — more predictable
- Graphic EQ is popular for live sound room correction and speaker management

---

## Minimum Phase vs Linear Phase EQ

### Minimum Phase
- Standard digital EQ implementation (biquad filters, IIR)
- Phase response is coupled to frequency response (Hilbert transform relationship)
- Boosts and cuts cause phase shift, particularly at high gains
- Large boosts cause **pre-ringing** and phase smearing in transient content
- **Latency**: zero latency (sample-accurate)
- Preferred for: tracking, mixing, real-time use, subtle adjustments

### Linear Phase
- Implemented with FIR filters and frequency-domain processing (FFT)
- Phase response is perfectly flat — no phase shift at any frequency
- Identical delays across all frequencies (linear group delay)
- **Pre-ringing artefact**: large cuts create audible ringing before transient events
- **Latency**: significant (block-size dependent, typically 10–50ms)
- Preferred for: mastering, parallel processing, large EQ moves in post-production

### When to Choose Which
- Use minimum phase for mixing — zero latency, transient integrity preserved for moderate boosts
- Use linear phase for mastering, broadcast delivery, large low-frequency shelving
- Never use linear phase on drum transients (pre-ringing is audible)

---

## Filter Implementation: Biquad

The standard second-order IIR (biquad) filter implements all EQ types.

```
y[n] = (b0/a0)*x[n] + (b1/a0)*x[n-1] + (b2/a0)*x[n-2]
                     - (a1/a0)*y[n-1] - (a2/a0)*y[n-2]
```

Coefficients for peak EQ (Audio EQ Cookbook — Robert Bristow-Johnson):

```python
# Peak EQ biquad coefficients
import math

def peak_eq(f0, sample_rate, dB_gain, Q):
    A  = 10 ** (dB_gain / 40.0)      # sqrt of linear gain
    w0 = 2 * math.pi * f0 / sample_rate
    alpha = math.sin(w0) / (2 * Q)
    
    b0 =  1 + alpha * A
    b1 = -2 * math.cos(w0)
    b2 =  1 - alpha * A
    a0 =  1 + alpha / A
    a1 = -2 * math.cos(w0)
    a2 =  1 - alpha / A
    
    return (b0/a0, b1/a0, b2/a0, a1/a0, a2/a0)
```

HPF biquad (12 dB/oct, Butterworth Q=0.7071):

```python
def hpf_biquad(f0, sample_rate, Q=0.7071):
    w0 = 2 * math.pi * f0 / sample_rate
    alpha = math.sin(w0) / (2 * Q)
    
    b0 =  (1 + math.cos(w0)) / 2
    b1 = -(1 + math.cos(w0))
    b2 =  (1 + math.cos(w0)) / 2
    a0 =   1 + alpha
    a1 =  -2 * math.cos(w0)
    a2 =   1 - alpha
    
    return (b0/a0, b1/a0, b2/a0, a1/a0, a2/a0)
```

---

## Frequency Reference Guide

### 20–60 Hz: Sub Bass
- 808 kick body, synthesizer sub bass, pipe organ fundamental
- Felt more than heard; audible only on subwoofers and large drivers
- Below 30 Hz: generally inaudible but consumes headroom
- EQ action: HPF below 25–30 Hz on all sources to remove inaudible rumble; boost 40–60 Hz for 808 weight

### 60–200 Hz: Bass
- Kick drum punch (80–100 Hz), bass guitar fundamentals (40–80 Hz open notes)
- Bass weight and fullness live here
- Cut: remove "boom" or "mud" by cutting 100–200 Hz on bass-heavy instruments
- Boost: add body to thin kick or bass

### 200–500 Hz: Low Midrange
- "Warmth" zone — adds fullness and body to instruments
- Also the "muddiness" zone — boxiness, honkiness in over-processed material
- Guitar and piano body frequencies; vocal chest resonance
- Cut 200–400 Hz on competing instruments to reduce mud in busy mixes
- Boost sparingly (+2–3 dB) to add warmth to thin sounds

### 500 Hz–2 kHz: Midrange
- The most important zone for intelligibility and presence
- Guitar body, snare fundamental, vocal formants, piano character
- Boost: adds forward, present quality
- Cut: recedes a sound in the mix, reduces harshness
- 800 Hz: classic "honk" frequency; cut if nasal

### 2–5 kHz: Upper Midrange (Presence Zone)
- Attack transients, pick/stick attacks, vocal consonants, guitar bite
- Most sensitive zone for human hearing (equal-loudness curves dip here)
- 2–3 kHz boost: adds attack, definition, cuts through mix
- 3–5 kHz: can become harsh and fatiguing with excess; "fizzy" zone
- Cut 3–4 kHz: useful for warming a harsh source without losing presence

### 5–10 kHz: Presence and Sibilance
- Sibilance (s, t, ch sounds): 5–8 kHz
- Brilliance and cut
- Hi-hat shimmer, cymbal detail
- Boost: adds sparkle and definition; cut: darkens/smooths
- De-esser typically targets 5–9 kHz

### 10–20 kHz: Air
- Sheen, openness, shimmer
- Harmonic overtones of cymbals and bowed strings
- Gentle high shelf boost above 10–12 kHz adds "air" without harshness
- 12–16 kHz: classic "air band" for mastering
- Above 16 kHz: extremely delicate; minimal content in most recordings

---

## Practical EQ Settings for Synth Sounds

### 808 Bass
- HPF at 25–30 Hz (remove sub rumble below fundamental)
- Boost 60–80 Hz, +2–4 dB (reinforce fundamental, add weight)
- Cut 150–300 Hz, −2–4 dB (reduce boom/muddiness)
- Optional boost 1.5–3 kHz, +2–3 dB (harmonic presence for small speakers — if distorted/saturated)
- LPF at 8–12 kHz (remove unnecessary high content)

### Lead Synth
- HPF at 80–120 Hz (clean up low end clutter, leave to bass instruments)
- Gentle presence boost 2–4 kHz, +2–3 dB (cut through mix)
- Optional air boost 10–12 kHz, +2 dB (sparkle)
- Cut 200–400 Hz if sounding boxy, −2 dB
- LPF only if sound is too bright/harsh

### Pad / Strings
- HPF at 80–150 Hz (keeps sub clear for bass elements)
- Gentle cut 2–5 kHz, −2–3 dB (prevents pad from competing with lead/vocals)
- Slight cut 300–600 Hz if sounding muddy (−2 dB)
- High shelf boost above 10 kHz if airy quality desired

### Synth Kick
- Boost 50–80 Hz, +3–4 dB (body and weight)
- Boost 3–5 kHz, +2–3 dB (click/attack)
- Cut 150–300 Hz, −3–4 dB (remove mud between body and click)
- HPF at 20–30 Hz (sub rumble)

### Pluck / Arp
- HPF at 100–150 Hz (thin up, leave room for bass)
- Boost 2–4 kHz, +2 dB (articulation and pick attack)
- Cut 800 Hz – 1.2 kHz if sounding honky, −2 dB

### Supersaw / Wide Lead
- HPF at 100 Hz (mono sub clarity)
- Cut 200–400 Hz if muddy (common in dense supersaw patches), −3–5 dB
- Boost 3–5 kHz slightly for presence
- High shelf above 10 kHz to taste

---

## Subtractive vs Additive EQ Philosophy

### Subtractive EQ (Preferred)
- Cut before you boost
- Identify and remove problematic frequencies first
- Cuts are often more transparent than boosts
- Remove the mud to reveal the clarity — don't boost to add clarity

### Additive EQ
- Boost to enhance character
- Use high Q for surgical additions, low Q for broad tonal shaping
- More likely to cause phase problems and inter-channel interactions
- Useful when the source genuinely lacks a frequency (e.g., thin recording needing body)

### Dynamic EQ
- EQ boost/cut is gain-modulated by a threshold (compressor + EQ hybrid)
- Only applies EQ when signal exceeds a level at the target frequency
- More transparent than static EQ for resonance control and de-essing
- Standard tool for mastering and problem frequency management

---

## M/S EQ (Mid-Side)

Process the mid (mono) and side (stereo difference) channels independently.

```
M = (L + R) / 2    # center/mono content
S = (L - R) / 2    # stereo/side content
```

After EQ:

```
L = M + S
R = M - S
```

### Common M/S EQ Moves
- HPF on S channel below 200 Hz → forces bass to mono (mono compatibility)
- High shelf boost on S channel above 8 kHz → widens stereo image
- Cut on S channel mid frequencies → narrows mix for mastering
- Boost on M channel 2–5 kHz → brings vocals/lead forward without affecting width

---

## EQ Interaction with Other Processing

- **EQ before compression**: EQ shapes what the compressor responds to — boosting lows before compression makes compressor more responsive to bass
- **EQ after compression**: shapes the final tone without affecting compression dynamics
- **EQ before saturation**: more harmonics generated from boosted frequencies
- **EQ after saturation**: cleans up harmonic content, removes harshness added by drive

---

## Frequency Masking Context

- Two sources occupying the same frequency band compete for audibility
- EQ notch in one source at the prominent frequency of another = carving space
- Example: vocals sit 800 Hz–4 kHz; guitar fills 400 Hz–3 kHz → gentle notch in guitar at 1–3 kHz
- Low-frequency masking is most problematic (kick/bass overlap at 60–120 Hz)
