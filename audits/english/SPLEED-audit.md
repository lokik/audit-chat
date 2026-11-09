# SPLEED separate definitions and audits

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

| Task | Schema | Executable | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| SPLEED | SPLEED | kkrspec | false | true | SPEC |

Spin-polarized low-energy electron diffraction in KKRSPEC. The audited interface uses fixed incidence geometry and a scan of absolute incident energy.

## Different energy semantics

SPEC_RSLAB recognizes SPLEED and sets IBLOCH=1, NSPLEE=0. SPEC_INPUT then for NE>1 reads EMINEV/EMAXEV again and uses them as **conquering energy in eV**, not as relative energy to EF. Take over ARPES −8..+5 eV would therefore be wrong. Definitions writes 11..60 eV, NE=200; descriptions of both EV parameters explicitly indicate this symantics.

Absolute EMIN/EMAX are rejected in this definition at a modern input because native SPLEED the branch is not read in this place instead of EV alternatives. NE=1 is rejected: reader in this case skips reload EV borders and EFEV then takes EMIN from previous general initialization; single point calculation therefore may not have the same meaning as multipoint eV scan. For the real one-point workflow it is necessary to solve this native string, not only set NE=1.

SPEC_PH deleted: native for SPLEED lists photon geometers as obsolete. It is not meaningful to promise dependence LEED to EPHOT or photon polarization.

## Defaults and writing

| Parameter |Separate definition|Native / Meaning|
|---|---|---|
| EMINEV/EMAXEV |11 / 60 eV, written|corresponds to SPLEED start range; direct eV|
| NE |200, entered| native SPLEED start EMESH=200 |
| THETA |45..45 degrees| native SPLEED default |
| PHI |270..270 degrees| native SPLEED default |
| NT/NP |1/1, written down|fixed angles|
| SPOL |1, entered| native IBLOCH=1 default; **unpolarized** |
| EWORK_EV |4.2 eV, written|generic explicit workfunction; native may use potential|
| IMV_INI_EV/IMV_FIN_EV |0.2/0.2 eV, written| SPLEED native start VIL/VIH |
| N_LAYER/NLAT_G_VEC |1/30, written down| SPLEED native start |
| N_LAYDBL |10/10, written down|explicitly sets both variables despite native to double LANZ1 assignment|
| POL_E | PZ |existing explicit convention; native default also PZ|
| CRYS_VECS |false, not written|preserves native use of primitive crystal vectors|

SPOL=1 is a non-polarized calculation even with a draw name SPLEED. This is not the hidden difference defaults; it is written and corresponds to the reader. For spin response, the user must select the appropriate native SPOL procedure and interpret results. All spin output branches are not numerically certified here.

Output work 4.2 can be worse than the value of a specific potential; because with writes it is a real explicit overwrite. The defaults cannot be evaluated only by equality with native. The dampening is in eV and is not electron temperature.

## Other errors and restrictions

- Taken ARPES option CRYS_VEC has the wrong native name. SPEC_RSLAB reads CRYS_VECS and sets USE_CRY_PRIM_VECS=false at its presence. The separate definition contains the correct key and description of shutting down primitive vectors. This is also relevant for the original ARPES definition; this separate original ARPES still retains the incorrect keyword.
- The fixed-angle range does not consciously permit angular scans, a moment scans or POL0/POL0L. Native some of the following procedures have, but polarization depends on the presence of keys: cannot be used simultaneously POL_E and nonzero POL0/POL0L. Blind sharing of all ARPES defaults could end up STOP.
- For INPVER=0 it is an external old input.inp; validation of modern energy keys is not performed in this branch. Existence and content of external files shall not be verified.
- IQ_AT_SURF, MILLER_HKL and the surface structure shall correspond to potential. The definition does not pretend to control this compliance from just input options. Full-potential and other special branches were not audited throughout the chain.
- mpi=False is a conservative choice of definition; no error of whole MPI has been demonstrated nor has a numerical comparison been made.

## Verification

Current regression modules: `test_spleed.py`. They are included in the integrated input-parameter and result_files suite (**1249 passed, 1 skipped**, 2026-11-09). Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
