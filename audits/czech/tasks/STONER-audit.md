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
