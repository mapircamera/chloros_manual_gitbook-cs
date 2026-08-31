# Síťové připojení a synchronizace času u DAQ-E

> Informace o fyzickém nastavení sítě pro senzor – kabeláž, PoE, přiřazení IP adresy a vlastní síťová nastavení zařízení – najdete v **[uživatelské příručce k DAQ](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Tato stránka se zabývá stránkou Chloros: připojením, synchronizací času a postupem v případě, že vyhledávání nepřinese žádné výsledky.

DAQ-E je ethernetovým členem rodiny DAQ: napájený přes PoE, detekovaný pomocí mDNS (služba `_daq-e._tcp`) a adresovatelný pomocí hostitelského jména odvozeného od ID senzoru — `daq-e-<6 hex>.local`, např. `daq-e-def330.local`. Tato stránka popisuje, jak zařízení přenáší data v síti a jak se podílí na časové synchronizaci PTP.

## Přenosové režimy

| Režim | Koncový bod | Příjemci | Poznámky |
| --- | --- | --- | --- |
| **Multicast** (výchozí) | UDP `239.10.10.10:5002` | Libovolný počet zařízení ve stejné síti LAN přijímá stejný datový proud | Každý datagram je ověřen pomocí CRC-16/CCITT |
| **Raw** | TCP port `5000` | Přesně jeden klient (exkluzivní) | Bezproblémově kompatibilní na úrovni bajtů s DAQ-U |

Chloros ve výchozím nastavení používá multicast, což umožňuje grafickému rozhraní, CLI a SDK sledovat jeden senzor současně.

## Požadavky na síť

* **Stejná vysílací doména.** Počítač, na kterém běží Chloros, musí být ve stejném segmentu sítě L2 jako senzor – vyhledávání mDNS neprochází přes směrovače.
* **Výzva brány firewall pro Windows: přijměte ji.** Při prvním navázání multicastových soketů programem Chloros se program Windows Defender zeptá jednou. Povolení zahrnuje data DAQ-E (UDP 5002), mDNS (UDP 5353) a PTP (UDP 319/320). U Linux se toto neozve.
* **Napájení PoE, žádná stavová LED.** DAQ-E nemá vlastní LED diodu – stav napájení ověřte pomocí indikátoru link/PoE na přepínači nebo portu injektoru a po zapnutí počkejte několik sekund, než se zařízení spustí a připojí k síti.

## Připojení

**GUI:** Záložka Světelné senzory → Připojit senzor → Typ zařízení „DAQ-E (Ethernet)“. Vyhledávání probíhá pouze po dobu, kdy je na obrazovce zobrazeno dialogové okno pro připojení (prohledávání mDNS a ARP na adrese Windows), a to s opakováním každých 15 sekund; tlačítko Obnovit spustí okamžité opětovné prohledávání. Nalezené senzory se zobrazí v rozevíracím seznamu; automaticky je vybrán první detekovaný senzor.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (běží backend):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

### Hostitelé s více síťovými kartami a první připojení po spuštění systému

Na hostitelích s více než jedním aktivním síťovým rozhraním může být **první** `pool-connect --eth` po spuštění prázdné, i když je senzor v pořádku — vyhledávání může vynechat rozhraní, na kterém senzor běží, dokud není ARP cache naplněna. Spolehlivým řešením je přeskočit vyhledávání a předat adresu explicitně:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` akceptuje název hostitele mDNS nebo IP adresu, vždy cílí na správný senzor a je doporučeným formátem pro skripty a instalace bez grafického rozhraní. V grafickém uživatelském rozhraní použijte tlačítko „Refresh“ (Obnovit) v dialogovém okně pro připojení a nechte proběhnout nový cyklus vyhledávání.

## Nastavení zařízení a firmware

Samotný senzor uchovává síťová nastavení — statická IP adresa vs. DHCP + adresování link-local, název zařízení, automatické spouštění streamu při startu systému, heslo pro OTA aktualizace. Tato nastavení na straně zařízení nejsou v dodaném CLI dostupná jako příkazy; spravují se prostřednictvím grafického rozhraní Chloros, kde jsou zobrazena, nebo s podporou MAPIR.

**Aktualizace firmwaru jsou integrovány do grafického uživatelského rozhraní.**Pokud připojený DAQ-E používá starší verzi firmwaru než obraz dodaný s vaší verzí Chloros, v řádku senzoru se zobrazí oranžová ikona**K dispozici je aktualizace** a v modálním okně nastavení s ozubeným kolečkem se objeví<version>

tlačítko</version> „Aktualizovat na<version>

“. Aktualizace se přes síť nainstaluje za přibližně 30 sekund; senzor se automaticky restartuje a znovu připojí, přičemž v případě přerušení přenosu zůstane aktuální firmware neporušený.

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## Synchronizace času PTP

Firmware DAQ-E v1.2.0+ se účastní standardu IEEE 1588 PTPv2 jako běžný (pouze podřízený) časovač. **Backend hostitelského zařízení Chloros je PTP grandmaster** — každý DAQ-E a každá kamera LATTICE v síti LAN se k němu připojují jako slave v doméně 0, čímž se udržují časová razítka všech zařízení s tolerancí ~1 ms. Právě díky tomuto sdílenému hodinovému signálu lze časová razítka měření DAQ porovnávat s expozicemi kamer (viz [Záznam a formát .daq](recording.md)).

Zkontrolujte synchronizaci z CLI:

| Příkaz | Zobrazuje |
| --- | --- |
| `chloros-cli time-sync status` | Stav hostitelského grandmastera, priority BMCA, identita hodin |
| `chloros-cli time-sync peers` | Všechny detekované podřízené jednotky (senzory DAQ-E + kamery LATTICE) |
| `chloros-cli time-sync cameras` | Stav PTP pro jednotlivé kamery (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Restart procesu grandmaster |

V grafickém uživatelském rozhraní (GUI) zobrazuje modální okno nastavení DAQ-E živý řádek **PTP Sync** s aktuálním stavem PTP daného senzoru.

Podrobnosti pro uživatele vyžadující přesné synchronizování:

* Každý streamovaný datagram obsahuje pole příznaků; **bit 2 je nastaven u rámců, jejichž časové razítko je synchronizováno s PTP**. Potrubí, která vyžadují přesné synchronizování kamery a DAQ, by měla být řízena tímto bitem.
* Před synchronizovaným snímáním ověřte, zda se snímač objevuje v `chloros-cli time-sync peers`. (Interní nástroj pro přímé hardwarové ovládání v MAPIR může také řídit nahrávání na základě zámku PTP pomocí příznaku `--wait-ptp`, který čeká až 15 s, než senzor dosáhne stavu SLAVE; tato funkce není součástí dodávaného CLI.)
* Zatímco je PTP aktivně ve stavu slave, senzor odmítá ruční zasílání hodinového signálu („PTP poskytuje hodinový signál“). To je záměrné — důvěřujte PTP.

## Poznámky k Linux

* **PTP vyžaduje `libcap2-bin` při instalaci.** Skript `.deb` po instalaci udělí oprávnění skriptu `cap_net_bind_service=+ep` na skriptu `/usr/lib/chloros/chloros-backend`, aby mohl bez oprávnění root navázat PTP porty 319/320. Pokud `libcap2-bin` chybí, tento krok se přeskočí a PTP se nespustí. Řešení:

  ```bash
  sudo apt install libcap2-bin
  sudo apt reinstall chloros
  ```

* **Jetson / Raspberry Pi bez grafického rozhraní:** při první instalaci se vygeneruje jednotka systemd `chloros-backend.service`, ale není povolena. Pro trvale spuštěný PTP (a dostupnost DAQ) bez grafického rozhraní:

  ```bash
  sudo systemctl enable --now chloros-backend.service
  ```

  Bez ní běží PTP pouze po dobu, kdy je otevřeno grafické rozhraní Chloros.

## Řešení potíží: „Nebyly nalezena žádná zařízení DAQ-E“

| Kontrola | Podrobnosti |
| --- | --- |
| Napájení | Žádná LED na senzoru — zkontrolujte indikátory PoE a připojení na přepínači/portu injektoru; po zapnutí počkejte několik sekund |
| Vysílací doména | Hostitel a senzor jsou ve stejném segmentu L2; mDNS nesměruje |
| Firewall Windows | Při prvním spuštění přijměte výzvu Defenderu (UDP 5002, 5353, 319/320) |
| Hostitel s více síťovými kartami | Při prvním vyhledávání po spuštění může dojít k vynechání senzoru — připojte se pomocí `--eth-host <ip-or-hostname>` |
| Opakované vyhledávání v grafickém rozhraní | Vyhledávání probíhá pouze při otevřeném dialogovém okně pro připojení; použijte tlačítko Obnovit |</version>
