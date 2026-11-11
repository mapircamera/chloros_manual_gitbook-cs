# CLI: Příkazový řádek

> **Kompletní příručka:**[Příručka k CLI](reference/cli-reference.md) dokumentuje**všechny parametry všech podpříkazů** a je optimalizována pro AI asistenty — vložte její URL do svého asistenta a požádejte o funkční příkaz: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Tip pro AI nástroje:** jakákoli stránka této příručky je dostupná ve formátu Markdown, stačí k její URL přidat `.md` (např. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), a `https://mapir.gitbook.io/chloros/llms.txt` indexuje celou příručku pro použití v LLM.

> **Chcete, aby váš asistent spouštěl Chloros sám?** `chloros-mcp`, server MCP nainstalovaný s Python SDK, funguje s Claude Desktop, Claude Code, Cursor a dalšími klienty MCP: viz [Používání Chloros s AI asistenty](ai-assistants.md).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.3.0 build so the version line reads "Chloros CLI 1.3.0" -->


## Co je to CLI


`chloros-cli` je rozhraní příkazového řádku pro stejný zpracovatelský engine, jaký používá desktopová aplikace Chloros
. Jedná se o jednoduchého HTTP
klienta připojeného k backendu Chloros
u (lokálnímu serveru na `127.0.0.1:5000`) — většina příkazů spustí backend automaticky, takže skriptu stačí jediné volání `chloros-cli process …`. Backend poté zůstane spuštěný pro další příkaz: zastaví se na `chloros-cli logout` (pokud není otevřena aplikace nebo neprobíhá spuštění či sestavení), na `chloros-cli backend stop`, nebo sám po 30 minutách bez klienta, spuštění či připojeného zařízení. `chloros-cli backend status` to ukazuje.

Běží na **Windows
10/11 (x64)**a**Linux
u (x86_64 a NVIDIA Jetson arm64 na JetPack 6)**, v jakémkoli terminálu, bez nutnosti grafického rozhraní. Ověřte si instalaci pomocí:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

Přehled skupin příkazů:

* **Zpracování a účet** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 jazyků — viz [Podporované jazyky](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (pouze Linux
/Jetson)
* **Funkční hardware** — `lattice` (ovládání kamery LATTICE, více než 45 podpříkazů), `daq pool-*` (světelné senzory DAQ), `time-sync` (PTP)
* **Automatizace** — `project` (spuštění uloženého projektu Chloros
. bez grafického rozhraní, včetně receptů pro snímání ve formátu YAML)

Globální volby, které stojí za to znát: `--port N` (port backendu, výchozí `5000`), `-v/--verbose`, `--restart` (vynutit restart backendu), `--backend-exe PATH`. Úplný seznam najdete v [referenční příručce k CLI
](reference/cli-reference.md).

***

## Instalace

CLI
**je součástí instalačního programu Chloros** na všech platformách — neexistuje samostatný soubor CLI
ke stažení. Instalační program si stáhněte ze stránky [Stahování](download.md).

### Windows


Instalační program umístí CLI
do:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

a tuto složku jednou přidá do vašeho systému `PATH` (aktualizace také odstraní duplicitní položky zanechané předchozími instalačními programy) — po instalaci **otevřete nový terminál**, aby se načítal aktualizovaný `PATH`. Instalační program také umístí spouštěcí skripty (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) do kořenového adresáře instalace a navíc vytvoří**zkratku Chloros
CLI
** zkratku v nabídce Start, z nichž každá otevře terminál s programem `chloros-cli` připraveným k použití.

### Linux


Nainstalujte `.deb` pro vaši architekturu pomocí `apt` (nikoli `dpkg -i`, který vynechává doporučené požadavky Python
SDK
):

```bash
# Linux x86_64
sudo apt install ./chloros_1.3.0_amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
```

Tím se nainstaluje `chloros-cli` až `/usr/bin/chloros-cli` (již na verzi `PATH`), backend a programy CLI
rozbalené do `/usr/lib/chloros/backend/` a `/usr/lib/chloros/cli/` (`/usr/lib/chloros/chloros-backend` je spouštěč, který spouští backend), runtime Arena SDK
, který backend potřebuje pro kamery LATTICE, a Python
SDK
pro systém `python3`. Podrobnosti najdete v části [Instalace na Linux
u](linux/linux-installation.md).

### Ověření

```bash
chloros-cli --version    # "Chloros CLI 1.3.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Přihlášení a licence

Přístup k CLI
(a Python
SDK
) vyžaduje **placený tarif Chloros
+**— ten je k dispozici u všech placených tarifů; bezplatný tarif jej neobsahuje. Omezení je vynucováno**na straně serveru** backendem, nikoli binárním souborem CLI
: odhlášené volání je odmítnuto s chybou `401 AUTH_REQUIRED`, a volání v přihlášeném stavu v bezplatném tarifu kódem `403 PLAN_UPGRADE_REQUIRED`, ať už pochází z `chloros-cli`, SDK
nebo ručně vytvořeného HTTP
klienta. Proveďte upgrade na [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Přihlaste se **jednou na každý počítač**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.3.0 build showing the license tier line -->


{% hint style="warning" %}
**Hesla se speciálními znaky**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` je shellem zkomolen (CLI
to detekuje na chybě 401 a automaticky to zkusí znovu, ale uvozovky tento problém zcela vyřeší).
{% endhint %}

Relace je uložena v mezipaměti pod `~/.chloros/user_session.json` a funguje offline po dobu odkladného období daného tarifu (30 dní u měsíčních tarifů, do vypršení platnosti u ročních). `chloros-cli status` funguje i bez placeného tarifu, takže důvod odmítnutí je vždy viditelný.

{% hint style="danger" %}
**Plánujete bezobslužnou práci? Nejprve se přihlaste.**Příkazy spouštějící backend (`process`, `status`, `export-status`, …) spuštěný**bez uložené relace**neskončí okamžitou chybou — přepne se do interaktivního příkazového řádku `Email:` / `Password:` na standardním vstupu. Automatizovaná úloha cronu nebo krok CI proto**zůstane viset a bude čekat na vstup**. Před naplánováním jakékoli úlohy spusťte na daném počítači jednou příkaz `chloros-cli login EMAIL 'PASSWORD'`.
{% endhint %}

***

## Vaše první zpracování

Nasměrujte příkaz `process` na složku se záznamy — automaticky detekuje Survey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` nebo jejich kombinaci:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Průběh se zobrazuje v reálném čase pro každé vlákno potrubí (Detekce, Analýza, Zpracování, Export) a úspěšný běh končí hlášením počtu zapsaných obrazových produktů: `Image products written: N`, následovaný `(plus M clamp masks)`, pokud běh také zapsal [clamp masky](output-image-formats.md#clamp-masks) (ty se nepočítají jako produkty).

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N (plus M clamp masks)" summary line -->


Pokud již běží backend Chloros
(například ten z desktopové aplikace), `process` jej využije a otevře v něm svůj projekt. Pokud je aplikace otevřená na projektu, který sama otevřela, proces z aplikace odmítne; v takovém případě je třeba tento projekt zavřít. Pokud se předchozí běh ještě dokončuje, počká na něj až 15 sekund, než odmítne. `--restart` vynutí spuštění nového backendu. V systému Linux
se backend spuštěný jako služba systemd nikdy nerestartuje: `process` jej používá tak, jak je, a v situacích, kdy by jej jinak restartoval (`--restart` nebo import, který nic nenašel), vypíše `sudo systemctl restart <unit>` a ukončí se `1`.

### Kam se ukládají výstupy

`process` zapisuje do **projektové složky**, nikoli do vaší vstupní složky:

* Bez `-o`: projekt se vytvoří ve vaší výchozí projektové složce (sdílené s grafickým rozhraním; spravujte ji pomocí `get-project-folder` / `set-project-folder`; pokud není nastaveno žádné, `Documents/MAPIR/Chloros Projects` ve vaší uživatelské složce), pojmenovaný podle `-n/--project-name` nebo časového razítka (`YYYYMMDD_HHMMSS`), pokud je vynechán.
* S `-o PATH`: tato složka **je** projektovou složkou. Pokud již obsahuje soubor `project.json`, vytvoří se místo přepsání soubor s příponou `_1`/`_2`… s příponou, místo aby byl přepsán. Relativní `PATH` se bere relativně ke složce, ze které spouštíte `chloros-cli`.

{% hint style="info" %}
**Relativní cesty (změněno ve verzi 1.3.0).** Každá cesta, kterou backend otevírá — `-o`/`--output` při jakémkoli příkazu (včetně výchozího `output` u příkazů pro zachycení), `--input`, `--burst-dir`, `--profile`, `--save` a `--mask` — je vyhodnocena podle složky, ze které spouštíte `chloros-cli`, takže `-o captures/` zapisuje do `<current folder>/captures/`. Ve verzi 1.2.0 je backend přiřazoval ve své vlastní instalační složce, což vedlo k chybě „Přístup byl odepřen“. V případě vzdáleného backendu (`CHLOROS_BACKEND_URL`) se cesty odesílají tak, jak byly zadány, a přiřazují se na daném počítači.
{% endhint %}

{% hint style="warning" %}
**Windows
: udržujte složku projektu krátkou, pokud nemáte zapnuté dlouhé cesty.** Pokud jsou ve Windows
vypnuté dlouhé cesty (výchozí nastavení Windows
), žádná cesta k souboru nesmí být delší než 260 znaků a produkty se nacházejí až asi 130 znaků pod složkou projektu. `process` toto zkontroluje před spuštěním: příliš hluboká projektová složka je odmítnuta s hlášením **„Projektová složka je pro tento počítač příliš hluboká“**, je navržena nejdelší projektová složka, která se vejde, a je uvedeno, jak zapnout dlouhé cesty — nic se nezpracovává. Použijte kratší `-o` (nebo `set-project-folder`) nebo zapněte dlouhé cesty ve Windows
(možnost instalačního programu **Povolit dlouhé cesty ve Windows**, ve výchozím nastavení zaškrtnutá) a restartujte Chloros
.
{% endhint %}

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

Složka kamery pro LATTICE je `LATT-<sensor>-<lens>-F<filter>` (odpovídá EXIF záznamu `Model`) a `<model>_<filter>` (např. `Survey3N_RGN`) pro Survey3
. Složka formátu navazuje na `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` nebo `tiff32` pro `TIFF (32-bit, Percent)`.

U multispektrální kamery M3C se ve složce formátu může vedle složek produktů objevit také složka `clamp_masks/` (například `tiff16/clamp_masks/Reflectance_Calibrated_Images/` a `tiff32/clamp_masks/Radiance_Images/`). Obsahuje masky zářivosti a odrazivosti na úrovni jednotlivých pixelů, u nichž proces oddělení pásem omezil záporné hodnoty na nulu — viz [Omezovací masky](output-image-formats.md#clamp-masks). Žádná složka s produkty nikdy neobsahuje masku.

{% hint style="info" %}
**Každý exportovaný produkt si zachovává název zdrojového souboru.**Export radiance z `capture_..._raw.tif` se stále jmenuje `capture_..._raw.tif` — pouze se nachází v adresáři `tiff32/Radiance_Images/`.**Produkt identifikuje složka, nikoli název souboru**, proto používejte globální vzor pro adresář, nikoli pro příponu `*radiance*`.
{% endhint %}

### Možnosti, které budete skutečně používat

| Příznak | Výchozí | Co dělá |
| --- | --- | --- |
| `-o, --output PATH` | výchozí složka projektu | Umístění složky projektu (viz výše). |
| `-n, --project-name NAME` | časové razítko | Název projektu. |
| `--format FMT` | `TIFF (16-bit)` | Jedna z hodnot `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros
neprovádí upscaling: u všech 8bitových snímků se formát s větší hloubkou zapíše jako `TIFF (8-bit)` a tato informace se uvede pouze jednou. |
| `--indices NAME [NAME ...]` | žádné | Vegetace indexy k exportu (viz [Vegetace indexy](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = neuronální debayer, pomalejší, nejvyšší kvalita (Chloros
+, vyžaduje GPU s podporou CUDA: bez něj se použije `standard`; není povoleno pro snímky LATTICE, u nichž se vždy použije `standard`). |
| `--vignette / --no-vignette` | zapnuto | Korekce vinětace. |
| `--reflectance / --no-reflectance` | zapnuto | Kalibrace odrazivosti; u LATTICE slouží také jako přepínač produktu odrazivosti. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Vynutit vstupní bod zpracování pro soubory TIFF z LATTICE. |

Všechny ostatní parametry — ladění detekce cílů, PPK, expoziční piny, příznaky zarovnání polí — najdete v [sekci `process` v referenční příručce k CLI
](reference/cli-reference.md).

***

## Výběr toho, co se má exportovat (produkty LATTICE)

Zpracování LATTICE se rozvětvuje na **všechny příslušné produkty v jednom průchodu**. Čtyři přepínače pro každý produkt jsou**ve výchozím nastavení zapnuté**; k vypnutí jednoho z nich použijte formulář `--no-`:

| Přepínač | Produkt |
| --- | --- |
| `--debayered` | Lineární demosaik → `Debayered_Images/`: odstranění tmavého posunu z kalibračního balíčku, následně demosaikování s ohledem na hrany, bez korekce linearity, vyrovnání pole nebo oddělení složek — stejný soubor, jaký zachycení uloží na úrovni `debayered`. Zachována nativní orientace snímače (zaznamenaná v XMP). |
| `--preview` | Náhled na displeji (vyvážení bílé + gama; roztažení falešných barev pro multispektrální snímky) → `Preview_Images/` |
| `--radiance` | zářivost typu float32, W/m²/sr/nm → `Radiance_Images/` (vždy `tiff32/`) |
| `--reflectance` | uint16 odrazivost, připraveno pro Pix4D → `Reflectance_Calibrated_Images/` |

Hlavní kamery RGB
vždy vysílají pouze data po odstranění Bayerova filtru + náhled — radiance/reflektance pro jednotlivá pásma nemá u širokopásmového senzoru smysl, takže tyto přepínače u nich nemají žádný účinek. Survey3
`.raw` ignoruje přepínače a řídí se standardní cestou pro reflektanci/cíl.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (výchozí `auto`) volí referenci odrazivosti: `auto` nastaví [kalibrační cíl](calibration-targets.md) v rámci snímku, který prošel kontrolou kvality (QA), jako absolutní referenci a v případě absence cíle se vrátí k poměru dopadajícího světla a světelného senzoru DAQ (ρ = π·L/E), pokud není k dispozici žádný cíl (cíl, který prošel kontrolou kvality s příznakem, soutěží s DAQ o naměřené skóre); `target` je přísný (žádná náhrada DAQ); `daq` je autoritativní pro DAQ. Skeny měřených cílů na jednotku lze poskytnout pomocí `--target-reflectance-dir`.

{% hint style="info" %}
**Čtení pixelů odrazivosti:**hodnota DN znamenající ρ = 1,0 je**na zdroj** — soubory LATTICE vkládají do XMP značku `Chloros:PixelScale=32768`; soubory Survey3
používají hodnotu 65535 a neobsahují značku `Chloros:PixelScale` (obsahují však jiné značky zpracování `Chloros:*`). Přečtěte značku a vydělte jí, místo abyste předpokládali konstantu. Podrobnosti a jeden záměrný okrajový případ bez měřítka najdete v [Referenci CLI
](reference/cli-reference.md).
{% endhint %}

**Zpracování vždy začíná od `raw`.** Odvozené produkty (exporty bez debayeringu, radiance a odrazivosti) se nikdy nevracejí zpět do zpracovatelského řetězce — jejich opětovný import a zpracování by znamenalo dvojí použití kalibračních výpočtů, proto je Chloros
přeskočí a na to upozorní. `--input-level` je záměrná úniková cesta pro případ, kdy skutečně potřebujete vynutit vstupní bod.

***

## Když se běh nezdaří

Od verze 1.2.0 hlásí `process` zřetelně selhání namísto „úspěchu“ bez jakéhokoli výstupu:

* Spuštění, které **požadovalo produkty, ale žádný nezapsalo**— pouze `project.json` a `calibration_data.json` — vypíše `Processing finished but wrote no image products.` a**ukončí se s nenulovým stavem**, takže to skripty mohou detekovat. Obvyklé příčiny: vstupní složka nebyla rozpoznána jako snímání (zkontrolujte rozložení a `--input-level`), nebo žádný z požadovaných produktů nebyl pro dané kamery použitelný (např. požadavek na radianci/odrazivost z kamer podporujících pouze RGB
).
* **Úmyslné spuštění pouze s metadaty** (všechny produkty vypnuté, bez `--indices`) je stále úspěšné — prázdný obrazový výstup je v tomto případě správným výsledkem.
* Spusťte proces znovu s `--verbose` a zkontrolujte protokol backendu, zda neobsahuje řádky `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, které vysvětlují vynechání jednotlivých kamer.
* Ve Windows
, kde jsou dlouhé cesty zakázány, bude projektová složka, jejíž cesta přesahuje limit 260 znaků, odmítnuta **ještě před** zahájením zpracování, přičemž bude použita nejdelší projektová složka, která se vejde (viz [Kam se ukládají výstupy](#where-the-outputs-land)).
* Příkazy pro záznam (`lattice capture`, `lattice array-capture` a další skupiny příkazů pro záznam) uvedou názvy všech souborů, které se nepodařilo zapsat, a ukončí se kódem `1`; opakované záznamy se zastaví, když je disk plný.

Kódy ukončení: `0` úspěch · `1` obecná chyba · `2` chyba argumentu · `130` přerušeno klávesovou zkratkou Ctrl+C.

***

## Vegetace indexy

Spusťte `--indices` s jedním nebo více přednastavenými názvy; každý index se uloží do vlastní složky `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI GNDVI OSAVI
```

22 přednastavených názvů, které `process --indices` přijímá:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

Do projektu je přidán název, který v něm dosud není uveden. Název, který není přednastavený nebo který kamery nemohou vytvořit, je přeskočen s varováním, v němž je tento název uveden. Pokud CLI
nemůže načíst aktuální seznam indexů projektu, ponechá seznam beze změny a místo jeho nahrazení zobrazí varování.

{% hint style="warning" %}
**Existují tři seznamy indexů — nezaměňujte je.**Rozbalovací nabídka „Nastavení projektu“ v grafickém rozhraní obsahuje 27 vzorců (přidává `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` – těchto pět je určeno pouze pro grafické rozhraní a**neplatí** pro `--indices`). Příkaz live/offline `lattice index --preset` používá vlastní samostatný seznam 22 předvoleb. Vzorce a výpočty pro jednotlivá pásma jsou popsány v [Vzorce multispektrálních indexů](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Světelné senzory DAQ: Stručný přehled

Řada `daq pool-*` řídí spektrální senzory DAQ společnosti MAPIR
(CLI-U přes USB, DAQ-M přes BLE, DAQ-E přes Ethernet) prostřednictvím trvalého fondu backendu — grafické uživatelské rozhraní (GUI), příkazový řádek (CLI
) a SDK
sdílejí jeden živý popisovač. **`pool-*` je podporovaná cesta k DAQ v dodávaném CLI
**; ostatní podpříkazy `daq`, na které můžete narazit, představují pouze interní rozhraní MAPIR
určené pro zdrojový kód a při ukončení vygenerují explicitní chybu odkazující na `pool-*`.

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

`pool-record` bez `--duration` běží až do `pool-record --stop`; výchozí výstupní adresář je `~/Documents/DAQ Live View/` **na stroji backendu**. Profil korekce kondenzátoru se volí při připojení (`--cap-id`, výchozí nastavení backendu `sunshine_cosine`) a lze jej za běhu změnit pomocí `pool-set-cap` — profily limitů a kalibrovaný rozsah senzoru jsou popsány v kapitolách o DAQ v této příručce.

{% hint style="warning" %}
**DAQ-E na hostitelském počítači s více síťovými kartami:** první automatické vyhledání `pool-connect --eth` po spuštění systému může selhat i v případě funkčního senzoru. `--eth-host <ip-or-hostname>` je spolehlivá alternativa — použijte ji vždy, když vyhledávání nedopadne úspěšně.
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

Kromě toho: `chloros-cli time-sync` poskytuje informace o PTP grandmasteru, který provozuje hostitel Chloros
(kamery LATTICE a senzory DAQ-E se k němu připojují jako slave pro časová razítka napříč zařízeními), a `chloros-cli project` otevírá uložený projekt Chloros
a řídí jeho kamery, pole a senzory bez grafického rozhraní — včetně skriptovaných postupů snímání v YAML.

Tyto tři rodiny (`lattice`, `project`, `daq pool-*`) jsou také jediné, které podporují `CHLOROS_BACKEND_URL` pro ovládání **vzdáleného** backendu; základní příkazy se vždy zaměřují na lokální stroj.

Kompletní návody najdete v kapitolách věnovaných LATTICE v této příručce; všechny příznaky jsou uvedeny v [Referenci CLI
](reference/cli-reference.md).

***

## Řešení problémů: Top 5

| Příznak | Řešení |
| --- | --- |
| `Login required` nebo naplánovaná úloha se zasekne na výzvě `Email:` | Spusťte na tomto počítači jednou příkaz `chloros-cli login EMAIL 'PASSWORD'` — příkazy bez uložené relace se budou provádět interaktivně namísto okamžitého selhání. |
| `backend unreachable` | Spusťte desktopovou aplikaci Chloros
nebo spusťte backend ručně: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` ve Windows
u (složka `backend` ve vaší instalační složce; nenachází se v `PATH`), `chloros-backend` v systému Linux
. `chloros-cli backend status` ukazuje, zda je některý spuštěn; `lattice`/`project`/`daq pool-*` nikdy žádný nespustí. Pokud je nasměrujete na vzdálený backend, zkontrolujte `CHLOROS_BACKEND_URL`. |
| Připojení k poli zablokováno: `FRAMES WILL DROP` / `Reduce ROI to enable` | Příjemní prstenec síťové karty hostitele byl resetován na výchozí hodnoty — nejčastější příčina toho, že dříve funkční zařízení odmítá navázat připojení, obvykle po aktualizaci ovladače síťové karty. Spusťte příkaz `chloros-cli lattice network --fix` z terminálu s **zvýšenými oprávněními** (nebo nastavte `ReceiveBufferLen=256`, `PendingReceives=64`); viz část *Nastavení a ladění síťové karty hostitele* v referenční příručce. |
| Podpříkaz `daq` se ukončí s hlášením: „vyžaduje kompletní balíček DAQ…“ | Použijte příkazy `daq pool-*`, které ovládají stejný senzor prostřednictvím backendu: připojení, streamování, záznam, výběr kapacity a v případě DAQ-E také síťová nastavení, restart a aktualizace firmwaru (nebo `chloros_sdk.connect_daq_sensor()` z Python
u). |
| Jetson před zpracováním velkých složek zobrazí varování ohledně swapování | Přidejte swap založený na souborech — CLI
zobrazí přesné příkazy `fallocate`/`swapon`, které je třeba spustit. |

***

## Nápověda

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Všechny příznaky, všechny podpříkazy:** [Referenční příručka CLI
](reference/cli-reference.md)
* **Python
ekvivalent:** [Python
SDK
](api-python-sdk.md) a [Referenční příručka SDK
](reference/sdk-reference.md)
* **Podpora:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
