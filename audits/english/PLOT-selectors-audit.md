# PLOTPOT / PLOTRHO / PLOTSFN — SPR-KKR 9.7

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

| Task | Schema | Executable | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| PLOTPOT | PLOTPOT | kkrgen | false | true | default |
| PLOTRHO | PLOTRHO | kkrgen | false | true | default |
| PLOTSFN | PLOTSFN | kkrgen | false | true | default |

## Native mapping

initvar.f:187–200: PLOTPRS turns on all output flags. If PLOTPRS is not present, PLOTPOT turns on flag1, PLOTRHO flag2, PLOTSFN flag3; any active flag sets internal TASK=PLOTPRS. These selectors can be combined in native input, individual definitions provide separate canonical options.

GEN dispatch gen.f:1461–1463 calls FPPLOTPRS. Routine fpplotprs.f:8–11 and the relevant branches provide:
- PLOTPOT: stored VT/BT, for FULLPOT additionally VNST/BNST;
- PLOTRHO: stored RHOCHR/RHOSPN, for FULLPOT additionally RHO2NS;
- PLOTSFN: stored radial coefficients FLMSF and LMISF shape functions for individual meshes, only FULLPOT (r.392). It is not a generator SFN or directly by 3D cell visualization.

It does not perform SCF or DOS. The existing potential/density/SFN data, quality and availability depends on POTFIL. Native outputs are Grace/XMGR, not images created ASE2SPRKKR.

## Defaults and restrictions

TASK and ADSI have an explicit corresponding selector. GRID0, NE1, ImE0 are explicit startup values, not claim native plot defaults; energy loop for PLOTPRS is disabled. EMIN/EMAX are not exposed, this branch does not need them.

PLOTSFN explicitly writes SCF.FULLPOT=true and rejects its false. Flag does not change ASA potential to correct FULLPOT file; actual format and SFN data must be present in potential. For ASA native PLOTSFN does not quietly create SFN graphs. Omitting SCF.FULLPOT delegates format recognition to the native program. PLOTPOT has ASA and FULLPOT branches. **Standalone PLOTRHO is broken in both:** RHORSQ is allocated only by the potential branch (fpplotprs.f:56–62), but the density branch uses it without that allocation (219–254,280). Combined PLOTPRS avoids this allocation error, but not the shared spin/orbital dataset error (282–287).

GEN forbids the PLOTPRS branches with DMFT/LDAU (gen.f:215–223). Definitions do not offer these MODE options; the presence/validity of stored correlation data potential has not been numerically certified. MPI is disabled, native GEN MPI guard PLOTPRS does not permit. If some types or meshes lack the corresponding data, the input definition alone will not detect this.

Classification: PLOTPOT has a meaningful diagnostic path; PLOTSFN requires a real FULLPOT potential; standalone PLOTRHO has a native writer defect.

## Verification

Current regression modules: `test_plotprs.py`, `test_plot_selector_file_manifests.py`. They are included in the integrated input-parameter and result_files suite (**1249 passed, 1 skipped**, 2026-11-09). Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
