# Úprava nastavení projektu

Před zpracováním snímků je důležité nakonfigurovat nastavení projektu tak, aby odpovídalo vašim požadavkům na pracovní postup. Panel „Nastavení projektu“ (<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">) umožňuje komplexní ovládání kalibrace, možností zpracování, multispektrálních indexů a formátů exportu.

## Otevření panelu Nastavení projektu

1. Otevřete svůj projekt v programu Chloros
2. Klikněte na ikonu **Nastavení projektu** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> v levém postranním panelu
3. Panel Nastavení projektu zobrazí všechny možnosti konfigurace

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>Panel Nastavení projektu — Zobrazení, detekce cílů a zpracování</p></figcaption></figure>{% hint style="info" %}
**Nastavení se ukládají automaticky** spolu s projektem. Při opětovném otevření projektu se všechna nastavení obnoví.
{% endhint %}

***

## Rychlé nastavení pro běžné pracovní postupy

### Výchozí nastavení (doporučeno pro většinu uživatelů)

Výchozí nastavení funguje dobře pro typické pracovní postupy s Survey3 a LATTICE:

* ✅ **Korekce vinětace**: Zapnuto
* ✅ **Kalibrace odrazivosti / vyvážení bílé**: Zapnuto (využívá terče MAPIR a/nebo data ze světelného senzoru DAQ)
* ✅ **Metoda debayeringu**: Standardní (rychlá, střední kvalita)
* ✅ **Formát exportu**: TIFF (16bitový)
* ✅ **Všechny exportované výstupy**: Zapnuto (LATTICE automaticky exportuje rozložení do formátů debayered, náhled, radiance a odrazivost)

Stačí importovat snímky a zahájit zpracování s těmito výchozími nastaveními.

***

## Přehled nastavení projektu

Panel Nastavení projektu je rozdělen do níže uvedených sekcí. Dvě další sekce — **Světelný senzor DAQ**a**Zarovnání pole** — se zobrazí automaticky, pokud váš projekt obsahuje příslušné soubory. Kompletní dokumentaci najdete v [Nastavení projektu](../project-settings/project-settings.md).

### Zobrazení

* **Rozlišení miniatur obrázků**: Rozlišení miniatur v mřížce obrázků. Možnosti:**Výchozí (512 px)**,**1024 px**,**2048 px**,**Plné rozlišení**. Pouze pro zobrazení — nikdy neovlivňuje zpracování. Vyšší hodnoty vypadají při přiblížení ostřeji, ale načítají se pomaleji.

### Detekce cílů

Ovládá způsob, jakým Chloros identifikuje kalibrační cíle ve vašich obrázcích.

**Klíčová nastavení:*** **Minimální plocha kalibračního vzorku (px)**: Prahová hodnota velikosti pro detekci cílů (výchozí:**25**, rozsah 0–10 000)
* **Minimální shlukování cílů (0–100)**: Prahová hodnota podobnosti pro seskupování oblastí cílů (výchozí:**60**)**Kdy upravit:**

* Zvětšete vzorkovou plochu, pokud dochází k falešným detekcím
* Zmenšete ji, pokud nejsou cíle detekovány
* Upravte seskupování, pokud jsou cíle rozděleny do více detekcí

{% hint style="info" %}
Tato nastavení jsou zašedlá, pokud je vypnutá **kalibrace odrazivosti / vyvážení bílé** — při jejím vypnutí se detekce cílů vůbec nespustí.
{% endhint %}

### Zpracování

Hlavní možnosti zpracování snímků a kalibrace.

**Klíčová nastavení:*** **Korekce vinětace**: Kompenzuje ztmavnutí okrajů objektivu ✅ Doporučeno
* **Kalibrace odrazivosti / vyvážení bílé**: Kalibruje snímky pomocí detekovaných cílů (Survey3) a/nebo dat světelného senzoru DAQ (LATTICE) ✅ Doporučeno
* **Metoda debayeringu**: Algoritmus pro převod formátu RAW na tříkanálový multispektrální formát
* **Minimální interval rekalibrace**: Minimální čas v sekundách mezi použitím kalibračního cíle (výchozí:**0** = použít všechny, rozsah 0–3600)**Nekalibrované náhradní výstupy:**Pokud nelze snímek kalibrovat podle odrazivosti (není k dispozici žádný cíl nebo je kalibrace deaktivována), je exportován jako jeden ze dvou náhradních výstupů —**z této dvojice existuje přesně jeden na jeden běh**, vybraný přepínačem korekce vinětace:

* **Exportovat odezvu snímače**: zapíše `Sensor_Response_Images` — používá se, když je korekce vinětace**vypnutá*** **Export s korekcí vinětace**: zapíše soubor `Vignette_Corrected_Images` — používá se, když je korekce vinětace**zapnutá**Zaškrtávací políčko, které není aktivní, je šedé. Odškrtnutím aktivního políčka se zápis daného souboru zcela zastaví.**Produkty exportu LATTICE** (zobrazují se u každého projektu; vztahují se na snímky pořízené systémem LATTICE):

* **Exportovat s odstraněním bayeru**: lineární obraz s odstraněným bayerem (`Debayered_Images`). Platí pro RGB a multispektrální moduly.
* **Export náhledu**: náhled na displeji (`Preview_Images`). RGB = vyvážení bílé (DAQ-zdroj světla, je-li k dispozici, jinak šedý svět) + gama; multispektrální = roztažení falešných barev.
* **Export radiance**: spektrální radiance typu float32 (`Radiance_Images`, W/m²/sr/nm). Pouze multispektrální moduly — nevztahuje se na mastery RGB.
* ****Export odrazivosti**: odrazivost typu uint16 (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0), pokud snímek pokrývá údaj o sestupném záření `.daq` nebo cíl v rámci snímku. Pouze multispektrální moduly.

Všechny čtyři jsou **ve výchozím nastavení zapnuté**— jeden importovaný surový snímek LATTICE se v jediném cyklu zpracování rozdělí do všech povolených a příslušných produktů. Zaškrtávací políčko**Exportovat odrazivost** je šedé, pokud je kalibrace odrazivosti vypnutá. Nastavení, která nelze použít kvůli nastavení nadřazeného přepínače, jsou vždy šedá a obsahují popisek s názvem přepínače, který je třeba změnit.**Pokročilá nastavení:*** **Časový posun světelného senzoru**: Počet hodin od UTC pro synchronizaci času světelného senzoru (výchozí: 0, rozsah −12 až +12)
* **Použít korekce PPK**: Používá data GPS/expozičních pinů ze souborů `.daq` (výchozí: vypnuto)
* **Expoziční piny 1/2**: Přiřazuje kamery k expozičním pinům pro konfigurace se dvěma kamerami

{% hint style="info" %}
**Úroveň vstupu LATTICE je automatická.** Snímky pořízené systémem LATTICE nesou svou úroveň zpracování v metadatech XMP a zpracování vždy vstupuje do zpracovatelského řetězce na úrovni surového snímku — v grafickém uživatelském rozhraní není třeba nic konfigurovat. (Příznak CLI `--input-level` slouží jako možnost přepsání pro pokročilé uživatele v případě snímků se ztracenými metadaty; viz [CLI Referenční příručka](../reference/cli-reference.md).)
{% endhint %}

### Metoda odstraňování pixelů

V současné době nabízíme v Chloros 2 metody odstraňování pixelů:

#### Standardní (rychlá, střední kvalita)

Standardní metoda odstraňování pixelů zpracovává data rychle, ale vykazuje barevný šum, což vede k méně přesným a šumivějším snímkům.

#### S ohledem na texturu (pomalé, nejvyšší kvalita) \[Pouze Chloros+]

Metoda „S ohledem na texturu“ využívá vysoce kvalitní debayering s ohledem na hrany v kombinaci s modelem odšumování založeným na AI/ML, který odstraňuje téměř veškerý šum způsobený debayeringem. K běhu modelu je zapotřebí paměť GPU (VRAM): s **7 GB nebo více VRAM** dokáže zpracovávat více obrázků současně; při méně než 7 GB zpracovává jeden obrázek po druhém (což je znatelně pomalejší). Viz [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**Snímky LATTICE vždy používají standardní demosaic.** Neexistuje žádný model Texture Aware vycvičený pro LATTICE, takže tato možnost není pro snímky LATTICE nabízena — snímky Survey3 ve stejném projektu ji však stále mohou používat.
{% endhint %}

### Index (multispektrální indexy)

Nastavte, které vegetační indexy se mají vypočítat a exportovat. Rozbalovací nabídka v grafickém uživatelském rozhraní nabízí **27 předdefinovaných vzorců indexů**.**Jak přidat indexy:**

1. Klikněte na tlačítko**„Přidat index“**

2. Vyberte index z rozevíracího menu (NDVI, NDRE, GNDVI atd.)
3. Nakonfigurujte nastavení vizualizace (barvy LUT, rozsahy hodnot)
4. Podle potřeby přidejte více indexů

**Oblíbené indexy:*** **NDVI**: Celkový zdravotní stav vegetace (nejběžnější)
* **NDRE**: Včasná detekce stresu s RedEdge
* **GNDVI**: Citlivý na koncentraci chlorofylu
* **OSAVI**: Dobře funguje u viditelné půdy
* **EVI**: Oblasti s vysokým indexem listové plochy (LAI)**Vlastní vzorce:**

* Vytvářejte vlastní vzorce multispektrálních indexů pomocí matematických operací s pásmy napříč všemi kanály snímku
* Ukládejte vlastní vzorce pro opakované použití
* Vlastní vzorce jsou funkcí Chloros+; dostupnost závisí na úrovni vašeho tarifu

Všechny dostupné indexy a vzorce – včetně toho, které názvy jsou dostupné pouze v grafickém uživatelském rozhraní a které fungují také v CLI/SDK – najdete v [Vzorce multispektrálních indexů](../project-settings/multispectral-index-formulas.md).

### Export

Ovládá formát výstupního souboru.

**Dostupné formáty**(nastavení:**Formát kalibrovaného snímku**, výchozí**TIFF (16bitový)**):

* **TIFF (16bitový)**: Doporučeno pro GIS a vědecké analýzy
* **TIFF (32bitový, procenta)**: Hodnoty s plovoucí desetinnou čárkou
* **PNG (8bitový)**: Bezztrátová komprese pro vizualizaci
* **JPG (8bitový)**: Nejmenší soubory, ztrátová komprese

Výstupy se ukládají do projektové složky, seskupené podle kamery a formátu: `<project>/<camera>/<format>/<Product>_Images/`. Radiance se **vždy** ukládá jako float32 do složky `tiff32` bez ohledu na toto nastavení. Exportované soubory si zachovávají název zdrojového souboru — produkt identifikuje složka. Kompletní výstupní strom najdete v části [Dokončení zpracování](finishing-the-processing.md).

{% hint style="warning" %}
**Čtení hodnot odrazivosti**: DN, při kterém platí ρ = 1,0, závisí na zdrojové kameře — LATTICE používá hodnotu 32768 (označenou v XMP jako `Chloros:PixelScale`), Survey3 používá hodnotu 65535. Přečtěte si tento tag, místo abyste předpokládali konstantní hodnotu. Viz [Formáty výstupních obrázků](../output-image-formats.md).
{% endhint %}

### Světelný senzor DAQ

Tato část uvádí všechny soubory DAQ pro sestupující sluneční záření (`.daq` / `.csv`) ve vašem projektu, jeden řádek na soubor, a zobrazuje model senzoru, název souboru a korekci **krytu** difuzoru platnou pro daný soubor.

* **Přepsání limitu (všechny soubory)**: jediné rozevírací menu pro celý projekt. Volba**Auto** (výchozí) používá limit zaznamenaný v každém souboru — pokud není nic zaznamenáno, předpokládá se sluneční svit, protože všechny soubory DAQ typu MAPIR jsou dodávány s korektorem slunečního svitu. Výběr limitu přepíše všechny soubory: surové záznamy jsou jím korigovány a záznamy, které již obsahují limit, jsou přereferencovány (zaznamenaná korekce je zrušena a je aplikován vybraný limit).
* Řádky upozorňují, pokud byla zaznamenaná mezní hodnota předpokládanou výchozí hodnotou rozbočovače, nikoli operátorem potvrzenou, a pokud vybraná mezní hodnota nemá profil pro daný model zařízení (přepsání je pro daný soubor odmítnuto).

Záznamy DAQ pořízené na kartě Světelné senzory se automaticky přidávají do otevřeného projektu a importované soubory `.daq` / `.csv` se zde zobrazí ihned po jejich přidání.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Dolní část nastavení projektu — Index, formát exportu, sekce DAQ světelných senzorů a ovládací prvky šablony/složky projektu</p></figcaption></figure>### Zarovnání pole

Tato sekce se zobrazí **pouze** v případě, že alespoň jeden snímek v projektu obsahuje transformaci pro zarovnání mezi moduly, kterou pole LATTICE vkládají při snímání (`Chloros:Alignment*` XMP). Ukazuje, kolik snímků obsahuje tyto značky a která kamera slouží jako reference, a nabízí následující ovládací prvky:

* **Použít zarovnání pole** (výchozí: zapnuto): deformuje každý zpracovaný výstup (odstranění matice / náhled / radiance / odrazivost / index) do společné referenční geometrie pole. Vypnuto = export v nativní geometrii snímače.
* **Oříznout na společný překryv** (výchozí: zapnuto): ořízne zarovnané exporty na oblast, kterou sdílejí všechny moduly, takže každé pásmo má stejnou stopu. Vypnuto zachová plnou plochu snímače (černá výplň mimo zdroj).
* **Převzorkování**:**Bilineární (hladké, výchozí)**,**Nejbližší (zachování přesných hodnot)**— bez míchání mezi pixely, pro přísnou radiometrickou analýzu — nebo**Kubické (nejostřejší)**.***

## Ukládání a načítání nastavení

### Uložení šablony projektu

Vytvořte opakovaně použitelné šablony pro konzistentní pracovní postupy:

1. Nakonfigurujte všechna požadovaná nastavení v panelu Nastavení projektu
2. Přejděte do sekce **„Uložit šablonu projektu“** v dolní části
3. Zadejte výstižný název šablony (např. „Survey3N\_RGN\_Agriculture“)
4. Klikněte na ikonu uložení

**Výhody:**

* Použijte identická nastavení napříč více projekty
* Sdílejte konfigurace s členy týmu
* Zachovejte konzistenci při opakovaných průzkumech

### Načtení šablony do nového projektu

Při vytváření nového projektu:

1. V hlavním menu vyberte **„Nový projekt“**

2. V volitelném výběru šablon vyberte šablonu projektu
3. Všechna nastavení ze šablony se automaticky použijí

### Pracovní adresář

Nastavení **„Pracovní adresář“** určuje, kam se nové projekty ve výchozím nastavení ukládají:

* **Výchozí umístění**: `C:\Users\[Username]\Chloros Projects`
* **Změna umístění**: Klikněte na ikonu úpravy a vyberte novou složku
* **Sdíleno s CLI**: `chloros-cli` používá stejné výchozí nastavení projektové složky
* **Kdy změnit**:
  * Síťová jednotka pro týmovou spolupráci
  * Jiná jednotka s větším úložným prostorem
  * Uspořádaná struktura složek podle roku/klienta

***

## Nastavení PPK (Post-Processed Kinematic)

Při použití datových záznamníků MAPIR s GPS pro přesnou geolokaci:

### Předpoklady

* DAQ MAPIR s modulem GPS (GNSS)
* Soubor protokolu .daq se záznamy o expozičních pinech
* Fotoaparát připojený k expozičním pinům DAQ během snímání

### Kroky konfigurace

1. Umístěte logový soubor .daq do složky projektu
2. V nastavení projektu zaškrtněte políčko **„Použít PPK korekce“**

3. V případě potřeby nastavte**„Časové posunutí světelného senzoru“** (výchozí hodnota: 0 pro UTC)
4. Přiřaďte kamery k expozičním pinům:
   * **Jedna kamera**: Automaticky přiřazena k pinu 1
   * **Dvě kamery**: Ručně přiřaďte každou kameru ke správnému pinu**Přiřazení expozičních pinů:*** **Expoziční pin 1**: Vyberte model kamery z rozevíracího seznamu
* **Expoziční pin 2**: Vyberte druhou kameru nebo možnost „Nepoužívat“
* Stejnou kameru nelze přiřadit k oběma pinům

{% hint style="warning" %}
**Důležité**: Piny expozice musí být správně přiřazeny k příslušným kamerám. Nesprávné přiřazení povede k nesprávným geolokačním údajům.
{% endhint %}

***

## Pokročilé scénáře

### Projekty s více kamerami

Při zpracování snímků z více kamer MAPIR v jednom projektu:

1. Chloros automaticky rozpozná každý model kamery (Survey3 i LATTICE)
2. Každá kamera obdrží příslušné profily zpracování a každá kamera má vlastní strom výstupních složek
3. PPK: Ručně přiřaďte každou kameru Survey3 ke správnému expozici
4. Všechny kamery používají stejný formát exportu a indexy

**Příklady**: Survey3W RGN + Survey3N OCN sestava se dvěma kamerami, nebo pole LATTICE kombinující hlavní kameru RGB s úzkopásmovými moduly

### Časosběrné nebo vícedátové průzkumy

Pro opakované průzkumy stejné oblasti v průběhu času:

1. Vytvořte šablonu se standardními nastaveními
2. Při každém měření používejte stejné uspořádání kalibračních terčů
3. Každý den zpracovávejte jako samostatný projekt
4. Pro srovnatelnost výsledků používejte identická nastavení
5. Exportujte ve stejném formátu pro časovou analýzu

### Velké datové soubory

Pro projekty s velkým počtem snímků (500+):

* Zvažte rozdělení na menší projekty podle data nebo oblasti
* Pro rychlejší výsledky použijte paralelní zpracování Chloros+
* Pro automatizaci dávkového zpracování zvažte CLI nebo API
* Upravte minimální interval rekalibrace, abyste zkrátili dobu detekce cílů

***

## Ověření nastavení

Před zahájením zpracování zkontrolujte tato klíčová nastavení:

* [ ] Model kamery je správně rozpoznán v prohlížeči souborů
* [ ] Korekce vinětace je zapnutá
* [ ] Kalibrace odrazivosti je zapnutá
* [ ] Pro Survey3: importován a zkontrolován alespoň jeden kalibrační cílový snímek; pro LATTICE: k dispozici je cíl a/nebo záznam směřující dolů (downwelling) z `.daq`
* [ ] Přidány požadované multispektrální indexy
* [ ] Formát exportu vhodný pro váš pracovní postup
* [ ] Nastavení PPK nakonfigurováno (pokud používáte soubory .daq s expozičními událostmi)

***

## Další kroky

Jakmile máte nastavení nakonfigurováno:

1. **Označte kalibrační cílové snímky** – viz [Výběr cílových snímků](choosing-target-images.md)
2. **Spusťte zpracování** – viz [Spuštění zpracování](starting-the-processing.md)
3. **Sledujte průběh** – viz [Sledování zpracování](monitoring-the-processing.md)

Úplné podrobnosti o všech dostupných nastaveních najdete v referenční dokumentaci [Nastavení projektu](../project-settings/project-settings.md).
