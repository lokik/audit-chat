# Společný audit SPR-KKR 9.7 a ASE2SPRKKR

## Zdrojový základ a platnost

Stav k **9. listopadu 2026**. Autoritou je prováděný kód SPR-KKR: čtečky vstupu, dispatch, alokace, výpočet a výstupní rutiny. Název tasku, komentář nebo úspěšný Python test samy nedokazují fyzikální význam výsledku.

Nativním základem je veřejný archiv **PUB9.7_260420_18h13.tar.gz**, SHA-256 `65c2f6e178fc127946ff8309cf0000c7ed9d64f51d8bec1e03c057147c872ff4`, lokálně rozbalený v `SPRKKR/9.7/`. Obsahuje 770 Fortranových souborů. Všech 110 zdrojových souborů ověřovaných při aktualizaci se s archivem shoduje po bajtech; čísla řádků v auditech platí pro tento základ.

Místní vývojový strom `SPRKKR/9_7` není totožná verze: jeho upravené `scf.f` implementuje také PULAY. Závěr o nepodporovaném PULAY platí pro veřejný archiv. Existující binárky v `9_7` nebyly pro toto ověření nově přeloženy z archivu; úspěch místního běhu proto není certifikací veřejné sestavy.

Python rozhraní bylo ověřeno v ASE2SPRKKR **49d7d0f1254dbc98cd7ae9a483b28747ddbe9bf6** (`develop`); nápověda PLOT2D uvádí kartézské jednotky ALAT. Definice jsou integrovány společně; samostatné definice se neinstalují. Rozsah numerického ověření uvádí sekce 7.

## Rozsah a význam kategorií

Audit zahrnuje dosud prošlé tasky, aliasy a selektory ve větvích INITVAR/GEN/RSP/KKRSPEC, včetně celého TUTORIAL. Není úplným auditem všech interních algoritmů, korelačních metod nebo experimentálních kombinací přepínačů.

| Kategorie | Význam |
|---|---|
| **Mají plný smysl** | Dosažitelná implementace odpovídá vystavenému účelu v uvedeném rozsahu; nebyla nalezena zásadní překážka tohoto workflow. Běžná omezení formalismu ani potřeba konvergence sama o sobě nejsou „velké ALE“. |
| **ALE — 2A: víceméně fungují** | Hlavní auditovaná cesta má smysl; vyžaduje vhodný stav či vstup nebo omezení na funkční větev. |
| **ALE — 2B: víceméně nefungují pro uvedený účel** | Hlavní deklarovaná nebo výslovně uvedená větev je nedokončená, dává vadná data nebo její fyzikální význam není doložen. Nestačí jen dodat správný stav. |
| **Rozbité** | Je doložena chyba dosažitelnosti, inicializace, indexování nebo datového toku, která blokuje běžný účel či znehodnocuje jeho výsledek. Definice vstupu ji neopravuje. |
| **Vypnuté** | Příslušná cesta je výslovně zastavena nebo nahrazena nedostupnou rutinou. Jejich společný seznam je až na úplném konci. |

Zařazení se vztahuje k dodané revizi SPR-KKR. U tasku s odlišnými větvemi je rozsah uveden výslovně. Platný input, úspěšný pytest ani regulérní ukončení programu nejsou důkazem správné fyzikální hodnoty.

## 1. Tyto tasky mají plný smysl

| Task / větev | Účel a auditovaný rozsah | Podstatné hranice podpory |
|---|---|---|
| **SCF** | Self-consistent výpočet; podporované volby BROYDEN2, ANDERSON, TCHEBY. | PULAY není platná volba čtečky veřejného archivu; místní upravené 9_7 jej podporuje. TCHEBY není pro FULLPOT. Exportní a zvláštní korelační workflow nebyla obecně certifikována. |
| **DOS** | Standardní DOS přes generickou GEN smyčku; nativní interní TASK je NONE. | GRID, počty a okno musí odpovídat požadovanému spektru. Pokročilé DOSREP větve mají zvláštní MPI rizika a nejsou automaticky pokryté. |
| **BSF** | Bloch spectral function; energetický scan EK a jednoenergetický řez KK. | KK není automaticky na EF. Existující EK default EMAX=-1 Ry je podezřelý explicitní interval; viz defaults níže. Mapování všech k cest a 2D transformace nebyly kompletně ověřeny. |
| **EKREL** | E(k) a Fermiho povrch pro uspořádané systémy. | CPA se odmítá nativně. TASK.EMIN/EMAX/NE řídí vlastní hledání, nikoli ENERGY. Preset k cesty závisejí na mřížce; ESLICE=EF může být mimo hledací interval. |
| **WFPLOT** | Diagnostické grafy vlnových funkcí BAND/CORE v ASA, FREL. | Používá první energetický bod; dostupnost slupky a atomového typu musí určit potenciál. FULLPOT a uvedené korelace nejsou podporované. |
| **CLXPS** | Core-level XPS v ASA/FREL s implementovanou ADA větví. | Jen implementované formy ME; konečný stav musí být nad EF. Geometrie mají skutečné suffixy 0 nebo souvislé 1..10. Kapacita ENERGY.NE musí stačit core stavům. |
| **CHI** | Auditovaná statická odezva v ASA/FREL, POINTS, serial. | NCHIITER je nativně vždy 1; nejde o iterativní solver řízený touto volbou. Lokální CHIORBPOL není totožné s obecným MODE.OP. |
| **POSANITAU** | Positronová annihilační integrace přes vlastní TAU cestu v ASA/FREL. | Vhodnost energetické meze EF a nulové dílčí rychlosti je nutné posoudit; 1/RATET není chráněno před nulou. Nejde o náhradu přípravy POSANI. |
| **JORB** | Orbitální proudová hustota, FREL, serial. | Potenciál musí poskytovat potřebnou úhlovou reprezentaci. Pomocné fort.* nejsou jednoznačný hlavní výstup; MPI dokončení nefunguje. |
| **PLOTPOT** | Selektivní graf uloženého potenciálu, v ASA i FULLPOT. | Neprovádí SCF. Data a jejich mesh musí být přítomna v potenciálu. |
| **PLOTSFN** | Graf uložených radiálních koeficientů shape functions ve FULLPOT. | Nevytváří SFN ani 3D tvar buňky; FULLPOT flag nepřemění ASA potenciál. V ASA se SFN graf nevytvoří. |
| **TUTORIAL: YLM, JLX, JLX-ASYMPT** | Matematické ukázky harmonik, Besselových funkcí a asymptotik. | L/M a meze jsou validovány; i matematické větve procházejí společnou inicializací a potřebují použitelné vstupní prostředí. |
| **TUTORIAL: DOS-FEG, DOS-SS, DOS-CLU, DOS-BULK, GEN** | Ukázky DOS / běžného GEN workflow v odpovídajícím modelu. | FEG a SS jsou single-site STANDARD. CLU vyžaduje skutečnou clusterovou geometrii a CLURAD/NSHL; samotný název tutorialu ji nevytvoří. BULK vyžaduje periodickou integraci. |

U BSF je třeba opravit nebo záměrně potvrdit výchozí okno před použitím bez vlastních parametrů. U POSANITAU patří nechráněný nulový dílčí příspěvek mezi konkrétní vadné případy; neznamená to, že každá nenulová celková annihilační integrace je rozbitá.

## 2. Tasky s ALE — rozdělení podle praktické použitelnosti

### 2A. Víceméně fungují: je třeba správný stav, vstup nebo omezení na funkční větev

V této skupině má hlavní auditovaný výpočet smysluplnou dosažitelnou cestu. Typicky je třeba vhodný potenciál, kompatibilní TAU/DOS, správná magnetická geometrie nebo použití jen implementovaného formalismu. To není totéž jako nedokončený hlavní solver. Některé tasky zároveň mají vadný vedlejší výstup nebo alternativní větev; ty jsou výslovně vyčleněny do 2B.

Zařazení do 2A není potvrzení numerické správnosti všech kombinací. „Správný stav“ zde znamená podmínky konkrétního fyzikálního workflow, nikoli libovolné vyladění parametrů, které jen skryje programátorskou chybu.

| Task / rodina | Podmínka použití nebo omezení | Praktický závěr |
|---|---|---|
| **MEPLOT** | Fotonový scan s více elektronovými energiemi přepisuje křivky. Přítomnost NEPHOT=1 sama přepne čtečku do range větve; SEFIN dostává počáteční ERYD. | Definice nezapisuje single-photon default NEPHOT a zakazuje současný dvojí scan. Význam SEFIN pro finální stav ještě vyžaduje fyzikální ověření. |
| **MECHECK** | NMEFORM se přepisuje, další test je compile-time vypnut a souhrnná smyčka má chybnou horní mez. | Dílčí ADA/NAB porovnání mají smysl; nelze slibovat všechny popsané kontroly ani GRV. |
| **SOCPAR** | Test ABS(... )<0 nikdy neobrátí znaménko. EXCME chybí převod Ry→eV, ačkoli osa říká eV. | SOC část a exchange část je nutné interpretovat odděleně. Exchange výstup má doložený nesoulad jednotek; opravit Fortran, ne input default. |
| **PSHIFT** | Fázové posuny existují, ale rezonanční štěpení může použít nenalezenou rezonanci; interpolace nehlídá skutečné překročení pi/2. | Fáze nejsou automaticky rozbité; odvozená rezonanční čísla nejsou spolehlivá bez opravy. NR neuniformní cutoffy a vysoké relativistické legendy mají další rizika. |
| **PLOTPRS** | Společný task obchází chybu samostatného PLOTRHO, ale spinová a orbitální křivka míří do stejného datasetu a limity panelu ignorují orbitální hustotu. | Potenciál, charge a SFN výstupy mají smysl; rozlišení magnetických hustot v grafu vyžaduje opravu writeru. |
| **STONER** | Kernel je pevně VWN; malý DOS se nahrazuje nefyzikálními jednotkovými příspěvky; některé normalizace dělí bez ochrany. | Výsledek není automaticky kernel použitého SCF funkcionálu. Potřebuje vhodný cutoff a nenulové normalizační veličiny. |
| **VBPES, VBUPS, VBXPS** | GEN implementace je společná; VBUPS nepřepíná na UV foton. DEFEP0/EWORK jsou jen diagnostické a custom geometrie může být přepsána. TAU a cache nejsou ověřené. | UV energii zadat výslovně. Tato rodina není totožná s KKRSPEC ARPES/AIPES. |
| **APS** | Čte starý DOS formát bez kontroly kapacity. Broadening rozšiřuje více polí a závisí na absolutní energii. | Bez broadeningu existuje použitelná cesta s kompatibilním DOS. GAMMA není běžná konstantní Lorentzova šířka; kladná GAMMA vyžaduje opravu či zvláštní ověření. |
| **AES** | Jen NQ=1; cache ME/MEi nemá kontrolu původu. NE z TAU může překročit alokaci. F slupky mohou překročit tabulku názvů. | Definice vylučuje f slupky a NEME<3; kompatibilní rovnoměrný rostoucí TAU mesh a čistá cache zůstávají nutné. |
| **FMAG** | Normalizace spinového/orbitálního form factoru dělí momentem při q=0 bez ochrany. Síť q je pevná. | Nulové momenty mohou dát NaN/Inf i v jinak korektním systému. Normalizované křivky nelze zpětně sčítat jako původní momenty. |
| **JALF** | Hlavní proudový soubor má účel, pomocná diagnostika ale směšuje typy a meshe; při NL<3 překračuje pole. | Definice chrání explicitní cutoff. Vektorový potenciál se standardně nepočítá, přestože kód obsahuje nedosaženou větev. |
| **TORQUE** | Potřebuje natočené momenty a příslušnou osu torque; běžný nenatočený potenciál může skončit STOP. Smíšené obsazení používá první typ. | Validace inputu bez potenciálu nemůže zaručit vhodnou magnetickou geometrii ani význam WGT_OCC. |
| **XAS** | Statická větev má účel, ale přijaté ME názvy neprokazují tři nezávislé gauges; non-dipole formalismus závisí na ASA/FULLPOT. Meze může nativně přepsat EF. | Sériové rozhraní (mpi=False), nastavitelný EMAX=4 Ry; MECHECK je samostatný task. Shoda TAU a účinné meze vyžadují kontrolu skutečných dat. |
| **PHAGEN** | Jen konvergovaný potenciál (SCFSTATUS=CONVERGED), ASA/VWN, NT=1, bez disorder. Původní total potential se exportuje s hustotou/Coulombovým potenciálem po dalším kroku; spin a mesh mají uvedené vady. | Pro jednoduchý NT1/mesh1 případ má export smysl. Výstupy jsou POTFIL_ex_sprkkr a POTFIL.only_coulomb_new; nejde o vlastní phase-shift solver. |
| **JXC** | FREL symetrická rotace je vyřazena podmínkou OR 1.EQ.1. Clusterová SP-SREL cesta běžně nemá připravené scratch soubory. | Jednoduchý periodický kolineární workflow má smysl; tensor na ekvivalentních místech a cluster nejsou obecně spolehlivé. |
| **DMI** | Dz se počítá odlišně od Dx/Dy. Omezení reprezentantů a párů nedoplňuje automaticky násobnosti úplné buňky. | Tři sloupce nejsou automaticky rovnocenné složky jednoho postupu. Mikromagnetické součty vyžadují kontrolu rámců, násobností a poloměru. |
| **COMPTON** | Nativní SPLITSS se čte ze dvou sekcí. EMD2D s nenulovým PNMAX škáluje podélný vektor dvakrát; sudé shifted meshe ztrácejí krajní body. | Definice zapisuje obě SPLITSS a vylučuje vadné EMD2D/shift kombinace. Základní kompatibilní workflow není tímto označen za rozbitý. |
| **POSANI** | Závisí na kompatibilních datech POSANIPREP; souborové TAUQQ má NQ/NT chybu. Core příspěvek je omezený a jádrová lifetime parametrizace má konkrétní pevný tvar. | Není samostatným univerzálním positronovým během. Před použitím připravených dat zkontrolovat jejich strukturu a původ. |
| **POSANIPREP** | TAUQQ writer/reader iteruje NT místo NQ: může vynechat site nebo překročit pole. Starý rho cache může přeskočit export wavefunctions. | Smysluplná příprava v kompatibilním jednoduchém případě; obecný NQ≠NT workflow je rozbitý. |
| **GILBERT** | GEFF dělí spinovým/celkovým momentem před ochranou. Tepelné mesh a normalizace potřebují konkrétní fyzikální data. | Základní damping má smysl pro vhodný magnetický systém; kompenzované či nulové momenty jsou vadné případy. |
| **CHILANDAU** | Nesprávná dokončovací podmínka tiskne průběžné integrály; RDTAUFIL nepřipraví potřebný energetický obsah. MPI má alokační/redukční chyby. | Serial výpočet posuzovat podle posledního úplného bloku. Reuse TAU a MPI nejsou funkční obecné cesty. |
| **SIGMA** | Fermi-surface a sea jsou odlišné větve; sea není automaticky zahrnuto. ImE a pevná jména sigmaXXXXX.dat ovlivňují použití. | Neprezentovat samotný povrchový default jako kompletní všechny příspěvky. Rozsah definice je auditovaná ASA/FREL/POINTS cesta. |
| **OPTABS** | NOMEGA je alokovaná kapacita, ale loop používá NEB. TEMPEL<=1 ponechá KBT nepřipravené. NEA je přepsáno a nenulové OMEGA1 porušuje auditované párování. | Definice vyžaduje TEMPEL>1, NOMEGA>=NEB a omezuje auditované okno. Task je OPTICS-ABS, nikoli obecný OPTICS. |
| **CHIXAS** | Potřebuje externí NEF/CHISPN/CHIVV údaje se správnými jednotkami, souvislým číslováním a rozměry potenciálu. | Není běh s univerzálními fyzikálními defaults. Testovací fixture nenahrazuje naměřené/vypočtené vstupní údaje. |
| **ARPES** | Staré TASK aliasy a nativní IMV_INI v Ry zapisující jinou damping proměnnou. CRYS_VECS má přítomnostní sémantiku. | Používat vlastní AIPES/BAND/SPLEED definice. Warning při validaci/uložení starého aliasu je DeprecationWarning a Python jej může filtrovat. |
| **SPLEED** | Sdílí KKRSPEC, ale má jinou větev a absolutní incidentní EV. Vystaven je konzervativní pevný geometrický workflow. | Není ARPES s jiným štítkem. Damping/jednotky a struktura surface potentialu musí být vhodné. |
| **AIPES** | IBLOCH=2 nastavuje lokální/incoherent výpočet všech atomů; nejde o obyčejnou numerickou úhlovou integraci ARPES. | Rozdíl proti ARPES je v modelu matrix elements a interferenci, nikoli jen ve výstupním úhlu. |
| **BAND** | Komplexní pásy používají -i LOG(EIG), cutoff je v Im(kz). Při neúspěšné EIGA/B chybí přerušení a může přijít LOG(0); popisky sloupců se liší podle geometrie. | Není náhrada EKREL pro obyčejné E(k). Je nutný vhodný eigenvalue cutoff, interpretace sloupců a oprava selhání solveru. |
| **CLPES** | Core window přepisuje zadané bounds. CPA a některé polarizace zastaví běh; ECOR v NE=1 cestě může pocházet z nesprávného posledního typu. | Sériová pevná geometrie a vhodný shell mají smysl; explicitní foton 1000 eV není zárukou překročení prahu. |
| **TUTORIAL: CHR-POT** | Používá společné PLOTPRS, včetně problémů magnetických grafů. | Je to graf načtených dat, nikoli nový SCF výpočet. |
| **TUTORIAL: POT-DECOMP, POT-XC** | ASA; NONMAG zůstává sticky mezi typy. POT-DECOMP rozkládá stav s původním Vxc, ukázané nové Vxc nemusí odpovídat původnímu. | POT-XC porovnává funkcionály na stejné hustotě, ne čtyři SCF řešení. PBE není podporovaný původní XC ve větvi POT-DECOMP. |
| **TUTORIAL: RWF-L, RWF-E** | Používá scalar reader, nikoli obecný FREL. Normalizace nemá ochranu před nulou; matching plot RWF-L čte nevyplněná data. | Radiální data mají diagnostický smysl ve scalar/ASA rozsahu, matching křivka RWF-L je rozbitá. |
| **TUTORIAL: CORE-ALL** | Jen první typ, nulové exchange field a vybrané výstupní komponenty. Bez core stavů mohou zůstat proměnné neinicializované; při jiném mesh indexu je chyba JRWS. | „ALL“ neznamená všechny typy a plný spinor. Vhodný první typ/mesh/core podmínky nejsou zaručeny inputem. |

**Příklady rozdílu:** TORQUE s nenatočeným potenciálem není důkaz nedokončeného torque solveru; CHIXAS bez správných externích susceptibilit nemá potřebné fyzikální vstupy. Naproti tomu chybné tensorové rotace JXC nebo neinicializované spektrální pole nejsou otázkou vhodného výchozího stavu.

### 2B. Víceméně nefungují pro uvedený účel: chyba, nedokončená hlavní větev nebo nespolehlivý výsledek

Tuto skupinu nepovažuji za běžně použitelnou jen po doplnění správného vstupu. Je nutná oprava implementace nebo ověření, že výpočet vůbec odpovídá deklarovanému účelu. U smíšených tasků se závěr vztahuje pouze k uvedené větvi; jejich funkční základ zůstává v 2A.

U XES a TDOBS je problém v doložení samostatné deklarované funkcionality. Audit zdrojů jej odhaluje, ale bez numerického referenčního porovnání neprokazuje, že každý jejich výstup je chybný. U ostatních níže uvedených programátorských vad je konkrétní příčina doložená.

| Task / konkrétní větev | Proč nestačí správný vstup | Praktický závěr |
|---|---|---|
| **PLOT2DPR / PLOT2DPOT / PLOT2DRHO: FULLPOT** | Bezpodmínečná experimentální PLY větev zapisuje obě skupiny a ukončí běh dřív než starší pravidelný grid writer. | Výsledkem nejsou slíbené selektivní pravidelné tabulky. NA/NB neřídí veškeré PLY vzorkování. |
| **PLOT2DPR / PLOT2DPOT: ASA** | ASA writer kreslí potenciál; celý PLOT2DPR nedodá deklarovanou hustotní část. | Potenciálové PLY jsou smysluplné, ale společný název není zárukou potenciálu i hustoty. |
| **XES** | RXES jde do společné absorption-rate větve; výstup je označen XAS a používá stejnou naming konvenci. | Nelze výsledek bez referenčního fyzikálního porovnání označit za ověřené emisní spektrum. Není to vypnutý placeholder. |
| **TDOBS** | Nativní cesta vede do generické DOS/observable smyčky; zvláštní časově závislá observable implementace nebyla doložena. | Název není zárukou samostatného fyzikálního tasku. Cache fallback i následné přepsání flags je nutné chápat podle GEN. |
| **JXC: obecný FREL tensor se symetricky ekvivalentními sousedy; clusterová SP-SREL cesta** | Rotace je vyřazena OR 1.EQ.1; clusterové scratch soubory nejsou v běžném toku připravené. | Jednoduchý periodický případ může fungovat, obecný tensorový/clusterový workflow potřebuje opravu. |
| **SOCPAR: exchange výstup** | Chybí převod Ry→eV a test znaménka ABS(... )<0 je nemožný. | Hodnoty s popiskem eV nelze přímo použít; funkční SOC část tím automaticky nezaniká. |
| **PSHIFT: rezonanční štěpení** | Součet používá i nenalezený druhý kanál a interpolace nehlídá ohraničený přechod pi/2. | Fázové posuny mají smysl, odvozené splitting hodnoty nejsou obecně spolehlivé. |
| **PLOTPRS / TUTORIAL CHR-POT: rozlišení spinové a orbitální křivky** | Obě křivky se zapisují do stejného datasetu; limity neberou v úvahu orbitální křivku. | Potenciál a charge grafy jsou použitelné; vadný magnetický graf je problém writeru. |
| **MECHECK: slíbená úplná sada kontrol a souhrn** | Další test je compile-time vypnut, NMEFORM se přepisuje a souhrnná smyčka se neprovede. | Dílčí ADA/NAB kontrola funguje; úplnou deklarovanou diagnostiku inputem nezapnete. |
| **MEPLOT: současný fotonový a elektronový scan** | Křivky se přepisují přes nesprávnou dimenzi. | Tato kombinace dává neúplný výstup; definice ji odmítá. Pochybnost o SEFIN je zvlášť neověřená interpretace. |
| **APS: broadening** | Rozšiřované sítě nejsou adekvátně kontrolovány proti kapacitám a šířka závisí na absolutní energetické nule. | Není ověřený běžný Lorentzův broadening; bez broadeningu může být základní APS použitelný. |
| **POSANIPREP / POSANI: obecná datová cesta NQ≠NT** | Writer/reader používá počet typů místo počtu míst. | Chybějící nebo mimo rozsah čtená data nevyřeší obecný validátor inputu. |
| **CHILANDAU: MPI a RDTAUFIL reuse** | Chyby alokace/redukce a nepřipravený energetický obsah. | Serial výpočet bez této reuse cesty je jiná, použitelnější větev. |
| **COMPTON: EMD2D s nenulovým PNMAX; některé sudé shifted meshe** | Dvojí škálování posouvá řez jinam; konstrukce ztrácí krajní body. | Výsledek neodpovídá deklarovanému řezu/meshi. Definice chrání omezením vstupu. |
| **BAND: selhání EIGA/B** | Chybí bezpečné přerušení a pokračuje se až k možnému LOG(0). | Úspěšná eigenvalue větev může fungovat; selhání solveru musí ošetřit native kód. |
| **CLPES: NE=1 s více typy** | ECOR může zůstat z posledního typu místo vybraného typu. | Pro tento případ může být energetická osa nesprávná; obecná pevná geometrie není sama problém. |
| **TUTORIAL: matching RWF-L** | Matching křivka používá nevyplněná data. | Radiální řešení a matching diagnostika nejsou totéž; graf potřebuje opravu. |
| **TUTORIAL: CORE-ALL mimo první vhodný typ/mesh/core případ** | Nesprávný JRWS index a nepřipravené proměnné při chybějících core stavech. | Nejde o obecně dokončené „všechny core stavy všech typů“. |
| **TUTORIAL: POT-DECOMP/POT-XC pro smíšené magnetické typy** | NONMAG se mezi typy neobnoví. | Rozlišení výstupu může být chybné; srovnání XC na fixní hustotě je jinak záměrná ukázka. |

Nechráněné nulové normalizace FMAG/GILBERT/POSANITAU a chyby cache nebo kapacit AES nejsou zde důkazem, že celý hlavní solver není hotový. Jsou to konkrétní vadné případy: při splnění podmínek existuje smysluplná cesta, při jejich porušení nelze výsledku věřit. Rozsah je uveden v 2A a v následujícím registru chyb.

Sekce 3 ponechává samostatně tasky s přímo doloženým rozbitím běžného hlavního workflow a přehled vadných případů. Kategorie 2B a 3 tedy nejsou dva soupeřící závěry: 2B zpřesňuje použitelnost bývalých „ALE“, sekce 3 uvádí konkrétní důkazy a potřebné opravy.

## 3. Tyto tasky nebo konkrétní větve jsou rozbité

### Celý zamýšlený běžný workflow

| Task | Doložený problém | Důkaz ve zdroji | Potřebná oprava |
|---|---|---|---|
| **FSCAT** | Chybný index harmonik s nadbytečným +L může překročit pole; fáze se počítá před ošetřením nulové amplitudy. | fscat.f:131,207–209; calc_chplm.f:155–162 | Opravit mapování LM, filtrovat nepřípustné/nulové CG složky a chránit fázi před 0/0. |
| **T1** | Podepsané kappa jsou použité jako kladné počty m_j; záporné kanály vypadnou a indexování celé báze se rozchází. | relt1.f:215–245 | Opravit kappa degenerace/indexy a ověřit relaxační rychlosti. PRINT/RUNELOOP workaround řeší jen další inicializační chyby. |
| **NRAES** | IAES se nuluje kratším rozsahem než alokované pole; konvoluce přičítá do neinicializovaných oblastí různých kanálů. | nraes.f:99,814 | Nulovat celé (2*NEMAX-1)*(NLMAX+1)*3 a přidat kontroly DOS metadat. |
| **BSDOSIJ** | J_ITJT se ukládá s energetickými indexy do párových dimenzí a čte jiným pořadím. Může překročit pole i vydat nesprávná data. | bsdosij.f:319–349,434–436 | Sjednotit dimenze/indexy, komponentové offsety, rotace a finální spektrální výstup. |
| **BONDING** | CLUBONDING čte scratch 18/19, které běžný TAU_DRIVE s KTAUIJ=0 nepřipraví. | tau_drive.f:70; tau_real_space.f:657–674,741–766; clubonding.f:78–108 | Propojit skutečný export TAUIJ se spotřebitelem; potom opravit centrum, rozměry a výstup. |
| **NUCSSCPL** | Odalkuje pole před první alokací; celé ME se nuluje znovu při každém typu a mesh je přepsán na 1. | nucsscpl.f:115–133,341,366–368 | Opravit životní cyklus polí, type-local inicializaci a mesh; NE=1 není oprava integračního tasku. |
| **XRS** | SIG(NEMAX+50,...) se nuluje pouze jako NEMAX; pozdější součet používá celý rozsah. | xrayspec.f:636–639 | Nulovat celé pole, opravit dodatečný mesh a zkontrolovat kapacity třetí integrace. |
| **BIS** | INITVAR jej rozpozná a GEN volá VBPES, ale rutina požaduje TASK(3:5)=PES, které BIS nesplňuje. | initvar.f:264; gen.f:1445; vbpes.f:174 | Implementovat odpovídající inverse-photoemission cestu. Alias BIS→VBPES by jen zamaskoval chybu a změnil význam. |
| **SIGMA-XAS** | Výpočtová větev existuje, ale standardní parser nenastaví interní TASK=SIGMA-XAS. Keyword SIGMA není prefixové rozpoznání. | rsp.f:941; sigme.f:499; initvar.f a SECTION_FIND_KEYWORD | Doplnit skutečnou vstupní cestu a ověřit XMCD workflow. Přidání SIGMA přepne na conductivity, nikoli tuto větev. |
| **PLOTRHO samostatně** | RHORSQ se alokuje pouze při potenciálové větvi, ale hustotní writer jej používá i bez ní. | fpplotprs.f:62,225,254,280 | Alokovat RHORSQ nezávisle v hustotní větvi. PLOTPRS s potenciálem tento konkrétní problém obchází. |
| **PLOTJALF / PLOTJORB** | FULLPOT skončí v experimentálním PLY writeru před čtením proudového souboru; ASA writer proud ignoruje. | fpplot2dpr.f:71,194; plot2dpr_fp_ply.f:926; plot2dpr_asa.f | Opravit dispatch/ukončení a actual current reader. Jsou to CONTROL selektory, nikoli samostatná nová implementace TASK. |

### Rozbité případy uvnitř jinak smysluplných tasků

| Rodina | Konkrétní vadná větev |
|---|---|
| PLOT2DRHO | ASA poskytuje potenciál, ne hustotu. Ve FULLPOT nefunguje deklarovaná selektivita a běžný grid writer je nedosažený. |
| POSANIPREP / POSANI datová cesta | NQ≠NT writer/reader může vynechat site nebo překročit pole. Platný input to nenapraví. |
| CHILANDAU | MPI a auditovaná RDTAUFIL reuse cesta. |
| GILBERT, FMAG, POSANITAU | Příslušné nulové jmenovatele; změnit ImE nebo počet bodů není obecná oprava normalizace. |
| PSHIFT | Rezonanční splitting při chybějícím druhém kanálu nebo bez ohraničeného přechodu pi/2. |
| SOCPAR | Exchange sloupec interpretovaný jako eV bez potřebného převodu. |
| COMPTON | EMD2D s nenulovým PNMAX a některé sudé shifted meshe; vystavená definice vadné kombinace odmítá. |
| TUTORIAL | Matching plot RWF-L; CORE-ALL bez core stavů / s neodpovídajícím mesh; sticky NONMAG v POT-XC/POT-DECOMP pro smíšené typy. |
| BAND | Selhání eigenvalue solveru bez bezpečného pokračování; možné LOG(0). |

## 4. Aktuální rozhraní ASE2SPRKKR

### Identita tasku a sdílená schémata

`InputParameters.for_task(name)` vybírá task a jeho metadata. JXC a DMI sdílejí schéma JXC; jejich identity deklarují `tasks/jxc.py` a `tasks/dmi.py`. TASK.JXC a TASK.DMI jsou nativní flags. DMI-only vstup má identitu DMI; současné zapnutí obou má identitu JXC a poskytuje oba výstupy. DMI vyžaduje magnetické FREL; samotné JXC dovoluje také SP-SREL. BSFEK a BSFKK obdobně sdílejí BSF, přičemž vlastní nativní režim určuje NE.

CLPES je samostatné schéma. Je dostupné přes `InputParameters.create("CLPES")` i `for_task("CLPES")` a je zahrnuto v `task_registry.names()` i v kanonickém výpisu schémat. Registr nabízí 55 taskových identit; sdílená schémata JXC/DMI a BSFEK/BSFKK zůstávají rozlišena od identity tasku.

BIS a SIGMA-XAS nemají předstíraný Python solver: jejich běžná nativní vstupní cesta je rozbitá. VBUPS/VBXPS sdílejí nativní VBPES; PLOTJALF/PLOTJORB jsou CONTROL selektory s vadným dispatch.

### Spektroskopie a čtení výstupů

ARPES, AIPES, SPLEED, BAND a CLPES používají KKRSPEC, ale liší se IBLOCH větví, modelem a významem energií/geometrie. VBPES používá GEN/VBPES. AIPES je lokální/incoherent model, nikoli obecná numerická kvadratura ARPES úhlů. Tyto postupy nejsou zaměnitelné.

ARPES a XAS deklarují `mpi=False`; XAS má EMAX=4 Ry a MECHECK je samostatný task. JXC používá NE=30 a ImE=0.01 Ry. CRYS_VECS=False se nevypisuje a ponechává USE_CRY_PRIM_VECS=true. Ve 3D konstrukci povrchu přítomnost CRYS_VECS nastaví USE_CRY_PRIM_VECS=false (spec_rslab.f:180–181). Stejnou definici sdílejí ARPES, AIPES, BAND, SPLEED a CLPES; singulární CRYS_VEC nativní čtečka nečte. Společný KKRSPEC základ se kopíruje z ARPES; není oddělen do nezávislého schématu.

ARPES a CLPES deklarují reader ARPES, AIPES/BAND/SPLEED reader SPEC a JXC/DMI reader JXC. U řady experimentálních diagnostik zůstává obecný reader logu; Grace/PLY či rate soubor tím není automaticky převeden na strukturovaný fyzikální výsledek.

### TUTORIAL

Definice obsahuje všech 14 voleb: YLM, JLX, JLX-ASYMPT, CHR-POT, POT-DECOMP, POT-XC, RWF-L, RWF-E, CORE-ALL, DOS-FEG, DOS-SS, DOS-CLU, DOS-BULK, GEN. Defaulty a validace závisejí na větvi. RWF používá scalar reader, DOS-CLU potřebuje skutečnou clusterovou geometrii a POT-XC porovnává funkcionály na stejné hustotě. Podpora vstupu neopravuje uvedené nativní chyby.

## 5. Defaults: vypsaná volba, nepřítomnost a skutečný fallback

**Pravidlo tohoto auditu:** default, který se nevypisuje a představuje známou hodnotu, musí odpovídat účinnému nativnímu fallbacku. `None` může označovat delegaci na EF, potenciál nebo metadata; není tvrzením, že nativní hodnota je Python None. Vypsaná hodnota může být jiná, ale rozdíl musí mít správně popsaný význam. Parametr, který solver později přepíše, není účinným uživatelským ovládáním jen proto, že se zapíše.

### Důležité společné nálezy

| Případ | Účinný význam |
|---|---|
| EMAX sentinel | INIT_MOD_ENERGY převádí hodnotu menší než -99 na EF. **EMAX=-1 není EF**, nýbrž absolutních -1 Ry. Vynechané EMAX po příslušném fallbacku může znamenat skutečnou EF. |
| Relativní vs. absolutní energie | Obecné EMINEV/EMAXEV jsou eV offsety vůči EF. EKREL TASK bounds jsou absolutní Ry. SPLEED incidentní EV je absolutní incidentní energie; nepřenášet sem ARPES význam. |
| Presence-sensitive options | NSHLCLU=0 přítomné není totéž jako vynechané NSHLCLU. NEPHOT=1 přítomné není totéž jako single-photon fallback bez keywordu. |
| GRID a NE | GRID=2 s NE=1 může dělit nulou. GRID=3 s NE=1 má použitelný první bod, ale nepřipravené integrační váhy; je vhodný jen tam, kde se neintegruje. GRID=0 má vlastní jednoenergetický význam. |
| ENERGY vs. TASK | COMPTON potřebuje SPLITSS v obou sekcích. EKREL hledací NE/EMIN/EMAX jsou v TASK. OPTABS skutečnou vlastní síť řídí TASK parametry. |
| Potential context | NL, IT rozsah, existence slupky, NQ/NT, KMROT, momenty a skutečný FULLPOT formát nejsou obecně zjistitelné z InputParameters bez POTFIL. |

### Rozdíly, které se vypisují a nejsou samy o sobě chybou

| Task | ASE2SPRKKR vs. nativní chování | Posouzení nativního defaultu |
|---|---|---|
| SCF | NE=32 proti nativním 30; některé vlastní mixing volby. Omitted MIXOP zůstane nativně 0.2, i když se později čte jiné MIX. | 30 může být levnější, nikoli automaticky stejně konvergované. MIXOP nelze tiše popsat jako aktuální MIX. |
| DOS | Explicitní GRID=3, NE=300, EMIN=-0.2, EMAX=1 Ry. Vynechaná nativní horní mez by byla EF. | EMAX=1 je smysluplný scan nezaplněných stavů; EF by omezila výstup na obsazené. |
| BSF EK | Explicitní EMAX=-1 Ry zůstalo zachováno. | Obvykle lepší je EF nebo vlastní rostoucí interval. Nesmí se označit -1 jako sentinel EF; při EMIN=-0.2 jde o sestupný scan. |
| BSF | NK=300 proti nativnímu 800; KK EMIN=0.7 Ry. | Hustotu je třeba konvergovat; KK default není zaručeným Fermiho řezem. |
| XAS | Explicitní EMIN=-0.2 Ry proti nativnímu EF; EMAX=4 Ry je nastavitelný člen ENERGY. | EF je přirozenější pro standardní absorpci, ale native XAS může EMIN stejně přepsat. Zkontrolovat EMF a účinné meze. |
| PHAGEN | NE=32 proti 30, ImE=0 proti 0.01; NITER/MIX jsou při exportu přepsány na 1/1. | Nepřejímat do popisu počet iterací ze zapsaného společného defaultu. Konvergence kontury nebyla porovnána. |
| FMAG/JALF/JORB | Zachované explicitní ImE=0, i když obecný nativní EIMAG je 0.01. | U GRID=5 rozhoduje celá komplexní kontura; samotná nula nedokazuje nepoužitelnost ani lepší volbu. |
| JXC / DMI / CHILANDAU / CHI | GRID=5, NE=30, ImE=0.01 převzaté podle nativního integračního workflow. CLURAD=2.2 je výslovná praktická volba. | Native počet bodů je vhodný výchozí základ; radius není zárukou konvergence. |
| SIGMA | ImE=0.01 proti surface nativní nule. | Smearing podstatně ovlivňuje transport; žádná z hodnot není univerzálně lepší bez convergence study. |
| GILBERT | GRID=0/NE=1 pro EF, bez neúčinných energetických mezí; NL=4 se explicitně vypisuje. | Native jednoenergetická cesta je zde lepší než neodpovídající lineární contour. |
| OPTABS | TEMPEL=300 K/NKT=6 proti reader hodnotám 1e6/99; NEB=30/NOMEGA=100, OMEGA1=0/OMEGA2=8 eV. | 300 K je praktičtější fyzikální start. Native extrémní teplota není důvod pro automatické převzetí; validovat skutečné teplotní použití. |
| ARPES | NE=300, EV=-8..5, EWORK=4.2 eV, damping 0.05/2 eV, NL=4 a vrstvy 50/57; native má jiné počty/damping. | Jde o zapsané volby, nikoli omitted bug. Nelze bez konvergence prohlásit native 200 bodů a jiné vrstvy za lepší. |
| AIPES / SPLEED / BAND | AIPES valenční EMAXEV=0; SPLEED vlastní 11..60 eV/200 bodů a 0.2/0.2 eV; BAND explicitní zero damping a EIG_IME. | Správné task-specific rozdíly; nesjednocovat slepě podle ARPES. |
| KKRSPEC N_LAYDBL | Explicitní dvě složky obcházejí nativní překlep dvojí inicializace LANZ1 místo LANZ2. | Nepřítomný „default“ není bezpečně inicializovaný; explicitní hodnoty jsou lepší do opravy native. |
| T1 | PRINT=1 a RUNELOOP=true jsou explicitní workaroundy vadné inicializace. | Native defaults zde nejsou lepší. Ani workaround neopravuje kappa sum. |
| Diagnostické ploty | GRID/NE často jen zajišťují společnou startup alokaci; poslední selektory používají GRID=0/NE=1/ImE=0. | Více bodů nezlepšuje radiální či prostorový graf. Nepřipisovat startup energii fyzikální spektrální význam. |
| APS/NRAES/AES/VBPES | Explicitní NE=300 je kapacita nebo předpokládaný mesh; skutečné NE se bere z DOS/TAU. | Lepší budoucí řešení je odvození a validace ze skutečné hlavičky souboru včetně reserve broadeningu. |
| WFPLOT / MEPLOT / MECHECK | WFPLOT má GRID=3/NE=1, EMAX=EMIN a explicitní NL=3. MEPLOT má NE=300 a ponechává horní mez i NL kontextu. MECHECK má NE=1. | WFPLOT použije první bod; MEPLOT skutečně prochází energie. Tyto diagnostiky nemají jednotný počet bodů ani stejné fallbacky. |

Konečná TUTORIAL definice zapisuje rozdílné energetické defaults: matematické větve GRID0/NE1/ImE0; RWF GRID3/NE10/ImE0 s absolutním 1e-7..1 Ry; DOS/GEN GRID3/NE300/ImE0.01 s -0.2..1 Ry. Jde o explicitní praktické volby. STANDARD a příslušný BZINT se rovněž zapisují podle větve.

### Jednotky PLOT2D a názvy výstupních souborů

AVEC/BVEC/DVEC jsou kartézské vektory v jednotkách ALAT, nikoli frakční souřadnice báze nebo přímo a.u. Důkazem je RWS/ALAT a převod vzdálenosti násobením ALAT před radiální interpolací (`plot2dpr_asa.f:138–139,173`; `plot2dpr_fp_ply.f:259`). Komentář ASA na řádku 102 o a.u. tomuto datovému toku neodpovídá.

KKRGEN uloží původní DATASET do DATSET0 a teprve k DATSET připojí neprázdné ADSI (`kkrgen.f:238–243`). Veřejná šablona **{DATASET} znamená DATSET0**, **{DATASET_ADSI} znamená nativní DATSET**. Prázdné ADSI nepřidává podtržítko. JALF/JORB a uložené potenciálové grafy používají první prefix; DOS/TAU a řada spekter druhý. Pevná jména se tím neizolují. PHAGEN je odvozen od POTFIL a zapisuje POTFIL_ex_sprkkr a POTFIL.only_coulomb_new.

TAUDIR má implicitně prefix rozšířený o ADSI. Pokud TAUPATH není `.`, native reader jej připojí i k TAUDIR začínajícímu `/` (`kkrgen.f:266–273`). Příprava inputu proto musí posoudit oba parametry společně; nejde o obecnou Python Path sémantiku.

## 6. MPI a výstupy

`mpi=True` v definici znamená deklaraci podpory, nikoli provedený MPI regression run. `mpi=False` může být důsledek doloženého native guardu nebo konzervativní omezení neověřeného workflow; tyto důvody se nemají směšovat.

| Skupina | Výsledek source auditu |
|---|---|
| SCF, standardní DOS, BSF, EKREL, JXC, DMI, COMPTON, POSANI, FMAG | Existuje zdrojové opodstatnění MPI pro auditovanou cestu; numerical serial/MPI shoda nebyla měřena. FMAG dokončuje DENSITY před obecným GEN guardem. |
| XAS/XES/XRS, STONER, VBPES, APS/NRAES/AES, JALF/JORB, diagnostické GEN ploty | Obecná energetická smyčka není záruka dokončeného postprocessingu. Příslušné native guardy nebo chybějící redukce odůvodňují serial rozhraní. |
| CHILANDAU, NUCSSCPL, PHAGEN | Konkrétní zdrojové chyby v MPI/alokacích/bariérách. |
| GILBERT | Aktuální definice deklaruje MPI=true. Nulové momenty a normalizace GEFF mají uvedené vady; numerická shoda serial/MPI není ověřena. |
| SIGMA, POSANIPREP, POSANITAU, nové KKRSPEC wrappers | Serial volba je v části případů konzervativní hranice auditu, nikoli důkaz, že ve zdroji neexistuje žádné MPI. BAND vlastní MPI implementaci skutečně má. |

U mnoha experimentálních definic není specializovaný Python parser datových souborů. Specializované KKRSPEC tasky deklarují SPEC reader, původní ARPES reader ARPES a JXC vlastní reader; nelze absenci parseru zobecnit. Grace/PLY nebo rate soubor není automaticky strukturovaným fyzikálním výsledkem jen díky společnému čtení logu.

Pevné názvy `ME`, `MEi`, `sigmaXXXXX.dat`, `curdns1..3`, `fort.*`, některé PLY a `arpes.dat` nejsou vždy izolované pomocí CONTROL.DATASET. U AES/PHAGEN/VBPES/POSANIPREP navíc existující soubory mohou změnit cache cestu. Numerické hlavičky nejsou vždy prosté ENERGY.NE: AES/NRAES mají 2*NE-1 spektrálních bodů, XRS přidává další mesh a JORB/JALF zapisují radiální/úhlová data.

## 7. Ověření a podrobné audity

Společně integrované definice byly ověřeny 9. 11. 2026 na Linuxu s Pythonem 3.14.4:

```sh
python -m pytest src/ase2sprkkr/input_parameters/test src/ase2sprkkr/result_files/test -q
```

Výsledek: **1249 passed, 1 skipped, 81 warnings, 354.11 s**. Testy pokrývají výběr tasků a schémat, validaci, serializaci/načtení, copy/pickle a souborové manifesty. Běh zahrnuje 1135 testů vstupních parametrů; zbývající testy ověřují výsledkové soubory. Deset parametrizovaných regresních případů kontroluje CRYS_VECS a výpis pěti KKRSPEC tasků; NeXus test ověřuje zachování tohoto klíče při exportu.

Samostatný cílený běh zahrnuje také nativní export PHAGEN:

```sh
python -m pytest src/ase2sprkkr/input_parameters/test/test_plot2dpr.py src/ase2sprkkr/input_parameters/test/test_plot_selector_file_manifests.py src/ase2sprkkr/input_parameters/test/test_phagen.py src/ase2sprkkr/sprkkr/test/test_calculator.py::TestCalculator::test_phagen -q
```

Výsledek: **32 passed, 5 warnings, 5.91 s**. Nativní PHAGEN používá konvergovaný Au_single_type.pot a ověřuje oba exportované soubory. Existující místní binárky pocházejí z `SPRKKR/9_7`; nebyl proveden nový překlad veřejného archivu ani porovnání s externím PHAGEN/MsSpec.

Celá sada ASE2SPRKKR, macOS a numerické serial/MPI reference všech tasků nebyly v tomto ověření spuštěny. Platný input ani regulérní konec programu necertifikují spektra, konvergenci nebo původ TAU/DOS/cache.

Následují aktuální dílčí audity. Uvedené moduly existují v `src/ase2sprkkr/input_parameters/test/`; jejich přítomnost není nativním numerickým testem.

| Rodina | Podrobný audit | Aktuální regresní moduly |
|---|---|---|
| AES | [AES](AES-audit.md) | `test_aes.py` |
| AIPES | [AIPES](AIPES-audit.md) | `test_aipes.py` |
| APS | [APS](APS-audit.md) | `test_aps.py` |
| ARPES | [ARPES](ARPES-audit.md) | `test_arpes_audit.py`, `test_arpes_energy.py` |
| BAND | [BAND](BAND-audit.md) | `test_band.py` |
| BIS | [BIS](BIS-audit.md) | zdrojový rozbor |
| BONDING | [BONDING](BONDING-audit.md) | `test_bonding.py` |
| BSDOSIJ | [BSDOSIJ](BSDOSIJ-audit.md) | `test_bsdosij.py` |
| BSF | [BSF](BSF-audit.md) | `test_bsf_audit.py`, `test_bsf_file_manifests.py` |
| CHI | [CHI](CHI-audit.md) | `test_chi.py` |
| CHILANDAU | [CHILANDAU](CHILANDAU-audit.md) | `test_chilandau.py` |
| CHIXAS | [CHIXAS](CHIXAS-audit.md) | `test_chixas.py` |
| CLPES | [CLPES](CLPES-audit.md) | `test_clpes.py` |
| CLXPS | [CLXPS](CLXPS-audit.md) | `test_clxps.py` |
| COMPTON | [COMPTON](COMPTON-audit.md) | `test_compton.py` |
| DMI | [DMI](DMI-audit.md) | `test_dmi.py`, `test_exchange_file_manifests.py` |
| DOS | [DOS](DOS-audit.md) | `test_dos.py` |
| EKREL | [EKREL](EKREL-audit.md) | `test_ekrel.py` |
| FMAG | [FMAG](FMAG-audit.md) | `test_fmag.py` |
| FSCAT | [FSCAT](FSCAT-audit.md) | `test_fscat.py` |
| GILBERT | [GILBERT](GILBERT-audit.md) | `test_gilbert.py` |
| JALF | [JALF](JALF-audit.md) | `test_jalf.py` |
| JORB | [JORB](JORB-audit.md) | `test_jorb.py` |
| JXC | [JXC](JXC-audit.md) | `test_jxc.py`, `test_jxc_definition.py`, `test_exchange_file_manifests.py` |
| MECHECK | [MECHECK](MECHECK-audit.md) | `test_mecheck.py` |
| MEPLOT | [MEPLOT](MEPLOT-audit.md) | `test_meplot.py` |
| NRAES | [NRAES](NRAES-audit.md) | `test_nraes.py` |
| NUCSSCPL | [NUCSSCPL](NUCSSCPL-audit.md) | `test_scattering_file_manifests.py`, `test_dataset_filenames.py` |
| OPTABS | [OPTABS](OPTABS-audit.md) | `test_optabs.py` |
| PHAGEN | [PHAGEN](PHAGEN-audit.md) | `test_phagen.py` |
| PLOT-selectors | [PLOT-selectors](PLOT-selectors-audit.md) | `test_plotprs.py`, `test_plot_selector_file_manifests.py` |
| PLOT2D-selectors | [PLOT2D-selectors](PLOT2D-selectors-audit.md) | `test_plot2dpr.py`, `test_plot_selector_file_manifests.py` |
| PLOT2DPR | [PLOT2DPR](PLOT2DPR-audit.md) | `test_plot2dpr.py` |
| PLOTJALF-PLOTJORB | [PLOTJALF-PLOTJORB](PLOTJALF-PLOTJORB-audit.md) | zdrojový rozbor |
| PLOTPRS | [PLOTPRS](PLOTPRS-audit.md) | `test_plotprs.py` |
| POSANI | [POSANI](POSANI-audit.md) | `test_posani.py` |
| POSANIPREP | [POSANIPREP](POSANIPREP-audit.md) | `test_posaniprep.py` |
| POSANITAU | [POSANITAU](POSANITAU-audit.md) | `test_posanitau.py` |
| PSHIFT | [PSHIFT](PSHIFT-audit.md) | `test_pshift.py` |
| SCF | [SCF](SCF-audit.md) | `test_scf_audit.py` |
| SIGMA-XAS | [SIGMA-XAS](SIGMA-XAS-audit.md) | zdrojový rozbor |
| SIGMA | [SIGMA](SIGMA-audit.md) | `test_sigma.py` |
| SOCPAR | [SOCPAR](SOCPAR-audit.md) | `test_socpar.py` |
| SPLEED | [SPLEED](SPLEED-audit.md) | `test_spleed.py` |
| STONER | [STONER](STONER-audit.md) | `test_stoner.py` |
| T1 | [T1](T1-audit.md) | `test_t1.py` |
| TDOBS | [TDOBS](TDOBS-audit.md) | `test_tdobs.py` |
| TORQUE | [TORQUE](TORQUE-audit.md) | `test_torque.py` |
| TUTORIAL | [TUTORIAL](TUTORIAL-audit.md) | `test_tutorial.py` |
| VBPES | [VBPES](VBPES-audit.md) | `test_vbpes.py` |
| VBUPS-VBXPS | [VBUPS-VBXPS](VBUPS-VBXPS-audit.md) | `test_vbpes.py` |
| WFPLOT | [WFPLOT](WFPLOT-audit.md) | `test_wfplot.py` |
| XAS | [XAS](XAS-audit.md) | `test_xas.py`, `test_xas_definition.py` |
| XES | [XES](XES-audit.md) | `test_xes.py` |
| XRS | [XRS](XRS-audit.md) | `test_xrs.py` |

## 8. Priorita dalších oprav

1. **SPR-KKR před fyzikálním použitím:** opravit pole/indexy FSCAT, NRAES, BSDOSIJ, NUCSSCPL a XRS; kappa sum T1; datový tok BONDING; writer PLOTRHO. To jsou doložené vady, které definice Pythonu nevyřeší.
2. **Dosažitelnost a význam:** vyřešit BIS a SIGMA-XAS; oddělit selective PLOT2D/current writer od experimentálního PLY ukončení; fyzikálně porovnat XES a upřesnit AIPES model.
3. **ASE2SPRKKR:** oddělit sdílený KKRSPEC základ a rozlišit serial metadata od nativního zákazu. Posoudit BSF default EMAX=-1 a potvrdit změnu energetického okna jako změnu veřejného chování.
4. **Metadata potenciálu/cache:** validovat TAU/DOS kapacity, skutečné NQ/NT, meshe, slupky a magnetickou geometrii tam, kde je soubor dostupný. Fixní NE ani kladné IT nenahrazují tuto kontrolu.
5. **Nativní regresní ověření:** společná sada input_parameters již prošla (1135 testů vstupních parametrů); dále dodat reprezentativní numerické a serial/MPI reference podporovaných fyzikálních cest. Zvlášť kontrolovat nativní zdroje a sestavu, nikoli jen existenci Python definice.

## 9. Tyto tasky jsou vypnuté — společný seznam na konci

Pouze seznam větví z dodané revize; nebyly pro ně vytvářeny další předstírané definice. U veřejné blokace jde výslovně o dodaný PUBLIC_VERSION. U NEGF je vypnutá stará cesta přes KKRRSP, nikoli tímto závěrem celý samostatný program KKRNEGF.

| Task / větev | Důvod | Místo |
|---|---|---|
| **FORCETEN** | UNDER_CONSTRUCTION před vlastním výpočtem | gen.f:1055–1057 |
| **XMO** | Veřejná verze: not available / UNDER_CONSTRUCTION | xrayspec.f:283–287 |
| **OPTICS** | Veřejná verze: not yet available | rsp.f:887–888 |
| **MAGNET** | LINRESP_MAGNET je stub se STOP | linresp_magnet.f:1–4 |
| **CHIDYN** | LINRESP_SUSC_DYN je stub se STOP | linresp_susc_dyn.f:1–4 |
| **PHONONS** | Veřejná verze explicitně zastaví task | check_input.f:34–36; rsp.f:962–963 |
| **PUMP** | LINRESP_PUMP je stub se STOP | linresp_pump.f:1–4 |
| **OPM** | OPM scheme not available | check_input.f:136 |
| **SIGMA-DNS** | SIG_DNS volá UNDER_CONSTRUCTION | sig_dns.f:13 |
| **NEGF přes KKRRSP** | Od verze 8.3 odkazuje na samostatný KKRNEGF a zastaví tuto cestu | rsp.f:905–907 |
