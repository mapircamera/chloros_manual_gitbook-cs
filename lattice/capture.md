# Nastavení a režimy snímání

Snímání na kartě „Kamery“ se ovládá pomocí jednoho červeného tlačítka **Capture All**a jednoho panelu**Capture Settings**, který určuje, co toto tlačítko spustí: které kamery se zapojí, jaké typy exportu každá kamera ukládá a zda se spoušť spustí jednou, nepřetržitě nebo v určitých intervalech. Tato stránka popisuje celý postup – konfiguraci, samotné snímání, umístění souborů na disku a způsob jejich pozdějšího zpracování do kalibrovaných výstupů. Ovládací prvky pro kamery a pole se nacházejí na stránce [Nastavení kamer](camera-settings.md).

{% hint style="info" %}
**Pro pořizování snímků je nutné mít otevřený projekt.** Tlačítko „Pořídit vše“ a ozubené kolečko „Nastavení pořizování“ jsou deaktivovány, dokud není otevřen projekt („Vytvořte nebo otevřete projekt pro uložení snímků“). Každý snímek se ukládá do projektové složky v `captures/`.
{% endhint %}

## Panel Nastavení snímání

Otevřete jej pomocí **ozubeného kolečka vedle položky Zachytit vše**v seznamu kamer na postranním panelu nebo pomocí tlačítka**„Otevřít nastavení snímání…“** v dolní části libovolného panelu nastavení pro jednotlivé kamery. V záhlaví je nápis „Nastavení snímání“ s tlačítkem ← zpět.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->

Vaše volby zde – zahrnuté kamery, zaškrtávací políčka pro jednotlivé typy a režim snímání – se ukládají **pro každý projekt zvlášť** a obnoví se, když projekt znovu otevřete.

### Režimy snímání

Tři tlačítka režimů v horní části panelu:

| Režim | Funkce | Doplňková nastavení (výchozí hodnoty) |
| --- | --- | --- |
| **Jednorázové** *(výchozí)* | Jeden snímek ze všech vybraných kamer. | — |
| **Sériový**| Snímky po sobě, dokud není splněna podmínka zastavení. | Zastavení podle**počtu snímků** (výchozí hodnota 1) *nebo* **délky snímání** (výchozí hodnota 10 s; jednotky: sekundy / minuty / hodiny / dny). |
| **Interval**(časosběr) | Série snímků podle časovače. |**Počet snímků / interval**(výchozí hodnota 1) ·**Každých**N jednotek (výchozí hodnota 5 s) ·**Po dobu** N jednotek (výchozí hodnota 1 m). |

V režimu Kontinuální nebo Interval se tlačítko Zachytit vše během provozu změní na tlačítko **Zastavit (N)** a počítá snímky, jakmile se objeví.

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->

### Výběr kamer a typů exportu

Nápověda v tomto panelu to shrnuje: vyberte, které kamery a typy exportu má funkce „Zachytit vše“ generovat — ve výchozím nastavení je vše zapnuto a volby se ukládají spolu s tímto projektem.

* Tlačítka **Vybrat vše / Nezvolit nic** najednou přepnou zaškrtávací políčka u všech kamer.
* **Přepínače typů hromadného exportu**(dva řádky tlačítek):**Všechny Raw / Všechny s odstraněným bayesovým efektem / Všechny náhledy / Všechny Radiance / Všechny odrazivosti / Všechny indexové**. Každé z nich má tři stavy: zelená ✓ = zapnuto pro všechny kamery, které daný typ podporují, oranžová – = zapnuto pro některé, šedá = žádné. Přepínač je deaktivován, pokud žádný připojený fotoaparát daný typ nepodporuje. Všechny jsou šedé, pokud je zapnuto „Nejrychlejší snímání“.
* **Řádky pro jednotlivé kamery**: zaškrtávací políčko pro zahrnutí a rozbalitelný (▸/▾) seznam příslušných typů exportu pro danou kameru s jednotlivými zaškrtávacími políčky. Řádek zobrazuje počet zapnutých položek, např. „4/6“.

### Typy exportu a kamery, které je podporují

Existuje šest typů exportu: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. V řádku každé kamery se zobrazují pouze ty, které jsou pro ni relevantní:

| Typ exportu | Obsah | RGB (FRGB) | Multispektrální Bayer (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Bayerova mozaika (mono: jedno pásmo) přímo ze snímače | ✓ | ✓ | ✓ |
| **Debayered** | Lineární demosaikování (mono: 1kanálová škála šedé) | ✓ | ✓ | ✓ |
| **Náhled** | Kompletní řetězec zpracování (vyvážení bílé + gama podle profilu fotoaparátu; multispektrální: roztažení falešných barev) | ✓ | ✓ | ✓ |
| **Zářivost** | float32 W/m²/sr/nm přes celý radiometrický řetězec | — (nenabízeno) | ✓ | ✓ |
| **Odrazivost** | uint16 ρ (32768 = 1,0) | — (nenabízeno) | ✓ — zobrazeno pouze v případě, že kamera má světelný senzor DAQ (vlastní nebo zděděný od pole) | stejné jako u multispektrálního režimu |
| **Index** | vykreslení vegetačního indexu (LUT) | — | ✓ — vyžaduje povolený, neprázdný indexový výraz na kameře a není k dispozici pro členy kombinovaného pole (pole vlastní jeden sdílený index) | — (index vyžaduje ≥2 pásma; viz [Mono kamery a vegetační indexy](mono-indices.md)) |

Radiance a odrazivost nejsou nikdy k dispozici pro kamery RGB — radiance na jeden Bayer není pro širokopásmový fotometrický senzor smysluplná.

### Nejrychlejší snímání

Přepínač **⚡ Nejrychlejší snímání — pouze raw**(oranžový, když je zapnutý) přepsá všechna nastavení exportu na**pouze raw** — plus bezplatný kompozit s kombinovaným indexem pro pole — takže snímek je k dispozici co nejrychleji: výpočty zářivosti, odrazivosti a zobrazení jsou v okamžiku snímání zcela vynechány.

{% hint style="info" %}
**Soubor `.daq` se stále ukládá.** Je-li přiřazen světelný senzor, funkce Nejrychlejší snímání i tak zapíše naměřené hodnoty DAQ směrem dolů vedle snímků ve formátu RAW — takže produkty zářivosti, odrazivosti a indexu lze všechny vytvořit později opětovným zpracováním (viz [Opětovné zpracování snímků](#re-processing-captures-into-calibrated-products)). Funkce Fastest Capture také neovlivňuje výběr zaškrtávacích políček: po jejím vypnutí se výběr vrátí do původního stavu.
{% endhint %}

### Ovládací prvky pro jednotlivé pole

Každé připojené pole má v panelu vlastní skupinovou kartu:

* **Zaškrtávací políčko „Include“** (třípolohové pro všechny členy) a název pole s režimem zobrazení: „(combined | separate)“.
* Zaškrtávací políčko **Aligned**(výchozí nastavení**zapnuto**): přizpůsobí exporty členů profilu zarovnání pole, takže exporty jsou pixelově zarovnány napříč kamerami. Surová data zůstávají bez deformace, ale v metadatech obsahují transformaci. (Samotný profil se vypočítává v [panelu nastavení pole](camera-settings.md#alignment-co-registration-combined-only).)
* Řádky jednotlivých kamer jsou v kartě vnořeny.

Karta pole také obsahuje dva záznamníky. Představte si je jako **monitorování vs. analýza**:

| Záznamník | Úroveň | Co zaznamenává |
| --- | --- | --- |
| **● Nahrávat indexové video / ■ Zastavit nahrávání** *(pouze u kombinovaných polí)* | **Monitorování** | Živý kompozit kombinovaného indexu do videa při 10 fps — 8 bitů, rozlišení náhledu, zabudovaná LUT. Vyžaduje otevřený projekt a streamovaný živý náhled. Zobrazuje snímky a uplynulý čas během nahrávání. |
| **⦿ Nahrávání surové série snímků / ■ Zastavit záznam surové série** *(libovolná matice)* | **Analýza**| Surové snímky v formátu Bayer při rychlosti živého snímání (bez zpracování) plus manifest pro každý snímek a hodnoty `.daq`, uložené do `captures/bursts/`. Po sérii snímků se zobrazí tlačítko**Vytvořit video**: offline přepracuje sérii snímků na kalibrované video — kombinovaný index a/nebo radianci / odrazivost / index pro každou kameru — plus volitelné soubory TIFF. Vytvoření kombinovaného indexu se spustí automaticky po zastavení série snímků.

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

|## Pracovní postup

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

„Capture All“ Stiskněte **Capture All** v seznamu kamer na postranním panelu:

1. Každá zahrnutá, viditelná a nezastavená kamera pořizuje záznam s vybranými typy exportu. **Seskupení kamer se spouští jako jeden synchronizovaný spouštěč** (jedna synchronizovaná skupina napříč všemi členy — viz [Seskupení více kamer](arrays.md)); samostatné kamery pořizují snímky individuálně.
2. Skryté (oko) nebo pozastavené kamery jsou přeskočeny. Pole je zcela zablokováno pouze tehdy, jsou-li *všichni* jeho členové skrytí nebo pozastavení.
3. Kdykoli je přiřazen světelný senzor, odpovídající hodnota DAQ pro dopadající záření se ukládá jako soubor `.daq` společně se snímky — a to i u záznamů pouze ve formátu RAW —, takže radiometrické produkty lze vždy odvodit později.
4. Tlačítko zobrazuje průběh v reálném čase — „Zaznamenává se… hotovo/celkem“ — a v režimu Kontinuální/Interval se změní na **Stop (N)**. Každá položka záznamu má časový limit 300 s.
5. Po dokončení průchodu se v výsledkovém hlášení zobrazí **„Uloženo N souborů“**nebo**„Uloženo N, F selhalo“**, plus „(S skryto/pozastaveno/přeskočeno)“, pokud byly některé kamery přeskočeny.

## Kam se ukládají záznamy

Záznamy se ukládají v rámci otevřeného projektu do složky `<project>/captures/`. Každý typ exportu se ukládá do **vlastní podsložky**, takže u víceúrovňového záznamu se typy nikdy nemíchají:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── *.daq          the downwelling reading matched to the capture
```

* `<ts>` je časové razítko záznamu a `<serial>` je sériové číslo kamery. Samostatné snímky mají název `capture_<ts>_SN<serial>_<level>`; snímky pole z jednoho synchronizovaného spouštěče mají název `sync_<ts>_SN<serial>_<level>` a **sdílejí jeden časový údaj pro všechny kamery ve skupině** (přípona úrovně se vynechá, pokud kamera ukládá pouze jednu úroveň).
* **Jedna nesrovnalost, kterou je třeba znát:** úroveň zobrazení je uložena ve složce s názvem `preview/`, zatímco soubory si v názvu zachovávají `_display` — složka a přípona se liší pouze pro tuto úroveň.
* Neznámé úrovně se ukládají do složky s vlastním názvem; pokud nelze vytvořit podsložku, soubor se zapíše do kořenového adresáře snímků, místo aby byl ztracen.
* Snímky ve formátu TIFF jsou ve výchozím nastavení bezeztrátově komprimovány (DEFLATE) a obsahují úplná metadata o kalibraci a zpracování **uvnitř souboru XMP** — snímky jsou samopopisné a neobsahují žádné doprovodné soubory kromě souboru `.daq`.

Jedná se o stejné uspořádání, jaké soubory `chloros-cli lattice capture` / `array-capture` zapisují do svého adresáře `-o` — popsáno v [CLI Reference § Jak vypadá složka s pořízenými snímky](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, a .daq file at the root, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Zpracování pořízených snímků do kalibrovaných produktů

Zachycené surové snímky spolu s uloženým souborem `.daq` jsou vše, co zpracovatelský proces potřebuje — proto je režim „Fastest Capture“ bezpečný pro skutečnou práci.

* **GUI**: přidejte složku se snímky do projektu ([Přidávání souborů do projektu](../processing-images-gui/adding-files-to-a-project.md)) a zpracujte je jako obvykle.
* **CLI**: nasměrujte `process` na**kořenovou složku snímků**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` obvykle importuje pouze složku, kterou zadáte, ale pokud tato složka neobsahuje žádné obrázky a má podsložky, prochází je automaticky — takže podsložky jednotlivých úrovní i soubory v kořenové složce `.daq` jsou načteny najednou. Každý záznam se importuje jako **jediný obrázek** s ostatními úrovněmi připojenými jako režimy zobrazení, nikoli jako jeden obrázek pro každou úroveň.

Přímo pojmenovat podsložku úrovně (např. `…/captures/raw/`) také funguje, ale kořenové soubory `.daq` zůstanou opomenuty — zkopírujte je společně s nimi, když znovu odvozujete radiometrický produkt z `raw/`, jinak nebude mít shoda časových razítek k čemu se přiřadit.

{% hint style="warning" %}
**Zpracování vždy začíná od souboru `raw`.**V rámci každého záznamu je zdrojem pro zpracovatelský řetězec surový snímek; soubory `debayered`, `radiance`, `reflectance` a `preview` se objevují jako zobrazitelné režimy, ale nikdy nejsou vráceny zpět do potrubí — opětovné zpracování odvozeného produktu by znovu aplikovalo vinětaci, barvy a výpočty jasu, které jsou již zapracovány do jeho pixelů, proto Chloros tento proces odmítne, aby nedošlo k dvojímu zpracování. Rendery `index/` a `composite/` nejsou zpracovány vůbec (jedná se o výstupy, nikoli o snímky). Složka „captures“ uložená**bez** importů surových dat se zobrazuje normálně, ale `process` ji přeskočí a oznámí to; `--input-level {raw,debayered,processed}` je záměrný únikový mechanismus, který vynutí vstupní bod. Přesné zprávy o přeskočení najdete v [Referenci k CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like).
{% endhint %}

Další dvě chování, která stojí za to znát při skriptování opakovaného zpracování:

* Spuštění `chloros-cli process`, které požadovalo výstupy, ale nezapsalo **žádné obrazové výstupy, skončí s hlášením chyby a ukončí se s nenulovým stavem** — nikdy nedojde k tichému prázdnému spuštění. Úspěšná spuštění hlásí počet výstupů. (Úmyslné spuštění pouze s metadaty se stále počítá jako úspěch.)
* Znovu importované zpracované výstupy nikdy nezabírají slot pro surová data daného snímku — původní surová data vždy zůstávají zdrojem pro zpracování.

## Ekvivalenty CLI

Všechno na této stránce lze spouštět bez grafického rozhraní. Režimy snímání v grafickém rozhraní se přímo mapují na `chloros-cli lattice array-capture`:

| GUI | CLI |
| --- | --- |
| Jednorázový | `chloros-cli lattice array-capture` |
| Kontinuální | `array-capture --continuous [--count N] [--duration S]` |
| Interval | `array-capture --interval S [--duration S]` |
| Nejrychlejší snímání | `array-capture --fastest` |
| Zarovnané zaškrtávací políčko | `--aligned / --no-aligned` |
| Zaškrtávací políčka typu exportu | `--processing LEVEL` nebo `--levels L1,L2,…` (výchozí `all`) |
| Záznam indexového videa | `chloros-cli lattice array-record` |
| Záznam surových snímků v sérii / Vytvoření videa | `chloros-cli lattice array-burst` / `array-build-video` |

Kompletní tabulky příznaků, možnost snímání s inteligentní automatickou expozicí (`--smart`) a model s konstantní snímkovou frekvencí jsou popsány v [CLI Referenční příručka § Režimy snímání, Záznamníky a offline zpracování](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
