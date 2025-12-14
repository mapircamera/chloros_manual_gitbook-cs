# Nastavení projektu

Nastavení projektu <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> v Chloros vám umožňují konfigurovat všechny aspekty zpracování obrazu, detekci kalibračního cíle, výpočty multispektrálních indexů a exportní možnosti pro váš projekt. Tato nastavení se ukládají spolu s vaším projektem a lze je uložit jako šablony pro opakované použití v různých projektech.

## Přístup k nastavení projektu

Přístup k nastavení projektu:

1. Otevřete projekt v Chloros
2. Klikněte na kartu **Nastavení projektu**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> v levém postranním panelu.
3. Panel nastavení zobrazí všechny dostupné konfigurační možnosti uspořádané podle kategorií.

***

## Detekce cílů

Tato nastavení řídí, jak Chloros detekuje a zpracovává kalibrační cíle ve vašich obrázcích.

### Minimální plocha kalibračního vzorku (px)

* **Typ**: Číslo
* **Rozsah**: 0 až 10 000 pixelů
* **Výchozí**: 25 pixelů
* **Popis**: Nastavuje minimální plochu (v pixelech) potřebnou k tomu, aby byla detekovaná oblast považována za platný kalibrační vzorek. Menší hodnoty detekují menší cíle, ale mohou zvýšit počet falešných pozitiv. Větší hodnoty vyžadují větší a jasnější cílové oblasti pro detekci.
* **Kdy upravit**:
  * Zvyšte, pokud dochází k falešným detekcím na malých artefaktech obrazu.
  * Snižte, pokud se vaše kalibrační cíle jeví na vašich snímcích jako malé a nejsou detekovány.

### Minimální seskupení cílů (0–100)

* **Typ**: Číslo
* **Rozsah**: 0 až 100
* **Výchozí hodnota**: 60
* **Popis**: Řídí prahovou hodnotu seskupování pro seskupování podobných barevných oblastí při detekci kalibračních cílů. Vyšší hodnoty vyžadují seskupení více podobných barev, což vede k konzervativnější detekci cílů. Nižší hodnoty umožňují větší barevnou variabilitu v rámci cílové skupiny.
* **Kdy upravit**:
  * Zvyšte, pokud jsou kalibrační cíle rozděleny do více detekcí.
  * Snižte, pokud kalibrační cíle s barevnými odchylkami nejsou plně detekovány.

***

## Zpracování

Tato nastavení řídí, jak Chloros zpracovává a kalibruje vaše obrázky.

### Korekce vinětace

* **Typ**: Zaškrtávací políčko
* **Výchozí nastavení**: Povoleno (zaškrtnuto)
* **Popis**: Použije korekci vinětace k vyrovnání ztmavnutí objektivu na okrajích snímků. Vinětace je běžný optický jev, při kterém jsou rohy a okraje snímku tmavší než střed kvůli vlastnostem objektivu.
* **Kdy deaktivovat**: Deaktivujte pouze v případě, že vaše kombinace fotoaparátu a objektivu již použila korekci vinětace, nebo pokud chcete vinětaci ručně korigovat v postprodukci.

### Kalibrace odrazivosti / vyvážení bílé

* **Typ**: Zaškrtávací políčko
* **Výchozí nastavení**: Zapnuto (zaškrtnuto)
* **Popis**: Zapíná automatickou kalibraci odrazivosti pomocí detekovaných kalibračních cílů ve vašich snímcích. To normalizuje hodnoty odrazivosti v celém datovém souboru a zajišťuje konzistentní měření bez ohledu na světelné podmínky.
* **Kdy vypnout**: Vypněte pouze v případě, že chcete zpracovávat nekalibrované snímky v surovém formátu nebo pokud používáte jiný kalibrační pracovní postup.

### Metoda debayer

* **Typ**: Rozbalovací nabídka
* **Možnosti**:
  * Vysoká kvalita (rychlejší) – v současné době jediná dostupná možnost
* **Výchozí nastavení**: Vysoká kvalita (rychlejší)
* **Popis**: Vybere algoritmus demosaicingu používaný k převodu surových dat snímače s Bayerovým vzorem na plnobarevné snímky. Metoda „Vysoká kvalita (rychlejší)“ poskytuje optimální rovnováhu mezi rychlostí zpracování a kvalitou obrazu.
* **Poznámka**: V budoucích verzích Chloros mohou být přidány další metody debayer.

### Minimální interval rekalibrace

* **Typ**: Číslo
* **Rozsah**: 0 až 3 600 sekund
* **Výchozí**: 0 sekund
* **Popis**: Nastavuje minimální časový interval (v sekundách) mezi použitím kalibračních cílů. Při nastavení na 0 bude Chloros používat všechny detekované kalibrační cíle. Při nastavení na vyšší hodnotu bude Chloros používat pouze kalibrační cíle, které jsou od sebe odděleny alespoň tímto počtem sekund, čímž se zkrátí doba zpracování datových sad s častým zachycováním kalibračních cílů.
* **Kdy upravit**:
  * Nastavte na 0 pro maximální přesnost kalibrace při měnících se světelných podmínkách.
  * Zvyšte (např. na 60–300 sekund) pro rychlejší zpracování, pokud je osvětlení konzistentní a máte časté snímky kalibračních cílů.

### Časové pásmo světelného senzoru

* **Typ**: Číslo
* **Rozsah**: -12 až +12 hodin
* **Výchozí**: 0 hodin
* **Popis**: Určuje časový posun (v hodinách od UTC) pro časová razítka dat světelného senzoru. Používá se při zpracování datových souborů PPK (Post-Processed Kinematic) k zajištění správné časové synchronizace mezi snímky a daty GPS.
* **Kdy upravit**: Nastavte na časový posun vašeho místního časového pásma, pokud vaše data PPK používají místní čas namísto UTC. Například:
  * Pacifický čas: -8 nebo -7 (v závislosti na letním čase)
  * Východní čas: -5 nebo -4 (v závislosti na letním čase)
  * Středoevropský čas: +1 nebo +2 (v závislosti na letním čase)

### Použít korekce PPK

* **Typ**: Zaškrtávací políčko
* **Výchozí**: Zakázáno (nezaškrtnuto)
* **Popis**: Umožňuje použití korekcí PPK (Post-Processed Kinematic) z rekordérů MAPIR DAQ obsahujících GPS (GNSS). Pokud je tato možnost povolena, Chloros použije všechny soubory protokolu .daq obsahující data expozičních pinů ve vašem projektovém adresáři a aplikuje na vaše snímky přesné korekce geolokace.
* **Požadavek**: V adresáři projektu musí být přítomen soubor protokolu .daq s položkami expozičního pinu.
* **Kdy povolit**: Doporučujeme vždy povolit korekci PPK, pokud máte v souboru protokolu .daq položky zpětné vazby expozice.

### Expozice 1

* **Typ**: Výběr z rozevíracího seznamu
* **Viditelnost**: Viditelné pouze v případě, že je povoleno „Použít korekce PPK“ A jsou k dispozici data expozice pro pin 1
* **Možnosti**:
  * Názvy modelů kamer detekovaných v projektu
  * „Nepoužívat“ – Ignorovat tento pin expozice
* **Výchozí**: Automaticky vybráno na základě konfigurace projektu
* **Popis**: Přiřadí konkrétní kameru k expozičnímu pinu 1 pro synchronizaci času PPK. Expoziční pin zaznamenává přesný čas spuštění závěrky kamery, což je rozhodující pro přesnou geolokaci PPK.
* **Chování automatického výběru**:
  * Jedna kamera + jeden pin: Automaticky vybere kameru
  * Jedna kamera + dva piny: Pin 1 automaticky přiřazen kameře
  * Více kamer: Je vyžadován ruční výběr

### Pin expozice 2

* **Typ**: Výběr z rozevíracího seznamu
* **Viditelnost**: Viditelný pouze v případě, že je povolena možnost „Použít korekce PPK“ A jsou k dispozici data expozice pro pin 2
* **Možnosti**:
  * Názvy modelů kamer detekovaných v projektu
  * „Nepoužívat“ – Ignorovat tento pin expozice
* **Výchozí**: Automaticky vybráno na základě konfigurace projektu
* **Popis**: Při použití dvou kamer přiřadí konkrétní kameru k expozičnímu pinu 2 pro synchronizaci času PPK.
* **Chování automatického výběru**:
  * Jedna kamera + jeden pin: Pin 2 automaticky nastaven na „Nepoužívat“
  * Jedna kamera + dva piny: Pin 2 se automaticky nastaví na „Nepoužívat“
  * Více kamer: Je vyžadován ruční výběr
* **Poznámka**: Stejnou kameru nelze přiřadit současně k pinu 1 i pinu 2.

***

## Index

Tato nastavení umožňují konfigurovat multispektrální indexy pro analýzu a vizualizaci.

### Přidat index

* **Typ**: Panel pro konfiguraci speciálních indexů
* **Popis**: Otevře interaktivní panel, ve kterém můžete vybrat a nakonfigurovat multispektrální vegetační indexy (NDVI, NDRE, EVI atd.) pro výpočet během zpracování obrazu. Můžete přidat více indexů, každý s vlastními nastaveními vizualizace.
* **Dostupné indexy**: Systém obsahuje více než 30 předdefinovaných multispektrálních indexů, včetně:
  * NDVI (normalizovaný rozdílový vegetační index)
  * NDRE (Normalizovaný rozdíl RedEdge)
  * EVI (Vylepšený index vegetace)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * A mnoho dalších (kompletní seznam najdete v části [Vzorec multispektrálního indexu](multispectral-index-formulas.md))
* **Funkce**:
  * Výběr z předdefinovaných vzorců indexu
  * Konfigurace barevných přechodů vizualizace (LUT – Look-Up Tables)
  * Nastavení prahových hodnot pro analýzu
  * Vytvoření vlastních vzorců indexu

### Vlastní vzorce (Funkce Chloros+)

* **Typ**: Pole definic vlastních vzorců
* **Popis**: Umožňuje vytvářet a ukládat vlastní vzorce multispektrálních indexů pomocí matematických operací s pásmy. Vlastní vzorce se ukládají spolu s nastavením projektu a lze je používat stejně jako vestavěné indexy.
* **Jak vytvořit**:
  1. V panelu konfigurace indexu vyhledejte možnost vlastního vzorce.
  2. Definujte svůj vzorec pomocí identifikátorů pásem (např. NIR, Red, Green, Blue).
  3. Uložte vzorec pod popisným názvem.
* **Syntaxe vzorce**: Podporovány jsou standardní matematické operace, včetně:
  * Aritmetika: `+`, `-`, `*`, `/`
  * Závorky pro pořadí operací
  * Odkazy na pásma: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Export

Tato nastavení řídí formát a kvalitu exportovaných zpracovaných obrázků.

### Kalibrovaný formát obrázku

* **Typ**: Výběr z rozevíracího seznamu
* **Možnosti**:
  * **TIFF (16 bitů)** – Nekomprimovaný 16bitový formát TIFF
  * **TIFF (32bitový, procenta)** – 32bitový formát TIFF s plovoucí desetinnou čárkou a hodnotami odrazivosti vyjádřenými v procentech
  * **PNG (8bitový)** – komprimovaný 8bitový formát PNG
  * **JPG (8bitový)** – komprimovaný 8bitový formát JPEG
* **Výchozí**: TIFF (16bitový)
* **Popis**: Vybere formát souboru pro ukládání zpracovaných a kalibrovaných obrázků.
* **Doporučené formáty**:
  * **TIFF (16bitový)**: Doporučeno pro vědecké analýzy a profesionální pracovní postupy. Zachovává maximální kvalitu dat bez kompresních artefaktů. Nejvhodnější pro multispektrální analýzu a další zpracování v softwaru GIS.
  * **TIFF (32bitový, procenta)**: Nejvhodnější pro pracovní postupy, které vyžadují hodnoty odrazivosti v procentech (0–100 %). Nabízí maximální přesnost pro radiometrická měření.
  * **PNG (8bitový)**: Vhodný pro prohlížení na webu a obecné vizualizace. Menší velikost souborů s bezeztrátovou kompresí, ale snížený dynamický rozsah.
  * **JPG (8bitový)**: Nejmenší velikost souborů, nejvhodnější pouze pro náhledy a zobrazení na webu. Používá ztrátovou kompresi, která není vhodná pro vědecké analýzy.

***

## Uložit šablonu projektu

Tato funkce vám umožňuje uložit aktuální nastavení projektu jako šablonu, kterou lze znovu použít.

* **Typ**: Zadání textu + tlačítko Uložit
* **Popis**: Zadejte popisný název pro šablonu nastavení a klikněte na ikonu uložit. Šablona uloží všechna aktuální nastavení projektu (detekce cíle, možnosti zpracování, indexy a formát exportu) pro snadné opětovné použití v budoucích projektech.
* **Případy použití**:
  * Vytvořte šablony pro různé kamerové systémy (RGB, multispektrální, NIR)
  * Uložte standardní konfigurace pro konkrétní typy plodin nebo analytické pracovní postupy
  * Sdílejte konzistentní nastavení v rámci týmu
* **Jak používat**:
  1. Nakonfigurujte všechna požadovaná nastavení projektu
  2. Zadejte název šablony (např. „RedEdge Survey3 NDVI Standard“).
  3. Klikněte na ikonu uložení.
  4. Šablonu lze nyní načíst při vytváření nových projektů.

***

## Uložit složku projektu

Toto nastavení určuje, kam se nové projekty standardně ukládají.

* **Typ**: Zobrazení cesty k adresáři + tlačítko Upravit
* **Výchozí**: `C:\Users\[Username]\Chloros Projects`
* **Popis**: Zobrazuje aktuální výchozí adresář, do kterého se vytvářejí nové projekty Chloros. Kliknutím na ikonu Upravit vyberte jiný adresář.
* **Kdy změnit**:
  * Nastavte síťový disk pro spolupráci v týmu.
  * Změňte disk s větším úložným prostorem pro velké datové soubory.
  * Uspořádejte projekty podle roku, klienta nebo typu projektu do různých složek.
* **Poznámka**: Změna tohoto nastavení má vliv pouze na NOVÉ projekty. Stávající projekty zůstanou na svých původních místech.

***

## Trvalost nastavení

Všechna nastavení projektu se automaticky ukládají spolu s projektovým souborem (formát projektu `.mapir`). Při opětovném otevření projektu se všechna nastavení obnoví přesně tak, jak jste je uložili.

### Hierarchie nastavení

Nastavení se aplikují v následujícím pořadí:

1. **Výchozí nastavení systému** – vestavěná výchozí nastavení definovaná Chloros
2. **Nastavení šablony** – pokud při vytváření projektu načtete šablonu
3. **Uložená nastavení projektu** – nastavení uložená s projektovým souborem
4. **Ruční úpravy** – všechny změny, které provedete během aktuální relace

### Nastavení a zpracování obrazu

Většina změn nastavení (zejména v kategoriích Zpracování a Export) spustí nové zpracování obrazů, aby se zohlednila nová nastavení. Některá nastavení jsou však „pouze pro export“ a nevyžadují okamžité nové zpracování:

* Uložit šablonu projektu
* Pracovní adresář
* Kalibrovaný formát obrazu (platí při exportu)

***

## Osvědčené postupy

1. **Začněte s výchozím nastavením**: Výchozí nastavení funguje dobře pro většinu kamerových systémů MAPIR a typické pracovní postupy.
2. **Vytvořte šablony**: Jakmile optimalizujete nastavení pro konkrétní pracovní postup nebo kameru, uložte je jako šablonu, abyste zajistili konzistenci napříč projekty.
3. **Před úplným zpracováním proveďte test**: Při experimentování s novými nastaveními proveďte test na malé podskupině obrázků, než zpracujete celý datový soubor.
4. **Zaznamenejte si nastavení**: Používejte popisné názvy šablon, které označují kamerový systém, typ zpracování a zamýšlené použití (např. „Survey3\_RGB\_NDVI\_Agriculture“).
5. **Výběr formátu exportu**: Vyberte formát exportu na základě konečného použití:
   * Vědecká analýza → TIFF (16bitový nebo 32bitový)
   * Zpracování GIS → TIFF (16bitový)
   * Rychlá vizualizace → PNG (8bitový)
   * Sdílení na webu → JPG (8bitový)

***

Další informace o multispektrálních indexech v Chloros naleznete na stránce [Vzorce multispektrálních indexů](multispectral-index-formulas.md).
