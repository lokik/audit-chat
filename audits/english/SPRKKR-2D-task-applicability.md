# Native 2D task applicability — SPR-KKR 9.7 source audit

**Date:** 19 November 2026. **Baseline:** `PUB9.7_260420_18h13` and the existing [task-interface audit](ASE2SPRKKR-task-interface-audit.md) / [functional audit](SPRKKR-ASE2SPRKKR-common-audit.md). This is an audit of reachable Fortran paths, **not** a newly run suite of native numerical calculations.

Native **2D** means `SYSTEM_DIMENSION=2D` (surface/layers/interface), not a vacuum-containing 3D supercell and not a two-dimensional output plot. **Yes** means a meaningful 2D source path exists, subject to explicit limitations below. **No** means no usable normal native-2D workflow. Separate **2D-specific blockers** from **general defects also affecting 3D**. Inventory: **56 headings, 41 Yes / 15 No**, including the schema-only `BSF` heading shared by `BSFEK` and `BSFKK`.

## 3D-capable but not native-2D-capable

| Task | 3D | Native 2D | Evidence |
|---|---|---|---|
| `POSANIPREP` | Yes, within its valid input scope | **No** | `gen.f:435–455` calls `POSANIPOT`; `posanipot.f:88–90` explicitly stops when `SYSTEM_DIMENSION != 3D`. |
| `POSANITAU` | Yes, within its valid input scope | **No** | Same mandatory `POSANIPOT` call and explicit dimension stop. |
| `CHILANDAU` | Yes, limited serial 3D path | **No** | `susc_landau.f:221–226` calls the 3D `TAU_STD` directly instead of `TAU_TB`. |
| `POSANI` | Yes with prepared .pan | **No for the normal end-to-end workflow** | The task bypasses `POSANIPOT` itself, but depends on .pan created by 2D-incompatible `POSANIPREP`. Externally supplied .pan may permit postprocessing; do not interpret that as full native 2D support. |

The other eleven **No** entries reflect general implementation defects or absent declared functionality, **not** proven 3D-versus-2D incompatibilities.

## Shared mode and parameter constraints

- `init_mod_calcmode.f:609–634`: 2D requires `TB-KKR` or `LAYER-KKR`; default is `TB-KKR`; `STANDARD-KKR` is rejected.
- `rsp.f:98–113`, `gil.f:172–249`, `opt_abs.f:637–798`, `sig.f:713–844`: relevant response tasks have **TB-KKR paths**, but reject **LAYER-KKR**.
- `tauij_drive.f:44–93`: the 2D off-diagonal implementation is `TAUIJ_TB`, not `TAUIJ_LKKR`; `JXC/DMI` require `IBZINT=2`, magnetic FREL setup, `KMROT=0`.
- `tau_drive.f:47–54`: `THERMAL_VIBRA_FLUCT` requires `STANDARD-KKR` or `REAL-SPACE`, conflicting with ordinary native-2D modes.
- `scf_drive.f:61–68`: `SCF.SCL_ALAT` is explicitly **3D-only**, though SCF itself works in 2D.
- `spec_rslab.f:134–195,262–300`: 2D `KKRSPEC` external structure file (`STRVER=0`) covers `VIR/LIV/VIV/LIR`; automatic construction (`STRVER=1`) accepts **only VIR/LIV**.
- `blochsf.f:274–282,431–437` and `ekrel.f:249–260`: dedicated handling for native-2D reciprocal geometry.
- `fpplot2dpr.f:287–404` has a 2D geometry branch; `plot2dpr_asa.f:123–130` uses third-vector periodic replicas. Plotting in a 2D plane is **not** evidence of a native-2D structure.
- `XAS` normalization uses `VOLUC*ALAT**3`; check physical interpretation of volume-normalized output for a 2D system.

## Full 56-heading classification

| Task | Native 2D | Restriction / blocker |
|---|---|---|
| `AES` | **Yes** | ASA; NQ=1; no FULLPOT. |
| `AIPES` | **Yes** | KKRSPEC; 2D STRVER restrictions. |
| `APS` | **Yes** | Compatible legacy DOS; positive GAMMA is problematic. |
| `ARPES` | **Yes** | KKRSPEC; 2D STRVER restrictions. |
| `BAND` | **Yes** | KKRSPEC; 2D STRVER restrictions. |
| `BONDING` | **No** | Generally broken TAUIJ setup (also in 3D). |
| `BSDOSIJ` | **No** | Generally incorrect indexing (also in 3D). |
| `BSF` | **Yes** | Shared BSFEK/BSFKK schema; BLOCHSF has a 2D path. |
| `BSFEK` | **Yes** | 2D BLOCHSF path; energy scan. |
| `BSFKK` | **Yes** | 2D BLOCHSF path; k scan. |
| `CHI` | **Yes** | TB-KKR only, not LAYER-KKR; no FULLPOT. |
| `CHILANDAU` | **No** | 2D-specific: SUSC_LANDAU calls TAU_STD. |
| `CHIXAS` | **Yes** | TB-KKR; compatible external NEF/CHISPN/CHIVV; no FULLPOT. |
| `CLPES` | **Yes** | KKRSPEC; no CPA; STRVER restrictions. |
| `CLXPS` | **Yes** | ASA/FREL; valid core states. |
| `COMPTON` | **Yes** | Uses TAU; SQQ_RESOLVED fails with MD2D/EMD2D; 2D output is not 2D geometry. |
| `DMI` | **Yes** | TB-KKR; IBZINT=2; FREL; KMROT=0; magnetic; not LAYER-KKR. |
| `DOS` | **Yes** | General 2D path through TAU_DRIVE. |
| `EKREL` | **Yes** | Explicit 2D geometry; no CPA. |
| `FMAG` | **Yes** | Nonzero moments for normalization; ordinary 2D GEN path. |
| `FSCAT` | **No** | General spherical-harmonic indexing defect. |
| `GILBERT` | **Yes** | TB-KKR; KMROT=0; IBZINT=2 or 6; not LAYER-KKR; no FULLPOT. |
| `JALF` | **Yes** | Ordinary 2D GEN; known auxiliary diagnostics defects. |
| `JORB` | **Yes** | Ordinary 2D GEN; serial only; MPI completion defective. |
| `JXC` | **Yes** | TB-KKR; IBZINT=2; FREL; KMROT=0; magnetic; not LAYER-KKR. |
| `MECHECK` | **Yes** | Local matrix-element checks; limited native test coverage. |
| `MEPLOT` | **Yes** | Do not combine photon and electron scans. |
| `NRAES` | **No** | Generally defective accumulator initialization. |
| `NUCSSCPL` | **No** | General allocation and matrix-element defects. |
| `OPTABS` | **Yes** | TB-KKR and ASA; TEMPEL>1; NOMEGA>=NEB; not LAYER-KKR. |
| `PHAGEN` | **Yes** | Potential export only; converged ASA/VWN; NT=1. |
| `PLOT2DPOT` | **Yes** | ASA plot may introduce 3D periodic replicas; FULLPOT PLY is experimental. |
| `PLOT2DPR` | **Yes** | ASA plots potential only; FULLPOT has 2D geometry but no complete standard grid. |
| `PLOT2DRHO` | **No** | Generally incomplete: ASA lacks density; FULLPOT lacks standard output. |
| `PLOTPOT` | **Yes** | Plots saved potential; no new self-consistency. |
| `PLOTPRS` | **Yes** | Plots saved radial data; known spin/orbital curve collisions. |
| `PLOTRHO` | **No** | General uninitialized/unallocated RHORSQ defect. |
| `PLOTSFN` | **Yes** | Only a potential containing FULLPOT shape functions. |
| `POSANI` | **No** | No working .pan producer in standard 2D workflow; externally supplied data may enable postprocessing. |
| `POSANIPREP` | **No** | 3D-specific: required POSANIPOT explicitly rejects 2D. |
| `POSANITAU` | **No** | 3D-specific: required POSANIPOT explicitly rejects 2D. |
| `PSHIFT` | **Yes** | ASA; derived resonance estimates have known limitations. |
| `SCF` | **Yes** | 2D SCF works; SCF.SCL_ALAT is 3D-only. |
| `SIGMA` | **Yes** | TB-KKR Kubo; not LAYER-KKR; SIGMA_LAYER unavailable in public build. |
| `SOCPAR` | **Yes** | ASA; exchange part has a unit-label defect. |
| `SPLEED` | **Yes** | KKRSPEC; 2D STRVER restrictions. |
| `STONER` | **Yes** | 2D GEN; fixed VWN kernel and normalization risks. |
| `T1` | **No** | Generally broken: nonrel STOP; relativistic branch numerically defective. |
| `TDOBS` | **No** | Generally no distinct time-dependent observable solver. |
| `TORQUE` | **Yes** | Requires properly rotated magnetic potential; do not combine with JXC. |
| `TUTORIAL` | **Yes** | Depends on TUT-TASK; math demos geometry-independent; DOS submodes follow GEN. |
| `VBPES` | **Yes** | FREL and compatible initial/final TAU; not equivalent to KKRSPEC ARPES. |
| `WFPLOT` | **Yes** | ASA/FREL; FULLPOT unsupported. |
| `XAS` | **Yes** | FREL, serial; compatible TAU and energy mesh; inspect volume normalization. |
| `XES` | **No** | Generally lacks a separately meaningful RXES emission path; shares absorption path. |
| `XRS` | **No** | General SIG initialization defect; result is not reliable. |

## Proposed ASE2SPRKKR metadata

Represent system kind, **not just periodic dimension**, e.g. `supported_systems={"3d", "2d", "cluster", "embedded"}`. `cluster` means a free finite cluster **without** host potential; `embedded` means an embedded cluster **with** host potential. Both might be described as 0D, so `0d` alone obscures the important distinction. For embedded clusters, the host's own dimension should be tracked separately.

**Always explicitly list `3d`** when supported. Omission means not supported; do not assume an implicit 3D fallback. Task and parameter validation are separate concerns: `KKRMODE`, `IBZINT`, `STRVER`, `SCF.SCL_ALAT`, etc. must be checked against the actual structure and inputs. Distinguish general native software defects from geometry support. Apply the separate [embedding audit](ASE2SPRKKR-task-interface-audit.md) before populating `embedded`.

```python
class SystemKind(StrEnum):
    PERIODIC_3D = "3d"
    LAYERED_2D = "2d"
    FREE_CLUSTER = "cluster"
    EMBEDDED_CLUSTER = "embedded"

# Example metadata, not an implemented API change:
# SCF: supports={"3d", "2d", "embedded"}
# POSANIPREP: supports={"3d"}
# CHILANDAU: supports={"3d"}
# A separate validator checks task/parameter combinations.
```


The free non-embedded cluster dimension is assessed separately in the [0D free-cluster audit](SPRKKR-0D-free-cluster-task-applicability.md); `TAU.CLUSTER` alone is not equivalent to the molecular `TAU.MOL` route.
