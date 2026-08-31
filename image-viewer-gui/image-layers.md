# Vrstvy obrázků

**Rozbalovací nabídka vrstev** v pravém horním rohu prohlížeče obrázků umožňuje přepínat mezi všemi verzemi obrázku, který právě prohlížíte – od zdrojového snímku přes jednotlivé zpracované výstupy až po vypočítané indexové obrázky – aniž byste museli prohlížeč opustit.

## Co jsou vrstvy snímků?

„Vrstva“ v programu Chloros je jeden **soubor produktu**přiřazený k jednomu zdrojovému snímku. Importem získáte zdrojové soubory; při zpracování se přidá jedna vrstva pro každý produkt, který byl v daném běhu vytvořen. Exportované soubory si zachovávají název zdrojového souboru – produkt identifikuje**složka** a název vrstvy je označení této složky v systému Chloros.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## Seznam vrstev

### Vždy přítomné

| Vrstva | Co to je |
| --- | --- |
| **JPG**(nebo**PNG**/**TIFF**) | Základní soubor, který byl součástí záznamu. Survey3 importuje soubor `.JPG` vedle každého souboru `.RAW`; Záznamy LATTICE obsahují náhled na obraz PNG nebo TIFF. Označeno podle toho, co bylo skutečně importováno |
| **RAW (originál)** | Zdrojový surový snímek, zbavený bayesovského kódování pro zobrazení bez aplikovaných korekcí. K dispozici od okamžiku importu — nevyžaduje žádné zpracování |

Snímek LATTICE, jehož základním souborem **je** jeho surový snímek, nemá samostatný základní záznam: `RAW (Original)` jej již pokrývá.

### Výstupy zpracování Survey3

| Vrstva | Zapsáno do | Existuje, když |
| --- | --- | --- |
| **RAW (Cíl)** | — | Snímek byl identifikován jako obsahující kalibrační cíl |
| **RAW (odrazivost)** | `Reflectance_Calibrated_Images/` | Kalibrace odrazivosti na tomto snímku proběhla úspěšně |
| **Opraveno proti vinětaci**| `Vignette_Corrected_Images/` | Snímek nebylo možné kalibrovat podle odrazivosti**a** byla zapnuta *korekce vinětace* |
| **Odezva snímače**| `Sensor_Response_Images/` | Snímek nebylo možné kalibrovat podle odrazivosti**a** *korekce vinětace* byla vypnutá |
| **Vyvážení bílé** | `White_Balanced_Images/` | Byl zapsán produkt s vyvážením bílé |

{% hint style="info" %}
**Korekce vinětace a odezva snímače jsou alternativy, nikdy se nepoužívají současně.** Pro každý model kamery existuje přesně jeden nekalibrovaný záložní produkt na jeden běh a přepínač *Korekce vinětace* určuje, který z nich se použije. Viz [Nastavení projektu](../project-settings/project-settings.md).
{% endhint %}

### Úrovně LATTICE

LATTICE zachycuje rozložení do těchto úrovní v jediném kroku zpracování. Které z nich existují, závisí na přepínačích exportu pro jednotlivé produkty v Nastaveních projektu a na tom, co se vztahuje na daný fotoaparát.

| Vrstva | Zapsáno do | Vztahuje se na |
| --- | --- | --- |
| **RAW (Debayered)** | `Debayered_Images/` | RGB a multispektrální |
| **RAW (náhled)** | `Preview_Images/` | Multispektrální (roztažení falešných barev) |
| **Vyvážení bílé** | `Preview_Images/` | Hlavní kamery RGB — náhled RGB je zaregistrován pod tímto názvem, aby se shodoval se stejnojmennou vrstvou Survey3 |
| **RAW (radiance)** | `Radiance_Images/` | Pouze multispektrální |
| **RAW (odrazivost)** | `Reflectance_Calibrated_Images/` | Pouze multispektrální, a to pouze v případě, že snímek pokrývá odpovídající záznam `.daq` směřující dolů nebo cíl v rámci snímku, který prošel kontrolou kvality |

Hlavní kamery RGB nemají radiometrii pro jednotlivá pásma, proto jsou u nich zářivost a odrazivost vynechány jako **neplatné** — protokol to uvádí, místo aby došlo k tichému selhání.

### Vrstvy indexu, LUT a sandboxu

| Vzor vrstvy | Příklad | Odkud pochází |
| --- | --- | --- |
| **RAW (`<INDEX>` Index)** | `RAW (NDVI Index)` | Jeden pro každý index nakonfigurovaný v nastavení projektu, vypočítán během zpracování |
| **`<INDEX>` LUT** | `NDVI LUT` | Barevně mapovaná verze indexu |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Jedna na každý exportní cyklus [Index/LUT Sandbox](index-lut-sandbox.md) |

Pokud je stejný název indexu nakonfigurován vícekrát s odlišnými nastaveními, druhý a další dostanou v názvu číslo (`RAW (NDVI2 Index)`), aby byly vrstvy rozlišitelné.

***

## Použití výběru vrstev

1. Otevřete obrázek na celou obrazovku kliknutím na miniaturu v mřížce
2. Klikněte na **rozevírací nabídku vrstev** v pravém horním rohu prohlížeče
3. Vyberte vrstvu — obrázek se okamžitě aktualizuje

V rozevíracím seznamu jsou na prvním místě uvedeny formáty **JPG, RAW (originál), RAW (cíl), RAW (odrazivost)** v tomto pořadí a za nimi jsou uvedeny všechny ostatní položky v pořadí, v jakém byly produkty zaregistrovány.

### Předvolba vrstvy při procházení

Stisknutím kláves **←**/**→** přejdete na další obrázek a program se pokusí zachovat stejnou vrstvu:

1. **Nejprve přesná shoda** — pokud má další obrázek vrstvu se stejným názvem, zobrazí se právě ta. Díky tomu zůstanete na vrstvě `RAW (NDVI Index)` při procházení celé sady
2. **Poté shoda podle typu** — indexová vrstva hledá jakoukoli indexovou vrstvu, LUT jakoukoli LUT, odrazivost jakoukoli odrazivost, cíl jakoukoli cílovou vrstvu, originál jakoukoli originální vrstvu, základ jakoukoli základní vrstvu
3. **Poté, pouze u exportních vrstev** — název se zachová, i když seznam vrstev ještě není aktuální, protože soubor již na disku existuje. Díky tomu můžete prohlížet výstupy, zatímco je proces stále zapisuje
4. **V ostatních případech** — první dostupná vrstva, kterou je obvykle základní obrázek

Soubory sidecar `.daq` a `.csv` v projektu jsou při navigaci pomocí šipek přeskočeny, takže při procházení obrázků nikdy nedojde k záznamu ze světelného senzoru.

Funkce přiblížení a posunu se přenášejí i mezi obrazy, což usnadňuje porovnání stejné polohy v poli před a po.

***

## Porozumění hodnotám pixelů podle vrstvy

Panel [Hodnoty kurzoru](opening-an-image-full-screen.md#cursor-values) zobrazuje skutečnou hodnotu pro každý kanál pod kurzorem v jednotkách, ve kterých je daná vrstva uložena. Jeho sloupce se mění v závislosti na vrstvě:

| Vrstva | Zobrazená jednotka | Poznámky |
| --- | --- | --- |
| Základní (JPG / PNG / náhled TIFF) | DN, 0–255 | Zobrazené hodnoty, v programu RGB korigované gama. Pouze pro vizuální kontrolu |
| RAW (originál) | DN | Surová digitální data ze snímače. Osa histogramu udává hloubku: 255 (8 bitů), 4095 (12 bitů) nebo 65535 (16 bitů) |
| RAW (po odstranění bayesovského vzoru) | DN | Lineární, bez roztažení zobrazení |
| RAW (náhled) / Vyvážení bílé | DN | Výstup pro zobrazení — roztažený nebo s korekcí gama. Není určeno k měření |
| RAW (zářivost) | **W/m²/sr/nm** | Fyzikální zářivost typu Float32. Bez sloupce DN |
| RAW (odrazivost) | DN **a %** | Procento vypočítané podle vlastní stupnice daného souboru — viz níže |
| Exporty indexu / LUT / sandbox | Hodnota indexu nebo složky RGB | Jednokanálový indexový soubor uvádí hodnotu indexu; soubor LUT s barevným mapováním uvádí složky Red/Green/Blue |

### Odrazivost: měřítko je uvedeno pro každý soubor zvlášť

{% hint style="warning" %}
**„Vydělit 65 535“ je správné pouze pro Survey3.** Odrazivost LATTICE je uložena v jiném měřítku a smíchání těchto dvou dělitelů je nejčastějším způsobem, jak získat hodnoty odrazivosti, které jsou přesně poloviční oproti tomu, jaké by měly být.
{% endhint %}

| Zdroj | DN odpovídající odrazivosti 1,0 | Identifikováno podle |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | XMP tag `Chloros:PixelScale=32768`, který je vložen do každého exportu odrazivosti LATTICE. Díky dvojnásobné rezervě lze hodnoty ρ nad 1,0 zobrazit, místo aby byly oříznuty |
| **Survey3**|**65535** | Bez značky měřítka XMP Chloros — kalibrace Survey3 zapíše ρ × dtype-max a ořízne na hodnotu 1,0 |

Pro GIS a skriptování: načtěte `Chloros:PixelScale` ze souboru a vydělte tím. Pokud značka chybí, soubor má měřítko Survey3 (65535). Prohlížeč, indexová/LUT sandbox a export indexu všechny určují měřítko stejným způsobem, takže číslo, které se zobrazí u kurzoru, je číslo použité při výpočtech indexu.

K tomuto měřítku se navíc přidává formátově specifické ukládání:

* **TIFF (32bitový, procenta)** ukládá DN / 65535 jako číslo s plovoucí desetinnou čárkou
* **PNG (8 bitů)**a**JPG (8 bitů)** ukládají hodnotu DN × 255 / 65 535
* **8bitový export TIFF ze záznamu s 8bitovým zdrojem** je oříznut na rozsah 0–255 namísto přepočítání a záměrně neobsahuje žádný tag měřítka. Panel u těchto souborů zobrazuje pouze hodnotu DN, bez sloupce s procenty

### Rozsahy indexových hodnot

| Rodina indexů | Typický rozsah | Hodnota |
| --- | --- | --- |
| Normalizovaný rozdíl (NDVI, GNDVI, NDRE, ENDVI…) | −1 až +1 | Zdravá vegetace obvykle 0,4–0,9; holá půda kolem 0; voda záporná |
| Úprava pro půdu (SAVI, OSAVI, MSAVI2…) | přibližně −1 až +1,5 | Hodnota podobná jako u NDVI s potlačeným půdním pozadím |
| Poměr (GRVI, GCI, MSR, CIRE…) | neomezeně vysoká hodnota | Poměry rostou bez omezení, jak se hodnota jmenovatele blíží nule |
| EVI / LAI | 0 až ~1, 0 až ~3,5 | Mraky a jiné nasycené pixely posouvají obě hodnoty mimo rozsah — nejprve je zamaskujte |

Přesný vzorec pro každou předvolbu najdete v [Vzorce multispektrálních indexů](../project-settings/multispectral-index-formulas.md).

***

## Běžné pracovní postupy

### Porovnání před a po

1. Vyberte **RAW (Originál)** a všimněte si vinětace a nekalibrovaných hodnot
2. Přepněte na **RAW (Reflexe)**

3. Porovnejte — vinětace odstraněna, hodnoty kalibrovány. Přiblížení a posun zůstávají zachovány, takže se díváte na stejnou oblast terénu

### Prohlédněte si jeden index v celé sadě

1. Otevřete první zpracovaný snímek a vyberte indexovou vrstvu
2. Opakovaně stiskněte **→** — indexová vrstva vás bude následovat od snímku k snímku
3. Sledujte přitom histogram v postranním panelu: snímek, u kterého dojde k výraznému skoku v rozložení, stojí za bližší prozkoumání

### Ověřte kalibrační cíle

1. Na cílovém snímku vyberte **RAW (Target)**

2. Ujistěte se, že je cíl jasně viditelný a detekovaný
3. Přejděte na další cílový snímek — vrstva cílů vás bude následovat

### Zkontrolujte přesnost hodnot odrazivosti

1. Vyberte **RAW (Reflectance)**

2. Přečtěte si sloupec**%** v panelu Hodnoty kurzoru — pro daný soubor je již správně škálován
3. Proveďte kontrolu správnosti porovnáním se známými materiály v snímku: zdravá vegetace má vysokou hodnotu NIR a nízkou hodnotu červené; kalibrační cíl by měl vykazovat hodnoty blízké jeho zveřejněné odrazivosti

***

## Řešení problémů

### Vrstva, kterou jsem očekával, není v rozevíracím seznamu

**Možné příčiny**

* Snímek nebyl nikdy zpracován — existují pouze základní vrstva a `RAW (Original)`
* V nastavení projektu není zaškrtnuto přepínací tlačítko pro export produktu
* Produkt se na danou kameru nevztahuje (radiance a odrazivost na masteru RGB; jakýkoli index na jednopásmové monochromatické kameře M3M)
* Kalibrace odrazivosti neměla s čím pracovat – chybělo pokrytí sestupným zářením `.daq` a v snímku nebyl žádný cíl, který by prošel kontrolou kvality – proto se snímek vrátil k režimu „Vignette Corrected“ nebo „Sensor Response“

**Co dělat**

1. Zkontrolujte protokol běhu: Chloros uvádí, kdy nebylo možné vygenerovat požadovaný exportní produkt a proč
2. Zkontrolujte přepínače exportu pro jednotlivé produkty v [Nastavení projektu](../project-settings/project-settings.md)
3. Ověřte, zda složka produktu existuje ve výstupní struktuře projektu
4. Zpracujte znovu s povoleným produktem

### Seznam vrstev vypadá zastarale

Chloros během probíhajícího běhu znovu prohledává složky produktů projektu a opravuje chybějící registrace vrstev na základě toho, co se skutečně nachází na disku, takže vrstva, jejíž export byl dokončen, se obvykle sama objeví v rámci dotazu. Přepnutí pryč od obrázku a zpět vynutí nové vyhodnocení.

### Hodnoty odrazivosti vypadají jako polovina toho, co by měly být

Téměř jistě dělíte soubor LATTICE číslem 65535. Použijte `Chloros:PixelScale` (32768) nebo si přečtěte sloupec **%**, kde je tento faktor již aplikován.

### Indexová vrstva existuje, ale obrázek je prázdný

Index vyžaduje pásma, která vaše vrstva neobsahuje — například index, který čte třetí kanál a je aplikován na jedno- nebo dvoukanálový soubor. Přepněte na vícepásmovou vrstvu (odrazivost nebo debayered) nebo vyberte index, který odpovídá filtru kamery.

***

## Další kroky

* [**Otevření snímku na celou obrazovku**](opening-an-image-full-screen.md) — odečet kurzoru, histogram a ovládání GSD
* [**Index/LUT Sandbox**](index-lut-sandbox.md) — interaktivní vizualizace indexu a export
* [**Vzorce multispektrálních indexů**](../project-settings/multispectral-index-formulas.md) — referenční hodnoty indexů
* [**Dokončení zpracování**](../processing-images-gui/finishing-the-processing.md) — strom výstupních složek, na které tyto vrstvy odkazují
