# PHAGEN audit of existing definition

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

| Task | Schema | Executable | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| PHAGEN | PHAGEN | kkrscf | false | false | default |

## What task does

TASK PHAGEN in INITVAR:179–184 switches internal TASK to SCF, sets RUN_PHAGEN and requires KKRSCF. The main program calls PHAGEN instead of normal SCF_DRIVE (kkrscf.f:1392–1400). PHAGEN then starts POTWRPHAGEN and ends regularly.

This is the export of input for external PHAGEN/MsSpec. He's not doing his own phase-shift calculation. PHAGEN_CALCTYPE is in phagen.f:34–36 fixed EXP, parameter reader is commented. It is therefore not offered as a user switch.

POTWRPHAGEN saves original VT and VMTZ, connects to POTFIL extension .only_coulomb, sets EXPORT_HARTREE_POTENTIAL=true, VXC=NONE, NSCFITER=1 and MIX=1 and performs SCF_DRIVE. It then exports radial mesh, r*V_Coulomb, r^2*rho and r*V_total. It's not a pure rewrite format without calculation. The total potential is from the original input, Coulomb potential and density from the subsequent step; interpretation must respect this distinction.

## Definition and validation

- The common MODE section selects relativity and permits validation of explicit DMFT/LDA+U.
- Validation rejects SCF.FULLPOT, SCF.VXC other than VWN, MODE.OP=DMFT/LDA+U, nonpositive NE and split contours.
- The executable is kkrscf, mpi=False.
- Export type is fixed to EXP; there is no TASK reader for phase-shift energies or the output filename.

**A genuinely converged potential is required:** export sets VXC=NONE and SCF requires SCFSTATUS=CONVERGED (scf.f:443–447). A deliberately unconverged one-iteration SCF is not valid preparation. Relabelling its convergence status is not a repair.

Checks NCPA, USENLCPA, NT and THERMAL_VIBRA_FLUCT depend on the potential and working status of the main programme. Native POTWRPHAGEN:49–81 requires NT=1, none CPA/NLCPA, no thermal fluctuations, ASA, VWN and KKRSCF. These potential properties do not know the InputParameters alone. FULLPOT from potential can also be active at SCF.FULLPOT=false; final check remains native.

## Defaults and effectiveness

| Option | SPR-KKR |Definitions|
| --- | --- | --- |
| PHAGEN_CALCTYPE |Fixed EXP|not offered|
| ENERGY.GRID |5 because internal task SCF|explicit5, left|
| ENERGY.NE |30 for SCF|explicit32, left|
| ENERGY.ImE |in general0,01 Ry|explicit0 Ry, left|
| ENERGY.EMIN | -0,2 Ry |explicitly -0,2 Ry|
| ENERGY.EMAX |EF for SCF contour|omitted, taken over EF|
| SCF.VXC |Export accepts only VWN|VWN, left|
|SCF.NITER / MIX at export step| vynuceno1 / 1 |General200 / 0,2 are still recorded but exports overwrite them|
| MODE |Commonly FREL|None, omitted; explicit NREL/SREL are not prohibited in the export routine|

NE32 versus30 is the explicit difference. Native30 would slightly reduce work, but without convergence comparison there is no reason to change existing32; it is not a non-compliance omitted default. ImE0 versus0,01 is also explicit. GRID5 in EPATH uses its own complex arc, not this parameter as a single constant height; therefore the difference does not prove a different result.

SCF.NITER and MIX do not represent the number of iterations and mixing PHAGEN export step. These values are not reloaded at EXPORT_HARTREE_POTENTIAL (scf.f:361–389). They are not only written in Python for cosmetic conformity: the audit accurately distinguishes input of common defaults and their native overwrite. It cannot be argued that their change will be obtained by the converted Coulomb SCF export.

## Major caveats in source

1. **Only one type and without disorder:** NT must be1. It's not a general converter of multi-type potential.
2. **Spin-polarized export is not implemented:** header and writer declare non-spin-polarizedASA and write VT/RHOCHR, not exchange field BT/spin density. Native safeguard NONMAG is missing. A successful run with magnetic potential thus does not prove itself maintaining spin information.
3. **Incorrect indexing of the radial mesh:** Writer detects IM=IMT(IT), but in itself WRITE uses R(IR,IT), not R(IR,IM) (potwrphagen.f:158–160). With a different mesh index, the wrong radial axis and its multiples can be exported. For NT=1 and IMT(1)=1 this particular problem is hidden.
4. **The density and potentials are not all from the same state:** original VTOTAL is copied before Hartree SCF step, RHOCHR is written to the input until after. It is not one universal self-consistant set of columns.
5. **MPI branch is dangerous:** SCF_DRIVE and some barriers are inside MPI_ID=0, so other processes do not participate in this call. DEALLOCATE(VTOTAL) is, on the other hand, outside this branch, although the allocation took place only on the process0. Existing mpi=False is correct; only barriers in routine are not proof support MPI.
6. **The relativity limit is commented:** safeguard IREL!=0 has been disabled. definition does not invent a ban on all relativistic regimes; their correctness for subsequent external PHAGEN is not numerically confirmed.
7. **Outputs derive from the original POTFIL:** POTFIL_ex_sprkkr and **POTFIL.only_coulomb_new**. POTWRPHAGEN adds .only_coulomb, then SCF_DRIVE adds _new (scf_drive.f:210–213). CONTROL.DATASET does not isolate these names. PHAGEN terminates the process after changing global POTFIL and VXC.

Task isn't off placeholder. V supported by simple ASA/VWN/NT1/mesh1 case has a substantive export purpose but these conditions and restrictions must be given in the final distribution tasks. The integration test checks both files using a converged Au potential; it does not compare external PHAGEN/MsSpec results or certify every exported column.

## Verification

Current regression modules: `test_phagen.py`. They are included in the integrated input-parameter and result_files suite (**1249 passed, 1 skipped**, 2026-11-09). Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. The native `TestCalculator::test_phagen` integration test also passed using converged `Au_single_type.pot` and checked both exported files. The targeted suite had **32 passed**. It used existing local `SPRKKR/9_7` binaries, not a fresh build of the public archive; external PHAGEN/MsSpec was not tested.
