# SCF — audit ASE2SPRKKR proti SPR-KKR 9.7

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

| Task | Schema | Executable | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| SCF | SCF | kkrscf | true | false | SCF |

## Supported options and validation

1. Public scf.f:427–438 accepts BROYDEN2, ANDERSON and TCHEBY. The task definition exposes those three names; PULAY is not mapped to ANDERSON. Locally modified SPRKKR/9_7/scf.f implements PULAY, but differs from the audited archive.
2. Both native code and validation reject TCHEBY with FULLPOT. FULLPOT can also be activated by potential, which input-only validation itself does not detect.
3. NITER and ISTBRY are positive integers. ITDEPT limited to 3..40: SCFBROYDEN converts values below 3 to 40 and limits values above 40 to capacity. The definition does not pretend that these requirements will be implemented accurately.
4. MIX, MIXOP and TOL must be finite positive values; EFGUESS must be finite. The upper limit MIX was not arbitraryly set at 1 because overrelaxation is not merely a type error.
5. NE is positive; single point GRID=2/3 is rejected due to native integration defects. Other advanced grid combinations are not fully controlled by this definition.
6. EFGUESS is the initial estimate EF in Ry and the omission of the initial search when creating starting potential. It does not prohibit all subsequent updates EF in SCF.

Task-specific SCF and ENERGY copies isolate SCF constraints from other definitions using the common sections. The common section ALG may still contain PULAY in other draw definitions; this is not a confirmation of its native support.

## Defaults and their writing

Normal values NITER=200, MIX=0.2, TOL=1e−5, ISTBRY=1 and ITDEPT=40 are written to the input and correspond to initialisation in scf.f:360–371. BROYDEN2 corresponds to MOD_SCF default and is default of the draw definition. VXC options, their default and metadata have not been modified; MOD_SCF initializes VWN, but specific potential and reader can continue to influence the status input. Native GRID=5/NE=30 follows INIT_MOD_ENERGY, while current Python explicitly writes GRID=5/NE=32; EMIN=−0.2 is written ASE option EMAX is omitted from the input and leaves native EF.

**Omitted MIXOP:** scf.f first initializes SCFMIX=0.2 and SCFMIXOP=SCFMIX, then reads MIX and MIXOP. With MIX=0.5 and no MIXOP keyword, ordinary non-export SCF retains MIXOP=0.2. The Python option is unset and omitted. Set MIXOP explicitly if it should match a changed MIX.

EFGUESS is not recorded without input; native is determined by the presence of the key, not by any replacement energy. FULLPOT=False is omitted from the input; it is not a promise that the full potential will change to ASA. TAU/CPA/STRCONST/SITES/MODE are common sections, here without interference.

## Other restrictions and branches

- Native EXPORT_HARTREE_POTENTIAL sets NITER=1, MIX=1 and skips reading VXC/NITER/MIX. The table of normal defaults is not transferable to this branch.
- ANDERSON mapuje IMIX=7, BROYDEN2 IMIX=4. TCHEBY uses a separate procedure; the algorithm name is not synonymous with the same numerical mix.
- Exhaustion NITER is not evidence of convergence. It is necessary to control the real convergence state, not just the existence of new potential or successful input parsing. This definition does not adjust output parser or calculator lifecycle.
- Native offers further management as SCFRESTART, MIXRHO, SIM, MAGCOUPL, SEMICORE and 2D Layer charge parameters. They are not completed here and their entire workflow is not covered by this audit.
- mpi=True preserved. SCF has parallel solutions and initial search; numerical match MPI/serial in this audit has not been tested.

## Verification

Current regression modules: `test_scf_audit.py`. They are included in the integrated input-parameter and result_files suite (**1249 passed, 1 skipped**, 2026-11-09). Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
