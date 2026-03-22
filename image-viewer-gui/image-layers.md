# Vrstvy obrázků

Rozbalovací nabídka „Vrstvy obrázků“ v prohlížeči obrázků Chloros umožňuje rychle přepínat mezi různými verzemi stejného obrázku – od původních snímků po zpracované výstupy odrazivosti a vypočítané indexové obrázky.

## Co jsou vrstvy obrázků?

V programu Chloros se **vrstvy** vztahují k různým výstupům snímků, které jsou k dispozici pro jeden zdrojový snímek. Při zpracování snímků vytvoří program Chloros několik verzí:

* **Původní snímky** (soubory JPG a RAW z fotoaparátu)
* **Výstupy s kalibrací odrazivosti** (pokud byla kalibrace odrazivosti povolena)
* **Cílové obrázky** (pokud obrázek obsahuje kalibrační cíle)
* **Indexové obrázky** (NDVI, NDRE, GNDVI atd., pokud byly konfigurovány indexy)**Rozbalovací nabídka Výběr vrstvy** v pravém horním rohu prohlížeče obrázků vám umožňuje okamžitě přepínat mezi těmito verzemi, aniž byste museli prohlížeč opustit.***

## Dostupné typy vrstev

### JPG

* Původní náhledový obrázek ve formátu JPG z vašeho fotoaparátu
* Vždy k dispozici pro všechny obrázky
* Nezpracovaný, tak jak byl zachycen fotoaparátem
* Nejrychlejší načítání a zobrazení

**Kdy zobrazit:**

* Rychlý náhled původního snímku
* Kontrola kompozice a rámování snímku
* Ověření kvality snímku před zpracováním

### RAW (původní)

* Původní data ze snímače ve formátu RAW z vašeho fotoaparátu
* Odbylována bez následného zpracování
* Vyšší bitová hloubka než u JPG (obvykle 12bitová nebo 14bitová data ze snímače)

**Kdy zobrazit:**

* Kontrola kvality původních dat ze snímače
* Kontrola případných problémů se snímačem nebo artefaktů
* Porovnání výsledků před a po zpracování

### RAW (Cíl)

* Zobrazí se pouze u snímků, u nichž byly identifikovány kalibrační cíle
* Zobrazuje původní RAW snímek s detekovaným cílem
* Slouží k ověření, zda byla detekce cíle úspěšná

**Kdy zobrazit:**

* Potvrzení, že kalibrační cíle byly detekovány správně
* Kontrola kvality obrazu cíle
* Řešení problémů s kalibrací

{% hint style="info" %}
**Vrstva cíle**: Tato vrstva se v rozevíracím seznamu zobrazí pouze u snímků, které obsahují kalibrační cíle. Běžné snímky tuto možnost nemají.
{% endhint %}

### RAW (odrazivost)

* Kalibrovaný výstupní snímek odrazivosti
* Opravená vinětace (pokud je povolena při zpracování)
* Odrazivost kalibrovaná pomocí dat cílů (pokud je povoleno)
* Vícepásmový snímek TIFF se všemi kanály kamery
* Hodnoty pixelů představují procentuální odrazivost (při použití procentního režimu)
* Připraveno k úpravám pomocí [Index/LUT Sandbox](index-lut-sandbox.md)

**Kdy zobrazit:**

* Kontrola kalibrovaných výsledků
* Ověření kvality kalibrace
* Kontrola hodnot pixelů z hlediska vědecké přesnosti
* Porovnání s originálem za účelem zjištění efektů kalibrace

{% hint style="success" %}
**Doporučeno**: Při kontrole hodnot pixelů pro vědecká měření a analýzy použijte vrstvu RAW (Reflectance).
{% endhint %}

### RAW (NDVI Index)... a podobné

* Vypočítaný obraz vegetačního indexu (v tomto příkladu NDVI)
* Název indexu se mění v závislosti na tom, který index byl nakonfigurován během zpracování
* Příklady: RAW (NDVI Index), RAW (NDRE Index), RAW (GNDVI Index) atd.
* Jednopásmový obrázek ve stupních šedi zobrazující výsledky výpočtu indexu
* Pro každý index nakonfigurovaný v nastavení projektu se zobrazí jedna vrstva

**Možné názvy indexů:**

* RAW (NDVI Index)
* RAW (NDRE Index)
* RAW (GNDVI Index)
* RAW (OSAVI Index)
* RAW (EVI Index)
* RAW (SAVI Index)
* A mnoho dalších... (viz [Vzorce multispektrálních indexů](../project-settings/multispectral-index-formulas.md))

**Kdy zobrazit:**

* Prohlížení výsledků výpočtu indexů
* Kontrola rozsahů hodnot indexů
* Identifikace oblastí zájmu
* Ověření indexových snímků před použitím v GIS nebo při analýze

***

## Použití výběru vrstev

### Otevření rozevíracího seznamu

1. Otevřete snímek v režimu celé obrazovky (klikněte na libovolnou miniaturu v prohlížeči snímků)
2. Najděte **rozevírací nabídku vrstev** v pravém horním rohu prohlížeče
3. Rozevírací nabídka zobrazuje aktuálně vybranou vrstvu (např. „JPG“)
4. Klikněte na rozevírací nabídku a zobrazte všechny dostupné vrstvy

### Přepínání vrstev

1. Klikněte na rozevírací nabídku vrstev a otevřete seznam
2. Zobrazí se všechny dostupné vrstvy pro aktuální snímek
3. Klikněte na název libovolné vrstvy a přepněte na tuto verzi
4. Obrázek se okamžitě aktualizuje a zobrazí vybranou vrstvu

**Rychlé přepínání:**

* Rozbalovací nabídka si pamatuje váš poslední výběr
* Při přechodu na další obrázek se Chloros pokusí zobrazit stejný typ vrstvy
* Pokud tato vrstva na dalším obrázku neexistuje, nastaví se výchozí hodnota na JPG

### Dostupnost vrstev

Ne všechny vrstvy jsou k dispozici pro každý obrázek:

**Vždy k dispozici:*** ✅ JPG (každý obrázek má náhled ve formátu JPG)

**Podmíněně k dispozici:**

* ⚠️ RAW (Originál) – Pouze pokud byl obrázek pořízen v režimu RAW nebo RAW+JPG
* ⚠️ RAW (Cíl) – Pouze pokud obrázek obsahuje detekované kalibrační cíle
* ⚠️ RAW (Reflexe) – Pouze po zpracování s povolenou kalibrací reflexe
* ⚠️ RAW (\[Index] Index) – Pouze po zpracování s nakonfigurovanými indexy

***

## Trvalost vrstev

### Přecházení mezi obrázky

Při přechodu na jiný obrázek (pomocí šipek nebo kliknutím na miniatury):**Předvolba vrstvy zůstává zachována:**

* Pokud prohlížíte „RAW (Reflectance)“, další snímek zobrazí „RAW (Reflectance)“ (je-li k dispozici)
* Pokud prohlížíte „RAW (NDVI Index)“, další snímek zobrazí „RAW (NDVI Index)“ (je-li k dispozici)
* Pokud stejná vrstva neexistuje, nastaví se výchozí hodnota JPG

**Příklad pracovního postupu:**

1. Otevřete obrázek 1, přepněte na RAW (NDVI Index)
2. Stiskněte → pro zobrazení obrázku 2
3. Obrázek 2 automaticky zobrazí vrstvu RAW (NDVI Index)
4. Pokračujte v procházení – všechny obrázky zobrazují vrstvu NDVI
5. Velmi efektivní pro kontrolu výsledků indexu napříč mnoha obrázky

***

## Běžné pracovní postupy

### Pracovní postup 1: Porovnání před/po

**Cíl**: Porovnat původní a kalibrovaný obrázek

1. Otevřete zpracovaný obrázek v prohlížeči obrázků
2. Z rozevíracího seznamu vyberte **RAW (Original)**

3. Poznamenejte si vinětaci a nekalibrované hodnoty
4. V rozevíracím seznamu přepněte na **RAW (Reflectance)**

5. Porovnejte – vinětace odstraněna, hodnoty kalibrovány

### Pracovní postup 2: Prohlížení indexu

**Cíl**: Rychlé prohlížení výsledků NDVI v rámci datového souboru

1. Otevřete první zpracovaný obrázek
2. Z rozevíracího seznamu vyberte **RAW (NDVI Index)**

3. Pomocí šipky → přejděte na další obrázek
4. Vrstva NDVI se automaticky zachová
5. Pokračujte přes všechny obrázky a kontrolujte vzory NDVI
6. Přepněte na **RAW (NDRE Index)** pro porovnání

### Pracovní postup 3: Ověření cílů

**Cíl**: Ověřit, zda byly všechny cílové snímky správně detekovány

1. Přejděte na cílový snímek
2. Z rozevíracího seznamu vyberte **RAW (Target)**

3. Ověřte, zda jsou kalibrační cíle jasně viditelné a detekované
4. Přejděte na další cílový snímek
5. Opakujte ověření u všech cílů

### Pracovní postup 4: Kontrola hodnot pixelů

**Cíl**: Zkontrolovat hodnoty odrazivosti z hlediska vědecké přesnosti

1. Otevřete zpracovaný snímek
2. Vyberte vrstvu **RAW (Reflectance)**

3. Zapněte režim**Pixel Percent** (tlačítko v pravém horním rohu panelu nástrojů)
4. Přesuňte kurzor nad oblasti vegetace
5. Ověřte, zda jsou hodnoty pixelů v očekávaných rozmezích (30–70 % pro NIR, 5–15 % pro Red)
6. Zkontrolujte, zda mají oblasti půdy a vody odpovídající hodnoty

***

## Porozumění hodnotám pixelů podle vrstvy

Různé vrstvy vykazují různá rozmezí hodnot pixelů:

### Vrstva JPG

* **Rozsah**: 0–255 (8 bitů)
* **Význam**: Zobrazené hodnoty, s korekcí gama
* **Použití**: Pouze vizuální kontrola, ne pro vědecká měření

### RAW (originální)

* **Rozsah**: 0–65 535 (16 bitů)
* **Význam**: Surová digitální data ze snímače
* **Použití**: Kontrola výkonu snímače, nekalibrované

### RAW (odrazivost)

* **Rozsah**: 0–65 535 (16bitový TIFF) nebo 0,0–1,0 (32bitové procento)
* **Význam**: Kalibrovaná procentuální odrazivost
* **Použití**: Vědecká měření a analýza**Pro 16bitový TIFF:**Vydělte hodnotu 65 535, abyste získali procentuální odrazivost**Pro 32bitový procentuální:** Hodnoty přímo představují procenta (0,5 = 50% odrazivost)

### RAW (Indexové snímky)

* **Rozsah**: Liší se podle indexu (obvykle -1,0 až +1,0 pro normalizované indexy)
* **Význam**: Výsledek výpočtu indexu
* **Příklady**:
  * NDVI: -1 až +1 (vegetace obvykle 0,4 až 0,9)
  * NDRE: -1 až +1 (detekce stresu)
  * EVI: 0 až 1 (vylepšená vegetace)

***

## Tipy a osvědčené postupy

### Efektivní přepínání vrstev

* **Znalost klávesových zkratek**: Ačkoli pro vrstvy neexistují klávesové zkratky, navigační šipky (←/→) fungují napříč všemi vrstvami
* **Konzistentní pracovní postupy**: Vyberte jednu vrstvu (např. NDVI) a zkontrolujte celý datový soubor, než přepnete na jinou
* **Rychlé srovnání**: Přepínejte mezi vrstvami Original a Reflectance, abyste ověřili kvalitu zpracování

### Úvahy o výkonu

* **JPG se načítá nejrychleji**: Použijte pro rychlou navigaci mezi mnoha obrázky
* **RAW vrstvy se načítá pomaleji**: Vyšší rozlišení a bitová hloubka
* **Indexové vrstvy**: Podobná rychlost jako u vrstev Reflectance
* **První načtení je nejpomalejší**: Následující zobrazení stejné vrstvy jsou uložena v mezipaměti a jsou rychlejší

### Ověření kvality

* **Vždy zkontrolujte RAW (Original)**: Ověřte kvalitu zdrojových dat, než budete důvěřovat zpracovaným výstupům
* **Porovnejte vrstvy**: Pomocí přepínání vrstev ověřte, zda zpracování proběhlo správně
* **Zkontrolujte rozsahy indexů**: Použijte režim Pixel Percent s indexovými vrstvami k ověření, zda jsou hodnoty přiměřené***

## Řešení problémů

### Vrstva není k dispozici

**Problém**: Očekávaná vrstva se nezobrazuje v rozevíracím seznamu**Možné příčiny:**

* Obrázek nebyl zpracován (k dispozici jsou pouze formáty JPG a RAW (Original))
* Během zpracování byla deaktivována kalibrace odrazivosti
* Konkrétní index nebyl nakonfigurován v nastavení projektu
* Snímek je pouze cílový (pro cíle se generují žádné indexy)

**Řešení:**

1. Ověřte, zda byl snímek zpracován (zkontrolujte výstupní složku, zda obsahuje zpracované soubory)
2. Zkontrolujte nastavení projektu a ověřte, zda byly nakonfigurovány indexy
3. Proveďte nové zpracování s povolenými požadovanými indexy

### Zobrazena nesprávná vrstva

**Problém**: Snímek se otevírá v neočekávané vrstvě**Příčina**: Předvolba vrstvy z předchozího snímku byla převzata, ale tato vrstva na aktuálním snímku neexistuje**Řešení**: Chloros automaticky přepne na JPG, pokud není preferovaná vrstva k dispozici – jedná se o normální chování

### Nejsou vidět kalibrační cíle

**Problém**: Vrstva RAW (Cílová) vrstva nezobrazuje detekci cílů**Možné příčiny:**

* Cíle nebyly během zpracování detekovány
* Obrázek ve skutečnosti neobsahuje cíle
* Nastavení detekce cílů je příliš přísné

**Řešení:**

1. Zkontrolujte v ladicím protokolu zprávy „Cíl nalezen“
2. Ověřte, zda obrázek skutečně obsahuje viditelné kalibrační cíle
3. Upravte nastavení detekce cílů v nastavení projektu
4. Viz [Výběr obrázků cílů](../processing-images-gui/choosing-target-images.md)

***

## Související funkce

### Nástroje prohlížeče obrázků

Při prohlížení jakékoli vrstvy můžete použít:

* **Ovládací prvky přiblížení**: Zvětšete obrázek pro prohlížení detailů
* **Posun**: Kliknutím a tažením se pohybujte po přiblíženém obrázku
* **Kontrola hodnoty pixelu**: Zobrazte hodnoty v místě kurzoru
* **Navigační šipky**: Přesuňte se mezi obrázky při zachování vrstvy
* **Režim procenta pixelu**: Přepínejte mezi zobrazením DN a procenty

Viz [Otevření obrázku na celou obrazovku](opening-an-image-full-screen.md) pro kompletní dokumentaci prohlížeče obrázků.

### Index/LUT Sandbox

Pro interaktivní testování a vizualizaci indexů:

* **Výpočet indexu v reálném čase**: Vyzkoušejte různé vzorce indexu
* **Mapování barev LUT**: Použijte barevné přechody na indexy ve stupních šedi
* **Export vizualizací**: Uložte barevné obrázky indexů

Podrobnosti najdete v [Index/LUT Sandbox](index-lut-sandbox.md).

***

## Další kroky

Nyní, když rozumíte obrazovým vrstvám:

* [**Otevření obrázku na celou obrazovku**](opening-an-image-full-screen.md) – Kompletní průvodce prohlížečem obrázků
* [**Index/LUT Sandbox**](index-lut-sandbox.md) – Interaktivní vizualizace indexů
* [**Vzorce multispektrálních indexů**](../project-settings/multispectral-index-formulas.md) – Referenční přehled dostupných indexů
* [**Dokončení zpracování**](../processing-images-gui/finishing-the-processing.md) – Porozumění zpracovaným výstupům
