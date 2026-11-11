# Nastavení a režimy snímání

Snímání na záložce „Kamery“ se ovládá pomocí jednoho červeného tlačítka **Capture All**a jednoho panelu**Capture Settings**, který určuje, co toto tlačítko spustí: které kamery se zapojí, jaké typy exportu každá kamera uloží a zda se spoušť spustí jednou, nepřetržitě nebo v určitých intervalech. Tato stránka popisuje celý postup – konfiguraci, samotné snímání, umístění souborů na disku a způsob jejich pozdějšího zpracování na kalibrované výstupy. Ovládací prvky pro kamery a pole se nacházejí na stránce [Nastavení kamer](camera-settings.md).

{% hint style="info" %}
**Pro snímání je nutné mít otevřený projekt.** Možnost „Zachytit vše“ a ozubené kolečko „Nastavení snímání“ jsou deaktivovány, dokud není otevřen projekt („Vytvořte nebo otevřete projekt pro uložení snímků“). Každý snímek se ukládá do projektové složky v `captures/`.
{% endhint %}

## Panel Nastavení snímání

Otevřete jej pomocí **ozubeného kolečka vedle tlačítka Zachytit vše**v seznamu kamer na postranním panelu nebo pomocí tlačítka**„Otevřít nastavení snímání…“** v dolní části libovolného panelu nastavení pro jednotlivé kamery. V záhlaví je nápis „Nastavení snímání“ s tlačítkem ← zpět.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->



Vaše volby zde – vybrané kamery, zaškrtávací políčka pro jednotlivé typy a režim snímání – se ukládají **pro každý projekt zvlášť** a obnoví se při jeho opětovném otevření.

### Režimy snímání

Tři tlačítka režimů v horní části panelu:

| Režim | Funkce | Podnastavení (výchozí hodnoty) |
| --- | --- | --- |
| **Jednorázový** *(výchozí)* | Jedno snímání ze všech vybraných kamer. | — |
| **Kontinuální**| Snímání v rychlém sledu až do splnění podmínky zastavení. | Zastavení podle**počtu snímků** (výchozí hodnota 1) *nebo* **délky snímání** (výchozí hodnota 10 s; jednotky: sekundy / minuty / hodiny / dny). |
| **Interval**(časosběr) | Sériové snímání podle časovače. |**Počet snímků / interval**(výchozí hodnota 1) ·**Každých**N jednotek (výchozí hodnota 5 s) ·**Po dobu** N jednotek (výchozí hodnota 1 m). |

V režimu Kontinuální nebo Interval se tlačítko Zachytit vše během běhu změní na tlačítko **Zastavit (N)**, přičemž se počítá počet snímků, jakmile se objeví. Naplnění disku zastaví běh (viz [Průběh funkce „Zachytit vše“](#the-capture-all-flow)).**Režim „Kontinuální“ u polí běží uvnitř backendu (novinka ve verzi 1.3.0).** Pokud všechny kamery v seznamu patří do pole (samostatné kamery jsou skryté nebo vyloučené), režim „Continuous“ již neodesílá jeden požadavek na každý záznam: backend Chloros

spouští smyčku záznamu sám, zapisuje jednu synchronizovanou skupinu a zároveň vybírá další, přičemž tlačítko zjišťuje její počet. Pokud jsou v seznamu samostatné kamery, Chloros

provádí záznam po jednotlivých průchodech jako dříve.

{% hint style="warning" %}
**Rychlé snímání ve Windows

: ponechte backend Chloros

vyloučený ze skenování Microsoft Defenderem.** Skenování každého souboru v reálném čase při vysokofrekvenčním snímání zabírá podle testů MAPIR

přibližně 30 % rychlosti snímání (stovky souborů za sekundu při malé oblasti zájmu). Instalační program pro Windows

nabízí možnost **„Vyloučit backend Chloros

u ze skenování Microsoft Defenderem (mnohem rychlejší snímání z kamery; doporučeno)“**, která je ve výchozím nastavení zaškrtnutá, a při odinstalaci toto vyloučení odstraní. Pokud jste tuto možnost odškrtli, spusťte instalační program znovu s zaškrtnutou volbou, nebo přidejte vyloučení procesu Defenderu pro `chloros-backend.exe` (ve složce `backend` v instalační složce Chloros

). Pokud Defender během nepřetržitého snímání pole využívá jedno nebo více jader procesoru, backend Chloros

to jednou zaznamená do svého protokolu a uvede tytéž dvě řešení.
{% endhint %}

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->




### Výběr kamer a typů exportu

Nápověda v tomto panelu to shrnuje: vyberte, které kamery a typy exportu má funkce „Capture All“ (Zachytit vše) generovat — ve výchozím nastavení je zapnuto vše a volby se ukládají spolu s tímto projektem.

* Tlačítka **Vybrat vše / Nevybrat nic** najednou přepnou zaškrtávací políčka u všech kamer.
* **Hromadné přepínače typů exportu**(dva řádky tlačítek):**Všechny Raw / Všechny Debayered / Všechny náhledy / Všechny Radiance / Všechny Reflectance / Všechny Index**. Každý z nich má tři stavy: zelená ✓ = zapnuto pro všechny kamery, které daný typ podporují, oranžová – = zapnuto pro některé, šedá = žádná. Přepínač je deaktivován, pokud žádný připojený fotoaparát daný typ nepodporuje. Všechny jsou zašedlé, pokud je zapnuta funkce „Fastest Capture“.
* **Řádky pro jednotlivé fotoaparáty**: zaškrtávací políčko pro zahrnutí a rozbalitelný (▸/▾) seznam příslušných typů exportu pro daný fotoaparát s jednotlivými zaškrtávacími políčky. Řádek zobrazuje počet aktivních položek, například „4/6“.

### Typy exportu a kamery, které je podporují

Existuje šest typů exportu: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. V řádku každé kamery se zobrazují pouze ty, které jsou pro ni relevantní:

| Typ exportu | Obsah | RGB

(FRGB) | Bayer multispektrální (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Bayerova mozaika (mono: jedno pásmo) přímo ze snímače | ✓ | ✓ | ✓ |
| **Debayered** | Lineární demosaikování: z raw souboru je odstraněn tmavý offset (DSNU) z kalibračního balíčku, následně je provedeno demosaikování s ohledem na hrany — bez korekce linearity, flat fieldu nebo unmixu. Uloženo jako 3kanálový RGB

TIFF

(mono: 1kanálová šedá stupnice) | ✓ | ✓ | ✓ |
| **Náhled** | Kompletní řetězec zobrazení (vyvážení bílé + gama podle profilu kamery, plus nastavení „Highlights“ u RGB

kamery; multispektrální: roztažení falešných barev) | ✓ | ✓ | ✓ |
| **Zářivost** | float32 W/m²/sr/nm prostřednictvím úplného radiometrického řetězce | — (nenabízeno) | ✓ | ✓ |
| **Odrazivost** | uint16 ρ (32768 = 1,0) | — (není k dispozici) | ✓ — zobrazeno pouze v případě, že kamera disponuje světelným senzorem DAQ (vlastním nebo zděděným z pole) | stejné jako u multispektrálních |
| **Index** | Zobrazení vegetačního indexu (LUT) | — | ✓ — vyžaduje povolený, neprázdný indexový výraz na kameře a není k dispozici pro členy kombinovaného pole (pole vlastní jeden sdílený index) | — (index vyžaduje ≥2 pásma; viz [Mono kamery a vegetační indexy](mono-indices.md)) |

Radiance a odrazivost nejsou nikdy k dispozici pro RGB

kamery — radiance na úrovni Bayerova filtru nemá pro širokopásmový fotometrický senzor smysl.

### Nejrychlejší snímání

Přepínač **⚡ Nejrychlejší pořízení — pouze raw**(oranžový, když je zapnutý) přepíše všechna nastavení exportu na**pouze raw** — plus bezplatný kompozit s kombinovaným indexem pro pole — takže snímek se uloží co nejrychleji: výpočty radiance/reflektance/zobrazení jsou při pořízení zcela přeskočeny.

{% hint style="info" %}
**Soubor `.daq` se stále ukládá.** Je-li přiřazen světelný senzor, funkce Nejrychlejší snímání i tak zapíše hodnotu DAQ pro sestupující záření (do složky `daq/` snímků) — takže produkty zářivosti, odrazivosti a indexu lze všechny vytvořit později opětovným zpracováním (viz [Zpracování snímků](#re-processing-captures-into-calibrated-products)). Funkce Fastest Capture také neovlivňuje vaše volby v zaškrtávacích polích: po jejím vypnutí se tyto volby obnoví.
{% endhint %}

### Ovládací prvky pro jednotlivá pole

Každé připojené pole má v panelu vlastní skupinovou kartu:

* **Zaškrtávací políčko „Include“** (třípolohové pro všechny členy) a název pole s režimem zobrazení: „(combined | separate)“.
* Zaškrtávací políčko **Aligned**(výchozí nastavení**zapnuto**): přizpůsobí exporty členů profilu zarovnání pole, takže exporty jsou pixelově zarovnány napříč kamerami. Surová data zůstávají bez warpování, ale v metadatech nesou transformaci. (Samotný profil se vypočítává v [panelu nastavení pole](camera-settings.md#alignment-co-registration).) Soubory členů pole se ukládají v nativní orientaci každé kamery, přičemž jakékoli nastavení zrcadlení/rotace se do souboru pouze zaznamená, nikoli aplikuje.
* Řádky jednotlivých kamer jsou vnořeny uvnitř karty.

Karta pole také obsahuje dva záznamníky. Představte si je jako **monitorování vs. analýza**:

| Záznamník | Úroveň | Co zaznamenává |
| --- | --- | --- |
| **● Nahrávat indexové video / ■ Zastavit nahrávání** *(pouze kombinovaná pole)* | **Monitorování** | Živý kompozitní indexový obraz do videa při 10 fps — 8 bitů, rozlišení náhledu, zabudovaná LUT. Vyžaduje otevřený projekt a živý streamovaný náhled. Během nahrávání zobrazuje snímky a uplynulý čas. |
| **⦿ Nahrávání surové série snímků / ■ Zastavit záznam surové série** *(libovolná matice)* | **Analýza**| Surové snímky v formátu Bayer v reálném čase (bez zpracování) plus manifest pro každý snímek a hodnoty `.daq`, uložené do souboru `captures/bursts/`. Po sérii snímků se zobrazí tlačítko**Vytvořit video**: offline přepracuje sérii snímků na kalibrované video — kombinovaný index a/nebo radianci / odrazivost / index pro každou kameru — plus volitelné soubory TIFF. Vytvoření kombinovaného indexu se spustí automaticky po zastavení série snímků. |

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

## Postup

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

„Capture All“

Stiskněte **Capture All** v seznamu kamer na postranním panelu:

1. Každá zahrnutá, viditelná a nezastavená kamera pořizuje snímky s vybranými typy exportu. **Seskupení kamer se spouští jako jeden synchronizovaný spouštěč** (jedna synchronizovaná skupina napříč všemi členy — viz [Seskupení více kamer](arrays.md)); samostatné kamery pořizují snímky individuálně.
2. Skryté (oko) nebo pozastavené kamery se přeskočí. Pole je plně zablokováno pouze tehdy, jsou-li *všichni* jeho členové skrytí nebo pozastavení.
3. Kdykoli je přiřazen světelný senzor, odpovídající hodnota DAQ pro dopadající záření se uloží jako soubor `.daq` do složky `daq/` záznamů — a to i u záznamů pouze ve formátu RAW —, takže radiometrické produkty lze vždy odvodit později.
4. Tlačítko zobrazuje průběh v reálném čase — „Zaznamenává se… hotovo/celkem“ — a v režimu Kontinuální/Interval se změní na **Zastavit (N)**. Každá položka záznamu má časový limit 300 s.
5. Po dokončení průchodu se v výsledkovém hlášení zobrazí **„Uloženo N souborů“**nebo**„Uloženo N, F selhalo“**, plus „(S skryto/pozastaveno/přeskočeno)“, pokud byly některé kamery přeskočeny. Pokud se disk zaplní, běh se zastaví a zobrazí hlášení**„Zastaveno: disk plný. Uloženo N, F selhalo“** — každý soubor, který nebylo možné zapsat, se počítá jako neúspěšný.

## Kam se ukládají záznamy

Záznamy se ukládají v rámci otevřeného projektu do složky `<project>/captures/`. Každý typ exportu se ukládá do **vlastní podsložky**, takže při víceúrovňovém záznamu se typy nikdy nemíchají:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_clampmask.tif, reflectance/…, index/… — unmix clamp masks,
│                  one subfolder per product folder, only when a band was clamped
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── daq/           the downwelling reading(s) matched to each capture (.daq), each with a
                   .daq.cameras.json naming the cameras it describes
```

* `<ts>` je časové razítko snímku a `<serial>` je sériové číslo kamery. Samostatné snímky mají název `capture_<ts>_SN<serial>_<level>`; snímky z pole pořízené jedním synchronizovaným spouštěčem mají název `sync_<ts>_SN<serial>_<level>` a **sdílejí jeden časový údaj pro všechny kamery ve skupině** (přípona úrovně se vynechává, pokud kamera ukládá pouze jednu úroveň).
* **Jedna nesrovnalost, kterou je třeba znát:** úroveň zobrazení se ukládá do složky s názvem `preview/`, zatímco soubory si v názvu zachovávají `_display` — složka a přípona se liší pouze u této úrovně.
* Neznámé úrovně se ukládají do složky s vlastním názvem; pokud nelze vytvořit podsložku, soubor se zapíše do kořenového adresáře snímků, místo aby byl ztracen.
* Snímky ve formátu TIFF jsou ve výchozím nastavení bezeztrátově komprimovány (DEFLATE) a obsahují úplná metadata o kalibraci a zpracování **uvnitř souboru XMP** — snímky jsou samopopisné a neobsahují žádné doprovodné soubory kromě souboru `.daq` (a jeho `.daq.cameras.json`).
* **Masky omezující hodnoty (novinka ve verzi 1.3.0).**Tam, kde muselo spektrální rozložení 3×3 omezit pásmo na 0 alespoň u jednoho pixelu, získají produkty radiance, odrazivosti a indexu masku pro každý pixel, `clamp_masks/<product folder>/<product name>_clampmask.tif` (bit 0 = pásmo v kanálu R produktu, bit 1 = G, bit 2 = B); na ni odkazuje XMP tag produktu `Chloros:UnmixClampMask`. Masky se nacházejí**vedle** složek produktů, nikdy uvnitř nich, takže složku produktu, jako je například `reflectance/`, lze předat fotogrammetrickému softwaru tak, jak je. Vlastní import programu Chloros

tyto masky přeskočí.
* **Orientace (1.3.0).** Produkty samostatného fotoaparátu typu Preview, Radiance, Reflectance a Index se otáčejí podle nastavení Mirror/Rotation; soubory Raw a Debayered zůstávají tak, jak je snímač načítal. Soubory v poli zůstávají všechny v původním formátu. Každý soubor zaznamenává nastavení ve svém XMP, takže při opětovném zpracování se použije stejným způsobem.
* **K které kameře patří soubor `.daq` (1.3.0).** `.daq` záznamu je při zpracování přiřazeno pouze k obrázkům z kamer, které uvádí jeho `.daq.cameras.json`. Samostatná kamera bez vlastního světelného senzoru proto již nepoužívá údaje z jiné kamery – přiřaďte k ní senzor nebo jej zaznamenejte na záložce Světelné senzory. Záznam ze světelného senzoru (a jakýkoli `.daq` bez tohoto souboru, včetně starších záznamů) stále slouží všem kamerám.

Jedná se o stejné uspořádání, které soubory `chloros-cli lattice capture` / `array-capture` zapisují do svého adresáře `-o` — zdokumentováno v [Referenci CLI

§ Jak vypadá složka snímků](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, the daq/ and clamp_masks/ folders, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Zpracování snímků do kalibrovaných produktů

Zachycené surové snímky spolu s uloženým souborem `.daq` jsou vše, co zpracovatelský proces potřebuje — proto je Fastest Capture bezpečný pro skutečnou práci.

* **GUI**: přidejte složku s pořízenými snímky do projektu ([Přidávání souborů do projektu](../processing-images-gui/adding-files-to-a-project.md)) a zpracujte je jako obvykle.
* **CLI**: nasměrujte `process` na **kořenovou složku s pořízenými snímky**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` obvykle importuje pouze složku, kterou zadáte, ale pokud tato složka neobsahuje žádné obrázky a má podsložky, automaticky se do nich dostane — takže podsložky jednotlivých úrovní a složka `daq/` jsou načteny najednou. Každý záznam se importuje jako **jeden obrázek** s ostatními úrovněmi připojenými jako režimy zobrazení, nikoli jako jeden obrázek na každou úroveň.

Přímé pojmenování podsložky úrovně (např. `…/captures/raw/`) také funguje, ale složka `daq/` zůstane opomenuta — zkopírujte ji vedle, když-odvozujete radiometrický produkt ze složky `raw/`, jinak nebude mít shoda časových razítek k čemu se přiřadit.

{% hint style="warning" %}
**Zpracování vždy začíná od složky `raw`.** V rámci každého záznamu je zdrojem pro zpracování surový snímek; `debayered`, `radiance`, `reflectance`, a `preview` se objevují jako zobrazitelné režimy, ale nikdy nejsou vráceny zpět do potrubí — opětovné zpracování odvozeného produktu by znovu aplikovalo vinětaci, barvu a výpočty jasu, které jsou již zapéčeny do jeho pixelů, proto Chloros

raději odmítne, než aby provedl dvojité zpracování. Rendery `index/` a `composite/` nejsou vůbec zpracovávány (jedná se o výstupy, nikoli o záznamy). Složka „captures“ uložená **bez** importu surových dat se zobrazuje normálně, ale `process` ji přeskočí a oznámí to; `--input-level {raw,debayered,processed}` je záměrná úniková cesta, která vynutí vstupní bod. Přesné zprávy o přeskočení najdete v [Referenci CLI

](../reference/cli-reference.md#what-a-captures-folder-looks-like).
{% endhint %}

Další dvě chování, která stojí za to znát při skriptování opětovného zpracování:

* Spuštění `chloros-cli process`, které požadovalo výstupy, ale **nezapsalo žádné obrazové výstupy, skončí s hlášením chyby a ukončí se s nenulovým stavem** — nikdy nedojde k tichému prázdnému spuštění. Úspěšná spuštění nahlásí počet svých výstupů. (Záměrné spuštění pouze s metadaty se stále počítá jako úspěch.)
* Znovu importované zpracované výstupy nikdy nezabírají slot pro surová data daného záznamu — původní surová data vždy zůstávají zdrojem pro zpracovatelský řetězec.

## Ekvivalenty v CLI



Všechny funkce na této stránce lze ovládat bez grafického rozhraní. Režimy snímání v grafickém rozhraní (GUI) odpovídají přímo příkazům `chloros-cli lattice array-capture`:

| GUI | CLI

|
| --- | --- |
| Jednorázový | `chloros-cli lattice array-capture` |
| Kontinuální | `array-capture --continuous [--count N] [--duration S]` |
| Interval | `array-capture --interval S [--duration S]` |
| Nejrychlejší snímání | `array-capture --fastest` |
| Zaškrtávací políčko „Aligned“ | `--aligned / --no-aligned` |
| Zaškrtávací políčka typu exportu | `--processing LEVEL` nebo `--levels L1,L2,…` (výchozí `all`) |
| Nahrávání indexového videa | `chloros-cli lattice array-record` |
| Nahrávání surových snímků v sérii / Vytvoření videa | `chloros-cli lattice array-burst` / `array-build-video` |

Kompletní tabulky příznaků, volba snímání se stabilizovanou automatickou expozicí (`--smart`) a model s konstantní snímkovou frekvencí jsou popsány v části [Referenční příručka CLI

, § Režimy snímání, záznamníky a offline zpracování](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
