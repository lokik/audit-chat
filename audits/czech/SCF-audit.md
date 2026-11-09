# SCF — audit ASE2SPRKKR proti SPR-KKR 9.7

Stav k **9. listopadu 2026**. Autoritou jsou zdroje SPR-KKR 9.7 **PUB9.7_260420_18h13**; rozhraní je ověřeno v ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Zdrojový základ a společné ověření](SPRKKR-ASE2SPRKKR-common-audit.md#zdrojový-základ-a-platnost).

| Task | Schéma | Program | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| SCF | SCF | kkrscf | true | false | SCF |

## Podporované volby a validace

1. Veřejné scf.f:427–438 přijímá BROYDEN2, ANDERSON a TCHEBY. Tasková definice nabízí tyto tři názvy; PULAY se na ANDERSON nemapuje. Místní upravené SPRKKR/9_7/scf.f PULAY implementuje, ale není totožné s auditovaným archivem.
2. TCHEBY s FULLPOT odmítá nativní kód i validace. FULLPOT může být aktivován i potenciálem, což input-only validace sama nezjistí.
3. NITER a ISTBRY jsou kladná celá čísla. ITDEPT omezen na 3..40: SCFBROYDEN přepisuje hodnoty pod 3 na 40 a hodnoty nad 40 omezuje na kapacitu. Definice nyní nepředstírá, že tyto požadavky budou provedeny přesně.
4. MIX, MIXOP a TOL musí být konečné kladné hodnoty; EFGUESS konečná energie. Horní hranice MIX nebyla svévolně stanovena na 1, protože overrelaxace není pouhým typovým omylem.
5. NE je kladné; jednobodový GRID=2/3 odmítnut kvůli nativním integračním vadám. Ostatní pokročilé grid kombinace nejsou tímto auditem kompletně kontrolovány.
6. EFGUESS je počáteční odhad EF v Ry a vynechání počátečního hledání při tvorbě startovního potenciálu. Nezakazuje všechny pozdější aktualizace EF v SCF.

Taskové kopie SCF a ENERGY izolují omezení SCF od dalších definic používajících společné sekce. Společná sekce ALG může v jiných taskových definicích stále obsahovat PULAY; to není potvrzení jeho nativní podpory.

## Defaults a jejich zápis

Běžné hodnoty NITER=200, MIX=0.2, TOL=1e−5, ISTBRY=1 a ITDEPT=40 se vypisují a odpovídají inicializaci ve scf.f:360–371. BROYDEN2 odpovídá MOD_SCF defaultu a je default taskové definice. MOD_SCF inicializuje VXC=VWN, ale konkrétní potenciál a reader mohou vstupní stav dále ovlivnit. Nativní energetický default je GRID=5/NE=30 podle INIT_MOD_ENERGY, aktuální Python však explicitně zapisuje GRID=5/NE=32; EMIN=−0.2 je vypsaná ASE volba, EMAX se nevypisuje a ponechává native EF.

**MIXOP je důležitý nevypsaný případ:** scf.f nejprve nastaví SCFMIX=0.2 a SCFMIXOP=SCFMIX, až potom čte MIX a MIXOP. Při MIX=0.5 a nezadaném MIXOP tedy nelze tvrdit, že MIXOP bude 0.5; pro běžný neexportní postup zůstává 0.2. ASE nevnucuje jiný skrytý default, MIXOP je nezadaný a nevypisuje se. Popis tuto závislost nyní výslovně upozorňuje. Doplnit explicitní MIXOP je lepší, pokud chce uživatel stejnou hodnotu jako změněný MIX; automatická změna defaultu by měnila native chování.

EFGUESS se bez zadání nevypisuje; native rozhoduje podle přítomnosti klíče, nikoli podle libovolné náhradní energie. FULLPOT=False se nevypisuje; není příslibem, že se full-potential potenciál změní na ASA. TAU/CPA/STRCONST/SITES/MODE jsou společné sekce, zde bez zásahů.

## Další omezení a větve

- Native EXPORT_HARTREE_POTENTIAL nastavuje NITER=1, MIX=1 a přeskočí čtení VXC/NITER/MIX. Tabulka běžných defaults není přenosná na tuto větev.
- ANDERSON mapuje IMIX=7, BROYDEN2 IMIX=4. TCHEBY využívá samostatný postup; název algoritmu není synonymem pro stejný numerický mix.
- Vyčerpání NITER není důkaz konvergence. Je nutné kontrolovat skutečný konvergenční stav, ne pouze existenci nového potenciálu nebo úspěšné parsování vstupu. Tato definice neupravuje výstupní parser ani calculator lifecycle.
- Native nabízí další řízení jako SCFRESTART, MIXRHO, SIM, MAGCOUPL, SEMICORE a 2D layer charge parametry. Definice je nevystavuje a jejich celé workflow není tímto auditem pokryto.
- Definice deklaruje mpi=True. SCF má paralelní řešení a počáteční search; numerická shoda MPI/serial v tomto auditu nebyla testována.

## Ověření

Aktuální regresní moduly: `test_scf_audit.py`. Jsou součástí společně ověřené sady vstupních parametrů a result_files (**1249 passed, 1 skipped**, 9. 11. 2026). Testy vstupu ověřují rozhraní a serializaci; necertifikují fyzikální hodnoty ani shodu serial/MPI. Samostatný numerický referenční výpočet tohoto tasku zde ověřen nebyl.
