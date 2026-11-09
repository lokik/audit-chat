# GILBERT — audit SPR-KKR 9.7 a ASE2SPRKKR

Stav k **9. listopadu 2026**. Autoritou jsou zdroje SPR-KKR 9.7 **PUB9.7_260420_18h13**; rozhraní je ověřeno v ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Zdrojový základ a společné ověření](SPRKKR-ASE2SPRKKR-common-audit.md#zdrojový-základ-a-platnost).

| Task | Schéma | Program | MPI | Experimental | Reader |
|---|---|---|---|---|---|
| GILBERT | GILBERT | kkrchi | true | false | obecný |

## Definice a validace

- SETFLUCT=MLIN podporuje lineární M(T)=1−DMAGDT*T i kombinaci s vibracemi; vystaveny jsou DMAGDT, TDEBYE a INITSVEC.
- M_T vyžaduje FLUCTFIL; ignorované TMPMIN/TMPMAX/NTMP validace odmítá.
- Definice obsahuje MODE, CPA a STRCONST. Auditovaný rozsah je magnetický FREL s BZINT=POINTS; nejde o tvrzení, že každá jiná nativní větev končí STOP.
- Validace vyžaduje kladné celočíselné počty, konečné nezáporné a uspořádané teploty, DMAGDT pouze s MLIN a úplnou teplotní síť pro vibrace bez SETFLUCT.
- GRID=0/NE=1 zajišťuje jednoenergetickou inicializaci. GIL nastavuje ERYD=EFERMI; obecné EMIN/EMAX nejsou ovládáním damping výpočtu a definice je nevystavuje. SPLITSS je odmítnuto.

## Defaults: vypsané versus nevypsané

GRID=0 a NE=1 se vypisují; jde o jednoenergetickou inicializaci. ImE=0 se vypisuje, vlastní damping však počítá na EF. SITES.NL=4 se rovněž vypisuje: jde o explicitní ASE default, nikoli vynechaný nativní fallback. Vhodnost NL závisí na systému a konvergenci.

Teplotní a fluktuační parametry nemají vnucené Python defaults a při nezadání se nevypisují. Native NFTET/NFPHI začínají na 1, INITSVEC na 0. NVIBRA je bez požadavku 1, ale při požadavku vibrací a INITSVEC=0 jej native přepíše na kanonických 14 směrů. Například NVIBRA=26 tedy neznamená 26 směrů; pro generovanou síť je nutné zvolit INITSVEC=1. TDEBYE=0 znamená tabulkovou hodnotu podle složení, nikoli automaticky dělení nulou. Záporné TDEBYE/RMSU0 postupy nejsou touto definicí pokryty.

Pro MLIN je vyžadován explicitní TMPMIN, protože nativní výchozí počáteční hodnota 9999 není užitečným běžným zadáním. Bez TMPMAX native používá jeden bod v TMPMIN; NTMP bez TMPMAX by ignoroval, proto se odmítá.

## Významná omezení nativního výpočtu

1. GIL vyžaduje KMROT=0 a IBZINT=2 nebo 6 (gil.f:89–93). Definice zpřístupňuje standardní POINTS. KMROT a magnetické momenty závisí na potenciálu a tato validace je z něj neověřuje.
2. gilsum.f:180–204 počítá GEFF=2*(1+Morb/Mspin) a další normalizaci bez ochrany před nulovým momentem. Výpočet probíhá i před podmínkou tisku Z>0. Prázdné nebo nemagnetické komponenty a kompenzovaný celkový moment mohou způsobit neplatné výsledky. CONTROL.NONMAG=False tomu samo nezabrání.
3. INITVAR zapíná RDTAUDIR; může být potřeba kompatibilní inicializační TAU soubor. Potenciál ani externí data zde nebyly numericky ověřeny.
4. Native obsahuje MPI větev v gil.f; existující ASE nastavení paralelizace nebylo měněno ani numericky certifikováno. MCS, RDLM a NLCPA nejsou součástí tohoto auditu.

## Opora ve zdrojích

SPR-KKR 9.7: initvar.f:376–382; kkrchi.f:1425–1427; rsp.f:913–925; gil.f:89–121,266–320; thermal_init.f:79–81,105–119,170–242,322–368,548–558; mod_thermal.f:16; thermal_rmsdisp.f; gilsum.f:180–204.

## Ověření

Aktuální regresní moduly: `test_gilbert.py`. Jsou součástí společně ověřené sady vstupních parametrů a result_files (**1249 passed, 1 skipped**, 9. 11. 2026). Testy vstupu ověřují rozhraní a serializaci; necertifikují fyzikální hodnoty ani shodu serial/MPI. Samostatný numerický referenční výpočet tohoto tasku zde ověřen nebyl.
