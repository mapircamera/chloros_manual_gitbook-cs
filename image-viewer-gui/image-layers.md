# Vrstvy obrázků

Pomocí **rozevíracího seznamu vrstev** v pravém horním rohu prohlížeče obrázků můžete přepínat mezi všemi verzemi obrázku, který si právě prohlížíte – od zdrojového snímku přes jednotlivé zpracované výstupy až po vypočítané indexové obrázky – aniž byste museli prohlížeč opustit.

## Co jsou vrstvy snímků?

„Vrstva“ v programu Chloros

je jeden **soubor produktu**přiřazený k jednomu zdrojovému snímku. Import vám poskytne zdrojové soubory; zpracování přidá vrstvu pro každý produkt, který byl v daném běhu vytvořen. Exportované soubory si zachovávají název zdrojového souboru — produkt identifikuje**složka** a název vrstvy je označení, které této složce přiřazuje Chloros

.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## Seznam vrstev

### Vždy k dispozici

| Vrstva | Co to je |
| --- | --- |
| **JPG**(nebo**PNG

**/**TIFF

**) | Základní soubor, který byl získán při snímání. Survey3

importuje soubor `.JPG` vedle každého souboru `.RAW`; snímky z LATTICE přinášejí náhled pro zobrazení ve formátu PNG

nebo TIFF

. Označeno podle toho, co bylo skutečně importováno |
| **RAW (originál)** | Zdrojový surový snímek, zbavený bayesovského kódování pro zobrazení bez aplikovaných korekcí. K dispozici od okamžiku importu — nevyžaduje žádné zpracování |

Záznam LATTICE, jehož základním souborem **je** jeho surový snímek, nemá samostatný základní záznam: `RAW (Original)` jej již pokrývá.

### Výstupy zpracování programu Survey3



| Vrstva | Zapsáno do | Existuje, když |
| --- | --- | --- |
| **RAW (Cíl)** | — | Snímek byl identifikován jako obsahující kalibrační cíl |
| **RAW (Odrazivost)** | `Reflectance_Calibrated_Images/` | Na tomto snímku proběhla úspěšně kalibrace odrazivosti |
| **S korekcí vinětace**| `Vignette_Corrected_Images/` | Snímek nebylo možné kalibrovat podle odrazivosti**a** byla zapnuta *korekce vinětace* |
| **Odezva snímače**| `Sensor_Response_Images/` | Snímek nebylo možné kalibrovat podle odrazivosti**a** *korekce vinětace* byla vypnutá |
| **Vyvážení bílé** | `White_Balanced_Images/` | Byl zapsán produkt s vyvážením bílé |

{% hint style="info" %}
**Korekce vinětace a odezva snímače jsou alternativy, nikdy se nepoužívají současně.** Pro každý model kamery existuje přesně jeden nekalibrovaný záložní produkt na jeden běh a přepínač *Korekce vinětace* určuje, který z nich se použije. Viz [Nastavení projektu](../project-settings/project-settings.md).
{% endhint %}

### Úrovně LATTICE

LATTICE zachycuje rozložení do těchto úrovní v jediném kroku zpracování. Které z nich existují, závisí na přepínačích exportu pro jednotlivé produkty v Nastaveních projektu a na tom, co platí pro daný fotoaparát.

| Vrstva | Zapsáno do | Platí pro |
| --- | --- | --- |
| **RAW (Debayered)** | `Debayered_Images/` | RGB

a multispektrální |
| **RAW (náhled)** | `Preview_Images/` | Multispektrální (roztažení falešných barev) |
| **Vyvážení bílé** | `Preview_Images/` | RGB

hlavní kamery — náhled RGB

je zaregistrován pod tímto názvem, aby odpovídal stejnojmenné vrstvě v programu Survey3

|
| **RAW (radiance)** | `Radiance_Images/` | Pouze multispektrální |
| **RAW (odrazivost)** | `Reflectance_Calibrated_Images/` | Pouze multispektrální, a to pouze v případě, že snímek pokrývá odpovídající záznam `.daq` směřující dolů nebo cíl v rámci snímku, který prošel kontrolou kvality |

Hlavní kamery RGB

nemají radiometrii pro jednotlivá pásma, takže u nich jsou zářivost a odrazivost vynechány jako **neplatné** — protokol to uvádí, místo aby došlo k tichému selhání.

{% hint style="info" %}
**Surové a debayered vrstvy se zarovnávají s produkty (novinka ve verzi 1.3.0).**Surové a debayered soubory LATTICE se ukládají v nativní orientaci snímače a do XMP zaznamenávají orientaci, která k nim patří; ostatní produkty se ukládají s již aplikovanou touto orientací. Prohlížeč a mřížka snímků pro účely zobrazení přepínají mezi**RAW (originál)**a**RAW (po odstranění bayesovského kódu)** podle tohoto záznamu, takže se snímek již neotočí, když přepínáte mezi vrstvou výstupu a surovou vrstvou. Jedná se pouze o zobrazení — soubory se nikdy nepřepisují. Soubory bez záznamu o orientaci (pořízené před verzí 1.3.0 nebo vytvořené jiným softwarem) se zobrazují tak, jak jsou uloženy. Viz [Výstupní formáty obrázků](../output-image-formats.md#orientation).
{% endhint %}

Masky omezovače (`*_clampmask.tif` ve složce `clamp_masks/`) nejsou vrstvy: Chloros

je při importu přeskočí. Viz [Výstupní formáty obrázků](../output-image-formats.md#clamp-masks).

### Vrstvy indexu, LUT a sandboxu

| Typ vrstvy | Příklad | Odkud pochází |
| --- | --- | --- |
| **RAW (`<INDEX>` Index)** | `RAW (NDVI Index)` | Jedna pro každý index nakonfigurovaný v nastavení projektu, vypočítaná během zpracování |
| **`<INDEX>` LUT** | `NDVI LUT` | Barevně mapovaná verze indexu |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Jeden na každý export [Index/LUT Sandbox](index-lut-sandbox.md) export |

Pokud je stejný název indexu nakonfigurován vícekrát s odlišnými nastaveními, druhý a další z nich dostanou v názvu číslo (`RAW (NDVI2 Index)`), aby byly vrstvy rozlišitelné.

***

## Použití výběru vrstev

1. Otevřete obrázek na celou obrazovku kliknutím na miniaturu v mřížce
2. Klikněte na **rozevírací nabídku vrstev** v pravém horním rohu prohlížeče
3. Vyberte vrstvu — obrázek se okamžitě aktualizuje

V rozevíracím seznamu jsou na prvních místech v tomto pořadí uvedeny formáty **JPG, RAW (Original), RAW (Target), RAW (Reflectance)** a za nimi jsou uvedeny všechny ostatní vrstvy v pořadí, v jakém byly produkty zaregistrovány.

### Předvolba vrstvy při procházení

Stisknutím kláves **←**/**→** přejdete na další obrázek a program se pokusí zachovat stejnou vrstvu:

1. **Nejprve přesná shoda** — pokud má další obrázek vrstvu se stejným názvem, zobrazí se právě ta. Díky tomu zůstanete na vrstvě `RAW (NDVI Index)` i při procházení celé sady
2. **Poté shoda podle typu** — indexová vrstva hledá jakoukoli indexovou vrstvu, LUT jakoukoli LUT, odrazivost jakoukoli odrazivost, cíl jakoukoli cílovou vrstvu, originál jakoukoli originální vrstvu, základ jakoukoli základní vrstvu
3. **Poté, pouze u exportních vrstev** — název se zachová, i když seznam vrstev ještě není aktuální, protože soubor již na disku existuje. Díky tomu můžete prohlížet výstupy, zatímco je proces ještě zapisuje
4. **V ostatních případech** — první dostupná vrstva, což je obvykle základní obrázek

Soubory sidecar `.daq` a `.csv` v projektu jsou při navigaci pomocí šipek přeskočeny, takže při procházení obrázků se nikdy nedostanete k záznamu ze světelného senzoru.

Přiblížení a posun se přenášejí i mezi obrazy, což usnadňuje porovnání stejné polohy pole před a po.

***

## Porozumění hodnotám pixelů podle vrstvy

Panel [Hodnoty kurzoru](opening-an-image-full-screen.md#cursor-values) zobrazuje skutečnou hodnotu pro každý kanál pod kurzorem v jednotkách, ve kterých je daná vrstva uložena. Jeho sloupce se mění v závislosti na vrstvě:

| Vrstva | Zobrazená jednotka | Poznámky |
| --- | --- | --- |
| Základní (náhled JPG / PNG

/ TIFF

) | DN, 0–255 | Zobrazené hodnoty, s korekcí gama pro RGB

. Pouze pro vizuální kontrolu |
| RAW (originál) | DN | Surová digitální čísla ze snímače. Osa histogramu udává hloubku: 255 (8 bitů), 4095 (12 bitů) nebo 65535 (16 bitů) |
| RAW (po odstranění bayerského filtru) | DN | Lineární, bez roztažení pro zobrazení |
| RAW (náhled) / s vyvážením bílé | DN | Výstup pro zobrazení — roztažený nebo s korekcí gama. Není určen k měření |
| RAW (radiance) | **W/m²/sr/nm** | Fyzická radiance typu Float32. Žádný sloupec DN |
| RAW (odrazivost) | DN **a %** | Procento vypočítané podle vlastní stupnice daného souboru — viz níže |
| Exporty indexu / LUT / sandbox | Hodnota indexu nebo složky RGB

| Jednokanálový indexový soubor uvádí hodnotu indexu; soubor LUT s barevným mapováním uvádí složky Red

/Green

/Blue

|

### Odrazivost: měřítko je specifické pro každý soubor

{% hint style="warning" %}
**„Vydělit 65 535“ platí pouze pro Survey3

.** Odrazivost LATTICE je uložena v jiném měřítku a smíchání těchto dvou dělitelů je nejčastějším způsobem, jak získat hodnoty odrazivosti, které jsou přesně poloviční oproti tomu, jaké by měly být.
{% endhint %}

| Zdroj | DN odpovídající odrazivosti 1,0 | Identifikováno podle |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | XMP značka `Chloros:PixelScale=32768` vložená do každého exportu odrazivosti LATTICE. Dvojnásobná rezerva znamená, že hodnoty ρ nad 1,0 lze zobrazit, místo aby byly oříznuty |
| **Survey3**| **65535** | Žádný XMP tag měřítka Chloros

— kalibrace Survey3

zapisuje ρ × dtype-max a ořezává na hodnotu 1,0 |

Pro GIS a skriptování: načtěte z souboru `Chloros:PixelScale` a vydělte tím. Pokud značka chybí, soubor má měřítko Survey3

(65535). Prohlížeč, index/LUT sandbox i export indexu určují měřítko stejným způsobem, takže číslo, které vidíte u kurzoru, je číslo použité při výpočtu indexu.

Ukládání specifické pro daný formát nad rámec tohoto měřítka:

* **Formát TIFF

(32bitový, procenta)** ukládá hodnotu DN / 65535 jako číslo typu float
* **Formáty PNG

(8bitový)**a**JPG (8bitový)** ukládají hodnotu DN × 255 / 65535
* **8bitový DN získaný z 8bitového snímku** nebo soubor odrazivosti vytvořený před verzí 1.3.0 je oříznut na rozsah 0–255 a neobsahuje značku měřítka. Panel u těchto souborů zobrazuje pouze hodnotu DN, bez sloupce s procenty

### Rozsahy hodnot indexů

| Rodina indexů | Typický rozsah | Hodnota |
| --- | --- | --- |
| Normalizovaný rozdíl (NDVI

, GNDVI

, NDRE

, ENDVI…) | −1 až +1 | Zdravá vegetace obvykle 0,4–0,9; holá půda blízko 0; voda záporná |
| Opravené o půdu (SAVI

, OSAVI

, MSAVI2…) | přibližně −1 až +1,5 | Hodnoty podobné NDVI

, avšak s potlačeným půdním pozadím |
| Poměr (GRVI

, GCI

, MSR, CIRE…) | neomezeně nad | Poměry rostou bez omezení, jak se jmenovatel blíží nule |
| EVI

/ LAI

| 0 až ~1, 0 až ~3,5 | Mraky a další nasycené pixely posouvají obě hodnoty mimo rozsah — nejprve je zamaskujte |

Přesné vzorce pro každou předvolbu najdete v [Vzorcích multispektrálních indexů](../project-settings/multispectral-index-formulas.md).

***

## Běžné pracovní postupy

### Porovnání před a po

1. Vyberte **RAW (originál)** a všimněte si vinětace a nekalibrovaných hodnot
2. Přepněte na **RAW (Reflectance)**

3. Porovnejte — vinětace odstraněna, hodnoty kalibrovány. Přiblížení a posun zůstávají zachovány, takže se díváte na stejnou oblast terénu

### Prohlédněte si jeden index v celé sadě

1. Otevřete první zpracovaný snímek a vyberte vrstvu s indexem
2. Opakovaně stiskněte **→** — vrstva s indexem vás bude následovat od snímku k snímku
3. Sledujte přitom histogram v postranním panelu: snímek, u kterého dochází k výkyvům v rozložení, stojí za bližší prozkoumání

### Ověření kalibračních cílů

1. Na cílovém snímku vyberte **RAW (Target)**

2. Ověřte, zda je cíl jasně viditelný a detekovaný
3. Přejděte na další cílový snímek — vrstva cílů vás následuje

### Zkontrolujte přesnost hodnot odrazivosti

1. Vyberte **RAW (Reflectance)**

2. Přečtěte si sloupec**%** v panelu Hodnoty kurzoru — pro daný soubor je již správně škálován
3. Proveďte kontrolu správnosti porovnáním se známými materiály v snímku: zdravá vegetace má vysokou odrazivost v NIR

a nízkou v červené; kalibrační cíl by měl vykazovat hodnoty blízké jeho publikované odrazivosti

***

## Řešení problémů

### Vrstva, kterou jsem očekával, není v rozevíracím seznamu

**Možné příčiny**

* Snímek nebyl nikdy zpracován — existují pouze vrstvy „base“ a „`RAW (Original)`“
* V nastavení projektu není zaškrtnuto přepínací tlačítko pro export produktu
* Produkt se na danou kameru nevztahuje (radiance a odrazivost u RGB

master; jakýkoli index u jednopásmové monochromatické kamery M3M)
* Kalibrace odrazivosti neměla s čím pracovat — chybí pokrytí `.daq` směrem dolů a žádný cíl v rámci snímku, který by prošel kontrolou kvality — proto se snímek vrátil k režimu „Vignette Corrected“ nebo „Sensor Response“

**Co dělat**

1. Zkontrolujte protokol běhu: Chloros

uvádí, kdy nebylo možné vygenerovat požadovaný exportní produkt a proč
2. Zkontrolujte přepínače exportu pro jednotlivé produkty v [Nastavení projektu](../project-settings/project-settings.md)
3. Ověřte, zda složka produktu existuje ve výstupní struktuře projektu
4. Zpracujte znovu s povoleným produktem

### Seznam vrstev vypadá zastarale

Program Chloros

během probíhajícího běhu znovu prohledává složky produktů projektu a opravuje chybějící registrace vrstev na základě toho, co se skutečně nachází na disku, takže vrstva, jejíž export byl dokončen, se obvykle sama objeví v rámci dotazu. Přepnutí pryč od obrázku a zpět vynutí nové vyřešení.

### Hodnoty odrazivosti se zdají být poloviční oproti očekávaným hodnotám

Téměř jistě dělíte soubor LATTICE číslem 65535. Použijte `Chloros:PixelScale` (32768) nebo si přečtěte sloupec **%**, kde je tento koeficient již aplikován.

### Indexová vrstva existuje, ale obrázek je prázdný

Index vyžaduje pásma, která vaše vrstva neobsahuje — například index, který čte třetí kanál, aplikovaný na jedno- nebo dvoukanálový soubor. Přepněte na vícepásmovou vrstvu (odrazivost nebo debayering) nebo vyberte index, který odpovídá filtru kamery.

***

## Další kroky

* [**Otevření snímku na celou obrazovku**](opening-an-image-full-screen.md) — zobrazení kurzoru, histogram a ovládání GSD
* [**Index/LUT Sandbox**](index-lut-sandbox.md) — interaktivní vizualizace indexu a export
* [**Vzorce multispektrálních indexů**](../project-settings/multispectral-index-formulas.md) — referenční příručka k indexům
* [**Dokončení zpracování**](../processing-images-gui/finishing-the-processing.md) — strom výstupních složek, na které tyto vrstvy odkazují
