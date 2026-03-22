# Zpracovací pipeline

Chloros 1.1.0 využívá čtyřvláknovou zpracovací pipeline, která funguje jako postupná montážní linka. Každé vlákno zpracovává samostatnou fázi pracovního postupu, což umožňuje souběžné zpracování více obrázků v různých fázích.

***

## Architektura pipeline

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Každý obrázek prochází postupně všemi čtyřmi vlákny. Díky vícevláknové zpracování v Chloros+ může být více obrázků současně v různých vláknech — zatímco vlákno 3 zpracovává jeden obrázek, vlákno 1 může detekovat další, vlákno 2 může kalibrovat další a vlákno 4 může zapisovat dříve zpracovaný obrázek na disk.

***

## Podrobnosti o vláknech

### Vlákno 1: Detekce

**Účel**: Načíst obrázky a detekovat kalibrační cíle.

* Čte obrazové soubory z disku (RAW, JPG)
* Extrahuje metadata EXIF (GPS, model fotoaparátu, časová razítka, expozice)
* Detekuje kalibrační cíle ArUco v označených cílových snímcích
* Výstupy: obrazová data + metadata + výsledky detekce cílů

Jedná se primárně o vlákno vázané na I/O a CPU.

### Vlákno 2: Kalibrace

**Účel**: Vypočítat kalibrační parametry z detekovaných cílů.

* Vypočítává kalibrační koeficienty odrazivosti z obrázků cílů
* Vypočítává parametry korekce vinětace
* Určuje kalibrační křivky pro jednotlivá pásma
* Výstupy: kalibrační parametry pro každý obrázek

Jedná se o výpočetní vlákno závislé na CPU.

### Vlákno 3: Zpracování (GPU)

**Účel**: Aplikovat korekce a vypočítat vegetační indexy.**Toto je výpočetně nejnáročnější vlákno.*** **Debayering**: Převádí data ve formátu RAW s Bayerovým vzorem na vícekanálové snímky
  * Standardní (rychlé, střední kvalita) — výchozí
  * S ohledem na texturu (pomalé, nejvyšší kvalita) — pouze Chloros+, využívá odšumování pomocí AI/ML
* **Korekce vinětace**: Aplikuje korekci vinětace objektivu na celý snímek
* **Kalibrace odrazivosti**: Použije kalibrační koeficienty k převodu na hodnoty odrazivosti
* **Výpočet indexů**: Vypočítá vegetační indexy (NDVI, NDRE, GNDVI atd.)
* Výstupy: zpracovaná obrazová data připravená k exportu

Toto vlákno nejvíce těží z akcelerace GPU. Systém [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) primárně optimalizuje chování tohoto vlákna.

### Vlákno 4: Export

**Účel**: Zápis zpracovaných obrázků na disk.

* Zapisuje výstupní soubory ve vybraném formátu (TIFF 16-bit, TIFF 32-bit %, PNG, JPG)
* Vkládá metadata EXIF do výstupních souborů (GPS, časová razítka, parametry zpracování)
* Uspořádává výstup do podsložek podle modelů fotoaparátů
* Výstupy: finální soubory na disku

Jedná se primárně o vlákno vázané na I/O. SSD úložiště výrazně zlepšuje výkon vlákna 4.

***

## Sekvenční vs. pipelined zpracování

### Volný režim (sekvenční)

Ve bezplatné verzi Chloros jsou obrázky zpracovávány **po jednom**, sekvenčně ve všech čtyřech fázích:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

Průběhový pruh v grafickém rozhraní zobrazuje 2 fáze: Detekce cíle a Zpracování.

### Režim Chloros+ (pipelined)

S licencí Chloros+ pracují všechna čtyři vlákna **souběžně** na různých snímcích:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

Indikátor průběhu v grafickém uživatelském rozhraní zobrazuje 4 fáze: Detekce, Analýza, Kalibrace, Export. Umístěte kurzor myši nad indikátor průběhu, abyste viděli průběh jednotlivých vláken.

{% hint style="success" %}
**Pipelined processing s Chloros+** může být 3–5x rychlejší než sekvenční zpracování, v závislosti na vašem hardwaru a velikosti datového souboru. Největší zrychlení je na systémech s rychlými GPU a SSD.
{% endhint %}

***

## Průběh exportu vlákna 4

Ve verzi Chloros 1.1.0 má exportní vlákno (vlákno 4) vlastní vyhrazené sledování průběhu. Průběh exportu můžete sledovat samostatně:**CLI:**
```bash
chloros-cli export-status
```

**SDK:**
```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Zpracování je dokončeno, jakmile vlákno 4 dosáhne 100 %.

***

## Vztah k dynamické adaptaci výpočtů

Systém [dynamické adaptace výpočtů](dynamic-compute-adaptation.md) ovlivňuje především **vlákno 3 (zpracování)**:

* **`GPU_PARALLEL`** strategie: vlákno 3 zpracovává více obrazů současně na GPU pomocí potrubí `fused_gpu`
* Strategie **`GPU_SINGLE`**: Vlákno 3 zpracovává jeden obrázek po druhém pomocí paměťově efektivního potrubí `tiled_gpu`
* Strategie **`CPU_PARALLEL`**: Vlákno 3 využívá zpracování založené na CPU s vícevláknovou paralelizací

Alokace paměti GPU vlákna 3 se také dynamicky mění s dokončováním vláken 1 a 2 — viz [Dynamická alokace paměti GPU](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Další kroky

* [Dynamická adaptace výpočtů](dynamic-compute-adaptation.md) — Jak Chloros vybírá optimální strategii pro váš hardware
* [Průvodce NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Chování pipeline specifické pro platformu na Jetsonu
* [Sledování zpracování](../processing-images-gui/monitoring-the-processing.md) — Sledování průběhu pomocí grafického uživatelského rozhraní
