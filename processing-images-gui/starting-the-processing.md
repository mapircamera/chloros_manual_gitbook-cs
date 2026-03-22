# Spuštění zpracování

Jakmile jste načetli snímky, označili kalibrační cíle a nakonfigurovali nastavení projektu, můžete zahájit zpracování. Tato stránka vás provede spuštěním zpracovatelského postupu Chloros.

## Kontrolní seznam před zpracováním

Než kliknete na tlačítko Start, ověřte, zda je vše připraveno:

* [ ] **Soubory importovány** – Všechny snímky se zobrazují v prohlížeči souborů
* [ ] **Cílové snímky označeny** – Sloupec Cíl je zaškrtnut u kalibračních snímků
* [ ] **Modely kamer detekovány** – Sloupec Model kamery zobrazuje správné kamery
* [ ] **Nastavení nakonfigurováno** – Nastavení projektu zkontrolováno a upraveno
* [ ] **Vybrané indexy** – Požadované multispektrální indexy byly přidány (v případě potřeby)
* [ ] **Zvolený formát exportu** – Výstupní formát vhodný pro váš pracovní postup

{% hint style="info" %}
**Tip**: Projděte si několik snímků v prohlížeči souborů a ověřte, zda se před zpracováním načtou správně.
{% endhint %}

***

## Spuštění zpracování

### Najděte tlačítko Start

Tlačítko Start/Přehrát se nachází v horní liště Chloros:

* Poloha: Horní střed okna
* Ikona: **Tlačítko Přehrát/Start** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
* Stav: Tlačítko je aktivní (svítí), když je připraveno ke zpracování

### Kliknutím spustíte

1. Klikněte na **tlačítko Přehrát/Spustit** v horní liště
2. Zpracování začne okamžitě
3. Během zpracování se tlačítko deaktivuje (zešedne)
4. Ukazatel průběhu se aktualizuje a zobrazuje stav zpracování

{% hint style="success" %}
**Zahájení zpracování**: Po kliknutí Chloros automaticky provede všechny kroky zpracování – detekci cíle, debayering, kalibraci, výpočet indexu a export.
{% endhint %}

***

## Vysvětlení režimů zpracování

Chloros pracuje ve dvou různých režimech zpracování v závislosti na vaší licenci:

### Bezplatný režim (sekvenční zpracování)

**K dispozici pro všechny uživatele**

**Jak to funguje:**

* Zpracovává snímky jeden po druhém, sekvenčně
* Jednovláknový provoz
* Nižší využití paměti

**Indikátor průběhu zobrazuje 2 fáze:**

1.**Detekce cíle** – Vyhledávání kalibračních cílů
2. **Zpracování** – Aplikace kalibrace a export obrázků**Doba zpracování:**

* Mnohem pomalejší než paralelní režim Chloros+
* Vhodné pro malé až střední datové sady (&lt; 200 obrázků)

### Režim Chloros+ (paralelní zpracování)

**Vyžaduje licenci Chloros+**

**Jak to funguje:**

* Zpracovává více obrázků současně pomocí [4-vláknového zpracovatelského potrubí](../processing-architecture/processing-pipeline.md)
* [Dynamická adaptace výpočtů](../processing-architecture/dynamic-compute-adaptation.md) automaticky vybere optimální strategii pro váš hardware
* Akcelerace GPU (CUDA) s grafickými kartami NVIDIA (stolní počítače a Jetson)
* Škálovatelné od Jetson Nano (1 pracovník) po stolní počítač s GPU 12 GB+ (3–4 pracovníci)

**Indikátor průběhu zobrazuje 4 fáze** (odpovídající 4 vláknům pipeline):

1. **Detekce** (vlákno 1) – Vyhledávání kalibračních cílů
2. **Analýza** (vlákno 2) – Prozkoumání metadat obrazu a výpočet kalibrace
3. **Kalibrace** (vlákno 3) – GPU debayering, korekce vinětace, výpočet indexu
4. **Export** (vlákno 4) – Uložení zpracovaných obrázků a indexů**Interakce s ukazatelem průběhu:*** **Přejděte myší** nad ukazatel a zobrazte podrobný rozbalovací panel se 4 fázemi
* **Klikněte** na ukazatel, aby se rozbalovací panel zafixoval na místě
* **Klikněte znovu** pro uvolnění a skrytí panelu**Doba zpracování:**

* Výrazně rychlejší než bezplatný režim
* Škáluje se s počtem jader CPU
* Akcelerace GPU dále zvyšuje rychlost

{% hint style="info" %}
**Chloros+ Rychlost**: Paralelní zpracování může být u velkých datových sad 5–10krát rychlejší než sekvenční režim. Projekt s 500 obrázky, který v bezplatném režimu trvá 2 hodiny, může být s Chloros+ dokončen za 15–20 minut.
{% endhint %}

***

## Co se děje během zpracování

### Fáze 1: Detekce cílů

**Co dělá Chloros:**

* Prohledává označené cílové snímky (nebo všechny snímky, pokud nejsou žádné označené)
* Identifikuje 4 kalibrační panely v každém cíli
* Extrahuje hodnoty odrazivosti z cílových panelů
* Zaznamenává časová razítka cílů pro plánování kalibrace

**Doba trvání:** 1–30 sekund (s označenými cíli), 5–30+ minut (neoznačené)

### Fáze 2: Demosaicing (konverze RAW)

**Co dělá Chloros:**

* Převádí data ve formátu RAW s Bayerovým vzorem na plnohodnotné snímky RGB
* Používá vysoce kvalitní algoritmus demosaicingu
* Zachovává maximální kvalitu a detaily snímku

**Doba trvání:** Liší se podle počtu snímků a rychlosti procesoru

### Fáze 3: Kalibrace

**Co dělá Chloros:*** **Korekce vinětace**: Odstraňuje ztmavnutí okrajů objektivu
* **Kalibrace odrazivosti**: Normalizuje pomocí cílových hodnot odrazivosti
* Aplikuje korekce napříč všemi pásmy/kanály
* Používá vhodný kalibrační cíl pro každý obrázek na základě časového razítka

**Doba trvání:** Většina času zpracování

### Fáze 4: Výpočet indexu

**Co dělá Chloros:**

* Vypočítává nakonfigurované multispektrální indexy (NDVI, NDRE atd.)
* Aplikuje matematické operace s pásmy na kalibrované snímky
* Generuje indexové snímky pro každý vybraný index

**Trvání:** Několik sekund na jeden snímek

### Fáze 5: Export

**Co dělá Chloros:**

* Uloží kalibrované snímky ve vybraném formátu
* Exportuje indexové snímky s nakonfigurovanými barvami LUT
* Zapíše soubory do podsložek podle modelů kamer
* Zachová původní názvy souborů s příponami

**Doba trvání:** Liší se podle formátu exportu a velikosti souboru***

## Chování zpracování

### Automatický zpracovatelský pipeline

Po spuštění běží celý pipeline automaticky:

* Není nutná žádná interakce uživatele
* Všechny nakonfigurované kroky se provádějí postupně
* Aktualizace průběhu se zobrazují v reálném čase

### Využití počítače během zpracování

**Režim Free:**

* Relativně nízké využití CPU (jednovláknový)
* Počítač zůstává odezvu pro jiné úkoly
* Bezpečné minimalizovat Chloros a pracovat v jiných aplikacích

**Chloros+ Paralelní režim:**

* Vysoké využití CPU (vícevláknové, až 16 jader)
* S akcelerací GPU: Vysoké využití GPU
* Počítač může být během zpracování méně odezvu
* Vyhněte se spouštění jiných úloh náročných na CPU

{% hint style="warning" %}
**Tip pro výkon**: Pro nejlepší Chloros+ výkon zavřete ostatní aplikace a nechte Chloros využívat plné systémové zdroje.
{% endhint %}

### Zpracování nelze pozastavit

**Důležité omezení:**

* Jakmile je zpracování spuštěno, nelze jej pozastavit
* Zpracování můžete zrušit, ale pokrok se ztratí
* Částečné výsledky se neukládají
* V případě zrušení je nutné začít od začátku

**Tip pro plánování:** U velmi rozsáhlých projektů zvažte zpracování v dávkách nebo použití CLI pro lepší kontrolu.***

## Sledování zpracování

Během zpracování můžete:

* **Sledovat ukazatel průběhu** – Zobrazit celkové procento dokončení
* **Zobrazit aktuální fázi** – Detekce, analýza, kalibrace nebo export
* **Zkontrolovat kartu protokolu** – Zobrazit podrobné zprávy a varování týkající se zpracování
* **Zobrazit náhled hotových obrázků** – Některé exportované soubory se mohou zobrazit během zpracování

Podrobné informace o sledování naleznete v části [Sledování zpracování](monitoring-the-processing.md).

***

## Zrušení zpracování

Pokud potřebujete zpracování zastavit:

### Jak zrušit

1. Najděte **tlačítko Zastavit/Zrušit** (během zpracování nahrazuje tlačítko Spustit)
2. Klikněte na tlačítko Zastavit
3. Zpracování se okamžitě zastaví
4. Částečné výsledky budou zahozeny

### Kdy zrušit

**Platné důvody pro zrušení:**

* Zjistili jste, že byla použita nesprávná nastavení
* Zapomněli jste označit cílové obrázky
* Byly importovány nesprávné obrázky
* Systém běží příliš pomalu nebo nereaguje

**Po zrušení:**

* Zkontrolujte a opravte případné problémy
* Upravte nastavení podle potřeby
* Spusťte zpracování znovu od začátku
* Pro co nejčistší výsledek úplně zavřete Chloros a restartujte

{% hint style="warning" %}
**Žádné částečné výsledky**: Zrušení vymaže veškerý pokrok. Chloros neukládá částečně zpracované obrázky.
{% endhint %}

***

## Odhady doby zpracování

Skutečná doba zpracování se značně liší v závislosti na:

* Počtu obrázků
* Rozlišení obrázků
* Vstupním formátu RAW vs. JPG
* Režimu zpracování (Free vs. Chloros+)
* Rychlosti procesoru a počtu jader
* Dostupnosti GPU (pouze Chloros+)
* Počtu indexů k výpočtu
* Složitosti exportního formátu

### Hrubé odhady (Chloros+, snímky 12 MP, moderní procesor)

| Počet obrázků | Režim Free | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 obrázků   | 15–20 min | 5–8 min        | 3–5 min        |
| 100 obrázků  | 30–40 min | 10–15 min      | 5–8 min        |
| 200 obrázků  | 1–1,5 hod. | 20–30 min      | 10–15 min      |
| 500 obrázků  | 2–3 hodiny   | 45–60 min      | 20–30 min      |
| 1000 obrázků | 4–6 hodin   | 1,5–2 hodiny      | 40–60 min      |

{% hint style="info" %}
**První spuštění**: Počáteční zpracování může trvat déle, protože Chloros vytváří mezipaměti a profily. Následné zpracování podobných datových sad bude rychlejší.
{% endhint %}

***

## Časté problémy při spuštění

### Tlačítko Start je deaktivováno (zašedlé)

**Možné příčiny:**

* Nejsou importovány žádné obrázky
* Backend není plně spuštěn
* Stále probíhá předchozí zpracování
* Projekt není plně načten

**Řešení:**

1. Počkejte, až se backend plně inicializuje (zkontrolujte ikonu v hlavním menu)
2. Ověřte, zda jsou obrázky importovány v prohlížeči souborů
3. Restartujte Chloros, pokud tlačítko zůstává deaktivované
4. Zkontrolujte protokol ladění, zda neobsahuje chybové zprávy

### Zpracování se spustí, ale okamžitě selže

**Možné příčiny:**

* V projektu nejsou žádné platné obrázky
* Poškozené obrazové soubory
* Nedostatek místa na disku
* Nedostatek paměti (RAM)

**Řešení:**

1. Zkontrolujte ladicí protokol <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> , zda neobsahuje chybové zprávy
2. Ověřte dostupné místo na disku
3. Zkuste zpracovat menší podmnožinu obrázků
4. Ověřte, zda obrázky nejsou poškozené

### Varování „Nebyly detekovány žádné cíle“

**Možné příčiny:**

* Zapomněli jste označit cílové obrázky
* Cílové obrázky neobsahují viditelné cíle
* Nastavení detekce cílů je příliš přísné

**Řešení:**

1. Přečtěte si [Výběr cílových obrázků](choosing-target-images.md)
2. Označte příslušné obrázky ve sloupci Cíl
3. Ověřte, zda jsou cíle na označených obrázcích viditelné
4. V případě potřeby upravte nastavení detekce cílů

***

## Tipy pro úspěšné zpracování

### Před spuštěním

1. **Nejprve proveďte test s malou podskupinou** – Zpracujte 10–20 obrázků a ověřte nastavení
2. **Zkontrolujte volné místo na disku** – Zajistěte 2–3násobek velikosti datového souboru
3. **Ukončete nepotřebné aplikace** – Uvolněte systémové zdroje
4. **Ověřte cílové obrázky** – Zkontrolujte náhled označených cílů a ujistěte se o jejich kvalitě
5. **Uložte projekt** – Projekt se ukládá automaticky, ale je dobré jej uložit i ručně

### Během zpracování

1. **Zabraňte přechodu systému do režimu spánku** – Vypněte režimy úspory energie
2. **Nechte Chloros v popředí** – Nebo alespoň viditelný na hlavním panelu
3. **Příležitostně sledujte průběh** – Zkontrolujte, zda se neobjevují varování nebo chyby
4. **Nespouštějte jiné náročné aplikace** – Zejména v paralelním režimu Chloros+

### Chloros+ Akcelerace GPU

Pokud používáte akceleraci GPU NVIDIA:

1. Aktualizujte ovladače NVIDIA na nejnovější verzi
2. Ujistěte se, že GPU má 4 GB+ VRAM
3. Zavřete aplikace náročné na GPU (hry, střih videa)
4. Sledujte teplotu GPU (zajistěte dostatečné chlazení)

***

## Další kroky

Jakmile se zpracování spustí:

1. **Sledujte průběh** – viz [Sledování zpracování](monitoring-the-processing.md)
2. **Počkejte na dokončení** – zpracování probíhá automaticky
3. **Zkontrolujte výsledky** – viz [Dokončení zpracování](finishing-the-processing.md)

Informace o tom, co dělat během zpracování, najdete v části [Sledování zpracování](monitoring-the-processing.md).
