# BSF audit of common definition BSFEK/BSFKK

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

| Task | Schema | Executable | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| BSFEK | BSF | kkrgen | true | false | BSF |
| BSFKK | BSF | kkrgen | true | false | BSF |

## Definition and validation

- NK1/NK2 and NK have minimum 2, NE minimum 1. BLOCHSF divides by NK1−1 and NK2−1 when parameterizing the K-K map (blochsf.f:939–940).
- The dynamic GRID default is 3 for E-K and 0 for K-K, and is explicit in the input. Single-point GRID=3 leaves an integration weight uninitialized.
- K-K validation rejects nonfinite directions and zero or collinear planes relative to KA.
- K1/K2 are endpoints; BLOCHSF subtracts KA. It orthogonalizes nonperpendicular relative directions, so the specified parallelogram need not be preserved (blochsf.f:438–455).

## Defaults: written versus omitted

E-K: preserved NE=200, GRID=3, NK=300, ImE=0.001 Ry, EMIN=−0.2 Ry and EMAX=−1 Ry. **EMAX=−1 is real energy, not sentinel for EF.**Native init_mod_energy uses substitution EF to EMAX<−99. The default range −0.2→−1 is therefore decreasing and surprising for the normal spectrum. This is an explicit warning ASE default, not a correction of an ignored parameter native reader. It is appropriate to set its own range, possibly in the future default, to change to a physically understandable range to EF; it is not possible to determine the appropriate absolute top energy without potential.

Native NK at reading E-K is 800, ASE writes 300. This is a valid explicit deviation; 800 may be better for fine paths, but the final number is divided between segments and may be adjusted according to k-table capacity. Neither 300 nor 800 is a guarantee of convergence.

K-K: generated energy EMIN=0.7 Ry is explicit ASE option, EMAX copy EMIN; it is not automatically EF. Written NE=1 and GRID=0 correspond to one point. The correct energy or relative energy EMINEV should be set to find the Fermi area. Unwritten KA native completes with zero start; Python validation uses the same beginning. NK1/NK2/K1/K2 do not have an imaginary defaults and must be supplied by the user.

The definition declares mpi=True. BLOCHSF has a parallel energy process and GEN allows it; however does not verify has a numerical conformity MPI result. MODE/CPA/STRCONST/SITES remain common sections unchanged.

## Other restrictions

E-K is selected by the actual energy count NE>1, K-K at NE=1; BSFEK/BSFKK are distinct Python task identities sharing schema BSF, with mode-specific initial defaults, not two different native dispatch keys. TASK with writes as BSF, CONTROL ADSI as BLOCHSF. Existing validation warns in conflict with the required alias and NE.

In E-K mode native divides points by segment length, with a minimum of two points per segment. Degenerate explicit segments and zero total length can lead to problematic distribution. Their complete control, together with the geometry of each KPATH/Bravais combination, is not solved in this definition.

For 2D native in K-K the branch zeros the z-folder K1/K2 before deduction KA. Input-only check does not know the potential dimension, so the plane valid according to specified 3D vectors can degenerate after native transcription. For 2D, explicit consistent plane input is required.

At native reading TAU cache can take BLOCHSF energy from cache; the consistency of external files is not verified here. This audit does not meet Advanced BSFSPN/BSFORB/BSFETD/BSFALF orEPHOT.

## Verification

Current regression modules: `test_bsf_audit.py`, `test_bsf_file_manifests.py`. They are included in the integrated input-parameter and result_files suite (**1249 passed, 1 skipped**, 2026-11-09). Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
