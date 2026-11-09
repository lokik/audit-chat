# SIGMA — SPR-KKR 9.7 / ASE2SPRKKR

Stav k **9. listopadu 2026**. Autoritou jsou zdroje SPR-KKR 9.7 **PUB9.7_260420_18h13**; rozhraní je ověřeno v ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Zdrojový základ a společné ověření](SPRKKR-ASE2SPRKKR-common-audit.md#zdrojový-základ-a-platnost).

| Task | Schéma | Program | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| SIGMA | SIGMA | kkrchi | false | true | obecný |

DC vodivost a vybrané response tensory v kkrchi, v auditovaném periodickém ASA/FREL rozsahu. Jednobodová Fermi-surface větev není kompletním Fermi-sea výpočtem.

## Mapování a možnosti

INITVAR rozpoznává TASK SIGMA, vyžaduje KKRCHI a pro běžnou větev bez NLCPA vypíná RDTAUDIR/WRTAUDIR. RSP volá SIG. Definice obsahuje CONTROL, TAU, MODE, CPA, STRCONST, ENERGY, TASK, SITES; minimální SCF dovoluje odmítnout FULLPOT jako mimo rozsah auditu. Native obsahuje FULLPOT rutiny, takže toto odmítnutí není tvrzením, že SIGMA FULLPOT nefunguje.

Auditovaný režim je FREL, BZINT=POINTS, bez many-body OP. NONMAG je přípustný. Native nabízí také IBZINT=4 a 6 a nerelativistické větve, které tato definice necertifikuje.

- GRID=0, NE=1: pouze Fermi-surface postup. SIG ignoruje energii inicializačního bodu a skutečně počítá EF+i*ImE.
- GRID=5, NE>1: jedna integrační kontura pro Fermi-sea část a dodatečný bod na EF pro Fermi-surface část. SPLITSS a jiné kombinace nejsou touto definicí pokryty.
- SIGSPROJ vybírá operátory 1..16; validuje se rozsah, délka, absence duplicit a nativní požadavek prvního indexu 1 nebo 9. SP_NO_ALPHA a SP_NO_NABLA upravují spinový polarizační operátor. NOSYMSIG vypíná i globální native symetrii, nikoli jen tisk nebo formát výsledku.
- FRAMETET/FRAMEPHI/FRAMEGAM jsou Eulerovy úhly response rámce ve stupních.

## Významná ALE

1. **NE=1 vynechá Fermi-sea integrál.** Native nastavuje NEINTEG=0; při NE>1 používá všechny body kontury a přidává bod na EF. Jednobodový výsledek nelze obecně označit za úplnou Hallovu či všechny ostatní odezvy. Pro příslušné tenzory je nutné zahrnout a konvergovat Fermi-sea část. Jde o rozdíl obsahu výpočtu, nikoli pouze jeho přesnosti.
2. **Pevný SIG_MODE=STREDA.** V SIG je napevno nastaveno STREDA a numerická energetická derivace používá DERYD=0.0001 Ry. Definice neslibuje přepnutí formalismu ani změnu tohoto kroku vstupním klíčem.
3. **K-mesh se za běhu mění.** SIG_DRV_KMESH odvozuje počet bodů pro jednotlivé energie z NKTABINP; při vyšším NKTAB jsou hlubší energie vzorkovány řidčeji, například podle (NEA+1−IEA)^3 a dolní hranice NKTABINP/1000. NKTAB není příslib stejného počtu bodů v celé kontuře. Společný default 250 není dokladem konvergence transportu.
4. **ImE není elektronová teplota.** Jde o imaginární část energie v Ry. Explicitních 0.01 Ry odpovídá přibližně 0.136 eV a může podstatně ovlivnit čistý materiál. Native nula nemusí být lepší, protože pro čisté systémy je DC limit singularní; vhodnou šířku a její konvergenci je nutné zvolit podle systému.
5. SIGSPROJ bez zadání ve FREL vybírá všech 16 operátorů, včetně spinových, torque a Edelsteinových odpovědí. Pro samotnou elektrickou vodivost je užitečné explicitně SIGSPROJ={1}; definice však zachovává nativní nevypsaný výběr. Reprezentace nabla-nabla má index 9.
6. Native PROJECT, NLCPA, thermal a many-body postupy zde nejsou auditovány. PROJECT provádí oddělené projekční průchody, ne pouhé rozdělení běžného tensoru. Native má MPI k-loop; zde ponecháno mpi=False, protože celý paralelní řetězec nebyl certifikován. Není tím prokázána chyba MPI.
7. SIGSUM zapisuje pevný název sigmaXXXXX.dat podle teplotního indexu a STATUS=REPLACE. Opakovaný výpočet ve stejném adresáři může přepsat předchozí soubor; použití jiného DATASET samo tento název nemění.

## Defaults a jejich zápis

| Parametr | Native | Definice |
|---|---|---|
| GRID / NE | SIGMA: 0 / 1 | 0 / 1, vypsáno; vynechává sea integrál |
| ImE | SIGMA: 0 Ry | 0.01 Ry, vypsáno; záměrná odchylka |
| EMIN | pro SIGMA nejprve EF | −0.2 Ry, vypsáno; surface bod jej nepoužívá, využije se při zvolené kontuře |
| EMAX | EF | nevypisuje se; zůstává skutečné native EF |
| SIGSPROJ | FREL: 1..16 | nezadáno, nevypisuje se; stejný native výběr |
| SP_NO_ALPHA / SP_NO_NABLA | false | false, nevypisují se |
| NOSYMSIG | hledání přítomnosti klíče | false, nevypisuje se |
| Eulerovy úhly | 0 a změna rámce podle přítomnosti klíče | nezadány, nevypisují se |

Defaults, které se nevypisují, nebyly nahrazeny jiným implicitním chováním. Default sea+surface s GRID=5/NE=30 by lépe pokryl obecnou Hallovu odezvu, ale byl by výrazně dražší a jeho EMIN i mesh by stále vyžadovaly konvergenci; ponechán nativní rychlý surface default s výslovným popisem omezení. FULLPOT=False se nevypisuje a nepřevádí full-potential potenciál na ASA; pokud jej aktivuje potenciál, input-only validace to sama nezjistí.

## Ověření

Aktuální regresní moduly: `test_sigma.py`. Jsou součástí společně ověřené sady vstupních parametrů a result_files (**1249 passed, 1 skipped**, 9. 11. 2026). Testy vstupu ověřují rozhraní a serializaci; necertifikují fyzikální hodnoty ani shodu serial/MPI. Samostatný numerický referenční výpočet tohoto tasku zde ověřen nebyl.
