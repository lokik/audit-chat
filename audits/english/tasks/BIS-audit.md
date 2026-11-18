# BIS — SPR-KKR 9.7

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

No separate canonical Python task is exposed; the native branch and its reachability are described below.

## Native control flow

1. `initvar.f:264–271` recognizes BIS and sets the ten-character TASK to `BIS       `, RDTAUDIR=true, IHYPER=0 and ICORE=0.
2. `check_input.f:91–93` requires IREL=3. Valid fully relativistic data do not remove the next obstacle.
3. `gen.f:1445–1448` passes BIS unchanged to VBPES.
4. `vbpes.f:174` rejects `TASK(3:5).NE.'PES'` with `TASK ???`. For BIS the substring is `S  `, so this check always stops the calculation if reached.

The examined VBPES routines contain no separate BIS branch bypassing this check. This is a broken ordinary input path, not a disabled placeholder. Rewriting TASK to VBPES changes the requested calculation to photoemission. Removing the check alone does not establish inverse-photoemission physics.

## Interface and required work

The current Python registry has no canonical BIS task. A fabricated VBPES alias would expose a different calculation, not functional BIS. Native implementation or correction must first establish initial/final energies and transition probabilities, followed by a physical reference case and a matching input definition. The source evidence does not justify treating this as merely a faulty guard.

## Verification

The reachability and native-defect findings follow from the source paths cited above. Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
