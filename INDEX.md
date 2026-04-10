# DSP Synth Expert — Knowledge Base Index

> **Usage:** Read this file first. Find the topic you need, then read the linked file.
> Agent: use `Read` on the listed path to get full technical details.

---

## Synth-Specific References

Surge XT: all oscillator types, filter types, envelope time scale, LFO shapes, FX types, waveshaper, modulation routing, parameter XML names
[synth_specific/surge_xt.md](synth_specific/surge_xt.md)

Surge XT factory preset analysis: what makes each category tick, typical envelope/filter/FX patterns derived from parsing 343 real patches
[synth_specific/surge_xt_presets.md](synth_specific/surge_xt_presets.md)

---

## DSP & Mathematics

Biquad filter design: LPF/HPF/BPF/Notch/Allpass/Shelf — coefficient formulas, difference equations, Q/frequency/sampleRate math
[dsp/filter_design.md](dsp/filter_design.md)

Moog ladder filter, state-variable filter (SVF), Chamberlin filter, resonance, self-oscillation
[dsp/filter_design.md](dsp/filter_design.md)

Oscillator mathematics: phase accumulator, PolyBLEP, BLIT, FM operator formula, hard sync
[dsp/oscillator_math.md](dsp/oscillator_math.md)

Wavetable synthesis: linear/cubic/sinc interpolation formulas, mipmap anti-aliasing, table size selection
[dsp/wavetable_interpolation.md](dsp/wavetable_interpolation.md)

Anti-aliasing in digital synthesis: Nyquist, aliasing, oversampling, PolyBLEP algorithm with code
[dsp/anti_aliasing.md](dsp/anti_aliasing.md)

Fourier analysis: DFT/FFT formulas, window functions, convolution theorem, phase vocoder, STFT
[dsp/fourier_analysis.md](dsp/fourier_analysis.md)

---

## Synthesizer Architecture

Subtractive synthesis: VCO→VCF→VCA signal flow, waveform harmonics, ADSR math, classic synths (Minimoog, Juno-106, Prophet-5)
[synth_architecture/subtractive_synthesis.md](synth_architecture/subtractive_synthesis.md)

FM synthesis: DX7 all 32 algorithms, modulation index β, C:M ratio table, sideband frequencies, Bessel functions, DX7 patch recipes
[synth_architecture/fm_synthesis.md](synth_architecture/fm_synthesis.md)

Additive synthesis: Fourier series, harmonic amplitude tables, Hammond organ drawbar model (footage values, registrations)
[synth_architecture/additive_synthesis.md](synth_architecture/additive_synthesis.md)

Granular synthesis: grain parameters (size/density/scatter/envelope), time-stretching, pitch-shifting, Mutable Instruments Clouds
[synth_architecture/granular_synthesis.md](synth_architecture/granular_synthesis.md)

Physical modeling: Karplus-Strong algorithm (with code), waveguide synthesis, modal synthesis, vocal tract, flute/clarinet/brass models
[synth_architecture/physical_modeling.md](synth_architecture/physical_modeling.md)

Modulation systems: modulation matrix, CV/Gate, Eurorack standard, MIDI CC, through-zero FM, ring mod, waveshaping, wavefolding
[synth_architecture/modulation_systems.md](synth_architecture/modulation_systems.md)

---

## Audio Effects

Compression: gain computer formula, attack/release ballistics, VCA/optical/FET/tube, sidechain, multiband, per-instrument settings
[effects/compression.md](effects/compression.md)

EQ: parametric/graphic/shelf/HPF/LPF, Q formula, minimum-phase vs linear-phase, frequency reference guide per instrument
[effects/eq.md](effects/eq.md)

Reverb algorithms: Schroeder network (delay times, feedback), Freeverb parameters, plate/spring models, convolution IR, RT60
[effects/reverb_algorithms.md](effects/reverb_algorithms.md)

Chorus/Flanger/Phaser: comb filter math, allpass stage count, BBD chips, Juno-106 chorus, Roland Dimension D
[effects/chorus_flanger.md](effects/chorus_flanger.md)

Distortion and saturation: transfer functions (tanh, hard clip, foldback), harmonic generation by circuit type, bit crushing, oversampling
[effects/distortion_saturation.md](effects/distortion_saturation.md)

---

## Psychoacoustics

Fletcher-Munson equal-loudness curves: phon/sone scales, ISO 226:2003, practical mixing implications, A-weighting
[psychoacoustics/fletcher_munson.md](psychoacoustics/fletcher_munson.md)

Auditory masking: simultaneous/temporal masking, critical bands (Bark scale), masking thresholds, Haas effect
[psychoacoustics/harmonic_masking.md](psychoacoustics/harmonic_masking.md)

Stereo perception: M/S encoding/decoding, mono compatibility rules, stereo width techniques, panning laws, HRTF
[psychoacoustics/stereo_perception.md](psychoacoustics/stereo_perception.md)

---

## Sound Design Recipes by Era & Genre

### Classic Analogue (1970s–1980s)

How to synthesize a classic 70s Moog bass (Minimoog patch with exact OSC/filter/envelope settings)
[genre_recipes/70s_moog_bass.md](genre_recipes/70s_moog_bass.md)

How to program the DX7 Rhodes electric piano, FM brass, marimba, and bass (operator ratios, levels, envelopes)
[genre_recipes/80s_dx7_fm_piano.md](genre_recipes/80s_dx7_fm_piano.md)

### Electronic Bass Sounds (1980s–2010s)

How to create acid 303 bass: TB-303 parameter settings for classic squelch, accent/slide mechanics
[genre_recipes/90s_acid_303.md](genre_recipes/90s_acid_303.md)

How to create a Reese bass: detuned sawtooth technique, beating frequency calculation, DnB context
[genre_recipes/reese_bass.md](genre_recipes/reese_bass.md)

How to synthesize a trap 808 sub bass: sine with pitch envelope, slide portamento, saturation for audibility
[genre_recipes/trap_808.md](genre_recipes/trap_808.md)

How to design a dubstep/neuro growl bass: FM sweep, Serum wavetable recipe, formant filter, Skrillex style
[genre_recipes/dubstep_growl.md](genre_recipes/dubstep_growl.md)

### Modern Electronic (2010s–Present)

How to design a modern Berlin techno kick and rumble: 909-style synthesis, layering transient/punch/body
[genre_recipes/techno_rumble.md](genre_recipes/techno_rumble.md)

How to create ambient granular textures and drone pads: shimmer reverb, granular freeze, Eno-style approach
[genre_recipes/ambient_granular.md](genre_recipes/ambient_granular.md)

---

## Quick Reference

| Sound | Primary File | Key Parameters |
|-------|-------------|----------------|
| Moog bass | [genre_recipes/70s_moog_bass.md](genre_recipes/70s_moog_bass.md) | Saw+Sub OSCs, 24dB LP, fast filter env |
| DX7 e-piano | [genre_recipes/80s_dx7_fm_piano.md](genre_recipes/80s_dx7_fm_piano.md) | Algorithm 5, Op2 ratio 14, velocity→brightness |
| Acid 303 | [genre_recipes/90s_acid_303.md](genre_recipes/90s_acid_303.md) | Saw, 75-95% resonance, accent+slide |
| Reese bass | [genre_recipes/reese_bass.md](genre_recipes/reese_bass.md) | 2× detuned saws, -12 to -18 cents |
| Trap 808 | [genre_recipes/trap_808.md](genre_recipes/trap_808.md) | Sine, pitch env drop, long decay |
| Dubstep growl | [genre_recipes/dubstep_growl.md](genre_recipes/dubstep_growl.md) | Wavetable+LFO or FM β sweep |
| Techno kick | [genre_recipes/techno_rumble.md](genre_recipes/techno_rumble.md) | Sine pitch env 180→55Hz, 30ms |
| Ambient pad | [genre_recipes/ambient_granular.md](genre_recipes/ambient_granular.md) | Detuned OSCs, 300ms attack, long reverb |
| Biquad LPF | [dsp/filter_design.md](dsp/filter_design.md) | w0=2πf/Fs, α=sin(w0)/(2Q), 5 coefficients |
| Moog ladder | [dsp/filter_design.md](dsp/filter_design.md) | 4-pole cascade, feedback coeff g, tanh NL |
| PolyBLEP | [dsp/anti_aliasing.md](dsp/anti_aliasing.md) | Residual correction at discontinuity |
| Reverb | [effects/reverb_algorithms.md](effects/reverb_algorithms.md) | 8 combs + 4 allpass (Freeverb) |
| Compression | [effects/compression.md](effects/compression.md) | GR = (1-1/R)×(thresh-input) |
