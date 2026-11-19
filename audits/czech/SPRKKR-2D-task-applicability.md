# Podpora nativní 2D geometrie — audit tasků SPR-KKR 9.7

**Stav:** 19. listopadu 2026. **Zdroj:** SPR-KKR `PUB9.7_260420_18h13` a stávající [audit rozhraní tasků](ASE2SPRKKR-task-interface-audit.md) / [souhrnný audit](SPRKKR-ASE2SPRKKR-common-audit.md). Jde o analýzu prováděných Fortranových větví, ne nově provedený numerický regresní test.

## Význam

**2D** znamená `SYSTEM_DIMENSION=2D` (povrch/vrstvy/rozhraní), nikoli 3D supercelu s vakuem nebo dvojrozměrný graf. Stav **Ano** znamená dosažitelnou smysluplnou nativní 2D cestu, případně **s konkrétním omezením uvedeným v tabulce**. **Ne** znamená, že běžné 2D workflow nefunguje. Důsledně odlišujeme **specifický zákaz 2D** od **obecného defektu platného i ve 3D**.

Počet položek: **56** (41 Ano s případnými omezeními, 15 Ne). `BSF` je společné schéma pro runnable `BSFEK` / `BSFKK`, nikoli samostatně spouštěný task; počítáme ho jako položku inventáře.

## 3D funguje, nativní 2D ne

| Task | 3D | 2D | Přímý důkaz |
|---|---|---|---|
| `POSANIPREP` | Ano v kompatibilním 3D workflow | **Ne** | `gen.f:435–455` volá `POSANIPOT`; `posanipot.f:88–90` má `STOP_MESSAGE('SYSTEM_DIMENSION.NE.3D')`. |
| `POSANITAU` | Ano v kompatibilním 3D workflow | **Ne** | Stejné povinné volání `POSANIPOT`; ve 2D explicitní STOP. |
| `CHILANDAU` | Ano v omezeném 3D serial workflow | **Ne** | `susc_landau.f:221–226` volá `TAU_STD`, nikoli nativní 2D `TAU_TB`. |
| `POSANI` | Ano s připravenými .pan daty | **Ne pro standardní workflow** | `POSANI` sice samo `POSANIPOT` nevolá, ale požaduje .pan data vytvářená `POSANIPREP`; externě připravená data představují zvláštní postprocessing, nikoli plnou 2D podporu. |

**Žádný další z ostatních 11 položek s výsledkem Ne není prokázanou výjimkou „funguje ve 3D, ale ne ve 2D“:** jde o obecné nativní chyby nebo nedokončenou deklarovanou funkcionalitu.

## Společná pravidla a konkrétní kombinace

- `init_mod_calcmode.f:609–634`: `SYSTEM_DIMENSION=2D` dovoluje pouze `TAU.KKRMODE=TB-KKR` nebo `LAYER-KKR`. Implicitně se pro 2D nastaví `TB-KKR`. Nelze použít `STANDARD-KKR`.
- `rsp.f:98–113`, `gil.f:172–249`, `opt_abs.f:637–798`, `sig.f:713–844`: pro příslušné response úlohy jsou větve `TB-KKR`, ale **ne `LAYER-KKR`**. Samotné `supports_2d=True` tedy nestačí bez validace skutečného `KKRMODE`.
- `tauij_drive.f:44–93`: `JXC` a `DMI` používají v 2D `TAUIJ_TB`, nikoli `TAUIJ_LKKR`; pro periodicitu vyžadují `IBZINT=2`, magnetický FREL systém a `KMROT=0`.
- `tau_drive.f:47–54`: `THERMAL_VIBRA_FLUCT` ve všeobecném TAU řešiči vyžaduje `STANDARD-KKR` nebo `REAL-SPACE` a s normálním 2D `TB-KKR`/ `LAYER-KKR` nelze kombinovat.
- `scf_drive.f:61–68`: `SCF.SCL_ALAT` je **pouze 3D**, přestože samotný SCF podporuje 2D.
- `spec_rslab.f:134–195,262–300`: `KKRSPEC` v 2D s `STRVER=0` přijímá externí structure file pro typy `VIR`, `LIV`, `VIV`, `LIR`; s `STRVER=1` má automatickou konstrukci pouze pro `VIR` nebo `LIV` (skutečná podmínka v kódu je užší než komentář).
- `blochsf.f:274–282,431–437` / `ekrel.f:249–260`: explicitní rozlišení 2D k-prostoru.
- `fpplot2dpr.f:287–404` má explicitní 2D geometrickou větev; `plot2dpr_asa.f:123–130` naopak používá také translace třetího vektoru, což může být pro nativní 2D plotovací oblast fyzikálně zavádějící. `PLOT2D*` označuje rovinu grafu, **nikoli** doklad podpory 2D systému.
- `XAS` používá `VOLUC*ALAT**3` pro normalizaci: ve 2D je nutno kontrolovat interpretaci objemové veličiny; nenahrazuje to zákaz běhu.

## Přehled všech 56 položek

| Task | Nativní 2D | Omezení / důvod |
|---|---|---|
| `AES` | **Ano** | ASA; NQ=1; bez FULLPOT. |
| `AIPES` | **Ano** | KKRSPEC; omezení STRVER pro 2D. |
| `APS` | **Ano** | Kompatibilní starý DOS; kladná GAMMA je problematická. |
| `ARPES` | **Ano** | KKRSPEC; omezení STRVER pro 2D. |
| `BAND` | **Ano** | KKRSPEC; omezení STRVER pro 2D. |
| `BONDING` | **Ne** | Obecně rozbitá příprava TAUIJ (i ve 3D). |
| `BSDOSIJ` | **Ne** | Obecně chybné indexování (i ve 3D). |
| `BSF` | **Ano** | Sdílené schéma BSFEK/BSFKK; BLOCHSF má 2D cestu. |
| `BSFEK` | **Ano** | 2D větev BLOCHSF; energetický scan. |
| `BSFKK` | **Ano** | 2D větev BLOCHSF; k-scan. |
| `CHI` | **Ano** | Pouze TB-KKR, ne LAYER-KKR; ne FULLPOT. |
| `CHILANDAU` | **Ne** | Specificky nepodporuje 2D: SUSC_LANDAU volá TAU_STD. |
| `CHIXAS` | **Ano** | TB-KKR; kompatibilní externí NEF/CHISPN/CHIVV; ne FULLPOT. |
| `CLPES` | **Ano** | KKRSPEC; bez CPA; omezení STRVER. |
| `CLXPS` | **Ano** | ASA/FREL; vhodné core states. |
| `COMPTON` | **Ano** | Používá TAU; SQQ_RESOLVED nepracuje s MD2D/EMD2D; 2D výstup není 2D geometrie. |
| `DMI` | **Ano** | TB-KKR; IBZINT=2; FREL; KMROT=0; magnetický systém; ne LAYER-KKR. |
| `DOS` | **Ano** | Obecná 2D cesta přes TAU_DRIVE. |
| `EKREL` | **Ano** | Explicitní 2D geometrie; bez CPA. |
| `FMAG` | **Ano** | Nenulové momenty pro normalizaci; běžná 2D GEN cesta. |
| `FSCAT` | **Ne** | Obecná chyba indexování harmonických. |
| `GILBERT` | **Ano** | TB-KKR; KMROT=0; IBZINT=2 nebo 6; ne LAYER-KKR, ne FULLPOT. |
| `JALF` | **Ano** | Standardní 2D GEN; známé chyby pomocné diagnostiky. |
| `JORB` | **Ano** | Standardní 2D GEN; používat serial, MPI dokončení je vadné. |
| `JXC` | **Ano** | TB-KKR; IBZINT=2; FREL; KMROT=0; magnetický systém; ne LAYER-KKR. |
| `MECHECK` | **Ano** | Lokální kontrola elementů; omezené nativní testy. |
| `MEPLOT` | **Ano** | Nekombinovat photon a electron scan. |
| `NRAES` | **Ne** | Obecně vadná inicializace akumulace. |
| `NUCSSCPL` | **Ne** | Obecné chyby alokace a maticových elementů. |
| `OPTABS` | **Ano** | TB-KKR a ASA; TEMPEL>1; NOMEGA>=NEB; ne LAYER-KKR. |
| `PHAGEN` | **Ano** | Pouze export potenciálu; konvergovaný ASA/VWN; NT=1. |
| `PLOT2DPOT` | **Ano** | ASA plot může použít 3D periodické kopie; FULLPOT PLY je experimentální. |
| `PLOT2DPR` | **Ano** | ASA vykreslí jen potenciál; FULLPOT má 2D geometrickou větev, ne kompletní grid. |
| `PLOT2DRHO` | **Ne** | Obecně nedokončené: ASA bez hustoty, FULLPOT bez standardního výstupu. |
| `PLOTPOT` | **Ano** | Graf uloženého potenciálu; bez nové self-consistency. |
| `PLOTPRS` | **Ano** | Graf uložených radiálních dat; známé kolize spin/orbital křivek. |
| `PLOTRHO` | **Ne** | Obecná chyba nepřipraveného RHORSQ. |
| `PLOTSFN` | **Ano** | Jen potenciál s FULLPOT shape functions. |
| `POSANI` | **Ne** | Standardní 2D workflow nemá funkčního producenta .pan; s externími daty může doběhnout postprocessing. |
| `POSANIPREP` | **Ne** | Specificky 3D: povinný POSANIPOT explicitně odmítá 2D. |
| `POSANITAU` | **Ne** | Specificky 3D: povinný POSANIPOT explicitně odmítá 2D. |
| `PSHIFT` | **Ano** | ASA; odvozená rezonanční čísla mají známá omezení. |
| `SCF` | **Ano** | 2D SCF funguje; SCF.SCL_ALAT je pouze pro 3D. |
| `SIGMA` | **Ano** | TB-KKR Kubo; ne LAYER-KKR; SIGMA_LAYER není ve veřejné verzi dostupné. |
| `SOCPAR` | **Ano** | ASA; exchange část má chybu popisu jednotek. |
| `SPLEED` | **Ano** | KKRSPEC; omezení STRVER pro 2D. |
| `STONER` | **Ano** | 2D GEN; pevný VWN kernel a rizika normalizace. |
| `T1` | **Ne** | Obecně rozbité: nonrel STOP, relativistická větev numericky chybná. |
| `TDOBS` | **Ne** | Obecně chybí samostatný výpočet time-dependent observable. |
| `TORQUE` | **Ano** | Vyžaduje vhodně rotovaný magnetický potenciál; nekombinovat s JXC. |
| `TUTORIAL` | **Ano** | Závisí na TUT-TASK; matematické demonstrace jsou bez geometrie, DOS submódy dle GEN. |
| `VBPES` | **Ano** | FREL a konzistentní initial/final TAU; není totožné s KKRSPEC ARPES. |
| `WFPLOT` | **Ano** | ASA/FREL; FULLPOT není podporován. |
| `XAS` | **Ano** | FREL, serial; konzistentní TAU a energy mesh; zkontrolovat normalizaci objemem. |
| `XES` | **Ne** | Obecně chybí ověřitelná samostatná emisní větev RXES; sdílí absorpční cestu. |
| `XRS` | **Ne** | Obecná chyba inicializace SIG; výsledek nelze považovat za spolehlivý. |

## Důsledky pro metadata ASE2SPRKKR

Doporučení: rozlišit **typ výpočetního systému**, ne jen počet periodických rozměrů. Nabídnout `supported_systems` (nebo zkráceně `supports`) jako množinu explicitních kategorií `{"3d", "2d", "cluster", "embedded"}`.

- `3d` = 3D periodický systém; `2d` = nativní vrstvová/povrchová geometrie.
- `cluster` = izolovaný konečný cluster bez hostitelského potenciálu (0D); `embedded` = cluster s hostitelským potenciálem. Tyto dvě kategorie neslučovat do `0d`.
- `embedded` může mít hostitelský potenciál odvozený od 3D nebo 2D hosta; rozměr hosta patří do samostatného metadata a případného validátoru.
- **`3d` zapisovat vždy explicitně:** absence kategorie znamená nepodporovaný typ; neexistuje skrytý „3D default“.
- Pravidla `KKRMODE`, `IBZINT`, `SCL_ALAT`, `STRVER` a ostatní kombinace **nezakódovat jen do množiny**; patří do validace dostupnosti tasku pro konkrétní `atoms` + parametry. `BSF` je schéma, vlastnosti patří na `BSFEK`/`BSFKK`.
- Položky s obecnými chybami **nesmějí být vykládány jako fyzikální zákaz 2D**; oddělit `supported_systems` od známého stavu implementace. Toto shrnutí samo nepovoluje žádnou novou cluster/embedded kombinaci nad rámec samostatného [embedding auditu](ASE2SPRKKR-task-interface-audit.md).

Příklad pro diskusi (není provedena úprava Python API):

```python
class SystemKind(StrEnum):
    PERIODIC_3D = "3d"
    LAYERED_2D = "2d"
    FREE_CLUSTER = "cluster"
    EMBEDDED_CLUSTER = "embedded"

# SCF: supports={"3d", "2d", "embedded"} # plus případný cluster až po samostatném auditu
# POSANIPREP: supports={"3d"}           # nikoli "2d"
# CHILANDAU: supports={"3d"}            # nikoli "2d"
# Konkrétní zákazy kombinací ověřuje validator.
```


Podpora volného clusteru bez host potenciálu je vyhodnocena samostatně v [0D auditu](SPRKKR-0D-free-cluster-task-applicability.md). Samotné `TAU.CLUSTER` není ekvivalent molekulárního `TAU.MOL`.
