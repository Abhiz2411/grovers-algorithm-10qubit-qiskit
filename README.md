# Grover's Algorithm – 10-Qubit Qiskit Implementation

**Author:** Abhijit Zende
**ID:** 26PGAI0009
**Course:** Quantum Computing – Quarter 5

---

## Overview

Implementation of Grover's search algorithm from scratch using Qiskit to locate two marked states in a 10-qubit (1024-state) search space.

- **Marked states:** `0110011010` (decimal 410) and `1101010001` (decimal 849)
- **No built-in** `qiskit.algorithms.Grover` used — oracle and diffuser are built manually with quantum gates
- Runs for k = 1, 3, 5, 10, and optimal (k = 17) iterations

---

## Files

```
├── Abhijit-Zende-Quantam_Project_grover_26PGAI0009.ipynb   # main notebook
├── requirements.txt
└── README.md
```

---

## Setup

```bash
pip install -r requirements.txt
```

Then launch the notebook:

```bash
jupyter notebook Abhijit-Zende-Quantam_Project_grover_26PGAI0009.ipynb
```

---

## How to Run

Run all cells top to bottom. The notebook is self-contained — no external data or config needed.

**What each section does:**

| Section | Description |
|---|---|
| Setup | Defines search space, computes optimal k, prints theoretical P(marked) per iteration |
| Oracle | Phase-flips the two marked states using X + MCZ gates |
| Diffusion | Implements 2\|s⟩⟨s\| − I via H → X → MCZ → X → H |
| Grover Circuit | Assembles oracle + diffuser for k iterations |
| Simulation | Runs all iteration counts on Aer simulator (8192 shots each) |
| Histograms | Measurement distributions for k = 1, 3, 5, 10, 17 — marked states in red |
| Convergence | Theory vs simulated P(marked) curve across iterations |

---

## Expected Output

- Histograms showing `0110011010` and `1101010001` with the highest counts
- At k = 17 (optimal), P(measuring a marked state) ≈ 99%
- `grover_results.png` and `grover_convergence.png` saved in the working directory

---

## Requirements

- Python 3.10+
- qiskit >= 1.0
- qiskit-aer >= 0.14
- matplotlib >= 3.7
