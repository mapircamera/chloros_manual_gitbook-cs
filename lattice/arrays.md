# Systémy s více kamerami

**Systém**LATTICE tvoří dvě nebo více kamer LATTICE propojených do jedné synchronizované jednotky. Jedna kamera je**hlavní**: vysílá hardwarový spouštěcí impuls GPIO na sdílenou synchronizační linku (výchozí**Line2**), takže všechny kamery exponují ve stejném okamžiku. Chloros přidává časovou synchronizaci PTP, živý náhled (mozaika jednotlivých kamer nebo jediný zarovnaný vícepásmový kompozit) a synchronizované snímání — každý snímací cyklus vytvoří jednu **skupinu snímků**, ve které mají všechny kamery stejný časový údaj a ID snímku (uváděné jako `fid:N` na výstupu snímání).

Pomocí matic vytvářejí monochromatické (M3M) kamery vegetační indexy – jedna kamera přispívá jedním pásmem a matice je zarovná do vícepásmového souboru. Viz [Mono kamery a vegetační indexy](mono-indices.md).

Existují tři rovnocenné způsoby připojení pole a všechny z nich spouštějí stejný pracovní postup „smart-prep“:

| Rozhraní | Vstupní bod |
| --- | --- |
| GUI | Karta Kamery → **Připojit pole** (modré tlačítko) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (první sériové číslo = master) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (první sériové číslo = master) |

Smart-prep provede: test síťové dostupnosti (ICMP DF ping + GVSP probe), výběr synchronizační úrovně, odhad snímkové frekvence při požadované velikosti snímku, rozhodnutí o komprimovaném 12bitovém přenosu (viz [Balení 12bitového přenosu](#packed-12-bit-transport)), aktivaci PTP, automatický výběr formátu pixelů pro každou kameru, nastavení počáteční hodnoty automatické expozice na základě uloženého stavu každé kamery a konfiguraci spouště GPIO na Line2.

**Ve výchozím nastavení se používá celý snímač (novinka ve verzi 1.3.0).** Connect zachovává požadovanou velikost snímku — plný snímač 2048×1536, bez binningu a ořezu, pokud si nezvolíte jinak — a nechá snímkovou frekvenci klesnout na úroveň, kterou poskytuje hostitelský počítač a síť. Nikdy sám neprovádí binning ani ořez (verze 1.2.0 automaticky spouštěla binning při frekvenci pod 1,5 fps). O binningu nebo menší oblasti zájmu (ROI) rozhodujete vy: v dialogovém okně Connect si jednu z možností vyberte a v části „Projected Outcome“ se zobrazí snímková frekvence, kterou daná volba poskytuje; `array-connect` vypíše rychlejší možnosti s jejich předpokládanými snímkovými frekvencemi.

{% hint style="info" %}
Aby cokoli z toho fungovalo, musí být kamery na spojení dostupné — viz [Připojení kamer](connecting.md) pro vyhledávání, adresování a stažení kalibračních dat při prvním připojení. U sestav s více kamerami záleží na nastavení přijímacího kruhu hostitelské síťové karty stejně jako na rychlosti spojení; kompletní tabulka příznaků a řešení je v [Referenci CLI § Nastavení a ladění hostitelské síťové karty](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Dialogové okno „Array Connect“

Karta „Kamery“ → **Connect Array**otevře tříkrokový průvodce:**Výběr → Režim zobrazení → Nastavení**.

### Krok 1 — Výběr hlavní kamery a podřízených kamer

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered (an FRGB among them). Table showing Camera / Serial / IP / Master radio / Slave checkbox columns in camera order (M3C first, FRGB at the top), with the FRGB pre-selected as master and the green "Sync cable found on N cameras. Any of them can be master; the RGB camera (else the first) is pre-selected." probe banner visible above the table. -->



Dialogové okno ihned po otevření prohledá síť („Prohledávání sítě...“) a poté otestuje synchronizační kabely („Testování synchronizačních kabelů...“), aby zjistilo, které kamery jsou připojeny k synchronizační lince M8. K vytvoření pole potřebujete alespoň **2 kamery**.

Synchronizační kabely M8 se sbíhají v jedné rozbočce, takže **hlavní kamerou může být libovolná připojená kamera**. Testovací procedura předem přiřadí role — kamera RGB (`-FRGB`) jako hlavní, pokud je připojena kabelem, jinak první připojená kamera, a ostatní připojené kamery jako podřízené — a zobrazí jeden ze tří nápisů:

| Nápis | Význam |
| --- | --- |
| „Na {n} kamerách byl nalezen synchronizační kabel. Jakákoli z nich může být hlavní; předem je vybrána RGB kamera (jinak první kamera).“ (zelená) | Sonda našla kamery na synchronizační lince; zaškrtávací políčka pro hlavní a podřízenou kameru jsou již vyplněna. |
| „Není detekováno žádné hlavní zařízení – zkontrolujte kabel GPIO“ (oranžová) | Žádná kamera nezaznamenala spouštěcí impuls; zkontrolujte synchronizační kabeláž. Role jsou stále předem vyplněny (RGB kamera, jinak první kamera v seznamu, jako hlavní) a můžete je změnit. |
| „Žádný synchronizační kabel: {seriální porty}“ (oranžová) | U uvedených kamer není připojen žádný synchronizační kabel. |

Tabulka kamer obsahuje sloupce **Kamera / Sériové číslo / IP / Hlavní (rádiová) / Podřízená (zaškrtávací políčko)**. Řádky jsou seřazeny podle**pořadí kamer** — kamery M3C (Bayer) před kamerami M3M (mono); mezi kamerami M3C, FRGB, FRGN, FOCN a poté FNGB; kamery M3M podle vlnové délky filtru, nejprve nejnižší (F450 před F808). Seznam kamer v postranním panelu a výchozí rozložení mřížky používají stejné pořadí.

* Vyberte přesně **jednoho hlavního**a**jednoho nebo více podřízených**. Opětovným kliknutím na přepínač aktuálního hlavního zařízení se jeho výběr zruší. Výběr jiného hlavního zařízení se provádí jedním kliknutím: nové hlavní zařízení opustí skupinu podřízených a předchozí hlavní zařízení se stane podřízeným (pokud nemá synchronizační kabel).
* Kameru označenou **„No Sync Cable“** nelze nikdy vybrat jako podřízenou – podřízená kamera bez spouštěcího kabelu by čekala na synchronizační linku donekonečna a poskytovala by nefunkční signál. Tuto kameru místo toho připojte jako samostatnou kameru.
* Kamery, které jsou již připojeny samostatně, *nejsou* deaktivovány: připojení do pole uvolní samostatnou relaci a znovu otevře kameru uvnitř pole.

**Další: Režim zobrazení →**se aktivuje, jakmile je vybrán hlavní přístroj a alespoň jeden podřízený přístroj.**Znovu prohledat** znovu spustí vyhledávání a testování zapojení.

{% hint style="warning" %}
**Zrušit** je deaktivováno, dokud probíhá skenování nebo testování – zrušení uprostřed testu může způsobit pád kamerového SDK na firmwaru kamery LATTICE. Počkejte, až se dokončí otáčející se ikona.
{% endhint %}

### Krok 2 — Režim zobrazení



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

| Režim | Co získáte |
| --- | --- |
| **Samostatné kamery** | Jedna živá dlaždice pro každou kameru, všechny se spouštějí současně, takže snímky zůstávají synchronizované. Každá kamera si zachovává vlastní barvu a nastavení. |
| **Kombinované kamery** *(výchozí)* | Jedna dlaždice zobrazující zarovnaný vícepásmový kompozit NDVI/index. Kamery sdílejí barvu pole. |

Režim zobrazení mění pouze prezentaci živého náhledu — chování při snímání je v obou případech stejné.

### Krok 3 — Nastavení pole a předpokládaný výsledek

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Trigger Rate / Wire Budget controls (1.3.0 has no "Pin resolution" checkbox), right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->



Při vstupu do této scény si Chloros vyžádá od backendu **doporučení**a aplikuje ho:**plný snímač — bez binningu, bez oříznutí ROI — kdykoli to snímková frekvence dovolí**, bez ohledu na to, jakou frekvenci poskytuje hostitel a přenosové médium. Teprve pokud žádná frekvence nedokáže zajistit plný snímač, doporučení snímek zmenší, a v takovém případě upřednostňuje binning před ořezáním ROI, protože binning zachovává plné zorné pole. Od tohoto bodu se rozhodujete mezi rozlišením a snímkovou frekvencí. Každá provedená změna spustí analýzu v reálném čase a aktualizuje panel**Očekávaný výsledek** na pravé straně.

Levý sloupec — nastavení:

| Ovládací prvek | Možnosti | Výchozí | Poznámky |
| --- | --- | --- | --- |
| **ROI (zorné pole)** | Plné (2048×1536) / Poloviční (1024×768) / Čtvrtinové (512×384) | Plné | Oříznutí snímače: Poloviční/čtvrtinové oříznutí na menší oblast při nativní velikosti pixelů. |
| **Binning** | 1× / 2× (součet 2×2) / 4× (součet 4×4) | 1× | Hardwarové binning: 2×2 = plné zorné pole při čtvrtinových nákladech na přenos; 4×4 = plné zorné pole při 1/16. Skryto, pokud kamery nepodporují binning. |
| **Obraz na straně sběrnice** (výstup) | — | — | Šířka × výška po binningu skutečně odeslaná po sběrnici, zaokrouhlená na násobky 16 (minimálně 64). |
| **Frekvence spouštění** | 0,5–60 fps, krok 0,1 | prázdné = auto | Frekvence spouštění masteru. Nechte prázdné, aby ji Chloros odvodil sám. |
| **Kapacita přenosu**| 20–2000 MB/s, krok 10 | prázdné = auto | Kolik hostitel dokáže skutečně zpracovat, v MB/s —**jediné číslo, na kterém závisí celé přidělení pole.** Automaticky detekováno ze síťového adaptéru. Snižte tuto hodnotu, pokud pole hlásí poškozené rámce: detekovaná hodnota nadhodnocuje USB adaptéry a sdílené přepínače. Změna této hodnoty spustí nové živé vypočítání. Pole se otevře s limitem, se kterým bylo naposledy připojeno pole (uloženo s projektem), takže limit, který jste jednou upravili, zůstane zachován. |

Zaškrtávací políčko **Rozlišení pinů** z verze 1.2.0 bylo odstraněno: při připojení se vždy zachová vámi zvolená velikost rámce a místo toho se sníží snímková frekvence.

Pravý sloupec — **Předpokládaný výsledek**:

* **Úroveň synchronizace** — „Souběžné snímání“ (zelená), „Souběžné snímání (FTD – střídavé vysílání)“ (zelená), „Střídavé snímání (100 ms zpoždění)“ (oranžová) nebo „Konfigurace je příliš velká“ (červená). Chloros vybere úroveň: simultánní snímání vždy, když lze sdílet linku rychlostí, na kterou lze přizpůsobit kamery (snížuje se snímková frekvence, jak se zmenšuje podíl jednotlivých kamer), a střídavé snímání pouze tehdy, když to není možné — takže pole se připojí k jakékoli síti při snímkové frekvenci, kterou daná síť umožňuje.
* **Projekce fps** — snímková frekvence, při které by měl systém běžet. Při simultánním snímání se jedná o jedno číslo: nejprve se určí snímková frekvence a automatická expozice vyplní daný časový úsek, poté zvýší zisk, aby ji neoslabila tmavá scéna (viz [Jak automatická expozice rozděluje expozici a zisk](camera-settings.md)). Při střídavém snímání se zobrazuje jako rozsah („tmavé → světlé“), protože expozice každé kamery prodlužuje časový posun v tmavé scéně.
* **Linka NIC** — rychlost spojení a trvalá propustnost („NIC {mbps} Mbps · trvalá {N} MB/s“).
* **Kontrola souběžného vysílání** — dokáže přijímací kruh síťové karty hostitele přijmout jeden souběžný datový proud ze všech kamer („Souběžné vysílání: X MB · využitelný kruh síťové karty: Y MB ✓/✗“).
* **Kontrola kapacity linky** — souhrnná poptávka v ustáleném stavu vs. strop linky bez kolizí („Kapacita linky: {poptávka} MB/s požadovaná {n} kamerami · strop {strop} MB/s ✓/✗ překročená kapacita“).
* **„Maximální počet kamer na tomto kabelu: {n} — nastaveno minimální šířkou pásma na kameru, takže seskupování tento počet nezvýší.“** — zobrazí se, když se blížíte k (nebo překračujete) maximálnímu počtu kamer.
* **„Při těchto nastaveních DOJDE K VYNECHÁVÁNÍ SNÍMKŮ.“**— červené varování s důvodem ze strany backendu, plus seznam překážek a modré**návrhy na řešení** („Aby se toto pole vešlo do sítě“ / „Aby se odblokovalo souběžné snímání“).

Tlačítko **Použít a připojit** je zablokováno, dokud neexistuje prognóza, a jeho popisek vám sdělí, proč je odmítáno:

| Popisek tlačítka | Význam | Co skutečně pomůže |
| --- | --- | --- |
| „Analyzuji...“ | Analýza stále probíhá. | Počkejte. |
| **„Příliš mnoho kamer pro tuto síť“**| Síť nedokáže přenášet pole ani po jedné kameře najednou (selhala kontrola agregace): je zde mnohem více kamer, než kolik síť zvládne, nebo na ní streamují jiné kamery. | Méně kamer, jumbo rámce v celém řetězci nebo rychlejší síťová karta.**Menší oblast zájmu (ROI) nepomůže** — viz níže. |
| **„Zmenšete oblast zájmu (ROI) pro povolení“** | Při těchto nastaveních by docházelo ke ztrátám snímků (selhala kontrola burst/ring). | Zmenšete oblast zájmu (ROI), zvyšte binning nebo opravte přijímací kruh síťové karty. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->



Během připojování se může zobrazit zelený **panel stahování kalibrace** s ukazatelem průběhu pro každé sériové číslo: při prvním připojení kamery k počítači si Chloros stáhne z kamery přes GigE její tovární kalibrační balíček o velikosti ~3,8 MB (přibližně 70 sekund na kameru). U kamer uložených v mezipaměti se tento panel nikdy nezobrazí; při více než šesti kamerách se jeho řádky posouvají. Viz [Připojení kamer](connecting.md).

### Komprimovaný 12bitový přenos

12bitové formáty (`BayerRG12`, `Mono12`) se obvykle přenášejí jako 16bitová slova. **Komprimovaný**přenos odesílá stejných 12 bitů v 1,5 bajtu na pixel a dekomprimuje je na hostitelském počítači, takže odeslání každého snímku trvá o 25 % méně času — při plném využití snímače to znamená zhruba o třetinu více snímků za sekundu.**Uložená data jsou identická.** Funkce „Array connect“ to rozhoduje automaticky (`auto`): komprimuje pouze tehdy, když je to výhodné a hostitelský počítač má volný výpočetní výkon pro dekompresi. Grafické uživatelské rozhraní (GUI) pro to nemá žádné přepínání; z příkazového řádku (CLI) použijte `array-connect --packed {auto,on,off}` (výchozí nastavení je `auto`). 8bitové a 16bitové formáty se nikdy nekomprimují.

## Šířka pásma: kolik kamer se vejde

To, kolik kamer může pole přenášet, závisí na připojení, nikoli nChloros

u, takže plánovací údaje najdete v manuálu k hardwaru: **[Plánování šířky pásma pole](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Jak s nimi nakládá Chloros: dialogové okno pro připojení spustí test sítě, odhadne dosažitelnou snímkovou frekvenci a vybere odpovídající úroveň. Pole se připojí k jakékoli síti: na pomalém spojení se každá kamera přizpůsobí svému podílu na šířce pásma a snímková frekvence klesne, místo aby bylo pole odmítnuto nebo aby byly jeho snímky zmenšeny. Odmítnuto bude pouze pole, u kterého linka nedokáže přenášet ani jednu kameru najednou (**Příliš mnoho kamer pro tuto síť**), místo aby docházelo k tichému zahazování paketů — viz panel s odhadovanými výsledky popsaný výše.

## Když snímky chybí

Kamera může v publikované skupině chybět ze dvou zcela odlišných důvodů,
a každý z nich vyžaduje opačné řešení. Chloros je počítá odděleně, místo aby uváděl jedno
„neúplné“ číslo, které ani jeden z nich neoznačuje:

| Co se stalo | Co to znamená | Kde hledat |
| --- | --- | --- |
| **Poškozený**— snímek dorazil, ale byl strukturálně poškozen | Ztráta paketů GVSP na síťové trase |**Kapacita linky**, přijímací kruh síťové karty, jumbo rámce, přepínač |
| **Nikdy nedorazil**— žádný rámec vůbec nepřišel | Kamera se nespustila, nebo z ní nic nevyšlo |**Synchronizační kabel M8**, synchronizační linka, zda jsou všichni členové v pohotovostním režimu |

Rozdělení se přehodnocuje každých 10 sekund během přenosu dat z pole. Při hodnotě nad 5 % se
zaznamená s uvedením obou čísel a každý poškozený buffer se nahlásí při prvním výskytu
pro každou kameru, poté se údaje agregují jednou za minutu, aby zůstala čitelná i dlouhá relace.

**Poškozené snímky s nulovým počtem „nikdy nedorazilo“ znamenají, že spouštění a synchronizace kabelu jsou v pořádku**a každý ztracený snímek je na síťové trase. Řešením je snížit**Wire Budget** a
znovu se připojit.

{% hint style="warning" %}
**Snížení frekvence spouštění nepomáhá u poškozených snímků.** Tempo paketů kamery
se nastavuje jednorázově při připojení. Snížení frekvence spouštění mění, jak často dochází k
sériovému přenosu, nikoli jak rychle se samotný sériový přenos dostává na kabel. Na testovaném systému se 4 kamerami
pětinásobné snížení spouštěcí frekvence nic nezměnilo, zatímco snížení wire budgetu z 240 na
200 MB/s snížilo podíl poškozených snímků u stejného systému z 10,4 % na nulu.
{% endhint %}

Spuštěné pole se nemůže samo přeplánovat — odpojte a znovu připojte, aby nástroj pro volbu času připojení
mohl pracovat s novým limitem.

### USB síťové adaptéry jsou omezeny

USB ethernetový adaptér udává svou *ethernetovou* přenosovou rychlost, ale to, co dokáže skutečně
udržet, je omezeno USB sběrnicí a jejím ovladačem. USB 10GbE adaptér býval uváděn
s propustností přibližně 1000 MB/s — což je hodnota, kterou nikdo nikdy neměřil — a přizpůsobení
čtyř kamer tomuto fiktivnímu výkonu způsobilo poškození 6–18 % snímků, zatímco pole
stále vykazovalo správnou cílovou snímkovou frekvenci. Adaptéry připojené přes USB jsou nyní omezeny na
**200 MB/s** v systému Windows a na platformě NVIDIA Jetson a **400 MB/s** v systému Linux x86_64. Tento limit je absolutní, nikoli procentuální, protože omezujícím faktorem je
sběrnice: adaptér USB 1 GbE dosahuje rychlosti asi 80 MB/s a není tímto omezením dotčen.

Pokud je váš hostitel skutečně rychlejší než tento limit, zvyšte hodnotu **Wire Budget**, aby to odrážela.

## Časová synchronizace PTP

*Synchronizace* snímků vychází z hardwarového spouštěče; **PTP** (IEEE 1588 PTPv2) zajišťuje srovnatelné *časové značky* napříč všemi zařízeními. Je ve výchozím nastavení zapnuta při připojení pole:

* **Backend hostitele Chloros spouští PTP grandmaster**. Kamery LATTICE a světelné senzory DAQ-E se k němu připojují jako slave v doméně 0, takže časová razítka snímků a spektra DAQ se shodují s jedním taktem (~1 ms).
* `--no-ptp` (CLI) tuto funkci deaktivuje pro práci na pracovním stole — časová razítka mezi kamerami pak **nejsou** srovnatelná.
* Zkontrolujte stav synchronizace pomocí CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity, health (OK, or why its packets are not reaching the network)
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Záložka „Kamery“ sama o sobě neobsahuje indikátor PTP; k dispozici jsou zde pouze pro čtení položky **Role**(Master/Slave),**Sync Line** a úroveň schopností pole. Stav PTP pro DAQ-E se zobrazuje v detailech senzorů na záložce „Světelné senzory“.

## Živé zobrazení

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows in camera order (M3C before M3M, monos by wavelength), and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

pole Hlavní oblast zobrazení nabízí dvě rozložení (přepínání v horní liště): **mřížkové zobrazení**(každá dlaždice je buňka; přetažením lze změnit pořadí, pokud je zámek mřížky odemčen) a**zobrazení seznamu**(řádky přes celou šířku:**kombinované**pole tvoří jeden řádek – jeho složená dlaždice –, zatímco**samostatné**pole přiděluje**každému členu vlastní řádek**s vlastním živým přenosem a vlastní tabulkou kalibračních cílů; pod poli je jedna samostatná kamera — ta aktivní). Posuvník**Zoom živého přenosu** mění velikost dlaždic; při šířce buňky pod 200 px se překryvy s názvem a fps automaticky skryjí. Členské kamery se zobrazují v postranním panelu a ve výchozím rozložení mřížky v pořadí kamer (M3C před M3M, nejprve FRGB, monochromatické podle vlnové délky); odemknutí mřížky a přetažení však i nadále přepisuje pořadí v mřížce.

<!-- SCREENSHOT-NEEDED: list view of a connected SEPARATE array (1.3.0): one full-width row per member camera, each with its live feed and its own Calibration Target table below it. -->



**Samostatný režim** zobrazuje jednu dlaždici na jednu kameru. Každá dlaždice obsahuje:

* název kamery (vlevo nahoře),
* **údaj o fps** (vpravo nahoře) — jedná se o *skutečnou snímkovou frekvenci* kamery hlášenou backendem, nikoli o frekvenci aktualizace náhledu (živý náhled je omezen na 30 fps bez ohledu na snímkovou frekvenci),
* stavovou tečku — zelenou (přenos) / oranžovou (načítání) / červenou (chyba),
* **otočný indikátor zastaralého snímku**, pokud po dobu 2 s nedorazil žádný nový snímek — běžné po dobu ~5 s po jakémkoli připojení/odpojení, zatímco backend přerozděluje šířku pásma mezi kamery.**Kombinovaný režim**zobrazuje jedinou složenou dlaždici: backend provádí debayering, škálování, zarovnání, odšumování, převod na radianci v jednotlivých pásmech (plus odrazivost DLS, je-li připojen světelný senzor), vyhodnotí indexový výraz pole, aplikuje LUT a výsledek streamuje ve formátu MJPEG. Dokud se nezobrazí první zarovnaný snímek, dlaždice informuje o svém stavu: „Příprava pole…“, „Kalibrace zarovnání…“, „Čekání na první snímek…“ nebo – pokud vyprší čas na opakované pokusy o automatické zarovnání dlaždice (asi jedna minuta) – „Vyžaduje se zarovnání“ s tlačítkem**Kalibrovat zarovnání**.

Zobrazení fps dlaždice pole také informuje o stavu streamu:**„Aplikuji nastavení…“**, když změna nastavení (formát, binning, ROI) restartuje kamery,**„Zastaveno“**, když je pole aktivní, ale přestalo dodávat skupiny snímků, a**„Zastaveno“**, když přestalo vysílat – v takovém případě pole znovu připojte. Pokud spektrální rozklad musel omezit pásmo zobrazené v kompozitu na velké části jeho pixelů (25 % nebo více), dlaždice zobrazí**„Data pásma s nízkou spolehlivostí · N %“**: indexové hodnoty založené na tomto pásmu jsou pochybné (obvykle jedno pásmo přijalo příliš málo světla).

Užitečné informace o kombinovaném režimu:

* Kompozit je zarovnán k snímku **hlavní**kamery. Zaměření AE-ROI a bodové měření expozice na kompozitu jsou přesné pro hlavní kameru a přibližné pro podřízené kamery; použijte**Split View** (nastavení pole → „Zobrazit členské kamery“) pro dlaždice s pixelovou přesností pro každou kameru, aniž byste museli otevírat další připojení kamer.
* ****Zobrazit vrstvy**(nastavení pole; výchozí nastavení vypnuto) umožňuje vybrat vrstvu v popředí a na pozadí — libovolnou členskou kameru nebo**Index**. Pokud je v popředí nastaveno**Index**, pixely mimo rozsah LUT Min/Max zobrazují vrstvu na pozadí.
* **Rozlišení renderování** (výchozí 720p) nastavuje výšku živého přenosu *a* velikost exportovaného kompozitu. Obrázky jednotlivých kamer se vždy exportují v plném rozlišení.
* Zarovnání se počítá pro každou relaci a nikdy se neukládá. Chloros **automaticky zarovná každou matici po připojení, v obou režimech zobrazení**, jakmile se ustálí automatická expozice členů (pokouší se o to až 5 minut). Sekce**Zarovnání**v panelu nastavení pole zobrazuje zbytkové hodnoty RMS a tlačítko**Pře kalibrovat zarovnání** — a to i pro samostatná pole, která nemají kompozitní dlaždici.

## Snímání: monitorování vs. analýza

Povrchy pro snímání pole se jasně dělí na **úroveň monitorování**(zaznamenává to, co vidíte) a**úroveň analýzy** (zaznamenává surová data, kalibruje se později):

| Pracovní postup | Úroveň | Co ukládá | GUI | CLI |
| --- | --- | --- | --- | --- |
| **Zachycení**(snímky) | Analýza | Jedna synchronizovaná skupina snímků na jeden průchod; soubory pro každou kameru na každé vybrané úrovni exportu (surové/odbyzerované/radiance/reflektance/náhled/index) + sidecar `.daq` |**Zaznamenat vše** + Nastavení záznamu | `lattice array-capture` |
| **Záznam indexového videa** | Monitorování | Zobrazený živý kompozit kombinovaného indexu — 8 bitů, rozlišení náhledu, zabudovaná LUT. Z CLI, aniž by bylo otevřeno grafické rozhraní, backend vykreslí NDVI pole (nebo `--expr`), jakmile je pole zarovnáno; záznam, který nezapíše žádný snímek, selže | ● Záznam indexového videa (kombinovaná pole) | `lattice array-record` |
| **Sériový záznam surových dat → sestavení videa**| Analýza | Surové snímky ze senzoru při plné snímkové frekvenci + manifest + `.daq`, následně offline rekonstrukce do kalibrovaného videa s radiancí / odrazivostí / indexem, časově synchronizovaného s hodnotami z DAQ | ⦿ Záznam sériového surového datového toku →**Sestavení videa** | `lattice array-burst` → `lattice array-build-video` |

Praktické pravidlo: pokud pixely budou sloužit k *měření*, použijte snímání nebo sérii snímků (pro analýzu); pokud potřebujete pouze *sledovat nebo předvést*, co snímač zachytil, nahrajte indexové video (pro monitorování).

### Nastavení snímání (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->



Ozubené kolečko vedle položky **Capture All** otevře panel Nastavení snímání (vyžaduje otevřený projekt — snímky se ukládají do něj):

* **Režim snímání**:**Jednorázové**(jeden průchod) /**Kontinuální**(za sebou; omezeno počtem snímků, výchozí hodnota 1, nebo dobou trvání, výchozí hodnota 10 s) /**Interval** (časosběr: N snímků v intervalu X, celkem Y; výchozí nastavení je 1 snímek každých 5 s po dobu 1 minuty).
* **Typy exportu pro každou kameru**: Raw, Debayered, Radiance, Reflectance, Preview, Index — ve výchozím nastavení jsou zapnuty všechny použitelné možnosti. Radiance/Reflectance jsou skryty u kamer s RGB filtrem; **Reflectance se zobrazí pouze tehdy, má-li kamera světelný senzor DAQ** (vlastní nebo zděděný z pole); Index vyžaduje nakonfigurovaný indexový výraz.
* **Aligned**(pro celé pole, výchozí nastavení**zapnuto**): přizpůsobí exporty jednotlivých členů profilu zarovnání pole, takže exporty jsou pixelově zarovnány. Surová data zůstávají vždy nezdeformovaná, ale transformaci nesou v metadatech. Soubory členů pole se ukládají v nativní orientaci každé kamery (jakékoli nastavení zrcadlení/rotace se zaznamená do souboru, ale nepoužije se).
* **Nejrychlejší snímání** (přepínač): pouze surová data + přiřazená hodnota DAQ + volná kompozice s kombinovaným indexem, přičemž se při snímání přeskočí kalibrační výpočty pro dosažení maximální rychlosti — radianci/reflektanci/index lze později znovu vypočítat z uloženého souboru `.daq`.
* Výběry zůstávají zachovány v rámci projektu. Skryté nebo pozastavené kamery se přeskočí.
* **Kontinuální** režim u polí běží uvnitř backendu Chloros (novinka ve verzi 1.3.0): pokud každá kamera v seznamu patří do pole, backend spustí snímací smyčku sám, zapisuje jednu skupinu a zároveň vybírá další, bez požadavku na každý jednotlivý snímek. Plný disk zastaví běh („Zastaveno: disk plný. Uloženo N, F selhalo“). Viz [Nastavení a režimy snímání](capture.md#capture-modes) pro poznámku k Microsoft Defenderu ohledně vysokých rychlostí snímání.

Ekvivalentní příkaz CLI (stejný koncový bod backendu, stejná sémantika):

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

Komprese TIFF u pro snímky je `deflate` (bezztrátová, výchozí) nebo `none` — úplné tabulky příznaků, struktura složky pro snímky a pravidla pro opětovné zpracování jsou uvedeny v [Referenci CLI](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Spárování světelného senzoru DAQ

Náhledy s korekcí odrazivosti a osvětlení vyžadují data o dopadajícím světle ze senzoru DAQ (připojeného na kartě **Světelné senzory**):

* V **řádku pole**na postranním panelu se zobrazuje**tlačítko „DAQ · zap/vyp“** — je *zapnuto*, pokud je nastaven světelný senzor na úrovni pole **nebo** má některá z kamer v poli svůj vlastní; v jeho popisku je přesně uvedeno, který senzor napájí kterou kameru.
* Přiřaďte nastavení pro celé pole v nastavení pole → **Senzor okolního světla**→ rozevírací nabídka**Světelný senzor**. Výběr zůstává zachován v rámci projektu, přenáší se na všechny kamery v poli a jednotlivé kamery jej mohou i nadále přepsat svým vlastním senzorem.
* Stavový řádek pod ním zobrazuje aktuální stav: **Vypnuto**→ „Čekání na první spektrum…“ →**„Aktivní — u všech kamer v poli je provedena korekce osvětlení“** → nebo, pokud v posledních 3 s nedorazilo žádné nové spektrum, upozornění na zastaralé údaje — nadále se používá poslední naměřená hodnota (naměřené hodnoty na snímací cestě nikdy nevyprší).

Je-li přiřazen snímač: stane se dostupný typ exportu „Reflectance“, živé náhledy jsou korigovány na osvětlení, prediktivní automatická expozice může spektrum využít a každý záznam odrazivosti zapíše skutečně použitou hodnotu z DAQ jako **soubor `.daq`** do složky `daq/`, takže záznam lze později znovu zpracovat. Vedle něj se nachází soubor `.daq.cameras.json`, který uvádí názvy kamer, jejichž snímky daná hodnota popisuje; při opětovném zpracování program Chloros přiřadí tuto hodnotu pouze k obrázkům z těchto kamer.

## `array-connect` – volby CLI



| Příznak | Výchozí | Popis |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automatické vyhledání všech kamer LATTICE (vyžaduje ≥2) | **První sériové číslo označuje MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | Synchronizační linka GPIO. |
| `--target-fps F` | auto | Frekvence spouštění Masteru. |
| `--binning {1,2,4}` | auto | Hardwarové binning. Nastavení „auto“ znamená plný snímač (bez binningu), kdykoli to snímková frekvence umožňuje; volba „connect never bins“ se používá pouze k udržení vysoké frekvence. |
| `--packed {auto,on,off}` | `auto` | [Komprimovaný 12bitový přenos](#packed-12-bit-transport): `auto` komprimuje pouze tehdy, když je to užitečné a host si může dovolit dekompresi. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Expertní přepsání volby synchronizační vrstvy. |
| `--wire-ceiling-mbps MB_PER_S` | automaticky detekováno | Kapacita sítě hostitele v MB/s — CLI ová forma pole **Wire Budget**. Snižte ji, pokud pole hlásí poškozené rámce. Ukládá se s projektem, takže při pozdějším opětovném připojení se obnoví. |
| `--no-recommend` | vypnuto | Přeskočit krok analýzy sítě. |
| `--no-ptp` | vypnuto | Zakázat PTP (časová razítka mezi kamerami pak nejsou srovnatelná). |

`lattice array-list`, `array-status` a `array-disconnect` spravují trvalou relaci. Úplný přehled podpříkazů, včetně zarovnání (`align-calibrate` / `align-apply`) a síťových nástrojů, najdete v [Referenci CLI § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); ekvivalenty v SDK (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) jsou uvedeny v [Referenci SDK](../reference/sdk-reference.md). Z pohledu jazyka Python je rozpočet přenosu `connect_array(..., wire_ceiling_mbps=120)` a rozdělení na živé, poškozené a nikdy nedoručené je uvedeno v [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
