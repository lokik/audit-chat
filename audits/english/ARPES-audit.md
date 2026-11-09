# ARPES audit SPR-KKR 9.7 and ASE2SPRKKR

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

| Task | Schema | Executable | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| ARPES | ARPES | kkrspec | false | false | ARPES |

## Definition and validation

- ENERGY.EWORK_EV is the **work function in eV**, not internal potential. SPEC_INPUT reads and converts /RY_EV; sets EWORK in the appropriate branch when entering.
- NT, NP and NK1..NK4 are positive integers; NK1=1 is permitted. The reader has an explicit single-k branch and guards division by NT−1/NP−1 with a >1 condition. A minimum of 2 would exclude valid native inputs.
- K1 requires NK1, at the same time K2 also NK2. Validation rejects nonpositive counts and inconsistent angular ranges and NT/NP.
- ARPES validation rejects K3/K4/NK3/NK4: in supplied SPEC_INPUT reader these keys are not read anywhere. In the public Python definition, they remain for reverse parsing, but validation does not pretend their effect.
- Finite nonnegative EWORK_EV and both IMV_*_EV, finite positive EPHOT; finite electron angles and k-vectors. Zero damping is permitted.
- N_LAYDBL must have exactly two non-negative integers. N_LAYER and NLAT_G_VEC are positive integers. INPVER/STRVER limited to 0/1. At INPVER=0, the new control of the modern reader is omitted because the data comes from an old external input.inp.

## Defaults: written versus omitted

| Parameter | ASE default |Comparison and writing|
|---|---|---|
| EWORK_EV | 4.2 eV |written; overwrites workfunction from SCF/Potential|
| IMV_INI_EV / IMV_FIN_EV | 0.05 / 2 eV |written; native photoemission start VIL/VIH is 0.1/1 eV|
| NE | 300 |written; native photoemission start EMESH=200|
| EMINEV / EMAXEV |−8 /+5 eV versus EF|written; explicit spectral range, not general physical limit|
| NL | 4 |written; explicit angular cutoff|
| NT / NP | 1 / 1 |written, consistent with zero angle range|
| Electron THETA/PHI |0..0 for ARPES|task-specific default; SPLEED has 45/270 degrees|
| EPHOT | 25 eV |written; explicit experimental option|
| N_LAYER | 50 |written; native photoemission start 30|
| NLAT_G_VEC | 57 |written; native start 30|
| N_LAYDBL | 10 / 10 |written; native has a typo LANZ1=10 twice instead of initialization LANZ2|
| INPVER / STRVER | 1 / 1 |written down; modern input and structure generation|

These deviations are explicit input values. N_LAYER=50 or more reciprocal vectors can improve convergence at the cost of time; it is not possible to recommend native smaller numbers as better without the system. Work function and damping are sensitive material/experimental parameters: the value of potential can be more suitable than generic 4.2 eV, but its use requires this explicit key to be omitted. Default is not just formatting settings.

Expert photon polarization and spin-amplitude defaults are generally not recorded on entry. This audit does not constitute a complete certification of all combinations POL_P/NPOL/ICIRC/IDREH or POL_E/POL0/POL0L. Reader rejects some combinations based on the presence of alternative keys; numerical matching default value alone is not sufficient to replace writing and omission.

## Specific native errors and restrictions

1. **IMV_INI v Ry has a typo:** spec_inputs.f:1257 calls SECTION_SET_REAL('IMV_INI',VIH,...), i.e. writes to the final state variable instead of VIL. ASE offers IMV_INI_EV, the branch of which writes correctly to VIL; the defective Ry alias is not exposed. VIL/VIH reader subsequently converts to internal spectroscopic input in eV.
2. **Double LANZ1 initialization:** reader sets LANZ1=10 twice. Explicit N_LAYDBL={10,10} in ASE sets both variables and is therefore better than blank missing fields with expectations of both native defaults.
3. K1/K2 in this reader are **displacement vectors added to KA**, not endpoints as in BLOCHSF. The number of points determines NT/NP and the given vectors are used for k-mesh. Names cannot be interpreted uniformly between tasks.
4. Native photon convention gives normal incidence THETA=180 degrees. It is not possible to automatically transfer geometric meaning THETA from other tasks. The specifics of the crystal frame and sample tilt must be verified for a specific surface.
5. Validation does not verify existence and content of external input.inp/struc.inp at INPVER/STRVER=0, consistency of surface structure with potential and convergence of Layer Doubling and reciprocal Cutoff.
6. The interface uses kkrspec and mpi=False. CRYS_VECS=False is omitted and preserves USE_CRY_PRIM_VECS=true. In 3D surface construction, presence of CRYS_VECS sets USE_CRY_PRIM_VECS=false (spec_rslab.f:180–181). ARPES, AIPES, BAND, SPLEED and CLPES share this definition; the native reader does not read singular CRYS_VEC.

## Verification

Current regression modules: `test_arpes_audit.py`, `test_arpes_energy.py`. They are included in the integrated input-parameter and result_files suite (**1249 passed, 1 skipped**, 2026-11-09). Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
