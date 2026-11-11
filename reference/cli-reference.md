# Referenční příručka k Chloros CLI

**Verze:**

1.3.0**Vytvořeno:**

29. 7. 2026 19:19 ·**Revidováno:**

6. 11. 2026**Cílová skupina:** Optimalizováno pro použití LLM; čitelné pro člověka.**Rozsah:** Všechny podpříkazy nástroje `chloros-cli` určené pro uživatele, včetně voleb a příkladů, které lze zkopírovat a vložit.

Tento dokument představuje kompletní referenční příručku k nástroji příkazového řádku `chloros-cli`, který je součástí balíčku MAPIR Chloros. Je záměrně vyčerpávající, aby LLM (nebo člověk) mohl sestavit jakýkoli podporovaný pracovní postup z níže uvedených seznamů bez nutnosti prohlížet zdrojový kód.

Pokud potřebujete pouze to nejdůležitější, přejděte na:
- [Pětiminutový rychlý start](#five-minute-quickstart)
- [Pracovní postup pro první připojení kamery LATTICE](#lattice-camera-first-connect-workflow)
- [Pracovní postup pro první připojení snímače DAQ](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Režimy snímání, záznamníky a offline zpracování](#capture-modes-recorders--offline-reprocess)

> **Ovládáte Chloros pomocí AI asistenta?** Claude Desktop, Claude Code, Cursor a další klienti MCP mohou spouštět zpracování, kamery a pole LATTICE a světelné senzory DAQ prostřednictvím `chloros-mcp`, server MCP, který se instaluje spolu s Python SDK (a s balíčkem .deb pro Linux, v `/usr/bin`): `claude mcp add chloros -- chloros-mcp`. Nastavení, možnosti a nástroje najdete v referenční příručce SDK v části [MCP Server (AI asistenti)](sdk-reference.md#mcp-server-ai-assistants). Asistent s příkazovým řádkem může také přímo spustit každý příkaz na této stránce.

> **Změny ve verzi 1.3.0** (co si všimne stávající skript nebo pracovní postup verze 1.2.0).
>
> **Změny, které může být nutné ve skriptu verze 1.2.0 zpracovat:**
>
> - **Platí `--preset`.** `lattice capture`, `continuous`, `calibrate`, `multi-capture`, `multi-stream`, `align-calibrate`, `align-apply` a `align-stream` byly ve verzi 1.2 odstraněny.0 a fungovaly na základě vlastních nastavení kamery. Nyní nastavují formát předvolby, expozici, zesílení a snímkovou frekvenci: `--preset high_speed` snímá v 8bitovém formátu při 5 ms, 6 dB, 60 snímků za sekundu. `lattice cam-connect --preset` zachovává také formát předvolby (verze 1.2.0 nastavila kameru na 12 bitů po použití předvolby). Viz [Společné možnosti](#common-options-shared-by-most-subcommands).
> - **Formát Bayerových pixelů u kamery M3M (mono) představuje její hloubku.** Předvolby uvádějí formáty Bayer, které mono snímač nedokáže zachytit: ve verzi 1.2.0 kamera tento formát odmítla a zachovala ten, který měla, takže `--preset high_speed` tam nebyl 8bitový. Nyní se jedná o formát Mono se stejnou hloubkou: `high_speed` a `--pixel-format BayerRG8` dávají `Mono8`, `high_quality`, `triggered` a `BayerRG12` dávají `Mono12`. Ve smíšeném poli M3C + M3M každá kamera používá sdílený formát ve své vlastní rodině a `network-analysis` / `analyze-array --pixel-format` promítají každou kameru v dané hloubce. Totéž platí i opačně: kamera M3C, pro kterou byl vyžádán formát Mono, přebírá formát Bayer dané hloubky, takže pole, jehož hlavním prvkem je M3M, již své kamery M3C v režimu Mono znovu neotevírá. Viz [Jednopásmové kamery Mono (M3M)](#mono-m3m-single-band-cameras).
> - **Exporty `lattice array-capture` jsou ve výchozím nastavení zarovnány, stejně jako exporty aplikace.** Soubory `debayered`, `display`, `radiance`, `reflectance` a indexové soubory jsou transformovány do snímku hlavní kamery a oříznuty na překryv členů (menší než snímač o posuny kamer, `XMP-mapir:AlignmentJSON` `"applied": true`), takže záznam z CLI obsahuje stejné soubory jako záznam stejné scény z aplikace Chloros; `raw` zůstává v nativním formátu. Dříve se při záznamu bez `--aligned` ukládaly všechny soubory v nativní velikosti, bez přizpůsobení. `--no-aligned` toto zachovává. Stejné výchozí nastavení platí pro `--smart`, `--continuous` a `--interval`, stejně jako pro SDK a `chloros-mcp`. Viz [režimy `array-capture`](#array-capture-modes).
> - **`lattice array-capture --compression` je ve výchozím nastavení nastaven na `none`.** Soubory se ukládají nekomprimované (přibližně 6,3 MB na snímek v plném rozlišení) pro nejrychlejší trvalou přenosovou rychlost; verze 1.2.0 ukládala ve formátu `deflate` (přibližně 4,1 MB). Pro menší soubory zadejte `--compression deflate`. Oba formáty jsou bezztrátové.
> - **Formáty PNG a JPEG podporují pouze režimy `display` / `preview` a indexované soubory.** Snímání s příponou `--format png` nebo `jpg` s úrovní, kterou 8bitový soubor nedokáže pojmout, je odmítnuto ještě před zahájením snímání (kód HTTP 400, ukončení `1`), stejně jako výchozí `--processing all` s `--format png`/`jpg`. Viz [Úrovně exportu zachycení](#capture-export-levels-the-all-default).
> - **`lattice dls` vyžaduje `--mac`.**Již neexistuje výchozí MAC (byla to adresa jedné jednotky); `daq pool-discover --only ble` vyhledá adresu senzoru.**`lattice dls --camera SERIAL`** je nový.
> - **Další chyby ukončí `1`.** `process`, pokud nebylo možné ověřit exporty běhu nebo došlo ke ztrátě obrazu, i když ostatní produkty jsou zapsány; příkaz zachycení s jakýmkoli souborem, který se nepodařilo zapsat (ostatní soubory se stále ukládají); `lattice continuous` a `multi-stream` při zaplněném disku; `array-burst` / `array-record`, když záznamník nahlásí chybu; `array-build-video`, když byl požadovaný produkt přeskočen nebo nebyl vytvořen podle požadavku; `project burst` / `stream`, když zachycení selže v průběhu; `project sensor log`, pokud záznam neobsahuje žádné údaje; `lattice exposure --set`, `gain --set` a `configure --set` s hodnotou, kterou kamera odmítá. `lattice network-analysis --json` končí stejně jako souhrn (`1` pro `needs_force_slip`). Naopak: `daq pool-disconnect` senzoru, který již není v poolu, ukončí `0`. Viz [Kódy ukončení](#exit-codes).
> - **Pomalý proces `lattice array-connect` nikdy neselže, pokud je pole připojeno.** Čeká až 600 s (180 s ve verzi 1.2.0; `project connect` 90 s), přičemž průběžně vypisuje kroky backendu, a připojení, které do té doby nebylo dokončeno, je backendem zrušeno — každá kamera se vrátí do stavu, v jakém byla nalezena — takže kód `Error:` (ukončení `1`) znamená, že není připojeno žádné pole. Ve verzi 1.2.0 mohlo připojení provedené ihned po zapnutí kamer trvat déle než čekací doba a připojit pole až po selhání příkazu. Kombinace kláves Ctrl+C nyní připojení zruší. Viz [Trvalá připojení](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-connect` zachovává celý snímač.** Bez `--width` / `--height` / `--binning` je každá kamera nastavena na plný snímač a snímková frekvence klesne na úroveň, kterou poskytuje tento počítač a síť; výstup příkazu connect uvádí rychlejší možnosti (binning nebo menší oblast zájmu (ROI)) s jejich předpokládanými snímkovými frekvencemi. Verze 1.2.0 zmenšila nebo seskupila snímky, aby udržela snímkovou frekvenci. Viz [Možnosti `array-connect`](#array-connect-options).
> - **Pomalá síť již nebrání provozu pole.** Každá kamera je přizpůsobena své části šířky pásma a snímková frekvence se tím snižuje. Pokud kamery nelze přizpůsobit tak pomalu, pole běží v režimu sériového vysílání (`slip-emit-and-capture`): každý snímek dorazí, ale kamery nejsou exponovány současně. `--force-tier sim-capture-sim-emit` vyžaduje simultánní expozici. Viz [Snímková frekvence pole a model sériového snímání](#array-fps--burst-model).
> - **`lattice network-analysis` / `analyze-array` určují, jaké připojení se použije**: bez `--pixel-format` určuje formáty nastavené v connect (Mono12 / BayerRG12, dříve BayerRG8) a volbu [zkomprimovaný 12bitový přenos](#packed-12-bit-transport---packed). Stejný příkaz proto může uvádět jinou snímkovou frekvenci než ve verzi 1.2.0.
> - **Relativní cesty se počítají od místa, kde příkaz spustíte.** `-o` a další cesty, které backend otevírá (`--input`, `--burst-dir`, `--profile`, `--save`, `--mask`), `output` kroku receptu, a výchozí `output/` z `lattice index --live` se vyřeší vzhledem ke složce, ze které spouštíte `chloros-cli`. Ve verzi 1.2.0 je backend vyhodnocoval ve své vlastní instalační složce, což v případě nainstalované sestavy vedlo k chybě „Přístup byl odepřen“. Viz [Konvence](#conventions).
> - **`lattice index` běží na backendu v jakékoli formě**, stejně jako ostatní příkazy `lattice`, proto nejprve spusťte backend. Offline index (`--input`) je vypočítáván backendem, který načte soubor TIFF a zapíše výsledky vedle něj nebo do adresáře `-o`, a příkazy `--list-presets` / `--list-gradients` vytisknou tabulky backendu. Ve Windowsu nyní fungují: dříve Windows CLI reagovalo na každou formu `lattice index`, včetně `--live`, odpovědí `Error: lattice_sdk not available`. V systému Linux se offline index dříve spouštěl v CLI bez backendu a bez přihlášení; nyní potřebuje obojí, stejně jako každý jiný příkaz CLI. Viz [Index / Vegetation Maths](#index--vegetation-maths).
> - **Nové soubory vedle produktů.** Každý produkt radiance, odrazivosti a indexu, ve kterém bylo v rámci rozkladu pásem omezováno (clamped) některé pixely, získá masku ve složce `clamp_masks/` vedle složky produktu a v výpisu se zobrazí řádek `clamp mask:`. Viz [Maska omezení při rozkladu](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on).
> - **Orientace kamery.** Záznam z jedné kamery ukládá soubory `raw` a `debayered` tak, jak je snímač načítá, a zaznamenává převrácení / otočení kamery do souboru XMP; soubor `process` tuto informaci aplikuje na vytvořené produkty. Soubory členů pole zůstávají v původním formátu, pokud nepředáte nový `--orient` (`array-capture`, `array-burst`, `project capture` / `burst` / `stream`). Viz [Orientace fotoaparátu](#jak-vypadá-složka-s-zachycenými-snímky).
> - **Název úrovně exportu, který není „one“, bude odmítnut.** Seznam `lattice capture --processing` a všechny seznamy `--levels` zahrnují `raw`, `debayered`, `radiance`, `reflectance` nebo `display` (nebo alias, jako je `preview`; `--processing` a také `all`). Jakýkoli jiný název je odmítnut ještě před dotykem kamery (HTTP 400 `UNKNOWN_LEVEL`, ukončení `1`); 1.2.0 pro ni uložil `raw` a ukončil činnost `0`.
> - **`daq pool-set-cap` vyžaduje `--cap-id`.** Bez něj se příkaz ukončí `2`; 1.2.0 odstranila korekci krytu, čímž zůstal holý snímač. `--cap-id none` vyžaduje holý snímač.
> - **`lattice resolution --binning-mode` již neexistuje.** Binning je vždy nastaven na průměr; příznak je stále akceptován, vypíše varování a neprovede žádnou akci. Viz [Nastavení snímače](#sensor-tuning).
> - **`process --debayer texture-aware` je u snímků LATTICE odmítnut.** U složky obsahující snímky LATTICE ukončí `process` po importu `1` a před jakýmkoli zpracováním uvede počet těchto snímků. Snímky LATTICE vždy procházejí standardním demosaikem, takže verze 1.2.0 tuto volbu přijala a ignorovala; aplikace Chloros ji pro ně rovněž nenabízí. Viz [`chloros-cli process`](#chloros-cli-process).
> - **Kamery se vrátí do stavu, v jakém byly nalezeny — pokud není zadán příkaz `--keep-settings`.** `lattice array-disconnect`, každý přechodný příkaz (`multi-capture` bez pole, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`) a `lattice continuous` vrátí každou kameru do původního stavu: její zapojení spouště a synchronizační linky, její PTP a všechna nastavení kamery, která se během relace změnila (expozice a zesílení, cíl AE, limity a tlumení, vyvážení bílé, úroveň černé, formát pixelů, binning a ROI, převrácení, snímková frekvence). `--keep-settings` naopak uloží nastavení každé kamerydo vlastní paměti kamery, takže se s nimi spustí; `--clear-settings` tuto paměť resetuje na tovární nastavení. V obou případech jsou kamery ponechány v režimu volného běhu. Projekt nastaví své kamery podle vlastních uložených nastavení, jakmile je připojí. Viz [Paměť kamery](#camera-memory----keep-settings----clear-settings).
> - **Všechny režimy ukládají projekt stejným způsobem.** Jakákoli změna kamery, pole nebo světelného senzoru připojeného v rámci projektu – provedená z aplikace, CLI, SDK nebo serveru MCP – se uloží do souboru `cameras.json` / `sensors.json`, a otevření projektu v jakémkoli režimu nastaví zařízení podle něj. Dříve ukládala projekt pouze aplikace a její další uložení přepsalo vše ostatní. Viz [Jeden záznam projektu, každý režim](#one-project-record-every-mode).
> - **Soubory vytvořené z několika kamer uvádějí název každé z nich.** Vícepásmový stack (`align-apply`, `align-stream`, `index --live --save-multiband`), živý index vytvořený z pásem několika kamer a snímek `composite/` z hromadného snímání obsahují sériové číslo, model a kalibrační balíček každé zdrojové kamery ve formě seznamů (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), zatímco EXIF `SerialNumber` uvádí sériová čísla oddělená čárkami. Dříve neobsahovaly žádné údaje o kameře vůbec, nebo pouze sériové číslo první kamery. Viz [Soubory vytvořené z několika kamer](#files-made-from-several-cameras).
> - **Pouze soubor `raw` obsahuje kalibrační značky kamery.** Sada kompatibilní s MicaSense (`BlackLevel`, `RadiometricCalibration`, soubor pro jednotlivé kanály `Band{R,G,B}RadiometricCalibration`, sady `MicaSenseVignetting*`) a užitečné zatížení `CalibrationJSON` popisují surové hodnoty senzoru, takže jsou obsaženy pouze v souboru záznamu `raw`. Všechny ostatní soubory — soubory záznamu `debayered`, `display`, `radiance`, `reflectance` a indexové soubory, snímky `lattice continuous` a `multi-stream`, a každý produkt typu `process` — obsahuje identifikátor kamery (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) a při pořízení snímku také vlastní záznam snímku (`XMP-mapir:FrameJSON`: expozice, zesílení, teplota, formát pixelů, čas pořízení). Ve verzi 1.2.0 obsahovaly také sadu surových dat, na kterou nástroj (Pix4D, Metashape) aplikoval na pixely, které nejsou surovými hodnotami. `process` vyhledá kalibraci takového souboru podle jeho sériového čísla a `CalBundleSha256`.
>
> **Další změny:**
>
> - **Automatická snímková frekvence pole se přizpůsobuje tomu, co pole poskytuje.** Frekvence, u které dochází ke ztrátě více než 1 % spouštěcích signálů, se sníží; toto posouzení se provádí na základě několika delších pozorování namísto jediného, a frekvence, ze které byla snížena, se čas od času znovu vyzkouší. Frekvence, kterou nastavíte sami, zůstane beze změny.
> - **Automatická expozice je stabilnější.** Udržuje se při konstantním osvětlení, ustálí se během několika sekund po změně, ignoruje drobné přesvětlené oblasti a obnoví funkci fotoaparátu, který přestane reagovat na přímém slunci. Viz [Jak se chová automatická expozice](#how-auto-exposure-behaves).
> - **`process` počítá pouze vaše snímky a nahrávky.** Při opakovaném spuštění na složce obsahující `camera_cal/` projektu již tyto kalibrační soubory nepočítá („Nalezeno 96 snímků“ u sady 90 snímků) a projekt, který je již přijal, je vyřadí.
> - **`lattice multi-test` vychází ze stavu zapnutí kamer.** Pro spuštění zapne PTP kamer, počká, až se jejich hodiny synchronizují a ustálí před prvním spuštěním – každá kamera se synchronizuje po dobu 120 s, což je asi 2 minuty od zapnutí (`--ptp-settle SECONDS` to nastavuje), protože právě synchronizované hodiny mají odchylku až ~200 us po dobu asi 2 minut — a vrátí PTP každé kamery do stavu, v jakém ho našel. Na kameru, která je stále synchronizovaná z předchozího testu, se již znovu nečeká. Dříve nechával PTP vypnutý v síti s jedním přepínačem, kde žádný DAQ-E neodpovídal, a četl volně běžící hodiny každé kamery jako hodiny PTP, které se nesynchronizovaly (`PTP is not locked`, při každém běhu). Viz [Více kamer (přechodné relace)](#multi-camera-transient-sessions).
> - **Záznam `.daq` udává, zda byl kryt vybrán, nebo předpokládán.** DAQ-U, DAQ-M a DAQ-E se dodávají bez krytky a Chloros nedokáže rozpoznat, zda je krytka nasazena, takže senzor připojený bez pojmenované krytky dostane ve výchozím nastavení hodnotu `sunshine_cosine`, což je *předpoklad*. Každý záznam Chlorosu (schéma 1.30) nyní ukládá `cap_id_source` vedle `cap_id`: `operator`, pokud jste krytku pojmenovali (včetně `none`), a `auto_default`, pokud šlo o výchozí nastavení. Při zpracování záznamu, u kterého byl krytka předpokládána, a přičemž nastavení krytky DAQ v projektu zůstalo na Auto, vygeneruje hlášení `[daq-dls] WARNING` s názvem souboru: holý DAQ-U bez korekce slunečního svitu vykazuje asi 30x vyšší hodnoty. **Při připojení pojmenujte použitý limit** nebo nastavte limit DAQ projektu a varování zmizí.
> - **V systému Windows se již nezobrazují okna konzole pip.** Na počítači s nainstalovaným Pythonem spustil backend a každý z jeho obrazových pracovníků při spuštění jednou kód `pip`, a každý z nich otevřel okno konzole. Nyní se to již neděje.
> - **Živé záznamy `.daq` přežijí výpadek proudu.**Záznam se zapisuje s deníkem a při každém potvrzení dochází k úplné synchronizaci disku, a to za senzorem, takže pomalý disk zpomaluje pouze soubor, nikdy však naměřené hodnoty. Záznam zastavený uprostřed potvrzení (výpadek proudu, ukončení procesu) může vedle sebe zanechat soubor `<name>.daq-journal`:**zkopírujte oba soubory dohromady.**Chloros při prvním načtení souboru vrátí deník zpět, takže se záznam otevře se všemi daty zapsanými před zastavením (ztratí se přibližně poslední 3 s); ve verzi 1.2.0 mohl výpadek napájení ve špatný okamžik způsobit, že celý soubor byl nečitelný („database disk image is malformed“).
> - **Záznam na vytíženém disku již nezpůsobuje zablokování senzoru.** `daq pool-record` zařazuje odečty do fronty a zapisuje je ve vlastním vlákně; odečet, který nebylo možné zapsat, se započítá (`daq pool-list` zobrazuje `N write error(s)` s poslední chybou) místo toho, aby došlo k zamrznutí senzoru (jeden zápis trval 8,4 s při současném ukládání souborů TIFF ze čtyř kamer).
> - **`lattice multi-capture` bez pole již neztrácí snímky, které uložil, když dojde k vypršení časového limitu spouště.** Vypršení časového limitu spouště kamery (`SC_ERR_TIMEOUT -1011`, pozorováno u čtyř kamer na jednom 1 GbE přepínači) ukončí běh u dané skupiny snímků: již uložené skupiny jsou uvedeny v seznamu s kódem `Saved:`, poté následuje chyba a ukončení `1`. Ve verzi 1.2.0 se jednalo o chybu HTTP500 bez seznamu, ačkoli soubory byly na disku.
> - **Kamery spouštěné softwarově již nevynechávají každý druhý spouštěcí signál.** Kamera v režimu softwarového spouštění nyní zařadí do fronty spouštěcí signál, který dorazí během jejího čtení (nebo Chloros počká, až bude připravena), stejně jako to již dělaly kamery synchronizované kabelem.
> - **`daq pool-discover` pojmenovává DAQ-E-S `DAQ-E-S`** ve sloupci `MODEL` (a `--json` ve sloupci `model`). U každého ethernetového senzoru bylo uvedeno `DAQ-E`; ten, který neodpoví na ověření identity do přibližně 2 s, je stále uveden jako `DAQ-E`.
> - **Vylepšené indexové produkty LATTICE** (NDVI a další indexy) od `process`, ve všech režimech zpracování.
> - **Kamery OCN: vylepšené produkty radiance, odrazivosti a indexů při slunečním světle.** Po oddělení pásem se aplikuje korekce flotily; kamery RGN zůstávají beze změny a hodnoty OCN se liší od verzí 1.2.0. To platí i pro snímky OCN pořízené dřívější verzí, pokud jsou zpracovávány na počítači, který obsahuje kalibraci použitou při pořízení snímku (program Chloros si uchovává kalibraci kamery po jejím připojení).
> - **`lattice continuous` zaznamenává surový snímek.** Běh programu `tiff` zapsal demosaikované snímky ve 3snímky, od nichž nemůže `process` vycházet. Jeho snímky jsou nyní souborem `raw`, který zapíše záznam pole: jedna mozaiková rovina s hodnotami samotného snímače, nekomprimovaná, s kalibračním paketem. Skript, který z těchto souborů čte tři roviny, přečte pouze jednu.
>
> - **Každý záznam ukládá hodnotu DAQ, kterou použil, stejně jako to dělá grafické uživatelské rozhraní.** `lattice capture`, `lattice continuous` (jeden soubor `.daq` na každý snímek), `array-capture` a `multi-capture` s připojeným polem (a SDK) zapisují soubory `.daq` a jeho `.match.json`, kdykoli je DAQ navázán nebo zařazen do fondu, včetně snímání pouze v surovém formátu; ve verzi 1.2.0 se při uložení surového nebo debayered snímku zapisuje pouze `--force-daq`. `--no-daq` tuto funkci vypíná, `--daq-tolerance-ms MS` nastavuje toleranci shody pro daný snímek a nový `daq pool-bind` váže kameru nebo pole k DAQ z CLI. Snímání uložené pomocí `.daq` je v `process` přiřazeno pouze k kamerám, které jsou v něm jmenovitě uvedeny. Záznam nyní čeká na další odečet z DAQ, aby mohl sloučit odečty z obou stran expozice. Viz [Čtení DAQ, které záznam ukládá](#the-daq-reading-a-capture-saves) a [Čtení DAQ daleko od expozice](#a-daq-reading-far-from-the-exposure).
> - **`process --indices NAME`** přidává do projektu index pro pásma každé kamery, včetně nového projektu. Ve verzi 1.2.0 byl název, který projekt dosud neuváděl, ignorován pomocí `Unknown index name(s) ignored`.
> - **`process` vedle živého pole:**již nespouští znovu backend, který obsahuje kamery nebo DAQ, a čeká až 15 s na dokončení běhu, který se chýlí ke konci.**Ctrl+C** zastaví běh v backendu, nejen v CLI. Viz [Poznámky k `process`](#notes).
> - **Složka projektu, která je příliš hluboko v adresáři a překračuje limit 260 znaků pro cestu ve Windows**, je rovnou odmítnuta s použitím nejdelší složky, která se vejde, namísto spuštění, které nic nezapíše. Instalátor pro Windows nyní nabízí možnost „Povolit dlouhé cesty ve Windows“ a vyloučení backendu z Microsoft Defenderu; obě možnosti jsou ve výchozím nastavení zaškrtnuté. Viz [Windows (.exe)](#windows-exe).
> - **`lattice array-capture --continuous`** spouští vlastní nepřetržité zachycování backendu: ukazatel průběhu každé 2 s, souhrn namísto výpisu jednotlivých souborů a ukončení s nenulovým kódem v případě jakéhokoli nezapsaného souboru nebo zaplněného disku. Viz [režimy `array-capture`](#array-capture-modes).
> - **První `array-capture` po `array-connect`** čeká až 20 s na první skupinu snímků pole namísto toho, aby vrátil chybu HTTP 503 „Pole se stále zahřívá“.
> - **Expozice pole v oblasti zájmu (ROI):** malá oblast zájmu již neomezuje expozici člena pole na dobu volného běhu snímače (4,9 ms při rozlišení 768 × 304). Viz [Expozice na členu pole](#exposure-on-an-array-member).
> - **Automatická expozice již nesnižuje automatickou snímkovou frekvenci pole.** Pole synchronizované kabelem běží frekvencí, kterou umožňuje síť a čtení z kamer; automatická expozice vyplní dobu, kterou tato frekvence poskytuje, a poté zvýší zisk. Ruční expozice delší než tato doba (`lattice exposure --set`, `configure --set`) zpomalí spouštění, aby se jí přizpůsobilo, místo aby byla odmítnuta, a snímková frekvence se obnoví, jakmile se člen vrátí do automatického režimu. Viz [Expozice na členu pole](#exposure-on-an-array-member).
> - **Pole poskytují méně neúplných skupin snímků** a méně chyb 503 („Žádná nová synchronizovaná skupina snímků“): spouštěcí vstup každé podřízené kamery je nyní zbaven rušivých signálů.
> - **Balíkový 12bitový přenos** (`--packed`, výchozí `auto`) zvyšuje snímkovou frekvenci pole s plným snímačem přibližně o třetinu. Nyní je použit režim `array-connect --packet-size` (verze 1.2.0 jej přijala, ale ignorovala). Viz [Komprimovaný 12bitový přenos](#packed-12-bit-transport---packed).
> - **Pole, jehož kamery se po vypnutí a zapnutí napájení samy obnoví.** Po vypnutí a zapnutí napájení přepínače se pole `array-connect`-otevře kamery, jakmile odpoví, a obnoví streamování s opět zapnutým PTP; ve verzi 1.2.0 se proces trvale zastavil a bylo zapotřebí nového `array-connect`. `array-status` hlásí `ptp_enabled: False` (a `ptp_off:`), zatímco kamera čte PTP jako vypnuté, zatímco dříve hlásilo nastavení, se kterým bylo pole připojeno. Viz [Trvalá připojení](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-record` zaznamenává data, aniž by bylo otevřeno grafické uživatelské rozhraní.** Je-li kombinovaný náhled v grafickém uživatelském rozhraní zavřený, backend vykresluje náhled sám:NDVI pole nebo index, který předáte s novým `--expr`. Záznam, který nezapisuje žádný snímek, nyní selže (`no-frames: …`, ukončení `1`). Dříve se nahrávalo pouze tehdy, když bylo otevřeno kombinované zobrazení grafického uživatelského rozhraní, a záznam, který nic nezaznamenal, přesto ukončil program `0`. Viz [`array-record`](#array-record--combined-index-videogif-monitoring-grade).
> - **Kamera, kterou již Chloros drží, se neotevře podruhé.** Příkaz, který by ji sám otevřel, místo toho vrátí odpověď HTTP 409 `CAMERA_HELD`, přičemž uvede, co ji drží, a příkaz, který ji uvolní. To se týká i kamery, kterou právě v tuto chvíli používá jiný příkaz (např. `multi-test` nebo `multi-capture` otevírající své kamery, `lattice capture`): druhý příkaz je odmítnut, dokud první nedokončí svou činnost. Jeho opětovné otevření by mohlo zastavit stream držitele nebo způsobit pád backendu. Viz [Trvalá připojení](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **Přechodné příkazy pro více kamer doručí všechny kamery nebo selžou.** `lattice multi-capture` bez připojeného pole, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream` a `index --live` ukončí činnost; `1` uvede název kamery, která nedodala žádný snímek. Datové toky přeskočí skupinu snímků, ve které chybí kamera, a zastaví se po třech po sobě jdoucích; `multi-test` vypíše jeden řádek za každé spuštění, `(no frame)` tam, kde kamera snímek vynechala; `align-stream` také ukončí `1` při chybě úlohy. Každý běh poté vrátí každou kameru do stavu, v jakém ji našel: její spouštěcí a synchronizační zapojení, její PTP (`PtpEnable`) a všechna nastavení kamery, která běh změnil — expozici a zesílení, cíl AE, limity a tlumení, vyvážení bílé, úroveň černé, formát pixelů, binning a ROI, převrácení, snímkovou frekvenci — pokud to nezakáže `--keep-settings` / `--clear-settings` (viz [Paměť kamery](#camera-memory----keep-settings----clear-settings)). Dříve tyto příkazy uchovávaly všechny přijaté snímky a ukončily `0` (`align-stream` vypsal chybu úlohy jako `Note:`), a nechávaly všechny kamery v režimu `TriggerMode` `On`. Také nastavovaly expozici každé kamery na základě scény před uložení snímku (maximálně několik sekund), přičemž na cílovém připojeném poli se každá kamera spouští s nastavením: 40 multispektrálních, 60 RGB (nastavuje to `--brightness N`) a udržuje je po celou dobu běhu; stejně tak i v případě jediného příkazu `lattice capture` pro kameru, kterou Chloros neřídí. Dříve byl první snímek každé kamery pořízen při vlastním expozičním limitu kamery, asi 100 ms, bez ohledu na světelné podmínky, a často docházelo k oříznutí. Viz [Vícekamerový režim (dočasné relace)](#multi-camera-transient-sessions).
> - **Dočasné příkazy pro více kamer sdílejí síť stejným způsobem jako `array-connect` a otevírají pouze `--serials`.** Každá kamera je přizpůsobena svému podílu na hostitelském spojení, takže čtyři kamery běží na 1 Gbps spojení; dříve si tyto čtyři kamery vyžádaly dohromady 125 MB/s, což představovalo celou šířku pásma 1 GbE, a každý snímek ztrácel pakety. Pole, které síť nedokáže přenést, je odmítnuto ještě předtím, než začne jakákoli kamera streamovat. `--serials` nyní vybírá kamery, přičemž první z nich spustí spouštěč; dříve každý příkaz otevřel všechny kamery v síti bez ohledu na to, co určoval `--serials`. Porucha spočívající ve ztrátě paketů GVSP je nyní takto označena, místo aby byla přičítána synchronizačnímu kabelu M8. Viz [Více kamer (přechodné relace)](#multi-camera-transient-sessions).
> - **`lattice array-disconnect` nechává kamery volně běžet, tak jak byly nalezeny.** `TriggerMode`, `Off` a vše ostatní tak, jak to připojení zjistilo — nastavení spouště a synchronizační linky (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`), PTP a nastavení kamery, která se během relace změnila – takže jedinýpříkaz pro kameru u bývalého člena získá snímky najednou a master již nezůstává na `TriggerSource` `Software`. `--keep-settings` uloží nastavení relace do paměti každé kamery, `--clear-settings` tuto paměť vynuluje (viz [Paměť kamery](#camera-memory----keep-settings----clear-settings)). `lattice continuous` také nastaví kameru, kterou najde v zapnutém režimu, na volný provoz pro daný běh (a po skončení běhu ji vrátí zpět), a běh bez snímku ukončí `1`.
> - **`lattice configure --get` / `lattice exposure`** vypisují `note:` sskutečnou expozicí nebo ziskem, zatímco běží její automatický režim. Odmítnutý příkaz `lattice exposure --set` / `configure --set` udává rozsah, který kamera akceptuje, a místo vypnutí automatické expozice vrátí automatický režim zpět. Viz [Nastavení snímače](#sensor-tuning).
> - **Nová ovládací prvky kamery:**`lattice highlights` (ztlumí jasné bílé plochy v zobrazení RGB) a `lattice exposure --brightness-auto`.**`lattice multi-detect`** vyhledá kabelové kamery během jednoho cyklu sondování a jako výchozí hlavní kameru nastaví kameru RGB.
> - **Nové příkazy DAQ:** `daq pool-bind`; `pool-enable-ptp`; a pro vlastní nastavení zařízení DAQ-E `pool-info`, `pool-set-network`, `pool-set-auto-stream`, `pool-reboot` a `pool-update-firmware`. Viz [Nastavení zařízení DAQ-E](#daq-e-device-settings).
> - **ZAŘÍZENÍ DAQ-E, které přestane přenášet data, je znovu připojeno backendem.** Když proud dat ze zařízení DAQ-E zařazeného do skupiny utichne (v důsledku vypnutí a zapnutí napájecího přepínače, přerušení spojení s kamerou), backend jej znovu připojí s původním nastavením a během doby, kdy není k dispozici, opakuje pokusy v prodlužujících se intervalech. Záznam na tomto zařízení pokračuje ve stejném souboru `.daq` s mezerou označenou v souboru, a soubor `pool-record --stop` uvádí jednotlivé mezery. Ve verzi 1.2.0 zůstával snímač neaktivní až do souborů `pool-disconnect` + `pool-connect`, což také ukončilo záznam. Viz [DAQ-E, který přestane dodávat data](#a-daq-e-that-stops-delivering).
> - **`time-sync status`** hlásí, zda pakety PTP grandmastera dorazí do sítě (`health`), a pokud ne, vypíše na stderr zprávu `WARNING:`. Viz [`time-sync`](#chloros-cli-time-sync).
> - **Balíček .deb pro Linux:** zvětšuje velikost přijímacích bufferů soketů jádra, které potřebují datové toky z kamer (při výchozím nastavení jádra dochází ke ztrátám snímků a poklesu snímkové frekvence), přidává backendovou službu, která běží pod vaším uživatelským účtem, a `lattice network` kontroluje stav těchto bufferů. CLI nikdy nezastaví backend, na kterém služba běží, a při aktualizaci se povolené backendové služby spustí znovu. Viz [Spuštění backendu jako služby](#running-the-backend-as-a-service-headless) a [Linuxové hostitelské systémy — přijímací buffery soketů](#linux-hosts--socket-receive-buffers).
> - **`lattice array-capture` uvádí, kdy se nepodařilo provést zarovnání:**každý důvod, proč byl záznam uložen nezarovnaný (chybějící profil zarovnání, členy s odlišným rozlišením, pole nesynchronizované s kabelem, rámec, na který nelze profil namapovat) se zobrazí jako řádek `Alignment:`.**`chloros-cli daq help`** je `daq --help` (ukončilo se to jako `2`) a v systému Linuxu jsou v nápovědě uvedeny příkazy pro přímé ovládání hardwaru `daq` , které daná kompilace obsahuje.
> - **Jetson:** balíček je `chloros_1.3.0_arm64_jp6.deb` pro JetPack 6 a jeho zpracování na GPU probíhá na Orinu pomocí CUDA PyTorch od JetPacku. Viz [Linux arm64 — Jetson](#linux-arm64--jetson-jetpack-6).

---

## Konvence

- Všechny příkazy mají předponu `chloros-cli`. Ve Windowsu je binární soubor `chloros-cli.exe`; v Linuxu/Jetsonu je to `chloros-cli`.
- Volitelné argumenty jsou zobrazeny jako `--flag`. Povinné poziční argumenty jsou zobrazeny bez závorek.
- Je-li uvedena výchozí hodnota, vynecháním příznaku se použije tato hodnota.
- CLI je jednoduchý HTTP klient využívající backend Chloros (server na adrese `127.0.0.1:5000`, který používá i desktopová aplikace). **Které příkazy jej spouštějí:**
  - Základní příkazy — `process`, `login`, `logout`, `status`, `export-status`, `time-sync` a `selftest` — využívají již spuštěný backend, nebo jej spustí, pokud žádný neběží, a po dokončení ho opět zastaví.
  - **Příkazy `lattice`, `project` a `daq` nikdy backend nespouštějí.** Vyžadují backend, který již běží: backend aplikace pro Windows, Linuxovou [backendovou službu](#running-the-backend-as-a-service-headless) nebo backend, který spustíte sami (viz [Spuštění backendu](#start-the-backend)). Bez něj vypisují kód chyby `Error: Chloros backend not reachable at http://127.0.0.1:5000` spolu s příkazem, který jej spouští, a ukončí se s kódem chyby `1`.
- `CHLOROS_BACKEND_URL=<url>` nasměruje rodiny příkazů **`lattice`**,**`project`**a**`daq pool-*`** na jiný backend (`lattice network-analysis` místo toho směřuje na `--backend-url`). Základní příkazy záměrně fixují `http://127.0.0.1:<port>` a ignorují jej (literál IPv4 zabraňuje penalizaci systému Windows ve výši přibližně 2 s na jeden požadavek při přeměně `localhost`→`::1`). Viz [Proměnné prostředí](#environment-variables).
- **Relativní cesty se vyhodnocují vzhledem ke složce, ze které spouštíte `chloros-cli`.** Backend je proces, který otevírá výstupní a vstupní složky, takže před odesláním cesty ji CLI převede na absolutní: `-o` / `--output` (včetně jeho výchozí hodnoty, `output`), `--input`, `--burst-dir`, `--profile`, `--save` a `--mask`, a krok [receptu](#recipe-dsl) `output` (výchozí `output`)a výchozí složka `output` pro `lattice index --live`. `-o captures/` proto zapisuje do `<current folder>/captures/` a vypsané cesty jsou absolutní. `~` je rozbalen. V případě vzdáleného backendu (`CHLOROS_BACKEND_URL` označující jiný počítač) jsou cesty odeslány tak, jak byly zadány, a jsou vyřešeny na daném počítači.
- Pro všechna volání SDK/CLI je vyžadováno přihlášení k účtu Chloros+ volání je vyžadováno přihlášení k účtu Chloros+ (spusťte `chloros-cli login` jednou na každém počítači; uloží se do mezipaměti v `~/.chloros/`).
- Příklady používají cesty v systému Linux; v systému Windows nahraďte `/home/user/...` za `C:/Users/.../...`. Sériový port zařízení DAQ-U'má ve stylu `COM3` ve Windowsu a `/dev/ttyUSB0` / `/dev/ttyACM0` v Linuxu.
- Přehledy zapisují vzájemně se vylučující příznaky jako `{--a | --b}` a volitelné příznaky jako `[...]`.

---

## Přehled nejvyšší úrovně

```
chloros-cli [global options] COMMAND [command options]
```

### Globální volby

| Příznak | Popis |
| --- | --- |
| `--backend-exe PATH` | Přepsat automaticky detekovaný spustitelný soubor backendu. |
| `--port N` | Port HTTPu backendu (výchozí: `5000`). Používá se pouze pro základní příkazy (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`); `lattice`, `project` a `daq` používají `CHLOROS_BACKEND_URL` (výchozí `http://127.0.0.1:5000`). |
| `-v, --verbose` | Zapnout podrobný výstup. |
| `--restart` | Vynutí restart backendu (ukončí všechny spuštěné instance `backend_server.py`). Backend spouštěný službou systemd (Linux .deb) se nikdy nezastaví: příkaz místo toho vypíše `sudo systemctl restart <unit>` a ukončí se `1`. |
| `--version` | Zobrazí verzi (`Chloros CLI 1.3.0`). |
| `--help` | Zobrazí nápovědu nejvyšší úrovně. |

### Rejstřík příkazů

| Příkaz | Účel |
| --- | --- |
| [`process`](#chloros-cli-process) | Zpracovat složku záznamů z aplikací Survey3 nebo LATTICE od začátku do konce. |
| [`login`](#chloros-cli-login) | Ověřit tento počítač pomocí účtu Chloros+. |
| [`logout`](#chloros-cli-logout) | Vymazat přihlašovací údaje z mezipaměti. |
| [`status`](#chloros-cli-status) | Zobrazit aktuální stav licence / ověření. |
| [`export-status`](#chloros-cli-export-status) | Průběh exportu Live Thread-4 během spuštění `process`. |
| [`language`](#chloros-cli-language) | Nastavení nebo zobrazení seznamu jazyků pro zobrazení CLI (podporováno 38 jazyků). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Výchozí složka projektu (sdílená s grafickým rozhraním). |
| [`update`](#chloros-cli-update) | Zkontrolovat a nainstalovat aktualizace Chlorosu (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Diagnostika systému + zkušební testy. |
| [`time-sync`](#chloros-cli-time-sync) | Stav a ovládání PTP grandmasteru. |
| [`lattice`](#chloros-cli-lattice) | Ovládání a snímání kamery LATTICE (více než 45 podpříkazů). |
| [`daq`](#chloros-cli-daq) | Ovládání spektrálních senzorů DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Otevření a spuštění uloženého projektu Chloros (kamery + DAQ). |

---

## Instalace

`chloros-cli` je součástí instalačního balíčku Chloros pro všechny podporované platformy — samostatný balíček pro příkazový řádek (CLI) není k dispozici ke stažení. Instalací balíčku pro danou platformu se `chloros-cli` přidá do vašeho `PATH` společně s desktopovou aplikací a binárním souborem backendu, který ovládá.

Nejnovější soubory ke stažení: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Instalační program také obsahuje praktické spouštěcí skripty (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`), které otevírají připravený příkazový řádek CLI.

### Windows (.exe)

1. Stáhněte si instalační program pro Windows ze stránky pro stahování.
2. Spusťte soubor `Chloros Installer 1.3.0.exe` a postupujte podle pokynů průvodce. Výchozí instalační cesta je `C:\Program Files\Chloros\` (CLI se nachází v `C:\Program Files\Chloros\cli\`, což instalační program přidá do proměnné PATH).
3. Otevřete nový terminál (`cmd.exe`, PowerShell nebo Terminál Windows), aby se načítal aktualizovaný soubor `PATH`.

```powershell
chloros-cli --version
```

Instalační program automaticky přidá `chloros-cli.exe` do vašeho systémového `PATH` a přibalí ovladače kamer LATTICE. Backendem je `C:\Program Files\Chloros\backend\chloros-backend.exe` (složka `backend` ve vaší instalační složce), který není součástí `PATH`.

**Nastavení systému.** Stránka *Vybrat další úkoly* v instalačním programu obsahuje pod položkou **Nastavení systému**dvě možnosti, přičemž obě jsou**ve výchozím nastavení zaškrtnuté**:

| Možnost (jak je uvedeno v instalačním programu) | Co dělá | Odmítnutí tiché instalace |
| --- | --- | --- |
| **Povolit dlouhé cesty v systému Windows** (umožňuje, aby složky projektů měly více než 260 znaků; doporučeno) | Nastaví systémové nastavení `LongPathsEnabled` v systému Windows na hodnotu 1. Systém Windows je dodáván s touto volbou vypnutou, a žádná cesta pak nesmí překročit 260 znaků; produkty Chloros se nacházejí až asi 130 znaků pod projektovou složkou, takže hlubší projektová složka je odmítnuta (viz [Řešení problémů](#troubleshooting-pointers)). Nastavení se projeví u programů spuštěných po instalaci. Při odinstalaci se ponechá zapnuté, protože na něm mohou záviset jiné programy. | `/MERGETASKS="!longpaths"` |
| **Vyloučit backend Chlorosu ze skenování Microsoft Defenderem**(mnohem rychlejší snímání z kamery; doporučeno) | Přidá vyloučení**procesu** Defenderu pro `chloros-backend.exe` ve složce instalace, takže ochrana v reálném čase neskenuje každý soubor, který backend zapíše, ať už jej zapíše kamkoli. Žádný jiný program není vyloučen; vyloučení se při odinstalaci odstraní. Pokud Defender není nainstalován nebo je zablokován zásadami, tento krok tiše selže a instalace pokračuje. | `/MERGETASKS="!defenderexclusion"` |**Proč je volba Defender důležitá:** při vysokých rychlostech snímání představuje skenování zachycených souborů v reálném čase asi 30 % rychlosti snímání (měřeno při nepřetržitém snímání 9 kamer: 77,7 snímků/s bez Defenderu, 52,7 snímků/s se skenováním). `lattice array-capture --continuous` zobrazí upozornění, pokud zjistí, že Defender při snímání využívá jedno nebo více jader procesoru; spusťte instalační program znovu s zaškrtnutou touto volbou.

Tichá instalace (`/SILENT` nebo `/VERYSILENT`) použije obě možnosti, pokud nejsou výslovně deaktivovány, např. `"Chloros Installer 1.3.0.exe" /VERYSILENT /MERGETASKS="!defenderexclusion"`.

**Brána firewall systému Windows.** Instalační program vždy přidá dvě pravidla pro příchozí provoz pro `chloros-backend.exe` do každého síťového profilu a odinstalační program je odstraní: `Chloros Backend (UDP-In)` pro datové toky z kamer, PTP, mDNS a datový proud DAQ-E, a `Chloros Backend (TCP-In)`, omezené na místní podsíť, pro aktualizaci firmwaru DAQ-E, při které se senzor připojí zpět k tomuto počítači za účelem získání snímku (viz [Nastavení zařízení DAQ-E](#daq-e-device-settings)). HTTP API backendu zůstává navázáno na `127.0.0.1`.

### Linux amd64 (.deb)

Pro Ubuntu 22.04 LTS nebo novější / pracovní stanice x86_64 založené na Debianu.

> **Ubuntu 20.04 není podporováno.** Seznam závislostí balíčku je odvozen od toho,
> na co se backend skutečně odkazuje, a to zahrnuje `libc6 (>= 2.35)`;
> focal dodává glibc 2.31. `apt` instalaci odmítne, místo aby ji nechal selhat za
> běhu. Seznam také uvádí `libgl1`, který načítá přibalené OpenCV a v minimálním
> obrazu serveru může chybět; `apt install` jej dodává.

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
```

Soubor .deb nainstaluje:
- `chloros-cli` až `/usr/bin/chloros-cli` a `chloros-backend` (spouští backend) do `/usr/bin/chloros-backend`
- Zkompilovaný backend a CLI, rozbalené, v `/usr/lib/chloros/backend/` a `/usr/lib/chloros/cli/`
- Ovladače kamer LATTICE
- Modely odšumovače, kalibrační balíčky a konfigurace aktualizačního kanálu
- Pythonové SDK pro systém `python3` (`import chloros_sdk`). `apt install` s sebou přináší také své požadavky, `python3-numpy` a `python3-requests` (doporučeno, není nutné: výše uvedené programy jsou samostatné; `dpkg -i` je přeskočí a výstup instalace uvede, co chybí). Ve virtuálním prostředí: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`
– `/etc/sysctl.d/60-chloros-gige.conf`, které zvětší velikost přijímacích bufferů jádra pro datové toky z kamery proudů na 32 MB a nastaví je najednou (již nastavená vyšší hodnota zůstane beze změny). Odinstalování balíčku odstraní tento soubor. Viz [Linuxové hostitelské systémy — přijímací buffery soketů](#linux-hosts--socket-receive-buffers).
- Dvě volitelné služby systemd pro backend, ani jedna z nich není povolena (další část)

#### Spuštění backendu jako služby (bez grafického rozhraní)

Bez služby běží backend (a s ním i PTP grandmaster) pouze tehdy, když jej něco spustí: `chloros-backend` v terminálu, skriptu SDK nebo příkazu core `chloros-cli` po dobu jeho běhu. Chcete-li mít backend neustále spuštěný na stroji bez grafického rozhraní, povolte službu **jako váš uživatel**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` spustí backend jako tento uživatel. Záznamy, které zapisuje, patří tomuto uživateli, jeho podrobný protokol se nachází v jeho `~/.cache/chloros/logs`, a používá jeho `chloros-cli login` (`~/.chloros`), takže se nejprve jednou přihlaste jako tento uživatel. PTP nepotřebuje oprávnění root. Stejně jako každý nainstalovaný backend naslouchá pouze na portu `127.0.0.1:5000`.

`chloros-backend.service` spouští backend jako **root**. Je stále nainstalován, aby počítač, na kterém je povolen, zůstal v chodu, ale vše, co root backend zapíše, patří uživateli root, takže uživatel nemůže smazat své vlastní záznamy. Chcete-li přepnout na svého uživatele:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

Podrobný protokol služby root se nachází v souboru `/var/log/chloros`, který je čitelný pro členy skupiny `adm` (služba nastavuje proměnnou `CHLOROS_LOG_DIR`; viz [Proměnné prostředí](#environment-variables)). Při instalaci balíčku se zobrazí příkaz `enable` s vyplněným vaším uživatelským jménem a také přepínač, pokud je služba root povolena.

**Povolení jedné backendové jednotky.** Současně běží pouze jedna backendová jednotka. Druhá povolená jednotka (`chloros-backend.service` vedle `chloros-backend@<user>.service` nebo dvě uživatelské jednotky `@`) se odmítne spustit a zůstane jako `failed` s výstupním stavem 98; systemd ji nerestartuje. Jeho `systemctl status` uvádí důvod: `Another instance is already running. Exiting.`, poté to, co zadržuje backend (port 5000, nebo zámkový soubor a uživatel, ke kterému patří), a poté `systemd will not restart <unit> (exit status 98)`. Po restartu se může spustit kterákoli z jednotek. Chcete-li přepnout, deaktivujte tu, kterou nepoužíváte, jak je uvedeno výše.**CLI nikdy nezastaví backend služby.** `chloros-cli` jej používá tak, jak je, stejně jako GUI a SDK: `process` na něm běží a načítá svůj vlastní projekt namísto toho posledního. Tam, kde by CLI jinak restartovalo backend (`--restart` nebo `process`, jehož import nic nenašel), pojmenuje jednotku, vypíše `sudo systemctl restart <unit>` a ukončí `1`, protože backend zastavený tímto způsobem by zůstal nefunkční. `GET /api/status` pojmenuje jednotku jako `service` (`null` v případě backendu, na kterém neběží žádná služba).**Aktualizace a odstranění.** Aktualizace balíčku zastaví všechny služby backendu a poté znovu spustí každou z nich, která je povolena, přičemž jako první se spustí jednotky `@<user>` (jednotka spuštěná ručně, ale nikdy nepovolena, zůstane zastavená, stejně jako po restartu). Je-li povoleno více než jedna služba, běží pouze jedna a ostatní zůstávají v nefunkčním stavu, jak je uvedeno výše; při aktualizaci se zobrazí kódy `Chloros: N backend units are enabled: …` a `Keep one enabled: sudo systemctl disable --now <unit>`. Odinstalace je zastaví a deaktivuje. Jeho úplné odstranění smaže také `/var/log/chloros`.

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Stejná struktura jako u balíčku .deb pro amd64, včetně přijímacích bufferů a [backendových služeb](#running-the-backend-as-a-service-headless), pro Jetson Orin / Orin NX / Orin Nano na **JetPack 6** (L4T R36). Jeho PyTorch je CUDA sestavení JetPack 6 pro Orin, takže zpracování na GPU, včetně debayeringu s podporou textur, běží na GPU Jetsonu. Balíček neobsahuje samotnou knihovnu CUDA: využívá knihovny CUDA a cuDNN, které nainstaluje JetPack 6 (`sudo apt install nvidia-jetpack` na obrazu, kde tyto knihovny chybí). Pokud není k dispozici žádný grafický procesor s podporou CUDA, zpracování probíhá na procesoru a funkce s podporou textur používá standardní debayer (viz [`process`](#chloros-cli-process)).

### Jednorázové ověření na každém zařízení

Každá platforma vyžaduje jednorázové přihlášení do Chloros+, než začnou fungovat volání SDK a CLI:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Přihlašovací údaje se ukládají do mezipaměti v souboru `~/.chloros/user_session.json`.

### Spuštění backendu

Příkazy `lattice`, `project` a `daq` komunikují s již spuštěným backendem; nikdy jej nespouštějí (viz [Konvence](#conventions)). Během jejich používání nechte jeden z nich spuštěný:

| Platforma | Jak |
| --- | --- |
| Windows | Otevřete desktopovou aplikaci Chloros (její backend obsluhuje i CLI) nebo spusťte příkaz `"C:\Program Files\Chloros\backend\chloros-backend.exe"` v samostatném terminálu (ve složce `backend` ve vaší instalační složce) a nechte jej otevřený. |
| Linux / Jetson | Spusťte `chloros-backend` v samostatném terminálu, nebo nechte jeden backend neustále běžet pomocí [backendové služby](#running-the-backend-as-a-service-headless). |

Současně běží pouze jeden backend; všichni klienti (GUI, CLI, skripty SDK) jej sdílejí. Základní příkazy (`process`, `login`, `status`, …) jej také používají a spustí nový, pokud žádný neběží.

**Backend běží nepřetržitě mezi jednotlivými příkazy**, takže další příkaz již neplatí za jeho spuštění znovu. Zastaví se, když se odhlásíte (`chloros-cli logout`, pokud není otevřena aplikace Chloros nebo neprobíhá běh či offline sestavení), když spustíte `chloros-cli backend stop`, nebo se zastaví sám po 30 minutách, pokud jej nic nevyužívá: žádný příkaz, skript ani aplikace, žádný běh ani offline sestavení, žádná připojená kamera ani světelný senzor. `backend stop` odmítne spuštění, pokud právě probíhá běh nebo offline sestavení, a uvede důvod; `--force` jej okamžitě zastaví a daný běh nebo sestavení se ztratí (série snímků se zachová a `array-build-video` ji sestaví znovu). Nastavte `CHLOROS_BACKEND_IDLE_MIN` před spuštěním backendu, abyste změnili 30 minut (`0`: nikdy); [služba backendu](#running-the-backend-as-a-service-headless) se nikdy sama nezastaví. `chloros-cli backend status` ukazuje, zda je některá spuštěná, co ji využívá a zda běží nějaká offline sestava. `process` otevře svůj projekt ve spuštěném backendu, pokud není otevřená aplikace Chloros s projektem, který otevřela: v takovém případě to odmítne a musíte nejprve zpracovat z aplikace nebo tento projekt zavřít. Projekt, který zůstal otevřený po předchozím spuštění `process` nebo skriptu SDK, je jednoduše nahrazen.

### Ověření instalace

```bash
chloros-cli --version           # prints "Chloros CLI 1.3.0"
chloros-cli selftest            # 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

`selftest` ukončí `0` pouze tehdy, když projde všech sedm kontrol. Poslední z nich vyžaduje GPU s podporou CUDA (GPU NVIDIA, nebo Jetson Orin) a modely odšumovače, takže na počítači bez něj se zobrazí kód `Failed: CUDA + Denoiser` a program se ukončí s kódem `1`; vše kromě debayera s podporou textur, který je určen výhradně pro GPU, tam stále funguje.

> **Vyžaduje se předplatné Chloros+.**CLI vyžaduje aktivní tarif Chloros+.**Copper**je základní úroveň Chloros+ — každý placený tarif Chloros+ zahrnuje přístup k CLI/SDK; pouze bezplatná úroveň**Iron** jej nezahrnuje. (Převodník ID tarifů: `0`=Iron/free, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Upgrade na [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Tento limit je vynucován na straně backendu, nikoli pouze prostřednictvím CLI: požadavek s příznakem SDK/CLI bez placeného tarifu je odmítnut s kódem `403 PLAN_UPGRADE_REQUIRED`, ať už pochází z `chloros-cli`, Python SDK nebo ručně vytvořeného HTTP klienta. Odhlášený volající obdrží místo toho kód `401 AUTH_REQUIRED`. Přístup funguje offline po dobu odkladného období daného tarifu (30 dní u měsíčního tarifu, do vypršení platnosti u ročního tarifu) a po jejím uplynutí se zastaví; kód `chloros-cli status` nadále funguje, aby byl důvod viditelný (jedná se o jedinou trasu SDK/CLI, která je osvobozena od omezení podle tarifu — `GET /api/license-status`).

---

## Pětiminutový rychlý start

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI GNDVI OSAVI

# 3. The lattice / daq commands need a running backend: on Windows open the
#    Chloros desktop app; on Linux start one (or see "Start the Backend")
chloros-backend &

# 4. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 5. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 6. Read a spectrum from a connected DAQ-U (Windows: --port COM3)
chloros-cli daq pool-connect --port /dev/ttyUSB0
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

`process` (krok 2) spustí backend pro vlastní běh, pokud žádný neběží, a zastaví jej po dokončení, takže jej spusťte pro krok 4 a dále. Viz [Spuštění backendu](#start-the-backend).

---

## `chloros-cli process`

Zpracujte složku obrázků prostřednictvím kompletního Chlorosova pipeline (detekce cíle → kalibrace → viněta → odrazivost → export indexu).

### Přehled

```
chloros-cli process INPUT [OPTIONS]
```

### Pozicionální argumenty

| Argument | Popis |
| --- | --- |
| `INPUT` | Cesta ke vstupní složce obsahující soubory `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) nebo `.dng`. |

### Obecné volby

| Příznak | Výchozí hodnota | Popis |
| --- | --- | --- |
| `-o, --output PATH` | nová složka s názvem `-n` ve výchozí projektové složce | Projektová složka. Projekt je pojmenován podle ní a složka, která již obsahuje soubor `project.json`, získá soubor `_1` / `_2`, místo aby byla přepsána. Výchozí složkou projektu je ta, kterou vybral `set-project-folder` (nebo grafické rozhraní), jinak `Documents/MAPIR/Chloros Projects`. |
| `-n, --project-name NAME` | auto (časové razítko) | Název projektu, používá se pouze **bez** `-o` (s `-o` projekt převezme název složky). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` používá neuronový debayer Chloros+ pro snímky Survey3; je pomalejší, ale poskytuje vyšší kvalitu. Je určen pouze pro Survey3: Snímky LATTICE vždy používají standardní demosaic, takže u složky obsahující snímky LATTICE `process` po importu odmítne `texture-aware` s chybou uvádějící počet těchto snímků a ukončí `1` ještě před zahájením jakéhokoli zpracování (aplikace Chloros pro ně rovněž nenabízí funkci Texture Aware). Vyžaduje grafický procesor s podporou CUDA (grafický procesor NVIDIA v systémech Windows / Linux x64, grafický procesor Orin na platformě Jetson): bez něj se spustí `standard` a backend zaznamená jeden řádek `[DEBAYER] … needs a CUDA GPU …` s vysvětlením důvodu (zobrazeno s `--verbose`). |
| `--vignette / --no-vignette` | `--vignette` | Korekce vinětace. |
| `--reflectance / --no-reflectance` | `--reflectance` | Kalibrace odrazivosti (používá panelový terč, pokud je nalezen, kalibraci NIST pro jednotlivé sériové čísla pro LATTICE). U multispektrálních dat LATTICE slouží zároveň jako **produkt** odrazivosti — viz [Přepínače exportu podle produktu](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | vypnuto | Použít korekce PPK GNSS ze souborů sidecar. |
| `--exposure-pin-1 CAMERA_MODEL` | žádné | Model kamery připojený k expozičnímu pinu 1, pro časovou synchronizaci PPK. |
| `--exposure-pin-2 CAMERA_MODEL` | none | Model kamery připojený k expozičnímu pinu 2 pro časovou synchronizaci PPK. |
| `--recal-interval SECONDS` | `0` | Minimální interval rekalibrace v sekundách (*Minimální interval rekalibrace* projektu). |
| `--timezone-offset HOURS` | `0` | Časové posunutí světelného senzoru vůči UTC v hodinách (projektové *Časové posunutí světelného senzoru*). |
| `--format FORMAT` | `TIFF (16-bit)` | Jedna z hodnot `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`. Program Chloros nikdy nezvyšuje bitovou hloubku obrázku: u všech 8bitových obrázků se formát s vyšší hloubkou zapisuje jako `TIFF (8-bit)` a v popisu se to uvádí jednou. V projektu kombinujícím 8bitové a 12/16bitové snímky se každý snímek exportuje ve své vlastní hloubce nebo nižší. Radiance je vždy 32bitová s plovoucí desetinnou čárkou. |
| `--indices NAME [NAME ...]` | žádné | Vegetace indexy podle názvu předvolby (`NDVI`, `GNDVI`, `OSAVI`, `SAVI`, `EVI`, `GCI`, …), bez rozlišování velkých a malých písmen. Každý z nich je přidán do projektu pro pásma dané kamery, včetně nového projektu. Název, který není předvolbou, nebo který nelze vytvořit z pásem kamery (např. `NDRE` vyžaduje RE a NIR, které žádná jednotlivá kamera neposkytuje), je uveden v varování a nedostane žádný indexový obrázek. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Vynutí vstupní bod zpracovatelského řetězce pro soubory LATTICE TIFF (soubory Survey3.raw to neovlivní). Také nouzové řešení, které umožňuje zpracování snímků **bez formátu raw** — viz [Jak vypadá složka snímků](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | zapnuto | Vydat lineární produkt po odstranění bayerského filtru (`Debayered_Images`). Viz [Přepínače exportu pro jednotlivé produkty](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | zapnuto | Vydává náhled na displeji (`Preview_Images`): RGB = vyvážení bílé (DAQ-zdroj světla, je-li k dispozici, jinak šedý svět) + gama; multispektrální = roztažení do falešných barev. |
| `--radiance / --no-radiance` | zapnuto | Vysílá radianci typu float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referenční hodnota pro produkt odrazivosti LATTICE: `auto` = absolutní referencí je cíl v rámci snímku, který prošel kontrolou kvality (cíl s příznakem QA soutěží s DAQ o naměřené skóre), záložní řešení DAQ-downwelling (ρ = π·L/E); `target` = přísné (bez nahrazení DAQ); `daq` = rozhodující je DAQ. Viz [Přepínače exportu pro jednotlivé produkty](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | žádné | Adresář **naměřených** skenů cílové odrazivosti pro jednotlivé jednotky (`<serial>.csv`); v případě chybějících dat se použijí nominální spektra T3/T4P. |
| `--array-alignment / --no-array-alignment` | zapnuto | Pole LATTICE: aplikují zarovnání mezi moduly, které je zaznamenáno v souboru XMP každého snímku (`Chloros:Alignment*`), na každý zpracovaný produkt (odstranění matice / náhled / radiance / odrazivost / index). Pro snímky bez těchto značek se neprovádí žádná operace. |
| `--array-alignment-crop / --no-array-alignment-crop` | oříznutí | Ořízne zarovnané výstupy na oblast společného překryvu pole tak, aby všechny moduly sdílely jednu stopu; `--no-…` zachovává plnou plochu snímače (černé vyplnění mimo zdroj). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Převzorkování pro deformaci při zarovnání. `nearest` zachovává přesné zdrojové dynamické rozsahy (DN) (žádné míchání radiometrických hodnot mezi pixely). |

### Možnosti detekce cílů

| Příznak | Popis |
| --- | --- |
| `--min-target-size PIXELS` | Minimální velikost kalibračního cíle v pixelech pro detektor. |
| `--target-clustering 0-100` | Procento seskupování cílů (výchozí hodnota `60`). |
| `--target / --targets` | Hledat kalibrační cíle pouze v podsložce `target` nebo `targets` vstupního souboru (rychlejší). |

### Příklady

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a project folder of your choice (the project is named after it)
chloros-cli process "C:/input" -o "C:/projects/Field_A_2026-05-26"

# Name the project and let it go to the default project folder
chloros-cli process "C:/input" -n "Field_A_2026-05-26"
```

### Přepínače exportu pro jednotlivé produkty (LATTICE multispektrální)

Zpracování LATTICE se rozvětvuje na **všechny příslušné produkty v jednom průchodu**. Čtyři přepínače pro jednotlivé typy — `--debayered`, `--preview`, `--radiance`, `--reflectance` — jsou**ve výchozím nastavení zapnuté**; pomocí formuláře `--no-<type>` můžete jednu z nich vypnout. Hlavní kamery RGB vždy vysílají pouze debayered + náhled (žádná radiance/reflektance pro jednotlivé pásma), proto jsou pro ně přepínače `--radiance`/`--reflectance` bez účinku. Přepínače jsou ignorovány u Survey3 `.raw` (který se řídí standardní cestou odrazivosti/cíle). *(Starý příznak `--radiometric-output {reflectance,radiance,sensor-response}` byl **odstraněn** a nahrazen těmito přepínači; úroveň `sensor-response` již neexistuje.)*

| Produkt | Výstup | Je nutný DAQ downwelling? |
| --- | --- | --- |
| `--debayered` | Lineární demosaikování (`Debayered_Images`): stejný soubor, jaký zachycení uloží na své úrovni `debayered` — tmavý offset (DSNU) kalibračního balíčku z surových dat, poté demosaikace s ohledem na hrany pomocí OpenCV; bez LUT linearity, vyrovnání pole ani oddělení barev. | Ne. |
| `--preview` | Náhled na displeji (`Preview_Images`): RGB = WB + gama; multispektrální = roztažení do falešných barev. | Ne. |
| `--radiance` | float32 W/m²/sr/nm z úplného radiometrického řetězce (`Radiance_Images`). | Č. |
| `--reflectance` | uint16 odrazivost ρ (`32768` = 1,0), kompatibilní s Pix4D. | **Ano**, pokud není ukotvena cílem v rámci snímku, který prošel kontrolou kvality (viz níže). |

`--reflectance-source` vybírá referenci odrazivosti:**`auto`**(výchozí) nastaví cíl v rámci snímku, který prošel kontrolou kvality, jako**absolutní referenci**— empirické řetězce linií ukotvené k cíli jsou vzájemně porovnávány na vyřazených panelech a naměřený vítěz je aplikován pásmo po pásmu: pásmo, které DAQ nedokáže kalibrovat, nebo u kterého selže přizpůsobení sklonu DAQ, převezme vlastní linii cíle (`ReflectanceElBandMethods` určuje, o kterou se jedná) — cíl, který prošel kontrolou kvality s výstrahou (vyřazený panel, nefunkční pásmo), také soutěží s rozdělením DAQ na základě tohoto naměřeného skóre a rámec, proti kterému prohraje, je označen jako `ReflectanceChain=daq` — v případě, že není přítomen žádný cíl nebo selže kontrola kvality (QA), se přejde na dělení sestupného toku DAQ (ρ = π·L/E);**`target`**je přísné (bez nahrazení DAQ);**`daq`**upřednostňuje chování řízené DAQ. Geometrie cíle (ArUco / pevná oblast zájmu / pás) pochází z konfigurace cíle projektu; `--target-reflectance-dir DIR` uchovává**naměřené** skeny (`<serial>.csv`) vyhledané podle sériového čísla/QR cílové jednotky, s nominálními spektry T3/T4P jako záložním řešením.

Cesta odrazivosti DAQ automaticky vyřeší **časově shodné sestupné záření**ze zaznamenaného**`.daq`**(DAQ-U/M/E)**nebo nativního souboru `.csv`**systému DAQ-M nalezeného společně se snímky. Pokud není kalibrační balíček pro danou kameru nebo systém DAQ uložen v místní mezipaměti,**pipeline jej při prvním použití automaticky načte z AWS** (vyžaduje jednorázové připojení k internetu; uloží se do mezipaměti pod `~/.chloros/`).**Čí je to měření.**Hodnota `.daq` uložená při záznamu je měření kamery (kamer), kterou popisuje — pojmenované v doprovodném souboru `<file>.daq.cameras.json` vedle ní (`lattice capture`, `multi-capture`, `array-capture` i v grafickém rozhraní; surová série snímků uvádí hodnoty jednotlivých členů ve svém manifestu) — a `process` je přiřazuje**pouze k obrázkům z těchto kamer**. Samostatná kamera bez vlastního záznamu proto již nevyužívá DAQ jiné kamery; přiřaďte k ní DAQ nebo proveďte vlastní záznam. Vlastní záznam DAQ (`daq pool-record` nebo záznam spuštěný v grafickém rozhraní) či jakýkoli `.daq` bez tohoto sidecaru (včetně starších snímků) stále slouží všem kamerám.

#### Čtení pixelů odrazivosti (Pix4D / Metashape / vaše vlastní skripty)

Odrazivost je uložena jako celé číslo DN a **hodnota DN, která znamená ρ = 1,0, závisí na zdrojové kameře**:

| Zdroj | ρ = 1,0 je | Jak zjistit |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (rezerva až do ρ 2,0) | Soubor je označen XMP tagem `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (omezeno na ρ 1,0) | Žádné značky XMP typu `Chloros:*` — právě tato absence *je* signálem. |

**Přečtěte si `Chloros:PixelScale` a vydělte tím**, místo abyste předpokládali konstantu. Značka je definována v doméně uint16, takže zůstává `32768` napříč výstupními formáty, které mění měřítko — `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)` a `TIFF (32-bit, Percent)` jsou všechny samopopisné (nejprve normalizujte uložený datový typ zpět na uint16: ×257 z 8 bitů, ×65535 z float).

> **Soubory ze starších verzí nemusí obsahovat žádné měřítko.** 8bitový TIFF s odrazivostí je přepočítán a označen jako `Chloros:PixelScale` stejně jako ostatní. Verze před 1.3.0 jej místo toho *ořezávaly* na rozsah 0..255, takže každá hodnota nad ρ≈0,008 byla zploštěna na 255, a vynechávaly jak `Chloros:PixelScale`, tak dvojici `MicaSense:RadiometricCalibration`. **Pokud v souboru odrazivosti LATTICE tento tag chybí, nepředpokládejte žádné měřítko — soubor znovu zpracujte**, místo abyste dělili pixely, které nikdy dělitelné nebyly.

#### Čtení indexových pixelů (index NDVI a další vegetační indexy)

**Celé číslo v indexovém rastru představuje kontrastní roztažení pro daný snímek, nikoli mapu v rozsahu −1…1.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` a `JPG (8-bit)` — včetně 8bitového souboru TIFF, do kterého se zapisuje 8bitový snímek — ukládají

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

kde `IndexMin` / `IndexMax` jsou **nejnižší a nejvyšší konečné hodnoty indexu daného snímku**. DN 0 není −1 a plný rozsah není +1.

Program Chloros označuje oba koncové body, aby mapování zůstalo invertibilní:

| Značka | Význam |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — konvence uvedená níže |
| `Chloros:IndexMin` | hodnota indexu uložená jako DN 0 |
| `Chloros:IndexMax` | hodnota indexu uložená jako plný rozsah |

Přečtěte je a invertujte, přičemž nejprve normalizujte uložené DN na rozsah 0…1 podle maximální hodnoty uloženého datového typu, aby jeden výraz pokryl všechny formáty celých čísel:

```python
import json, subprocess
import numpy as np, tifffile

tags = json.loads(subprocess.run(
    ["exiftool", "-j", "-XMP:all", path],
    capture_output=True, text=True).stdout)[0]

# float(): exiftool hands back a negative real as a quoted string, so
# IndexMin arrives as str on most real NDVI data while IndexMax is a number.
lo, hi = float(tags["IndexMin"]), float(tags["IndexMax"])

raw = tifffile.imread(path)                    # uint16 for TIFF (16-bit)
ndvi = lo + raw.astype(np.float64) / np.iinfo(raw.dtype).max * (hi - lo)
```

Čtení nevyžaduje `-config pix4d.config` — exiftool čte tyto vlastnosti XMP samostatně. Konfigurace je nutná pouze pro jejich *zápis*.

> **`IndexMin == IndexMax`** znamená, že každý pixel daného snímku měl stejnou indexovou hodnotu. Rastr je vyplněn v polovině rozsahu a není čím dělit — vezměte koncovou hodnotu.

> **Žádné značky `Chloros:Index*` znamenají, že není zadáno žádné mapování.** Export LUT je kolorovaný výstup, jehož pixely jsou barevné přechody vázané na vaše prahové okno LUT, a snímek bez pixelu s konečným indexem nemá žádný rozsah, který by bylo třeba zadat. V žádném z těchto případů nepředpokládejte mapování — platí stejné pravidlo jako u chybějícího `PixelScale`.**Protože se roztažení provádí pro každý snímek zvlášť, nelze dva snímky z jednoho letu porovnávat z hlediska DN**— každý má své vlastní koncové body, takže stejná hodnota NDVI odpovídá v různých snímcích odlišným hodnotám DN. Pro kvantitativní analýzu v rámci celého letu (zónové statistiky, pevné prahové hodnoty, časové řady) exportujte**`TIFF (32-bit, Percent)`**: indexové rastry jsou tam zapsány jako float32**samotné hodnoty indexu**, bez nutnosti přepočítání a bez nutnosti tagů. Navzdory názvu formátu nejsou indexové produkty násobeny ×100 — toto škálování se vztahuje pouze na odrazivost s celočíselným datovým typem.

#### Maska omezovače při spektrálním rozkladu (na které pixely bylo pásmo omezeno)

Spektrální rozklad 3×3 může způsobit, že se pásmo stane záporným — tam, kde světlo dané pásmo téměř neobsahuje, nebo tam, kde scéna leží mimo rozsah popsaný rozkladovou maticí kamery. Nejedná se o skutečnou hodnotu, proto ji export nastaví na 0 — a živý náhled zobrazuje stejnou hodnotu 0. Pásmo může být omezováno na několik pixelů nebo téměř na všechny; maska zaznamenává obojí. `Chloros:Band{R,G,B}UnmixClampFraction` udává, jak velká část snímku byla takto ovlivněna; maska omezování udává, **které pixely**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— ve složce `clamp_masks/`**vedle** složky produktu, nikdy uvnitř ní (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`; export z `process` do `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`) — pro každý produkt radiance, odrazivost a index, u nichž došlo k omezení alespoň jednoho pixelu: to platí jak pro snímky, tak pro exporty (a také pro TIFF soubory jednotlivých snímků). Jednopásmové soubory, na vlastní pixelové mřížce produktu (včetně orientace, oblasti zájmu (ROI), binningu a zarovnání pole). Každá maska uvádí název kamery svého produktu (`Make` / `Model` / sériové číslo), čas pořízení a značky objektivu a uvádí seznam pásem, která její bity představují (`XMP-Camera:BandName`, nejprve bit 0); její `ImageDescription` a `Software` ji označují jako omezovací masku a neobsahuje žádnou kalibraci.
- **Bity**: bit 0 = pásmo v kanálu R produktu (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Nastavený bit znamená, že hodnota pixelu v daném pásmu byla oříznuta na 0 — považujte jej a jakýkoli index z něj odvozený za málo spolehlivý.
- **`Chloros:UnmixClampMask`** je vyznačeno na každém produktu, u kterého byl spuštěn proces oddělování: cesta k masce relativní ke složce samotného produktu (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, vždy oddělená znakem `/` — připojte ji ke složce snímku, abyste masku otevřeli), nebo `none`, pokud nebyl oříznut žádný pixel. Chybějící značka znamená, že u produktu nebyl nikdy spuštěn proces unmix (kamery RGB nebo monochromatické, úrovně raw/debayered).
- `lattice capture` / `array-capture` vytisknou cestu k masce pod uloženým souborem (`clamp mask: …`) a výsledky snímání ji uvádějí jako `clamp_mask` daného snímku.
- `lattice index --live --save-multiband` zapíše jednu masku pro každou zdrojovou kameru pro vícepásmový stack — viz [`lattice index` — masky pro vícepásmový stack](#lattice-index--multiband-stack-masks). Video nemá žádnou masku: `array-build-video` místo toho vykazuje hodnoty omezovacích frakcí pro každý snímek (viz [`array-build-video`](#array-build-video--offline-reprocess-a-saved-burst)).

Protože žádná složka produktu neobsahuje masku, v žádné úrovni, lze složku produktu (`radiance/`, `reflectance/`, složka `*_Images/` v exportu `process`) předat Pix4D, Metashape nebo jinému fotogrammetrickému softwaru tak, jak je — včetně podsložek — a přijímá pouze snímky. Předávejte mu složku s produktem, nikoli složku nad ní (ta obsahuje také `clamp_masks/` a všechny ostatní typy exportů). Vlastní import programu Chloros masky přeskočí. Čtení masky: `(mask & 1) != 0` znamená „zde bylo oříznuto pásmo R-kanálu“, `& 2` G, `& 4` B.

#### EXIF přenesený do exportu

`process` zkopíruje **blok GPS a jeho ExifIFD** ze zdrojového snímku do každého výstupu, takže
export obsahuje `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` a
`CameraSerialNumber` spolu s georeferencí.

**`FocalLength` není pro fotogrammetrii volitelný.** Pix4D vypočítává vzdálenost vzorku na zemi (GSD) na základě
ohniskové vzdálenosti a nadmořské výšky; pokud tento tag chybí, použije se místo toho zcela nesprávné měřítko. Při jednom
letu nad pomerančovým hájem s 49 snímky proměnil chybějící tag lokalitu o rozměrech 411 m × 160 m na rekonstruovanou
47,8 km × 13 km – ortofotomapou o rozlišení 455 MP, která obsahovala převážně prázdná data, což se poté jevilo jako problém s dlaždicováním a
problém s formátem BigTIFF, než někdo zkontroloval GSD. Pokud vaše ortofotomapa vyjde v nepravděpodobném
měřítku, spusťte nejprve na exportovaném produktu příkaz `exiftool -FocalLength`.

Kopie záměrně **není** `-all:all`: Strukturální značky IFD0'jeho strukturální tagy narušují výstup LATTICE při
kopírování a `ExifImageWidth` / `ExifImageHeight` jsou vyloučeny, protože popisují
*zdrojový* snímek – export, jehož rozměry byly kdykoli změněny, by jinak obsahoval rozměry
v rozporu s vlastní rastrou. XMP se zapisuje přímo, nikoli kopírováním, protože ExifTool
při kopírování bloku XMP vyřazuje XMP tagy ze stejného volání (což by vedlo ke ztrátě kalibračních tagů MAPIR).

### Kam se výstupy ukládají

Výstupy se ukládají **do projektové složky, seskupené podle fotoaparátu a poté podle formátu souboru**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Složka fotoaparátu pro LATTICE je `LATT-<sensor>-<lens>-F<filter>` (odpovídá EXIF
`Model`) a `<model>_<filter>` pro Survey3 — dvě kamery, které sdílejí snímač a filtr, ale liší se
objektivem, mají oddělené stromy, protože se liší vinětace, zorné pole a zkreslení. Formát
složky je následující: `--format`: `tiff16`, `tiff8`, `png8`, `jpg8`, případně `tiff32` pro
`TIFF (32-bit, Percent)`.

> **Každý exportovaný produkt si zachovává název zdrojového souboru.** Export radiance souboru
> `capture_…_raw.tif` se stále nazývá `capture_…_raw.tif` — pouze se nachází v
> `tiff32/Radiance_Images/`. **Produkt identifikuje složka, nikoli název souboru**, takže globální vyhledávání
> podle `*radiance*.tif` nic nenajde; místo toho použijte shodu podle adresáře.

### Záznamy ze světelného senzoru — kalibrované `.daq` + `.csv`

`process` zpracovává také záznamy `.daq` ve vaší vstupní složce a **nepotřebuje**
k tomu nepotřebuje žádné snímky: samostatně letící DAQ-U / DAQ-M / DAQ-E představuje kompletní
zachycení a složka obsahující pouze soubory `.daq` je platným vstupem.

Záznam z DAQ lze provést **bez** jeho kalibrace — to je to, co veřejně dostupné
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) ve výchozím nastavení: zapisují surová data ze senzorů a označují soubor tak, aby
program Chloros načítal tovární kalibraci daného senzoru **přes sériový port** (nejprve z lokální mezipaměti,
poté z cloudu MAPIR) a aplikuje ji. `process` zapíše výsledek zpět:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` obsahuje jeden řádek na jedno měření: časové razítko UTC, integrační čas, celkový výkon,
fotopický/skotopický lux, PPFD (a jeho rozdělení na modrou, zelenou a červenou složku), špičkovou vlnovou délku a poté
celé spektrum na vlastní vlnové mřížce senzoru. Soubor `.daq` se znovu importuje, aniž by byl
podroben druhé kalibraci.

`<name>` je vlastní název souboru záznamu. Dva záznamy v různých složkách mohou sdílet
jednu hodnotu — údaje pro jednotlivé snímky (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) uložené ve
stejné milisekundě dvěma kamerami — a každý z nich přesto získá své vlastní výstupy: první si ponechá
`<name>`, druhá přidá složky, které ji odlišují, **za** názvem
(`<name>__<capture>_daq_calibrated.daq`), takže datum a čas záznamu jsou stále na začátku.

V případě úspěchu hlásí běh `Light-sensor products written: N (calibrated .daq + .csv)`.
Text v závorce popisuje, co bylo skutečně zapsáno, takže zní
`(RAW COUNTS — this sensor has no calibration bundle)` pro senzor bez balíčku a
`(N calibrated, M raw counts)` pro složku obsahující obojí. Vlastní záhlaví backendu
`[DAQ-EXPORT]` a `[RUN-SUMMARY]` odvozují své znění stejným způsobem — žádný z
těchto tří záznamů nelze označit za kalibrovaný surový export.

Záznam DAQ-U / DAQ-M / DAQ-E, jehož kalibrační balíček nelze načíst – jste
offline nebo daný senzor nemá v souboru žádnou kalibraci – je **přeskočen s uvedením důvodu** na
řádku `[DAQ-EXPORT]` a nikdy není zapsán jako „kalibrovaný“ soubor obsahující surová měření.
Připojte se k internetu a spusťte proces znovu. Důvodem je ten, který čtečka skutečně
zjistila pro daný soubor (nečitelná struktura, chybějící balíček, chyba zápisu), a souhrn
běhu uvádí **jednotlivé** důvody — dvacet přeskočených souborů z jednoho důvodu se vykazuje jako jeden
důvod, nikoli jako dvacet opakování tohoto důvodu. CLI na konci
běhu pojmenovává každý přeskočený záznam spolu s důvodem:

```
⚠ Light-sensor recording not exported: <file>.daq: no calibration bundle available for …
```

Složka se záznamy, z nichž se **žádný** neexportuje, selže: viz
[Spuštění, při kterém se nezapíší žádné obrázky, selže](#a-run-that-writes-no-images-fails).

Záznam, jehož název končí na `_calibrated` nebo `_raw` (`field_raw.daq`), **není**importován**: jedná se o názvy, které `process` přiděluje svým vlastním výstupům, a výstup načtený zpět
jako záznam by byl započítán dvakrát. CLI to uvádí a uvádí název souboru —
`Not imported as a light-sensor recording: field_raw.daq … Rename the file to import it.`
— místo toho, aby hlásilo prázdnou složku. Takový záznam před zpracováním přejmenujte.

#### Záznamy DAQ-A se exportují jako surové počty

Rodina **DAQ-A** předchází systému balíčků pro jednotlivá sériová čísla a nemá žádný kalibrační balíček,
který by bylo třeba načíst — místo toho se kalibruje v terénu pomocí kalibračního terče, což je
důvod, proč jej nikdy nepotřebovala. Odmítnutí těchto záznamů by jim znemožnilo
vůbec získat jejich hodnoty, proto se exportují pod **jiným názvem**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Jde o odlišný název souboru, nikoli o příznak uvnitř souboru, protože název musí zůstat zachován
i při zasílání e-mailem jako pouhý název. Záhlaví `.csv` uvádí
`raw spectral sensor counts (NOT irradiance)` a upozorňuje, že hodnoty jsou srovnatelné
**v rámci** souboru — což je přesně to, k čemu je kalibrace pomocí kalibračního terče využívá — a
nikoli mezi senzory. Fotometrické sloupce závislé na výkonu (celkový výkon, fotopický a
skotopický lux, PPFD) jsou zapsány jako **NULL** namísto integrace z počtů, a souhrn
běhu uvádí `RAW COUNTS`, takže „exportované“ v protokolu nelze interpretovat jako ozáření.

Starší záznamy **v1.01 / v1.02** (tyto zapisuje DAQ-A-SD) neobsahují epochu pro jednotlivé odečty,
pouze čas zápisu souboru. Nástroj pro porovnávání snímků ↔ sestupného záření je stále odmítá — porovnání
snímku s časem zápisu by bylo neviditelně nesprávné — ale exportér je načte a
CSV-soubor vypíše `clock=daq_created_on`, takže produkt uvádí, na jakém časovém systému se nachází.

### Poznámky

- `process` automaticky rozpozná, zda se ve vaší složce nacházejí soubory typu Survey3, „LATTICE“ nebo smíšené.
- Průběh se přenáší prostřednictvím Server-Sent Events; CLI zobrazuje aktuální průběh pro jednotlivá vlákna (Detecting, Analyzing, Processing, Exporting).
- V případě Linuxu/Jetsonu CLI kontroluje swap a před zpracováním velkých složek může zobrazit varování. Debayer s podporou textur také automaticky aplikuje omezení frekvence GPU na Jetsonech s nízkou spotřebou (Nano, Orin Nano).
- V případě úspěchu hlášení o provedení uvádí, kolik obrazových výstupů bylo zapsáno (`Image products written: N`, případně `(plus M clamp masks)`, pokud při odstraňování mixu došlo k oříznutí některého pixelu).
- **`process` bezpečně sdílí spuštěný backend.** Automaticky restartuje již spuštěný backend pouze v případě, že je tento backend nečinný, má načtený předchozí projekt a neobsahuje žádné kamery, pole ani DAQ. Backend, který obsahuje cokoli z výše uvedeného (GUI streamující pole, skript SDK, záznam DAQ), se používá tak, jak je, a nikdy se nerestartuje, takže `process` může běžet souběžně s aktivním polem. Pokud backend hlásí probíhající běh, `process` počká až 15 s na jeho dokončení (vyčištění právě dokončeného běhu trvá asi jednu sekundu), poté odmítne s chybou `Backend is mid-run (processing). Wait for it to finish, or pass --restart to force a fresh backend (this kills the in-flight run).` Globální příznak `--restart` stále vynutí spuštění nového backendu a ukončí na něm jakoukoli relaci kamery, pole nebo DAQ. Backend, který běží jako služba Linuxu systemd, se nikdy nerestartuje: `process` jej využívá a `--restart` místo toho vypíše `sudo systemctl restart <unit>` (viz [Spuštění backendu jako služby](#running-the-backend-as-a-service-headless)). Stejně tak ani vlastní backend aplikace Chloros: při otevřené aplikaci jej používá `process` a načítá svůj projekt namísto projektu aplikace (backend spuštěný na jeho místě by otevřenou aplikaci zablokoval).
- **Probíhající úloha si zachovává svůj projekt.** Zatímco probíhá zpracování — spuštěné aplikací, `process`, skriptem SDK nebo `chloros-mcp` — backend odmítá vytvářet, otevírat, importovat do, přidávat nebo odebírat soubory z, přesouvat, smazat nebo měnit nastavení zpracování projektu, a to s chybou HTTP 409 `PROCESSING_IN_PROGRESS` a `A processing run is in progress on project '<name>', so ... now is refused`; pokud se zobrazí `process`, proces se ukončí a zobrazí se `1`. Nastavení zobrazení (velikost miniatur) se stále ukládá. Režim zpracování a nastavení cílové odrazivosti projektu jsou odmítnuty stejným způsobem (spuštění si zachovává režim, ve kterém bylo zahájeno). Záznam uložený do projektu uprostřed běhu (z aplikace, CLI, SDK nebo `chloros-mcp`) se uloží jako obvykle, ale do seznamu souborů projektu se přidá až po ukončení běhu; což potvrzuje i příslušná odpověď (`project_add`), stejně jako záznam ze světelného senzoru zastavený v rámci projektu. Dříve mohl druhý klient během běhu přepnout otevřený projekt a probíhající běh se exportoval do jiného projektu.
- **Klávesová zkratka Ctrl+C zastaví běh v backendu, nikoli pouze v CLI.** CLI požádá backend o zastavení zpracování, vypíše `Waiting for the backend to stop (Ctrl+C again to stop waiting)...` a čeká až 120 s na `Processing stopped.`, aby se `chloros-cli process … && next` nespustil000865 na backendu, který je stále zaneprázdněn spouštěním. Druhé stisknutí kláves Ctrl+C ukončí čekání. `process` poté ukončí `130`, stejně jako jakýkoli přerušený příkaz.

#### Spuštění, které nezapíše žádné obrazy, selže

Pokud jste požadovali produkty a běh nezapsal **žádné** — pouze `project.json` a
`calibration_data.json` — `process` to považuje za selhání: vypíše
`Processing finished but wrote no image products.` a **ukončí se s nenulovým kódem**, takže to skript může
detekovat. Zpráva uvádí název projektové složky a obvyklé příčiny:

- vstupní složka nebyla rozpoznána jako záznam (zkontrolujte rozložení a `--input-level`), nebo
- všechny požadované produkty byly přeskočeny jako nepoužitelné pro dané kamery (např. požadavek na
  radianci/reflektanci z kamer podporujících pouze RGB).

Spusťte program znovu s parametrem `--verbose` a zkontrolujte protokol backendu, zda neobsahuje řádky `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
které vysvětlují vynechání u jednotlivých kamer, která se jinak nedostanou do výstupu CLI.

Záměrné spuštění pouze s metadaty — všechny produkty vypnuté a bez `--indices` — je stále
**úspěšné**, protože prázdný obrazový výstup je v tomto případě správným výsledkem.

#### Spuštění, u kterého nebylo možné ověřit exporty, selže

Po spuštění `process` zkontroluje tagy exportovaných souborů pomocí ExifTool. Pokud se ExifTool nepodařilo spustit,
vypíše `Ověření exportu SELHALO: ExifTool se nepodařilo spustit, takže tagy N z M exportovaných souborů
nebyly zkontrolovány (viz řádky [EXIFTOOL] v protokolu backendu)` and `Výstupy jsou zapsány a zůstávají
na svém místě; spusťte proces znovu, abyste je ověřili.`, does not print `Zpracování dokončeno!` a **ukončí se s nenulovým kódem**.
Výstupy jsou zachovány.

#### Které snímky se do běhu zahrnou

Zahrnuty jsou snímky dokončené před spuštěním `process`. Snímky uložené během běhu zahrnuty nejsou; `process`
upozorní, kolik jich bylo vynecháno, a při opětovném spuštění je zahrne.

#### Běh, při kterém došlo ke ztrátě snímku, selže

Pokud během běhu selhal snímek, `process` vypíše `N image(s) failed and were not exported` s
nejčastější příčinou, nevypíše `Processing complete!` a **ukončí se s nenulovým kódem**. Ostatní výstupy jsou
zapsány a zůstávají na svém místě a backendový protokol obsahuje každou chybu.

Totéž platí pro **spuštění pouze se světelným senzorem**: složka záznamů `.daq` z definice neobsahuje žádné snímky k exportu
a běh je posuzován na základě kalibrovaných souborů `.daq` / `.csv`, které místo toho zapsal.
Pokud nezapsal **žádný** záznam — všechny záznamy byly přeskočeny — vypíše
`Processing finished but wrote no light-sensor products.`, pojmenuje každý záznam podle
důvodu, proč byl přeskočen, a **ukončí se s nenulovým stavem**.

---

## `chloros-cli login`

Ověřte tento počítač pomocí cloudového účtu Chloros+. Přihlašovací údaje jsou bezpečně uloženy v mezipaměti v souboru `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Příklady

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (odstraněním části hesla nebo zduplikováním jeho částí). Při chybě 401 CLI automaticky zkusí znovu s `$$`,, poté s odduplikovanou polovinou hesla; pokud se opakovaný pokus podaří, přihlásí vás a vypíše správnou syntaxi s jednoduchými uvozovkami, kterou můžete použít příště.

> **Použití bez grafického rozhraní/prostřednictvím skriptu: absence uložené relace znamená výzvu k přihlášení.** Základní příkazy (`process`, `status`, `export-status`, `time-sync`, …) spuštěný bez uložené relace před pokračováním vyžádá `Email:` / `Password:` ze standardního vstupu. Je-li vstup stdin uzavřen (cron, jednotka systemd), dojde okamžitě k selhání s chybou `login cancelled`; je-li vstupem stdin otevřený terminál nebo potrubí, program čeká na vstup. Před naplánováním úloh bez grafického rozhraní spusťte na každém počítači jednou program `chloros-cli login EMAIL PASSWORD`.

---

## `chloros-cli logout`

Vymaže mezipaměť relace a při příštím volání vynutí nové přihlášení.

Odhlášení během probíhajícího zpracování jej nezastaví: zpracování se dokončí v režimu, ve kterém bylo zahájeno, a další zpracování proběhne ve standardním režimu. Uvádí to `logout` (`The processing run in progress finishes in <mode> mode; the next run uses standard`) a také aplikace při odhlášení odhlásíte.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Zobrazí aktuální úroveň licence (Iron/Copper/Bronze/Silver/Gold), ověřeného uživatele a počet vazeb na zařízení.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Zjišťuje aktuální průběh exportu Thread-4. Lze bezpečně volat **během** spuštění příkazu `process` z jiného shellu.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Nastaví jazyk zobrazení v CLI (podporováno 38 jazyků, včetně CJK, RTL a indických jazyků). Na starších konzolích, které nedokážou skript vykreslit, se plynule přepne na angličtinu.

```
chloros-cli language [LANG_CODE] [--list]
```

### Příklady

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Příkazy pro složku projektu

Tyto příkazy spravují výchozí umístění složky projektu (sdílené s grafickým rozhraním).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

`get-project-folder` vypíše složku, do které se ukládají nové projekty. Pokud nakonfigurovaná složka již neexistuje (byla smazána, přejmenována nebo se nachází na disku, který není připojen), CLI, stejně jako grafické rozhraní, použije výchozí složku: výchozí složka se vypíše na stdout, varování s názvem chybějící složky se zapíše na stderr a výstupní kód je `0`.

---

## `chloros-cli update`

Pouze pro Linux/Jetson. Zkontroluje `version_url` z `/etc/chloros/update.conf` a pokud existuje novější verze, stáhne odpovídající `.deb` a ihned jej nainstaluje pomocí `sudo dpkg -i` (bez výzvy k potvrzení; `sudo` může vyžadovat zadání hesla). Na zařízení Jetson je aktualizace nabízena pouze pro systém JetPack 6. Ve systémWindows tento příkaz neprovede žádnou akci a vygeneruje hlášení `up to date`: aktualizujte pomocí nového instalačního programu.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

V systému Linuxu/na zařízení Jetson provádí CLI **při každém spuštění automatickou kontrolu aktualizací** (neblokující, nikdy nezpomalí příkaz): načte `/etc/chloros/update.conf`, uloží výsledek do mezipaměti na 1 hodinu v `~/.chloros/update_cache.json` a vypíše `Update available: vX.Y.Z / Run: chloros-cli update`, pokud existuje novější verze. V případě jakékoli chyby a v systému Windows se tato kontrola tiše přeskočí; příkaz `CHLOROS_DISABLE_UPDATE_CHECK=1` ji vypne. Kvůli hodinové mezipaměti se může stát, že verze vydaná v průběhu této hodiny bude stále vyhodnocena jako aktuální.

---

## `chloros-cli selftest`

Provede 7krokový zkušební test: verze, dostupnost portů, spuštění backendu, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), přítomnost modelu odšumovače, připravenost CUDA+odšumovače. Využívá běžící backend nebo jej pro účely testu spustí a poté zastaví. Vypíše `N/7 checks passed` a neúspěšné kontroly a ukončí se `0` pouze v případě, že všech sedm kontrol proběhne úspěšně; kontrola CUDA+odšumovače selže na počítači bez GPU s podporou CUDA (viz [Ověření instalace](#verify-the-install)).

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

Stav a řízení PTP grandmasteru. Host Chloros spouští PTP grandmaster; kamery LATTICE a jednotky DAQ-E se k němu připojují jako slave pro časová razítka napříč zařízeními (cca 1 ms mezi zařízeními). PTP synchronizuje **časová razítka**: kamery v poli exponují současně, protože je spouští hardwarový spouštěč M8 (s prodlevou méně než 100 µs mezi moduly), nikoli kvůli PTP. `chloros-cli time-sync` bez podpříkazu spustí `status`.

| Podpříkaz | Popis |
| --- | --- |
| `status` | Zobrazí stav grandmastera, priority BMCA, identitu hodin a informaci, zda jeho pakety dosahují sítě: `health` je `OK` (`STOPPED`, pokud grandmaster neběží), nebo `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` s `health_detail`, a poté se na stderr vypíše řádek `WARNING:` (stdout zůstává ve formátu JSON). `sync_sent` / `announce_sent` počítají pouze zprávy, které odešly; `*_send_errors` a `last_send_error` počítají ty, které neodešly. Po výpadku a obnovení spojení s kamerou grandmaster sám obnoví své sokety (`rebinds`, `last_rebind`). |
| `peers` | Seznam podřízených zařízení detekovaných prostřednictvím Delay_Req (kamery + senzory DAQ-E). |
| `cameras` | Stav PTP pro jednotlivé kamery (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Restart procesu grandmastera. |
| `set-priority --priority1 N --priority2 N` | Přepsání priorit BMCA. |

Grandmaster vysílá oznámení na síťové karty kamer (na jakémkoli adaptéru s adresou `169.254.x.x`). Pokud není žádný k dispozici, přepne se na adaptér výchozí trasy pouze v případě, že je připojen kabelem, nikdy na Wi-Fi ani virtuální adaptér (Tailscale, WSL), protože žádná kamera ani DAQ-E se nenachází za takovým adaptérem. Pokud není co oznamovat, běží bez soketů (`health` `NO_SOCKETS`) a při připojení další kamery nebo DAQ se znovu vytvoří na síťové kartě, která je v tu chvíli aktivní; `time-sync restart` to provede okamžitě.

Grandmaster běží ve vlastním procesu, vedle backendu (`own_process` / `process_pid` v `time-sync status`), s vysokou prioritou ve Windowsu (`process_priority`), takže zaneprázdněný backend (zpracování, export, několik příkazů najednou) nemůže zpozdit časy, které zaznamenává; dříve mohlo jedno takové zpoždění posunout čas všech kamer najednou. Tento proces končí s backendem; pokud jej nelze spustit, grandmaster běží uvnitř backendu jako dříve (`own_process` false, což potvrzuje i protokol backendu).

### Příklady

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

Ovládání kamer LATTICE. Každý podpříkaz prochází backendem Chloros; backend spravuje fond kamer, takže následující volání z CLIu znovu používají stejný otevřený popisovač.

### Běžné volby (společné pro většinu podpříkazů)

| Příznak | Popis |
| --- | --- |
| `-d, --device N` | Index kamery (výchozí: 0). |
| `-s, --serial SN` | Konkrétní sériové číslo; přepisuje `--device`. |
| `--serials SN1,SN2,…` | Sériová čísla oddělená čárkami pro provoz s více kamerami. Příkazy [pro dočasné relace s více kamerami](#multi-camera-transient-sessions) otevřou pouze tyto kamery, přičemž první z nich spustí spoušť. |
| `--all` | Provoz na všech nalezených kamerách. |
| `--exposure US` | Doba expozice v mikrosekundách. |
| `--gain DB` | Zisk v dB. |
| `--pixel-format FMT` | např. `BayerRG8`, `BayerRG12`; `Mono8`, `Mono12` na M3M. Název určuje bitovou hloubku v rámci vlastní rodiny kamery: na čipu M3M určuje název typu Bayer formát Mono pro danou hloubku (`BayerRG8` určuje `Mono8`), na čipu M3C určuje název typu Mono název typu Bayer (`Mono12` nastavuje `BayerRG12`). |
| `--width N` / `--height N` | Rozměry snímku. |
| `--preset {default,high_quality,high_speed,triggered}` | Použít předvolbu nastavení: formát pixelů, expozici, zesílení a snímkovou frekvenci (`high_speed`: `BayerRG8`, 5 ms, 6 dB, 60 fps; `high_quality`: `BayerRG12`, 20 ms, 0 dB; `lattice presets` je uvádí). Na M3M je formát Mono se stejnou hloubkou: `high_speed` zachycuje `Mono8`, `high_quality` a `triggered` `Mono12`. Příznak, který je s ním zadán (`--exposure`, `--gain`, `--pixel-format`) má přednost před předvolbou a nastavení, které předvolba neuvádí, zachovává vlastní nastavení kamery. Všechny volnéprovoz, s výjimkou `triggered`, který aktivuje kameru na hardwarovou změnu stavu na lince 2 — pokud na této lince není žádný signál, bude kamera čekat donekonečna, místo aby pořizovala snímky. |
| `-o, --output DIR` | Výstupní adresář (výchozí: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Velikost paketu GVSP. `auto` spouští sondy ICMP+GVSP; `jumbo` = 9000; `standard` = 1500. |

### Pracovní postup prvního připojení kamery LATTICE

```bash
# 0. A backend must be running: the Chloros desktop app (Windows),
#    `chloros-backend`, or the Linux service (see "Start the Backend")

# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Full sensor (no binning, no ROI); the frame rate falls to
#        what the host and wire deliver, and the faster options are
#        printed
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Přehled podpříkazů

#### Vyhledávání a informace

| Podpříkaz | Účel |
| --- | --- |
| `lattice info` | Seznam připojených kamer (výrobce, model, sériové číslo, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analýza hostitelského systému za účelem optimální konfigurace kamery. `--no-discover` přeskočí vyhledávání kamer (rychlejší, analýza pouze pomocí síťové karty). |
| `lattice network [--fix] [--estimate] [--cameras N] [--pixel-format FMT]` | Kontrola/oprava nastavení síťové karty (v systému Linux také vyrovnávací paměti pro příjem soketů jádra — viz [Linuxové hostitelské systémy](#linux-hosts--socket-receive-buffers)); odhad šířky pásma/FPS. |
| `lattice network-analysis --master SN [--slaves SN1,SN2,…] [--width N] [--height N] [--pixel-format FMT] [--binning {1,2,4}] [--force-tier TIER] [--packed {auto,on,off}] [--backend-url URL] [--json]` | Doporučení ohledně síťových+ doporučení pole (vrací `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `--width` / `--height` jsou ve výchozím nastavení nastaveny na plný senzor, `2048` × `1536`. Ukončuje se u `0` pro `ok`, `auto_capped_fps` a `auto_shrunk`, a z `1` na `needs_force_slip` a `error`, s nebo bez `--json`. Dosáhne backendu na `--backend-url` (výchozí hodnota `http://127.0.0.1:5000`), nikoli na `CHLOROS_BACKEND_URL`. `auto_capped_fps` zachovává požadované rozlišení, ale omezuje cílovou snímkovou frekvenci (fps) — přečtěte si `recommended.recommended_target_fps` a předávejte jej jako cíl připojení; považujte to za úspěch, nikoli jako chybu. `auto_shrunk` se objeví pouze v případě, že požadovaný snímek nelze doručit při žádné snímkové frekvenci; connect nikdy sám od sebe nezmenšuje snímky. |
| `lattice analyze-array [--models M1,M2,…] [--binning {1,2,4}] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--packed {auto,on,off}] [--json]` | Analýza „co by, kdyby“ bez otevření kamer (`--width` / `--height` výchozí `2048` × `1536`). Jak tento, tak i `network-analysis` předpovídají, co bude connect streamovat: bez `--pixel-format` platí formáty, které connect používá pro jednotlivé kamery (Mono12 / BayerRG12), a s `--packed auto` (výchozí) se použije rozhodnutí, které by connect učinil na tomto počítači ohledně [zbaleného 12bitového](#packed-12-bit-transport---packed), které by Connect na tomto počítači zvolil — uvedená hodnota fps tedy představuje rychlost, s jakou pole běží. Oba výstupy zobrazují volbu balení. **`--n-active` je celkový počet kamer v síti, nikoli pouze počet kamer v tomto poli**— zvyšte jej, pokud souběžně streamují samostatné kamery nebo je kapacita sítě vypočítána na základě požadavku, který je podhodnocuje (výchozí: `len(--models)`). Vždy vypíše souhrnné řádky `Wire budget:` (požadovaných MB/s vs. strop bezpečný proti kolizím) a `Max cameras:` a označí `** OVER-SUBSCRIBED**`, když pole přetěžuje linku — viz [Model fps a burst pole](#array-fps--burst-model). |
| `lattice gpu` | Zobrazit stav GPU. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Zkontrolovat nebo aktualizovat firmware kamery. Místní volba `.fwa` je pevně nastavená: soubor v `firmware/<MODEL_PREFIX>/` odpovídající `MIN_FIRMWARE_VERSION` dané sestavení se nahraje, pokud je k dispozici (nejvyšší verze pouze jako záložní řešení), takže novější obraz dodavatele připravený na disku zůstává neaktivní, dokud nedojde ke změně tohoto nastavení — záměrně novější verze se do jednotek dostávají prostřednictvím podepsaného manifestu AWS, což je upřednostňováno, pokud je novější. |
| `lattice presets [--apply NAME]` | Zobrazí seznam předvoleb kamery nebo je aplikuje. `--apply` nastavuje parametry předvoleb (formát pixelů, expozici, zesílení, snímkovou frekvenci, spouštěč) a vypíše tato nastavení tak, jak je kamera načte poté: na M3M formát Mono pro hloubku předvolby; snímkovou frekvenci, při které běží, která může být nižší, než předvolba vyžaduje (`high_speed` vyžaduje 60 fps; full-frame kamera na 1 Gbps spojení běží přibližně na 35). Všechna ostatní nastavení, která program „open“ zapsal, se vrátí do původního stavu. Příkaz `--apply default` resetuje kameru na tovární výchozí hodnoty: tovární nastavení s volně běžící automatickou expozicí na Line2, vypnutým omezením propustnosti spojení a vypnutou korekcí vad, přičemž u RGB kamery zůstane zachována kalibrovaná vyváženost bílé (u všech ostatních je nastavena na jednotku). Do paměti kamery se nic neukládá, takže při příštím zapnutí se spustí z vlastní paměti stejně jako předtím. Jako `--preset` při snímání nebo připojení neexistuje předvolba `default`. |
| `lattice status` | Stav kamery v reálném čase. |

#### Snímání

| Podpříkaz | Účel |
| --- | --- |
| `lattice capture [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--no-daq] [--daq-tolerance-ms MS]` | Jeden snímek. **Ve výchozím nastavení ukládá všechny typy exportu**(`--processing all`); viz [Úrovně exportu záznamu](#capture-export-levels-the-all-default). `--processing` přijímá `all`, `raw`, `debayered`, `radiance`, `reflectance` nebo `display` (`preview` je přijat pro `display`); jakýkoli jiný název je odmítnut před dotykem kamery (ukončení `1`). `--levels` uloží explicitní podmnožinu (přepíše `--processing`). Pro záznam se otevře kamera, kterou Chloros nedrží, a její expozice se nejprve nastaví podle scény (nanejvýš na několik sekund), stejně jako u [dočasných příkazů](#multi-camera-transient-sessions); jakmile snímání skončí, ať už jakýmkoli způsobem, je tato kamera vrácena do stavu, v jakém byla nalezena (formát, expozice, barevný pipeline, všechna nastavení, která snímání změnilo — seznam v části [Více kamer (dočasné relace)](#multi-camera-transient-sessions)). Hodnota DAQ, která načítá použitý záznam, se uloží jako sidecar `.daq`, včetně záznamu pouze v surovém formátu; `--no-daq` / `--daq-tolerance-ms` jej upravují ([DAQ, který načítá záznam, ukládá](#the-daq-reading-a-capture-saves)). `--force-daq` je stále akceptován. `--jpeg-quality` = kvalita JPEG 1–100 (výchozí hodnota 95).**Soubory uložené do otevřeného projektu se k němu připojí.** Stejně jako v aplikaci se záznam, jehož soubory skončí ve složce projektu otevřeného v Chloros, přidá do seznamu souborů daného projektu; soubory uložené jinde zůstanou beze změny. Příkaz vypíše jeden řádek s touto informací (`Project: ...`), nebo – pokud právě probíhá zpracování – že jsou soubory zadrženy a připojí se po skončení zpracování. Totéž platí pro `multi-capture`, `array-capture` (`--smart` také) a `project capture`. |
| `lattice continuous [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--queue-depth N] [--no-daq] [--daq-tolerance-ms MS]` | Zápis na disk až do stisknutí Ctrl+C. Snímky z běhu `tiff`jsou snímky, které program `raw` zapisuje do souboru jako pole zachycených dat z dané kamery: mozaika snímače, jedna rovina s vlastními hodnotami, nekomprimovaná, s kalibračním paketem (2048 × 1536 12bitový snímek má velikost 6,0 MiB plus paket, bez ohledu na scénu), takže `process` z nich vychází. Soubory `png` a `jpg` obsahují 8 bitů, takže se jedná o snímky po demosaikování určené k prohlížení, od nichž soubor `process` nezačíná. V případě vazby na DAQ (`daq pool-bind`) nebo v poolu dostane každý snímek svůj vlastní soubor `.daq` a `.match.json` ve složce `daq/` dané série, protože `lattice capture` jeden uloží; `--no-daq` tuto funkci vypne a při zastavení se vypíše počet snímků. Každý snímek se zapíše jednou, včetně všech metadat; při stisknutí Ctrl+C se již zachycené snímky zapíší před uvolněním kamery. Program se sám zastaví, když se disk zaplní, vypíše `Failed:` / `Disk full:` a ukončí se jako `1`; každý snímek, který se nepodařilo zapsat, také ukončí program jako `1`. Kamera ponechaná v pohotovostním režimu pro hardwarové spouštění (`TriggerMode` `On`) je pro daný běh nastavena na volný provoz. Po skončení běhu je kamera vrácena do stavu, v jakém byla nalezena — její nastavení spouště a všechna nastavení, která běh změnil (expozice, zesílení, formát, …) — s výjimkou případů `--keep-settings`, kdy se nastavení běhu uloží do vlastní paměti kamery (volný běh), nebo `--clear-settings`, kdy se tato paměť resetuje na tovární nastavení. Záznam, který nezaznamená žádný snímek, je považován za selhání: pokud po dobu 10 s není zaznamenán žádný snímek, program se sám zastaví, vypíše `No frames:` s důvodem a ukončí se `1`. Snímky, s nimiž zapisovače nestíhaly držet krok, se počítají jako vynechané a při zastavení se vypíše kód `Dropped:` s důvodem (snímková frekvence kamery, zapisovačů a fronty); nižší snímková frekvence nebo 8bitový formát pixelů je zachová všechny. Příkaz se každých 2 s hlásí backendu; pokud dojde k jeho selhání bez zastavení záznamu (`kill -9`, nedostatek paměti), backend jej zastaví 30 s po poslední kontrole a vrátí kameru zpět (`stop_reason` `client_gone` a záznam v protokolu backendu). `multi-stream` a `align-stream` se chovají stejně. Spuštění, při kterém otevření kamery trvá déle, než je doba čekání příkazu (60 s; 180 s pro `multi-stream` / `align-stream`) — kamera se stále spouští — je backendem zrušeno: kamera je vrácena do původního stavu a uzavřena a žádný záznam není ponechán v chodu (504 HTTP `CLIENT_DEADLINE_PASSED`). |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Živý náhled MJPEG v prohlížeči: připojí kameru, vypíše URL streamu a otevře jej ve vašem prohlížeči. `--ae-damping` nastaví tlumení automatické expozice (0,4–100). |

##### Živé přenosy (MJPEG; bez RTSP)

Chloros poskytuje živé video jako **MJPEG přes HTTP**(`multipart/x-mixed-replace`, mezní hodnota `frame`) z backendu.**RTSP není podporován.** Otevřete URL v prohlížeči nebo ji načtěte pomocí libovolného MJPEG klienta (`cv2.VideoCapture(url)` z OpenCV, ffmpeg, VLC). Kamera musí být nejprve připojena v backendu (`lattice cam-connect`, `lattice array-connect`, `lattice viewer` nebo v grafickém rozhraní).

| URL (`http://127.0.0.1:5000` + …) | Stream |
| --- | --- |
| `/api/camera/stream/<serial>` | Náhled na displeji, až cca 30 snímků za sekundu. Pro prvek pole platí `?max_h=` (výchozí `720`) a `?jpeg_q=` (výchozí `80`) nastavují výšku a kvalitu JPEG. Toto je URL, kterou `lattice viewer` otevírá. |
| `/api/camera/<serial>/stream-annotated` | Stejný náhled s vykreslenými překryvy grafického uživatelského rozhraní: `zebra=1`, `crosshair=1`, `grid=3x3` (nebo `9x9`), `histogram=1`, `peaking=1`, `spot=1`; bez dotazu se použijí uložená nastavení překryvů projektu pro danou kameru. |
| `/api/camera/<serial>/stream-reflectance` | 8bitová odrazivost na pásmo: ρ = pixel ÷ část`X-Chloros-Pixel-Scale` (255, pokud není nastaveno `refl_max`). `fps` (výchozí `10`), `jpeg_q` (výchozí hodnota `95`), `max_h` (výchozí hodnota `0` = nativní), `band=<name>`, `align=1`. Formát JPEG, tedy se ztrátou. Odmítnuto s kódem HTTP 409, pokud kamera nemá odrazivost (RGB kamera, žádný kalibrační balíček, žádné údaje z DAQ). |

- **Srovnávání snímků napříč kamerami.**Každá část, kterou**člen pole** odešle, ve všech třech proudech nese hlavičky `X-Chloros-Group-Id` (číslovač synchronizované skupiny pole), `X-Chloros-Frame-Id` a `X-Chloros-Timestamp-Ns`. Části z různých kamer se stejným `X-Chloros-Group-Id` pocházejí ze stejného spouštěče. Samostatná kamera nemá žádnou skupinu a neodesílá žádné ID skupiny. Prohlížeče ignorují hlavičky jednotlivých částí; klient, který sám analyzuje vícedílný proud, je čte.
- **Pozastaveno, dokud je otevřeno zobrazení pole.** `/api/camera/stream/<serial>` člena pole odesílá statický snímek „paused“, dokud je otevřen kombinovaný živý proud pole (například zobrazení pole v grafickém uživatelském rozhraní), a obnoví se po jeho zavření.

#### Nastavení senzorů

| Podpříkaz | Účel |
| --- | --- |
| `lattice configure [{--get N1 N2… \| --set N=V N=V… \| --dump}] [--json]` | Čtení/zápis libovolného uzlu GenICam. `--set ExposureTime=…` (`Gain=…`) bez přepínačů `ExposureAuto` (`GainAuto`), které nejprve vypnou automatický režim, jak to dělá `lattice exposure --set`; pokud kamera odmítne zadanou hodnotu, vypíše `Failed to set <node>: …` na stderr a ukončí se `1`. `--get` a výchozí seskupené zobrazení vypisují `note:` pod `ExposureTime` / `Gain`, zatímco tento automatický režim běží — viz [Čtení expozice a zesílení v automatickém režimu](#reading-exposure-and-gain-under-auto-mode). |
| `lattice exposure [{--auto \| --auto-once \| --off \| --set US}] [--brightness 0-255] [--brightness-auto] [--damping 0.4-100] [--upper-limit US]` | Expozice a AE. Hodnota expozice `--set` nebo cíl `--brightness` se zachovají jako vaše (automatické řízení expozice Chlorosu ji nechává beze změny); zisk zůstává takový, jaký byl, takže v automatickém režimu se neustále přizpůsobuje směrem k cíli, dokud nenastavíte `lattice gain`. `--auto` vrátí ovládání expozice, `--brightness-auto` vrátí cíl zpět automatickému nastavení Chlorosu. Bez příznaků načte uzly expozice, a to se stejnými hodnotami `note:` jako `configure --get`. Pokud fotoaparát odmítne hodnotu `--set`, vypíše na stderr `Failed to set ExposureTime: ExposureTime=<value> refused (<camera error>); the camera accepts <min>..<max> now (ExposureAuto put back to <mode>)` — v automatickém režimu se vypnutý zápis obnoví, místo aby zůstal vypnutý — nevypíše řádek `Exposure: … us (auto OFF)` a ukončí se `1`. |
| `lattice gain [{--auto \| --off \| --set DB}]` | Zesílení a automatické zesílení. Při hodnotě `--set` kamera odmítá vypsat `Failed to set Gain: …` na stderr, nevypíše řádek `Gain: … dB (auto OFF)` a ukončí se s kódem `1`. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N]` | Oblast zájmu (ROI) snímače a binning. Binning je vždy **průměr** — viz níže. |
| `lattice format [--set FMT] [--list]` | Formát pixelů. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Hardwarová/softwarová spoušť. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (žádné příznaky = jednorázové vyvážení bílé) | Operace vyvážení bílé. Pouze kamery RGB/Bayer; u monochromatické M3M se tato operace neprovádí (přeskočí se). Funkce one-shot zachovává vypočítanou vyváženost bílé (`BalanceWhiteAuto` a zesílení); všechna ostatní nastavení, na která se otevřený zápis vztahuje, se vrátí do původního stavu, a to v plném rozsahu, pokud nelze funkci one-shot spustit. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct 2000-10000] [--get]` | Barevný pipeline pro zobrazení RGB. `natural` (výchozí) představuje levné živé zpracování; `enhanced` přidává odstranění barevných pruhů + živost + lokální kontrast CLAHE pro plný vzhled s paritou hubu při ~2× vyšších nákladech na zpracování na snímek, tedy při nižší **živé** snímkové frekvenci — uložené snímky vždy dostanou plné zpracování v každém případě. Pouze pro kamery RGB/Bayer; přeskočeno u monochromatického M3M. |
| `lattice color [--saturation 0-200] [--contrast 0-200] [--reset] [--get]` | Sytost/kontrast v živém náhledu (kamery s RGB filtrem); `100` je neutrální, `--reset` nastaví obě hodnoty na 100. Uložené snímky tím nejsou ovlivněny. Přeskočeno u monochromatické kamery M3M. |
| `lattice highlights [{--set -100..0 \| --reset \| --get}]` (žádné příznaky = číst) | Zobrazení **světlých oblastí** (kamery s RGB filtry): potlačí nejjasnější, nejbělejší pixely — např. jasný bílý povrch, který se vykreslí jako plochá bílá skvrna. `0` = vypnuto, `-100` = nejsilnější. Živý náhled a snímky na úrovni zobrazení se tím řídí; jas a odrazivost se tím nikdy neřídí a nelze tím obnovit oříznutá data snímače (pro to snižte expozici). Přeskočeno na monochromatickém M3M. |
| `lattice filter [--list]` | Vypíše modely filtrů, které Chloros zná (`RGN-IMX265`, `OCN`, `NGB`, …). Filtr kamery se načítá z řetězce modelu a nelze jej nastavit: `--set NAME` pouze vyhledá model a poté ukončí `1` pomocí `setting the filter model is not supported`. |
| `lattice power [--sleep]` | Zjišťuje stav napájecích a teplotních uzlů; přepíná do úsporného režimu. `--sleep` ponechá kameru v režimu spánku (limit propustnosti jejího spojení je zapnutý (`DeviceLinkThroughputLimitMode` `On`) na minimální hodnotě, úsporný režim je zapnutý, pokud jej kamera má) u všech ostatních nastavení se otevřená data vrátí do původního stavu; spánkový stav, který selže, se obnoví v plném rozsahu. Probuzení vrátí spojení do stavu, v jakém ho spánkový režim našel (limit, režim a zpoždění paketů `GevSCPD`, které kamera udržuje na hodnotě 0, dokud je limit zapnutý; záznam se uchovává na tomto počítači, takže jej najde i probuzení po restartu backendu), nebo v případě kamery, kterou tento počítač neuspával, ale která vypadá jako spící (její limit je zapnutý na minimální hodnotě), s vypnutým limitem a hodnotou `GevSCPD` ponechanou na 0 při 80 (tovární nastavení); limit nastavený nad minimální hodnotou zůstane beze změny. Sonda a probuzení vrátí zpět to, co jejich otevření zapsalo. |

> **Binning je vždy průměrný.** `lattice resolution --binning N` nastavuje průměrný binning a
> nenabízí přepínání režimů. Binning typu „Sum“ vynásobí každou hodnotu DN o N², což stojí zhruba dva clonové stupně
> rezervy pro světlé oblasti na každý krok binningu; Chloros tento faktor před
> radiometrickým řetězcem opět vydělí, ale pixely, které režim „Sum“ zatlačil až na strop snímače, jsou oříznuty a nelze je
> obnovit. Průměrné binning nepřináší nic, co by průměrné binning nepřinášelo – index je invariantní vůči měřítku a skaláry radiance
> jsou kalibrovány v měřítku průměrného binningu. Pokud je kamera dodána již nastavená na součtové binning (nastavená
> jiným nástrojem nebo obnovením továrního nastavení), Chloros to detekuje z zpětného čtení uzlu a
> kompenzuje to; spuštěním `lattice resolution --binning N` se to vrátí zpět na „Average“. Příznak 1.2.0
> `--binning-mode` je stále akceptován, takže starší skripty běží, ale vypíše pouze varování:
> binning zůstává na hodnotě „Average“.

> **Změna binningu, formátu pixelů a spouště na kameře se neuloží.** Na kameře, kde
> backend zůstává otevřený a streamuje (`cam-connect`, grafické rozhraní), se `lattice resolution --binning`
> nemá žádný účinek (vypíše, že binning stále platí, `Binning: 1x1`), a
> `lattice format --set` a `lattice trigger --mode` jsou kamerou odmítnuty
> (`SC_ERR_ERROR -1001`). Nejprve je uvolněte (`lattice cam-disconnect -s <SN>`) nebo je nastavte
> při připojení (`cam-connect --pixel-format …`, `array-connect --binning …`).

##### Čtení expozice a zesílení v automatickém režimu

Zatímco v kameře běží `ExposureAuto` (`GainAuto`), její uzel `ExposureTime` (`Gain`) uchovává **poslední zapsanou hodnotu**, nikoli aktuální stav kamery. U kamery, jejíž backend zůstává otevřený (`cam-connect`, `array-connect`, grafické rozhraní), `lattice configure --get`, výchozí zobrazení `lattice configure` a `lattice exposure` (bez příznaků) proto vypisují hodnotu uzlu a pod ní poznámku s nejnovějším snímkem kamery:

```

ExposureTime = 2000.448
  note: ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448); the camera's newest frame was at 4900 us.
```

Hodnotu snímku přečtěte z poznámky; hodnota uzlu představuje pouze poslední zápis. Přes HTTP stejná odpověď `POST /api/camera/<serial>/read-nodes` ponechává `nodes` beze změny a přidává `frame_values` (`{"ExposureTime": <us>, "Gain": <dB>}`, nejnovější snímek) a `notes` (výše uvedená věta). Oba jsou přítomny pouze tehdy, pokud automatický režim není `Off` a je znám snímek; kamera otevřená pouze pro čtení nemá žádný snímek a nezíská ani jeden z nich. `lattice gain` (bez příznaků) vypíše pouze uzel — pro poznámku použijte `lattice configure --get Gain`.

##### Expozice na členu pole

Zatímco pole vysílá, každý člen běží s **`AcquisitionFrameRateEnable`**, a to při snímkové frekvenci mírně vyšší než spouštěcí frekvence pole: její perioda je spouštěcí perioda minus 15 % (minimálně 3 ms), nebo vlastní maximální frekvence kamery, pokud se spouštěcí frekvence již blíží této hodnotě. `configure --get` na prvku proto při tomto nastavení čte hodnoty `AcquisitionFrameRateEnable` jako `True` a `AcquisitionFrameRate`; povolení se vypne, jakmile se pole zastaví.**Na členu pole nezapisujte tyto dva uzly** — pole je udržuje v synchronizaci se svým spouštěčem a přepisuje je.

Toto nastavení je také horní hranicí expozice člena: přibližně 85 ms při spouštění 10 fps (perioda 100 ms), při **libovolné oblasti zájmu (ROI) nebo binningu**. Před verzí 1.3.0 malá oblast zájmu (ROI) omezovala expozici člena pole na periodu volného běhu snímače (4,9 ms při rozlišení 768 × 304), bez ohledu na to, jak pomalu se pole spouštělo.**Ručně zadaná hodnota `ExposureTime` delší než horní limit zpomalí spouštění tak, aby se této hodnotě přizpůsobilo**, pokud je zapsána pomocí `lattice exposure --set` nebo `configure --set` (nebo z grafického uživatelského rozhraní). Pole nejprve otevře horní limit daného člena pro danou hodnotu a nechá pomalejší frekvenci běžet několik spouštěcích period, takže zápis může trvat o několik sekund déle; spoušť se poté spouští ne rychleji, než to expozice dovoluje. U pole s automatickou frekvencí se původní frekvence obnoví, jakmile se každý člen, jehož manuální expozice ji zpomalila, vrátí k automatické expozici. Hodnota, kterou kamera sama nemůže přijmout, je i nadále odmítnuta, přičemž v chybové zprávě je uveden rozsah, který kamera akceptuje, a automatická expozice člena je obnovena, pokud byla vypnuta pouze pro tento zápis. Expozice podle receptu `apply` spoušť nezpomalí: je udržována na horní hranici. Chloros nikdy nezkracuje manuální expozici, aby se přizpůsobil rychlejšímu spouštění; pevně nastavená frekvence (`array-connect --target-fps F`), která je rychlejší, než umožňuje expozice, ponechává tomuto členu horní limit dostatečně dlouhý pro ni, a tento člen přijímá spouštění ne rychleji, než umožňuje jeho expozice.

#### Kalibrace a senzory

| Podpříkaz | Účel |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibrace pomocí reflexního terče. Prohledává se až `--attempts` snímků (výchozí hodnota `30`); neúplný rámec (ztráta paketu GVSP) stojí jeden pokus, nikoli celý běh. Dojde k selhání při hodnotě `no target detected in N attempts`, nebo při hodnotě `no complete frame in N attempts: <last error>`, pokud nedorazil žádný celý rámec. |
| `lattice dls {--connect \| --spectrum \| --irradiance} --mac MAC [--filter NAME] [--camera SERIAL] [--json]` | Spektrum a amplitudy pásem zařízení DAQ-M, načtené přes Bluetooth (BLE) prostřednictvím backendu. `--mac` je vlastní BLE MAC senzoru a je vyžadován u kterékoli ze tří akcí: neexistuje žádná výchozí hodnota, protože každá jednotka má svou vlastní (`daq pool-discover --only ble` je vypisuje); bez něj se příkaz ukončí `1`. Pokud není zadána žádná akce, vypíše se nápověda. `--camera SERIAL` integruje vypsané hodnoty pásem pomocí vlastní naměřené spektrální odezvy dané kamery namísto křivky filtrového modelu — u RGB kamer, pokud je kalibrační balíček kamery načten (připojen) nebo uložen v mezipaměti tohoto počítače. V záhlaví je uvedeno, čí křivka byla použita: `Band magnitudes (camera <SERIAL>):`, nebo `Band magnitudes (model <filter>):`, pokud kamera žádnou nemá (nejedná se o RGB kameru, nebo není načten ani uložen v mezipaměti) a místo toho byla použita standardní křivka. Přes HTTP přijme `POST /api/dls/cli-read` vstup `camera_serial` a vrátí `band_magnitudes_source`. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Použije korekci vinětace na stávající snímky, přičemž každý snímek má vlastní ploché pole objektivu (z jeho EXIF Model). `--lens-model` se aplikuje na snímky, jejichž Model neuvádí žádný fotoaparát; snímek, jehož Model je s tím v rozporu, je přeskočen. |

#### Více fotoaparátů (dočasné relace)

Dočasný příkaz otevře kamery na jedno spuštění a po jeho skončení je zavře: `multi-capture` bez připojeného trvalého pole, `multi-stream` a `multi-test` zde, a `align-calibrate`, `align-apply`, `align-stream` a `index --live` níže. Pro všechny z nich platí následující pravidla:

- **`--serials SN1,SN2,…` vybírá kamery.** Otevřou se pouze tyto kamery a první z nich spustí spouštěč (v případě `align-calibrate` a `--reference`, jsou-li zadány). Sériové zařízení, které není v síti, způsobí selhání běhu ještě před otevřením jakékoli kamery. Bez `--serials` se otevřou všechny kamery v síti. Kamera, kterou již Chloros drží (GUI, `cam-connect`, `array-connect`), odmítne spuštění, ať už ji `--serials` jmenuje či nikoli: sdílí síť, pro kterou je spuštění určeno. (`index --live` bez `--serials` místo toho otevře kamery povolené v jeho profilu: viz [Index / Vegetation Maths](#index--vegetation-maths).)
- **Kamery sdílejí síť stejným způsobem, jakým ji sdílí `array-connect`.** Každá kamera je přizpůsobena svému podílu na tom, co přenáší hostitelské připojení (zpoždění mezi pakety, nikoli limit propustnosti kamery, jehož minimální hodnotu 31,25 MB/s již čtyři kamery na 1 GbE překračují), takže čtyři kamery běží na 1 Gbps spojení. Pokud kamery nemohou všechny odesílat najednou nejpomalejším tempem, kterého dosahují, spouštějí se jedna po druhé, každá s vlastním připojením, a příkaz vypíše `Serial emit: the wire cannot carry these cameras at once, so each is triggered <ms> ms after the one before; their frames are not exposed together`. Pokud se ve chvíli nevejde ani jedna kamera, je spuštění odmítnuto ještě předtím, než začne jakákoli kamera streamovat: `array over-subscribes the wire: <n> cameras demand <x> MB/s but the collision-safe ceiling is <y> MB/s. …`, HTTP 409, ukončení `1`.
- **Všechny kamery musí odesílat data, jinak operace selže.** Kamera, která neodesílá žádný snímek, představuje chybu: příkaz vypíše `Error: … no frame from <n> of <total> cameras: SN:<serial> (<why>), …`, kde `<why>` označuje chybu načtení dané kamery, následně uvádí příčiny a ukončí se `1`. Snímek, který dorazil se ztrátou paketů GVSP (`IncompleteBufferError: GVSP packet loss …`), znamená, že síť nepřenáší celkovou šířku pásma kamer (`Every missing frame arrived with GVSP packet loss: …`): zkontrolujte rychlost připojení hostitele a uplinku přepínače (`lattice network`), použijte jumbo rámce, nebo spusťte méně kamer pomocí `--serials`. Kamera, která nic neodeslala, obvykle nedostává spouštěcí signál od hlavní kamery: zkontrolujte její synchronizační kabel M8 a poté spusťte `lattice multi-detect`. Co každý příkaz provede se skupinou rámců, ve které chybí kamera, je uvedeno v příslušném řádku.
- **Každá kamera se vrátí do stavu, v jakém byla nalezena.** Po skončení běhu se každá kamera vrátí do stavu, v jakém byla při spuštění běhu: `TriggerMode`, `TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, a `LineMode` / `LineSource` na `Line2` a `Line3` (a odrušovač spouštěcího vstupu pole poskytuje linku podřízeného zařízení, `LineFilterSelector` / `LineFilterWidth`); jeho `PtpEnable` (a s ním nastavené `PtpSlaveOnly` / `PtpDomainNumber`); `ExposureAuto` / `GainAuto` (a `ExposureTime` / `Gain`, kde byla nastavena ruční hodnota); `TargetBrightness`, `ExposureAutoLimitAuto` / `ExposureAutoUpperLimit` / `ExposureAutoLowerLimit`, `GainAutoUpperLimit`, `ExposureAutoDamping`; `BalanceWhiteAuto` (a každý záznam `BalanceRatio`, kde bylo vyvážení bílé nastaveno ručně); `BlackLevelAuto` / `BlackLevel`; `PixelFormat`, binning, `Width` / `Height` / `OffsetX` / `OffsetY`; `GammaEnable` (a `Gamma`, kde byla zapnutá gama korekce), `LUTEnable`, `ColorTransformationEnable`; `ReverseX` / `ReverseY`; `UserSetSelector`; velikost paketu (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`); tempo, `DeviceLinkThroughputLimitMode` (a `DeviceLinkThroughputLimit`, kde byl režim `On`), zpoždění mezimezi pakety `GevSCPD` a časový posun spuštění `GevSCFTD`, které každá kamera poskytuje; data bloků, která každý proud aktivuje (`ChunkModeActive`, každý `ChunkEnable` pod svým `ChunkSelector`); `AcquisitionFrameRateEnable` (a `AcquisitionFrameRate`, kde bylo povolení zapnuto — při jeho vypnutí je rychlost pouze pro čtení a čte maximum, které tempo umožňuje, takže se nejedná o nastavení). Zápis se provádí pouze u odlišných hodnot: nejprve se zastaví kamera, poté se zaznamená frekvence a snímková frekvence před expozicí, která vyžaduje jejich snímkovou periodu (kamera omezuje hodnotu `ExposureTime` na jednu snímkovou periodu). S `--keep-settings` se propojení spouště a synchronizační linky vrátí zpět, kamera běží volně a zbytek se uloží do vlastní paměti kamery podle nastavení běhu; `--clear-settings` tam nastaví tovární nastavení.
- **Expozice se nastavuje na základě scény před zachycením prvního snímku.** Sériové snímání pořizuje synchronizované skupiny snímků a vyřazuje je, zatímco na základě těchto snímků nastavuje expozici a zesílení každé kamery. Cílem je dosáhnout průměrné jasnosti na cíli kamery, přičemž nesmí být oříznuto více než 1 % snímku; scéna s jasnými světly nedosáhne tohoto cíle. To obvykle trvá 2–6 skupin snímků a maximálně 5 s.
  - Každá kamera míří na cíl, od kterého ji spustí připojené pole, a to podle svého filtru: TargetBrightness **40**u multispektrální kamery (RGN, OCN, NGB) a**60** u RGB kamery. To je polovina cílové expozice, kterou udává hodnota 80 (tři čtvrtiny u RGB), takže je méně pravděpodobné, že dojde k oříznutí jasného odrazového panelu. `--brightness N` (0–255) místo toho nastavuje cíl pro každou kameru (`multi-test` nemá `--brightness`: jeho automatická expozice je vypnutá). Názvy záznamů v backendu uvádějí cílovou hodnotu každé kamery: `[transient-ae] AE target per camera (filter default): SN:<serial> <target>, …`.
  - Expozice má přednost, a to až do limitu automatické expozice dané kamery. Zesílení pokrývá pouze to, co zbývá.
  - Stanovená expozice a zesílení se zachovávají po celou dobu běhu, takže každý snímek kamery `multi-stream` je pořízen se stejnou expozicí. Snímání, které musí sledovat měnící se světelné podmínky, patří do propojeného pole (`array-connect`), jehož automatická expozice se neustále přizpůsobuje.
  - Snímky `--exposure` a `--gain` zůstávají přesně tak, jak byly zadány. Jsou-li zadány obě, žádný snímek není vyřazen.
  - Po skončení běhu se u každé kamery znovu zapne automatická expozice.
  - Protokol backendu vykazuje výsledek: `[array-ae] exposure solved in <s> s (<n> warm-up groups): SN:<serial> <ms> ms <dB> dB mean <m>/<target>, …` nebo `exposure NOT solved after …` s uvedením důvodu a `(not settled)` u každé kamery, u které k tomu nedošlo. V takovém případě se snímky i nadále pořizují.
  - `CHLOROS_TRANSIENT_AE_SETTLE_S` mění 5sekundový limit; `0` tuto funkci vypíná (viz [Proměnné prostředí](#environment-variables)).

**Dokud Chloros drží jakoukoli kameru** (GUI, `cam-connect` nebo připojené pole), `multi-stream`, `multi-test`, `multi-detect`, `align-apply`, `align-stream` a `index --live` se odmítnou spustit, protože otevírají všechny kamery v síti a druhé otevření zablokované kamery může způsobit pád backendu. (`index --live` odmítne spuštění, pokud je kamera, kterou by měl otevřít, zablokovaná: jedná se o kameru `--serials` nebo kamery z jeho profilu a jakoukoli kameru pro profil, který žádnou kameru neuvádí.) Backend odpoví HTTP 409 `CAMERA_HELD`, přičemž uvede každou zablokovanou kameru a příkazy k uvolnění (`chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`) a příkaz se ukončí (`1`). Je-li připojeno pole, `array-capture --continuous` jej zaznamená namísto `multi-stream` a `array-capture` (ve výchozím nastavení zarovnané) uloží zarovnanou sadu na místo `align-apply`; `multi-capture` a `align-calibrate` používají připojené pole, které obsahuje jejich kamery, místo aby je otevíraly.

| Podpříkaz | Účel |
| --- | --- |
| `lattice multi-info [--serials SN1,SN2,…]` | Vypíše všechny kamery s synchronizačními rolemi; s `--serials` pouze ty, první jako `MASTER`. |
| `lattice multi-capture [--serials SN1,SN2,…] [--format FMT] [--jpeg-quality N] [--processing LEVEL] [--brightness N] [--no-daq] [--daq-tolerance-ms MS]` | Jeden synchronizovaný snímek z každé kamery. **Ve výchozím nastavení**ukládá všechny typy exportu, pokud je připojeno trvalé pole, a hodnotu DAQ, která jej načítá ([Hodnota DAQ, kterou zachycení ukládá](#the-daq-reading-a-capture-saves)); dočasná záložní varianta bez pole je**pouze debayered** (pro zbytek nejprve spusťte `array-connect`). `--brightness` se vztahuje pouze na náhradní režim: připojené pole si zachovává vlastní automatickou expozici. Náhradní režim neukládá `.daq` (`--no-daq` / `--daq-tolerance-ms` zde nemají žádný účinek) a `multi-capture` nepřijímá `--force-daq`. V náhradním režimu kamera, která nedodá žádný snímek, způsobí selhání snímání (`Error: multi-capture failed: no frame from …`, ukončení `1`) a žádný soubor z této skupiny snímků se neuloží. |
| `lattice multi-stream [--serials SN1,SN2,…] [--fps F] [--count N] [--format FMT] [--jpeg-quality N] [--brightness N]` | Synchronizované snímky ze streamu (dočasné), `--fps` výchozí `3`, dokud nedojde ke stisknutí Ctrl+C nebo k vytvoření skupin `--count`. Skupina snímků, pro kterou kamera nic nedodala, se nezapíše, a tři po sobě ukončí stream. Pokud byla nějaká skupina přeskočena, běh končí vypsáním `Incomplete: <n> frame group(s) not written, a camera delivered nothing (SN:<serial> x<count>, …)`, plus `Error: the stream stopped: …` s uvedením názvu kamery, u které stream skončil, a ukončí se `1`X. Program se sám zastaví, když se disk zaplní, vypíše `Failed:` / `Disk full:` a ukončí se `1`; jakýkoli soubor, který se nepodařilo zapsat, také ukončí program `1`. |
| `lattice multi-test [--serials SN1,SN2,…] [--count N] [--ptp-settle SECONDS]` | Test synchronizačního časování GPIO (přechodný), `--count` výchozí spouštěče `10`. Vypíše, jak je síť sdílena (`Wire: each camera paced to <x> MB/s (<n> on a <y> MB/s ceiling, <link> Mbps link)` a řádek `Serial emit:`, když jsou kamery spouštěny jedna po druhé, což se pak projeví v časových razítkách), jeden řádek `Sync group: master SN:<serial> -> SN:<serial>, …` pro každou synchronizační skupinu (`(no slaves: not synced)` pro kameru bez synchronizačního kabelu, která se spouští samostatně), poté jeden řádek na jedno spuštění: `[  0] SN:<serial>=<timestamp> \| …`, přičemž `SN:<serial>=(no frame)` označuje kameru, která pro dané spuštění nic neodeslala. Poté `Offsets from master SN:<serial> (<PTP / device (PTP off)> clock, <n> trigger(s)):` a jeden řádek `SN:<serial>: mean <±x> us, jitter <y> us` pro každou kameru, každá kamera vůči masteru v rámci jednoho taktu: spouštění, u něhož se v časových razítkách mísí zařízeníové hodiny kamer a PTP (PTP vypnutý nebo stále se synchronizuje), přičemž je počítá `Note:`. Jakýkoli vynechaný snímek znamená neúspěch testu (`Error: multi-test failed: <k> of <n> frame groups incomplete (SN:<serial> missed <k>); …`, ukončení `1`); tři neúplné spouště za sebou test předčasně ukončí. Test také selže (ukončení `1`), pokud méně než dva spouštěcí signály zajistí, že všechny kamery budou synchronizovány na jeden takt (`… the clocks cannot be compared`), nebo pokud se hodiny nikdy neshodnou: Hodiny PTP se liší od hlavního hodinového signálu o více než 1 ms (plus případné zpoždění na sériové úrovni) (`… PTP is not locked`), nebo se zpoždění hodin jednotlivých zařízení mezi spouštěcími signály posune o více než tuto hodnotu. Spustí PTP grandmaster, pokud ještě neběží, zapne PTP u každé kamery (stejně jako to dělá `array-connect`, bez ohledu na konfiguraci sítě) a před prvním spouštěčem počká, až se PTP hodiny každé kamery synchronizují s grandmasterem a ustálí se: `PtpStatus` Slave na jednom grandmasteru, s odchylkou do 100 us a stabilní po dobu 3 s, a každá kamera takto synchronizovaná, bez přerušení, po dobu 120 s — přibližně 2 minuty od zapnutí. Hodiny, které se právě synchronizovaly, vykazují odchylku až ~200 us po dobu asi 2 minut poté, co se načte stav „Slave“, a vlastní `PtpOffsetFromMaster` kamery to nezobrazuje; po 120 s se kamery dostanou do odchylky ~10 us od ustálených hodin. Kamera je stále synchronizována od předchozího testu, při kterém zůstalo PTP zapnuté, jak to tento backend zaznamenal, počítá od tohoto synchronizace, takže test prováděný bezprostředně po předchozím nemusí znovu čekat; kamera, kterou test zapne, nebo kamera nalezená se zapnutým PTP, u které tento backend nezaznamenal synchronizaci (zapnutá jinde nebo před restartem backendu), čeká svých 120 s, a v poli obou kamer, které se právě synchronizovaly, stále čekají na svou dobu. Kamera, která opustí režim Slave, překročí 100 us nebo během čekání sleduje jiného grandmastera po dobu 3 odečtů v řadě (1,5 s), začne počítat znovu; jeden nebo dva takové odečty jsou zaznamenány (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) a odpočítávání pokračuje; ID grandmastera, které nelze přečíst, je považováno za neznámé, nikoli za změnu. Pokud se počítání všech kamer restartuje najednou (během jedné sekundy), znamená to, že se posouvá čas grandmastera, nikoli kamer: protokol backendu uvádí `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` jej uvádí v záznamu `grandmaster_steps` a čekání, jehož čas poté vyprší, začíná svou chybou s kódem `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Limit se pro ni neprodlužuje. `--ptp-settle SECONDS` nastavuje dobu ustálení pro daný běh (0–600; `0` pro čekání bez ustálení). Čekání je omezeno na dobu ustálení plus 90 s (výchozí hodnota 210 s, minimálně 90 s) a příkaz čeká na backend 240 s plus dobu ustálení (výchozí hodnota 360 s). Vypíše `PTP: locked after <s> s (each camera locked <settle> s, already locked: SN:<serial>, …; from the grandmaster: SN:<serial> <±x> us, …)` a protokol backendu obsahuje každých 5 s řádek se stavem a offsetem každé kamery, jak dlouho je uzamčena a jak dlouho jí ještě zbývá: `[array-sync] PTP at <s> s: SN:<serial> Slave <±x> us for <s> s (<s> s left), … -- <s> s left, the wait gives up at <s> s`. Každá kamera se zapne s vypnutým PTP; od té chvíle se hodiny synchronizují přibližně za 10 s. Poté je každá kamera vrácena do stavu, v jakém ji test našel – její PTP, automatická expozice a zesílení, které test během běhu udržuje vypnuté, a zbytek (pokud není `--keep-settings`) — takže kamera, u které zůstalo PTP zapnuté (`lattice configure -s <serial> --set PtpEnable=True`, nebo běh s `--keep-settings`), zůstává synchronizovaná a jakmile tento backend zjistí, že je synchronizovaná po dobu ustálení, pozdější test na ni nečeká. Pokud se hodiny během čekání neusadí, dojde k selhání ještě před spuštěním (`PTP: NOT locked …`, ukončení `1`) a uvede, co je třeba udělat: kamera, která se nikdy nezamkla (`PtpStatus Listening`), znamená, že grandmaster k ní nedosahuje (`time-sync status`); ta, která neustále vypadávala, je označena dobou, po kterou byla zamčena (`… had not been locked for the 120 s settle time … (SN:<serial> locked <s> s, <s> s to go)`); hodiny se stále synchronizují: zkontrolujte `time-sync status`, a aby byly před testem synchronizovány, zapněte PTP a nechte jej zapnutý (`lattice configure -s <serial> --set PtpEnable=True` pro každou kameru, nebo spusťte test s `--keep-settings`; test obnoví zjištěné nastavení, takže zůstane zapnutý), poté test spusťte. |
| `lattice multi-detect [--line LINE] [--json]` | Najděte kamery na synchronizační lince M8 a výchozího mastera: kameru RGB (model končící na `-FRGB`), pokud je připojena kabelem, jinak první kameru připojenou kabelem podle sériového čísla. Kabely M8 se sbíhají v jedné rozbočce, takže **hlavní kamerou může být jakákoli připojená kamera** — vytištěné označení `MASTER` je pouze výchozí hodnota, nikoli skutečný stav zapojení. Každá kamera postupně řídí linku a ostatní kamery jsou kontrolovány na přítomnost jejího impulsu; kamera, která již byla nalezena v zapojené skupině, není znovu testována, takže prozkoumání plně zapojené sady trvá jedno kolo. Pokud není k dispozici `--line`, prozkoumá se `Line2`, poté `Line3`. Vypíše `MASTER: SN:…`, po jednom `SLAVE: SN:…` pro každou další zapojenou kameru a `(probed on <line>)`; `--json` přidá celkový výsledek (`master`, `slaves`, `no_cable` — kamery, ke kterým impuls nedorazil — `line` a pro každou kameru `results`). Po dokončení se každé kameře vrátí původní nastavení spouště a synchronizační linky (stejné uzly jako výše). |

#### Zarovnání

| Podpříkaz | Účel |
| --- | --- |
| `lattice align-calibrate [--serials SN1,SN2,…] [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N] [--brightness N]` — plus ovládací prvky detektoru/porovnávače `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, RANSAC `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, kombinace více snímků `[--averaging mean\|median\|inlier_weighted]`, geometrická omezení `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, prostorová omezení `[--roi X0,Y0,X1,Y1] [--mask PATH]` aslave `[--per-cam-override SN:KEY=VALUE]` (opakovatelné) | Vypočítá profil zarovnání z živých kamer. Výchozí hodnoty: `--method orb`, `--model rigid`, `--frames 3`, `--save alignment_profile.json` (ve složce, ze které se spouští), `--max-features 2000`, `--ratio-threshold 0.75`, `--matcher bf`, `--knn-k 2`, `--ransac-threshold-px 3.0`, `--ransac-iters 2000`, `--ransac-confidence 0.99`, `--averaging mean`. (`project align calibrate` má vlastní výchozí nastavení; viz [jeho tabulku](#project-align-calibrate-options).) `--prefilter` má jako výchozí nastavení `gradient` (mapa hran; odpovídá zarovnávači v grafickém rozhraní/pole — hrany přetrvávají napříč spektrálními pásmy). `--matcher flann` se vyplatí při počtu prvků nad ~5000; `--averaging median` je odolný vůči jednomu špatnému snímku, `inlier_weighted` váží podle počtu shod; `--lock-scale` promítá na nejbližší rotaci (bez měřítka), `--lock-axis` vynuluje jednu složku posunu; `--mask` se vztahuje na všechny kamery (pro nastavení pro jednotlivé kamery použijte `--per-cam-override`, např. `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` odmítne uložit kalibraci, jejíž RMS reprojekce překračuje limit. `--method manual` čerpá své body z `--points PATH`: JSONový objekt `{"<serial>": [[x, y], ...], ...}`, který udává souřadnice pixelů stejných prvků pro každou kameru, včetně referenční kamery (minimálně 2 na kameru); každý z objektů `--frames` je vyřešen z těchto bodů. Pokud kamery tvoří propojené pole, kalibrace využívá živé snímky tohoto pole a profil se zároveň stává zarovnáním pole pro jeho záznamy. V opačném případě se otevře [dočasné pole](#multi-camera-transient-sessions): kamera, která nedodává žádný snímek v žádném z `--frames`, neprojde kalibrací (ukončí `1` s uvedením jejího názvu) a profil se neuloží. |
| `lattice align-apply --profile PATH [--serials SN1,SN2,…] [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N] [--brightness N]` | Zachytí jeden zarovnaný vícepásmový snímek. `--bit-depth` se ve výchozím nastavení přizpůsobí kameře; `--no-crop` zachovává celý snímek (vyplní černou barvou); `--interpolation` (výchozí `linear`) a `--border-mode`/`--border-value` (výchozí `constant`/0) řídí deformaci CPU — cesta GPU je v každém případě bilineární. Otevře [dočasné pole](#multi-camera-transient-sessions): kamera, která nedodává žádný snímek, způsobí selhání snímání (ukončení `1` s uvedením jejího názvu) a nic se nezapíše. |
| `lattice align-stream --profile PATH [--serials SN1,SN2,…] [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N] [--brightness N]` | Streamově zarovnané vícepásmové snímky (stejné ovládací prvky pro warp jako u `align-apply`), výchozí nastavení `--fps` a `3`. Otevře [dočasné pole](#multi-camera-transient-sessions): skupina snímků, pro kterou kamera nic nedodala, se neuloží a tři po sobě ukončí stream. Pokud byla nějaká skupina přeskočena, běh skončí vypsáním `Incomplete: <n> frame group(s) not saved, a camera delivered nothing (SN:<serial> x<count>, …)` a `Error: …`, a ukončí se kódem `1`. Průběh, který skončí z jakékoli jiné chyby, rovněž vypíše kódy `Error: …` a ukončí se kódem `1`. |
| `lattice align-info --profile PATH [--json]` | Zobrazit podrobnosti profilu. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Změnit pořadí vrstev. |

#### Index / Matematika vegetace

```bash
# Offline: compute NDVI from an aligned multi-band TIFF (an RGN camera's bands)
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: open the cameras, capture one aligned set with a profile made by
# `lattice align-calibrate`, and compute the index from it
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --save-multiband -o output/
```

**`--channel` přiřazuje symboly předvolby k pásmům** (NDVI: `red`, `nir`), vždy jeden `--channel SYMBOL=BAND`.

- **`--live`: pouze pro symbol, na který stack nedokáže odpovědět.** Stejně jako u `--input`, symbol předvolby, který vynecháte, převezme jedno pásmo ze zásobníku (pásma povolených kamer profilu), jehož název tímto symbolem začíná. Toto se kontroluje před otevřením jakékoli kamery: symbol, na který neodpovídá žádné pásmo nebo několik pásem, je poté odmítnut, přičemž se uvedou jejich názvy, a program opustí `1`. Vlastní `--formula` vyžaduje pro každý symbol `--channel`.
- **Produkty `--live` uvádějí, komu patří.** `index.tif` a `index_LUT.png` obsahují údaje o kameře ve stacku (značka, model, sériové číslo), čas pořízení snímku (expozice referenční kamery, UTC) a objektiv, přičemž oba uvádějí index jako své jediné pásmo (`XMP-Camera:BandName` `NDVI`; barevný soubor PNG uvádí index, nikoli své barevné kanály), stejně jako to dělá produkt `--input`. Index složený z pásem několika kamer místo toho pojmenovává každé z nich a jeho `Model` je `LATTICE` — viz [Soubory vytvořené z několika kamer](#files-made-from-several-cameras).
- **`--input`: pouze pro symbol, na který soubor nedokáže odpovědět.** Symbol, který vynecháte, převezme jedno pásmo souboru, jehož název tímto symbolem začíná: `red` převezme `Red_660` nebo `Red_660_RGN`, `nir` převezme `NIR_850`. Názvy pocházejí ze samotného souboru: z vlastního seznamu pásem zásobníku, nebo, v případě snímku radiance / odrazivosti Chloros nebo produktu `process` jeho značky `Chloros:Band{R,G,B}Name` / `XMP-Camera:BandName`. `lattice index --input <reflectance capture> --preset NDVI` tedy nepotřebuje `--channel`.
- **Dvě pásma odpovídající jednomu symbolu se mezi sebou nerozlišují** (stoh z 9 kamer má dvě NIR pásma): pojmenujte jej. Symbol, kterému nic neodpovídá, je odmítnut pomocí `channel_map missing entries`, ukončete `1`.**Názvy pásem ve stacku** nesou název filtru kamery: pásma kamery M3C jsou `<band>_<filter>` — `NIR_850_RGN`, `Green_550_RGN`, `Red_660_RGN` u kamery typu RGN; `NIR_808_OCN`, `Cyan_490_OCN`, `Orange_615_OCN` na kameru typu OCN; `Blue_475_RGB`, `Green_550_RGB`, `Red_625_RGB` na kameru typu RGB; `Blue_475_NGB`, `Green_550_NGB`, `NIR_850_NGB` na NGB — aje `Mono_<filter>` (`Mono_660`, `Mono_850`). `lattice align-info --profile PATH` uvádí seznam pásem profilu.

Úplná sada příznaků: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (opakovatelné), `--capture-level raw|debayered|radiance|reflectance|unknown` (přepíše úroveň snímání zaznamenanou ve zdrojovém souboru TIFF; výchozí: čteno z metadat souboru TIFF, v opačném případě se použije hodnota `Chloros:ProcessingLevel` ze souboru Chloros, takže při snímání odrazivosti se vykazuje hodnota `reflectance`), `--output PATH` / `-o` (offline: cesta; s příponou obrázku jsou soubory pojmenovány podle kořene — `-o ndvi_FRGN.png` zapíše `ndvi_FRGN.tif` a `ndvi_FRGN_LUT.png` — a bez ní se jedná o předponu, `-o ndvi` -> `ndvi.tif`, `ndvi_LUT.png`; výchozí hodnota vedle vstupu, jako `<input name>_<preset>`; s `--live` a bez `-o`, složka `output/` pod složkou, ze které spouštíte příkaz), `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. U souboru `--live` se také použijí ovládací prvky pro deformaci zarovnání: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`, `--brightness N` (cíl automatické expozice kamer, viz pravidla pro přechodné stavy), `--serials SN1,SN2,…`. `--live` otevře [pole přechodných stavů](#multi-camera-transient-sessions): kamera, která nedodá žádný snímek, způsobí selhání běhu (ukončí `1` a pojmenuje ji), a nezapíše se žádný zásobník ani index. Otevře `--serials` (první spustí spouštěč), jinak kamery profilu, u nichž je povolena položka `layer_order`, nejprve její reference — reference i s deaktivovanými pásmy, protože ostatní jsou k ní zarovnány a spouští spouštěč. Kamera, jejíž pásma jsou deaktivována, se nikdy nekalibruje, takže RGB kamera deaktivovaná v profilu již nevyvolá selhání indexu odrazivosti (`no-absolute-radiance-scale`). Profil, který nejmenuje žádné kamery, otevře všechny kamery v síti, stejně jako dříve.

Každý formulář běží na backendu, takže ten musí být spuštěn: spusťte jej z grafického rozhraní Chloros nebo pomocí příkazu, který CLI vypíše, když se k němu nemůže připojit. V režimu offline backend čte `--input` a sám zapisuje TIFF s plovoucí desetinnou čárkou a kolorovaný PNG, takže relativní `--input` nebo `-o` se vyřeší podle složky, ze které příkaz spouštíte (viz [Konvence](#conventions)). Oba offline výstupy obsahují záznam o fotoaparátu ze vstupního souboru, stejně jako export indexu ve formátu `process`: značka / model / sériové číslo, GPS, údaje o objektivu, čas pořízení (vlastní čas vstupního souboru; pokud EXIF neuvádí žádný, pak okamžik expozice zaznamenaný fotoaparátem nebo časové razítko v názvu souboru, v UTC) a identitu kalibrace MAPIR — nikdy kalibraci raw-DN, která se na index nevztahuje — plus `Chloros:ProcessingLevel=index` a výraz indexu; plovoucí číslo TIFF označuje jedno pásmo, předvolbu. `--list-presets` a `--list-gradients` vypisují tabulky předvoleb a přechodů backendu. Vzorec, který backend odmítne, nebo pásmo `--channel`, které soubor TIFF neobsahuje, ukončí `1` s uvedením důvodu backendu.

> **U symbolů `--channel` se rozlišují velká a malá písmena.** Strana symbolu se musí přesně shodovat s názvy kanálů předvolby (předvolby používají malá písmena, např. NDVI = `red`,`nir` — zkontrolujte `--list-presets`)a strana s pásmem se musí shodovat s názvem pásma v zarovnaném stohu (nebo být indexem pásma počínaje od 0 v offline režimu). `--channel red=Red_660_RGN --channel nir=NIR_850_RGN` funguje; `--channel RED=660` selže s chybou `channel_map missing entries` .

#### `lattice index` — masky pro vícepásmové sady snímků

Při použití `--capture-level radiance` nebo `reflectance` může 3×3 oddělení jednotlivých kanálů u každé kamery omezit hodnotu pásma na 0 (viz [Maska omezující oddělení](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on)). Vícepásmový stack obsahuje pásma z několika kamer a každá kamera omezuje své vlastní, takže `--live --save-multiband` zapíše **jednu masku pro každou zdrojovou kameru** vedle stacku:

```
output/
├── index_multiband.tif
└── clamp_masks/
    ├── index_multiband_SN<serial A>-<filter A>_clampmask.tif
    └── index_multiband_SN<serial B>-<filter B>_clampmask.tif
```

- Každá maska je jednopásmová `uint8` na pixelové mřížce **stohu** (po provedení stejného zarovnání, deformace a oříznutí), s vlastními bity dané kamery: bit 0 = pásmo jejího kanálu R, bit 1 = G, bit 2 = B. Pásmo zásobníku převzaté z kanálu *c* kamery je omezováno tam, kde je nastaven bit *c* dané kamery (R = 0, G = 1, B = 2).
- Vložený původ každého pásma pojmenovává masku příslušné kamery (`unmix_clamp_mask`, relativně ke složce sady, nebo `none`) a bit, který se má číst (`unmix_clamp_mask_bit`).
- Kamera, u které funkce unmix nic neomezila, nedostane žádný soubor; u monochromatické nebo RGB kamery se funkce unmix nespustí a také nedostane žádný soubor. CLI vypíše každou masku pod stackem (`clamp mask SN…: …`); odpověď je uvádí jako `clamp_masks`.
- Indexový soubor TIFF/PNG ze stejného běhu se vypočítává ze stejného zásobníku na stejné mřížce, takže jej masky také popisují. `--capture-level raw` neprovádí oddělení barev a nezapisuje žádnou masku.
- Sada a index se nacházejí přímo v `--output`, takže masky jsou uloženy v `--output` vlastním `clamp_masks/` — nikdy nad složkou, kterou jste pojmenovali. Tato složka obsahuje různé typy výstupů (stoh, index, kolorovaný render): vyberte z ní pouze potřebné soubory, nepředávejte celou složku fotogrammetrickému softwaru.

#### Trvalá připojení (Smart-Prep, ekvivalent toku v grafickém rozhraní)

Tyto příkazy udržují kamery otevřené v backendovém fondu napříč jednotlivými spuštěními příkazového řádku (CLI).

| Podpříkaz | Účel |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Přidá jednu kameru do fondu (jedna kamera, bez pole). [Společné volby](#common-options-shared-by-most-subcommands) `--exposure`, `--gain`, `--preset`, `--pixel-format`, `--width` / `--height` a `--packet-size` (`standard` = 1500, `jumbo` = 9000, nebo bajty; sondy `auto`) se aplikují při připojení (přepínače `--exposure` nebo `--gain`, které automatický režim vypnou). Vrátí se, jakmile se spustí datový proud kamery (na to se čeká asi 15 s, poté 10 s na první úplný buffer) a vypíše `stream: streaming` nebo `stream: no_frames -- connected, no frames yet: …`, pokud datový proud nic nedodal — u kamery s jumbo pakety, s doporučením znovu se připojit pomocí `--packet-size standard`. Zatímco se jedna kamera připojuje (při prvním připojení se stahuje její kalibrace, což trvá asi 70 s), další příkaz, který kameru potřebuje, na ni čeká maximálně 5 s, poté selže s chybou HTTP 503 `CAMERA_POOL_BUSY` s uvedením názvu kamery, která se připojuje; spusťte jej znovu za chvíli. `lattice status` a seznam kamer nikdy nečekají: kamera uprostřed připojování se zobrazuje jako připojující se, s uvedením fáze. |
| `lattice cam-disconnect [--serial SN] [--all] [--keep-settings \| --clear-settings]` | Uvolnění. Kamera se vrátí do stavu, v jakém byla nalezena při připojení (zapojení spouště, PTP, nastavení); `--keep-settings` místo toho uloží její nastavení do vlastní paměti, `--clear-settings` resetuje tuto paměť na tovární nastavení (viz [Paměť kamery](#camera-memory----keep-settings----clear-settings)). Kamera, která je členem pole, uvolní celé pole. |
| `lattice cam-list` | Seznam kamer v poolu. |
| **`lattice array-connect`**|**Připojte trvalé synchronizované pole (DOPORUČENÝ vstupní bod).** Spustí kompletní proces inteligentní přípravy v grafickém rozhraní. U 9 kamer to trvá asi minutu; hned po zapnutí kamer to může trvat i několik minut. Během čekání vypisuje každý krok, kterého backend dosáhne, jako `    <s> s: <step>` (například `    41 s: open SN:214000535 (4 of 9)`), a protokol backendu obsahuje jeden řádek s časem pro každý krok (`[array-connect <id>] <step>: <s> s …`) a poslední řádek s nejpomalejšími kroky. Čeká až 600 s. Pokud do té doby nedojde k navázání spojení, backend ho zruší: každá kamera, kterou otevřel, se vrátí do stavu, v jakém byla nalezena, a uzavře se, příkaz vypíše `Error:` s krokem, kterého dosáhl, a ukončí se `1`, přičemž není připojeno žádné pole (HTTP 504 `CLIENT_DEADLINE_PASSED`). Klávesová zkratka Ctrl+C zruší připojení stejným způsobem (`Interrupted: …`); pokud bylo pole již připojeno, řádek to uvede a uvede název `array-disconnect`, který ho uvolní. Příkaz ukončený během čekání (`kill -9`) způsobí, že připojení bude zrušeno, pokud backend neobdrží od něj žádnou odezvu po dobu 30 s. Pole, které je aktivní, ale ještě nemá synchronizovanou skupinu rámců, je stále hlášeno jako připojené, s jeho proudem jako `no_frames`. |
| `lattice array-disconnect [--array-id ID] [--all] [--keep-settings \| --clear-settings]` | Uvolnění pole. Každá kamera zůstává v režimu volného běhu (`TriggerMode` `Off`), přičemž vše ostatní zůstává tak, jak to připojení našlo — nastavení spouště a synchronizační linky (u masteru`TriggerSource` se vrátí do původního stavu, její synchronizační linka se vrátí na vstup), její PTP a nastavení kamery, která změnila relace — takže příkaz pro jednu kameru na členovi funguje okamžitě. `--keep-settings` uloží nastavení každé kamerytak, jak je zanechala relace, do vlastní paměti kamery; `--clear-settings` tuto paměť resetuje na tovární nastavení. Viz [Paměť kamery](#camera-memory----keep-settings----clear-settings). |
| `lattice array-list` | Seznam připojených polí. |
| `lattice array-status [--array-id ID]` | Snímková frekvence živého obrazu, PTP, poslední chyba. `ptp_enabled` je rovno `True` pouze tehdy, když všichni členové mají zapnuté čtení PTP; `ptp_off:` označuje člena, u kterého tomu tak není. Řádek `state:` čte `restarting (applying settings)` během restartu nastavení, `recovering (…)` při opětovném otevření člena, který ztratil napájení (viz poznámka níže), `stalled (…)` při běhu smyčky grab, která však nezveřejňuje žádnou skupinu rámců, a `stopped (…)` poté, co smyčka grab ukončila činnost (znovu připojte pole). |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--orient] [--index\|--no-index] [--no-daq] [--daq-tolerance-ms MS] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Jedno synchronizované zachycení z živého pole — Jednorázové / Kontinuální / Intervalové / Nejrychlejší. **Výchozí nastavení je `all`**(jeden soubor pro každý příslušný typ exportu na každou kameru),**zarovnané** stejně jako snímání aplikace (`--no-aligned` pro nativní soubory). Přeskočené kamery (např. RGB, vyloučené z radiance/reflektance) jsou vykazovány s `Skipped: SN:<serial> (<reason>)`; hodnota DAQ, která načítá použitý záznam, je uložena společně s ním, včetně záznamu pouze v surovém formátu, a vykazována s `DAQ: <path>` ([Hodnota DAQ při načítání záznamu se ukládá](#the-daq-reading-a-capture-saves)). Viz [Režimy snímání, záznamníky a offline zpracování](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only] [--expr EXPR]` | Záznam živého zobrazení kombinovaného indexu do videa/GIF (v monitorovací kvalitě; pokud není otevřeno žádné grafické rozhraní, backend vykreslí NDVI pole, nebo `--expr`). |
| `lattice array-burst [--duration S] [--max-frames N] [--orient] [--build] [--products …]` | Sériové snímání v surovém formátu Bayer s vysokou snímkovou frekvencí (pro analytické účely; offline zpracování). Stejně jako `array-record` a `array-capture --continuous` se při každém dotazu hlásí backendu: ukončeno bez zastavení (`kill -9`), backend jej zastaví 30 s po poslední kontrole (`stop_reason` `client_gone`; manifest snímkové série je finalizován a je zahájeno vytváření kombinovaného indexu, stejně jako při zastavení) a pole zůstává připojeno. |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Zpracovat uloženou surovou sérii snímků do kalibrovaného videa (videí). |

> **Kamera, která ztratí napájení nebo spojení, se sama znovu připojí.** Když člen přestane odpovídat (vypnutí a zapnutí napájení přepínače restartuje všechny kamery, které napájí), pole na něj počká, znovu jej otevře, jakmile začne opět odpovídat, a vrátí jí nastavení, které určilo `array-connect`: PTP, spouštěcí a synchronizační linku, síťové tempo, formát pixelů a velikost snímku, a automatickou expozici. Poté pokračuje ve streamování bez nového `array-connect`. Mezitím `array-status` zobrazuje `state: recovering (...)` a `last_error` udává, na kterou kameru se čeká; poté se vypíše `re-opened:`. Fotoaparát, který se restartoval, se vrátí s vypnutým PTP, takže dokud není znovu otevřen, je `ptp_enabled` rovno `False` a `ptp_off:` jej pojmenuje.

> **Zadržená kamera se nikdy neotevře podruhé.** Druhé otevření zastaví datový proud držitele a přepíše jeho nastavení, což může způsobit pád backendu. Takže u kamery, kterou drží pool (GUI nebo `cam-connect`), u prvku pole nebo u kamery, kterou drží úloha na pozadí (běžící `lattice continuous`, `multi-stream` nebo `align-stream`, ať už vlastní nebo jiného klienta), `lattice continuous`, `lattice capture`, `cam-connect`, `power --sleep`, `calibrate`, jednorázovýzáběr `white-balance` a `presets --apply` odmítají s kódem HTTP 409 `CAMERA_HELD`, přičemž uvádějí držitele a příkaz k uvolnění (`chloros-cli lattice cam-disconnect -s <SN>`, `chloros-cli lattice array-disconnect --array-id <ID>`, nebo u úlohy její ID a kdo ji spustil) a ukončí se `1`. Totéž platí pro kameru, kterou právě používá jiný příkaz (`multi-test` nebo `multi-capture` otevírající své kamery, další jednorázový příkaz), a to pro každý příkaz zde: chyba říká, že je třeba počkat, až se daný příkaz dokončí. `lattice power` bez `--sleep` čte prostřednictvím vlastního identifikátoru držitele, pokud existuje (pool, pole, otevřená kamera úlohy), a je odmítnut v případě kamery, kterou právě používá jiný příkaz. Každá odpověď `CAMERA_HELD` obsahuje kód jak v `code`, tak v `error_code`. Příkazy, které otevírají všechny kamery, najdete v části [Více kamer (dočasné relace)](#multi-camera-transient-sessions).

##### Možnosti `array-connect`

| Příznak | Výchozí | Popis |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automatické vyhledání všech kamer LATTICE (vyžaduje ≥2) | První sériové číslo označuje MASTER. Pokud je parametr vynechán, vyhledávání se omezí na modely LATTICE (M3C / M3M) a připojí se všechny z nich. |
| `--line {Line0,Line2,Line3}` | `Line2` | Synchronizační linka GPIO. |
| `--target-fps F` | auto | Frekvence spouštění Masteru. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Přepsání volby úrovně. |
| `--wire-ceiling-mbps MB_PER_S` | automaticky detekováno | **Trvalá šířka pásma hostitele v MB/s — hodnota, na které závisí alokace celého pole.** Snižte jej, pokud pole hlásí poškozené rámce GVSP: automatická hodnota je odvozena z inzerované rychlosti spojení síťové karty, která nadhodnocuje USB adaptéry, úzké PCIe linky a vytížené sdílené struktury. Ukládá se v bloku zachycení pole projektu, takže jej obnoví opětovné otevření / CLI / SDK. Viz [Stav pole](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | žádné (celý snímač) | Hardwarové sdružování (vždy průměr). |
| `--width N` / `--height N` / `--pixel-format FMT` | celý snímač; formát pro každou kameru | Explicitní velikost snímku nebo formát pixelů pro každou kameru namísto volby „celý snímač“ a formátu pro každou kameru (Mono12 / BayerRG12). |
| `--no-recommend` | vypnuto | Přeskočit krok síťové analýzy (každá kamera si pak zachová svou aktuální velikost snímku namísto nastavení na celý snímač). |
| `--no-ptp` | vypnuto | Zakázat PTP (časová razítka mezi kamerami pak **nejsou** srovnatelná). |
| `--packed {auto,on,off}` | `auto` | Streamovat 12bitové snímky v komprimované formě a dekomprimovat je na tomto počítači — více snímků za sekundu při plném rozlišení snímače, identická data. `auto` komprimuje pouze tehdy, je to užitečné a tento počítač to zvládne; `off` nikdy nekomprimuje. Viz [Komprimovaný 12bitový přenos](#packed-12-bit-transport---packed). |
| `--packet-size {auto,jumbo,standard,N}` | `auto` | Velikost paketu GVSP pro každou kameru v poli. `auto` umožňuje každé kameře prozkoumat síť; `jumbo` = 9000, `standard` = 1500 nebo velikost v bajtech ji omezuje. Před verzí 1.3.0 `array-connect` tento příznak přijímal, ale ignoroval jej. |

> **Ve výchozím nastavení se používá plný snímač.** Bez `--width` / `--height` / `--binning` /
> `--pixel-format`, `array-connect` nastaví každou kameru na plný snímač — bez
> binningu, žádné ořezávání ROI — a nechá snímkovou frekvenci klesnout na úroveň, kterou tento počítač a
> síť poskytují, namísto zmenšování snímků za účelem udržení frekvence. Řádek
> s analýzou sítě zobrazuje předpokládanou frekvenci a následující řádek uvádí
> rychlejší možnosti (binning nebo menší ROI) s jejich předpokládanými frekvencemi; přejděte
> `--binning` nebo `--width`/`--height`, abyste si jednu vybrali. Connect nikdy neprovádí binning ani ořez
> sám od sebe.

##### Balený 12bitový přenos (`--packed`)

Při plném využití senzoru je snímková frekvence pole určena tím, jak rychle může každá kamera odesílat
své snímky. 12bitové formáty (`BayerRG12`, `Mono12`) se obvykle přenášejí jako 16bitová
slova; **komprimovaný** přenos odesílá stejných 12 bitů v 1,5 bajtu na pixel a
je na tomto počítači rozbalí, takže odeslání každého snímku trvá o 25 % méně času —
přibližně **o 33 % více snímků za sekundu při plném využití snímače** (naměřeno: 10,7 → 14,0 fps
u pole s 9 kamerami). **Uložená data jsou identická**: stejné hodnoty, stejné
16bitové soubory, stejná metadata.

- `auto` (výchozí nastavení) komprimuje pouze tehdy, když to má smysl **a** tento počítač si může dovolit
  dekompresi. Při připojení backend změří svou vlastní rychlost rozbalování, počet jader,
  aktuální zatížení procesoru a síťovou kapacitu, takže stolní počítač s volnými jádry komprimuje
  při plném využití snímače, zatímco zaneprázdněný nebo nízkoenergetický počítač streamuje data v rozbalené podobě. Malé oblasti zájmu (ROI) a sjednocené snímky přinášejí
  jen málo.
- `off` nikdy nekomprimuje — použijte jej k vyloučení komprese při diagnostice nebo na
  počítači, který musí veškerý výkon procesoru vyhradit pro jinou práci.
- `on` vždy komprimuje 12bitové formáty.

Výstup příkazu connect zobrazuje rozhodnutí a jeho důvod (`komprimováno 12 bitů: zapnuto (auto:
odhadováno 11,0 -> 14,4 fps, …)`); `/api/camera/array/status` hlásí
`packed_transport` a `packed_decision`. Chcete-li nastavit výchozí hodnotu pro celý počítač
namísto pokaždého předávání příznaku, nastavte `CHLOROS_ARRAY_PACKED12=off` (nebo
`on`) v prostředí backendu — explicitní `--packed on|off` má stále přednost.
8bitové a 16bitové formáty se nikdy nekomprimují.

### Jeden projektový záznam, všechny režimy

Složka projektu obsahuje nastavení všech zařízení: `cameras.json` — pro každou kameru (`cameras[].settings`, pro jednu kameru i pro každý člen pole) a nastavení platná pro celé pole (`arrays[].capture`: formát, ROI, binning, frekvence spouštění, limit přenosu dat) — a `sensors.json`, pro každý světelný senzor (integrační čas, průměr snímku, limit). Aplikace, CLI, SDK i server MCP čtou a zapisují stejné soubory stejným způsobem:

- **Zařízení připojené v rámci projektu je do něj uloženo a zaznamenává do něj každou změnu.** „V rámci projektu“ znamená s projektem otevřeným v backendu (aplikace) nebo — z pohledu SDK — prostřednictvím souboru `ChlorosProject`, který určuje vlastní složku. Kamera, pole nebo světelný senzor, které projekt dosud neobsahuje, se do jeho souborů přidají při připojení (`lattice cam-connect`, `array-connect`, `daq pool-connect`, aplikace, SDK, `chloros-mcp`) spolu s jeho nastavením. Změna z jakéhokoli režimu (`lattice exposure`, `gain`, `configure --set`, `resolution`, nastavení polea frekvence spouštění, kryt světelného senzoru, ovládací prvky aplikace) se o chvíli později uloží do souborů daného projektu. Expozice, zesílení nebo cíl automatické expozice (AE) nastavené na kameře se uloží jako aktuální nastavení, takže při opětovném otevření projektu se tato nastavení znovu použijí.
- **Otevření projektu nastaví jeho zařízení podle něj**, a to ve všech režimech: kamera, pole nebo světelný senzor připojený bez vlastního nastavení převezme uložená nastavení projektu (`lattice cam-connect` / `array-connect` při otevřeném projektu, `chloros-cli project connect`, `open_project().connect_all()`, opětovné otevření aplikace). Pole připojené pod novým ID aktualizuje pole, které projekt uložil se stejnými kamerami; pole s jinými kamerami se přidá vedle něj.
- **Vlastní uložení aplikace nikdy nepřepíše aktuální nastavení zařízení** starší kopií a zachovává přepsání pro jednotlivé členy uložená volajícím z CLI nebo SDK.
- **Formát pixelů, který nastavíte, je ten, který si projekt uchová.** Formát nastavený v nastavení pole (Array Settings), pomocí `--pixel-format` nebo `connect_array(pixel_format=)`, včetně 8bitového, se uloží a při opětovném připojení se přesně obnoví. Pokud není nastaveno nic, každá kamera se připojí ve svém výchozím 12bitovém nastavení (`BayerRG12`, `Mono12` u monochromatického snímače). Název formátu označuje jeho bitovou hloubku; rodina odpovídá snímači, takže barevná kamera není nikdy nastavena na formát Mono ani černobílá kamera na formát Bayer, a řádek `[PixelFormat <serial>] ... written` to uvádí, pokud byl formát namapován. Exporty snímků v 8-bitový záznam se exportuje jako 8-bitový, včetně odrazivosti: je přepočítán a označen kódem `PixelScale`.
- Zařízení připojené bez projektu nic nezaznamenává.
- **Záznam se řídí projektem vytvořeným po něm; otevření projektu ponechá záznam v původním projektu.**Při vytvoření projektu — ať už jde o SDK `create_project()`, `chloros-mcp` v rámci `create_project` nebo o volbu**Nový projekt**— nebo duplikován (funkce**Duplikovat**v aplikaci, která otevře kopii), zatímco záznam stále zapisuje do projektu, který byl otevřený předtím, záznam pokračuje a to, co zapisuje, skončí v novém projektu, na stejném místě uvnitř něj (níže). Otevření existujícího projektu — `chloros-cli process`, `project devices --reconnect`, `ChlorosProject.process()`, funkce**Otevřít projekt** v aplikaci — ponechá nahrávání stále probíhající v projektu, do kterého se nahrává: tam se dokončí a jeho soubory se připojí k seznamu souborů daného projektu (aplikace zastaví každé nahrávání před otevřením projektu).
  - `array-record` a `array-burst`: video nebo složka s jednotlivými snímky se po zastavení nahrávání přesune tam vcelku (je to `--duration`, Ctrl+C). `Burst dir`, který se vypíše, je nový, a `--build` a `array-build-video` z něj vycházejí. Pokud disk přesun odmítne, zůstane záznam tam, kam byl zapsán;konec protokolu uvádí důvod, stejně jako HTTP odpověď při zastavení (`relocation_error`). Surové snímky zastavené série a `.daq` se přidají do seznamu souborů projektu, do kterého se dostanou; video (`array-record`, video sestavené ze série) se do seznamu souborů projektu nepřidá.
  - `array-capture --continuous`: každý záznam po provedení změny se uloží do nového projektu; ty, které byly uloženy předtím, zůstanou na svém místě.
  - `daq pool-record` a `project sensor log` (a vlastní záznam aplikace pomocí světelného senzoru): soubor `.daq` se po ukončení nahrávání přesune do složky nového projektusložku `light_sensor` a připojí se k seznamu souborů tohoto projektu.

  Záznam do složky mimo otevřený projekt (např. `-o` jinde) zůstává tam, kde je, a soubory `lattice continuous` a `multi-stream` si zachovají složku, do které byly uloženy.
- **V aplikaci se záznamy po vytvoření nového projektu ukládají do něj.** Vytvoření nebo duplikování projektu zachová všechny probíhající snímací úlohy a nahrávání a každý záznam pořízený poté (Zachytit vše, Spustit snímání) se uloží do nového projektu a přidá se do jeho seznamu souborů. Otevření existujícího projektu v aplikaci nejprve zastaví všechny tyto úlohy.
- **Projekt nelze přesunout během nahrávání.**Pokus o přesunutí otevřeného projektu do jiného pracovního adresáře (v aplikaci v části**Nastavení → Pracovní adresář**; žádný příkaz CLI projekt nepřesune, ale HTTP klient odesílající data na `/api/move-project` ano) je odmítnuto s chybou HTTP 409 `PROJECT_RECORDING`, pokud do něj zapisuje záznam pole nebo sériový záznam, nepřetržité snímání pole, úloha typu `lattice continuous` / `multi-stream` nebo záznam ze světelného senzoru do něj zapisuje: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.` Aplikace odmítne stejným způsobem, ještě předtím, než požádá o výběr složky, zatímco běží její vlastní funkce „Zachytit vše“, „Spustit snímání“ nebo záznam ze světelného senzoru.

Toto je nezávislé na tom, v jakém stavu jsou kamery při odpojení (viz [Paměť kamery](#camera-memory----keep-settings----clear-settings)): projekt si při otevření nastavuje své kamery sám.

### Paměť kamery — `--keep-settings` / `--clear-settings`

Když je připojeno pole, jeho kamery převezmou nastavení pole a vy je poté můžete změnit. Když Chloros uvolní kamery — `array-disconnect`, konec přechodového příkazu (`multi-capture` bez pole, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`), zastavení `lattice continuous`, ukončení `lattice capture` kamery, kterou neobsahuje — každou z nich vrátí do stavu, v jakém byla nalezena před připojením: její zapojení spouště a synchronizační linky, nastavení PTP a kamery (úplný seznam je v části [Více kamer (dočasné relace)](#multi-camera-transient-sessions)).

Pro použití kamer mimo projekt zapíší dva příznaky u kteréhokoli z těchto příkazů do **vlastní paměti** kamery — tedy do nastavení, se kterým se kamera spouští v jakémkoli softwaru:

- **`--keep-settings`** uloží nastavení každé kamery do její paměti tak, jak je zanechala relace (UserSet2, vytvořený jako spouštěcí sada). Kamery zůstávají i nadále v režimu volného běhu (`TriggerMode` `Off`), zapojení synchronizační linky zůstává beze změny. Kamera bez UserSet2 ukládá do UserSet1; tento počítač si to pamatuje, takže Chlorosovo pole se samo opraví — což resetuje UserSet1, který se odchýlil od základní hodnoty systému Chloros — jej ponechá beze změny.
- **`--clear-settings`** resetuje paměť každé kamery na tovární nastavení: základní hodnotu systému Chloros, kterou jeho samooprava pole zakotvuje — tovární výchozí nastavení pomocí `TriggerMode`, `Off`, `TriggerSource` a `Line2`, automatická expozice, zachovaná kalibrovaná vyvážení bílé u RGB kamery — v UserSet1, vytvořeném jako spouštěcí sada.
- Tyto dvě možnosti nelze kombinovat. Každá kamera dostane jeden řádek: `SN:…: settings saved to the camera's memory (UserSet2); it powers up with them.` nebo `SN:…: camera memory reset to the factory settings.`; odpověď `camera_memory` je pro každou kameru stejná.

Aplikace tyto příznaky nenabízí.

**Každý způsob ukončení vrátí kamery do původního stavu.**Stejný návrat do původního stavu se provede při každé cestě ukončení, nejen u výše uvedených příkazů: `cam-disconnect` (a kamera**X**v aplikaci); ale i**X** z aplikace na poli nebo v řádku libovolného člena pole (pole nemůže běžet bez člena, takže se uvolní celé pole); při zavření aplikace (ta dává backendu až 20 s na to, aby vrátil všechny kamery, pole a spuštěný příkaz `lattice continuous`, než ji zastaví; příkaz, který v tu chvíli stále běží — přechodný příkaz otevírající své kamery nebo čekající na jejich PTP hodiny, jednorázový příkaz `lattice capture`, `power`, `white-balance` nebo `presets --apply` — se zastaví v nejbližším bezpečném bodě, nejprve vrátí své kamery do původního stavu, a to až po dobu 8 s, a odpoví HTTP 503 `SHUTTING_DOWN`; stále běžící připojení typu „array“ — `lattice array-connect`, připojení aplikace — se zastaví před svou další kamerou a vrátí kamery, které otevřelo, a to na dobu až 10 s, a odpoví HTTP 503 `SHUTTING_DOWN`; a ten, který dorazí během vypínání, nic neotevře: HTTP 503 `SHUTTING_DOWN`); a příkaz CLI, který je ukončen nebo jehož terminál se zavře (SIGTERM / SIGHUP zastaví jeho úlohu — a vrátí kamery do původního stavu — předtím, než se ukončí (`143` / `129`, podobně jako Ctrl+C). `lattice continuous`, `multi-stream` nebo `align-stream`, které byly přímo ukončeny (SIGKILL) a nemohou nic zastavit, mají svou úlohu zastavenu a kamery vráceny do původního stavu backendem 30 s po jeho poslední aktualizaci. Backend, který havaruje nebo je násilně ukončen, nemůže nic obnovit; restart vrátí každou kameru do její paměti a při dalším připojení Chloros se kamera, kterou najde v pohotovostním režimu, znovu připojí.**Projekt si nastavuje své kamery sám.** Když se pole připojí k otevřenému projektu, každý člen získá nastavení, která pro něj projekt uložil — expozici, zesílení nebo cíl AE, které jsou pro něj nastaveny (považovány za vaše, takže automatická expozice je nechá beze změny) a režim vyvážení bílé (korekce Chloros; vlastní vyvážení bílé kamery zůstává vypnuté) — cokoliv, co paměť kamery obsahuje.

### Smart-AE / Smart-Capture

Sestavy LATTICE spouštějí nepřetržitou automatickou expozici (AE) na pozadí, jakmile se připojí, ale u nově nasměrované scény trvá chvíli, než se expozice ustálí. `array-capture --smart` je **praktické řešení**: počká, až se AE ustálí na všech kamerách v poli, a teprve poté spustí snímání. Použijte jej, když během relace měníte scény.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Výchozí nastavení stabilizace je konzervativní: časový limit 5 s, stabilizační okno 1,5 s, tolerance rozptylu expozice ±5 %. Pokud potřebujete od automatizace odlišné chování, proveďte ladění pomocí SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`).

#### Jak funguje automatická expozice

- **Nejprve expozice, až poté zisk.** Je-li zisk nastaven na Auto, zůstává na hodnotě 0 dB, dokud expozice nedosáhne nejdelší povolené hodnoty.
- **Při ručním nastavení expozice zůstává zesílení v režimu Auto**, dokud nezměníte zesílení nebo jej nepřepnete do ručního režimu. Po návratu do režimu Auto je limit zesílení stejný jako předtím.
- **Malé světlé oblasti scénu neztmavují.** Odlesky do 1 % snímku jsou ignorovány a u panelů známého kalibračního terče až do 0,5 % panelu.
- **Horní limit expozice DAQ platí pouze tehdy, když je v zorném poli kalibrační terč.** Zabraňuje to oříznutí bílého panelu; pokud není žádný panel vidět, kamera exponuje podle scény.
- **Stálé osvětlení zůstává neměnné a změněné osvětlení se ustálí během několika sekund.** Při stálém osvětlení zůstávají expozice a její limit neměnné a po výrazné změně osvětlení se obraz ustálí během několika sekund. Přesvětlení již trvale neomezuje expozici a při změně oblasti zájmu (ROI) se expozice znovu nastaví.
- **Cílová hodnota jasu zůstává v dosahu.** Nestoupá, pokud je kamera na svém expozičním limitu a její zesílení nemůže vzrůst, takže se vracející světlo nezpůsobí přeexponování snímku.
- **Kamera, jejíž automatická expozice přestane reagovat, je udržována na úrovni odpovídající aktuálnímu osvětlení.** Za jasného slunečního svitu může kamera klesnout na nejkratší expozici a zůstat tam s tmavým snímkem, přičemž by všechna její nastavení vykazovala normální hodnoty. Chloros sleduje snímek, udržuje kameru na úrovni, kterou si zachovala pro dané osvětlení, a uvolní ji, jakmile se světlo změní. Tmavý nebo zakrytý světelný senzor tento proces nenarušuje.

### Úrovně exportu snímků (výchozí nastavení `all`)

Od této verze, `lattice capture`, `lattice multi-capture` a `lattice array-capture` **výchozí nastavení je `--processing all`** — jeden uložený soubor pro každý typ exportu, který se vztahuje na každou kameru, což odpovídá chování funkce „Capture All“ v grafickém uživatelském rozhraní. Úrovně jsou:

| Úroveň | Výstup | Vztahuje se na |
| --- | --- | --- |
| `raw` | Jednokanálový Bayer (černobílé kamery: jeden pásmo) přímo ze snímače. | Všechny kamery. |
| `debayered` | 3kanálová barva, uložená v pořadí kanálů **RGB**(soubor TIFF s fotometrickou interpretací RGB; soubory PNG/JPEG se rovněž otevírají jako RGB): z raw dat je odečten tmavý posun (DSNU) z kalibračního balíčku, poté je použita demosaikace OpenCV s ohledem na hrany— bez LUT linearity, flat fieldu nebo unmixu (černobílé kamery: 1kanálová škála šedé). Snímky po demosaikování v paměti, které vrací Pythonové SDK (`grab()`, `frame_stream()`), jsou v pořadí**BGR** podle OpenCV; pouze uložený soubor je ve formátu RGB. | Všechny kamery. |
| `radiance` | float32 W/m²/sr/nm prostřednictvím úplného radiometrického řetězce. | Pouze multispektrální (M3C/M3M) — **vynecháno u kamery s RGB filtrem**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), připraveno pro Pix4D. | Pouze multispektrální a **pouze pokud je přiřazen DAQ a kamera je kalibrována**; jinak se přeskočí. |
| `display` | Kompletní řetězec náhledu v grafickém rozhraní (CCM + WB + gama podle profilu kamery). Uloženo ve složce `preview/`. `lattice capture` také akceptuje pro ni název `preview`; `array-capture` / `multi-capture` přijímají pouze `display`. | Všechny kamery. |

Zadejte jednu úroveň, chcete-li uložit pouze tu jednu (`--processing debayered`). Když zadáte `all`, úrovně, které se na danou vačku nevztahují, jsou přeskočeny (a nahlášeny), ale nevzniká chyba — nepřipojená nebo nekalibrovaná kamera stále obdrží `raw` / `debayered` / `preview`.

#### DAQ, který čte záznam, ukládá

Kdykoli je DAQ připojen k kameře nebo soustavě kamer, nebo je zařazen do skupiny, je hodnota směřující dolů použitá při snímání zapsána do souboru **`.daq`** ve složce „captures“ v podsložce `daq/`, spolu s hodnotou `.match.json` (jak daleko byla hodnota od expozice) a `.cameras.json` (čí hodnota to je), a je uvedena na řádku `DAQ:`. To platí pro každou úroveň, včetně surových nebo debayered snímků, takže jakýkoli snímek lze později znovu zpracovat na radianci, odrazivost nebo index. Jedná se o stejný soubor se stejným názvem a doprovodnými soubory, ať už snímek pochází z grafického uživatelského rozhraní (GUI), příkazového řádku (CLI) nebo sady SDK. Pokud není k dispozici vůbec žádná hodnota z DAQ, neexistuje položka `.daq` a položka `multi-capture` bez připojeného pole (nouzové řešení pro přechodné stavy) neuloží nic.

| Příznak | Účinek |
| --- | --- |
| `--no-daq` | Neukládat naměřenou hodnotu u snímku, jehož úrovně ji nepoužívají (`raw`, `debayered`). Úroveň `radiance`, `reflectance` nebo `display` ji stále zapíše, protože opětovný import potřebuje naměřenou hodnotu, s níž byl produkt vytvořen. |
| `--daq-tolerance-ms MS` | Jak daleko od expozice může být naměřená hodnota a stále se považovat za v toleranci (výchozí hodnota 500). Mění se pouze výsledný verdikt (u `.match.json`, `in_tolerance` a `tolerance_ms`, u odrazivosti `DlsInTolerance`), nikdy se nemění použitá naměřená hodnota. Viz [Hodnota DAQ vzdálená od expozice](#a-daq-reading-far-from-the-exposure). |
| `--force-daq` | Starší způsob, jak vyžádat hodnotu `.daq` při snímání pouze ve formátu RAW. Nyní je to výchozí nastavení a `lattice capture` a `array-capture` tento příznak stále akceptují (`multi-capture` nikoli). `--no-daq` má přednost. |

Který DAQ: ten, který je vázán na kameru nebo pole (výběr světelného senzoru v grafickém rozhraní, uložená vazba projektu nebo `chloros-cli daq pool-bind`), jinak nejnovější, který má backend k dispozici. Série snímků ve formátu RAW (`array-burst`) si zachovává vlastní `.daq` pro každé jednotlivé měření ve složce burst, stejně jako dříve.

**Formáty PNG a JPEG mají 8 bitů na kanál, takže přijímají pouze soubory `display` / `preview` a indexové soubory.** `--format png` nebo `jpg` s úrovní, kterou nemohou pojmout, jsou odmítnuty ještě před zachycením, přičemž se uvede úroveň a formát TIFF, který ji pojme:

- `radiance` (float32) a `reflectance` (uint16, `32768` = 1,0) — vždy, takže `--processing all` (výchozí) s `--format png`/`jpg` je rovněž odmítnuto: použijte `--format tiff` nebo `--levels display`.
- `raw` a `debayered` z kamery vysílající více než 8 bitů (12bitové snímky M3C / Mono12 M3M) — CLI nedokáže rozpoznat bitovou hloubku kamery, proto je backend odmítá (HTTP 400). Jednotlivýsnímek `lattice capture --processing debayered` zachytí 8bitový snímek pro formáty PNG/JPEG a není odmítnut, stejně jako `multi-capture` bez trvalého pole (8bitový bez debayeringu).

Všechny trasy zachycování odpovídají stejným způsobem (`error_code` `FORMAT_CANNOT_HOLD_LEVEL`): `lattice capture`, `multi-capture`, `array-capture`, `project capture` / `burst` a trasy HTTP, které se za nimi nacházejí.

### Jak vypadá složka se záznamy

Každý typ exportu se ukládá do **vlastní podsložky**v rámci složky `-o`, takže při víceúrovňovém záznamu se typy nikdy nemíchají. Nákres znázorňuje**výchozí záznam `all` z jedné kamery pomocí `lattice capture`** — několik úrovní na jednu kameru, takže každý soubor nese příponu `_<level>`:

```
output/
├── raw/           capture_<ts>_SN<serial>-<filter>_raw.tiff
├── debayered/     capture_<ts>_SN<serial>-<filter>_debayered.tiff
├── radiance/      capture_<ts>_SN<serial>-<filter>_radiance.tiff
├── reflectance/   capture_<ts>_SN<serial>-<filter>_reflectance.tiff
├── preview/       capture_<ts>_SN<serial>-<filter>_display.tiff
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_radiance_clampmask.tif, reflectance/…, index/… — the unmix clamp
│                  masks, one subfolder per product folder, only when a band clamped
└── daq/           daq_data_<sensor id>_<ts>.daq: the downwelling reading matched to the capture
                   (+ .daq.cameras.json: whose)
```

Jak se liší názvy souborů:

- **Pole kamery — sériové číslo, poté filtr:** každý soubor záznamu nese název kamery, která jej pořídila, ve formátu `SN<serial>-<filter>`, kde `<filter>` je část označující filtr v modelu kamery: `SN261902142-F450` pro kameru `M3M-L87-F450`, `SN213800234-FRGN` pro model `M3C-L87-FRGN`, `SN214701292-FRGB` pro kameru RGB. Filtr spojuje sériové číslo s `-`, nikoli `_`, takže název má stejná pole oddělená `_` jako dříve (předpona, datum, čas, milisekundy, kamera, úroveň). Kamera, jejíž model nelze přečíst (vymazané DeviceUserID), je pojmenována pouze jako `SN<serial>`. Snímky z dřívějších verzí nemají v názvu žádný filtr; Chloros čte oba typy.
- **Předpona:** `lattice capture` pojmenovává soubory `capture_…`; `array-capture`, `multi-capture` s připojeným polem a `project capture` / `burst` / `stream` z pole pojmenovává soubory jako `sync_<ts>_SN<serial>-<filter>…`.
- **Přípona úrovně pouze u více úrovní.**Kamera, která ukládá**několik**úrovní v jednom snímku (`all` nebo `--levels a,b`), přidává `_<level>`. Kamera, která ukládá**jednu** úroveň, zachovává jednoduchý název bez přípony — `--processing reflectance` zapíše `reflectance/sync_<ts>_SN<serial>-<filter>.tiff` — takže skripty napsané pro snímky s jednou úrovní budou i nadále fungovat. Úroveň je stále určena složkou.
- **Masky omezovače následují za názvem produktu:** `<product name>_clampmask.tif`. Snímání odrazivosti na jedné úrovni zapíše `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`; snímání `all` zapíše `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`.
- **Přípona** navazuje na `--format`: výchozí `tiff` dává `.tiff`, `--format tif` dává `.tif` (masky omezovače jsou vždy `.tif`).
- **Jedna výjimka:** `lattice capture --processing debayered` (nebo `--levels debayered`) přijímá rychlou jednotlivoua zapíše `capture_<ts>_SN<serial>-<filter>.tiff` přímo do `-o` — bez podsložky.
- **Proudy a dávky** nesou stejné pole: `lattice continuous` zapíše `frame_<seq>_SN<serial>-<filter>.tiff`, `lattice multi-stream`, `stream_<seq>_SN<serial>-<filter>_<frame id>.tiff` a surovou sérii `<burst>_f<frame>_SN<serial>-<filter>_raw.tiff`.

`<ts>` je časové razítko záznamu a `<serial>` sériové číslo kamery, takže jedna synchronizovaná skupina sdílí
časové razítko napříč kamerami. **Všimněte si jedné nesrovnalosti:** úroveň `display` je uložena ve složce
s názvem `preview/`, zatímco víceúrovňové soubory si zachovávají `_display` v názvu — složka a přípona se liší
pouze pro tuto úroveň. Neznámé úrovně se ukládají do složky s vlastním názvem a pokud nelze podsložku
vytvořit, soubor se zapíše do kořenového adresáře výstupu, místo aby byl ztracen.

**Opětovné zpracování složky záznamů:**nasměrujte `chloros-cli process` na**kořenový adresář „captures“**
(`output/`). `process` obvykle importuje pouze složku, kterou určíte, ale pokud tato složka neobsahuje žádné
obrázky a obsahuje podsložky, automaticky se prohledá i do nich — takže podsložky dané úrovně i
podsložka `daq/` jsou načteny najednou. Každá úroveň snímku se importuje jako jeden obrázek, přičemž
ostatní úrovně jsou k dispozici jako režimy, nikoli jako jeden obrázek na úroveň.

Funguje také přímé pojmenování **podsložky úrovně** (např. `output/raw/`). Tímto způsobem však zůstane `daq/`
vynechán, proto zkopírujte nebo nasměrujte údaj z DAQ vedle něj, kdyžodvozujete radiometrický
produkt z `raw/` — jinak nebude mít shoda časových značek k čemu se přiřadit.

**Zpracování vždy začíná od `raw`.** V rámci každého snímku je zdrojem pro zpracování surový snímek;
`debayered`, `radiance`, `reflectance` a `preview` se objevují jako zobrazitelné režimy, ale nikdy se nevracejí
zpět do zpracování. Opětovné zpracování odvozeného produktu by znamenalo znovu aplikovat vinětaci, CCM a
výpočty radiance, které jsou již zapéčeny do jeho pixelů, proto Chloros upouští od
dvojitého zpracování. Dvě důsledky, o kterých je dobré vědět:

- Rendery `index/` a `composite/` nejsou **nikdy** nezpracovány. Jedná se o výstupy, nikoli o snímky —
  renderování pomocí LUT pro NDVI nemá žádný smysluplný výklad radiance.
- Složka snímků exportovaná **bez** `raw` (např. `array-capture --processing reflectance`) není
  žádný platný zdroj pro zpracování. Tyto záznamy se importují a zobrazují normálně, ale `process` je
  přeskočí a oznámí to:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Pokud skutečně potřebujete prostrčit odvozený produkt — relaci hubu zachycenou s
  zapnutým `demosaic` nebo ze starší složky — `--input-level {raw,debayered,processed}` vynutí vstupní
  bod a přepíše přeskočení. Tento příznak je záměrným únikovým východem; `auto` (výchozí nastavení)
  nikdy nezpracuje záznam, který neobsahuje surová data.

**Kalibrační značky: pouze u `raw`.** Soubor `raw` ze záznamu obsahuje kalibrační paket kamery: užitečné zatížení `CalibrationJSON` a značky kompatibilní s MicaSense (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), které popisují hodnoty v surovém formátu snímače. Všechny ostatní soubory na jednotlivých úrovních (`debayered`, `display`, `radiance`, `reflectance`, render `index/`) a každý snímek `lattice continuous` / `multi-stream` obsahuje identifikátor kamery (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) a vlastní záznam snímku, `XMP-mapir:FrameJSON` (expozice, zesílení, teplota, formát pixelů, čas pořízení, shoda s DAQ), protože každý produkt `process` nese tuto identifikaci. `process` vyhledá kalibraci takového souboru podle jeho sériového čísla a `CalBundleSha256` v kalibrační mezipaměti Chloros, přičemž balíček stáhne pouze jednou, pokud jej počítači chybí.

#### Soubory vytvořené z několika kamer

Soubor vytvořený z pásem více než jedné kamery — multibandový stack (`align-apply`, `align-stream`, `index --live --save-multiband`), živý index vytvořený z několika kamer (`index.tif`, `index_LUT.png`), snímek `composite/` z hromadného snímání — uvádí názvy všech kamer, z nichž byl vytvořen:

- **Seznamy podle jednotlivých kamer, v pořadí podle pásem** (kompozit: vrstva pozadí, poté popředí; indexová vrstva, kamery, z nichž pocházejí její pásma): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (kalibrační balíček, kterým prošla pásma jednotlivých kamer, jinak — sada `align-apply` / `align-stream`, jejíž pásma neprocházejí žádnou kalibrací — balíček kamery, který si Chloros uložil do mezipaměti, ten, podle kterého jsou pojmenovány její záznamové soubory), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — paralelní seznamy, přičemž položka *n* v každém z nich popisuje stejnou kameru; `-`, kde kamera nemá žádnou hodnotu (žádný kalibrační balíček, neznámý objektiv). Každý fotoaparát představuje **jednu** položku: pokud má jeho balíček pro dané pole více hodnot — například certifikáty halogenových žárovek v balíčku — jsou spojeny pomocí `;` (`F-1479-1;F-1480-1;F-1481-1`), nikdy čárkou, takže seznam obsahuje jednu položku na jednu kameru, bez ohledu na to, jak je vytištěn.
- **EXIF `SerialNumber`** obsahuje stejná sériová čísla, oddělená čárkami (`214000535,214000551`).
- Neobsahuje **žádnou** identifikaci jednotlivé kamery (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): tyto identifikují jednu kameru a její sadu a `process` by z nich určil kalibraci souboru. Index vypočítaný z pásem několika kamer (`index.tif`, `index_LUT.png`) obsahuje v EXIFu `Model` `LATTICE` (systém), nikoli model `LATT-…` jedné kamery.

Soubor obsahující pásma **jedné** kamery — buď soubor TIFF pro každý pásmo (`--per-band`), snímek `align-apply --per-camera`, zásobník nebo živý index jedné kamery — obsahuje identifikační značky dané kamery, stejně jako každý jiný soubor se záznamem. Sada souborů `align-apply`, soubory pro jednotlivé pásma a pro jednotlivé kamery, stejně jako každá sada souborů `align-stream` a její soubory pro jednotlivé pásma, nesou čas pořízení své skupiny (expozici referenční kamery, UTC), a snímek `--per-camera` z monochromatické kamery tvoří jednu rovinu, stejně jako zásobník `--bit-depth`.**Orientace kamery (překlopení / otočení).** Nastavení orientace kamery (funkce Mirror H /
Mirror V / rotace) určuje, jak se snímek uloží:

- **`raw` a `debayered` se ukládají přesně tak, jak je snímač načte**, bez ohledu na nastavení.
  Otočená nebo převrácená Bayerova mozaika by představovala odlišný vzor barevných filtrů a kalibrační
  mapy (dark offset, flat field) jsou v mřížce samotného snímače.
- **`preview`/`display`, `radiance`, `reflectance` a render `index/` jsou orientovány** — otočeny
  jednou, až po všech kalibračních výpočtech, takže se otevírají tak, jak byl snímek komponován. Jejich
  soubory `clamp_masks/…_clampmask.tif` se nacházejí na stejné mřížce.
- **Každý soubor snímku zaznamenává nastavení** v XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, ve směru hodinových ručiček, po
  otočení) a `ChlorosOrientationApplied` — `false` u surových a debayered (nativních pixelů),
  `true` na orientovaných výstupech. `ChlorosProcessing` uvádí operace aplikované na pixely daného souboru
  (`none` v případě nativních pixelů). To zahrnuje každý snímek, který `lattice continuous` vysílá (
  vlastní nastavení kamery, nepoužité), a každý snímek `lattice multi-stream` (identita: jeho
  členské snímky jsou nativní, jako u snímání pole). `lattice align-stream` / `align-apply`
  stohy zaznamenávají nastavení hlavní kamery, neaplikované: jejich pásma jsou deformována do snímku hlavní kamery,
  ale nejsou otočena.
- **`process` aplikuje zaznamenanou orientaci** při zpracování surových dat: nejprve tmavý offset, vyrovnání
  pole, demosaikování a rozložení v souřadnicích snímače, poté deformace pro zarovnání pole a
  orientace jako poslední — takže výstupy náhledu, radiance, odrazivosti a indexu vycházejí přesně jako
  u živého snímání a zaznamenávají se do `ChlorosOrientationApplied=true`. Nekalibrované záložní
  (`Sensor_Response_Images`, `Vignette_Corrected_Images`) jsou orientovány stejným způsobem, a to po
  aplikace flat field. Produkt po odstranění bayerského vzoru zůstává v původním formátu.
- **Prohlížeč obrázků Chloros zobrazuje soubor ve formátu raw nebo po odstranění bayerského vzoru otočený podle jeho záznamu** (pouze pro zobrazení —
  soubor se nemění), takže vypadá jako jeho produkty; miniatura, zobrazení v plné velikosti,
  odčítání kurzorem a testovací prostředí Index/LUT – všechny používají stejnou rotaci. Soubor bez záznamu a
  produkt (již orientovaný) se zobrazují tak, jak jsou uloženy. Export do testovacího prostředí zaznamenává orientaci, kterou
  pixely nesou (`ChlorosOrientationApplied=true`; identifikátor pro vrstvu bez záznamu).
- **Záznamy bez záznamu se zpracovávají jako nativní** (beze změny chování). Před verzí 1.3.0
  záběr z jedné kamery s nastaveným převrácením nebo otočením sám otočil své soubory RAW a soubory po odstranění Bayerova filtru;
  program Chloros nedokáže rozpoznat takový jižotočený surový soubor od nativního a nehádá, takže jej
  zpracovává tak, jak je uložen. Kalibrační mapy jsou na vlastní mřížce snímače, takže pro takový
  záběr použijte ten pořízený s verzí 1.3.0, pokud na kalibraci záleží.
- **Pole ve výchozím nastavení neotáčí žádný členský soubor** (snímek pro zarovnání a XMP soubor `Alignment*` jsou v
  vlastních, neotočených snímcích kamer) a zaznamenávají identitu. `array-capture --orient` otočí produkty každého
  člena po zarovnávací transformaci: podle orientace **hlavního snímku** v okamžiku, kdy byly
  transformovány do jeho snímku (aby členy zůstaly vzájemně zaregistrované), jinak podle orientace jejich vlastní kamery. Každý
  soubor zaznamenává orientaci snímku, ve kterém se nacházejí jeho vlastní pixely — surový údaj, orientaci vlastní kamery —
  a surový údaj obsahující transformační údaje o zarovnání také zaznamenává `ChlorosAlignedFlipHorizontal` /
  `ChlorosAlignedFlipVertical` / `ChlorosAlignedRotation`: orientaci vzorového snímku, pro výstup, který byl
  do snímku hlavního snímku. `process` otočí každý produkt o hodnotu, která patří k jeho pixelům:
  o vyrovnanou hodnotu, pokud deformuje soubor RAW (`--array-alignment`, výchozí hodnota — produkty živého vyrovnaného
  zachycení), nebo podle vlastní kamery, pokud se nevyrovnává (`--no-array-alignment` — živé
  nezarovnané snímky). Stejná možnost existuje pro `array-burst`, `project capture` / `burst` /
  `stream`, recepty a SDK.

### Přeskočené snímky ve smíšených polích filtrů

Pokud v jednom poli kombinujete RGB a multispektrální kamery, `array-capture --processing radiance` (nebo `reflectance`) uloží multispektrální snímky a **přeskočí** RGB— zářivost na jeden Bayerův čip nemá pro širokopásmový snímač význam. CLI výslovně vypisuje každý uložený soubor (s úrovní exportu), každý zapsaný záznam `.daq` a každé přeskočení, takže počet souborů není překvapivý:

```
  Saved: …/output/reflectance/sync_…_SN213800234-FRGN.tiff [reflectance] (SN:213800234, fid:1)
  Saved: …/output/reflectance/sync_…_SN214000533-FOCN.tiff [reflectance] (SN:214000533, fid:1)
  Saved: …/output/reflectance/sync_…_SN214701288-FNGB.tiff [reflectance] (SN:214701288, fid:1)
  DAQ:   …/output/daq/daq_data_daq-e-54b5e0_….daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Tokeny důvodů přeskočení mají formát `<level>-not-applicable-to-rgb-cam`. Kamera, u které není načten kalibrační balíček, přeskočí `radiance` / `reflectance` s `<level>-needs-calibration (connect the camera so its cal pack loads)`, a při chybějící použitelné hodnotě odrazivosti z DAQ přeskočí na `reflectance-skipped-no-dls`. Odrazivost lze také přeskočit na `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, a u `dls-uncalibrated-band-<nm>`, pokud se pásmo nachází převážně mimo radiometricky kalibrovaný rozsah světelného senzoru DAQ (~374–974 nm) — z dodávaných SKU pouze F988, jehož podporovanou cestou je pracovní postup s panelem odrazivosti.

Použijte `--processing debayered` (nebo `display`) pro zahrnutí všech kamer bez ohledu na typ filtru, nebo výchozí `all` pro získání všech příslušných úrovní pro každou kameru najednou.

### Hodnota DAQ vzdálená od expozice

Každá hodnota `.daq` uložená při snímání představuje hodnotu DAQ odpovídající dané expozici, zatímco hodnota `.match.json` vedle ní zaznamenává, jak daleko od sebe tyto dvě hodnoty byly (`gap_ms`) a zda se tato odchylka vejde do tolerance (`in_tolerance`, 500 ms, pokud není v nastavení snímání zadáno jinak `--daq-tolerance-ms`). Hodnota mimo tuto toleranci se přesto použije, takže nedojde ke ztrátě žádného snímku, a je označena: soubor XMP souboru odrazivosti obsahuje `DlsInTolerance=False` a `DlsMatchGapS`. `lattice capture`, `multi-capture` a `array-capture` jej také vypsají pod příslušným `.daq`, přičemžpři vysvětlování:

**Jak se naměřená hodnota přiřazuje k expozici.** Zachycení čeká na první naměřenou hodnotu DAQ po expozici, aby mohlo sloučit naměřené hodnoty na obou stranách této hodnoty (`method` `interpolated`). DAQ-E odesílá hodnotu každých 0,4 s za denního světla a každých 1–2 s v interiéru, takže čekání prodlužuje příkaz o zhruba jeden a půl takového intervalu, nikoli expozici. Na DAQ, který přestal odesílat, se nečeká. Hodnota DAQ-E je průměr osvětlení za interval od předchozího měření, takže je umístěno uprostřed tohoto intervalu: `gap_ms` se měří od expozice k tomuto středu a `bracket_stamps_ns` udává vlastní časová razítka měření. `array-capture --continuous` čeká, aniž by zpomaloval snímání: jeho surové soubory se zapisují okamžitě a jejich `.daq` se zapíše, jakmile dorazí hodnota. S `--interval` každý průchod čeká jako při jediném snímání.

```
  DAQ:   …/output/daq/daq_data_daq-e-54b774_….daq
         DAQ reading 114.9 s from the exposure (tolerance 500 ms): reflectance uses it, labelled DlsInTolerance=False
         DAQ stream fault: no frame accepted from daq-e-54b774.local in 25 s, after 1932 good frame(s). …
```

Pokud z daného odečtu nebyl zapsán žádný soubor odrazivosti, první řádek místo toho končí `saved anyway; its .match.json says in_tolerance: false`.

Zaznamenání stále končí kódem `0`: všechny soubory byly zapsány a uvádějí, co obsahují. DAQ, jehož datový proud se zastavil, nadále dodává své poslední měření, proto jej zkontrolujte pomocí kódu `chloros-cli daq pool-list`, než se spolehnete na odrazivost z tohoto zaznamenání. Backend znovu připojí sdružený DAQ-E, který přestal dodávat data (viz [DAQ-E, který přestane dodávat data](#a-daq-e-that-stops-delivering)). Hodnota mimo toleranci vypíše pouze řádek `DAQ:`, pokud se nejedná o uchovanou hodnotu (viz níže).

**Hodnota uchovaná i po odpojení příslušného DAQ.** Pokud žádný DAQ nemůže poskytnout hodnotu pro danou expozici (připojený DAQ není k dispozici nebo žádný DAQ ze sdíleného fondu žádnou hodnotu nemá, např. po `daq pool-disconnect`), zachycení znovu použije poslední hodnotu, kterou měl Chloros, a jeho `.match.json` popisuje tuto hodnotu pro danou expozici: `method`, `retained`, `gap_ms` z této expozice ke vzorkům, z nichž byla hodnota naměřena, `in_tolerance` z této mezery a `retained_from_epoch_ns`, což je expozice, ke které byla hodnota původně přiřazena. Snímání pořízené dlouho po odpojení DAQ proto vytiskne výše uvedené řádky a pod je ještě jedna, která se vytiskne i tehdy, když je naměřená hodnota stále v toleranci:

```
         This is the DAQ's last reading: the DAQ is no longer reporting, so each capture reuses it until a new one arrives
```

Jeho `.daq` stále označuje DAQ, který naměřil hodnotu, spolu s kalibrací a krytkou, pod kterými byla hodnota naměřena, takže při opětovném importu je zřejmé, který DAQ, kalibrace a krytka ji vyprodukovaly.

### Soubory, které se nepodaří zapsat

Kamera, jejíž soubor nelze zapsat – zaplněný disk, chyba oprávnění – nezabrání ukládání souborů ostatních kamer. Každý příkaz k zachycení (`lattice capture`, `multi-capture`, `array-capture`; `project capture`, `burst`, `stream`; a recepty `capture`, `burst` a `stream`) vypíše každý soubor, u kterého došlo k selhání, do stderr, poté **ukončí program `1`**, i když se zbytek uložil:

```
  Failed: SN:214701292 [reflectance] OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

Jsou zahrnuty všechny soubory, které záznam obsahuje, včetně jedinéhosnímku s odstraněným bayesovým kódem `lattice capture --levels debayered`. Záznam `.daq`, jehož sidecar s informacemi o původu `.match.json` nebylo možné zapsat, je uveden jako `[match]` (samotný soubor `.daq` byl uložen).

Na zaplněném disku se opakující záznam (`array-capture --continuous` / `--interval`, `project burst` / `stream`, recept `burst` / `stream`) **zastaví se v daném průchodu** namísto toho, aby po zbytek běhu na stejný disk nic nezapisoval. Jakákoli jiná chyba je hlášena v každém průchodu, ve kterém k ní dojde, a běh pokračuje až do konce, poté se ukončí `1`. `array-capture --continuous` (kontinuální sběr dat backendu) neuvádí žádný seznam souborů, včetně těch, u nichž došlo k selhání: počítá je v řádcích s průběhem a souhrnem (`N failed`, `N file(s) failed to write.`) a ukončí se `1` — viz [Kontinuální režim](#continuous-mode---continuous).

Záznamníky — `array-burst` a `array-record` — hlásí své vlastní chyby. `array-burst` se sám zastaví, když se disk zaplní, nebo těsně předtím. Oba pokračují i přes jakékoli další neúspěšné zápisy. Když některý z nich nahlásí chybu, je tato chyba vypsána za čítači a příkaz **ukončí `1`**:

```
  Recorder error: disk-full: OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

`Disk low:` namísto `Disk full:` znamená, že sériový záznam byl zastaven dříve, než došlo k zaplnění disk. Snímky zaznamenané před zastavením se zachovají. `array-burst --build` přeskočí sestavení, když dojde místo, protože videa by se ukládala na stejný disk. Nejprve uvolněte místo a poté spusťte `array-build-video --burst-dir …`. Po jakékoli jiné chybě sestavení stále pokračuje.

Chyba `array-record`, při které nebyl zapsán žádný snímek, je také neúspěšným záznamem (`Recorder error: no-frames: the combined view published no frame to record …`, ukončete `1`) — viz [`array-record`](#array-record--combined-index-videogif-monitoring-grade).

---

## Režimy snímání, rekordéry a offline zpracování

Všechny tyto funkce pracují s **trvalým polem** (nejprve spusťte `array-connect`). Odrážejí panel snímání v grafickém uživatelském rozhraní.

### Režimy `array-capture`

`array-capture` je jediný příkaz se čtyřmi režimy spouště a sadou přepínačů pro export:

| Režim | Příznak | Chování |
| --- | --- | --- |
| **Jednorázový** *(výchozí)* | (žádný) | Jedna synchronizovaná skupina snímků, poté ukončení. |
| **Kontinuální** | `--continuous` | Snímání jeden po druhém, dokud se neprovedou snímky `Ctrl+C`, `--count N`, `--duration S` nebo do zaplnění disku. Spustí vlastní nepřetržité snímání backendu (režim „Nepřetržitý“ v grafickém rozhraní) — viz [Režim „Nepřetržitý“](#continuous-mode---continuous) níže. |
| **Interval** | `--interval S` | Jeden průchod každých `S` sekund (měřeno od začátku každého průchodu), stejné meze. |
| **Nejrychlejší** | `--fastest` | Pouze surová+ přiřazená hodnota DAQ + kompozit kombinovaného indexu; přeskočí výpočty radiance/reflektance/zobrazení, aby se snímek zobrazil rychle. Implikuje `--processing raw --force-daq`. Uložené `.daq` později znovu zpracujte na kalibrované produkty. |

Přepínače exportu (lze kombinovat s libovolným režimem; všechny sdílejí koncový bod GUI/SDK):

| Příznak | Účinek |
| --- | --- |
| `--array-id ID` | Pole, ze kterého se má zachytit data (vynechat, pokud je připojeno pouze jedno). |
| `-o, --output DIR` | Výstupní složka (výchozí `output`, ve složce, ze které spouštíte příkaz). |
| `--format {tiff,tif,png,jpg,jpeg}` | Formát souboru (výchozí `tiff`). Formáty PNG / JPEG podporují pouze `display` a vykreslení indexu; viz [Úrovně exportu snímků](#capture-export-levels-the-all-default). |
| `--processing LEVEL` | Jedna úroveň exportu (`raw`, `debayered`, `radiance`, `reflectance`, `display`) nebo `all` (výchozí). |
| `--levels L1,L2,…` | Explicitní podmnožina typů exportu (např. `raw,radiance,reflectance`); **přepíše `--processing`**. |
| `--aligned` / `--no-aligned` | **Výchozí nastavení: zarovnané, stejně jako zachycení stejného pole aplikacíChloros**(její nastavení zachycení**Zarovnané** pro každé pole, zapnuté, pokud jej nevypnete). Ne-surový export každého člena (`debayered`, `display`, `radiance`, `reflectance`, vykreslení indexu) je transformován do snímku hlavní kamery a oříznut na společnou překrývající se oblast členů, takže soubory jsou pixelově zarovnány a mají stejnou velikost, oříznutí a tagy `Alignment*` jako soubory aplikace — menší než snímač (nebo oblasti zájmu (ROI)) o posuny jednotlivých členů; v rámci oblasti zájmu (ROI) se transformace přepočítává na okno. `XMP-mapir:AlignmentJSON` uvádí `"applied": true` a poskytuje `crop_roi`. Surový soubor zůstává bez deformace v původní velikosti, ale nese transformaci v metadatech (`"applied": false`). `--no-aligned` ukládá každý soubor v původní velikosti, bez deformace. `--aligned` je výchozí nastavení. Ať už je deformace použita či nikoli, každý soubor v souboru zaznamenává transformaci `Alignment*` své kamery, jakmile má pole profil zarovnání — s výjimkou kamery, kterou profil nevyrovná (vynechána nebo vynucena do identické polohy v důsledku neúspěšného řešení), a referenční kamery, pokud není vyrovnána žádná jiná kamera: ty neobsahují žádné značky `Alignment*`. Pokud pole nemá profil, je záznam uložen nevyrovnaný a CLI vypíše důvod na řádku `Alignment:`, stejně jako v každém jiném případě, kdy nebylo možné použít warp (členy s odlišným rozlišením, pole nesynchronizované kabelem, snímek, na který nelze profil namapovat). Automatické zarovnání pole vytvoří profil, jakmile se ustálí expozice; `align-calibrate` jej vytvoří na požádání. |
| `--orient` | Vytvořte **výstupy**pro každý člen — `display`, `radiance`, `reflectance`, render `index/` a `composite/` — podle orientace kamery po zarovnávací deformaci: produkt deformovaný do rámce hlavního snímku (výchozí nastavení) podle**orientace vzorového snímku**, takže součásti zůstávají vzájemně zarovnané; nesrovnaný snímek (`--no-aligned` nebo snímek, na který nebylo možné deformaci aplikovat) podle orientace vlastní kamery.**Výchozí nastavení: vypnuto — každý soubor součásti zůstává v původním formátu**, jak je tomu vždy u snímků pořízených v poli. `raw` a `debayered` zůstávají v nativním formátu v obou případech. Každý soubor zaznamenává orientaci, kterou mají jeho produkty (viz [Orientace kamery](#jak-vypadá-složka-se-zachycenými-snímky)); soubor XMP `Alignment*` zůstává transformací nativního snímku, aplikovanou před orientací, takže `process` (deformace, poté orientace) reprodukuje živý výstup. |
| `--index` / `--no-index` | Uložit / přeskočit překryv indexu vegetace pro jednotlivé kamery, pokud je nakonfigurován. Výchozí nastavení: vykreslit. |
| `--no-daq` | Přeskočit sidecar `.daq`, pokud žádná zvolená úroveň nepoužívá DAQ (zachycení pouze v surovém formátu). Ve výchozím nastavení se naměřená hodnota uloží, takže snímky lze offline znovu zpracovat na odrazivost/index. `--force-daq` je stále akceptováno (výchozí nastavení). |
| `--daq-tolerance-ms MS` | Tolerance shody DAQ pro tyto snímky (výchozí hodnota 500 ms). |
| `--smart` | Před spuštěním počkejte, až se AE ustálí na všech kamerách (viz [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | Komprese pixelů ve formátu TIFF. `none` (výchozí od verze 1.3.0; verze 1.2.0 zapisovala `deflate`) = nekomprimované — pixely snímače přesně tak, jak byly načteny, ~6,3 MB na snímek v plném rozlišení a nejvyšší trvalá rychlost; `deflate` = bezeztrátová komprese zlib L1 + horizontální prediktor, ~4,1 MB na snímek, pomalejší. Oba formáty jsou bezeztrátové a při importu se načtou identicky. |

> **TIFF s jedním zápisem + model s trvalou rychlostí.**Každý soubor se zapíše**jednou**: pixely, balíček XMP, IFD0 Make/Model/CameraSerialNumber a EXIF tagy `ExposureTime` / `SerialNumber` v jednom průchodu, přičemž se nic následně nepřepisuje. `--continuous` zachycuje po sobě jdoucí synchronizované skupiny — každý záznam převezme skupinu následující po té předchozí, včetně té, která byla zveřejněna ještě během ukládání předchozího záznamu — takže běží rychlostí pole. DEFLATE drží GIL v Pythonu, takže komprimované zápisy**nejsou**paralelizovány napříč vlákny zapisovače pro jednotlivé kamery; trvalé zachycování v plném rozlišení při rychlosti snímače vyžaduje nekomprimované zápisy (výchozí nastavení)**a** disk třídy NVMe (~600 MB/s trvale pro pole 9 kamer při ~10 fps). Stejný ovladač je dostupný jako `compression` na `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (aligned is the default; drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --no-index -o out/

# Each member at its native size, unwarped
chloros-cli lattice array-capture --processing reflectance --no-aligned -o native/

# Continuous raw capture for 60 s, as fast as the array publishes
chloros-cli lattice array-capture --continuous --duration 60 --fastest -o flightline/
```

#### Režim nepřetržitého záznamu (`--continuous`)

`array-capture --continuous` (bez `--interval` nebo `--smart`) předává řízení **vlastnímu nepřetržitému snímání backendu** — relaci, ve které běží režim Continuous v grafickém uživatelském rozhraní. Backend ukládá každou synchronizovanou skupinu sám, přičemž má v běhu dvě snímání a mezi skupinami nedochází k žádnému požadavku z CLI, takže drží krok s rychlým polem, kde by požadavek na jedno snímání zaostával (naměřeno 73–77 snímání/s na poli s 9 kamerami při ROI 512 pixelů). Každé výše uvedené přepnutí exportu se vztahuje na každý záznam.

- **Omezení:** `--count N` se zastaví po N záznamech (jeden záznam = jedna synchronizovaná skupina: soubory všech členů); `--duration S` po S sekundách. Bez zadání některého z těchto parametrů běží program až do `Ctrl+C`, zaplnění disku, výskytu chyby nebo odpojení pole. `Ctrl+C` zastaví relaci backendu a poté vypíše souhrn.
- **Průběh** každé 2 s, s připojením `, N failed` a `, N groups skipped`, pokud je hodnota nenulová (přeskočená skupina je taková, kterou pole zveřejnilo, ale u které nedošlo k žádnému zachycení):

  ```
     12.0s  880 captures (73.3/s), 7920 files
  ```

- **Žádný výpis podle jednotlivých souborů.** Spuštění končí jedním souhrnným řádkem — zachycené soubory, rychlost, soubory a výstupní složka — následovaným kódy `N file(s) failed to write.` / `N group(s) skipped.`, pokud jsou nenulové:

  ```
  Done — 880 captures (73.3/s), 7920 files in /home/user/flightline.
  ```

- **Zastaví se** v případě zaplněného disku s kódem `Continuous capture stopped: the drive is full.` a v případě chyby nebo odpojení pole s kódem `Continuous capture stopped: <reason>.` (oba na stderr).
- **Stav ukončení:** `0`, pokud byly uloženy snímky a byly zapsány všechny soubory, včetně `Ctrl+C`; `1` v případě, že se nepodařilo zapsat jakýkoli soubor, při zaplněném disku, chybě, odpojení pole nebo při úplné absenci záznamu.
- **Microsoft Defender:** pokud Defender při skenování souborů, které záznam zapisuje, zabírá jedno jádro procesoru nebo více, spustí se při běhu jednou na stderr výpis: `Note: Microsoft Defender is using N.N CPU cores scanning the capture files, CPU the capture needs. Re-run the Chloros installer with "Exclude the Chloros backend from Microsoft Defender scanning" checked.` Viz [Systémová nastavení](#windows-exe).
- Najednou je možné provádět pouze jeden nepřetržitý záznam na pole; druhý bude odmítnut s chybou `a continuous capture is already running for this array`. Přes HTTP je relace `POST /api/camera/array/continuous/start` (tělo `/api/camera/array/capture` plus `max_captures` / `duration_s`), `GET /api/camera/array/continuous/status?array_id=ID` a `POST /api/camera/array/continuous/stop`.

**Interval**(`--interval S`, s nebo bez `--continuous`) a**`--continuous --smart`** zachovávají smyčku na jeden požadavek: jeden požadavek na zachycení na jeden průchod — v každém naplánovaném okamžiku pro režim Interval, po ustálení AE pro `--smart` — se soubory každého průchodu uvedenými jako v režimu Single, řádkem `Done — N capture pass(es).` na konci a `--count N` počítajícím průchody.

### `array-record` — video/GIF s kombinovaným indexem (pro monitorování)

Zaznamenává **živý kombinovaný indexový náhled** pole do souboru `.avi` (a volitelně do souboru `.gif`). Je-li otevřen kombinovaný náhled v grafickém rozhraní, zaznamená tento náhled tak, jak je zobrazen. Pokud není otevřeno žádné okno, backend vykreslí zobrazení sám: index, který předáte pomocí `--expr`, nebo ve výchozím nastavení NDVI pole, jakmile je pole zarovnáno. Výchozí NDVI používá červené a NIR pásmo nejblíže 660 nm a 850 nm, stejně jako živý náhled. Náhled v grafickém uživatelském rozhraní otevřený během záznamu se zaznamenává, dokud zůstává otevřený. Každé 2 s zjišťuje stav a zastaví se na `--duration`, `Ctrl+C`, nebo když se záznamník sám ukončí.

Záznam, při kterém nedojde k zápisu žádného snímku, selže: záznamník nahlásí kód `no-frames: …` (pole se možná stále zarovnává nebo stabilizuje expozici) a příkaz **ukončí činnost s kódem `1`**.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF (no GUI needed)
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# Record a chosen index instead of the default NDVI
chloros-cli lattice array-record --duration 30 \
  --expr "(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)" -o monitoring/
```

| Příznak | Výchozí hodnota | Popis |
| --- | --- | --- |
| `--array-id ID` | pouze pole | Cílové pole (vynechte, pokud je připojeno pouze jedno). |
| `-o, --output DIR` | `output` | Výstupní adresář. Relativní cesta se nachází ve složce, ze které spouštíte příkaz. |
| `--fps F` | `10` | Snímková frekvence záznamu. |
| `--duration S` | do stisknutí Ctrl+C | Automatické zastavení po `S` sekundách. |
| `--gif` | vypnuto | Zapsat také animovaný soubor GIF. |
| `--gif-only` | vypnuto | Zapsat pouze GIF (bez `.avi`). |
| `--expr EXPR` | NDVI pole | Indexový výraz pro záznam, pokud není otevřeno žádné grafické rozhraní, v názvech pásem pole (např. `'(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)'`; viz [názvy pásem](#index--vegetation-maths)). Krátký název barvy (`NIR`, `Red`) funguje pouze tehdy, pokud jej má jedno pásmo pole. Je-li otevřeno okno GUI, zaznamená se místo toho jeho zobrazení. |

### `array-burst` — sériové snímání raw-Bayer s vysokou snímkovou frekvencí (pro analytické účely)

Čte přímo synchronizovaný skupinový buffer snímací smyčky — **není potřeba kalibrační řetězec, exiftool ani živý náhled** — takže běží při plné snímací frekvenci fotoaparátu. Zapisuje snímky ve formátu RAW + manifest pro každý snímek + jeden soubor `.daq` pro každé odlišné čtení DLS pod `<output>/bursts/<base>/`. Zpracujte offline (další příkaz) nebo předávejte `--build`, aby se to provedlo okamžitě po zastavení.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Příznak | Výchozí | Popis |
| --- | --- | --- |
| `--array-id ID` | pouze pole | Cílové pole. |
| `-o, --output DIR` | `output` | Výstupní adresář (výstup se ukládá do `<DIR>/bursts/<base>/`). |
| `--duration S` | do stisknutí Ctrl+C | Automatické zastavení po `S` sekundách. |
| `--max-frames N` | neomezeno | Automatickézastavení po `N` surových snímcích. |
| `--orient` | vypnuto | Výstupy vytvořené sestavením z této série snímků se otáčejí podle orientace kamery — výstupy jednotlivých kamer podle orientace jejich vlastní kamery, kombinovaný index podle orientace hlavní kamery — stejně jako to dělá `array-capture --orient`. Surové snímky zůstávají v původním formátu; `manifest.json` této série zaznamenává orientaci každého snímku a zvolenou možnost. |
| `--build` | vypnuto | Po zastavení okamžitě znovu zpracovat sérii (stejně jako `array-build-video`). |
| `--products …` | `combined:index` | S `--build`: které video(a) sestavit (viz níže). |
| `--fps F` | `10` | Spolu s `--build`: počet snímků za sekundu (fps) výstupního videa. |
| `--save-tiffs` | vypnuto | S `--build`: ukládat také kalibrované soubory TIFF pro každý snímek (každý s vlastní [maskou omezovače unmix](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on), pokud byl pixel oříznut; výsledek kompilace je uvádí jako `clamp_masks`). |
| `--gif` | vypnuto | S `--build`: také zapisuje animované soubory GIF. |

### `array-build-video` — offline přepracování uložené série snímků

Časově přiřadí každý surový snímek k nejbližšímu uloženému měření `.daq` a provede jej **stejným řetězcem radiance / odrazivosti / indexu jako importní pipeline**, čímž vykreslí jedno nebo více videí.

`--products` je seznam položek `kind:level` oddělených čárkami, kde `kind` ∈ `per_cam` | `combined` a `level` ∈ `radiance` | `reflectance` | `index`. Samotný `level` (bez `kind:`) má výchozí hodnotu `per_cam`. Výchozí hodnota je `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Příznak | Výchozí hodnota | Popis |
| --- | --- | --- |
| `--burst-dir DIR` | (povinné) | Cesta ke složce burst (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | Seznam `kind:level`, viz výše. |
| `--fps F` | `10` | Počet snímků za sekundu (fps) výstupního videa. |
| `--save-tiffs` | vypnuto | Uložit také kalibrované soubory TIFF pro každý snímek společně s videem (videa). |
| `--gif` | off | Zápis animovaných souborů GIF. |

**Video neobsahuje žádnou omezovací masku** (jeden obrazový proud, 8 bitů). Místo toho výsledek sestavení obsahuje `clamp_fractions`: pro každý video výstup podíl pixelů každého snímku, které byly při odstraňování smíšení oříznuty v pásmu, které obsahuje jeho kanál R / G / B — hodnotu `Chloros:Band{R,G,B}UnmixClampFraction`, kterou by byl označen produkt radiance/reflektance daného snímku (pro kombinovaný index, pro každou kameru v sestavě). Sestavení vypíše nejhorší snímek pro každý produkt na řádku `Clamp:`. `--save-tiffs` přidává masky pro jednotlivé pixely pro soubory TIFF, v souboru `clamp_masks/<level>/` vedle složky `<level>/` s TIFF soubory.**Předčasně ukončená kompilace nezanechá žádné poškozené video.** Každé video se zapisuje jako `<name>.partial.avi` a svůj konečný název získá až po dokončení, takže starší video s tímto názvem zůstane zachováno, dokud nebude jeho náhrada kompletní. Spusťte sestavení znovu, abyste nahradili video `.partial.avi`, které zůstalo nedokončené.

Běh vypíše každé video, které zapsal, poté jakýkoli produkt `Skipped:` (požadovaná kamera/úroveň – nebo kombinovaný index –, který nevytvořil žádné video, s uvedením důvodu – např. chybějící kalibrační balíček, chybějící DLS měření odrazivosti nebo chybějící pár pásem červené a NIR pro kombinovaný index) a všechny položky typu `Note:`. RGB kamera, u které byla požadována radiance nebo odrazivost, místo toho obdrží své zobrazovací video s vyvážením bílé, pokud je k dispozici kalibrační balíček kamery; kamera vykreslená bez něj je uvedena v záznamu `Note:`. Funkce **vrací nenulovou hodnotu**, pokud byl požadovaný produkt přeskočen nebo nebyl vytvořen podle požadavku (konfigurovaný indexový výraz není vyhodnocován offline); videa, která byla vytvořena, jsou i tak zapsána a uvedena v seznamu.

> **Vyberte správný záznamník.** `array-record` je určen pro *monitorování* — zaznamenává vykreslený živý kompozitní obraz: kombinovaný pohled z grafického uživatelského rozhraní (GUI) nebo, pokud není otevřeno žádné GUI, vlastní pohled backendu NDVI (nebo `--expr`). `array-burst` → `array-build-video` je určen pro *analýzu* — ukládá surová data ze senzoru v plné rychlosti a následně rekonstruuje kalibrovaná videa s radiancí/reflektancí/indexem, aniž by byl vyžadován živý náhled.

### Mono (M3M) jednopásmové kamery

Řada **M3M**je monochromatickou variantou modelu Bayer**M3C**: jedna úzkopásmová interferenční filtrační mřížka na kameru (`M3M-<lens>-F<wavelength>`, např. `M3M-L87-F685`), takže snímač poskytuje**jediný pásmo v odstínech šedé** bez Bayerovy mozaiky. Není co demosaikovat, není třeba odstraňovat přeslech mezi kanály a není třeba nastavovat vyvážení bílé — celý proces zpracování barev pro zobrazení v RGB se zde jednoduše neuplatňuje.

Co to znamená v CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**rozpoznají černobílou kameru a**přeskočí ji s jednořádkovou zprávou**, místo aby vnucovaly nesmyslná nastavení. V téže relaci stále fungují normálně s kamerou typu RGB/Bayer M3C.
- **Formát pixelů Bayer určuje jejich hloubku.** M3M nabízí pouze monochromatické formáty, takže formát Bayer z `--pixel-format` nebo `--preset` nastaví monochromatický formát stejné hloubky: `BayerRG8` a `--preset high_speed` dávají `Mono8`; `BayerRG12`, `high_quality` a `triggered` dávají `Mono12`; zkomprimovaný název zůstává zkomprimovaný (`BayerRG12p` dává `Mono12p`). Soubory a `lattice format` označují formát Mono. Pole, které kombinuje kamery M3C a M3M, používá jeden formát stejným způsobem: Bayer na každém M3C, jeho hloubka Mono na každém M3M. Název odpovídá jeho bitové hloubce: totéž platí i opačně, takže M3C se nikdy neuloží ve formátu Mono (`--pixel-format Mono12` je `BayerRG12` na M3C, bez ohledu na to, která kamera je hlavním prvkem pole), a `lattice format --set` se mapuje stejným způsobem a vytiskne formát, který nastavil.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** stále fungují — radiance a odrazivost jsou radiometrické mapy *na jedno pásmo* a jsou pro jedno pásmo dokonale dobře definovány. Monochromatické snímky nesou **identickou** matici odezvy snímače (bez 3×3 rozkladu), takže rovina prochází kalibračními výpočty beze změny.
- **Jedna monochromatická kamera nemůže vypočítat vegetační index.**NDVI, NDRE a podobné indexy vyžadují alespoň dvě pásma (např. Red + NIR). Chcete-li získat index z monochromatického hardwaru, nasměrujte**několik** kamer M3M na různé vlnové délky, sjednoťte je do jednoho vícepásmového souboru a vypočítejte index *z toho*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Mono_660 --channel nir=Mono_850 \
  --save-multiband -o output/
```

`index --live` otevírá kamery sám (jako [dočasnou relaci](#multi-camera-transient-sessions)), takže nejprve nic nepřipojujte: `--serials`, jinak by se aktivovaly kamery v profilu (viz [Index / Vegetation Maths](#index--vegetation-maths)). Pokud Chloros drží jakoukoli kameru (GUI, `cam-connect`, `array-connect`) je žádost odmítnuta s chybou HTTP 409 `CAMERA_HELD` a program se ukončí `1`; uvolněte je pomocí `chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`.

`--channel` symboly se musí **přesně** shodovat s názvy kanálů v předvolbě (rozlišuje se velká a malá písmena; názvy NDVI jsou psány malými písmeny `red`, `nir` — viz `--list-presets`) a název strany pásma označuje pásmo ve zarovnaném stohu (režim offline také akceptuje indexy pásem počínající od 0, např. `--channel red=0 --channel nir=1`).

Rozlišovacím znakem v celém stohu je token `M3M` v řetězci modelu (nikdy se nevyskytuje v řetězci `M3C`), který se v grafickém rozhraní (GUI) a SDKu zobrazuje jako `is_mono`.

---

## Nastavení a ladění síťové karty hostitele (systémy LATTICE)

Kamery LATTICE streamují GVSP přes ethernetový adaptér hostitele, takže u systémů s více kamerami záleží na **ovladači**a**velikost přijímacího kruhu** jsou stejně důležité jako přenosová rychlost. Nesprávná nastavení se projeví jako `FRAMES WILL DROP` / `Reduce ROI to enable` v panelu Nastavení pole (a v `lattice network-analysis` / SDK `analyze_array_network()`), a to i v případě, že samotné kamery fungují správně.

### USB 10GbE adaptéry — Realtek RTL8157 („Realtek USB 10GbE Family Controller“)

Níže uvedená nastavení jsou vlastnosti ovladače pro systém Windows. V systému Linux má USB adaptér také omezení (400 MB/s na x86_64, 200 MB/s na NVIDIA Jetson) (viz [Model fps a burstu pole](#array-fps--burst-model)), a pro každý adaptér hrají roli přijímací buffery soketů jádra — viz [Hostitelské systémy Linux](#linux-hosts--socket-receive-buffers).

| Položka | Požadovaná hodnota | Proč je to důležité |
| --- | --- | --- |
| **Verze ovladače**|**≥ v10.67 (leden 2026)**, INF `rtump64x64sta.inf` | Starší ovladač z roku**2016**(v10.65, `rtump64x64.inf`) nesprávně zpracovává vypínání a chybové kontroly s**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**při vypínání/restartu/uspání. Přechod se zasekne (časový limit ~5 minut), uživatel provede vynucené vypnutí a opakované nečisté vypnutí**poškodí úložiště WMI**(PowerShell a nástroje začnou selhávat s chybou `Invalid class`) a**zablokují USB stack** při dalším spuštění (síťová karta se neaktivuje; USB disky přestanou být rozpoznávány). Před provedením čistého restartu si stáhněte aktualizaci z realtek.com (nebo od výrobce hardwarového klíče). |
| **Příjmové vyrovnávací paměti**— klíčové slovo `ReceiveBufferLen` |**256**(maximum ovladače) | Přijímací kruh síťové karty. Výchozí hodnota ovladače**32**ponechává pouze ~0,26 MB použitelného kruhu — což je pro sérii dat z více kamer příliš málo — takže panel pole hlásí chybu `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` a blokuje připojení. Při hodnotě**256**je kruh velký (**~13,5 MB naměřeno na laboratorním hostiteli s 10GbE**), což dává přijímacímu potrubí skutečnou rezervu pro sériové přenosy z více kamer v režimu GVSP. (O tom, zda se daná konfigurace skutečně *připojí*, rozhodují dvě kontroly — **kontrola vyprázdnění**a kontrola**agregovaného nadměrného předplatného** — nejde o pouhé porovnání burstu s prstencem; viz [Model fps a burstů pole](#array-fps--burst-model).) |
| **Příjmové URB**— klíčové slovo `PendingReceives` |**64** (max.) | Bloky USB požadavků v přenosu; zvyšujte společně s přijímacími vyrovnávacími paměťmi pro absorpci burstů. |
| **Jumbo rámec** — klíčové slovo `*JumboPacket` | **9014** | Potřebné pro pakety GVSP o velikosti 9000 bajtů (6× méně paketů na rámec než u 1500). |

> ⚠️ **Aktualizace ovladače síťové karty RESETUJE tyto pokročilé vlastnosti na výchozí hodnoty.**Po aktualizaci nebo výměně ovladače adaptéru**znovu použijte** `ReceiveBufferLen=256` a `PendingReceives=64`, jinak se panel pole opět zablokuje, i když se „na hardwaru nic nezměnilo“. To je hlavní příčina toho, že dříve funkční zařízení náhle odmítá navázat připojení.

Použijte z **PowerShellu s oprávněními správce** (nahraďte název svého adaptéru, např. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` pokrývá adaptéry USB 10GbE.** Nyní rozpozná typ adaptéru a nastaví správné klíčové slovo pro přijímací kruh: `*ReceiveBuffers`→2048 pro síťové karty PCIe (Intel I219, atd.) nebo `ReceiveBufferLen`→256 + `PendingReceives`→64 pro **USB** 10GbE řadič Realtek (který nezveřejňuje `*ReceiveBuffers`). Hodnoty jsou omezeny na maximální hodnotu hlášenou jednotlivými ovladači (`NumericParameterMaxValue`), takže se nikdy nezapíše hodnota mimo rozsah. Spusťte to z terminálu s **právo**; stejně jako u jakéhokoli ladění založeného na registru se změna projeví až po restartu adaptéru nebo restartu počítače. Ruční příkazy `Set-NetAdapterAdvancedProperty` uvedené výše zůstávají dobrou alternativou — uplatní se okamžitě (přepojí adaptér) bez nutnosti restartu.

### Linuxové hostitelské systémy — přijímací vyrovnávací paměti soketů

V systému Linux, včetně NVIDIA Jetson, mají přijímací vyrovnávací paměti soketů jádra stejný význam jako přijímací kruh síťové karty. Při výchozím nastavení jádra (`net.core.rmem_max` / `net.core.rmem_default` = 212 992 bajtů) přeteče proud paketů GVSP z kamery do streamovacího soketu ještě předtím, než je Chloros stihne přečíst. Pakety jsou v jádře zahozeny, zatímco samotná síťová karta nehlásí žádnou ztrátu, a pole kamer ztratí skupiny snímků a sníží svou snímkovou frekvenci. Při měření na poli se 4 kamerami připojenými přes adaptér USB 10 GbE zvýšení obou hodnot na 32 MB zvýšilo snímkovou frekvenci pole po připojení z 4,96 na 7,52 fps a rychlost nepřetržitého záznamu surových dat ze 4,7 na 7,6 snímků/s, přičemž v jádře nedošlo k žádnému zahození paketů.

- **Soubor .deb tyto hodnoty nastavuje.** Instalace balíčku zapíše hodnoty `/etc/sysctl.d/60-chloros-gige.conf` (obě na hodnotu `33554432`) a okamžitě je aplikuje. Hodnota, která je již nastavena na vyšší hodnotu, zůstane beze změny.
- **`lattice network` je kontroluje.** Pokud je některá z hodnot nižší, zobrazí se `socket receive buffers too small for GigE Vision (…)` v sekci *Nalezené problémy* a protokol backendu obsahuje stejné varování při prvním připojení kamery nebo pole.
- **Chcete-li je zvýšit ručně** (do dalšího restartu; soubor balíčku je aplikuje při každém spuštění):

```bash
  sudo sysctl -w net.core.rmem_max=33554432 net.core.rmem_default=33554432
  ```

  `lattice network --fix` je také zvýší a udrží, ale pouze pokud backend běží jako root. Backend, který běží pod vaším uživatelským účtem (spuštěný s `chloros-backend` nebo [`chloros-backend@<user>`](#spuštění-backendu-jako-služby-bez-grafického-rozhraní)) nemůže měnit nastavení jádra a místo toho vypíše řádek `sudo sysctl`.

**Automatické ladění adaptérů se týká pouze připojených adaptérů.** Při prvním připojení kamery backend běžící jako root vyladí kabelové adaptéry, jejichž přijímací kruh je malý. Nyní ladí pouze adaptéry s aktivním spojením, takže odpojený integrovaný port si zachová svou hodnotu MTU a přijímací vyrovnávací paměti zůstávají beze změny. `lattice network` již neuvádí MTU odpojeného adaptérujako problém. `lattice network`, `--fix`, `lattice probe` a ladění adaptérů zohledňují pouze kabelové ethernetové porty, nikoli s CAN sběrnicí Jetsonu (`can0`) ani s USB spojeními v režimu zařízení (`usb0`, `usb1`), které nabízí hostitelskému PC; USB ethernetový adaptér pro kamery se stále počítá.

### Základy sítě (všechna propojení LATTICE)

- **Adresování:** link-local `169.254.0.0/16` (GigE Vision LLA). Hostitel používá statickou adresu `169.254.x.x/16`; kamery a DAQ-E si samy přidělují adresy ve stejném rozsahu. Není vyžadován DHCP ani brána.
- **Velikost paketu:**upřednostňujte jumbo (9000), ale nechte ji určit automatickým testováním — pomocí testu GVSP obejde 1500-bajtový limit ICMP kamery, takže se nastaví na jumbo tam, kde to kabel skutečně umožňuje.**Každá kamera se měří jednou za relaci backendu a znovu při změně jejího připojení (jiný adaptér, jiná rychlost připojení nebo přerušení spojení, jako například při výměně přepínače); při opětovném připojení se výsledek použije znovu. Po změně za přepínačem, který zůstal v provozu, restartujte backend, aby se měření provedlo znovu.**První připojení může při této kontrole občas trvat několik minut a protokol to v takovém případě zaznamená. Použijte fixaci (pin) s kódem `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` pouze tehdy, pokud víte více než sonda, a upřednostňujte fixaci na jednotlivé příkazy před trvalou: fixace přeskočí sondu, takže pokud cesta ve skutečnosti nedokáže přenést 9000,**každé** zachycení vyprší s chybou `SC_ERR_TIMEOUT -1011` (viz [Proměnné prostředí](#environment-variables)).
- **Velikost kruhu RX se mění s hodnotou `ReceiveBufferLen`:**při výchozí hodnotě `32` je použitelný prstenec ~0,26 MB (příliš malý pro jakýkoli burst z více kamer); při maximální hodnotě `256` je velký (~13,5 MB naměřeno na laboratorním hostiteli s 10GbE), což poskytuje skutečnou rezervu. O tom, zda se konfigurace připojí, pak rozhoduje kontrola připustnosti zohledňující odběr**a** níže uvedená kontrola agregovaného nadměrného předplatného — nejde o pouhé srovnání burstu s prstencem.

### Model fps a burst pole

Jak číst panel Nastavení pole (a `lattice analyze-array` / `analyze_array_network` v SDK):

- **Burst se sčítá pro každou kameru zvlášť v reálném formátu pixelů dané kamery.**Mono kamery**M3M**streamují**Mono12 (2 B/px)**, nebo s `--pixel-format` mono formát své hloubky (`BayerRG8` je `Mono8`, 1 bit na pixel); kamery**M3C**s filtrem Bayer vysílají 8- nebo 12bitový signál (firmware M3C tiše vysílá BayerRG12, i když je požadován BayerRG8). Snímek v plném rozlišení ze 4 kamer tedy má**~12,6 MB, pokud jsou všechny 8bitové, ale ~25 MB se třemi 12bitovými mono kamerami**. Projekce určuje formát každé kamery na základě jejího modelu (identity cache), takže datový proud odpovídá tomu, co se skutečně přenáší po kabelu — nejde o paušální předpoklad formátu BayerRG8.
- **USB ethernetový adaptér je omezen na 200 MB/s bez ohledu na údaje na štítku.** Tabulka účinnosti, která převádí rychlost spojení na trvalou hodnotu, je odvozena od PCIe; USB síťová karta inzeruje svou *ethernetovou* rychlost spojení, ale je omezena sběrnicí USB a jejím ovladačem. USB 10GbE adaptér dosahoval ~1063 MB/s „trvalou“ rychlostí – hodnotu, která nebyla nikdy ověřena – a výsledné omezení rychlosti poškodilo 6–18 % rámců, přičemž stále vykazovalo správnou cílovou hodnotu fps. Síťové karty připojené přes USB jsou nyní omezeny absolutně: **200 MB/s** ve Windows a na NVIDIA Jetson, **400 MB/s** v systému Linux x86_64 (v systému Linux je adaptér připojený k portu USB 2.0 omezen na **30 MB/s** a v popisu specifikací se doporučuje připojit jej k portu USB 3) (omezení je dáno USB sběrnicí a ovladačem hostitelského počítače, takže se nezvyšuje s uvedenými technickými parametry; adaptér USB 1 GbE dosahuje rychlosti ~80 MB/s a není tímto omezen). V systému Linux je adaptér identifikován prostřednictvím sysfs (s náhradním označením `ethtool -i` pro `bus-info`). Každá hodnota je naměřena: ve Windows u stejné pole 4 kamer poškodilo 6–10 % snímků při rychlosti nad 200 MB/s; v Linux u x86_64 fungoval stejný adaptér bezchybně při každé maximální rychlosti až do 480 MB/s (dosahovaná rychlost 273 MB/s, což je vlastní limit kamer). Na platformě NVIDIA Jetson fungoval stejný adaptér bez chyb pouze při 200 MB/s a při vyšších rychlostech docházelo k přerušovanému ztrácení snímků. `wire_ceiling_source` v záznamu o schopnostech to uvádí slovy a `nic_is_usb` na to upozorňuje. V obou případech lze nastavení přepsat pomocí `--wire-ceiling-mbps`.
- **Přijímání zohledňuje odběr, nikoli porovnání celého burstu s kruhem.** Souběžný burst se musí vejít pouze do *dočasného backlogu* = `max(0, Σ per-cam arrival − host drain) × emit_window`, nikoli do celého burstu. V síti s rychlým hostitelem a pomalými kamerami (**PCIe**10G hostitel + 4× 1 GbE kamery: příchozí data ≈ 320 MB/s, odvod ≈ 1063 MB/s) se host vyprázdňuje rychleji, než se kamery naplňují, backlog ≈ 0, takže simulovaný výstup v plném rozlišení**je povolen**, i když 25 MB burst přesahuje 13,5 MB prsten. Umístěte tytéž čtyři kamery za**USB** 10GbE adaptér ve systémWindows
a odvod bude 200 MB/s, nikoli 1063 — příchozí data ho předstihnou a ztráta se projeví jako poškozené snímky, nikoli jako nižší snímková frekvence. Na hostitelském počítači s 1 GbE způsobuje minimální hodnota DLThr kamer 31,25 MB/s to, že příchozí data předstihují odvod → systém správně **blokuje** (pro *tuto* třídu bloků zmenšete ROI nebo použijte binning ≥ 2). Admittance je jednou ze **dvou** kontrolních bran – druhou je níže uvedená kontrola agregované nadměrné předplatnosti.
- **Předpokládaná snímková frekvence (fps) odpovídá cyklu nejpomalejší kamery.**Při simultánním vysílání každá kamera odesílá svůj snímek ve stejný okamžik, takže pole pracuje v cyklu, který odpovídá delší z následujících hodnot: doba čtení snímače nejpomalejšího člena pole a doba vysílání (počet bajtů snímku ÷ jeho vysílací rychlost:**přístupové připojení** kamery, 1 GbE ≈ 80 MB/s, nebo její podíl na maximální přenosové rychlosti hostitele, je-li tato nižší). Každá expozice se překrývá se čtením předchozího snímku, takže expozice nevstupuje do cyklu: automatická expozice vyplní periodu danou přenosovou rychlostí, poté zvýší zisk a tmavá scéna již nesnižuje přenosovou rychlost (ve verzi 1.2.0 tomu tak bylo). Pouze ruční expozice delší než tato perioda zpomaluje spouštění (viz [Expozice na členu pole](#exposure-on-an-array-member)). Přidání kamer nezpomalí pole, dokud se nezmenší jejich podíl na přenosu. V sériovém vysílání (`slip-emit-and-capture`) se místo toho sčítají časy vysílání kamer a expozice každého člena se promítá do časového rozložení. Měřeno při plném rozlišení snímače, 12 bitů, na poli s 9 kamerami s hostitelským počítačem PCIe 10 GbE: 10,7 fps, nebo 14,0 fps s [komprimovaným přenosem](#packed-12-bit-transport---packed).
- **Po nepřetržitém snímání projekce udává, jakou snímkovou frekvenci si udržela.** Snímání trvající 30 s nebo déle při automatické snímkové frekvenci zaznamenává frekvenci, na které se systém při dané velikosti pole na tomto počítači ustálil; krátké pokusy regulátoru snímkové frekvence o vyšší frekvenci jsou vynechány. Od té chvíle řádek `array-connect` a panel Nastavení pole uvádějí tuto rychlost, pokud je nižší než rychlost modelu, spolu s tím, co ji omezilo: `clock-bound` (pole běželo rychlostí, na které se ustálilo, smyčka snímání čekala na spouštěcí takt), `loop-bound` (smyčka snímání neměla dostatek času), `capture-bound` (snímání zaostávalo a přeskočilo skupiny) nebo `measured` (není uvedeno, o kterou se jedná). Hodnota modelu je uvedena v `wire_fps_max`. Pole stále stoupá až na hodnotu, kterou přenáší síť.
- **Pole funguje v jakékoli síti — záleží pouze na snímkové frekvenci.**Každá kamera dostává rovný podíl z**stropu pro přenos bez kolizí**(`sustained × sim_emit_factor`), synchronizovaného s GevSCPD, a všechny stále exponují na stejný spouštěcí signál. Na pomalém kabelu je tento podíl malý a s ním klesá i snímková frekvence: 8 kamer s plným rozlišením a 12bitovou hloubkou na 1 GbE bez jumbo rámců dostane každá 52 / 8 = 6,5 MB/s, což je asi**1 fps**(asi 1,3 fps v komprimovaném formátu), zatímco hostitel s 10G je provozuje rychlostí 11–14. Pokud kamery nelze synchronizovat tak pomalu — zjistěte to pomocí příkazu connect, který načte rozsah GevSCPD každé kamery — pole přepne na**sériový**výstup (`slip-emit-and-capture`): kamery se na lince střídají při plné rychlosti spojení, spouštějí se jedna po druhé, takže každý rámec stále dorazí, ale členy**nejsou vystaveny současně**. `analyze-array` a výstup příkazu connect označují tento režim; nastavte `--force-tier sim-capture-sim-emit` na „refuse“, chcete-li odmítnout přepnutí na náhradní režim. Velikost snímku do tohoto sdílení nezasahuje, takže**binning a menší oblast zájmu (ROI) zvyšují snímkovou frekvenci, ale nemění to, zda se pole vejde**. Pouze linka, která nedokáže přenášet pole ani sériově, je**přetížená**— např. mnohem více kamer, než kolik jedna linka zvládne, nebo mnoho dalších kamer vysílajících na ní — a v takovém případě se projevuje ztrátou paketů GVSP, nikoli nižší rychlostí: `analyze-array` vynuluje hodnoty dosažitelné snímkové frekvence (fps) a vypíše `** OVER-SUBSCRIBED**`, připojení pokračuje s hlasitým varováním a HTTP připojení s `pin_resolution: true` (přísný režim) **odmítne**. Řešením je snížit počet kamer, použít jumbo rámce v celém řetězci nebo použít rychlejší síťovou kartu. `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` zmírní odmítnutí na hlasité varování pro testovací účely — viz [Proměnné prostředí](#environment-variables).

### Stav pole — který subsystém ztrácí rámce

`GET /api/camera/array/<array_id>/capability` připojeného pole nese živý
blok `health`, který se přehodnocuje v průběžném **10sekundovém** okně. Rozděluje ztrátu rámců
na dvě příčiny, které vyžadují opačná řešení, namísto hlášení jedné „neúplné“
míry, která žádnou z nich neoznačuje:

| Pole | Co to znamená | Který subsystém |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (pro každý sériový port) | Rámec **dorazil, ale byl strukturálně poškozen**— ztráta paketů GVSP. |**Síť**: kapacita linky, tempo, přijímací kruh NIC, MTU |
| `never_arrived_rate_pct` (pro každý sériový port) | Rámec **vůbec nedorazil**— kamera se nespustila nebo z ní nic nevyšlo. |**Spoušť / synchronizace**: kabel M8, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Nejhorší míra neúplnosti pro každou kameru. | — |
| `per_cam_rate_pct` | Kombinovaná míra neúplnosti na kameru (obě příčiny dohromady). | — |
| `stable_for_seconds` | Jak dlouho každá kamera zůstala pod 0,01 %. | — |

Při hodnotě nad 5 % backend zaznamená do protokolu řádek `[array-health <id>] WARN` s názvem daného rozdělení — při
prvním porušení, při změně pásma závažnosti, jednou za minutu, dokud stav přetrvává, a jednou, když
se stav vyřeší. Poškozená polovina zapíše do logového souboru backendu řádek `[gvsp-corrupt <SN>]` při
prvním výskytu pro každou kameru a důvod, poté souhrn každých 60 s; panel logů aplikace místo toho zobrazuje řádek
místo toho řádek `[array-health <id>]`. Každé vyhodnocení se stále ukládá do backendového logového souboru;
počítadla se posouvají s každým bufferem bez ohledu na to, co se vypíše.

Stejný záznam uvádí číslo, na kterém visí celá alokace:

| Pole | Co to znamená |
| --- | --- |
| `wire_ceiling_mbps` | Aktuální trvalý přenosový limit hostitele, MB/s. |
| `wire_ceiling_source` | Odkud tato hodnota pochází, slovy — např. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` (na Linux u x86_64, `USB-capped 400 MB/s (…; sysfs: bus=usb (usb2/2-1/2-1:1.0))`) nebo `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, pokud jej nastavuje `--wire-ceiling-mbps` (nebo pole **Wire Budget** v grafickém rozhraní). |
| `nic_is_usb` | `true` pro USB ethernetový adaptér — viz výše uvedený limit pro USB. |

**Výklad:** nenulová hodnota `gvsp_corrupt_rate_pct` při hodnotě `never_arrived_rate_pct` rovné 0
znamená, že spouštění a synchronizace kabelu jsou v pořádku a 100 % ztráty je na síťové
trase — snižte hodnotu `--wire-ceiling-mbps` a znovu připojte. Opačný vzorec naopak ukazuje na
synchronizační kabel nebo spouštěcí linku.

> **Hodnota `--target-fps` není rozhodujícím faktorem pro poškozené rámce.** Frekvence GevSCPD se zapisuje
> pouze jednou při připojení, takže snížení spouštěcí frekvence mění pracovní cyklus, nikoli
> rychlost souběžného vysílání. Naměřené 5× snížení požadavků nepřineslo žádné zlepšení;
> snížení maximální přenosové rychlosti z 240 na 200 MB/s snížilo u stejného zařízení podíl poškozených dat z 10,4 %
> na 0,00 %.

> **Běžící pole se nemůže samo přeplánovat na firmwaru LATTICE (M3C / M3M).**
> Tuto situaci nedokáže sama vyřešit; odpojte a znovu připojte, aby nástroj pro výběr času připojení mohl
> provést nové plánování s novou maximální rychlostí.

### Příznak → řešení

| Příznak (Nastavení pole / připojení / `analyze_array_network`) | Příčina | Řešení |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` se resetuje na 32 (obvykle po aktualizaci ovladače) | Nastavte `ReceiveBufferLen`→256, `PendingReceives`→64; znovu otevřete panel (restartujte backend, pokud si uložil do mezipaměti starou velikost kruhu) |
| Zablokování při restartu/vypnutí; později chyby WMI `Invalid class`, NIC se nedá aktivovat, chybí USB disky | Starý ovladač Realtek USB 10GbE z roku 2016 → BSOD `0x9F` → vynucené vypnutí | Aktualizujte ovladač adaptéru na verzi ≥ v10.67 (2026), poté znovu použijte výše uvedená nastavení přijímacího kruhu |
| Pole se připojuje při plném využití senzoru, ale s nízkou snímkovou frekvencí | Režim „Auto“ udržuje plné využití senzoru a nechává frekvenci klesnout na úroveň, kterou unese kabel | Zvolte jednu z rychlejších možností uvedených v seznamu výstupů připojení (`--binning 2` / `4` nebo menší `--width`/`--height`), povolte jumbo rámce nebo vylepšete připojení |
| Pole hlásí správnou cílovou hodnotu fps, ale dodává jen zlomek této hodnoty; `health.gvsp_corrupt_rate_pct` nenulová hodnota, `never_arrived_rate_pct` 0 | Odhadovaný přenosový limit hostitele nadhodnocuje to, co ve skutečnosti zvládá (typické u USB ethernetového adaptéru, úzké PCIe linky nebo sdílené sítě) | Znovu se připojte s nižší hodnotou `--wire-ceiling-mbps` a znovu zkontrolujte stav bloku. **Ne** `--target-fps` — Pacing GevSCPD je při připojení pevně nastaven |
| Linux: pole ztrácí skupiny rámců a snižuje svou snímkovou frekvenci, zatímco síťová karta nehlásí žádné ztráty; `lattice network` uvádí `socket receive buffers too small` | Příjmové buffery soketů jádra jsou nastaveny na výchozí hodnotu, takže pakety z kamery jsou v jádře zahozeny | Zvyšte hodnotu `net.core.rmem_max` / `rmem_default` na 32 MB — viz [hostitelské systémy Linux](#linux-hosts--socket-receive-buffers). Balíček .deb to provede při instalaci |
| Kamery chybí ve zveřejněných skupinách; hodnota `health.never_arrived_rate_pct` je nenulová, `gvsp_corrupt_rate_pct` 0 | Spouštěcí / synchronizační cesta — kamery se nespouštějí, nejde o síťový problém | Zkontrolujte synchronizační kabel M8 a `--line`; ověřte, zda jsou všechny členy zapnuté (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` překročena v `analyze-array`, nebo odmítnutí připojení v přísném režimu (`pin_resolution`; `array over-subscribes the wire`) | Kabel nedokáže přenášet pole dat při jakékoli frekvenci, které kamery dosahují, ať už současně nebo postupně — je zde mnohem více kamer, než kolik kabel dokáže obsloužit, nebo na něm streamují i jiné kamery | Menší počet kamer, použití jumbo rámců v celém řetězci nebo rychlejší síťová karta. **ROI/binning nepomůže** (sdílení je nezávislé na velikosti rámce). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` přepisuje na testovacím stanovišti (akceptuje ztrátu paketů) |
| Pole se připojuje v `slip-emit-and-capture` (sériově) na pomalé síti | Kamery nelze nastavit na dostatečně pomalou synchronizaci, aby mohly sdílet kabel současně, takže se střídají; snímky nejsou pořizovány ve stejném okamžiku | Jumbo rámce nebo rychlejší síťová karta obnoví souběžné snímání; `--force-tier sim-capture-sim-emit` na tom trvá (a místo toho ztrácí pakety) |

---

## `chloros-cli daq`

Příkazy pro spektrální senzor. Příkazy `pool-*` jsou jednoduché HTTP klienty, které ovládají senzor prostřednictvím trvalého fondu backendu, takže potřebují spuštěný backend (viz [Spuštění backendu](#start-the-backend)). Backend vlastní transport, takže skripty CLI a SDK sdílejí jeden aktivní handle, místo aby se přetahovaly o sériový port. `pool-discover` je jediný způsob, jak zjistit BLE MAC zařízení DAQ-M.

> **`chloros-cli daq --help`** (nebo `-h`, nebo `daq help`) vypisuje podpříkazy `pool-*` (v systému Linux spolu s příkazy pro přímé ovládání hardwaru, které obsahuje sestavení), a `chloros-cli daq <subcommand> --help` (nebo `daq help <subcommand>`) uvádí možnosti každého z nich. Zahrnují připojení, streamování, záznam kalibrovaných souborů `.daq`, výměnu profilů kapacit a připojení DAQ k kameře a v případě DAQ-E také síťová nastavení, PTP, restart a aktualizace firmwaru. DAQ lze také ovládat z Python u pomocí `chloros_sdk.connect_daq_sensor()`, který využívá stejnou sdílenou cestu.
>
> **Používejte příkazy `pool-*` na všech platformách.** Chovají se stejně ve Windows u, Linux u i na Jetsonu. Ve Windows u jsou jsou `daq discover` a `daq list` aliasy příkazu `pool-discover` a jakéhokoli jiného příkazu `daq`, který není `pool-*` ukončí příkaz `1` s výstupem `DAQ direct-hardware commands require the full daq package`. V systému Linux a na platformě Jetson jsou příkazy `daq discover`, `daq list` a další příkazy `pool-*`, které nejsou typu `daq`, přímými-hardwarové příkazy: otevírají senzor ve vlastním procesu CLI, mimo fond backendu (tedy ne na senzoru, který drží backend, grafické rozhraní nebo skript), a `daq discover` tam přijímají `--only` a `--timeout`, ale ne `--json`. Tato příručka dokumentuje pouze příkazy `pool-*`.

### Postup prvního připojení senzoru DAQ

```bash
# 1. Scan every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U serial port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq pool-discover
chloros-cli daq pool-discover --only ble           # BLE only
chloros-cli daq pool-discover --json               # machine-readable

# 2. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect                                   # smart-detect
chloros-cli daq pool-connect --port /dev/ttyUSB0               # DAQ-U on a serial port (Windows: COM3)
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 3. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 4. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

#    Bind it to a camera, so that camera's captures use and save its readings
chloros-cli daq pool-bind --camera 213800234 CB-7C-A8-2E-5F

# 5. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 6. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Referenční příručka k `pool-*`

| Podpříkaz | Účel |
| --- | --- |
| `daq pool-connect` (smart-detect) | Otevře senzor v backendovém fondu. |
| `daq pool-connect --port PORT` (`-p`) | DAQ-U na konkrétním sériovém portu. |
| `daq pool-connect --ble` | DAQ-M přes BLE, automatické vyhledávání MAC adresy. |
| `daq pool-connect --mac MAC` (`-m`) | DAQ-M přes BLE na známou adresu MAC (vyžaduje `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E přes Ethernet na známém názvu hostitele nebo IP adrese (vyžaduje `--eth`). |
| `daq pool-connect --eth` | DAQ-E přes Ethernet, hostitel automaticky zjištěn (mDNS + záložní ARP; funguje i s prázdnou mezipamětí ARP ve Windows u a Linux u). |
| `daq pool-connect --integration-time MS (-t) --frame-avg N (-f) --no-ae` | Nastavení integračního okna / stavu AE. |
| `daq pool-connect --no-stream` | Připojit, ale zatím nespouštět streamování (pokračovat s `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Profil korekce Cap. Výchozí nastavení na backendu je `sunshine_cosine`; `none` je holý senzor. Senzor [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor) jej ignoruje (pevný difuzor). **Senzory se dodávají bez krytu, proto pojmenujte kryt, který jste nasadili:** výchozí hodnota se zaznamenává jako předpokládaná (`cap_id_source=auto_default`) a systém zpracování na to upozorní, protože holý DAQ-U při korekci slunečního záření vykazuje přibližně 30x vyšší hodnoty. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Prohledávejte každý transportní kanál pro senzory, ke kterým byste se mohli připojit, aniž byste se připojovali. **Takto zjistíte BLE MAC adresu modulu DAQ-M.** `--timeout` se nastavuje pro každý transport (výchozí 5 s, 1–20). Senzory, které jsou již otevřené v poolu, nejsou uvedeny – připojený DAQ-M přestane vysílat reklamy – proto pro ně použijte `pool-list`. `usb` vypisuje pouze sériová zařízení USB: vlastní sériové porty počítače (`COM1`, `/dev/ttyS*` v Linux u) nemohou být senzory a jsou vynechány. |
| `daq pool-list` | Zobrazí všechny senzory v backendovém fondu. |
| `daq pool-bind (--camera SERIAL \| --array ARRAY_ID) {SENSOR \| --clear}` | Připojí kameru nebo pole k DAQ, stejně jako to dělá výběr světelného senzoru v grafickém rozhraní: jeho snímání využívá, porovnává a ukládá naměřené hodnoty tohoto DAQ. `SENSOR` je ID senzoru z `pool-list` nebo adresa, na které je DAQ dostupný. `--clear` (ne `SENSOR`) zruší přiřazení a snímání se vrátí k nejnovějšímu DAQ. Vazba trvá, dokud se backend nerestartuje, nebo dokud není daná kamera, pole nebo DAQ odpojeno: odpojení (zde z SDK nebo v aplikaci) zruší všechny vazby na odpojený hardware, bez ohledu na to, kdo je nastavil, včetně vazeb členů pole na samotné pole. Kamera musí být připojena v otevřeném projektu nebo již dříve otevřena na tomto počítači a pole musí být připojeno nebo v otevřeném projektu; jakékoli jiné sériové číslo nebo ID je podle názvu odmítnuto a nic není navázáno. |
| `daq pool-disconnect {--sensor-id ID \| --all}` | Uvolnění — bez ohledu na to, kdo senzor otevřel, uvolní se tedy i senzor, který v poolu zanechal havarovaný skript (nejprve se dokončí veškeré nahrávání na něm). Senzor, který není v poolu, nemá co uvolnit: vypíše `ID: already released (not in the pool)` a ukončí se `0`; uvolnění, které backend odmítl nebo nepotvrdil, ukončí se `1`. SDK: `DAQSensorSession.force_disconnect()`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Posledních N spektrálních snímků (výchozí hodnota 1). |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Obnovit / pozastavit streamování. |
| `daq pool-record --sensor-id ID [--duration SEC] [-o DIR] [--device-name NAME] [--confirm-after S] [--heartbeat S]` / `--stop` | Spustit / zastavit záznam .daq. `-o` je na počítači backendu nastaveno jako výchozí na `~/Documents/DAQ Live View` (relativní cesta je pod složkou, ze které spouštíte příkaz); `--duration` (`-d`, výchozí hodnota `0` = až do `--stop`). `--confirm-after S` vydá varování, pokud S sekund po spuštění nepřijde žádná hodnota (výchozí hodnota 20; 0 tuto funkci vypne); s `--duration` `--heartbeat S` vypíše počet naměřených hodnot každých S sekund a upozorní, když přestane růst (výchozí hodnota 30; 0 tuto funkci vypne). Záznam na DAQ-E, který přestane vysílat, pokračuje ve stejném souboru, jakmile se backend znovu připojí, a `--stop` vypisuje mezery ([DAQ-E, který přestane dodávat data](#a-daq-e-that-stops-delivering)). Kombinace kláves Ctrl+C předčasně ukončí záznam `--duration` a nahlásí to; bez `--duration` nahrávání po stisknutí Ctrl+C pokračuje (ukončení `130`) až do `--stop`. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Přepnutí profilu korekce kondenzátoru za běhu. Je vyžadován `--cap-id`; `--cap-id none` odstraní korekci kondenzátoru (holý senzor). DAQ-E-S zachovává nastavení `as_recorded` bez ohledu na aktuální nastavení. |
| `daq pool-enable-ptp --sensor-id ID` | Znovu zapne PTP u zařízení DAQ-E, u kterého je PTP vypnuté. Nastavení se uloží do paměti zařízení, takže zařízení, u kterého je PTP vypnuté, zůstane vypnuté — jeho časová razítka nejsou synchronizována s PTP — dokud se tento příkaz nespustí. `pool-list` takové zařízení označí (`!! PTP is switched OFF`) a vypíše pro něj tento příkaz. Chloros nikdy zařízení znovu nezapne sám, protože vypnutí může být záměrné. |
| `daq pool-info --sensor-id ID [--json]` | Vlastní nastavení jednotky DAQ-E: její firmware (a zda tato verze Chloros u obsahuje novější verzi), její adresa a způsob jejího získání (DHCP / link-local / statická), uložené síťové nastavení platné do doby, než jej senzor spustí (včetně nastavení uloženého pomocí `--no-reboot`), datový proud při zapnutí a PTP. U senzoru, jehož datový proud nedosahuje tohoto počítače, program při novém připojení senzor načte a oznámí to. |
| `daq pool-set-network --sensor-id ID {--auto \| --static IP [--mask M] [--gateway G]} [--no-reboot] [--force] [--timeout S]` | Nastavení DAQ-E na DHCP (`--auto`) nebo statickou adresu (`--mask`, výchozí `255.255.255.0`). Senzor se restartuje, aby nastavení uplatnil, a znovu se připojí s původními nastaveními, přičemž na to čeká až `--timeout` sekund (výchozí hodnota 45); `--no-reboot` ji pouze uloží. Viz [Nastavení zařízení DAQ-E](#daq-e-device-settings). |
| `daq pool-set-auto-stream --sensor-id ID {--on \| --off}` | Určuje, zda zařízení DAQ-E zahájí přenos dat při zapnutí, aniž by byl připojený software (uloženo na senzoru). Aplikace Chloros spustí přenos v obou případech sama. |
| `daq pool-reboot --sensor-id ID [--timeout S]` | Restartuje zařízení DAQ-E a znovu jej připojí s předchozími nastaveními (čeká maximálně `--timeout`, výchozí hodnota 45 s). |
| `daq pool-update-firmware --sensor-id ID [--timeout S]` | Nahrajte firmware zařízení DAQ-E dodaný s touto verzí Chloros u do senzoru s starší verzí a poté jej znovu připojte (trvá to asi minutu; nevypínejte jej; `--timeout` výchozí doba 45 s, než se znovu připojí). Senzor, který již má aktuální verzi, se nechá být. |

### DAQ-E, který přestane vysílat

DAQ-E v bazénu, který po dobu 20 sekund nevysílá žádné spektrum, je backendem znovu připojen. Toho je dosaženo vypnutím a zapnutím přepínače, který napájí senzor: senzor se s ním restartuje a znovu nezačne vysílat, dokud k tomu nedostane pokyn (pokud není nastaveno `pool-set-auto-stream --on`).

- **Nejprve ověří, zda se jedná o stejný senzor.** Na adrese, kterou senzor měl, a bez jakýchkoli změn, se dotáže na název senzoru, MAC a sériové číslo. Dotazuje se také pod názvem senzoru `daq-e-<id>.local`, ale této odpovědi důvěřuje pouze tehdy, pokud pochází z vlastního síťového adaptéru (MAC) senzoru. Žádné jiné zařízení, které odpoví, není znovu připojeno ani do něj není zapisováno.
- **Znovu se připojí s nastavením relace:** expoziční čas, průměrování snímků, automatická expozice a profil omezovače zůstávají stejné, nikoli výchozí nastavení snímače po zapnutí. Snímač, který měl zapnutý protokol PTP a vrátil se s vypnutým protokolem, se znovu zapne. Poté se restartuje datový proud a přijímač počítače pro něj.
- **Opakuje pokusy v prodlužujících se intervalech:** okamžitě, poté po 5, 10, 20 a 40 sekundách, následně každých 60 sekund, dokud je snímač nedostupný. Při každém pokusu se senzor pouze dotazuje, o jaký senzor se jedná. Senzor, který odpoví, ale i po třech pokusech o opětovné připojení stále nic neodesílá, je ponechán bez dalšího zásahu a nahlášen (`auto-reconnect GAVE UP`). `pool-stream --start` to zkusí znovu.
- **`pool-list` ukazuje aktuální stav.** Zatímco senzor mlčí, jeho řádek `STREAM FAULT` informuje o tom, co se při opětovném připojování děje. Řádek se vymaže s prvním spektrem a senzor poté vykazuje hodnotu `reconnected automatically N time(s)`.
- **Záznam pokračuje ve stejném `.daq`.** Soubor označuje každou mezeru dvěma řádky: `event_type` 5 u posledního spektra před mezerou a 6 u prvního po ní, každý pouze s časovým razítkem. `pool-record --duration` vypíše `!! STREAM LOST …`, když je snímač pryč, a `stream resumed after …`, když se vrátí. `pool-record --stop` vypisuje každou mezeru a vydává varování, stejně jako v případě prázdný soubor (záznam, který neobsahuje žádná měření, ukončí se výstupem `1`).
- **Záznam si zachovává jednu kalibraci.** Pokud se snímač během běhu záznamu vrátí s odlišnou kalibrací, zbytek záznamu si zachová tu, se kterou začal, a varování při zastavení na to upozorní. `pool-disconnect` + `pool-connect` převezme tu novou.
- **Vaše příkazy mají přednost.** Senzor, u kterého zadáte `pool-disconnect` nebo pozastavíte pomocí `pool-stream --stop`, se nikdy znovu nepřipojí, a to ani v případě, že je již v procesu opětovného připojení.
- **`pool-info` na neaktivním senzoru** načte senzor při novém připojení a přidá `!! stream NOT reaching this computer`. Senzor, který neodpovídá, je okamžitě nahlášen. `pool-list` již nezobrazuje stav PTP pro senzor, který přestal odpovídat.
- DAQ-U a DAQ-M nejsou zahrnuty. DAQ-M se přes Bluetooth připojí znovu sám, zatímco DAQ-U odpojený od USB vyžaduje opět `pool-connect`.

### DAQ-E-S — pevný difuzor a snímač sklonu

**DAQ-E-S** je zařízení DAQ-E se dvěma rozdíly, které mají význam pro software. Vše ostatní — vyhledávání, připojování, PTP, záznam, níže uvedená nastavení zařízení — funguje přesně stejně jako u DAQ-E.**Jeho difuzor je pevně zabudovaný, takže nevyžaduje žádnou korekci krytu.** Difuzor typu Sunshine je zabudovaný a byl na jednotce již při měření její tovární kalibrace, takže tato kalibrace již udává skutečné sálavé záření směřující dolů. Chloros nastavuje jeho kryt na `as_recorded` (bez profilu) bez ohledu na to, co je požadováno: kryt pojmenovaný podle něj (`sunshine_cosine`, `fov_*`, `none`) je ignorován s hláškou a jeho záznamy ukládají `cap_id_source=model`, takže se u něj nikdy nespustí varování o předpokládaném omezení. Nepoužívejte na jeho data ani ruční omezení: `sunshine_cosine` by v horní poloze vykazoval asi 11násobně vyšší hodnoty.

**Je vybaven tříosým akcelerometrem.** Kosínusová odezva difuzoru slunečního záření platí pouze tehdy, je-li v horizontální poloze, takže nakloněný senzor hlásí nesprávnou intenzitu záření — a odrazivost tuto chybu přebírá — bez jakýchkoli dalších příznaků. Každá hodnota z DAQ-E-S proto obsahuje informaci o své orientaci:

| Pole | Význam |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Úhly ve stupních. `None`, pokud firmware označí úhel jako nedefinovaný (volný pád, nefunkční osa). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Zrychlení v mili-g. Hodnota `mag_mg` výrazně odlišná od ~1000 znamená, že se jednotka pohybovala a úhly pro daný vzorek nemají význam. |
| `cal_applied` | Nastaveno: osy a úhly jsou v souřadnicovém systému difuzoru (je použita kalibrace upevnění jednotky). Vymazáno: surový souřadnicový systém desky, který může vykazovat sklon až ~2,3°, který není skutečný. |
| `sample_age_ms` | Stáří vzorku z akcelerometru v okamžiku odeslání spektra. |
| `temp_raw` | Nekalibrovaný **trend** teploty čipu — pouze rozdíly, nikdy naměřená hodnota teploměru. |

`daq pool-latest --json` je vrací pod klíčem `imu` každého snímku (`null` na DAQ-U, DAQ-M nebo běžném DAQ-E). Záznam `.daq` z DAQ-E-S ukládá stejné hodnoty do sloupců `imu_*` (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) vedle každého spektra, takže čtenář může rozeznat mrak od převráceného stativu. U všech ostatních modelů jsou tyto sloupce prázdné.

Proud dat o poloze se vykazuje samostatně: `daq pool-list` vypíše `!! ATTITUDE STREAM FAULT (spectra unaffected)`, když je přístroj mimo provoz, a kolik poruch se mu poté podařilo vyřešit. Porucha v tomto proudu nikdy nezpůsobí mezeru ve spektrálním záznamu. `daq pool-discover` uvádí jednotku jako `DAQ-E-S` (jako `DAQ-E`, pokud neodpoví na kontrolu identity do cca 2 s).

### Nastavení zařízení DAQ-E

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330

# A fixed address on the network this computer is on (192.168.1.x)
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1

# Back to the factory setting: DHCP, falling back to a 169.254.x.x
# link-local address when no DHCP server answers
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto

chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

- **`--static` bez `--gateway` neukládá žádnou bránu** (jakákoli dříve nastavená brána se zruší), což je vhodné pro síť kamer bez směrovače.
- **Statická adresa musí být v síti tohoto počítače.** Chloros odmítne takovou, jejíž podsíť neobsahuje adresu tohoto počítače v síti senzoru: po restartu by se k senzoru nic nedostalo, aby bylo možné nastavení vrátit zpět. Nastavení `--force` použijte pouze v případě, že bude tomuto počítači přiřazena nová adresa nebo bude přesměrován do nové podsítě.
- **Restart je odmítnut, pokud snímač právě nahrává.** Nejprve zastavte nahrávání (`pool-record --stop`). Nahrávání spuštěné těsně před restartem je tímto restartem ukončeno, a `pool-set-network`, `pool-reboot` a `pool-update-firmware` končí řádkem `Warning:` s názvem souboru záznamu (i když se senzor poté nevrátí).
- **Po restartu Chloros znovu připojí pouze samotný senzor.** Čeká na místě, kde se senzor nacházel – a po řádku `--auto` také pod jeho názvem `daq-e-<id>.local` – na odpověď obsahující název senzoru, název hostitele a model, od síťového adaptéru (MAC), který měl, a znovu jej připojí na adresu, která odpověděla. Odpověď pod daným názvem je přijata pouze s touto adresou MAC, takže z počítače, který se k senzoru připojuje přes router, nebo z WSL2 v jeho výchozím NAT síťovém nastavení, se senzor, který byl přes DHCP přesunut na novou adresu, znovu nepřipojí: příkaz to oznámí a vypíše příkaz `pool-connect --eth-host <its address> …`, který jej obnoví s původními nastaveními citlivosti a expozice.
- **`pool-update-firmware` odmítne senzor, jehož verzi firmwaru nedokáže přečíst.** Senzor, jehož návrat po blesku trvá déle než nastavená čekací doba, nezaznamenal selhání: jakmile odpoví, je znovu připojen s vyplněnými nastaveními nového firmwaru a příkaz upozorní, že byl pomalý.
- **Aktualizace firmwaru vyžaduje příchozí TCP připojení.** Snímač se znovu připojuje k tomuto počítači, aby získal snímek. Chloros naslouchá na adaptéru, který směruje data ke snímači (na všech adaptérech pouze v případě, že nelze určit, který z nich to je), a obsluhuje pouze vlastní adresu snímače; jakékoli jiné připojení je uzavřeno bez obsloužení. Ve Windows jej propouští pravidlo brány firewall `Chloros Backend (TCP-In)` v instalačním programu (viz [Windows (.exe)](#windows-exe)); firewall, který ji blokuje, způsobí selhání aktualizace s chybou `… device never connected back on TCP <port> …`.
- **V prostředí WSL2 použijte zrcadlené síťové připojení** (`networkingMode=mirrored` v `%UserProfile%\.wslconfig`). Ve výchozím režimu NAT je adresa tohoto počítače v síti senzoru adresou samotného WSL, takže každá statická adresa je odmítnuta (s výjimkou `--force`), živý přenos se nedostane do WSL a aktualizace firmwaru nelze dokončit, protože senzor se připojuje zpět k počítači, aby přijal obraz.
- **ID senzoru a název hostitele se nemění.** Ať už je jeho adresa jakákoli, DAQ-E je stále `daq-e-<id>` a `daq-e-<id>.local`, takže `--eth-host daq-e-<id>.local` jej stále dokáže najít.
- **GUI má stejná nastavení**: záložka Světelné senzory, nastavení senzoru (ozubené kolečko), Síť. GUI si udržuje vlastní připojení k senzoru, takže pokud změníte senzor, ke kterému má GUI otevřené připojení, přímo v GUI: restart z CLI ukončí relaci grafického rozhraní s tímto senzorem.
- Ekvivalenty v SDK jsou `DAQSensorSession.device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` a `update_firmware()`.

---

## `chloros-cli project`

Otevření, připojení a ovládání uloženého projektu Chloros (složku obsahující soubory `cameras.json` + `sensors.json` + `project.json`). Vše prochází přes backend, takže grafické rozhraní (GUI) i příkazový řádek (CLI) generují identický stav hardwaru.

### Přehled podpříkazů

| Podpříkaz | Účel |
| --- | --- |
| `project open PATH [--relink FOLDER]` | Vytiskne seznam zařízení projektu (kamery, pole, senzory), načte jej ze souborů a zkontroluje zdrojové soubory (viz [Přesunutý projekt](#a-moved-project---relink)). Projekt se v backendu neotevře: projekt, který je tam otevřený, i jeho zařízení a záznamy, zůstanou beze změny. Ukončí `1`, pokud nelze najít žádný ze zdrojových souborů projektu. |
| `project devices PATH [--reconnect]` | Vypíše seznam nebo znovu spustí vyhledávání. |
| `project connect PATH [--cameras-only] [--sensors-only] [--packed {auto,on,off}]` | Připojí každou uloženou kameru / pole / senzor. Každé pole se připojuje stejně jako v případě `lattice array-connect`: až 600 s, přičemž jednotlivé kroky se vypisují na stderr (`[array <id>]    <s> s: <step>`; stdout je zpráva ve formátu JSON), a pole, které se v této době nepřipojilo, se zruší — jeho kamery se vrátí do stavu, v jakém byly nalezeny — a nahlásí se pod `errors`, nikdy nezůstane připojené. `--packed` nastaví u každého pole komprimovaný 12bitový přenos (výchozí hodnota `auto`; projekt neukládá nastavení pro jednotlivá pole; viz [Zkomprimovaný 12bitový přenos](#packed-12-bit-transport---packed)). Každý světelný senzor je otevřen v DAQ poolu backendu na své uložené adrese s uloženým integračním časem, průměrováním snímků, automatickou expozicí a omezovačem: snímač bez zvoleného omezení dostane `sunshine_cosine`, snímač uložený jako „bare“ (`none`) zůstane v tomto stavu a snímač typu DAQ-E-S nikdy nepřijímá omezení. Panel DAQ v grafickém uživatelském rozhraní není součástí fondu: nejprve tam senzor zavřete. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Jednotlivý snímek z pojmenované kamery nebo pole (`--prefix` výchozí, `capture`). `--orient` (pole): pole ukládá surová data, která zůstávají v nativním formátu — tím se zaznamená, že produkty, které z nich vytváří `process`, mají být orientovány jako `array-capture --orient`. Výchozí: nativní. `project burst` / `stream` a položky receptu `capture` / `burst` / `stream` (`orient: true`) mají stejnou volbu. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Série N snímků z pojmenované kamery nebo pole (`-n/--count` výchozí hodnota 5; `-i/--interval` interval mezi snímky v sekundách, výchozí hodnota 0). Série snímků z pole odstraňují duplicity opakovaných synchronizovaných skupin (kontrolní mechanismus zastaralosti), takže pole s částečným cyklem nemůže vrátit N kopií jednoho snímku; vypíše výsledky za každou iteraci. Selhání snímání v průběhu zastaví sérii a ukončí `1` s uvedením počtu snímků uložených před selháním. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--orient]` | Snímání na disk ve smyčce: `--fps F` zachytí za sekundu (výchozí hodnota 3), jeden snímek na zachycení z kamery, jednu synchronizovanou skupinu snímků z pole (se stejným kontrolním mechanismem zastaralosti jako `project burst`), dokud `-n N` nezachytí (výchozí nastavení `0` = do stisknutí klávesové zkratky Ctrl+C, což vypíše počet uložených snímků a ukončí program `0`, nebo `1`, pokud nebyl uložen žádný snímek nebo došlo k selhání souboru; stisknutí klávesy Ctrl+C před zachycením `-n N` ukončí program `130`). Selhání zachycování v jeho průběhu zastaví proud dat a ukončí `1` s uvedením počtu snímků uložených před selháním. `--poll-interval` je zastaralé a ignorované: stále je akceptován, takže starší skripty fungují, avšak s poznámkou na stderr. |
| `project sensor read PATH NAME [--json]` | Poslední rámec spektra. |
| `project sensor log PATH NAME [--seconds SEC] [-o DIR] [--device-name NAME]` | Zaznamená soubor .daq pro `--seconds` (výchozí hodnota 10). Záznam, který neobsahuje žádná naměřená data, je nahlášen na stderr a program `1` se ukončí. |
| `project run PATH RECIPE.yaml` | Spustit recept na snímání ve formátu YAML/JSON. `--dry-run` provede ověření bez spuštění. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Vypočítá zarovnání pro pole — viz [tabulka příznaků níže](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Vypíše aktuální profil zarovnání. |
| `project align clear PATH NAME` | Vymaže profil z mezipaměti. |
| `project align tweak PATH NAME --serial SN [--dx N] [--dy N] [--rotation-deg N] [--scale N]` | Posune transformaci jednoho podřízeného uzlu (výchozí hodnoty `0`, `0`, `0`, `1.0`: beze změny). |
| `project align export PATH NAME --to FILE` | Uložit profil do formátu JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Načíst uložený profil. |

#### Možnosti `project align calibrate`

| Příznak | Výchozí | Popis |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Metoda zarovnání. **Tyto názvy se liší od `lattice align-calibrate`**, který přijímá zkrácené formy `orb` / `akaze` / `phase`; u tohoto příznaku nejsou tyto dva příkazy zaměnitelné. |
| `--model {translation, rigid, affine, homography}` | `affine` | Transformace modelu tak, aby zapadl. |
| `--frames N` | `1` | Synchronizované snímky snímků do průměru. |
| `--reference SN` | hlavní | Sériové číslo referenční kamery; všechny ostatní členy jsou na ni deformovány. |
| `--max-features N` | `5000` | Maximální počet ORB charakteristik. |
| `--ratio-threshold F` | `0.75` | Test poměru Lowe'test poměru. |
| `--ransac-threshold-px F` | `3.0` | Prahová hodnota pro vnitřní body v RANSACu. |
| `--min-matches N` | `15` | **Kritérium kvality** — odmítnout řešení s méně než tímto počtem shodných bodů. |
| `--max-reproj-err-px F` | `4.0` | **Kritérium kvality** — odmítnout řešení s chybou reprojekce RMS vyšší než tato hodnota. |
| `--checkerboard RxC` | — | Geometrie desky pro `--method checkerboard`, např. `9x6`. |
| `--name PROFILE` | prázdné | Název profilu vložený do uloženého souboru JSON. **Nejedná se o název pole** — tím je poziční `NAME`. |

Tyto dvě kontroly kvality jsou důvodem, proč může kalibrace úspěšně vyřešit problém, ale přesto
odmítnout uložení: profil, který nesplní kteroukoli z nich, by bez varování nesprávně zaregistroval každé
následující snímání, a proto je odmítnut, místo aby byl uložen.

### Přesunutý projekt (`--relink`)

Projekt `project.json` pojmenovává každý zdrojový soubor — každý snímek a záznam ze světelného senzoru
— podle jeho úplné cesty. Přesuňte složku projektu nebo celou složku projektů
na jiný disk a tyto cesty budou stále odkazovat na původní umístění.

Otevření projektu opraví, co může, samo: soubor, který byl uvnitř složky projektu,
se hledá ve složce, ve které se projekt nyní nachází, a každá cesta, která se tam najde,
se přenastaví a uloží (`project.json` je nejprve zálohován jako
`project.json.before-relink-<date>-<time>`). K tomu dochází při každém otevření – aplikace,
`chloros-cli project open`, `chloros-cli process` do existujícího projektu, SDK a `chloros-mcp`. Cesta se přesměruje pouze na soubor, který se tam nachází.

Soubory, které se nacházely mimo složku projektu – například v rámci relace Hubu nebo při importu pomocí funkce „Přidat složku“ –
tímto způsobem nelze najít. Zadejte název složky, ve které se nyní nacházejí:

```bash
chloros-cli project open "N:/Chloros Projects/Field_A" --relink "E:/backup/Hub Projects"
```

`--relink` použije složku, do které byly soubory přesunuty, nebo jakoukoli nadřazenou složku: složka každého chybějícího
souboru se hledá pod ní podle koncových názvů složek
(`Hub Projects/<session>/raw/<serial>`, poté `<session>/raw/<serial>`, ...), a soubory v každé
soubory ze složky, ve které jich bylo nalezeno nejvíce. `project open` vypíše,
co bylo přesměrováno, kolik zdrojových souborů bylo nalezeno a ve kterých složkách se nacházely
zbylé soubory. Některé stále chybí: varování, ukončení `0` – projekt se otevře se zbytkem. Žádné
nenalezeno: chyba s uvedením názvů těchto složek a `--relink`, ukončení `1`; backend rovněž
odmítá otevřít takový projekt pro `process` nebo `project devices --reconnect` (HTTP 409 `PROJECT_SOURCES_MISSING`).

### Příklady

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Its hub captures moved to another drive: point it at them
chloros-cli project open "/home/user/Chloros Projects/Field_A" --relink "/media/backup/Hub Projects"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### DSL pro recepty

`project run RECIPE.yaml` přijímá soubor ve formátu YAML nebo JSON, který popisuje posloupnost akcí:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        auto_exposure: "Off"
        gain_db: 6.0
        auto_gain: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Podporované akce: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. `apply` převezme klíče `cameras.json`, `settings` — `exposure_us`, `auto_exposure`, `gain_db`, `auto_gain`, `target_brightness`, … (starší názvy `exposure_time`, `exposure_auto`, `gain`, `gain_auto` jsou akceptovány jako aliasy). Expozice, zisk nebo cíl nastavené tímto způsobem jsou považovány za vaše: automatické řízení expozice v programu Chloros (včetně expozice měřené pomocí DAQ) je nechává beze změny. Ruční expozice ponechá zisk beze změny: v automatickém režimu fotoaparát neustále upravuje zisk (v celém svém rozsahu) směrem k cílové jasnosti, dokud nenastavíte `gain_db` nebo `auto_gain: "Off"`. Následující příkaz `auto_exposure: "Continuous"` vrátí expozici zpět do automatického režimu a příkaz `target_brightness_auto: true` vrátí zafixovanou cílovou hodnotu zpět do automatického nastavení cíle fotoaparátu Chloros. Akce `burst` vyžaduje parametry `name` (povinný), `count` (výchozí hodnota 5), `interval` (v sekundách, výchozí hodnota 0), `output`, `format`, a `settings` (stejný tvar nastavení pro jednotlivé kamery jako `apply`); sekvence snímků v poli používají stejný hlídací mechanismus pro čerstvě synchronizovanou skupinu jako `project burst`.

V kroku `capture`, `burst` nebo `stream` se výchozí hodnota `output` nastaví na `output`. Relativní `output` — výchozí hodnota je zahrnuta — se vyhodnocuje podle složky, ze které spouštíte `chloros-cli project run`, nikoli podle složky souboru receptu nebo složky projektu (ve verzi 1.2.0 se nacházela ve složce instalace backendu).

Spusťte jej:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

**Každé spuštění je nejprve zkontrolováno vzhledem k projektu.**Každá akce musí mít jako klíč přesně jedno z výše uvedených sloves — `- capture: {name: FrontLeft}`, nikoli `- action: capture` nebo `- type: capture` — v případě `name` má projekt `cameras.json` / `sensors.json` kameru nebo pole pro `apply` / `capture` / `burst` / `stream` a světelný senzor pro `sensor`; `stream` vyžaduje, aby `count` > 0, a `wait` vyžaduje dobu trvání. Recept s jakoukoli takovou chybou**neprovede** žádnou akci (`[recipe] error: …`, `[recipe] not run: …`, ukončení `1`), a to ani s `stop_on_error: false`. Klíče, které krok nepoužívá, jsou `[recipe] warning:`, stejně jako každé pole nebo světelný senzor, který recept řídí a který není právě připojen. Recept nikdy nepřipojuje zařízení: nejprve spusťte `chloros-cli project connect PATH`. `--dry-run` vypíše recept, poté stejné chyby a varování a `[recipe] valid: …` / `NOT valid: …`, nic nespustí a ukončí se `1`, pokud recept obsahuje chybu.

---

## Proměnné prostředí

| Proměnná | Účinek |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Přepíše URL backendu (výchozí `http://127.0.0.1:5000`) — **respektováno pouze rodinami příkazů `lattice`, `project` a `daq pool-*`** (`lattice network-analysis` místo toho přijímá `--backend-url`). Základní příkazy (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) nastavují `http://127.0.0.1:<port>` a ignorují tuto proměnnou (literál IPv4 obchází penalizaci systému Windows `localhost`→`::1` ~2 s na jeden požadavek), takže se vždy zaměřují na místní počítač. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | Nastaveno v prostředí backendu. `1` zmírňuje odmítnutí v důsledku nadměrného předplatného — připojení pole s `pin_resolution` nebo [příkaz pro dočasné vícekamerové relace](#multi-camera-transient-sessions) (`error_code` `ARRAY_OVERSUBSCRIBED`) — na hlasité varování, přičemž akceptuje ztrátu paketů GVSP. Pouze pro testovací účely — viz [Model fps a burst pole](#array-fps--burst-model). |
| `CHLOROS_ARRAY_PACKED12` | Nastavení v prostředí backendu: `on` nebo `off` nastaví výchozí hodnotu [komprimovaný 12bitový přenos](#packed-12-bit-transport---packed) pro každé pole (výchozí hodnota `auto`). Explicitní nastavení `--packed` má stále přednost. |
| `CHLOROS_DISABLE_UPDATE_CHECK` | `1` (nebo `true` / `yes`) vypne [automatickou kontrolu aktualizací](#chloros-cli-update) v Linux CLI při spuštění. |
| `CHLOROS_CLI_MODE` | Nastavuje se přímo v CLI; nařizuje backendu, aby povolil paralelní zpracování. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` přeskočí záložní test GVSP (pouze výsledky ICMP). **Tím se vypne podpora jumbo paketů, nejde pouze o potlačení záznamů v logu** — kamera odpovídá na pingy DF pouze do 1500 na každé trase, takže tento test je jedinou možností, jak detekovat jumbo pakety. Ušetří ~1 s na kameru na jedno připojení; stojí ~1,45× maximální šířku pásma, pokud by síť *mohla* přenášet jumbo pakety. SDK při nastavení vydá varování. |
| `CHLOROS_GVSP_SESSION_REUSE` | Nastavte v prostředí backendu: `0` měří velikost paketu při každém připojení namísto jednou na kameru za relaci backendu. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Fixuje velikost paketu GVSP na N bajtů; zcela vynechává testování. Upřednostňujte nastavení pro jednotlivé příkazy (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) před trvalým nastavením: pevně nastavená velikost se přestane přizpůsobovat síti před ní a nastavení hodnoty 9000 na trase, která nepodporuje jumbo pakety, způsobí, že **každý** záznam vyprší s časovým limitem při použití `SC_ERR_TIMEOUT -1011`. |
| `CHLOROS_TRANSIENT_AE_SETTLE_S` | Nastavte v prostředí backendu: maximální počet sekund, po který [dočasný příkaz pro více kamer](#multi-camera-transient-sessions) nebo `lattice capture` kamery, kterou nic neudržuje, stráví nastavením expozice každé kamery ze scény předtím, než si uchová snímek (výchozí `5`). `0` tuto funkci vypne: automatická expozice každé kamery se pak spustí od prvního snímku, počínaje jejím expozičním limitem. |
| `CHLOROS_LOG_DIR` | Nastavuje se v prostředí backendu: složka, do které se ukládá podrobný protokol. Výchozí hodnota je `%LOCALAPPDATA%\Chloros\logs` ve Windowsu, `~/.cache/chloros/logs` v Linuxu (`$XDG_CACHE_HOME/chloros/logs`, pokud je to nastaveno). Kořenová služba balíčku .deb nastavuje proměnnou `/var/log/chloros`, ke které má přístup skupina `adm` — viz [Spuštění backendu jako služby](#running-the-backend-as-a-service-headless). |
| `TMPDIR` (Linux) | Umístění dočasných souborů. CLI automaticky použije `/mnt/ssd/tmp`, pokud existuje. |

---

## Výstupní kódy

| Kód | Význam |
| --- | --- |
| `0` | Úspěch. |
| `1` | Obecná chyba (většina chyb podpříkazů). Záznamový příkaz také ukončí činnost s kódem `1`, pokud se nepodařilo zapsat jakýkoli soubor, o jehož zápis byl požádán, i když ostatní soubory byly uloženy, a s kódem `array-burst` / `array-record` ukončí program `1`, když záznamník nahlásí chybu — viz [Soubory, které se nepodařilo zapsat](#files-that-fail-to-write). `process` také ukončí činnost `1`, pokud nebylo možné ověřit exporty daného běhu nebo pokud došlo ke ztrátě obrazu, i když ostatní produkty byly zapsány. |
| `2` | Chyba argumentu (neznámý nebo chybějící příznak; také `time-sync set-priority` bez uvedení priority). |
| `130` | Přerušeno klávesovou zkratkou Ctrl+C: jakýkoli příkaz, který tím dojde k předčasnému ukončení — `lattice`, `project`, `daq` i `process` (`process` poté, co backend zastavil běh, viz [Poznámky k `process`](#notes)). Příkaz, jehož normální zastavení se provádí klávesovou zkratkou Ctrl+C — `lattice array-capture --continuous` / `--interval`, `lattice viewer`, `lattice continuous`, `multi-stream`, `align-stream`, `array-record`, `array-burst` a `project stream` s `-n 0` — ukončí se podle toho, co provedl: `0`, když bylo vše zapsáno, `1`, když došlo k jakékoli chybě. |
| `143` / `129` | Ukončeno signálem SIGTERM / SIGHUP (128 + signál) po provedení úklidových operací příkazu: spuštěná úloha byla zastavena a její kamery vráceny do původního stavu, zpracování v backendu bylo zastaveno. `lattice continuous` nebo jiný záznam, jehož normální zastavení se provádí klávesovou zkratkou Ctrl+C, ukončí také `143`, nikoli `0`. |

`lattice network-analysis` ukončí `1` v případě verdiktů `needs_force_slip` a `error`, a program `selftest` ukončí program `1`, pokud neprojdou všechny sedm kontrol.

---

## Tipy pro řešení problémů

- **„Vyžaduje se přihlášení“** → Spusťte na tomto počítači jednou program `chloros-cli login EMAIL PASSWORD`.
- **„Složka projektu je pro tento počítač příliš hluboká“** → Na tomto počítači jsou dlouhé cesty systému Windows vypnuté (výchozí nastavení systému Windows), takže žádná cesta nesmí překročit 260 znaků a produkty se nacházejí až asi 130 znaků pod složkou projektu. Nedošlo ke zpracování. Zpráva uvádí nejdelší složku projektu, která se vejde: použijte kratší (`-o` nebo `chloros-cli set-project-folder`), nebo zapněte dlouhé cesty v systému Windows a restartujte Chloros. Instalační program pro Windows nabízí tuto možnost jako „Povolit dlouhé cesty ve Windows“, která je ve výchozím nastavení zaškrtnuta; jinak použijte zásady skupiny: *Konfigurace počítače > Šablony pro správu > Systém > Souborový systém > Povolit dlouhé cesty Win32*.
- **„backend unreachable“** → Spusťte desktopovou aplikaci Chloros (Windows) nebo spusťte backend ručně: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` ve Windows (složka `backend` ve vaší instalační složce; není v `PATH`), `chloros-backend` v systému Linux. Nebo zkontrolujte `CHLOROS_BACKEND_URL`, pokud se jedná o vzdálený přístup.
- **Příkazy `lattice` selhávají s chybou „LATTICE camera drivers not found“** → Backendu se nepodařilo načíst runtime kamery LATTICE, se kterým je nainstalován. Každý příkaz `lattice` se spouští na backendu; samotné CLI neobsahuje žádné ovladače kamer. Přeinstalujte Chloros: instalační program pro Windows i balíček .deb nainstalují runtime vedle backendu.
- **V okně Array connect / Array Settings se zobrazuje hlášení „FRAMES WILL DROP“ nebo „Reduce ROI to enable“** → Příjemní prstenec síťové karty hostitele je příliš malý (obvykle se po aktualizaci ovladače síťové karty resetuje na 32). Viz [Nastavení a ladění hostitelské síťové karty](#host-nic-setup--tuning-lattice-arrays) — nastavte `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Počítač se při restartu/vypnutí zasekne, poté se zobrazí chyba WMI `Invalid class` / síťová karta se nedá aktivovat / chybí USB disky** → Zastaralý ovladač USB 10GbE adaptéru způsobuje chybu `DRIVER_POWER_STATE_FAILURE` (modrá obrazovka smrti `0x9F`). Aktualizujte ovladač adaptéru — viz [Nastavení a ladění síťové karty hostitele](#host-nic-setup--tuning-lattice-arrays).
- **Systém Linux: záznamy patří uživateli root, takže je nelze smazat, a protokol backendu není ve vašem adresáři `~/.cache`** → Backend běží jako root prostřednictvím `chloros-backend.service`. Přepněte na `chloros-backend@<user>.service`, kde běží pod vaším uživatelským účtem — viz [Spuštění backendu jako služby](#running-the-backend-as-a-service-headless). Do té doby se protokol služby root nachází v `/var/log/chloros` (čitelné pro skupinu `adm`).
- **Linux: backendová jednotka je `failed` s hlášením „Another instance is already running“ (stav ukončení 98)** → Port již obsazuje jiná backendová jednotka: najednou může běžet pouze jedna. Pokud jsou povoleny dvě backendové jednotky, ponechte pouze jednu: `sudo systemctl disable --now <unit>` pro tu, kterou nepoužíváte — viz [Spuštění backendu jako služby](#running-the-backend-as-a-service-headless). Pokud je povolena pouze tato jednotka, port drží backend spuštěný mimo systemd (pomocí `chloros-backend`, `chloros-cli` nebo skriptu SDK): zastavte tento backend a poté spusťte `sudo systemctl start <unit>`.
- **Varování ohledně swapového prostoru na Jetsonu** → Přidejte swapový prostor založený na souborech; CLI vypíše přesné příkazy `fallocate` / `swapon`.
- **„backend není dostupný“ při příkazu `lattice`, `project` nebo `daq`** → žádný backend neběží: tyto příkazy žádný backend nespouštějí. Spusťte příkaz `chloros-cli backend status` a postupujte podle pokynů v části [Spuštění backendu](#start-the-backend).
- **„`daq net ...` není v Chloros k dispozici“** (Linux a Jetson) → Některá vlastní nastavení zařízení DAQ-E jsou vyhrazena pro službu MAPIR. Síťová nastavení, restartování a aktualizace firmwaru se provádějí pomocí `pool-set-network`, `pool-reboot` a `pool-update-firmware`; v ostatních případech kontaktujte podporu MAPIR.
- **„Přímé hardwarové příkazy pro DAQ vyžadují kompletní balíček daq“** (Windows) → Použijte `pool-*`, který ovládá stejný senzor prostřednictvím backendu, nebo `chloros_sdk.connect_daq_sensor()` z Pythonu.

---

## Viz také

- [Referenční příručka k Python SDK](sdk-reference.md) — programový ekvivalent každého příkazu CLI.
- [Světelné senzory DAQ](https://mapir.gitbook.io/chloros/daq-light-sensors) v příručce k Chlorosu — senzory, jejich maximální hodnoty a kalibrovaný rozsah, záznam, síťové propojení DAQ-E.
- [Kamery LATTICE](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) v příručce k Chlorosu — připojení, nastavení kamery, režimy snímání, pole.
- Online dokumentace: `https://mapir.gitbook.io/chloros/cli`
