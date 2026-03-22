# Nastavení projektu

Postranní panel Nastavení projektu <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> v Chloros vám umožňuje konfigurovat všechny aspekty zpracování obrazu, detekci kalibračních cílů, výpočty multispektrálních indexů a možnosti exportu pro váš projekt. Tato nastavení se ukládají spolu s projektem a lze je uložit jako šablony pro opakované použití v různých projektech.

## Přístup k nastavení projektu

Přístup k nastavení projektu:

1. Otevřete projekt v Chloros
2. Klikněte na záložku **Nastavení projektu**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> v levém postranním panelu
3. Panel nastavení zobrazí všechny dostupné možnosti konfigurace uspořádané podle kategorií

***

## Detekce cílů

Tato nastavení určují, jak Chloros detekuje a zpracovává kalibrační cíle ve vašich snímcích.

### Minimální plocha kalibračního vzorku (px)

* **Typ**: Číslo
* **Rozsah**: 0 až 10 000 pixelů
* **Výchozí**: 25 pixelů
* **Popis**: Nastavuje minimální plochu (v pixelech) potřebnou k tomu, aby byla detekovaná oblast považována za platný vzorek kalibračního cíle. Menší hodnoty detekují menší cíle, ale mohou zvýšit počet falešných pozitiv. Větší hodnoty vyžadují pro detekci větší a jasnější oblasti cílů.
* **Kdy upravit**:
  * Zvyšte hodnotu, pokud dochází k falešným detekcím na malých artefaktech v obraze
  * Snižte hodnotu, pokud se vaše kalibrační cíle v obrazech jeví jako malé a nejsou detekovány

### Minimální shlukování cílů (0–100)

* **Typ**: Číslo
* **Rozsah**: 0 až 100
* **Výchozí**: 60
* **Popis**: Řídí prahovou hodnotu seskupování pro seskupování oblastí podobné barvy při detekci kalibračních cílů. Vyšší hodnoty vyžadují, aby bylo seskupeno více podobných barev, což vede k konzervativnější detekci cílů. Nižší hodnoty umožňují větší barevnou variabilitu v rámci cílové skupiny.
* **Kdy upravit**:
  * Zvyšte, pokud se kalibrační cíle rozdělují na více detekcí
  * Snižte, pokud nejsou plně detekovány kalibrační cíle s barevnými odchylkami

***

## Zpracování

Tato nastavení určují, jak Chloros zpracovává a kalibruje vaše snímky.

### Korekce vinětace

* **Typ**: Zaškrtávací políčko
* **Výchozí**: Zapnuto (zaškrtnuto)
* **Popis**: Použije korekci vinětace k vyrovnání ztmavnutí objektivu na okrajích snímků. Vinětace je běžný optický jev, při kterém se rohy a okraje snímku jeví tmavší než střed kvůli vlastnostem objektivu.
* **Kdy deaktivovat**: Deaktivujte pouze v případě, že vaše kombinace fotoaparátu a objektivu již použila korekci vinětace, nebo pokud chcete vinětaci ručně opravit při následném zpracování.

### Kalibrace odrazivosti / vyvážení bílé

* **Typ**: Zaškrtávací políčko
* **Výchozí**: Zapnuto (zaškrtnuto)
* **Popis**: Zapíná automatickou kalibraci odrazivosti pomocí detekovaných kalibračních cílů ve vašich snímcích. To normalizuje hodnoty odrazivosti v celém datovém souboru a zajišťuje konzistentní měření bez ohledu na světelné podmínky.
* **Kdy vypnout**: Vypněte pouze v případě, že chcete zpracovávat surové, nekalibrované snímky, nebo pokud používáte jiný kalibrační postup.

### Metoda debayeringu

* **Typ**: Rozbalovací nabídka
* **Možnosti**:
  * Standardní (rychlé, střední kvalita)
  * S ohledem na texturu (pomalé, nejvyšší kvalita) \[Chloros+]
* **Výchozí**: Standardní (rychlé, střední kvalita)
* **Popis**: Vybírá algoritmus demosaicingu použitý k převodu surových dat snímače s Bayerovým vzorem na plnobarevné snímky. Metoda „Standardní (rychlé, střední kvalita)“ poskytuje optimální rovnováhu mezi rychlostí zpracování a kvalitou snímku. Metoda „S ohledem na texturu (pomalá, nejvyšší kvalita)“ \[Chloros+] využívá vysoce kvalitní demosaicing s ohledem na hrany v kombinaci s modelem odšumování AI/ML, který odstraní téměř veškerý šum způsobený demosaicingem. Model „S ohledem na texturu“ vyžaduje ke spuštění paměť GPU (VRAM). Pro rychlejší zpracování doporučujeme jej používat, pokud máte k dispozici více než 4 GB VRAM.
* **Poznámka**: V budoucích verzích Chloros mohou být přidány další metody debayeringu.

### Minimální interval rekalibrace

* **Typ**: Číslo
* **Rozsah**: 0 až 3 600 sekund
* **Výchozí**: 0 sekund
* **Popis**: Nastavuje minimální časový interval (v sekundách) mezi použitím kalibračních cílů. Při nastavení na 0 bude Chloros používat každý detekovaný kalibrační cíl. Při nastavení na vyšší hodnotu bude Chloros používat pouze kalibrační cíle, které jsou od sebe odděleny alespoň tímto počtem sekund, což zkracuje dobu zpracování u datových sad s častým zachycením kalibračních cílů.
* **Kdy upravit**:
  * Nastavte na 0 pro maximální přesnost kalibrace, když se mění světelné podmínky
  * Zvyšte (např. na 60–300 sekund) pro rychlejší zpracování, když je osvětlení konzistentní a máte časté snímky kalibračních cílů

### Časový posun světelného senzoru

* **Typ**: Číslo
* **Rozsah**: -12 až +12 hodin
* **Výchozí hodnota**: 0 hodin
* **Popis**: Určuje časové posunutí (v hodinách od UTC) pro časová razítka dat světelného senzoru. Používá se při zpracování datových souborů PPK (Post-Processed Kinematic) k zajištění správné časové synchronizace mezi snímky a daty GPS.
* **Kdy upravit**: Nastavte toto na časové posunutí vašeho místního časového pásma, pokud vaše PPK data používají místní čas namísto UTC. Například:
  * Pacifický čas: -8 nebo -7 (v závislosti na letním čase)
  * Východní čas: -5 nebo -4 (v závislosti na letním čase)
  * Středoevropský čas: +1 nebo +2 (v závislosti na letním čase)

### Použít korekce PPK

* **Typ**: Zaškrtávací políčko
* **Výchozí**: Zakázáno (nezaškrtnuto)
* **Popis**: Umožňuje použití korekcí Post-Processed Kinematic (PPK) z datových záznamníků MAPIR obsahujících GPS (GNSS). Pokud je tato funkce povolena, Chloros použije všechny soubory protokolu .daq obsahující data o expozici ve vašem projektovém adresáři a aplikuje na vaše snímky přesné korekce geolokace.
* **Požadavek**: V projektovém adresáři musí být přítomen soubor protokolu .daq se záznamy o expozici
* **Kdy povolit**: Doporučuje se vždy povolit korekci PPK, pokud máte v souboru protokolu .daq záznamy o zpětné vazbě expozice.

### Expozice Pin 1

* **Typ**: Výběr z rozevíracího seznamu
* **Viditelnost**: Viditelné pouze tehdy, když je povoleno „Použít korekce PPK“ A jsou k dispozici data expozice pro Pin 1
* **Možnosti**:
  * Názvy modelů kamer detekované v projektu
  * „Nepoužívat“ – Ignorovat tento expoziční pin
* **Výchozí**: Automaticky vybráno na základě konfigurace projektu
* **Popis**: Přiřadí konkrétní kameru k pinu expozice 1 pro časovou synchronizaci PPK. Pin expozice zaznamenává přesný čas spuštění závěrky kamery, což je zásadní pro přesnou geolokaci PPK.
* **Chování automatického výběru**:
  * Jedna kamera + jeden pin: Automaticky vybere kameru
  * Jedna kamera + dva piny: Pin 1 je automaticky přiřazen kameře
  * Více kamer: Je vyžadován ruční výběr

### Expozice Pin 2

* **Typ**: Výběr z rozevíracího seznamu
* **Viditelnost**: Viditelné pouze tehdy, je-li povolena možnost „Použít PPK korekce“ A jsou-li k dispozici údaje o expozici pro Pin 2
* **Možnosti**:
  * Názvy modelů kamer detekované v projektu
  * „Nepoužívat“ – tento pin expozice ignorovat
* **Výchozí**: Automaticky vybráno na základě konfigurace projektu
* **Popis**: Při použití konfigurace se dvěma kamerami přiřadí konkrétní kameru k pinu expozice 2 pro časovou synchronizaci PPK.
* **Chování automatického výběru**:
  * Jedna kamera + jeden pin: Pin 2 se automaticky nastaví na „Nepoužívat“
  * Jedna kamera + dva piny: Pin 2 se automaticky nastaví na „Nepoužívat“
  * Více kamer: Je vyžadován ruční výběr
* **Poznámka**: Stejnou kameru nelze přiřadit současně k pinu 1 i k pinu 2.***

## Index

Tato nastavení vám umožňují konfigurovat multispektrální indexy pro analýzu a vizualizaci.

### Přidat index

* **Typ**: Speciální panel pro konfiguraci indexů
* **Popis**: Otevře interaktivní panel, kde můžete vybrat a nakonfigurovat multispektrální vegetační indexy (NDVI, NDRE, EVI atd.) pro výpočet během zpracování snímků. Můžete přidat více indexů, každý s vlastním nastavením vizualizace.
* **Dostupné indexy**: Systém obsahuje více než 30 předdefinovaných multispektrálních indexů, včetně:
  * NDVI (Normalizovaný rozdíl vegetace)
  * NDRE (Normalizovaný rozdíl RedEdge)
  * EVI (vylepšený vegetační index)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * A mnoho dalších (kompletní seznam najdete v [Vzorce multispektrálních indexů](multispectral-index-formulas.md))
* **Funkce**:
  * Výběr z předdefinovaných vzorců indexů
  * Konfigurace barevných přechodů vizualizace (LUT – Look-Up Tables)
  * Nastavení prahových hodnot pro analýzu
  * Vytváření vlastních vzorců indexů

### Vlastní vzorce (funkce Chloros+)

* **Typ**: Pole definic vlastních vzorců
* **Popis**: Umožňuje vytvářet a ukládat vlastní vzorce multispektrálních indexů pomocí matematických operací s pásmy. Vlastní vzorce se ukládají spolu s nastavením projektu a lze je používat stejně jako vestavěné indexy.
* **Jak vytvořit**:
  1. V panelu konfigurace indexu vyhledejte možnost vlastního vzorce
  2. Definujte svůj vzorec pomocí identifikátorů pásem (např. NIR, Red, Green, Blue)
  3. Uložte vzorec pod popisným názvem
* **Syntaxe vzorce**: Podporovány jsou standardní matematické operace, včetně:
  * Aritmetika: `+`, `-`, `*`, `/`
  * Závorky pro pořadí operací
  * Odkazy na pásma: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Export

Tato nastavení určují formát a kvalitu exportovaných zpracovaných obrázků.

### Formát kalibrovaného obrázku

* **Typ**: Výběr z rozevíracího seznamu
* **Možnosti**:
  * **TIFF (16bitový)** – Nekomprimovaný 16bitový formát TIFF
  * **TIFF (32bitový, procenta)** – 32bitový formát TIFF s hodnotami odrazivosti vyjádřenými v procentech
  * **PNG (8bitový)** - Komprimovaný 8bitový formát PNG
  * **JPG (8bitový)** - Komprimovaný 8bitový formát JPEG
* **Výchozí**: TIFF (16bitový)
* **Popis**: Volí formát souboru pro ukládání zpracovaných a kalibrovaných snímků.
* **Doporučení ohledně formátu**:
  * **TIFF (16bitový)**: Doporučeno pro vědeckou analýzu a profesionální pracovní postupy. Zachovává maximální kvalitu dat bez kompresních artefaktů. Nejvhodnější pro multispektrální analýzu a další zpracování v GIS softwaru.
  * **TIFF (32bitový, procenty)**: Nejvhodnější pro pracovní postupy, které vyžadují hodnoty odrazivosti v procentech (0–100 %). Nabízí maximální přesnost pro radiometrická měření.
  * **PNG (8bitový)**: Vhodný pro prohlížení na webu a obecnou vizualizaci. Menší velikost souborů díky bezeztrátové kompresi, ale snížený dynamický rozsah.
  * **JPG (8-bit)**: Nejmenší velikost souborů, nejvhodnější pouze pro náhledy a zobrazení na webu. Používá ztrátovou kompresi, která není vhodná pro vědeckou analýzu.***

## Uložit šablonu projektu

Tato funkce vám umožňuje uložit aktuální nastavení projektu jako šablonu pro opakované použití.

* **Typ**: Textové pole + tlačítko Uložit
* **Popis**: Zadejte popisný název pro šablonu nastavení a klikněte na ikonu uložení. Šablona uloží všechna aktuální nastavení projektu (detekci cíle, možnosti zpracování, indexy a formát exportu) pro snadné opětovné použití v budoucích projektech.
* **Případy použití**:
  * Vytvořte šablony pro různé kamerové systémy (RGB, multispektrální, NIR)
  * Uložte standardní konfigurace pro konkrétní typy plodin nebo pracovní postupy analýzy
  * Sdílejte jednotná nastavení v rámci týmu
* **Jak používat**:
  1. Nakonfigurujte všechna požadovaná nastavení projektu
  2. Zadejte název šablony (např. „RedEdge Survey3 NDVI Standard“)
  3. Klikněte na ikonu uložení
  4. Šablonu lze nyní načíst při vytváření nových projektů

***

## Složka pro ukládání projektů

Toto nastavení určuje, kam se nové projekty ukládají ve výchozím nastavení.

* **Typ**: Zobrazení cesty k adresáři + tlačítko Upravit
* **Výchozí (Windows)**: `C:\Users\[Username]\Chloros Projects`
* **Výchozí (Linux)**: `~/.local/share/chloros/projects`
* **Popis**: Zobrazuje aktuální výchozí adresář, do kterého se vytvářejí nové projekty Chloros. Kliknutím na ikonu úprav vyberte jiný adresář.
* **Kdy změnit**:
  * Nastavte síťový disk pro spolupráci v týmu
  * Změňte na disk s větším úložným prostorem pro velké datové sady
  * Uspořádejte projekty podle roku, klienta nebo typu projektu do různých složek
* **Poznámka**: Změna tohoto nastavení se týká pouze NOVÝCH projektů. Stávající projekty zůstanou na svých původních místech.***

## Trvalost nastavení

Všechna nastavení projektu se automaticky ukládají spolu s vaším projektovým souborem (formát projektu `.mapir`). Když projekt znovu otevřete, všechna nastavení se obnoví přesně tak, jak jste je zanechali.

### Hierarchie nastavení

Nastavení se aplikují v následujícím pořadí:

1. **Výchozí nastavení systému** – Vestavěná výchozí nastavení definovaná Chloros
2. **Nastavení šablony** – pokud při vytváření projektu načtete šablonu
3. **Uložená nastavení projektu** – nastavení uložená spolu se souborem projektu
4. **Ruční úpravy** – jakékoli změny, které provedete během aktuální relace

### Nastavení a zpracování obrázků

Většina změn nastavení (zejména v kategoriích Zpracování a Export) spustí nové zpracování snímků, aby se zohlednila nová nastavení. Některá nastavení jsou však „pouze pro export“ a nevyžadují okamžité nové zpracování:

* Uložit šablonu projektu
* Pracovní adresář
* Formát kalibrovaného snímku (platí při exportu)

***

## Osvědčené postupy

1. **Začněte s výchozími nastaveními**: Výchozí nastavení funguje dobře pro většinu kamerových systémů MAPIR a typických pracovních postupů.
2. **Vytvořte šablony**: Jakmile optimalizujete nastavení pro konkrétní pracovní postup nebo kameru, uložte je jako šablonu, abyste zajistili konzistenci napříč projekty.
3. **Otestujte před úplným zpracováním**: Při experimentování s novými nastaveními otestujte nejprve malou část snímků, než zpracujete celý datový soubor.
4. **Zaznamenejte svá nastavení**: Používejte popisné názvy šablon, které označují kamerový systém, typ zpracování a zamýšlené použití (např. „Survey3\_RGB\_NDVI\_Agriculture“).
5. **Výběr formátu exportu**: Vyberte formát exportu podle konečného použití:
   * Vědecká analýza → TIFF (16bitový nebo 32bitový)
   * Zpracování GIS → TIFF (16bitový)
   * Rychlá vizualizace → PNG (8bitový)
   * Sdílení na webu → JPG (8bitový)

***

Další informace o multispektrálních indexech v Chloros najdete na stránce [Vzorce multispektrálních indexů](multispectral-index-formulas.md).
