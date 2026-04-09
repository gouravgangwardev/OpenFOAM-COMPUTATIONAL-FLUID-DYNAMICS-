# CFD Complete Project — Heated Square Cylinder at Re = 100
## Transient Forced and Mixed Convection | buoyantPimpleFoam | OpenFOAM v2306

---

## Project Summary

| Parameter        | Value                              |
|------------------|------------------------------------|
| Solver           | buoyantPimpleFoam (transient)      |
| Reynolds number  | Re = 100                           |
| Heat flux cases  | q = 100, 500, 1000 W/m²            |
| Strouhal number  | St = 0.154                         |
| Cd (transient)   | 1.534 (steady: 1.513)              |
| Cl amplitude     | ±0.462                             |
| Nu enhancement   | +3.7% to +5.8% (transient vs steady)|
| Time scheme      | Backward (BDF2), CFL ≤ 0.8         |
| Mesh             | ~42,000 structured hex cells       |

---

## Folder Structure

```
CFD_Complete_Project/
│
├── README.md                          ← This file
│
├── OpenFOAM_Case/                     ← Full runnable OpenFOAM case
│   ├── Allrun                         ← Run script (serial or parallel)
│   ├── 0/
│   │   ├── U                          ← Velocity initial condition
│   │   ├── T                          ← Temperature initial condition
│   │   └── p_rgh                      ← Modified pressure IC
│   ├── system/
│   │   ├── controlDict                ← Transient settings, adaptive dt, CFL
│   │   ├── fvSchemes                  ← BDF2 time, linearUpwind spatial
│   │   └── fvSolution                 ← PIMPLE loop, GAMG solver
│   └── constant/
│       ├── thermophysicalProperties   ← Air at 30°C
│       ├── turbulenceProperties       ← Laminar model
│       ├── g                          ← Gravity vector
│       └── polyMesh/
│           └── blockMeshDict          ← Structured hex mesh definition
│
├── Python_Scripts/
│   └── post_process_shedding.py       ← FFT analysis: St, Cl/Cd, plots, CSV
│
├── Charts/                            ← Publication-quality figures (180 dpi PNG)
│   ├── fig1_Cl.png                    ← Lift coefficient time history
│   ├── fig2_Cd.png                    ← Drag coefficient time history
│   ├── fig3_FFT.png                   ← FFT power spectrum → St = 0.154
│   ├── fig4_phase.png                 ← Phase portrait (figure-8 Lissajous)
│   ├── fig5_Nu.png                    ← Nusselt number: steady vs transient
│   └── fig6_Ri.png                    ← Richardson number regime classification
│
├── Reports/
│   ├── CFD_Report_WithCharts.docx     ← Full report with embedded charts
│   └── CFD_Final_Report_SubmissionReady.docx ← Full report (text + tables)
│
└── JavaScript_Builders/               ← Source code to regenerate reports
    ├── build_report.js                ← Initial report builder
    ├── build_final_report.js          ← Full 14-section report
    └── build_report_with_charts.js    ← Report with embedded PNG charts
```

---

## How to Run the OpenFOAM Case

### Prerequisites
- OpenFOAM v2306 (or compatible version)
- Source the OpenFOAM environment before running

```bash
source /opt/openfoam2306/etc/bashrc   # adjust path as needed
```

### Step 1 — Copy steady-state solution as initial condition (recommended)
```bash
# Place your converged buoyantSimpleFoam time directory here
# e.g. copy time directory "5000" into the case folder
# This reduces spin-up time significantly
```

### Step 2 — Run the case
```bash
cd OpenFOAM_Case/
chmod +x Allrun

# Serial run:
./Allrun

# Parallel run (4 cores):
./Allrun --parallel 4
```

### Step 3 — Post-process (FFT + Strouhal number)
```bash
python3 ../Python_Scripts/post_process_shedding.py \
    --case_dir . \
    --spinup_cycles 3
```

**Outputs saved to:** `postProcessing/FFT_analysis/`
- `shedding_analysis.png` — Cl(t), Cd(t), PSD, phase portrait
- `shedding_summary.csv` — St, f_shed, Cd_mean, Cl_rms, etc.

---

## Key Numerical Settings

### controlDict
| Setting              | Value         | Reason                              |
|----------------------|---------------|-------------------------------------|
| application          | buoyantPimpleFoam | Transient buoyancy-coupled solver |
| startFrom            | latestTime    | Restart from steady solution        |
| endTime              | 40540 s       | 10 shedding cycles                  |
| adjustTimeStep       | yes           | Adaptive CFL control                |
| maxCo                | 0.8           | CFL ≤ 1 enforced                   |
| maxDeltaT            | 5.0 s         | Safety cap                          |

### fvSchemes
| Term       | Scheme            | Order  |
|------------|-------------------|--------|
| ddt         | backward (BDF2)   | 2nd    |
| div(phi,U) | linearUpwind      | 2nd    |
| div(phi,h) | linearUpwind      | 2nd    |
| laplacian  | Gauss linear corrected | 2nd |

### fvSolution — PIMPLE
| Parameter              | Value | Reason                          |
|------------------------|-------|---------------------------------|
| nOuterCorrectors       | 3     | Momentum-pressure coupling      |
| nCorrectors            | 2     | Inner PISO pressure correctors  |
| nNonOrthogonalCorrectors | 2   | Mesh non-orthogonality at corners |
| momentumPredictor      | yes   | Required with buoyancy          |

---

## How to Regenerate Reports

Requires Node.js and the `docx` npm package:

```bash
npm install -g docx
cd JavaScript_Builders/
node build_report_with_charts.js   # generates report with embedded charts
```

The script reads the PNG files from `../Charts/` relative to its location. Adjust paths if needed.

---

## Physics Summary

| Quantity              | Steady (buoyantSimpleFoam) | Transient (buoyantPimpleFoam) |
|-----------------------|---------------------------|-------------------------------|
| Lift Cl               | ≈ 0 (SIMPLE enforces symmetry) | ±0.462 at f_s            |
| Drag Cd               | 1.513 (fixed)             | 1.534 mean, ripples at 2f_s   |
| Wake structure        | Symmetric bubbles         | von Kármán vortex street      |
| Nu (q=100 W/m²)       | 5.38                      | 5.58 (+3.7%)                  |
| Strouhal number       | Not applicable            | 0.154                         |

**Key finding:** The steady solution suppresses vortex shedding and enforces a symmetric wake,
leading to underprediction of both drag and heat transfer.

**Closing statement:** Transient CFD is not optional but essential for accurately capturing
bluff-body flow physics, as steady formulations fundamentally suppress key transport mechanisms
governing drag and heat transfer.

---

## References

1. Sohankar et al. (1998) — IJNMF 26(1), 39–56
2. Sharma & Eswaran (2004) — Num. Heat Transfer A, 45(3), 247–269
3. Breuer et al. (2000) — Int. J. Heat Fluid Flow, 21, 186–196
4. Sahu et al. (2009) — Int. J. Heat Mass Transfer, 52(3–4), 839–850
5. Dhiman et al. (2005) — Num. Heat Transfer A, 47(4), 291–312
