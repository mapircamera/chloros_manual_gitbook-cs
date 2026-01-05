# Značky na mapě

Záložka Mapa zobrazuje vaše snímky na interaktivní 2D mapě na základě jejich GPS souřadnic. Poskytuje geografický přehled vaší snímací relace a pomáhá vizualizovat prostorové pokrytí. Je také užitečná při prvním importu snímků, kdy můžete rychle odstranit všechny snímky, které nepotřebujete zpracovat.

## Přístup k záložce Mapa

1. Otevřete nebo vytvořte projekt v Chloros.
2. Importujte obrázky, které obsahují metadata GPS.
3. Klikněte na záložku **Mapa** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> v levém postranním panelu.
4. Na mapě se zobrazí značky v místě GPS každého snímku.

{% hint style=&quot;info&quot; %}
**Vyžaduje GPS**: Na mapě se zobrazí pouze obrázky s GPS souřadnicemi vloženými do metadat EXIF. Ujistěte se, že je během pořizování snímků ve vašem fotoaparátu zapnutá funkce GPS.
{% endhint %}

***

## Úprava obrázků na kartě Mapa

Karta **Mapa**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> má stejné funkce přidání  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  a odstranění  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  tlačítka jako karta [**Prohlížeč souborů**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Zobrazuje také stejný seznam projektových souborů, ale s jinými záhlavími sloupců:

### Název souboru

* Původní název souboru z fotoaparátu
* Zachovává konvenci pojmenování fotoaparátu (např. IMG\_0001.RAW)

### Zeměpisná šířka

* Zeměpisná šířka snímku

### Zeměpisná délka

* Zeměpisná délka snímku

### Nadmořská výška

* Nadmořská výška snímku

{% hint style=&quot;info&quot; %}
Kliknutím na záhlaví sloupců tabulky se také seřadí data řádků.
{% endhint %}

***

## Značky obrázků

Každý obrázek s GPS daty je na mapě znázorněn značkou:

### Zobrazení značek

* Značky označují přesné GPS souřadnice, kde byl každý obrázek pořízen.
* Seskupené značky se mohou při oddálení zobrazení seskupit.
* Přiblížením zobrazení uvidíte jednotlivé polohy obrázků.

{% hint style=&quot;success&quot; %}
SUPER-ZOOM: Když dosáhnete maximální úrovně přiblížení od poskytovatele mapových dlaždic, dlaždice se při dalším přiblížení zvětší, což vám umožní vidět značky, které jsou blízko u sebe.
{% endhint %}

### Náhled při najetí myší

* **Přejděte myší** na libovolný značkovač a zobrazí se náhled miniatury daného obrázku.
* To umožňuje rychlou vizuální identifikaci, aniž byste museli opustit zobrazení mapy.
* Užitečné pro vyhledání konkrétních obrázků v rámci rozsáhlé relace snímání.

***

## Poskytovatelé mapových dlaždic

{% hint style=&quot;success&quot; %}
**Automatický výběr**: Chloros automaticky vybere službu dlaždic, která poskytuje nejlepší úroveň přiblížení pro vaši aktuální polohu na mapě. V případě potřeby můžete ručně přepínat mezi poskytovateli.
{% endhint %}

Karta Mapa podporuje dva poskytovatele dlaždic pro obrázky pozadí mapy:

### Google Maps

* Standardní satelitní a mapové snímky od Google
* Nejlepší pro obecné celosvětové pokrytí

### ESRI

* Satelitní a letecké snímky od ESRI ArcGIS
* Často poskytuje snímky s vyšším rozlišením v určitých regionech

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

### Ovládací prvky přiblížení

* **Přiblížit/oddálit**: Použijte kolečko myši nebo tlačítka přiblížení.
* **Celá obrazovka**: Zobrazí mapu na celou obrazovku.

### Ovládací prvky posunu

* **Posun**: Kliknutím a tažením se můžete pohybovat po mapě.***

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

**Řešení**: Ověřte, zda je GPS ve fotoaparátu zapnuté, a znovu importujte původní soubory.

### Značky na nesprávném místě

**Možné příčiny:**

* GPS fotoaparátu mělo špatné satelitní připojení.
* Během pořizování snímků došlo k odchylce GPS.

**Řešení**: Jedná se obvykle o problém s časem pořízení snímku. Zvažte použití PPK/RTK GPS pro přesné aplikace.
