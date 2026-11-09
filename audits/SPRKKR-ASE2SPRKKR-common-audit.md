# Consolidated audit of SPR-KKR 9.7 and ASE2SPRKKR

## Source baseline and scope

Current as of **9 November 2026**. The authority is executable SPR-KKR code: input readers, dispatch, allocations, computation and output routines. A task name, comment or passing Python test alone does not establish the physical meaning of a result.

The native baseline is the public archive **PUB9.7_260420_18h13.tar.gz**, SHA-256 `65c2f6e178fc127946ff8309cf0000c7ed9d64f51d8bec1e03c057147c872ff4`, extracted locally under `SPRKKR/9.7/`. It contains 770 Fortran files. All 110 source files checked for this update match the archive byte for byte; source line numbers refer to that baseline.

The local development tree `SPRKKR/9_7` is different: its modified `scf.f` also implements PULAY. The PULAY rejection applies to the public archive. Existing binaries in `9_7` were not rebuilt from the archive for this verification; a successful local run does not certify the public build.

The Python interface was checked at ASE2SPRKKR **49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6** (`develop`); PLOT2D help specifies Cartesian ALAT units. Definitions are integrated together; separate patches are not installation instructions. Section 7 records the numerical verification scope.

## Scope and meaning of the categories

The audit covers the tasks, aliases, and selectors reviewed so far in INITVAR/GEN/RSP/KKRSPEC, including the complete TUTORIAL task. It is not a comprehensive audit of every internal algorithm, correlation method, or experimental combination of switches.

| Category |Meaning|
|---|---|
| **Fully meaningful** | The reachable implementation matches the exposed purpose within the stated scope; no fundamental obstacle to that workflow was found. Ordinary formalism restrictions or convergence requirements do not by themselves constitute major caveats. |
| **Caveats — 2A: mostly functional** | The main audited path is meaningful; it requires a suitable state or input, or restriction to a working branch. |
| **Caveats — 2B: mostly nonfunctional for the stated purpose** | The main declared or explicitly identified branch is unfinished, produces defective data, or has an unsubstantiated physical interpretation. Supplying the correct state alone is insufficient. |
| **Broken** | A demonstrated defect in reachability, initialization, indexing, or data flow blocks the normal purpose or corrupts the result. An input definition does not fix it. |
| **Disabled** | The relevant path is explicitly stopped or replaced with an unavailable routine. The combined list appears at the very end. |

The classification applies to the supplied SPR-KKR revision. For a task with distinct branches, the scope is stated explicitly. Valid input, a passing pytest run, or normal program termination does not establish a correct physical result.

## 1. These tasks are fully meaningful

|Task / Branch|Purpose and audited scope|Important support boundaries|
|---|---|---|
| **SCF** |Self-consistency calculation; supported options BROYDEN2, ANDERSON, TCHEBY.|PULAY is not valid option of this native reader. TCHEBY is not for FULLPOT. The export and special correlation workflow has not been generally certified.|
| **DOS** |Standard DOS over generic GEN loop; native internal TASK is NONE.|GRID, numbers and windows shall correspond to the required spectrum. Advanced DOSREP branches have specific MPI risks and are not automatically covered.|
| **BSF** |Bloch spectral function; energy scan EK and single energy cut KK.|KK is not automatically to EF. Existing EK default EMAX=-1 Ry is a suspect explicit interval; see defaults below. All path mapping and 2D transformations have not been fully verified.|
| **EKREL** |E(k) and Fermi surface for arranged systems.|CPA is rejected natively. TASK.EMIN/EMAX/NE controls its own search, not ENERGY. The preset to the path depends on the grid; ESLICE=EF may be outside the search interval.|
| **WFPLOT** |Diagnostic charts of wave functions BAND/CORE in ASA, FREL.|It uses the first energy point; availability shells and atomic type must determine potential. FULLPOT and those correlations are not supported.|
| **CLXPS** |Core-level XPS in ASA/FREL with implemented ADA branches.|Only implemented forms ME; the final state must be above EF. Geometry has actual sufixes 0 or continuous 1..10. Capacity ENERGY.NE must be sufficient to core states.|
| **CHI** |Audited static response in ASA/FREL, POINTS, serial.|NCHIITER is natively always 1; it is not an iterative solve controlled by this option. Local CHIORBPOL is not identical to general MODE.OP.|
| **POSANITAU** |Positron annihilation integration through own TAU path in ASA/FREL.|The suitability of the energy limit EF and the zero partial speed shall be assessed; 1/RATET is not protected from zero. This is not a substitute for preparation POSANI.|
| **JORB** |Orbital current density, FREL, serial.|The potential must provide the necessary angular representation. Auxiliary fort.* are not a clear main output; MPI completion does not work.|
| **PLOTPOT** |Selective chart of saved potential, in ASA and FULLPOT.|Does not perform SCF. The data and their mesh must be present in potential.|
| **PLOTSFN** |Graph of stored radial coefficients shape functions in FULLPOT.|Does not create SFN or 3D cell shape; FULLPOT flag does not convert ASA potential. In ASA, the SFN plot is not produced.|
| **TUTORIAL: YLM, JLX, JLX-ASYMPT** |Mathematical examples of harmonics, Bessel functions and asymptotics.|L/M and limits are validated; even mathematical branches undergo common initialization and need usable input environment.|
| **TUTORIAL: DOS-FEG, DOS-SS, DOS-CLU, DOS-BULK, GEN** |Samples DOS / normal GEN workflow in the corresponding model.|FEG and SS are single-site STANDARD. CLU requires actual cluster geometry and CLURAD/NSHL; the name of tutorial alone does not create it. BULK requires periodic integration.|

For BSF needs to be corrected or intentionally confirmed default window before use without its own parameters. U POSANITAU includes an unprotected zero partial contribution among specific defective cases; does not mean the fact that each non-zero total annihilation integration is broken.

## 2. Tasks with caveats — classification by practical usability

### 2A. Mostly functional: require the correct state, input, or restriction to a working branch

V for this group the main audited calculation has a meaningful reachable path. Typically, there is a need for suitable potential, compatible TAU/DOS, correct magnetic geometry or use of only implemented formalism. It's not the same as the unfinished main solver. Some tasks also have defective secondary output or alternative branch; these are explicitly set aside to 2B.

Classification to 2A is not confirmation of the numerical correctness of all combinations. ©The correct status here means the conditions of a specific physical workflow, not any tuning of parameters that only hides a programming error.

| Task / rodina |Condition of use or restriction|Practical Conclusion|
|---|---|---|
| **MEPLOT** |A photon scan with more electron energy rewrites curves. The presence NEPHOT=1 itself switches the reader to the range branch; SEFIN receives the initial ERYD.|The definition omits the single-photon NEPHOT default and rejects simultaneous photon and electron scans. Meaning SEFIN for the final state still requires physical verification.|
| **MECHECK** |NMEFORM is overwritten, the next test is complete-time disabled and the summary loop has an incorrect upper limit.|Sub-total ADA/NAB comparisons make sense; all checks described cannot be promised GRV.|
| **SOCPAR** |Test ABS(... )<0 never reverses the mole. EXCME is missing Ry→eV conversion, although the axis says eV.|SOC part and exchange part must be interpreted separately. Exchange output has documented non-compliance of units; correct Fortran, not input default.|
| **PSHIFT** |Phase shifts exist, but resonance fracturing may be used as an unfound resonance; interpolation does not control actual pi/2 exceedance.|Phases are not automatically broken; derived resonance numbers are not reliable without repair. NR non-uniform cutoffs and high relativistic legends have other risks.|
| **PLOTPRS** |Common task bypasses the error of the separate PLOTRHO, but the spin and orbital curve points to the same dataset and the panel limits ignore the orbital density.|Potential, charge and SFN outputs make sense; resolution of magnetic density in the graph requires correction of Writer.|
| **STONER** |Kernel is fixed VWN; small DOS is replaced by nonphysical unit contributions; some normalization is separated without protection.|The result is not automatically the kernel used SCF of the functional. It needs suitable cutoff and nonzero standardization variables.|
| **VBPES, VBUPS, VBXPS** |GEN implementation is common; VBUPS does not switch to UV photon. DEFEP0/EWORK are only diagnostic and custom geometry can be overwritten. TAU and caches are not verified.|UV specify the energy. This family is not identical to KKRSPEC ARPES/AIPES.|
| **APS** |Reads old DOS format without capacity control. Broadening expands more fields and depends on absolute energy.|Without Broadening, there is a compatible path DOS. GAMMA is not normal constant Lorentz width; positive GAMMA requires correction or specific verification.|
| **AES** |Only NQ=1; cache ME/MEi does not control origin. NE of TAU may exceed the allocation. F shells may exceed the table of names.|The definition excludes f shells and NEME<3; compatible uniform growing TAU mesh and pure cache remain necessary.|
| **FMAG** |Spin/orbital form normalization divides by moment at q=0 without protection. Network q is fixed.|Zero moments can give NaN/Inf even in a otherwise correct system. The normalized curves cannot be recounted as the original moments.|
| **JALF** |The main current file has a purpose, but the auxiliary diagnostic mixes types and meshes; at NL<3 exceeds the field.|The definition protects explicit cutoff. Vector potential does not count as standard, although the code contains an unachieved branch.|
| **TORQUE** |Requires rotated moments and the appropriate torque axis; an unrotated potential can cause STOP. Mixed occupation uses the first type.|The validation of input without potential cannot guarantee an appropriate magnetic geometry or meaning WGT_OCC.|
| **XAS** |The static branch has a purpose but accepted ME names do not show three independent Gauges; non-dipole formalism depends on ASA/FULLPOT. The limits can natively overwrite EF.|Serial interface (mpi=False), configurable EMAX=4 Ry; MECHECK is a separate task. TAU compatibility and effective bounds require checking actual data.|
| **PHAGEN** | Requires a converged potential (SCFSTATUS=CONVERGED), ASA/VWN, NT=1 and no disorder. Exports original total potential with subsequent Coulomb-step density; spin/mesh caveats remain. | Meaningful for a simple NT1/mesh1 case. Actual files: POTFIL_ex_sprkkr and POTFIL.only_coulomb_new. Not a phase-shift solver. |
| **JXC** |FREL symmetrical rotation is eliminated by condition OR 1.EQ.1. Cluster SP-SREL path normally does not have prepared scratch files.|Simple periodic collinear workflow makes sense; tensor at equivalent locations and cluster are generally not reliable.|
| **DMI** |Dz is calculated differently from Dx/Dy. The limitation of representatives and couples does not automatically complement the multipleness of the complete cell.|Three columns are not automatically equivalent components of one procedure. Micromagnetic totals require inspection of frames, multiples and radius.|
| **COMPTON** |Native SPLITSS is read from two sections. EMD2D with nonzero PNMAX scales the longitudinal vector twice; even shifted meshes loses the extreme points.|Definition writes both SPLITSS and excludes defective EMD2D/shift combinations. The basic compatible workflow is not marked as broken by this.|
| **POSANI** |Depends on compatible data POSANIPREP; file TAUQQ has NQ/NT error. Core contribution is limited and core lifetime parameterization has a specific fixed shape.|It's not a standalone universal positron calculation. Before using prepared data, check their structure and origin.|
| **POSANIPREP** |TAUQQwriter/reader iterates NT instead of NQ: can skip site or cross the field. Old rho cache can skip export waves.|Meaningful preparation in a compatible simple case; general NQ≠NT workflow is broken.|
| **GILBERT** |GEFF divides by spin/total moments before checking them. Heat mesh and standardisation need specific physical data.|Basic Damping has a sense of suitable magnetic system; compensated or zero moments are defective cases.|
| **CHILANDAU** |Incorrect finishing condition prints continuous integrals; RDTAUFIL does not prepare the necessary energy content. MPI has allocation/reduction errors.|Serial calculation to assess by the last complete block. Reuse TAU and MPI are not functional general routes.|
| **SIGMA** |Fermi-surface and sea are different branches; the sea is not automatically included. ImE and fixed names sigmaXXXXX.dat affect use.|Do not present the surface itself default as complete all posts. The scope of the definition is audited ASA/FREL/POINTS path.|
| **OPTABS** |NOMEGA is the allocated capacity but the loop uses NEB. TEMPEL<=1 leaves KBT unprepared. NEA is overwritten and non-zero OMEGA1 violates audited pairing.|The definition requires TEMPEL>1, NOMEGA>=NEB and limits the audited window. Task is OPTICS-ABS, not generic OPTICS.|
| **CHIXAS** |It needs external NEF/CHISPN/CHIVV data with the right units, continuous numbering and potential dimensions.|No running with universal physical defaults. Test fixation does not replace measured/calculated input data.|
| **ARPES** |Legacy TASK aliases and native IMV_INI in Ry writing the wrong damping variable. CRYS_VECS has presence-sensitive semantics.|Use custom AIPES/BAND/SPLEED definition. Warning when validating/ storing an old alias is DeprecationWarning and Python can filter it.|
| **SPLEED** |Shares KKRSPEC but has a different branch and an absolute incident EV. Exhibited is conservative solid geometric workflow.|Not ARPES with a different label. The Damping/unit and surface potential structure shall be appropriate.|
| **AIPES** |IBLOCH=2 sets local/incoherent calculation of all atoms; it is not an ordinary numerical angular integration ARPES.|The difference against ARPES is in the model matrix elements and interference, not just at the output angle.|
| **BAND** |Complex bands use -i LOG(EIG), cutoff is in Im(kz). When failed EIGA/B there is no interruption and may come LOG(0); column labels vary according to geometry.|There is no substitution EKREL for ordinary E(k) Appropriate eigenvalue cutoff is required, column interpretation and solver failure repair.|
| **CLPES** |Core window rewrites the assigned bounds. CPA and some polarization stops running; ECOR in NE=1 path may come from the wrong last type.|Serial fixed geometry and suitable shell have meaning; explicit photon 1000 eV is not a guarantee of crossing the threshold.|
| **TUTORIAL: CHR-POT** |It uses common PLOTPRS, including magnetic graph problems.|It's a graph of read data, not a new SCF calculation.|
| **TUTORIAL: POT-DECOMP, POT-XC** |ASA; NONMAG remains sticky between types. POT-DECOMP breaks down the state with the original Vxc, the shown new Vxc does not have to correspond to the original.|POT-XC compares functionals at the same density, not four SCF solutions. PBE is not supported original XC in the branch POT-DECOMP.|
| **TUTORIAL: RWF-L, RWF-E** |It uses scalar reader, not generic FREL. Normalization has no protection from zero; matching fence RWF-L reads blank data.|Radial data have a diagnostic sense in scalar/ASA range, matching curve RWF-L is broken.|
| **TUTORIAL: CORE-ALL** |Only the first type, zero exchange field and selected output components. Without core states, variables uninitialized may remain; in another mesh index, the error is JRWS.|ALL does not mean all types and full spinor. Suitable first type/mesh/core conditions are not guaranteed by input.|

**Examples of difference:** TORQUE with unrecorded potential is not proof of unfinished torque solver; CHIXAS without proper external susceptibility does not have necessary physical inputs. By contrast, faulty tensor rotations JXC or uninitialized spectral fields are not a matter of appropriate baseline.

### 2B. Mostly nonfunctional for the stated purpose: defects, an unfinished main branch, or unreliable results

I do not consider this group to be normally applicable only after the correct entry has been completed. It is necessary to correct the implementation or verification that the calculation corresponds to the declared purpose at all. U mixed tasks the conclusion refers only to the specified branch; their functional basis remains in 2A.

For XES and TDOBS, the issue is whether a separate implementation of the declared functionality is established. Source inspection reveals this concern, but without a numerical reference comparison it does not prove that every output is wrong. For the other programming defects listed below, a specific cause is demonstrated.

| Task / specific branch | Why correct input is insufficient | Practical conclusion |
|---|---|---|
| **PLOT2DPR / PLOT2DPOT / PLOT2DRHO: FULLPOT** |Unconditional experimental PLY branch writes both groups and end run before older regular gridwriter.|The result is not the promised selective regular tables. NA/NB does not control all PLY sampling.|
| **PLOT2DPR / PLOT2DPOT: ASA** |ASA Writer draws potential; whole PLOT2DPR does not deliver the declared density part.|Potential PLY are meaningful but the common name is not a guarantee of both potential and density.|
| **XES** |RXES goes into the common absorption-rate branch; the output is marked XAS and uses the same name convention.|The result cannot be identified as a verified emission spectrum without reference physical comparison. It's not a setholder off.|
| **TDOBS** |Native path leads to generic DOS/observable loops; special time-related observational implementation has not been documented.|The name is not a guarantee of separate physical task. Cache fallback and subsequent rewrite flags must be understood by GEN.|
|**JXC: common FREL tensor with symmetrical equivalent neighbors; cluster SP-SREL path**|Rotation is disabled OR 1.EQ.1; cluster scratch files are not prepared in normal flow.|A simple periodic case can work, a general tensor/cluster workflow needs repair.|
|**SOCPAR: exchange output**|Unable to convert Ry→eV and test sign ABS(... )<0.|Values with eV label cannot be directly used; functional SOC part does not automatically disappear.|
|**PSHIFT: resonance fracturing**|The sum is used as well as the unearthed second channel and the interpolation does not control the bordered transition pi/2.|Phase shifts make sense, the derived splitting values are generally not reliable.|
|**PLOTPRS /TUTORIAL CHR-POT: spin and orbital curve resolution**|Both curves are entered into the same dataset; limits do not take into account the orbital curve.|Potential and charge graphs are applicable; faulty magnetic graph is the problem of Writer.|
|**MECHECK: promised full set of checks and summary**|Another test is compile-time disabled, NMEFORM is overwritten and the cumulative loop is not performed.|Sub-ADA/NAB control works; you will not switch on the full declared diagnostic by input.|
|**MEPLOT: current photon and electron scan**|The curves are rewritten through the wrong dimension.|This combination gives incomplete output; the definition rejects it. Doubt about SEFIN is a particularly unverified interpretation.|
| **APS: broadening** |Expanded networks are not adequately controlled against capacities and the width depends on absolute energy zero.|The standard Lorentz breaking is not verified; without Broadening it can be basic APS applicable.|
|**POSANIPREP /POSANI: general data path NQ≠NT**|Writer/reader uses the number of types instead of the number of places.|Missing or outside the range of read data will not solve the general input validator.|
|**CHILANDAU: MPI and RDTAUFIL use**|Allocation/reduction errors and unprepared energy content.|Serial calculation without this reuse path is a different, more useful branch.|
|**COMPTON: EMD2D with non-zero PNMAX; some even shifted meshes**|Double scaling shifts the cut to another location; the construction loses the extreme points.|The result does not correspond to the declared cut/meshe. The definition protects entry restrictions.|
|**BAND: default EIGA/B**|Safe interruption is missing and continued until possible LOG(0).|Successful eigenvalue branch can work; solver failure must treat native code.|
|**CLPES: NE=1 with multiple types**|ECOR may remain from the last type instead of the selected type.|In this case, the energy axis may be incorrect; general solid geometry is not a problem itself.|
| **TUTORIAL: matching RWF-L** |Matching curve uses blank data.|Radial solutions and matching diagnostics are not the same; the graph needs repair.|
|**TUTORIAL: CORE-ALL outside the first suitable type/mesh/core case**|Incorrect JRWS index and unprepared variables in missing core states.|It is not generally completed © all core states of all types.|
|**TUTORIAL: POT-DECOMP/POT-XC for mixed magnetic types**|NONMAG is not renewed between types.|The resolution of output may be incorrect; comparison XC on fixed density is otherwise a deliberate example.|

Unprotected zero normalization FMAG/GILBERT/POSANITAU and cache or capacity errors AES are not proof that the entire main solver is not ready. These are specific faulty cases: there is a meaningful way in which conditions are met, the result cannot be trusted in their breach. The range is given in 2A and in the following error register.

Section 3 leaves separately tasks with directly documented breakage of the normal main workflow and overview of defective cases. The categories 2B and 3 are therefore not two competing conclusions: 2B specifies the applicability of the former caveats, sections 3 presents specific evidence and necessary corrections.

## 3. These tasks or specific branches are broken

### The intended normal workflow as a whole

| Task |Imported Problem|Evidence in the Source|Necessary correction|
|---|---|---|---|
| **FSCAT** |The error index of the harmonics with excess +L may exceed the field; the phase is calculated before treatment with a zero amplitude.| fscat.f:131,207–209; calc_chplm.f:155–162 |Fix mapping LM, filter inadmissible/zero CG folders and protect the phase from 0/0.|
| **T1** |Signed kappas are used as positive numbers m_j; negative channels fall out and indexing the whole base is broken down.| relt1.f:215–245 |Fix kappa degeneration/index and verify relaxation speeds. PRINT/RUNELOOP workaround only solves other initialization errors.|
| **NRAES** |IAES is zeroed by a shorter range than the allocated field; the convolution is added to the unitialized areas of different channels.| nraes.f:99,814 |Zero whole (2*NEMAX-1)*[NLMAX+1)*3 and add checks DOS metadata.|
| **BSDOSIJ** |J_ITJT is stored with energy indices into pair dimensions and read in a different order. It may cross the field and issue incorrect data.| bsdosij.f:319–349,434–436 |Unite dimensions/indexs, component offsets, rotations and final spectral output.|
| **BONDING** |CLUBONDING reads scratch 18/19 which does not prepare normal TAU_DRIVE with KTAUIJ=0.| tau_drive.f:70; tau_real_space.f:657–674,741–766; clubonding.f:78–108 |Connect actual exports TAUIJ to the consumer; then fix the centre, dimensions and output.|
| **NUCSSCPL** | Arrays are deallocated before their first allocation; the whole ME array is zeroed again for every type, and the mesh index is overridden to 1. | nucsscpl.f:115–133,341,366–368 | Fix array lifetimes, type-local initialization, and the mesh; NE=1 is not a fix for an integration task. |
| **XRS** |SIG(NEMAX+50,...) is zeroed only as NEMAX; the later sum uses the full range.| xrayspec.f:636–639 |Zero the entire field, correct the additional mesh and check the third integration capacity.|
| **BIS** |INITVAR recognizes it and GEN calls VBPES, but the routine requires TASK(3:5)=PES that BIS does not meet.| initvar.f:264; gen.f:1445; vbpes.f:174 |Implement the corresponding inverse-photoemission path. Alias BIS→VBPES would only mask the error and change meaning.|
| **SIGMA-XAS** |The calculation branch exists but the standard parser does not set internal TASK=SIGMA-XAS. Keyword SIGMA is not prefix recognition.|rsp.f:941; sigme.f:499; initvar.f and SECTION_FIND_KEYWORD|Complete the actual input path and verify XMCD workflow. Adding SIGMA switches to conductivity, not this branch.|
|**PLOTRHO separately**|RHORSQ is allocated only for the potential branch, but the density writer uses it without it.| fpplotprs.f:62,225,254,280 |Allocate RHORSQ independently in density branch. PLOTPRS with potential, this particular problem goes around.|
| **PLOTJALF / PLOTJORB** |FULLPOT ends up in an experimental PLY script before reading the current file; ASAwriter current ignores.| fpplot2dpr.f:71,194; plot2dpr_fp_ply.f:926; plot2dpr_asa.f |Fix dispatch/end and actual current reader. These are CONTROL selectors, not separate new implementations TASK.|

### Broken cases within otherwise meaningful tasks

| Rodina |Specific defective branch|
|---|---|
| PLOT2DRHO |ASA provides potential, not density. In FULLPOT the declared selectivity does not work and the common grid writer is not achieved.|
|POSANIPREP / POSANI data path|NQ≠NTwriter/reader may skip site or cross the field. The valid input will not correct it.|
| CHILANDAU |MPI and audited RDTAUFIL use path.|
| GILBERT, FMAG, POSANITAU |Relevant zero denominators; change ImE or number of points is not a general correction of normalization.|
| PSHIFT |Resonance splitting for missing second channel or without limited pi/2.|
| SOCPAR |Exchange column interpreted as eV without necessary transfer.|
| COMPTON |EMD2D with non-zero PNMAX and some even shifted meshes; rejects the definition of defective combination.|
| TUTORIAL |Matching plot RWF-L; CORE-ALL without core states / with non-compliant mesh; sticky NONMAG in POT-XC/POT-DECOMP for mixed types.|
| BAND |Failure of eigenvalue solver without safe continuation; possible LOG(0).|

## 4. Current ASE2SPRKKR interface

### Task identities and shared schemas

`InputParameters.for_task(name)` resolves a task and its metadata. JXC and DMI share schema JXC; `tasks/jxc.py` and `tasks/dmi.py` declare their identities. TASK.JXC and TASK.DMI are native flags. DMI-only input has identity DMI; enabling both gives identity JXC and both output families. DMI requires magnetic FREL; JXC alone also permits SP-SREL. BSFEK and BSFKK similarly share BSF, with NE determining the native mode.

CLPES is an independent schema. It is available through both `InputParameters.create("CLPES")` and `for_task("CLPES")`, and appears in `task_registry.names()` and canonical schema discovery. The registry lists 55 task identities; task identity remains distinct from the shared JXC/DMI and BSFEK/BSFKK schemas.

BIS and SIGMA-XAS have no artificial Python solver definitions: their ordinary native input paths are broken. VBUPS/VBXPS share native VBPES; PLOTJALF/PLOTJORB are CONTROL selectors with defective dispatch.

### Spectroscopy and output readers

ARPES, AIPES, SPLEED, BAND and CLPES use KKRSPEC but differ in IBLOCH branch, model and energy/geometry semantics. VBPES uses GEN/VBPES. AIPES is a local/incoherent model, not general numerical quadrature over ARPES angles. These workflows are not interchangeable.

ARPES and XAS declare `mpi=False`; XAS has EMAX=4 Ry and MECHECK is a separate task. JXC uses NE=30 and ImE=0.01 Ry. CRYS_VECS=False is omitted and preserves USE_CRY_PRIM_VECS=true. In 3D surface construction, presence of CRYS_VECS sets USE_CRY_PRIM_VECS=false (spec_rslab.f:180–181). ARPES, AIPES, BAND, SPLEED and CLPES share this definition; the native reader does not read singular CRYS_VEC. The shared KKRSPEC base is copied from ARPES rather than separated into an independent schema.

ARPES and CLPES declare reader ARPES, AIPES/BAND/SPLEED reader SPEC, and JXC/DMI reader JXC. Many experimental diagnostics use a general log reader; that does not automatically parse Grace/PLY or rate data into structured physical results.

### TUTORIAL

The definition includes all 14 choices: YLM, JLX, JLX-ASYMPT, CHR-POT, POT-DECOMP, POT-XC, RWF-L, RWF-E, CORE-ALL, DOS-FEG, DOS-SS, DOS-CLU, DOS-BULK, GEN. Defaults and validation depend on the branch. RWF uses a scalar reader, DOS-CLU requires actual cluster geometry, and POT-XC compares functionals on one density. Input support does not repair the documented native defects.

## 5. Defaults, omission and effective native behaviour

An omitted parameter representing a known value must agree with the effective native fallback. `None` can delegate to EF, the potential or file metadata; it does not mean that the numerical solver receives Python None. Explicit values may deliberately differ from native defaults. A parameter overwritten later by the solver is not an effective user control merely because it appears in the schema.

### Shared rules

| Case | Effective behaviour |
|---|---|
| EMAX sentinel | INIT_MOD_ENERGY converts values below -99 to EF. **EMAX=-1 means absolute -1 Ry**, not EF. Omission can delegate to EF where the task permits it. |
| Energy references | General EMINEV/EMAXEV are offsets from EF in eV. EKREL TASK bounds are absolute Ry. SPLEED incident energy is absolute and does not inherit ARPES semantics. |
| Presence-sensitive parameters | Present NSHLCLU=0 differs from omission. Present NEPHOT=1 selects a different reader branch from the single-photon fallback without NEPHOT. |
| GRID and NE | GRID=2/NE=1 divides by zero. GRID=3/NE=1 supplies a first point but leaves integration weights uninitialized; it is usable only where those weights are not consumed. GRID=0 has separate single-energy semantics. |
| Section ownership | COMPTON reads SPLITSS in both ENERGY and TASK. EKREL search bounds/counts belong to TASK. OPTABS uses its own TASK energy controls. |
| Potential context | Type/site bounds, shells, cutoffs, NQ/NT, KMROT, moments and actual FULLPOT data generally cannot be certified from input parameters alone. |

### Explicit choices and their limits

| Task | Current ASE2SPRKKR choice | Interpretation |
|---|---|---|
| SCF | NE=32 versus native 30. Omitted MIXOP remains 0.2 even when another MIX is read later. | A lower point count needs convergence checks; omitted MIXOP is not automatically the changed MIX. |
| DOS | GRID=3, NE=300, EMIN=-0.2, EMAX=1 Ry. | The explicit upper bound includes unoccupied states; omission would delegate to EF. It is an absolute energy. |
| BSFEK | Explicit EMAX=-1 Ry at EMIN=-0.2 Ry. | This is a descending interval, not an EF sentinel. Changing it is a public behaviour decision requiring a justified energy window. |
| BSF | NK=300 versus native 800; BSFKK starts at EMIN=0.7 Ry. | Converge k sampling; the KK default is not guaranteed to be EF. |
| XAS | Explicit EMIN=-0.2 Ry and real EMAX=4 Ry member. | Native XAS may overwrite EMIN with EF depending on EMF. Check the effective interval and TAU compatibility. |
| PHAGEN | NE=32 versus 30, ImE=0 versus generic 0.01; export forces NITER/MIX=1/1. | Requires a converged potential before export. Common SCF defaults do not describe export-step iteration control. |
| FMAG/JALF/JORB | Explicit ImE=0 versus generic EIMAG=0.01. | GRID=5 constructs its own complex contour; the scalar difference alone does not establish a numerical error. |
| JXC/DMI/CHILANDAU/CHI | GRID=5, NE=30, ImE=0.01 Ry. | Converge the integration and, where exposed, the explicit CLURAD=2.2 pair cutoff. |
| SIGMA | ImE=0.01 versus zero in the native surface default. | Smearing affects transport. Neither value is universally better without a convergence study. |
| GILBERT | GRID=0/NE=1 at EF, no ineffective energy bounds; explicit NL=4. | The task evaluates damping at EF rather than a general linear energy scan. |
| OPTABS | TEMPEL=300 K/NKT=6 versus native reader values 1e6/99; NEB=30/NOMEGA=100, OMEGA1=0/OMEGA2=8 eV. | These are explicit practical choices. Validate actual temperature use, capacity and energy pairing. |
| ARPES | NE=300, -8..5 eV relative to EF, EWORK=4.2 eV, damping 0.05/2 eV, NL=4, layer/cutoff counts 50/57. | Material-dependent choices, not omitted defaults. Native smaller counts are not automatically more accurate. |
| AIPES/SPLEED/BAND | AIPES ends at EMAXEV=0. SPLEED uses 11..60 eV/200 points and 0.2/0.2 eV damping. BAND uses explicit zero damping and EIG_IME. | Preserve task-specific energy and damping semantics. |
| KKRSPEC N_LAYDBL | Explicit two-component value. | Initializes both components despite the native typo assigning LANZ1 twice instead of LANZ1/LANZ2. |
| T1 | Explicit PRINT=1 and RUNELOOP=true. | Avoids two initialization defects; it does not repair the signed-kappa sum. |
| Plot diagnostics | Selectors use GRID=0/NE=1/ImE=0; general PLOTPRS/PLOT2DPR use GRID=1/NE=1/ImE=0. | Startup allocations do not turn spatial/radial plots into spectra. |
| APS/NRAES/AES/VBPES | NE=300 is capacity or an assumed mesh; actual energies/counts come from DOS/TAU. | Check the real header and any broadening reserve; a fixed count does not validate the file. |
| WFPLOT/MEPLOT/MECHECK | WFPLOT: GRID=3/NE=1, EMAX=EMIN, NL=3. MEPLOT: NE=300, contextual EMAX/NL. MECHECK: NE=1. | WFPLOT uses one point; MEPLOT scans energies. These diagnostics do not share a single mesh policy. |

TUTORIAL defaults depend on the branch: mathematical/plot branches use GRID=0/NE=1/ImE=0; RWF uses GRID=3/NE=10/ImE=0 and absolute 1e-7..1 Ry; DOS/GEN uses GRID=3/NE=300/ImE=0.01 and -0.2..1 Ry. STANDARD and BZINT are selected according to the branch. These are explicit interface choices, not one universal native fallback.

### PLOT2D units and output filenames

AVEC/BVEC/DVEC are Cartesian vectors in ALAT units, not fractional basis coordinates or lengths directly in atomic units. Evidence is the use of RWS/ALAT and multiplication by ALAT before radial interpolation (`plot2dpr_asa.f:138–139,173`; `plot2dpr_fp_ply.f:259`). The ASA comment about atomic units at line 102 does not match this data flow.

KKRGEN saves the original DATASET in DATSET0, then appends nonempty ADSI to DATSET (`kkrgen.f:238–243`). Public template **{DATASET} means DATSET0** and **{DATASET_ADSI} means native DATSET**. Empty ADSI adds no underscore. JALF/JORB and stored-potential plots use the first prefix; DOS/TAU and many spectra use the second. Fixed filenames are not isolated this way. PHAGEN derives filenames from POTFIL and writes POTFIL_ex_sprkkr and POTFIL.only_coulomb_new.

The default TAUDIR includes ADSI. When TAUPATH differs from `.`, the native reader prepends it even to TAUDIR starting with `/` (`kkrgen.f:266–273`). Input preparation must therefore consider both values together; this is not general Python Path behaviour.

## 6. MPI and output semantics

`mpi=True` declares interface support; it is not evidence of a numerical MPI regression run. `mpi=False` can reflect a native guard, a demonstrated defect or a conservative unverified scope. Those reasons are distinct.

| Group | Native-source finding |
|---|---|
| SCF, standard DOS, BSF, EKREL, JXC, DMI, COMPTON, POSANI, FMAG | The audited paths have MPI support in source; numerical serial/MPI agreement was not measured. FMAG completes through DENSITY before the general GEN guard. |
| XAS/XES/XRS, STONER, VBPES, APS/NRAES/AES, JALF/JORB, diagnostic GEN plots | An energy loop does not guarantee completed postprocessing. Native guards or missing reductions justify serial interfaces. |
| CHILANDAU, NUCSSCPL, PHAGEN | Specific allocation, reduction or barrier defects are documented in the individual audits. |
| GILBERT | The current definition declares MPI=true; zero moments and GEFF normalization retain their documented defects. |
| SIGMA, POSANIPREP, POSANITAU, KKRSPEC wrappers | Serial support is partly a conservative audit boundary, not proof that the native source has no MPI implementation. BAND does contain MPI code. |

Fixed names such as ME/MEi, sigmaXXXXX.dat, curdns1..3, fort.*, some PLY files and arpes.dat are not universally isolated by CONTROL.DATASET. Existing files can also select cache paths. Actual output shapes differ by task: AES/NRAES produce 2*NE-1 spectral points, XRS adds a mesh, and JALF/JORB write radial/angular data. Declared file discovery is separate from parsing or validating the physical contents.

## 7. Verification and detailed audits

The integrated definitions were tested together on 2026-11-09 on Linux with Python 3.14.4:

```sh
python -m pytest src/ase2sprkkr/input_parameters/test src/ase2sprkkr/result_files/test -q
```

Result: **1249 passed, 1 skipped, 81 warnings, 354.11 s**. Tests cover task/schema resolution, validation, serialization/parsing, copy/pickle and file manifests. The run includes 1135 input-parameter tests; the remaining tests cover result files. Ten parametrized regression cases exercise CRYS_VECS and discovery of five KKRSPEC tasks; the NeXus test checks preservation of the keyword on export.

A separate targeted run also covers native PHAGEN export:

```sh
python -m pytest src/ase2sprkkr/input_parameters/test/test_plot2dpr.py src/ase2sprkkr/input_parameters/test/test_plot_selector_file_manifests.py src/ase2sprkkr/input_parameters/test/test_phagen.py src/ase2sprkkr/sprkkr/test/test_calculator.py::TestCalculator::test_phagen -q
```

Result: **32 passed, 5 warnings, 5.91 s**. The native PHAGEN test uses converged Au_single_type.pot and checks both exported files. Existing local binaries under `SPRKKR/9_7` were used; the public archive was not rebuilt and external PHAGEN/MsSpec was not compared. The native test runs outside the sandbox, which otherwise denies the MPI startup socket.

The complete ASE2SPRKKR suite, macOS and numerical serial/MPI references for all tasks were not run in this verification. Valid input or regular termination does not certify spectra, convergence or TAU/DOS/cache provenance.

The table links current individual audits. Listed modules exist under `src/ase2sprkkr/input_parameters/test/`; their presence is not a native numerical test.

| Family | Detailed audit | Current regression modules |
|---|---|---|
| AES | [AES](english/AES-audit.md) | `test_aes.py` |
| AIPES | [AIPES](english/AIPES-audit.md) | `test_aipes.py` |
| APS | [APS](english/APS-audit.md) | `test_aps.py` |
| ARPES | [ARPES](english/ARPES-audit.md) | `test_arpes_audit.py`, `test_arpes_energy.py` |
| BAND | [BAND](english/BAND-audit.md) | `test_band.py` |
| BIS | [BIS](english/BIS-audit.md) | source analysis |
| BONDING | [BONDING](english/BONDING-audit.md) | `test_bonding.py` |
| BSDOSIJ | [BSDOSIJ](english/BSDOSIJ-audit.md) | `test_bsdosij.py` |
| BSF | [BSF](english/BSF-audit.md) | `test_bsf_audit.py`, `test_bsf_file_manifests.py` |
| CHI | [CHI](english/CHI-audit.md) | `test_chi.py` |
| CHILANDAU | [CHILANDAU](english/CHILANDAU-audit.md) | `test_chilandau.py` |
| CHIXAS | [CHIXAS](english/CHIXAS-audit.md) | `test_chixas.py` |
| CLPES | [CLPES](english/CLPES-audit.md) | `test_clpes.py` |
| CLXPS | [CLXPS](english/CLXPS-audit.md) | `test_clxps.py` |
| COMPTON | [COMPTON](english/COMPTON-audit.md) | `test_compton.py` |
| DMI | [DMI](english/DMI-audit.md) | `test_dmi.py`, `test_exchange_file_manifests.py` |
| DOS | [DOS](english/DOS-audit.md) | `test_dos.py` |
| EKREL | [EKREL](english/EKREL-audit.md) | `test_ekrel.py` |
| FMAG | [FMAG](english/FMAG-audit.md) | `test_fmag.py` |
| FSCAT | [FSCAT](english/FSCAT-audit.md) | `test_fscat.py` |
| GILBERT | [GILBERT](english/GILBERT-audit.md) | `test_gilbert.py` |
| JALF | [JALF](english/JALF-audit.md) | `test_jalf.py` |
| JORB | [JORB](english/JORB-audit.md) | `test_jorb.py` |
| JXC | [JXC](english/JXC-audit.md) | `test_jxc.py`, `test_jxc_definition.py`, `test_exchange_file_manifests.py` |
| MECHECK | [MECHECK](english/MECHECK-audit.md) | `test_mecheck.py` |
| MEPLOT | [MEPLOT](english/MEPLOT-audit.md) | `test_meplot.py` |
| NRAES | [NRAES](english/NRAES-audit.md) | `test_nraes.py` |
| NUCSSCPL | [NUCSSCPL](english/NUCSSCPL-audit.md) | `test_scattering_file_manifests.py`, `test_dataset_filenames.py` |
| OPTABS | [OPTABS](english/OPTABS-audit.md) | `test_optabs.py` |
| PHAGEN | [PHAGEN](english/PHAGEN-audit.md) | `test_phagen.py` |
| PLOT-selectors | [PLOT-selectors](english/PLOT-selectors-audit.md) | `test_plotprs.py`, `test_plot_selector_file_manifests.py` |
| PLOT2D-selectors | [PLOT2D-selectors](english/PLOT2D-selectors-audit.md) | `test_plot2dpr.py`, `test_plot_selector_file_manifests.py` |
| PLOT2DPR | [PLOT2DPR](english/PLOT2DPR-audit.md) | `test_plot2dpr.py` |
| PLOTJALF-PLOTJORB | [PLOTJALF-PLOTJORB](english/PLOTJALF-PLOTJORB-audit.md) | source analysis |
| PLOTPRS | [PLOTPRS](english/PLOTPRS-audit.md) | `test_plotprs.py` |
| POSANI | [POSANI](english/POSANI-audit.md) | `test_posani.py` |
| POSANIPREP | [POSANIPREP](english/POSANIPREP-audit.md) | `test_posaniprep.py` |
| POSANITAU | [POSANITAU](english/POSANITAU-audit.md) | `test_posanitau.py` |
| PSHIFT | [PSHIFT](english/PSHIFT-audit.md) | `test_pshift.py` |
| SCF | [SCF](english/SCF-audit.md) | `test_scf_audit.py` |
| SIGMA-XAS | [SIGMA-XAS](english/SIGMA-XAS-audit.md) | source analysis |
| SIGMA | [SIGMA](english/SIGMA-audit.md) | `test_sigma.py` |
| SOCPAR | [SOCPAR](english/SOCPAR-audit.md) | `test_socpar.py` |
| SPLEED | [SPLEED](english/SPLEED-audit.md) | `test_spleed.py` |
| STONER | [STONER](english/STONER-audit.md) | `test_stoner.py` |
| T1 | [T1](english/T1-audit.md) | `test_t1.py` |
| TDOBS | [TDOBS](english/TDOBS-audit.md) | `test_tdobs.py` |
| TORQUE | [TORQUE](english/TORQUE-audit.md) | `test_torque.py` |
| TUTORIAL | [TUTORIAL](english/TUTORIAL-complete-audit.md) | `test_tutorial.py` |
| VBPES | [VBPES](english/VBPES-audit.md) | `test_vbpes.py` |
| VBUPS-VBXPS | [VBUPS-VBXPS](english/VBUPS-VBXPS-audit.md) | `test_vbpes.py` |
| WFPLOT | [WFPLOT](english/WFPLOT-audit.md) | `test_wfplot.py` |
| XAS | [XAS](english/XAS-audit.md) | `test_xas.py`, `test_xas_definition.py` |
| XES | [XES](english/XES-audit.md) | `test_xes.py` |
| XRS | [XRS](english/XRS-audit.md) | `test_xrs.py` |

## 8. Remaining priorities

1. Repair native indexing/allocation in FSCAT, NRAES, BSDOSIJ, NUCSSCPL and XRS; the signed-kappa sum in T1; scratch-file data flow in BONDING; and RHORSQ allocation in standalone PLOTRHO.
2. Repair BIS/SIGMA-XAS reachability and PLOT2D/current-plot dispatch. Establish physical reference comparisons for XES and the unresolved DMI/SEFIN interpretations.
3. Separate the shared KKRSPEC schema and distinguish conservative serial metadata from native MPI prohibitions. Decide the BSFEK default energy window explicitly as a public behaviour change.
4. Validate actual TAU/DOS capacities, NQ/NT, meshes, shells, magnetic geometry and cache provenance where files are available.
5. Add representative numerical and serial/MPI references for supported physical paths, recording native source and build provenance. The integrated input suite already passes; it does not replace those references.

## 9. Disabled tasks

Only the list of branches from the submitted revision; no other pretend definitions were created for them. U public blockage is PUBLIC_VERSION build. U NEGF is the off old path over KKRRSP, not by this conclusion the whole separate program KKRNEGF.

|Task / Branch|Reason|Place|
|---|---|---|
| **FORCETEN** |UNDER_CONSTRUCTION before calculation| gen.f:1055–1057 |
| **XMO** |Public version: not available / UNDER_CONSTRUCTION| xrayspec.f:283–287 |
| **OPTICS** |Public version: not yet available| rsp.f:887–888 |
| **MAGNET** |LINRESP_MAGNET is a stub with STOP| linresp_magnet.f:1–4 |
| **CHIDYN** |LINRESP_SUSC_DYN is a stub with STOP| linresp_susc_dyn.f:1–4 |
| **PHONONS** |Public version explicitly stops task| check_input.f:34–36; rsp.f:962–963 |
| **PUMP** |LINRESP_PUMP is a stub with STOP| linresp_pump.f:1–4 |
| **OPM** | OPM scheme not available | check_input.f:136 |
| **SIGMA-DNS** |SIG_DNS to UNDER_CONSTRUCTION| sig_dns.f:13 |
|**NEGF over KKRRSP**|From version 8.3 refers to separate KKRNEGF and stops this path| rsp.f:905–907 |
