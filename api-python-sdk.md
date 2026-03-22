# API : Python SDK

**Chloros Python SDK** poskytuje programový přístup k enginu pro zpracování obrazu Chloros, což umožňuje automatizaci, vlastní pracovní postupy a hladkou integraci s vašimi aplikacemi Python a výzkumnými procesy.

### Klíčové vlastnosti

* 🐍 **Nativní Python** – Čistý, pythonský API pro zpracování obrazu
* 🔧 **Plný přístup k API** – Úplná kontrola nad zpracováním Chloros
* 🚀 **Automatizace** – Vytvářejte vlastní pracovní postupy pro dávkové zpracování
* 🔗 **Integrace** – Vložte Chloros do stávajících Python aplikací
* 📊 **Připraveno pro výzkum** – Ideální pro vědecké analytické procesy
* ⚡ **Paralelní zpracování** – Škálovatelné podle počtu jader vašeho procesoru (Chloros+)

### Požadavky

| Požadavek          | Podrobnosti                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros nainstalován** | Windows: Instalační program pro stolní počítače; Linux: balíček `.deb`                  |
| **Licence**          | Chloros+ ([vyžaduje placený tarif](https://cloud.mapir.camera/pricing)) |
| **Operační systém** | Windows 10/11 (64bitový), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Python**           | Python 3.7 nebo vyšší                                                |
| **Paměť**           | Minimálně 8 GB RAM (doporučeno 16 GB)                                  |
| **Internet**         | Vyžadováno pro aktivaci licence                                     |

{% hint style="warning" %}
**Požadavky na licenci**: Python SDK vyžaduje placené předplatné Chloros+ pro přístup k API. Standardní (bezplatné) tarify nemají přístup k API/SDK. Chcete-li provést upgrade, navštivte [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).
{% endhint %}

## Rychlý start

### Instalace

Instalace přes pip:

```bash
pip install chloros-sdk
```

{% hint style="info" %}
**První nastavení**: Před použitím SDK aktivujte licenci Chloros+ otevřením Chloros, Chloros (prohlížeč) nebo Chloros CLI a přihlášením se pomocí svých přihlašovacích údajů. Toto je třeba provést pouze jednou. V Linux (bez grafického rozhraní) použijte: `chloros-cli login user@example.com 'password'`
{% endhint %}

### Základní použití

Zpracujte složku pomocí několika řádků:

```python
from chloros_sdk import process_folder

# One-line processing (Windows)
results = process_folder("C:\\DroneImages\\Flight001")

# One-line processing (Linux)
results = process_folder("/home/user/drone_images/flight001")
```

{% hint style="info" %}
**Cesty napříč platformami**: Příklady kódu na této stránce používají cesty ve stylu Windows (např. `C:\\DroneImages\\Flight001`). V systému Linux použijte místo toho cesty ve stylu Linux (např. `/home/user/drone_images/flight001` nebo `~/drone_images/flight001`). SDK funguje na obou platformách stejně.
{% endhint %}

### Plná kontrola

Pro pokročilé pracovní postupy:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")  # Windows
# chloros.import_images("/home/user/drone_images/flight001")  # Linux

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## Průvodce instalací

### Předpoklady

Před instalací SDK se ujistěte, že máte:

1. **nainstalovaný Chloros** — Windows: Instalační program pro stolní počítače ([stáhnout](download.md)); Linux: balíček `.deb` ([Instalace](linux/linux-installation.md))
2. **Python 3.7+** nainstalováno ([python.org](https://www.python.org))
3. **Aktivní licence Chloros+** ([upgrade](https://cloud.mapir.camera/pricing))

### Instalace přes pip

**Standardní instalace:**

```bash
pip install chloros-sdk
```

**S podporou sledování průběhu:**

```bash
pip install chloros-sdk[progress]
```

**Vývojová instalace:**

```bash
pip install chloros-sdk[dev]
```

### Ověření instalace

Ověřte, zda je SDK nainstalován správně:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## První nastavení

### Aktivace licence

SDK používá stejnou licenci jako Chloros, Chloros (prohlížeč) a Chloros CLI. Aktivujte jednou prostřednictvím grafického uživatelského rozhraní nebo CLI:**Windows:**Otevřete**Chloros nebo Chloros (prohlížeč)** a přihlaste se na kartě Uživatel <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> nebo použijte CLI.**Linux:** Použijte CLI (GUI není k dispozici):

```bash
chloros-cli login user@example.com 'your_password'
```

Licence je uložena v lokální mezipaměti a zůstává zachována i po restartu.

{% hint style="success" %}
**Jednorázové nastavení**: Po přihlášení přes grafické rozhraní nebo CLI použije SDK automaticky uloženou licenci. Není potřeba žádné další ověřování!
{% endhint %}

{% hint style="info" %}
**Odhlášení**: Uživatelé SDK mohou programově vymazat uložené přihlašovací údaje pomocí metody `logout()`. Viz [metoda logout()](#logout) v referenční příručce API.
{% endhint %}

### Test připojení

Ověřte, zda se SDK může připojit k Chloros:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## Referenční příručka API

### Třída ChlorosLocal

Hlavní třída pro lokální zpracování obrazu Chloros.

#### Konstruktor

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**Parametry:**

| Parametr                 | Typ | Výchozí hodnota                   | Popis                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL lokálního backendu Chloros          |
| `auto_start_backend`      | bool | `True`                    | Automaticky spustit backend v případě potřeby |
| `backend_exe`             | str  | `None` (automatická detekce)      | Cesta ke spustitelnému souboru backendu            |
| `timeout`                 | int  | `30`                      | Časový limit požadavku v sekundách            |
| `backend_startup_timeout` | int  | `60`                      | Časový limit pro spuštění backendu (sekundy) |

**Příklady:**

```python
# Default (auto-start backend, auto-detect path on Windows and Linux)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path (Windows)
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom backend path (Linux)
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")

# Custom timeout with longer startup (e.g., for Jetson)
chloros = ChlorosLocal(timeout=60, backend_startup_timeout=120)
```

{% hint style="info" %}
**Automatická detekce napříč platformami**: SDK automaticky vyzkouší správnou cestu k backendu pro vaši platformu:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (ručně)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

***

### Metody

#### `create_project(project_name, camera=None)`

Vytvořte nový projekt Chloros.

**Parametry:**

| Parametr      | Typ | Povinné | Popis                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Ano      | Název projektu                                     |
| `camera`       | str  | Ne       | Šablona kamery (např. „Survey3N\_RGN“, „Survey3W\_OCN“) |

**Vrací:** `dict` – Odpověď na vytvoření projektu**Příklad:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Import obrázků ze složky.

**Parametry:**

| Parametr     | Typ     | Povinný | Popis                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Ano      | Cesta ke složce s obrázky         |
| `recursive`   | bool     | Ne       | Prohledat podsložky (výchozí: False) |

**Vrací:** `dict` - Výsledky importu s počtem souborů**Příklad:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Nakonfigurujte nastavení zpracování.

**Parametry:**

| Parametr                 | Typ | Výchozí hodnota                 | Popis                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | „Standard (rychlé, střední kvalita)“ | Metoda debayeringu            |
| `vignette_correction`     | bool | `True`                  | Povolit korekci vinětace      |
| `reflectance_calibration` | bool | `True`                  | Zapnout kalibraci odrazivosti  |
| `indices`                 | list | `None`                  | Vegetace indexy k výpočtu |
| `export_format`           | str  | „TIFF (16-bit)“         | Výstupní formát                   |
| `ppk`                     | bool | `False`                 | Povolit korekce PPK          |
| `custom_settings`         | dict | `None`                  | Pokročilá vlastní nastavení        |

**Formáty exportu:**

* `"TIFF (16-bit)"` – Doporučeno pro GIS/fotogrammetrii
* `"TIFF (32-bit, Percent)"` – Vědecká analýza
* `"PNG (8-bit)"` – Vizuální kontrola
* `"JPG (8-bit)"` – Komprimovaný výstup

**Dostupné indexy:**NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 a další.**Příklad:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

Zpracujte obrázky projektu.

**Parametry:**

| Parametr           | Typ     | Výchozí hodnota      | Popis                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Režim zpracování: „parallel“ nebo „serial“   |
| `wait`              | bool     | `True`       | Čekání na dokončení                       |
| `progress_callback` | callable | `None`       | Funkce zpětného volání pro průběh (progress, msg) |
| `poll_interval`     | float    | `2.0`        | Interval dotazování na průběh (sekundy)   |

**Vrací:** `dict` – Výsledky zpracování

{% hint style="warning" %}
**Paralelní režim**: Vyžaduje licenci Chloros+. Automaticky se škáluje podle počtu jader vašeho procesoru (až 16 pracovníků).
{% endhint %}

**Příklad:**

```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```

***

#### `get_config()`

Získá aktuální konfiguraci projektu.

**Vrací:** `dict` – Aktuální konfigurace projektu**Příklad:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Získá informace o stavu backendu včetně průběhu zpracování pro jednotlivá vlákna.

**Vrací:** `dict` – Stav backendu s následující strukturou:

```python
{
    "running": True,
    "url": "http://localhost:5000",
    "processing": {
        "percent": 75.0,
        "phase": "processing"
    },
    "export": {
        "percent": 50.0,
        "phase": "exporting",
        "active": True
    }
}
```

**Příklad:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
print(f"Processing: {status['processing']['percent']}%")
print(f"Export: {status['export']['percent']}% - Active: {status['export']['active']}")
```

***

#### `shutdown_backend()`

Ukončí backend (pokud byl spuštěn příkazem SDK).

**Příklad:**

```python
chloros.shutdown_backend()
```

***

#### `logout()`

Vymaže uložené přihlašovací údaje z mezipaměti lokálního systému.

**Popis:**

Programově se odhlásí odstraněním uložených přihlašovacích údajů. To je užitečné pro:
* Přepínání mezi různými účty Chloros+
* Vymazání přihlašovacích údajů v automatizovaných prostředích
* bezpečnostní účely (např. odstranění přihlašovacích údajů před odinstalací)

**Vrací:** `dict` – výsledek operace odhlášení**Příklad:**

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Clear cached credentials
result = chloros.logout()
print(f"Logout successful: {result}")

# After logout, login required via GUI/CLI/Browser before next SDK use
```

{% hint style="info" %}
**Vyžadováno opětovné ověření**: Po volání `logout()` se musíte znovu přihlásit pomocí Chloros, Chloros (prohlížeč) nebo Chloros CLI před použitím SDK.
{% endhint %}

***

### Pomocné funkce

#### `process_folder(folder_path, **options)`

Jednořádková pomocná funkce pro zpracování složky.

**Parametry:**

| Parametr                 | Typ     | Výchozí hodnota         | Popis                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Povinné        | Cesta ke složce s obrázky     |
| `project_name`            | str      | Automaticky generováno  | Název projektu                   |
| `camera`                  | str      | `None`          | Šablona fotoaparátu                |
| `indices`                 | seznam     | `["NDVI"]`      | Indexy pro výpočet           |
| `vignette_correction`     | bool     | `True`          | Povolit korekci vinětace     |
| `reflectance_calibration` | bool     | `True`          | Zapnout kalibraci odrazivosti |
| `export_format`           | str      | &quot;TIFF (16-bit)&quot; | Výstupní formát                  |
| `mode`                    | str      | `"parallel"`    | Režim zpracování                |
| `progress_callback`       | callable | `None`          | Zpětné volání průběhu              |

**Vrací:** `dict` - Výsledky zpracování**Příklad:**

```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```

***

## Podpora správce kontextu

SDK podporuje správce kontextu pro automatické uvolnění:

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***

## Kompletní příklady

{% hint style="info" %}
**Uživatelé Linux**: Všechny níže uvedené příklady používají cesty Windows. Nahraďte cesty `C:\\...` svými cestami Linux (např. `/home/user/...` nebo `~/...`). Veškeré funkce SDK jsou na všech platformách identické.
{% endhint %}

### Příklad 1: Základní zpracování

Zpracujte složku s výchozím nastavením:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Příklad 2: Vlastní pracovní postup

Plná kontrola nad zpracovatelským potrubím:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### Příklad 3: Dávkové zpracování více složek

Zpracování více letových datových sad:

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***

### Příklad 4: Integrace do výzkumného procesu

Integrace Chloros do analýzy dat:

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```

***

### Příklad 5: Vlastní sledování průběhu

Pokročilé sledování průběhu s protokolováním:

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***

### Příklad 6: Zpracování chyb

Robustní zpracování chyb pro produkční použití:

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros is installed (Windows installer or Linux .deb package)."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```

***

### Příklad 7: Správa účtů a odhlášení

Programové řízení přihlašovacích údajů:

```python
from chloros_sdk import ChlorosLocal

def switch_account():
    """Clear credentials to switch to a different account"""
    try:
        chloros = ChlorosLocal()
        
        # Clear current credentials
        result = chloros.logout()
        print("✓ Credentials cleared successfully")
        print("Please log in with new account via Chloros, Chloros (Browser), or CLI")
        
        return True
    
    except Exception as e:
        print(f"✗ Logout failed: {e}")
        return False

def secure_cleanup():
    """Remove credentials for security purposes"""
    try:
        chloros = ChlorosLocal()
        chloros.logout()
        print("✓ Credentials removed for security")
        
    except Exception as e:
        print(f"Warning: Cleanup error: {e}")

# Switch accounts
if switch_account():
    print("\nRe-authenticate via Chloros GUI/CLI/Browser before next SDK use")

# Or perform secure cleanup
# secure_cleanup()
```

***

### Příklad 8: Nástroj příkazového řádku

Vytvořte vlastní nástroj CLI pomocí SDK:

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    parser.add_argument('--logout', action='store_true',
                       help='Clear cached credentials before processing')
    
    args = parser.parse_args()
    
    # Handle logout if requested
    if args.logout:
        from chloros_sdk import ChlorosLocal
        chloros = ChlorosLocal()
        chloros.logout()
        print("Credentials cleared. Please re-login via Chloros GUI/CLI/Browser.")
        return 0
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**Použití:**

```bash
# Process multiple folders
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI

# Clear cached credentials
python my_processor.py --logout
```

***

## Zpracování výjimek

SDK poskytuje specifické třídy výjimek pro různé typy chyb:

### Hierarchie výjimek

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### Příklady výjimek

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros is installed (Windows installer or Linux .deb package).")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Pokročilá témata

### Vlastní konfigurace backendu

Použijte vlastní umístění nebo konfiguraci backendu:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Neblokující zpracování

Spusťte zpracování a pokračujte v dalších úkolech:

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### Správa paměti

U velkých datových sad zpracovávejte v dávkách:

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***

## Řešení problémů

### Backend se nespustí

**Problém:** SDK se nedaří spustit backend**Řešení:**

1. Ověřte, zda je nainstalován Chloros:

```python
import os
import platform

# Auto-detect backend path
if platform.system() == "Windows":
    backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
else:
    backend_path = "/usr/lib/chloros/chloros-backend"

print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Zkontrolujte firewall (Windows) nebo dostupnost portu (Linux: `lsof -i :5000`)
3. Zkuste ruční zadání cesty k backendu:

```python
# Windows
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")

# Linux
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")
```

***

### Licence nebyla detekována**Problém:** SDK upozorňuje na chybějící licenci**Řešení:**

1. Otevřete Chloros, Chloros (prohlížeč) nebo Chloros CLI a přihlaste se.
2. Ověřte, zda je licence uložena v mezipaměti:

```python
from pathlib import Path
import os
import platform

# Check cache location
if platform.system() == "Windows":
    cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
else:
    cache_path = Path.home() / '.cache' / 'chloros'

print(f"Cache exists: {cache_path.exists()}")
```

3. Pokud máte problémy s přihlašovacími údaji, vymažte uložené přihlašovací údaje a přihlaste se znovu:

```python
from chloros_sdk import ChlorosLocal

# Clear cached credentials
chloros = ChlorosLocal()
chloros.logout()

# Then login again via Chloros, Chloros (Browser), or Chloros CLI
```

4. Kontaktujte podporu: info@mapir.camera

***

### Chyby importu**Problém:** `ModuleNotFoundError: No module named 'chloros_sdk'`**Řešení:**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### Časový limit zpracování**Problém:** Vypršel časový limit zpracování**Řešení:**

1. Zvyšte časový limit:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Zpracovávejte menší dávky
3. Zkontrolujte dostupné místo na disku
4. Sledujte systémové zdroje

***

### Port již používán**Problém:** Port 5000 na backendu je obsazen**Řešení:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Nebo vyhledejte a ukončete konfliktní proces:

```powershell
# Windows PowerShell
Get-NetTCPConnection -LocalPort 5000
```

```bash
# Linux
lsof -i :5000
kill $(lsof -t -i :5000)
```

***

## Tipy pro zvýšení výkonu

### Optimalizace rychlosti zpracování

1. **Použijte paralelní režim** (vyžaduje Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Snižte výstupní rozlišení** (pokud je to přijatelné)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Zakázat nepotřebné indexy**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Zpracovávat na SSD** (ne na HDD)***

### Optimalizace paměti

Pro velké datové sady:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Zpracování na pozadí

Uvolněte Python pro jiné úkoly:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## Příklady integrace

### Integrace s Django

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyter Notebook

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***

## Často kladené otázky

### Otázka: Vyžaduje SDK připojení k internetu?

**Odpověď:** Pouze pro počáteční aktivaci licence. Po přihlášení přes Chloros, Chloros (prohlížeč) nebo Chloros CLI se licence uloží do lokální mezipaměti a funguje offline po dobu 30 dnů.***

### Otázka: Mohu použít SDK na serveru bez grafického rozhraní?**Odpověď:** Ano! SDK funguje bez grafického rozhraní na serverech Windows i Linux.**Linux (doporučeno pro bezhlavý režim):**
* Instalace prostřednictvím balíčku `.deb`
* Aktivace licence: `chloros-cli login user@example.com 'password'`

**Server Windows:**
* Windows Server 2016 nebo novější
* Nainstalován Chloros (jednorázově)
* Licence aktivována pomocí CLI nebo na jakémkoli počítači

***

### Otázka: Jaký je rozdíl mezi Desktop, CLI a SDK?

| Funkce         | Grafické rozhraní Desktop | Příkazový řádek CLI | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Rozhraní**   | Myš a kliknutí | Příkazový řádek | Python API  |
| **Nejvhodnější pro**    | Vizuální práci | Skriptování        | Integraci |
| **Automatizace**  | Omezená     | Dobrá             | Vynikající   |
| **Flexibilita** | Základní       | Dobrá             | Maximální     |
| **Licence**     | Chloros+    | Chloros+         | Chloros+    |***

### Otázka: Mohu distribuovat aplikace vytvořené pomocí SDK?**Odpověď:** Kód SDK lze integrovat do vašich aplikací, ale:

* Koncoví uživatelé musí mít nainstalován Chloros
* Koncoví uživatelé musí mít aktivní licence Chloros+
* Komerční distribuce vyžaduje OEM licenci

V případě dotazů ohledně OEM se obraťte na info@mapir.camera.

***

### Otázka: Jak aktualizuji SDK?

```bash
pip install --upgrade chloros-sdk
```

***

### Otázka: Kam se ukládají zpracované obrázky?

Ve výchozím nastavení do cesty projektu:

```

Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### Otázka: Mohu zpracovávat obrázky ze skriptů Python spouštěných podle plánu?**Odpověď:** Ano! Použijte plánovač operačního systému se skripty Python:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("/data/flights/today")  # Linux
# results = process_folder("C:\\Flights\\Today")  # Windows
```

**Windows:** Naplánujte pomocí Plánovače úloh denní spuštění.**Linux:** Naplánujte pomocí cronu:

```cron
# Run at 2 AM daily
0 2 * ** /usr/bin/python3 /home/user/scheduled_processing.py >> /var/log/chloros.log 2>&1
```

***

### Otázka: Podporuje SDK async/await?**Odpověď:** Aktuální verze je synchronní. Pro asynchronní chování použijte `wait=False` nebo spusťte v samostatném vlákně:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

### Otázka: Jak přepínám mezi různými účty Chloros+?**Odpověď:** Použijte metodu `logout()` k vymazání uložených přihlašovacích údajů a poté se znovu přihlaste pomocí nového účtu:

```python
from chloros_sdk import ChlorosLocal

# Clear current credentials
chloros = ChlorosLocal()
chloros.logout()

# Re-login via Chloros, Chloros (Browser), or Chloros CLI with new account
```

Po odhlášení se ověřte pomocí nového účtu přes grafické rozhraní, prohlížeč nebo CLI, než znovu použijete SDK.

***

## Získání pomoci

### Dokumentace

* **API Reference**: Tato stránka

### Kanály podpory

* **E-mail**: info@mapir.camera
* **Webové stránky**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Ceny**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Ukázkový kód

Všechny zde uvedené příklady jsou otestovány a připraveny k použití v produkčním prostředí. Zkopírujte je a přizpůsobte pro svůj případ použití.

***

## Licence**Proprietární software** – Copyright (c) 2025 MAPIR Inc.

SDK vyžaduje aktivní předplatné Chloros+. Neoprávněné použití, distribuce nebo úpravy jsou zakázány.
