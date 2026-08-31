# CLI Rychlý start (pool-*)

Dodávané řadiče `chloros-cli` ovládají senzory DAQ prostřednictvím rodiny příkazů **`daq pool-*`** — tenké klienty HTTP, které ovládají senzor prostřednictvím trvalého fondu senzorů backendu Chloros. Backend řídí přenos, takže grafické uživatelské rozhraní (GUI), skripty CLI a SDK sdílejí jeden aktivní popisovač, místo aby se přetahovaly o port. Vše, co zákazník potřebuje, je dostupné prostřednictvím `pool-*`: připojení, streamování, záznam kalibrovaných souborů `.daq` a výměna profilů kondenzátorů.

`pool-*` je také **jediným** rozhraním DAQ ve vydaných verzích. `chloros-cli daq --help` vypisuje podpříkazy `pool-*`, a volání podpříkazu pro přímé hardwarové DAQ ve vydané verzi skončí explicitní chybou, která uvede název chybějícího balíčku a odkáže vás zpět na `pool-*` — nic neselže bez upozornění. (Příkazy pro přímý přístup k hardwaru lze spustit pouze ze zdrojového kódu MAPIR; ani `pip install chloros-sdk` je neposkytuje.)

***

## Předpoklady

* **Musí být spuštěn backend Chloros** — příkazy `pool-*` jsou klienti HTTP, nikoli hardwarové ovladače. Na Windows spusťte desktopovou aplikaci Chloros (ta spustí backend). Na bezdisplejovém zařízení Linux/Jetson povolte službu: `sudo systemctl enable --now chloros-backend.service`.
* **Přihlášení k Chloros+ (placená úroveň)**: nejprve spusťte `chloros-cli login`. Ověřování probíhá na straně serveru — bez přihlášení se příkazy u `401 AUTH_REQUIRED` nezpracují; v bezplatném tarifu (Iron) se nezpracují u `403 PLAN_UPGRADE_REQUIRED`.
* Příkazy jsou ve výchozím nastavení určeny pro `http://127.0.0.1:5000`; řada `daq pool-*` respektuje proměnnou prostředí `CHLOROS_BACKEND_URL`, pokud váš backend běží jinde.

***

## Pětiminutová relace

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)

# 2. List the pool — this shows the sensor_id used by every command below
chloros-cli daq pool-list

# 3. Read the most recent calibrated spectrum frame (add --json for scripting)
chloros-cli daq pool-latest --sensor-id daq-e-def330 --json

# 4. Record a calibrated .daq file for 60 seconds
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 60 \
  --device-name "field-A"

# 5. Release the sensor when done
chloros-cli daq pool-disconnect --sensor-id daq-e-def330
```

***

## `pool-connect` — otevření senzoru v poolu

| Varianta | Význam |
| --- | --- |
| `daq pool-connect` | Inteligentní detekce: vyhledat libovolný DAQ na tomto počítači. |
| `daq pool-connect --port PORT` | DAQ-U na konkrétním sériovém portu (např. `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | DAQ-M přes BLE, automatické vyhledávání MAC adresy. |
| `daq pool-connect --mac MAC` | DAQ-M na známé adrese BLE MAC (implikuje `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E se známým názvem hostitele nebo IP adresou — **spolehlivá cesta**. |
| `daq pool-connect --eth` | DAQ-E s automatickým vyhledáváním (mDNS, s náhradním řešením ARP). Viz upozornění níže. |

Nastavovací příznaky, všechny volitelné:

| Příznak | Význam |
| --- | --- |
| `--integration-time MS` / `-t MS` | Ručně nastavená doba integrace v milisekundách. |
| `--frame-avg N` / `-f N` | Průměrný počet snímků na jedno vyhodnocené spektrum. |
| `--no-ae` | Vypnout automatickou expozici (AE je ve výchozím nastavení zapnuto). |
| `--no-stream` | Připojit bez spuštění streamu (pokračovat později pomocí `pool-stream --start`). |
| `--cap-id CAP` | Profil korekce limitu; výchozí nastavení backendu je `sunshine_cosine`. Viz [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

{% hint style="warning" %}
**Upozornění týkající se automatického zjišťování `--eth`.** Na hostiteli s více síťovými rozhraními (více než jedno aktivní síťové rozhraní) může být *první* volání `pool-connect --eth` po spuštění systému prázdné, i když je senzor v pořádku — vyhledávání může rozhraní senzoru přehlédnout, dokud není ARP cache naplněna. Pokud příkaz `--eth` nic nenajde, zkuste to znovu, nebo vyhledávání úplně přeskočte pomocí příkazu `--eth-host <ip-or-hostname>`, což je spolehlivý postup na počítačích s více síťovými rozhraními. Název hostitele DAQ-E je `daq-e-<id>.local` (např. `daq-e-def330.local`); funguje i jeho prostá IP adresa.
{% endhint %}

## `pool-list` — zobrazení připojených zařízení

Zobrazí všechny senzory v backendovém poolu, včetně `sensor_id`, které potřebují všechny ostatní příkazy:

| Model | Formát `sensor_id` | Příklad |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5 oktetů s pomlčkami | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

## `pool-latest` — čtení spektrálních rámců

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Vrací nejnovější rámec nebo nejnovější rámce `--recent N`; `--json` generuje strojově čitelný výstup pro skriptování. Snímky představují radiometricky kalibrované spektrální ozáření (W/m²/nm) na mřížce o 135 bodech v rozsahu 340–1010 nm, přičemž je již aplikován profil krytu senzoru. Pro získání kvantitativních hodnot ozáření je třeba provést průměrování snímků po dobu alespoň 15 sekund — jedná se o charakteristiku přístroje, nikoli o vadu.

## `pool-stream` — pozastavení nebo obnovení streamování

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` — záznam souboru `.daq`

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| Příznak | Výchozí | Význam |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | Délka záznamu v sekundách; `0` znamená, že se program bude spouštět, dokud nezadáte příkaz `--stop`. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Výstupní adresář, určený **na stroji, na kterém běží backend**. |
| `--device-name NAME` | — | Štítek uložený spolu se záznamem. |
| `--stop` | — | Zastaví probíhající záznam. |

{% hint style="info" %}
Nahrávání probíhá v backendu, takže soubor `.daq` se ukládá do souborového systému **počítače s backendem** — ve výchozím nastavení do adresáře `~/Documents/DAQ Live View/`, ne nutně tam, kde jste spustili program CLI. Názvy souborů obsahují ID senzoru a časové razítko.
{% endhint %}

## `pool-set-cap` — deklarace nasazeného krytu

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

ID krytky určuje továrně naměřený korekční profil aplikovaný na každé spektrum a **musí odpovídat krytce fyzicky nasazené na snímači** — ani snímač, ani software nedokážou krytku samy detekovat a tento výběr je zaznamenán do každého souboru `.daq`. Všude je výchozí hodnotou `sunshine_cosine` (každý DAQ se dodává s nainstalovaným krytem Sunshine s kosinovou korekcí, jehož konstrukce zajišťuje útlum přibližně 12× — nedeklarovaná změna krytu způsobí nesprávnou korekci spekter zhruba o tento faktor).

| `--cap-id` | K dispozici pro |
| --- | --- |
| `sunshine_cosine` (výchozí) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | Pouze DAQ-U |
| `none` | Pouze DAQ-E — viz poznámka |

ID krytu, které nespadá do sady senzoru, je při připojení odmítnuto s jasnou chybou. `none` (DAQ-E) znamená, že krytka je fyzicky odstraněna — stále se však uplatňuje tovární geometrický profil pro zapuštěný skleněný difuzor DAQ-E, takže se nejedná o nefunkční stav, a holý DAQ-E představuje laboratorní konfiguraci, nikoli podporovanou konfiguraci pro provoz v terénu. (Holý snímač DAQ-U je skutečně holý a nevyžaduje vůbec žádný korekční profil; snímač DAQ-M se používá se svým krytem Sunshine.)

## `pool-disconnect` — uvolnění snímačů

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

***

## Přehled příkazů

| Příkaz | Účel |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Otevřít senzor v backendovém fondu. |
| `daq pool-list` | Zobrazit všechny senzory v fondu spolu s jejich `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Posledních N kalibrovaných spektrálních snímků. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Obnovit / pozastavit streamování. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--stop]` | Spustit / zastavit záznam `.daq` (na straně backendu). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Výměna profilu korekce šumové špičky za běhu. |
| `daq pool-disconnect --sensor-id ID [--all]` | Uvolnění jednoho nebo všech senzorů. |

***

## Řešení problémů při prvním připojení DAQ-E

1. DAQ-E nemá stavovou LED diodu — ověřte napájení pomocí indikátoru PoE/link na přepínači nebo portu injektoru a po zapnutí počkejte několik sekund, než se zařízení spustí a připojí k síti.
2. Backendový počítač musí být ve **stejné vysílací doméně** jako senzor — mDNS neprochází přes směrovače.
3. Na Windows při prvním spuštění potvrďte výzvu brány firewall Defender (mDNS UDP 5353, data DAQ-E UDP 5002, PTP UDP 319/320).
4. Stále nic od `--eth`? Použijte `--eth-host` s názvem hostitele zařízení (`daq-e-<id>.local`) nebo jeho IP adresou – to je spolehlivější způsob, zejména u hostitelů s více připojeními.

***{% hint style="info" %}**Tip pro AI asistenty.** Každá stránka tohoto manuálu je poskytována ve formátu Markdown – k malému slugu stránky (např. URL) přidejte `.md` (tato stránka: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); strojově čitelný index je `https://mapir.gitbook.io/chloros/llms.txt`. Kompletní dokumentaci na úrovni příznaků pro `chloros-cli daq` a všechny ostatní rodiny příkazů najdete v [CLI Referenci](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); cesta k Python je `chloros_sdk.connect_daq_sensor()` v [Referenci SDK](../reference/sdk-reference.md).
{% endhint %}
