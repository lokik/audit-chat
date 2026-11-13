# ASE2SPRKKR — Task Interface, Parameter and Documentation Audit

**Updated:** 13 November 2026  
**Reference functional audit:** [Consolidated SPR-KKR 9.7 / ASE2SPRKKR audit](../SPRKKR-ASE2SPRKKR-common-audit.md) (9 November 2026), and linked individual audits.  
**Implementation baseline:** SPR-KKR public 9.7 archive `PUB9.7_260420_18h13`; the reference functional audit checked ASE2SPRKKR `develop` at `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. Current schema/registry and file naming were additionally spot-checked at `develop` `8aa03363433b45b98e2594bfaecc7578a46be94c` (12 November 2026). **The full suite was not rerun at this newer revision.**  
**Manual comparison:** public SPR-KKR User's Guide (8.6-era reference used in the earlier task audit); classification of parameter names is carried over and should be considered **provisional** unless a per-task audit cites the relevant manual section.

## Purpose and interpretation

This document is an **ASE2SPRKKR interface audit**, complementing rather than duplicating the existing [source-code/functionality audit](../SPRKKR-ASE2SPRKKR-common-audit.md). It records each task's purpose, concise operational limitations, file contract, task-specific parameters and which of them are documented in the available manual. Technical fault analysis is linked, not copied.

- **Task list:** 56 entries including the `BSF` **schema-only** heading; the reference audit reports **55 registered task identities**. `BSFEK`/`BSFKK` share the BSF schema, while `DMI` has its own identity with JXC's schema. Unregistered native-only names (e.g. BIS, SIGMA-XAS) remain documented in the functional audit rather than masquerading as runnable ASE2SPRKKR tasks.
- **Input files:** generated SPR-KKR input is normally delivered via standard input and refers to `CONTROL.POTFIL`. Required external files, optional reusable cache files, and input-mode auxiliary files are distinguished. `TAUDIR` / `TAUPATH` may affect cache location.
- **Output files:** names follow current declared file contracts where known. `{DATASET}` means the original native `DATSET0`, while `{DATASET_ADSI}` means `DATASET` with optional nonempty `ADSI` suffix. Not every native output is represented by a manifest or structured parser. Fixed names such as `fort.*`, PLY, `sigma?????.dat` and `arpes.dat` can collide across runs.
- **Parameters:** the two lists contain **task-specific** input keys and task-specific overrides, not every generic `CONTROL/TAU/ENERGY/MODE/CPA/STRCONST/SITES` member. “Not covered by manual” means *not demonstrated in the available guide*, not rejected by native code. The older coverage split is preserved for traceability rather than claimed as a new exhaustive comparison.
- **Verification:** schema and regression tests, an actual native run, correct file discovery, a structured result reader and numerical/physical validation are separate levels. The 9 November reference suite reports **1249 passed, 1 skipped** for input-parameter/result-file tests; this is **not** a numerical certification. The targeted native **PHAGEN** test checked the export files (32 passing targeted tests); it did not compare results with external MsSpec. Unless stated otherwise, no independent native numerical reference result was established for a task in that audit.
- **Status:** short practical assessment **derived from the functional audit**, not a new claim that every task was run. Review the linked report for exact broken branches and conditions.

## Index

| Task | Status (short) | In manual |
|---|---|---|
| [`AES`](#aes) | Works only in the limited audited workflow; NQ=1, no f shells, with cache/TAU-capacity risks | Yes |
| [`AIPES`](#aipes) | Works in the audited fixed-geometry workflow; it is not numerical angular integration of ARPES | Yes |
| [`APS`](#aps) | Works with a compatible old-format DOS; positive-`GAMMA` broadening follows a problematic native branch | Yes |
| [`ARPES`](#arpes) | Works, but the current definition retains native, presence-sensitive `CRYS_VECS` supported; low-level/expert branches are not all certified | Yes |
| [`BAND`](#band) | Works in the audited workflow; an eigen-solver failure may continue natively as far as `LOG(0)` | Yes |
| [`BONDING`](#bonding) | Broken: normal `TAU_DRIVE` prepares `KTAUIJ=0`, while `CLUBONDING` reads scratch units requiring `KTAUIJ>=2` | No |
| [`BSDOSIJ`](#bsdosij) | Broken: native `J_ITJT` is written with energy indices into pair dimensions and later read in a different order | No |
| [`BSF`](#bsf) | Schema only; native BSF works; the E-k default `EMAX=-1 Ry` is a suspicious convenience default and need not mean EF | Yes |
| [`BSFEK`](#bsfek) | Works like `BSF`; the alias only preselects the expected E-k mode through `ENERGY.NE` | Yes, as `BSF`; `BSFEK` is an ASE2SPRKKR alias |
| [`BSFKK`](#bsfkk) | Works like `BSF`; the alias only preselects the expected single-energy k-k mode | Yes, as `BSF`; `BSFKK` is an ASE2SPRKKR alias |
| [`CHI`](#chi) | Works in the audited static workflow; native `NCHIITER` is forcibly reset to 1 before the loop, so the iterative magnetization part does not actually run | Yes |
| [`CHILANDAU`](#chilandau) | Works serially with limitations; MPI and `RDTAUFIL` reuse are broken, and native finish logic prints partial sums | No |
| [`CHIXAS`](#chixas) | Works only with physically consistent external `NEF/CHISPN/CHIVV`; no universally safe defaults exist | No |
| [`CLPES`](#clpes) | Works in the audited serial fixed-geometry workflow; CPA and some polarization modes stop natively, and `NE=1` requires a valid core-state index | No |
| [`CLXPS`](#clxps) | Works in the audited ASA/FREL workflow; advanced angular geometries and numerical output were not independently certified | Yes |
| [`COMPTON`](#compton) | Works with limitations; `EMD2D` double-scales nonzero `PNMAX`, and shifted even grids lose endpoints | Yes |
| [`DMI`](#dmi) | Works with significant caveats: native DMI components are not equivalent (especially Dz), and frame/multiplicity/radius must be checked | No as a separate task in the available manual; JXC is documented |
| [`DOS`](#dos) | Works | Yes |
| [`EKREL`](#ekrel) | Works; CPA is rejected natively, and BSF should be used for disorder | Yes |
| [`FMAG`](#fmag) | Works with a limitation: a zero magnetic moment can cause division by zero during normalization | Yes |
| [`FSCAT`](#fscat) | Broken: an erroneous spherical-harmonic `+L` index can overflow, and phase is calculated before zero amplitude is handled | No |
| [`GILBERT`](#gilbert) | Works for suitable magnetic systems; zero or compensated moments can cause division by zero | Yes |
| [`JALF`](#jalf) | Works with limitations; helper diagnostics contain bugs and small `NL` is risky, which the definition guards against | No |
| [`JORB`](#jorb) | Works serially; native MPI does not reach final JORB output and does not reduce the accumulators | No |
| [`JXC`](#jxc) | Works for a simple periodic collinear workflow; the general FREL tensor for symmetry-equivalent neighbors has a native rotation bug | Yes |
| [`MECHECK`](#mecheck) | Only partially works: `NMEFORM` is always overwritten to 2, another consistency test is disabled at compile time, and the summary loop is faulty | No |
| [`MEPLOT`](#meplot) | Works except for native curve overwriting in simultaneous photon+electron scans; the definition rejects that combination | No |
| [`NRAES`](#nraes) | Broken: `IAES` is only partially zeroed and convolution accumulates into uninitialized regions | Yes |
| [`NUCSSCPL`](#nucsscpl) | Broken: native code deallocates arrays before first allocation, clears matrix elements for each type, and forces the mesh to 1 | No |
| [`OPTABS`](#optabs) | Works with major caveats: `TEMPEL<=1` leaves `KBT` uninitialized and `NOMEGA<NEB` can overflow allocation; the definition rejects these states | No |
| [`PHAGEN`](#phagen) | Works as a restricted ASA/VWN converter; NT=1, no disorder, not a general phase-shift solver | No |
| [`PLOT2DPOT`](#plot2dpot) | Works in ASA for potential; FULLPOT enters an experimental PLY writer and exits before the normal plane-grid output | No |
| [`PLOT2DPR`](#plot2dpr) | Partially works: in ASA it actually writes potential only; FULLPOT exits in the experimental PLY writer | No |
| [`PLOT2DRHO`](#plot2drho) | Practically incomplete: ASA cannot plot density; FULLPOT exits in the test PLY writer before the normal plane-grid writer | No |
| [`PLOTPOT`](#plotpot) | Works for stored ASA and FULLPOT potential data | No |
| [`PLOTPRS`](#plotprs) | Works with limitations: magnetic spin/orbital curves can collide in the same dataset and panel limits ignore the orbital contribution | No |
| [`PLOTRHO`](#plotrho) | Broken: `RHORSQ` is allocated only in the potential branch, but the standalone rho writer needs it | No |
| [`PLOTSFN`](#plotsfn) | Works for a genuine FULLPOT potential containing shape-function data | No |
| [`POSANI`](#posani) | Works only with a compatible `POSANIPREP`; native data handling assumes `NQ=NT`, and general `NQ!=NT` is broken | Yes |
| [`POSANIPREP`](#posaniprep) | Works for a simple 3D ASA/FREL `NQ=NT` case; TAUQQ export is broken for general `NQ!=NT` | Yes |
| [`POSANITAU`](#posanitau) | Works in the audited 3D ASA/FREL workflow; integration bounds need physical validation and the enhancement factor is fixed natively | No |
| [`PSHIFT`](#pshift) | Works, but resonance splitting is unreliable in some cases | Yes |
| [`SCF`](#scf) | Works for `BROYDEN2`, `ANDERSON`, and `TCHEBY` within the allowed scope; `PULAY` is invalid and `TCHEBY` is unavailable for FULLPOT | Yes |
| [`SIGMA`](#sigma) | Works with limitations; the Fermi-sea contribution is not included automatically, and modes use fixed file conventions/ImE | Yes |
| [`SOCPAR`](#socpar) | Works for the main SOC output; exchange-related output has a wrong unit label and one sign-flip test is unreachable natively | Yes |
| [`SPLEED`](#spleed) | Works in a conservative fixed-angle workflow; momentum-scan and alternative polarization branches were not audited | Yes |
| [`STONER`](#stoner) | Works with limitations; uses a hard-coded VWN susceptibility kernel, replaces small DOS values with unit contributions, and has unguarded normalizations | No |
| [`T1`](#t1) | Broken for numerical use: native `RELT1` treats signed κ as positive counts of mj channels, dropping negative channels and corrupting basis indexing | No |
| [`TDOBS`](#tdobs) | Declared purpose is not demonstrated: no separate time-dependent observable implementation was found; it behaves as a generic DOS path | No |
| [`TORQUE`](#torque) | Works, but requires a rotated magnetic potential/geometry; a normal unrotated magnetic potential may stop natively | Yes |
| [`TUTORIAL`](#tutorial) | Mixed: YLM/JLX/JLX-ASYMPT and DOS-FEG/DOS-SS/DOS-CLU/DOS-BULK/GEN are usable; other branches have specific native bugs/limitations | No |
| [`VBPES`](#vbpes) | Works with limitations; VBUPS does not automatically select a UV photon energy and the calculation depends on compatible TAU/final-state setup | Yes |
| [`WFPLOT`](#wfplot) | Works in the ASA/FREL serial workflow; BAND uses only the first energy point and FULLPOT is unsupported | Yes |
| [`XAS`](#xas) | Works with known caveats; the static branch is usable, but TAU compatibility and the effective scan start must be checked | Yes |
| [`XES`](#xes) | Not fully verified: RXES enters the shared absorption-rate branch and shares output naming/interpretation with XAS; the emission meaning was not confirmed | Yes |
| [`XRS`](#xrs) | Broken for reliable numerical results: native `SIG(NEMAX+50,...)` is zeroed only up to `NEMAX` but later summed over the full range | Yes (XRS is at least listed as a supported spectroscopy task; the detailed 9.7 interface is broader) |

## Tasks

### AES

- **What it does:** Calculates fully relativistic Auger electron spectra from existing electronic TAU data and matrix elements.
- **Status:** Works only in the limited audited workflow; NQ=1, no f shells, with cache/TAU-capacity risks. The numerical solver was not run during the audit.
- **Input files:** Generated SPR-KKR input, `POTFIL`, required compatible TAU directory; optional fixed-name `ME`, `ME1`…`ME12` cache.
- **Output files:** `{DATASET_ADSI}<type>.<core-shell>.aes` (native type/shell-dependent name); diagnostics may have fixed names.
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** `TASK.IT`, `TASK.CL`, `TASK.NEME`, `TASK.PRINTME`
- **Parameters not covered by the manual:** `TASK.PRINTAME`

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_aes.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [AES](AES-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### AIPES

- **What it does:** Calculates native local/incoherent angle-integrated valence photoemission with a surface contribution.
- **Status:** Works in the audited fixed-geometry workflow; it is not numerical integration of ARPES over angles.
- **Input files:** Generated KKRSPEC input and `POTFIL`; conditional `input.inp` (`INPVER=0`) and `struc.inp` (`STRVER=0`).
- **Output files:** `{DATASET_ADSI}_SPEC.out` and polarization-dependent `{DATASET_ADSI}_data*.spc` files.
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** `TASK.IQ_AT_SURF`, `TASK.MILLER_HKL`, `TASK.STRVER`, `TASK.INPVER`, `ENERGY.EWORK_EV`, `ENERGY.IMV_INI_EV`, `ENERGY.IMV_FIN_EV`, `SPEC_PH.THETA`, `SPEC_PH.PHI`, `SPEC_PH.POL_P`, `SPEC_PH.EPHOT`, `SPEC_EL.THETA`, `SPEC_EL.PHI`, `SPEC_EL.NT`, `SPEC_EL.NP`, `SPEC_EL.KA`, `SPEC_EL.K1`, `SPEC_EL.NK1`, `SPEC_EL.K2`, `SPEC_EL.NK2`, `SPEC_EL.POL_E`, `SPEC_EL.SPOL`, `SPEC_STR.N_LAYDBL`, `SPEC_STR.NLAT_G_VEC`, `SPEC_STR.N_LAYER`, `SPEC_STR.SURF_BAR`, `SPEC_STR.TRANSP_BAR`
- **Parameters not covered by the manual: `TASK.CRYS_VECS`, `SPEC_PH.ALQ`, `SPEC_PH.DELQ`, `SPEC_PH.NPOL`, `SPEC_PH.ICIRC`, `SPEC_PH.IDREH`, `SPEC_PH.IFSP`, `SPEC_PH.THETA_FIX`, `SPEC_EL.K3`, `SPEC_EL.NK3`, `SPEC_EL.K4`, `SPEC_EL.NK4`, `SPEC_EL.PSPIN`, `SPEC_EL.BETA1`, `SPEC_EL.BETA2`, `SPEC_EL.ROTAXIS`, `SPEC_EL.TYP`, `SPEC_EL.ISTR`, `SPEC_EL.POL0`, `SPEC_EL.POL0L`, `SPEC_EL.Q1`, `SPEC_EL.Q2`, `SPEC_EL.Q3`, `SPEC_EL.Q4`

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_aipes.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** `SPEC` selected by the task definition (file parsing does not guarantee physical validation).
- **Detailed source/functionality audit:** [AIPES](AIPES-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### APS

- **What it does:** Calculates an appearance-potential spectrum by postprocessing a previously calculated DOS.
- **Status:** Works with a compatible old-format DOS; positive `GAMMA` broadening follows a problematic native branch.
- **Input files:** Generated input + `POTFIL` + existing `{DATASET_ADSI}.dos` (old spin-resolved DOS format and adequate energy capacity required).
- **Output files:** `{DATASET_ADSI}.<type>.<core-shell>.aps`; with `GAMMA>0`, also `{DATASET_ADSI}.dos.brd`; optional CALCLS/OUTGNU auxiliaries.
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** `TASK.IT`, `TASK.CL`, `TASK.GAMMA`, `TASK.OUTGNU`, `TASK.CALCLS`
- **Parameters not covered by the manual:** —

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_aps.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [APS](APS-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### ARPES

- **What it does:** Calculates angle-resolved photoemission using KKRSPEC.
- **Status:** The source-defined ARPES workflow is meaningful in the audited range. The current interface uses the correct presence-sensitive `TASK.CRYS_VECS`; advanced numerical branches are not certified.
- **Input files:** Generated KKRSPEC input and `POTFIL`; conditional `input.inp` when `TASK.INPVER=0`, `struc.inp` when `TASK.STRVER=0`.
- **Output files:** `{DATASET_ADSI}_SPEC.out` and `{DATASET_ADSI}_data.spc`, or `{DATASET_ADSI}_data_pol1.spc` and `_data_pol2.spc` for dual-polarization modes.
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** `TASK.IQ_AT_SURF`, `TASK.MILLER_HKL`, `TASK.STRVER`, `TASK.INPVER`, `ENERGY.EWORK_EV`, `ENERGY.IMV_INI_EV`, `ENERGY.IMV_FIN_EV`, `SPEC_PH.THETA`, `SPEC_PH.PHI`, `SPEC_PH.POL_P`, `SPEC_PH.EPHOT`, `SPEC_EL.THETA`, `SPEC_EL.PHI`, `SPEC_EL.NT`, `SPEC_EL.NP`, `SPEC_EL.KA`, `SPEC_EL.K1`, `SPEC_EL.NK1`, `SPEC_EL.K2`, `SPEC_EL.NK2`, `SPEC_EL.POL_E`, `SPEC_EL.SPOL`, `SPEC_STR.N_LAYDBL`, `SPEC_STR.NLAT_G_VEC`, `SPEC_STR.N_LAYER`, `SPEC_STR.SURF_BAR`, `SPEC_STR.TRANSP_BAR`
- **Parameters not covered by the manual:** `TASK.CRYS_VECS`, `SPEC_PH.ALQ`, `SPEC_PH.DELQ`, `SPEC_PH.NPOL`, `SPEC_PH.ICIRC`, `SPEC_PH.IDREH`, `SPEC_PH.IFSP`, `SPEC_PH.THETA_FIX`, `SPEC_EL.K3`, `SPEC_EL.NK3`, `SPEC_EL.K4`, `SPEC_EL.NK4`, `SPEC_EL.PSPIN`, `SPEC_EL.BETA1`, `SPEC_EL.BETA2`, `SPEC_EL.ROTAXIS`, `SPEC_EL.TYP`, `SPEC_EL.ISTR`, `SPEC_EL.POL0`, `SPEC_EL.POL0L`, `SPEC_EL.Q1`, `SPEC_EL.Q2`, `SPEC_EL.Q3`, `SPEC_EL.Q4`

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_arpes_audit.py`, `src/ase2sprkkr/input_parameters/test/test_arpes_energy.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** `ARPES` selected by the task definition (file parsing does not guarantee physical validation).
- **Detailed source/functionality audit:** [ARPES](ARPES-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### BAND

- **What it does:** Calculates complex Bloch bands through a transfer-matrix eigenproblem in KKRSPEC.
- **Status:** Works in the audited workflow; an eigen-solver failure can continue natively as far as `LOG(0)`. This is not `BSF` or `EKREL`.
- **Input files:** Generated KKRSPEC input and `POTFIL`; conditional `input.inp` / `struc.inp` depending on version selectors.
- **Output files:** `{DATASET_ADSI}_SPEC.out`, `{DATASET_ADSI}_data.spc` (native complex-band columns require interpretation).
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** `TASK.IQ_AT_SURF`, `TASK.MILLER_HKL`, `TASK.STRVER`, `TASK.INPVER`, `ENERGY.EWORK_EV`, `ENERGY.IMV_INI_EV`, `ENERGY.IMV_FIN_EV`, `SPEC_PH.THETA`, `SPEC_PH.PHI`, `SPEC_PH.POL_P`, `SPEC_PH.EPHOT`, `SPEC_EL.THETA`, `SPEC_EL.PHI`, `SPEC_EL.NT`, `SPEC_EL.NP`, `SPEC_EL.KA`, `SPEC_EL.K1`, `SPEC_EL.NK1`, `SPEC_EL.K2`, `SPEC_EL.NK2`, `SPEC_EL.POL_E`, `SPEC_EL.SPOL`, `SPEC_STR.N_LAYDBL`, `SPEC_STR.NLAT_G_VEC`, `SPEC_STR.N_LAYER`, `SPEC_STR.SURF_BAR`, `SPEC_STR.TRANSP_BAR`
- **Parameters not covered by the manual:** `TASK.CRYS_VECS`, `SPEC_PH.ALQ`, `SPEC_PH.DELQ`, `SPEC_PH.NPOL`, `SPEC_PH.ICIRC`, `SPEC_PH.IDREH`, `SPEC_PH.IFSP`, `SPEC_PH.THETA_FIX`, `SPEC_EL.K3`, `SPEC_EL.NK3`, `SPEC_EL.K4`, `SPEC_EL.NK4`, `SPEC_EL.PSPIN`, `SPEC_EL.BETA1`, `SPEC_EL.BETA2`, `SPEC_EL.ROTAXIS`, `SPEC_EL.TYP`, `SPEC_EL.ISTR`, `SPEC_EL.POL0`, `SPEC_EL.POL0L`, `SPEC_EL.Q1`, `SPEC_EL.Q2`, `SPEC_EL.Q3`, `SPEC_EL.Q4`, `SPEC_BAND.NEIGMAX`, `SPEC_BAND.EIG_IME`, `SPEC_BAND.CALC_TRANS_PROB`

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_band.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** `SPEC` selected by the task definition (file parsing does not guarantee physical validation).
- **Detailed source/functionality audit:** [BAND](BAND-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### BONDING

- **What it does:** Is intended to calculate an experimental bonding analysis for a molecular cluster.
- **Status:** Broken: normal `TAU_DRIVE` prepares `KTAUIJ=0`, while `CLUBONDING` reads scratch units requiring `KTAUIJ>=2`.
- **Input files:** standard input + `POTFIL`; natively expects internal scratch data on units 18/19, which the normal workflow does not prepare.
- **Output files:** `fort.22`.
- **In the SPR-KKR manual:** No.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** —

- **Interface status:** Registered task identity; source assessment: Broken native workflow.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_bonding.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [BONDING](BONDING-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### BSDOSIJ

- **What it does:** Is intended to calculate site-resolved back-scattering DOS between pairs of sites.
- **Status:** Broken: native `J_ITJT` is written with energy indices into pair dimensions and later read in a different order.
- **Input files:** standard input + `POTFIL`.
- **Output files:** `{DATASET_ADSI}_XCPLTEN_Jij.dat` is declared, but native indexing corrupts its meaning.
- **In the SPR-KKR manual:** No.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** `TASK.CLURAD`, `TASK.NSHLCLU`, `TASK.SELECT`, `TASK.EXCLUDE`, `TASK.QREPONLY`

- **Interface status:** Registered task identity; source assessment: Broken native workflow.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_bsdosij.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [BSDOSIJ](BSDOSIJ-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### BSF

- **What it does:** Calculates the Bloch spectral function either along an E-k path or in a k-k plane at a single energy.
- **Status:** Works; the E-k default `EMAX=-1 Ry` is a suspicious convenience default and need not mean EF.
- **Input files:** standard input + `POTFIL`; may reuse an existing TAU cache.
- **Output files:** `{DATASET_ADSI}.bsf`, optionally `{DATASET_ADSI}_spol.bsf`.
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** `TASK.NK`, `TASK.KPATH`, `TASK.NKDIR`, `TASK.KA*`, `TASK.KE*`, `TASK.NK1`, `TASK.NK2`, `TASK.K1`, `TASK.K2`
- **Parameters not covered by the manual:** —

- **Interface status:** Schema only; not in task registry; source assessment: Meaningful native workflow within stated restrictions.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_bsf_audit.py`, `src/ase2sprkkr/input_parameters/test/test_bsf_file_manifests.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** `BSF` selected by the task definition (file parsing does not guarantee physical validation).
- **Interface detail:** `BSF` is a schema, not an independent registered task. Use `InputParameters.for_task('BSFEK')` or `for_task('BSFKK')` (or `for_definition('BSF')` for the schema).
- **Detailed source/functionality audit:** [BSF](BSF-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### BSFEK

- **What it does:** ASE2SPRKKR alias for `BSF` in E-k mode.
- **Status:** Works like `BSF`; the alias only preselects the expected E-k mode through `ENERGY.NE`.
- **Input files:** Same generated input and potential as the BSF schema; optional reusable TAU cache.
- **Output files:** `{DATASET_ADSI}.bsf`, optionally `{DATASET_ADSI}_spol.bsf` (BSF E–k).
- **In the SPR-KKR manual:** Yes, as `BSF`; the name `BSFEK` is an ASE2SPRKKR alias.
- **Parameters covered by the manual:** `same as BSF E-k: TASK.NK, KPATH, NKDIR, KA*, KE*`
- **Parameters not covered by the manual:** —

- **Interface status:** Registered task identity; source assessment: Meaningful native workflow within stated restrictions.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_bsf_audit.py`, `src/ase2sprkkr/input_parameters/test/test_bsf_file_manifests.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** `BSF` selected by the task definition (file parsing does not guarantee physical validation).
- **Interface detail:** Separate registered task identity; shares the BSF schema and selects E–k via `ENERGY.NE`.
- **Detailed source/functionality audit:** [BSF](BSF-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### BSFKK

- **What it does:** ASE2SPRKKR alias for `BSF` in k-k mode.
- **Status:** Works like `BSF`; the alias only preselects the expected single-energy k-k mode.
- **Input files:** Same generated input and potential as the BSF schema; optional reusable TAU cache.
- **Output files:** `{DATASET_ADSI}.bsf`, optionally `{DATASET_ADSI}_spol.bsf` (BSF k–k).
- **In the SPR-KKR manual:** Yes, as `BSF`; the name `BSFKK` is an ASE2SPRKKR alias.
- **Parameters covered by the manual:** `same as BSF k-k: TASK.KA, K1, K2, NK1, NK2`
- **Parameters not covered by the manual:** —

- **Interface status:** Registered task identity; source assessment: Meaningful native workflow within stated restrictions.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_bsf_audit.py`, `src/ase2sprkkr/input_parameters/test/test_bsf_file_manifests.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** `BSF` selected by the task definition (file parsing does not guarantee physical validation).
- **Interface detail:** Separate registered task identity; shares the BSF schema and selects k–k via `ENERGY.NE`.
- **Detailed source/functionality audit:** [BSF](BSF-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### CHI

- **What it does:** Calculates static magnetic susceptibility in the serial ASA/FREL workflow.
- **Status:** Works in the audited static workflow; native `NCHIITER` is forcibly reset to 1 before the loop, so the iterative magnetization part does not actually run.
- **Input files:** standard input + `POTFIL`.
- **Output files:** native CHI/susceptibility output; no explicit output manifest in the definition.
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** `TASK.CHIPRINT`, `TASK.NOSYMCHI`
- **Parameters not covered by the manual:** `MODE.PRINT`, `MODE.NCHIITER`

- **Interface status:** Registered task identity; source assessment: Meaningful native workflow within stated restrictions.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_chi.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [CHI](CHI-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### CHILANDAU

- **What it does:** Calculates Landau orbital magnetic susceptibility.
- **Status:** Works serially with limitations; MPI and `RDTAUFIL` reuse are broken, and native finish logic prints partial sums.
- **Input files:** standard input + `POTFIL`.
- **Output files:** native Landau-susceptibility output; the last serial block is the relevant result.
- **In the SPR-KKR manual:** No.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** `TASK.CLURAD`, `TASK.NSHLCLU`, `TASK.SELECT`, `TASK.EXCLUDE`, `TASK.QREPONLY`, `TASK.PRINT`

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_chilandau.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [CHILANDAU](CHILANDAU-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### CHIXAS

- **What it does:** Calculates field-induced dichroism in XAS from user-supplied susceptibilities and DOS at EF.
- **Status:** Works only with physically consistent external `NEF/CHISPN/CHIVV`; no universally safe defaults exist.
- **Input files:** standard input + `POTFIL`; externally supplied `NEF*`, `CHISPN`, `CHIVV` values.
- **Output files:** native CHIXAS/XAS response output.
- **In the SPR-KKR manual:** No.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** `TASK.ITXRAY`, `TASK.CL`, `TASK.NEF*`, `TASK.CHISPN`, `TASK.CHIVV`, `TASK.ABSUNIT`, `TASK.PRINT`

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_chixas.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [CHIXAS](CHIXAS-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### CLPES

- **What it does:** Calculates core-level photoemission/photoelectron diffraction in KKRSPEC.
- **Status:** Works in the audited serial fixed-geometry workflow; CPA and some polarization modes stop natively, and with `NE=1` the core-state index must be valid.
- **Input files:** Generated KKRSPEC input and `POTFIL`; conditional `input.inp` (`INPVER=0`) / `struc.inp` (`STRVER=0`).
- **Output files:** `{DATASET_ADSI}_SPEC.out` and polarization-dependent `{DATASET_ADSI}_data*.spc` files.
- **In the SPR-KKR manual:** No.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual: `TASK.IQ_AT_SURF`, `TASK.MILLER_HKL`, `TASK.STRVER`, `TASK.INPVER`, `ENERGY.EWORK_EV`, `ENERGY.IMV_INI_EV`, `ENERGY.IMV_FIN_EV`, `SPEC_PH.THETA`, `SPEC_PH.PHI`, `SPEC_PH.POL_P`, `SPEC_PH.EPHOT`, `SPEC_EL.THETA`, `SPEC_EL.PHI`, `SPEC_EL.NT`, `SPEC_EL.NP`, `SPEC_EL.KA`, `SPEC_EL.K1`, `SPEC_EL.NK1`, `SPEC_EL.K2`, `SPEC_EL.NK2`, `SPEC_EL.POL_E`, `SPEC_EL.SPOL`, `SPEC_STR.N_LAYDBL`, `SPEC_STR.NLAT_G_VEC`, `SPEC_STR.N_LAYER`, `SPEC_STR.SURF_BAR`, `SPEC_STR.TRANSP_BAR`, `TASK.CRYS_VECS`, `SPEC_PH.ALQ`, `SPEC_PH.DELQ`, `SPEC_PH.NPOL`, `SPEC_PH.ICIRC`, `SPEC_PH.IDREH`, `SPEC_PH.IFSP`, `SPEC_PH.THETA_FIX`, `SPEC_EL.K3`, `SPEC_EL.NK3`, `SPEC_EL.K4`, `SPEC_EL.NK4`, `SPEC_EL.PSPIN`, `SPEC_EL.BETA1`, `SPEC_EL.BETA2`, `SPEC_EL.ROTAXIS`, `SPEC_EL.TYP`, `SPEC_EL.ISTR`, `SPEC_EL.POL0`, `SPEC_EL.POL0L`, `SPEC_EL.Q1`, `SPEC_EL.Q2`, `SPEC_EL.Q3`, `SPEC_EL.Q4`, `SPEC_CL.IT`, `SPEC_CL.CL`, `SPEC_CL.IE_ICST`

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_clpes.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** `ARPES` selected by the task definition (file parsing does not guarantee physical validation).
- **Detailed source/functionality audit:** [CLPES](CLPES-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### CLXPS

- **What it does:** Calculates fully relativistic core-level XPS in KKRGEN.
- **Status:** Works in the audited ASA/FREL workflow; advanced angular geometries and numerical output were not independently certified.
- **Input files:** standard input + `POTFIL`.
- **Output files:** native CL-XPS spectrum (`DATASET.pes`; PLOT/postprocessing may create `.xps`/xmgrace files).
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** `TASK.IT`, `TASK.CL`, `TASK.EPHOT`, `TASK.ImE`, `TASK.USETSS`
- **Parameters not covered by the manual:** `TASK.ME`, `TASK.MECHECK`, `TASK.LINPOL`, `TASK.ANGRES`, `TASK.ANGQ0..10`, `TASK.ANGE0..10`, `TASK.ANGK0..10`

- **Interface status:** Registered task identity; source assessment: Meaningful native workflow within stated restrictions.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_clxps.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [CLXPS](CLXPS-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### COMPTON

- **What it does:** Calculates electron Compton momentum density and profiles.
- **Status:** Works with limitations; `EMD2D` double-scales nonzero `PNMAX`, and shifted even grids lose their endpoints.
- **Input files:** Generated input + `POTFIL` + required compatible electronic TAU directory (`TAUDIR`/`TAUPATH`).
- **Output files:** `{DATASET_ADSI}_BND.mcp`, `{DATASET_ADSI}_COR.mcp`, `{DATASET_ADSI}_TOT.mcp`.
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** `TASK.MD2D`, `TASK.SHFT`, `TASK.PNVEC`, `TASK.NPN`, `TASK.NPP`, `TASK.PNMAX`, `TASK.PPMAX`
- **Parameters not covered by the manual:** `TASK.SPLITSS`, `TASK.MEMODE`, `TASK.NPMAX`, `TASK.USE_PEQUIV`, `TASK.PPVEC1`, `TASK.PPVEC2`, `TASK.EMD2D`, `TASK.SHFT_PP`, `TASK.SHFT_PN`, `TASK.INSIDE_SPHERE`, `TASK.SQQ_RESOLVED`

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_compton.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [COMPTON](COMPTON-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### DMI

- **What it does:** ASE2SPRKKR alias of the shared JXC/DMI definition for calculating the Dzyaloshinskii-Moriya vector.
- **Status:** Works with significant caveats: the individual DMI components are not equivalent in the native source (especially Dz), and frame/multiplicity/radius must be checked.
- **Input files:** standard input + `POTFIL`.
- **Output files:** `{DATASET_ADSI}_DMIVEC_Dij.dat`; optional component plots `{DATASET_ADSI}_DMIVEC_Dij_[xyz]_*.agr`.
- **In the SPR-KKR manual:** No as a separate task in the available manual; JXC is documented.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** `TASK.DMI`, `TASK.JXC`, `TASK.CLURAD`, `TASK.NSHLCLU`, `TASK.SELECT`, `TASK.EXCLUDE`, `TASK.QREPONLY`

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_dmi.py`, `src/ase2sprkkr/input_parameters/test/test_exchange_file_manifests.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** `JXC` selected by the task definition (file parsing does not guarantee physical validation).
- **Interface detail:** Shares the JXC schema but has its own registered task identity; native DMI-only and JXC+DMI requests are different.
- **Detailed source/functionality audit:** [DMI](DMI-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### DOS

- **What it does:** Calculates the density of states.
- **Status:** Works.
- **Input files:** standard input + `POTFIL`.
- **Output files:** `{DATASET_ADSI}.dos`.
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** —

- **Interface status:** Registered task identity; source assessment: Meaningful native workflow within stated restrictions.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_dos.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** `DOS` selected by the task definition (file parsing does not guarantee physical validation).
- **Detailed source/functionality audit:** [DOS](DOS-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### EKREL

- **What it does:** Calculates E(k) dispersion or searches for the Fermi surface in ordered systems.
- **Status:** Works; CPA is rejected natively, and BSF should be used for disorder.
- **Input files:** standard input + `POTFIL`.
- **Output files:** native EKREL band/Fermi-surface data.
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** `TASK.NK`, `TASK.EMIN`, `TASK.EMAX`, `TASK.NE`, `TASK.F-SURF`, `TASK.ESLICE`, `TASK.KPATH`, `TASK.NKDIR`, `TASK.KA1..9`, `TASK.KE1..9`
- **Parameters not covered by the manual:** —

- **Interface status:** Registered task identity; source assessment: Meaningful native workflow within stated restrictions.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_ekrel.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [EKREL](EKREL-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### FMAG

- **What it does:** Calculates spin/orbital density and the magnetic form factor.
- **Status:** Works with a limitation: a zero magnetic moment can lead to division by zero during normalization.
- **Input files:** standard input + `POTFIL`.
- **Output files:** `{DATASET_ADSI}_fmag_AT.agr` and `{DATASET_ADSI}_dens_AT.agr` for individual types.
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** —

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_fmag.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [FMAG](FMAG-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### FSCAT

- **What it does:** Calculates scattering-amplitude matrices for three fixed backscattering geometries.
- **Status:** Broken: an erroneous spherical-harmonic `+L` index can overflow, and phase is calculated before zero amplitude is handled.
- **Input files:** standard input + `POTFIL`.
- **Output files:** native scattering-amplitude diagnostic files.
- **In the SPR-KKR manual:** No.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** —

- **Interface status:** Registered task identity; source assessment: Broken native workflow.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_fscat.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [FSCAT](FSCAT-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### GILBERT

- **What it does:** Calculates Gilbert damping, optionally including temperature-dependent spin and lattice fluctuations.
- **Status:** Works for suitable magnetic systems; zero or compensated moments can cause division by zero.
- **Input files:** standard input + `POTFIL`; with `SETFLUCT=M_T`, `TASK.FLUCTFIL` is required.
- **Output files:** `{DATASET_ADSI}alfa_x.dat`, `{DATASET_ADSI}alfa_T.dat`, `{DATASET_ADSI}_alfa_T.agr` (note the native naming conventions).
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** `TASK.SETFLUCT`, `TASK.TMPMIN`, `TASK.TMPMAX`, `TASK.NTMP`, `TASK.NVIBRA`
- **Parameters not covered by the manual:** `TASK.FLUCTFIL`, `TASK.NFTET`, `TASK.NFPHI`, `TASK.DMAGDT`, `TASK.TDEBYE`, `TASK.INITSVEC`

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_gilbert.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Interface detail:** The current task definition enables MPI; this is *not* evidence of verified serial/MPI numerical agreement.
- **Detailed source/functionality audit:** [GILBERT](GILBERT-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### JALF

- **What it does:** Calculates valence current density from Dirac α matrices and the associated magnetic moment.
- **Status:** Works with limitations; helper diagnostics contain bugs and small `NL` is risky, which the definition guards against.
- **Input files:** standard input + `POTFIL`.
- **Output files:** `{DATASET}_JALF.dat`, with fixed-name diagnostics `curdns1`, `curdns2`, `curdns3`.
- **In the SPR-KKR manual:** No.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** —

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_jalf.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [JALF](JALF-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### JORB

- **What it does:** Calculates relativistic paramagnetic orbital current density.
- **Status:** Works serially; native MPI does not reach the final JORB output and does not reduce the accumulators.
- **Input files:** standard input + `POTFIL`.
- **Output files:** `{DATASET}_JORB.dat`, `{DATASET}_JORB_UP.dat`, `{DATASET}_JORB_DN.dat`; possible `fort.<unit>` diagnostics.
- **In the SPR-KKR manual:** No.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** —

- **Interface status:** Registered task identity; source assessment: Meaningful native workflow within stated restrictions.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_jorb.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [JORB](JORB-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### JXC

- **What it does:** Calculates Heisenberg exchange couplings and, in FREL, tensor components.
- **Status:** Works for a simple periodic collinear workflow; the general FREL tensor for symmetry-equivalent neighbors has a native rotation bug, and the cluster SP-SREL scratch flow is unreliable.
- **Input files:** standard input + `POTFIL`.
- **Output files:** Depending on relativistic mode: `{DATASET_ADSI}_J_ij.dat` / `{DATASET_ADSI}_Jij_*.agr` or `{DATASET_ADSI}_XCPLTEN_Jij.dat` / `..._Jij_*.agr`; optional DMI files when enabled.
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** `TASK.CLURAD`
- **Parameters not covered by the manual:** `TASK.JXC`, `TASK.DMI`, `TASK.NSHLCLU`, `TASK.SELECT`, `TASK.EXCLUDE`, `TASK.QREPONLY`

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_jxc.py`, `src/ase2sprkkr/input_parameters/test/test_jxc_definition.py`, `src/ase2sprkkr/input_parameters/test/test_exchange_file_manifests.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** `JXC` selected by the task definition (file parsing does not guarantee physical validation).
- **Detailed source/functionality audit:** [JXC](JXC-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### MECHECK

- **What it does:** Diagnostically compares NAB and ADA matrix elements.
- **Status:** Only partially works: `NMEFORM` is always overwritten to 2 natively, another consistency test is disabled at compile time, and the summary loop is faulty.
- **Input files:** standard input + `POTFIL`.
- **Output files:** native matrix-element diagnostics; no explicit output manifest.
- **In the SPR-KKR manual:** No.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** `TASK.ITSEL`, `TASK.PRINT`, `TASK.INITIALSTATE`, `TASK.CL`, `TASK.ERYDI`, `TASK.ERYDF`, `TASK.NONMAG`, `TASK.NMEFORM`

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_mecheck.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [MECHECK](MECHECK-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### MEPLOT

- **What it does:** Calculates/plots dipole matrix elements for BAND or CORE initial states.
- **Status:** Works except for native curve overwriting in simultaneous photon+electron scans; the definition rejects that combination. The meaning of `SEFIN` still requires physical verification.
- **Input files:** standard input + `POTFIL`.
- **Output files:** native `.agr`/matrix-element plot data; no explicit output manifest.
- **In the SPR-KKR manual:** No.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** `TASK.ITSEL`, `TASK.IPRINT`, `TASK.INITIALSTATE`, `TASK.L`, `TASK.CL`, `TASK.NONMAG`, `TASK.SEFIN`, `TASK.EPHOT`, `TASK.NEPHOT`, `TASK.EPHOTMIN`, `TASK.EPHOTMAX`

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_meplot.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [MEPLOT](MEPLOT-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### NRAES

- **What it does:** Calculates nonrelativistic Auger spectra by postprocessing an existing DOS.
- **Status:** Broken: `IAES` is only partially zeroed and convolution accumulates into uninitialized regions.
- **Input files:** Generated input + `POTFIL` + existing `{DATASET_ADSI}.dos` (old spin-resolved DOS format).
- **Output files:** `{DATASET_ADSI}.<type>.<core-shell>.aes`; optional CALCLS/OUTGNU auxiliary files (native numerical routine is defective).
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** `TASK.IT`, `TASK.CL`, `TASK.OUTGNU`, `TASK.CALCLS`
- **Parameters not covered by the manual:** —

- **Interface status:** Registered task identity; source assessment: Broken native workflow.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_nraes.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [NRAES](NRAES-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### NUCSSCPL

- **What it does:** Is intended to calculate the nuclear spin-spin coupling tensor.
- **Status:** Broken: native code deallocates arrays before first allocation, clears matrix elements for each type, and forces the mesh to 1.
- **Input files:** standard input + `POTFIL`.
- **Output files:** `{DATASET_ADSI}_A_ij.dat`, `RK.dat`, `tab_aij.dat`.
- **In the SPR-KKR manual:** No.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** `TASK.CLURAD`, `TASK.NSHLCLU`, `TASK.SELECT`, `TASK.EXCLUDE`

- **Interface status:** Registered task identity; source assessment: Broken native workflow.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_scattering_file_manifests.py`, `src/ase2sprkkr/input_parameters/test/test_dataset_filenames.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [NUCSSCPL](NUCSSCPL-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### OPTABS

- **What it does:** Calculates the absorptive part of the optical conductivity.
- **Status:** Works with major caveats: `TEMPEL<=1` leaves `KBT` uninitialized and `NOMEGA<NEB` can overflow the allocation; the definition rejects these states.
- **Input files:** standard input + `POTFIL`.
- **Output files:** native OPTICS-ABS conductivity output.
- **In the SPR-KKR manual:** No.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** `TASK.NEB`, `TASK.NOMEGA`, `TASK.OMEGA1`, `TASK.OMEGA2`, `TASK.TEMPEL`, `TASK.NKT`, `TASK.FRAMETET`, `TASK.FRAMEPHI`, `TASK.FRAMEGAM`

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_optabs.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [OPTABS](OPTABS-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### PHAGEN

- **What it does:** Exports spherical potential/charge/Coulomb data for PHAGEN/MsSpec.
- **Status:** Works as a restricted ASA/VWN converter; NT=1, no disorder, and it is not a general phase-shift solver.
- **Input files:** Generated input + converged, nonmagnetic ASA/VWN potential selected by `POTFIL` (NT=1; not a fresh phase-shift calculation).
- **Output files:** `<POTFIL>_ex_sprkkr` and `<POTFIL>.only_coulomb_new` (derived from `POTFIL`, not `DATASET`).
- **In the SPR-KKR manual:** No.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** —

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_phagen.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** Native PHAGEN export integration test passed for converged Au, verifying both output files; no comparison with MsSpec or independent physical benchmark.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [PHAGEN](PHAGEN-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### PLOT2DPOT

- **What it does:** Is intended to plot a 2D potential slice in a user-defined plane.
- **Status:** Works in ASA for potential; the FULLPOT branch enters an experimental PLY writer and exits before the normal plane-grid output.
- **Input files:** standard input + `POTFIL`.
- **Output files:** Fixed PLY names as in PLOT2DPR; ASA produces potential PLY, FULLPOT uses the incomplete experimental PLY route.
- **In the SPR-KKR manual:** No.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** `TASK.AVEC`, `TASK.BVEC`, `TASK.DVEC`, `TASK.NA`, `TASK.NB`

- **Interface status:** Registered task identity; source assessment: Declared purpose or main branch not reliably achieved.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_plot2dpr.py`, `src/ase2sprkkr/input_parameters/test/test_plot_selector_file_manifests.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Interface detail:** `TASK.AVEC/BVEC/DVEC` use Cartesian **ALAT units**.
- **Detailed source/functionality audit:** [PLOT2D-selectors](PLOT2D-selectors-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### PLOT2DPR

- **What it does:** Is intended to plot a general 2D slice through stored radial/potential data.
- **Status:** Partially works: in ASA it actually writes only the potential; the FULLPOT branch exits in the experimental PLY writer.
- **Input files:** standard input + `POTFIL`.
- **Output files:** Fixed PLY names: `PLY-V-ASA-*.ply`, `POT-ASA-LINES.ply` (ASA) or `PLY-V-FP-*.ply`, `PLY-V-MT-*.ply`, `PLY-FP-*.ply` and related files (FULLPOT experimental path).
- **In the SPR-KKR manual:** No.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** `TASK.AVEC`, `TASK.BVEC`, `TASK.DVEC`, `TASK.NA`, `TASK.NB`

- **Interface status:** Registered task identity; source assessment: Declared purpose or main branch not reliably achieved.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_plot2dpr.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Interface detail:** `TASK.AVEC/BVEC/DVEC` are Cartesian vectors in **ALAT units** (not direct Bohr or fractional cell coordinates).
- **Detailed source/functionality audit:** [PLOT2DPR](PLOT2DPR-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### PLOT2DRHO

- **What it does:** Is intended to plot a 2D slice of the electron density.
- **Status:** Practically incomplete: ASA cannot plot density; FULLPOT exits in the test PLY writer before the normal plane-grid writer.
- **Input files:** standard input + FULLPOT `POTFIL`.
- **Output files:** Fixed PLY names as in PLOT2DPR; ASA cannot deliver density, FULLPOT selectivity is not implemented.
- **In the SPR-KKR manual:** No.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** `TASK.AVEC`, `TASK.BVEC`, `TASK.DVEC`, `TASK.NA`, `TASK.NB`

- **Interface status:** Registered task identity; source assessment: Declared purpose or main branch not reliably achieved.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_plot2dpr.py`, `src/ase2sprkkr/input_parameters/test/test_plot_selector_file_manifests.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Interface detail:** `TASK.AVEC/BVEC/DVEC` use Cartesian **ALAT units**.
- **Detailed source/functionality audit:** [PLOT2D-selectors](PLOT2D-selectors-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### PLOTPOT

- **What it does:** Writes stored radial potential data for plotting without a new SCF/DOS calculation.
- **Status:** Works for stored ASA and FULLPOT potential data.
- **Input files:** standard input + `POTFIL`.
- **Output files:** native radial-potential plot files.
- **In the SPR-KKR manual:** No.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** —

- **Interface status:** Registered task identity; source assessment: Meaningful native workflow within stated restrictions.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_plotprs.py`, `src/ase2sprkkr/input_parameters/test/test_plot_selector_file_manifests.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [PLOT-selectors](PLOT-selectors-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### PLOTPRS

- **What it does:** Writes stored potential, charge/spin density, and FULLPOT shape-function plots.
- **Status:** Works with limitations: magnetic spin/orbital curves can collide in the same dataset, and panel limits ignore the orbital contribution.
- **Input files:** standard input + `POTFIL`.
- **Output files:** native radial potential/density/shape-function plot files.
- **In the SPR-KKR manual:** No.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** —

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_plotprs.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [PLOTPRS](PLOTPRS-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### PLOTRHO

- **What it does:** Is intended to write stored radial density as a standalone plot output.
- **Status:** Broken: `RHORSQ` is allocated only in the potential branch, but the standalone rho writer needs it.
- **Input files:** standard input + `POTFIL`.
- **Output files:** native density-plot output, but the normal standalone run is unreliable.
- **In the SPR-KKR manual:** No.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** —

- **Interface status:** Registered task identity; source assessment: Broken native workflow.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_plotprs.py`, `src/ase2sprkkr/input_parameters/test/test_plot_selector_file_manifests.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [PLOT-selectors](PLOT-selectors-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### PLOTSFN

- **What it does:** Writes stored radial FULLPOT shape-function coefficients.
- **Status:** Works for a genuine FULLPOT potential containing shape-function data.
- **Input files:** standard input + FULLPOT `POTFIL`.
- **Output files:** native shape-function plot files.
- **In the SPR-KKR manual:** No.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** —

- **Interface status:** Registered task identity; source assessment: Meaningful native workflow within stated restrictions.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_plotprs.py`, `src/ase2sprkkr/input_parameters/test/test_plot_selector_file_manifests.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [PLOT-selectors](PLOT-selectors-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### POSANI

- **What it does:** Calculates positron-annihilation momentum density and profiles.
- **Status:** Works only with a compatible `POSANIPREP`; the native data path assumes `NQ=NT`, and general `NQ!=NT` is broken.
- **Input files:** Generated input + electron `POTFIL` + required electron TAU directory + `{DATASET_ADSI}_pos_dat.pan` from POSANIPREP.
- **Output files:** `{DATASET_ADSI}_BND.mpa`, `_COR.mpa`, `_TOT.mpa`; depending on options also `..._pan2d.dat` or `..._data.mpa`.
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** `TASK.MD2D`, `TASK.SHFT`, `TASK.PNVEC`, `TASK.NPN`, `TASK.NPP`, `TASK.PNMAX`, `TASK.PPMAX`
- **Parameters not covered by the manual:** `TASK.SPLITSS`, `TASK.MEMODE`, `TASK.NPMAX`, `TASK.USE_PEQUIV`, `TASK.PPVEC1`, `TASK.PPVEC2`, `TASK.EMD2D`, `TASK.SHFT_PP`, `TASK.SHFT_PN`, `TASK.INSIDE_SPHERE`, `TASK.SQQ_RESOLVED`

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_posani.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [POSANI](POSANI-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### POSANIPREP

- **What it does:** Prepares the s-like positron state, wave functions, and scattering data for `POSANI`.
- **Status:** Works for a simple 3D ASA/FREL `NQ=NT` case; TAUQQ export is broken for general `NQ!=NT`. The positron-band-bottom energy must be chosen by the user.
- **Input files:** standard input + electronic `POTFIL`.
- **Output files:** `{DATASET_ADSI}_pos_dat.pan`.
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** `TASK.POSVC`

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_posaniprep.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [POSANIPREP](POSANIPREP-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### POSANITAU

- **What it does:** Integrates positron density and calculates the annihilation rate/lifetime.
- **Status:** Works in the audited 3D ASA/FREL workflow; integration-energy bounds need physical validation and the enhancement factor is fixed natively.
- **Input files:** Generated input + electron `POTFIL`; optional existing `{DATASET_ADSI}_positron.rho` cache.
- **Output files:** Native annihilation rate/lifetime in run output; optional `{DATASET_ADSI}_positron.rho` cache.
- **In the SPR-KKR manual:** No.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** `TASK.POSVC`

- **Interface status:** Registered task identity; source assessment: Meaningful native workflow within stated restrictions.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_posanitau.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [POSANITAU](POSANITAU-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### PSHIFT

- **What it does:** Calculates/plots scattering phase shifts.
- **Status:** Works, but resonance splitting is unreliable in some cases.
- **Input files:** standard input + `POTFIL`.
- **Output files:** native phase-shift plot/data files.
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** —

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_pshift.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [PSHIFT](PSHIFT-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### SCF

- **What it does:** Performs a self-consistent calculation and produces a new potential.
- **Status:** Works for `BROYDEN2`, `ANDERSON`, and `TCHEBY` within the allowed scope; `PULAY` is not a valid choice and `TCHEBY` is unavailable for FULLPOT.
- **Input files:** standard input + starting `POTFIL`; FULLPOT may reuse `CONTROL.SFNFIL`.
- **Output files:** `<POTFIL>_new` and standard SCF log; a start-up DOS may also be produced.
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** `SCF.VXC`, `SCF.ALG`, `SCF.NITER`, `SCF.MIX`, `SCF.TOL`, `SCF.ISTBRY`, `SCF.ITDEPT`, `SCF.QION`, `SCF.MSPIN`, `SCF.USEVMATT`
- **Parameters not covered by the manual:** `SCF.MIXOP`, `SCF.EFGUESS`

- **Interface status:** Registered task identity; source assessment: Meaningful native workflow within stated restrictions.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_scf_audit.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** `SCF` selected by the task definition (file parsing does not guarantee physical validation).
- **Detailed source/functionality audit:** [SCF](SCF-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### SIGMA

- **What it does:** Calculates DC conductivity/Kubo-Středa response, either surface-only or sea+surface.
- **Status:** Works with limitations; the Fermi-sea contribution is not included automatically, and the modes use fixed file conventions/ImE.
- **Input files:** standard input + `POTFIL`.
- **Output files:** Native fixed names `sigma?????.dat` (not isolated by `DATASET`).
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** `TASK.SIGSPROJ`, `TASK.SP_NO_ALPHA`, `TASK.SP_NO_NABLA`, `TASK.NOSYMSIG`, `TASK.FRAMETET`, `TASK.FRAMEPHI`, `TASK.FRAMEGAM`

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_sigma.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [SIGMA](SIGMA-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### SOCPAR

- **What it does:** Calculates spin-orbit coupling parameters.
- **Status:** Works for the main SOC output; exchange-related output has a wrong unit label and one sign-flip test is unreachable in the native code.
- **Input files:** standard input + `POTFIL`.
- **Output files:** native SOCPAR tables/plot data.
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** —

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_socpar.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [SOCPAR](SOCPAR-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### SPLEED

- **What it does:** Calculates spin-polarized low-energy electron diffraction in KKRSPEC.
- **Status:** Works in a conservative fixed-angle workflow; momentum-scan and alternative-polarization branches were not audited.
- **Input files:** Generated KKRSPEC input and `POTFIL`; conditional `input.inp` / `struc.inp` depending on version selectors.
- **Output files:** `{DATASET_ADSI}_SPEC.out`, `{DATASET_ADSI}_data.spc` (SPC may contain only a header).
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** `TASK.IQ_AT_SURF`, `TASK.MILLER_HKL`, `TASK.STRVER`, `TASK.INPVER`, `ENERGY.EWORK_EV`, `ENERGY.IMV_INI_EV`, `ENERGY.IMV_FIN_EV`, `SPEC_PH.THETA`, `SPEC_PH.PHI`, `SPEC_PH.POL_P`, `SPEC_PH.EPHOT`, `SPEC_EL.THETA`, `SPEC_EL.PHI`, `SPEC_EL.NT`, `SPEC_EL.NP`, `SPEC_EL.KA`, `SPEC_EL.K1`, `SPEC_EL.NK1`, `SPEC_EL.K2`, `SPEC_EL.NK2`, `SPEC_EL.POL_E`, `SPEC_EL.SPOL`, `SPEC_STR.N_LAYDBL`, `SPEC_STR.NLAT_G_VEC`, `SPEC_STR.N_LAYER`, `SPEC_STR.SURF_BAR`, `SPEC_STR.TRANSP_BAR`
- **Parameters not covered by the manual: `TASK.CRYS_VECS`, `SPEC_PH.ALQ`, `SPEC_PH.DELQ`, `SPEC_PH.NPOL`, `SPEC_PH.ICIRC`, `SPEC_PH.IDREH`, `SPEC_PH.IFSP`, `SPEC_PH.THETA_FIX`, `SPEC_EL.K3`, `SPEC_EL.NK3`, `SPEC_EL.K4`, `SPEC_EL.NK4`, `SPEC_EL.PSPIN`, `SPEC_EL.BETA1`, `SPEC_EL.BETA2`, `SPEC_EL.ROTAXIS`, `SPEC_EL.TYP`, `SPEC_EL.ISTR`, `SPEC_EL.POL0`, `SPEC_EL.POL0L`, `SPEC_EL.Q1`, `SPEC_EL.Q2`, `SPEC_EL.Q3`, `SPEC_EL.Q4`

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_spleed.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** `SPEC` selected by the task definition (file parsing does not guarantee physical validation).
- **Detailed source/functionality audit:** [SPLEED](SPLEED-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### STONER

- **What it does:** Calculates the Stoner parameter from the DOS and radial wave functions at EF.
- **Status:** Works with limitations; it uses a hard-coded VWN susceptibility kernel, natively replaces small DOS values with unit contributions, and some zero normalizations are unguarded.
- **Input files:** standard input + `POTFIL`; may reuse a TAU cache.
- **Output files:** native STONER result in the standard output.
- **In the SPR-KKR manual:** No.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** —

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_stoner.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [STONER](STONER-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### T1

- **What it does:** Is intended to calculate nuclear spin-lattice relaxation at EF.
- **Status:** Broken for numerical use: native `RELT1` treats signed κ values as positive counts of mj channels, so negative channels disappear and basis indexing is wrong.
- **Input files:** standard input + `POTFIL`.
- **Output files:** native T1/RELT1 output, but the numerical rates are not trustworthy.
- **In the SPR-KKR manual:** No.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** —

- **Interface status:** Registered task identity; source assessment: Broken native workflow.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_t1.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [T1](T1-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### TDOBS

- **What it does:** Is intended to represent a time-dependent-observable task, but in 9.7 it is effectively a generic DOS/observable energy loop with attempted TAU reuse.
- **Status:** The declared purpose is not demonstrated: no separate time-dependent-observable implementation was found; as a generic DOS path it can run serially.
- **Input files:** standard input + `POTFIL`; may reuse `TAUDIR`/`TAUPATH` cache.
- **Output files:** `{DATASET_ADSI}.dos`.
- **In the SPR-KKR manual:** No.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** `TAU.TAUDIR`, `TAU.TAUPATH`

- **Interface status:** Registered task identity; source assessment: Declared purpose or main branch not reliably achieved.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_tdobs.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [TDOBS](TDOBS-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### TORQUE

- **What it does:** Calculates magnetic torque for pre-rotated magnetic moments.
- **Status:** Works, but requires a rotated magnetic potential/geometry; a normal unrotated magnetic potential may stop natively, and mixed occupancy uses the first type.
- **Input files:** standard input + rotated magnetic `POTFIL`.
- **Output files:** native TORQUE output.
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** `TASK.THETAQ`, `TASK.PHIQ`
- **Parameters not covered by the manual:** —

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_torque.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [TORQUE](TORQUE-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### TUTORIAL

- **What it does:** Runs native educational/diagnostic branches for YLM, JLX, radial-wavefunction, and DOS demonstrations.
- **Status:** Mixed: YLM/JLX/JLX-ASYMPT and DOS-FEG/DOS-SS/DOS-CLU/DOS-BULK/GEN are usable; CHR-POT, POT-DECOMP/POT-XC, RWF-L/RWF-E, and CORE-ALL have specific native bugs/limitations.
- **Input files:** standard input + `POTFIL` (even purely mathematical demonstrations still pass through native startup that expects it).
- **Output files:** various native `.agr`/diagnostic/DOS outputs depending on `TUT-TASK`.
- **In the SPR-KKR manual:** No.
- **Parameters covered by the manual:** —
- **Parameters not covered by the manual:** `TASK.TUT-TASK`, `TASK.L`, `TASK.M`, `TASK.LMAX`, `TASK.NX`, `TASK.XMIN`, `TASK.XMAX`, `TASK.X-RANGE`, `TASK.ITSEL`, `TASK.NORMALISATION`

- **Interface status:** Registered task identity; source assessment: Mixed branches; see the detailed TUTORIAL audit.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_tutorial.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [TUTORIAL](TUTORIAL-complete-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### VBPES

- **What it does:** Calculates angle-integrated valence photoemission in KKRGEN, separately from KKRSPEC ARPES.
- **Status:** Works with limitations; VBUPS does not automatically select a UV photon energy natively, and the calculation depends on a compatible TAU/final-state setup.
- **Input files:** Generated input + `POTFIL` + required initial-state TAU; optional final-state `{DATASET_ADSI}.tau.dir.F_<photon>` cache.
- **Output files:** Fixed `arpes.dat`, `arpes_spol.dat`, `spin.matrix.xps`; optional final-state `{DATASET_ADSI}.dos.F_<photon>`.
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** `TASK.EPHOT`, `TASK.USETAUNN`
- **Parameters not covered by the manual:** `TASK.DEFEP0`, `TASK.ME`, `TASK.MODE`, `TASK.EWORK`, `TASK.GEOMETRY`, `TASK.QVEC`, `TASK.QVECPH`, `TASK.TETPH`, `TASK.PHIPH`, `TASK.NSPIN`, `TASK.ImEFIN`, `TASK.USETFIN`, `ENERGY.ERANGE`

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_vbpes.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [VBPES](VBPES-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### WFPLOT

- **What it does:** Plots radial BAND or CORE wave functions.
- **Status:** Works in the ASA/FREL serial workflow; BAND uses only the first energy point and FULLPOT is unsupported.
- **Input files:** standard input + `POTFIL`.
- **Output files:** native xmgrace/radial-wavefunction plot files.
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** `TASK.ITSEL`, `TASK.STATE`, `TASK.PRINT`, `TASK.L`, `TASK.CL`, `TASK.MJ`
- **Parameters not covered by the manual:** —

- **Interface status:** Registered task identity; source assessment: Meaningful native workflow within stated restrictions.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_wfplot.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [WFPLOT](WFPLOT-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### XAS

- **What it does:** Calculates X-ray absorption spectra through XRAYSPEC.
- **Status:** Works with known caveats; the static branch is usable, but TAU compatibility and the effective start of the energy scan must be checked. Native MPI exits before XRAYSPEC.
- **Input files:** standard input + `POTFIL`; may reuse a TAU cache.
- **Output files:** `{DATASET_ADSI}_*_<core-shell>.rat` (atom/shell-dependent native rate file), along with other XRAYSPEC diagnostics.
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** `TASK.IT`, `TASK.CL`, `TASK.OUTPUT`
- **Parameters not covered by the manual:** `TASK.ME`, `TASK.POLOBS`, `TASK.DIPOL`, `TASK.GFCONVRH`, `TASK.WRREGIRR`, `TASK.NONDIPE2`, `TASK.NONDIPM1`, `TASK.FRAMETET`, `TASK.FRAMEPHI`, `TASK.FRAMEGAM`

- **Interface status:** Registered task identity; source assessment: Meaningful restricted branch; important caveats.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_xas.py`, `src/ase2sprkkr/input_parameters/test/test_xas_definition.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [XAS](XAS-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### XES

- **What it does:** Calculates X-ray emission through the common XRAYSPEC reader.
- **Status:** Not fully verified: RXES ends in the shared absorption-rate branch and output naming/interpretation is shared with XAS; the physical emission meaning was not confirmed.
- **Input files:** standard input + `POTFIL`; compatible occupied-state TAU/cache.
- **Output files:** `{DATASET_ADSI}_*_<core-shell>.rat` (the native calculation shares absorption-rate logic; emission interpretation not certified).
- **In the SPR-KKR manual:** Yes.
- **Parameters covered by the manual:** `TASK.IT`, `TASK.CL`
- **Parameters not covered by the manual:** `TASK.ME`, `TASK.OUTPUT`, `TASK.POLOBS`, `TASK.DIPOL`, `TASK.TAUCORE`, `TASK.GFCONVRH`, `TASK.WRREGIRR`, `TASK.NONDIPE2`, `TASK.NONDIPM1`, `TASK.FRAMETET`, `TASK.FRAMEPHI`, `TASK.FRAMEGAM`

- **Interface status:** Registered task identity; source assessment: Declared purpose or main branch not reliably achieved.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_xes.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [XES](XES-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

### XRS

- **What it does:** Calculates X-ray resonant scattering through a three-integration XRAYSPEC workflow.
- **Status:** Broken for reliable numerical results: native `SIG(NEMAX+50,...)` is zeroed only up to `NEMAX`, but is later summed over the full range.
- **Input files:** standard input + `POTFIL`; compatible TAU/cache.
- **Output files:** `{DATASET_ADSI}_*_<core-shell>.rat` (native tensor contents affected by uninitialized memory).
- **In the SPR-KKR manual:** Yes (XRS is at least listed as a supported spectroscopy task; the detailed 9.7 interface is broader).
- **Parameters covered by the manual:** `TASK.IT`, `TASK.CL`
- **Parameters not covered by the manual:** `TASK.ME`, `TASK.NE3`, `TASK.EMAX3`, `TASK.TSELECT`, `TASK.POLOBS`, `TASK.DIPOL`, `TASK.TAUCORE`, `TASK.GFCONVRH`, `TASK.WRREGIRR`, `TASK.NONDIPE2`, `TASK.NONDIPM1`, `TASK.FRAMETET`, `TASK.FRAMEPHI`, `TASK.FRAMEGAM`

## Native SPR-KKR tasks intentionally not exposed as normal ASE2SPRKKR definitions

These are not counted among the 56 user-facing names above, but they are relevant to a task audit because their absence is intentional rather than an omission:

- **BIS** — the native parser recognizes the task, but the VBPES routine contains a guard that inevitably stops for `BIS`; no ASE2SPRKKR definition should pretend that inverse photoemission works.
- **SIGMA-XAS** — the source branch exists, but the standard input parser cannot set internal `TASK=SIGMA-XAS`; no normal definition is exposed.
- **PLOTJALF / PLOTJORB** — selectors exist around the plotting/current code, but they are not coherent standalone calculation tasks; FULLPOT exits in the PLY writer and ASA does not produce the intended current plot.
- **FORCETEN, XMO, OPTICS, MAGNET, CHIDYN, PHONONS, PUMP, OPM, SIGMA-DNS, NEGF** — disabled/unavailable public-source branches (typically an explicit stop or unfinished routine), and are therefore intentionally not presented as supported ASE2SPRKKR tasks.

## Main sources used

1. Current ASE2SPRKKR `tasks` branch definitions under `src/ase2sprkkr/input_parameters/definitions/`.
2. Consolidated October 2026 source audit `SPRKKR-ASE2SPRKKR-common-audit.md` and the individual `*-audit.md` task reports.
3. Public SPR-KKR User's Guide repository entry (`SPRKKR 8.6 Manual`, submitted 2022-02-08) and searchable public User's Guide text for explicit task/parameter coverage.
4. Public SPR-KKR 9.7 source package `PUB9.7_260420_18h13` used by the individual audits.

## Caveats worth keeping with this document

- Manual coverage is deliberately **not inferred from source code**. A parameter can be valid in SPR-KKR 9.7 and still be classified as manual-uncovered.
- The public guide is older than the audited 9.7 source; source-only parameters are therefore expected, especially in response, positron, and plotting workflows.
- A task being registered in ASE2SPRKKR means that an input definition exists; it does not override a proven native SPR-KKR bug. Broken tasks are retained here because the goal is an audit, not a list of recommended calculations.
- `BSFEK`, `BSFKK`, and `DMI` are public ASE2SPRKKR aliases/shared definitions, not independent native implementation modules. `TUTORIAL` is one ASE2SPRKKR task with 14 native `TUT-TASK` submodes.
- **Interface status:** Registered task identity; source assessment: Broken native workflow.
- **ASE2SPRKKR tests:** `src/ase2sprkkr/input_parameters/test/test_xrs.py`. These are tests **reported in the 9 Nov 2026 reference audit**, not a new execution at the latest HEAD.
- **Native / numerical verification:** No independent task-specific native numerical benchmark established by the reference audit. Passing Python tests and declared output readers do not certify physics.
- **Structured result reader:** No dedicated reader established here; native logs or specialized output files must not be mistaken for validated structured results.
- **Detailed source/functionality audit:** [XRS](XRS-audit.md); [consolidated source assessment](../SPRKKR-ASE2SPRKKR-common-audit.md).

