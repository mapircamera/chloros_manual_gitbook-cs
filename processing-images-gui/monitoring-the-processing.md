# Sledování zpracování

Jakmile se zpracování spustí, Chloros nabízí několik způsobů, jak sledovat průběh, kontrolovat případné problémy a zjistit, co se s vaším datovým souborem děje. Na této stránce se dozvíte, jak sledovat průběh zpracování a jak interpretovat informace, které Chloros poskytuje.

## Přehled ukazatele průběhu

Ukazatel průběhu v horní záhlaví zobrazuje stav zpracování v reálném čase a procento dokončení. Průběh je přenášen živě z backendu prostřednictvím Server-Sent Events (SSE), takže ukazatel odráží to, co procesní řetězec právě provádí.

### Ukazatel průběhu v bezplatném režimu

Pro uživatele bez licence Chloros+:

**Zobrazení průběhu ve 2 fázích:**

1.**Detekce cílů** – Vyhledávání kalibračních cílů v obrazech
2. **Zpracování** – Aplikace korekcí a export**Indikátor průběhu zobrazuje:**

* Celkové procento dokončení (0–100 %)
* Název aktuální fáze
* Jednoduchá vizualizace v podobě vodorovného pruhu

### Indikátor průběhu v Chloros+

Pro uživatele s licencí Chloros+:

**Zobrazení průběhu ve 4 fázích:**

1.**Detekce** – Vyhledávání kalibračních cílů
2. **Analýza** – Prozkoumání snímků a příprava zpracovatelského řetězce
3. **Kalibrace** – Aplikace korekcí vinětace a odrazivosti
4. **Export** – Uložení zpracovaných souborů**Interaktivní funkce:*** **Přejděte kurzorem myši** nad ukazatel průběhu a zobrazte rozbalený panel se 4 fázemi
* **Kliknutím** na ukazatel průběhu rozbalený panel zafixujete
* **Klikněte znovu** pro uvolnění a automatické skrytí při odjetí myši
* Každá fáze zobrazuje individuální průběh (0–100 %)

{% hint style="info" %}
**Parita CLI**: během běhu `chloros-cli process` hlásí stejná čtyři vlákna stavy „Detecting“, „Analyzing“, „Processing“ a „Exporting“, zatímco `chloros-cli export-status` zobrazuje v reálném čase průběh exportu vlákna 4 z jiného terminálu. Viz [Referenční příručka k CLI](../reference/cli-reference.md).
{% endhint %}

***

## Vysvětlení jednotlivých fází zpracování

{% hint style="info" %}
**Architektura potrubí**: Tyto 4 fáze v grafickém uživatelském rozhraní odpovídají [4-vláknovému zpracovatelskému potrubí](../processing-architecture/processing-pipeline.md). Na systémech s akcelerací GPU využívá vlákno 3 (Kalibrace) výhody [dynamické adaptace výpočtů](../processing-architecture/dynamic-compute-adaptation.md), která optimalizuje zpracování pro váš konkrétní hardware.
{% endhint %}

### Fáze 1: Detekce (detekce cílů)

**Co se děje:**

* Chloros prohledává obrázky, které jste označili zaškrtnutím políčka „Cíl“ (všechny obrázky, pokud není zaškrtnuto žádné)
* Algoritmy počítačového vidění identifikují kalibrační panely
* Z každého panelu jsou extrahovány hodnoty odrazivosti
* Zaznamenávají se časová razítka cílů pro správné naplánování kalibrace

**Trvání:**

* S označenými cíli: 10–60 sekund
* Bez označených cílů: 5–30+ minut (prohledává všechny snímky)

**Ukazatel průběhu:**

* Detekce: 0 % → 100 %
* Počet naskenovaných snímků (počítá se pouze počet snímků, které jsou skutečně skenovány)
* Počet nalezených cílů

**Na co si dát pozor:**

* Pokud jsou cíle správně označeny, proces by měl proběhnout rychle
* Pokud to trvá příliš dlouho, cíle nemusí být označeny
* Zkontrolujte v ladicím protokolu zprávy „Cíl nalezen“

### Fáze 2: Analýza

**Co se děje:**

* Čtení metadat EXIF snímků (časová razítka, nastavení expozice)
* Stanovení kalibrační strategie na základě časových razítek cílů a dostupných dat DAQ o slunečním záření směřujícím k Zemi
* Uspořádání fronty pro zpracování snímků
* Příprava procesů pro paralelní zpracování (pouze Chloros+)

**Trvání:** 5–30 sekund**Ukazatel průběhu:**

* Analýza: 0 % → 100 %
* Rychlá fáze, obvykle se dokončí rychle

**Na co si dát pozor:**

* Průběh by měl být plynulý bez přestávek
* Varování ohledně chybějících metadat se zobrazí v ladicím protokolu

### Fáze 3: Kalibrace

**Co se děje:*** **Debayering**: Převod RAW vzoru Bayer na 3 kanály (u monochromatických modulů LATTICE se tento krok přeskočí, s poznámkou)
* **Korekce vinětace**: Odstranění ztmavnutí okrajů objektivu
* **Kalibrace odrazivosti**: Normalizace podle cílových hodnot a/nebo sestupného toku dat (DAQ downwelling)
* **Výpočet indexů**: Výpočet multispektrálních indexů
* Zpracování každého snímku v rámci celého zpracovatelského řetězce

**Doba trvání:** Většina celkové doby zpracování (60–80 %)**Ukazatel průběhu:**

* Kalibrace: 0 % → 100 %
* Právě se zpracovává snímek
* Zpracované snímky / Celkový počet snímků

**Chování při zpracování:*** **Režim Free**: Zpracovává jeden snímek po druhém postupně
* **Režim Chloros+**: Spouští pracovní skupinu přizpůsobenou hardwaru — 1–4 souběžné pracovní procesy na systémech s GPU (podle VRAM), jeden pracovní proces na fyzické jádro (minus jeden) na systémech pouze s CPU. Viz [Dynamická adaptace výpočtů](../processing-architecture/dynamic-compute-adaptation.md)
* **Akcelerace GPU**: Výrazně urychluje tuto fázi**Na co si dát pozor:**

* Plynulý postup podle počtu obrázků
* Zkontrolujte ladicí protokol, zda obsahuje zprávy o dokončení jednotlivých obrázků
* Varování týkající se kvality snímků nebo problémů s kalibrací

### Fáze 4: Export

**Co se děje:**

* Zápis zpracovaných snímků na disk ve vybraném formátu, jakmile jsou dokončeny
* **LATTICE**: každý snímek se rozdělí do všech povolených produktů (debayering / náhled / radiance / odrazivost)
* Export multispektrálních indexových snímků s barvami LUT
* Vytvoření výstupního stromu `<project>/<camera>/<format>/<Product>_Images/` — exportované soubory si zachovávají název zdrojového souboru; složka identifikuje produkt

**Trvání:** 10–20 % celkové doby zpracování**Ukazatel průběhu:**

* Export: 0 % → 100 %
* Probíhá zápis souborů
* Formát exportu a cílové umístění

**Na co si dát pozor:**

* Varování o nedostatku místa na disku
* Chyby při zápisu souborů
* Dokončení všech nakonfigurovaných výstupů

***

## Karta Debug Log (Log ladění)

Log ladění poskytuje podrobné informace o průběhu zpracování a případných problémech. Do konzole logu se také přehrávají zprávy o spuštění backendu, takže log poskytuje ucelený přehled i v případě, že jej otevřete s určitým zpožděním.

### Přístup k ladicímu protokolu

1. Klikněte na ikonu **Ladicí protokol** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

v levém postranním panelu
2. Otevře se panel protokolu zobrazující zprávy o zpracování v reálném čase
3. Panel se automaticky posouvá, aby zobrazoval nejnovější zprávy

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written), with the 1.3.0 bug icon (Report a bug) visible at the top of the tab -->

### Vysvětlení zpráv v protokolu Řádky protokolu

Chloros u mají na začátku značky v závorkách, které označují subsystém — například `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Nejdůležitější je znát **souhrn běhu**, který se zobrazuje na konci každého běhu (včetně zastavených běhů):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Další řádky s tipy `[RUN-SUMMARY]` následují vždy, když je třeba něco vysvětlit — například běh, který nic nevytvořil, nebo kamera, jejíž požadovaný výstup byl vynechán jako nepoužitelný. Řádky `[EXPORT-CHECK]` vysvětlují vynechání u jednotlivých kamer (např. proč kamera RGB nezískala žádný výstup radiance).

Obecné úrovně závažnosti zpráv (níže uvedené příklady jsou ilustrativní, nikoli doslovné):

#### Informační zprávy (bílá/šedá)

Běžné aktualizace zpracování: zahájení zpracování, detekce cílů (s počtem panelů), průběh kalibrace jednotlivých snímků, export souborů, dokončení zpracování.

#### Varovné zprávy (žluté)

Nekritické problémy, které nezastaví zpracování — např. chybějící GPS data v snímku, velká časová mezera mezi snímky cílů nebo nízký kontrast v kalibračním panelu.

**Akce:** Zkontrolujte varování po dokončení zpracování, ale nepřerušujte ho

#### Chybové zprávy (červené – Red)

Kritické problémy, které mohou způsobit selhání zpracování — např. zaplněný disk, poškozený obrazový soubor nebo nezjištění žádných cílů při požadavku na kalibraci odrazivosti.

**Akce:** Zastavte zpracování, vyřešte chybu a restartujte

### Běžné situace v protokolu

| Situace                             | Význam                                       | Požadovaná akce                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Cíl detekován v \[název souboru]        | Kalibrační cíl úspěšně nalezen         | Žádná – normální                                         |
| Čáry postupu pro jednotlivé snímky              | Aktualizace aktuálního postupu                       | Žádná – normální                                         |
| Nebyly nalezeny žádné cíle                      | Nebyly detekovány žádné kalibrační cíle               | Označte obrázky cílů nebo deaktivujte kalibraci odrazivosti |
| Nedostatek místa na disku               | Nedostatek úložného prostoru pro výstup                 | Uvolněte místo na disku                                    |
| Přeskočení poškozeného souboru               | Obrazový soubor je poškozen                         | Znovu zkopírujte soubor z SD karty                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | Snímání bez surového snímku nelze zpracovat | Proveďte nové snímání se surovým snímkem, nebo použijte CLI `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | Spuštění nevytvořilo žádné obrazové výstupy — hlášeno jako selhání s nápovědou | Přečtěte si řádky s nápovědou; zkontrolujte, co bylo přeskočeno a proč |
| `[PROCESS] refused: The project folder is too deep for this computer…` | Dlouhé cesty v systému Windows jsou vypnuté a složka projektu je příliš hluboko pro limit 260 znaků v cestě; nic nebylo zpracováno | Použijte kratší složku projektu (zpráva uvádí nejdelší možnou cestu, která se vejde), nebo zapněte dlouhé cesty ve Windows u a restartujte Chloros — viz [Spuštění zpracování](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer) |
| `[unmix-clamp-mask] …` | Nelze zapsat masku clamp produktu LATTICE; samotný produkt je v pořádku a hlásí `Chloros:UnmixClampMask=unavailable` | Zkontrolujte místo na disku a oprávnění ke složce, poté proveďte zpracování znovu, pokud masku potřebujete |

### Kopírování dat protokolu

Chcete-li zkopírovat protokol pro účely řešení problémů nebo technické podpory:

1. Otevřete panel Debug Log (Log ladění)
2. Klikněte na tlačítko **Copy**(Kopírovat) (zkopíruje protokol do schránky) nebo na**Download** (Stáhnout) a uložte jej jako textový soubor
3. Vložte do textového souboru nebo e-mailu
4. V případě potřeby odešlete na podporu MAPIR



### Nahlášení chyby z karty protokolu

Novinka ve verzi 1.3.0: ikona chyby (**Nahlásit chybu**) v horní části karty Debug Log odešle hlášení přímo do MAPIR.

1. Klikněte na ikonu chyby v horní části karty Debug Log
2. Popište problém: co jste dělali, co jste očekávali a co se místo toho stalo (povinné)
3. Volitelně zaškrtněte **Přidat několik obrázků (až 10)**a vyberte obrázky z otevřeného projektu, nebo použijte**Procházet...** pro jakýkoli jiný obrazový soubor
4. Klikněte na **Odeslat**

Zpráva obsahuje váš popis, všechny přiložené obrázky, protokoly Chloros a podrobnosti o systému (verze Chloros, operační systém, GPU a nastavení projektu). Abyste mohli zprávu odeslat, musíte být přihlášeni do Chloros (karta „User“). Během odesílání můžete okno zavřít; po dokončení se do protokolu zapíše odkaz, například `Bug report sent. Reference: BR-…` – uveďte jej, když budete kontaktovat MAPIR. Pokud nelze zprávu odeslat (například kvůli chybějícímu připojení k internetu), Chloros ji uloží do vašeho počítače a zobrazí cestu k ní, abyste ji mohli odeslat e-mailem na adresu support@mapir.camera.

***

## Sledování systémových zdrojů

### Využití CPU

**Režim Free:**

* 1 jádro procesoru na ~100 %
* Ostatní jádra jsou nečinná nebo k dispozici
* Systém zůstává odezvu

**Paralelní režim Chloros+:**

* Vysoké využití více jader — počet závisí na strategii zvolené v [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md)
* Systém může působit méně odezvě

**Jak sledovat:**

* Správce úloh systému Windows (Ctrl+Shift+Esc)
* Karta Výkon → sekce CPU
* Hledejte procesy „Chloros “ nebo „chloros-backend“

### Využití paměti (RAM)

**Typické využití:**

* Malé projekty (< 100 obrázků): 2–4 GB
* Střední projekty (100–500 obrázků): 4–8 GB
* Velké projekty (500 a více obrázků): 8–16 GB
* Paralelní režim Chloros+ spotřebovává více paměti RAM

**Pokud je paměť nedostatečná:**

* Zpracovávejte menší dávky
* Ukončete ostatní aplikace
* Pokud pravidelně zpracováváte velké datové sady, navýšte kapacitu RAM

### Využití GPU (Chloros+ s CUDA)

Je-li povoleno akcelerace GPU:

* GPU NVIDIA vykazuje vysoké využití (60–90 %)
* Využití VRAM se zvyšuje (vyžaduje 4 GB a více VRAM; 7 GB a více pro souběžné odstraňování bayeringu s podporou textur)
* Fáze kalibrace je výrazně rychlejší

**Sledujte:**

* Ikona NVIDIA v systémové liště
* Správce úloh → Výkon → GPU
* GPU-Z nebo podobný monitorovací nástroj

### Diskové I/O

**Co lze očekávat:**

* Vysoká čtecí zátěž disku během fáze analýzy
* Vysoká zapisovací zátěž disku během fáze exportu
* SSD je výrazně rychlejší než HDD

**Tip pro zvýšení výkonu:**

* Pokud je to možné, použijte pro projektovou složku SSD
* U velkých datových sad se vyhýbejte síťovým diskům
* Ujistěte se, že disk není téměř plný (ovlivňuje to rychlost zápisu)

***

## Detekce problémů během zpracování

### Varovné signály

**Zastavení postupu (žádná změna po dobu 5 a více minut):**

* Zkontrolujte, zda v ladicím protokolu nejsou chyby
* Ověřte, zda je k dispozici dostatek místa na disku
* V Správci úloh zkontrolujte, zda běží Chloros **Často se zobrazují chybové zprávy:**

* Zastavte zpracování a zkontrolujte chyby
* Časté příčiny: nedostatek místa na disku, poškozené soubory, problémy s pamětí
* Viz část Řešení problémů níže

**Systém přestane reagovat:**

* Paralelní režim Chloros u+ spotřebovává příliš mnoho zdrojů
* Zvažte snížení počtu souběžných úloh nebo upgrade hardwaru
* Režim Free je méně náročný na zdroje

### Kdy zastavit zpracování

Zastavte zpracování, pokud se objeví:

* ❌ Chyby „Disk plný“ nebo „Nelze zapsat soubor“
* ❌ Opakované chyby poškození obrazových souborů
* ❌ Systém zcela zamrzl (nereaguje)
* ❌ Zjistili jste, že byla nastavena nesprávná konfigurace
* ❌ Byly importovány nesprávné obrazy

**Jak zastavit:**

1. Klikněte na**tlačítko Zastavit** (nahrazuje tlačítko Spustit) — stačí jednou
2. Na liště se zobrazí „Zastavuje se...“, dokud se nedokončí právě zpracovávaný obraz, poté se proces ukončí ve stavu zastavený
3. Již exportované produkty zůstanou na disku; protokol vygeneruje přesný záznam `[RUN-SUMMARY]` o tom, co bylo dokončeno
4. Opravte problémy a restartujte — běh začne od začátku

***

## Řešení problémů během zpracování

### Zpracování je velmi pomalé

**Možné příčiny:**

* Neoznačené cílové obrázky (skenování všech obrázků)
* Ukládání na HDD místo SSD
* Nedostatečné systémové zdroje
* Nastaveno mnoho indexů
* Přístup k síťovému disku

**Řešení:**

1. Pokud se proces právě spustil a je ve fázi detekce: Zastavte ho, označte cíle a restartujte
2. Pro budoucnost: Používejte SSD, snižte počet indexů, vylepšete hardware
3. Zvažte použití příkazového řádku (CLI) pro dávkové zpracování velkých datových sad

### Varování „Místo na disku“

**Řešení:**

1. Okamžitě uvolněte místo na disku
2. Přesuňte projekt na disk s větším prostorem
3. Snižte počet indexů k exportu
4. Deaktivujte exportní produkty LATTICE, které nepotřebujete (Nastavení projektu → Zpracování)
5. Používejte formát JPG místo TIFF u (menší soubory)

### Časté zprávy o „poškozených souborech“

**Řešení:**

1. Znovu zkopírujte snímky z SD karty, abyste zajistili jejich integritu
2. Otestujte SD kartu na chyby
3. Odstraňte poškozené soubory z projektu
4. Pokračujte ve zpracování zbývajících snímků

### Přehřívání systému / omezení výkonu

**Řešení:**

1. Zajistěte dostatečné větrání
2. Odstraňte prach z ventilačních otvorů počítače
3. Snižte zátěž při zpracování (použijte režim „Free“ namísto „Chloros+“)
4. Zpracovávejte v chladnějších částech dne

***

## Oznámení o dokončení zpracování

Po dokončení zpracování:

* Ukazatel průběhu dosáhne 100 %
* V ladicím protokolu se objeví řádky `[RUN-SUMMARY]` s konečnými počty
* Tlačítko „Start“ se opět aktivuje
* Všechny výstupní soubory se nacházejí ve výstupní struktuře projektu pro jednotlivé kamery: `<project>/<camera>/<format>/<Product>_Images/`

***

## Další kroky

Po dokončení zpracování:

1. **Zkontrolujte výsledky** – viz [Dokončení zpracování](finishing-the-processing.md)
2. **Zkontrolujte výstupní složku** – ověřte, zda byly všechny soubory správně exportovány
3. **Prohlédněte si ladicí protokol** – Zkontrolujte, zda neobsahuje varování nebo chyby
4. **Prohlédněte si náhledy zpracovaných snímků** – Použijte prohlížeč obrázků nebo externí software

Informace o prohlížení a používání zpracovaných výsledků najdete v části [Dokončení zpracování](finishing-the-processing.md).
