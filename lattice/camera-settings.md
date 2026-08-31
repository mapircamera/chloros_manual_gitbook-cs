# Nastavení kamer

Záložka **Kamery**představuje živý ovládací panel aplikace Chloros pro kamery LATTICE: hlavní oblast s živým přenosem, kde se každá připojená kamera zobrazuje jako živá dlaždice, a postranní panel, který umožňuje přepínání mezi třemi stránkami —**seznam kamer**,**panel nastavení**(nastavení pro jednotlivé kamery, pole kamer nebo snímání – vždy jedno najednou) a**Index Calculator**. Tato stránka popisuje všechny ovládací prvky v seznamu kamer, v panelu nastavení pro jednotlivé kamery a v panelu nastavení pole. Režimy snímání, výběr typu exportu a postup „Capture All“ jsou popsány na související stránce [Nastavení a režimy snímání](capture.md).

Karta „Kamery“ se zobrazí v postranním panelu, jakmile je backend Chloros připraven. Všechny níže uvedené ovládací prvky komunikují s lokálním backendem přes `127.0.0.1:5000`; změny se okamžitě projeví na živém obrazu z kamery, není-li uvedeno jinak.

## Typy kamer používané na této stránce

Ovládací prvky se zobrazují nebo skryjí v závislosti na tom, jaký typ kamery je vybrán. V této příručce se používají následující termíny:

| Termín | Význam | Kanály filtru |
| --- | --- | --- |
| **Kamera RGB** | LATTICE M3C s filtrem FRGB (model obsahuje `-FRGB`) | Red / Green / Blue |
| **Multispektrální snímač Bayer** | LATTICE M3C s FRGN, FOCN nebo FNGB | FRGN: Red / Green / NIR · FOCN: Orange / Cyan / NIR · FNGB: NIR / Green / Blue |
| **Mono (M3M)** | LATTICE M3M — jeden úzkopásmový filtr, jedno kalibrované pásmo | Jedno pásmo |
| **Člen pole** | Kamera připojená jako součást synchronizovaného pole (kombinované nebo samostatné zobrazení) | Podle svého filtru |

Kamery RGB procházejí fotometrickým zpracováním (vyvážení bílé, barevné profily, gama); multispektrální a monochromatické kamery procházejí radiometrickým řetězcem a přeskočí fotometrické ovládací prvky. Členové pole předávají nastavení na úrovni datového proudu (formát pixelů, rozlišení, binning, spouštění, snímková frekvence) do pole — tyto řádky se v panelu pro jednotlivé kamery stanou pouze pro čtení a přesunou se místo toho do panelu nastavení pole.

## Hlavní oblast

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

přenosu Pokud nejsou připojeny žádné kamery, zobrazuje se v oblasti přenosu úvodní obrazovka s textem **„Připojte kameru a začněte“**a dvěma tlačítky:**Připojit kameru**(zelené, otevře dialogové okno pro připojení jedné kamery) a**Připojit pole** (modré, otevře dialogové okno pro připojení pole). Samotná dialogová okna pro připojení jsou popsána v [Připojování kamer](connecting.md); pojmy týkající se pole (synchronizace, úrovně, šířka pásma) v [Pole s více kamerami](arrays.md). Při otevření uloženého projektu, který obsahuje kamery, se na úvodní obrazovce místo toho zobrazí ikona načítání s textem „Znovu otevírám N uložených kamer…“, zatímco Chloros obnovuje streamy z poslední relace.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Horní lišta

| Ovládací prvek | Funkce |
| --- | --- |
| **Přepínání režimu zobrazení**| Přepíná mezi**mřížkovým zobrazením**(všechny dlaždice jako buňky) a**seznamovým zobrazením** (pole v plné šířce nahoře, JEDNA aktivní kamera dole). Nápovědy: „Přepnout na mřížkové zobrazení“ / „Přepnout na seznamové zobrazení“. |
| **Zámek mřížky**(visací zámek) | Výchozí nastavení**zamčeno** — dlaždice jsou zafixovány na místě. Odemkněte zámek a přetahováním dlaždic je můžete přesunout do libovolného slotu (mezery zůstanou zachovány). Mřížka se automaticky znovu zamkne pokaždé, když se připojí nová kamera. Popisky: „Odemknout mřížku (povolit přetahování dlaždic)“ / „Zamknout mřížku (zamknout dlaždice na místě)“. |
| Posuvník **Zvětšení kanálu** | Velikost dlaždice, od 60 px až po plnou šířku kontejneru. Buňky si zachovávají poměr stran 4:3. Při šířce buňky pod 200 px se překryvy s názvem a fps skryjí, aby dlaždice zůstala přehledná. |

### Dlaždice živého přenosu

Každá kamera vykresluje složenou živou dlaždici; kamera může navíc zobrazovat tři dlaždice ve stupních šedi s **rozdělením podle kanálů** (viz [Rozdělení kanálů](#display-overlays-drawn-over-the-live-feed)) a pole vykreslují kombinovanou dlaždici. Aktivní dlaždice má výběrový prstenec v barvě kamery (nebo pole).

Při najetí myší na dlaždici se zobrazí tlačítko pro zavření **X**:

* Zavření **složené** dlaždice, zatímco její rozdělené kanály zůstávají viditelné, pouze skryje složenou dlaždici.
* Zavření **posledního viditelného panelu samostatné kamery** odpojí danou kameru.
* **Rozdělené panely, které jsou součástí kombinovaného pole, nikdy kameru neodpojí** — pouze ji skryjí.

Je-li mřížka odemčena, přetáhněte libovolný panel na libovolné místo; rozložení se uloží spolu s projektem.

## Postranní panel — seznam

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

kamer První stránka postranního panelu obsahuje seznam všech připojených kamer a sestav:

* **Připojit kameru**(zelené) /**Připojit sestavu** (modré, během skenování se zobrazuje „Detekuji...“). Obě tlačítka jsou deaktivována, pokud je otevřeno dialogové okno pro připojení.
* **Zachytit vše** (červené) — zachytí všechny uvedené kamery s typy exportu vybranými v Nastavení zachycení. Vyžaduje otevřený projekt. Podrobně popsáno v [Nastavení a režimy zachycení](capture.md).
* **Ozubené kolečko Nastavení snímání** (vedle tlačítka Zachytit vše) — otevře [panel Nastavení snímání](capture.md#the-capture-settings-pane). Je deaktivováno, pokud není otevřen projekt nebo během snímání.

### Řádky kamer

Každý řádek kamery zobrazuje barevně odlišený okraj (vlastní barva kamery), označení „CAM“ — s modrým písmenem **M**(master) nebo zeleným písmenem**S** (slave) pro členy pole — a zobrazovaný název. Výchozí název je `LATTICE-MODEL (serial)`; přejmenujte jej v panelu nastavení jednotlivých kamer. Tlačítka v řádku:

| Tlačítko | Účinek |
| --- | --- |
| **Oko**| Přepíná viditelnost. Skryté kamery zmizí z mřížky a jsou**vyloučeny z funkce Zachytit vše**. |
| **Ozubené kolečko** | Otevře panel nastavení pro jednotlivé kamery (další část). |
| **Pozastavit / Přehrát**| Zastaví živý náhled**pouze na straně zobrazení** — záznam na serveru pokračuje. Pozastavené kamery nemohou nahrávat. |
| **X** | Odpojit. Uživatelské rozhraní se aktualizuje okamžitě (v optimálním případě); samotné odpojení na serverové straně může trvat 10–30 s. |

### Řádky pole

Řádek pole zobrazuje štítek „ARRAY“ v barvě pole, název pole (lze přejmenovat v nastavení pole) a **DAQ · zapnuto/vypnuto**—**zapnuto**, pokud je nastaven světelný senzor na úrovni pole *nebo* má některý člen senzor pro jednotlivou kameru; jeho popisek uvádí, který senzor přenáší signál kam. Kamery jednotlivých členů jsou uvedeny odsazené pod ním ve vlastních řádcích. Tlačítka v řádku pole: **oko**(skryje/zobrazí VŠECHNY členy najednou),**ozubené kolečko**(panel nastavení pole),**X**(odpojí celé pole).

Stav světelného senzoru (DLS) používaný v řádcích pole a v panelu nastavení pole má čtyři stavy:**vypnuto**,**čeká**(zatím žádné spektrum),**aktivní**(spektrum dorazilo během posledních 3 s) a**zastaralé** — žádné nové spektrum za poslední 3 s, ale poslední hodnota se *stále používá* (hodnoty DAQ nikdy nevyprší na cestě zachycení).

Samostatné kamery a celé skupiny polí můžete v postranním panelu přetahovat přes sebe a měnit tak pořadí seznamu; jednotlivé členy pole nelze přetahovat samostatně.

## Panel nastavení pro jednotlivé kamery

Otevřete jej pomocí **ozubeného kolečka** v řádku kamery. Panel se posune přes seznam kamer.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->

**Záhlaví**:**barevný vzorník**kamery (kliknutím otevřete nativní výběr barev — nastavuje barvu ohraničení postranního panelu a výběrového kruhu dlaždic),**název**s tlačítkem**Přejmenovat**ve tvaru tužky (při uložení prázdného názvu se vrátí výchozí název `MODEL (serial)`) a**×** pro zavření.

### Živý histogram

V horní části panelu se nachází živý histogram jasu vypočítaný z náhledu JPEG při frekvenci ~8 Hz. Průměr je vážen podle Bayerova algoritmu — (R+2G+B)/4 — aby odpovídal vlastnímu měření expozice kamery.

* **Orange přerušovaná čára**= cíl automatického měření expozice.**Přetažením vodorovně lze změnit cíl** — při uvolnění se odešle jeden příkaz a přetažením se režim cíle automatického nastavení expozice přepne na manuální.
* **Green plná čára** = skutečná průměrná hodnota jasu (to, co v daném okamžiku poskytuje automatické nastavení expozice).
* ****Tlačítko RGB** (vpravo nahoře): přepíná překryvné histogramy jednotlivých pásem, jejichž barva je dána filtrem fotoaparátu (např. u FRGN: šedá NIR, zelená, červená). U monochromatických (M3M) kamer se na tlačítku zobrazuje „MONO“ a je deaktivováno — u monochromatických kamer se vždy zobrazuje jednopásmový histogram jasu.
* Popisky na ose X se řídí bitovou hloubkou snímače aktuálního formátu pixelů: 0..255, 0..1023, 0..4095 nebo 0..65535.

### Řádky

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

s informacemi o kameře | Řádek | Chování |
| --- | --- |
| **Model** | Pouze ke čtení (např. `LATT-M3C-L87-FRGN`). |
| **Radiometrická kalibrace**| Green**„Aktivní“**s popiskem uvádějícím úroveň kalibrace, hash, datum kalibrace a seznam pásem, načtenými z kalibračního balíčku kamery (viz [Tovární radiometrická kalibrace](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Skryto u kamer RGB** — tyto kamery mají fotometrickou kalibraci vyvážení bílé, nikoli radianci pro jednotlivá pásma. |
| **Kalibrační zpráva**| Tlačítko**Stáhnout** — otevře PDF certifikát NIST o kalibraci kamery podle sériového čísla v prohlížeči vašeho operačního systému. Pokud certifikát ještě není uložen v mezipaměti, Chloros místo toho zobrazí nápovědu. |
| **Sériové číslo** | Pouze pro čtení. |
| **Firmware**| Zobrazuje aktuální verzi a následně vyhledá dostupnou verzi pro tento model (uloženo v mezipaměti pro každý model — pole N kamer provede kontrolu serveru pouze jednou). Stavy: „Kontroluje se…“ → tlačítko**„Aktualizovat na X“**→ „Aktualizace probíhá…“ → „Aktualizováno z A na B“ / „Selhalo: …“ / „Přeskočeno: …“ / zelené**„Aktuální“**. Popisek tlačítka aktualizace: „Obnovení továrního nastavení + flashování + přeprogramování UserSet1. ~2–3 minuty; neodpojujte.“ |
| **IP** | Pouze pro čtení. |
| **Teplota** | Pouze pro čtení, aktualizuje se každé 3 s. Zbarví se do oranžova při ≥65 °C a do červena s ⚠ při ≥75 °C. |
| Zaškrtávací políčko **Kalibrační cíl** | Povoluje detekci reflektančního cíle ArUco pomocí validační tabulky NDVI pro jednotlivé panely pod živým přenosem (zobrazení seznamu). Pouze pro danou relaci — vždy se otevírá v deaktivovaném stavu. |
| Rozbalovací nabídka **Rozbalovací nabídka**Světelný senzor** | Připojí světelný senzor DAQ (DAQ-E/M/U, ze seznamu na záložce Světelné senzory) k této kameře pro korekci osvětlení sestupujícím světlem (DLS) a prediktivní automatickou expozici. Volba „Žádný“ zruší přiřazení. Pokud nejsou připojeny žádné senzory, rozevírací nabídka zobrazí „(žádné připojené senzory — otevřete záložku DAQ)“. Přiřazení se uloží spolu s projektem. |

### Expozice a zesílení

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Všechny číselné vstupy zde používají otočné ovladače s akcelerací při podržení: klepnutí = ±1, podržení &gt;1,5 s = ±10, podržení &gt;3 s = ±100. Hodnota se odešle do kamery po uvolnění.

| Ovládací prvek | Rozsah / možnosti | Výchozí hodnota | Vztahuje se na | Funkce |
| --- | --- | --- | --- | --- |
| **Expozice (us)**| Minimální/maximální hodnoty kamery v reálném čase | Auto | Vše | Doba expozice v mikrosekundách, s přepínačem**Auto/Ručně**. Auto = nepřetržitá automatická expozice na straně kamery. |
| **Zisk (dB)**| Minimální a maximální hodnoty kamery v reálném čase (např. až 48 dB) | Ruční (vypnuto) | Vše | Analogový/digitální zisk s vlastním přepínačem**Auto/Ruční**. |
| **Cílová jasnost AE**| 0–255 | 80, režim**Auto**| Vše (upravitelné, je-li zapnuto AE nebo automatické zesílení) | Jas, na který se AE zaměřuje. V režimu**Auto**(výchozí nastavení) si cílovou hodnotu vybírá sám backendový regulátor na základě histogramu, přičemž expozici udržuje na 60–75 % maxima snímače. Zadáním hodnoty nebo přetažením oranžové čáry na histogramu se přepne do režimu**Manuální**. |
| **Vyhlazení AE** | 0,5–40, krok 0,1 | 8,0 | Vše | Tlumení AE. Popisek: „Nižší hodnota = AE reaguje rychleji (může pulzovat při vysokých snímkových frekvencích). Vyšší hodnota = plynulejší / pomalejší.“ Hodnoty výrazně nižší než výchozí mohou způsobit pulzování AE a destabilizovat streamování při vysokých snímkových frekvencích; 8,0 je stabilní výchozí hodnota. |
| **Oblast zájmu AE**| Zaškrtávací políčko „Zapnout“ + tlačítko**Zamířit**| Vypnuto | Vše | Je-li zapnuto, AE měří pouze zelenou přerušovanou oblast namísto celého snímku.**Aim** aktivuje funkci „kliknutím umístit“ na živém přenosu: kliknutím vycentrujete oblast na 30 % snímku; kliknutím a tažením nakreslíte vlastní obdélník (minimálně 5 % × 5 %). Funkce „Aim“ se po jednom umístění sama deaktivuje. Oblast se mapuje zpět do nativních souřadnic kamery podle jakékoli rotace/zrcadlení, které jste nastavili, a ukládá se spolu s projektem. |
| **Rychlost ladění AE** | 0,1–5, krok 0,1 | 1,0 | Pouze pro členy skupiny | Jak rychle cíl automatického AE sleduje změny jasu scény; při hodnotě 1,0× se provádí kontrola každých 2,5 s. |
| **Ochrana světlých oblastí** | Přísná (1 %) / Normální (5 %) / Volná (15 %) | Přísná | Kamery, které toto nastavení podporují | Jak velká část snímku se může oříznout na bílou, než AE obraz ztmaví. |

{% hint style="info" %}
**Požadavky na osvětlení pro multispektrální kamery typu Bayer (RGN / OCN / NGB):** scéna musí mít dostatek světla ve všech třech kanálech, jinak kalibrace nebude fungovat správně — jediná expozice snímače pokrývá všechna tři spektra. K měření osvětlení použijte světelný senzor DAQ nebo přepněte do režimu „all-mono“ (M3M), aby každé pásmo mělo vlastní expozici. Pokud snímek tuto podmínku poruší, Chloros to detekuje a upozorní vás (oznámení „unmix-clamp“).
{% endhint %}

### Formát pixelů a

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

rozlišení**Členové pole** zobrazují řádky „Current“ (formát + ŠxV) a „Binning“ pouze pro čtení s poznámkou „Nastaveno v nastavení pole“ — restart streamu u jednoho člena by narušil synchronizaci, proto se tyto parametry spravují v [panelu nastavení pole](#array-settings-pane).**Samostatné kamery** mají:

| Ovládací prvek | Možnosti | Funkce |
| --- | --- | --- |
| **Formát pixelů** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Formát pixelů snímače (bitová hloubka). |
| **Rozlišení** | Plné / Poloviční / Čtvrtinové | V závislosti na aktuálním binningu: Plné = 2048/N × 1536/N pro binning N×N. |
| **Slučování** | 1x1 (žádné) / 2x2 / 4x4 | Hardwarové N×N slučování — vyšší hodnoty snižují rozlišení, ale zvyšují poměr signálu k šumu (SNR) a snímkovou frekvenci. Jeho změna restartuje stream a resetuje všechny oblasti zájmu (ROI) na nové plné zorné pole. |
| **Aktuální** | pouze pro čtení | Skutečné hodnoty Š × V a posun (x, y), které jsou právě aktivní. |

### Živý náhled

Všechno v této sekci se týká **pouze zobrazení**— mění to, co vidíte v živém přenosu, zatímco uložené snímky zůstávají lineární a nezměněné — s jednou výjimkou:**Vigneta** je radiometrická a ovlivňuje i exporty (viz níže).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->

| Ovládací prvek | Rozsah / možnosti | Výchozí | Platí pro | Co dělá |
| --- | --- | --- | --- | --- |
| **Rozlišení renderování** | 360p (nejrychlejší) / 480p / 720p / 1080p / Nativní rozlišení snímače (nejpomalejší) | 720p | Vše | Výška, při které backend spouští radiometrický řetězec náhledů. Nižší hodnota zvyšuje snímkovou frekvenci beze změny zorného pole. |
| **Index**| Zaškrtávací políčko + ozubené kolečko | Vypnuto | Pouze multispektrální snímače typu Bayer,**ne** členy kombinovaných matic | Živý náhled vegetačního indexu. Ozubené kolečko otevře sdílený panel [Index Calculator](#index-calculator-pane) předem načtený s přirozenými pásmy filtrů kamery (např. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Vlastní výraz plus LUT (zapnuto/vypnuto, výchozí úroveň 3, výchozí minimum 0,2, výchozí maximum 1) se počítá u každého snímku náhledu. Členové kombinovaného pole tento řádek skryjí — pole vlastní jeden sdílený index. |
| **Vyvážení bílé** | Vypnuto / Jednou / Kontinuální + tlačítko pro nové zachycení | Kontinuální | Pouze RGB | Vyvážení bílé v reálném čase. Tlačítko pro obnovení znovu zachytí vyvážení bílé z aktuálního spektra DLS (vypnuto, pokud je režim Vypnuto). |
| **Gamma** | Zapnuto / Vypnuto | Zapnuto | Pouze RGB | Zobrazení gama (γ = 2,2 LUT) v živém náhledu. Uložené snímky zůstávají lineární. |
| **Odšumění** | Zaškrtávací políčko + síla 0–100 | Vypnuto / 50 | Vše (pro každou kameru zvlášť, i uvnitř polí) | Bilaterální filtr v živém náhledu. Vyšší hodnota = hladší, ale méně ostré detaily. |
| **Ostrost** | Zaškrtávací políčko + síla 0–100 | Vypnuto / 30 | Vše | Maska neostrosti v živém náhledu, aplikovaná jako poslední. Může zesílit šum. Pouze v náhledu. |
| **Vigneta**| Zaškrtávací políčko + síla 0–100 | Vypnuto / 0 | Vše | Ruční odstranění zbytkové vignety (zesvětluje rohy), vrstvené nad odhadem inteligentní vignety pole.**Radiometrické — ovlivňuje živý náhled I export**, na rozdíl od odšumování a ostrosti. |
| **Barevný profil** | Raw / Lineární / Přirozený / Vylepšený / Vlastní teplota | Přirozený | Pouze RGB | Viz níže. |
| **Barevná teplota** | 2000–10000 K, krok 100 | 5500 K | Pouze RGB, profil Vlastní teplota | Upevňuje vyvážení bílé na pevnou korelovanou barevnou teplotu (vstup DLS je ignorován). Poslední zvolená hodnota v Kelvinech se uchovává i při přepínání profilů. |
| **Sytost** | 0–200 (100 = neutrální) | 100 | Pouze RGB | Sytost HSV v živém náhledu. |
| **Kontrast** | 0–200 (100 = neutrální) | 100 | Pouze RGB | Lineární kontrast kolem střední šedé v živém náhledu. |
| **Zrcadlit H / Zrcadlit V** | Zaškrtávací políčka | Vypnuto | Všechny | Otočí náhled vodorovně / svisle. |
| **Otočení**| 0° / 90° / 180° / 270° | 0° | Vše | Otočí náhled. Orientace se aplikuje na konci řetězce náhledů na straně backendu —**uložené snímky zůstávají v nativní orientaci fotoaparátu** a kompozitní zobrazení pole ji ignorují. |**Sémantika barevných profilů** (kamery RGB):

* **Raw** — zcela obejde řetězec zpracování.
* **Lineární** — tmavý signál + vyrovnání pole + vyvážení bílé; bez barevné matice, bez gama.
* **Přirozený** *(výchozí)* — lineární plus naměřená matice barevné korekce a tónová křivka přizpůsobená scéně.
* **Vylepšený**— Přirozený plus živost a lokální kontrast CLAHE. Příplatek se vztahuje**pouze na živý náhled** — uložené snímky vždy procházejí kompletním zpracováním bez ohledu na profil.
* **Vlastní teplota** — Natural s vyvážením bílé nastaveným na vámi zvolenou hodnotu v kelvinech.

{% hint style="warning" %}
U režimů „Přirozený“, „Vylepšený“ a „Vlastní teplota“ se v panelu zobrazuje poznámka k tónům: snímky jsou zesvětleny podle dané scény, takže uložené *zobrazené* obrázky nelze porovnávat snímek po snímku. **Pro měření exportujte radianci nebo odrazivost.**
{% endhint %}

### Zobrazovací překryvy (kreslené přes živý přenos)

Jedná se o prvky pouze v uživatelském rozhraní — jsou nakresleny přes video, nikdy neovlivňují stream ani pořízené snímky.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Překryv | Ovládací prvky | Výchozí nastavení | Funkce |
| --- | --- | --- | --- |
| **Zebra** | Zaškrtávací políčko + prahová hodnota 200–255 | Vypnuto / 250 | Magentové diagonální pruhy na oříznutých pixelech. |
| **Záměrník** | Zaškrtávací políčko | Vypnuto | Značka středu snímku. |
| **Mřížka** | Vypnuto / 3 × 3 / 9 × 9 | Vypnuto | Kompoziční mřížka. |
| **Histogram** | Zaškrtávací políčko + šířka 0,10–0,90 snímku | Vypnuto / 0,25 | Pás histogramu při přenosu snímků. |
| **Focus Peak** | Zaškrtávací políčko + prahová hodnota 20–200 + vzorek barvy | Vypnuto / 80 / `#ff5722` | Sobelovo zvýraznění hran pro zaostření. |
| **Rozdělení kanálů** | „Zobrazit rozdělení (Red / Green / NIR)“ / Tlačítko „Skrýt rozdělení“ | Skryto | Přidá tři nezávislé dlaždice ve stupních šedi pro každý kanál vedle kompozitního obrazu (označení tlačítek odpovídá filtračním kanálům kamery). Každou rozdělenou dlaždici lze přetahovat a má barvu ohraničení kamery. Není k dispozici u černobílých kamer. Uloží se s projektem. |

### Bodový expozimetr

* Zaškrtávací políčko **Kliknutím odeberte vzorek**: kliknutím na živý obraz odeberete vzorek z jednoho pixelu (označí se křížkem) nebo kliknutím a tažením vyberete oblast pro výpočet průměru pixelů. Tlačítko**Vymazat**vymaže vzorek a křížek. Tato funkce se vzájemně vylučuje s režimem**Zamíření** AE-ROI.
* Rozbalovací nabídka **Zobrazit**:**Raw (bitová hloubka)**— nativní digitální hodnoty v bitové hloubce snímače (např. 12 bitů → 0..4095) — nebo**Zobrazení (8 bitů)** (výchozí). Je-li aktivní živý index, zobrazí se v poli Display namísto toho vypočítaná hodnota indexu (např. NDVI).
* Panel s údaji uvádí souřadnice pixelů, velikost snímku, formát pixelů, bitovou hloubku a tabulku kanálů (Chan / Hodnota / %) s označením pásem a vlnovými délkami; páry zelených pixelů podle Bayerova schématu jsou zprůměrovány; vzorky z dané oblasti zobrazují „N px avg“.

Stav bodového měřiče platí pouze pro danou relaci.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Prediktivní automatická expozice (řízená DLS)

Tato sekce se zobrazí pouze v případě, že **je připojen alespoň jeden světelný senzor DAQ** — řešič potřebuje živé spektrum dopadajícího záření k jejímu řízení.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->

| Ovládací prvek | Rozsah | Výchozí nastavení | Funkce |
| --- | --- | --- | --- |
| **Povolit** | Zaškrtávací políčko | Zapnuto (samostatné kamery) | Řešitel uzavřeného typu využívá spektrum DLS a skalární hodnoty z kalibračního balíčku kamery k tomu, aby nejjasnější pásmo dosáhlo téměř saturace, zatímco nejtmavší pásmo zůstane nad minimální hodnotou SNR — jeden zápis expozice na jedno řešení, bez stabilizační smyčky. Určeno pro časosběrné snímání napájené solární energií, kde musí být každý snímek správně exponován. Backend se tiše přepne na reaktivní automatickou expozici (AE), kdykoli je hodnota DLS zastaralá/chybí nebo není načten kalibrační balíček. |
| **Vyhlazení (α)** | 0,05–1,0, krok 0,05 | 0,3 | Vyhlazení po sobě jdoucích prediktivních řešení (nižší hodnota = hladší výsledek). |
| **Odrazivost scény**| Tlačítko**Rekalibrovat ρ** | — | Znovu odhaduje faktor odrazivosti scény, který řešič používá. |

{% hint style="info" %}
**Funkce „Array connect“ ve výchozím nastavení vypíná prediktivní automatickou expozici** — u maticových sestav se automatická expozice řídí funkcí „Smart AE plus“ kamery Chloroszajišťuje automatickou expozici (s ochranou proti přesycení) a jediný odhad odrazivosti scény prediktivní AE není bezpečný u smíšených scén. Zde ji můžete znovu povolit pro jednotlivé kamery, pokud konkrétně požadujete radiometrickou expozici řízenou DLS.
{% endhint %}

**Strop expozice řízený DAQ a AE vázaná na dopadající světlo.**Nezávisle na výše uvedeném zaškrtávacím políčku, pokud je světelný senzor DAQ přiřazen k kameře RGB, Chloros vypočítá — na základě naměřeného absolutního ozáření směřujícího dolů — maximální expozici × zesílení, při kterém povrch se 100 % odrazivostí zůstane pod hranicí oříznutí, a použije jej jako**horní limit**pro automatickou expozici. Dokud je tento limit aktivní, kamera je**v režimu fixace na dopadající světlo**: pracuje v režimu otevřené smyčky s expozicí měřenou podle dopadajícího světla a ziskem nastaveným na 0 dB — expozice se řídí naměřeným světlem, nikoli obsahem scény. Jelikož horní limit může expozici pouze zkrátit, nemůže sama o sobě způsobit oříznutí. Horní limit se automaticky deaktivuje – a obnoví se normální automatická expozice scény – vždy, když chybí hodnota z DAQ, je zastaralá (&gt;30 s) nebo tmavá, nebo pokud ≥15 % snímku je oříznuto při fixované expozici (což znamená, že snímač a kamera vnímají odlišné osvětlení). Neexistuje žádný přepínač v grafickém uživatelském rozhraní; jedná se o standardní chování vždy, když je kamera RGB propojena s DAQ.

### Členové pole Acquisition &amp; Trigger

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

navíc zobrazují řádky **Role**(Master v modré barvě / Slave v zelené barvě),**Sync Line**a**Peers**, které jsou pouze pro čtení.

| Ovládací prvek | Možnosti | Výchozí | Poznámky |
| --- | --- | --- | --- |
| **Režim spouštění** | Vypnuto / Zapnuto | Zapnuto | Zakázáno pro členy pole (spouštění řídí pole). |
| **Zdroj spouštění** | Software / Line0 (M8) / Line1 / Line2 | Line0 | Skryto, pokud je režim spouštění vypnutý; u členů pole je tato možnost zakázána. Linka 0 je opticky izolovaný externí spouštěcí vstup M8. |
| **Snímková frekvence**| Auto / Ručně + hodnota | Auto |**Auto**: omezení snímkové frekvence kamery je vypnuté — expozice určuje počet snímků za sekundu (fps) a pole zobrazuje aktuální živou frekvenci.**Ručně**: omezení počtu snímků za sekundu nastavíte pomocí posuvníku (v rozmezí 1 až po maximum dané šířkou pásma), přičemž výchozí hodnotou je aktuální snímková frekvence. Členové pole vidí pouze pro čtení „N fps (živě)“ s poznámkou „Nastaveno v nastavení pole“. |

### Síť / Přenos

| Řádek | Chování |
| --- | --- |
| **Velikost paketu**| 1500 (Standard) / 9000 (Jumbo) — výchozí hodnota**Jumbo**. |
| **Propustnost** | Limit propustnosti spojení v MB/s, pouze pro čtení. Backend tuto hodnotu při každém připojení/odpojení přerozdělí mezi všechny připojené kamery. |
| **Zpracování vyrovnávací paměti** | Režim zpracování vyrovnávací paměti, pouze pro čtení. |

### Snímání

Panel končí tlačítkem **„Otevřít nastavení snímání…“**, které přejde na [panel Nastavení snímání](capture.md#the-capture-settings-pane) (deaktivováno, dokud není otevřen projekt — „Vytvořte nebo otevřete projekt, abyste mohli ukládat snímky“). Pokud je kamera skrytá nebo pozastavená, upozornění vám připomene, abyste ji před snímáním odkryli nebo obnovili.

## Panel nastavení pole

Otevřete pomocí **ozubeného kolečka**v řádku POLE. Záhlaví: název pole s tužkou pro přejmenování a**×** pro zavření. Níže uvedené části označené *pouze kombinované* se zobrazují pouze u polí připojených v kombinovaném režimu zobrazení.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->

### Synchronizace

Řádky **Master**,**Slaves**a**Sync Line** jsou pouze pro čtení.

### Senzor okolního světla

Zobrazuje se u kombinovaných i samostatných polí:

* Zaškrtávací políčko **Kalibrační cíl** — „Detekovat cíl ArUco MAPIR a ověřit NDVI proti LUT odrazivosti panelu“; řídí překryv cíle a ověřovací tabulku kombinované dlaždice.
* Rozbalovací nabídka **Snímač osvětlení** — přiřadí jeden DAQ k celému poli. Výběr se okamžitě uloží, promítne se do rozbalovací nabídky Snímač osvětlení u každé kamery v poli (stále můžete provést přepsání u jednotlivých kamer) a začne předávat spektra do pole.
* Řádek **Stav** v reálném čase: Vypnuto · „Čekání na první spektrum…“ · „Aktivní — u všech kamer v poli je provedena korekce osvětlení“ · „Žádné nové spektrum za poslední 3 s — stále se používá poslední měření (žádný časový limit pro neaktuální data)…“.
* Poznámka v panelu: „Radiometrická korekce pro celé pole. Nastavení jednotlivých kamer mají přednost.“

### Snímání — jednotná nastavení snímače *(pouze kombinovaná)*

Tato nastavení se jednotně vztahují na všechny členy pole (změny u jednotlivých členů by narušily synchronizaci). Úpravy se shromažďují a aplikují společně.

<!-- SCREENSHOT-NEEDED: array settings Capture section — Pixel Format, Binning, Resolution preset, the ROI crop W/H/X/Y fields with the "max WxH" hint and Reset button, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons; ideally with the live orange crop-preview box visible on the array tile. -->

| Ovládací prvek | Možnosti / rozsah | Funkce |
| --- | --- | --- |
| **Formát pixelů** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Jednotný formát snímače pro všechny členy. |
| **Binning** | 1x1 / 2x2 / 4x4 | Hardwarové binning — zachovává plné zorné pole a zároveň zvyšuje poměr signálu k šumu (SNR) a snímkovou frekvenci. Jeho změna resetuje pole ROI na nové plné zorné pole. |
| **Předvolba rozlišení** | Plné / Poloviční / Čtvrtinové | V závislosti na binningu; vyplní pole ROI oříznutým výřezem se středovým bodem. |
| **Ořez ROI (px)**| Číselná pole Š / V / X / Y | Ořez snímače. Šířka/výška se přichytávají k násobkům 16 (minimálně 64); posuny se přichytávají k násobkům 4. Nápověda „max WxH“ ukazuje horní hranici a tlačítko**Reset** vrátí nastavení na plné zorné pole. Během úprav se na dlaždici pole vykresluje živý oranžový rámeček s náhledem oříznutí (včetně schématu celého snímače při rozšiřování oříznutí směrem ven). |
| **Frekvence spouštění**| Auto / Ruční přepínač + fps 0,5–10, krok 0,5 |**Auto**(výchozí): backend odvozuje frekvenci spouštění z rozlišení a šířky pásma — zadávání je deaktivováno a zobrazuje odvozenou hodnotu.**Ruční**: při stisknutí tlačítka Použít se hodnota uzamkne. |

Poznámka v panelu: „Změny formátu/rozlišení krátkodobě restartují všechny kamery. Frekvence spouštění se aplikuje v reálném čase.“ Tlačítka **Použít / Zrušit** se nacházejí ve spodní části panelu.

### Zarovnání (korelace) *(pouze v kombinaci)*

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* Políčko **Zbytková chyba**: „RMS x,xx px“ — zelená barva při hodnotě pod 1 px, oranžová při hodnotě pod 3 px, červená v ostatních případech nebo pokud selhala některá z kamer; „žádný profil“ před prvním vyřešením.
* Souhrnný řádek: „✓ Všechny kamery zarovnány (N)“ / „⚠ p/N kamer zarovnáno — <serial (filter)="">selhalo“ / „Ořez aktivní — překalibrujte pro zarovnání (využívá celý snímač)“ / „Čeká se na ustálení expozice…“.
* Tabulka pro jednotlivé kamery: kamera (poslední 4 znaky sériového čísla + filtr), chyba reprojekce v px s počtem shod („ref“ pro hlavní kameru) a skóre normalizované křížové korelace překryvu vzhledem k minimálnímu prahu pro úspěšnost 0,35.
* Tlačítko **„Rekalibrovat zarovnání“** (před prvním profilem se zobrazuje „Kalibrovat zarovnání“ před prvním profilem) — znovu spustí ko-registraci na nových snímcích.
* Zaškrtávací políčko **„Automaticky exponovat kamery pro zarovnání“** (výchozí nastavení zaškrtnuto) — dočasně zesvětlí tmavé nebo ploché kamery (nejprve expozice, poté zisk), aby měly texturu pro porovnání, a poté obnoví automatickou expozici.

Kombinovaný náhled se při otevření automaticky zarovná; v případě změny zaostření nebo hloubky scény proveďte rekalibraci. Zarovnání je **záměrně omezeno pouze na danou relaci** — nikdy se neukládá do profilu, protože závisí na aktuální vzdálenosti scény. Snímky lze i nadále exportovat s pixelovou registrací (viz [Vyrovnané exporty](capture.md#per-array-controls)).

### Inteligentní vinětace

* Zaškrtávací políčko **Povolit korekci**— aplikuje odhad vinětace pro jednotlivé kamery na radiometrický řetězec (v reálném čase**i** při exportu).
* **Kalibrovat z aktuálního pohledu**— nejprve nasměrujte pole na jednotný cíl (plochý panel, zeď nebo oblohu); každá kamera je vyrovnána individuálně a stav hlásí zisk rovinnosti „n/N kamer · −x,x %“. Tlačítko**Vymazat** odstraní odhad.
* Proveďte jemné doladění pro každou kameru pomocí posuvníku **Vigneta** v [Živém náhledu](#live-preview).

### Živý náhled *(pouze kombinovaný)** **Index**: zaškrtávací políčko + ozubené kolečko — otevře sdílenou [Kalkulačku indexu](#index-calculator-pane) s pruhy nakreslenými ze**všech** zapojených kamer. Řádek s náhledem výrazu pod ním zobrazuje aktuální výraz („Není nastaven žádný výraz — otevřete kalkulátor a vytvořte jej“), který se aktualizuje každou sekundu.
* Rozbalovací nabídka **Rozlišení renderování**(stejné předvolby jako pro jednotlivé kamery, výchozí hodnota 720p): výška živého přenosu**a** velikost uloženého kompozitního exportu. Poznámka v panelu: „Náhled + velikost uloženého kompozitu. Snímky jednotlivých kamer se vždy exportují v plném rozlišení.“

### Zobrazovací vrstvy *(pouze kombinované)** Zaškrtávací políčko **Povolit** (výchozí nastavení vypnuto — hlavní kamera se zobrazuje přímo; zapnuto = vrstvený kompozit).
* Rozbalovací nabídky **Popředí**/**Pozadí**: jednotlivé členské kamery (podle názvu) nebo**Index**. Je-li v poli Popředí vybrána možnost Index, pixely mimo rozsah LUT Min/Max zobrazují vrstvu Pozadí.

### Rozdělené zobrazení *(pouze kombinované)*

**„Zobrazit členské kamery“**— tlačítko**Rozdělit / Skrýt členské kamery**, které přidá živý přenos každé členské kamery jako samostatné dlaždice mřížky vedle kompozice. Dlaždice čtou stávající rámcový buffer pole (bez dalšího připojení kamery). Pouze v mřížkovém zobrazení; ukládá se pro každé pole spolu s projektem.

### Funkce

Panel pouze pro čtení, aktualizovaný každých 5 s:

* **Označení úrovně**: „Souběžné snímání“ (zelená) · „Souběžné snímání (FTD – střídavé vysílání)“ (zelená) · „Střídavé snímání (100 ms odchylka)“ (oranžová) · „Konfigurace příliš velká“ (červená).
* **Stav snímku**: „x,xx % neúplné“ — zelená pod 1 %, oranžová pod 5 %, červená při 5 % a více.
* **Linková linka**: „NIC {mbps} Mbps – trvalý {MB/s} MB/s“.

Toto je aktuální šířka pásma pole. Informace o základních snímcích za sekundu (fps) a síťovém modelu — a o tom, co změnit, když se úroveň změní na oranžovou nebo červenou — najdete v [Pole s více kamerami](arrays.md) a [Referenční příručku CLI](../reference/cli-reference.md).



<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->## Panel Kalkulátor indexu

Třetí stránka postranního panelu, společná pro nástroj Index pro jednotlivé kamery a nástroj Index pro kombinované pole (vždy pouze jeden – nadpis zní „Kalkulátor indexu — <camera name="">“ nebo „Kalkulátor indexu —<array name="">

“). Zde se zobrazuje seznam pásem (přírodní pásma filtru kamery nebo všechna pásma napříč členy pole), aktuální výraz a konfigurace LUT (zapnuto/vypnuto, úroveň – výchozí hodnota 3, min. – výchozí hodnota 0,2, max. – výchozí hodnota 1) a živý histogram indexu. **Tlačítko**Použít** potvrdí výraz; změny LUT se okamžitě projeví v náhledu.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Nastavení pro jednotlivé kamery vs. nastavení spravovaná polem

Stručný přehled toho, co se kde nachází, když je kamera členem pole:

| Spravováno polem (v panelu kamery pouze pro čtení) | Stále pro jednotlivé kamery v rámci pole |
| --- | --- |
| Formát pixelů, rozlišení, binning | Automatická expozice (expozice, zisk, cíl, vyhlazení, ROI) |
| Režim/zdroj spouštění, snímková frekvence | Odšumění, ostrost, vinětace |
| | Orientace (zrcadlení/rotace), překryvy na displeji, bodový expozimetr |
| | Index (pole se samostatným zobrazením), přiřazení světelného senzoru |

Další obecná chování:

* **Kombinované vs. samostatné zobrazení** se volí při připojení k poli: kombinované = jedna zarovnaná složená dlaždice (členové přenášejí obraz pouze přes Split View); oddělené = každý člen vykresluje vlastní synchronizovanou dlaždici. Kamera nikdy nezobrazuje současně samostatný přenos i dlaždici pole.
* **Automatické opětovné připojení**: otevření uloženého projektu obnoví jeho kamery a pole a znovu aplikuje všechna uložená nastavení na backend před obnovením přenosů.
* **Omezení snímání**: skryté nebo pozastavené kamery jsou vyloučeny z funkce „Capture All“; pole je zcela zablokováno pouze tehdy, jsou-li VŠECHNY členy skryté/pozastavené. Viz [Nastavení a režimy snímání](capture.md).

## Jak se nastavení ukládají

Stav karty „kamery“ se ukládá **spolu s projektem**, nikoli v prohlížeči:

* Při každé reaktivní změně se pořídí snímek kamer a polí do souboru `cameras.json` projektu (s odskokem 500 ms). To zahrnuje názvy a barvy kamer, nastavení expozice/zisku/AE, formát pixelů/rozlišení/binning, frekvenci spouštění, nastavení náhledu (rozlišení renderování, odšumování, ostrost, vinětace, barevný profil, sytost/kontrast), orientaci, překryvy, rozdělení kanálů, konfiguraci indexu, nastavení prediktivního AE, AE ROI, názvy polí, režim zobrazení, nastavení snímání polí (včetně polohy oříznutí ROI) a blok mřížky (zoom kanálu, režim zobrazení, uzamčení mřížky, ruční pořadí dlaždic, skryté kamery, uzavřené dlaždice, aktivní kamera).
* Přiřazení světelných senzorů se ukládají do souboru projektu `sensors.json`.
* Při opětovném otevření projektu dojde k opětovnému připojení hardwaru a k opětovnému použití všech těchto nastavení.
* **Žádný otevřený projekt = pouze relace**: bez projektu se po zavření programu Chloros nic neuloží.
* Pouze relace bez ohledu na projekt: stav pozastavení, vzorky bodového měřiče, zaškrtávací políčko Kalibrační cíl pro jednotlivé kamery (vždy se otevírá v nezaškrtnutém stavu) a profil zarovnání pole (záměrně přepočítávaný pro každou relaci).
* Jedna výjimka: výběry pro export v části **Nastavení snímání** a režim snímání zůstávají uloženy pro každý projekt v lokálním úložišti aplikace, nikoli v `cameras.json` — viz [Nastavení a režimy snímání](capture.md).</array></camera></serial>
