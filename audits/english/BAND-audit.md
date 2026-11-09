# BAND in KKRSPEC excl. separate definitions and audits

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

| Task | Schema | Executable | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| BAND | BAND | kkrspec | false | true | SPEC |

Complex Bloch roots in KKRSPEC and their Im(kz) filter. This is distinct from ordinary BSF or EKREL E(k) calculations.

## Meaning of the native calculation

SPEC_RSLAB recognizes BAND and sets IBLOCH=0. SPEC_BLOCH in spec_band.f solves a generalized problem and creates a complex kz z -i*log(eigenvalue)/BRUSEP, with further correction AKCP. It adopts roots according to abs(Im(kz))<EIG_IME_TOL. EIG_IME Therefore, there is no energy in Ry or linewidth in eV; it is a cutoff imaginary part of Wavector in native measuring convention.

SPEC_INPUT at NT=1 prints the label PROJECTED BAND STRUCTURE and at NT>1 NORMAL (along kz). However, SPEC_BLOCH itself stores real kz in K_EIG for NT=1 and KPARA for NT>1. For the interpretation of k-columns, these are actually written to the input variables, not taking the general label from the reader. SPEC_VBRUN subtracts EFERM from E_EIG at the end.

## Separate definition

Shares structure and preparation with ARPES but sets TASK BAND, CONTROL ADSI BAND and adds SPEC_BAND:

- NEIGMAX: positive capacity of stored roots, default 100000 Native ends STOP when exceeded.
- EIG_IME: final positive cutoff, explicit 0.01. Zero would not accept exact real root at strict native inequality.
- CALC_TRANS_PROB: optional attribute of calculation TRLEED transmission properties; default false is not written and corresponds to MOD_SPEC_BAND default.

Default K1={0,0},NK1=1 is written and selected by explicit moment point k_parallel=0. Native has a valid single point to the branch. The path can be set K1 and NK1>1 or K2/NK2 if applicable. These are shift vectors added to KA as in SPEC_INPUT, not endpoints BLOCHSF.

The validation requires this explicit moment procedure, positive numbers, finite vectors and their numbers, one energy mesh without SPLITSS and the finite nonnegative workfunction/mute. Refuses K3/K4/NK3/NK4 which reader does not read. INPVER=0 is governed by the external old input.inp, the content of which does not Verifyme here.

## Defaults: explicit deviations and omitted behaviour

In BAND definitions are **written** IMV_INI_EV=0 and IMV_FIN_EV=0 eV. For non-absorption band procedures are more suitable than taken photoemission 0.05/2 eV, but for complex/disordered band workflow the user can choose nonzero damping. Damping is not electronic temperature.

EIG_IME=0.01 is **written**. Native starts at 0.01 but with omitted EIG_IME and VIL>1e−4 sets 9999, thus practically shuts down the filter. Python therefore does not promise unwritten default 0.01 and does not leave the filter dependent hidden when changing damping.

NEIGMAX=100000 is written to the input and corresponds to native initialisation. Zero CALC_TRANS_PROB is omitted from the input and retained native false. CRYS_VECS=False is omitted and preserves USE_CRY_PRIM_VECS=true. In 3D surface construction, presence of CRYS_VECS sets USE_CRY_PRIM_VECS=false (spec_rslab.f:180–181). ARPES, AIPES, BAND, SPLEED and CLPES share this definition; the native reader does not read singular CRYS_VEC.

Listed NE=300, EMINEV=−8 and EMAXEV=+5 eV against EF, EWORK_EV=4.2 eV, NL=4, N_LAYER=50, NLAT_G_VEC=57 and N_LAYDBL={10,10} remained practical choices. For bands containing both occupied and unoccupied states, +5 eV has a sense as opposed to valence AIPES. The number of points and cutoffs must be converged; native smaller initialization numbers are not automatically better.

SPEC_PH is left as part of shared native spectroscopic preparation. It cannot be assumed that BAND is the photoemission intensity or that EPHOT is the shift of all energy bands: SPEC_VBRUN for IBLOCH=0 sets EHIGH=ELOW before FINALSTATE. This audit does not certify any dependence of auxiliary preparation on photon/angle choices.

## Specific native errors and ALE

1. spec_band.f:70–80 at an inappropriate ratio EIGA/EIGB writes SPEC_BLOCH: not converted but does not perform CYCLE. EIGENVAL remains zero and follows CDLOG(EIG). The failure thus reported cannot be ignored; it can create an invalid complex kz. Python definition to does not fix.
2. The imaginary kz filter and non-absorption default is not proof of the physical correctness or existence of all belts. Numerical tolerance, energy mesh, surface direction and reciprocal basis affect the roots taken.
3. CALC_TRANS_PROB is native available but transmission amplitudes and their normalization have not been numerically verified. Default false preserved.
4. Native BAND in SPEC_VBRUN enforces ATA=1; disorder/CPA result therefore requires own interpretation and is not generally certified by this definition.
5. mpi=False is a conservative setting. Native has its own parallel collection of band data over rank files; this string has not been numerically tested. There is no evidence of error of all MPI calculations.
6. Surface structure, workflow, FULLPOT, many-body and thermal branches are not completely certified by this input-only audit.

## Verification

Current regression modules: `test_band.py`. They are included in the integrated input-parameter and result_files suite (**1249 passed, 1 skipped**, 2026-11-09). Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
