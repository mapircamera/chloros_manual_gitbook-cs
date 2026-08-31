# Sledování zpracování

Jakmile se zpracování spustí, Chloros nabízí několik způsobů, jak sledovat průběh, kontrolovat případné problémy a porozumět tomu, co se s vaším datovým souborem děje. Na této stránce se dozvíte, jak sledovat průběh zpracování a jak interpretovat informace poskytované Chloros.

## Přehled ukazatele průběhu

Ukazatel průběhu v horní záhlaví zobrazuje stav zpracování v reálném čase a procentuální dokončení. Průběh je přenášen živě z backendu prostřednictvím Server-Sent Events (SSE), takže ukazatel odráží to, co pipeline právě provádí.

### Ukazatel průběhu v bezplatném režimu

Pro uživatele bez licence Chloros+:

**Zobrazení průběhu ve 2 fázích:**

1.**Detekce cílů** – Vyhledávání kalibračních cílů v obrazech
2. **Zpracování** – Aplikace korekcí a export**Indikátor průběhu zobrazuje:**

* Celkové procento dokončení (0–100 %)
* Název aktuální fáze
* Jednoduchá vizualizace v podobě vodorovného pruhu

### Indikátor průběhu Chloros+

Pro uživatele s licencí Chloros+:

**Zobrazení průběhu ve 4 fázích:**

1.**Detekce** – Vyhledávání kalibračních cílů
2. **Analýza** – Prozkoumání snímků a příprava zpracovatelského řetězce
3. **Kalibrace** – Aplikace korekcí vinětace a odrazivosti
4. **Export** – Uložení zpracovaných souborů**Interaktivní funkce:*** **Přejděte kurzorem** nad ukazatel průběhu, aby se zobrazil rozbalený čtyřfázový panel
* **Kliknutím** na ukazatel průběhu panel zamrazíte/připnete
* **Klikněte znovu** pro uvolnění a automatické skrytí při odjetí myši
* Každá fáze zobrazuje individuální průběh (0–100 %)

{% hint style="info" %}
**CLI parita**: během spuštění `chloros-cli process` vykazují stejná čtyři vlákna stavy Detekce, Analýza, Zpracování, Export a `chloros-cli export-status` zobrazuje aktuální průběh exportu vlákna 4 z jiného terminálu. Viz [Referenční příručka k programu CLI](../reference/cli-reference.md).
{% endhint %}

***

## Vysvětlení jednotlivých fází zpracování

{% hint style="info" %}
**Architektura potrubí**: Tyto 4 fáze grafického uživatelského rozhraní odpovídají [4-vláknovému zpracovatelskému potrubí](../processing-architecture/processing-pipeline.md). Na systémech s akcelerací GPU využívá vlákno 3 (Kalibrace) výhody [dynamické adaptace výpočtů](../processing-architecture/dynamic-compute-adaptation.md), která optimalizuje zpracování pro váš konkrétní hardware.
{% endhint %}

### Fáze 1: Detekce (detekce cílů)

**Co se děje:**

* Chloros prohledává obrázky, které jste označili zaškrtnutím políčka „Cíl“ (všechny obrázky, pokud není zaškrtnuto žádné)
* Algoritmy počítačového vidění identifikují kalibrační panely
* Z každého panelu jsou extrahovány hodnoty odrazivosti
* Zaznamenávají se časová razítka cílů pro správné naplánování kalibrace

**Trvání:**

* S označenými cíli: 10–60 sekund
* Bez označených cílů: 5–30+ minut (prohledává všechny obrázky)

**Ukazatel průběhu:**

* Detekce: 0 % → 100 %
* Počet naskenovaných snímků (počítá pouze snímky, které jsou skutečně skenovány)
* Počet nalezených cílů

**Na co si dát pozor:**

* Pokud jsou cíle správně označeny, proces by měl proběhnout rychle
* Pokud to trvá příliš dlouho, cíle nemusí být označeny
* Zkontrolujte v ladicím protokolu zprávy „Cíl nalezen“

### Fáze 2: Analýza

**Co se děje:**

* Čtení metadat EXIF snímků (časová razítka, nastavení expozice)
* Stanovení kalibrační strategie na základě časových razítek cílů a dostupných dat DAQ o dopadu slunečního záření
* Organizace fronty pro zpracování snímků
* Příprava procesů pro paralelní zpracování (pouze Chloros+)

**Trvání:** 5–30 sekund**Ukazatel průběhu:**

* Analýza: 0 % → 100 %
* Rychlá fáze, obvykle se dokončí rychle

**Na co si dát pozor:**

* Průběh by měl být plynulý bez přerušení
* Varování o chybějících metadatech se zobrazí v ladicím protokolu

### Fáze 3: Kalibrace

**Co se děje:*** **Debayering**: Převod RAW vzoru Bayer na 3 kanály (u monochromatických modulů LATTICE se tato fáze přeskočí, s poznámkou)
* **Korekce vinětace**: Odstranění ztmavnutí okrajů objektivu
* **Kalibrace odrazivosti**: Normalizace podle cílových hodnot a/nebo dopadajícího záření z DAQ
* **Výpočet indexů**: Výpočet multispektrálních indexů
* Zpracování každého snímku v rámci celého zpracovatelského řetězce

**Trvání:** Většina celkové doby zpracování (60–80 %)**Ukazatel průběhu:**

* Kalibrace: 0 % → 100 %
* Právě se zpracovává aktuální snímek
* Dokončené snímky / Celkový počet snímků

**Chování při zpracování:*** **Režim Free**: Zpracovává jeden snímek po druhém postupně
* **Režim Chloros+**: Spouští pracovní skupinu přizpůsobenou hardwaru — 1–4 souběžné pracovní procesy na systémech s GPU (podle VRAM), jeden pracovní proces na fyzické jádro (minus jeden) na systémech pouze s CPU. Viz [Dynamická adaptace výpočtů](../processing-architecture/dynamic-compute-adaptation.md)
* **GPU akcelerace**: Výrazně zrychluje tuto fázi**Na co si dát pozor:**

* Plynulý postup podle počtu obrázků
* Zkontrolujte ladicí protokol ohledně zpráv o dokončení jednotlivých obrázků
* Varování týkající se kvality obrázků nebo problémů s kalibrací

### Fáze 4: Export

**Co se děje:**

* Zápis zpracovaných snímků na disk ve vybraném formátu, jakmile jsou hotové
* **LATTICE**: každý snímek se rozděluje do všech povolených produktů (debayering / náhled / radiance / odrazivost)
* Export multispektrálních indexových snímků s barvami LUT
* Vytvoření výstupního stromu `<project>/<camera>/<format>/<Product>_Images/` — exportované soubory si zachovávají název zdrojového souboru; složka identifikuje produkt

**Trvání:** 10–20 % celkové doby zpracování**Ukazatel průběhu:**

* Export: 0 % → 100 %
* Probíhá zápis souborů
* Formát exportu a cílové umístění

**Na co si dát pozor:**

* Varování ohledně místa na disku
* Chyby při zápisu souborů
* Dokončení všech nakonfigurovaných výstupů

***

## Záložka Debug Log (Log ladění)

Log ladění poskytuje podrobné informace o průběhu zpracování a všech zjištěných problémech. Do konzole logu se také přehrávají zprávy o spuštění backendu, takže log poskytuje ucelený přehled i v případě, že jej otevřete se zpožděním.

### Přístup k ladicímu protokolu

1. Klikněte na ikonu **Ladicí protokol**<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

v levém postranním panelu
2. Otevře se panel protokolu zobrazující zprávy o zpracování v reálném čase
3. Panel se automaticky posouvá, aby zobrazoval nejnovější zprávy

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written) -->

### Vysvětlení zpráv v protokolu

Řádky protokolu Chloros jsou označeny předponou v závorkách, která označuje název subsystému – například `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Nejdůležitější je znát **souhrn běhu**, který se zobrazuje na konci každého běhu (včetně zastavených běhů):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Další řádky s tipy `[RUN-SUMMARY]` následují vždy, když je třeba něco vysvětlit – například běh, který nic nevytvořil, nebo kamera, jejíž požadovaný produkt byl vynechán jako nepoužitelný. Řádky `[EXPORT-CHECK]` vysvětlují vynechání jednotlivých kamer (např. proč kamera RGB nezískala žádný produkt radiance).

Obecné úrovně závažnosti zpráv (níže uvedené příklady jsou ilustrativní, nejedná se o doslovné citace):

#### Informační zprávy (Bílá/Šedá)

Aktualizace o běžném zpracování: zahájení zpracování, detekce cílů (s počtem panelů), průběh kalibrace jednotlivých snímků, exportovaných souborů, dokončení zpracování.

#### Varovné zprávy (Žlutá)

Nekritické problémy, které nezastaví zpracování — např. chybějící GPS data v snímku, velká časová mezera mezi snímky cílů nebo nízký kontrast v kalibračním panelu.

**Akce:** Zkontrolujte varování po zpracování, ale nepřerušujte ho

#### Chybové zprávy (Red)

Kritické problémy, které mohou způsobit selhání zpracování — např. zaplněný disk, poškozený obrazový soubor nebo nezjištění žádných cílů při požadavku na kalibraci odrazivosti.

**Akce:** Zastavte zpracování, vyřešte chybu a restartujte

### Běžné situace v protokolu

| Situace                             | Význam                                       | Potřebný postup                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Cíl detekován v \[název souboru]        | Kalibrační cíl úspěšně nalezen         | Žádný – normální                                         |
| Čáry postupu pro jednotlivé snímky              | Aktuální informace o postupu                       | Žádné – normální                                         |
| Nebyly nalezeny žádné cíle                      | Nebyly detekovány žádné kalibrační cíle               | Označte obrázky cílů nebo deaktivujte kalibraci odrazivosti |
| Nedostatek místa na disku               | Nedostatek úložného prostoru pro výstup                 | Uvolněte místo na disku                                    |
| Přeskočení poškozeného souboru               | Obrazový soubor je poškozen                         | Znovu zkopírujte soubor z SD karty                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | Snímání bez snímku ve formátu RAW nelze zpracovat | Znovusnímku s raw nebo použijte CLI `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | Spuštění nevytvořilo žádné obrazové výstupy — hlášeno jako selhání s nápovědou | Přečtěte si řádky s nápovědou; zkontrolujte, co bylo přeskočeno a proč |

### Kopírování dat protokolu

Chcete-li zkopírovat protokol pro účely řešení problémů nebo technické podpory:

1. Otevřete panel ladicího protokolu
2. Klikněte na tlačítko **„Kopírovat protokol“** (nebo klikněte pravým tlačítkem myši → Vybrat vše)
3. Vložte do textového souboru nebo do e-mailu
4. V případě potřeby odešlete na podporu MAPIR

***

## Sledování systémových zdrojů

### Využití procesoru

**Režim Free:**

* 1 jádro procesoru na ~100 %
* Ostatní jádra jsou nečinná nebo k dispozici
* Systém zůstává odezvu

**Režim Chloros+ Parallel:**

* Více jader s vysokým využitím — počet závisí na strategii zvolené funkcí [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md)
* Systém může působit méně odezvějící

**Jak sledovat:**

* Správce úloh (Ctrl+Shift+Esc)
* Karta Výkon → sekce CPU
* Vyhledejte procesy „Chloros“ nebo „chloros-backend“

### Využití paměti (RAM)

**Typické využití:**

* Malé projekty (&lt; 100 obrázků): 2–4 GB
* Střední projekty (100–500 obrázků): 4–8 GB
* Velké projekty (500 a více obrázků): 8–16 GB
* Paralelní režim Chloros+ spotřebovává více RAM

**Pokud je málo paměti:**

* Zpracovávejte menší dávky
* Ukončete ostatní aplikace
* Pokud pravidelně zpracováváte velké datové sady, navýšte kapacitu paměti RAM

### Využití GPU (Chloros+ s CUDA)

Je-li povoleno akcelerace pomocí GPU:

* GPU NVIDIA vykazuje vysoké využití (60–90 %)
* Využití VRAM se zvyšuje (vyžaduje 4 GB+ VRAM; 7 GB+ pro souběžné odstraňování bayeringu s podporou textur)
* Fáze kalibrace je výrazně rychlejší

**Sledujte:**

* Ikona NVIDIA v systémové liště
* Správce úloh → Výkon → GPU
* GPU-Z nebo podobný monitorovací nástroj

### Diskové I/O

**Co lze očekávat:**

* Vysoká četnost čtení z disku během fáze analýzy
* Vysoká četnost zápisu na disk během fáze exportu
* SSD je výrazně rychlejší než HDD

**Tip pro zvýšení výkonu:**

* Pokud je to možné, použijte pro projektovou složku SSD
* U velkých datových sad se vyhýbejte síťovým diskům
* Ujistěte se, že disk není téměř zaplněný (ovlivňuje to rychlost zápisu)

***

## Detekce problémů během zpracování

### Varovné signály

**Zastavení průběhu (žádná změna po dobu 5 a více minut):**

* Zkontrolujte protokol ladění (Debug Log) kvůli chybám
* Ověřte dostupné místo na disku
* V Správci úloh zkontrolujte, zda běží proces Chloros

**Často se zobrazují chybové zprávy:**

* Zastavte zpracování a zkontrolujte chyby
* Časté příčiny: nedostatek místa na disku, poškozené soubory, problémy s pamětí
* Viz část Řešení problémů níže

**Systém přestává reagovat:**

* Paralelní režim Chloros+ spotřebovává příliš mnoho systémových prostředků
* Zvažte snížení počtu souběžných úloh nebo upgrade hardwaru
* Volný režim je méně náročný na systémové prostředky

### Kdy zastavit zpracování

Zastavte zpracování, pokud se objeví:

* ❌ Chyby „Disk je plný“ nebo „Nelze zapsat soubor“
* ❌ Opakované chyby poškození obrazových souborů
* ❌ Systém zcela zamrzl (nereaguje)
* ❌ Zjistili jste, že byla nastavena nesprávná konfigurace
* ❌ Byly importovány nesprávné obrazy

**Jak zastavit:**

1. Klikněte na**tlačítko Zastavit** (nahrazuje tlačítko Spustit) — stačí jednou
2. Na liště se zobrazí „Zastavuje se...“, dokud se nedokončí právě zpracovávaný obraz, poté se proces ukončí ve zastaveném stavu
3. Již exportované produkty zůstanou na disku; protokol vygeneruje přesný záznam `[RUN-SUMMARY]` o tom, co bylo dokončeno
4. Opravte problémy a restartujte — proces začne od začátku

***

## Řešení problémů během zpracování

### Zpracování je velmi pomalé

**Možné příčiny:**

* Neoznačené cílový obrázky (skenování všech obrázků)
* Použití pevného disku (HDD) namísto SSD
* Nedostatečné systémové zdroje
* Nastaveno příliš mnoho indexů
* Přístup k síťovému disku

**Řešení:**

1. Pokud byl proces právě spuštěn a nachází se ve fázi detekce: Zastavte jej, označte cíle a restartujte
2. Do budoucna: Použijte SSD, snižte počet indexů, vylepšete hardware
3. Zvažte použití CLI pro dávkové zpracování velkých datových sad

### Varování „Místo na disku“

**Řešení:**

1. Okamžitě uvolněte místo na disku
2. Přesuňte projekt na disk s větším prostorem
3. Snižte počet indexů k exportu
4. Deaktivujte exportní produkty LATTICE, které nepotřebujete (Nastavení projektu → Zpracování)
5. Použijte formát JPG namísto TIFF (menší soubory)

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
3. Snižte zátěž při zpracování (použijte režim Free namísto Chloros+)
4. Zpracovávejte v chladnějších částech dne

***

## Oznámení o dokončení zpracování

Po dokončení zpracování:

* Ukazatel průběhu dosáhne 100 %
* V ladicím protokolu se objeví řádky `[RUN-SUMMARY]` s konečnými počty
* Tlačítko Start se opět stane aktivním
* Všechny výstupní soubory se nacházejí ve výstupní struktuře projektu pro jednotlivé kamery: `<project>/<camera>/<format>/<Product>_Images/`

***

## Další kroky

Po dokončení zpracování:

1. **Zkontrolujte výsledky** – viz [Dokončení zpracování](finishing-the-processing.md)
2. **Zkontrolujte výstupní složku** – ověřte, zda byly všechny soubory správně exportovány
3. **Projděte ladicí protokol** – zkontrolujte, zda neobsahuje varování nebo chyby
4. **Zobrazte náhled zpracovaných snímků** – Použijte prohlížeč obrázků nebo externí software

Informace o kontrole a použití zpracovaných výsledků najdete v části [Dokončení zpracování](finishing-the-processing.md).
