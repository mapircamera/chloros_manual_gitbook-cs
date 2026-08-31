# Světelné senzory DAQ

> **Hledáte informace o hardwaru?**Samotné senzory – modely, způsob montáže, krytky, porty, napájení a aplikace SCANNER – jsou popsány v**[uživatelské příručce k DAQ](https://mapir.gitbook.io/daq)**. Tato kapitola se zabývá jejich používáním od modelu Chloros.

Světelné senzory **DAQ** z MAPIR měří okolní světlo jako radiometricky kalibrovaná spektra. V Chloros plní dvě funkce:

* **Samostatný spektrální přístroj** — grafy spektra v reálném čase, kolorimetrická data a záznamy `.daq`, to vše z [kartu Světelné senzory](gui.md), [CLI](cli-quick-start.md) nebo z Python a SDK.
* **Zdroj sestupného záření pro odrazivost** — během zpracování Chloros interpoluje vaše naměřené hodnoty `.daq` na časové razítko expozice každého snímkučasového razítka expozice a pomocí naměřeného slunečního záření převádí radianci kamery na odrazivost (`--reflectance-source daq`); pro kalibrovaná pásma není nutný panel v záběru.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Tři modely, jeden datový formát

| Model | Přenos | Objev |
| --- | --- | --- |
| **DAQ-U** | USB (sériové) | skenování sériového portu |
| **DAQ-M** | Bluetooth Low Energy | skenování BLE podle názvu |
| **DAQ-E** | Ethernet (IPv4, napájení přes PoE) | mDNS `_daq-e._tcp` (název hostitele `daq-e-<id>.local`) |

Všechny tři používají stejný komunikační protokol a poskytují identická data:

* **135bodové spektrum v rozsahu 340 až 1010 nm s krokem 5 nm**, plus tristimulační hodnoty CIE XYZ, v každém snímku.
* **Radiometricky kalibrovaná spektrální intenzita záření v W/m²/nm** — předtím, než se data dostanou k vám, je aplikován tovární kalibrační balíček každého přístroje (včetně jeho aktivního profilu korekce krytky).
* Stejný **formát záznamu `.daq`** (soubor SQLite). Následné zpracování je identické bez ohledu na to, jaký přenosový kanál soubor vytvořil.

Přenosové vrstvy (sériové USB, BLE, mDNS/zeroconf) jsou součástí backendu Chloros — není třeba nic instalovat, abyste mohli komunikovat s kterýmkoli z těchto tří modelů prostřednictvím grafického uživatelského rozhraní nebo příkazů `pool-*` v CLI.

***

## Kalibrovaný rozsah: hlášený 340–1010 nm, kalibrovaný ~374–974 nm

Senzor hlásí plnou mřížku 340–1010 nm, ale radiometrický zisk sledovatelný podle NIST se rozprostírá přibližně v rozsahu **374–974 nm**. Chloros odmítá výpočet absolutní odrazivosti pro jakékoli pásmo kamery, u kterého méně než polovina spektrální váhy spadá do tohoto kalibrovaného rozsahu; vynechané pásmo je nahlášeno s důvodem vynechání `dls-uncalibrated-band-<nm>`.

Z dodávaných filtrů LATTICE se to týká pouze modelu **F988**:

Odrazivost modelu F988 je kalibrována pomocí panelu odrazivosti umístěného v dané scéně: dané pásmo leží mimo kalibrovaný rozsah světelného senzoru DAQ, proto Chloros použije váš nejnovější záznam z panelu a zachová jej mezi jednotlivými měřeními panelu.

Pokud je snímání F988 zpracováváno pouze s dostupnými daty DAQ, Chloros odmítne odrazivost založenou na DAQ pro dané pásmo s důvodem přeskočení `dls-uncalibrated-band-988` — [pracovní postup s panelem odrazivosti](../calibration-targets.md) je pro F988 podporovanou cestou.

***

## ID senzorů

Každý DAQ hlásí stabilní ID senzoru. Jeho tvar se liší podle modelu:

| Model | Tvar ID | Příklad |
| --- | --- | --- |
| DAQ-U | 5 oktetů s pomlčkami | `CB-7C-A8-2E-5F` |
| DAQ-M | 5 oktetů s pomlčkami | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

ID senzoru je:

* vtisknuto do každého souboru `.daq`, který zaznamenává,
* klíč, který Chloros používá k načtení balíčku tovární kalibrace daného zařízení,
* hodnota, kterou předáváte do `--sensor-id` v příkazech CLI a `pool-*`, a
* u jednotky DAQ-E také její název hostitele mDNS (`daq-e-def330.local`) — hodnotu, kterou přijímá `--eth-host`.

***

## Tovární kalibrace a cloud

Každá jednotka DAQ je individuálně kalibrována z výroby pomocí radiometrického řetězce sledovatelného podle NIST a Chloros načítá kalibrační balíček každé jednotky podle jejího ID senzoru. Kalibrační protokol pro jednotlivé jednotky (PDF) lze stáhnout z nastavení senzoru na [kartě Světelné senzory](gui.md).

{% hint style="warning" %}
**Modely DAQ-U a DAQ-M vyžadují pro kalibraci přístup k cloudu.**Žádný z těchto modelů neukládá data na paměťové médium: jejich tovární kalibrační balíčky jsou uloženy v cloudu MAPIR a jsou načítány podle ID senzoru (následně se ukládají do lokální mezipaměti). Chloros potřebuje připojení k internetu, aby mohl poskytovat kalibrovaná data v jednotkách W/m²/nm z DAQ-U nebo DAQ-M.**Výjimkou je DAQ-E** — ten má kalibraci uloženou přímo v zařízení.

<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

{% endhint %}***

## Kam se ukládají záznamy

| Povrch | Výchozí cíl `.daq` |
| --- | --- |
| Grafické uživatelské rozhraní — záložka Světelné senzory | `<project folder>/light_sensor/` (dokončené záznamy se do projektu přidávají automaticky) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` na stroji, na kterém běží backend |

Každý název souboru `.daq` obsahuje ID snímače a časové razítko.

***

## V této kapitole

* [**Karta DAQ v Chloros**](gui.md) — kompletní průvodce grafickým uživatelským rozhraním: připojení jednotlivých modelů, nastavení jednotlivých senzorů, spektrální grafy, živá kolorimetrická data, odrazivost dvou senzorů a záznam.
* [**Rychlý start CLI (pool-\*)**](cli-quick-start.md) — ovládání senzorů DAQ z `chloros-cli daq pool-*`, podporovaná cesta z příkazového řádku.
* [**Profily limitů a kalibrovaný rozsah**](caps-and-range.md) — které limity existují pro jednotlivé modely, jak je deklarovat a podrobnosti o kalibrovaném spektrálním rozsahu.
* [**Záznam a formát .daq**](recording.md) — formát SQLite `.daq` a pracovní postupy při záznamu.
* [**Síťové propojení DAQ-E a časová synchronizace**](ethernet-ptp.md) — přenosové režimy DAQ-E a časová synchronizace PTP.
* [**Pracovní postupy pro odrazivost**](reflectance.md) — využití dat DAQ pro sestupné záření k výpočtu odrazivosti.
* Kompletní dokumentaci na úrovni příznaků najdete v [CLI Referenci](../reference/cli-reference.md) (sekce `chloros-cli daq`) a v [referenci k SDK](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), přičemž obě jsou napsány tak, aby je mohly přímo využívat asistenti umělé inteligence.
