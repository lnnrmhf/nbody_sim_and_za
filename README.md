# 3D N-Body Simulations and the Zel'dovich Approximation

**Final project — Course: Cosmic Structures: Theory, Numerics, and Statistics**  
Institute for Astronomy (IfA), University of Vienna — August 2026

Prof. Oliver Hahn & Dr. Thomas Flöss

---

## Overview

In this work, a simple N-body simulation code is implemented using Zel'dovich-approximation (ZA) initial conditions and a symplectic leapfrog integrator with gravitational acceleration computed via the non-uniform fast Fourier transform (NUFFT). The goal is to analyze the difference between the full N-body evolution and the pure Zel'dovich-approximation evolution of a Gaussian random field over time and to identify the scales and epochs at which the Zel'dovich approximation breaks down and the full non-linear dynamics become necessary to correctly capture structure growth.

<p align="center">
  <img src="figures/density_field_evolution.png" width="600" title="Evolution of an initial Gaussian field">
</p>

<p align="center">
  <em>Evolution of an initial Gaussian field set up with L = 1 and N = 64 for time a ∈ [0.1, 0.3, 0.5, 1, 3, 5]. The left column shows the ZA+Leapfrog evolution, the right column shows the pure ZA evolution. Points are coloured by 1+δ.</em>
</p>

The two solutions agree closely at early times but diverge markedly after shell-crossing. The leapfrog integrator produces well-defined halos clustered along filaments, while particles in the ZA case remain significantly more diffuse — the Zel'dovich approximation applies no gravitational force after the initial displacement is set and particles simply continue to stream along their initial linear trajectories. This is confirmed quantitatively by the evolution of the standard deviation of the density field, which grows much more strongly for the leapfrog integrator after shell-crossing, and by the power spectrum, which shows that ZA fails to reproduce power at small scales as soon as they become non-linear, with the discrepancy progressively extending to larger scales as time progresses.

The full methods, results and discussion are written up in the final report: [`Final_CosmicStructures_Einramhof.pdf`](Final_CosmicStructures_Einramhof.pdf).

---

## Repository structure

```
.
├── final_Einramhof.ipynb                    # Main notebook: simulation, validation, analysis
└── Final_CosmicStructures_Einramhof.pdf     # Full written report
```

---

## Methods

### Initial conditions

A 3D Gaussian random field with a power-law power spectrum P(k) = Ak⁻ᵅ is generated using `generate_gaussian_field()`. In addition to the density field, the first and second derivatives of the initial gravitational potential ϕ⁽ⁱⁿⁱ⁾ are computed in Fourier space and returned in real space. These are needed for the Zel'dovich displacement and the shell-crossing calculation.

### Simulation

The simulation assumes an Einstein-de Sitter cosmology (Ω_m = 1, Ω_Λ = Ω_r = 0), which allows integration in superconformal time t̂ = 2 − 2/√a. In this variable, the growth factor equals the scale factor, D₊ = a.

Particles are first placed on a regular Lagrangian grid and evolved using the Zel'dovich approximation while the system is in the linear regime. The time of first shell-crossing a* is estimated by tracking min_**q** det(∂**x**/∂**q**) over the grid for increasing a and finding the first value at which it vanishes. The simulation switches from ZA to the full leapfrog integrator at a_switch = 0.5a*, chosen to ensure that the particle distribution is still smooth and well-described by linear theory when the leapfrog integrator takes over.

The leapfrog integrator follows a symmetric drift-kick-drift (DKD) scheme in superconformal time. The gravitational acceleration is obtained via `calc_acc_nufft()`, which solves the Poisson equation in Fourier space using `finufft.nufft3d`, where the spatial derivative becomes algebraic: Â_i(**k**) = −ik_i/|**k**|² δ̂(**k**).

### Validation

Before running full 3D simulations, several checks were performed:

- **Power spectrum** — the measured P(k) of the initial Gaussian field is compared to the theoretical k⁻¹·⁵ slope, confirming the field correctly encodes the intended power-law spectrum.
- **1D plane wave evolution** — the ZA and leapfrog solutions are compared in x-v phase space. Prior to shell-crossing the two overlap perfectly; after shell-crossing, the ZA produces unphysical folded sheets while the leapfrog integrator produces the characteristic converging spiral, a sign of genuine gravitational collapse.
- **PM force resolution check** — the NUFFT-based force is compared to an exact 1D Newtonian force computed via a sorting algorithm. The NUFFT solution converges towards the exact result as grid resolution N increases, quantifying the effective smoothing scale h = 1/N below which structure cannot be gravitationally resolved.
- **Shell-crossing time** — the crossing time is validated for both the single-mode plane wave (a* ≈ 1, matching the analytic prediction) and a full Gaussian field (a* ≈ 0.25, as expected from its broad range of high-curvature small-scale modes).

---

## Requirements

```bash
pip install numpy finufft matplotlib plotly scipy
```

Open `final_Einramhof.ipynb` and run the cells in order. The simulation parameters (N, L, α, a_end) are set at the top of each section.

---

## References

- Dehnen, W. & Read, J. I. 2011, European Physical Journal Plus, 126 — *N-body simulations of gravitational dynamics*
- Michaux, M., Hahn, O., Rampf, C., & Angulo, R. E. 2020, MNRAS, 500, 663 — *Accurate initial conditions for N-body simulations*
- Rampf, C., List, F., & Hahn, O. 2025, JCAP, 2025, 020 — *BULLFROG integrator*
- Zel'dovich, Ya. B. 1970, A&A, 5, 84 — *The Zel'dovich approximation*

Full bibliography in the report.

---

*This project was completed as part of the Cosmic Structures: Theory, Numerics, and Statistics course, University of Vienna, 2026.*
