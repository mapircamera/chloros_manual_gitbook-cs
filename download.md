---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Stažení

Stáhněte si nejnovější verzi programu Chloros a začněte se zpracováním multispektrálních snímků.

### Systémové požadavky

#### Windows


| Požadavek          | Minimální                                              | Doporučené                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Operační systém** | Windows 10 (64bitový)                                  | Windows 11 (64bitový)                                  |
| **Procesor**        | Intel Core i5 nebo ekvivalent                          | Intel Core i7 nebo lepší                              |
| **Paměť (RAM)**     | 8 GB                                                  | 16 GB nebo více                                         |
| **Grafická karta**    | Kompatibilní s DirectX 11                                | Grafická karta NVIDIA s 4 GB+ VRAM                            |
| **Úložiště**          | 6 GB volného místa                                       | SSD s 10 GB+ volného místa                            |
| **Displej**          | 1920x1080                                            | 2560x1440 nebo vyšší                                  |
| **Internet**         | Vyžadováno pro [volitelnou] aktivaci licence Chloros+ | Vyžadováno pro [volitelnou] aktivaci licence Chloros+ |

#### Linux amd64 (x86_64)

| Požadavky       | Minimální                    | Doporučené               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribuce**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Procesor**     | x86\_64 (Intel/AMD)        | Intel Core i7 nebo lepší   |
| **Paměť (RAM)**  | 8 GB                        | 16 GB nebo více              |
| **Grafická karta** | Žádná (zpracování na CPU)      | Grafická karta NVIDIA s 4 GB+ VRAM |
| **Úložiště**       | 2 GB volného místa             | SSD s 10 GB+ volného místa       |
| **Python**        | Python 3.7+ (pro SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Požadavky      | Minimální                      | Doporučené                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platforma**     | NVIDIA Jetson s JetPack 6 | Jetson Orin NX 16 GB nebo AGX Orin |
| **Paměť (RAM)** | 8 GB (sdílená mezi GPU a CPU)         | 16 GB+ sdílená                    |
| **Úložiště**      | 2 GB volného místa               | NVMe SSD s 10 GB+ volného místa        |
| **Python**       | Python 3.7+ (pro SDK)        | Python 3.10+                    |

{% hint style="info" %}
**Akcelerace GPU**: Uživatelé Chloros+ s grafickými kartami NVIDIA mohou využít akceleraci CUDA pro výrazně rychlejší zpracování. Tato funkce je k dispozici u stolních grafických karet NVIDIA v systémech Windows a Linux amd64 a na platformě NVIDIA Jetson s JetPack 6. Uživatelé Chloros+ navíc získávají multithreadové zpracování pro maximální rychlost.
{% endhint %}

***

## Stáhnout Chloros


### Nejnovější stabilní verze: Verze 1.3.0 (11. listopadu 2026)

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Stažení Chloros pro Windows (.exe)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Stažení Chloros pro Linux amd64 (.deb)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Stáhnout Chloros pro Linux arm64 / Jetson (.deb)</a>


#### Instalační program pro Windows (GUI + CLI + backend)

* **Typ souboru**: .exe (instalační program pro Windows)

**Postup instalace:**

1. Stáhněte si výše uvedený soubor .exe
2. Poklepejte na instalační program a spusťte instalaci
3. Postupujte podle pokynů instalačního průvodce
4. Vyberte instalační adresář (výchozí: `C:\Program Files\Chloros\`)
5. Na stránce **Vybrat další úkoly**nechte zaškrtnuté obě možnosti**Systémová nastavení** (doporučeno – viz níže)
6. Dokončete instalaci a spusťte Chloros nebo Chloros CLI
7. Přihlaste se pomocí svého [účtu MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (nebo pokračujte s bezplatnou verzí)

**Možnosti systémových nastavení** (obě jsou ve výchozím nastavení zaškrtnuté):

| Možnost, jak je uvedena v instalačním programu | Co dělá | Vypnutí tiché instalace |
| --- | --- | --- |
| **Povolit dlouhé cesty v systému Windows** (umožňuje, aby složky projektů měly názvy delší než 260 znaků; doporučeno) | Nastaví systémové nastavení `LongPathsEnabled` pro celý počítač v systému Windows na hodnotu 1. Windows se dodává s touto funkcí vypnutou, a proto žádná cesta k souboru nesmí být delší než 260 znaků; produkty Chloros se nacházejí až asi 130 znaků pod projektovou složkou, takže při vypnutých dlouhých cestách je hlubší projektová složka odmítnuta. Nastavení se projeví u programů spuštěných po instalaci. Po odinstalaci zůstává zapnuté, protože na něm mohou záviset jiné programy. | `/MERGETASKS="!longpaths"` |
| **Vyloučit backend Chloros u ze skenování Microsoft Defenderem** (mnohem rychlejší snímání z kamery; doporučeno) | Přidá vyloučení procesu Microsoft Defenderu pro `chloros-backend.exe` ve složce instalace, takže ochrana v reálném čase neskenuje každý soubor, který backend zapíše, ať už jej zapíše kamkoli. Při vysokých rychlostech snímání činí náklady na skenování přibližně 30 % rychlosti snímání (77,7 oproti 52,7 snímkům za sekundu při nepřetržitém snímání z 9 kamer). Žádný jiný program není vyloučen a vyloučení se při odinstalaci odstraní. Pokud Defender chybí nebo je zablokován zásadami, tento krok tiše selže a instalace pokračuje. | `/MERGETASKS="!defenderexclusion"` |

Tichá instalace (`/SILENT` nebo `/VERYSILENT`) použije obě možnosti, pokud se z toho neodhlásíte pomocí zobrazené hodnoty `/MERGETASKS`.

{% hint style="success" %}
Instalační program přidá složku CLI (`C:\Program Files\Chloros\cli\`) do vašeho systému `PATH` pouze jednou — při přeinstalaci nebo aktualizaci již nebude přidán duplicitní záznam a budou odstraněny duplicity přidané dřívějšími instalačními programy. Po instalaci otevřete nový terminál, abyste mohli používat `chloros-cli`. Instalační program také přidá dvě pravidla brány Windows Firewall pro backend do každého síťového profilu: **Chloros Backend (UDP-In)**, pro datové toky z kamer, PTP, mDNS a datový tok DAQ-E, a**Chloros Backend (TCP-In)**, omezené na místní podsíť, pro aktualizaci firmwaru DAQ-E, při které se senzor připojuje zpět k tomuto počítači za účelem sběru obrazu. Oba jsou při odinstalaci odstraněny.
{% endhint %}

#### Linux amd64 (balíček .deb — CLI + backend)

* **Typ souboru**: .deb (balíček pro Debian/Ubuntu)
* **Architektura**: x86_64 (amd64)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

#### Linux arm64 — NVIDIA Jetson (balíček .deb — CLI + backend)

* **Typ souboru**: .deb (JetPack 6)
* **Architektura**: aarch64 (arm64)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

Použijte název souboru, který jste skutečně stáhli, a ponechte `./`, aby `apt` nainstaloval právě tento soubor, místo aby hledal balíček podle názvu. Nainstalujte pomocí `apt install`, nikoli pomocí `dpkg -i`: apt nainstaluje také požadavky Python SDK, které balíček doporučuje, nikoli vyžaduje, a `dpkg -i` je přeskočí. Od verze 1.3.0 se backend a CLI instalují v rozbaleném stavu pod `/usr/lib/chloros/`, takže se spouštějí bez toho, aby se nejprve samy rozbalily.

Balíček pro Jetson používá verzi PyTorch pro CUDA z JetPacku 6, takže zpracování, včetně debayera Texture Aware, probíhá na GPU Jetsonu. Využívá knihovny CUDA a cuDNN, které nainstaluje JetPack 6 (`sudo apt install nvidia-jetpack` na obrazu, kde tyto knihovny chybí).

Podrobné pokyny k nastavení najdete v části [Instalace na Linux u](linux/linux-installation.md) a pokyny specifické pro Jetson v [Průvodci NVIDIA Jetson](linux/nvidia-jetson-guide.md).

#### Python SDK (všechny platformy)

Každý instalační program obsahuje odpovídající balíček `chloros_sdk`, takže verze SDK vždy odpovídá nainstalovanému grafickému rozhraní (GUI), příkazové řádce (CLI) nebo backendu. Ve Windows u jej instalační program automaticky nainstaluje do systémového Python u. V systému Linux nainstaluje balíček `.deb` SDK pro samotný systém `python3` (do `/usr/lib/python3/dist-packages`), takže `import chloros_sdk` funguje ihned po instalaci, bez pip a bez připojení k síti. Pouze do virtuálního prostředí (virtualenv), které nevidí systémové balíčky, je třeba nainstalovat přiložený balíček typu wheel:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Pro hostitele používající pouze pip (bez nainstalovaného balíčku Chloros; SDK pak řídí backend na jiném stroji) je SDK také k dispozici na PyPI:

```bash
pip install chloros-sdk
```

Dokumentaci najdete v [API: Python SDK](api-python-sdk.md) a v [SDK Reference](reference/sdk-reference.md).

{% hint style="info" %}
**Uživatelé Linux u**: Balíček `.deb` nainstaluje CLI, backend a Python SDK. Pro Linux neexistuje žádné grafické uživatelské rozhraní (GUI) — veškerá interakce probíhá přes CLI nebo SDK.
{% endhint %}

***

## Další zdroje

### Python SDK


Pro vývojáře a automatizační pracovní postupy je součástí každého instalačního balíčku Chloros Python SDK: instalační program pro Windows jej nainstaluje do vašeho Python u a balíček `.deb` pro Linux jej nainstaluje pro systém `python3`. Na počítači bez balíčku Chloros (který řídí backend na jiném počítači) jej nainstalujte z PyPI:

```bash
pip install chloros-sdk
```

**Dokumentace**: [API: Python SDK](api-python-sdk.md)

**Požadavky**: backend Chloros, který bude řídit (instaluje jej instalační program pro Windows nebo balíček `.deb` pro Linux), a přihlašovací údaje k Chloros+ v placené verzi

***

## Co je součástí balení

### Instalační program pro Windows


* ✅ **GUI Chloros** – plně funkční grafické rozhraní
* ✅ **CLI Chloros** – rozhraní příkazového řádku (vyžaduje licenci Chloros+)
* ✅ **Backend Chloros** – zpracovatelský engine
* ✅ **Python SDK** – nainstaluje se do vašeho Python u, pokud je nainstalován (vyžaduje licenci Chloros+)
* ✅ **Profily kamer** – předkonfigurované šablony kamer MAPIR


### Balíček .deb pro Linux


* ✅ **Chloros CLI** – rozhraní příkazového řádku (vyžaduje licenci Chloros+)
* ✅ **Chloros Backend** – zpracovatelský engine
* ✅ **Python SDK** – nainstalováno pro systém `python3` (vyžaduje licenci Chloros+)
* ✅ **Profily kamer** – předkonfigurované šablony kamer MAPIR

* ❌ Bez grafického rozhraní — Linux je bez grafického rozhraní, pouze CLI/SDK


### Python SDK (PyPI, pro stroje bez balíčku Chloros)

* ✅ **Chloros SDK** – Python API (vyžaduje licenci Chloros+)

***

## Přechod na Chloros+

Odemkněte pokročilé funkce předplatným Chloros+:

* 🚀 **Vícevláknové zpracování** – paralelní zpracování snímků
* ⚡ **Akcelerace GPU (CUDA)** – Využijte výkon GPU NVIDIA
* 💻 **Přístup přes CLI** – Automatizujte pomocí nástrojů příkazového řádku
* 🐍 **Python SDK** – Programový přístup k API

* 📱 **Více zařízení** – Používejte na 2–10+ zařízeních (v závislosti na tarifu)
* **🐻 Pokročilá metoda debayeringu s ohledem na textury** – vysoce kvalitní debayering s ohledem na hrany v kombinaci s modelem odšumování založeným na AI/ML, který odstraňuje téměř veškerý šum vznikající při debayeringu.
* 🧮 **Vlastní vzorce** – vytvářejte vlastní multispektrální indexy

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Zobrazit tarify a ceny Chloros+</a></p>***

## Nápověda k instalaci

### Řešení problémů

**Instalace selže s chybovou zprávou:**

* Ujistěte se, že máte práva správce
* Dočasně deaktivujte antivirový software
* Zkontrolujte, zda splňujete minimální systémové požadavky

**Aplikace se nespustí (systém Windows):**

* Ověřte, zda máte nainstalovaný Windows 10/11 (64bitová verze)
* Aktualizujte grafické ovladače
* Podrobnosti o chybě zkontrolujte v Prohlížeči událostí systému Windows


* Kontaktujte podporu s protokoly chyb

**CLI se nespustí (Linux):**

* Ověřte, zda je balíček `.deb` správně nainstalován: `dpkg -l | grep chloros`
* Zkontrolujte oprávnění: `sudo chmod +x /usr/bin/chloros-cli`
* Spusťte diagnostiku: `chloros-cli selftest`
* Zkontrolujte, zda nechybí nějaké knihovny: `ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"` (od verze 1.3.0 je `/usr/lib/chloros/chloros-backend` malý spouštěcí skript; samotný program se nachází v `/usr/lib/chloros/backend/`)

**Problémy s aktivací licence:**

* Ujistěte se, že máte aktivní připojení k internetu
* Ověřte přihlašovací údaje na [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Zkontrolujte, zda firewall neblokuje Chloros

* Podrobné pokyny najdete na stránce [Chloros+ Login](chloros+-login.md)

### Podpora

Potřebujete pomoc s instalací nebo nastavením?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Webové stránky**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentace**: [Začínáme](./)
* ❓ **Často kladené otázky**: [Často kladené otázky](faq.md)***

## Aktualizace softwaru

Chloros kontroluje dostupnost aktualizací, oznamuje, když je k dispozici nová verze, a odkazuje na tuto stránku pro stažení — aktualizaci provedete spuštěním nového podepsaného instalačního programu. Vaše nastavení, projekty a přihlašovací údaje zůstanou po aktualizaci zachovány. V systému Linux a na Jetsonu program `chloros-cli update` zkontroluje, zda je k dispozici novější verze, a nabídne stažení a instalaci odpovídajícího balíčku `.deb` (tento příkaz je k dispozici pouze v systému Linux).

***

## Seznam změn**Verze 1.3.0 (nejnovější, 11. listopadu 2026)**— úplný seznam najdete v části**Co je nového v Chloros

u 1.3.0** na stránce [Začínáme](./).**Verze 1.2.0**— viz**Co je nového v Chloros u 1.2.0** na stránce [Začínáme](./).

<details>

<summary>Verze 1.0.5</summary>

**Datum vydání: 10. února 2026**

**Nové funkce*** **Metoda debayeringu s ohledem na textury \[pouze Chloros+] –** Metoda s ohledem na textury využívá vysoce kvalitní debayering s ohledem na hrany v kombinaci s modelem odšumování založeným na AI/ML, který odstraňuje téměř veškerý šum vznikající při debayeringu.
* **Podpora kalibračních terčů T4P*** **Rychlejší zpracování na GPU v Chloros+, lepší správa paměti**

**Opravy chyb*** Zcela nové uživatelské rozhraní (GUI), které by nyní mělo fungovat na všech počítačích s operačním systémem Windows.

</details>

<details>

<summary>Verze 1.0.4</summary>

**Datum vydání: 5. ledna 2026**

**Nové funkce*** **Přepínání mezi obrázky a metadaty**: Do prohlížeče souborů bylo přidáno přepínání, které umožňuje zobrazit metadata vybraného obrázku v tabulce namísto mřížky obrázků
* **Posuvník pro změnu měřítka mřížky obrázků**: Nový posuvník v uživatelském rozhraní pro úpravu velikosti miniatur (podporuje také klávesovou zkratku CTRL + kolečko myši)
* **Tlačítka pro export mřížky obrázků**: Tlačítka v horním řádku pro přepnutí miniatur z formátu JPG na zpracované výstupy (Targets, Reflectance, Index, LUT)
* **Záložka Mapa**: Nová interaktivní 2D mapa zobrazující značky GPS polohy obrázků
  * Podporuje mapové dlaždice Google Maps a ESRI (automaticky vybírá nejlepší službu dlaždic na základě dostupnosti úrovně přiblížení)
  * Náhled miniatury při najetí myší na značky na mapě

**Opravy chyb*** Vylepšená podpora instalace aplikace Chloros na počítačích s jiným než anglickým jazykem

</details>

<details>

<summary>Verze 1.0.3</summary>

**Datum vydání: 20. prosince 2025**

**Nové funkce*** První vydání

**Vylepšení*** První vydání

**Opravy chyb*** První vydání

**Známé problémy*** První vydání

</details>

***

## Licenční smlouva**Proprietární software** – Copyright (c) 2026 MAPIR Inc.

Neoprávněné použití, šíření nebo úpravy jsou zakázány.

**Bezplatná verze**: K dispozici pro osobní i komerční použití s omezenými funkcemi**Chloros+**: Licence na základě předplatného pro pokročilé funkce a komerční nasazení
