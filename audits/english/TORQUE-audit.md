# TORQUE audit of existing definition and correction of validation

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

| Task | Schema | Executable | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| TORQUE | TORQUE | kkrgen | true | false | default |

## What Parameters Mean

THETAQ/PHIQ in TASK are not rotation magnetization. They determine the unit vector for the axis evaluated by the torquu. TORQUE:74–75 first requires MOMENTS_ROTATED; KKRGEN:846 sets this flag for KMROT=1 or 2. MROTQ comes from shooting moments. TORQUE:168–175 controls m dot u = 0 with tolerance1e-8. The angles TASK themselves do not create this condition or rotation of the moments.

The order of values is by site IQ, not by atomic type IT. SECTION_SET_REAL_ARRAY (scan_input.f:701–707) repeats the last item delivered to the remaining site. So the single element list really applies to all sites. The longer field must not exceed NQ; the definition without potential does not know NQ and does not control this limit. The lengths of both lists may not be the same, because each one extends separately.

## Defaults

| Parameter |SPR-KKR missed|ASE2SPRKKR|
| --- | --- | --- |
| THETAQ/PHIQ |if both are missing:45°/0°; if only one is entered, the other field remains0|explicitly90°/90°, left|
| ENERGY.GRID |TORQUE does not have its own initialization default|explicit8, left|
| ENERGY.NE |without overwrite MOD_ENERGY starts at0|explicit36, left|
| ENERGY.ImE | 0,01 Ry |explicit0 Ry, left|
| ENERGY.EMIN | -0,2 Ry |explicit -0,2 Ry, left|
| ENERGY.EMAX |General Sentinel replacement for EF|not at all a member of this definition, natively EF|
| MODE |Commonly FREL| None, omitted |

90°/90° gives u along y. Native45°/0° gives direction in the plane xz. Neither choice is universally better: it must be perpendicular to real magnetization. Default ASE2SPRKKR is explicitly written, so its difference is not a mistake. It is appropriate to choose any change together with the convention of shooting moments; this definition default does not change.

GRID=8 uses a logarithmic integration contour. EPATH:249–264 sets fixed EPS=0,050 and transmits it to EGRID8_IME instead of extracting this parameter from ENERGY.ImE. The difference between explicit ImE=0 and native0,01 does not thus show a different contour GRID8. The suitability of GRID8/36 against another integration contura requires a convergence comparison.

ENERGY.EMIN is explicitly -0.2 Ry; the upper bound delegates to native EF. The interface does not rely on an ineffective EMIN key in the defaults dictionary.

## Definition and validation

- THETAQ/PHIQ are nonempty lists of real values; NaN/Inf is rejected at final validation.
- ENERGY.NE must be positive.
- Refusal to SPLITSS and more routes GRID/NE: TORQUE reads always WETAB(IECURR,1) and ETAB(IECURR,1), not the general route. Without protection, the entry could give an impression of the calculation of both contributions.
- The documentation of the parameters distinguishes the axis of the torquu from the recording of magnetization and describes the expansion of lists.

The definition warns for NREL/SREL and NONMAG. Their text claims that the calculation fails but owns TORQUE does not control NONMAG/IREL. It uses fully relativistic operators J=L+S and depends on the surrounding workflow. The actual availability of other schemes cannot be decided only by this warning; no ban is transferred to the definitive native. The warning does not cover SP-SREL. This remains the outstanding question support of modes, not demonstrated functional scalar-relativistic TORQUE.

## Finding and limiting native calculation

1. The requirement of MOMENTS_ROTATED and the perpendicularity of m is not verifiable from InputParameters alone. Existing default input with standard unrecorded potential can end up STOP.
2. The routine sets IPRINT=1 regardless of CONTROL.PRINT and produces large matrix even in silent input.
3. J and UJ are filled into global NKM but local multiplication is used NKMQ(IQ). WKM4 then copies the difference to 1:NKM, i.e. even outside the locally just filled submats with different cutoffs. The specific impact on the trace must be verified; this is not a proven error of all results.
4. At WGT_OCC, only ITOQ(1,IQ) is taken. The Site with multiple components thus requires checking the importance of weighing according to the first type.
5. The main results are the text statement of site torqus and the sum; the individual T(IQ) are in internal Ries, the sum is also translated into meV. TASK does not have its own output file option or a new Python parser.

MPI=true is supported here: TORQUE:186–199 reduces T over MPI_SUM_R and completes the extract on the process0; GEN:1264 has an additional call for energy parallelisation. This distinguishes TORQUE from unfinished MPI branches of some previous tasks. Numerical MPI results have not been verified.

## Verification

Current regression modules: `test_torque.py`. They are included in the integrated input-parameter and result_files suite (**1249 passed, 1 skipped**, 2026-11-09). Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
