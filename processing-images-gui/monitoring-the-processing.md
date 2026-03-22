# Sledování zpracování

Jakmile se zpracování spustí, nabízí Chloros několik způsobů, jak sledovat průběh, kontrolovat případné problémy a zjistit, co se s vaším datovým souborem děje. Na této stránce se dozvíte, jak sledovat průběh zpracování a jak interpretovat informace, které Chloros poskytuje.

## Přehled ukazatele průběhu

Ukazatel průběhu v horní záhlaví zobrazuje stav zpracování v reálném čase a procento dokončení.

### Ukazatel průběhu ve volném režimu

Pro uživatele bez licence Chloros+:

**Zobrazení průběhu ve 2 fázích:**

1.**Detekce cílů** – Vyhledávání kalibračních cílů v obrazech
2. **Zpracování** – Aplikace korekcí a export**Indikátor průběhu zobrazuje:**

* Celkové procento dokončení (0–100 %)
* Název aktuální fáze
* Jednoduchou vizualizaci v podobě vodorovného pruhu

### Indikátor průběhu Chloros+

Pro uživatele s licencí Chloros+:

**Zobrazení průběhu ve 4 fázích:**

1.**Detekce** – Vyhledávání kalibračních cílů
2. **Analýza** – Prohlížení snímků a příprava postupu
3. **Kalibrace** – Aplikace korekcí vinětace a odrazivosti
4. **Export** – Uložení zpracovaných souborů**Interaktivní funkce:*** **Přejděte kurzorem myši** nad ukazatel průběhu a zobrazte rozbalený 4stupňový panel
* **Klikněte** na ukazatel průběhu a rozbalený panel zamrazte/připněte
* **Klikněte znovu** pro odblokování a automatické skrytí při odchodu myši
* Každá fáze zobrazuje individuální průběh (0–100 %)

***

## Vysvětlení jednotlivých fází zpracování

{% hint style="info" %}
**Architektura potrubí**: Tyto 4 fáze grafického uživatelského rozhraní odpovídají [4vláknové zpracovatelské potrubí](../processing-architecture/processing-pipeline.md). Na systémech s akcelerací GPU využívá vlákno 3 (Kalibrace) výhody [Dynamické adaptace výpočtů](../processing-architecture/dynamic-compute-adaptation.md), která optimalizuje zpracování pro váš konkrétní hardware.
{% endhint %}

### Fáze 1: Detekce (detekce cílů)

**Co se děje:**

* Chloros prohledává obrázky označené zaškrtávacím políčkem Cíl
* Algoritmy počítačového vidění identifikují 4 kalibrační panely
* Hodnoty odrazivosti extrahované z každého panelu
* Zaznamenávají se časová razítka cílů pro správné naplánování kalibrace

**Trvání:**

* S označenými cíli: 10–60 sekund
* Bez označených cílů: 5–30+ minut (prohledává všechny obrázky)

**Indikátor průběhu:**

* Detekce: 0 % → 100 %
* Počet prohledaných obrázků
* Počet nalezených cílů

**Na co si dát pozor:**

* Pokud jsou cíle správně označeny, mělo by to proběhnout rychle
* Pokud to trvá příliš dlouho, cíle nemusí být označeny
* Zkontrolujte v ladicím protokolu zprávy „Cíl nalezen“

### Fáze 2: Analýza

**Co se děje:**

* Čtení metadat EXIF snímků (časové značky, nastavení expozice)
* Určení strategie kalibrace na základě časových razítek cílů
* Uspořádání fronty zpracování obrázků
* Příprava procesů pro paralelní zpracování (pouze Chloros+)

**Trvání:** 5–30 sekund**Indikátor průběhu:**

* Analýza: 0 % → 100 %
* Rychlá fáze, obvykle se dokončí rychle

**Na co si dát pozor:**

* Průběh by měl být plynulý bez pauz
* Varování o chybějících metadatech se zobrazí v ladicím protokolu

### Fáze 3: Kalibrace

**Co se děje:*** **Debayering**: Převod RAW Bayerova vzoru na 3 kanály
* **Korekce vinětace**: Odstranění ztmavnutí okrajů objektivu
* **Kalibrace odrazivosti**: Normalizace podle cílových hodnot
* **Výpočet indexů**: Výpočet multispektrálních indexů
* Zpracování každého snímku v rámci celého procesu

**Trvání:** Většina celkové doby zpracování (60–80 %)**Indikátor průběhu:**

* Kalibrace: 0 % → 100 %
* Právě se zpracovává aktuální snímek
* Dokončené snímky / Celkový počet snímků

**Chování při zpracování:*** **Režim Free**: Zpracovává jeden snímek po druhém postupně
* **Režim Chloros+**: Zpracovává až 16 snímků současně
* **Akcelerace GPU**: Výrazně zrychluje tuto fázi**Na co si dát pozor:**

* Plynulý postup podle počtu obrázků
* Zkontrolujte ladicí protokol pro zprávy o dokončení jednotlivých obrázků
* Varování ohledně kvality obrázků nebo problémů s kalibrací

### Fáze 4: Export

**Co se děje:**

* Zápis kalibrovaných snímků na disk ve vybraném formátu
* Export multispektrálních indexových snímků s barvami LUT
* Vytváření podsložek modelů kamer
* Zachování původních názvů souborů s příslušnými příponami

**Doba trvání:** 10–20 % celkové doby zpracování**Indikátor průběhu:**

* Export: 0 % → 100 %
* Probíhá zápis souborů
* Formát exportu a cíl

**Na co si dát pozor:**

* Varování o volném místě na disku
* Chyby při zápisu souborů
* Dokončení všech nakonfigurovaných výstupů

***

## Karta Debug Log (Log ladění)

Log ladění poskytuje podrobné informace o průběhu zpracování a všech zjištěných problémech.

### Přístup k logu ladění

1. Klikněte na ikonu **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> v levém postranním panelu
2. Otevře se panel protokolu zobrazující zprávy o zpracování v reálném čase
3. Automaticky se posouvá, aby zobrazoval nejnovější zprávy

### Vysvětlení zpráv protokolu

#### Informační zprávy (bílé/šedé)

Normální aktualizace zpracování:

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Varovné zprávy (žluté)

Nekritické problémy, které nezastaví zpracování:

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Akce:** Zkontrolujte varování po zpracování, ale nepřerušujte ho

#### Chybové zprávy (Red)

Kritické problémy, které mohou způsobit selhání zpracování:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Akce:** Zastavte zpracování, vyřešte chybu a restartujte

### Běžné zprávy protokolu

| Zpráva                          | Význam                                | Požadovaná akce                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| „Cíl detekován v [název souboru]“ | Kalibrační cíl nalezen úspěšně  | Žádná – normální                                         |
| „Zpracovává se obrázek X z Y“        | Aktualizace aktuálního průběhu                | Žádná – normální                                         |
| „Nebyly nalezeny žádné cíle“               | Nebyly detekovány žádné kalibrační cíle        | Označte cílové obrázky nebo deaktivujte kalibraci odrazivosti |
| „Nedostatek místa na disku“        | Nedostatek úložného prostoru pro výstup          | Uvolněte místo na disku                                    |
| „Přeskočení poškozeného souboru“        | Obrazový soubor je poškozen                  | Znovu zkopírujte soubor z SD karty                             |
| „PPK data použita“               | Použity GPS korekce ze souboru .daq | Žádné – normální                                         |

### Kopírování dat protokolu

Kopírování protokolu pro účely řešení problémů nebo podpory:

1. Otevřete panel Debug Log (Protokol ladění)
2. Klikněte na tlačítko **„Copy Log“** (Kopírovat protokol) (nebo klikněte pravým tlačítkem myši → Vybrat vše)
3. Vložte do textového souboru nebo e-mailu
4. V případě potřeby odešlete na podporu MAPIR

***

## Sledování systémových zdrojů

### Využití CPU

**Režim Free:**

* 1 jádro CPU na ~100 %
* Ostatní jádra jsou nečinná nebo volná
* Systém zůstává odezvu

**Režim Chloros+ Parallel:**

* Více jader na 80–100 % (až 16 jader)
* Vysoké celkové využití CPU
* Systém může působit méně odezvě

**Sledování:**

* Windows Správce úloh (Ctrl+Shift+Esc)
* Karta Výkon → sekce CPU
* Vyhledejte procesy „Chloros“ nebo „chloros-backend“

### Využití paměti (RAM)

**Typické využití:**

* Malé projekty (&lt; 100 obrázků): 2–4 GB
* Střední projekty (100–500 obrázků): 4–8 GB
* Velké projekty (500+ obrázků): 8–16 GB
* Paralelní režim Chloros+ využívá více RAM

**Pokud je málo paměti:**

* Zpracovávejte menší dávky
* Zavřete ostatní aplikace
* Pokud pravidelně zpracováváte velké datové sady, vylepšete RAM

### Využití GPU (Chloros+ s CUDA)

Když je povoleno zrychlení GPU:

* GPU NVIDIA vykazuje vysoké využití (60–90 %)
* Využití VRAM se zvyšuje (vyžaduje 4 GB+ VRAM)
* Fáze kalibrace je výrazně rychlejší

**Sledujte:**

* Ikona NVIDIA v systémové liště
* Správce úloh → Výkon → GPU
* GPU-Z nebo podobný monitorovací nástroj

### Diskové I/O

**Co můžete očekávat:**

* Vysoké čtení z disku během fáze analýzy
* Vysoký zápis na disk během fáze exportu
* SSD je výrazně rychlejší než HDD

**Tip pro zvýšení výkonu:**

* Pokud je to možné, použijte pro projektovou složku SSD
* Vyhněte se síťovým diskům pro velké datové sady
* Ujistěte se, že disk není téměř plný (ovlivňuje to rychlost zápisu)

***

## Detekce problémů během zpracování

### Varovné signály

**Zastavení průběhu (žádná změna po dobu 5 a více minut):**

* Zkontrolujte protokol ladění, zda neobsahuje chyby
* Ověřte dostupné místo na disku
* Zkontrolujte Správce úloh, zda běží proces Chloros

**Časté zobrazování chybových hlášení:**

* Zastavte zpracování a zkontrolujte chyby
* Časté příčiny: nedostatek místa na disku, poškozené soubory, problémy s pamětí
* Viz část Řešení problémů níže

**Systém nereaguje:**

* Paralelní režim Chloros+ využívá příliš mnoho zdrojů
* Zvažte snížení počtu souběžných úloh nebo upgrade hardwaru
* Volný režim je méně náročný na zdroje

### Kdy zastavit zpracování

Zastavte zpracování, pokud se objeví:

* ❌ Chyby „Disk plný“ nebo „Nelze zapsat soubor“
* ❌ Opakované chyby poškození obrazových souborů
* ❌ Systém zcela zamrzl (nereaguje)
* ❌ Zjistili jste, že byla nakonfigurována nesprávná nastavení
* ❌ Byly importovány nesprávné obrazy

**Jak zastavit:**

1. Klikněte na**tlačítko Zastavit/Zrušit** (nahrazuje tlačítko Start)
2. Zpracování se zastaví, průběh se ztratí
3. Opravte problémy a začněte znovu od začátku

***

## Řešení problémů během zpracování

### Zpracování je velmi pomalé

**Možné příčiny:**

* Neoznačené cílové obrázky (skenování všech obrázků)
* HDD místo SSD úložiště
* Nedostatečné systémové zdroje
* Nastaveno mnoho indexů
* Přístup k síťovému disku

**Řešení:**

1. Pokud jste právě začali a jste ve fázi detekce: Zrušte, označte cíle, restartujte
2. Pro budoucnost: Použijte SSD, omezte počet indexů, vylepšete hardware
3. Zvažte použití CLI pro dávkové zpracování velkých datových sad

### Varování „Místo na disku“

**Řešení:**

1. Okamžitě uvolněte místo na disku
2. Přesuňte projekt na disk s větším prostorem
3. Snižte počet indexů k exportu
4. Použijte formát JPG namísto TIFF (menší soubory)

### Časté zprávy o „poškozených souborech“

**Řešení:**

1. Znovu zkopírujte snímky z SD karty, abyste zajistili jejich integritu
2. Otestujte SD kartu na chyby
3. Odstraňte poškozené soubory z projektu
4. Pokračujte ve zpracování zbývajících snímků

### Přehřívání systému / omezení výkonu

**Řešení:**

1. Zajistěte dostatečné větrání
2. Odstraňte prach z větracích otvorů počítače
3. Snižte zátěž při zpracování (použijte režim Free namísto Chloros+)
4. Zpracovávejte v chladnějších částech dne

***

## Oznámení o dokončení zpracování

Po dokončení zpracování:

* Ukazatel průběhu dosáhne 100 %
* V ladicím protokolu se zobrazí zpráva **„Zpracování dokončeno“**
* Tlačítko Start se opět aktivuje
* Všechny výstupní soubory se nacházejí v podsložce modelu fotoaparátu

***

## Další kroky

Po dokončení zpracování:

1. **Zkontrolujte výsledky** – viz [Dokončení zpracování](finishing-the-processing.md)
2. **Zkontrolujte výstupní složku** – ověřte, zda byly všechny soubory správně exportovány
3. **Projděte si protokol ladění** – zkontrolujte, zda neobsahuje varování nebo chyby
4. **Prohlédněte si náhledy zpracovaných snímků** – použijte prohlížeč obrázků nebo externí software

Informace o prohlížení a používání zpracovaných výsledků najdete v článku [Dokončení zpracování](finishing-the-processing.md).
