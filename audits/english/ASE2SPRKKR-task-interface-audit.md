# ASE2SPRKKR — Task Interface, Parameter and Documentation Audit

**Updated:** 13 November 2026  
**Embedding and executable review:** 18 November 2026 (SPR-KKR 9.7 sources, `ase2sprkkr` `cluster` branch)  
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

## Embedding support — 17 November 2026 review

This revision separately classifies **native embedding support in SPR-KKR 9.7** (`PUB9.7_260420_18h13`), not the availability or numerical validation of the corresponding ASE2SPRKKR/GUI workflow. The evidence is from actual Fortran dispatch branches, host-potential requirements and off-diagonal TAU handling.

- **Yes:** an explicit reachable native embedding calculation path or specialized cluster implementation exists, subject to its input/mode prerequisites.
- **Conditional:** the task or auxiliary routine can be reached through embedded `CLUGEN`, but needs particular modes/data or lacks independent validation of the host+cluster output. **This is not a claim of tested support.**
- **No:** the native code explicitly rejects embedding, uses an incompatible standalone driver without a cluster counterpart, or has a confirmed defect preventing a usable result.
- **Unverified:** a generic `GEN` call can reach related code, but a complete embedded input/output contract and physical interpretation have not been established. This is **not** a positive support claim, nor proof of impossibility.

**Principal source evidence:** `kkrmain.f:300–307,1392–1447` rejects KKR executables for additional cluster sites and dispatches `KKRSCF`, `KKRGEN`, `KKRCHI`, `CLU*`, `KKRSPEC` separately; `cludrive.f:398–413,619–664` activates `EMBEDDED-CLUSTER`, runs `SCF`/`GEN` and delegates `CHI` to `CLU_CHI_DRIVE`; `gen.f:352–365,923–1002` contains embedding-specific JXC/DMI TAU/tensor paths; `cluxcpltensor.f:63–69` sets tensor restrictions; `clu_chi_drive.f:39–49,79–95` rejects embedded CHI and does not implement `GIL`; `initvar.f:160–184,342–354` constrains MECHECK, PHAGEN, OPTABS and SIGMA.

**Execution prerequisites:** being supported by ordinary `KKRGEN` does not imply cluster support. Native `CLUSCF`/`CLUGEN` further require the host potential, cluster geometry and host TAUIJ (`cludrive.f:398–413,619–629,735–830`).

### Executable selection

**Deep recheck, 18 November 2026:** Of the seven previously unverified tasks, `APS`, `COMPTON`, `POSANI`, `POSANIPREP` and `VBPES` are now **No** due to incompatible old-DOS handling, periodic k-space operators, or lack of an embedded final-state calculation. `POSANITAU` is **Conditional** because `posanipot.f` explicitly handles cluster sites, but the physical host+cluster normalization is not validated. `STONER` remains **Unverified**: its `SUSC_GAMMA` postprocessing lacks a demonstrated embedded data contract. Native embedded executables remain enabled in `ase2sprkkr` only for `SCF`, `DOS`, `JXC`, and `DMI`.

Normal calculations use the executable specified by their task definition. True embedding requires the **special** `embscf` (`CLUSCF`) or `embgen` (`CLUGEN`) program, not generic `kkrscf`/`kkrgen`. `KKRSPEC` and `KKRCHI` have no equivalent operational embedding driver. For **Conditional**/**Unverified** tasks, `embgen?` indicates only a possible entry through `CLUGEN`, **not** an enabled task in `ase2sprkkr`. `BSF` is schema-only; the runnable aliases are `BSFEK`/`BSFKK`.

**Recheck, 18 November:** `PLOTRHO`, `NRAES` and `T1` are now **No** because of native defects independent of embedding. Native FREL embedded JXC/DMI writes `*_JJij.dat` and `*_Dij_[xyz].dat`, **not** periodic `XCPLTEN_*`/`DMIVEC_*` outputs; `TAU.BZINT=POINTS` is required for host `TAUIJ` and must not be blindly changed to `CLUSTER`. Embedded SP-SREL JXC is not enabled pending further verification.

## Index

| Task | Operational status | Embedding (SPR-KKR 9.7) | Manual coverage | Executable (normal → embedding) |
|---|---|---|---|---|
| [`AES`](#aes) | Works only in the limited audited workflow; NQ=1, no f shells, with cache/TAU-capacity risks. The numerical solver was not run during the audit. | **No** | Yes. | `kkrgen` → — |
| [`AIPES`](#aipes) | Works in the audited fixed-geometry workflow; it is not numerical integration of ARPES over angles. | **No** | Yes. | `kkrspec` → — |
| [`APS`](#aps) | Works with a compatible old-format DOS; positive `GAMMA` broadening follows a problematic native branch. | **No** | Yes. | `kkrgen` → — |
| [`ARPES`](#arpes) | The source-defined ARPES workflow is meaningful in the audited range. The current interface uses the correct presence-sensitive `TASK.CRYS_VECS`; advanced numerical branches are not certified. | **No** | Yes. | `kkrspec` → — |
| [`BAND`](#band) | Works in the audited workflow; an eigen-solver failure can continue natively as far as `LOG(0)`. This is not `BSF` or `EKREL`. | **No** | Yes. | `kkrspec` → — |
| [`BONDING`](#bonding) | Broken: normal `TAU_DRIVE` prepares `KTAUIJ=0`, while `CLUBONDING` reads scratch units requiring `KTAUIJ>=2`. | **No** | No. | `kkrgen` → — |
| [`BSDOSIJ`](#bsdosij) | Broken: native `J_ITJT` is written with energy indices into pair dimensions and later read in a different order. | **No** | No. | `kkrgen` → — |
| [`BSF`](#bsf) | Schema only, not a task. Works; the E-k default `EMAX=-1 Ry` is a suspicious convenience default and need not mean EF. | **No** | Yes. | — (BSF schema; BSFEK/BSFKK: kkrgen) |
| [`BSFEK`](#bsfek) | Works like `BSF`; the alias only preselects the expected E-k mode through `ENERGY.NE`. | **No** | Yes, as `BSF`; the name `BSFEK` is an ASE2SPRKKR alias. | `kkrgen` → — |
| [`BSFKK`](#bsfkk) | Works like `BSF`; the alias only preselects the expected single-energy k-k mode. | **No** | Yes, as `BSF`; the name `BSFKK` is an ASE2SPRKKR alias. | `kkrgen` → — |
| [`CHI`](#chi) | Works in the audited static workflow; native `NCHIITER` is forcibly reset to 1 before the loop, so the iterative magnetization part does not actually run. | **No** | Yes. | `kkrchi` → — |
| [`CHILANDAU`](#chilandau) | Works serially with limitations; MPI and `RDTAUFIL` reuse are broken, and native finish logic prints partial sums. | **No** | No. | `kkrchi` → — |
| [`CHIXAS`](#chixas) | Works only with physically consistent external `NEF/CHISPN/CHIVV`; no universally safe defaults exist. | **No** | No. | `kkrchi` → — |
| [`CLPES`](#clpes) | Works in the audited serial fixed-geometry workflow; CPA and some polarization modes stop natively, and with `NE=1` the core-state index must be valid. | **No** | No. | `kkrspec` → — |
| [`CLXPS`](#clxps) | Works in the audited ASA/FREL workflow; advanced angular geometries and numerical output were not independently certified. | **Conditional** | Yes. | `kkrgen` → `embgen?` |
| [`COMPTON`](#compton) | Works with limitations; `EMD2D` double-scales nonzero `PNMAX`, and shifted even grids lose their endpoints. | **No** | Yes. | `kkrgen` → — |
| [`DMI`](#dmi) | Works with significant caveats: the individual DMI components are not equivalent in the native source (especially Dz), and frame/multiplicity/radius must be checked. | **Yes** | No as a separate task in the available manual; JXC is documented. | `kkrgen` → `embgen` |
| [`DOS`](#dos) | Works. | **Yes** | Yes. | `kkrgen` → `embgen` |
| [`EKREL`](#ekrel) | Works; CPA is rejected natively, and BSF should be used for disorder. | **No** | Yes. | `kkrgen` → — |
| [`FMAG`](#fmag) | Works with a limitation: a zero magnetic moment can lead to division by zero during normalization. | **Conditional** | Yes. | `kkrgen` → `embgen?` |
| [`FSCAT`](#fscat) | Broken: an erroneous spherical-harmonic `+L` index can overflow, and phase is calculated before zero amplitude is handled. | **No** | No. | `kkrgen` → — |
| [`GILBERT`](#gilbert) | Works for suitable magnetic systems; zero or compensated moments can cause division by zero. | **No** | Yes. | `kkrchi` → — |
| [`JALF`](#jalf) | Works with limitations; helper diagnostics contain bugs and small `NL` is risky, which the definition guards against. | **Conditional** | No. | `kkrgen` → `embgen?` |
| [`JORB`](#jorb) | Works serially; native MPI does not reach the final JORB output and does not reduce the accumulators. | **Conditional** | No. | `kkrgen` → `embgen?` |
| [`JXC`](#jxc) | Works for a simple periodic collinear workflow; the general FREL tensor for symmetry-equivalent neighbors has a native rotation bug, and the cluster SP-SREL scratch flow is unreliable. | **Yes** | Yes. | `kkrgen` → `embgen` |
| [`MECHECK`](#mecheck) | Only partially works: `NMEFORM` is always overwritten to 2 natively, another consistency test is disabled at compile time, and the summary loop is faulty. | **No** | No. | `kkrgen` → — |
| [`MEPLOT`](#meplot) | Works except for native curve overwriting in simultaneous photon+electron scans; the definition rejects that combination. The meaning of `SEFIN` still requires physical verification. | **Conditional** | No. | `kkrgen` → `embgen?` |
| [`NRAES`](#nraes) | Broken: `IAES` is only partially zeroed and convolution accumulates into uninitialized regions. | **No** | Yes. | `kkrgen` → — |
| [`NUCSSCPL`](#nucsscpl) | Broken: native code deallocates arrays before first allocation, clears matrix elements for each type, and forces the mesh to 1. | **No** | No. | `kkrgen` → — |
| [`OPTABS`](#optabs) | Works with major caveats: `TEMPEL<=1` leaves `KBT` uninitialized and `NOMEGA<NEB` can overflow the allocation; the definition rejects these states. | **No** | No. | `kkrchi` → — |
| [`PHAGEN`](#phagen) | Works as a restricted ASA/VWN converter; NT=1, no disorder, and it is not a general phase-shift solver. | **No** | No. | `kkrscf` → — |
| [`PLOT2DPOT`](#plot2dpot) | Works in ASA for potential; the FULLPOT branch enters an experimental PLY writer and exits before the normal plane-grid output. | **Conditional** | No. | `kkrgen` → `embgen?` |
| [`PLOT2DPR`](#plot2dpr) | Partially works: in ASA it actually writes only the potential; the FULLPOT branch exits in the experimental PLY writer. | **Conditional** | No. | `kkrgen` → `embgen?` |
| [`PLOT2DRHO`](#plot2drho) | Practically incomplete: ASA cannot plot density; FULLPOT exits in the test PLY writer before the normal plane-grid writer. | **Conditional** | No. | `kkrgen` → `embgen?` |
| [`PLOTPOT`](#plotpot) | Works for stored ASA and FULLPOT potential data. | **Conditional** | No. | `kkrgen` → `embgen?` |
| [`PLOTPRS`](#plotprs) | Works with limitations: magnetic spin/orbital curves can collide in the same dataset, and panel limits ignore the orbital contribution. | **Conditional** | No. | `kkrgen` → `embgen?` |
| [`PLOTRHO`](#plotrho) | Broken: `RHORSQ` is allocated only in the potential branch, but the standalone rho writer needs it. | **No** | No. | `kkrgen` → — |
| [`PLOTSFN`](#plotsfn) | Works for a genuine FULLPOT potential containing shape-function data. | **Conditional** | No. | `kkrgen` → `embgen?` |
| [`POSANI`](#posani) | Works only with a compatible `POSANIPREP`; the native data path assumes `NQ=NT`, and general `NQ!=NT` is broken. | **No** | Yes. | `kkrgen` → — |
| [`POSANIPREP`](#posaniprep) | Works for a simple 3D ASA/FREL `NQ=NT` case; TAUQQ export is broken for general `NQ!=NT`. The positron-band-bottom energy must be chosen by the user. | **No** | Yes. | `kkrgen` → — |
| [`POSANITAU`](#posanitau) | Works in the audited 3D ASA/FREL workflow; integration-energy bounds need physical validation and the enhancement factor is fixed natively. | **Conditional** | No. | `kkrgen` → `embgen?` |
| [`PSHIFT`](#pshift) | Works, but resonance splitting is unreliable in some cases. | **Conditional** | Yes. | `kkrgen` → `embgen?` |
| [`SCF`](#scf) | Works for `BROYDEN2`, `ANDERSON`, and `TCHEBY` within the allowed scope; `PULAY` is not a valid choice and `TCHEBY` is unavailable for FULLPOT. | **Yes** | Yes. | `kkrscf` → `embscf` |
| [`SIGMA`](#sigma) | Works with limitations; the Fermi-sea contribution is not included automatically, and the modes use fixed file conventions/ImE. | **No** | Yes. | `kkrchi` → — |
| [`SOCPAR`](#socpar) | Works for the main SOC output; exchange-related output has a wrong unit label and one sign-flip test is unreachable in the native code. | **Conditional** | Yes. | `kkrgen` → `embgen?` |
| [`SPLEED`](#spleed) | Works in a conservative fixed-angle workflow; momentum-scan and alternative-polarization branches were not audited. | **No** | Yes. | `kkrspec` → — |
| [`STONER`](#stoner) | Works with limitations; it uses a hard-coded VWN susceptibility kernel, natively replaces small DOS values with unit contributions, and some zero normalizations are unguarded. | **Unverified** | No. | `kkrgen` → `embgen?` |
| [`T1`](#t1) | Broken for numerical use: native `RELT1` treats signed κ values as positive counts of mj channels, so negative channels disappear and basis indexing is wrong. | **No** | No. | `kkrgen` → — |
| [`TDOBS`](#tdobs) | The declared purpose is not demonstrated: no separate time-dependent-observable implementation was found; as a generic DOS path it can run serially. | **No** | No. | `kkrgen` → — |
| [`TORQUE`](#torque) | Works, but requires a rotated magnetic potential/geometry; a normal unrotated magnetic potential may stop natively, and mixed occupancy uses the first type. | **Conditional** | Yes. | `kkrgen` → `embgen?` |
| [`TUTORIAL`](#tutorial) | Mixed: YLM/JLX/JLX-ASYMPT and DOS-FEG/DOS-SS/DOS-CLU/DOS-BULK/GEN are usable; CHR-POT, POT-DECOMP/POT-XC, RWF-L/RWF-E, and CORE-ALL have specific native bugs/limitations. | **No** | No. | `kkrgen` → — |
| [`VBPES`](#vbpes) | Works with limitations; VBUPS does not automatically select a UV photon energy natively, and the calculation depends on a compatible TAU/final-state setup. | **No** | Yes. | `kkrgen` → — |
| [`WFPLOT`](#wfplot) | Works in the ASA/FREL serial workflow; BAND uses only the first energy point and FULLPOT is unsupported. | **Conditional** | Yes. | `kkrgen` → `embgen?` |
| [`XAS`](#xas) | Works with known caveats; the static branch is usable, but TAU compatibility and the effective start of the energy scan must be checked. Native MPI exits before XRAYSPEC. | **Conditional** | Yes. | `kkrgen` → `embgen?` |
| [`XES`](#xes) | Not fully verified: RXES ends in the shared absorption-rate branch and output naming/interpretation is shared with XAS; the physical emission meaning was not confirmed. | **Conditional** | Yes. | `kkrgen` → `embgen?` |
| [`XRS`](#xrs) | Broken for reliable numerical results: native `SIG(NEMAX+50,...)` is zeroed only up to `NEMAX`, but is later summed over the full range. | **No** | Yes (XRS is at least listed as a supported spectroscopy task; the detailed 9.7 interface is broader). | `kkrgen` → — |

## Tasks

### AES

- **What it does:** Calculates fully relativistic Auger electron spectra from existing electronic TAU data and matrix elements.
- **Status:** Works only in the limited audited workflow; NQ=1, no f shells, with cache/TAU-capacity risks. The numerical solver was not run during the audit.
- **Embedding (native SPR-KKR 9.7):** **No** — The native AES path is restricted to `NQ=1`, whereas `CLUDRIVE` extends host sites with cluster sites (`gen.f:1402–1413`, `cludrive.f:412–413`).
- **Executable (normal / embedding):** `kkrgen` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **No** — AIPES is handled by `KKRSPEC`, which the cluster driver does not call (`kkrmain.f:1392–1447`).
- **Executable (normal / embedding):** `kkrspec` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **No** — `initvar.f:281–287` expects a pre-existing old-format DOS and disables the energy loop, but embedded `CLUDRIVE` forces `RUNELOOP=.TRUE.` and reopens/writes `{DATSET}.dos` (`cludrive.f:631–655`), the very file consumed by `NRAPS` (`nraps.f:829–863`). The embedded driver breaks this input contract.
- **Executable (normal / embedding):** `kkrgen` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **No** — Full ARPES is handled by `KKRSPEC`/`SPECMAIN`, not by the cluster driver (`kkrmain.f:1431–1447`). An input selector is not embedding support.
- **Executable (normal / embedding):** `kkrspec` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **No** — The band-spectrum implementation is in `KKRSPEC`, not the `CLU*` driver (`kkrmain.f:1431–1447`).
- **Executable (normal / embedding):** `kkrspec` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **No** — Although `GEN` calls `CLUBONDING` when `IBZINT=0`, the previous source audit found an invalid `KTAUIJ`/scratch-file contract (`gen.f:1012–1025`).
- **Executable (normal / embedding):** `kkrgen` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **No** — A `CALC_BSDOSIJ` selector exists, but the previous native audit documents inconsistent energy/pair indexing. Cluster output is not reliable.
- **Executable (normal / embedding):** `kkrgen` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **No** — `GEN` reaches `BLOCHSF`, but no implementation of the k-resolved Bloch spectrum for an embedded local cluster is evidenced (`gen.f:1499–1506`).
- **Executable (normal / embedding):** not directly runnable; `BSFEK`/`BSFKK` use `kkrgen`; embedding unsupported
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
- **Embedding (native SPR-KKR 9.7):** **No** — Alias of BSF (E–k): the `BLOCHSF` path has not been established for `EMBEDDED-CLUSTER` (`gen.f:1499–1506`).
- **Executable (normal / embedding):** `kkrgen` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **No** — Alias of BSF (k–k): the `BLOCHSF` path has not been established for `EMBEDDED-CLUSTER` (`gen.f:1499–1506`).
- **Executable (normal / embedding):** `kkrgen` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **No** — `CLUDRIVE` calls `CLU_CHI_DRIVE`, but it rejects embedded `KKRMODE` as incorrect (`cludrive.f:656–663`, `clu_chi_drive.f:39–49`).
- **Executable (normal / embedding):** `kkrchi` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **No** — `KKRCHI/RSP` rejects embedded `KKRMODE`, and `CLU_CHI_DRIVE` does not implement CHILANDAU (`kkrmain.f:1425–1433`, `rsp.f:103–114`).
- **Executable (normal / embedding):** `kkrchi` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **No** — `CLU_CHI_DRIVE` implements no CHIXAS calculation; `RSP` rejects embedded `KKRMODE` (`clu_chi_drive.f:77–96`, `rsp.f:103–114`).
- **Executable (normal / embedding):** `kkrchi` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **No** — The photoemission calculation uses `KKRSPEC`, not `CLUDRIVE → SPECMAIN` (`kkrmain.f:1392–1447`).
- **Executable (normal / embedding):** `kkrspec` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **Conditional** — The `CLUGEN → GEN → CLXPS` call path exists and is limited to ASA (`gen.f:1434–1450`); embedded matrix elements/results remain unvalidated.
- **Executable (normal / embedding):** `kkrgen` → `embgen?` (possible route only; disabled on `cluster` branch)
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
- **Embedding (native SPR-KKR 9.7):** **No** — The momentum-resolved code constructs its own periodic k-space matrix using `STRSET`/`SETKKR` and inverts it (`compton.f:917–935`), instead of using the embedded cluster scattering-path operator; it also reads an external TAU directory (`compton.f:633–654`). No embedded equivalent of this calculation is implemented.
- **Executable (normal / embedding):** `kkrgen` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **Yes** — FREL embedding calls `CLUXCPLTENSOR` and writes both J and D products (`*_JJij.dat`, `*_Dij_[xyz].dat`), even in DMI-only mode (`gen.f:967–1000`; `cluxcpltensor.f:278,432–440`); this is not periodic `DMIVECTOR`.
- **Executable (normal / embedding):** `kkrgen` → `embgen`
- **Input files:** standard input + `POTFIL`.
- **Output files:** `{DATASET_ADSI}_DMIVEC_Dij.dat`; optional component plots `{DATASET_ADSI}_DMIVEC_Dij_[xyz]_*.agr`. **Embedded FREL:** `CLUXCPLTENSOR` writes `{DATASET_ADSI}_JJij.dat` and `{DATASET_ADSI}_Dij_[xyz].dat` even for DMI-only runs, plus corresponding AGR plots (`cluxcpltensor.f`).
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
- **Embedding (native SPR-KKR 9.7):** **Yes** — `CLUGEN` opens the DOS file, maps cluster atoms, then calls `GEN` (`cludrive.f:631–658`); DOS is accumulated in the GEN energy loop.
- **Executable (normal / embedding):** `kkrgen` → `embgen`
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
- **Embedding (native SPR-KKR 9.7):** **No** — `GEN` calls `EKREL`, but no meaningful embedded-cluster k-resolved eigenenergy implementation is established (`gen.f:1494–1497`).
- **Executable (normal / embedding):** `kkrgen` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **Conditional** — `CALC_FMAG` is selected in `initvar.f:534–550` and processed in the generic embedded `GEN` path, but cluster/host output allocation is not validated.
- **Executable (normal / embedding):** `kkrgen` → `embgen?` (possible route only; disabled on `cluster` branch)
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
- **Embedding (native SPR-KKR 9.7):** **No** — A generic `FSCAT` call exists (`gen.f:1413–1420`), but the earlier native audit found task defects; no working embedding variant is established.
- **Executable (normal / embedding):** `kkrgen` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **No** — `CLU_CHI_DRIVE` rejects embedded KKRMODE and the `CALL GIL` statement is commented out; `GEN` has no replacement calculation (`clu_chi_drive.f:39–49,79–95`).
- **Executable (normal / embedding):** `kkrchi` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **Conditional** — `CALC_JALF` reaches `CURDNS` via `GEN` (`gen.f:1569–1579`), but validity of embedded cluster current density is unverified.
- **Executable (normal / embedding):** `kkrgen` → `embgen?` (possible route only; disabled on `cluster` branch)
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
- **Embedding (native SPR-KKR 9.7):** **Conditional** — `CALC_JORB` reaches `ORBCURDNS` via `GEN` (`gen.f:1579–1588`), but embedded cluster output has not been validated.
- **Executable (normal / embedding):** `kkrgen` → `embgen?` (possible route only; disabled on `cluster` branch)
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
- **Embedding (native SPR-KKR 9.7):** **Yes** — The explicitly supported route is FREL `CLUXCPLTENSOR` (`gen.f:967–1000`, `cluxcpltensor.f:63–69`), producing `*_JJij.dat` and `*_Dij_[xyz].dat`. SP-SREL `CLUXCPLJIJ` depends on `IBZINT` and is not treated here as a validated embedded workflow.
- **Executable (normal / embedding):** `kkrgen` → `embgen`
- **Input files:** standard input + `POTFIL`.
- **Output files:** Periodic: `{DATASET_ADSI}_J_ij.dat` / `{DATASET_ADSI}_Jij_*.agr` (SP-SREL) or `{DATASET_ADSI}_XCPLTEN_Jij.dat` and `XCPLTEN_*` plots (FREL); periodic DMI produces `DMIVEC_*`. **Embedded FREL:** `{DATASET_ADSI}_JJij.dat`, `{DATASET_ADSI}_Jij_*.agr`, `{DATASET_ADSI}_Dij_x.dat`, `_Dij_y.dat`, `_Dij_z.dat` and `Dij_[xyz]_*.agr` (`cluxcpltensor.f:278,432–440`).
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
- **Embedding (native SPR-KKR 9.7):** **No** — `INITVAR` explicitly requires `PROGNAME=KKRGEN` for MECHECK test 11, so CLUGEN is rejected (`initvar.f:160–167`).
- **Executable (normal / embedding):** `kkrgen` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **Conditional** — The generic embedded GEN path can reach `ME_PLOT` (`gen.f:1481–1485`); embedded matrix elements are unvalidated.
- **Executable (normal / embedding):** `kkrgen` → `embgen?` (possible route only; disabled on `cluster` branch)
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
- **Embedding (native SPR-KKR 9.7):** **No** — The native NRAES convolution accumulates into incompletely initialized `IAES`; the defect persists in generic `GEN` and therefore in `CLUGEN`.
- **Executable (normal / embedding):** `kkrgen` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **No** — The previous native task audit found defects in `NUCSSCPL`; no corrected cluster-specific computation exists in `GEN`.
- **Executable (normal / embedding):** `kkrgen` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **No** — `INITVAR` requires `KKRCHI` for OPTABS/OPTICS (`initvar.f:342–345`), and `RSP` rejects embedded KKRMODE.
- **Executable (normal / embedding):** `kkrchi` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **No** — The PHAGEN selector explicitly requires `KKRSCF`, not `CLUSCF` (`initvar.f:179–184`).
- **Executable (normal / embedding):** `kkrscf` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **Conditional** — The selector activates `PLOT2DPR`; `GEN` calls ASA/FULLPOT plotting routines, but host/cluster radial-data projection is unverified (`gen.f:1457–1476`).
- **Executable (normal / embedding):** `kkrgen` → `embgen?` (possible route only; disabled on `cluster` branch)
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
- **Embedding (native SPR-KKR 9.7):** **Conditional** — `CLUGEN → GEN` can invoke `PLOT2DPR_ASA`/`FPPLOT2DPR` (`gen.f:1457–1476`); correctness for embedded data is unverified.
- **Executable (normal / embedding):** `kkrgen` → `embgen?` (possible route only; disabled on `cluster` branch)
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
- **Embedding (native SPR-KKR 9.7):** **Conditional** — Same generic `PLOT2DPR` routing (`gen.f:1457–1476`); this plots stored radial data, not a standalone embedding calculation.
- **Executable (normal / embedding):** `kkrgen` → `embgen?` (possible route only; disabled on `cluster` branch)
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
- **Embedding (native SPR-KKR 9.7):** **Conditional** — The selector invokes the generic PLOTPRS plotting path; host/cluster selection is unverified (`initvar.f:189–196`, `gen.f:1453–1461`).
- **Executable (normal / embedding):** `kkrgen` → `embgen?` (possible route only; disabled on `cluster` branch)
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
- **Embedding (native SPR-KKR 9.7):** **Conditional** — `CLUGEN → GEN → FPPLOTPRS` is reachable with the matching selector (`gen.f:1452–1463`), but host/cluster data handling is unverified.
- **Executable (normal / embedding):** `kkrgen` → `embgen?` (possible route only; disabled on `cluster` branch)
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
- **Embedding (native SPR-KKR 9.7):** **No** — Standalone native `PLOTRHO` needs `RHORSQ`, which is not allocated on that branch; invoking it from `CLUGEN` cannot repair this writer bug. See the PLOTPRS source audit.
- **Executable (normal / embedding):** `kkrgen` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **Conditional** — The PLOTPRS selector is reachable, but requires stored FULLPOT shape functions; embedded cluster data handling is unverified.
- **Executable (normal / embedding):** `kkrgen` → `embgen?` (possible route only; disabled on `cluster` branch)
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
- **Embedding (native SPR-KKR 9.7):** **No** — Shares the periodic momentum-space `COMPTON` driver (`gen.f:1572–1577`, `compton.f:917–935`) and requires periodic positron pair data `{DATSET}_pos_dat.pan` (`compton.f:424–426`). Calling this from `CLUGEN` does not replace periodic final-state integration with embedded scattering.
- **Executable (normal / embedding):** `kkrgen` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **No** — `POSANIPREPARE` explicitly creates a periodic k-mesh and calls `KLOOPSIJ` for `TAUQQ` (`posaniprep.f:116–145`), not embedded-cluster `TAUIJ`. Its output also loops over `NT` types instead of `NQ` sites (`posaniprep.f:268–272`), which breaks general host+cluster mapping.
- **Executable (normal / embedding):** `kkrgen` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **Conditional** — There is an actual cluster-aware native path: `POSANIPOT` handles `NTCLU` and shifts cluster-atom potentials (`posanipot.f:95–110,267–275`), while `GEN` computes cluster TAU before `CHRDNS` and `POSANILAMBDA` (`gen.f:783–807,882–897,1509–1564`). `POSANILAMBDA` integrates over active types `ITBOT:ITTOP` (`posanilambda.f:59–105`); physical normalization across host+cluster is unverified. Conditional native path only, disabled in `ase2sprkkr`.
- **Executable (normal / embedding):** `kkrgen` → `embgen?` (possible route only; disabled on `cluster` branch)
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
- **Embedding (native SPR-KKR 9.7):** **Conditional** — `GEN` calls `PSHIFT` in ASA (`gen.f:1427–1435`), but cluster phase-shift output has not been validated.
- **Executable (normal / embedding):** `kkrgen` → `embgen?` (possible route only; disabled on `cluster` branch)
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
- **Embedding (native SPR-KKR 9.7):** **Yes** — `CLUSCF` prepares host/cluster data and directly calls `SCF`; requires host Madelung potential and charges (`cludrive.f:619–629`).
- **Executable (normal / embedding):** `kkrscf` → `embscf`
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
- **Embedding (native SPR-KKR 9.7):** **No** — `INITVAR` explicitly requires `KKRCHI` (`initvar.f:350–354`), while `RSP` rejects `EMBEDDED-CLUSTER`.
- **Executable (normal / embedding):** `kkrchi` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **Conditional** — `GEN` calls SOCPAR in ASA (`gen.f:1421–1430`); site mapping and embedded output are unverified.
- **Executable (normal / embedding):** `kkrgen` → `embgen?` (possible route only; disabled on `cluster` branch)
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
- **Embedding (native SPR-KKR 9.7):** **No** — SPLEED runs via `KKRSPEC`/`SPECMAIN`, not through the `CLU*` driver (`kkrmain.f:1431–1447`).
- **Executable (normal / embedding):** `kkrspec` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **Unverified** — `GEN` contains STONER (`gen.f:1589–1611`), but no independently validated embedded response implementation.
- **Executable (normal / embedding):** `kkrgen` → `embgen?` (possible route only; disabled on `cluster` branch)
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
- **Embedding (native SPR-KKR 9.7):** **No** — Native `RELT1` incorrectly uses signed κ values as positive mj channel counts, making the result unreliable independently of embedding.
- **Executable (normal / embedding):** `kkrgen` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **No** — No dedicated TDOBS computation exists in `CLU_CHI_DRIVE` or the GEN task dispatch; simply calling GEN does not calculate TDOBS.
- **Executable (normal / embedding):** `kkrgen` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **Conditional** — `GEN` calls TORQUE when selected (`gen.f:1004–1009`); combined JXC+TORQUE is forbidden (`initvar.f:542–551`), and cluster numerics are not validated.
- **Executable (normal / embedding):** `kkrgen` → `embgen?` (possible route only; disabled on `cluster` branch)
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
- **Embedding (native SPR-KKR 9.7):** **No** — The special TUTORIAL driver is called only in the `KKRGEN` branch, whereas `CLU*` calls `CLUDRIVE → GEN` (`kkrmain.f:1411–1433`).
- **Executable (normal / embedding):** `kkrgen` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **No** — The photoemission workflow reads an existing electron TAU directory and constructs periodic final states (`initvar.f:221–255`, `gen.f:1445–1450`, per-task source audit of `vbpes.f:282–291`). No embedded final-state construction or mapping of photoemission matrix elements to the cluster is established; `CLUGEN` does not implement this variant.
- **Executable (normal / embedding):** `kkrgen` → no supported embedding executable
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
- **Embedding (native SPR-KKR 9.7):** **Conditional** — `GEN` can reach `WFPLOT` (`gen.f:1456–1460`) if wavefunctions are present; mapping to host/cluster is not validated.
- **Executable (normal / embedding):** `kkrgen` → `embgen?` (possible route only; disabled on `cluster` branch)
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
- **Embedding (native SPR-KKR 9.7):** **Conditional** — `GEN → XRAYSPEC` is reachable for XAS (`gen.f:1393–1402`); physically valid embedded cluster spectra have not been independently verified.
- **Executable (normal / embedding):** `kkrgen` → `embgen?` (possible route only; disabled on `cluster` branch)
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
- **Embedding (native SPR-KKR 9.7):** **Conditional** — `GEN → XRAYSPEC` is reachable for XES (`gen.f:1393–1402`); assignment of core states to embedded sites is unverified.
- **Executable (normal / embedding):** `kkrgen` → `embgen?` (possible route only; disabled on `cluster` branch)
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
- **Embedding (native SPR-KKR 9.7):** **No** — Although `GEN` can call `XRAYSPEC`, the previous audit found a severe native XRS buffer bug; working embedded results cannot be claimed.
- **Executable (normal / embedding):** `kkrgen` → no supported embedding executable
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

