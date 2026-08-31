# Profily krytek a kalibrovaný rozsah

> Samotné krytky – tedy to, která krytka se dodává s kterým snímačem, jak se montují a jaké mají optické vlastnosti – jsou popsány v **[uživatelské příručce k systému DAQ](https://mapir.gitbook.io/daq)**. Tato stránka se zabývá *deklarací* nasazeného krytu v Chloros, což je nezbytné pro správnou korekci.

Tovární radiometrická kalibrace každého světelného senzoru DAQ popisuje *holý* senzor. Fyzický kryt nasazený na difuzor mění světlo, které senzor snímá, a proto Chloros aplikuje na kalibrační balíček **profil korekce krytu** naměřený v továrně. Deklarace správného krytu je součástí získávání kalibrovaných dat – tato stránka popisuje, jaké kryty existují pro jednotlivé modely, jak je deklarovat a jaký je ve skutečnosti kalibrovaný spektrální rozsah snímače.

## Dostupnost krytek podle modelu

| Profil krytky (`cap_id`) | Fyzická krytka | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Krytka pro korekci kosinusu slunečního svitu (**výchozí u všech modelů**) | Ano | Ano | Ano |
| `fov_15` / `fov_45` / `fov_90` | Kužely omezující zorné pole (15° / 45° / 90°) | Ano | — | Ano |
| `fov_30` / `fov_60` | Kužely omezující zorné pole (30° / 60°) | Ano | — | — |
| `none` | Bez nasazeného krytu | — | — | Ano |

Poznámky k jednotlivým modelům:

* **DAQ-M má jeden profil krytu: `sunshine_cosine`.** „Bare-plus-Sunshine-cap“ je jeho produktová definice a holý DAQ-M nepotřebuje žádný geometrický profil.
* **Holý DAQ-U je skutečně holý** — nepotřebuje vůbec žádný geometrický profil, proto pro něj neexistuje profil `none`.
* **`none` na DAQ-E NENÍ nefunkční.** Zapuštěný, sklem zakrytý difuzér DAQ-E má vlastní skutečnou geometrickou korekci, takže „bez krytu“ je u tohoto modelu samo o sobě měřeným profilem.
* **Holý DAQ-E nedokáže měřit přímé sluneční záření v žádné výšce** — kryt Sunshine je konfigurací pro terénní použití. Neplánujte práci venku s holým DAQ-E.

V nastavení jednotlivých senzorů v grafickém uživatelském rozhraní (ikona ozubeného kola na záložce Světelné senzory) nabízí rozevírací seznam **Kryt** u modelů DAQ-U a DAQ-M také možnost „Žádný (holý senzor)“ — u těchto dvou modelů „holý“ jednoduše znamená, že se neuplatňuje žádná korekce krytu, jak je uvedeno výše. Tuto možnost vyberte pouze v případě, že je krytka fyzicky odstraněna.

## Deklarace krytky — a proč je to důležité

**Deklarované označení `cap_id` se musí shodovat s krytkou, která je fyzicky nasazena na senzoru.** Ani snímač, ani software nedokážou detekovat nasazený kryt. Deklarace ovlivňuje dvě věci:

1. **Korekci v reálném čase** aplikovanou na každé spektrum.
2. **Značku krytu zapsanou do každého záznamu `.daq`**, na kterou se spoléhá následné zpracování odrazivosti.

Krytka Sunshine **konštrukčně**tlumí přibližně**12×**, takže záznam s nesprávně deklarovanou krytkou způsobuje nesprávné škálování spekter přibližně o tento faktor. Změny krytky ihned deklarujte.

### Nastavení krytky

GUI: záložka Světelné senzory → ikona ozubeného kolečka v řádku senzoru → rozevírací nabídka **Krytka**. Výchozí hodnotou pro každý model je `sunshine_cosine` (všechny senzory DAQ se dodávají s nainstalovaným kosinovým korektorem) a tento výběr zůstává zachován v rámci projektu.

<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->

CLI (musí běžet backend):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

CLI syntakticky akceptuje celý seznam `cap_id` (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); každý profil je při připojení ověřen podle modelu senzoru, takže nedostupné ID kondenzátoru (například ID pouze typu E na DAQ-U) vyvede jasnou chybu, místo aby došlo k nesprávné korekci. Výchozí hodnota backendu, pokud není předáno nic, je `sunshine_cosine`.

Python SDK poznámka: `cap_id` **není** ovladač SDK — `connect_daq_sensor()` / `DAQSensorSession` neposkytují žádný parametr pro kondenzátor. Kondenzátor vyberte pomocí výše uvedených příkazů CLI nebo z rozevíracího seznamu v grafickém rozhraní; viz [Referenční příručka k SDK](../reference/sdk-reference.md).

Pokročilé: profily jsou součástí instalace Chloros v položce `daq/cap_profiles/<u|m|e>/<cap_id>.json` a lze je přepsat pro jednotlivé uživatele v položce `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Nezávisle na limitech senzory, které nikdy nebyly překalibrovány, automaticky obdrží malé vylepšení tmavého posunu odvozené z flotily — bez zásahu uživatele.

## Výkon limitu slunečního svitu (konfigurace pro venkovní použití)

Čísla, na nichž můžete založit postupy:

| Vlastnost | Hodnota |
| --- | --- |
| Zorné pole | 180° hemisférické |
| Chyba kosinové odezvy | ≤ ±4 % až do úhlu dopadu 60°; ≤ ±4,5 % až do úhlu dopadu 70° |
| Mez pro nízko stojící slunce | Nedoporučuje se pod ~15° elevace Slunce |
| Útlum | ~12× (podle konstrukce) |
| Opakovatelnost opětovné montáže krytu | ≈ 1,5 % |
| Kvantitativní ozáření | Průměr z **≥ 15 s** měření (charakteristika přístroje, nejedná se o vadu) |

Pro jakoukoli hodnotu kvantitativního ozáření — včetně referenčních hodnot odrazivosti — použijte průměr z alespoň 15 sekund měření namísto jediného snímku.

## Kalibrovaný spektrální rozsah

| Vlastnost | Hodnota |
| --- | --- |
| Spektrální vzorkování | 340–1010 nm v krocích po 5 nm (135 bodů) |
| Radiometricky kalibrovaný rozsah | **~374–974 nm** (vynuceno softwarem) |

Senzor zaznamenává celou mřížku 340–1010 nm, ale radiometrický zisk sledovatelný podle NIST pokrývá rozsah ~374–974 nm. Chloros **odmítá dělení podle absolutní odrazivosti** u jakéhokoli pásma kamery, u něhož méně než polovina spektrální váhy spadá do tohoto rozsahu, a namísto vytvoření nekalibrovaného výstupu nahlásí důvod vynechání `dls-uncalibrated-band-<nm>`. Z aktuálně dodávaných modelů kamer spadá mimo tento rozsah pouze filtr F988; ten místo toho využívá pracovní postup s panelem odrazivosti — viz [Pracovní postupy s odrazivostí](reflectance.md).

Informace o modelech snímačů, typech přenosu a ID snímačů najdete v [přehledu DAQ](README.md). Informace o tom, jak se při zpracování spotřebovává limit počtu snímků, najdete v části [Záznam a formát .daq](recording.md).
