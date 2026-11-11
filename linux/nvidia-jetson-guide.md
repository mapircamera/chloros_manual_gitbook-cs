# Průvodce NVIDIA Jetson Aplikace

Chloros
pro NVIDIA Jetson umožňuje zpracování multispektrálních snímků na okraji sítě – v terénu, na bezpilotních letadlech (UAV) a ve vzdálených instalacích. Chloros
1.3.0 při spuštění rozpozná váš model Jetson a optimalizuje strategii zpracování podle zjištěného hardwaru. **Není nutné žádné ruční ladění.**
{% hint style="info" %}
**Balíček Jetson 1.3.0 provádí zpracování na GPU zařízení Jetson.** `chloros_1.3.0_arm64_jp6.deb` je sestaven pro JetPack 6 a využívá CUDA verzi PyTorchu z JetPacku 6, takže zpracování, včetně debayera s podporou textur, probíhá na GPU Orinu. Balíček neobsahuje samotnou knihovnu CUDA: využívá knihovny CUDA a cuDNN nainstalované JetPackem 6 (`sudo apt install nvidia-jetpack` na obrazu, kde tyto knihovny chybí). Pokud není k dispozici žádný grafický procesor s podporou CUDA, zpracování probíhá na procesoru a funkce Texture Aware používá standardní debayer.
{% endhint %}

***

## Podporované modely Jetson

| Model                | RAM            | Strategie zpracování                                     | Doporučené použití                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB sdílené | `GPU_PARALLEL` (2 pracovní procesy)                              | Maximální výkon, velké datové sady                      |
| **Jetson Orin NX**   | 8–16 GB sdílené | `GPU_PARALLEL` (2 pracovní procesy, 16 GB) / `GPU_SINGLE` (8 GB)   | Hlavní doporučení pro nasazení v terénu              |
| **Jetson Orin Nano** | 8 GB sdílené paměti     | `GPU_SINGLE` (1 pracovní proces, sekvenční)                     | Základní úroveň výpočtů na okraji sítě                                 |

{% hint style="info" %}
Balíček pro Linux
arm64 vyžaduje **JetPack 6**, který je k dispozici pro řadu Jetson Orin. Starší modely (Nano, TX2, Xavier NX) nemohou spustit JetPack 6 a nejsou aktuálním balíčkem podporovány.
{% endhint %}

***

## Požadavky

* **JetPack 6.x** (doporučena nejnovější verze)
* **NVIDIA CUDA a cuDNN** z balíčku JetPack 6 (`sudo apt install nvidia-jetpack`, pokud je váš obraz neobsahuje)
* **Placený tarif Chloros
+** — úroveň Copper nebo vyšší (vyžadováno pro veškerý přístup k CLI
a SDK
; vynucováno na straně serveru)

## Instalace

```bash
# Install the arm64 .deb package (JetPack 6) -- apt, not dpkg -i, so the
# Python SDK's requirements (python3-numpy, python3-requests) come too
sudo apt install ./chloros_1.3.0_arm64_jp6.deb

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.3.0"
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # the .deb installs the SDK

# Run system diagnostics
chloros-cli selftest
```

`.deb` nainstaluje Python
ové SDK
pro systém `python3`, odpovídající této verzi. Pouze do virtuálního prostředí (virtualenv) je třeba nainstalovat přiložený balíček wheel: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` (zcela offline). Obecné podrobnosti o instalaci v systému Linux
, umístění souborů a řešení problémů najdete v části [Instalace v systému Linux
](linux-installation.md).

{% hint style="info" %}
**Při spuštění již nedochází k rozbalování.** Od verze 1.3.0 se backend a CLI
instalují v rozbaleném stavu do adresáře `/usr/lib/chloros/`, takže se již při každém spuštění nerozbalují do dočasného adresáře. Dočasné soubory se i nadále automaticky ukládají do adresáře `/mnt/ssd/tmp`, pokud tento adresář existuje; v opačném případě můžete nastavit `TMPDIR` na cestu na vašem disku NVMe (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Dynamická adaptace výpočetního výkonu na Jetsonu

### Jak to funguje

Při spuštění Chloros
provede profilování vašeho systému:

1. **Detekuje model Jetsonu** prostřednictvím `/proc/device-tree/model`
2. **Načte dostupnou sdílenou paměť GPU/CPU** (Jetson používá sjednocenou paměť)
3. **Zeptá se PyTorchu, zda může používat CUDA** — pokud je nainstalován JetPack 6 s CUDA a cuDNN, může
4. **Automaticky vybere strategii zpracování** (`GPU_PARALLEL`, `GPU_SINGLE` nebo `CPU_PARALLEL`) a počet pracovníků

Rozhodnutí závisí na **celkové velikosti sdílené RAM**, nikoli na názvu modelu:

* **Při celkové RAM pod 12 GB**(všechny Jetsony s 8 GB):**1 pracovní proces — záměrné sekvenční zpracování**. Paměť je pro souběžné pracovní procesy příliš omezená, takže se obrázky zpracovávají po jednom. Na zařízeních Jetson s**8 GB nebo méně** vlákno 3 zcela vynechává fond pracovníků a provádí zpracování jednotlivých snímků přímo v rámci procesu. Backendový protokol označuje tuto strategii jako `GPU_SINGLE`.
* **12 GB nebo více**(Orin NX 16 GB, AGX Orin): sjednocená paměť splňuje podmínky pro `GPU_PARALLEL`, ale počet pracovníků je**na Jetsonu omezen na 2** — GPU, RAM pracovních procesů i kontexty CUDA pro jednotlivé pracovníky čerpají ze stejného sdíleného fondu, takže při větším počtu pracovních procesů hrozí selhání z důvodu nedostatku paměti.
* **Žádná použitelná CUDA** (chybí knihovny CUDA pro JetPack): `CPU_PARALLEL` při 12 GB a více, níže je uveden jeden obraz najednou.

Automatický výběr můžete přepsat pomocí proměnné prostředí `CHLOROS_STRATEGY` — viz [Dynamická adaptace výpočtů](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Chování podle modelu

| Model Jetson                | Strategie       | Pracovní procesy | Spuštění                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Sekvenční smyčka v rámci procesu (`tiled_gpu` při nedostatku paměti) |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 1       | Sekvenční smyčka v rámci procesu                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Souběžné pracovní procesy, cesta `fused_gpu`  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2       | Souběžné pracovní procesy, cesta `fused_gpu`  |

Klíčovým rozdílem mezi platformami je **paměť**. 8GB Jetson musí při vysokém zatížení zpracovávat obrázky po jednom pomocí paměťově efektivního dlaždicového přístupu, zatímco Orin s 16 GB a více může pomocí fúzovaného potrubí s vyšší propustností zpracovávat 2 obrázky současně přes GPU.

### Rozpočet GPU na model

Každý model Jetson má také hardwarový profil, který omezuje, kolik prostředků ze sdíleného fondu může zpracování nárokovat, a škáluje velikosti dávek:

| Model | Horní hranice rozpočtu GPU | Násobitel velikosti dávky | Rezervováno pro systém/displej |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70 % | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75 % | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80 % | ×1,5 | 4,0 GB |

Profil se přizpůsobuje podle detekované velikosti RAM: u zařízení Jetson s **16 GB nebo více** se násobitel velikosti dávky zvýší na ×1,2. Základní velikost dávky před použitím násobitelů je 8 obrázků.

Kompletní referenční informace o přizpůsobení výpočetního výkonu najdete v článku [Dynamické přizpůsobení výpočetního výkonu](../processing-architecture/dynamic-compute-adaptation.md).

***

## Omezení frekvence GPU pro funkci Texture Aware na modelech Nano a Orin Nano

Debayer funkce Texture Aware provádí inferenci neuronové sítě na GPU, což může u nízkoenergetických modelů Jetson (třída 10–15 W) při plné taktovací frekvenci GPU vyvolat **varování před nadproudem**. Pokud zadáte `--debayer texture-aware` na**Jetson Nano nebo Orin Nano**, CLI
zkontroluje maximální frekvenci GPU a omezí ji na **510 MHz** (510000000), pokud je aktuálně vyšší:

* Pokud může CLI
zapisovat do uzlu sysfs pro frekvenci GPU, omezení se **použije automaticky** a zobrazí se potvrzení.
* Pokud ne (vyžaduje oprávnění root), CLI
vypíše přesný příkaz `sudo` pro ruční nastavení omezení, chvíli počká, abyste si jej mohli přečíst, a poté pokračuje — zpracování stále běží, ale mohou se zobrazovat varování před nadproudem.

Chcete-li limit nastavit sami před zpracováním:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Modely s vyšším výkonem (Orin NX 25 W, AGX Orin 60 W) běží na plné rychlosti GPU; žádný limit se neuplatňuje. Standardní debayer nikdy nespustí limit na žádném modelu.

{% hint style="info" %}
**Funkce „Texture Aware“ na Jetsonu vždy zpracovává jeden obrázek najednou.** Každý pracovní proces by potřeboval vlastní kontext CUDA (~1 GB) plus vlastní kopii modelu odšumovače, což si sjednocená paměť nemůže dovolit — proto je na Jetsonu cesta Texture Aware přiřazena k jedinému pracovnímu procesu se serializovaným přístupem k GPU. Očekávejte, že Texture Aware bude na jakémkoli Jetsonu výrazně pomalejší než Standard. Funkce „Texture Aware“ se vztahuje na snímky z Survey3
: snímky pořízené systémem LATTICE vždy procházejí standardním debayerem. Bez použitelného GPU s podporou CUDA se spustí režim „Standard“ a backend zaznamená jeden řádek `[DEBAYER] … needs a CUDA GPU …` s vysvětlením důvodu (zobrazeno s `chloros-cli --verbose`).
{% endhint %}

***

## Řízení teploty

Zařízení Jetson mají omezenou tepelnou rezervu, zejména v uzavřených prostorech nebo při nasazení ve vzduchu. Chloros
monitoruje teplotu SoC a automaticky omezuje velikost dávek:

| Teplota         | Akce                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normální provoz — plná výpočetní rychlost          |
| **70 °C** (Varování)  | Velikost dávky se postupně snižuje (100 % → 50 % v rozmezí 70 °C až 80 °C) |
| **80 °C** (Kritické) | Agresivní omezení výkonu (50 % → 0 % v rozmezí 80 °C až 90 °C) |
| **90 °C** (Vypnutí) | Úplné zastavení zpracování GPU — nutné ochlazení |

{% hint style="warning" %}
**Zajistěte dostatečné větrání a odvod tepla** pro nepřetržité zpracování, zejména v uzavřených terénních skříních nebo leteckých systémech. Tepelné omezování sníží výpočetní výkon, aby se ochránil hardware.
{% endhint %}

***

## Správa paměti

Zařízení Jetson používají **unifikovanou paměť** — GPU a CPU sdílejí stejnou fyzickou RAM. Uváděná VRAM (např. ~15,3 GB u Orin NX 16 GB) není vyhrazená paměť GPU; jedná se o stejnou RAM, kterou používá operační systém a všechny ostatní procesy.

### Varování a doporučení ohledně swapování

Před zpracováním na zařízení Jetson CLI
spočítá počet RAW snímků ve vaší vstupní složce (`.tif`, `.tiff`, `.raw`, `.dng` — náhledy JPG se nepočítají), odhadne maximální paměťovou náročnost úlohy a **před spuštěním vás varuje**, pokud je pravděpodobné, že RAM + swapová paměť nebudou stačit. Varování má záhlaví `LOW MEMORY WARNING - Jetson Detected`, vypíše počet vašich obrázků, velikost RAM, aktuální velikost odkládacího prostoru a odhadovanou maximální spotřebu, a poté vydá přesné příkazy `fallocate` / `chmod` / `mkswap` / `swapon` přizpůsobené velikosti váš projekt (nikdy menší než 8 GB). Na několik sekund se pozastaví, aby se zpráva neztratila v historii, a poté pokračuje ve zpracování.**Odhady paměti použité ve varování:**

| Režim debayeringu | Základní | Na obrázek |
| --- | --- | --- |
| Standardní | ~1,5 GB | ~10 MB |
| S ohledem na textury | ~2,5 GB (model + runtime Pythonu) | ~15 MB |

Varování se spustí, když odhadovaná špičková hodnota překročí RAM + swap minus bezpečnostní rezervu 1 GB, přičemž se počítá pouze **swap založený na souborech** — konfigurace využívající pouze zram bude i tak označena.

Chcete-li ručně přidat swap (příklad: 8 GB):

```bash
# Check current memory and swap
free -h

# Create a swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->


### Zpracování OOM (Out of Memory)

Během zpracování Chloros
monitoruje paměť GPU a místo pádu systému plynule snižuje výkon:

1. Jakmile využití paměti GPU překročí **85 %**, velikosti dávek se preventivně zmenší
2. Pokud i přesto dojde k události nedostatku paměti, velikost dávky se **sníží na polovinu** a při každém dalším výskytu OOM se opět sníží na polovinu; každá následující úspěšně zpracovaná dávka tuto penalizaci posune o jeden krok zpět
3. Při dlouhodobém přetížení se pipeline přepne z cesty `fused_gpu` na paměťově úspornou cestu `tiled_gpu` a jako poslední možnost na zpracování na CPU

Vzhledem k tomu, že GPU sdílí RAM zařízení Jetson, běh je stále omezen velikostí RAM a odkládacího prostoru — výše uvedené varování ohledně odkládacího prostoru je tedy varováním, které je třeba brát vážně.

***

## Nasazení v terénu

### Úvahy ohledně spotřeby energie

| Model Jetson     | Typický příkon | Poznámky                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7–15 W              | DC konektor typu „barrel“          |
| Jetson Orin NX   | 10–25 W             | DC konektor typu „barrel“          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD nebo konektor typu „barrel“ |

Naplánujte si energetický rozpočet pro nepřetržité zpracování — špičkový odběr energie nastává během operace Thread 3 (Processing), která intenzivně využívá GPU.

### Doporučení ohledně úložiště

* **NVMe SSD** se důrazně doporučuje pro nasazení na platformě arm64
* SD karty jsou pro zpracování příliš pomalé — používejte je pouze jako spouštěcí médium
* Počítejte s 2–3násobkem velikosti surových obrazových dat pro zpracovaný výstup

### Provoz bez grafického rozhraní přes SSH


Chloros
je ideální pro nasazení Jetsonu bez grafického rozhraní:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Neustále spuštěný backend pro synchronizaci času LATTICE / DAQ-E

Pokud váš Jetson bezmonitorově ovládá kamery LATTICE nebo světelné senzory DAQ-E, povolte backendovou službu systemd **pod vaším uživatelským účtem**, aby grandmaster PTP běžel nepřetržitě a skripty SDK
našly spuštěný backend (jednotka je nainstalována, ale ve výchozím nastavení není povolena):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
chloros-cli time-sync status
```

Backend pak běží pod vaším účtem: záznamy, které zapisuje, patří vám a používá váš `chloros-cli login`. Podrobnosti, včetně souboru root `chloros-backend.service` a toho, jak balíček umožňuje vázání portů PTP 319/320 bez oprávnění root, najdete v části [Instalace v systému Linux
](linux-installation.md#always-on-ptp-for-headless-hosts).

### Automatizované zpracování pomocí systemd

Vytvořte službu systemd pro automatizované zpracování:

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

`chloros-cli process` ukončí běh s nenulovým kódem, pokud běh, který požadoval výstupy, nezapíše žádné snímky, takže stav selhání v systemd má význam pro monitorování.

Pro naplánované zpracování zkombinujte se systémovým časovačem systemd:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## Příkladové pracovní postupy

### Základní zpracování na Jetsonu

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI GNDVI OSAVI
```

### Python
SDK
na Jetsonu

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Dávkové zpracování více letů

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI OSAVI
    echo "Completed $name"
done
```

***

## Doporučené systémy Jetson pro použití v terénu

Pro nasazení v terénu a ve vzduchu zvažte tyto varianty nosných desek Jetson Orin NX 16 GB:

* **Ve vzduchu/drony**: Systémy odolné proti vibracím (MIL-STD), lehké (do 300 g), s pasivním chlazením
* **Náročné terénní podmínky**: Vodotěsné kryty s krytím IP67/IP69K a připojením kamery přes PoE GigE
* **Minimální/cenově výhodné**: Vývojářské sady s přídavnými kryty

Obraťte se na [podporu MAPIR
](https://www.mapir.camera/community/contact) a získejte konkrétní doporučení ohledně hardwaru pro váš scénář nasazení.

***

## Další kroky

* [Instalace Linux
u](linux-installation.md) — Obecné podrobnosti k instalaci Linux
u
* [Dynamická adaptace výpočetního výkonu](../processing-architecture/dynamic-compute-adaptation.md) — Kompletní přehled strategií výpočetního výkonu
* [Zpracovací pipeline](../processing-architecture/processing-pipeline.md) — Vysvětlení 4-vláknového pipeline
* [CLI
: Příkazový řádek](../CLI.md) — Průvodce CLI

* [API
: Python
SDK
](../api-python-sdk.md) — Průvodce SDK

* [Referenční příručka k CLI
](../reference/cli-reference.md) a [Referenční příručka k SDK
](../reference/sdk-reference.md) — Úplný seznam příkazů a API
pro verzi 1.3.0
