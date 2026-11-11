# Používání Chlorosu s AI asistenty

Tento manuál je určen pro dvě cílové skupiny: lidi a AI asistenty, se kterými lidé stále častěji pracují. Každá stránka obsahuje přesné hodnoty, výchozí nastavení a příkazy, které lze zkopírovat a vložit, aby asistent (Claude, ChatGPT, Copilot, programovací agent atd.) mohl na první pokus napsat funkční automatizaci v Chloros.

Verze Chloros: **

1.3.0**. Platformy CLI/SDK: Windows 10/11 x64 a Linux (x86\_64 / Jetson aarch64).

## Co předat vašemu asistentovi

| Zdroj | URL | K čemu slouží |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Strojově čitelný index všech stránek v této příručce. |
| **Referenční příručka CLI** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Kompletní přehled příkazů `chloros-cli`: všechny příkazy, příznaky, výchozí hodnoty, kódy ukončení a pravidla pro výstupní složky. Napsáno pro použití v LLM. |
| **Referenční příručka k SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Kompletní Python API pro `chloros_sdk`: třídy, signatury, výjimky a praktické příklady. Určeno pro použití v LLM. |
| **Libovolná stránka ve formátu surového Markdownu** | připojte `.md` k URL stránky | např. `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` vrátí stránku ve formátu surového Markdownu — ideální pro vložení do kontextového okna nebo načtení z agenta. |

Odkazy v manuálu: [Referenční příručka CLI](reference/cli-reference.md) · [Referenční příručka SDK](reference/sdk-reference.md).

{% hint style="info" %}
Obě referenční stránky jsou samostané: asistent, který si jednu z nich přečetl, nepotřebuje zbytek příručky k napsání správného skriptu.
{% endhint %}

## Nechte svého asistenta spustit Chloros přímo: `chloros-mcp`

Ve všech výše uvedených případech váš asistent *napíše* skript nebo příkaz. S `chloros-mcp` asistent, který komunikuje pomocí [Model Context Protocol](https://modelcontextprotocol.io) (MCP), jako například Claude Desktop, Claude Code nebo Cursor, řídí samotný program Chloros. Stačí zadat požadavek v běžném jazyce a asistent vyvolá nástroje serveru:

* **Zpracování snímků:** vytvoření projektu, import složky se snímky, výběr nastavení, zpracování a vygenerování zprávy o výsledcích.
* **Ovládání hardwaru:** připojí kamery LATTICE, pole a světelné senzory DAQ, zobrazí náhled, nastaví expozici a barvy, pořídí snímky, zaznamená data a aktualizuje firmware.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

Nástroje serveru pokrývají funkce [Python SDK](api-python-sdk.md). Instalují se společně s SDK, takže obvykle není třeba instalovat nic dalšího: ve Windows instalační program Chloros umístí `chloros-mcp.exe` do složky `Scripts` vašeho Pythonu, a v systému Linux `.deb` umístí soubor `chloros-mcp` do adresáře `/usr/bin`.

**Kód Claude:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**přidejte tento záznam do souboru `claude_desktop_config.json` (otevřete jej v nabídce**Nastavení &gt; Vývojář &gt; Upravit konfiguraci**) a poté restartujte Claude Desktop.**Kurzor** vyžaduje stejný záznam v souboru `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

Pokud klient hlásí, že se server nepodařilo spustit, pravděpodobně není položka `chloros-mcp` uvedena v seznamu `PATH`, který klient vidí: zadejte místo toho úplnou cestu (`where chloros-mcp` ve Windowsu, `which chloros-mcp` v Linuxu).

Na stejném počítači musíte mít nainstalovaný Chloros a být tam přihlášeni, a to buď v aplikaci, nebo pomocí `chloros-cli login`, a to v jakékoli placené úrovni Chloros+ (Copper nebo vyšší). Server se za vás nemůže přihlásit.

{% hint style="info" %}
Nastavení klienta pro každou aplikaci, možnosti, jednotlivé nástroje a chování při spuštění: [MCP Server (AI Assistants)](reference/sdk-reference.md#mcp-server-ai-assistants) v referenční příručce k SDK.
{% endhint %}

## Šablony příkazů

Zkopírujte, vyplňte `<placeholders>` a vložte do svého asistenta.

### 1. Zpracování složky letových dat do formátu NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Hromadné sledování adresáře se zachycenými daty

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

### 4. Záznam spekter světelných senzorů DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
Skriptování DAQ z příkazového řádku vždy probíhá prostřednictvím rodiny `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Ostatní podpříkazy typu `daq`, které si váš asistent může vymyslet, nejsou v dodávaných verzích k dispozici a jejich spuštění skončí chybou.
{% endhint %}

## Proč skripty napsané umělou inteligencí dobře fungují s Chlorosem

Každý z těchto příkladů představuje skutečné a ověřené chování Chlorosu 1.3.0 – odstraňují klasické způsoby selhání automatizace napsané strojem:

* **Žádné složité nastavování.**Pomocné funkce SDK pro inteligentní připojení (`connect_camera`, `connect_array`, `connect_daq_sensor`) a vstupní body zpracování (`ChlorosLocal`, `process_folder`)**automaticky spouštějí lokální backend**. Generovaný skript nepotřebuje otevřené grafické rozhraní ani ručně spuštěný server — stačí pouze nainstalovaný balíček pro desktop nebo CLI.
* **Celý proces je jedno volání.** `chloros_sdk.process_folder("path", indices=["NDVI"])` provádí import → kalibraci → odrazivost → export indexu od začátku do konce. Menší rozsah, méně míst, kde by se generovaný skript mohl pokazit.
* **Spuštění bez výstupu provádí autodiagnostiku.** Po skončení `process()` je k výsledku připojen souhrn běhu a každý tip ke zpracování (např. *proč* běh nevytvořil žádný výstup) je rovněž znovu vygenerován jako Pythonový skript `UserWarning` — takže i skript, který nikdy nekontroluje výsledný slovník, zobrazí tuto diagnostiku.
* **CLI selže s jasným hlášení.**Běh `chloros-cli process`, který požadoval výstupy, ale žádný nezapsal, vypíše `Processing finished but wrote no image products.` a**ukončí se s nenulovým kódem**, takže shellové skripty a CI to detekují jednoduchou kontrolou kódu ukončení. Úspěšná spuštění hlásí `Image products written: N` a navíc `(plus M clamp masks)`, pokud byly zapsány masky. Příkazy pro zachycení rovněž uvádějí názvy všech souborů, do kterých se jim nepodařilo zapsat, a ukončí se s kódem `1`.
* **Relativní cesty znamenají to, co skript očekává.** U lokálního backendu se relativní výstupní cesta (`-o out/` v CLI, `output_dir="out/"` v SDK) řeší vzhledem ke složce, ze které se příkaz nebo skript spouští.

Jedna nesrovnalost, kterou by měl asistent znát: `process()` v SDK záměrně **nevyprovokuje** výjimku při běhu s nulovým výstupem — místo toho to nahlásí prostřednictvím souhrnu/tipů. Pokud se Pythonový pipeline musí při prázdném běhu zastavit, zkontrolujte souhrn (recept 2 to dělá).

## Upozornění

* **Vyžaduje se přihlášení k Chloros+.**CLI a SDK vyžadují**placenou** úroveň Chloros+, což je vynucováno na straně serveru: požadavky selžou s chybou `401 AUTH_REQUIRED`, pokud nejste přihlášeni, a s chybou `403 PLAN_UPGRADE_REQUIRED` v bezplatné úrovni. Před spuštěním vygenerovaných skriptů spusťte na každém stroji jednou příkaz `chloros-cli login`. Viz [Přihlášení do Chloros+](chloros+-login.md).
* **Příkazy pro snímání ovládají skutečný hardware.** Příkazy `lattice` / `daq` / `project`, objekty relace SDK a nástroje `chloros-mcp` se připojují k fyzickým kamerám a senzorům, přenášejí z nich data a spouštějí je. Před prvním spuštěním zkontrolujte vygenerovaný skript a spusťte jej za přítomnosti hardwaru.
* **Provedťe namátkovou kontrolu výstupů.** Před zveřejněním výsledků ověřte složky s výstupy a několik hodnot pixelů. Zejména soubory TIFF s odrazivostí jsou škálovány podle zdroje – přečtěte si XMP tag `Chloros:PixelScale` (soubory LATTICE jej obsahují: 32768 = odrazivost 1,0; soubory Survey3 jej neobsahují a používají 65535 = 1,0) místo předpokládání dělitele. Obě referenční stránky to dokumentují v části „Čtení pixelů odrazivosti“.
* **Drobné úskalí, která mohou způsobit potíže s generovaným kódem:**`pool-record` zapisuje do souborového systému**backendového hostitele** (výchozí nastavení `~/Documents/DAQ Live View/`); na počítačích s několika síťovými rozhraními upřednostněte `daq pool-connect --eth-host <ip-or-hostname>` před automatickým vyhledáváním; použijte `http://127.0.0.1:5000` (nikdy `localhost`) všude tam, kde se objevuje URL backendu; skript, který prochází výstupní stromovou strukturu projektu, by měl přeskočit složky `clamp_masks/` (jejich soubory `*_clampmask.tif` jsou masky, nikoli obrázky — viz [Formáty výstupních obrázků](output-image-formats.md#clamp-masks)); a ve Windows, kde je vypnutá funkce dlouhých cest, udržujte názvy projektových složek krátké — projektová složka s názvem delším než přibližně 130 znaků bude před zahájením zpracování odmítnuta.
