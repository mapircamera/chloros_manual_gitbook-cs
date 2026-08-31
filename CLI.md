# CLI : Příkazový řádek

> **Kompletní příručka:**[CLI Reference](reference/cli-reference.md) dokumentuje**všechny parametry všech podpříkazů** a je optimalizována pro AI asistenty — vložte její URL do svého asistenta a požádejte o funkční příkaz: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Tip pro AI nástroje:** jakákoli stránka tohoto manuálu je dostupná ve formátu surového Markdownu, stačí k její adrese URL připojit `.md` (např. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), a `https://mapir.gitbook.io/chloros/llms.txt` indexuje celou příručku pro použití velkými jazykovými modely (LLM).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->


## Co je toCLI


`chloros-cli` je příkazový řádek sloužící jako rozhraní pro stejný zpracovatelský engine, jaký používá desktopová aplikaceChloros
. Jedná se o tenkého klienta typu „HTTP
“ nad backendemChloros
(lokální server na `127.0.0.1:5000`) — většina příkazů spustí backend automaticky, takže skriptu stačí jediné volání `chloros-cli process …`.

Běží na **Windows
10/11 (x64)**a**Linux
(x86_64 a NVIDIA Jetson arm64 na JetPack 6)**, v jakémkoli terminálu, bez nutnosti grafického rozhraní. Ověřte si instalaci pomocí:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

Přehled skupin příkazů:

* **Zpracování a účet** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 jazyků — viz [Podporované jazyky](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (pouzeLinux
/ Jetson)
* **Hardware v reálném čase** — `lattice` (ovládání kamery LATTICE, více než 45 podpříkazů), `daq pool-*` (světelné senzory DAQ), `time-sync` (PTP)
* **Automatizace** — `project` (spouštění uloženého projektuChloros
v bezhlavém režimu, včetně receptů pro snímání ve formátu YAML)

Globální volby, které stojí za to znát: `--port N` (port backendu, výchozí `5000`), `-v/--verbose`, `--restart` (vynucený restart backendu), `--backend-exe PATH`. Úplný seznam najdete v [CLI
Referenci](reference/cli-reference.md).

***

## Instalace

CLI
**je součástí instalačního programuChloros** na všech platformách — neexistuje samostatný soubor ke staženíCLI
. Instalační program si stáhněte ze stránky [Stáhnout](download.md).

###Windows


Instalační program umístíCLI
do adresáře:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

a přidá tuto složku do vašeho systému `PATH` — po instalaci **otevřete nový terminál**, aby se načítal aktualizovaný `PATH`. Instalační program také umístí spouštěcí skripty (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) do kořenového adresáře instalace a navíc**Chloros
CLI
** zkratku v nabídce Start, z nichž každá otevře terminál s `chloros-cli` připraveným k použití.

###Linux


Nainstalujte `.deb` pro vaši architekturu:

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Tím se nainstaluje `chloros-cli` až `/usr/bin/chloros-cli` (již na verzi `PATH`) a backend na verzi `/usr/lib/chloros/chloros-backend`, spolu s runtime ArenaSDK
potřebným pro kamery LATTICE. Podrobnosti najdete v [InstalaciLinux
](linux/linux-installation.md).

### Ověření

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Přihlášení a licence

CLI
(aPython
SDK
) vyžaduje **placený tarifChloros
+**— ten je k dispozici u všech placených tarifů; bezplatný tarif jej neobsahuje. Omezení je vynucováno**na straně serveru** backendem, nikoli binárním souboremCLI
: volání bez přihlášení je odmítnuto s kódem `401 AUTH_REQUIRED`, a volání od přihlášeného uživatele v bezplatném tarifu kódem `403 PLAN_UPGRADE_REQUIRED`, ať už pochází z `chloros-cli`,SDK
nebo z ručně vytvořeného klientaHTTP
. Upgradujte na [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Přihlaste se **jednou na každém počítači**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->


{% hint style="warning" %}
**Hesla se speciálními znaky**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` je shellem zkomolené (CLI
to detekuje na chybě 401 a automaticky to zkusí znovu, ale uvozovky tento problém zcela vyřeší).
{% endhint %}

Relace je uložena v mezipaměti v `~/.chloros/user_session.json` a funguje offline po dobu tolerančního období daného tarifu (30 dní u měsíčních tarifů, do vypršení platnosti u ročních). `chloros-cli status` funguje i bez placeného tarifu, takže důvod odmítnutí je vždy viditelný.

{% hint style="danger" %}
**Plánujete bezobslužnou práci? Nejprve se přihlaste.**Příkazy spouštějící backend (`process`, `status`, `export-status`, …) spuštěný**bez uložené relace**neskončí okamžitým selháním — přejde do interaktivního příkazového řádku `Email:` / `Password:` na stdin. Automatizovaná úloha cronu nebo krok CI proto**zůstane viset a bude čekat na vstup**. Před naplánováním jakékoli úlohy spusťte na daném počítači jednou příkaz `chloros-cli login EMAIL 'PASSWORD'`.
{% endhint %}

***

## Vaše první spuštění zpracování

Nasměrujte `process` na složku se zachycenými daty — automaticky detekujeSurvey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng`, nebo jejich kombinaci:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Průběh se zobrazuje v reálném čase pro každé vlákno potrubí (Detekce, Analýza, Zpracování, Export) a úspěšný běh končí hlášením počtu zapsaných obrazových produktů (`Image products written: N`).



<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### Kam se ukládají výstupy

`process` zapisuje do **projektové složky**, nikoli do vaší vstupní složky:

* Bez `-o`: projekt se vytvoří ve vaší výchozí projektové složce (sdílené s grafickým uživatelským rozhraním; spravujte ji pomocí `get-project-folder` / `set-project-folder`, záložní `~/Chloros Projects`), pojmenovaný podle `-n/--project-name` nebo časového razítka (`YYYYMMDD_HHMMSS`), pokud je vynechán.
* S `-o PATH`: tato složka **je** projektovou složkou. Pokud již obsahuje soubor `project.json`, vytvoří se místo přepsání souběžný soubor s příponou `_1`/`_2`…

Uvnitř projektu jsou produkty seskupeny **podle fotoaparátu a poté podle formátu souboru**:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Složka fotoaparátu je `LATT-<sensor>-<lens>-F<filter>` pro LATTICE (odpovídá EXIF záznamu `Model`) a `<model>_<filter>` (např. `Survey3N_RGN`) proSurvey3
. Složka formátu navazuje na `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` nebo `tiff32` pro `TIFF (32-bit, Percent)`.

{% hint style="info" %}
**Každý exportovaný produkt si zachovává název zdrojového souboru.**Export dat o radianci souboru `capture_..._raw.tif` se stále jmenuje `capture_..._raw.tif` — pouze se nachází v adresáři `tiff32/Radiance_Images/`.**Produkt identifikuje složka, nikoli název souboru**, proto použijte globální výraz pro adresář, nikoli pro příponu `*radiance*`.
{% endhint %}

### Možnosti, které budete skutečně používat

| Příznak | Výchozí | Co dělá |
| --- | --- | --- |
| `-o, --output PATH` | výchozí složka projektu | Umístění složky projektu (viz výše). |
| `-n, --project-name NAME` | časové razítko | Název projektu. |
| `--format FMT` | `TIFF (16-bit)` | Jedno z `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | žádné | Vegetace indexy k exportu (viz [Vegetace indexy](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = neuronový debayer, pomalejší, nejvyšší kvalita (Chloros
+, GPU NVIDIA). |
| `--vignette / --no-vignette` | zapnuto | Korekce vinětace. |
| `--reflectance / --no-reflectance` | zapnuto | Kalibrace odrazivosti; u formátu LATTICE slouží také jako přepínač produktu odrazivosti. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Vynutí vstupní bod zpracování pro soubory TIFF v LATTICE. |

Všechny ostatní parametry — ladění detekce cíle, PPK, expoziční piny, příznaky zarovnání polí — najdete v [sekci `process` v referenční příručceCLI
](reference/cli-reference.md).

***

## Výběr toho, co se má exportovat (produkty LATTICE)

Zpracování LATTICE se rozvětvuje na **všechny příslušné produkty v jednom průchodu**. Čtyři přepínače pro jednotlivé produkty jsou**ve výchozím nastavení zapnuté**; k vypnutí jednoho z nich použijte formulář `--no-`:

| Přepínač | Produkt |
| --- | --- |
| `--debayered` | Lineární demosaik → `Debayered_Images/` |
| `--preview` | Zobrazení náhledu (vyvážení bílé + gama; roztažení falešných barev pro multispektrální snímky) → `Preview_Images/` |
| `--radiance` | zářivost typu float32, W/m²/sr/nm → `Radiance_Images/` (vždy `tiff32/`) |
| `--reflectance` | uint16 odrazivost, připraveno pro Pix4D → `Reflectance_Calibrated_Images/` |

RGB
hlavní kamery vždy vysílají pouze data po odstranění Bayerova filtru + náhled — radiance/reflektance pro jednotlivá pásma nemá pro širokopásmový senzor smysl, takže tyto přepínače pro ně nemají žádný účinek.Survey3
`.raw` ignoruje přepínače a řídí se standardní cestou pro reflektanci/cíl.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (výchozí `auto`) volí referenci odrazivosti: `auto` vytvoří v rámci snímku [kalibrační cíl](calibration-targets.md) splňující požadavky kontroly kvality jako absolutní referenci a v případě absence cíle se vrací k dělení sestupného toku světla světelným senzorem DAQ (ρ = π·L/E); `target` je přísný (bez nahrazení hodnotami z DAQ); `daq` se řídí hodnotami z DAQ. Skeny měřených terčů v jednotkách lze poskytnout pomocí `--target-reflectance-dir`.

{% hint style="info" %}
**Čtení pixelů odrazivosti:**hodnota DN znamenající ρ = 1,0 je**na zdroj** — Soubory LATTICE vkládají do XMP značku `Chloros:PixelScale=32768`; souborySurvey3
používají hodnotu 65535 (a neobsahují žádné značky `Chloros:*`). Načtěte značku a vydělte touto hodnotou, místo abyste předpokládali konstantu. Podrobnosti a jeden záměrný okrajový případ bez měřítka najdete v [CLI
Referenci](reference/cli-reference.md).
{% endhint %}

**Zpracování vždy začíná od `raw`.** Odvozené produkty (exporty bez debayeringu, radiance a odrazivosti) se nikdy nevracejí zpět do zpracovatelského řetězce — jejich opětovný import a zpracování by znamenalo dvojí použití kalibračních výpočtů, proto jeChloros
přeskočí a oznámí to. `--input-level` je záměrná úniková cesta pro případ, kdy skutečně potřebujete vynutit vstupní bod.

***

## Když se spuštění nezdaří

Od verze 1.2.0 hlásí `process` zřetelně selhání namísto „úspěchu“ bez jakéhokoli výstupu:

* Spuštění, které **požadovalo produkty, ale žádný nezapsalo**— pouze `project.json` a `calibration_data.json` — vypíše `Processing finished but wrote no image products.` a**ukončí se s nenulovým stavem**, takže to skripty mohou detekovat. Obvyklé příčiny: vstupní složka nebyla rozpoznána jako záznam (zkontrolujte rozložení a `--input-level`), nebo žádný z požadovaných produktů nebyl pro dané kamery použitelný (např. požadavek na radianci/odrazivost u kamer podporujících pouzeRGB
).
* **Úmyslné spuštění pouze s metadaty** (všechny produkty vypnuté, bez `--indices`) je stále úspěšné — prázdný obrazový výstup je v tomto případě správným výsledkem.
* Spusťte proces znovu s parametrem `--verbose` a zkontrolujte protokol backendu, zda neobsahuje řádky `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, které vysvětlují vynechání jednotlivých kamer.

Kódy ukončení: `0` úspěch · `1` obecná chyba · `2` chyba argumentu · `130` přerušeno klávesovou zkratkou Ctrl+C.

***

## Vegetace indexy

Spusťte `--indices` s jedním nebo více názvy předvoleb; každý index se uloží do vlastní složky `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

22 přednastavených názvů, které `process --indices` přijímá:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**Existují tři seznamy indexů – nezaměňujte je.**Rozbalovací nabídka „Nastavení projektu“ v grafickém uživatelském rozhraní obsahuje 27 vzorců (přidává `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — těchto pět vzorců je určeno pouze pro grafické rozhraní a**neplatí** pro `--indices`). Příkaz live/offline `lattice index --preset` používá vlastní samostatný seznam 22 předvoleb. Vzorce a výpočty pásem jsou popsány v [Vzorce multispektrálních indexů](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Světelné senzory DAQ: Stručný přehled

Řada `daq pool-*` řídí spektrální senzory DAQMAPIR
(DAQ-U přes USB, DAQ-M přes BLE, DAQ-E přes Ethernet) prostřednictvím trvalého fondu backendu — grafické uživatelské rozhraní (GUI),CLI
aSDK
sdílejí jeden živý popisovač. **`pool-*` je podporovaná cesta DAQ v dodávaném balíčkuCLI
**; ostatní podpříkazy typu `daq`, na které můžete narazit, jsou pouze interním rozhraním typu „MAPIR
“ a ukončí se explicitní chybou odkazující na `pool-*`.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record` bez `--duration` běží až do `pool-record --stop`; výchozí výstupní adresář je `~/Documents/DAQ Live View/` **na stroji backendu**. Profil korekce kondenzátoru se volí při připojení (`--cap-id`, výchozí nastavení backendu `sunshine_cosine`) a lze jej za běhu změnit pomocí `pool-set-cap` — profily limitů a kalibrovaný rozsah snímače jsou popsány v kapitolách o DAQ v této příručce.

{% hint style="warning" %}
**DAQ-E na hostitelském počítači s více síťovými kartami:** první automatické vyhledání `pool-connect --eth` po spuštění systému může selhat i u funkčního senzoru. `--eth-host <ip-or-hostname>` je spolehlivější varianta — použijte ji vždy, když vyhledání nedopadne úspěšně.
{% endhint %}

***

## Kamery LATTICE, PTP a automatizace projektů

Řada `lattice` (více než 45 podpříkazů) pokrývá práci s kamerami LATTICE od začátku do konce: detekci, jednotlivé snímky, trvalá synchronizovaná pole s procesem připojení „smart-prep“ v grafickém uživatelském rozhraní, živý náhled v prohlížeči, zarovnání, výpočty indexů a diagnostiku hostitelské síťové karty. Ukázka:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Kromě toho: `chloros-cli time-sync` poskytuje informace o PTP grandmasteru, který běží na hostitelském počítačiChloros
(kamery LATTICE a senzory DAQ-E se k němu připojují jako slave pro časová razítka napříč zařízeními), a `chloros-cli project` otevírá uložený projektChloros
a bez grafického rozhraní ovládá jeho kamery, pole a senzory — včetně skriptovaných receptů pro snímání ve formátu YAML.

Tyto tři rodiny (`lattice`, `project`, `daq pool-*`) jsou také jediné, které podporují `CHLOROS_BACKEND_URL` pro ovládání **vzdáleného** backendu; základní příkazy se vždy zaměřují na lokální stroj.

Kompletní návody najdete v kapitolách věnovaných LATTICE v této příručce; všechny příznaky jsou uvedeny v [CLI
Referenci](reference/cli-reference.md).

***

## Řešení problémů: Top 5

| Příznak | Řešení |
| --- | --- |
| `Login required` nebo naplánovaná úloha se zasekne na výzvě `Email:` | Spusťte na tomto počítači jednou příkaz `chloros-cli login EMAIL 'PASSWORD'` — příkazy bez uložené relace se budou spouštět interaktivně, místo aby okamžitě selhaly. |
| `backend unreachable` | Spusťte desktopovou aplikaciChloros
nebo přímo spusťte binární soubor backendu (`chloros-backend`). Pokud nasměrujete `lattice`/`project`/`daq pool-*` na vzdálený backend, zkontrolujte `CHLOROS_BACKEND_URL`. |
| Blokováno připojení k poli: `FRAMES WILL DROP` / `Reduce ROI to enable` | Obnovení výchozích nastavení přijímacího kruhu síťové karty hostitele — nejčastější příčina toho, že dříve funkční zařízení odmítá navázat připojení, obvykle po aktualizaci ovladače síťové karty. Spusťte příkaz `chloros-cli lattice network --fix` z terminálu s **právo** (nebo nastavte `ReceiveBufferLen=256`, `PendingReceives=64`); viz část *Nastavení a ladění síťové karty hostitele* v příručce. |
| Podpříkaz `daq` se ukončí s hlášením: „vyžaduje kompletní balíček daq…“ | Očekávané u dodávaných sestavení — zkompilovaný balíčekCLI
obsahuje pouze rodinu příkazů `daq pool-*`, která pokrývá připojení, streamování, záznam a výběr kap. Použijte `pool-*` (nebo `chloros_sdk.connect_daq_sensor()` zPython
). |
| Jetson před zpracováním velkých složek zobrazí varování ohledně swapování | Přidejte swap založený na souborech — souborCLI
vypíše přesné příkazy `fallocate`/`swapon`, které je třeba spustit. |

***

## Nápověda

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Všechny příznaky, všechny podpříkazy:** [CLI
Reference](reference/cli-reference.md)
* **Ekvivalent vPython
:** [Python
SDK
](api-python-sdk.md) a [SDK
Reference](reference/sdk-reference.md)
* **Podpora:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
