# Úprava nastavení projektu

Před zpracováním snímků je důležité nakonfigurovat nastavení projektu tak, aby odpovídalo vašim požadavkům na pracovní postup. Panel „Nastavení projektu“ <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> umožňuje komplexní ovládání kalibrace, možností zpracování, multispektrálních indexů a formátů exportu.

## Otevření panelu „Nastavení projektu“

1. Otevřete svůj projekt v aplikaci Chloros
2. Klikněte na ikonu **Nastavení projektu** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> v levém postranním panelu
3. Panel „Nastavení projektu“ zobrazí všechny možnosti konfigurace

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>Panel Nastavení projektu — Zobrazení, detekce cílů a zpracování</p></figcaption></figure>

{% hint style="info" %}
**Nastavení se ukládají automaticky** spolu s projektem. Při opětovném otevření projektu se všechna nastavení obnoví.
{% endhint %}

***

## Rychlé nastavení pro běžné pracovní postupy

### Výchozí nastavení (doporučeno pro většinu uživatelů)

Výchozí nastavení funguje dobře pro typické pracovní postupy v programech Survey3 a LATTICE:

* ✅ **Korekce vinětace**: Zapnuto
* ✅ **Kalibrace odrazivosti / vyvážení bílé**: Zapnuto (využívá cíle MAPIR a/nebo data ze světelného senzoru DAQ)
* ✅ **Metoda debayeringu**: Standardní (rychlá, střední kvalita)
* ✅ **Formát exportu**: TIFF (16 bitů) (TIFF (8 bitů) pro projekty, kde jsou všechna data v 8 bitech)
* ✅ **Všechny exportované výstupy**: Zapnuto (LATTICE automaticky exportuje rozložení s odstraněným bayerským vzorem, náhled, radianci a odrazivost)

Stačí importovat snímky a zahájit zpracování s těmito výchozími nastaveními.

***

## Přehled nastavení projektu

Panel Nastavení projektu je rozdělen do níže uvedených sekcí. Dvě další sekce — **Světelný senzor DAQ**a**Zarovnání pole** — se zobrazí automaticky, pokud váš projekt obsahuje příslušné soubory. Kompletní dokumentaci najdete v [Nastavení projektu](../project-settings/project-settings.md).

### Zobrazení

* **Rozlišení miniatur obrázků**: Rozlišení miniatur v mřížce obrázků. Možnosti:**Výchozí (512 px)**,**1024 px**,**2048 px**,**Plné rozlišení**. Pouze pro zobrazení – nemá žádný vliv na zpracování. Vyšší hodnoty vypadají při přiblížení ostřeji, ale načítají se pomaleji.

### Detekce cílů

Určuje, jak Chloros identifikuje kalibrační cíle ve vašich snímcích.

**Klíčová nastavení:*** **Minimální plocha kalibračního vzorku (px)**: Prahová hodnota velikosti pro detekci cílů (výchozí:**25**, rozsah 0–10 000)
* **Minimální shlukování cílů (0–100)**: Prahová hodnota podobnosti pro seskupování oblastí cílů (výchozí:**60**)**Kdy upravit:**

* Zvětšete vzorkovou plochu, pokud dochází k falešným detekcím
* Zmenšete ji, pokud se cíle nedetekují
* Upravte seskupování, pokud se cíle rozdělují na více detekcí

{% hint style="info" %}
Tato nastavení jsou zašedlá, pokud je vypnutá **kalibrace odrazivosti / vyvážení bílé** — při jejím vypnutí se detekce cílů vůbec nespustí.
{% endhint %}

### Zpracování

Hlavní možnosti zpracování snímků a kalibrace.

**Klíčová nastavení:*** **Korekce vinětace**: Kompenzuje ztmavnutí okrajů objektivu ✅ Doporučeno
* **Kalibrace odrazivosti / vyvážení bílé**: Kalibruje snímky pomocí detekovaných cílů (program Survey3) a/nebo dat světelného senzoru DAQ (LATTICE) ✅ Doporučeno
* **Metoda debayeringu**: Algoritmus pro převod formátu RAW na tříkanálový multispektrální formát
* **Minimální interval rekalibrace**: Minimální čas v sekundách mezi použitím kalibračního cíle (výchozí:**0** = použít všechny, rozsah 0–3600)**Nekalibrované náhradní výstupy:**Pokud nelze u snímku provést kalibraci odrazivosti (není k dispozici žádný cíl nebo je kalibrace deaktivována), je exportován jako jeden ze dvou náhradních výstupů —**z této dvojice existuje přesně jeden na jeden běh**, vybraný přepínačem korekce vinětace:

* **Exportovat odezvu snímače**: zapíše `Sensor_Response_Images` — používá se, když je korekce vinětace**vypnutá*** **Export s korekcí vinětace**: zapíše `Vignette_Corrected_Images` — používá se, když je korekce vinětace**zapnutá**Zaškrtávací políčko, které není aktivní, je šedé. Odškrtnutím aktivního políčka se zápis daného souboru zcela zastaví.**Výstupní soubory LATTICE** (zobrazují se u každého projektu; vztahují se na snímky pořízené systémem LATTICE):

* **Exportovat s odstraněným bayesovým filtrem**: lineární obraz po odstranění mozaiky (`Debayered_Images`): odstraněn tmavý offset kalibračního balíčku, následně provedeno odstranění mozaiky, bez plochého pole ani odmíchání — stejný soubor, jaký záznam ukládá na úrovni po odstranění mozaiky — v nativní orientaci snímače. Platí pro RGB a multispektrální moduly.
* **Export náhledu**: náhled na displeji (`Preview_Images`). RGB = vyvážení bílé (DAQ-illuminant, je-li k dispozici, jinak gray-world) + gama; multispektrální = roztažení falešných barev.
* **Export radiance**: spektrální radiance typu float32 (`Radiance_Images`, W/m²/sr/nm). Pouze multispektrální moduly – nevztahuje se na RGB mastery.
* **Export odrazivosti**: odrazivost typu uint16 (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0), pokud snímek pokrývá hodnota `.daq` pro sestupující záření nebo cíl v rámci snímku. Pouze multispektrální moduly.

Všechny čtyři jsou **ve výchozím nastavení zapnuté**— jeden importovaný surový snímek LATTICE se v jediném cyklu zpracování rozdělí do všech povolených a použitelných produktů. Zaškrtávací políčko**Exportovat odrazivost** je zašedlé, pokud je kalibrace odrazivosti vypnutá. Nastavení, která nelze provést kvůli nastavení nadřazeného přepínače, jsou vždy zašedlá a obsahují popisek s názvem přepínače, který je třeba změnit.**Pokročilá nastavení:*** **Časový posun světelného senzoru**: Počet hodin od UTC pro synchronizaci času světelného senzoru (výchozí: 0, rozsah −12 až +12)
* **Použít korekce PPK**: Používá data GPS/expozičních pinů ze souborů `.daq` (výchozí: vypnuto)
* **Expoziční piny 1/2**: Přiřazuje kamery k expozičním pinům pro konfigurace se dvěma kamerami

{% hint style="info" %}
**Úroveň vstupu LATTICE je automatická.** Snímky pořízené systémem LATTICE obsahují úroveň zpracování v metadatech XMP a zpracování vždy vstupuje do zpracovatelského řetězce na úrovni surového snímku — v grafickém uživatelském rozhraní není třeba nic konfigurovat. (Příkaz CLI `--input-level` slouží jako pokročilé přepsání pro snímky se ztracenými metadaty; viz [CLI Reference](../reference/cli-reference.md).)
{% endhint %}

### Metoda odstraňování pixelů

V současné době nabízíme v Chloros 2 metody odstraňování pixelů:

#### Standardní (rychlá, střední kvalita)

Standardní metoda odstraňování pixelů zpracovává rychle, ale vykazuje barevný šum způsobený odstraňováním pixelů, což vede k méně přesným a šumovějším obrázkům.

#### S ohledem na texturu (pomalé, nejvyšší kvalita) \[Pouze Chloros+]

Metoda s ohledem na texturu využívá vysoce kvalitní debayering s ohledem na hrany v kombinaci s modelem odšumování založeným na AI/ML, který odstraní téměř veškerý šum způsobený debayeringem. Model k běhu vyžaduje paměť GPU (VRAM): s **7 GB nebo více VRAM** dokáže zpracovávat více obrázků současně; při méně než 7 GB zpracovává jeden obrázek po druhém (což je znatelně pomalejší). Viz [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**Snímky LATTICE vždy používají standardní demosaic.** Neexistuje žádný model Texture Aware vycvičený pro LATTICE, takže tato možnost není pro snímky LATTICE k dispozici — snímky Survey3 ve stejném projektu ji však stále mohou používat.
{% endhint %}

### Index (multispektrální indexy)

Nastavte, které vegetační indexy se mají vypočítat a exportovat. Rozbalovací nabídka v grafickém uživatelském rozhraní nabízí **27 předdefinovaných vzorců indexů**.**Jak přidat indexy:**

1. Klikněte na tlačítko**„Přidat index“**

2. Vyberte index z rozevíracího menu (NDVI, NDRE, GNDVI atd.)
3. Nastavte parametry vizualizace (barvy LUT, rozsahy hodnot)
4. Podle potřeby přidejte více indexů

**Oblíbené indexy:*** **NDVI**: Celkový zdravotní stav vegetace (nejběžnější)
* **NDRE**: Včasná detekce stresu pomocí RedEdge
* **GNDVI**: Citlivý na koncentraci chlorofylu
* **OSAVI**: Dobře funguje u viditelné půdy
* **EVI**: Oblasti s vysokým indexem listové plochy (LAI)

**Vlastní vzorce:**

* Vytvářejte vlastní vzorce multispektrálních indexů pomocí matematických operací s pásmy napříč všemi kanály snímku
* Ukládejte vlastní vzorce pro opakované použití
* Vlastní vzorce jsou funkcí Chloros+; jejich dostupnost závisí na úrovni vašeho tarifu

Všechny dostupné indexy a vzorce – včetně toho, které názvy jsou dostupné pouze v grafickém uživatelském rozhraní (GUI) a které fungují také v CLI nebo SDK – najdete v článku [Vzory multispektrálních indexů](../project-settings/multispectral-index-formulas.md).

### Export

Ovládá formát výstupního souboru.

**Dostupné formáty**(nastavení:**Formát kalibrovaného snímku**, výchozí**TIFF (16 bitů)**):

* **TIFF (16 bitů)**: Doporučeno pro GIS a vědecké analýzy
* **TIFF (8 bitů)**: 8bitový TIFF
* **TIFF (32 bitů, procenta)**: Hodnoty s plovoucí desetinnou čárkou
* **PNG (8 bitů)**: Bezztrátová komprese pro vizualizaci
* **JPG (8 bitů)**: Nejmenší soubory, ztrátová komprese

Seznam ukazuje, jaké formáty mohou obrázky v projektu obsahovat: Chloros neprovádí zvýšení bitové hloubky. U všech 8bitových obrázků jsou k dispozici pouze formáty TIFF (8bit), PNG (8bit) a JPG (8bit), přičemž výchozím formátem je TIFF (8bit). Ve smíšeném projektu se každý snímek exportuje ve své vlastní hloubce nebo nižší. Radiance zůstává 32bitová s plovoucí desetinnou čárkou.

Výstupy se ukládají do projektové složky, seskupené podle kamery a formátu: `<project>/<camera>/<format>/<Product>_Images/`. Radiance se **vždy** ukládá jako float32 do složky `tiff32` bez ohledu na toto nastavení. Exportované soubory si zachovávají název zdrojového souboru – produkt identifikuje složka. Multispektrální exporty LATTICE mohou také zapisovat masky omezující hodnoty na úrovni jednotlivých pixelů do složky `clamp_masks/` vedle složek s produkty. Úplnou strukturu výstupních souborů najdete v části [Dokončení zpracování](finishing-the-processing.md).

{% hint style="warning" %}
**Čtení hodnot odrazivosti**: hodnota DN, která odpovídá ρ = 1,0, závisí na zdrojové kameře — LATTICE používá 32768 (označeno jako XMP `Chloros:PixelScale`), Survey3 používá 65535. Přečtěte si hodnotu tagu, místo abyste předpokládali konstantní hodnotu. Viz [Formáty výstupních obrázků](../output-image-formats.md).
{% endhint %}

### Světelný senzor DAQ

Tato část uvádí všechny soubory DAQ s dopadajícím světlem (`.daq` / `.csv`) ve vašem projektu, jeden řádek na soubor, s uvedením modelu senzoru, názvu souboru a **korekce** difuzoru platnou pro daný soubor.

* **Přepsání krytu (všechny soubory)**: jediné rozevírací menu pro celý projekt.**Auto** (výchozí nastavení) používá kryt zaznamenaný v každém souboru — pokud není nic zaznamenáno, předpokládá se sluneční svit, ale modely DAQ-U, DAQ-M a DAQ-E se dodávají bez krytu a program Chloros nedokáže rozpoznat, jaký kryt byl nasazen, takže při zpracování se zobrazí varování ohledně předpokládaného krytu: vyberte kryt, který byl nasazen. Výběr horní hranice přepíše všechny soubory kromě souboru DAQ-E-S, jehož difuzor je součástí jeho kalibrace: surové záznamy jsou pomocí něj korigovány a záznamy, které již obsahují horní hranici, jsou znovu referencovány (zaznamenaná korekce je zrušena a je aplikována vybraná horní hranice).
* Řádky zobrazují varování, pokud byl zaznamenaný kryt považován za výchozí hodnotu rozbočovače, nikoli za hodnotu potvrzenou obsluhou, a pokud vybraný kryt nemá profil pro daný model zařízení (přepsání je pro tento soubor odmítnuto).

Záznamy DAQ pořízené na kartě Světelné senzory se automaticky přidávají do otevřeného projektu a importované soubory `.daq` / `.csv` se zde zobrazí, jakmile jsou přidány.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Dolní část nastavení projektu — Index, formát exportu, sekce DAQ světelných senzorů a ovládací prvky šablony/složky projektu</p></figcaption></figure>### Zarovnání pole

Tato sekce se zobrazí **pouze** v případě, že alespoň jeden snímek v projektu obsahuje transformaci pro zarovnání mezi moduly, kterou pole LATTICE vkládají při snímání (`Chloros:Alignment*` XMP). Zobrazuje, kolik snímků obsahuje značky a která kamera slouží jako reference, s těmito ovládacími prvky:

* **Použít zarovnání pole** (výchozí: zapnuto): deformuje každý zpracovaný výstup (odbyzerovaný / náhled / radiance / odrazivost / index) do sdílené referenční geometrie pole. Vypnuto = export v nativní geometrii snímače.
* **Oříznout na společný překryv** (výchozí: zapnuto): ořízne zarovnané exporty na oblast, kterou sdílejí všechny moduly, takže každé pásmo má stejnou stopu. Vypnuto zachovává plnou plochu snímače (černá výplň mimo zdroj).
* **Převzorkování**:**Bilineární (hladké, výchozí)**,**Nejbližší (zachování přesných hodnot)**— bez míchání mezi pixely, pro přísnou radiometrickou analýzu — nebo**Kubické (nejostřejší)**.***

## Ukládání a načítání nastavení

### Uložení šablony projektu

Vytvořte opakovaně použitelné šablony pro konzistentní pracovní postupy:

1. Nakonfigurujte všechna požadovaná nastavení v panelu Nastavení projektu
2. Přejděte do sekce **„Uložit šablonu projektu“** v dolní části
3. Zadejte výstižný název šablony (např. „Survey3N_RGN_Agriculture“)
4. Klikněte na ikonu uložení

**Výhody:**

* Používejte identická nastavení napříč více projekty
* Sdílejte konfigurace s členy týmu
* Zachovejte konzistenci při opakovaných průzkumech

### Načtení šablony do nového projektu

Při vytváření nového projektu:

1. V hlavním menu vyberte **„Nový projekt“**

2. V volitelném výběru šablon vyberte šablonu projektu
3. Všechna nastavení ze šablony se automaticky použijí

### Pracovní adresář

Nastavení **„Pracovní adresář“** určuje, kam se nové projekty ve výchozím nastavení ukládají:

* **Výchozí umístění**: `Documents\MAPIR\Chloros Projects` ve složce Dokumenty systému Windows (obvykle `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`; sleduje složku Dokumenty, pokud ji přesune OneDrive nebo roamingový profil)
* **Změnit umístění**: Klikněte na ikonu úprav a vyberte novou složku
* **Sdíleno s CLI**: `chloros-cli` používá stejné výchozí nastavení projektové složky
* **V systému Windows volte krátké názvy**: pokud nejsou zapnuty dlouhé cesty v systému Windows, bude projektová složka s názvem delším než přibližně 130 znaků před zpracováním odmítnuta — viz [Spuštění zpracování](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer)
* **Kdy změnit**:
  * Síťová jednotka pro týmovou spolupráci
  * Jiná jednotka s větším úložným prostorem
  * Uspořádaná struktura složek podle roku/klienta

***

## Nastavení PPK (Post-Processed Kinematic)

Při použití záznamníků MAPIR DAQ s GPS pro přesnou geolokaci:

### Předpoklady

* Záznamník MAPIR DAQ s modulem GPS (GNSS)
* Soubor protokolu .daq se záznamy o expozičních pinech
* Fotoaparát připojený k expozičním pinům záznamníku DAQ během snímání

### Kroky konfigurace

1. Umístěte soubor protokolu .daq do složky projektu
2. V nastavení projektu zaškrtněte políčko **„Použít korekce PPK“**

3. V případě potřeby nastavte**„Časový posun světelného senzoru“** (výchozí hodnota: 0 pro UTC)
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

Při zpracování snímků z více kamer MAPIR v rámci jednoho projektu:

1. Chloros automaticky detekuje každý model kamery (Survey3 i LATTICE)
2. Každá kamera dostane příslušné profily zpracování a každá kamera má vlastní strom výstupních složek
3. PPK: Ručně přiřaďte každou kameru Survey3 ke správnému expozičnímu pinu
4. Všechny kamery používají stejný formát exportu a indexy

**Příklady**: Dvoukamerová sestava Survey3W RGN + Survey3N OCN, nebo pole LATTICE kombinující hlavní RGB kameru s úzkopásmovými moduly

### Časosběrné snímky nebo snímky z různých dat

Pro opakované snímání stejné oblasti v průběhu času:

1. Vytvořte šablonu se standardními nastaveními
2. Při každém snímání používejte konzistentní uspořádání kalibračních terčů
3. Každý den zpracovávejte jako samostatný projekt
4. Pro srovnatelnost výsledků používejte identická nastavení
5. Exportujte ve stejném formátu pro časovou analýzu

### Rozsáhlé datové sady

Pro projekty s velkým počtem snímků (500+):

* Zvažte rozdělení na menší projekty podle data nebo oblasti
* Pro rychlejší výsledky využijte paralelní zpracování v programu Chloros+
* Zvažte použití rozhraní CLI nebo API pro automatizaci dávkového zpracování
* Upravte minimální interval rekalibrace, abyste zkrátili dobu detekce kalibračních terčů

***

## Ověření nastavení

Před zahájením zpracování zkontrolujte tato klíčová nastavení:

* [ ] Model kamery byl správně rozpoznán v prohlížeči souborů
* [ ] Je zapnuta korekce vinětace
* [ ] Je zapnuta kalibrace odrazivosti
* [ ] Pro Survey3: byl importován a zkontrolován alespoň jeden kalibrační cílový snímek; pro LATTICE: je k dispozici cíl a/nebo záznam `.daq` směřující dolů
* [ ] Přidány požadované multispektrální indexy
* [ ] Formát exportu vhodný pro váš pracovní postup
* [ ] Nastavení PPK nakonfigurováno (pokud používáte soubory .daq s expozičními událostmi)

***

## Další kroky

Jakmile máte nastavení nakonfigurováno:

1. **Označte kalibrační cílové snímky** – viz [Výběr cílových snímků](choosing-target-images.md)
2. **Spusťte zpracování** – viz [Spuštění zpracování](starting-the-processing.md)
3. **Sledujte průběh** – viz [Sledování zpracování](monitoring-the-processing.md)

Podrobné informace o všech dostupných nastaveních najdete v referenční dokumentaci [Nastavení projektu](../project-settings/project-settings.md).
