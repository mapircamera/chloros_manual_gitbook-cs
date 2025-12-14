# Dokončení zpracování

Jakmile Chloros dokončí zpracování, je čas zkontrolovat výsledky, ověřit kvalitu výstupu a připravit zpracované obrázky pro použití ve vašem pracovním postupu. Tato stránka vás provede posledními kroky a dalšími akcemi.

## Indikace dokončení zpracování

Po úspěšném dokončení zpracování se zobrazí několik indikátorů:

* ✅ **Indikátor průběhu**: Dosáhne 100% dokončení
* ✅ **Debugovací protokol**: Zobrazí zprávu „Zpracování dokončeno“
* ✅ **Tlačítko Start**: Znovu se aktivuje (připraveno pro další zpracování)
* ✅ **Výstupní soubory**: Všechny zpracované obrázky jsou uloženy do podsložky modelu fotoaparátu

***

## Vyhledání zpracovaných obrázků

### Otevření výstupní složky

1. Klikněte na ikonu **Hlavní nabídka** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> (vlevo nahoře)
2. Vyberte **„Otevřít složku projektu“**
3. Otevře se průzkumník souborů v adresáři projektu
4. Vyhledejte projekt podle názvu

***

## Kontrola zpracovaných obrázků

### Rychlý náhled v prohlížeči souborů

**Windows vestavěný náhled:**

1. Přejděte do podsložky modelu fotoaparátu
2. Vyberte obrazový soubor
3. Náhled se zobrazí v okně náhledu Windows Explorer
4. Pomocí kláves se šipkami procházejte obrázky

### Náhled v externích prohlížečích obrázků

**Doporučené prohlížeče:**

* **QGIS** – bezplatný software GIS (nejlepší pro georeferenční multispektrální analýzu)
* **IrfanView** – rychlý, lehký prohlížeč obrázků (podporuje TIFF)
* **Adobe Photoshop** – profesionální editace (podpora TIFF)
* **GIMP** – bezplatná alternativa k Photoshopu
* **Windows Photos** – základní prohlížení (nemusí podporovat 16bitové TIFF)

### Náhled v prohlížeči obrázků Chloros

Pro pokročilou vizualizaci použijte vestavěný prohlížeč obrázků Chloros:

1. Klikněte na miniaturu obrázku v prohlížeči souborů.
2. Obrázek se otevře v hlavní oblasti náhledu.
3. Klikněte na kartu **Prohlížeč obrázků** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> v levém postranním panelu.
4. Pro interaktivní analýzu použijte [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md).

Podrobné pokyny najdete v části [Prohlížeč obrázků](../image-viewer-gui/opening-an-image-full-screen.md).

***

## Kontrola protokolu ladění

### Kontrola varování nebo chyb

1. Otevřete kartu **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> .
2. Projděte si zprávy.
3. Hledejte žlutá varování nebo červené chyby.
4. Zkontrolujte všechny zaznamenané problémy.
5. Požádejte o pomoc podporu MAPIR.

### Uložení protokolu

Chcete-li uchovat záznam o zpracování nebo jej odeslat podpoře MAPIR:

1. Klikněte na tlačítko **„Kopírovat“** nebo **„Stáhnout“**.
2. Uložte jako textový soubor do složky projektu.
3. Přiložte k dokumentaci projektu.
4. V případě potíží odešlete na podporu MAPIR.

***

## Časté problémy s výstupem a jejich řešení

### Problém: Chybějící výstupní soubory

**Možné příčiny:**

* Soubory nesplňovaly kritéria zpracování.
* Pouze cílové obrázky (vyloučené z exportu).
* Během exportu došlo k vyčerpání místa na disku.
* Poškození souboru během zpracování.

**Řešení:**

1. Zkontrolujte protokol ladění, zda neobsahuje zprávy o přeskočení/chybách.
2. Ověřte, zda bylo dostatek místa na disku.
3. Spočítejte soubory: Měly by se shodovat (původní počet – cílový počet) × (indexy + 1)
4. Znovu importujte a znovu zpracujte všechny chybějící soubory.

### Problém: Tmavé nebo světlé okraje (vigneta je stále viditelná)

**Možné příčiny:**

* Korekce vignety je deaktivována.
* Fotoaparát/objektiv není v databázi profilů Chloros.
* Extrémní vigneta přesahující možnosti korekce.

**Řešení:**

1. Ověřte, zda byla v nastavení projektu povolena korekce vinětace.
2. Zkontrolujte, zda byl správně detekován model fotoaparátu.
3. Pokud vinětace přetrvává, kontaktujte podporu MAPIR.

### Problém: Nesprávné barvy nebo hodnoty

**Možné příčiny:**

* Nebyly detekovány žádné kalibrační cíle.
* Byl vybrán nesprávný model kalibračního cíle.
* Kalibrace odrazivosti je deaktivována.
* Cílové obrázky jsou nekvalitní.

**Řešení:**

1. Ověřte, zda byla povolena kalibrace odrazivosti.
2. Zkontrolujte zprávy „Cíl nalezen“ v protokolu ladění.
3. Zkontrolujte kvalitu obrazu cíle.
4. Zpracujte znovu s označenými správnými cíli.

### Problém: Hodnoty NDVI se zdají nesprávné

**Očekávané rozsahy NDVI:**

* **Voda, skály, půda**: -0,1 až 0,2
* **Řídká/nezdravá vegetace**: 0,2 až 0,4
* **Střední vegetace**: 0,4 až 0,6
* **Zdravá, hustá vegetace**: 0,6 až 0,9

**Pokud hodnoty leží mimo tyto rozsahy:**

1. Ověřte, zda byla použita kalibrace odrazivosti.
2. Ověřte, zda byl zahrnut protokol světelného senzoru.
3. Zkontrolujte, zda byly detekovány kalibrační cíle.
4. Ujistěte se, že byl detekován správný model fotoaparátu.
5. Zkontrolujte načasování a podmínky pořízení cílového snímku.

***

## Použití zpracovaných snímků

### Pro fotogrammetrii / tvorbu ortomozaiky

**Doporučený pracovní postup:**

1. **Importujte kalibrované obrazy odrazivosti** do fotogrammetrického softwaru:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Zachovejte metadata EXIF**: Zajistěte, aby byla zachována data GPS pro geotagging.
3. **Kalibrované pracovní postupy**: Pro vědeckou přesnost použijte obrazy odrazivosti.
4. **Zpracujte indexové mozaiky**: Vytvořte ortomozaiky NDVI z jednotlivých indexových snímků
5. **Exportujte georeferencované GeoTIFF**: Pro použití v aplikacích GIS

### Pro analýzu GIS

**Doporučený pracovní postup:**

1. **Načtěte do QGIS, ArcGIS nebo podobného programu**
2. **Použijte 16bitové obrazy odrazivosti TIFF** pro multibandovou analýzu
3. **Použijte indexové obrazy** (NDVI, NDRE) jako připravené vrstvy vegetace
4. **Rastrová kalkulačka**: Kombinujte pásma pro vlastní analýzu
5. **Export**: Vytvořte klasifikační mapy, detekci změn, mapy zdravotního stavu vegetace.

### Pro přímou analýzu / reporting

**Doporučený pracovní postup:**

1. **Použijte indexové snímky s barvami LUT** pro vizuální reporty.
2. **Extrahujte statistiky**: Průměrná hodnota NDVI na pole/parcelu.
3. **Časové řady**: Porovnejte indexy napříč více relacemi
4. **Generujte reporty**: Zahrňte mapy, statistiky a vizualizace

***

## Archivace a zálohování

### Doporučená strategie zálohování

**Co uložit:**

* ✅ **Originální RAW/JPG obrázky** – archivujte na samostatném disku/v cloudu
* ✅ **Zpracované výstupy** – uchovejte kalibrované obrázky a indexy
* ✅ **Projektový soubor** – obsahuje všechna nastavení pro případné opětovné zpracování
* ✅ **Debugovací protokol** – dokumentuje podrobnosti zpracování
* ✅ **Kalibrační cílové obrázky** – pro ověření a opětovné zpracování

**Doporučení pro ukládání:**

* **Okamžitá záloha**: Externí pevný disk
* **Dlouhodobý archiv**: Cloudové úložiště (Google Drive, Dropbox atd.)
* **Kritická data**: Uchovávejte 2–3 kopie na různých místech

***

## Další zpracování

### Opětovné použití nastavení projektu

Pokud budete v budoucnu zpracovávat podobné datové sady:

1. **Uložte šablonu projektu** (pokud jste tak ještě neučinili)
2. **Vytvořte nový projekt** pomocí uložené šablony
3. **Importujte nové obrázky**
4. **Zpracujte** s identickými nastaveními pro zachování konzistence

### Hromadné zpracování více relací

Pro více relací/datových sad:

**Možnost 1: GUI – více projektů**

* Vytvořte samostatný projekt pro každou relaci.
* Použijte konzistentní nastavení šablony.
* Zpracovávejte po jednom.

**Možnost 2: Chloros CLI (pouze Chloros+)**

* Automatizujte dávkové zpracování.
* Zpracovávejte více složek pomocí skriptů.
* Viz [CLI Dokumentace](../CLI.md)

**Možnost 3: Python SDK (pouze Chloros+)**

* Programové ovládání
* Integrace s analytickými procesy
* Viz [API dokumentace](../api-python-sdk.md)

***

## Řešení problémů při následném zpracování

### Opakované zpracování s jinými nastaveními

Pokud výsledky nejsou uspokojivé:

1. Ponechte původní obrázky (nikdy je nemazejte)
2. Otevřete stejný projekt v Chloros
3. Upravte nastavení v panelu Nastavení projektu
4. Zpracujte znovu – výstupy přepíšou předchozí výsledky

### Zpracování podsady obrázků

Chcete-li znovu zpracovat pouze konkrétní obrázky:

1. Vytvořte nový projekt
2. Importujte pouze obrázky, které je třeba znovu zpracovat
3. Použijte stejnou šablonu nastavení
4. Zpracujte menší datový soubor

### Získání pomoci

Pokud narazíte na problémy:

* 📧 **E-mail**: info@mapir.camera (včetně ladicího protokolu)
* 🌐 **Podpora**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Často kladené otázky**: [Často kladené otázky](../faq.md)
* 📖 **Dokumentace**: [Příručka Chloros](../)

***

## Shrnutí: Kompletní pracovní postup

Nyní jste dokončili celý pracovní postup zpracování Chloros:

1. ✅ **Vytvořený projekt** – viz [Projekty](../projects.md)
2. ✅ **Přidány soubory** – viz [Přidávání souborů](adding-files-to-a-project.md)
3. ✅ **Upravila se nastavení** – viz [Úprava nastavení projektu](adjusting-project-settings.md)
4. ✅ **Označené cíle** – viz [Výběr cílových obrázků](choosing-target-images.md)
5. ✅ **Zahájeno zpracování** – viz [Zahájení zpracování](starting-the-processing.md)
6. ✅ **Sledovaný průběh** – viz [Sledování zpracování](monitoring-the-processing.md)
7. ✅ **Zkontrolované výsledky** – tato stránka

**Vaše kalibrované multispektrální snímky s korekcí odrazivosti jsou připraveny k analýze!**

***

## Další zdroje

### Pokročilé funkce

* [**Prohlížeč obrázků**](../image-viewer-gui/opening-an-image-full-screen.md) – Interaktivní vizualizace a analýza
* [**Index/LUT Sandbox**](../image-viewer-gui/index-lut-sandbox.md) – Testování vlastního indexu
* [**Vzorec multispektrálního indexu**](../project-settings/multispectral-index-formulas.md) – Kompletní reference indexu

### Automatizace a integrace

* [**CLI Dokumentace**](../CLI.md) – Hromadné zpracování příkazového řádku
* [**Python SDK**](../api-python-sdk.md) – Programová automatizace
* [**Chloros+ Funkce**](../#chloros) – Pokročilé možnosti zpracování

### Podpora a vzdělávání

* [**Často kladené otázky**](../faq.md) – Odpovědi na běžné otázky
* [**Kalibrační cíle**](../calibration-targets.md) – Porozumění kalibraci odrazivosti
* [**Podporované fotoaparáty**](../supported-cameras.md) – Kompatibilní hardware
