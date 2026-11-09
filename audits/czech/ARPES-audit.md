# ARPES — audit SPR-KKR 9.7 a ASE2SPRKKR

Stav k **9. listopadu 2026**. Autoritou jsou zdroje SPR-KKR 9.7 **PUB9.7_260420_18h13**; rozhraní je ověřeno v ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Zdrojový základ a společné ověření](SPRKKR-ASE2SPRKKR-common-audit.md#zdrojový-základ-a-platnost).

| Task | Schéma | Program | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| ARPES | ARPES | kkrspec | false | false | ARPES |

## Definice a validace

- ENERGY.EWORK_EV je **výstupní práce v eV**, nikoli vnitřní potenciál. SPEC_INPUT jej čte a převádí /RY_EV; v příslušné větvi při zadání nastavuje EWORK.
- NT, NP a NK1..NK4 jsou kladná celá čísla; NK1=1 zůstává přípustné. Reader má pro jedno k explicitní větev a dělení NT−1/NP−1 chrání podmínkou >1. Vyžadovat minimum 2 by neoprávněně vyřadilo platný native postup.
- Při K1 se vyžaduje NK1, při současném K2 také NK2. Validace odmítá záporné a nulové počty i nesoulad angular ranges a NT/NP.
- Pro ARPES odmítnuty K3/K4/NK3/NK4: v dodaném SPEC_INPUT readeru se tyto klíče nikde nečtou. Ve veřejné Python definici zůstávají pro zpětné parsování, ale validace nepředstírá jejich účinek.
- Konečné nezáporné EWORK_EV a obě IMV_*_EV, kladné konečné EPHOT; konečné elektronové úhly a k-vektory. Nulové tlumení není svévolně zakázáno.
- N_LAYDBL musí mít přesně dvě nezáporná celá čísla. N_LAYER a NLAT_G_VEC jsou kladná celá čísla. INPVER/STRVER omezeny na 0/1. Při INPVER=0 je nové řízení moderního readeru vynecháno, protože data pocházejí ze starého externího input.inp.

## Defaults: vypsané versus nevypsané

| Parametr | ASE default | Srovnání a zápis |
|---|---|---|
| EWORK_EV | 4.2 eV | vypsáno; přepíše workfunction ze SCF/potenciálu |
| IMV_INI_EV / IMV_FIN_EV | 0.05 / 2 eV | vypsáno; native photoemission start VIL/VIH je 0.1/1 eV |
| NE | 300 | vypsáno; native photoemission start EMESH=200 |
| EMINEV / EMAXEV | −8 / +5 eV vůči EF | vypsáno; explicitní spektrální rozsah, ne obecná fyzikální mez |
| NL | 4 | vypsáno; explicitní angular cutoff |
| NT / NP | 1 / 1 | vypsáno, konzistentní s nulovým rozsahem úhlů |
| Electron THETA/PHI | 0..0 pro ARPES | task-specific default; SPLEED má 45/270 stupňů |
| EPHOT | 25 eV | vypsáno; explicitní experimentální volba |
| N_LAYER | 50 | vypsáno; native photoemission start 30 |
| NLAT_G_VEC | 57 | vypsáno; native start 30 |
| N_LAYDBL | 10 / 10 | vypsáno; native má překlep LANZ1=10 dvakrát místo inicializace LANZ2 |
| INPVER / STRVER | 1 / 1 | vypsáno; moderní vstup a generování struktury |

Tyto odchylky se zapisují explicitně. N_LAYER=50 či více reciprocal vectors mohou zlepšit konvergenci za cenu času; nelze bez systému doporučit nativní menší čísla jako lepší. Výstupní práce a tlumení jsou citlivé materiálové/experimentální parametry: hodnota z potenciálu může být vhodnější než generická 4.2 eV, ale její použití vyžaduje vynechat tento explicitní klíč. Default není pouhé formátovací nastavení.

Expert photon polarization a spin-amplitude defaults se při nezadání obecně nevypisují. Tento audit nepředstavuje kompletní certifikaci všech kombinací POL_P/NPOL/ICIRC/IDREH ani POL_E/POL0/POL0L. Reader podle přítomnosti alternativních klíčů některé kombinace odmítá; samotná shoda numerické default hodnoty nestačí k zaměnitelnosti zápisu a vynechání.

## Konkrétní nativní chyby a omezení

1. **IMV_INI v Ry má překlep:** spec_inputs.f:1257 volá SECTION_SET_REAL('IMV_INI',VIH,...), tedy zapisuje do final-state proměnné místo VIL. ASE nabízí IMV_INI_EV, jehož větev zapisuje správně do VIL; chybný Ry alias není vystaven. VIL/VIH reader následně převádí do interního spektroskopického zadání v eV.
2. **Dvojí LANZ1 inicializace:** reader nastavuje LANZ1=10 dvakrát. Explicitní N_LAYDBL={10,10} v ASE nastaví obě proměnné, a je proto lepší než slepé vynechání pole s očekáváním obou native defaults.
3. K1/K2 v tomto readeru jsou **posunové vektory přičtené ke KA**, nikoli endpointy jako v BLOCHSF. Počet bodů určuje NT/NP a zadané vektory se použijí pro k-mesh. Názvy nelze interpretovat jednotně mezi tasky.
4. Native photon convention uvádí normal incidence THETA=180 stupňů. Nelze automaticky přenášet geometrický význam THETA z jiných tasků. Specifika krystalového rámce a sample tilt je nutné ověřit pro konkrétní povrch.
5. Validace neověřuje existenci a obsah externího input.inp/struc.inp při INPVER/STRVER=0, soulad povrchové struktury s potenciálem ani konvergenci layer doubling a reciprocal cutoffů.
6. Rozhraní používá kkrspec a mpi=False. CRYS_VECS=False se nevypisuje a ponechává USE_CRY_PRIM_VECS=true. Ve 3D konstrukci povrchu přítomnost CRYS_VECS nastaví USE_CRY_PRIM_VECS=false (spec_rslab.f:180–181). Stejnou definici sdílejí ARPES, AIPES, BAND, SPLEED a CLPES; singulární CRYS_VEC nativní čtečka nečte.

## Ověření

Aktuální regresní moduly: `test_arpes_audit.py`, `test_arpes_energy.py`. Jsou součástí společně ověřené sady vstupních parametrů a result_files (**1249 passed, 1 skipped**, 9. 11. 2026). Testy vstupu ověřují rozhraní a serializaci; necertifikují fyzikální hodnoty ani shodu serial/MPI. Samostatný numerický referenční výpočet tohoto tasku zde ověřen nebyl.
