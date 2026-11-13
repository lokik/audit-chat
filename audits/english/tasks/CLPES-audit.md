# CLPES — SPR-KKR 9.7 / ASE2SPRKKR

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

| Task | Schema | Executable | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| CLPES | CLPES | kkrspec | false | true | ARPES |

CLPES is an independent schema. It is available through both `InputParameters.create("CLPES")` and `for_task("CLPES")`, and appears in `task_registry.names()` and canonical schema discovery. The registry lists 55 task identities; task identity remains distinct from the shared JXC/DMI and BSFEK/BSFKK schemas.

Core-level photoemission and photoelectron diffraction through KKRSPEC, IBLOCH=3 (`spec_rslab.f:172–173,226,239–241,862`; driver `spec_clpes.f`). The interface limits the audited workflow to serial execution with fixed electron geometry.

## Parameters and defaults

SPEC_CL is the actual name of the section (spec_inputs.f:1877–1921). Implementation IT (explicitly 1), CL (explicitly 2P), IE_ICST (default 1). CL is always written: without the present CL reader leaves 1S, although SECTION_SET_STRING states fallback 2P. Optional CE is not exposed in this definition; IT determines potential type and native selection includes also same atomic count Z (spec_clpes.f:213–216), not isolated site selection.

NE=300 written, single mesh without SPLITSS. Driver rewrites the energy window in the range of core energy extended by +/-3 eV (spec_clpes.f:301–306); EMIN/EMAX are not user spectral limits. NE=1 selected IE_ICST and validated 4*l+2. Core energy is related to EF (r.224).

EPHOT=1000 eV is explicit; the user must check the threshold of the selected core shell. It does not guarantee access to every core level. Explicit EWORK_EV=4.2 eV, IMV_INI_EV=.05 eV, IMV_FIN_EV=2 eV, NL=4, N_LAYER=50 and NLAT_G_VEC=57 are not claimed as native defaults. CRYS_VECS=False is omitted and preserves USE_CRY_PRIM_VECS=true. In 3D surface construction, presence of CRYS_VECS sets USE_CRY_PRIM_VECS=false (spec_rslab.f:180–181). ARPES, AIPES, BAND, SPLEED and CLPES share this definition; the native reader does not read singular CRYS_VEC. Omitted NPOL corresponds to native 1; 0 and 3 trigger STOP and validation rejects them.

## Major caveats

- NCPA>0 vede k STOP (spec_clpes.f:157). Definition does not verify occupation of potential and cannot guarantee the ordered system.
- The audited range is a fixed final electron geometry. Angular and moment scans are rejected; old external input INPVER=0 their control goes around.
- NE=1 uses ECOR/XNORM by loop of atoms (spec_clpes.f:310–319); for multiple types/site may correspond to the last processed set, not the desired type. This branch cannot be considered to be generally verified.
- The selected shell must exist as core shell, IT must exist and EPHOT must exceed the threshold. Python validation without potential will not guarantee these conditions.
- MPI has not been certified, automatic use is disabled; this is not a claim that native MPI is missing.

## Verification

Current regression modules: `test_clpes.py`. They are included in the integrated input-parameter and result_files suite (**1249 passed, 1 skipped**, 2026-11-09). Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
