# Záložka DAQ v Chloros

Karta DAQ — označená jako **Světelné senzory** v postranním panelu Chloros — slouží jako ovládací rozhraní pro [světelné senzory DAQ-U, DAQ-M a DAQ-E](README.md): připojte senzory přes libovolný přenosový protokol, sledujte kalibrovaná spektra v reálném čase, vypočítávejte živou odrazivost z dvojice senzorů a zaznamenávejte soubory `.daq` přímo do vašeho projektu.

Karta se zpřístupní, jakmile se dokončí spouštění backendu Chloros. Grafy na této záložce jsou napájeny službou DAQ systému Chloros prostřednictvím živého připojení, které se v případě přerušení automaticky znovu naváže (s prodlevou 2–10 s); pokud je služba nedostupná, v řádku Stav senzoru se zobrazí **Žádný server**.

Rozložení tvoří **postranní panel senzorů**(jeden řádek pro každý připojený senzor) a**oblast grafů** (jedna dlaždice grafu pro každý senzor nebo skupinu).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Připojení senzoru

Klikněte na **Připojit senzor** v horní části postranní lišty. V hlavní oblasti se otevře dialogové okno pro připojení (nebo jako překryvná vrstva při přidávání dalšího senzoru – v takovém případě se zobrazí tlačítko Zrušit).

| Ovládací prvek | Chování |
| --- | --- |
| **Typ zařízení** | `DAQ-U (USB)` (výchozí), `DAQ-M (Bluetooth)` nebo `DAQ-E (Ethernet)`. Přepnutím se spustí nové vyhledávání pro nově vybraný způsob přenosu. |
| **Port / Zařízení BLE / Název hostitele / IP** | Zobrazuje seznam nalezených zařízení jako `device - description`; automaticky se vybere první položka rozpoznaná jako senzor. Během vyhledávání se zobrazuje `Scanning...` (USB), `Scanning (N)...` s 8sekundovým odpočítáváním (BLE) nebo `Discovering ethernet sensors (N)...` s 5sekundovým odpočítáváním (Ethernet). Prázdné výsledky se zobrazují jako `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Obnovit** | Okamžitě znovu prohledá vybraný transport (vypnuto během skenování BLE/Ethernetu). |
| **Připojit** | Aktivuje se po výběru zařízení; během navazování připojení se označení změní na `Connecting...`. |

Vyhledávání probíhá pouze **po dobu, kdy je na obrazovce zobrazeno dialogové okno pro připojení**, a opakuje se každých 15 sekund pouze pro vybraný přenosový protokol — pouhé otevření záložky nevyvolá skenování. V případě selhání se v dialogovém okně zobrazí: *„Připojení se nezdařilo. Zkuste odpojit a znovu připojit senzor a poté znovu klikněte na Připojit.“*

Postranní panel se automaticky otevře, jakmile se připojí váš první senzor.

{% hint style="info" %}
**DAQ-E se nezobrazuje?** DAQ-E nemá stavovou LED diodu — zkontrolujte indikátor PoE/propojení na přepínači nebo portu injektoru, ke kterému je připojen, a po zapnutí počkejte několik sekund, než se zařízení spustí. Zařízení Chloros musí být ve stejné vysílací doméně (mDNS neprochází přes směrovače). Na zařízení Windows potvrďte výzvu brány firewall Defender při prvním navázání multicastových soketů zařízením Chloros (mDNS UDP 5353, data DAQ-E UDP 5002, PTP UDP 319/320). Dvě jednotky DAQ-E v jedné síti LAN jsou detekovány samostatně, každá pod svým vlastním názvem hostitele `daq-e-<id>.local`.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Typ zařízení nabízí DAQ-U (USB), DAQ-M (Bluetooth) a DAQ-E (Ethernet)</figcaption></figure>***

## Postranní panel senzorů

Každý připojený senzor má jeden řádek (plus jeden řádek pro každou skupinu „Ambient+Object“). Řádky lze přetahováním měnit v pořadí a jejich pořadí zároveň mění pořadí dlaždic grafu. Kliknutím na řádek nastavíte daný senzor/skupinu jako aktivní graf v zobrazení seznamu.

| Prvek | Význam |
| --- | --- |
| Barevný levý okraj | Barva grafu daného senzoru. |
| Štítek přenosu | `DAQ-U` / `DAQ-M` / `DAQ-E`, případně zelený štítek `REF` pro skupinu odrazivosti Ambient+Object. |
| Název zařízení | Výchozí hodnota je sériové číslo senzoru (jeho stabilní identifikátor pro kalibraci, názvy souborů `.daq` a přiřazování při importu); vlastní názvy se uchovávají pro každý projekt. |
| **Kalibrováno** (zelená) | Zobrazuje se, když je načten tovární kalibrační balíček snímače, tj. spektra jsou v jednotkách W/m²/nm. |
| Ikona **K dispozici aktualizace** (jantarová, pouze DAQ-E) | Aktuální firmware je starší než obraz dodávaný s touto verzí Chloros. Během aktualizace zobrazuje průběh v reálném čase (`Flashing… N%`, `Restarting sensor…`, poté `Updated X → Y` nebo `Failed`). |
| Oko | Přepíná viditelnost tohoto senzoru v grafu. |
| Ozubené kolečko | Otevře modální okno nastavení pro jednotlivé senzory (níže). |
| ✕ (červené) | Odpojí senzor nebo odstraní skupinu „Ambient+Object“. |

Nad řádky se nacházejí dvě tlačítka:

* **Připojit snímač** — otevře dialogové okno pro připojení (během připojování se přejmenuje na `Connecting...`).
* **Zaznamenat vše / Zastavit vše**— spustí nebo zastaví záznam `.daq` na**všech**připojených senzorech. Vyžaduje alespoň jeden senzor**a otevřený projekt** (popisek: „Otevřete projekt pro záznam“); během jakéhokoli probíhajícího záznamu se tlačítko zbarví červeně.

V prázdném stavu se zobrazuje „Žádné připojené senzory“.

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Nastavení jednotlivých senzorů (modální okno s ozubeným kolečkem)

Otevře se kliknutím na ikonu ozubeného kolečka v řádku senzoru. Obsah v pořadí:

* **Řádky s informacemi** — Typ zařízení (DAQ-U/M/E), připojení (`Serial (USB)` / `Bluetooth` / `Ethernet`), port (COM port, adresa BLE nebo host) a sériové číslo.
* **Kalibrační zpráva: Stáhnout** — stáhne kalibrační certifikát tohoto přístroje s prokazatelným původem podle NIST (PDF) a otevře jej ve vašem prohlížeči PDF. K dispozici, jakmile je známo sériové číslo; certifikát se uloží do mezipaměti při prvním připojení.
* **Název zařízení** — kliknutím na tužku můžete změnit název; nastavení se uchovává pro daný projekt.
* **Barva čáry grafu** — vzorník barev; nastavení se uchovává pro daný projekt.
* **Integrační čas (ms)**— posuvník + číslo,**1–500 ms**, výchozí hodnota**32 ms**. Zakázáno, pokud je AE zapnuto.
* **Průměr snímků**— posuvník + číslo,**1–50 snímků**, výchozí hodnota**20**.
* **AE: ZAP/VYP**— přepínač automatické expozice;**výchozí nastavení ZAP** při připojení. Vypněte jej, chcete-li nastavit expoziční čas ručně.
* **Zastavit přenos / Spustit přenos** — pozastaví nebo obnoví živý přenos.
* **Nahrávat / Zastavit nahrávání** — nahrávání pro jednotlivé snímače `.daq` (vyžaduje otevřený projekt).
* **Cap** — profil korekce clony (další část).
* **Řádky s živými informacemi** — doba integrace (ms), FPS, vzorky, nahrávání (červená `REC` nebo `Off`) a stav (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Pouze DAQ-E: řádky týkající se sítě, firmwaru a PTP

* **Název hostitele / IP** — aktuální adresa jednotky.
* **Firmware** — aktuální verze firmwaru, plus akční buňka:<version\>

tlačítko</version\>

**Aktualizovat na \<version\>** se zobrazí, pokud tato sestavení Chloros obsahuje novější obraz firmwaru pro DAQ-E. Aktualizace se přes síť nainstaluje za přibližně 30 sekund; senzor se automaticky restartuje a znovu připojí, přičemž přerušený přenos ponechá aktuální firmware beze změny. Průběh aktualizace se zobrazuje v reálném čase (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`) a pole zobrazuje hodnotu `Up to date`, je-li aktuální.
* **Synchronizace PTP** — aktuální stav PTP (přepne se zpět na `unknown`). Firmware DAQ-E verze 1.2.0 a vyšší se účastní standardu IEEE 1588 PTPv2 pouze jako podřízený časovač; backend hostitele s adresou Chloros je PTP grandmaster a všechny kamery DAQ-E a LATTICE v síti LAN se k němu připojují jako slave v doméně 0, přičemž časové značky se udržují s přesností přibližně 1 ms.

U skupiny „Ambient+Object“ zobrazuje modální okno zařízení pouze zdrojové senzory skupiny, název zařízení a barvu grafické čáry.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->

### Výběr krytky

Rozbalovací nabídka **Cap** sděluje Chloros, který fyzický kryt je nasazen na difuzor senzoru, a na každé spektrum aplikuje korekční profil tohoto krytu naměřený při výrobě. Možnosti volby závisí na modelu:

| Model | Možnosti krytu |
| --- | --- |
| DAQ-U | Žádný (holý senzor), zorné pole 15°, zorné pole 30°, zorné pole 45°, zorné pole 60°, zorné pole 90°, Sunshine (kosinový korektor) |
| DAQ-M | Žádný (holý senzor), Sunshine (kosinový korektor) |
| DAQ-E | Žádný (holý senzor), zorné pole 15°, zorné pole 45°, zorné pole 90°, Sunshine (kosinový korektor) |

**Výchozí nastavení pro každý model je „Sunshine“ (kosinová korekce)** — Společnost MAPIR dodává každý DAQ s nainstalovaným krytem „Sunshine“, což je standardní konfigurace pro venkovní použití: 180° polokulový výhled s kosinovou chybou ≤ ±4 % do 60° a ≤ ±4,5 % do 70° (nedoporučuje se při elevaci Slunce pod ~15°), konstrukčně tlumený (~12×). Váš výběr zůstane v projektu zachován.

{% hint style="warning" %}
**Výběr krytky musí odpovídat fyzicky nasazené krytce.**Ani senzor, ani software nedokážou rozpoznat, která krytka je nasazena. Výběr ovlivňuje jak korekci v reálném čase, tak i údaj zapsaný do každého souboru `.daq` — vzhledem k ~12× útlumu krytu Sunshine způsobí neoznámená změna krytu nesprávnou korekci spektra přibližně o tento faktor. (Odstranění a opětovné nasazení stejného krytu vede k opakování odchylky přibližně na úrovni 1,5 %.) Volbu**None (holý snímač)** použijte pouze v případě, že je krytka fyzicky odstraněna; u přístroje DAQ-E se i při volbě „None“ stále aplikuje tovární geometrický profil pro jeho zapuštěný skleněný difuzor – nejedná se o nečinnost – a holý přístroj DAQ-E představuje laboratorní konfiguraci, nikoli podporovanou konfiguraci pro použití v terénu.
{% endhint %}

{% hint style="info" %}
Aktualizace z dřívějšího manuálu: přepínač „Sunshine Diffuser Installed“ na straně prohlížeče z verze 1.1.0 již neexistuje. Manipulace s krytkou se nyní řídí tímto profilem krytky pro každý senzor, který se aplikuje na straně serveru.
{% endhint %}

***

## Oblast grafů

V horní liště, která zůstává vždy viditelná, se nachází **přepínač mezi zobrazením seznamu a mřížky**a posuvník**zoomu grafu** (velikost dlaždic 200–2000 px). Zobrazení se automaticky přepne do mřížky, pokud existuje více než jedna skupina grafů, a zpět na seznam, pokud je jedna nebo méně. Režim zobrazení a velikost grafu se ukládají pro každý projekt.**Spektrální graf** pro každý senzor zobrazuje:

* **Osa X** — vlnová délka (nm). Mřížka senzoru je 340–1010 nm s roztečí 5 nm (135 bodů), pro zobrazení interpolovaná na 1 nm.
* **Osa Y** — výkon (W/m²), s automaticky zvolenou předponou SI (m/µ/n) na základě špičkové hodnoty. Spektra jsou na všech třech nosičích radiometricky kalibrovaná spektrální ozářenost (W/m²/nm).
* Duhové spektrální vybarvení pod jednotlivou křivkou; více senzorů na jednom grafu se překrývají jako barevné čáry s tlumeným vybarvením.
* **Přejetí myší**— svislý kurzor s vlnovou délkou a hodnotou pro jednotlivé senzory;**přetažením** lze měnit měřítko (při přiblížení se zobrazí tlačítko pro oddálení).
* Tlačítko **+** (pouze v mřížkovém zobrazení) pro přidání senzoru do tohoto grafu nebo vytvoření skupiny (viz níže).
* Název zařízení vycentrovaný nahoře a otočné kolečko, dokud nedorazí první snímek.

**Nasycení** není na samotném grafu označeno: nasycený senzor zobrazuje červený stavový text `SATURATED` a červený řádek `Saturated: Yes` v tabulce živých dat. Pro odstranění tohoto stavu zkraťte integrační čas nebo znovu zapněte automatickou expozici (AE).

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->

***

## Tabulka živých dat (zobrazení seznamu)

Pod grafem v zobrazení seznamu, aktualizováno každých 500 ms:

* **Všechny modely**: Vzorek barvy světla (sRGB z CIE XYZ), Nasycený (Ano/Ne), CIE 1931 X/Y/Z, Chromatickost x/y, CIE u′/v′, CCT (K), CRI (Ra), dominantní vlnová délka (nm), špičková vlnová délka (nm), čistota excitace, Duv, CIE L\*/a\*/b\* a Munsell H/V/C.
* **Pouze kalibrované senzory**(libovolný z modelů DAQ-U / DAQ-M / DAQ-E po načtení továrního kalibračního balíčku — poznáte to podle zelené značky**Kalibrováno** v řádku senzoru): Celkový výkon (W/m²), fotopický lux (lx), skotopický lux (lx), poměr S/P, PPFD a PPFD Red/Green/Blue (µmol/m²/s), a opické ozáření — S-čípek, melanopické, rhodopické, M-čípek, L-čípek (všechny v W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Skupiny odrazivosti (okolní světlo + objekt)

Dva propojené senzory lze zkombinovat do živého zobrazení odrazivosti — bez použití kamery:

1. V mřížkovém zobrazení klikněte na **+**na dlaždici grafu a vyberte**Sloučit okolí + objekt**.
2. Vyberte snímač **Zdroje okolního světla**a snímač**Skenér objektu**(dva různé snímače) a poté klikněte na**Vytvořit**.

Chloros vypočítává R(λ) = objekt(λ) / okolní světlo(λ) pro každou vlnovou délku ze dvou živých datových toků (0, pokud okolní světlo ≤ 0). Název skupiny se řídí kalibrační třídou senzorů:

* Oba senzory kalibrovány (balíček načten) → **„Zdánlivá odrazivost“**.
* Jeden ze senzorů není kalibrován → **„Relativní odrazivost“**.

Skupina se zobrazí jako zelený řádek `REF` v postranním panelu a má vlastní graf (duhová výplň, hodnoty se zobrazí po najetí myší s přesností na 4 desetinná místa, přetažením lze měnit měřítko).

Nabídka **+**nabízí také možnost**Přidat nový senzor** se třemi možnostmi umístění: *Sloučit s novým senzorem* (připojit k tomuto grafu), *Přesunout sem existující senzor* nebo *Zobrazit nový senzor* (vlastní graf).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->

### Tabulka vegetačních indexů

V seznamu se pod grafem skupiny odrazivosti nachází tabulka vegetačních indexů, vypočítaná z aktuální odrazivosti ve středních hodnotách pásem **modrá 450 / zelená 550 / červená 670 / NIR 800 nm** (hodnoty na 4 desetinná místa, `---`, pokud nelze vypočítat; najedete-li kurzorem na název indexu, zobrazí se jeho úplný název):

* **Vždy zobrazeno** (nezávislé na měřítku, libovolná kombinace senzorů): NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR.
* **Pouze pokud jsou oba senzory kalibrovány podle výkonu** (oba balíčky načteny): EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI.

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

***

## Záznam souborů `.daq`

* K záznamu je nutný **otevřený projekt** — v opačném případě jsou tlačítka „Záznam všeho“ (v postranním panelu) i „Záznam“ u jednotlivých senzorů deaktivována.
* Soubory se ukládají do složky **`<project folder>/light_sensor/`**; názvy souborů obsahují ID senzoru a časové razítko a název zařízení se ukládá spolu se záznamem.
* Po zastavení záznamu (tlačítkem „Zastavit“, „Zastavit vše“ nebo v případě odpojení během záznamu) se dokončený soubor `.daq` **automaticky přidá do otevřeného projektu** — objeví se v seznamu souborů projektu bez nutnosti ručního přidání a je připraven sloužit jako data o dopadajícím záření pro [zpracování odrazivosti](README.md).
* Během záznamu se v živých řádcích modálního okna nastavení zobrazuje červený indikátor `REC`.

Pro získání kvantitativních hodnot ozáření je třeba provést průměrování dat z alespoň 15 sekund — jedná se o charakteristiku přístroje, nikoli o vadu.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->

***

## Rozložení více senzorů a trvalost projektu

* Kombinujte několik senzorů do jednoho grafu (sdílené osy), udržujte samostatné grafy (automatické rozložení mřížky), přesouvejte senzory mezi grafy, přetahujte a měňte pořadí řádků/dlaždic a skryjte jednotlivé senzory pomocí přepínače ve tvaru oka.
* V rámci každého projektu se u Chloros uchovávají: názvy zařízení, barvy grafů, velikost grafu, režim zobrazení a nastavení každého senzoru (integrační čas, průměrování snímků, stav AE, výběr limitu).
* **Při opětovném otevření projektu se senzory automaticky znovu připojí** podle adresy — COM port pro DAQ-U, zařízení BLE pro DAQ-M, název hostitele mDNS pro DAQ-E (vyřeší se i v případě změny IP adresy jednotky) — a znovu se použije uložený profil krytu, průměrování snímků, stav AE a ručně nastavená doba integrace každého senzoru.***

## Spárování kamery (DLS)

Není co spárovat. Na rozdíl od pracovních postupů DLS u dronů, které předem přiřazují světelný senzor ke kameře, Chloros přiřazuje data DAQ ke snímkům až dodatečně: v okamžiku importu/zpracování se hodnoty z `.daq` interpolují k časovému razítku expozice každého snímku. Nahrávejte s jakýmkoli připojeným senzorem (`.daq` se do projektu přidá automaticky) a zpracování odrazivosti najde správné hodnoty podle času — viz [Světelné senzory DAQ](README.md), kde se dozvíte, jak se používají data o dopadajícím světle.</version\>
