# SPR-KKR and ASE2SPRKKR audits

This repository contains source-level reviews of the **SPR-KKR 9.7** native implementation and the **ASE2SPRKKR** Python interface. It separates physical functionality and known defects from the documentation and file/parameter contracts exposed to users.

## Task interface, parameters and documentation

These two complementary task-by-task inventories cover **56 headings** (55 registered task identities plus the shared BSF schema). They record each task's purpose, documented versus undocumented task-specific parameters, required inputs, expected output and cache files, reader selection, and evidence for ASE2SPRKKR test coverage. They also classify **native embedding support in SPR-KKR 9.7** for all 56 entries (**4 yes / 18 conditional / 34 no / 0 unverified**) and identify the **normal executable** versus the embedding-specific `embscf` or `embgen` (or note that no supported embedding executable exists). Conditional/unverified `embgen?` entries are *not* enabled in ASE2SPRKKR; notably, STONER is **not supported** because `SUSC_GAMMA` loops over host types and reads host wave-function records that `CLUDRIVE` deliberately removes when reopening a cluster-only scratch file. Correct TAU inputs are assumed independently. The latest native source recheck distinguishes the explicitly cluster-aware `POSANITAU` setup from periodic momentum-space COMPTON/POSANI/POSANIPREP calculations. This is distinct from native numerical validation or GUI readiness. “Works” in a source audit is **not** evidence that a native numerical regression was performed.

- [English — ASE2SPRKKR task interface audit](audits/english/ASE2SPRKKR-task-interface-audit.md)
- [Česky — audit rozhraní tasků ASE2SPRKKR](audits/czech/ASE2SPRKKR-task-interface-audit.md)

## Native functionality, limitations and bug evidence

- [English — consolidated SPR-KKR 9.7 / ASE2SPRKKR source audit](audits/english/SPRKKR-ASE2SPRKKR-common-audit.md) — native capabilities, limitations and verification.
- [Česky — souhrnný audit SPR-KKR 9.7 a ASE2SPRKKR](audits/czech/SPRKKR-ASE2SPRKKR-common-audit.md) — funkčnost, omezení a chyby.
- [English individual task audits](audits/english/tasks/) — native routines, line-level findings and limitations.
- [České audity jednotlivých tasků](audits/czech/tasks/) — české podrobné zprávy.

## Verification and baseline

The consolidated audit (9 November 2026) reports **1249 passed, 1 skipped** in a Python input-parameter and result-file regression suite; these tests do not establish numerical correctness. It also records a targeted native PHAGEN export integration test (checking two files) without independent MsSpec comparison. The interface audits distinguish these levels of evidence per task. Their manual coverage comparison derives from an older public SPR-KKR User's Guide and is explicitly marked provisional where primary manual references were not confirmed.

The latest interface inventory additionally cross-checked task registry and selected manifest implementations against ASE2SPRKKR `develop` at `8aa03363433b45b98e2594bfaecc7578a46be94c` (12 November 2026); no new complete native numerical test campaign is claimed.
