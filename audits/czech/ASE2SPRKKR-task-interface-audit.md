# ASE2SPRKKR — audit rozhraní tasků, parametrů a dokumentace

**Aktualizace:** 13. listopadu 2026  
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

## Index

| Task | Status (short) | In manual |
|---|---|---|
| [`AES`](#aes) | Funguje jen v omezeném auditovaném workflow; NQ=1, bez f-shellů a s riziky v cache/TAU kapacitě | Ano |
| [`AIPES`](#aipes) | Funguje v auditovaném fixed-geometry workflow; není to numerická integrace ARPES přes úhly | Ano |
| [`APS`](#aps) | Funguje s kompatibilním starým DOS; pozitivní `GAMMA` broadening má nativní problematickou větev | Ano |
| [`ARPES`](#arpes) | Funguje, ale současná definice má správnou podporu nativního, presence-sensitive `CRYS_VECS`; nízkoúrovňové/expert větve … | Ano |
| [`BAND`](#band) | Funguje v auditovaném workflow; selhání eigen-solveru může nativně pokračovat až k `LOG(0)` | Ano |
| [`BONDING`](#bonding) | Rozbité: běžný `TAU_DRIVE` připraví `KTAUIJ=0`, ale `CLUBONDING` čte scratch jednotky vyžadující `KTAUIJ>=2` | Ne |
| [`BSDOSIJ`](#bsdosij) | Rozbité: nativní `J_ITJT` je zapisováno energetickými indexy do pair dimenzí a později čteno v jiném pořadí | Ne |
| [`BSF`](#bsf) | Pouze schéma; nativní BSF funguje; E-k default `EMAX=-1 Ry` je podezřelý convenience default a nemusí znamenat EF | Ano |
| [`BSFEK`](#bsfek) | Funguje stejně jako `BSF`; alias pouze předvolí očekávaný E-k režim přes `ENERGY.NE` | Ano, jako `BSF`; jméno `BSFEK` je ASE2SPRKKR alias |
| [`BSFKK`](#bsfkk) | Funguje stejně jako `BSF`; alias pouze předvolí očekávaný single-energy k-k režim | Ano, jako `BSF`; jméno `BSFKK` je ASE2SPRKKR alias |
| [`CHI`](#chi) | Funguje v auditovaném statickém workflow; nativní `NCHIITER` je před smyčkou natvrdo resetováno na 1, takže iterativn… | Ano |
| [`CHILANDAU`](#chilandau) | Funguje sériově s omezeními; MPI a reuse `RDTAUFIL` jsou rozbité a nativní finish logic vypisuje mezisoučty | Ne |
| [`CHIXAS`](#chixas) | Funguje jen při fyzikálně konzistentních externích `NEF/CHISPN/CHIVV`; univerzální bezpečné defaulty neexistují | Ne |
| [`CLPES`](#clpes) | Funguje v auditovaném sériovém fixed-geometry workflow; CPA a některé polarizační režimy nativně stopnou, při `NE=1` … | Ne |
| [`CLXPS`](#clxps) | Funguje v auditovaném ASA/FREL workflow; advanced angular geometries a numerický output nebyly samostatně certifikovány | Ano |
| [`COMPTON`](#compton) | Funguje s omezeními; `EMD2D` má při nenulovém `PNMAX` dvojí škálování a posunuté sudé mřížky ztrácejí krajní body | Ano |
| [`DMI`](#dmi) | Funguje s významnými ALE: jednotlivé DMI komponenty nejsou v nativním zdroji ekvivalentní (zejména Dz) a je nutné ově… | Ne jako samostatný task v dostupném manuálu; JXC je popsán |
| [`DOS`](#dos) | Funguje | Ano |
| [`EKREL`](#ekrel) | Funguje; CPA je nativně odmítnuto, pro disorder se má použít BSF | Ano |
| [`FMAG`](#fmag) | Funguje s omezením: nulový magnetický moment může při normalizaci vést k dělení nulou | Ano |
| [`FSCAT`](#fscat) | Rozbité: chybný spherical-harmonic index `+L` může přetéct a fáze se počítá před ošetřením nulové amplitudy | Ne |
| [`GILBERT`](#gilbert) | Funguje pro vhodné magnetické systémy; nulové nebo kompenzované momenty mohou způsobit dělení nulou | Ano |
| [`JALF`](#jalf) | Funguje s omezeními; helper diagnostiky mají chyby a malý `NL` je rizikový, což definice hlídá | Ne |
| [`JORB`](#jorb) | Funguje sériově; nativní MPI workflow nedojde k finálnímu JORB výstupu a neredukuje akumulátory | Ne |
| [`JXC`](#jxc) | Funguje pro jednoduchý periodický kolineární workflow; obecný FREL tensor se symetricky ekvivalentními sousedy má nat… | Ano |
| [`MECHECK`](#mecheck) | Funguje jen částečně: `NMEFORM` je nativně vždy přepsáno na 2, další consistency test je compile-time vypnutý a summa… | Ne |
| [`MEPLOT`](#meplot) | Funguje, až na nativní přepisování křivek při současném photon+electron scanu; definice tuto kombinaci odmítá | Ne |
| [`NRAES`](#nraes) | Rozbité: `IAES` je jen částečně nulováno a convolution akumuluje do neinicializovaných oblastí | Ano |
| [`NUCSSCPL`](#nucsscpl) | Rozbité: nativní kód deallocuje pole před první alokací, vynuluje matrix elements pro každý type a nutí mesh na 1 | Ne |
| [`OPTABS`](#optabs) | Funguje s velkými ALE: `TEMPEL<=1` nechává `KBT` neinicializované a `NOMEGA<NEB` může přepsat alokaci; definice tyto … | Ne |
| [`PHAGEN`](#phagen) | Funguje jako omezený ASA/VWN converter; NT=1, bez disorder, ne jako obecný phase-shift solver | Ne |
| [`PLOT2DPOT`](#plot2dpot) | Funguje v ASA pro potential; FULLPOT větev jde do experimentálního PLY writeru a skončí před běžným plane-grid outputem | Ne |
| [`PLOT2DPR`](#plot2dpr) | Částečně funguje: v ASA ve skutečnosti zapisuje jen potential; FULLPOT větev skončí v experimentálním PLY writeru | Ne |
| [`PLOT2DRHO`](#plot2drho) | Prakticky neúplné: ASA density neumí; FULLPOT skončí v testovacím PLY writeru před běžným plane-grid writerem | Ne |
| [`PLOTPOT`](#plotpot) | Funguje pro ASA i FULLPOT stored potential data | Ne |
| [`PLOTPRS`](#plotprs) | Funguje s omezeními: magnetic spin/orb curves mohou kolidovat ve stejném datasetu a panel limits ignorují orbital con… | Ne |
| [`PLOTRHO`](#plotrho) | Rozbité: `RHORSQ` se alokuje jen v potential branch, ale standalone rho writer ho potřebuje | Ne |
| [`PLOTSFN`](#plotsfn) | Funguje pro skutečný FULLPOT potential se shape-function daty | Ne |
| [`POSANI`](#posani) | Funguje jen s kompatibilním `POSANIPREP`; nativní data path předpokládá `NQ=NT` a obecný `NQ!=NT` workflow je rozbitý | Ano |
| [`POSANIPREP`](#posaniprep) | Funguje pro jednoduchý 3D ASA/FREL `NQ=NT`; export TAUQQ je pro obecné `NQ!=NT` rozbitý | Ano |
| [`POSANITAU`](#posanitau) | Funguje v auditovaném 3D ASA/FREL workflow; integrační energy bounds je třeba fyzikálně ověřit a enhancement factor j… | Ne |
| [`PSHIFT`](#pshift) | Funguje, ale resonance splitting je v některých případech nespolehlivý | Ano |
| [`SCF`](#scf) | Funguje pro `BROYDEN2`, `ANDERSON` a `TCHEBY` v povoleném scope; `PULAY` není validní a `TCHEBY` není pro FULLPOT | Ano |
| [`SIGMA`](#sigma) | Funguje s omezeními; Fermi-sea není zahrnuta automaticky a režimy používají fixní file conventions/ImE | Ano |
| [`SOCPAR`](#socpar) | Funguje pro hlavní SOC výstup; exchange-related output má chybný label jednotek a jeden sign-flip test je nativně ned… | Ano |
| [`SPLEED`](#spleed) | Funguje v konzervativním fixed-angle workflow; momentum-scan a alternativní polarizační větve nejsou auditovány | Ano |
| [`STONER`](#stoner) | Funguje s omezeními; používá natvrdo VWN susceptibility kernel, malé DOS hodnoty nativně nahrazuje jednotkovým příspě… | Ne |
| [`T1`](#t1) | Rozbité pro numerický účel: nativní `RELT1` používá signed κ jako kladné počty mj kanálů, takže záporné kanály mizí a… | Ne |
| [`TDOBS`](#tdobs) | Deklarovaný účel není prokázaný: nenašel se samostatný time-dependent observable implementation; jako generic DOS pat… | Ne |
| [`TORQUE`](#torque) | Funguje, ale vyžaduje rotated magnetic potential/geometry; běžný nerotovaný magnetic potential může nativně STOPnout … | Ano |
| [`TUTORIAL`](#tutorial) | Smíšený stav: YLM/JLX/JLX-ASYMPT a DOS-FEG/DOS-SS/DOS-CLU/DOS-BULK/GEN jsou použitelné; CHR-POT, POT-DECOMP/POT-XC, R… | Ne |
| [`VBPES`](#vbpes) | Funguje s omezeními; VBUPS si nativně automaticky nevolí UV photon energy a výpočet závisí na kompatibilním TAU/final… | Ano |
| [`WFPLOT`](#wfplot) | Funguje v ASA/FREL serial workflow; BAND používá jen první energy point, FULLPOT není podporován | Ano |
| [`XAS`](#xas) | Funguje se známými ALE; static branch je použitelný, ale TAU compatibility a efektivní začátek energy scanu je nutné … | Ano |
| [`XES`](#xes) | Ne zcela ověřené: RXES skončí ve sdílené absorption-rate větvi a output naming/interpretace je sdílená s XAS; fyzikál… | Ano |
| [`XRS`](#xrs) | Rozbité pro spolehlivý numerický výsledek: nativní `SIG(NEMAX+50, | Ano (v manuálu je XRS alespoň jako podporovaný spectroscopy task; detailní 9.7 interface je širší) |

## Tasks

### AES

- **Co dělá:** Počítá plně relativistické Augerovo elektronové spektrum z existujícího elektronového TAU a maticových elementů.
- **Stav:** Funguje jen v omezeném auditovaném workflow; NQ=1, bez f-shellů a s riziky v cache/TAU kapacitě. Numerický solver nebyl v auditu spuštěn.
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
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** `{DATASET_ADSI}_DMIVEC_Dij.dat`, volitelně grafy `{DATASET_ADSI}_DMIVEC_Dij_[xyz]_*.agr`.
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
- **Vstupní soubory:** standardní vstup + `POTFIL`.
- **Výstupní soubory:** Podle režimu `{DATASET_ADSI}_J_ij.dat` / `{DATASET_ADSI}_Jij_*.agr` nebo `{DATASET_ADSI}_XCPLTEN_Jij.dat` / `..._Jij_*.agr`; při aktivovaném DMI také DMI soubory.
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

