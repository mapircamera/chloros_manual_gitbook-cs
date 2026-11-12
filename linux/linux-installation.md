# Instalace na Linuxu

Chloros je pro Linux distribuován jako balíčky `.deb`, které instalují CLI, backendový server a Python SDK (pro systém `python3`, plus wheel odpovídající verzi pro virtualenvs).

Názvy balíčků obsahují označení verze a architektury: `chloros_1.3.0_amd64.deb` pro x86_64, a `chloros_1.3.0_arm64_jp6.deb` pro arm64 (NVIDIA Jetson, JetPack 6). V níže uvedených příkazech nahraďte název souboru tím, který jste si skutečně stáhli.

{% hint style="info" %}
**Novinka ve verzi 1.3.0: programy se dodávají v rozbaleném stavu.** Backend a CLI se instalují jako složky, `/usr/lib/chloros/backend/` a `/usr/lib/chloros/cli/`, namísto jednotlivých samorozbalovacích souborů. Už se při každém spuštění nerozbalují do dočasné složky, takže se spouštějí mnohem rychleji (na x86_64 trvalo rozbalení backendu při každém spuštění dříve asi 100 sekund) a již nepotřebují několik GB volného místa v adresáři `/tmp`. Verze pro amd64 `.deb` má rovněž přibližně poloviční velikost oproti dřívějšku.
{% endhint %}

***

## Linux, amd64 (x86_64)

### Systémové požadavky

| Požadavek | Minimální | Doporučené |
| --- | --- | --- |
| **Distribuce** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Procesor** | x86_64 (Intel/AMD) | Intel Core i7 nebo lepší |
| **Paměť (RAM)** | 8 GB | 16 GB nebo více |
| **Grafická karta** | Žádná (zpracování na CPU) | Grafická karta NVIDIA s 4 GB+ VRAM (12 GB+ odemkne `GPU_PARALLEL`, 7 GB+ udržuje funkci Texture Aware mimo cestu pro jeden obrázek) |
| **Úložiště** | 2 GB volného místa | SSD s 10 GB+ volného místa |
| **Python** | Python 3.7+ (pro SDK) | Python 3.10+ |

> **Ubuntu 20.04 a Debian 11 nejsou podporovány.** Seznam závislostí `.deb` je
> odvozen od toho, na co se backend Chloros skutečně odkazuje, a to zahrnuje
> `libc6` novější než glibc 2.31, kterou dodávají distribuce Focal a Bullseye, takže `apt` instalaci
> rovnou odmítne, místo aby ji nechal později selhat za běhu.

### Instalace

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**Použijte `apt install ./…`, nikoli `dpkg -i`.** `apt` vyřeší závislosti balíčku a nainstaluje také to, co doporučuje: požadavky Python SDK, `python3-numpy` a `python3-requests`. `dpkg -i` neinstaluje ani jedno z nich. CLI a backend jsou samostatné a fungují i bez nich, ale `import chloros_sdk` je potřebuje; výstup instalace uvádí všechny chybějící soubory. Zachovejte `./`, aby `apt` nainstaloval tento soubor, místo aby hledal balíček s tímto názvem. Balíček také deklaruje `libgl1`, který načítá přibalené OpenCV a v minimálním obrazu serveru může chybět.
{% endhint %}

Ověřte instalaci:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### Systémové požadavky

| Požadavek | Minimální | Doporučené |
| --- | --- | --- |
| **Platforma** | NVIDIA Jetson s JetPack 6 | Jetson Orin NX 16 GB nebo AGX Orin |
| **JetPack** | JetPack 6.x | Nejnovější JetPack 6 |
| **Paměť (RAM)** | 8 GB (sdílená mezi GPU a CPU) | 16 GB+ sdílené (12 GB+ je prahová hodnota pro paralelní zpracování) |
| **Úložiště** | 2 GB volného místa | NVMe SSD s 10 GB+ volného místa |
| **Python** | Python 3.7+ (pro SDK) | Python 3.10+ |

### Instalace

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Stejné uspořádání jako u amd64 `.deb` (včetně Python SDK, přijímacích bufferů a backendových služeb), pro Jetson Orin / Orin NX / Orin Nano na **JetPack 6** (L4T R36). Jeho PyTorch je CUDA sestavení JetPack 6 pro Orin, takže zpracování na GPU, včetně debayeringu s podporou textur, běží na GPU Jetsonu. Balíček neobsahuje samotnou knihovnu CUDA: využívá knihovny CUDA a cuDNN nainstalované JetPackem 6 (`sudo apt install nvidia-jetpack` na obrazu, kde tyto knihovny chybí). Pokud není k dispozici žádný grafický procesor s podporou CUDA, zpracování probíhá na procesoru a `--debayer texture-aware` používá standardní debayer (backend zaznamená jeden řádek `[DEBAYER] … needs a CUDA GPU …` s vysvětlením důvodu). Informace o chování Jetsonu z hlediska paměti, tepelného chování a nasazení v terénu najdete v [Příručce k NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Instalace Python SDK (všechny verze Linuxu)**`.deb` nainstaluje SDK za vás.** Oba balíčky (amd64 i Jetson) jej nainstalují pro systém `python3`, takže `import chloros_sdk` funguje ihned po instalaci, bez pip a bez připojení k síti. Jeho balíčky (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` a projektové zavaděče) se umístí do `/usr/lib/python3/dist-packages`, který patří k balíčku: při aktualizaci jsou nahrazeny a při odstranění balíčku jsou odstraněny. Verze vždy odpovídá nainstalovanému rozhraní CLI a backendu.

Vlastní požadavky SDK, `python3-numpy` a `python3-requests`, jsou **doporučené, nikoli povinné** balíčkem: `sudo apt install ./chloros_<version>_<arch>.deb` je nainstaluje, `dpkg -i` nikoli. Výstup instalace uvádí verzi SDK, kterou `python3` importuje, všechny chybějící požadavky a jakékoli soubory `chloros_sdk` jinde v cestě balíčku `python3`, které přepisují kopii balíčku (například ty nainstalované dříve pomocí `pip install --user`).**Ve virtuálním prostředí (virtualenv)**, které nevidí systémové balíčky, nainstalujte do něj přiložený soubor wheel (zcela offline):

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**Z PyPI** — pouze pro počítač, na kterém není nainstalován balíček Chloros a jehož SDK řídí backend na jiném počítači (viz [Režim vzdáleného backendu](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

Volitelná doplňková volba pro instalaci pomocí pip (ve virtualenv nebo z PyPI):

| Doplněk | Příkaz | Přidá |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` pro živé streamování průběhu |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` pro přenos BLE (DAQ-M) |

Ověřte SDK:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Pythonové SDK komunikuje s backendem, který `.deb` nainstaluje, prostřednictvím lokálního HTTP API (`http://127.0.0.1:5000`). `ChlorosLocal` a pomocné programy pro připojení jej v případě potřeby automaticky spustí; uložený projekt `connect_all()` to nikdy neudělá, proto na stroji bez grafického rozhraní spouštějte backend jako [službu](#always-on-ptp-for-headless-hosts). Vždy používejte doslovnou adresu IPv4 namísto `localhost` — `localhost` se může přeložit na `::1` a zabere přibližně dvě sekundy na jeden požadavek.
{% endhint %}

***

## První nastavení

### 1. Přihlášení

Přístup k CLI a SDK vyžaduje placený tarif Chloros+ (**Copper** nebo vyšší), což je vynucováno na straně serveru: odhlášený volající obdrží adresu `401 AUTH_REQUIRED` a volající s bezplatným tarifem (Iron) obdrží adresu `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Přihlašovací údaje jsou uloženy v mezipaměti v `~/.chloros/user_session.json`.

{% hint style="info" %}
**Při upgradu zůstávají vaše přihlašovací údaje zachovány; při odstranění se vymažou.** Instalace novější verze `.deb` přes stávající instalaci (včetně instalace přes `chloros-cli update`) zachová `~/.chloros/user_session.json` a uloženou licenci v mezipaměti, takže úlohy CLI a SDK bez grafického rozhraní budou i nadále fungovat. Odstranění balíčku (`sudo apt remove chloros`) vymaže uložené přihlašovací údaje a mezipaměť licence pro každého uživatele na daném počítači, takže nová instalace bude opět vyžadovat `chloros-cli login`.
{% endhint %}

### 2. Zkontrolujte stav licence

```bash
chloros-cli status
```

`chloros-cli status` funguje na jakékoli úrovni (včetně bezplatné), takže vždy můžete zjistit, proč je nebo není přístup k dispozici.

### 3. Spusťte diagnostiku systému

```bash
chloros-cli selftest
```

Provede se sedm kontrol v daném pořadí a pokud některá z nich selže, příkaz skončí s nenulovým kódem:

| # | Kontrola | Co ověřuje |
| --- | --- | --- |
| 1 | **Verze** | CLI ohlásí svou verzi (`v1.3.0`). |
| 2 | **Port k dispozici** | Port 5000 je volný, *nebo* na něj již odpověděl funkční backend Chlorosu (což se počítá jako úspěšný test). |
| 3 | **Spuštění backendu** | Spustí se binární soubor backendu. |
| 4 | **Test API (`/api/test`)** | Backend odpoví `status: ok`. |
| 5 | **Informace o systému** | Vypíše `GPU: <name>, CUDA: <bool>, PyTorch: <version>` z `/api/system-info`. |
| 6 | **Modely odšumovače** | Najde modely `*.pth.enc` (v systému Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + odšumovač**| Funkce Texture Aware je skutečně použitelná — vyžaduje CUDA**a** alespoň jeden modelový soubor. |

Spuštění končí s `N/7 checks passed`, přičemž jsou uvedeny všechny chyby podle názvu.

### 4. Zpracujte svůj první datový soubor

```bash
chloros-cli process ~/datasets/flight001
```

***

## Soubory a adresáře

### Pro každého uživatele

Chloros ukládá své přihlašovací údaje a konfiguraci CLI do jediného adresáře pro všechny platformy, **`~/.chloros/`** (ve Windows `%USERPROFILE%\.chloros\`). Dvě mezipaměti specifické pro Linux naopak dodržují konvence XDG – při nastavení používají názvy `XDG_CONFIG_HOME` / `XDG_CACHE_HOME`.

| Cesta | Účel |
| --- | --- |
| `~/.chloros/user_session.json` | Mezipaměť přihlašovací relace zapsaná souborem `chloros-cli login` (zachována i při aktualizacích balíčku; vymazána při odstranění balíčku) |
| `~/.chloros/working_directory.txt` | Přepsání výchozí složky projektu (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Předvolba jazyka rozhraní CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Nastavení jazyka sdílené s grafickým rozhraním Windowsu — hodnota `language` zde má přednost před hodnotou `cli_language.json` |
| `~/.chloros/update_cache.json` | Hodinová mezipaměť pro kontrolu aktualizací při spuštění systému Linux/Jetson |
| `~/.chloros/backend.log` | Protokol backendu při jeho spuštění prostřednictvím CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Uložené kalibrační balíčky LATTICE pro jednotlivé kamery, indexované podle sériového čísla a hash balíčku |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Volitelné uživatelské přepsání profilů korekce kapacity DAQ |
| `~/.config/chloros/system_config.json` | Uložený profil hardwaru z Dynamic Compute Adaptation — jeho smazáním vynutíte nové rozpoznání hardwaru |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Protokoly backendového serveru, jeden soubor na jedno spuštění |
| `~/Documents/MAPIR/Chloros Projects/` | Výchozí složka projektu, pokud není nastaveno žádné přepsání |

### Systémové

| Cesta | Účel |
| --- | --- |
| `/usr/bin/chloros-cli` | Obalový skript — nastaví `LD_LIBRARY_PATH` pro přibalené nativní knihovny (a `TMPDIR` na `/mnt/ssd/tmp`, pokud tato složka existuje), poté spustí skutečný binární soubor |
| `/usr/bin/chloros-backend` | Obalový skript — totéž pro backend, který označí jako nainstalovanou produkční sestavu |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | Zkompilované CLI a backend, nainstalované v rozbaleném stavu (novinka ve verzi 1.3.0); programy jsou `cli/chloros-cli` a `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Dvouřádkové spouštěcí skripty, které spouštějí výše uvedené skutečné programy — jsou uloženy v těchto cestách, protože SDK, CLI a starší skripty je zde hledají |
| `/usr/lib/chloros/arena_runtime/` | Runtime SDKu Arena vyžadovaný kamerami LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Šifrované modely odšumovače používané debayerem Texture Aware |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (a `lattice_sdk`, `lattice_cam`, `arena_api`, načítací skripty projektu) | Pythonové SDK, nainstalované pro systém `python3` a patřící k balíčku |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Pythonové SDK ve formátu wheel odpovídající přesně této sestavě, pro virtuální prostředí |
| `/usr/lib/chloros/exiftool` | Součástí balíčku je spouštěč ExifTool (odkazovaný symlinkem na `/usr/local/bin/exiftool` pouze v případě, že neexistuje žádný systémový exiftool). Od verze 1.3.0 je skript ExifTool dodáván s vlastní knihovnou Perl v balíčku `/usr/lib/chloros/exiftool-dist/`, takže běží bez systémového balíčku `libimage-exiftool-perl` |
| `/etc/chloros/update.conf` | Konfigurace kanálu aktualizací načtená programem `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Nastaví `net.ipv4.ip_unprivileged_port_start = 319` tak, aby backend mohl navázat PTP porty bez oprávnění root |
| `/etc/sysctl.d/60-chloros-gige.conf` | Zvětší přijímací buffery socketů jádra (`net.core.rmem_max`, `net.core.rmem_default`) na 32 MB pro datové toky kamer LATTICE a nastaví je při instalaci (již nastavená vyšší hodnota zůstane beze změny). Při výchozím nastavení jádra jsou pakety z kamery v jádře zahozeny a pole ztrácí snímky i snímkovou frekvenci; `chloros-cli lattice network` hlásí stav vyrovnávacích pamětí, když jsou téměř plné. Viz [Hostitelské systémy Linuxu — vyrovnávací paměti pro příjem soketů](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Nasměruje dynamický zavaděč na `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Udělí přihlášenému uživateli přístup k sériovému mostu DAQ-U USB (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | Služba pro zapnutí trvale běžícího backendu, která spouští backend jako uživatel, kterého určíte (nainstalováno, **není povoleno**) — tu je třeba povolit, viz [Always-On PTP](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | Stejná služba spouštějící backend jako **root**(nainstalováno,**není povoleno**), ponecháno tak, aby počítač, na kterém je povoleno, zůstal v chodu |
| `/var/log/chloros/` | Podrobný protokol služby root, čitelný pro skupinu `adm` (smazán při úplném odstranění balíčku) |
| `/usr/share/applications/chloros-cli.desktop` | Položka v nabídce aplikací „Chloros CLI“, která otevírá terminál |

**Firewall (novinka ve verzi 1.3.0).** Pokud je v době instalace aktivní `ufw` nebo `firewalld`, balíček otevře UDP porty `5002` (data DAQ-E), `5353` (vyhledávání mDNS), `319` a `320` (PTP) a při odinstalování balíčku tato pravidla odstraní. Pokud je brána firewall nainstalována, ale není aktivní, nic nepřidává a vypíše porty, které je třeba povolit, pokud ji později zapnete. Neotevírá žádný port TCP: aktualizace firmwaru DAQ-E, při které se senzor připojuje zpět k tomuto počítači přes TCP za účelem sběru snímku, selže s chybou `… device never connected back on TCP <port> …`, pokud aktivní firewall toto připojení blokuje.

## Umístění spustitelného souboru backendu

CLI a SDK automaticky detekují backend:

| Komponenta | Cesta |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` (spouštěč pro `/usr/lib/chloros/backend/chloros-backend`) |

Cestu k backendu lze přepsat pomocí příznaku CLIu `--backend-exe` nebo parametru konstruktoru SDKu `backend_exe`, a port pomocí `--port` (výchozí hodnota `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` nasměruje rodiny příkazů **`lattice`**,**`project`**a**`daq pool-*`** na vzdálený backend. Základní příkazy (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) jej záměrně ignorují a vždy cílí na `http://127.0.0.1:<port>`.
{% endhint %}

***

## Kamery LATTICE a světelné senzory DAQ v systému Linux

Všechny rodiny příkazů live-hardware fungují v systému Linux (amd64 a Jetson):

* **`chloros-cli lattice`** — vyhledává, připojuje, konfiguruje a pořizuje snímky z kamer LATTICE a synchronizovaných polí. `.deb` sdružuje runtime Arena SDK, který tyto komponenty vyžadují, a registruje jej u dynamického zavaděče.
* **`chloros-cli daq pool-*`** — připojuje světelné senzory DAQ-U/M/E prostřednictvím backendového fondu, streamuje kalibrovaná spektra, zaznamenává soubory `.daq` a vyměňuje profily krytek; u DAQ-E navíc mění jeho síťová nastavení, restartuje jej a aktualizuje jeho firmware. `chloros-cli daq discover` vyhledává senzory, ke kterým se můžete připojit, a `chloros-cli daq --help` vypisuje podpříkazy `pool-*`.
* **`chloros-cli project`** — spustí uložený projekt (jeho kamery, senzory a nastavení zpracování) bez grafického rozhraní.
* **`chloros-cli time-sync`** — zkontrolujte PTP grandmaster, na kterém běží backend Chloros pro kamery LATTICE a senzory DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` je vyžadován příkazy `pool-latest`, `pool-stream`, `pool-record` a `pool-set-cap`; `pool-list` zobrazuje ID, která jsou aktuálně v poolu.

{% hint style="info" %}
**`--eth-host` přeskočí vyhledávání.** `pool-connect --eth` vyhledá zařízení DAQ-E pomocí mDNS s náhradním řešením ARP, které funguje i z prázdné mezipaměti ARP; předáním IP adresy nebo názvu hostitele senzoru pomocí `--eth-host` se k němu připojí přímo.
{% endhint %}

**Oprávnění pro sériové rozhraní DAQ-U** jsou řízena nainstalovaným pravidlem udev (`uaccess` + skupina `dialout`). Pokud zůstává již připojený senzor nepřístupný, znovu načtěte pravidla nebo jej znovu připojte:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Úplný seznam příkazů najdete v [referenci CLI](../CLI.md).

### Trvale aktivní PTP pro hostitele bez grafického rozhraní

Balíček nainstaluje dvě služby systemd pro backend, přičemž ani jedna z nich není povolena. Na hostiteli Jetson bez grafického rozhraní nebo na serveru, kde by měla synchronizace času PTP běžet nepřetržitě pro senzory DAQ-E a kamery LATTICE, nebo kde skripty SDK připojují uložené projekty (`connect_all()` nikdy nespustí backend), povolte službu **jako váš uživatel**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` spouští backend jako tento uživatel: záznamy, které zapisuje, patří tomuto uživateli, jeho podrobný protokol se nachází v jeho `~/.cache/chloros/logs` a používá jeho `chloros-cli login` (`~/.chloros`), proto se nejprve jednou přihlaste jako tento uživatel. Při instalaci balíčku se zobrazí tento příkaz s vyplněným vaším uživatelským jménem.

Bez služby běží backend (a s ním i PTP grandmaster) pouze tehdy, když jej něco spustilo — příkaz `chloros-cli` nebo skript SDK.

Jednotka váže backend na nastavení prostředí `127.0.0.1:5000` (nastavení prostředí `CHLOROS_HOST` / `CHLOROS_PORT` uvnitř jednotky; přepsat pomocí `sudo systemctl edit chloros-backend@.service`) a v případě selhání jej po 5 sekundách restartuje.

**Služba root.**`chloros-backend.service` spouští backend jako**root**. Je stále nainstalována, aby počítač, na kterém je povolena, zůstal v provozu, ale vše, co backend s oprávněním root zapíše, patří uživateli root, takže uživatel nemůže smazat své vlastní záznamy; jeho protokol se nachází v `/var/log/chloros` a je čitelný pro skupinu `adm`. Chcete-li přepnout na svého uživatele:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**Backend služby je sdílený a klient jej nikdy nezastaví.** Skripty `chloros-cli` a SDK jej používají tak, jak jej najdou: `process` na něm běží a načítá svůj vlastní projekt namísto toho posledního. Tam, kde by CLI jinak restartovalo backend (`--restart` nebo `process`, jehož import nic nenašel), označí jednotku, vypíše `sudo systemctl restart <unit>` a ukončí `1`, protože backend zastavený tímto způsobem by zůstal neaktivní. Restartujte jej sami pomocí `sudo systemctl restart chloros-backend@$USER.service`. Backend služby se nikdy sám nezastaví v nečinnosti a příkaz `chloros-cli backend stop` to odmítne a uvede název příkazu `systemctl stop`.**Jak PTP získává své porty.** PTP používá porty UDP 319/320, které jsou oba pod běžnou hranicí 1024 pro privilegované porty. Balíček `postinst` zapíše `/etc/sysctl.d/60-chloros-ptp.conf` s `net.ipv4.ip_unprivileged_port_start = 319`, což umožňuje backendu je navázat při běhu pod vaším uživatelem. Od verze 1.3.0 balíček nenastavuje na backendu žádné oprávnění pro soubory (`setcap`) a `libcap2-bin` již není závislostí: na rozbaleném backendu by oprávnění pro soubory zabránilo jeho spuštění, takže ji sami nepřidávejte.

***

## Příklady skriptů v Bashu

{% hint style="info" %}
**Výstupní kódy vhodné pro skriptování.**`chloros-cli process` ukončí `0` v případě úspěchu a**vydá nenulový kód v případě selhání — včetně běhu, který požadoval obrazové produkty, ale žádný nezapsal** (vypíše `Processing finished but wrote no image products.` a uvede název projektové složky a obvyklé příčiny). Úspěšná spuštění hlásí, kolik obrazových produktů bylo zapsáno (`Image products written: N`, plus `(plus M clamp masks)`, pokud byly zapsány masky). Kódy ukončení: `0` úspěch, `1` selhání, `2` chyba argumentu, `130` přerušeno.
{% endhint %}

### Zpracování více datových sad

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Zpracování s vlastními nastaveními

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

Platných hodnot `--format` jsou přesně čtyři a obsahují mezery – vždy je uveďte v uvozovkách:

| Hodnota `--format` | Výstupní složka |
| --- | --- |
| `TIFF (16-bit)` *(výchozí)* | `tiff16` |
| `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` přijímá `standard` (výchozí) nebo `texture-aware` (Chloros+).

### Automatizované zpracování pomocí Cronu

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Příklad PythonSDK

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Řešení problémů

### Po instalaci nelze najít CLI

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Oprávnění odepřeno

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### Backend se nespustí po přidání oprávnění k souborům

Nespouštějte `setcap` na backendu. Od verze 1.3.0 balíček nenastavuje žádné oprávnění souborů a porty PTP 319/320 pocházejí z nastavení `sysctl` popsaného v části [Jak PTP získává své porty](#always-on-ptp-for-headless-hosts). Souborová schopnost uvede rozbalený backend do režimu bezpečného spuštění, ve kterém nemůže najít své vlastní knihovny, a proto se nespustí. Pokud jste nějakou přidali, odstraňte ji:

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### PTP se nespustí / Nelze navázat port 319

Ověřte, zda byla snížena minimální hodnota pro neprivilegované porty, a pokud ne, znovu ji nastavte pro aktuální spuštění:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Poté zkontrolujte grandmaster:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### `lattice` Příkazy selhávají s chybou „LATTICE camera drivers not found“

Backend nemohl načíst runtime kamery LATTICE, se kterým je nainstalován. Každý příkaz `lattice` se spouští na backendu; samotné CLI neobsahuje žádné ovladače kamer. Přeinstalujte Chloros (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`): `.deb` nainstaluje runtime vedle backendu, v `/usr/lib/chloros/arena_runtime`, a zaregistruje jej u dynamického zavaděče. Chcete-li tuto registraci ověřit:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
ls /usr/lib/chloros/arena_runtime | head
```

### Spuštění backendu se nezdařilo

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

Protokoly backendu týkající se neúspěšného spuštění se nacházejí v souboru `~/.cache/chloros/logs/`. Pokud port 5000 patří službě backendu (`systemctl status chloros-backend@$USER.service`), restartujte službu, místo abyste ukončili její proces.

### CUDA nebyla detekována

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

`chloros-cli selftest` hlásí totéž v jednom řádku: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Chybějící sdílené knihovny

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### Pomalé spouštění na systémech s SD kartou

Od verze 1.3.0 se backend a CLI instalují v rozbaleném stavu, takže se již při každém spuštění nerozbalují do dočasného adresáře. Dočasné soubory se automaticky ukládají do adresáře `/mnt/ssd/tmp`, pokud tento adresář existuje; v opačném případě je můžete nasměrovat do rychlého souborového systému:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Aktualizace Chlorosu na Linuxu

Příkaz `update` je k dispozici pouze pro Linux/Jetson. Zkontroluje verzi zveřejněnou v aktualizačním kanálu nakonfigurovaném v `/etc/chloros/update.conf` a nabídne stažení a instalaci odpovídajícího `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

V systému Linuxu/Jetsonu provádí CLI při každém spuštění také neblokující kontrolu aktualizací (výsledek je uložen v mezipaměti po dobu jedné hodiny v `~/.chloros/update_cache.json`) a v případě existence novější verze vypíše `Update available: vX.Y.Z`. Vaše nastavení, projekty a přihlašovací údaje zůstanou po aktualizaci zachovány.

Aktualizace balíčku během instalace zastaví všechny backendové služby, takže spuštěný skript ztratí přístup ke kamerám a senzorům a poté znovu spustí všechny, které jsou povoleny (skript spuštěný ručně, ale nikdy nepovolený, zůstane zastavený, stejně jako po restartu).

## Odinstalace

```bash
sudo apt remove chloros
```

Odinstalace zastaví a deaktivuje všechny backendové služby, obnoví výchozí minimální hodnotu pro neprivilegované porty (1024), odstraní konfigurační soubor pro přijímací buffer (aktuální hodnoty zůstanou zachovány až do restartu), symbolický odkaz na nástroj exiftool, konfiguraci zavaděče Arena, pravidla brány firewall přidaná balíčkem a nainstalované Python SDK a vymaže uložené přihlašovací údaje z mezipaměti. Vaše projekty a datové soubory `~/.chloros/` zůstanou nedotčeny. `sudo apt purge chloros` také odstraní `/var/log/chloros`.

***

## Další kroky

* [Průvodce NVIDIA Jetson](nvidia-jetson-guide.md) — Optimalizace a nasazení specifické pro Jetson
* [CLI: Příkazový řádek](../CLI.md) — průvodce CLI
* [API: Python SDK](../api-python-sdk.md) — průvodce SDK
* [Referenční příručka CLI](../reference/cli-reference.md) a [Referenční příručka SDK](../reference/sdk-reference.md) — vyčerpávající seznamy příkazů a API pro verzi 1.3.0
* [Dynamická adaptace výpočetního výkonu](../processing-architecture/dynamic-compute-adaptation.md) — jak se Chlorospřizpůsobuje vašemu hardwaru
