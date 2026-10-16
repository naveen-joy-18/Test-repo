# BB84 Quantum Key Distribution (QKD) in Qiskit

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue.svg)](https://www.python.org/)
[![Qiskit](https://img.shields.io/badge/Qiskit-2.6.0-613394.svg)](https://qiskit.org/)
[![Qiskit Aer](https://img.shields.io/badge/Qiskit--Aer-0.17.2-613394.svg)](https://github.com/Qiskit/qiskit-aer)
[![Tests](https://img.shields.io/badge/Tests-9%20Passing-brightgreen.svg)](tests/test_bb84.py)

A pure **Qiskit** implementation of the **BB84 Quantum Key Distribution Protocol** (Bennett & Brassard, 1984), evaluating key generation across an **Ideal Channel** and five physical noise models simulated with **Qiskit Aer Noise**.

---

## Quickstart & Reproducibility

```bash
# 1. Clone repository and set up virtual environment
git clone https://github.com/your-username/BB84.git
cd BB84
python -m venv venv

# Windows:
.\venv\Scripts\activate
# Linux/macOS:
source venv/bin/activate

# 2. Install dependencies
pip install -r requirements.txt

# 3. Run unit tests (9 tests)
python -m unittest tests/test_bb84.py

# 4. Run full simulation suite (all 6 models + sweeps + plots)
python main.py
```

### Run a Specific Noise Channel
```bash
python main.py --noise-type bit_phase_flip --noise-prob 0.12
python main.py --noise-type amplitude_damping --noise-prob 0.12
python main.py --noise-type bit_flip --noise-prob 0.12
python main.py --noise-type phase_flip --noise-prob 0.12
python main.py --noise-type depolarizing --noise-prob 0.12
python main.py --noise-type none
```

---

## Comparative Results ($N=24, p=0.12$)

| Metric | Ideal Channel | Bit Flip ($X$) | Phase Flip ($Z$) | Bit & Phase Flip ($Y$) | Amplitude Damping | Depolarizing |
|---|---|---|---|---|---|---|
| **Noise Parameter ($p / \gamma$)** | `0.00` | `0.12` | `0.12` | `0.12` | `0.12` | `0.12` |
| **Transmitted Qubits** | 24 | 24 | 24 | 24 | 24 | 24 |
| **Sifted Key Length** | 11 bits | 11 bits | 11 bits | 11 bits | 11 bits | 11 bits |
| **Sifting Efficiency** | 45.83% | 45.83% | 45.83% | 45.83% | 45.83% | 45.83% |
| **Total Sifted QBER** | **0.00%** | **9.09%** | **9.09%** | **18.18%** | **9.09%** | **9.09%** |
| **$Z$-Basis Error Rate ($e_Z$)** | 0.00% | 16.67% | 0.00% | 16.67% | 16.67% | 16.67% |
| **$X$-Basis Error Rate ($e_X$)** | 0.00% | 0.00% | 20.00% | 20.00% | 0.00% | 0.00% |
| **Threshold ($\le 11.0\%$)** | **PASSED** | **PASSED** | **PASSED** | **EXCEEDED** | **PASSED** | **PASSED** |
| **Final Secret Key Length** | 9 bits | 9 bits | 9 bits | 9 bits (Aborted) | 9 bits | 9 bits |
| **Raw Key Bit Errors** | 0 | 1 | 1 | 1 | 1 | 1 |
| **Security Decision** | **100% Match** | **Reconcile** | **Reconcile** | **Abort Communication** | **Reconcile** | **Reconcile** |

> **Key Theoretical Takeaway**:
> - **Bit Flip ($X$)** flips computational states ($e_Z > 0$) but leaves $X$-basis eigenstates immune ($e_X = 0$).
> - **Phase Flip ($Z$)** flips diagonal superposition phases ($e_X > 0$) but leaves $Z$-basis eigenstates immune ($e_Z = 0$).
> - **Bit & Phase Flip ($Y = iXZ$)** attacks both bases simultaneously ($e_Z=16.7\%, e_X=20.0\%$), driving QBER to $18.18\% > 11\%$ (Protocol Abort).
> - **Amplitude Damping ($\mathcal{E}_{AD}$)** simulates asymmetric energy relaxation ($|1\rangle \to |0\rangle$), producing $Z$-basis errors while remaining secure for reconciliation ($9.09\% \le 11\%$).

---

## Generated Visualizations & Circuit Diagrams

All visual assets are rendered at 300 DPI in the `circuits/` folder:

| File | Type | Description |
|---|---|---|
| [`bb84_canonical_states.png`](circuits/bb84_canonical_states.png) | Circuit Primitive | Alice's 4 state preparations: $\|0\rangle, \|1\rangle, \|+\rangle, \|-\rangle$. |
| [`bb84_basis_combinations.png`](circuits/bb84_basis_combinations.png) | Circuit Primitive | 4 basis measurement scenarios (deterministic match vs 50/50 collapse). |
| [`bb84_full_circuit.png`](circuits/bb84_full_circuit.png) | Quantum Circuit | Complete 24-qubit BB84 transmission circuit for the Ideal Channel. |
| [`bb84_noisy_circuit.png`](circuits/bb84_noisy_circuit.png) | Quantum Circuit | 24-qubit circuit with channel identity/noise gates. |
| [`bb84_bit_phase_flip_circuit.png`](circuits/bb84_bit_phase_flip_circuit.png) | Quantum Circuit | Dedicated transmission circuit under Pauli $Y$ noise. |
| [`bb84_amplitude_damping_circuit.png`](circuits/bb84_amplitude_damping_circuit.png) | Quantum Circuit | Dedicated transmission circuit under $T_1$ energy relaxation noise. |
| [`key_distribution_analysis.png`](circuits/key_distribution_analysis.png) | Dashboard | Ideal channel stages, sifting efficiency, basis distribution, and key match. |
| [`bb84_bit_phase_flip_metrics_analysis.png`](circuits/bb84_bit_phase_flip_metrics_analysis.png) | Dashboard | Diagnostics for Bit & Phase Flip: QBER (18.18%) vs 11% threshold (Abort). |
| [`bb84_amplitude_damping_metrics_analysis.png`](circuits/bb84_amplitude_damping_metrics_analysis.png) | Dashboard | Diagnostics for Amplitude Damping: QBER (9.09%) $\le 11\%$ (Reconcile). |
| [`bit_phase_flip_and_amplitude_damping_analysis.png`](circuits/bit_phase_flip_and_amplitude_damping_analysis.png) | Comparative Plot | Side-by-side dashboard comparing Bit & Phase Flip vs Amplitude Damping. |
| [`noise_models_comparison_analysis.png`](circuits/noise_models_comparison_analysis.png) | Master Dashboard | 4-panel comparison of all 6 noise channels, basis errors, and security decisions. |
| [`all_noise_models_sweep.png`](circuits/all_noise_models_sweep.png) | Parametric Curve | Empirical QBER sweep ($p \in [0, 0.20]$) showing secure vs abort boundaries. |

---

## Project Structure

```text
BB84/
├── circuits/            # 22 high-resolution circuit diagrams and analytics charts
├── results/             # JSON metrics payloads and detailed bit audit TXT reports
├── src/
│   ├── bb84_protocol.py # Core BB84Protocol class (supports Qiskit Aer NoiseModel)
│   ├── visualize.py     # Qiskit circuit drawer & matplotlib analytical visualizers
│   └── utils.py         # Table formatting, data export, and comparison helpers
├── tests/
│   └── test_bb84.py     # 9 automated unit tests verifying all noise models
├── main.py              # CLI runner for full suite or single-channel runs
├── requirements.txt     # Pinned dependencies (Qiskit 2.6.0, Qiskit Aer 0.17.2)
└── README.md            # Minimal project overview and reproducibility guide
```

---

## CLI Options

| Argument | Type | Default | Description |
|---|---|---|---|
| `--qubits` | `int` | `24` | Number of qubits transmitted ($N$) |
| `--seed` | `int` | `10` | Random seed for reproducibility |
| `--noise-prob` | `float` | `0.12` | Channel noise parameter ($p$ or $\gamma$) |
| `--sample-ratio` | `float` | `0.25` | Fraction of sifted key checked for QBER |
| `--noise-type` | `str` | `all` | `all`, `none`, `bit_flip`, `phase_flip`, `bit_phase_flip`, `amplitude_damping`, `depolarizing` |
