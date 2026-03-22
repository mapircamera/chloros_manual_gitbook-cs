# Dynamické přizpůsobení výpočetního výkonu

Verze Chloros 1.1.0 přináší inteligentní detekci hardwaru a automatický výběr strategie zpracování. Zpracovací engine se přizpůsobí vašemu hardwaru – od Jetson Nano až po pracovní stanici s více grafickými kartami – bez jakékoli ruční konfigurace.

***

## Jak to funguje

Po spuštění Chloros automaticky provede profilování vašeho systému:

1. **Detekuje operační systém** — Windows nebo Linux
2. **Identifikuje jádra CPU a celkovou RAM**

3.**Detekuje přítomnost GPU** — schopnost NVIDIA CUDA, VRAM, model
4. **Identifikuje model Jetson** (pokud je k dispozici) — prostřednictvím `/proc/device-tree/model`
5. **Kontroluje teplotní senzory** (Jetson) — pro zpracování s ohledem na teplotu
6. **Vybere optimální výpočetní strategii** — na základě veškerého detekovaného hardwaru
7. **Automaticky nakonfiguruje počet pracovníků, typ pipeline a alokaci paměti**Výsledek se ukládá do mezipaměti, takže následující spuštění začínají rychleji. Pokud dojde ke změnám hardwaru (např. přidání GPU), Chloros při dalším spuštění provede nové profilování.***

## Výpočetní strategie

Chloros vybírá jednu ze tří výpočetních strategií na základě vašeho hardwaru:

| Strategie | Požadovaná GPU | Pracovníci | Pipeline | Nejvhodnější pro |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`** | Ano (12 GB+ VRAM nebo 16 GB+ sdílené) | 3–4 | `fused_gpu` | Stolní GPU s 12 GB+, Jetson Orin NX 16 GB, AGX Orin |
| **`GPU_SINGLE`** | Ano (&lt; 12 GB VRAM) | 1–3 | `tiled_gpu` | Základní grafické karty, Jetson Nano, Orin Nano |
| **`CPU_PARALLEL`** | Ne | jádra – 1 | `cpu_fallback` | Systémy bez grafického procesoru NVIDIA |

### Typy pipeline

* **`fused_gpu`** — Plná cesta zpracování grafického procesoru. Všechny operace debayeringu, korekce a indexování probíhají na GPU v jediném fúzovaném průchodu. Nejvyšší propustnost, ale vyžaduje více VRAM.
* **`tiled_gpu`** — Cesta GPU s efektivním využitím paměti. Zpracovává obrázky v dlaždicích, aby se vešly do omezené paměti GPU. Nižší propustnost, ale funguje na zařízeních s omezenou pamětí.
* **`cpu_fallback`** — Zpracování pouze na CPU s využitím vícevláknové paralelizace. Používá se, když není k dispozici GPU NVIDIA.***

## Chování specifické pro platformu

| Platforma | Strategie | Pracovníci | Pipeline | Poznámky |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sériové) | Režim šetrný k paměti, zpracovává jeden obrázek najednou |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` (souběžné) | Doporučené okrajové zařízení — skutečné paralelní zpracování GPU |
| **Jetson AGX Orin 64 GB** | `GPU_PARALLEL` | 4 | `fused_gpu` (souběžné) | Maximální výkon na okraji sítě |
| **Stolní počítač s 8GB GPU** | `GPU_SINGLE` | 3 | `tiled_gpu` | Dobrý výkon stolního počítače s paměťově efektivními dlaždicemi |
| **Stolní počítač s GPU 12 GB+** | `GPU_PARALLEL` | 3–4 | `fused_gpu` | Optimální výkon na stolním počítači |
| **Systém pouze s CPU** | `CPU_PARALLEL` | jádra – 1 | `cpu_fallback` | Není vyžadován GPU, používá ThreadPool |

{% hint style="info" %}
**Unifikovaná paměť Jetson**: Zařízení Jetson sdílejí paměť GPU a CPU. Jetson Orin NX 16 GB vykazuje ~15,3 GB VRAM, ale jedná se o stejnou fyzickou RAM, kterou používají procesy operačního systému a CPU. Chloros to zohledňuje při nastavování prahových hodnot pro přidělování paměti.
{% endhint %}

***

## Dynamické přidělování paměti GPU

Chloros používá [4vláknové zpracovatelské potrubí](processing-pipeline.md):

* **Vlákno 1** (Detekce) — Načítání obrázků, analýza EXIF, detekce cíle
* **Vlákno 2** (Kalibrace) — Výpočet kalibrace odrazivosti
* **Vlákno 3** (Zpracování) — Debayer GPU, korekce vinětace, výpočet indexu
* **Vlákno 4** (Export) — Zápis souboru, vkládání metadat

Jakmile předchozí vlákna potrubí dokončí svou práci (např. byly detekovány všechny obrázky), uvolní se jejich alokace paměti GPU a **přečlenní se na zbývající aktivní vlákna**. To znamená, že vlákno 3 (fáze náročná na GPU) získává postupně více paměti s postupem pipeline, čímž se zvyšuje propustnost pro výpočetně nejnáročnější práci.

### Fáze alokace

| Fáze | Aktivní vlákna | Rozdělení paměti GPU |
| --- | --- | --- |
| **Počáteční** | 1, 2, 3, 4 | Rozděleno mezi všechna vlákna |
| **Středně raná** | 2, 3, 4 | Paměť vlákna 1 přerozdělena |
| **Středně pozdní** | 3, 4 | Paměť vláken 1+2 přechází na 3+4 |
| **Pozdní** | 3 nebo 4 | Maximální paměť pro zbývající vlákno |***

## Zpracování s podporou textur

Metoda debayeringu s podporou textur (pouze Chloros+) využívá výrazně více paměti GPU než standardní metoda kvůli modelu odšumování AI/ML:

* Systémy s **&lt; 7 GB VRAM** jsou v režimu s podporou textur nuceny do synchronní zpracovatelské smyčky (jeden obrázek po druhém)
* Systémy s **7 GB a více VRAM** mohou zpracovávat Texture Aware souběžně, avšak s nižším počtem pracovníků ve srovnání se Standard***

## Řízení teploty (Jetson)

Zařízení Jetson mají teplotní omezení, zejména v uzavřených nebo leteckých nasazeních. Chloros monitoruje teploty GPU a CPU a automaticky upravuje zpracování:

| Teplota | Reakce |
| --- | --- |
| **&lt; 70 °C** | Normální provoz — plná rychlost |
| **70 °C** (Varování) | Snížit velikost dávky |
| **80 °C** (Kritické) | Agresivní omezení — snížit souběžnost a počet pracovníků |
| **90 °C** (Vypnutí) | Úplné zastavení zpracování GPU |

Sledování teploty využívá na platformách Jetson nástroj `tegrastats`. Na stolních systémech s dostatečným chlazením se tepelné omezování spouští jen zřídka.

***

## Řešení přetížení paměti

Chloros monitoruje tlak na systémovou paměť během zpracování:

* **Prahová hodnota paměti**: 85% využití spouští konzervativní chování
* **Redukce OOM**: Pokud dojde k události nedostatku paměti, alokace se sníží o 25 % (multiplikátor 0,75x)
* **Zpětný přechod potrubí**: Při velkém zatížení paměti se pipeline automaticky přepne z `fused_gpu` na `tiled_gpu`
* **Doporučení ohledně swapování**: Na Jetsonu vás Chloros upozorní, pokud je swapový prostor nedostatečný pro velikost vašeho datového souboru***

## Sledování přizpůsobení výpočetního výkonu

### Výstup stavu CLI

Po spuštění zpracování zobrazí CLI detekovaný profil hardwaru:

```

Chloros CLI 1.1.0
Platform: Linux aarch64 (Jetson Orin NX 16GB)
Strategy: GPU_PARALLEL | Workers: 3 | Pipeline: fused_gpu
CUDA: Available | GPU Memory: 15.3 GB (shared)
```

### Diagnostika systému

Spusťte `chloros-cli selftest`, abyste si prohlédli úplný profil hardwaru a ověřili výpočetní schopnosti:

```bash
chloros-cli selftest
```

Tím se zkontroluje dostupnost CUDA, paměť GPU, modely odšumovače a připojení backendu.

***

## Další kroky

* [Zpracovací pipeline](processing-pipeline.md) — Porozumění architektuře 4-vláknové pipeline
* [Průvodce NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Nasazení a optimalizace specifické pro Jetson
* [CLI : Příkazový řádek](../CLI.md) — Kompletní reference CLI
