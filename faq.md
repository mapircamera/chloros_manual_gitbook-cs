---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Často kladené otázky

<details>

<summary>Mohu pomocí Chloros zpracovávat snímky z kamer, které nejsou značky MAPIR?</summary>

Ne, Chloros podporuje pouze zpracování snímků z kamer MAPIR. Další informace najdete v seznamu [podporovaných modelů kamer](supported-cameras.md). Zpracování snímků z jiných kamer nabízíme na platformě MAPIR Cloud, kompletní seznam najdete [zde](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Mohu kalibrovat své snímky na odrazivost bez kalibračního terče?</summary>

Ne. Bez snímku kalibračního terče pořízeného v době, kdy jsou pořizovány snímky bez terče, nebudete moci přiřadit hodnoty pixelů snímku ke známému procentuálnímu odrazivosti. Pokud také nezahrnete záznam ze světelného senzoru MAPIR, nebude změřeno spektrum okolního světla a výsledky odrazivosti nebudou přesné.

</details>

<details>

<summary>Mohu upravit své snímky před zpracováním v Chloros?</summary>

Ne. Chloros předpokládá, že vstupní data nebyla upravena. Neměňte názvy souborů.

</details>

<details>

<summary>Mohu nastavit své kamery MAPIR a Survey3 na automatickou expozici a zpracovat snímky v programu Chloros?</summary>

Ne. Datové sady snímků Survey3 musí mít pevnou/uzamčenou expozici, takže nelze použít automatickou rychlost závěrky ani automatické ISO. Všechny snímky stejného modelu kamery musí mít identickou rychlost závěrky a ISO (expozici).

</details>

<details>

<summary>Může Chloros zpracovávat nebo analyzovat ortomozaikové snímky?</summary>

Ne. Podporovány jsou pouze jednotlivé snímky z kamery MAPIR, nikoli složené snímky, jako je ortomozaická mapa.

</details>

<details>

<summary>Jak mohu urychlit krok detekce cílů v programu Chloros?</summary>

V tabulce prohlížeče souborů předvolbou cílových snímků v pravém sloupci dáte Chloros pokyn, aby hledal kalibrační cíle pouze v těchto snímcích, což výrazně zrychlí zpracování.

</details>

<details>

<summary>Pokud budu nahrávat své snímky do <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">cloudu MAPIR,</a> měl bych je před nahráním zpracovat v programu Chloros?</summary>

Pokud plánujete nahrát obrázky na naši online platformu pro zpracování [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), neupravujte je před nahráním. Cloud provede stejné zpracování a ještě více.

</details>

<details>

<summary>Bude MAPIR někdy podporovat funkci X? Opravdu bych si přál, aby MAPIR nabízelo X.</summary>

Vždy nás zajímá zpětná vazba k našim produktům. Pokud narazíte na problém s našimi produkty nebo máte návrh, jak je můžeme vylepšit, prosím [KONTAKTUJTE NÁS](https://www.mapir.camera/community/contact) a sdělte nám své nápady. Většina našeho výzkumu a vývoje se řídí tím, že nasloucháme nejdůležitějším potřebám našich zákazníků.

</details>

<details>

<summary>Je Chloros k dispozici pro Linux?</summary>

Ano! Chloros 1.1.0 podporuje Linux amd64 (x86\_64) a arm64 (NVIDIA Jetson JetPack 6) prostřednictvím balíčků `.deb`. CLI a Python SDK jsou plně podporovány na Linux. Pro Linux neexistuje žádné grafické uživatelské rozhraní (GUI) — veškerá interakce probíhá prostřednictvím [CLI](CLI.md) nebo [Python SDK](api-python-sdk.md). Podrobnosti najdete v [Linux Přehled](linux/linux-overview.md).

</details>

<details>

<summary>Mohu spustit Chloros na NVIDIA Jetson?</summary>

Ano! Chloros 1.1.0 podporuje platformy NVIDIA Jetson, včetně Jetson Nano, Orin Nano, Orin NX a AGX Orin s JetPack 6. Chloros automaticky detekuje váš model Jetson a optimalizuje strategii zpracování. Pokyny k nastavení a nasazení najdete v [Průvodci NVIDIA Jetson](linux/nvidia-jetson-guide.md).

</details>

<details>

<summary>Optimalizuje se Chloros automaticky pro můj hardware?</summary>

Ano! Chloros 1.1.0 obsahuje [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md), která automaticky detekuje váš CPU, GPU, RAM a (na Jetsonu) teplotní senzory. Poté vybere optimální strategii zpracování – od `GPU_PARALLEL` na systémech s velkou pamětí přes `GPU_SINGLE` na zařízeních s omezenými možnostmi až po `CPU_PARALLEL` na systémech bez grafického procesoru NVIDIA. Není potřeba žádná ruční konfigurace.

</details>

<details>

<summary>Co je to 4-vláknový zpracovatelský pipeline?</summary>

Chloros 1.1.0 používá 4-vláknovou pipelined architekturu pro uživatele Chloros+: Vlákno 1 (Detekce) načítá obrázky a detekuje kalibrační cíle, vlákno 2 (Kalibrace) vypočítává kalibraci odrazivosti, vlákno 3 (Zpracování) provádí GPU akcelerované odstraňování bayeringu a výpočet indexu a vlákno 4 (Export) zapisuje výstupní soubory. Pro maximální propustnost může být v různých vláknech současně více obrázků. Podrobnosti najdete v části [Zpracovací pipeline](processing-architecture/processing-pipeline.md).

</details>

<details>

<summary>Jak spustím diagnostiku na mé instalaci Chloros?</summary>

Pomocí příkazu `selftest` spusťte 7 systémových diagnostik, včetně kontroly verze, dostupnosti portů, spuštění backendu, připojení API, informací o systému, modelů odšumovače a dostupnosti CUDA:

```bash
chloros-cli selftest
```

To je užitečné zejména na systémech Linux/Jetson k ověření nastavení GPU a CUDA.

</details>
