# Native free 0D cluster applicability — SPR-KKR 9.7 task audit

**Date:** 19 November 2026. **Baseline:** `PUB9.7_260420_18h13`, previously audited Fortran paths in the [task-interface audit](ASE2SPRKKR-task-interface-audit.md), [functional audit](SPRKKR-ASE2SPRKKR-common-audit.md), and [individual task reports](tasks/); cross-checked against the [2D audit](SPRKKR-2D-task-applicability.md) and [SPR-KKR User's Guide](https://www.scribd.com/document/873750459/kkr-manual) (§ 2.2.3). **No new numerical validation is claimed.**

## Precisely what “cluster” means here

**Cluster means an isolated finite system without a host potential**, not a finite cutout of a periodic crystal or an impurity embedded in a host. The native distinction is **`TAU.MOL`**, molecular real-space KKR, with the relevant real-space `TAU.BZINT=CLUSTER` / `IBZINT=0` path. `NQ` is the count of molecular atomic sites, and the formal lattice basis does not describe a physical periodic host.

**`TAU.CLUSTER` by itself** is a finite real-space approximation to an otherwise periodic structure: its central site has the intended τ, whereas the other sites cannot be treated as independent bulk-equivalent atoms. In **`TAU.MOL`** all molecular sites receive the appropriate τ. Neither equals **embedding**, which needs a host potential and uses `embscf`/`embgen`. A free cluster uses the normal executable (`kkrscf`, `kkrgen`, etc.), **not `emb*`**.

**Yes** = meaningful native 0D path, also including genuinely local potential-only postprocessing, subject to the stated restrictions. **No** = **do not advertise support for an arbitrary free 0D cluster**: periodic-only operator, general native defect, or no complete supported molecular input/output path established. In the last category, **No is not proof of mathematical impossibility**. Conservatively treat Yes as task applicability, not numerical certification, and distinguish local postprocessing from a physical cluster solver.

Inventory: **56 headings, 19 Yes / 37 No**. `BSF` is a shared schema, not a runnable task; `BSFEK` and `BSFKK` are runnable aliases. 0D and embedded are distinct kinds of support.

## Full task matrix

| Task | Free 0D cluster | Rationale / limitation |
|---|---|---|
| `AES` | **No** | Requires NQ=1; not a general multi-site free-cluster workflow. |
| `AIPES` | **No** | KKRSPEC uses crystal/photoemission geometry rather than molecular TAU. |
| `APS` | **No** | Depends on legacy DOS format; no complete molecular workflow established. |
| `ARPES` | **No** | Angular KKRSPEC geometry and Bloch states, not an isolated molecule. |
| `BAND` | **No** | Periodic E(k) dispersion requires reciprocal momentum. |
| `BONDING` | **No** | Designed for molecular clusters, but TAU_DRIVE omits required scratch units 18/19. |
| `BSDOSIJ` | **No** | General native pair/energy indexing defect prevents reliable result. |
| `BSF` | **No** | Schema only; reciprocal-space spectral function is not a 0D task. |
| `BSFEK` | **No** | E–k Bloch spectral function requires periodicity. |
| `BSFKK` | **No** | k–k Bloch spectral function requires periodicity. |
| `CHI` | **No** | Independent KKRCHI path; no established molecular 0D susceptibility workflow. |
| `CHILANDAU` | **No** | SUSC_LANDAU calls TAU_STD directly, not TAU_REAL_SPACE. |
| `CHIXAS` | **No** | External NEF/CHISPN/CHIVV and KKRCHI; molecular contract not established. |
| `CLPES` | **No** | KKRSPEC uses periodic/photoemission geometry. |
| `CLXPS` | **Yes** | Local GEN core matrix elements; ASA/FREL and valid occupied core states only. |
| `COMPTON` | **No** | Builds its own periodic reciprocal-space matrix with STRSET/SETKKR. |
| `DMI` | **No** | Normal FREL JXC/DMI requires IBZINT=2/periodic TAUIJ; embedded route is not a free cluster. |
| `DOS` | **Yes** | GEN using molecular real-space TAU; all molecular sites are represented. |
| `EKREL` | **No** | k-dependent eigenenergies have no isolated-cluster workflow. |
| `FMAG` | **Yes** | Local densities/moments from real-space TAU; ASA and nonzero normalization moments. |
| `FSCAT` | **No** | General native spherical-harmonic indexing defect. |
| `GILBERT` | **No** | GIL requires IBZINT=2/6, not molecular real-space mode. |
| `JALF` | **Yes** | Local relativistic current-density integration in GEN; serial only, sufficient NL. |
| `JORB` | **Yes** | Local orbital-current density in GEN; serial FREL only. |
| `JXC` | **No** | Normal route requires IBZINT=2 and periodic TAUIJ; CLUXCPLTENSOR is embedded-only. |
| `MECHECK` | **Yes** | Local matrix-element test; some diagnostics are defective, not a full certification. |
| `MEPLOT` | **Yes** | Local atomic matrix elements, no essential Bloch geometry; avoid combined scans. |
| `NRAES` | **No** | Generally defective AES accumulator initialization. |
| `NUCSSCPL` | **No** | General allocation and matrix-element overwrite bugs. |
| `OPTABS` | **No** | OPT_ABS uses its TB-KKR response path, not a molecular 0D solver. |
| `PHAGEN` | **Yes** | Existing-potential conversion, not an 0D solver; converged ASA/VWN and NT=1 only. |
| `PLOT2DPOT` | **No** | ASA plotting introduces periodic replicas; not reliable molecular geometry. |
| `PLOT2DPR` | **No** | ASA gives potential with periodic replicas; FULLPOT remains experimental. |
| `PLOT2DRHO` | **No** | Generally incomplete density output. |
| `PLOTPOT` | **Yes** | POTFIL radial-potential postprocessing; no reciprocal periodicity needed. |
| `PLOTPRS` | **Yes** | Local POTFIL radial data; beware magnetic curve collisions. |
| `PLOTRHO` | **No** | General unprepared RHORSQ defect. |
| `PLOTSFN` | **Yes** | Plots existing FULLPOT shape functions only; does not solve a cluster. |
| `POSANI` | **No** | Needs .pan from the periodic POSANIPREP workflow. |
| `POSANIPREP` | **No** | Mandatory POSANIPOT and periodic k-mesh; no molecular real-space route. |
| `POSANITAU` | **No** | Mandatory POSANIPOT and native 3D periodic assumptions. |
| `PSHIFT` | **Yes** | Atomic-local phase shift from POTFIL; some resonance estimates are limited. |
| `SCF` | **Yes** | Molecular TAU.MOL handles all NQ sites without host potential; normal kkrscf. |
| `SIGMA` | **No** | Kubo/BZ conductivity in STANDARD/TB-KKR, not an isolated finite cluster. |
| `SOCPAR` | **Yes** | Local spin–orbit parameters from atomic radial data; exchange unit label defect. |
| `SPLEED` | **No** | KKRSPEC surface diffraction requires an extended structure. |
| `STONER` | **Yes** | Normal GEN/SUSC_GAMMA has all MOL types active; embedded host-scratch defect does not apply; FREL/serial. |
| `T1` | **No** | General native relativistic κ-channel defect. |
| `TDOBS` | **No** | No separate functional time-dependent observable implementation established. |
| `TORQUE` | **Yes** | GEN local moments/sites; requires appropriately rotated magnetic potential. |
| `TUTORIAL` | **Yes** | Only geometry-independent math tests and molecular-compatible examples; not all TUT-TASK modes. |
| `VBPES` | **No** | Requires periodic initial/final-state TAU and photoemission geometry. |
| `WFPLOT` | **Yes** | Local core/band radial functions from potential; ASA/FREL, serial, no FULLPOT. |
| `XAS` | **Yes** | Local spectroscopy with real-space TAU; FREL, serial, selected core site, consistent energy mesh. |
| `XES` | **No** | No distinct validated RXES emission branch; shares absorption output. |
| `XRS` | **No** | General native SIG array initialization defect. |

## Evidence and limitations

1. **MOL versus CLUSTER:** SPR-KKR manual § 2.2.3 describes molecular `MOL` as using all `NQ` sites and ordinary `CLUSTER` as physically meaningful primarily at the central site. The [BONDING review](tasks/BONDING-audit.md) records `init_mod_kmesh.f:717` (real-space branch), `tau_real_space.f` (molecular site range), and the broken `KTAUIJ` scratch contract.
2. **Executable dispatch:** [Embedding audit](ASE2SPRKKR-task-interface-audit.md), `kkrmain.f:300–307,1392–1447` and `cludrive.f:398–413,619–664`: `CLUSCF/CLUGEN` use host-dependent drivers; `TAU.MOL` does not.
3. **Periodic-only routes:** normal `JXC/DMI` require `IBZINT=2` (`gen.f:355–393`); `GILBERT` requires `IBZINT=2/6` (`gil.f:89–93`); `CHILANDAU` calls `TAU_STD` (`susc_landau.f:221–226`); `COMPTON` creates a periodic k-space matrix (`compton.f:917–935`). See [task reviews](tasks/).
4. **Local operators:** the normal `GEN` path invokes `DENSITY` (`FMAG`), `CURDNS` (`JALF`), `ORBCURDNS` (`JORB`), `SUSC_GAMMA` (`STONER`) and `XRAYSPEC` (`XAS`). Their caveats and general defects still apply. In particular, the host-wavefunction scratch error diagnosed for **embedded** `STONER` must not be automatically inferred for normal `MOL` with no host types; unrelated `SUSC_GAMMA` numerical risks remain.
5. **Restricted Yes cases:** `PHAGEN` only converts a previously converged ASA/VWN `NT=1` potential; `PLOTPOT`, `PLOTPRS`, `PLOTSFN`, `WFPLOT`, `SOCPAR`, `PSHIFT`, `MECHECK` and `MEPLOT` are local potential-based operations, **not independent proof of a cluster SCF solver**.
6. **Validation level:** this is source-path applicability based on the **existing** Fortran audit, not a new native integration suite. Yes does not guarantee every `FULLPOT`, `MPI`, magnetic, `ENERGY` or `CPA` combination or physical precision. Actual structure/parameter validation must be separate from `supports={"3d","2d","cluster","embedded"}`.

**GUI rule:** use this matrix's Yes column for the free-cluster kind—not the mere presence of a `TAU.CLUSTER` keyword and not the embedding matrix. Postprocessing requires an existing suitable potential; physical calculation tasks must ensure `MOL`, `IBZINT` and the all-site `NQ` representation are consistent.
