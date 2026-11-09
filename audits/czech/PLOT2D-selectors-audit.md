# PLOT2DPOT / PLOT2DRHO — SPR-KKR 9.7

Stav k **9. listopadu 2026**. Autoritou jsou zdroje SPR-KKR 9.7 **PUB9.7_260420_18h13**; rozhraní je ověřeno v ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Zdrojový základ a společné ověření](SPRKKR-ASE2SPRKKR-common-audit.md#zdrojový-základ-a-platnost).

| Task | Schéma | Program | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| PLOT2DPOT | PLOT2DPOT | kkrgen | false | true | obecný |
| PLOT2DRHO | PLOT2DRHO | kkrgen | false | true | obecný |

## Dispatch

initvar.f:201–205 nastavuje PLOT2DPR(1) podle PLOT2DPOT a flag2 podle PLOT2DRHO; oba mapují interní TASK=PLOT2DPR. GEN (gen.f:1466–1478) vybírá FPPLOT2DPR pro FULLPOT a PLOT2DPR_ASA pro ASA.

## Zásadní skutečné chování

ASA routine plot2dpr_asa.f neobsahuje čtení density arrays ani podmíněné rozlišení PLOT2DPR flag1/2. Vytváří potential PLY a geometrické line výstupy (pevná jména PLY-V-ASA-... a POT-ASA-LINES.ply). PLOT2DRHO proto na ASA není density plot. Definice odmítá explicitní FULLPOT=false pro PLOT2DRHO; skutečný potenciál musí být FULLPOT, flag jej nekonvertuje.

FULLPOT routine fpplot2dpr.f:70 nejprve bezpodmínečně volá PLOT2DPR_FP_PLY. Ten prochází potential i density arrays a nevětví výstupy podle jednotlivých PLOT2DPR flagů; na ř.926 volá STOP_REGULAR s textem run of test version completed. Následující běžný plane-grid výpočet je v této cestě nedosažitelný. Native selectory tedy neoddělují PLY potential/density produkty tak, jak jejich názvy slibují. Nejde jen o input validation problém.

Další STOP v fpplot2dpr.f:671 po zápisu FULLPOT-pot.ply je až v následné nedosažitelné rutině; nesmí být zaměňován za první ukončení. Oprava pouze tohoto STOP by nestačila. V této následné rutině se navíc V<-1 ořezává na -1 před zápisem PLY; nepovažujeme ji za aktuálně dosažitelný výstup.

## Geometrie a defaults

TASK.AVEC/BVEC/DVEC jsou tříprvkové kartézské vektory v jednotkách ALAT, nikoli přímo v a.u. ani frakční souřadnice báze; oba readers defaults A=(2,0,0), B=(0,2,0), D=0. B se při nenulovém A.B ortogonalizuje a zachovává svou původní délku. Nulové/paralelní vektory způsobují dělení nulou, definice je odmítá, stejně jako nefinite geometrie.

Důkaz jednotek: plot2dpr_asa.f:138–139 používá RWS/ALAT a ř.173 převádí prostorovou vzdálenost na radiální a.u. násobením ALAT; FULLPOT PLY dělá totéž na ř.259. Komentář ASA na ř.102 o a.u. je v rozporu s prováděnými převody. Python nápověda uvádí kartézské jednotky ALAT.

NA=50 explicitní default odpovídá readeru. NB=50 je explicitně zapsáno; při nepřítomném NB native dopočítává nint(NA*|B|/|A|), což pro velmi malý B může dát0. Explicitní50 je pro předvídatelnost bezpečnější, ale mění aspect-ratio sampling při různě dlouhých vektorech. NA/NB>=1.

PLOT2DPR_FP_PLY má vlastní řadu pevných sampling parametrů: NA/NB nejsou záruka běžného rectangular grid výstupu. Podpora input parametrů neznamená, že se v test writeru každý použije stejně jako v pozdějším plane-grid writeru.

GRID0/NE1/ImE0 jsou explicitní startup hodnoty; plot task nevykonává energetickou smyčku. Nepřítomné FULLPOT pro PLOT2DPOT nechává native rozpoznat formát potenciálu; PLOT2DRHO zapisuje true. MPI vypnuto, protože GEN guard plot tasky nepovoluje. DMFT/LDAU plot větve native rovněž odmítá; definice tyto MODE parametry nenabízejí.

## Ověření

Aktuální regresní moduly: `test_plot2dpr.py`, `test_plot_selector_file_manifests.py`. Jsou součástí společně ověřené sady vstupních parametrů a result_files (**1249 passed, 1 skipped**, 9. 11. 2026). Testy vstupu ověřují rozhraní a serializaci; necertifikují fyzikální hodnoty ani shodu serial/MPI. Samostatný numerický referenční výpočet tohoto tasku zde ověřen nebyl.
