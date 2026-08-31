# API : Python SDK

{% hint style="info" %}
**Hledáte kompletní API?** Tato stránka je praktickým návodem. Všechny veřejné třídy, metody, přesné signatury a příklady, které lze zkopírovat a vložit, najdete v [referenční příručce k SDK](reference/sdk-reference.md), která je optimalizována pro AI asistenty.**Pracujete s AI asistentem?** Vložte tento URL do chatu, aby měl k dispozici úplnou a aktuální verzi Chloros 1.2.0 API:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Každá stránka této příručky je k dispozici jako surový markdown pod názvem ve malých písmenech + `.md` a celá příručka je indexována na `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` na PyPI) řídí vše, co desktopová aplikace dokáže od Python: dávkové zpracování obrázků, živé ovládání kamery LATTICE a pole, relace DAQ se světelnými senzory a automatizaci uložených projektů. Jedná se o tenkou vrstvu nad stejným lokálním backendem, který používají grafické uživatelské rozhraní i CLI (HTTP na `127.0.0.1:5000`), takže chování je na všech třech rozhraních identické.

## Instalace

Instalace probíhá ve dvou krocích: nejprve nainstalujte balíček Chloros pro stolní počítače (poskytuje backend pro zpracování a hardwarové runtime), poté balíček Python.

**Krok 1 — Nainstalujte Chloros.** Windows: spusťte instalační program pro stolní počítače (výchozí cesta `C:\Program Files\MAPIR\Chloros\`) ze stránky [Stáhnout](download.md). Linux: nainstalujte balíček `.deb` ([Instalace Linux](linux/linux-installation.md)).**Krok 2 — Nainstalujte SDK** (Python 3.7+):

```bash
pip install chloros-sdk
```

Možná ani nebudete potřebovat pip: každý instalační balíček obsahuje odpovídající balíček SDK. Instalační program Windows jej automaticky nainstaluje do vašeho systému Python; instalační program Linux `.deb` jej umístí do adresáře `/usr/lib/chloros/sdk/` a vypíše přesný příkaz `pip install --user`. PyPI se aktualizuje při vydávání nových verzí, takže `pip install chloros-sdk` odpovídá nejnovější stabilní verzi.

**Krok 3 — Přihlaste se jednou na každém počítači:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Přihlašovací údaje se ukládají do mezipaměti v souboru `~/.chloros/` (na obou platformách). Na Windows se můžete přihlasit stejným způsobem prostřednictvím karty „User“ (Uživatel) v aplikaci pro stolní počítače <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">. SDK vyžaduje placený tarif Chloros+ — viz [Požadavky na licenci](#license-requirement) níže.

| Požadavek | Podrobnosti |
| --- | --- |
| **Nainstalovaný Chloros** | Windows: instalační program pro stolní počítače; Linux: balíček `.deb` (poskytuje binární soubor backendu) |
| **Python** | 3.7 nebo vyšší (vyvinuto/testováno na verzi 3.10) |
| **Operační systém** | Windows 10/11 64bit, Ubuntu 22.04 LTS nebo novější, nebo NVIDIA Jetson (JetPack 6) |
| **Licence** | Aktivní přihlášení k Chloros+, libovolná placená úroveň (Copper nebo vyšší) |

## Úspěch za 60 sekund

Jedním voláním vytvoříte projekt, naimportujete složku, nakonfigurujete zpracování a spustíte pipeline — přičemž se automaticky spustí backend, pokud ještě není spuštěn:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(Na Linux použijte cesty Linux: `/home/user/drone_images/flight001`. SDK funguje na obou platformách stejně.)

Zpracováváte složku snímků z LATTICE? Použijte obalový program přizpůsobený pro LATTICE — ten použije správná výchozí nastavení (žádná detekce cílového panelu, standardní debayer):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — plná kontrola nad zpracovatelským řetězcem

Pro cokoli složitějšího než jednořádkový příkaz použijte `ChlorosLocal`. Při prvním použití spustí backend (`auto_start_backend=True`), vytvoří a nakonfiguruje projekty, sleduje průběh a po dokončení vrátí souhrn.

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

{% hint style="info" %}
Používejte výchozí `http://127.0.0.1:5000` namísto nahrazení `localhost` — u Windows `localhost` se nejprve převede na `::1` a u backendu podporujícího pouze IPv4 trvá zpracování jednoho požadavku přibližně 2 sekundy.
{% endhint %}

Použijte jej jako správce kontextu pro zaručené uvolnění paměti:

```python
import chloros_sdk

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

`configure()` přijímá tato klíčová slova: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` a `custom_settings`. Hlavní hodnoty:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Ovládací prvky specifické pro LATTICE (`input_level`, `radiometric_output`, řada `array_alignment*`) jsou zdokumentovány spolu s úplnými tabulkami hodnot v [Referenční příručce k SDK](reference/sdk-reference.md#supported-values).

### Sledování průběhu

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Čtení souhrnu po dokončení běhu — a zachycení prázdných běhů

Po dokončení připojí `process()` souhrn zpracování backendu jako `result["summary"]`. Každá položka v `summary["hints"]` je úplná věta vysvětlující cokoli významného — například proč běh vyprodukoval nulový výstup — a každý tip je také znovu vyslán jako Python `UserWarning`, takže prázdné běhy se diagnostikují samy, i když slovník nikdy neprohlížíte:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` se nevypíše, pokud běh nevytvoří žádné obrázky.** Toto je jediné místo, kde se SDK a CLI záměrně liší: `chloros-cli process` považuje stav „byly požadovány výstupy, ale žádné nebyly zapsány“ za selhání a ukončí se nenulovým stavem, zatímco SDK se vrátí normálně a o tomto stavu informuje prostřednictvím `summary` / hints. Pokud by se váš pipeline měl zastavit při prázdném běhu, zkontrolujte to sami — zkontrolujte skript `summary` (nebo spočítejte soubory ve složce projektu), místo abyste se spoléhali na výjimku.
{% endhint %}

## Smart Connect — živý hardware

Tři pomocné skripty otevírají trvalé relace v hardwarovém fondu backendu – ve stejném fondu, který používá grafické uživatelské rozhraní, takže skripty SDK koexistují s desktopovou aplikací, aniž by si konkurovaly o sériové porty nebo šířku pásma sítě. Všechny tři automaticky spustí lokální backend, pokud žádný neběží.

### Jedna kamera LATTICE — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### Synchronizované pole — `connect_array`

`connect_array` je doporučený výchozí bod pro sestavy s více kamerami. Spouští stejný proces inteligentní přípravy jako grafické uživatelské rozhraní: analýzu sítě, automatický výběr synchronizační úrovně, časovou synchronizaci PTP, výběr formátu pixelů pro každou kameru, nastavení automatické expozice a aktivaci spouště GPIO. **První sériové zařízení je master** (vysílá hardwarový spouštěcí impuls); ostatní jsou slave.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Přidejte `smart=True` k jakémukoli snímání pole, aby se před spuštěním počkalo na ustálení automatické expozice u všech kamer. Informace o režimech snímání (Jednotlivé / Sériové / Intervalové / Nejrychlejší), záznamnících, sériovém záznamu do videa a zarovnání pole najdete v [Referenční příručce k SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Světelný senzor DAQ — `connect_daq_sensor`

Bez argumentů funkce `connect_daq_sensor()` automaticky detekuje přenosový protokol (v pořadí: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Každý rámec obsahuje 135bodovou hodnotu `spectrum` (W/m²/nm po kalibraci), příznak `is_saturated` a CIE `x`, `y`, `z`. Chcete-li přiřadit konkrétní senzor nebo přenosový protokol – což je spolehlivá volba na hostitelských počítačích s více síťovými rozhraními, kde automatické vyhledávání sítě Ethernet může při prvním pokusu přehlédnout funkční DAQ-E – předávejte jeden explicitní údaj:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Upozorňujeme, že profily korekce velkých písmen (`cap_id`) **nejsou** ovládacím prvkem typu SDK — vyberte je místo toho pomocí `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Uložené projekty — `open_project`

Uložený projekt Chloros si zachovává připojený hardware (`cameras.json` + `sensors.json` spolu s `project.json`), a `chloros_sdk.open_project(path)` dokáže vše znovu připojit najednou a řídit snímání podle názvu zařízení. Viz [Automatizace projektů](reference/sdk-reference.md#project-automation--chlorosproject) v referenční příručce.

## Co získáte při instalaci pouze přes pip

Před použitím hardwarových povrchů zkontrolujte příznaky dostupnosti na úrovni modulů:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Na hostitelském počítači, kde je **pouze** balíček `pip install chloros-sdk` a není nainstalován balíček Chloros pro pracovní plochu:

* `ChlorosLocal`, `process_folder` a `process_lattice_capture` **nefungují** — vyžadují binární soubor backendu, který je součástí instalačního balíčku pro desktop.
* Pomocné programy Smart-Connect (`connect_camera`, `connect_array`, `connect_daq_sensor`) jsou čistě klienti typu HTTP, takže fungují s backendem na jiném počítači – dodávané backendy se však vážou pouze na smyčku, takže musíte port přesměrovat sami (např. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) a předat `backend_url="http://127.0.0.1:5000"` spolu s `auto_start_backend=False`. Viz [Režim vzdáleného backendu](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Třídy LATTICE pro přímý přístup k hardwaru (`LatticeCamera`, `CameraPool`, …) lze importovat, ale vyžadují runtime Arena SDK z balíčku pro stolní počítače — bez něj je `CAMERA_AVAILABLE` rovno `False`.
* `daq_sdk` (třídy pro přímé sběr dat) je součástí instalace pro stolní počítače, nikoli balíčku PyPI, takže `DAQ_AVAILABLE` je na hostiteli používajícím pouze pip ekvivalentem `False` — místo toho ovládejte senzory DAQ prostřednictvím `connect_daq_sensor()` proti (tunelovanému) backendu.

## Licenční požadavky

Přístup k SDK vyžaduje aktivní přihlášení k Chloros+ v jakémkoli placeném tarifu — **Copper nebo vyšším**(Copper / Bronze / Silver / Gold); bezplatný tarif Iron nemá přístup k SDK/CLI. Kontrola se provádí**na straně serveru**: každý požadavek SDK musí obsahovat jak aktivní relaci, tak placený tarif, jinak backend vrátí `403` / `PLAN_UPGRADE_REQUIRED` (vyvoláno jako `ChlorosLicenseError` funkcí `ChlorosLocal` a jako `ChlorosConnectError` pomocnými funkcemi `connect_*`). Odhlášený volající obdrží místo toho chyby `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — opakované spuštění `chloros-cli login` vyřeší první případ, ale ne ten druhý.

Offline použití funguje v rámci ochranné lhůty tarifu: úroveň přístupu se načítá z mezipaměti pro ověření serverem (5 minut) nebo z mezipaměti podepsané licence vázané na konkrétní zařízení (30 dní u měsíčních tarifů; do vypršení předplatného u ročních tarifů). Po uplynutí lhůty se tarif přepne na bezplatnou verzi a přístup k SDK se zastaví, dokud se zařízení alespoň jednou nepřipojí k serveru. `chloros-cli status` zůstává dostupný v rámci bezplatné úrovně, takže důvod je vždy viditelný. Viz [Chloros+ Přihlášení](chloros+-login.md).

## Výjimky

Zachyťte základní třídu pro zpracování „všech případů, kdy se něco Chloros pokazilo“:

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

Všechny výjimky potrubí (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) pocházejí z výjimky `ChlorosError`. Jedna past: `ChlorosConnectError` — vyvolává pouze `connect_camera` / `connect_array` / `connect_daq_sensor` — pochází z prostého `Exception`, **nikoli** z `ChlorosError`, takže `except ChlorosError` ji nezachytí. Úplná hierarchie je uvedena v [Referenci SDK](reference/sdk-reference.md#exceptions).

## Viz také

* [Referenční příručka k SDK](reference/sdk-reference.md) — kompletní rozhraní API, optimalizované pro AI asistenty.
* [CLI Reference](reference/cli-reference.md) — každý podpříkaz CLI odpovídá volání SDK.
* [Stáhnout](download.md) — instalační programy pro Windows a Linux.
