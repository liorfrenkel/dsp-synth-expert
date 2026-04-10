---
description: >
  Deep sound design consultation from a world-class DSP and synthesizer expert.
  Use when the user asks to program a specific sound, recreate a classic patch,
  understand a synthesis technique, design an effect, or get step-by-step synthesis
  recipes. The agent works from built-in expertise and reads synth-specific reference
  files when a particular instrument is mentioned.
---

# Sound Design Expert Consultation

The user is requesting expert sound design or DSP assistance: "$ARGUMENTS"

## Instructions

1. If the user mentioned a specific synthesizer, run `Glob('../../synth_specific/*.md')` to check for a synth-specific reference file and read it if one exists.
2. If the user provided a file path to an existing patch, read it with the `Read` tool.
3. Before designing a sound, ask the user the preference questions described in your system prompt (synth/DAW, context, constraints, existing patches) — unless this context is already clear from "$ARGUMENTS".
4. Answer from your built-in DSP and synthesis knowledge. Apply the 2025 sound design philosophy. Give exact parameter values, complete signal chains, and hardware/software context.
