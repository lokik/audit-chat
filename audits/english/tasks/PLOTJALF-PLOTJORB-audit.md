# PLOTJALF / PLOTJORB — SPR-KKR 9.7

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

No separate canonical Python task is exposed; the native branch and its reachability are described below.

## Intended input

CONTROL.PLOTJALF and CONTROL.PLOTJORB are actual flags (initvar.f:122–123), not separate TASK keywords. Each set internal TASK=PLOT2DPR (lines 205–206). The geometry is read from TASK.AVEC/BVEC/DVEC/NA/NB. In the expected current branch fpplot2dpr.f:194–231 is loaded DATSET0_JALF.dat or DATSET0_JORB.dat; if both are set simultaneously, JALF has priority. It requires corresponding types, radial meshes and cutoff, otherwise STOP data file not consistent. The input data should come from an earlier JALF/JORB calculation.

## Why the definition cannot ensure functionality

GEN Dispatch (gen.f:1466–1478) for internal PLOT2DPR select FPPLOT2DPR to FULLPOT, PLOT2DPR_ASA to ASA.

FPPLOT2DPR calls unconditionally PLOT2DPR_FP_PLY (fpplot2dpr.f:70) at first. This routine does not read PLOT_JALF/PLOT_JORB or current file. It works with potential/density and ends at the end STOP_REGULAR with run of test version completed (plot2dpr_fp_ply.f:926). Return to the subsequent current branch is therefore not started in normal FULLPOT even if PLOT2DPR(1/2) is not switched on. Conditional RETURN to 103 does not help here: requires NOT FULLPOT.

ASA routine PLOT2DPR_ASA current flags or data does not read, draws potential. Therefore, the change FULLPOT flag is not bypassing the problem.

The result may be a different plot output or an earlier error, but never this documented path required by rendering saved JALF/JORB field. The presence of the current file will not repair the dispatch itself.

## Verification

The reachability and native-defect findings follow from the source paths cited above. Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
