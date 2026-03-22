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
| **Grafická karta**    | Kompatibilní s DirectX 11                                | GPU NVIDIA s 4 GB+ VRAM                            |
| **Úložiště**          | 6 GB volného místa                                       | SSD s 10 GB+ volného místa                            |
| **Displej**          | 1920x1080                                            | 2560x1440 nebo vyšší                                  |
| **Internet**         | Vyžadováno pro \[volitelnou] aktivaci licence Chloros+ | Vyžadováno pro \[volitelnou] aktivaci licence Chloros+ |

#### Linux amd64 (x86\_64)

| Požadavky       | Minimální                    | Doporučené               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribuce**  | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+             |
| **Procesor**     | x86\_64 (Intel/AMD)        | Intel Core i7 nebo lepší   |
| **Paměť (RAM)**  | 8 GB                        | 16 GB nebo více              |
| **Grafická karta** | Žádná (zpracování CPU)      | GPU NVIDIA s 4 GB+ VRAM |
| **Úložiště**       | 2 GB volného místa             | SSD s 10 GB+ volného místa       |
| **Python**        | Python 3.7+ (pro SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Požadavek      | Minimální                      | Doporučené                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platforma**     | NVIDIA Jetson s JetPack 6 | Jetson Orin NX 16 GB nebo AGX Orin |
| **Paměť (RAM)** | 8 GB (sdílená GPU/CPU)         | 16 GB+ sdílená                    |
| **Úložiště**      | 2 GB volného místa               | NVMe SSD s 10 GB+ volného místa        |
| **Python**       | Python 3.7+ (pro SDK)        | Python 3.10+                    |

{% hint style="info" %}
**Akcelerace GPU**: Uživatelé Chloros+ s grafickými kartami NVIDIA mohou využít akceleraci CUDA pro výrazně rychlejší zpracování. Funguje to jak na Windows (desktopové grafické karty), tak na Linux (desktopové grafické karty a NVIDIA Jetson). Uživatelé Chloros+ získávají také vícevláknové zpracování pro maximální rychlost.
{% endhint %}

***

## Stáhnout Chloros

### Nejnovější stabilní verze (23. března 2026): Verze 1.1.0

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Stáhnout Chloros pro Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Stáhnout Chloros pro Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Stáhnout Chloros pro Linux arm64 / Jetson (.deb)</a>

#### Instalační program Windows (GUI + CLI + Backend)

* **Typ souboru**: .exe (instalační program Windows)**Kroky instalace:**

1. Stáhněte si výše uvedený soubor .exe
2. Poklepejte na instalační program a spusťte instalaci
3. Postupujte podle pokynů instalačního průvodce
4. Vyberte instalační adresář (výchozí: `C:\Program Files\[USER]\Chloros\`)
5. Dokončete instalaci a spusťte Chloros nebo Chloros CLI
6. Přihlaste se pomocí svého [účtu MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (nebo pokračujte s bezplatnou verzí)

{% hint style="success" %}
Instalační program automaticky přidá `chloros-cli` do systémové proměnné PATH pro přístup z příkazového řádku.
{% endhint %}

#### Linux amd64 (balíček .deb — CLI + backend)

* **Typ souboru**: .deb (balíček Debian/Ubuntu)
* **Architektura**: x86\_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (balíček .deb — CLI + backend)

* **Typ souboru**: .deb (JetPack 6)
* **Architektura**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Podrobné pokyny k nastavení najdete v části [Instalace Linux](linux/linux-installation.md) a pokyny specifické pro Jetson v [Průvodci NVIDIA Jetson](linux/nvidia-jetson-guide.md).

#### Python SDK (všechny platformy)

```bash
pip install chloros-sdk
```

Dokumentaci najdete v [API : Python SDK](api-python-sdk.md).

{% hint style="info" %}
**Uživatelé Linux**: Balíček `.deb` nainstaluje CLI a backend. Python a SDK se instalují samostatně přes pip. Pro Linux neexistuje žádné grafické uživatelské rozhraní — veškerá interakce probíhá přes CLI nebo SDK.
{% endhint %}

***

## Další zdroje

### Python SDK

Pro vývojáře a automatizační pracovní postupy nainstalujte Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentace**: [API: Python SDK](api-python-sdk.md)**Požadavky**: Musí být nainstalován Chloros (instalační program Windows nebo balíček Linux `.deb`), je vyžadováno přihlášení k licenci Chloros+***

## Co je součástí balení

### Instalační program Windows

* ✅ **Chloros GUI** – Plnohodnotné grafické rozhraní
* ✅ **Chloros CLI** – rozhraní příkazového řádku (vyžaduje licenci Chloros+)
* ✅ **Chloros Backend** – zpracovatelský engine
* ✅ **Profily kamer** – Předkonfigurované šablony kamer MAPIR

### Balíček Linux .deb

* ✅ **Chloros CLI** - Rozhraní příkazového řádku (vyžaduje licenci Chloros+)
* ✅ **Chloros Backend** – Zpracovací engine
* ✅ **Profily kamer** – Předkonfigurované šablony kamer MAPIR
* ❌ Bez grafického uživatelského rozhraní — Linux je pouze bezhlavý CLI/SDK

### Python SDK (pip, všechny platformy)

* ✅ **Chloros SDK** - Python API (vyžaduje licenci Chloros+)***

## Upgrade na Chloros+

Odemkněte pokročilé funkce s předplatným Chloros+:

* 🚀 **Vícevláknové zpracování** – Zpracovávejte obrázky paralelně
* ⚡ **Akcelerace GPU (CUDA)** – Využijte výkon GPU NVIDIA
* 💻 **Přístup k CLI** – Automatizujte pomocí nástrojů příkazového řádku
* 🐍 **Python SDK** – Programový přístup k API
* 📱 **Více zařízení** – Použití na 2–10+ zařízeních (v závislosti na tarifu)
* **🐻 Pokročilá metoda debayeringu s rozpoznáním textur** – vysoce kvalitní debayering s rozpoznáním hran v kombinaci s modelem odšumování AI/ML, který odstraní téměř veškerý šum způsobený debayeringem.
* 🧮 **Vlastní vzorce** – Vytvářejte vlastní multispektrální indexy

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Zobrazit tarify a ceny Chloros+</a></p>***

## Nápověda k instalaci

### Řešení problémů

**Instalace selže s chybovou zprávou:**

* Ujistěte se, že máte práva správce
* Dočasně deaktivujte antivirový software
* Zkontrolujte, zda splňujete minimální systémové požadavky

**Aplikace se nespustí (Windows):**

* Ověřte, zda je nainstalován Windows 10/11 (64bitová verze)
* Aktualizujte grafické ovladače
* Zkontrolujte podrobnosti o chybě v Prohlížeči událostí Windows
* Kontaktujte podporu s protokoly chyb

**CLI se nespustí (Linux):**

* Ověřte, zda je balíček `.deb` nainstalován správně: `dpkg -l | grep chloros`
* Zkontrolujte oprávnění: `sudo chmod +x /usr/bin/chloros-cli`
* Spusťte diagnostiku: `chloros-cli selftest`
* Zkontrolujte, zda nechybí nějaké knihovny: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Problémy s aktivací licence:**

* Ujistěte se, že je aktivní připojení k internetu
* Ověřte přihlašovací údaje na [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Zkontrolujte, zda firewall neblokuje Chloros
* Podrobné pokyny najdete na [Chloros+ Přihlášení](chloros+-login.md)

### Získání podpory

Potřebujete pomoc s instalací nebo nastavením?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Webové stránky**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentace**: [Začínáme](./)
* ❓ **FAQ**: [Často kladené otázky](faq.md)***

## Seznam změn

<details>

<summary>Verze 1.1.0 (nejnovější)</summary>

**Datum vydání: březen 2026**

**Nové funkce*** **Podpora Linux** — Nativní CLI a SDK pro Linux amd64 (x86\_64) a arm64 (NVIDIA Jetson JetPack 6). Instalace prostřednictvím balíčků `.deb`.
* **Podpora NVIDIA Jetson** — Optimalizované zpracování pro zařízení Jetson Nano, Orin Nano, Orin NX a AGX Orin.
* **Dynamická adaptace výpočtů** — Automatická detekce hardwaru a optimalizace strategie zpracování. Chloros se přizpůsobí vašemu hardwaru od Jetson Nano až po pracovní stanici s více GPU.
* **4-vláknový zpracovatelský pipeline** — Souběžné vlákna detekce, kalibrace, zpracování a exportu s dynamickým přidělováním paměti GPU.
* **Nové příkazy CLI** — `selftest` (systémová diagnostika) a `update` (správa aktualizací Linux).
* **Nové příznaky procesu CLI** — `--debayer` (standardní/s podporou textur), `--indices` (určit indexy), `--target` (pro rychlejší detekci nejprve prohledat cílovou podsložku).
* **Nové položky v nabídce GUI** — Přidat soubory, Přidat složku a Spustit/Zastavit zpracování jsou nyní dostupné z rozbalovací nabídky hlavního menu.**Vylepšení**

* Automatická detekce backendu napříč platformami (cesty Windows a Linux)
* Vylepšené SDK `get_status()` se sledováním průběhu pro každý vlákno
* Nové výjimky SDK: `ChlorosConfigurationError`, `ChlorosAuthenticationError`
* Řízení teploty a adaptivní omezení výkonu pro NVIDIA Jetson
* Automatická správa paměti s přepnutím na dlaždicové zpracování GPU v případě nedostatku paměti (OOM)

</details>

<details>

<summary>Verze 1.0.5</summary>

**Datum vydání: 10. února 2026**

**Nové funkce*** **Metoda debayeringu s podporou textur \[Pouze Chloros+] -** Metoda s podporou textur využívá vysoce kvalitní debayering s rozpoznáním hran v kombinaci s modelem odšumování AI/ML, který odstraní téměř veškerý šum způsobený debayeringem.
* **Podpora kalibračních terčů T4P*** **Rychlejší zpracování na GPU Chloros+, lepší správa paměti**

**Opravy chyb*** Zcela nové uživatelské rozhraní (GUI), které by nyní mělo fungovat na všech počítačích Windows.

</details>

<details>

<summary>Verze 1.0.4</summary>

**Datum vydání: 5. ledna 2026**

**Nové funkce*** **Přepínání obrázků/metadat**: Do prohlížeče souborů bylo přidáno přepínání pro zobrazení metadat vybraného obrázku v tabulce namísto mřížky obrázků
* **Posuvník pro přiblížení mřížky obrázků**: Nový posuvník v uživatelském rozhraní pro úpravu velikosti miniatur (podporuje také klávesu CTRL + kolečko myši)
* **Tlačítka pro export mřížky obrázků**: Tlačítka v horním řádku pro přepnutí miniatur z formátu JPG na zpracované exporty (cíle, odrazivost, index, LUT)
* **Záložka Mapa**: Nová interaktivní 2D mapa zobrazující značky GPS polohy obrázků
  * Podporuje mapové dlaždice Google Maps a ESRI (automaticky vybírá nejlepší službu dlaždic na základě dostupnosti úrovně přiblížení)
  * Náhled miniatury při najetí myší na značky na mapě

**Opravy chyb*** Vylepšená podpora instalace Chloros na počítačích s jiným než anglickým jazykem

</details>

<details>

<summary>Verze 1.0.3</summary>

**Datum vydání: 20. prosince 2025**

**Nové funkce*** První vydání

**Vylepšení*** První vydání

**Opravy chyb*** První vydání

**Známé problémy*** První vydání

</details>***

## Licenční smlouva**Proprietární software** – Copyright (c) 2026 MAPIR Inc.

Neoprávněné použití, distribuce nebo úpravy jsou zakázány.

**Bezplatná verze**: K dispozici pro osobní i komerční použití s omezenými funkcemi**Chloros+**: Licence na základě předplatného pro pokročilé funkce a komerční nasazení
