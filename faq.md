---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Často kladené otázky

<details>

<summary>Mohu pomocí Chloros zpracovávat snímky z kamer jiných značek než MAPIR?</summary>

Ne, Chloros podporuje pouze zpracování snímků z kamer MAPIR — řady Survey3 a LATTICE. Další informace najdete v seznamu [podporovaných modelů kamer](supported-cameras.md). Zpracování snímků z jiných kamer nabízíme v rámci služby MAPIR Cloud, úplný seznam najdete [zde](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Podporuje Chloros kamery LATTICE?</summary>

Ano. Chloros 1.2.0 podporuje kamerové moduly LATTICE M3C a M3M v plném rozsahu: **živé ovládání**— vyhledávání, připojení, náhled a snímání z karty „Kamery“ v grafickém uživatelském rozhraní, `chloros-cli lattice` nebo Python SDK, včetně synchronizovaných sestav více kamer s časovou synchronizací PTP — a**kompletní radiometrické zpracování** pořízených snímků (surový → debayering → radiance → odrazivost → index). Viz [Podporované kamery](supported-cameras.md) a [příručka LATTICE](lattice/README.md).

</details>

<details>

<summary>Mohu kalibrovat své snímky na odrazivost bez kalibračního terče?</summary>

**Survey3:** Ne. Bez snímku kalibračního terče pořízeného přibližně ve stejnou dobu jako snímky bez terče nebudete schopni přiřadit hodnoty pixelů snímku ke známému procentuálnímu podílu odrazivosti. Pokud navíc nepřiložíte záznam ze světelného senzoru MAPIR, nebude změřeno spektrum okolního světla a výsledky odrazivosti nebudou přesné.**LATTICE:** Ano. Odrazivost lze vztažit k intenzite slunečního záření směřujícího dolů, naměřené světelným senzorem DAQ namísto panelu (ρ = π·L/E). Pokud je v záběru přítomen cíl, který prošel kontrolou kvality (QA), stává se ve výchozím nastavení absolutní referencí (`--reflectance-source auto`). Jedna výjimka: „Odrazivost F988 se kalibruje pomocí panelu odrazivosti v záběru: pásmo leží mimo kalibrovaný rozsah světelného senzoru DAQ, takže Chloros použije váš nejnovější záznam panelu a zachová jej mezi jednotlivými pozorováními panelu.“ Viz [Kalibrační terče](calibration-targets.md).

</details>

<details>

<summary>Potřebuji světelný senzor DAQ?</summary>

Ne pro radianci: Exporty radiance v LATTICE vycházejí z tovární radiometrické kalibrace každé kamery a nevyžadují ani snímač DAQ, ani kalibrační cíl. Pro **odrazivost**potřebujete referenci pro okolní světlo — buď měření sestupujícího světla pomocí světelného snímače DAQ, nebo kalibrační cíl v záběru. Snímač DAQ vám umožňuje získat kalibrované hodnoty odrazivosti**bez umístění jakýchkoli panelů do scény**. Zaznamenané soubory `.daq` se automaticky přiřadí k vašim snímkům podle časového razítka. Viz [Kalibrační terče](calibration-targets.md) a [Referenční materiál CLI](reference/cli-reference.md).

</details>

<details>

<summary>Mohu Chloros použít s AI asistentem (Claude, ChatGPT atd.)?</summary>

Ano — tento manuál a soubory CLI/SDK jsou pro to připraveny:

* Kompletní rejstřík manuálu je dostupný na adrese `https://mapir.gitbook.io/chloros/llms.txt`, aby AI asistenti mohli najít každou stránku.
* Surový markdown každé stránky je k dispozici na stránce s názvem v malých písmenech URL s připojeným `.md` (například `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [Referenční příručka k CLI](reference/cli-reference.md) a [SDK](reference/sdk-reference.md) jsou napsány pro použití v LLM: přesné příznaky, výchozí hodnoty, sémantika ukončení a příkazy, které lze zkopírovat a vložit.

V části [AI asistenti](ai-assistants.md) najdete informace o tom, jak nasměrovat svého asistenta na Chloros.

</details>

<details>

<summary>Kam se ukládají moje zpracované výstupní soubory?</summary>

Výstupy se ukládají do projektové složky, seskupené podle kamery a následně podle formátu souboru:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **složka-kamery** — `LATT-<sensor>-<lens>-F<filter>` pro LATTICE, `<model>_<filter>` (např. `Survey3N_RGN`) pro Survey3
* **složka formátu** — `tiff16`, `tiff8`, `png8`, `jpg8` nebo `tiff32`
* **složky produktů** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (vždy pod `tiff32`), `<INDEX>_Index_Images/`**Exportované soubory si zachovávají název zdrojového souboru — produkt identifikuje složka, nikoli přípona souboru.**U CLI se projektová složka vytvoří vedle vstupní složky, pokud nezadáte `-o`. Všimněte si, že spuštění `chloros-cli process`, které požadovalo produkty, ale žádný nezapsalo, vypíše `Processing finished but wrote no image products.` a**ukončí se s nenulovým stavem**, takže to skripty mohou detekovat. Viz [Formáty výstupních obrázků](output-image-formats.md) a [Referenční příručka k CLI](reference/cli-reference.md).

</details>

<details>

<summary>Mohu upravit své obrázky před zpracováním v Chloros?</summary>

Ne. Chloros předpokládá, že vstupní data nebyla upravena. Neměňte názvy souborů.

</details>

<details>

<summary>Mohu nastavit své kamery MAPIR a Survey3 na automatickou expozici a zpracovat snímky v programu Chloros?</summary>

Ne. Datové sady snímků Survey3 musí mít pevnou/uzamčenou expozici, takže nesmí být použita automatická rychlost závěrky ani automatické ISO. Všechny snímky pořízené stejným modelem kamery musí mít identickou rychlost závěrky a ISO (expozici).

Kamery LATTICE toto omezení nemají: Chloros řídí expozici v reálném čase (Smart AE) a při každém snímku zaznamenává skutečně použitou expozici a zisk, což radiometrický zpracovací řetězec zohledňuje.

</details>

<details>

<summary>Může Chloros zpracovávat nebo analyzovat ortomozaikové snímky?</summary>

Ne. Podporovány jsou pouze jednotlivé snímky z kamery MAPIR, nikoli složené snímky, jako je ortomozaiková mapa.

</details>

<details>

<summary>Jak mohu urychlit krok detekce cílů v programu Chloros?</summary>

V tabulce prohlížeče souborů předvolbou cílových snímků v pravém sloupci dáte programu Chloros pokyn, aby hledal kalibrační cíle pouze v těchto snímcích, což výrazně zrychlí zpracování.

</details>

<details>

<summary>Pokud hodlám nahrát své snímky do <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">cloudu MAPIR Cloud,</a> měl bych je před nahráním zpracovat v programu Chloros?</summary>

Pokud plánujete nahrát snímky na naši online platformu pro zpracování [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), neupravujte je před nahráním. Cloud provede stejné zpracování a ještě další úkony.

</details>

<details>

<summary>Bude MAPIR někdy podporovat funkci X? Opravdu bych si přál, aby MAPIR nabízelo funkci X.</summary>

Vždy nás zajímají zpětné vazby k našim produktům. Pokud narazíte na problém s našimi produkty nebo máte návrh, jak je můžeme vylepšit, [KONTAKTUJTE NÁS](https://www.mapir.camera/community/contact) a sdělte nám své nápady. Většina našeho výzkumu a vývoje se řídí tím, že nasloucháme nejdůležitějším potřebám našich zákazníků.

</details>

<details>

<summary>Je Chloros k dispozici pro Linux?</summary>

Ano! Chloros 1.2.0 podporuje Linux amd64 (x86_64) a arm64 (NVIDIA Jetson JetPack 6) prostřednictvím balíčků `.deb`. CLI a Python SDK jsou na Linux plně podporovány, včetně živého ovládání kamer LATTICE a senzorů DAQ. Pro Linux není k dispozici žádné grafické uživatelské rozhraní — veškerá interakce probíhá prostřednictvím [CLI](CLI.md) nebo [Python SDK](api-python-sdk.md). Podrobnosti najdete v [Linux Přehledu](linux/linux-overview.md).

</details>

<details>

<summary>Mohu spustit Chloros na platformě NVIDIA Jetson?</summary>

Ano! Chloros podporuje platformy NVIDIA Jetson, včetně modelů Jetson Nano, Orin Nano, Orin NX a AGX Orin s operačním systémem JetPack 6. Chloros automaticky rozpozná váš model Jetson a optimalizuje strategii zpracování. Pokyny k nastavení a nasazení najdete v [Průvodci NVIDIA Jetson](linux/nvidia-jetson-guide.md).

</details>

<details>

<summary>Optimalizuje se Chloros automaticky pro můj hardware?</summary>

Ano! Chloros obsahuje funkci [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md), která automaticky detekuje váš procesor (CPU), grafický procesor (GPU), paměť RAM a (na zařízeních Jetson) teplotní senzory. Následně vybere optimální strategii zpracování – od `GPU_PARALLEL` na systémech s velkou pamětí přes `GPU_SINGLE` na zařízeních s omezenými zdroji až po `CPU_PARALLEL` na systémech bez grafického procesoru NVIDIA. Není potřeba žádná ruční konfigurace.

</details>

<details>

<summary>Co je to 4-vláknový zpracovatelský pipeline?</summary>

Chloros využívá čtyřvláknovou architekturu s pipeline pro uživatele Chloros+: Vlákno 1 (Detekce) načítá obrázky a detekuje kalibrační cíle, vlákno 2 (Kalibrace) vypočítává kalibraci odrazivosti, vlákno 3 (Zpracování) provádí GPU-akcelerované odstraňování bayeru a výpočet indexu a vlákno 4 (Export) zapisuje výstupní soubory. Pro dosažení maximální propustnosti může být v různých vláknech současně zpracováváno více snímků. Podrobnosti najdete v části [Zpracovací pipeline](processing-architecture/processing-pipeline.md).

</details>

<details>

<summary>Jak spustím diagnostiku na své instalaci Chloros?</summary>

Pomocí příkazu `selftest` spusťte sedmibodový test funkčnosti: verze, dostupnost portů, spuštění backendu, připojení API (`/api/test`), informace o systému (`/api/system-info` — GPU/CUDA/PyTorch), přítomnost modelu odšumovače a připravenost CUDA + odšumovače:

```bash
chloros-cli selftest
```

To je obzvláště užitečné na systémech Linux/Jetson k ověření nastavení GPU a CUDA.

</details>
