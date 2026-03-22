# Dokončení zpracování

Jakmile nástroj Chloros dokončí zpracování, je na čase zkontrolovat výsledky, ověřit kvalitu výstupu a připravit zpracované snímky k použití ve vašem pracovním postupu. Tato stránka vás provede závěrečnými kroky a dalšími úkony.

## Indikace dokončení zpracování

Po úspěšném dokončení zpracování uvidíte několik indikátorů:

* ✅ **Průběhový pruh**: Dosáhne 100% dokončení
* ✅ **Log ladění**: Zobrazí zprávu „Zpracování dokončeno“
* ✅ **Tlačítko Start**: Znovu se aktivuje (připraveno pro další běh zpracování)
* ✅ **Výstupní soubory**: Všechny zpracované obrázky uložené do podsložky modelu fotoaparátu***

## Vyhledání zpracovaných obrázků

### Otevření výstupní složky

1. Klikněte na ikonu **Hlavní menu** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (vlevo nahoře)
2. Vyberte **„Otevřít složku projektu“**

3. Otevře se Průzkumník souborů v adresáři projektu
4. Vyhledejte svůj projekt podle názvu

***

## Prohlížení zpracovaných obrázků

### Rychlý náhled v Průzkumníku souborů

**Windows vestavěný náhled:**

1. Přejděte do podsložky modelu kamery
2. Vyberte obrazový soubor
3. Náhled se zobrazí v Windows panelu náhledu Průzkumníka
4. Pomocí kláves se šipkami procházejte obrázky

### Náhled v externích prohlížečích obrázků

**Doporučené prohlížeče:*** **QGIS** – bezplatný GIS software (nejlepší pro georeferencovanou multispektrální analýzu)
* **IrfanView** – rychlý, lehký prohlížeč obrázků (podporuje TIFF)
* **Adobe Photoshop** – profesionální úpravy (podpora formátu TIFF)
* **GIMP** – bezplatná alternativa k programu Photoshop
* **Windows Photos** – základní prohlížení (nemusí podporovat 16bitový formát TIFF)

### Náhled v prohlížeči obrázků Chloros

Pro pokročilou vizualizaci použijte vestavěný prohlížeč obrázků Chloros:

1. Klikněte na miniaturu obrázku v prohlížeči souborů
2. Obrázek se otevře v hlavní oblasti náhledu
3. Klikněte na záložku **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> v levém postranním panelu
4. Pro interaktivní analýzu použijte [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md)

Podrobné pokyny najdete v [Prohlížeči obrázků](../image-viewer-gui/opening-an-image-full-screen.md).

***

## Kontrola protokolu ladění

### Zkontrolujte varování nebo chyby

1. Otevřete kartu **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> 2. Projděte si zprávy
3. Hledejte žlutá varování nebo červené chyby
4. Zkontrolujte všechny zaznamenané problémy
5. Kontaktujte podporu MAPIR a požádejte o pomoc

### Uložení protokolu

Chcete-li si uchovat záznam o zpracování nebo jej odeslat na podporu MAPIR:

1. Klikněte na tlačítko **„Kopírovat“**nebo**„Stáhnout“**

2. Uložte jako textový soubor do složky projektu
3. Přiložte k dokumentaci projektu
4. V případě výskytu problémů odešlete na podporu MAPIR

***

## Časté problémy s výstupem a jejich řešení

### Problém: Chybějící výstupní soubory

**Možné příčiny:**

* Soubory nesplňovaly kritéria zpracování
* Obrázky pouze pro cíl (vyloučené z exportu)
* Během exportu došlo k vyčerpání místa na disku
* Poškození souboru během zpracování

**Řešení:**

1. Zkontrolujte protokol ladění, zda neobsahuje zprávy o přeskočení/chybách
2. Ověřte, zda bylo k dispozici dostatek místa na disku
3. Spočítejte soubory: Počet by měl odpovídat (původní počet – počet v cíli) × (indexy + 1)
4. Chybějící soubory znovu importujte a znovu zpracujte

### Problém: Tmavé nebo světlé okraje (stále viditelné vinětace)

**Možné příčiny:**

* Korekce vinětace je vypnutá
* Fotoaparát/objektiv není v databázi profilů Chloros
* Extrémní vinětace přesahující možnosti korekce

**Řešení:**

1. Ověřte, zda byla korekce vinětace povolena v nastavení projektu
2. Zkontrolujte, zda byl správně detekován model fotoaparátu
3. Pokud vinětace přetrvává, kontaktujte podporu MAPIR

### Problém: Nesprávné barvy nebo hodnoty

**Možné příčiny:**

* Nebyly detekovány kalibrační cíle
* Byl vybrán nesprávný model kalibračního cíle
* Kalibrace odrazivosti je vypnutá
* Špatná kvalita obrázků cílů

**Řešení:**

1. Ověřte, zda byla povolena kalibrace odrazivosti
2. Zkontrolujte zprávy „Target found“ v ladicím protokolu
3. Zkontrolujte kvalitu snímků cílů
4. Zpracujte znovu s označením správných cílů

### Problém: Hodnoty NDVI se zdají nesprávné

**Očekávané rozsahy NDVI:*** **Voda, skály, půda**: -0,1 až 0,2
* **Řídká/nezdravá vegetace**: 0,2 až 0,4
* **Střední vegetace**: 0,4 až 0,6
* **Zdravá, hustá vegetace**: 0,6 až 0,9**Pokud jsou hodnoty mimo tyto rozsahy:**

1. Ověřte, zda byla použita kalibrace odrazivosti
2. Ověřte, zda byl zahrnut protokol světelného senzoru
3. Zkontrolujte, zda byly detekovány kalibrační cíle
4. Ujistěte se, že byl detekován správný model kamery
5. Zkontrolujte načasování a podmínky pořízení cílového snímku

***

## Použití zpracovaných snímků

### Pro fotogrammetrii / tvorbu ortomozaiky

**Doporučený pracovní postup:**

1.**Importujte kalibrované snímky odrazivosti** do fotogrammetrického softwaru:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Zachovejte metadata EXIF**: Zajistěte, aby byla zachována data GPS pro geotagování
3. **Kalibrované pracovní postupy**: Používejte snímky odrazivosti pro vědeckou přesnost
4. **Zpracujte indexové mozaiky**: Vytvořte ortomozaiky NDVI z jednotlivých indexových snímků
5. **Exportujte georeferencované GeoTIFF**: Pro použití v GIS aplikacích

### Pro GIS analýzu

**Doporučený postup:**

1.**Načtěte do QGIS, ArcGIS nebo podobného programu**

2.**Použijte 16bitové TIFF** snímky odrazivosti pro vícepásmovou analýzu
3. **Použijte indexové snímky** (NDVI, NDRE) jako připravené vrstvy vegetace
4. **Rastrový kalkulátor**: Kombinujte pásma pro vlastní analýzu
5. **Export**: Vytvářejte klasifikační mapy, detekci změn, mapy zdravotního stavu vegetace

### Pro přímou analýzu / reporting

**Doporučený pracovní postup:**

1.**Používejte indexové snímky s barvami LUT** pro vizuální reporty
2. **Extrahujte statistiky**: Průměr NDVI na pole/parcelu
3. **Časové řady**: Porovnejte indexy napříč více sezeními
4. **Vytvořte zprávy**: Zahrňte mapy, statistiky a vizualizace***

## Archivace a zálohování

### Doporučená strategie zálohování

**Co uložit:*** ✅ **Původní snímky RAW/JPG** – archivujte na samostatném disku/v cloudu
* ✅ **Zpracované výstupy** – Uchovejte kalibrované snímky a indexy
* ✅ **Soubor projektu** – Obsahuje všechna nastavení pro případné opětovné zpracování
* ✅ **Log ladění** – Dokumentuje podrobnosti zpracování
* ✅ **Snímky kalibračních terčů** – Pro ověření a opětovné zpracování**Doporučení pro ukládání:*** **Okamžitá záloha**: Externí pevný disk
* **Dlouhodobý archiv**: Úložiště v cloudu (Google Drive, Dropbox atd.)
* **Kritická data**: Uchovejte 2–3 kopie na různých místech***

## Další cykly zpracování

### Opětovné použití nastavení projektu

Pokud budete v budoucnu zpracovávat podobné datové sady:

1. **Uložte šablonu projektu** (pokud jste tak již neučinili)
2. **Vytvořte nový projekt** pomocí uložené šablony
3. **Importujte nové obrázky**

4.**Zpracujte**s identickými nastaveními pro zajištění konzistence

### Hromadné zpracování více relací

Pro více relací/datových sad:**Možnost 1: GUI – více projektů**

* Vytvořte samostatný projekt pro každou relaci
* Použijte konzistentní nastavení šablony
* Zpracovávejte po jednom

**Možnost 2: Chloros CLI (pouze Chloros+)**

* Automatizujte dávkové zpracování
* Zpracovávejte více složek pomocí skriptů
* Viz [Dokumentace k CLI](../CLI.md)

**Možnost 3: Python SDK (pouze Chloros+)**

* Programové ovládání
* Integrace s analytickými pipeline
* Viz [API Dokumentace](../api-python-sdk.md)

***

## Řešení problémů při následném zpracování

### Opětovné zpracování s odlišnými nastaveními

Pokud výsledky nejsou uspokojivé:

1. Ponechte původní obrázky (nikdy je nemažte)
2. Otevřete stejný projekt v Chloros
3. Upravte nastavení v panelu Nastavení projektu
4. Zpracujte znovu – výstupy přepíšou předchozí výsledky

### Zpracování podskupiny obrázků

Chcete-li znovu zpracovat pouze konkrétní obrázky:

1. Vytvořte nový projekt
2. Importujte pouze obrázky, které vyžadují nové zpracování
3. Použijte stejnou šablonu nastavení
4. Zpracujte menší datový soubor

### Získání pomoci

Pokud narazíte na problémy:

* 📧 **E-mail**: info@mapir.camera (přiložte ladicí protokol)
* 🌐 **Podpora**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Často kladené otázky**: [Často kladené otázky](../faq.md)
* 📖 **Dokumentace**: [Chloros Příručka](../)***

## Shrnutí: Kompletní pracovní postup

Právě jste dokončili celý pracovní postup zpracování Chloros:

1. ✅ **Vytvořený projekt** – viz [Projekty](../projects.md)
2. ✅ **Přidány soubory** – viz [Přidávání souborů](adding-files-to-a-project.md)
3. ✅ **Upravena nastavení** – viz [Úprava nastavení projektu](adjusting-project-settings.md)
4. ✅ **Označené cíle** – viz [Výběr cílových obrázků](choosing-target-images.md)
5. ✅ **Spuštěno zpracování** – viz [Spuštění zpracování](starting-the-processing.md)
6. ✅ **Sledován průběh** – viz [Sledování zpracování](monitoring-the-processing.md)
7. ✅ **Zkontrolované výsledky** – Tato stránka**Vaše kalibrované multispektrální snímky s korekcí odrazivosti jsou připraveny k analýze!**

***

## Další zdroje

### Pokročilé funkce

* [**Prohlížeč snímků**](../image-viewer-gui/opening-an-image-full-screen.md) – Interaktivní vizualizace a analýza
* [**Index/LUT Sandbox**](../image-viewer-gui/index-lut-sandbox.md) – Testování vlastních indexů
* [**Vzorce multispektrálních indexů**](../project-settings/multispectral-index-formulas.md) – Kompletní referenční přehled indexů

### Automatizace a integrace

* [**Dokumentace CLI**](../CLI.md) – Dávkové zpracování z příkazového řádku
* [**Python SDK**](../api-python-sdk.md) – Programová automatizace
* [**Chloros+ Funkce**](../#chloros) – Pokročilé možnosti zpracování

### Podpora a vzdělávání

* [**Často kladené otázky**](../faq.md) – Odpovědi na běžné otázky
* [**Kalibrační terče**](../calibration-targets.md) – Vysvětlení kalibrace odrazivosti
* [**Podporované kamery**](../supported-cameras.md) – Kompatibilní hardware
