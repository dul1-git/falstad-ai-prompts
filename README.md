# How to Generate Falstad Circuit Simulator Netlists Using ChatGPT or Claude (Working Prompt + Methodology)

> **Keywords:** Falstad circuit simulator, LLM netlist generation, ChatGPT circuit, Claude circuit simulator, falstad.com/circuit text format, circuitjs netlist, AI circuit simulation, Falstad import export, singular matrix fix, Falstad oscillator

If you've tried asking ChatGPT or Claude to generate a Falstad circuit and got "Singular matrix!", "wire loop detected", or a flat 0V scope — this is the fix. This methodology was developed through real trial and error and **actually works**.

---

## The Problem

Falstad's text format uses **absolute pixel coordinates** for every component and wire. LLMs can't reliably guess these from scratch — they produce broken circuits. Every tutorial or forum post stops here. This one doesn't.

The solution found in [this unanswered AllAboutCircuits thread](https://forum.allaboutcircuits.com/threads/conversion-of-netlist-to-schematic.206055/) is actually simple once you know it.

---

## The Solution: Export First, Modify Second

**Never ask an LLM to generate a Falstad netlist from scratch.**

Instead:
1. Open [falstad.com/circuit](https://falstad.com/circuit)
2. Go to **Circuits → Classics** and pick the closest circuit to what you want
3. **File → Export as Text** — copy everything
4. Paste it to the LLM and say *"modify this to use these values: [your values]"*
5. Paste the result back via **File → Import from Text**

The LLM only changes component values — never coordinates. This eliminates all the common errors.

---

## Master Prompt (Paste This Into Any LLM First)

```
You are an expert at generating circuit netlists for the Falstad circuit 
simulator (falstad.com/circuit).

CRITICAL RULES:

1. NEVER generate coordinates from scratch. Always ask me to export an 
   existing Falstad circuit as text first (File → Export as Text), then 
   modify only the component values. Keep all wire coordinates identical.

2. Falstad text format reference:
   $ 1 [timestep] [timescale] [zoom] [grid] [flags] 5e-11  ← header
   a x1 y1 x2 y2 flags V+ V- gain inV+ inV- output        ← op-amp
   r x1 y1 x2 y2 flags resistance                          ← resistor
   c x1 y1 x2 y2 flags capacitance initialVoltage 0 0      ← capacitor
   w x1 y1 x2 y2 0                                         ← wire
   g x1 y1 x2 y2 0 0                                       ← ground
   o elm# 8 0 flags scale 0.1 channel 2 elm# 3             ← scope probe

3. Op-amp: always use flag 8 (real/non-ideal model).

4. Capacitor initial voltage (7th field in c line):
   - Set to 0.5 for ALL oscillator circuits, NEVER 0
   - At 0 the simulator stays frozen (perfectly balanced, no noise)
   - 0.5 simulates the real-world thermal noise kick-start

5. Simulation speed = 3rd value in $ header line:
   - 10.20027730826997 = 1x real-time speed (use this as default)
   - Higher number = slower simulation

6. Scope probes (o lines):
   - Index = position of component in file, counting from 0
   - Two scopes need two o lines with channel 0 and channel 1

7. Common errors:
   "Singular matrix"    → a node has no path to ground
   "wire loop"          → duplicate wire creating a short  
   "Max=0V" flat scope  → capacitor initial voltage is 0, change to 0.5
   "0.3x speed"         → 3rd $ value too large, change to 10.2
```

---

## Verified Working Example: Schmitt Trigger Oscillator

This circuit works. Paste it into **File → Import from Text**:

```
$ 1 0.000005 10.20027730826997 50 5 50 5e-11
a 304 368 432 368 8 12 -12 1000000 0 0 100000
w 304 352 304 304 0
r 304 304 432 304 0 50000
w 432 304 432 368 0
w 432 368 432 480 0
r 432 480 304 480 0 10000
w 304 384 304 480 0
r 304 480 208 480 0 100000
g 208 480 208 512 0 0
c 304 304 208 304 4 0.000001 0.5 0 0
g 208 304 208 336 0 0
o 0 8 0 4098 1.25 0.00009765625 0 2 0 3
o 9 8 0 4098 9.607113551536424 0.0001 0 2 9 3
```

**Components:** Op-amp ±12V, RC resistor 50kΩ, feedback resistors 10kΩ and 100kΩ, capacitor 1µF  
**Scopes:** Green = square wave output, Yellow = capacitor charge/discharge ramp  
**To change frequency:** change `50000` on line 4 (the RC resistor). Frequency ≈ 1/(2RC·ln3)

---

## What the Scope Lines Mean

Each scope in Falstad shows two lines:
- **Green** = voltage waveform
- **Yellow** = current through that component

Right-click scope → Properties → uncheck "Show Current" to hide the yellow line.

---

## Why Oscillators Show 0V Until You Add the Kick

Falstad simulates ideal components with zero noise. A perfectly balanced oscillator at 0V stays at 0V forever — mathematically stable but physically wrong. In real life, thermal noise in resistors starts it instantly. Setting capacitor initial voltage to `0.5` simulates this. Without it, the circuit is technically "astable" but the simulator never discovers that.

---

## Key Insight: The $ Header

```
$ 1 0.000005 10.20027730826997 50 5 50 5e-11
        ↑           ↑
    timestep    timescale (controls simulation speed display)
```

The timescale value is auto-set by Falstad when you export — it's tuned to the circuit's natural frequency. If you paste a circuit and it runs at 0.3x, change the 3rd value to `10.20027730826997`.

---

## How This Was Found

Developed through iterative trial and error with Claude (Anthropic), June 2026, while working through a university op-amp comparator lab (Activities 1–5: comparator, Schmitt trigger, capacitance meter, oscillator).

The breakthrough: instead of asking the LLM to write netlists, ask it to *modify* netlists exported from Falstad's own built-in examples. The coordinates are already valid — only values need changing.

This directly solves the problem described in [this AllAboutCircuits forum thread](https://forum.allaboutcircuits.com/threads/conversion-of-netlist-to-schematic.206055/) which identified the coordinate problem but had no working answer.

---

## Quick Reference: Common Falstad Circuits to Start From

| Want to build | Start from (Circuits menu) |
|---|---|
| Oscillator | Circuits → Classics → Relaxation Oscillator |
| Amplifier | Circuits → Classics → Inverting Amplifier |
| Filter | Circuits → Classics → Low-pass Filter |
| Comparator | Circuits → Basics → Comparator |

Export whichever is closest, paste to LLM, modify values only.
