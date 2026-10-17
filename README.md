# Quantum BB84 Communication

### Exploring Quantum Key Distribution Using the BB84 Protocol: From Ideal Communication to Noise-Induced Errors

**Team Members:**

- **Sonet D Thomas**
- **Naveen Joy**

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue.svg?logo=python&logoColor=white)](https://www.python.org/)
[![Qiskit](https://img.shields.io/badge/Qiskit-2.6.0-613394.svg?logo=qiskit&logoColor=white)](https://qiskit.org/)
[![Qiskit Aer](https://img.shields.io/badge/Qiskit--Aer-0.17.2-613394.svg)](https://github.com/Qiskit/qiskit-aer)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Tests](https://img.shields.io/badge/Unit%20Tests-9%20Passing-brightgreen.svg)](tests/test_bb84.py)
[![Protocol](<https://img.shields.io/badge/Protocol-BB84%20(1984)-orange.svg>)](https://doi.org/10.1016/j.tcs.2014.05.025)

---

## 1. Problem Statement & Proposed Solution

### Problem Statement

To develop a Quantum Key Distribution (QKD) framework using the BB84 protocol, investigate the effects of different noise models, and explore pathways toward reliable and secure quantum communication.

### Proposed Solution

- **Pure Qiskit BB84 Framework**: Built an end-to-end 24-qubit BB84 simulation using **Qiskit** and **Qiskit Aer** to demonstrate how Alice and Bob generate shared random secret keys using conjugate non-orthogonal bases ($Z$ and $X$).
- **Multi-Noise Channel Evaluation**: Systematically modeled and simulated five physical quantum noise channels via Kraus operators at $p = 0.12$:
  1. **Bit Flip ($X$)**: Flips $|0\rangle \leftrightarrow |1\rangle$ in the computational basis.
  2. **Phase Flip ($Z$)**: Inverts superposition phase.
  3. **Bit & Phase Flip ($Y / XZ$)**: Simultaneously perturbs bit and phase.
  4. **Amplitude Damping ($\mathcal{E}_{AD}$)**: Models physical $T_1$ energy relaxation ($|1\rangle \to |0\rangle$) with asymmetric decay, remaining secure for reconciliation.
  5. **Depolarizing Noise**: Uniform isotropic degradation.
- **Shor-Preskill Security Threshold Benchmarking**: Evaluated measured Quantum Bit Error Rate (QBER) against the theoretical **11.0% Shor-Preskill bound**.

```
+---------------------------------------------------------------------------------------------------+
|                                  BB84 PROTOCOL WORKFLOW                                           |
+---------------------------------------------------------------------------------------------------+
|  ALICE (Transmitter)               QUANTUM CHANNEL                BOB (Receiver)                  |
|  -------------------               ---------------                --------------                  |
|  [Random Bit in {0,1}]                                                                            |
|          |                                                                                        |
|  [Random Basis in {Z,X}]                                          [Random Basis in {Z,X}]         |
|          |                                                                      |                 |
|  [Quantum State Prep] ----------> (Qiskit Aer Noise Model) ------> [Measurement Projection]       |
|   |0>, |1>, |+>, |->               Bit Flip, Phase Flip,            In basis beta                 |
|                                    Amplitude Damping, etc.                      |                 |
|                                                                    [Raw Bit Outcome]              |
|                                                                                                   |
|  PUBLIC CLASSICAL CHANNEL:                                                                        |
|  [Basis Sifting] <================== Sift where alpha == beta ==================> [Basis Sifting] |
|  [Sifted Key A]                                                                   [Sifted Key B]  |
|  [Sample Error Check] <============= Sample QBER Estimation ===============> [Sample Error Check] |
|                                                                                                   |
|  CRYPTOGRAPHIC DECISION:                                                                          |
|  If QBER <= 11.0%: Proceed to Information Reconciliation (Cascade/LDPC) & Privacy Amplification   |
|  If QBER >  11.0%: ABORT SESSION (Channel compromised by noise or eavesdropper Eve)               |
+---------------------------------------------------------------------------------------------------+
```

---

## 2. Setup & Installation Steps

### Prerequisites

- **Python 3.10+** (64-bit)
- **Git**

### Step-by-Step Environment Setup

```bash
# 1. Clone the repository
git clone https://github.com/your-username/BB84.git
cd BB84

# 2. Create an isolated virtual environment
python -m venv venv

# 3. Activate the virtual environment
# Windows (PowerShell):
.\venv\Scripts\Activate.ps1
# Windows (Command Prompt):
venv\Scripts\activate.bat
# Linux / macOS:
source venv/bin/activate

# 4. Install dependencies
pip install --upgrade pip
pip install -r requirements.txt
```

### Verify Environment

Run this one-line command to verify the installation:

```bash
python -c "import qiskit, qiskit_aer, matplotlib, numpy; print(f'Environment OK: Qiskit {qiskit.__version__} | Aer {qiskit_aer.__version__}')"
```

_Expected output: `Environment OK: Qiskit 2.6.0 | Aer 0.17.2`_

---

## 3. How to Run the Code (Demo Instructions)

### 1. Run Automated Unit Tests (9 Tests)

Verifies protocol logic, basis matching, and all noise models:

```bash
python -m unittest tests/test_bb84.py
```

_Expected output: `Ran 9 tests in ~1.8s ... OK`_

### 2. Run the Full Multi-Noise Simulation Demo

Executes all 6 configurations (Ideal + 5 Noise Channels), outputs side-by-side terminal comparison tables, exports JSON/TXT reports, and generates all visualizations:

```bash
python main.py
```

### 3. Run Individual Noise Channel Demos

Use `--noise-type` to demonstrate any specific channel:

```bash
# Bit & Phase Flip (Y) - Demonstrates Protocol Abort (QBER = 18.18% > 11%)
python main.py --noise-type bit_phase_flip --noise-prob 0.12

# Amplitude Damping - Demonstrates T1 energy decay (|1> -> |0>, QBER = 9.09% <= 11%)
python main.py --noise-type amplitude_damping --noise-prob 0.12

# Bit Flip (X) - Demonstrates Z-basis error with X-basis eigenstate immunity
python main.py --noise-type bit_flip --noise-prob 0.12

# Phase Flip (Z) - Demonstrates X-basis error with Z-basis eigenstate immunity
python main.py --noise-type phase_flip --noise-prob 0.12

# Depolarizing Channel - Demonstrates symmetric degradation
python main.py --noise-type depolarizing --noise-prob 0.12

# Ideal Channel - Demonstrates 100% clean baseline key exchange (QBER = 0%)
python main.py --noise-type none
```

### CLI Options Reference

| Argument         | Type    | Default | Description                                                                                                    |
| ---------------- | ------- | ------- | -------------------------------------------------------------------------------------------------------------- |
| `--qubits`       | `int`   | `24`    | Total number of quantum bits transmitted ($N$)                                                                 |
| `--seed`         | `int`   | `10`    | Random seed for exact reproducibility                                                                          |
| `--noise-prob`   | `float` | `0.12`  | Channel noise parameter ($p$ or $\gamma$)                                                                      |
| `--sample-ratio` | `float` | `0.25`  | Proportion of sifted key checked for QBER estimation                                                           |
| `--noise-type`   | `str`   | `all`   | Channel to run: `all`, `none`, `bit_flip`, `phase_flip`, `bit_phase_flip`, `amplitude_damping`, `depolarizing` |

---

## 4. Explanation of the Output

Running the demo produces three outputs: **Terminal Audit Tables**, **Saved Visualizations (`circuits/`)**, and **Data Reports (`results/`)**.

### 4.1 Terminal Output: Stage-by-Stage Transmission Table

Traces each qubit from Alice's state preparation to Bob's measurement:

```text
---------------------------------------------------------------------------------------------------
 Qubit | Alice Bit | Alice Basis |  State  | Bob Basis | Bob Result | Basis Match? | Sifted Result
---------------------------------------------------------------------------------------------------
   0   |     0     |      Z      |   |0>   |     X     |     0      |   MISMATCH   | -- (Discard)
   3   |     0     |      Z      |   |0>   |     Z     |     0      |  MATCH (OK)  |   0 (Valid)
   4   |     0     |      X      |   |+>   |     X     |     0      |  MATCH (OK)  |   0 (Valid)
  11   |     1     |      Z      |   |1>   |     Z     |     0      |ERROR (NOISE) |   A:1!=B:0
  14   |     1     |      X      |   |->   |     X     |     1      |  MATCH (OK)  |   1 (Valid)
---------------------------------------------------------------------------------------------------
```

- **MISMATCH**: Bases differed (Qubit 0: Alice $Z$, Bob $X$). The state collapses randomly and is discarded. Sifting efficiency is $\approx 50\%$.
- **MATCH (OK)**: Bases matched; transmission succeeded without disturbance.
- **ERROR (NOISE)**: Bases matched, but channel noise corrupted the flying qubit (Qubit 11: Alice sent $|1\rangle$, noise caused relaxation/flip so Bob measured `0`).

### 4.2 Comparative Results Across All Noise Models ($N=24, p=0.12$)

```text
===================================================================================================================
                                    BB84 NOISE MODELS COMPARATIVE SUMMARY
===================================================================================================================
----------------------------------------------------------------------------------------------------------------------------------------------------------
Metric                       |   Ideal Channel    |    Bit Flip (X)    |   Phase Flip (Z)   |  Bit & Phase Flip  | Amplitude Damping  |    Depolarizing
----------------------------------------------------------------------------------------------------------------------------------------------------------
Noise Probability (p)        |        0.00        |        0.12        |        0.12        |        0.12        |        0.12        |        0.12
Transmitted Qubits           |         24         |         24         |         24         |         24         |         24         |         24
Sifted Key Length            |      11 bits       |      11 bits       |      11 bits       |      11 bits       |      11 bits       |      11 bits
Sifting Efficiency           |       45.83%       |       45.83%       |       45.83%       |       45.83%       |       45.83%       |       45.83%
Sample QBER                  |        0.0%        |        0.0%        |        0.0%        |       50.0%        |        0.0%        |        0.0%
Total Sifted QBER            |        0.0%        |       9.09%        |       9.09%        |       18.18%       |       9.09%        |       9.09%
Z-Basis QBER                 |        0.0%        |       16.67%       |        0.0%        |       16.67%       |       16.67%       |       16.67%
X-Basis QBER                 |        0.0%        |        0.0%        |       20.0%        |       20.0%        |        0.0%        |        0.0%
Threshold (11.0%)            |       PASSED       |       PASSED       |       PASSED       |      EXCEEDED      |       PASSED       |       PASSED
Final Key Length             |       9 bits       |       9 bits       |       9 bits       |       9 bits       |       9 bits       |       9 bits
Raw Key Errors               |         0          |         1          |         1          |         1          |         1          |         1
Key Integrity                |    MATCH (100%)    |     RECONCILE      |     RECONCILE      |     RECONCILE      |     RECONCILE      |     RECONCILE
----------------------------------------------------------------------------------------------------------------------------------------------------------
```

### 4.3 Quantum Mechanical Basis Asymmetry

- **Bit Flip ($X$) Asymmetry**: Produces errors in $Z$-basis ($e_Z = 16.7\%$), while $X$-basis states are **immune** ($e_X = 0.0\%$) as eigenstates of $X$.
- **Phase Flip ($Z$) Asymmetry**: Produces errors in $X$-basis ($e_X = 20.0\%$), while $Z$-basis states are **immune** ($e_Z = 0.0\%$) as eigenstates of $Z$.
- **Bit & Phase Flip ($Y / XZ$) Dual Vulnerability**: Disrupts both bases, elevating QBER to **18.18%** and forcing a **Protocol Abort**.
- **Amplitude Damping ($\mathcal{E}_{AD}$)**: Simulates physical $T_1$ energy relaxation ($|1\rangle \to |0\rangle$). Produces asymmetric $Z$-basis decay ($e_Z = 16.7\%$), remaining within the secure ceiling for reconciliation ($9.09\% \le 11.0\%$).

### 4.4 Generated Visual Outputs (`circuits/` & `svg/` folders)

All figures are automatically saved in both high-resolution PNG (300 DPI, in `circuits/`) and scalable vector graphic format (in `svg/`):

| Figure File Base                                | Type              | Description                                                                                                                                                                                                 |
| ----------------------------------------------- | ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `bb84_canonical_states`                         | Circuit Primitive | Alice's 4 state preparation circuits: $\|0\rangle, \|1\rangle, \|+\rangle, \|-\rangle$. ([PNG](circuits/bb84_canonical_states.png) / [SVG](svg/bb84_canonical_states.svg))                                  |
| `bb84_basis_combinations`                       | Circuit Primitive | 4 measurement scenarios: deterministic match vs 50/50 state collapse. ([PNG](circuits/bb84_basis_combinations.png) / [SVG](svg/bb84_basis_combinations.svg))                                                |
| `bb84_full_circuit`                             | Quantum Circuit   | Clean 24-qubit BB84 transmission circuit for the Ideal Channel. ([PNG](circuits/bb84_full_circuit.png) / [SVG](svg/bb84_full_circuit.svg))                                                                  |
| `bb84_noisy_circuit`                            | Quantum Circuit   | 24-qubit circuit with channel identity/noise gates. ([PNG](circuits/bb84_noisy_circuit.png) / [SVG](svg/bb84_noisy_circuit.svg))                                                                            |
| `bb84_bit_phase_flip_circuit`                   | Quantum Circuit   | Dedicated 24-qubit circuit for Pauli $Y$ noise. ([PNG](circuits/bb84_bit_phase_flip_circuit.png) / [SVG](svg/bb84_bit_phase_flip_circuit.svg))                                                              |
| `bb84_amplitude_damping_circuit`                | Quantum Circuit   | Dedicated 24-qubit circuit for Amplitude Damping. ([PNG](circuits/bb84_amplitude_damping_circuit.png) / [SVG](svg/bb84_amplitude_damping_circuit.svg))                                                      |
| `key_distribution_analysis`                     | Dashboard         | Ideal Channel stages, sifting efficiency pie, basis counts, and 100% key match. ([PNG](circuits/key_distribution_analysis.png) / [SVG](svg/key_distribution_analysis.svg))                                  |
| `bb84_bit_phase_flip_metrics_analysis`          | Dashboard         | Bit & Phase Flip diagnostics: QBER ($18.18\%$) vs 11% threshold (Abort). ([PNG](circuits/bb84_bit_phase_flip_metrics_analysis.png) / [SVG](svg/bb84_bit_phase_flip_metrics_analysis.svg))                   |
| `bb84_amplitude_damping_metrics_analysis`       | Dashboard         | Amplitude Damping diagnostics: QBER ($9.09\%$) vs 11% threshold (Secure). ([PNG](circuits/bb84_amplitude_damping_metrics_analysis.png) / [SVG](svg/bb84_amplitude_damping_metrics_analysis.svg))            |
| `bit_phase_flip_and_amplitude_damping_analysis` | Comparative Plot  | Side-by-side dashboard directly comparing BPF vs AD and basis asymmetry. ([PNG](circuits/bit_phase_flip_and_amplitude_damping_analysis.png) / [SVG](svg/bit_phase_flip_and_amplitude_damping_analysis.svg)) |
| `noise_models_comparison_analysis`              | Master Dashboard  | 4-panel comparison of all 6 models, basis error breakdowns, and security decisions. ([PNG](circuits/noise_models_comparison_analysis.png) / [SVG](svg/noise_models_comparison_analysis.svg))                |
| `all_noise_models_sweep`                        | Parametric Curve  | Empirical sweep ($p \in [0, 0.20]$) showing QBER response curves against the 11% threshold. ([PNG](circuits/all_noise_models_sweep.png) / [SVG](svg/all_noise_models_sweep.svg))                            |

### 4.5 Generated Data Artifacts (`results/` folder)

- **`bb84_all_noise_models_comparison.json`**: Complete machine-readable JSON payload containing all parameters, sifted keys, basis counts, and QBER metrics.
- **`bb84_all_noise_models_comparison.txt`**: Clean ASCII comparative table ready for presentation.
- **Individual Channel Files**: `bb84_<noise_type>_run_summary.json` and `bb84_<noise_type>_detailed_report.txt` for all channels.

---

## 5. Repository Structure

```text
BB84/
├── circuits/                                                    # High-resolution PNG diagrams (DPI=300)
│   ├── all_noise_models_sweep.png                              # Empirical multi-noise sweep curves
│   ├── bb84_amplitude_damping_circuit.png                      # Dedicated Amplitude Damping circuit diagram
│   ├── bb84_amplitude_damping_metrics_analysis.png             # Dedicated Amplitude Damping analytics dashboard
│   ├── bb84_basis_combinations.png                             # 4 basis measurement scenarios
│   ├── bb84_bit_flip_circuit.png                               # Dedicated Bit Flip circuit diagram
│   ├── bb84_bit_flip_metrics_analysis.png                      # Dedicated Bit Flip analytics dashboard
│   ├── bb84_bit_phase_flip_circuit.png                         # Dedicated Bit & Phase Flip circuit diagram
│   ├── bb84_bit_phase_flip_metrics_analysis.png                # Dedicated Bit & Phase Flip analytics dashboard
│   ├── bb84_canonical_states.png                               # Canonical BB84 state preparations
│   ├── bb84_depolarizing_circuit.png                           # Dedicated Depolarizing circuit diagram
│   ├── bb84_depolarizing_metrics_analysis.png                  # Dedicated Depolarizing analytics dashboard
│   ├── bb84_full_circuit.png                                   # Ideal transmission circuit diagram
│   ├── bb84_noisy_circuit.png                                  # Generic noisy transmission circuit diagram
│   ├── bb84_phase_flip_circuit.png                             # Dedicated Phase Flip circuit diagram
│   ├── bb84_phase_flip_metrics_analysis.png                    # Dedicated Phase Flip analytics dashboard
│   ├── bit_phase_flip_and_amplitude_damping_analysis.png       # Comparative dashboard: BPF vs AD
│   ├── key_distribution_analysis.png                           # Ideal channel protocol analytics
│   └── noise_models_comparison_analysis.png                    # Multi-noise 4-panel master dashboard
├── svg/                                                         # Scalable Vector Graphics (SVG vector assets)
│   ├── all_noise_models_sweep.svg                              # Vector multi-noise sweep curves
│   ├── bb84_amplitude_damping_circuit.svg                      # Vector Amplitude Damping circuit diagram
│   ├── bb84_amplitude_damping_metrics_analysis.svg             # Vector Amplitude Damping analytics dashboard
│   ├── bb84_basis_combinations.svg                             # Vector basis measurement scenarios
│   ├── bb84_bit_flip_circuit.svg                               # Vector Bit Flip circuit diagram
│   ├── bb84_bit_flip_metrics_analysis.svg                      # Vector Bit Flip analytics dashboard
│   ├── bb84_bit_phase_flip_circuit.svg                         # Vector Bit & Phase Flip circuit diagram
│   ├── bb84_bit_phase_flip_metrics_analysis.svg                # Vector Bit & Phase Flip analytics dashboard
│   ├── bb84_canonical_states.svg                               # Vector canonical BB84 state preparations
│   ├── bb84_depolarizing_circuit.svg                           # Vector Depolarizing circuit diagram
│   ├── bb84_depolarizing_metrics_analysis.svg                  # Vector Depolarizing analytics dashboard
│   ├── bb84_full_circuit.svg                                   # Vector ideal transmission circuit diagram
│   ├── bb84_noisy_circuit.svg                                  # Vector generic noisy transmission circuit diagram
│   ├── bb84_phase_flip_circuit.svg                             # Vector Phase Flip circuit diagram
│   ├── bb84_phase_flip_metrics_analysis.svg                    # Vector Phase Flip analytics dashboard
│   ├── bit_phase_flip_and_amplitude_damping_analysis.svg       # Vector comparative dashboard: BPF vs AD
│   ├── key_distribution_analysis.svg                           # Vector ideal channel protocol analytics
│   └── noise_models_comparison_analysis.svg                    # Vector multi-noise 4-panel master dashboard
├── results/                                                     # JSON payloads & bit audit reports
│   ├── bb84_all_noise_models_comparison.json                   # Consolidated multi-model comparison JSON
│   ├── bb84_all_noise_models_comparison.txt                    # Formatted multi-model comparative table
│   ├── bb84_bit_phase_flip_run_summary.json                    # Bit & Phase Flip JSON metrics
│   ├── bb84_bit_phase_flip_detailed_report.txt                 # Bit & Phase Flip bit audit report
│   ├── bb84_amplitude_damping_run_summary.json                 # Amplitude Damping JSON metrics
│   ├── bb84_amplitude_damping_detailed_report.txt              # Amplitude Damping bit audit report
│   ├── bb84_bit_flip_run_summary.json                          # Bit Flip JSON metrics
│   ├── bb84_phase_flip_run_summary.json                        # Phase Flip JSON metrics
│   ├── bb84_depolarizing_run_summary.json                      # Depolarizing JSON metrics
│   └── bb84_ideal_run_summary.json                             # Ideal channel JSON metrics
├── src/
│   ├── __init__.py                                             # Package initializer
│   ├── bb84_protocol.py                                        # Core BB84Protocol class (supports NoiseModel)
│   ├── visualize.py                                            # Circuit drawers and analytical visualizers (PNG + SVG)
│   └── utils.py                                                # Table formatting, data export, and helpers
├── tests/
│   └── test_bb84.py                                            # 9 automated unit tests verifying all noise models
├── main.py                                                     # Master CLI runner (Full study or single-model)
├── requirements.txt                                            # Pinned dependencies (Qiskit 2.6.0, Aer 0.17.2)
└── README.md                                                   # Complete project documentation
```
