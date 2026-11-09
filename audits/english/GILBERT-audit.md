# GILBERT audit SPR-KKR 9.7 and ASE2SPRKKR

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

| Task | Schema | Executable | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| GILBERT | GILBERT | kkrchi | true | false | default |

## Definition and validation

- SETFLUCT=MLIN supports linear M(T)=1−DMAGDT*T and combined vibrations; DMAGDT, TDEBYE and INITSVEC are exposed.
- M_T requires FLUCTFIL; validation rejects ignored TMPMIN/TMPMAX/NTMP.
- MODE, CPA and STRCONST are available. The audited scope is magnetic FREL with BZINT=POINTS; this does not imply that every other native branch stops.
- Validation requires positive integer counts, finite nonnegative ordered temperatures, DMAGDT only with MLIN, and a complete temperature mesh for vibrations without SETFLUCT.
- GRID=0/NE=1 provides single-energy initialization. GIL sets ERYD=EFERMI; generic EMIN/EMAX do not control damping and are not exposed. SPLITSS is rejected.

## Defaults: written versus omitted

GRID=0 and NE=1 are written to the input; this supplies single-point initialization. ImE=0 is written to the input, but own damping counts to EF. SITES.NL=4 is also reported: an explicit ASE default, not an omitted native fallback. The suitability NL depends on the system and convergence.

Temperature and fluctuation parameters do not have forced Python defaults and are not recorded on entry. Native NFTET/NFPHI start at 1, INITSVEC to 0. NVIBRA is without requirement 1 but with vibration requirement INITSVEC=0 it native overwrites it to canonical 14 directions. For example NVIBRA=26 i.e. does not mean 26 directions; for the generated network it is necessary to choose INITSVEC=1. TDEBYE=0 means the table value by composition, not automatically division by zero. Negative TDEBYE/RMSU0 procedures are not covered by this definition.

For MLIN explicit TMPMIN is required because native default initial value 9999 is not a useful common entry. Without TMPMAX native uses one point in TMPMIN; NTMP without TMPMAX would ignore it, therefore it is rejected.

## Significant restrictions native calculation

1. GIL requires KMROT=0 and IBZINT=2 or 6 (gil.f:89–93). The definition makes the standard POINTS accessible. KMROT and magnetic moments depend on potential and this validation is does not verify.
2. gilsum.f:180–204 counts GEFF=2*(1+Morb/Mspin) and other non-zero-time normalizations. The calculation also takes place before the condition of printing Z>0. Empty or nonmagnetic components and compensated total moment can cause invalid results. CONTROL.NONMAG=False does not prevent it itself.
3. INITVAR turns on RDTAUDIR; a compatible initialization TAU file may be needed. The potential and external data have not been numerically verified here.
4. Native contains MPI branch in gil.f; existing ASE parallelisation setting has not been modified or numerically certified. MCS, RDLM and NLCPA are not part of this audit.

## Accessories

SPR-KKR 9.7: initvar.f:376–382; kkrchi.f:1425–1427; rsp.f:913–925; gil.f:89–121,266–320; thermal_init.f:79–81,105–119,170–242,322–368,548–558; mod_thermal.f:16; thermal_rmsdisp.f; gilsum.f:180–204.

## Verification

Current regression modules: `test_gilbert.py`. They are included in the integrated input-parameter and result_files suite (**1249 passed, 1 skipped**, 2026-11-09). Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
