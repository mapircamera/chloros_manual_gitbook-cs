# Profily krytek a kalibrovaný rozsah

> Samotné krytky — které se dodávají s jakým senzorem, jak se montují a jaké mají optické vlastnosti — jsou popsány v **[uživatelské příručce k DAQ](https://mapir.gitbook.io/daq)**. Tato stránka se zabývá *deklarací* namontované krytky v systému Chloros, což je nezbytné pro správnou korekci.

Tovární radiometrická kalibrace každého světelného senzoru DAQ popisuje *holý* senzor. Fyzický kryt nasazený na difuzor mění světlo, které senzor snímá, proto Chloros aplikuje na kalibrační balíček **profil korekce krytu** naměřený při výrobě. Nahlášení správného krytu je součástí získávání kalibrovaných dat – tato stránka popisuje, jaké kryty existují pro jednotlivé modely, jak je nahlásit a jaký je skutečný kalibrovaný spektrální rozsah snímače.

## Dostupnost krytek podle modelu

| Profil krytky (`cap_id`) | Fyzická krytka | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Krytka pro korekci kosinusu slunečního záření (**výchozí u všech modelů**) | Ano | Ano | Ano |
| `fov_15` / `fov_45` / `fov_90` | Kužely omezující zorné pole (15° / 45° / 90°) | Ano | — | Ano |
| `fov_30` / `fov_60` | Kužely omezující zorné pole (30° / 60°) | Ano | — | — |
| `none` | Bez nasazeného krytu | — | — | Ano |

Poznámky k jednotlivým modelům:

* **DAQ-M má jeden profil krytu: `sunshine_cosine`.** „Bare-plus-Sunshine-cap“ je jeho produktová definice a holý DAQ-M nepotřebuje žádný geometrický profil.
* **Holý DAQ-U je skutečně holý** — nepotřebuje vůbec žádný geometrický profil, proto pro něj neexistuje profil `none`.
* **`none` na DAQ-E NENÍ nefunkční.** Zapuštěný difuzor DAQ-E pokrytý sklem má vlastní skutečnou geometrickou korekci, takže „bez krytu“ je u tohoto modelu samo o sobě měřeným profilem.
* **Holý snímač DAQ-E nedokáže měřit přímé sluneční záření v žádné výšce** — krytka Sunshine je součástí konfigurace pro terénní použití. Neplánujte práci venku s holým snímačem DAQ-E.

V nastavení jednotlivých senzorů v grafickém uživatelském rozhraní (ikona ozubeného kola na záložce Světelné senzory) nabízí rozevírací seznam **Kryt** u modelů DAQ-U a DAQ-M také možnost „Žádný (holý senzor)“ — u těchto dvou modelů „holý“ jednoduše znamená, že se neuplatňuje žádná korekce krytu, jak je uvedeno výše. Tuto možnost vyberte pouze v případě, že je krytka fyzicky odstraněna.

## Deklarace krytky — a proč je to důležité

**Deklarovaný kód `cap_id` musí odpovídat krytce, která je fyzicky nasazena na senzoru.** Ani snímač, ani software nedokážou detekovat nasazený kryt. Deklarace ovlivňuje dvě věci:

1. **Korekci v reálném čase** aplikovanou na každé spektrum.
2. **Zápis o krytu zapsaný do každého záznamu `.daq`**, na který se spoléhá následné zpracování odrazivosti.

Krytka Sunshine má **konstrukčně**útlum přibližně**12×**, takže záznam s nesprávně deklarovanou krytkou způsobí nesprávné škálování spekter přibližně o tento faktor. Změny krytky ihned deklarujte.

### Nastavení krytky

GUI: záložka Světelné senzory → ikona ozubeného kola v řádku senzoru → rozevírací nabídka **Krytka**. Výchozí hodnota pro každý model je `sunshine_cosine`, což je předpoklad (modely DAQ-U, DAQ-M a DAQ-E se dodávají bez krytky a záznam uvádí, že krytka byla předpokládána, dokud si žádnou nevyberete), a tento výběr zůstává zachován v rámci projektu.



CLI<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->
(backend musí běžet):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

CLI syntakticky akceptuje celý seznam `cap_id` (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); každý profil je při připojení ověřen podle modelu senzoru, takže nedostupné ID krytu (například ID určené pouze pro E na DAQ-U) vyhodí jasnou chybu, místo aby došlo k nesprávné korekci. Výchozí hodnota backendu, pokud není předáno nic, je `sunshine_cosine`.

Python SDK: při připojení pojmenujte kapu pomocí `connect_daq_sensor(cap_id="sunshine_cosine")`, nebo ji za běhu změňte pomocí `DAQSensorSession.set_cap("fov_45")`; viz [Referenční příručka SDK](../reference/sdk-reference.md#daq-sensor-sessions).

Pokročilé: profily jsou součástí instalace Chloros pod názvem `daq/cap_profiles/<u|m|e>/<cap_id>.json` a lze je přepsat pro jednotlivé uživatele pomocí `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Nezávisle na mezních hodnotách dostávají senzory, které nikdy nebyly rekalibrovány, automaticky malé vylepšení tmavého posunu odvozené z celé flotily – bez jakéhokoli zásahu uživatele.

## DAQ-E-S: pevný difuzor a náklon

**DAQ-E-S** je zařízení DAQ-E se dvěma rozdíly, které jsou pro Chloros důležité. Všechno ostatní (vyhledávání, připojení, PTP, záznam, nastavení zařízení) funguje přesně stejně jako u DAQ-E.

* **Jeho difuzor je pevně namontovaný, takže nevyžaduje žádnou korekci krytu.** Difuzor byl na jednotce již v době měření její tovární kalibrace, takže tato kalibrace již udává skutečné ozáření směřující dolů. Chloros na něj neaplikuje žádný profil omezení, ať je požadováno cokoli: omezení pojmenované pro něj je ignorováno s hlášením, jeho záznamy uvádějí, že omezení pochází z modelu, a varování o předpokládaném omezení se u něj nikdy nespustí. V nastavení projektu → Světelný senzor DAQ se v jeho řádku nezobrazuje žádný limit a celoprojektové přepsání limitu se na něj nevztahuje. Nepoužívejte na jeho data ručně žádný limit: korekce slunečního svitu by v takovém případě vykazovala přibližně 11× vyšší hodnoty.
* **Je vybaven tříosým akcelerometrem.** Kosínusová odezva difuzoru platí pouze tehdy, je-li v horizontální poloze, takže nakloněný senzor hlásí nesprávnou intenzitu záření bez jakýchkoli dalších příznaků. Každé měření DAQ-E-S proto obsahuje údaje o naklonění, bočním naklonění a sklonu, a záznam `.daq` z jednoho takového senzoru je ukládá vedle každého spektra (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, …), takže můžete rozlišit mrak od nakloněného držáku.

Informace o všech polohových polích a způsobu hlášení poruchy v datovém toku polohy najdete v [Referenční příručce k CLI pro DAQ-E-S](../reference/cli-reference.md#daq-e-s--fixed-diffuser-and-tilt-sensor).

## Výkon sluneční clony (konfigurace pro venkovní použití)

Čísla, na nichž můžete založit postupy:

| Vlastnost | Hodnota |
| --- | --- |
| Zorné pole | 180° polokulové |
| Chyba kosinové odezvy | ≤ ±4 % do úhlu dopadu 60°; ≤ ±4,5 % do úhlu dopadu 70° |
| Mez pro nízké slunce | Nedoporučuje se pod ~15° elevace slunce |
| Útlum | ~12× (podle konstrukce) |
| Opakovatelnost opětovné montáže krytu | ≈ 1,5 % |
| Kvantitativní ozáření | Průměr z **≥ 15 s** měření (charakteristika přístroje, nejedná se o vadu) |

Pro jakoukoli hodnotu kvantitativního ozáření — včetně referenčních hodnot odrazivosti — použijte průměr z měření trvajících alespoň 15 sekund namísto jediného snímku.

## Kalibrovaný spektrální rozsah

| Vlastnost | Hodnota |
| --- | --- |
| Spektrální vzorkování | 340–1010 nm v krocích po 5 nm (135 bodů) |
| Radiometricky kalibrovaný rozsah | **~374–974 nm** (vynuceno softwarem) |

Senzor zaznamenává celou mřížku 340–1010 nm, ale radiometrický zisk sledovatelný podle NIST pokrývá rozsah ~374–974 nm. Chloros **odmítá rozdělení podle absolutní odrazivosti** u jakéhokoli pásma kamery, jehož spektrální váha v tomto rozsahu nedosahuje ani poloviny, a namísto vytvoření nekalibrovaného výstupu nahlásí důvod přeskočení `dls-uncalibrated-band-<nm>`. Z aktuálně dodávaných modelů kamer se mimo tento rozsah nachází pouze filtr F988; ten místo toho využívá pracovní postup s panelem odrazivosti — viz [Pracovní postupy s odrazivostí](reflectance.md).

Informace o modelech snímačů, transportech a ID snímačů najdete v [přehledu DAQ](README.md). Informace o tom, jak se během zpracování spotřebovává limitní počet snímků, najdete v části [Záznam a formát .daq](recording.md).
