# BSF — audit společné definice BSFEK/BSFKK

Stav k **9. listopadu 2026**. Autoritou jsou zdroje SPR-KKR 9.7 **PUB9.7_260420_18h13**; rozhraní je ověřeno v ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Zdrojový základ a společné ověření](SPRKKR-ASE2SPRKKR-common-audit.md#zdrojový-základ-a-platnost).

| Task | Schéma | Program | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| BSFEK | BSF | kkrgen | true | false | BSF |
| BSFKK | BSF | kkrgen | true | false | BSF |

## Definice a validace

- NK1/NK2 a NK mají minimum 2, NE minimum 1. BLOCHSF při parametrizaci K-K mapy dělí NK1−1 a NK2−1 (blochsf.f:939–940).
- Dynamický GRID je 3 pro E-K a 0 pro K-K a zapisuje se. Jednobodový GRID=3 nemá inicializovanou integrační váhu.
- K-K validace odmítá nekonečné/NaN směry a nulovou či kolineární rovinu vzhledem ke KA.
- K1/K2 jsou koncové body; BLOCHSF od nich odečítá KA. Nekolmé relativní směry ortogonalizuje, takže zadaný rovnoběžník nemusí zachovat (blochsf.f:438–455).

## Defaults: vypsané versus nevypsané

E-K: zachovány vypsané NE=200, GRID=3, NK=300, ImE=0.001 Ry, EMIN=−0.2 Ry a EMAX=−1 Ry. **EMAX=−1 je skutečná energie, nikoli sentinel pro EF.** Native init_mod_energy používá náhradu EF až pro EMAX<−99. Výchozí rozsah −0.2→−1 je tedy klesající a pro běžné spektrum překvapivý. Toto je upozornění na explicitní ASE default, nikoli oprava parametru ignorovaného native readerem. Je vhodné nastavit vlastní rozsah, případně v budoucnu default změnit na fyzikálně srozumitelný rozsah vůči EF; bez potenciálu nelze určit vhodnou absolutní horní energii.

Native NK při čtení E-K je 800, ASE zapisuje 300. Jde o platnou explicitní odchylku; 800 může být lepší pro jemné cesty, ale konečný počet se rozděluje mezi segmenty a může se upravit podle kapacity k-tabulek. Ani 300 ani 800 není zárukou konvergence.

K-K: vypsaná energie EMIN=0.7 Ry je explicitní ASE volba, EMAX kopíruje EMIN; nejde automaticky o EF. Vypsané NE=1 a GRID=0 odpovídají jednomu bodu. Pro hledání Fermiho plochy je třeba nastavit správnou energii či relativní EMINEV. Nevypsané KA native doplní nulovým počátkem; Python validace používá tentýž počátek. NK1/NK2/K1/K2 nemají vymyšlené defaults a musí je dodat uživatel.

Definice deklaruje mpi=True. BLOCHSF má paralelní energetický postup a GEN jej připouští; tento audit však neověřuje numerickou shodu MPI výsledku. MODE/CPA/STRCONST/SITES zůstávají společné sekce beze změn.

## Další omezení

E-K se vybírá skutečným počtem energií NE>1, K-K při NE=1; BSFEK/BSFKK jsou samostatné Python taskové identity se sdíleným schématem BSF a vlastními počátečními defaulty, nikoli dva rozdílné native dispatch klíče. TASK se zapisuje jako BSF, CONTROL ADSI jako BLOCHSF. Existující validace varuje při rozporu požadovaného aliasu a NE.

V E-K režimu native rozděluje body podle délky segmentů, s minimem dvou bodů na segment. Degenerované explicitní segmenty a nulová celková délka mohou vést k problematickému rozdělení. Jejich úplná kontrola, spolu s geometrií každé KPATH/Bravais kombinace, není v tomto auditu řešena.

Pro 2D native v K-K větvi nuluje z-složky K1/K2 před odečtením KA. Input-only kontrola nezná dimensionalitu potenciálu, takže rovina platná podle zadaných 3D vektorů může po nativním přepisu degenerovat. Pro 2D je nutné explicitně konzistentní rovinné zadání.

Při native čtení TAU cache může BLOCHSF převzít počet energií z cache; konzistence externích souborů zde není ověřena. Tento audit nedoplňuje advanced BSFSPN/BSFORB/BSFETD/BSFALF ani EPHOT.

## Ověření

Aktuální regresní moduly: `test_bsf_audit.py`, `test_bsf_file_manifests.py`. Jsou součástí společně ověřené sady vstupních parametrů a result_files (**1249 passed, 1 skipped**, 9. 11. 2026). Testy vstupu ověřují rozhraní a serializaci; necertifikují fyzikální hodnoty ani shodu serial/MPI. Samostatný numerický referenční výpočet tohoto tasku zde ověřen nebyl.
