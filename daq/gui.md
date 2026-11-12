# Záložka DAQ v programu Chloros

Záložka DAQ — v postranním panelu programu Chloros označená jako **Světelné senzory** — slouží jako ovládací rozhraní pro [světelné senzory DAQ-U, DAQ-M a DAQ-E](README.md): připojte senzory přes libovolný přenosový protokol, sledujte kalibrovaná spektra v reálném čase, vypočítávejte živou odrazivost z dvojice senzorů a zaznamenávejte soubory `.daq` přímo do svého projektu.

Záložka se zpřístupní, jakmile se dokončí spouštění backendu Chloros. Grafy na této záložce jsou napájeny službou DAQ Chloros prostřednictvím živého připojení, které se v případě přerušení automaticky znovu naváže (s prodlevou 2–10 s); pokud je služba nedostupná, v řádku Stav senzoru se zobrazí **Žádný server**.

Rozložení tvoří **postranní panel senzorů**(jeden řádek pro každý připojený senzor) a**oblast grafů** (jedna dlaždice grafu pro každý senzor nebo skupinu).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Připojení senzoru

Klikněte na **Připojit senzor** v horní části postranní lišty. V hlavní oblasti se otevře dialogové okno pro připojení (nebo jako překryvné okno při přidávání dalšího senzoru – v takovém případě se zobrazí tlačítko Zrušit).

| Ovládací prvek | Chování |
| --- | --- |
| **Typ zařízení** | `DAQ-U (USB)` (výchozí), `DAQ-M (Bluetooth)` nebo `DAQ-E (Ethernet)`. Přepnutím se spustí nové vyhledávání pro nově vybraný přenosový protokol. |
| **Port / Zařízení BLE / Název hostitele / IP** | Zobrazuje seznam nalezených zařízení jako `device - description`; automaticky se vybere první položka rozpoznaná jako senzor. Během vyhledávání se zobrazuje `Scanning...` (USB), `Scanning (N)...` s 8sekundovým odpočítáváním (BLE) nebo `Discovering ethernet sensors (N)...` s 5sekundovým odpočítáváním (Ethernet). Prázdné výsledky se zobrazují jako `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Obnovit** | Okamžitě znovu prohledá vybraný přenosový kanál (během skenování BLE/Ethernetu je tato funkce deaktivována). |
| **Připojit senzor** | Aktivuje se po výběru zařízení; během navazování připojení se označení změní na `Connecting...`. |

Vyhledávání probíhá pouze **po dobu, kdy je na obrazovce zobrazen dialog pro připojení**, a opakuje se každých 15 sekund pouze pro vybraný přenosový kanál — pouhé otevření záložky nevyvolá skenování. V případě selhání se v dialogu zobrazí: *„Připojení se nezdařilo. Zkuste odpojit a znovu připojit senzor a poté znovu klikněte na Připojit.“*

Postranní panel se automaticky rozbalí, jakmile se připojí váš první senzor.

{% hint style="info" %}
**DAQ-E se nezobrazuje?**DAQ-E nemá stavovou LED diodu — zkontrolujte indikátor PoE/propojení na přepínači nebo portu injektoru, ke kterému je připojen, a po zapnutí počkejte několik sekund, než se zařízení spustí. Zařízení Chloros musí být ve stejné vysílací doméně (mDNS neprochází přes směrovače). Pravidlo firewallu**Chloros Backend (UDP-In)** v instalačním programu povoluje mDNS (UDP 5353), data DAQ-E (UDP 5002) a PTP (UDP 319/320); u firewallu jiného výrobce je třeba povolit stejné porty. Dvě jednotky DAQ-E v jedné síti LAN jsou detekovány samostatně, každá pod vlastním názvem hostitele `daq-e-<id>.local`.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Typ zařízení nabízí DAQ-U (USB), DAQ-M (Bluetooth) a DAQ-E (Ethernet)</figcaption></figure>

***

## Postranní panel senzorů

Každý připojený senzor má jeden řádek (plus jeden řádek na skupinu „Ambient+Object“). Řádky lze přetahováním měnit v pořadí a jejich pořadí zároveň mění pořadí dlaždic grafu. Kliknutím na řádek nastavíte daný senzor/skupinu jako aktivní graf v seznamu.

| Prvek | Význam |
| --- | --- |
| Barevný levý okraj | Barva grafu daného senzoru. |
| Štítek přenosu | `DAQ-U` / `DAQ-M` / `DAQ-E`, případně zelený štítek `REF` pro skupinu odrazivosti Ambient+Object. |
| Název zařízení | Výchozí hodnota je sériové číslo senzoru (jeho stabilní identifikátor pro kalibraci, názvy souborů `.daq` a přiřazování při importu); vlastní názvy zůstávají zachovány pro každý projekt. |
| Ikona **Kalibrováno** (zelená) | Zobrazuje se, když je načten tovární kalibrační balíček snímače, tj. spektra jsou v jednotkách W/m²/nm. |
| Ikona **K dispozici aktualizace** (jantarová, pouze DAQ-E) | Aktuální firmware je starší než obraz dodaný s touto verzí Chloros. Během aktualizace zobrazuje průběh v reálném čase (`Flashing… N%`, `Restarting sensor…`, poté `Updated X → Y` nebo `Failed`). |
| Oko | Přepíná viditelnost tohoto senzoru v grafu. |
| Ozubené kolečko | Otevře modální okno nastavení pro jednotlivé senzory (níže). |
| ✕ (červené) | Odpojí senzor nebo odstraní skupinu „Ambient+Object“. |

Nad řádky se nacházejí dvě tlačítka:

* **Připojit senzor** — otevře dialogové okno pro připojení (během činnosti se přejmenuje na `Connecting...`).
* **Zaznamenat vše / Zastavit vše**— spustí nebo zastaví záznam `.daq` na**všech**připojených senzorech. Vyžaduje alespoň jeden senzor**a otevřený projekt** (popisek: „Otevřete projekt pro záznam“); během jakéhokoli probíhajícího záznamu se tlačítko zbarví červeně.

V prázdném stavu se zobrazuje text „Žádné připojené senzory“.

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Nastavení jednotlivých senzorů (modální okno s ozubeným kolečkem)

Otevřete pomocí ikony ozubeného kola v řádku snímače. Obsah v pořadí:

* **Řádky s informacemi** — Typ zařízení (DAQ-U/M/E), Připojení (`Serial (USB)` / `Bluetooth` / `Ethernet`), port (COM port, adresa BLE nebo host) a sériové připojení.
* **Kalibrační zpráva: Stáhnout** — stáhne kalibrační certifikát tohoto zařízení s odkazem na NIST (PDF) a otevře jej ve vašem prohlížeči PDF. K dispozici, jakmile je známo sériové číslo; certifikát se při prvním připojení uloží do mezipaměti.
* **Název zařízení** — kliknutím na tužku můžete změnit název; nastavení zůstává zachováno pro daný projekt.
* **Barva čáry grafu** — vzorník barev; nastavení zůstává zachováno pro daný projekt.
* **Integrační čas (ms)**— posuvník + číslo,**1–500 ms**, výchozí hodnota**32 ms**. Je deaktivováno, pokud je zapnuta automatická expozice (AE).
* **Průměr snímků**— posuvník + číslo,**1–50 snímků**, výchozí hodnota**20**.
* **AE: ZAP/VYP**— přepínač automatické expozice;**výchozí nastavení ZAP** při připojení. Vypněte jej, chcete-li nastavit dobu integrace ručně.
* **Zastavit přenos / Spustit přenos** — pozastaví nebo obnoví živý přenos.
* **Nahrávat / Zastavit nahrávání** — nahrávání `.daq` pro jednotlivé senzory (vyžaduje otevřený projekt).
* **Cap** — profil korekce cap (další část).
* **Řádky s živými informacemi** — doba integrace (ms), FPS, vzorky, nahrávání (červené `REC` nebo `Off`) a stav (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Pouze DAQ-E: řádky síť, firmware a PTP

* **Název hostitele / IP** — aktuální adresa jednotky.
* **Firmware** — aktuální verze firmwaru, plus akční buňka:<version\>

tlačítko</version\>

**Aktualizovat na \<version\>** se zobrazí, pokud tato verze Chloros obsahuje novější obraz firmwaru pro DAQ-E. Aktualizace se přes síť nainstaluje za přibližně 30 sekund; senzor se automaticky restartuje a znovu připojí, přičemž přerušený přenos ponechá aktuální firmware beze změny. Průběh se zobrazuje v reálném čase (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`) a při aktuálním stavu se v buňce zobrazuje hodnota `Up to date`.
* **Synchronizace PTP** — aktuální stav PTP (v případě výpadku se vrátí na `unknown`). Firmware DAQ-E verze 1.2.0 a vyšší se účastní standardu IEEE 1588 PTPv2 pouze jako podřízený časovač; backend hostitele Chloros je PTP grandmaster a každá kamera DAQ-E a LATTICE v síti LAN se k němu připojuje jako slave v doméně 0, přičemž časová razítka se udržují s odchylkou přibližně 1 ms.
* **Síť**— adresa, na které jednotka běží, a způsob jejího získání (DHCP, link-local nebo statická), a uložené nastavení, které se použije po restartu.**Změnit…**otevře formulář:**Automaticky (DHCP)**nebo**Statická IP**s IP adresou, maskou podsítě a volitelnou bránou.**Použít a restartovat** nastavení uloží, restartuje senzor a znovu jej připojí. Statická adresa mimo síť tohoto počítače je odmítnuta, protože Chloros by po restartu nemohl senzor kontaktovat; následně se zobrazí pole, které to přesto povolí, a to pro počítač, který bude přadresován nebo směrován do této sítě. **Streamovat při zapnutí**určuje, zda senzor začne streamovat ihned po zapnutí, ještě před připojením jakéhokoli softwaru.**Restartovat senzor** senzor restartuje a znovu jej připojí s aktuálním nastavením. Restart je zamítnut, pokud senzor právě zaznamenává data. Novinka ve verzi 1.3.0.

U skupiny „Ambient+Object“ zobrazuje modální okno s ozubeným kolečkem pouze zdrojové senzory skupiny, název zařízení a barvu čáry grafu.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->



### Výběr krytky

Rozbalovací nabídka **Krytka** sděluje aplikaci Chloros, která fyzická krytka je nasazena na difuzor senzoru, a na každé spektrum aplikuje korekční profil této krytky naměřený při výrobě. Možnosti výběru závisí na modelu:

| Model | Možnosti krytky |
| --- | --- |
| DAQ-U | Žádný (holý senzor), FOV 15°, FOV 30°, FOV 45°, FOV 60°, FOV 90°, Sunshine (kosinový korektor) |
| DAQ-M | Žádný (holý senzor), Sunshine (kosinový korektor) |
| DAQ-E | Žádný (holý senzor), zorné pole 15°, zorné pole 45°, zorné pole 90°, Sunshine (kosinový korektor) |

**Výchozí nastavení pro každý model je „Sunshine“ (kosinová korekce)** — společnost MAPIR dodává každý model DAQ s nainstalovaným krytem „Sunshine“, což je standardní konfigurace pro venkovní použití: 180° polokulový záběr s kosinovou chybou ≤ ±4 % do 60° a ≤ ±4,5 % do 70° (nedoporučuje se při elevaci Slunce pod ~15°), konstrukčně zeslabený (~12×). Váš výběr zůstane v projektu zachován.

{% hint style="warning" %}
**Výběr krytky musí odpovídat fyzicky nasazené krytce.**Ani senzor, ani software nedokážou rozpoznat, která krytka je nasazena. Tento výběr ovlivňuje jak korekci v reálném čase, tak i záznam zapsaný do každého souboru `.daq` — vzhledem k ~12× útlumu krytky Sunshine nedeklarovaná změna krytky způsobí nesprávnou korekci spektra přibližně o tento faktor. (Odstranění a opětovné nasazení stejné krytky vede k odchylce přibližně 1,5 %.) Volbu**None (holý senzor)** použijte pouze v případě, že je krytka fyzicky odstraněna; u zařízení DAQ-E se i při volbě „None“ stále aplikuje tovární geometrický profil pro jeho zapuštěný skleněný difuzor – nejedná se o nečinnost – a holé zařízení DAQ-E představuje laboratorní konfiguraci, nikoli podporovanou konfiguraci pro použití v terénu.
{% endhint %}

{% hint style="info" %}
Aktualizace oproti dřívějšímu manuálu: přepínač „Sunshine Diffuser Installed“ na straně prohlížeče z verze 1.1.0 byl odstraněn. Manipulace s krytkami se nyní řídí profilem krytky pro každý senzor, který se aplikuje na straně serveru.
{% endhint %}

***

## Oblast grafu

V horní liště, která zůstává vždy viditelná, se nachází **přepínač mezi zobrazením seznamu a mřížky**a posuvník**zoomu grafu** (velikost dlaždic 200–2000 px). Zobrazení se automaticky přepne na mřížku, pokud existuje více než jedna skupina grafů, a zpět na seznam, pokud je jich jedna nebo méně. Režim zobrazení a velikost grafu se ukládají pro každý projekt zvlášť.**Spektrální graf** pro každý senzor zobrazuje:

* **Osa X** — vlnová délka (nm). Mřížka snímače je 340–1010 nm s roztečí 5 nm (135 bodů), pro zobrazení interpolovaná na 1 nm.
* **Osa Y**— u kalibrovaného snímače (zelená ikona**Kalibrováno**),**Výkon (W/m²/nm)**— radiometricky kalibrovaná spektrální ozářenost ve všech třech jednotkách — s automaticky zvolenou předponou SI (m/µ/n) podle hodnoty na vrcholu. Senzor, jehož kalibrační balíček není načten, vykresluje své surové hodnoty jako**Intenzita (počty)**, bez škálování. Pokud graf překrývá oba typy, výkon je na levé ose a počty na pravé ose.
* Duhová spektrální výplň pod jednou křivkou; více senzorů na jednom grafu se překrývají jako barevné čáry s tlumenými výplněmi.
* **Přejetí myší**— svislý kurzor s vlnovou délkou a hodnotou pro jednotlivé senzory (W/m²/nm u kalibrovaného senzoru, jinak celkový počet impulzů);**přetažením** lze měnit měřítko (při přiblížení se zobrazí tlačítko pro oddálení).
* Tlačítko **+** (pouze v mřížkovém zobrazení) pro přidání senzoru do tohoto grafu nebo vytvoření skupiny (viz níže).
* Název zařízení vycentrovaný nahoře a otočné kolečko, dokud nedorazí první snímek.

**Nasycení** není na samotném grafu označeno: nasycený senzor zobrazuje červený stavový text `SATURATED` a červený řádek `Saturated: Yes` v tabulce živých dat. Pro odstranění tohoto stavu zkraťte integrační čas nebo znovu zapněte AE.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->



***

## Tabulka živých dat (zobrazení seznamu)

Pod grafem v zobrazení seznamu, aktualizováno každých 500 ms:

* **Všechny modely**: Vzorek barvy světla (sRGB z CIE XYZ), Nasycený (Ano/Ne), CIE 1931 X/Y/Z, Chromatickost x/y, CIE u′/v′, CCT (K), CRI (Ra), dominantní vlnová délka (nm), špičková vlnová délka (nm), čistota excitace, Duv, CIE L\*/a\*/b\* a Munsell H/V/C.
* **Pouze kalibrované senzory**(libovolný z modelů DAQ-U / DAQ-M / DAQ-E po načtení balíčku tovární kalibrace — poznáte to podle zeleného označení**Kalibrováno** v řádku senzoru): Celkový výkon (W/m²), fotopický lux (lx), skotopický lux (lx), poměr S/P, PPFD a PPFD Red /Green /Blue (µmol/m²/s) a opické ozáření — S-kónus, melanopické, rhodopické, M-kónus, L-kónus (vše v jednotkách W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Skupiny odrazivosti (Okolní světlo + Objekt)

Dva propojené senzory lze zkombinovat do živého zobrazení odrazivosti — bez použití kamery:

1. V mřížkovém zobrazení klikněte na **+**na dlaždici grafu a vyberte**Kombinovat okolní světlo + objekt**.
2. Vyberte senzor **Zdroje okolního světla**a senzor**Skeneru objektu**(dva odlišné senzory) a poté klikněte na**Vytvořit**.

Chloros vypočítá R(λ) = objekt(λ) / okolí(λ) pro každou vlnovou délku ze dvou živých datových toků (0, pokud okolí ≤ 0). Název skupiny se řídí kalibrační třídou senzorů:

* Oba senzory jsou kalibrované (balíček načten) → **„Zdánlivá odrazivost“**.
* Jeden ze senzorů není kalibrován → **„Relativní odrazivost“**.

Skupina se zobrazí jako zelený řádek `REF` v postranním panelu a má vlastní graf (duhová výplň, hodnoty při najetí myší na 4 desetinná místa, přetažením lze měnit měřítko).

Nabídka **+**také nabízí možnost**Přidat nový senzor** se třemi možnostmi umístění: *Sloučit nový senzor* (přidat do tohoto grafu), *Přesunout sem existující senzor* nebo *Zobrazit nový senzor* (vlastní graf).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->



### Tabulka vegetačních indexů

V zobrazení seznamu se pod grafem skupiny odrazivosti nachází tabulka vegetačních indexů, vypočítaná z aktuální odrazivosti ve středních frekvencích pásem **modrá 450 / zelená 550 / červená 670 / NIR 800 nm** (hodnoty s přesností na 4 desetinná místa, `---`, pokud nelze vypočítat; najedete-li kurzorem na název indexu, zobrazí se jeho plný název):

* **Vždy zobrazeno** (nezávislé na měřítku, libovolná kombinace senzorů): NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR.
* **Pouze pokud jsou oba senzory kalibrovány na výkon** (načteny oba balíčky): EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI.

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

***

## Záznam souborů `.daq`

* Záznam vyžaduje **otevřený projekt** — v opačném případě jsou deaktivována jak tlačítko „Záznam vše“ (v postranním panelu), tak tlačítka „Záznam“ pro jednotlivé senzory.
* Soubory se ukládají do složky **`<project folder>/light_sensor/`**; názvy souborů obsahují ID senzoru a časové razítko a název zařízení se ukládá spolu se záznamem.
* Po zastavení záznamu (tlačítko „Zastavit“, „Zastavit vše“ nebo odpojení během záznamu) se dokončený soubor `.daq` **automaticky přidá do otevřeného projektu** — objeví se v seznamu souborů projektu bez nutnosti ručního přidání a je připraven sloužit jako data o dopadajícím záření pro [zpracování odrazivosti](README.md).
* Během záznamu se v živých řádcích modálního okna nastavení zobrazuje červený indikátor `REC`.

Pro získání kvantitativních hodnot ozáření je třeba průměrovat data z alespoň 15 sekund — jedná se o charakteristiku přístroje, nikoli o vadu.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->



***

## Rozložení více senzorů a uchování nastavení projektu

* Kombinujte několik senzorů na jednom grafu (sdílené osy), udržujte samostatné grafy (automatické rozložení mřížky), přesouvejte senzory mezi grafy, přetahujte a měňte pořadí řádků/dlaždic a skryjte jednotlivé senzory pomocí přepínače ve tvaru oka.
* Chloros si pro každý projekt uchovává: názvy zařízení, barvy grafů, velikost grafu, režim zobrazení a nastavení každého senzoru (integrační čas, průměrování snímků, stav AE, výběr krytky).
* **Při opětovném otevření projektu se senzory automaticky znovu připojí** podle adresy – COM port pro DAQ-U, zařízení BLE pro DAQ-M, název hostitele mDNS pro DAQ-E (vyřešeno i v případě změny IP adresy zařízení) — a znovu použije uložený profil krytu, průměrování snímků, stav AE a ručně nastavenou integrační dobu každého senzoru.

***

## Spárování kamery (DLS)

Není třeba nic spárovat. Na rozdíl od pracovních postupů DLS u dronů, které předem propojují světelný senzor s kamerou, Chloros přiřazuje data DAQ k snímkům až v následném zpracování: při importu/zpracování jsou hodnoty `.daq` interpolovány k časovému razítku expozice každého snímku. Zaznamenávejte pomocí libovolného připojeného senzoru (`.daq` se do projektu přidá automaticky) a zpracování odrazivosti najde správné hodnoty podle času — viz [Světelné senzory DAQ](README.md), kde se dozvíte, jak se používají data o dopadajícím světle. U **živých**snímků LATTICE vyberte snímač v rozevíracím seznamu**Light Sensor** na kartě Cameras; každý záznam pak uloží naměřenou hodnotu, která byla použita, a která odpovídá pouze kamerám daného záznamu (viz [Pracovní postupy s odrazivostí](reflectance.md)).</version\>
