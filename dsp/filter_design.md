---
title: "Filter Design for Synthesizers"
category: "DSP"
tags: [filters, biquad, moog-ladder, state-variable, EQ, resonance, DSP]
---

# Filter Design for Synthesizers

## Overview

Filters are the core timbral shaping tool in synthesis. A filter selectively attenuates certain frequency regions. In digital synthesis, two fundamental IIR filter forms dominate: the **biquad** (second-order, universal building block) and the **state-variable filter** (SVF, preferred for real-time cutoff modulation).

---

## The Biquad (Second-Order IIR)

A biquad is a second-order IIR (Infinite Impulse Response) filter with 2 poles and 2 zeros. It is the most versatile building block in digital audio. Source: Nigel Redmon / EarLevel Engineering.

### Canonical Difference Equation

**Standard convention (Robert Bristow-Johnson / Audio EQ Cookbook):**

```
y[n] = b0*x[n] + b1*x[n-1] + b2*x[n-2]
             - a1*y[n-1] - a2*y[n-2]
```

Where:
- `x[n]` = current input sample
- `y[n]` = current output sample
- `b0, b1, b2` = feedforward (numerator) coefficients
- `a1, a2` = feedback (denominator) coefficients (a0 normalized to 1)
- Delay elements: `x[n-1]`, `x[n-2]`, `y[n-1]`, `y[n-2]`

Note: Some implementations (including EarLevel) swap a/b labeling. Always verify convention by checking whether normalization is on the numerator or denominator. The RBJ Cookbook convention (b = numerator, a = denominator) is the most widely adopted.

### Transfer Function (Z-domain)

```
        b0 + b1*z^-1 + b2*z^-2
H(z) = ─────────────────────────
         1 + a1*z^-1 + a2*z^-2
```

### Coefficient Derivation Parameters

For all filter types, define:
```
w0 = 2 * π * f0 / Fs          // angular frequency (radians/sample)
α  = sin(w0) / (2 * Q)        // bandwidth parameter
A  = sqrt(10^(dBgain/20))     // linear gain (for peaking/shelving)
```

Alternatively, using bilinear transform approach (EarLevel / Nigel Redmon):
```
K = tan(π * Fc)   // where Fc = f0/Fs (normalized frequency 0..0.5)
```

---

## Biquad Filter Types: Exact Coefficient Formulas

### Low-Pass Filter (LPF)

Passes frequencies below cutoff, attenuates above. -12 dB/octave roll-off. -3 dB at f0 when Q = 0.7071.

**RBJ Cookbook:**
```
b0 = (1 - cos(w0)) / 2
b1 =  1 - cos(w0)
b2 = (1 - cos(w0)) / 2
a0 =  1 + α
a1 = -2 * cos(w0)
a2 =  1 - α

// Normalize: divide all by a0
```

**EarLevel bilinear (using K = tan(π*Fc)):**
```c
norm = 1 / (1 + K/Q + K*K);
a0   = K*K * norm;
a1   = 2 * a0;
a2   = a0;
b1   = 2 * (K*K - 1) * norm;
b2   = (1 - K/Q + K*K) * norm;
```

### High-Pass Filter (HPF)

Passes frequencies above cutoff, attenuates below. -12 dB/octave roll-off below f0.

**RBJ Cookbook:**
```
b0 =  (1 + cos(w0)) / 2
b1 = -(1 + cos(w0))
b2 =  (1 + cos(w0)) / 2
a0 =   1 + α
a1 =  -2 * cos(w0)
a2 =   1 - α
```

**EarLevel bilinear:**
```c
norm = 1 / (1 + K/Q + K*K);
a0   = 1 * norm;
a1   = -2 * a0;
a2   = a0;
b1   = 2 * (K*K - 1) * norm;
b2   = (1 - K/Q + K*K) * norm;
```

### Band-Pass Filter (BPF)

Peak at f0, attenuates on both sides. Two variants: constant skirt gain vs. constant 0 dB peak.

**RBJ Cookbook (constant 0 dB peak gain):**
```
b0 =  α
b1 =  0
b2 = -α
a0 =  1 + α
a1 = -2 * cos(w0)
a2 =  1 - α
```

**EarLevel bilinear:**
```c
norm = 1 / (1 + K/Q + K*K);
a0   = K/Q * norm;
a1   = 0;
a2   = -a0;
b1   = 2 * (K*K - 1) * norm;
b2   = (1 - K/Q + K*K) * norm;
```

### Notch Filter (Band-Reject)

Null at f0, passes everything else. Useful for hum removal (50/60 Hz).

**RBJ Cookbook:**
```
b0 =  1
b1 = -2 * cos(w0)
b2 =  1
a0 =  1 + α
a1 = -2 * cos(w0)
a2 =  1 - α
```

**EarLevel bilinear:**
```c
norm = 1 / (1 + K/Q + K*K);
a0   = (1 + K*K) * norm;
a1   = 2 * (K*K - 1) * norm;
a2   = a0;
b1   = a1;
b2   = (1 - K/Q + K*K) * norm;
```

### All-Pass Filter

Flat magnitude response, shifts phase. Used for phaser effects and stereo widening.

**RBJ Cookbook:**
```
b0 =  1 - α
b1 = -2 * cos(w0)
b2 =  1 + α
a0 =  1 + α
a1 = -2 * cos(w0)
a2 =  1 - α
```

Note: b0=a2, b1=a1, b2=a0 — the transfer function is the reverse of the denominator polynomial.

### Peaking EQ (Bell)

Boosts or cuts a frequency band. Used in parametric equalizers. dBgain > 0 boosts, < 0 cuts.

**RBJ Cookbook:**
```
b0 =  1 + α*A
b1 = -2 * cos(w0)
b2 =  1 - α*A
a0 =  1 + α/A
a1 = -2 * cos(w0)
a2 =  1 - α/A

where A = sqrt(10^(dBgain/20))
      α = sin(w0) / (2*Q)
```

**EarLevel bilinear (boost case):**
```c
V = pow(10, fabs(peakGain) / 20.0);
K = tan(M_PI * Fc);
if (peakGain >= 0) {  // boost
    norm = 1 / (1 + 1/Q * K + K*K);
    a0   = (1 + V/Q * K + K*K) * norm;
    a1   = 2 * (K*K - 1) * norm;
    a2   = (1 - V/Q * K + K*K) * norm;
    b1   = a1;
    b2   = (1 - 1/Q * K + K*K) * norm;
} else {              // cut
    norm = 1 / (1 + V/Q * K + K*K);
    a0   = (1 + 1/Q * K + K*K) * norm;
    a1   = 2 * (K*K - 1) * norm;
    a2   = (1 - 1/Q * K + K*K) * norm;
    b1   = a1;
    b2   = (1 - V/Q * K + K*K) * norm;
}
```

### Low Shelf Filter

Boosts or cuts all frequencies below the shelf frequency. Used for bass control.

**RBJ Cookbook:**
```
b0 =    A * [ (A+1) - (A-1)*cos(w0) + 2*sqrt(A)*α ]
b1 =  2*A * [ (A-1) - (A+1)*cos(w0)               ]
b2 =    A * [ (A+1) - (A-1)*cos(w0) - 2*sqrt(A)*α ]
a0 =        [ (A+1) + (A-1)*cos(w0) + 2*sqrt(A)*α ]
a1 =   -2 * [ (A-1) + (A+1)*cos(w0)               ]
a2 =        [ (A+1) + (A-1)*cos(w0) - 2*sqrt(A)*α ]
```

**EarLevel bilinear (boost):**
```c
norm = 1 / (1 + sqrt(2)*K + K*K);
a0   = (1 + sqrt(2*V)*K + V*K*K) * norm;
a1   = 2 * (V*K*K - 1) * norm;
a2   = (1 - sqrt(2*V)*K + V*K*K) * norm;
b1   = 2 * (K*K - 1) * norm;
b2   = (1 - sqrt(2)*K + K*K) * norm;
```

### High Shelf Filter

Boosts or cuts all frequencies above the shelf frequency. Used for treble control.

**EarLevel bilinear (boost):**
```c
norm = 1 / (1 + sqrt(2)*K + K*K);
a0   = (V + sqrt(2*V)*K + K*K) * norm;
a1   = 2 * (K*K - V) * norm;
a2   = (V - sqrt(2*V)*K + K*K) * norm;
b1   = 2 * (K*K - 1) * norm;
b2   = (1 - sqrt(2)*K + K*K) * norm;
```

---

## Biquad Topologies

### Direct Form I

Four delay elements (x[n-1], x[n-2], y[n-1], y[n-2]):

```c
// Direct Form I
y[n] = b0*x[n] + b1*x[n-1] + b2*x[n-2]
                - a1*y[n-1] - a2*y[n-2]
```

- **Best for fixed-point processors** (single summation point, extended accumulator handles intermediate overflow)
- More memory than Direct Form II (4 delay elements vs 2)
- Better for coefficient changes (no transient artifacts from delay memory)

### Direct Form II

Two delay elements (shared between numerator and denominator):

```c
// Direct Form II (Canonical)
w[n] = x[n] - a1*w[n-1] - a2*w[n-2]
y[n] = b0*w[n] + b1*w[n-1] + b2*w[n-2]
```

- Only 2 delay elements (memory efficient)
- **Preferred for floating point** (saves memory)
- Numerically less stable than transposed version at extreme settings

### Transposed Direct Form II (Recommended for Floating Point)

```c
// Transposed Direct Form II
double out = in * a0 + z1;
z1         = in * a1 + z2 - b1 * out;
z2         = in * a2       - b2 * out;
return out;
```

- Only 2 state variables (`z1`, `z2`)
- **Best numerical behavior for floating point** (adds similar magnitude numbers)
- Used in Nigel Redmon's EarLevel biquad implementation
- Preferred for audio plugins and software synthesizers

---

## Complete C++ Biquad Implementation (EarLevel)

```cpp
// Biquad.h — Nigel Redmon, EarLevel Engineering (earlevel.com)
enum {
    bq_type_lowpass = 0,
    bq_type_highpass,
    bq_type_bandpass,
    bq_type_notch,
    bq_type_peak,
    bq_type_lowshelf,
    bq_type_highshelf
};

class Biquad {
public:
    Biquad();
    Biquad(int type, double Fc, double Q, double peakGainDB);
    ~Biquad();
    void setType(int type);
    void setQ(double Q);
    void setFc(double Fc);        // Fc = f0/sampleRate, range [0, 0.5]
    void setPeakGain(double peakGainDB);
    void setBiquad(int type, double Fc, double Q, double peakGainDB);
    float process(float in);

protected:
    void calcBiquad(void);
    int type;
    double a0, a1, a2, b1, b2;
    double Fc, Q, peakGain;
    double z1, z2;              // state variables for transposed DF-II
};

// Inline process function (called every sample)
inline float Biquad::process(float in) {
    double out = in * a0 + z1;
    z1         = in * a1 + z2 - b1 * out;
    z2         = in * a2       - b2 * out;
    return out;
}
```

### Usage

```cpp
// Set a 1kHz lowpass at 44100 Hz sample rate, Q=0.707
Biquad *lpf = new Biquad();
lpf->setBiquad(bq_type_lowpass, 1000.0 / 44100.0, 0.707, 0.0);

// Process buffer in-place
for (int i = 0; i < bufSize; i++) {
    buf[i] = lpf->process(buf[i]);
}

// Cascade for higher order (4-pole, 24 dB/oct):
Biquad *lp1 = new Biquad(bq_type_lowpass, fc/sr, 0.541, 0.0);
Biquad *lp2 = new Biquad(bq_type_lowpass, fc/sr, 1.307, 0.0);
// Q values for 4th-order Butterworth: 0.5412 and 1.3066
out = lp2->process(lp1->process(in));
```

---

## Butterworth Filter

- **Maximally flat magnitude** in the passband (no ripple)
- -3 dB exactly at the cutoff frequency
- For 2nd-order biquad: **Q = 1/√2 = 0.7071**
- For 4th-order (two cascaded biquads):
  - Stage 1: Q = 1/(2*sin(π/8)) ≈ **0.5412**
  - Stage 2: Q = 1/(2*sin(3π/8)) ≈ **1.3066**
- For 6th-order (three cascaded biquads):
  - Q values: 0.5176, 0.7071, 1.9319
- Monotonically decreasing response (no ripple in passband or stopband)
- Phase response: nonlinear (all-pole)

---

## Moog Ladder Filter

The Moog VCF (Voltage Controlled Filter) is a 4-pole (24 dB/octave) lowpass filter with resonance. Analog original uses four cascaded transistor ladder stages. Digital approximation from Stilson/Smith CCRMA paper, implemented as four cascaded one-pole filters with overall feedback.

### Moog VCF Algorithm (Huovilainen/Antti)

```c
// Init:
// cutoff = cutoff freq in Hz
// fs     = sample rate (e.g. 44100)
// res    = resonance [0.0 = min, 1.0 = max/self-oscillation]

f     = 2.0 * cutoff / fs;              // normalized frequency [0..1]
k     = 3.6*f - 1.6*f*f - 1.0;         // empirical tuning
p     = (k + 1.0) * 0.5;
scale = exp((1.0 - p) * 1.386249);
r     = res * scale;

y1 = y2 = y3 = y4 = 0.0;
oldx = oldy1 = oldy2 = oldy3 = 0.0;

// Per-sample loop:
x  = input - r * y4;                    // feedback (inverted)
y1 = x*p   + oldx*p   - k*y1;          // stage 1
y2 = y1*p  + oldy1*p  - k*y2;          // stage 2
y3 = y2*p  + oldy2*p  - k*y3;          // stage 3
y4 = y3*p  + oldy3*p  - k*y4;          // stage 4

// Soft clipping (band-limited sigmoid):
y4     = y4 - (y4*y4*y4) / 6.0;

oldx   = x;
oldy1  = y1;
oldy2  = y2;
oldy3  = y3;
// Output = y4
```

### Moog VCF Key Properties

- **4-pole**: -24 dB/octave slope
- **Resonance feedback**: fed back to input (inverted) — controls resonance/Q
- At `res ≈ 1.0`: self-oscillation, filter acts as sine oscillator
- **Nonlinear saturation**: `tanh()` or cubic approximation `x - x³/6` applied to output or at each stage for authenticity
- Cutoff range: typically 20 Hz – 20 kHz, modulated by control voltage in analog, by coefficient recalculation in digital
- **Dynamic modulation**: can be swept every sample because each stage's coefficient is just a single multiply; smooth transitions

### Huovilainen Improved Moog (with nonlinearity at each stage)

```c
// Apply tanh saturation inside each stage for better authenticity:
y1 = tanh(x*p + oldx*p - k*y1);
y2 = tanh(y1*p + oldy1*p - k*y2);
y3 = tanh(y2*p + oldy2*p - k*y3);
y4 = tanh(y3*p + oldy3*p - k*y4);
```

The key insight: saturation inside the loop creates the characteristic Moog warmth and prevents harsh digital clipping.

---

## State-Variable Filter (SVF)

### Chamberlin State-Variable Filter

Described in Hal Chamberlin's *Musical Applications of Microprocessors*. The SVF gives simultaneous LP, HP, BP, and BR (band-reject) outputs from a single structure.

**Coefficients:**
```
f = 2 * sin(π * Fc / Fs)   // frequency coefficient
q = 1 / Q                   // damping (inverse of Q)
```

**Per-sample iteration:**
```c
// Chamberlin SVF (Huovilainen corrected form):
LP = LP + f * BP;
HP = input - LP - q * BP;
BP = f * HP + BP;
BR = HP + LP;           // band-reject (notch)

// Output: LP, HP, BP, BR all simultaneously available
```

Note: The basic Chamberlin form becomes unstable above ~Fs/6 (≈ 8 kHz at 48 kHz). **Always oversample or use the corrected double-sampled version** for synthesis with swept cutoff.

### Double-Sampled Stable SVF

Run the filter twice per sample to double the stable frequency range:
```c
// Run filter at 2x internal rate:
for (int i = 0; i < 2; i++) {
    LP = LP + f * BP;
    HP = input - LP - q * BP;
    BP = f * HP + BP;
}
// Doubles stable cutoff range to ~Fs/3
```

### SVF Key Properties

- **Independent frequency and Q coefficients**: trivial to modulate separately
- **Simultaneous outputs**: LP, HP, BP, BR — interpolate between for morphing
- **Excellent low-frequency performance** (where biquads struggle)
- **Weak at high frequencies** unless oversampled
- LP output: -12 dB/oct roll-off
- BP output: peak at Fc, -6 dB/oct each side
- **Sine oscillator mode**: set q=0 (no damping), it oscillates indefinitely

### SVF as Sine Oscillator

```c
// Initialize:
sinZ = 0.0;
cosZ = amplitude;   // start value (peak amplitude)
f    = 2.0 * sin(M_PI * freq / sampleRate);

// Per sample:
sinZ = sinZ + f * cosZ;
cosZ = cosZ - f * sinZ;
// sinZ = sine output, cosZ = cosine output (90° apart)
```

For low frequencies, approximate: `f ≈ 2 * π * freq / sampleRate` (skips sin() computation, valid below ~Fs/20).

---

## Filter Cascade and Higher Orders

### Cascading Biquads for Steeper Slopes

- Two cascaded biquads → 4th order (24 dB/octave)
- Three cascaded biquads → 6th order (36 dB/octave)
- Important: when cascading identical biquads for a sharper filter, the -3 dB point shifts. Adjust Q values accordingly.

```cpp
// 4-pole Butterworth LPF via two biquads:
Biquad *stage1 = new Biquad(bq_type_lowpass, fc/sr, 0.5412, 0.0);
Biquad *stage2 = new Biquad(bq_type_lowpass, fc/sr, 1.3066, 0.0);
output = stage2->process(stage1->process(input));
```

### Filter Topology Comparison

| Property             | Biquad DF-I | Biquad DF-II Transposed | Chamberlin SVF | Moog Ladder |
|----------------------|-------------|-------------------------|----------------|-------------|
| Order                | 2nd         | 2nd                     | 2nd            | 4th         |
| Delay elements       | 4           | 2                       | 2              | 4           |
| Fixed-point safe     | Yes         | No                      | Moderate       | Moderate    |
| Sweep-able           | Poor        | Poor                    | Excellent      | Excellent   |
| Low-freq stability   | Poor        | Poor                    | Excellent      | Excellent   |
| High-freq stability  | Excellent   | Excellent               | Poor (>Fs/6)   | Moderate    |
| Simultaneous outputs | No          | No                      | LP/HP/BP/BR    | LP only     |
| Nonlinear character  | None        | None                    | Optional       | Integral    |
| Slope (per section)  | 12 dB/oct   | 12 dB/oct               | 12 dB/oct      | 24 dB/oct   |

---

## Resonance and Self-Oscillation

- **Q factor**: ratio of center frequency to bandwidth. Higher Q = sharper peak = more resonance.
- **Butterworth**: Q = 0.7071 (maximally flat, minimal overshoot)
- **Resonance sweep**: in SVF and Moog ladder, Q can be changed per sample; in biquad, recalculating coefficients every sample is expensive
- **Self-oscillation threshold**:
  - Biquad BPF: poles reach the unit circle as Q → ∞ (system unstable)
  - Moog ladder: `res ≈ 0.9–1.0` causes sustained sine oscillation at cutoff frequency
  - SVF: `q = 0` (no damping) causes eternal oscillation
- **Oscillating filter as sine source**: set cutoff via coefficient and initialize with a nonzero value — elegant sine generator with no CPU-expensive sin() call

---

## Key-Tracking Formula

Key-tracking maps MIDI note to filter cutoff, so the filter follows the played pitch:

```
// Map MIDI note to Hz:
freq_hz = 440.0 * pow(2.0, (midi_note - 69) / 12.0)

// Key-tracking blend (0.0 = no tracking, 1.0 = full 1:1 tracking):
cutoff_hz = base_cutoff_hz + key_track_amount * freq_hz

// Example: base cutoff 1000 Hz, 50% key tracking:
cutoff_hz = 1000.0 + 0.5 * note_freq_hz

// Set biquad Fc (normalized):
biquad->setFc(cutoff_hz / sample_rate);
```

---

## Numerical Stability Considerations

- **32-bit float**: adequate for audio filters above ~200 Hz at 44100 Hz. Use double for stability below that.
- **At low frequencies**: poles cluster near z=1; coefficient differences become tiny, causing precision loss. Use **first-order noise shaping** (fixed point) or **double precision** (floating point).
- **Coefficient clamping**: always clamp `Fc` to `[0.001, 0.499]` (a small margin below 0.5 prevents NaN from tan(π * 0.5)):
  ```c
  if (Fc < 0.001) Fc = 0.001;
  if (Fc > 0.499) Fc = 0.499;
  ```
- **16-bit fixed point**: generally not adequate for audio without double-precision arithmetic
- **High sample rates**: coefficient sensitivity worsens at high rates; double precision strongly recommended at 96 kHz+

---

## 303-Style Resonant Filter

The Roland TB-303 used a diode ladder with resonance. A common digital approximation:

```c
// Simple resonant LP (Huovilainen-style, 3-pole with saturation):
// Parameters: cutoff [0..1], resonance [0..~1]
cutf = tan(M_PI * cutoff / sampleRate);
res  = resonance;

// Feedback with nonlinearity:
for (each sample) {
    x   = input - res * y3;
    y1 += cutf * (tanh(x) - tanh(y1));
    y2 += cutf * (y1 - y2);
    y3 += cutf * (y2 - y3);
    output = y3;
}
```

The tanh inside the loop creates the characteristic 303 "squelch" on resonance sweeps.

---

## RBJ Audio EQ Cookbook Summary

Robert Bristow-Johnson's Audio EQ Cookbook (widely cited, freely available) provides the canonical biquad coefficient formulas for all common filter types. Key parameters:

- `f0`: center/corner frequency in Hz
- `Fs`: sample rate in Hz
- `Q`: quality factor (dimensionless)
- `dBgain`: gain in dB (for peaking/shelving only)
- `BW`: bandwidth in octaves (alternative to Q for bandpass/notch)
- `S`: shelf slope parameter (for shelving filters)

Bandwidth to Q conversion: `Q = 1 / (2 * sinh(ln(2)/2 * BW * w0/sin(w0)))`

For shelf slope: `α = sin(w0)/2 * sqrt( (A + 1/A)*(1/S - 1) + 2 )`

---

## Practical Filter Design Recommendations

1. **For static EQ** (mastering, mixing, plugin EQ): use biquad in transposed direct form II with double precision
2. **For swept synth filter** (modulated by envelope/LFO): use SVF (double-sampled) or Moog ladder — NOT direct-form biquad
3. **For low-frequency control filtering** (smoothing automation): use 1-pole IIR: `y[n] = a*x[n] + (1-a)*y[n-1]` where `a = 1 - exp(-2π*fc/fs)`
4. **For analog character**: add tanh saturation inside feedback loops
5. **For multiple outputs** (morph between LP/HP/BP): use SVF
6. **For 24 dB/oct Moog sound**: use digital Moog ladder with tanh saturation at each stage
