# PLOTPOT / PLOTRHO / PLOTSFN — SPR-KKR 9.7

Stav k **9. listopadu 2026**. Autoritou jsou zdroje SPR-KKR 9.7 **PUB9.7_260420_18h13**; rozhraní je ověřeno v ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Zdrojový základ a společné ověření](SPRKKR-ASE2SPRKKR-common-audit.md#zdrojový-základ-a-platnost).

| Task | Schéma | Program | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| PLOTPOT | PLOTPOT | kkrgen | false | true | obecný |
| PLOTRHO | PLOTRHO | kkrgen | false | true | obecný |
| PLOTSFN | PLOTSFN | kkrgen | false | true | obecný |

## Native mapování

initvar.f:187–200: PLOTPRS zapne všechna výstupní flags. Pokud není PLOTPRS přítomno, PLOTPOT zapne flag1, PLOTRHO flag2, PLOTSFN flag3; libovolný aktivní flag nastaví interní TASK=PLOTPRS. Tyto selektory lze v native inputu kombinovat, jednotlivé definice poskytují samostatné canonical volby.

GEN dispatch gen.f:1461–1463 volá FPPLOTPRS. Routine fpplotprs.f:8–11 a příslušné větve poskytují:
- PLOTPOT: uložené VT/BT, u FULLPOT navíc VNST/BNST;
- PLOTRHO: uložené RHOCHR/RHOSPN, u FULLPOT navíc RHO2NS;
- PLOTSFN: uložené radiální koeficienty FLMSF a LMISF shape functions pro jednotlivé meshe, pouze FULLPOT (ř.392). Nejde o generátor SFN ani přímo o 3D vizualizaci buňky.

Neprovádí SCF ani DOS. Plotuje se existující potenciál/density/SFN data, kvalita a dostupnost závisí na POTFIL. Native výstupy jsou Grace/XMGR, ne obrázky vytvořené ASE2SPRKKR.

## Defaults a omezení

TASK i ADSI mají explicitní příslušný selector. GRID0, NE1, ImE0 jsou explicitní startup hodnoty, nikoliv tvrzení o native plot defaults; energetická smyčka pro PLOTPRS je vypnutá. EMIN/EMAX nejsou exponovány, tato větev je nepotřebuje.

PLOTSFN explicitně zapisuje SCF.FULLPOT=true a odmítá jeho false. Flag nepromění ASA potenciál na správný FULLPOT soubor; skutečný format a SFN data musí být přítomná v potenciálu. Pro ASA native PLOTSFN tiše nevytvoří SFN grafy. Nepřítomné SCF.FULLPOT ponechává rozpoznání formátu souboru nativnímu programu. PLOTPOT má ASA i FULLPOT větev. **Samostatné PLOTRHO je v obou formalismech vadné:** RHORSQ se alokuje jen v potenciálové větvi (fpplotprs.f:56–62), zatímco hustotní větev je používá bez této alokace (219–254,280). PLOTPRS s potenciálovou větví obchází tuto chybu, nikoli chybu společného datasetu spinové/orbitální křivky (282–287).

V GEN jsou PLOTPRS větve zakázány pro DMFT/LDAU (gen.f:215–223). Definice tyto MODE volby nenabízejí; přítomnost/platnost uložených korelačních dat potenciálu nebyla numericky certifikována. MPI je vypnuté, native GEN MPI guard PLOTPRS nepovoluje. Pokud metadata některých typů/meshů nemají odpovídající data, samotná input definice to neodhalí.

Zařazení: PLOTPOT má smysluplnou diagnostickou cestu; PLOTSFN vyžaduje skutečný FULLPOT potenciál; samostatné PLOTRHO je rozbité kvůli writeru.

## Ověření

Aktuální regresní moduly: `test_plotprs.py`, `test_plot_selector_file_manifests.py`. Jsou součástí společně ověřené sady vstupních parametrů a result_files (**1249 passed, 1 skipped**, 9. 11. 2026). Testy vstupu ověřují rozhraní a serializaci; necertifikují fyzikální hodnoty ani shodu serial/MPI. Samostatný numerický referenční výpočet tohoto tasku zde ověřen nebyl.
