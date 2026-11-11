# Nastavení projektu

Boční panel „Nastavení projektu“ <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

v aplikaci Chloros

vám umožňuje konfigurovat všechny aspekty zpracování snímků, detekci kalibračních cílů, výpočty multispektrálních indexů a možnosti exportu pro váš projekt. Tato nastavení se ukládají spolu s projektem a lze je uložit jako šablony pro opakované použití v různých projektech.

## Otevření nastavení projektu

Chcete-li otevřít nastavení projektu:

1. Otevřete projekt v programu Chloros

.
2. Klikněte na záložku **Nastavení projektu** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

v levém postranním panelu.
3. V panelu nastavení se zobrazí všechny dostupné konfigurační možnosti seřazené podle kategorií.



<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->

{% hint style="info" %}
**Nastavení, která závisí na jiných nastaveních, jsou šedá.** Pokud nadřazený přepínač znemožňuje provedení určitého nastavení (například odškrtnutí položky *Kalibrace odrazivosti / vyvážení bílé* znemožňuje *Export odrazivosti*), je závislý ovládací prvek deaktivován a v jeho popisku je uveden název přepínače, který je třeba změnit.
{% endhint %}

***

## Zobrazení

### Rozlišení miniatur obrázků

* **Typ**: Výběr z rozevíracího seznamu
* **Možnosti**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Výchozí**: Výchozí (512 px)
* **Popis**: Rozlišení (nejdelší hrana, v pixelech), ve kterém se vykreslují miniatury v mřížce obrázků. Vyšší hodnoty vypadají při přiblížení ostřeji, ale načítají se pomaleji a zabírají více paměti. Plné rozlišení odpovídá původní velikosti obrázku.
* **Poznámka**: Pouze pro zobrazení — toto nikdy neovlivňuje zpracování ani exportované soubory.***

## Detekce cílů

Tato nastavení určují, jak program Chloros

detekuje a zpracovává kalibrační cíle ve vašich snímcích. Obě jsou aktivní pouze při zapnuté funkci **Kalibrace odrazivosti / vyvážení bílé** (v opačném případě jsou zašedlá, protože detekce cílů je zcela vynechána).

### Minimální plocha kalibračního vzorku (px)

* **Typ**: Číslo
* **Rozsah**: 0 až 10 000 pixelů
* **Výchozí hodnota**: 25 pixelů
* **Popis**: Nastavuje minimální plochu (v pixelech) potřebnou k tomu, aby byla detekovaná oblast považována za platný vzorek kalibračního cíle. Menší hodnoty detekují menší cíle, ale mohou zvýšit počet falešných pozitiv. Větší hodnoty vyžadují pro detekci větší a zřetelnější oblasti cílů.
* **Kdy upravit**:
  * Zvyšte hodnotu, pokud dochází k falešným detekcím u malých obrazových artefaktů
  * Snižte hodnotu, pokud se vaše kalibrační cíle na snímcích jeví jako malé a nejsou detekovány

### Minimální shlukování cílů (0–100)

* **Typ**: Číslo
* **Rozsah**: 0 až 100
* **Výchozí hodnota**: 60
* **Popis**: Řídí prahovou hodnotu pro seskupování podobně zbarvených oblastí při detekci kalibračních cílů. Vyšší hodnoty vyžadují, aby byly seskupeny podobnější barvy, což vede k konzervativnější detekci cílů. Nižší hodnoty umožňují větší barevnou variabilitu v rámci jedné skupiny cílů.
* **Kdy upravit**:
  * Zvyšte hodnotu, pokud se kalibrační cíle rozdělují na více detekcí
  * Snižte hodnotu, pokud nejsou plně detekovány kalibrační cíle s barevnými odchylkami

***

## Zpracování

Tato nastavení určují, jak program Chloros

zpracovává a kalibruje vaše snímky.

### Korekce vinětace

* **Typ**: Zaškrtávací políčko
* **Výchozí**: Zapnuto (zaškrtnuto)
* **Popis**: Použije korekci vinětace k vyrovnání ztmavnutí okrajů snímků způsobeného objektivem. Vinětace je běžný optický jev, při kterém se rohy a okraje snímku jeví tmavší než střed v důsledku vlastností objektivu.
* **Vedlejší účinek**: Tímto přepínačem se také volí, který *nekalibrovaný náhradní produkt* se při zpracování uloží (viz níže).

### Kalibrace odrazivosti / vyvážení bílé

* **Typ**: Zaškrtávací políčko
* **Výchozí nastavení**: Zapnuto (zaškrtnuto)
* **Popis**: Zapíná kalibraci odrazivosti — na základě detekovaných kalibračních cílů v záběru a/nebo dat o dopadajícím světle ze světelného senzoru DAQ, v závislosti na kameře a dostupných zdrojích. Tím se normalizují hodnoty odrazivosti v celém datovém souboru a zajistí se konzistentní měření bez ohledu na světelné podmínky.
* **Při deaktivaci**: Detekce cílů je zcela vynechána a**žádná kamera nemůže vytvořit žádný produkt odrazivosti** — ať už se jedná o systém řízený cíli Survey3

, nebo systém řízený DAQ LATTICE. Související nastavení (*Exportovat odrazivost*, *Minimální interval rekalibrace* a prahové hodnoty detekce cílů) jsou zašedlá.

### Náhradní výstupy pro nekalibrované snímky: Export odezvy snímače / Export s korekcí vinětace

* **Typ**: Dvě zaškrtávací políčka
* **Výchozí nastavení**: Obě povolena (zaškrtnuta)
* **Popis**: Pokud nelze u snímku provést kalibraci odrazivosti (nebyl nalezen kalibrační cíl nebo je kalibrace odrazivosti vypnuta), je místo toho uložen jako *nekalibrovaný náhradní produkt*. **Pro každý model kamery existuje přesně jeden z těchto dvou náhradních produktů na jeden běh**, který se volí přepínačem *Korekce vinětace*:
  * Korekce vinětace **zapnuta**→ `Vignette_Corrected_Images/` (řízeno nastavením**Exportovat s korekcí vinětace**)
  * Korekce vinětace **vypnuta**→ `Sensor_Response_Images/` (řízeno nastavením**Exportovat odezvu snímače**)
* Náhradní produkt, který není aktivní, je zašedlý. Odškrtnutím aktivního produktu se zabrání zápisu daného souboru.

### Výstupní produkty LATTICE

U projektů obsahujících snímky LATTICE se každý importovaný snímek LATTICE v rámci jediného cyklu zpracování rozdělí do všech povolených **a použitelných**produktů. Rozdělení řídí čtyři zaškrtávací políčka (všechna jsou ve výchozím nastavení**zapnutá**):

| Nastavení | Výstupní složka | Co se exportuje |
| --- | --- | --- |
| **Exportovat po odstranění mřížky** | `Debayered_Images/` | Lineární obraz po odstranění mřížky: odstraněno tmavé posunutí kalibračního balíčku, následně demosaikování s ohledem na hrany, bez korekce linearity, flat fieldu nebo unmixu — stejný soubor, jaký záznam ukládá na úrovni bez bayeringu. Zachována nativní orientace snímače (zaznamenaná v XMP). Platí pro RGB

ové a multispektrální kamery. |
| **Export náhledu** | `Preview_Images/` | Náhled na displeji. RGB

= vyvážení bílé (světelný zdroj DAQ, je-li k dispozici, jinak šedý svět) + gama; multispektrální = roztažení do falešných barev. |
| **Export radiance** | `Radiance_Images/` | Spektrální radiance typu Float32 v jednotkách W/m²/sr/nm. Pouze pro multispektrální snímky (M3C/M3M) — nevztahuje se na RGB

mastery. Vždy se zapisuje jako 32bitový soubor TIFF

bez ohledu na nastavení *Kalibrovaný formát obrázku*. |
| **Export odrazivosti**| `Reflectance_Calibrated_Images/` | Odrazivost typu Uint16, škálovaná tak, že**32768 = odrazivost 1,0** (označeno jako XMP `Chloros:PixelScale`). Pouze multispektrální, zaznamenáváno, pokud snímek pokrývá odpovídající záznam `.daq` směřující dolů (nebo cíl v rámci snímku, který prošel kontrolou kvality). |

* Hlavní kamery RGB

vysílají data po odstranění bayesovského vzoru + náhled; radiance/reflektance se u nich vynechávají, protože se na ně nevztahují.
* Bitová hloubka dat po odstranění bayesovského vzoru/náhledu se řídí nastavením *Formát kalibrovaného obrazu*; radiance je vždy float32.
* Zpracování Survey3

není těmito čtyřmi přepínači ovlivněno.
* Produkty radiance a reflectance multispektrálních kamer M3C mohou také zapisovat **masku omezující hodnoty** do složky `clamp_masks/` vedle složek s produkty, čímž zaznamenávají, kde došlo při oddělení pásem k oříznutí záporné hodnoty na nulu. Viz [Formáty výstupních obrázků](../output-image-formats.md#clamp-masks).

Stejné čtyři přepínače existují i v bezhlavém režimu jako `chloros-cli process --debayered / --preview / --radiance / --reflectance`. Nahradily starý příznak CLI

`--radiometric-output`, který již neexistuje.

{% hint style="warning" %}
**Vypnutí všech příslušných výstupů způsobí selhání běhu.** Od verze 1.2.0 hlásí zpracování, u kterého byly požadovány výstupy, ale nezapsal žádný obrazový výstup, selhání a CLI

se ukončí s nenulovým stavem, místo aby hlásilo tiché úspěšné dokončení. Protokol uvádí název výstupu, který se nepodařilo zapsat, a důvod. Úmyslný běh pouze s metadaty (nic nebylo požadováno) je stále úspěšný.
{% endhint %}

### Zdroj odrazivosti (nastavení projektu, nastavuje se přes CLI

)

Projekt také ukládá, kterou **referenci odrazivosti** produkt odrazivosti LATTICE používá. V panelu nastavení není k dispozici žádný speciální ovládací prvek; hodnota je uložena v konfiguraci projektu jako `Processing → "Target reflectance source"` a nastavuje se pomocí `chloros-cli process --reflectance-source {auto,target,daq}` (SDK

`configure()` pro to nemá žádné klíčové slovo):

* **`auto`** (výchozí): kalibrační cíl v rámci snímku, který prošel kontrolou kvality (QA), se stává absolutní referencí; pokud žádný cíl není k dispozici nebo kontrola kvality selže, použije se jako náhrada dělení sestupného toku z DAQ (ρ = πL/E). Cíl, který prošel kontrolou kvality s výhradou (vypadlý panel nebo nefunkční pásmo), není automaticky upřednostňován: řetězce cíl, cíl+DAQ a pouze DAQ jsou zkoušeny v pořadí podle jejich naměřeného skóre na vyřazených panelech.
* **`target`**: přísná odrazivost řízená cílem — bez nahrazení DAQ.
* **`daq`**: odrazivost s rozhodujícím vlivem DAQ; cíle v rámci snímku se nepoužívají jako reference.

Uložená hodnota se porovnává bez ohledu na velká a malá písmena a několik variant zápisu je akceptováno jako aliasy: `target`, `target_image`, `empirical` a `empirical_line` – všechny znamenají **cíl**; `daq`, `dls`, `light_sensor` a `sensor` všechny znamenají**daq**. Cokoli jiného — včetně chybějícího klíče — se vyhodnotí jako**auto**.

Skeny **naměřených** cílů pro jednotlivé jednotky se vyhledávají podle sériového čísla/QR kódu cílové jednotky, například `<serial>.csv`, na třech místech: v adresáři zadaném pomocí `--target-reflectance-dir` (uloženém jako `Processing → "Target reflectance dir"`), ve vlastní složce projektu `target_reflectance/` a v cestě uvedené v proměnné prostředí `CHLOROS_TARGET_REFLECTANCE_DIR`. Pokud pro danou jednotku neexistuje žádný naměřený sken, použije se místo toho nominální publikovaná křivka pro cílový model.

### Metoda debayeringu

* **Typ**: Výběr z rozevíracího seznamu
* **Možnosti**:
  * Standardní (rychlé, střední kvalita)
  * S ohledem na texturu (pomalé, nejvyšší kvalita) \[Chloros

+]
* **Výchozí**: Standardní (rychlé, střední kvalita)
* **Popis**: Volí algoritmus demosaicingu používaný k převodu surových dat ze snímače s Bayerovým vzorem na plnobarevné snímky. Metoda „Standardní (rychlá, střední kvalita)“ poskytuje optimální rovnováhu mezi rychlostí zpracování a kvalitou obrazu. Metoda „S ohledem na texturu (pomalá, nejvyšší kvalita)“ \[Chloros

+] využívá vysoce kvalitní demosaicing s ohledem na hrany v kombinaci s modelem odšumování založeným na AI/ML, který odstraňuje téměř veškerý šum vznikající při demosaicingu. Model „S ohledem na texturu“ vyžaduje ke spuštění paměť GPU (VRAM). Pro rychlejší zpracování doporučujeme jej používat, pokud máte k dispozici více než 4 GB VRAM.
* **Pokud je řádek vůbec rozevíracím seznamem**: rozevírací seznam se dvěma možnostmi se zobrazí pouze tehdy, když**obě** podmínky jsou splněny — jste přihlášeni s platným předplatným Chloros

+ a **projekt** neobsahuje žádné snímky LATTICE. V opačném případě se řádek zobrazí jako prostý text `Standard (Fast, Medium Quality)` bez možnosti výběru.
* **Poznámka k LATTICE**: Neexistuje žádný model Texture Aware trénovaný pro LATTICE a zpracovatelský řetězec vynucuje standardní demosaikování pro snímky LATTICE bez ohledu na uloženou hodnotu. Pokud přidáte složku LATTICE do projektu, ve kterém již byla vybrána možnost Texture Aware, Chloros

přepíše nastavení zpět na Standard, místo aby ponechal zastaralou hodnotu v `project.json`.

### Minimální interval rekalibrace

* **Typ**: Číslo
* **Rozsah**: 0 až 3 600 sekund
* **Výchozí hodnota**: 0 sekund
* **Popis**: Nastavuje minimální časový interval (v sekundách) mezi použitím kalibračních cílů. Při nastavení na 0 bude Chloros

používat každý detekovaný kalibrační cíl. Při nastavení na vyšší hodnotu bude Chloros

používat pouze kalibrační cíle, které jsou od sebe odděleny alespoň tímto počtem sekund, čímž se zkrátí doba zpracování u datových sad s častým zachycením kalibračních cílů.
* **Kdy upravit**:
  * Nastavte na 0 pro maximální přesnost kalibrace při proměnlivých světelných podmínkách
  * Zvyšte hodnotu (např. na 60–300 sekund) pro rychlejší zpracování, pokud je osvětlení stálé a máte časté snímky kalibračních cílů

### Časové posunutí světelného senzoru

* **Typ**: Číslo
* **Rozsah**: -12 až +12 hodin
* **Výchozí hodnota**: 0 hodin
* **Popis**: Určuje časové posunutí (v hodinách od UTC) pro časová razítka dat světelného senzoru, které se používá při přiřazování záznamů světelného senzoru k časům pořízení snímků. Novější záznamy `.daq` obsahují vlastní informaci o časové zóně, takže toto nastavení je potřebné hlavně pro starší záznamy pořízené v místním čase.

### Použít korekce PPK

* **Typ**: Zaškrtávací políčko
* **Výchozí nastavení**: Vypnuto (nezaškrtnuto)
* **Popis**: Povoluje použití korekcí Post-Processed Kinematic (PPK) z datových záznamníků MAPIR

, které obsahují GPS (GNSS). Je-li tato funkce povolena, Chloros

použije všechny soubory protokolů .daq obsahující údaje o expozičním pinu ve vašem projektovém adresáři a aplikuje na vaše snímky přesné korekce geolokace.
* **Požadavek**: V adresáři projektu musí být přítomen soubor protokolu .daq se záznamy o expozičních pinech
* **Kdy povolit**: Doporučuje se vždy povolit korekci PPK, pokud máte ve svém souboru protokolu .daq záznamy o expozici.

### Expozice pin 1

* **Typ**: Výběr z rozevíracího seznamu
* **Viditelnost**: Viditelné pouze tehdy, když je povolena možnost „Použít korekce PPK“ A jsou k dispozici údaje o expozici pro pin 1
* **Možnosti**:
  * Názvy modelů kamer detekované v projektu
  * „Nepoužívat“ – tento expoziční pin ignorovat
* **Výchozí nastavení**: Automaticky vybráno na základě konfigurace projektu
* **Popis**: Přiřadí konkrétní kameru k expozici Pin 1 pro časovou synchronizaci PPK. Expozice Pin zaznamenává přesný čas spuštění závěrky kamery, což je zásadní pro přesnou geolokaci PPK.
* **Chování automatického výběru**:
  * Jedna kamera + jeden pin: Kamera se vybere automaticky
  * Jedna kamera + dva piny: Pin 1 se automaticky přiřadí k kameře
  * Více kamer: Je nutný ruční výběr

### Expozice – pin 2

* **Typ**: Výběr z rozevíracího seznamu
* **Viditelnost**: Viditelné pouze tehdy, je-li povolena možnost „Použít korekce PPK“ A jsou-li k dispozici údaje o expozici pro pin 2
* **Možnosti**:
  * Názvy modelů kamer detekované v projektu
  * „Nepoužívat“ – tento pin expozice ignorovat
* **Výchozí nastavení**: Automaticky vybráno na základě konfigurace projektu
* **Popis**: Při použití konfigurace se dvěma kamerami přiřadí konkrétní kameru k pinu expozice 2 pro časovou synchronizaci PPK.
* **Chování automatického výběru**:
  * Jedna kamera + jeden pin: Pin 2 se automaticky nastaví na „Nepoužívat“
  * Jedna kamera + dva piny: Pin 2 se automaticky nastaví na „Nepoužívat“
  * Více kamer: Je vyžadován ruční výběr
* **Poznámka**: Stejnou kameru nelze přiřadit současně k pinu 1 i k pinu 2.***

## Světelný senzor DAQ

Tato sekce se zobrazuje v nastavení projektu a obsahuje seznam všech souborů DAQ s údaji o slunečním záření v projektu — záznamy `.daq` a soubory DAQ-M `.csv`. Záznamy pořízené na kartě Světelné senzory se do otevřeného projektu přidávají automaticky.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->



Každý řádek zobrazuje soubor, model senzoru a korekci difuzorového krytu, která je pro daný soubor aktuálně platná. Nad řádky se nachází jediný ovládací prvek pro celý projekt:

### Přepsání nastavení krytu (všechny soubory)

* **Typ**: Výběr z rozevíracího seznamu
* **Možnosti**: `Auto` plus profily korekce difuzorového krytu platné pro typy senzorů přítomné v projektu
* **Výchozí**: Auto
* **Uloženo jako**: `Processing → "DAQ cap id"` (výchozí `auto`)
* **Popis**: `Auto` používá kryt zaznamenaný v každém souboru (pokud není nic zaznamenáno, předpokládá se kryt Sunshine, ale DAQ-U, DAQ-M a DAQ-E se dodávají bez krytu a program Chloros

nedokáže rozpoznat, jaký kryt je nasazen, proto zpracování vydá varování ohledně předpokládaného krytu: vyberte kryt, který byl nasazen). Výběr konkrétního krytu přepíše **všechny** soubory s daty o slunečním záření v projektu s výjimkou souboru z DAQ-E-S, jehož difuzor je součástí jeho kalibrace; projekt obsahující pouze DAQ-E-S nezobrazuje rozevírací seznam: surové záznamy jsou korigovány pomocí tohoto krytu, a záznamy, které již obsahují krytku, jsou znovu referencovány (zaznamenaná korekce je zrušena a je aplikována vybraná korekce).
* **Důležité**: Vybraná krytka se musí shodovat s krytkou, která byla fyzicky nasazena během záznamu. Ani senzor, ani software nedokážou detekovat fyzickou krytku — nesprávné ID krytky vede k nesprávné korekci spekter.

Záměrně je k dispozici **jedno** nastavení pro celý projekt namísto rozevíracích nabídek pro jednotlivé soubory: toto nastavení se vztahuje na všechny zdroje s dopadem dolů v projektu.***

## Zarovnání pole

Tato sekce se zobrazí **pouze** v případě, že alespoň jeden snímek v projektu obsahuje transformaci pro zarovnání mezi moduly, kterou pole LATTICE vkládají při pořízení (značky XMP `Chloros:Alignment*`). Ukazuje, kolik snímků obsahuje značky zarovnání, která kamera slouží jako referenční (značka `REF`), a tabulku počtu snímků podle jednotlivých kamer.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Použít zarovnání pole

* **Typ**: Zaškrtávací políčko
* **Výchozí**: Zapnuto (zaškrtnuto)
* **Uloženo jako**: `Processing → "Array alignment"`
* **Popis**: Deformuje každý zpracovaný výstup (debayering / náhled / radiance / odrazivost / index) do sdílené referenční geometrie pole pomocí transformace zaznamenané při pořízení. Vypnuto = export v nativní geometrii jednotlivých snímačů.

### Oříznout na společný překryv

* **Typ**: Zaškrtávací políčko (aktivní pouze při zapnuté funkci *Použít zarovnání pole*)
* **Výchozí nastavení**: Zapnuto (zaškrtnuto)
* **Uloženo jako**: `Processing → "Array alignment crop"`
* **Popis**: Ořízne zarovnané exporty na oblast, kterou sdílejí všechny kamerové moduly, takže každé pásmo má stejnou stopu. Vypnuto zachovává plnou plochu snímače (černá výplň mimo zdroj).

### Převzorkování

* **Typ**: Rozbalovací nabídka (aktivní pouze při zapnuté volbě *Použít zarovnání pole*)
* **Možnosti**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Výchozí**: Bilineární
* **Uloženo jako**: `Processing → "Array alignment interpolation"`
* **Popis**: Interpolace používaná při zarovnání. Volba Nejbližší zachovává přesné hodnoty zdroje (bez míchání mezi pixely) pro přesnou radiometrickou analýzu; volba Bilineární je nejvhodnější pro mapování a vizuální použití.

Stejné tři možnosti existují i bez předpony jako `chloros-cli process --array-alignment`, `--array-alignment-crop` a `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Index

Tato nastavení umožňují konfigurovat multispektrální indexy pro analýzu a vizualizaci.

### Přidat index

* **Typ**: Speciální panel pro konfiguraci indexů
* **Popis**: Otevře interaktivní panel, ve kterém můžete vybrat a nakonfigurovat multispektrální vegetační indexy (NDVI

, NDRE

, EVI

atd.), které se mají vypočítat během zpracování snímku. Můžete přidat více indexů, z nichž každý má vlastní nastavení vizualizace.
* **Dostupné indexy**: Rozbalovací nabídka grafického uživatelského rozhraní obsahuje**27** předdefinovaných vzorců multispektrálních indexů (úplný seznam, včetně názvů, které jsou akceptovány také volbou CLI

/SDK

`--indices`).
* **Funkce**:
  * Výběr z předdefinovaných vzorců indexů
  * Přetažení filtračních kanálů kamery do pozic pro jednotlivá pásma ve vzorci
  * Konfigurace barevných přechodů pro vizualizaci (LUT – Look-Up Tables)
  * Nastavte prahové hodnoty a režimy ořezávání
  * Vytvářejte vlastní vzorce indexů
* **Poznámka**: U jednopásmových černobílých kamer LATTICE M3M se indexy nepočítají — vícepásmové indexy nejsou pro jeden pásmo definovány. Kamery Survey3

a LATTICE M3C tím nejsou ovlivněny.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->



Každý index, který přidáte, vykreslí svůj vzorec jako matematický výraz s barevným kruhem pro každý slot pásma: červená = Red

, zelená = Green

, modrá = Blue

, oranžová = Orange

, azurová = Cyan

, fialová = NIR

, magenta = RE. Přetáhněte kruh z řádku nad vzorcem na slot, abyste jej přiřadili; dvojitým kliknutím na přiřazený slot jej vymažete. Index se vypočítá pouze tehdy, pokud každý slot, který vzorec používá, obsahuje kanál.

### Vlastní vzorce (funkce Chloros

+)

* **Typ**: Pole definic vlastních vzorců
* **Dostupnost**: Vyžaduje přihlášení s platným předplatným Chloros

+.
* **Popis**: Umožňuje vytvářet a ukládat vlastní vzorce multispektrálních indexů pomocí matematických operací s pásmy. Vlastní vzorce se ukládají spolu s nastavením projektu a lze je používat stejně jako vestavěné indexy.
* **Jak vytvořit**:
  1. V panelu konfigurace indexu otevřete kalkulátor vlastních vzorců
  2. Napište vzorec pomocí **symbolů pásem a slotů**, nikoli názvů pásem
  3. Vzorec uložte pod výstižným názvem – poté se zobrazí v dolní části rozevíracího seznamu vzorců a kruhy kanálů vaší kamery můžete přetáhnout do příslušných slotů přesně stejně jako u vestavěných předvoleb
* **Syntaxe vzorce**:
  * Sloty pásem: `x`, `y`, `z`, `a`, `b`, `c` — šest pozic, které přiřadíte ke skutečným kanálům přetažením
  * Operátory: `+`, `-`, `*`, `/`, `^` a `()` pro seskupování
  * Funkce: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Proč symboly a ne názvy kapel**: vzorec zapsaný jako `(y-x)/(y+x)` funguje na jakékoli kameře, protože přiřazení metodou „drag-and-drop“ určuje, zda `y` představuje 850 nm NIR

filtru RGN

, nebo 808 nm NIR

filtru OCN

. vestavěné předvolby jsou uloženy stejným způsobem — přesný symbolický tvar všech 27 předvoleb najdete v [Vzorce multispektrálních indexů](multispectral-index-formulas.md).
* **Kde fungují**: vlastní vzorce se ukládají spolu s nastavením projektu a lze je použít jak v [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md), tak při zpracování. Nejsou**akceptovány** v seznamu názvů CLI

/SDK

`--indices`, který rozšiřuje pouze 22 názvů vestavěných předvoleb.

***

## Export

Tato nastavení určují formát a kvalitu exportovaných zpracovaných snímků.

### Formát kalibrovaného obrázku

* **Typ**: Výběr z rozevíracího seznamu
* **Možnosti**:
  * **TIFF

(16bitový)** – nekomprimovaný 16bitový formát TIFF


  * **TIFF

(8bitový)** – nekomprimovaný 8bitový formát TIFF


  * **TIFF

(32bitový, procenta)** – 32bitový formát TIFF

s plovoucí desetinnou čárkou, kde jsou hodnoty odrazivosti vyjádřeny v procentech
  * **PNG

(8bitový)** – komprimovaný 8bitový formát PNG


  * **JPG (8bitový)** – komprimovaný 8bitový formát JPEG


* **Výchozí**: TIFF

(16bitový); u projektu, který je celý v 8bitovém formátu, se zobrazí TIFF

(8bitový) a jsou k dispozici pouze 8bitové formáty
* **Popis**: Volí formát souboru pro ukládání zpracovaných a kalibrovaných snímků. Exportované soubory se ukládají do podsložek podle formátu uvnitř složky příslušné kamery (`tiff16`, `tiff32`, `png8`, `jpg8`), přičemž na každý produkt připadá jedna složka `<Product>_Images/`. Exportované soubory si zachovávají původní název souboru – produkt identifikuje složka, nikoli přípona souboru.
* **Doporučené formáty**:
  * **TIFF

(16bitový)**: Doporučeno pro vědecké analýzy a profesionální pracovní postupy. Zachovává maximální kvalitu dat bez kompresních artefaktů. Nejvhodnější pro multispektrální analýzu a další zpracování v GIS softwaru.
  * **Formát TIFF

(32bitový, procenta)**: Nejvhodnější pro pracovní postupy, které vyžadují hodnoty odrazivosti vyjádřené v procentech (0–100 %). Nabízí maximální přesnost pro radiometrická měření.
  * **PNG

(8bitový)**: Vhodný pro prohlížení na webu a obecnou vizualizaci. Menší velikost souborů díky bezeztrátové kompresi, ale snížený dynamický rozsah.
  * **JPG (8bitový)**: Nejmenší velikost souborů, vhodný pouze pro náhledy a zobrazení na webu. Používá kompresi se ztrátou, která není vhodná pro vědeckou analýzu.
* **Poznámka**: Radiance z LATTICE se vždy exportuje jako 32bitový TIFF

s plovoucí desetinnou čárkou, bez ohledu na toto nastavení.

***

## Uložit šablonu projektu

Tato funkce vám umožňuje uložit aktuální nastavení projektu jako šablonu pro opakované použití.

* **Typ**: Textové pole + tlačítko Uložit
* **Popis**: Zadejte popisný název pro šablonu nastavení a klikněte na ikonu uložení. Šablona uloží všechna vaše aktuální nastavení projektu (detekce cílů, možnosti zpracování, indexy, a formát exportu) pro snadné opětovné použití v budoucích projektech. Šablony se ukládají do složky `Project Templates/` ve složce pro ukládání projektů a lze je také vybrat nebo exportovat z hlavního menu (*Vybrat šablonu* / *Uložit šablonu* / *Exportovat šablonu*).
* **Příklady použití**:
  * Vytvářejte šablony pro různé kamerové systémy (RGB

, multispektrální, NIR

)
  * Ukládejte standardní konfigurace pro konkrétní druhy plodin nebo analytické pracovní postupy
  * Sdílejte jednotná nastavení v rámci týmu
* **Jak používat**:
  1. Nakonfigurujte všechna požadovaná nastavení projektu
  2. Zadejte název šablony (např. „RedEdge

Survey3

NDVI

Standard“)
  3. Klikněte na ikonu uložení
  4. Šablonu lze nyní načíst při vytváření nových projektů

***

## Složka pro ukládání projektů

Toto nastavení určuje, kam se nové projekty standardně ukládají.

* **Typ**: Zobrazení cesty k adresáři + tlačítko Upravit
* **Výchozí (systém Windows

)**: `Documents\MAPIR\Chloros Projects` ve složce Dokumenty — obvykle `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`, případně tam, kam služba OneDrive nebo roamingový profil přesunula složku Dokumenty
* **Výchozí (Linux

)**: `~/Documents/MAPIR/Chloros Projects`
* **Popis**: Zobrazuje aktuální výchozí adresář, do kterého se vytvářejí nové projekty Chloros

. Kliknutím na ikonu úprav vyberte jiný adresář. Tato změna se ukládá jako jeden řádek textu v souboru `~/.chloros/working_directory.txt` — ve Windows

u se jedná o soubor `C:\Users\<Username>\.chloros\working_directory.txt`. Pokud tento soubor chybí nebo uvádí cestu, která již neexistuje, Chloros

se vrátí k výše uvedenému výchozímu nastavení. CLI

čte a zapisuje do stejného souboru, takže `chloros-cli` a grafické rozhraní se vždy shodují na umístění projektů.
* **Šablony projektů** se nacházejí v podsložce `Project Templates/` tohoto adresáře.
* **Kdy provést změnu**:
  * Nastavte síťovou jednotku pro týmovou spolupráci
  * Změňte na jednotku s větším úložným prostorem pro rozsáhlé datové sady
  * Uspořádejte projekty podle roku, klienta nebo typu projektu do různých složek
* **Poznámka**: Změna tohoto nastavení se týká pouze NOVÝCH projektů. Stávající projekty zůstanou ve svých původních umístěních.
* **Délka cesty ve Windows**: pokud nejsou povoleny dlouhé cesty ve Windows

(možnost **Povolit dlouhé cesty ve Windows** v instalačním programu), bude složka projektu s názvem delším než přibližně 130 znaků před zpracováním odmítnuta, protože její výstupy by překročily limit 260 znaků pro cestu ve Windows

. Viz [Spuštění zpracování](../processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

***

## Trvalost nastavení

Projekt Chloros

je **složka**. Veškerá nastavení projektu se ukládají do souboru `project.json` v ní; spolu s nimi se v souborech `cameras.json` a `sensors.json` ukládají i údaje o připojeném hardwaru, takže při opětovném otevření projektu dojde také k opětovnému připojení kamer a světelných senzorů. Při opětovném otevření projektu se všechna nastavení obnoví přesně tak, jak jste je zanechali. Uložené projekty lze také ovládat bez grafického rozhraní pomocí souboru `chloros-cli project` nebo souboru `open_project` z balíčku SDK

.

### Hierarchie nastavení

Nastavení se aplikují v následujícím pořadí:

1. **Výchozí nastavení systému** – Vestavěná výchozí nastavení definovaná společností Chloros

2. **Nastavení šablony** – Pokud při vytváření projektu načtete šablonu
3. **Nastavení uloženého projektu** – Nastavení uložená spolu se souborem projektu
4. **Ruční úpravy** – Jakékoli změny, které provedete během aktuální relace

### Nastavení a zpracování obrazu

Nastavení zpracování se načtou při spuštění zpracování. Změna nastavení nemá zpětný vliv na výstupy, které jsou již uloženy na disku – pro použití nových nastavení je třeba zpracování spustit znovu. Některá nastavení nemají na zpracování vůbec žádný vliv:

* Rozlišení miniatur obrázků (pouze pro zobrazení)
* Uložit šablonu projektu
* Uložit složku projektu

***

## Přehled konfiguračních klíčů

Pro automatizaci (CLI

`--config`, SDK

`configure` nebo přímé čtení `project.json`) jsou toto přesné klíče pod `Project Settings`:

| Cesta ke klíči | Typ | Výchozí hodnota |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | číslo 0–10 000 | `25` |
| `Target Detection → Minimum Target Clustering (0-100)` | číslo 0–100 | `60` |
| `Processing → Vignette correction` | bool | `true` |
| `Processing → Reflectance calibration / white balance` | bool | `true` |
| `Processing → Export sensor response` | bool | `true` |
| `Processing → Export vignette corrected` | bool | `true` |
| `Processing → Export debayered` | bool | `true` |
| `Processing → Export preview` | bool | `true` |
| `Processing → Export radiance` | bool | `true` |
| `Processing → Export reflectance` | bool | `true` |
| `Processing → Array alignment` | bool | `true` |
| `Processing → Array alignment crop` | bool | `true` |
| `Processing → Array alignment interpolation` | `"Bilinear" \| "Nearest" \| "Cubic"` | `"Bilinear"` |
| `Processing → Debayer method` | `"Standard (Fast, Medium Quality)" \| "Texture Aware (Slow, Highest Quality)"` | Standard |
| `Processing → Minimum recalibration interval` | číslo 0-3600 | `0` |
| `Processing → Light sensor timezone offset` | číslo -12..12 | `0` |
| `Processing → Apply PPK corrections` | bool | `false` |
| `Processing → DAQ cap id` | ID profilu limitu nebo `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | seznam konfigurací indexů | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (8-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

Klíče `Array alignment` se zapíší při prvním vykreslení sekce Array Alignment nebo při jejich nastavení voláním automatizace. Pokud chybí, potrubí používá stejné hodnoty, jaké jsou uvedeny výše (`true`, `true`, bilineární), takže projekt .json bez nich se chová stejně jako ten, který je obsahuje.

### Klíče uložené v `project.json` bez ovládacích prvků v panelu nastavení

Tyto klíče se nacházejí ve stejném stromu `Project Settings` a jsou načítány při zpracování, ale v postranním panelu pro ně nenajdete žádný ovládací prvek:

| Cesta ke klíči | Typ | Výchozí hodnota | Nastaveno |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK

`input_level=`. Přepisuje způsob interpretace vstupních souborů TIFF

v LATTICE; `auto` odvozuje z XMP tagu každého souboru `Chloros:ProcessingLevel` a počtu kanálů. Ignorováno u snímků `.raw` pořízených v režimu Survey3

. Záměrně se nejedná o nastavení v grafickém uživatelském rozhraní — v běžných případech je automatické nastavení správné. |
| `Processing → Target reflectance dir` | řetězec cesty | `""` | `chloros-cli process --target-reflectance-dir`, nebo cílové API

projektu |
| `Processing → Target reflectance config` | slovník s klíči podle sériového čísla kamery | `{}` | Registrace cíle v rámci snímku (režim `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | řetězec cesty | `""` | SDK

`process_folder(daq_log_path=…)`. Odkazuje na záznam `.daq` nebo na složku s takovými záznamy |
| `Target Detection → Minimum calibration target squares` | číslo | `4` | Starší výchozí nastavení; bez ovládacích prvků a bez příznaku CLI

|
| `UI → Grid thumbnail size` | číslo | `160` | Vlastní posuvník pro přiblížení miniatur v mřížce obrázků |

Dvě předvolby prohlížeče jsou uloženy **na nejvyšší úrovni v `project.json`**, zcela mimo `Project Settings`, protože se jedná spíše o stav zobrazení než o nastavení zpracování:

| Cesta ke klíči | Typ | Výchozí hodnota | Nastaveno |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | celé číslo 1–256 | `1` | Ovládací prvek GSD (px) na kartě obrázku — viz [Otevření obrázku na celou obrazovku](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Osvědčené postupy

1. **Začněte s výchozími nastaveními**: Výchozí nastavení funguje dobře pro většinu kamerových systémů MAPIR

a typických pracovních postupů.
2. **Vytvářejte šablony**: Jakmile optimalizujete nastavení pro konkrétní pracovní postup nebo kameru, uložte je jako šablonu, abyste zajistili jednotnost napříč projekty.
3. **Otestujte před úplným zpracováním**: Při experimentování s novými nastaveními nejprve otestujte na malé podskupině snímků, než zpracujete celý datový soubor.
4. **Zaznamenejte si nastavení**: Používejte popisné názvy šablon, které uvádějí kamerový systém, typ zpracování a zamýšlené použití (např. „Survey3

_RGB_NDVI_Agriculture“).
5. **Výběr formátu exportu**: Formát exportu vyberte podle konečného použití:
   * Vědecká analýza → TIFF (16bitový nebo 32bitový)
   * Zpracování v GIS → TIFF

(16bitový)
   * Rychlá vizualizace → PNG

(8bitový)
   * Sdílení na webu → JPG (8bitový)

***

Další informace o multispektrálních indexech v programu Chloros

najdete na stránce [Vzorce multispektrálních indexů](multispectral-index-formulas.md).
