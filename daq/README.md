# Světelné senzory DAQ

> **Hledáte informace o hardwaru?**Samotné senzory – modely, montáž, krytky, porty, napájení a aplikace SCANNER – jsou popsány v**[uživatelské příručce k DAQ](https://mapir.gitbook.io/daq)**. Tato kapitola se zabývá jejich používáním v aplikaci Chloros. Světelné senzory**DAQ** od společnosti

MAPIR měří okolní světlo jako radiometricky kalibrovaná spektra. V aplikaci Chloros plní dvě funkce:

* **Samostatný spektrální přístroj** — živé spektrální grafy, kolorimetrická data a záznamy `.daq`, to vše z [kartu Světelné senzory](gui.md), [CLI](cli-quick-start.md) nebo Python SDK.
* **Zdroj dopadajícího záření pro odrazivost** — během zpracování Chloros interpoluje vaše naměřené hodnoty `.daq` na časový údaj expozice každého snímku a pomocí naměřeného dopadajícího světla převádí radianci kamery na odrazivost (`--reflectance-source daq`); pro kalibrované pásma není nutný panel v rámci scény.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Tři modely, jeden datový formát

| Model | Přenos | Vyhledávání |
| --- | --- | --- |
| **DAQ-U** | USB (sériové) | skenování sériového portu |
| **DAQ-M** | Bluetooth Low Energy | skenování BLE podle názvu |
| **DAQ-E** | Ethernet (IPv4, napájení přes PoE) | mDNS `_daq-e._tcp` (název hostitele `daq-e-<id>.local`) |**DAQ-E-S (novinka ve verzi 1.3.0).** DAQ-E s pevným difuzorem a snímačem náklonu. V Chlorosu je uveden jako `DAQ-E-S` a funguje přesně jako DAQ-E, s tím rozdílem, že neprovádí korekci krytu a každá naměřená hodnota obsahuje údaje o náklonu, bočním naklonění a sklonu jednotky. Viz [DAQ-E-S: pevný difuzor a náklon](caps-and-range.md#daq-e-s-fixed-diffuser-and-tilt).

Všechny tři používají stejný komunikační protokol a poskytují identická data:

* **135bodové spektrum v rozsahu 340 až 1010 nm s krokem 5 nm**, plus tristimulační hodnoty CIE XYZ, v každém snímku.
* **Radiometricky kalibrovaná spektrální intenzita záření v W/m²/nm** — předtím, než se data dostanou k vám, je na ně aplikován tovární kalibrační balíček každého přístroje (včetně jeho aktivního profilu korekce krytky).
* Stejný **formát záznamu `.daq`** (soubor SQLite). Následné zpracování je identické bez ohledu na to, který přenosový kanál soubor vytvořil.

Přenosové protokoly (sériové USB, BLE, mDNS/zeroconf) jsou integrovány do backendu Chloros — není třeba nic instalovat, abyste mohli komunikovat s kterýmkoli ze tří modelů prostřednictvím grafického rozhraní nebo příkazů `pool-*` v CLI.

***

## Kalibrovaný rozsah: hlášený 340–1010 nm, kalibrovaný ~374–974 nm

Senzor hlásí plnou mřížku 340–1010 nm, ale radiometrický zisk sledovatelný podle NIST se rozprostírá přibližně v rozmezí **374–974 nm**. Chloros odmítá rozdělení na absolutní odrazivost u jakéhokoli pásma kamery, jehož spektrální váha v tomto kalibrovaném rozsahu nedosahuje ani poloviny; vynechané pásmo je hlášeno s důvodem vynechání `dls-uncalibrated-band-<nm>`.

Z dodávaných modelů filtrů LATTICE se to týká pouze modelu **F988**:

Odrazivost modelu F988 je kalibrována pomocí panelu odrazivosti umístěného v místě snímání: pásmo leží mimo kalibrovaný rozsah světelného senzoru DAQ, proto Chloros použije váš nejnovější záznam panelu a zachová jej mezi jednotlivými měřeními panelu.

Pokud je snímání F988 zpracováváno pouze s dostupnými daty DAQ, Chloros odmítne odrazivost založenou na DAQ pro dané pásmo s důvodem vynechání `dls-uncalibrated-band-988` — podporovaným postupem pro F988 je [pracovní postup s panelem odrazivosti](../calibration-targets.md).

***

## ID senzorů

Každý DAQ hlásí stabilní ID senzoru. Jeho formát se liší podle modelu:

| Model | Formát ID | Příklad |
| --- | --- | --- |
| DAQ-U | 5 oktetů s pomlčkami | `CB-7C-A8-2E-5F` |
| DAQ-M | 5 oktetů s pomlčkami | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

ID senzoru je:

* vloženo do každého souboru `.daq`, který zaznamenává,
* klíč, který Chloros používá k načtení balíčku tovární kalibrace dané jednotky,
* hodnota, kterou předáváte `--sensor-id` v příkazech CLI, a
* u DAQ-E také jeho mDNS hostname (`daq-e-def330.local`) — hodnota, kterou `--eth-host` přijímá.

***

## Tovární kalibrace a cloud

Každá jednotka DAQ je individuálně kalibrována ve výrobě pomocí radiometrického řetězce sledovatelného podle NIST a společnost Chloros načítá kalibrační balíček každé jednotky podle jejího ID senzoru. Kalibrační protokol pro jednotlivé jednotky (PDF) lze stáhnout z nastavení senzoru na [kartě Světelné senzory](gui.md).

{% hint style="warning" %}
**Modely DAQ-U a DAQ-M vyžadují pro kalibraci přístup do cloudu.**Žádný z těchto modelů neukládá data přímo v paměti: jejich tovární kalibrační balíčky jsou uloženy v cloudu společnosti MAPIR a jsou načítány podle ID senzoru (následně se ukládají do lokální mezipaměti). Chloros potřebuje připojení k internetu, aby mohl poskytovat kalibrovaná data v jednotkách W/m²/nm ze zařízení DAQ-U nebo DAQ-M.**Výjimkou je model DAQ-E** — ten má kalibrační údaje uloženy přímo v zařízení.
{% endhint %}
<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

***

## Kam se ukládají záznamy

| Povrch | Výchozí cíl `.daq` |
| --- | --- |
| Grafické uživatelské rozhraní — záložka Světelné senzory | `<project folder>/light_sensor/` (hotové záznamy se do projektu přidávají automaticky) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` na počítači, na kterém běží backend (relativní `-o` se vyhodnocuje podle složky, ve které jste příkaz spustili, pokud je backend na stejném počítači) |

Každý název souboru `.daq` obsahuje ID senzoru a časové razítko.

***

## V této kapitole

* [**Karta DAQ v programu Chloros**](gui.md) — kompletní průvodce grafickým uživatelským rozhraním: připojení jednotlivých modelů, nastavení jednotlivých senzorů, spektrální grafy, živá kolorimetrická data, odrazivost dvou senzorů a záznam.
* [**Rychlý start CLI (pool-\*)**](cli-quick-start.md) — ovládání senzorů DAQ z `chloros-cli daq pool-*`, podporovaná cesta příkazového řádku.
* [**Profily krytek a kalibrovaný rozsah**](caps-and-range.md) — které krytky jsou k dispozici pro jednotlivé modely, jak je deklarovat a podrobný popis kalibrovaného spektrálního rozsahu.
* [**Záznam a formát .daq**](recording.md) — formát SQLite `.daq` a pracovní postupy při záznamu.
* [**Síťové propojení DAQ-E a časová synchronizace**](ethernet-ptp.md) — přenosové režimy DAQ-E a časová synchronizace PTP.
* [**Pracovní postupy pro odrazivost**](reflectance.md) — využití dat DAQ o dopadajícím záření k výpočtu odrazivosti.
* Kompletní dokumentaci na úrovni příznaků najdete v [Referenci CLI](../reference/cli-reference.md) (sekce `chloros-cli daq`) a v [Referenci SDK](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), které jsou napsány tak, aby je mohly přímo využívat AI asistenti.
