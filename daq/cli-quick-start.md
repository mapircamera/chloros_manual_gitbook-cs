# Rychlý start CLI (pool-*)

Dodávané ovladače `chloros-cli` řídí senzory DAQ prostřednictvím rodiny příkazů **`daq pool-*`** — jednoduchých HTTP klientů, které ovládají senzory prostřednictvím trvalého fondu senzorů backendu Chloros. Backend řídí přenos, takže grafické rozhraní (GUI), CLI i skripty SDK sdílejí jeden aktivní popisovač, místo aby se přetahovaly o port. Vše, co zákazník potřebuje, je dostupné prostřednictvím `pool-*`: připojení, streamování, záznam kalibrovaných souborů `.daq` a výměna profilů kap.

`pool-*` je podporované rozhraní DAQ ve vydaných verzích. `chloros-cli daq --help` vypisuje podpříkazy `pool-*` a `daq discover` / `daq list` spustí stejné skenování jako `pool-discover`. Pokud není k dispozici podpříkaz pro přímé hardwarové sběr dat (DAQ), program se ukončí s explicitní chybou, která uvádí název chybějícího balíčku a odkazuje zpět na příkaz `pool-*` — nic neselže bez upozornění. (`pip install chloros-sdk` rovněž neposkytuje příkazy pro přímé ovládání hardwaru.)

***

## Předpoklady

* **Backend Chloros musí být spuštěn** — příkazy `pool-*` jsou HTTP klienti, nikoli hardwarové ovladače. Ve Windows spusťte desktopovou aplikaci Chloros (ta spustí backend). Na bezmonitorovém Linuxu/Jetsonu povolte službu jako svůj uživatel: `sudo systemctl enable --now chloros-backend@$USER.service` (viz [Instalace na Linuxu](../linux/linux-installation.md#always-on-ptp-for-headless-hosts)).
* **Přihlášení do Chloros+ (placená úroveň)**: nejprve spusťte `chloros-cli login`. Ověřování probíhá na straně serveru — bez přihlášení příkazy selžou s chybou `401 AUTH_REQUIRED`; v bezplatném tarifu (Iron) selžou s chybou `403 PLAN_UPGRADE_REQUIRED`.
* Příkazy jsou ve výchozím nastavení určeny pro `http://127.0.0.1:5000`; skupina `daq pool-*` respektuje proměnnou prostředí `CHLOROS_BACKEND_URL`, pokud váš backend běží jinde.

***

## Pětiminutová relace

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)
#    (a DAQ-M's BLE MAC is not printed anywhere: find it with `chloros-cli daq pool-discover --only ble`)

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
| `daq pool-connect` | Inteligentní detekce: vyhledat libovolné zařízení DAQ na tomto počítači. |
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
| `--no-stream` | Připojit bez spuštění datového toku (pokračovat později pomocí `pool-stream --start`). |
| `--cap-id CAP` | Profil korekce limitu; výchozí nastavení backendu je `sunshine_cosine`. Viz [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

Připojení zařízení DAQ-E, které je již v poolu pod jinou svou adresou — svým hostitelským jménem `daq-e-<id>.local` nebo svou IP adresou — vrátí existující záznam v poolu namísto druhého otevření jednotky.

{% hint style="warning" %}
**Upozornění týkající se automatického vyhledávání `--eth`.** Na hostiteli s více síťovými rozhraními (více než jedno aktivní síťové rozhraní) může být *první* `pool-connect --eth` po spuštění prázdný, i když je senzor v pořádku — vyhledávání může rozhraní senzoru přehlédnout, dokud není ARP cache naplněna. Pokud příkaz `--eth` nic nenajde, zkuste to znovu, nebo vyhledávání úplně přeskočte pomocí příkazu `--eth-host <ip-or-hostname>`, což je spolehlivý postup na počítačích s více síťovými rozhraními. Název hostitele DAQ-E je `daq-e-<id>.local` (např. `daq-e-def330.local`); funguje i jeho prostá IP adresa.
{% endhint %}

## `pool-discover` — skenování bez připojení

```bash
chloros-cli daq pool-discover                  # USB, BLE and Ethernet
chloros-cli daq pool-discover --only ble       # BLE only: this is how you find a DAQ-M's MAC
chloros-cli daq pool-discover --json           # machine-readable
```

Vyhledává senzory, ke kterým byste se mohli připojit, a zobrazuje adresu pro připojení, aniž by došlo k připojení. `--only` přijímá podskupinu `usb,ble,eth` oddělenou čárkami; `--timeout SEC` nastavuje dobu skenování pro jednotlivé transporty (výchozí hodnota 5 s, omezeno na 1–20 s). Senzory, které jsou již otevřené v poolu, nejsou uvedeny (připojený DAQ-M přestane vysílat signál) — pro tyto použijte `pool-list`. V dodané verzi provádějí stejnou kontrolu příkazy `chloros-cli daq discover` a `daq list`.

## `pool-list` — zobrazení připojených zařízení

Zobrazí všechny senzory v backendovém poolu, včetně `sensor_id`, který potřebují všechny ostatní příkazy:

| Model | `sensor_id` tvar | Příklad |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5 oktetů s pomlčkami | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

Zařízení DAQ-E, u kterého byl vypnut PTP (nastavení je uloženo v jednotce), je v `pool-list` označeno pomocí `!! PTP is switched OFF on this unit (saved in its flash): its timestamps are not PTP-synced.` a příkazem, který jej znovu zapne — viz [`pool-enable-ptp`](#pool-enable-ptp-switch-a-daq-es-ptp-back-on).

## `pool-latest` — čtení rámců spektra

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Vrací nejnovější rámec nebo nejnovější rámce `--recent N`; `--json` generuje strojově čitelný výstup pro skriptování. Rámce představují radiometricky kalibrované spektrální ozáření (W/m²/nm) na mřížce o 135 bodech v rozsahu 340–1010 nm, přičemž je již aplikován profil krytu senzoru. Pro získání kvantitativních hodnot ozáření je třeba provést průměrování snímků po dobu alespoň 15 sekund — jedná se o charakteristiku přístroje, nikoli o vadu.

## `pool-stream` — pozastavení nebo obnovení přenosu dat

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
| `--duration SEC` / `-d SEC` | `0` | Délka záznamu v sekundách; `0` znamená, že se bude provádět, dokud nezadáte `--stop`. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Výstupní adresář. U backendu na tomto počítači se relativní cesta vyhodnocuje vzhledem ke složce, ve které byl příkaz spuštěn; u vzdáleného backendu se cesta vyhodnocuje **na počítači, na kterém běží backend**. |
| `--device-name NAME` | — | Štítek uložený spolu se záznamem. |
| `--stop` | — | Zastaví probíhající záznam. |
| `--confirm-after SEC` | `20` | Po spuštění počkejte maximálně tuto dobu na zápis prvního řádku a v případě, že žádný nedorazí, vydá hlasité varování (`!! WARNING: no rows written after …s. This recording is producing an EMPTY file.`). `0` tuto kontrolu deaktivuje. |
| `--heartbeat SEC` | `30` | S `--duration` se každých SEC sekund vytiskne počet řádků a pokud přestane růst, zobrazí se varování. `0` tuto funkci deaktivuje. |

{% hint style="info" %}
Záznam probíhá na backendu, takže soubor `.daq` se ukládá do souborového systému **backendového stroje** — ve výchozím nastavení v `~/Documents/DAQ Live View/` tam, ne nutně tam, kde jste spustili CLI. Názvy souborů obsahují ID senzoru a časové razítko.
{% endhint %}

## `pool-set-cap` — deklarace nasazeného krytu

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

ID krytky určuje továrně naměřený korekční profil aplikovaný na každé spektrum a **musí odpovídat krytce fyzicky nasazené na senzoru** — ani senzor, ani software nedokážou krytku samy detekovat a výběr je zaznamenán do každého souboru `.daq`. Všude je výchozí hodnotou `sunshine_cosine` (každý DAQ se dodává s nainstalovaným krytem Sunshine s kosinovou korekcí, jehož konstrukce zajišťuje útlum přibližně 12× — nedeklarovaná změna krytu způsobí nesprávnou korekci spekter zhruba o tento faktor).

| `--cap-id` | K dispozici pro |
| --- | --- |
| `sunshine_cosine` (výchozí) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | Pouze DAQ-U |
| `none` | Pouze DAQ-E — viz poznámka |

ID krytu mimo sadu senzorů je při připojení odmítnuto s jasnou chybou. `none` (DAQ-E) znamená, že krytka je fyzicky odstraněna — stále se však uplatňuje tovární geometrický profil pro zapuštěný skleněný difuzor DAQ-E, takže se nejedná o nefunkční stav, a holý DAQ-E představuje laboratorní konfiguraci, nikoli podporovanou konfiguraci pro provoz v terénu. (Holý DAQ-U je skutečně holý a nepotřebuje vůbec žádný korekční profil; DAQ-M se používá se svým krytem Sunshine.)

## `pool-disconnect` — uvolnění senzorů

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

`pool-disconnect` uvolní senzor bez ohledu na to, kdo jej otevřel, takže uvolní i senzor, který v poolu zůstal po havarovaném skriptu (nejprve se dokončí záznam na něm). Senzor, který není v poolu, nemá co uvolnit: příkaz vypíše `<sensor-id>: already released (not in the pool)` a ukončí se `0`. Uvolnění, které backend odmítl nebo nepotvrdil, ukončí se `1`.

## `pool-enable-ptp` — opětovné zapnutí PTP na DAQ-E

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Zařízení DAQ-E uchovává nastavení zapnutí/vypnutí PTP ve své vlastní paměti, takže jednotka, u které byl PTP vypnut, zůstává vypnutá — a její časová razítka nejsou synchronizována s PTP — dokud není znovu zapnuta. `pool-list` takovou jednotku označí a vypíše pro ni tento příkaz. Program Chloros nikdy sám jednotku znovu nezapne, protože vypnutí může být záměrné. Novinka ve verzi 1.3.0.

## Nastavení zařízení DAQ-E

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto
chloros-cli daq pool-set-auto-stream --sensor-id daq-e-def330 --on
chloros-cli daq pool-reboot --sensor-id daq-e-def330
chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

`pool-info` zobrazí firmware senzoru (a zda tato verze Chloros obsahuje novější verzi), jeho adresu a způsob, jakým ji získal, uložené síťové nastavení, pokud se liší od aktuálního, streamování při zapnutí a PTP. `pool-set-network` restartuje senzor, aby se změna uplatnila, a znovu jej připojí s původními nastaveními; `--no-reboot` nastavení pouze uloží. V síti tohoto počítače musí být k dispozici statická adresa, jinak ji Chloros odmítne; `--force` toto omezení obejde, avšak pouze pro počítač, kterému bude přiřazena nová adresa nebo který bude na ni přesměrován. Restarty jsou odmítnuty, pokud senzor právě zaznamenává data. `pool-update-firmware` je ekvivalentem položky **Aktualizovat na \<version\>** </version\>v grafickém uživatelském rozhraní <version\>pro příkazový řádek a slouží k aktualizaci zařízení DAQ-E v systému Linux, který nemá grafické uživatelské rozhraní. Novinka ve verzi 1.3.0.***

## Přehled příkazů

| Příkaz | Účel |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Otevře senzor v backendovém fondu. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Vyhledá senzory bez navázání spojení (najde BLE MAC adresu zařízení DAQ-M). |
| `daq pool-list` | Zobrazí všechny senzory v fondu spolu s jejich `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Posledních N kalibrovaných spektrálních rámců. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Obnovit / pozastavit streamování. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--confirm-after SEC] [--heartbeat SEC] [--stop]` | Spustit / zastavit záznam `.daq` (na straně backendu). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Výměna profilu korekce limitu za běhu. |
| `daq pool-disconnect --sensor-id ID [--all]` | Uvolnění jednoho nebo všech senzorů. |
| `daq pool-enable-ptp --sensor-id ID` | Znovu zapnout PTP pro zařízení DAQ-E, u kterého je PTP vypnuté. |
| `daq pool-info --sensor-id ID [--json]` | Firmware zařízení DAQ-E, adresa, uložené síťové nastavení, streamování při zapnutí a PTP. |
| `daq pool-set-network --sensor-id ID (--auto \| --static IP [--mask M] [--gateway G]) [--no-reboot] [--force]` | DHCP nebo statická adresa; restartuje a znovu připojí senzor. |
| `daq pool-set-auto-stream --sensor-id ID (--on \| --off)` | Streamování při zapnutí. |
| `daq pool-reboot --sensor-id ID` | Restartuje zařízení DAQ-E a znovu jej připojí. |
| `daq pool-update-firmware --sensor-id ID` | Nahraje firmware pro zařízení DAQ-E z této verze do staršího senzoru. |

***

## Řešení problémů při prvním připojení DAQ-E

1. DAQ-E nemá stavovou LED diodu — ověřte napájení pomocí indikátoru PoE/link na přepínači nebo portu injektoru a po zapnutí počkejte několik sekund, než se zařízení spustí a připojí k síti.
2. Backendový počítač musí být ve **stejné vysílací doméně** jako senzor — mDNS neprochází přes směrovače.
3. Firewall musí povolit průchod mDNS (UDP 5353), datům DAQ-E (UDP 5002) a PTP (UDP 319/320). Instalační program pro Windows přidá pro tento účel pravidlo a v Linuxu `.deb` je otevře v `ufw` / `firewalld`, který byl aktivní v době instalace; firewall třetí strany nebo firewall aktivovaný později je musí povolit.
4. Stále nic z `--eth`? Použijte `--eth-host` s názvem hostitele zařízení (`daq-e-<id>.local`) nebo IP adresou, čímž se přeskočí vyhledávání.

***
{% hint style="info" %}
**Tip pro AI asistenty.** Každá stránka tohoto manuálu je poskytována ve formátu Markdown – připojte `.md` k malým písmenům URLu stránky (tato stránka: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); strojově čitelný index je `https://mapir.gitbook.io/chloros/llms.txt`. Kompletní dokumentaci na úrovni příznaků pro `chloros-cli daq` a všechny ostatní rodiny příkazů najdete v [referenční příručce k CLI](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); Pythonová cesta je `chloros_sdk.connect_daq_sensor()` v [SDK Reference](../reference/sdk-reference.md).
{% endhint %}
</version\>
