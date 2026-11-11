# Zpracovací pipeline

Chloros 1.3.0 využívá čtyřvláknovou zpracovací pipeline, která funguje jako stupňovitá montážní linka. Každé vlákno zpracovává samostatnou fázi pracovního postupu, takže v různých fázích může být současně zpracováváno několik obrázků.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Architektura zpracování

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Každý obrázek prochází postupně všemi čtyřmi vlákny. Díky vícevláknové zpracováníChloros+ může více obrázků současně zabírat různá vlákna — zatímco vlákno 3 zpracovává jeden obrázek, vlákno 1 může detekovat další, vlákno 2 kalibrovat další a vlákno 4 zapisovat hotový obrázek na disk.

Průběh zpracování je hlásen pro každé vlákno a přenášen prostřednictvím Server-Sent Events (backend je publikuje na `/api/events`). V živém zobrazení průběhu v CLI jsou tyto čtyři fáze označeny jako **Detekce, Analýza, Zpracování, Export**.***

## Podrobnosti o vláknech

### Vlákno 1: Detekce

**Účel**: Načíst snímky a detekovat kalibrační cíle.

* Čte obrazové soubory z disku — páry Survey3 `.raw`+`.jpg`, snímky LATTICE `.tif`/`.tiff` a `.dng`
* Extrahuje metadata EXIF (GPS, model fotoaparátu, časová razítka, expozice)
* Detekuje kalibrační cíle: geometrie cílů označené ArUco pro snímky LATTICE a klasický panelový detektor pro fotografie kalibračních cílů Survey3
* Výstupy: obrazová data + metadata + výsledky detekce cílů

Jedná se především o vlákno závislé na vstupu/výstupu a výkonu procesoru.

### Vlákno 2: Kalibrace

**Účel**: Výpočet kalibračních parametrů z detekovaných cílů.

* Vypočítává kalibrační koeficienty odrazivosti z obrázků cílů
* Vypočítává parametry korekce vinětace
* Určuje kalibrační křivky pro jednotlivé pásma
* Výstupy: kalibrační parametry pro každý obrázek

Výpočetní vlákno závislé na CPU. Při běhu Survey3 nebo smíšeném běhu na něj čeká vlákno 3, pokud je povolena kalibrace odrazivosti, takže jeho koeficienty jsou připraveny ještě před zpracováním jakéhokoli snímku. Při běhu pouze s LATTICE běží vlákno 3 během detekce: každý snímek se spustí, jakmile je jeho cílový kotevní bod konečný. Snímek bere časově nejbližší cíl na obou stranách, takže čeká, dokud nejsou naskenovány všechny snímky z kamery, které jsou časově blíže než jeho aktuální nejbližší cíl. Výstupy jsou stejné, pouze dříve k dispozici.

### Vlákno 3: Zpracování (GPU)

**Účel**: Aplikuje korekce a vypočítává vegetační indexy.**Jedná se o výpočetně nejnáročnější vlákno.*** **Debayering**: převádí surová data formátu RAW Bayer na vícekanálové snímky
  * Standardní (rychlé, střední kvalita) — výchozí, `--debayer standard`
  * S ohledem na texturu (pomalé, nejvyšší kvalita) — pouze Chloros+, `--debayer texture-aware`, používá model odšumování založený na AI/ML
  * Snímky LATTICE mono (M3M) jsou jednopásmové: kroky demosaic a vyvážení bílé se u nich přeskočí (s jednořádkovou zprávou v protokolu), zatímco všechny snímky M3C/Bayer ve stejném běhu je stále procházejí
* **Korekce vinětace**: aplikuje korekci vinětace objektivu na celý snímek
* **Kalibrace odrazivosti**: aplikuje kalibrační koeficienty pro převod na hodnoty odrazivosti
* **Výpočet indexů**: vypočítává vegetační indexy (NDVI, NDRE, GNDVI, …)
* Výstupy: zpracovaná obrazová data připravená k exportu

Toto vlákno nejvíce těží z akcelerace GPU a je to právě vlákno, které ladí [Dynamic Compute Adaptation](dynamic-compute-adaptation.md).

### Vlákno 4: Export

**Účel**: Zápis zpracovaných snímků na disk.

* Zapisuje výstupní soubory ve vybraném formátu — `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Do výstupních souborů vkládá metadata (GPS, časová razítka, parametry zpracování) — včetně značek zpracování `Chloros:*`, které jsou od verze 1.3.0 zapisovány v každém režimu zpracování, a to jak pro Survey3, tak pro LATTICE
* Uspořádává výstup do projektové složky jako `<camera>/<format>/<Product>_Images/` — například `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Exportované soubory si zachovávají název zdrojového souboru; produkt identifikuje složka.**
* U snímků pořízených programem LATTICE se jeden zdrojový snímek může rozdělit na několik produktů (Debayered, Preview, Radiance, Reflectance, Index), z nichž každý je uložen ve vlastní produktové složce. Indexové produkty se vypočítávají z produktu odrazivosti, který byl při daném běhu vygenerován (pokud žádný neexistuje, pak z jiného produktu vygenerovaného při daném běhu, například z produktu radiance). Produkty jasu a odrazivosti z multispektrálních kamer M3C mohou také získat masku omezující hodnoty na úrovni jednotlivých pixelů v `<format>/clamp_masks/` (viz [Formáty výstupních obrázků](../output-image-formats.md#clamp-masks))
* Výstupy: finální soubory na disku

Jedná se především o vlákno omezené vstupem a výstupem — úložiště SSD jeho výkon znatelně zlepšuje.

***

## Pod kapotou: Exekutoři

V rámci vlákna 3 je práce na jednotlivých snímcích paralelizována následovně:

* **Strategie GPU**(`GPU_SINGLE`, `GPU_PARALLEL`) využívají skupinu**pracovních procesů**, z nichž každý je samostatným procesem operačního systému s vlastním kontextem CUDA. Chloros spouští každý pracovní proces tak, že znovu spustí svůj vlastní nainstalovaný backendový program — nikoli prostřednictvím Pythonu `multiprocessing`, jehož spouštění pracovních procesů nefunguje uvnitř balíkové aplikace
* **`CPU_PARALLEL`** využívá fond vláken (funkce `ThreadPoolExecutor` v Pythonu) — NumPy a OpenCV uvolňují GIL, takže vlákna postačují
* Zařízení Jetson se sdílenou RAM o velikosti 8 GB nebo méně vynechávají executor úplně a zpracovávají úlohy v rámci procesu, sekvenčně
* Funkce Texture Aware na GPU s méně než 7 GB VRAM také běží sekvenčně — model odšumovače se tam nevejde více než jednou.

Chloros nepoužívá žádný distribuovaný framework třetích stran (jako například Ray). Viz [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) pro informace o tom, jak se volí strategie a počet pracovníků.

***

## Sekvenční vs. pipelined zpracování

### Volný režim (sekvenční)

Ve bezplatné verzi Chlorosu jsou obrázky zpracovávány **po jednom**, sekvenčně ve všech čtyřech fázích:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

GUI zobrazuje v bezplatném režimu zjednodušený ukazatel průběhu; jeho sekvenční fáze jsou označeny jako **Detekce cílů**a poté**Zpracování**.

### Režim Chloros+ (pipelined)

S licencí Chloros+ pracují všechna čtyři vlákna **souběžně** na různých snímcích:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

Indikátor průběhu v grafickém uživatelském rozhraní (GUI) zobrazuje tyto čtyři fáze; při najetí myší na něj uvidíte průběh jednotlivých vláken. V příkazovém řádku (CLI) se tyto čtyři fáze zobrazují v reálném čase jako **Detekce, Analýza, Zpracování, Export**.

{% hint style="info" %}
**Jeden popisek, dva názvy.** CLI označuje fázi 3 jako _Zpracování_. Průběhový kanál backendu v prémiovém režimu – ten, který vykresluje indikátor průběhu v grafickém rozhraní – označuje stejnou fázi jako _Kalibrace_. Jedná se o stejné vlákno provádějící stejnou práci (vlákno 3: debayer, korekce, indexy).
{% endhint %}

{% hint style="success" %}
**Zpracování v potrubí (pipelined processing) s Chloros+** může být 3–5krát rychlejší než sekvenční zpracování, v závislosti na vašem hardwaru a velikosti datového souboru. Největší zrychlení je dosaženo na systémech s rychlými grafickými kartami a SSD disky.
{% endhint %}

***

## Průběh exportu v vlákně 4

Exportní vlákno má vlastní sledování průběhu, které můžete dotazovat samostatně:**Příkazový řádek (CLI):**

```bash
chloros-cli export-status
```

**Sada SDK:**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Zpracování je dokončeno, jakmile vlákno 4 dosáhne 100 %.

{% hint style="info" %}
**Spuštění, při kterém nedojde k zápisu žádných obrazů, je považováno za neúspěšné.**V případě úspěchu hlásí `chloros-cli process`, kolik obrazových produktů bylo zapsáno (`Image products written: N`). Pokud byly produkty vyžádány a**žádný**nebyl zapsán — pouze `project.json` a `calibration_data.json` — CLI vypíše `Processing finished but wrote no image products.` a**ukončí se s nenulovým stavem**, přičemž uvede název projektové složky a obvyklé příčiny (vstupní složka nebyla rozpoznána jako záznam – zkontrolujte rozložení a `--input-level` – nebo žádný z požadovaných produktů nebyl pro dané kamery použitelný). Skripty se mohou spolehnout na kód ukončení.
{% endhint %}

***

## Vztah k dynamické adaptaci výpočtů

[Dynamická adaptace výpočtů](dynamic-compute-adaptation.md) ovlivňuje především **vlákno 3 (zpracování)**:

* **`GPU_PARALLEL`**: Vlákno 3 zpracovává více obrázků současně na GPU pomocí potrubí `fused_gpu`
* **`GPU_SINGLE`**: Vlákno 3 serializuje přístup k GPU pomocí semaforu, zatímco pracovní procesy překrývají operace I/O, a to s využitím potrubí `fused_gpu` nebo paměťově efektivního potrubí `tiled_gpu`
* **`CPU_PARALLEL`**: Vlákno 3 využívá zpracování na CPU s vícevláknovou paralelizací

Alokace paměti GPU vlákna 3 se také zvětšuje, jakmile vlákna 1 a 2 dokončí svou práci — viz [Dynamická alokace paměti GPU](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Další kroky

* [Dynamická adaptace výpočtů](dynamic-compute-adaptation.md) — Jak Chloros vybírá optimální strategii pro váš hardware
* [Průvodce NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Chování potrubí specifické pro platformu Jetson
* [Sledování zpracování](../processing-images-gui/monitoring-the-processing.md) — Sledování průběhu pomocí grafického uživatelského rozhraní
* [Referenční příručka k CLI](../reference/cli-reference.md) — `process`, `export-status`, kódy ukončení a rozložení výstupu
