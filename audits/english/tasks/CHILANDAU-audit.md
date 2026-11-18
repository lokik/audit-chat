# CHILANDAU — SPR-KKR 9.7 / ASE2SPRKKR

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

| Task | Schema | Executable | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| CHILANDAU | CHILANDAU | kkrchi | false | true | default |

Landau orbital magnetic susceptibility. The interface exposes the serial periodic ASA/FREL path; MPI and RDTAUFIL reuse have the defects documented below.

## Definitions and validation

Executive kkrchi; native INITVAR recognizes CHILANDAU, RSP calls SUSC_LANDAU. The definition contains CONTROL, TAU, MODE, CPA, STRCONST, ENERGY, TASK and SITES; minimum optional SCF section allows to refuse FULLPOT, which RSP explicitly prohibits.

Required BZINT=POINTS (native IBZINT=2), one GRID=5 contour without SPLITSS and positive energy. SUSC_LANCALC otherwise at NEPATH!=1 or GRID!=5 silently returns without calculation. It returns at KMROT!=0; KMROT is a property native status/potential and does not detect it. FREL is a limitation of the audited procedure: angular and wavefunction routines use relativistic CGC and spinor channels, not found explicit STOP for all other modes. NONMAG is permissible © it is not a spin susceptibility requiring a magnetic base state.

TASK contains CLURAD, NSHLCLU, SELECT, EXCLUDE, QREPONLY and PRINT. SELECT/EXCLUDE are mutually exclusive; radius is finite nonnegative. NSHLCLU including zero takes precedence over radio. The definition does not offer all possible native calculation modifications.

## Defaults

**Written out:** GRID=5, NE=30, ImE=0.01, EMIN=−0.2 and CLURAD=2.2. These are explicit practical choices ASE; number of points and radius require convergence. Native INIT_MOD_TAUIJ_STAR does not have a usable default radius: without CLURAD and NSHLCLU ends STOP Therefore, the published radius is better than its omission, but the value 2.2 is not a physical guarantee of convergence.

**Unwritten:** EMAX is actually left to native EF; QREPONLY=False agrees to MOD_SITES.TREAT_QREP_ONLY=False. NSHLCLU is not forced to zero because its mere presence changes the choice of cluster. PRINT is single: SUSC_LANDAU after initialization sets IPRINT=0 and derives CHIPRINT from it. FULLPOT=False is not written; if the potential itself activates FULLPOT, native RSP it still rejects it. Python default does not switch full-potential potential to ASA itself.

## Specific errors and limitations native sources

1. **MPI: uninitialized field.** SUSC_LANCALC allocates SAVE angular field only at IECURR=1. SUSC_LANDAU divides energy between wounds. The other energy ratio does not have these fields allocated (susc_lancalc.f:66–88; susc_landau.f:193–214).
2. **MPI: reduction after the end of the field.** CHILANLT has a shape (0:NLMAX,NTMAX,2), but MPI_SUM_R starts at CHILANLT(0,1,2) and transmits the length of both blocks. Since the beginning of the second block, only half of the required length (susc_landau.f, final MPI total) remains. This is an independent memory error. mpi=False is therefore a deliberate default definition; explicit enforcement MPI does not correct this error via calculator.
3. ** Continuous outputs.** Condition IECURR.LT.-NETAB(1) in SUSC_LANCALC is always false for positive energy indices. The output of the susceptiblite is printed at each energy, not at the end. V serial run is required to use the last block; earlier blocks are only partial integrals.
4. **Reading TAU branch.** RDTAUFIL opens and reads the text file header, but the energy loop lacks the corresponding reading TAU and the calculation TAU is skipped at RDTAUFIL. This reuse procedure is not offered or certified by this definition.
5. The calculation uses the final sum of the neighbours and the specific Cartesian component. Convergence, geometric range, multiplication at QREPONLY and interpretation for general anisotropics need to be verified. Lower dimensionality and non-standard rotational states are not covered by this audit.

## Verification

Current regression modules: `test_chilandau.py`. They are included in the integrated input-parameter and result_files suite (**1249 passed, 1 skipped**, 2026-11-09). Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
