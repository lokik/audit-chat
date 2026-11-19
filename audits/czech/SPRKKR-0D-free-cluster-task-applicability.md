# Nativní podpora volného 0D clusteru — audit tasků SPR-KKR 9.7

**Datum:** 19. listopadu 2026. **Baseline:** `PUB9.7_260420_18h13` a Fortranové závěry v [auditu rozhraní tasků](ASE2SPRKKR-task-interface-audit.md), [souhrnném auditu](SPRKKR-ASE2SPRKKR-common-audit.md) a [auditech jednotlivých úloh](tasks/). Křížová kontrola s [2D auditem](SPRKKR-2D-task-applicability.md) a [uživatelskou příručkou SPR-KKR](https://www.scribd.com/document/873750459/kkr-manual) (§ 2.2.3). **Nejsou zde deklarovány nově spuštěné numerické testy.**

## Přesná definice „cluster“

Zde **cluster = izolovaný konečný systém bez hostitelského potenciálu**, nikoli malá část periodického krystalu ani impurita v hostiteli. Nativní reprezentace je **`TAU.MOL`**, tj. molekulární real-space KKR; pro přístup do real-space větve je relevantní **`TAU.BZINT=CLUSTER` / `IBZINT=0`**. Počet míst `NQ` odpovídá počtu atomových pozic molekuly; třetí periodická mříž nepopisuje fyzický host a její vektory slouží jako pomocná souřadná báze.

**Pozor:** samotné **`TAU.CLUSTER`** bez `MOL` znamená konečný výřez jinak periodického systému pro přiblížení centrálního místa; `TAU.MOL` počítá operátor pro všechna místa molekuly. Ani jeden přepínač neznamená **embedding**, který vyžaduje host potential a vlastní `embscf`/`embgen` flow. Pro neembedovaný 0D výpočet zůstává normální executable (`kkrscf`, `kkrgen` atd.), **nepřepínat na `emb*`**.

**Ano** = smysluplná nativní 0D větev (včetně čistě lokálních operací nad potenciálem), za konkrétních uvedených omezení. **Ne** = daný task **neinzerovat jako podporovaný pro obecný volný 0D cluster**: je periodický, obecně nativně nefunkční nebo chybí doložený celistvý molekulární vstup/výstup. „Ne“ v posledním případě **neprohlašuje matematickou nemožnost**. Rozhodnutí o dostupnosti v GUI má být konzervativní; 0D postprocessing není automaticky 0D fyzikální solver.

Celkem **56 položek: 19 Ano, 37 Ne**. `BSF` je jen sdílené schéma pro `BSFEK`/`BSFKK`. Podpora **0D není totéž co podpora embeddingu**.

## Všechny tasky

| Task | Volný 0D cluster | Důvod / omezení |
|---|---|---|
| `AES` | **Ne** | Vyžaduje NQ=1; nepokrývá obecný víceatomový volný cluster. |
| `AIPES` | **Ne** | KKRSPEC pracuje s krystalovou/fotoemisní geometrií, nikoli molekulárním TAU. |
| `APS` | **Ne** | Navazuje na starý formát DOS; úplná molekulární cesta není doložena. |
| `ARPES` | **Ne** | Úhlově rozlišená KKRSPEC geometrie a Blochovy stavy, ne izolovaná molekula. |
| `BAND` | **Ne** | Periodická disperze E(k) vyžaduje k-prostor. |
| `BONDING` | **Ne** | Zamýšlen pro molekulární cluster, ale TAU_DRIVE nepřipraví požadované scratch 18/19. |
| `BSDOSIJ` | **Ne** | Obecná nativní chyba indexování párů/energií; nelze garantovat výsledek. |
| `BSF` | **Ne** | Pouze sdílené schéma; spektrální funkce v k-prostoru není 0D task. |
| `BSFEK` | **Ne** | Blochova spektrální funkce E–k vyžaduje periodicitu. |
| `BSFKK` | **Ne** | Blochova spektrální funkce k–k vyžaduje periodicitu. |
| `CHI` | **Ne** | Samostatná KKRCHI větev; není doložena molekulární 0D susceptibilita. |
| `CHILANDAU` | **Ne** | SUSC_LANDAU volá TAU_STD přímo, nikoli TAU_REAL_SPACE. |
| `CHIXAS` | **Ne** | Externí NEF/CHISPN/CHIVV a KKRCHI; chybí doložený molekulární kontrakt. |
| `CLPES` | **Ne** | KKRSPEC s periodickou/fotoemisní geometrií. |
| `CLXPS` | **Ano** | Lokální core-matrix elementy v GEN; jen ASA/FREL a kompatibilní obsazené jádrové stavy. |
| `COMPTON` | **Ne** | Vytváří vlastní periodickou k-prostorovou matici STRSET/SETKKR. |
| `DMI` | **Ne** | Normální FREL JXC/DMI vyžaduje IBZINT=2/periodický TAUIJ; embedded alternativa není free cluster. |
| `DOS` | **Ano** | GEN přes molekulární real-space TAU; výsledek pro všechna molekulární místa. |
| `EKREL` | **Ne** | Vlastní k-závislé vlastní energie bez významu pro izolovaný cluster. |
| `FMAG` | **Ano** | Lokální hustoty/momenty z real-space TAU; ASA, nenulové normalizační momenty. |
| `FSCAT` | **Ne** | Obecná nativní chyba indexování sférických harmonik. |
| `GILBERT` | **Ne** | GIL vyžaduje IBZINT=2/6, nikoli molekulární real-space režim. |
| `JALF` | **Ano** | Integrace lokální relativistické proudové hustoty v GEN; pouze sériově a s dostatečným NL. |
| `JORB` | **Ano** | Lokální orbitální proudová hustota v GEN; pouze sériově, FREL. |
| `JXC` | **Ne** | Normální větev vyžaduje IBZINT=2 a periodický TAUIJ; CLUXCPLTENSOR patří embeddingu. |
| `MECHECK` | **Ano** | Lokální test maticových elementů; částečně nefunkční diagnostiky nepředstavují plnou certifikaci. |
| `MEPLOT` | **Ano** | Lokální atomové maticové elementy, bez nezbytné Blochovy geometrie; ne kombinovaný scan. |
| `NRAES` | **Ne** | Obecně vadná inicializace akumulátoru AES. |
| `NUCSSCPL` | **Ne** | Obecné chyby alokace a přepisu maticových elementů. |
| `OPTABS` | **Ne** | OPT_ABS má standardní TB-KKR response větev, nikoli molekulární 0D cestu. |
| `PHAGEN` | **Ano** | Převod již existujícího potenciálu, nikoli 0D řešič; jen konvergovaný ASA/VWN a NT=1. |
| `PLOT2DPOT` | **Ne** | ASA kreslení vytváří periodické kopie; není spolehlivá molekulární geometrie. |
| `PLOT2DPR` | **Ne** | ASA vytváří pouze potenciál a periodické kopie; FULLPOT experimentální. |
| `PLOT2DRHO` | **Ne** | Obecně nedokončený výstup hustoty. |
| `PLOTPOT` | **Ano** | Postprocessing radiálního potenciálu z POTFIL; nezávisí na k-periodicitě. |
| `PLOTPRS` | **Ano** | Lokální radiální data POTFIL; pozor na kolize magnetických křivek. |
| `PLOTRHO` | **Ne** | Obecná chyba nepřipravené RHORSQ. |
| `PLOTSFN` | **Ano** | Pouze graf existujících FULLPOT shape functions; nespočítá cluster. |
| `POSANI` | **Ne** | Potřebuje .pan připravený periodickým POSANIPREP workflow. |
| `POSANIPREP` | **Ne** | Povinné POSANIPOT a periodický k-mesh; ne molekulární real-space cesta. |
| `POSANITAU` | **Ne** | Povinné POSANIPOT a nativně 3D periodické předpoklady. |
| `PSHIFT` | **Ano** | Atomově lokální phase-shift z POTFIL; některé odhady rezonancí jsou omezené. |
| `SCF` | **Ano** | Molekulární TAU.MOL zpracovává všechna NQ místa bez host potenciálu; normální kkrscf. |
| `SIGMA` | **Ne** | Kubo/BZ conductivity ve STANDARD/TB-KKR, nikoli izolovaný konečný cluster. |
| `SOCPAR` | **Ano** | Lokální spin–orbit parametry z atomových radiálních dat; známá chyba jednotky u exchange části. |
| `SPLEED` | **Ne** | Difrakce na povrchu v KKRSPEC vyžaduje rozšířenou strukturu. |
| `STONER` | **Ano** | Běžná GEN/SUSC_GAMMA má v MOL všechny typy aktivní; neplatí host-scratch chyba embeddingu; FREL/serial. |
| `T1` | **Ne** | Obecná nativní chyba relativistických κ kanálů. |
| `TDOBS` | **Ne** | Chybí ověřitelná samostatná implementace time-dependent observable. |
| `TORQUE` | **Ano** | Lokální momenty/sites z GEN; jen s korektně rotovaným magnetickým potenciálem. |
| `TUTORIAL` | **Ano** | Pouze nezávislé matematické testy a molekulárně kompatibilní příklady; neznamená podporu všech TUT-TASK. |
| `VBPES` | **Ne** | Požaduje kompatibilní periodický initial/final-state TAU a fotoemisní geometrii. |
| `WFPLOT` | **Ano** | Lokální jádrové/pásové radiální funkce z potenciálu; ASA/FREL, serial, bez FULLPOT. |
| `XAS` | **Ano** | Lokální spektroskopie s real-space TAU; FREL, serial, zvolený core site, konzistentní energetická síť. |
| `XES` | **Ne** | Obecně nedoložená samostatná emisní větev RXES; sdílí absorpční výstup. |
| `XRS` | **Ne** | Obecná nativní chyba inicializace pole SIG. |

## Zdrojové uzly a hranice závěru

1. **MOL oproti CLUSTER:** manuál SPR-KKR § 2.2.3 říká, že `MOL` modeluje všechna `NQ` místa izolovaného systému, zatímco `CLUSTER` má správné centrální `τqq` a zbytek pouze jako aproximaci bulku. [Detail BONDING](tasks/BONDING-audit.md) zachycuje `init_mod_kmesh.f:717` (real space), `tau_real_space.f` (molekulární rozsah míst) a jeho defektní `KTAUIJ` scratch kontrakt.
2. **Výběr programu:** [embedding audit](ASE2SPRKKR-task-interface-audit.md) `kkrmain.f:300–307,1392–1447`, `cludrive.f:398–413,619–664` — `CLUSCF`/`CLUGEN` jsou jiná host-dependent cesta, nejsou synonymem `TAU.MOL`.
3. **Periodické blokátory:** `JXC/DMI` vyžadují normálně `IBZINT=2` (`gen.f:355–393`); `GILBERT` `IBZINT=2/6` (`gil.f:89–93`); `CHILANDAU` volá `TAU_STD` (`susc_landau.f:221–226`); `COMPTON` vytváří periodickou k-matici (`compton.f:917–935`). Viz [jednotlivé audity](tasks/).
4. **Lokální výpočty:** `GEN` provádí `DENSITY` (`FMAG`), `CURDNS` (`JALF`), `ORBCURDNS` (`JORB`), `SUSC_GAMMA` (`STONER`) a `XRAYSPEC` (`XAS`). Jejich fyzikální interpretace a popsané chyby zůstávají v jednotlivých auditech. Pro `STONER` se specifická host-scratch chyba **embeddingu** nesmí přenášet na běžný `MOL`; jiné nativní problémy `SUSC_GAMMA` přetrvávají.
5. **Specifická `Ano`:** `PHAGEN` jen konvertuje existující konvergovaný ASA/VWN potenciál s `NT=1`; `PLOTPOT`, `PLOTPRS`, `PLOTSFN`, `WFPLOT`, `SOCPAR`, `PSHIFT`, `MECHECK`, `MEPLOT` jsou lokální operace. Nepotvrzují samostatně SCF ani výpočet nekonečné soustavy.
6. **Verifikace:** klasifikace je **dostupnost podle zdrojových cest a dřívějšího Fortran auditu**, ne numerické srovnání clusteru s referencí. `Ano` není příslib přesnosti ani všech kombinací (`FULLPOT`, `MPI`, magnetismus, `ENERGY`, `CPA`). Omezení a defekty se musí aplikovat při validaci **konkrétní struktury a inputu**. Pro technickou implementaci metadata používat `supports={"3d","2d","cluster","embedded"}` s oddělenými podmínkami.

**Doporučení pro GUI:** jako podporu volného clusteru používat výlučně sloupec **Ano**; nikoli `TAU.CLUSTER` flag ani embedding-status. U postprocessingu vyžadovat odpovídající existující potenciál; u výpočetních tasků skutečně nastavit/ověřit `MOL`, `IBZINT` a datovou reprezentaci všech NQ míst.
