# XES definition and native behaviour

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

| Task | Schema | Executable | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| XES | XES | kkrgen | false | true | default |

## Interface

INITVAR recognizes TASK.XES and sets X-ray workflow IXRAY=1, ICORE=0, RDTAUDIR=true and INITELOOP=true. GEN:1378–1385 calls XRAYSPEC which internally renames XES to RXES. CHECK_INPUT:43–47 requires IREL=3. The definition therefore rejects other MODE.

XRAYSPEC:376–380 requires ENERGY.NE and its equality of points from the first TAU contour. The definition itself does not read the content TAU directory: its mesh, number of points, types and potential must correspond to the actual calculation. When available TAU is read, otherwise the main workflow can prepare the loop; the numerical link has not been started here.

TASK exhibits IT, CL, ME, OUTPUT, POLOBS, DIPOL, GFCONVRH, WRREGIRR, NONDIPE2, NONDIPM1, TAUCORE and Euler angles FRAMETET/FRAMEPHI/FRAMEGAM. This is not an interface ARPES or VBPES. Frame angles rotate a common coordinate frame, not a simple single vector of an electric field; init_mrot_frame.f:48 form a matrix GETMROT(-gamma,-theta,-phi).

## Defaults and writing

| Option | Native effect |Definitions|
| --- | --- | --- |
| IT | 1 | 1, omitted |
| CL |2P from common reader| 2P, omitted |
| ME | ADA | ADA, omitted |
| OUTPUT |BARN; normal output in Mb| BARN, omitted |
| POLOBS | S | S, omitted |
| DIPOL |0; for NAB it is overwritten to 1| 0, omitted |
| GFCONVRH |false in absence of keyword| false, omitted |
| WRREGIRR |DATA lists true but SECTION_FIND_KEYWORD overwrites them by presence| false, omitted |
| TAUCORE |Two zeros, hodnoty<1e-6 take over table WCOREHOLE| None, omitted |
|Euler angles| 0/0/0 |None, omitted; absence does not switch CHANGE_FRAME|
| GRID/NE |XES does not have its own initialization default; NE=0 leads to invalid capacity|explicitly3/300|
| ImE | 0,01 Ry |explicitly0,01 Ry|
| EMIN | -0,2 Ry |explicitly -0,2 Ry|
| EMAX |XES explicitly overwrites to EF| None, omitted |

The definition limits n<=5 because SHELL contains only K,L,M,N,O, and l<n. IT must be positive; its upper limit and the existence of core shells require potential. NE>=2 protects GRID3 from uninitialized ESTEP at one point. Split and two paths are rejected due to the use of the first TAU contour. ME=NAB s DIPOL=0 fires overwrite alerts; NONDIPE2 takes precedence over NONDIPM1.

## Major caveats and native errors

1. **MPI does not complete the spectrum.** GEN:1355–1368 terminates XES after the energy loop, before XRAYSPEC. Both XES and XAS Python interfaces use mpi=False. This follows from the native guard, not a numerical MPI comparison.
2. **RXES shares its own rate calculation with RXAS.** The only occurrence RXES outside the comment is when renaming task. The calculation and output branch distinguishes XMO/XRS against all others, so XES enters into the common absorption-rate branch. Therefore, the result cannot be declared as a verified emission spectrum without physical comparison with the reference. It does not have a simple proxy STOP as FORCETEN so that task has not been skipped but will have a significant ALE in a joint audit.
3. **The output is still marked XAS.** The .rat name and text SPEC use the same core shell convention as XAS; the text states XAS spectrum. XAS and XES for the same DATASET, type and shell may write the same file.
4. **MESPLIT is not a switch:** after loading the keyword, K_ME_SPLIT always sets true (297–298). The definition does not expose it as an effective choice. DYNAMICAL is read only for RXAS/XMO, therefore it is not offered for XES.
5. **TAUCORE is read but its use for RXES rate is not documented.** Values are transferred from eV to Ry or taken WCOREHOLE. The subsequent use is in extended XMO/XRS branches; the standard matrix-element call sets CECOR=0. Parameter is not designated as effective emission Broadening management. Change TAUCORE does not have to change the spectrum itself RXES.
6. **ME is not proof of three full Gauge calculations.** MEFORM is loaded; NAB forces DIPOL=1, GRV prepares radial derivations. However, the actual normal matrix elements is calculated ME_ALF_ALF, without delivery MEFORM. Only acceptance ADA/NAB/GRV can not be confused with verified independent implementation of all three forms.
7. **Non-dipole variants depend on formalism.** ME_ALF_ALF_NONDIP is called only for .NOT.FULLPOT and DIPOL=0 (805–812). For FULLPOT the same flags do not automatically guarantee the same contributions. Potential formalism is not read pure InputParameters validator.
8. **Only one atomic type:** NTXRAY=1 and IT is loaded as a single number; it is not a list of types. The output has three polarization records for each core state and energy, not a single intensity column. POLOBS=S gives (+,-,z), C gives (x,y,z).
9. **XAS has its own current definition:** EMAX is a real member defaulting to 4 Ry, and MECHECK is not an XAS option. Native bound overrides and TAU compatibility remain separate conditions; XES does not bypass them.

## Verification

Current regression modules: `test_xes.py`. They are included in the integrated input-parameter and result_files suite (**1249 passed, 1 skipped**, 2026-11-09). Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
