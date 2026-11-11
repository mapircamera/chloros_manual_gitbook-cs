# API: Python SDK

{% hint style="info" %}
**Hledáte kompletní API?** Tato stránka je praktickým návodem. Všechny veřejné třídy, metody, přesné podpisy a příklady, které lze zkopírovat a vložit, najdete v [SDK Reference](reference/sdk-reference.md), která je optimalizována pro AI asistenty.**Pracujete s AI asistentem?** Vložte tuto URL do chatu, aby měl k dispozici úplné a aktuální API Chloros 1.3.0:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Každá stránka této příručky je k dispozici jako surový markdown pod názvem v malých písmenech + `.md` a celá příručka je indexována na adrese `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` na PyPI) řídí vše, co desktopová aplikace dokáže, z jazyka Python: dávkové zpracování obrázků, živé ovládání kamery LATTICE a pole, relace sběru dat ze světelných senzorů a automatizaci uložených projektů. Jedná se o tenkou vrstvu nad stejným lokálním backendem, který používají grafické uživatelské rozhraní (GUI) i příkazový řádek (HTTP) (HTTP na `127.0.0.1:5000`), takže chování je ve všech třech prostředích identické.

## Instalace

SDK je součástí balíčku Chloros, který obsahuje také backend pro zpracování a hardwarová runtime prostředí a jehož verze se vždy shoduje s verzí balíčku.

**Krok 1 — Nainstalujte Chloros.** Windows: spusťte instalační program pro desktop (výchozí cesta `C:\Program Files\Chloros\`) ze stránky [Stáhnout](download.md). Linux: nainstalujte balíček `.deb` pomocí `sudo apt install ./chloros_1.3.0_<arch>.deb` ([Instalace pro Linux](linux/linux-installation.md)).**Krok 2 — Zkontrolujte SDK** (Python 3.7+). Obvykle není třeba nic instalovat:

* **Windows:** instalační program nainstaluje přiložený balíček SDK wheel do vašeho systémového Pythonu (pokud není nainstalován žádný Python, tento krok přeskočí).
* **Linux:** balíček `.deb` nainstaluje SDK pro systémový balíček `python3` do balíčku `/usr/lib/python3/dist-packages`. Požadované balíčky, `python3-numpy` a `python3-requests`, doporučuje balíček: `apt install` je obsahuje, `dpkg -i` nikoli.
* **Prostředí virtualenv** nevidí systémové balíčky; nainstalujte do něj přiložený soubor wheel: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` v systému Linux nebo `chloros_sdk-*.whl` ve složce `sdk` v instalační složce systému Windows (ve výchozím nastavení `C:\Program Files\Chloros\sdk\`).

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

Na počítači bez balíčku Chloros, jehož SDK řídí backend na jiném počítači (viz [Co získáte při instalaci pouze pomocí pip](#what-a-pip-only-install-gets)), nainstalujte jej z PyPI: `pip install chloros-sdk`. PyPI se aktualizuje při vydání nových verzí, takže odpovídá nejnovější stabilní verzi.

**Krok 3 — Přihlaste se jednou na každém počítači:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Přihlašovací údaje se ukládají do mezipaměti v `~/.chloros/` (na obou platformách). Ve Windowsu se můžete přihlasit stejným způsobem prostřednictvím karty „Uživatel“ <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> v desktopové aplikaci. SDK vyžaduje placený tarif Chloros+ — viz [Požadavky na licenci](#license-requirement) níže.

| Požadavek | Podrobnosti |
| --- | --- |
| **Nainstalovaný Chloros** | Windows: instalační program pro stolní počítače; Linux: balíček `.deb` (obsahuje binární soubor backendu a SDK) |
| **Jazyk Python** | 3.7 nebo vyšší (vyvinuto/testováno na verzi 3.10) |
| **Operační systém** | Windows 10/11 (64bitová verze), Ubuntu 22.04 LTS nebo novější, případně NVIDIA Jetson (JetPack 6) |
| **Licence** | Aktivní přihlášení k Chloros+, libovolný placený tarif (Copper nebo vyšší) |

## Úspěch za 60 sekund

Jedním voláním vytvoříte projekt, naimportujete složku, nakonfigurujete zpracování a spustíte pipeline — přičemž se automaticky spustí backend, pokud ještě není spuštěn:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

(V systému Linux použijte Linuxové cesty: `/home/user/drone_images/flight001`. SDK funguje na obou platformách stejně.)

Zpracováváte složku se záznamy z LATTICE? Použijte wrapper přizpůsobený pro LATTICE – použije správná výchozí nastavení (žádná detekce cílového panelu, standardní debayer):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — plná kontrola nad pipeline

Pro cokoli složitějšího než příkaz na jeden řádek použijte `ChlorosLocal`. Využívá backend, který již běží (desktopové aplikace), nebo při prvním použití spustí nainstalovaný backend (`auto_start_backend=True`; ve Windowsu ze složky `backend` v instalační složce Chloros, ve výchozím nastavení `C:\Program Files\Chloros\backend\`), vytváří a konfiguruje projekty, sleduje průběh a po dokončení vrátí souhrn.

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
Ponechte výchozí `http://127.0.0.1:5000` namísto nahrazení `localhost` — v systému Windows se `localhost` nejprve převede na `::1` a u backendu podporujícího pouze IPv4 to trvá přibližně 2 sekundy na jeden požadavek.
{% endhint %}

Použijte jej jako správce kontextu. Opuštění bloku ukončí používání backendu touto instancí, nikoli samotný backend: ten zůstane spuštěný pro další skript a zastaví se sám po 30 minutách nečinnosti. `shutdown_backend()` zastaví jednu instanci, kterou tato instance spustila.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` přijímá tato klíčová slova: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` a `custom_settings`. Hlavní hodnoty:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Ovládací prvky specifické pro LATTICE (`input_level`, `radiometric_output`, řada `array_alignment*`) jsou zdokumentovány spolu s úplnými tabulkami hodnot v [Referenci SDK](reference/sdk-reference.md#supported-values).

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

### Čtení souhrnu po spuštění — a zachycení prázdných běhů

Po dokončení připojí `process()` souhrn zpracování backendu jako `result["summary"]`. Každá položka v `summary["hints"]` je úplná věta vysvětlující vše důležité — například proč běh nevrátil žádný výstup — a každý tip je také znovu vyslán jako Pythonový `UserWarning`, takže prázdné běhy se diagnostikují samy, i když slovník nikdy neprohlížíte:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` nevzbudí výjimku, když běh nevytvoří žádné obrázky.** Toto je jediné místo, kde se SDK a CLI záměrně liší: `chloros-cli process` považuje stav „byly vyžádány výstupy, žádné nebyly zapsány“ za selhání a ukončí se s nenulovým stavem, zatímco SDK se vrátí normálně a o tomto stavu informuje prostřednictvím `summary` / nápovědy. Pokud by se váš pipeline měl zastavit při prázdném běhu, zkontrolujte to sami — prozkoumejte `summary` (nebo spočítejte soubory ve složce projektu), místo abyste se spoléhali na výjimku.
{% endhint %}

## Smart Connect — živý hardware

Tři pomocné skripty otevírají trvalé relace v hardwarovém fondu backendu — ve stejném fondu, který používají CLI, jiné skripty a záložky kamer v grafickém rozhraní, takže skripty SDK s nimi koexistují, aniž by se přetahovaly o sériové porty nebo šířku pásma sítě. Všechny tři automaticky spustí lokální backend, pokud žádný neběží. Panel DAQ v grafickém uživatelském rozhraní není součástí tohoto fondu: před otevřením světelného senzoru ze skriptu jej nejprve zavřete v grafickém uživatelském rozhraní.

### Jedna kamera LATTICE — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**Relativní cesty směřují do adresáře, kde se skript spouští (změněno ve verzi 1.3.0).** S lokálním backendem je relativní `output_dir` (včetně výchozího `"output"`) — a ostatní cesty, které backend otevírá (`burst_dir`, `profile_path`, `save_path`, `mask_path`) — se přepočítávají na pracovní adresář vašeho skriptu, takže `cam.capture("output/")` zapisuje do `<current folder>/output/`. Ve verzi 1.2.0 je nainstalovaný backend vyhodnocoval ve své vlastní instalační složce a selhával s chybou „Přístup odepřen“. U vzdáleného backendu se cesty odesílají tak, jak jsou zadané.
{% endhint %}

### Synchronizované pole — `connect_array`

`connect_array` je doporučený vstupní bod pro sestavy s více kamerami. Spouští stejný proces inteligentní přípravy jako grafické uživatelské rozhraní: analýzu sítě, automatický výběr synchronizační úrovně, časovou synchronizaci PTP, výběr formátu pixelů pro každou kameru, nastavení počátečních hodnot AE a aktivaci spouště GPIO. **První sériové zařízení je master** (vysílá hardwarový spouštěcí impuls); ostatní jsou slave.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Přidejte `smart=True` k jakémukoli snímání pole, aby se před spuštěním počkalo na ustálení automatické expozice u všech kamer.

Novinkou ve verzi 1.3.0 je, že `ArraySession` vám také přímo poskytuje synchronizované snímky, aniž by zapisoval soubory:

* `arr.grab()` vrací jeden snímek na kameru, všechny z jednoho spouštěcího signálu, jako pole numpy (`{serial: array}`); `.group_id` je sdílený spouštěč a `.errors` označuje jakoukoli vynechanou kameru.
* `arr.frame_stream(fps=5.0)` poskytuje jednu takovou sadu za každý tik pro živé zpracování ve vašem vlastním zpracování.
* `arr.continuous("output/")` spouští vlastní nepřetržité snímání backendu (režim „Continuous“ v grafickém rozhraní) a vrací `ContinuousHandle` s `status()`, `wait()` a `stop()` — použijte jej namísto snímací smyčky, pokud potřebujete ukládat každý spouštěcí signál s vysokou frekvencí.

Informace o režimech snímání (Jednorázový / Kontinuální / Intervalový / Nejrychlejší), záznamnících, burst-to-video a zarovnání pole najdete v [Referenci SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Světelný senzor DAQ — `connect_daq_sensor`

Bez argumentů `connect_daq_sensor()` automaticky detekuje přenosový protokol (v pořadí: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Každý rámec obsahuje 135bodovou hodnotu `spectrum` (W/m²/nm po kalibraci), příznak `is_saturated` a CIE `x`, `y`, `z`. Chcete-li přiřadit konkrétní senzor nebo přenosový protokol, předávejte jeden explicitní údaj:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Upozorňujeme, že profily korekce jasu (`cap_id`) **nejsou** ovládacím prvkem v rámci SDK — vyberte je místo toho pomocí `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Uložené projekty — `open_project`

Uložený projekt Chloros si zachovává připojený hardware (`cameras.json` + `sensors.json` spolu s `project.json`), a `chloros_sdk.open_project(path)` dokáže okamžitě znovu připojit své kamery, pole a světelné senzory a řídit snímání podle názvu zařízení. Světelné senzory se znovu připojují prostřednictvím spuštěného backendu Chloros, a to se svými uloženými nastaveními a kapacitou. Viz [Automatizace projektu](reference/sdk-reference.md#project-automation--chlorosproject) v referenční příručce.

## Co přináší instalace pouze s pipem

Před použitím hardwarových povrchů zkontrolujte příznaky dostupnosti na úrovni modulů:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Na hostitelském počítači, kde je **pouze** nainstalován modul `pip install chloros-sdk` a není nainstalován balíček Chloros pro desktop:

* `ChlorosLocal`, `process_folder` a `process_lattice_capture` **nefungují** — vyžadují binární soubor backendu, který je součástí instalačního balíčku pro desktop.
* Pomocné programy smart-connect (`connect_camera`, `connect_array`, `connect_daq_sensor`) jsou čistě HTTP klienti, takže fungují s backendem na jiném počítači — dodávané backendy se však vážou pouze na smyčku, takže musíte port přesměrovat sami (např. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) a předat `backend_url="http://127.0.0.1:5000"` společně s `auto_start_backend=False`. Viz [Režim vzdáleného backendu](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Třídy LATTICE pro přímé připojení k hardwaru (`LatticeCamera`, `CameraPool`, …) lze importovat, ale vyžadují runtime Arena SDK z balíčku pro stolní počítače — bez něj je `CAMERA_AVAILABLE` rovno `False`.
* Senzory DAQ jsou řízeny pouze prostřednictvím (tunelovaného) backendu — `connect_daq_sensor()` nebo projektového `proj.sensors`; backend otevírá senzor na svém vlastním počítači.

## AI asistenti — `chloros-mcp`

SDK obsahuje `chloros-mcp`, server MCP, který umožňuje aplikacím Claude Desktop, Claude Code, Cursor a dalším klientům MCP ovládat Chloros: zpracovávat snímky a ovládat kamery LATTICE, pole a světelné senzory DAQ. Jeho nástroje pokrývají funkce tohoto SDK. Instaluje se společně s SDK, takže pro Claude Code stačí k nastavení jediný řádek:

```bash
claude mcp add chloros -- chloros-mcp
```

Ostatní klienti: [Používání Chloros s AI asistenty](ai-assistants.md). Možnosti a všechny nástroje: [MCP Server (AI Assistants)](reference/sdk-reference.md#mcp-server-ai-assistants).

## Licenční požadavky

Přístup k SDK vyžaduje aktivní přihlášení do Chloros+ v jakémkoli placeném tarifu — **Copper nebo vyšším**(Copper / Bronze / Silver / Gold); bezplatný tarif Iron nemá přístup k SDK ani CLI. Kontrola se provádí**na straně serveru**: každý požadavek SDK musí obsahovat jak aktivní relaci, tak placený tarif, jinak backend vrátí chybu `403` / `PLAN_UPGRADE_REQUIRED` (vypovězeno jako `ChlorosLicenseError` pomocnými funkcemi `ChlorosLocal` a jako `ChlorosConnectError` pomocnými funkcemi `connect_*`). Odhlášený volající obdrží místo toho chyby `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — opětovné spuštění `chloros-cli login` vyřeší první případ, ale ne druhý.

Offline použití funguje v rámci ochranné lhůty tarifu: úroveň přístupu se načítá z mezipaměti pro ověření serverem (5 minut) nebo z mezipaměti podepsaných licencí vázaných na konkrétní zařízení (30 dní u měsíčních tarifů; do vypršení předplatného u ročních tarifů). Po uplynutí lhůty se tarif změní na bezplatný a přístup k SDK se zastaví, dokud se zařízení jednou nepřipojí k serveru. `chloros-cli status` zůstává dostupný i v bezplatném tarifu, takže důvod je vždy viditelný. Viz [Přihlášení do Chloros+](chloros+-login.md).

## Výjimky

Zachyťte základní třídu pro zpracování „jakékoli chyby v Chloros“:

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

Všechny výjimky v potrubí (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) pocházejí z `ChlorosError`. Jedna past: `ChlorosConnectError` — vyvolává pouze `connect_camera` / `connect_array` / `connect_daq_sensor` — pochází z prostého `Exception`, **nikoli** z `ChlorosError`, takže `except ChlorosError` ji nezachytí. Úplná hierarchie je uvedena v [Referenci SDK](reference/sdk-reference.md#exceptions).

## Viz také

* [Referenci SDK](reference/sdk-reference.md) — kompletní rozhraní API, optimalizované pro AI asistenty.
* [Referenční příručka CLI](reference/cli-reference.md) — každý podpříkaz CLI odpovídá volání SDK.
* [Používání Chlorosu s AI asistenty](ai-assistants.md) — návody na nastavení a příkazy pro `chloros-mcp`.
* [Stažení](download.md) — instalační programy pro Windows a Linux.
