# Dynamické přizpůsobení výpočetního výkonu

Chloros
1.3.0 využívá detekci hardwaru a automatický výběr strategie zpracování. Zpracovací engine se přizpůsobí vašemu hardwaru – od Jetson Orin Nano až po pracovní stanici s více grafickými kartami – bez jakékoli ruční konfigurace.

***

## Jak to funguje

Při spuštění Chloros
provede profilování vašeho systému:

1. **Detekuje operační systém** — Windows
nebo Linux
2. **Identifikuje jádra CPU a celkovou velikost RAM**

3.**Detekuje přítomnost GPU** — podporu NVIDIA CUDA, VRAM, model
4. **Identifikuje model Jetson** (je-li relevantní) — prostřednictvím `/proc/device-tree/model`
5. **Zkontroluje teplotní senzory** (Jetson) — pro zpracování zohledňující teplotu
6. **Vybere výpočetní strategii** — na základě veškerého detekovaného hardwaru
7. **Automaticky konfiguruje počet pracovníků, typ potrubí a alokaci paměti**

Zjištěný profil se pro danou relaci ukládá do mezipaměti v paměti i na disku, takže pozdější spuštění probíhá rychleji:

| Platforma | Profil v mezipaměti |
| --- | --- |
| **Linux
/ Jetson** | `~/.config/chloros/system_config.json` (upřednostňuje `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Smazáním tohoto souboru vynutíte nové detekování — což je užitečné po přidání GPU nebo většího množství RAM. Chloros
také automaticky provede nové detekování, pokud byla mezipaměť zapsána nekompatibilní starší verzí.

***

## Výpočetní strategie

Chloros
vybírá jednu ze tří výpočetních strategií na základě vašeho hardwaru:

| Strategie | Vybráno, když | Pracovní procesy | Spouštěč | Potrubí |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| GPU s podporou CUDA hlásící**12 GB+ VRAM**(na sjednocené paměti Jetson, vyžaduje také celkem 12 GB+ sdílené RAM) | `min(4, ⌊(VRAM − 0.5GB) × 0.9 ÷ 3.5GB⌋)`, minimálně 2 —**na Jetsonu omezeno na 2** | Pracovní procesy (samostatné kontexty CUDA) | `fused_gpu` |
| **`GPU_SINGLE`**| GPU s podporou CUDA s**2–12 GB VRAM**| 3 (překrývání I/O; přístup k GPU serializován semaforem).**1 (sekvenční) na zařízeních Jetson s méně než 12 GB RAM** | Pracovní procesy (samostatné kontexty CUDA); sekvenční zpracování v rámci procesu na zařízeních Jetson s nízkou kapacitou RAM | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Bez CUDA GPU nebo s méně než 2 GB VRAM | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Praktické příklady vzorce pro počet pracovníků `GPU_PARALLEL`: 12 GB VRAM → 2 pracovníci, 16 GB → 3 pracovníci, 20 GB a více → 4 pracovníci. Každému pracovníkovi je přiděleno 3,5 GB, což odpovídá naměřené potřebě úlohy typu Texture Aware, a to v rámci 90 % VRAM zbývající po rezervě 0,5 GB.

{% hint style="info" %}
**Strategie pro GPU vyžadují PyTorch, který umí využívat CUDA.** Počítač, jehož PyTorch nemá žádné použitelné CUDA zařízení, dostane `CPU_PARALLEL`, bez ohledu na jeho GPU. Balíček Jetson 1.3.0 používá verzi PyTorchu pro CUDA z JetPacku 6, takže Jetson splňuje podmínky, jakmile jsou nainstalovány knihovny CUDA a cuDNN z JetPacku. Jetson s méně než 12 GB RAM vždy zpracovává jeden obrázek najednou, a to přímo v procesu (zaznamenáno jako `GPU_SINGLE`).
{% endhint %}

Strategie využívající GPU spouštějí skupinu pracovních procesů, z nichž každý je samostatným procesem operačního systému s vlastním kontextem CUDA, které Chloros
spouští opětovným spuštěním svého vlastního nainstalovaného backendového programu (nikoli prostřednictvím `multiprocessing` v Python
u, jehož spouštění pracovních procesů nefunguje uvnitř balíkové aplikace). Strategie CPU využívá fond vláken (Python
`ThreadPoolExecutor`). Chloros
nepoužívá žádný distribuovaný framework třetích stran (například Ray).

### Typy pipeline

* **`fused_gpu`** — Cesta s plným zpracováním na GPU. Operace debayeringu, korekce a indexování probíhají na GPU v jediném sloučeném průchodu. Nejvyšší propustnost, vyžaduje nejvíce VRAM.
* **`tiled_gpu`** — Cesta na GPU s efektivním využitím paměti. Zpracovává obrázky po částech, aby se vešly do omezené paměti GPU. Nižší propustnost, ale funguje na zařízeních s omezenou pamětí.
* **`cpu_fallback`** — Zpracování výhradně na CPU s využitím vícevláknové paralelizace. Používá se, když není k dispozici žádné GPU NVIDIA, a jako nouzové řešení v krajním případě, pokud selžou obě cesty na GPU.

Řetězec záložních řešení v běhovém prostředí je vždy `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Ruční přepsání strategie

Nastavte proměnnou prostředí `CHLOROS_STRATEGY`, abyste vynutili konkrétní strategii — jedná se o expertní nouzové řešení pro případ, kdy automatická detekce vybere něco nevhodného pro vaši situaci (například pokud chcete ponechat grafický procesor volný pro jinou práci):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

Při porovnávání proměnné se nerozlišují velká a malá písmena; vše, co není jedním ze tří názvů, je ignorováno a automatická detekce pokračuje normálně. I při přepsání strategie za vás Chloros
stále vybírá počet pracovníků:

| Přepsání | Použitý počet pracovníků |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

Přepsání na `GPU_SINGLE` nebo `GPU_PARALLEL` na stroji bez použitelného zařízení CUDA je odmítnuto s řádkem `[STRATEGY]` v protokolu, a běh použije `CPU_PARALLEL`.

Doporučujeme nastavit tuto hodnotu pro jednotlivé příkazy, nikoli trvale, aby se běžné běhy mohly i nadále automaticky přizpůsobovat.

***

## Chování specifické pro platformu

| Platforma | Strategie | Pracovníci | Pipeline | Poznámky |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekvenční) | Režim šetrný k paměti, jeden obraz najednou |
| **Jetson Orin NX 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekvenční) | Sdílená RAM pod 12 GB vynucuje sekvenční zpracování |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (souběžné) | Doporučené okrajové zařízení — u Jetsonu omezeno na 2 pracovníky |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (souběžné) | Maximální výkon na okraji sítě (také omezeno na 2 pracovníky u Jetsonu) |
| **Stolní počítač s 8 GB GPU** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 pracovníky se překrývají při I/O, zatímco semafor serializuje přístup k GPU |
| **Stolní počítač s GPU 12 GB a více** | `GPU_PARALLEL` | 2–4 | `fused_gpu` (souběžné) | Optimální výkon stolního počítače: 12 GB → 2 pracovníky, 16 GB → 3, 20 GB a více → 4 |
| **Systém pouze s CPU** | `CPU_PARALLEL` | fyzická jádra − 1 (min. 2) | `cpu_fallback` | Není vyžadována GPU, využívá fond vláken |

{% hint style="info" %}
**Unifikovaná paměť Jetson**: Zařízení Jetson sdílejí paměť GPU a CPU. Zařízení Jetson Orin NX s 16 GB vykazuje ~15,3 GB VRAM, ale jedná se o stejnou fyzickou RAM, kterou používají operační systém a procesy CPU. Proto se zařízení Jetson s 16 GB a více kvalifikují pro `GPU_PARALLEL` stejně jako stolní GPU s 12 GB a více, avšak jsou omezena na 2 pracovníky — GPU, pracovní procesy a jejich kontexty CUDA pro jednotlivé pracovníky čerpají ze stejného sdíleného fondu.
{% endhint %}

### Rozpočet GPU podle VRAM (diskrétní GPU)

Na hostitelských počítačích x86_64 s diskrétním GPU NVIDIA určuje detekovaná VRAM také to, kolik paměti si může vyhradit zpracování na kartě a jak velké mohou být dávky:

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

Chloros
používá [4-vláknový zpracovatelský kanál](processing-pipeline.md):

* **Vlákno 1** (Detekce) — Načítání obrázků, analýza EXIF, detekce cíle
* **Vlákno 2** (Kalibrace) — Výpočet kalibrace odrazivosti
* **Vlákno 3** (Zpracování) — Odstranění matice pixelů na GPU, korekce vinětace, výpočet indexu
* **Vlákno 4** (Export) — Zápis souboru, vkládání metadat

Vlákna 1, 2 a 4 jsou pro GPU nenáročná; vlákno 3 je to nejnáročnější. Jakmile se dokončí předchozí vlákna zpracovatelského řetězce, jejich GPU kapacita se **přerozdělí mezi zbývající aktivní vlákna**, takže vlákno 3 získává v průběhu zpracování postupně více paměti.

### Fáze alokace

| Fáze | Aktivní vlákna | Rozdělení paměti GPU |
| --- | --- | --- |
| **Počátek** | 1, 2, 3, 4 | Rozděleno mezi všechna vlákna, většina připadá na vlákno 3 |
| **Střední fáze** | 2, 3, 4 | Podíl vlákna 1 je přerozdělen |
| **Středně pozdní** | 3, 4 | Podíly vláken 1 a 2 přecházejí na vlákna 3 a 4 |
| **Pozdní** | 3 nebo 4 | Poslední aktivní vlákno získá maximální přidělenou paměť |

Čísla se řídí dvěma pravidly:

* Vlákno, které je **jediným** aktivním, obdrží maximální alokaci podle svého profilu.
* Je-li aktivních více než jeden *náročný* úkol na GPU, základní alokace každého náročného úkolu se mezi ně rozdělí (nikdy však neklesne pod nakonfigurované minimum).

Hodnota skutečně použitá při běhu je **nižší** z alokace profilu platformy a aktuálního doporučení z monitoru paměti GPU, takže zaneprázdněná karta vždy zvítězí nad optimistickým profilem.***

## Zpracování s ohledem na textury

Debayer s podporou textur (**pouze Chloros
+** — `--debayer texture-aware`) spouští model odšumování pomocí AI/ML, který potřebuje přibližně 1,75 GB VRAM v režimu FP16 na jednu kopii, takže využívá mnohem více paměti GPU než standardní metoda:

* Systémy s **méně než 7 GB VRAM**zpracovávají Texture Aware v**synchronní smyčce, jeden obrázek po druhém** — několik kopií modelu se tam nevejde a pool pracovníků by jen zvýšil konflikt
* Systémy s **7 GB a více VRAM** mohou Texture Aware zpracovávat souběžně, i když s nižším počtem pracovníků ve srovnání se Standard
* Na **Jetsonu** je Texture Aware vždy přiřazen k jedinému pracovníkovi
* Pokud **není k dispozici použitelná GPU s podporou CUDA**, běh použije standardní debayer a backend zaznamená do protokolu jeden řádek `[DEBAYER] … needs a CUDA GPU …` s vysvětlením důvodu
* Na nízkoenergetických zařízeních Jetson (Nano, Orin Nano) CLI
při požadavku na Texture Aware omezí frekvenci GPU — viz [Průvodce NVIDIA Jetson](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)

***

## Řízení teploty (Jetson)

Zařízení Jetson mají teplotní omezení, zejména při nasazení v uzavřených prostorech nebo ve vzduchu. Chloros
monitoruje integrované teplotní senzory Jetsonu a automaticky přizpůsobuje velikost dávek:

| Teplota | Reakce |
| --- | --- |
| **&lt; 70 °C** | Normální provoz — plná rychlost |
| **70 °C** (Varování) | Velikost dávky se postupně snižuje (100 % → 50 % v rozmezí 70 °C až 80 °C) |
| **80 °C** (Kritické) | Agresivní omezení výkonu (50 % → 0 % v rozmezí 80 °C až 90 °C) |
| **90 °C** (Vypnutí) | Úplné zastavení zpracování GPU |

Na stolních počítačích s dostatečným chlazením dochází k tepelnému omezování výkonu jen zřídka.

***

## Řešení přetížení paměti

Program Chloros
během zpracování nepřetržitě sleduje paměť GPU a reaguje na třech úrovních.

**Velikost dávky.** Dávka začíná na 8 obrázcích násobených koeficientem platformy z výše uvedených tabulek. Chloros
poté zkontroluje volnou VRAM, rezervuje 20 % z ní pro vlastní režii PyTorchu a předpokládá přibližně 100 MB paměti GPU na jeden obrázek o rozlišení 12 MP – velikost dávky je dána menší z těchto dvou hodnot: limitem odvozeným z paměti nebo základním limitem platformy. Nikdy neklesne pod hodnotu 1.

**Preventivní zmenšení.**Při**využití VRAM nad 85 %** se velikost dávek zmenší ještě předtím, než dojde k selhání.**Omezení alokace na vlákno.** Jak stoupá aktuální využití, rozpočet GPU pro každé vlákno se snižuje: ×0,75 při využití nad 80 %, ×0,5 při využití nad 90 %. Monitorovací prahy jsou 70 % (konzervativní), 85 % (normální provozní limit) a 95 % (riziko OOM).**OOM backoff a zotavení.** Pokud přesto dojde k události nedostatku paměti (OOM):

* velikost dávky se **sníží na polovinu** a při každém dalším nedostatku paměti se sníží znovu na polovinu — každá následující úspěšná dávka posune tuto penalizaci o jeden krok zpět
* alokace aktivních vláken se sníží na 70 % jejich aktuální hodnoty a alokátor přepne na konzervativní strategii, která se opět uvolní po sérii úspěšných alokací
* při silném zatížení se pipeline přepne z režimu `fused_gpu` na `tiled_gpu` a jako poslední možnost na `cpu_fallback`

**Paměť RAM hostitele (Jetson).** Před zpracováním odhadne CLI
špičkovou paměť hostitele na základě počtu snímků a režimu debayeringu a varuje, pokud je pravděpodobné, že RAM plus swap založený na souborech nebude stačit, přičemž vypíše přesné příkazy pro přidání swapového prostoru — viz [Průvodce NVIDIA Jetson](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).

***

## Sledování přizpůsobení výpočetní vrstvy

### Diagnostika systému

`chloros-cli selftest` je nejrychlejší způsob, jak ověřit, co vidí výpočetní vrstva:

```bash
chloros-cli selftest
```

Jeho 7 kontrol zahrnuje verzi, dostupnost portů, spuštění backendu, `/api/test`, systémové informace, přítomnost modelu odšumovače a připravenost CUDA + odšumovače. Kontrola 5 přímo vypíše řádek s hardwarem:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

Kontrola 7 vypíše `CUDA: <bool>, Denoiser: <bool>` — obě podmínky musí být splněny, aby bylo možné funkci Texture Aware vůbec použít.

### Protokoly backendu

Strategie a počet pracovníků se volí uvnitř backendu na začátku každého běhu — neexistuje žádný banner v příkazovém řádku (CLI
), který by je oznamoval. Pokud se něco chová neočekávaně (přepnutí na alternativní cestu GPU, nedostatek paměti (OOM), odšumovač, který se nenačte), objeví se to v protokolu backendu pro danou relaci:

| Platforma | Umístění protokolu |
| --- | --- |
| **Linux
/ Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (jeden soubor na jedno spuštění) |
| **Linux
, backend spuštěný přes CLI** | rovněž `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Průběžný stav

Během běhu zobrazuje CLI
v reálném čase průběh jednotlivých vláken (Detekce, Analýza, Zpracování, Export) streamovaný prostřednictvím Server-Sent Events — praktický údaj o tom, zda je vlákno 3 úzkým hrdlem. Viz [Zpracovací pipeline](processing-pipeline.md).

***

## Další kroky

* [Zpracovací pipeline](processing-pipeline.md) — Porozumění architektuře 4-vláknového pipeline
* [Průvodce NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Nasazení a optimalizace specifické pro Jetson
* [CLI
: Příkazový řádek](../CLI.md) — Průvodce příkazovým řádkem
* [Referenční příručka CLI
](../reference/cli-reference.md) — Úplný seznam příkazů pro verzi 1.3.0
