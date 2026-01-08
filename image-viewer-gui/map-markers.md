# Značky na mapě

Karta Mapa zobrazuje vaše snímky na interaktivní 2D mapě na základě jejich GPS souřadnic. To poskytuje geografický přehled o vaší snímací relaci a pomáhá vizualizovat prostorové pokrytí. Je to také užitečné při prvním importu snímků, abyste mohli rychle odstranit všechny snímky, které nepotřebujete zpracovat.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Přístup k záložce Mapa

1. Otevřete nebo vytvořte projekt v Chloros.
2. Importujte obrázky, které obsahují metadata GPS.
3. Klikněte na záložku **Mapa** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> v levém postranním panelu.
4. Na mapě se zobrazí značky v místě GPS každé snímky.

{% hint style="info" %}
**Vyžaduje GPS**: Na mapě se zobrazí pouze obrázky s GPS souřadnicemi vloženými do metadat EXIF. Ujistěte se, že máte během pořizování snímků ve fotoaparátu zapnutou funkci GPS.
{% endhint %}

***

## Úprava obrázků na kartě Mapa

Karta **Mapa** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> má stejné funkce přidání  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  a odstranění  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  tlačítka jako karta [**Prohlížeč souborů**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Zobrazuje také stejný seznam projektových souborů, ale s jinými záhlavími sloupců:

### Název souboru

* Původní název souboru z fotoaparátu
* Zachovává konvenci pojmenování fotoaparátu (např. IMG\_0001.RAW)

### Zeměpisná šířka

* Zeměpisná šířka snímku

### Zeměpisná délka

* Zeměpisná délka snímku

### Nadmořská výška

* Nadmořská výška snímku

{% hint style="info" %}
Kliknutím na záhlaví sloupců tabulky se také seřadí data řádků.
{% endhint %}

***

## Značky snímků

Každý snímek s GPS daty je na mapě znázorněn značkou:

### Zobrazení značek

* Značky označují přesné GPS souřadnice, kde byl každý obrázek pořízen.
* Při oddálení se mohou značky seskupit.
* Přiblížením si můžete prohlédnout jednotlivé polohy obrázků.

{% hint style="success" %}
SUPER-ZOOM: Když dosáhnete maximální úrovně přiblížení od poskytovatele mapových dlaždic, dlaždice se při dalším přiblížení zvětší, což vám umožní vidět značky, které jsou blízko u sebe.
{% endhint %}

### Náhled při najetí myší

* **Najetím myší** na libovolnou značku zobrazíte náhled daného snímku.
* To umožňuje rychlou vizuální identifikaci, aniž byste museli opustit zobrazení mapy.
* Užitečné pro vyhledání konkrétních snímků v rámci rozsáhlé relace pořizování snímků.

***

## Poskytovatelé mapových dlaždic

{% hint style="success" %}
**Automatický výběr**: Chloros automaticky vybere službu dlaždic, která poskytuje nejlepší úroveň přiblížení pro vaši aktuální polohu na mapě. V případě potřeby můžete ručně přepínat mezi poskytovateli.
{% endhint %}

Karta Mapa podporuje dva poskytovatele dlaždic pro obrázky pozadí mapy:

### Google Maps

* Standardní satelitní a mapové snímky od společnosti Google.
* Nejlepší pro obecné pokrytí celého světa.

### ESRI

* Satelitní a letecké snímky od společnosti ESRI ArcGIS.
* Často poskytuje snímky s vyšším rozlišením v určitých regionech.

***

## Typy mapových dlaždic

Můžete si vybrat typ mapové vrstvy (zleva doprava):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="line">### Terén

Zobrazuje výškové profily a mapové dlaždice s podrobnostmi (silnice atd.)

### Mapa

Zobrazuje standardní (nižší šířka pásma) mapové dlaždice s podrobnostmi (silnice atd.)

### Satelit

Zobrazuje podrobné (vyšší šířka pásma) satelitní mapové dlaždice

### Hybridní

Zobrazuje satelitní mapové dlaždice s přidanými podrobnostmi (silnice atd.)

***

## Navigace po mapě

### Ovládací prvky pro přiblížení

* **Přiblížení/oddálení**: Použijte kolečko myši nebo tlačítka pro přiblížení.
* **Celá obrazovka**: Zobrazí mapu na celé obrazovce.

### Ovládací prvky pro posun

* **Posun**: Kliknutím a tažením se můžete pohybovat po mapě.

***

## Případy použití

### Vizualizace letové dráhy

* Zobrazení oblasti pokrytí snímků pořízených dronem
* Identifikace mezer v pokrytí snímků
* Ověření provedení letové dráhy

### Kontrola pozemního průzkumu

* Zobrazení prostorového rozložení pozemních snímků
* Lokalizace kalibračních cílových snímků vzhledem k oblasti průzkumu
* Plánování dalších míst pro pořízení snímků

### Kontrola kvality

* Rychlá identifikace snímků pořízených na neočekávaných místech.
* Ověření přesnosti GPS v celém datovém souboru.
* Porovnání polohy snímků s terénními poznámkami.

***

## Řešení problémů

### Nezobrazují se značky

**Možné příčiny:**

* Snímky neobsahují metadata GPS.
* Během pořizování snímků bylo v kameře vypnuto GPS.
* Externí software odstranil data EXIF.

**Řešení**: Ověřte, zda je GPS ve vašem fotoaparátu zapnuté, a znovu importujte původní soubory.

### Značky na nesprávném místě

**Možné příčiny:**

* GPS fotoaparátu mělo špatné satelitní připojení.
* GPS se během snímání odchýlilo.

**Řešení**: Jedná se obvykle o problém s časem snímání; zvažte použití PPK/RTK GPS pro přesné aplikace.
