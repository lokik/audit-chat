# BSDOSIJ — audit

Stav k **9. listopadu 2026**. Autoritou jsou zdroje SPR-KKR 9.7 **PUB9.7_260420_18h13**; rozhraní je ověřeno v ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Zdrojový základ a společné ověření](SPRKKR-ASE2SPRKKR-common-audit.md#zdrojový-základ-a-platnost).

| Task | Schéma | Program | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| BSDOSIJ | BSDOSIJ | kkrgen | true | true | obecný |

## Vstup a defaulty

TASK BSDOSIJ spouští site-resolved back-scattering DOS v kkrgen; initvar.f:540–554 mění interní task na E-INTEGRAL. GEN:358–386 připravuje off-diagonal TAUIJ a vyžaduje IBZINT=2, tedy BZINT=POINTS. INIT_MOD_TAUIJ_STAR čte TASK.CLURAD nebo NSHLCLU, seznamy chemických symbolů SELECT/EXCLUDE a používá QREPONLY z INITVAR.

| Volba | Nativní chování při vynechání | Definice a zápis |
| --- | --- | --- |
| GRID | E-INTEGRAL nastaví 5, BSDOSIJ následně vyžaduje 3 | explicitně 3; nutná oprava nefunkčního nativního defaultu |
| NE | 30 | explicitně 300 pro energetické rozlišení; konvergence neověřena |
| ImE | 0,01 Ry | explicitně 0,01 Ry |
| EMIN | -0,2 Ry | explicitně -0,2 Ry |
| EMAX | EF | vynecháno, None deleguje na EF |
| CLURAD | žádný použitelný default; bez NSHLCLU program zastaví | explicitně 2,2 ALAT, stejně jako existující JXC definice |
| NSHLCLU | nepřítomnost volí CLURAD | vynecháno; při zadání má přednost před CLURAD |
| SELECT / EXCLUDE | prázdné seznamy | vynecháno; vzájemně výlučné, nejvýše 100 symbolů |
| QREPONLY | false při nepřítomnosti klíčového slova | false; nezapisuje se |
| MODE | běžně FREL | vynecháno; jiné MODE odmítnuty |

Rozdíly GRID, NE a CLURAD jsou explicitní, nikoli chybné vynechané defaulty. Nativní GRID=5 není pro tento task lepší: vede k STOP. NE=30 může stačit pro hrubou kontrolu, ale 300 poskytuje jemnější energetickou síť; vhodnost musí ověřit výpočet. Poloměr 2,2 není univerzálně konvergovaný. ImE zůstává nativní.

Definice odmítá NONMAG, jiné MODE, jiný GRID, dvě energetické cesty, SPLITSS, nekladné NE a jiné BZINT. Zdroj navíc vyžaduje KMROT=0, NKM=NKMMAX a reprezentativní IQAT(1,IT); tyto podmínky závisejí na potenciálu a modul je neověřuje. QREPONLY mění centrální sites, ale nezaručuje splnění všech těchto kontrol.

## Závažné nálezy ve SPR-KKR

1. **Komponenta matice:** bsdosij.f:319–349 iteruje IC=1..3, ale první CMATMUL dostává BAR_M_JT_JQ bez offsetu IC. Předává tak počátek pole, tedy první komponentu, zatímco druhá lokální matice používá IC. To zřejmě zaměňuje komponenty příspěvku na JQ.
2. **Vypnutá symetrická rotace:** podmínka JQ.EQ.IQREPQ(JQ) .OR. 1.EQ.1 na řádku325 je vždy pravdivá. Větev rotace ME_ROT_MREG je nedosažitelná.
3. **Chybné výstupní indexování:** J_ITJT je alokováno jako (NIJ,NT,NT,3,3), ale řádky434–436 ukládají J_ITJT(IE,I,IT,JT,IC), jako by mělo energetickou dimenzi. IE může překročit NIJ, I překročit NT a JT překročit 3. Následné čtení používá zase indexy (I,IT,JT,component,component). I při náhodném dodržení mezí tedy nečte správně uložené hodnoty; nevyplněné části nejsou inicializovány.
4. **Výstup kopíruje exchange coupling:** soubor se jmenuje DATASET_XCPLTEN_Jij.dat a další výstupy/ploty jsou označeny jako výměnné vazby. Finální zpracování neprochází všechny energie BSDOS_IJ. Název a zamýšlené DOS chování tak neodpovídají dokončenému výstupnímu workflow.

Tyto chyby neopravuje definiční soubor Pythonu. Podpora vstupu je experimentální; numerické výsledky SPR-KKR 9.7 nejsou tímto auditem potvrzeny.

## MPI

mpi=True odpovídá implementované redukci BSDOS_IJ přes MPI_SUM_C (bsdosij.f:112–126) a výstupu na procesu0. Samotná numerická správnost MPI nebyla testována.

### Prefix výstupu

V názvech výstupu této rodiny znamená nativní DATSET prefix rozšířený o neprázdné ADSI, tedy Python šablonu `{DATASET_ADSI}`. Zápis DATASET v popisu názvu souboru se nesmí zaměnit za neexpandovaný DATSET0; pevná jména zůstávají uvedena zvlášť.

## Ověření

Aktuální regresní moduly: `test_bsdosij.py`. Jsou součástí společně ověřené sady vstupních parametrů a result_files (**1249 passed, 1 skipped**, 9. 11. 2026). Testy vstupu ověřují rozhraní a serializaci; necertifikují fyzikální hodnoty ani shodu serial/MPI. Samostatný numerický referenční výpočet tohoto tasku zde ověřen nebyl.
