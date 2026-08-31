# Dynamické přizpůsobení výpočetního výkonu

Chloros 1.2.0 využívá detekci hardwaru a automatický výběr strategie zpracování. Zpracovací engine se přizpůsobí vašemu hardwaru – od Jetson Orin Nano až po pracovní stanici s více GPU – bez jakékoli ruční konfigurace.

***

## Jak to funguje

Při spuštění Chloros provede profilování vašeho systému:

1. **Detekuje operační systém** — Windows nebo Linux
2. **Identifikuje jádra CPU a celkovou velikost RAM**

3.**Detekuje přítomnost GPU** — podporu NVIDIA CUDA, VRAM, model
4. **Identifikuje model Jetson** (pokud je k dispozici) — prostřednictvím `/proc/device-tree/model`
5. **Zkontroluje teplotní senzory** (Jetson) — pro zpracování s ohledem na teplotu
6. **Vybere výpočetní strategii** — na základě veškerého detekovaného hardwaru
7. **Automaticky konfiguruje počet pracovníků, typ pipeline a alokaci paměti**

Detekovaný profil se pro danou relaci ukládá do mezipaměti v paměti i na disku, takže pozdější spuštění probíhá rychleji:

| Platforma | Profil v mezipaměti |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (upřednostňuje `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Odstraněním tohoto souboru vynutíte nové rozpoznání — což je užitečné po přidání grafické karty nebo větší paměti RAM. Chloros také automaticky provede nové rozpoznání, pokud byla mezipaměť zapsána nekompatibilní starší verzí.

***

## Výpočetní strategie

Chloros vybírá jednu ze tří výpočetních strategií na základě vašeho hardwaru:

| Strategie | Vybráno, když | Pracovníci | Exekutor | Pipeline |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| GPU s podporou CUDA vykazující**12 GB a více VRAM**(v jednotné paměti Jetson, vyžaduje také celkem 12 GB a více sdílené RAM) | `min(4, VRAM ÷ 4GB)`, minimálně 2 —**na Jetsonu omezeno na 2** | `ProcessPoolExecutor` (spawn) | `fused_gpu` |
| **`GPU_SINGLE`**| GPU s podporou CUDA s**2–12 GB VRAM**| 3 (překrývání I/O; přístup k GPU serializován semaforem).**1 (sekvenční) na zařízeních Jetson s méně než 12 GB RAM** | `ProcessPoolExecutor` (spawn); sekvenční v rámci procesu na zařízeních Jetson s nízkou kapacitou RAM | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Bez GPU s podporou CUDA nebo s méně než 2 GB VRAM | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Praktické příklady vzorce pro počet pracovníků `GPU_PARALLEL`: 12 GB VRAM → 3 pracovníci, 16 GB a více → 4 pracovníci, jakýkoli Jetson → 2 pracovníci.

Paralelismus je implementován pomocí standardního `concurrent.futures` z Python: strategie GPU používají `ProcessPoolExecutor` s metodou spuštění **spawn** (každý pracovník je samostatný proces s vlastním kontextem CUDA — `fork` by zkopíroval již inicializovaný stav CUDA a poškodil by potomky), zatímco strategie pro CPU používá `ThreadPoolExecutor`. Chloros nepoužívá žádný distribuovaný framework třetí strany (například Ray).

### Typy potrubí

* **`fused_gpu`** — Plná cesta zpracování na GPU. Operace debayeringu, korekce a indexování probíhají na GPU v jediném sloučeném průchodu. Nejvyšší propustnost, vyžaduje nejvíce VRAM.
* **`tiled_gpu`** — Cesta na GPU s efektivním využitím paměti. Zpracovává obrázky po dlaždicích, aby se vešly do omezené paměti GPU. Nižší propustnost, ale funguje na zařízeních s omezenou pamětí.
* **`cpu_fallback`** — Zpracování výhradně na CPU s využitím vícevláknové paralelizace. Používá se, když není k dispozici žádné GPU NVIDIA, a jako poslední možnost v případě selhání obou cest na GPU.

Řetězec záložních řešení v běhovém prostředí je vždy `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Ruční přepsání strategie

Nastavte proměnnou prostředí `CHLOROS_STRATEGY`, abyste vynutili konkrétní strategii — jedná se o „záchranný východ“ pro odborníky pro případ, že automatická detekce vybere něco nevhodného pro vaši situaci (například pokud chcete ponechat GPU volné pro jinou práci):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

Při porovnávání proměnné se nerozlišují velká a malá písmena; vše, co není jedním ze tří názvů, je ignorováno a automatická detekce pokračuje normálně. I při přepsání vám Chloros stále vybere počet pracovníků:

| Přepsání | Použitý počet pracovníků |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

Doporučujeme nastavit tuto hodnotu pro jednotlivé příkazy, nikoli trvale, aby se běžné spuštění mohlo i nadále automaticky přizpůsobovat.

***

## Chování specifické pro jednotlivé platformy

| Platforma | Strategie | Pracovní procesy | Pipeline | Poznámky |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekvenční) | Režim s úsporným využitím paměti, jeden obrázek najednou |
| **Jetson Orin NX 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekvenční) | Sdílená RAM menší než 12 GB vynucuje sekvenční zpracování |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (souběžné) | Doporučené okrajové zařízení — u Jetsonu omezeno na 2 pracovníky |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (souběžné) | Maximální výkon na okraji sítě (také omezeno na 2 pracovníky na Jetsonu) |
| **Stolní počítač s 8 GB GPU** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 pracovníky se překrývají při vstupu/výstupu, zatímco semafor serializuje přístup k GPU |
| **Stolní počítač s GPU 12 GB a více** | `GPU_PARALLEL` | 3–4 | `fused_gpu` (souběžné) | Optimální výkon stolního počítače: 12 GB → 3 pracovníky, 16 GB a více → 4 |
| **Systém pouze s CPU** | `CPU_PARALLEL` | fyzická jádra − 1 (min. 2) | `cpu_fallback` | Není vyžadována GPU, využívá fond vláken |

{% hint style="info" %}
**Unifikovaná paměť Jetson**: Zařízení Jetson sdílejí paměť GPU a CPU. Zařízení Jetson Orin NX 16 GB vykazuje ~15,3 GB VRAM, ale jedná se o stejnou fyzickou RAM, kterou využívají operační systém a procesy CPU. Proto se zařízení Jetson s 16 GB a více kvalifikují pro `GPU_PARALLEL` stejně jako stolní GPU s 12 GB a více, avšak jsou omezena na 2 pracovníky — GPU, pracovní procesy a jejich kontexty CUDA pro jednotlivé pracovníky čerpají ze stejného sdíleného fondu.
{% endhint %}

### Rozpočet GPU podle VRAM (diskrétní GPU)

Na hostitelích x86_64 s diskrétním GPU NVIDIA určuje detekovaná VRAM také to, kolik si může zpracování karty nárokovat a jak velké mohou být dávky:

| Detekovaná VRAM | Horní hranice rozpočtu GPU | Násobitel velikosti dávky |
| --- | --- | --- |
| **8 GB a více** | 90 % | ×2,0 |
| **6–8 GB** | 85 % | ×1,75 |
| **3,5–6 GB** | 80 % | ×1,5 |
| **2–3,5 GB** | 75 % | ×1,25 |
| **Méně než 2 GB** | 70 % | ×1,0 |

Diskrétní grafické karty rezervují pro systém pouze 0,5 GB, protože nesdílejí systémovou RAM. Profily Jetson rezervují mnohem více a mají nižší limit — viz [Průvodce NVIDIA Jetson](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Dynamické přidělování paměti GPU

Chloros využívá [4vláknový zpracovatelský kanál](processing-pipeline.md):

* **Vlákno 1** (Detekce) — Načtení obrázku, analýza EXIF, detekce cíle
* **Vlákno 2** (Kalibrace) — Výpočet kalibrace odrazivosti
* **Vlákno 3** (Zpracování) — Odstranění matice pixelů pomocí GPU, korekce vinětace, výpočet indexu
* **Vlákno 4** (Export) — Zápis souboru, vložení metadat

Vlákna 1, 2 a 4 jsou pro GPU nenáročná; vlákno 3 je nejnáročnější. Jakmile se dokončí předchozí vlákna procesního řetězce, jejich GPU kapacita se **přerozdělí mezi zbývající aktivní vlákna**, takže vlákno 3 získává v průběhu běhu postupně více paměti.

### Fáze alokace

| Fáze | Aktivní vlákna | Rozdělení paměti GPU |
| --- | --- | --- |
| **Počátek** | 1, 2, 3, 4 | Rozděleno mezi všechna vlákna, většina připadá na vlákno 3 |
| **Střední fáze** | 2, 3, 4 | Podíl vlákna 1 je přerozdělen |
| **Středně pozdní** | 3, 4 | Podíly vláken 1 a 2 přecházejí na vlákna 3 a 4 |
| **Pozdní** | 3 nebo 4 | Poslední aktivní vlákno získá maximální přidělenou paměť |

Čísla se řídí dvěma pravidly:

* Vlákno, které je **jediným** aktivním, obdrží maximální přidělení podle svého profilu.
* Je-li aktivních více než jedna *náročná* úloha na GPU, základní přidělení každé náročné úlohy se mezi ně rozdělí (nikdy však neklesne pod nakonfigurované minimum).

Hodnota skutečně použitá při běhu je **nižší** z alokace profilu platformy a aktuálního doporučení z monitoru paměti GPU, takže zaneprázdněná karta vždy zvítězí nad optimistickým profilem.***

## Zpracování s ohledem na textury

Debayer s podporou textur (**pouze Chloros+** — `--debayer texture-aware`) spouští model odšumování založený na AI/ML, který potřebuje přibližně 1,75 GB VRAM v FP16 na jednu kopii, takže využívá mnohem více paměti GPU než standardní metoda:

* Systémy s **méně než 7 GB VRAM**zpracovávají Texture Aware v**synchronní smyčce, jeden obrázek po druhém** — nevejde se do nich více kopií modelu a pool pracovníků by jen zvýšil konkurenci o přístup k paměti
* Systémy s **7 GB a více VRAM** mohou funkci Texture Aware zpracovávat souběžně, avšak s nižším počtem pracovníků ve srovnání s metodou Standard
* Na **Jetsonu** je režim Texture Aware vždy přiřazen k jedinému pracovnímu procesu a u modelů s nízkou spotřebou (Nano, Orin Nano) se také automaticky uplatňuje omezení frekvence GPU — viz [Průvodce NVIDIA Jetson](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)***

## Řízení teploty (Jetson)

Zařízení Jetson mají teplotní omezení, zejména při nasazení v uzavřených prostorech nebo ve vzduchu. Chloros monitoruje integrované teplotní senzory Jetsonu a automaticky přizpůsobuje velikost dávek:

| Teplota | Reakce |
| --- | --- |
| **&lt; 70 °C** | Normální provoz — plná rychlost |
| **70 °C** (Varování) | Velikost dávky se postupně snižuje (100 % → 50 % v rozmezí 70 °C až 80 °C) |
| **80 °C** (Kritické) | Agresivní omezení výkonu (z 50 % na 0 % v rozmezí 80 °C až 90 °C) |
| **90 °C** (Vypnutí) | Úplné zastavení zpracování GPU |

Na stolních počítačích s dostatečným chlazením dochází k tepelnému omezování výkonu jen zřídka.

***

## Řízení zatížení paměti

Chloros během zpracování nepřetržitě sleduje paměť GPU a reaguje na třech úrovních.

**Velikost dávky.** Dávka začíná na 8 obrázcích násobených koeficientem platformy z výše uvedených tabulek. Chloros poté zkontroluje volnou VRAM, rezervuje 20 % z ní pro vlastní režii PyTorchu a předpokládá přibližně 100 MB paměti GPU na obrázek o rozlišení 12 MP — velikost dávky je dána menší z těchto hodnot: limit odvozený z paměti nebo základní hodnota pro danou platformu. Nikdy neklesne pod 1.**Preventivní zmenšení.**Při**využití VRAM nad 85 %** se velikost dávek zmenší dříve, než dojde k selhání.**Omezení přidělování paměti na vlákno.** Jak vzrůstá aktuální využití, rozpočet GPU pro každé vlákno se snižuje: ×0,75 při využití nad 80 %, ×0,5 při využití nad 90 %. Monitorovací prahy jsou 70 % (konzervativní), 85 % (normální provozní limit) a 95 % (riziko nedostatku paměti).**OOM backoff a zotavení.** Pokud přesto dojde k události nedostatku paměti:

* velikost dávky se **sníží na polovinu** a při každém dalším výskytu nedostatku paměti se sníží znovu na polovinu — každá následující úspěšná dávka posune tuto penalizaci o jeden krok zpět
* alokace aktivních vláken se sníží na 70 % jejich aktuální hodnoty a alokátor přepne na konzervativní strategii, která se po sérii úspěšných alokací opět uvolní
* při silném zatížení se pipeline přepne z režimu `fused_gpu` na `tiled_gpu` a jako poslední možnost na `cpu_fallback`

**Paměť RAM hostitele (Jetson).** Před zpracováním odhadne režim CLI špičkovou paměť hostitele na základě počtu snímků a režimu debayeringu a varuje, pokud je pravděpodobné, že RAM spolu se swapem založeným na souborech nebude stačit, přičemž vypíše přesné příkazy pro přidání swapu — viz [Průvodce NVIDIA Jetson](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).***

## Sledování přizpůsobení výpočetní vrstvy

### Diagnostika systému

`chloros-cli selftest` je nejrychlejší způsob, jak ověřit, co vidí výpočetní vrstva:

```bash
chloros-cli selftest
```

Jeho 7 kontrol zahrnuje verzi, dostupnost portů, spuštění backendu, `/api/test`, systémové informace, přítomnost modelu odšumovače a připravenost CUDA + odšumovače. Kontrola 5 vypíše přímo řádek s hardwarem:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

Kontrola 7 vypíše `CUDA: <bool>, Denoiser: <bool>` — obě musí být splněny, aby bylo možné funkci Texture Aware vůbec použít.

### Protokoly backendu

Strategie a počet pracovníků se volí uvnitř backendu na začátku každého běhu — neexistuje žádný záhlaví CLI, které by je oznamovalo. Pokud se něco chová neočekávaně (přepnutí na GPU cestu, OOM, denoiser, který se nenačte), objeví se to v protokolu backendu pro danou relaci:

| Platforma | Umístění protokolu |
| --- | --- |
| **Linux / Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (jeden soubor na jedno spuštění) |
| **Linux, CLI-started backend** | také `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Průběh v reálném čase

Během běhu zobrazuje soubor CLI průběh v reálném čase pro jednotlivá vlákna (Detekce, Analýza, Zpracování, Export) streamovaný prostřednictvím Server-Sent Events — praktický údaj o tom, zda je vlákno 3 úzkým hrdlem. Viz [Zpracovací pipeline](processing-pipeline.md).

***

## Další kroky

* [Zpracovací pipeline](processing-pipeline.md) — Porozumění architektuře 4-vláknového pipeline
* [Průvodce NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Nasazení a optimalizace specifické pro Jetson
* [CLI : Příkazový řádek](../CLI.md) — Průvodce CLI
* [CLI Reference](../reference/cli-reference.md) — Úplný seznam příkazů pro verzi 1.2.0
