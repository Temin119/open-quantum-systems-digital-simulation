# Reproducing Open Quantum-System Dynamics on Digital Quantum Circuits

This repository reproduces and extends selected results from:

**M. Jo and M. S. Kim, _Simulating open quantum many-body systems using optimised circuits in digital quantum simulation_ (2022).**

The project starts from simple spin dynamics and builds step by step toward a digital simulation of a dissipative two-spin transverse Ising model using Lindblad dynamics, quantum trajectories, an optimised modified stochastic Schrödinger equation (MSSE), and ancilla-assisted quantum circuits.

The final notebook adds an original extension: studying how the preferred simulation timestep changes when depolarising gate noise is included.

---

## Project aims

The main goals are to:

- build the quantum-mechanical foundations needed for the paper;
- reproduce open-system dynamics with the Lindblad master equation;
- reproduce the paper's stochastic-trajectory / MSSE approach;
- implement the dynamics as a gate-based digital quantum circuit;
- compare exact, stochastic and circuit-based simulations;
- investigate the trade-off between timestep size, circuit depth and hardware noise.

---

## Repository structure

```text
open-quantum-systems-digital-simulation/
│
├── README.md
├── requirements.txt
│
├── notebooks/
│   ├── 01_spin_operators.ipynb
│   ├── 02_two_spin_dynamics.ipynb
│   ├── 03_lindblad_qutip.ipynb
│   ├── 04_reproduce_figure1.ipynb
│   ├── 05_digital_simulation.ipynb
│   └── 06_extension.ipynb
│
├── figures/
│   ├── 01_exact_lindblad_dynamics.png
│   ├── 02_msse_figure1_reproduction.png
│   ├── 03_digital_simulation_validation.png
│   ├── 04_timestep_convergence.png
│   ├── 05_timestep_noise_tradeoff.png
│   └── 06_optimal_timestep_vs_noise.png
│
└── data/
    └── notebook04_results.npz
```

---

## Notebook overview

### 01 — Spin operators

Introduces the basic two-level spin system used throughout the project.

Topics include:

- Pauli matrices;
- tensor-product operators;
- spin expectation values;
- magnetisation observables.

This notebook establishes the operator conventions used in later simulations.

---

### 02 — Two-spin closed-system dynamics

Builds the two-spin transverse Ising Hamiltonian:

```math
H
=
-J\,\sigma_z^{(1)}\sigma_z^{(2)}
+
\Delta
\left(
\sigma_x^{(1)}
+
\sigma_x^{(2)}
\right)
```

and studies unitary time evolution under the Schrödinger equation.

This provides the closed-system reference before environmental effects are introduced.

---

### 03 — Lindblad dynamics and QuTiP

Extends the system to an open quantum system using the Lindblad master equation:

```math
\frac{d\rho}{dt}
=
-i[H,\rho]
+
\sum_{\ell}
\left(
L_{\ell}\rho L_{\ell}^{\dagger}
-
\frac{1}{2}
\left\{
L_{\ell}^{\dagger}L_{\ell},
\rho
\right\}
\right)
```

The notebook covers:

- density matrices;
- mixed versus pure states;
- amplitude damping;
- two-spin dissipation;
- magnetisation and purity;
- validation against QuTiP.

The QuTiP solution is used as the exact numerical reference in later notebooks.

---

### 04 — MSSE and Figure 1 reproduction

This notebook reproduces the paper's optimised stochastic simulation for the two-spin dissipative transverse Ising model.

The Lindblad evolution is unravelled into stochastic pure-state trajectories and approximated using the modified stochastic Schrödinger equation.

The Hamiltonian evolution is split as:

```math
e^{-i(1-x)H\Delta t}
\,\mathcal{D}\,
e^{-ixH\Delta t}
```

where the parameter $x$ controls the ordering between coherent and dissipative evolution.

For the initial state, the optimisation gives:

```math
x(0)\approx 0.49057
```

consistent with the value $x=0.4906$ reported in the paper.

The numerical comparison obtained in the reproduction was:

```text
RMSE, x = 0              : 0.090643
RMSE, x = 1              : 0.099840
RMSE, x = 0.4906         : 0.025819
RMSE, optimised x(t)     : 0.023203
```

This confirms that the optimised ordering substantially improves the finite-timestep approximation.

<p align="center">
  <img src="https://raw.githubusercontent.com/Temin119/open-quantum-systems-digital-simulation/main/figures/02_msse_figure1_reproduction.png" alt="MSSE reproduction" width="800">
</p>

---

### 05 — Digital quantum simulation

The stochastic algorithm is then translated into a digital quantum circuit.

The implementation uses:

- symmetric Trotterisation for the Hamiltonian;
- an ancilla qubit to represent dissipative jumps;
- mid-circuit measurement;
- ancilla reset and reuse;
- repeated circuit execution to reconstruct ensemble observables.

The circuit replaces the classical random jump process with quantum measurement outcomes.

The final comparison brings together:

```math
\text{Exact Lindblad}
\;\longleftrightarrow\;
\text{Classical MSSE}
\;\longleftrightarrow\;
\text{Digital circuit}
```

<p align="center">
  <img src="https://raw.githubusercontent.com/Temin119/open-quantum-systems-digital-simulation/main/figures/03_digital_simulation_validation.png" alt="Digital simulation validation" width="800">
</p>

---

## Original extension — timestep versus hardware noise

Notebook 06 investigates a question beyond the direct reproduction:

> **How should the timestep be chosen when the digital quantum hardware is noisy?**

A smaller timestep improves the discretisation of the continuous-time dynamics, but it also increases the number of Trotter steps and therefore the circuit depth.

For fixed final time:

```math
N_{\text{steps}}
\propto
\frac{1}{\Delta t}
```

This creates a competition:

```math
\Delta t \downarrow
\quad\Rightarrow\quad
\text{lower discretisation error}
```

but also:

```math
\Delta t \downarrow
\quad\Rightarrow\quad
\text{deeper circuit}
\quad\Rightarrow\quad
\text{more accumulated gate noise}
```

The extension first measures timestep convergence in the noiseless simulator.

<p align="center">
  <img src="https://raw.githubusercontent.com/Temin119/open-quantum-systems-digital-simulation/main/figures/04_timestep_convergence.png" alt="Timestep convergence" width="800">
</p>

Depolarising noise is then added using the simplified model discussed in the paper:

- two-qubit gate error: $p$;
- one-qubit gate error: $p/10$.

The tested values were:

```math
p
=
0,\;
10^{-4},\;
10^{-3},\;
10^{-2},\;
3\times10^{-2}
```

The resulting error landscape shows that the preferred timestep changes as the hardware becomes noisier.

<p align="center">
  <img src="https://raw.githubusercontent.com/Temin119/open-quantum-systems-digital-simulation/main/figures/05_timestep_noise_tradeoff.png" alt="Timestep-noise trade-off" width="800">
</p>

Representative results from the scan were:

| Two-qubit error $p$ | Best tested timestep $\Delta t$ | Circuit depth | CX gates |
|---:|---:|---:|---:|
| 0 | 0.2 | 1726 | 400 |
| $10^{-4}$ | 0.1 | 3446 | 800 |
| $10^{-3}$ | 0.4 | 866 | 200 |
| $10^{-2}$ | 0.4 | 866 | 200 |
| $3\times10^{-2}$ | 0.8 | 436 | 100 |

The low-noise ordering is sensitive to finite-shot sampling, so the main conclusion is not that one particular noiseless timestep is uniquely optimal.

The robust result is the broader shift toward **coarser, shallower circuits as gate noise increases**.

For example:

```text
p = 0.01

RMSE at dt = 0.1 : 0.114895
RMSE at dt = 0.8 : 0.075682
Best tested dt    : 0.4
Best RMSE         : 0.072239
```

and at:

```text
p = 0.03

RMSE at dt = 0.1 : 0.146206
RMSE at dt = 0.8 : 0.090299
Best tested dt    : 0.8
```

This demonstrates that the mathematically finest discretisation is not necessarily the most accurate implementation once hardware noise is included.

---

## Main conclusions

The project reproduces the main algorithmic chain used for the two-spin dissipative Ising example:

```math
\text{Schrödinger dynamics}
\rightarrow
\text{Lindblad equation}
\rightarrow
\text{quantum trajectories}
\rightarrow
\text{MSSE}
\rightarrow
\text{optimised ordering}
\rightarrow
\text{ancilla-assisted digital circuit}
```

The reproduction shows that optimising the MSSE ordering parameter strongly reduces finite-timestep error.

The extension then shows that timestep selection becomes hardware dependent: smaller timesteps reduce discretisation error, but the associated increase in circuit depth can make them worse under sufficiently strong gate noise.

---

## Limitations

This project focuses on the two-spin dissipative transverse Ising example rather than reproducing every result in the paper.

In particular:

- the larger quantum contact process simulations are not reproduced;
- Aer noise models are simplified approximations to hardware noise;
- noisy simulation results are not equivalent to running the circuits on a real quantum processor;
- finite-shot sampling introduces statistical uncertainty;
- the timestep scan explores a finite parameter grid rather than proving a global optimum.

---

## Software

The project uses:

- Python
- NumPy
- SciPy
- Matplotlib
- QuTiP
- Qiskit
- Qiskit Aer

---

## Running the notebooks

Create and activate a Python environment, install the dependencies, and run the notebooks in numerical order.

```bash
pip install -r requirements.txt
```

Then open the repository in Jupyter or VS Code and run:

```text
01_spin_operators.ipynb
02_two_spin_dynamics.ipynb
03_lindblad_qutip.ipynb
04_reproduce_figure1.ipynb
05_digital_simulation.ipynb
06_extension.ipynb
```

Some later notebooks use outputs or concepts established in earlier notebooks.

---

## Reference

M. Jo and M. S. Kim,  
**"Simulating open quantum many-body systems using optimised circuits in digital quantum simulation"**  
2022.

---

## Author note

This repository was developed as a learning and reproduction project in open quantum systems and digital quantum simulation.

The emphasis is on understanding the full path from the Lindblad master equation to an executable noisy quantum-circuit model, rather than treating the circuit implementation as a black box.
