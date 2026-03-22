# Linux – přehled

Verze Chloros 1.1.0 přináší nativní podporu pro modely **CLI**a**Python SDK**, což umožňuje bezhlavé zpracování multispektrálních snímků na pracovních stanicích, serverech a okrajových zařízeních NVIDIA Jetson.

{% hint style="info" %}
**Na Linux není k dispozici grafické uživatelské rozhraní.** Grafické uživatelské rozhraní Chloros Desktop je k dispozici pouze na Windows. Uživatelé Linux komunikují s Chloros prostřednictvím [CLI](../CLI.md) a [Python SDK](../api-python-sdk.md).
{% endhint %}

***

## Matice podpory platforem

| Funkce | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Desktopové grafické rozhraní** | Ano | N/A | Ne | Ne |
| **CLI** | Ano | Ano | Ano | Ano |
| **Python SDK** | Ano | Ano | Ano | Ano |
| **Akcelerace GPU (CUDA)** | Ano | Ano | Ano | Ano (JetPack 6) |
| **Debayer s podporou textur** | Ano (Chloros+) | Ano (Chloros+) | Ano (Chloros+) | Ano (Chloros+) |
| **Dynamická adaptace výpočtů** | Ano | Ano | Ano | Ano |***

## Podporované architektury

| Architektura | Popis | Způsob instalace |
| --- | --- | --- |
| **amd64 (x86_64)** | Standardní procesory pro stolní počítače/servery (Intel, AMD) | Balíček `.deb` |
| **arm64 (aarch64)** | Procesory založené na architektuře ARM, především NVIDIA Jetson | Balíček `.deb` (JetPack 6) |

## Podporované distribuce Linux

* **Ubuntu 20.04+** (amd64)
* **Debian 11+** (amd64)
* **NVIDIA JetPack 6** (arm64 — platformy Jetson)***

## Co uživatelé Linux získají

* **Chloros CLI** — Plnohodnotné rozhraní příkazového řádku pro dávkové zpracování, automatizaci a skriptování
* **Chloros Python SDK** — Programové rozhraní Python (`pip install chloros-sdk`) pro integraci do výzkumných pipeline a vlastních nástrojů
* **Akcelerace GPU** — Zpracování akcelerované technologií CUDA na GPU NVIDIA (stolní počítače a Jetson)
* **Dynamická adaptace výpočtů** — Automatická detekce hardwaru a optimalizace strategie zpracování
* **Všechny funkce zpracování** — Stejný multispektrální zpracovatelský pipeline jako Windows (kalibrace, korekce vinětace, vegetační indexy, všechny exportní formáty)
* **Funkce Chloros+** — Vícevláknové zpracování, debayer s podporou textur, vlastní indexy (s licencí Chloros+)

## Co uživatelé Linux nedostanou

* **Desktopové grafické rozhraní** — Žádné grafické rozhraní; veškerá interakce probíhá přes CLI nebo Python SDK
* **Prohlížeč obrázků** — Žádný interaktivní prohlížeč obrázků, zobrazení mřížky ani značky na mapě
* **Vizuální správa projektů** — Projekty se spravují pomocí příkazů CLI a volání SDK***

## Začínáme s Linux

1. **Instalace Chloros** — Informace o instalaci balíčku `.deb` najdete v části [Instalace Linux](linux-installation.md)
2. **Nainstalujte Python SDK** (volitelné) — `pip install chloros-sdk`
3. **Aktivujte licenci** — `chloros-cli login your@email.com 'password'`
4. **Zpracujte svůj první datový soubor** — `chloros-cli process ~/datasets/flight001`

Uživatelé NVIDIA Jetson najdou informace o nastavení a optimalizaci pro konkrétní platformu v [Průvodci NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Další kroky

* [Instalace Linux](linux-installation.md) — Podrobné pokyny k instalaci pro amd64 a arm64
* [Průvodce NVIDIA Jetson](nvidia-jetson-guide.md) — Nastavení specifické pro Jetson, řízení teploty a nasazení v terénu
* [CLI : Příkazový řádek](../CLI.md) — Kompletní reference k CLI
* [API : Python SDK](../api-python-sdk.md) — Kompletní reference k SDK
* [Dynamická adaptace výpočetního výkonu](../processing-architecture/dynamic-compute-adaptation.md) — Jak se Chloros přizpůsobuje vašemu hardwaru
