# XAS — audit definice a nativního chování

Stav k **9. listopadu 2026**. Autoritou jsou zdroje SPR-KKR 9.7 **PUB9.7_260420_18h13**; rozhraní je ověřeno v ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Zdrojový základ a společné ověření](SPRKKR-ASE2SPRKKR-common-audit.md#zdrojový-základ-a-platnost).

| Task | Schéma | Program | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| XAS | XAS | kkrgen | false | false | obecný |

## Rozhraní a nativní podmínky

1. **MPI nedokončí spektrum.** GEN:1355–1368 po MPI energetické smyčce zastaví XAS i XES před voláním XRAYSPEC (1378–1385). Rozhraní používá mpi=False. Závěr vyplývá ze zdrojů; numerické MPI srovnání neproběhlo.
2. **EMAX je skutečný člen s defaultem 4 Ry.** ENERGY jej vytváří přes argument emax; lze jej standardně nastavit a 4 Ry se explicitně vypíší. Samotný klíč v defaults by takový parametr nevytvořil.
3. **MECHECK je samostatný task.** INITVAR:162–167 jej přepíná na test11 s jinými workflow flags. XRAYSPEC tuto volbu nečte a XAS ji nevystavuje. Kombinace taskových keywordů není doloženou podporou lokální XAS kontroly.
4. **CL vybírá core slupku:** povoleny jsou n=1..5, s/p/d/f a l<n. XRAYSPEC používá SHELL(5) pro K,L,M,N,O a indexuje NCXRAY na řádku 491. Existence slupky v konkrétním potenciálu vyžaduje kontrolu jeho obsahu.
5. **IT musí být kladný index typu.** Horní mez a existence zvolené core slupky závisejí na skutečném potenciálu.
6. **Rotace rámce:** FRAMEPHI, FRAMETET a FRAMEGAM jsou tři Eulerovy úhly; INIT_MROT_FRAME tvoří GETMROT(-gamma,-theta,-phi). Nejde o jediný vektor elektrického pole. XRAYSPEC počítá tři polarizační kanály.

## Parametry a validace

Definice obsahuje ME, POLOBS, DIPOL, GFCONVRH, WRREGIRR, NONDIPE2 a NONDIPM1 podle XRAYSPEC:295–329,442–465. POLOBS=S vypisuje (+,-,z), C vypisuje (x,y,z). ME=NAB přepíše DIPOL na1. Pokud jsou přítomné oba non-dipole flags, NONDIPE2 má přednost. Definice na obě kombinace upozorňuje.

Validace odmítá jiné MODE než FREL, neplatné core slupky, nečíselné rámcové úhly, nekladné/jednobodové NE a více energetických cest. CHECK_INPUT:43–47 přímo požaduje IREL=3. XRAYSPEC:376–380 požaduje shodu ENERGY.NE s počtem bodů prvního TAU contouru; obsah TAU a potenciálu modul nečte, takže tuto shodu nevydává za ověřenou.

Rozhraní nevystavuje MESPLIT (po čtení je K_ME_SPLIT vždy true), TAUCORE jako údajně účinné řízení XAS broadeningu ani DYNAMICAL bez auditu jeho samostatné rutiny. TAUCORE se společnou čtečkou načte, ale jeho účinné použití je ve větvích XMO/XRS; standardní call ME_ALF_ALF nastavuje CECOR=0. DYNAMICAL pro RXAS/XMO skutečně přepíná na XRAYSPECDYN, což není tento auditovaný statický workflow.

## Defaulty: zapisované a vynechané

| Volba | Nativní XAS | ASE2SPRKKR |
| --- | --- | --- |
| ENERGY.GRID | 6 | 6 explicitně |
| ENERGY.NE | 180 | 180 explicitně |
| ENERGY.ImE | 0,01 Ry | 0,01 explicitně |
| ENERGY.EMIN | po obecné čtečce XAS default EF | -0,2 Ry explicitně |
| ENERGY.EMAX | 4 Ry | 4 Ry explicitně, nastavitelný parametr |
| TASK.IT / CL | 1 / 2P | 1 / 2P explicitně |
| OUTPUT | BARN, výstup v Mb | None vynecháno; přijímá BARN, MBARN, SIGMA |
| POLOBS | S | S vynecháno |
| ME / DIPOL | ADA / 0 | ADA / 0 vynecháno |
| GFCONVRH | false při nepřítomnosti keywordu | false vynecháno |
| WRREGIRR | efektivně false: SECTION_FIND_KEYWORD přepíše DATA true | false vynecháno |
| Frame úhly | 0/0/0, bez keywordu CHANGE_FRAME=false | stejné nuly vynecháno |
| MODE | FREL | None vynecháno |

EMIN=-0,2 Ry není chybně vynechaný default: zapisuje se. Nativní EF je pro standardní absorpční interval zpravidla přirozenější a vyhne se části pod EF. Explicitní ASE2SPRKKR mez dovoluje požadovat širší interval; její skutečná účinnost závisí na nativní inicializaci níže. Konkrétní vhodnost závisí na EF a požadovaném spektru; konvergence nebyla testována.

Pozor: v INIT_MOD_ENERGY:234–249 se XAS při nepřítomnosti EMF vrací k EMIN=EF, takže samotné napsané absolutní EMIN nemusí vždy zůstat účinné. Relativní dvojice EMINEV/EMAXEV a její následné zpracování vyžadují kontrolu této větve. definice zde nezavádí fiktivní záruku, že Python default vždy určuje skutečný začátek nativního scanu.

Výstup SIGMA je v 10^15/s, ostatní OUTPUT používá Mb. OUTPUT přijímá BARN, MBARN a SIGMA. Default None se nevypisuje a deleguje jednotky na nativní BARN.

## Další nativní omezení

- MEFORM se načítá, NAB ovlivňuje DIPOL a GRV připravuje derivace. Standardní výpočet přesto volá ME_ALF_ALF bez předání MEFORM. Nelze tvrdit, že tři přijaté názvy znamenají tři nezávislé, numericky ověřené gauge implementace.
- Non-dipole call je jen pro .NOT.FULLPOT a DIPOL=0. FULLPOT proto nemusí vyhodnotit stejné příspěvky při stejných flags.
- Výpočet vybere jediný typ IT (NTXRAY=1). Nevystavuje seznam typů.
- Počet bodů, mesh a druh potenciálu se musí shodovat s TAU directory. Pouhá úspěšná serializace inputu tuto podmínku netestuje.
- XAS a XES se stejným DATASET, typem a slupkou mohou používat stejný .rat název. Je nutné rozlišovat výstupy při práci s oběma tasky.

## Ověření

Aktuální regresní moduly: `test_xas.py`, `test_xas_definition.py`. Jsou součástí společně ověřené sady vstupních parametrů a result_files (**1249 passed, 1 skipped**, 9. 11. 2026). Testy vstupu ověřují rozhraní a serializaci; necertifikují fyzikální hodnoty ani shodu serial/MPI. Samostatný numerický referenční výpočet tohoto tasku zde ověřen nebyl.
