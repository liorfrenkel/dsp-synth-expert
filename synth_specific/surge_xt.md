---
title: "Surge XT — Complete Parameter Reference"
category: "Synth Specific"
tags: [Surge XT, synthesizer, parameters, oscillators, filters, modulation, FX]
---

# Surge XT — Complete Parameter Reference

Surge XT is a free, open-source hybrid synthesizer. Each patch has two independent **Scenes (A/B)** — each a full synthesis engine — that can run in Split, Layer, or Single mode. This file covers every parameter the agent needs to construct or describe a Surge XT patch precisely.

---

## Architecture Overview

```
Scene A                          Scene B
  3× Oscillators                   3× Oscillators
  Noise Generator                  Noise Generator
  Ring Mod (Osc1×Osc2, Osc2×Osc3) Ring Mod
  Mixer (levels per source)        Mixer
  Waveshaper                       Waveshaper
  Filter 1 → Filter 2 (serial/parallel/various)
  Amp Envelope (Env 1)
  Filter Envelope (Env 2)
  6× LFOs / Modulators (per-voice or scene-level)
  Send → Global FX Chain

Global FX Chain
  8 FX slots (FX1–FX8) — any order
  Scene A/B send to FX1/FX2 independently
  Volume, Character (global tone shaping)
```

---

## Scene Modes

| Value | Mode | Description |
|-------|------|-------------|
| Poly | Polyphonic | Multiple simultaneous voices |
| Mono | Monophonic | Single voice, new note steals |
| Mono-Always | Mono legato | Retrigger envelope even in legato |
| Mono-ST | Mono single trigger | Don't retrigger in legato |
| Latch | Latch | Notes latch on/off |

**Polyphony limit**: `polylimit` (default 16 voices)

---

## Oscillators

Each scene has 3 oscillators. Set type with `a_osc1_type` (integer):

| Type # | Name | Character | Key Parameters |
|--------|------|-----------|----------------|
| 0 | **Classic** | Supersaw / pulse / saw / square with sub-osc | param0=unison voices, param1=unison detune, param2=unison blend, param3=sub octave, param4=sub level, param5=drift |
| 1 | **Modern** | Clean bandlimited saw/pulse | Similar to Classic, less coloured, no aliasing |
| 2 | **Wavetable** | Wavetable morphing | param0=wavetable position (morph), param1=morph speed |
| 3 | **Window** | Wavetable with windowing | Smoother wavetable playback |
| 4 | **Sine** | Pure sine with feedback | param0=feedback, creates FM-like self-modulation |
| 5 | **FM2** | 2-operator FM | param0=M/C ratio, param1=M feedback, param2=M detune, param3=modulation index |
| 6 | **FM3** | 3-operator FM | 3-op stack, param0=M1/C ratio, param1=M2/C ratio, param3=M1 index, param4=M2 index |
| 7 | **SurgeFM** | Legacy FM | Older FM engine |
| 8 | **String** | Physical string model | param0=exciter blend, param1=exciter level, param2=stiffness, param3=decay |
| 9 | **Twist** | Mutable Instruments Plaits | param0=engine select (16 engines), param1=harmonics, param2=timbre, param3=morph |
| 10 | **Alias** | Deliberately aliased/lo-fi | param0=waveform, creates harsh digital artefacts |
| 11 | **S&H Noise** | Sample & Hold noise source | Stepped random, useful for percussion/texture |
| 12 | **Audio In** | External audio input | Routes DAW audio through Scene |

**Classic oscillator key params (most used oscillator — 80%+ of factory patches):**
- `param0` = unison voices (0=1, 0.17=2, 0.33=3, 0.5=4, 0.67=5, 0.83=6, 1.0=7)
- `param1` = unison detune (0 to 1.0, maps to 0–100 cents spread)
- `param2` = unison blend (0=all same phase, 1=random phases)
- `param3` = sub oscillator octave (-2, -1, 0)
- `param4` = sub oscillator level (0–1)
- `param5` = drift / analog instability (0=none, 0.17=subtle)

**Global per-oscillator params:**
- `a_osc1_octave`: octave shift (integer: -3 to +3)
- `a_osc1_pitch`: fine pitch in semitones (float: -7 to +7)
- `a_osc1_keytrack`: 0 or 1 (default 1 = tracks keyboard pitch)
- `a_osc1_retrigger`: 0 or 1 (retrigger oscillator phase on new note)

---

## Mixer

Each source has a level (0–1) and mute:
- `a_level_o1`, `a_level_o2`, `a_level_o3` — oscillator levels
- `a_level_noise` — noise generator level
- `a_level_ring12` — ring mod Osc1×Osc2
- `a_level_ring23` — ring mod Osc2×Osc3
- `a_route_o1/o2/o3` — route to filter path A (0) or B (1)

**Noise colour**: `a_noisecol` — -1.0 = pink/warm, 0 = white, +1.0 = blue/bright

---

## Waveshaper

Applies before the filter. Adds harmonics (saturation/distortion).

| Type # | Name | Character |
|--------|------|-----------|
| 0 | None | Bypass |
| 1 | **Tanh** | Soft clip, smooth 2nd+3rd harmonics, tube-like |
| 2 | **Hard** | Hard clip, strong odd harmonics, aggressive |
| 3 | **Asym** | Asymmetric clip, adds even harmonics, warm |
| 4 | **Sine** | Sine fold, complex harmonic reshaping |
| 5 | **Digital** | Digital soft clip, slight 12-bit character |
| 6 | OJD | OD/distortion model |

- `a_ws_type`: integer 0–6
- `a_ws_drive`: drive amount in dB (0 = bypass, 6–18 = typical range)

**Most common in factory**: Asym (warm bass/plucks), Tanh (leads), Digital (modern/digital), Hard (aggressive)

---

## Filters

Two filters per scene, configurable routing (serial, parallel, etc.).

### Filter Routing (`a_fb_config`)
| Value | Routing |
|-------|---------|
| 0 | Serial (Filt1 → Filt2) |
| 1 | Parallel |
| 2 | 2× (stereo split) |
| 3 | Ring (experimental) |
| 4 | Multi (various) |

### Filter Types (`a_filter1_type`)

| Type # | Name | Slope | Character |
|--------|------|-------|-----------|
| 0 | **LP 12dB** | 2-pole | Clean, fast, SSM-like |
| 1 | **LP 24dB** | 4-pole | Punchy, clear resonance |
| 2 | LP Legacy | 4-pole | Older Surge character |
| 3 | **LP Moog** | 4-pole ladder | Non-linear, warm, self-oscillates beautifully |
| 4 | **BP** | Bandpass | Mid-focus, vowel-like |
| 5 | **HP 12dB** | 2-pole | Remove lows, bright |
| 6 | **HP 24dB** | 4-pole | Steep high-pass |
| 7 | **Notch** | Notch | Wah/vowel effect |
| 8 | **Comb+** | Comb | Resonant comb, physical/metallic |
| 9 | **Comb-** | Comb | Inverted comb |
| 10 | S&H | Sample & Hold | Stepped filter |
| 11 | Allpass | Phase shift | Phaser-like, no level change |

**Most popular in factory**: LP-Moog (warmth), LP24 (clarity), LP-Legacy (vintage)

### Filter Key Parameters
- `a_filter1_cutoff`: frequency in semitones, roughly -60 (very closed) to +30 (open). Center ≈ 0 semitones above base pitch. **-24 = darkish, -10 = medium, 0 = bright, +10 = wide open**
- `a_filter1_resonance`: 0.0 (flat) to 1.0 (self-oscillation). Factory avg 0.30–0.50 depending on category
- `a_filter1_envmod`: filter envelope amount in semitones. 0 = no movement. +20 to +40 = significant sweep. Factory Sequences avg 23.5, Leads avg 16.7
- `a_filter1_keytrack`: 0 = fixed cutoff, 1.0 = full key tracking (cutoff follows pitch perfectly)
- `a_filter2_cutoff`: when serial, sets second filter cutoff

---

## Envelopes

Two envelopes per scene — Amp (Env1) and Filter/Mod (Env2). Parameters use Surge's log-time scale:

**Time scale**: `time_ms ≈ 2^value × 1000`. Key reference points:
- `-8` ≈ **4ms** (fastest)
- `-5` ≈ **31ms**
- `-3` ≈ **125ms**
- `-1` ≈ **500ms**
- `0` ≈ **1000ms** (1 second)
- `1` ≈ **2000ms**
- `2` ≈ **4000ms**
- `3` ≈ **8000ms**

### Envelope Parameters
- `a_env1_attack` / `a_env1_decay` / `a_env1_sustain` / `a_env1_release`
- `a_env1_attack_shape`: 0=linear, 1=convex (fast initial rise), -1=concave (slow initial rise)
- `a_env1_decay_shape` / `a_env1_release_shape`: same options
- `a_env1_mode`: 0=ADSR, 1=DAHD (no sustain, gate-independent)

### Typical Envelope Ranges by Category (from factory analysis)

| Category | Attack avg | Release avg | Sustain avg | Notes |
|----------|-----------|-------------|-------------|-------|
| Leads | 52ms | 167ms | 1.0 | Mostly 4ms snap, some slow attacks |
| Pads | 1600ms | 2214ms | 1.0 | Long attack is the defining feature |
| Basses | 5ms | 63ms | 0.80 | Very fast, tight |
| Plucks | 9ms | 1450ms | 0.50 | Fast attack, long natural decay, half sustain |
| Polysynths | 113ms | 683ms | 0.90 | Medium attack, sustained chords |
| Keys | 7ms | 366ms | 0.60 | Fast attack, medium decay |
| Sequences | 4ms | 31ms | 0.70 | Extremely snappy for rhythm |
| Brass | 186ms | 85ms | 1.0 | Slow swell-in attack |
| Percussion | 4ms | 76ms | 0.60 | Always instant attack |

---

## LFOs and Modulators

Surge XT has **12 modulators per scene** (LFO0–LFO11). LFO0–LFO5 are voice-level (per-voice, polyphonic). LFO6–LFO11 are scene-level (single shared instance).

### LFO Shapes (`a_lfo0_shape`)

| Value | Shape | Best For |
|-------|-------|---------|
| 0 | **Sine** | Smooth vibrato, tremolo, filter movement (most common — 60%+ of factory) |
| 1 | **Triangle** | Sharper vibrato, harder tremolo |
| 2 | **Square** | Hard tremolo, choppy effects |
| 3 | Ramp Up | Rising sweep |
| 4 | Ramp Down | Falling sweep |
| 5 | S&H | Random stepped — robotic, glitchy |
| 6 | **Noise Smooth** | Organic random movement, subtle life (2nd most common) |
| 7 | **Noise Stepped** | Rhythmic random — sequences use this for variation |
| 8 | Envelope | LFO acts as one-shot envelope |
| 9 | MSEG | Multi-segment envelope — full custom shape |
| 10 | Formula | Lua-scripted LFO |

### LFO Key Parameters
- `a_lfo0_rate`: LFO rate (log scale, roughly -7 to +9). `0 ≈ 0.5Hz`, `-3 ≈ 0.06Hz`, `3 ≈ 4Hz`
- `a_lfo0_magnitude`: depth (0–1, where 1 = full modulation range)
- `a_lfo0_phase`: start phase 0–1 (0.5 = 180°)
- `a_lfo0_delay`: seconds before LFO starts (for vibrato fade-in)
- `a_lfo0_attack`: fade-in time for LFO (ramp from zero)
- `a_lfo0_deform`: shape deformation (e.g. skews sine into asymmetric shape)
- `a_lfo0_unipolar`: 0 = bipolar (-1 to +1), 1 = unipolar (0 to +1)
- `a_lfo0_trigmode`: 0 = free-running, 1 = keytrigger, 2 = random phase per note

### Modulation Routing (in patch XML)
Modulation is stored as child elements of the target parameter:
```xml
<a_filter1_cutoff value="-24.0">
  <modrouting source="0" depth="0.25" muted="0" source_index="0" />
</a_filter1_cutoff>
```
- `source="0"` = LFO0, `source="1"` = LFO1... `source="17"` = Mod Wheel, etc.
- `depth` = modulation depth (-1 to +1 normalized to parameter range)

---

## Analog Drift

`a_drift` (0–1): introduces subtle pitch instability per voice, simulating analog oscillator imperfection. Factory usage: 6% (Sequences) to 42% (Plucks, Pads). Leads use 32%.

**Recommended starting values**: 0.1–0.15 for subtle analog warmth, 0.3–0.5 for noticeably unstable vintage character.

---

## Portamento

`a_portamento` (log-time scale, same as envelopes):
- `-8` = off (no portamento)
- `-5` to `-3` = short glide (31–125ms)
- `-1` to `0` = medium glide (500ms–1s)

Additional portamento options:
- `porta_const_rate="1"` = constant rate (same time regardless of interval)
- `porta_gliss="1"` = glissando (stepped semitone portamento)
- `porta_retrigger="1"` = retrigger envelope on each note even in legato

**Factory usage**: Leads 82% (mostly short), Basses 59%, Polysynths 34%, Pads 20%.

---

## Effects (FX)

8 global FX slots. Named `fx1_type` through `fx8_type` in patch XML. Scene A/B send to FX via `a_send_fx_1` through `a_send_fx_4` (0–1 level).

### FX Type Reference

| Type # | Name | Key Parameters | Best For |
|--------|------|----------------|---------|
| 0 | None | — | Empty slot |
| 1 | **Chorus** | Rate, depth, feedback, time | Thickening — most common FX in factory (73% of polysynths, 85% of leads) |
| 2 | **Distortion** | Drive, tone, feedback | Saturation, harmonics, weight |
| 3 | **EQ** | Low/Mid/High bands | Tone shaping |
| 4 | Flanger | Rate, depth, feedback | Jet/comb effect |
| 5 | **Phaser** | Rate, depth, stages | Sweeping phase shift |
| 6 | **Rotary** | Speed, distance, stereo | Leslie-style rotation — extremely common (35% of leads) |
| 7 | **Delay** | Time (L/R), feedback, mix | Rhythmic echo |
| 8 | **Reverb 1** | Size, decay, pre-delay | Algorithmic reverb |
| 9 | Freq Shift | Shift amount | Inharmonic transposition |
| 10 | Conditioner | EQ + stereo enhancer | Mastering-style |
| 12 | **Reverb 2** | More parameters | Better reverb |
| 13 | Chorus 2 | Enhanced chorus | Deeper modulation |
| 14 | **Nimbus** | Granular freeze reverb | Mutable Instruments Clouds-style |
| 15 | Ring Mod | Modulator freq | Metallic, inharmonic |
| 16 | Treemonster | Pitch tracking ring mod | Octave/interval effects |
| 18 | Exciter | Harmonic enhancement | High-frequency saturation |
| 19 | CHOW | Tape distortion model | Warm tape saturation |
| 22 | Tape Delay | Tape-style echo | Vintage delay character |
| 23 | Spring Reverb | Spring reverb simulation | Guitar amp style |
| 24 | BBD | Bucket brigade delay | Analog chorus/flanger |
| 25 | **Ensemble** | Multi-voice chorus | Roland Juno-style |
| 26 | Combulator | 4 parallel comb filters | Metallic resonator |
| 28 | Mid-Side | M/S processing | Stereo control |

### Factory FX Usage Patterns

| Category | Primary FX | Secondary FX | Notes |
|----------|-----------|--------------|-------|
| Leads | Chorus (84%) | Rotary (35%) | Almost always chorused |
| Pads | Chorus (100%) | Distortion (59%) | Distortion adds weight to pads |
| Basses | Rotary (33%) | Phaser (17%) | Subtle modulation only |
| Plucks | Chorus (84%) | Distortion (44%) | Always send to chorus |
| Polysynths | Chorus (100%) | Distortion (40%) | Chorus on every polysynth patch |
| Sequences | Chorus (100%) | Reverb/Phaser (15%) | Always chorus |
| Brass | Chorus+Distortion (100%) | — | Both always present |

---

## Wavetables (179 factory wavetables)

Organized in categories: Basic, Generated, Oneshot, Rhythmic, Sampled, Waldorf

**Basic (6)**: Sine, Triangle, Tri-Saw, Sine Octaves, Sine-to-Saw, Sine-to-Square

**Generated (55+)**: PWM, Pulse variants, Saw variants (CS-80, Detuned, Havoc, Sync), Sine FM/PD/Power variants, Phase-distorted shapes

**Sampled (30+)**: Acoustic instruments (guitar, harp, koto, cello, piano), vocal formants, mallets, music box

**Waldorf (40+)**: Classic Waldorf Microwave/Wave wavetables — B3Waves, DXBASS, FMBell, KlingKlan, Formant variants

**Rhythmic (16)**: Percussive/textural — Bata, Drumbeat, Computer, Laser, Sparkly, Sprinkles

---

## Key Workflow Notes

1. **Scene A is the default** — most patches use only Scene A. Scene B is used for splits/layers.
2. **Filter cutoff values** are in semitones relative to a base pitch, NOT in Hz. A value of -24 means roughly 6 octaves below the open position.
3. **Chorus is on almost everything** — factory patches use it for thickening. Don't skip it.
4. **Rotary speaker** is used extensively on leads (35%) — it adds stereo movement and subtle pitch modulation beyond what a standard chorus does.
5. **LFO0 is almost always active** — 87–100% of patches across all categories. A completely static patch is unusual.
6. **Waveshaper before filter** — this is the intended signal path. Drive into a closed filter for FM-like timbral effects.
7. **Drift is not on by default** — add 0.1–0.2 to any Classic oscillator patch to make it feel more alive.

---

## Creating and Writing FXP Patch Files

Surge XT patches are `.fxp` files: a VST2 FXPreset binary container wrapping an XML document.

### Binary format (verified from factory patches)

```
Offset  Size  Value         Description
------  ----  ----------    -----------
0       4     CcnK          VST magic bytes (always this value)
4       4     0             Chunk size field — Surge writes 0, ignored on load
8       4     FPCh          Preset type: FX Preset with Chunk data
12      4     1 (BE)        Format version (big-endian uint32)
16      4     cjs3          Surge XT plugin ID (Claes Johanson Surge 3)
20      4     1 (BE)        Plugin version (big-endian uint32)
24      4     1 (BE)        Num programs = 1
28      28    name\x00...   Patch name, null-padded to 28 bytes
56      4     N (BE)        Total chunk size in bytes (big-endian uint32)
--- CHUNK starts at byte 60 ---
60      4     sub3          Surge sub-chunk magic
64      4     M (LE)        XML length in bytes (little-endian uint32), M = N - 32
68      24    \x00...       24 zero bytes (padding)
92      M     <?xml...      The XML patch data (UTF-8)
```

Total pre-XML overhead: **92 bytes** (60 header + 32 chunk prefix).

### Python: modify an existing patch and save as new FXP

The easiest approach — read a factory or user patch, modify parameters, write back:

```python
import struct, xml.etree.ElementTree as ET

def read_fxp(path):
    """Read FXP and return (header_bytes, patch_name, xml_string)."""
    with open(path, 'rb') as f:
        data = f.read()
    chunk_sz = struct.unpack('>I', data[56:60])[0]
    xml_data = data[92:60+chunk_sz].decode('utf-8')
    patch_name = data[28:56].rstrip(b'\x00').decode('utf-8', errors='replace')
    return data[:60], patch_name, xml_data

def write_fxp(output_path, patch_name, xml_string):
    """Write a Surge XT FXP file from a patch name and XML string."""
    xml_bytes = xml_string.encode('utf-8')
    xml_len = len(xml_bytes)
    chunk_sz = xml_len + 32  # 32 bytes of sub-header

    header = (
        b'CcnK'                              # VST magic
        + struct.pack('>I', 0)               # size field (Surge writes 0)
        + b'FPCh'                            # preset type
        + struct.pack('>I', 1)               # format version
        + b'cjs3'                            # Surge XT plugin ID
        + struct.pack('>I', 1)               # plugin version
        + struct.pack('>I', 1)               # num programs
        + patch_name.encode('utf-8')[:28].ljust(28, b'\x00')  # name, 28 bytes
        + struct.pack('>I', chunk_sz)        # chunk size
        # sub-chunk:
        + b'sub3'                            # sub-chunk magic
        + struct.pack('<I', xml_len)         # XML length (little-endian)
        + b'\x00' * 24                       # 24 bytes padding
    )

    with open(output_path, 'wb') as f:
        f.write(header + xml_bytes)

def modify_patch(source_fxp, output_fxp, new_name, param_changes):
    """
    Load source_fxp, apply param_changes dict, save as output_fxp.
    param_changes: dict of {xml_param_name: new_value_string}

    Example:
        modify_patch(
            '/path/to/Init Saw.fxp',
            '/path/to/My Pad.fxp',
            'My Pad',
            {
                'a_env1_attack':  '-3.0',     # ~125ms attack
                'a_env1_release': '0.0',      # ~1s release
                'a_filter1_type': '1',        # LP 24dB
                'a_filter1_cutoff': '-10.0',  # medium open
                'a_filter1_resonance': '0.35',
                'a_lfo0_magnitude': '0.15',   # subtle LFO
                'a_ws_type': '1',             # Tanh waveshaper
                'a_ws_drive': '6.0',
                'a_drift': '0.15',            # analog drift
            }
        )
    """
    _, _, xml_string = read_fxp(source_fxp)
    root = ET.fromstring(xml_string)

    # Update meta name
    meta = root.find('meta')
    if meta is not None:
        meta.set('name', new_name)

    # Update parameters
    params = root.find('parameters')
    param_dict = {e.tag: e for e in params}
    for param_name, new_value in param_changes.items():
        if param_name in param_dict:
            param_dict[param_name].set('value', new_value)
        else:
            # Parameter not present — add it
            new_el = ET.SubElement(params, param_name)
            new_el.set('type', '2')  # float type
            new_el.set('value', new_value)

    xml_out = '<?xml version="1.0" encoding="UTF-8" standalone="yes" ?>'
    xml_out += ET.tostring(root, encoding='unicode')
    write_fxp(output_fxp, new_name, xml_out)
    print(f"Saved: {output_fxp}")
```

### Init Saw default values (canonical starting point)

These are the defaults in `Templates/Init Saw.fxp` — use this as the base for any new patch:

```
scenemode:              0        (Poly)
a_osc1_type:            0        (Classic)
a_osc2_type:            0        (Classic, muted by default)
a_osc3_type:            0        (Classic, muted by default)
a_level_o1:             1.0      (Osc 1 on)
a_level_o2:             1.0      (Osc 2 on but muted in mixer)
a_level_o3:             1.0      (Osc 3 on but muted in mixer)
a_filter1_type:         0        (LP 12dB)
a_filter1_cutoff:       3.0      (wide open — high positive semitone value)
a_filter1_resonance:    0.0      (no resonance)
a_env1_attack:          -8.0     (~4ms)
a_env1_decay:           -2.0     (~250ms)
a_env1_sustain:         1.0      (full sustain)
a_env1_release:         -5.0     (~31ms)
a_env2_attack:          -8.0
a_env2_decay:           -2.0
a_env2_sustain:         1.0
a_env2_release:         -5.0
a_portamento:           -8.0     (off)
a_ws_type:              0        (no waveshaper)
a_ws_drive:             0.0
a_lfo0_shape:           0        (Sine)
a_lfo0_rate:            0.0
a_lfo0_magnitude:       1.0      (note: magnitude=1 but mod routing determines actual depth)
a_drift:                0.0      (no analog drift)
```

### Factory Init patch location

The Init Saw template is at:
```
/Library/Application Support/Surge XT/patches_factory/Templates/Init Saw.fxp
```
Use this as `source_fxp` in `modify_patch()` for any new patch from scratch.

### Workflow for creating a new patch

1. Use `modify_patch()` with `Init Saw.fxp` as the source
2. Set the parameters that define the sound (oscillator type, filter, envelopes, waveshaper, FX)
3. Leave unspecified parameters at their Init defaults
4. Save to the user's Surge XT user patches directory:
   ```
   ~/Documents/Surge XT/patches_user/
   ```
   or
   ```
   ~/Library/Application Support/Surge XT/patches_user/
   ```
5. Reload patches in Surge XT: Menu → Patch Library → Refresh
