# XAS audit of the definition and native behaviour

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

| Task | Schema | Executable | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| XAS | XAS | kkrgen | false | false | default |

## Definition and validation

1. **MPI does not complete the spectrum.** GEN:1355–1368 stops XAS and XES after the MPI energy loop, before XRAYSPEC (1378–1385). The interface uses mpi=False. This is source evidence; numerical MPI agreement was not measured.
2. **EMAX is a real member defaulting to 4 Ry.** ENERGY creates it through the emax argument; it can be set normally, and 4 Ry is written explicitly. An entry in defaults alone would not create that parameter.
3. **MECHECK is a separate task.** INITVAR:162–167 selects test11 with different workflow flags. XRAYSPEC does not read this option, and XAS does not expose it. Combining task keywords does not establish a local XAS check.
4. **CL selects a core shell:** validation permits n=1..5, s/p/d/f and l<n. XRAYSPEC uses SHELL(5) for K,L,M,N,O and indexes NCXRAY at line 491. Availability of the selected shell requires checking the actual potential.
5. **IT must be a positive type index.** Its upper bound and the existence of the selected core shell depend on the actual potential.
6. **Frame rotation:** FRAMEPHI, FRAMETET and FRAMEGAM are three Euler angles; INIT_MROT_FRAME constructs GETMROT(-gamma,-theta,-phi). They do not specify one electric-field vector. XRAYSPEC computes three polarization channels.

## Parameters and validation

The definition exposes ME, POLOBS, DIPOL, GFCONVRH, WRREGIRR, NONDIPE2 and NONDIPM1 as read by XRAYSPEC:295–329,442–465. POLOBS=S writes (+,-,z), C writes (x,y,z). ME=NAB forces DIPOL=1. When both non-dipole flags are present, NONDIPE2 takes precedence; validation warns about these interactions.

Validation requires FREL, valid core shells, finite frame angles, NE>=2 and a single energy path. CHECK_INPUT:43–47 explicitly requires IREL=3. XRAYSPEC:376–380 requires ENERGY.NE to match the first TAU contour. The input definition does not read TAU or the potential and therefore cannot certify that match.

The interface does not expose MESPLIT (after reading is K_ME_SPLIT always true), TAUCORE as supposedly effective management XAS breaking or DYNAMICAL without auditing its separate routine. TAUCORE with a common reader reads but its effective application is in branches XMO/XRS; standard call ME_ALF_ALF sets CECOR=0. DYNAMICAL for RXAS/XMO actually switches to XRAYSPECDYN, which is not this audited static workflow.

## Defaults: written to the input and omitted

| Option | Native XAS |ASE2SPRKKR|
| --- | --- | --- |
| ENERGY.GRID | 6 |6 explicitly, unchanged|
| ENERGY.NE | 180 |180 explicitly, unchanged|
| ENERGY.ImE | 0,01 Ry |0,01 explicitly, unchanged|
| ENERGY.EMIN |After a general reader XAS default EF|-0,2 Ry explicitly, left|
| ENERGY.EMAX | 4 Ry |4 Ry explicitly, newly actual parameter|
| TASK.IT / CL | 1 / 2P |1 / 2P explicitly, left|
| OUTPUT |BARN, output in Mb|None omitted; accepts BARN, MBARN, SIGMA|
| POLOBS | S | S omitted |
| ME / DIPOL | ADA / 0 | ADA / 0 omitted |
| GFCONVRH |false in absence of keyword| false omitted |
| WRREGIRR |Effectively false: SECTION_FIND_KEYWORD overwrites DATA true| false omitted |
|Frame angles|0/0/0, without keyword CHANGE_FRAME=false|Same zeros omitted|
| MODE | FREL | None omitted |

EMIN=-0.2 Ry is written explicitly. Native EF is a natural lower bound for standard absorption, but the effective interval also depends on the native initialization and the requested scan. The explicit default is not a convergence result.

Note: in INIT_MOD_ENERGY:234–249 is XAS in absence EMF returns to EMIN=EF, so the absolute EMIN itself may not always remain effective. Relative pairs EMINEV/EMAXEV and its subsequent processing require inspection of this branch. definition does not introduce a fictitious guarantee that Python default always determines the actual start native scan.

OUTPUT=SIGMA uses 10^15/s; the other OUTPUT modes use Mb. Omitting OUTPUT leaves native BARN. The definition accepts the native BARN name and compatible MBARN spelling without forcing an explicit default.

## Other native restrictions

- MEFORM is loaded, NAB affects DIPOL and GRV prepares the derivative. The standard calculation still calls ME_ALF_ALF without giving MEFORM. It cannot be argued that three accepted names mean three independent, numerically verified Gauge implementations.
- Non-dipole call is only for .NOT.FULLPOT and DIPOL=0. FULLPOT does not therefore need to evaluate the same contributions with the same flags.
- The calculation selects a single type IT (NTXRAY=1). It does not issue a list of types.
- The number of points, mesh and type of potential must be consistent with TAU directors. Only successful serialization input does not test this condition.
- XAS and XES with the same DATASET, type and shell may use the same .rat name. It is necessary to distinguish outputs at work with both tasks.

## Verification

Current regression modules: `test_xas.py`, `test_xas_definition.py`. They are included in the integrated input-parameter and result_files suite (**1249 passed, 1 skipped**, 2026-11-09). Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
