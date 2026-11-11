# Nastavení kamer

Záložka **Kamery** představuje živý ovládací panel aplikace Chloros

pro kamery LATTICE: hlavní oblast živého přenosu, kde se každá připojená kamera zobrazuje jako živá dlaždice, a postranní panel, který umožňuje přepínání mezi třemi stránkami – **seznamem kamer**,**panelem nastavení**(nastavení pro jednotlivé kamery, pole kamer nebo nastavení snímání – vždy jedno najednou) a**Index Calculator**. Tato stránka popisuje všechny ovládací prvky v seznamu kamer, v panelu nastavení jednotlivých kamer a v panelu nastavení pole kamer. Režimy snímání, výběr typu exportu a postup „Capture All“ jsou popsány na související stránce [Nastavení a režimy snímání](capture.md).

Záložka „Kamery“ se v postranním panelu zobrazí, jakmile je backend Chloros

připraven. Všechny níže uvedené ovládací prvky komunikují s lokálním backendem přes `127.0.0.1:5000`; změny se na živém obrazu kamery projeví okamžitě, není-li uvedeno jinak.

## Typy kamer používané na této stránce

Ovládací prvky se zobrazují nebo skrývají v závislosti na tom, jaký typ kamery je vybrán. V této příručce se používají následující termíny:

| Termín | Význam | Kanály filtru |
| --- | --- | --- |
| **RGB

kamera** | LATTICE M3C s filtrem FRGB (model obsahuje `-FRGB`) | Red

/ Green

/ Blue

|
| **Multispektrální kamera typu Bayer** | LATTICE M3C s filtrem FRGN, FOCN nebo FNGB | FRGN: Red

/ Green

/ NIR

· FOCN: Orange

/ Cyan

/ NIR

· FNGB: NIR

/ Green

/ Blue

|
| **Mono (M3M)** | LATTICE M3M — jeden úzkopásmový filtr, jedno kalibrované pásmo | Jedno pásmo |
| **Člen pole** | Kamera připojená jako součást synchronizovaného pole (kombinované nebo samostatné zobrazení) | Podle svého filtru |

Kamery RGB

procházejí fotometrickým zpracováním (vyvážení bílé, barevné profily, gama); multispektrální a mono kamery procházejí radiometrickým řetězcem a přeskočí fotometrická nastavení. Členové pole předávají nastavení na úrovni datového proudu (formát pixelů, rozlišení, binning, spouštění, snímková frekvence) do pole — tyto řádky se v panelu pro jednotlivé kamery stanou pouze pro čtení a přesunou se do panelu nastavení pole.

## Hlavní oblast živého přenosu

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->



Pokud nejsou připojeny žádné kamery, zobrazí se v oblasti živého přenosu úvodní obrazovka **„Připojte kameru a začněte“**se dvěma tlačítky:**Připojit kameru**(zelené, otevře dialogové okno pro připojení jedné kamery) a**Připojit pole** (modré, otevře dialogové okno pro připojení pole). Samotná dialogová okna pro připojení jsou popsána v [Připojování kamer](connecting.md); pojmy týkající se pole (synchronizace, vrstvy, šířka pásma) v [Seskupení více kamer](arrays.md). Při otevření uloženého projektu, který obsahuje kamery, se na úvodní obrazovce místo toho zobrazí ikona načítání s textem „Znovu otevírám N uložených kamer…“, zatímco Chloros

obnovuje streamy z poslední relace.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Horní lišta

| Ovládací prvek | Funkce |
| --- | --- |
| **Přepínání režimu zobrazení**| Přepíná mezi**mřížkovým zobrazením**(všechny dlaždice jako buňky) a**seznamovým zobrazením**(řádky přes celou šířku: každá kombinovaná sestava jako jeden složený řádek, každý člen**samostatné** sestavy jako vlastní řádek s vlastním přenosem a tabulkou kalibračních cílů, a pod tím JEDNA samostatná kamera – ta aktivní). Popisky: „Přepnout na mřížkové zobrazení“ / „Přepnout na seznamové zobrazení“. |
| **Zamknutí mřížky**(visací zámek) | Výchozí nastavení**zamčeno** — dlaždice jsou zafixovány na místě. Odemkněte a přetáhněte dlaždice do libovolného slotu (mezery zůstanou zachovány). Mřížka se automaticky znovu uzamkne pokaždé, když se připojí nová kamera. Nápovědy: „Odemknout mřížku (povolit přetahování dlaždic)“ / „Zamknout mřížku (zafixovat dlaždice na místě)“. |
| Posuvník **Zoom kanálu** | Velikost dlaždice, od 60 px až po plnou šířku kontejneru. Buňky si zachovávají poměr stran 4:3. Při šířce buňky pod 200 px se překryvy s názvem a fps skryjí, aby dlaždice zůstala přehledná. |

### Dlaždice živého přenosu

Každá kamera vykresluje kompozitní živou dlaždici; kamera může navíc zobrazovat tři dlaždice ve stupních šedi **s rozdělením podle kanálů** (viz [Rozdělení kanálů](#display-overlays-drawn-over-the-live-feed)), a pole kamer vykreslují kombinovanou dlaždici. Aktivní dlaždice je označena výběrovým prstencem v barvě kamery (nebo pole kamer).

Při najetí myší na dlaždici se zobrazí tlačítko pro zavření **X**:

* Zavření **složeného** dlaždice, zatímco její rozdělené kanály zůstávají zobrazeny, pouze skryje složený obraz.
* Zavření **poslední viditelné dlaždice samostatné kamery** odpojí danou kameru.
* **Rozdělené dlaždice členů kombinovaného pole nikdy kameru neodpojí** — pouze ji skryjí.

Je-li mřížka odemčena, přetáhněte libovolnou dlaždici do libovolného slotu; rozložení se uloží spolu s projektem.

## Postranní panel — seznam

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

kamer První stránka postranního panelu obsahuje seznam všech připojených kamer a polí:

* **Připojit kameru**(zelené) /**Připojit pole** (modré, během skenování se zobrazuje „Detekuji...“). Obě možnosti jsou deaktivovány, pokud je otevřeno dialogové okno pro připojení.
* **Zachytit vše** (červené) — zachytí všechny uvedené kamery s typy exportu vybranými v nastavení zachycování. Vyžaduje otevřený projekt. Podrobná dokumentace je k dispozici v [Nastavení a režimy snímání](capture.md).
* **Ozubené kolečko Nastavení snímání** (vedle tlačítka Zachytit vše) — otevře [panel Nastavení snímání](capture.md#the-capture-settings-pane). Je deaktivováno, pokud není otevřen projekt nebo během snímání.

### Řádky kamer

Každý řádek kamery zobrazuje barevně odlišený okraj (vlastní barva kamery), označení „CAM“ — s modrým písmenem **M**(master) nebo zelené**S** (slave) pro členy pole — a zobrazovaný název. Výchozí název je `LATTICE-MODEL (serial)`; přejmenujte jej v panelu nastavení jednotlivých kamer. Tlačítka v řádku:

| Tlačítko | Účinek |
| --- | --- |
| **Oko**| Přepíná viditelnost. Skryté kamery zmizí z mřížky a jsou**vyloučeny z funkce Zachytit vše**. |
| **Ozubené kolečko** | Otevře panel nastavení jednotlivých kamer (další část). |
| **Pozastavit / Přehrát**| Zastaví živý náhled**pouze na straně zobrazení** — záznam na serveru pokračuje. Pozastavené kamery nemohou nahrávat. |
| **X** | Odpojit. Uživatelské rozhraní se aktualizuje okamžitě (v optimálním případě); samotné odpojení na pozadí může trvat 10–30 s. |

### Řádky pole

Řádek pole zobrazuje štítek „ARRAY“ v barvě pole, název pole (lze přejmenovat v nastavení pole) a tlačítko **DAQ · zapnuto/vypnuto**—**zapnuto**, pokud je nastaven světelný senzor na úrovni pole *nebo* má některý člen senzor pro jednotlivé kamery; v jeho popisku je přesně uvedeno, který senzor co snímá. Kamery v poli jsou uvedeny odsazené pod ním ve vlastních řádcích. Tlačítka v řádku pole: **oko**(skryje/zobrazí VŠECHNY členy najednou),**ozubené kolečko**(panel nastavení pole),**X**(odpojí celé pole).

Stav světelného senzoru (DLS) používaný v řádcích pole a v panelu nastavení pole má čtyři stavy:**vypnuto**,**čeká**(zatím žádné spektrum),**aktivní**(spektrum dorazilo během posledních 3 s) a**neaktuální** — žádné nové spektrum za 3 s, ale poslední měření se *stále používá* (hodnoty DAQ na snímací cestě nikdy nevyprší).

Dokud nezměníte pořadí, seznam nejprve zobrazuje pole, poté samostatné kamery, přičemž kamery (a jednotlivé členy každého pole) jsou seřazeny v **pořadí kamer**: M3C (Bayer) před M3M (mono); mezi M3C, FRGB, FRGN, FOCN a poté FNGB; M3M podle vlnové délky filtru, nejprve nejnižší. Výchozí rozložení mřížky se řídí stejným pořadím. Samostatné kamery a celé skupiny polí můžete v postranním panelu přetahovat přes sebe a měnit tak pořadí v seznamu; členy pole nelze přetahovat samostatně.

## Panel nastavení pro jednotlivé kamery

Otevřete jej pomocí **ozubeného kolečka** v řádku kamery. Panel se vysune přes seznam kamer.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->



**Záhlaví**:**vzorník barev**kamery (kliknutím otevřete nativní výběr barev — nastavuje barvu ohraničení postranního panelu a výběrového kruhu dlaždic),**název**s tlačítkem**Přejmenovat**ve tvaru tužky (uložení prázdného názvu vrátí výchozí hodnotu `MODEL (serial)`) a**×** pro zavření.

### Živý histogram

V horní části panelu se nachází živý histogram jasu vypočítaný z náhledu ve formátu JPEG

při frekvenci ~8 Hz. Průměr je vážen podle Bayerova algoritmu — (R+2G+B)/4 — aby odpovídal vlastnímu měření automatické expozice (AE) kamery.

* **OranžováOrange

á přerušovaná čára**= cíl automatického měření expozice (AE).**Přetažením vodorovně můžete cíl změnit** — při uvolnění se odešle jeden příkaz a přetažením se režim cíle AE přepne na manuální.
* **ZelenáGreen

á plná čára** = skutečná průměrná jasová hodnota (to, co AE aktuálně poskytuje).
* **Tlačítko RGB** (vpravo nahoře): přepíná překryvné histogramy jednotlivých pásem zbarvené podle filtru fotoaparátu (např. u FRGN: šedá NIR

, zelená, červená). U monochromatických (M3M) fotoaparátů je na tlačítku nápis „MONO“ a je deaktivováno — u monochromatických fotoaparátů se vždy zobrazuje jednopásmový histogram jasu.
* Popisky na ose X se řídí bitovou hloubkou snímače aktuálního formátu pixelů: 0..255, 0..1023, 0..4095 nebo 0..65535.

### Řádky

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

s informacemi o kameře

| Řádek | Chování |
| --- | --- |
| **Model** | Pouze pro čtení (např. `LATT-M3C-L87-FRGN`). |
| **Radiometrická kalibrace** | Green

ý **„Aktivní“**štítek s popiskem zobrazujícím úroveň kalibrace, hash, datum kalibrace a seznam pásem, načtených z kalibračního balíčku kamery (viz [Tovární radiometrická kalibrace](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**U kamer RGB

skryto** — tyto kamery disponují fotometrickou kalibrací vyvážení bílé, nikoli radiancí pro jednotlivá pásma. |
| **Kalibrační zpráva**| Tlačítko**Stáhnout** — otevře PDF certifikát NIST o kalibraci kamery podle sériového čísla ve vašem prohlížeči v operačním systému. Pokud certifikát ještě není uložen v mezipaměti, Chloros

místo toho zobrazí nápovědu. |
| **Sériové číslo** | Pouze pro čtení. |
| **Firmware**| Zobrazuje aktuální verzi a následně vyhledá dostupnou verzi pro tento model (uloženo v mezipaměti pro každý model — pole N kamer provede kontrolu serveru pouze jednou). Stavy: „Kontroluje se…“ → tlačítko**„Aktualizovat na X“**→ „Aktualizace probíhá…“ → „Aktualizováno z A na B“ / „Selhalo: …“ / „Přeskočeno: …“ / zelené**„Aktuální“**. Popisek tlačítka aktualizace: „Obnovení továrního nastavení + flashování + přeprogramování UserSet1. ~2–3 minuty; neodpojujte.“ |
| **IP** | Pouze pro čtení. |
| **Teplota** | Pouze ke čtení, aktualizuje se každých 3 s. Zbarví se oranžově při ≥65 °C a červeně s ikonou ⚠ při ≥75 °C. |
| **Kalibrační cíl** zaškrtávací políčko | Zapíná detekci odrazivostního cíle ArUco s tabulkou ověření NDVI

pro jednotlivé panely pod živým přenosem (zobrazení seznamu) — viz [Tabulka kalibračních cílů](#the-calibration-target-table). Pouze pro danou relaci — vždy se otevírá v deaktivovaném stavu. |
| Rozbalovací nabídka **Světelný senzor** | Připojí světelný senzor DAQ (DAQ-E/M/U, ze seznamu na záložce Světelné senzory) k této kameře pro korekci osvětlení dopadajícím světlem (DLS) a prediktivní automatickou expozici. „„Žádný“ zruší přiřazení. Pokud nejsou připojeny žádné senzory, rozevírací seznam zobrazí „(žádné senzory připojeny — otevřete záložku DAQ)“. Přiřazení se ukládá spolu s projektem. |

#### Tabulka kalibračních terčů

Tabulka slouží **pouze k ověření** („pouze ověření — nemění kalibraci“). V záhlaví jsou uvedeny sériové číslo kamery, filtr, číslo terče a velikost značky v pixelech; tabulka obsahuje jeden řádek pro každý panel:

* **Panel**, poté jeden sloupec pro každé pásmo s hodnotou panelu — kamera typu Bayer zobrazuje svá tři pásma podle názvu a vlnové délky (např. FRGN: Red

660 nm / Green

550 nm / NIR

850 nm); **monochromatická (M3M) kamera zobrazuje své jediné pásmo** (např. `F750`) a nemá sloupce NDVI

.
* **Sat** — podíl pixelů panelu, které jsou nasycené (jantarová barva od 1 %, červená od 5 %).
* **NDVI

Meas**,**NDVI

Theo** (referenční NDVI

panelu na základě odrazivosti) a **Δ**— pouze u multispektrálních kamer typu Bayer. Malá značka za „NDVI

Meas“ udává, z čeho bylo naměřené NDVI

vypočítáno:**ρ**(odrazivost — je k dispozici údaj ze světelného senzoru),**L**(radiance — není k dispozici údaj ze světelného senzoru) nebo**DN**(surové hodnoty kamery). Pouze**ρ** NDVI

je srovnatelné s NDVI

Theo: u L a DN je sloupec Δ zašedlý a poznámka na to upozorňuje („Chybí údaj ze světelného senzoru: NDVI

je odvozeno ze zářivosti, takže Δ neslouží ke kontrole kalibrace“ / „NDVI

je odvozeno ze surových hodnot kamery, takže Δ neslouží ke kontrole kalibrace“). U hodnoty ρ je Δ zelené pod 0,02, oranžové pod 0,05, v ostatních případech červené.

<!-- SCREENSHOT-NEEDED: a 1.3.0 Calibration Target table under a live feed in list view — a Bayer camera's table showing the band columns, Sat, and "NDVI Meas ρ" / NDVI Theo / Δ; ideally next to a mono camera's single-band table with no NDVI columns. -->

### Expozice a zesílení

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Všechny číselné vstupy zde používají otočné ovladače s funkcí „podržet pro zrychlení“: klepnutí = ±1, podržení &gt;1,5 s = ±10, podržení &gt;3 s = ±100. Hodnota se odešle do kamery po uvolnění.

| Ovládací prvek | Rozsah / možnosti | Výchozí hodnota | Platí pro | Funkce |
| --- | --- | --- | --- | --- |
| **Expozice (us)**| Min./max. v reálném čase kamery | Auto | Vše | Doba expozice v mikrosekundách, s přepínačem**Auto/Ručně**přepínačem. Auto = nepřetržitá automatická expozice na straně kamery. Nastavení manuální expozice ponechá**Zisk** v režimu Auto, pokud v něm byl, a umožní volné využití celého rozsahu zisku kamery, dokud nezměníte nastavení zisku nebo jej nepřepnete do režimu Manuální. |
| **Zisk (dB)**| Aktuální min./max. hodnoty kamery (např. až 48 dB) | Ruční (vypnuto) | Vše | Analogový/digitální zisk s vlastním přepínačem**Auto/Ručně**. |
| **Cílová jasnost AE**| 0–255 | 80, režim**Auto**| Vše (upravitelné, je-li zapnuto AE nebo automatický zisk) | Jas, na který se AE zaměřuje. V režimu**Auto**(výchozí nastavení) si cílovou hodnotu vybírá sám řídicí modul na základě histogramu, přičemž expozici udržuje na 60–75 % maxima snímače. Zadáním hodnoty nebo přetažením oranžové čáry na histogramu se přepne do režimu**Ruční**. |
| **Vyhlazení AE** | 0,5–40, krok 0,1 | 8,0 | Vše | Tlumení AE. Popisek: „Nižší hodnota = AE reaguje rychleji (může pulzovat při vysokých snímkových frekvencích). Vyšší hodnota = plynulejší / pomalejší.“ Hodnoty výrazně nižší než výchozí mohou způsobit pulzování AE a destabilizovat streamování při vysokých snímkových frekvencích; 8,0 je stabilní výchozí hodnota. |
| **Oblast zájmu AE**| Zaškrtávací políčko „Povolit“ + tlačítko**Zamířit**| Vypnuto | Vše | Je-li zapnuto, AE měří pouze oblast vyznačenou zelenou čárkovanou čarou namísto celého snímku.**Aim** aktivuje funkci „kliknutím umístit“ v živém přenosu: kliknutím vycentrujete oblast na 30 % snímku; kliknutím a tažením nakreslíte vlastní obdélník (minimálně 5 % × 5 %). Funkce „Aim“ se po jednom umístění sama deaktivuje. Oblast se mapuje zpět do nativních souřadnic kamery podle jakékoli rotace/zrcadlení, které jste nastavili, a ukládá se spolu s projektem. |
| **Rychlost ladění AE** | 0,1–5, krok 0,1 | 1,0 | Pouze pro členy pole | Jak rychle cíl automatického AE sleduje změny jasu scény; při hodnotě 1,0× se provádí nová kontrola každých 2,5 s. |
| **Ochrana světlých oblastí** | Přísná (1 %) / Normální (5 %) / Uvolněná (15 %) | Přísná | Kamery, které toto nastavení podporují | Jak velká část snímku se může oříznout na bílou, než AE obraz ztmaví. |

{% hint style="info" %}
**Požadavky na osvětlení pro multispektrální kamery typu Bayer (RGN

/ OCN

/ NGB

):** scéna musí mít dostatek světla ve všech třech kanálech, jinak kalibrace nebude fungovat správně — jediná expozice snímače pokrývá všechna tři spektra. K měření osvětlení použijte světelný senzor DAQ nebo přepněte do režimu „all-mono“ (M3M), aby každé pásmo mělo vlastní expozici. Pokud snímek tuto podmínku poruší, Chloros

to detekuje a upozorní vás (oznámení o omezení oddělení). V reálném čase se v dlaždici s rozdělenými kanály zobrazí **„Data pásma s nízkou spolehlivostí · N %“**, pokud funkce unmix omezila toto pásmo na 25 % nebo více jeho pixelů.
{% endhint %}

{% hint style="info" %}
**Jak funkce automatické expozice využívá expozici a zisk (1.3.0).** Je-li zisk nastaven na „Auto“, Chloros

udržuje zisk na 0 dB, dokud expozice nedosáhne nejdelší povolené hodnoty (horní limit AE, spouštěcí perioda pole nebo strop expozice DAQ); teprve poté se zisk zvýší. V poli má přednost spouštěcí frekvence: automatická expozice vyplní období dané touto frekvencí, a teprve poté zvyšuje zisk, takže ztmavující se scéna již nesnižuje frekvenci spouštění v režimu Auto (ve verzi 1.2.0 tomu tak bylo). **Ruční**expozice na členském zařízení pole, která trvá déle než spouštěcí perioda, zpomalí spouštění pole tak, aby se tomu přizpůsobilo; automatická frekvence se obnoví, jakmile se každý prvek, který ji zpomalil, vrátí k automatické expozici. Pokud jsou parametry**Expozice**a**Cílová jasnost AE** nastaveny na Auto a světelný senzor DAQ poskytuje naměřené hodnoty, expozice každé kamery se udržuje po ustálení automatické expozice, dokud naměřené světlo zůstává v rozmezí přibližně ±25 %; větší a trvalá změna osvětlení, restart nastavení (například změna oblasti zájmu) nebo vaše vlastní změna expozice ji znovu aktivují.
{% endhint %}

{% hint style="info" %}
**Jak funguje automatická expozice (1.3.0).**

* Při ruční expozici zůstává zesílení v režimu Auto, dokud jej nezměníte.
* Malé světlé oblasti scénu neztmavují: odlesky do 1 % snímku jsou ignorovány a u panelů známého kalibračního terče až 0,5 % panelu.
* Horní limit expozice DAQ platí pouze tehdy, je-li v zorném poli kalibrační terč; není-li žádný panel vidět, kamera exponuje podle scény.
* Expozice zůstává stabilní při konstantním osvětlení a ustálí se během několika sekund po výrazné změně. Přesvětlení již trvale neomezuje expozici a změna oblasti zájmu (ROI) znovu nastaví správnou expozici.
* Cílová hodnota jasu nepřekračuje dosah kamery, takže návrat světla nezpůsobí přeexponování snímku.
* Kamera, jejíž automatická expozice přestane reagovat na přímém slunci, je udržována na úrovni odpovídající aktuálnímu osvětlení, které snímá světelný senzor, nebo – pokud není připojen – vlastní snímky kamery, a uvolní se, jakmile se světlo změní. Tmavý nebo zakrytý světelný senzor tento proces nenarušuje.
{% endhint %}

### Formát pixelů a rozlišení

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

**Členové pole** zobrazují řádky „Aktuální“ (formát + ŠxV) a „Binning“ pouze pro čtení s poznámkou „Nastaveno v nastavení pole“ — restart streamu u jednoho člena by narušil synchronizaci, proto jsou tyto parametry spravovány v [panelu nastavení pole](#array-settings-pane).**Samostatné kamery** mají:

| Ovládací prvek | Možnosti | Funkce |
| --- | --- | --- |
| **Formát pixelů** | 8-bit / 10-bit / 12-bit / 16bitový | Hloubka bitů snímače; formát odpovídá formátu snímače. Výchozí hodnota je 12 bitů a projekt uloží nastavenou hodnotu, včetně 8 bitů, a při opětovném připojení se k ní vrátí. |
| **Rozlišení** | Plné / Poloviční / Čtvrtinové | V závislosti na aktuálním binningu: Plné = 2048/N × 1536/N pro binning N×N. |
| **Binning** | 1×1 (žádný) / 2×2 / 4×4 | Hardwarové N×N binning — vyšší hodnoty snižují rozlišení, ale zvyšují poměr signálu k šumu (SNR) a snímkovou frekvenci. Jeho změna restartuje datový proud a resetuje všechny oblasti zájmu (ROI) na nové plné zorné pole. |
| **Aktuální** | pouze pro čtení | Skutečné rozměry Š×V a posun (x, y), které jsou právě aktivní. |

### Živý náhled

Vše v této sekci se týká **pouze zobrazení**— mění to, co vidíte v živém přenosu, zatímco uložené snímky zůstávají lineární a nezměněné — s jednou výjimkou:**Vignette** je radiometrická a ovlivňuje i exporty (viz níže).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->



| Ovládací prvek | Rozsah / možnosti | Výchozí | Platí pro | Co dělá |
| --- | --- | --- | --- | --- |
| **Rozlišení renderování** | 360p (nejrychlejší) / 480p / 720p / 1080p / Nativní rozlišení snímače (nejpomalejší) | 720p | Vše | Výška, ve které backend spouští radiometrický řetězec náhledů. Nižší hodnota zajišťuje vyšší snímkovou frekvenci bez změny zorného pole. |
| **Index**| Zaškrtávací políčko „Povolit“ + ozubené kolečko | Vypnuto | Pouze multispektrální snímače typu Bayer,**ne** členy kombinovaných matic | Živý náhled vegetačního indexu. Kliknutím na ozubené kolečko se otevře sdílený panel [Index Calculator](#index-calculator-pane), který je předem načten s přirozenými pásmy filtrů kamery (např. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Vlastní výraz spolu s LUT (zapnuto/vypnuto, výchozí úroveň 7 zastavení, výchozí minimum 0,2, výchozí maximum 1) se vypočítává u každého snímku náhledu. Členové kombinovaného pole tento řádek skryjí — pole vlastní jeden sdílený index. |
| **Vyvážení bílé** | Vypnuto / Jednou / Kontinuálně + tlačítko pro nové zachycení | Kontinuálně | Pouze RGB

| Vyvážení bílé v reálném čase. Tlačítko pro obnovení znovu zachytí vyvážení bílé z aktuálního spektra DLS (vypnuto, pokud je režim Vypnuto). |
| **Gamma** | Zapnuto / Vypnuto | Zapnuto | Pouze RGB

| Zobrazí gammu (γ = 2,2 LUT) v živém náhledu. Uložené snímky zůstávají lineární. |
| **Odšumění** | Zaškrtávací políčko + síla 0–100 | Vypnuto / 50 | Vše (pro každou kameru, i uvnitř polí) | Bilaterální filtr v živém náhledu. Vyšší hodnota = hladší, ale méně ostré detaily. |
| **Ostrost** | Zaškrtávací políčko + síla 0–100 | Vypnuto / 30 | Vše | Maska neostrosti v živém náhledu, aplikovaná jako poslední. Může zesílit šum. Pouze v náhledu. |
| **Vigneta**| Zaškrtávací políčko + síla 0–100 | Vypnuto / 0 | Vše | Ruční odstranění zbytkové vignety (zesvětluje rohy), vrstvené nad odhadem inteligentní vignety pole.**Radiometrické — ovlivňuje živý náhled I exporty**, na rozdíl od odšumění a ostrosti. |
| **Barevný profil** | Raw / Lineární / Přirozený / Vylepšený / Vlastní teplota | Přirozený | Pouze RGB

| Viz níže. |
| **Barevná teplota** | 2000–10000 K, krok 100 | 5500 K | Pouze RGB

, vlastní profil teploty | Upevňuje vyvážení bílé na pevnou korelovanou barevnou teplotu (vstup DLS je ignorován). Poslední zvolená hodnota v kelvinech se uchovává i při přepínání profilů. |
| **Světlé oblasti** | −100 až 0, krok 1 | 0 (vypnuto) | Pouze RGB

| Novinka ve verzi 1.3.0. Ztmavuje nejjasnější a nejbělejší oblasti obrázku (0 = beze změny, −100 = nejsilnější). Platí pro živý náhled **a pro snímky na úrovni displeje (Náhled)**, které toto nastavení zaznamenávají (`Chloros:DisplayHighlights`); exporty radiance a odrazivosti nejsou nikdy ovlivněny. Nelze tím obnovit oříznutá data snímače — v takovém případě snižte expozici. Zebra overlay se přizpůsobí ztmavenému obrazu. Deaktivováno s barevným profilem Raw („Nepoužívá se s barevným profilem Raw.“). |
| **Sytost** | 0–200 (100 = neutrální) | 100 | Pouze RGB

| Sytost HSV v živém náhledu. |
| **Kontrast** | 0–200 (100 = neutrální) | 100 | Pouze RGB

| Lineární kontrast kolem střední šedé v živém náhledu. |
| **Zrcadlit vodorovně / Zrcadlit svisle** | Zaškrtávací políčka | Vypnuto | Vše | Otočit náhled vodorovně / svisle. |
| **Otočení**| 0° / 90° / 180° / 270° | 0° | Vše | Otočit náhled. Orientace se aplikuje na konci řetězce náhledů na straně backendu.**Uložené snímky (1.3.0):**produkty samostatného fotoaparátu (Preview, Radiance, Reflectance a Index) se otáčejí pomocí funkcí Zrcadlení/Otočení, zatímco**formáty Raw a Debayered zůstávají v nativní orientaci kamery**; každý soubor snímku zaznamenává toto nastavení do svého XMP, takže při pozdějším zpracování formátu Raw se toto nastavení aplikuje stejným způsobem.**Soubory v poli zůstávají v nativní orientaci** (nastavení je zaznamenáno, ale není aplikováno) a kompozitní pohledy pole jej ignorují. |**Sémantika barevných profilů** (kamery RGB

):

* **Raw** — zcela obejde zpracovatelský řetězec.
* **Lineární** — tmavý signál + vyrovnání pole + vyvážení bílé; bez barevné matice, bez gama.
* **Natural** *(výchozí)* — lineární plus naměřená matice barevné korekce a tónová křivka přizpůsobená scéně.
* **Enhanced**— Natural plus vibrance a lokální kontrast CLAHE. Dodatečné náklady se vztahují**pouze na živý náhled** — uložené snímky vždy získají plnou úpravu bez ohledu na profil.
* **Vlastní teplota** — režim „Přirozený“ s vyvážením bílé nastaveným na vámi zvolenou hodnotu v kelvinech.

{% hint style="warning" %}
U režimů „Přirozený“, „Vylepšený“ a „Vlastní teplota“ se v panelu zobrazuje poznámka k tónům: snímky jsou zesvětleny podle konkrétní scény, takže uložené *zobrazené* obrázky nelze porovnávat mezi jednotlivými snímky. **Pro měření exportujte radianci nebo odrazivost.**
{% endhint %}

### Zobrazovací překryvy (kreslené přes živý přenos)

Jedná se o prvky pouze v uživatelském rozhraní — jsou nakresleny přes video, nikdy neovlivňují živý přenos ani pořízené snímky.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Překryv | Ovládací prvky | Výchozí nastavení | Funkce |
| --- | --- | --- | --- |
| **Zebra** | Zaškrtávací políčko + prahová hodnota 200–255 | Vypnuto / 250 | Magentové diagonální pruhy na oříznutých pixelech. |
| **Mířidla** | Zaškrtávací políčko | Vypnuto | Značka středu snímku. |
| **Mřížka** | Vypnuto / 3 × 3 / 9 × 9 | Vypnuto | Kompoziční mřížka. |
| **Histogram** | Zaškrtávací políčko + šířka 0,10–0,90 snímku | Vypnuto / 0,25 | Pruh histogramu přímo ve streamu. |
| **Focus Peak** | Zaškrtávací políčko + prahová hodnota 20–200 + vzorek barvy | Vypnuto / 80 / `#ff5722` | Sobelovo zvýraznění hran pro zaostřování. |
| **Rozdělení kanálů** | Tlačítko „Zobrazit rozdělení (Red

/ Green

/ NIR

)“ / Tlačítko „Skrýt rozdělení“ | Skryto | Přidá tři nezávislé dlaždice ve stupních šedi pro každý kanál vedle kompozitního obrazu (označení tlačítek odpovídá filtračním kanálům fotoaparátu). Každou dlaždici lze přetahovat a má barvu ohraničení fotoaparátu. Není k dispozici u monochromatických fotoaparátů. Uloží se s projektem. |

### Bodový expozimetr

* Zaškrtávací políčko **Kliknutím odeberte vzorek**: kliknutím na živý obraz odeberete vzorek z jednoho pixelu (označí jej křížová mřížka) nebo kliknutím a tažením vyberete oblast pro výpočet průměru pixelů. Tlačítko**Vymazat**vymaže vzorek a mřížku. Tato funkce se vzájemně vylučuje s režimem**Aim** v AE-ROI.
* **Rozbalovací nabídka**Zobrazit**:**Raw (bitová hloubka)**— nativní digitální hodnoty v bitové hloubce snímače (např. 12 bitů → 0..4095) — nebo**Zobrazení (8 bitů)** (výchozí). Je-li aktivní živý index, zobrazí se v poli „Display“ místo toho vypočítaná hodnota indexu (např. NDVI

).
* Panel s údaji uvádí souřadnice pixelů, velikost snímku, formát pixelů, bitovou hloubku a tabulku kanálů (Chan / Hodnota / %) s označením pásem a vlnovými délkami; páry zelených pixelů podle Bayerova systému jsou zprůměrovány; vzorky z oblasti zobrazují „N px avg“.

Stav bodového měřiče platí pouze pro danou relaci.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Prediktivní automatická expozice (řízená DLS)

Tato sekce se zobrazí pouze v případě, že **je připojen alespoň jeden světelný senzor DAQ** — řešič potřebuje k řízení živé spektrum dopadajícího slunečního záření.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->



| Ovládací prvek | Rozsah | Výchozí | Funkce |
| --- | --- | --- | --- |
| **Povolit** | Zaškrtávací políčko | Zapnuto (samostatné kamery) | Řešič uzavřeného typu využívá spektrum DLS a skalární hodnoty z kalibračního balíčku kamery k tomu, aby nejjasnější pásmo dosáhlo hodnoty blížící se saturaci, zatímco nejtmavší pásmo zůstane nad minimální hodnotou SNR — jeden zápis expozice na jedno řešení, bez stabilizační smyčky. Určeno pro časosběrné snímání napájené solární energií, kde musí být každý snímek správně exponován. Backend automaticky přepne na reaktivní automatickou expozici (AE), pokud je hodnota DLS zastaralá/chybí nebo není načten kalibrační balíček. |
| **Vyhlazení (α)** | 0,05–1,0, krok 0,05 | 0,3 | Vyhlazení po sobě jdoucích prediktivních řešení (nižší hodnota = větší vyhlazení). |
| **Odrazivost scény**| Tlačítko**Rekalibrovat ρ** | — | Znovu odhaduje faktor odrazivosti scény, který řešič používá. |

{% hint style="info" %}
**Propojení pole ve výchozím nastavení vypíná prediktivní AE** — u polí se o expozici starají inteligentní AE od Chloros

a automatická expozice na straně fotoaparátu (s ochranou proti přesycení), přičemž odhad odrazivosti pro jednotlivé scényodhad odrazivosti není bezpečný v případě smíšených scén. Zde ji můžete znovu povolit pro jednotlivé kamery, pokud konkrétně požadujete radiometrickou expozici řízenou DLS.
{% endhint %}

**Strop expozice řízený DAQ a AE vázaná na dopadající světlo.** Nezávisle na výše uvedeném zaškrtávacím políčku, pokud je světelný senzor DAQ přiřazen k RGB

kameře, Chloros

vypočítá — na základě naměřeného absolutního ozáření směřujícího dolů — maximální expozici × zisk, při kterém povrch se 100 % odrazivostí zůstane pod hranicí oříznutí, a použije ji jako **horní hranici**automatické expozice. Od verze 1.3.0 se tato horní hranice uplatňuje pouze tehdy, je-li kalibrační cíl v zorném poli dané kamery (zachycen v jejím záběru během posledních 10 s); není-li cíl v zorném poli, běží místo toho běžná automatická expozice kamery s vlastní ochranou proti přepálení, takže zbytek scény není udržován tmavší, než je nutné. Zatímco je limit aktivní, kamera je**v režimu fixace na dopadající světlo**: pracuje v režimu otevřené smyčky s expozicí měřenou podle dopadajícího světla a ziskem nastaveným na 0 dB — expozice se řídí naměřeným světlem, nikoli obsahem scény. Jelikož horní limit může expozici pouze zkrátit, nemůže sám o sobě způsobit oříznutí. Od verze 1.3.0 se horní limit počítá pomocí vlastní expoziční konstanty kamery z jejího kalibračního balíčku (průměrná hodnota flotily pouze v případě, že balíček žádnou neobsahuje). Horní limit se automaticky deaktivuje – a obnoví se normální automatická expozice scény – vždy, když není v zorném poli žádný kalibrační cíl, chybí údaj z DAQ, je zastaralý (&gt;30 s), nebo je tmavý, nebo pokud ≥15 % snímku vyběhne při pevně nastavené expozici (což znamená, že snímač a kamera vnímají odlišné osvětlení). Neexistuje žádný přepínač v grafickém rozhraní; jedná se o standardní chování vždy, když je k RGB

kameře připojen DAQ.

### Snímání a spouštění

Členové<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

pole navíc zobrazují řádky **Role**(Master v modré / Slave v zelené),**Sync Line**a**Peers**, které jsou pouze pro čtení.

| Ovládací prvek | Možnosti | Výchozí | Poznámky |
| --- | --- | --- | --- |
| **Režim spouštění** | Vypnuto / Zapnuto | Zapnuto | U prvků pole je tato možnost zakázána (spouštění řídí pole). |
| **Zdroj spouštění** | Software / Linka 0 (M8) / Linka 1 / Linka 2 | Linka 0 | Skryto, pokud je režim spouštění vypnutý; u členů pole je tato možnost deaktivována. Linka 0 je opticky izolovaný externí spouštěcí vstup M8. |
| **Snímková frekvence**| Auto / Ručně + hodnota | Auto |**Auto**: omezení snímkové frekvence kamery je vypnuté — expozice určuje počet snímků za sekundu (fps) a v poli se zobrazuje aktuální snímková frekvence v reálném čase.**Ručně**: počet snímků za sekundu omezíte pomocí posuvníku (v rozmezí 1 až po maximum dané šířkou pásma), přičemž výchozí hodnota vychází z aktuální skutečné snímkové frekvence. U členů pole se zobrazuje hodnota „N fps (živě)“ pouze pro čtení s poznámkou „Nastaveno v nastavení pole“. |

### Síť / Přenos

| Řádek | Chování |
| --- | --- |
| **Velikost paketu**| 1500 (standardní) / 9000 (Jumbo) — výchozí hodnota**Jumbo**. |
| **Propustnost** | Limit propustnosti spojení v MB/s, pouze pro čtení. Backend tuto hodnotu při každém připojení/odpojení přerozdělí mezi všechny připojené kamery. |
| **Zpracování vyrovnávací paměti** | Režim zpracování vyrovnávací paměti, pouze pro čtení. |

### Snímání

Panel končí tlačítkem **„Otevřít nastavení snímání…“**, které přejde na [panel Nastavení snímání](capture.md#the-capture-settings-pane) (deaktivováno, dokud není otevřen projekt — „Vytvořte nebo otevřete projekt pro uložení snímků“). Pokud je kamera skrytá nebo pozastavená, zobrazí se upozornění, abyste ji před snímáním odkryli/obnovili.

## Panel nastavení pole

Otevřete pomocí **ozubeného kolečka**v řádku POLE. Záhlaví: název pole s tužkou pro přejmenování a**×** pro zavření. Sekce níže označené *pouze kombinované* se zobrazují pouze u polí připojených v kombinovaném režimu zobrazení.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->



### Synchronizace

Řádky **Master**,**Slaves**a**Sync Line** jsou pouze pro čtení.

### Senzor okolního světla

Zobrazuje se u kombinovaných i samostatných polí:

* Rozbalovací nabídka **Light Sensor**— přiřadí jeden DAQ k celému poli. Výběr se okamžitě projeví, přenese se do rozevíracího seznamu**Senzor osvětlení** u každé kamery v poli (nastavení lze u jednotlivých kamer stále přepsat) a začne předávat spektra do pole.
* Řádek **Stav** v reálném čase: Vypnuto · „Čekání na první spektrum…“ · „Aktivní — u všech kamer v poli je provedena korekce osvětlení“ · „Žádné nové spektrum za poslední 3 s — stále se používá poslední měření (žádný časový limit pro neaktuální data)…“.
* Poznámka v panelu: „Radiometrická korekce pro celé pole. Nastavení jednotlivých kamer má přednost.“

### Kalibrační cíl

Vlastní skupina přímo za položkou Senzor okolního osvětlení, a to jak pro kombinovaná, tak pro samostatná pole:

* Zaškrtávací políčko **Povolit** — „Detekovat cíl MAPIR

ArUco a ověřit LUT pro NDVI

vs. odrazivost panelu“; ovládá překryv cíle a ověřovací tabulky kombinované dlaždice. U **samostatného** pole zapíná vlastní [tabulku kalibračního cíle](#the-calibration-target-table) (jedna pro každý řádek kamery v zobrazení seznamu).

### Snímání — jednotná nastavení senzorů

Zobrazuje se jak pro kombinovaná, tak pro samostatná pole (nadpis skupiny zní „Snímání (jednotné pro všechny kamery)“). Tato nastavení se jednotně vztahují na všechny členy (změny pro jednotlivé členy by narušily synchronizaci). Úpravy se shromažďují a aplikují společně pomocí tlačítka **Použít**— s výjimkou přepínače**ROI zapnuto / ROI vypnuto**, který se aplikuje okamžitě.

<!-- SCREENSHOT-NEEDED: array settings Capture section (1.3.0) — Pixel Format, Binning, Resolution preset, the ROI crop row with the "ROI on" toggle, the "Select" button, the "max WxH" hint and Reset button, the W/H/X/Y fields, the full-sensor ROI map below them with the orange crop box and its "W×H @ X,Y" label above the box, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons. -->

| Ovládací prvek | Možnosti / rozsah | Co dělá |
| --- | --- | --- |
| **Formát pixelu** | 8bitový / 10bitový / 12bitový / 16bitový | Jednotná bitová hloubka pro všechny členy; každá kamera použije formát této hloubky pro svůj vlastní snímač. |
| **Binning** | 1x1 / 2x2 / 4x4 | Hardwarové binning — zachovává plné zorné pole a zároveň zvyšuje poměr signálu k šumu (SNR) a snímkovou frekvenci. Jeho změna resetuje pole ROI na nové plné zorné pole. |
| Předvolba **rozlišení** | Plné / Poloviční / Čtvrtinové | V závislosti na binningu; vyplní pole ROI oříznutým výřezem se středovým bodem. |
| **Ořez ROI (px)**| Přepínač**ROI zapnuto / vypnuto**, tlačítko**Vybrat**, pole Š / V / X / Y, mapa celého snímače | Ořez snímače — viz [Editor ROI](#the-roi-editor). Šířka/výška se přichytávají k násobkům 16 (minimálně 64); posuny se zarovnávají na násobky 4. Nápověda „max WxH“ ukazuje horní hranici a tlačítko**Reset** vrátí pole do plného zorného pole. Během úprav oranžový ořezový rámeček sleduje každou změnu pole na mapě i na dlaždici pole. |
| **Frekvence spouštění**| Přepínač Auto / Ručně + fps 0,5–10, krok 0,5 |**Auto**(výchozí): backend odvozuje frekvenci spouštění z rozlišení a šířky pásma — vstup je deaktivován a zobrazuje odvozenou hodnotu.**Ruční**: při stisknutí tlačítka**Použít** se hodnota zafixuje. Zafixovaná frekvence, kterou nová velikost snímku nedokáže zvládnout, se omezí na nejvyšší frekvenci, kterou daná velikost umožňuje, a obnoví se, jakmile to velikost snímku opět dovolí. |

Poznámka v panelu: „Změny formátu/rozlišení krátkodobě restartují všechny kamery. Frekvence spouštění se aplikuje v reálném čase.“ Tlačítka **Použít / Zrušit** se nacházejí ve spodní části panelu.

#### Editor ROI

Novinka ve verzi 1.3.0. Řádek oříznutí ROI a mapa pod ním fungují společně:

* **ROI zapnuto / ROI vypnuto**—**okamžitě**zapne nebo vypne oříznutí snímače (kamery se na krátkou dobu restartují; dlaždice pole zobrazuje „Používání nastavení…“).**Vypnuto**načte celý snímač a**zapamatuje si vaše oříznutí**— pole jej nadále zobrazují;**Zapnuto** jej znovu použije. Zapnutí oblasti zájmu bez nastaveného oříznutí použije vycentrované oříznutí na poloviční velikost („Nebylo nastaveno žádné oříznutí, proto bylo použito vycentrované oříznutí na poloviční velikost. Upravte jej pomocí polí.“).
* **Vybrat**— nakreslí ořez na mapě: přetáhněte kurzor přes požadovanou oblast nebo klikněte na dva protilehlé rohy (klávesa Esc zruší akci). Obdélník se zapíše do polí W/H/X/Y, přizpůsobí se podle výše uvedených kroků; nic se nepoužije, dokud nestisknete**Použít**, takže jej můžete nejprve doladit pomocí šipek v polích. Jedno stisknutí tlačítka Vybrat vytvoří jeden obdélník.
* **Mapa**zobrazuje oříznutí na celém snímači, takže jej můžete znovu rozšířit: „Celý snímač, živý obraz“, zatímco kamery čtou celý snímač, nebo „Snímek v plném formátu z {čas}. Kamery snímají pouze oříznutou oblast“, zatímco jsou oříznuté. Pokud pole dosud streamovalo pouze oříznuté snímky, mapa zobrazuje „Zatím žádný náhled v plném formátu: toto pole streamovalo pouze oříznuté snímky.“ s tlačítkem**Zobrazit plný snímek**, které vypne oblast zájmu (ROI). Štítek oříznutí `W×H @ X,Y` se nachází mimo rámeček (nad ním, nebo pod ním, pokud se rámeček dotýká horního okraje), na mapě i na živé dlaždici.
* **Použít**potvrdí nastavení polí: oříznutí zapne oblast zájmu (ROI); plný snímek (po**Reset**) ji nechá vypnutou, přičemž si systém stále pamatuje předchozí oříznutí; oblast zájmu, která je vypnutá a jejíž pole stále zobrazují zapamatované oříznutí, zůstane vypnutá i po použití další změny (formát, binning, snímková frekvence).
* **U zarovnaného pole je ořezové okno každého člena umístěno podle zarovnání**, takže každá kamera zabírá stejnou část scény. Pole oříznuté ještě před zarovnáním se nejprve zarovná na celý snímač (ořez se zruší, vyřeší se zarovnání a ořez se obnoví s okny pro jednotlivé kamery).
* Po změně oříznutí se automatická expozice znovu přizpůsobí novému záběru. I při oříznutí může expozice člena trvat až do konce spouštěcího období.
* Stav zapnutí/vypnutí oblasti zájmu (ROI) a zapamatované oříznutí se ukládají spolu s projektem.

### Zarovnání (ko-registrace)

Zobrazeno pro **kombinovaná i samostatná pole** (verze 1.2.0 to zobrazovala pouze pro kombinovaná pole); záhlaví skupiny nese název „Alignment“.

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* **Zbytková** hodnota: „RMS x,xx px“ — zelená při hodnotě pod 1 px, oranžová při hodnotě pod 3 px, červená v ostatních případech nebo pokud selhala některá z kamer; „žádný profil“ před prvním výpočtem.
* Souhrnný řádek: „✓ Všechny kamery zarovnány (N)“ / „⚠ p/N kamer zarovnáno — <serial (filter)="">selhání“ / „Oříznutí aktivní — překalibrujte pro zarovnání (využívá celý snímač)“ / „Čekání na ustálení expozice…“ / „Aplikace nastavení…“ (kamery se restartují) / „Systém přestal dodávat snímky — znovu připojte systém“.
* Tabulka pro jednotlivé kamery: kamera (poslední 4 znaky sériového čísla + filtr), chyba reprojekce v px s počtem shod („ref“ pro hlavní kameru) a skóre normalizované křížové korelace překryvu vzhledem k minimálnímu prahu 0,35.
* **Rekalibrovat zarovnání** (před prvním profilem se zobrazuje „Kalibrovat zarovnání“) — znovu spustí ko-registraci na nových snímcích.
* **Zaškrtávací políčko „Automaticky exponovat kamery pro zarovnání“** (výchozí nastavení: zaškrtnuto) — dočasně zesvětlí tmavé nebo ploché kamery (nejprve expozice, poté zisk), aby měly texturu pro porovnání, a poté obnoví automatickou expozici.

Chloros automaticky zarovná každé pole po připojení, a to v obou režimech zobrazení, jakmile se ustálí automatická expozice jednotlivých kamer (pokouší se o to až 5 minut); kombinovaný náhled se také zarovná při otevření. Samostatné pole nemá kompozitní dlaždici, takže tato část zjišťuje stav zarovnání a sama spouští funkci „Recalibrate“. Proveďte rekalibraci, pokud se změnilo zaostření nebo hloubka scény. Zarovnání je **záměrně omezeno pouze na danou relaci** — nikdy se neukládá do profilu, protože závisí na aktuální vzdálenosti scény. Snímky lze i nadále exportovat s pixelovou registrací (viz [Vyrovnané exporty](capture.md#per-array-controls)).

### Inteligentní vinětace

* Zaškrtávací políčko **Povolit korekci**— aplikuje odhad vinětace pro jednotlivé kamery na radiometrický řetězec (v reálném čase**i** při exportu).
* **Kalibrovat z aktuálního pohledu**— nejprve nasměrujte pole na jednotný cíl (plochý panel, zeď nebo oblohu); každá kamera je vyrovnána individuálně a stav hlásí „n/N kamer · −x,x %“ zlepšení rovinnosti. Tlačítko**Vymazat** odstraní odhad.
* Stav: „Kalibrováno“, „Nekalibrováno“ nebo „Částečně kalibrováno — spustit znovu“, pokud se v aktuálním záběru stále uplatňují odhady pouze u některých kamer (například poté, co funkce „Rekalibrovat“ posunula ořezové okno některé kamery) — spusťte kalibraci znovu.
* Proveďte jemné doladění pro každou kameru pomocí posuvníku **Vignette** pro jednotlivé kamery v [Živém náhledu](#live-preview).

### Živý náhled

* Zaškrtávací políčko **Histogram**+ posuvník šířky (0,10–0,90 snímku) — pruh histogramu v živém náhledu na dlaždici každého člena najednou (včetně dlaždic s rozdělenými kanály), [Překrytí histogramem](#display-overlays-drawn-over-the-live-feed) nastavené pro celé pole. Zobrazuje se jako zapnuté, pokud jej již mají všichni členové. Na**samostatném** poli je to jediný řádek ve skupině.
* *Pouze v kombinaci:* **Index**: zaškrtávací políčko + ozubené kolečko — otevře sdílenou [kalkulačku indexu](#index-calculator-pane) s pásmy nakreslenými ze**všech** kamer členů. Řádek s náhledem výrazu pod ním zobrazuje aktuální výraz („Není nastaven žádný výraz — otevřete kalkulačku a vytvořte jej“), který se aktualizuje každou sekundu.
* *Pouze v kombinovaném režimu:* rozevírací nabídka **Rozlišení renderování**(stejné předvolby jako u jednotlivých kamer, výchozí hodnota 720p): výška živého přenosu**a** velikost uloženého kompozitního exportu. Poznámka v panelu: „Náhled + velikost uloženého kompozitu. Snímky z jednotlivých kamer se vždy exportují v plném rozlišení.“

### Zobrazovací vrstvy *(pouze v kombinovaném režimu)** Zaškrtávací políčko **Povolit** (výchozí nastavení vypnuto — hlavní kamera se zobrazuje přímo; zapnuto = vrstvený kompozit).
* Rozbalovací nabídky **Rozbalovací nabídky**Popředí**/**Pozadí**: jednotlivé členské kamery (podle názvu) nebo**Index**. Je-li v poli Popředí vybrána možnost Index, pixely mimo rozsah LUT Min/Max zobrazují vrstvu Pozadí.

### Rozdělené zobrazení *(pouze kombinované)*

**„Zobrazit členské kamery“**— tlačítko**Rozdělit / Skrýt členské kamery**, které přidá živý přenos každé členské kamery jako samostatné dlaždice mřížky vedle kompozice. Dlaždice čtou stávající rámcový buffer pole (bez nutnosti připojení další kamery). Pouze v mřížkovém zobrazení; ukládá se pro každé pole společně s projektem.

### Funkce

Panel pouze pro čtení, aktualizovaný každých 5 s:

* **Označení úrovně**: „Souběžné snímání“ (zelená) · „Souběžné snímání (FTD – střídavé vysílání)“ (zelená) · „Střídavé snímání (100 ms zpoždění)“ (oranžová) · „Konfigurace příliš velká“ (červená).
* **Stav snímku**: „x,xx % neúplné“ — zelená pod 1 %, oranžová pod 5 %, červená při 5 % a více.
* ****Linková linka**: „NIC {mbps} Mbps – trvalý {MB/s} MB/s“.

Toto je aktuální rozpočet šířky pásma pole. Informace o základních hodnotách fps a síťovém modelu — a o tom, co je třeba změnit, když se úroveň změní na oranžovou nebo červenou — najdete v části [Sestavy s více kamerami](arrays.md) a v [Referenční příručce CLI](../reference/cli-reference.md).

<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->

## Panel Kalkulátor indexu

Třetí stránka postranního panelu, společná pro nástroj Index pro jednotlivé kamery a nástroj Index pro kombinované pole (vždy pouze jeden – záhlaví zní „Kalkulátor indexu — <camera name="">“ nebo „Kalkulátor indexu — <array name="">“). Zde se zobrazuje seznam pásem (přirozená pásma filtrupřirozená pásma kamery nebo všechna pásma napříč členy pole), aktuální výraz a konfiguraci LUT (zapnuto/vypnuto, úroveň – výchozí 7 zastavení, min. – výchozí 0,2, max. – výchozí 1), plus živý histogram indexu. Tlačítko **Použít** potvrdí výraz; změny LUT se v reálném čase promítají do náhledu.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Nastavení pro jednotlivé kamery vs. nastavení spravovaná polem

Stručný přehled toho, co se kde nachází, když je kamera členem pole:

| Spravováno polem (v panelu kamery pouze pro čtení) | Stále pro jednotlivé kamery uvnitř pole |
| --- | --- |
| Formát pixelů, rozlišení, binning | Automatická expozice (expozice, zesílení, cíl, vyhlazení, ROI) |
| Režim/zdroj spouštění, snímková frekvence | Odšumování, ostrost, vinětace |
| | Orientace (zrcadlení/otočení), překryvy na displeji (nastavení pole → živý náhled nastaví histogram na všech členech najednou), bodové měření |
| | Index (pole se samostatným zobrazením), přiřazení světelného senzoru |

Další průřezové chování:

* **Kombinované vs. samostatné zobrazení** se volí při připojení pole: kombinované = jedna zarovnaná složená dlaždice (členy vysílají pouze přes Split View); samostatné = každý člen vykresluje svou vlastní synchronizovanou dlaždici. Kamera nikdy nezobrazuje současně samostatný přenos i dlaždici pole.
* ****Automatické opětovné připojení**: otevření uloženého projektu obnoví jeho kamery a pole a znovu aplikuje všechna uložená nastavení na backend před obnovením přenosů.
* **Omezení snímání**: skryté nebo pozastavené kamery jsou vyloučeny z funkce „Capture All“; pole je zcela zablokováno pouze tehdy, jsou-li VŠECHNY členy skryté/pozastavené. Viz [Nastavení a režimy snímání](capture.md).

## Jak se nastavení uchovávají

Stav karty kamery se ukládá **spolu s projektem**, nikoli v prohlížeči:

* Každá reaktivní změna pořídí snímek kamer a polí do souboru `cameras.json` projektu (s odskokem 500 ms). To zahrnuje názvy a barvy kamer, nastavení expozice/zisku/AE, formát pixelů/rozlišení/binning, frekvenci spouštění, nastavení náhledu (rozlišení renderování, odšumování, ostrost, vinětace, barevný profil, světla, sytost/kontrast), orientaci, překryvy, rozdělení kanálů, konfiguraci indexu, nastavení prediktivního AE, AE ROI, názvy polí, režim zobrazení, nastavení snímání polí (včetně oříznutí oblasti zájmu, zda je oblast zájmu zapnutá a zapamatované oříznutí, když je vypnutá), rozpočet vodičů pole a blok mřížky (zoom přenosu, režim zobrazení, uzamčení mřížky, ruční pořadí dlaždic, skryté kamery, uzavřené dlaždice, aktivní kamera).
* Přiřazení světelných senzorů se ukládají do souboru projektu `sensors.json`.
* Při opětovném otevření projektu dojde k opětovnému připojení hardwaru a k opětovnému použití všech těchto nastavení.
* **Žádný otevřený projekt = pouze relace**: bez projektu se po zavření programu Chloros nic neuloží.
* Pouze pro danou relaci bez ohledu na projekt: stav pozastavení, vzorky bodového měření, zaškrtávací políčko Kalibrační cíl pro jednotlivé kamery (vždy se otevírá v nezaškrtnutém stavu) a profil zarovnání pole (záměrně přepočítávaný pro každou relaci).
* Jedna výjimka: výběry pro export v části **Nastavení snímání** a režim snímání zůstávají uloženy pro každý projekt v lokálním úložišti aplikace, nikoli v `cameras.json` — viz [Nastavení a režimy snímání](capture.md).</array></camera></serial>
