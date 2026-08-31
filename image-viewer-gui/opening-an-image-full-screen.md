# Otevření obrázku na celou obrazovku

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption><p>Obrázek otevřený na celou obrazovku, s výběrem vrstev v pravém horním rohu</p></figcaption></figure>

Prohlížeč obrázků Chloros je rozhraní na celou obrazovku určené k prohlížení, kontrole a měření vašich obrázků. Právě zde můžete číst **skutečné hodnoty pixelů** — DN pro každý kanál, procentuální odrazivost nebo radianci v W/m²/sr/nm — namísto roztaženého náhledu, který se zobrazuje na obrazovce.

## Spuštění prohlížeče obrázků

### Z prohlížeče souborů

1. Otevřete kartu **Prohlížeč souborů** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Klikněte na libovolnou **miniaturu** v [mřížce snímků](image-grid.md)
3. Snímek se otevře na celé obrazovce na záložce **Prohlížeč snímků**

Snímek se otevře v tom produktu, který byl zobrazen v mřížce. Pokud je mřížka nastavena na `RAW (Reflectance)`, otevře se právě tato vrstva.

### Otevření postranního panelu prohlížeče obrázků

Kliknutím na ikonu **Prohlížeč obrázků** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> v levém postranním panelu vysunete panel analýzy. Obsahuje (shora dolů):

* název snímku a model fotoaparátu
* tlačítko **Exportovat/Uložit snímky** (pouze pokud je aktivní index nebo LUT)
* zaškrtávací políčka **Index**a**LUT** a panel konfigurace indexu — viz [Index/LUT Sandbox](index-lut-sandbox.md)
* panel **Hodnoty kurzoru**: odečet podle kanálů, histogram vrstvy a ovládací prvek GSD***

## Navigace a přiblížení

### Procházení snímků

* **Další snímek**: tlačítko → nebo klávesa**→** (šipka vpravo)
* **Předchozí snímek**: tlačítko ← nebo klávesa**←** (šipka vlevo)
* **Přeskočení na konkrétní snímek**: vraťte se do mřížky a klikněte na jeho miniaturu

Přiblížení a posun zůstávají zachovány při přecházení mezi snímky, takže můžete procházet sadu snímků a přitom zůstat na stejné části snímku.

### Přiblížení

Přiblížení se ovládá pomocí **kolečka myši** v krocích po 15 %, s ukotvením na kurzoru — bod pod ukazatelem zůstává pod ukazatelem. Rozsah je omezen velikostí obrázku a okna: nelze oddálit více, než je velikost okna, a horní hranice je dána nativním rozlišením obrázku.

V prohlížeči na celou obrazovku nejsou žádné speciální klávesy pro změnu měřítka. (V mřížce slouží kombinace **Ctrl + `+` / `−`** ke změně velikosti miniatur – jedná se o jinou funkci.)

### Posun při přiblížení

Klikněte a podržte levé tlačítko myši nad obrázkem a táhněte. Posun je omezen, takže obrázek nelze přetáhnout mimo obrazovku.

### Kontrola jednotlivých pixelů při vysokém zvětšení

Jakmile efektivní zvětšení překročí **60×**, Chloros nakreslí zvýrazňovací rámeček kolem jednotlivého zobrazeného pixelu pod kurzorem a vedle něj se zobrazí plovoucí hodnota.

„Efektivní“ zvětšení zohledňuje velikost bloku GSD: při velikosti bloku 8 se zvýraznění objeví již při zvětšení 7,5×, nikoli až při 60×, protože jeden zobrazený pixel již odpovídá 8 × 8 zdrojovým pixelům. Pokud se vrátíte k menšímu zvětšení pod tuto prahovou hodnotu, zvýraznění zmizí.

### Klávesové zkratky

| Klávesa                             | Kde       | Akce                              |
| ------------------------------- | ----------- | ----------------------------------- |
| **→**                           | Celá obrazovka | Další obrázek                          |
| **←**                           | Celá obrazovka | Předchozí obrázek                      |
| **Ctrl + R**                    | Celá obrazovka | Obnovit index/LUT sandbox         |
| **Ctrl + `+`**/**Ctrl + `=`** | Mřížka        | Větší miniatury (4 px za stisk)  |
| **Ctrl + `−`**                  | Mřížka        | Menší miniatury (4 px za stisk) |***

## Hodnoty kurzoru

Pohybujte kurzorem po obrázku a panel **Hodnoty kurzoru** zobrazí hodnotu každého kanálu pod ním.

{% hint style="success" %}
**Jedná se o skutečné hodnoty souboru.** Plátno na obrazovce je 8bitový roztažený náhled a nemůže tyto hodnoty poskytnout, proto Chloros pro zobrazení vzorkuje skutečný produktový soubor. Proto 12bitový surový snímek zobrazuje hodnoty nad 255 a vrstva s jasem typu float32 zobrazuje fyzikální jednotky.
{% endhint %}

### Co znamenají jednotlivé sloupce

Panel se přizpůsobuje vrstvě, kterou právě prohlížíte:

| Prohlížená vrstva              | Zobrazené sloupce    | Poznámky                                                                                           |
| ---------------------------------- | ---------------- | ----------------------------------------------------------------------------------------------- |
| Odrazivost                        | **DN**a**%** | Procento se počítá podle vlastního měřítka daného souboru — viz níže                                      |
| Radiance                           | **W/m²/sr/nm**   | Fyzikální hodnoty typu float; sloupec DN není k dispozici, protože DN zde nemá smysl                           |
| Raw / Debayered / náhled / JPG    | **DN**           | Celá čísla                                                                         |
| 32bitové exporty procentuální odrazivosti | pouze **%**       | Uložená hodnota typu float není DN, takže zaokrouhlení na celé číslo by vedlo k zobrazení nesmyslného `0` nebo `1` |

Každý řádek je označen názvem kanálu filtru vaší kamery — `Red / Green / NIR` pro RGN, `Orange / Cyan / NIR` pro OCN, `NIR / Green / Blue` pro NGB, `Red / Green / Blue` pro RGB a název jediného pásma pro kamery RE, NIR a pro monokromní kamery M3M. Každý štítek nese barevnou tečku, která odpovídá kruhům kanálů používaným v editoru indexových vzorců.

Uložené obrázky **indexu a LUT** představují zvláštní případ: obsahují komponenty barevné mapy namísto spektrálních pásem, takže jejich řádky jsou označeny jako `Red / Green / Blue` (nebo `Index` u jednokanálového indexového souboru) namísto názvů filtrů kamery.

Je-li v testovacím prostředí aktivní index, objeví se pod kanály další řádek zobrazující **hodnotu indexu** v místě kurzoru, spolu s názvem indexu a bílou tečkou, která odpovídá jeho značce na histogramu.

### Procentuální odrazivost používá vlastní stupnici každého souboru

{% hint style="warning" %}
**Nepředpokládejte, že 65535 = 100 %.** Chloros ukládá odrazivost v různých měřítkách v závislosti na tom, který fotoaparát ji vyprodukoval, a prohlížeč pro každý soubor určí to správné.
{% endhint %}

| Zdroj                  | Hodnota DN odpovídající odrazivosti 1,0 | Jak se identifikuje                                                                                                                               |
| ----------------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **LATTICE**(M3C / M3M) |**32768**                      | XMP tag `Chloros:PixelScale=32768` zapsaný do každého exportu odrazivosti LATTICE. 2× rezerva umožňuje souboru nést ρ nad 1,0 bez oříznutí |
| **Survey3**|**65535**                      | Žádný XMP tag měřítka Chloros — kalibrace Survey3 zapisuje ρ × dtype-max a ořezává na hodnotu 1,0                                                               |

Prohlížeč, index/LUT sandbox a export indexu všechny řeší škálování prostřednictvím stejné jediné implementace, takže hodnota, kterou přečtete na kurzoru, je stejná jako hodnota použitá v indexových výpočtech.

Dva důsledky, o kterých je dobré vědět:

* **32bitové procentní**TIFF ukládá hodnotu DN/65535 jako číslo typu float a**8bitové** PNG/JPG ukládá hodnotu DN × 255/65535 — prohlížeč před zobrazením procenta obě hodnoty převede zpět.
* Jeden případ nelze obnovit: **8bitový export TIFF ze snímku s 8bitovým zdrojem** je oříznut na rozsah 0–255 namísto přepočítání a záměrně neobsahuje žádný tag měřítka. U těchto souborů panel zobrazuje pouze hodnotu DN, bez sloupce s procenty. Toto je upřímná odpověď, nejedná se o chybu.***

## Histogram vrstvy

Pod řádky kurzoru se nachází živý histogram vrstvy, kterou právě prohlížíte, v **256 bodech**. Ve výchozím nastavení vykresluje jednu kombinovanou křivku, váženou `(R + 2G + B) / 4` — stejný měřicí prostor, jaký používají histogramy kamery LATTICE. Zapnutím**RGB** se tato křivka nahradí křivkami pro jednotlivé kanály v barvách kanálů, které jsou aditivně smíchány, aby překryvy zůstaly čitelné. Mono vrstvy vždy vykreslují jedinou křivku.

Vodorovná osa je v jednotkách dané vrstvy:

| Vrstva       | Jednotka osy  | Maximální hodnota osy                                               |
| ----------- | ---------- | ---------------------------------------------------------- |
| Odrazivost | procenta    | 125 % — rezerva produktu umožňuje ρ nad 1,0           |
| Radiance    | W/m²/sr/nm | Vlastní maximum snímku, zaokrouhleno nahoru na dvě platné číslice |
| 8bitová data  | DN         | 255                                                        |
| 12bitová data | DN         | 4095                                                       |
| 16bitová data | DN         | 65535                                                      |

Pokud je osa nastavena na DN a dosáhne jedné z těchto tří horních hranic, Chloros také rozpozná bitovou hloubku zobrazeného obrazu.

Nad histogramem se nacházejí tři tlačítka:

| Tlačítko     | Výchozí nastavení | Účinek                                                                                                                                                                                                                                                                                   |
| ---------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **KURZOR** | Zapnuto | Nakreslí na histogramu značky v přesných hodnotách uvedených v řádcích výše, takže můžete vidět, kde se pixel pod kurzorem nachází v rozložení snímku. V režimu RGB je pro každý kanál jedna značka ve své vlastní barvě; v ostatních případech je zobrazena jedna bílá značka na kombinované hodnotě |
| **INDEX**| Zapnuto      | Zobrazuje se pouze při aktivním indexu. Přepne histogram ze zdrojových pásem na**rozdělení indexových hodnot**, přičemž obě prahové hodnoty oříznutí jsou znázorněny oranžovými přerušovanými čarami a indexová hodnota kurzoru bílou čarou                                                          |
| **RGB**| Vypnuto     | Přepne z kombinované křivky na křivky pro jednotlivé kanály. U monochromatického snímače se na tomto tlačítku zobrazuje**MONO** a je deaktivováno — k zobrazení je k dispozici pouze jeden kanál                                                                                                                                  |

Histogram se počítá na základě **viditelných bloků**, nikoli zdrojových pixelů za nimi: při změně velikosti bloku GSD se rozložení přepočítá, takže histogram, značka kurzoru a zobrazený obrázek se vždy shodují.***

## Velikost bloku GSD

Ve spodní části panelu se nachází ovládací prvek **GSD (px)**: číselné pole, posuvník v rozsahu**1 až 256**a tlačítko**RESET**.

Zhrubšuje _zobrazený_ obrázek tím, že zprůměruje blok zdrojových pixelů o rozměrech N × N do jednoho zobrazeného pixelu. `1` je nativní rozlišení.

* Ovlivňuje **zobrazení na celé obrazovce, miniatury v mřížce, údaje kurzoru a oba histogramy** — vše, co zobrazuje obrázek, používá stejné základní rozlišení.
* Jedná se **pouze o zobrazení**. Zpracování a export zůstávají nedotčeny. Jediná výjimka je záměrná: export do [Index/LUT Sandbox](index-lut-sandbox.md) uloží to, na co se právě díváte, takže přebírá aktuální velikost bloku, a panel exportu vás upozorní, pokud je velikost bloku vyšší než 1.
* Hodnota je uložena **pro každý projekt zvlášť** jako `viewer_display.gsd_bin` v souboru `project.json`, takže zůstává zachována i po zavření a opětovném otevření.
* Pokud je velikost bloku větší než 1, kurzor zobrazuje hodnotu bloku, nikoli zdrojového pixelu – zobrazená hodnota je průměr bloku pod vaším kurzorem.

{% hint style="info" %}
**Proč „velikost bloku“ a ne centimetry na pixel?** Hodnota v cm/px vyžaduje údaj o výšce nad zemí. EXIF údaje jednoho snímku obsahují GPS nadmořskou výšku nad průměrnou hladinou moře, nikoli nad terénem, na který byl fotoaparát namířen, proto Chloros nevytiskne vzdálenost k zemi, kterou nemůže spolehlivě odvodit. Velikost bloku ve zdrojových pixelech je stejným náhradním řešením, jaké používají nástroje pro práci s mraky v MAPIR, když není známa vzdálenost vzorku k zemi.
{% endhint %}

***

## Typy obrázků, které lze zobrazit

Rozbalovací nabídka vrstev v pravém horním rohu prohlížeče obsahuje seznam všech verzí aktuálního obrázku. To, které položky se zobrazí, závisí na kameře a na tom, co bylo zpracováno — úplný seznam a informace o fungování rozevíracího seznamu najdete v části [Obrázkové vrstvy](image-layers.md).

### Survey3

* **JPG** — vlastní náhledový soubor kamery
* **RAW (Original)** — zdrojový soubor `.RAW`, zbavený bayerského filtru pro zobrazení, bez korekcí
* **RAW (Target)** — snímek identifikovaný jako obsahující kalibrační cíl
* **RAW (odrazivost)** — kalibrovaný produkt odrazivosti (65535 = ρ 1,0)
* **S korekcí vinětace**/**Odpověď snímače** — nekalibrovaný záložní produkt
* **S vyvážením bílé** — produkt s vyvážením bílé
* **RAW (`<INDEX>` Index)**a**`<INDEX>` LUT** — vypočítané indexové snímky

### LATTICE

Snímky LATTICE využívají stejné rozbalovací menu s názvy úrovní zpracovatelského řetězce:

| Vrstva                 | Co obsahuje                                                        |
| --------------------- | -------------------------------------------------------------------- |
| **RAW (originál)**    | Zdrojový snímek ve formátu RAW v době pořízení                                     |
| **RAW (bez bayerského filtru)**   | Lineární obraz bez bayerského filtru                                           |
| **RAW (Náhled)**     | Náhled na displeji — roztažení do falešných barev pro multispektrální kamery |
| **Vyvážení bílé**    | Náhled na displeji pro hlavní kamery RGB (vyvážení bílé + gama)   |
| **RAW (zář)**    | Spektrální zář typu Float32 v jednotkách W/m²/sr/nm                              |
| **RAW (odrazivost)** | Odrazivost typu uint16, 32768 = ρ 1,0                                    |

Zářivost a odrazivost jsou k dispozici pouze v multispektrálním režimu: hlavní kamera RGB nemá radiometrii pro jednotlivá pásma, takže tyto vrstvy pro ni nejsou generovány.

***

## Použití indexů a LUT

Multispektrální indexy a barevné vyhledávací tabulky (LUT) aplikujte z postranního panelu:

1. Otevřete postranní panel **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Zaškrtněte **Index**

3. Vyberte filtr své kamery a vzorec indexu, poté přetáhněte kruhy kanálů do příslušných polí vzorce
4. Přidejte LUT a vyberte gradient, prahové hodnoty a režim ořezání
5. Přečtěte si hodnoty v místě kurzoru a výsledek uložte pomocí **Export/Save Image(s)**Kompletní návod najdete na stránce [Index/LUT Sandbox](index-lut-sandbox.md).***

## Řešení problémů

### Obrázek se neotevře

**Možné příčiny**: soubor byl po importu přesunut nebo smazán; produkt nebyl nikdy zapsán; nedostatek paměti pro velmi velký obrázek.**Co dělat**:

1. Zkontrolujte, zda soubor vrstvy stále existuje ve výstupním stromu projektu
2. Otevřete soubor v externím prohlížeči a ověřte, zda je neporušený
3. Ukončete ostatní aplikace, abyste uvolnili paměť

### Obrázek je černý, bílý nebo má nepřirozené barvy

**Možné příčiny**: roztažení obrazu nemá s čím pracovat (téměř konstantní snímek); vrstva typu float32 s neobvyklými hodnotami; index, který nevytvořil žádná platná data.**Co dělat**:

1. Zkontrolujte hodnoty kurzoru — pokud je každý kanál na nule nebo blízko nuly, problém je v datech, nikoli ve zobrazení
2. Zkontrolujte histogram: jediný výkyv na jednom konci znamená, že snímek je oříznutý nebo prázdný
3. Zkontrolujte protokol zpracování pro běh, který vrstvu vygeneroval

### Hodnoty vypadají nesprávně

**Možné příčiny**: pracujete s jinou vrstvou, než si myslíte; porovnáváte procenta s nezpracovanou hodnotou DN; porovnáváte soubor LATTICE se souborem Survey3 při použití stejného dělitele.**Co dělat**:

1. Ověřte vybranou vrstvu v rozevíracím seznamu – jednotky v panelu se řídí vrstvou
2. U odrazivosti použijte sloupec **%** namísto toho, abyste hodnotu DN dělili sami; pokud musíte hodnotu dělit, použijte hodnotu `Chloros:PixelScale` daného souboru (32768 pro LATTICE, pokud není uveden, znamená to 65535 pro Survey3)
3. Nastavte velikost bloku GSD zpět na 1 – při hodnotě vyšší než 1 čtete průměr bloku, nikoli hodnotu pixelu
4. Zkontrolujte, zda pro daný snímek skutečně proběhla kalibrace odrazivosti; nekalibrovaný náhradní produkt (Sensor Response / Vignette Corrected) není odrazivostí

***

## Další kroky

* [**Obrázkové vrstvy**](image-layers.md) — názvy všech vrstev (pokud existují) a význam jejich hodnot
* [**Index/LUT Sandbox**](index-lut-sandbox.md) — vytváření, ladění a export vizualizací indexů
* [**Mapové značky**](map-markers.md) — stejná sada snímků na mapě
* [**Vzorce multispektrálních indexů**](../project-settings/multispectral-index-formulas.md) — reference k indexům

Informace o pracovním postupu zpracování najdete v části [Zpracování snímků (GUI)](../processing-images-gui/adding-files-to-a-project.md).
