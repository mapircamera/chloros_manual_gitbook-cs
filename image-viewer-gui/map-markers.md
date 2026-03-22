# Značky na mapě

Na kartě Mapa se vaše snímky zobrazují na interaktivní 2D mapě podle jejich GPS souřadnic. Získáte tak geografický přehled o vaší snímací relaci a můžete si lépe představit prostorové pokrytí. Tato funkce se hodí také při prvním importu snímků, kdy můžete rychle odstranit všechny snímky, které nepotřebujete zpracovávat.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Přístup na kartu Mapa

1. Otevřete nebo vytvořte projekt v Chloros
2. Importujte snímky, které obsahují metadata GPS
3. Klikněte na kartu **Mapa** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> v levém postranním panelu
4. Na mapě se zobrazí značky v GPS poloze každého snímku

{% hint style="info" %}
**Vyžaduje GPS**: Na mapě se zobrazí pouze snímky s GPS souřadnicemi vloženými do metadat EXIF. Ujistěte se, že máte při pořizování snímků ve fotoaparátu zapnutou funkci GPS.
{% endhint %}

***

## Úprava snímků na kartě Mapa

Karta **Mapa**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> má stejné tlačítko pro přidání  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  a odstranění  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  jako karta [**Prohlížeč souborů**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Zobrazuje také stejný seznam projektových souborů, ale s odlišnými záhlavími sloupců:

### Název souboru

* Původní název souboru z fotoaparátu
* Zachovává konvenci pojmenování fotoaparátu (např. IMG\_0001.RAW)

### Šířka

* Šířka snímku

### Délka

* Délka snímku

### Nadmořská výška

* Nadmořská výška snímku

{% hint style="info" %}
Kliknutím na záhlaví sloupců tabulky se data v řádcích také seřadí
{% endhint %}

***

## Značky snímků

Každý snímek s GPS daty je na mapě znázorněn značkou:

### Zobrazení značek

* Značky označují přesné GPS souřadnice místa, kde byl každý snímek pořízen
* Při oddálení se mohou seskupené značky spojit dohromady
* Přiblížením si můžete prohlédnout umístění jednotlivých snímků

{% hint style="success" %}
SUPER-ZOOM: Když dosáhnete maximální úrovně přiblížení od poskytovatele mapových dlaždic, dlaždice se při dalším přiblížení zvětší, což vám umožní vidět značky, které jsou blízko u sebe.
{% endhint %}

### Náhled při najetí myší

* **Najedete-li myší** na libovolnou značku, zobrazí se náhled daného snímku
* To umožňuje rychlou vizuální identifikaci, aniž byste museli opustit zobrazení mapy
* Užitečné pro vyhledání konkrétních snímků v rámci rozsáhlé snímací relace

***

## Poskytovatelé mapových dlaždic

{% hint style="success" %}
**Automatický výběr**: Chloros automaticky vybere službu dlaždic, která poskytuje nejlepší úroveň přiblížení pro vaši aktuální polohu na mapě. V případě potřeby můžete ručně přepínat mezi poskytovateli.
{% endhint %}

Karta Mapa podporuje dva poskytovatele dlaždic pro snímky pozadí mapy:

### Google Maps

* Standardní satelitní a mapové snímky od Google
* Nejlepší pro obecné celosvětové pokrytí

### ESRI

* Satelitní a letecké snímky z ESRI ArcGIS
* V určitých regionech často poskytuje snímky s vyšším rozlišením

***

## Typy mapových dlaždic

Můžete si vybrat typ mapové vrstvy (zleva doprava):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="original">### Terén

Zobrazuje výškové profily a mapové dlaždice s detaily (silnice atd.)

### Mapa

Zobrazuje standardní (nižší šířka pásma) mapové dlaždice s detaily (silnice atd.)

### Satelit

Zobrazuje detailní (vyšší šířka pásma) satelitní mapové dlaždice

### Hybridní

Zobrazuje satelitní mapové dlaždice s přidanými detaily (silnice atd.)

***

## Navigace po mapě

### Ovládací prvky pro přiblížení

* **Přiblížení/oddálení**: Použijte kolečko myši nebo tlačítka pro přiblížení
* **Celá obrazovka**: Zobrazte mapu na celé obrazovce

### Ovládací prvky pro posun

* **Posun**: Klikněte a táhněte pro pohyb po mapě***

## Případy použití

### Vizualizace letové trasy

* Zobrazení oblasti pokrytí při snímkování dronem
* Identifikace mezer v pokrytí snímků
* Ověření provedení letové trasy

### Kontrola pozemního průzkumu

* Zobrazení prostorového rozložení pozemních snímků
* Lokalizace kalibračních cílových snímků vzhledem k oblasti průzkumu
* Plánování dalších míst pro snímkování

### Kontrola kvality

* Rychlá identifikace snímků pořízených na neočekávaných místech
* Ověření přesnosti GPS v celém datovém souboru
* Porovnání polohy snímků s terénními poznámkami

***

## Řešení problémů

### Nezobrazují se značky

**Možné příčiny:**

* Snímky neobsahují metadata GPS
* Během snímání byla na kameře vypnutá funkce GPS
* Data EXIF byla odstraněna externím softwarem

**Řešení**: Ověřte, zda je GPS ve fotoaparátu zapnuté, a znovu importujte původní soubory

### Značky na nesprávném místě

**Možné příčiny:**

* GPS fotoaparátu mělo špatný signál ze satelitu
* Posun GPS během pořizování

**Řešení**: Jedná se obvykle o problém v době pořízení; zvažte použití PPK/RTK GPS pro přesné aplikace
