# Falstad Circuit Simulator: LLM Netlist Generation Methodology

> **Keywords:** Falstad circuit simulator, LLM netlist generation, ChatGPT circuit, Claude circuit simulator, falstad.com/circuit text format, circuitjs netlist, AI circuit simulation, Falstad import export, singular matrix fix, Falstad oscillator

Automated generation of Falstad circuit netlists using Large Language Models (LLMs) typically results in fatal errors (e.g., "Singular matrix", "wire loop detected", or flat 0V scopes). This occurs because the `.txt` export format relies on absolute pixel coordinates. LLMs cannot reliably synthesize spatial grid layouts from scratch.

This methodology bypasses spatial hallucination by constraining the LLM to modify existing topologies rather than generating new ones.

---

## The Workflow

**Do not prompt an LLM to generate a Falstad netlist from scratch.**

1. Open [falstad.com/circuit](https://falstad.com/circuit).
2. Navigate to **Circuits** and load the closest topological match (e.g., **Classics → Relaxation Oscillator**).
3. Select **File → Export as Text** and copy the netlist.
4. Provide the netlist to the LLM with instructions to modify component values only.
5. Paste the output back via **File → Import from Text**.

By locking the coordinate geometry and restricting the LLM to value manipulation, structural integrity is guaranteed.

---

## LLM System Prompt

Supply this prompt to configure the LLM before providing the baseline netlist:

```text
You are a Falstad circuit simulator (circuitjs) netlist engineer.

CRITICAL CONSTRAINTS:
1. NEVER generate spatial coordinates. You will be provided an exported Falstad netlist. Modify component values only. Leave all x/y coordinates identical.
2. Falstad format reference:
   $ 1 [timestep] [timescale] [zoom] [grid] [flags] 5e-11
   a x1 y1 x2 y2 flags V+ V- gain inV+ inV- output (Op-amp)
   r x1 y1 x2 y2 flags resistance (Resistor)
   c x1 y1 x2 y2 flags capacitance initial_V 0 0 (Capacitor)
   w x1 y1 x2 y2 0 (Wire)
   g x1 y1 x2 y2 0 0 (Ground)
   o elm# 8 0 flags scale 0.1 channel 2 elm# 3 (Scope)

3. Op-amps: Enforce flag 8 (real/non-ideal model).
4. Capacitors (Oscillators): The 7th field (initial_V) must be set to 0.5. At 0.0, the simulator remains mathematically locked. 0.5V introduces necessary asymmetry to initiate oscillation.
5. Timescale: The 3rd value in the $ header dictates simulation speed. Default to 10.20027730826997 for 1x real-time.
6. Scopes (o lines): Index references the component's zero-indexed position in the file.
```

---

## The 0V Simulator Lock

Ideal simulation environments lack thermal noise. A perfectly balanced astable circuit initializes at 0V and remains mathematically locked, despite being physically unstable. Real-world oscillation is initiated by microvolt thermal noise. 

To bridge this gap in the simulator, the initial capacitor voltage must be explicitly declared (e.g., `0.5V`). This breaks mathematical symmetry and accurately initiates the oscillation cascade.

---

## Verified Implementation: Schmitt Trigger Oscillator

The following netlist demonstrates the methodology. It resolves the zero-state lock and utilizes real op-amp models.

```text
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

**Parameters:**
- Op-amp: ±12V
- RC Resistor: 50kΩ (Line 4). Alter this to adjust frequency: `f ≈ 1 / (2RC · ln3)`
- Feedback network: 10kΩ and 100kΩ
- Capacitor: 1µF (0.5V initialization)
```
