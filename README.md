---
metaLinks: {}
---

# První kroky

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros

je softwarová aplikace od [MAPIR

](https://www.mapir.camera) určená ke zpracování multispektrálních snímků, živému ovládání hardwaruMAPIR

a záznamu dat ze senzorů.Chloros

1.2.0 podporuje celou produktovou řaduMAPIR

:

* **KamerySurvey3** — zpracovávají snímky ve formátu RAW+JPG na kalibrované mapy odrazivosti a vegetačních indexů. Viz [Podporované kamery](supported-cameras.md).
* **Kamery LATTICE** — připojte multispektrální kamerové moduly GigE v reálném čase, jednotlivě nebo jako synchronizovaná pole více kamer: náhled, snímání a zpracování do kalibrovaných produktů zářivosti a odrazivosti. Viz [sekce LATTICE](lattice/README.md).
* **Světelné senzory DAQ** — spektrální senzory DAQ-U (USB), DAQ-M (Bluetooth) a DAQ-E (Ethernet): živá kalibrovaná spektra, záznamy `.daq` a osvětlení směrem dolů pro zpracování odrazivosti. Viz [sekce DAQ](daq/README.md).

{% hint style="success" %}
**Novinky ve verziChloros

1.2.0**: ovládání kamer a polí LATTICE v reálném čase, integrace světelných senzorů DAQ, režimy snímání a záznamníky, kompletní radiometrický zpracovatelský řetězec LATTICE, automatizace projektů zCLI

/SDK

a mnoho dalšího. Podívejte se na seznam novinek níže a [stáhněte si](download.md) seznam změn.
{% endhint %}

{% hint style="info" %}
**PoužíváteChloros

s AI asistentem?** Tato příručka je pro to jako stvořená. Nasměrujte svého asistenta na:

* `https://mapir.gitbook.io/chloros/llms.txt` — strojově čitelný index všech stránek.
* Jakoukoli stránku ve formátu Markdown — připojte `.md` k její adreseURL

(např. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI

Reference](reference/cli-reference.md) a [SDK

Reference](reference/sdk-reference.md) — kompletní referenční stránky s přesnými hodnotami napsané pro použití velkými jazykovými modely (LLM).

Příklad zadání: *„Přečti si https://mapir.gitbook.io/chloros/reference/cli-reference.md, a poté napiš skript, který se přihlásí a zpracuje složku ~/flights/flight_001 do formátu GeoTIFF s odrazivostí aNDVI

.“*

Kompletní průvodce: [PoužíváníChloros

s AI asistenty](ai-assistants.md).
{% endhint %}

***

## Co je nového v aplikaci „Chloros

“ 1.2.0

* **Ovládání kamery v reálném čase — nová záložka „Kamery“.** Připojte kamery LATTICE jednotlivě nebo jako synchronizované soustavy více kamer (časová synchronizace PTP, snímání spouštěné hardwarem) s překryvnými živými náhledy, histogramy pro jednotlivé pásma, inteligentní automatickou expozicí, kalkulátorem živého indexu a aktualizacemi firmwaru kamer přímo v aplikaci.
* **Světelné senzory — nová záložka „Světelné senzory“.** Připojte senzory DAQ-U (USB), DAQ-M (Bluetooth) a DAQ-E (Ethernet); prohlížejte si živá kalibrovaná spektra (W/m²/nm), ukládejte soubory `.daq` do svého projektu, vybírejte profily korekce kap a aktualizujte firmware DAQ-E přes síť.
* **Režimy snímání a záznamníky.** Jednorázové / nepřetržité / intervalové snímání plus režim „Fastest Capture“ pouze pro surová data; výběr pro každý projekt, které kamery a typy exportu generuje funkce „Capture All“; pole záznamníků pro indexové video v monitorovací kvalitě a surové série snímků v analytické kvalitě s offline sestavením videa.
* **Zpracovací pipeline LATTICE.** Importujte složky se záznamy z LATTICE a rozdělte každý surový snímek na produkty s odstraněním Bayerova filtru, náhled, radianci float32 (W/m²/sr/nm) a odrazivost s přepínači pro jednotlivé produkty. Reflexe může pocházet z kalibračního cíle v rámci snímku nebo z datového toku DAQ; při exportu se aplikuje zarovnání pole; chybějící tovární kalibrace se automaticky stáhne podle sériového čísla kamery.
* **Projekty si pamatují hardware.** Připojené kamery a světelné senzory se ukládají spolu s projektem (`cameras.json` / `sensors.json`) a při opětovném otevření projektu se znovu připojí se svými uloženými nastaveními. Viz [GUI: Projekty](projects.md).
* **Vylepšení prohlížeče obrázků.** Zobrazení pixelů/indexu kurzoru se správným škálováním odrazivosti pro každý soubor, histogramy vrstev, posuvník pro seskupování GSD, režimy mřížky „Per Trigger“ / „Per Camera“, zobrazení produktů LATTICE a export indexu/LUT do sandboxu na disk.
* **Výrazně rozšířené funkce „CLI

“ a „SDK

“.** Nové rodiny příkazů `lattice`, `daq pool-*`, `project` a `time-sync`; nové volby `process` (`--input-level`, přepínače pro jednotlivé produkty, `--reflectance-source`, příznaky zarovnání pole);SDK

inteligentní připojovací identifikátory (`connect_camera` / `connect_array` / `connect_daq_sensor`), které automaticky spouštějí backend; automatizace `open_project()`; balíček „SDK

“ je součástí instalačních programů a je publikován na PyPI jako `chloros-sdk`.
* **Poctivá sémantika selhání.** Spuštění `chloros-cli process`, které požadovalo produkty, ale žádný nezapsalo, nyní hlasitě selže a ukončí se s nenulovým stavem; úspěšná spuštění hlásí, kolik obrazových produktů zapsala.
* **Nové rozložení výstupu.** Výstupy se ukládají do složek `<project>/<camera>/<format>/<Product>_Images/` a zachovávají si název zdrojového souboru — produkt identifikuje složka, nikoli přípona souboru. Viz [Formáty výstupních obrázků](output-image-formats.md).
* **Více vstupů, plánů a jazyků.** Podpora vstupů `.dng`; všech 38 jazyků rozhraní je plně obsazeno; hardwarová omezení pro jednotlivé plány s bezplatným (bez přihlášení) použitím až 4 kamer a 2 světelných senzorů.
* **Spolehlivost.** Funkce „Zastavit zpracování“ se ukončuje čistě s přehledným souhrnem běhu, projekty s více kamerami exportují data ze všech kamer a aktualizace instalačního programu již nezpůsobují odhlášení.***

Chloros

je k dispozici ve 3 aplikačních rozhraních:

##Chloros

: Desktopová aplikace s grafickým uživatelským rozhraním

Samostatné okno se všemi funkcemi, včetně záložek „Live Cameras“ (Živé kamery) a „Light Sensors“ (Světelné senzory). _Pouze pro Windows._

## [Chloros

CLI

: Rozhraní příkazového řádku](CLI.md)

Dávkové zpracování z příkazového řádku a příkazy v reálném čase `lattice`, `daq pool-*`, `project` a `time-sync`. Ideální pro automatizaci, skriptování a provoz bez grafického rozhraní. K dispozici na **Windows

,Linux

amd64 aLinux

arm64 (NVIDIA Jetson)**. _Pro přístup k CLI je vyžadován placený tarifChloros

+._

## [Chloros

API

:Python

SDK

](api-python-sdk.md)

Programové rozhraníPython

pro automatizaci a vlastní pracovní postupy: zpracování celého procesu, živé relace s kamerami/polemi, relace senzorů DAQ a automatizace uložených projektů. Instalováno s balíčkem desktop/CLI

a také publikováno jako `pip install chloros-sdk`. _Pro přístup k API je vyžadován placený tarifChloros

+._

***

## Podporované platformy

| Platforma | GUI |CLI

|Python

SDK

|
| --- | --- | --- | --- |
| **Windows

10/11 (x64)** | Ano | Ano | Ano |
| **Linux

amd64 (x86_64)** | Ne | Ano | Ano |
| **Linux

arm64 (NVIDIA Jetson)** | Ne | Ano | Ano |

Pokyny k instalaci v systémuLinux

najdete v části [Linux

a Edge Computing](linux/linux-overview.md).

***

## Začínáme ve třech krocích

1. **Instalace** — stáhněte si a spusťte instalační program pro vaši platformu. Viz [Stažení](download.md).
2. **Přihlášení (volitelné pro grafické rozhraní)** — grafické rozhraní zpracovává obrázky zdarma i bez účtu. [Chloros

+ přihlášení](chloros+-login.md) odemkne paralelní zpracování, akceleraci GPU, vyšší limity zařízení a přístup kCLI

/SDK

.
3. **Vytvořte svůj první projekt** — otevřeteChloros

, vytvořte [Nový projekt](projects.md), [přidejte své obrázky](processing-images-gui/adding-files-to-a-project.md) a [spusťte zpracování](processing-images-gui/starting-the-processing.md). Chcete-li místo toho ovládat živý hardware, otevřete záložku „Kamery“ nebo „Světelné senzory“ — viz [GUI: Navigace](navigation.md).

***

##Chloros

+

Ačkoli jeChloros

pro většinu úkolů k dispozici zdarma, možná zjistíte, že potřebujete více. Právě v takovém případě vám může být užitečná placená licence proChloros

+. S licencíChloros

+ můžete odemknout nové funkce, jako jsou:

* **Vícevláknové zpracování**: výrazně zrychlete zpracování obrazu u větších projektů díky současnému zpracování obrázků v rámci zpracovatelského řetězce.
* **Akcelerace pomocí GPU (CUDA)**: využijte dnešní možnosti vyšší kapacity paměti GPU k dalšímu zrychlení zpracovatelského potrubí obrázků. Pro dosažení nejlepších výsledků doporučujeme 4 GB nebo více VRAM.
* **Přístup kChloros**[**CLI**](CLI.md) **:** spusťte příkazChloros

+ z příkazového řádku a automatizujte a integrujte jej do svého vlastního softwaru. K dispozici u všech placených tarifů; vynucováno na straně serveru.
* **Chloros

+**[**API**](api-python-sdk.md) **Přístup:** spusťte příkazChloros

+ z adresyPython

pro programové ovládání, což umožňuje hladkou integraci s vašimi výzkumnými procesy, pracovními postupy pro analýzu dat a vlastními aplikacemi. K dispozici ve všech placených tarifech; vynucováno na straně serveru.
* **Vyšší hardwarové limity**: připojte více kamer a světelných senzorů najednou. Bez přihlášení lze přes grafické uživatelské rozhraní připojit až 4 kamery a 2 světelné senzory DAQ; placené tarify zvyšují oba limity:

| Tarif | Kamery | Světelné senzory DAQ |
| --- | --- | --- |
| Iron (zdarma, bez přihlášení) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Použití více zařízení**: každá licenceChloros

+ umožňuje registraci 2 a více zařízení. K správě registrovaných zařízení použijte svůj cloudový účetMAPIR

. Podporu pro další zařízení získáte upgradem licenceChloros

+.
* **Pokročilá metoda debayeringu s ohledem na texturu:** vysoce kvalitní debayering s ohledem na hrany v kombinaci s modelem odšumování založeným na AI/ML, který odstraňuje téměř veškerý šum vznikající při debayeringu.
* **Vlastní vzorce multispektrálních indexů:** zadejte vlastní multispektrální indexy do rastrových kalkulátorůChloros

, a to jak pro zpracování, tak pro testovací prostředí pro prohlížení snímků.
* **SlužbaLinux

a Edge Computing:** spouštějte službuChloros

na platformáchLinux

x86\_64 a ARM64, včetně NVIDIA Jetson, pro zpracování v terénu a na okraji sítě. Viz [Linux

Přehled](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Ceny a registrace</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.2.0 build so the banner prints "Chloros CLI 1.2.0". -->
