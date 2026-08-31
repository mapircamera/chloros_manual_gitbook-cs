# Záznam a formát .daq

Soubor `.daq` je formát záznamu světelného senzoru Chloros: jedná se o **databázi SQLite** obsahující kalibrované spektrální snímky z jednoho senzoru DAQ. Pokud takový soubor zaznamenáte během snímací relace, může proces zpracování odrazivosti později každý snímek vydělit intenzitou slunečního záření dopadajícího na zem naměřenou v daném okamžiku.

## Co obsahuje soubor .daq

| Vlastnost | Hodnota |
| --- | --- |
| Kontejner | Databáze SQLite, jeden soubor na senzor a na záznam |
| Název souboru | Obsahuje **ID senzoru**a**časové razítko**, např. `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spektrum na snímek | 135 bodů, 340–1010 nm v krocích po 5 nm, plus tristimulus CIE XYZ |
| Jednotky | Kalibrované spektrální ozáření, **W/m²/nm** (aplikován balíček tovární kalibrace + profil krytu) |
| Vložená metadata | ID senzoru (klíč pro načtení tovární kalibrace dané jednotky) a platný profil krytu — viz [Profily krytů a kalibrovaný rozsah](caps-and-range.md) |

Formát je identický pro DAQ-U, DAQ-M i DAQ-E, takže při následném zpracování nezáleží na tom, který přenosový modul záznam pořídil.

Kalibrovaný záznam vyžaduje balíček tovární kalibrace snímače. U modelů DAQ-U a DAQ-M backend načte sadu z cloudu MAPIR podle ID senzoru (pokud se to nepodaří, je záznam odmítnut); jednotky DAQ-E jsou z tohoto pravidla vyňaty, protože si kalibraci uchovávají přímo v zařízení.

## Záznam z grafického uživatelského rozhraní

Záznam v grafickém uživatelském rozhraní vyžaduje **otevřený projekt** (v opačném případě jsou tlačítka pro záznam deaktivována):

* **Záznam vše / Zastavit vše** — v horní části postranního panelu světelných senzorů; spustí nebo zastaví záznam `.daq` na všech připojených senzorech najednou.
* **Záznam / Zastavit záznam** — pro jednotlivé senzory, v modálním okně nastavení (ikona ozubeného kolečka). Během záznamu se v řádcích s aktuálními informacemi o senzoru zobrazuje červený indikátor „REC“.

Soubory se ukládají do složky `<project>/light_sensor/` a po zastavení záznamu — ať už pomocí tlačítka Zastavit, Zastavit vše nebo odpojením záznamového senzoru — je hotový soubor `.daq` **automaticky přidán do otevřeného projektu**. Objeví se v seznamu souborů projektu bez nutnosti ručního přidávání a je již připraven k zpracování odrazivosti.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->

## Záznam ze zařízení CLI

CLI provádí záznam prostřednictvím skupiny senzorů backendu (backend musí být spuštěn – tyto příkazy jsou odlehčenými klienty HTTP):

```bash
# Connect the sensor into the backend pool
chloros-cli daq pool-connect --eth-host daq-e-def330.local

# Record for 150 seconds, with a human-friendly device label
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
    -o ./out --device-name "rooftop-A"

# Or run open-ended and stop explicitly
chloros-cli daq pool-record --sensor-id daq-e-def330            # --duration defaults to 0 = run until --stop
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

Získejte hodnotu `--sensor-id` z `chloros-cli daq pool-list`. Dvě výchozí hodnoty, které stojí za zmínku:

| Možnost | Výchozí hodnota |
| --- | --- |
| `--duration` | `0` — zaznamenávat až do `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` na souborovém systému **backendu**, nikoli na CLI |

Rozdíl ve výstupním adresáři má význam, když CLI cílí na backend na jiném počítači: soubor se uloží tam, kde běží backend.

## Záznam z Python

`DAQSensorSession` (vrácený funkcí `chloros_sdk.connect_daq_sensor()`) zpřístupňuje stejný sdílený záznam: `record_start(output_dir=None, device_name=None)` vrací cestu k souboru, `record_stop()` vrací `{path, rows}`. Úplnou relaci API najdete v [Referenci SDK](../reference/sdk-reference.md). Třídy přímého hardwaru SDK (pouze pro instalace na stolních počítačích) zapisují záznamy ve výchozím nastavení do `~/Documents/DAQ/`; u vydaných verzí je podporovanou cestou výše uvedená sdílená cesta.

## Použití souboru .daq při zpracování

K výpočtu odrazivosti ze snímků potřebuje Chloros hodnotu sestupného záření odpovídající každé expozici:

* **Uchovejte soubor `.daq` společně se snímky.**Při zpracování proces automaticky určí**sluneční záření směřující dolů s odpovídajícím časovým razítkem** na základě zaznamenaného souboru `.daq` (libovolný model DAQ) — nebo na základě nativního souboru `.csv` — nalezeného společně se snímky. Záznamy z grafického uživatelského rozhraní (GUI) tuto podmínku splňují automaticky, protože jsou do projektu přidány ihned po ukončení.
* **Kalibrace se načítá na vyžádání.** Pokud balíček tovární kalibrace pro danou kameru nebo DAQ ještě není lokálně uložen v mezipaměti, Chloros jej při prvním použití automaticky stáhne z cloudu MAPIR (jednou je nutné připojení k internetu; uloženo do mezipaměti pod `~/.chloros/`).
* **Živé snímky ukládají vlastní doprovodný soubor.** U každého snímku odrazivosti pořízeného v reálném čase se skutečně použitá hodnota z DAQ uloží jako doprovodný soubor `.daq` vedle samotného snímku, takže snímek lze později znovu zpracovat i bez původního záznamu.

## Získání irradiance zpět

Zpracování projektu také exportuje všechny záznamy ze světelných senzorů, které obsahuje, do
složky `Light Sensor/` vedle obrazových produktů. K tomu **není** zapotřebí snímky:
světelný senzor letící samostatně představuje kompletní záznam a složka obsahující pouze soubory `.daq`
je platným vstupem. Běh vykazuje, kolik produktů ze světelných senzorů zapsal.

| Produkt | Co to je |
| --- | --- |
| `<name>_calibrated.daq` | Archiv vhodný k dalšímu zpracování ve stejném schématu jako živý záznam, nyní s deklarací kalibračního balíčku, který jej vytvořil. Jeho opětovný import **ne** provede druhou kalibraci. |
| `<name>_calibrated.csv` | Spektrální ozáření v W/m²/nm na vlastní vlnové mřížce senzoru, jeden řádek na jedno měření, plus fotometrické sloupce: celkový výkon, fotopický a skotopický lux, PPFD s rozdělením na modrou, zelenou a červenou složku a špičková vlnová délka. |

DAQ-U nebo DAQ-M, u nichž nelze načíst kalibrační balíček — jste offline nebo
daný senzor nemá v souboru žádnou kalibraci — jsou **přeskočeny s uvedením důvodu**, nikdy nejsou zapsány
jako „kalibrovaný“ soubor obsahující surová měření. Připojte se k internetu a spusťte proces znovu. DAQ-E
má vlastní kalibraci, takže ji potřebuje pouze v případě, že není připojen a
v lokální paměti není nic uloženo.

### DAQ-A: surové počty a proč je to správné řešení

**DAQ-A** vznikl ještě před zavedením systému kalibračních balíčků pro jednotlivé sériové porty a nemá žádný balíček, který by bylo možné
načíst. Nejedná se o opomenutí: přístroj DAQ-A se kalibruje v terénu pomocí
reflexního terče a kalibrace založená na terči vyžaduje pouze *relativní*
odpověď senzoru — což přesně odpovídá jeho surovým počtům. Chloros se s nimi dnes kalibruje.

Záznam z DAQ-A se tedy exportuje, ale pod jiným názvem:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, nikoli `_calibrated` — jde o odlišný název souboru, nikoli o příznak uvnitř souboru,
protože tato informace musí zůstat zachována i při odeslání souboru e-mailem pouze jako holý název. Záhlaví `.csv`
uvádí `raw spectral sensor counts (NOT irradiance)` a upozorňuje, že hodnoty jsou
srovnatelné **v rámci** souboru, nikoli mezi senzory. Sloupce, které mají význam
pouze pro skutečnou intenzitu záření — celkový výkon, lux, PPFD — jsou ponechány prázdné, místo aby byly
vypočítány z počtů.

Starší záznamy DAQ-A-SD (schéma v1.01 / v1.02) zaznamenávají pouze čas zápisu souboru, nikoli
časové razítko pro každé měření. Chloros nebude snímky s těmito údaji porovnávat — přiřazení snímku k
času zápisu by bylo nesprávné, i když by to na první pohled vypadalo správně — ale export je čte bez problémů a
CSV uvádí, o jaký typ hodin se jedná.

Kompletní informace o odrazivosti – jeden senzor s kamerou a dva senzory pro okolí/objekt – najdete v [Pracovních postupech pro odrazivost](reflectance.md).
