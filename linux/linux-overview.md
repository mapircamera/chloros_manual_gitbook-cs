# Linux – přehled

Chloros 1.2.0 poskytuje nativní podporu Linux pro **CLI**a**Python SDK** — zpracování multispektrálních snímků bez grafického rozhraní, včetně živého ovládání kamer LATTICE a světelných senzorů DAQ — na pracovních stanicích, serverech a okrajových zařízeních NVIDIA Jetson.

{% hint style="info" %}
**Na Linux není k dispozici grafické uživatelské rozhraní pro stolní počítače.**Grafické uživatelské rozhraní pro stolní počítače Chloros je dostupné pouze na Windows. Uživatelé Linux komunikují s Chloros prostřednictvím [CLI](../CLI.md) a [Python SDK](../api-python-sdk.md). `.deb` přidává položku**Chloros CLI** do nabídky vaší aplikace — jednoduše otevře emulátor terminálu, na kterém běží `chloros-cli`.
{% endhint %}

***

## Matice podpory platforem

| Funkce | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Grafické uživatelské rozhraní pro stolní počítače** | Ano | N/A | Ne | Ne |
| **CLI** (`chloros-cli`) | Ano | Ano | Ano | Ano |
| **Python SDK** (`chloros-sdk`) | Ano | Ano | Ano | Ano |
| **Zpracování obrazu** | Ano | Ano | Ano | Ano |
| **Ovládání kamer LATTICE (živý přenos)** | Ano (záložka Kamery) | Ano (`chloros-cli lattice`, SDK) | Ano | Ano |
| **Světelné senzory DAQ (živě)** | Ano (karta Světelné senzory) | Ano (`chloros-cli daq pool-*`, SDK) | Ano | Ano |
| **Časová synchronizace PTP (hostitel je grandmaster)** | Ano | Ano (`chloros-cli time-sync`) | Ano | Ano |
| **Akcelerace GPU (CUDA)** | Ano | Ano | Ano | Ano (JetPack 6) |
| **Debayer s podporou textur** | Ano (Chloros+) | Ano (Chloros+) | Ano (Chloros+) | Ano (Chloros+) |
| **Dynamická adaptace výpočtů** | Ano | Ano | Ano | Ano |
| **Backend jako systémová služba** (`chloros-backend.service`) | Ne | Ne | Ano (na přání) | Ano (na přání) |
| **Aktualizace na místě** (`chloros-cli update`) | Ne (spusťte instalační program) | Ne (spusťte instalační program) | Ano | Ano |***

## Podporované architektury

| Architektura | Popis | Balíček |
| --- | --- | --- |
| **amd64 (x86_64)** | Standardní procesory pro stolní počítače a servery (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | Procesory ARM — řada NVIDIA Jetson Orin | `chloros_<version>_arm64_jp6.deb` (sestavení JetPack 6) |

## Podporované distribuce Linux

* **Ubuntu 22.04 LTS nebo novější** (amd64)
* **Debian 12 nebo novější** (amd64)
* **NVIDIA JetPack 6** (arm64 — platformy Jetson Orin)***

## Co získají uživatelé Linux

* **Chloros CLI** — kompletní rozhraní příkazového řádku pro dávkové zpracování, automatizaci a skriptování
* **Chloros Python SDK** — programové rozhraní Python pro výzkumné pracovní postupy a vlastní nástroje (lze nainstalovat z PyPI a je také součástí balíčku `.deb` jako wheel s odpovídající verzí)
* **Ovládání kamer LATTICE** — vyhledávání, připojování, konfigurace a snímání z kamer LATTICE a synchronizovaných soustav více kamer prostřednictvím `chloros-cli lattice` a SDK; `.deb` obsahuje runtime Arena SDK, který kamery vyžadují
* **Ovládání světelných senzorů DAQ** — připojujte senzory DAQ-U/M/E, streamujte kalibrovaná spektra a zaznamenávejte soubory `.daq` pomocí `chloros-cli daq pool-*` a SDK
* **Časová synchronizace PTP** — backend Chloros provozuje PTP grandmaster, ke kterému jsou kamery LATTICE a senzory DAQ-E připojeny jako slave; zkontrolujte jej pomocí `chloros-cli time-sync`, a udržujte jej v bezobslužném provozu pomocí jednotky systemd `chloros-backend.service` (viz [Instalace Linux](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Automatizace projektů** — spouštějte uložené projekty v bezobslužném režimu pomocí `chloros-cli project` a `open_project` z balíčku SDK
* **Akcelerace pomocí GPU** — zpracování akcelerované technologií CUDA na grafických kartách NVIDIA (stolní počítače a Jetson)
* **Dynamická adaptace výpočtů** — automatická detekce hardwaru a výběr strategie zpracování, s možností přepsání pomocí `CHLOROS_STRATEGY` jako nouzové cesty pro odborníky
* **Všechny funkce zpracování** — stejný proces jako u Windows: kalibrace, korekce vinětace, vegetační indexy a všechny formáty exportu
* **Funkce Chloros+** — vícevláknové (pipeline) zpracování, debayer s podporou textur a vlastní indexy, v rámci placeného tarifu Chloros+

## Co uživatelé Linux nedostanou

* **Grafické uživatelské rozhraní** — žádné grafické rozhraní; veškerá interakce probíhá přes CLI nebo Python SDK
* **Prohlížeč obrázků** — žádný interaktivní prohlížeč obrázků, zobrazení v mřížce ani značky na mapě
* **Vizuální správa projektů** — projekty se vytvářejí a řídí pomocí příkazů CLI a volání SDK (samotný hardware — kamery, senzory, snímání — zůstává plně ovladatelný z terminálu)***

## Požadavky na licenci

Přístup k CLI a SDK vyžaduje **placenou úroveň Chloros+ — Copper nebo vyšší**(Copper, Bronze, Silver, Gold). Bezplatná úroveň**Iron** nemá přístup k funkcím CLI/SDK. Toto omezení je vynucováno na backendové úrovni, nikoli pouze v rámci CLI:

| Situace | Odpověď backendu |
| --- | --- |
| Není přihlášeno | `401` s `error_code: AUTH_REQUIRED` |
| Přihlášeno na bezplatné úrovni Iron | `403` s `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` funguje na jakémkoli tarifu — je to jediná trasa, na kterou se brána nevztahuje — takže důvod odmítnutí je vždy viditelný.

***

## Začínáme s Linux

1. **Nainstalujte Chloros** — viz [Instalace Linux](linux-installation.md) pro instalaci `.deb`
2. **Ověřte** — `chloros-cli --version` vytiskne `Chloros CLI 1.2.0`; `chloros-cli selftest` spustí 7krokovou diagnostiku
3. **Nainstalujte Python a SDK** (volitelné) — `pip install chloros-sdk`
4. **Přihlaste se** — `chloros-cli login your@email.com 'your-password'` (jednou na každém zařízení a znovu po každé aktualizaci balíčku)
5. **Zpracujte svůj první datový soubor** — `chloros-cli process ~/datasets/flight001`

Pro NVIDIA Jetson viz speciální [Průvodce NVIDIA Jetson](nvidia-jetson-guide.md) s informacemi o nastavení specifickém pro danou platformu, tepelném chování a nasazení v terénu.

***

## Další kroky

* [Instalace Linux](linux-installation.md) — podrobný návod k instalaci, umístění souborů a řešení problémů pro architektury amd64 a arm64
* [Příručka NVIDIA Jetson](nvidia-jetson-guide.md) — nastavení specifické pro Jetson, chování paměti a tepelné vlastnosti, nasazení v terénu
* [CLI : Příkazový řádek](../CLI.md) — průvodce CLI
* [API : Python SDK](../api-python-sdk.md) — příručka SDK
* [CLI Referenční příručka](../reference/cli-reference.md) a [SDK Referenční příručka](../reference/sdk-reference.md) — vyčerpávající seznam příkazů pro verzi 1.2.0
* [Dynamická adaptace výpočetního výkonu](../processing-architecture/dynamic-compute-adaptation.md) — jak se Chloros přizpůsobuje vašemu hardwaru

{% hint style="info" %}
**Programové čtení této příručky.** Každá stránka je také k dispozici jako nezpracovaný Markdown na své vlastní stránce URL a `.md` (například `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), přičemž rejstřík celého manuálu je zveřejněn na [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
