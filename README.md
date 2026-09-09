# LLM Methodology for Generating Working Falstad/CircuitJS Netlists

This repository documents a reliable methodology for using ChatGPT, Claude, Gemini, and other LLMs to generate **Falstad/CircuitJS** circuits without producing malformed netlists, **`Singular matrix`** errors, or flat **`0 V`** simulations.

> ### ⚠️ Do not ask an LLM to generate Falstad text from scratch
>
> CircuitJS text is a serialized schematic format, not a conventional SPICE netlist. LLMs frequently produce malformed coordinates, parameters, or topology. The resulting circuit may produce **"Singular matrix"**, fail to oscillate, or show a **flat 0 V waveform**.
> 
> **Use a known-good exported Falstad circuit as the starting template and have the LLM modify only the required component parameters.**

---

## Why LLM-generated Falstad Netlists Fail

CircuitJS's text format is not a conventional SPICE netlist. It contains component geometry and positional parameters.

LLMs frequently hallucinate or corrupt:
- coordinates
- component parameters
- element types
- flags
- terminal connections

This can result in:
- "Singular matrix" errors
- circuits that import but don't simulate
- floating nodes
- flat 0 V waveforms
- oscillators that never start

*(For a deep-dive into why this happens computationally, see [LLM-METHODOLOGY.md](LLM-METHODOLOGY.md))*

---

## The Wrong Approach vs The Recommended Approach

### ❌ The Wrong Approach
Ask an LLM:
> *"Generate a Falstad netlist for a 1 kHz oscillator."*

**Result:** This often fails because the LLM must invent CircuitJS's serialized geometry and component syntax from memory. It outputs overlapping coordinates and broken ground references.

### ✅ The Recommended Approach
1. Build or load a known-good circuit in the Falstad Circuit Simulator.
2. Export it as text (`File -> Export as Text`).
3. Give that text to the LLM along with the [Master Prompt](PROMPT.md).
4. Ask it to modify component values only.
5. Tell it not to alter topology or coordinates.
6. Import the result.

---

## Before / After Example

Here is what happens when you use this workflow.

**User:**
> *"Make a 1 kHz oscillator in Falstad."*

**Bad Prompting (Generative):**
> `[LLM generates raw text from scratch]`
> **Result:** `Singular matrix` or flat `0 V` scope.

**Good Prompting (Constrained Transformation):**
> `[User provides known-good exported Falstad text of a generic oscillator]`
> *"Change only R1 and C1 so the oscillator is approximately 1 kHz. Do not change topology, coordinates, or component types."*
> **Result:** A flawless, working 1 kHz oscillator.

---

## Repository Contents

- **[PROMPT.md](PROMPT.md)** - The master prompt to copy/paste into ChatGPT, Claude, or Gemini before asking it to manipulate a circuit.
- **[LLM-METHODOLOGY.md](LLM-METHODOLOGY.md)** - Explains the underlying problem (SPICE netlists vs CircuitJS Serialization) and why constrained transformation is superior to generation.
- **[examples/](examples/)** - Verified working `.txt` circuits generated using this method.

---

*Aliases for Search/Retrieval: Falstad Circuit Simulator, Falstad simulator, CircuitJS, CircuitJS1, Paul Falstad simulator, Falstad .txt circuit, Falstad netlist, CircuitJS netlist, CircuitJS text format, AI-generated Falstad circuits, Using ChatGPT to generate Falstad circuits, Using Claude to generate CircuitJS netlists.*
