# Grafické uživatelské rozhraní: Navigace

Při prvním spuštění aplikací Chloros a Chloros (prohlížeč) se spustí jejich backend. Jakmile bude připraven, zobrazí se ikona hlavního menu v levém horním rohu <img src=".gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

Horní záhlaví obsahuje zleva doprava:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Hlavní nabídka

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

Z hlavního menu můžete:

* **Nový projekt** — vytvořit nový projekt
* **Otevřít projekt** — otevřít existující projekt
* **Otevřít složku projektu** — otevřít složku projektu ve vašem prohlížeči souborů
* **Přidat soubory** — přidat jednotlivé obrazové soubory do aktuálního projektu _(viditelné po otevření projektu)_
* **Přidat složku** — přidat složku s obrázky do aktuálního projektu _(viditelné po otevření projektu)_
* **Spustit zpracování / Zastavit zpracování** — spustit nebo zastavit proces zpracování obrázků _(aktivní po přidání souborů)_

{% hint style="info" %}
**Pouze Windows**: Grafické uživatelské rozhraní Chloros Desktop je k dispozici na Windows. Uživatelé Linux by měli vidět [CLI](CLI.md) a [Python SDK](api-python-sdk.md) týkající se zpracování bez grafického rozhraní.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1).png" alt="" data-size="line"> Tlačítko Přehrát/Spustit

Je-li tato funkce povolena, tlačítko pro spuštění zpracování spustí proces zpracování obrazu.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> Indikátor průběhu <img src=".gitbook/assets/image (5).png" alt="" data-size="line">V bezplatném režimu Chloros, který zpracovává všechny soubory postupně, ukazuje ukazatel průběhu 2 fáze: Detekce cíle a Zpracování.

V placeném režimu s licencí Chloros+, který zpracovává všechny soubory současně, ukazuje ukazatel průběhu 4 fáze: Detekce, Analýza, Kalibrace, Export. Pokud najedete kurzorem myši na ukazatel průběhu Chloros+, rozbalí se rozšířený panel se 4 ukazateli průběhu, abyste mohli sledovat průběh. Kliknutím na horní ukazatel průběhu panel rozbalený panel zamrazíte, dalším kliknutím jej opět odblokujete.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Boční nabídka

Levá boční nabídka obsahuje různé ikony, se kterými můžete pracovat:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Nastavení projektu](project-settings/project-settings.md)

Záložka Nastavení projektu umožňuje upravit globální nastavení projektu a nastavení zpracování. Upravte je před zahájením zpracování souborů.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Prohlížeč souborů

Přidávejte soubory/složky a odstraňujte soubory z projektu. Duplicitní soubory jsou ignorovány. Zaškrtněte políčko v sloupci cíl pro libovolný cílový obrázek a zpracování bude hledat cíle pouze u zaškrtnutých obrázků, což výrazně zrychlí dobu zpracování. Pomocí přepínače Obrázek/Metadata můžete přepínat mezi zobrazením mřížky miniatur vybraného obrázku a podrobnou tabulkou metadat.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Prohlížeč obrázků](image-viewer-gui/opening-an-image-full-screen.md)

Po kliknutí na obrázek v hlavním prohlížeči obrázků se obrázek otevře na celé obrazovce na kartě Prohlížeč obrázků.

#### <img src=".gitbook/assets/image (7).png" alt="" data-size="line"> [Mapa](image-viewer-gui/map-markers.md)

Prohlížejte si své snímky na interaktivní 2D mapě podle jejich GPS souřadnic. Podporuje poskytovatele mapových dlaždic Google Maps a ESRI a automaticky vybírá nejlepší službu pro vaši polohu. Umístěte kurzor na značky a zobrazte náhledy miniatur snímků.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Log ladění

V případě potíží zkontrolujte log pro výpisy ladění. Zkopírujte/stáhněte log a odešlete jej na [MAPIR podporu](https://www.mapir.camera/community/contact) pro pomoc.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Přihlášení uživatele](chloros+-login.md)

Postranní panel pro přihlášení uživatele vám umožňuje přihlásit se k vašemu účtu Chloros+ a odemknout pokročilé funkce. Můžete také zobrazit aktuální verzi aplikace a upravit jazyk zobrazeného textu v grafickém rozhraní Chloros a CLI.
