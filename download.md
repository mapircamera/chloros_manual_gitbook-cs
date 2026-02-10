---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Stažení

Stáhněte si nejnovější verzi Chloros a začněte s multispektrálním zpracováním obrazu.

### Systémové požadavky

| Požadavek          | Minimální                                              | Doporučené                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Operační systém** | Windows 10 (64bitový)                                  | Windows 11 (64bitový)                                  |
| **Procesor**        | Intel Core i5 nebo ekvivalentní                          | Intel Core i7 nebo lepší                              |
| **Paměť (RAM)**     | 8 GB                                                  | 16 GB nebo více                                         |
| **Grafická karta**    | Kompatibilní s DirectX 11                                | NVIDIA GPU s 4 GB+ VRAM                            |
| **Úložiště**          | 6 GB volného místa                                       | SSD s 10 GB+ volného místa                            |
| **Displej**          | 1920x1080                                            | 2560x1440 nebo vyšší                                  |
| **Internet**         | Vyžadováno pro \[volitelnou] aktivaci licence Chloros+ | Vyžadováno pro \[volitelnou] aktivaci licence Chloros+ |

{% hint style="info" %}
**GPU akcelerace**: Uživatelé Chloros+ s grafickými kartami NVIDIA mohou využít akceleraci CUDA pro výrazně rychlejší zpracování. Uživatelé Chloros+ také získávají vícevláknové zpracování pro maximální rychlost.
{% endhint %}

***

## Stáhnout Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Stáhnout Chloros zde</a>

### Nejnovější stabilní verze

**Chloros Instalační program pro Windows*** **Verze**: 1.0.5
* **Datum vydání**: 10. února 2026
* **Velikost souboru (ke stažení)**: 1,6 GB
* **Velikost souboru (po instalaci)**: 5,7 GB
* **Typ souboru**: .exe (instalační program Windows)

#### **Kroky instalace:**

1. Stáhněte soubor `CHLOROS INSTALLER - CURRENT VERSION.exe`
2. Poklepejte na instalační program a spusťte instalaci
3. Postupujte podle pokynů instalačního průvodce
4. Vyberte instalační adresář (výchozí: `C:\Program Files\[USER]\Chloros\`)
5. Dokončete instalaci a spusťte Chloros nebo Chloros CLI
6. Přihlaste se pomocí svého účtu [MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (nebo pokračujte s bezplatnou verzí).

{% hint style="success" %}
Instalační program automaticky přidá `chloros-cli` do systémové cesty PATH pro přístup z příkazového řádku.
{% endhint %}

***

## Další zdroje

### Python SDK

Pro vývojáře a automatizační pracovní postupy nainstalujte Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentace**: [API: Python SDK](api-python-sdk.md)**Požadavky**: Chloros Desktop musí být nainstalován, Chloros+ vyžaduje přihlášení k licenci.***

## Co je součástí balení

Instalace Chloros zahrnuje:

* ✅ **Chloros** – plně funkční grafické rozhraní (GUI)
* ✅ **Chloros CLI** – rozhraní příkazového řádku (vyžaduje licenci Chloros+)
* ✅ **Chloros SDK** – Python API (vyžaduje licenci Chloros+)
* ✅ **Profily kamer** – předkonfigurované šablony kamer MAPIR***

## Upgrade na Chloros+

Odemkněte pokročilé funkce s předplatným Chloros+:

* 🚀 **Vícevláknové zpracování** – paralelní zpracování obrazů
* ⚡ **Akcelerace GPU (CUDA)** – využití výkonu GPU NVIDIA
* 💻 **Přístup k CLI** – automatizace pomocí nástrojů příkazového řádku
* 🐍 **Python SDK** – programový přístup API
* 📱 **Více zařízení** – použití na 2–10+ zařízeních (v závislosti na plánu)
* **🐻 Pokročilá metoda debayeringu s rozpoznáním textur** – vysoce kvalitní debayer s rozpoznáním hran v kombinaci s modelem AI/ML pro odstraňování šumu, který odstraní téměř veškerý šum debayeringu. 
* 🧮 **Vlastní vzorce** – Vytvářejte vlastní multispektrální indexy

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Zobrazit XPROTX000044 Plány a ceny XPROTX</a></p>***

## Nápověda k instalaci

### Řešení problémů

**Instalace selhala s chybovou zprávou:**

* Ujistěte se, že máte práva správce
* Dočasně deaktivujte antivirový software
* Zkontrolujte, zda splňujete minimální systémové požadavky

**Aplikace se nespustí:**

* Ověřte, zda je nainstalován Windows 10/11 (64bitový)
* Aktualizujte grafické ovladače
* Zkontrolujte podrobnosti o chybě v Windows Prohlížeči událostí
* Kontaktujte podporu s protokoly chyb

**Problémy s aktivací licence:**

* Ujistěte se, že je aktivní připojení k internetu.
* Ověřte přihlašovací údaje na [https://cloud.mapir.camera](https://cloud.mapir.camera).
* Zkontrolujte, zda firewall neblokuje Chloros.
* Podrobné pokyny najdete na [Chloros+ Přihlášení](chloros+-login.md)

### Získání podpory

Potřebujete pomoc s instalací nebo nastavením?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Webové stránky**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentace**: [Začínáme](./)
* ❓ **Často kladené otázky**: [Často kladené otázky](faq.md)***

## Seznam změn

<details>

<summary>Verze 1.0.5</summary>

#### **Datum vydání**: 10. února 2026**Nové funkce*** **Metoda Debayer s rozpoznáním textury \[Pouze Chloros+] –** Texture Aware používá vysoce kvalitní debayer s rozpoznáním hran v kombinaci s modelem AI/ML pro odstranění šumu, který odstraní téměř veškerý šum debayeringu.
* **Podpora kalibračních cílů T4P*** **Rychlejší zpracování GPU Chloros+, lepší správa paměti**

**Oprava chyb*** Zcela nové uživatelské rozhraní (GUI), které by nyní mělo fungovat na všech počítačích Windows.

</details>

<details>

<summary>Verze 1.0.4</summary>

#### **Datum vydání**: 5. ledna 2026**Nové funkce*** **Přepínání obrázků/metadat**: Do prohlížeče souborů bylo přidáno přepínání, které umožňuje zobrazit metadata vybraného obrázku v tabulce namísto mřížky obrázků
* **Posuvník pro přiblížení mřížky obrázků**: Nový posuvník v uživatelském rozhraní pro úpravu velikosti miniatur (podporuje také CTRL + kolečko myši).
* **Tlačítka pro export mřížky obrázků**: Tlačítka v horním řádku pro přepnutí miniatur z JPG na zpracované exporty (cíle, odrazivost, index, LUT).
* **Karta Mapa**: Nová interaktivní 2D mapa zobrazující značky GPS polohy obrázků.
  * Podporuje mapové dlaždice Google Maps a ESRI (automaticky vybírá nejlepší dlaždicovou službu na základě dostupnosti úrovně přiblížení).
  * Náhled miniatury při najetí myší na značky na mapě.

**Oprava chyb*** Vylepšená podpora instalace Chloros na počítačích s jiným než anglickým jazykem.

</details>

<details>

<summary>Verze 1.0.3</summary>

#### **Datum vydání**: 20. prosince 2025**Nové funkce*** První spuštění

**Vylepšení*** První spuštění

**Oprava chyb*** První spuštění

**Známé problémy*** První spuštění

</details>***

## Licenční smlouva**Proprietární software** – Copyright (c) 2026 MAPIR Inc.

Neoprávněné použití, distribuce nebo úprava jsou zakázány.

**Bezplatná verze**: K dispozici pro osobní a komerční použití s omezenými funkcemi.**Chloros+**: Licence na základě předplatného pro pokročilé funkce a komerční nasazení.
