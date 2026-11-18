# POSANIPREP — audit SPR-KKR 9.7 / ASE2SPRKKR

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

| Task | Schema | Executable | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| POSANIPREP | POSANIPREP | kkrgen | false | true | default |

## State and scope

The POSANIPREP definition prepares positron
documents POSANI over kkrgen, for standard 3D ASA/FREL system, one energy
GRID=0, NE=1 and electron input potential.

Category: **works but has significant caveats** in limited workflow NQ=NT;
**broken** in export TAUQQ for general NQ≠NT. Task is active. Numeric
SPR-KKR calculation not started.

## Involvement and Real Meaning

`gen.f:414–426` detects POSANIPREP only after initial energy initialization;
POSANI takes precedence over POSANIPREP. Definition does not write both keywords.
`gen.f:431–455` creates the necessary fields, calls POSANIPOT and turns on regular
i irregular wave functions and matrix elements.
`gen.f:1566–1571` then calls POSANIPREPARE.

Preparation is not independent minimization or finding the bottom of the positron band.
`posaniprep.f:86–88` only requires NETAB(1)=1 and consistency of current energy
s ETAB(1,1). For GRID=0, NE=1 sets EPATH energy directly to EMIN
(`epath.f:81–95`). **The user must select the energy of the positron state**, typically
at the bottom of the belt; -0.2 Ry is not found result. Definitions in validation are noted.

DATASET_pos_data.pan output contains selected positron wave functions and
scattering data. The following POSANI uses electron potential and this
File; POSANIPREP by contrast, internally converts electron potential to
Positron (`posanipot.f:124–125` and follow-up correction).

## Parameters

- TASK.POSVC = BM / DR / IPM. `posanipot.f:240–252` reads the current keyword
Section TASK, selects VCEPBM, VCEPDR or zero correlation for IPM. He's exposed
  pouze v ASA rozsahu. This is not the final conversion parameter POSANI spectrum.
- ENERGY.GRID=0, NE=1, EMIN selected absolute energy in Ry.
- TAU.NKTAB and other common k-strip parameters. POSANIPREPARE reads again
NKTAB (`posaniprep.f:116`) and produces TAUQQ over KMESHS/KLOOPSIJ.
- MODE must be FREL; NONMAG allowed. Natively still needed KMROT=0
(`posaniprep.f:72–77`) and 3D (`posanipot.f:88–89`).
- SCF.FULLPOT is only exposed to rejection of unaudited FULLPOT workflow.
False is omitted from the input and empty section SCF with does not create. Native FULLPOT branch
exists; it is not declared off. FULLPOT set by potential without
the keyword cannot be excluded by the entry validation itself.

## Defaults and writing

| Parameter |Definitions and writing|Native effective behaviour|Assessment|
|---|---|---|---|
| GRID, NE | writes 0, 1 |POSANIPREP requires a single energy but does not have its own initialization of the mesh defaults before GEN|explicit compatible option|
| EMIN |writes -0.2 Ry, user changes|Common default -0.2 Ry|the same value, physically need not be the day of the positron band|
| ImE | writes 0 Ry |Common default 0.01 Ry but GRID=0 has real energy|different explicit parameter, in this mesh does not change energy|
| EMAX |not issued is omitted from the input|the sentinel is converted to EF|the same native behaviour; GRID=0/NE=1 uses EMIN, not EMAX|
| POSVC |BM, writes| BM |Taken native default|
| SPLITSS | false, is omitted from the input | false |same; turns off|
| MODE |FREL without keyword| FREL |the same|
| SCF.FULLPOT | false, is omitted from the input |actual mode may be based on potential|absence native mode does not overwrite; obligation to use ASA potential|
| TAU.BZINT |POINTS, writes|Common selection depends on the system|explicit limitation of the standard periodic workflow|
| TAU.NKTAB |Common ASE default 250, writes|Common native initialization k-strip|explicit choice, not guarantee of convergence|

We take native BM and general -0.2 Ry; we don't add invented default energy
the bottom of the belt. EMINEV is rejected so that energy is not related to electron EF
which could be misleading for this entry. Alternative relative energy is not
declared to be natively prohibited; it is a restriction on the audited definition.

## Documented errors and caveats

1. **NT instead of NQ in export TAUQQ.** `posaniprep.f:65–69` allocates TAUQQ
as a location field NQMAX×NQMAX but `:268–272` writes in cycles 1..NT.
The following POSANIREAD uses the same error range. For NQ>NT part
physical pairs are missing; POSANI reads them from a uninitialized array. For NT>NQMAX
export can access out of bounds. NQ=NT eliminates this particular non-compliance,
does not constitute a general proof of correctness. The condition depends on potential and is not
automatically verified by validation InputParameters.
2. **Restriction to s_1/2 positron wave.** `posaniprep.f:91–106` selects
ZG/ZF(:,1,1), not the whole relativistic state. Later POSANIREAD will also suppress
imaginary part ZGPOS. These are native approximations that require physical
Interpretation; these are not errors defaults definition.
3. **The presence of DATASET_positron.rho can suppress the preparation of wave functions.**
`gen.f:500–504` reads the existing density and sets RUNELOOP=false. Preparatory
The routine still requires current wave functions and energy. The very existence
old density is not a guarantee of the necessary materials. This binding must be at:
real run checked; definition of files does not delete and cache does not automatically solve.
4. **Same size NKMQ in all types/places.** `posaniprep.f:174–185`
controls identical dimension, otherwise STOP. It also requires KMROT=0 and 3D. Without
potential these conditions are not all decisive.
5. **Tential conversion depends on SCFVXC and density.** POSANIPOT has native
branches NONE/VBH/MJW/VWN/PBE, other variants may end STOP. Definitions here
does not overwrite VXC or fake an audit of the entire conversion of all functionals;
The consistency with the electron calculation remains a condition of use.

## MPI

The definition has mpi=False. POSANIPREPARE does not have export protection by MPI_ID; and
opens/writes common DATASET_pos_dat.pan. Keyword POSANIPREP may not change
Internal TASK of NONE, so normal final MPI control GEN does not need this procedure
ani zastavit. Without separate numerical verification of lifecycle and collective
calculations cannot be considered as safe for parallel preparation. It's not a claim that
native code contains explicit prohibition MPI.

### Output prefix

For this output family, native DATSET includes nonempty ADSI and corresponds to Python template `{DATASET_ADSI}`. DATASET shorthand in a filename description must not be confused with unexpanded DATSET0; fixed filenames are identified separately.

## Verification

Current regression modules: `test_posaniprep.py`. They are included in the integrated input-parameter and result_files suite (**1249 passed, 1 skipped**, 2026-11-09). Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
