# Instalace Linux

Chloros je distribuován pro Linux jako balíčky `.deb`, které instalují CLI a backendový server. Python SDK je samostatný balíček pip (také součástí balíčku `.deb` jako wheel s odpovídající verzí).

Názvy souborů balíčků obsahují verzi a architekturu: `chloros_1.2.0_amd64.deb` pro x86_64 a `chloros_1.2.0_arm64_jp6.deb` pro sestavení JetPack 6 pro Jetson. V níže uvedených příkazech nahraďte název souboru tím, který jste si skutečně stáhli.

***

## Linux amd64 (x86_64)

### Systémové požadavky

| Požadavek | Minimální | Doporučené |
| --- | --- | --- |
| **Distribuce** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Procesor** | x86_64 (Intel/AMD) | Intel Core i7 nebo lepší |
| **Paměť (RAM)** | 8 GB | 16 GB nebo více |
| **Grafická karta** | Žádná (zpracování na procesoru) | Grafická karta NVIDIA s 4 GB+ VRAM (12 GB+ odemkne `GPU_PARALLEL`, 7 GB+ udržuje funkci Texture Aware mimo cestu pro jeden obrázek) |
| **Úložiště** | 2 GB volného místa | SSD s 10 GB+ volného místa |
| **Python** | Python 3.7+ (pro SDK) | Python 3.10+ |

> **Ubuntu 20.04 a Debian 11 nejsou podporovány.** Seznam závislostí `.deb` je
> odvozen od toho, na co se backend Chloros skutečně odkazuje, a to zahrnuje
> `libc6 (>= 2.34)`. Verze Focal i Bullseye obsahují glibc 2.31, takže `apt` instalaci
> rovnou odmítne, místo aby ji nechal selhat později při spuštění.

### Instalace

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
`dpkg -i` nevyřeší závislosti. Pokud hlásí chybějící balíčky, `sudo apt-get install -f` (nebo `sudo apt --fix-broken install`) dokončí instalaci – jedná se o normální postup, nikoli o chybu.
{% endhint %}

Ověřte instalaci:



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```***

## Linux arm64 (NVIDIA Jetson)

### Systémové požadavky

| Požadavek | Minimální | Doporučené |
| --- | --- | --- |
| **Platforma** | NVIDIA Jetson s JetPackem 6 | Jetson Orin NX 16 GB nebo AGX Orin |
| **JetPack** | JetPack 6.x | Nejnovější JetPack 6 |
| **Paměť (RAM)** | 8 GB (sdílená mezi GPU a CPU) | 16 GB+ sdílené (12 GB+ je minimální požadavek pro paralelní GPU pracovníky) |
| **Úložiště** | 2 GB volného místa | NVMe SSD s 10 GB+ volného místa |
| **Python** | Python 3.7+ (pro SDK) | Python 3.10+ |

### Instalace

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Stejná struktura jako u amd64 `.deb`, s verzí CUDA vyladěnou pro Jetson Orin / Orin NX / Orin Nano. Informace o chování Jetsonu z hlediska paměti, teploty a nasazení v terénu najdete v [Příručce NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Instalace Python a SDK (všechny Linux)

SDK je čistě Python klient pro backend, takže stejný balíček funguje na amd64 i arm64. Dva zdroje:**Z PyPI** — zveřejněná stabilní verze:

```bash
pip install chloros-sdk
```

**Z přiloženého souboru wheel** — zaručeně odpovídá právě nainstalovanému backendu (použijte jej, pokud je vaše verze novější než na PyPI):

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**Distribuce podle PEP 668** (Ubuntu 23.10+, Debian 12+) nepovolují instalace pomocí pip v celém systému. Použijte `pip install --user …`, virtuální prostředí nebo `sudo pip install --break-system-packages …`. Instalační program balíčku nikdy automaticky nenainstaluje SDK do vašeho systémového Python — toto rozhodnutí je na vás.
{% endhint %}

Volitelná doplňková zařízení:

| Doplněk | Příkaz | Přidá |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` pro živé streamování průběhu |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` pro přenos BLE (DAQ-M) |

Ověřte SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
`.deb` nainstaluje Chloros, CLI a backend. Python SDK komunikuje s tímto backendem přes lokální HTTP API (`http://127.0.0.1:5000`) a v případě potřeby jej automaticky spustí. Vždy používejte doslovnou adresu IPv4 namísto `localhost` — `localhost` se může přeložit na `::1` a každý dotaz trvá přibližně dvě sekundy.
{% endhint %}

***

## První nastavení

### 1. Přihlášení

Přístup k adresám CLI a SDK vyžaduje placený tarif Chloros+ (**Copper** nebo vyšší), což je vynucováno na straně serveru: odhlášený volající obdrží `401 AUTH_REQUIRED` a volající s bezplatným tarifem (Iron) obdrží `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Přihlašovací údaje jsou uloženy v mezipaměti v `~/.chloros/user_session.json`.

{% hint style="warning" %}
**Po každé instalaci nebo aktualizaci se musíte znovu přihlásit.** Skript balíčku `prerm` záměrně vymaže `~/.chloros/user_session.json` a uloženou licenci pro každého uživatele na daném počítači, takže nová verze vždy znovu ověří licenci, místo aby se spoléhala na zastaralou mezipaměť.
{% endhint %}

### 2. Zkontrolujte stav licence

```bash
chloros-cli status
```

`chloros-cli status` funguje v jakékoli úrovni (včetně bezplatné), takže vždy můžete zjistit, proč je nebo není přístup k dispozici.

### 3. Spusťte diagnostiku systému

```bash
chloros-cli selftest
```

Provede se sedm kontrol v daném pořadí a pokud některá z nich selže, příkaz skončí s nenulovým kódem:

| # | Kontrola | Co ověřuje |
| --- | --- | --- |
| 1 | **Verze** | CLI nahlásí svou verzi (`v1.2.0`). |
| 2 | **Port k dispozici** | Port 5000 je volný, *nebo* na něj již odpověděl funkční backend Chloros (což se počítá jako úspěšný test). |
| 3 | **Spuštění backendu** | Spustí se binární soubor backendu. |
| 4 | **Test API (`/api/test`)** | Backend odpoví `status: ok`. |
| 5 | **Informace o systému** | Vypíše `GPU: <name>, CUDA: <bool>, PyTorch: <version>` z `/api/system-info`. |
| 6 | **Modely odšumovače** | Najde modely `*.pth.enc` (na Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + odšumovač**| Funkce „Texture Aware“ je skutečně použitelná — vyžaduje CUDA**a** alespoň jeden modelový soubor. |

Spuštění končí modelem `N/7 checks passed` a vypisuje všechny chyby podle názvu.

### 4. Zpracujte svůj první datový soubor

```bash
chloros-cli process ~/datasets/flight001
```

***

## Soubory a adresáře

### Pro každého uživatele

Chloros uchovává své přihlašovací údaje a konfiguraci CLI v jediném adresáři pro všechny platformy, **`~/.chloros/`** (na Windows, `%USERPROFILE%\.chloros\`). Dvě mezipaměti specifické pro Linux se místo toho řídí konvencemi XDG — tyto respektují `XDG_CONFIG_HOME` / `XDG_CACHE_HOME`, jsou-li nastaveny.

| Cesta | Účel |
| --- | --- |
| `~/.chloros/user_session.json` | Mezipaměť přihlašovací relace zapsaná souborem `chloros-cli login` (vymazána při každé instalaci/aktualizaci balíčku) |
| `~/.chloros/working_directory.txt` | Přepsání výchozího projektového adresáře (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Nastavení jazyka CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Nastavení jazyka sdílené s grafickým rozhraním Windows — hodnota `language` zde má přednost před `cli_language.json` |
| `~/.chloros/update_cache.json` | Hodinová mezipaměť pro kontrolu aktualizací při spuštění Linux/Jetson |
| `~/.chloros/backend.log` | Protokol backendu při jeho spuštění pomocí CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Uložené kalibrační balíčky LATTICE pro jednotlivé kamery, indexované podle sériového čísla a hash balíčku |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Volitelné uživatelské přepsání profilů korekce DAQ |
| `~/.config/chloros/system_config.json` | Uložený hardwarový profil z Dynamic Compute Adaptation — jeho smazáním vynutíte nové rozpoznání hardwaru |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Protokoly backendového serveru, jeden soubor na jedno spuštění |
| `~/Chloros Projects/` | Výchozí složka projektu, pokud není nastaveno žádné přepsání |

### Systémové

| Cesta | Účel |
| --- | --- |
| `/usr/bin/chloros-cli` | Obalový skript — nastaví `LD_LIBRARY_PATH` pro přibalené nativní knihovny a poté spustí skutečný binární soubor |
| `/usr/bin/chloros-backend` | Obalový skript — totéž, plus `CHLOROS_PRODUCTION=1`, aby se autentizační brána backendu nemohla nikdy tiše deaktivovat |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Zkompilované binární soubory |
| `/usr/lib/chloros/arena_runtime/` | Runtime Arena SDK vyžadovaný kamerami LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Šifrované modely odšumovače používané debayerem Texture Aware |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK balíček odpovídající přesně této sestavě |
| `/usr/lib/chloros/exiftool` | Přiložený exiftool (odkazující na `/usr/local/bin/exiftool` pouze v případě, že neexistuje systémový exiftool) |
| `/etc/chloros/update.conf` | Konfigurace kanálu aktualizací načítaná `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Nastaví `net.ipv4.ip_unprivileged_port_start = 319` tak, aby backend mohl navázat PTP porty bez oprávnění root |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Nasměruje dynamický zavaděč na `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Udělí přihlášenému uživateli přístup k sériovému mostu DAQ-U USB (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend.service` | Zapnutí trvale běžící backendové služby (nainstalováno, **není povoleno**) |
| `/usr/share/applications/chloros-cli.desktop` | Položka v nabídce aplikace „Chloros CLI“, která otevírá terminál |

## Umístění spustitelného souboru backendu

CLI a SDK automaticky detekují backend:

| Komponenta | Cesta |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` |

Cestu k backendu lze přepsat pomocí příznaku `--backend-exe` CLI nebo parametru konstruktoru `backend_exe` SDK, a port pomocí `--port` (výchozí hodnota `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` směřuje k **`lattice`**,**`project`**a**`daq pool-*`** na vzdálený backend. Základní příkazy (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) jej záměrně ignorují a vždy cílí na `http://127.0.0.1:<port>`.
{% endhint %}

***

## Kamery LATTICE a světelné senzory DAQ na Linux

Všechny rodiny příkazů live-hardware fungují na Linux (amd64 a Jetson):

* **`chloros-cli lattice`** — vyhledávání, připojování, konfigurace a snímání z kamer LATTICE a synchronizovaných polí. `.deb` obsahuje runtime Arena SDK, který tyto funkce vyžadují, a registruje jej v dynamickém zavaděči.
* **`chloros-cli daq pool-*`** — připojuje světelné senzory DAQ-U/M/E prostřednictvím backendového poolu, streamuje kalibrovaná spektra a zaznamenává soubory `.daq`. Kompilovaná verze CLI obsahuje pouze rodinu `pool-*`: `pool-connect`, `pool-disconnect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`.
* **`chloros-cli project`** — spustit uložený projekt (jeho kamery, senzory a nastavení zpracování) bez grafického rozhraní.
* **`chloros-cli time-sync`** — zkontrolujte PTP grandmaster, na kterém běží backend Chloros pro kamery LATTICE a senzory DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` je vyžadován pro `pool-latest`, `pool-stream`, `pool-record`, a `pool-set-cap`; `pool-list` zobrazuje ID, která jsou aktuálně v zásobníku.

{% hint style="info" %}
**Pro první připojení DAQ-E na stroji s více síťovými rozhraními upřednostněte `--eth-host`.** Funkce automatického vyhledávání prohledává mDNS a může přehlédnout rozhraní senzoru kvůli prázdné mezipaměti ARP, takže první připojení `pool-connect --eth` po spuštění systému může selhat, i když je senzor zcela funkční. Zadáním IP adresy nebo názvu hostitele senzoru se vyhledávání zcela přeskočí.
{% endhint %}

**Oprávnění pro sériové rozhraní DAQ-U** jsou řízena nainstalovaným pravidlem udev (`uaccess` + skupina `dialout`). Pokud zůstává již připojený senzor nepřístupný, znovu načtěte pravidla nebo jej znovu připojte:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Úplný seznam příkazů najdete v [referenci k CLI](../CLI.md).

### Trvale aktivní PTP pro hostitele bez grafického rozhraní

Při první instalaci se vygeneruje jednotka systemd `chloros-backend.service`, která však **není povolena**. Na hostiteli Jetson bez grafického rozhraní nebo na serveru, který by měl udržovat synchronizaci času PTP nepřetržitě v chodu pro senzory DAQ-E a kamery LATTICE, ji povolte:

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

Bez ní běží PTP pouze tehdy, když je spuštěn backend Chloros — tedy během aktivní relace CLI/SDK.

Jednotka váže backend k nastavením prostředí `127.0.0.1:5000` (nastavení prostředí `CHLOROS_HOST` / `CHLOROS_PORT` uvnitř jednotky; přepsat pomocí `sudo systemctl edit chloros-backend.service`) a v případě selhání jej po 5 sekundách restartuje.

**Jak PTP získává své porty.** PTP používá UDP 319/320, oba pod běžnou hranicí 1024 privilegovaných portů. Balíček `postinst` zapíše do souboru `/etc/sysctl.d/60-chloros-ptp.conf` hodnoty `net.ipv4.ip_unprivileged_port_start = 319`, což umožňuje backendu navázat na tyto porty při běhu pod vaším uživatelským účtem. Jako preventivní opatření také aplikuje `setcap cap_net_bind_service,cap_net_raw=+ep` na binární soubor backendu – proto je `libcap2-bin` deklarovanou závislostí balíčku.***

## Příklady skriptů v Bash

{% hint style="info" %}
**Výstupní kódy vhodné pro skriptování.**`chloros-cli process` ukončí `0` v případě úspěchu a**nenulovou hodnotou v případě selhání — včetně běhu, který požadoval obrazové produkty, ale žádný nezapsal** (vypíše `Processing finished but wrote no image products.` a uvede název projektové složky a obvyklé příčiny). Úspěšná spuštění hlásí, kolik obrazových výstupů bylo zapsáno (`Image products written: N`). Kódy ukončení: `0` úspěch, `1` selhání, `2` chyba argumentu, `130` přerušeno.
{% endhint %}

### Zpracování více datových sad

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Zpracování s vlastním nastavením

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

Platných hodnot `--format` jsou přesně čtyři a obsahují mezery – vždy je uveďte v uvozovkách:

| Hodnota `--format` | Výstupní složka |
| --- | --- |
| `TIFF (16-bit)` *(výchozí)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` přijímá `standard` (výchozí) nebo `texture-aware` (Chloros+).

### Automatizované zpracování pomocí Cronu

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK Příklad

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Řešení problémů

### CLI nebyl po instalaci nalezen

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Oprávnění odepřeno

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### „setcap selhalo“ během instalace

`.deb` aplikuje `cap_net_bind_service` na `/usr/lib/chloros/chloros-backend`, aby bylo možné připojit PTP porty 319/320 bez oprávnění root. Pokud při instalaci chyběl `libcap2-bin`, volání se přeskočí. Nainstalujte jej a balíček nainstalujte znovu:

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### PTP se nespustí / nelze navázat port 319

Ověřte, zda byla snížena mezní hodnota pro porty bez oprávnění, a pokud ne, znovu ji použijte pro aktuální spuštění:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Poté zkontrolujte grandmaster:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### „Nebyly nalezeny ovladače kamer LATTICE“

Runtime Arena SDK se nenašel. Ověřte, zda je konfigurace zavaděče, kterou balíček zapisuje, přítomna a aktualizována:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
ls /usr/lib/chloros/arena_runtime | head
```

### Selhalo spuštění backendu

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

Protokoly backendu týkající se neúspěšného spuštění se nacházejí v souboru `~/.cache/chloros/logs/`.

### CUDA nebyla detekována

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

Soubor `chloros-cli selftest` hlásí totéž v jednom řádku: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Chybějící sdílené knihovny

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### Pomalé spouštění na systémech s SD kartou

Zkompilované binární soubory se při každém spuštění samy rozbalí do dočasného adresáře. Pokud existuje soubor `/mnt/ssd/tmp`, Chloros jej automaticky použije; v opačném případě nastavte `TMPDIR` na rychlý souborový systém:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Aktualizace Chloros na Linux

Příkaz `update` funguje pouze na Linux/pouze pro Jetson. Zkontroluje verzi zveřejněnou v aktualizačním kanálu nakonfigurovaném v `/etc/chloros/update.conf` a nabídne stažení a instalaci odpovídajícího `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Na Linux/Jetson provádí CLI při každém spuštění také neblokující kontrolu aktualizací (výsledek je v `~/.chloros/update_cache.json` uložen v mezipaměti po dobu jedné hodiny) a v případě existence novější verze zobrazí `Update available: vX.Y.Z`. Vaše nastavení a projekty zůstanou po aktualizaci zachovány; poté se budete muset znovu přihlásit.

## Odinstalace

```bash
sudo apt remove chloros
```

Odinstalace zastaví proces `chloros-backend.service`, obnoví výchozí minimální hodnotu pro neoprávněné porty (1024), odstraní symbolický odkaz na přibalený nástroj exiftool a konfiguraci zavaděče Arena a vymaže uložené přihlašovací údaje z mezipaměti. Vaše projekty a datové soubory `~/.chloros/` zůstanou nedotčeny.

***

## Další kroky

* [Průvodce NVIDIA Jetson](nvidia-jetson-guide.md) — optimalizace a nasazení specifické pro Jetson
* [CLI : Příkazový řádek](../CLI.md) — příručka CLI
* [API : Python SDK](../api-python-sdk.md) — příručka SDK
* [CLI Reference](../reference/cli-reference.md) a [SDK Reference](../reference/sdk-reference.md) — úplný seznam příkazů/API pro verzi 1.2.0
* [Dynamická adaptace výpočetního výkonu](../processing-architecture/dynamic-compute-adaptation.md) — jak se Chloros přizpůsobuje vašemu hardwaru
