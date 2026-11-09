# MEPLOT

Current as of **9 November 2026**. The authority is SPR-KKR 9.7 source **PUB9.7_260420_18h13**; the interface was checked at ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Source baseline and integrated verification](../SPRKKR-ASE2SPRKKR-common-audit.md#source-baseline-and-scope).

| Task | Schema | Executable | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| MEPLOT | MEPLOT | kkrgen | false | true | default |

## What task does

ME_PLOT counts and draws dipole matrix elements. Output curves
contain absolute values standard matrix elements, not
the entire absorption spectrum. Runs over kkrgen, mpi=False.

- INITIALSTATE=BAND, NEPHOT=1: electron scan; ENERGY determines the initial
energy and final energy is the initial energy + EPHOT/RY_EV.
- INITIALSTATE=BAND, NEPHOT>1: photon scan at a single initial energy.
- INITIALSTATE=CORE: ENERGY determines final electron energy, initial
the core state is selected CL. Photon input parameters are not read in this branch.

Initial m_j=+1/2, polarisation IPOL=2 (q=0,z) and form matrix elements
MEFORM='ADA' are fixed in the source. Parameters MJ, MJI, IPOL or MEFORM therefore
we don't invent and expose as supported inputs.

## Defaults and writing

| Parameter |Effective SPR-KKR default|Definition|Writing default?|
|---|---|---|---|
| ITSEL | 1 | 1 | No |
| IPRINT |Current CONTROL.PRINT| Dynamicky CONTROL.PRINT | No |
| INITIALSTATE | BAND | BAND | No |
| L |All initial band orbitals; LISEL=-1|None = no selection of l| No |
| CL | 2P | 2P | No |
| TASK.NONMAG | False | False |No, flag with writes only for True|
| SEFIN | 0 | 0 | No |
| EPHOT | 100 eV | 100 eV | No |
| NEPHOT | 1 | 1 |No, even after explicit setting to 1|
| EPHOTMIN/EPHOTMAX |Unset local variables| None | No |
| ENERGY.EMIN | -0.2 Ry | -0.2 Ry | Yes |
| ENERGY.EMAX |EF at missing explicit limit|None = ponechat actual EF| No |
| ENERGY.ImE | 0.01 Ry | 0.01 Ry | Yes |
| ENERGY.GRID |It does not have a separate usable MEPLOT default| 3 | Yes |
| ENERGY.NE |NETAB initialized to 0; without explicit NE the calculation may end| 300 | Yes |
| SITES.NL |Context 3 or 4|Shared current definition: None, delegation to SPR-KKR| No |

NE=300 and GRID=3 are consciously written to the input ASE options for smooth energy
a curve. We are not declaring them to be SPR-KKR default. GRID=3 is more suitable for the fence
than the general arc GRID=5: runs along the real axis at the distance of ImE.
For photon scans, the user must set ENERGY.NE=1.

EMAX=None delegates the upper bound to the actual EF from the potential; None is not a numerical EF. L=None leaves all orbitals selected. SITES.NL is omitted, preserving the atom-dependent native cutoff.

## Substantial differences against WFPLOT

- ITSEL is not a mandatory keyword: STOP_KEY=0 preserves the initial value 1.
- The local printing key is IPRINT, not PRINT.
- The status selection is INITIALSTATE, not STATE.
- Missing L does not mean highest orbital. The reader may first use
NLQ-1 but at FOUND_INTEGER=False subsequently returns LISEL=-1 and selects everything.
- ME_PLOT actually passes through NETAB(1) energy, not just ETAB(1,1).
- FULLPOT has implemented FPSSITE branches. There is no trace as in WFPLOT;
The numerical accuracy of these branches has not been verified.
- IREL=3 is not enforced for MEPLOT. Channel construction assumes relativistic
kappa/j/m_j indexing. For others MODE we issue warnings, not claims,
Fortran explicitly rejects them. Their safety and physical importance
require separate verification.

## Native errors and limitations

1. ME_PLOT distinguishes photon scan mode by presence NEPHOT
or some limits, not only according to NEPHOT>1. Explicit NEPHOT=1 without
the limits therefore enter the range branch and can print uninitialized
EPHOTMIN/EPHOTMAX; also misses reading EPHOT. The module default 1 never
   does not write. This also protects the combination NEPHOT=1, EPHOT=150.
2. Fortran's incomplete control only captures part of the invalid combinations.
The definition therefore requires both limits for NEPHOT>1 and rejects the limits in the scheme
   NEPHOT=1. For a single energy is used EPHOT.
3. At NEPHOT>1, YY is indexed over IEPHOT, not over a pair IE/IEPHOT.
S multiple electron energies with the same photon curves repeatedly
they rewrite; eventually there will be the last electron energy left. Definitions
rejects and requires ENERGY.NE=1. This is deliberately stricter protection.
than in Fortran, which does not stop the combination.
4. SEFIN is automatically switched off at NEPHOT>1. The definition draws attention to this;
the value does not automatically overwrite.
5. For CORE, L and photon parameters are not read. Pronounceable setting fires
Attention. However, SEFIN can also be used in this branch.
6. SEFIN receives ERYD, not ERYDF. U BAND is ERYD initial energy,
while the final state has energy ERYD+EPHOT/RY_EV. Vzhledem k popisu
SEFIN as a self-energy final state is suspicious; without numerical
We do not call this a proven error and we do not change the source.
7. GEN has an explicit prohibition DMFT/LDA+U for some tasks including WFPLOT,
but MEPLOT is not included in this list. The definition does not invent such a prohibition;
the correct use of these methods with MEPLOT remains unverified.

## Validation

ITSEL>=1, L>=0, NEPHOT>=1, SEFIN=0..3, positive electron energy counts.
INITIALSTATE is BAND/CORE. CL has n=1..6, s/p/d/f; physical control l<n
is more stringent than the Fortran reader itself. For explicit uniform NL
Verify l<NL. No potential does not verifyme upper limit ITSEL, actual
existence of core shells or mapping of non-single NL to atomic types.
Incomplete editing states are allowed at set; final validation occurs
by saving/parse. Existing pair validation EMINEV/EMAXEV is used.

## Examples

```python
from ase2sprkkr import InputParameters

ip = InputParameters.create("MEPLOT")
ip.CONTROL.POTFIL = "Fe.pot"
ip.TASK.EPHOT = 150.0  # eV
ip.ENERGY.set({"EMINEV": -10.0, "EMAXEV": 0.0})
ip.save_to_file("Fe-meplot.in", atoms=False)
```

Photon scan:

```python
ip.ENERGY.set({"NE": 1, "EMINEV": -2.0, "EMAXEV": -2.0})
ip.TASK.EPHOT.clear()
ip.TASK.set({"NEPHOT": 20, "EPHOTMIN": 10.0, "EPHOTMAX": 200.0})
```

For CORE, the INITIALSTATE=CORE, CL and the final energy interval shall be set;
previous photon parameters should be cleaned because they are ignored.

## Source evidence and verification

- me_plot.f:132–146: fixed MJI/IPOL and initialisation defaults.
- me_plot.f:151–211: actual parameter names, L fallback, photon reader,
ignoring SEFIN with multiple photon energies.
- me_plot.f:297–306, 349–356, 564–571: dimension and rewrite of photon curves.
- me_plot.f:465–476: final state energy and argument ERYD for SEFIN.
- me_plot.f:379–387, 499–507: FPSSITE branches for FULLPOT.
- sefin.f:25–28, 107–108: meaning and limits SEFIN.
- initvar.f:159–160, 208–218: detection MEPLOT and switching off normal ELOOP.
- gen.f:210–222: List of prohibitions DMFT/LDA+U, without MEPLOT.
- gen.f:1357–1370, 1480–1483: MPI Guard and self-dispatching.

## Verification

Current regression modules: `test_meplot.py`. They are included in the integrated input-parameter and result_files suite (**1249 passed, 1 skipped**, 2026-11-09). Input tests check the interface and serialization; they do not certify physical results or serial/MPI agreement. No independent numerical reference calculation of this task was verified here.
