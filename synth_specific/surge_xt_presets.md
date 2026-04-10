---
title: "Surge XT Factory Presets — High-Level Analysis"
category: "Synth Specific"
tags: [Surge XT, presets, factory patches, sound design patterns, analysis]
---

# Surge XT Factory Presets — Analysis

This file is derived from parsing **343 factory `.fxp` patches** directly from the Surge XT installation. All statistics are computed from actual patch data, not approximations.

---

## Library Overview

| Category | Count | Defining Character |
|----------|-------|-------------------|
| Leads | 99 | Punchy, chorused, waveshaped Classic oscillator |
| Polysynths | 73 | Chorus on every patch, medium attack, LP24/LP-Moog |
| Plucks | 63 | Fast attack + long decay, high filter env, chorus |
| Basses | 42 | Tight, fast, Moog/LP24 filter, portamento 59% |
| Pads | 34 | Long attack (avg 1.6s), always LFO, heavy chorus |
| Sequences | 33 | Instant attack/release, Noise-Stepped LFO, always chorus |
| FX | 17 | Experimental textures, longer attacks, distortion |
| Keys | 7 | Fast attack, Noise-Smooth LFO, always rotary |
| Brass | 6 | Slow swell attack, high filter env mod, chorus+dist |
| Percussion | 8 | Instant attack, Noise-Smooth LFO, HP filters |
| MPE | 3 | LP-Moog filter, FM oscillators, expressive design |
| Splits | 3 | Mono mode, scene-based splits |

**Total**: 343 patches by a single author (Claes Johanson, Surge XT lead developer)

---

## LEADS — Design Patterns

**99 patches. The most important category to understand.**

### The formula that works
1. Classic oscillator (96% of all osc slots)
2. LP-Moog or LP24 filter (66% combined)
3. Waveshaper — always (Tanh 15%, Asym 13%, Digital 9%, Hard 8%)
4. LFO0 sine modulating pitch (vibrato) — 87/99 patches
5. Chorus FX — 84/99 patches (85%)

### Envelope signature
- Attack: **mostly 4ms** (instant), some up to 1796ms for pad-like leads
- Release: avg 167ms, range 4ms–2921ms
- Portamento: **82% have it enabled** — often subtle, for expressiveness

### Filter behaviour
- Resonance avg: **0.50** (moderate — enough to add character)
- Filter env mod avg: **16.7 semitones** — significant filter sweep on attack
- Common filter routing: Filter1 open → Filter2 adds character

### What makes a Surge lead sound like a Surge lead
- The Rotary speaker FX on 35% — gives a subtle 3D rotation the ear perceives as "alive"
- Waveshaper drive into an LP-Moog produces the characteristic slightly gritty, non-linear sound
- 32% have analog drift — imperceptible consciously but felt as "organic"
- High filter env mod with very fast attack = "snap" that defines the transient character
- The Distortion FX (16%) is used at low drive for warmth, not obvious grit

### Standout patches to study
- **Classic Lead 1**: Init-style saw, LP-Moog, Tanh waveshaper, LFO vibrato, send to Chorus — the archetype
- **Crisp PWM**: Pulse width lead with PWM via LFO, LP24, no waveshaper
- **FM Is Growing On Me**: FM3 oscillator lead — rare FM lead in the library

---

## PADS — Design Patterns

**34 patches.**

### The formula
1. Classic oscillator almost exclusively
2. LP24 or LP-Moog
3. **Very long attack**: avg 1600ms (range 4ms–8020ms)
4. **Very long release**: avg 2214ms (range 31ms–4733ms)
5. **Every single pad has active LFO** (34/34)

### What defines a pad vs a polysynth
The attack time is everything. A polysynth at 100ms becomes a pad at 1000ms+. The factory library is clear on this — pads average 1600ms attack; polysynths average 113ms.

### Surprising finding: Distortion on pads (59%)
20 out of 34 pad patches use the Distortion FX. Not for grit — for **weight and harmonic density**. Subtle distortion on a slow pad makes the harmonics bloom and occupy space in the mix. Typically combined with Chorus (36/34 FX slots).

### LFO usage on pads
- Sine (16) and Triangle (10) dominate
- Magnitude is typically low (0.15–0.25) — subtle movement, not vibrato
- Rate is slow (rate value around -3 to -2, corresponding to 0.06–0.12Hz)
- 4 patches use Noise-Smooth for organic, non-periodic movement

### Drift: 41% of pads have it
More than leads (32%). Makes sense — a pad held for 4+ seconds with perfectly stable oscillators sounds synthetic. Drift of 0.1–0.2 makes slow pads feel like real strings or choirs.

---

## BASSES — Design Patterns

**42 patches.**

### The formula
1. Classic oscillator (85% of slots), Modern (9%), FM2 (5%)
2. LP-Moog or LP24 (equal — 31% each)
3. Very fast attack: avg **5.4ms**
4. Medium-short release: avg **63ms**, sustain 0.80
5. Portamento enabled on **59%** — slides between notes
6. LFO0 active on 41/42 — almost always Sine

### What the bass LFO is actually doing
All basses have an active LFO but FX send is low (10/42 send to FX1). The LFO is modulating the **oscillator pitch slightly** (subtle vibrato) or **filter cutoff** (subtle movement) — not vibrato you hear consciously, but that makes the bass feel alive rather than locked.

### Filter env mod: 15.6 semitones average
Significant. The filter envelope is working hard on basses — it's what creates the "thump" on attack. A closed filter (cutoff -30 to -40) + high env mod (+15–25 semitones) = punch without sustained brightness.

### FM basses (FM2 oscillator)
5 patches use FM2. These produce the characteristic "plucked" or "slap" FM bass character (DX7 bass style). The FM2 operator ratio and modulation index completely define the timbre — much more so than the filter.

### Waveshaper on basses
15 patches (36%) use waveshaper. Asym and Digital most common. Purpose: adds 2nd and odd harmonics to make the bass cut through on small speakers and remain audible when the fundamental is below hearing threshold.

---

## PLUCKS — Design Patterns

**63 patches. The most technically interesting category.**

### The formula
1. Fast attack: avg **9ms**
2. Long release: avg **1450ms** with sustain **0.50**
3. High filter env mod: avg **19.9 semitones** (highest of any category)
4. Drift: **42%** (highest of any category alongside Pads)
5. Chorus send: 59/63 patches (94%)

### The sustain=0.50 signature
Half sustain level means the patch decays naturally even while holding a note. The amp envelope mimics a physical string: instant attack, long decay to silence. The "pluck" is in the filter envelope doing a massive sweep (+20 semitones) in the first 50–200ms then closing.

### Why plucks need heavy waveshaping (19%)
A pluck needs attack transients with rich harmonics that then decay. Waveshaping (Asym, Tanh) adds harmonics that feed into the filter's sweep — you hear the harmonics bloom and then disappear as the filter closes. Without it, the pluck sounds dull and synthetic.

### Drift is essential for plucks
42% have drift — highest category. A plucked string has micro-intonation variations. Even 0.1–0.15 drift makes a synth pluck feel organic. With 0 drift, it sounds like a MIDI note triggering a sample.

---

## POLYSYNTHS — Design Patterns

**73 patches.**

### The defining stat: Chorus on 100% of patches
Every single polysynth patch sends to the Chorus FX. This is the most reliable pattern in the entire library. A polysynth without chorus is not a polysynth in this library — it becomes something else.

### The formula
1. Classic oscillator (94% of slots)
2. LP24 (44%) or LP-Moog (19%)
3. Medium attack: avg **113ms** (range 4ms–5194ms)
4. Longer release: avg **683ms**
5. Chorus (100%) + Distortion (40%) + Rotary (32%)
6. LFO0 Sine (67%) or Noise-Smooth (13%)

### Distortion at 40%
Same pattern as pads — subtle distortion for harmonic weight. Not obvious overdrive. The combination Chorus + Distortion + Rotary is the signature Surge XT thick polysynth sound.

### What separates polysynths from leads
- Attack is longer (113ms vs 52ms for leads)
- Resonance similar (0.40 vs 0.50)
- But filter env mod is similar (17.8 vs 16.7 semitones)
- The key difference is context: polysynths are played in chords, leads are melodic

---

## SEQUENCES — Design Patterns

**33 patches. The most distinctive LFO pattern in the library.**

### The formula
1. Instant attack + instant release: avg **4ms attack, 31ms release**
2. Sustain 0.70
3. **LFO shape: Noise-Stepped in 73% of patches** (24/33)
4. High filter env mod: avg **23.5 semitones** (highest category)
5. Heavy waveshaping: Asym (18%), Hard (15%)
6. Chorus on 100% of patches

### Why Noise-Stepped LFO?
Sequences are rhythmic patches. The Noise-Stepped LFO creates **random stepped modulation** locked to tempo (when rate synced) or free-running. Modulating filter cutoff with Noise-Stepped creates the characteristic randomly-varying filter on each step — making an arpeggiated sequence feel alive rather than mechanical.

### Filter env mod at 23.5 semitones
Sequences are designed to be punchy and rhythmic. The filter opens dramatically on each note (23 semitones = almost 2 octaves of cutoff sweep) and then closes. This is the "percussive filter" technique — more envelope modulation than any other category.

---

## KEYS — Design Patterns

**7 patches.**

### Dominant finding: Noise-Smooth LFO on everything (4/7)
Unlike other categories that use Sine for smooth vibrato, Keys patches use Noise-Smooth. This mimics the irregular timbral variation of acoustic keys (piano hammers, organ air). A piano or organ doesn't vibrato regularly — it has organic variation.

### Multiple Rotary effects
The House Organ patch uses 3 Rotary effects in series — unusual and instructive. The Rotary FX is the definitive organ effect. At fast speed it produces vibrato/tremolo; at slow speed it produces the Leslie slow rotation.

### Fast attack, partial sustain
- Attack: avg 7ms (instant)
- Sustain: avg 0.60 (keys decay, they don't sustain forever)
- Release: avg 366ms (keys ring a little after release)

---

## BRASS — Design Patterns

**6 patches. Highly consistent.**

### Formula: always Classic, always Chorus+Distortion, always slow attack
- 100% Classic oscillator
- 100% use both Chorus AND Distortion FX
- Attack avg: **186ms** — the defining characteristic of brass is the swell-in
- Filter env mod: **22.8 semitones** — huge filter sweep on attack mimics brass breath
- Resonance low: 0.20 — brass doesn't self-resonate, it blooms

### The filter env + slow attack combination
Brass = closed filter + slow attack + massive filter env mod. The filter opens AS the amplitude rises. This mimics how a brass player increases air pressure and the instrument "opens up". No other category replicates this combination.

---

## PERCUSSION — Design Patterns

**8 patches.**

### HP filter dominates (not LP)
Percussion patches use HP12 (3 patches) and Comb+ — unlike every other category which uses LP. The high-pass removes the long low-frequency tail so percussion doesn't accumulate sub energy.

### Noise-Smooth LFO on everything (7/8)
Not for modulation — the LFO is likely modulating the filter cutoff subtly to create timbral variation across a drum machine pattern. Each hit sounds slightly different.

### FM for percussion (12%)
Higher FM usage than most categories (only Basses match at 9%). FM2 produces the characteristic metallic "clang" of electronic toms and percussion. Modulation index determines how metallic vs. tonal the sound is.

---

## Cross-Category Observations

### The Universal Rules (observed from data)
1. **Classic oscillator is 80%+ across all categories** — Surge's Classic is versatile enough for everything except dedicated FM or wavetable sounds
2. **LFO0 is active on 87–100% of patches** — nothing is ever completely static
3. **Chorus FX appears on Polysynths (100%), Sequences (100%), Brass (100%), Plucks (94%), Leads (85%)** — use it by default
4. **Rotary is not a niche effect** — 35% of leads, 33% of polysynths, 71% of keys use it
5. **Distortion = weight, not grit** — used at subtle settings across pads, plucks, polysynths for harmonic density

### When to use which filter
- **LP-Moog**: warm, vintage, slightly non-linear character — leads, basses, pads
- **LP24**: clean punch, precise resonance — polysynths, plucks, sequences
- **LP-Legacy**: very vintage, slightly different character — brass, older-style pads
- **HP**: only in percussion and winds
- **Comb+**: metallic resonator effect — sequences, wind simulations

### Waveshaper strategy
- **Tanh**: use first — smooth, musical saturation for leads
- **Asym**: for bass and plucks — adds 2nd harmonics (warmth + cut)
- **Digital**: modern/tech character, slight 12-bit grit
- **Hard**: aggressive leads, sequences — strong odd harmonics
- **None**: only when you explicitly want clean/digital character
