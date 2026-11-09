# CLPES — SPR-KKR 9.7 / ASE2SPRKKR

Stav k **9. listopadu 2026**. Autoritou jsou zdroje SPR-KKR 9.7 **PUB9.7_260420_18h13**; rozhraní je ověřeno v ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Zdrojový základ a společné ověření](SPRKKR-ASE2SPRKKR-common-audit.md#zdrojový-základ-a-platnost).

| Task | Schéma | Program | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| CLPES | CLPES | kkrspec | false | true | ARPES |

CLPES je samostatné schéma. Je dostupné přes `InputParameters.create("CLPES")` i `for_task("CLPES")` a je zahrnuto v `task_registry.names()` i v kanonickém výpisu schémat. Registr nabízí 55 taskových identit; sdílená schémata JXC/DMI a BSFEK/BSFKK zůstávají rozlišena od identity tasku.

Core-level fotoemise a fotoelektronová difrakce přes KKRSPEC, IBLOCH=3 (`spec_rslab.f:172–173,226,239–241,862`; driver `spec_clpes.f`). Rozhraní omezuje auditovaný postup na sériový výpočet s pevnou elektronovou geometrií.

## Parametry a defaults

SPEC_CL je skutečný název sekce (spec_inputs.f:1877–1921). Implementováno IT (explicitně 1), CL (explicitně 2P), IE_ICST (default 1). CL je vždy zapsáno: bez přítomného CL reader ponechá 1S, přestože SECTION_SET_STRING uvádí fallback 2P. Volitelné CE není v této definici exponováno; IT určuje potenciálový typ a native výběr zahrnuje také stejný atomový počet Z (spec_clpes.f:213–216), nejde o izolovaný site selector.

NE=300 zapsáno, single mesh bez SPLITSS. Driver přepisuje energetické okno rozsahem core energií rozšířeným o +/-3 eV (spec_clpes.f:301–306); EMIN/EMAX nejsou uživatelské spektrální meze. NE=1 vybírá IE_ICST a validuje se počet 4*l+2. Core energie jsou vztažené k EF (ř.224).

EPHOT=1000 eV se explicitně zapisuje; uživatel musí ověřit práh konkrétního core shellu. Nejde o záruku dostupnosti každé core hladiny. Explicitní EWORK_EV=4.2 eV, IMV_INI_EV=.05 eV, IMV_FIN_EV=2 eV, NL=4, N_LAYER=50 a NLAT_G_VEC=57 nejsou deklarovány jako nativní defaults. CRYS_VECS=False se nevypisuje a ponechává USE_CRY_PRIM_VECS=true. Ve 3D konstrukci povrchu přítomnost CRYS_VECS nastaví USE_CRY_PRIM_VECS=false (spec_rslab.f:180–181). Stejnou definici sdílejí ARPES, AIPES, BAND, SPLEED a CLPES; singulární CRYS_VEC nativní čtečka nečte. Nepřítomné NPOL odpovídá nativnímu 1; 0 a 3 vyvolají STOP a validátor je odmítá.

## Velká ALE

- NCPA>0 vede k STOP (spec_clpes.f:157). Definice neověřuje obsazení potenciálu a nemůže zaručit ordered systém.
- Auditovaný rozsah je pevná konečná elektronová geometrie. Angular a momentum scans jsou odmítnuty; starý externí input INPVER=0 jejich kontrolu obchází.
- NE=1 používá ECOR/XNORM po smyčce atomů (spec_clpes.f:310–319); pro více typů/site může odpovídat poslední zpracované sadě, ne požadovanému typu. Tuto větev nelze považovat za obecně ověřenou.
- Vybraný shell musí v potenciálu existovat jako core shell, IT musí existovat a EPHOT musí překročit práh. Python validace bez potenciálu tyto podmínky nezaručí.
- MPI nebylo certifikováno, automatické použití je vypnuté; to není tvrzení, že native MPI chybí.

## Ověření

Aktuální regresní moduly: `test_clpes.py`. Jsou součástí společně ověřené sady vstupních parametrů a result_files (**1249 passed, 1 skipped**, 9. 11. 2026). Testy vstupu ověřují rozhraní a serializaci; necertifikují fyzikální hodnoty ani shodu serial/MPI. Samostatný numerický referenční výpočet tohoto tasku zde ověřen nebyl.
