# Otevření obrázku na celou obrazovku

Prohlížeč obrázků Chloros poskytuje speciální rozhraní na celou obrazovku pro prohlížení, analýzu a manipulaci s multispektrálními obrázky. Ať už prohlížíte původní obrázky nebo zpracované výstupy, prohlížeč obrázků nabízí výkonné nástroje pro kontrolu a analýzu.

## Přístup k prohlížeči obrázků

### Z prohlížeče souborů

Nejběžnější způsob otevření obrázku v prohlížeči obrázků:

1. Ujistěte se, že jste na kartě **Prohlížeč souborů**. <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Klikněte na libovolnou **miniaturu obrázku** v mřížce obrázků.
3. Obrázek se otevře v **hlavní oblasti náhledu** (uprostřed obrazovky).
4. Obrázek je nyní načten a připraven k prohlížení na celé obrazovce.

### Otevření karty prohlížeče obrázků

Jakmile je obrázek načten v oblasti náhledu:

1. Klikněte na ikonu **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> v levém postranním panelu.
2. Otevře se karta Prohlížeč obrázků, která zobrazí vybraný obrázek na celé obrazovce.
3. V levém postranním panelu se zobrazí pokročilé nástroje pro prohlížení a analýzu.

***

## Přehled rozhraní prohlížeče obrázků

### Hlavní zobrazovací oblast

Největší část obrazovky zobrazuje váš obrázek:

* **Plné rozlišení**: Obrázky se zobrazují v nativním rozlišení.
* **Zoomovatelné**: K přiblížení použijte ovládací prvky nebo kolečko myši
* **Posuvné**: Po přiblížení klikněte a tažením se pohybujte po obrázku
* **Zachování poměru stran**: Obrázky se mění proporcionálně

***

## Možnosti prohlížení

### Základní navigace v obrázcích

#### Procházení obrázků

V sadě obrázků se pohybujte pomocí klávesových zkratek nebo tlačítek:

* **Další obrázek**: Klikněte na tlačítko → nebo stiskněte klávesu **→** (šipka doprava)
* **Předchozí obrázek**: Klikněte na tlačítko ← nebo stiskněte klávesu **←** (šipka doleva)
* **Přejít na konkrétní obrázek**: Vraťte se do prohlížeče souborů a klikněte na požadovanou miniaturu

#### Ovládací prvky pro přiblížení

Upravte zvětšení pro prohlížení detailů obrázku:

**Přiblížení:**

* Klikněte na tlačítko **+** (plus)
* Stiskněte klávesu **+** nebo **=**
* Posuňte kolečko myši **nahoru**

**Oddalení:**

* Klikněte na tlačítko **−** (mínus)
* Stiskněte klávesu **−** (mínus)
* Posuňte kolečko myši **dolů**

**Přizpůsobit obrazovce:**

* Klikněte na tlačítko **↔** (Přizpůsobit)
* Stiskněte klávesu **0** (Nula)
* Poklepejte na obrázek

#### Posouvání při přiblížení

Při přiblížení nad velikost obrazovky:

1. Přesuňte kurzor myši nad obrázek
2. Klikněte a **podržte levé tlačítko myši**
3. **Přetažením** přesuňte obrázek
4. Uvolněním zastavíte posun

**Alternativa**: Pomocí kláves se šipkami posouvejte obrázek po malých krocích

***

## Kontrola hodnoty pixelů

### Zobrazení hodnot pixelů u kurzoru

Při přesunutí kurzoru myši nad obrázek se v reálném čase zobrazí hodnoty pixelů:

**Umístění zobrazení hodnoty:**

* **Plovoucí číslo a červená čára v legendě gradientu indexu LUT na pravé straně**
* **Při dalším přiblížení se zobrazí plovoucí hodnota v blízkosti kurzoru a zvýrazněného pixelu**
* Zobrazuje hodnoty pro pixel **pod kurzorem nebo zvýrazněný**
* Aktualizuje se při pohybu myši

***

## Typy obrázků, které můžete zobrazit

### Původní obrázky (před zpracováním)

**Obrázky RAW + JPG z fotoaparátu:**

* Zobrazení dat RAW jako náhled
* Zobrazení původních, neopravených hodnot
* Užitečné pro kontrolu kvality obrázku před zpracováním

### Kalibrované obrázky odrazivosti

**Po zpracování:**

* Opravená viněta
* Kalibrovaná odrazivost
* Vícepásmové TIFF (Red, Green, NIR atd.)
* Vědecká data připravená k analýze

### Indexové snímky

**NDVI, NDRE, GNDVI atd. (soubory \_NDVI.tif):**

* Jednopásmové obrázky ve stupních šedi
* Hodnoty pixelů představují výsledky výpočtu indexu
* Rozsah typicky -1 až +1 pro normalizované indexy
* Pro vizualizaci lze použít barevné LUT

***

## Aplikace indexu a LUT

Použijte multispektrální indexy a barevné vyhledávací tabulky:

1. Najděte **Index/LUT Sandbox** v **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> postranní liště
2. Vyberte index vegetace (NDVI, NDRE atd.)
3. Vyberte multispektrální vzorec nebo vytvořte vlastní (pouze Chloros+)
4. Použijte barevný LUT gradient pro vizualizaci
5. Upravte rozsahy hodnot a prahové hodnoty

Podrobné pokyny najdete v části [Index/LUT Sandbox](index-lut-sandbox.md).

***

## Klávesové zkratky

### Navigace

* **→** (šipka doprava): Další obrázek
* **←** (šipka doleva): Předchozí obrázek
* **Home**: První obrázek v seznamu
* **End**: Poslední obrázek v seznamu

### Přiblížení

* **+** nebo **=**: Přiblížit
* **−**: Oddálit
* **0** (nula): Přizpůsobit obrazovce
* **Kolečko myši**: Přiblížit/oddálit

### Ovládací prvky zobrazení

* **P**: Přepnutí režimu procentuálního zobrazení pixelů
* **L**: Přepnutí panelu vrstev
* **Esc**: Zavření celé obrazovky nebo návrat do prohlížeče souborů

### Ostatní

* **Ctrl+S**: Uložení aktuálního obrázku
* **F**: Režim celé obrazovky (je-li k dispozici)

***

### Ověření výpočtů indexů

Zkontrolujte, zda jsou indexy vypočítány správně:

1. Otevřete NDVI nebo jiný indexový obrázek
2. Zkontrolujte oblasti vegetace:
   * **NDVI**: U zdravých rostlin by měla být hodnota 0,4–0,9
   * **NDRE**: Vyšší hodnoty pro bujný růst
   * **GNDVI**: Podobné jako NDVI, ale citlivé na chlorofyl
3. Zkontrolujte oblasti bez vegetace:
   * **Půda**: Blízko 0 nebo mírně záporná hodnota.
   * **Voda**: Záporné hodnoty (-0,5 až 0).

***

## Řešení problémů se zobrazením

### Obrázek se neotevře

**Možné příčiny:**

* Soubor byl během zpracování poškozen.
* Nepodporovaný formát souboru.
* Nedostatečná paměť pro velký obrázek.

**Řešení:**

1. Zkuste soubor otevřít v externím prohlížeči a ověřte jeho integritu.
2. Zkontrolujte, zda formát souboru odpovídá očekávanému typu.
3. Zavřete ostatní aplikace, abyste uvolnili paměť.
4. Zkuste menší/jiný obrázek.

### Černobílý obrázek

**Možné příčiny:**

* Rozsah hodnot mimo možnosti zobrazení.
* 32bitový obrázek s neobvyklými hodnotami.
* Chyba výpočtu indexu.

**Řešení:**

1. Zkontrolujte hodnoty pixelů – pokud jsou všechny velmi nízké nebo velmi vysoké, upravte rozsah zobrazení.
2. Zkuste soubor otevřít v QGIS nebo podobném programu s automatickým nastavením rozsahu.
3. Zkontrolujte protokol ladění ze zpracování, zda neobsahuje chyby.

### Hodnoty pixelů se zdají nesprávné

**Možné příčiny:**

* Zobrazení nesprávného obrázku (originál vs. zpracovaný)
* Kalibrace nebyla provedena správně.
* Data světelného senzoru nebyla zahrnuta do vstupu.
* Režim procent byl přepnut nesprávně.

**Řešení:**

1. Ověřte, zda prohlížíte zpracovaný výstup (zkontrolujte příponu souboru).
2. Zkontrolujte stav tlačítka režimu procent.
3. Porovnejte s obrázky ze stejného datového souboru, o kterých víte, že jsou v pořádku.

***

## Další kroky

Nyní, když můžete zobrazit obrázky na celou obrazovku:

* [**Vrstvy obrázků**](image-layers.md) – Informace o vizualizaci více pásem
* [**Index/LUT Sandbox**](index-lut-sandbox.md) – Použití vlastních indexů a mapování barev
* [**Vzorec multispektrálního indexu**](../project-settings/multispectral-index-formulas.md) – porozumění dostupným indexům

Pro pracovní postup zpracování viz:

* [**Zpracování obrázků (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – kompletní průvodce zpracováním
