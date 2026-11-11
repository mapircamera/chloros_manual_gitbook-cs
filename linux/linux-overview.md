# Přehled pro Linux

Chloros 1.3.0 nabízí nativní podporu pro Linux v rámci **CLI**a**Python SDK** – zpracování multispektrálních snímků bez grafického rozhraní a živé ovládání kamer LATTICE a světelných senzorů DAQ – na pracovních stanicích, serverech a okrajových zařízeních NVIDIA Jetson s operačním systémem Linux.

{% hint style="info" %}
**Na Linuxu není k dispozici grafické uživatelské rozhraní (GUI).**Grafické uživatelské rozhraní Chloros Desktop je dostupné pouze pro Windows. Uživatelé Linuxu komunikují s Chloros prostřednictvím [CLI](../CLI.md) a [Python SDK](../api-python-sdk.md). `.deb` sice přidá položku**Chloros CLI** do nabídky aplikací — jednoduše však spustí emulátor terminálu, na kterém běží `chloros-cli`.
{% endhint %}

***

## Matice podpory platforem

| Funkce | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/ --- | --- | --- | --- |
| **GUI pro stolní počítače** | Ano | N/A | Ne | Ne |
| **CLI** (`chloros-cli`) | Ano | Ano | Ano | Ano |
| **Python SDK** (`chloros-sdk`) | Ano | Ano | Ano | Ano |
| **Pipeline pro zpracování obrazu** | Ano | Ano | Ano | Ano |
| **Ovládání kamery LATTICE (živě)** | Ano (karta Kamery) | Ano (`chloros-cli lattice`, SDK) | Ano | Ano |
| **Světelné senzory DAQ (živě)** | Ano (karta Světelné senzory) | Ano (`chloros-cli daq pool-*`, SDK) | Ano | Ano |
| **Časová synchronizace PTP (hostitel je grandmaster)** | Ano | Ano (`chloros-cli time-sync`) | Ano | Ano |
| **Akcelerace GPU (CUDA)** | Ano | Ano | Ano | Ano (JetPack 6) |
| **Debayer s podporou textur** (vyžaduje GPU s podporou CUDA) | Ano (Chloros+) | Ano (Chloros+) | Ano (Chloros+) | Ano (Chloros+) |
| **Dynamická adaptace výpočtů** | Ano | Ano | Ano | Ano |
| **Backend jako systémová služba** (`chloros-backend@<user>.service`) | Ne | Ne | Ano (na přání) | Ano (na přání) |
| **Aktualizace na místě** (`chloros-cli update`) | Ne (spusťte instalační program) | Ne (spusťte instalační program) | Ano | Ano |***

## Podporované architektury

| Architektura | Popis | Balíček |
| --- | --- | --- |
| **amd64 (x86_64)** | Standardní procesory pro stolní počítače a servery (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | Procesory ARM — řada NVIDIA Jetson Orin (JetPack 6) | `chloros_<version>_arm64_jp6.deb` |

## Podporované distribuce Linuxu

* **Ubuntu 22.04 LTS nebo novější** (amd64)
* **Debian 12 nebo novější** (amd64)
* **NVIDIA JetPack 6** (arm64 — platformy Jetson Orin)***

## Co získají uživatelé Linuxu

* **Chloros CLI** — kompletní rozhraní příkazového řádku pro dávkové zpracování, automatizaci a skriptování
* **Chloros Python SDK** — programové rozhraní v jazyce Python pro výzkumné pracovní postupy a vlastní nástroje, instalované balíčkem `.deb` pro systém `python3` (wheel s odpovídající verzí pro virtualenvs je v balíčku `/usr/lib/chloros/sdk/`; PyPI je určeno pro stroje bez balíčku Chloros)
* **Ovládání kamer LATTICE** — vyhledávání, připojování, konfigurace a snímání z kamer LATTICE a synchronizovaných soustav více kamer prostřednictvím balíčku `chloros-cli lattice` a SDK; balíček `.deb` obsahuje runtime Arena SDK, který kamery vyžadují
* **Ovládání světelných senzorů DAQ** — připojte senzory DAQ-U/M/E, streamujte kalibrovaná spektra a zaznamenávejte soubory `.daq` prostřednictvím `chloros-cli daq pool-*` a SDK
* **Časová synchronizace PTP** — backend Chloros spouští PTP grandmaster, ke kterému se připojují kamery LATTICE a senzory DAQ-E jako slave; zkontrolujte jej pomocí `chloros-cli time-sync` a udržujte jej v bezobslužném provozu pomocí jednotky systemd `chloros-backend@<user>.service` (viz [Instalace na Linuxu](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Automatizace projektů** — spouštějte uložené projekty bezobslužně pomocí `chloros-cli project` a `open_project` z SDK
* **Akcelerace GPU** — zpracování akcelerované technologií CUDA na grafických kartách NVIDIA (stolní grafické karty na platformě amd64 a grafická karta Jetson s JetPack 6)
* **Dynamická adaptace výpočtů** — automatická detekce hardwaru a výběr strategie zpracování, s možností přepsání pomocí `CHLOROS_STRATEGY` jako nouzového řešení pro pokročilé uživatele
* **Všechny funkce zpracování** — stejný postup jako ve Windows: kalibrace, korekce vinětace, vegetační indexy a všechny formáty exportu
* **Funkce Chloros+** — vícevláknové (pipelinové) zpracování, debayer s podporou textur (na grafickém procesoru CUDA) a vlastní indexy, s placeným tarifem Chloros+

## Co uživatelé Linuxu nedostanou

* **Grafické uživatelské rozhraní** — žádné grafické rozhraní; veškerá interakce probíhá přes CLI nebo Python SDK
* **Prohlížeč snímků** — žádný interaktivní prohlížeč snímků, mřížkové zobrazení ani mapové značky
* **Vizuální správa projektů** — projekty se vytvářejí a řídí pomocí příkazů CLI a volání SDK (samotný hardware — kamery, senzory, snímání — zůstává plně ovladatelný z terminálu)***

## Licenční požadavky

Přístup k CLI a SDK vyžaduje **placený tarif Chloros+ — Copper nebo vyšší**(Copper, Bronze, Silver, Gold). Bezplatný tarif**Iron** nemá přístup k CLI/SDK. Toto omezení je vynucováno na úrovni backendu, nikoli pouze v CLI:

| Situace | Odpověď backendu |
| --- | --- |
| Nejsem přihlášen | `401` s `error_code: AUTH_REQUIRED` |
| Přihlášeno na bezplatném tarifu Iron | `403` s `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` funguje na jakémkoli tarifu — je to jediná trasa osvobozená od brány — takže důvod odmítnutí je vždy viditelný.

***

## Začínáme na Linuxu

1. **Nainstalujte Chloros** — viz [Instalace na Linuxu](linux-installation.md) pro instalaci `.deb`
2. **Ověřte** — `chloros-cli --version` vypíše `Chloros CLI 1.3.0`; `chloros-cli selftest` spustí 7krokovou diagnostiku
3. **Zkontrolujte Python SDK** — `.deb` jej nainstaluje: `python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"` (ve virtuálním prostředí virtualenv, `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`)
4. **Přihlaste se** — `chloros-cli login your@email.com 'your-password'` (jednou na každém počítači; při aktualizaci balíčku se přihlašovací údaje zachovají, odstranění balíčku je vymaže)
5. **Zpracujte svůj první datový soubor** — `chloros-cli process ~/datasets/flight001`

Pro NVIDIA Jetson si přečtěte speciální [Průvodce NVIDIA Jetson](nvidia-jetson-guide.md), kde najdete informace o nastavení specifickém pro danou platformu, teplotním chování a nasazení v terénu.

***

## Další kroky

* [Instalace v systému Linux](linux-installation.md) — podrobný postup instalace, umístění souborů a řešení problémů pro architektury amd64 a arm64
* [Průvodce pro NVIDIA Jetson](nvidia-jetson-guide.md) — nastavení specifické pro Jetson, chování paměti a tepelné chování, nasazení v terénu
* [CLI: Příkazový řádek](../CLI.md) — příručka k CLI
* [API: Python SDK](../api-python-sdk.md) — příručka k SDK
* [Referenční příručka CLI](../reference/cli-reference.md) a [Referenční příručka SDK](../reference/sdk-reference.md) — vyčerpávající seznamy příkazů a API pro verzi 1.3.0
* [Dynamická adaptace výpočetního výkonu](../processing-architecture/dynamic-compute-adaptation.md) — jak se Chlorospřizpůsobuje vašemu hardwaru

{% hint style="info" %}
**Programové čtení této příručky.** Každá stránka je také dostupná jako surový Markdown na vlastní URL plus `.md` (například `https://mapir.gitbook.io/chloros/linux/linux-installation.md`) a rejstřík celé příručky je zveřejněn na [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
