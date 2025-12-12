# Sledování zpracování

Po zahájení zpracování nabízí Chloros několik způsobů, jak sledovat průběh, kontrolovat problémy a porozumět tomu, co se děje s vaším datovým souborem. Na této stránce je vysvětleno, jak sledovat zpracování a interpretovat informace, které Chloros poskytuje.

## Přehled ukazatele průběhu

Indikátor průběhu v horní záhlaví zobrazuje stav zpracování v reálném čase a procento dokončení.

### Indikátor průběhu v bezplatném režimu

Pro uživatele bez licence Chloros+:

**Zobrazení průběhu ve 2 fázích:**

1. **Detekce cíle** – vyhledání kalibračních cílů v obrazech
2. **Zpracování** – aplikace oprav a export

**Indikátor průběhu zobrazuje:**

* Celkové procento dokončení (0–100 %)
* Název aktuální fáze
* Jednoduché vizualizace horizontálního pruhu

### Indikátor průběhu Chloros+

Pro uživatele s licencí Chloros+:

**Zobrazení průběhu ve 4 fázích:**

1. **Detekce** – vyhledávání kalibračních cílů
2. **Analýza** – zkoumání obrázků a příprava potrubí
3. **Kalibrace** – aplikace korekcí viněty a odrazivosti
4. **Export** – ukládání zpracovaných souborů

**Interaktivní funkce:**

* **Přejděte myší** nad ukazatel průběhu a zobrazí se rozšířený 4stupňový panel
* **Kliknutím** na ukazatel průběhu panel rozbalíte a zafixujete
* **Dalším kliknutím** panel odblokujete a automaticky skryjete po odjetí myší
* Každá fáze zobrazuje individuální průběh (0–100 %)

***

## Popis jednotlivých fází zpracování

### Fáze 1: Detekce (detekce cílů)

**Co se děje:**

* Chloros skenuje obrázky označené zaškrtávacím políčkem Cíl
* Algoritmy počítačového vidění identifikují 4 kalibrační panely
* Hodnoty odrazivosti extrahované z každého panelu
* Časová razítka cílů zaznamenaná pro správné naplánování kalibrace

**Doba trvání:**

* S označenými cíli: 10–60 sekund
* Bez označených cílů: 5–30+ minut (skenuje všechny obrázky)

**Indikátor průběhu:**

* Detekce: 0 % → 100 %
* Počet naskenovaných obrázků
* Počet nalezených cílů

**Na co si dát pozor:**

* Pokud jsou cíle správně označeny, mělo by být dokončeno rychle.
* Pokud to trvá příliš dlouho, cíle nemusí být označeny.
* Zkontrolujte protokol ladění, zda neobsahuje zprávy „Target found“ (Cíl nalezen).

### Fáze 2: Analýza

**Co se děje:**

* Čtení metadat EXIF obrázků (časová razítka, nastavení expozice)
* Určení kalibrační strategie na základě časových razítek cílů
* Organizace fronty zpracování obrázků
* Příprava paralelních zpracovatelů (pouze Chloros+)

**Doba trvání:** 5–30 sekund

**Indikátor průběhu:**

* Analýza: 0 % → 100 %
* Rychlá fáze, obvykle se dokončí rychle

**Na co si dát pozor:**

* Měl by postupovat plynule bez přestávek
* Varování o chybějících metadatech se zobrazí v protokolu ladění

### Fáze 3: Kalibrace

**Co se děje:**

* **Debayering**: Převod RAW Bayerova vzoru na 3 kanály
* **Korekce vinětace**: Odstranění ztmavnutí okrajů objektivu
* **Kalibrace odrazivosti**: Normalizace pomocí cílových hodnot
* **Výpočet indexu**: Výpočet multispektrálních indexů
* Zpracování každého obrazu prostřednictvím celého procesu

**Doba trvání:** Většina celkové doby zpracování (60–80 %)

**Indikátor průběhu:**

* Kalibrace: 0 % → 100 %
* Aktuálně zpracovávaný obrázek
* Dokončené obrázky / Celkový počet obrázků

**Chování zpracování:**

* **Volný režim**: Zpracovává jeden obrázek za druhým postupně
* **Režim Chloros+**: Zpracovává až 16 obrázků současně
* **GPU akcelerace**: Výrazně zrychluje tuto fázi

**Na co si dát pozor:**

* Stabilní postup podle počtu obrázků
* Zkontrolujte protokol ladění pro zprávy o dokončení jednotlivých obrázků
* Varování o kvalitě obrázků nebo problémech s kalibrací

### Fáze 4: Export

**Co se děje:**

* Zápis kalibrovaných obrázků na disk ve vybraném formátu
* Export multispektrálních indexových obrázků s barvami LUT
* Vytvoření podsložek modelů kamer
* Zachování původních názvů souborů s příslušnými příponami

**Doba trvání:** 10–20 % celkové doby zpracování

**Indikátor průběhu:**

* Export: 0 % → 100 %
* Soubory se zapisují
* Formát exportu a cíl

**Na co je třeba dávat pozor:**

* Varování o volném místě na disku
* Chyby při zápisu souborů
* Dokončení všech nakonfigurovaných výstupů

***

## Karta Debug Log (Logs)

Debug Log poskytuje podrobné informace o průběhu zpracování a všech problémech, které se vyskytly.

### Přístup k Debug Log

1. Klikněte na ikonu **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> v levém postranním panelu.
2. Otevře se panel protokolu zobrazující zprávy o zpracování v reálném čase.
3. Automaticky se posouvá, aby zobrazil nejnovější zprávy.

### Porozumění zprávám protokolu

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

**Akce:** Po zpracování zkontrolujte varování, ale nepřerušujte zpracování.

#### Chybové zprávy (Red)

Kritické problémy, které mohou způsobit selhání zpracování:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Akce:** Zastavte zpracování, vyřešte chybu a restartujte.

### Běžné zprávy protokolu

| Zpráva                          | Význam                                | Potřebná akce                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| „Cíl detekován v \[název souboru]“ | Kalibrační cíl úspěšně nalezen  | Žádná – normální                                         |
| „Zpracování obrázku X z Y“        | Aktualizace aktuálního průběhu                | Žádná – normální                                         |
| „Nenalezeny žádné cíle“               | Nebyly detekovány žádné kalibrační cíle        | Označte obrázky cílů nebo deaktivujte kalibraci odrazivosti |
| „Nedostatek místa na disku“        | Nedostatek úložného prostoru pro výstup          | Uvolněte místo na disku                                    |
| „Přeskočení poškozeného souboru“        | Obrazový soubor je poškozen                  | Znovu zkopírujte soubor z karty SD                             |
| „Použita data PPK“               | Použity korekce GPS ze souboru .daq | Žádné – normální                                         |

### Kopírování dat protokolu

Kopírování protokolu pro účely řešení problémů nebo podpory:

1. Otevřete panel Debug Log (Protokol ladění).
2. Klikněte na tlačítko **„Copy Log“** (Kopírovat protokol) (nebo klikněte pravým tlačítkem myši → Vybrat vše).
3. Vložte do textového souboru nebo e-mailu.
4. V případě potřeby odešlete na podporu MAPIR.

***

## Sledování systémových zdrojů

### Využití CPU

**Volný režim:**

* 1 jádro CPU na ~100 %
* Ostatní jádra nečinná nebo dostupná
* Systém zůstává responzivní

**Chloros+ Paralelní režim:**

* Více jader na 80–100 % (až 16 jader)
* Vysoké celkové využití CPU
* Systém může být méně responzivní

**Sledování:**

* Windows Správce úloh (Ctrl+Shift+Esc)
* Karta Výkon → sekce CPU
* Vyhledejte procesy „Chloros“ nebo „chloros-backend“

### Využití paměti (RAM)

**Typické využití:**

* Malé projekty (&lt; 100 obrázků): 2–4 GB
* Střední projekty (100–500 obrázků): 4–8 GB
* Velké projekty (500+ obrázků): 8–16 GB
* Chloros+ paralelní režim využívá více RAM

**Pokud je paměť nedostatečná:**

* Zpracovávejte menší dávky
* Ukončete ostatní aplikace
* Pokud pravidelně zpracováváte velké datové soubory, upgradujte RAM

### Využití GPU (Chloros+ s CUDA)

Při zapnuté akceleraci GPU:

* GPU NVIDIA vykazuje vysoké využití (60–90 %)
* Využití VRAM se zvyšuje (vyžaduje 4 GB+ VRAM)
* Fáze kalibrace je výrazně rychlejší

**Sledujte:**

* Ikona NVIDIA v systémové liště
* Správce úloh → Výkon → GPU
* GPU-Z nebo podobný monitorovací nástroj

### Disk I/O

**Co můžete očekávat:**

* Vysoké čtení disku během fáze analýzy
* Vysoký zápis na disk během fáze exportu
* SSD je výrazně rychlejší než HDD

**Tip pro zvýšení výkonu:**

* Pokud je to možné, použijte SSD pro složku projektu
* Vyhněte se síťovým diskům pro velké datové sady
* Ujistěte se, že disk není téměř plný (ovlivňuje rychlost zápisu)

***

## Detekce problémů během zpracování

### Varovné signály

**Zastavení procesu (žádná změna po dobu 5+ minut):**

* Zkontrolujte protokol ladění, zda neobsahuje chyby.
* Ověřte dostupný prostor na disku.
* Zkontrolujte Správce úloh, zda běží Chloros.

**Časté zobrazení chybových zpráv:**

* Zastavte zpracování a zkontrolujte chyby.
* Běžné příčiny: nedostatek místa na disku, poškozené soubory, problémy s pamětí.
* Viz část Řešení problémů níže.

**Systém nereaguje:**

* Chloros+ paralelní režim využívá příliš mnoho zdrojů
* Zvažte snížení počtu souběžných úloh nebo upgrade hardwaru
* Volný režim je méně náročný na zdroje

### Kdy zastavit zpracování

Zastavte zpracování, pokud se zobrazí:

* ❌ Chyby „Disk je plný“ nebo „Nelze zapsat soubor“
* ❌ Opakované chyby poškození obrazového souboru
* ❌ Systém zcela zamrzl (nereaguje)
* ❌ Zjistili jste, že byla nakonfigurována nesprávná nastavení
* ❌ Byly importovány nesprávné obrázky

**Jak zastavit:**

1. Klikněte na **tlačítko Zastavit/Zrušit** (nahrazuje tlačítko Start)
2. Zpracování se zastaví, pokrok se ztratí
3. Opravte problémy a restartujte od začátku

***

## Řešení problémů během zpracování

### Zpracování je velmi pomalé

**Možné příčiny:**

* Neoznačené cílové obrázky (skenování všech obrázků)
* HDD místo SSD úložiště
* Nedostatečné systémové zdroje
* Konfigurováno mnoho indexů
* Přístup k síťovému disku

**Řešení:**

1. Pokud jste právě začali a jste ve fázi detekce: Zrušte, označte cíle, restartujte
2. Do budoucna: Používejte SSD, snižte počet indexů, vylepšete hardware
3. Zvažte použití CLI pro dávkové zpracování velkých datových sad

### Varování „Místo na disku“

**Řešení:**

1. Okamžitě uvolněte místo na disku
2. Přesuňte projekt na disk s větším prostorem
3. Snižte počet indexů pro export.
4. Použijte formát JPG namísto TIFF (menší soubory).

### Časté zprávy „Poškozený soubor“

**Řešení:**

1. Znovu zkopírujte obrázky z karty SD, abyste zajistili integritu.
2. Otestujte kartu SD na chyby.
3. Odstraňte poškozené soubory z projektu.
4. Pokračujte ve zpracování zbývajících obrázků.

### Přehřátí systému / omezení výkonu

**Řešení:**

1. Zajistěte dostatečné větrání.
2. Vyčistěte prach z ventilačních otvorů počítače.
3. Snižte zátěž zpracování (použijte režim Free namísto Chloros+).
4. Zpracovávejte v chladnějších částech dne.

***

## Oznámení o dokončení zpracování

Po dokončení zpracování:

* Ukazatel průběhu dosáhne 100 %
* V protokolu ladění se zobrazí zpráva **„Zpracování dokončeno“**
* Tlačítko Start se znovu aktivuje
* Všechny výstupní soubory jsou v podsložce modelu fotoaparátu

***

## Další kroky

Po dokončení zpracování:

1. **Zkontrolujte výsledky** – viz [Dokončení zpracování](finishing-the-processing.md)
2. **Zkontrolujte výstupní složku** – ověřte, zda byly všechny soubory exportovány správně
3. **Zkontrolujte protokol ladění** – zkontrolujte, zda neobsahuje varování nebo chyby
4. **Zobrazte náhled zpracovaných obrázků** – použijte prohlížeč obrázků nebo externí software

Informace o kontrole a použití zpracovaných výsledků naleznete v části [Dokončení zpracování](finishing-the-processing.md).
