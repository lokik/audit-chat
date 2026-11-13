# VBUPS / VBXPS — aliasy VBPES v SPR-KKR 9.7

Stav k **9. listopadu 2026**. Autoritou jsou zdroje SPR-KKR 9.7 **PUB9.7_260420_18h13**; rozhraní je ověřeno v ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Zdrojový základ a společné ověření](SPRKKR-ASE2SPRKKR-common-audit.md#zdrojový-základ-a-platnost).

| Task | Schéma | Program | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| VBPES | VBPES | kkrgen | false | true | obecný |

Nativní aliasy VBPES: initvar.f:224–249 vybírá VBXPS/VBPES/ARPES/VBUPS a podle ANGLE_INTEGRATED přepíná interní VBPES nebo ARPES. Samostatné VBUPS/VBXPS bez dalších přepínačů zvolí VBPES; Python rozhraní používá task VBPES.

## Důležitý omyl v předpokládaných defaults

vbpes.f:197 čte TASK.EPHOT s defaultem1253.6 eV, pro oba aliasy stejně. Není větvení podle VBUPS/VBXPS. NINT(EPHOT)=1 znamená1253.6 a NINT(EPHOT)=2 znamená1486.6 eV. VBUPS tedy samo nenastaví21.2 ani40.8 eV. Fotonovou energii pro UV fotoemisi musí uživatel zadat explicitně v definici VBPES. Přidávat nepřítomný UV default, který se nezapíše, by bylo chybné; explicitně zapsaný21.2 eV by byl možný, ale byl by novou Python konvencí, nikoliv native defaultem aliasu.

## Kombinace s ARPES

VBXPS následně samostatně kontroluje ARPES a podle jeho přítomnosti mění ANGLE_INTEGRATED. Nejde o záruku, že libovolná kombinace tokenů zachová integrovaný výpočet. Nové aliasy nepředstavujeme jako náhradu KKRSPEC ARPES/AIPES; executable a native implementace jsou podstatné.

## Doporučené použití a výstup auditu

Používat existující definici VBPES a nastavit TASK.EPHOT v eV podle zdroje. Nevytváří se duplicitní samostatná input definition ani regresní test kopírující stejné native mapování. Tato kontrola doplňuje dřívější audit VBPES; neověřuje numerické PES výsledky.

## Ověření

Aktuální regresní moduly: `test_vbpes.py`. Jsou součástí společně ověřené sady vstupních parametrů a result_files (**1249 passed, 1 skipped**, 9. 11. 2026). Testy vstupu ověřují rozhraní a serializaci; necertifikují fyzikální hodnoty ani shodu serial/MPI. Samostatný numerický referenční výpočet tohoto tasku zde ověřen nebyl.
