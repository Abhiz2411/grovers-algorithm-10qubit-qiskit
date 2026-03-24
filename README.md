# Grover's Algorithm – 10-Qubit Qiskit Implementation

**Author:** Abhijit Zende &nbsp;|&nbsp; **ID:** 26PGAI0009 &nbsp;|&nbsp; **Course:** Quantum Computing – Quarter 5

---

## Overview

This project implements Grover's quantum search algorithm entirely from scratch using Qiskit. The goal is to locate two marked elements in a 10-qubit search space (2¹⁰ = 1024 possible states) by amplifying their measurement probability through repeated oracle + diffusion iterations.

All components — the oracle, diffusion operator, and Grover circuit — are built manually using primitive quantum gates. No built-in `qiskit.algorithms.Grover` or `grover_operator()` is used.

---

## Problem Setup

| Parameter | Value |
|---|---|
| Qubits | 10 |
| Search space | 2¹⁰ = 1024 states |
| Marked states | `0110011010` (decimal 410), `1101010001` (decimal 849) |
| Number of marked states (M) | 2 |
| Simulator | Qiskit Aer – `AerSimulator` |
| Shots per run | 8192 |
| Iteration counts studied | 1, 3, 5, 10, 17 (optimal) |

---

## Background

### Classical vs Quantum Search

A classical unstructured search over N items requires O(N) queries in the worst case — for N = 1024 that's up to 1024 checks. Grover's algorithm reduces this to O(√N) queries, achieving a quadratic speedup. For this problem:

- Classical worst case: **1024 queries**
- Grover's optimal: **≈ 25 queries** (√1024 ≈ 32, scaled by π/4)
- Speedup factor: **~30×**

### How Grover's Algorithm Works

Starting from an equal superposition of all N states, Grover's algorithm repeatedly applies two operations:

1. **Oracle (Uƒ):** Flips the phase of marked states — |ω⟩ → −|ω⟩ — without revealing which states are marked.
2. **Diffusion operator (Us):** Reflects amplitudes about their mean, amplifying states with negative phase (the marked ones) and suppressing all others.

Each iteration rotates the quantum state by 2θ in the 2D subspace spanned by |good⟩ and |bad⟩, where:

```
θ = arcsin(√(M/N)) = arcsin(√(2/1024)) ≈ 0.044 rad
```

After k iterations, the probability of measuring a marked state is:

```
P(marked) = sin²((2k + 1) · θ)
```

The optimal number of iterations before the probability starts decreasing:

```
k_opt = ⌊ π / (4θ) ⌋ = 17
```

---

## Circuit Architecture

### Oracle

For each marked state, the oracle applies a phase flip (−1) using the following gate sequence:

```
1. X gates on qubits where target bit = '0'   (maps target → |11…1⟩)
2. MCZ gate = H · MCX(q0..q8 → q9) · H       (phase flip of |11…1⟩)
3. X gates again                               (restore qubit values)
```

This is applied independently for each of the two marked states, so the oracle contains two such blocks back to back.

**Bit ordering note:** Qiskit uses little-endian convention — qubit `i` corresponds to character at position `n−1−i` in the bitstring. The oracle accounts for this mapping explicitly.

### Diffusion Operator

The diffusion operator implements **2|s⟩⟨s| − I** (inversion about the mean) using:

```
H⊗n → X⊗n → MCZ → X⊗n → H⊗n
```

The inner `MCZ` is again implemented as `H · MCX · H` on the last qubit. This transforms the phase flip of |11…1⟩ (after the X layer) into an effective phase flip of |00…0⟩, producing the desired inversion about the uniform superposition.

### Full Circuit (k iterations)

```
|0⟩⊗10 → H⊗10 → [Oracle → Diffusion]^k → Measure
```

---

## Theoretical Results

| k | P(marked) – Theory | Note |
|---|---|---|
| 1 | 0.0176 | Very low — algorithm just started |
| 3 | 0.0927 | Slowly building up |
| 5 | 0.2186 | Noticeable amplification |
| 10 | 0.6406 | Strong amplification |
| **17** | **0.9990** | **Optimal — near-certain measurement** |

---

## Observations

### Effect of Iteration Count

- **k = 1:** Both marked states are barely distinguishable from the background. Probability is spread nearly uniformly across all 1024 states (~0.1% each), with only a slight bump at the marked states.

- **k = 3:** Amplification begins to show. Marked states appear with ~4.6% probability each, already higher than any unmarked state, but the distribution is still noisy.

- **k = 5:** Clear separation. Marked states visually dominate the histogram with ~11% probability each. The algorithm is working, though not yet at peak.

- **k = 10:** Marked states appear with ~32% probability each (~64% combined), far above the remaining background noise. This is well past the halfway point on the convergence curve.

- **k = 17 (optimal):** Near-perfect amplification. The two marked states account for ~99% of all measurements. The histogram is almost a delta function at the two targets. Remaining ~1% is spread across all other 1022 states.

### Convergence Behaviour

The probability follows a sinusoidal pattern — it peaks at k = 17 and would decline if iterations continued beyond this point. This over-rotation effect is a key feature of Grover's algorithm: running too many iterations degrades performance, which is why knowing M (number of marked states) matters for choosing k.

### Oracle Design

Using the X-gate open-control trick avoids the need for an ancilla qubit. The MCZ gate (implemented via H–MCX–H) correctly applies a −1 phase to the target state without disturbing any other state. Both marked states are handled sequentially within the same oracle circuit, and the combined phase flip is verified by the simulation results.

### Simulator Accuracy

Simulated probabilities closely match theoretical predictions across all iteration counts. The small deviations are due to shot noise (finite sampling with 8192 shots). At k = 17, the simulated P(marked) consistently lands above 0.98.

---

## Files

```
├── Abhijit-Zende-Quantam_Project_grover_26PGAI0009.ipynb   # main notebook
├── requirements.txt                                         # Python dependencies
└── README.md
```

---

## Setup & Running

**Install dependencies:**

```bash
pip install -r requirements.txt
```

**Launch the notebook:**

```bash
jupyter notebook Abhijit-Zende-Quantam_Project_grover_26PGAI0009.ipynb
```

Run all cells top to bottom. No external data or configuration required.

**Notebook sections:**

| Cell | What it does |
|---|---|
| Imports | Loads Qiskit, Aer, matplotlib |
| Setup | Defines targets, computes θ, k_opt, prints theory table |
| Oracle | Defines and inspects `build_oracle()` |
| Diffusion | Defines and inspects `build_diffusion()` |
| Grover Circuit | Assembles full circuit for k iterations, draws structure |
| Simulation | Runs all 5 iteration counts, prints theory vs simulated table |
| Histograms | 5-panel bar chart — marked states highlighted in red |
| Convergence | Theory curve + simulated points + summary table |

**Output files generated:**
- `grover_results.png` — measurement histograms for all iteration counts
- `grover_convergence.png` — amplitude amplification convergence curve

---

## Requirements

- Python 3.10+
- qiskit >= 1.0
- qiskit-aer >= 0.14
- matplotlib >= 3.7
- ipykernel
- notebook
