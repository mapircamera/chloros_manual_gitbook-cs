# Referenční příručka k Chloros Python SDK

**Verze:**

1.3.0**Vytvořeno:**

29. 7. 2026 19:19 ·**Revidováno:**

6. 11. 2026**Balíček:** `chloros-sdk` (PyPI)**Cílová skupina:** Optimalizováno pro použití v LLM; čitelné pro člověka.**Rozsah:** Všechny veřejné třídy, funkce a pomocné funkce poskytované balíčkem `import chloros_sdk`, včetně příkladů, které lze zkopírovat a vložit, pokrývajících zpracování obrazu, ovládání jedné kamery, synchronizovaná pole, senzory DAQ a automatizaci projektů, plus server `chloros-mcp` pro AI asistenty.

Pokud vás zajímají pouze hlavní body, přejděte na:
- [Instalace a rychlý start](#instalace)
- [Smart-Connect pro pole LATTICE](#smart-connect-for-lattice-cameras)
- [Relace senzorů DAQ](#daq-sensor-sessions)
- [Nástroje pro kamery](#camera-utilities)
- [Přechodová pole, profily zarovnání a indexy](#transient-arrays-alignment-profiles--indices)
- [Projektová složka a recepty](#project-folder--recipes)
- [Automatizace projektů](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Živé snímky ve vašem vlastním pracovním postupu](#live-frames-in-your-own-pipeline)
- [Server MCP (AI asistenti)](#mcp-server-ai-assistants)

> **Změny ve verzi 1.3.0** (co zaznamená stávající skript verze 1.2.0).
>
> **Změny, které může skript verze 1.2.0 potřebovat zpracovat:**
>
> - **Nyní se vygeneruje výjimka, pokud backend nepotvrdí vydání.** `CameraSession.disconnect()`, `ArraySession.disconnect()` / `force_disconnect()` u pole, které relace otevřela, `DAQSensorSession.disconnect()` / `force_disconnect()` na senzoru, který relace otevřela, a chyby `ChlorosProject.disconnect_all()` vyvolají chybu `ChlorosConnectError`, pokud backend uvolnění odmítne nebo jej včas nepotvrdí, protože zařízení může být stále zablokováno. Ve verzi 1.2.0 se vracely bez hlášení. Ukončení bloku `with` stále nikdy nevyvolá výjimku.
> - **Vypůjčené zařízení není nikdy uvolněno.** `with attach_array(...)` a explicitní `disconnect()` na něm ponechají pole v chodu (ve verzi 1.2.0 opuštění bloku zařízení uvolnilo, i když ho vlastnilo grafické rozhraní); zavolejte `force_disconnect()`, pokud jej chcete ukončit. Stejně tak `connect_daq_sensor()`, který zjistil, že senzor již je v poolu (včetně inteligentní detekce s nulovým příznakem), jej ponechá otevřený na `disconnect()` a při opuštění bloku `with`.
> - **Formáty PNG a JPEG přijímají pouze produkty typu `display` a indexové produkty.** `capture()` s `format="png"` / `"jpg"` (`ext=".png"` / `".jpg"`) a na úrovni `radiance` nebo `reflectance`, nebo `processing="all"`, vyvolá před jakýmkoli požadavkem `ValueError`; 12bitový `raw` / `debayered` je backendem odmítnut. Viz [PNG / JPEG](#png--jpeg-take-the-display-and-index-products-only).
> - **`connect_daq_sensor(sensor_id=...)` otevře daný senzor nebo vyvolá** `ChlorosConnectError` s výpisem toho, co našel; nikdy nepřepne na jiný senzor. Viz [Pinned Transport](#pinned-transport).
> - **`frame_stream()` vyvolá výjimku místo toho, aby přestal komunikovat:** `FrameFetchError` v případě odmítnutí by v následujícím taktu opakoval (čtení odrazivosti zcela bez DAQ), po 60 s bez rámce a po 60 s bez nového spouštěče. Viz [Jeden spouštěč na jedno čtení](#one-trigger-per-read--group_id).
> - **Časový limit čtení nebo přerušení připojení vyvolá chybu `ChlorosConnectError`** z každého volání smart-connect a chybu `analyze_array_network()`, kde verze 1.2.0 vyvolala základní časový limit nebo chybu připojení.
> - **`connect_array()` ve výchozím nastavení čeká 600 s (`timeout`, 180 s ve verzi 1.2.0) a připojení, které se mu nepodaří navázat, je zrušeno.** Volání odešle kód chyby `timeout` do backendu, který vrátí všechny kamery do stavu, v jakém byly nalezeny, a nepřipojí nic, pokud se připojení nepodaří včas; volání vyvolá kód chyby `ChlorosConnectError` (HTTP 504 `CLIENT_DEADLINE_PASSED`). Ve verzi 1.2.0 mohlo připojení provedené bezprostředně po zapnutí kamer trvat déle než volání a připojit pole i poté, co již došlo k vyvolání chyby. `connect_all()` čeká na každé uložené pole stejným způsobem (120 s ve verzi 1.2.0). Viz [`connect_array()` Signature](#connect_array-signature).
> - **`connect_array()` zachovává kompletní snímač.** Bez `width` / `height` / `binning` je každá kamera nastavena na plný snímač a snímková frekvence klesne na úroveň, kterou poskytuje hostitel a síť; verze 1.2.0 zmenšila nebo sdružila snímky, aby udržela snímkovou frekvenci. Viz [Synchronizované pole](#synchronized-array--arraysession-smart-prep).
> - **Formát pixelů Bayer u kamery M3M (mono) představuje její hloubku.** Kamera M3M nabízí pouze formáty Mono, přičemž předvolby označují formáty Bayer: ve verzi 1.2.0 kamera tento formát odmítla a zachovala svůj vlastní, zatímco `settings.pixel_format` označoval formát Bayer. `LatticeCamera(preset=...)`, `apply_preset()`, `set_format()`, `CameraSettings(pixel_format=...)`, `connect_camera(preset=...)` a polesdílených formátů nyní nastavují formát Mono stejné hloubky na M3M: `"BayerRG8"` a `high_speed` dávají `Mono8`; `"BayerRG12"`, `high_quality` a `triggered` dávají `Mono12`; `cam.settings.pixel_format` jej pojmenovává. Totéž platí i opačně: M3C se nikdy neuloží ve formátu Mono, takže `"Mono12"` – z kteréhokoli z těchto, `connect_array(pixel_format=...)`, `set_settings(pixel_format=...)` nebo uloženého formátu pole – dává na M3C `BayerRG12`, a to bez ohledu na to, která kamera je hlavním členem pole. Přenesený objekt `CameraSettings` si zachovává svůj název, takže stejný objekt stále žádá každou kameru o danou hloubku v rámci své vlastní rodiny. `connect_camera(preset=...)` zachovává formát předvolby (verze 1.2.0 nastaví kameru na 12 bitů po použití předvolby) a kamera otevřená na předvolbě již nezapisuje to, co pořídila, do `PRESETS`. Jakýkoli jiný formát, který kamera odmítne (který chybí v jejím seznamu `PixelFormat`, jako například `RGB8` na M3M nebo `BayerRG16` na firmwaru, který jej neobsahuje), není pojmenován jako samostatný: po připojení `LatticeCamera` se `apply_settings()` nebo `set_format()`, `cam.settings.pixel_format` a `packed_transport` označují formát, ve kterém kamera vysílá datový proud, a jeden vytištěný řádek uvádí formát, který byl odmítnut.
> - **`analyze_array_network()` udává, které připojení se použije:** `pixel_format=None` (dříve `"BayerRG8"`) označuje formáty nastavené připojením (Mono12 / BayerRG12) a `packed="auto"` určuje volbu komprimovaného přenosu, takže udávaná hodnota fps se může lišit od 1.2.0'. `pixel_format`, který předáte, projektuje M3M v dané hloubce. Viz [Network-Analysis Helper](#network-analysis-helper).
> - **`CameraSession.capture()` vrací `CaptureResult`** (stále seznam slovníků rámců) s hodnotami `.skipped`, `.failed`, `.daq_files` a `.daq_matches`. Výsledek `ArraySession.capture()` obsahuje také `.alignment` (zda bylo použito zarovnání a proč ne) a `.oriented`.
> - **Zachycení polí jsou ve výchozím nastavení zarovnána, stejně jako v aplikaci.** `ArraySession.capture()`, `continuous()`, `capture_repeated()` a `ArrayHandle.capture()` / `capture_smart()` zarovnávají soubory `debayered`, `display`, `radiance`, `reflectance` a indexové soubory do snímku hlavní kamery a ořízne je na překryv členů (menší než snímač o posuny kamer, `XMP-mapir:AlignmentJSON` `"applied": true`), takže se jedná o stejné soubory jako záznam stejné scény v aplikaci Chloros; `raw` zůstává v původním formátu. Dříve se při pořízení bez `aligned=True` ukládal každý soubor v původní velikosti, bez korekce zkreslení. `aligned=False` toto zachovává. `chloros-mcp``capture_array` a `start_capture_run` používají stejné výchozí nastavení. Viz [Metody `ArraySession`](#arraysession-methods).
> - **`connect_all()` označuje backend, který odmítá SDK**. Spuštěný backend, který je odhlášený nebo má tarif nižší než Copper, je hlášen pod `errors` s vlastním důvodem (HTTP 401 / 403), nikoli jako „neběží žádný backend Chloros“, a každé volání `connect_all()` backend znovu zkontroluje, takže jeho volání po přihlášení funguje na stejném projektovém objektu.
> - **Relativní `output_dir` funguje s lokálním backendem.** Relativní cesty se před odesláním převádějí na absolutní cesty vzhledem k pracovnímu adresáři vašeho skriptu. Ve verzi 1.2.0 je backend vyhodnocoval ve své vlastní instalační složce, což vedlo k selhání s hlášením „Přístup byl odepřen“.
> - **`numpy` je nyní požadavkem** SDK (volání live-frame vrací pole numpy).
> - **Název na úrovni exportu, který není „one“, je odmítnut.** `processing=` a `levels=` přijímají `raw`, `debayered`, `radiance`, `reflectance` nebo `display` (nebo alias, jako je `preview`; `processing=` a také `"all"`). Jakékoli jiné jméno je odmítnuto před dotykem kamery (HTTP 400 `UNKNOWN_LEVEL`: `capture()` vyvolá `ChlorosConnectError`, `grab()` / `frame_stream()` `FrameFetchError`); verze 1.2.0 pro něj uložila nebo vrátila `raw`.
> - **Funkce Texture Aware je pro snímky LATTICE odmítnuta.** V projektu obsahujícím snímky LATTICE vyvolá `configure(debayer="Texture Aware (Slow, Highest Quality)")` chybu `ChlorosConfigurationError`, stejně jako `process()` po chybě `configure()`, která o to před importem. Hodnota „Texture Aware“, kterou projekt dříve uložil, je nastavena na „Standard“ s chybou `UserWarning`. Snímky LATTICE vždy používají standardní demosaic, takže verze 1.2.0 toto nastavení přijala a ignorovala; aplikace Chloros ji pro ně rovněž nenabízí. Viz [Podporované hodnoty](#supported-values).
> - **Kamery se vrátí do stavu, v jakém byly nalezeny — pokud není zadán kód `keep_settings=True`.** `ArraySession.disconnect()` / `force_disconnect()`, na konci každé přechodové funkce (`capture_cameras_transient()`, `start_multi_camera_stream()`, `test_array_sync()`, `calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index()`) a zastavení funkce `start_camera_continuous()` vrátí každou kameru do původního stavu: její zapojení spouště a synchronizační linky, její PTP a všechna nastavení kamery, která se během relace změnila (expozice a zesílení, cíl automatického nastavení expozice, limity a tlumení, vyvážení bílé, úroveň černé, formát pixelů, binning a oblast zájmu (ROI), převrácení obrazu, snímková frekvence). `keep_settings=True` naopak uloží nastavení každé kamery do její vlastní paměti, takže se s nimi kamera spustí; `clear_settings=True` tuto paměť resetuje na tovární nastavení. V obou případech jsou kamery ponechány v režimu volného běhu. Projekt nastaví své kamery podle vlastních uložených nastavení, jakmile je připojí. Viz [Paměť kamery](#camera-memory--keep_settings--clear_settings).
> - **Všechny režimy ukládají projekt stejným způsobem.** Jakákoli změna kamery, pole nebo světelného senzoru připojeného v rámci projektu — provedená z SDK, aplikace, CLI nebo serveru MCP — se uloží do souboru `cameras.json` / `sensors.json`, a otevření projektu v jakémkoli režimu nastaví zařízení podle něj; `ChlorosProject` pojmenuje svou vlastní složku v backendu. Dříve ukládala projekt pouze aplikace. Viz [Jeden záznam projektu, každý režim](#one-project-record-every-mode).
> - **Soubory vytvořené z několika kamer uvádějí sériové číslo, model a kalibrační balíček každé z nich.** Vícepásmový stack, živý index vytvořený z pásem několika kamer a snímek z hromadného snímání `composite/` obsahují sériové číslo, model a kalibrační balíček každé zdrojové kamery ve formě seznamů (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), zatímco EXIF `SerialNumber` uvádí sériová čísla oddělená čárkami. Dříve neobsahovaly žádné údaje o kameře nebo pouze sériové číslo první kamery. Viz [Soubory vytvořené z několika kamer](#files-made-from-several-cameras).
> - **Pouze soubor `raw` obsahuje kalibrační značky kamery.** Sada kompatibilní s MicaSense (`BlackLevel`, `RadiometricCalibration`, soubory pro jednotlivékanálový `Band{R,G,B}RadiometricCalibration`, sady `MicaSenseVignetting*`) a užitečné zatížení `CalibrationJSON` popisují surové hodnoty senzoru, takže jsou přeneseny pouze v souboru záznamu `raw`. Všechny ostatní soubory — soubory záznamu`debayered`, `display`, `radiance`, `reflectance` a indexové soubory, snímky z nepřetržitého záznamu a z více kamera každý produkt `process()` — nese identifikaci kamery (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) a při snímání také vlastní záznam snímku (`XMP-mapir:FrameJSON`: expozice, zesílení, teplota, formát pixelů, čas pořízení). Ve verzi 1.2.0 obsahovaly také sadu surových dat, kterou nástroj pro jejich zpracování (Pix4D, Metashape) aplikoval na pixely, které nejsou surovými hodnotami. Program Processing určuje kalibraci takového souboru podle jeho sériového čísla a položky `CalBundleSha256`.
>
> **Další změny:**
>
> - **Automatická snímková frekvence pole se přizpůsobuje tomu, co pole poskytuje.** Frekvence, u které dochází ke ztrátě více než 1 % spouštěcích signálů, se sníží; toto posouzení se provádí na základě několika delších pozorování, nikoli pouze jednoho, a frekvence, ze které byla snížena, se čas od času znovu vyzkouší. Frekvence, kterou nastavíte sami, zůstane beze změny.
> - **Automatická expozice je stabilnější.** Udržuje se při konstantním osvětlení, ustálí se během několika sekund po změně, ignoruje drobné přesvětlené oblasti a obnoví funkci fotoaparátu, který přestane reagovat na přímém slunci. Viz [Jak funguje automatická expozice](#how-auto-exposure-behaves).
> - **`import_images(recursive=True)` počítá pouze vaše snímky a nahrávky.** Ve složce obsahující `camera_cal/` projektu se tyto kalibrační soubory již nepočítají jako snímky a nahrávky a projekt, který je již přijal, je vyřadí.
> - **`configure(radiometric_output=...)` se uplatňuje.** Nyní nastavuje produkty projektu `Export radiance` / `Export reflectance`; dříve měla přednost vlastní nastavení produktů každého projektu, takže `radiance`, `sensor-response` a `all` všechny exportovaly radianci a odrazivost.
> - **`test_array_sync()` přechází ze stavu zapnutí kamer.** Zapne PTP kamer pro daný běh, počká, až se jejich hodiny synchronizují a ustálí před prvním spouštěcím signálem (`ptp_lock`) — každá kamera se synchronizuje po dobu 120 s, což je přibližně 2 minuty od zapnutí (`ptp_settle_s=` jej nastavuje), protože právě synchronizovaný časovač má odchylku až ~200 us po dobu přibližně 2 minut — a vrátí PTP každé kamery do stavu, v jakém jej našla. Na kameru, která je stále synchronizovaná z předchozího testu, se již znovu nečeká. Dříve nechával PTP vypnutý v síti s jedním přepínačem, kde žádný DAQ-E neodpovídal, a četl volně běžící hodiny každé kamery jako hodiny PTP, které se nesynchronizovaly („PTP není synchronizováno“, při každém běhu).
> - **Záznam `.daq` udává, zda byl jeho kryt zvolen nebo předpokládán.** DAQ-U, DAQ-M a DAQ-E se dodávají bez krytky a Chloros nedokáže rozpoznat, zda je krytka nasazena, takže senzor připojený bez pojmenované krytky dostane ve výchozím nastavení označení `sunshine_cosine`, což je *předpoklad*. Každý záznam Chloros (schéma 1.30) nyní ukládá `cap_id_source` vedle `cap_id`: `operator`, pokud jste kryt pojmenovali (včetně `none`), `auto_default`, pokud to byl výchozí parametr. Při zpracování záznamu, u kterého byl kondenzátor předpokládán, a přičemž je kondenzátor DAQ v projektu ponechán na Auto, se vypíše `[daq-dls] WARNING` s názvem souboru: samotný DAQ-U bez korekce slunečního svitu vykazuje asi 30x vyšší hodnoty. **Pojmenujte kondenzátor, který jste nainstalovali** při připojení (`connect_daq_sensor(cap_id=...)` nebo `DAQSensorSession.set_cap()`, `chloros-cli daq pool-connect --cap-id` / `daq pool-set-cap`, nebo z rozevíracího seznamu kondenzátorů v grafickém rozhraní), nebo nastavte kondenzátor DAQ v projektu a varování zmizí.
> - **V systému Windows se již nezobrazují okna konzole pip.** Na počítači s nainstalovaným Pythonem spouštěly backend a každý z jeho obrazových pracovníků při spuštění jednou program `pip` a každý z nich otevřel okno konzole. Nyní se to již neděje.
> - **Živé záznamy `.daq` přežijí výpadek napájení.**Záznam se zapisuje s deníkem a při každém potvrzení dochází k úplné synchronizaci disku, a to za senzorem, takže pomalý disk zpomaluje pouze soubor, nikdy však naměřené hodnoty. Záznam zastavený uprostřed potvrzení (výpadek proudu, ukončení procesu) může vedle sebe zanechat soubor `<name>.daq-journal`:**zkopírujte oba soubory dohromady.**Chloros při prvním načtení souboru vrátí deník zpět, takže se záznam otevře se všemi daty zapsanými před zastavením (ztratí se přibližně poslední 3 s); ve verzi 1.2.0 mohl výpadek napájení ve špatný okamžik způsobit, že celý soubor byl nečitelný („database disk image is malformed“).
> - **`DAQSensorSession.record_start()` na vytíženém disku již nezpůsobuje zablokování senzoru**: hodnoty se zařazují do fronty a zapisují se ve vlastním vlákně, a hodnota, kterou nebylo možné zapsat, se započítá do `status()` v rámci `record_errors` (společně s `record_last_error`).
> - **Přímý `LatticeCamera`:**snímek bez expozice/zisku bloku nyní hlásí**skutečnou** expozici a zisk kamery (záložní řešení četlo hodnotu z mezipaměti, která mohla zůstat nezměněná po celou dobu běhu automatické expozice), a `apply_settings(trigger_source='LineN')` / `set_trigger_source('LineN')` aplikují stejné 5 µs odrušování vstupu, jaké dostávají kabelové slave kamery Chloros (nefiltrované, slave spustila snímky bez odeslaného spouštěcího signálu).
> - **Kamery spouštěné softwarově již nevynechávají každý druhý spouštěcí signál.** Kamera v režimu softwarového spouštění nyní zařadí do fronty spouštěcí signál, který dorazí během jejího čtení (nebo Chloros počká, až bude připravena), stejně jako to již dělaly kamery synchronizované kabelem.
> - **`discover_daq_sensors()` pojmenovává DAQ-E-S jako `DAQ-E-S`.** Jeho `model` byl `DAQ-E` pro každý ethernetový senzor; ten, který neodpoví na kontrolu identity do cca 2 s, je stále uveden v seznamu jako `DAQ-E`.
> - **Novinka: `chloros-mcp`, server MCP pro AI asistenty.** Claude Desktop, Claude Code, Cursor a další klienti MCP mohou zpracovávat snímky (vytvořit projekt, importovat snímky, zvolit nastavení, zpracovat, vygenerovat zprávu o výsledcích) a ovládat kamery LATTICE, pole a světelné senzory DAQ: připojit, zobrazit náhled, nastavit expozici a barvy, pořídit snímek, zaznamenat. Jeho nástroje pokrývají funkce tohoto SDK. Instaluje se společně s SDK a balíček .deb pro Linux jej umístí do adresáře `/usr/bin`. Viz [MCP Server (AI asistenti)](#mcp-server-ai-assistants).
> - **Novinka: nástroje pro kamery**, příkazy `chloros-cli lattice` jako volání SDK: barevný profil, sytost / kontrast, světlé oblasti, vyvážení bílé (jednorázové, automatické, pevné poměry), předvolby, modely filtrů, stav a aktualizace firmwaru, napájení (test, spánek, probuzení), kalibrace cíle, síťové adaptéry (kontrola, odhad, oprava), korekce vinětace u souborů, čtení DLS a test hostitele. Viz [Nástroje pro kameru](#camera-utilities).
> - **Novinka: zbytek CLI jako volání SDK**, takže CLI, SDK a `chloros-mcp` nabízejí stejné funkce (CLI si z bezpečnostních důvodů ponechává pouze přihlášení, provisioning zařízení DAQ-E a přímé servery DAQ): kamery otevřené jedním příkazem (`start_camera_continuous()`, `start_multi_camera_stream()`, `capture_cameras_transient()`, `test_array_sync()`, `detect_array_master()`, `analyze_array_models()`), soubory profilů zarovnání (`calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `alignment_profile_info()`, `reorder_alignment_profile()`), indexy (`index_catalog()`, `compute_index()`, `capture_live_index()`), hlavní soubor PTP (`time_sync_status()` a související soubory), projektovou složku (`project_folder()`, `set_project_folder()`, `reset_project_folder()`) a recepty (`run_project_recipe()`, vlastní spouštěč CLI). Pro světelné senzory: `connect_daq_sensor(cap_id=...)`, `DAQSensorSession.set_cap()`, `daq_bindings()` / `set_daq_bindings()`, `daq_band_presets()` a `light_sensor_reflectance()` (odrazivost a indexy okolního prostředí a objektů u dvou senzorů); pro kameru: `CameraSession.status()` a `capture_repeated()`; pro pole: `ArraySession.capability()` a `set_member_settings()` (expozice, zisk člena pole, ...). Viz [Přechodová pole, profily zarovnání a indexy](#transient-arrays-alignment-profiles--indices), [Složka projektu a recepty](#project-folder--recipes) a [Relace senzorů DAQ](#daq-sensor-sessions).
> - **Vylepšené indexové produkty LATTICE** (NDVI a další indexy) z `process()`, ve všech režimech zpracování.
> - **Kamery LATTICE OCN: vylepšené produkty radiance, odrazivosti a indexů při slunečním světle.** Po rozložení pásem se aplikuje korekce flotily — v reálném čase, při snímání i v `process()`; kamery RGN zůstávají beze změny. Viz [Radiometrický výstup](#radiometric-output-lattice-multispectral-pipeline).
> - **Každý snímek ukládá hodnotu DAQ, kterou použil, stejně jako grafické uživatelské rozhraní.** `CameraSession.capture()` a `ArraySession.capture()` / `continuous()` zapisují do souboru `.daq` a jeho `.match.json`, kdykoli je DAQ navázán nebo zařazen do skupiny, včetně záznamu pouze v surovém formátu; ve verzi 1.2.0 se zachycení v surovém formátu nebo po odstranění Bayerova filtru ukládalo pouze s `force_daq=True`. `daq=False` tuto funkci vypíná a `daq_tolerance_ms=` nastavuje toleranci shody; oba výsledky udávají, co bylo uloženo v `.daq_files` a `.daq_matches`. Snímání nyní čeká na další odečet z DAQ, aby mohlo sloučit odečty z obou stran expozice. Viz [Odečet z DAQ, který snímání ukládá](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
> - **`ArraySession` obsahuje živé snímky a nepřetržitý záznam na straně serveru.** `grab()` a `frame_stream()` vrací snímky v numpy z jednoho spouštěcího signálu (stejné čtení jako `ArrayHandle.grab()` / `frame_stream()` v projektovém API), a `continuous()` ukládá každou synchronizovanou skupinu do backendu a vrací [`ContinuousHandle`](#continuoushandle). Viz [Živé rámce ve vašem vlastním potrubí](#live-frames-in-your-own-pipeline).
> - **Pole `grab()` je přiřazeno k jednomu spouštěči** (`ArrayHandle.grab()` a nový `ArraySession.grab()`). Ve verzi 1.2.0 čte 12 z 20 čtyřkamerových `ArrayHandle.grab()` se rozkládalo na dva spouštěče. Každý snímek nyní nese `group_id` a pole `frame_stream()` poskytuje každý spouštěč jednou. Viz [Jeden spouštěč na čtení](#one-trigger-per-read--group_id).
> - **Snímky podle pásma a zarovnané snímky:** `grab()` / `frame_stream()` přijímají `band=`, `align=`, `bit_depth=` a `refl_max=` na identifikátorech kamery a pole. Viz [Export jednoho pásma](#exporting-a-single-band--band).
> - **Nové metody a argumenty `ArraySession`:** `set_target_fps()`, `force_disconnect()`, `capture(after_seq=..., orient=...)` (společně s `CaptureResult.group_seq` a `.failed`), `continuous()`, `connect_array(packed=...)`; `capture_repeated()` řetězí po sobě jdoucí skupiny a zastaví se při zaplnění disku.
> - **`ArraySession.record()` zaznamenává data bez otevřeného grafického rozhraní.** Je-li kombinovaný pohled grafického rozhraní zavřený, backend vykreslí pohled sám: poleNDVI pole nebo index, který předáte jako `expr=`. `RecorderHandle.stop()` vyvolá výjimku `ChlorosConnectError` v případě selhání záznamu, při kterém nebyl zapsán žádný snímek, a zachová výsledek zastavení v `.result` (při ponechání bloku `with` nedojde k vyvolání výjimky: zkontrolujte `rec.result`). Viz [`RecorderHandle`](#recorderhandle).
> - **Nové metody `DAQSensorSession`:** `enable_ptp()`, `force_disconnect()` a pro vlastní nastavení zařízení DAQ-E `device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` a `update_firmware()`. `record_stop()` vrací `warning` a `gaps`. Restart, který uzavřel záznam, jej pojmenuje ve své odpovědi nebo, pokud restart selže, v položce `ChlorosConnectError` (`recording_closed`; viz [Nastavení zařízení DAQ-E](#daq-e-device-settings)). Viz [Metody `DAQSensorSession`](#daqsensorsession-methods).
> - **Sdružený DAQ-E, který přestane odesílat data, je znovu připojen backendem** s původním nastavením a záznam na něm pokračuje ve stejném `.daq` s mezerou označenou v souboru; `record_stop()` vrací mezery jako `gaps`. Ve verzi 1.2.0 senzor zůstával neaktivní, dokud nebyl odpojen a znovu připojen, což ukončilo záznam. Viz část [DAQ-E, který přestane odesílat data](cli-reference.md#a-daq-e-that-stops-delivering).
> - **`open_project()` funguje na základě nainstalovaného SDK.** Wheel (instalační balíčeka PyPI) nyní obsahuje potřebné zavaděče projektu; dříve `open_project()` vyvolával chybu `ImportError` s názvem `project_cameras` / `project_sensors` v jakémkoli SDK nainstalovaném přes pip. `connect_all()` připojuje uložené kamery, pole a světelné senzory prostřednictvím spuštěného backendu (světelné senzory prostřednictvím svého DAQ fondu) a uložené zařízení, které nemůže vyhledat, je uvedeno pod položkou `errors` s uvedením důvodu, místo aby bylo bez vysvětlení přeskočeno. Uložené pole, které již v backendu běží (v grafickém rozhraní, nebo v `chloros-cli project connect`) je připojeno a ponecháno v provozu, zatímco dříve to bylo hlášeno jako chyba („Kamera  je<sn> již v poli<id>“). Viz [Metody `ChlorosProject`](#chlorosproject-methods).
> - **`ChlorosLocal()`, `process_folder()` a `process_lattice_capture()` vyhledávají Chloros ve Windowsu.** Hledají v umístění, kam instalační program umístí backend (ve výchozím nastavení `<install folder>\backend\`, `C:\Program Files\Chloros\backend\`, vlastní složku z vlastního záznamu instalačního programu) a připojí se k již spuštěnému backendu, aniž by ho vůbec hledaly. Ve verzi 1.2.0 vyvolávaly na nainstalovaném počítači hlášku „Chloros backend not found“, pokud jste nepředali `backend_exe=`.
> - **Backend, který selže při spuštění, je nahlášen okamžitě.** Pokud `ChlorosLocal()` nebo volání smart-connect spustí backend, který se ukončí dříve, než odpoví, volání během několika sekund vygeneruje chybu `ChlorosBackendError` z `ChlorosLocal` a `ChlorosConnectError` z volání smart--connect, spolu s kódem ukončení, příkazem, který spustil, a posledními řádky, které backend vypsal. Verze 1.2.0 vyčkala 60sekundový časový limit spuštění a neoznámila důvod. Viz [Automatické spuštění backendu](#backend-auto-start).
> - **Backend pokračuje v běhu mezi skripty.** Opuštění bloku `with ChlorosLocal(...)`, `process_folder()` / `process_lattice_capture()` a návratu `chloros-mcp` již nezastaví backend spuštěný SDK; další skript jej znovu využije. Zastaví se na `logout()` (pokud není otevřena aplikace Chloros nebo neprobíhá spuštění), na `shutdown_backend()`, nebo sám po 30 minutách, pokud jej nic nevyužívá (`CHLOROS_BACKEND_IDLE_MIN` v prostředí backendu to mění; `0`: nikdy).
> - **Backend spuštěný SDKm je vyzván k ukončení, není násilně ukončen.** `shutdown_backend()` umožní backendu uvolnit jeho kamery a světelné senzory a dokončit veškeré nahrávání; je ukončen pouze v případě, že stále běží i po 10 sekundách. Verze 1.2.0 jej ukončila okamžitě (ve Windows došlo k násilnému ukončení), což mohlo způsobit, že světelný senzor DAQ připojený k tomuto backendu přestal reagovat. Backend, který SDK zjistilo jako spuštěný, zůstává nadále nedotčen.
> - **V systému Linux automatické spuštění nikdy nespustí `backend_server.py` z aktuální složky.** Pokud nebyl nainstalován žádný backend, spustil jej skript spuštěný ve složce, která jej obsahovala; nyní se spouští pouze nainstalovaný backend (nebo, v případě `ChlorosLocal`, ten, který `backend_exe=` určí).
> - **`configure(indices=...)`**přidá do projektu každou předvolbu pro pásma dané kamery, včetně nového projektu, a upozorní (`UserWarning`) na název, který nelze přidat.**`import_images()`** hlásí záznamy `.daq` jako `scan_count` a již nevydává varování ohledně složky pro záznamy LATTICE.
> - **`read_image_audit_tags()` funguje po instalaci pomocí pip.** Sám načítá metadata snímku; ve verzi 1.2.0 vracel `{}` bez interních souborů desktopové instalace v cestě k Pythonu.
> - **`capture()` hned po `connect_array()`** čeká až 20 s na první skupinu snímků pole namísto toho, aby vyhodil chybu HTTP 503 „Pole se stále zahřívá“.
> - **`status()` hlásí zablokování.** Nové pole `stalled` má hodnotu True, zatímco se spouští smyčka snímání, ale nic nezveřejňuje, a `actual_fps` je pak 0. Ve verzi 1.2.0 si `actual_fps` zachovávalo svou poslední hodnotu během zablokování, takže zablokované pole zobrazovalo hodnotu `actual_fps > 0`, zatímco snímání selhalo s chybou 503 „No fresh synced frame group“.
> - **Méně neúplných sad snímků a chyb 503 při snímání:** spouštěcí vstup každé podřízené kamery je nyní zbaven rušivých signálů.
> - **Komprimovaný 12bitový přenos** (`connect_array(packed="auto")`) zvyšuje snímkovou frekvenci pole s plným snímačem přibližně o třetinu. Viz [Komprimovaný 12bitový přenos](#packed-12-bit-transport-packed).
> - **Expozice pole v oblasti zájmu (ROI):** malá oblast zájmu již neomezuje expozici člena na dobu volného běhu snímače (4,9 ms při rozlišení 768 × 304). Viz [Expozice na členech pole](#exposure-on-array-members).
> - **Automatická expozice již nesnižuje automatickou snímkovou frekvenci pole.** Pole synchronizované kabelem běží rychlostí, kterou umožňuje síť a čtení kamer; automatická expozice vyplní dobu, kterou tato rychlost poskytuje, a poté zvýší zisk. Ruční expozice delší než tato doba zpomalí spouštění tak, aby se jí přizpůsobilo, místo aby byla odmítnuta. Viz [Expozice na prvcích pole](#exposure-on-array-members).
> - **`read_nodes()`**přidává `frame_values` a `notes` během běhu automatické expozice (viz [`read_nodes()` při automatické expozici](#read_nodes-under-auto-exposure--frame_values-and-notes)). Pokud kamera odmítne expozici nebo zisk,**`set_settings()`** uvede rozsah, který akceptuje, a obnoví automatický režim, který pro danou hodnotu vypnula; `set_settings(target_brightness_auto=True)` je nový.
> - **`orient=`** u `ArraySession.capture()` / `capture_fastest()` / `continuous()` / `burst()` a `ArrayHandle.capture()` / `capture_smart()` otáčí produkty podle orientace kamery. Viz [Metody `ArraySession`](#arraysession-methods).
> - **`DLS()` vyžaduje vlastní `mac_address` daného snímače.** Výchozí adresa MAC již neexistuje (jednalo se o adresu jedné jednotky); volání `DLS()` bez této adresy vyvolá chybu `DLSError`. Viz [velikosti pásem DAQ-M](#daq-m-band-magnitudes--dlsband_magnitudes).
> - **Instalační program pro Windows** nabízí možnosti „Povolit dlouhé cesty ve Windows“ a „Vyloučit backend Chloros ze skenování Microsoft Defender“, které jsou ve výchozím nastavení zaškrtnuté. Viz [Krok 1](#step-1--install-the-chloros-platform-package).
> - **Linuxový balíček .deb** nainstaluje Pythonové SDK pro systém `python3` (dříve pouze ukládal balíček wheel na disk), zvětší velikost přijímacích bufferů socketů jádra, které potřebují datové toky z kamer, a přidá backendovou službu, která běží pod vaším uživatelským účtem. Balíček pro Jetson je `chloros_1.3.0_arm64_jp6.deb`, určený pro JetPack 6, s CUDA PyTorchem pro JetPack. Viz [Krok 1](#step-1--install-the-chloros-platform-package) a [Krok 2](#linux-deb).

---

## Architektura za 60 sekund

SDK je tenká vrstva v jazyce Python nad backendem Chloros (stejný server Flask, jaký používá desktopové grafické rozhraní i CLI). Pro automatizaci importujete `chloros_sdk` a voláte metody vyšší úrovně; v pozadí se každé volání se promění v HTTP požadavek na lokální backend na portu 5000 — `http://127.0.0.1:5000/api/...` (záměrně ne `localhost`, který se ve Windows nejprve přeloží na `::1` a u backendu podporujícího pouze IPv4 zabere ~2 s na jeden požadavek). Backend spravuje hardwarový fond — kamery, senzory DAQ, profily zarovnání, rámcové vyrovnávací paměti — takže skripty SDK mohou koexistovat s grafickým uživatelským rozhraním (GUI), aniž by se musely přetahovat o sériové porty nebo šířku pásma síťové karty.

K dispozici jsou tři rozhraní, která budete používat:

1. **`ChlorosLocal` + volné funkce** (`process_folder`, `process_lattice_capture`) — Zpracovací pipeline obrazu. Spusťte zpracování celé složky včetně kalibrace, debayeringu a exportu indexu jedním voláním v jazyPythonu.
2. **Ovládací prvky Smart-connect** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Otevření trvalé relace backendu pro živý hardware. Stejný postup „smart-prep“ jako v grafickém uživatelském rozhraní: testování sítě, automatický výběr úrovně, PTP, inicializace AE, konfigurace spouštěče GPIO.
3. **`ChlorosProject` / `open_project`** — Načte uložený projekt (složku obsahující soubory `cameras.json` + `sensors.json` + `project.json`), najednou připojí jeho kamery, pole a světelné senzory a spustí snímání pomocí pojmenovaných identifikátorů.

Povrchy 1 a 2 **automaticky spustí lokální backend**, pokud ještě žádný neposlouchá (stejný balíček binárních souborů, který spouští GUI/CLI) — takže holý skript funguje z nového shellu, aniž byste museli nejprve spouštět backend: `ChlorosLocal` (a `process_folder()` / `process_lattice_capture()`), `connect_camera()`, `connect_array()`, `connect_daq_sensor()`, `discover_lattice_cameras()` a `discover_daq_sensors()`. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` a `analyze_array_network()` se nikdy nespustí: potřebují spuštěný backend. Předáním argumentu `auto_start_backend=False` lze tuto funkci deaktivovat (např. při odkazování na vzdálený backend, který se nikdy nespustí). Viz [Automatické spuštění backendu](#backend-auto-start). Surface 3 se chová odlišně: `open_project()` nepřijímá žádný parametr `auto_start_backend` a `connect_all()` nikdy nespustí backend — jednou zkusí kontaktovat `http://127.0.0.1:5000` a pokud nic neodpoví, přejde na přímé (bez backendu) ovládání zařízení `lattice_sdk`. To vyžaduje runtime SDKu Arena a OpenCV (`cv2`), které lze importovat do vašeho Pythonu, což balíček SDK wheel neinstaluje; bez nich je každá uložená kamera a pole hlášeno jako chyba s uvedením důvodu. Světelné senzory projektu nemají možnost provozu bez backendu: pokud není spuštěn žádný backend, je každý uložený senzor hlášeno jako chyba s uvedením důvodu (viz [`SensorHandle`](#sensorhandle)). Pouze `proj.process()` a `stream(..., overlays=True)` vytvářejí `ChlorosLocal()` (který se spouští automaticky) až při potřebě.

Všechny tři vyžadují autorizaci– vyžadují autorizaci: spusťte `chloros-cli login` jednou na daném počítači nebo se přihlaste přes grafické rozhraní na ploše. Co vyvolá volání bez platné relace, závisí na rozhraní (viz [Výjimky](#exceptions)): `ChlorosAuthenticationError` z `ChlorosLocal` (a `process_folder()` / `process_lattice_capture()`), `ChlorosConnectError` z volání smart-connect a jejich relací, `status: "error"` s `error_code` z `analyze_array_network()` a záznam pod `errors` ve zprávě `connect_all()`.

Požadavky:
- Python 3.7+ (jak uvádí balíček; vyvinuto/testováno na verzi 3.10)
- Lokálně nainstalovaný Chloros Desktop (binární soubor backendu je součástí instalačního balíčku)
- Aktivní přihlášení k Chloros+ v **libovolné placené úrovni Chloros+ (Copper nebo vyšší)**; bezplatná úroveň**Iron**nemá přístup k SDK ani CLI. Toto omezení je vynucováno**na straně serveru**: každý požadavek označený SDK/CLI musí obsahovat jak aktivní relaci, tak placený tarif. Tarif nižší než Copper dostane `403` s `error_code: PLAN_UPGRADE_REQUIRED` (`ChlorosLicenseError` z `ChlorosLocal`, `ChlorosConnectError` u volání smart-connect). Odhlášený volající obdrží místo toho kódy `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError` z `ChlorosLocal`, `ChlorosConnectError` z hovorů Smart Connect) — tyto dva kódy se liší, protože opětovné spuštění `chloros-cli login` vyřeší chybu `401`, ale nedokáže vyřešit chybu `403`, která vyžaduje upgrade plánu.
- Offline použití je podporováno v rámci ochranné lhůty tarifu: úroveň přístupu se načítá z mezipaměti pro ověření serverem (5 minut) nebo z mezipaměti podepsané licence vázané na konkrétní zařízení (30 dní u měsíčních tarifů, do vypršení předplatného u ročních tarifů). Po uplynutí této lhůty se tarif převede na bezplatnou verzi a přístup k SDK a CLI se zastaví, dokud se zařízení alespoň jednou nepřipojí k serveru. Chyba `chloros-cli status` (`GET /api/license-status`) zůstává dostupná i v bezplatné verzi, takže je důvod chyby viditelný.
- Windows 10/11 64bit, **Ubuntu 22.04 LTS nebo novější**, nebo Jetson (JetPack 6). Ubuntu 20.04**není** podporováno: závislosti `.deb` jsou odvozeny od toho, na co se backend odkazuje, včetně `libc6 (>= 2.35)`, a Focal dodává glibc 2.31.

---

## Instalace

Pythonové SDK je tenká Pythonová vrstva nad backendem Chloros. Pokud nepoužíváte backend na jiném počítači (viz [Režim vzdáleného backendu](#remote-backend-mode-pip-only-host-via-tunnel)), potřebujete **lokálně nainstalovaný balíček Chloros pro desktop** (instalační program pro Windows nebo Linux `.deb`), který obsahuje světelné senzory DAQ. Balíček obsahuje binární soubor backendu, runtime SDKu Arena pro kamery LATTICE a kalibrační balíčky.

Nejnovější soubory ke stažení: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Krok 1 — Nainstalujte balíček platformy Chloros

#### Windows (.exe)

1. Stáhněte si soubor `Chloros Installer 1.3.0.exe` ze stránky pro stahování.
2. Spusťte instalační program a postupujte podle pokynů průvodce. Výchozí instalační cesta je `C:\Program Files\Chloros\`.
3. Spusťte Chloros alespoň jednou a přihlaste se pomocí svého účtu Chloros+.

Instalační program povolí přes bránu Windows Firewall v každém síťovém profilu přístup pouze pro backend (`chloros-backend.exe`, nic jiného) projít bránou Windows: příchozí UDP pro streamy z kamer, PTP, mDNS a stream z DAQ-E, a příchozí TCP pouze z místní podsítě pro připojení, které DAQ-E navazuje zpět k počítači během aktualizace firmwaru ([`update_firmware()`](#daq-e-device-settings)). Odinstalační program obě pravidla odstraní. HTTP API backendu zůstává v každém případě na adrese `127.0.0.1`.

Skupina **Nastavení systému** v průvodci obsahuje dvě možnosti, které jsou ve výchozím nastavení zaškrtnuté:

| Možnost | Co dělá | Odhlášení z tiché instalace |
| --- | --- | --- |
| **Povolit dlouhé cesty v systému Windows** | Umožňuje, aby složky projektů byly umístěny hlouběji, než je limit 260 znaků pro cesty v systému Windows. Pokud jsou dlouhé cesty vypnuté (výchozí nastavení systému Windows), bude složka projektu, která je pro dané produkty příliš hluboko, odmítnuta ještě před zpracováním čehokoli — viz [Řešení problémů](#troubleshooting-pointers). | `/MERGETASKS="!longpaths"` |
| **Vyloučit backend Chloros z kontroly programem Microsoft Defender** | Zastaví kontrolu v reálnémskenování každého souboru, který zapíše snímání z kamery. Při vysokých rychlostech snímání zabírá skenování asi 30 % rychlosti: při ROI 512 na sestavě 9 kamer dosahovalo nepřetržité snímání ve formátu RAW rychlosti 52,7 snímků/s se skenováním a 77,7 snímků/s s vyloučeným backendem. | `/MERGETASKS="!defenderexclusion"` |

Kontinuální snímání hlásí zatížení programu Defender jako `defender_cores` (viz [`ContinuousHandle`](#continuoushandle)).

#### Linux, amd64 (.deb)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Nainstalujte pomocí `apt install ./…`, nikoli `dpkg -i`: apt nainstaluje také požadavky Python SDK, které balíček doporučuje, nikoli vyžaduje (viz [Krok 2](#linux-deb)).

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Pro Jetson Orin / Orin NX / Orin Nano na JetPack 6 (L4T R36). PyTorch je zde sestaven pro CUDA na JetPack 6, takže zpracování na GPU probíhá na GPU zařízení Orin; využívá knihovny CUDA a cuDNN, které nainstaluje JetPack 6 (`sudo apt install nvidia-jetpack` na obrazu, kde tyto knihovny chybí).

#### Linux: přijímací vyrovnávací paměti a backendová služba

Oba balíčky .deb zvětšují velikost přijímacích vyrovnávacích pamětí soketů jádra, které potřebují datové toky kamer LATTICE, na 32 MB (`/etc/sysctl.d/60-chloros-gige.conf`). Při výchozím nastavení jádra jsou pakety z kamery v jádře zahozeny a pole ztrácí snímky i snímkovou frekvenci; `chloros-cli lattice network` hlásí stav vyrovnávacích pamětí, když jsou téměř plné. Viz část [Linuxové hostitelské systémy — vyrovnávací paměti pro příjem soketů](cli-reference.md#linux-hosts--socket-receive-buffers) v referenční příručce k CLI.

Na stroji bez grafického rozhraní spusťte backend jako službu **pod vaším uživatelským účtem**, aby skript našel spuštěný backend (`connect_all()` nikdy backend nespouští):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
```

Backend pak běží jako vy: záznamy, které zapisuje, jsou vaše a používá váš `chloros-cli login`. Rootový `chloros-backend.service` je stále nainstalován pro stroje, které jej mají povolený, ale každý soubor, který rootový backend zapíše, patří uživateli root. Viz část [Spuštění backendu jako služby](cli-reference.md#running-the-backend-as-a-service-headless) v referenční příručce k CLI.

Backend spouštěný službou je sdílený a klient jej nikdy nezastaví: skripty SDK a `chloros-cli` jej používají tak, jak jej najdou (`ChlorosLocal.shutdown_backend()` zastaví pouze backend, který sám spustil `ChlorosLocal`), a `GET /api/status` pojmenovává jednotku jako `service`. Restartujte jej pomocí `sudo systemctl restart chloros-backend@$USER.service`. Aktualizace balíčku zastaví backendové služby během instalace, takže spuštěný skript ztratí přístup ke kamerám a senzorům a poté znovu spustí všechny povolené; odstranění balíčku je zastaví a zakáže ty povolené. Současně běží pouze jeden backend, proto povolte pouze jednu jednotku: druhá se odmítne spustit a zůstane v stavu `failed` (stav ukončení 98, není restartována) a její `systemctl status` uvádí důvod. Chcete-li přepnout jednotky, deaktivujte tu, kterou nepoužíváte.</id></sn>

### Krok 2 — Instalace Pythonového SDK

**Instalační program Chloros obsahuje odpovídající balíček SDK ve formátu wheel.** Každý instalační program pro Windows a balíček .deb pro Linux umístí na disk soubor `chloros_sdk-X.Y.Z-py3-none-any.whl`, který přesně odpovídá verzi grafického rozhraní, příkazového řádku (CLI) a backendu. Nemusíte sledovat PyPI, abyste zůstali v synchronizaci.

#### Windows

Instalační program automaticky spustí soubor `pip install` s přiloženým balíčkem wheel pomocí vašeho systémového Pythonu (upřednostňuje se spouštěč `py.exe`, v případě potíží se použije `python -m pip`). Není třeba nic dělat — po úspěšné instalaci bude `import chloros_sdk` fungovat ve vašem Pythonovém prostředí. Pokud na počítači není nainstalován Python, instalační program tento krok tiše přeskočí a GUI i CLI budou i nadále fungovat.

#### Linux (.deb)

Oba balíčky .deb (x86_64 a Jetson) nainstalují SDK pro systém `python3`, takže `import chloros_sdk` po instalaci funguje bez pip a bez připojení k síti. Balíčky (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` a načítací programy projektu) se umístí do `/usr/lib/python3/dist-packages`, a příkaz SDK `chloros-mcp` ([MCP server](#mcp-server-ai-assistants)) do adresáře `/usr/bin`; všechny tyto soubory patří k balíčku: při aktualizaci jsou nahrazeny a při odstranění balíčku jsou odstraněny.

Vlastní požadavky SDK, `python3-numpy` a `python3-requests`, jsou **doporučené, nikoli povinné** (GUI, CLI a backend jsou samostatné): `sudo apt install ./chloros_<version>_amd64.deb` (`…_arm64_jp6.deb` na Jetsonu) je nainstaluje, `dpkg -i` nikoli. Výstup instalace uvádí verzi SDK, kterou `python3` importuje, případné chybějící požadavky a o existenci souboru `chloros_sdk` jinde v cestě k `python3`, který zastíní kopii balíčku (například ten, který byl nainstalován dříve pomocí `pip install --user`).

Ve virtuálním prostředí (virtualenv), které nevidí systémové balíčky, nainstalujte do něj přiložený soubor wheel:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

U nasazení na zařízeních Jetson v izolovaném prostředí (air-gapped) se vše odehrává zcela offline — SDK i soubor wheel jsou již na disku.

#### Veřejný PyPI

Pro hostitele používající pouze pip (bez nainstalovaného balíčku Chloros pro desktop; SDK pak řídí backend na jiném stroji, viz [Režim vzdáleného backendu](#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

PyPI obsahuje nejnovější stabilní verzi. Udržujte SDK ve stejné verzi jako backend Chloros, který řídí (viz [Verze a kompatibilita](#versioning--compatibility)). Pomocný program DAQ-M `DLS` také vyžaduje `bleak`: `pip install "chloros-sdk[camera]"` (nebo `pip install bleak`).

#### Ověření

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Vyžaduje se předplatné Chloros+.** Všechna volání SDK vyžadují aktivní přihlášení do Chloros+. Spusťte `chloros-cli login user@example.com 'YourPassword'` jednou na každém počítači; přihlašovací údaje se ukládají do mezipaměti v `~/.chloros/`.

### Potřebuji balíček Desktop Package?

Samotný balíček pip pro většinu pracovních postupů **nestačí**. Zde je přehled toho, co jednotlivé rozhraní SDK vyžadují:

| Rozhraní SDK | Vyžaduje balíček Desktop Package? | Proč |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Ano**(k jeho spuštění)**/ Ne** (backend je již dostupný) | Použít spuštěný backend (z desktopové aplikace nebo z jiného spuštěného skriptu); jinak automaticky spustit nainstalovaný: `/usr/lib/chloros/chloros-backend` (Linux) nebo `<install folder>\backend\chloros-backend.exe` (systém Windows; ve výchozím nastavení `C:\Program Files\Chloros\backend\`, vlastní instalační složka se načte ze záznamu instalačního programu). Předáním `backend_exe=` jej můžete pojmenovat sami. |
| `chloros-mcp` ([MCP server](#mcp-server-ai-assistants)) | **Ano**(pro spuštění)**/ Ne** (backend je již dostupný) | Vychází z `ChlorosLocal`, tedy stejné jako v řádku výše. |
| `read_image_audit_tags` | **Ne** | Čte vlastní metadata zpracovaného obrázku (pouze standardní knihovna Pythonu). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Ano**(lokální)**/ Ne**(vzdálený) | Čistě HTTP klienti přes backend. Lokální backend → vyžaduje se balíček pro desktop. Vzdálený backend → `backend_url=`**přes tunel** (viz Režim vzdáleného backendu — dodávané backendy se vážou pouze na loopback). |
| `ChlorosProject` / `open_project` | **Ano** | Spouští uložené projekty přes backend. |
| Přímé třídy LATTICE (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Ano** | Vyžaduje runtime Arena SDK, který je součástí balíčku pro stolní počítače, a OpenCV (`cv2`) ve vašem Pythonu, což ani balíček pro stolní počítače, ani SDK neinstalují (`pip install opencv-python`). `CAMERA_AVAILABLE` se jinak při importu chová jako `False`. `DLS` také vyžaduje `bleak`. |

### Režim vzdáleného backendu (host pouze s pip, přes tunel)

> **Nainstalovaný backend není dostupný přes LAN.** Naslouchá pouze na
> smyčkovém portu (`127.0.0.1` / `::1`), takže `backend_url="http://<lan-ip>:5000"`
> **nemůže dosáhnout nainstalovaného Chlorosu**. Chcete-li ovládat backend na jiném
> počítači, přesměrujte jeho smyčkový port sami a nasměrujte SDK na
> tunel:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"     # the tunnel endpoint
serials = ["213800234", "214000533"]   # master first

# auto_start_backend=False: the tunnel URL looks local, so without it a tunnel
# that is down would start this machine's own backend, if one is installed.
arr = chloros_sdk.connect_array(serials, backend_url=BACKEND,
                                auto_start_backend=False)
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                     backend_url=BACKEND,
                                     auto_start_backend=False)

# The backend opens every path on ITS machine: pass absolute paths valid there.
# (A relative path would be made absolute against this machine's working folder.)
arr.capture("/home/chloros/captures/run1", processing="raw")
```

Hostitelé bez grafického rozhraní / CI / robotika mohou mít jeden počítač s plnou instalací desktopu jako „Chloros server“ a `pip install chloros-sdk` všude jinde — ale přenos mezi nimi probíhá přes výše uvedený tunel zřízený uživatelem, nikoli přes přímou LAN URL.

> **`ChlorosLocal` na hostiteli pouze s pip.** `ChlorosLocal(backend_url=BACKEND)` se připojuje k backendu, ke kterému se může dostat bez instalace desktopu: hledá lokální binární soubor backendu pouze tehdy, když jej musí SPUSTIT (nic neodpovídá a `auto_start_backend=True`). Až do verze 1.2.0 nejprve hledal binární soubor a na hostiteli pouze s pip vyvolal chybu `ChlorosBackendError` („Backend Chloros nenalezen…“) i v případě, že byl backend dostupný. Složky, které předáte (`process_folder`, `import_images`) jsou na tomto počítači zkontrolovány a načteny backendem, takže v případě vzdáleného backendu musí mít na obou stranách stejný název, například sdílený disk.

### Pracovní postup pouze s DAQ (hostitel pouze s pip)

Samotný `pip install chloros-sdk` nemůže ovládat senzor DAQ: senzor DAQ je vždy ovládán prostřednictvím backendu Chlorosu pomocí `connect_daq_sensor()`:

- **Senzor na tomto počítači:** nainstalujte zde balíček Chloros pro stolní počítače. `connect_daq_sensor()` spustí svůj backend, pokud žádný neběží.
- **Senzor na jiném počítači:** tento počítač potřebuje balíček pro stolní počítače; připojte se k jeho backendu přes tunel v [režimu vzdáleného backendu](#remote-backend-mode-pip-only-host-via-tunnel). Backend otevře senzor na svém vlastním počítači, takže k tomuto počítači je připojen DAQ-U, DAQ-M se nachází v dosahu jeho Bluetooth a DAQ-E je v síti, ke které má přístup.

```python
import chloros_sdk

# http://127.0.0.1:5000 is the local desktop install's backend, or the tunnel.
with chloros_sdk.connect_daq_sensor() as daq:   # smart-detect USB / BLE / ETH
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

V obou případech backend vynucuje plán „Chloros+“: přihlaste se na počítači, na kterém běží backend (`chloros-cli login` nebo desktopová aplikace), s jakýmkoli placeným tarifem (Copper nebo vyšším). Odhlášený backend i tarif nižší než Copper vyvolají chybu `ChlorosConnectError` obsahující důvod backendu (`AUTH_REQUIRED` / `PLAN_UPGRADE_REQUIRED`).

---

## Rychlý start

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Index API nejvyšší úrovně

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by CameraSession.capture / ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job
chloros_sdk.ContinuousHandle                      # handle for an array continuous() capture
chloros_sdk.GrabResult                            # dict returned by grab() / yielded by frame_stream()
chloros_sdk.FrameFetchError                       # raised by grab()/frame_stream() on a refusal

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()
chloros_sdk.daq_bindings() / set_daq_bindings(cameras=, arrays=)   # which DAQ feeds which camera
chloros_sdk.daq_band_presets(filter_code=None)    # band sets for Ambient + Object reflectance
chloros_sdk.light_sensor_reflectance(ambient_id, object_id, ...)   # two DAQs -> reflectance + indices
chloros_sdk.time_sync_status() / time_sync_peers() / time_sync_cameras()   # PTP grandmaster
chloros_sdk.restart_time_sync() / set_time_sync_priority(priority1=, priority2=)

# === Camera utilities (the CLI's lattice commands; by serial) ===
chloros_sdk.camera_color_profile(serial) / set_camera_color_profile(serial, ...)
chloros_sdk.camera_color_adjust(serial) / set_camera_color_adjust(serial, ...)
chloros_sdk.camera_highlights(serial) / set_camera_highlights(serial, value)
chloros_sdk.camera_white_balance_once(serial) / set_camera_white_balance(serial, ...)
chloros_sdk.camera_presets() / apply_camera_preset(serial, preset)
chloros_sdk.camera_filter_models() / camera_filter_model_info(name)
chloros_sdk.camera_firmware_status() / update_camera_firmware(serial)
chloros_sdk.camera_power_probe(serial) / sleep_camera(serial) / wake_camera(serial)
chloros_sdk.calibrate_camera_with_target(serial, ...)
chloros_sdk.network_adapters(...) / fix_network_settings()
chloros_sdk.apply_vignette_correction(input_dir, output_dir, ...)
chloros_sdk.read_dls(mac, ...) / probe_system(...) / system_info()

# === Transient cameras, alignment profiles, indices (cameras opened for one command) ===
chloros_sdk.start_camera_continuous(serial, ..., daq=None, daq_tolerance_ms=None)  # → CliJob; a DAQ bound to the camera records one .daq per frame (daq=False: none)
chloros_sdk.start_multi_camera_stream(serials, ...)   # → CliJob
chloros_sdk.capture_cameras_transient(serials, ...)
chloros_sdk.test_array_sync(serials, count, ptp_settle_s=None) / detect_array_master(line) / analyze_array_models(models)
chloros_sdk.calibrate_alignment_profile(serials, save_path, ...)
chloros_sdk.capture_aligned_multiband(profile_path, ...) / start_aligned_stream(profile_path, ...)  # → dict / CliJob
chloros_sdk.alignment_profile_info(path) / reorder_alignment_profile(path, ...)
chloros_sdk.index_catalog() / compute_index(input_path, ...) / capture_live_index(profile_path, ...)
chloros_sdk.CliJob                                # stats() / stop() / wait()

# === Project lifecycle ===
chloros_sdk.open_project(path, relink=None)       # → ChlorosProject
chloros_sdk.project_folder() / set_project_folder(path) / reset_project_folder()
chloros_sdk.run_project_recipe(recipe, project_path)  # `chloros-cli project run`
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics               # DLS: DAQ-M over Bluetooth; needs bleak

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosProjectFilesMissingError      # a ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True when chloros_sdk.project imported; open_project()
                                 # still raises ImportError without the project loaders
```

---

## Zpracování obrazu — `ChlorosLocal`

Hlavní třída pipeline. Při instanciování spustí backend (pokud žádný neodpovídá a `auto_start_backend=True`), vytvoří a nakonfiguruje projekty, sleduje průběh a po dokončení vrátí souhrnné údaje.

### Konstruktor

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Metody

| Metoda | Popis |
| --- | --- |
| `create_project(project_name, camera=None)` | Vytvoří nový projekt (volitelně s šablonou kamery, jako je `"Survey3N_RGN"`). |
| `open_project(project_path, relink=None)` | Otevře uložený projekt (jeho složku nebo jeho název ve složce projektů) a nastaví jej jako otevřený projekt. Cesty přesunutého projektu jsou přesměrovány na složku, ve které se nyní nachází; `relink` pojmenovává složku, ve které se nyní nacházejí ostatní chybějící soubory. Vrací odpověď backendu pomocí `source_files`; vyvolá výjimku `ChlorosProjectFilesMissingError`, pokud nelze žádný soubor najít. Viz **Přesunutý projekt** níže. |
| `import_images(folder_path, recursive=False)` | Import obrázků ve formátech RAW/TIF/JPG/DNG **a `.daq` záznamů ze světelného senzoru**. Vrací `count` (obrázky) a `scan_count` (záznamy). Vydá varování pouze v případě, že složka neobsahuje ani jedno z nich. |
| `export_light_sensor(daq=True, csv=True)` | Zapíše kalibrované soubory `.daq` + `.csv` pro každý záznam ze světelného senzoru v projektu do složky `<project>/Light Sensor/`. Viz [Záznamy světelného senzoru](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Nastavit ovládací prvky zpracování. `indices` jsou názvy předvoleb (`NDVI`, `GNDVI`, `OSAVI`, …), z nichž každý je přidán do projektu pro příslušná pásma dané kamery; název, který není předvolbou, nebo který daná pásma nemohou vytvořit (`NDRE` vyžaduje RE a NIR, což žádná jednotlivá kamera neposkytuje), dostane název `UserWarning` a žádný indexový obrázek. `daq_log_path` označuje záznam ze světelného senzoru (soubor typu `.daq` nebo `.csv`, případně složku s těmito soubory), s nímž se porovnává každý snímek, a to jak v LATTICE, tak v Survey3, namísto záznamů importovaných spolu s obrázky; pokud je cesta neexistující nebo neobsahuje žádný záznam, použijí se importované záznamy. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Spustí zpracování. Vrátí `{"status": "complete", "async": False}`, plus klíč `summary`, pokud jej backend poskytne — viz [Souhrn a tipy po spuštění](#post-run-summary--hints) — `export_check_failure`, pokud nebylo možné ověřit výstupy běhu, a `image_failure`, pokud došlo ke ztrátě obrazu. `wait=False` vrací `{"status": "started", "async": True}` najednou. |
| `get_config()` / `get_status()` / `status()` | Zkontrolujte stav backendu. |
| `logout()` | Odhlášení: vymaže uložené přihlašovací údaje z mezipaměti a uvolní slot zařízení tohoto počítače (ve výsledku se zobrazí `device_deregistration`). Probíhající cyklus zpracování není zastaven: dokončí se v režimu, ve kterém byl spuštěn (`run_in_progress: true` a `run_mode` ve výsledku), a následující cyklus použije standardní zpracování. |
| `shutdown_backend()` | Zastaví backend, pokud jej spustila tato instance; backend, který byl nalezen jako spuštěný, zůstane beze změny. Zanechání bloku `with` nebo provedení garbage collection na instanci jej nezastaví: backend pokračuje v běhu pro další skript a zastaví se sám po 30 minutách, pokud jej nic nevyužívá. Backend je vyzván k vypnutí, takže nejprve uvolní své kamery a světelné senzory a dokončí veškeré nahrávání; je ukončen pouze v případě, že o 10 s později stále běží. |
| `discover_cameras()` | Vyhledá kamery LATTICE **prostřednictvím backendu této instance** (`/api/camera/discover`). Vrací seznam slovníků (`serial`, `model`, `ip`, …) — ve stejném formátu, jaký vidí GUI/CLI. Prázdný seznam, pokud nebyly nalezeny žádné kamery nebo je backend nedostupný. |
| `camera_capture(output_dir, format="tiff", **settings)` | Zachytí jeden snímek**prostřednictvím backendu**(automaticky spuštěného tímto identifikátorem), takže projde stejnou přípravou jako v GUI/CLI (výchozí 12 bitů, opětovné použití poolu, vložená metadata kalibrace). Cíl vyřešte pomocí `serial=` nebo `device_index=`; předávejte `exposure`/`gain`/`pixel_format`/`preset` jako `**settings`. Vrací slovník starších metadat (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Vytváří překryvné kompozitní náhledové snímky z sdružené kamery — odlehčený MJPEG klient přes trasu `/api/camera/<serial>/stream-annotated` backendu (zebra / mřížka / nitkový kříž / histogram / peaking / bod nakreslený na straně serveru). `decode=True` poskytuje pole BGR a vyžaduje OpenCV (`cv2`), které SDK neinstaluje; `False` generuje surové bajty JPEG. Dostupné také na úrovni projektu jako `ChlorosProject.stream(overlays=True)`. |

**Spuštěný proces si zachovává svůj projekt.** Zatímco probíhá zpracování — ať už tohoto skriptu, aplikace, `chloros-cli process` nebo `chloros-mcp` — backend odmítá vytvořit, otevřít, importovat do projektu nebo změnit jeho nastavení zpracování (HTTP 409 `PROCESSING_IN_PROGRESS`): `create_project()` a `import_images()` vyvolají výjimku `ChlorosProcessingError`, a `configure()` vyvolává `ChlorosConfigurationError`, a `process()` s jiným režimem než je ten aktuálně spuštěný vyvolává `ChlorosProcessingError` (spuštění si zachovává režim, ve kterém bylo zahájeno), vždy se zprávou backendu. Zachycená data uložená do projektu se k němu připojí po skončení spuštění (uvádí to odpověď `project_add`). Počkejte, až se běh dokončí (`get_status()`), a pak pokračujte.**Přesunutý projekt.** `project.json` projektu pojmenovává každý zdrojový soubor (každý obrázek a záznam ze světelného senzoru) podle jeho úplné cesty, takže přesunutí složky projektu – nebo celé složky „projects“ na jiný disk – způsobí, že soubory budou stále odkazovat na původní umístění. Při každém otevření se opraví, co se dá, automaticky: soubor, který byl původně ve složce projektu, se vyhledá ve složce, kde se projekt nyní nachází, a každá nalezená cesta se přenastaví a uloží (`project.json` je nejprve zálohován jako `project.json.before-relink-<date>-<time>`). Cesta je vždy přesměrována pouze na soubor, který se v daném umístění skutečně nachází. Soubory, které se nacházely mimo složku projektu – relace hubu, import pomocí funkce „Přidat složku“ – se najdou pouze v `relink=<the folder they are in now>` (nebo v jakékoli nadřazené složce). `open_project()` vrátí `source_files`: `total`, `found`, `missing`, `missing_roots` (složky, ve kterých se nacházely chybějící soubory), `missing_outside_project` a `relink` (co bylo přesměrováno, odkud kam). Projekt, ve kterém některé soubory chybí, se otevře se zbývajícími soubory; projekt, ve kterém nebyly nalezeny žádné soubory, se neotevře, a `open_project()` vyvolá chybu `ChlorosProjectFilesMissingError` (`.missing_roots`, `.audit`; HTTP 409 `PROJECT_SOURCES_MISSING`). Chyby `project open --relink` a `chloros-mcp` v CLI`open_project` (`relink_folder`) dělají totéž.

Použijte jako správce kontextu pro zaručené uvolnění:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Záznamy ze světelného senzoru — kalibrované `.daq` + `.csv`

Záznamy z DAQ-U / DAQ-M / DAQ-E lze pořizovat **bez** jejich kalibračního balíčku. To je to,
co veřejně dostupné záznamníky [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) ve výchozím nastavení: zapisují surová data ze senzorů a označují
soubor tak, aby Chloros nejprve načítal tovární kalibraci daného senzoru **podle sériového čísla** — nejprve z lokální mezipaměti, poté z cloudu MAPIR — a aplikuje ji při importu.

Chloros zapíše výsledek zpět jako dva produkty na jeden záznam, pod
`<project>/Light Sensor/`:

| Produkt | Co to je |
| --- | --- |
| `<name>_calibrated.daq` | Archiv vhodný k opětovnému zpracování — stejné schéma jako živý záznam, nyní s uvedením balíčku, který jej vytvořil. Jeho opětovný import **ne** provede kalibraci podruhé. |
| `<name>_calibrated.csv` | Spektrální ozáření v W/m²/nm na vlastní vlnové mřížce senzoru, jeden řádek na jedno měření, plus fotometrické sloupce (celkový výkon, fotopický/skotopický lux, PPFD a jeho rozdělení na modrou, zelenou a červenou složku, špičková vlnová délka). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Pouze senzory bez balíčku (DAQ-A).** Surové spektrální počty senzoru — *ne* ozáření. Viz níže. |

`<name>` je název souboru daného záznamu. Dva záznamy v různých složkách mohou sdílet
jednu hodnotu — měření za jednotlivé snímky (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) uloženou ve
stejné milisekundě dvěma kamerami — a každá z nich přesto získá své vlastní výstupy: první si ponechá
`<name>`, další přidává složky, které jej odlišují **za** názvem
(`<name>__<capture>_daq_calibrated.daq`), takže datum a čas záznamu jsou stále na prvním místě.

`process()` provádí tento export jako jednu ze svých fází. **Nevyžaduje** snímky:
samostatně letící světelný senzor představuje prvotřídní pracovní postup a takový projekt má ze své podstaty nulový
počet snímků.

**Záznamy DAQ-A se exportují jako surové počty.** Řada DAQ-A předchází systému svazků podle sériového čísla
a nemá žádný svazek, který by bylo třeba načíst — místo toho se kalibruje v terénu proti
reflexnímu terči, a proto žádný svazek nikdy nepotřebovala. Tyto záznamy se exportují
pod kořenem `_raw` namísto `_calibrated`: jde o odlišný název souboru namísto příznaku
uvnitř souboru, protože název musí zůstat zachován i při odeslání e-mailem jako holý název.
Záhlaví `.csv` uvádí `raw spectral sensor counts (NOT irradiance)` a upozorňuje, že
hodnoty jsou srovnatelné **v rámci** souboru — přesně to, k čemu slouží kalibrace pomocí terče
— a nikoli napříč senzory. Fotometrické sloupce závislé na výkonu (celkový výkon,
fotopický/skotopický lux, PPFD) se vrací jako **NULL** namísto integrace z počtů.

Zařízení DAQ-U / DAQ-M / DAQ-E, jehož balíček se prostě nepodařilo načíst, je stále **přeskočeno**,
nezaznamenává se v surové podobě: v takovém případě balíček existuje a „znovu se připojit a znovu zpracovat“ je skutečná rada.

Starší záznamy **v1.01 / v1.02** (tyto zapisuje DAQ-A-SD) neobsahují epochu pro jednotlivé odečty,
pouze čas zápisu souboru. Porovnávač obrazu↔sestupného toku je stále odmítá — porovnání
s časem zápisu by vedlo k neviditelné chybě — exportér je však načte a
soubor CSV vypíše `clock=daq_created_on`, takže produkt uvádí, na jakém časovém systému se nachází.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Záznam, jehož kalibrační balíček nelze načíst (offline nebo senzor bez
kalibrace v souboru), je nahlášen pod `skipped` **s uvedením důvodu**. Nikdy se
nezapíše jako „kalibrovaný“ soubor obsahující surová měření — připojte se k internetu a
spusťte proces znovu, export se dokončí.

### Zpětná volání o průběhu

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Shrnutí a tipy po dokončení

Po dokončení `process()` načte `GET /api/processing-summary` a připojí tělo jako `result["summary"]`. Načítání probíhá podle nejlepšího úsilí a nikdy neblokuje úspěšný návrat — pokud není souhrn k dispozici, `process()` se vrátí k jednoduchému formátu `{"status": "complete", "async": False}`. Každý záznam v `summary["hints"]` — celé věty s navrhovaným řešením, např. proč běh vyprodukoval nulový výstup — je rovněž znovu vygenerován jako Pythonový `UserWarning`, takže běhy s nulovým výstupem jsou samodiagnostické, i když slovník nikdy neprohlížíte:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` je strojově čitelná polovina:

| Klíč | Co počítá |
| --- | --- |
| `models` | Skupiny kamer v běhu. |
| `images_in_groups` | Zdrojové snímky v těchto skupinách. |
| `targets_found` | Detekované reflektanční cíle. |
| `images_calibrated` | Snímky, pro které běh vytvořil produkt odrazivosti. |
| `images_failed` | Snímky, u kterých došlo k selhání a nebyly exportovány. |
| `exported_files` | **Soubory obrazových produktů vytvořené v rámci běhu**: všechny produkty (odbyzerované, náhledy, radiance, odrazivost, index, rendery cílů) ve všech formátech. |
| `reflectance_files` | Mezi nimi jsou produkty odrazivosti (`Reflectance_Calibrated_Images`), ať už byly kalibrovány pomocí cíle nebo ozáření světelného senzoru (DAQ). Běh bez cíle, který přesto zapsal odrazivost, to uvádí v poznámce; pouze u běhu, který nezapsal nic, je uvedeno, že výstupy odrazivosti chybí. |
| `clamp_masks_written` | [Masky omezující smíchání](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) uvedené vedle nich — jedná se o doplňkové údaje, nikoli o výstupy, takže nejsou v `exported_files` (CLI je počítá odděleně stejným způsobem). |
| `captures_left_out` | Surové snímky ve složce `captures` projektu, které tento běh nepořídil. Spuštění při svém zahájení čeká na to, co bylo zachyceno před ním, takže tato hodnota je 0, pokud během spuštění nebylo uloženo žádné zachycení nebo pokud nevypršela doba čekání. `summary["captures_left_out_note"]` uvádí, jak s nimi naložit. |
| `daq_recordings_exported` / `daq_recordings_skipped` | Záznamy ze světelného senzoru, záměrně počítané samostatně — pocházejí z jiné fáze a existují i u běhů, které neobsahují žádné snímky, takže jejich zahrnutí by způsobilo, že by běh pouze s DAQ vypadal, jako by exportoval snímky. |

Vedle nich: `summary["output_dirs"]` (každý adresář, do kterého se zapisovalo),
`summary["light_sensor_export"]`, `summary["stopped"]` (platí, když uživatel přerušil
běh, aby se částečné počty neinterpretovaly jako dokončený běh s nedostatečným výstupem) a
`summary["groups"]` (rozpis podle skupin).

`exported_files` je zaznamenáván potrubím **při zápisu**, nikoli dodatečně načítán z
objektů obrazu projektu. Paralelní a GPU strategie vytvářejí své vlastní objekty obrazu
(v podprocesech pracovníků pro GPU cesty), takže staré skenování hlásilo
`0 file(s) written` a následně vyslal nápovědu o nulových exportů — u běhů,
kde vše fungovalo. Pokud vytvoříte skript na základě tohoto čísla, zdravý paralelní běh nyní
hlásí nenulový počet.

Přeskočení světelným senzorem hlásí důvod, který čtečka skutečně zjistila pro každý soubor —
nečitelné schéma, chybějící balíček, chyba zápisu — **deduplikovaný**, takže dvacet souborů
přeskočených z jednoho důvodu se vyhodnotí jako jeden důvod, nikoli jako dvacet jeho opakování.

> **`process()` se nevypíše, pokud běh nevytvoří žádné obrazy.** Toto je jedno ze dvou míst, kde se SDK a
> CLI záměrně liší: `chloros-cli process` považuje „byly požadovány výstupy, žádné nebyly
> zapsány“ za selhání a ukončí se nenulovým stavem, zatímco SDK se vrátí normálně a nahlásí
> stav prostřednictvím `summary` / hints. Pokud by se váš pipeline měl zastavit při prázdném běhu, zkontrolujte to
> sami — prozkoumejte `summary` (nebo spočítejte soubory ve složce projektu), místo abyste se spoléhali na
> nepřítomnost výjimky. Obvyklými příčinami jsou vstupní složka, která nebyla rozpoznána jako
> záznam, a produkty přeskočené jako nevhodné pro přítomné kamery (např. radiance z kamer podporujících pouze RGB
>).
>
> `process()` se nevyvolá ani v případě, že nebylo možné ověřit výstupy běhu: zobrazí varování a nastaví
> `result["export_check_failure"]`. Výstupy jsou zapsány. Chcete-li proces v tomto případě zastavit, zkontrolujte tento klíč. Stejně se chová běh, u kterého
> došlo ke ztrátě snímku: zobrazí varování a nastaví `result["image_failure"]`, při jehož nenulové hodnotě
> `chloros-cli process` ukončí běh.

### Pomocné funkce

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "GNDVI", "OSAVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# process_folder() / process_lattice_capture() also take input_level=,
# array_alignment=, array_alignment_crop=, array_alignment_interpolation=,
# progress_callback=, poll_interval=, timeout=, processing_stuck_timeout=,
# backend_exe= and api_url= (NOT backend_url=). indices defaults to ["NDVI"].

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

Každý běh probíhá uvnitř vlastního `with ChlorosLocal(...)`; backend, který musel spustit, pokračuje v běhu i po návratu, pro další volání, a zastaví se sám po 30 minutách, pokud jej nikdo nepoužívá. Ani jedna z nich nepřijímá `auto_start_backend`, `radiometric_output` ani `daq_log_path`; pro tyto případy použijte `ChlorosLocal` a `configure()`. `process_lattice_capture()` nepřijímá `camera`, `debayer` ani `ppk`: přeskočí detekci panelu a použije standardní debayer.

### Podporované hodnoty

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"TIFF (8-bit)"            # all-8-bit images are written as this; never raised
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer for Survey3 imagery, Chloros+ only.
                                         # Survey3 only: on a project holding LATTICE captures
                                         # (which always use the standard demosaic) configure()
                                         # or process() raises ChlorosConfigurationError; a
                                         # Texture Aware value the project saved earlier is set
                                         # to Standard with a UserWarning, as the app does. Needs a
                                         # CUDA GPU the backend's PyTorch can use (an NVIDIA
                                         # GPU, or a Jetson Orin's). Without one, process()
                                         # uses the standard debayer, and the backend prints
                                         # one "[DEBAYER] ... needs a CUDA GPU" line saying why.
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as Chloros's saved debayered (3-channel) product
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Radiometrický výstup (multispektrální pipeline LATTICE)

Úroveň exportu multispektrálních dat (M3C/M3M) z potrubí `process` — `reflectance` (výchozí), `radiance`, `sensor-response` nebo `all` (všechny příslušné režimy pro každý snímek) — nastavuje **„Exportovat radianci“** a **„Exportovat odrazivost“**: `radiance` exportuje pouze radianci, `reflectance` pouze odrazivost, `sensor-response` ani jedno (pouze produkty po odstranění bayerského filtru a náhledy), `all` exportuje obojí. Produkty po odstranění Bayerova vzoru a náhledové produkty nelze na této úrovni měnit. `configure()` má pro to vyhrazené klíčové slovo:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Pokročilá úniková cesta — zápis produktů prostřednictvím `custom_settings` — stále funguje, ale mějte na paměti, že nahrazuje celý blok nastavení (viz varování níže). Zadejte klíče `"Export <product>"`: vlastní nastavení projektu `"Export radiance"` / `"Export reflectance"` mají přednost před starším klíčem `"Radiometric output"`, takže tento klíč sám o sobě nic nemění.

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Export radiance": True, "Export reflectance": False},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (výchozí nastavení) dělí radianci kamery **sestupným signálem DAQ s odpovídajícím časovým razítkem**, který se automaticky vypočítá ze zaznamenaného klíče `.daq` (DAQ-U/M/E)**nebo nativního produktu typu DAQ-M `.csv`**nalezeného společně se snímky; jakýkoli balíček kalibrace pro jednotlivé kamery nebo DAQ, který lokálně chybí, se při prvním použití**automaticky načte z AWS**. Rozhraní CLI to zpřístupňuje podlepřepínače typu produktu na `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.**Čí je to naměřená hodnota?**Naměřená hodnota uložená v záznamu typu `.daq` je naměřená hodnota kamery (kamer), kterou popisuje — pojmenovaná v doprovodném souboru typu `<file>.daq.cameras.json` vedle ní (jednotlivé, synchronizované a pole záznamů; sériový záznam v surovém formátu uvádí název záznamu každého člena ve svém manifestu) — a je přiřazen**pouze k obrázkům z těchto kamer**. Samostatná kamera bez vlastního záznamu proto již nepoužívá DAQ jiné kamery; přiřaďte k ní DAQ nebo proveďte vlastní záznam. Vlastní záznam DAQ (`chloros-cli daq pool-record` nebo záznam spuštěný v grafickém rozhraní) či jakýkoli soubor `.daq` bez tohoto souboru sidecar (včetně starších záznamů) stále slouží všem kamerám.**Kamery OCN na slunci.** Každý kalibrační balíček OCN je sestaven za úzkopásmového osvětlení, s jednou LED diodou na pásmo, zatímco sluneční světlo pásma mísí. Chloros 1.3.0 po oddělení pásem aplikuje hromadnou korekci založenou na slunečním světle, což zlepšuje produkty OCN pro radianci, odrazivost a index ve slunečním světle. Funguje všude tam, kde probíhá oddělení pásem — `process()`, snímky, `grab()` / `frame_stream()` a živý náhled — u balíčkOCN, které to podporují; kamery RGN zůstávají beze změny. Snímky OCN pořízené starší verzí tuto korekci také získají, pokud jsou zpracovány na počítači, který obsahuje kalibraci, s níž byly pořízeny (Chloros si uchovává kalibraci kamery, jakmile je kamera připojena); v opačném případě jsou zpracovány jako dříve. Hodnoty radiance, odrazivosti a indexu OCN se proto liší od hodnot verze 1.2.0.

> `custom_settings` **nahrazuje** celý blok vypočítaných nastavení (záměrně obchází ostatní klíčová slova a ověření `configure()`). Při jeho použití zahrňte všechna klíčová slova `Project Settings`, na kterých vám záleží, jako v příkladu výše.

#### Čtení indexových pixelů (index NDVI a další vegetační indexy)

**Celé číslo indexové rastry představují roztažení kontrastu pro jednotlivé snímky, nikoli mapu v rozsahu −1…1.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` a `JPG (8-bit)` — včetně 8bitového souboru TIFF, do kterého se zapisuje 8bitový snímek — ukládají

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

kde `IndexMin` / `IndexMax` jsou **nejnižší a nejvyšší konečné hodnoty indexu daného snímku**. DN 0 není −1 a plný rozsah není +1.

Systém Chloros označuje oba koncové body, aby mapování zůstalo invertibilní:

| Značka | Význam |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — konvence uvedená níže |
| `Chloros:IndexMin` | hodnota indexu uložená jako DN 0 |
| `Chloros:IndexMax` | hodnota indexu uložená jako plný rozsah |

Přečtěte je a invertujte, přičemž nejprve normalizujte uložený DN na rozsah 0…1 podle maximální hodnoty uloženého datového typu, aby jeden výraz pokryl všechny formáty celých čísel:

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

> **`IndexMin == IndexMax`** znamená, že každý pixel daného snímku měl stejnou hodnotu indexu. Rastr je vyplněn v polovině rozsahu a není čím dělit — vezměte koncovou hodnotu.

> **Žádné značky `Chloros:Index*` znamenají, že není zadáno žádné mapování.** Export LUT je kolorovaný výstup, jehož pixely představují barevné přechody vázané na prahové okno vaší LUT, a snímek bez pixelu s konečným indexem nemá žádný rozsah, který by bylo možné uvést. V žádném z těchto případů nepředpokládejte existenci mapování — platí stejné pravidlo jako u chybějícího `PixelScale`.**Vzhledem k tomu, že roztažení se provádí pro každý snímek zvlášť, nelze dva snímky z jednoho letu porovnávat z hlediska DN**— každý má své vlastní koncové body, takže stejná hodnota NDVI odpovídá v různých snímcích odlišným hodnotám DN. Pro kvantitativní analýzu v rámci jednoho letu (zónové statistiky, pevné prahové hodnoty, časové řady) exportujte**`TIFF (32-bit, Percent)`**: indexové rastry jsou zde zapsány jako float32**samotné hodnoty indexu**, bez nutnosti přepočítání a bez nutnosti tagů. Navzdory názvu formátu nejsou indexové produkty násobeny ×100 — toto škálování se vztahuje pouze na odrazivost s celočíselným datovým typem.

#### Maska omezovače při odstraňování spektrálního smíšení — na kterých pixelech bylo pásmo omezováno

Spektrální odstraňování smíšení 3×3 může způsobit, že se pásmo stane záporným — tam, kde světlo dané pásmo téměř neobsahuje, nebo tam, kde scéna leží mimo rozsah popsaný maticí odstraňování smíšení kamery. To není skutečná hodnota, takže export ji nastaví na 0 — a živý náhled zobrazuje stejnou hodnotu 0. Pásmo může být omezováno na několika pixelech nebo téměř na všech; maska zaznamenává obojí. `Chloros:Band{R,G,B}UnmixClampFraction` udává, jak velká část snímku tím byla ovlivněna; maska omezování udává, **které pixely**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— ve složce `clamp_masks/`**vedle** složky produktu, nikdy uvnitř ní (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`) — pro každý produkt radiance, odrazivosti a indexu, ve kterém došlo k omezování u alespoň jednoho pixelu. Jednopásmový `uint8`, na vlastní pixelové mřížce produktu (včetně orientace, ROI, binning a warp pro zarovnání pole v ceně). Každá maska uvádí název kamery daného produktu (`Make` / `Model` / sériové číslo), čas pořízení a značky objektivu a uvádí seznam pásem, která její bity představují (`XMP-Camera:BandName`, nejprve bit 0); její `ImageDescription` a `Software` ji označují jako omezovací masku a neobsahuje žádnou kalibraci.
- **Bity**: bit 0 = pásmo v kanálu R produktu (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Nastavený bit znamená, že hodnota pixelu v daném pásmu byla oříznuta na 0 — považujte jej a jakýkoli index z něj odvozený za málo spolehlivý.
- **`Chloros:UnmixClampMask`** je vyznačeno na každém produktu, u kterého byl spuštěn proces oddělování: cesta k masce relativně ke složce samotného produktu (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, vždy odděleno znakem `/`odděleno mezerou — připojte ji ke složce obrázku, abyste masku otevřeli), nebo `none`, pokud nebyl oříznut žádný pixel. Chybějící značka znamená, že u produktu nebyl nikdy proveden proces unmix (kamery RGB nebo monochromatické, úrovně raw/debayered).
- Výsledky snímání jej obsahují jako **`clamp_mask`** snímku (cesta k masce, nebo `None`, pokud není nic oříznuto) — ve výsledcích `CameraSession.capture()` jsou to snímky `ArraySession.capture()`snímcích `CaptureResult` a v HTTP-odpovědích na snímání. Offline sestavení uvádí své masky jako `clamp_masks`.
- Vícepásmový zásobník živého indexu (`/api/camera/cli-array-index` s `save_multiband`) získává jednu masku na každou zdrojovou kameru, a to ve vlastní složce výstupu `clamp_masks/` — viz referenční příručka CLI`lattice index`. Video nemá žádnou masku: výsledek offline sestavení místo toho uvádí podíl oříznutí každého snímku.

Protože žádná složka produktu neobsahuje masku, a to v žádné úrovni, lze složku produktu předat fotogrammetrickému softwaru tak, jak je – včetně podsložek – a ten přijímá pouze snímky. Předávejte mu složku produktu, nikoli složku nad ní. Vlastní import programu Chloros masky přeskočí.

```python
import os, numpy as np, tifffile

mask = tifffile.imread(
    "clamp_masks/radiance/capture_<ts>_SN<serial>-<filter>_radiance_clampmask.tif")
# or, from the image: os.path.join(os.path.dirname(image_path), tag), with
# tag = the image's Chloros:UnmixClampMask
r_clamped = (mask & 1) != 0      # the band Chloros:BandRName names
g_clamped = (mask & 2) != 0
b_clamped = (mask & 4) != 0
```

---

## Smart-Connect pro kamery LATTICE

Trvalé backendové relace pro živý hardware. Používají se stejné koncové body jako v grafickém rozhraní, takže chování je identické v SDK, CLI i GUI.

### Jedna kamera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

Expozice, zisk a cíl zde nastavené zůstávají zachovány jako vaše: automatické řízení expozice systému Chloros (včetně expozice měřené DAQ) je neovlivňuje. `cam.set_settings(exposure_auto="Continuous")` (nebo `gain_auto=`) předává expozici (zisk) zpět, a `cam.set_settings(target_brightness_auto=True)` vrací cíl zpět automatickému nastavení cíle v Chlorosu.

#### `connect_camera()` Podpis

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

- **Vrací se se spuštěným datovým tokem.** Backend čeká, s časovým omezením (asi 15 s na spuštění streamu, poté 10 s na první kompletní buffer — ne na první náhledový snímek, který následuje asi o 1,5 s později), takže `status()` hned poté, co se objeví `streaming`. `connect_stream` relace je blok `stream` odpovědi: `state` `streaming`, `no_frames` (datový proud byl spuštěn, ale nic nedoručil — to uvádí `note` a u kamery s jumbo pakety je třeba se znovu připojit s velikostí paketu 1500 bajtů), `starting` nebo `start_failed`, kde `waited_s`. `None` pro kameru, která již byla připojena. Náhled (`get_camera_preview`, `/api/camera/jpeg/<serial>`) vyžádaný hned po čekání, omezeném na 4 s, na první náhledový snímek kamery namísto odpovědi „Zatím není k dispozici žádný snímek“; kamera, jejíž datový proud zatím nic nedodal (`no_frames`, nebo připravená na hardwarový spouštěč), stále odpoví okamžitě.
- **Velikost paketu si můžete nastavit sami.** `settings={"gev_packet_size": 1500, "gev_packet_size_auto": False}` otevírá kameru s pakety o velikosti 1500 bajtů, bez testování (`chloros-cli lattice cam-connect --packet-size standard` odesílá totéž).
- **Kamerový fond je obsazen vždy pouze jedním připojením**: otevření, test velikosti paketu, první stažení kalibračních dat (cca 70 s u kamery, která je na tomto počítači nová), spuštění streamu. Žádost, která mezitím potřebuje kameru (`read_nodes`, `set_settings`, záznam, druhé připojení) na ni čeká maximálně 5 s, poté selže s chybou HTTP 503 `CAMERA_POOL_BUSY`, přičemž uvede název připojující se kamery (`ChlorosConnectError`); zeptejte se po chvíli znovu. Seznamy nikdy nečekají: `list_cameras()` (`GET /api/camera/pool`), `list_arrays()` a `CameraSession.status()` odpovídají okamžitě. Kamera uprostřed připojování je uvedena pod odpovědí skupiny `connecting` (`serial`, `phase` — `opening the camera`, `rebalancing the pool bandwidth`, `loading its calibration`, `starting its stream`, `waiting for its stream` — a `elapsed_s`), a jeho `status()` uvádí `connecting: True` s jeho `phase`; ostatní kamery se čtou v reálném čase, pokud je pool volný, jinak se odpovídá na základě jejich posledního čtení (`live: False`).

#### Metody `CameraSession`

| Metoda | Popis |
| --- | --- |
| `status(timeout=15.0)` | Aktuální stav kamery, jak jej načítá `chloros-cli lattice status`: `streaming`, `stream_state` (`starting`, `no_frames`, `streaming`, `start_failed` nebo `stopped`), `device` (model, sériové číslo, firmware, teplota, `is_mono`) a jeho aktuální nastavení. |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Načte uzly GenICam; vrátí `{nodes, errors, enums, device, was_in_pool, in_array_id}`, plus `frame_values` a `notes` během běhu automatické expozice (viz níže). |
| `set_settings(**kwargs)` | Zápis do uzlů podle popisného názvu (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `target_brightness_auto`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). Hodnota `exposure_time` (`gain`) představuje ruční expozici (zisk): při zadání bez `exposure_auto` (`gain_auto`) se nejprve vypne automatický režim, aby hodnota nebyla odmítnuta běžícím automatickým režimem. Výsledná hodnota `applied` zachycuje to, co kamera zaznamenala; zápis, který fotoaparát odmítne, se nachází v `errors` a není uchován. Chyba udává rozsah, který fotoaparát nyní akceptuje (maximum `ExposureTime` se řídí snímkovou frekvencí fotoaparátu), a automatický režim, který byl vypnut pouze kvůli odmítnuté hodnotě, se znovu zapne; `restored` jej uvádí (např. `{"ExposureAuto": "Continuous"}`). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, daq=None, daq_tolerance_ms=None, settings=None, timeout=None)` | Zachytit **jediný** snímek. Vrací `CaptureResult`: seznam slovníků metadat snímků (jeden pro každou uloženou úroveň), s `.skipped`, `.failed`, `.daq_files` a `.daq_matches`. (Sériové/více snímkové zachycování bylo odstraněno — pokud potřebujete sérii, zavolejte `capture()` ve smyčce.) |
| `capture_repeated(output_dir="output", *, count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Snímání v režimu „back-to-back“ (`interval_s=0`) nebo v intervalech z této kamery, pro snímání pomocí `count` a/nebo `duration_s` (alespoň jedno, aby se proces ukončil), stejně jako to dělá `ArraySession.capture_repeated()` pro pole. Vrací seznam `CaptureResult`; zastaví se po snímku, který hlásí zaplněný disk. `on_capture(i, result)` se spustí po každém z nich; ostatní klíčová slova směřují do `capture()`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Uvolnění z fondu, kamera se vrátí do stavu, v jakém byla nalezena při připojení (zapojení spouště, PTP, nastavení); `keep_settings=True` místo toho uloží nastavení do vlastní paměti, `clear_settings=True` tuto paměť vynuluje (viz [Paměť kamery](#camera-memory--keep_settings--clear_settings)); v případě požadavku na kteroukoli z těchto akcí vrátí `camera_memory`. Kamera, která je členem pole, uvolní celé pole. Nevykoná se žádná akce, pokud jsme se připojili k již otevřené relaci. Vrátí se, jakmile backend potvrdí uvolnění, nebo pokud není co uvolnit (kamera již neexistuje nebo neběží žádný backend); uvolnění, u kterého došlo k vypršení časového limitu nebo které backend odmítl, vyvolá chybu `ChlorosConnectError`, protože kamera může být stále zadržována a další připojení by s ní kolidovalo. Ukončení bloku `with` nikdy nevyvolá chybu. |

Atributy: `serial`, `device` (záznam o nalezení kamery) a `backend_url`.

Ovládací prvky exportu `capture()` (stejný model jako pole + grafické uživatelské rozhraní):

- `processing` / `levels` — `processing="all"` ukládá všechny použitelné typy exportu; `levels=["raw","radiance"]` ukládá pouze ty (přepíše `processing`). Pro výchozí nastavení backendu vynechte oba.
- `daq` — DAQ, který načítá použitý záznam, je ve výchozím nastavení uložen jako sidecar `.daq`, včetně záznamu pouze v surovém formátu, takže snímek lze později znovu zpracovat na odrazivost/index. `daq=False` jej přeskočí, pokud jej žádná úroveň nepoužívá. `force_daq=True` je starší zápis výchozího nastavení. Bez DAQ, žádný `.daq`.
- `daq_tolerance_ms` — jak daleko od expozice může být měření a stále být v toleranci (výchozí hodnota 500). Mění se pouze výsledek.

#### `read_nodes()` při automatické expozici — `frame_values` a `notes`

Zatímco běží `ExposureAuto` (`GainAuto`), uzel `ExposureTime` (`Gain`) načítá **poslední zapsanou hodnotu**, nikoli aktuální stav kamery. `nodes` stále uchovává tuto hodnotu uzlu. Pokud požádáte o `ExposureTime` nebo `Gain` u připojené kamery, jejíž automatický režim není `Off`, odpověď doplňuje:

- `frame_values` — hodnotu, kterou nesl nejnovější snímek kamery, pod názvem uzlu (`ExposureTime` v µs, `Gain` v dB);
- `notes` — jedna věta pro každý takový uzel, uvádějící obě hodnoty.

```python
r = cam.read_nodes(["ExposureTime", "Gain"])
print(r["nodes"]["ExposureTime"])                  # last value written, e.g. 2000.448
print(r.get("frame_values", {}).get("ExposureTime"))  # what the newest frame used, e.g. 19600.0
for name, note in r.get("notes", {}).items():
    print(note)
# ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448);
# the camera's newest frame was at 19600 us.
```

Oba klíče chybí, pokud je automatický režim `Off` (uzel pak představuje skutečnou hodnotu kameryskutečnou hodnotou kamery) a u kamery, která není připojena k backendu. U pole poskytuje `ArraySession.status()["member_ae"]` hodnoty snímků všech prvků najednou.

#### Horní limit expozice měřený DAQ (HTTP)

Je-li připojen světelný senzor DAQ, Chloros omezuje expozici × zesílení kamery na základě naměřeného denního světla, aby nedošlo k oříznutí bílého referenčního panelu. Tento limit platí pouze tehdy, když je kalibrační cíl v zorném poli dané kamery (zachycen v jejím snímku během posledních 10 s); pokud není cíl v zorném poli, funguje místo toho běžná automatická expozice kamery s vlastní ochranou proti přepálení, takže zbytek scény není ztmaven více, než je nutné. Neexistuje žádný SDK wrapper; tuto funkci lze načíst nebo přepnout pro jednotlivé kamery pomocí `GET` / `POST /api/camera/<serial>/daq-exposure-cap` (tělo požadavku POST s kódem `{"enabled": false}` tuto funkci pro danou kameru vypne). Odpověď obsahuje `enabled`, `cap_us` (horní limit v reálném čase, `None` v neaktivním stavu, včetně případů, kdy není v zorném poli žádný kalibrační cíl), `k_exp_g` (konstanta, pomocí které se horní hranice počítá) a:

| Pole | Význam |
| --- | --- |
| `k_exp_g_source` | Která konstanta se používá: `camera` — vlastní této kamery, vypočítaná z laboratorního bodu v jejím kalibračním balíčku (kamery RGB); `fleet` — konstanta flotily, používaná v případě, že sada neobsahuje laboratorní bod; `override` — přepsání pole nastavené na backendu. |
| `k_exp_g_fleet` | Konstanta flotily, pro srovnání. |

### Synchronizované pole — `ArraySession` (Smart-Prep)

`connect_array` je **doporučeným výchozím bodem** pro konfigurace s více kamerami. V pozadí spouští kompletní pracovní postup Smart-Prep s grafickým uživatelským rozhraním:

1. **Analýza sítě**(`/api/camera/array/recommend`) — nastaví**celý snímač** (bez binningu, bez oříznutí ROI) a nechá snímkovou frekvenci klesnout na úroveň, kterou poskytuje tento hostitel a síť, a to s ohledem na formáty pixelů a způsob balení, které bude Connect používat. Snímky se nikdy nezmenšují, aby se udržela vysoká frekvence: předávejte menší `width` / `height` nebo `binning=2` / `4`, abyste vyměnili rozlišení za snímkovou frekvenci (koncový bod`alternatives` uvádí tyto možnosti s jejich předpokládanými fps).
2. **Automatický výběr úrovně** — `sim-capture-sim-emit` (všechny kamery exponují na stejný spouštěcí signál, snímky jsou vysílány v taktu, aby se sdílel přenosový kanál; snímková frekvence klesá s tím, jak se sdílený kanál zužuje) v jakékoli síti, kde lze kamery synchronizovat tak, aby sdílely přenosový kanál; `slip-emit-and-capture` (kamery se na přenosové lince střídají, neexponují současně) pouze v případě, že to není možné.
3. **PTP povoleno**ve výchozím nastavení — časová razítka napříč kamerami se shodují s jedním sdíleným časem s přesností**~1 ms**, [PTP grandmaster](#ptp-grandmaster--time_sync_status) backendu. Současná expozice je zajištěna hardwarovým spouštěčem M8 (**< 100 µs** mezi moduly), nikoli protokolem PTP: PTP synchronizuje *časová razítka*, nikoli expozice.
4. **Automatický výběr formátu pixelů pro každou kameru** — černobílá → `Mono12`, barevná → `BayerRG12`.
5. **Inicializace AE** — pořídí snímky aktuálního stavu AE každé kamery, aby připojení během provozu nevynulovalo expozici.
6. **Konfigurace spouštění GPIO** — `connect_array` aktivuje všechny kamery (`TriggerMode=On`, `TriggerSource=Line2`), takžeimpuls řídí podřízené kamery přes kabel M8. Jedná se o krok určený pouze pro pole: samostatná kamera otevřená příkazem `LatticeCamera` místo toho běží v režimu volného běhu. Uvolnění pole (`disconnect()` / `force_disconnect()`, `chloros-cli lattice array-disconnect`) vrátí všechny kamery zpět do volného běhu (`TriggerMode=Off`), přičemž vše ostatní zůstane tak, jak to připojení zjistilo: nastavení spouště a synchronizační linky (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`; synchronizační linku hlavního zařízení zpět na vstup), PTP a nastavení kamer, která se během relace změnila – pokud se nezměnila `keep_settings=True` / `clear_settings=True` (viz [Paměť kamery](#camera-memory--keep_settings--clear_settings)).

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

Atributy: `array_id`, `master_serial`, `slave_serials`, `ptp_enabled`, `sync_mode`, `packed_transport` a `packed_reason` (poslední čtyři tvoří `None` na rukojeti z `attach_array`), `connect_stream` a `backend_url`. `connect_stream` je blok `stream` odpovědi na připojení, tvar jednoduchého připojení plus `members` (`{serial: {state, why?}}`): `streaming`, jakmile dorazí první synchronizovaná skupina pole (backend na ni čeká, s omezením přibližně 10 s), nebo `no_frames` s posledním selháním načtení každého člena jako `why` a, u jumbo paketů, náhradní hodnota 1500 bajtů v `note`. `None` na handle `attach_array`.

#### Podpis `connect_array()`

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # pin the master trigger rate (auto if None); see set_target_fps
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    packed="auto",                        # packed 12-bit transport: "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=600.0,                        # s; also the backend's deadline (see below)
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`timeout` udává, jak dlouho volání čeká na připojení, a je odesláno do
backendu jako jeho časový limit. Pokud do té doby nedojde k navázání spojení, je operace zrušena: každá kamera, kterou
otevřela, je vrácena do stavu, v jakém byla nalezena, a uzavřena, žádné pole není připojeno a
volání vyvolá chybu `ChlorosConnectError` (HTTP 504 `CLIENT_DEADLINE_PASSED`, s uvedením
kroku, do kterého se připojení dostalo). Volání, které vyvolalo výjimku, tedy nikdy nezanechá pole
za sebou žádné připojené pole. Pole s 9 kamerami se připojí za přibližně jednu minutu; hned po
zapnutí kamer to může trvat i několik minut. Pole, které je v provozu, ale ještě nemá
synchronizovanou skupinu snímků, je vráceno s chybou `connect_stream["state"]`
`no_frames`. Protokol backendu obsahuje jeden řádek s časem pro každý krok připojení
(`[array-connect <id>] <step>: <s> s …`) a poslední řádek označující nejpomalejší z nich.

Hodnoty `force_tier`:
- `"sim-capture-sim-emit"` — skutečně simultánní (všechny kamery se spouštějí na stejné hraně hodinového signálu).
- `"sim-capture-ftd-stagger"` — flexibilní časové rozložení (kamery vysílají v mírně posunutých časech, takže se pakety na vedení sériově řadí).
- `"slip-emit-and-capture"` — sekvenční snímání pro každou kameru (bez časové synchronizace). Volí se automaticky pouze v případě, že kamery nelze nastavit na dostatečně pomalý rytmus, aby mohly sdílet přenosovou linku současně.

`wire_ceiling_mbps` přepisuje **trvalý limit šířky pásma hostitele** v MB/s — jediné
číslo, na kterém závisí alokace celého pole. Nechte hodnotu `None` na automaticky detekované
hodnotě. Snižte ji, pokud pole hlásí poškozené rámce GVSP: automatická hodnota je odvozena
z inzerované rychlosti spojení síťové karty, která nadhodnocuje USB adaptéry, úzké PCIe linky a
vytížené sdílené struktury — a toto nadhodnocení se projevuje jako poškozené rámce spíše než jako
viditelně pomalé spojení. Hodnota je uložena v bloku zachycování pole projektu, takže při
opětovném otevření nebo pozdějším nastavení `connect_array` se obnoví stejně jako jakékoli jiné nastavení pole.
Viz [Stav pole](#array-health--which-subsystem-is-losing-frames).

#### Balený 12bitový přenos (`packed=`)

Při plném využití senzorů je snímková frekvence pole určena tím, jak rychle může každá kamera odesílat
své snímky. 12bitové formáty (`BayerRG12`, `Mono12`) se obvykle přenášejí jako 16bitová
slova; komprimovaný přenos odesílá stejných 12 bitů v 1,5 bajtu na pixel a
dekomprimuje je na hostitelském počítači, takže odeslání každého snímku trvá o 25 % méně času — přibližně
**33 % více snímků za sekundu při plném využití snímače** (naměřeno: 10,7 → 14,0 fps na
sestavě z 9 kamer). **Uložená data jsou identická**: stejné hodnoty, stejné 16bitové
soubory, stejná metadata.

- `packed="auto"` (výchozí nastavení) komprimuje pouze tehdy, když to má smysl **a** hostitelské zařízení si může
  dovolit dekompresi: při připojení backend změří svou vlastní rychlost dekomprese,
  počet jader CPU, aktuální zatížení a síťovou kapacitu. Stolní počítač s volnými jádry
  komprimuje při plném využití senzorů; zaneprázdněný nebo nízkoenergetický host (jednočipové počítače,
  vestavěné řadiče) streamuje nekomprimovaná data. Malé oblasti zájmu (ROI) a sjednocené snímky z toho mají jen malý prospěch.
- `packed="off"` (nebo `False`) nikdy nekomprimuje — pro vyloučení této možnosti při diagnostice
  nebo na hostiteli, který musí udržet své CPU volné pro jinou práci.
- `packed="on"` (nebo `True`) vždy komprimuje 12bitové formáty.
- Jakákoli jiná hodnota vyvolá `ValueError`. 8bitové a 16bitové formáty se nikdy
  nekomprimují.

`analyze_array_network(..., packed="auto")` se řídí stejným rozhodnutím,
takže jeho fps představuje rychlost, s jakou bude pole běžet (odpověď obsahuje
`packed_transport` a `packed_reason`); jeho `pixel_format` je ve výchozím nastavení nastaven na to, co
používá connect (Mono12 / BayerRG12), nikoli na 8 bitů.

Rozhodnutí a jeho důvod jsou uvedeny v relaci:

```python
arr = chloros_sdk.connect_array(serials)          # packed="auto"
print(arr.packed_transport, arr.packed_reason)
# True  auto: projected 11.0 -> 14.4 fps, unpack 0.16 of 1.50 budgeted cores

arr = chloros_sdk.connect_array(serials, packed="off")
```

`ChlorosProject.connect_all(packed=...)` použije stejné přepsání na každé
pole v projektu (výchozí hodnota `None`, což znamená `"auto"`: projekt neukládá žádné
nastavení komprimace pro jednotlivá pole).
Pro výchozí nastavení pro celý počítač nastavte `CHLOROS_ARRAY_PACKED12=off` (nebo `on`) v
prostředí backendu; výslovné nastavení `packed="on"` / `"off"` má stále přednost.
`/api/camera/array/status` hlásí `packed_transport` a `packed_decision`.

#### Očekávané snímkové frekvence

Naměřené snímkové frekvence pole (synchronizované hardwarem, 12 bitů; plný snímač = 2048 × 1536). Použijte je k ověření správnosti `status()["actual_fps"]`; `analyze_array_network()` odhaduje snímkovou frekvenci pro váš vlastní hostitelský počítač a síť.

| Nastavení | Snímková frekvence pole |
| --- | --- |
| Plný snímač, hostitelská síťová karta PCIe 10 GbE, komprimovaný přenos (`packed="auto"` na stolním počítači) | ~14 snímků za sekundu |
| Plný senzor, hostitelská síťová karta PCIe 10 GbE, nekomprimovaný přenos (`packed="off"`) | ~10,5 snímků za sekundu |
| Plný senzor, 8 kamer na adaptéru USB 10 GbE | ~3,8 snímků za sekundu |
| Plný snímač, hostitelské připojení 1 GbE | 1,2–1,7 snímků za sekundu |
| Binning 2 × 2 (`binning=2`), hostitelská síťová karta PCIe 10 GbE | ~30–33 snímků za sekundu |

- Na spojení PCIe 10 GbE jsou při plném využití snímače limitem **kamery**, nikoli síť: 4 i 8 kamer běží stejnou rychlostí.
- **`target_fps` představuje spouštěcí frekvenci a pole nikdy neběží rychleji.** `target_fps=1` provozuje pole rychlostí 1 fps; `None` (výchozí nastavení) jej provozuje nejvyšší rychlostí, kterou hostitelský počítač a síť zvládnou. Hodnota přesahující tuto rychlost se sníží na ni (viz [`set_target_fps`](#interval-capture-for-drone-surveys)).
- **`link_speed_mbps` z `4294` v `status()` není rychlost spojení.** Znamená to, že systém Windows nemohl přečíst rychlost adaptéru (ovladač nahlásil největší 32bitovou hodnotu), což je typické pro USB ethernetové adaptéry. Použitelná rychlost USB adaptéru ve Windows je v každém případě omezena na 200 MB/s (viz [Stav pole](#array-health--which-subsystem-is-losing-frames)); informace o ladění najdete v referenční příručce k příkazové řádce v části „USB 10GbE adaptéry“ pod [Nastavení a ladění hostitelské síťové karty](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Omezení pro USB platí i v systému Linux: 400 MB/s na x86_64, 200 MB/s na NVIDIA Jetson.** Adaptér je identifikován z sysfs (v případě selhání se použije `ethtool -i`), takže USB 5 GbE nebo 10 GbE adaptér je omezen tímto limitem, nikoli svou jmenovitou rychlostí 500 / ~1060 MB/s. Hodnota pro x86_64 byla naměřena v systému Linux: stejný adaptér, který ve Windows poškodí snímky při rychlosti nad 200 MB/s, zvládl bezchybně zpracovat pole čtyř kamer při každé maximální rychlosti až do 480 MB/s. Na platformě NVIDIA Jetson fungoval stejný adaptér bezchybně pouze při 200 MB/s a při vyšších rychlostech docházelo k přerušovanému ztrácení rámců. Pokud váš hostitelský počítač zpracovává více rámců bez `gvsp_corrupt`, zvýšte tuto hodnotu pomocí `wire_ceiling_mbps`. V systému Linux je adaptéru připojenému k portu USB 2.0 přiděleno 30 MB/s; připojte jej k portu USB 3.

#### Překročení kapacity (minimální rychlost na kameru)

Pole funguje v jakékoli síti; záleží na snímkové frekvenci. Sim-emit pacing přiděluje každé kameře rovný podíl z kolizně bezpečného přenosového limitu a všechny se stále spouštějí na stejný signál, takže na pomalém připojení je tento podíl — a snímková frekvence — malý: 8 plnohodnotnýchrozlišení 12bitových kamer na síti 1 GbE bez jumbo rámců dostane každá 6,5 MB/s, což je asi **1 fps**(asi 1,3 fps v komprimované podobě). Podíl je omezen na nejnižší rychlost, na kterou lze kamery nastavit (`per_cam_floor_bps`: při připojení se načte rozsah GevSCPD každé kamery a nikdy neklesne pod 1 MB/s). Pokud síť vyžaduje nižší rychlost, připojení přejde na**sériový**výstup (`slip-emit-and-capture`): kamery se střídají při plné rychlosti linky, takže každý rámec dorazí, ale členy**nejsou vystaveny současně**— úroveň odezvy a blokátory (`SERIAL: ...`) to potvrzují; předáním `force_tier="sim-capture-sim-emit"` lze tento ústup odmítnout. Velikost snímku nehraje při sdílení roli, takže**binning a ROI zvyšují snímkovou frekvenci, ale nemění to, zda se pole vejde**. Pouze linka, která nedokáže přenášet pole při jakémkoli tempu, ať už současně nebo střídavě,**překračuje kapacitu** — příčinou selhání je ztráta paketů GVSP, nikoli nižší snímková frekvence. Řešení: méně kamer, jumbo rámce v celém řetězci nebo rychlejší síťová karta.

- Odpovědi `analyze_array_network()` a `/api/camera/array/connect` obsahují `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` a `per_cam_floor_bps`. Pokud je hodnota `oversubscribed` true, projekce **nastaví pole fps na nulu** (`achievable_fps_max` / `fps_bright` / `fps_dark`), místo aby vykazovala zavádějící hodnotu pomalé, ale funkční rychlosti.
- `POST /api/camera/array/connect` přijímá parametr těla `pin_resolution` (**pouze HTTP — nejedná se o klíčový argument SDK**; `connect_array` jej nezpřístupňuje). Jedná se o přísný režim: připojení s nadměrným počtem připojení a nastaveným parametrem `pin_resolution` je**kategoricky odmítnuto** s chybou uvádějící všechny možné nápravné opatření. Bez něj se připojení uskuteční s varováním, že pole ztratí pakety. V obou případech připojení zachová požadovanou velikost rámce — nikdy ji samo od sebe neslučuje ani neomezuje.
- Pouze pro testování: nastavte v prostředí backendu parametr `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1`, aby se odmítnutí změnilo na hlasité varování – připojíte se i tak a akceptujete ztrátu paketů.

#### Stav pole — který subsystém ztrácí rámce

`ArraySession.capability()` (`GET /api/camera/array/<array_id>/capability`) přenáší živý blok `health` na
připojeném poli, který je průběžně vyhodnocován v **10sekundovém** okno. Rozděluje ztrátu rámců
na dvě příčiny, které vyžadují opačná řešení, namísto jedné míry „neúplnosti“, která
žádnou z nich neoznačuje:

| Pole | Co to znamená | Který subsystém |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (podle sériového čísla) | Snímek **dorazil, ale byl strukturálně poškozen**— ztráta paketů GVSP. |**Síť**: kapacita linky, tempo, kruh přijímače NIC, MTU |
| `never_arrived_rate_pct` (podle sériového čísla) | Rámec **vůbec nedorazil**— kamera se nespustila, nebo z ní nic nevyšlo. |**Spoušť / synchronizace**: kabel M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Nejhorší rychlost kamery pro každou z nich. | — |
| `per_cam_rate_pct` | Kombinovaná míra neúplnosti na kameru (obě příčiny dohromady). | — |
| `stable_for_seconds` | Jak dlouho každá kamera zůstala pod 0,01 %. | — |

Vedle položky `health` uvádí stejný záznam také číslo, na kterém visí celá alokace:

| Pole | Co to znamená |
| --- | --- |
| `wire_ceiling_mbps` | Aktuální trvalý rozpočet šířky pásma hostitele, MB/s. |
| `wire_ceiling_source` | Odkud tato hodnota pochází, slovy — např. `USB-capped 200 MB/s (was theoretical 1062; …)` nebo `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, pokud jej nastavil `wire_ceiling_mbps=`. |
| `nic_is_usb` | `true` pro USB ethernetový adaptér. |

```python
import chloros_sdk

serials = ["213800234", "214000533"]           # master first
arr = chloros_sdk.attach_array(serials)
h = arr.capability()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    # force_disconnect(), not disconnect(): this handle came from attach_array,
    # and disconnect() never tears down an array the session only attached to.
    # It returns only once the backend confirms the release; a teardown that
    # timed out or was refused raises ChlorosConnectError instead, so the
    # reconnect below never races a half-released array.
    arr.force_disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Výklad:** nenulová hodnota `gvsp_corrupt_rate_pct` při hodnotě 0 u `never_arrived_rate_pct` znamená,
že spouštění a synchronizace kabelu jsou v pořádku a 100 % ztráty je na síťové trase — snižte
`wire_ceiling_mbps` a znovu připojte. Opačný vzorec naopak ukazuje na synchronizační kabel nebo
spouštěcí linku.

> **`target_fps` není rozhodujícím faktorem pro poškozené rámce.** Pacing GevSCPD se zapíše jednou při
> připojení, takže snížení spouštěcí frekvence mění pracovní cyklus, nikoli
> rychlost souběžného vysílání burstů. Naměřené 5× snížení požadavků nepřineslo žádné zlepšení, zatímco
> snížení maximální přenosové rychlosti z 240 na 200 MB/s snížilo u stejného zařízení podíl poškozených rámců z 10,4 % na
> 0,00 %.

> **Běžící pole se nemůže samo přeplánovat na firmwaru LATTICE (M3C / M3M).** Nemůže
> tento problém vyřešit sama; odpojte a znovu připojte zařízení, aby nástroj pro výběr času připojení provedl nové plánování na základě
> nové horní hranice.

**USB ethernetový adaptér je sondou omezen na 200 MB/s** bez ohledu na jeho
označení: tabulka účinnosti, která převádí rychlost spojení na trvalou hodnotu, je
odvozena od PCIe, a USB síťová karta inzeruje svou ethernetovou rychlost spojení, přičemž je omezena
USB sběrnicí a jejím ovladačem. Toto omezení je absolutní, nikoli relativní — USB adaptér 1 GbE
dosahuje ~80 MB/s a není tímto omezením ovlivněn. Platí to pro Windows i Linux (včetně Jetsonu);
viz [Očekávané snímkové frekvence](#expected-frame-rates) pro informace o tom, kde bylo toto omezení naměřeno.

#### Metody `ArraySession`

| Metoda | Popis |
| --- | --- |
| `status(timeout=10.0)` | Aktuální stav spuštěného pole: `target_fps` / `target_fps_auto` (frekvence spouštění a zda ji řídí regulátor frekvence), `target_fps_clamped_reason` (důvod, proč je pevně nastavená frekvence udržována na nižší hodnotě, jinak `None`), `actual_fps` (naměřeno; 0, jakmile smyčka uchopení vzdala nebo se zasekla), `recommended_fps`, `streaming` (False, jakmile smyčka grabu vzdá snahu — zkontrolujte toto, ne jen `last_error`; zůstává True i po restartu nastavení, což signalizuje `restarting`), `stalled` (True, dokud smyčka grab běží, ale nezveřejnila žádnou skupinu snímků po dobu přibližně deseti snímkových period, 2–30 s; `last_error` to signalizuje až po 60 s), `last_frame_age_s`, `last_error`, `serials`, `ptp_enabled` (`True` pouze tehdy, když bylo pole připojeno k PTP **a** každý člen naposledy četl `PtpEnable` v zapnutém stavu; není to totéž jako uzamčený stav, který měří `ptp_epoch`), `ptp_configured` (to, co požadovalo připojení), `ptp_off_members` / `ptp_unread_members` (členové, kteří naposledy četli `PtpEnable` v režimu off / neodpověděli), `member_recovery` (`None`, nebo člen, který ztratil napájení nebo spojení: `state`, `waiting`, i když neodpovídá, `reopening`, následně `recovered`, spolu s `members` a `why` — pole jej znovu připojí, jakmile znovu odpoví, a vrátí mu jeho PTP z doby připojení, spouště a zapojení synchronizační linky, nastavení frekvence a kamery, poté znovu vysílá bez nových `connect_array`) a `ptp_epoch` (naměřeno), `sync_mode` / `sync_tier`, `packed_transport` / `packed_decision`, `per_cam_emit_mbps`, `link_speed_mbps` (`4294` znamená, že systém Windows nemohl přečíst rychlost adaptéru — viz [Očekávané snímkové frekvence](#expected-frame-rates)), `trigger_pipeline` / `pipeline_tune`, `radiometric_chain` (zda je řetězec radiance pro jednotlivé pásma použitelný a který člen jej narušil) a `band_clamp` — zatímco živý náhled vykresluje pole: podíl osvětlených pixelů, které odstraňovač smíšení omezil na 0, pro každý prvek (`members`: `{serial: [band0, band1, band2]}` v pořadí pásem R, G, B kamery) a pro každý pásmo, které kompozit zobrazuje (`composite`: `{band name: share}`), přičemž `warn` udává podíl, který grafické rozhraní označuje jako „data pásma s nízkou spolehlivostí“ (měřeno přibližně jednou za sekundu; `None`, pokud není spuštěn živý náhled) a `member_ae` — pro každý prvek expozice (`exposure_us`) a zesílení (`gain_db`) posledního zveřejněného snímku, získané přímo ze snímku (`read_nodes` z `ExposureTime` / `Gain` vrací poslední hodnotu zapsanou během běhu automatické expozice, nikoli aktuální stav). |
| `set_member_settings(serial, **kwargs)` | Zapíše nastavení jedné kamery ze skupiny, jak je `CameraSession.set_settings()` přijímá (expozice, zesílení, cílový jas, ...); člen zůstává v poli. Vyvolá výjimku `ChlorosConnectError` pro sériové číslo, které není členem. Viz [Expozice u členů pole](#exposure-on-array-members). |
| `capability(timeout=10.0)` | To, na čem se smart-prep dohodl (úroveň snímání, velikost snímku, `pixel_format`), živý blok `health` a rozpočet hostitele na přenos. Viz [Stav pole](#array-health--which-subsystem-is-losing-frames). |
| `set_target_fps(fps, timeout=10.0)` | Změna spouštěcí frekvence během provozu pole bez nutnosti opětovného připojení. Číslo ji fixuje (regulátor frekvence ji nechá beze změny); `None` jej vrátí do automatického režimu. Vrací `{requested, target_fps, target_fps_auto, clamped}` — `clamped` je True, pokud byla frekvence snížena na úroveň, kterou hostitel a síť dokážou přenášet. Viz [Intervalové snímání pro průzkumy pomocí dronů](#interval-capture-for-drone-surveys). |
| `grab(processing="debayered", with_metadata=False, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, bit_depth=None, band=None, align=False, refl_max=None)` | Jeden snímek z každé kamery, **všechny z jednoho spouštěče**, jako pole numpy (neukládají se žádné soubory). Vrací `GrabResult` (nebo `dict`): `{serial: array}`, nebo `{serial: (array, metadata)}` s `with_metadata=True`. `.group_id` je spouštěcí signál, který snímky sdílejí; `.errors` přiřazuje každou vynechanou kameru k důvodu (kamera, která nemůže být obsloužena při daném spouštěcím signálu, je vynechána a nikdy není nahrazena jinou). `processing` je `raw` / `debayered` / `radiance` / `reflectance` / `display`. Stejné čtení jako v projektovém API [`ArrayHandle.grab()`](#one-trigger-per-read--group_id) — jedna sdílená implementace — takže zde funguje každé klíčové slovo (`band`, `align`, `bit_depth`, `refl_max`) zde funguje. Vyžaduje `numpy`. |
| `frame_stream(*, processing="debayered", fps=5.0, duration=None, count=None, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, with_metadata=False, bit_depth=None, band=None, align=False, refl_max=None)` | Iterátor vrhající jeden výsledek `grab()` za tick (pouze klíčové argumenty), s tempem `fps`, omezeno hodnotou `duration` (v sekundách) a/nebo `count` (`None` = neomezeno). **Každá vrácená množina je novým spouštěčem:** tik, jehož nejnovějším spouštěčem je množina, kterou již vygeneroval, čeká na další spouštěč namísto opakování, takže proud běží pomaleji než `fps` nebo vlastní snímková frekvence pole. Vyvolá `FrameFetchError` namísto toho, aby ztichl: okamžitě, když každá kamera z nějakého důvodu odmítne, že by se další tick opakoval, po 60 s ticků bez snímku a po 60 s bez nového spouštěče („žádný nový spouštěč po dobu 60 s“). Stejná implementace jako `ArrayHandle.frame_stream()`. Viz [Živé snímky ve vašem vlastním zpracování](#live-frames-in-your-own-pipeline). |
| `continuous(output_dir="output", *, format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, max_captures=None, duration_s=None, daq=None, daq_tolerance_ms=None, timeout=30.0)` | Spustí vlastní nepřetržité snímání backendu (režim Continuous v grafickém rozhraní) → [`ContinuousHandle`](#continuoushandle). Ukládá **každou** synchronizovanou skupinu, kterou pole publikuje, stejně jako by ji uložil `capture()` (stejná ovládací prvky exportu), dokud není zastaveno nebo dokud není spuštěno `max_captures` / `duration_s` (`None` = dokud nebude zastaven). Vyvolá výjimku `ChlorosConnectError`, pokud pro toto pole již běží nějaký proces. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, smart=False, after_seq=None, daq=None, daq_tolerance_ms=None, timeout=300.0)` | Jedna synchronizovaná skupina snímků. Vrátí `CaptureResult` (seznam slovníků snímků + `.skipped` + `.failed` + `.group_seq` + `.daq_files` + `.daq_matches` + `.alignment` + `.oriented`). Předá předchozí výsledek`group_seq` jako `after_seq`, abyste získali další synchronizovanou skupinu po ní, místo čekání na novou. Níže jsou uvedeny ovládací prvky exportu. |
| `capture(..., smart=True)` | **Inteligentní snímání** — čeká, až se AE ustálí na všech kamerách, a poté spustí snímání. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, orient=None, timeout=120.0)` | Nejrychlejší snímání: pouze surová data + přiřazená hodnota DAQ (+ volný kombinovaný index). Odpovídá tlačítku „Nejrychlejší snímání“ v grafickém uživatelském rozhraní. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Jednorázové / nepřetržité / intervalové snímání v jedné ohraničené smyčce. Vrací `list[CaptureResult]`.**Vyžaduje `count` a/nebo `duration_s`**, aby se proces ukončil (SDK nemá Ctrl+C). Také se předčasně ukončí po dokončení průchodu, když průchod nahlásí plný disk (záznam `.failed` s `disk_full`); tento průchod je posledním výsledkem. |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, expr=None, timeout=30.0)` | Spustí nahrávání živého zobrazení kombinovaného indexu do videa/GIF → `RecorderHandle`. Video je ve formátu Motion-JPEG `.avi` (ffmpeg není potřeba). Pokud není otevřeno žádné grafické rozhraní, backend jej vykreslí: `expr` (indexový výraz v názvech pásem pole, např. `"(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)"`), nebo ve výchozím nastavení NDVI pole. Jeden kompozitní záznamník na jedno pole. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, orient=None, timeout=30.0)` | Spuštění sériového snímání v formátu raw-Bayer s vysokou snímkovou frekvencí → `RecorderHandle`. Offline zpracování pomocí `build_video()`. `orient=True`: výstupy generované `build_video()` se otáčejí podle orientace kamery (výstupy jednotlivých kamer podle orientace dané kamery, kombinovaný index podle orientace hlavní kamery); snímky ve formátu RAW zůstávají v původní orientaci a manifest snímkové série zaznamenává orientaci každého snímku. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Offline přepracujte uloženou sérii surových snímků na kalibrované video(a). Blokuje se až do dokončení (`wait=True`) a vrátí výsledek sestavení: `{success, outputs, errors, skipped, substitutions, notes, degraded, combined}` — viz [výsledek sestavení](#the-build_video-result) níže. |
| `build_video_status(job_id, timeout=15.0)` | Zjišťuje stav offline úlohy sestavení: `{running, result, error, burst_dir}`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Uvolní celé pole — **pokud jej tato relace otevřela** (`connect_array`): pak se jedná o `force_disconnect()` a vyvolá výjimku `ChlorosConnectError`, pokud uvolnění není potvrzeno (ukončení bloku `with` nikdy nevyvolá výjimku). Při použití handle z `attach_array` ponechá pole v provozu a vypíše na stderr jeden řádek s touto informací: pole může patřit grafickému uživatelskému rozhraní (GUI) nebo jinému skriptu. Každá kamera je ponechána v volném běhu a jinak v takovém stavu, v jakém ji připojení našlo; `keep_settings=True` uloží její nastavení do vlastní paměti kamery, `clear_settings=True` resetuje tuto paměť na tovární nastavení (viz [Paměť kamery](#camera-memory--keep_settings--clear_settings)). |
| `force_disconnect(timeout=60.0, keep_settings=False, clear_settings=False)` | Uvolní celé pole bez ohledu na to, kdo jej otevřel. Použijte jej na připojeném handle, pokud máte na mysli ukončení pole (např. za účelem opětovného připojení s odlišným nastavením). Vrátí se, jakmile backend potvrdí uvolnění, nebo pokud není co uvolnit (pole již neexistuje nebo neběží žádný backend); odpojení, u kterého došlo k vypršení časového limitu nebo které backend odmítl, vyvolá výjimku `ChlorosConnectError`. S libovolným příznakem vrátí to, co získala paměť každé kamery (`camera_memory`), jinak `None`. |

`capture()` ovládací prvky exportu (stejný koncový bod, jaký používá GUI/CLI):

- `processing` / `levels` — `processing="all"` (nebo `levels=["raw","radiance",…]`) uloží každý příslušný typ exportu pro každou kameru; jediná hodnota `processing` ukládá pouze tuto úroveň.
- `aligned` — **ve výchozím nastavení zapnuto (`None`), protože snímání stejného pole aplikací Chloros**(její nastavení zachycení**Aligned** pro dané pole). Ne-surový export každého člena je transformován do snímku hlavní kamery pomocí [profilu zarovnání](#array-alignment) a oříznut na společný překryv členů, takže soubory jsou pixelově zarovnány a mají stejnou velikost, ořez a značky `Alignment*` jako soubory aplikace (v rámci ROI se transformace přepočítává na okno); `XMP-mapir:AlignmentJSON` uvádí `"applied": true` s jeho `crop_roi`. Soubory ve formátu Raw zůstávají v původní velikosti bez deformace, ale v metadatech nesou transformaci. `aligned=False` ukládá každý soubor v původní velikosti, bez deformace; `aligned=True` je výchozí nastavení. `continuous()`, `capture(smart=True)`, `capture_repeated()` a na úrovni projektu `ArrayHandle.capture()` / `capture_smart()` používají stejné výchozí nastavení. Ať už je deformace použita či nikoli, každý členský soubor zaznamenává transformaci `Alignment*` své kamery, jakmile má pole profil zarovnání — s výjimkou kamery, kterou profil nezarovná (byla vynechána nebo byla v důsledku neúspěšného řešení vynucena do identické transformace) a referenční kamera, pokud není zarovnána žádná jiná kamera: ty neobsahují žádné značky `Alignment*`. Pokud pole nemá žádný profil, je záznam uložen bez zarovnání a výsledek to uvádí: `result.alignment` je `{"requested": True, "applied": False, "warnings": [...]}`, jedno varování za každý důvod, proč nebylo možné warp použít (chybějící profil, členy s odlišným rozlišením, pole, které nelze, snímek, na který nelze profil namapovat). `applied` je `True` pouze tehdy, když každý export, který požadoval warp, jej také obdržel. Automatické zarovnání pole vytvoří profil, jakmile se ustálí expozice; `ArrayHandle.calibrate_alignment()` / `align-calibrate` v CLI jej vytvoří na požádání. `result.oriented` udává, zda `orient=True` otočil produkty.
- `render_index=False` — přeskočit překryv indexu vegetace pro jednotlivé kamery; ve výchozím nastavení se vykresluje tam, kde je nakonfigurován.
- `daq` / `daq_tolerance_ms` — hodnota z DAQ se ve výchozím nastavení ukládá jako sidecar `.daq`, i když to žádná zvolená úroveň nevyžaduje; `daq=False` to v takovém případě přeskočí, a `daq_tolerance_ms` nastavuje toleranci shody (výchozí hodnota 500). `force_daq=True` je starší zápis výchozí hodnoty. Viz [Hodnoty DAQ, které zachycení ukládá](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
- `orient=True` — převede **produkty**každého člena (`display`, `radiance`, `reflectance`, index pro jednotlivé kamery a kompozit) podle orientace kamery po deformaci při zarovnání: podle orientace**hlavní kamery**v okamžiku, kdy byly transformovány do jejího rámce (výchozí nastavení `aligned` — členy zůstávají zarovnané), jinak podle orientace jejich vlastních kamer.**Výchozí nastavení: každý soubor prvku v nativním formátu**, jak je tomu vždy u snímků pořízených v poli; soubory `raw` a `debayered` zůstávají v nativním formátu v obou případech. Každý soubor zaznamenává orientaci snímku, ve kterém se jeho pixely nacházejí (`XMP-Camera:ChlorosRotation` / `ChlorosFlip*` / `ChlorosOrientationApplied` — soubor RAW, orientace vlastní kamery), a soubor XMP `Alignment*` zachovává transformaci nativního snímku, aplikovanou před orientací; soubor RAW obsahující tuto transformaci také zaznamenává `ChlorosAligned*` (orientaci referenčního snímku), tedy orientaci výsledného snímku transformovaného do orientace referenčního snímku. Zpracování otočí každý výsledný snímek o hodnotu, která odpovídá jeho pixelům: o hodnotu zarovnání, pokud se provádí transformace souboru RAW (zapnuté zarovnání pole), nebo o hodnotu vlastní fotoaparátu, pokud neprovádí zarovnání. `capture_repeated()` a `capture_fastest()` jej propouštějí; na úrovni projektu jej přebírají také `ArrayHandle.capture()` / `capture_smart()`.

**Komprese TIFF (ovládací prvek HTTP):** `ArraySession.capture()` neodesílá žádný klíč `compression`, proto platí výchozí nastavení backendu — `POST /api/camera/array/capture` čte parametr těla `compression`, ve výchozím nastavení: nekomprimované, pixely snímače přesně tak, jak byly načteny (~6,3 MB na snímek v plném rozlišení). `"deflate"` (bezztrátová komprese zlib L1 + horizontální prediktor, ~4,1 MB/snímek) zapisuje menší soubory pomaleji — obě komprese jsou bezztrátové a při importu se čtou identicky. SDK pro to neposkytuje žádný argument; únikovou cestou je `chloros-cli lattice array-capture --compression deflate` nebo surové HTTP. DEFLATE blokuje Python GIL, takže komprimované zápisy nelze paralelizovat napříč vlákny zapisovače pro jednotlivé kamery — pro trvalé snímání v plném rozlišení při rychlosti snímače je nutné použít nekomprimované výchozí nastavení. Podrobnosti: [Referenční příručka CLI → array-capture](cli-reference.md).**Po sobě jdoucí skupiny (protokol HTTP):** koncový bod vrací `group_seq`; smyčka, která jej při dalším požadavku odešle zpět jako `after_seq`, získá další synchronizovanou skupinu — včetně té, která byla publikována během ukládání předchozího snímku — namísto čekání na skupinu publikovanou až po odeslání požadavku, a `skipped_groups` hlásí všechny skupiny, které smyčka vynechala. `capture_repeated`, `--continuous` v CLI i režim Continuous v GUI toto provádějí; intervalové průchody zůstávají aktuální v každém naplánovaném okamžiku.**Přepsání exportu pro jednotlivé členy (pouze HTTP):**stejný koncový bod také přijímá `exclude_serials` (seznam — vyřazení členů z uložené sady; pole se stále spouští jako jedna synchronizovaná skupina a vyloučení členové jsou vráceni v `excluded`), `serial_levels` (přepsání na úrovni jednotlivých kamer `{serial: [level tokens]}`) a `serial_index` (přepsání překrytí indexu jednotlivých kamer `{serial: bool}`). Jedná se o parametry těla odpovídající grafickému uživatelskému rozhraní (GUI) a**zatím ne o klíčové argumenty SDK**; členy, které v mapách chybí, nahrazují parametry platné pro celé pole `levels` / `render_index`.

##### Kontrola přeskočených kamer — `CaptureResult.skipped`

`ArraySession.capture()` vrací `CaptureResult`, což je podtřída `list`: iterujte přes ni, indexujte ji, `len()` — všechny stávající vzory nadále fungují. Nový kód může zkontrolovat atribut `.skipped`, aby zjistil, které kamery byly vyloučeny a proč. Nejčastějším případem jsou kamery RGB ve smíšeném filtrovém poli, když požadujete `processing="radiance"` nebo `"reflectance"` — zářivost na jeden Bayerův pixel nemá u širokopásmového snímače smysl, takže backend tyto kamery přeskočí, místo aby generoval nesmyslné údaje.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Tokeny důvodů mají formát `<level>-not-applicable-to-rgb-cam` (jeden záznam pro každou přeskočenou úroveň, přičemž každý obsahuje `level`). Kamera, u které není načten kalibrační balíček, přeskočí `radiance` / `reflectance` s `<level>-needs-calibration (connect the camera so its cal pack loads)`, a v případě, že není k dispozici použitelné měření odrazivosti z DAQ, přeskočí na `reflectance-skipped-no-dls`. Další přeskočení související s odrazivostí jsou `reflectance-skipped-no-fresh-dls` (není k dispozici žádné nové měření sestupujícího záření), `reflectance-skipped-bound-daq-unavailable (…)` (nelze dosáhnout na připojený DAQ) a `dls-uncalibrated-band-<nm>` — pásmo leží převážně mimo radiometricky kalibrovaný rozsah světelného senzoru DAQ (~374–974 nm), takže absolutní rozdělení podle odrazivosti založené na DAQ je odmítnuto a snímek je výrazně přeřazen na odezvu snímače. Z dodávaných modelů to spouští pouze F988; podporovanou cestou pro tuto kameru je pracovní postup s panelem odrazivosti.

##### Soubory, které nebylo možné zapsat — `CaptureResult.failed`

Záznam, u kterého se podařilo zapsat pouze některé soubory, stále vrací: seznam obsahuje všechny soubory, které **byly** zapsány, a `.failed` uvádí zbytek, jeden slovník na soubor s `serial` (`None` pro kombinovaný kompozit a sidecar pole `.daq`), `level` (`raw`, `radiance`, …, `index`, `composite`, `daq`, nebo `match` — původ souboru `.daq` v souboru `.match.json`, kdy byl soubor `.daq` zapsán, ale jeho sidecar nebyl) a `error`. Na zaplněném disku obsahuje záznam také `disk_full: True`; každý pozdější záznam na tento disk by selhal stejným způsobem, a proto `capture_repeated()` končí právě zde. Při částečném záznamu není vyvolána žádná výjimka a soubor v `.failed` nezůstal na disku napůl zapsaný. `.failed` je při čistém záznamu prázdný.

```python
result = arr.capture("output/", processing="all")
for f in result.failed:
    print(f"SN:{f['serial']} {f['level']} not written: {f['error']}")
if any(f.get("disk_full") for f in result.failed):
    raise SystemExit("drive full -- free space and capture again")
```

##### DAQ, který čte záznam, ukládá soubory — `.daq_files` a `.daq_matches`

Kdykoli je DAQ připojen ke kameře nebo poli, případně zařazen do sdíleného fondu, každý záznam ukládá naměřenou hodnotu dopadajícího záření, kterou použil: soubor `.daq` v podsložce `daq/` složky se záznamem, spolu se soubory `.match.json` a `.cameras.json` (jehož hodnota to je). To platí i pro snímky ve formátu raw nebo po odstranění bayesovského filtru a jedná se o stejný soubor bez ohledu na to, zda byl snímek pořízen z grafického uživatelského rozhraní (GUI), příkazového řádku (CLI) nebo sady SDK. `daq=False` tuto hodnotu vynechá u snímku, jehož úrovně nepoužívají úroveň DAQ (úroveň typu `radiance`, `reflectance` nebo `display` ji stále zapisuje); `daq_tolerance_ms` posouvá hranici tolerance pro daný zachycení a nikdy ne pro použité měření. Který DAQ: ten, který je vázán na kameru nebo pole (výběr světelného senzoru v grafickém rozhraní, vazba projektu nebo `chloros-cli daq pool-bind`), jinak ten nejnovější, který má backend k dispozici. CLI označuje ovládací prvky `--no-daq` a `--daq-tolerance-ms` ([CLI Reference](cli-reference.md#the-daq-reading-a-capture-saves)).

```python
r = cam.capture("out", levels=["raw"])      # a CameraSession; ArraySession results match
for m in r.daq_matches:                      # one per .daq in r.daq_files
    print(m["daq_file"], m["gap_ms"], m["in_tolerance"])
```

Zachycení uloží hodnotu ze světelného senzoru, kterou použilo, jako `.daq` spolu s hodnotou `<file>.daq.match.json`, která udává, jak byla tato hodnota svázána s expozicí: `method`, `gap_ms` (od expozice k nejbližší hodnotě DAQ) a `in_tolerance` (v rozmezí 500 ms, resp. `daq_tolerance_ms` snímku). XMP soubor souboru odrazivosti obsahuje stejný údaj jako `Chloros:DlsMatchMethod`, `Chloros:DlsMatchGapS` a `Chloros:DlsInTolerance`.

Záznam čeká na první odečet z DAQ po expozici, aby mohl sloučit odečty na obou stranách tohoto bodu (`method` `interpolated`). DAQ-E odesílá naměřenou hodnotu každých 0,4 s za denního světla a každých 1–2 s v interiéru, takže `capture()` se vrátí až asi o jeden a půl takového intervalu později; samotná expozice není zpožděna. Na DAQ, který přestal odesílat data, se nečeká. Hodnota DAQ-E představuje průměrné osvětlení za interval od předchozího měření, takže je umístěna uprostřed tohoto intervalu: `gap_ms` se měří k tomuto středu a `.match.json`'`bracket_stamps_ns` poskytuje vlastní časová razítka měření. `ArraySession.continuous()` čeká, aniž by zpomalil snímání. Zpátečnínečekají, takže si zachovávají svou frekvenci a jsou přiřazeny k nejbližšímu měření před expozicí.

Měření nikdy nevyprší, takže nedojde ke ztrátě žádného snímku kvůli opožděnému měření — místo toho je označeno:

- **Proud z DAQ se zastavil** — použije se jeho nejnovější hodnota a `gap_ms` udává, jak stará je (`in_tolerance: false` více než 500 ms).
- **Žádný DAQ k načtení**— odpojen (`disconnect()`, `chloros-cli daq pool-disconnect`) nebo není možné se připojit k napojenému DAQ — použije se poslední hodnota z kamery a přiřadí se k**této** expozici: `method`, `retained`, `gap_ms` z této expozice, `in_tolerance` se přepočítá a `retained_from_epoch_ns` uvádí expozici, ke které byl původně přiřazen. Kamera, která nikdy neměla naměřenou hodnotu, stále nemá žádnou odrazivost a výsledek uvádí důvod (`reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, případně odmítnutí snímku).

Soubory, které snímek uloží do složky projektu otevřeného v aplikaci Chloros, se připojí k seznamu souborů daného projektu, stejně jako je tomu u snímků pořízených přímo v aplikaci (soubory uložené jinde zůstávají beze změny): `CameraSession.capture()`, `ArraySession.capture()` (stejně tak `smart=True`) a `capture_cameras_transient()`. Výsledek to uvádí v souboru `.project_add` (`capture_cameras_transient()`: slovník `project_add`) -- `held`, `files` a `message` v backendu; `held` je `True`, dokud probíhá zpracování, a soubory se připojí, jakmile skončí. `None`, pokud do projektu nebyl přidán žádný soubor.

`CameraSession.capture()` a `ArraySession.capture()` je vrací jako výsledné soubory `.daq_files` a `.daq_matches`, jako odpovědi zachycené protokolem HTTP (`/api/camera/<serial>/cli-capture`, `/api/camera/capture`, `/api/camera/capture-all-synced`, `/api/camera/array/capture`) obsahují `daq_files` a `daq_matches`, po jednom na každý `.daq`: `daq_file`, `cameras`, `method`, `gap_ms`, `in_tolerance`, `tolerance_ms` a `stream_fault` — vlastní chyba datového akvizice (DAQ), vyhledávaná pouze v případě, že naměřená hodnota překračuje toleranci. Než se spolehnete na odrazivost ze záznamu, zkontrolujte `in_tolerance` v jeho `.match.json` nebo XMP, případně v [`status()`](#daqsensorsession-methods).

Úrovně `processing`:

| Úroveň | Výstup |
| --- | --- |
| `"raw"` | Jednokanálový Bayer (mono kamery: jediný pásmo) přímo ze snímače. |
| `"debayered"` *(výchozí nastavení SDK)* | 3kanálová barva: odečtení tmavého posunu (DSNU) z kalibračního balíčku z surových dat, následně demosaikace s ohledem na hrany pomocí OpenCV — bez LUT linearity, vyrovnání pole ani oddělení složek (černobílé kamery: 1kanálová šedá stupnice). Uložený soubor TIFF je v pořadí kanálů **RGB**(fotometrické RGB, takže jakýkoli prohlížeč obrázků zobrazí skutečné barvy); pole v paměti, která vrací `grab()` / `frame_stream()`, jsou v pořadí**BGR** podle OpenCV — pro porovnání s uloženým souborem obraťte kanály (`frame[..., ::-1]`). `chloros-cli process` zapíše stejný výsledek jako `Debayered_Images`. |
| `"radiance"` | float32 W/m²/sr/nm prostřednictvím úplného radiometrického řetězce. Pouze multispektrální — kamery RGB jsou vynechány. |
| `"reflectance"` | uint16 0..32768 (kompatibilní s Pix4D); vyžaduje spárování s živým DAQ pro absolutní referenci. Pouze multispektrální. |
| `"display"` | Plný řetězec odpovídající náhledu v grafickém rozhraní (CCM + WB + gama podle profilu kamery). |
| `"all"` | **Jeden soubor pro každou příslušnou úroveň** pro každou kameru (odpovídá výchozímu nastavení „Capture All“ v grafickém rozhraní / CLI). Vrácený soubor `CaptureResult` pak obsahuje jeden slovník snímku na každý soubor `(cam, level)`, přičemž úroveň je uvedena v každém slovníku; neplatné úrovně se objevují v `.skipped`. DAQ, který načítá použitý záznam, je uložen jako sidecar `.daq`. |

> **Poznámka — výchozí nastavení se liší od CLI.** `ArraySession.capture()` má výchozí hodnotu `processing="debayered"`; příkaz `chloros-cli lattice array-capture` má výchozí hodnotu `processing="all"`. Explicitně zadejte `processing="all"` z SDK, abyste napodobili víceúrovňové ukládání v CLI/GUI.

##### Názvy souborů zachycených dat

Každá úroveň se ukládá do vlastní podsložky adresáře `output_dir` (`raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` pro `display`, `index/`, `composite/`). Každý soubor má v názvu označení kamery ve formátu `SN<serial>-<filter>`: sériové číslo, poté část názvu modelu kamery označující filtr — `SN214000533-FOCN` pro kameru `M3C-L87-FOCN`, `SN261902142-F450` pro model `M3M-L87-F450` — spojené s `-`, takže název má stejná pole oddělená `_` jako dříve. Kamera, jejíž model nelze přečíst, má pouze `SN<serial>`, a snímky z dřívějších verzí nemají v názvu žádný filtr. Název souboru závisí na tom, kolik úrovní bylo od člena požadováno:

| Požadovaný počet úrovní | Název souboru | Příklad (`reflectance`, výchozí `format="tiff"`) |
| --- | --- | --- |
| **Jedna úroveň** (`processing="reflectance"`, nebo `levels` s jednou položkou) | `sync_<timestamp>_SN<serial>-<filter>.<format>` — bez příponu úrovně, stejně jako v dřívějších verzích, takže stávající skripty nadále fungují | `reflectance/sync_<ts>_SN214000533-FOCN.tiff` |
| **Několik úrovní** (`processing="all"` nebo `levels` s více než jednou položkou) | `sync_<timestamp>_SN<serial>-<filter>_<level>.<format>` | `reflectance/sync_<ts>_SN214000533-FOCN_reflectance.tiff` |

Za názvem produktu následuje [maska omezovače](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on), `clamp_masks/<product folder>/<product name>_clampmask.tif`: záznam odrazivosti na jednéúrovňové zachycení odrazivosti zapíše `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`, zatímco `processing="all"` zapíše `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`. Pravidlo pro příponu se určuje pro každý prvek na základě úrovní, o které byl požádán, takže vynechaná úroveň (např. odrazivost u RGB kamery) jej nemění. Čtěte cesty z vrácených snímků, místo abyste je vytvářeli. Snímání jednou kamerou (`CameraSession.capture()`) vždy přidává úroveň: `capture_<ts>_SN<serial>-<filter>_<level>.tiff`.

##### Formáty PNG a JPEG: zpracovávají pouze zobrazené a indexované produkty

Formáty PNG nebo JPEG mají 8 bitů na kanál. Požadavek na formát s úrovní, kterou daný formát nepodporuje, bude zamítnut ještě před zachycením dat, a to s hlášením uvádějícím danou úroveň a formát TIFF, který ji podporuje:

- **`radiance`**(float32) a**`reflectance`**(uint16, `32768` = 1,0) — a tedy i `processing="all"`, který je zahrnuje. SDK to kontroluje na argumentech: `capture()` vyvolá**`ValueError`** bez požadavku (`CameraSession`, `ArraySession`, `CameraHandle`, `ArrayHandle`, `capture_smart`).
- **`raw` / `debayered` z kamery vysílající více než 8 bitů** (12bitový M3C, Mono12 M3M) — backend je odmítá, protože zná bitovou hloubku každé kamery: HTTP 400 s kódy `error_code` a `FORMAT_CANNOT_HOLD_LEVEL`, vygenerované jako `ChlorosConnectError` (`CameraSession` / `ArraySession`) nebo `RuntimeError` (`CameraHandle` / `ArrayHandle`). Jednoúrovňový záznam `debayered` z jedné kamery zachycuje 8bitový snímek pro formáty PNG a JPEG a není odmítnut.

Radiometrická data ukládejte jako `format="tiff"` (`ext=".tiff"`); pro `display` a indexové rendery použijte formáty PNG/JPEG.

#### Jak funguje automatická expozice

- **Nejprve expozice, až poté zisk.** Je-li zisk nastaven na Auto, zůstává na hodnotě 0 dB, dokud expozice nedosáhne nejdelší povolené hodnoty.
- **Při ručním nastavení expozice zůstává zesílení v režimu Auto**, dokud nezměníte zesílení nebo jej nenastavíte na ruční režim. Po návratu do režimu Auto je limit zesílení stejný jako předtím.
- **Malé světlé oblasti scénu neztmavují.** Odlesky do 1 % snímku jsou ignorovány a u panelů známého kalibračního terče až do 0,5 % panelu.
- **Horní limit expozice DAQ platí pouze tehdy, když je v zorném poli kalibrační terč.** Zabraňuje to oříznutí bílého panelu; pokud není žádný panel v záběru, kamera exponuje podle scény.
- **Při konstantním osvětlení zůstává expozice stabilní a po změně osvětlení se obraz ustálí během několika sekund.** Při konstantním osvětlení zůstávají expozice a její limit nezměněny a po výrazné změně osvětlení se obraz ustálí během několika sekund. Přesvětlení již trvale neomezuje expozici a při změně oblasti zájmu (ROI) se expozice znovu nastaví.
- **Cílová hodnota jasu zůstává v dosahu.** Nezvyšuje se, když je kamera na svém expozičním limitu a její zesílení nemůže stoupat, takže se při návratu světla v záběru nevytvoří odlesk.
- **Kamera, jejíž automatická expozice přestane reagovat, je udržována na úrovni odpovídající aktuálnímu osvětlení.** Za jasného slunečního svitu by kamera mohla klesnout na nejkratší expozici a zůstat tam s tmavým snímkem, přičemž všechna její nastavení by vykazovala normální hodnoty. Chloros sleduje snímek, udržuje kameru na úrovni, kterou si zachovala pro dané osvětlení, a uvolní ji, jakmile se osvětlení změní. Temný nebo zakrytý světelný senzor tento proces nenarušuje.

#### Expozice u členů pole

Kamera LATTICE omezuje hodnotu `ExposureTime` na jednu periodu své snímkové frekvence. Zatímco pole vysílá data, Chloros spouští každý člen s aktivovanou hodnotou `AcquisitionFrameRateEnable`, a to rychlostí těsně nad spouštěcí frekvencí, takže horní hranice expozice člena leží o něco pod spouštěcím obdobím, **v jakékoli oblasti zájmu (ROI)**:

- Obrazová perióda prvku je perióda spouštění minus rezerva 15 % této periódy (minimálně 3 ms, maximálně polovina periódy) a jeho horní limit expozice je tato obrazová perióda minus přibližně 0,1 ms. Příklad: spouštěcí frekvence 10 fps (100 ms) dává frekvenci člena přibližně 11,8 fps a horní hranici expozice přibližně 85 ms.
- **Ruční nastavení `ExposureTime` delší než tento strop zpomalí spouštění, aby se mu přizpůsobilo.** Chloros nejprve otevře strop daného člena na tuto hodnotu a nechá pomalejší frekvenci běžet několik spouštěcích period (zápis může trvat o několik sekund déle), poté ji zapíše, a spoušť se spouští maximálně tak rychle, jak to expozice dovoluje. U pole s automatickou frekvencí se původní frekvence obnoví, jakmile se každý člen, jehož manuální expozice spoušť zpomalila, vrátí k automatické expozici. Hodnota, kterou kamera sama nemůže přijmout, je stále odmítnuta, přičemž rozsah, který nyní přijímá, je uveden v chybové zprávě, a automatická expozice člena je obnovena, pokud byla vypnuta pouze pro tento zápis. Před verzí 1.3.0 omezovala malá oblast zájmu (ROI) každého člena na vlastní volnou periodu snímače, bez ohledu na frekvenci spouštění: 4,9 ms při rozlišení 768 × 304, takže žádný člen nemohl být zesvětlen.
- Chloros nikdy nezkracuje manuální expozici, aby se přizpůsobil rychlejšímu spouštění. Rychlost automatického režimu zůstává v mezích, které umožňují manuální expozice; pevně nastavená rychlost vyšší, než umožňuje expozice, ponechává danému členu dostatečně dlouhou dobu pro jeho expozici a přijímá spouštění, které není rychlejší, než umožňuje jeho expozice. Snižte expozici nebo pevně nastavenou rychlost, pokud musí každý člen stihnout každé spouštění.
- **Zpětné čtení:** při čtení uzlů prvku se zobrazí `AcquisitionFrameRateEnable` True a `AcquisitionFrameRate` v frekvenci prvku; povolení se vypne, jakmile pole zastaví.
- **Pole vlastní tyto dva uzly. Nezapisujte `AcquisitionFrameRateEnable` ani `AcquisitionFrameRate` na člen pole** — místo toho nastavte rychlost polepomocí `target_fps` / `set_target_fps()`.

Zapište expozici člena pomocí `arr.set_member_settings(serial, exposure_time=20000)` (klíčová slova `CameraSession.set_settings()`; `chloros-cli lattice exposure --serial` funguje stejně). Člen zůstane v poli. Neotevírejte člena pole pomocí `connect_camera()`: funkce connect odmítne kameru, která již je v poli, a funkce `CameraSession`, kterou by vrátila, odstraní kameru z pole při odpojení. Funkce `ArraySession.status()["member_ae"]` nahlásí expozici každého členaexpozici a zisk z jeho nejnovějšího snímku.

### Jeden záznam projektu, každý režim

Složka projektu obsahuje nastavení každého zařízení: `cameras.json` — pro každou kameru (`cameras[].settings`, pro jednu kameru i pro každý člen pole) a pro každé pole jako celek (`arrays[].capture`: formát, ROI, binning, frekvence spouštění, rozpočet na přenos dat) — a `sensors.json`, nastavení pro každý světelný senzor (integrační čas, průměrování snímků, limit). Aplikace, CLI, SDK i server MCP čtou a zapisují stejné soubory stejným způsobem:

- **Zařízení připojené v rámci projektu se do něj uloží a zaznamenává do něj každou změnu.** „V rámci projektu“ znamená s projektem otevřeným v backendu (aplikace) nebo — prostřednictvím souboru `ChlorosProject` (`open_project()`), který v backendu určuje vlastní složku (`project_path`). Kamera, pole nebo světelný senzor, které projekt dosud neobsahuje, se při připojení přidají do jeho souborů (`connect_camera()`, `connect_array()`, připojení světelného senzoru, CLI, aplikace, `chloros-mcp`), spolu s příslušnými nastaveními. Změna z jakéhokoli režimu (`set_settings()`, `ArraySession.set_member_settings()`, `set_target_fps()`, nastavení pole, `set_cap()`, CLI, ovládací prvky v aplikaci) se o chvíli později uloží do souborů daného projektu. Nastavená expozice, zisk nebo cíl AE na kameře se uloží jako zachované, takže při opětovném otevření projektu se tyto hodnoty znovu použijí.
- **Otevření projektu nastaví jeho zařízení podle něj**, a to v každém režimu: fotoaparát, pole nebo světelný senzor připojený bez vlastních nastavení převezmeuložená nastavení (`lattice cam-connect` / `array-connect` při otevřeném projektu, `chloros-cli project connect`, `open_project().connect_all()`, opětovné otevření aplikace). Pole připojené pod novým ID aktualizuje pole, které projekt uložil se stejnými kamerami; pole s jinými kamerami se přidá vedle něj.
- **Vlastní uložení aplikace nikdy nepřepíše aktuální nastavení zařízení** starší kopií a zachovává přepsání jednotlivých členů uložená volajícím z CLI nebo SDK.
- **Formát pixelů, který nastavíte, je ten, který si projekt uchová.** Formát nastavený v nastavení pole (Array Settings), pomocí `--pixel-format` nebo `connect_array(pixel_format=)`, včetně 8bitového, se uloží a při opětovném připojení se přesně zachová. Pokud není nastaveno nic, každá kamera se připojí ve svém výchozím 12bitovém nastavení (`BayerRG12`, `Mono12` u monochromatického snímače). Název formátu označuje jeho bitovou hloubku; rodina odpovídá snímači, takže barevná kamera není nikdy nastavena na monochromatický formát ani monochromatická kamera na formát Bayer, a řádek `[PixelFormat <serial>] ... written` to uvádí, pokud byl formát namapován. Výstupy z 8bitového snímání jsou 8bitové, včetně odrazivosti: přepočítané a označené svým `PixelScale`.
- Zařízení připojené bez projektu nezaznamenává .
- **Záznam se přiřadí k projektu vytvořenému po něm; otevření projektu ponechá záznam v původním projektu.**Při vytvoření projektu — `create_project()`, `chloros-mcp`, `create_project` nebo pomocí funkce**Nový projekt**v aplikaci — nebo duplikován (funkce**Duplikovat**v aplikaci, která otevře kopii), zatímco záznam stále zapisuje do projektu, který byl otevřený předtím, záznam pokračuje a to, co zapíše, skončí v novém projektu, na stejném místě uvnitř něj (níže). Otevření existujícího projektu — `ChlorosProject.process()` (což otevře jeho projekt na pozadí), `chloros-cli process`, možnost**Otevřít projekt** v aplikaci — ponechá nahrávání stále probíhající v rámci projektu, do kterého se nahrává: tam se dokončí a jeho soubory se připojí k seznamu souborů daného projektu (aplikace zastaví každé nahrávání před otevřením projektu).
  - `ArraySession.record()` a `ArraySession.burst()`: video nebo složka sériových snímků se všemi snímky se tam přesune jako celek, jakmile se nahrávání zastaví. Výsledek zastavení uvádí novou cestu (`out_dir`, `video_path`) a z ní se sestaví sériový záznam. Pokud disk přesun odmítne, zůstane záznam tam, kam byl zapsán, a výsledek to uvádí (`relocation_error`). Surové snímky zastavené série a soubor `.daq` se přidají do seznamu souborů projektu, do kterého se série zařadí; video se do seznam souborů projektu.
  - `ArraySession.continuous()`: každý záznam po změně se uloží do nového projektu; ty, které byly uloženy dříve, zůstanou tam, kde jsou.
  - `DAQSensorSession.record_start()` (a vlastní záznam aplikace z světelného senzoru): po ukončení nahrávání se `.daq` přesune do složky `light_sensor` nového projektu; `record_stop()` vrátí novou cestu a soubor se zařadí do seznamu souborů daného projektu.

  Záznam do složky mimo otevřený projekt zůstane na svém místě a úlohy typu `start_camera_continuous()` / multi-stream si zachovají složku, do které byly přiřazeny.
- **V aplikaci se záznamy po vytvoření nového projektu ukládají do něj.** Vytvoření nebo duplikování projektu zachová všechny probíhající snímací úlohy a nahrávání, a každý záznam pořízený poté (Capture All, Start Capturing) se uloží do nového projektu a přidá se do jeho seznamu souborů. Otevření existujícího projektu v aplikaci nejprve zastaví všechny tyto procesy.
- **Projekt se během nahrávání nepřesouvá.**Přesunutí otevřeného projektu do jiného pracovního adresáře (v aplikaci**Nastavení → Pracovní adresář**; žádné volání SDK projekt nepřesune, HTTP klient odesílající na `/api/move-project` ano) je odmítnuto s chybou HTTP 409 `PROJECT_RECORDING`, pokud do něj právě zapisuje záznam pole nebo sériový záznam, nepřetržité snímání pole, úloha nepřetržitého / vícestreamového snímání kamery nebo záznam světelného senzoru: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.`

Toto je odlišné od toho, v jakém stavu zůstávají kamery při odpojení (viz [Paměť kamery](#camera-memory--keep_settings--clear_settings)): projekt si při otevření nastavuje své kamery sám.

### Paměť kamery — `keep_settings` / `clear_settings`

Po připojení pole převezmou kamery nastavení pole a vy je poté můžete změnit. Když program Chloros uvolní kamery — `ArraySession.disconnect()` / `force_disconnect()`, na konci přechodové funkce, při zastavení `start_camera_continuous()` — vrátí každou z nich do stavu, v jakém byla nalezena před připojením: její zapojení spouště a synchronizační linky, nastavení PTP a kamery — `ExposureAuto` / `GainAuto` (a ruční `ExposureTime` / `Gain`), `TargetBrightness` a limity AE a tlumení, vyvážení bílé, úroveň černé, `PixelFormat`, binning a oblast zájmu (ROI), barevný proces (`GammaEnable` / `Gamma`, `LUTEnable`, `ColorTransformationEnable`), `ReverseX` / `ReverseY`, `UserSetSelector`, PTP: `PtpSlaveOnly` / `PtpDomainNumber`, velikost paketu (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`), tempo (`DeviceLinkThroughputLimitMode` / `DeviceLinkThroughputLimit`, `GevSCPD`, `GevSCFTD`), data bloku (`ChunkModeActive`, každý `ChunkEnable` pod svým `ChunkSelector`), odšumování spouštěcího vstupu a pole poskytující synchronizační linku podřízeného zařízení (`LineFilterSelector` / `LineFilterWidth`) a snímkovou frekvenci snímání (`AcquisitionFrameRate` pouze v případě, že bylo povolení zapnuto: při vypnutém povolení se snímková frekvence čte na maximální hodnotu, kterou tempo umožňuje, nikoli podle nastavení). Zapsáno je pouze to, co se liší: zda se kamera zastavila jako první, tempo a snímková frekvence před expozicí, která vyžaduje jejich snímkové období.

Pro použití kamer mimo projekt zapisují dva argumenty do **vlastní paměti** kamery — to, s čím se kamera spouští, v jakémkoli softwaru (na `disconnect()` / `force_disconnect()`, ve všech přechodných funkcích, `start_camera_continuous()` a `ChlorosProject.disconnect_all()`; MCP: stejné argumenty jako u `disconnect_device` a přechodných nástrojů; CLI: `--keep-settings` / `--clear-settings`):

- **`keep_settings=True`** uloží nastavení každé kamery tak, jak je zanechala relace, do její paměti (UserSet2, vytvořeno jako spouštěcí sada). Kamery jsou stále ponechány v režimu volného běhu (`TriggerMode` `Off`), zapojení synchronizační linky zůstává beze změny. Kamera bez UserSet2 ukládá do UserSet1; tento počítač si to pamatuje, takže samoléčba pole Chloros — která resetuje UserSet1, který se odchýlil od základní linie Chloros — ji nechá na pokoji.
- **`clear_settings=True`** resetuje paměť každé kamery na tovární nastavení: Chlorosovu základní hodnotu, kterou jeho samoopravný systém pole zafixuje — tovární výchozí nastavení pomocí `TriggerMode`, `Off`, `TriggerSource`, `Line2`, automatická expozice, kalibrovaná vyvážení bílé u RGB kamery zachována — v UserSet1, vytvořena spouštěcí sada.
- Ne obojí (`ValueError`; backend vrací odpověď 400). Odpověď obsahuje `camera_memory`, pro každou kameru `{"saved_to": "UserSet2" | "UserSet1" | None, "errors": [...]}` nebo `{"cleared": bool, "errors": [...]}` — `force_disconnect()` to vrací; pro úlohu je to `stop()`.

Aplikace tyto hodnoty nenabízí.

**Každý způsob ukončení vrátí kamery zpět.**Vracení do původního stavu se provádí při každé cestě ukončení: `CameraSession.disconnect()` (kamera**X**aplikace),**X** v poli nebo na libovolném členu pole (celé pole je uvolněno), při zavření aplikace (backend má až 20 s na vrácení všech kamer, polí a běžících nepřetržitých nahrávek; přechodná funkce nebo jednorázové volání, které stále běží, se zastaví v nejbližším bezpečném bodě, nejprve vrátí své kamery, a to po dobu až 8 s, a odpoví HTTP 503 `SHUTTING_DOWN`; stále běžící `connect_array()` se zastaví před svou další kamerou a vrátí kamery, které otevřela, a to po dobu až 10 s, a vygeneruje `ChlorosConnectError` (HTTP 503 `SHUTTING_DOWN`); a ten, který dorazí během vypínání, nic neotevře: HTTP 503 `SHUTTING_DOWN`)a příkaz `chloros-cli`, který je ukončen nebo jehož terminál se zavře (jeho úloha je zastavena a kamery jsou nejprve vráceny). Backend, který havaruje nebo je násilně ukončen, nemůže nic vrátit; restart vrátí každou kameru do její paměti.**Projekt si nastavuje kamery sám.** Když se pole připojí prostřednictvím projektu — otevřeného projektu aplikace nebo `ChlorosProject`, který se sám identifikuje backendu (`project_path`) — každý člen získá nastavení, která pro něj projekt uložil: expozici, zesílení nebo cíl automatické expozice (uložené jako vaše, takže automatická expozice je neovlivňuje) a režim vyvážení bílé (korekce Chloros; vlastní vyvážení bílé fotoaparátu zůstává vypnuté), ať už paměť fotoaparátu obsahuje cokoli.

### Režimy snímání a záznamníky

Povrch pole odráží panel snímání v grafickém uživatelském rozhraní: režimy Jednorázový / Sériový / Intervalový / Nejrychlejší závěrka, plus dva záznamníky (živé kompozitní video a sériové snímky ve formátu RAW → offline zpracování).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Continuous — every synced group for 60 s, saved by the backend itself
    with arr.continuous("run1/", processing="raw", duration_s=60) as run:
        run.wait()
    print(run.result["captures"], run.result["captures_per_s"],
          run.result["stop_reason"])

    # Combined-index video/GIF recorder: the GUI's combined view, or with no
    # GUI open the array's NDVI (pass expr="..." for another index)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    if rec.result and rec.result.get("error"):   # leaving the block does not raise
        print("recording failed:", rec.result["error"])
    else:
        print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**je smyčka pro režimy Sériové/Intervalové snímání v SDK. Protože neexistuje `Ctrl+C`, který by ji ze skriptu přerušil,**musíte** předat `count` a/nebo `duration_s` (smyčka se zastaví, jakmile je dosaženo jedné z těchto hodnot). `interval_s` se měří od začátku každého průchodu (stejně jako v grafickém rozhraní). Kontinuální průchody (`interval_s=0`) řetězí hodnoty `group_seq` → `after_seq`, takže se jedná o po sobě jdoucí synchronizované skupiny v rámci rychlosti pole. Zbývající kwargs se předávají přímo do `capture()`.
- **`continuous`**je vlastní nepřetržité zachycování backendu — režim „Continuous“ v grafickém rozhraní.**Použijte jej namísto `capture_repeated()`, pokud chcete, aby se každý spouštěcí signál ukládal na disk s vysokou snímkovou frekvencí:** `capture_repeated()` odesílá ze skriptu jeden HTTP požadavek na skupinu, zatímco `continuous()` udržuje dvě snímání v běhu uvnitř backendu bez přenosu mezi skupinami. Naměřeno při ROI 512 na sestavě s 9 kamerami: 73–77 snímků/s, oproti sestavě publikující přibližně 78. `capture_repeated()` si ponechte pro intervalové snímání (`interval_s`) a pro snímky, které kontrolujete jeden po druhém. Ve Windowsu vyloučte backend z Microsoft Defenderu, abyste dosáhli vysokých rychlostí (viz [Krok 1](#step-1--install-the-chloros-platform-package)).
- **`record`** je určen pro *monitorování*: zaznamenává vykreslený živý kompozit kombinovaného indexu. Je-li otevřeno kombinované zobrazení v grafickém uživatelském rozhraní, zaznamenává toto zobrazení tak, jak je zobrazeno. Není-li žádné otevřeno, backend vykreslí zobrazení sám: `expr`, pokud jej předáte, jinak NDVI pole (červené a NIR pásma nejblíže 660 nm a 850 nm, jak je zachycuje živý náhled), jakmile je pole zarovnáno. Pojmenujte pásma v `expr` podle toho, jak je pojmenovává pole: `<band>_<filter>` pro kameru M3C (`NIR_850_RGN`, `Red_660_RGN`, `Orange_615_OCN`), `Mono_<filter>` pro kameru M3M (`Mono_850`); krátký název barvy (`NIR`, `Red`) funguje pouze tehdy, má-li jej alespoň jeden pruh pole. Okno grafického uživatelského rozhraní otevřené během záznamu se zaznamenává, dokud zůstává otevřené. Záznam, který nezapíše žádný snímek, selže s chybami `error` a `"no-frames: …"` (pole se možná stále vyrovnává nebo stabilizuje expozici), a dojde k vyvolání výjimky `stop()` (viz [`RecorderHandle`](#recorderhandle)). Jeden kompozitní záznamník na jedno pole (vyvolá výjimku, pokud již jeden běží).
- **`burst` → `build_video`** je *analytické provedení*: `burst` zapisuje surové snímky + manifest pro každý snímek + jeden `.daq` pro každé odlišné čtení DLS pod `<output>/bursts/<base>/` při plné rychlosti snímací smyčky (bez řetězce, bez exiftool, bez živého náhledu). `build_video` časově přiřadí každý snímek k nejbližšímu `.daq` a znovu spustí řetězec radiance/reflexe/indexu. `products` je seznam `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (výchozí: kombinovaný index). `burst().stop()` také automaticky spustí sestavení kombinovanéhoindex, který je ve výsledku zastavení vrácen jako `build_job`.

##### Výsledek `build_video`

| Klíč | Význam |
| --- | --- |
| `success` | `True` pouze v případě, že byl každý požadovaný produkt sestaven podle požadavku: alespoň jeden výstup, nic v `skipped`, a ne v `degraded`. |
| `outputs` | Cesty k zapsaným videím (a GIFům). |
| `errors` | Problémy, jako text. |
| `skipped` | Jeden záznam `{"serial", "level", "reason"}` pro každý požadovaný produkt, u kterého **nebylo** vytvořeno žádné video — buď kvůli (chybí kalibrační sada pro danou kameru, chybí DLS měření odrazivosti, selhání zpracovatelského řetězce) nebo kombinovaného indexu, který obsahuje `"kind": "combined"` a `serial` `None` (mezi kamerami není žádná dvojice pásem červené/NIR, žádná skupina s radiancí pro obě pásma). Jakýkoli záznam způsobí `success` `False`; ostatní videa se stále zapisují a uvádějí v seznamu. |
| `substitutions` | Kamery vykreslené na jiné úrovni, než bylo požadováno, a důvod — RGB-kamera, u které byla požadována radiance/reflektance, obdrží své zobrazovací video s vyvážením bílé, pokud je k dispozici kalibrační balíček kamery. Nejedná se o chybu. |
| `notes` | Informační text, nejedná se o chyby — např. kamera bez výrazu indexu pro danou kameru nemá indexové video, nebo zobrazovací video RGB-kamery nemá vyváženou bílou (chybí kalibrační balíček nebo je v něm chybí tovární vyvážení bílé), s uvedením názvu kamery. Nejedná se o chybu. |
| `degraded` | `True` v případě, že byl produkt zapsán, ale nejedná se o požadovaný produkt: offline kombinovaný index je vestavěný normalizovaný rozdíl červené a NIR, proto není vyhodnocen nakonfigurovaný indexový výraz. Vytvoří `success` `False`. |
| `combined` | Souhrn kombinovaného indexu: použitá pásma, rozsah LUT, vytvořené snímky. |
| `clamp_masks` | Cesty k [maskám omezujícím smíchání](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) pro jednotlivésnímku kalibrovaných souborů TIFF (`save_tiffs=True`), v `clamp_masks/<level>/` vedle složky `<level>/` těchto souborů TIFF, po jedné pro každý TIFF, ve kterém došlo k omezování pixelu. Video žádné neobsahuje. |
| `clamp_fractions` | Co video obsahuje místo masky: jeden soubor `{"kind", "serial", "level", "frames"}` na každý videozáznam (`kind` `"combined"` pro kamery patřící do kombinovaného indexu), `frames` seznam `[frame, R, G, B]` — podíl pixelů daného snímku, které byly při odstraňování smíšení oříznuty v pásmu, které zabírá jeho kanál R / G / B (`Chloros:Band{R,G,B}UnmixClampFraction`). |

#### `RecorderHandle`

Vráceno funkcemi `ArraySession.record()` a `ArraySession.burst()`. Použijte jej jako správce kontextu pro automatické zastavení při ukončení rozsahu, nebo jej ovládejte ručně.

| Člen | Popis |
| --- | --- |
| `job_id` | ID úlohy backendu (řetězec). |
| `kind` | `"composite"` (z `record`) nebo `"raw"` (z `burst`). |
| `start_stats` | Slovník vrácený voláním `start`. |
| `result` | `None` během běhu; konečný slovník výsledků po zastavení. |
| `stats(timeout=10.0)` | Aktuální statistiky úlohy (zapsané snímky, skutečná snímková frekvence, uplynulý). |
| `stop(timeout=60.0)` | Zastavení záznamníku; vrací a ukládá do mezipaměti konečný výsledek (`.result` zůstává `None`, pokud samotný požadavek na zastavení selhal). V případě selhání záznamu `record()` — tedy záznamu, který nezapsal žádný snímek (`error` `"no-frames: …"`) — uloží výsledek do mezipaměti na `.result` a poté vyvolá výjimku `ChlorosConnectError`. Idempotentní (druhé volání vrátí výsledek z mezipaměti, aniž by vyvolalo výjimku). Opuštění bloku `with` zastaví záznamník bez vyvolání chyby, proto po něm zkontrolujte `rec.result.get("error")`. |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

#### `ContinuousHandle`

Vráceno funkcí `ArraySession.continuous()`. Použijte jej jako správce kontextu — opuštění bloku `with` zastaví záznam, pokud ještě běží — nebo jej ovládejte pomocí `status()` / `wait()` / `stop()`. V daném okamžiku běží vždy pouze jeden nepřetržitý záznam na jedno pole.

| Člen | Popis |
| --- | --- |
| `start_status` | Slovník stavu vrácený voláním start. |
| `result` | `None` během běhu; konečný stavový slovník po ukončení sběru (nastavený funkcí `stop()`, nebo voláním `status()` / `wait()`, které zaznamenalo jeho ukončení). |
| `status(timeout=10.0)` | Slovník aktuálního stavu (pole níže). |
| `wait(timeout=None, poll_s=0.5)` | Blokovat, dokud snímání neskončí samo — dosažení `max_captures` nebo `duration_s`, vyčerpání pole, zaplněný disk, chyba — a vrátit konečný stav. S `timeout` vrátí nejnovější stav po uplynutí daného počtu sekund, i když proces stále běží: zkontrolujte `running`. |
| `stop(timeout=30.0)` | Zastaví snímání a vrátí jeho konečný stav. Idempotentní (druhé volání vrátí výsledek z mezipaměti). |

Pole stavů:

| Pole | Význam |
| --- | --- |
| `running` | True, dokud snímání běží. |
| `captures` | Uloženy synchronizované skupiny. |
| `files` | Soubory zapsány. |
| `failed_files` | Soubory, které nebylo možné zapsat (viz [`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed)). |
| `failed_captures` | Pokusy o zachycení, které selhaly jako celek, včetně těch, u nichž bylo zachycení opakováno (chyba HTTP 503, zatímco pole nemělo včas žádnou novou skupinu). |
| `skipped_groups` | Skupiny, které pole publikovalo, ale nebyly uloženy (zachycování zaostávalo). |
| `captures_per_s` | Průměrná rychlost ukládání od začátku. |
| `elapsed_s` / `started_at` | Dosavadní doba běhu v sekundách; čas zahájení (sekundy od unixové epochy). |
| `last_group_seq` | Nejnovější uložená synchronizovaná skupina. |
| `last_error` | Poslední chyba, nebo `None`. |
| `stop_reason` | Po ukončení: `count` (dosaženo `max_captures`), `duration` (bylo dosaženo kódu `duration_s`), `stopped` (byl volán kód `stop()`), `array_gone` (pole bylo odpojeno), `error` (20 selhání v řadě nebo požadavek, který backend odmítl), `disk_full` (soubor, pro který na disku nebylo místo). `None` během běhu. |
| `max_captures` / `duration_s` | Počáteční rozsah (`0` = žádný). |
| `defender_cores` | Jádra procesoru, která Microsoft Defender využil v posledních 10 sekundách; `None` dokud není naměřeno žádné, a vždy `None` na hostitelském počítači bez Defenderu. |
| `defender_warned` | Hodnota True, jakmile záznam zaznamená, že Defender ho zpomaluje (na 1 jádru nebo více). Vyloučte backend ze skenování Defenderem — viz [Krok 1](#step-1--install-the-chloros-platform-package). |

```python
run = arr.continuous("captures/run1", processing="raw", max_captures=5000)
try:
    while True:
        st = run.wait(timeout=5)              # returns every 5 s while running
        print(f"{st['captures']} captures, {st['captures_per_s']:.1f}/s, "
              f"{st['skipped_groups']} skipped")
        if not st["running"]:
            break
finally:
    final = run.stop()                        # no-op once it has ended
if final["failed_files"] or final["stop_reason"] in ("disk_full", "error"):
    raise SystemExit(f"continuous capture: {final['stop_reason']}: "
                     f"{final['last_error']}")
```

### Živé rámce ve vašem vlastním zpracování

Chcete-li synchronizované snímky přivádět do vlastního kódu — strojového učení, vlastního prohlížeče, vlastního záznamníku — použijte `ArraySession.frame_stream()`. Každý tik generuje `GrabResult`: `{serial: numpy array}` pro každou kameru, **vše z jednoho spouštěče**, s ID daného spouštěče v `.group_id`. Jedná se o stejnou implementaci jako `ArrayHandle.frame_stream()` v projektovém API, takže vše pod [Jeden spouštěč na čtení](#one-trigger-per-read--group_id), [`band`](#exporting-a-single-band--band), [`align`](#aligning-the-streams--align) a [`bit_depth`](#bit_depth--trading-precision-for-bandwidth) platí pro oba.

```python
import chloros_sdk
from chloros_sdk import FrameFetchError

serials = ["213800234", "214000533", "214701288", "214701292"]   # master first

with chloros_sdk.connect_array(serials) as arr:
    try:
        for frames in arr.frame_stream(processing="raw", fps=10, duration=300,
                                       with_metadata=True):
            if not frames:              # no camera answered this tick
                continue
            if frames.errors:           # cameras missing from this trigger
                print("left out:", {s: str(e) for s, e in frames.errors.items()})
            for serial, (img, hdr) in frames.items():
                # img: numpy array (raw: one Bayer or mono plane)
                # hdr: group_id, frame_id, timestamp_ns, exposure_us, gain_db, ...
                my_pipeline(frames.group_id, serial, img, hdr["timestamp_ns"])
    except FrameFetchError as exc:      # the stream cannot continue
        print("stream ended:", exc.status, exc)
```

- **Každá vydaná sada představuje nový spouštěč.** `frames.group_id` se v rámci jednoho proudu nikdy neopakuje; pokud požadujete více `fps`, než kolik pole zpracovává, proud čeká na další spouštěč, takže běží podle pomalejšího z těchto dvou.
- **Chybějící kamery jsou pojmenovány, nejsou doplňovány.** Kamera, která nemohla být obsloužena při daném spouštěči, chybí ve slovníku a je uvedena v `frames.errors`.
- **Končí s velkým ohlasem.** `FrameFetchError` se v případě odmítnutí opakuje při dalším tik, po 60 s bez snímku nebo po 60 s bez nového spouštěče.**Co to přináší.**`frame_stream()` přenáší celé snímky numpy přes lokální HTTP připojení a dosahuje maximální rychlosti téměř**170 MB/s**. To stačí pro živé zpracování surových snímků, binárních dat nebo pole ROI, ale ne pro každý spouštěcí signál v plném rozlišení — chcete-li zachovat**každý** spouštěcí signál, uložte na disk pomocí [`continuous()`](#continuoushandle). Měření provedeno na sestavě čtyř kamer (FOCN, FRGN, FRGB, F450) s komprimovaným přenosem; každý tik představoval kompletní sadu jednoho spouštěcího signálu:

| | Plný snímač (pole 14,2–14,4 fps) | Binning 2 × 2 (pole ~33 fps) |
| --- | --- | --- |
| `frame_stream(processing="raw")` | 6,7 sad/s | 22,2 sad/s |
| `frame_stream(processing="debayered")` | 2,3 sady/s | 9,3 sady/s |
| `frame_stream(processing="display")` | 0,8 sady/s | 3,0 sady/s |
| MJPEG pro každou kameru, 4 streamy v rozlišení 720p | 13,5 snímků za sekundu každý; každý snímek přiřazen ke spouštěči | ~25 snímků za sekundu každý; ~45 % spouštěčů přítomných ve všech 4 |

Pro propustnost a demosaikování na vaší straně použijte `processing="raw"`; úroveň `debayered` také odstraňuje tmavý offset kalibračního balíčku, což demosaikování na úrovni `raw` nedělá.

#### MJPEG streamy pro každou kameru (bez RTSP)

**RTSP není podporováno.** Rozhraní pro streamování je MJPEG přes HTTP (`multipart/x-mixed-replace`, jeden soubor JPEG na část, každá část s `Content-Length`), jedna URL na kameru:

| URL | Co poskytuje |
| --- | --- |
| `GET /api/camera/stream/<serial>` | 8bitový náhled, až ~30 snímků za sekundu. U členů pole určují velikost a kvalitu parametry `max_h` (výchozí 720, 64–2160) a `jpeg_q` (výchozí 80) nastavují velikost a kvalitu. |
| `GET /api/camera/<serial>/stream-annotated` | Stejný náhled s nakreslenými překryvy grafického uživatelského rozhraní (zebra, mřížka, nitkový kříž, histogram, peaking, bodový expozimetr). `ChlorosLocal.camera_stream()` jej za vás přečte. |
| `GET /api/camera/<serial>/stream-reflectance` | 8bitová odrazivost na pásmo — viz [Živý proud odrazivosti](#live-reflectance-stream-mjpeg). |

- **Spouštěcí hlavičky.** U prvku pole obsahuje každá část všech tří datových toků `X-Chloros-Group-Id`, `X-Chloros-Frame-Id` a `X-Chloros-Timestamp-Ns`. Části z různých kamer se stejným `X-Chloros-Group-Id` představují stejný spouštěcí signál: shromážděte části a seskupte je podle této hlavičky, abyste sestavili sadu. Samostatná kamera nemá žádnou skupinu: její části náhledu nenesou žádnou z těchto hlaviček (její součásti pro odrazivost nesou prázdný záhlaví `X-Chloros-Group-Id`).
- **Proud náhledu při zatížení vynechává snímky.** Poskytuje nejnovější snímek a přeskočí všechny, které nestihl včas odeslat, takže při vysokých frekvencích pole se ne každý spouštěcí signál dostane do proudu každé kamery (viz tabulka výše). Pro každý spouštěcí signál použijte `frame_stream()` nebo `continuous()`.
- **`/api/camera/stream/<serial>` se pozastaví, dokud je otevřen kombinovaný živý stream pole** (`/api/camera/array/stream/<array_id>` — otevírá jej zobrazení pole v grafickém uživatelském rozhraní). Stream každého člena pak odešle jeden zástupný snímek „Paused (array stream active)“ bez hlaviček `X-Chloros-*` a nic dalšího, dokud se kombinovaný proud nezavře; poté se sám obnoví. `stream-annotated` a `stream-reflectance` se nezastaví. Zavřete zobrazení pole v grafickém uživatelském rozhraní, když skript čte náhledové proudy jednotlivých kamer.
- **Pro čísla, nikoli obrázky, použijte `frame_stream()`.**Formát JPEG je ztrátový a 8bitový; `frame_stream()` vrací stejné výstupy bez kodeku v cestě.

### Intervalové snímání pro průzkumy pomocí dronů

Průzkumný let vyžaduje jednu synchronizovanou skupinu snímků každých několik metrů letu, nikoli každý snímek, který pole dokáže vyprodukovat. To řídí dvě nastavení:**`target_fps`**určuje, jak často kamery pořizují snímek, a**`capture_repeated(interval_s=…)`** určuje, jak často se snímek ukládá.**Kamery pokračují ve snímání i mezi ukládáním.** Při každém intervalu se uloží nejnovější synchronizovaná skupina; snímky mezi nimi se nezapisují. Automatická expozice se upravuje u každého snímku, takže v okamžiku uložení je expozice již ustálená. Neexistuje režim, ve kterém by se kamery mezi jednotlivými snímky zastavovaly a pro každý snímek se znovu probouzely: každý snímek by pak musel čekat, až se spustí stream a ustálí se expozice.**Nastavte snímkovou frekvenci podle potřeb intervalu.**Pokud ponecháte automatické nastavení, pole běží nejvyšší frekvencí, kterou hostitel a síť zvládnou. Snížením frekvence se sníží zahřívání kamer, zatížení sítě a procesoru hostitele, aniž by se změnilo to, co se ukládá. Zvolte frekvenci, která poskytuje**2–4 snímky na interval**:

- Každé ukládání čeká na další snímek po jeho vyžádání, takže uložená skupina se dostane až o jednu snímkovou periodu později než v plánovaném okamžiku. Vyšší snímková frekvence tento odstup zmenšuje.
- Automatická expozice se upravuje jednou za snímek, takže velmi nízká snímková frekvence zpomaluje, jak rychle expozice reaguje na přelétávající mrak.

**Interval vypočítejte z letového plánu:**

```
spacing    = along-track footprint × (1 − forward overlap)
interval_s = spacing / ground speed
```

Například stopa o šířce 60 m s 80% překrytím vpřed odpovídá rozestupu 12 m; při rychlosti 10 m/s to je `interval_s=1.2`, a fixovaná hodnota `target_fps=3` odpovídá 3 až 4 snímkům na interval. Snímání je časově řízené: SDK nemá režim spouštěný vzdáleností ani GPS, proto udržujte na měřicích trasách stálou rychlost vůči zemi.

**Ukládejte surová data přímo na letadle a kalibrujte je později.** Hodnota `interval_s` se měří od začátku každého ukládání, takže pokud ukládání trvá déle než interval, začne další ukládání se zpožděním a rozestup se posune. Uložení surových dat spolu s hodnotou světelného senzoru (`processing="raw"`; tato hodnota se ukládá ve výchozím nastavení) zajišťuje, že každé ukládání je krátké a každý snímek lze následně převést na odrazivost.

```python
import chloros_sdk

INTERVAL_S = 1.2      # from the flight plan
FLIGHT_S = 20 * 60    # bounds the loop; it also stops at count= if given

with chloros_sdk.connect_array(serials, target_fps=3.0) as arr:
    groups = arr.capture_repeated(
        "survey/", interval_s=INTERVAL_S, duration_s=FLIGHT_S,
        processing="raw", render_index=False,
        on_capture=lambda i, r: print(f"group {i}: {len(r)} frames"))
```

- **`connect_array(target_fps=…)`**nastaví frekvenci od začátku;**`arr.set_target_fps(fps)`**ji mění během letu bez opětovného připojení a**`arr.set_target_fps(None)`** ji vrátí do automatického režimu. Rychlost vyšší, než jakou mohou hostitel a síť zvládnout, je snížena na tuto horní hranici — zkontrolujte `clamped` ve výsledku — a nejnižší rychlost je 0,1 fps.
- **Nepoužívejte `burst()` k redukci snímků.** Zapisuje každý snímek, který pole vyprodukuje; navazuje na pevně nastavený `target_fps`, takže je to správný nástroj pouze v případě, že jsou požadovány všechny snímky.
- **`target_fps` neopravuje poškozené snímky.** Jeho snížením se sníží počet generovaných snímků, nikoli rychlost odesílání jednotlivých snímků; v případě poškozených snímků v GVSP místo toho snižte hodnotu `wire_ceiling_mbps` (viz [Stav pole](#array-health--which-subsystem-is-losing-frames)).

```python
r = arr.set_target_fps(5.0)
if r["clamped"]:
    print(f"asked for {r['requested']} fps, running at {r['target_fps']}")
```

### Připojení k již připojenému poli — `attach_array`

Pokud je pole již spuštěno (bylo otevřeno v grafickém rozhraní nebo předchozí relace SDK volala `connect_array`), použijte `attach_array` k získání jeho identifikátoru namísto opětovného připojení. `connect_array` v takové situaci vždy vyhodí chybu „Kamera  <sn>je již v poli<id>“, protože odeslání požadavku POST s kódem `/array/connect` pro člena v poolu není idempotentní; `attach_array` čte `/api/camera/array/list` a porovnává buď podle array_id, nebo podle sériových čísel.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Vzor: Skripty SDK, které sdílejí prostředí s desktopovým grafickým uživatelským rozhraním, by měly nejprve zkusit `attach_array` a přejít na `connect_array`, pokud v poolu ještě není žádné pole.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Připojené pole je zapůjčené, není ve vlastnictví.** Handle z `attach_array` nikdy pole nezruší: neuvolní ho ani ukončení bloku `with`, ani explicitní volání `disconnect()` (explicitní volání vypíše na stderr jeden řádek s informací, že pole zůstalo spuštěné). Platí stejné pravidlo jako u `CameraSession` / `DAQSensorSession`. Společné sdílení s grafickým rozhraním je tedy bezpečné s `with` i bez něj:
>
> ```python
> with chloros_sdk.attach_array(serials) as arr:
>     arr.capture("output/", processing="reflectance")
> # … the array stays up for the GUI
> ```
>
> Pokud jej chcete skutečně ukončit – například abyste se znovu připojili k jinému `wire_ceiling_mbps` – zavolejte `arr.force_disconnect()`. Opětovné připojení po pouhém `disconnect()` na připojeném handle selže s hláškou „Kamera je<sn> již v <id>poli“, protože nic nebylo uvolněno.

### Pomocník pro analýzu sítě

Užitečné před otevřením pole — předpovídá, zda se vaše navrhovaná nastavení vejdou:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    # width / height are sensor pixels BEFORE binning (what connect_array
    # takes with binning=); out_width / out_height is what is streamed.
    print(f"Connect with width={r['width']} height={r['height']} "
          f"binning={r['binning']} ({r['out_width']}x{r['out_height']} streamed)")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

```python
analyze_array_network(
    master_serial,
    slave_serials=None,
    width=2048, height=1536,              # sensor pixels before binning
    pixel_format=None,                    # None = what connect applies (Mono12 / BayerRG12)
    binning=1,                            # 1 | 2 | 4
    force_tier=None,
    packed="auto",                        # "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",
    timeout=30.0,
) -> dict
```

Nikdy nespustí backend: ten, který neodpovídá, vyvolá chybu `ChlorosConnectError`, zatímco odmítnutý požadavek se vrátí jako výsledek. Chybný `packed` vyvolá chybu `ValueError`.

`status` je jedním z kódů `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (jinak `error` — včetně zamítnuté žádosti: není přihlášen, `error_code: AUTH_REQUIRED`, nebo plán pod minimální hranicí SDK, `PLAN_UPGRADE_REQUIRED`). `auto_capped_fps` znamená, že požadované rozlišení vyhovuje prstenci RX pouze při omezené spouštěcí frekvenci — zachovejte rozlišení a přejděte z `target_fps=result["recommended"]["recommended_target_fps"]` na `connect_array` (viz [Příklad 6](#6-capability-probe-before-connecting-a-4-cam-array)). `auto_shrunk` se objeví pouze tehdy, když nelze požadovaný snímek doručit při žádné snímkové frekvenci; connect nikdy sám o sobě snímky nezmenšuje, proto použijte `recommended` sami.

**Jak číst projekci** (stejný model jako panel Nastavení pole v grafickém uživatelském rozhraní):

- **Burst (`frame_bytes_total`) se sčítá pro každou kameru zvlášť v reálném formátu pixelů dané kamery.**Mono**M3M**kamery streamují Mono12 (2 B/px) nebo s nastavením `pixel_format` mono formát své hloubky (`"BayerRG8"` je `Mono8`, 1 bit/pixel), takže bez něj má snímek v plném rozlišení ze čtyř kamer**~25 MB** se třemi mono kamerami, nikoli ~12,6 MB, jak by vyplývalo z předpokladu, že všechny jsou 8bitové. Backend určuje formát každé kamery na základě jejího modelu.
- **Admittance (`burst_fits_nic_ring`) zohledňuje odvod dat**, nikoli na rozdílu mezi celým burstem a prstencem: sim-emit se použije, když host vyprázdní přijímací prstenec rychleji, než jej kamery naplní. Host 10G + kamery 1 GbE**přijímají** snímky v plném rozlišení, i když burst přesahuje kapacitu prstence; 1 GbE hostitel blokuje (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` je konzervativní horní hranice sériového načítání** — `max(readout+emit, N×emit)` s vysíláním na kameru omezeným na 1 GbE Camera Link, nezávisle na expozici. Např. ~2,8 fps pro pole se 4 kamerami v plném rozlišení a 12 bitech (odpovídá naměřeným hodnotám ~2,7–3,0 v runtime). Úplný model: [Referenční příručka CLI→ Model snímkové frekvence a sériového snímání pole](cli-reference.md#array-fps--burst-model).
- **Po nepřetržitém snímání projekce udává, jakou hodnotu si udržela.** Snímání trvající 30 s nebo déle při automatické snímkové frekvenci zaznamenává frekvenci, na které se systém při dané velikosti pole na tomto počítači ustálil; krátké pokusy regulátoru snímkové frekvence o vyšší frekvenci jsou vynechány. Od té chvíle řádek v `array-connect`, analýza `connect_array()` a panel Nastavení pole uvádějí tuto snímkovou frekvenci, pokud je nižší než frekvence modelu, spolu s tím, co ji omezilo v `cycle_limiter`: `"clock"` (pole běželo rychlostí, na které se ustálilo, smyčka snímání čekala na spouštěcí takt), `"loop"` (smyčka snímání již neměla dostatek času), `"capture"` (snímání zaostávalo a přeskočilo skupiny) nebo `"measured"` (není uvedeno, o který případ se jedná). Hodnota modelu je uvedena v `wire_fps_max`. Pole stále stoupá na hodnotu, kterou přenáší linka.
- **Přetížení (`oversubscribed: true`) znamená, že linka nedokáže přenášet pole při žádné frekvenci, které kamery dosahují** — pole fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) vykazují hodnotu 0 a ani binning to nedokáže napravit (snižuje počet bajtů na snímek, nikoli počet bajtů za sekundu při daném tempu). Samotné pomalé připojení k síti k tomuto jevu nevede: místo toho klesá snímková frekvence a pokud nelze tempo kamer dostatečně zpomalit, úroveň se změní na `slip-emit-and-capture` (sériová). Řešením je menší počet kamer, použití jumbo rámců nebo rychlejší síťová karta; `max_cams_collision_safe` udává maximální kapacitu vašeho připojení a kamer. Odpověď obsahuje také kódy `aggregate_demand_bps`, `collision_safe_ceiling_bps` a `per_cam_floor_bps`. Viz [Překročení kapacity](#over-subscription-the-per-cam-floor).

### Vyhledávání a výpis

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

Každá z nich přijímá pouze klíčová slova `backend_url=` a `timeout=30.0` a vrací seznam slovníků. `discover_lattice_cameras()` (`serial`, `ip`, `mac`, `model`, `lattice_model`, `vendor`, `connected`, `array_id`, …) přijímá také `auto_start_backend=True`; volání `list_*` nikdy nespustí backend.

---

## Smart-AE / Smart-Capture

Pole LATTICE spouští nepřetržité automatické zaostřování (AE) na pozadí, jakmile jsou připojena, ale u nově nasměrované scény trvá chvíli, než dojde ke konvergenci. **Smart-capture** je praktická funkce: zjišťuje expozici každé kamery, počká, až bude pole stabilní v celém okně, a poté spustí snímání. Je to ekvivalent v grafickém uživatelském rozhraní: tlačítko „smart“ capture v desktopové aplikaci volá stejný koncový bod backendu.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Při ovládání přes `ChlorosProject` (další část) získáte více nastavení:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Zásady inteligentní automatické expozice (smart-AE) jsou ve výchozím nastavení konzervativní. Pro náročnou radiometrickou práci nastavte přísnější hodnoty pomocí `exposure_tolerance_pct`; pro rychle se měnící scény, kde vám stačí „přibližný výsledek“, nastavte volnější hodnoty.

---

## Relace senzorů DAQ

Trvalý fond backendů pro spektrální senzory (DAQ-U přes USB, DAQ-M přes BLE, DAQ-E přes Ethernet). Odráží povrch kamery: inteligentní detekce, opětovné použití fondu, idempotentní připojení.

### Inteligentní detekce (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Priorita: Ethernet → BLE → USB. Zadejte libovolný explicitní parametr k pevné volbě přenosového protokolu.

### Pevně zvolený přenosový protokol

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# A specific sensor by id (as list_daq_sensors() shows it; ':' or '-' alike)
daq = chloros_sdk.connect_daq_sensor(sensor_id="CB-7C-A7-5A-73")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

`sensor_id=` určuje jeden senzor a otevřen bude vždy pouze tento senzor. Je porovnán se senzory, které jsou již otevřené; poté je u DAQ-E, jehož síťový sken vidí pouze podle IP adresy, vyžádáno (pouze pro čtení) jeho jméno, MAC a sériové číslo, takže jej najde i kterýkoli z nich), BLE MAC adresa zařízení DAQ-M a ID, které zařízení DAQ-U nebo DAQ-M nahlásí po otevření (jakékoli jiné otevřené zařízení pro kontrolu se znovu zavře). Pokud není nalezeno, `connect_daq_sensor` vyvolá `ChlorosConnectError` s výpisem toho, co bylo nalezeno; včetně adresy (`port=` / `mac=` / `eth_host=`), na které je připojen jiný snímač, vyvolá chybu také. Nikdy se nevrátí k tomu snímači, který je viditelný — k tomu slouží volání s příznakem nula.</id></sn></id></sn>

### `connect_daq_sensor()` Podpis

```python
connect_daq_sensor(
    *,                                    # every argument is keyword-only
    sensor_id=None,                       # one named sensor (see above)
    transport=None,                       # "usb" | "ble" | "eth"; None = smart-detect
    port=None,                            # DAQ-U serial port ("COM3", "/dev/ttyUSB0")
    mac=None,                             # DAQ-M BLE MAC (implies transport="ble")
    eth_host=None,                        # DAQ-E hostname or IP (implies transport="eth")
    integration_time=None,                # ms; None = the sensor's own
    frame_avg=None,
    enable_ae=None,
    cap_id=None,                          # cap correction: "sunshine_cosine", "fov_30", ..., "none" (bare)
    start_streaming=True,
    backend_url="http://127.0.0.1:5000",
    timeout=60.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> DAQSensorSession
```

Senzor, který je již v sadě, se vrací v původním stavu (vypůjčený): `integration_time`, `frame_avg` a `enable_ae` se na něj nevztahují. Nejprve jej odpojte, abyste jej mohli otevřít s vašimi nastaveními. Výjimkou je `cap_id`: na vypůjčeném senzoru se nastavuje pomocí `set_cap()`. Pokud se vynechá, platí výchozí nastavení backendu (viz poznámka pod tabulkou metod).

### Metody `DAQSensorSession`

| Metoda | Popis |
| --- | --- |
| `status(timeout=10.0)` | Souhrn záznamu v poolu (stav streamování/nahrávání, rozsah vlnových délek, kalibrační sha, integrační čas, frame_avg, stav AE). Pro DAQ-E, jehož stream utichl: `reconnect` (pokud funguje automatické opětovné připojení backendu: `state`, `attempts`, `next_attempt_in_s`, `last_error`; `None` během přenosu), `stream_lost_s`, `reconnects` a `last_reconnect`, a `record_gaps` pro průběžný záznam; `rx_stream_fault` udává stav opětovného připojení, dokud nedorazí první spektrum. |
| `latest(*, n=1, timeout=10.0)` | Vrátí až N nejnovějších spektrálních rámců (každý `ts_ns`, `spectrum`, `is_saturated`, `integration_time`, `x`, `y`, `z`). |
| `stream_start()` / `stream_stop()` | Obnovit / pozastavit streamování (handle zůstává otevřený). |
| `record_start(*, output_dir=None, device_name=None, timeout=15.0)` | Spustí nahrávání souboru .daq. Vrátí cestu k souboru. Je odmítnuto pro DAQ-U/M bez kalibračního balíčku AWS (DAQ-E je výjimkou). |
| `record_stop()` | Zastaví záznam. Vrátí `{path, rows, warning, gaps}`; `warning` je `None` v případě čistého záznamu, v opačném případě diagnostiku backendu (například prázdný soubor nebo mezeru ve streamu), která se také vypíše na stderr. `gaps` vypisuje každý úsek, ve kterém proud nic nedodal: `start_utc`, `end_utc` (`None`: stále ztraceno při zastavení), `duration_s`, `reconnected`, `marked_in_file`. Při nepřetržitém záznamu je tento záznam prázdný. |
| `set_cap(cap_id, timeout=15.0)` | Nastavte profil korekce maximálních hodnot, který se bude aplikovat na naměřené hodnoty počínaje dalším záznamem: `"sunshine_cosine"`, `"fov_15"` ... `"fov_90"`, nebo `"none"` (nebo `None`) pro holý snímač. Vrací souhrn snímačesouhrn, který `cap_id` zpětně načte. DAQ-E-S udržuje hodnotu `as_recorded` bez ohledu na dotaz (má vestavěný difuzor). Vypne `ChlorosConnectError` pro profil, který se nepodaří načíst. CLIA: `chloros-cli daq pool-set-cap`. |
| `enable_ptp(timeout=15.0)` | Pouze DAQ-E: znovu zapne PTP u jednotky, u které je PTP vypnuté (`status()["rx_ptp_enabled"]` je `False`; nastavení se ukládá do jednotky, takže zůstává vypnutý, dokud se znovu nezapne). Vrací aktualizovaný stav. Vyvolá `ChlorosConnectError` u DAQ-U / DAQ-M nebo v případě, že zařízení nepotvrdí stav. Chloros nikdy sám zařízení znovu nezapne, protože vypnutý stav může být záměrný. CLI: `chloros-cli daq pool-enable-ptp`. |
| `device_info(timeout=15.0)` | Pouze DAQ-E: vlastní nastavení jednotky. `firmware` (`running`, `bundled`, `update_available`), `ip` a `ip_mode` (`dhcp` / `linklocal` / `static`: to, co běží nyní), `network` (uložené nastavení: `mode`, plus `ip` / `mask` / `gateway` v klidovém stavu), `network_change_pending` (platí, dokud jednotka neprovede uložené nastavení, které zahrnuje i `set_network(..., reboot=False)`), `auto_stream`, `ptp`. `firmware["update_available"]` je `None`, pokudverzi zařízení nebylo možné načíst. Dokud proud dat ze zařízení nedosáhne tohoto počítače, je načten při novém připojení a slovník přidá `stream_silent` (`True`), `stream_fault` a `reconnect`; jednotka, která neodpovídá, vyvolá chybu. CLI: `pool-info`. |
| `set_network(mode="auto", *, ip=None, mask=None, gateway=None, reboot=True, force=False, comeback_timeout=45.0)` | DAQ-E pouze: DHCP (`"auto"`) nebo statická adresa. Pro uplatnění nastavení se zařízení restartuje a znovu připojí; tato relace pokračuje v práci a `address` sleduje zařízení. `reboot=False` nastavení pouze uloží. Viz [Nastavení zařízení DAQ-E](#daq-e-device-settings). CLI: `pool-set-network`. |
| `set_auto_stream(enabled, timeout=15.0)` | Pouze DAQ-E: streamování při zapnutí bez připojeného softwaru (uloženo v jednotce). Vrátí potvrzené nastavení. CLI: `pool-set-auto-stream`. |
| `reboot(*, comeback_timeout=45.0)` | Pouze DAQ-E: restartujte zařízení a znovu jej připojte s původními nastaveními. CLI: `pool-reboot`. |
| `update_firmware(*, comeback_timeout=45.0)` | Pouze DAQ-E: nahrajte firmware dodaný s touto verzí Chloros do zařízení s starší verzí a poté jej znovu připojte (trvá to asi minutu; nevypínejte napájení). Vrací kódy `updated`, `fw_before`, `fw_after`, `bundled` a `warning`, pokud se zařízení připojilo pomalu nebo se připojilo s jinou verzí; `updated` je `False` pro zařízení, které již má aktuální verzi. Vyvolá výjimku `ChlorosConnectError` u jednotky, jejíž verzi nelze přečíst. CLI: `pool-update-firmware`. |
| `disconnect()` | Uvolnění z fondu. Žádnápro připojené, ale nevlastněné popisovače. U senzoru, který otevřela tato relace, je to `force_disconnect()` a vyvolá chybu `ChlorosConnectError`, pokud uvolnění není potvrzeno (ukončení bloku `with` nikdy nevyvolá výjimku). |
| `force_disconnect(timeout=30.0)` | Uvolnění senzoru bez ohledu na to, kdo jej otevřel — například senzor, který v poolu zanechal havarovaný skript a ke kterému se nový skript dostane pouze jako připojená rukojeť, jejíž `disconnect()` neprovádí žádnou akci. Ukončení senzoru jiného klienta dokončí veškeré záznamy, které právě provádí. Vrátí se, jakmile backend potvrdí uvolnění, nebo pokud není co uvolnit (senzor není v poolu nebo neběží žádný backend); uvolnění, u kterého vypršel časový limit nebo které backend odmítl, vyvolá chybu `ChlorosConnectError`. CLI: `chloros-cli daq pool-disconnect --sensor-id ID`. |

Atributy: `sensor_id`, `model`, `transport`, `address` (následuje po DAQ-E, které `set_network()` přeadresovalo), `info` a `backend_url`.

> **Uveďte název krytky.** Modely DAQ-U, DAQ-M a DAQ-E se dodávají bez kondenzátoru a Chloros nedokáže rozpoznat, zda je kondenzátor nainstalován: senzor připojený bez `cap_id` dostane hodnotu `sunshine_cosine`, což je předpoklad, že podle záznamů byl (`cap_id_source=auto_default`). Nechráněný DAQ-U po této korekci vykazuje asi 30x vyšší hodnoty, proto pro nechráněný snímač zadejte `cap_id="none"` (nebo `set_cap("none")`) a profil krytky, kterou jste jinak nasadili.

### DAQ-E-S — pevný difuzor a snímač náklonu

**DAQ-E-S** je model DAQ-E se dvěma rozdíly, které mají význam pro software. Všechno ostatní — vyhledávání, připojení, PTP, záznam, níže uvedená nastavení zařízení — funguje přesně stejně jako u DAQ-E.**Jeho difuzor je pevně zabudovaný, takže nevyžaduje žádnou korekci krytu.** Difuzor typu Sunshine je zabudovaný a byl na jednotce již při měření její tovární kalibrace, takže tato kalibrace již udává skutečné sálavé ozáření směrem dolů. Chloros nastavuje svůj kryt na `as_recorded` (bez profilu) bez ohledu na to, co je požadováno: kryt pojmenovaný podle něj (`sunshine_cosine`, `fov_*`, `none`) je ignorován s hláškou a jeho záznamy ukládají hodnotu `cap_id_source=model`, takže se u něj nikdy nespustí varování o předpokládaném limitu. Nepoužívejte na jeho data ani ruční omezení: `sunshine_cosine` v horní poloze by vykazoval asi 11násobně vyšší hodnoty.

**Je vybaven tříosým akcelerometrem.** Kosinusová odezva difuzoru slunečního záření platí pouze tehdy, je-li v horizontální poloze, takže nakloněný senzor hlásí nesprávnou intenzitu záření — a odrazivost tuto chybu přebírá — bez dalších příznaků. Každá hodnota z DAQ-E-S proto obsahuje informaci o své poloze:

| Pole | Význam |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Úhly ve stupních. `None`, pokud firmware označí úhel jako nedefinovaný (volný pád, nefunkční osa). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Zrychlení v mili-g. `mag_mg`, pokud se hodnota výrazně liší od ~1000, znamená to, že se jednotka pohybovala a úhly pro daný vzorek nemají význam. |
| `cal_applied` | Nastaveno: osy a úhly jsou v souřadnicovém systému difuzoru (je použita kalibrace upevnění jednotky). Vymazáno: surový souřadnicový systém desky, který může vykazovat až ~2,3° náklonu, který není skutečný. |
| `sample_age_ms` | Stáří vzorku z akcelerometru v okamžiku odeslání spektra. |
| `temp_raw` | Nekalibrovaný **trend** teploty čipu — pouze rozdíly, nikdy naměřená hodnota teploměru. |

`DAQSensorSession.latest()` je vrací pod klíčem `imu` každého snímku (`None` na DAQ-U, DAQ-M nebo běžném DAQ-E). `.daq` zaznamenané z DAQ-E-S ukládá stejné hodnoty do svých sloupců `imu_*` (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) vedle každého spektra, takže čtenář může rozlišit mrak od převráceného stativu. U všech ostatních modelů jsou tyto sloupce prázdné.

Datový proud o poloze podává zprávy ve vlastních polích v `status()` (`rx_imu_stream_fault`, `rx_imu_stream_faults`, `rx_last_imu_stream_fault`), odděleně od spektrálních, takže výpadek akcelerometru není nikdy interpretován jako mezera v záznamu. `discover_daq_sensors()` a `model` dané relace uvádějí `DAQ-E-S`.

### Nastavení zařízení DAQ-E

```python
with chloros_sdk.connect_daq_sensor(eth_host="daq-e-def330.local") as daq:
    info = daq.device_info()
    print(info["ip"], info["ip_mode"], info["firmware"])

    # A fixed address on the network this computer is on
    daq.set_network("static", ip="192.168.1.50", gateway="192.168.1.1")
    print(daq.address)              # 192.168.1.50: the session follows it

    if info["firmware"]["update_available"]:
        daq.update_firmware()
```

- **V síti tohoto počítače musí být nastavena statická adresa.** `set_network` vyvolá chybu `ChlorosConnectError` u zařízení, jehož podsíť neobsahuje adresu tohoto počítačev síti senzoru: po restartu by se k senzoru nic nedostalo, aby bylo možné nastavení vrátit zpět. Chybu `force=True` přejděte pouze v případě, že bude tomuto počítači přiřazena nová adresa nebo bude přesměrován do nové podsítě.
- **`set_network`, `reboot` a `update_firmware` restartují senzor** a vyvolají chybu `ChlorosConnectError` během záznamu, a pokud se nevrátí do `comeback_timeout` sekund. ID senzoru a název hostitele `daq-e-<id>.local` se nikdy nemění.
- **Záznam, který byl spuštěn těsně před restartem, je tímto restartem ukončen a pojmenován.** Když se senzor vrátí, vrácený slovník obsahuje `recording_closed` (cestu k souboru) a hodnotu `warning`, která to potvrzuje. Pokud se senzor nevrátí nebo selže přenos firmwaru, uvede to místo toho hodnota `ChlorosConnectError`: její zpráva končí varováním a jsou nastaveny hodnoty `e.recording_closed` / `e.warning` (oba `None`, pokud nebyl uzavřen žádný záznam). Jakmile se senzor vrátí, spusťte záznam znovu.
- **Znovu se připojí pouze samotný senzor.** Po restartu musí jednotka odpovědět na dotaz, kde se nacházela -- a po `set_network("auto")` také pod svým jménem `.local` -- svým jménem, názvem hostitele a modelem, z síťového adaptéru (MAC), který měl; `address` se stane IP adresou, která odpověděla. Odpověď pod daným názvem se počítá pouze s touto MAC adresou, takže z počítače, který se k senzoru připojuje přes router, nebo z výchozího NAT síťového prostředí WSL2, není senzor, který byl přesunut službou DHCP, znovu připojen (`ChlorosConnectError`; připojte jej znovu na jeho novou adresu).
- **Aktualizace firmwaru vyžaduje, aby se senzor znovu připojil.** `update_firmware()` vyzve jednotku a ta otevře zpětné TCP připojení k tomuto počítači pro stažení obrazu na portu, který vybere operační systém. Backend naslouchá na adaptéru, který se připojuje k jednotce (na všech adaptérech, pouze pokud nelze určit, který to je), a obsluhuje pouze tuto jednotku: jakékoli jiné připojení je odmítnuto (`[DAQ-FW] … refused a firmware connection from <ip>` ve výstupu backendu), zatímco backend čeká. Firewall, který blokuje připojení, způsobí selhání aktualizace s chybou `device never connected back on TCP <port>`. Instalační program pro Windows přidá pro něj pravidlo (viz [Krok 1](#windows-exe)); na jiném firewallu povolte příchozí TCP spojení do backendu ze podsítě senzoru.
- **V prostředí WSL2 použijte zrcadlenou síť** (`networkingMode=mirrored` v `.wslconfig`): ve výchozím režimu NAT je každá statická adresa odmítnuta (vlastní adresa WSL není v síti senzoru), datový proud se nedostane do WSL a aktualizace firmwaru nelze dokončit.
- Grafické uživatelské rozhraní (GUI) udržuje vlastní připojení k senzoru: restart ze skriptu ukončí relaci GUI s tímto senzorem. GUI má stejná nastavení (karta Světelné senzory, nastavení senzoru, Síť).

### Vyhledávání — nalezení adresy pro připojení

`discover_daq_sensors()` prohledává rozhraní USB / BLE / ETH a hledá senzory, které *by* bylo možné otevřít. Jedná se o protějšek `discover_lattice_cameras()` pro DAQ a jediný způsob, jak získat **BLE MAC adresu zařízení DAQ-M** — DAQ-E má název hostitele a DAQ-U COM port, ale MAC adresa není ani vytištěna na zařízení, ani uvedena v operačním systému.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               DAQ-U  {'vid': '0403', 'pid': '6001', 'manufacturer': 'FTDI'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Pole | Popis |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM port / BLE MAC / název hostitele — předat do `connect_daq_sensor` jako `port=` / `mac=` / `eth_host=`. |
| `display` | Člověkem čitelný popisek. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` \| `DAQ-E-S`, nebo `None`. Ethernetový senzor má označení `DAQ-E-S` nebo `DAQ-E`, jak se sám identifikuje při dotazu, pouze pro čtení, během skenování (`DAQ-E`, pokud neodpoví včas). Sériový port USB vykazuje hodnotu `DAQ-U`, pokud jeho mostový čip patří mezi ty, které používá DAQ-U (FTDI, CH340, CP210x, Prolific) — což naznačuje popis portu, nejedná se o sondu — a `None` pro jakékoli jiné USB zařízení, které je stále uvedeno v seznamu, takže jej můžete předat jako `port=`. Sériový port bez identifikátoru výrobce USB a bez takového mostového čipu (UART na základní desce, jako je například `/dev/ttyS*` v systému Linux, sériové připojení Bluetooth, Intel AMT) nemůže být DAQ-U a není uveden v seznamu. |
| `extra` | Podrobnosti podle typu přenosu (inzerovaný název BLE; USB `vid`, `pid`, `manufacturer`, `serial_number`; DAQ-E ip/fw/…). Prázdné hodnoty jsou vynechány. |

| Parametr | Výchozí hodnota | Popis |
| --- | --- | --- |
| `transports` | všechny tři | Sekvence (nebo řetězec ve formátu CSV) omezující skenování. Stojí za to zadat, pokud víte, co chcete — BLE je pomalejší část. |
| `scan_timeout` | 5 | Okno skenování pro jednotlivé transporty v sekundách; backend omezuje rozsah na 1–20. |
| `timeout` | 60,0 | Horní limit HTTP pro celé volání (stejně jako jinde v SDK). |
| `auto_start_backend` | `True` | Spustí lokální backend, pokud žádný neběží. Nikdy se nespustí pro vzdálený `backend_url`. |

> **Senzory, které jsou již otevřené v poolu, se nezobrazují.** Připojené periferní zařízení BLE přestane vysílat reklamy a otevřený COM port nelze prozkoumat, takže seznam zjištěných zařízení uvádí pouze to, co je *k dispozici pro připojení*. Hned po připojení zařízení je očekáván prázdný výsledek — pro zařízení, která již máte připojená, použijte `list_daq_sensors()`. Transporty, u nichž nelze provést skenování (není nainstalován bleak / zeroconf), jsou přeskočeny, místo aby vyvolávaly chybu, takže počítač bez Bluetooth stále dostává odpovědi pro USB a ETH.

### Výpis

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Který DAQ napájí kterou kameru — `daq_bindings()`

Reflexe snímku a soubor `.daq`, do kterého se ukládá, používají DAQ přiřazený k dané kameře nebo poli, jinak nejnovější DAQ, který má backend k dispozici. Nastavuje se to výběrem světelného senzoru v grafickém rozhraní a souborem `chloros-cli daq pool-bind`; stejně jako:

```python
chloros_sdk.set_daq_bindings(cameras={"214701281": "COM7"},          # a sensor id or an address
                             arrays={"array-1779862544497": "daq-e-def330"})
chloros_sdk.set_daq_bindings(cameras={"214701281": None})            # None clears it
for serial, b in chloros_sdk.daq_bindings()["cameras"].items():
    print(serial, b["binding"], b["source"], b["resolved"]["in_pool"])
```

`daq_bindings()` vrací `{"cameras": {serial: b}, "arrays": {array_id: b}}`: každý `b` obsahuje `binding` (nastavená hodnota), `source` (`project`, z `sensors.json` projektu, nebo `runtime`), `known` / `known_as` (zda název klíče kamery nebo pole označuje něco, co Chloros zná, a jak: `connected`, `project` — `cameras.json` otevřeného projektu — nebo, v případě kamery, `seen`: již dříve otevřeno na tomto počítači; `known: False` pro klíč, který nic neoznačuje) a `resolved` (senzor fondu, ke kterému je nyní namapován: `in_pool`, `sensor_id`, `address`, `model`, `streaming`). Vazba za běhu má přednost před vazbami v projektupro danou relaci; projektový soubor se nemění. Odpojení kamery, pole nebo DAQ zruší všechna přiřazení tohoto hardwaru, bez ohledu na to, kdo je nastavil (členy pole se přiřazují k poli).

`set_daq_bindings()` váže pouze kameru, která je připojena, nachází se v otevřeném projektu nebo byla dříve otevřena na tomto počítači, a pole, které je připojeno nebo se nachází v otevřeném projektu (nebo klíč, který již byl vázán). Jakýkoli jiný klíč vyvolá `ChlorosConnectError` s názvem (`400`, `code`, `UNKNOWN_BINDING_TARGET`, s klíči `unknown`), a **nic** v tomto volání není přiřazeno. `None` vymaže jakýkoli klíč.

### Odrazivost okolí + objektu — `light_sensor_reflectance()`

Dva sdružené DAQ, z nichž jeden směřuje vzhůru k obloze (okolí) a druhý dolů k cíli (objekt), poskytují odrazivost cíle a vegetační indexy na základě svých nejnovějších měření: dvojice „Okolí + Objekt“ v grafickém uživatelském rozhraní a `chloros-cli daq reflectance`, aniž by bylo nutné senzory otevírat podruhé. Oba musí být připojeny a musí vysílat data.

```python
r = chloros_sdk.light_sensor_reflectance("daq-e-def330", "daq-e-54b774")
print(r["reflectance_type"])                       # "Apparent Reflectance" when both are calibrated
for name, band in r["bands"].items():
    print(name, band["rho"] if band["valid"] else band["reason"])
for idx in r["indices"]:
    print(idx["name"], idx["value"])
```

```python
light_sensor_reflectance(
    ambient_sensor_id, object_sensor_id, *,
    bands=None,          # [{"name", "center_nm", "fwhm_nm", "role"}]; role blue / green / red / nir
    preset=None,         # a preset id from daq_band_presets(); default: the suggested one, else "GENERIC"
    expressions=None,    # custom indices: [{"name": "MyNDVI", "expr": "(NIR_850 - Red_660) / (NIR_850 + Red_660)"}]
    builtins=True,       # include the built-in indices
    backend_url="http://127.0.0.1:5000", timeout=30.0,
) -> dict
```

- **Pásma.** Odrazivost každého pásma je pásmovánad okolním zářením. `daq_band_presets(filter_code=None)` uvádí sady pásem: jednu pro každý filtr LATTICE M3C (`RGN`, `OCN`, `NGB`, vlastní pásmový model kamery, takže NDVI z DAQ a NDVI z kamery představují stejnou veličinu) a `GENERIC` pro širokopásmové měření. Jeho `suggested` je předvolba odpovídající `filter_code`, jinak se jedná o filtr první připojené kamery (`suggested_from` určuje, který).
- **Vrací** `bands` (pro jednotlivá pásma `rho`, `valid`, `reason`, `e_ambient`, `e_object`), `indices` (vestavěné), `custom` (každý s `value` nebo `valid: False` a `error`), `invalid_bands`, `reflectance_type`, `preset` a hodnota `timestamps` (`ts_ns`). Rozsah, který okolní senzor nemohl změřit (mimo jeho rozsah, na úrovni šumového prahu), je neplatný, s uvedením důvodu, a každý index, který jej potřebuje, je vynechán a nikdy se nepočítá proti nule.
- **Zdánlivé nebo relativní.** Indexy závislé na měřítku (SAVI, OSAVI, EVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) vyžadují absolutní poměr, proto se vypočítají pouze tehdy, jsou-li oba senzory kalibrovány na výkon (každý má `calibration_bundle_sha`): `"Apparent Reflectance"`. V opačném případě je výsledkem `"Relative Reflectance"`, obsahující pouze indexy nezávislé na měřítku.
- **Dvě různé mřížky vlnových délek** se porovnávají podle jejich překryvu: body okolního senzoru uvnitř dosahu objektu, spektrum objektu na ně interpolované.
- **Vždy dva senzory, nikdy jen jeden.** Pojmenování jednoho senzoru jako obou (stejné ID, v jakémkoli zápisu, který pool akceptuje) vyvolá chybu `ChlorosConnectError`: hodnota senzoru naměřená na sobě samém je 1,0 v každém pásmu, což nic neměří. Backend to rovněž odmítá (`POST /api/daq/indices` odpovídá na `400`, `code` na `SAME_SENSOR`, zatímco `ambient_sensor_id` a `object_sensor_id`, které odesílá SDK, představují jeden sdružený senzor, např. jeho ID a jeho port).

### Sdílení s CLI a dalšími skripty

Fond senzorů DAQ backendu je sdílen všemi klienty CLI a SDK na daném počítači. Pokud je senzor již ve fondu — otevřený příkazem `chloros-cli daq pool-connect` nebo jiným skriptem — volání `connect_daq_sensor(port="COM3")` z Pythonu vrátí vypůjčený identifikátor: jeho `disconnect()` je bez účinku, takže váš skript při ukončení rozsahu neodpojí senzor klientovi, který jej otevřel. Totéž platí pro `sensor_id=` (zadané s `:` nebo `-`, obojí je platné), pro DAQ-E pojmenovaného podle jeho IP adresy, pokud byl otevřen pomocí hostname (nebo naopak), a pro inteligentní detekci s nulovým příznakem, když backend odpoví senzorem, který již držel — blok `with` uvolňuje vždy pouze senzor, který sám otevřel.

GUI Chloros **není** součástí tohoto fondu: jeho panel DAQ provozuje vlastní připojení k senzoru, takže `connect_daq_sensor` si nikdy nepůjčuje senzor, který má otevřený GUI. Nepoužívejte stejný senzor současně z grafického rozhraní a ze skriptu. Senzor typu DAQ-U nebo DAQ-M, který drží grafické rozhraní, se obvykle nepodaří otevřít, ale DAQ-E přijme druhou relaci: příkaz connect ve skriptu změní nastavení sběru dat jednotky pod grafickým rozhraním a jeho `disconnect()` zastaví datový proud jednotky, což ukončí živý náhled v grafickém rozhraní a veškeré nahrávání, které právě probíhá. Nejprve zavřete senzor v grafickém rozhraní.

### PTP Grandmaster — `time_sync_status()`

Backend provozuje PTP grandmaster, ze kterého kamery LATTICE a jednotky DAQ-E čerpají své časové značky. Spouští se na vyžádání: `connect_camera()`, `connect_array()`, `connect_daq_sensor()` a `connect_all()` jej spustí, pokud ještě neběží — včetně připojení, které následně selže, jako například `connect_all()`, který se pokouší připojit k jednotce DAQ-E, která není k dispozici — a totéž platí pro `test_array_sync()` a každé volání, které otevírá kamery pro jeden příkaz (`capture_cameras_transient()`, `start_multi_camera_stream()`, volání pro zarovnání, `capture_live_index()`).

- **Kde se to oznamuje.** Na kabelových síťových adaptérech, ke kterým jsou připojeny kamery (včetně adresy link-local `169.254.x.x`), jinak na adaptéru výchozí trasy, pokud je tento kabelový. Nikdy na Wi-Fi ani na virtuálním adaptéru (Tailscale, WSL): za žádným z nich není žádná kamera ani DAQ-E. Pokud žádný takový adaptér není aktivní, běží s oznamováním na nic (`health` `NO_SOCKETS`) a přiřadí se k adaptéru, jakmile se připojí další kamera nebo DAQ.
- **Po výpadku spojení.** Když adaptér kamery ztratí spojení a znovu ho získá (vytáhnutí kabelu, výměna přepínače, opětovné rozpoznání USB adaptéru), grandmaster sám znovu vytvoří své sokety bez restartu, a znovu zkontroluje všechny připojené DAQ-E, přičemž znovu aktivuje ty, u nichž zůstala mezera. Obnoví se také, pokud všechny kamery a DAQ-E, které obsluhoval, zůstávají 15 s neaktivní.
- **Dosahuje k kabelu?** Přečtěte si `time_sync_status()` (`GET /api/time-sync/status`). `state` pouze udává, že vlákno grandmastera běží; `health` udává, zda jeho pakety odcházejí: `OK` (žádná chyba zjištěna), `STOPPED`, `NO_SOCKETS`, `SENDS_FAILING`, `DEGRADED` (některé adaptéry selhávají) nebo `PEERS_SILENT` (kamery a DAQ-E, které obsluhoval, přestaly odpovídat), s větou v `health_detail`. `announce_sent` / `sync_sent` / `delay_resps_sent` počítají pouze zprávy, které odešly; `announce_send_errors`, `sync_send_errors`, `follow_up_send_errors`, `delay_resp_send_errors`, `last_send_error`, `failing_interfaces` a `last_delay_req_age_s` popisují ty, které neodešly, a `rebinds` / `last_rebind` popisují obnovy. `sync_sends_slow` počítá synchronizace, jejichž odeslání trvalo déle než 200 us, a `sync_send_max_us` je nejdelší; čas odeslání každého Follow_Up se počítá maximálně do 200 us. Grandmaster běží ve vlastním procesu (`own_process`, `process_pid`) s vysokou prioritou ve Windows (`process_priority`), takže zaneprázdněný backend nemůže zpožďovat časová razítka a posouvat hodiny všech kamer najednou; pokud nelze tento proces spustit, běží uvnitř backendu jako dříve (`own_process` false).

```python
st = chloros_sdk.time_sync_status()
if st.get("health") not in (None, "OK", "STOPPED"):
    print("PTP:", st["health"], "-", st.get("health_detail"))
    chloros_sdk.restart_time_sync()
```

| Funkce | Co dělá |
| --- | --- |
| `time_sync_status()` | Stav grandmastera, jeho zdraví, identifikátor hodin, priority a počítadla zpráv (viz výše). `chloros-cli time-sync status`. |
| `time_sync_peers()` | PTP slave, které slyší (kamery, DAQ-E): `peers`, každý z `port_identity`, `last_seen_ip`, `last_delay_req_age_ms`, `delay_req_count`. `time-sync peers`. |
| `time_sync_cameras()` | Každá připojená maticová kamera: `PtpStatus`, `PtpOffsetFromMaster` a `PtpMeanPathDelay` (kamera, jejíž firmware neobsahuje žádné uzly PTP, má pouze sériové číslo). `time-sync cameras`. |
| `restart_time_sync()` | Zastaví a restartuje grandmaster; poté se znovu zkontroluje PTP každého sdruženého DAQ-E (`daq_ptp_rechecked`). Vyvolá `ChlorosConnectError`, pokud se neobnoví. `time-sync restart`. |
| `set_time_sync_priority(priority1=None, priority2=None)` | Jeho priority PTP (0–255, nižší hodnota vyhrává volbu nejlepšího masteru) pro síť s jiným PTP masterem. Vyžaduje, aby byl spuštěn. `time-sync set-priority`. |

Každý z nich vezme `backend_url=` a `timeout=` a spustí lokální backend, který není spuštěn.

---

## Nástroje pro kamery

Co `chloros-cli lattice` dělá kromě připojení a snímání, jako běžné funkce v `chloros_sdk` (modul `chloros_sdk.camera_utils`). Každý z nich volá backendovou trasu, kterou používá příkaz CLI, s pravidly CLI, takže se obě chovají stejně. Přijímají **sériové** číslo kamery, nikoli relaci, a stejná klíčová slova `backend_url=` / `timeout=` jako volání smart-connect; spustí se backend, který není spuštěn, stejně jako to dělá `connect_camera()`. V případě odmítnutí je vyvolána výjimka `ChlorosConnectError` s důvodem uvedeným backendem.

```python
import chloros_sdk as c

c.set_camera_color_profile("214701281", profile="natural")
c.set_camera_highlights("214701281", -40)
print(c.camera_firmware_status()["cameras"])
st = c.network_adapters(estimate=True, cameras=4, pixel_format="BayerRG12")
```

| Funkce | Co dělá |
| --- | --- |
| `camera_color_profile(serial)` / `set_camera_color_profile(serial, profile=, custom_cct_k=)` | Profil barev displeje: `raw`, `linear`, `natural`, `enhanced` nebo `custom_temp` a jeho barevná teplota (omezena na 2000–10 000 K; načteno, pokud je profilem `custom_temp`). |
| `camera_color_adjust(serial)` / `set_camera_color_adjust(serial, saturation=, contrast=, reset=False)` | Sytost a kontrast displeje, každý v rozmezí 0–200 % (100 = beze změny); `reset=True` nastaví obě hodnoty zpět na 100. |
| `camera_highlights(serial)` / `set_camera_highlights(serial, value)` | Sníží jas nejjasnějších a nejbělejších pixelů zobrazení: -100..0 (zkráceno směrem k nule, poté omezováno; 0 = vypnuto). |
| `camera_white_balance_once(serial, settle_s=None)` | Jednorázové vyvážení bílé: kamera krátce streamuje, provede konvergenci a vrátí hodnotu `red_ratio` / `blue_ratio` (`ratios_read` False, pokud je nemůže nahlásit). Zachová vyvážení bílé, které vypočítala; všechna ostatní nastavení, která její otevřený kanál zapsal, se vrátí tak, jak byla nalezena. |
| `set_camera_white_balance(serial, mode=None, red=None, blue=None)` | `mode="auto"` (kontinuální) nebo `"off"`; nebo pevné poměry vyvážení `red` / `blue`, u nichž se automatické nastavení vypne jako první. Poměr, který kamera odmítá, je uveden pod `errors`. |
| `camera_presets()` / `apply_camera_preset(serial, preset)` | Předvolby kamery a jejich použití: kamera zachovává nastavení podle názvů předvoleb (odpověď je `settings`, jak je kamera čte poté, např. snímková frekvence, při které běží; `None` pro pole, které předvolba nepojmenovává), a všechna ostatní nastavení, která její otevřený soubor zapsal, se vrátí do původního stavu. `"default"` resetuje kameru na tovární výchozí hodnoty (tovární nastavení s volně běžící automatickou expozicí, vypnutým omezením propustnosti linky a vypnutou korekcí vad, přičemž se zachová kalibrovaná vyváženost bílé u RGB kamery), které se neukládají do její paměti: odpověď obsahuje `factory_defaults: True` a všechna pole z `settings`. V `camera_presets()` je `default` rovno `{factory_defaults: True, description}`. |
| `camera_filter_models()` / `camera_filter_model_info(name)` | Modely filtrů a podrobnosti o jednom modelu (pásma, kondiční číslo spektrální odezvy). Model filtru kamery se načítá z řetězce modelu; nenastavuje se ručně. |
| `camera_firmware_status(serial=None)` | Firmware jednotlivých kamermá svůj firmware: `current_version`, `available_version`, `update_offered`, `needs_minimum_update` a `minimum_firmware`. |
| `update_camera_firmware(serial, force=False)` | Nahrajte firmware této verze Chloros do jedné kamery: obnovení továrního nastavení, nahrání, restart, přepsání základních nastavení Chloros; kalibrace kamery zůstane zachována. Trvá to několik minut; kamera musí zůstat zapnutá. `force=True` znovu nahraje firmware do stávající kamery. |
| `camera_power_probe(serial)` | Napájecí a teplotní uzly kamery a její teplota. |
| `sleep_camera(serial)` / `wake_camera(serial)` | Nízkoenergetický klidový režim kamery (zůstává v režimu spánku, dokud není probudena: je zapnut limit propustnosti linky (`DeviceLinkThroughputLimitMode` `On`) na minimální hodnotě, je zapnut úsporný režim, pokud jej kamera má), s teplotou před a po; probuzení vrátí spojení do stavu, v jakém ho našel režim spánku (limit, režim a zpoždění paketů `GevSCPD`, které kamera udržuje na hodnotě 0, dokud je limit zapnutý; záznam se uchovává na tomto počítači, takže ho najde i probuzení z pozdějšího backendu), nebo v případě kamery, kterou tento počítač neuspával, ale která vypadá, že spí (její limit je nastaven na minimální hodnotu), s vypnutým limitem a hodnotou `GevSCPD` ponechanou na 0 při 80 (tovární nastavení); limit nastavený nad minimální hodnotou se ponechá beze změny. Uchovává se pouze stav napájení; všechna ostatní nastavení, na která se otevřený soubor zapisuje, se vrátí do původního stavu. |
| `calibrate_camera_with_target(serial, filter=None, attempts=30, settings=None)` | Snímá snímky, dokud není v zorném poli kalibrační cíl MAPIR, a přizpůsobí koeficienty pro jednotlivé kanály podle jeho panelů: `target_id`, `panel_dns`, `coefficients`. |
| `network_adapters(estimate=False, cameras=1, pixel_format="BayerRG8")` | Ethernetové adaptéry tohoto počítače pro GigE Vision (MTU, rychlost spojení, jumbo rámce) a jakékoli `issues`; `estimate=True` přidává snímkovou frekvenci a šířku pásma, které by kamery `cameras` dosahovaly. |
| `fix_network_settings()` | Použije nastavení adaptéru GigE Vision na kamerový adaptér tohoto počítače. Změní nastavení adaptéru a funguje pouze v případě, že je Chloros spuštěn jako správce; v opačném případě se spustí s protokolem o zjištěných nálezech. |
| `apply_vignette_correction(input_dir, output_dir, lens_model=None)` | Proveďte korekci vinětace snímků ve složce, přičemž každý snímek bude mít vlastní mapu rovinného pole objektivu: `processed`, `lens_models` (objektiv, pro který byl každý soubor korigován), `skipped` (s odůvodněním). Bez `lens_model` se objektiv každého souboru odvozuje z jeho EXIF Model; soubor `lens_model` se vztahuje na soubory, jejichž pole „Model“ neuvádí žádný fotoaparát, a soubor, jehož pole „Model“ je s ním v rozporu, je přeskočen. Pouze soubory. |
| `read_dls(mac, filter=None, camera_serial=None)` | Jedno měření z DLS (senzor sestupujícího světla přes Bluetooth): spektrum, intenzita záření v jednotlivých pásmech a hodnoty pásem prostřednictvím vlastní odezvy fotoaparátu, pokud je zadán `camera_serial`. `discover_daq_sensors(transports=["ble"])` vyhledá MAC. |
| `probe_system(pixel_format="BayerRG12", discover=True)` / `system_info()` | Hardware hostitele pro kamery a jeho doporučená konfigurace (`report_text`) a CPU / GPU / runtime PyTorch systému. |

- **Některé potřebují kameru pro sebe.** `camera_white_balance_once()`, `apply_camera_preset()`, `sleep_camera()` / `wake_camera()`, `calibrate_camera_with_target()` a `update_camera_firmware()` otevírají kameru. Pokud ji právě drží Chloros (je připojená v aplikaci, s `connect_camera()` jako člen pole nebo je právě používána jiným příkazem, jako je například test pole nebo zachycení otevírající jeho kamery), backend to odmítne s chybou HTTP 409 `CAMERA_HELD` (jak v `code`, tak v `error_code`), s uvedením způsobu, jak jej uvolnit. Nastavení barev, zvýraznění, `camera_power_probe()` a čtení fungují na připojené kameře.
- **Barva, zvýraznění a vyvážení bílé jsou určeny pro RGB kamery.** U jednopásmové monochromatické (M3M) kamery tyto nastavovací funkce nic nemění a vrací hodnotu `{"skipped": True, "reason": ...}`, stejně jako CLI. Jedná se o nastavení zobrazení: živý náhled a snímky na úrovni displeje, nikdy se nejedná o radianci ani odrazivost.

---

## Přechodová pole, profily zarovnání a indexy

Co dělá `chloros-cli lattice` bez trvalého připojení, stejně jako funkce v `chloros_sdk` (modul `chloros_sdk.array_utils`): `continuous`, `multi-capture`, `multi-stream`, `multi-test`, `multi-detect`, `analyze-array`, `align-calibrate`, `align-apply`, `align-stream`, `align-info`, `align-reorder` a `index`. Každý z nich odesílá požadavek, který odesílá příkaz CLI, s výchozími nastaveními CLI.

```python
import chloros_sdk as c

prof = c.calibrate_alignment_profile(["214000535", "214000551", "214000556"],
                                     "D:/rig/alignment_profile.json", method="orb", model="rigid")
print({sn: t["reprojection_error"] for sn, t in prof["transforms"].items()})

c.capture_aligned_multiband("D:/rig/alignment_profile.json", "D:/rig/out")
idx = c.capture_live_index("D:/rig/alignment_profile.json", preset="NDVI",
                           output_dir="D:/rig/ndvi", capture_level="reflectance")

with c.start_multi_camera_stream(["214000535", "214000551"], "D:/rig/stream", fps=5, count=50) as job:
    print(job.wait()["final_stats"])          # waits for the 50 groups, then closes the cameras
```

- **Samy otevírají kamery.** Každý příkaz otevře své kamery, provede svou činnost a kamery zavře, takže potřebuje kamery, které Chloros nedrží (připojené v aplikaci, pomocí `connect_camera()` nebo jako člen pole); backend to odmítne s chybou HTTP 409 `CAMERA_HELD` a sdělí, jak kameru uvolnit. Dočasné otevření trvá chvíli (asi 40 s pro několik kamer). Pro opakované snímání je je `connect_array()` nejrychlejší způsob a ušetří čas u každého typu exportu, zatímco dočasné snímání ukládá obraz po odstranění bayesovského efektu. Výjimkou je `calibrate_alignment_profile()`: u připojeného pole těchto kamer řeší problém z vlastních snímků pole (`used_live_array`).
- **`serials`** jsou kamery, které se mají otevřít, přičemž první z nich spustí spoušť; `None` otevře všechny kamery v síti.
- **`settings`** je slovník přepisů nastavení kamer, pole, která nastavují příznaky v CLI: `exposure_time` (µs), `gain` (dB), `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `frame_rate`, `gev_packet_size` a `preset` (předvolba kamery podle názvu). Zadaná expozice nebo zesílení vypne automatické řízení, stejně jako příznaky.
- **`gpu`**: `True` vynutí warp na GPU, `False` na CPU (`--gpu` / `--no-gpu`); `None` ponechává volbu na backendu.
- **`keep_settings`**/**`clear_settings`** (všechny funkce, které otevírají kamery, výchozí nastavení je `False`): na konci se každá kamera vrátí do stavu, v jakém byla nalezena; `keep_settings=True` místo toho uloží nastavení běhu do vlastní paměti kamery (volný běh), `clear_settings=True` tuto paměť resetuje na tovární nastavení — viz [Paměť kamery](#camera-memory--keep_settings--clear_settings). Odpověď (pro úlohu, v případě `stop()`) obsahuje `camera_memory`.
- **Cesty jsou cesty backendu**: relativní cesty se převádějí na absolutní vzhledem k vašemu pracovnímu adresáři, pokud se backend nachází na tomto počítači. Funkce pro práci se soubory profilů zarovnání (`alignment_profile_info()`, `reorder_alignment_profile()`) čtou soubor na tomto počítači, stejně jako to dělá CLI.

| Funkce | Co dělá |
| --- | --- |
| `start_camera_continuous(serial, output_dir="output", *, format="tiff", jpeg_quality=95, queue_depth=64, settings=None)` | Nepřetržitě nahrává z jedné kamery, dokud není zastavena. Vrací hodnotu typu `CliJob`. Snímky pořízené během běhu `tiff` tvoří soubor typu `raw`, do kterého se zapisují snímky z dané kamery (jedna mozaiková rovina v hodnotách senzoruvlastních hodnotách, nekomprimované); `png` a `jpg` obsahují 8 bitů, takže tyto běhy představují snímky po demosaicování určené k prohlížení. Vázané nebo sdružené DAQ zapisuje jeden soubor `.daq` na každý snímek v běhusložce `daq/` (`daq_files` ve statistikách). Každý snímek je zapsán jednou s úplnými metadaty (model, čas pořízení, objektiv, pásma, kalibrace na surovém snímku); `stop()` se vrátí, jakmile jsou zapsány všechny již zachycené snímky (až 25 s), a teprve poté se kamera uzavře. Kamera ponechaná v pohotovostním režimu pro hardwarový spouštěč (`TriggerMode` `On`) je pro daný běh nastavena na volný provoz (odpověď na start `trigger` udává, co bylo zjištěno). Při zastavení se kamera vrátí do stavu, v jakém byla nalezena – včetně nastavení spouště a všech nastavení, která běh změnil – pokud není zadán kód `keep_settings=True`, který uloží nastavení běhu do vlastní paměti kamery (volný běh), nebo `clear_settings=True`, který tuto paměť vynuluje. Běh, který nezaznamená žádný snímek, se po 10 s sám ukončí pomocí `stop_reason` a `no_frames` a důvod je uveden v `last_error`, a jeho `stop()` tento důvod vrací jako `error`. Snímky vynechané z důvodu, že zapisovače nestíhaly, se počítají, přičemž `drop_reason` udává důvod. Pro kameru, kterou drží Chloros, použijte `CameraSession.capture_repeated()`. `lattice continuous`. |
| `start_multi_camera_stream(serials=None, output_dir="output", *, fps=3.0, count=None, format="tiff", jpeg_quality=95, settings=None)` | Synchronizované skupiny snímků v proudu na `fps`, pro skupiny `count` nebo dokud není zastaveno. Vrací `CliJob` (jeho `start_reply` má `n_cams` a přenos `pacing`); jeho konečné statistiky zahrnují počet zachycených, zapsaných a vyřazených skupin, soubory, které zapisovač nemohl zapsat, a skupiny, pro které kamera nic nedodala. `lattice multi-stream`. |
| `capture_cameras_transient(serials=None, output_dir="output", *, format="tiff", jpeg_quality=95, settings=None)` | Jedno synchronizované zachycení, jeden obraz po odstranění bayesovského kódu na kameru: `results` (`filepath`, `serial`, `frame_id`) a `pacing`. Objeví se hlášení o chybějící kameře, které nejprve uvádí počet uložených souborů. `lattice multi-capture` bez připojeného pole. |
| `test_array_sync(serials=None, count=10, ptp_settle_s=None, timeout=None)` | `count` synchronizované snímky s pevnou expozicí: `sync_groups`, `master`, jeden řádek `group_timestamps` na jeden spouštěcí impuls (kamera, která jeden snímek vynechala, v daném řádku chybí) a `sync`: posun každé druhé kamery vůči hlavnímu snímku, spoušť po spoušti (`offsets_us`: pro každou kameru `mean_us`, `min_us`, `max_us`, `jitter_us`, `groups`), na JEDNOM taktu (`clock_domain`: `ptp`, nebo `device` — PTP vypnuto, vlastní časovač provozní doby každé kamery, takže její posun odpovídá okamžiku spuštění a synchronizaci měří pouze kolísání). Spouštěč, jehož časová razítka kombinují hodinový signál zařízení a PTP (PTP vypnuto nebo stále v synchronizaci), je vynechán a započítán (`groups_mixed_domain`), stejně jako ten, u kterého jsou hodiny PTP stále více než `agree_within_us` od hlavního (`groups_clocks_disagree`); `notes` to potvrzují. `clock_offset_mean_us` / `clock_offset_jitter_us` představují největší průměr a kolísání na jednu kameru. `success` je false, stejně jako `error`, pokud došlo ke zmeškání spouště, pokud méně než dvě spouště synchronizují všechny kamery na jeden takt, nebo pokud se takty nikdy neshodují (`sync.problems`); řádky se vrátí v obou případech. Spustí PTP grandmaster, pokud není spuštěn (`time_sync` uvádí `grandmaster_running` a `grandmaster_started_by_this_test`), zapne PTP u každé kamery (stejně jako to dělá `connect_array()`) a před prvním spouštěčem počká, až se PTP hodiny každé kamery synchronizují s grandmasterem a ustálí se: `PtpStatus` Slave na jednom grandmasteru, s odchylkou do 100 us a stabilní po dobu 3 s, přičemž každá kamera je takto synchronizována, bez přerušení, po dobu 120 s — přibližně 2 minuty od zapnutí. Hodiny, které se právě synchronizovaly, vykazují odchylku až ~200 us po dobu asi 2 minut poté, co se načte stav „Slave“, a vlastní `PtpOffsetFromMaster` kamery to nezobrazuje; za 120 s se kamery dostanou do odchylky ~10 us od ustáleného času. Kamera, která zůstala synchronizovaná od předchozího testu, při kterém zůstalo PTP zapnuté (jak to viděl tento backend), počítá od této synchronizace, takže test prováděný bezprostředně po předchozím nemusí znovu čekat; kamera, kterou test zapne, nebo kamera nalezená se zapnutým PTP, u které tento backend nezaznamenal synchronizaci (zapnutá jinde nebo před restartem backendu), počká svých 120 s, a v poli obou kamer, které se právě synchronizovaly, stále čekají na svou dobu. Kamera, která opustí režim Slave, překročí 100 us nebo sleduje jiného grandmastera po dobu 3 měření za sebou (1,5 s) během čekání, začne odpočítávat znovu; jedno nebo dvě taková měření se zaznamenají (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) a odpočítávání pokračuje, přičemž ID grandmastera, které nelze přečíst, se považuje za neznámé, nikoli za změnu. Pokud se počítání všech kamer restartuje najednou (během jedné sekundy), znamená to, že se posouvá čas grandmastera, nikoli čas kamer: protokol backenduuvádí `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` je uvádí v záznamu `grandmaster_steps` a čekání, jehož čas poté vyprší, začíná chybový záznam s hodnotou `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Limit pro ni není prodloužen. `ptp_settle_s` (0–600; `0` pro čekání bez ustálení) nastavuje dobu ustálení pro daný běh; jiná hodnota je odmítnuta (kód HTTP 400). Čekání je omezeno na dobu vypořádání plus 90 s (výchozí hodnota 210 s, minimálně 90 s) a volání čeká na backend `timeout` s: ve výchozím nastavení 240 plus doba vypořádání (360 s), stejně jako u `chloros-mcp` a `test_array_sync`. `ptp_lock` má `converged`, `waited_s`, `timeout_s`, `settle_s`, `locked_s` (jak dlouho jsou všechny kamery zamčené a stabilní), `found_settled` (kamery, které již byly zamčené po dobu ustálení v okamžiku zjištění) a pro každou kameru zvlášť: `status`, `offsets_ns`, `slave_s` (jak dlouho je synchronizována; None: není synchronizována) a `remaining_s` (None, pokud se u žádné kamery nezapne PTP). Protokol backendu obsahuje každých 5 s řádek s informacemi o stavu každé kamery, jak dlouho je synchronizována a jak dlouho jí ještě zbývá. Každá kamera se zapíná s vypnutým PTP; od té chvíle se hodiny synchronizují přibližně za 10 s. Poté je každá kamera vrácena do stavu, v jakém ji test našel – její PTP, automatická expozice a zesílení jsou během testu nastaveny na Vypnuto (`ExposureAuto` / `GainAuto` a ruční `ExposureTime` / `Gain`), a ostatní, pokud není nastaveno `keep_settings=True` — tedy kamera, u které zůstane PTP zapnuté (`PtpEnable=True`, nebo běh s `keep_settings=True`) zůstává zamčená, a jakmile tento backend zjistí, že je zamčená po dobu ustálení, pozdější test na ni nečeká. Pokud se hodiny neustálí, je `success` před jakýmkoli spouštěčem nepravdivé a `error` určuje, co se má udělat: kamera, která se nikdy nezamkla (`not_slave`), znamená, že grandmaster k ní nedosahuje (`time_sync_status()`); ta, která neustále vypadávala, je pojmenována podle toho, jak dlouho byla zamčena (`remaining_s`); hodiny se stále sbližují (`far`): zkontrolujte `time_sync_status()` a abyste zajistili jejich uzamčení před testem, zapněte PTP a nechte jej zapnutý (`PtpEnable=True` na každé kameře, nebo `keep_settings=True`; test obnoví zjištěné nastavení, takže zůstane zapnutý), poté test spusťte. `lattice multi-test`. |
| `detect_array_master(line=None)` | Kamery na synchronizační lince a výchozí hlavní kamera: `master_serial`, `slave_serials`, které byly otestovány pomocí `line`. `lattice multi-detect`. |
| `analyze_array_models(camera_models=None, *, width=2048, height=1536, n_active=None, force_tier=None, binning=1, packed="auto", pixel_format=None)` | Jaká to řada těchto modelů kamer (názvy produktů, `"LATT-M3C-L87-FOCN"`), by se v síti tohoto počítače objevilo: `tier`, `achievable_fps_max`, rozpočet na kabeláž (`max_cams_collision_safe`, `oversubscribed`), `cam_caps` a `fix_suggestions`. Ve výchozím nastavení se zobrazí modely nalezených kamer; žádná kamera není otevřena. Informace o kamerách, které vlastníte, včetně nastavení, která by připojení používalo, najdete v [`analyze_array_network()`](#network-analysis-helper). `lattice analyze-array`. |
| `calibrate_alignment_profile(serials=None, save_path="alignment_profile.json", *, frames=3, method="orb", model="rigid", reference_serial=None, checkerboard=None, control_points=None, preview=False, vignette=False, roi=None, mask_path=None, rms_threshold_px=None, per_cam_overrides=None, settings=None, ...)` | Vypočítejte profil zarovnání z živých snímků a uložte jej. `method`: `orb`, `akaze`, `phase`, `checkerboard` (spolu s `checkerboard="9x6"`, vnitřní rohy) nebo `manual` (spolu s `control_points={serial: [[x, y], ...]}`, také referenční kamera). `model`: `translation`, `rigid`, `affine` nebo `homography`. Podrobnější možnosti představují příznaky CLI: `max_features`, `ratio_threshold`, `matcher` (`bf` / `flann`), `knn_k`, `prefilter` (`none` / `gradient` / `clahe` / `blur` / `hist_match`), `ransac_threshold_px`, `ransac_max_iters`, `ransac_confidence`, `averaging`, `lock_rotation`, `lock_scale`, `lock_axis`; `per_cam_overrides` je `{serial: {option: value}}`. `rms_threshold_px` odmítá uložit horší řešení. Vrací `save_path`, `output_width` / `output_height`, `crop_roi`, `reference_serial`, `transforms` (pro každou kameru `reprojection_error`, `num_matches`) a `preview_path`. `lattice align-calibrate`. |
| `capture_aligned_multiband(profile_path, output_dir="output", *, serials=None, format="tiff", bit_depth=None, gpu=None, no_crop=False, bands=None, order=None, vignette=False, per_band=False, per_camera=False, interpolation="linear", border_mode="constant", border_value=0, settings=None)` | Jeden zarovnaný vícepásmový snímek v rámci profilu: `files` (vícepásmový soubor TIFF s jeho `bands` a velikostí, plus jeden na pásmo s `per_band` a jeden na kameru s `per_camera`) a `vignette_corrected`. `bands` vybírá a `order` řadí pásma podle názvu (seznam nebo řetězec oddělený čárkami). `lattice align-apply`. |
| `start_aligned_stream(profile_path, output_dir="output", *, serials=None, fps=3.0, count=None, ...)` | Srovná vícepásmové snímky podle streamu (stejné možnosti warpování jako u `capture_aligned_multiband()`), pro `count` nebo dokud není zastaveno. Vrací `CliJob`; jeho konečné statistiky zahrnují `frames_written` a případné `frame_groups_incomplete`. `lattice align-stream`. |
| `alignment_profile_info(path)` | Soubor profilu shrnutý jako `align-info` jej vytiskne: `name`, `created`, `method`, `reference_serial`, `sensor_resolution`, `output_width` / `output_height`, `crop_roi`, `camera_serials`, `transforms` (na jednu kameru `model`, `reprojection_error`, `num_matches`, `reference`), `camera_models`, `layer_order` (pro každou kameru `band_names`, `wavelengths_nm`, `enabled`); `profile` je celý soubor. |
| `reorder_alignment_profile(path, order=None, enable=None, disable=None)` | Změna profilového souboru na místě: `order` jsou názvy pásem (každý posune vrstvu své kamery nahoru, v daném pořadí), `enable` / `disable` jsou sériová čísla. Vrátí nový `layer_order`. `lattice align-reorder`. |
| `index_catalog()` | Přednastavené indexy (`presets`: každý z `name`, `channels`, `formula`) a barevné `gradients`. `lattice index --list-presets` / `--list-gradients`. |
| `compute_index(input_path, *, preset=None, formula=None, channel_map=None, output=None, capture_level="unknown", output_format="all", colorize=False, gradient="RdYlGn", vmin=None, vmax=None, percentile=None, bg_mode="clip")` | Index pro zarovnaný vícepásmový soubor TIFF: `preset` (`NDVI`, `NDRE`, ...) nebo `formula` nad symboly `channel_map` mapuje na pásma (`{"NIR": "NIR_850", "Red": 0}`, podle názvu nebo indexu). Přednastavenýsymbol předvolby `channel_map` vynechává a vybírá jedno pásmo souboru, jehož název tímto symbolem začíná (`red` -> `Red_660`, `nir` → `NIR_850_RGN`), pojmenovaný samotným souborem (jeho seznamem pásem nebo `Chloros:Band{R,G,B}Name` v záznamu či produktu Chloros), takže NDVIu na snímku odrazivosti Chloros není třeba `channel_map`; dvě pásma odpovídající jednomu symbolu jsou odmítnuta, nikoli odhadnuta. `capture_level` udává, co pásma obsahují: `reflectance` nebo `radiance` činí hodnoty kvantitativními; nekalibrovaný stack (`align-apply`) poskytuje kvalitativní hodnoty; `"unknown"` to načte ze souboru, včetně `Chloros:ProcessingLevel` ze souboru Chloros. Vrací `stats` (tvar, rozsah 1–99. percentilu, rozsah LUT, použitý `capture_level`) a `files` zapsaný vedle vstupu (`<input stem>_<preset>.tif`, `..._LUT.png`) nebo na `output`, což je cesta: v případě příponu obrázku jsou soubory pojmenovány podle svého kořene (`output="D:/out/ndvi_FRGN.png"` zapíše `ndvi_FRGN.tif` a `ndvi_FRGN_LUT.png`), bez ní se jedná o předponu (`D:/out/ndvi` → `ndvi.tif`, `ndvi_LUT.png`); načtěte cesty od `files`. Obsahují záznam o fotoaparátu ze vstupu (značka / model / sériové číslo, GPS, objektiv, čas pořízení v UTC, identifikační údaje kalibrace, ale nikdy kalibraci raw-DN), `Chloros:ProcessingLevel=index` a index jako svůj jediný kanál (`XMP-Camera:BandName`, TIFF i LUT PNG). Pouze soubory; bez kamery. `lattice index --input`. |
| `capture_live_index(profile_path, *, preset=None, formula=None, channel_map=None, serials=None, output_dir="output", output_prefix="index", capture_level="reflectance", save_multiband=False, ...)` | Pořídit, kalibrovat (`reflectance` nebo `radiance`; `raw` tento krok přeskočí), zarovnání a výpočet indexu v jednom kroku; `save_multiband=True` uchová kalibrovaný stack, který může později převzít `compute_index()`. Vrací `stats` (spolu s `serials`, jehož pásma použil) a `files`. Otevře `serials` (první spustí spoušť), jinak kamery profilu, u nichž je povolena položka `layer_order`, nejprve referenční (otevře se i s deaktivovanými pásmy); kamera s deaktivovanými pásmy se nikdy nekalibruje. `channel_map` je potřebný pouze pro symbol, na který zásobník nemůže odpovědět, jak je uvedeno v `compute_index()`: symbol předvolby převezme jedno povolené pásmo, jehož název tímto symbolem začíná, což se zkontroluje před otevřením jakékoli kamery (pokud neodpoví žádná nebo odpoví více kamer, jsou odmítnuty a pojmenovány); vlastní `formula` je potřebný pro každý symbol. `index.tif` a LUT nesou kameru zásobníku, čas snímání (expozici referenční kamery, UTC) a objektiv, a oba označují index jako své jediné pásmo (`XMP-Camera:BandName`, např. `NDVI`: barevný soubor PNG uvádí jako název index, nikoli své barevné kanály). `lattice index --live`. |

### `CliJob`

Identifikátor, který vrací všechny tři datové toky. Úloha běží na backendu, který spravuje příslušné kamery, dokud není zastavena.

| Člen | Popis |
| --- | --- |
| `stats(timeout=10.0)` | `running`, `state`, `stats` (snímek uložen / zapsané, vynechané, `fps`, `elapsed`, chyby zápisu, `stop_reason`; u nepřetržité úlohy také `frames_grabbed` a `frames_queued` – každý zachycený snímek je zapsán, zařazen do fronty, vyřazen nebo selhal – a `drop_reason`, `writers` a `write_ms`, průměrná doba zápisu jednoho snímku) a `output_dir`. `state` je `running`, `ended` (sám se zastavil a dosud nebyl zastaven), `stopping` (zastavený proces vypisuje svou frontu: aktuální statistiky) a `stopped_by`) nebo `finished` (jeho konečné statistiky, `error` a `stopped_by` zastaveného úlohy). Záznam o zastavené úloze se uchovává po dobu 10 minut (maximálně posledních 32), a to pro každého klienta: jiný skript nebo `chloros-mcp` čte stejný stav a konečné statistiky. Úloha `chloros-cli` byla spuštěna (`lattice continuous`, `multi-stream`, `align-stream`) je zastavena backendem, když se CLIpo 30 sekundách přestane hlásit – byla ukončena, její počítač je v režimu spánku – s `stop_reason`, `client_gone` a `stopped_by` na backendu; čtení jeho statistik z tohoto místa jej neudrží v chodu. Úloha spuštěná z SDK běží, dokud není zastavena. Každé spuštění odešle svůj `timeout` jako termín backendu (`client_deadline_s`): kamera, která se otevře až po něm (ta se ještě spouští), je vrácena a uzavřena a není spuštěna žádná úloha, a volání vyvolá `ChlorosConnectError` (`CLIENT_DEADLINE_PASSED`). Kamera, kterou drží úloha, je odmítnuta při každém dalším otevření (`connect_camera()`, další spuštění, přechodné zachycení) s chybou HTTP 409 `CAMERA_HELD`, uvádějící název úlohy a jméno osoby, která ji spustila. |
| `stop(timeout=30.0)` | Zastavte ji a zavřete její kamery: `final_stats`, `output_dir` a `error`, pokud se zastavení neprovedlo bezchybně. Druhé volání vrátí stejnou odpověď, stejně jako zastavení z jiného klienta: to, které bylo odesláno, zatímco první ještě zapisuje svou frontu, na něj čeká (až 25 s; po uplynutí této doby dojde k výjimce a následují konečné statistiky na `stats()`). |
| `wait(timeout=None, poll_s=0.5)` | Počkejte, až se proces ukončí sám (dosáhne `count`, zaplní se disk, dojde k chybě), poté jej ukončete pomocí `stop()`, abyste získali konečné počty. `timeout` vyvolá výjimku `TimeoutError`, zatímco úloha stále běží; úloha bez `count` skončí až po zastavení. |
| `job_id`, `kind`, `output_dir`, `start_reply`, `stopped` | Jeho ID a typ (`continuous`, `multi-stream`, `align-stream`), kam zapisuje, odpověď počáteční trasy a zda byl spuštěn `stop()`. |

Použijte jej jako správce kontextu, aby se při opuštění rozsahu zastavil.

---

## Projektová složka a recepty

**Projektová složka** je složka, ve které se vytvářejí nové projekty, sdílená aplikací Chloros, CLI a SDK (modul `chloros_sdk.system_utils`; `chloros-cli project-folder`):

```python
print(chloros_sdk.project_folder())                # the configured folder, else the default
chloros_sdk.set_project_folder("D:/Flights")       # created if missing
chloros_sdk.reset_project_folder()                 # -> {"path": <default>, "was_set": True}
```

Pokud nakonfigurovaná složka již neexistuje, použije se výchozí složka (`Documents/MAPIR/Chloros Projects`), stejně jako v aplikaci. Relativní cesta zadaná pro `set_project_folder()` se bere z vašeho pracovního adresáře, pokud je backend na tomto počítači.

**Recept** je seznam akcí ve formátu YAML / JSON, který `chloros-cli project run PATH RECIPE` spouští na uloženém projektu. `run_project_recipe()` jej spouští pomocí CLIvlastním spouštěčem (nainstalovaným s SDK), takže se jeden recept chová stejně v obou případech:

```python
r = chloros_sdk.run_project_recipe("D:/campaign/morning.yaml", "D:/Projects/Field_A")
print(r["ok"], r["actions"])
print(r["log"])                                    # what the CLI would have printed

chloros_sdk.run_project_recipe({"actions": [{"apply": {"name": "FrontLeft", "settings": {"exposure_us": 5000}}},
                                            {"wait": "2s"},
                                            {"capture": {"name": "FrontLeft", "output": "pose_a/", "format": "tiff"}}]},
                               "D:/Projects/Field_A")
```

```python
run_project_recipe(recipe, project_path, *, dry_run=False, echo=False,
                   backend_url="http://127.0.0.1:5000") -> dict
```

- **`recipe`** je soubor `.yaml` / `.yml` / `.json`, nebo samotný recept: slovník s `actions` (`apply`, `capture`, `burst`, `stream`, `sensor`, `wait`) a volitelně `overrides` a `stop_on_error` (výchozí hodnota true: spuštění se zastaví při první akci, která selže). Akce jsou definovány v [Recipe DSL](cli-reference.md#recipe-dsl) v referenci CLI; relativní `output` se vyhodnocuje vzhledem k pracovnímu adresáři vašeho skriptu.
- **Vrací** `ok` (všechny akce byly úspěšné), `actions` (kolik), `log` (výstup spouštěče; `echo=True` jej také vypisuje během běhu) a `validation`. Chybějící nebo neparsovatelný soubor receptu vyvolá chybu `ChlorosConnectError`.
- **Zkontrolováno před spuštěním.**`validation` je `{"ok", "errors", "warnings", "steps"}`: každá akce musí mít jako klíč přesně jedno známé sloveso (`{"capture": {"name": ...}}`, nikoli `{"action": "capture"}` nebo `{"type": "capture"}`), které označuje kameru, pole nebo senzor, který projekt obsahuje, a to správného typu. Recept s chybou neprovede**žádnou** akci (`ok` false, chyby v `log`), bez ohledu na to, co uvádí `stop_on_error`. Nepoužité klíče, a každé pole nebo světelný senzor, který recept ovládá a který není v danou chvíli připojen, představují `warnings`. Recept zařízení nepřipojuje: připojte je nejprve v projektu (`ChlorosProject.connect_all()`, `chloros-cli project connect`, nebo nástroj MCP `connect_project_devices`). `dry_run=True` nic nespouští a vrací `{"recipe": ..., "validation": ...}`.
- V rámci procesu běží vždy pouze jeden recept. Recept v jazyce YAML vyžaduje knihovnu PyYAML.

---

## Automatizace projektu — `ChlorosProject`

Uložený projekt Chloros je složka obsahující `cameras.json` + `sensors.json` + `project.json`. `open_project(path, relink=None)` načte manifest (`relink`: složka, ve které se nyní nacházejí chybějící zdrojové soubory přesunutého projektu, používaná při načítání backendem — `process()` — viz **Přesunutý projekt** pod `ChlorosLocal`), a `connect_all` uvede do provozu všechny uložené kamery, pole a světelné senzory s jejich uloženými nastaveními — ve stejném hardwarovém stavu, jaký by vytvořilo grafické uživatelské rozhraní. Světelné senzory se připojují přes [pool senzorů DAQ](#daq-sensor-sessions), takže potřebují spuštěný backend (viz [`SensorHandle`](#sensorhandle)).

### Minimální příklad

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Light sensors are addressable by name OR slot_id
reading = proj.sensors["Sky"].read()   # (spectrum, is_saturated, ...) or None

# One frame from every camera and array, one spectrum from every sensor
proj.capture_all("./out")

proj.disconnect_all()
```

Nebo jako správce kontextu:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### Metody `ChlorosProject`

| Metoda | Popis |
| --- | --- |
| `connect_all(*, cameras=True, arrays=True, sensors=True, verbose=False, align=None, packed=None)` | Vyhledá a připojí všechny uložené kamery, pole a světelné senzory. Vrátí zprávu o připojení pro každou třídu (`{"cameras": {...}, "arrays": {...}, "sensors": {...}}`, každá s `connected`, `missing` a `errors`; `arrays` také obsahuje `alignment`). Pokud backend naslouchá na adrese `127.0.0.1:5000`, jsou uložené kamery a pole nalezeny a připojeny prostřednictvím něj. Pokud backend neexistuje, tento proces je vyhledá a řídí sám prostřednictvím `lattice_sdk`, který vyžaduje runtime SDK pro Arenu a OpenCV (`cv2`) — balíček SDK z balíčku wheel neinstaluje ani jedno; pokud se to nepodaří, všechny uložené kamery a pole se zobrazí pod `errors` s uvedením důvodu a `missing` zůstane prázdný. Backend, který běží, ale odmítá SDK (odhlášen: HTTP 401; plán pod Copper: 403), není považován za „žádný backend“: každá uložená kamera, pole a světelný senzor jsou uvedeny pod `errors` s důvodem backendu, a v tomto procesu není nic otevřeno. Každé volání znovu kontroluje backend, takže `connect_all()` se po přihlášení nebo po spuštění Chloros připojí ke stejnému objektu projektu. Uložená matice, která již v backendu existuje se stejnými kamerami (z grafického rozhraní nebo z `chloros-cli project connect`), je připojena tak, jak je, bez opětovného použití jejích uložených nastavení, a `disconnect_all()` jej nechá běžet. Světelné senzory vždy procházejí přes backend (viz [`SensorHandle`](#sensorhandle)). Nikdy nespouští backend. |
| `disconnect_all()` | Vše ukončit. Nejprve se uvolní všechna zařízení; poté, pokud backend nepotvrdil uvolnění kamery, pole nebo světelného senzoru (vypršel časový limit ukončení nebo bylo odmítnuto), vytvoří se pro každé z nich jeden `ChlorosConnectError`. Pokud zařízení již neexistuje nebo pro které neběží backend, mlčí. Pole nebo světelný senzor, který měl otevřený jiný klient, zůstává v provozu. Ukončení projektu `with` se nikdy nespustí. |
| `capture_all(output_dir=".")` | Jeden snímek z každé připojené kamery + pole, a jedno spektrum (`SensorHandle.read()`) z každého připojeného světelného senzoru, pořizované postupně (nejprve kamery, poté pole, poté senzory), nikoli najednou. |
| `stream(camera, overlays=False, fps=10.0)` | Generátor vytvářející snímky BGR `numpy` z pojmenované kamery (nebo pole). `overlays=False` čte snímky s identifikátorem`grab()` na úrovni `debayered`: prostřednictvím backendu, pokud je projekt připojen přes něj, jinak přímou smyčkou snímání `lattice_sdk` (pole generují slovníky `{serial: frame}`). `overlays=True` směruje přes `ChlorosLocal.camera_stream()` → MJPEG kanál `/api/camera/<serial>/stream-annotated` backendu, přičemž uložený blok kamery je předán jako parametry dotazu. Vyžaduje režim backendu, OpenCV (`cv2`) a **samostatnou kameru**: kamera v přímém režimu vyvolá chybu `RuntimeError` (backend nemůže získat kameru, kterou vlastní tento proces) a pole vyvolá chybu `NotImplementedError` (složené překryvy pro každou kameru — streamujte člen podle jména). Ekvivalent pro jednorázové použití: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Spustit zarovnání na každém aktuálně připojeném poli. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Spustit kalibrační / indexovací pipeline na snímcích projektu (zahrnuje `ChlorosLocal.process`; tyto čtyři jsou **jediné** přijímané klíčové argumenty — `indices=` atd. vyvolá výjimku `TypeError`; nastaví indexy pomocí `ChlorosLocal.configure()`). Odloženě vytvoří `ChlorosLocal()`, který automaticky spustí backend. |

Atributy:
- `proj.cameras` — `Dict[str, CameraHandle]` s klíči podle názvu A sériového čísla.
- `proj.arrays` — `Dict[str, ArrayHandle]` s klíči podle názvu A array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` s klíči podle názvu A slot_id.
- `proj.config` — `project.json["config"]` slovník.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Úrovně zpracování.** `capture()`, `grab()` a `frame_stream()` všechny přijímají stejný token `processing`
a řetězec je kumulativní — každá úroveň spouští vše, co je nad ní:

| Úroveň | Výstup | Poznámky |
| --- | --- | --- |
| `raw` | 1kanálový Bayer, nativní pro snímač | Bez demosaikování. Na této úrovni nejsou k dispozici překryvy. |
| `debayered` | 3kanálová barva (**výchozí** pro `CameraHandle` a všechny `grab()` / `frame_stream()`; `ArrayHandle.capture()`, `capture_smart()` a `stream()` mají jako výchozí nastavení `raw`): BGR v paměti (`grab()` / `frame_stream()`), uložené jako RGB pomocí `capture()` | V režimu backendu, jako snímání: tmavý offset (DSNU) z kalibračního balíčku odečtený od surových dat, poté demosaikace s ohledem na hrany pomocí OpenCV — bez LUT linearity, flat fieldu nebo unmixu. Jediná úroveň, která funguje bez režimu backendu, kde se jedná o prostou bilineární demosaikaci knihovny kamery (bez tmavého posunu). Uložený soubor TIFF je ve fotometrickém RGB, takže je třeba obrátit kanály snímku v paměti (`frame[..., ::-1]`), abyste jej mohli porovnat se souborem. |
| `radiance` | float32, W/m²/sr/nm, **jedna rovina na pásmo** | Úplný radiometrický řetězec: demosaikování + 3×3 unmix (multispektrální) + DSNU + flat-field + NIST stupnice, s odečtením expozice × zisku, takže hodnoty jsou absolutní. |
| `reflectance` | uint16, 32768 = 1,0 (nebo uint8, 255 = 1,0 — viz `bit_depth`), **jedna rovina na pásmo** | Radiance dělená dopadajícím zářením (ρ = π·L/E). Vyžaduje údaj z DLS/DAQ — viz poznámka níže. |
| `display` | 8bitový, podobný sRGB | Vykreslení odpovídající grafickému rozhraní: CCM + vyvážení bílé + gama prostřednictvím aktivního barevného profilu kamery. |

V přímém (bez backendu) režimu je k dispozici pouze `debayered`: `grab()` na jiné úrovni vyvolává
`NotImplementedError`, `capture()` ignoruje `processing` a `frame_stream()` vyvolává
`RuntimeError` na každé úrovni. `reflectance` potřebuje použitelnou hodnotu měření směrem dolů — koncový bod snímku automaticky vtáhne
sdružený DAQ do slotu DLS kamery, ale pokud není vázán žádný DAQ, řetězec odmítne
výstup odrazivosti a upřímně označí snížení úrovně ve vrácených metadatech, místo aby tiše
vrátit produkt nižší kvality.

> **Stupnice DN odrazivosti — neukládat ji pevně do kódu.** Reflektance LATTICE používá `32768` = ρ 1,0 a označuje
> XMP `Chloros:PixelScale=32768`; reflektance Survey3 používá `65535` = ρ 1,0 a neobsahuje žádné
> značky `Chloros:*`. Přečtěte značku a vydělte jí hodnotu. Je definována v doméně uint16, takže zůstává
> `32768` pro každý formát, který mění měřítko (16bitový TIFF, 8bitový TIFF, 8bitový PNG/JPG, 32bitové procento)
> — nejprve normalizujte uložený datový typ zpět na uint16 (×257 z 8bitového, ×65535 z float). Jedinou
> výjimkou je soubor z verze starší než 1.3.0: 8bitový TIFF byl tehdy *oříznut*, nikoli přepočítán, takže
> nelze jej popsat žádným měřítkem a neobsahuje tuple `PixelScale` ani MicaSense. Chybějící značku v
> souboru odrazivosti LATTICE považujte za „neplatné měřítko“, nikoli za výchozí hodnotu.

**Kalibrační značky: pouze u `raw`.** Soubor `raw` ze záznamu obsahuje kalibrační paket kamery: užitečné zatížení `CalibrationJSON` a značky kompatibilní s MicaSense (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), které popisují surové hodnoty snímače. Soubory všech ostatních úrovní (`debayered`, `display`, `radiance`, `reflectance`, render `index/`) a každý snímek z nepřetržitého záznamu / vícekamerového streamu obsahuje identifikátor kamery (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) a vlastní záznam snímku, `XMP-mapir:FrameJSON` (expozice, zesílení, teplota, formát pixelů, čas pořízení, shoda DAQ), protože každý produkt typu `process()` nese tuto identifikaci. Zpracování vyhledá kalibraci takového souboru podle jeho sériového čísla a `CalBundleSha256` v kalibrační mezipaměti programu Chloros a v případě, že tento počítač kalibraci nemá, stáhne balíček jednou.

#### Soubory vytvořené z několika kamer

Soubor vytvořený z pásem více než jedné kamery — multibandový stack (`capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index(save_multiband=True)`, `Registration.save_multiband()`), živý index vytvořený z několika kamer (`index.tif`, `index_LUT.png`), snímek `composite/` z hromadného snímání — uvádí názvy všech kamer, které se na něm podílely:

- **Vkamery, v pořadí podle pásem** (kompozice: vrstva pozadí, poté popředí; indexová vrstva, kamery, z nichž pocházejí její pásma): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (kalibrační balíček, kterým prošly pásma každé kamery, jinak — sada `capture_aligned_multiband()` / `start_aligned_stream()`, jejíž pásma neprocházejí žádným — balíček kamery, který si Chloros uložil do mezipaměti, ten, podle kterého jsou pojmenovány její soubory záznamů), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — paralelní seznamy, položka *n* v každém z nich popisuje stejnou kameru; `-`, kde kamera nemá žádnou hodnotu (žádný kalibrační balíček, neznámý objektiv). Každá kamera představuje **jednu** položku: pokud její balíček obsahuje pro dané pole více hodnot — certifikáty halogenových lamp v balíčku — jsou spojeny pomocí `;` (`F-1479-1;F-1480-1;F-1481-1`), nikdy čárkou, takže seznam obsahuje jednu položku na fotoaparát bez ohledu na to, jak je vytištěn.
- **EXIF `SerialNumber`** obsahuje stejné sériové čísla, oddělené čárkami (`214000535,214000551`).
- Neobsahuje **žádnou** identifikaci jednotlivé kamery (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): ty označují jednu kameru a její sadu, a zpracování by z nich určilo kalibraci souboru. Index vypočítaný z pásem několika kamer (`index.tif`, `index_LUT.png`) má EXIF `Model` `LATTICE` (sestava), nikoli model `LATT-…` jedné kamery.

Soubor obsahující pásma **jedné** kamery — TIFF pro každé pásmo, snímek `capture_aligned_multiband(per_camera=True)`, stack nebo živý index jedné kamery — nese identifikační značky této kamery, stejně jako každý jiný soubor se záznamem. Sada snímků `capture_aligned_multiband()`, soubory pro jednotlivé pásma a pro jednotlivé kamery, stejně jako každá sada snímků `start_aligned_stream()` a její soubory pro jednotlivá pásma obsahují čas pořízení své skupiny (expozice referenční kamery, UTC) a snímek `per_camera` z monochromatické kamery je v jedné rovině, stejně jako sada `bit_depth`.**Orientace kamery (překlopení / otočení).** Snímky z `grab()` / `frame_stream()` jsou vždy v
nativní orientaci kamery, na všech úrovních. Uložené snímky se řídí nastavením orientace kamery
(možnosti GUI „Mirror H“ / „Mirror V“ / otočení):

- **Soubory `raw` a `debayered` přesně odpovídají tomu, co snímač zaznamenal**, bez ohledu na nastavení — otočená nebo převrácená Bayerova
  mozaika by představovala odlišný vzor barevných filtrů a kalibrační mapy jsou
  na vlastní mřížce snímače.
- **Soubory `display`, `radiance`, `reflectance` a indexové soubory jsou orientovány** — otočeny o jeden úhel po provedení všech
  kalibračních výpočtů. Jejich omezovací masky leží na stejné mřížce.
- **Každý snímaný soubor zaznamenává nastavení** v XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, ve směru hodinových ručiček, po
  otočení) a `ChlorosOrientationApplied` — `false` na surových a debayered snímcích, `true` na orientovaných
  výstupech. `ChlorosProcessing` uvádí seznam operací aplikovaných na pixely daného souboru (`none` v případě
  nativního formátu). To zahrnuje každý snímek z nepřetržitého záznamu (vlastní nastavení kamery,
  nepoužito) i z vícekamerového proudu (identita: jeho jednotlivé snímky jsou v nativním formátu, stejně jako u
  záběru z pole).
- **Zpracování projektu aplikuje zaznamenanou orientaci** na produkty, které vytváří z surových
  (nejprve kalibrace v souřadnicích snímače, poté deformace pro zarovnání pole, nakonec orientace),
  takže odpovídají výstupům živého snímání — včetně nekalibrovaných výstupů s odezvou snímače a výstupů s korekcí vinětace;
  výstup po odstranění bayerského filtru zůstává v nativním formátu.
- **Prohlížeč obrázků Chloros zobrazuje soubor ve formátu raw nebo po odstranění bayeru otočený podle záznamu** — pouze pro zobrazení,
  soubor se nemění; soubor bez záznamu se zobrazuje tak, jak je uložen.
- **Záznamy bez záznamu se zpracovávají jako nativní.** Před verzí 1.3.0 snímání z jedné kamery
  samo převracelo své soubory ve formátu RAW a po odstranění bayesovského vzoru; Chloros nedokáže rozlišit takový soubor RAW od nativního a
  neodhaduje to.
- **Snímání z pole zachovává ve výchozím nastavení každý členský soubor v nativním formátu** a zaznamenává identitu;
  `ArraySession.capture(orient=True)` zpracovává jejich výstupy (viz ovládací prvky pro snímání pole).

#### Výstup podle jednotlivých pásem a smíšená pole

Na úrovních `radiance` a `reflectance` kamera vrací **jednu rovinu pro každé pásmo dané
kamery** — tři pro M3C (její tři nemíchaná pásma, pořadí kanálů BGR), jednu pro M3M (její
jediný úzkopásmový filtr). Replikované roviny monokamery jsou sloučeny, takže **smíšená
pole M3C/M3M poskytuje různé tvary pole pro každou sérii**. Přečtěte si klíč `bands` z metadat snímku, abyste zjistili, o jakou rovinu se jedná; nevyvozujte to z počtu kanálů:

```python
for serial, (arr, hdr) in arr_handle.grab(processing="reflectance",
                                          with_metadata=True).items():
    print(serial, arr.shape, hdr["bands"])   # ('…', (1536, 2048, 3), ['nir','green','red'])
                                             # ('…', (1536, 2048, 1), ['F685'])
    rho = arr.astype("float32") / hdr["pixel_scale"]
```

Před použitím hodnot vždy zkontrolujte `hdr["processing_level"] == "reflectance"`. Snížení úrovně
(žádné DAQ čtení, žádný kalibrační balíček, neznámé rozložení pásem) je hlášeno v `processing_level` a
`radiometric_refused`, nikoli zvýšení úrovně — pole, které získáte zpět, je stále platný rámec, jen
nejde o produkt, o který jste požádali. Uložený záznam, který byl degradován, nese ve svém XMP stejný kód
jako `Chloros:RadiometricRefused` (např. `no-band-layout`, `no-dls-snapshot`).

#### Jeden spouštěč na jedno čtení — `group_id`

**Členové pole jsou obsluhováni ze synchronizované skupiny snímků pole, nikoli z vlastního
snímacího bufferu každé kamery.** Kamery hardwarově synchronizovaného pole exponují společně, ale N čtení na každou kameru
neprovádějí *vzorkování* společně — čtení nejnovějšího snímku z bufferu každé kamery překračuje hranice spouště
a jediný vynechaný snímek desynchronizuje sadu, aniž by si toho někdo všiml.

Poskytování dat ze skupiny je nezbytné, ale samo o sobě nestačí: každý požadavek stále
pořizuje snímek *nejnovější* skupiny v daném okamžiku a smyčka snímání může mezitím publikovat další snímky.
Při měření na živém poli se 4 kamerami **12 z 20 paralelních čtení překrývalo dvě skupiny** — vždy
přesně sousedící. Čtení je tedy **fixováno**:

1. Každý snímek nese `group_id`, monotónní čítač publikování soustavy.
2. `grab()` načte data z hlavní kamery (nebo, pokud hlavní kamera nemůže poskytnout data, z další kamery, která to dokáže),
   poté přiřadí zbývající kamery k ID skupiny, na které se zastavilo (`?group_id=N` na HTTP
   koncovém bodě).
3. Backend uchovává nedávné skupiny po dobu **alespoň půl sekundy**, takže sourozenec, který již
   byl předběhnut pozdějšími publikacemi, je stále obsluhován tímto spouštěčem — dostatečně dlouho na to, aby se stihlo
   exportu odrazivosti, a to i v případě pole s 9 kamerami s plným snímačem. Přiřazení, které backend již
   neudržuje, je odmítnuto kódy `409` a `group_aged_out: true`, namísto tiché odpovědi
   s jiným spouštěčem — `grab()` znovu načte sadu.

Při opakovaném měření s fixací: **20 z 20 čtení vrátilo jeden spouštěcí signál.** `grab()` to za vás provede;
`group_id=` potřebujete pouze při přímém ovládání koncového bodu HTTP.

- Kamera, kterou nelze obsloužit v připnuté skupině — chybí v ní nebo odmítá požadavek
  (`band`, který nemá, úroveň, kterou nemůže vygenerovat) — je **vynechána, místo aby byla doplněna**
  z jiného spouštěče a ostatní kamery jsou stále vráceny.
- Slovník, který `grab()` vrací, je `GrabResult` (obyčejná podtřída `dict`;
  `from chloros_sdk import GrabResult`): `.errors` přiřazuje každé
  vynechané sériové číslo k výjimce, kterou vyvolalo jeho čtení, a `.group_id` je sdílená skupina. Prázdný
  `.errors` znamená, že do této skupiny patří všechny kamery.
- Samostatná kamera nemá žádnou skupinu, hlásí `group_id: None` a **odmítá** pin `group_id=`
  namísto jeho ignorování.

```python
frames = arr_handle.grab(processing="reflectance")
for serial, err in frames.errors.items():
    print(f"SN:{serial} omitted: {err}")       # e.g. status=400 ... band 'F685' not in [...]

frames = arr_handle.grab(processing="reflectance", with_metadata=True)
ids = {hdr["group_id"] for _a, hdr in frames.values()}
assert len(ids) == 1, f"straddled triggers: {ids}"     # belt and braces
```

Stejný čítač je publikován v **každém MJPEG proudu pro jednotlivé kamery** člena pole — v
náhledu (`/api/camera/stream/<serial>`), v anotovaném náhledu (`/stream-annotated`) a v
reflexní stream (`/stream-reflectance`) — jako `X-Chloros-Group-Id` v každé části, kromě
`X-Chloros-Frame-Id` a `X-Chloros-Timestamp-Ns`. Proudy jsou spojité, nikoli
bodové, takže se přiřazují shromážděním částí a seskupením podle této hlavičky — není třeba
nastavovat žádný pin. Samostatná kamera nemá žádnou skupinu. Viz
[MJPEG toky pro jednotlivé kamery](#per-camera-mjpeg-streams-no-rtsp).

**`frame_stream()` nikdy nezmlkne.** Tick, který selže z důvodu, který se může sám vyřešit,
se přeskočí – proud z kamery pro něj nevrátí nic, proud z pole vrátí kamery, které
odpověděly (možná žádná). Těmito důvody jsou: zatím žádný snímek; vlákno pro snímání kamery se na
sekundu či dvě pozastavilo (`404 not streaming`, na což backend odpoví vždy, když se připojí
nebo odpojí další kamera, nebo když změna nastavení vyváží fond); restart backendu nebo `5xx`; zarovnání
se stále ustálí (`409 … retry shortly`); a odmítnutí odrazivosti, protože DAQ zatím nemá žádné měření
(`no-dls-snapshot`, zastaralé nebo prázdné měření). Odmítnutí v následujícím taktu by se pouze opakovalo —
neznámé `band`, `bit_depth=8` u radiance, kalibrační balíček bez stupnice radiance, odmítnutí odrazivosti
**bez jakéhokoli DAQ** (žádné není přiřazeno k kameře ani k jejímu poli, žádné není připojeno k
backendu: `dls_source` odmítnutí je `"none"`), jakýkoli jiný `4xx` —
vyvolá `chloros_sdk.FrameFetchError` (`RuntimeError`, nesoucí `.status` a `.body`)
najednou. Stejně tak **60 sekund** vynechaných tiků bez mezilehlého snímku, ať už je `fps` cokoli — dostatečně
dlouhá doba na to, aby se ustálilo zarovnání a restartoval se backend, a dostatečně krátká na to, aby kamera, která je
skutečně nefunguje, ukončí stream. Tato chyba vzdání se je také `FrameFetchError`, a to jak pro kameru, tak pro
pole, přičemž výjimka posledního čtení je `.errors` (a u kamery jako `__cause__`). U
pole znamená „refused“ (odmítnuto), že všechny kamery odmítly; výjimka `.errors` obsahuje důvod
každé kamery.

**Pole `frame_stream()` vydá každý spouštěč jednou** (stejně jako `ArrayHandle` a `ArraySession`).
Tik, jehož nejnovější sada je spouštěčem, který již vrátil, čeká na další spouštěč namísto toho, aby
znovu vrátil stejnou sadu, takže proud běží pomaleji než `fps` nebo vlastní frekvence sady, a po sobě jdoucí sady nikdy nesdílejí `group_id`. Pole, které přestane publikovat, vyvolá
`FrameFetchError` („žádný nový spouštěč po dobu 60 s“) namísto opakování své poslední sady. Bez toho by
proud, který požadoval více, než pole publikovalo — nebo který byl spuštěn hned po připojení — vydával
opakování: v jednom měřeném běhu obsahovalo 20 sad 11 různých identifikátorů skupin.

```python
from chloros_sdk import FrameFetchError

try:
    for arr in cam.frame_stream(processing="radiance", bit_depth=8):
        my_analysis(arr)
except FrameFetchError as exc:
    print(exc.status, exc.body)        # the HTTP status and the backend's reason
```

#### Exportování jednoho pásma — `band`

`grab()` / `frame_stream()` použijte `band="<name>"` k exportu pouze jednoho z pásem kamery jako
jednokanálového pole `(H, W, 1)` — stejného tvaru, jaký produkuje monochromatická kamera, takže výstup z jednoho pásma je
zpracován stejně, ať už pochází z M3M nebo z rozdělení M3C. Červené pásmo FRGN:

```python
red = cam.grab(processing="reflectance", band="red")        # (1536, 2048, 1)
```

Přijatelné zápisy jsou buď úplný název rozložení (`Red_660`, `NIR_850`, `Green_550`), nebo jeho zkrácená
hlavička (`red`, `nir`, `green`), přičemž se nerozlišují velká a malá písmena. Neznámý název je odmítnut a jsou zobrazeny dostupné
názvy.

> **Výběr se provádí podle názvu, nikoli podle čísla kanálu — záměrně.** „Kanál 1“ je `Green_550` v
> pořadí kanálů BGR pole, ale *červený* v pořadí písmen filtračního kódu(`RGN`), a tyto dva
> se neshodují. Výběr nesprávného kanálu vede k naprosto věrohodnému rámci nesprávného pásma, který
> nic na výstupu nedokáže detekovat. Z tohoto důvodu neexistuje žádný číselný index.

#### Sladění datových toků — `align`

`align=True` transformuje snímek do společného referenčního rámce svého pole. Každá kamera je transformována
pomocí **stejného** profilu rozlišení a oříznuté oblasti zájmu (ROI), takže datové toky jednotlivých kamer otevřené nezávisle na sobě
jsou **vzájemně zaregistrované** — nemusíte je vyžádat společně:

```python
frames = arr_handle.grab(processing="reflectance", align=True)   # {serial: array}
# or one MJPEG stream per camera, each co-registered on its own:
#   GET /api/camera/<serial>/stream-reflectance?align=1
```

Požadavky a chování:

- Kamera musí být v **hardwarově synchronizovaném** poli (synchronizační kabel M8), které má profil zarovnání
  — společná registrace předpokládá, že členy pole jsou exponovány současně, což pole v režimu **free-run** nebo
  volně běžící pole nezajišťuje. V opačném případě bude požadavek odmítnut s kódem `409`.
- **Odmítnuto, nikdy není tiše nesrovnáno.** Cesta pro snímání může uložit nesrovnané snímky s varováním,
  protože soubor si zachovává své značky původu a lze jej později znovu zaregistrovat; živý přenos takovou
  druhou šanci nemá, takže chybějící profil je chybou, nikoli náhradním řešením.
- **Žádost o zarovnání jej spustí.** Požadavek na `align` u pole bez profilu spustí
  kalibraci a vrátí `409` s tím, na co čeká (např. *„čekání na ustálení expozice
  na 214000551 (1/3)“*) — při opakovaném pokusu se to podaří, obvykle během několika desítek sekund.
  To má význam při bezhlavém provozu: automatické zarovnání při připojení se po 30 s vzdá a každý pokus
  v tomto časovém okně je přeskočen, dokud se inteligentní AE ještě ustaluje, takže pole, jehož expozice
  se ustaluje pomalu, dříve skončilo bez profilu pro celou relaci a bez možnosti jej získat
  bez otevření kombinovaného zobrazení na ploše. `alignment_status` také hlásí
  `ae_align_holdouts`, přičemž uvádí názvy kamer, které se stále stabilizují, a jejich průběh.
- Každému proudu jednoho pole přiřaďte **stejný `max_h`**, jinak se jejich výřezy ocitnou na různých pixelových
  mřížkách.
- Zarovnání se provádí *před* kódováním bitové hloubky, takže warp interpoluje hodnoty s plnou přesností.

#### `bit_depth` — výměna přesnosti za šířku pásma

`grab()` a `frame_stream()` přijímají `bit_depth=16` (výchozí) nebo `bit_depth=8`:

| `bit_depth` | Kódování odrazivosti | Náklady |
| --- | --- | --- |
| `16` | uint16, 32768 = ρ 1,0, rezerva do ρ 2,0 | Plná přesnost. |
| `8` | uint8 pokrývající `0..refl_max` (výchozí ρ 1,0) | Polovina bajtů; kvantizace ~0,004 ρ při výchozí horní hranici. Hodnoty nad horní hranicí jsou oříznuty — viz níže. |

##### 8bitová horní hranice — `refl_max`

`refl_max` nastavuje hodnotu ρ, kterou 8bitový rozsah pokrývá, takže `pixel_scale = 255 / refl_max` a ρ je stále
`pixel / pixel_scale`. `grab()` a `frame_stream()` jej berou jako klíčové slovo jak na kameře, tak
v handlech pole (`?refl_max=` na HTTP-koncovém bodě). Zvyšte tuto hodnotu, abyste získali rezervu pro odlesky,
a to v poměrných hrubších krocích:

```python
frame = cam.grab(processing="reflectance", bit_depth=8, refl_max=1.5)   # 0..1.5 in 0..255
```

| `refl_max` | `pixel_scale` | krok |
| --- | --- | --- |
| `1.0` (výchozí) | 255 | 0,0039 ρ |
| `1.5` | 170 | 0,0059 ρ |
| `2.0` | 127,5 | 0,0078 ρ |

> **Automatická expozice nedokáže opravit 8bitové oříznutí a její použití k tomuto účelu data ještě zhorší.**
> ρ = π·L/E po vydělení expozice a zesílení, takže ρ je vlastností scény, nikoli
> nastavení fotoaparátu — při měření na živém poli snížení expozice o polovinu posunulo střední hodnotu ρ o 1,3 %. Zvýšení
> expozice za účelem „přizpůsobení“ rozsahu naopak vede k saturaci pixelů **snímače**, což
> podhodnocuje L a posouvá ρ **směrem dolů**, aniž by byla vygenerována chyba (měřeno: max. ρ 1,203 → 0,985 při
> dvojnásobné expozici, průměr −23 %). Rozsah zvolte pomocí `refl_max`; zabránění
> nasycení snímače je již úkolem automatického nastavení expozice (AE), což chrání *platnost* hodnoty ρ spíše než její rozsah.

**Přečtěte si `reflectance_clipped_fraction`, ne příznak.** `reflectance_clipped_at_1_0` se spustí kvůli
jedinému pixelu — ve skutečné scéně hlásilo `True`, zatímco podíl byl `0.000001`, jeden pixel
z milionu. Právě tento zlomek vám říká, zda je třeba zvýšit horní hranici.

**`bit_depth=8` se týká pouze odrazivosti.** Žádost o `radiance` je zamítnuta, nikoli
vyřízena: odrazivost má pevný rozsah 0..1, který vydrží 8bitové kódování, zatímco radiance nemá
pevný rozsah, takže její vtěsnění do 8 bitů vyžaduje pro každý snímek procentilové roztažení, jehož mapování
se mění s každým snímkem — jde o obrázek, nikoli o měření.

#### Živý datový proud odrazivosti (MJPEG)

`GET /api/camera/<serial>/stream-reflectance` poskytuje 8bitovou odrazivost pro každý pásmo jako
`multipart/x-mixed-replace` — M3C jako 3kanálový JPEG, M3M jako šedá stupnice. Parametry dotazu:
`fps` (výchozí 10), `jpeg_q` (výchozí 95), `max_h` (0 = nativní), `band` (jedno pojmenované pásmo, jak
je uvedeno výše), `align` (`1` pro společnou registraci — stejná pravidla jako u `align` v SDK), `refl_max` (horní hranice 8bitového
kódování, viz výše). Každá část nese
`X-Chloros-Group-Id`, `X-Chloros-Frame-Id`, `X-Chloros-Timestamp-Ns`, `X-Chloros-Bands`,
`X-Chloros-Pixel-Scale`, `X-Chloros-Aligned` a `X-Chloros-Reflectance-Clipped`.
**Části z různých kamer nesoucí stejný `X-Chloros-Group-Id` představují stejný spouštěč** —
takto se znovu sestavují jednotlivé proudy z jedné matice pro každou kameru a je to zaručeno
na straně serveru, protože členy matice jsou poskytovány ze synchronizované skupiny snímků (viz
[Jeden spouštěč na čtení](#one-trigger-per-read--group_id)). Vrátí `409`, pokud není odrazivost
k dispozici v okamžiku připojení, a ukončí stream, pokud se stane nedostupnou uprostřed běhu, takže zastavený
stream znamená „odrazivost již není k dispozici“ spíše než „stále v pořádku, jen statický“.

> **Formát JPEG je ztrátový — tyto pixely představují měření.** Chyba komprese se přímo promítá do
> hodnot odrazivosti a je nejhorší na okrajích, kde se nacházejí hranice materiálů. Z tohoto důvodu je
> výchozí hodnota kvality nastavena na 95. Pokud potřebujete zachovat čísla beze změny, použijte
> `grab(processing="reflectance")` / `frame_stream(...)`, které vrátí stejný výsledek bez
> kodeku v cestě (a `bit_depth=8`, pokud je omezením cena přenosu, nikoli kodek).

> **EXIF se přenáší do exportu.** `process()` zkopíruje blok GPS zdrojového záznamu
> **a jeho ExifIFD** do každého produktu, takže exporty obsahují `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` a `CameraSerialNumber`, stejně jako
> georeferencování. `FocalLength` je hodnota, ze které Pix4D vypočítává vzdálenost mezi body na zemi – bez ní
> se rekonstrukce vrátí k naprosto nesprávnému měřítku (v jednom změřeném případě se z lokality o rozloze 411 m
> stala lokalita o rozloze 47,8 km). Kopie záměrně není `-all:all`: strukturální značky IFD0 narušují
> výstup LATTICE a `ExifImageWidth`/`Height` jsou vyloučeny, protože popisují zdrojové
> snímání, nikoli exportovaný rastr.

Podznačky fáze snímání (vztahují se na radiometrické úrovně — `radiance`, `reflectance`, `display`):

| Příznak | Výchozí hodnota | Význam |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + flat-field + 3x3 unmix + radiometrická stupnice NIST. |
| `apply_white_balance` | `True` | WB LUT. Zohledňuje DLS, pokud je k fotoaparátu připojen DAQ. |
| `apply_index` | `False` | Vyhodnocení vegetačního indexu. |
| `index_expression` | `None` | Přepsání vzorce. Prázdnéprázdné → automaticky aktivuje index. |
| `annotated` | `False` | Překrytí dekorací grafického rozhraní (zebra/mřížka/peaking). Není k dispozici pro `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Návratová hodnota je `chloros_sdk.CapturePaths`, typu `CapturePathMap`, nikoli `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` je `Dict[str, Union[str, List[str]]]`: jednovrstvý
> `processing` přiřadí každému sériovému číslu jednu cestu, zatímco `processing="all"` mu přiřadí
> **seřazený seznam** všech produktů uložených pro danou kameru. `ArrayHandle.capture()` nepřijímá
> žádné `levels=` a ignoruje klíčová slova, která nezná (`levels`, `daq`,
> `render_index`, `force_daq`), proto předávejte pouze ty, které jsou zde uvedeny (`aligned=False`
> ukládá nativní soubory, stejně jako `ArraySession.capture()`); pro tyto ovládací prvky použijte
> `ArraySession.capture()`. Živý kombinovaný kompozit, pokud byl streamován, dorazí pod dodatečným
> klíče `"combined"`, nikoli pod sériovým číslem. Kód, který předpokládá `str`, selže v
> seznamovém formátu, aniž by proti tomu vznesl námitku jakýkoli typový kontrolor — anotace uváděla `Dict[str, str]`
> ještě nějakou dobu po vydání formy seznamu, a proto tento alias existuje. Normalizujte
> v případě, že chcete plochou formu:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

> **Co nebylo možné napsat: `.failed`.** Mapa, kterou vrací `capture()`, je
> `chloros_sdk.CapturePaths` — prostá podtřída `dict`, takže vše výše uvedené funguje
> beze změny — která také obsahuje `.failed`: soubory, které backend nemohl zapsat,
> stejné položky jako [`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed)
> (`serial`, `level`, `error` a `disk_full: True` na zaplněném disku). Částečné
> zachycení stále vrací všechny cesty, na které se zapsalo; na zaplněném disku je mapa
> prázdná a soubor `.failed` uvádí důvod. `.skipped` uvádí přeskočené záznamy pro jednotlivé kamery a
> `.group_seq` synchronizovanou skupinu, a `.alignment` / `.oriented` jsou
> záznamy backendu, stejně jako u `CaptureResult`. `capture_smart()` je vrací jako
> `result["frames"].failed` a `result["failed"]`.
>
> ```python
> paths = arr.capture("./out", processing="all")
> if any(f.get("disk_full") for f in paths.failed):
>     raise SystemExit("drive full -- free space and capture again")
> ```

### Zarovnání pole

`ArrayHandle` zpřístupňuje celou zarovnávací plochu. Profily jsou ve výchozím nastavení platné pouze pro danou relaci — pro trvalé uložení explicitně zavolejte `export_alignment()`.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Manual control points: each camera's pixel coordinates of the same
# features, the reference camera's included (>= 2 per camera). Every one
# of num_frames is solved from these points.
arr.calibrate_alignment(method="manual", model="rigid", control_points={
    "214701173": [[412, 300], [1630, 310], [1020, 1180]],   # reference (master)
    "214701292": [[418, 297], [1637, 306], [1027, 1177]],
})

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Zarovnání při připojení

`connect_all(align=...)` dokáže automaticky zarovnat každé pole při připojení:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Pokud není zadáno, použije se `project.json["config"]["auto_align_on_connect"]`.

### `SensorHandle`

Uložený světelný senzor (DAQ-U, DAQ-M nebo DAQ-E), otevřený pomocí `connect_all()` v [poolu senzorů DAQ](#daq-sensor-sessions) — ve stejném poolu, jaký používají `connect_daq_sensor()` a `chloros-cli daq pool-connect`.

```python
reading = proj.sensors["Sky"].read()
if reading is not None:
    spectrum, is_saturated, integration_time, x, y, z = reading
```

- `read()` čeká na **další** spektrum senzoru — to, které je již v paměti vyrovnávací při volání, se nepočítá — a vrací `(spectrum, is_saturated, integration_time, x, y, z)`, nebo `None`, pokud žádné nedorazí do 5 s. Pozastavený datový tok je obnoven. `spectrum` je vyjádřeno v W/m²/nm, pokud je senzor kalibrován.
- `.sensor` je [`DAQSensorSession`](#daq-sensor-sessions), ve všech ostatních případech: `latest(n=...)`, `status()`, `record_start()` / `record_stop()`.
- `.slot_id`, `.model`, `.serial`, `.name` a `.settings` jsou uložené položky z `sensors.json`.

**Co obnovuje `connect_all`.** Každý snímač se otevře na své uložené adrese — jeho COM portu, adrese Bluetooth nebo hostiteli DAQ-E — a pokud projekt zaznamenal identifikátor snímače, musí se jednat o tento snímač: jiná jednotka na této adrese je odmítnuta, není připojena. Použijí se jeho uložená doba integrace, průměrování snímků a automatická expozice, stejně jako korekce krytky:

| Uložený profil krytu | Propojen s |
| --- | --- |
| Profil krytu (např. `fov_30`) | Tento profil. |
| Žádný nevybrán | `sunshine_cosine`, výchozí při připojení, zaznamenán jako předpokládaný (`cap_id_source=auto_default`), proto zpracování vydá varování. Senzory se dodávají bez krytu: vyberte kryt, který je nasazen. |
| `none` — vybráno jako **Žádný (senzor bez krytu)** v grafickém uživatelském rozhraní | Žádná korekce krytu. |

Senzor DAQ-E-S nepřijímá žádný profil krytky, bez ohledu na to, co bylo uloženo: jeho difuzor je součástí jeho kalibrace (viz [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor)). Projekt uložený dřívějším programem Chloros zaznamenal holý senzor jako jedinou možnost, proto se znovu připojí k senzoru `sunshine_cosine`; v grafickém rozhraní znovu vyberte **Žádný (holý senzor)** a projekt uložte.

Senzor, který již má jiný klient otevřený v poolu (CLI, jiný skript), se připojí tak, jak je, bez opětovného použití nastavení, a `disconnect_all()` jej nechá běžet; senzor otevřený `connect_all()` je uvolněn. Stejně jako u `connect_daq_sensor()` není grafické rozhraní Chloros** součástí fondu: před připojením projektu ze skriptu uzavřete senzor v grafickém rozhraní.**Požadavky a zpráva.** Světelné senzory vyžadují spuštěný backend Chloros s přihlášením k placenému tarifu Chloros+ (Copper nebo vyšší), a `connect_all()` jej nikdy nespustí. Pokud žádný neběží, každý uložený senzor je uveden pod `report["sensors"]["errors"]` s touto poznámkou. Senzor, který backend nemůže otevřít, je zde také uveden spolu s důvodem backendu; `report["sensors"]["missing"]` zůstává pro světelné senzory prázdný.

---

## Přímý hardware (bez backendu)

Pokud chcete, aby kamery LATTICE (CI, bezhlavé roboty, vestavěné) nebyly vůbec závislé na backendu, importujte přímo `lattice_sdk` — ten je znovu exportován prostřednictvím `chloros_sdk`. Upozornění k `CAMERA_AVAILABLE`: `lattice_sdk` je součástí balíčku SDK, ale vyžaduje přítomnost runtime Arena SDK a OpenCV (`cv2`), přičemž balíček neinstaluje ani jedno z nich. Světelné senzory procházejí backendem — [`connect_daq_sensor()`](#daq-sensor-sessions) nebo přes `proj.sensors` daného projektu — s jednou výjimkou, a to pomocným programem pro amplitudu pásma DAQ-M [`DLS`](#daq-m-band-magnitudes--dlsband_magnitudes), který rovněž vyžaduje `bleak`.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Předvolby a spouštěč

Tři ze čtyř předvoleb fungují v režimu **free-run**: kamera exponuje nepřetržitě a
`capture()` vrátí další snímek. Výjimkou je `triggered` — připravuje
kameru na hardwarový signál na lince 2, takže nic nezachytí, dokud signál nepřijde.

| Předvolba | Spouštěč | Použijte, když |
| --- | --- | --- |
| `default` | volný běh | obecné použití |
| `high_speed` | volný běh | 8 bitů, omezení na 60 fps, krátká expozice |
| `high_quality` | volný chod | 12 bitů, bez omezení fps — obvyklá volba pro statické snímky |
| `triggered` | **připraveno, linka 2** | kamera je připojena k synchronizačnímu kabelu M8 a spouští ji něco jiného |

Pokud zvolíte `triggered` (nebo sami nastavíte `trigger_mode="On"`) bez toho, aby
něco řídilo linku 2, každý `capture()` vyprší — což je správné, protože jste
kameru požádali, aby čekala. SDK to vysvětluje, když k tomu dojde; viz
[SC_ERR_TIMEOUT během snímání](#direct-hardware-backend-free).

> **Poznámka — Zprávy „GVSP probe“ / `SC_ERR_TIMEOUT -1011` při připojení nejsou chybami.**
> Při připojení se SDK pokouší vyjednat**jumbo rámce** (9000-bajtové pakety GVSP) pro vyšší propustnost. Na přímém spojení typu point-to-point přes síťovou kartu (např. adresa `169.254.x.x` v rámci link-local) síť obvykle nedokáže přenášet jumbo rámce, takže tato sonda vyprší a do protokolu se zapíší řádky jako:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Jedná se o **záměrné nouzové řešení**: SDK se automaticky vrátí ke standardním paketům o velikosti 1500 bajtů a kamera pokračuje v připojování normálně (následující řádky `[chunk-enable …]` jsou součástí běžné sekvence připojení). Snímání stále funguje.
>
> Tuto sondu můžete přeskočit, ale **nejedná se pouze o potlačení záznamů do protokolu — vypíná jumbo rámce.** Kamera odpovídá na pingy s příkazem „Don't-Fragment“ pouze do velikosti 1500 bajtů, bez ohledu na to, jak kvalitní je vaše síť, takže samotný pingový test nikdy nedokáže odhalit jumbo rámce; tento test je jediný, který to dokáže. Pokud jej deaktivujete, kamera bude navždy používat standardní 1500-bajtové pakety, a to v jakékoli síti:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Má to smysl pouze v síti, o které *víte*, že nepodporuje jumbo rámce, kde to ušetří zhruba jednu sekundu času připojení na jednu kameru. Jelikož se jedná o skutečný kompromis, nikoli jen kosmetickou změnu, SDK to nyní při použití upozorňuje:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Nechte to tak, pokud k tomu nemáte důvod.**Pokud je tato funkce zapnutá, každá kamera se měří jednou za backendovou relaci a znovu při změně jejího připojení (jiný adaptér, jiná rychlost připojení nebo přerušení připojení, jako například při výměně přepínače); při opětovném připojení se výsledek znovu použije.**Po změně za přepínačem, který zůstal v provozu, restartujte backend, aby se měření provedlo znovu.** První připojení může při této kontrole občas trvat několik minut a v takovém případě to bude uvedeno v protokolu.
>
> Pokud *chcete* propustnost jumbo, povolte jumbo end-to-end (MTU síťové karty 9000 + přepínač, který je propouští), nebo ji zafixujte pomocí `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`, pokud víte, že ji spojení podporuje — upřednostňujte však použití `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` pro jednotlivé příkazy před trvalým nastavením, protože pevně nastavená velikost přeskočí testování a přestane se přizpůsobovat síti před ním. **Každé** zařízení na trase musí jumbo pakety propouštět — včetně jakéhokoli PoE rozbočovače nebo injektoru, což je obvyklý důvod, proč jinak jumbo-kompatibilní konfigurace tyto pakety nedokáže přenášet.

> **`SC_ERR_TIMEOUT -1011` během `capture()` / `grab*()` je jiný problém — v tomto případě se jedná o skutečnou chybu.**
> Výše uvedená poznámka se týká pouze chyby `-1011` zaznamenané**sondou connect-time**. Stejná chyba vyvolaná při**zachycení** znamená, že kamera se připojila v pořádku, ale neodesílá žádné snímky:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Rozhodujícím znakem je kamera, jejíž *řídicí* kanál je v pořádku — vyhledávání funguje, nastavení i zápisy `[chunk-enable …]` jsou úspěšné — zatímco *každý* snímek vyprší.
>
> **Obvyklou příčinou je, že kamera je nastavena na hardwarové spouštění.** V případě chyb `trigger_mode="On"` a `trigger_source="Line2"` kamera nevysílá vůbec nic, dokud na synchronizačním kabelu M8 nedojde k elektrické změně stavu. Pokud nemáte kabel, který by tuto linku řídil, každé zachycení snímku čeká donekonečna. Kamera není vadná a síť funguje správně — dělá přesně to, co má.
>
> Modely `CameraSettings()` a `default` / `high_speed` / `high_quality` nastavují volný běh a snímání, u kterého dojde k vypršení časového limitu v aktivovaném stavu, se vysvětlí samo, místo aby se zobrazilo pouhé `-1011`. `PRESETS["triggered"]` aktivuje linku 2, jak je zamýšleno.
>
> Chcete-li vynutit volný běh u jakékoli kamery:
>
> ```python
> import copy
> settings = copy.deepcopy(PRESETS["high_quality"])   # PRESETS entries are shared
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Pokud i s příkazem `trigger_mode="Off"` stále dochází k vypršení časového limitu, kamera skutečně neposílá data — zašlete nám protokol a `ip link show`.

#### Barevné profily (živý náhled RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` volí barevný profil displeje pro **živý náhled** u kamer RGB (multispektrální kamery toto nastavení ignorují):

| Profil | Význam |
| --- | --- |
| `raw` | Úplně obejít radiometrický řetězec. |
| `linear` | DSNU + flat + WB, bez CCM, bez gama. |
| `natural` | Lineární + naměřený CCM + gama sRGB, pouze s jednoduchým dopracováním (vyhlazení chrominance + desaturace světlých oblastí) — realistické výchozí nastavení. |
| `enhanced` | `natural` plus kompletní úprava typu „hub-parity“ (odstranění pruhů, živost, lokální kontrast CLAHE). Bohatší vzhled při zhruba **dvojnásobných nákladech na úpravu na snímek**, tedy nižší snímková frekvence v režimu LIVE. |
| `custom_temp` | `natural`, ale vyvážení bílé (WB) pevně nastaveno na `custom_cct_k` Kelvin (DLS ignorováno; na straně backendu omezeno na 2000–10000 K). |

Tento profil je **pouze pro živý náhled** a slouží k nastavení rychlosti a vzhledu: uložené snímky vždy získají plnou a bohatou úpravy bez ohledu na vybraný profil, takže volba `natural` za účelem získání času na snímku nesnižuje kvalitu toho, co se uloží na disk. Neznámý profil vyvolá `ValueError`; je-li dostupný backend chloros, je změna také odeslána (POST) do něj, takže se projeví v následujícím náhledovém snímku (uživatelé přímého SDKu bez backendu stále dostávají změnu nastavení).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Zvýraznění (displej RGB) — `set_highlights`

`LatticeCamera.set_highlights(value)` omezuje nejjasnější a nejbělejší pixely **displeje** RGB kamerách — oprava pro jasný bílý povrch, který se vykresluje jako plochá bílá skvrna. `value` je `-100`..`0` (omezeno); `0` je vypnutý a výsledný obraz je pak přesně takový, jaký by byl bez tohoto nastavení. Silnější nastavení snižují bílou barvu displeje (na přibližně 60 % při `-100`) a rozšiřují tónovou křivku, takže povrch, který automatické nastavení jasu v náhledu posunulo do čistě bílé, znovu získá své detaily.

Jedná se o ovládací prvek displeje, ale na rozdíl od sytosti a kontrastu (které mění pouze živý náhled) mění také to, co ukládáte: živý náhled a snímky na **úrovni displeje** se tím řídí, takže i tyto soubory vycházejí tmavší (obsahují `Chloros:DisplayHighlights`); exporty radiance a reflectance se tím nikdy neovlivní a nelze tím obnovit oříznutá data ze snímače — v takovém případě snižte expozici. Multispektrální a monochromatické kamery tuto hodnotu ignorují. Hodnota se ukládá do `settings.highlights` a odesílá se pomocí POST na dostupný backend chloros, přesně jako `set_color_profile`; ekvivalent v CLI je `lattice highlights --set N`.

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_highlights(-60)                      # 0 = off, -100 = strongest
```

#### Mono (M3M) kamery a `Calibration`

Mono **M3M** kamera (`M3M-<lens>-F<wavelength>`) je jednopásmová: jedna rovina v odstínech šedé, bez Bayerovy mozaiky, bez spektrální matice 3×3 pro potlačení přeslechu. `Calibration` ji rozpozná a zpřístupní příznak `is_mono`. Reflexe se stále uplatňuje jako radiometrická mapa pro jednotlivé pásma (demix je identitní matice), ale vícepásmové výpočty na jedné kameře dávají smysl, místo aby vracely nesmyslné výsledky:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Chcete-li vytvořit vegetační index z monochromatického hardwaru, zkombinujte několik kamer M3M s různými vlnovými délkami do zarovnaného vícepásmového stohu (viz [Zarovnání pole](#array-alignment)) a vypočítejte index napříč tímto stohem namísto na jedné kameře.

#### Magnitudy pásem DAQ-M — `DLS.band_magnitudes`

`DLS(mac_address, filter_model="RGN-IMX265")` se připojuje k DAQ-M přes Bluetooth pomocí vlastní adresy BLE MAC. Jedná se o jedinou třídu světelných senzorů, která komunikuje se senzorem přímo, nikoli přes backend, takže vyžaduje to, co potřebují třídy přímých kamer (`CAMERA_AVAILABLE`: runtime SDKu Arena a OpenCV), plus `bleak` pro Bluetooth (`pip install "chloros-sdk[camera]"` nebo `pip install bleak`; bez něj vyvolá `DLS` výjimku `DLSError`). Pokud zařízení DAQ-M připojené jinde (v poolu backendu, v projektu, v GUI) přestane vysílat, proto jej tam nejprve uvolněte. **`mac_address` je povinné**: neexistuje výchozí hodnota, protože každá jednotka má svou vlastní, a `DLS()` bez této hodnoty vyvolá `DLSError` ještě předtím, než se pokusí o jakékoli připojení. Najděte MAC pomocí `discover_daq_sensors(transports=["ble"])` (viz [Vyhledávání](#discovery--finding-an-address-to-connect-with)) nebo `chloros-cli daq pool-discover --only ble`.

`DLS.band_magnitudes(spectrum=None, *, spectral_response=None)` integruje spektrum DAQ-M (poslední naměřená hodnota, pokud je `spectrum` rovno `None`; `DLSError`, pokud ještě žádné není) do odezvy tří kanálů kamery a vrací `{"red": …, "green": …, "nir": …}` — klíče pojmenovávají kanály v tomto pořadí; u křivek RGBové kamery je třetí (`nir`) její modrý kanál.

- Bez `spectral_response` použije křivku flotily filtračního modelu (`filter_model`, na jejímž základě byl `DLS` vytvořen), která byla naměřena na jednom kusu.
- `spectral_response` místo toho používá **vlastní** naměřené křivky kamery: slovník `{"wavelengths": [...], "R": [...], "G": [...], "B": [...]}`, uložený v kalibračním balíčku dané kamery (`color_correction.spectral_response_smoothed` — RGB kamery; multispektrální balíčky takovou křivku neobsahují). Pokud nemůže použít daný slovník, použije se křivka flotily.

```python
from chloros_sdk import DLS

# camera_curves: {"wavelengths": [...], "R": [...], "G": [...], "B": [...]}
with DLS(mac_address="AA:BB:CC:DD:EE:FF") as dls:
    reading = dls.acquire()
    fleet = dls.band_magnitudes(reading)                         # fleet curve
    own = dls.band_magnitudes(reading,
                              spectral_response=camera_curves)   # the camera's own
```

Příkaz `chloros-cli lattice dls --camera SERIAL` v CLI provádí totéž prostřednictvím backendu, vyhledá křivku kamery v načteném nebo uloženém kalibračním balíčku a uvede, čí křivku použil. Přes HTTP se jedná o `POST /api/dls/cli-read` s `mac` (povinné: bez toho je odpověď HTTP 400, `code` `MAC_REQUIRED`) a `camera_serial` v těle zprávy; odpověď `band_magnitudes_source` je `camera <serial>`, pokud byla použita vlastní křivka dané kamery, nebo `model <filter>` pro křivku flotily (kamera bez křivky: není RGB nebo není načten žádný balíček ani v mezipaměti).

---

## Výjimky

Zachyťte základní třídu pro zpracování „všeho, co se v Chloros pokazilo“:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` a `ChlorosConfigurationError` jsou atributy `chloros_sdk` (a `chloros_sdk.exceptions`), ale ne v `__all__`, takže `from chloros_sdk import *` je nezahrnuje: importujte je podle názvu. Rodina `ChlorosError` je odvozena od `ChlorosLocal` a funkcí na ní postavených (`process_folder()`, `process_lattice_capture()`); povrch smart-connect zdědí `ChlorosConnectError`.

Hierarchie:

```

ChlorosError
├── ChlorosBackendError           (backend binary not found / failed to start)
├── ChlorosConnectionError        (HTTP transport failure; no backend running
│                                  with auto_start_backend=False)
├── ChlorosLicenseError           (403 PLAN_UPGRADE_REQUIRED: plan below Copper)
├── ChlorosAuthenticationError    (401 AUTH_REQUIRED: not logged in)
├── ChlorosConfigurationError     (bad configure() inputs)
└── ChlorosProcessingError        (pipeline failed; create/import refused)
    └── ChlorosProjectFilesMissingError  (open_project(): none of the
                                  project's source files found, even after
                                  the relink; .missing_roots, .audit)

ChlorosConnectError                (the smart-connect surface: connect_*, attach_array,
                                    list_*, discover_*, every CameraSession /
                                    ArraySession / DAQSensorSession method,
                                    RecorderHandle, ContinuousHandle,
                                    disconnect_all(), and analyze_array_network()
                                    when no backend answers; 401 and 403 included.
                                    Derives from plain Exception, NOT from
                                    ChlorosError, so `except ChlorosError` will
                                    not catch it. .warning: a warning the
                                    backend sent beside its error (also at the
                                    end of the message); .recording_closed: the
                                    recording a failed DAQ-E restart closed;
                                    both None otherwise)

FrameFetchError (RuntimeError)     (grab() / frame_stream() refusals; .status,
                                    .body, .errors)
ValueError                         (capture() arguments: PNG/JPEG with a level they
                                    cannot hold; a bad packed=)
FileNotFoundError / ImportError    (open_project(): no such folder / no project
                                    loaders)
RuntimeError                       (CameraHandle / ArrayHandle: a backend refusal)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Kompletní příklady

### 1. Zpracování složky s vlastním ukazatelem průběhu

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Pole LATTICE v reálném čase → odrazivost + referenční data DAQ

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Sběr dat řízený projektem

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Proud snímků z více kamer → NumPy pipeline

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Skript pro snímání bez grafického rozhraní přímo z hardwaru (bez backendu)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Test funkčnosti před připojením pole se 4 kamerami

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["width"], height=r["height"],     # before binning
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Ekvivalent receptu pro snímání (čistý Python)

`run_project_recipe()` spouští recept pro příkazový řádek (CLI) v nezměněné podobě (viz [Projektová složka a recepty](#project-folder--recipes)). Stejné kroky, napsané v jazyce Python, jsou:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply: connect_all() applied each camera's saved settings. CameraHandle
    # has no settings method; a CameraSession on the same pooled camera does
    # (it is borrowed, so it never releases the camera)
    chloros_sdk.connect_camera(cam.serial).set_settings(
        exposure_time=5000, gain=6.0)

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Server MCP (AI asistenti)

`chloros-mcp` umožňuje AI asistentovi, který komunikuje pomocí [Model Context Protocol](https://modelcontextprotocol.io) (MCP), jako jsou například Claude Desktop, Claude Code nebo Cursor, ovládat Chloros: zpracovávat snímky (vytvořit projekt, importovat složku se snímky, vybrat nastavení, zpracovat data a vygenerovat zprávu o výsledcích exportu) a ovládat kamery LATTICE, pole kamer a světelné senzory DAQ (připojit, zobrazit náhled, nastavit expozici a barvy, pořídit snímky, zaznamenat data, aktualizovat firmware). Jeho nástroje pokrývají funkce tohoto SDK. Stačí zadat požadavek v běžném jazyce a asistent vyvolá nástroje serveru:

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

The assistant calls:
  create_project          {"project_name": "Flight_2026-10-17"}
  import_images           {"folder_path": "D:\\Flights\\2026-10-17"}
  configure_processing    {"indices": ["NDVI"]}
  start_processing        {}
  get_processing_status   {}      (every 30-60 s until its state is "finished")

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.

The assistant calls:
  list_devices            {}
  get_camera_preview      {"array_id": "array-1779862544497"}
  start_capture_run       {"array_id": "array-1779862544497", "mode": "interval",
                           "interval_s": 10, "duration_s": 300,
                           "processing": "reflectance"}
  get_job_status          {"job_id": "..."}   (until its state is "finished")
```

Server je postaven na tomto SDK: nástroje pro zpracování volají [`ChlorosLocal`](#image-processing--chloroslocal) a hardwarové nástroje volají [funkce smart-connect](#smart-connect-for-lattice-cameras), [DAQ relace](#daq-sensor-sessions), `ChlorosProject` a [nástroje pro kamery](#camera-utilities), přičemž všechny komunikují se stejným backendem, jaký používá aplikace Chloros. Její požadavky nesou `X-Chloros-Client: sdk`, takže backend provede kontroly přihlášení a tarifu podle SDK a zároveň se uplatní pravidla SDK (například funkce Texture Aware je pro snímky z kamer LATTICE odmítnuta).

### Instalace

Server je dodáván v balíčku `chloros-sdk`; není třeba instalovat nic dalšího. SDK nainstaluje příkaz `chloros-mcp`, který je stejný jako `python -m chloros_sdk.mcp_server`:

```bash
pip install chloros-sdk        # not needed where the Chloros installer set up the SDK (Step 2)
chloros-mcp --help
```

- **Windows:** instalační program Chloros nainstaluje SDK pomocí pip do vašeho Pythonu, čímž se `chloros-mcp.exe` umístí do složky `Scripts` daného Pythonu.
- **Linux (.deb):** balíček nainstaluje SDK pro systémový adresář `python3` a umístí příkaz `chloros-mcp` do adresáře `/usr/bin`, takže je k dispozici v adresáři `PATH`. Ve virtuálním prostředí (virtualenv) přidá `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` `chloros-mcp` do adresáře `bin` daného virtuálního prostředí.

Server používá pouze standardní knihovnu nad SDK (Python 3.7+). Spouští jej klient MCP a komunikuje s ním přes stdin/stdout; sami jej nespouštíte, s výjimkou `--help`.

### Nastavení klienta

**Claude Desktop:**přidejte server do `claude_desktop_config.json` (ve Windowsu do `%APPDATA%\Claude\claude_desktop_config.json`; otevřete jej přes**Nastavení > Vývojář > Upravit konfiguraci**), poté restartujte Claude Desktop:

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

**Claude Code:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Kurzor:** stejný záznam `mcpServers` jako v Claude Desktop, v souboru `~/.cursor/mcp.json` (v každém projektu; `%USERPROFILE%\.cursor\mcp.json` ve Windows) nebo `.cursor/mcp.json` v jednom projektu.

Možnosti se zadávají v `args` nebo za příkazem pro Claude Code (`claude mcp add chloros -- chloros-mcp --no-auto-start`). Chcete-li místo Python SDK v `chloros-mcp` na `PATH` použít konkrétní Python SDK, spusťte modul s tímto Pythonem (jeho úplnou cestou jako `command`, pokud to není první `python3` v `PATH`):

```json
{
  "mcpServers": {
    "chloros": {"command": "python3", "args": ["-m", "chloros_sdk.mcp_server"]}
  }
}
```

Klient spouští server bez vašeho shellu, takže pokud hlásí, že se server nepodařilo spustit, `chloros-mcp` pravděpodobně není na `PATH`, který klient vidí. Zadejte místo toho úplnou cestu (`where chloros-mcp` ve Windowsu, `which chloros-mcp` v Linuxu, kde .deb je `/usr/bin/chloros-mcp`), nebo spusťte modul pomocí Pythonu, který obsahuje SDK.

### Možnosti

| Možnost | Výchozí | Co dělá |
| --- | --- | --- |
| `--backend-url URL` | `http://127.0.0.1:5000` | Backend Chloros, který se má použít. |
| `--no-auto-start` | off | Nikdy nespouštět backend: použít již spuštěný, například backend aplikace Chloros, backendovou službu pro Linux nebo tunel k (viz [Režim vzdáleného backendu](#remote-backend-mode-pip-only-host-via-tunnel); cesty ke složkám pak musí na obou počítačích odkazovat na stejnou složku). |
| `--backend-exe PATH` | tam, kam jej nainstaluje Chloros | Backend, který se spustí, pokud žádný neběží, pro Chloros nainstalovaný v umístění, kde SDK nehledá. |
| `--log-level LEVEL` | `WARNING` | `DEBUG`, `INFO`, `WARNING` nebo `ERROR`. Protokoly se ukládají do stderr, což většina klientů zobrazuje ve svém protokolu MCP; stdout přenáší pouze protokolové zprávy. |

### Nástroje

Argumenty každého nástroje odpovídají argumentům volání SDK, které tento nástroj obaluje; klient uvádí každý z nich s úplným schématem. Cesty jsou absolutní.

**Zpracování**

| Nástroj | Volání SDK | Co dělá |
| --- | --- | --- |
| `create_project` | `ChlorosLocal.create_project` | Vytvoří projekt a nastaví jej jako otevřený projekt, na kterém pracují nástroje pro zpracování. Zadáním prostého názvu se složka vytvoří ve výchozí projektové složce; zadáním absolutní cesty se vytvoří přímo v ní. |
| `open_project` | `ChlorosLocal.open_project` | Otevře uložený projekt a nastaví jej jako otevřený projekt. Cesty přesunutého projektu jsou přesměrovány na složku, ve které se nyní nachází; `relink_folder` (absolutní) označuje složku, ve které se nyní nacházejí ostatní chybějící soubory. Vrací `source_files` a `image_count`. Nic nalezeno: chybový výpis s uvedením jejich původního umístění a pojmenováním `relink_folder`; nic není otevřeno. |
| `import_images` | `import_images` | Importuje páry souborů Survey3: `.RAW`+`.JPG`, LATTICE `.tif` snímky a DAQ `.daq` záznamy; vrátí `count` (obrázky) a `scan_count` (záznamy). |
| `configure_processing` | `configure` | Změní předané nastavení a žádné další (hodnoty: [Podporované hodnoty](#supported-values)); `custom_settings` odešle celou konfiguraci tak, jak je zadána. |
| `get_project_settings` | `get_config` | Načte uložená nastavení zpracování, exportu a indexu otevřeného projektu. |
| `start_processing` | `process(wait=False)` | Spustí zpracování otevřeného projektu a okamžitě se vrátí. |
| `get_processing_status` | `get_status` | Vygeneruje zprávu `state` (`idle`, `starting`, `processing` nebo `finished`) a průběh; po dokončení se zapíší `summary`, `export_check_failure` v případě, že exporty nebylo možné ověřit, a `image_failure` v případě ztráty obrazů: ostatní výstupy se zapíší; běh se nahlásí jako neúspěšný. |
| `process_folder` | `process_folder` / `process_lattice_capture` | Vytvoří projekt pro složku, importuje jej, nakonfiguruje a spustí běh v jednom volání (`lattice: true` pro snímky LATTICE). Pokud již běh probíhá, operace se odmítne ještě předtím, než se cokoli vytvoří, a backend v tu chvíli rovněž odmítne změnu projektu z jakéhokoli nástroje. |
| `export_light_sensor` | `export_light_sensor` | Zapíše kalibrované soubory `.daq` a `.csv` pro záznamy světelného senzoru projektu bez zpracování snímků a načte tovární kalibraci z MAPIR Cloud, pokud není uložena v mezipaměti (viz [Záznamy světelného senzoru](#light-sensor-recordings--calibrated-daq--csv)). |
| `read_image_audit_tags` | `read_image_audit_tags` | Čte metadata auditu Chlorosu ze zpracovaného snímku (zdroj, kalibrace, použité fáze). |

**Kamery a pole**

| Nástroj | Volání SDK | Co dělá |
| --- | --- | --- |
| `list_devices` | `list_cameras`, `list_arrays`, `list_daq_sensors` | Co je aktuálně připojeno v libovolném klientovi, vždy s `connected_here`; plus úlohy spuštěné na serveru. |
| `discover_devices` | `discover_lattice_cameras`, `discover_daq_sensors` | Kamery a světelné senzory, které by mohly být připojeny. |
| `get_device_status` | `CameraSession.status`, `ArraySession.status` / `capability`, `DAQSensorSession.status` | Aktuální stav jedné kamery, pole kamer (včetně jeho funkčnosti a stavu) nebo světelného senzoru. |
| `analyze_array_network` | `analyze_array_network` | Co tento hostitel a síť mohou zajistit pro navrhované pole. |
| `connect_camera` / `connect_array` | `connect_camera` / `connect_array` | Připojte kameru nebo synchronizovanou sestavu (nejprve hlavní kameru). |
| `connect_project_devices` | `open_project(...).connect_all` | Připojte všechna zařízení, která si uložený projekt pamatuje, včetně jejich nastavení. |
| `disconnect_device` | `disconnect` / `force_disconnect` | Uvolněte kameru, pole nebo světelný senzor (viz Chování). `keep_settings: true` na poli uloží nastavení každé kamery do její vlastní paměti, `clear_settings: true` tuto paměť vynuluje (jeho výsledek nese `camera_memory`). |
| `disconnect_project_devices`, `align_project_arrays`, `capture_project_devices` | `disconnect_all`, `align_arrays`, `capture_all` | Zařízení projektu, dohromady. |
| `read_camera_settings` / `set_camera_settings` | `CameraSession.read_nodes` / `set_settings` | Čtení nebo zápis nastavení kamery podle popisného jména nebo názvu uzlu GenIC. |
| `set_array_frame_rate` | `ArraySession.set_target_fps` | Nastavení pevné snímkové frekvence pole nebo návrat k automatické snímkové frekvenci. |
| `capture_camera` / `capture_array` | `CameraSession.capture` / `ArraySession.capture` | Jedno zachycení (`fastest` a `wait_for_exposure` jsou `capture_fastest` a snímek s nastavenou expozicí). Pokud není k dispozici složka `output_dir`, uloží se do složky otevřeného projektusložku `captures` a soubory se připojí k projektu (zadrženy až do ukončení zpracování); odpověď `project_add` to potvrzuje. |
| `start_capture_run` | `ArraySession.continuous`, `capture_repeated` | Kontinuální nebo intervalové snímání jako úloha. |
| `start_array_recording` | `ArraySession.record` / `burst` | Video s kombinovaným indexem nebo nezpracovaná série snímků, jako úloha. |
| `build_burst_video` | `ArraySession.build_video` | Zpracování série snímků do kalibrovaného videa jako úloha. |
| `get_job_status` / `stop_job` | | Sledování nebo zastavení úlohy. |
| `get_camera_preview` | | Živý náhled z každé kamery, vrácený jako obrázek. |
| `grab_frames` | `ArraySession.grab`, `CameraHandle.grab` | Jeden snímek (nebo jedna synchronizovaná sada) na úrovni zpracování, bez uložení: statistiky pro každou rovinu a soubor PNG pro každou z nich. |
| `calibrate_array_alignment`, `get_array_alignment`, `clear_array_alignment`, `nudge_array_alignment`, `export_array_alignment`, `import_array_alignment` | `ArrayHandle.calibrate_alignment`, `alignment_status`, `clear_alignment`, `tweak_alignment`, `export_alignment`, `import_alignment` | Zarovnání pole. |

**Světelné senzory**

| Nástroj | Volání SDK | Co dělá |
| --- | --- | --- |
| `connect_light_sensor` | `connect_daq_sensor` | Připojení zařízení DAQ-U, DAQ-M nebo DAQ-E (`cap_id` nastavuje korekci krytu). |
| `get_light_sensor_reading` | `DAQSensorSession.latest` | Nejnovější naměřené hodnoty: časové razítko, saturace, rozsah spektra a maximum, a spektrum zprůměrované do vlnových binů (`full_spectrum` pro každý bod). |
| `set_light_sensor_streaming` | `stream_start` / `stream_stop` | Spuštění nebo zastavení datového proudu. |
| `start_light_sensor_recording` / `stop_light_sensor_recording` | `record_start` / `record_stop` | Zaznamenat soubor `.daq` jako úlohu; `duration_s` jej sám zastaví. |
| `set_light_sensor_cap` | `DAQSensorSession.set_cap` | Korekce krytu (`none` pro holý snímač). |
| `get_light_sensor_bindings` / `set_light_sensor_bindings` | `daq_bindings` / `set_daq_bindings` | Který snímač dodává údaje o odrazivosti jednotlivých kamer a matic. |
| `get_light_sensor_band_presets` / `get_light_sensor_reflectance` | `daq_band_presets` / `light_sensor_reflectance` | Sady pásem a odrazivost a indexy okolního prostředí a objektu ze dvou senzorů pro streamování. |
| `get_light_sensor_device_info`, `set_light_sensor_network`, `set_light_sensor_auto_stream`, `enable_light_sensor_ptp`, `reboot_light_sensor`, `update_light_sensor_firmware` | `device_info`, `set_network`, `set_auto_stream`, `enable_ptp`, `reboot`, `update_firmware` | Vlastní nastavení zařízení DAQ-E, uložené v jednotce. |

**Nástroje pro kamery** ([Nástroje pro kamery](#camera-utilities))

| Nástroj | Volání SDK | Co dělá |
| --- | --- | --- |
| `get_camera_color_settings` / `set_camera_color_settings` | `camera_color_profile`, `camera_color_adjust`, `camera_highlights` a jejich nastavovací funkce | Barvy zobrazení RGB kamery: profil a barevná teplota, sytost / kontrast, světlé oblasti. |
| `run_camera_white_balance` / `set_camera_white_balance` | `camera_white_balance_once` / `set_camera_white_balance` | Jednorázové vyvážení bílé, nebo automatické / vypnuté / pevné poměry. |
| `list_camera_presets` / `apply_camera_preset` | `camera_presets` / `apply_camera_preset` | Předvolby kamery. |
| `list_filter_models` | `camera_filter_models`, `camera_filter_model_info` | Modely filtrů, případně jeden podle názvu. |
| `get_camera_firmware_status` / `update_camera_firmware` | `camera_firmware_status` / `update_camera_firmware` | Verze firmwaru a flash (ve výchozím nastavení úloha). |
| `get_camera_power_info` / `set_camera_power` | `camera_power_probe`, `sleep_camera`, `wake_camera` | Stav napájení a teploty; režim spánku nebo probuzení. |
| `calibrate_camera_with_target` | `calibrate_camera_with_target` | Koeficienty z kalibračního terče v zorném poli. |
| `get_network_adapters` / `fix_network_settings` | `network_adapters` / `fix_network_settings` | Síťové adaptéry kamery, odhad a oprava (pouze pro správce). |
| `apply_vignette_correction` | `apply_vignette_correction` | Korekce vinětace ve složce s obrázky. |
| `read_dls` | `read_dls` | Jedno čtení DLS. |
| `probe_system` / `get_system_info` | `probe_system` / `system_info` | Hardware a runtime hostitele. |

**Přechodné kamery, profily zarovnání a indexy** ([Přechodná pole, profily zarovnání a indexy](#transient-arrays-alignment-profiles--indices)). Každý nástroj zde, který otevírá kamery, používá `keep_settings` / `clear_settings` (výchozí hodnota false): viz [Paměť kamery](#camera-memory--keep_settings--clear_settings).

| Nástroj | Volání SDK | Co dělá |
| --- | --- | --- |
| `start_camera_continuous` / `start_multi_camera_stream` | `start_camera_continuous` / `start_multi_camera_stream` | Nahrává jednu kameru nebo synchronizovaně streamuje několik kamer jako úlohu (`duration_s` ji sám zastaví). |
| `capture_cameras_transient` | `capture_cameras_transient` | Jedno synchronizované nahrávání bez připojení pole. |
| `test_array_sync` / `detect_array_master` | `test_array_sync` / `detect_array_master` | Kontrola synchronizace spouště a které kamery jsou na synchronizační lince. |
| `analyze_array_models` | `analyze_array_models` | Jaké výsledky by v této síti dosáhla sestava těchto modelů. |
| `calibrate_alignment_profile` | `calibrate_alignment_profile` | Vyřešení a uložení souboru profilu zarovnání (`options` obsahuje podrobnější možnosti řešiče). |
| `capture_aligned_multiband` / `start_aligned_stream` | `capture_aligned_multiband` / `start_aligned_stream` | Zarovnaný vícepásmový snímek nebo jejich proud jako úloha. |
| `read_alignment_profile` / `reorder_alignment_profile` | `alignment_profile_info` / `reorder_alignment_profile` | Obsah profilového souboru; pořadí pásem a povolené kamery. |
| `list_index_presets`, `compute_index`, `capture_live_index` | `index_catalog`, `compute_index`, `capture_live_index` | Předvolby a přechody indexu; index pro vícepásmový soubor TIFF; pořízení snímku a vytvoření indexu v jednom kroku. |

**Synchronizace času, projektová složka a recepty** ([PTP Grandmaster](#ptp-grandmaster--time_sync_status), [Projektová složka a recepty](#project-folder--recipes))

| Nástroj | Volání SDK | Co dělá |
| --- | --- | --- |
| `get_time_sync_status` | `time_sync_status`, `time_sync_peers`, `time_sync_cameras` | Stav PTP grandmastera, jeho peerů a PTP stav každé kamery v poli. |
| `restart_time_sync` / `set_time_sync_priority` | `restart_time_sync` / `set_time_sync_priority` | Restartujte jej; nastavte jeho priority. |
| `get_project_folder` / `set_project_folder` / `reset_project_folder` | `project_folder` / `set_project_folder` / `reset_project_folder` | Složka, ve které se vytvářejí nové projekty. |
| `run_project_recipe` | `run_project_recipe` | Spusťte recept uloženého projektu jako úlohu s příslušným protokolem. Nejprve se provede kontrola: recept s chybou je odmítnut a nic se nespustí; na nepřipojená pole / senzory se zobrazí varování (nepřipojí se: nejprve se spustí `connect_project_devices`). `dry_run` vrátí výsledek kontroly a nic nespustí. |

Neexistuje žádný nástroj pro otevření uloženého projektu ke zpracování: každá zpracovatelská úloha začíná příkazem `create_project` (nebo `process_folder`).

### Požadavky

- **Chloros nainstalovaný na stejném počítači.** Zpracování provádí backend Chloros. Server může místo toho použít backend dostupný přes tunel (`--backend-url` s `--no-auto-start`, viz [Možnosti](#options)).
- **Přihlášení na daném počítači**, v aplikaci Chloros nebo pomocí `chloros-cli login`. Server se za vás nemůže přihlásit: nástroj spuštěný při odhlášení odpoví „Nejste přihlášeni do Chloros“ s důvodem uvedeným backendem a asistent vás na to upozorní.
- **Jakýkoli placený tarif Chloros+ (Copper nebo vyšší).** Server využívá přístup přes SDK, zatímco bezplatný tarif Iron jej nemá: nástroj spuštěný v rámci bezplatného tarifu odpoví „Není k dispozici v tomto tarifu Chloros“ s uvedením důvodu ze strany backendu.
- Python 3.7 nebo novější s SDK a klient MCP, který spouští lokální (stdio) servery.

### Chování

- **Backend.** Spuštění serveru nebo zobrazení seznamu jeho nástrojů nic nespustí; teprve první volání nástroje naváže spojení. Pokud běží aplikace Chloros (nebo backendová služba pro Linux), server použije její backend a nechá jej běžet. Pokud není spuštěn žádný backend, server spustí nainstalovaný backend bez okna. Když server skončí (ukončíte klienta nebo klient restartuje server), zastaví úlohy snímání a záznamy ze světelných senzorů, které spustil, a uvolní připojená zařízení; backend zůstává spuštěný, takže nedokončený proces se dokončí až do konce a backend se sám zastaví po 30 minutách, pokud ho nic nevyužívá. U `--no-auto-start` server nikdy nespustí backend: volání nástroje, když žádný backend neběží, vás vyzve k otevření aplikace Chloros a další volání po jejím otevření naváže spojení.
- **Jeden otevřený projekt.** Backend má otevřený jeden projekt, který sdílí s aplikací Chloros, pokud je aplikace spuštěna, proto v aplikaci nepracujte na jiném projektu, zatímco asistent pracuje.
- **Změní se pouze předaná nastavení.** `configure_processing` ponechá každé nastavení, které není zadáno, na jeho uložené hodnotě (SDK `configure()` resetuje debayer, korekci vinětace, kalibraci odrazivosti, PPK a formát exportu na jejich výchozí hodnoty, pokud nejsou předány; server je doplní z projektu). Pokud nemůže načíst uložená nastavení projektu, nic nemění a oznámí to, pokud nejsou zadány všechny pět. `indices` nahradí indexový seznam projektu. Výsledek uvádí, co bylo odesláno jako `settings_sent` a pod `warnings` pojmenovává všechny indexy, které pásma kamery nemohou vytvořit.
- **Zpracování probíhá podle principu „start-then-poll“.** `start_processing` se vrátí, jakmile se spustí běh, který pak trvá několik minut až hodin, a asistent na něj navazuje voláním `get_processing_status`. `state` je buď `idle` (žádné zpracování), `starting` (backend ještě neohlásil běh, maximálně po dobu 20 s), `processing` nebo `finished`, s [souhrn](#post-run-summary--hints): co exportoval a upozornění, pokud exportoval méně, než se očekávalo (žádné snímky, nebyl nalezen kalibrační cíl). Výstupy se ukládají do projektové složky, stejně jako z aplikace. `start_processing` je odmítnut, pokud běh právě probíhá, a backend, který přestane odpovídat, je nahlášen jako chyba, nikoli jako dokončený běh.
- **Co vidí asistent.** Varování SDK se zobrazují ve výsledku nástroje pod kódem `warnings` a selhání (odhlášení, plán, nesprávná cesta) jako chybový text, který dokáže přečíst a předat vám. Seznam s více než 20 položkami je zkrácen na prvních 20, přičemž jeho plná délka je uvedena jako `<key>_total`. Cesty musí být absolutní.
- **Zařízení jsou sdílená.** Kamery, pole a světelné senzory se nacházejí na backendu, takže asistent může ovládat zařízení připojené k aplikaci Chloros a veškeré změny se v aplikaci zobrazí. `disconnect_device` vrací informace o tom, k čemu se server připojil; zařízení, ke kterému se připojil jiný klient (aplikace, `chloros-cli`, skript, další `chloros-mcp`) je uvolněno pouze pomocí `force: true`, což je `force_disconnect` v rámci SDK, a v případě odmítnutí se uvádí jméno toho, kdo zařízení připojil, pokud to backend zná; zařízení, které backend vůbec neobsahuje, je hlášeno jako nepřipojené (`released: false`, `connected: false`, není co uvolnit). Záznam nebo stream, který backend spouští, vede k zobrazení `get_job_status` s `counts` — `frames_written`, `frames_dropped`, `frames_failed`, `drop_reason` (skupiny snímků pro stream z více kamer) — a jednořádkový výpis `summary` („220 snímků zapsáno, 55 vynecháno, 0 selhalo: ...“) během běhu a po dokončení s konečnými čísly; výpis od jiného klienta (`other_clients_jobs`) obsahuje stejné údaje (`list_cameras()`, `list_arrays()` a `list_daq_sensors()` obsahují údaje pro každé zařízení`connected_by`: `client`, `label`, `name`, `at`). Každé zařízení `chloros-mcp` se v backendu označuje jako `chloros-mcp (pid N)` (`chloros_sdk.connect.CLIENT_LABEL`, odesláno jako `X-Chloros-Client-Label`; může to nastavit jakýkoli skript SDK). Zastavení záznamu ze světelného senzoru jiného klienta vyžaduje také `force`.
- **Dlouhá práce je úkol.** `start_capture_run`, `start_array_recording`, `build_burst_video`, `start_light_sensor_recording`, `start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`, `run_project_recipe`, a jakékoli zachycení, řešení nebo indexování požadované pro `background` okamžitě vrátí `job_id`; následuje `get_job_status` a `stop_job` to ukončí . Běh pokračuje, dokud není zastaven, pokud mu nezadáte `max_captures`, `duration_s` (nebo v případě dávkového zpracování `max_frames`); `min_free_gb` odmítne spustit, pokud má disk málo volného místa. Žádné omezení se neuplatňuje, pokud jej nenastavíte. Záznam nebo stream, který backend spouští (`start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`) si uchovává ID úlohy backendu, takže další `chloros-mcp` — spuštěný později — ji podle tohoto ID najde: `get_job_status` na něj navazuje (a vypisuje takové úlohy jako `other_clients_jobs`, od `GET /api/camera/cli-jobs`, každou s `started_by`), a `stop_job` ji zastaví pouze pomocí `force: true`. Jakákoli jiná úloha (zachytávací úloha, záznam do pole nebo burst, zachytávání na pozadí) běží v úloze `chloros-mcp`, která ji spustila, a pouze tento server o ní ví: jak potvrdí jiný server, který o něj požádal. Zastavená úloha čte `stopping` při vypisování své fronty, poté `finished` s konečnými počty, po dobu 10 minut a z jakéhokoli serveru `chloros-mcp`: ten, který to spustil, vidí vynucené zastavení jiného stejným způsobem. `get_job_status` nikdy nečeká na zastavení: během zastavení (svého vlastního `stop_job`, zastavení jiného klientanebo zastavení, které uvolňuje kamery z úlohy, která skončila sama) okamžitě odpoví pomocí `stopping` a aktuálních počtů, a `stop_job` z úlohy, která se již zastavuje, to oznámí. Úloha, která skončila, uvede důvod v `stop_reason`: vlastní důvod (`no_frames`, `disk_full`, `member_missing`, nebo u souvislého běhu pole `count`, `duration`, `array_gone`), `limit reached` pouze tehdy, když její počty ukazují její `count` (nebo `max_frames` u série, `duration_s` u záznamu)`duration_s`) dosáhlo, `error`, `stopped` (s `stopped_by`), pokud jej zastavil klient, jinak `finished`.
- **Kam se ukládají záznamy.** Do složky `captures` otevřeného projektu, kam je aplikace ukládá, pokud není zadán parametr `output_dir`.
- **Obrázky.** `get_camera_preview` a `grab_frames` vrací snímky, které si asistent může prohlédnout, vedle jejich JSON dat. Snímky `grab_frames` jsou pro prohlížení kontrastně upraveny; jejich statistické údaje představují skutečné hodnoty.
- **Při ukončení serveru** se zastaví úlohy, které spustil, a uzavře se záznam ze světelného senzoru, který spustil. Pokud server spustil backend, uvolní před jeho zastavením také zařízení, ke kterým se připojil; v backendu aplikace zůstávají připojena.
- **Protokol.** Pouze stdio (žádný HTTPový koncový bod), pouze nástroje (žádné zdroje ani výzvy), textové a obrazové výsledky, verze protokolu MCP od 5. 11. 2024 do 25. 11. 2025.**Co není zahrnuto.** Přihlášení nebo odhlášení (účet je váš; server se nemůže přihlásit) a zastavení backendu, který server nespustil. Na vše ostatní, co toto SDK dělá se zpracováním a hardwarem, existuje nástroj. (Provisioning zařízení DAQ-E – jeho název, autentizační token, heslo OTA, požadavek na podpis, obnovení továrního nastavení – a přímé servery DAQ jsou výhradně v kompetenci `chloros-cli`, a to i v rámci SDK: nesprávný název nebo token zablokuje každou instalaci Chloros v daném zařízení.)

---

## Automatické spuštění backendu

Vstupní body smart-connect — `connect_camera`, `connect_array`, `connect_daq_sensor`, `discover_lattice_cameras` a `discover_daq_sensors` — jsou jednoduchí HTTP klienti, kteří předpokládají, že backend naslouchá na adrese `127.0.0.1:5000` (výchozí URL rozhraní smart-connect). Pokud již běží grafické rozhraní (GUI) nebo příkazový řádek (CLI), jeden z nich již běží. Při spuštění z prostého skriptu k tomu nemusí dojít — proto tyto funkce **automaticky spustí přiložený binární soubor backendu** (bez okna, stejně jako to dělá `ChlorosLocal`) před svým prvním voláním a poté čekají až 60 s, než se spustí. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` a `analyze_array_network()` jej nikdy nespouštějí.

Pravidla:

- **Spouští se pouze lokální URL.** `backend_url` směřující na `localhost` / `127.0.0.1` / `[::1]` je přípustný; jakýkoli jiný hostitel je považován za stroj někoho jiného a nikdy není spuštěn.
- **Backend, který se ukončí dříve, než odpoví, ukončí čekání.** Volání se okamžitě vrátí s kódem ukončení, příkazem, který backend spustil, a posledními řádky, které backend vypsal, místo toho, aby čekalo 60 s. Výjimkou je kód ukončení 98: jiný backend již drží port nebo zámkový soubor (aplikace Chloros, backendová služba nebo jiný skript, který se ještě spouští), takže volání nadále čeká, až tento backend odpoví. `ChlorosLocal` se chová stejně.
- **Backend zůstává spuštěný pro opětovné použití** (stejně jako v CLI) — při ukončení skriptu nedochází k implicitnímu vypnutí. Při opětovném spuštění skriptu se znovu použije aktivní backend.
- **Tuto funkci lze deaktivovat pomocí `auto_start_backend=False`** u kteréhokoli z těchto volání (např. pokud jste nasměrovali na vzdálený backend nebo pokud životný cyklus backendu spravujete sami).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Pokud nelze najít nebo spustit přiložený binární soubor, následující volání HTTP vyvolá akční, **platformově specifickou** chybu `ChlorosConnectError` namísto prosté stopy o odmítnutí připojení — ve Windowsu vás nasměruje na desktopovou aplikaci nebo na příkaz `chloros-cli`; v Linuxu (bez grafického rozhraní) vás nasměruje na příkaz `chloros-cli` nebo na `.deb`.

---

## Prostředí a hlavičky

SDK označuje každé backendové HTTP volání kódem `X-Chloros-Client: sdk`. Backend uplatňuje licenční pravidla SDK/CLI (vyžaduje se přihlášení **a** placený tarif Chloros+), nikoli bezplatnou verzi pro grafické rozhraní. Toto nastavení se provede automaticky při importu – nemusíte nic dělat.

Záhlaví se přidá ke každému volání `requests` / `urllib`, jehož URL obsahuje `localhost` nebo `127.0.0.1`, na jakémkoli portu – včetně vašich vlastních lokálních služeb — a nahrazuje hodnotu, kterou jste nastavili sami. Volání na jiné hostitele zůstávají beze změny.

URLu backendu můžete přepsat předáním `backend_url=` (`ChlorosLocal` přijímá pouze `backend_url=` nebo `api_url=`; `process_folder()` a `process_lattice_capture()` přijímají pouze `api_url=`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Nainstalovaný backend naslouchá pouze na smyčce, takže se k němu nemůže připojit `backend_url` v síti LAN; viz [Režim vzdáleného backendu](#remote-backend-mode-pip-only-host-via-tunnel) pro vzor tunelu.)

---

## Verze a kompatibilita

- Verze SDKu je označena jako `chloros_sdk.__version__`.
- SDK vázá chování na verzi dodaného backendu. Kombinace staršího SDK s novějším backendem obvykle funguje (koncové body jsou kompatibilní s novějšími verzemi), ale kombinace novějšího SDK se starším backendem může na nových koncových bodech vyvolat chyby `404` — aktualizujte desktopovou aplikaci tak, aby odpovídala.
- Rozhraní smart-connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) a koncový bod pro analýzu sítě vrací stabilní schémata JSON; nová pole jsou doplňková.

---

## Tipy pro řešení problémů

- **`ChlorosAuthenticationError: Login required`** (z `ChlorosLocal`; volání funkce Smart Connect vyvolávají chybu `ChlorosConnectError: Login required. …` ze stejného důvodu) → Spusťte na tomto počítači jednou příkaz `chloros-cli login EMAIL PASSWORD` nebo se přihlaste prostřednictvím desktopové aplikace Chloros.
- **`ChlorosProcessingError: Failed to start processing: … The project folder is too deep for this computer …`** (nebo `Failed to create project: …` z `create_project()`) → Na tomto počítači jsou dlouhé cesty v systému Windows zakázány (výchozí nastavení systému Windows), proto žádná cesta nesmí přesáhnout 260 znaků a produkty se nacházejí až asi 130 znaků pod projektovou složkou. Nic nebylo zpracováno. Zpráva uvádí nejdelší projektovou složku, která se vejde: použijte kratší, nebo zapněte dlouhé cesty v systému Windows a restartujte Chloros. Instalační program Windows to nabízí jako možnost „Povolit dlouhé cesty ve Windows“, která je ve výchozím nastavení zaškrtnuta; jinak použijte zásady skupiny: *Konfigurace počítače > Šablony pro správu > Systém > Souborový systém > Povolit dlouhé cesty Win32*.
- **`ChlorosConnectError: No Chloros backend is running …`** → Funkce Smart-Connect volá automatické spuštění lokálního backendu, takže se tato chyba zobrazí pouze v případě, že nelze najít/spustit přiložený binární soubor (např. hostitel typu pipbez balíčku pro desktop). Zpráva se liší podle platformy: ve Windows otevřete desktopovou aplikaci nebo spusťte libovolný příkaz `chloros-cli`; v systému Linux spusťte příkaz `chloros-cli` (neexistuje žádné grafické rozhraní) nebo nainstalujte `.deb`. Pro vzdálený backend předávejte `backend_url=` (a `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** při importu → `lattice_sdk` se nepodařilo načíst: runtime Arena SDK nebo OpenCV (`cv2`) nelze v tomto Pythonu importovat. Povrch řízený backendem (`connect_*`, promítá se přes backend, `ChlorosLocal`) stále funguje.
- **Pole se připojuje při plném rozsahu snímače, ale s nízkou snímkovou frekvencí**→ Tak je to navrženo: režim auto zachovává plný rozsah snímače a nechá frekvenci klesnout na úroveň, kterou unese přenosové rozhraní. Pro více snímků za sekundu použijte `binning=2` / `4` nebo menší `width` / `height` pro vyšší počet snímků za sekundu, případně povolte jumbo rámce / vylepšete připojení. Binning a ROI**neřeší** celkovou nadměrnou přetíženost (`oversubscribed: true`, pole fps 0): příliš mnoho kamer pro dané připojení nelze vyřešit pomocí binningu/ROI — snižte počet kamer, povolte jumbo rámce nebo přejděte na rychlejší síťovou kartu (viz [Překročení kapacity](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` hlásí, že přijímací kruh síťové karty je velmi malý (~0,26 MB) / propojovací brány s hlášením „FRAMES WILL DROP“** → Přijímací kruh hostitelské síťové karty je na výchozí hodnotě (často se po aktualizaci ovladače síťové karty resetuje na 32). U adaptéru Realtek USB 10GbE nastavte `ReceiveBufferLen=256` a `PendingReceives=64` (s rozšířenými oprávněními) a poté restartujte backend, aby znovu načítal kruh. Kompletní postup: [Referenční příručka CLI → Nastavení a ladění hostitelské síťové karty](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Hostitel se zasekne při restartu/vypnutí, později se objeví chyby WMI `Invalid class` / síťová karta se neaktivuje** → Zastaralý ovladač USB 10GbE způsobuje chybu `DRIVER_POWER_STATE_FAILURE` (modrá obrazovka smrti BSOD `0x9F`). Aktualizujte ovladač adaptéru na aktuální verzi (≥ 2026) a znovu použijte nastavení přijímacího kruhu. Viz [Referenční příručka CLI→ Nastavení a ladění síťové karty hostitele](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Zachycení pole selhalo s chybou HTTPu 503 „No fresh synced frame group“** → Pole nezveřejnilo žádnou novou skupinu rámců během čekání. Zkontrolujte, zda `arr.status()`: `stalled: True` (nebo `streaming: False`) s `actual_fps`; hodnota 0 znamená, že smyčka zachycování přestala publikovat — odpojte a znovu připojte pole. Pokud je hodnota `stalled` False a hodnota `actual_fps` je větší než 0, zkuste to znovu. Hned po `connect_array()` již zachycování čeká až 20 s na první skupinu.
- **`status()` zobrazuje `link_speed_mbps: 4294`** → Systém Windowsu nemohl přečíst rychlost adaptéru; to je typické pro USB ethernetové adaptéry a nejedná se o chybu. Viz [Očekávané rychlosti rámců](#expected-frame-rates).
- **`update_firmware()` selže s `device never connected back on TCP <port>`** → Brána firewall zablokovala připojení zařízení DAQ-E zpět k tomuto počítači. Instalační program systému Windows přidá příslušné pravidlo (viz [Nastavení zařízení DAQ-E](#daq-e-device-settings)); u jiných brán firewall povolte příchozí TCP připojení k backendu ze podsítě senzoru.
- **Odrazivost odmítnuta** → Pro měření odrazivosti v absolutní škále musí být k kameře (nebo soustavě kamer) přiřazen aktivní DAQ. Proveďte přiřazení buď prostřednictvím grafického uživatelského rozhraní, nebo použijte `processing="radiance"` (W/m²/sr/nm), který nevyžaduje spárovaný senzor.
- **Zaznamenávání pomocí `smart=True` trvá déle, než se očekávalo** → Konvergence AE závisí na dynamice scény. U `ArrayHandle.capture_smart()` zvětšete `exposure_tolerance_pct`, zkrácení `stability_window_s` nebo snížení hodnoty `settle_timeout_s` pro rychlejší a méně ustálenou spoušť.

---

## Viz také

- [Referenční příručka CLI](cli-reference.md) — každý podpříkaz CLI odpovídá volání SDK.
- [Světelné senzory DAQ](https://mapir.gitbook.io/chloros/daq-light-sensors) v příručce k Chlorosu — senzory, jejich kryty a kalibrovaný rozsah, záznam, síťové propojení DAQ-E.
- [Kamery LATTICE](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) v příručce k systému Chloros — připojení, nastavení kamer, režimy snímání, pole.
- Online dokumentace: `https://mapir.gitbook.io/chloros/api-python-sdk`
