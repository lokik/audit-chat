# BONDING audits

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

| Task | Schema | Executable | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| BONDING | BONDING | kkrgen | false | true | default |

## Input

INITVAR:505–512 sets TASK=BONDING, switches off reading/writing TAU directory and turns on core/hyperfine. GEN:1013–1025 allows CLUBONDING only for IBZINT=0. TAU.BZINT=CLUSTER sets this mode; INIT_MOD_KMESH:717 switches the normal calculation to REAL-SPACE. CLUBONDING does not read any TASK parameters.

The definition exposes a conservative molecular variant with explicit TAU.MOL. TAU_REAL_SPACE for MOL works with IQ1=1 to IQ2=NQ; its reader also asks for NSHLCLU or CLURAD in this mode. The definition therefore lists NSHLCLU=0. Scratch output requires one cluster copy of each site (TAU_REAL_SPACE:663 and748); a regular periodic cluster does not have to meet this condition. The extension to it is not verified by this definition.

Full relativity is a conservative limitation of the definition: CLUBONDING while reading has a branch IREL<=2 but subsequently uses transformation REL>CLM and double-spin indexing without condition. The functionality of the scalar workflow from this branch cannot be demonstrated. KMROT!=0 and NEPATH!=1 cause silent RETURN (53–54). One energy path is verified by a validator, KMROT depends on the potential.

## Defaults and writing

| Option |Native behaviour|Definitions|
| --- | --- | --- |
| TAU.BZINT |Normal integration does not meet BONDING|Explicitly CLUSTER|
| TAU.MOL |false in absence|Explicitly true|
| TAU.NSHLCLU |if missing, requires CLURAD|Explicitly 0|
| ENERGY.GRID |BONDING does not have its own branch default|explicit 3, energy scan|
| ENERGY.NE |MOD_ENERGY initializes 0; at NEMAX=0 STOP|Explicitly 300|
| ENERGY.ImE | 0,01 Ry |explicitly 0,01 Ry|
| ENERGY.EMIN | -0,2 Ry |explicitly -0,2 Ry|
| ENERGY.EMAX |Sentinel -99,99 Ry is converted to INIT_MOD_ENERGY:182 to EF|explicitly 1 Ry|
| MODE |Commonly FREL| omitted, None |

MOL and NSHLCLU have in general TAU definition an expert mode that defaults does not always writes. V of this module is their writing forced: omitted MOL would not correspond to Python default true and omitted NSHLCLU would result in missing CLURAD reading. The regression test checks the presence of both options.

EMAX is intentionally written; if omitted, EF would be taken over. INIT_MOD_ENERGY:182 maps the upper-bound sentinel to EF. Explicit range corresponds to normal DOS scan ASE2SPRKKR; its physical suitability and convergence are not verified. Native NE=0 not applicable default Native upper limit EF would be more appropriate for analysis limited to occupied states; definition selects scan even above EF. NE must be at least 2 because EPATH GRID=3 uses ESTEP even at NE=1 when it was not assigned.

## Native workflow errors and limitations

1. **Missing auxiliary files:** TAU_DRIVE:70 sets KTAUIJ=0. TAU_REAL_SPACE creates a scratch unit18 only for KTAUIJ>=2 and a unit19 only for KTAUIJ!=0 (657–674,741–766). CLUBONDING still rewinds and reads (78–108). The normal flow thus does not prepare the inputs of this routine. Correction requires intervention in Fortran; the definition file does not solve it.
2. **Central site is firmly 1:** CLUBONDING:228 sets IQ=1 and processes only its types and neighbors. There is no reader TASK.IQ or general selection of the analysis centre.
3. **Strong output unit22:** rows355–358 write without local OPEN and without DATASET name. V Normal Fortran runtime can arise fort.22. The output does not provide an identifier of the type or header of individual rows.
4. **Shell indices after insertion:** when inserting a new radius into DSH, the values DSH are moved but ISHQ already assigned sites are not updated (164–173). The shell listing could be wrong. The calculation of contributions below passes directly JQ, therefore, without further analysis it cannot be argued that this diagnostic error changes all DOS values.
5. **Local matrix dimension:** N is assigned in the previous loop through types according to their IQAT. For subsequent fixed IQ=1, no calculation shall be made. Miscellaneous NKMQ may mean incorrect scope of operations.
6. **MPI:** routines use fixed scratch units, local stored shell tables initialized only at IECURR=1 and output22 without reduction. The module therefore selects mpi=False; parallel workflow has not been documented.

Support is only the definition of input. SPR-KKR 9.7 BONDING cannot be declared functionally task based on this audit.

## Verification

Current regression modules: `test_bonding.py`. They are included in the integrated input-parameter and result_files suite (**1249 passed, 1 skipped**, 2026-11-09). Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
