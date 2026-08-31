# Mřížka obrázků

Po importu obrázků do projektu se v hlavní oblasti zobrazí uspořádané do mřížky. V této mřížce si vyberete, **kterou verzi každého obrázku si právě prohlížíte** — tlačítka nad ní přepínají všechny miniatury najednou mezi zdrojovými soubory a jednotlivými zpracovanými výstupy.

## Velikost miniatur

Pomocí posuvníku zvětšení v pravém horním rohu můžete upravit velikost miniatur obrázků. Rozsah posuvníku je od **64 px do 1200 px**.

* **Kombinace kláves Ctrl + kolečko myši** také mění měřítko miniatur.
* **Ctrl + `+`**/**Ctrl + `=`**a**Ctrl + `−`** mění velikost o 4 px při každém stisknutí. Rozsah klávesových zkratek končí na 64 px na spodní hranici a na horní hranici na jakékoli velikosti, která se přesně vejde do dvou miniatur v jednom řádku v aktuálním okně.
* Velikost, kterou zvolíte, se uloží spolu s projektem (`UI → Grid thumbnail size` v `project.json`, výchozí `160`), takže při opětovném otevření projektu se obnoví.

<figure><img src="../.gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>*Rozlišení* miniatur je nastavení odlišné od *velikosti* miniatur: viz **Zobrazení → Rozlišení miniatur obrázků** v [Nastavení projektu](../project-settings/project-settings.md) (výchozí hodnota 512 px na delší straně). Velikost udává, jak velká je dlaždice vykreslena; rozlišení udává, kolik detailů se načte k jejímu vyplnění.***

## Panel nástrojů mřížky

Řada tlačítek nad mřížkou obsahuje až tři skupiny, zleva doprava:

1. **Podle spouštěče / Podle kamery** — režim seskupení. Zobrazuje se pouze u projektů obsahujících snímky z LATTICE.
2. **Tlačítka filtru kamer** — jedno pro každou kameru LATTICE. Zobrazují se pouze v režimu „Na kameru“.
3. **Tlačítka režimu exportu/zobrazení** — určují, který produkt se u jednotlivých miniatur zobrazí.

Pokud je okno příliš úzké na to, aby se vešly všechny, skupiny se sbalí zprava doleva do rozbalovacích nabídek při najetí myší: nejprve se sbalí tlačítka exportu/zobrazení, poté tlačítka kamer. Sbalená skupina zanechá jediné tlačítko označené aktuálně aktivní volbou a při najetí myší se posune celá sada dolů. **Skupiny „Per Trigger“ a „Per Camera“ se nikdy nesbalí.

<!-- SCREENSHOT-NEEDED: Image grid toolbar of a LATTICE array project at full width, showing all three button groups inline: Per Trigger / Per Camera, three camera filter buttons labelled "LATT-M3M (serial)", and the export/view buttons including TIFF, RAW (Original), RAW (Radiance), RAW (Reflectance). -->

*****

## Tlačítka pro export a zobrazení

Tato tlačítka přepínají miniatury v mřížce mezi typy obrázků. **Tlačítko se zobrazí, jakmile existuje produkt, jehož název nese** — což u zdrojových souborů znamená okamžitě při importu, nikoli až po zpracování. Chloros znovu prohledává produkty projektu během probíhajícího cyklu, takže tlačítka se objevují během zpracování, jakmile se jednotlivé produkty začnou ukládat na disk.

### Základní tlačítko

Tlačítko exportu zcela vlevo je označeno podle **toho, co jste skutečně importovali**:

| Co jste importovali | Popisek tlačítka |
| --- | --- |
| Survey3 RAW+JPG | `JPG` |
| Snímky LATTICE s náhledem na displeji vedle snímku ve formátu RAW | `PNG` nebo `TIFF`, podle toho, jaké jsou náhledy |
| Snímky LATTICE, kde je základním souborem **právě** snímek ve formátu RAW | *žádné tlačítko* — `RAW (Original)` již tento soubor zobrazuje |

Ve smíšeném projektu se název tlačítka řídí podle přípony, kterou používá většina snímků.

### Tlačítka produktů

| Tlačítko | Zobrazuje | Kdy se zobrazí |
| --- | --- | --- |
| **Cíle** | Snímky s detekovaným kalibračním cílem | Po běhu, při kterém byly detekovány cíle |
| **Odrazivost** | Kalibrované snímky odrazivosti | Pouze projekty Survey3 — projekty LATTICE místo toho používají `RAW (Reflectance)`, takže mřížka nikdy nezobrazuje dvě tlačítka odrazivosti |
| **Vyvážení bílé** | Výstup s vyvážením bílé (kamery RGB) | Po zpracování |
| **Korekce vinětace** | Nekalibrovaná záložní varianta s korekcí vinětace | Po běhu, při kterém nebylo možné použít kalibraci odrazivosti a byla zapnuta *korekce vinětace* |
| **Odezva snímače** | Nekalibrovaná záložní varianta s odezvou snímače | Stejné, ale s vypnutou *korekcí vinětace* |
| **`RAW (<INDEX> Index)`** | Jedno tlačítko pro každý vypočítaný index | Po běhu s nakonfigurovanými indexy |
| **`<INDEX> LUT`** | Jedno tlačítko pro každý index s barevnou mapou | Po běhu s nakonfigurovanou LUT |
| **`<Index> <Index\|LUT> <NNN>`** | Jedno tlačítko pro každý exportní běh [Index/LUT Sandbox](index-lut-sandbox.md) | V okamžiku dokončení exportu do sandboxu |

### Tlačítka úrovní LATTICE

Projekty obsahující záznamy LATTICE přidávají tato tlačítka, označená názvem úrovně namísto názvu produktu:

| Tlačítko | Úroveň |
| --- | --- |
| **RAW (Originální)** | Zdrojový surový snímek, tak jak byl importován |
| **RAW (Radiance)** | Spektrální radiance typu Float32, W/m²/sr/nm |
| **RAW (Reflectance)** | Odrazivost typu uint16, 32768 = ρ 1,0 |

`RAW (Original)` je k dispozici ihned po importu — nevyžaduje žádné zpracování. Pokud import LATTICE nemá vůbec žádné základní tlačítko (základním souborem každého snímku je jeho surový snímek), mřížka se přesune na první dostupné tlačítko úrovně, aby zvýraznění na panelu nástrojů odpovídalo tomu, co vidíte.

Dvouúrovňové exporty Chloros **nemají vlastní tlačítko mřížky**:

* **Debayered** — zobrazení `RAW (Original)` je již vykresleno bez bayesovského filtru, takže druhé tlačítko na vizuálně identickém obrázku by bylo zbytečné. Produkt `RAW (Debayered)` se stále ukládá na disk a lze jej stále vybrat z rozevíracího seznamu vrstev na celé obrazovce.
* **Náhled** — u kamer RGB je náhled zaregistrován jako vrstva `White Balanced`, která tlačítko má. U multispektrálních kamer je zaregistrován jako `RAW (Preview)` a je přístupný z rozevíracího seznamu vrstev na celé obrazovce.

{% hint style="info" %}
Tato tlačítka úrovní se vykreslují pouze u projektů, které skutečně obsahují snímky LATTICE. Projekty typu Survey3 registrují některé stejné interní názvy vrstev a tlačítka jsou u nich odfiltrována, takže mřížka typu Survey3 si zachovává svou obvyklou sadu `JPG / Targets / Reflectance`.
{% endhint %}

Kliknutím na miniaturu mřížky se otevře [Prohlížeč obrázků](opening-an-image-full-screen.md) na celé obrazovce u **stejného produktu, který mřížka zobrazuje** — pokud je mřížka nastavena na `Targets`, miniatura otevře exportovaný cílový obrázek.

<figure><img src="../.gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: This GIF predates the LATTICE level buttons and the toolbar group separators. Reshoot on a LATTICE project cycling base -> RAW (Original) -> RAW (Radiance) -> RAW (Reflectance) -> an index button, so the new button set and the level names are visible. -->

***

## Seskupování projektu LATTICE: Podle spouštěče vs. podle kamery

Snímky z pole zachycují několik obrázků stejného okamžiku z různých kamerových modulů. Seskupování určuje, jak je mřížka tyto snímky uspořádá. V obou režimech se zobrazují sbalitelné záhlaví přes celou šířku; **každá skupina je zpočátku rozbalená** a Chloros si pamatuje ty, které zavřete. Stav sbalení se sleduje odděleně pro každý režim, takže zavření skupiny v režimu „podle kamery“ nezavře nic v režimu „podle spouště“.

### Podle kamery (výchozí)

Jedna skupina na jeden kamerový modul. Záhlaví zobrazuje model kamery a sériové číslo (`LATT-M3M — <serial>`) a počet fotografií. Snímky uvnitř skupiny jsou seřazeny chronologicky podle události pořízení.

V tomto režimu se na panelu nástrojů také zobrazí jedno **tlačítko filtru kamery pro každou kameru** s označením `MODEL (SERIAL)`. Všechny kamery jsou zpočátku vybrané; kliknutím na tlačítko zrušíte výběr dané kamery a odstraníte její skupinu z mřížky. Jedná se o rychlý způsob, jak zkontrolovat jedno pásmo z celého letu.

### Podle spouštěcí události

Jedna skupina na jednu spouštěcí událost — soubor snímků, které všechny moduly pořídily při stejné spouštěcí události. V záhlaví se zobrazuje čas pořízení, počet zapojených kamer a značka pro každý model kamery ve skupině. Dlaždice uvnitř skupiny jsou seřazeny podle sériového čísla kamery, takže stejné pásmo se u každého spouštěcího signálu nachází ve stejném sloupci.

<!-- SCREENSHOT-NEEDED: Image grid in Per Trigger mode for a 3-camera LATTICE array, showing two consecutive trigger groups with their header bars (chevron, capture timestamp, "3 cameras", and the three model badges) and one group collapsed to show the closed state. -->
Snímky, které nejsou ve formátu LATTICE, nejsou ve smíšeném projektu seskupeny — zobrazují se jako běžné dlaždice za skupinami.

***

## Miniatury v mřížce se řídí velikostí bloku GSD

Pokud jste v postranním panelu záložky „Snímek“ nastavili velikost bloku **GSD (px)**, miniatury mřížky se zobrazují ve stejném rozlišení na zemi – nejen v zobrazení na celou obrazovku. Velikost bloku 8 znamená, že každý zobrazený pixel je průměrem bloku 8 × 8 zdrojových pixelů, a to všude v aplikaci, kde se obrázek zobrazuje.

Vzhledem k tomu, že dlaždice má zpočátku šířku jen několik set pixelů, hrubé velikosti bloků přestávají mít viditelný vliv na mřížku mnohem dříve než v zobrazení na celou obrazovku: rámeček o velikosti 4000 px vykreslený do dlaždice o velikosti 160 px již představuje přibližně 25 zdrojových pixelů na jeden zobrazený pixel. Viz [Otevření obrázku na celou obrazovku](opening-an-image-full-screen.md#gsd-block-size) pro samotný ovládací prvek.

***

## Související stránky

* [**Otevření obrázku na celou obrazovku**](opening-an-image-full-screen.md) — prohlížeč na celou obrazovku, hodnoty kurzoru a histogram
* [**Vrstvy obrázku**](image-layers.md) — rozevírací nabídka vrstev v prohlížeči na celou obrazovku
* [**Index/LUT Sandbox**](index-lut-sandbox.md) — vytváření a export vizualizací indexu
* [**Nastavení projektu**](../project-settings/project-settings.md) — přepínače exportu, které určují, které produkty vůbec existují
