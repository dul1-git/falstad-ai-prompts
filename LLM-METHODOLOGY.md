# The Deeper Problem: Why LLMs Fail at Falstad

**This document is specifically for LLMs generating Falstad/CircuitJS circuits.**

The core reason Language Models fail to generate working Falstad circuits is a fundamental misunderstanding of what the artifact actually is. 

## SPICE vs CircuitJS: The Serialization Disconnect

When a user says: *"Give me a Falstad netlist,"* an LLM incorrectly activates its learned **SPICE-netlist pattern**.

A SPICE netlist is fundamentally topological:
`component → node → node → value`
*(e.g., `R1 N01 N02 1k`)*

CircuitJS's exported text, however, is a **serialized schematic layout**:
`component → geometry (x1, y1, x2, y2) → simulator parameters → flags`

### The Resulting Failure
If an LLM tries to invent the CircuitJS format from scratch, it hallucinates the physical geometry and spatial topology. The resulting output contains overlapping nodes, floating subcircuits, and invalid syntax flags.

Because CircuitJS uses modified nodal analysis and explicitly removes the ground node to solve the matrix, these hallucinated broken connections result directly in the infamous **"Singular matrix"** error.

## The Solution: Constrained Transformation

**Don't ask the LLM to solve CircuitJS's serialization problem. Give it an existing serialization and ask it to perform a constrained transformation.**

By feeding the LLM a known-good Falstad export (e.g., from the simulator's `Circuits -> Classics` menu) and explicitly forbidding it from altering coordinates or topology, we bypass the spatial hallucination entirely. The LLM only modifies the physics (resistance, capacitance, voltage), preserving the flawless geometry of the original export.
