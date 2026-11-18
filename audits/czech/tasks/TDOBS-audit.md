# TDOBS — SPR-KKR 9.7 / ASE2SPRKKR

Stav k **9. listopadu 2026**. Autoritou jsou zdroje SPR-KKR 9.7 **PUB9.7_260420_18h13**; rozhraní je ověřeno v ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Zdrojový základ a společné ověření](SPRKKR-ASE2SPRKKR-common-audit.md#zdrojový-základ-a-platnost).

| Task | Schéma | Program | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| TDOBS | TDOBS | kkrgen | false | true | obecný |

Vstupní volba pro generický DOS/observable tok kkrgen. Samostatný časově závislý fyzikální výpočet není ve zdrojích doložen.

## Doložené chování

initvar.f:523–530 rozpozná TDOBS, nastaví RDTAUDIR=true, WRTAUDIR=false a zapne initialization/energy loop. kkrgen.f:932–961 ověřuje tau cache: s dostupnou cache původně vypne RUNELOOP/INITELOOP, bez ní nastaví recomputation a zápis tau. Následně kkrgen.f:971–974 pro TDOBS vždy znovu zapne obě smyčky. GEN v energy loop volá CALCDOS (gen.f:869–876) a CALCDOS_WRITE (ř.1151). Žádná další dispatch větev TDOBS ve zdrojích nalezena nebyla.

CALC_VECOBS je odlišný interní přepínač, inicializovaný false v mod_calcmode.f:116; parser TDOBS jej nenastavuje. Název TDOBS proto neprezentujeme jako záruku vektorové nebo časově závislé analýzy. Obecné DOBS arrays samy takovou analýzu nedokládají.

GEN MPI guard (gen.f:1360–1371) TDOBS nepovoluje a končí STOP_REGULAR; mpi=False je nutné. Podpora obecné DOS energetické smyčky není numerické ověření všech případů CPA/FULLPOT či opětovného použití cache.

## Vstupy a defaults

TASK=TDOBS a CONTROL.ADSI=TDOBS se zapisují. TAU.TAUDIR/TAUPATH jsou optional bez Python defaultu: native použije DATASET.tau.dir a aktuální adresář (kkrgen.f:266–273). Cache musí odpovídat potenciálu, energii a konfiguraci; definice její obsah nekontroluje.

GRID3, NE300, ImE.01, EMIN-.2, EMAX1 Ry jsou explicitní spektrální defaults převzaté z auditované DOS definice. Nemusejí odpovídat generic native initial defaults, ale nejsou tiše vynechány. EMAX1 není EF sentinel a zahrnuje neobsazené stavy; užší okno může být vhodnější podle cíle. Neexponujeme RDTAUDIR jako nepravdivě přepsatelnou volbu: jeho true je native chování tasku. Chybějící tau nezpůsobuje automaticky konec; native zvolí recomputation fallback.

Validace NE>=1, GRID2/3 nejméně2 body, konečné bounds, ImE>=0 a warning při GRID3/ImE0. Definice obsahuje MODE, CPA a STRCONST a používá vlastní taskové schéma.

## Ověření

Aktuální regresní moduly: `test_tdobs.py`. Jsou součástí společně ověřené sady vstupních parametrů a result_files (**1249 passed, 1 skipped**, 9. 11. 2026). Testy vstupu ověřují rozhraní a serializaci; necertifikují fyzikální hodnoty ani shodu serial/MPI. Samostatný numerický referenční výpočet tohoto tasku zde ověřen nebyl.
