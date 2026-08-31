# Pracovní postupy pro měření odrazivosti

Světelný senzor systému DAQ převádí radiometrické snímky na hodnoty odrazivosti. Existují dva odlišné pracovní postupy:

1. **Jeden senzor** — jeden senzor DAQ měří intenzitu slunečního záření dopadajícího na zem, zatímco kamera pořizuje snímky, a Chloros vydělí jas kamery touto referenční hodnotou.
2. **Dva senzory** — dva senzory DAQ, z nichž jeden sleduje oblohu a druhý objekt, vytvářejí živou spektrální křivku odrazivosti bez zapojení kamery.

## Jeden senzor + kamera (referenční hodnota dopadajícího záření)

Senzor DAQ funguje jako senzor dopadajícího světla (DLS): kamera měří radianci směřující nahoru **L**(W/m²/sr/nm), DAQ měří ozáření směřující dolů**E** (W/m²/nm) a Chloros vypočítá odrazivost pro jednotlivá pásma podle vzorce:

> ρ = π · L / E

Hodnota naměřená DAQ je vždy **časově synchronizována s expozicí** — proto DAQ a kamery sdílejí hodiny řízené protokolem PTP (viz [Síťové propojení a synchronizace času DAQ-E](ethernet-ptp.md)). Pro práci venku si nasaďte sluneční kosinovou čepici a správně ji deklarujte; deklarace čepice přímo ovlivňuje hodnotu E (viz [Profily čepic a kalibrovaný rozsah](caps-and-range.md)). Pro kvantitativní měření mějte na paměti charakteristiku přístroje: kvantitativní ozáření se vypočítá jako průměr naměřených hodnot z období nejméně 15 s.

### Snímání v reálném čase

Propojte DAQ s kamerou na záložce „Kamery“: panel nastavení každé kamery obsahuje rozevírací nabídku **Světelný senzor**, která uvádí všechny připojené DAQ (DAQ-U/M/E) ze záložky „Světelné senzory“; u synchronizovaného pole se výběr světelného senzoru pro celé pole přenese na všechny členy (jednotlivé kamery mohou stále provést přepsání). Po přiřazení se spektra senzoru přenášejí do slotu DLS kamery a exportované hodnoty odrazivosti se dělí odpovídajícím naměřeným hodnotám.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->

Dvě chování, která stojí za to znát:

* **Není přiřazeno žádné zařízení DAQ → odrazivost je odmítnuta, není simulována.** Chloros odmítne produkt odrazivosti a zaznamená důvod vynechání, místo aby tiše vrátil produkt nižší kvality.
* **Použitá hodnota se zachová.** U každého snímku odrazivosti se skutečně použité naměřené hodnota DAQ zapíše jako sidecar `.daq` vedle snímku, takže záznam lze později znovu zpracovat ([Záznam a formát .daq](recording.md)).

### Zpracování zaznamenaných snímků

Pro zpracování po letu zaznamenejte během relace soubor `.daq` a uložte jej společně se snímky — zpracovatelský řetězec automaticky vyřeší sestupné záření s odpovídajícím časovým razítkem a při prvním použití načte chybějící tovární kalibraci z cloudu MAPIR. Záznamy z grafického uživatelského rozhraní se po ukončení automaticky přidají do otevřeného projektu.

Referenční odrazivost lze vybrat v okamžiku zpracování — `--reflectance-source` na `chloros-cli process`, nebo nastavení zdroje odrazivosti v nastaveních projektu v grafickém uživatelském rozhraní:

| Hodnota | Chování |
| --- | --- |
| `auto` (výchozí) | Absolutní referencí je kalibrační cíl v rámci snímku, který prošel kontrolou kvality (QA); záložním řešením je sestupný tok DAQ (ρ = π·L/E) |
| `daq` | DAQ jako rozhodující |
| `target` | Přísný cíl v rámci snímku; žádná náhrada pomocí DAQ |

Viz [Kalibrační cíle](../calibration-targets.md) pro pracovní postupy s cíli a [kapitola LATTICE](../lattice/README.md) a [CLI Reference](../reference/cli-reference.md) pro kompletní zpracovatelský postup. Při čtení exportovaných pixelů odrazivosti použijte označenou stupnici (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3: 65535) — viz [Formáty výstupních obrázků](../output-image-formats.md).

### Pásma mimo kalibrovaný rozsah DAQ

Radiometricky kalibrovaný rozsah DAQ je ~374–974 nm. Chloros odmítá odrazivost založenou na DAQ pro jakékoli pásmo kamery, u kterého je méně než polovina jeho spektrální váhy v tomto rozmezí, a hlásí důvod vynechání `dls-uncalibrated-band-<nm>`. Z dodávaných modelů se to týká pouze modelu F988: odrazivost kamery F988 je kalibrována pomocí panelu odrazivosti umístěného v měřené scéně; dané pásmo leží mimo kalibrovaný rozsah světelného senzoru DAQ, takže Chloros použije váš nejnovější záznam panelu a zachová jej mezi jednotlivými měřeními panelu. Pokud je kamera F988 provozována pouze v režimu DAQ, kód Chloros odmítne odrazivost založenou na DAQ pro dané pásmo s důvodem přeskočení `dls-uncalibrated-band-988` — podporovaným postupem je práce s panelem.

## Dva senzory (okolní světlo + objekt)

Dva senzory DAQ — libovolná dvojice, napříč libovolnými transporty — poskytují živé spektrum odrazivosti bez kamery: jeden senzor směřuje k obloze (**zdroj okolního světla**), druhý směřuje k objektu (**skener objektu**) a Chloros provádí výpočet pro každou vlnovou délku:

> R(λ) = objekt(λ) / okolní(λ)

(nula, pokud okolní ≤ 0).

### V grafickém uživatelském rozhraní

Po připojení obou senzorů na kartě Světelné senzory otevřete překryvné okno pro přidání senzoru (tlačítko „+“ na dlaždici grafu v mřížkovém zobrazení) a vyberte možnost **Kombinovat okolní světlo + objekt**. Z rozevíracích seznamů Ambient Light Source a Object Scanner vyberte oba senzory a klikněte na Create. Skupina se zobrazí jako samostatný graf a jako řádek v postranním panelu se zelenou značkou**REF**.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->

Pod grafem odrazivosti (zobrazení seznamu) vypočítává živá **tabulka vegetačních indexů** indexy z křivky pomocí středů pásem na modré 450 / zelené 550 / červené 670 / NIR 800 nm. Indexy založené na poměrech, které ruší absolutní měřítko (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR) se zobrazují vždy; indexy, které vyžadují absolutní odrazivost (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) se zobrazují pouze v případě, že oba senzory jsou modely s kalibrací podle výkonu.

### Zdánlivá vs. relativní — pravidlo pro označování

Chloros označuje výstup duálního senzoru podle toho, co může daná dvojice senzorů skutečně poskytnout:

| Dvojice senzorů | Označení |
| --- | --- |
| Oba senzory kalibrovány — načtený tovární kalibrační balíček | **Zdánlivá odrazivost** |
| Jeden ze senzorů není kalibrován | **Relativní odrazivost** |

Všechny tři modely jsou radiometrické: jakmile je načtena tovární kalibrační sada senzoru, jeho spektra jsou vyjádřena v absolutních hodnotách W/m²/nm, takže poměr mezi dvěma kalibrovanými senzory udává absolutní zdánlivou odrazivost — transportní režim o tom nerozhoduje. Senzor, který stále vysílá surová měření (balíček není dostupný), snižuje výsledek na relativní křivku (tvar spektra zůstává platný). Oba senzory by měly mít správně deklarované limity ([Profily limitů a kalibrovaný rozsah](caps-and-range.md)).

### Z Python

V sdíleném rozhraní SDK neexistuje speciální volání pro dva senzory: otevřete dvě relace pomocí `chloros_sdk.connect_daq_sensor()` a sami porovnejte jejich spektra `latest()`, přičemž použijte stejnou konvenci označování. (Nástroj pro záznam z dvou senzorů existuje také na interním hardwarovém rozhraní MAPIR, uvedené v [Referenci k CLI](../reference/cli-reference.md) – není však součástí dodávaného CLI; výše uvedený pracovní postup v grafickém uživatelském rozhraní je podporovanou cestou.)
