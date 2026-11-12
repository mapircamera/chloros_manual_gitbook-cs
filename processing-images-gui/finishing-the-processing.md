# Dokončení zpracování

Jakmile program Chloros dokončí zpracování, je čas zkontrolovat výsledky, ověřit kvalitu výstupu a připravit zpracované obrázky k použití ve vašem pracovním postupu. Tato stránka vás provede závěrečnými kroky a dalšími úkony.

## Indikace dokončení zpracování

Po úspěšném dokončení zpracování se zobrazí několik indikátorů:

* ✅ **Ukazatel průběhu**: Dosáhne 100 % dokončení
* ✅ **Log ladění**: Zobrazí závěrečné řádky `[RUN-SUMMARY]` s počty (snímky, skupiny kamer, cíle, kalibrované snímky, zapsané soubory)
* ✅ **Tlačítko Start**: Znovu se aktivuje (připraveno pro další běh zpracování)
* ✅ **Výstupní soubory**: Všechny zpracované snímky uložené do výstupní struktury projektu (níže)

{% hint style="warning" %}
**Cyklus, který nezapíše žádné snímky, je považován za neúspěšný.** Pokud jste požadovali obrazové výstupy a běh žádný nezapsal, Chloros to nahlásí jako selhání — `[RUN-SUMMARY]` v názvu protokolu naznačuje pravděpodobnou příčinu (nic nebylo importováno, nebyl detekován žádný cíl nebo byly všechny požadované výstupy přeskočeny jako nepoužitelné). Ekvivalent v CLI ukončí běh s nenulovým kódem. Záměrný běh pouze s metadaty (všechny exportované produkty vypnuté, žádné indexy) je stále úspěšný. Viz [Referenční příručka k CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Vyhledání zpracovaných snímků

### Otevření výstupní složky

1. Klikněte na ikonu **Hlavní nabídka** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">
(vlevo nahoře)
2. Vyberte **„Otevřít složku projektu“**

3. Otevře se Průzkumník souborů s adresářem projektu
4. Vyhledejte svůj projekt podle názvu

### Strom výstupních souborů

Výstupy se ukládají **do projektové složky, seskupené podle fotoaparátu a následně podle formátu souboru**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per selected index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

* **Složka fotoaparátu**: `LATT-<sensor>-<lens>-F<filter>` pro LATTICE (odpovídá EXIF záznamu `Model`), `<model>_<filter>` pro Survey3 (např. `Survey3N_RGN`). Dvě kamery, které sdílejí snímač a filtr, ale liší se objektivem, mají oddělené stromy — liší se vinětace, zorné pole a zkreslení.
* **Složka formátu**: řídí se nastavením exportního formátu — `tiff16`, `tiff8`, `png8`, `jpg8`, nebo `tiff32` pro formát TIFF (32bitový, procenty). Radiance je vždy typu float32 a vždy se ukládá pod `tiff32`.
* **Složky produktů**:
  * `Reflectance_Calibrated_Images/` — kalibrovaná odrazivost
  * `Debayered_Images/` — lineární demosaikování (LATTICE): odstraněno tmavé posunutí a provedena demosaika, bez flat fieldu ani unmixu, v nativní orientaci snímače
  * `Preview_Images/` — náhled na displeji (LATTICE)
  * `Radiance_Images/` — spektrální radiance typu float32, W/m²/sr/nm (multispektrální LATTICE)
  * `Vignette_Corrected_Images/` **nebo** `Sensor_Response_Images/` — nekalibrovaná záložní varianta pro snímky bez referenční odrazivosti; v každém běhu existuje přesně jeden z těchto dvou souborů, vybraný nastavením korekce vinětace
  * `<INDEX>_Index_Images/` — jedna složka pro každý vybraný index (např. `NDVI_Index_Images`)
* **`clamp_masks/`** (novinka ve verzi 1.3.0, pouze pro multispektrální kamery LATTICE M3C): objeví se ve formátové složce vedle produktových složek, pokud pásmo při oddělování spektrálních složek ořízne záporné hodnoty na nulu v produktu s radiancí nebo odrazivostí. Obsahuje jednu masku na pixel pro každý dotčený produkt, např. `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif`. Masky se nikdy nenacházejí uvnitř složky s produkty, takže složku s produkty lze předat fotogrammetrickému softwaru tak, jak je. Viz [Masky omezující hodnoty](../output-image-formats.md#clamp-masks).

{% hint style="info" %}
**Každý exportovaný produkt si zachovává název zdrojového souboru.**Export radiance produktu `capture_..._raw.tif` se stále nazývá `capture_..._raw.tif` — pouze se nachází ve složce `tiff32/Radiance_Images/`.**Produkt identifikuje složka, nikoli název souboru**, takže při hledání `*radiance*.tif` se nic nenajde; místo toho hledejte podle adresáře.
{% endhint %}

<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-M3C multispectral camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images, clamp_masks) and tiff32 (Radiance_Images, clamp_masks) subfolders visible -->


### Kolik souborů by mělo být?

Nepočítejte podle vzorce — počet výstupů závisí na tom, které produkty byly povoleny a které se vztahují na jednotlivé kamery (např. kamery RGB nemají radianci/odrazivost). Rozhodující počet najdete v protokolu: poslední řádek `[RUN-SUMMARY]` uvádí přesný počet zapsaných souborů a vysvětlující řádky objasňují vše, co bylo vynecháno.

***

## Kontrola zpracovaných snímků

### Rychlý náhled v Průzkumníku souborů

**Integrovaný náhled v systému Windows:**

1. Přejděte do složky s výstupy (např. `tiff16/Reflectance_Calibrated_Images/`)
2. Vyberte obrazový soubor
3. Náhled se zobrazí v panelu náhledu Průzkumníku Windows

4. Pomocí kláves se šipkami procházejte obrázky

### Náhled v externích prohlížečích obrázků

**Doporučené prohlížeče:*** **QGIS** – bezplatný GIS software (nejlepší pro georeferencovanou multispektrální analýzu)
* **IrfanView** – rychlý a nenáročný prohlížeč obrázků (podporuje formát TIFF)
* **Adobe Photoshop** – profesionální úpravy (podpora formátu TIFF)
* **GIMP** – bezplatná alternativa k programu Photoshop
* **Windows Photos** – základní prohlížení (nemusí podporovat 16bitový formát TIFF)

### Náhled v prohlížeči obrázků Chloros


Pro pokročilou vizualizaci použijte vestavěný prohlížeč obrázků Chloros:

1. Klikněte na miniaturu obrázku v prohlížeči souborů
2. Obrázek se otevře v hlavní oblasti náhledu
3. Klikněte na záložku **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
v levém postranním panelu
4. Pro interaktivní analýzu použijte [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md)

Podrobné pokyny najdete v [prohlížeči obrázků](../image-viewer-gui/opening-an-image-full-screen.md).

***

## Čtení hodnot odrazivosti pixelů (GIS / Pix4D / Skripty)

Odrazivost je uložena jako celé číslo DN a **hodnota DN, která odpovídá ρ = 1,0, závisí na zdrojové kameře**:

| Zdroj          | ρ = 1,0 je | Jak zjistit                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (rezerva až do ρ 2,0) | Soubor je opatřen značkou XMP `Chloros:PixelScale=32768` |
| Survey3          | **65535** (omezeno na ρ 1,0)     | Žádný tag `Chloros:PixelScale` — tato absence je signálem (produkty Survey3 obsahují jiné tagy zpracování typu `Chloros:*`) |

**Přečtěte značku `Chloros:PixelScale` a vydělte jí**, místo abyste předpokládali paušální hodnotu 65535 — vydělením odrazivosti LATTICE číslem 65535 se každá hodnota tiše sníží na polovinu. Soubor odrazivosti z doby před verzí 1.3.0, který byl zapsán jako 8bitový, byl oříznut, nikoli přepočítán, a neobsahuje značku měřítka: místo dělení jej znovu exportujte v 16bitovém nebo 32bitovém formátu. Podrobnosti najdete v části [Formáty výstupních obrázků](../output-image-formats.md).

***

## Metadata přenášená do exportů

Každý výstup si zachovává **blok GPS**a**podblok EXIF** zdrojového snímku, takže
export obsahuje soubory `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` a
`CameraSerialNumber`, stejně jako georeferencování.

{% hint style="warning" %}
**Pokud se ortomozaika vygeneruje v nesmyslné měřítku, nejprve zkontrolujte `FocalLength`.**
Pix4D vypočítává vzdálenost vzorku na zemi (GSD) na základě ohniskové vzdálenosti a nadmořské výšky. Bez tohoto tagu
se vrátí k naprosto nesprávnému měřítku – při jednom měřeném letu se 49 snímky byl pomerančový háj o rozměrech 411 m × 160 m
pomerančový háj byl rekonstruován jako 47,8 km × 13 km, což vedlo k vytvoření 455-megapixelové ortomosaiky převážně
prázdného prostoru. Pomalé skládání dlaždic a nečekaně velký soubor jsou příznaky tohoto problému, nikoli samostatné
problémy.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Nekopírují se *všechny* tagy. Strukturální tagy IFD0 jsou záměrně vynechány (jejich kopírování
poškozuje výstup LATTICE) a `ExifImageWidth` / `ExifImageHeight` jsou vyloučeny,
protože popisují původní snímání — export, jehož velikost byla změněna, by jinak
uváděl rozměry, které jsou v rozporu s jeho vlastním rastrem.

Chloros také zapisuje vlastní XMP tagy `Chloros:*` popisující, co bylo s každým produktem provedeno
(například `Chloros:ProcessingLevel`, `Chloros:CalibrationApplied`,
`Chloros:VignetteCorrected`). Od verze 1.3.0 je produkty Survey3 obsahují ve všech režimech zpracování
režimu zpracování, indexové produkty popisují samotný index (`Chloros:ProcessingLevel=index`,
vzorec a zda byl vypočítán z odrazivosti nebo radiance) namísto opakování
značek zdrojových surových dat a produkty ve složkách s názvem delším než 260 znaků si zachovávají svá metadata
také. Viz [Formáty výstupních obrázků](../output-image-formats.md#index-products).

***

## Kontrola ladicího protokolu

### Kontrola varování a chyb

1. Otevřete kartu **Log ladění** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

2. Projděte si zprávy
3. Hledejte žlutá varování nebo červené chyby
4. Přečtěte si řádky `[RUN-SUMMARY]` a případné rady
5. Obraťte se na podporu MAPIR


### Uložení protokolu

Chcete-li si uchovat záznam o zpracování nebo jej odeslat technické podpoře MAPIR:

1. Klikněte na tlačítko **„Kopírovat“**nebo**„Stáhnout“**

2. Uložte jako textový soubor do složky projektu
3. Přiložte k dokumentaci projektu
4. V případě výskytu problémů odešlete záznam technické podpoře MAPIR Chcete-li nahlásit problém přímo společnosti MAPIR, použijte ikonu chyby (**Nahlásit chybu**) v horní části karty Debug Log — viz [Nahlášení chyby z karty protokolu](monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).

***

## Časté problémy s výstupem a jejich řešení

### Problém: Chybějící výstupní soubory

**Možné příčiny:**

* Produkt se na danou kameru nevztahuje (např. radiance/reflektance pro RGB kamery — je to uvedeno v protokolu)
* Chyběla požadovaná reference (např. odrazivost bez cíle a bez `.daq` sestupného záření)
* Zaškrtávací políčko pro export produktu bylo v nastavení projektu deaktivováno
* Během exportu došlo k vyčerpání místa na disku

**Řešení:**

1. Zkontrolujte rady `[RUN-SUMMARY]` a řádky `[EXPORT-CHECK]` v ladicím protokolu — vysvětlují vynechání jednotlivých kamer
2. Zkontrolujte zaškrtávací políčka pro export produktů v [Nastavení projektu](adjusting-project-settings.md)
3. Zkontrolujte, zda bylo k dispozici dostatek místa na disku
4. Po odstranění příčiny proveďte zpracování znovu

### Problém: Tmavé nebo světlé okraje (stále viditelné vinětace)

**Možné příčiny:**

* Korekce vinětace je vypnutá
* Kamera/objektiv není v databázi profilů Chloros

* Extrémní vinětace přesahující možnosti korekce

**Řešení:**

1. Ověřte, zda byla v Nastaveních projektu zapnuta korekce vinětace
2. Zkontrolujte, zda byl správně detekován model kamery
3. Pokud vinětace přetrvává, kontaktujte podporu MAPIR


### Problém: Nesprávné barvy nebo hodnoty

**Možné příčiny:**

* Nebyly detekovány kalibrační cíle
* Byl vybrán nesprávný model kalibračního cíle
* Kalibrace odrazivosti je vypnutá
* Špatná kvalita snímků cílů

**Řešení:**

1. Ověřte, zda byla kalibrace odrazivosti zapnutá
2. Zkontrolujte zprávy „Target found“ v ladicím protokolu
3. Zkontrolujte kvalitu snímků kalibračních terčů
4. Proveďte nové zpracování s označením správných terčů

### Problém: Hodnoty NDVI se zdají nesprávné

**Očekávané rozsahy hodnot NDVI:*** **Voda, skály, půda**: -0,1 až 0,2
* **Řídká/nezdravá vegetace**: 0,2 až 0,4
* **Středně hustá vegetace**: 0,4 až 0,6
* **Zdravá, hustá vegetace**: 0,6 až 0,9**Pokud hodnoty leží mimo tyto rozsahy:**

1. Ověřte, zda byla použita kalibrace odrazivosti
2. Ověřte, zda byl zahrnut protokol světelného senzoru
3. Zkontrolujte, zda byly detekovány kalibrační cíle
4. Ujistěte se, že byl detekován správný model kamery
5. Zkontrolujte načasování a podmínky pořízení snímků kalibračních terčů
6. Pokud indexy počítáte sami ze souborů odrazivosti, ověřte, zda jste provedli dělení hodnotou `Chloros:PixelScale` uvedenou v souboru (viz výše)

***

## Použití zpracovaných snímků

### Pro fotogrammetrii / tvorbu ortomozaiky

**Doporučený pracovní postup:**

1.**Importujte kalibrované snímky odrazivosti** do fotogrammetrického softwaru:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Zachovejte metadata EXIF**: Zajistěte, aby byla zachována GPS data pro geotagování
3. **Kalibrované pracovní postupy**: Pro vědeckou přesnost používejte snímky odrazivosti — odrazivost LATTICE obsahuje kalibrační značky XMP, které Pix4D čte
4. **Zpracujte indexové mozaiky**: Vytvořte ortomozaiky NDVI z jednotlivých indexových snímků
5. **Exportujte georeferencovaný formát GeoTIFF**: Pro použití v aplikacích GIS

### Pro analýzu v GIS

**Doporučený pracovní postup:**

1.**Načtěte do QGIS, ArcGIS nebo podobného programu**

2.**Použijte 16bitové TIFF** snímky odrazivosti pro vícepásmovou analýzu (vydělte hodnotou `Chloros:PixelScale` uvedenou v souboru)
3. **Použijte indexové snímky** (NDVI, NDRE) jako hotové vrstvy vegetace
4. **Rastrový kalkulátor**: Kombinujte pásma pro vlastní analýzu
5. **Export**: Vytvářejte klasifikační mapy, detekci změn, mapy zdravotního stavu vegetace

### Pro přímou analýzu / tvorbu zpráv

**Doporučený postup:**

1.**Použijte indexové snímky s barvami LUT** pro vizuální zprávy
2. **Extrahujte statistiky**: Průměrná hodnota NDVI na pole/parcelu
3. **Časové řady**: Porovnávejte indexy napříč více sezeními
4. **Vytvářejte zprávy**: Zahrňte mapy, statistiky a vizualizace***

## Archivace a zálohování

### Doporučená strategie zálohování

**Co ukládat:*** ✅ **Původní snímky ve formátu RAW/JPG nebo surové záznamy LATTICE** – archivujte na samostatném disku/v cloudu; surové údaje jsou zdrojem pro zpracování a vše ostatní lze z nich znovu vygenerovat
* ✅ **Soubory ze světelných senzorů `.daq` / `.csv`** – Potřebné pro pozdější opětovný výpočet odrazivosti
* ✅ **Zpracované výstupy** – Uchovejte kalibrované snímky a indexy
* ✅ **Projektová složka** (`project.json` a související soubory) – Obsahuje všechna nastavení pro případné opětovné zpracování
* ✅ **Log ladění** – Dokumentuje podrobnosti zpracování
* ✅ **Snímky kalibračních terčů** – Pro ověření a opětovné zpracování**Doporučení pro ukládání:*** **Okamžitá záloha**: Externí pevný disk
* **Dlouhodobý archiv**: Cloudové úložiště (Google Drive, Dropbox atd.)
* **Kritická data**: Uchovejte 2–3 kopie na různých místech***

## Další cykly zpracování

### Opětovné použití nastavení projektu

Pokud budete v budoucnu zpracovávat podobné datové sady:

1. **Uložte šablonu projektu** (pokud jste tak již neučinili)
2. **Vytvořte nový projekt** pomocí uložené šablony
3. **Importujte nové obrázky**

4.**Zpracujte**se stejnými nastaveními pro zajištění konzistence

### Hromadné zpracování více relací

Pro více relací/datových sad:**Možnost 1: Grafické uživatelské rozhraní (GUI) – více projektů**

* Vytvořte samostatný projekt pro každou relaci
* Použijte konzistentní nastavení šablony
* Zpracovávejte po jednom

**Možnost 2: Chloros CLI (pouze Chloros+)**

* Automatizujte hromadné zpracování
* Zpracovávejte více složek pomocí skriptů
* Viz [Dokumentace CLI](../CLI.md) a [Referenční příručka CLI](../reference/cli-reference.md)

**Možnost 3: Python SDK (pouze Chloros+)**

* Programové ovládání
* Integrace s analytickými pipeline
* Viz [dokumentace k API](../api-python-sdk.md) a [referenční příručka k SDK](../reference/sdk-reference.md)

***

## Řešení problémů při následném zpracování

### Opětovné zpracování s odlišnými nastaveními

Pokud výsledky nejsou uspokojivé:

1. Ponechte původní obrázky (nikdy je nemažte)
2. Otevřete stejný projekt v Chloros
3. Upravte nastavení v panelu Nastavení projektu
4. Proveďte zpracování znovu — výstupy se ukládají do stejných produktových složek, takže soubory se stejným názvem z předchozího běhu budou nahrazeny

### Zpracování podskupiny snímků

Chcete-li znovu zpracovat pouze konkrétní snímky:

1. Vytvořte nový projekt
2. Importujte pouze snímky, které vyžadují nové zpracování
3. Použijte stejnou šablonu nastavení
4. Zpracujte menší datový soubor

### Jak získat pomoc

Pokud narazíte na problémy:

* 📧 **E-mail**: info@mapir.camera (přiložte ladicí protokol)
* 🌐 **Podpora**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Často kladené otázky**: [Často kladené otázky](../faq.md)
* 📖 **Dokumentace**: [Příručka k Chloros](../)

***

## Shrnutí: Kompletní pracovní postup

Právě jste dokončili celý pracovní postup zpracování v Chloros:

1. ✅ **Vytvořen projekt** – viz [Projekty](../projects.md)
2. ✅ **Přidání souborů** – viz [Přidávání souborů](adding-files-to-a-project.md)
3. ✅ **Upravili jste nastavení** – viz [Úprava nastavení projektu](adjusting-project-settings.md)
4. ✅ **Označili jste cíle** – viz [Výběr cílových obrázků](choosing-target-images.md)
5. ✅ **Spuštěno zpracování** – viz [Spuštění zpracování](starting-the-processing.md)
6. ✅ **Sledován průběh** – viz [Sledování průběhu zpracování](monitoring-the-processing.md)
7. ✅ **Zkontrolované výsledky** – Tato stránka**Vaše kalibrované multispektrální snímky s korekcí odrazivosti jsou připraveny k analýze!**

***

## Další zdroje

### Pokročilé funkce

* [**Prohlížeč snímků**](../image-viewer-gui/opening-an-image-full-screen.md) – Interaktivní vizualizace a analýza
* [**Index/LUT Sandbox**](../image-viewer-gui/index-lut-sandbox.md) – Testování vlastních indexů
* [**Vzorce multispektrálních indexů**](../project-settings/multispectral-index-formulas.md) – Kompletní přehled indexů

### Automatizace a integrace

* [**Dokumentace k CLI**](../CLI.md) – Dávkové zpracování z příkazového řádku
* [**Python SDK**](../api-python-sdk.md) – Programová automatizace
* [**Funkce Chloros+**](../#chloros) – Pokročilé možnosti zpracování

### Podpora a vzdělávání

* [**Často kladené otázky**](../faq.md) – Odpovědi na běžné dotazy
* [**Kalibrační terče**](../calibration-targets.md) – Vysvětlení kalibrace odrazivosti
* [**Podporované kamery**](../supported-cameras.md) – Kompatibilní hardware
