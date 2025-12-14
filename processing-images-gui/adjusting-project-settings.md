# Úprava nastavení projektu

Před zpracováním obrázků je důležité nakonfigurovat nastavení projektu tak, aby odpovídalo požadavkům vašeho pracovního postupu. Panel Nastavení projektu <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> poskytuje komplexní kontrolu nad kalibrací, možnostmi zpracování, multispektrálními indexy a formáty exportu.

## Přístup k nastavení projektu

1. Otevřete svůj projekt v Chloros
2. Klikněte na ikonu **Nastavení projektu** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> v levém postranním panelu
3. Panel Nastavení projektu zobrazí všechny možnosti konfigurace

{% hint style=&quot;info&quot; %}
**Nastavení se ukládají automaticky** spolu s projektem. Při opětovném otevření projektu se všechna nastavení obnoví.
{% endhint %}

***

## Rychlé nastavení pro běžné pracovní postupy

### Výchozí nastavení (doporučeno pro většinu uživatelů)

Pro typické pracovní postupy s kamerou MAPIR Survey3 fungují dobře výchozí nastavení:

* ✅ **Korekce vinětace**: Zapnuto
* ✅ **Kalibrace odrazivosti**: Zapnuto (vyžaduje obrázky cílů MAPIR)
* ✅ **Metoda Debayer**: Vysoká kvalita (rychlejší)
* ✅ **Formát exportu**: TIFF (16 bitů)

Jednoduše importujte své snímky a začněte je zpracovávat s těmito výchozími nastaveními.

***

## Přehled nastavení projektu

Panel Nastavení projektu je rozdělen do několika kategorií. Níže je uveden souhrn jednotlivých sekcí. Kompletní dokumentaci najdete v části [Nastavení projektu](../project-settings/project-settings.md).

### Detekce cílů

Ovládá způsob, jakým Chloros identifikuje kalibrační cíle ve vašich obrázcích.

**Klíčová nastavení:**

* **Minimální plocha vzorku pro kalibraci**: Prahová hodnota velikosti pro detekci cílů (výchozí nastavení: 25 pixelů)
* **Minimální shlukování cílů**: Prahová hodnota podobnosti pro seskupování cílových oblastí (výchozí nastavení: 60)

**Kdy upravit:**

* Zvětšete plochu vzorku, pokud dochází k falešným detekcím.
* Zmenšete ji, pokud cíle nejsou detekovány.
* Upravte shlukování, pokud jsou cíle rozděleny do více detekcí.

### Zpracování

Hlavní možnosti zpracování obrazu a kalibrace.

**Klíčová nastavení:**

* **Korekce vinětace**: Kompenzuje ztmavnutí objektivu na okrajích ✅ Doporučeno
* **Kalibrace odrazivosti**: Normalizuje hodnoty pomocí kalibračních cílů ✅ Doporučeno
* **Metoda Debayer**: Algoritmus pro převod RAW na 3kanálové multispektrální
* **Minimální interval rekalibrace**: Čas mezi použitím kalibračních cílů (0 = použít všechny)

**Pokročilá nastavení:**

* **Časový posun světelného senzoru**: Pro synchronizaci času PPK (výchozí: 0)
* **Použít korekce PPK**: Používá data GPS/expozice z souborů .daq
* **Expozice pin 1/2**: Přiřazuje kamery k expozičním pinům pro nastavení dvou kamer

### Index (multispektrální indexy)

Nakonfigurujte, které vegetační indexy se mají počítat a exportovat.

**Jak přidat indexy:**

1. Klikněte na tlačítko **„Přidat index“**
2. Vyberte index z rozevíracího menu (NDVI, NDRE, GNDVI atd.)
3. Nakonfigurujte nastavení vizualizace (barvy LUT, rozsahy hodnot)
4. Podle potřeby přidejte více indexů

**Populární indexy:**

* **NDVI**: Obecný stav vegetace (nejběžnější)
* **NDRE**: Včasná detekce stresu pomocí RedEdge
* **GNDVI**: Citlivý na koncentraci chlorofylu
* **OSAVI**: Funguje dobře s viditelnou půdou
* **EVI**: Oblasti s vysokým indexem listové plochy (LAI)

**Vlastní vzorce (pouze Chloros+):**

* Vytvořte vlastní vzorce multispektrálního indexu
* Použijte matematiku pásem se všemi kanály obrazu
* Uložte vlastní vzorce pro opětovné použití

Všechny dostupné indexy a vzorce najdete v části [Vzorce multispektrálního indexu](../project-settings/multispectral-index-formulas.md).

### Export

Ovládá formát a kvalitu výstupního souboru.

**Dostupné formáty:**

* **TIFF (16bitový)**: Doporučeno pro GIS a vědeckou analýzu (rozsah 0–65 535)
* **TIFF (32bitový, procenta)**: Hodnoty odrazivosti s plovoucí desetinnou čárkou (rozsah 0,0–1,0)
* **PNG (8bitový)**: Bezeztrátová komprese pro vizualizaci (rozsah 0–255)
* **JPG (8bitový)**: Nejmenší soubory, ztrátová komprese (rozsah 0–255)

***

## Ukládání a načítání nastavení

### Uložení šablony projektu

Vytvořte opakovaně použitelné šablony pro konzistentní pracovní postupy:

1. Nakonfigurujte všechna požadovaná nastavení v panelu Nastavení projektu.
2. Přejděte do sekce **„Uložit šablonu projektu“** v dolní části.
3. Zadejte popisný název šablony (např. „Survey3N\_RGN\_Agriculture“).
4. Klikněte na ikonu uložení.

**Výhody:**

* Použijte identická nastavení pro více projektů.
* Sdílejte konfigurace s členy týmu.
* Zachovejte konzistenci opakovaných průzkumů.

### Načtení šablony do nového projektu

Při vytváření nového projektu:

1. V hlavním menu vyberte **„Nový projekt“**.
2. Vyberte možnost **„Načíst ze šablony“**.
3. Vyberte uloženou šablonu.
4. Všechna nastavení se automaticky použijí.

### Pracovní adresář

Nastavení **„Uložit složku projektu“** určuje, kde se nové projekty vytvářejí ve výchozím nastavení:

* **Výchozí umístění**: `C:\Users\[Username]\Chloros Projects`
* **Změnit umístění**: Klikněte na ikonu úprav a vyberte novou složku.
* **Kdy změnit**:
  * Síťový disk pro spolupráci týmu.
  * Jiný disk s větším úložným prostorem.
  * Uspořádaná struktura složek podle roku/klienta.

***

## Nastavení PPK (Post-Processed Kinematic)

Při použití záznamníků MAPIR DAQ s GPS pro přesnou geolokaci:

### Předpoklady

* MAPIR DAQ s modulem GPS (GNSS)
* Soubor protokolu .daq se záznamy expozičních pinů
* Kamera připojená k expozičním pinům DAQ během snímání

### Konfigurační kroky

1. Umístěte soubor protokolu .daq do složky projektu.
2. V nastavení projektu zaškrtněte políčko **„Použít korekce PPK“**.
3. V případě potřeby nastavte **„Časový posun světelného senzoru“** (výchozí hodnota: 0 pro UTC).
4. Přiřaďte kamery k expozičním pinům:
   * **Jedna kamera**: Automaticky přiřazena k pinu 1.
   * **Dvě kamery**: Ručně přiřaďte každou kameru ke správnému pinu.

**Přiřazení pinů expozice:**

* **Pin expozice 1**: Vyberte model kamery z rozevíracího seznamu.
* **Pin expozice 2**: Vyberte druhou kameru nebo „Nepoužívat“.
* Stejnou kameru nelze přiřadit k oběma pinům.

{% hint style=&quot;warning&quot; %}
**Důležité**: Piny expozice musí být správně přiřazeny příslušným kamerám. Nesprávné přiřazení bude mít za následek nesprávné údaje o geolokaci.
{% endhint %}

***

## Pokročilé scénáře

### Projekty s více kamerami

Při zpracování snímků z více kamer MAPIR v jednom projektu:

1. Chloros automaticky detekuje každý model kamery
2. Každá kamera dostane příslušný profil zpracování
3. PPK: Ručně přiřaďte každé kameře správný expoziční pin.
4. Všechny kamery používají stejný exportní formát a indexy.

**Příklad**: Survey3W RGN + Survey3N OCN zařízení se dvěma kamerami

### Časosběrné nebo vícedatové průzkumy

Pro opakované průzkumy stejné oblasti v průběhu času:

1. Vytvořte šablonu se standardními nastaveními.
2. Používejte konzistentní nastavení kalibračního cíle pro každou relaci.
3. Zpracovávejte každý den jako samostatný projekt.
4. Používejte identická nastavení pro srovnatelné výsledky.
5. Exportujte ve stejném formátu pro časovou analýzu.

### Velké datové soubory

Pro projekty s velkým počtem obrázků (500+):

* Zvažte rozdělení na menší projekty podle data nebo oblasti.
* Pro rychlejší výsledky použijte paralelní zpracování Chloros+.
* Pro automatizaci dávkového zpracování zvažte použití CLI nebo API.
* Upravte minimální interval rekalibrace, abyste zkrátili dobu detekce cíle.

***

## Ověření nastavení

Před zahájením zpracování zkontrolujte tato klíčová nastavení:

* [ ] Správně detekovaný model kamery v prohlížeči souborů
* [ ] Zapnutá korekce viněty
* [ ] Zapnutá kalibrace odrazivosti
* [ ] Importován alespoň jeden kalibrační cílový obrázek
* [ ] Přidány požadované multispektrální indexy
* [ ] Exportní formát vhodný pro váš pracovní postup
* [ ] Nakonfigurována nastavení PPK (pokud používáte .daq s expozičními událostmi)

***

## Další kroky

Po nakonfigurování nastavení:

1. **Označte kalibrační cílové snímky** – viz [Výběr cílových snímků](choosing-target-images.md)
2. **Spusťte zpracování** – viz [Spuštění zpracování](starting-the-processing.md)
3. **Sledujte průběh** – viz [Sledování zpracování](monitoring-the-processing.md)

Úplné podrobnosti o všech dostupných nastaveních najdete v referenční dokumentaci [Nastavení projektu](../project-settings/project-settings.md).
