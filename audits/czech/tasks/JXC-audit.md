# JXC — audit existující definice

Stav k **9. listopadu 2026**. Autoritou jsou zdroje SPR-KKR 9.7 **PUB9.7_260420_18h13**; rozhraní je ověřeno v ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Zdrojový základ a společné ověření](SPRKKR-ASE2SPRKKR-common-audit.md#zdrojový-základ-a-platnost).

| Task | Schéma | Program | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| JXC | JXC | kkrgen | true | false | JXC |

## Rozsah a opravy

Definice podporuje JXC. INITVAR čte klíčové slovo JXC, nastaví CALC_JXC a interní task E-INTEGRAL. GEN:930–990 rozlišuje SP-SREL (IREL=2) s XCPLJ0/XCPLJIJ a FREL (IREL=3) s XCPLTENSOR. Validace odmítá NREL/SREL a přijímá SP-SREL; rozdíl pokrývají regresní testy.

Definice vystavuje nativně čtené TASK.NSHLCLU, SELECT, EXCLUDE a QREPONLY. INIT_MOD_TAUIJ_STAR:125–161 čte seznamy chemických symbolů, zakazuje jejich současné použití a při přítomném NSHLCLU dává shell count přednost před CLURAD. QREPONLY čte INITVAR uvnitř E-INTEGRAL větve; ovlivňuje seznam centrálních site v INIT_MOD_TAUIJ_STAR. CLURAD je v jednotkách ALAT, nikoli angstromů.

Konečná validace odmítá NREL/SREL/NONMAG, nekladné NE, více energetických cest, neauditované integrace a současné SELECT/EXCLUDE. Pro režim a NONMAG existuje také warning_condition. Pro standardní periodický výpočet je BZINT=POINTS explicitní. GEN:358–386 pro off-diagonal TAUIJ vyžaduje IBZINT=2.

Rozsah této definice je běžný periodický JXC na GRID5. Ve zdroji existují další větve embedded-cluster, occupation-weighted GRID3 (WGT_OCC), MULTISPIN, SLC_JXC a SLC_TRQ. Nemají zde kompletní audit; validátor proto není tvrzením, že SPR-KKR obecně nikdy neumí GRID3 nebo cluster. Pro zpřístupnění rozšířených větví je nutný samostatný krok.

## Defaulty a jejich zápis

| Volba | SPR-KKR | Definice |
| --- | --- | --- |
| TASK.CLURAD | bez NSHLCLU se vyžaduje hodnota; obecný default není | explicitně2,2 ALAT |
| NSHLCLU | při nepřítomnosti volí CLURAD | None, vynecháno |
| SELECT/EXCLUDE | prázdné seznamy | None, vynecháno |
| QREPONLY | false při nepřítomnosti | false, vynecháno |
| ENERGY.GRID | 5 pro E-INTEGRAL | explicitně5 |
| ENERGY.NE | 30 pro E-INTEGRAL | aktuálně explicitně30 |
| ENERGY.ImE | obecně0,01 Ry | aktuálně explicitně0,01 Ry |
| ENERGY.EMIN | -0,2 Ry | explicitně -0,2 Ry |
| ENERGY.EMAX | obecná náhrada sentinelu za EF | vynecháno, přebírá se EF |
| MODE | běžně FREL | None, vynecháno; SP-SREL se zapisuje při zadání |
| TAU.BZINT | obecná volba závisí na workflow | explicitněPOINTS pro doložený off-diagonal TAU tok |

JXC zapisuje NE=30 a ImE=0,01 Ry. DMI sdílí stejné schéma s vlastní taskovou identitou. GRID5 tvoří vlastní komplexní oblouk. Poloměr 2,2 není zárukou konvergovaného J_ij; NE, spodní mez, k-mesh a dosah součtu vyžadují numerické ověření.

## Závažné ALE ve SPR-KKR

1. **Vypnutá rotace podle symetrie ve FREL:** XCPLTENSOR:312 má JQ.EQ.IQREPQ(JQ) .OR. 1.EQ.1. Druhá větev s rotací ME_ROT_MREG tedy nikdy neproběhne. Pro nereprezentativní sousední site může být použitá matice ve špatném rámci. QREPONLY omezuje centrální site, neřeší automaticky všechny sousedy. Toto neoznačuji za rozbití každého jednoatomového kolineárního běhu; jde ale o důležitou chybu obecného tensorového workflow.
2. **FREL má potenciálové podmínky:** NONMAG=false, KMROT=0, NKM=NKMMAX a reprezentativní IQAT(1,IT) jsou přímo kontrolovány XCPLTENSOR:74–80,155. Python zná explicitní NONMAG, ostatní podmínky vyžadují skutečný potenciál. Nelze zadáním QREPONLY certifikovat všechny jeho vlastnosti.
3. **Více cest není zpracováno obecně:** rutiny používají ETAB/WETAB první cesty a mají NEPATH=1 guard. Pozdější vynucení E-INTEGRAL tuto podmínku nezmění na podporu split contouru.
4. **Clusterový SP-SREL není běžný periodický tok:** CLUXCPLJIJ čte scratch unit19, ale TAU_DRIVE běžně volá TAU_REAL_SPACE s KTAUIJ=0, který její export vypne. Bez auditu alternativního připravení unit19 není správné slíbit tento workflow jen přepnutím BZINT=CLUSTER. FREL rozlišuje embedded-cluster zvlášť.
5. **Vyšší spinové a spin-lattice větve mají vlastní interface:** GEN načítá MULTISPIN/SLC_JXC/SLC_TRQ a volí jiné rutiny. Tato definice neprezentuje tyto keywordy jako plně podporované pouhým přidáním názvu.

Na rozdíl od BSDOSIJ používá XCPLTENSOR pro JC správný offset BAR_M_JT_JQ(1,1,JC) a výstupní J_ITJT má konzistentní párové indexování. Chyby z BSDOSIJ nelze mechanicky připsat JXC jen kvůli podobnému zdroji.

## MPI a výstup

mpi=True má zdrojové opodstatnění: J_IJINT je při prvním volání celé nulováno, MPI_SUM_C provede redukci a GEN:1193–1241 má dodatečné dokončovací volání. SP-SREL má vlastní XCPLJIJ redukci. Numerická shoda serial/MPI nebyla měřena.

FREL zapisuje DATASET_XCPLTEN_Jij.dat, tensor v meV (DIMAG(J_IJINT)/pi*RY_EV*1000), a další výstupy; SP-SREL má jinou strukturu a scalar Jij. Tento audit nemění stávající output parser ani konvenci Hamiltoniánu/faktory převzaté dalším softwarem.

### Prefix výstupu

V názvech výstupu této rodiny znamená nativní DATSET prefix rozšířený o neprázdné ADSI, tedy Python šablonu `{DATASET_ADSI}`. Zápis DATASET v popisu názvu souboru se nesmí zaměnit za neexpandovaný DATSET0; pevná jména zůstávají uvedena zvlášť.

## Ověření

Aktuální regresní moduly: `test_jxc.py`, `test_jxc_definition.py`, `test_exchange_file_manifests.py`. Jsou součástí společně ověřené sady vstupních parametrů a result_files (**1249 passed, 1 skipped**, 9. 11. 2026). Testy vstupu ověřují rozhraní a serializaci; necertifikují fyzikální hodnoty ani shodu serial/MPI. Samostatný numerický referenční výpočet tohoto tasku zde ověřen nebyl.
