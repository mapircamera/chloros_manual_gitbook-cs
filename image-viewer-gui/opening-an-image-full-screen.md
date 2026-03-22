# Otevření obrázku na celou obrazovku

Prohlížeč obrázků Chloros nabízí specializované rozhraní na celou obrazovku pro prohlížení, analýzu a úpravy vašich multispektrálních snímků. Ať už prohlížíte původní snímky nebo zpracované výstupy, prohlížeč obrázků poskytuje výkonné nástroje pro kontrolu a analýzu.

## Přístup k prohlížeči obrázků

### Z prohlížeče souborů

Nejběžnější způsob otevření obrázku v prohlížeči obrázků:

1. Ujistěte se, že jste na záložce **Prohlížeč souborů** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Klikněte na libovolnou **miniaturu snímku** v mřížce snímků
3. Snímek se otevře v **hlavní oblasti náhledu** (uprostřed obrazovky)
4. Snímek je nyní načten a připraven k prohlížení na celé obrazovce

### Otevření karty prohlížeče snímků

Jakmile je snímek načten v oblasti náhledu:

1. Klikněte na ikonu **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> v levém postranním panelu
2. Otevře se karta Prohlížeč obrázků, která zobrazí vybraný obrázek na celé obrazovce
3. V levém postranním panelu se zobrazí pokročilé nástroje pro prohlížení a analýzu

***

## Přehled rozhraní prohlížeče obrázků

### Hlavní zobrazovací oblast

Největší část obrazovky zobrazuje váš obrázek:

* **Plné rozlišení**: Obrázky se zobrazují v nativním rozlišení
* **Možnost přiblížení**: K přiblížení použijte ovládací prvky nebo kolečko myši
* **Možnost posunu**: Po přiblížení klikněte a táhněte pro posun po obrázku
* **Zachování poměru stran**: Obrázky se mění proporcionálně***

## Možnosti prohlížení

### Základní navigace v obrázcích

#### Procházení obrázků

V sadě obrázků se pohybujte pomocí klávesových zkratek nebo tlačítek:

* **Další obrázek**: Klikněte na tlačítko → nebo stiskněte klávesu**→** (šipka vpravo)
* **Předchozí obrázek**: Klikněte na tlačítko ← nebo stiskněte klávesu**←** (šipka vlevo)
* **Přejít na konkrétní obrázek**: Vraťte se do prohlížeče souborů a klikněte na požadovanou miniaturu

#### Ovládací prvky pro přiblížení

Upravte zvětšení pro prohlížení detailů obrázku:

**Přiblížení:*** Klikněte na tlačítko **+** (plus)
* Stiskněte klávesu **+**nebo**=*** Posuňte kolečko myši **nahoru**

**Oddalení:*** Klikněte na tlačítko **−** (mínus)
* Stiskněte klávesu **−** (mínus)
* Posuňte kolečko myši **dolů**

#### Posun při přiblížení

Při přiblížení nad velikost obrazovky:

1. Přesuňte kurzor myši nad obrázek
2. Klikněte a **podržte levé tlačítko myši**

3.**Přetažením** posuňte obrázek
4. Uvolněním tlačítka zastavte posun

**Alternativa**: Použijte šipky k posunu v malých krocích***

## Kontrola hodnot pixelů

### Zobrazení hodnot pixelů u kurzoru

Při pohybu kurzoru myši nad obrázkem se hodnoty pixelů zobrazují v reálném čase:**Umístění zobrazení hodnot:*** **Plovoucí číslo a červená čára v legendě gradientu LUT na pravé straně*** **Při dalším přiblížení plovoucí hodnota poblíž kurzoru a zvýrazněného pixelu*** Zobrazuje hodnoty pro pixel **pod kurzorem nebo zvýrazněný*** Aktualizuje se při pohybu myši

***

## Typy obrázků, které můžete prohlížet

### JPG

**Obrázky JPG z fotoaparátu:**

* Zobrazí data JPG tak, jak jsou v náhledu
* Zobrazí původní, neopravené hodnoty
* Užitečné pro kontrolu kvality obrázku před zpracováním

### RAW (původní)

### RAW (odrazivost)

**Po zpracování:**

* Opravená viněta
* Kalibrovaná odrazivost
* Vícepásmové TIFF (Red, Green, NIR atd.)
* Vědecká data připravená k analýze

### RAW (Index)

**NDVI, NDRE, GNDVI atd. (soubory \_NDVI.tif):**

* Jednopásmové snímky ve stupních šedi
* Hodnoty pixelů představují výsledky výpočtu indexu
* Rozsah obvykle -1 až +1 pro normalizované indexy
* Pro vizualizaci lze použít barevné LUT

***

## Použití indexů a LUT

Použijte multispektrální indexy a barevné Look-Up Tables:

1. Najděte **Index/LUT Sandbox**v**Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> postranní liště
2. Vyberte vegetační index (NDVI, NDRE atd.)
3. Vyberte multispektrální vzorec nebo si vytvořte vlastní (pouze Chloros+)
4. Pro vizualizaci použijte barevný LUT gradient
5. Upravte rozsahy hodnot a prahové hodnoty

Podrobné pokyny najdete v [Index/LUT Sandbox](index-lut-sandbox.md).

***

## Klávesové zkratky

### Navigace

* **→** (šipka vpravo): Další obrázek
* **←** (šipka vlevo): Předchozí obrázek
* **Home**: První obrázek v seznamu
* **End**: Poslední obrázek v seznamu

### Přiblížení

* **+**nebo**=**: Přiblížit
* **−**: Oddálit
* **Kolečko myši**: Přiblížit/oddálit***

### Ověření výpočtů indexů

Zkontrolujte, zda jsou indexy vypočítány správně:

1. Otevřete NDVI nebo jiný obrázek s indexem
2. Zkontrolujte oblasti vegetace:
   * **NDVI**: U zdravých rostlin by měla být hodnota 0,4–0,9
   * **NDRE**: Vyšší hodnoty pro bujný růst
   * **GNDVI**: Podobné jako NDVI, ale citlivé na chlorofyl
3. Zkontrolujte neporost:
   * **Půda**: Blízko 0 nebo mírně záporné
   * **Voda**: Záporné hodnoty (-0,5 až 0)***

## Řešení problémů s prohlížením

### Obrázek se neotevře

**Možné příčiny:**

* Soubor byl během zpracování poškozen
* Nepodporovaný formát souboru
* Nedostatek paměti pro velký obrázek

**Řešení:**

1. Zkuste soubor otevřít v externím prohlížeči a ověřte jeho integritu
2. Zkontrolujte, zda formát souboru odpovídá očekávanému typu
3. Ukončete ostatní aplikace, abyste uvolnili paměť
4. Zkuste menší/jiný obrázek

### Zobrazení černého nebo bílého obrázku

**Možné příčiny:**

* Rozsah hodnot mimo zobrazovací schopnosti
* 32bitový obrázek typu float s neobvyklými hodnotami
* Chyba při výpočtu indexu

**Řešení:**

1. Zkontrolujte hodnoty pixelů – pokud jsou všechny velmi nízké nebo velmi vysoké, upravte rozsah zobrazení
2. Zkuste soubor otevřít v QGIS nebo podobném programu s automatickým nastavením rozsahu
3. Zkontrolujte protokol ladění ze zpracování, zda neobsahuje chyby

### Hodnoty pixelů se zdají být nesprávné

**Možné příčiny:**

* Prohlížíte nesprávný obrázek (originál vs. zpracovaný)
* Kalibrace nebyla provedena správně
* Data ze světelného senzoru nebyla zahrnuta do vstupu
* Režim procent byl přepnut nesprávně

**Řešení:**

1. Ověřte, zda prohlížíte zpracovaný výstup (zkontrolujte příponu názvu souboru)
2. Zkontrolujte stav tlačítka režimu procent
3. Porovnejte s obrázky ze stejného datového souboru, o kterých víte, že jsou v pořádku

***

## Další kroky

Nyní, když můžete prohlížet snímky na celé obrazovce:

* [**Vrstvy snímků**](image-layers.md) – Seznamte se s vizualizací v několika pásmech
* [**Index/LUT Sandbox**](index-lut-sandbox.md) – Použijte vlastní indexy a mapování barev
* [**Vzorce multispektrálních indexů**](../project-settings/multispectral-index-formulas.md) – Seznamte se s dostupnými indexy

Informace o pracovním postupu zpracování naleznete zde:

* [**Zpracování snímků (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Kompletní průvodce zpracováním
