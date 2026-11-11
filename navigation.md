# Grafické uživatelské rozhraní: Navigace

Při prvním spuštění aplikace Chloros se spustí její zpracovatelský backend. Jakmile je backend připraven, zobrazí se v levém horním rohu ikona hlavního menu <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> a v levém postranním panelu se odemknou záložky „Kamery“ a „Světelné senzory“ (do té doby jsou šedé).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Horní záhlaví obsahuje zleva doprava:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Hlavní nabídka

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>Z hlavního menu můžete:

* **Nový projekt**— vytvořit nový projekt. Pokud máte uložené šablony projektů, zobrazí se rozevírací nabídka**Vybrat šablonu**, díky čemuž nový projekt začne s nastavením šablony.
* **Otevřít projekt**— otevřít existující projekt. Seznam obsahuje tlačítko**Otevřít složku projektu**, které otevře složku s projekty ve vašem průzkumníku souborů.
* **Duplikovat projekt** — zkopíruje aktuálně otevřený projekt pod novým názvem (navrhuje se volný název, např. „MůjProjekt (2)“) a otevře kopii. _(viditelné po otevření projektu)_
* **Přidat soubory** — přidá jednotlivé obrazové soubory do aktuálního projektu _(viditelné po otevření projektu)_
* **Přidat složku** — přidá jednu nebo více složek s obrázky do aktuálního projektu _(viditelné po otevření projektu)_
* **Spustit zpracování / Zastavit zpracování** — spustí nebo zastaví proces zpracování obrázků _(aktivní po přidání souborů)_
* **Připojit k kameře** — přejde na [kartu Kamery](lattice/) pro připojení kamery nebo pole kamer LATTICE. Funguje i bez otevřeného projektu.
* **Připojit k světelnému senzoru** — přejde na [kartu Světelné senzory](daq/) pro připojení světelného senzoru DAQ. Funguje i bez otevřeného projektu.

{% hint style="info" %}
**Pouze pro Windows**: Grafické uživatelské rozhraní Chloros

Desktop je k dispozici pro systém Windows

. Uživatelé systému Linux

by si měli pro zpracování bez grafického rozhraní prostudovat dokumentaci k [CLI

](CLI.md) a [Python

SDK

](api-python-sdk.md).
{% endhint %}

### <img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">

Tlačítko Přehrát/Spustit

Je-li tato funkce povolena, tlačítko pro spuštění zpracování zahájí proces zpracování obrazu.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line">

Ukazatel průběhu <img src=".gitbook/assets/image (5).png" alt="" data-size="line">



V bezplatném režimu Chloros

, který zpracovává všechny soubory postupně, ukazuje ukazatel průběhu 2 fáze: Detekce cíle a Zpracování.

V placeném režimu s licencí Chloros

+, který zpracovává všechny soubory současně, ukazuje ukazatel průběhu 4 fáze: Detekce, Analýza, Kalibrace, Export. Pokud najedete kurzorem myši na ukazatel průběhu Chloros

+, rozbalí se rozšířený panel se čtyřmi ukazateli průběhu, díky čemuž můžete sledovat průběh zpracování. Kliknutím na horní ukazatel průběhu panel rozbalený stav pozastavíte, dalším kliknutím jej opět obnovíte.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Boční nabídka

Levá boční nabídka obsahuje různé ikony pro interakci, a to v tomto pořadí shora dolů:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Nastavení projektu](project-settings/project-settings.md)

Karta Nastavení projektu umožňuje upravit globální nastavení projektu a nastavení zpracování. Nastavte je před zahájením zpracování souborů.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Prohlížeč souborů

Přidávejte soubory/složky do projektu a odstraňujte je z něj. Duplicitní soubory jsou ignorovány. Zaškrtněte políčko v sloupci „Cíl“ u libovolného cílového obrázku a zpracování bude pro cíle zohledňovat pouze zaškrtnuté obrázky, což výrazně zrychlí dobu zpracování. Pomocí přepínače Obrázek/Metadata můžete přepínat mezi zobrazením mřížky miniatur vybraných obrázků a podrobnou tabulkou metadat.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Prohlížeč obrázků](image-viewer-gui/opening-an-image-full-screen.md)

Po kliknutí na obrázek v hlavním prohlížeči obrázků se obrázek otevře na celé obrazovce na kartě Prohlížeč obrázků.

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Prohlížeč map](image-viewer-gui/map-markers.md)

Prohlížejte si své obrázky na interaktivní 2D mapě podle jejich GPS souřadnic. Podporuje poskytovatele mapových dlaždic Google Maps a ESRI a automaticky vybírá nejlepší službu pro vaši polohu. Přejděte kurzorem myši nad značky a zobrazte náhledy miniatur snímků.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Kamery](lattice/)

Připojujte a ovládejte kamery LATTICE v reálném čase – jednotlivě nebo jako synchronizované soustavy více kamer. Na této záložce se zobrazují dlaždice s živým náhledem s překryvnými vrstvami a histogramy, nastavení pro jednotlivé kamery i pro celé soustavy a nastavení snímání, která určují, které kamery a typy exportu funkce „Capture All“ použije. K dispozici, jakmile bude připraven backend; kompletní návod najdete v [sekci LATTICE](lattice/).

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Světelné senzory](daq/)

Připojte světelné senzory DAQ — DAQ-U (USB), DAQ-M (Bluetooth) a DAQ-E (Ethernet) — a prohlížejte si jejich živé kalibrované spektrální grafy v jednotkách W/m²/nm. Zde můžete do otevřeného projektu zaznamenávat soubory `.daq`, přejmenovávat senzory, vybírat profily korekce kapacity a aktualizovat firmware DAQ-E. Funkce bude k dispozici, jakmile bude backend připraven; kompletní návod najdete v [sekci DAQ](daq/).

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Loga ladění

V případě problémů zkontrolujte log, zda neobsahuje výpisy ladění. Zkopírujte nebo stáhněte log a zašlete jej [podpoře MAPIR](https://www.mapir.camera/community/contact) s žádostí o pomoc.

Novinka ve verzi 1.3.0: ikona chyby v horní části záložky protokolu (**Nahlásit chybu**) odešle společnosti MAPIR zprávu s vaším popisem, až 10 obrázky, protokoly Chlorosu a podrobnostmi o systému. K odeslání zprávy musíte být přihlášeni. Viz [Nahlášení chyby z karty protokolu](processing-images-gui/monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Přihlášení uživatele](chloros+-login.md)

Postranní panel pro přihlášení uživatele vám umožňuje přihlásit se k vašemu účtu Chloros+, abyste odemkli pokročilé funkce. Můžete si také zobrazit aktuální verzi aplikace a nastavit jazyk zobrazeného textu v grafickém rozhraní Chloros a v CLI.
