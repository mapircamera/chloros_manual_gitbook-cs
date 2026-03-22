# Průvodce NVIDIA Jetson

Chloros na platformě NVIDIA Jetson umožňuje zpracování multispektrálních snímků na okraji sítě – v terénu, na bezpilotních letadlech (UAV) a ve vzdálených instalacích. Chloros automaticky rozpozná váš model Jetson a optimalizuje strategii zpracování pro váš hardware.

***

## Podporované modely Jetson

| Model                | RAM            | Strategie zpracování                                   | Doporučené použití                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB sdílené | `GPU_PARALLEL` (4 pracovníky)                            | Maximální výkon, velké datové sady                      |
| **Jetson Orin NX**   | 8–16 GB sdílené  | `GPU_PARALLEL` (3 pracovníky, 16 GB) / `GPU_SINGLE` (8 GB) | Primární doporučení pro nasazení ve vzduchu a v terénu |
| **Jetson Orin Nano** | 8 GB sdílené     | `GPU_SINGLE` (1 pracovník)                               | Základní úroveň výpočetního výkonu na okraji sítě                                 |
| **Jetson Nano**      | 4–8 GB sdílené   | `GPU_SINGLE` (1 pracovník)                               | Základní úroveň, omezená paměť                          |

{% hint style="info" %}
**Starší modely Jetson** (TX2, TX1, Xavier NX) nemusí být podporovány. Výkon se bude lišit v závislosti na dostupné paměti GPU a schopnostech CUDA.
{% endhint %}

***

## Požadavky

* **JetPack 6.x** (doporučena nejnovější verze)
* **NVIDIA CUDA** (součástí balíčku JetPack)
* **Licence Chloros+** (vyžadována pro přístup k CLI/SDK)

## Instalace

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

Obecné podrobnosti o instalaci Linux najdete v části [Instalace Linux](linux-installation.md).

***

## Dynamická adaptace výpočtů na Jetsonu

Chloros automaticky detekuje váš model Jetson a vybere optimální strategii zpracování. **Není nutné žádné ruční ladění.**

### Jak to funguje

Při spuštění Chloros provede profilování vašeho systému:

1. **Detekuje model Jetson** pomocí `/proc/device-tree/model`
2. **Načte dostupnou GPU/sdílenou paměť**

3.**Vybere strategii zpracování** (`GPU_PARALLEL`, `GPU_SINGLE` nebo `CPU_PARALLEL`)
4. **Automaticky nastaví počet pracovníků, typ potrubí a alokaci paměti**

### Chování podle modelu

| Model Jetson                | Strategie       | Pracovníci | Pipeline                       | Souběžnost |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8 GB**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (úsporné využití paměti) | Sériové  |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | Sériové  |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | Serializováno  |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 3       | `fused_gpu` (plná cesta GPU)    | Souběžné  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | Souběžné  |

{% hint style="success" %}
**Jetson Orin NX 16 GB** je ideální volbou pro nasazení na okraji sítě – využívá strategii `GPU_PARALLEL` se 3 souběžnými pracovníky a poskytuje skutečné paralelní zpracování GPU v kompaktním provedení.
{% endhint %}

Klíčovým rozdílem mezi platformami je **paměť**. Jetson Nano s 8 GB sdílené paměti musí zpracovávat obrázky po jednom pomocí paměťově efektivního dlaždicového přístupu, zatímco Orin NX s 16 GB může pomocí fúzovaného potrubí s vyšší propustností zpracovávat 3 obrázky přes GPU současně.

Kompletní referenci k adaptaci výpočetního výkonu najdete v [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md).

***

## Řízení teploty

Zařízení Jetson mají omezenou tepelnou rezervu, zejména v uzavřených nebo leteckých aplikacích. Chloros zahrnuje automatické monitorování teploty a omezování výkonu:

| Teplota         | Akce                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normální provoz — plná rychlost zpracování          |
| **70 °C** (Varování)  | Automatické snížení velikosti dávky                   |
| **80 °C** (Kritické) | Agresivní omezení výkonu — nižší souběžnost         |
| **90 °C** (Vypnutí) | Úplné zastavení zpracování GPU — nutné ochlazení |

{% hint style="warning" %}
**Zajistěte dostatečné větrání a odvod tepla** pro nepřetržité zpracování, zejména v uzavřených terénních skříních nebo leteckých systémech. Tepelné omezení sníží propustnost zpracování, aby ochránilo hardware.
{% endhint %}

***

## Správa paměti

Zařízení Jetson používají **unifikovanou paměť** — GPU a CPU sdílejí stejnou fyzickou RAM. To znamená, že uváděná VRAM (např. 15,3 GB na Orin NX 16 GB) není vyhrazená paměť GPU; je sdílená s operačním systémem a dalšími procesy.

### Doporučení ohledně swapování

U velkých datových sad nebo při zpracování debayerem s podporou textur může Chloros doporučit vytvoření swapového prostoru:

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Odhadované nároky na paměť na jeden obrázek:**

* Standardní debayer: ~10 MB na obrázek
* Texture Aware debayer: ~15 MB na obrázek

Chloros automaticky vypočítá potřebnou paměť na základě velikosti vašeho datového souboru a upozorní vás, pokud je doporučeno vytvoření swapového prostoru.

### Záložní řešení při nedostatku paměti (OOM)

Pokud je během zpracování detekován stav nedostatku paměti:

1. Chloros automaticky sníží počet GPU workerů
2. Přepne se z potrubí `fused_gpu` na potrubí `tiled_gpu` (úspornější z hlediska paměti)
3. Pokračuje ve zpracování se sníženou propustností, místo aby došlo k selhání

***

## Nasazení v terénu

### Úvahy ohledně napájení

| Model Jetson     | Typický odběr energie | Poznámky                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5–10 W              | USB-C nebo konektor typu barrel    |
| Jetson Orin Nano | 7–15 W              | DC konektor typu barrel          |
| Jetson Orin NX   | 10–25 W             | DC konektor typu barrel          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD nebo konektor typu barrel |

Naplánujte si spotřebu energie pro nepřetržité zpracování — špičkový odběr energie nastává během GPU-náročného Thread 3 (zpracování).

### Doporučení pro úložiště

* **NVMe SSD** se důrazně doporučuje pro nasazení arm64
* SD karty jsou pro zpracování příliš pomalé — používejte je pouze jako spouštěcí médium
* Počítejte s 2–3násobkem velikosti surových obrazových dat pro zpracovaný výstup

### Provoz bez monitoru přes SSH

Chloros CLI je ideální pro nasazení Jetsonu bez monitoru:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

### Automatizované zpracování se systemd

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

Spojte se s časovačem systemd pro plánované zpracování:

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

## Příklad pracovních postupů

### Základní zpracování na Jetsonu

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI
```

### Python SDK na Jetsonu

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
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
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Doporučené systémy Jetson pro použití v terénu

Pro nasazení v terénu a ve vzduchu zvažte tyto možnosti nosných desek Jetson Orin NX 16 GB:

* **Ve vzduchu/drony**: Systémy s odolností proti vibracím (MIL-STD), lehké (pod 300 g), pasivní chlazení
* **Náročné terénní podmínky**: Vodotěsné kryty IP67/IP69K s připojením kamery PoE GigE
* **Minimální/cenově výhodné**: Vývojářské sady s přídavnými kryty

Kontaktujte [MAPIR podporu](https://www.mapir.camera/community/contact) ohledně konkrétních doporučení k hardwaru pro váš scénář nasazení.

***

## Další kroky

* [Linux Instalace](linux-installation.md) — Obecné podrobnosti o instalaci Linux
* [Dynamická adaptace výpočetního výkonu](../processing-architecture/dynamic-compute-adaptation.md) — Kompletní reference ke strategii výpočetního výkonu
* [Zpracovací pipeline](../processing-architecture/processing-pipeline.md) — Vysvětlení 4vláknové pipeline
* [CLI : Příkazový řádek](../CLI.md) — Kompletní reference k CLI
* [API : Python SDK](../api-python-sdk.md) — Kompletní reference k SDK
