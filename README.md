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

| Symbol | Meaning | Symbol | Meaning |
|---|---|---|---|
| $N$ | number of units | $\beta$ | sine–cosine amplitude factor |
| $\mathbf{x}_i$ | position (dispatch) of particle $i$ (MW) | $\sigma$ | decay factor of the EMFO term |
| $\mathbf{x}^{\ast}$ | best position found so far (MW) | $t,T$ | current iteration and iteration cap |
| $J$ | penalised fitness (USD/h) | $r_1,r_2,r_3$ | uniform random numbers in $[0,1]$ |
| $\rho$ | penalty coefficient (USD/MWh) | $A_i,B_i,C_i$ | EMFO coefficient vectors |
| $\alpha$ | attraction coefficient | $P_D$ | demand (MW) |

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

| Demand (MW) | 147.28 | 160.10 | 181.54 | 186.41 | 186.90 | 192.95 | 209.07 | 235.20 | 248.43 | 273.17 | 301.32 | 303.25 | 307.96 | 346.08 | 348.18 | 351.97 | 372.91 | 378.01 | 382.93 | 416.18 | 444.52 | 456.03 | 495.56 | 521.52 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| EMFO-SCA cost | 103,309.43 | 110,320.32 | 126,404.89 | 132,798.78 | 130,601.50 | 142,481.77 | 159,850.08 | 160,834.06 | 184,651.72 | 174,927.24 | 195,125.14 | 210,886.22 | 189,560.79 | 201,276.79 | 239,038.81 | 209,039.90 | 242,071.93 | 263,191.59 | 216,019.34 | 234,712.15 | 304,498.30 | 276,700.54 | 318,395.87 | 332,483.75 |
| EMFO-SCA time (s) | 0.0884 | 0.0458 | 0.0016 | 0.0045 | 0.0027 | 0.0044 | 0.0015 | 0.0027 | 0.0009 | 0.0009 | 0.0009 | 0.0009 | 0.0009 | 0.0009 | 0.0009 | 0.0009 | 0.0009 | 0.0009 | 0.0010 | 0.0009 | 0.0009 | 0.0009 | 0.0009 | 0.0009 |
| KKA cost | 106,770.01 | 111,020.86 | 151,642.75 | 167,906.52 | 164,254.30 | 135,820.85 | 140,635.85 | 190,202.66 | 182,211.27 | 200,498.09 | 198,661.32 | 199,683.88 | 202,850.79 | 203,040.15 | 198,982.30 | 229,103.72 | 218,768.28 | 230,476.76 | 246,932.88 | 260,342.60 | 249,559.82 | 267,398.47 | 305,636.89 | 309,694.01 |
| KKA time (s) | 0.0112 | 0.0163 | 0.0011 | 0.0005 | 0.0004 | 0.0005 | 0.0094 | 0.0004 | 0.0004 | 0.0004 | 0.0004 | 0.0012 | 0.0004 | 0.0004 | 0.0004 | 0.0004 | 0.0004 | 0.0004 | 0.0005 | 0.0004 | 0.0004 | 0.0004 | 0.0004 | 0.0039 |
| Cost saving | 3,460.58 | 700.54 | 25,237.85 | 35,107.75 | 33,652.80 | -6,660.92 | -19,214.23 | 29,368.59 | -2,440.45 | 25,570.84 | 3,536.18 | -11,202.34 | 13,289.99 | 1,763.36 | -40,056.52 | 20,063.82 | -23,303.65 | -32,714.83 | 30,913.54 | 25,630.44 | -54,938.48 | -9,302.07 | -12,758.97 | -22,789.74 |

EMFO-SCA is cheaper on 13 of the 24 load levels by 0.95% on average, with a best gain of 20.91% and a worst loss of 22.01%. The hybrid does not dominate level by level, so the claim is an average improvement rather than uniform superiority. Mean run time is 0.0133 s against 0.0041 s for KKA.

### Daily aggregate over the 24 levels

| Metric | EMFO-SCA | KKA | Difference |
|---|---|---|---|
| Total cost | 4,859,180.93 | 4,872,095.04 | 12,914.11 |
| Total time (s) | 0.1663 | 0.0508 | -0.1155 |

Summed over the day the hybrid costs 4,859,180.93 against 4,872,095.04 for KKA, 0.27% lower, and spends 0.1663 s against 0.0508 s. The daily margin is smaller than the average per-level margin because KKA wins several of the low-demand levels.

### Result workbooks

Raw solver output: [`results/EMFO-SCA.xlsx`](results/EMFO-SCA.xlsx), [`results/KKA.xlsx`](results/KKA.xlsx).

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
