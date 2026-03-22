---
metaLinks: {}
---

# Začínáme

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros je softwarová aplikace od [MAPIR](https://www.mapir.camera) určená ke zpracování obrazů a dalších dat ze senzorů.

***{% hint style="success" %}**Novinky ve verzi Chloros 1.1.0**: Nativní podpora Linux (amd64 a arm64), edge computing NVIDIA Jetson, Dynamic Compute Adaptation, 4-vláknový zpracovatelský pipeline, nové příkazy a volby CLI. Kompletní seznam změn najdete v sekci [Stáhnout](download.md).
{% endhint %}

Chloros je k dispozici ve 3 režimech aplikace:

## Chloros: Desktopová aplikace s grafickým rozhraním

Samostatné okno se všemi funkcemi. _Pouze pro Windows._

## [Chloros CLI: Rozhraní příkazového řádku](CLI.md)

Dávkové zpracování z příkazového řádku. Ideální pro automatizaci, skriptování a provoz bez grafického rozhraní. K dispozici na **Windows, Linux amd64 a Linux arm64 (NVIDIA Jetson)**. _Pro přístup k CLI je vyžadována licence Chloros+._

## [Chloros API: Python SDK](api-python-sdk.md)

Programové rozhraní Python pro automatizaci a vlastní pracovní postupy. Ideální pro výzkumné procesy, integraci se stávajícími aplikacemi Python a vytváření vlastních nástrojů. K dispozici na **všech platformách** prostřednictvím `pip install chloros-sdk`. _Pro přístup k API je vyžadována licence Chloros+._***

## Podporované platformy

| Platforma | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11** | Ano | Ano | Ano |
| **Linux amd64 (x86_64)** | Ne | Ano | Ano |
| **Linux arm64 (NVIDIA Jetson)** | Ne | Ano | Ano |

Pokyny k instalaci Linux najdete v části [Linux &amp; Edge Computing](linux/linux-overview.md).

***

## Chloros+

Ačkoli je Chloros pro většinu úkolů zdarma, možná zjistíte, že potřebujete více. Právě v takovém případě vám může být užitečná placená licence pro Chloros+. S licencí Chloros+ můžete odemknout nové funkce, jako jsou:

* **Vícevláknové zpracování**: výrazně zrychlete zpracování obrazu u větších projektů díky simultánnímu zpracování obrazů v rámci pipeline.
* **Akcelerace GPU (CUDA)**: využijte dnešní možnosti vyšší paměti GPU k dalšímu zrychlení pipeline zpracování obrazu. Pro nejlepší výsledky doporučujeme 4 GB nebo více VRAM.
* **Chloros+**[**CLI**](CLI.md)**Přístup**: spusťte Chloros+ z příkazového řádku pro automatizaci a integraci do vašeho vlastního softwaru.
* **Chloros+**[**API**](api-python-sdk.md)**Přístup:** spusťte Chloros+ z Python pro programové ovládání, což umožňuje hladkou integraci s vašimi výzkumnými procesy, pracovními postupy analýzy dat a vlastními aplikacemi.
* **Použití na více zařízeních**: každá licence Chloros+ umožňuje registraci 2 a více zařízení. K správě registrovaných zařízení použijte svůj účet MAPIR Cloud. Přidejte podporu pro další zařízení upgradem licence Chloros+.
* **Pokročilá metoda debayeringu s ohledem na textury:** vysoce kvalitní debayering s ohledem na hrany v kombinaci s modelem odšumování AI/ML, který odstraní téměř veškerý šum způsobený debayeringem. 
* **Vlastní vzorce multispektrálních indexů:** zadejte vlastní multispektrální indexy do rastrových kalkulátorů Chloros, a to jak pro zpracování, tak pro sandbox pro prohlížení obrázků.
* **Linux a edge computing:** spusťte Chloros na platformách Linux x86_64 a ARM64, včetně NVIDIA Jetson, pro zpracování v terénu a na okraji sítě. Viz [Přehled Linux](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Ceny a registrace pro Chloros+</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
