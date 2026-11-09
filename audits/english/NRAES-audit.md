# NRAES audit SPR-KKR 9.7 and definition

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

| Task | Schema | Executable | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| NRAES | NRAES | kkrgen | false | true | default |

## Input

Nonrelativistic Auger electron spectroscopy in kkrgen. initvar.f:290–296 sets INITELOOP=false, RUNELOOP=false, RDDOS=true and USELPLS1=true. DOS is read from DATASET.dos, open to unit 10. The reader requires an old format spin-resolved DOS; there is no automatic conversion DOS from potential.

TASK reads only IT, CL, CALCLS and OUTGNU. GAMMA from a similar routine NRAPS does not belong here. The definition allows n=1..5 and s/p/d/f at physical l<n. The upper limit IT, the number of types and cutoff must correspond to the potential and DOS; the definition itself does not have their metadata.

## Defaults and writing

| Option |Native behaviour|Definitions| Serialization |
| --- | --- | --- | --- |
| TASK.IT | 1 | 1 | default omitted |
| TASK.CL | 2P | 2P | default omitted |
| CALCLS / OUTGNU |false in absence| false | omitted |
| MODE | IREL=min(1,IREL) | SREL |explicitly entered|
| ENERGY.GRID / NE / ImE |without specific NRAES meshes; energy read from DOS| 1 / 300 / 0 Ry |explicitly entered|
| EMIN / EMAX |EMIN general -0,2 Ry; EMAX initialized to EF, then ETAB read from DOS|bounds are not exposed| omitted |

Deleted TASK defaults correspond to SPR-KKR. Explicit SREL expresses native switching the normal relativistic mode to IREL=1; NREL is also allowed. The definition warns at FREL/SP-SREL because the source overwrites them. FULLPOT zakazuje check_input.f:60–62. DMFT/LDA+U prohibits GEN and rejects definitions.

GRID=1, NE=300 and ImE=0 are explicit initialization options, not native draw defaults. NE determines NEMAX and thus field capacity; actual energy and count NE are read from the file. Better than solid NE=300 would be derived from the header of a specific DOS. This definition metadata does not automatically read the file. ENERGY bounds should not be presented as control of spectral range NRAES when the table overwrites the table.

## Native errors and prerequisites

1. **IAES is only partially initialized.**nraes.f:99 allocates IAES(2*NEMAX-1,0:NLMAX,0:2). However, before the convolution it is called RINIT only with the number NEMAX*(1+NLMAX)*(1+2) (row 817), instead (2*NEMAX-1)*[1+NLMAX)*3. The subsequent loop runs over 2*NE-1 and adds to IAES. Part of the array may contain uninitialized values. Due to the column-major storage, the problem cannot be described only as a lack of zeroing of last energies: partial zeroing affects different channels. This requires the correction of Fortran; input definition does not replace it.
2. **Capacity check missing DOS.** Before reading, DOS and ETAB is allocated to NEMAX. The routine then reads NE, NT and NQ from DOS header and writes lines ETAB/DOS without verification against capacity and potential (rows 211–246). ENERGY.NE must be sufficient for the file; the definition will only verify the minimum capacity 2.
3. **Mesh must be growing and even.** DE=real(ETAB(2)-ETAB(1)) is used without controlling the number of energy, then as a single step of the entire spectrum. One energy or non-uniform mesh is not adequately treated. Interpolation matrix elements uses DEME=(EMAX-EBOT)/(NEME-1); EMAX=EBOT causes zero step. This property specific DOS is not verifiable only InputParameters.
4. **Limited shell-name table.** SHELL contains only K,L,M,N,O (nraes.f:86) and is indexed by NCXRAY. n=6 exceeds that table. Both NRAES and APS definitions therefore limit n to 5; the wider core-label parser syntax is not safe for these routines.
5. **The output energy is relative to onset.** EONSET=2*EFERMI-E1 and EFIN=(2*EBOT-E1+(IE-1)*DE) -EONSET (rows 995–998). E1 is therefore deleted from the axis by deduction. This is not the absolute kinetic energy of the Auger electron; the shift to the absolute axis requires the corresponding onset.
6. **The output data do not correspond to the number of spectrum points.** WRHEAD gets NE read from DOS (row 198), but the main spectrum with writes with 2*NE-1 points (rows 996–999). Parser cannot assume without further control that the header NE is the number of spectral lines.
7. **MPI the branch does not complete NRAES.** NRAES is not in the list allowed tasks after MPI the energy loop GEN. The definition has mpi=False.

CALCLS turns on another (l,s)-distinguished output. OUTGNU turns on additional gnuplot data. Specialized parser NRAES has not been created and numerical correctness has not been verified by running SPR-KKR.

## Verification

Current regression modules: `test_nraes.py`. They are included in the integrated input-parameter and result_files suite (**1249 passed, 1 skipped**, 2026-11-09). Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
