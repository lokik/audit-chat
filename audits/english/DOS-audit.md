# DOS — audit SPR-KKR 9.7 / ASE2SPRKKR

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

| Task | Schema | Executable | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| DOS | DOS | kkrgen | true | false | DOS |

## Scope and validation

DOS already supported. Standard workflow has **fully meaningful**; numerical accuracy
the specific system does not confirm this audit. DOSREP and other additional outputs
have their own limitations and are not exposed to this definition.

The definition contains CONTROL, TAU, ENERGY, TASK, SITES, MODE, CPA and STRCONST. It accepts NREL, SREL, SP-SREL and common CPA/structure-constant settings; FREL is the native mode without a keyword. Energy defaults are explicit in the input.

Validation requires positive NE, at least two points for GRID=2/3, finite
energy bounds and finite nonnegative ImE. ImE=0 to GRID=3 causes a warning,
not a ban. The checks shall not apply to individual settings related
parameters, but in the final validation/write.

## Native control flow and MPI

INITVAR does not collect a separate internal value TASK='DOS' by keyword DOS;
normal kkrgen procedure remains TASK='NONE' (`mod_calcmode.f:95`, `initvar.f`).
DOS here denotes the standard calculation of the Green function and the output of density of states,
rather than a separate final routine. CONTROL.ADSI=DOS is not a command itself
has switched internal task.

Energy loop GEN calls CALCDOS_WRITE (`gen.f:1149–1153`).
`calcdos_write.f:48–66` reduces DOBS_LTEX over MPI_SUM_R and leaves the output
0. `mpi=True` is justified for standard DOS. Final check
GEN allows TASK=NONE; it cannot be misdeduced that DOS ends up in MPI only because
that there is no string DOS in its list. The actual MPI run has not been performed.

## Defaults and their writing

| Parameter | ASE2SPRKKR |Native behavior without parameter|Assessment|
|---|---|---|---|
| GRID |3, writes|is not a separate early DOS override in INIT_MOD_ENERGY|explicit suitable choice for energy differentiated DOS|
| NE |300, writes|is not independent early DOS override; NETAB has a general initial status 0|explicit resolution option, need to verify convergence|
| ImE |0.01 Ry, writes|Common default 0.01 Ry|the same|
| EMIN |-0.2 Ry, writes| -0.2 Ry |the same|
| EMAX |1.0 Ry, writes|the sentinel is converted to electron EF (`init_mod_energy.f:182`)|different written default, purposely allows even vacant states|
| EMINEV/EMAXEV |singles, omitted from the input|No relative interval|the same; both relative limits shall be transmitted when entering|
| SPLITSS | false, is omitted from the input | false |the same|
| MODE |without keyword, FREL| FREL |the same|
| TAU.NKTAB |Common ASE default 250, writes|joint initialization of k-strip|explicit choice, not guarantee of convergence|
|CPA/STRCONST|Common defaults common definitions|Common native reading|no DOS-specific override|

EMAX=EF would be more appropriate in the role limited to occupied stocks but as default
DOS scan would miss a vacant portion of the spectrum. Therefore, the recorded EMAX=1.0 is left.
It is absolute energy relative to the reference of potential, not 1 Ry above EF**.
For the interval around EF the pair is used EMINEV/EMAXEV. ImE=0.01 Ry corresponds
wide blur; its suitability must be verified by convergence, not confused with
numerically negligible quantity.

Deleted defaults do not lead to any other native energy interval:
EMAX is actually writes here as opposed to some other tasks.

## Documented limitations

### GRID=3 and NE=1

`epath.f:136` sets ESTEP only for NE>1. But then he uses it for scales.
WETAB and at NE=1 (`:143–145`). This is a uninitialized integration step.
The definition rejects this combination. GRID=2 divides NE-1 (`:115`), therefore also
requires at least two points. This is a necessary check, not a complete guarantee appropriate
geometry of rectangular contour for all points and energy ranges.
GRID=0, NE=1 is a valid separate point and remains permitted.

### DOSREP is not common l-resolved DOS

`kkrgen.f:1169–1196` reads in CONTROL only at FREL DOSREP=RLM/CLM/REL
DOSCORSYS=LOC/GLO, WRKAPDOS and WRPOLAR. These options are not exposed; their correctness requires a separate audit.

WRDOSREP stores an energetically differentiated field and writes it at IECURR=NETAB(1)
(`wrdosrep.f:152–185`). Not MPI reduction and part of field DOS with before use
it is not completely zeroed. With energies distributed across MPI ranks, the final rank has only part of the energy grid.
energy. Normal reduction CALCDOS_WRITE **does not include this local field**.
The additional DOSREP output cannot therefore be considered correctly in the energy MPI
assembled only because the standard DOS works. It is a documented problem of this
secondary branches, not standard DATASET.dos.

Comment WRDOSREP on controls of the cubic paramagnetic system does not represent
forced restriction: CHECK is compile-time false (`wrdosrep.f:32–35`).

### Meze auditu

- MODE.FREL and scalar/nonrelativistic inputs are supported by format;
their specific result depends on the potential and is not numerically compared here.
- FULLPOT can be determined by potential; the definition does not add the whole SCF configuration.
- Common ENERGY provides additional contours and SPLITSS. This audit shall be verified by:
Standard GRID=3 scan and required checks, not all combinations
advanced contours, their number of points and numerical scales.
- ImE=0 is formally allowed, but calculation directly on the real axis requires control
poles, k-strips density and convergence.
- There are no heat fluctuations audited, NLCPA, impurity/embedded cluster,
self-energy or other advanced branches. They're not marked off.

### Output prefix

For this output family, native DATSET includes nonempty ADSI and corresponds to Python template `{DATASET_ADSI}`. DATASET shorthand in a filename description must not be confused with unexpanded DATSET0; fixed filenames are identified separately.

## Verification

Current regression modules: `test_dos.py`. They are included in the integrated input-parameter and result_files suite (**1249 passed, 1 skipped**, 2026-11-09). Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
