# DSP Synth Expert — Knowledge Base Index

> **Usage:** Read this file first. Find the topic you need, then read the linked file.
> Agent: use `Read` on the listed path to get full technical details.

---

## DSP & Mathematics

Biquad filter design: LPF/HPF/BPF/Notch/Allpass/Shelf — coefficient formulas, difference equations, Q/frequency/sampleRate math
[Read file: ./dsp/filter_design.md]

Moog ladder filter, state-variable filter (SVF), Chamberlin filter, resonance, self-oscillation
[Read file: ./dsp/filter_design.md]

Oscillator mathematics: phase accumulator, PolyBLEP, BLIT, FM operator formula, hard sync
[Read file: ./dsp/oscillator_math.md]

Wavetable synthesis: linear/cubic/sinc interpolation formulas, mipmap anti-aliasing, table size selection
[Read file: ./dsp/wavetable_interpolation.md]

Anti-aliasing in digital synthesis: Nyquist, aliasing, oversampling, PolyBLEP algorithm with code
[Read file: ./dsp/anti_aliasing.md]

Fourier analysis: DFT/FFT formulas, window functions, convolution theorem, phase vocoder, STFT
[Read file: ./dsp/fourier_analysis.md]

---

## Synthesizer Architecture

Subtractive synthesis: VCO→VCF→VCA signal flow, waveform harmonics, ADSR math, classic synths (Minimoog, Juno-106, Prophet-5)
[Read file: ./synth_architecture/subtractive_synthesis.md]

FM synthesis: DX7 all 32 algorithms, modulation index β, C:M ratio table, sideband frequencies, Bessel functions, DX7 patch recipes
[Read file: ./synth_architecture/fm_synthesis.md]

Additive synthesis: Fourier series, harmonic amplitude tables, Hammond organ drawbar model (footage values, registrations)
[Read file: ./synth_architecture/additive_synthesis.md]

Granular synthesis: grain parameters (size/density/scatter/envelope), time-stretching, pitch-shifting, Mutable Instruments Clouds
[Read file: ./synth_architecture/granular_synthesis.md]

Physical modeling: Karplus-Strong algorithm (with code), waveguide synthesis, modal synthesis, vocal tract, flute/clarinet/brass models
[Read file: ./synth_architecture/physical_modeling.md]

Modulation systems: modulation matrix, CV/Gate, Eurorack standard, MIDI CC, through-zero FM, ring mod, waveshaping, wavefolding
[Read file: ./synth_architecture/modulation_systems.md]

---

## Audio Effects

Compression: gain computer formula, attack/release ballistics, VCA/optical/FET/tube, sidechain, multiband, per-instrument settings
[Read file: ./effects/compression.md]

EQ: parametric/graphic/shelf/HPF/LPF, Q formula, minimum-phase vs linear-phase, frequency reference guide per instrument
[Read file: ./effects/eq.md]

Reverb algorithms: Schroeder network (delay times, feedback), Freeverb parameters, plate/spring models, convolution IR, RT60
[Read file: ./effects/reverb_algorithms.md]

Chorus/Flanger/Phaser: comb filter math, allpass stage count, BBD chips, Juno-106 chorus, Roland Dimension D
[Read file: ./effects/chorus_flanger.md]

Distortion and saturation: transfer functions (tanh, hard clip, foldback), harmonic generation by circuit type, bit crushing, oversampling
[Read file: ./effects/distortion_saturation.md]

---

## Psychoacoustics

Fletcher-Munson equal-loudness curves: phon/sone scales, ISO 226:2003, practical mixing implications, A-weighting
[Read file: ./psychoacoustics/fletcher_munson.md]

Auditory masking: simultaneous/temporal masking, critical bands (Bark scale), masking thresholds, Haas effect
[Read file: ./psychoacoustics/harmonic_masking.md]

Stereo perception: M/S encoding/decoding, mono compatibility rules, stereo width techniques, panning laws, HRTF
[Read file: ./psychoacoustics/stereo_perception.md]

---

## Sound Design Recipes by Era & Genre

### Classic Analogue (1970s–1980s)

How to synthesize a classic 70s Moog bass (Minimoog patch with exact OSC/filter/envelope settings)
[Read file: ./genre_recipes/70s_moog_bass.md]

How to program the DX7 Rhodes electric piano, FM brass, marimba, and bass (operator ratios, levels, envelopes)
[Read file: ./genre_recipes/80s_dx7_fm_piano.md]

### Electronic Bass Sounds (1980s–2010s)

How to create acid 303 bass: TB-303 parameter settings for classic squelch, accent/slide mechanics
[Read file: ./genre_recipes/90s_acid_303.md]

How to create a Reese bass: detuned sawtooth technique, beating frequency calculation, DnB context
[Read file: ./genre_recipes/reese_bass.md]

How to synthesize a trap 808 sub bass: sine with pitch envelope, slide portamento, saturation for audibility
[Read file: ./genre_recipes/trap_808.md]

How to design a dubstep/neuro growl bass: FM sweep, Serum wavetable recipe, formant filter, Skrillex style
[Read file: ./genre_recipes/dubstep_growl.md]

### Modern Electronic (2010s–Present)

How to design a modern Berlin techno kick and rumble: 909-style synthesis, layering transient/punch/body
[Read file: ./genre_recipes/techno_rumble.md]

How to create ambient granular textures and drone pads: shimmer reverb, granular freeze, Eno-style approach
[Read file: ./genre_recipes/ambient_granular.md]

---

## Quick Reference

| Sound | Primary File | Key Parameters |
|-------|-------------|----------------|
| Moog bass | genre_recipes/70s_moog_bass.md | Saw+Sub OSCs, 24dB LP, fast filter env |
| DX7 e-piano | genre_recipes/80s_dx7_fm_piano.md | Algorithm 5, Op2 ratio 14, velocity→brightness |
| Acid 303 | genre_recipes/90s_acid_303.md | Saw, 75-95% resonance, accent+slide |
| Reese bass | genre_recipes/reese_bass.md | 2× detuned saws, -12 to -18 cents |
| Trap 808 | genre_recipes/trap_808.md | Sine, pitch env drop, long decay |
| Dubstep growl | genre_recipes/dubstep_growl.md | Wavetable+LFO or FM β sweep |
| Techno kick | genre_recipes/techno_rumble.md | Sine pitch env 180→55Hz, 30ms |
| Ambient pad | genre_recipes/ambient_granular.md | Detuned OSCs, 300ms attack, long reverb |
| Biquad LPF | dsp/filter_design.md | w0=2πf/Fs, α=sin(w0)/(2Q), 5 coefficients |
| Moog ladder | dsp/filter_design.md | 4-pole cascade, feedback coeff g, tanh NL |
| PolyBLEP | dsp/anti_aliasing.md | Residual correction at discontinuity |
| Reverb | effects/reverb_algorithms.md | 8 combs + 4 allpass (Freeverb) |
| Compression | effects/compression.md | GR = (1-1/R)×(thresh-input) |
