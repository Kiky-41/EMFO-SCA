# EMFO-SCA

**Reducing Operational Costs in Sulselbar’s 150 kV System with Electromagnetic Field and Sine-Cosine Optimization**

Ikhsan, Rifki Rahman Nur · Marsuki, Aminah Indahsari · Wijaya, I Gede Putu Oka Indra  
*JAREE, 9(1), 2025*  
[![DOI](https://img.shields.io/badge/DOI-10.12962%2Fjaree.v9i1.458-blue)](https://doi.org/10.12962/jaree.v9i1.458)

Hybrid Electromagnetic Field Optimization + Sine-Cosine Algorithm for dynamic economic dispatch on the Sulselbar 150 kV system (benchmark: Kho-Kho Algorithm).

## Abstract

Dynamic economic dispatch (DED) is a crucial task in modern power systems, requiring efficient optimization to minimize generation costs while satisfying operational constraints. This study introduces a Hybrid Electromagnetic Field Optimization with Sine-Cosine Algorithm (EMFO-SCA) tailored to address the unique challenges of the Sulselbar 150 kV power system. Specifically, the algorithm is designed to handle non-linear cost functions, complex constraints, and dynamic load variations across a 24-hour scheduling period. EMFO-SCA achieves a balanced integration of global exploration (via Electromagnetic Field Optimization) and local exploitation (via the Sine-Cosine Algorithm), resulting in robust optimization performance. Applied to a system with seven active generator buses, EMFO-SCA demonstrates an average operational cost reduction of 0.27% compared to the Kho-Kho Algorithm (KKA). This improvement translates to measurable cost savings while maintaining strict adherence to generation limits, ramp rate constraints, and power balance at all intervals. For instance, during peak demand at 521.52 MW (hour 12), the method effectively minimizes costs without compromising operational reliability. The dual-phase design of EMFO-SCA enables faster convergence and higher accuracy than conventional methods, making it a scalable solution for real-world DED challenges. By optimizing power generation schedules dynamically and reliably, this study establishes EMFO-SCA as a significant advancement in energy system optimization, with clear potential for practical deployment in similar power systems.


## Proposed method

EMFO-SCA is a two-stage hybrid. Electromagnetic field optimisation (EMFO) supplies an attraction term that pulls each particle toward the incumbent best, and the sine–cosine algorithm (SCA) supplies an oscillating perturbation that alternates between sine and cosine. Both stages share one global best, and a scaling repair restores the power balance after every iteration. The method is tested on the 7-unit Sulselbar 150 kV system and compared with the Kho-Kho algorithm.

### Fitness

The equality constraint is handled with a penalty, so the search minimises

$$J(\mathbf{P})=\sum_{i=1}^{N}\left(a_i+b_iP_i+c_iP_i^2\right)+\rho\left|P_D-\sum_{i=1}^{N}P_i\right|,\qquad \rho=10^{4}.$$

The first term is the fuel cost and the second term grows with any mismatch between generation and demand, so infeasible particles are pushed back toward the balance manifold.

### Notation

| Symbol | Meaning | Unit |
|---|---|---|
| $N$ | number of units | – |
| $\mathbf{x}_i$ | position (dispatch) of particle $i$ | MW |
| $\mathbf{x}^{*}$ | best position found so far | MW |
| $J$ | penalised fitness | USD/h |
| $\rho$ | penalty coefficient | USD/MWh |
| $\alpha$ | attraction coefficient | – |
| $\beta$ | sine–cosine amplitude factor | – |
| $\sigma$ | decay factor of the EMFO term | – |
| $t,T$ | current iteration and iteration cap | – |
| $r_1,r_2,r_3$ | uniform random numbers in $[0,1]$ | – |
| $A_i,B_i,C_i$ | EMFO coefficient vectors | – |
| $P_D$ | demand | MW |

### EMFO stage

For particle $i$ at iteration $t$ of $T$, the EMFO coefficients are

$$A_i=\alpha e^{-\sigma t/T}\left(2r_{1,i}-1\right),\qquad B_i=\alpha e^{-\sigma t/T}\left(2r_{2,i}-1\right),\qquad C_i=e^{-\sigma t/T}\cos\left(2\pi r_{1,i}\right).$$

The exponential factor weakens the attraction as the run proceeds, and the cosine term gives the update an oscillating component. The position update is

$$\mathbf{x}_i\leftarrow\mathbf{x}_i+A_i\odot\left(\mathbf{x}^{*}-\mathbf{x}_i\right)+B_i\odot C_i.$$

The first term moves the particle toward the best solution, scaled per coordinate; the second adds a bounded exploration offset. The reported settings are $\alpha=0.5$ and $\sigma=0.5$.

### SCA stage

After the EMFO move, each particle is perturbed again with the sine–cosine rule:

$$\mathbf{x}_i\leftarrow
\begin{cases}
\mathbf{x}_i+\beta r_2\sin(2\pi r_3)\left(\mathbf{x}^{*}-\mathbf{x}_i\right), & r_1<0.5,\\
\mathbf{x}_i+\beta r_2\cos(2\pi r_3)\left(\mathbf{x}^{*}-\mathbf{x}_i\right), & r_1\ge0.5.
\end{cases}$$

The random angle $2\pi r_3$ decides the direction, $r_2$ scales the move, and the sine/cosine switch balances outward exploration and inward exploitation. The reported amplitude is $\beta=0.5$.

### Repair and parameters

After each iteration the incumbent is rescaled to the demand and clipped to the unit limits:

$$\mathbf{x}^{*}\leftarrow\operatorname{clip}\left(\mathbf{x}^{*}\frac{P_D}{\sum_i x_i^{*}},\quad\mathbf{P}^{\min},\quad\mathbf{P}^{\max}\right).$$

The rescaling preserves the shape of the incumbent while enforcing the total output; the clip then restores individual limits. The population is 50, the iteration cap is 100, and the run stops early when $\left|P_D-\sum_ix_i\right|<10^{-4}$ MW.

### Implementation note

The reporting helper `calculate_fuel_cost` evaluates $a_iP_i^2+b_iP_i+c_i$, while the fitness function uses $a_i+b_iP_i+c_iP_i^2$. For the Sulselbar coefficients the constant and quadratic terms are both very small, so the difference is negligible in the reported cost, but the two expressions are not identical. The README follows the paper and the fitness function.

## Contents

```
.gitignore
EMFO-SCA.ipynb
benchmarks/
  KKA.ipynb
results/
  EMFO-SCA.xlsx
  KKA.xlsx
```

## Data

Input data is **not included**. Place files in `data/` (notebooks) or next to the `.m` script (MATLAB):

| File | Columns |
|---|---|
| `input7u.xlsx` | Sheet1: `Unit`, `a`, `b`, `c`, `pmin`, `pmax`, `rampup`, `rampdown` |
| `power_demand7u.xlsx` | Sheet1: `load`, `hour` |

## Requirements

Python. Python notebooks: `numpy`, `pandas`, `scipy`, `matplotlib` (as imported). MATLAB scripts: R2021a+.

## Citation

```bibtex
@article{Ikhsan_2025, title={Reducing Operational Costs in Sulselbar’s 150 kV System with Electromagnetic Field and Sine-Cosine Optimization}, volume={9}, ISSN={2580-0361}, url={http://dx.doi.org/10.12962/jaree.v9i1.458}, DOI={10.12962/jaree.v9i1.458}, number={1}, journal={JAREE (Journal on Advanced Research in Electrical Engineering)}, publisher={Lembaga Penelitian dan Pengabdian kepada Masyarakat ITS}, author={Ikhsan, Rifki Rahman Nur and Marsuki, Aminah Indahsari and Wijaya, I Gede Putu Oka Indra}, year={2025}, month=Feb }
```

## License

[![CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](LICENSE)

Licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). **Attribution is required**: any use of this code, results, or figures, including in publications, must cite:

> Rifki Rahman Nur Ikhsan; Aminah Indahsari Marsuki; I Gede Putu Oka Indra Wijaya (2025). Reducing Operational Costs in Sulselbar’s 150 kV System with Electromagnetic Field and Sine-Cosine Optimization. *JAREE (Journal on Advanced Research in Electrical Engineering)*. https://doi.org/10.12962/jaree.v9i1.458
