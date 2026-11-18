# STONER — audit

Stav k **9. listopadu 2026**. Autoritou jsou zdroje SPR-KKR 9.7 **PUB9.7_260420_18h13**; rozhraní je ověřeno v ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Zdrojový základ a společné ověření](SPRKKR-ASE2SPRKKR-common-audit.md#zdrojový-základ-a-platnost).

| Task | Schéma | Program | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| STONER | STONER | kkrgen | false | true | obecný |

## Vstup a podporovaný workflow

STONER spouští kkrgen; TASK nečte žádné vlastní parametry. initvar.f:384–390 nastavuje ICORE=0, NOWRDOS=true a RDTAUDIR=true. gen.f:1592–1610 sestaví DOS váhy a volá SUSC_GAMMA.

Definice obsahuje CONTROL, TAU, MODE, STRCONST, ENERGY, TASK a SITES. Vystavuje konzervativní workflow s jednou reálnou energií na EF a relativistickými vlnovými funkcemi. Alternativní energetické skeny zdroj obecně dovolí, ale STONER pak používá aktuální DOBS_LTX a vlnové funkce; definice je odmítá, aby výsledek neoznačoval poslední energii skenu za EF. Toto je omezení definice, nikoliv tvrzení o nativním zákazu skenů.

## Defaulty

| Volba | Nativní STONER default | Definice | Zápis |
| --- | --- | --- | --- |
| ENERGY.GRID | 0 | 0 | explicitní |
| ENERGY.NE | 1 | 1 | explicitní |
| ENERGY.ImE | 0 Ry | 0 Ry | explicitní |
| ENERGY.EMIN | EF | None, delegace na SPR-KKR | vynechán |
| ENERGY.EMAX | EF | None, delegace na SPR-KKR | vynechán |
| ENERGY.EMINEV/EMAXEV | neuvedené | None | vynechány |
| ENERGY.SPLITSS | false | false | vynechán |

Defaulty energie jsou přímo v init_mod_energy.f:108–113. Ze společné definice ENERGY se nesmí převzít explicitní EMIN=-0,2 Ry: přepsal by nativní EF. Absolutní EF není při tvorbě vstupu známé, proto oba absolutní bounds zůstávají nepřítomné. Povoleny jsou také nulové relativní offsety; nativní čtečka uplatňuje relativní meze společně při přítomnosti obou. Nenulové offsety a explicitní absolutní meze tato definice odmítá.

Nevzniká zde rozdíl mezi fyzikálním nativním defaultem a defaultem ASE2SPRKKR. Obecná SITES.NL může být neurčena a přenechána potenciálu; validátor kontroluje pouze explicitní hodnoty. Skutečné NLQ jednotlivých míst z potenciálu musí být pro tento workflow dostatečné.

## Nálezy a omezení

1. **MPI nedokončí STONER.** gen.f:1355–1368 má seznam tasků povolených po energetické smyčce. STONER v něm není a MPI běh skončí STOP_REGULAR před voláním SUSC_GAMMA. Definice má mpi=False.
2. **Relativistický formát je použit bez větvení podle IREL.** SUSC_GAMMA čte WAVFUN_READ_REL (susc_gamma.f:108) a pracuje s relativistickými indexy kappa/mj. Definice odmítá NREL, SREL a SP-SREL. Jde o bezpečné omezení podle použitého formátu; v initvar není samostatný zákaz těchto režimů pro STONER.
3. **D kanál je použit i bez Brooks OP.** NLOP=3 je nastaven na začátku SUSC_GAMMA a GAMMAL(IR,NLOP,IS) se používá bez podmínky (řádek 207). SUSC_ORBPOL rovněž zapisuje PRE(4), přičemž PRE má velikost 2*(NLMAX-1). NL<3 tedy může znamenat přístup mimo pole nebo čtení nenaplněného kanálu. Definice odmítá explicitní NL<3. Nativní větev BROOKS-F používá NLOP=4 a potřebuje f kanál; přes sdílený MODE.OP tato zvláštní hodnota není vystavena a tato větev nebyla testována.
4. **Kernel je pevně VWN.** SUSC_GAMMA:236 volá přímo SUSC_VWN. Výsledek proto automaticky nepoužívá kernel stejného funkcionálu jako SCF potenciál, pokud ten byl vytvořen jiným funkcionálem. Definice nevymýšlí TASK.VXC.
5. **Malý DOS není fyzikálně neutrální fallback.** SUSC_GAMMA:87–90 při NEFT<=1e-8 nastaví všechny NEFTL=1 a NEFT=NL. Zdroj sám upozorňuje, že Stoner enhancement bude nereálný. Toto nelze předem zjistit jen ze vstupu.
6. **Normalizace má nechráněná dělení.** SUSC_ORBPOL:100 dělí BRACAH/ORBSQINT(NLOP,IS) bez kontroly nulového jmenovatele; diagnostický výstup SUSC_GAMMA:290–291 také dělí RHORINT. To jsou numerická rizika zdroje, nikoliv opravitelné defaulty vstupu.
7. **Nerovnoměrné cutoffy zaslouží numerický audit.** gen.f plní NEFTL podle globálního NL, SUSC_GAMMA je zpracovává podle lokálního NLQ. Bez ověření inicializace a konkrétního potenciálu netvrdím, že jsou takové konfigurace chybné.

FULLPOT není pro STONER v check_input explicitně zakázán. Nevytvářím zákaz jen podle sférické struktury této rutiny; fyzikální úplnost jejího FULLPOT výsledku však není ověřena. Totéž platí pro kombinace s korelacemi. Numerický běh SPR-KKR nebyl proveden, ani nevznikl specializovaný parser výstupu.

## Ověření

Aktuální regresní moduly: `test_stoner.py`. Jsou součástí společně ověřené sady vstupních parametrů a result_files (**1249 passed, 1 skipped**, 9. 11. 2026). Testy vstupu ověřují rozhraní a serializaci; necertifikují fyzikální hodnoty ani shodu serial/MPI. Samostatný numerický referenční výpočet tohoto tasku zde ověřen nebyl.

## Podpora embeddingu — doplňující revize 18. 11. 2026

**Klasifikace: Podmíněně; není to povolení spouštění v ASE2SPRKKR.**
Běžný executable je `kkrgen`; pro embedded cluster se používá speciální `embgen` (nativní program `CLUGEN`), nikoli `kkrgen` s dodatečným parametrem.

### Ověřená výpočetní cesta

1. `initvar.f:384–390` rozpozná `STONER`, nastaví `RDTAUDIR=.TRUE.`, `ICORE=0` a `NOWRDOS=.TRUE.`. Jde o žádost o znovupoužití TAU, nikoli bezpodmínečné čtení cache.
2. `kkrmain.f:932–959` zavolá `TAU_DIR_INQUIRE`. Pokud TAU cache **není dostupná**, přepne `RDTAUDIR=.FALSE.` a `RUNELOOP=.TRUE.`. To opravuje dřívější chybný předpoklad, že STONER vždy čte předpočtenou hostitelskou TAU.
3. `cludrive.f:398–425,619–658` sestaví host+cluster systém, omezí aktivní typy na `NTHOST+1:NTHOST+NTCLU` a pro `CLUGEN` volá `GEN`. `gen.f:744–807` vytvoří jednocentrové vlnové funkce a při `RDTAUDIR=.FALSE.` volá `TAU_DRIVE` místo `TAU_DIR_READ`. Takto se může dostat k embeddingové výpočetní větvi, stejně jako `DOS`.
4. `gen.f:869–875` volá `CALCDOS` a `gen.f:1592–1610` následně `SUSC_GAMMA`, tedy existuje dosažitelná nativní cesta k parametrům STONER.

### Nevyřešené otázky vlastní podpory embeddingu

**Přímá kontrola `SUSC_GAMMA` (18. 11. 2026):** Zpřístupněná sada zdrojů `PUB9.7_260420_18h13` v knihovně obsahuje 478 souborů a končí abecedně na `scfmad0d_back.f`; `susc_gamma.f` v ní není. V otevřených podkladech nebylo možné získat její kompletní zdroj. Proto **nelze tvrdit, že bylo ověřeno**, zda vlastní smyčky `SUSC_GAMMA` běží přes `ITBOT:ITTOP`, nebo přes `1:NT`. Z doložených `gen.f:1592–1610`, `calcdos.f:246–253` a `cludrive.f:414–425` lze potvrdit pouze: `NEFTL` má hodnoty DOS clusteru a nulové hodnoty hostitele a celé pole se předá rutině `SUSC_GAMMA`. Samotné předání všech prvků není důkazem jejich použití. **Vhodné ověření**: získat kompletní `susc_gamma.f`, projít všechny vnější smyčky přes typy, `WAVFUN_READ_REL`, výpočet `NEFT` a DOS fallback a zjistit, zda se hostitelské typy dostávají do akumulovaných výsledků. Výběr TAU je samostatný problém a v tomto hodnocení nepředstavuje omezení.

- **Indexy atomových typů:** `calcdos.f:246–253` vynuluje celé `DOBS_LTX` a naplní jen `ITBOT:ITTOP`, tj. cluster. `gen.f:1601–1605` přesto sestaví `NEFTL` pro **všech** `IT=1..NT`. Zda `SUSC_GAMMA` správně omezuje vlastní smyčky na `ITBOT:ITTOP` a nepoužívá nuly hostitele (případně je nenahrazuje umělými jednotkami), není v této revizi potvrzeno. Starší audit upozornil i na fallback `NEFTL=1` při malém DOS (`susc_gamma.f:87–90`).
- **Numerika:** chybí nativní end-to-end test pro `embgen`, nezávislé srovnání s hostitelským a clusterovým referenčním výpočtem i ověření významu lokálního Stonerova parametru. Standardní omezení FREL, `NL>=3`, VWN kernelu a vypnutého MPI tím nezanikají.

**Rozhodnutí pro ASE2SPRKKR:** `TaskDefinition.supports_embedded_cluster=False` ponechat; `embgen` pro STONER zatím nevybírat. Pozitivní změnu podmínit ověřením smyček a datového kontraktu `SUSC_GAMMA` pro cluster a nativním výpočetním testem. **Předpokládáme správné vstupní TAU; jeho výběr, původ a případné znovupoužití sem nepatří.**
