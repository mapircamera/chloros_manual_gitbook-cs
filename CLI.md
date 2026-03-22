# CLI : Příkazový řádek

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** poskytuje výkonný přístup z příkazového řádku k enginu pro zpracování obrazu Chloros, což umožňuje automatizaci, skriptování a bezobslužný provoz vašich pracovních postupů v oblasti zpracování obrazu.

### Klíčové funkce

* 🚀 **Automatizace** – Skriptové dávkové zpracování více datových sad
* 🔗 **Integrace** – Začlenění do stávajících pracovních postupů a procesů
* 💻 **Provoz bez grafického rozhraní** – Spuštění bez grafického uživatelského rozhraní
* 🌍 **Více jazyků** – Podpora 38 jazyků
* ⚡ **Paralelní zpracování** – [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) se automaticky optimalizuje pro váš hardware

### Požadavky

| Požadavek          | Podrobnosti                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Operační systém** | Windows 10/11 (64-bit), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Licence**          | Chloros+ ([vyžaduje placený tarif](https://cloud.mapir.camera/pricing)) |
| **Paměť**           | Minimálně 8 GB RAM (doporučeno 16 GB)                                  |
| **Internet**         | Nutný pro aktivaci licence                                     |
| **Místo na disku**       | Liší se podle velikosti projektu                                              |

{% hint style="warning" %}
**Požadavky na licenci**: CLI vyžaduje placené předplatné Chloros+. Standardní (bezplatné) tarify nemají přístup k CLI. Chcete-li provést upgrade, navštivte [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).
{% endhint %}

## Rychlý start

### Instalace

#### Windows

CLI je automaticky součástí instalačního programu Chloros:

1. Stáhněte a spusťte **Chloros Installer.exe**

2. Dokončete průvodce instalací
3. CLI nainstalován do: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
Instalační program automaticky přidá `chloros-cli` do systémové proměnné PATH. Po instalaci restartujte terminál.
{% endhint %}

#### Linux

Nainstalujte balíček `.deb` pro vaši architekturu:

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Podrobné informace o nastavení Linux najdete v části [Instalace Linux](linux/linux-installation.md).

### První nastavení

Před použitím CLI aktivujte licenci Chloros+:

**Windows:**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### Základní použití

Zpracujte složku s výchozím nastavením:

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## Referenční příkazy

### Obecná syntaxe

```
chloros-cli [global-options] <command> [command-options]
```

***

## Příkazy

### `process` – Zpracování obrázků

Zpracuje obrázky ve složce s kalibrací.

**Syntaxe:**

```bash
chloros-cli process <input-folder> [options]
```

**Příklady:**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### Možnosti příkazu zpracování

| Možnost                | Typ    | Výchozí hodnota        | Popis                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Cesta    | _Povinné_     | Složka obsahující multispektrální snímky ve formátu RAW/JPG                                         |
| `-o, --output`        | Cesta    | Stejná jako vstup  | Výstupní složka pro zpracované snímky                                                     |
| `-n, --project-name`  | Řetězec  | Automaticky generováno | Vlastní název projektu                                                                    |
| `--vignette`          | Příznak    | Zapnuto        | Zapnout korekci vinětace                                                             |
| `--no-vignette`       | Příznak    | -              | Vypnout korekci vinětace                                                            |
| `--reflectance`       | Příznak    | Zapnuto        | Zapnout kalibraci odrazivosti                                                         |
| `--no-reflectance`    | Příznak    | -              | Vypnout kalibraci odrazivosti                                                        |
| `--ppk`               | Příznak    | Zakázáno       | Použít korekce PPK z dat světelného senzoru .daq                                      |
| `--format`            | Volba  | TIFF (16bitový)  | Výstupní formát: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Celé číslo | Auto           | Minimální velikost cíle v pixelech pro detekci kalibračního panelu                          |
| `--target-clustering` | Celé číslo | Auto           | Prahová hodnota shlukování cílů (0–100)                                                    |
| `--debayer`           | Volba  | `standard`     | Metoda debayeringu: `standard` nebo `texture-aware` (pouze Chloros+)                          |
| `--target`, `--targets` | Příznak  | Zakázáno       | Hledat kalibrační cíle pouze v podsložce „target“ nebo „targets“ (urychluje zpracování) |
| `--indices`           | Seznam    | Žádné           | Vegetace indexy k výpočtu (např. `--indices NDVI NDRE GNDVI`)                    |
| `--exposure-pin-1`    | Řetězec  | Žádné           | Uzamknout expozici pro model kamery (Pin 1)                                                 |
| `--exposure-pin-2`    | Řetězec  | Žádné           | Uzamčení expozice pro model kamery (pin 2)                                                 |
| `--recal-interval`    | Celé číslo | Auto           | Interval rekalibrace v sekundách                                                      |
| `--timezone-offset`   | Celé číslo | 0              | Časový posun v hodinách                                                               |

***

### `login` – Ověření účtu

Přihlaste se pomocí svých přihlašovacích údajů Chloros+, abyste povolili zpracování CLI.

**Syntaxe:**

```bash
chloros-cli login <email> <password>
```

**Příklad:**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Zvláštní znaky**: Hesla obsahující znaky jako `$`, `!` nebo mezery uveďte v jednoduchých uvozovkách.
{% endhint %}

**Výstup:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` – Vymazat přihlašovací údaje

Vymazat uložené přihlašovací údaje a odhlásit se z účtu.

**Syntaxe:**

```bash
chloros-cli logout
```

**Příklad:**

```bash
chloros-cli logout
```

**Výstup:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**Uživatelé SDK**: Python SDK také poskytuje programovou metodu `logout()` pro vymazání přihlašovacích údajů v rámci skriptů Python. Podrobnosti najdete v [dokumentaci k Python SDK](api-python-sdk.md#logout).
{% endhint %}

***

### `status` – Zkontrolovat stav licence

Zobrazí aktuální stav licence a ověření.

**Syntaxe:**

```bash
chloros-cli status
```

**Příklad:**

```bash
chloros-cli status
```

**Výstup:**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` – Zkontrolovat průběh exportu

Sledujte průběh exportu vlákna 4 během zpracování nebo po něm.

**Syntaxe:**

```bash
chloros-cli export-status
```

**Příklad:**

```bash
chloros-cli export-status
```

**Případ použití:** Tento příkaz vyvolejte během zpracování, abyste zkontrolovali průběh exportu.***

### `language` – Správa jazyka rozhraní

Zobrazte nebo změňte jazyk rozhraní CLI.

**Syntaxe:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Příklady:**

```bash
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Podporované jazyky (celkem 38)

| Kód    | Jazyk              | Nativní název      |
| ------- | --------------------- | ---------------- |
| `en`    | Angličtina               | English          |
| `es`    | Španělština               | Español          |
| `pt`    | Portugalština            | Português        |
| `fr`    | Francouzština                | Français         |
| `de`    | Němčina                | Deutsch          |
| `it`    | Italština               | Italiano         |
| `ja`    | Japonština              | 日本語              |
| `ko`    | Korejština                | 한국어              |
| `zh`    | Čínština (zjednodušená)  | 简体中文             |
| `zh-TW` | Čínština (tradiční) | 繁體中文             |
| `ru`    | Ruština               | Русский          |
| `nl`    | Nizozemština                 | Nederlands       |
| `ar`    | Arabština                | العربية          |
| `pl`    | Polština                | Polski           |
| `tr`    | Turečtina               | Türkçe           |
| `hi`    | hindština                 | हिंदी            |
| `id`    | indonéština            | Bahasa Indonesia |
| `vi`    | vietnamština            | Tiếng Việt       |
| `th`    | Thajština                  | ไทย              |
| `sv`    | Švédština               | Svenska          |
| `da`    | Dánština                | Dansk            |
| `no`    | Norština             | Norsk            |
| `fi`    | finština               | Suomi            |
| `el`    | řečtina                 | Ελληνικά         |
| `cs`    | čeština                 | Čeština          |
| `hu`    | Maďarština             | Magyar           |
| `ro`    | Rumunština              | Română           |
| `uk`    | Ukrajinština             | Українська       |
| `pt-BR` | Brazilská portugalština  | Português Brasileiro |
| `zh-HK` | Kantonská čínština             | 粵語             |
| `ms`    | Malajština                 | Bahasa Melayu    |
| `sk`    | Slovákština                | Slovenčina       |
| `bg`    | Bulharština             | Български        |
| `hr`    | Chorvatština              | Hrvatski         |
| `lt`    | Litevština            | Lietuvių         |
| `lv`    | Lotyština               | Latviešu         |
| `et`    | Estonština              | Eesti            |
| `sl`    | Slovinština             | Slovenščina      |

{% hint style="success" %}
**Automatické uložení**: Vaše jazykové nastavení se uloží do souboru `~/.chloros/cli_language.json` a zůstane zachováno po celou dobu všech relací.
{% endhint %}

***

### `set-project-folder` - Nastavení výchozí složky projektu

Změňte umístění výchozí složky projektu (sdílené s GUI v Windows).

**Syntaxe:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Příklady:**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` – Zobrazit složku projektu

Zobrazí aktuální umístění výchozí složky projektu.

**Syntaxe:**

```bash
chloros-cli get-project-folder
```

**Příklad:**

```bash
chloros-cli get-project-folder
```

**Výstup:**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` – Obnovit výchozí nastavení

Obnoví výchozí umístění složky projektu.

**Syntaxe:**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` – Spustit diagnostiku systému

Spustí 7 diagnostických kontrol k ověření konfigurace systému.

**Syntaxe:**

```bash
chloros-cli selftest
```

**Provedené diagnostické testy:**

1. Kontrola verze
2. Dostupnost portu (5000)
3. Spuštění backendu
4. Test připojení API
5. Informace o systému a detekce GPU
6. Ověření modelů odšumovače
7. Kontrola dostupnosti CUDA

{% hint style="info" %}
**Užitečné pro řešení problémů**: Po instalaci spusťte `selftest`, abyste ověřili, zda je váš systém správně nakonfigurován, zejména na Linux/Jetson, kde může být nutné ověřit nastavení GPU a CUDA.
{% endhint %}

***

### `update` – Kontrola aktualizací (pouze Linux)

Zkontrolujte a nainstalujte aktualizace CLI na systémech Linux.

**Syntaxe:**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| Možnost    | Popis                        |
| --------- | ---------------------------------- |
| `--check` | Pouze vyhledat aktualizace, neinstalovat |

{% hint style="info" %}
Tento příkaz je k dispozici pouze na Linux. Na Windows jsou aktualizace dodávány prostřednictvím instalačního programu.
{% endhint %}

***

## Globální možnosti

Tyto možnosti platí pro všechny příkazy:

| Možnost            | Typ    | Výchozí       | Popis                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | Cesta    | Automaticky detekováno | Cesta ke spustitelnému souboru backendu                       |
| `--port`          | Celé číslo | 5000          | Číslo portu backendu API                          |
| `--restart`       | Příznak    | -             | Vynutit restart backendu (ukončí stávající procesy) |
| `--version`       | Příznak    | -             | Zobrazit informace o verzi a ukončit                |
| `--help`          | Příznak    | -             | Zobrazit nápovědu a ukončit                   |

{% hint style="info" %}
**Automatická detekce backendu**: Cesta `--backend-exe` je automaticky detekována podle platformy:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (ručně)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**Příklad s globálními možnostmi:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux:**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## Průvodce nastavením zpracování

### Paralelní zpracování a dynamická adaptace výpočtů

Chloros 1.1.0 obsahuje [dynamickou adaptaci výpočtů](processing-architecture/dynamic-compute-adaptation.md) — zpracovatelský engine **automaticky detekuje váš hardware** a vybere optimální strategii:

| Platforma | Strategie | Pracovníci | Pipeline | Poznámky |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` | Úsporné na paměť, serializované |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` | Souběžné zpracování na GPU |
| **Stolní počítač s 8 GB GPU** | `GPU_SINGLE` | 3 | `tiled_gpu` | Dobrý výkon stolního počítače |
| **Stolní počítač s GPU 12 GB a více** | `GPU_PARALLEL` | 3–4 | `fused_gpu` | Optimální výkon stolního počítače |
| **Systém pouze s CPU** | `CPU_PARALLEL` | jádra – 1 | `cpu_fallback` | Není vyžadována GPU |

{% hint style="success" %}
**Není nutná žádná ruční konfigurace!** Chloros automaticky detekuje váš procesor, grafickou kartu, paměť RAM a (na Jetsonu) teplotní senzory a poté automaticky nakonfiguruje optimální zpracovatelský pipeline.
{% endhint %}

### Metody debayeringu

| Metoda | CLI Příznak | Kvalita | Rychlost | Licence |
| --- | --- | --- | --- | --- |
| **Standardní (rychlé, střední kvalita)** | `--debayer standard` | Dobrá | Rychlé | Zdarma / Chloros+ |
| **S ohledem na texturu (pomalá, nejvyšší kvalita)** | `--debayer texture-aware` | Nejvyšší | Pomalá | Pouze Chloros+ |

Výchozí metodou debayeringu je **Standard**. Metoda**Texture Aware** využívá model odšumování AI/ML pro výstup v nejvyšší kvalitě, ale vyžaduje licenci Chloros+ a grafickou kartu NVIDIA.

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### Korekce vinětace

**Co dělá:** Koriguje pokles intenzity světla na okrajích obrazu (tmavší rohy, které jsou běžné u snímků z fotoaparátů).

* **Ve výchozím nastavení zapnuto** – Většina uživatelů by měla tuto funkci nechat zapnutou
* K vypnutí použijte `--no-vignette`

{% hint style="success" %}
**Doporučení**: Vždy zapněte korekci vinětace, abyste zajistili rovnoměrný jas v celém snímku.
{% endhint %}

### Kalibrace odrazivosti

Převádí surové hodnoty snímače na standardizované procentuální hodnoty odrazivosti pomocí kalibračních panelů.

* **Ve výchozím nastavení zapnuto** – Nezbytné pro analýzu vegetace
* Vyžaduje kalibrační panely v obrazech
* K vypnutí použijte `--no-reflectance`

{% hint style="info" %}
**Požadavky**: Pro přesný převod odrazivosti zajistěte, aby byly kalibrační panely ve vašich snímcích správně exponované a viditelné.
{% endhint %}

### Korekce PPK

**Co dělá:** Používá korekce Post-Processed Kinematic (PPK) s využitím dat protokolu DAQ-A-SD pro zvýšení přesnosti GPS.

* **Ve výchozím nastavení je vypnuto**
* K zapnutí použijte `--ppk`
* Vyžaduje soubory .daq ve složce projektu ze světelného senzoru DAQ-A-SD MAPIR.

### Výstupní formáty

<table><thead><tr><th width="197">Formát</th><th width="130.20001220703125">Bitová hloubka</th><th width="116.5999755859375">Velikost souboru</th><th>Nejvhodnější pro</th></tr></thead><tbody><tr><td><strong>TIFF (16bitový)</strong> ⭐</td><td>16bitové celé číslo</td><td>Velká</td><td>GIS analýza, fotogrammetrie (doporučeno)</td></tr><tr><td><strong>TIFF (32bitové, procenta)</strong></td><td>32bitové číslo s plovoucí desetinnou čárkou</td><td>Velmi velké</td><td>Vědecká analýza, výzkum</td></tr><tr><td><strong>PNG (8bitový)</strong></td><td>8bitové celé číslo</td><td>Střední</td><td>Vizuální kontrola, sdílení na webu</td></tr><tr><td><strong>JPG (8bitový)</strong></td><td>8bitové celé číslo</td><td>Malý</td><td>Rychlý náhled, komprimovaný výstup</td></tr></tbody></table>***

## Automatizace a skriptování

### Dávkové zpracování v PowerShellu (Windows)

Automatické zpracování více složek s datovými sadami v Windows:

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Dávkový skript Windows (Windows)

Jednoduchá smyčka pro dávkové zpracování na Windows:

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Dávkové zpracování v Bash (Linux)

Zpracování více složek s datovými sadami na Linux:

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### Automatizační skript Python (multiplatformní)

Pokročilá automatizace s řešením chyb (funguje na Windows a Linux):

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## Pracovní postup zpracování

### Standardní pracovní postup

1. **Vstup**: Složka obsahující páry obrázků ve formátu RAW/JPG
2. **Vyhledání**: CLI automaticky vyhledá podporované obrazové soubory
3. **Zpracování**: Paralelní režim se přizpůsobí počtu jader vašeho procesoru (Chloros+)
4. **Výstup**: Vytvoří podsložky podle modelů fotoaparátů se zpracovanými obrázky

### Příklad struktury výstupu

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### Odhady doby zpracování

Typické doby zpracování pro 100 obrázků (každý 12 MP):

| Platforma | Režim | Odhadovaná doba | Poznámky |
| --- | --- | --- | --- |
| **Stolní počítač s GPU 12 GB+** | `GPU_PARALLEL` | 5–10 min | Nejrychlejší možnost |
| **Stolní počítač s GPU 8 GB** | `GPU_SINGLE` | 10–15 min | Dobrý výkon |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 15–25 min | Edge computing |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 30–60 min | Omezená paměť |
| **Pouze CPU** | `CPU_PARALLEL` | 20–40 min | Není vyžadován GPU |

{% hint style="info" %}
**Tip k výkonu**: Doba zpracování se liší v závislosti na počtu snímků, rozlišení, metodě debayeringu a hardwaru. Metoda Texture Aware debayer trvá výrazně déle než Standard. Podrobnosti najdete v [Dynamická adaptace výpočtů](processing-architecture/dynamic-compute-adaptation.md).
{% endhint %}

***

## Řešení problémů

### CLI nenalezeno

**Chyba Windows:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Windows Řešení:**

1. Ověřte umístění instalace:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Pokud není v PATH, použijte úplnou cestu:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Ručně přidejte do proměnné PATH:
   * Otevřete Vlastnosti systému → Proměnné prostředí
   * Upravte proměnnou PATH
   * Přidejte: `C:\Program Files\Chloros\resources\cli`
   * Restartujte terminál

**Chyba Linux:**

```
chloros-cli: command not found
```

**Linux Řešení:**

1. Ověřte instalaci:

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. Obnovte shell:

```bash
source ~/.bashrc
```

3. Zkontrolujte oprávnění:

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### Spuštění backendu se nezdařilo**Chyba:**

```

Backend failed to start within 30 seconds
```

**Řešení:**

1. Zkontrolujte, zda backend již neběží (nejprve jej zavřete)
2. Zkontrolujte, zda ho neblokuje firewall (Windows) nebo zkontrolujte dostupnost portu (Linux: `lsof -i :5000`)
3. Zkuste jiný port:

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. Vynuťte restart backendu:

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. V případě chyby Linux zkontrolujte, zda existuje spustitelný soubor backendu:

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### Problémy s licencí / ověřením**Chyba:**

```

Chloros+ license required for CLI access
```

**Řešení:**

1. Ověřte, zda máte aktivní předplatné Chloros+
2. Přihlaste se pomocí svých přihlašovacích údajů:

```bash
chloros-cli login user@example.com 'password'
```

3. Zkontrolujte stav licence:

```bash
chloros-cli status
```

4. Kontaktujte podporu: info@mapir.camera

***

### Nebyly nalezeny žádné obrázky**Chyba:**

```

No images found in the specified folder
```

**Řešení:**

1. Ověřte, zda složka obsahuje podporované formáty (.RAW, .TIF, .JPG)
2. Zkontrolujte, zda je cesta ke složce správná (u cest s mezerami použijte uvozovky)
3. Ujistěte se, že máte oprávnění ke čtení této složky
4. Zkontrolujte, zda jsou příponami souborů správné

***

### Zpracování se zastaví nebo zamrzne**Řešení:**

1. Zkontrolujte dostupné místo na disku (ujistěte se, že je dostatek místa pro výstup)
2. Zavřete ostatní aplikace, abyste uvolnili paměť
3. Snižte počet obrázků (zpracovávejte v dávkách)

***

### Port je již používán**Chyba:**

```

Port 5000 is already in use
```

**Řešení:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux:**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## Často kladené otázky

### Otázka: Potřebuji licenci pro CLI?

**Odpověď:**Ano! CLI vyžaduje placenou**licenci Chloros+**.

* ❌ Standardní (bezplatný) tarif: CLI je deaktivován
* ✅ Tarify Chloros+ (placené): CLI je plně aktivován

Přihlaste se k odběru na: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### Otázka: Mohu používat CLI na serveru bez grafického uživatelského rozhraní?**Odpověď:** Ano! CLI běží zcela bez grafického rozhraní. To je hlavní způsob použití na Linux.**Windows Server:**
* Windows Server 2016 nebo novější
* Nainstalovaný balíček Visual C++ Redistributable

**Linux Server:**
* Ubuntu 20.04+ / Debian 11+ (amd64) nebo JetPack 6 (arm64)
* Instalace prostřednictvím balíčku `.deb`

**Obě platformy:**
* Minimálně 8 GB RAM (doporučeno 16 GB)
* Jednorázová aktivace licence: `chloros-cli login user@example.com 'password'`

***

### Otázka: Kam se ukládají zpracované snímky?**Odpověď:**Ve výchozím nastavení se zpracované snímky ukládají do**stejné složky jako vstupní** do podsložek podle modelu fotoaparátu (např. `Survey3N_RGN/`).

Pro určení jiné výstupní složky použijte volbu `-o`:

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### Otázka: Mohu zpracovat více složek najednou?**A:** Ne přímo jedním příkazem, ale můžete použít skriptování k postupnému zpracování složek. Viz část [Automatizace a skriptování](CLI.md#automation--scripting).***

### Q: Jak uložím výstup CLI do souboru protokolu?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash:**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### Otázka: Co se stane, když během zpracování stisknu klávesy Ctrl+C?**Odpověď:** CLI:

1. Úhledně zastaví zpracování
2. Vypne backend
3. Ukončí se s kódem 130

Částečně zpracované obrázky mohou zůstat ve výstupní složce.

***

### Otázka: Mohu automatizovat zpracování CLI?**Odpověď:** Samozřejmě! CLI je navržen pro automatizaci. Viz [Automatizace a skriptování](CLI.md#automation--scripting) pro PowerShell (Windows), Batch (Windows), Bash (Linux) a Python (multiplatformní).***

### Otázka: Jak zkontroluji verzi CLI?**Odpověď:**

```bash
chloros-cli --version
```

**Výstup:**

```

Chloros CLI 1.1.0
```

***

## Získání nápovědy

### Nápověda k příkazovému řádku

Zobrazte informace o nápovědě přímo v CLI:

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Kanály podpory

* **E-mail**: info@mapir.camera
* **Webové stránky**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Ceny**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Kompletní příklady

### Příklad 1: Základní zpracování

Zpracování s výchozím nastavením (vinětace, odrazivost):

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### Příklad 2: Vysoce kvalitní vědecký výstup

32bitová plovoucí desetinná čárka TIFF:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### Příklad 3: Rychlé zpracování náhledu

8bitové PNG bez kalibrace pro rychlou kontrolu:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### Příklad 4: Zpracování s korekcí PPK

Použijte korekce PPK s odrazivostí:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### Příklad 5: Vlastní umístění výstupu

Zpracujte do jiného umístění s konkrétním formátem:

**Windows:**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux:**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### Příklad 6: Pracovní postup ověřování

Kompletní postup ověřování (stejný na všech platformách):

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Příklad 7: Použití více jazyků

Změna jazyka rozhraní (stejná na všech platformách):

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```
