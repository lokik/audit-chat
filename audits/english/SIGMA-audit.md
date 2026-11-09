# SIGMA — SPR-KKR 9.7 / ASE2SPRKKR

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

| Task | Schema | Executable | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| SIGMA | SIGMA | kkrchi | false | true | default |

DC conductivity and selected response tensors in kkrchi within the audited periodic ASA/FREL scope. Single-point Fermi-surface output is not a complete Fermi-sea calculation.

## Mapping and Options

INITVAR recognizes TASK SIGMA, requires KKRCHI and turns off RDTAUDIR/WRTAUDIR for normal branch without NLCPA. RSP calls SIG. The definition contains CONTROL, TAU, MODE, CPA, STRCONST, ENERGY, TASK, SITES; minimum SCF allows to refuse FULLPOT as outside the scope of the audit. Native contains FULLPOT routines, so this rejection is not a claim that SIGMA FULLPOT does not work.

The audited scheme is FREL, BZINT=POINTS, without many-points OP. NONMAG is permissible. Native also offers IBZINT=4 and 6 and nonrelativistic branches that this definition does not certify.

- GRID=0, NE=1: pouze Fermi-surface postup. SIG ignores the initialisation point energy and actually counts EF+i*ImE.
- GRID=5, NE>1: one integration contour for Fermi-sea part and an additional point at EF for Fermi-surface part. SPLITSS and other combinations are not covered by this definition.
- SIGSPROJ selects operators 1..16; valid range, length, absence of duplication and native requirement of first index 1 or 9. SP_NO_ALPHA and SP_NO_NABLA adjust spin polarization operator. NOSYMSIG turns off global native symmetry, not just print or output format.
- FRAMETET/FRAMEPHI/FRAMEGAM are Euler frames in degrees.

## Significant ALE

1. **NE=1 misses Fermi-sea integral.** Native sets NEINTEG=0; at NE>1 uses all contour points and adds a point to EF. A single point result cannot generally be classified as a complete Hall or all other responses. For relevant tensors it is necessary to include and converge the Fermi-sea part. This is the difference in the content of the calculation, not only its accuracy.
2. **Strong SIG_MODE=STREDA.** V SIG is fixed STREDA and numerical energy derivative uses DERYD=0.0001 Ry. The definition does not promise to switch formalism or change this step through the input key.
3. **K-mesh is changed at runtime.**SIG_DRV_KMESH results in a number of points for each energy from NKTABINP; at higher NKTAB, deeper energy is sampled more effectively, for example according to (NEA+1−IEA)^3 and lower limit NKTABINP/1000. NKTAB is not a promise of the same number of points throughout the contura. Common default 250 is not evidence of convergence of transport.
4. **ImE is not an electron temperature.** It is an imaginary part of energy in Ry. Explicit 0.01 Ry corresponds to approximately 0.136 eV and can significantly influence clean material. Native zero may not be better because for clean systems the limit is DC singular; appropriate width and its convergence must be selected according to the system.
5. SIGSPROJ without input in FREL selects all 16 operators including spin, torque and Edelstein responses. For electric conductivity itself, it is useful explicitly SIGSPROJ={1}; the definition retains native unwritten selection. The nabla-nabla representation has an index 9.
6. Native PROJECT, NLCPA, thermal and many-body procedures are not audited here. PROJECT performs separate projection passages, not a mere distribution of normal tensor. Native has MPI k-loop; left here mpi=False because the whole parallel string has not been certified. This does not demonstrate an error MPI.
7. SIGSUM writes fixed name sigmaXXXXX.dat by temperature index and STATUS=REPLACE. Repeated calculation in the same directory can overwrite the previous file; using another DATASET does not change this name itself.

## Defaults and their writing

| Parameter | Native |Definitions|
|---|---|---|
| GRID / NE | SIGMA: 0 / 1 |0 / 1, written down; skips the sea integral|
| ImE | SIGMA: 0 Ry |0.01 Ry, written; deliberate derogation|
| EMIN |for SIGMA first EF|−0.2 Ry, written down; surface point does not use it, used in the selected contura|
| EMAX | EF |is omitted from the input; remains actual native EF|
| SIGSPROJ | FREL: 1..16 |not entered, is omitted from the input; same native selection|
| SP_NO_ALPHA / SP_NO_NABLA | false |false not to be published|
| NOSYMSIG |search for the presence of a key| false, is omitted from the input |
|Euler angles|0 and change of frame by presence of key|not entered, omitted from the input|

The defaults that are omitted from the input were not replaced by other implicit behaviours. Default sea+surface with GRID=5/NE=30 would better cover the general Hall response, but would be significantly more expensive and its EMIN and mesh would still require convergence; left native quick surface default with an explicit description of the restriction. FULLPOT=False is omitted from the input and does not transfer full-potential potential to ASA; if the potential activates it, input-only validation itself will not find out.

## Verification

Current regression modules: `test_sigma.py`. They are included in the integrated input-parameter and result_files suite (**1249 passed, 1 skipped**, 2026-11-09). Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
