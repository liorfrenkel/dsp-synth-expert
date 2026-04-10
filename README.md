# DSP Synth Expert — Claude Code Plugin

A world-class DSP and synthesizer knowledge base packaged as a Claude Code Plugin. Install it once and any Claude Code session becomes an expert sound designer and DSP engineer — fully offline, no internet required.

## What This Plugin Does

Activates a specialist `dsp-expert` agent with deep knowledge of:

- **Filter design**: biquad coefficients, Moog ladder, state-variable, Butterworth
- **Oscillator mathematics**: PolyBLEP, BLIT, FM operator math, wavetable interpolation  
- **Synthesis architectures**: subtractive, FM (DX7), additive, granular, physical modeling
- **Audio effects**: compression, EQ, reverb (Schroeder/Freeverb), chorus/flanger, distortion
- **Psychoacoustics**: Fletcher-Munson curves, masking, stereo perception, M/S processing
- **Sound recipes**: 70s Moog bass, 80s DX7 FM piano, acid 303, Reese bass, trap 808, dubstep growl, techno kick, ambient granular textures

## Installation

### Load for a single session
```bash
claude --plugin-dir /path/to/dsp-synth-expert
```

### Install permanently (when marketplace available)
```
/plugin install /path/to/dsp-synth-expert
```

## Usage

Once installed, Claude Code automatically activates the `dsp-expert` agent. Ask any question:

```
How do I program a classic DX7 electric piano?
Give me the biquad coefficient formula for a low-pass filter at 1kHz with Q=0.707
What are the exact parameter settings for a TB-303 acid squelch?
Explain the Karplus-Strong string synthesis algorithm with code
How does the Schroeder reverb network work?
```

Or use the skill directly:
```
/dsp-synth-expert:sound-design make a Berlin techno kick drum
```

## Knowledge Base Structure

```
dsp/
├── filter_design.md          — Biquad, Moog ladder, SVF, coefficient formulas
├── oscillator_math.md        — PolyBLEP, BLIT, phase accumulator, FM math  
├── wavetable_interpolation.md — Linear/cubic/sinc interpolation, mipmap tables
├── anti_aliasing.md          — Nyquist, oversampling, PolyBLEP algorithm
└── fourier_analysis.md       — FFT, window functions, convolution, STFT

synth_architecture/
├── subtractive_synthesis.md  — VCO/VCF/VCA, harmonics, ADSR, classic synths
├── fm_synthesis.md           — DX7 algorithms (all 32), C:M ratios, operator params
├── additive_synthesis.md     — Fourier series, Hammond organ, partial tracking
├── granular_synthesis.md     — Grain params, time-stretch, Clouds module
├── physical_modeling.md      — Karplus-Strong, waveguide, modal synthesis
└── modulation_systems.md     — Mod matrix, CV/Gate, Eurorack, through-zero FM

effects/
├── compression.md            — Gain computer, VCA/FET/optical/tube, sidechain
├── eq.md                     — Parametric, graphic, frequency reference guide
├── reverb_algorithms.md      — Schroeder, Freeverb, plate, spring, convolution
├── chorus_flanger.md         — Comb filter math, allpass stages, BBD chips
└── distortion_saturation.md  — Transfer functions, harmonic generation, waveshaping

psychoacoustics/
├── fletcher_munson.md        — Equal-loudness curves, phon/sone, mixing implications
├── harmonic_masking.md       — Simultaneous/temporal masking, critical bands, Haas
└── stereo_perception.md      — M/S encoding, mono compatibility, stereo width

genre_recipes/
├── 70s_moog_bass.md          — Minimoog patch with exact OSC/filter/env settings
├── 80s_dx7_fm_piano.md       — E-piano, brass, marimba operator parameters
├── 90s_acid_303.md           — TB-303 squelch, accent/slide mechanics
├── reese_bass.md             — Detuned saws, beating frequency, DnB context
├── trap_808.md               — Sine + pitch env, slide, saturation for audibility
├── dubstep_growl.md          — Serum recipe, FM sweep, formant filter
├── techno_rumble.md          — 909-style kick, transient/punch/body layering
└── ambient_granular.md       — Shimmer reverb, granular freeze, Eno approach
```

## Plugin Structure

```
.claude-plugin/
└── plugin.json               — Plugin manifest
agents/
└── dsp-expert.md             — Expert agent definition
skills/
└── sound-design/
    └── SKILL.md              — Sound design skill
settings.json                 — Activates dsp-expert as default agent
INDEX.md                      — Scannable topic index for the agent
README.md                     — This file
```

## Stats

- **30 knowledge base files**
- **~11,600 lines** of dense technical content
- **Zero internet dependency** after installation
- Covers DSP mathematics through to genre-specific production recipes

## License

MIT
