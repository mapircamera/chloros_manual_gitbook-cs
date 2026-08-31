# Systémy s více kamerami

**Systém**LATTICE tvoří dvě nebo více kamer LATTICE propojených do jedné synchronizované jednotky. Jedna kamera je**hlavní**: vysílá hardwarový spouštěcí impuls GPIO na sdílenou synchronizační linku (výchozí**Line2**), takže všechny kamery pořizují snímek ve stejném okamžiku. Chloros přidává časovou synchronizaci PTP, živý náhled (mozaika jednotlivých kamer nebo jediný zarovnaný vícepásmový kompozit) a synchronizované snímání — každý snímací cyklus vytvoří jednu**skupinu snímků**, ve které mají všechny kamery stejný časový údaj a ID snímku (na výstupu snímání uváděno jako `fid:N`).

Pomocí polí vytvářejí monochromatické (M3M) kamery vegetační indexy – jedna kamera přispívá jedním pásmem a pole je zarovná do vícepásmového souboru. Viz [Monochromatické kamery a vegetační indexy](mono-indices.md).

Existují tři rovnocenné způsoby připojení pole a všechny z nich používají stejný pracovní postup „smart-prep“:

| Rozhraní | Vstupní bod |
| --- | --- |
| Grafické uživatelské rozhraní | Karta Kamery → **Připojit pole** (modré tlačítko) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (první sériové číslo = hlavní) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (první sériové číslo = master) |

Smart-prep provádí v tomto pořadí: test síťové dostupnosti (ICMP DF ping + GVSP test), výběr synchronizační vrstvy, automatické zmenšení velikosti rámce tak, aby se vešel do linky, aktivaci PTP, automatický výběr formátu pixelů pro každou kameru, automatické nastavení počáteční expozice na základě uloženého stavu každé kamery a konfiguraci spouštění přes GPIO na lince 2.

{% hint style="info" %}
Aby cokoli z toho fungovalo, musí být kamery na spojení dostupné — viz [Připojení kamer](connecting.md) ohledně vyhledávání, adresování a stažení kalibračních dat při prvním připojení. U sestav s více kamerami jsou nastavení přijímacího kruhu hostitelské síťové karty stejně důležité jako rychlost připojení; kompletní tabulka příznaků a řešení najdete v [CLI Referenci § Nastavení a ladění hostitelské síťové karty](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Dialogové okno „Připojit pole“

Karta Kamery → **Připojit pole**otevře tříkrokový průvodce:**Výběr → Režim zobrazení → Nastavení**.

### Krok 1 — Výběr hlavní a podřízených

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered. Table showing Camera / Serial / IP / Master radio / Slave checkbox columns, with the green "GPIO master detected — selections pre-populated" probe banner visible above the table. -->

kamer Dialogové okno ihned po otevření prohledá síť („Prohledávání sítě...“) a poté otestuje zapojení spouštěče GPIO („Testování zapojení GPIO...“). K sestavení pole potřebujete alespoň **2 kamery**.

Test zapojení předvyplní výběr rolí, pokud je to možné, a zobrazí jeden ze tří nápisů:

| Nápis | Význam |
| --- | --- |
| „Zjištěn hlavní modul GPIO — výběry předvyplněny“ (zelená) | Testovací procedura zjistila topologii spouštěcího signálu; zaškrtávací políčka pro hlavní a podřízené kamery jsou již zaškrtnuta. |
| „Žádný master nezjištěn – zkontrolujte kabel GPIO“ (oranžová) | Žádná kamera nezaznamenala spouštěcí impuls; zkontrolujte synchronizační kabeláž. Role můžete stále vybrat ručně. |
| „Žádný synchronizační kabel: {seriální čísla}“ (oranžová) | U uvedených kamer není připojen žádný synchronizační kabel. |

Tabulka kamer obsahuje sloupce **Kamera / Sériové číslo / IP / Master (rádiové tlačítko) / Slave (zaškrtávací políčko)**:

* Vyberte přesně **jednoho mastera**a**jednoho nebo více slaveů**. Opětovným kliknutím na rádiové tlačítko aktuálního mastera se jeho výběr zruší.
* Kameru označenou **„Žádný synchronizační kabel“** nelze nikdy vybrat jako podřízenou – podřízená kamera bez spouštěcího zapojení by čekala na synchronizační lince donekonečna a poskytovala by nefunkční obraz. Tuto kameru místo toho připojte jako samostatnou kameru.
* Kamery, které jsou již připojeny samostatně, *nejsou* deaktivovány: připojení do pole uvolní samostatnou relaci a kameru znovu otevře v rámci pole.

**Další: Režim zobrazení →**se aktivuje, jakmile je vybrán hlavní přístroj a alespoň jeden podřízený přístroj.**Znovu prohledat** znovu spustí vyhledávání a testování zapojení.

{% hint style="warning" %}
**Zrušit** je deaktivováno, dokud probíhá skenování nebo test propojení – zrušení uprostřed testu může způsobit pád kamery SDK s firmwarem LATTICE. Počkejte, až se dokončí otáčení ikony.
{% endhint %}

### Krok 2 — Režim

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

zobrazení | Režim | Co získáte |
| --- | --- |
| **Samostatné kamery** | Jedna živá dlaždice pro každou kameru, všechny se spouštějí společně, takže snímky zůstávají synchronizované. Každá kamera si zachovává vlastní barvu a nastavení. |
| **Kombinované kamery** *(výchozí)* | Jedna dlaždice zobrazující zarovnaný vícepásmový kompozit NDVI/index. Kamery sdílejí barvu pole. |

Režim zobrazení mění pouze prezentaci živého náhledu — chování při snímání je v obou případech stejné.

### Krok 3 — Nastavení pole a předpokládaný výsledek

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Pin resolution / Trigger Rate controls, right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

Při vstupu do této scény si Chloros vyžádá od backendu **doporučení**a automaticky aplikuje kombinaci oblasti zájmu (ROI) a binningu, která odpovídá přijímacímu prstenci vašeho NIC (upřednostňuje binning před ořezáním ROI, protože binning zachovává celé zorné pole). Každá změna, kterou provedete, spustí analýzu v reálném čase a aktualizuje panel**Očekávaný výsledek** na pravé straně.

Levý sloupec — nastavení:

| Ovládací prvek | Možnosti | Výchozí | Poznámky |
| --- | --- | --- | --- |
| **ROI (zorné pole)** | Plné (2048×1536) / Poloviční (1024×768) / Čtvrtinové (512×384) | Plné | Oříznutí snímače: Poloviční/čtvrtinové oříznutí na menší oblast při nativní velikosti pixelů. |
| **Binning** | 1× / 2× (součet 2×2) / 4× (součet 4×4) | 1× | Hardwarové binning: 2×2 = plné zorné pole při čtvrtinových nákladech na výstup; 4×4 = plné zorné pole při 1/16. Skryto, pokud kamery nepodporují binning. |
| **Obraz na straně sběrnice** (čtení) | — | — | Šířka × výška po binningu skutečně odeslaná po sběrnici, zaokrouhlená na násobky 16 (minimálně 64). |
| **Rozlišení pinů**| zaškrtávací políčko | vypnuto | Chloros obvykle automaticky zvýší binning při připojení, pokud předpokládaná rychlost klesne pod**1,5 fps**. Zafixování zachová vámi zvolenou velikost snímku a akceptuje nižší rychlost — a změní konfiguraci s nadměrným zatížením na tvrdé odmítnutí připojení namísto automatického snížení rychlosti. |
| **Spouštěcí frekvence** | 0,5–60 fps, krok 0,1 | prázdné = auto | Spouštěcí frekvence masteru. Nechte pole prázdné, aby ji Chloros odvodil sám. |
| **Kapacita linky**| 20–2000 MB/s, krok 10 | prázdné = auto | Kolik hostitel skutečně zvládne přijmout, v MB/s —**jediné číslo, na kterém závisí celé přidělení pole.** Automaticky detekováno ze síťového adaptéru. Snižte tuto hodnotu, pokud pole hlásí poškozené rámce: detekovaná hodnota nadhodnocuje výkon USB adaptérů a sdílených přepínačů. Změna této hodnoty spustí nové živé propočty. |

Pravý sloupec — **Předpokládaný výsledek**:

* **Úroveň synchronizace** — „Souběžný záznam“ (zelená), „Souběžný záznam (FTD – postupné vysílání)“ (zelená), „Postupný záznam (100 ms zpoždění)“ (oranžová) nebo „Konfigurace příliš velká“ (červená).
* ****Projekce fps** — zobrazena jako rozsah („tmavá → světlá“), protože rychlost synchronizovaného pole je omezena expozicí nejpomalejší kamery.
* **Řádek NIC** — rychlost připojení a trvalá propustnost („NIC {mbps} Mbps · trvalá {N} MB/s“).
* **Kontrola simulovaného vysílání burstů** — dokáže přijímací kruh síťové karty hostitele přijmout jeden souběžný burst ze všech kamer („Simulované vysílání burstů: X MB · využitelný kruh síťové karty: Y MB ✓/✗“).
* **Kontrola kapacity linky** — souhrnná poptávka v ustáleném stavu vs. strop linky bez kolizí („Kapacita linky: {demand} MB/s požadovaných {n} kamerami · strop {ceiling} MB/s ✓/✗ překročení kapacity“).
* **„Maximální počet kamer na tomto kanálu: {n} — nastaveno minimální šířkou pásma na kameru, takže seskupování tento počet nezvýší.“** — zobrazí se, když se blížíte k (nebo překračujete) maximálnímu počtu kamer.
* **„PŘI TĚCHTO NASTAVENÍCH DOJDE K VYNECHÁVÁNÍ SNÍMKŮ.“**— červené varování s důvodem ze strany backendu, plus seznam překážek a modré**návrhy řešení** („Aby se tato sestava vešla do sítě“ / „Aby se odemklo souběžné snímání“).

Tlačítko **Použít a připojit** je zablokováno, dokud neexistuje prognóza, a jeho popisek uvádí důvod odmítnutí:

| Popisek tlačítka | Význam | Co skutečně pomůže |
| --- | --- | --- |
| „Analýza probíhá...“ | Analýza stále probíhá. | Počkejte. |
| **„Příliš mnoho kamer pro tuto síť“**| Pole přetěžuje linku (selhala kontrola agregace). | Méně kamer, jumbo rámce v celém řetězci nebo rychlejší síťová karta.**Menší ROI nepomůže** — viz níže. |
| **„Zmenšete ROI pro povolení“** | Při těchto nastaveních by docházelo ke ztrátám rámců (selhala kontrola burst/ring). | Zmenšete ROI, zvyšte binning nebo opravte přijímací kruh síťové karty. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->

Během připojování se může objevit zelený **panel stahování kalibrace** s ukazatelem průběhu pro každý sériový port: při prvním připojení kamery k počítači si Chloros stáhne z kamery přes GigE její tovární kalibrační balíček o velikosti ~3,8 MB (přibližně 70 sekund na kameru). U kamer uložených v mezipaměti se tento panel nikdy nezobrazí. Viz [Připojení kamer](connecting.md).

## Šířka pásma: kolik kamer se vejde

To, kolik kamer může pole přenášet, závisí na vlastnostech sítě, nikoli na Chloros, takže plánovací údaje najdete v příručce k hardwaru: **[Plánování šířky pásma pole](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Co s nimi dělá Chloros: dialogové okno pro připojení spustí test sítě, odhadne dosažitelnou snímkovou frekvenci a vybere odpovídající úroveň. Pokud pole přetěžuje linku, odmítne připojení, místo aby tiše zahazovalo pakety — viz panel s odhadovanými výsledky popsaný výše.

## Když dochází ke ztrátě rámců

Kamera může v publikované skupině chybět ze dvou zcela odlišných důvodů,
které vyžadují opačná řešení. Chloros je počítá odděleně, místo aby hlásil jedno
„neúplné“ číslo, které ani jeden z nich neoznačuje:

| Co se stalo | Co to znamená | Kde hledat |
| --- | --- | --- |
| **Poškozený**— rámec dorazil, ale byl strukturálně poškozen | Ztráta paketů GVSP na síťové trase |**Kapacita linky**, přijímací kruh síťové karty, jumbo rámce, přepínač |
| **Nikdy nedorazil**— žádný rámec vůbec nepřišel | Kamera se nespustila, nebo z ní nic nevyšlo |**Synchronizační kabel M8**, synchronizační linka, zda jsou všechny členy systému v pohotovostním režimu |

Rozdělení se přehodnocuje každých 10 sekund během přenosu dat z pole. Při hodnotě nad 5 % se
zaznamená s uvedením obou čísel a každý poškozený buffer se nahlásí při prvním výskytu
pro každou kameru, poté se údaje agregují jednou za minutu, aby zůstala čitelná i dlouhá relace.

**Poškozené snímky s nulovým počtem „nikdy nedorazilo“ znamenají, že spouštění a synchronizace kabelu jsou v pořádku**a každý ztracený snímek je na síťové trase. Řešením je snížit**Wire Budget** a
znovu se připojit.

{% hint style="warning" %}
**Snížení frekvence spouštění nepomáhá u poškozených snímků.** Pacing paketů kamery
se nastaví pouze jednou, při připojení. Snížení frekvence spouštění mění to, jak často dochází k datové dávce,
nikoli to, jak rychle se samotná dávka přenáší po kabelu. Na měřeném systému se 4 kamerami
pětinové snížení spouštěcí frekvence nic nezměnilo, zatímco snížení wire budgetu z 240 na
200 MB/s snížilo podíl poškozených rámců u stejného systému z 10,4 % na nulu.
{% endhint %}

Běžící pole se nemůže samo přeplánovat – odpojte a znovu připojte, aby nástroj pro výběr času připojení
mohl pracovat s novým limitem.

### Síťové adaptéry USB jsou omezeny na 200 MB/s

USB ethernetový adaptér udává svou *ethernetovou* přenosovou rychlost, ale to, co dokáže skutečně
udržet, je omezeno sběrnicí USB a jejím ovladačem. USB 10GbE adaptér býval uváděn
s propustností přibližně 1000 MB/s – což je číslo, které nikdo nikdy neměřil – a při
čtyř kamer na základě této fiktivní rezervy způsobilo poškození 6–18 % snímků, zatímco pole
stále vykazovalo správnou cílovou snímkovou frekvenci. Adaptéry připojené přes USB jsou nyní omezeny na
**200 MB/s**. Tento limit je absolutní, nikoli procentuální, protože omezujícím faktorem je
sběrnice: adaptér USB 1 GbE dosahuje rychlosti asi 80 MB/s a není tímto omezením dotčen.

Pokud je váš hostitelský počítač skutečně rychlejší než tento limit, zvyšte hodnotu **Wire Budget**, aby to odrážela.

## Časová synchronizace PTP

*Synchronizace* snímků vychází z hardwarového spouštěče; **PTP** (IEEE 1588 PTPv2) zajišťuje srovnatelné *časové značky* napříč všemi zařízeními. Je ve výchozím nastavení povoleno při připojení pole:

* **Hostitelský backend Chloros provozuje PTP grandmaster**. Kamery LATTICE a světelné senzory DAQ-E se k němu připojují jako slave v doméně 0, takže časová razítka snímků a spektra DAQ jsou synchronizována na jeden takt (~1 ms).
* `--no-ptp` (CLI) jej deaktivuje pro práci na zkušební lavici — časová razítka mezi kamerami pak **nejsou** srovnatelná.
* Zkontrolujte stav synchronizace pomocí CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Záložka „Kamery“ sama o sobě neobsahuje indikátor PTP; k dispozici jsou zde pouze pro jednotlivé kamery položky **Role**(Master/Slave),**Sync Line** a úroveň schopností pole, které jsou pouze pro čtení. Stav PTP modulu DAQ-E se zobrazuje v detailech senzorů na záložce „Light Sensors“.

## Živé zobrazení

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows, and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

pole Hlavní oblast přenosu nabízí dvě rozložení (přepínání v horní liště): **mřížkové zobrazení**(každá dlaždice je buňka; přetažením lze změnit pořadí, pokud je zámek mřížky odemčen) a**zobrazení seznamu**(pole v plné šířce nahoře, jedna aktivní kamera dole). Posuvník**Zoom živého přenosu** mění velikost dlaždic; při šířce buňky pod 200 px se překryvy s názvem a snímkovou frekvencí automaticky skryjí.**Samostatný režim** zobrazuje jednu dlaždici na jednu kameru. Každá dlaždice obsahuje:

* název kamery (vlevo nahoře),
* **údaj o fps** (vpravo nahoře) — jedná se o *skutečnou snímkovou frekvenci* kamery hlášenou backendem, nikoli o frekvenci aktualizace náhledu (živý náhled je omezen na 30 fps bez ohledu na snímkovou frekvenci),
* stavovou tečku — zelenou (přenos) / oranžovou (načítání) / červenou (chyba),
* **otočný indikátor zastaralého snímku**, pokud po dobu 2 s nedorazil žádný nový snímek — běžné po dobu ~5 s po jakémkoli připojení/odpojení, zatímco backend přerozděluje šířku pásma mezi kamery.**Kombinovaný režim**zobrazuje jedinou složenou dlaždici: backend provádí debayering, škálování, zarovnání, odšumování, převod na zářivost v jednotlivých pásmech (plus odrazivost DLS, je-li připojen světelný senzor), vyhodnocuje indexový výraz pole, aplikuje LUT a výsledek streamuje ve formátu MJPEG. Dokud se nezobrazí první zarovnaný snímek, dlaždice informuje o svém stavu: „Příprava pole…“, „Kalibrace zarovnání…“, „Čekání na první snímek…“ nebo — pokud vyprší limit pokusů o automatické zarovnání (~30 s) — „Vyžaduje se zarovnání“ s tlačítkem**Kalibrovat zarovnání**.

Užitečné informace o kombinovaném režimu:

* Kompozit je zarovnán k snímku **hlavní**kamery. Zaměření AE-ROI a bodové měření expozice na kompozitu jsou přesné pro hlavní kameru a přibližné pro podřízené kamery; použijte**Rozdělené zobrazení** (nastavení pole → „Zobrazit členské kamery“) pro dlaždice s pixelovou přesností pro každou kameru, aniž byste museli otevírat další připojení kamer.
* **Zobrazit vrstvy**(nastavení pole; výchozí vypnuto) umožňuje vybrat vrstvu popředí a pozadí — libovolnou členskou kameru nebo**Index**. Pokud je popředí = Index, pixely mimo rozsah LUT Min/Max zobrazují vrstvu pozadí.
* **Rozlišení renderování** (výchozí 720p) nastavuje výšku živého přenosu *a* velikost exportovaného kompozitu. Snímky jednotlivých kamer se vždy exportují v plném rozlišení.
* Zarovnání se počítá pro každou relaci a nikdy se neukládá — viz část o zarovnání v panelu nastavení pole, kde najdete zbytkové hodnoty RMS a tlačítko „Rekalibrovat“.

## Snímání: monitorování vs. analýza

Povrchy pro snímání pole se jasně dělí na **úroveň monitorování**(zaznamenává to, co vidíte) a**úroveň analýzy** (zaznamenává surová data, kalibruje se později):

| Pracovní postup | Úroveň | Co se ukládá | GUI | CLI |
| --- | --- | --- | --- | --- |
| **Zaznamenávání**(snímky) | Analýza | Jedna synchronizovaná skupina snímků na jeden průchod; soubory pro každou kameru na každé vybrané úrovni exportu (surové/odbyzerované/radiance/reflektance/náhled/index) + sidecar `.daq` | Tlačítko**Zachytit vše** + Nastavení zachycení | `lattice array-capture` |
| **Záznam indexového videa** | Sledování | Zobrazený živý kompozit kombinovaného indexu — 8 bitů, rozlišení náhledu, zabudovaná LUT; vyžaduje otevřený živý stream | ● Záznam indexového videa (kombinované pole) | `lattice array-record` |
| **Sériový záznam surových dat → sestavení videa**| Analýza | Surové snímky ze snímače při plné snímkové frekvenci + manifest + `.daq`, následně offline rekonstrukce do kalibrovaného videa s indexem radiance / odrazivosti / indexu, časově synchronizovaného s údaji z DAQ | ⦿ Zaznamenat sériový záznam surových dat →**Vytvoření videa** | `lattice array-burst` → `lattice array-build-video` |

Pravidlo: pokud mají pixely poskytovat *měření*, použijte snímání nebo sérii snímků (pro analýzu); pokud potřebujete pouze *sledovat nebo předvést*, co snímač zaznamenal, nahrajte indexové video (pro monitorování).

### Nastavení snímání (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->

Ozubené kolečko vedle položky **Capture All** otevírá panel Nastavení snímání (vyžaduje otevřený projekt — snímky se ukládají do něj):

* **Režim snímání**:**Single**(jeden průchod) /**Continuous**(nepřetržitě; omezeno počtem snímků, výchozí hodnota 1, nebo dobou trvání, výchozí hodnota 10 s) /**Interval** (časosběr: N snímků v intervalu X, celkem Y; výchozí nastavení je 1 snímek každých 5 s po dobu 1 minuty).
* **Typy exportu pro každou kameru**: Raw, Debayered, Radiance, Reflectance, Preview, Index — ve výchozím nastavení jsou zapnuty všechny použitelné možnosti. Radiance/Reflectance jsou skryty u kamer RGB-filter;**Reflectance se zobrazí pouze v případě, že kamera má světelný senzor DAQ** (vlastní nebo zděděný z pole); Index vyžaduje nakonfigurovaný indexový výraz.
* **Aligned**(pro pole, výchozí**zapnuto**): přizpůsobí exporty jednotlivých prvků profilu zarovnání pole, takže exporty jsou pixelově zarovnané. Formát Raw zůstává vždy nepřizpůsobený, ale nese transformaci v metadatech.
* **Nejrychlejší snímání** (přepínač): pouze surová data + přiřazená hodnota DAQ + bezplatný kompozit s kombinovaným indexem, přičemž se při snímání přeskočí kalibrační výpočty pro dosažení maximální rychlosti — radianci/odrazivost/index se později znovu vypočítají z uloženého souboru `.daq`.
* Výběry zůstávají zachovány v rámci projektu. Skryté nebo pozastavené kamery se přeskočí.

Ekvivalentní CLI (stejný koncový bod backendu, stejná sémantika):

```bash
# One synced group, every applicable export level per camera (the default)
chloros-cli lattice array-capture -o output/

# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# 30-second monitoring clip of the combined index view, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# 5-second analysis-grade raw burst, then build the combined index video
chloros-cli lattice array-burst --duration 5 --build --products combined:index --fps 10 -o capture/
```

Komprese snímků pomocí TIFF je `deflate` (bezztrátová, výchozí) nebo `none` — úplné tabulky příznaků, struktura složky se záznamy a pravidla pro opětovné zpracování jsou uvedeny v [CLI Referenci](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Spárování světelného senzoru DAQ

Náhledy s korekcí odrazivosti a osvětlení vyžadují data o dopadajícím světle ze senzoru DAQ (připojeného na kartě **Světelné senzory**):

* V **řádku pole**na postranním panelu se zobrazuje**políčko „DAQ · zap/vyp“** — je *zapnuto*, pokud je nastaven světelný senzor na úrovni pole **nebo** pokud má některá z kamer v poli svůj vlastní; v jeho popisku je přesně uvedeno, který senzor napájí kterou kameru.
* Přiřaďte nastavení pro celé pole v nastavení pole → **Senzor okolního světla**→ rozevírací nabídka**Světelný senzor**. Výběr zůstává zachován v rámci projektu, přenáší se na všechny kamery v poli a jednotlivé kamery jej mohou stále přepsat svým vlastním senzorem.
* Stavový řádek pod ním zobrazuje aktuální stav: **Vypnuto**→ „Čekání na první spektrum…“ →**„Aktivní — u všech kamer v poli je provedena korekce osvětlení“** → nebo, pokud v posledních 3 s nedorazilo žádné nové spektrum, upozornění na zastaralé údaje — nadále se používá poslední naměřená hodnota (hodnoty v cestě snímání nikdy nevyprší).

Je-li přiřazen snímač: stane se dostupný typ exportu „Odrazivost“, živé náhledy jsou korigovány na osvětlení, prediktivní automatická expozice může spektrum využít a každý záznam odrazivosti zapíše údaj DAQ, který byl skutečně použit, jako **`.daq` sidecar** vedle snímku, takže záznam lze později znovu zpracovat.

## Možnosti `.daq` CLI

| Příznak | Výchozí | Popis |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automatické vyhledání všech kamer LATTICE (vyžaduje ≥2) | **První sériové číslo je MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | Synchronizační linka GPIO. |
| `--target-fps F` | auto | Frekvence spouštění masteru. |
| `--binning {1,2,4}` | auto | Hardwarové seskupování. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Expertní přepsání volby synchronizační úrovně. |
| `--wire-ceiling-mbps MB_PER_S` | automaticky detekováno | Kapacita hostitelského připojení v MB/s — forma pole **Wire Budget** v CLI. Snižte jej, pokud pole hlásí poškozené rámce. Ukládá se spolu s projektem, takže při pozdějším opětovném připojení se obnoví. |
| `--no-recommend` | vypnuto | Přeskočit krok analýzy sítě. |
| `--no-ptp` | vypnuto | Zakázat PTP (časová razítka mezi kamerami pak nebudou srovnatelná). |

`lattice array-list`, `array-status` a `array-disconnect` spravují trvalou relaci. Úplný přehled podpříkazů, včetně zarovnání (`align-calibrate` / `align-apply`) a síťových nástrojů, se nachází v [CLI Reference § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); ekvivalenty SDK (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) se nacházejí v [SDK Reference](../reference/sdk-reference.md). Od Python je rozpočet na vedení `connect_array(..., wire_ceiling_mbps=120)` a rozdělení živého poškozeného/nikdy nedoručených je v [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
