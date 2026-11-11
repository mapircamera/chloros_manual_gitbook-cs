---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Často kladené otázky

<details>

<summary>Mohu v programu Chloros zpracovávat snímky z kamer jiných značek než MAPIR?</summary>

Ne, Chloros podporuje pouze zpracování snímků z kamer MAPIR — řady Survey3 a LATTICE. Další informace najdete v seznamu [podporovaných modelů kamer](supported-cameras.md). Zpracování snímků z jiných kamer nabízíme v rámci služby MAPIR Cloud, kompletní seznam najdete [zde](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>PodporujeChloros kamery LATTICE?</summary>

Ano. Chloros 1.3.0 podporuje kamerové moduly LATTICE M3C a M3M v plném rozsahu: **živé ovládání**— vyhledávání, připojení, náhled a snímání z karty „Kamery“ v grafickém uživatelském rozhraní (`chloros-cli lattice`) nebo pomocí Python SDK, včetně synchronizovaných sestav více kamer s časovou synchronizací PTP — a**kompletní radiometrické zpracování** pořízených snímků (surové → debayering → radiance → odrazivost → index). Viz [Podporované kamery](supported-cameras.md) a [průvodce LATTICE](lattice/README.md).

</details>

<details>

<summary>Mohu kalibrovat své snímky na odrazivost bez kalibračního terče?</summary>

**Survey3

:** Ne. Bez snímku kalibračního terče pořízeného přibližně ve stejnou dobu, kdy byly pořízeny snímky bez terče, nebudete schopni přiřadit hodnoty pixelů snímku ke známému procentuálnímu podílu odrazivosti. Pokud navíc nepřiložíte záznam ze světelného senzoru MAPIR

, nebude změřeno spektrum okolního světla a výsledky odrazivosti nebudou přesné.

**LATTICE:** Ano. Odrazivost lze odvodit z intenzity slunečního záření směřujícího dolů, měřené světelným senzorem DAQ namísto panelu (ρ = π·L/E). Pokud je v záběru přítomen cíl, který prošel kontrolou kvality (QA), stává se ve výchozím nastavení absolutní referencí (`--reflectance-source auto`), ledaže by prošel s příznakem, v takovém případě se cíl a řetězce DAQ porovnávají na základě naměřeného skóre. Jedna výjimka: „Odrazivost F988 se kalibruje pomocí panelu odrazivosti v záběru: pásmo leží mimo kalibrovaný rozsah světelného senzoru DAQ, proto Chloros

použije váš nejnovější záznam panelu a zachová jej mezi jednotlivými pozorováními panelu.“ Viz [Kalibrační cíle](calibration-targets.md).

</details>

<details>

<summary>Potřebuji světelný senzor DAQ?</summary>

Ne pro radianci: Exporty radiance v LATTICE vycházejí z tovární radiometrické kalibrace každé kamery a nevyžadují ani snímač DAQ, ani kalibrační cíl. Pro **odrazivost**potřebujete referenci pro okolní světlo — buď měření sestupujícího světla snímačem DAQ, nebo kalibrační cíl v záběru. Snímač DAQ vám umožňuje získat kalibrovanou odrazivost**bez umístění jakýchkoli panelů do scény**. Zaznamenané soubory `.daq` se automaticky přiřadí k vašim snímkům podle časového razítka. Viz [Kalibrační terče](calibration-targets.md) a [Referenční příručka k CLI

](reference/cli-reference.md).

</details>

<details>

<summary>Mohu používat Chloros s AI asistentem (Claude, ChatGPT atd.)?</summary>

Ano — tento manuál i CLI a SDK jsou k tomu určeny:

* Kompletní rejstřík manuálu je k dispozici na adrese `https://mapir.gitbook.io/chloros/llms.txt`, aby AI asistenti mohli najít každou stránku.
* Surový markdown každé stránky je k dispozici na její URL adrese v malých písmenech s připojeným `.md` (například `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [Referenční příručka k CLI](reference/cli-reference.md) a [Referenční příručka k SDK](reference/sdk-reference.md) jsou napsány pro použití velkými jazykovými modely (LLM): obsahují přesné příznaky, výchozí hodnoty, sémantiku ukončení a příkazy, které lze zkopírovat a vložit.

V části [AI asistenti](ai-assistants.md) najdete informace o tom, jak nastavit svého asistenta na Chloros.

</details>

<details>

<summary>Kam se ukládají moje zpracované výstupní soubory?</summary>

Výstupy se ukládají do projektové složky, seskupené podle kamery a poté podle formátu souboru:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **složka-kamery** — `LATT-<sensor>-<lens>-F<filter>` pro LATTICE, `<model>_<filter>` (např. `Survey3N_RGN`) pro Survey3
* **složka-formátu** — `tiff16`, `tiff8`, `png8`, `jpg8` nebo `tiff32`
* **složky produktů** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (vždy pod `tiff32`), `<INDEX>_Index_Images/`
* **`clamp_masks/`** — pouze multispektrální data LATTICE: masky na úrovni jednotlivých pixelů vedle složek s produkty, viz níže**Exportované soubory si zachovávají název zdrojového souboru — produkt identifikuje složka, nikoli přípona souboru.**Při použití CLI se projektová složka vytvoří ve vaší výchozí projektové složce (`Documents/MAPIR/Chloros Projects`, pokud jste ji nezměnili), pokud nezadáte `-o`; relativní `-o` se bere relativně ke složce, ze které spouštíte příkaz. Všimněte si, že spuštění s parametrem `chloros-cli process`, které požadovalo produkty, ale žádný nezapsalo, vypíše `Processing finished but wrote no image products.` a**ukončí se s nenulovým stavem**, takže to skripty mohou detekovat. Viz [Formáty výstupních obrázků](output-image-formats.md) a [Referenční příručka CLI](reference/cli-reference.md).

</details>

<details>

<summary>Co jsou složky clamp_masks a soubory *_clampmask.tif?</summary>

Novinka ve verzi 1.3.0. Produkty radiance a reflectance z multispektrální kamery LATTICE M3C procházejí separací pásem 3×3 a tam, kde v některém pásmu vznikne záporná hodnota, program Chloros tuto hodnotu ořízne na nulu. Produkt s alespoň jedním oříznutým pixelem získá jednopásmovou masku `uint8`, která označuje, které pixely a pásma byly oříznuty (bit 0 = kanál R produktu, bit 1 = G, bit 2 = B). Masky se ukládají do složky `clamp_masks/` **vedle** složek produktů — například `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif` — a XMP tag produktu `Chloros:UnmixClampMask` uvádí název své masky (nebo uvádí `none`).

Nejedná se o snímky scény: vlastní import programu Chloros je přeskočí a protože se nikdy nenacházejí uvnitř složky produktu, můžete složku produktu předat programům Pix4D nebo Metashape tak, jak je. Viz [Masky svorek](output-image-formats.md#clamp-masks).

</details>

<details>

<summary>Proč Chloros hlásí „Složka projektu je pro tento počítač příliš hluboká“?</summary>

Toto se týká pouze systému Windows. Systém Windows je dodáván s vypnutou podporou dlouhých cest, a proto žádná cesta k souboru nesmí být delší než 260 znaků. Produkty Chlorosu se nacházejí až asi 130 znaků pod projektovou složkou, takže Chloros před zpracováním odmítne hlubší projektovou složku – namísto toho, aby se spustil a nic nezapsal, jak se mohlo stát před verzí 1.3.0. Zpráva uvádí nejdelší projektovou složku, která se vejde.

Problém vyřešte jedním z následujících způsobů:

* Použijte kratší složku projektu (položku **Pracovní adresář** v nastavení projektu nebo `-o` / `chloros-cli set-project-folder` pro příkazový řádek (CLI)).
* Zapněte podporu dlouhých cest ve Windows a restartujte Chloros. Tuto funkci zajišťuje volba **Povolit dlouhé cesty ve Windows** v instalačním programu Chloros (ve výchozím nastavení zaškrtnutá); pokud jste ji odškrtli, spusťte instalační program znovu.

Viz [Spuštění zpracování](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

</details>

<details>

<summary>Mohu upravovat své snímky před jejich zpracováním v programu Chloros?</summary>

Ne. Chloros předpokládá, že vstupní data nebyla upravena. Neměňte názvy souborů.

</details>

<details>

<summary>Mohu nastavit své kamery MAPIRSurvey3 na automatickou expozici a zpracovat snímky v programu Chloros?</summary>

Ne. Datové sady snímků z kamer Survey3 musí mít pevnou/uzamčenou expozici, tedy bez automatické rychlosti závěrky ani automatického ISO. Všechny snímky pořízené stejným modelem kamery musí mít identickou rychlost závěrky a ISO (expozici).

Kamery LATTICE toto omezení nemají: Chloros řídí jejich expozici v reálném čase (Smart AE) a při každém snímku zaznamenává skutečně použitou expozici a zesílení, což radiometrický proces zohledňuje.

</details>

<details>

<summary>Může program Chloros zpracovávat nebo analyzovat ortomozaikové snímky?</summary>

Ne. Podporovány jsou pouze jednotlivé snímky z kamer MAPIR, nikoli složené snímky, jako je ortomozaiková mapa.

</details>

<details>

<summary>Jak mohu urychlit krok detekce cílů v programu Chloros?</summary>

V tabulce prohlížeče souborů předvolbou cílových snímků v pravém sloupci dáte programu Chloros pokyn, aby hledal kalibrační cíle pouze v těchto snímcích, což výrazně zrychlí zpracování.

</details>

<details>

<summary>Pokud hodlám nahrát své snímky do <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> měl bych je před nahráním zpracovat v programu Chloros?</summary>

Pokud plánujete nahrát snímky na naši online platformu pro zpracování [MAPIR

Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), neupravujte je před nahráním. Cloud provede stejné zpracování a ještě více.

</details>

<details>

<summary>Bude MAPIR někdy podporovat funkci X? Opravdu bych si přál, aby MAPIR funkci X nabízel.</summary>

Vždy nás zajímá zpětná vazba k našim produktům. Pokud narazíte na problém s našimi produkty nebo máte návrh, jak je můžeme vylepšit, prosím [KONTAKTUJTE NÁS](https://www.mapir.camera/community/contact) a sdělte nám své nápady. Většina našeho výzkumu a vývoje se řídí tím, že nasloucháme nejdůležitějším potřebám našich zákazníků.

</details>

<details>

<summary>Je Chloros k dispozici pro Linux?</summary>

Ano! Chloros 1.3.0 podporuje Linux amd64 (x86_64) a arm64 (NVIDIA Jetson JetPack 6) prostřednictvím balíčků `.deb`. CLI a Python SDK jsou na Linux plně podporovány, včetně živého ovládání kamer LATTICE a senzorů DAQ. Pro Linux není k dispozici žádné grafické uživatelské rozhraní (GUI) – veškerá interakce probíhá prostřednictvím [CLI](CLI.md) nebo [Python SDK](api-python-sdk.md). Podrobnosti najdete v [Přehledu pro Linux](linux/linux-overview.md).

</details>

<details>

<summary>Mohu spustit Chloros na NVIDIA Jetson?</summary>

Ano! Chloros podporuje **řadu Orin** od NVIDIA Jetson — Orin Nano, Orin NX a AGX Orin — s operačním systémem JetPack 6. Starší modely (Jetson Nano, TX2, Xavier NX) nemohou spustit JetPack 6 a nejsou aktuálním balíčkem podporovány. Chloros automaticky rozpozná váš model Jetson a optimalizuje strategii zpracování. Balíček pro Jetson využívá verzi PyTorch pro CUDA z JetPacku 6, takže zpracování probíhá na GPU zařízení Jetson. Pokyny k nastavení a nasazení najdete v [Průvodci NVIDIA Jetson](linux/nvidia-jetson-guide.md).

</details>

<details>

<summary>Optimalizuje se Chloros automaticky pro můj hardware?</summary>

Ano! Chloros obsahuje funkci [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md), která automaticky detekuje váš procesor (CPU), grafický procesor (GPU), paměť RAM a (na Jetsonu) teplotní senzory. Následně vybere optimální strategii zpracování — od `GPU_PARALLEL` pro GPU s velkou pamětí přes `GPU_SINGLE` pro menší GPU až po `CPU_PARALLEL` pro systémy bez GPU s podporou CUDA, které může PyTorch využívat. Není potřeba žádná ruční konfigurace.

</details>

<details>

<summary>Co je to 4-vláknový zpracovatelský pipeline?</summary>

Chloros
využívá pro uživatele Chloros
+ architekturu s 4-vláknovým zpracováním: Vlákno 1 (Detekce) načítá obrázky a detekuje kalibrační cíle, vlákno 2 (Kalibrace) provádí výpočet kalibrace odrazivosti, vlákno 3 (Zpracování) provádí GPU-akcelerované odstraňování bayerského vzoru a výpočet indexu a vlákno 4 (Export) zapisuje výstupní soubory. Pro maximální propustnost může být v různých vláknech současně zpracováno více snímků. Podrobnosti najdete v části [Zpracovací pipeline](processing-architecture/processing-pipeline.md).

</details>

<details>

<summary>Jak spustím diagnostiku na své instalaci Chloros?</summary>

Pomocí příkazu `selftest` spusťte sedmibodový test funkčnosti: verze, dostupnost portů, spuštění backendu, připojení k API (`/api/test`), systémové informace (`/api/system-info` — GPU/CUDA/PyTorch), přítomnost modelu odšumovače a připravenost CUDA + odšumovače:

```bash
chloros-cli selftest
```

To je obzvláště užitečné na systémech Linuxu k ověření nastavení GPU a CUDA. Na zařízení Jetson obvykle `CUDA: False` znamená, že chybí knihovny CUDA a cuDNN pro JetPack 6: nainstalujte je pomocí `sudo apt install nvidia-jetpack`.

</details>
