# ASE2SPRKKR — audit rozhraní tasků, parametrů a dokumentace

**Aktualizace:** 13. listopadu 2026  
**Revize podpory embeddingu a executable:** 18. listopadu 2026 (nativní zdroje SPR-KKR 9.7, `ase2sprkkr` větev `cluster`)  
**Referenční funkční audit:** [Souhrnný audit SPR-KKR 9.7 a ASE2SPRKKR](../SPRKKR-ASE2SPRKKR-common-audit.md) (9. listopadu 2026) a navazující jednotlivé audity.  
**Základ implementace:** veřejné zdroje SPR-KKR 9.7 `PUB9.7_260420_18h13`; referenční funkční audit kontroloval ASE2SPRKKR `develop` na `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. Aktuální registry, schémata a některé souborové kontrakty jsem navíc ověřil vůči `develop` `8aa03363433b45b98e2594bfaecc7578a46be94c` (12. listopadu 2026). **Celá testovací sada nebyla nad novější revizí znovu spuštěna.**  
**Podklad manuálu:** veřejný SPR-KKR User's Guide (starší verze 8.6 použitá v původním auditu); zařazení parametrů je **předběžné**, pokud jednotlivý audit neodkazuje přímo na odpovídající část manuálu.

## Účel a interpretace

Jde o **audit rozhraní ASE2SPRKKR**, který doplňuje existující [audit zdrojových chyb a funkcionality](../SPRKKR-ASE2SPRKKR-common-audit.md), nikoli jej nahrazuje. U každého tasku uvádí účel, stručné omezení, soubory, task-specifické parametry a jejich pokrytí dostupným manuálem. Detailní rozbor chyb zůstává v odkazovaných podkladech.

- **Seznam:** 56 položek včetně `BSF`, což je pouze **schéma**, nikoli samostatně registrovaný task; referenční audit uvádí **55 registrovaných identit**. `BSFEK`/`BSFKK` sdílejí schéma BSF, `DMI` má vlastní identitu se schématem JXC. Nezaregistrované nativní názvy (např. BIS, SIGMA-XAS) jsou vysvětlené ve funkčním auditu, nikoli předstírané jako spustitelné tasky.
- **Vstupní soubory:** běžně jde o generovaný SPR-KKR vstup (standardní vstup procesu) a potenciál z `CONTROL.POTFIL`. Rozlišuji externí povinné soubory, volitelnou cache a pomocné soubory pro historické formáty. Umístění TAU cache může záviset na `TAUDIR`/`TAUPATH`.
- **Výstupní soubory:** tam, kde je známý konkrétní kontrakt, uvádím skutečné názvy. `{DATASET}` je původní nativní `DATSET0`, `{DATASET_ADSI}` je `DATASET` s případným neprázdným `ADSI`. Ne všechny nativní soubory jsou v manifestu a ne každý má strukturovaný parser. Pevné názvy `fort.*`, PLY, `sigma?????.dat` či `arpes.dat` mohou mezi běhy kolidovat.
- **Parametry:** dvě skupiny uvádějí **task-specifické** klíče a změny společných parametrů; neopisuji všechny sdílené sekce `CONTROL/TAU/ENERGY/MODE/CPA/STRCONST/SITES`. „Nepokryto manuálem“ znamená pouze to, že popis nebyl prokázán v dostupné verzi; neznamená neplatný parametr. Původní rozdělení zachovávám kvůli dohledatelnosti, nevydávám je za novou úplnou kontrolu manuálu.
- **Vyzkoušenost:** existence definice, Python regresní testy, nativní výpočet, obsluha souborů, parser a numerická správnost jsou odlišné úrovně. Referenční testy z 9. listopadu uvádějí **1249 passed, 1 skipped** pro input-parameters/result-files; **nepotvrzují numerickou správnost**. Samostatný nativní test **PHAGEN** kontroloval exportované soubory (32 úspěšných cílených testů), nikoli shodu s externím MsSpec. Pro ostatní tasky tento audit neprokazuje nezávislý numerický referenční výpočet.
- **Stav:** krátké praktické shrnutí **převzaté z funkčního auditu**. Příčiny chyb a přesné omezené větve jsou popsány v odkazovaném auditu.

## Podpora embeddingu — revize 17. 11. 2026

Tato revize samostatně klasifikuje **nativní podporu embeddingu v SPR-KKR 9.7** (`PUB9.7_260420_18h13`), nikoli automaticky dostupnost či otestovanost workflow v aktuálním ASE2SPRKKR/GUI. Srovnání je na úrovni skutečných Fortranových dispatch větví a jejich nároků na vstup, host potential a off-diagonální TAU; nejde o numerickou certifikaci.

- **Ano:** existuje explicitní embeddingová výpočetní cesta (specializovaná větev/clusterová rutina), podle zdroje dosažitelná při splnění jejích předpokladů.
- **Podmíněně:** task či jeho pomocná rutina je z embeddingového `CLUGEN` dosažitelná, ale je omezena režimem, přítomností příslušných dat nebo nemá samostatně ověřenou interpretaci výsledku pro host+cluster. **Neznamená otestovanou podporu.**
- **Ne:** zdrojový kód embedding výslovně odmítá, task používá nekompatibilní samostatný driver bez clusterové alternativy, případně má doloženou nativní chybu znemožňující funkční výsledek.
- **Nedoloženo:** obecné volání `GEN` sice může dojít k rutině, ale z prohlédnutých zdrojů nelze potvrdit kompletní embeddingové vstupy, výstupy nebo fyzikální význam. Bez samostatného ověření to **není souhlas s podporou**; zároveň to není tvrzení, že funkčnost je nemožná.

**Nejdůležitější doklady:** `kkrmain.f:300–307,1392–1447` kontroluje `CLU*` při dalších clusterových uzlech a odděluje `KKRSCF`, `KKRGEN`, `KKRCHI`, `CLUSCF/CLUGEN`, `KKRSPEC`; `cludrive.f:398–413,619–664` nastavuje `SYSTEM_TYPE=EMBEDDED-CLUSTER`, spouští `SCF`/`GEN` a pro `CHI` používá `CLU_CHI_DRIVE`; `gen.f:352–365,923–1002` rozlišuje clusterové JXC/DMI a jejich TAU; `cluxcpltensor.f:63–69` omezuje relativistický tensor; `clu_chi_drive.f:39–49,79–95` zastavuje embedded CHI a neimplementuje `GIL`; `initvar.f:160–184,342–354` obsahuje výslovná omezení MECHECK, PHAGEN, OPTABS a SIGMA.

**Poznámka ke spuštění:** běžný task, který funguje pod `KKRGEN`, tím ještě není podporovaný v embeddingu. Navíc kompatibilita `CLUGEN`/`CLUSCF` vyžaduje host potential, správně připravenou geometrii clusteru a host `TAUIJ` (`cludrive.f:398–413,619–629,735–830`).

### Výběr executable

**Výsledek hloubkové kontroly 18. 11. 2026:** Mezi sedmi dříve nedoloženými tasky jsou `APS`, `COMPTON`, `POSANI`, `POSANIPREP` a `VBPES` nově **Ne** kvůli neslučitelnému původnímu DOS, periodickým k-prostorovým operátorům či chybějící embedded final-state cestě. `POSANITAU` je **Podmíněně**: `posanipot.f` obsahuje explicitní clusterovou větev, ale fyzikální normalizace do hostitele není doložena. `STONER` je nově **Podmíněně**: cache-miss v `kkrmain.f:932–959` přepne `RDTAUDIR` na `.FALSE.`, čímž je dostupná výpočetní cesta přes embedded `TAU_DRIVE`; ovšem `CALCDOS` naplňuje jen clusterové typy a `SUSC_GAMMA` nemá pro tento případ doložené zacházení s hostitelskými typy. Výpočet není numericky potvrzen a v `ase2sprkkr` zůstává blokován. Speciální executable zůstává mimo `ase2sprkkr` povolený jen pro explicitní tasky `SCF`, `DOS`, `JXC`, `DMI`.

Pro běžné výpočty se používá executable uvedený v registru tasků. Skutečný embedding vyžaduje **speciální** program `embscf` (`CLUSCF`) nebo `embgen` (`CLUGEN`), nikoli generický `kkrscf`/`kkrgen`. `KKRSPEC` a `KKRCHI` nemají ekvivalentní funkční embeddingový driver. U položek **Podmíněně**/**Nedoloženo** znamená uvedený `embgen?` pouze hypotetickou vstupní cestu přes `CLUGEN`, nikoli aktuálně povolený executable v `ase2sprkkr`. `BSF` je pouze schéma, které používají aliasy `BSFEK`/`BSFKK`.

**Rozšířená kontrola 18. 11.:** `PLOTRHO`, `NRAES` a `T1` jsou nově **Ne** kvůli doloženým nativním chybám nezávislým na embeddingu. FREL JXC/DMI ve skutečnosti zapisují `*_JJij.dat` a `*_Dij_[xyz].dat`, nikoli periodická `XCPLTEN_*`/`DMIVEC_*`; pravidlo `TAU.BZINT=POINTS` se vztahuje k hostitelským `TAUIJ` a nemá se bez ověření měnit na `CLUSTER`. SP-SREL cluster JXC není zatím bezpečně povolen.

## Index

| Task | Stručný stav | Embedding (SPR-KKR 9.7) | Pokrytí manuálem | Executable (běžný → embedding) |
|---|---|---|---|---|
| [`AES`](#aes) | Funguje jen v omezeném auditovaném workflow; NQ=1, bez f-shellů a s riziky v cache/TAU kapacitě. Numerický solver nebyl v auditu spuštěn. | **Ne** | Ano. | `kkrgen` → — |
| [`AIPES`](#aipes) | Funguje v auditovaném fixed-geometry workflow; není to numerická integrace ARPES přes úhly. | **Ne** | Ano. | `kkrspec` → — |
| [`APS`](#aps) | Funguje s kompatibilním starým DOS; pozitivní `GAMMA` broadening má nativní problematickou větev. | **Ne** | Ano. | `kkrgen` → — |
| [`ARPES`](#arpes) | Nativní ARPES workflow je v auditovaném rozsahu smysluplné. Současné rozhraní používá správné presence-sensitive `TASK.CRYS_VECS`; pokročilé numerické větve nejsou ověřené. | **Ne** | Ano. | `kkrspec` → — |
| [`BAND`](#band) | Funguje v auditovaném workflow; selhání eigen-solveru může nativně pokračovat až k `LOG(0)`. Není to `BSF` ani `EKREL`. | **Ne** | Ano. | `kkrspec` → — |
| [`BONDING`](#bonding) | Rozbité: běžný `TAU_DRIVE` připraví `KTAUIJ=0`, ale `CLUBONDING` čte scratch jednotky vyžadující `KTAUIJ>=2`. | **Ne** | Ne. | `kkrgen` → — |
| [`BSDOSIJ`](#bsdosij) | Rozbité: nativní `J_ITJT` je zapisováno energetickými indexy do pair dimenzí a později čteno v jiném pořadí. | **Ne** | Ne. | `kkrgen` → — |
| [`BSF`](#bsf) | Pouze schéma, ne task. Funguje; E-k default `EMAX=-1 Ry` je podezřelý convenience default a nemusí znamenat EF. | **Ne** | Ano. | — (BSF schema; BSFEK/BSFKK: kkrgen) |
| [`BSFEK`](#bsfek) | Funguje stejně jako `BSF`; alias pouze předvolí očekávaný E-k režim přes `ENERGY.NE`. | **Ne** | Ano, jako `BSF`; jméno `BSFEK` je ASE2SPRKKR alias. | `kkrgen` → — |
| [`BSFKK`](#bsfkk) | Funguje stejně jako `BSF`; alias pouze předvolí očekávaný single-energy k-k režim. | **Ne** | Ano, jako `BSF`; jméno `BSFKK` je ASE2SPRKKR alias. | `kkrgen` → — |
| [`CHI`](#chi) | Funguje v auditovaném statickém workflow; nativní `NCHIITER` je před smyčkou natvrdo resetováno na 1, takže iterativní magnetizační část ve skutečnosti neběží. | **Ne** | Ano. | `kkrchi` → — |
| [`CHILANDAU`](#chilandau) | Funguje sériově s omezeními; MPI a reuse `RDTAUFIL` jsou rozbité a nativní finish logic vypisuje mezisoučty. | **Ne** | Ne. | `kkrchi` → — |
| [`CHIXAS`](#chixas) | Funguje jen při fyzikálně konzistentních externích `NEF/CHISPN/CHIVV`; univerzální bezpečné defaulty neexistují. | **Ne** | Ne. | `kkrchi` → — |
| [`CLPES`](#clpes) | Funguje v auditovaném sériovém fixed-geometry workflow; CPA a některé polarizační režimy nativně stopnou, při `NE=1` je nutné hlídat core-state index. | **Ne** | Ne. | `kkrspec` → — |
| [`CLXPS`](#clxps) | Funguje v auditovaném ASA/FREL workflow; advanced angular geometries a numerický output nebyly samostatně certifikovány. | **Podmíněně** | Ano. | `kkrgen` → `embgen?` |
| [`COMPTON`](#compton) | Funguje s omezeními; `EMD2D` má při nenulovém `PNMAX` dvojí škálování a posunuté sudé mřížky ztrácejí krajní body. | **Ne** | Ano. | `kkrgen` → — |
| [`DMI`](#dmi) | Funguje s významnými ALE: jednotlivé DMI komponenty nejsou v nativním zdroji ekvivalentní (zejména Dz) a je nutné ověřovat frame/multiplicity/radius. | **Ano** | Ne jako samostatný task v dostupném manuálu; JXC je popsán. | `kkrgen` → `embgen` |
| [`DOS`](#dos) | Funguje. | **Ano** | Ano. | `kkrgen` → `embgen` |
| [`EKREL`](#ekrel) | Funguje; CPA je nativně odmítnuto, pro disorder se má použít BSF. | **Ne** | Ano. | `kkrgen` → — |
| [`FMAG`](#fmag) | Funguje s omezením: nulový magnetický moment může při normalizaci vést k dělení nulou. | **Podmíněně** | Ano. | `kkrgen` → `embgen?` |
| [`FSCAT`](#fscat) | Rozbité: chybný spherical-harmonic index `+L` může přetéct a fáze se počítá před ošetřením nulové amplitudy. | **Ne** | Ne. | `kkrgen` → — |
| [`GILBERT`](#gilbert) | Funguje pro vhodné magnetické systémy; nulové nebo kompenzované momenty mohou způsobit dělení nulou. | **Ne** | Ano. | `kkrchi` → — |
| [`JALF`](#jalf) | Funguje s omezeními; helper diagnostiky mají chyby a malý `NL` je rizikový, což definice hlídá. | **Podmíněně** | Ne. | `kkrgen` → `embgen?` |
| [`JORB`](#jorb) | Funguje sériově; nativní MPI workflow nedojde k finálnímu JORB výstupu a neredukuje akumulátory. | **Podmíněně** | Ne. | `kkrgen` → `embgen?` |
| [`JXC`](#jxc) | Funguje pro jednoduchý periodický kolineární workflow; obecný FREL tensor se symetricky ekvivalentními sousedy má nativní rotační bug a cluster SP-SREL scratch flow je nespolehlivý. | **Ano** | Ano. | `kkrgen` → `embgen` |
| [`MECHECK`](#mecheck) | Funguje jen částečně: `NMEFORM` je nativně vždy přepsáno na 2, další consistency test je compile-time vypnutý a summary loop má chybu. | **Ne** | Ne. | `kkrgen` → — |
| [`MEPLOT`](#meplot) | Funguje, až na nativní přepisování křivek při současném photon+electron scanu; definice tuto kombinaci odmítá. Význam `SEFIN` potřebuje fyzikální ověření. | **Podmíněně** | Ne. | `kkrgen` → `embgen?` |
| [`NRAES`](#nraes) | Rozbité: `IAES` je jen částečně nulováno a convolution akumuluje do neinicializovaných oblastí. | **Ne** | Ano. | `kkrgen` → — |
| [`NUCSSCPL`](#nucsscpl) | Rozbité: nativní kód deallocuje pole před první alokací, vynuluje matrix elements pro každý type a nutí mesh na 1. | **Ne** | Ne. | `kkrgen` → — |
| [`OPTABS`](#optabs) | Funguje s velkými ALE: `TEMPEL<=1` nechává `KBT` neinicializované a `NOMEGA<NEB` může přepsat alokaci; definice tyto stavy zakazuje. | **Ne** | Ne. | `kkrchi` → — |
| [`PHAGEN`](#phagen) | Funguje jako omezený ASA/VWN converter; NT=1, bez disorder, ne jako obecný phase-shift solver. | **Ne** | Ne. | `kkrscf` → — |
| [`PLOT2DPOT`](#plot2dpot) | Funguje v ASA pro potential; FULLPOT větev jde do experimentálního PLY writeru a skončí před běžným plane-grid outputem. | **Podmíněně** | Ne. | `kkrgen` → `embgen?` |
| [`PLOT2DPR`](#plot2dpr) | Částečně funguje: v ASA ve skutečnosti zapisuje jen potential; FULLPOT větev skončí v experimentálním PLY writeru. | **Podmíněně** | Ne. | `kkrgen` → `embgen?` |
| [`PLOT2DRHO`](#plot2drho) | Prakticky neúplné: ASA density neumí; FULLPOT skončí v testovacím PLY writeru před běžným plane-grid writerem. | **Podmíněně** | Ne. | `kkrgen` → `embgen?` |
| [`PLOTPOT`](#plotpot) | Funguje pro ASA i FULLPOT stored potential data. | **Podmíněně** | Ne. | `kkrgen` → `embgen?` |
| [`PLOTPRS`](#plotprs) | Funguje s omezeními: magnetic spin/orb curves mohou kolidovat ve stejném datasetu a panel limits ignorují orbital contribution. | **Podmíněně** | Ne. | `kkrgen` → `embgen?` |
| [`PLOTRHO`](#plotrho) | Rozbité: `RHORSQ` se alokuje jen v potential branch, ale standalone rho writer ho potřebuje. | **Ne** | Ne. | `kkrgen` → — |
| [`PLOTSFN`](#plotsfn) | Funguje pro skutečný FULLPOT potential se shape-function daty. | **Podmíněně** | Ne. | `kkrgen` → `embgen?` |
| [`POSANI`](#posani) | Funguje jen s kompatibilním `POSANIPREP`; nativní data path předpokládá `NQ=NT` a obecný `NQ!=NT` workflow je rozbitý. | **Ne** | Ano. | `kkrgen` → — |
| [`POSANIPREP`](#posaniprep) | Funguje pro jednoduchý 3D ASA/FREL `NQ=NT`; export TAUQQ je pro obecné `NQ!=NT` rozbitý. Energii dna pozitronového pásu si uživatel musí zvolit. | **Ne** | Ano. | `kkrgen` → — |
| [`POSANITAU`](#posanitau) | Funguje v auditovaném 3D ASA/FREL workflow; integrační energy bounds je třeba fyzikálně ověřit a enhancement factor je nativně fixní. | **Podmíněně** | Ne. | `kkrgen` → `embgen?` |
| [`PSHIFT`](#pshift) | Funguje, ale resonance splitting je v některých případech nespolehlivý. | **Podmíněně** | Ano. | `kkrgen` → `embgen?` |
| [`SCF`](#scf) | Funguje pro `BROYDEN2`, `ANDERSON` a `TCHEBY` v povoleném scope; `PULAY` není validní a `TCHEBY` není pro FULLPOT. | **Ano** | Ano. | `kkrscf` → `embscf` |
| [`SIGMA`](#sigma) | Funguje s omezeními; Fermi-sea není zahrnuta automaticky a režimy používají fixní file conventions/ImE. | **Ne** | Ano. | `kkrchi` → — |
| [`SOCPAR`](#socpar) | Funguje pro hlavní SOC výstup; exchange-related output má chybný label jednotek a jeden sign-flip test je nativně nedosažitelný. | **Podmíněně** | Ano. | `kkrgen` → `embgen?` |
| [`SPLEED`](#spleed) | Funguje v konzervativním fixed-angle workflow; momentum-scan a alternativní polarizační větve nejsou auditovány. | **Ne** | Ano. | `kkrspec` → — |
| [`STONER`](#stoner) | Funguje s omezeními; používá natvrdo VWN susceptibility kernel, malé DOS hodnoty nativně nahrazuje jednotkovým příspěvkem a nulové normalizace nejsou vždy ošetřeny. | **Podmíněně** | Ne. | `kkrgen` → `embgen?` |
| [`T1`](#t1) | Rozbité pro numerický účel: nativní `RELT1` používá signed κ jako kladné počty mj kanálů, takže záporné kanály mizí a basis indexing je chybný. | **Ne** | Ne. | `kkrgen` → — |
| [`TDOBS`](#tdobs) | Deklarovaný účel není prokázaný: nenašel se samostatný time-dependent observable implementation; jako generic DOS path může běžet sériově. | **Ne** | Ne. | `kkrgen` → — |
| [`TORQUE`](#torque) | Funguje, ale vyžaduje rotated magnetic potential/geometry; běžný nerotovaný magnetic potential může nativně STOPnout a mixed occupancy používá první type. | **Podmíněně** | Ano. | `kkrgen` → `embgen?` |
| [`TUTORIAL`](#tutorial) | Smíšený stav: YLM/JLX/JLX-ASYMPT a DOS-FEG/DOS-SS/DOS-CLU/DOS-BULK/GEN jsou použitelné; CHR-POT, POT-DECOMP/POT-XC, RWF-L/RWF-E a CORE-ALL mají specifické nativní chyby/omezení. | **Ne** | Ne. | `kkrgen` → — |
| [`VBPES`](#vbpes) | Funguje s omezeními; VBUPS si nativně automaticky nevolí UV photon energy a výpočet závisí na kompatibilním TAU/final-state setupu. | **Ne** | Ano. | `kkrgen` → — |
| [`WFPLOT`](#wfplot) | Funguje v ASA/FREL serial workflow; BAND používá jen první energy point, FULLPOT není podporován. | **Podmíněně** | Ano. | `kkrgen` → `embgen?` |
| [`XAS`](#xas) | Funguje se známými ALE; static branch je použitelný, ale TAU compatibility a efektivní začátek energy scanu je nutné kontrolovat. Native MPI končí před XRAYSPEC. | **Podmíněně** | Ano. | `kkrgen` → `embgen?` |
| [`XES`](#xes) | Ne zcela ověřené: RXES skončí ve sdílené absorption-rate větvi a output naming/interpretace je sdílená s XAS; fyzikální emission význam nebyl potvrzen. | **Podmíněně** | Ano. | `kkrgen` → `embgen?` |
| [`XRS`](#xrs) | Rozbité pro spolehlivý numerický výsledek: nativní `SIG(NEMAX+50,...)` je nulováno jen do `NEMAX`, ale později se sčítá přes celý rozsah. | **Ne** | Ano (v manuálu je XRS alespoň jako podporovaný spectroscopy task; detailní 9.7 interface je širší). | `kkrgen` → — |

## Tasks

### AES

- **Co dělá:** Počítá plně relativistické Augerovo elektronové spektrum z existujícího elektronového TAU a maticových elementů.
- **Stav:** Funguje jen v omezeném auditovaném workflow; NQ=1, bez f-shellů a s riziky v cache/TAU kapacitě. Numerický solver nebyl v auditu spuštěn.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — `AES` omezuje nativní výpočet na `NQ=1`, zatímco `CLUDRIVE` přidává hostitelské i clusterové uzly; `gen.f:1402–1413`, `cludrive.f:412–413`.
- **Executable (běžný / embedding):** `kkrgen` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** Generovaný SPR-KKR vstup, `POTFIL`, povinná kompatibilní TAU directory; volitelná cache se jmény `ME`, `ME1`…`ME12`.
- **Výstupní soubory:** `{DATASET_ADSI}<type>.<core-shell>.aes` (název závisí na typu a slupce); další diagnostické soubory mohou mít pevná jména.
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** `TASK.IT`, `TASK.CL`, `TASK.NEME`, `TASK.PRINTME`
- **Parametry nepokryté manuálem:** `TASK.PRINTAME`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_aes.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [AES](AES-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### AIPES

- **Co dělá:** Počítá nativní lokální/incoherentní úhlově integrovanou valenční fotoemisi se surface contribution.
- **Stav:** Funguje v auditovaném fixed-geometry workflow; není to numerická integrace ARPES přes úhly.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — AIPES je geometricky specifický task `KKRSPEC`; `CLUSCF`/`CLUGEN` volají pouze `SCF`/`GEN`, nikoli `SPECMAIN` (`kkrmain.f:1392–1447`).
- **Executable (běžný / embedding):** `kkrspec` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** Generovaný KKRSPEC vstup, `POTFIL`; podle přepínačů případně `input.inp` a `struc.inp`.
- **Výstupní soubory:** `{DATASET_ADSI}_SPEC.out` a polarizačně závislé soubory `{DATASET_ADSI}_data*.spc`.
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** `TASK.IQ_AT_SURF`, `TASK.MILLER_HKL`, `TASK.STRVER`, `TASK.INPVER`, `ENERGY.EWORK_EV`, `ENERGY.IMV_INI_EV`, `ENERGY.IMV_FIN_EV`, `SPEC_PH.THETA`, `SPEC_PH.PHI`, `SPEC_PH.POL_P`, `SPEC_PH.EPHOT`, `SPEC_EL.THETA`, `SPEC_EL.PHI`, `SPEC_EL.NT`, `SPEC_EL.NP`, `SPEC_EL.KA`, `SPEC_EL.K1`, `SPEC_EL.NK1`, `SPEC_EL.K2`, `SPEC_EL.NK2`, `SPEC_EL.POL_E`, `SPEC_EL.SPOL`, `SPEC_STR.N_LAYDBL`, `SPEC_STR.NLAT_G_VEC`, `SPEC_STR.N_LAYER`, `SPEC_STR.SURF_BAR`, `SPEC_STR.TRANSP_BAR`
- **Parametry nepokryté manuálem: `TASK.CRYS_VECS`, `SPEC_PH.ALQ`, `SPEC_PH.DELQ`, `SPEC_PH.NPOL`, `SPEC_PH.ICIRC`, `SPEC_PH.IDREH`, `SPEC_PH.IFSP`, `SPEC_PH.THETA_FIX`, `SPEC_EL.K3`, `SPEC_EL.NK3`, `SPEC_EL.K4`, `SPEC_EL.NK4`, `SPEC_EL.PSPIN`, `SPEC_EL.BETA1`, `SPEC_EL.BETA2`, `SPEC_EL.ROTAXIS`, `SPEC_EL.TYP`, `SPEC_EL.ISTR`, `SPEC_EL.POL0`, `SPEC_EL.POL0L`, `SPEC_EL.Q1`, `SPEC_EL.Q2`, `SPEC_EL.Q3`, `SPEC_EL.Q4`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_aipes.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** `SPEC` deklarovaný taskem (samotné parsování není fyzikální ověření).
- **Podrobný audit zdrojů/funkčnosti:** [AIPES](AIPES-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### APS

- **Co dělá:** Počítá appearance-potential spectrum postprocessingem dříve spočteného DOS.
- **Stav:** Funguje s kompatibilním starým DOS; pozitivní `GAMMA` broadening má nativní problematickou větev.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — `initvar.f:281–287` předepisuje čtení existujícího starého DOS a vypnutou energetickou smyčku, ale `cludrive.f:631–655` v `CLUGEN` násilně nastaví `RUNELOOP=.TRUE.` a znovu otevře a přepisuje `{DATSET}.dos` jako výstup. `NRAPS` právě z tohoto DOS čte (`nraps.f:829–863`). Nativní cluster driver tudíž neudržuje vstupní kontrakt APS; nejde o ověřenou embeddingovou úlohu.
- **Executable (běžný / embedding):** `kkrgen` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** Generovaný vstup + `POTFIL` + existující `{DATASET_ADSI}.dos` (starý spinově rozlišený formát a dostatečná kapacita).
- **Výstupní soubory:** `{DATASET_ADSI}.<type>.<core-shell>.aps`; pro `GAMMA>0` také `{DATASET_ADSI}.dos.brd`, případně soubory CALCLS/OUTGNU.
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** `TASK.IT`, `TASK.CL`, `TASK.GAMMA`, `TASK.OUTGNU`, `TASK.CALCLS`
- **Parametry nepokryté manuálem:** —

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_aps.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [APS](APS-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### ARPES

- **Co dělá:** Počítá úhlově rozlišenou fotoemisi pomocí KKRSPEC.
- **Stav:** Nativní ARPES workflow je v auditovaném rozsahu smysluplné. Současné rozhraní používá správné presence-sensitive `TASK.CRYS_VECS`; pokročilé numerické větve nejsou ověřené.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — Plná ARPES implementace je v `KKRSPEC`; cesta `CLU*` nevolá `SPECMAIN` (`kkrmain.f:1392–1447`). Přítomnost selektoru `ARPES` v `INITVAR` není embeddingová implementace.
- **Executable (běžný / embedding):** `kkrspec` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** Generovaný KKRSPEC vstup, `POTFIL`; při `TASK.INPVER=0` také `input.inp`, při `TASK.STRVER=0` také `struc.inp`.
- **Výstupní soubory:** `{DATASET_ADSI}_SPEC.out` a `{DATASET_ADSI}_data.spc`, případně dvojice `{DATASET_ADSI}_data_pol1.spc` / `_data_pol2.spc` podle polarizace.
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** `TASK.IQ_AT_SURF`, `TASK.MILLER_HKL`, `TASK.STRVER`, `TASK.INPVER`, `ENERGY.EWORK_EV`, `ENERGY.IMV_INI_EV`, `ENERGY.IMV_FIN_EV`, `SPEC_PH.THETA`, `SPEC_PH.PHI`, `SPEC_PH.POL_P`, `SPEC_PH.EPHOT`, `SPEC_EL.THETA`, `SPEC_EL.PHI`, `SPEC_EL.NT`, `SPEC_EL.NP`, `SPEC_EL.KA`, `SPEC_EL.K1`, `SPEC_EL.NK1`, `SPEC_EL.K2`, `SPEC_EL.NK2`, `SPEC_EL.POL_E`, `SPEC_EL.SPOL`, `SPEC_STR.N_LAYDBL`, `SPEC_STR.NLAT_G_VEC`, `SPEC_STR.N_LAYER`, `SPEC_STR.SURF_BAR`, `SPEC_STR.TRANSP_BAR`
- **Parametry nepokryté manuálem:** `TASK.CRYS_VECS`, `SPEC_PH.ALQ`, `SPEC_PH.DELQ`, `SPEC_PH.NPOL`, `SPEC_PH.ICIRC`, `SPEC_PH.IDREH`, `SPEC_PH.IFSP`, `SPEC_PH.THETA_FIX`, `SPEC_EL.K3`, `SPEC_EL.NK3`, `SPEC_EL.K4`, `SPEC_EL.NK4`, `SPEC_EL.PSPIN`, `SPEC_EL.BETA1`, `SPEC_EL.BETA2`, `SPEC_EL.ROTAXIS`, `SPEC_EL.TYP`, `SPEC_EL.ISTR`, `SPEC_EL.POL0`, `SPEC_EL.POL0L`, `SPEC_EL.Q1`, `SPEC_EL.Q2`, `SPEC_EL.Q3`, `SPEC_EL.Q4`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_arpes_audit.py`, `src/ase2sprkkr/input_parameters/test/test_arpes_energy.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** `ARPES` deklarovaný taskem (samotné parsování není fyzikální ověření).
- **Podrobný audit zdrojů/funkčnosti:** [ARPES](ARPES-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### BAND

- **Co dělá:** Počítá komplexní Blochovy bandy přes transfer-matrix eigenproblem v KKRSPEC.
- **Stav:** Funguje v auditovaném workflow; selhání eigen-solveru může nativně pokračovat až k `LOG(0)`. Není to `BSF` ani `EKREL`.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — Spektrální band task běží přes `KKRSPEC`, nikoli `CLUDRIVE`; nenavazuje na clusterové `TAU` (`kkrmain.f:1431–1447`).
- **Executable (běžný / embedding):** `kkrspec` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** Generovaný KKRSPEC vstup, `POTFIL`; podle přepínačů případně `input.inp` a `struc.inp`.
- **Výstupní soubory:** `{DATASET_ADSI}_SPEC.out`, `{DATASET_ADSI}_data.spc` (interpretace sloupců komplexních pásů vyžaduje kontrolu).
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** `TASK.IQ_AT_SURF`, `TASK.MILLER_HKL`, `TASK.STRVER`, `TASK.INPVER`, `ENERGY.EWORK_EV`, `ENERGY.IMV_INI_EV`, `ENERGY.IMV_FIN_EV`, `SPEC_PH.THETA`, `SPEC_PH.PHI`, `SPEC_PH.POL_P`, `SPEC_PH.EPHOT`, `SPEC_EL.THETA`, `SPEC_EL.PHI`, `SPEC_EL.NT`, `SPEC_EL.NP`, `SPEC_EL.KA`, `SPEC_EL.K1`, `SPEC_EL.NK1`, `SPEC_EL.K2`, `SPEC_EL.NK2`, `SPEC_EL.POL_E`, `SPEC_EL.SPOL`, `SPEC_STR.N_LAYDBL`, `SPEC_STR.NLAT_G_VEC`, `SPEC_STR.N_LAYER`, `SPEC_STR.SURF_BAR`, `SPEC_STR.TRANSP_BAR`
- **Parametry nepokryté manuálem:** `TASK.CRYS_VECS`, `SPEC_PH.ALQ`, `SPEC_PH.DELQ`, `SPEC_PH.NPOL`, `SPEC_PH.ICIRC`, `SPEC_PH.IDREH`, `SPEC_PH.IFSP`, `SPEC_PH.THETA_FIX`, `SPEC_EL.K3`, `SPEC_EL.NK3`, `SPEC_EL.K4`, `SPEC_EL.NK4`, `SPEC_EL.PSPIN`, `SPEC_EL.BETA1`, `SPEC_EL.BETA2`, `SPEC_EL.ROTAXIS`, `SPEC_EL.TYP`, `SPEC_EL.ISTR`, `SPEC_EL.POL0`, `SPEC_EL.POL0L`, `SPEC_EL.Q1`, `SPEC_EL.Q2`, `SPEC_EL.Q3`, `SPEC_EL.Q4`, `SPEC_BAND.NEIGMAX`, `SPEC_BAND.EIG_IME`, `SPEC_BAND.CALC_TRANS_PROB`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_band.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** `SPEC` deklarovaný taskem (samotné parsování není fyzikální ověření).
- **Podrobný audit zdrojů/funkčnosti:** [BAND](BAND-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### BONDING

- **Co dělá:** Má počítat experimentální bonding analýzu molekulárního clusteru.
- **Stav:** Rozbité: běžný `TAU_DRIVE` připraví `KTAUIJ=0`, ale `CLUBONDING` čte scratch jednotky vyžadující `KTAUIJ>=2`.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — `gen.f:1012–1025` umí zavolat `CLUBONDING` při `IBZINT=0`, ale původní source audit doložil nefunkční kontrakt `KTAUIJ`/scratch jednotek; nelze označit jako funkční embedding.
- **Executable (běžný / embedding):** `kkrgen` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** standardní vstup + `POTFIL`; nativně očekává interní scratch data jednotek 18/19, která běžný workflow nepřipraví.
- **Výstupní soubory:** `fort.22`.
- **V SPR-KKR manuálu:** Ne.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** —

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Rozbité nativní workflow.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_bonding.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [BONDING](BONDING-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### BSDOSIJ

- **Co dělá:** Má počítat site-resolved back-scattering DOS mezi páry míst.
- **Stav:** Rozbité: nativní `J_ITJT` je zapisováno energetickými indexy do pair dimenzí a později čteno v jiném pořadí.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — `CALC_BSDOSIJ` je čitelný v `initvar.f:534–551`, ale původní audit dokumentuje nesprávné indexování energetických/párových dimenzí; clusterový výsledek není spolehlivý.
- **Executable (běžný / embedding):** `kkrgen` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** Deklarován `{DATASET_ADSI}_XCPLTEN_Jij.dat`, ale obsah je kvůli nativní chybě indexace nespolehlivý.
- **V SPR-KKR manuálu:** Ne.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** `TASK.CLURAD`, `TASK.NSHLCLU`, `TASK.SELECT`, `TASK.EXCLUDE`, `TASK.QREPONLY`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Rozbité nativní workflow.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_bsdosij.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [BSDOSIJ](BSDOSIJ-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### BSF

- **Co dělá:** Počítá Blochovu spektrální funkci buď podél E-k dráhy, nebo v k-k rovině při jedné energii.
- **Stav:** Funguje; E-k default `EMAX=-1 Ry` je podezřelý convenience default a nemusí znamenat EF.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — `GEN` sice volá `BLOCHSF` (`gen.f:1499–1506`), ale jde o k-resolved Blochovský task bez doloženého zobrazení lokálního embeddingového clusteru; samotné volání `GEN` nestačí.
- **Executable (běžný / embedding):** nepoužívá se přímo; `BSFEK`/`BSFKK` volají `kkrgen`; embedding není podporován
- **Vstupní soubory:** standardní vstup + `POTFIL`; může reuse existující TAU cache.
- **Výstupní soubory:** `{DATASET_ADSI}.bsf`, volitelně `{DATASET_ADSI}_spol.bsf`.
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** `TASK.NK`, `TASK.KPATH`, `TASK.NKDIR`, `TASK.KA*`, `TASK.KE*`, `TASK.NK1`, `TASK.NK2`, `TASK.K1`, `TASK.K2`
- **Parametry nepokryté manuálem:** —

- **Stav rozhraní:** Pouze schéma, není v task registry; závěr zdrojového auditu: Smysluplné nativní workflow v uvedeném rozsahu.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_bsf_audit.py`, `src/ase2sprkkr/input_parameters/test/test_bsf_file_manifests.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** `BSF` deklarovaný taskem (samotné parsování není fyzikální ověření).
- **Detail rozhraní:** `BSF` je schéma, ne samostatně registrovaný task. Použij `InputParameters.for_task('BSFEK')` nebo `for_task('BSFKK')` (případně `for_definition('BSF')` pro schéma).
- **Podrobný audit zdrojů/funkčnosti:** [BSF](BSF-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### BSFEK

- **Co dělá:** ASE2SPRKKR alias pro `BSF` v E-k režimu.
- **Stav:** Funguje stejně jako `BSF`; alias pouze předvolí očekávaný E-k režim přes `ENERGY.NE`.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — Alias `BSF` (E–k); zdrojová větev `BLOCHSF` není doložená pro `EMBEDDED-CLUSTER` (`gen.f:1499–1506`).
- **Executable (běžný / embedding):** `kkrgen` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** Stejný vstup a potenciál jako schéma BSF; volitelná znovupoužitelná TAU cache.
- **Výstupní soubory:** `{DATASET_ADSI}.bsf`, volitelně `{DATASET_ADSI}_spol.bsf` (režim E–k).
- **V SPR-KKR manuálu:** Ano, jako `BSF`; jméno `BSFEK` je ASE2SPRKKR alias.
- **Parametry pokryté manuálem:** `stejné jako BSF E-k: TASK.NK, KPATH, NKDIR, KA*, KE*`
- **Parametry nepokryté manuálem:** —

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Smysluplné nativní workflow v uvedeném rozsahu.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_bsf_audit.py`, `src/ase2sprkkr/input_parameters/test/test_bsf_file_manifests.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** `BSF` deklarovaný taskem (samotné parsování není fyzikální ověření).
- **Detail rozhraní:** Samostatná registrovaná identita tasku; sdílí schéma BSF a režim E–k volí přes `ENERGY.NE`.
- **Podrobný audit zdrojů/funkčnosti:** [BSF](BSF-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### BSFKK

- **Co dělá:** ASE2SPRKKR alias pro `BSF` v k-k režimu.
- **Stav:** Funguje stejně jako `BSF`; alias pouze předvolí očekávaný single-energy k-k režim.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — Alias `BSF` (k–k); zdrojová větev `BLOCHSF` není doložená pro `EMBEDDED-CLUSTER` (`gen.f:1499–1506`).
- **Executable (běžný / embedding):** `kkrgen` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** Stejný vstup a potenciál jako schéma BSF; volitelná znovupoužitelná TAU cache.
- **Výstupní soubory:** `{DATASET_ADSI}.bsf`, volitelně `{DATASET_ADSI}_spol.bsf` (režim k–k).
- **V SPR-KKR manuálu:** Ano, jako `BSF`; jméno `BSFKK` je ASE2SPRKKR alias.
- **Parametry pokryté manuálem:** `stejné jako BSF k-k: TASK.KA, K1, K2, NK1, NK2`
- **Parametry nepokryté manuálem:** —

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Smysluplné nativní workflow v uvedeném rozsahu.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_bsf_audit.py`, `src/ase2sprkkr/input_parameters/test/test_bsf_file_manifests.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** `BSF` deklarovaný taskem (samotné parsování není fyzikální ověření).
- **Detail rozhraní:** Samostatná registrovaná identita tasku; sdílí schéma BSF a režim k–k volí přes `ENERGY.NE`.
- **Podrobný audit zdrojů/funkčnosti:** [BSF](BSF-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### CHI

- **Co dělá:** Počítá statickou magnetickou susceptibilitu v sériovém ASA/FREL workflow.
- **Stav:** Funguje v auditovaném statickém workflow; nativní `NCHIITER` je před smyčkou natvrdo resetováno na 1, takže iterativní magnetizační část ve skutečnosti neběží.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — `CLUDRIVE` volá `CLU_CHI_DRIVE`, ale ten pro `KKRMODE=EMBEDDED-CLUSTER` končí chybou `KKRMODE incorrect` (`cludrive.f:656–663`, `clu_chi_drive.f:39–49`).
- **Executable (běžný / embedding):** `kkrchi` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** nativní CHI/susceptibility output; bez explicitního output manifestu v definici.
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** `TASK.CHIPRINT`, `TASK.NOSYMCHI`
- **Parametry nepokryté manuálem:** `MODE.PRINT`, `MODE.NCHIITER`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Smysluplné nativní workflow v uvedeném rozsahu.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_chi.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [CHI](CHI-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### CHILANDAU

- **Co dělá:** Počítá Landauovu orbitální magnetickou susceptibilitu.
- **Stav:** Funguje sériově s omezeními; MPI a reuse `RDTAUFIL` jsou rozbité a nativní finish logic vypisuje mezisoučty.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — Lineární odezva používá `KKRCHI/RSP`, jehož inicializace odmítá `EMBEDDED-CLUSTER`; `CLU_CHI_DRIVE` pro CHILANDAU nemá výpočet (`kkrmain.f:1425–1433`, `rsp.f:103–114`).
- **Executable (běžný / embedding):** `kkrchi` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** nativní Landau susceptibility output; poslední serial block je relevantní výsledek.
- **V SPR-KKR manuálu:** Ne.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** `TASK.CLURAD`, `TASK.NSHLCLU`, `TASK.SELECT`, `TASK.EXCLUDE`, `TASK.QREPONLY`, `TASK.PRINT`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_chilandau.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [CHILANDAU](CHILANDAU-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### CHIXAS

- **Co dělá:** Počítá field-induced dichroismus v XAS z uživatelem dodaných susceptibilit a DOS na EF.
- **Stav:** Funguje jen při fyzikálně konzistentních externích `NEF/CHISPN/CHIVV`; univerzální bezpečné defaulty neexistují.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — Pro CHIXAS není v `CLU_CHI_DRIVE` výpočet; běžná odezva `RSP` odmítá embedded `KKRMODE` (`clu_chi_drive.f:77–96`, `rsp.f:103–114`).
- **Executable (běžný / embedding):** `kkrchi` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** standardní vstup + `POTFIL`; externě dodané hodnoty `NEF*`, `CHISPN`, `CHIVV`.
- **Výstupní soubory:** nativní CHIXAS/XAS response output.
- **V SPR-KKR manuálu:** Ne.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** `TASK.ITXRAY`, `TASK.CL`, `TASK.NEF*`, `TASK.CHISPN`, `TASK.CHIVV`, `TASK.ABSUNIT`, `TASK.PRINT`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_chixas.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [CHIXAS](CHIXAS-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### CLPES

- **Co dělá:** Počítá core-level photoemission/photoelectron diffraction v KKRSPEC.
- **Stav:** Funguje v auditovaném sériovém fixed-geometry workflow; CPA a některé polarizační režimy nativně stopnou, při `NE=1` je nutné hlídat core-state index.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — Implementaci fotoemise obsluhuje `KKRSPEC`; embeddingový `CLUDRIVE` nevolá `SPECMAIN` (`kkrmain.f:1392–1447`).
- **Executable (běžný / embedding):** `kkrspec` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** Generovaný KKRSPEC vstup, `POTFIL`; podle přepínačů případně `input.inp` a `struc.inp`.
- **Výstupní soubory:** `{DATASET_ADSI}_SPEC.out` a polarizačně závislé soubory `{DATASET_ADSI}_data*.spc`.
- **V SPR-KKR manuálu:** Ne.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem: `TASK.IQ_AT_SURF`, `TASK.MILLER_HKL`, `TASK.STRVER`, `TASK.INPVER`, `ENERGY.EWORK_EV`, `ENERGY.IMV_INI_EV`, `ENERGY.IMV_FIN_EV`, `SPEC_PH.THETA`, `SPEC_PH.PHI`, `SPEC_PH.POL_P`, `SPEC_PH.EPHOT`, `SPEC_EL.THETA`, `SPEC_EL.PHI`, `SPEC_EL.NT`, `SPEC_EL.NP`, `SPEC_EL.KA`, `SPEC_EL.K1`, `SPEC_EL.NK1`, `SPEC_EL.K2`, `SPEC_EL.NK2`, `SPEC_EL.POL_E`, `SPEC_EL.SPOL`, `SPEC_STR.N_LAYDBL`, `SPEC_STR.NLAT_G_VEC`, `SPEC_STR.N_LAYER`, `SPEC_STR.SURF_BAR`, `SPEC_STR.TRANSP_BAR`, `TASK.CRYS_VECS`, `SPEC_PH.ALQ`, `SPEC_PH.DELQ`, `SPEC_PH.NPOL`, `SPEC_PH.ICIRC`, `SPEC_PH.IDREH`, `SPEC_PH.IFSP`, `SPEC_PH.THETA_FIX`, `SPEC_EL.K3`, `SPEC_EL.NK3`, `SPEC_EL.K4`, `SPEC_EL.NK4`, `SPEC_EL.PSPIN`, `SPEC_EL.BETA1`, `SPEC_EL.BETA2`, `SPEC_EL.ROTAXIS`, `SPEC_EL.TYP`, `SPEC_EL.ISTR`, `SPEC_EL.POL0`, `SPEC_EL.POL0L`, `SPEC_EL.Q1`, `SPEC_EL.Q2`, `SPEC_EL.Q3`, `SPEC_EL.Q4`, `SPEC_CL.IT`, `SPEC_CL.CL`, `SPEC_CL.IE_ICST`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_clpes.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** `ARPES` deklarovaný taskem (samotné parsování není fyzikální ověření).
- **Podrobný audit zdrojů/funkčnosti:** [CLPES](CLPES-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### CLXPS

- **Co dělá:** Počítá plně relativistickou core-level XPS v KKRGEN.
- **Stav:** Funguje v auditovaném ASA/FREL workflow; advanced angular geometries a numerický output nebyly samostatně certifikovány.
- **Embedding (nativní SPR-KKR 9.7):** **Podmíněně** — `CLUGEN → GEN → CLXPS` je dosažitelná větev (`cludrive.f:631–658`, `gen.f:1434–1450`), omezená na ASA; správné clusterové maticové elementy ani výsledky zde nebyly numericky ověřeny.
- **Executable (běžný / embedding):** `kkrgen` → `embgen?` (pouze hypotetická cesta, ve větvi `cluster` nepovoleno)
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** nativní CL-XPS spektrum (`DATASET.pes`; PLOT/postprocessing může vytvářet `.xps`/xmgrace soubory).
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** `TASK.IT`, `TASK.CL`, `TASK.EPHOT`, `TASK.ImE`, `TASK.USETSS`
- **Parametry nepokryté manuálem:** `TASK.ME`, `TASK.MECHECK`, `TASK.LINPOL`, `TASK.ANGRES`, `TASK.ANGQ0..10`, `TASK.ANGE0..10`, `TASK.ANGK0..10`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Smysluplné nativní workflow v uvedeném rozsahu.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_clxps.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [CLXPS](CLXPS-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### COMPTON

- **Co dělá:** Počítá elektronovou Comptonovu momentum density a profily.
- **Stav:** Funguje s omezeními; `EMD2D` má při nenulovém `PNMAX` dvojí škálování a posunuté sudé mřížky ztrácejí krajní body.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — `COMPTON` v `compton.f:917–935` pro momentum-resolved část volá `STRSET` a `SETKKR`, následně invertuje periodickou k-prostorovou matici místo použití embeddingového scattering-path operátoru z `CLUDRIVE`; `compton.f:633–654` navíc čte externí TAU cache. Kód neobsahuje ekvivalentní clusterovou formulaci.
- **Executable (běžný / embedding):** `kkrgen` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** Generovaný vstup + `POTFIL` + povinná kompatibilní elektronová TAU directory (`TAUDIR`/`TAUPATH`).
- **Výstupní soubory:** `{DATASET_ADSI}_BND.mcp`, `{DATASET_ADSI}_COR.mcp`, `{DATASET_ADSI}_TOT.mcp`.
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** `TASK.MD2D`, `TASK.SHFT`, `TASK.PNVEC`, `TASK.NPN`, `TASK.NPP`, `TASK.PNMAX`, `TASK.PPMAX`
- **Parametry nepokryté manuálem:** `TASK.SPLITSS`, `TASK.MEMODE`, `TASK.NPMAX`, `TASK.USE_PEQUIV`, `TASK.PPVEC1`, `TASK.PPVEC2`, `TASK.EMD2D`, `TASK.SHFT_PP`, `TASK.SHFT_PN`, `TASK.INSIDE_SPHERE`, `TASK.SQQ_RESOLVED`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_compton.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [COMPTON](COMPTON-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### DMI

- **Co dělá:** ASE2SPRKKR alias sdílené JXC/DMI definice pro výpočet Dzyaloshinskii-Moriyova vektoru.
- **Stav:** Funguje s významnými ALE: jednotlivé DMI komponenty nejsou v nativním zdroji ekvivalentní (zejména Dz) a je nutné ověřovat frame/multiplicity/radius.
- **Embedding (nativní SPR-KKR 9.7):** **Ano** — FREL embedding používá `CLUXCPLTENSOR`, který vytváří jak J, tak D soubory (`*_JJij.dat`, `*_Dij_[xyz].dat`), i když se požaduje jen DMI (`gen.f:967–1000`; `cluxcpltensor.f:278,432–440`). Není to standardní periodický `DMIVECTOR`.
- **Executable (běžný / embedding):** `kkrgen` → `embgen`
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** `{DATASET_ADSI}_DMIVEC_Dij.dat`, volitelně grafy `{DATASET_ADSI}_DMIVEC_Dij_[xyz]_*.agr`. **Embedding FREL:** `CLUXCPLTENSOR` zapisuje `{DATASET_ADSI}_JJij.dat` a `{DATASET_ADSI}_Dij_[xyz].dat` i při samostatném DMI, včetně odpovídajících AGR (`cluxcpltensor.f`).
- **V SPR-KKR manuálu:** Ne jako samostatný task v dostupném manuálu; JXC je popsán.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** `TASK.DMI`, `TASK.JXC`, `TASK.CLURAD`, `TASK.NSHLCLU`, `TASK.SELECT`, `TASK.EXCLUDE`, `TASK.QREPONLY`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_dmi.py`, `src/ase2sprkkr/input_parameters/test/test_exchange_file_manifests.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** `JXC` deklarovaný taskem (samotné parsování není fyzikální ověření).
- **Detail rozhraní:** Sdílí schéma JXC, ale má vlastní registrovanou identitu tasku; čisté DMI a současné JXC+DMI nejsou ekvivalentní.
- **Podrobný audit zdrojů/funkčnosti:** [DMI](DMI-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### DOS

- **Co dělá:** Počítá hustotu stavů.
- **Stav:** Funguje.
- **Embedding (nativní SPR-KKR 9.7):** **Ano** — `CLUGEN` po přípravě host `TAUIJ` otevře `.dos`, nastaví mapu clusterových atomů a zavolá `GEN` (`cludrive.f:631–658`); DOS se integruje v `gen.f:868–913`.
- **Executable (běžný / embedding):** `kkrgen` → `embgen`
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** `{DATASET_ADSI}.dos`.
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** —

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Smysluplné nativní workflow v uvedeném rozsahu.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_dos.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** `DOS` deklarovaný taskem (samotné parsování není fyzikální ověření).
- **Podrobný audit zdrojů/funkčnosti:** [DOS](DOS-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### EKREL

- **Co dělá:** Počítá E(k) disperzi nebo Fermi-surface search pro uspořádané systémy.
- **Stav:** Funguje; CPA je nativně odmítnuto, pro disorder se má použít BSF.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — `GEN` obsahuje `EKREL` (`gen.f:1494–1497`), ale jeho k-resolved eigenenergetický výstup nemá doloženou interpretaci na `EMBEDDED-CLUSTER`; nezaměňovat s dosažitelností `GEN`.
- **Executable (běžný / embedding):** `kkrgen` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** nativní EKREL band/Fermi-surface data.
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** `TASK.NK`, `TASK.EMIN`, `TASK.EMAX`, `TASK.NE`, `TASK.F-SURF`, `TASK.ESLICE`, `TASK.KPATH`, `TASK.NKDIR`, `TASK.KA1..9`, `TASK.KE1..9`
- **Parametry nepokryté manuálem:** —

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Smysluplné nativní workflow v uvedeném rozsahu.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_ekrel.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [EKREL](EKREL-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### FMAG

- **Co dělá:** Počítá spinovou/orbitální hustotu a magnetický form factor.
- **Stav:** Funguje s omezením: nulový magnetický moment může při normalizaci vést k dělení nulou.
- **Embedding (nativní SPR-KKR 9.7):** **Podmíněně** — `initvar.f:534–550` aktivuje `CALC_FMAG`, a `CLUGEN` volá obecný energetický loop `GEN`; není prokázána samostatně validovaná distribuce výstupu na host/cluster (`gen.f`, `cludrive.f:631–658`).
- **Executable (běžný / embedding):** `kkrgen` → `embgen?` (pouze hypotetická cesta, ve větvi `cluster` nepovoleno)
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** `{DATASET_ADSI}_fmag_AT.agr` a `{DATASET_ADSI}_dens_AT.agr` pro jednotlivé typy.
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** —

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_fmag.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [FMAG](FMAG-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### FSCAT

- **Co dělá:** Počítá scattering-amplitude matice pro tři pevné backscattering geometrie.
- **Stav:** Rozbité: chybný spherical-harmonic index `+L` může přetéct a fáze se počítá před ošetřením nulové amplitudy.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — `GEN` má volání `FSCAT` (`gen.f:1413–1420`), ale původní task audit uvádí nativní selhání/omezení; funkční embeddingový režim nelze potvrdit.
- **Executable (běžný / embedding):** `kkrgen` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** nativní scattering-amplitude diagnostické soubory.
- **V SPR-KKR manuálu:** Ne.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** —

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Rozbité nativní workflow.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_fscat.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [FSCAT](FSCAT-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### GILBERT

- **Co dělá:** Počítá Gilbertův damping, případně včetně teplotních spinových/lattice fluktuací.
- **Stav:** Funguje pro vhodné magnetické systémy; nulové nebo kompenzované momenty mohou způsobit dělení nulou.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — `CLU_CHI_DRIVE` odmítá embeddingový `KKRMODE` a volání `GIL` je zakomentované; `GEN` pro GILBERT nenabízí náhradní výpočet (`clu_chi_drive.f:39–49,79–95`).
- **Executable (běžný / embedding):** `kkrchi` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** standardní vstup + `POTFIL`; při `SETFLUCT=M_T` povinný `TASK.FLUCTFIL`.
- **Výstupní soubory:** `{DATASET_ADSI}alfa_x.dat`, `{DATASET_ADSI}alfa_T.dat`, `{DATASET_ADSI}_alfa_T.agr` (respektují nativní způsob pojmenování).
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** `TASK.SETFLUCT`, `TASK.TMPMIN`, `TASK.TMPMAX`, `TASK.NTMP`, `TASK.NVIBRA`
- **Parametry nepokryté manuálem:** `TASK.FLUCTFIL`, `TASK.NFTET`, `TASK.NFPHI`, `TASK.DMAGDT`, `TASK.TDEBYE`, `TASK.INITSVEC`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_gilbert.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Detail rozhraní:** Současná definice tasku povoluje MPI; to *není* důkaz ověřené numerické shody mezi sériovým a MPI během.
- **Podrobný audit zdrojů/funkčnosti:** [GILBERT](GILBERT-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### JALF

- **Co dělá:** Počítá valenční proudovou hustotu z Diracových matic α a související magnetický moment.
- **Stav:** Funguje s omezeními; helper diagnostiky mají chyby a malý `NL` je rizikový, což definice hlídá.
- **Embedding (nativní SPR-KKR 9.7):** **Podmíněně** — `CALC_JALF` vede i při volání z `CLUGEN` do `CURDNS` (`initvar.f:534–550`, `gen.f:1569–1579`), ale platnost clusterové proudové hustoty nebyla ověřena.
- **Executable (běžný / embedding):** `kkrgen` → `embgen?` (pouze hypotetická cesta, ve větvi `cluster` nepovoleno)
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** `{DATASET}_JALF.dat` a diagnostika s pevnými názvy `curdns1`, `curdns2`, `curdns3`.
- **V SPR-KKR manuálu:** Ne.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** —

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_jalf.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [JALF](JALF-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### JORB

- **Co dělá:** Počítá relativistickou paramagnetickou orbitální proudovou hustotu.
- **Stav:** Funguje sériově; nativní MPI workflow nedojde k finálnímu JORB výstupu a neredukuje akumulátory.
- **Embedding (nativní SPR-KKR 9.7):** **Podmíněně** — `CALC_JORB` vede v `GEN` do `ORBCURDNS` (`initvar.f:534–550`, `gen.f:1579–1588`); bez důkazu správného clusterového výstupu jen podmíněná podpora.
- **Executable (běžný / embedding):** `kkrgen` → `embgen?` (pouze hypotetická cesta, ve větvi `cluster` nepovoleno)
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** `{DATASET}_JORB.dat`, `{DATASET}_JORB_UP.dat`, `{DATASET}_JORB_DN.dat`, případně `fort.<unit>`.
- **V SPR-KKR manuálu:** Ne.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** —

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Smysluplné nativní workflow v uvedeném rozsahu.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_jorb.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [JORB](JORB-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### JXC

- **Co dělá:** Počítá Heisenbergovy exchange couplings, ve FREL i tensorové komponenty.
- **Stav:** Funguje pro jednoduchý periodický kolineární workflow; obecný FREL tensor se symetricky ekvivalentními sousedy má nativní rotační bug a cluster SP-SREL scratch flow je nespolehlivý.
- **Embedding (nativní SPR-KKR 9.7):** **Ano** — Doloženou embeddingovou větví je FREL `CLUXCPLTENSOR` (`gen.f:967–1000`, `cluxcpltensor.f:63–69`), zapisující `*_JJij.dat` a `*_Dij_[xyz].dat`. SP-SREL `CLUXCPLJIJ` závisí na `IBZINT` a není zde považováno za ověřený embeddingový workflow.
- **Executable (běžný / embedding):** `kkrgen` → `embgen`
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** Periodicky: `{DATASET_ADSI}_J_ij.dat` / `{DATASET_ADSI}_Jij_*.agr` (SP-SREL), nebo `{DATASET_ADSI}_XCPLTEN_Jij.dat` a `XCPLTEN_*` ploty (FREL); periodické DMI má `DMIVEC_*`. **Embedding FREL:** `{DATASET_ADSI}_JJij.dat`, `{DATASET_ADSI}_Jij_*.agr`, `{DATASET_ADSI}_Dij_x.dat`, `_Dij_y.dat`, `_Dij_z.dat` a `Dij_[xyz]_*.agr` (`cluxcpltensor.f:278,432–440`).
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** `TASK.CLURAD`
- **Parametry nepokryté manuálem:** `TASK.JXC`, `TASK.DMI`, `TASK.NSHLCLU`, `TASK.SELECT`, `TASK.EXCLUDE`, `TASK.QREPONLY`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_jxc.py`, `src/ase2sprkkr/input_parameters/test/test_jxc_definition.py`, `src/ase2sprkkr/input_parameters/test/test_exchange_file_manifests.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** `JXC` deklarovaný taskem (samotné parsování není fyzikální ověření).
- **Podrobný audit zdrojů/funkčnosti:** [JXC](JXC-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### MECHECK

- **Co dělá:** Diagnosticky porovnává NAB/ADA maticové elementy.
- **Stav:** Funguje jen částečně: `NMEFORM` je nativně vždy přepsáno na 2, další consistency test je compile-time vypnutý a summary loop má chybu.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — `INITVAR` explicitně vyžaduje `PROGNAME=KKRGEN` pro `MECHECK` (test 11), tedy odmítne `CLUGEN` (`initvar.f:160–167`).
- **Executable (běžný / embedding):** `kkrgen` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** nativní matrix-element diagnostika; bez explicitního output manifestu.
- **V SPR-KKR manuálu:** Ne.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** `TASK.ITSEL`, `TASK.PRINT`, `TASK.INITIALSTATE`, `TASK.CL`, `TASK.ERYDI`, `TASK.ERYDF`, `TASK.NONMAG`, `TASK.NMEFORM`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_mecheck.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [MECHECK](MECHECK-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### MEPLOT

- **Co dělá:** Počítá/plotuje dipólové maticové elementy pro BAND nebo CORE počáteční stav.
- **Stav:** Funguje, až na nativní přepisování křivek při současném photon+electron scanu; definice tuto kombinaci odmítá. Význam `SEFIN` potřebuje fyzikální ověření.
- **Embedding (nativní SPR-KKR 9.7):** **Podmíněně** — `CLUGEN` předává task do `GEN`, který volá `ME_PLOT` (`gen.f:1481–1485`); není doloženo, že jsou potřebné clusterové maticové elementy v embeddingu kompatibilní.
- **Executable (běžný / embedding):** `kkrgen` → `embgen?` (pouze hypotetická cesta, ve větvi `cluster` nepovoleno)
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** nativní `.agr`/matrix-element plot data; bez explicitního output manifestu.
- **V SPR-KKR manuálu:** Ne.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** `TASK.ITSEL`, `TASK.IPRINT`, `TASK.INITIALSTATE`, `TASK.L`, `TASK.CL`, `TASK.NONMAG`, `TASK.SEFIN`, `TASK.EPHOT`, `TASK.NEPHOT`, `TASK.EPHOTMIN`, `TASK.EPHOTMAX`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_meplot.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [MEPLOT](MEPLOT-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### NRAES

- **Co dělá:** Počítá nerelativistické Augerovo spektrum postprocessingem existujícího DOS.
- **Stav:** Rozbité: `IAES` je jen částečně nulováno a convolution akumuluje do neinicializovaných oblastí.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — Nativní `NRAES` pracuje s částečně neinicializovaným polem `IAES`; tato chyba brání věrohodnému výsledku i v obecném `GEN`, tedy i při volání z `CLUGEN`.
- **Executable (běžný / embedding):** `kkrgen` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** Generovaný vstup + `POTFIL` + existující `{DATASET_ADSI}.dos` (starý spinově rozlišený formát).
- **Výstupní soubory:** `{DATASET_ADSI}.<type>.<core-shell>.aes`; případné doplňky CALCLS/OUTGNU (nativní numerická rutina obsahuje chybu).
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** `TASK.IT`, `TASK.CL`, `TASK.OUTGNU`, `TASK.CALCLS`
- **Parametry nepokryté manuálem:** —

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Rozbité nativní workflow.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_nraes.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [NRAES](NRAES-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### NUCSSCPL

- **Co dělá:** Má počítat tensor jaderné spin-spin coupling.
- **Stav:** Rozbité: nativní kód deallocuje pole před první alokací, vynuluje matrix elements pro každý type a nutí mesh na 1.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — Původní task audit uvádí nativní chyby v `NUCSSCPL`; `GEN` neobsahuje pro cluster opravenou specializovanou variantu.
- **Executable (běžný / embedding):** `kkrgen` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** `{DATASET_ADSI}_A_ij.dat`, `RK.dat`, `tab_aij.dat`.
- **V SPR-KKR manuálu:** Ne.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** `TASK.CLURAD`, `TASK.NSHLCLU`, `TASK.SELECT`, `TASK.EXCLUDE`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Rozbité nativní workflow.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_scattering_file_manifests.py`, `src/ase2sprkkr/input_parameters/test/test_dataset_filenames.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [NUCSSCPL](NUCSSCPL-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### OPTABS

- **Co dělá:** Počítá absorpční část optické vodivosti.
- **Stav:** Funguje s velkými ALE: `TEMPEL<=1` nechává `KBT` neinicializované a `NOMEGA<NEB` může přepsat alokaci; definice tyto stavy zakazuje.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — `INITVAR` vynucuje pro OPTABS/OPTICS `KKRCHI` (`initvar.f:342–345`); lineární odezva `RSP` odmítá embedded `KKRMODE` (`rsp.f:103–114`).
- **Executable (běžný / embedding):** `kkrchi` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** nativní OPTICS-ABS conductivity output.
- **V SPR-KKR manuálu:** Ne.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** `TASK.NEB`, `TASK.NOMEGA`, `TASK.OMEGA1`, `TASK.OMEGA2`, `TASK.TEMPEL`, `TASK.NKT`, `TASK.FRAMETET`, `TASK.FRAMEPHI`, `TASK.FRAMEGAM`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_optabs.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [OPTABS](OPTABS-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### PHAGEN

- **Co dělá:** Exportuje sférický potenciál/charge/Coulomb data pro PHAGEN/MsSpec.
- **Stav:** Funguje jako omezený ASA/VWN converter; NT=1, bez disorder, ne jako obecný phase-shift solver.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — Při `TASK.PHAGEN` je přímo vyžadován `KKRSCF` (`initvar.f:179–184`); `CLUSCF` by skončil před výpočtem.
- **Executable (běžný / embedding):** `kkrscf` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** Generovaný vstup + konvergovaný nemagnetický ASA/VWN potenciál `POTFIL` (NT=1, neprovádí vlastní řešení fázových posuvů).
- **Výstupní soubory:** `<POTFIL>_ex_sprkkr` a `<POTFIL>.only_coulomb_new` (názvy vycházejí z `POTFIL`, nikoli `DATASET`).
- **V SPR-KKR manuálu:** Ne.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** —

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_phagen.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Nativní integrační test exportu PHAGEN prošel pro konvergované Au a ověřil oba výstupní soubory; bez srovnání s MsSpec či fyzikálním benchmarkem.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [PHAGEN](PHAGEN-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### PLOT2DPOT

- **Co dělá:** Má kreslit 2D řez potenciálem v uživatelem zadané rovině.
- **Stav:** Funguje v ASA pro potential; FULLPOT větev jde do experimentálního PLY writeru a skončí před běžným plane-grid outputem.
- **Embedding (nativní SPR-KKR 9.7):** **Podmíněně** — Selektor nastavuje `PLOT2DPR`; `GEN` může zavolat `PLOT2DPR_ASA` či `FPPLOT2DPR` (`initvar.f:199–205`, `gen.f:1457–1476`), ale nebyla ověřena projekce hostitelských a clusterových radiálních dat.
- **Executable (běžný / embedding):** `kkrgen` → `embgen?` (pouze hypotetická cesta, ve větvi `cluster` nepovoleno)
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** Pevná PLY jména jako PLOT2DPR; ASA zapisuje potenciál, FULLPOT používá nedokončenou experimentální větev.
- **V SPR-KKR manuálu:** Ne.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** `TASK.AVEC`, `TASK.BVEC`, `TASK.DVEC`, `TASK.NA`, `TASK.NB`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Deklarovaný účel nebo hlavní větev není spolehlivě dosažena.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_plot2dpr.py`, `src/ase2sprkkr/input_parameters/test/test_plot_selector_file_manifests.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Detail rozhraní:** `TASK.AVEC/BVEC/DVEC` používají kartézské **jednotky ALAT**.
- **Podrobný audit zdrojů/funkčnosti:** [PLOT2D-selectors](PLOT2D-selectors-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### PLOT2DPR

- **Co dělá:** Má kreslit obecný 2D řez stored radial/potential daty.
- **Stav:** Částečně funguje: v ASA ve skutečnosti zapisuje jen potential; FULLPOT větev skončí v experimentálním PLY writeru.
- **Embedding (nativní SPR-KKR 9.7):** **Podmíněně** — `CLUGEN → GEN` dosáhne `PLOT2DPR_ASA`/`FPPLOT2DPR` (`gen.f:1457–1476`). Funkčnost pro skutečná embedded data a FULLPOT zůstává podmíněná.
- **Executable (běžný / embedding):** `kkrgen` → `embgen?` (pouze hypotetická cesta, ve větvi `cluster` nepovoleno)
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** Pevná jména PLY: `PLY-V-ASA-*.ply`, `POT-ASA-LINES.ply` (ASA), nebo `PLY-V-FP-*.ply`, `PLY-V-MT-*.ply`, `PLY-FP-*.ply` a související soubory (experimentální FULLPOT větev).
- **V SPR-KKR manuálu:** Ne.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** `TASK.AVEC`, `TASK.BVEC`, `TASK.DVEC`, `TASK.NA`, `TASK.NB`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Deklarovaný účel nebo hlavní větev není spolehlivě dosažena.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_plot2dpr.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Detail rozhraní:** `TASK.AVEC/BVEC/DVEC` jsou kartézské vektory v **jednotkách ALAT**, nikoli přímo v Bohrech nebo frakčních souřadnicích.
- **Podrobný audit zdrojů/funkčnosti:** [PLOT2DPR](PLOT2DPR-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### PLOT2DRHO

- **Co dělá:** Má kreslit 2D řez elektronovou hustotou.
- **Stav:** Prakticky neúplné: ASA density neumí; FULLPOT skončí v testovacím PLY writeru před běžným plane-grid writerem.
- **Embedding (nativní SPR-KKR 9.7):** **Podmíněně** — Stejná cesta jako `PLOT2DPR` (`initvar.f:199–205`, `gen.f:1457–1476`); jde o vykreslení existujících radiálních dat, nikoli důkaz embeddingového solveru.
- **Executable (běžný / embedding):** `kkrgen` → `embgen?` (pouze hypotetická cesta, ve větvi `cluster` nepovoleno)
- **Vstupní soubory:** standardní vstup + FULLPOT `POTFIL`.
- **Výstupní soubory:** Pevná PLY jména jako PLOT2DPR; ASA nedodává hustotu a FULLPOT nezajišťuje výběr pouze hustoty.
- **V SPR-KKR manuálu:** Ne.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** `TASK.AVEC`, `TASK.BVEC`, `TASK.DVEC`, `TASK.NA`, `TASK.NB`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Deklarovaný účel nebo hlavní větev není spolehlivě dosažena.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_plot2dpr.py`, `src/ase2sprkkr/input_parameters/test/test_plot_selector_file_manifests.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Detail rozhraní:** `TASK.AVEC/BVEC/DVEC` používají kartézské **jednotky ALAT**.
- **Podrobný audit zdrojů/funkčnosti:** [PLOT2D-selectors](PLOT2D-selectors-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### PLOTPOT

- **Co dělá:** Zapisuje stored radial potential pro plotování bez nového SCF/DOS výpočtu.
- **Stav:** Funguje pro ASA i FULLPOT stored potential data.
- **Embedding (nativní SPR-KKR 9.7):** **Podmíněně** — Selektor pod `PLOTPRS` předává `GEN` požadavek na vykreslení uloženého potenciálu (`initvar.f:189–196`, `gen.f:1453–1461`); host/cluster výběr nebyl ověřen.
- **Executable (běžný / embedding):** `kkrgen` → `embgen?` (pouze hypotetická cesta, ve větvi `cluster` nepovoleno)
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** nativní radial potential plot files.
- **V SPR-KKR manuálu:** Ne.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** —

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Smysluplné nativní workflow v uvedeném rozsahu.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_plotprs.py`, `src/ase2sprkkr/input_parameters/test/test_plot_selector_file_manifests.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [PLOT-selectors](PLOT-selectors-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### PLOTPRS

- **Co dělá:** Zapisuje stored potential, charge/spin density a FULLPOT shape-function grafy.
- **Stav:** Funguje s omezeními: magnetic spin/orb curves mohou kolidovat ve stejném datasetu a panel limits ignorují orbital contribution.
- **Embedding (nativní SPR-KKR 9.7):** **Podmíněně** — `CLUGEN → GEN → FPPLOTPRS` je dosažitelný pro zvolený selektor (`gen.f:1452–1463`); neověřená práce s daty hostitele a clusteru.
- **Executable (běžný / embedding):** `kkrgen` → `embgen?` (pouze hypotetická cesta, ve větvi `cluster` nepovoleno)
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** nativní radial potential/density/shape-function plot files.
- **V SPR-KKR manuálu:** Ne.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** —

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_plotprs.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [PLOTPRS](PLOTPRS-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### PLOTRHO

- **Co dělá:** Má samostatně zapisovat stored radial density.
- **Stav:** Rozbité: `RHORSQ` se alokuje jen v potential branch, ale standalone rho writer ho potřebuje.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — Samostatný nativní `PLOTRHO` obsahuje chybu: `RHORSQ` není pro tuto větev alokováno, takže `CLUGEN` ji nečiní funkční. Viz individuální audit PLOTPRS.
- **Executable (běžný / embedding):** `kkrgen` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** nativní density plot output, ale běžný standalone běh je nespolehlivý.
- **V SPR-KKR manuálu:** Ne.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** —

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Rozbité nativní workflow.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_plotprs.py`, `src/ase2sprkkr/input_parameters/test/test_plot_selector_file_manifests.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [PLOT-selectors](PLOT-selectors-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### PLOTSFN

- **Co dělá:** Zapisuje stored radial FULLPOT shape-function coefficients.
- **Stav:** Funguje pro skutečný FULLPOT potential se shape-function daty.
- **Embedding (nativní SPR-KKR 9.7):** **Podmíněně** — Selektor `PLOTPRS` lze předat do `GEN`; k vykreslení jsou nutné uložené FULLPOT shape functions a jejich interpretace pro cluster nebyla ověřena (`initvar.f:189–196`, `gen.f:1452–1463`).
- **Executable (běžný / embedding):** `kkrgen` → `embgen?` (pouze hypotetická cesta, ve větvi `cluster` nepovoleno)
- **Vstupní soubory:** standardní vstup + FULLPOT `POTFIL`.
- **Výstupní soubory:** nativní shape-function plot files.
- **V SPR-KKR manuálu:** Ne.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** —

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Smysluplné nativní workflow v uvedeném rozsahu.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_plotprs.py`, `src/ase2sprkkr/input_parameters/test/test_plot_selector_file_manifests.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [PLOT-selectors](PLOT-selectors-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### POSANI

- **Co dělá:** Počítá pozitronovou annihilation momentum density a profily.
- **Stav:** Funguje jen s kompatibilním `POSANIPREP`; nativní data path předpokládá `NQ=NT` a obecný `NQ!=NT` workflow je rozbitý.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — `POSANI` používá totožný periodický k-prostorový `COMPTON` driver (`gen.f:1572–1577`, `compton.f:917–935`) a závisí na periodických pozitronových párových datech `{DATSET}_pos_dat.pan` (`compton.f:424–426`). Prosté spuštění pod `CLUGEN` nenahradí periodickou finální integraci embeddingovým TAU.
- **Executable (běžný / embedding):** `kkrgen` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** Generovaný vstup + elektronový `POTFIL` + povinné elektronové TAU + `{DATASET_ADSI}_pos_dat.pan` z POSANIPREP.
- **Výstupní soubory:** `{DATASET_ADSI}_BND.mpa`, `_COR.mpa`, `_TOT.mpa`; podle voleb také `..._pan2d.dat` nebo `..._data.mpa`.
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** `TASK.MD2D`, `TASK.SHFT`, `TASK.PNVEC`, `TASK.NPN`, `TASK.NPP`, `TASK.PNMAX`, `TASK.PPMAX`
- **Parametry nepokryté manuálem:** `TASK.SPLITSS`, `TASK.MEMODE`, `TASK.NPMAX`, `TASK.USE_PEQUIV`, `TASK.PPVEC1`, `TASK.PPVEC2`, `TASK.EMD2D`, `TASK.SHFT_PP`, `TASK.SHFT_PN`, `TASK.INSIDE_SPHERE`, `TASK.SQQ_RESOLVED`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_posani.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [POSANI](POSANI-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### POSANIPREP

- **Co dělá:** Připraví s-like pozitronový stav, wavefunctions a scattering data pro `POSANI`.
- **Stav:** Funguje pro jednoduchý 3D ASA/FREL `NQ=NT`; export TAUQQ je pro obecné `NQ!=NT` rozbitý. Energii dna pozitronového pásu si uživatel musí zvolit.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — `POSANIPREPARE` explicitně vytváří periodickou k-mřížku a volá `KLOOPSIJ` pro `TAUQQ` (`posaniprep.f:116–145`), místo clusterových `TAUIJ`. Export `TAUQQ` navíc iteruje přes typy `NT`, nikoli místa `NQ` (`posaniprep.f:268–272`). Nemá správnou embedded-cluster implementaci.
- **Executable (běžný / embedding):** `kkrgen` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** standardní vstup + elektronový `POTFIL`.
- **Výstupní soubory:** `{DATASET_ADSI}_pos_dat.pan`.
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** `TASK.POSVC`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_posaniprep.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [POSANIPREP](POSANIPREP-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### POSANITAU

- **Co dělá:** Integruje pozitronovou hustotu a počítá annihilation rate/lifetime.
- **Stav:** Funguje v auditovaném 3D ASA/FREL workflow; integrační energy bounds je třeba fyzikálně ověřit a enhancement factor je nativně fixní.
- **Embedding (nativní SPR-KKR 9.7):** **Podmíněně** — Na rozdíl od `POSANI` má skutečnou cluster-aware cestu: `POSANIPOT` zpracovává `NTCLU` a posouvá potenciál clusterových atomů (`posanipot.f:95–110,267–275`), `GEN` využívá `TAU_DRIVE` a následně `CHRDNS`/`POSANILAMBDA` (`gen.f:783–807,882–897,1509–1564`). `POSANILAMBDA` integruje vybrané typy `ITBOT:ITTOP` (`posanilambda.f:59–105`); normalizace v konečném clusteru a příspěvek hostitele nejsou fyzikálně ověřeny. Pouze podmíněná nativní cesta, v `ase2sprkkr` nepovolena.
- **Executable (běžný / embedding):** `kkrgen` → `embgen?` (pouze hypotetická cesta, ve větvi `cluster` nepovoleno)
- **Vstupní soubory:** Generovaný vstup + elektronový `POTFIL`; volitelná cache `{DATASET_ADSI}_positron.rho`.
- **Výstupní soubory:** Nativní lifetime/rate ve výstupu programu; volitelná cache `{DATASET_ADSI}_positron.rho`.
- **V SPR-KKR manuálu:** Ne.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** `TASK.POSVC`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Smysluplné nativní workflow v uvedeném rozsahu.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_posanitau.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [POSANITAU](POSANITAU-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### PSHIFT

- **Co dělá:** Počítá/plotuje scattering phase shifts.
- **Stav:** Funguje, ale resonance splitting je v některých případech nespolehlivý.
- **Embedding (nativní SPR-KKR 9.7):** **Podmíněně** — `GEN` má `PSHIFT` větev (`gen.f:1427–1435`), ovšem pouze pro ASA; validita clusterového phase-shift výstupu je neověřená.
- **Executable (běžný / embedding):** `kkrgen` → `embgen?` (pouze hypotetická cesta, ve větvi `cluster` nepovoleno)
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** nativní phase-shift plot/data files.
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** —

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_pshift.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [PSHIFT](PSHIFT-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### SCF

- **Co dělá:** Provádí self-consistent výpočet a vytváří nový potenciál.
- **Stav:** Funguje pro `BROYDEN2`, `ANDERSON` a `TCHEBY` v povoleném scope; `PULAY` není validní a `TCHEBY` není pro FULLPOT.
- **Embedding (nativní SPR-KKR 9.7):** **Ano** — `CLUSCF` připraví host/cluster data a přímo zavolá `SCF`; vyžaduje host Madelung potential a náboje (`cludrive.f:619–629`).
- **Executable (běžný / embedding):** `kkrscf` → `embscf`
- **Vstupní soubory:** standardní vstup + startovní `POTFIL`; FULLPOT může reuse `CONTROL.SFNFIL`.
- **Výstupní soubory:** `<POTFIL>_new` a běžný SCF log; při inicializaci může vzniknout i startovací DOS.
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** `SCF.VXC`, `SCF.ALG`, `SCF.NITER`, `SCF.MIX`, `SCF.TOL`, `SCF.ISTBRY`, `SCF.ITDEPT`, `SCF.QION`, `SCF.MSPIN`, `SCF.USEVMATT`
- **Parametry nepokryté manuálem:** `SCF.MIXOP`, `SCF.EFGUESS`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Smysluplné nativní workflow v uvedeném rozsahu.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_scf_audit.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** `SCF` deklarovaný taskem (samotné parsování není fyzikální ověření).
- **Podrobný audit zdrojů/funkčnosti:** [SCF](SCF-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### SIGMA

- **Co dělá:** Počítá DC conductivity/Kubo-Středa response, surface-only nebo sea+surface.
- **Stav:** Funguje s omezeními; Fermi-sea není zahrnuta automaticky a režimy používají fixní file conventions/ImE.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — Pro `SIGMA` `INITVAR` explicitně vyžaduje `KKRCHI` (`initvar.f:350–354`), avšak `RSP` odmítá `EMBEDDED-CLUSTER` (`rsp.f:103–114`).
- **Executable (běžný / embedding):** `kkrchi` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** Pevná nativní jména `sigma?????.dat` (bez izolace pomocí `DATASET`).
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** `TASK.SIGSPROJ`, `TASK.SP_NO_ALPHA`, `TASK.SP_NO_NABLA`, `TASK.NOSYMSIG`, `TASK.FRAMETET`, `TASK.FRAMEPHI`, `TASK.FRAMEGAM`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_sigma.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [SIGMA](SIGMA-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### SOCPAR

- **Co dělá:** Počítá spin-orbit coupling parameters.
- **Stav:** Funguje pro hlavní SOC výstup; exchange-related output má chybný label jednotek a jeden sign-flip test je nativně nedosažitelný.
- **Embedding (nativní SPR-KKR 9.7):** **Podmíněně** — `GEN` dovoluje `SOCPAR` (`gen.f:1421–1430`) v ASA; kompatibilita spin-orbit výstupu s clusterovým mapováním nebyla ověřena.
- **Executable (běžný / embedding):** `kkrgen` → `embgen?` (pouze hypotetická cesta, ve větvi `cluster` nepovoleno)
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** nativní SOCPAR tables/plot data.
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** —

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_socpar.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [SOCPAR](SOCPAR-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### SPLEED

- **Co dělá:** Počítá spin-polarized low-energy electron diffraction v KKRSPEC.
- **Stav:** Funguje v konzervativním fixed-angle workflow; momentum-scan a alternativní polarizační větve nejsou auditovány.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — SPLEED je výpočet `KKRSPEC` a výpočetní tok `CLU*` k `SPECMAIN` nevede (`kkrmain.f:1431–1447`).
- **Executable (běžný / embedding):** `kkrspec` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** Generovaný KKRSPEC vstup, `POTFIL`; podle přepínačů případně `input.inp` a `struc.inp`.
- **Výstupní soubory:** `{DATASET_ADSI}_SPEC.out`, `{DATASET_ADSI}_data.spc` (SPC může obsahovat jen hlavičku).
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** `TASK.IQ_AT_SURF`, `TASK.MILLER_HKL`, `TASK.STRVER`, `TASK.INPVER`, `ENERGY.EWORK_EV`, `ENERGY.IMV_INI_EV`, `ENERGY.IMV_FIN_EV`, `SPEC_PH.THETA`, `SPEC_PH.PHI`, `SPEC_PH.POL_P`, `SPEC_PH.EPHOT`, `SPEC_EL.THETA`, `SPEC_EL.PHI`, `SPEC_EL.NT`, `SPEC_EL.NP`, `SPEC_EL.KA`, `SPEC_EL.K1`, `SPEC_EL.NK1`, `SPEC_EL.K2`, `SPEC_EL.NK2`, `SPEC_EL.POL_E`, `SPEC_EL.SPOL`, `SPEC_STR.N_LAYDBL`, `SPEC_STR.NLAT_G_VEC`, `SPEC_STR.N_LAYER`, `SPEC_STR.SURF_BAR`, `SPEC_STR.TRANSP_BAR`
- **Parametry nepokryté manuálem: `TASK.CRYS_VECS`, `SPEC_PH.ALQ`, `SPEC_PH.DELQ`, `SPEC_PH.NPOL`, `SPEC_PH.ICIRC`, `SPEC_PH.IDREH`, `SPEC_PH.IFSP`, `SPEC_PH.THETA_FIX`, `SPEC_EL.K3`, `SPEC_EL.NK3`, `SPEC_EL.K4`, `SPEC_EL.NK4`, `SPEC_EL.PSPIN`, `SPEC_EL.BETA1`, `SPEC_EL.BETA2`, `SPEC_EL.ROTAXIS`, `SPEC_EL.TYP`, `SPEC_EL.ISTR`, `SPEC_EL.POL0`, `SPEC_EL.POL0L`, `SPEC_EL.Q1`, `SPEC_EL.Q2`, `SPEC_EL.Q3`, `SPEC_EL.Q4`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_spleed.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** `SPEC` deklarovaný taskem (samotné parsování není fyzikální ověření).
- **Podrobný audit zdrojů/funkčnosti:** [SPLEED](SPLEED-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### STONER

- **Co dělá:** Počítá Stonerův parametr z DOS a radiálních wavefunctions na EF.
- **Stav:** Funguje s omezeními; používá natvrdo VWN susceptibility kernel, malé DOS hodnoty nativně nahrazuje jednotkovým příspěvkem a nulové normalizace nejsou vždy ošetřeny.
- **Embedding (nativní SPR-KKR 9.7):** **Podmíněně** — `INITVAR` pro STONER sice nastaví `RDTAUDIR=.TRUE.` (`initvar.f:384–390`), ale při neexistující cache jej `KKRMAIN` přepne na `.FALSE.` (`kkrmain.f:932–959`); potom `CLUGEN → GEN` může spočítat skutečné embedded TAU (`gen.f:788–807`) a vytvořit lokální DOS/vlnové funkce. `CALCDOS` však plní `DOBS_LTX` jen pro `ITBOT:ITTOP` (`calcdos.f:246–253`), které v `CLUDRIVE` odpovídají clusterovým typům (`cludrive.f:414–425`), zatímco STONER skládá `NEFTL` pro `IT=1..NT` (`gen.f:1592–1610`). Není doloženo, zda `SUSC_GAMMA` ignoruje nulové položky hostitele nebo je započte do výsledku. Původ ani výběr TAU zde nejsou kritériem podpory embeddingu. **Výpočetní cesta existuje, numerická správnost potvrzena není; v `ase2sprkkr` zůstává embedding zablokován.**
- **Executable (běžný / embedding):** `kkrgen` → `embgen?` (pouze hypotetická cesta, ve větvi `cluster` nepovoleno)
- **Vstupní soubory:** standardní vstup + `POTFIL`; může reuse TAU cache.
- **Výstupní soubory:** nativní STONER output v standardním výstupu.
- **V SPR-KKR manuálu:** Ne.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** —

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_stoner.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [STONER](STONER-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### T1

- **Co dělá:** Má počítat nuclear spin-lattice relaxation na EF.
- **Stav:** Rozbité pro numerický účel: nativní `RELT1` používá signed κ jako kladné počty mj kanálů, takže záporné kanály mizí a basis indexing je chybný.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — Nativní `RELT1` chybně používá podepsané κ jako počty kanálů mj; takto poškozený výpočet není podporou embeddingu.
- **Executable (běžný / embedding):** `kkrgen` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** nativní T1/RELT1 output, ale numerické rates nejsou důvěryhodné.
- **V SPR-KKR manuálu:** Ne.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** —

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Rozbité nativní workflow.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_t1.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [T1](T1-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### TDOBS

- **Co dělá:** Má představovat time-dependent observable task, ale v 9.7 jde prakticky o obecnou DOS/observable energy loop s pokusem reuse TAU.
- **Stav:** Deklarovaný účel není prokázaný: nenašel se samostatný time-dependent observable implementation; jako generic DOS path může běžet sériově.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — Není nalezena samostatná implementace `TDOBS` v `CLU_CHI_DRIVE` ani explicitní `GEN` task branch; obecný průchod `GEN` není výpočet TDOBS.
- **Executable (běžný / embedding):** `kkrgen` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** standardní vstup + `POTFIL`; může reuse `TAUDIR`/`TAUPATH` cache.
- **Výstupní soubory:** `{DATASET_ADSI}.dos`.
- **V SPR-KKR manuálu:** Ne.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** `TAU.TAUDIR`, `TAU.TAUPATH`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Deklarovaný účel nebo hlavní větev není spolehlivě dosažena.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_tdobs.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [TDOBS](TDOBS-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### TORQUE

- **Co dělá:** Počítá magnetický torque pro předem rotované magnetic moments.
- **Stav:** Funguje, ale vyžaduje rotated magnetic potential/geometry; běžný nerotovaný magnetic potential může nativně STOPnout a mixed occupancy používá první type.
- **Embedding (nativní SPR-KKR 9.7):** **Podmíněně** — `GEN` volá `TORQUE` při `CALC_TORQUE` (`gen.f:1004–1009`). `initvar.f:542–551` však zakazuje kombinaci JXC a torque; clusterová numerika nebyla ověřena.
- **Executable (běžný / embedding):** `kkrgen` → `embgen?` (pouze hypotetická cesta, ve větvi `cluster` nepovoleno)
- **Vstupní soubory:** standardní vstup + rotovaný magnetic `POTFIL`.
- **Výstupní soubory:** nativní TORQUE output.
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** `TASK.THETAQ`, `TASK.PHIQ`
- **Parametry nepokryté manuálem:** —

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_torque.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [TORQUE](TORQUE-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### TUTORIAL

- **Co dělá:** Spouští nativní vzdělávací/diagnostické větve YLM, JLX, radial-wavefunction a DOS demonstrace.
- **Stav:** Smíšený stav: YLM/JLX/JLX-ASYMPT a DOS-FEG/DOS-SS/DOS-CLU/DOS-BULK/GEN jsou použitelné; CHR-POT, POT-DECOMP/POT-XC, RWF-L/RWF-E a CORE-ALL mají specifické nativní chyby/omezení.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — `TUTORIAL` se speciálně volá jen ve větvi `KKRGEN`, kdežto `CLU*` jde přes `CLUDRIVE → GEN` (`kkrmain.f:1411–1433`).
- **Executable (běžný / embedding):** `kkrgen` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** standardní vstup + `POTFIL` (kromě čistě matematických demonstrací, kde jej nativní startup přesto očekává).
- **Výstupní soubory:** různé nativní `.agr`/diagnostické/DOS výstupy podle `TUT-TASK`.
- **V SPR-KKR manuálu:** Ne.
- **Parametry pokryté manuálem:** —
- **Parametry nepokryté manuálem:** `TASK.TUT-TASK`, `TASK.L`, `TASK.M`, `TASK.LMAX`, `TASK.NX`, `TASK.XMIN`, `TASK.XMAX`, `TASK.X-RANGE`, `TASK.ITSEL`, `TASK.NORMALISATION`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Smíšený stav jednotlivých větví; viz TUTORIAL audit.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_tutorial.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [TUTORIAL](TUTORIAL-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### VBPES

- **Co dělá:** Počítá úhlově integrovanou valenční photoemission v KKRGEN, odděleně od KKRSPEC ARPES.
- **Stav:** Funguje s omezeními; VBUPS si nativně automaticky nevolí UV photon energy a výpočet závisí na kompatibilním TAU/final-state setupu.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — Periodické fotoemisní workflow používá existující elektronickou TAU directory a vlastní k-prostorovou přípravu finálních stavů (`initvar.f:221–255`, `gen.f:1445–1450`, viz individuální audit `vbpes.f:282–291`). Nebyla nalezena embeddingová konstrukce finálních stavů ani clusterové mapování fotoemisních maticových elementů. Nativní cesta `CLUGEN` tak není podporou tohoto tasku.
- **Executable (běžný / embedding):** `kkrgen` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** Generovaný vstup + `POTFIL` + povinné TAU počátečního stavu; volitelná cache finálního stavu `{DATASET_ADSI}.tau.dir.F_<photon>`.
- **Výstupní soubory:** Pevná jména `arpes.dat`, `arpes_spol.dat`, `spin.matrix.xps`; volitelný finální `{DATASET_ADSI}.dos.F_<photon>`.
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** `TASK.EPHOT`, `TASK.USETAUNN`
- **Parametry nepokryté manuálem:** `TASK.DEFEP0`, `TASK.ME`, `TASK.MODE`, `TASK.EWORK`, `TASK.GEOMETRY`, `TASK.QVEC`, `TASK.QVECPH`, `TASK.TETPH`, `TASK.PHIPH`, `TASK.NSPIN`, `TASK.ImEFIN`, `TASK.USETFIN`, `ENERGY.ERANGE`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_vbpes.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [VBPES](VBPES-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### WFPLOT

- **Co dělá:** Plotuje radiální BAND nebo CORE wavefunctions.
- **Stav:** Funguje v ASA/FREL serial workflow; BAND používá jen první energy point, FULLPOT není podporován.
- **Embedding (nativní SPR-KKR 9.7):** **Podmíněně** — `GEN` může vyvolat `WFPLOT` (`gen.f:1456–1460`), pouze při přítomnosti odpovídajících řešení; jejich mapování na host/cluster není ověřené.
- **Executable (běžný / embedding):** `kkrgen` → `embgen?` (pouze hypotetická cesta, ve větvi `cluster` nepovoleno)
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** nativní xmgrace/radial-wavefunction plot files.
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** `TASK.ITSEL`, `TASK.STATE`, `TASK.PRINT`, `TASK.L`, `TASK.CL`, `TASK.MJ`
- **Parametry nepokryté manuálem:** —

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Smysluplné nativní workflow v uvedeném rozsahu.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_wfplot.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [WFPLOT](WFPLOT-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### XAS

- **Co dělá:** Počítá X-ray absorption spectra přes XRAYSPEC.
- **Stav:** Funguje se známými ALE; static branch je použitelný, ale TAU compatibility a efektivní začátek energy scanu je nutné kontrolovat. Native MPI končí před XRAYSPEC.
- **Embedding (nativní SPR-KKR 9.7):** **Podmíněně** — `GEN → XRAYSPEC` je dosažitelné pro XAS (`gen.f:1393–1402`), ale jde o obecnou implementaci: není doložená kvalita specificky clusterového spektra ani plná podpora v GUI.
- **Executable (běžný / embedding):** `kkrgen` → `embgen?` (pouze hypotetická cesta, ve větvi `cluster` nepovoleno)
- **Vstupní soubory:** standardní vstup + `POTFIL`; může reuse TAU cache.
- **Výstupní soubory:** `{DATASET_ADSI}_*_<core-shell>.rat` (nativní soubor závisí na typu a slupce) a další XRAYSPEC diagnostika.
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** `TASK.IT`, `TASK.CL`, `TASK.OUTPUT`
- **Parametry nepokryté manuálem:** `TASK.ME`, `TASK.POLOBS`, `TASK.DIPOL`, `TASK.GFCONVRH`, `TASK.WRREGIRR`, `TASK.NONDIPE2`, `TASK.NONDIPM1`, `TASK.FRAMETET`, `TASK.FRAMEPHI`, `TASK.FRAMEGAM`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Použitelná omezená větev s významnými výhradami.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_xas.py`, `src/ase2sprkkr/input_parameters/test/test_xas_definition.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [XAS](XAS-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### XES

- **Co dělá:** Počítá X-ray emission přes společný XRAYSPEC reader.
- **Stav:** Ne zcela ověřené: RXES skončí ve sdílené absorption-rate větvi a output naming/interpretace je sdílená s XAS; fyzikální emission význam nebyl potvrzen.
- **Embedding (nativní SPR-KKR 9.7):** **Podmíněně** — `GEN → XRAYSPEC` je dosažitelné i pro XES (`gen.f:1393–1402`), ale detaily přiřazení core stavů embedded uzlům nejsou ověřené.
- **Executable (běžný / embedding):** `kkrgen` → `embgen?` (pouze hypotetická cesta, ve větvi `cluster` nepovoleno)
- **Vstupní soubory:** standardní vstup + `POTFIL`; kompatibilní occupied-state TAU/cache.
- **Výstupní soubory:** `{DATASET_ADSI}_*_<core-shell>.rat`; nativní cesta sdílí absorpční rutinu, význam emisního spektra není ověřen.
- **V SPR-KKR manuálu:** Ano.
- **Parametry pokryté manuálem:** `TASK.IT`, `TASK.CL`
- **Parametry nepokryté manuálem:** `TASK.ME`, `TASK.OUTPUT`, `TASK.POLOBS`, `TASK.DIPOL`, `TASK.TAUCORE`, `TASK.GFCONVRH`, `TASK.WRREGIRR`, `TASK.NONDIPE2`, `TASK.NONDIPM1`, `TASK.FRAMETET`, `TASK.FRAMEPHI`, `TASK.FRAMEGAM`

- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Deklarovaný účel nebo hlavní větev není spolehlivě dosažena.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_xes.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [XES](XES-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

### XRS

- **Co dělá:** Počítá X-ray resonant scattering přes tří-integrační XRAYSPEC workflow.
- **Stav:** Rozbité pro spolehlivý numerický výsledek: nativní `SIG(NEMAX+50,...)` je nulováno jen do `NEMAX`, ale později se sčítá přes celý rozsah.
- **Embedding (nativní SPR-KKR 9.7):** **Ne** — `GEN` umí volat `XRAYSPEC`, avšak původní audit odhalil zásadní numerickou chybu v `XRS` (`SIG` buffer); nelze označit za funkčně podporované v embeddingu.
- **Executable (běžný / embedding):** `kkrgen` → žádný podporovaný embeddingový executable
- **Vstupní soubory:** standardní vstup + `POTFIL`; kompatibilní TAU/cache.
- **Výstupní soubory:** `{DATASET_ADSI}_*_<core-shell>.rat`; obsah tenzoru ovlivňuje neinicializovaná paměť.
- **V SPR-KKR manuálu:** Ano (v manuálu je XRS alespoň jako podporovaný spectroscopy task; detailní 9.7 interface je širší).
- **Parametry pokryté manuálem:** `TASK.IT`, `TASK.CL`
- **Parametry nepokryté manuálem:** `TASK.ME`, `TASK.NE3`, `TASK.EMAX3`, `TASK.TSELECT`, `TASK.POLOBS`, `TASK.DIPOL`, `TASK.TAUCORE`, `TASK.GFCONVRH`, `TASK.WRREGIRR`, `TASK.NONDIPE2`, `TASK.NONDIPM1`, `TASK.FRAMETET`, `TASK.FRAMEPHI`, `TASK.FRAMEGAM`

## Native SPR-KKR tasks intentionally not exposed as normal ASE2SPRKKR definitions

These are not counted among the 56 user-facing names above, but they are relevant to a task audit because their absence is intentional rather than an omission:

- **BIS** — native parser recognizes the task, but the VBPES routine contains a guard that inevitably stops for `BIS`; no ASE2SPRKKR definition should pretend that inverse photoemission works.
- **SIGMA-XAS** — source branch exists, but the standardní vstup parser cannot set the internal `TASK=SIGMA-XAS`; no normal definition is exposed.
- **PLOTJALF / PLOTJORB** — selectors exist around plotting/current code, but they are not coherent standalone calculation tasks; FULLPOT exits in the PLY writer and ASA does not produce the intended current plot.
- **FORCETEN, XMO, OPTICS, MAGNET, CHIDYN, PHONONS, PUMP, OPM, SIGMA-DNS, NEGF** — disabled/unavailable public-source branches (typically explicit stop or unfinished routine), therefore intentionally not presented as supported ASE2SPRKKR tasks.

## Main sources used

1. Current ASE2SPRKKR `tasks` branch definitions under `src/ase2sprkkr/input_parameters/definitions/`.
2. Consolidated October 2026 source audit `SPRKKR-ASE2SPRKKR-common-audit.md` and the individual `*-audit.md` task reports.
3. Public SPR-KKR User's Guide repository entry (`SPRKKR 8.6 Manual`, submitted 2022-02-08) and searchable public User's Guide text for explicit task/parameter coverage.
4. Public SPR-KKR 9.7 source package `PUB9.7_260420_18h13` used by the individual audits.

## Caveats worth keeping with this document

- Manual coverage is deliberately **not inferred from source code**. A parameter can be valid in SPR-KKR 9.7 and still be classified as manual-uncovered.
- The public guide is older than the audited 9.7 source; source-only parameters are therefore expected, especially in response, positron and plotting workflows.
- A task being registered in ASE2SPRKKR means that an input definition exists; it does not override a proven native SPR-KKR bug. Broken tasks are retained here because the goal is an audit, not a list of recommended calculations.
- `BSFEK`, `BSFKK` and `DMI` are public ASE2SPRKKR aliases/shares, not independent native implementation modules. `TUTORIAL` is one ASE2SPRKKR task with 14 native `TUT-TASK` submodes.
- **Stav rozhraní:** Registrovaná identita tasku; závěr zdrojového auditu: Rozbité nativní workflow.
- **Testy ASE2SPRKKR:** `src/ase2sprkkr/input_parameters/test/test_xrs.py`. Jde o **testy popsané v auditu z 9. 11. 2026**, nikoli nové spuštění nad aktuálním HEAD.
- **Nativní/numerické ověření:** Referenční audit neprokazuje nezávislý numerický benchmark tohoto tasku. Python testy ani existence parseru nepotvrzují fyzikální správnost.
- **Strukturovaný reader výsledků:** Samostatný reader zde není doložen; log ani nativní soubory nejsou automaticky validovanými strukturovanými výsledky.
- **Podrobný audit zdrojů/funkčnosti:** [XRS](XRS-audit.md); [souhrnný zdrojový audit](../SPRKKR-ASE2SPRKKR-common-audit.md).

