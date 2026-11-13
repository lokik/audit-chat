# PLOT2DPOT / PLOT2DRHO — SPR-KKR 9.7

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

| Task | Schema | Executable | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| PLOT2DPOT | PLOT2DPOT | kkrgen | false | true | default |
| PLOT2DRHO | PLOT2DRHO | kkrgen | false | true | default |

## Dispatch

initvar.f:201–205 sets PLOT2DPR(1) by PLOT2DPOT and flag2 by PLOT2DRHO; both maps internal TASK=PLOT2DPR. GEN (gen.f:1466–1478) selects FPPLOT2DPR for FULLPOT and PLOT2DPR_ASA for ASA.

## Essential Real Behavior

ASA route plot2dpr_asa.f does not contain density array reading or conditional resolution PLOT2DPR flag1/2. Creates potential PLY and geometric line outputs (solid names PLY-V-ASA-... and POT-ASA-LINES.ply). PLOT2DRHO is therefore not the density of the fence to ASA. Definition rejects explicit FULLPOT=false for PLOT2DRHO; actual potential must be FULLPOT, flag does not convert it.

FULLPOT routine fpplot2dpr.f:70 first calls unconditionally PLOT2DPR_FP_PLY. This passes through both potential and density arrays and non-wind outputs according to individual PLOT2DPR flags; to 926 calls STOP_REGULAR with run of test version completed. The following common plane-grid calculation is unattainable in this path. Native selectors therefore do not separate PLY potential/density products as promised by their names. This is not just a input validation problem.

The next STOP in fpplot2dpr.f:671 after writing FULLPOT-pot.ply is in a subsequent unreachable routine; it must not be confused with the first termination. A correction of only this STOP would not suffice. V of this subsequent routine is additionally V<-1 cut to -1 before writing PLY; we do not consider it to be currently available output.

## Geometry and defaults

TASK.AVEC/BVEC/DVEC are three-component Cartesian vectors in units of ALAT, not directly in atomic units or fractional basis coordinates; both readers defaultsA=(2,0,0), B=[0,2,0), D=0. B is orthogonalized and maintained at non-zero A.B by its original length. Zero/parallel vectors cause division by zero, definition rejects them as well as non-finite geometry.

Unit evidence: plot2dpr_asa.f:138–139 uses RWS/ALAT and line 173 converts distance to radial atomic units by multiplying ALAT; FULLPOT PLY does the same at line 259. The ASA comment about atomic units at line 102 conflicts with the actual conversions. Python help uses Cartesian ALAT units.

NA=50 explicit default corresponds to the reader. NB=50 is explicitly written; if absent NB native is calculated nint(NA*|B|/|A|), which can give0 for very small B. Explicit50 is safer for predictability, but it changes aspect-ratio sampling with different long vectors. NA/NB>=1.

PLOT2DPR_FP_PLY has its own range of fixed sampling parameters: NA/NB are not a guarantee of normal rectangular grid output. Support input parameters does not mean that each of them is used in the script test as in the later plane-grid writer.

GRID0/NE1/ImE0 are explicit startup values; plot task does not exercise an energy loop. Absent FULLPOT for PLOT2DPOT leaves native to recognize potential format; PLOT2DRHO writes true. MPI disabled because GEN guard plot tasks does not permit. DMFT/LDAU plot branches native also rejects; definition do not offer these MODE parameters.

## Verification

Current regression modules: `test_plot2dpr.py`, `test_plot_selector_file_manifests.py`. They are included in the integrated input-parameter and result_files suite (**1249 passed, 1 skipped**, 2026-11-09). Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
