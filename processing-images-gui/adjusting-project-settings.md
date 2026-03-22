# Nastavení projektu

Před zpracováním snímků je důležité nakonfigurovat nastavení projektu tak, aby odpovídalo vašim pracovním postupům. Panel Nastavení projektu <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> umožňuje komplexní ovládání kalibrace, možností zpracování, multispektrálních indexů a formátů exportu.

## Otevření nastavení projektu

1. Otevřete svůj projekt v Chloros
2. Klikněte na ikonu **Nastavení projektu** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> v levém postranním panelu
3. Panel Nastavení projektu zobrazí všechny možnosti konfigurace

{% hint style="info" %}
**Nastavení se ukládají automaticky** spolu s projektem. Při opětovném otevření projektu se všechna nastavení obnoví.
{% endhint %}

***

## Rychlé nastavení pro běžné pracovní postupy

### Výchozí nastavení (doporučeno pro většinu uživatelů)

Pro typické pracovní postupy s kamerami MAPIR a Survey3 fungují výchozí nastavení dobře:

* ✅ **Korekce vinětace**: Zapnuto
* ✅ **Kalibrace odrazivosti**: Zapnuto (vyžaduje snímky MAPIR terčů)
* ✅ **Metoda debayeringu**: Standardní (rychlá, střední kvalita)
* ✅ **Formát exportu**: TIFF (16bitový)

Stačí importovat snímky a zahájit zpracování s těmito výchozími nastaveními.

***

## Přehled nastavení projektu

Panel Nastavení projektu je rozdělen do několika kategorií. Níže je uveden souhrn jednotlivých sekcí. Kompletní dokumentaci najdete v [Nastavení projektu](../project-settings/project-settings.md).

### Detekce cílů

Ovládá způsob, jakým Chloros identifikuje kalibrační cíle ve vašich snímcích.

**Klíčová nastavení:*** **Minimální oblast kalibračního vzorku**: Prahová hodnota velikosti pro detekci cílů (výchozí: 25 pixelů)
* **Minimální shlukování cílů**: Prahová hodnota podobnosti pro seskupování oblastí cílů (výchozí: 60)**Kdy upravit:**

* Zvětšete oblast vzorku, pokud dochází k falešným detekcím
* Zmenšete ji, pokud se cíle nedetekují
* Upravte seskupování, pokud se cíle rozdělují do více detekcí

### Zpracování

Hlavní možnosti zpracování obrazu a kalibrace.

**Klíčová nastavení:*** **Korekce vinětace**: Kompenzuje ztmavnutí objektivu na okrajích ✅ Doporučeno
* **Kalibrace odrazivosti**: Normalizuje hodnoty pomocí kalibračních cílů ✅ Doporučeno
* **Metoda debayeringu**: Algoritmus pro převod RAW na 3kanálový multispektrální obraz
* **Minimální interval rekalibrace**: Čas mezi použitím kalibračních cílů (0 = použít všechny)**Pokročilá nastavení:*** **Časový posun světelného senzoru**: Pro časovou synchronizaci PPK (výchozí: 0)
* **Použít korekce PPK**: Používá data GPS/expozičních pinů ze souborů .daq
* **Expoziční pin 1/2**: Přiřazuje kamery k expozičním pinům pro konfigurace se dvěma kamerami

### Metoda debayeringu

V Chloros aktuálně nabízíme 2 metody debayeringu:

#### Standardní (rychlá, střední kvalita)

Standardní debayering se zpracovává rychle, ale vykazuje barevný šum, což vede k méně přesným a šumovějším snímkům.

#### Texture Aware (pomalé, nejvyšší kvalita) \[Pouze Chloros+]

Texture Aware používá vysoce kvalitní debayering s rozpoznáním hran v kombinaci s modelem odšumování AI/ML, který odstraní téměř veškerý šum způsobený debayeringem. Model Texture Aware vyžaduje k provozu paměť GPU (VRAM). Pro rychlejší zpracování doporučujeme jej používat, pokud máte k dispozici více než 4 GB VRAM.

### Index (multispektrální indexy)

Nakonfigurujte, které vegetační indexy se mají vypočítat a exportovat.

**Jak přidat indexy:**

1. Klikněte na tlačítko**„Přidat index“**

2. Vyberte index z rozevíracího menu (NDVI, NDRE, GNDVI atd.)
3. Nakonfigurujte nastavení vizualizace (barvy LUT, rozsahy hodnot)
4. Podle potřeby přidejte více indexů

**Oblíbené indexy:*** **NDVI**: Obecný zdravotní stav vegetace (nejběžnější)
* **NDRE**: Včasná detekce stresu s RedEdge
* **GNDVI**: Citlivý na koncentraci chlorofylu
* **OSAVI**: Funguje dobře s viditelnou půdou
* **EVI**: Oblasti s vysokým indexem listové plochy (LAI)**Vlastní vzorce (pouze Chloros+):**

* Vytvářejte vlastní vzorce multispektrálních indexů
* Používejte matematické operace s pásmy u všech kanálů snímku
* Ukládejte vlastní vzorce pro opakované použití

Všechny dostupné indexy a vzorce najdete v [Vzorce multispektrálních indexů](../project-settings/multispectral-index-formulas.md).

### Export

Ovládá formát a kvalitu výstupního souboru.

**Dostupné formáty:*** **TIFF (16bitový)**: Doporučeno pro GIS a vědeckou analýzu (rozsah 0–65 535)
* **TIFF (32bitový, procenta)**: Hodnoty odrazivosti s plovoucí desetinnou čárkou (rozsah 0,0–1,0)
* **PNG (8bitový)**: Bezztrátová komprese pro vizualizaci (rozsah 0–255)
* **JPG (8bitový)**: Nejmenší soubory, ztrátová komprese (rozsah 0–255)***

## Ukládání a načítání nastavení

### Uložení šablony projektu

Vytvořte opakovaně použitelné šablony pro konzistentní pracovní postupy:

1. Nakonfigurujte všechna požadovaná nastavení v panelu Nastavení projektu
2. Přejděte do sekce **„Uložit šablonu projektu“** v dolní části
3. Zadejte popisný název šablony (např. „Survey3N\_RGN\_Agriculture“)
4. Klikněte na ikonu uložení

**Výhody:**

* Použijte identická nastavení pro více projektů
* Sdílejte konfigurace s členy týmu
* Zachovejte konzistenci při opakovaných průzkumech

### Načtení šablony do nového projektu

Při vytváření nového projektu:

1. V hlavním menu vyberte **„Nový projekt“**

2. Zvolte možnost**„Načíst ze šablony“**

3. Vyberte uloženou šablonu
4. Všechna nastavení se automaticky použijí

### Pracovní adresář

Nastavení **„Složka pro uložení projektu“** určuje, kam se nové projekty standardně ukládají:

* **Výchozí umístění**: `C:\Users\[Username]\Chloros Projects`
* **Změna umístění**: Klikněte na ikonu úprav a vyberte novou složku
* **Kdy změnit**:
  * Síťový disk pro spolupráci v týmu
  * Jiný disk s větším úložným prostorem
  * Uspořádaná struktura složek podle roku/klienta

***

## Nastavení PPK (Post-Processed Kinematic)

Při použití záznamníků MAPIR DAQ s GPS pro přesnou geolokaci:

### Předpoklady

* DAQ MAPIR s modulem GPS (GNSS)
* Soubor protokolu .daq se záznamy o expozičních pinech
* Kamera připojená k expozičním pinům DAQ během snímání

### Kroky konfigurace

1. Umístěte soubor protokolu .daq do složky projektu
2. V nastavení projektu zaškrtněte políčko **„Použít korekce PPK“**

3. V případě potřeby nastavte**„Časový posun světelného senzoru“** (výchozí: 0 pro UTC)
4. Přiřaďte kamery k pinům expozice:
   * **Jedna kamera**: Automaticky přiřazena k pinu 1
   * **Dvě kamery**: Ručně přiřaďte každou kameru ke správnému pinu**Přiřazení pinů expozice:*** **Expoziční pin 1**: Vyberte model kamery z rozevíracího seznamu
* **Expoziční pin 2**: Vyberte druhou kameru nebo „Nepoužívat“
* Stejnou kameru nelze přiřadit k oběma pinům

{% hint style="warning" %}
**Důležité**: Expoziční piny musí být správně přiřazeny k příslušným kamerám. Nesprávné přiřazení bude mít za následek nesprávná geolokační data.
{% endhint %}

***

## Pokročilé scénáře

### Projekty s více kamerami

Při zpracování snímků z více kamer MAPIR v jednom projektu:

1. Chloros automaticky detekuje každý model kamery
2. Každá kamera dostane příslušný profil zpracování
3. PPK: Ručně přiřaďte každou kameru ke správnému expozici
4. Všechny kamery používají stejný exportní formát a indexy

**Příklad**: Survey3W RGN + Survey3N OCN sestava se dvěma kamerami

### Časosběrné nebo vícerozměrné průzkumy

Pro opakované průzkumy stejné oblasti v průběhu času:

1. Vytvořte šablonu se standardními nastaveními
2. Používejte při každé relaci konzistentní nastavení kalibračního terče
3. Zpracovávejte každý den jako samostatný projekt
4. Používejte identická nastavení pro srovnatelné výsledky
5. Exportujte ve stejném formátu pro časovou analýzu

### Velké datové sady

Pro projekty s velkým počtem snímků (500+):

* Zvažte rozdělení na menší projekty podle data nebo oblasti
* Pro rychlejší výsledky použijte paralelní zpracování Chloros+
* Pro automatizaci dávkového zpracování zvažte CLI nebo API
* Upravte minimální interval rekalibrace, abyste zkrátili dobu detekce cílů

***

## Ověření nastavení

Před zahájením zpracování zkontrolujte tato klíčová nastavení:

* [ ] Model kamery správně detekován v prohlížeči souborů
* [ ] Korekce vinětace povolena
* [ ] Kalibrace odrazivosti povolena
* [ ] Importován alespoň jeden obrázek kalibračního cíle
* [ ] Přidány požadované multispektrální indexy
* [ ] Formát exportu vhodný pro váš pracovní postup
* [ ] Nastavení PPK nakonfigurováno (pokud používáte .daq s událostmi expozice)

***

## Další kroky

Jakmile máte nastavení nakonfigurováno:

1. **Označte kalibrační cílové obrázky** – viz [Výběr cílových obrázků](choosing-target-images.md)
2. **Spusťte zpracování** – viz [Spuštění zpracování](starting-the-processing.md)
3. **Sledujte průběh** – viz [Sledování zpracování](monitoring-the-processing.md)

Úplné podrobnosti o všech dostupných nastaveních najdete v referenční dokumentaci [Nastavení projektu](../project-settings/project-settings.md).
