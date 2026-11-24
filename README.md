# EMFO-SCA

**Reducing Operational Costs in Sulselbar’s 150 kV System with Electromagnetic Field and Sine-Cosine Optimization**

Ikhsan, Rifki Rahman Nur · Marsuki, Aminah Indahsari · Wijaya, I Gede Putu Oka Indra  
*JAREE, 9(1), 2025*  
[![DOI](https://img.shields.io/badge/DOI-10.12962%2Fjaree.v9i1.458-blue)](https://doi.org/10.12962/jaree.v9i1.458)

Hybrid Electromagnetic Field Optimization + Sine-Cosine Algorithm for dynamic economic dispatch on the Sulselbar 150 kV system (benchmark: Kho-Kho Algorithm).

## Abstract

Dynamic economic dispatch (DED) is a crucial task in modern power systems, requiring efficient optimization to minimize generation costs while satisfying operational constraints. This study introduces a Hybrid Electromagnetic Field Optimization with Sine-Cosine Algorithm (EMFO-SCA) tailored to address the unique challenges of the Sulselbar 150 kV power system. Specifically, the algorithm is designed to handle non-linear cost functions, complex constraints, and dynamic load variations across a 24-hour scheduling period. EMFO-SCA achieves a balanced integration of global exploration (via Electromagnetic Field Optimization) and local exploitation (via the Sine-Cosine Algorithm), resulting in robust optimization performance. Applied to a system with seven active generator buses, EMFO-SCA demonstrates an average operational cost reduction of 0.27% compared to the Kho-Kho Algorithm (KKA). This improvement translates to measurable cost savings while maintaining strict adherence to generation limits, ramp rate constraints, and power balance at all intervals. For instance, during peak demand at 521.52 MW (hour 12), the method effectively minimizes costs without compromising operational reliability. The dual-phase design of EMFO-SCA enables faster convergence and higher accuracy than conventional methods, making it a scalable solution for real-world DED challenges. By optimizing power generation schedules dynamically and reliably, this study establishes EMFO-SCA as a significant advancement in energy system optimization, with clear potential for practical deployment in similar power systems.

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
