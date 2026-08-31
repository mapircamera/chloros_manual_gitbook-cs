# Zpracovatelský kanál

ChlorosVerze 1.2.0 využívá čtyřvláknový zpracovatelský kanál, který funguje jako postupná montážní linka. Každé vlákno zpracovává samostatnou fázi pracovního postupu, takže v různých fázích může být současně zpracováváno několik obrázků.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Architektura potrubí

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Každý obrázek prochází postupně všemi čtyřmi vlákny. Díky vícevláknovému zpracování v Chloros+ může více obrázků současně zabírat různá vlákna — zatímco vlákno 3 zpracovává jeden obrázek, vlákno 1 může detekovat další, vlákno 2 kalibrovat další a vlákno 4 zapisovat hotový obrázek na disk.

Průběh se hlásí pro každé vlákno a přenáší se prostřednictvím Server-Sent Events (backend je publikuje na `/api/events`). V živém zobrazení průběhu v nástroji CLI jsou tyto čtyři fáze označeny jako **Detekce, Analýza, Zpracování, Export**.***

## Podrobnosti o vláknech

### Vlákno 1: Detekce

**Účel**: Načíst snímky a detekovat kalibrační cíle.

* Čte obrazové soubory z disku — páry Survey3 `.raw`+`.jpg`, snímky LATTICE `.tif`/`.tiff` a `.dng`
* Extrahuje metadata EXIF (GPS, model fotoaparátu, časová razítka, expozice)
* Detekuje kalibrační cíle: geometrie cílů označených ArUco pro snímky LATTICE a klasický panelový detektor pro fotografie kalibračních cílů Survey3
* Výstupy: obrazová data + metadata + výsledky detekce cílů

Jedná se především o vlákno vázané na I/O a CPU.

### Vlákno 2: Kalibrace

**Účel**: Výpočet kalibračních parametrů z detekovaných cílů.

* Vypočítává kalibrační koeficienty odrazivosti z obrázků cílů
* Vypočítává parametry korekce vinětace
* Určuje kalibrační křivky pro jednotlivá pásma
* Výstupy: kalibrační parametry pro každý obrázek

Výpočetní vlákno závislé na CPU. Vlákno 3 na něj čeká, pokud je povolena kalibrace odrazivosti, takže jeho koeficienty jsou připraveny ještě před zpracováním jakéhokoli obrázku.

### Vlákno 3: Zpracování (GPU)

**Účel**: Aplikuje korekce a vypočítává vegetační indexy.**Toto je výpočetně nejnáročnější vlákno.*** **Debayering**: převádí surová data formátu RAW Bayer na vícekanálové snímky
  * Standardní (rychlé, střední kvalita) — výchozí nastavení, `--debayer standard`
  * S ohledem na texturu (pomalé, nejvyšší kvalita) — pouze pro snímky typu „Chloros+“, `--debayer texture-aware`, využívá model odšumování založený na AI/ML
  * Snímky LATTICE mono (M3M) jsou jednopásmové: kroky demosaiku a vyvážení bílé se u nich přeskočí (s jednovřádkovou zprávou v protokolu), zatímco všechny snímky M3C/Bayer ve stejném běhu tyto kroky stále procházejí
* **Korekce vinětace**: aplikuje korekci vinětace objektivu na celý snímek
* **Kalibrace odrazivosti**: aplikuje kalibrační koeficienty pro převod na hodnoty odrazivosti
* **Výpočet indexů**: vypočítává vegetační indexy (NDVI, NDRE, GNDVI, …)
* Výstupy: zpracovaná obrazová data připravená k exportu

Toto vlákno nejvíce těží z akcelerace pomocí GPU a je to právě vlákno, které ladí funkce [Dynamic Compute Adaptation](dynamic-compute-adaptation.md).

### Vlákno 4: Export

**Účel**: Zápis zpracovaných snímků na disk.

* Zapisuje výstupní soubory ve vybraném formátu — `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Do výstupních souborů vkládá metadata (GPS, časová razítka, parametry zpracování)
* Výstup uspořádává do složky projektu jako `<camera>/<format>/<Product>_Images/` — například `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Exportované soubory si zachovávají název zdrojového souboru; produkt identifikuje složka.**
* U snímků pořízených pomocí LATTICE se jeden zdrojový snímek může rozdělit na několik produktů (Debayered, Preview, Radiance, Reflectance, Index), z nichž každý je uložen ve vlastní produktové složce
* Výstupy: finální soubory na disku

Jedná se především o vlákno omezené vstupem a výstupem — úložiště SSD jeho výkon znatelně zlepšuje.

***

## Pod kapotou: Exekutoři

V rámci vlákna 3 je práce na jednotlivých snímcích paralelizována pomocí standardního `concurrent.futures` z balíčku „Python“:

* **Strategie GPU**(`GPU_SINGLE`, `GPU_PARALLEL`) používají metodu**spawn** — každý pracovní proces je samostatným procesem s vlastním kontextem CUDA (`fork` by zdědil inicializovaný stav CUDA rodičovského procesu a poškodil by potomky)
* **`CPU_PARALLEL`** používá `ThreadPoolExecutor` — NumPy a OpenCV uvolňují GIL, takže vlákna stačí
* Zařízení Jetson s 8 GB nebo méně sdílené RAM vynechávají executor úplně a zpracovávají data v rámci procesu, sekvenčně
* Funkce Texture Aware na GPU s méně než 7 GB VRAM také běží sekvenčně — model odšumovače se tam nevejde více než jednou

Chlorosnepoužívá žádný distribuovaný framework třetích stran (jako například Ray). Viz [Dynamic Compute Adaptation](dynamic-compute-adaptation.md), kde je popsáno, jak se volí strategie a počet pracovníků.

***

## Sekvenční vs. pipelined zpracování

### Režim Free (sekvenční)

Ve bezplatné verzi Chloros se snímky zpracovávají **po jednom**, postupně ve všech čtyřech fázích:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

GUI zobrazuje v bezplatném režimu zjednodušený ukazatel průběhu; jeho jednotlivé fáze jsou označeny jako **Detekce cíle**a poté**Zpracování**.

### Režim „Chloros“ (pipelined)

S licencí „Chloros“ pracují všechna čtyři vlákna **souběžně** na různých snímcích:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

Indikátor průběhu v grafickém uživatelském rozhraní (GUI) zobrazuje čtyři fáze; najedete-li na něj myší, uvidíte průběh jednotlivých vláken. V nástroji CLI se tyto čtyři fáze zobrazují v reálném čase jako **Detekce, Analýza, Zpracování, Export**.

{% hint style="info" %}
**Jeden název, dvě označení.** CLI označuje fázi 3 jako _Zpracování_. Kanál průběhu v prémiovém režimu backendu – ten, který vykresluje indikátor průběhu v grafickém rozhraní – označuje stejnou fázi jako _Kalibrace_. Jedná se o stejné vlákno provádějící stejnou práci (vlákno 3: debayer, korekce, indexy).
{% endhint %}

{% hint style="success" %}
**Pipelined processing s Chloros+** může být 3–5x rychlejší než sekvenční zpracování, v závislosti na vašem hardwaru a velikosti datového souboru. Zrychlení je největší na systémech s rychlými GPU a SSD.
{% endhint %}

***

## Průběh exportu vlákna 4

Exportní vlákno má vlastní sledování průběhu, které můžete dotazovat samostatně:**CLI:**

```bash
chloros-cli export-status
```

**SDK:**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Zpracování je dokončeno, jakmile vlákno 4 dosáhne 100 %.

{% hint style="info" %}
**Spuštění, při kterém se nezapíší žádné obrázky, je neúspěšné.**V případě úspěchu nahlásí `chloros-cli process`, kolik obrazových produktů zapsal (`Image products written: N`). Pokud byly produkty vyžádány a**žádný**nebyl zapsán — pouze `project.json` a `calibration_data.json` — program CLI vypíše `Processing finished but wrote no image products.` a**ukončí se s nenulovým stavem**, přičemž uvede název projektové složky a obvyklé příčiny (vstupní složka nebyla rozpoznána jako záznam – zkontrolujte rozložení a `--input-level` – nebo žádný z požadovaných produktů nebyl pro dané kamery použitelný). Skripty se mohou spolehnout na kód ukončení.
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
* [Průvodce NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Chování pipeline specifické pro platformu Jetson
* [Sledování zpracování](../processing-images-gui/monitoring-the-processing.md) — Sledování průběhu pomocí grafického uživatelského rozhraní
* [Referenční příručka k CLI](../reference/cli-reference.md) — `process`, `export-status`, kódy ukončení a rozložení výstupu
