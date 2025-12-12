# CLI : Příkazový řádek

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** poskytuje výkonný přístup z příkazového řádku k modulu pro zpracování obrazu Chloros, což umožňuje automatizaci, skriptování a bezhlavý provoz vašich pracovních postupů v oblasti zpracování obrazu.

### Klíčové vlastnosti

* 🚀 **Automatizace** – dávkové zpracování více datových sad pomocí skriptů
* 🔗 **Integrace** – začlenění do stávajících pracovních postupů a procesů
* 💻 **Bezhlavý provoz** – spuštění bez grafického uživatelského rozhraní
* 🌍 **Vícejazyčnost** – podpora 38 jazyků
* ⚡ **Paralelní zpracování** – dynamické škálování podle výkonu vašeho procesoru (až 16 paralelních procesů)

### Požadavky

| Požadavek          | Podrobnosti                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Operační systém** | Windows 10/11 (64bitový)                                              |
| **Licence**          | Chloros+ ([vyžaduje placený tarif](https://cloud.mapir.camera/pricing)) |
| **Paměť**           | Minimálně 8 GB RAM (doporučeno 16 GB)                                  |
| **Internet**         | Vyžadován pro aktivaci licence                                     |
| **Místo na disku**       | Liší se podle velikosti projektu                                              |

{% hint style=&quot;warning&quot; %}
**Požadavky na licenci**: CLI vyžaduje placené předplatné Chloros+. Standardní (bezplatné) plány nemají přístup k CLI. Chcete-li provést upgrade, navštivte [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).
{% endhint %}

## Rychlý start

### Instalace

CLI je automaticky součástí instalačního programu Chloros:

1. Stáhněte a spusťte **Chloros Installer.exe**
2. Dokončete instalačního průvodce
3. CLI nainstalován do: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style=&quot;success&quot; %}
Instalační program automaticky přidá `chloros-cli` do systémové cesty PATH. Po instalaci restartujte terminál.
{% endhint %}

### První nastavení

Před použitím CLI aktivujte licenci Chloros+:

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### Základní použití

Zpracujte složku s výchozím nastavením:

```powershell
chloros-cli process "C:\Images\Dataset001"
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

Zpracování obrázků ve složce s kalibrací.

**Syntaxe:**

```bash
chloros-cli process <input-folder> [options]
```

**Příklad:**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### Možnosti příkazu zpracování

| Možnost                | Typ    | Výchozí        | Popis                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Cesta    | _Povinné_     | Složka obsahující multispektrální obrázky RAW/JPG                                         |
| `-o, --output`        | Cesta    | Stejná jako vstup  | Výstupní složka pro zpracované obrázky                                                     |
| `-n, --project-name`  | Řetězec  | Automaticky generovaný | Vlastní název projektu                                                                    |
| `--vignette`          | Příznak    | Povoleno        | Povolit korekci viněty                                                             |
| `--no-vignette`       | Příznak    | -              | Zakázat korekci viněty                                                            |
| `--reflectance`       | Příznak    | Povoleno        | Povolit kalibraci odrazivosti                                                         |
| `--no-reflectance`    | Příznak    | -              | Zakázat kalibraci odrazivosti                                                        |
| `--ppk`               | Příznak    | Zakázáno       | Použít korekce PPK z dat světelného senzoru .daq                                      |
| `--format`            | Volba  | TIFF (16bitové)  | Výstupní formát: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Celé číslo | Automaticky           | Minimální cílová velikost v pixelech pro detekci kalibračního panelu                          |
| `--target-clustering` | Celé číslo | Automaticky           | Prahová hodnota seskupení cílů (0–100)                                                    |
| `--exposure-pin-1`    | Řetězec  | Žádný           | Zamknout expozici pro model kamery (pin 1)                                                 |
| `--exposure-pin-2`    | Řetězec  | Žádný           | Zamknout expozici pro model kamery (pin 2)                                                 |
| `--recal-interval`    | Celé číslo | Automaticky           | Interval rekalibrace v sekundách                                                      |
| `--timezone-offset`   | Celé číslo | 0              | Časový posun v hodinách                                                               |

***

### `login` – Ověření účtu

Přihlaste se pomocí svých přihlašovacích údajů Chloros+, abyste povolili zpracování CLI.

**Syntaxe:**

```bash
chloros-cli login <email> <password>
```

**Příklad:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Zvláštní znaky**: Použijte jednoduché uvozovky kolem hesel obsahujících znaky jako `$`, `!` nebo mezery.
{% endhint %}

**Výstup:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` – Vymazání přihlašovacích údajů

Vymazání uložených přihlašovacích údajů a odhlášení z účtu.

**Syntaxe:**

```bash
chloros-cli logout
```

**Příklad:**

```powershell
chloros-cli logout
```

**Výstup:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

***

### `status` – Zkontrolovat stav licence

Zobrazí aktuální stav licence a ověření.

**Syntaxe:**

```bash
chloros-cli status
```

**Příklad:**

```powershell
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

### `export-status` – Kontrola průběhu exportu

Sleduje průběh exportu vlákna 4 během zpracování nebo po něm.

**Syntaxe:**

```bash
chloros-cli export-status
```

**Příklad:**

```powershell
chloros-cli export-status
```

**Případ použití:** Tento příkaz vyvolejte během zpracování, abyste zkontrolovali průběh exportu.

***

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

```powershell
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
| `nl`    | Holandština                 | Nederlands       |
| `ar`    | Arabština                | العربية          |
| `pl`    | Polština                | Polski           |
| `tr`    | Turečtina               | Türkçe           |
| `hi`    | Hindština                 | हिंदी            |
| `id`    | Indonéština            | Bahasa Indonesia |
| `vi`    | Vietnamština            | Tiếng Việt       |
| `th`    | Thajština                  | ไทย              |
| `sv`    | Švédština               | Svenska          |
| `da`    | Dánština                | Dansk            |
| `no`    | Norština             | Norsk            |
| `fi`    | Finský               | Suomi            |
| `el`    | Řecký                 | Ελληνικά         |
| `cs`    | Český                 | Čeština          |
| `hu`    | Maďarština             | Magyar           |
| `ro`    | Rumunština              | Română           |
| `uk`    | Ukrajinština             | Українська       |
| `pt-BR` | Brazilská portugalština  | Português Brasileiro |
| `zh-HK` | Kantonská čínština | 粵語             |
| `ms`    | Malajština                 | Bahasa Melayu    |
| `sk`    | Slovenština                | Slovenčina       |
| `bg`    | Bulharština             | Български        |
| `hr`    | Chorvatština              | Hrvatski         |
| `lt`    | Litevština            | Lietuvių         |
| `lv`    | Lotyština               | Latviešu         |
| `et`    | Estonština              | Eesti            |
| `sl`    | Slovinština             | Slovenščina      |

{% hint style=&quot;success&quot; %}
**Automatické uchování**: Vaše jazykové preference jsou uloženy v `~/.chloros/cli_language.json` a zůstávají zachovány po celou dobu trvání všech relací.
{% endhint %}

***

### `set-project-folder` – Nastavení výchozí složky projektu

Změňte umístění výchozí složky projektu (sdílené s GUI).

**Syntaxe:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Příklad:**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder` – Zobrazit složku projektu

Zobrazí aktuální umístění výchozí složky projektu.

**Syntaxe:**

```bash
chloros-cli get-project-folder
```

**Příklad:**

```powershell
chloros-cli get-project-folder
```

**Výstup:**

```
ℹ Current project folder: C:\Projects\2025
```

***

### `reset-project-folder` – Obnovit výchozí nastavení

Obnoví výchozí umístění složky projektu.

**Syntaxe:**

```bash
chloros-cli reset-project-folder
```

***

## Globální možnosti

Tyto možnosti se vztahují na všechny příkazy:

| Možnost          | Typ    | Výchozí nastavení       | Popis                                      |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | Cesta    | Automaticky detekováno | Cesta k spustitelnému souboru backendu                       |
| `--port`        | Celé číslo | 5000          | Číslo portu backendu API                          |
| `--restart`     | Příznak    | -             | Vynutit restart backendu (ukončí existující procesy) |
| `--version`     | Příznak    | -             | Zobrazit informace o verzi a ukončit                |
| `--help`        | Příznak    | -             | Zobrazit nápovědu a ukončit                   |

**Příklad s globálními možnostmi:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## Průvodce nastavením zpracování

### Paralelní zpracování

Chloros+ CLI **automaticky škálovat** paralelní zpracování tak, aby odpovídalo schopnostem vašeho počítače:

**Jak to funguje:**

* Detekuje jádra CPU a RAM
* Přiděluje pracovníky: **2× jádra CPU** (používá hyperthreading)
* **Maximálně: 16 paralelních pracovníků** (pro stabilitu)

**Úrovně systému:**

| Typ systému   | CPU        | RAM      | Pracovníci  | Výkon     |
| ------------- | ---------- | -------- | -------- | --------------- |
| **High-End**  | 16+ jader  | 32+ GB   | Až 16 | Maximální rychlost   |
| **Střední třída** | 8–15 jader | 16–31 GB | 8–16     | Vynikající rychlost |
| **Nízká třída**   | 4–7 jader  | 8–15 GB  | 4–8      | Dobrá rychlost      |

{% hint style=&quot;success&quot; %}
**Automatická optimalizace**: CLI automaticky detekuje specifikace vašeho systému a nakonfiguruje optimální paralelní zpracování. Není potřeba žádná ruční konfigurace!
{% endhint %}

### Metody debayerování

CLI používá jako výchozí a doporučený algoritmus debayerování **Vysoká kvalita (rychlejší)**:

| Metoda                      | Kvalita | Rychlost | Popis                                 |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **Vysoká kvalita (rychlejší)** ⭐ | ⭐⭐⭐⭐    | ⚡⚡⚡   | Algoritmus s ohledem na okraje (výchozí, doporučené) |

### Korekce vinětace

**Co dělá:** Koriguje pokles jasu na okrajích obrazu (tmavší rohy běžné u snímků z fotoaparátu).

* **Výchozí nastavení** – Většina uživatelů by měla tuto funkci ponechat zapnutou.
* K vypnutí použijte `--no-vignette`.

{% hint style=&quot;success&quot; %}
**Doporučení**: Vždy povolte korekci vinětace, abyste zajistili rovnoměrný jas v celém snímku.
{% endhint %}

### Kalibrace odrazivosti

Převádí surové hodnoty senzoru na standardizovaná procenta odrazivosti pomocí kalibračních panelů.

* **Výchozí nastavení** – nezbytné pro analýzu vegetace.
* Vyžaduje kalibrační panely v obrazech.
* K deaktivaci použijte `--no-reflectance`.

{% hint style=&quot;info&quot; %}
**Požadavky**: Pro přesný převod odrazivosti se ujistěte, že kalibrační panely jsou ve vašich snímcích správně exponované a viditelné.
{% endhint %}

### PPK korekce

**Co dělá:** Používá postprocesní kinematické korekce pomocí dat protokolu DAQ-A-SD pro zlepšení přesnosti GPS.

* **Ve výchozím nastavení je tato funkce vypnuta.**
* K zapnutí použijte `--ppk`.
* Vyžaduje soubory .daq v projektové složce z MAPIR DAQ-A-SD světelného senzoru.

### Výstupní formáty

<table><thead><tr><th width="197">Formát</th><th width="130.20001220703125">Bitová hloubka</th><th width="116.5999755859375">Velikost souboru</th><th>Nejvhodnější pro</th></tr></thead><tbody><tr><td><strong>TIFF (16 bitů)</strong> ⭐</td><td>16bitové celé číslo</td><td>Velké</td><td>GIS analýza, fotogrammetrie (doporučeno)</td></tr><tr><td><strong>TIFF (32bitové, procenta)</strong></td><td>32bitové číslo s plovoucí desetinnou čárkou</td><td>Velmi velké</td><td>Vědecká analýza, výzkum</td></tr><tr><td><strong>PNG (8bitový)</strong></td><td>8bitové celé číslo</td><td>Střední</td><td>Vizuální kontrola, sdílení na webu</td></tr><tr><td><strong>JPG (8bitové)</strong></td><td>8bitové celé číslo</td><td>Malý</td><td>Rychlý náhled, komprimovaný výstup</td></tr></tbody></table>***

## Automatizace a skriptování

### Hromadné zpracování v PowerShellu

Automatické zpracování více složek datových sad:

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

### Windows Hromadný skript

Jednoduchá smyčka pro hromadné zpracování:

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

### Python Automatizační skript

Pokročilá automatizace s řešením chyb:

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

1. **Vstup**: Složka obsahující páry obrázků RAW/JPG
2. **Objevování**: CLI automaticky vyhledává podporované obrazové soubory
3. **Zpracování**: Paralelní režim se přizpůsobuje jádrům vašeho procesoru (Chloros+)
4. **Výstup**: Vytváří podsložky podle modelů fotoaparátů se zpracovanými obrázky

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

### Odhadované doby zpracování

Typické doby zpracování pro 100 obrázků (každý o velikosti 12 MP):

| Režim              | Doba      | Hardware                                     |
| ----------------- | --------- | -------------------------------------------- |
| **Paralelní režim** | 5–10 min  | i7/Ryzen 7, 16 GB RAM, SSD (až 16 pracovníků) |
| **Paralelní režim** | 10–15 min | i5/Ryzen 5, 8 GB RAM, HDD (až 8 pracovníků)   |

{% hint style=&quot;info&quot; %}
**Tip pro zvýšení výkonu**: Doba zpracování se liší v závislosti na počtu obrázků, rozlišení a specifikacích počítače.
{% endhint %}

***

## Řešení problémů

### CLI nenalezeno

**Chyba:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Řešení:**

1. Ověřte umístění instalace:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Pokud není v PATH, použijte úplnou cestu:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Ručně přidejte do PATH:
   * Otevřete Vlastnosti systému → Proměnné prostředí.
   * Upravte proměnnou PATH.
   * Přidejte: `C:\Program Files\Chloros\resources\cli`
   * Restartujte terminál.

***

### Selhání spuštění backendu

**Chyba:**

```
Backend failed to start within 30 seconds
```

**Řešení:**

1. Zkontrolujte, zda backend již běží (nejprve jej zavřete).
2. Zkontrolujte, zda Windows Firewall neblokuje.
3. Zkuste jiný port:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. Vynuťte restart backendu:

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### Problémy s licencí / ověřením

**Chyba:**

```
Chloros+ license required for CLI access
```

**Řešení:**

1. Ověřte, zda máte aktivní předplatné Chloros+.
2. Přihlaste se pomocí svých přihlašovacích údajů:

```powershell
chloros-cli login user@example.com 'password'
```

3. Zkontrolujte stav licence:

```powershell
chloros-cli status
```

4. Kontaktujte podporu: info@mapir.camera

***

### Nebyly nalezeny žádné obrázky

**Chyba:**

```
No images found in the specified folder
```

**Řešení:**

1. Ověřte, zda složka obsahuje podporované formáty (.RAW, .TIF, .JPG).
2. Zkontrolujte, zda je cesta ke složce správná (pro cesty s mezerami použijte uvozovky).
3. Ujistěte se, že máte oprávnění ke čtení pro složku.
4. Zkontrolujte, zda jsou přípony souborů správné.

***

### Zpracování se zastaví nebo zamrzne

**Řešení:**

1. Zkontrolujte dostupný prostor na disku (ujistěte se, že je dostatek místa pro výstup).
2. Zavřete ostatní aplikace, abyste uvolnili paměť.
3. Snižte počet obrázků (zpracovávejte je po dávkách).

***

### Port je již používán

**Chyba:**

```
Port 5000 is already in use
```

**Řešení:**

Zadejte jiný port:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## Často kladené otázky

### Otázka: Potřebuji licenci pro CLI?

**Odpověď:** Ano! CLI vyžaduje placenou **licenci Chloros+**.

* ❌ Standardní (bezplatný) plán: CLI deaktivován
* ✅ Plány Chloros+ (placené): CLI plně povolen

Přihlaste se na: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### Otázka: Mohu používat CLI na serveru bez grafického uživatelského rozhraní?

**Odpověď:** Ano! CLI běží zcela bez grafického rozhraní. Požadavky:

* Windows Server 2016 nebo novější
* Nainstalovaný balíček Visual C++ Redistributable
* Dostatečná paměť RAM (minimálně 8 GB, doporučeno 16 GB)
* Jednorázová aktivace licence GUI na libovolném počítači

***

### Otázka: Kam se ukládají zpracované obrázky?

**Odpověď:** Ve výchozím nastavení se zpracované obrázky ukládají do **stejné složky jako vstupní data** v podsložkách podle modelu kamery (např. `Survey3N_RGN/`).

Pomocí možnosti `-o` můžete určit jinou výstupní složku:

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### Otázka: Mohu zpracovat více složek najednou?

**A:** Ne přímo v jednom příkazu, ale můžete použít skriptování k postupnému zpracování složek. Viz část [Automatizace a skriptování](CLI.md#automation--scripting).

***

### Q: Jak uložím výstup CLI do souboru protokolu?

**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

***

### Otázka: Co se stane, když během zpracování stisknu klávesy Ctrl+C?

**Odpověď:** CLI:

1. Ukončí zpracování
2. Vypne backend
3. Ukončí se s kódem 130

Částečně zpracované obrázky mohou zůstat ve výstupní složce.

***

### Otázka: Mohu automatizovat zpracování CLI?

**Odpověď:** Samozřejmě! CLI je navržen pro automatizaci. Příklady pro PowerShell, Batch a Python najdete v části [Automatizace a skriptování](CLI.md#automation--scripting).

***

### Otázka: Jak zkontroluji verzi CLI?

**Odpověď:**

```powershell
chloros-cli --version
```

**Výstup:**

```
Chloros CLI 1.0.2
```

***

## Nápověda

### Nápověda příkazového řádku

Nápovědu si můžete zobrazit přímo v CLI:

```powershell
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
* **Ceny**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

## Kompletní příklady

### Příklad 1: Základní zpracování

Zpracování s výchozím nastavením (viněta, odrazivost):

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### Příklad 2: Vysoce kvalitní vědecký výstup

32bitové plovoucí TIFF:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### Příklad 3: Rychlé zpracování náhledu

8bitové PNG bez kalibrace pro rychlou kontrolu:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### Příklad 4: Zpracování s korekcí PPK

Aplikujte korekce PPK s odrazivostí:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### Příklad 5: Vlastní umístění výstupu

Zpracujte na jiný disk s konkrétním formátem:

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### Příklad 6: Pracovní postup ověřování

Dokončete pracovní postup ověřování:

```powershell
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
chloros-cli process "C:\Datasets\Field_A"

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Příklad 7: Vícejazyčné použití

Změňte jazyk rozhraní:

```powershell
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
chloros-cli process "C:\Vuelos\Campo_A"

# Change back to English
chloros-cli language en
```
