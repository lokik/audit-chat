# VBUPS / VBXPS — aliasy VBPES v SPR-KKR 9.7

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

| Task | Schema | Executable | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| VBPES | VBPES | kkrgen | false | true | default |

Native VBPES aliases: initvar.f:224–249 recognizes VBXPS/VBPES/ARPES/VBUPS and selects internal VBPES or ARPES according to ANGLE_INTEGRATED. VBUPS/VBXPS alone select VBPES; the Python interface uses task VBPES.

## Important error in expected defaults

vbpes.f:197 reads TASK.EPHOT with default1253.6 eV, for both aliases as well. Not branching according to VBUPS/VBXPS. NINT(EPHOT)=1 means1253.6 and NINT(EPHOT)=2 means1486.6 eV. VBUPS does not set itself21.2 or40.8 eV. Photon energy for UV photoemisi must be specified by the user explicitly in the definition VBPES. Adding an absent UV default which is not written would be incorrect; explicitly written21.2 eV would be possible, but would be a new Python convention, not native default alias.

## Combination with ARPES

VBXPS then independently controls ARPES and changes ANGLE_INTEGRATED depending on its presence. It is not a guarantee that any combination of tokens will maintain an integrated calculation. We do not represent new aliases as compensation KKRSPEC ARPES/AIPES; executive and native implementations are essential.

## Recommended use and audit output

Use existing definition VBPES and set TASK.EPHOT in eV by source. No duplicate stand-alone input definition or regression test is created, copying the same native mapping. This check complements earlier audit VBPES; does not verify numerical PES results.

## Verification

Current regression modules: `test_vbpes.py`. They are included in the integrated input-parameter and result_files suite (**1249 passed, 1 skipped**, 2026-11-09). Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
