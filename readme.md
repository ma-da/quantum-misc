# Quantum Hash Search Experiments — Session Report

**Date:** November 6, 2026  
**Platform:** IBM Quantum, ibm_marrakesh (156 physical qubits)  
**Software:** Python / Jupyter, Qiskit 2.6.2, IBM Runtime 0.51.0  
**Project:** Experimental quantum acceleration of cryptographic-style hash inversion

## 1. Research Objective

Investigate whether Grover's quantum search algorithm can recover unknown hash inputs more efficiently than conventional brute-force search, using an actual physical quantum computer rather than simulation.

We developed a family of miniature nonlinear hash functions (Hash3 through Hash10), each with a search space of \(2^n\) possible inputs.

The long-term target is Hash10, involving 1,024 possible inputs. An ideal Grover search requires approximately 25 iterations versus 512.5 classical evaluations on average.

The experiments distinguish theoretical quantum query advantage from actual hardware performance, including gate errors and execution overhead.

## 2. Experimental Setup

IBM Quantum's free Open Plan provided access to three 156-qubit processors. All physical experiments used **ibm_marrakesh**.

Two Grover oracle designs were tested:

- **Full hash oracle:** Reversibly computes HashN, checks the output against a target hash, marks matching inputs, and uncomputes the hash. This constitutes an actual miniature hash-preimage search.
- **Direct-marking control:** Marks the known correct input without computing the hash. This tests quantum amplitude amplification with considerably fewer gates.

Circuits were compiled using Qiskit's optimization-level-3 transpiler, and physical experiments generally used 2,048 measurement shots per circuit.

## 3. Physical Hardware Results

| Experiment | 2Q depth | Measured success | Random expectation |
|---|---:|---:|---:|
| Hash3, full oracle | 91 | 30.08% | 12.50% |
| Hash4, original full oracle (initial) | 163 | 6.64% | 6.25% |
| Hash4, direct control | 69 | 24.32% | 6.25% |
| Hash5, direct control | 103 | 10.60% | 3.13% |
| Hash6, direct control | 147 | 2.83% | 1.56% |
| Hash8, direct control | 211 | 0.59% | 0.39% |
| Hash8, full oracle | 395 | 0.20% | 0.39% |
| Hash4, original full oracle (retest) | 153 | 10.74% | 6.25% |
| Hash4, RCCX optimized (seed 1) | 110 | 11.04% | 6.25% |
| **Hash4, RCCX optimized (seed 2)** | **107** | **13.77%** | **6.25%** |

**Principal findings:**

- Physical Grover amplification was successfully observed with full Hash3 and Hash4 oracles.
- Direct-marking searches demonstrated statistically meaningful amplification through six input bits (64 possibilities).
- Larger circuits experienced dramatic performance degradation, with Hash8 showing no detectable amplification.
- Full hash computation introduces substantial overhead relative to direct marking.
- Different transpilation choices and hardware execution conditions can materially affect success rates.

These experiments demonstrate quantum amplitude amplification, but not faster end-to-end execution than classical computing.

## 4. Major Breakthrough: Reducing Circuit Depth

The most promising development was replacing conventional Toffoli (CCX) operations with relative-phase Toffoli (RCCX) operations in the reversible hash and phase-marking circuitry.

RCCX can implement certain reversible computations using fewer expensive two-qubit gates, with appropriate handling of relative phases.

**Best compiled Hash4 comparison:**

| Metric | Original | RCCX optimized | Improvement |
|---|---:|---:|---:|
| Two-qubit depth | 153 | 107 | 30.1% reduction |
| Two-qubit gates | 241 | 178 | 26.1% reduction |
| Physical recovery rate, same-job comparison | 10.74% | 13.77% | 28.2% relative improvement |

The optimized circuit recovered the correct Hash4 input 282 times in 2,048 shots, compared with 128 expected from uniform random guessing.

This represents a **2.20× amplification over chance** using a circuit that actually computes the reversible hash.

However, the improvement over the original circuit may also involve physical qubit placement and calibration effects. The complete optimized circuit's equivalence to the original still needs explicit confirmation in the recorded results.

## 5. Interpretation

The principal limitation observed was not insufficient physical qubit capacity, but **the increasing depth and error sensitivity of quantum circuits**.

A 156-qubit processor could execute our 22-qubit Hash8 circuit, but its roughly 395 two-qubit layers left no detectable search advantage.

Conversely, reducing the Hash4 circuit from 153 to 107 layers coincided with improved physical results.

The evidence strongly motivates circuit-depth optimization, although gate counts, connectivity, calibration, and qubit fidelity must also be considered.

## 6. Next Session: Optimization Roadmap

**Primary objective: Further reduce two-qubit circuit depth while preserving correct reversible hash computation.**

Priorities:

1. **Verify mathematical equivalence.** Confirm original and RCCX-optimized quantum states agree for Hash4–Hash6 before further hardware runs.
2. **Optimize multi-controlled gates.** Investigate alternative decompositions for hash comparisons, phase marking, and Grover diffusion.
3. **Reduce compute–uncompute overhead.** Explore phase-oracle constructions, relative-phase cancellation, ancilla reuse, and alternative reversible hash designs.
4. **Optimize hardware mapping.** Evaluate transpiler seeds, physical qubit layouts, connectivity, and calibration-dependent gate errors.
5. **Measure improvements locally first.** Compare two-qubit depth, gate count, logical equivalence, and ideal recovery probability before spending QPU time.
6. **Resume physical experiments.** Test the best full Hash4 design, then extend successful optimizations to Hash5, Hash6, and eventually Hash8–Hash10.

An initial engineering target is to reduce full Hash4 depth below 90 layers, preferably approaching the 69-layer direct-marking control.

## 7. Current Status

**Established:** Real quantum search amplification using reversible miniature hash functions on IBM quantum hardware.

**Most promising technique:** Relative-phase Toffoli optimization, producing approximately 30% lower two-qubit circuit depth in Hash4.

**Outstanding challenge:** Preserve reliable amplitude amplification as hash size and oracle complexity increase.

**Long-term objective:** Demonstrate the largest practical reversible hash-input search achievable on IBM's physical quantum processors, with independently reproducible classical and quantum benchmarks.

The next phase should prioritize **smarter, shallower quantum circuits rather than simply increasing the number of qubits or Grover iterations**.
