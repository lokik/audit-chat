# JXC audit of existing definition

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

| Task | Schema | Executable | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| JXC | JXC | kkrgen | true | false | JXC |

## Scope and validation

The definition supports JXC. INITVAR reads the keyword JXC, sets CALC_JXC and internal task E-INTEGRAL. GEN:930–990 distinguishes SP-SREL (IREL=2) with XCPLJ0/XCPLJIJ and FREL (IREL=3) with XCPLTENSOR. Validation rejects NREL/SREL and accepts SP-SREL; regression tests cover this distinction.

The definition exposes native TASK.NSHLCLU, SELECT, EXCLUDE and QREPONLY. INIT_MOD_TAUIJ_STAR:125–161 reads lists of chemical symbols, rejects their simultaneous use and gives shell count preference over CLURAD when present NSHLCLU. QREPONLY reads INITVAR inside E-INTEGRAL branches; affects the list of central sites in INIT_MOD_TAUIJ_STAR. CLURAD is in units ALAT, not angstroms.

Final validation rejects NREL/SREL/NONMAG, not positive NE, more energy routes, unaudited integration and current SELECT/EXCLUDE. A warning_condition also covers mode and NONMAG. For the standard periodic calculation, BZINT=POINTS is explicit. GEN:358–386 for off-diagonal TAUIJ requires IBZINT=2.

The range of this definition is normal periodic JXC to GRID5. In the source there are other branches embedded-cluster, occupation-weighted GRID3 (WGT_OCC), MULTISPIN, SLC_JXC and SLC_TRQ. They do not have a complete audit; therefore, the validator is not a claim that SPR-KKR generally never knows GRID3 or cluster. A separate step is needed to access the expanded branches.

## Defaults and their writing

| Option | SPR-KKR |Definitions|
| --- | --- | --- |
| TASK.CLURAD |without NSHLCLU a value is required; a generic default is not|explicit2,2 ALAT, left|
| NSHLCLU |selects CLURAD in absence| None, omitted |
| SELECT/EXCLUDE |blank lists| None, omitted |
| QREPONLY |false in absence| false, omitted |
| ENERGY.GRID |5 for E-INTEGRAL|explicit5, left|
| ENERGY.NE |30 for E-INTEGRAL|currently explicit30|
| ENERGY.ImE |generally0.01 Ry|currently explicit0.01 Ry|
| ENERGY.EMIN | -0,2 Ry |explicit -0,2 Ry, left|
| ENERGY.EMAX |General Sentinel replacement for EF|omitted, taken over EF|
| MODE |Commonly FREL|None, omitted; SP-SREL with writes when entering|
| TAU.BZINT |general choice depends on workflow|explicitly POINTS for documented off-diagonal TAU flow|

JXC writes NE=30 and ImE=0.01 Ry. DMI shares the same schema with a distinct task identity. GRID5 constructs its own complex contour. Radius 2.2 does not establish convergence; NE, lower bound, k-mesh and pair cutoff require numerical checks.

## Serious ALE in SPR-KKR

1. **Out rotation according to symmetry at FREL:** XCPLTENSOR:312 has JQ.EQ.IQREPQ(JQ) .OR.1.EQ.1. Thus, the second branch with rotation ME_ROT_MREG will never run. For non-representative neighboring site can be used matrix in the wrong framework. QREPONLY restricts central site, does not automatically solve all neighbors. This is not what I call breaking up every single-atomic collinear run; it is an important mistake in the general tenor workflow.
2. **FREL has potential conditions:** NONMAG=false, KMROT=0, NKM=NKMMAX and representative IQAT(1,IT) are directly controlled XCPLTENSOR:74–80,155. Python knows explicit NONMAG, other conditions require real potential. Cannot enter QREPONLY to certify all its properties.
3. **More routes are not processed in general:** routines use ETAB/WETAB first routes and have NEPATH=1 guard. Later enforcement E-INTEGRAL does not change this condition to support split contour.
4. **Cluster SP-SREL is not a normal periodic flow:** CLUXCPLJIJ reads scratch unit19 but TAU_DRIVE normally calls TAU_REAL_SPACE with KTAUIJ=0 which shuts down its export. Without an alternative unit preparation audit19, it is not right to promise this workflow only by switching BZINT=CLUSTER. FREL distinguishes embedded-cluster separately.
5. **Higher spin and spin-lattice branches have their own interface:** GEN reads MULTISPIN/SLC_JXC/SLC_TRQ and selects other routines. This definition does not represent these keywords as fully supported by simply adding a name.

Unlike BSDOSIJ uses XCPLTENSOR for JC correct offset BAR_M_JT_JQ(1,1,JC) and output J_ITJT has consistent pair indexing. Errors from BSDOSIJ cannot be mechanically attributed JXC only due to a similar source.

## MPI and output

mpi=True is supported by the source: J_IJINT is zeroed at the first call, MPI_SUM_C performs reduction and GEN:1193–1241 has an additional finishing call. SP-SREL has its own XCPLJIJ reduction. Numerical serial/MPI match has not been measured.

FREL writes DATASET_XCPLTEN_Jij.dat, tensor in meV (DIMAG(J_IJINT)/pi*RY_EV*1000), and other outputs; SP-SREL has a different structure and scalar Jij. This audit does not change the existing output parser nor the Hamiltonian Convention/factors taken over by other software.

### Output prefix

For this output family, native DATSET includes nonempty ADSI and corresponds to Python template `{DATASET_ADSI}`. DATASET shorthand in a filename description must not be confused with unexpanded DATSET0; fixed filenames are identified separately.

## Verification

Current regression modules: `test_jxc.py`, `test_jxc_definition.py`, `test_exchange_file_manifests.py`. They are included in the integrated input-parameter and result_files suite (**1249 passed, 1 skipped**, 2026-11-09). Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
