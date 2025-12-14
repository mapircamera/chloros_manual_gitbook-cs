# Vrstvy obrázků

Rozbalovací nabídka Vrstvy obrázků v prohlížeči obrázků Chloros umožňuje rychle přepínat mezi různými verzemi stejného obrázku – od původních snímků po zpracované výstupy odrazivosti a vypočítané indexové obrázky.

## Co jsou vrstvy obrázků?

V Chloros se **vrstvy** vztahují k různým výstupům obrázků, které jsou k dispozici pro jeden zdrojový obrázek. Při zpracování obrázků vytvoří Chloros několik verzí:

* **Původní obrázky** (soubory JPG a RAW z fotoaparátu)
* Výstupy **s kalibrovanou odrazivostí** (pokud byla povolena kalibrace odrazivosti)
* **Cílové obrázky** (pokud obrázek obsahuje kalibrační cíle)
* **Indexové obrázky** (NDVI, NDRE, GNDVI atd., pokud byly nakonfigurovány indexy)

**Rozbalovací nabídka pro výběr vrstvy** v pravém horním rohu prohlížeče obrázků vám umožňuje okamžitě přepínat mezi těmito verzemi, aniž byste museli prohlížeč opustit.

***

## Dostupné typy vrstev

### JPG

* Původní náhledový obrázek JPG z vašeho fotoaparátu
* Vždy k dispozici pro všechny obrázky
* Nezpracovaný, tak jak byl zachycen fotoaparátem
* Nejrychlejší načítání a zobrazení

**Kdy zobrazit:**

* Rychlý náhled původního snímku
* Kontrola kompozice a rámování obrázku
* Ověření kvality snímku před zpracováním

### RAW (původní)

* Originální data RAW ze snímače fotoaparátu
* Debayering bez následného zpracování
* Vyšší bitová hloubka než JPG (obvykle 12bitová nebo 14bitová data snímače)

**Kdy zobrazit:**

* Kontrola kvality originálních dat snímače
* Kontrola problémů se snímačem nebo artefaktů
* Porovnání výsledků před a po zpracování

### RAW (cíl)

* Zobrazuje se pouze u obrázků identifikovaných jako obsahující kalibrační cíle
* Zobrazuje původní RAW obrázek s detekovaným cílem
* Používá se k ověření, zda byla detekce cíle úspěšná

**Kdy zobrazit:**

* Potvrzení, že kalibrační cíle byly detekovány správně
* Kontrola kvality cílového obrázku
* Řešení problémů s kalibrací

{% hint style=&quot;info&quot; %}
**Vrstva cíle**: Tato vrstva se zobrazí pouze v rozevíracím seznamu u obrázků, které obsahují kalibrační cíle. Běžné snímky tuto možnost nemají.
{% endhint %}

### RAW (odrazivost)

* Kalibrovaný výstupní obrázek odrazivosti
* Korekce vinětace (pokud je povoleno při zpracování)
* Odrazivost kalibrována pomocí cílových dat (pokud je povoleno)
* Vícekanálové TIFF se všemi kanály kamery
* Hodnoty pixelů představují procentuální odrazivost (při použití procentního režimu)
* Připraveno k manipulaci pomocí [Index/LUT Sandbox](index-lut-sandbox.md)

**Kdy zobrazit:**

* Kontrola kalibrovaných výsledků
* Ověření kvality kalibrace
* Kontrola hodnot pixelů z hlediska vědecké přesnosti
* Porovnání s originálem za účelem zjištění účinků kalibrace

{% hint style=&quot;success&quot; %}
**Doporučeno**: Při kontrole hodnot pixelů pro vědecká měření a analýzy použijte vrstvu RAW (odrazivost).
{% endhint %}

### RAW (NDVI Index)... a podobné

* Vypočítaný obraz vegetačního indexu (v tomto příkladu NDVI)
* Název indexu se mění podle toho, který index byl nakonfigurován během zpracování
* Příklady: RAW (NDVI Index), RAW (NDRE Index), RAW (GNDVI Index) atd.
* Jednopásmový obrázek ve stupních šedi zobrazující výsledky výpočtu indexu
* Pro každý index nakonfigurovaný v nastavení projektu se zobrazí jedna vrstva

**Možné názvy indexů:**

* RAW (index NDVI)
* RAW (index NDRE)
* RAW (index GNDVI)
* RAW (OSAVI Index)
* RAW (EVI Index)
* RAW (SAVI Index)
* A mnoho dalších... (viz [Vzorec multispektrálního indexu](../project-settings/multispectral-index-formulas.md))

**Kdy zobrazit:**

* Kontrola výsledků výpočtu indexu
* Kontrola rozsahů hodnot indexu
* Identifikace oblastí zájmu
* Ověření indexových obrázků před použitím v GIS nebo analýze

***

## Použití výběru vrstev

### Otevření rozevíracího seznamu

1. Otevřete obrázek v režimu celé obrazovky (klikněte na libovolnou miniaturu v prohlížeči obrázků)
2. Najděte **rozevírací nabídku vrstev** v pravém horním rohu prohlížeče
3. Rozevírací nabídka zobrazuje aktuálně vybranou vrstvu (např. „JPG“)
4. Kliknutím na rozevírací nabídku zobrazíte všechny dostupné vrstvy

### Přepínání vrstev

1. Kliknutím na rozevírací nabídku vrstev otevřete seznam
2. Zobrazí se všechny dostupné vrstvy pro aktuální obrázek
3. Kliknutím na název libovolné vrstvy přepnete na danou verzi.
4. Obrázek se okamžitě aktualizuje a zobrazí vybranou vrstvu.

**Rychlé přepínání:**

* Rozbalovací nabídka si pamatuje vaši poslední volbu.
* Při přechodu na další obrázek se Chloros pokusí zobrazit stejný typ vrstvy.
* Pokud daná vrstva na dalším obrázku neexistuje, použije se výchozí formát JPG.

### Dostupnost vrstev

Ne všechny vrstvy jsou k dispozici pro každý obrázek:

**Vždy k dispozici:**

* ✅ JPG (každý obrázek má náhled JPG)

**Podmíněně k dispozici:**

* ⚠️ RAW (originál) – pouze pokud byl obrázek pořízen v režimu RAW nebo RAW+JPG
* ⚠️ RAW (cíl) – pouze pokud obrázek obsahuje detekované kalibrační cíle
* ⚠️ RAW (odrazivost) – pouze po zpracování s povolenou kalibrací odrazivosti
* ⚠️ RAW (\[Index] Index) – pouze po zpracování s nakonfigurovanými indexy

***

## Trvalost vrstev

### Přecházení mezi obrázky

Při přechodu na jiný obrázek (pomocí kláves se šipkami nebo kliknutím na miniatury):

**Preference vrstvy zůstává zachována:**

* Pokud prohlížíte „RAW (Reflexe)“, další obrázek zobrazí „RAW (Reflexe)“ (je-li k dispozici)
* Pokud prohlížíte „RAW (NDVI Index)“, další obrázek zobrazí „RAW (NDVI Index)“ (je-li k dispozici)
* Pokud stejná vrstva neexistuje, je výchozí nastavení JPG

**Příklad pracovního postupu:**

1. Otevřete obrázek 1, přepněte na RAW (NDVI Index)
2. Stiskněte → pro zobrazení obrázku 2
3. Obrázek 2 automaticky zobrazí vrstvu RAW (NDVI Index)
4. Pokračujte v navigaci – všechny obrázky zobrazují vrstvu NDVI.
5. Velmi efektivní pro kontrolu výsledků indexu u mnoha obrázků.

***

## Běžné pracovní postupy

### Pracovní postup 1: Porovnání před a po

**Cíl**: Porovnat původní a kalibrovaný obrázek.

1. Otevřete zpracovaný obrázek v prohlížeči obrázků.
2. Z rozevíracího seznamu vyberte **RAW (Original)**.
3. Poznamenejte si vinětaci a nekalibrované hodnoty.
4. V rozevíracím seznamu přepněte na **RAW (Reflectance)**.
5. Porovnejte – vinětace odstraněna, hodnoty kalibrovány.

### Pracovní postup 2: Kontrola indexu

**Cíl**: Rychlá kontrola výsledků NDVI v datovém souboru

1. Otevřete první zpracovaný obrázek.
2. Z rozevíracího seznamu vyberte **RAW (NDVI Index)**.
3. Pomocí klávesy → přejděte na další obrázek.
4. Vrstva NDVI zůstává automaticky zachována.
5. Pokračujte u všech obrázků a zkontrolujte vzory NDVI.
6. Přepněte na **RAW (NDRE Index)** pro porovnání.

### Pracovní postup 3: Ověření cíle

**Cíl**: Ověřit, zda byly všechny cílové obrázky správně detekovány.

1. Přejděte na cílový obrázek.
2. Z rozevíracího seznamu vyberte **RAW (Target)**.
3. Ověřte, zda jsou kalibrační cíle jasně viditelné a detekované.
4. Přejděte k dalšímu cílovému obrázku.
5. Opakujte ověření pro všechny cíle.

### Pracovní postup 4: Kontrola hodnoty pixelů

**Cíl**: Zkontrolovat hodnoty odrazivosti z hlediska vědecké přesnosti.

1. Otevřete zpracovaný obrázek.
2. Vyberte vrstvu **RAW (Reflectance)**.
3. Zapněte režim **Pixel Percent** (tlačítko v pravém horním rohu panelu nástrojů).
4. Přesuňte kurzor nad oblasti vegetace.
5. Ověřte, zda jsou hodnoty pixelů v očekávaných rozmezích (30–70 % pro NIR, 5–15 % pro Red).
6. Zkontrolujte, zda oblasti půdy a vody mají správné hodnoty.

***

## Porozumění hodnotám pixelů podle vrstvy

Různé vrstvy zobrazují různé rozsahy hodnot pixelů:

### Vrstva JPG

* **Rozsah**: 0–255 (8 bitů)
* **Význam**: Zobrazené hodnoty, korigované gama
* **Použití**: Pouze vizuální kontrola, ne pro vědecká měření

### RAW (původní)

* **Rozsah**: 0–65535 (16 bitů)
* **Význam**: Surová digitální čísla senzoru
* **Použití**: Kontrola výkonu senzoru, nekalibrované

### RAW (odrazivost)

* **Rozsah**: 0–65 535 (16bitové TIFF) nebo 0,0–1,0 (32bitové procento)
* **Význam**: Kalibrovaná procentuální odrazivost
* **Použití**: Vědecká měření a analýzy

**Pro 16bitový TIFF:** Vydělte hodnotou 65 535, abyste získali procentuální odrazivost **Pro 32bitové procento:** Hodnoty přímo představují procento (0,5 = 50% odrazivost)

### RAW (indexové obrázky)

* **Rozsah**: Liší se podle indexu (obvykle -1,0 až +1,0 pro normalizované indexy)
* **Význam**: Výsledek výpočtu indexu
* **Příklady**:
  * NDVI: -1 až +1 (vegetace obvykle 0,4 až 0,9)
  * NDRE: -1 až +1 (detekce stresu)
  * EVI: 0 až 1 (vylepšená vegetace)

***

## Tipy a osvědčené postupy

### Efektivní přepínání vrstev

* **Znalost klávesových zkratek**: Ačkoli pro vrstvy neexistují žádné klávesové zkratky, navigační šipky (←/→) fungují ve všech vrstvách
* **Konzistentní pracovní postupy**: Vyberte jednu vrstvu (např. NDVI) a zkontrolujte celý datový soubor, než přepnete na jinou
* **Rychlé porovnání**: Přepínejte mezi originálem a odrazivostí, abyste ověřili kvalitu zpracování

### Úvahy o výkonu

* **JPG se načítá nejrychleji**: Použijte pro rychlou navigaci mezi mnoha obrázky.
* **Vrstvy RAW se načítá pomaleji**: Vyšší rozlišení a bitová hloubka.
* **Indexové vrstvy**: Podobná rychlost jako vrstvy odrazivosti.
* **První načtení je nejpomalejší**: Následná zobrazení stejné vrstvy jsou uložena v mezipaměti a jsou rychlejší.

### Ověření kvality

* **Vždy zkontrolujte RAW (originál)**: Ověřte kvalitu zdrojových dat, než budete důvěřovat zpracovaným výstupům.
* **Porovnejte vrstvy**: Pomocí přepínání vrstev ověřte, zda zpracování proběhlo správně.
* **Zkontrolujte rozsahy indexů**: Pomocí režimu Pixel Percent s indexovými vrstvami ověřte, zda jsou hodnoty přiměřené.

***

## Řešení problémů

### Vrstva není k dispozici

**Problém**: Očekávaná vrstva se nezobrazuje v rozevíracím seznamu

**Možné příčiny:**

* Obrázek nebyl zpracován (k dispozici jsou pouze formáty JPG a RAW (originál))
* Během zpracování byla deaktivována kalibrace odrazivosti
* V nastavení projektu nebyl nakonfigurován konkrétní index
* Obrázek je pouze cílový obrázek (pro cíle nebyly generovány žádné indexy)

**Řešení:**

1. Ověřte, zda byl obrázek zpracován (zkontrolujte výstupní složku, zda obsahuje zpracované soubory).
2. Zkontrolujte nastavení projektu a ověřte, zda byly nakonfigurovány indexy.
3. Zpracujte obrázek znovu s povolenými požadovanými indexy.

### Zobrazena nesprávná vrstva

**Problém**: Obrázek se otevře v neočekávané vrstvě.

**Příčina**: Předvolba vrstvy z předchozího obrázku byla přenesena, ale tato vrstva v aktuálním obrázku neexistuje.

**Řešení**: Chloros automaticky přejde na formát JPG, pokud není k dispozici preferovaná vrstva – jedná se o normální chování.

### Nelze vidět kalibrační cíle

**Problém**: Vrstva RAW (cíl) nezobrazuje detekci cílů.

**Možné příčiny:**

* Cíle nebyly během zpracování detekovány
* Obrázek ve skutečnosti neobsahuje cíle
* Nastavení detekce cílů je příliš přísné

**Řešení:**

1. Zkontrolujte protokol ladění, zda neobsahuje zprávy „Cíl nalezen“
2. Ověřte, zda obrázek skutečně obsahuje viditelné kalibrační cíle
3. Upravte nastavení detekce cílů v nastavení projektu
4. Viz [Výběr cílových obrázků](../processing-images-gui/choosing-target-images.md)

***

## Související funkce

### Nástroje prohlížeče obrázků

Při prohlížení jakékoli vrstvy můžete použít:

* **Ovládací prvky zoomu**: Zvětšete obrázek a prohlédněte si detaily.
* **Posun**: Kliknutím a tažením se pohybujte po zvětšeném obrázku.
* **Kontrola hodnoty pixelů**: Zobrazte hodnoty v místě kurzoru.
* **Navigační šipky**: Přesun mezi obrázky při zachování vrstvy
* **Režim procent pixelů**: Přepínání mezi zobrazením DN a procent

Kompletní dokumentaci k prohlížeči obrázků najdete v části [Otevření obrázku na celou obrazovku](opening-an-image-full-screen.md).

### Index/LUT Sandbox

Pro interaktivní testování a vizualizaci indexů:

* **Výpočet indexu v reálném čase**: Vyzkoušejte různé vzorce indexů.
* **Mapování barev LUT**: Použijte barevné přechody na indexy ve stupních šedi.
* **Export vizualizací**: Uložte barevné obrázky indexů.

Podrobnosti najdete v části [Index/LUT Sandbox](index-lut-sandbox.md).

***

## Další kroky

Nyní, když rozumíte vrstvám obrázků:

* [**Otevření obrázku na celou obrazovku**](opening-an-image-full-screen.md) – Kompletní průvodce prohlížečem obrázků
* [**Index/LUT Sandbox**](index-lut-sandbox.md) – Interaktivní vizualizace indexu
* [**Vzorec multispektrálního indexu**](../project-settings/multispectral-index-formulas.md) – Referenční dostupné indexy
* [**Dokončení zpracování**](../processing-images-gui/finishing-the-processing.md) – Porozumění zpracovaným výstupům
