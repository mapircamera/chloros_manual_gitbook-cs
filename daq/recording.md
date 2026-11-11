# Záznam a formát .daq

Soubor `.daq` je formát záznamu světelného senzoru Chloros: **databáze SQLite** obsahující kalibrované spektrální snímky z jednoho senzoru DAQ. Zaznamenejte jeden takový soubor během snímací relace a proces zpracování odrazivosti může později každý snímek vydělit intenzitou slunečního záření dopadajícího na zem, naměřenou v přesně daném okamžiku.

## Co obsahuje soubor .daq

| Vlastnost | Hodnota |
| --- | --- |
| Kontejner | Databáze SQLite, jeden soubor na jeden senzor na jeden záznam |
| Název souboru | Obsahuje **ID senzoru**a**časové razítko**, např. `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spektrum na snímek | 135 bodů, 340–1010 nm v krocích po 5 nm, plus tristimulus CIE XYZ |
| Jednotky | Kalibrované spektrální ozáření, **W/m²/nm** (aplikován balíček tovární kalibrace + profil krytu) |
| Vložená metadata | ID senzoru (klíč pro načtení tovární kalibrace dané jednotky) a platný profil krytu — viz [Profily krytů a kalibrovaný rozsah](caps-and-range.md) |

Formát je identický pro modely DAQ-U, DAQ-M, DAQ-E a DAQ-E-S, takže při následném zpracování nezáleží na tom, který přenosový kanál záznam pořídil. Záznam z DAQ-E-S ukládá vedle každého spektra také údaje o náklonu, bočním naklonění a sklonu jednotky.

**Záznamy přežijí výpadek napájení (1.3.0).**Živý záznam se zapisuje s deníkem a při každém potvrzení dochází k úplné synchronizaci s diskem, takže výpadek napájení nebo ukončení procesu způsobí ztrátu pouze posledních přibližně 3 sekund, nikoli celého souboru. Záznam zastavený uprostřed zápisu může vedle souboru `.daq` zanechat soubor `<name>.daq-journal`:**oba soubory si ponechte a zkopírujte je společně.** Program Chloros při prvním načtení souboru vrátí deník zpět. Zaneprázdněný disk zpožďuje zápis souboru, nikdy však naměřené hodnoty senzoru.

**DAQ-E, který přestane odesílat data, se znovu připojí (verze 1.3.0).** Pokud dojde k výpadku datového toku z DAQ-E (vypnutí a zapnutí napájení přepínače, přerušení spojení), backend jej znovu připojí s původním nastavením a záznam na něm pokračuje ve stejném souboru `.daq` s mezerou označenou v souboru.

Kalibrovaný záznam vyžaduje balíček tovární kalibrace snímače. U zařízení DAQ-U a DAQ-M backend načte balíček z cloudu společnosti MAPIR podle ID snímače (pokud se to nepodaří, záznam je odmítnut); zařízení DAQ-E jsou z tohoto požadavku vyjmuta, protože si kalibraci uchovávají přímo v zařízení.

## Záznam z grafického uživatelského rozhraní

Záznam v grafickém uživatelském rozhraní vyžaduje **otevřený projekt** (v opačném případě jsou tlačítka pro záznam deaktivována):

* **Záznam všeho / Zastavit vše** — v horní části postranního panelu Světelné senzory; spustí nebo zastaví záznam `.daq` na všech připojených senzorech najednou.
* **Záznam / Zastavit záznam** — pro každý senzor zvlášť, v modálním okně nastavení s ikonou ozubeného kola. Během záznamu se v řádcích s aktuálními informacemi o senzoru zobrazuje červený indikátor „REC“.

Soubory se ukládají do formátu `<project>/light_sensor/` a po zastavení záznamu — ať už pomocí tlačítka Zastavit, Zastavit vše nebo odpojením záznamového senzoru — je hotový soubor `.daq` **automaticky přidán do otevřeného projektu**. Objeví se v seznamu souborů projektu bez nutnosti ručního přidávání a je již připraven k zpracování odrazivosti.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->



## Nahrávání z CLI



CLI nahrává prostřednictvím skupiny senzorů backendu (backend musí být spuštěn – tyto příkazy jsou jednoduché HTTP klienty):

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
| `--duration` | `0` — nahrávat až do `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` v souborovém systému **backendu**, nikoli v souborovém systému CLI |

Rozdíl ve výstupním adresáři hraje roli, když CLI směřuje na backend na jiném počítači: soubor se uloží tam, kde běží backend. Pokud je backend na stejném počítači, relativní `-o` (například výše uvedený `./out`) se vyhodnotí vzhledem ke složce, ve které jste příkaz spustili.

## Záznam z Python

u `DAQSensorSession` (vrácený příkazem `chloros_sdk.connect_daq_sensor()`) zpřístupňuje stejný sdružený záznam: `record_start(output_dir=None, device_name=None)` vrací cestu k souboru, `record_stop()` vrací `{path, rows}`. Úplné API relace najdete v [Referenci SDK](../reference/sdk-reference.md). Třídy SDK pro přímý přístup k hardwaru (pouze pro instalace na stolních počítačích) zapisují záznamy ve výchozím nastavení do `~/Documents/DAQ/`; u vydaných verzí je podporovanou cestou výše uvedená sdružená cesta.

## Použití souboru .daq při zpracování

K výpočtu odrazivosti ze snímků potřebuje program Chloros údaje o slunečním záření směřujícím k zemi (downwelling irradiance) odpovídající každé expozici:

* **Uchovejte soubor `.daq` společně se snímky.**Při zpracování systém automaticky určí**sluneční záření směřující dolů s odpovídajícím časovým razítkem** ze zaznamenaného souboru `.daq` (libovolný model DAQ) — nebo z nativního souboru DAQ-M `.csv` — nalezeného vedle snímků. Záznamy z grafického uživatelského rozhraní (GUI) tuto podmínku splňují automaticky, protože jsou do projektu přidány v okamžiku, kdy se zastaví.
* **Kalibrace se načítá na vyžádání.** Pokud balíček tovární kalibrace pro konkrétní kameru nebo DAQ ještě není lokálně uložen v mezipaměti, Chloros jej při prvním použití automaticky načte z cloudu MAPIR (jednou je nutné připojení k internetu; uloží se do mezipaměti pod názvem `~/.chloros/`).
* **Živé záznamy zapisují vlastní doprovodný soubor.** U každého snímku odrazivosti pořízeného v reálném čase se skutečně použitá hodnota z DAQ uloží jako soubor `.daq` do složky `daq/` daného záznamu, takže záznam lze později znovu zpracovat i bez původního záznamu.
* **Záznamy z rekordérů `chloros_scripts` (1.3.0).**Soubor `.daq` vytvořený veřejnými záznamníky [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) je uveden jako model DAQ, který jej vytvořil, a jeho limit lze nastavit v**Nastavení projektu → Světelný senzor DAQ** stejně jako u jakéhokoli jiného záznamu. Tyto záznamníky zapisují surová data z čítače senzoru; Chloros načte tovární kalibraci senzoru přes sériové rozhraní a aplikuje ji.
* **Čí je daná hodnota (1.3.0).**Vedle každého záznamu `.daq` je v položce `.daq.cameras.json` uvedena kamera (kamery), jejíž snímky daná hodnota popisuje, a zpracování ji přiřazuje**pouze k obrázkům z těchto kamer**. Samostatná kamera bez vlastního světelného senzoru si proto již nebere hodnotu z jiné kamery – je třeba k ní přiřadit senzor nebo zaznamenat vlastní. Záznam pořízený samotným DAQ (GUI Record, `daq pool-record`) a jakýkoli soubor `.daq` bez tohoto souboru (včetně starších snímků) stále slouží pro všechny kamery.

## Získání ozáření zpět

Zpracování projektu také exportuje všechny záznamy světelných senzorů, které obsahuje, do složky
složku `Light Sensor/` vedle obrazových produktů. K tomu **nejsou** potřeba žádné snímky:
samostatný záznam ze světelného senzoru je kompletní záznam a složka obsahující pouze soubory `.daq`
je platným vstupem. Projekt, který obsahuje pouze záznamy ze světelných senzorů, je vypisuje a
zpracovává jak z aplikace, tak z CLI, a po spuštění zůstávají v seznamu. Spuštění
uvádí, kolik produktů ze světelných senzorů bylo zapsáno.

| Produkt | Co to je |
| --- | --- |
| `<name>_calibrated.daq` | Archiv, který lze znovu zpracovat, ve stejném schématu jako živý záznam, nyní s deklarací kalibračního balíčku, který jej vytvořil. Jeho opětovný import **ne** provede druhou kalibraci. |
| `<name>_calibrated.csv` | Spektrální ozáření v W/m²/nm na vlastní vlnové mřížce senzoru, jeden řádek na jedno měření, plus fotometrické sloupce: celkový výkon, fotopický a skotopický lux, PPFD s rozdělením na modrou, zelenou a červenou složku a špičková vlnová délka. |

DAQ-U nebo DAQ-M, jehož kalibrační balíček nelze načíst — jste offline nebo
daný senzor nemá v souboru žádnou kalibraci — je **přeskočen s uvedením důvodu**, nikdy není zapsán
jako „kalibrovaný“ soubor obsahující surová měření. Připojte se k internetu a spusťte proces znovu. DAQ-E
má vlastní kalibraci, takže ji potřebuje pouze v případě, že není připojen a
v lokální mezipaměti není nic uloženo.

### DAQ-A: surové počty a proč je to správné řešení

**DAQ-A** vznikl ještě před zavedením systému kalibračních balíčků pro jednotlivé sériové čísla a nemá žádný balíček, který by bylo možné
načíst. Nejedná se o opomenutí: přístroj DAQ-A se kalibruje v terénu pomocí
reflexního terče a kalibrace založená na terči vyžaduje pouze *relativní*
odpověď senzoru — což přesně odpovídá jeho surovým počtům. Společnost Chloros s nimi kalibruje dodnes.

Záznam z DAQ-A se tedy exportuje, ale pod jiným názvem:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, nikoli `_calibrated` — jde o odlišný název souboru, nikoli o příznak uvnitř souboru,
protože tento údaj musí zůstat zachován i při odeslání souboru e-mailem jako pouhý název. Záhlaví `.csv`
uvádí `raw spectral sensor counts (NOT irradiance)` a upozorňuje, že hodnoty jsou
srovnatelné **v rámci** souboru, nikoli mezi jednotlivými senzory. Sloupce, které mají význam
pouze pro skutečné ozáření — celkový výkon, lux, PPFD — jsou ponechány prázdné, místo aby byly
vypočítány z počtů.

Starší záznamy DAQ-A-SD (schéma v1.01 / v1.02) zaznamenávají pouze čas zápisu souboru, nikoli
časové razítko pro každé měření. Chloros nebude snímky s těmito údaji porovnávat — přiřazení snímku k
času zápisu by bylo nesprávné, i když by to na první pohled vypadalo správně — ale export je čte bez problémů a
soubor CSV uvádí, o který časový údaj se jedná.

Kompletní informace o odrazivosti – jeden senzor s kamerou a dva senzory (okolí/objekt) – najdete v [Pracovních postupech pro odrazivost](reflectance.md).
