# Používání Chloros s AI asistenty

Tento manuál je určen pro dvě cílové skupiny: lidi a AI asistenty, s jejichž pomocí lidé stále častěji pracují. Na každé stránce jsou uvedeny přesné hodnoty, výchozí nastavení a příkazy, které lze zkopírovat a vložit, aby asistent (Claude, ChatGPT, Copilot, programovací agent atd.) mohl na první pokus napsat funkční automatizaci Chloros.

Verze Chloros: **

1.2.0**. Platformy CLI/SDK: Windows 10/11 x64 a Linux (x86\_64 / Jetson aarch64).

## Co předat vašemu asistentovi

| Zdroj | URL | K čemu slouží |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Strojově čitelný index všech stránek v této příručce. |
| **CLI Reference** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Kompletní přehled příkazů `chloros-cli`: všechny příkazy, příznaky, výchozí hodnoty, kódy ukončení a pravidla pro výstupní složky. Určeno pro použití v LLM. |
| **SDK Reference** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Kompletní `chloros_sdk` Python API: třídy, podpisy, výjimky a praktické příklady. Napsáno pro použití v LLM. |
| **Libovolná stránka jako surový Markdown** | připojte `.md` ke stránce URL | např. `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` vrátí stránku ve formátu surového Markdownu — ideální pro vložení do kontextového okna nebo načtení z agenta. |

Odkazy v manuálu: [CLI Reference](reference/cli-reference.md) · [SDK Referenční stránka](reference/sdk-reference.md).

{% hint style="info" %}
Tyto dvě referenční stránky jsou samostané: asistent, který si jednu z nich přečetl, nepotřebuje zbytek příručky, aby napsal správný skript.
{% endhint %}

## Vzorové příkazy

Zkopírujte, vyplňte `<placeholders>` a vložte do svého asistenta.

### 1. Zpracování složky letů do NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Hromadné sledování adresáře se záznamy

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Připojení pole LATTICE a pořízení dat

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Zaznamenání spekter světelného senzoru DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
Skriptování DAQ z příkazového řádku vždy probíhá prostřednictvím rodiny `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Ostatní podpříkazy typu `daq`, které si váš asistent může vymyslet, nejsou v dodávaných verzích k dispozici a vedou k ukončení s chybou.
{% endhint %}

## Proč skripty napsané umělou inteligencí dobře fungují s Chloros

Každý z těchto příkazů představuje skutečné a ověřené chování Chloros 1.2.0 — odstraňují klasické způsoby selhání automatizace napsané strojem:

* **Žádné složité nastavování.**Pomocné funkce SDK pro inteligentní připojení (`connect_camera`, `connect_array`, `connect_daq_sensor`) a vstupní body zpracování (`ChlorosLocal`, `process_folder`)**automaticky spouštějí lokální backend**. Generovaný skript nevyžaduje otevřené grafické rozhraní ani ručně spuštěný server — stačí, aby byl nainstalován balíček desktop/CLI.
* **Celý proces je jedno volání.** `chloros_sdk.process_folder("path", indices=["NDVI"])` provádí import → kalibraci → odrazivost → export indexu od začátku do konce. Menší rozsah, méně míst, kde by se generovaný skript mohl pokazit.
* **Spuštění bez výstupu provádí autodiagnostiku.** Po `process()` je k výsledku připojeno shrnutí běhu a každý tip ke zpracování (např. *proč* běh nevytvořil žádný výstup) je rovněž znovu odeslána jako Python `UserWarning` — takže i skript, který nikdy nekontroluje slovník výsledků, zobrazí diagnózu.
* **CLI selže s výraznou chybou.**Běh s kódem `chloros-cli process`, který požadoval výstupy, ale žádný nezapsal, vypíše kód `Processing finished but wrote no image products.` a**ukončí se s nenulovým kódem**, takže skripty v shellu a CI to detekují pouhou kontrolou kódu ukončení. Úspěšná spuštění hlásí kód `Image products written: N`.

Jedna nesrovnalost, o které by asistent měl vědět: `process()` v rámci SDK záměrně **nevzbudí** výjimku při běhu bez produktů — místo toho to nahlásí prostřednictvím souhrnu/tipů. Pokud se musí pipeline Python zastavit při prázdném běhu, zkontrolujte souhrn (recept 2 to dělá).

## Upozornění

* **Chloros+ vyžaduje přihlášení.**CLI a SDK vyžadují**placenou** úroveň služby Chloros+, vynucenou na straně serveru: požadavky s kódem chyby `401 AUTH_REQUIRED` selžou, pokud nejste přihlášeni, a s kódem chyby `403 PLAN_UPGRADE_REQUIRED` v bezplatné úrovni. Před spuštěním vygenerovaných skriptů spusťte na každém počítači příkaz `chloros-cli login`. Viz [Chloros+ Přihlášení](chloros+-login.md).
* **Příkazy pro zachycení řídí skutečný hardware.** Příkazy `lattice` / `daq` / `project` a objekty relace SDK navazují spojení s fyzickými kamerami a senzory, přenášejí z nich data a spouštějí je. Před prvním spuštěním zkontrolujte vygenerovaný skript a spusťte jej za přítomnosti hardwaru.
* **Provedete namátkovou kontrolu výstupů.** Před zveřejněním výsledků ověřte složky s výstupy a několik hodnot pixelů. Zejména soubory TIFF s odrazivostí jsou škálovány podle zdroje — přečtěte si XMP tag `Chloros:PixelScale` (LATTICE: 32768 = odrazivost 1,0; Survey3: 65535) místo toho, abyste předpokládali dělič. Obě referenční stránky to dokumentují v části „Čtení pixelů odrazivosti“.
* **Drobné úskalí, která mohou způsobit chyby v generovaném kódu:**`pool-record` zapisuje do souborového systému**hostitelského backendu** (výchozí nastavení `~/Documents/DAQ Live View/`); na počítačích s několika síťovými rozhraními upřednostněte `daq pool-connect --eth-host <ip-or-hostname>` před automatickým zjišťováním; a kdekoli se objeví backend URL, použijte `http://127.0.0.1:5000` (nikdy `localhost`).
