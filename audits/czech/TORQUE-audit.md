# TORQUE — audit existující definice a oprava validace

Stav k **9. listopadu 2026**. Autoritou jsou zdroje SPR-KKR 9.7 **PUB9.7_260420_18h13**; rozhraní je ověřeno v ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Zdrojový základ a společné ověření](SPRKKR-ASE2SPRKKR-common-audit.md#zdrojový-základ-a-platnost).

| Task | Schéma | Program | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| TORQUE | TORQUE | kkrgen | true | false | obecný |

## Co znamenají parametry

THETAQ/PHIQ v TASK nejsou rotace magnetizace. Určují jednotkový vektor u, osu vyhodnocovaného torquu. TORQUE:74–75 nejprve vyžaduje MOMENTS_ROTATED; KKRGEN:846 nastaví tento příznak pro KMROT=1 nebo2. MROTQ pochází z natočení momentů. TORQUE:168–175 kontroluje m dot u = 0 s tolerancí1e-8. Úhly TASK samy tuto podmínku ani rotaci momentů nevytvoří.

Pořadí hodnot je podle site IQ, nikoli atomového typu IT. SECTION_SET_REAL_ARRAY (scan_input.f:701–707) opakuje poslední dodaný prvek do zbývajících site. Jednoprvkový seznam tedy skutečně platí pro všechny site. Delší pole nesmí překročit NQ; definice bez potenciálu nezná NQ a tuto mez nekontroluje. Délky obou seznamů nemusejí být stejné, protože každý se rozšiřuje samostatně.

## Defaulty

| Parametr | SPR-KKR při vynechání | ASE2SPRKKR |
| --- | --- | --- |
| THETAQ/PHIQ | když chybí oba:45°/0°; pokud je zadán jen jeden, druhé pole zůstane0 | explicitně90°/90° |
| ENERGY.GRID | TORQUE nemá vlastní inicializační default | explicitně8 |
| ENERGY.NE | bez přepsání MOD_ENERGY začíná0 | explicitně36 |
| ENERGY.ImE | 0,01 Ry | explicitně0 Ry |
| ENERGY.EMIN | -0,2 Ry | explicitně -0,2 Ry |
| ENERGY.EMAX | obecná náhrada sentinelu za EF | vůbec není členem této definice, nativně EF |
| MODE | běžně FREL | None, vynecháno |

90°/90° dává u podél y. Nativní45°/0° dává směr v rovině xz. Ani jedna volba není univerzálně lepší: u musí být kolmé ke skutečné magnetizaci. Default ASE2SPRKKR je explicitně zapisovaný, takže jeho odlišnost sama není omyl. Případnou změnu je vhodné volit společně s konvencí natočení momentů; tato definice default nemění.

GRID=8 používá logaritmickou integrační konturu. EPATH:249–264 nastavuje pevné EPS=0,050 a předává ho do EGRID8_IME, místo aby tento parametr odvodil z ENERGY.ImE. Samotný rozdíl explicitního ImE=0 vůči nativnímu0,01 tak neprokazuje rozdílnou konturu GRID8. Vhodnost GRID8/36 proti jiné integrační kontuře vyžaduje konvergenční srovnání.

ENERGY.EMIN je explicitní -0,2 Ry; horní mez se deleguje na nativní EF. Rozhraní nepoužívá neúčinný klíč EMIN ve slovníku defaults.

## Definice a validace

- THETAQ/PHIQ jsou neprázdné seznamy reálných hodnot; při konečné validaci se odmítá NaN/Inf.
- ENERGY.NE musí být kladné.
- Odmítá se SPLITSS a více cest GRID/NE: TORQUE čte vždy WETAB(IECURR,1) a ETAB(IECURR,1), nikoli obecnou cestu. Bez ochrany mohl vstup vzbudit dojem výpočtu obou příspěvků.
- Dokumentace parametrů rozlišuje osu torquu od natočení magnetizace a popisuje rozšiřování seznamů.

Definice varuje pro NREL/SREL a NONMAG. Jejich text tvrdí, že výpočet selže, ale vlastní TORQUE nemá kontrolu NONMAG/IREL. Používá plně relativistické operátory J=L+S a závisí na okolním workflow. Skutečnou dostupnost jiných režimů nelze rozhodnout pouhým tímto varováním; do definitivního nativního zákazu se nepřevádí. SP-SREL toto varování nepokrývá. Toto zůstává nedořešená otázka podpory režimů, nikoli doložený funkční scalar-relativistic TORQUE.

## Nálezy a omezení nativního výpočtu

1. Požadavek MOMENTS_ROTATED a kolmosti u vůči m není ze samotných InputParameters ověřitelný. Existující default vstup se standardním nenatočeným potenciálem může skončit STOP.
2. Rutina nastavuje IPRINT=1 bez ohledu na CONTROL.PRINT a produkuje rozsáhlé matice i v tichém vstupu.
3. J a UJ se plní do globálního NKM, ale lokální násobení používá NKMQ(IQ). WKM4 pak kopíruje rozdíl do 1:NKM, tedy i mimo lokálně právě naplněné submatice při rozdílných cutoffs. Konkrétní dopad na trace je nutné ověřit; není to dokázaná chyba všech výsledků.
4. Při WGT_OCC se bere pouze ITOQ(1,IQ). Site s více komponentami tak vyžaduje kontrolu významu vážení podle prvního typu.
5. Hlavní výsledky jsou textový výpis site torquů a součtu; jednotlivé T(IQ) jsou v interních Ry, součet se také vypíše přepočtený na meV. TASK nemá vlastní volbu výstupního souboru ani nový Python parser.

MPI=true je zde podložené: TORQUE:186–199 redukuje T přes MPI_SUM_R a na procesu0 dokončí výpis; GEN:1264 má dodatečné volání pro energetickou paralelizaci. To odlišuje TORQUE od nedokončených MPI větví některých předchozích tasků. Numerické MPI výsledky nebyly ověřeny.

## Ověření

Aktuální regresní moduly: `test_torque.py`. Jsou součástí společně ověřené sady vstupních parametrů a result_files (**1249 passed, 1 skipped**, 9. 11. 2026). Testy vstupu ověřují rozhraní a serializaci; necertifikují fyzikální hodnoty ani shodu serial/MPI. Samostatný numerický referenční výpočet tohoto tasku zde ověřen nebyl.
