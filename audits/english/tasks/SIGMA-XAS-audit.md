# SIGMA-XAS — SPR-KKR 9.7

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

No separate canonical Python task is exposed; the native branch and its reachability are described below.

## Reachability defect

`rsp.f:941–943` dispatches internal TASK='SIGMA-XAS ' to SIG_XAS; `sigme.f:499` has the corresponding matrix-element branch. `sig_xas.f` contains a real implementation for electric-field-induced XMCD.

However, `initvar.f:351–355` recognizes the complete keyword SIGMA and sets TASK='SIGMA '. The supplied initialization routines do not select SIGMA-XAS. SECTION_FIND_KEYWORD requires spaces around the complete keyword (`scan_input.f:144–201`); SIGMA-XAS is not a SIGMA prefix match. Adding SIGMA selects conductivity, and CONTROL.ADSI does not assign internal TASK. KKRCHI passes the selected TASK to RSP (`kkrchi.f:1425–1427`). Thus ordinary input cannot reach this branch in the audited version.

## Internal reader: source map, not a supported input recipe

SIG_XAS reads TASK.SIGSPROJ, GRID (default 3; allowed 3/6), NEA=50, EMIN=EF, required EMAX, IME=0.01, NEB=1, DEB=0.05, EPS=0.02, IPOLEFIELD=2, IT or IQ, and TAUCORE. These are TASK controls, not automatically the usual ENERGY section. `sig_xas.f:293` requires first current operator 1 or 9; line 299 requires IBZINT=2/4/6. Core-shell selection, lifetime and initialization still need an independent audit.

## Interface and required work

The Python interface has no artificial SIGMA/XAS alias for this branch. A native parser selection is required first, then verification of initialization and a numerical reference case. Reaching the existing routine would not by itself certify its numerical correctness.

## Verification

The reachability and native-defect findings follow from the source paths cited above. Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
