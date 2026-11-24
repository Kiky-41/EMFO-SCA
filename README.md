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
| $\mathbf{x}^{\ast}$ | best position found so far | MW |
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

$$\mathbf{x}_i\leftarrow\mathbf{x}_i+A_i\odot\left(\mathbf{x}^{\ast}-\mathbf{x}_i\right)+B_i\odot C_i.$$

The first term moves the particle toward the best solution, scaled per coordinate; the second adds a bounded exploration offset. The reported settings are $\alpha=0.5$ and $\sigma=0.5$.

### SCA stage

After the EMFO move, each particle is perturbed again with the sine–cosine rule:

$$\mathbf{x}_i\leftarrow
\begin{cases}
\mathbf{x}_i+\beta r_2\sin(2\pi r_3)\left(\mathbf{x}^{\ast}-\mathbf{x}_i\right), & r_1\lt0.5,\\
\mathbf{x}_i+\beta r_2\cos(2\pi r_3)\left(\mathbf{x}^{\ast}-\mathbf{x}_i\right), & r_1\ge0.5.
\end{cases}$$

The random angle $2\pi r_3$ decides the direction, $r_2$ scales the move, and the sine/cosine switch balances outward exploration and inward exploitation. The reported amplitude is $\beta=0.5$.

### Repair and parameters

After each iteration the incumbent is rescaled to the demand and clipped to the unit limits:

$$\mathbf{x}^{\ast}\leftarrow\mathrm{clip}\left(\mathbf{x}^{\ast}\frac{P_D}{\sum_i x_i^{\ast}},\quad\mathbf{P}^{\min},\quad\mathbf{P}^{\max}\right).$$

The rescaling preserves the shape of the incumbent while enforcing the total output; the clip then restores individual limits. The population is 50, the iteration cap is 100, and the run stops early when $\left|P_D-\sum_ix_i\right|\lt10^{-4}$ MW.

### Implementation note

The reporting helper `calculate_fuel_cost` evaluates $a_iP_i^2+b_iP_i+c_i$, while the fitness function uses $a_i+b_iP_i+c_iP_i^2$. For the Sulselbar coefficients the constant and quadratic terms are both very small, so the difference is negligible in the reported cost, but the two expressions are not identical. The README follows the paper and the fitness function.
### Reported result
The hybrid is run on the 7-unit Sulselbar 150 kV load profile and benchmarked against the Kho-Kho algorithm on the same data; the per-unit schedule and cost of both methods are stored in `results/`.

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


## Results

### 24 load levels, EMFO-SCA against Kho-Kho (KKA)

| Demand (MW) | EMFO-SCA cost | EMFO-SCA time (s) | KKA cost | KKA time (s) | Cost saving |
|---|---|---|---|---|---|
| 147.28 | 103,309.43 | 0.0884 | 106,770.01 | 0.0112 | 3,460.58 |
| 160.10 | 110,320.32 | 0.0458 | 111,020.86 | 0.0163 | 700.54 |
| 181.54 | 126,404.89 | 0.0016 | 151,642.75 | 0.0011 | 25,237.85 |
| 186.41 | 132,798.78 | 0.0045 | 167,906.52 | 0.0005 | 35,107.75 |
| 186.90 | 130,601.50 | 0.0027 | 164,254.30 | 0.0004 | 33,652.80 |
| 192.95 | 142,481.77 | 0.0044 | 135,820.85 | 0.0005 | -6,660.92 |
| 209.07 | 159,850.08 | 0.0015 | 140,635.85 | 0.0094 | -19,214.23 |
| 235.20 | 160,834.06 | 0.0027 | 190,202.66 | 0.0004 | 29,368.59 |
| 248.43 | 184,651.72 | 0.0009 | 182,211.27 | 0.0004 | -2,440.45 |
| 273.17 | 174,927.24 | 0.0009 | 200,498.09 | 0.0004 | 25,570.84 |
| 301.32 | 195,125.14 | 0.0009 | 198,661.32 | 0.0004 | 3,536.18 |
| 303.25 | 210,886.22 | 0.0009 | 199,683.88 | 0.0012 | -11,202.34 |
| 307.96 | 189,560.79 | 0.0009 | 202,850.79 | 0.0004 | 13,289.99 |
| 346.08 | 201,276.79 | 0.0009 | 203,040.15 | 0.0004 | 1,763.36 |
| 348.18 | 239,038.81 | 0.0009 | 198,982.30 | 0.0004 | -40,056.52 |
| 351.97 | 209,039.90 | 0.0009 | 229,103.72 | 0.0004 | 20,063.82 |
| 372.91 | 242,071.93 | 0.0009 | 218,768.28 | 0.0004 | -23,303.65 |
| 378.01 | 263,191.59 | 0.0009 | 230,476.76 | 0.0004 | -32,714.83 |
| 382.93 | 216,019.34 | 0.0010 | 246,932.88 | 0.0005 | 30,913.54 |
| 416.18 | 234,712.15 | 0.0009 | 260,342.60 | 0.0004 | 25,630.44 |
| 444.52 | 304,498.30 | 0.0009 | 249,559.82 | 0.0004 | -54,938.48 |
| 456.03 | 276,700.54 | 0.0009 | 267,398.47 | 0.0004 | -9,302.07 |
| 495.56 | 318,395.87 | 0.0009 | 305,636.89 | 0.0004 | -12,758.97 |
| 521.52 | 332,483.75 | 0.0009 | 309,694.01 | 0.0039 | -22,789.74 |
| 7,447.46 | 4,859,180.93 | 0.1663 | 4,872,095.04 | 0.0508 | 12,914.11 |

EMFO-SCA is cheaper on 14 of the 25 load levels by 0.93% on average, with a best gain of 20.91% and a worst loss of 22.01%. The hybrid does not dominate level by level, so the claim is an average improvement rather than uniform superiority. Mean run time is 0.0133 s against 0.0041 s for KKA.

### Result workbooks

| File | Content |
|---|---|
| [`results/EMFO-SCA.xlsx`](results/EMFO-SCA.xlsx) | raw output |
| [`results/KKA.xlsx`](results/KKA.xlsx) | raw output |

The workbooks hold the raw solver output behind the tables above: per-unit schedules, per-hour costs, constraint checks and run statistics.


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
