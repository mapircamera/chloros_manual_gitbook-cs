---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Stažení

Stáhněte si nejnovější verzi programu Chloros a začněte zpracovávat multispektrální snímky.

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

#### Linux amd64 (x86\_64)

| Požadavek       | Minimální                    | Doporučené               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribuce**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Procesor**     | x86\_64 (Intel/AMD)        | Intel Core i7 nebo lepší   |
| **Paměť (RAM)**  | 8 GB                        | 16 GB nebo více              |
| **Grafická karta** | Žádná (zpracování na CPU)      | Grafická karta NVIDIA s 4 GB a více VRAM |
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
**Akcelerace GPU**: Uživatelé Chloros+ s grafickými kartami NVIDIA mohou využít akceleraci CUDA pro výrazně rychlejší zpracování. Funguje to jak na Windows (stolní grafické karty), tak na Linux (stolní grafické karty a NVIDIA Jetson). Uživatelé Chloros+ navíc získávají multithreadové zpracování pro maximální rychlost.
{% endhint %}

***

## Stáhnout Chloros

### Nejnovější stabilní verze: Verze 1.2.0

<!-- NOLAN: replace installer links + release date for 1.2.0 — the three download buttons below still point at the 1.1.0 Google Drive files, and the release date needs to be added to the heading above. -->



### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Stáhnout Chloros pro Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Stáhnout Chloros pro Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Stáhnout Chloros pro Linux arm64 / Jetson (.deb)</a>

#### Instalační program Windows (GUI + CLI + backend)

* **Typ souboru**: .exe (instalační program Windows)**Postup instalace:**

1. Stáhněte si výše uvedený soubor .exe
2. Dvojitým kliknutím na instalační program spusťte instalaci
3. Postupujte podle pokynů instalačního průvodce
4. Vyberte instalační adresář (výchozí: `C:\Program Files\MAPIR\Chloros\`)
5. Dokončete instalaci a spusťte Chloros nebo Chloros CLI
6. Přihlaste se pomocí svého [účtu MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (nebo pokračujte s bezplatnou verzí)

{% hint style="success" %}
Instalační program automaticky přidá `chloros-cli` do systémové proměnné PATH pro přístup z příkazového řádku.
{% endhint %}

#### Linux amd64 (balíček .deb — CLI + backend)

* **Typ souboru**: .deb (balíček pro Debian/Ubuntu)
* **Architektura**: x86_64 (amd64)

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

Podrobné pokyny k instalaci najdete v části [Instalace Linux](linux/linux-installation.md) a pokyny specifické pro Jetson v [Průvodci NVIDIA Jetson](linux/nvidia-jetson-guide.md).

#### Python SDK (všechny platformy)

Každý instalační balíček obsahuje odpovídající `chloros_sdk` wheel, takže verze SDK vždy odpovídá nainstalovanému GUI/CLI/backendu. Na Windows instalační program nainstaluje kolo automaticky do vašeho systému Python; u Linux umístí `.deb` balíček typu „wheel“ do adresáře `/usr/lib/chloros/sdk/` a vypíše instalační příkaz:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Pro hostitele pouze s pip (bez nainstalovaného balíčku Chloros) je SDK k dispozici také na PyPI:

```bash
pip install chloros-sdk
```

Viz [API : Python SDK](api-python-sdk.md) a [SDK Reference](reference/sdk-reference.md) pro dokumentaci.

{% hint style="info" %}
**Uživatelé Linux**: Balíček `.deb` instaluje CLI a backend. Pro Linux neexistuje grafické uživatelské rozhraní — veškerá interakce probíhá prostřednictvím CLI nebo SDK.
{% endhint %}

***

## Další zdroje

### Python SDK

Pro vývojáře a automatizační pracovní postupy nainstalujte Chloros, Python a SDK:

```bash
pip install chloros-sdk
```

**Dokumentace**: [API: Python SDK](api-python-sdk.md)**Požadavky**: Musí být nainstalován Chloros (instalační program Windows nebo balíček Linux `.deb`), je vyžadováno přihlášení k licenci Chloros+***

## Co je součástí balení

### Instalační program Windows

* ✅ **Chloros GUI** – Plně funkční grafické rozhraní
* ✅ **Chloros CLI** – Rozhraní příkazového řádku (vyžaduje licenci Chloros+)
* ✅ **Chloros Backend** – Zpracovací jádro
* ✅ **Profily kamer** – Předkonfigurované šablony kamer MAPIR

### Balíček .deb Linux

* ✅ **Chloros CLI** – rozhraní příkazového řádku (vyžaduje licenci Chloros+)
* ✅ **Chloros Backend** – Zpracovací engine
* ✅ **Profily kamer** – Předkonfigurované šablony kamer MAPIR
* ❌ Bez grafického uživatelského rozhraní — Linux je pouze bez grafického rozhraní CLI/SDK

### Python SDK (pip, všechny platformy)

* ✅ **Chloros SDK** – Python API (vyžaduje licenci Chloros+)***

## Upgrade na Chloros+

Odemkněte pokročilé funkce s předplatným Chloros+:

* 🚀 **Vícevláknové zpracování** – Zpracovávejte obrázky paralelně
* ⚡ **Akcelerace pomocí GPU (CUDA)** – Využijte výkon grafických karet NVIDIA
* 💻 **Přístup k CLI** – Automatizujte pomocí nástrojů příkazového řádku
* 🐍 **Python SDK** – Programový přístup k API
* 📱 **Více zařízení** – Použití na 2–10 a více zařízeních (v závislosti na tarifu)
* **🐻 Pokročilá metoda debayeringu s ohledem na textury** – vysoce kvalitní debayering s ohledem na hrany v kombinaci s modelem odšumování AI/ML, který odstraňuje téměř veškerý šum vznikající při debayeringu.
* 🧮 **Vlastní vzorce** – vytvářejte vlastní multispektrální indexy

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
* Podívejte se do Prohlížeče událostí na Windows, kde najdete podrobnosti o chybě
* Kontaktujte podporu s protokoly chyb

**CLI se nespustí (Linux):**

* Ověřte, zda je balíček `.deb` správně nainstalován: `dpkg -l | grep chloros`
* Zkontrolujte oprávnění: `sudo chmod +x /usr/bin/chloros-cli`
* Spusťte diagnostiku: `chloros-cli selftest`
* Zkontrolujte, zda nechybí nějaké knihovny: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Problémy s aktivací licence:**

* Ujistěte se, že máte aktivní připojení k internetu
* Ověřte přihlašovací údaje na [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Zkontrolujte, zda brána firewall neblokuje Chloros
* Podrobné pokyny najdete na stránce [Chloros+ Přihlášení](chloros+-login.md)

### Podpora

Potřebujete pomoc s instalací nebo nastavením?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Webové stránky**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentace**: [Začínáme](./)
* ❓ **Často kladené dotazy**: [Často kladené dotazy](faq.md)***

## Aktualizace softwaru

Chloros kontroluje dostupnost aktualizací, upozorní na novou verzi a poskytne odkaz na tuto stránku pro stažení — aktualizaci provedete spuštěním nového podepsaného instalačního programu. Vaše nastavení a projekty zůstanou po aktualizaci zachovány. Na Linux a Jetson zkontroluje `chloros-cli update`, zda je k dispozici novější verze, a nabídne stažení a instalaci odpovídajícího `.deb` (tento příkaz je k dispozici pouze v Linux).

***

## Seznam změn**Verze 1.2.0 (nejnovější)**— úplný seznam funkcí najdete v části**Co je nového ve verzi Chloros 1.2.0** na stránce [Začínáme](./).

<details>

<summary>Verze 1.0.5</summary>

**Datum vydání: 10. února 2026**

**Nové funkce*** **Metoda debayeringu s ohledem na textury \[pouze Chloros+] –** Metoda s ohledem na textury využívá vysoce kvalitní debayering s ohledem na hrany v kombinaci s modelem odšumování založeným na AI/ML, který odstraňuje téměř veškerý šum vznikající při debayeringu.
* **Podpora kalibračních terčů T4P*** **Rychlejší zpracování na GPU (Chloros+), lepší správa paměti**

**Opravy chyb*** Zcela nové uživatelské rozhraní (GUI), které by nyní mělo fungovat na všech počítačích Windows.

</details>

<details>

<summary>Verze 1.0.4</summary>

**Datum vydání: 5. ledna 2026**

**Nové funkce*** **Přepínání mezi obrázky a metadaty**: Do prohlížeče souborů bylo přidáno přepínání, které umožňuje zobrazit metadata vybraného obrázku v tabulce namísto mřížky obrázků
* **Posuvník pro změnu měřítka mřížky obrázků**: Nový posuvník v uživatelském rozhraní pro úpravu velikosti miniatur (podporuje také klávesovou zkratku CTRL + kolečko myši)
* **Tlačítka pro export mřížky obrázků**: Tlačítka v horním řádku pro přepnutí miniatur z formátu JPG na zpracované exporty (Cíle, Odrazivost, Index, LUT)
* **Karta Mapa**: Nová interaktivní 2D mapa zobrazující značky GPS polohy obrázků
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

Neoprávněné použití, šíření nebo úpravy jsou zakázány.

**Bezplatná verze**: K dispozici pro osobní i komerční použití s omezenými funkcemi**Chloros+**: Licence na základě předplatného pro pokročilé funkce a komerční nasazení
