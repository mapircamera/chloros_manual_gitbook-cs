# API : Python SDK

**Chloros Python SDK** poskytuje programový přístup k enginu pro zpracování obrazu Chloros, což umožňuje automatizaci, přizpůsobení pracovních postupů a hladkou integraci s vašimi aplikacemi Python a výzkumnými procesy.

### Klíčové vlastnosti

* 🐍 **Nativní Python** - Čistý, pythonský API pro zpracování obrazu
* 🔧 **Plný přístup k API** - Kompletní kontrola nad zpracováním Chloros
* 🚀 **Automatizace** - Vytvářejte vlastní pracovní postupy pro dávkové zpracování
* 🔗 **Integrace** – Vložte Chloros do stávajících aplikací Python
* 📊 **Připraveno pro výzkum** – Ideální pro vědecké analytické procesy
* ⚡ **Paralelní zpracování** – Škálovatelné podle počtu jader vašeho procesoru (Chloros+)

### Požadavky

| Požadavek          | Podrobnosti                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros Desktop**  | Musí být nainstalován lokálně                                           |
| **Licence**          | Chloros+ ([vyžaduje placený tarif](https://cloud.mapir.camera/pricing)) |
| **Operační systém** | Windows 10/11 (64bitový)                                              |
| **Python**           | Python 3.7 nebo vyšší                                                |
| **Paměť**           | Minimálně 8 GB RAM (doporučeno 16 GB)                                  |
| **Internet**         | Nutný pro aktivaci licence                                     |

{% hint style=&quot;warning&quot; %}
**Požadavky na licenci**: Python SDK vyžaduje placené předplatné Chloros+ pro přístup k API. Standardní (bezplatné) plány nemají přístup k API/SDK. Chcete-li provést upgrade, navštivte [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).
{% endhint %}

## Rychlý start

### Instalace

Instalace přes pip:

```bash
pip install chloros-sdk
```

{% hint style=&quot;info&quot; %}
**První nastavení**: Před použitím SDK aktivujte licenci Chloros+ otevřením Chloros, Chloros (prohlížeč) nebo Chloros CLI a přihlášením se pomocí svých přihlašovacích údajů. Toto je nutné provést pouze jednou.
{% endhint %}

### Základní použití

Zpracujte složku s několika řádky:

```python
from chloros_sdk import process_folder

# One-line processing
results = process_folder("C:\\DroneImages\\Flight001")
```

### Plná kontrola

Pro pokročilé pracovní postupy:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")

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

1. Nainstalovaný **Chloros Desktop** ([stáhnout](download.md))
2. Nainstalovaný **Python 3.7+** ([python.org](https://www.python.org))
3. **Aktivní licence Chloros+** ([upgrade](https://cloud.mapir.camera/pricing))

### Instalace pomocí pip

**Standardní instalace:**

```bash
pip install chloros-sdk
```

**S podporou sledování průběhu:**

```bash
pip install chloros-sdk[progress]
```

**Instalace pro vývoj:**

```bash
pip install chloros-sdk[dev]
```

### Ověření instalace

Otestujte, zda je SDK nainstalován správně:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## První nastavení

### Aktivace licence

SDK používá stejnou licenci jako Chloros, Chloros (prohlížeč) a Chloros CLI. Aktivujte jednou prostřednictvím grafického uživatelského rozhraní nebo CLI:

1. Otevřete **Chloros nebo Chloros (prohlížeč)** a přihlaste se na kartě Uživatel <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> . Nebo otevřete **CLI**.
2. Zadejte své přihlašovací údaje Chloros+ a přihlaste se
3. Licence je uložena v místní mezipaměti (zůstává zachována i po restartu)

{% hint style=&quot;success&quot; %}
**Jednorázové nastavení**: Po přihlášení přes GUI nebo CLI automaticky použije SDK uloženou licenci. Není potřeba žádné další ověření!
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

## API Reference

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
| `auto_start_backend`      | bool | `True`                    | V případě potřeby automaticky spustit backend |
| `backend_exe`             | str  | `None` (automatická detekce)      | Cesta k spustitelnému souboru backendu            |
| `timeout`                 | int  | `30`                      | Časový limit požadavku v sekundách            |
| `backend_startup_timeout` | int  | `60`                      | Časový limit pro spuštění backendu (sekundy) |

**Příklady:**

```python
# Default (auto-start backend)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom timeout
chloros = ChlorosLocal(timeout=60)
```

***

### Metody

#### `create_project(project_name, camera=None)`

Vytvoří nový projekt Chloros.

**Parametry:**

| Parametr      | Typ | Povinný | Popis                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Ano      | Název projektu                                     |
| `camera`       | str  | Ne       | Šablona kamery (např. „Survey3N\_RGN“, „Survey3W\_OCN“) |

**Vrací:** `dict` – Odpověď na vytvoření projektu

**Příklad:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Importuje obrázky ze složky.

**Parametry:**

| Parametr     | Typ     | Povinný | Popis                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Ano      | Cesta ke složce s obrázky         |
| `recursive`   | bool     | Ne       | Prohledat podsložky (výchozí: False) |

**Vrací:** `dict` – Výsledky importu s počtem souborů

**Příklad:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Konfigurace nastavení zpracování.

**Parametry:**

| Parametr                 | Typ | Výchozí hodnota                 | Popis                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | „Vysoká kvalita (rychlejší)“ | Metoda Debayer                  |
| `vignette_correction`     | bool | `True`                  | Povolit korekci viněty      |
| `reflectance_calibration` | bool | `True`                  | Povolit kalibraci odrazivosti  |
| `indices`                 | seznam | `None`                  | Vegetační indexy pro výpočet |
| `export_format`           | str  | „TIFF (16 bitů)“         | Výstupní formát                   |
| `ppk`                     | bool | `False`                 | Povolit korekce PPK          |
| `custom_settings`         | dict | `None`                  | Pokročilá vlastní nastavení        |

**Formáty exportu:**

* `"TIFF (16-bit)"` – doporučeno pro GIS/fotogrammetrii
* `"TIFF (32-bit, Percent)"` – vědecká analýza
* `"PNG (8-bit)"` – vizuální kontrola
* `"JPG (8-bit)"` – komprimovaný výstup

**Dostupné indexy:**

NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 a další.

**Příklad:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="High Quality (Faster)",
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
| `progress_callback` | callable | `None`       | Funkce zpětného volání průběhu (progress, msg) |
| `poll_interval`     | float    | `2.0`        | Interval dotazování průběhu (sekundy)   |

**Vrací:** `dict` – Výsledky zpracování

{% hint style=&quot;warning&quot; %}
**Paralelní režim**: Vyžaduje licenci Chloros+. Automaticky se přizpůsobí jádrům vašeho procesoru (až 16 pracovníků).
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

**Vrací:** `dict` – aktuální konfigurace projektu.

**Příklad:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Získá informace o stavu backendu.

**Vrací:** `dict` – stav backendu

**Příklad:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
```

***

#### `shutdown_backend()`

Vypne backend (pokud byl spuštěn pomocí SDK).

**Příklad:**

```python
chloros.shutdown_backend()
```

***

### Pomocné funkce

#### `process_folder(folder_path, **options)`

Jednořádková pomocná funkce pro zpracování složky.

**Parametry:**

| Parametr                 | Typ     | Výchozí hodnota         | Popis                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Povinné        | Cesta ke složce s obrázky     |
| `project_name`            | str      | Automaticky generované  | Název projektu                   |
| `camera`                  | str      | `None`          | Šablona kamery                |
| `indices`                 | list     | `["NDVI"]`      | Indexy pro výpočet           |
| `vignette_correction`     | bool     | `True`          | Povolit korekci viněty     |
| `reflectance_calibration` | bool     | `True`          | Povolit kalibraci odrazivosti |
| `export_format`           | str      | „TIFF (16bitový)“ | Výstupní formát                  |
| `mode`                    | str      | `"parallel"`    | Režim zpracování                |
| `progress_callback`       | volatelný | `None`          | Zpětné volání průběhu              |

**Vrací:** `dict` – Výsledky zpracování

**Příklad:**

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

SDK podporuje správce kontextu pro automatické čištění:

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

### Příklad 1: Základní zpracování

Zpracování složky s výchozím nastavením:

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
    debayer="High Quality (Faster)",
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

### Příklad 3: Hromadné zpracování více složek

Zpracování více datových sad letů:

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

### Příklad 4: Integrace výzkumného potrubí

Integrace Chloros s analýzou dat:

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
        return False, f"Backend error: {e}. Ensure Chloros Desktop is installed."
    
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

### Příklad 7: Nástroj příkazového řádku

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
    
    args = parser.parse_args()
    
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
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI
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
    print("Backend failed to start. Ensure Chloros Desktop is installed.")

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

### Ne blokující zpracování

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

**Problém:** SDK se nedaří spustit backend

**Řešení:**

1. Ověřte, zda je nainstalován Chloros Desktop:

```python
import os
backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Zkontrolujte, zda Windows Firewall neblokuje
3. Zkuste ruční cestu k backendu:

```python
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")
```

***

### Licence nebyla detekována

**Problém:** SDK varuje před chybějící licencí

**Řešení:**

1. Otevřete Chloros, Chloros (prohlížeč) nebo Chloros CLI a přihlaste se.
2. Ověřte, zda je licence uložena v mezipaměti:

```python
from pathlib import Path
import os

# Check cache location (Windows)
cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
print(f"Cache exists: {cache_path.exists()}")
```

3. Kontaktujte podporu: info@mapir.camera

***

### Chyby při importu

**Problém:** `ModuleNotFoundError: No module named 'chloros_sdk'`

**Řešení:**

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

### Časový limit zpracování

**Problém:** Časový limit zpracování vypršel.

**Řešení:**

1. Prodloužit časový limit:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Zpracovat menší dávky.
3. Zkontrolovat dostupný prostor na disku.
4. Sledovat systémové zdroje.

***

### Port již je používán

**Problém:** Backend port 5000 je obsazen.

**Řešení:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Nebo najděte a ukončete konfliktní proces:

```powershell
# PowerShell
Get-NetTCPConnection -LocalPort 5000
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

3. **Deaktivujte nepotřebné indexy**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Zpracujte na SSD** (nikoli HDD)

***

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

### Integrace Django

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

**Odpověď:** Pouze pro počáteční aktivaci licence. Po přihlášení přes Chloros, Chloros (prohlížeč) nebo Chloros CLI se licence uloží do místní mezipaměti a funguje offline po dobu 30 dnů.

***

### Otázka: Mohu používat SDK na serveru bez grafického uživatelského rozhraní?

**Odpověď:** Ano! Požadavky:

* Windows Server 2016 nebo novější
* Nainstalovaný Chloros (jednorázově)
* Licence aktivovaná na libovolném počítači (licence uložená v mezipaměti zkopírovaná na server)

***

### Otázka: Jaký je rozdíl mezi Desktop, CLI a SDK?

| Funkce         | Desktop GUI | CLI Příkazový řádek | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Rozhraní**   | Ukazatel myši | Příkazový řádek | Python API  |
| **Nejvhodnější pro**    | Vizuální práce | Skriptování        | Integrace |
| **Automatizace**  | Omezená     | Dobrá             | Vynikající   |
| **Flexibilita** | Základní       | Dobrá             | Maximální     |
| **Licence**     | Chloros+    | Chloros+         | Chloros+    |

***

### Otázka: Mohu distribuovat aplikace vytvořené pomocí SDK?

**Odpověď:** Kód SDK lze integrovat do vašich aplikací, ale:

* Koncoví uživatelé musí mít nainstalovaný Chloros.
* Koncoví uživatelé musí mít aktivní licence Chloros+.
* Komerční distribuce vyžaduje OEM licence.

Pro dotazy týkající se OEM kontaktujte info@mapir.camera.

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

### Otázka: Mohu zpracovávat obrázky ze skriptů Python spouštěných podle plánu?

**Odpověď:** Ano! Použijte plánovač úloh Windows se skripty Python:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("C:\\Flights\\Today")
```

Naplánujte pomocí plánovače úloh denní spouštění.

***

### Otázka: Podporuje SDK async/await?

**Odpověď:** Aktuální verze je synchronní. Pro asynchronní chování použijte `wait=False` nebo spusťte v samostatném vlákně:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

## Nápověda

### Dokumentace

* **API Reference**: Tato stránka

### Kanály podpory

* **E-mail**: info@mapir.camera
* **Webové stránky**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Ceny**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Ukázkový kód

Všechny zde uvedené příklady jsou otestované a připravené k použití. Zkopírujte je a přizpůsobte pro své potřeby.

***

## Licence

**Proprietární software** – Copyright (c) 2025 MAPIR Inc.

SDK vyžaduje aktivní předplatné Chloros+. Neoprávněné použití, distribuce nebo úprava jsou zakázány.
