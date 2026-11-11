# Síťové připojení a synchronizace času u DAQ-E

> Fyzické nastavení sítě pro senzor – kabeláž, PoE, přiřazení IP adresy a vlastní síťová nastavení zařízení – najdete v **[uživatelské příručce k DAQ](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Tato stránka se zabývá stránkou Chloros

: připojením, synchronizací času a postupem v případě, že vyhledávání nepřinese žádné výsledky.

DAQ-E je ethernetovým členem rodiny DAQ: napájen je přes PoE, vyhledává se pomocí mDNS (služba `_daq-e._tcp`) a lze jej adresovat pomocí hostitelského jména odvozeného od jeho ID senzoru — `daq-e-<6 hex>.local`, např. `daq-e-def330.local`. Tato stránka popisuje, jak zařízení přenáší data v síti a jak se podílí na časové synchronizaci PTP.

## Režimy přenosu

| Režim | Koncový bod | Příjemci | Poznámky |
| --- | --- | --- | --- |
| **Multicast** (výchozí) | UDP `239.10.10.10:5002` | Libovolný počet zařízení ve stejné síti LAN přijímá stejný datový proud | Každý datagram je ověřen pomocí CRC-16/CCITT |
| **Raw** | TCP port `5000` | Přesně jeden klient (exkluzivně) | Bezproblémově kompatibilní na úrovni bajtů s DAQ-U |

Chloros

ve výchozím nastavení používá multicast, což umožňuje, aby GUI, CLI

a SDK

mohly všechny současně sledovat jeden senzor.

## Požadavky na síť

* **Stejná vysílací doména.** Počítač, na kterém běží Chloros

, musí být ve stejném segmentu sítě L2 jako senzor — vyhledávání mDNS neprochází přes směrovače.
* **Firewall.** Ve Windows

u instalační program přidá do každého síťového profilu pravidlo firewallu **Chloros

Backend (UDP-In)**pro backend, které pokrývá data DAQ-E (UDP 5002), mDNS (UDP 5353) a PTP (UDP 319/320), a pravidlo**„Chloros

Backend (TCP-In)“**, omezené na lokální podsíť, pro připojení, které zařízení DAQ-E navazuje zpět k počítači během aktualizace firmwaru. V systému Linux

u otevře `.deb` porty UDP v `ufw` nebo `firewalld`, pokud je tento firewall v době instalace aktivní. U firewallu jiného výrobce nebo u firewallu aktivovaného později je třeba povolit stejné porty.
* **Napájení PoE, bez stavové LED.** Zařízení DAQ-E nemá vlastní LED diodu — napájení ověřte pomocí indikátoru link/PoE na přepínači nebo portu injektoru a po zapnutí počkejte několik sekund, než se zařízení spustí a připojí k síti.

## Připojení

**GUI:** Záložka Světelné senzory → Připojit senzor → Typ zařízení „DAQ-E (Ethernet)“. Vyhledávání probíhá pouze dokud je na obrazovce dialogové okno pro připojení (procházení mDNS a ARP sweep v systému Windows

), opakuje se každých 15 sekund; tlačítko Obnovit spustí okamžité opakované vyhledávání. Objevené senzory se zobrazí v rozevíracím seznamu; automaticky se vybere první detekovaný senzor.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (běží backend):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

DAQ-E, který již je v seznamu, je rozpoznán jak podle názvu hostitele, tak podle IP adresy: při opětovném připojení pod jinou adresou se vrátí stávající záznam v seznamu, místo aby se jednotka otevřela podruhé.

### Hostitelé s více síťovými kartami a první připojení po spuštění

Na hostitelích s více než jedním aktivním síťovým rozhraním může být **první** připojení `pool-connect --eth` po spuštění prázdné, i když je senzor v pořádku — vyhledávání může přehlédnout rozhraní, na kterém senzor běží, dokud není ARP cache naplněna. Spolehlivým řešením je přeskočit vyhledávání a předat adresu explicitně:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` akceptuje název hostitele mDNS nebo IP adresu, vždy se zaměří na správný senzor a je doporučeným formátem pro skripty a instalace bez grafického rozhraní. V grafickém uživatelském rozhraní použijte tlačítko Obnovit v dialogovém okně připojení a počkejte na dokončení cyklu opětovného vyhledávání.

## Nastavení zařízení a firmware

Samotný senzor uchovává svá síťová nastavení: DHCP (výchozí nastavení, při neúspěšném spojení s DHCP serverem se přepne na lokální adresu 169.254.x.x) nebo statickou adresu. Uchovává také informaci, zda po zapnutí začne streamovat. Tyto nastavení můžete změnit v grafickém rozhraní (v nastavení senzoru, ikona ozubeného kola, **Síť**:**Změnit…**,**Přenos při zapnutí**,**Restartovat senzor**; viz [Záložka DAQ](gui.md)), nebo z CLI

a SDK

pomocí `pool-set-network`, `pool-set-auto-stream` a `pool-reboot` (`DAQSensorSession.set_network()`, `set_auto_stream()`, `reboot()`; viz [Referenční příručka CLI

](../reference/cli-reference.md#daq-e-device-settings)). Senzor se restartuje, aby se uplatnila změna v síti, a Chloros

jej znovu připojí. V síti počítače, na kterém běží Chloros

, musí být statická adresa: Chloros

odmítne takovou, ke které se po restartu nedokáže připojit. ID senzoru a jeho hostname `daq-e-<id>.local` se nikdy nemění. Novinka ve verzi 1.3.0.

**Aktualizace firmwaru jsou integrovány do grafického uživatelského rozhraní (GUI).** Pokud připojené zařízení DAQ-E používá starší verzi firmwaru, než je obraz dodaný s vaší verzí Chloros

, zobrazí se v řádku tohoto senzoru oranžová ikona **K dispozici aktualizace** a v modálním okně nastavení s ozubeným kolečkem se objeví <version>tlačítko</version>

„Aktualizovat na <version>“. Aktualizace se přes síť nainstaluje za přibližně 30 sekund; senzor se automaticky restartuje a znovu připojí, přičemž v případě přerušení přenosu zůstane aktuální firmware neporušený. Z příkazového řádku (CLI) nebo sady SDK a v systému Linux, který nemá grafické uživatelské rozhraní, spusťte příkaz `chloros-cli daq pool-update-firmware --sensor-id <id>` (`DAQSensorSession.update_firmware()`).

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## Synchronizace času PTP

Firmware DAQ-E v1.2.0+ se účastní standardu IEEE 1588 PTPv2 jako běžný (pouze podřízený) časovač. **Backend hostitele Chloros je PTP grandmaster** — každý DAQ-E a každá kamera LATTICE v síti LAN se k němu připojují jako slave v doméně 0, čímž udržují časová razítka všech zařízení v toleranci ~1 ms. Právě díky tomuto sdílenému hodinovému signálu lze časová razítka měření z DAQ porovnat s expozicemi kamer (viz [Záznam a formát .daq](recording.md)).

Zkontrolujte synchronizaci z CLI:

| Příkaz | Zobrazí |
| --- | --- |
| `chloros-cli time-sync status` | Stav hostitelského grandmastera, priority BMCA, identitu hodin a `health`: `OK` nebo `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` s `health_detail` uvádějícím důvod, proč jeho pakety nedosahují sítě |
| `chloros-cli time-sync peers` | Zobrazeny všechny podřízené jednotky (senzory DAQ-E + kamery LATTICE) |
| `chloros-cli time-sync cameras` | Stav PTP pro jednotlivé kamery (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Restartovat proces grandmasteru |

V grafickém uživatelském rozhraní (GUI) zobrazuje modální okno nastavení DAQ-E živý řádek **PTP Sync** s aktuálním stavem PTP daného senzoru.**Zařízení DAQ-E s vypnutým PTP zůstává vypnuté.** Jednotka uchovává nastavení zapnutí/vypnutí PTP ve své vlastní paměti, takže jednotka, u které byl PTP vypnut, zůstává vypnutá i po opětovném připojení nebo vypnutí a zapnutí napájení a její časová razítka nejsou synchronizována pomocí PTP. `chloros-cli daq pool-list` takovou jednotku označí (`!! PTP is switched OFF on this unit …`) a vypíše příkaz, který to napraví:

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Chloros nikdy sám jednotku znovu nezapne, protože vypnutý stav může být záměrný.

Podrobnosti pro uživatele vyžadující přísné synchronizace:

* Každý streamovaný datagram obsahuje pole příznaků; **bit 2 je nastaven u rámců, jejichž časové razítko je synchronizováno s PTP**. Potrubí, která vyžadují přísnou synchronizaci kamery/DAQ, by měla být řízena tímto bitem.
* Před synchronizovaným snímáním ověřte, zda se senzor objevuje v `chloros-cli time-sync peers` a zda `chloros-cli daq pool-list` neoznačuje jeho PTP jako vypnuté. (Interní nástroje MAPIR pro přímé ovládání hardwaru mohou také řídit záznam na základě zámku PTP pomocí příznaku `--wait-ptp`, který čeká až 15 s, než senzor dosáhne stavu SLAVE; tyto nástroje nejsou součástí dodávaného CLI.)
* Zatímco je PTP aktivně v režimu slave, senzor odmítá ruční zasílání hodin („PTP poskytuje hodiny“). To je záměrné — důvěřujte PTP.

## Poznámky k Linuxu

* **PTP porty bez oprávnění root.** PTP využívá porty UDP 319/320. `.deb` zapisuje `/etc/sysctl.d/60-chloros-ptp.conf` (`net.ipv4.ip_unprivileged_port_start = 319`), takže backend je váže a běží pod vaším uživatelským účtem. Od verze 1.3.0 balíček nenastavuje na backendu žádnou oprávnění k souborům (`setcap`) a nepotřebuje `libcap2-bin`; nepřidávejte oprávnění sami, protože rozbalený backend by se pak nespustil. Pokud PTP nemůže navázat spojení na portu 319, viz [Instalace na Linuxu](../linux/linux-installation.md#ptp-will-not-start--cannot-bind-port-319).

* **Jetson bez grafického rozhraní / hostitelský počítač s Linuxem:** balíček nainstaluje služby backendu, ale žádnou z nich neaktivuje. Chcete-li zajistit nepřetržitý provoz PTP (a dostupnost DAQ) bez nutnosti spouštět příkaz, aktivujte službu pod svým uživatelským účtem:

  ```bash
  sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
  ```

  Bez toho bude PTP běžet pouze tehdy, když něco spustilo backend — příkaz `chloros-cli` nebo skript SDK. Viz [Instalace na Linuxu](../linux/linux-installation.md#always-on-ptp-for-headless-hosts) pro informace o uživateli root `chloros-backend.service` a o tom, jak z něj přepnout.

## Řešení problémů: „Nebyly nalezena žádná zařízení DAQ-E“

| Kontrola | Podrobnosti |
| --- | --- |
| Napájení | Na senzoru nesvítí žádná LED — zkontrolujte indikátory PoE a připojení na přepínači/portu injektoru; po zapnutí počkejte několik sekund |
| Vysílací doména | Hostitel a senzor jsou ve stejném segmentu L2; mDNS nesměruje |
| Firewall | Windows: pravidlo **Chloros Backend (UDP-In)** v instalačním programu povoluje vstup datového proudu DAQ-E (UDP 5002), mDNS (5353) a PTP (319/320); u firewallu jiného výrobce je třeba nastavit totéž. Linux: `.deb` otevírá tyto porty v `ufw` / `firewalld` pouze v případě, že byl firewall aktivní v době instalace — v případě pozdějšího zapnutí je musíte povolit ručně |
| Hostitel s více síťovými kartami | Vyhledávání prochází mDNS s náhradním řešením ARP; `--eth-host <ip-or-hostname>` vynechá vyhledávání a připojí se přímo |
| Opětovné prohledání v grafickém rozhraní | Vyhledávání běží pouze při otevřeném dialogovém okně pro připojení; použijte tlačítko Obnovit |</version>
