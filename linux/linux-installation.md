# Instalace Linux

Chloros je pro Linux distribuován jako balíčky `.deb`, které instalují CLI a backend. Python SDK se instaluje samostatně pomocí pip.

***

## Linux amd64 (x86_64)

### Systémové požadavky

| Požadavek | Minimální | Doporučené |
| --- | --- | --- |
| **Distribuce** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **Procesor** | x86_64 (Intel/AMD) | Intel Core i7 nebo lepší |
| **Paměť (RAM)** | 8 GB | 16 GB nebo více |
| **Grafická karta** | Žádná (zpracování CPU) | GPU NVIDIA s 4 GB+ VRAM |
| **Úložiště** | 2 GB volného místa | SSD s 10 GB+ volného místa |
| **Python** | Python 3.7+ (pro SDK) | Python 3.10+ |

### Instalace

Stáhněte balíček `.deb` a nainstalujte jej:

```bash
sudo dpkg -i chloros-amd64.deb
```

Ověřte instalaci:

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### Systémové požadavky

| Požadavek | Minimální | Doporučené |
| --- | --- | --- |
| **Platforma** | NVIDIA Jetson s JetPack 6 | Jetson Orin NX 16 GB nebo AGX Orin |
| **JetPack** | JetPack 6.x | Nejnovější JetPack 6 |
| **Paměť (RAM)** | 8 GB (sdílená GPU/CPU) | 16 GB+ sdílená |
| **Úložiště** | 2 GB volného místa | NVMe SSD s 10 GB+ volného místa |
| **Python** | Python 3.7+ (pro SDK) | Python 3.10+ |

### Instalace

Stáhněte balíček JetPack 6 `.deb` a nainstalujte jej:

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

Ověřte instalaci:

```bash
chloros-cli --version
```

Podrobné informace o nastavení Jetsonu, včetně řízení teploty a nasazení v terénu, najdete v [Průvodci NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Python SDK Instalace (vše Linux)

Python SDK se instaluje samostatně přes pip a funguje jak na amd64, tak na arm64:

```bash
pip install chloros-sdk
```

Chcete-li zahrnout volitelnou podporu streamování průběhu:

```bash
pip install chloros-sdk[progress]
```

Ověřte SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Balíček `.deb` nainstaluje Chloros CLI a backend. Python SDK je samostatný balíček pip, který komunikuje s backendem prostřednictvím lokálního HTTP API.
{% endhint %}

***

## Konfigurační adresáře

Chloros na Linux se řídí [specifikací základního adresáře XDG](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html):

| Účel | Linux Cesta | Windows Ekvivalent |
| --- | --- | --- |
| **Konfigurace** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **Data / Projekty** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **Cache / Přihlašovací údaje** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## Umístění spustitelných souborů backendu

Balíček `.deb` nainstaluje backend do standardního umístění. Balíčky CLI a SDK automaticky detekují cestu k backendu:

| Způsob instalace | Cesta k backendu |
| --- | --- |
| Balíček `.deb` | `/usr/lib/chloros/chloros-backend` |
| Ruční / vlastní | `/opt/mapir/chloros/backend/chloros-backend` |

Cestu k backendu můžete přepsat pomocí příznaku `--backend-exe` CLI nebo parametru konstruktoru `backend_exe` SDK.

***

## První nastavení

### 1. Aktivujte svou licenci

Pro přístup k CLI a SDK je vyžadována licence Chloros+:

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. Zkontrolujte stav licence

```bash
chloros-cli status
```

### 3. Zpracujte svůj první datový soubor

```bash
chloros-cli process ~/datasets/flight001
```

### 4. Spusťte diagnostiku systému

Ověřte, zda je váš systém správně nakonfigurován:

```bash
chloros-cli selftest
```

Tím se spustí 7 diagnostických kontrol, včetně verze, spuštění backendu, připojení API a dostupnosti CUDA/GPU.

***

## Příklady skriptů v Bash

### Zpracování více datových sad

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### Zpracování s vlastními nastaveními

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### Automatizované zpracování pomocí Cronu

Přidejte do svého crontabu (`crontab -e`) pro automatické zpracování nových datových sad:

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

### CLI nenalezeno po instalaci

Pokud po instalaci balíčku `.deb` není nalezeno `chloros-cli`:

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### Oprávnění odepřeno

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
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

### CUDA nebyla detekována

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### Chybějící sdílené knihovny

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## Aktualizace Chloros na Linux

K vyhledání a instalaci aktualizací použijte vestavěný příkaz update:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## Další kroky

* [Průvodce NVIDIA Jetson](nvidia-jetson-guide.md) — Optimalizace a nasazení specifické pro Jetson
* [CLI : Příkazový řádek](../CLI.md) — Kompletní referenční příručka k příkazům CLI
* [API : Python SDK](../api-python-sdk.md) — Kompletní referenční příručka k SDK
* [Dynamická adaptace výpočetního výkonu](../processing-architecture/dynamic-compute-adaptation.md) — Jak se Chloros přizpůsobuje vašemu hardwaru
