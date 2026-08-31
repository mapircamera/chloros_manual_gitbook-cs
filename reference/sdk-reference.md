# Chloros Python SDK Reference

**Verze:**

1.2.0**Vytvořeno:**

29. 7. 2026 19:19 ·**Revidováno:**

30. 8. 2026**Balíček:** `chloros-sdk` (PyPI)**Cílová skupina:** Optimalizováno pro použití v LLM; čitelné pro člověka.**Rozsah:** Všechny veřejné třídy, funkce a pomocná funkce zpřístupněná balíčkem `import chloros_sdk`, s příklady, které lze zkopírovat a vložit, pokrývajícími zpracování obrazu, ovládání jedné kamery, synchronizovaná pole, senzory DAQ a automatizaci projektů.

Pokud vás zajímají pouze hlavní body, přejděte na:
- [Instalace a rychlý start](#installation)
- [Smart-Connect pro pole LATTICE](#smart-connect-for-lattice-cameras)
- [Relace senzorů DAQ](#daq-sensor-sessions)
- [Automatizace projektu](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## Architektura za 60 sekund

SDK je tenká vrstva typu „Python“ nad backendem Chloros (stejný server Flask, jaký používá desktopové grafické rozhraní a CLI). Pro automatizaci importujete `chloros_sdk` a voláte metody vyšší úrovně; v pozadí se každé volání promění v požadavek HTTP na místní backend na portu 5000 — `http://127.0.0.1:5000/api/...` (záměrnězáměrně ne `localhost`, který se nejprve přeloží na `::1` na adrese Windows a u backendu podporujícího pouze IPv4 stojí ~2 s na jeden požadavek). Backend spravuje hardwarový fond — kamery, senzory DAQ, profily zarovnání, rámcové vyrovnávací paměti — takže skripty SDK mohou koexistovat s grafickým uživatelským rozhraním, aniž by se musely přetahovat o sériové porty nebo šířku pásma síťové karty.

K dispozici jsou tři rozhraní, která budete používat:

1. **`ChlorosLocal` + volné funkce** (`process_folder`, `process_lattice_capture`) – Zpracovací pipeline obrazu. Spusťte zpracování celé složky včetně kalibrace, debayeringu a exportu indexu jediným voláním funkce `Python`.
2. **Ovládací prvky Smart-connect** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Otevření trvalé relace backendu pro živý hardware. Stejný postup „smart-prep“ jako v grafickém uživatelském rozhraní: testování sítě, automatický výběr úrovně, PTP, inicializace AE, konfigurace spouštěče GPIO.
3. **`ChlorosProject` / `open_project`** — Načtení uloženého projektu (složka obsahující `cameras.json` + `sensors.json` + `project.json`), připojte vše najednou a provádějte záznamy pomocí pojmenovaných handleů.

Povrchy 1 a 2 **automaticky spustí lokální backend**, pokud ještě žádný neposlouchá (stejný balíček binárních souborů, jaký spouští GUI/CLI) — takže holý skript funguje z nového terminálu, aniž byste museli nejprve spouštět backend. Předáním `auto_start_backend=False` tuto funkci deaktivujete (např. při nasměrování na vzdálený backend, který se nikdy nespustí). Viz [Automatické spuštění backendu](#backend-auto-start). Surface 3 se chová odlišně: `open_project()` nepřijímá žádný parametr `auto_start_backend` a `connect_all()` nikdy nespustí backend— jednou zkusí kontaktovat `http://127.0.0.1:5000` a pokud nic neodpoví, tiše přejde na přímé (bez backendu) ovládání zařízení `lattice_sdk`. Pouze `proj.process()` a `stream(..., overlays=True)` odloženě vytvoří `ChlorosLocal()` (které se spouští automaticky).

Všechny tři vyžadují autorizaci: na daném počítači je třeba jednou spustit `chloros-cli login` nebo se přihlásit přes grafické rozhraní na ploše. Volání SDK bez platné relace vyvolá chybu `ChlorosAuthenticationError`.

Požadavky:
- Python 3.7+ (jak uvádí balíček; vyvinuto/testováno na verzi 3.10)
- Lokálně nainstalovaná aplikace Chloros Desktop (binární soubor backendu je součástí instalačního balíčku)
- Aktivní přihlašovací údaje na Chloros+. Minimální úroveň pro SDK / CLI je **Copper**nebo vyšší (Copper / Bronze / Silver / Gold); bezplatná úroveň**Iron**nemá přístup k SDK / CLI. Toto omezení je vynucováno**na straně serveru**: každý požadavek s příznakem SDK / CLI musí obsahovat jak aktivní relaci, tak placený tarif, jinak backend vrátí kód chyby `403` s kódem `error_code: PLAN_UPGRADE_REQUIRED` (zobrazený jako `ChlorosLicenseError` pomocnými funkcemi `ChlorosLocal` a jako `ChlorosConnectError` pomocnými funkcemi `connect_*`). Odvolatel, který je odhlášen, obdrží chybu `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — tyto dva kódy se liší, protože opětovné spuštění `chloros-cli login` opraví první z nich, ale druhý opravit nedokáže.
- Offline použití je podporováno v rámci ochranné lhůty tarifu: úroveň se načítá z mezipaměti pro ověření serverem (5 minut) nebo z mezipaměti podepsaných licencí vázaných na počítač (30 dní u měsíčních tarifů, do vypršení předplatného u ročních). Po uplynutí této lhůty se tarif převede na bezplatný a přístup k SDK / CLI se zastaví, dokud se počítač alespoň jednou nepřipojí k serveru. `chloros-cli status` (`GET /api/license-status`) zůstává v bezplatné úrovni dostupný, takže důvod je zřejmý – jedná se o jedinou trasu SDK / CLI, která není omezena úrovní služby.
- Windows 10/11 64bitová verze, **Ubuntu 22.04 LTS nebo novější**, nebo Jetson (JetPack 6). Ubuntu 20.04**není** podporováno: závislosti `.deb` jsou odvozeny od toho, na co se odkazuje backend, včetně `libc6 (>= 2.34)`, a Focal dodává glibc 2.31.

---

## Instalace

Python SDK je tenká vrstva Python nad backendem Chloros. Pro vše, co přesahuje několik pracovních postupů zaměřených výhradně na sběr dat (DAQ), potřebujete **místně nainstalovaný balíček Chloros pro desktop** (instalační program Windows nebo Linux `.deb`) — ten poskytuje binární soubor backendu, runtime Arena SDK pro kamery LATTICE a kalibrační balíčky.

Nejnovější soubory ke stažení: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Krok 1 — Nainstalujte balíček platformy Chloros

#### Windows (.exe)

1. Stáhněte si soubor `Chloros-Setup-x.y.z.exe` ze stránky pro stahování.
2. Spusťte instalační program a postupujte podle pokynů průvodce. Výchozí instalační cesta je `C:\Program Files\MAPIR\Chloros\`.
3. Spusťte alespoň jednou stránku Chloros a přihlaste se pomocí svého účtu Chloros+.

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### Krok 2 — Nainstalujte balíček „Python“ SDK

**Instalátor Chloros obsahuje odpovídající balíček „SDK“ ve formátu wheel.** Každý instalátor Windows a balíček Linux ve formátu .deb umístí na disk soubor `chloros_sdk-X.Y.Z-py3-none-any.whl`, který přesně odpovídá verzi GUI / CLI / backendu. Nemusíte sledovat PyPI, abyste zůstali v synchronizaci.

#### Windows

Instalační program automaticky spustí soubor `pip install` s přiloženým balíčkem wheel pomocí vašeho systémového Python (upřednostňuje se spouštěč `py.exe`, v případě selhání se použije `python -m pip`). Není třeba nic dělat — po úspěšné instalaci bude `import chloros_sdk` fungovat ve vašem prostředí Python. Pokud na počítači není nainstalován Python, instalační program tento krok tiše přeskočítento krok a grafické rozhraní i CLI nadále fungují.

#### Linux (.deb)

Balíček .deb umístí wheel do adresáře `/usr/lib/chloros/sdk/`. `postinst` vypíše přesný příkaz — distribuce dodržující PEP 668 ve výchozím nastavení odmítají globální zápisy pomocí pip, proto neprovádíme automatickou instalaci:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

U nasazení Jetsonu v izolovaném prostředí (air-gapped) probíhá tento proces zcela offline — balíček wheel je již na disku.

#### Veřejný PyPI

Pro hostitele používající pouze pip (bez nainstalovaného balíčku Chloros pro desktop; pracovní postupy s vzdáleným backendem nebo pouze s DAQ):

```bash
pip install chloros-sdk
```

PyPI se aktualizuje při sestavení instalačního programu pro danou verzi, takže zveřejněný balíček typu „wheel“ odpovídá nejnovější stabilní verzi. Vývojové sestavení (např. `1.1.4.dev1`) se dodávají pouze prostřednictvím balíčku typu „wheel“ v instalačním programu.

#### Ověření

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Vyžaduje se předplatné Chloros+.** Všechna volání SDK vyžadují aktivní přihlášení na Chloros+. Spusťte `chloros-cli login user@example.com 'YourPassword'` jednou na každém počítači; přihlašovací údaje se ukládají do mezipaměti v `~/.chloros/`.

### Potřebuji balíček pro stolní počítače?

Samotný balíček pip pro většinu pracovních postupů **nestačí**. Zde je přehled toho, co jednotlivé povrchy SDK potřebují:

| PovrchSDKu | Potřebuje balíček Desktop Package? | Proč |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Ano** | Automaticky spouští binární soubor backendu na `/usr/lib/chloros/chloros-backend` (Linux) nebo `C:\Program Files\MAPIR\Chloros\…` (Windows). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Ano**(lokální)**/ Ne**(vzdálený) | Čistě HTTP klienti přes backend. Lokální backend → vyžaduje balíček pro desktop. Vzdálený backend → `backend_url=`**přes tunel** (viz Režim vzdáleného backendu — dodávané backendy se vážou pouze na smyčku). |
| `ChlorosProject` / `open_project` | **Ano** | Řídí uložené projekty přes backend. |
| Přímé třídy LATTICE (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Ano** | Vyžaduje nativní runtime Arena SDK, který je součástí balíčku pro stolní počítače. `CAMERA_AVAILABLE` je jinak při importu `False`. |
| Třídy Direct DAQ (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **Ne** | Čistý Python přes pyserial/bleak/zeroconf. Prostředí využívající pouze pip dokáže řídit DAQ od začátku do konce. |

### Režim vzdáleného backendu (hostitel pouze s pip, přes tunel)

> **Dodávaný backend není dostupný přes LAN.** Produkční
> sestavení se vážou pouze na loopback (obě rodiny loopbacků) a kategoricky odmítají
> režim bez loopbacku (`CHLOROS_CLOUD_MODE`), takže
> `backend_url="http://<lan-ip>:5000"` **nemůže fungovat s nainstalovaným
> Chloros** — tento vzor fungoval vždy pouze s backendem ze zdrojového kódu/dev
> backendu. Chcete-li ovládat backend na jiném stroji, přesměrujte jeho loopbackový
> port sami a nasměrujte SDK na tunel:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

Hostitelé bez grafického rozhraní / CI / robotika mohou ponechat jeden počítač s plnou instalací desktopu jako „server Chloros“ a `pip install chloros-sdk` všude jinde — přenos mezi nimi však probíhá prostřednictvím výše uvedeného tunelu nastaveného uživatelem, nikoli přímým LAN URL.

> **Známé omezení — `ChlorosLocal` nepodporuje výhradně pip.** `ChlorosLocal(backend_url=BACKEND)` v současné době vyřeší lokální binární soubor backendu ve svém konstruktoru *před* prozkoumáním URL a vyvolá chybu `ChlorosBackendError` („Chloros backend not found…“), pokud není nainstalován žádný balíček pro desktop — a to i v případě, že je vzdálený backend dostupný. Pouze výše uvedené rozhraní smart-connect (`connect_camera` / `connect_array` / `connect_daq_sensor`, plus `analyze_array_network` a pomocné programy `list_*` / `discover_*`) funguje z hostitele pouze s balíčkem pip.

### Pracovní postup pouze pro DAQ (hostitel pouze s balíčkem pip)

Pokud potřebujete pouze senzory DAQ a nepoužíváte kamery LATTICE ani zpracování obrazu, balíček pip je samostatný:

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

Žádný backend, žádný balíček .deb, žádné přihlášení na Chloros+ není nutné pro práci s DAQ přímo na hardwaru.

---

## Rychlý start

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Index nejvyšší úrovně API

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## Zpracování obrazu — `ChlorosLocal`

Hlavní třída pipeline. Při prvním použití spustí backend, vytvoří a nakonfiguruje projekty, sleduje průběh a vrací souhrny po dokončení běhu.

### Konstruktor

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Metody

| Metoda | Popis |
| --- | --- |
| `create_project(project_name, camera=None)` | Vytvoří nový projekt (volitelně s šablonou kamery, jako je `"Survey3N_RGN"`). |
| `import_images(folder_path, recursive=False)` | Importuje snímky ve formátech RAW/TIF/JPG/DNG **a záznamy ze světelného senzoru `.daq`**. Vrací `count` (snímky) a `scan_count` (záznamy). Vydá varování pouze v případě, že složka neobsahuje ani jedno z nich. |
| `export_light_sensor(daq=True, csv=True)` | Zapíše kalibrované soubory `.daq` + `.csv` pro každý záznam ze světelného senzoru v projektu do souboru `<project>/Light Sensor/`. Viz [Záznamy světelného senzoru](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Nastavte ovládací prvky zpracování. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Spusťte zpracovatelský řetězec. Vrátí `{"status": "complete", "async": False}` a navíc klíč `summary`, pokud jej backend poskytne — viz [Souhrn a tipy po spuštění](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | Zkontrolujte stav backendu. |
| `logout()` | Vymazat přihlašovací údaje z mezipaměti. |
| `shutdown_backend()` | Ukončit backend (pokud byl spuštěSDK). |
| `discover_cameras()` | Vyhledat kamery LATTICE **prostřednictvím backendu této instance** (`/api/camera/discover`). Vrátí seznam slovníků (`serial`, `model`, `ip`, …) — ve stejné struktuře, jakou vidí GUI/ CLI. Prázdný seznam, pokud nebyly nalezeny žádné kamery nebo je backend nedostupný. |
| `camera_capture(output_dir, format="tiff", **settings)` | Zachycení jednoho snímku**prostřednictvím backendu**(automaticky spuštěno tímto identifikátorem), takže se na něj aplikuje stejná příprava jako v GUI/ CLI (výchozí nastavení 12bitů ve výchozím nastavení, opětovné použití poolu, vložená metadata kalibrace). Cíl vyřešte pomocí `serial=` nebo `device_index=`; předávejte `exposure`/`gain`/`pixel_format`/`preset` jako `**settings`. Vrátí slovník starších metadat (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Vytváří překryvné kompozitní náhledové snímky z seskupených kamer — odlehčený MJPEG klient přes trasu `/api/camera/<serial>/stream-annotated` backendu (zebra / mřížka / nitkový kříž / histogram / peaking / bod kreslené na straně serveru). `decode=True` poskytuje pole BGR; `False` poskytuje surové bajty JPEG. Dostupné také na úrovni projektu jako `ChlorosProject.stream(overlays=True)`. |

Použijte jako správce kontextu pro zaručené vyčištění:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Záznamy ze světelných senzorů — kalibrované `.daq` + `.csv`

DAQ-U / DAQ-M / DAQ-E lze zaznamenávat **bez** jeho kalibračního balíčku. To je to,
co veřejně dostupné záznamníky [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
( (`record_daq.py`) ve výchozím nastavení: zapisují surová data ze senzorů a označují
soubor tak, aby Chloros načítal tovární kalibraci daného senzoru **podle sériového čísla** — nejprve z lokální mezipaměti
, poté z cloudu MAPIR — a aplikoval ji při importu.

Chloros Výsledek se zapíše zpět jako dva produkty na jeden záznam pod
`<project>/Light Sensor/`:

| Produkt | Co to je |
| --- | --- |
| `<name>_calibrated.daq` | Archiv, který lze znovu zpracovat — stejné schéma jako živý záznam, nyní deklarující balíček, který jej vytvořil. Jeho opětovný import **ne** kalibruje podruhé. |
| `<name>_calibrated.csv` | Spektrální ozáření v W/m²/nm na vlastní vlnové mřížce senzoru, jeden řádek na jedno měření, plus fotometrické sloupce (celkový výkon, fotopický/skotopický lux, PPFD a jeho modré/zelené/červené rozdělení, špičková vlnová délka). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Pouze senzory bez balíčku (DAQ-A).** Surové spektrální počty snímače — *ne* ozáření. Viz níže. |

`process()` provádí tento export jako jednu ze svých fází. **Nevyžaduje** snímky:
samostatně letící světelný senzor představuje plnohodnotný pracovní postup a takový projekt má ze své podstaty nulový
počet snímků.

**Záznamy DAQ-A se exportují jako surové počty.** Řada DAQ-A je starší než systém svazků
pro jednotlivé sériové čísla a nemá žádný svazek, který by bylo třeba načíst — místo toho se kalibruje v terénu proti
reflexnímu terči, a proto žádný svazek nikdy nepotřebovala. Tyto záznamy se exportují
pod kořenovým názvem `_raw` namísto `_calibrated`: jde o odlišný název souboru, nikoli o příznak
uvnitř souboru, protože tento údaj musí zůstat zachován i při odeslání e-mailem jako pouhý název. Záhlaví
`.csv` uvádí `raw spectral sensor counts (NOT irradiance)` a upozorňuje, že
hodnoty jsou srovnatelné **v rámci** souboru — přesně k tomu je využívá kalibrace založená na cíli
— a nikoli napříč senzory. Fotometrické sloupce závislé na výkonu (celkový výkon,
fotopický/skotopický lux, PPFD) vrací hodnotu **NULL** namísto integrace z počtů.

DAQ-U / DAQ-M / DAQ-E, jehož balíček se prostě nepodařilo načíst, je stále **přeskočen**,
nezapisuje se v surovém formátu: v takovém případě balíček existuje a „znovu se připojit a znovu zpracovat“ je skutečná rada.

Starší záznamy **v1.01 / v1.02** (DAQ-A-SD je zapisuje) neobsahují epochu pro jednotlivé odečty,
pouze čas zápisu souboru. Porovnávač obrazů↔světelného toku je stále odmítá — porovnání
snímku s časem zápisu by bylo neviditelně nesprávné — ale exportér je čte a
CSV vypíše `clock=daq_created_on`, takže produkt uvádí, na jakém časovači se nachází.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Záznam, jehož kalibrační balíček nelze načíst (offline nebo senzor bez
kalibrace v souboru) je nahlášen pod kódem `skipped` **s uvedením důvodu**. Nikdy není
zapsán jako „kalibrovaný“ soubor obsahující surová data – připojte se k internetu a
spusťte proces znovu, export se dokončí.

### Zpětná volání o průběhu

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Shrnutí po spuštění a tipy

Po dokončení `process()` načte `GET /api/processing-summary` a připojí tělo jako `result["summary"]`. Načítání probíhá podle principu „best-effort“ a nikdy neblokuje úspěšný návrat — pokud není souhrn k dispozici, `process()` se vrátí k jednoduchému formátu `{"status": "complete", "async": False}`. Každý záznam v `summary["hints"]` — úplné věty s navrhovaným řešením, např. proč běh vyprodukoval nulový výstup — je rovněž znovu odeslán jako Python `UserWarning`, takže běhy s nulovým výstupem se diagnostikují samy, i když slovník nikdy neprohlížíte:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` představuje strojově čitelnou část:

| Klíč | Co počítá |
| --- | --- |
| `models` | Skupiny kamer v běhu. |
| `images_in_groups` | Zdrojové snímky napříč těmito skupinami. |
| `targets_found` | Detekované reflektanční cíle. |
| `images_calibrated` | Snímky, na jejichž základě byl běh kalibrován. |
| `exported_files` | **Soubory s obrazovými výstupy, které běh vytvořil.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | Záznamy, záměrně počítané samostatně — pocházejí z jiné fáze a existují i u běhů, které vůbec neobsahují snímky, takže jejich zahrnutí by způsobilo, že by běh pouze s DAQ vypadal, jako by exportoval snímky. |

Vedle nich: `summary["output_dirs"]` (každý adresář, do kterého se zapisovalo),
`summary["light_sensor_export"]`, `summary["stopped"]` (platí, když uživatel přerušil
běh, takže částečné počty se nezobrazují jako dokončený běh s nedostatečným výstupem) a
`summary["groups"]` (rozpis podle skupin).

`exported_files` je zaznamenán potrubím **při zápisu**, není skenován z
objektů obrazu projektu až dodatečně. Paralelní a GPU strategie vytvářejí své vlastní obrazové
objekty (v podprocesech pracovníků pro GPU cesty), takže starý sken hlásil
`0 file(s) written` pro každý takový běh a poté vyslal nápovědu o nulových exportu — u běhů,
, kde vše fungovalo. Pokud píšete skripty na základě tohoto čísla, zdravý paralelní běh nyní
hlásí nenulový počet.

Přeskočení světelným senzorem hlásí důvod, který čtečka skutečně zjistila pro každý soubor —
nečitelnou strukturu, chybějící balíček, chybu zápisu — **bez duplicit**, takže dvacet souborů
přeskočených z jednoho důvodu se vyhodnotí jako jeden důvod, nikoli jako dvacet jeho opakování.

> **`process()` se nevyskytuje, když běh nevytvoří žádné snímky.** Toto je jediné místo, kde se SDK a
> CLI záměrně liší: `chloros-cli process` považuje „byly vyžádány výstupy, žádné nebyly
> zapsány“ za selhání a ukončí se nenulovým stavem, zatímco SDK se vrátí normálně a nahlásí
> tuto situaci prostřednictvím `summary` / hints. Pokud by se váš pipeline měl zastavit při prázdném běhu, zkontrolujte to
> sami — prozkoumejte `summary` (nebo spočítejte soubory ve složce projektu), místo abyste se spoléhali na
> nepřítomnost výjimky. Obvyklými příčinami jsou vstupní složka, která nebyla rozpoznána jako
> záznam, a výstupy přeskočené jako nepoužitelné pro přítomné kamery (např. radiance z kamer podporujících pouze
> RGB).

### Pomocné funkce

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### Podporované hodnoty

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Radiometrický výstup (multispektrální pipeline LATTICE)

Úroveň exportu multispektrálních dat (M3C/M3M) z potrubí `process` — `reflectance` (výchozí), `radiance`, `sensor-response` nebo `all` (každý použitelný režim pro každý snímek) — odpovídá nastavení **„Radiometrický výstup“** . `configure()` má pro to vyhrazené klíčové slovo:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Pokročilý únikový mechanismus — zápis klíče projektu `"Radiometric output"` prostřednictvím `custom_settings` — stále funguje, ale mějte na paměti, že nahrazuje celý blok nastavení (viz varování níže):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (výchozí nastavení) dělí radianci kamery **sestupným tokem DAQ s odpovídajícím časovým razítkem**, který se automaticky vypočítá ze zaznamenaného `.daq` (DAQ-U/M/E)**nebo nativního `.csv`**nalezeného společně se snímky; jakýkoli balíček kalibrace pro jednotlivé kamery nebo DAQ, který lokálně chybí, je**při prvním použití automaticky stažen z AWS**. Rozhraní CLI toto zpřístupňuje jako přepínače produktů podle typu na `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> `custom_settings` **nahrazuje** celý blok vypočítaných nastavení (záměrně obchází ostatní klíčová slova a ověření `configure()`). Při jeho použití zahrňte všechny klíče `Project Settings`, na kterých vám záleží, jako v příkladu výše.

---

## Smart-Connect pro kamery LATTICE

Trvalé relace backendu pro živý hardware. Používají se stejné koncové body jako v grafickém uživatelském rozhraní (GUI), takže chování je identické na SDK / CLI / v GUI.

### Jedna kamera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

#### `connect_camera()` Podpis

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

#### `CameraSession` Metody

| Metoda | Popis |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Čte uzly GenICam; vrací `{nodes, errors, enums, device}`. |
| `set_settings(**kwargs)` | Zapisuje uzly podle popisného jména (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | Zachytí **jeden** snímek. Vrátí seznam s jedním prvkem obsahující slovníky metadat snímku. (Sériové/více snímkové zachycování bylo odstraněno — pokud potřebujete sérii, zavolejte `capture()` ve smyčce.) |
| `disconnect()` | Uvolnění z fondu. Žádná operace, pokud jsme se připojili k již otevřené relaci. |

Ovládací prvky exportu `capture()` (stejný model jako pole + grafické rozhraní):

- `processing` / `levels` — `processing="all"` uloží všechny použitelné typy exportu; `levels=["raw","radiance"]` uloží pouze ty (přepíše `processing`). Vynechte obě pro výchozí nastavení backendu.
- `force_daq=True` — uloží přiřazené měření DAQ/DLS jako sidecar `.daq` i v případě zachycení pouze v surovém formátu, aby mohl být snímek později znovu zpracován na odrazivost/index. Žádná operace, pokud není propojen žádný DAQ.

### Synchronizované pole — `ArraySession` (Smart-Prep)

`connect_array` je **doporučený vstupní bod** pro konfigurace s více kamerami. V pozadí spouští kompletní proces Smart-Prep prostřednictvím grafického uživatelského rozhraní:

1. **Analýza sítě** (`/api/camera/array/recommend`) — vyhledá největší velikost snímku, která se vejde do vrstvy sim-emit bez ztráty snímků.
2. **Automatický výběr úrovně** — `sim-capture-sim-emit`, pokud to přenosová linka zvládne; jinak `sim-capture-ftd-stagger` nebo `slip-emit-and-capture`.
3. **Automatické zmenšení**— tiše zmenší velikost rámce / zvýší binning, když přenosová linka nedokáže udržet požadované rozlišení.**Tato bezpečnostní síť neřeší agregované přetížení**: příliš mnoho kamer pro danou přenosovou linku nelze vyřešit zmenšením rámců — viz [Přetížení](#over-subscription-the-per-cam-floor).
4. **PTP je ve výchozím nastavení zapnuto**— časová razítka napříč kamerami se synchronizují na jeden sdílený časový signál s přesností**~1 ms**. Současná expozice je zajištěna hardwarovým spouštěčem M8 (**&lt; 100 µs** mezi moduly), nikoli protokolem PTP: PTP synchronizuje *časová razítka*, nikoli expozice.
5. **Automatický výběr formátu pixelů pro každou kameru** — kamery RGB → `BayerRG8`, multispektrální → `BayerRG12`.
6. **Inicializace AE** — zaznamenává aktuální stav AE každé kamery, aby připojení během provozu neresetovalo expozici.
7. **Konfigurace spouštění přes GPIO** — `connect_array` aktivuje každou kameru (`TriggerMode=On`, `TriggerSource=Line2`), takže impuls z hlavní kamery řídí podřízené kamery přes kabel M8. Jedná se o krok určený pouze pro pole kamer: v případě jedné kamery se místo toho spustí `LatticeCamera` free-běží samostatně.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### `connect_array()` Podpis

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

Hodnoty `force_tier`:
- `"sim-capture-sim-emit"` — skutečně simultánní (všechny kamery se spouštějí na stejné hraně hodinového signálu).
- `"sim-capture-ftd-stagger"` — flexibilní časové rozložení (kamery vysílají s mírným časovým posunem, takže se pakety na vedení serializují).
- `"slip-emit-and-capture"` — sekvenční snímání podle jednotlivých kamer (bez časové synchronizace; jediná možnost, pokud žádná velikost rámce neodpovídá simultánnímu režimu).

`wire_ceiling_mbps` přepisuje **trvalý rozpočet hostitele na přenosovou linku** v MB/s — jediné
číslo, na kterém závisí celá alokace pole. Nechte hodnotu `None`, aby se použila automaticky detekovaná
hodnota. Snižte ji, pokud pole hlásí rámce poškozené GVSP: automatická hodnota je odvozena
z inzerované rychlosti spojení síťové karty, která u USB adaptérů, úzkých PCIe linek a
vytížených sdílených struktur — a tento nadhodnocený údaj se projeví spíše jako poškozené rámce než jako
viditelně pomalé spojení. Hodnota je uložena v bloku zachycování pole projektu, takže při
opětovném otevření nebo pozdějším `connect_array` se obnoví stejně jako jakékoli jiné nastavení pole.
Viz [Stav pole](#array-health--which-subsystem-is-losing-frames).

#### Překročení kapacity (minimální limit na kameru)

Funkce Sim-emit pacing přiděluje každé kameře podíl z rozpočtu pro přenos bez kolizí, s minimální hodnotou **8 MB/s na kameru**(`per_cam_floor_bps`). Jakmile `N × floor` překročí horní hranici bezpečnou proti kolizím, pole**překročí kapacitu sítě**— režim selhání představuje ztráta paketů GVSP, nikoli nižší snímková frekvence — a neexistuje žádné řešení založené na velikosti snímku:**binning a ROI snižují počet bajtů na snímek, nikoli počet bajtů za sekundu s regulovaným tempem**, které porovnává agregovaná kontrola. Praktické stropy pro plné rozlišení na hostiteli s 1 GbE:**6 kamer s MTU 1500, 9 s jumbo rámci** (`max_cams_collision_safe` v odpovědi analýzy uvádí horní hranici pro vaše připojení). Řešení: méně kamer, jumbo rámce v celém řetězci nebo rychlejší síťová karta.

- Odpovědi `analyze_array_network()` a `/api/camera/array/connect` obsahují `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` a `per_cam_floor_bps`. Pokud je hodnota `oversubscribed` pravdivá, projekce **vynuluje pole fps** (`achievable_fps_max` / `fps_bright` / `fps_dark`), místo než aby hlásila zavádějící hodnotu, která je sice pomalá, ale funkční.
- `POST /api/camera/array/connect` přijímá parametr těla `pin_resolution` (**pouze HTTP — nejedná se o klíčový argument SDK**; `connect_array` jej nezpřístupňuje). Fixace odstraňuje bezpečnostní síť v podobě snižování binningu, takže připojení s nadměrnýmpočet předplatitelů s nastaveným `pin_resolution`**tvrdě odmítnuto** s chybou uvádějící všechny možné nápravné kroky. Bez fixace spojení pokračuje v postupném snižování, ale varuje, že zmenšení nemůže vyčistit agregát.
- Únikový východ pro testování: nastavte v prostředí backendu `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1`, aby se odmítnutí snížilo na hlasité varování — připojíte se i tak a akceptujete ztrátu paketů.

#### Stav pole — který subsystém ztrácí rámce

`GET /api/camera/array/<array_id>/capability` nese aktivní blok `health` na
připojeném poli, který je přehodnocován v průběžném **10sekundovém** okně. Rozděluje ztrátu rámců
na dvě příčiny, které vyžadují opačná řešení, namísto jedné míry „neúplnosti“, která
žádnou z nich nespecifikuje:

| Pole | Co to znamená | Který subsystém |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (podle sériového čísla) | Snímek **dorazil a byl strukturálně poškozen**— ztráta paketů GVSP. |**Síť**: kapacita linky, tempo přenosu, kruh přijímače síťové karty, MTU |
| `never_arrived_rate_pct` (podle sériového čísla) | Rámec **vůbec nedorazil**— kamera se nespustila, nebo z ní nic nevyšlo. |**Spoušť / synchronizace**: kabel M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Nejhoršípro každý případ. | — |
| `per_cam_rate_pct` | Kombinovaná míra neúplnosti na kameru (obě příčiny dohromady). | — |
| `stable_for_seconds` | Jak dlouho zůstala každá kamera pod 0,01 %. | — |

Vedle položky `health` uvádí stejný záznam také číslo, na kterém závisí celá alokace:

| Pole | Co to znamená |
| --- | --- |
| `wire_ceiling_mbps` |trvalý platný rozpočet šířky pásma, MB/s. |
| `wire_ceiling_source` | Odkud tato hodnota pochází, slovy — např. `USB-capped 200 MB/s (was theoretical 1062; …)` nebo `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, když jej nastavil `wire_ceiling_mbps=`. |
| `nic_is_usb` | `true` pro USB ethernetový adaptér. |

Pro tento koncový bod neexistuje žádný SDK wrapper — čtěte jej přímo:

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Čtení:** nenulová hodnota `gvsp_corrupt_rate_pct` s hodnotou `never_arrived_rate_pct` rovnou 0 znamená,
že spouštění a synchronizace kabelu jsou v pořádku a 100 % ztráty je na síťové trase — snižte
`wire_ceiling_mbps` a znovu se připojte. Opačný vzorec naopak ukazuje na synchronizační kabel nebo
spouštěcí linku.

> **Hodnota `target_fps` není rozhodujícím faktorem pro poškozené rámce.** Frekvence GevSCPD se nastavuje jednorázově při
> připojení, takže snížení spouštěcí frekvence mění pracovní cyklus, nikoli
> rychlost souběžného vysílání. Změřená 5× redukce požadavků nepřinesla žádné zlepšení, zatímco
> snížení horní hranice linky z 240 na 200 MB/s snížilo u stejného zařízení podíl poškozených dat z 10,4 % na
> 0,00 %.

> **Automatické zmenšení uprostřed přenosu není ve firmwaru TRI032S k dispozici.** Běžící pole nemůže
> tento problém samo vyřešit; odpojte a znovu připojte zařízení, aby nástroj pro výběr času připojení znovu naplánoval přenos podle
> nové horní hranice.

**USB ethernetový adaptér je sondou omezen na 200 MB/s** bez ohledu na jeho
typové označení: tabulka účinnosti, která převádí rychlost spojení na trvalou hodnotu, je
odvozena od PCIe a USB síťová karta inzeruje svou rychlost ethernetového spojení, přičemž je omezena
USB sběrnicí a jejím ovladačem. Omezení je absolutní, nikoli relativní — USB adaptér 1 GbE
dosahuje rychlosti ~80 MB/s a není tímto omezením ovlivněn.

#### Metody `ArraySession`

| Metoda | Popis |
| --- | --- |
| `status(timeout=10.0)` | Spustí `{fps, ptp, frame_count, last_error, …}`. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | Jedna synchronizovaná skupina snímání. Vrací `CaptureResult` (seznam slovníků rámců + `.skipped`). Ovládací prvky pro export níže. |
| `capture(..., smart=True)` | **Inteligentní snímání** — počká, až se AE ustálí na všech kamerách, a poté spustí snímání. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Nejrychlejší snímání: pouze surová data + přiřazená hodnota DAQ (+ volný kombinovaný index). Odpovídá tlačítku „Fastest Capture“ v grafickém uživatelském rozhraní. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Jednorázové / kontinuální / intervalové snímání v jedné ohraničené smyčce. Vrací `list[CaptureResult]`.**Vyžaduje `count` a/nebo `duration_s`**, aby bylo možné proces ukončit (SDK nemá klávesovou zkratku Ctrl+C). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Spustí nahrávání živého zobrazení kombinovaného indexu do formátu video/GIF → `RecorderHandle`. Jeden kompozitní záznamník na jedno pole. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Spuštění sériového snímání v surovém formátu Bayer s vysokou snímkovou frekvencí → `RecorderHandle`. Offline přepracování pomocí `build_video()`. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Offline zpracování uloženého sériového snímání ve formátu RAW do kalibrovaného videa(s). Blokuje se až do dokončení (`wait=True`) a vrací `{outputs, errors, combined}`. |
| `build_video_status(job_id, timeout=15.0)` | Zjišťuje stav offline úlohy sestavení: `{running, result, error, burst_dir}`. |
| `disconnect()` | Uvolní celé pole. |

`capture()`vládací prvky exportu  (stejný koncový bod, jaký používá grafické rozhraní/CLI):

- `processing` / `levels` — `processing="all"` (nebo `levels=["raw","radiance",…]`) uloží všechny příslušné typy exportu pro každou kameru; jedna hodnota `processing` uloží pouze danou úroveň.
- `aligned=True` — deformuje ne-surový export každého člena podle [profilu zarovnání](#array-alignment) pole (spoluregistrovaného); surová data zůstávají bez zarovnání, ale v metadatech nesou transformaci. Pokud pole nemá žádný profil, použije se výchozí nezarovnané zarovnání (s varováním zobrazeným v `alignment` výsledku).
- `render_index=False` — přeskočit překryv indexu vegetace; ve výchozím nastavení se vykresluje tam, kde je nakonfigurováno.
- `force_daq=True` — uloží přiřazenou hodnotu DAQ/DLS jako soubor sidecar `.daq`, i když to žádná zvolená úroveň nevyžaduje.

**KompreseTIFF (pouze přepínač HTTP):**`ArraySession.capture()` neodesílá žádný klíč `compression`, takže se použije výchozí nastavení backendu — `POST /api/camera/array/capture` čte parametr těla `compression`, `"deflate"` ve výchozím nastavení (bezztrátová komprese zlib L1 + horizontální prediktor, ~4,1 MB na snímek v plném rozlišení). `"none"` zapisuje nekomprimované údaje (~6,3 MB/snímek) s**~5× rychlejším zápisem** — oba jsou bezeztrátové a při importu se čtou identicky. Rozhraní SDK pro to neposkytuje žádný argument; únikovou cestou je `chloros-cli lattice array-capture --compression none` nebo surový HTTP. DEFLATE také drží GIL pro Python, takže komprimované zápisy nelze paralelizovat napříč jednotlivými— pro nepřetržité snímání v plném rozlišení s 8 kamerami při rychlosti snímače je třeba použít `compression: "none"`. Podrobnosti: [CLI Reference → array-capture](cli-reference.md).**Přepsání exportu pro jednotlivé členy (pouze HTTP):**stejný koncový bod přijímá také `exclude_serials` (seznam — odstranění prvků z uložené sady; pole se stále spouští jako jedna synchronizovaná skupina a vyloučené prvky jsou vráceny v `excluded`), `serial_levels` (přepsání na úrovni jednotlivých kamer `{serial: [level tokens]}`) a `serial_index` (přepsání překryvů indexů pro jednotlivé kamery v `{serial: bool}`). Jedná se o parametry těla s paritou s grafickým uživatelským rozhraním a**zatím ne o klíčové argumenty typu „SDK“**; členy, které v mapách chybí, se vrátí k celopoleovým parametrům `levels` / `render_index`.

##### Kontrola přeskočených kamer — `CaptureResult.skipped`

`ArraySession.capture()` vrací `CaptureResult`, což je podtřída `list`: iterujte přes ni, indexujte ji, `len()` — všechny stávající vzory fungují i nadále. Nový kód může zkontrolovat atribut `.skipped`, aby zjistil, které kamery byly vyloučeny a proč. Nejběžnějším případem jsou kamery typu RGB ve smíšeném filtrovém poli, když požadujete `processing="radiance"` nebo `"reflectance"` — zářivost na jeden Bayerův pixel nemá u širokopásmového snímače smysl, proto backend tyto kamery přeskočí, místo aby generoval nesmyslné údaje.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Tokeny důvodů mají vzorec `<level>-not-applicable-to-rgb-cam` (jeden záznam pro každou přeskočenou úroveň, každý s hodnotou `level`). Přeskočení specifická pro odrazivost jsou `reflectance-skipped-no-fresh-dls` (není k dispozici nové měření dopadajícího záření), `reflectance-skipped-bound-daq-unavailable (…)` (nelze se připojit k připojenému DAQ)a `dls-uncalibrated-band-<nm>` — pásmo leží převážně mimo radiometricky kalibrovaný rozsah světelného senzoru DAQ (~374–974 nm), takže absolutní dělení odrazivosti založené na DAQodmítnuto a snímek je výrazně degradován na odezvu snímače. Z dodávaných modelů to spouští pouze F988; podporovaným postupem pro tuto kameru je pracovní postup s odrazivostním panelem.

Úrovně `processing`:

| Úroveň | Výstup |
| --- | --- |
| `"raw"` | Jednokanálový Bayer (mono kamery: jedno pásmo) přímo ze snímače. |
| `"debayered"` *(výchozí nastavení SDK)* | 3kanálový BGR prostřednictvím bilineárního demosaiku (černobílé kamery: 1kanálová šedá stupnice). |
| `"radiance"` | float32 W/m²/sr/nm prostřednictvím úplného radiometrického řetězce. Pouze multispektrální — kamery typu „RGB“ jsou přeskočeny. |
| `"reflectance"` | uint16 0..32768 (připraveno pro Pix4D); vyžaduje živé spárování s DAQ pro absolutní referenci. Pouze multispektrální. |
| `"display"` | Kompletní řetězec odpovídající náhledu v grafickém rozhraní (CCM + WB + gama podle profilu kamery). |
| `"all"` | **Jeden soubor pro každou příslušnou úroveň** pro každou kameru (odpovídá výchozímu nastavení GUI „Capture All“ / CLI). Vrácený soubor `CaptureResult` pak obsahuje jeden slovník snímků pro každý soubor `(cam, level)`, přičemž úroveň je uvedena v každém slovníku; nepoužitelné úrovně se objevují v souboru `.skipped`. Hodnota z DAQ použitá pro jakýkoli rámec odrazivosti, se ukládá jako sidecar `.daq`. |

> **Poznámka — výchozí nastavení se liší od CLI.** Výchozí hodnotou pro `ArraySession.capture()` je `processing="debayered"`; výchozí hodnotou pro příkaz `chloros-cli lattice array-capture` je `processing="all"`. Zadejte explicitně `processing="all"` z SDK, abyste napodobili víceúrovňové ukládání CLI /GUI.

### Režimy snímání a záznamníky

Rozhraní pole odráží panel snímání v grafickém uživatelském rozhraní (GUI): režimy Single (jednorázový), Continuous (kontinuální), Interval (intervalový) a Fastest shutter (nejrychlejší závěrka), plus dva záznamníky (živý kompozitní video signál a surová série snímků → offline zpracování).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**je smyčka režimů Continuous/Interval v nástroji SDK. Jelikož neexistuje `Ctrl+C`, který by ji ze skriptu přerušil,**musíte** předat `count` a/nebo `duration_s` (smyčka se zastaví, jakmile je dosaženo jedné z těchto hodnot). `interval_s` se měří od začátku každého průchodu (stejně jako v grafickém rozhraní). Zbývající kwargs se předávají přímo do `capture()`.
- **`record`** je *monitorovací*: zachycuje živý kompozitní obraz kombinovaného indexu tak, jak se zobrazuje, takže kombinovaný proud musí být otevřený, aby se do něj mohly ukládat snímky. Jeden záznamník kompozitního signálu na pole (vyvolá výjimku, pokud již jeden běží).
- **`burst` → `build_video`** je určen pro *analýzu*: `burst` zapisuje surové snímky + manifest pro každý snímek + jeden `.daq` pro každou odlišnou hodnotu DLS v rámci `<output>/bursts/<base>/` při plné rychlosti snímací smyčky (bez řetězce, bez nástroje exif, bez živého náhledu). `build_video` časově přiřadí každý snímek k nejbližšímu `.daq` a znovu spustí řetězec radiance/reflektance/indexu importního potrubí. `products` je seznam `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (výchozí: kombinovaný index). `burst().stop()` také automaticky spustí sestavení kombinovaného indexu s maximálním úsilím, jehož výsledek je vrácen jako `build_job` ve výsledku zastavení.

#### `RecorderHandle`

Vráceno funkcemi `ArraySession.record()` a `ArraySession.burst()`. Použijte jej jako správce kontextu k automatickému zastavení při opuštění rozsahu, nebo jej ovládejte ručně.

| Člen | Popis |
| --- | --- |
| `job_id` | ID úlohy backendu (řetězec). |
| `kind` | `"composite"` (z `record`) nebo `"raw"` (z `burst`). |
| `start_stats` | Slovník vrácený voláním `start`. |
| `result` | `None` během běhu; konečný slovník výsledků zastavení po zastavení. |
| `stats(timeout=10.0)` | Aktuální statistiky úlohy (zapsané snímky, dosažené fps, uplynulý). |
| `stop(timeout=60.0)` | Zastaví záznam; vrátí a uloží do mezipaměti konečný výsledek. Idempotentní (druhé volání vrátí výsledek z mezipaměti). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### Připojení k již připojenému poli — `attach_array`

Pokud je pole již spuštěno (otevřelo jej grafické rozhraní nebo předchozí relace SDK volala `connect_array`), použijte `attach_array` k získání jeho identifikátoru namísto opětovného připojení. `connect_array` <sn><id>v této situaci</id></sn> vždy vyhodí chybu „Kamera  <sn>je již v poli<id>“, protože odeslání požadavku POST s `/array/connect` pro člena v-pool není idempotentní; `attach_array` čte `/api/camera/array/list` a porovnává podle array_id nebo sériových čísel.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Vzor: Skripty SDK, které sdílejí prostor s desktopovým grafickým rozhraním, by měly nejprve zkusit `attach_array` a přejít na `connect_array`, pokud v poolu ještě není žádné pole.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Důležité — ukončení context-manageru spojení ODPOJÍ.**`ArraySession.disconnect()` vždy odesílá POST na `/array/disconnect`; neexistuje žádná připojenástrážce typu „-not-owned“, jako je tomu u `CameraSession` / `DAQSensorSession`. Pokud používátes grafickým rozhraním a nechcete pole při ukončení rozsahu zrušit,**nepoužívejte blok `with`** — uložte popisovač do běžné proměnné a přeskočte explicitní `disconnect()`:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Pomocník pro analýzu sítě

Užitečné před otevřením pole — předpovídá, zda se vaše navrhovaná nastavení vejdou:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

`status` je jedním z `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (jinak `error`). `auto_capped_fps` znamená, že požadované rozlišení vyhovuje prstenci RX pouze při omezené frekvenci spouštění — zachovejte rozlišení a předejte `target_fps=result["recommended"]["recommended_target_fps"]` na `connect_array` (viz [Příklad 6](#6-capability-probe-before-connecting-a-4-cam-array)).

**Jak číst projekci** (stejný model jako panel Nastavení pole v grafickém rozhraní):

- **Sériový záznam (`frame_bytes_total`) se sčítá pro každou kameru v reálném formátu pixelů dané kamery.**Mono**M3M**kamery streamují Mono12 (2 B/px) bez ohledu na předaný parametr `pixel_format`, takže snímek v plném rozlišení ze 4 kamer má velikost**~25 MB** se třemi mono kamerami, nikoli ~12,6 MB, jak by vyplývalo z předpokladu, že všechny kamery používají 8bitový formát. Backend určí formát každé kamery na základě jejího modelu.
- **Admittance (`burst_fits_nic_ring`) zohledňuje odběr**, nikoli porovnání celého burstu s-prstenec: sim-emit funguje, když host vyprázdňuje přijímací prstenec rychleji, než jej kamery naplňují. 10G hostitel + 1 GbE kamery**přijímají** plné rozlišení, i když burst přesahuje kapacitu prstence; 1 GbE hostitel blokuje (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` představuje konzervativní horní hranici sériového načítání** — `max(readout+emit, N×emit)` s vysíláním na kameru omezeným na kamerové rozhraní 1 GbE, nezávislý na expozici. Např. ~2,8 fps pro pole se 4 kamerami v plném rozlišení a 12 bitech (odpovídá naměřeným ~2,7–3,0 v běhu). Úplný model: [CLI Reference → Model snímkové frekvence pole a sériového snímání](cli-reference.md#array-fps--burst-model).
- **Nadměrné předběžné přidělení (`oversubscribed: true`) znamená, že minimální hodnota N × na kameru překračuje strop bezpečný z hlediska kolizí** — pole snímkové frekvence (`achievable_fps_max` / `fps_bright` / `fps_dark`) vykazují hodnotu 0 a automatické zmenšení/binning to nedokáže vyřešit (tyto funkce snižují počet bajtů na snímek, nikoli počet bajtů za sekundu v rovnoměrném tempu). Řešením je snížení počtu kamer, použití jumbo rámců nebo rychlejší síťová karta; `max_cams_collision_safe` hlásí horní limit (6 kamer v plném rozlišení na 1 GbE při 1500 MTU, 9 s jumbo rámci). Odpověď obsahuje také kódy `aggregate_demand_bps`, `collision_safe_ceiling_bps` a `per_cam_floor_bps` (8 MB/s). Viz [Překročení kapacity](#over-subscription-the-per-cam-floor).

### Objevování a výpis

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

Pole LATTICE spouštějí nepřetržité automatické nastavení expozice (AE) na pozadí, jakmile jsou připojena, ale u nově nasměrované scény trvá chvíli, než se expozice ustálí. **Smart-Capture** je praktické řešení: zjišťuje expozici každé kamery, počká, až se pole v daném okně ustálí, a poté spustí snímání. Je to ekvivalent grafického rozhraní: tlačítko „smart“ v desktopové aplikaci volá stejný backendový endpoint.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Při ovládání přes `ChlorosProject` (další sekce) máte k dispozici více nastavení:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Zásady inteligentní automatické expozice (smart-AE) jsou ve výchozím nastavení konzervativní. Pro náročnou radiometrickou práci nastavte přísnější hodnoty pomocí `exposure_tolerance_pct`; pro rychle se měnící scény, kde vám stačí „dostatečná přesnost“, nastavte volnější hodnoty.

---

## Relace senzorů DAQ

Trvalý fond backendů pro spektrální senzory (DAQ-U přes USB, DAQ-M přes BLE, DAQ-E přes Ethernet). Odráží chování kamery: inteligentní detekce, opakované použití fondu, idempotentní připojení.

### Inteligentní detekce (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Priorita: Ethernet → BLE → USB. Zadejte libovolný explicitní pokyn k fixaci transportu.

### Zafixovaný přenosový kanál

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### Metody `DAQSensorSession`

| Metoda | Popis |
| --- | --- |
| `status(timeout=10.0)` | Souhrn položky fondu (stav streamování/záznamu, rozsah vlnových délek, kalibrační SHA, doba integrace, frame_avg, stav AE). |
| `latest(n=1, timeout=10.0)` | Vrátí až N nejnovějších spektrálních rámců. |
| `stream_start()` / `stream_stop()` | Obnoví / pozastaví streamování (handle zůstává otevřený). |
| `record_start(output_dir=None, device_name=None)` | Spustí záznam souboru .daq. Vrátí cestu k souboru. Nepovolí se u DAQ-U/M bez kalibračního balíčku AWS (DAQ-E je výjimkou). |
| `record_stop()` | Zastaví záznam. Vrátí `{path, rows}`. |
| `disconnect()` | Uvolní z fondu. Bez účinku pro připojené, ale nevlastněné handle. |

> **Profily korekce horní hranice (`cap_id`) nejsou ovládacím prvkem typu „SDK“.** `connect_daq_sensor()` / `DAQSensorSession` neobsahují žádný parametr `cap_id` ani metodu `set_cap`. Profil korekce limitu flotily vyberte prostřednictvím CLI (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) nebo trasy typu „HTTP“ (`/api/daq`) v backendu (`/api/daq/connect` a `/api/daq/<id>/cap-id` akceptují `cap_id`).

### Vyhledávání — nalezení adresy pro připojení

`discover_daq_sensors()` prohledává rozhraní USB / BLE / ETH a hledá senzory, které *by* bylo možné otevřít. Jedná se o ekvivalent `discover_lattice_cameras()` pro DAQ a jediný způsob, jak získat **BLE MAC adresu zařízení DAQ-M** — zařízení DAQ-E má název hostitele a DAQ-U má COM port, ale MAC adresa není ani vytištěna na zařízení, ani uvedena v seznamu operačního systému.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Pole | Popis |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM port / BLE MAC / název hostitele — předat do `connect_daq_sensor` jako `port=` / `mac=` / `eth_host=`. |
| `display` | Člověkem čitelný popisek. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` nebo `None` pro port, který skenování nedokáže identifikovat (sériové adaptéry USB nelze bez sondy rozlišit, proto jsou neznámé položky zobrazeny, nikoli skryty). |
| `extra` | Podrobnosti podle typu přenosu (inzerovaný název BLE, výrobce USB, IP/fw/… zařízení DAQ-E). Prázdné hodnoty jsou vynechány. |

| Parametr | Výchozí hodnota | Popis |
| --- | --- | --- |
| `transports` | všechny tři | Sekvence (nebo řetězec ve formátu CSV) omezující skenování. Stojí za to zadat, pokud víte, co chcete — BLE je nejpomalejší část. |
| `scan_timeout` | 5 | Okno skenování pro jednotlivé transporty v sekundách; backend omezuje na 1–20. |
| `timeout` | 60,0 | Horní hranice „HTTP“ pro celé volání (stejně jako jinde v „SDK“). |
| `auto_start_backend` | `True` | Spustí lokální backend, pokud žádný neběží. Nikdy se nespustí pro vzdálený `backend_url`. |

> **Senzory, které jsou již otevřené v poolu se nezobrazí.** Připojené periferní zařízení BLE přestane vysílat a otevřený COM port nelze prozkoumat, takže vyhledávání uvádí pouze to, co je *k dispozici pro připojení*. Hned po připojení zařízení je očekáván prázdný výsledek — pro zařízení, která již máte, použijte `list_daq_sensors()`. Transporty, u nichž nelze spustit skenování (není nainstalován bleak / zeroconf), jsou přeskočeny, místo aby vyvolávaly chybu, takže zařízení bez Bluetooth stále dostává odpovědi pro USB a ETH.

### Seznam

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Spolupráce s GUI / CLI

Pokud má GUI již otevřený senzor, volání `connect_daq_sensor(port="COM3")` z Python vrátí handle označený jako `already_connected=True`. `disconnect()` dané relace pak neprovede žádnou akci, takže váš skript SDK neodstraní senzor z pod GUI při ukončení relace .

### Třídy pro přímé ovládání hardwaru (bez backendu)

`daq_sdk` jeexportován funkcí `chloros_sdk`, takže můžete senzory ovládat také přímo v procesu bez backendu:

> **Dostupnost:**`daq_sdk` je součástí instalace desktopové verze Chloros,**není** s balíčkem z PyPI — `pip install chloros-sdk` vám poskytuje `lattice_sdk`, ale vynechává `chloros_sdk.DAQ_AVAILABLE == False`. Před použitím těchto tříd si tento příznak zkontrolujte; na hostitelském počítači, kde je k dispozici pouze pip, ovládejte senzor místo toho pomocí [`connect_daq_sensor()`](#daq-sensor-sessions), který nevyžaduje žádné lokální transportní knihovny.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

Upřednostňujte cestu smart-connect (`connect_daq_sensor`) , pokud chcete sdílené vlastnictví se grafickým uživatelským rozhraním; pro skripty bez grafického rozhraní, které vlastní senzor výlučně, použijte přímé třídy.

---

## Automatizace projektu — `ChlorosProject`

Uložený projekt Chloros je složka obsahující soubory `cameras.json` + `sensors.json` + `project.json`. `open_project` načte manifest a `connect_all` připojí všechna uložená zařízení s jejich uloženými nastaveními — ve stejném hardwarovém stavu, jaký by vytvořilo grafické uživatelské rozhraní.

### Minimální příklad

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

Nebo jako správce kontextu:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### Metody `ChlorosProject`

| Metoda | Popis |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Objeví a připojí všechna uložená zařízení. Vrátí zprávu o připojení pro každou třídu. Používá spuštěný backend, pokud nějaký naslouchá na `127.0.0.1:5000`; v opačném případě se bez upozornění přepne na přímé (bez backendu) ovládání zařízení `lattice_sdk` — nikdy nespouští backend. |
| `disconnect_all()` | Ukončí vše. |
| `capture_all(output_dir=".")` | Jeden snímek z každé kamery + pole + spektrum z každého senzoru. |
| `stream(camera, overlays=False, fps=10.0)` | Generátor poskytující snímky BGR `numpy` z pojmenované kamery (nebo pole). `overlays=False` je přímá smyčka snímání `lattice_sdk` (pole generují slovníky `{serial: frame}`). `overlays=True` směřuje přes `ChlorosLocal.camera_stream()` → MJPEG kanál backendu `/api/camera/<serial>/stream-annotated`, přičemž uložený blok kamery `ui.overlay` blokem kamery předaným jako parametry dotazu. Vyžaduje režim backendu a **samostatnou kameru**: kamera v přímém režimu vyvolá `RuntimeError` (backend nemůže získat kameru, kterou vlastní tento proces) a pole vyvolá `NotImplementedError` (překryvy kompozitu podle kamery — streamování člena podle jména). Ekvivalent jednorázového volání: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Spustí zarovnání na každém aktuálně připojeném poli. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Spustí kalibrační / indexovací pipeline na snímcích projektu (zahrnuje `ChlorosLocal.process`; tyto čtyři jsou **jediné** povolené klíčové argumenty — `indices=` atd. vyvolají chybu `TypeError`; nastavte indexy pomocí `ChlorosLocal.configure()`). Odloženě vytvoří `ChlorosLocal()`, který automaticky spustí backend. |

Atributy:
- `proj.cameras` — `Dict[str, CameraHandle]` indexovaný podle názvu A sériového čísla.
- `proj.arrays` — `Dict[str, ArrayHandle]` indexovaný podle názvu A array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` s klíči podle názvu A slot_id.
- `proj.config` — `project.json["config"]` slovník.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Úrovně zpracování.** `capture()`, `grab()` a `frame_stream()` všechny přijímají stejný token `processing`
a řetězec je kumulativní — každá úroveň spouští vše, co je nad ní:

| Úroveň | Výstup | Poznámky |
| --- | --- | --- |
| `raw` | 1kanálový Bayer, nativní pro snímač | Bez demosaiku. Na této úrovni nejsou k dispozici překryvy. |
| `debayered` | 3kanálový BGR (**výchozí**) | Bilineární demosaikování. Jediná úroveň, která funguje bez režimu backendu. |
| `radiance` | float32, W/m²/sr/nm | Úplný radiometrický řetězec: demosaikování + 3×3 separace (multispektrální) + DSNU + flat-field + NIST stupnice, s odečtením expozice × zesílení, takže hodnoty jsou absolutní. |
| `reflectance` | uint16, 32768 = 1,0 | Radiance dělená dopadajícím zářením (ρ = π·L/E). Vyžaduje údaj z DLS/DAQ — viz poznámka níže. |
| `display` | 8bitový (podobný sRGB) | Zobrazení ekvivalentní grafickému rozhraní: CCM + vyvážení bílé + gama prostřednictvím aktivního barevného profilu kamery. |

Cokoli jiného než `debayered` vyžaduje režim backendu; kamera v přímém režimu vyvolá
`NotImplementedError`. `reflectance` vyžaduje použitelné měření dopadajícího záření — koncový bod snímku automaticky vtáhne
shromážděná data DAQ do slotu DLS kamery, ale bez přiřazeného DAQ řetězec odmítne
výstup odrazivosti a poctivě označí snížení kvality ve vrácených metadatech, místo aby tiše
vrátil produkt nižší kvality.

> **Stupnice DN odrazivosti — neukládat pevně do kódu.** Odrazivost LATTICE používá `32768` = ρ 1,0 a označuje
> XMP `Chloros:PixelScale=32768`; odrazivost Survey3 používá `65535` = ρ 1,0 a neobsahuje žádné
> značky `Chloros:*`. Přečtěte značku a vydělte jí. Je definována v doméně uint16, takže zůstává
> `32768` pro každý formát, který mění měřítko (16bitové TIFF, 8bitové PNG /JPG, 32bitové procenta) — nejprve normalizujte
> uložený datový typ zpět na uint16 (×257 z 8bitového, ×65535 z typu float). Jediná výjimka:
> záznam z 8bitového zdroje uložený jako 8bitový TIFF je *oříznut*, nikoli přepočítán, takže jej žádný měřítko nepopisuje
> — Chloros v takovém případě zcela vynechá `PixelScale` a tuple MicaSense. Chybějící
> značku v souboru odrazivosti LATTICE považujte za „žádné platné měřítko“, nikoli za výchozí hodnotu.

> **EXIF se přenáší do exportu.** `process()` zkopíruje blok GPS zdrojového záznamu
> **a jeho ExifIFD** do každého výstupu, takže exporty obsahují `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` a `CameraSerialNumber`, stejně jako
> georeferencování. `FocalLength` je hodnota, ze které Pix4D vypočítává vzdálenost vzorku na zemi — bez ní
> by rekonstrukce vykazovala zcela nesprávné měřítko (v jednom změřeném případě se lokalita o rozloze 411 m
> na lokalitu o rozloze 47,8 km). Kopie záměrně neobsahuje soubor `-all:all`: strukturální značky IFD0 narušují
> výstup LATTICE a soubory `ExifImageWidth`/`Height` jsou vyloučeny, protože popisují
> zachycení zdroje, nikoli exportovaný rastr.

Podvlajky fáze snímání (vztahují se na radiometrické úrovně — `radiance`, `reflectance`, `display`):

| Vlajka | Výchozí hodnota | Význam |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + flat-field + 3x3 unmix + radiometrická stupnice NIST. |
| `apply_white_balance` | `True` | WB LUT. Zohledňuje DLS, pokud je k kameře připojen DAQ. |
| `apply_index` | `False` | Vyhodnocení vegetačního indexu. |
| `index_expression` | `None` | Přepsání vzorce. Prázdnéprázdné → automaticky aktivuje index. |
| `annotated` | `False` | Překrytí dekorací GUI (zebra/mřížka/peaking). Není k dispozici pro `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Typ návratové hodnoty je `CapturePathMap`, nikoli `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` je `Dict[str, Union[str, List[str]]]`: jednovrstvýúrovně
> `processing` přiřadí každému sériovému číslu jednu cestu, zatímco víceúrovňový (`"all"` nebo
> explicitní seznam `levels`) mu poskytuje **seřazený seznam** všech produktů uložených pro danou
> kameru. Živý kombinovaný kompozit, pokud by byl streamován, se objeví pod dodatečným
> `"combined"`, nikoli pod sériovým číslem. Kód, který předpokládá `str`, selže v
> seznamovém formátu, aniž by proti tomu nějaký typový kontrolor namítal — anotace uváděla `Dict[str, str]`
> ještě nějakou dobu po vydání formuláře seznamu, a proto tento alias existuje. Normalizujte
> v případě, že chcete plochý formát:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Zarovnání pole

`ArrayHandle` zpřístupňuje celou zarovnávací plochu. Profily jsou ve výchozím nastavení platné pouze pro danou relaci — pro trvalé uložení explicitně zavolejte `export_alignment()`.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Zarovnání při připojení

`connect_all(align=...)` dokáže automaticky zarovnat každé pole při připojení:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Pokud není zadáno, použije se jako výchozí `project.json["config"]["auto_align_on_connect"]`.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## Přímý hardware (bez backendu)

Pokud chcete nulovou závislost na backendu (CI, roboty bez grafického rozhraní, vestavěné systémy), importujte přímo `lattice_sdk` a `daq_sdk` — oba jsou znovu exportovány pomocí `chloros_sdk`. Omezení na `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`: `lattice_sdk` je součástí balíčku PyPI (vyžaduje však přítomnost runtime prostředí Arena SDK), zatímco `daq_sdk` je dodáván pouze s instalací pro stolní počítače.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Předvolby a spouštěč

Tři ze čtyř předvoleb fungují v režimu **free-run**: kamera exponuje nepřetržitě a
`capture()` vrátí další snímek. `triggered` je výjimkou — připraví
kameru na hardwarový signál na lince 2, takže nic nezachytí, dokud signál nepřijde.

| Předvolba | Spouštěč | Použijte, když |
| --- | --- | --- |
| `default` | volný běh | obecné použití |
| `high_speed` | volný běh | 8 bitů, omezení na 60 fps, krátká expozice |
| `high_quality` | volný běh | 12 bitů, bez omezení fps — obvyklá volba pro statické snímky |
| `triggered` | **připraven, linka 2** | fotoaparát je připojen k synchronizačnímu kabelu M8 a spouští ho něco jiného |

Pokud zvolíte `triggered` (nebo si sami nastavíte `trigger_mode="On"`) a nic
neřídí Linku 2, každý `capture()` vyprší — správně, protože jste
kameru požádali, aby čekala. SDK to vysvětluje, když k tomu dojde; viz
[SC_ERR_TIMEOUT během snímání](#direct-hardware-backend-free).

> **Poznámka — Zprávy „GVSP probe“ / zprávy `SC_ERR_TIMEOUT -1011` při připojení nejsou chyby.**&gt; Při připojení se SDK pokusí vyjednat**jumbo rámce** (9000-byte GVSP pakety) pro vyšší propustnost. Na přímém spojení-k-bodovém propojení síťových karet (např. adresa `169.254.x.x` v rámci lokální sítě) síť obvykle nedokáže přenášet jumbo rámce, takže tato sonda vyprší a do protokolu se zapíší řádky jako:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Toto je **předem navržený záložní postup**: síťová karta SDK se automaticky vrátí ke standardním paketům o velikosti 1500 bajtů a kamera se nadále připojuje normálně (následující řádky `[chunk-enable …]` jsou součástí běžné sekvence připojení). Zachycování stále funguje.
>
> Tuto kontrolu můžete přeskočit, ale **nejedná se pouze o potlačení záznamů v logu – vypíná to jumbo rámce.** Kamera odpovídá na pingy s příkazem „Don&#x27;t-Fragment“ pouze do velikosti 1500 bajtů, bez ohledu na to, jak kvalitní je vaše síť, takže samotný pingový test nikdy nedokáže odhalit podporu jumbo rámců; to dokáže pouze tato sonda. Pokud ji deaktivujete, kamera bude v jakékoli síti navždy používat standardní 1500-bajtové pakety:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Vyplatí se to pouze v síti, o které *víte*, , že nepodporuje jumbo pakety, kde to ušetří zhruba jednu sekundu času připojení na každou kameru. Jelikož se jedná o skutečný kompromis, nikoli jen kosmetickou změnu, SDK to nyní při použití jasně uvádí:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Nechte to být, pokud k tomu nemáte důvod.** Pokud zůstane povoleno, při každém připojení se znovu změří vaše aktuální síť: připojte se ke switchi podporujícímu jumbo pakety a při dalším připojení se jumbo pakety nastaví automaticky, bez nutnosti jakékoli konfigurace a restartu.
>
> Pokud *chcete* dosáhnout jumbo propustnosti, povolte jumbo end-to-end (MTU síťové karty 9000 + přepínač, který je propouští), nebo ji zafixujte pomocí `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`, pokud víte, že dané připojení tuto funkci podporuje — upřednostňujte však použití `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` pro jednotlivé příkazy před trvalým nastavením, protože pevně nastavená velikost přeskočí test a přestane se přizpůsobovat síti před ním. **Každé** zařízení na trase musí jumbo pakety propouštět — včetně jakéhokoli PoE rozbočovače nebo injektoru, což je obvyklý důvod, proč jinak jumbo-kompatibilní konfigurace nedokáže tyto pakety přenášet.

> **`SC_ERR_TIMEOUT -1011` během `capture()` / `grab*()` je jiný problém — ten je skutečnou chybou.**&gt; Výše uvedená poznámka se týká pouze chyby `-1011` zaznamenané**sondou connect-time**. Stejná chyba vyvolaná**zachycením** znamená, že kamera se připojila v pořádku, ale neodesílá žádné snímky:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Rozhodujícím znakem je kamera, jejíž *řídicí* kanál je v pořádku — vyhledání funguje, nastavení i zápisy `[chunk-enable …]` jsou úspěšné — zatímco *každý* snímek vyprší časový limit.
>
> **Obvyklou příčinou je, že je kamera nastavena na hardwarové spuštění.** V případě `trigger_mode="On"` a `trigger_source="Line2"` kamera nevysílá vůbec nic, dokud na synchronizačním kabelu M8 nedojde k elektrické změně stavu. Pokud nemáte kabel, který by tuto linku řídil, každý pokus o zachycení snímku čeká donekonečna. Kamera není rozbitá a síť je v pořádku — dělá přesně to, co má.
>
> Chyby `CameraSettings()` a předvolby `default` / `high_speed` / `high_quality` umožňují volný provoz, a snímání, u kterého dojde k vypršení časového limitu při aktivaci, zobrazí vysvětlení namísto pouhého `-1011`. `PRESETS["triggered"]` aktivuje Line2, jak je navrženo.
>
> Chcete-li vynutit volný provoz u jakékoli kamery:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Pokud i s `trigger_mode="Off"` stále dochází k vypršení časového limitu, kamera skutečně neposílá data — zašlete nám protokol a `ip link show`.

#### Barevné profily (živý náhled RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` volí barevný profil displeje pro **živý náhled** u kamer typu „RGB“ (kamery typu multispec toto nastavení ignorují):

| Profil | Význam |
| --- | --- |
| `raw` | Úplně obejít radiometrický řetězec. |
| `linear` | DSNU + flat + WB, bez CCM, žádná gama. |
| `natural` | Lineární + naměřený CCM + gama sRGB, pouze s levným dokončením (vyhlazení chrominance + desaturace světlých oblastí) — realistické výchozí nastavení. |
| `enhanced` | `natural` plus kompletní úprava s paritou hubu (odstranění pruhů, živost, lokální kontrast CLAHE). Bohatější vzhled za zhruba **dvojnásobné náklady na úpravy na snímek**, tedy nižší snímkovou frekvenci v reálném čase. |
| `custom_temp` | `natural`, ale vyvážení bílé (WB) pevně nastaveno na `custom_cct_k` Kelvin (DLS ignorováno; omezeno na 2000–10000 K na straně backendustraně). |

Profil je **pouze pro živý náhled**, slouží k nastavení rychlosti a vzhledu: uložené snímky vždy získají plně bohatý vzhled bez ohledu na vybraný profil, takže výběr `natural` za účelem získání času na snímku nesnižuje kvalitu toho, co se uloží na disk. Neznámý profil zvýší hodnotu `ValueError`; je-li dostupný backend chloros, je změna odeslána i na něj, takže ji zohlední již další náhledový snímek (uživatelé s přímým připojením SDK bez backendu stále dostávají změnu nastavení).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Mono (M3M) kamery a `Calibration`

Mono **M3M** kamera (`M3M-<lens>-F<wavelength>`) je jednopásmová: jedna rovina v odstínech šedé, bez Bayerovy mozaiky, bez matice spektrálního přeslechu 3×3. `Calibration` ji rozpozná a zpřístupní příznak `is_mono`. Odrazivost se stále uplatňuje jako radiometrická mapa pro jednotlivé pásma (demix je identitní matice), ale vícepásmové výpočty na jedné kameře vedou spíše k smysluplným výsledkům než k nesmyslům:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Chcete-li vytvořit vegetační index z monochromatického hardwaru, zkombinujte několik kamer M3M s různými vlnovými délkami do zarovnaného vícepásmového stohu (viz [Zarovnání pole](#array-alignment)) a vypočítejte index napříč tímto stohem namísto na jedné kameře.

Přímý režim DAQ:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **`apply_sensor_settings` přijímané klíče**— přesně `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E; zastaralé, nahrazeno `cap_id`), `filter_model` (DAQ-M)a `cap_id` (všechny typy DAQ; `None`/`""`/`"none"` = holý snímač, bez korekce krytu). Neznámé klíče jsou**tichým způsobem ignorovány** — např. `{"integration_time": 64}` neprovede nic (musí to být `integration_time_ms`). Vrací `{"applied": [...], "errors": {...}}` a nikdy nevyvolá výjimku.

`chloros_sdk` reexportuje pouze základní rozhraní použité výše. Úplné veřejné rozhraní `daq_sdk` (API, 22 názvů) přidává následující — importujte je přímo z `daq_sdk`:

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## Výjimky

Zachyťte základní třídu pro zpracování „všeho, co se v souboru Chloros pokazilo“:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` a `ChlorosConfigurationError` jsou exportovány na nejvyšší úrovni spolu s ostatními; lze je také importovat z `chloros_sdk.exceptions`, jak je znázorněno.

Hierarchie:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Kompletní příklady

### 1. Zpracování složky s vlastním ukazatelem průběhu

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Pole LATTICE v reálném čase → odrazivost + reference DAQ

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Snímací kampaň řízená projektem

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Proud snímků z více kamer → Pipeline NumPy

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Skript pro snímání bez grafického rozhraní přímo z hardwaru (bez backendu)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Test funkčnosti před připojením sestavy se 4 kamerami

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Ekvivalent receptu pro snímání (čistý Python)

Receptový jazyk DSL nástroje „CLI“ má ekvivalent v přímém Python:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Automatické spuštění backendu

Vstupní body smart-connect — `connect_camera`, `connect_array`, `connect_daq_sensor` a `discover_lattice_cameras` — jsou tenké klienty typu „HTTP“, které předpokládají, že backend naslouchá na `127.0.0.1:5000` (výchozí adresa URL rozhraní Smart-Connect). Pokud již běží grafické rozhraní (GUI) nebo CLI, backend již běží. V případě prostého skriptu tomu tak nemusí být — proto tyto funkce **automaticky spustí přiložený binární soubor backendu** (bez okna, stejným způsobem jako `ChlorosLocal`) před svým prvním voláním a poté čekají až do `backend_startup_timeout`, než se spustí.

Pravidla:

- **Spouští se vždy pouze lokální URL.** `backend_url` odkazující na `localhost` / `127.0.0.1` / `[::1]` je přípustný; jakýkoli jiný host je považován za stroj někoho jiného a nikdy se nespustí.
- **Backend zůstává spuštěný pro opětovné použití** (stejně jako v případě CLI) — při ukončení skriptu nedochází k implicitnímu vypnutí. Při opětovném spuštění skriptu se znovu použije aktivní backend.
- **Odhlášení pomocí `auto_start_backend=False`** při kterémkoli z těchto volání (např. když jste nasměrovali na vzdálený backend, nebo pokud životný cyklus backendu spravujete sami).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Pokud nelze najít nebo spustit přiložený binární soubor, následující volání HTTP vyvolá akční, **platformově specifickou** chybu `ChlorosConnectError` namísto prostého hlášení o odmítnutí připojení — na Windows vás nasměruje na desktopovou aplikaci nebo na příkaz `chloros-cli`; na Linux (bez grafického rozhraní) vás nasměruje na příkaz `chloros-cli` nebo na `.deb`.

---

## Prostředí a hlavičky

SDK označuje každé volání backendu HTTP pomocí `X-Chloros-Client: sdk`. Backend uplatňuje licenční pravidla SDK / CLI (vyžaduje se přihlášení **a** placený tarif Chloros+), nikoli bezplatnou verzi s grafickým rozhraním. Toto nastavení se provede automaticky při importu — nemusíte nic dělat.

`http://localhost` a `http://127.0.0.1` jsou detekovány jako lokální backend. Volání na jiné hostitele (např. vaši vlastní analytickou službu) zůstávají beze změny.

PřepišURLu backendu předáním `backend_url=` (nebo `api_url=` na `ChlorosLocal`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(`backend_url` bez loopbacku dosáhne pouze backendu source/dev — dodávané backendy se vážou pouze na loopback; viz Režim vzdáleného backendu pro vzor tunelu.)

---

## Verze a kompatibilita

- Verze „SDK“ je dostupná jako `chloros_sdk.__version__`.
- „SDK“ vázá chování na verzi dodaného backendu. Kombinace staršího „SDK“ s novějším backendem obvykle funguje (koncové body jsou kompatibilní s novějšími verzemi), ale kombinace novějšího SDK se starším backendem může na nových koncových bodech vyvolat chyby `404` — v takovém případě aktualizujte desktopovou aplikaci tak, aby odpovídala.
- Rozhraní Smart Connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) a koncový bod pro analýzu sítě vrací stabilní schémata JSON; nová pole jsou doplňková.

---

## Tipy pro řešení problémů

- **`ChlorosAuthenticationError: Login required`** → Spusťte na tomto počítači jednou příkaz `chloros-cli login EMAIL PASSWORD` nebo se přihlaste prostřednictvím desktopové aplikace Chloros.
- **`ChlorosConnectError: No Chloros backend is running …`** → Funkce Smart-Connect automaticky spouští lokální backend, takže se tato chyba zobrazí pouze v případě, že nelze najít nebo spustit přiložený binární soubor (např. na hostiteli, kde je k dispozici pouze pip a není nainstalován balíček pro desktop). Zpráva je závislá na platformě: na Windows otevřete desktopovou aplikaci nebo spusťte libovolný příkaz `chloros-cli`; na Linux spusťte příkaz `chloros-cli` (neexistuje žádné grafické rozhraní) nebo nainstalujte `.deb`. V případě vzdáleného backendu předávejte `backend_url=` (a `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** při importu → `lattice_sdk` se nepodařilo načíst (obvykle nejsou nainstalovány runtime DLL Arena SDK). Povrch bez kamery stále funguje.
- **Funkce Array connect vrací rozlišení nižší než nativní**→ Funkce smart-prep backendu automaticky zmenšuje velikost snímku, aby se vešel do přenosové linky. Použijte `analyze_array_network()` k zjištění příčiny, poté buď vylepšete připojení, přijměte zmenšení, nebo předávejte `force_tier="slip-emit-and-capture"` pro sekvenční snímání. Bezpečnostní opatření proti zmenšení**ne** pokrývá agregované přetížení (`oversubscribed: true`, pole fps 0): příliš mnoho kamer pro dané připojení nelze vyřešit binningem/ROI – snižte počet kamer, povolte jumbo rámce, nebo přejděte na rychlejší síťovou kartu (viz [Nadměrné předplatné](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` hlásí, že přijímací kruh síťové karty je příliš malý (~0,26 MB) / připojte brány s hlášením „FRAMES WILL DROP“** → Příjmový kruh hostitelské síťové karty je nastaven na výchozí hodnotu (často se po aktualizaci ovladače síťové karty resetuje na 32). U adaptéru Realtek USB 10GbE nastavte `ReceiveBufferLen=256` a `PendingReceives=64` (s vyššími oprávněními) a poté restartujte backend, aby znovu načítal kruh. Kompletní postup: [CLI Reference → Nastavení a ladění hostitelské síťové karty](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Host se zasekne při restartu/vypnutí, později dochází k chybám WMI `Invalid class` / síťová karta se neaktivuje** → Zastaralý ovladač USB 10GbE způsobuje `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Aktualizujte ovladač adaptéru na aktuální verzi (≥ 2026) a znovu použijte nastavení přijímacího kruhu. Viz [CLI Reference → Nastavení a ladění hostitelské síťové karty](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Reflexe odmítnuta** → Pro měření odrazivosti v absolutním měřítku musí být ke kameře (nebo soustavě) přiřazen aktivní DAQ. Proveďte přiřazení buď přes grafické rozhraní, nebo použijte `processing="radiance"` (W/m²/sr/nm), kterývyžaduje spárovaný senzor.
- **Zaznamenávání pomocí `smart=True` trvá déle, než se očekávalo** → Konvergence AE závisí na dynamice scény; pokud chcete rychlejší (méně stabilní) spouštění.

---

## Viz také

- [Referenční příručka k CLI](cli-reference.md) — každý podpříkaz „CLI“ odpovídá volání „SDK“.
- [Průvodce senzory DAQ](../daq/README.md) — pravidla pro zapojení, kalibraci a záznam specifická pro jednotlivé senzory.
- Online dokumentace: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>
