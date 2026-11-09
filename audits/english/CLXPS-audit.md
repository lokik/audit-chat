# CLXPS audit SPR-KKR 9.7 and definition ASE2SPRKKR

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

| Task | Schema | Executable | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| CLXPS | CLXPS | kkrgen | false | true | default |

## Scope

Core-level XPS in kkrgen, ASA and fully relativistic mode IREL=3. FULLPOT zakazuje check_input.f:85–88, DMFT/LDA+U gen.f:211–222. The definition rejects explicitly incompatible MODE and correlation; FULLPOT from potential is controlled in SPR-KKR. IT is a type 1 index, its upper limit depends on potential.

TASK contains ME, IT, CL, EPHOT, ImE, MECHECK, LINPOL, USETSS, ANGRES and geometry ANGQ0..10, ANGE0..10, ANGK0..10. The angles are a pair of polar and azimutal angles in degrees. EPHOT is in eV, ImE in Ry. USETSS replaces a single-site t matrix scattering-path operator. LINPOL switches from circular to linear polarization.

## Defaults and their writing

| Parameter | SPR-KKR |Definition| Serialization default |
| --- | --- | --- | --- |
| TASK.ME | ADA | ADA | omitted |
| TASK.IT | 1 | 1 | omitted |
| TASK.CL | 2P | 2P | omitted |
| TASK.EPHOT | 1253,6 eV | 1253,6 eV | omitted |
| TASK.ImE |First 0, then at abs(ImE)<1e-8 change to 0,01 Ry| 0,01 Ry | omitted |
|four switches TASK|false in absence| false | omitted |
| ENERGY.GRID / NE / ImE |3 / 18 / 0,01 Ry for CLXPS|the same|explicitly entered|
| geometrie |internal zero angles, number according to three found|n.e.c. None| omitted |

TASK.ImE corresponds to effective native default, not intermediate 0. Explicit almost zero value will cause an alert to native refund. There is no reason to change native defaults. General initialisation ENERGY.EMIN is not a physical start XPS spectrum; final energy is compiled as ECOR + EPHOT + i*TASK.ImE (clxps.f:203–210), not by scanning ENERGY.EMIN/EMAX.

## Native-source findings

1. **ME=GRV is not completed.** clxps.f:186 prepares the derivative for GRV, but subsequent branches of support ADA and others end up STOP. The definition therefore accepts only ADA. This is not a numerical verification of accuracy ADA.
2. **One geometry is actually read with an index 0.** The loop is assembled ANGQ0, ANGE0, ANGK0. Subsequent LL=4 does not change the already compiled names transmitted to the reader. Therefore, I do not add the seemingly intended unsufixed ANGQ/ANGE/ANGK. Complete geometry 0 takes precedence over 1..10.
3. **The numberd geometry must be continuous.** Reading 1..10 ends in the first incomplete three. So the source can quietly ignore the other geometry. The definition rejects gaps and incomplete triples; warns if geometry 0 overlaps another.
4. **ANGRES without geometry can leave NGEO=0.** Definition requires complete geometry. Without ANGRES the source sets a single geometry; in multiple defined definitions it alerts.
5. **The size of the energy table must suffice the number of core states.** Routine overwrites NETAB(1)=4*l+2 and writes so many lines ETAB. The definition maintains native NE=18 and controls the allocated range when reducing it. For f shell, 14 states are required. Also controls physical l<n.
6. **Final energy below EF stops calculation.** This condition cannot be verified only from input without core energy potential; remains control SPR-KKR.

MPI is conservatively disabled for this postprocessing and its outputs in the definition. This is not a claim that the source explicitly prohibits a parallel energy loop; parallel CLXPS has not been verified.

## Verification

Current regression modules: `test_clxps.py`. They are included in the integrated input-parameter and result_files suite (**1249 passed, 1 skipped**, 2026-11-09). Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
