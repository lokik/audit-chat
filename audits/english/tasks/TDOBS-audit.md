# TDOBS — SPR-KKR 9.7 / ASE2SPRKKR

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

| Task | Schema | Executable | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| TDOBS | TDOBS | kkrgen | false | true | default |

Input selection for the generic kkrgen DOS/observable path. A distinct time-dependent physical calculation is not established by the source.

## Native behaviour

initvar.f:523–530 detects TDOBS, sets RDTAUDIR=true, WRTAUDIR=false and turns on initialization/energy loop. kkrgen.f:932–961 checks tau cache: with available cache initially shuts down RUNELOOP/INITELOOP, without it sets recuperation and writing tau. Then kkrgen.f:971–974 for TDOBS always turns both loops back on. GEN in energy loop calling CALCDOS (gen.f:869–876) and CALCDOS_WRITE (r.1151). No other dispatch branch TDOBS was found in the sources.

CALC_VECOBS is a different internal switch, initialized false in mod_calcmode.f:116; parser TDOBS does not set it. The name TDOBS is therefore not represented as a guarantee of vector or time-dependent analysis. General DOBS areays themselves do not support such analysis.

GEN MPI guard (gen.f:1360–1371) TDOBS does not permit and terminate STOP_REGULAR; mpi=False is required. Support general DOS energy loop is not numerical verification of all cases CPA/FULLPOT or re-use of cache.

## Inputs and defaults

TASK=TDOBS and CONTROL.ADSI=TDOBS are recorded. TAU.TAUDIR/TAUPATH are optional without Python default: native uses DATASET.tau.dir and current directory (kkrgen.f:266–273). Cache must correspond to potential, energy and configuration; definition does not control its content.

GRID3, NE300, ImE.01, EMIN-.2, EMAX1 Ry are explicit spectral defaults taken from audited DOS definition. They do not have to correspond to generic native initial defaults but are not silently omitted. EMAX1 is not EF sentinel and includes vacant states; a narrower window may be more suitable according to the objective. We do not exhibit RDTAUDIR as an untrue overwritten option: its true is native behavior task. The missing tau does not automatically cause the end; native selects the recomputation fallback.

Validation NE>=1, GRID2/3 at least2 points, finite bounds, ImE>=0 and warning at GRID3/ImE0. MODE, CPA and STRCONST are available in a task-specific schema.

## Verification

Current regression modules: `test_tdobs.py`. They are included in the integrated input-parameter and result_files suite (**1249 passed, 1 skipped**, 2026-11-09). Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
