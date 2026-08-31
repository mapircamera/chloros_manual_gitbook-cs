# Chloros Referenční příručka k příkazu „CLI“

**Verze:**

1.2.0**Vytvořeno:**

29. 7. 2026 19:19 ·**Revidováno:**

30. 8. 2026**Cílová skupina:** Optimalizováno pro použití velkými jazykovými modely (LLM); srozumitelné pro člověka.**Rozsah:** Všechny podpříkazy nástroje `chloros-cli` určené pro uživatele, včetně voleb a příkladů, které lze zkopírovat a vložit.

Tento dokument představuje kompletní referenční příručku k nástroji příkazového řádku `chloros-cli`, který je součástí balíčku MAPIR Chloros. Je záměrně vyčerpávající, aby LLM (nebo člověk) mohl sestavit jakýkoli podporovaný pracovní postup z níže uvedených seznamů, aniž by musel prohlížet zdrojový kód.

Pokud potřebujete pouze to nejdůležitější, přejděte na:
- [Pětiminutový rychlý start](#five-minute-quickstart)
- [První připojení kamery LATTICE](#lattice-camera-first-connect-workflow)
- [První připojení snímače DAQ](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Režimy snímání, záznamníky a offline zpracování](#capture-modes-recorders--offline-reprocess)

---

## Konvence

- Všechny příkazy mají předponu `chloros-cli`. Na Windows je binární soubor `chloros-cli.exe`; na Linux /Jetson je to `chloros-cli`.
- Volitelné argumenty jsou uvedeny ve formátu `--flag`. Povinné poziční argumenty jsou uvedeny bez závorek.
- Je-li uvedena výchozí hodnota, vynecháním příznaku se použije tato hodnota.
- CLI je tenký klient typu „HTTP“ využívající backend Chloros (server Flask na `127.0.0.1:5000`). Backend je automaticky spouštěn většinou příkazů. `CHLOROS_BACKEND_URL=<url>` směřuje rodiny příkazů **`lattice`**,**`project`**a**`daq pool-*`** na vzdálený backend — základní příkazy (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) záměrně zafixujte `http://127.0.0.1:<port>` a ignorujte jej (literál IPv4 zabraňuje penalizaci ~2 s na požadavek způsobené „Windows“ u `localhost`→`::1`). Viz [Proměnné prostředí](#environment-variables).
- Pro všechna volání SDK / CLI je vyžadováno přihlášení pomocí účtu Chloros+ (spusťte `chloros-cli login` jednou na každém počítači; uloží se do mezipaměti v `~/.chloros/`).
- Příklady používají cesty Linux; na Windows nahraďte `/home/user/...` za `C:/Users/.../...`.

---

## Přehled nejvyšší úrovně

```
chloros-cli [global options] COMMAND [command options]
```

### Globální volby

| Příznak | Popis |
| --- | --- |
| `--backend-exe PATH` | Přepsat automaticky detekovaný spustitelný soubor backendu. |
| `--port N` | Port backendu HTTP (výchozí: `5000`). |
| `-v, --verbose` | Zapnout podrobný výstup. |
| `--restart` | Vynutit restart backendu (ukončí všechny spuštěné instance `backend_server.py`). |
| `--version` | Zobrazí verzi (`Chloros CLI 1.2.0`). |
| `--help` | Zobrazí nápovědu nejvyšší úrovně. |

### Rejstřík příkazů

| Příkaz | Účel |
| --- | --- |
| [`process`](#chloros-cli-process) | Zpracovat složku záznamů typu „Survey3“ nebo „LATTICE“ od začátku do konce. |
| [`login`](#chloros-cli-login) | Ověřit tento počítač pomocí účtu Chloros+. |
| [`logout`](#chloros-cli-logout) | Vymazat přihlašovací údaje z mezipaměti. |
| [`status`](#chloros-cli-status) | Zobrazit aktuální stav licence / ověření. |
| [`export-status`](#chloros-cli-export-status) | Průběh exportu Live Thread-4 během spuštění `process`. |
| [`language`](#chloros-cli-language) | Nastavení nebo zobrazení seznamu jazyků pro zobrazení příkazového řádku CLI (podporováno 38 jazyků). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Výchozí projektová složka (sdílená s grafickým rozhraním). |
| [`update`](#chloros-cli-update) | Vyhledat a nainstalovat aktualizace CLI (Linux /Jetson). |
| [`selftest`](#chloros-cli-selftest) | Diagnostika systému + základní testy. |
| [`time-sync`](#chloros-cli-time-sync) | Stav a ovládání PTP grandmasteru. |
| [`lattice`](#chloros-cli-lattice) | Ovládání a snímání kamery LATTICE (více než 45 podpříkazů). |
| [`daq`](#chloros-cli-daq) | Ovládání spektrálních senzorů DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Otevření a spuštění uloženého projektu Chloros (kamery + DAQ). |

---

## Instalace

`chloros-cli` je součástí instalačního balíčku pro stolní počítače Chloros na všech podporovaných platformách — není k dispozici samostatný CLI ke stažení. Instalací balíčku pro danou platformu se `chloros-cli` přidá do vašeho `PATH` společně s aplikací pro stolní počítače abinární soubor, který řídí.

Nejnovější soubory ke stažení: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Instalační program také obsahuje praktické spouštěcí skripty (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`), které otevírají připravený shell CLI; jsou popsány v [CLI Uživatelské příručce](../CLI.md) a zde nejsou opakovány.

### Windows (.exe)

1. Stáhněte si instalační program Windows ze stránky pro stahování.
2. Spusťte soubor `Chloros-Setup-x.y.z.exe` a postupujte podle pokynů průvodce. Výchozí instalační cesta je `C:\Program Files\Chloros\` (CLI se umístí do složky `C:\Program Files\Chloros\cli\`, kterou instalační program přidá do proměnné PATH).
3. Otevřete nový terminál (`cmd.exe`, PowerShell nebo terminál systému Windows), aby se načítal aktualizovaný soubor `PATH`.

```powershell
chloros-cli --version
```

Instalační program automaticky přidá `chloros-cli.exe` do vašeho systémového `PATH` a zabalí do něj runtime Arena SDK potřebný pro kamery LATTICE.

### Linux amd64 (.deb)

Pro Ubuntu 22.04 LTS nebo novější / pracovní stanice x86_64 založené na Debianu.

> **Ubuntu 20.04 není podporováno.** Seznam závislostí balíčku je odvozen od toho,
> na co se backend skutečně odkazuje, a to zahrnuje `libc6 (>= 2.34)`;
> focal dodává glibc 2.31. `apt` odmítne instalaci, místo aby ji nechal selhat při
> běhu.

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

Balíček .deb nainstaluje:
- `chloros-cli` do `/usr/bin/chloros-cli`
- Zkompilovaný backend na `/usr/lib/chloros/chloros-backend`
- Runtime Arena SDK (pro kamery LATTICE)
- Modely odšumovače, kalibrační balíčky a konfigurace aktualizačního kanálu

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Stejná struktura jako u balíčku .deb pro amd64, s verzí CUDA vyladěnou pro Jetson Orin / Orin NX / Orin Nano.

### Jednorázové ověření na každém zařízení

Každá platforma vyžaduje jednorázové přihlášení na adrese Chloros+, než začnou fungovat volání SDK / CLI:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Přihlašovací údaje jsou uloženy v souboru `~/.chloros/user_session.json`.

### Ověření instalace

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **Je vyžadováno předplatné Chloros+.**Služba CLI vyžaduje aktivní tarif Chloros+.**Copper**je základní úroveň Chloros+ — každá placená úroveň Chloros+ má přístup k CLI / SDK; pouze bezplatná úroveň**Iron** tento přístup nemá. (Převodník ID tarifů: `0`=Iron/bezplatný, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Upgradujte na [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Tento limit je vynucován na straně backendu, nikoli pouze prostřednictvím CLI: požadavek s příznakem SDK / CLI bez placeného tarifu je odmítnut s kódem `403 PLAN_UPGRADE_REQUIRED`, ať už pochází z `chloros-cli`, Python SDK, nebo z ručně vytvořeného klienta HTTP. Odhlášený volající obdrží místo toho kód `401 AUTH_REQUIRED`. Přístup funguje offline po dobu odkladného období daného tarifu (30 dní u měsíčního tarifu, do vypršení platnosti u ročního tarifu) a po uplynutí této lhůty se zastaví; kód `chloros-cli status` zůstává aktivní, aby byl důvod viditelný (jedná se jedná se o trasu SDK / CLI, která je vyňata z omezení podle tarifu — `GET /api/license-status`).

---

## Pětiminutový rychlý start

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

Zpracujte složku obrázků prostřednictvím kompletního Chloros pracovního postupu (detekce cíle → kalibrace → vinětace → odrazivost → export indexu).

### Přehled

```
chloros-cli process INPUT [OPTIONS]
```

### Polohové argumenty

| Argument | Popis |
| --- | --- |
| `INPUT` | Cesta ke vstupní složce obsahující soubory `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) nebo `.dng`. |

### Běžné volby

| Příznak | Výchozí hodnota | Popis |
| --- | --- | --- |
| `-o, --output PATH` | nová složka s časovým razítkem ve výchozí cestě projektu (`~/Chloros Projects`, pokud není nakonfigurováno jinak) | Projektová složka, která se má vytvořit nebo znovu použít. Pokud složka již obsahuje soubor `project.json`, namísto přepsání se vytvoří sourozenecká složka `_1`/`_2`. |
| `-n, --project-name NAME` | auto (časové razítko) | Název projektu. |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` používá neuronální debayer Chloros+; pomalejší, ale s vyšší kvalitou. |
| `--vignette / --no-vignette` | `--vignette` | Korekce vinětace. |
| `--reflectance / --no-reflectance` | `--reflectance` | Kalibrace odrazivosti (používá panelový terč, pokud je nalezen, kalibraci NIST pro každou sérii u LATTICE). U multispektrálních snímků LATTICE slouží tato volba zároveň jako přepínač **produktu** odrazivosti — viz [Přepínače exportu podle produktu](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | vypnuto | Použít PPK GNSS korekce ze souborů sidecar. |
| `--exposure-pin-1 MODEL` | vypnuto | Zafixovat model „pin-1“ u dvoukamerového zařízení typu „Survey3“ . |
| `--exposure-pin-2 MODEL` | vypnuto | Upevnit model „pin-2“. |
| `--recal-interval SECONDS` | 0 | Vynutit opakované spuštění kalibračních výpočtů každých N sekund záznamu. |
| `--timezone-offset HOURS` | local | Přepsat časové posunutí zakódované ve výstupních metadatech. |
| `--format FORMAT` | `TIFF (16-bit)` | Jedna z hodnot `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | žádné | Vegetace indexy (`NDVI`, `NDRE`, `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …). |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Vynutit vstupní bod zpracování pro soubory LATTICE TIFF (soubory Survey3 .raw nejsou ovlivněny). Také nouzová cesta, která umožňuje, aby nebyl zpracován žádný záznam s příponou **bez formátu raw** vůbec zpracovat – viz [Jak vypadá složka s pořízenými snímky](#jak-vypadá-složka-zachycených-dat). |
| `--debayered / --no-debayered` | zapnuto | Vydá lineární produkt po odstranění Bayerova vzoru (`Debayered_Images`). Viz [Přepínače exportu pro jednotlivé produkty](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | zapnuto | Vydává náhled pro zobrazení (`Preview_Images`): RGB = vyvážení bílé (DAQ-zdroj světla, je-li k dispozici, jinak šedý svět) + gama; multispec = roztažení do falešných barev. |
| `--radiance / --no-radiance` | zapnuto | Vysílá radianci typu float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referenční hodnota pro produkt odrazivosti LATTICE: `auto` = cíl v rámci snímku, který prošel kontrolou kvality (QA), je absolutní referencí, záložní hodnota při poklesu(ρ = π·L/E); `target` = přísný (bez nahrazení DAQ); `daq` = autoritativní DAQ. Viz [Přepínače exportu pro jednotlivé produkty](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | žádné | Adresář **naměřených** skenů odrazivosti cíle pro jednotlivé jednotky (`<serial>.csv`); v případě selhání se použije nominální spektra T3/T4P. |
| `--array-alignment / --no-array-alignment` | zapnuto | Pole LATTICE: aplikovat zarovnání modul k modulu, které je zaznamenáno v souboru XMP každého snímku (`Chloros:Alignment*`), na každý zpracovaný produkt (odbayering / náhled / radiance / odrazivost / index). Pro snímky bez těchto značek se neprovádí žádná operace. |
| `--array-alignment-crop / --no-array-alignment-crop` | oříznutí | Ořízněte zarovnané výstupy na oblast společného překryvu pole tak, aby všechny moduly sdílely jednu stopu; `--no-…` zachovává plnou plochu snímače (černé vyplnění mimo zdroj). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Převzorkování pro deformaci při zarovnání. `nearest` zachovává přesné zdrojové dynamické rozsahy (DN) (žádné míchání radiometrických hodnot mezi pixely). |

### Možnosti detekce cíle

| Příznak | Popis |
| --- | --- |
| `--min-target-size PIXELS` | Minimální velikost panelového cíle (px) pro detektor. |
| `--target-clustering 0-100` | Citlivost shlukování. |
| `--target / --targets` | Zpracovat vstupní složku jako pouze panel cíle (přeskočit detekci průzkumu). |

### Příklady

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### Přepínače exportu pro jednotlivé produkty (multispektrální LATTICE)

Zpracování LATTICE se rozšiřuje na **všechny příslušné produkty v jednom průchodu**. Čtyři přepínače pro jednotlivé typy — `--debayered`, `--preview`, `--radiance`, `--reflectance` — jsou**ve výchozím nastavení zapnuté**; k vypnutí některého z nich použijte formulář `--no-<type>`. Hlavní kamery RGB vždy vysílají pouze data po odstranění bayerského filtru + náhled (žádná radiance/reflektance pro jednotlivé pásma), takže `--radiance`/`--reflectance` pro ně nemají žádný účinek. Přepínače jsou ignorovány pro Survey3 `.raw` (který se řídí standardní cestou odrazivosti/cíle). *(Starý příznak `--radiometric-output {reflectance,radiance,sensor-response}` byl **odstraněn** a nahrazen těmito přepínači; úroveň `sensor-response` již neexistuje.)*

| Produkt | Výstup | Je zapotřebí DAQ pro sestupné záření? |
| --- | --- | --- |
| `--debayered` | Lineární demosaikování (`Debayered_Images`). | Ne. |
| `--preview` | Náhled na displeji (`Preview_Images`): RGB = WB + gama; multispec = roztažení do falešných barev. | Ne. |
| `--radiance` | float32 W/m²/sr/nm z úplného radiometrického řetězce (`Radiance_Images`). | Č. |
| `--reflectance` | uint16 odrazivost ρ (`32768` = 1,0), připraveno pro Pix4D. | **Ano**, pokud není ukotveno cílem v rámci snímku, který prošel kontrolou kvality (viz níže). |

`--reflectance-source` volí referenci odrazivosti:**`auto`**(výchozí) stanoví cíl v rámci snímku, který prošel kontrolou kvality, jako**absolutní referenci**— empirické řetězce ukotvené v cíli jsou křížově porovnávány na vyřazených panelech a použije se naměřený vítěz — v případě absence cíle nebo neúspěchu kontroly kvality se přejde na rozdělení podle dat z DAQ (ρ = π·L/E);**`target`**je přísný (bez nahrazení DAQ);**`daq`**upřednostňuje chování řízené DAQ. Geometrie cíle (ArUco / pevný-ROI / pruh) pochází z konfigurace cíle projektu; `--target-reflectance-dir DIR` uchovává**naměřené** skeny na jednotku (`<serial>.csv`), vyhledávané podle sériového čísla/QR jednotky cíle, s nominálními spektry T3/T4P jako záložním řešením.

Cesta odrazivosti DAQ automaticky vyřeší **časově shodné sestupné záření**ze zaznamenaného**`.daq`**(DAQ-U/M/E)**nebo nativního souboru DAQ-M `.csv`**, který se nachází společně se snímky. Pokud není balíček kalibrace pro jednotlivé kamery nebo pro DAQ lokálně uložen v mezipaměti,**pipeline jej při prvním použití automaticky načte z AWS** (vyžaduje jednorázové připojení k internetu; uloží se do mezipaměti pod `~/.chloros/`).

#### Čtení pixelů odrazivosti (Pix4D / Metashape / vaše vlastní skripty)

Odrazivost je uložena jako celé číslo DN a **hodnota DN, která odpovídá ρ = 1,0, závisí na zdrojové kameře**:

| Zdroj | ρ = 1,0 je | Jak zjistit |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (rezerva až do ρ 2,0) | Soubor je označen XMP `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (omezeno na ρ 1,0) | Žádné značky XMP typu `Chloros:*` — právě tato absence *je* signálem. |

**Přečtěte si hodnotu `Chloros:PixelScale` a vydělte jí**, místo abyste předpokládali konstantu. Značka je definována v doméně uint16, takže zůstává `32768` napříč výstupními formáty, které mění měřítko — `TIFF (16-bit)`, `PNG (8-bit)`, `JPG (8-bit)` a `TIFF (32-bit, Percent)` jsou všechny samopopisné (nejprve normalizujte uložený datový typ zpět na uint16: ×257 z 8-bitového, ×65535 z float).

> **Jeden případ záměrně neobsahuje žádné měřítko.** Když je 8bitový zdrojový záznam (BayerRG8) je zapsán jako 8bitovýTIFF, potrubí namísto přepočítání hodnot provede *oříznutí* na rozsah 0..255, takže každá hodnota nad ρ≈0,008 se zploští na 255 a soubor není popsán žádným měřítkem. Chloros záměrně vynechává jak `Chloros:PixelScale`, tak i `MicaSense:RadiometricCalibration` a zaznamenává důvod. **Pokud v souboru odrazivosti LATTICE tento tag chybí, nepředpokládejte žádné měřítko — proveďte opětovný export v 16bitovém nebo 32bitů**, místo abyste dělili pixely, které nikdy dělitelné nebyly.

#### EXIF přenesený do exportu

`process` zkopíruje **GPS blok a jeho ExifIFD** do každého produktu, takže
export obsahuje `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` a
`CameraSerialNumber` spolu s georeferencí.

**`FocalLength` není pro fotogrammetrii volitelný.** Pix4D vypočítává vzdálenost vzorku na zemi z
ohniskové vzdálenosti a nadmořské výšky; pokud tento tag chybí, použije se velmi nepřesné měřítko. Při jednom
letu nad pomerančovým hájem s 49 snímky proměnil chybějící tag lokalitu o rozměrech 411 m × 160 m na rekonstruovanou
lokalitu o rozměrech 47,8 km × 13 km — ortofotomapou o velikosti 455 MP, která obsahovala převážně „nodata“, což se poté interpretovalo jako problém s dlaždicováním a
problém s formátem BigTIFF , ještě než někdo zkontroloval GSD. Pokud váš ortofoto má nepravděpodobné
měřítko, spusťte nejprve na exportovaném produktu program `exiftool -FocalLength`.

Kopie záměrně **není** `-all:all`: strukturální značky IFD0 narušují výstup LATTICE při
kopírování a `ExifImageWidth` / `ExifImageHeight` jsou vyloučeny, protože popisují
*zdrojový* snímek — export, jehož rozměry byly někdy změněny, by jinak obsahoval rozměry
v rozporu s vlastním rastrem. XMP se zapisuje přímo, nikoli kopírováním, protože ExifTool
při kopírování bloku XMP vyřazuje XMP tagy ze stejného volání (což by vedlo ke ztrátě kalibračních tagů MAPIR
).

### Kam se ukládají výstupy

Výstupy se ukládají **do projektové složky, seskupené podle fotoaparátu a poté podle formátu souboru**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Složka fotoaparátu je `LATT-<sensor>-<lens>-F<filter>` pro LATTICE (odpovídá EXIF záznamu
`Model`) a `<model>_<filter>` pro Survey3 — dvě kamery sdílející snímač a filtr, ale lišící se
objektivem, mají oddělené adresáře, protože se liší vinětace, zorné pole a zkreslení. Formát
adresáře navazuje na `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` nebo `tiff32` pro
`TIFF (32-bit, Percent)`.

> **Každý exportovaný produkt si zachovává název souboru.** Export v radiance formátu
> `capture_…_raw.tif` se stále jmenuje `capture_…_raw.tif` — pouze se nachází v
> `tiff32/Radiance_Images/`. **Produkt identifikuje složka, nikoli název souboru**, takže globální vyhledávání
> pro `*radiance*.tif` nic nenajde; místo toho použijte shodu na adresář.

### Záznamy ze světelného senzoru — kalibrované `.daq` + `.csv`

`process` zpracovává také záznamy `.daq` ve vašem vstupním a **nepotřebuje**
k tomu žádné snímky: samostatně letící DAQ-U / DAQ-M / DAQ-E představuje kompletní
zaznamenání, a složka obsahující pouze soubory `.daq` je platným vstupem.

Záznam z DAQ lze pořídit **bez** jeho kalibrace — právě to umožňují veřejně dostupné
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) záznamníky
(`record_daq.py`) dělají ve výchozím nastavení: zapisují surová data z čidel a označují soubor tak, aby
Chloros načítal tovární kalibraci daného čidla**podle sériového čísla** (nejprve z lokální mezipaměti,
poté z cloudu MAPIR) a aplikuje ji. `process` zapíše výsledek zpět:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` obsahuje jeden řádek na jedno měření: časové razítko UTC, integrační čas, celkový výkon,
fotopický/skotopický lux, PPFD (a jeho rozdělení na modrou/zelenou/červenou), špičková vlnová délka a poté
celé spektrum na vlastní vlnové mřížce senzoru. `.daq` se znovu importuje, aniž by bylo
nutné provádět druhou kalibraci.

V případě úspěchu hlásí běh `Light-sensor products written: N (calibrated .daq + .csv)`.
Text v závorce popisuje, co bylo skutečně zapsáno, takže se zobrazí
`(RAW COUNTS — this sensor has no calibration bundle)` pro snímač bez balíčku a
`(N calibrated, M raw counts)` pro složku obsahující obojí. Vlastní nadpisy backendu
`[DAQ-EXPORT]` a `[RUN-SUMMARY]` odvozují své znění stejným způsobem — žádný z
těchto tří nemůže označit surový export kalibrovaný.

Záznam DAQ-U / DAQ-M / DAQ-E, jehož kalibrační balíček nelze načíst – jste
offline nebo daný senzor nemá v souboru žádnou kalibraci – je **přeskočen s uvedením důvodu** na řádku
`[DAQ-EXPORT]` a nikdy není zapsán jako „kalibrovaný“ soubor obsahující surové počty.
Připojte se k internetu a spusťte proces znovu. Důvodem je ten, který čtečka skutečně
zjistila u daného souboru (nečitelná struktura, chybějící balíček, chyba zápisu), a souhrnný
**uvádí jednotlivé** důvody – dvacet souborů přeskočených z jednoho důvodu se vykazuje jako jeden
důvod, nikoli jako dvacet opakování tohoto důvodu.

#### Export záznamů DAQ-A jako surových počtů

Řada **DAQ-A** předchází systému svazků pro jednotlivá sériová čísla a nemá žádný kalibrační svazek,
který by bylo třeba načíst — místo toho se kalibruje v terénu pomocí reflexního terče, což je
důvod, proč žádný svazek nikdy nepotřebovala. Odmítnutí těchto záznamů jim znemožnilo jakýmkoli způsobem
získat jejich čísla, takže se exportují pod **jiným názvem**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Jde o odlišný název souboru, nikoli o příznak uvnitř souboru, protože údaj musí zůstat zachován i při
zasílání e-mailem jako pouhý název. Záhlaví `.csv` uvádí
`raw spectral sensor counts (NOT irradiance)` a upozorňuje, že hodnoty jsou srovnatelné
**v rámci** souboru — což je přesně to, k čemu je využívá kalibrace založená na cíli — a
nikoli napříč senzory. Fotometrické sloupce závislé na výkonu (celkový výkon, fotopický a
skotopický lux, PPFD) jsou zapsány jako **NULL** namísto integrace z počtů, a souhrn
běhu uvádí `RAW COUNTS`, takže „exportované“ údaje v protokolu nelze číst jako ozáření.

Starší záznamy **v1.01 / v1.02** (tyto zapisuje DAQ-A-SD) neobsahují žádné údaje očtení,
pouze čas zápisu souboru. Nástroj pro porovnávání snímků ↔ dopadajícího záření je stále odmítá — porovnání
snímku s časem zápisu by bylo neviditelně nesprávné — ale exportér je načte a
CSV vytiskne `clock=daq_created_on`, takže produkt uvádí, o který čas se jedná používá.

### Poznámky

- `process` automaticky rozpozná, zda je vaše složkSurvey3, LATTICE nebo smíšená.
- Průběh se přenáší přes Server-Sent Events; na stránce CLI se zobrazuje živý průběh pro jednotlivá vlákna (Detekce, Analýza, Zpracování, Export).
- U Linux /JetsCLI zkontroluje swap a může před zpracováním velkých složek zobrazit varování. Debayer s podporou textur také automaticky aplikuje omezení frekvence GPU na Jetsony s nízkou spotřebou (Nano, Orin Nano).
- V případě úspěchu běh nahlásí, kolik obrazových produktů zapsal (`Image products written: N`).

#### Spuštění, při kterém nejsou zapsány žádné obrázky, selže

Pokud jste požadovali výstupy a spuštění nezapsalo **žádný** — pouze `project.json` a
`calibration_data.json` — `process` to považuje za selhání: vypíše
`Processing finished but wrote no image products.` a **ukončí se nenulovou hodnotou**, takže skript to může
detekovat. Zpráva uvádí název projektové složky a obvyklé příčiny:

- vstupní složka nebyla rozpoznána jako snímání (zkontrolujte rozložení a `--input-level`), nebo
- všechny požadované produkty byly přeskočeny jako nepoužitelné pro dané kamery (např. požadavek na
  radianci/odrazivost z kamer typu RGB ).

Spusťte skript znovu s parametrem `--verbose` a zkontrolujte protokol backendu, zda obsahuje řádky `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
které vysvětlují přeskočení u jednotlivých kamer, která by se jinak nedostala do výstupu CLI.

Záměrný běh pouze s metadaty — všechny produkty vypnuté a bez `--indices` — je stále
**úspěšný**, protože prázdný obrazový výstup je v tomto případě správným výsledkem.

Stejně tak i **spuštění pouze se světelnými senzory**: složka se záznamy `.daq` neobsahuje žádné snímky k exportu
z definice a běh se hodnotí podle kalibrovaného `.daq` / `.csv`, které místo toho vytvořil.

---

## `chloros-cli login`

Ověřte tento počítač pomocí účtu Chloros+ v cloudu. Přihlašovací údaje jsou bezpečně uloženy v mezipaměti v souboru `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Příklady

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (odstraněním části hesla nebo duplikováním jeho částí). Při chybě 401 CLI automaticky zkusí přihlášení znovu s připojením `$$`, poté s odduplikovanou polovinou hesla; pokud se opakovaný pokus podaří, přihlásí vás a zobrazí správnou syntaxi s jednoduchými uvozovkami, kterou můžete použít příště.

> **Použití bez grafického rozhraní/prostřednictvím skriptu: absence uložené relace znamená interaktivní výzvu, nikoli rychlou chybu.** Jakýkoli příkaz spouštějící backend (`process`, `status`, `export-status`, `time-sync`, …) spuštěný bez uložené licence/relace přejde před pokračováním do interaktivního příkazového řádku `Email:` / `Password:` na standardním vstupu. Úloha bez obsluhy bez relace v mezipaměti se proto zasekne a bude čekat na vstup — před naplánováním úlohy bez grafického rozhraní spusťte příkaz `chloros-cli login EMAIL PASSWORD` jednou na každém počítači.

---

## `chloros-cli logout`

Vymaže uloženou relaci v mezipaměti a vynutí nové přihlášení při příštím volání.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Zobrazí aktuální úroveň licence (Iron/Copper/Bronze/Silver/Gold), ověřeného uživatele a počet vazeb na zařízení.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Zjišťuje aktuální průběh exportu Thread-4. Lze bezpečně volat **během** spuštění příkazu `process` z jiného shellu.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Nastaví jazyk zobrazení na konzoliCLI (podporováno 38 jazyků, včetně CJK, RTL a indické). Na starších konzolích, které nedokážou vykreslit skript, se plynule přepne na angličtinu.

```
chloros-cli language [LANG_CODE] [--list]
```

### Příklady

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Příkazy pro složku projektu

Tyto příkazy spravují výchozí umístění složky projektu (sdílené s grafickým uživatelským rozhraním).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update`

Linux/ Pouze pro Jetson. Zkontroluje `version_url` z `/etc/chloros/update.conf` a nabídne stažení a instalaci odpovídajícího `.deb`.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Na Linux /Jetson provádí CLI také **automatickou kontrolu aktualizací při každém spuštění** (neblokující, nikdy nezdržuje příkaz): načte `/etc/chloros/update.conf`, uloží výsledek do mezipaměti na 1 hodinu v `~/.chloros/update_cache.json` a vypíše `Update available: vX.Y.Z / Run: chloros-cli update`, pokud existuje novější verze. V případě jakékoli chyby a na Windows se aktualizace tiše přeskočí.

---

## `chloros-cli selftest`

Spustí 7krokový zkušební test: verze, dostupnost portu, spuštění backendu, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), přítomnost modelu odšumovače, připravenost CUDA+odšumovače.

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

Stav a ovládání PTP grandmastera. Hostitel „Chloros“ provozuje PTP grandmastera; kamery LATTICE a jednotky DAQ-E se k němu připojují jako podřízené zařízení pro časová razítka napříč zařízeními.

| Podpříkaz | Popis |
| --- | --- |
| `status` | Zobrazit stav grandmastera, priority BMCA, identitu hodin. |
| `peers` | Vypisovat podřízené jednotky detekované prostřednictvím Delay_Req (kamery + senzory DAQ-E). |
| `cameras` | Stav PTP pro jednotlivé kamery (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Restartovat proces grandmasteru. |
| `set-priority --priority1 N --priority2 N` | Přepsat priority BMCA. |

### Příklady

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

Ovládání kamery LATTICE. Každý podpříkaz prochází přes backend Chloros; tento backend spravuje fond kamer, takže následující volání CLI znovu použijí stejný otevřený popisovač.

### Běžné volby (společné pro většinu podpříkazů)

| Příznak | Popis |
| --- | --- |
| `-d, --device N` | Index kamery (výchozí: 0). |
| `-s, --serial SN` | Konkrétní sériové číslo; přepisuje `--device`. |
| `--serials SN1,SN2,…` | Sériová čísla oddělená čárkami pro provoz s více kamerami. |
| `--all` | Provádět operaci na každé nalezené kameře. |
| `--exposure US` | Doba expozice v mikrosekundách. |
| `--gain DB` | Zisk v dB. |
| `--pixel-format FMT` | např. `BayerRG8`, `BayerRG12`. |
| `--width N` / `--height N` | Rozměry snímku. |
| `--preset {default,high_quality,high_speed,triggered}` | Použít předvolbu nastavení. Všechny fungují v režimu volného běhu kromě `triggered`, který aktivuje kameru na hardwarový signál na lince 2 — pokud tato linka není aktivována, bude kamera čekat donekonečna, místo aby pořizovala snímky. |
| `-o, --output DIR` | Výstupní adresář (výchozí: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Velikost paketu GVSP. `auto` spouští sondy ICMP+GVSP; `jumbo` = 9000; `standard` = 1500. |

### Pracovní postup prvního připojení kamery LATTICE

```bash
# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Auto-shrink frame size to fit the wire
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Přehled podpříkazů

#### Vyhledávání a informace

| Podpříkaz | Účel |
| --- | --- |
| `lattice info` | Seznam připojených kamer (výrobce, model, sériové číslo, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analýza hostitelského systému pro optimální konfiguraci kamery. `--no-discover` skips vyhledávání kamer (rychlejší, analýza pouze pomocí síťové karty). |
| `lattice network [--fix] [--estimate] [--cameras N]` | Kontrola/oprava nastavení síťové karty; odhad šířky pásma/FPS. |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | Stabilní schéma backendu sítě+ doporučení pole (vrací `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `auto_capped_fps` zachovává požadované rozlišení, ale omezuje cílovou hodnotu fps — přečtěte `recommended.recommended_target_fps` a předávejte jej jako cíl připojení; považujte to za úspěch, nikoli za chybu. |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | Analýza „co by, kdyby“ bez spouštění kamer. **`--n-active` je celkový počet kamer v síti, nikoli pouze v tomto poli**— zvyšte tuto hodnotu, pokud souběžně streamují samostatné kamery nebo je kapacita sítě vypočítána na základě požadavku, který je podhodnocuje (výchozí: `len(--models)`). Vždy vytiskne souhrnné řádky `Wire budget:` (požadovaných MB/s vs. strop bez kolizí) a `Max cameras:` a označí `** OVER-SUBSCRIBED**`, pokud pole přetěžuje síť — viz [FPS pole a model burst](#array-fps--burst-model). |
| `lattice gpu` | Zobrazit stav GPU. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Zkontrolovat nebo aktualizovat firmware kamery. Místní volba `.fwa` je zafixována: soubor v `firmware/<MODEL_PREFIX>/`, který odpovídá `MIN_FIRMWARE_VERSION` dané sestavení, je nahrán, pokud je k dispozici (pouze nejvyšší verze jako záložní řešení), takže novější obraz dodavatele připravený na disku zůstává neaktivní, dokud nedojde ke změně tohoto nastavení — novější verze se záměrně dostávají do zařízení prostřednictvím podepsaného manifestu AWS, který je upřednostňován, pokud je novější. |
| `lattice presets [--apply NAME]` | Zobrazit nebo použít předvolby kamery. |
| `lattice status` | Stav kamery v reálném čase. |

#### Snímání

| Podpříkaz | Účel |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | Jeden snímek. **Ve výchozím nastavení ukládá všechny typy exportu** (`--processing all`); viz [Úrovně exportu záznamu](#capture-export-levels-the-all-default). `--levels` uloží explicitní podmnožinu (přepíše `--processing`); `--force-daq` zapíše přiřazenou hodnotu DAQ jako sidecar `.daq` i při snímání pouze v surovém formátu. `--jpeg-quality` = JPEG kvalita 1–100 (výchozí 95). |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | Zaznamenává na disk až do stisknutí Ctrl+C. |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Živý náhled MJPEG v prohlížeči. `--ae-damping` nastavuje tlumení automatické expozice (0,4–100). |

#### Nastavení snímače

| Podpříkaz | Účel |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | Čtení/zápis libovolného uzlu GenICam. |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | Expozice a automatická expozice. |
| `lattice gain [--auto] [--off] [--set DB]` | Zisk a automatický zisk. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | Oblast zájmu snímače a binning. |
| `lattice format [--set FMT] [--list]` | Formát pixelů. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Hardwarová/softwarová spoušť. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (bez příznaků = jednorázové vyvážení bílé) | Operace vyvážení bílé. RGB /Pouze kamery s maticí Bayer; u monochromatických M3M se tato funkce neprovádí (přeskočí se). |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` |RGBní barevný pipeline pro zobrazení. `natural` (výchozí) představuje levné zpracování živého obrazu; `enhanced` přidává odstranění barevných okrajů + vibrance + lokální kontrast CLAHE pro plný vzhled s paritou hubu při přibližně dvojnásobných nákladech na zpracování každého snímku, tedy nižší **živou** snímkovou frekvenci — uložené záznamy vždy dostanou plnou úpravu v obou případech. RGB /Pouze kamery s filtrem Bayer; u mono M3M se přeskočí. |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | Zobrazení sytosti/kontrastu (kamery s filtrem RGB). U mono M3M se přeskočí. |
| `lattice filter [--set NAME] [--list]` | Nastavení modelu filtru kamery (`RGN-IMX265`, `OCN`, `NGB`, …). |
| `lattice power [--sleep]` | Napájecí/tepelné uzly sondy; přepnutí do úsporného režimu. |

#### Kalibrace a senzory

| Podpříkaz | Účel |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibrace pomocí reflektančního terče. |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | Příkazy pro vestavěný senzor dopadajícího světla. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Aplikovat korekci vinětace na existující snímky. |

#### Více kamer (přechodné relace)

| Podpříkaz | Účel |
| --- | --- |
| `lattice multi-info` | Vypíše všechny kamery s synchronizačními rolemi. |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | Jeden synchronizovaný snímek z každé kamery. **Ve výchozím nastavení**ukládá všechny typy exportu, pokud je připojeno trvalé pole; dočasná náhrada bez pole je**pouze bez debayeringu** (pro zbytek nejprve spusťte `array-connect`). |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | Streamování synchronizovaných snímků (dočasné). |
| `lattice multi-test [--count N]` | Test časování synchronizace GPIO. |
| `lattice multi-detect [--line LINE] [--json]` | Automatická detekce zapojení GPIO zapojení master/slave. |

#### Zarovnání

| Podpříkaz | Účel |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — plus ovládací prvky detektoru/porovnávače `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, ovládací prvky RANSAC `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, kombinace vícesnímků `[--averaging mean\|median\|inlier_weighted]`, geometrická omezení `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, prostorová omezení `[--roi X0,Y0,X1,Y1] [--mask PATH]` a přepsání nastavení pro jednotlivé slave `[--per-cam-override SN:KEY=VALUE]` () | Vypočítá profil zarovnání z živých kamer. `--prefilter` je ve výchozím nastavení nastaven na `gradient` (mapa hran; odpovídá zarovnávači v grafickém rozhraní/pole — hrany přetrvávají napříč spektrálními pásmy). `--matcher flann` se vyplatí při více než ~5000 prvcích; `--averaging median` je odolný vůči jednomu vadnému snímku, `inlier_weighted` váží podle počtu shod; `--lock-scale` promítá na nejbližší rotaci (bez měřítka), `--lock-axis` vynuluje jednu složku posunu; `--mask` se vztahuje na všechny kamery (pro nastavení pro jednotlivé kamery použijte `--per-cam-override`, např. `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` odmítne uložit kalibraci, jejíž RMS reprojekce překračuje mezní hodnotu. |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | Zachytí jeden zarovnaný vícepásmový snímek. `--bit-depth` se ve výchozím nastavení přizpůsobí kameře; `--no-crop` zachová celý snímek (vyplňuje černou barvou); `--interpolation` (výchozí `linear`) a `--border-mode`/`--border-value` (výchozí `constant`/0) řídí deformaci na straně CPU — cesta na straně GPU je v každém případě bilineární. |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | Vícepásmové snímky zarovnané podle datového proudu (stejné ovládací prvky pro warp jako u `align-apply`). |
| `lattice align-info --profile PATH [--json]` | Zobrazit podrobnosti profilu. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Změnit pořadí vrstev. |

#### Index / Matematika vegetace

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

Kompletní sada příznaků: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI / NDRE / EVI / SAVI / GNDVI /…), `--formula EXPR`, `--channel SYM=BAND` (opakovatelné), `--capture-level raw|debayered|radiance|reflectance|unknown` (přepíše úroveň snímání zaznamenanou ve zdrojovém souboru TIFF; výchozí nastavení: načteno z metadat souboru TIFF), `--output PATH`, `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. U `--live` se rovněž uplatňují ovladače pro vyrovnání: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`.

> **`--channel` – u symbolů se rozlišují velká a malá písmena.** Symboly musí přesně odpovídat názvům kanálů v předvolbě (předvolby používají malá písmena, např. NDVI = `red`,`nir` — zkontrolujte `--list-presets`), a část s označením pásma se musí shodovat s názvem pásma v zarovnaném zásobníku (nebo být indexem pásma počínajícím od 0 v offline režimu). `--channel red=Red_660 --channel nir=NIR_850` funguje; `--channel RED=660` selže s chybou `channel_map missing entries`.

#### Trvalá připojení (Smart-Prep, postup ekvivalentní grafickému rozhraní)

Tyto příkazy udržují kamery otevřené v backendovém fondu napříč voláními `CLI`.

| Podpříkaz | Účel |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Přidat jednu kameru do fondu (jedna kamera, bez pole). |
| `lattice cam-disconnect [--serial SN] [--all]` | Uvolnit. |
| `lattice cam-list` | Vypisuje kamery v fondu. |
| **`lattice array-connect`**|**Připojení trvalého synchronizovaného pole (DOPORUČENÝ vstupní bod).** Spustí kompletní proces inteligentní přípravy v grafickém uživatelském rozhraní. |
| `lattice array-disconnect [--array-id ID] [--all]` | Uvolnit pole. |
| `lattice array-list` | Zobrazit seznam připojených polí. |
| `lattice array-status [--array-id ID]` | Živé snímky za sekundu (fps), PTP, poslední chyba. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Jedno synchronizované zachycení z živého pole — Jednorázové / Kontinuální / Intervalové / Nejrychlejší. **Výchozí hodnota je `all`** (jeden soubor pro každý příslušný typ exportu na každou kameru). Přeskočené kamery (např. RGB vyloučené z měření radiance/reflektance) jsou vykazovány pomocí `Skipped: SN:<serial> (<reason>)`; hodnota DAQ použitá pro reflektanci je uložena společně s nimi a vykazována pomocí `DAQ: <path>`. Viz [Režimy snímání, záznamníky a offline zpracování](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | Záznam živého zobrazení kombinovaného indexu do videa/GIF (pro monitorovací účely; vyžaduje otevřený kombinovaný datový proud). |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | Série surových snímků Bayer s vysokou snímkovou frekvencí (pro analytické účely; offline zpracování). |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Zpracovat uloženou sérii snímků ve formátu raw do kalibrovaného videa (videí). |

##### Možnosti `array-connect`

| Příznak | Výchozí | Popis |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automaticky vyhledat všechny kamery LATTICE (vyžaduje ≥2) | První v pořadí je MASTER. Je-li vynecháno, se vyhledávání omezí na modely LATTICE (`TRI032*`) a všechny se připojí. |
| `--line {Line0,Line2,Line3}` | `Line2` | Synchronizační linka GPIO. |
| `--target-fps F` | auto | Frekvence spouštění Masteru. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Přepsat volbu úrovně. |
| `--wire-ceiling-mbps MB_PER_S` | automaticky detekováno | **Hostitelůvtrvalý rozpočet přenosové šířky v MB/s — hodnota, na které závisí alokace celého pole.** Snižte ji, pokud pole hlásí rámce poškozené GVSP: automatická hodnota je odvozena z inzerované rychlosti připojení síťové karty, která nadhodnocuje USB adaptéry, úzké PCIe linky a vytížené sdílené sítě. Ukládá se v bloku zachycení pole projektu, takže se obnoví po opětovném otevření / CLI / SDK. Viz [Stav pole](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | auto | Hardwarové seskupování. |
| `--no-recommend` | off | Přeskočit krok analýzy sítě. |
| `--no-ptp` | vypnuto | Zakázat PTP (časová razítka mezi kamerami pak **nejsou** srovnatelná). |

### Smart-AE / Smart-Capture

Pole LATTICE spouštějí nepřetržité automatické ostření (AE) na pozadí, jakmile jsou připojena, ale u nově zaostřené scény trvá chvíli, než se obraz ustálí. `array-capture --smart` je **praktické řešení**: počká, až se automatická expozice ustálí na všech kamerách v poli, a teprve poté spustí snímání. Použijte jej, když měníte scénu uprostřed relace.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Zásady ustálení jsou ve výchozím nastavení konzervativní: časový limit 5 s, stabilizační okno 1,5 s, tolerance rozptylu expozice ±5 %. Pokud potřebujete od automatizace odlišné chování, proveďte ladění pomocí nástroje „SDK“ (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`).

### Úrovně exportu snímků (výchozí nastavení `all`)

Od této verze mají `lattice capture`, `lattice multi-capture` a `lattice array-capture` **výchozí nastavení `--processing all`** — jeden uložený soubor pro každý typ exportu, který se vztahuje na každou kameru, což odpovídá chování funkce „Zachytit vše“ v grafickém uživatelském rozhraní. Úrovně jsou:

| Úroveň | Výstup | Vztahuje se na |
| --- | --- | --- |
| `raw` | Jednokanálový Bayer (černobílé kamery: jeden pásmo) přímo ze snímače. | Všechny kamery. |
| `debayered` | 3kanálová demosaika BGR (černobílé kamery: 1kanálová šedá stupnice). | Všechny kamery. |
| `radiance` | float32 W/m²/sr/nm přes celý radiometrický řetězec. | Pouze multispektrální (M3C/M3M) pouze — **přeskočeno u kamer s filtrem „RGB“**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), připraveno pro Pix4D. | Pouze multispektrální, a **pouze pokud je připojen DAQ a kamera je kalibrována**; jinak vynecháno. |
| `preview` / `display` | Plný řetězec náhledu v grafickém rozhraní (CCM + WB + gama podle profilu kamery). `lattice capture` pojmenovává tento řetězec jako `preview`; `array-capture`/`multi-capture` použijte `display`. | Všechny kamery. |

Zadejte jednu úroveň, chcete-li uložit pouze tu jednu (`--processing debayered`). Pokud zadáte `all`, úrovně, které sese nevztahují na danou kameru, jsou přeskočeny (a nahlášeny), nevzniká chyba — nepřipojená nebo nekalibrovaná kamera stále obdrží `raw` / `debayered` / `preview`.

U každého snímku odrazivosti je skutečně použitá hodnota odrazivosti směrem dolů z DAQ zapsána do **`.daq`** sidecaru vedle snímku (aby bylo možné záznam později znovu zpracovat) a vykazována v řádku `DAQ:`.

### Jak vypadá složka se snímky

Každý typ exportu je uložen ve své **vlastní podsložce** pod `-o`, takže u víceúrovňového snímku se typy nikdy nemíchají:

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>` je časové razítko snímku a `<serial>` sériové číslo kamery, takže jedna synchronizovaná skupina sdílí
časové razítko napříč kamerami. **Všimněte si jedné nesrovnalosti:** úroveň `display` je uložena ve složce
s názvem `preview/`, zatímco samotné soubory si v názvu zachovávají `_display` — složka a přípona se liší
pouze pro tuto úroveň. Neznámé úrovně se ukládají do složky s vlastním názvem a pokud nelze podsložku
nelze vytvořit, soubor se zapíše do kořenového adresáře výstupu, místo aby byl ztracen.

**Opětovné zpracování složky snímků:**nasměrujte `chloros-cli process` na**kořenový adresář snímků**
(`output/`). `process` obvykle importuje pouze složku, kterou určíte, ale pokud tato složka neobsahuje žádné
obrázky a má podsložky, prohledá je automaticky — takže podsložky na úrovni kořenového adresáře i
kořenový adresář `.daq` jsou načteny najednou. Každá úroveň snímku se importuje jako jeden obrázek, přičemž
další úrovně jsou k dispozici jako režimy, nikoli jako jeden obrázek na úroveň.

Přímo pojmenovat **podadresář úrovně** (např. `output/raw/`) také funguje. Tímto způsobem zůstane kořenový
adresář `.daq` opomenut, proto zkopírujte nebo nasměrujte údaje z DAQ vedle něj, když znovu odvozujete radiometrický
produkt z `raw/` — jinak nebude mít shoda časových značek k čemu se přiřadit.

**Zpracování vždy začíná od `raw`.** V rámci každého záznamu je zdrojem pro zpracování surový snímek;
`debayered`, `radiance`, `reflectance` a `preview` se objevují jako zobrazitelné režimy, ale nikdy nejsou
vraceny zpět do zpracování. Opětovné zpracování odvozeného produktu by znovu aplikovalo vinětaci, CCM a
výpočty radiance , které jsou již zapracovány do jeho pixelů, a proto Chloros tento postup odmítá, místo aby
prováděl dvojí zpracování. Dvě důsledky, o kterých je dobré vědět:

- Rendery `index/` a `composite/` nejsou **nikdy** zpracovány. Jedná se o výstupy, nikoli o snímky —
  render s LUT „NDVI“ nemá žádnou smysluplnou interpretaci radiance.
- Složka „captures“ exportovaná **bez** `raw` (např. `array-capture --processing reflectance`) nemá
  žádný legitimní zdroj pro pipeline. Tyto záznamy se importují a zobrazují normálně, ale `process` je
  a hlásí to:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Pokud skutečně potřebujete prostrčit odvozený produkt – relaci hubu zachycenou s
  zapnutým `demosaic` nebo starší složku – `--input-level {raw,debayered,processed}` vynutí vstupní
  bod a přepsá přeskočení. Tento příznak je záměrným únikovým východem; `auto` (výchozí nastavení)
  nikdy nezpracuje záznam, který neobsahuje surová data.

### Přeskočené záznamy ve smíšených soustavách filtrů

Pokud v jednom poli kombinujete kamery typu „RGB“ a multispektrální kamery, `array-capture --processing radiance` (nebo `reflectance`) uloží multispektrální snímky a **přeskočí** snímky z kamer typu „RGB“ — radiance na jeden Bayerův čtvereček nemá pro širokopásmový snímač smysl. Nástroj CLI výslovně vypíše každý uložený soubor (s úrovní exportu), každý zapsaný záznam `.daq` a každé přeskočení, takže počet souborů není překvapivý:

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Tokeny důvodů přeskočení mají formát `<level>-not-applicable-to-rgb-cam`. Odrazivost může být také přeskočena pomocí `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)` a pomocí `dls-uncalibrated-band-<nm>`, pokud pásmo leží převážně mimo radiometricky kalibrovaný rozsah světelného senzoru DAQ (~374–974 nm) — z dostupných modelů pouze F988, jehož podporovanou cestou je pracovní postup s panelem odrazivosti.

Použijte `--processing debayered` (nebo `display`) pro zahrnutí všech kamer bez ohledu na typ filtru, nebo výchozí `all` pro získání všech příslušných úrovní pro každou kameru najednou.

---

## Režimy snímání, záznamníky a offline zpracování

Všechny tyto funkce pracují s **trvalým polem** (nejprve spusťte `array-connect`). Odrážejí panel snímání v grafickém uživatelském rozhraní.

### Režimy `array-capture`

`array-capture` je jediný příkaz se čtyřmi režimy spouště a sadou přepínačů pro export:

| Režim | Příznak | Chování |
| --- | --- | --- |
| **Jednorázový** *(výchozí)* | (žádný) | Jedna synchronizovaná skupina snímků, poté ukončení. |
| **Kontinuální** | `--continuous` | Po sobě jdoucí průchody až do `Ctrl+C`, `--count N`, nebo `--duration S`. |
| **Interval** | `--interval S` | Jeden průchod každých `S` sekund (měřeno od začátku každého průchodu), stejné meze. |
| **Nejrychlejší** | `--fastest` | Pouze surová+ přiřazená hodnota z DAQ + kompozit kombinovaného indexu; vynechává výpočty radiance/reflektance/zobrazení, aby se snímek zobrazil rychle. Implikuje `--processing raw --force-daq`. Uložená data `.daq` později znovu zpracujte na kalibrované produkty. |

Přepínače exportu (lze kombinovat s libovolným režimem; všechny sdílejí koncový bod GUI/ SDK):

| Příznak | Účinek |
| --- | --- |
| `--processing LEVEL` | Jedna úroveň exportu, nebo `all` (výchozí). |
| `--levels L1,L2,…` | Explicitní podmnožina typů exportu (např. `raw,radiance,reflectance`); **přepíše `--processing`**. |
| `--aligned` / `--no-aligned` | Provést warp u všech ne-surových exportů členů podle [profilu zarovnání](#alignment) pole (spoluregistrované). Surová data zůstávají bez transformace, ale v metadatech nesou informaci o transformaci. Pokud pole nemá žádný profil, použije se výchozí nastavení bez zarovnání (s varováním). |
| `--index` / `--no-index` | Uložit / přeskočit překryv indexu vegetace pro jednotlivé kamery, pokud je nakonfigurován. Výchozí nastavení: vykreslit. |
| `--force-daq` | Uložit přiřazené hodnoty DAQ/DLS jako sidecar `.daq`, i když to žádná zvolená úroveň nevyžaduje (např. snímání pouze v surovém formátu), aby bylo možné snímky offline znovu zpracovat na odrazivost/index. |
| `--smart` | Před spuštěním počkat, až se AE ustálí na všech kamerách (viz [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | Komprese pixelůTIFF. `deflate` (výchozí) = bezztrátová zlib L1 + horizontální prediktor, ~4,1 MB na snímek v plném rozlišení; `none` = nekomprimované, ~5× rychlejší zápis při ~6,3 MB na snímek — použijte pro maximální trvalou rychlost, pokud to disk umožňuje. Obě varianty jsou bezztrátové a při importu se čtou identicky. |

> **Jednorázovýzápis TIFF + model s trvalou rychlostí.**Snímky se zapisují v**jednom**průchodu do souboru TIFF, který obsahuje pixely + XMP + IFD0 Značka/Model (měřeno na Mono12 v plném rozlišení: 36 ms komprimováno / 6,5 ms nekomprimováno, oproti ~148 ms u starého způsobu „zapsat a poté přepsat pomocí ExifTool“); jediná zbývající práce ExifToolu (dolaďování EXIF sub-IFD) běží v asynchronním pozadí a snímek je kompletní a připravený k importu, i když se tato úloha nikdy nespustí. Poznámka: Komprese DEFLATE drží GIL Python, takže komprimované zápisy se**ne**paralelizují napříč vlákny zapisovače pro jednotlivé kamery — trvalé snímání v plném rozlišení z 8 kamer při rychlosti snímače (~10,4 fps) vyžaduje `--compression none`**a** disk třídy NVMe (~500 MB/s trvalého zápisu). Stejný ovladač je k dispozici jako `compression` na `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — video/GIF s kombinovaným indexem (pro monitorovací účely)

Zaznamenává vše, co zobrazuje **živý náhled s kombinovaným indexem**, na `.avi` (a volitelně na `.gif`). Jelikož využívá živý kompozitní signál, musí být kombinovaný proud otevřený (např. pole se zobrazuje v náhledu v grafickém uživatelském rozhraní), aby se snímky mohly ukládat. Každé 2 s zjišťuje stav a zastaví se na souborech `--duration`, `Ctrl+C`, nebo když se záznamník sám ukončí.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| Příznak | Výchozí | Popis |
| --- | --- | --- |
| `--array-id ID` | pouze pole | Cílové pole (vynechat, pokud je připojeno pouze jedno). |
| `-o, --output DIR` | `output` | Výstupní adresář (lokální pro backend). |
| `--fps F` | `10` | Snímková frekvence záznamu. |
| `--duration S` | do stisknutí Ctrl+C | Automatické zastavení po `S` sekund. |
| `--gif` | vypnuto | Zapsat také animovaný GIF. |
| `--gif-only` | vypnuto | Zapsat pouze GIF (bez `.avi`). |

### `array-burst` — sériové snímání v formátu raw-Bayer s vysokou snímkovou frekvencí (pro analytické účely)

Čte přímo synchronizovaný skupinový buffer snímací smyčky — **bez kalibračního řetězce, žádný exiftool, není potřeba živý náhled** — takže běží při plné snímací rychlosti fotoaparátu. Zapisuje surové snímky + manifest pro každý snímek + jeden `.daq` pro každé odlišné čtení DLS v rámci `<output>/bursts/<base>/`. Zpracujte offline (další příkaz), nebo předávejte `--build`, aby se to provedlo okamžitě po zastavení.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Příznak | Výchozí | Popis |
| --- | --- | --- |
| `--array-id ID` | pouze pole | Cílové pole. |
| `-o, --output DIR` | `output` | Výstupní adresář (výstup se ukládá do `<DIR>/bursts/<base>/`). |
| `--duration S` | až do stisknutí Ctrl+C | Automatické zastavení po `S` sekundách. |
| `--max-frames N` | neomezeno | Automatické zastavení po `N` surových snímcích. |
| `--build` | vypnuto | Po zastavení okamžitě znovu zpracovat sérii (stejně jako `array-build-video`). |
| `--products …` | `combined:index` | S `--build`: které video(a) sestavit (viz níže). |
| `--fps F` | `10` | S `--build`: počet snímků za sekundu (fps) výstupního videa. |
| `--save-tiffs` | vypnuto | S `--build`: ukládat takésnímku kalibrované soubory TIFF. |
| `--gif` | vypnuto | S `--build`: také zapisovat animované soubory GIF. |

### `array-build-video` — offline přepracovat uloženou sérii snímků

Časově přiřadí každý surový snímek k nejbližšímu uloženému `.daq` měření a prostrčí jej **stejným řetězcem radiance / odrazivosti / index** a vykreslí jedno nebo více videí.

`--products` je seznam položek `kind:level` oddělených čárkami, kde `kind` ∈ `per_cam` | `combined` a `level` ∈ `radiance` | `reflectance` | `index`. Samotný `level` (bez `kind:`) má výchozí hodnotu `per_cam`. Výchozí hodnota je `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Příznak | Výchozí hodnota | Popis |
| --- | --- | --- |
| `--burst-dir DIR` | (povinné) | Cesta ke složce burst (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | Seznam `kind:level`, viz výše. |
| `--fps F` | `10` | Počet snímků za sekundu (fps) výstupního videa. |
| `--save-tiffs` | vypnuto | Ukládat také kalibrované soubory TIFF pro každý snímek společně s videem. |
| `--gif` | vypnuto | Zároveň zapisovat animované soubory GIF. |

> **Vyberte správný záznamník.** `array-record` je *monitorovací*pro monitorování* — zaznamenává živý kompozitní obraz tak, jak se zobrazuje, a vyžaduje otevřený datový proud. `array-burst` → `array-build-video` je *pro analýzu* — ukládá surová data ze snímače v plné rychlosti a následně rekonstruuje kalibrovaná videa s jasem, odrazivostí a indexem, aniž by vyžadoval živý náhled.

### Mono (M3M) jednopásmové kamery

Řada **M3M**je monochromatickou obdobou řady Bayer**M3C**: jedna úzkopásmová interferenční filtrační mřížka na kameru (`M3M-<lens>-F<wavelength>`, např. `M3M-L87-F685`), takže snímač poskytuje**jediný pásmo v odstínech šedé** bez Bayerovy mozaiky. Není co demosaikovat, není třeba odstraňovat přeslech mezi kanály a není třeba nastavovat vyvážení bílé — celý proces zpracování barev od snímače po displej (RGB) se zde jednoduše neuplatňuje.

Co to znamená u modelu CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**detekují černobílou kameru a**přeskočí to s jednořádkovou zprávou** namísto použití nesmyslných nastavení. V téže relaci stále fungují normálně s kamerami typu „RGB“ / „Bayer M3C“.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** stále fungují — radiance a odrazivost jsou *pro jednotlivé pásma* a jsou pro jedno pásmo dokonale definovány. Monochromatické snímky obsahují **identickou** matici odezvy snímače (bez 3×3 unmixu), takže rovina prochází kalibračními výpočty beze změny.
- **Jedna monochromatická kamera nemůže vygenerovat vegetační index.**NDVI / NDRE /atd. vyžadují alespoň dvě pásma (např.Red + NIR). Chcete-li získat index z monochromatického hardwaru, nasměrujte**několik** kamer M3M na různé vlnové délky, sjednoťte je do jednoho vícepásmového souboru a vypočítejte index *z toho*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` symboly musí **přesně** odpovídat názvům kanálů v předvolbě (rozlišuje se velká a malá písmena; NDVI&#x27;jsou malá písmena `red`,`nir` — viz `--list-presets`), a název pásma označuje konkrétní pásmo ve srovnaném stohu (režim offline také akceptuje indexy pásem počínající od 0, např. `--channel red=0 --channel nir=1`).

Rozlišovacím znakem v celém zásobníku je token `M3M` v řetězci modelu (nikdy se neobjevuje v řetězci `M3C`), který se v grafickém rozhraní (GUI) / SDK zobrazuje jako `is_mono`.

---

## Nastavení a ladění síťové karty hostitele (pole LATTICE)

Kamery LATTICE streamují GVSP přes ethernetový adaptér hostitele, takže u polí s více kamerami jsou **ovladač**adaptéru a**velikost přijímacího kruhu** jsou stejně důležité jako přenosová rychlost. Nesprávná nastavení se projeví jako brána `FRAMES WILL DROP` / `Reduce ROI to enable` v panelu Nastavení pole (a v `lattice network-analysis` / v souboru „SDK“`analyze_array_network()`), a to i v případě, že samotné kamery fungují správně.

### USB 10GbE adaptéry — Realtek RTL8157 („Realtek USB 10GbE Family Controller“)

| Položka | Požadovaná hodnota | Proč je to důležité |
| --- | --- | --- |
| **Verze ovladače**|**≥ v10.67 (leden 2026)**, INF `rtump64x64sta.inf` | Starší ovladač z roku**2016**(v10.65, `rtump64x64.inf`) nesprávně zpracovává vypínání a vyvolává chyby**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**při vypínání/restartu/uspání. Přechod se zasekne (časový limit ~5 minut), uživatel provede nouzové vypnutí a opakovaná nečistá vypnutí**poškodí úložiště WMI**(PowerShell a nástroje začnou selhávat s chybou `Invalid class`) a**zablokují USB stack** při dalším spuštění (síťová karta se neaktivuje; USB zařízení přestanou být rozpoznávána). Před provedením čistých restartů si stáhněte aktualizaci z realtek.com (nebo od výrobce hardwarového klíče). |
| **Příjmové vyrovnávací paměti**— klíčové slovo `ReceiveBufferLen` |**256**(maximum ovladače) | Příjmový kruh síťové karty. Výchozí hodnota ovladače**32**ponechává pouze ~0,26 MB použitelného kruhu — což je pro burst z více kamer příliš málo — proto panel pole hlásí `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` a blokuje připojení. Při hodnotě**256**je kruh velký (**~13,5 MB naměřeno na laboratorním hostiteli s 10GbE**), což dává přijímacímu potrubí skutečnou rezervu pro sériové přenosy z více kamer v režimu GVSP. (O tom, zda daná konfigurace skutečně *naváže spojení*, rozhodují dvě kontroly — kontrola připustitelnosti **s ohledem na vyprázdnění**a kontrola**agregovaného nadměrného předplatného** — nejedná se o pouhé srovnání burstu s prstencem; viz [Model fps a burstů pole](#array-fps--burst-model).) |
| **Příjmové URB**— klíčové slovo `PendingReceives` |**64** (max.) | Bloky USB požadavků v přenosu; zvyšujte spolu s přijímacími vyrovnávacími paměťmi pro absorpci burstů. |
| **Jumbo rámec** — klíčové slovo `*JumboPacket` | **9014** | Potřebné pro 9000-bajtové pakety GVSP (6× méně paketů na rámec než u 1500). |

> ⚠️ **Aktualizace ovladače síťové karty VYNULUJE tyto pokročilé vlastnosti na výchozí hodnoty.**Po aktualizaci nebo výměně ovladače adaptéru**znovu použijte** `ReceiveBufferLen=256` a `PendingReceives=64`, jinak se panel pole opět zablokuje, i když se „na hardwaru nic nezměnilo“. To je #příčina toho, že zařízení, které dříve fungovalo, se najednou odmítá připojit.

Použijte v **právo s oprávněním správce** v PowerShellu (nahraďte název svého adaptéru, např. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` se vztahuje na adaptéry USB 10GbE.** Nyní rozpozná typ adaptéru a nastaví správné klíčové slovo pro přijímací kruh: `*ReceiveBuffers`→2048 pro PCIe síťové karty (Intel I219, atd.) nebo `ReceiveBufferLen`→256 + `PendingReceives`→64 pro **USB** 10GbE řadič Realtek (který nezveřejňuje `*ReceiveBuffers`). Cílové hodnoty jsou omezeny na maximální hodnotu hlášenou jednotlivými ovladači (`NumericParameterMaxValue`), takže se nikdy nezapíše hodnota mimo rozsah. Spusťte příkaz z terminálu s **právo**; stejně jako u jakéhokoli ladění založeného na registru, změna se projeví až po restartu adaptéru nebo restartu počítače. Ruční příkazy `Set-NetAdapterAdvancedProperty` uvedené výše zůstávají dobrou alternativou — uplatní se okamžitě (znovu naváží adaptér) bez nutnosti restartu.

### Základy sítě (všechny odkazy LATTICE)

- **Adresování:** link-local `169.254.0.0/16` (GigE Vision LLA). Hostitel má statickou adresu `169.254.x.x/16`; kamery a DAQ-E si samy přiřazují adresy ve stejném rozsahu. Není potřeba DHCP ani brána.
- **Velikost paketu:**upřednostňujte jumbo (9000), ale nechte ji určit automatickým testováním — při každém připojení se provádí nové měření a pomocí testu GVSP již překračuje limit 1500 bajtů pro ICMP u kamery, takže se nastaví na jumbo tam, kde to kabel skutečně zvládne. Použijte pin s adresou `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` pouze tehdy, pokud víte více než testovací funkce, a upřednostňujte nastavení pro jednotlivé příkazy před trvalým: pevné nastavení obejde testovací funkci, takže pokud trasa ve skutečnosti nedokáže přenášet 9000,**každý** záznam vyprší s `SC_ERR_TIMEOUT -1011` (viz [Proměnné prostředí](#environment-variables)).
- **Velikost kruhu RX se mění s `ReceiveBufferLen`:**při výchozí hodnotě `32` je použitelný kruh ~0,26 MB (příliš malý pro jakoukoli sérii snímků z více kamer); při maximální hodnotě `256` je velký (~13,5 MB naměřeno na laboratorním hostiteli s 10GbE), což poskytuje skutečnou rezervu. O tom, zda se konfigurace připojí, pak rozhoduje kontrola připustnosti zohledňující odběr**a** následující kontrolou agregovaného přetížení — nikoli prostým porovnáním datového toku s kapacitou prstence.

### Model snímkové frekvence a datového toku pole

Jak číst panel Nastavení pole (a `lattice analyze-array` / `analyze_array_network` v SDK):

- **Burst se sčítá pro každou kameru zvlášť v reálném formátu pixelů dané kamery.**Mono**M3M**kamery streamují**Mono12 (2 B/px)**;**M3C**kamery s maticí Bayer vysílají 8- nebo 12bitový signál (TRI032S tiše vysílá BayerRG12, i když je požadován BayerRG8). Snímek v plném rozlišení ze 4 kamer tedy má velikost**~12,6 MB, pokud jsou všechny 8bitové, ale ~25 MB se třemi 12bitovými mono kamerami**. Projekce určí formát každé kamery na základě jejího modelu (cache identit), takže datový proud odpovídá tomu, co se skutečně přenáší po kabelu — nejde o univerzální předpoklad formátu BayerRG8.
- **USB ethernetový adaptér je omezen na 200 MB/s bez ohledu na údaje na štítku.** Tabulka účinnosti, která převádí přenosovou rychlost na trvalý výkon, je odvozena od PCIe; USB síťová karta sice udává svou *ethernetovou* přenosovou rychlost, ale je omezena USB sběrnicí a svým ovladačem. USB 10GbE adaptér dříve dosahoval „trvalé“ rychlosti ~1063 MB/s — číslo, které nebylo nikdy ověřeno — a výsledné omezování rychlosti poškodilo 6–18 % rámců, přičemž stále vykazovalo správnou cílovou hodnotu snímků za sekundu (fps). Síťové karty připojené přes USB jsou nyní absolutně omezeny na **200 MB/s** (limitem je sběrnice, takže se neřídí údaji na štítku; USB adaptér 1 GbE dosahuje ~80 MB/s a není tímto omezením dotčen). `wire_ceiling_source` v záznamu o schopnostech to uvádí slovy a `nic_is_usb` to označuje. Obojí lze přepsat pomocí `--wire-ceiling-mbps`.
- **Admittance zohledňuje odběr, nikoli na rozlišení mezi celým burstem a prstencem.** Souběžný burst se musí vejít pouze do *přechodného backlogu* = `max(0, Σ per-cam arrival − host drain) × emit_window`, nikoli do celého burstu. V síti s rychlým hostitelem a pomalými kamerami (**PCIe**10G hostitel + 4× 1 GbE kamery: příchod ≈ 320 MB/s, odvod ≈ 1063 MB/s) se hostitel vyprazdňuje rychleji, než se kamery plní, backlog ≈ 0, takže simulace vysílání v plném rozlišení**projde**, i když burst o velikosti 25 MB přesahuje kapacitu prstence 13,5 MB. Umístěte tytéž čtyři kamery za**USB**10GbE adaptér, a odesílání činí 200 MB/s, nikoli 1063 — příchozí data ho předběhnou a ztráta se projeví jako poškozené rámce, nikoli jako nižší snímková frekvence. Na hostitelském počítači s 1 GbE způsobí minimální hodnota DLThr kamer 31,25 MB/s to, že příchod dat předstihne odběr → systém správně**blokuje** (u *této* třídy bloků zmenšete oblast zájmu (ROI) nebo použijte binning ≥ 2). Přístup je jednou ze **dvou** spojovacích bran – druhou je níže uvedená kontrola agregovaného nadměrného předplatného.
- **Předpokládaná snímková frekvence (fps) představuje konzervativní horní hranici sériového načítání.**Smyčka hostitele pro načítání dat v současné době stahuje vyrovnávací paměť každé kamery**sériově**(přibližně jedno okno odesílání na kameru), takže cyklus je omezen `max(readout+emit, N × emit)`, přičemž vysílání na kameru je omezeno**přístupovým spojem**kamery (1 GbE ≈ 80 MB/s), nikoli uplinkem hostitele. Pro pole se 4 kamerami v plném rozlišení a 12 bitech to činí**~2,8 fps**, což odpovídá naměřeným hodnotám ~2,7–3,0. Hodnota fps je záměrně**nezávislá na expozici**, takže v tmavých scénách může skutečná hodnota mírně klesnout pod horní hranici s prodlužující se expozicí. Skutečným omezovačem fps je sériové načítání; jeho paralelizace by posunula horní hranici směrem k rychlosti jediné emise.
- **Agregátní nadměrné přidělení je tvrdou překážkou připojení.**Minimální přidělená šířka pásma na jednu kameru činí**8 MB/s**(`ARRAY_PER_CAM_FLOOR_BPS`), takže jakmile se tato minimální hodnota ustálí, může celková poptávka (`per_cam × N`) překročit**strop pro kolizně bezpečné připojení**(`sustained × sim_emit_factor`). Praktické stropy pro plné rozlišení na 1 GbE:**6 kamer při 1500 MTU, 9 s jumbo rámci**. Tento strop je dán výhradně vlastnostmi sítě a dolní hranicí — je**nezávislý na velikosti rámce**, takže**bina menší ROI nepomohou** (snižují počet bajtů na *rám*, nikoli počet bajtů za *sekundu* řízený GevSCPD); jedinými řešeními je menší počet kamer, použití jumbo rámců v celé síti nebo rychlejší síťová karta. Příznakem by byla ztráta paketů GVSP, nikoli plynulé snížení fps, takže `analyze-array` vynuluje hodnoty dosažitelného fps a vypíše `**OVER-SUBSCRIBED**`, zatímco `array-connect` s pevně nastaveným rozlišením **odmítne navázat spojení** (funkce walk-down jinak sráží snímky, což tuto třídu bloků rovněž nevyřeší). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` sníží odmítnutí na hlasité varování pro práci na testovacím prostředí — viz [Proměnné prostředí](#environment-variables).

### Stav pole — který subsystém ztrácí snímky

`GET /api/camera/array/<array_id>/capability` připojeného pole nese živý
blok `health`, který je přehodnocován v průběžném **10sekundovém** okno. Rozděluje ztrátu rámců
na dvě příčiny, které vyžadují opačná řešení, namísto hlášení jedné „neúplné“
míry, která žádnou z nich nejmenuje:

| Pole | Co to znamená | Který subsystém |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (pro každý sériový port) | Rámec **dorazil a byl strukturálně poškozen**— ztráta paketů GVSP. |**Síť**: kapacita linky, tempo, kruh NIC RX, MTU |
| `never_arrived_rate_pct` (na sériový port) | Rámec **vůbec nedorazil**— kamera se nespustila, nebo z ní nic nevyšlo. |**Spoušť / synchronizace**: kabel M8, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Nejhorší míra selhání kamery pro každou z nich. | — |
| `per_cam_rate_pct` | Kombinovaná míra neúplnosti na kameru (obě příčiny dohromady). | — |
| `stable_for_seconds` | Jak dlouho každá kamera zůstala pod 0,01 %. | — |

Při hodnotě nad 5 % backend zaznamená řádek `[array-health <id>] WARN` s uvedením rozdělení — při
prvním porušení, při změně pásma závažnosti, jednou za minutu, dokud stav přetrvává, a jednou, když
se stav vyřeší. Poškozená polovina vypíše `[gvsp-corrupt <SN>]` při prvním výskytu pro každou kameru a
důvod, poté sekaždých 60 s. Každé vyhodnocení se stále ukládá do logového souboru backendu;
počítadla se posouvají u každého bufferu bez ohledu na to, co se vypíše.

Stejný záznam uvádí číslo, na kterém visí celá alokace:

| Pole | Co znamená |
| --- | --- |
| `wire_ceiling_mbps` | Aktuální trvalý rozpočet šířky pásma hostitele, MB/s. |
| `wire_ceiling_source` | Odkud tato hodnota pochází, slovy — např. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` nebo `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, pokud jej nastavuje `--wire-ceiling-mbps` (nebo pole **Wire Budget** v grafickém rozhraní). |
| `nic_is_usb` | `true` pro USB ethernetový adaptér — viz výše uvedené omezení na 200 MB/s. |

**Interpretace:** nenulová hodnota `gvsp_corrupt_rate_pct` při hodnotě `never_arrived_rate_pct` rovné 0
znamená, že spouštění a synchronizace kabelu jsou v pořádku a 100 % ztráty je na síťové
trase — snižte hodnotu `--wire-ceiling-mbps` a znovu připojte. Opačný vzorec naopak poukazuje na
synchronizační kabel nebo spouštěcí linku.

> **`--target-fps` není příčinou poškozených rámců.** Tempo GevSCPD se zapíše
> pouze jednou při připojení, takže snížení spouštěcí frekvence mění pracovní cyklus, nikoli však
> rychlost souběžného vysílání burstů. Naměřené 5× snížení požadavků nepřineslo žádné zlepšení;
> snížení maximální rychlosti linky z 240 na 200 MB/s snížilo u stejného zařízení podíl poškozených dat z 10,4 %
> na 0,00 %.

> **Automatické zmenšení uprostřed přenosu není ve firmwaru TRI032S k dispozici.** Běžící pole
> to nemůže opravit samo; odpojte a znovu připojte, aby nástroj pro výběr času připojení mohl
> znovu naplánovat s novým stropem.

### Příznak → řešení

| Příznak (Nastavení pole / připojení / `analyze_array_network`) | Příčina | Řešení |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` se resetuje na 32 (obvykle po aktualizaci ovladače) | Nastavte `ReceiveBufferLen`→256, `PendingReceives`→64; znovu otevřete panel (restartujte backend, pokud si uložil do mezipaměti starou velikost prstence) |
| Restart/vypnutí se zasekne; později `Invalid class` chyby WMI, síťová karta se neaktivuje, chybí USB disky | Starý ovladač Realtek USB 10GbE z roku 2016 → BSOD `0x9F` → vynucené vypnutí | Aktualizujte ovladače adaptéru na verzi ≥ v10.67 (2026) a poté znovu použijte výše uvedená nastavení přijímacího prstence |
| Připojení se podaří, ale vrací rozlišení nižší než nativní | Funkce Smart-prep automaticky zmenšila rámec, aby se vešel do linky | Upgradujte spojení / přijměte zmenšení / `--force-tier slip-emit-and-capture` |
| Pole hlásí správnou cílovou hodnotu snímků za sekundu (fps), ale dodává jen zlomek této hodnoty; `health.gvsp_corrupt_rate_pct` nenulová hodnota, `never_arrived_rate_pct` 0 | Odvozený rozpočet šířky pásma hostitele přesahujeuvádí, než ve skutečnosti vydrží (typické u USB ethernetového adaptéru, úzké PCIe linky nebo sdílené sítě) | Znovu se připojte s nižší hodnotou `--wire-ceiling-mbps` a znovu zkontrolujte blok stavu. **Ne** `--target-fps` — Pacing GevSCPD je při připojení pevně nastaven |
| Chybějící kamery ve zveřejněných skupinách; `health.never_arrived_rate_pct` nenulová, `gvsp_corrupt_rate_pct` 0 | Spouštěcí / synchronizační cesta — kamery se nespouštějí, nejde o síťový problém | Zkontrolujte synchronizační kabel M8 a hodnotu `--line`; ověřte, zda je každý člen v pohotovostním režimu (`TriggerMode=On`) |
| Hodnota `**OVER-SUBSCRIBED**` / `Wire budget` překročena v `analyze-array`, nebo odmítnutí připojení s pevně nastaveným rozlišením (`array over-subscribes the wire`) | Souhrnná poptávka na jednu kameru (minimálně 8 MB/s × N kamer) překračuje horní hranici bezpečnou proti kolizím — 6 kamer v plném rozlišení na 1 GbE při 1500 MTU, 9 s jumbo rámci | Menší počet kamer, jumbo rámce v celém řetězci nebo rychlejší síťová karta. **ROI/binning nepomůže** (strop je nezávislý na velikosti rámce). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` přepisuje nastavení na testovací platformě (akceptuje ztrátu paketů) |

---

## `chloros-cli daq`

Příkazy pro spektrální senzor. Dvě třídy:
- **`pool-*`**— tenké HTTP klienty, které ovládají senzor prostřednictvím trvalého fondu backendu.**Toto je podporovaná cesta a jediná, která je k dispozici v dodávaném balíčku CLI.** Backend vlastní transport, takže grafické rozhraní, skripty CLI a SDK sdílejí jedno aktivní handle, místo aby se přetahovaly o sériový port.
- **Všechno ostatní**(`test`, `record`, `live`, `stream`, `connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`, `serve`, `ws`, `udp`, `mqtt`, `reflectance`, `login`, `logout`, `status`) — přímý přístup k hardwaru, pro úplnost popsán níže. Tyto funkce vyžadují balíček `daq` Python, který**není součástí žádného dodávaného artefaktu**: zkompilovaný balíček CLI jej neobsahuje (`scripts/Build-CLI.ps1` nastavuje `--nofollow-import-to=daq` a transporty `pyserial` / `bleak` / `zeroconf`), a balíček PyPI SDK jej rovněž neobsahuje. Fungují pouze ze zdrojového kódu, takže je považujte spíše za MAPIR -interní vývojovou cestu než za něco, k čemu byste se měli uchýlit.
- **`discover` / `list`** se nacházejí někde mezi: jedná se o přímé hardwarové příkazy ze zdrojového kódu, ale v dodané sestavě se přepnou na `pool-discover` a skenování provede backend. Skenování tedy funguje všude — což je důležité, protože je to jediný způsob, jak zjistit BLE MAC zařízení DAQ-M.

> **`chloros-cli daq --help`** (a `-h` / `help`) vypisuje podpříkazy `pool-*` — nápověda je záměrně směrována na klienta poolu, aby odrážela příkazy, které se skutečně spouštějí. Pokud v dodané verzi vyvoláte podpříkaz pro přímé ovládání hardwaru, program se ukončí s explicitní chybou s uvedením chybějícího balíčku a odkazem zpět na `pool-*`; nic neselže bez upozornění. (`discover` / `list` jsou výjimkou — přesměrují na `pool-discover` a prostě fungují.)
>
> **Vše, co zákazník potřebuje, je dostupné prostřednictvím `pool-*`** — připojení, streamování, záznam kalibrovaných souborů `.daq` a výměna profilů kondenzátorů. DAQ lze také ovládat z Python pomocí `chloros_sdk.connect_daq_sensor()`, který využívá stejnou sdílenou cestu.

### Pracovní postup při prvním připojení senzoru DAQ

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Referenční příručka `pool-*`

| Podpříkaz | Účel |
| --- | --- |
| `daq pool-connect` (smart-detect) | Otevře senzor v backendovém poolu. |
| `daq pool-connect --port PORT` | DAQ-U na konkrétním sériovém portu. |
| `daq pool-connect --ble` | DAQ-M přes BLE, automatické vyhledávání MAC adres. |
| `daq pool-connect --mac MAC` | DAQ-M přes BLE na známé adrese MAC (vyžaduje `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E přes Ethernet na známém hostiteli. |
| `daq pool-connect --eth` | DAQ-E přes Ethernet, hostitel automaticky zjištěn (mDNS + záložní ARP; funguje i s prázdnou mezipamětí ARP na adresách Windows a Linux). |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | Nastavení integračního okna / stavu AE. |
| `daq pool-connect --no-stream` | Připojit, ale zatím nezačít streamovat (pokračovat pomocí `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Profil korekce Cap. Výchozí nastavení na backendu je `sunshine_cosine`. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Prohledat každý transportní kanál pro senzory, ke kterým se lze připojit, aniž by došlo k připojení. **Takto najdete BLE MAC adresu zařízení DAQ-M.** `daq discover` / `daq list` se v dodávaných verzích automaticky přesměrují sem. Senzory, které jsou již otevřené v poolu, nejsou uvedeny — připojený DAQ-M přestane vysílat — proto pro ně použijte `pool-list`. |
| `daq pool-list` | Zobrazit všechny senzory v backendovém fondu. |
| `daq pool-disconnect --sensor-id ID [--all]` | Uvolnit. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Nejnovější rámce spektra N. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Obnovit / pozastavit streamování. |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | Spustit / zastavit záznam .daq. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Výměna profilu korekce kapacitního efektu za běhu. |

### Podpříkazy pro přímé ovládání hardwaru (pouze ve zdrojovém kódu – nejsou součástí distribuovaných sestavení)

> Uvedeno pro úplnost. Tyto příkazy vyžadují balíček `daq` Python a navíc `pyserial` / `bleak` / `zeroconf`, z nichž žádný není součástí v kompilované verzi CLI ani na PyPI SDK — fungují pouze ze staženého zdrojového kódu MAPIR. **Pokud používáte vydanou verzi Chloros, použijte místo toho výše uvedené příkazy `pool-*`**; pokrývají připojení, streamování, nahrávání a výběr kap.

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

Otevření, připojení a ovládání uloženého projektu Chloros (složka obsahující `cameras.json` + `sensors.json` + `project.json`). Vše prochází přes backend, takže grafické uživatelské rozhraní a příkaz CLI poskytují identický stav hardwaru.

### Přehled podpříkazů

| Podpříkaz | Účel |
| --- | --- |
| `project open PATH` | Vytiskne seznam zařízení v projektu (kamery, pole, senzory). |
| `project devices PATH [--reconnect]` | Zobrazí seznam nebo znovu spustí vyhledávání. |
| `project connect PATH [--cameras-only] [--sensors-only]` | Připojí všechny uložené kamery / pole / senzory. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Jednotlivé snímky z určené kamery nebo pole. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Série N snímků z určené kamery nebo pole (`-n/--count` výchozí hodnota 5; `-i/--interval` interval mezi snímky v sekundách, výchozí hodnota 0). Série snímků z pole odstraňují duplicity opakovaných synchronizovaných skupin (kontrolní mechanismus zastaralosti), takže částečně cyklické pole nemůže vrátit N kopií jednoho snímku; vypisuje výsledky za každou iteraci. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | Přepis na disk prostřednictvím úlohy na backendu. `--poll-interval` = interval v sekundách mezi dotazy `/stats` (výchozí hodnota 2,0). |
| `project sensor read PATH NAME [--json]` | Nejnovější spektrální rámec. |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | Zaznamenat .daq. |
| `project run PATH RECIPE.yaml` | Spuštění receptu pro sběr dat ve formátu YAML/JSON. `--dry-run` provede ověření bez spuštění. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Výpočet zarovnání pro pole — viz [tabulka příznaků níže](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Vytisknout aktuální profil zarovnání. |
| `project align clear PATH NAME` | Vymazat profil z mezipaměti. |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | Posunout transformaci jednoho slave. |
| `project align export PATH NAME --to FILE` | Uložit profil do souboru JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Načíst uložený profil. |

#### Možnosti `project align calibrate`

| Příznak | Výchozí | Popis |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Metoda zarovnání. **Tyto názvy se liší od `lattice align-calibrate`**, který používá zkrácené formy `orb` / `akaze` / `phase`; tyto dva příkazy v tomto parametru nejsou zaměnitelné. |
| `--model {translation, rigid, affine, homography}` | `affine` | Transformovat model tak, aby se vešel. |
| `--frames N` | `1` | Synchronizované snímky snímků s průměrem. |
| `--reference SN` | hlavní | Sériové číslo referenční kamery; všechny ostatní členy se na ni deformují. |
| `--max-features N` | `5000` | Omezení počtu prvků ORB. |
| `--ratio-threshold F` | `0.75` | Loweův test poměru. |
| `--ransac-threshold-px F` | `3.0` | Prahová hodnota pro vnitřní body v RANSACu. |
| `--min-matches N` | `15` | **Kritérium kvality** — odmítnout řešení s menším počtem shodných bodů. |
| `--max-reproj-err-px F` | `4.0` | **Kritérium kvality** — odmítnout řešení při překročení této chyby reprojekce RMS. |
| `--checkerboard RxC` | — | Geometrie desky pro `--method checkerboard`, např. `9x6`. |
| `--name PROFILE` | prázdné | Název profilu vložený do uloženého souboru JSON. **Nejedná se o název pole** — tím je poziční `NAME`. |

Tyto dvě kontroly kvality jsou důvodem, proč může kalibrace úspěšně vyřešit výpočet a přesto
odmítnout uložení: profil, který nesplňuje podmínky ani jedné z nich, by bez upozornění nesprávně zaregistroval každé
následující zachycení, a proto je odmítnut, místo aby byl uložen.

### Příklady

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### DSL receptu

`project run RECIPE.yaml` přijímá soubor YAML nebo JSON popisující posloupnost akcí:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Podporované akce: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. Akce `burst` vyžaduje `name` (povinné), `count` (výchozí hodnota 5), `interval` (v sekundách, výchozí hodnota 0), `output`, `format` a `settings` (stejný tvar nastavení pro jednotlivé kamery jako `apply`); série snímků z pole používají stejný watchdog pro čerstvě synchronizovanou skupinu jako `project burst`.

Spusťte:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## Proměnné prostředí

| Proměnná | Účinek |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Přepíše backend URL (výchozí hodnota `http://127.0.0.1:5000`) — **respektováno pouze rodinami příkazů `lattice`, `project` a `daq pool-*`.** Základní příkazy (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) připojují proměnnou `http://127.0.0.1:<port>` a tuto proměnnou ignorují (literál IPv4 obchází Windows `localhost`→`::1` penalizaci ~2 s na jeden požadavek), takže se vždy zaměřují na místní počítač. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1` snižuje limit pro odmítnutí připojení v důsledku nadměrného předplatného pole (souhrnná poptávka na kameru &gt; kolizní-bezpečný limit pro linku s `pin_resolution`) na hlasité varování a pokračování, přičemž akceptuje ztrátu paketů GVSP. Pouze pro testovací účely — viz [Model fps a burst pole](#array-fps--burst-model). |
| `CHLOROS_CLI_MODE` | Nastavuje samotný modul CLI; sděluje backendu, aby povolil paralelní zpracování. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` přeskočí záložní test GVSP (pouze výsledky ICMP). **Tím se vypne režim jumbo, nejenže utlumí protokol** — kamera odpovídá na pingy DF pouze do 1500 na každé trase, takže tato sonda je jedinou věcí, která dokáže detekovat jumbo. Ušetří ~1 s na kameru na připojení; stojí ~1,45× maximální šířku pásma, pokud síť *mohla* přenášet pakety typu jumbo. Při nastavení vás na to upozorní SDK. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Fixuje velikost paketu GVSP na N bajtů; zcela vynechává testování. Upřednostňujte nastavení pro jednotlivé příkazy (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) před trvalým nastavením: fixovaná velikost přestane přizpůsobovat síti před ní a fixace9000 na trase, která nepodporuje jumbo pakety, způsobí, že **každý** záznam vyprší s chybou `SC_ERR_TIMEOUT -1011`. |
| `TMPDIR` (Linux) | Přepíše výchozí adresář pro extrakci souborů Nuitka. Příkaz CLI automaticky použije `/mnt/ssd/tmp`, je-li zadán. |

---

## Výstupní kódy

| Kód | Význam |
| --- | --- |
| `0` | Úspěch. |
| `1` | Obecná chyba (většina chyb podpříkazů). |
| `2` | Chyba argumentu. |
| `130` | Přerušeno klávesovou zkratkou Ctrl+C. |

---

## Tipy pro řešení potíží

- **„Vyžaduje se přihlášení“** → Spusťte na tomto počítači jednou příkaz `chloros-cli login EMAIL PASSWORD`.
- **„Backend není dostupný“** → Spusťte desktopovou aplikaci Chloros nebo přímo spusťte binární soubor backendu (`chloros-backend`), nebo v případě vzdáleného přístupu zkontrolujte `CHLOROS_BACKEND_URL`.
- **Příkazy `lattice` selhávají s hláškou „LATTICE camera drivers not found“** → Není nainstalováno runtime prostředí Arena SDK; balíček CLI obsahuje `win32api` na Windows, ale runtime prostředí C je součástí grafického instalačního programu.
- **V okně Array connect / Array Settings se zobrazuje „FRAMES WILL DROP“ nebo „Reduce ROI to enable“** → Příjmový kruh síťové karty hostitele je příliš malý (obvykle se po aktualizaci ovladače síťové karty resetuje na 32). Viz [Nastavení a ladění síťové karty hostitele](#host-nic-setup--tuning-lattice-arrays) — nastavte `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Počítač se při restartu/vypnutí zasekne, poté WMI `Invalid class` / síťová karta seaktivovat / chybí USB disky** → Zastaralý ovladač adaptéru USB 10GbE způsobuje chybu `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Aktualizujte ovladač adaptéru — viz [Nastavení a ladění hostitelské síťové karty](#host-nic-setup--tuning-lattice-arrays).
- **Varování ohledně swapového prostoru na Jetsonu** → Přidejte swapový prostor založený na souborech; příkaz `CLI` vypíše přesné příkazy `fallocate` / `swapon`.
- **Chybějící přímé příkazy DAQ** → Očekávané: dodávaný balíček `chloros-cli` záměrně vylučuje balíček `daq`, takže je k dispozici pouze `pool-*` (ani na PyPI SDK není k dispozici). Použijte `pool-*`, který ovládá stejný senzor prostřednictvím backendu, nebo `chloros_sdk.connect_daq_sensor()` z Python.

---

## Viz také

- [Python SDK Reference](sdk-reference.md) — programový ekvivalent každého příkazu CLI.
- [Průvodce senzory DAQ](../daq/README.md) — zapojení a kalibrace konkrétních senzorů.
- Online dokumentace: `https://mapir.gitbook.io/chloros/cli`
