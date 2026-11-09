# BIS — SPR-KKR 9.7

Stav k **9. listopadu 2026**. Autoritou jsou zdroje SPR-KKR 9.7 **PUB9.7_260420_18h13**; rozhraní je ověřeno v ASE2SPRKKR `49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6`. [Zdrojový základ a společné ověření](SPRKKR-ASE2SPRKKR-common-audit.md#zdrojový-základ-a-platnost).

Samostatný kanonický task v Python registru není vystaven; dosažitelnost nativní větve je popsána níže.

## Dosažitelná cesta

1. `initvar.f:264–271` rozpozná BIS a nastaví interní desetiznakový TASK na `BIS       `, RDTAUDIR=true a příslušné core flags.
2. `check_input.f:91–93` požaduje plně relativistický režim. Ani správný potenciál a TAU ale neodstraní následující překážku.
3. `gen.f:1445–1448` předá nezměněný TASK rutině VBPES.
4. `vbpes.f:174` kontroluje `TASK(3:5).NE.'PES'` a volá STOP_MESSAGE s textem `TASK ???`. Pro BIS je podřetězec `S  `; podmínka je vždy splněna. Výpočet může skončit dříve na jiné podmínce, ale přes tento bod nepokračuje.

Závěr: **rozbitá běžná vstupní cesta**, nikoli vypnutý placeholder. Přepsání BIS na VBPES by změnilo požadovaný výpočet na fotoemisi. Samotné odstranění guardu zase není dokladem implementace inverse photoemission.

## Rozhraní a meze ověření

Aktuální registr ASE2SPRKKR neobsahuje kanonický BIS task. Není vhodné přidávat falešný alias nebo doporučovat jeho „funkční defaulty“. Nejprve je nutná nativní implementace/oprava a fyzikální ověření počátečních/konečných energií a přechodových pravděpodobností, potom odpovídající Python rozhraní.

Tento závěr vychází z kontroly datového toku zdrojů; nebyl spuštěn nový numerický BIS výpočet. Python test napodobující uvedenou podmínku by nativní solver neověřoval.

## Ověření

Závěr o dosažitelnosti a nativních vadách je založen na uvedeném zdrojovém toku. Testy vstupu ověřují rozhraní a serializaci; necertifikují fyzikální hodnoty ani shodu serial/MPI. Samostatný numerický referenční výpočet tohoto tasku zde ověřen nebyl.
