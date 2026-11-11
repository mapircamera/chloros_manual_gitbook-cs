# Pracovní postupy pro výpočet odrazivosti

Světelný senzor DAQ převádí radiometrické snímky na odrazivost. Existují dva odlišné pracovní postupy:

1. **Jeden senzor** — jeden senzor DAQ měří dopadající ozáření, zatímco kamera pořizuje snímek, a program Chloros vydělí jas z kamery touto referenční hodnotou.
2. **Dva senzory** — dva senzory DAQ, z nichž jeden sleduje oblohu a druhý objekt, vytvářejí živou spektrální křivku odrazivosti bez zapojení kamery.

## Jeden senzor + kamera (referenční hodnota dopadajícího záření)

Senzor DAQ funguje jako senzor dopadajícího světla (DLS): kamera měří radianci směřující nahoru **L**(W/m²/sr/nm), DAQ měří ozáření směřující dolů**E** (W/m²/nm) a Chloros vypočítá odrazivost pro jednotlivá pásma podle vzorce:

> ρ = π · L / E

Hodnota naměřená DAQ je vždy **časově synchronizována s expozicí** — proto DAQ a kamery sdílejí hodiny řízené protokolem PTP (viz [Síťové propojení a synchronizace času DAQ-E](ethernet-ptp.md)). Pro práci venku nasaďte kosinovou čepici Sunshine a správně ji deklarujte; deklarace čepice přímo škáluje hodnotu E (viz [Profily čepic a kalibrovaný rozsah](caps-and-range.md)). Pro kvantitativní měření mějte na paměti charakteristiku přístroje: kvantitativní ozáření se vypočítává jako průměr naměřených hodnot za minimálně 15 s.

### Snímání v reálném čase

Propojte DAQ s kamerou na záložce „Kamery“: panel nastavení každé kamery obsahuje rozevírací nabídku **Světelný senzor**, která obsahuje seznam všech připojených DAQ (DAQ-U/M/E) ze záložky „Světelné senzory“; u synchronizovaného pole se výběr světelného senzoru pro celé pole promítne do každého člena (jednotlivé kamery mohou stále provést přepsání). Po přiřazení se spektra senzoru přenášejí do slotu DLS kamery a exportované hodnoty odrazivosti se dělí odpovídající naměřenou hodnotou. V poli používá člen s vlastním senzorem svou vlastní naměřenou hodnotu — pro svá pásma v živém kompozitu, své exporty a automatickou expozici — a ostatní členové používají hodnoty celého pole.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->


Dvě chování, která stojí za zmínku:

* **Žádné připojení k DAQ → odrazivost je odmítnuta, není simulována.** Program Chloros odmítne výstup odrazivosti a zaznamená důvod přeskočení, místo aby tiše vrátil méně přesný výstup.
* **Použitá naměřená hodnota je zachována.** Pro každý snímek odrazivosti se skutečně použité naměřené hodnoty z DAQ zapíší jako soubor `.daq` do složky `daq/` daného záznamu, takže záznam lze později znovu zpracovat ([Záznam a formát .daq](recording.md)). Soubor `.daq.cameras.json` vedle něj uvádí názvy kamer, které popisuje, a při opětovném zpracování se přiřadí pouze k snímkům těchto kamer.

### Zpracování zaznamenaných snímků

Pro zpracování po letu nahrajte během relace soubor `.daq` a uložte jej společně se snímky — zpracovatelský řetězec automaticky přiřadí snímky s odpovídajícím časovým razítkem a při prvním použití načte chybějící tovární kalibraci z cloudu společnosti MAPIR. Záznamy z grafického uživatelského rozhraní se po ukončení automaticky přidají do otevřeného projektu.

Referenční hodnotu odrazivosti lze vybrat v okamžiku zpracování – `--reflectance-source` na `chloros-cli process`, nebo nastavení zdroje odrazivosti v nastavení projektu v grafickém uživatelském rozhraní:

| Hodnota | Chování |
| --- | --- |
| `auto` (výchozí) | Absolutní referencí je kalibrační cíl v rámci snímku, který prošel kontrolou kvality (QA); záložní volbou je sestupný tok DAQ (ρ = π·L/E). Cíl, který prošel kontrolou kvality s výhradou (vypadlý panel nebo nefunkční pásmo), není automaticky upřednostňován: řetězce cíl, cíl+DAQ a pouze DAQ jsou zkoušeny v pořadí podle jejich naměřeného skóre na vyřazených panelech. |
| `daq` | DAQ jako rozhodující |
| `target` | Přísný cíl v rámci snímku; žádná náhrada DAQ |

Viz [Kalibrační terče](../calibration-targets.md) pro pracovní postupy s terči a [kapitolu LATTICE](../lattice/README.md) a [referenci CLI](../reference/cli-reference.md) pro kompletní zpracovatelský postup. Při čtení exportovaných pixelů odrazivosti použijte označenou stupnici (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3: 65535) — viz [Formáty výstupních obrázků](../output-image-formats.md).

### Pásma mimo kalibrovaný rozsah DAQ

Radiometricky kalibrovaný rozsah DAQ je ~374–974 nm. Chloros odmítá odrazivost založenou na DAQ pro jakékoli pásmo kamery, u kterého je méně než polovina jeho spektrální váhy v tomto rozmezí, a hlásí důvod vynechání `dls-uncalibrated-band-<nm>`. Z aktuálně dodávaných modelů se to týká pouze modelu F988: odrazivost kamery F988 je kalibrována pomocí panelu odrazivosti umístěného v měřené scéně; dané pásmo leží mimo kalibrovaný rozsah světelného senzoru DAQ, proto Chloros použije váš nejnovější záznam panelu a zachová jej mezi jednotlivými měřeními panelu. Pokud je kamera F988 provozována pouze v režimu DAQ, Chloros odmítne odrazivost založenou na DAQ pro dané pásmo s důvodem přeskočení `dls-uncalibrated-band-988` — podporovaným postupem je práce s panelem.

## Dva senzory (okolní světlo + objekt)

Dva senzory DAQ – libovolná dvojice, napříč jakýmikoli přenosy – poskytují živé spektrum odrazivosti bez kamery: jeden senzor směřuje k obloze (**zdroj okolního světla**), druhý k objektu (**skener objektu**) a program Chloros provádí výpočet pro každou vlnovou délku:

> R(λ) = objekt(λ) / okolní(λ)

(nula, pokud okolní světlo ≤ 0).

### V grafickém uživatelském rozhraní

Jakmile jsou oba senzory připojeny na záložce Světelné senzory, otevřete překryvné okno pro přidání senzoru (tlačítko „+“ na dlaždici grafu v mřížkovém zobrazení) a vyberte možnost **Kombinovat okolní světlo + objekt**. Z rozevíracích seznamů**Zdroje okolního světla**a**Skenér objektu**vyberte oba senzory a klikněte na**Vytvořit**. Skupina se zobrazí jako samostatný graf a jako řádek v postranním panelu se zelenou značkou**REF**.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->


Pod grafem odrazivosti (zobrazení seznamu) se v reálném čase vypočítávají indexy **vegetačního indexu** na základě křivky s využitím středů spektrálních pásem: modrá 450 / zelená 550 / červená 670 / NIR 800 nm. Vždy se zobrazují indexy založené na poměru, které ruší absolutní měřítko (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR) se zobrazují vždy; indexy, které vyžadují absolutní odrazivost (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI), se zobrazují pouze v případě, že oba senzory jsou modely s kalibrací výkonu.

### Zdánlivá vs. relativní — pravidlo označování

Chloros označuje výstup dvou senzorů podle toho, co může daná dvojice senzorů skutečně poskytnout:

| Dvojice senzorů | Označení |
| --- | --- |
| Oba senzory kalibrovány — načtený tovární kalibrační balíček | **Zdánlivá odrazivost** |
| Jeden ze senzorů není kalibrován | **Relativní odrazivost** |

Všechny tři modely jsou radiometrické: jakmile je načtena tovární kalibrační sada senzoru, jeho spektra jsou vyjádřena v absolutních hodnotách W/m²/nm, takže poměr mezi dvěma kalibrovanými senzory odpovídá absolutní zdánlivé odrazivosti — transportní model o tom nerozhoduje. Senzor, který stále vysílá surová měření (balíček není dostupný), převede výsledek na relativní křivku (spektrální tvar zůstává platný). Oba senzory by měly mít správně deklarované limity ([Profily limitů a kalibrovaný rozsah](caps-and-range.md)).

### Z jazyka Python


V sjednoceném rozhraní SDK neexistuje žádná specializovaná funkce pro práci se dvěma senzory: otevřete dvě relace pomocí `chloros_sdk.connect_daq_sensor()` a poměry jejich spekter `latest()` si vypočítejte sami, přičemž použijte stejnou konvenci označování. (Nástroj pro záznam s dvěma senzory existuje také v interním rozhraní MAPIR pro přímé ovládání hardwaru, který je pro úplnost uveden v [CLI Reference](../reference/cli-reference.md) — není však součástí dodávaného CLI; podporovaným postupem je výše uvedený pracovní postup v grafickém uživatelském rozhraní.)
