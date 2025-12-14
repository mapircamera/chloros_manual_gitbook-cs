# Zahájení zpracování

Jakmile importujete obrázky, označíte kalibrační cíle a nakonfigurujete nastavení projektu, můžete zahájit zpracování. Tato stránka vás provede zahájením zpracování Chloros.

## Kontrolní seznam před zpracováním

Než kliknete na tlačítko Start, zkontrolujte, zda je vše připraveno:

* [ ] **Soubory importovány** – všechny obrázky se zobrazují v prohlížeči souborů
* [ ] **Obrázky cílů označeny** – sloupec Cíl zkontrolován pro kalibrační obrázky
* [ ] **Detekovány modely kamer** – ve sloupci Camera Model jsou zobrazeny správné kamery
* [ ] **Nastavení nakonfigurováno** – nastavení projektu zkontrolováno a upraveno
* [ ] **Indexy vybrány** – přidány požadované multispektrální indexy (v případě potřeby)
* [ ] **Formát exportu vybrán** – výstupní formát vhodný pro váš pracovní postup

{% hint style=&quot;info&quot; %}
**Tip**: Před zpracováním klikněte na několik obrázků v prohlížeči souborů a ověřte, zda se načtou správně.
{% endhint %}

***

## Spuštění zpracování

### Vyhledání tlačítka Start

Tlačítko Start/Přehrát se nachází v horní liště záhlaví Chloros:

* Poloha: V horní části okna uprostřed
* Ikona: **Tlačítko Přehrát/Start** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">
* Stav: Tlačítko je aktivní (světlé), když je připraveno ke zpracování

### Klikněte pro spuštění

1. Klikněte na **tlačítko Přehrát/Spustit** v horní liště
2. Zpracování začne okamžitě
3. Tlačítko se během zpracování deaktivuje (zšedne)
4. Ukazatel průběhu se aktualizuje a zobrazuje stav zpracování

{% hint style=&quot;success&quot; %}
**Zpracování zahájeno**: Po kliknutí Chloros automaticky zpracuje všechny kroky zpracování – detekci cíle, debayering, kalibraci, výpočet indexu a export.
{% endhint %}

***

## Porozumění režimům zpracování

Chloros pracuje ve dvou různých režimech zpracování v závislosti na vaší licenci:

### Bezplatný režim (sekvenční zpracování)

**K dispozici pro všechny uživatele**

**Jak to funguje:**

* Zpracovává obrázky jeden po druhém, sekvenčně.
* Jednovláknový provoz.
* Nižší využití paměti.

**Indikátor průběhu zobrazuje 2 fáze:**

1. **Detekce cíle** – vyhledávání kalibračních cílů.
2. **Zpracování** – aplikace kalibrace a export obrázků.

**Doba zpracování:**

* Mnohem pomalejší než paralelní režim Chloros+
* Vhodné pro malé až střední datové sady (&lt; 200 obrázků)

### Režim Chloros+ (paralelní zpracování)

**Vyžaduje licenci Chloros+**

**Jak to funguje:**

* Zpracovává více obrázků současně
* Vícevláknový provoz (až 16 paralelních pracovníků)
* Využívá více jader CPU
* Volitelná akcelerace GPU (CUDA) s grafickými kartami NVIDIA

**Indikátor průběhu zobrazuje 4 fáze:**

1. **Detekce** – vyhledávání kalibračních cílů
2. **Analýza** – zkoumání metadat obrázků a příprava potrubí
3. **Kalibrace** – aplikace oprav a kalibrací
4. **Export** – ukládání zpracovaných obrázků a indexů

**Interakce s ukazatelem průběhu:**

* **Přejděte myší** nad ukazatel a zobrazí se podrobný rozbalovací panel se 4 fázemi
* **Kliknutím** na ukazatel průběhu rozbalovací panel zafixujete na místě
* **Dalším kliknutím** panel uvolníte a skryjete

**Doba zpracování:**

* Výrazně rychlejší než volný režim
* Škáluje se podle počtu jader CPU
* Akcelerace GPU dále zvyšuje rychlost

{% hint style=&quot;info&quot; %}
**Chloros+ Rychlost**: Paralelní zpracování může být u velkých datových sad 5–10krát rychlejší než sekvenční režim. Projekt s 500 obrázky, který v bezplatném režimu trvá 2 hodiny, lze s Chloros+ dokončit za 15–20 minut.
{% endhint %}

***

## Co se děje během zpracování

### Fáze 1: Detekce cíle

**Co dělá Chloros:**

* Skenuje označené cílové obrázky (nebo všechny obrázky, pokud nejsou žádné označené)
* Identifikuje 4 kalibrační panely v každém cíli
* Extrahuje hodnoty odrazivosti z cílových panelů
* Zaznamenává časová razítka cílů pro plánování kalibrace

**Doba trvání:** 1–30 sekund (s označenými cíli), 5–30+ minut (bez označení)

### Fáze 2: Debayering (konverze RAW)

**Co dělá Chloros:**

* Konvertuje data RAW Bayerova vzoru na plné obrázky RGB
* Používá vysoce kvalitní algoritmus demosaicingu
* Zachovává maximální kvalitu a detaily obrazu

**Trvání:** Liší se podle počtu obrázků a rychlosti CPU

### Fáze 3: Kalibrace

**Co dělá Chloros:**

* **Korekce vinětace**: Odstraňuje ztmavnutí objektivu na okrajích
* **Kalibrace odrazivosti**: Normalizuje pomocí cílových hodnot odrazivosti
* Aplikuje korekce napříč všemi pásmy/kanály
* Používá vhodný kalibrační cíl pro každý obraz na základě časového razítka

**Doba trvání:** Většina času zpracování

### Fáze 4: Výpočet indexu

**Co dělá Chloros:**

* Vypočítává nakonfigurované multispektrální indexy (NDVI, NDRE atd.)
* Aplikuje matematické operace s pásmy na kalibrované snímky
* Generuje indexové snímky pro každý vybraný index

**Doba trvání:** Několik sekund na jeden snímek

### Fáze 5: Export

**Co dělá Chloros:**

* Ukládá kalibrované snímky ve vybraném formátu
* Exportuje indexové snímky s nakonfigurovanými barvami LUT
* Zapisuje soubory do podsložek modelu kamery
* Zachovává původní názvy souborů s příponami

**Doba trvání:** Liší se podle formátu exportu a velikosti souboru

***

## Chování zpracování

### Automatický zpracovatelský proces

Po spuštění běží celý proces automaticky:

* Není nutná žádná interakce uživatele
* Všechny nakonfigurované kroky se provádějí postupně
* Aktualizace průběhu se zobrazují v reálném čase

### Využití počítače během zpracování

**Volný režim:**

* Relativně nízké využití CPU (jednovláknové)
* Počítač zůstává reagovatelný pro jiné úkoly
* Bezpečné minimalizovat Chloros a pracovat v jiných aplikacích

**Chloros+ Paralelní režim:**

* Vysoké využití CPU (vícevláknové, až 16 jader)
* S akcelerací GPU: Vysoké využití GPU
* Počítač může být během zpracování méně responzivní
* Vyhněte se spouštění jiných úkolů náročných na CPU

{% hint style=&quot;warning&quot; %}
**Tip pro výkon**: Pro nejlepší výkon Chloros+ zavřete ostatní aplikace a nechte Chloros využívat všechny systémové zdroje.
{% endhint %}

### Zpracování nelze pozastavit

**Důležité omezení:**

* Po spuštění nelze zpracování pozastavit
* Zpracování můžete zrušit, ale pokrok se ztratí
* Částečné výsledky se neukládají
* V případě zrušení je nutné začít znovu od začátku

**Tip pro plánování:** U velmi rozsáhlých projektů zvažte zpracování v dávkách nebo použití CLI pro lepší kontrolu.

***

## Sledování zpracování

Během zpracování můžete:

* **Sledovat ukazatel průběhu** – zobrazit celkové procento dokončení
* **Zobrazit aktuální fázi** – detekce, analýza, kalibrace nebo export
* **Zkontrolovat kartu protokolu** – zobrazit podrobné zprávy a varování o zpracování
* **Zobrazit náhled dokončených obrázků** – během zpracování se mohou zobrazit některé exportované soubory

Podrobné informace o monitorování naleznete v části [Monitorování zpracování](monitoring-the-processing.md).

***

## Zrušení zpracování

Pokud potřebujete zpracování zastavit:

### Jak zrušit

1. Najděte **tlačítko Zastavit/Zrušit** (během zpracování nahrazuje tlačítko Spustit).
2. Klikněte na tlačítko Zastavit.
3. Zpracování se okamžitě zastaví.
4. Částečné výsledky se vymažou.

### Kdy zrušit

**Platné důvody pro zrušení:**

* Zjistili jste, že byly použity nesprávné nastavení.
* Zapomněli jste označit cílové obrázky.
* Byly importovány nesprávné obrázky.
* Systém běží příliš pomalu nebo nereaguje.

**Po zrušení:**

* Zkontrolujte a opravte všechny problémy.
* Upravte nastavení podle potřeby.
* Restartujte zpracování od začátku.
* Pro nejčistší zážitek zcela zavřete Chloros a restartujte.

{% hint style=&quot;warning&quot; %}
**Žádné částečné výsledky**: Zrušením se vymaže veškerý pokrok. Chloros neukládá částečně zpracované obrázky.
{% endhint %}

***

## Odhadovaná doba zpracování

Skutečná doba zpracování se značně liší v závislosti na:

* Počtu obrázků
* Rozlišení obrázků
* Vstupním formátu RAW vs. JPG
* Režimu zpracování (Free vs. Chloros+)
* Rychlosti CPU a počtu jader
* Dostupnosti GPU (pouze Chloros+)
* Počtu indexů k výpočtu
* Složitosti exportního formátu

### Hrubé odhady (Chloros+, obrázky 12 MP, moderní CPU)

| Počet obrázků | Režim Free | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 obrázků   | 15–20 min | 5–8 min        | 3–5 min        |
| 100 obrázků  | 30–40 min | 10–15 min      | 5–8 min        |
| 200 obrázků  | 1–1,5 hodiny | 20–30 minut      | 10–15 minut      |
| 500 obrázků  | 2–3 hodiny   | 45–60 minut      | 20–30 minut      |
| 1000 obrázků | 4–6 hodin   | 1,5–2 hodiny      | 40–60 minut      |

{% hint style=&quot;info&quot; %}
**První spuštění**: Počáteční zpracování může trvat déle, protože Chloros vytváří mezipaměti a profily. Následné zpracování podobných datových sad bude rychlejší.
{% endhint %}

***

## Časté problémy při spuštění

### Tlačítko Start je deaktivované (šedé)

**Možné příčiny:**

* Nejsou importovány žádné obrázky.
* Backend není zcela spuštěn.
* Předchozí zpracování stále běží.
* Projekt není zcela načten.

**Řešení:**

1. Počkejte, až se backend zcela inicializuje (zkontrolujte ikonu v hlavním menu).
2. Ověřte, zda jsou obrázky importovány v prohlížeči souborů
3. Pokud tlačítko zůstává deaktivované, restartujte Chloros
4. Zkontrolujte, zda v protokolu ladění nejsou chybové zprávy

### Zpracování se spustí, ale okamžitě selže

**Možné příčiny:**

* V projektu nejsou žádné platné obrázky
* Poškozené obrazové soubory
* Nedostatečný prostor na disku
* Nedostatečná paměť (RAM)

**Řešení:**

1. Zkontrolujte protokol ladění <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> , zda neobsahuje chybové zprávy.
2. Zkontrolujte dostupný prostor na disku.
3. Zkuste zpracovat menší podmnožinu obrázků.
4. Zkontrolujte, zda obrázky nejsou poškozené.

### Varování „Nebyly detekovány žádné cíle“

**Možné příčiny:**

* Zapomněli jste označit cílové obrázky.
* Cílové obrázky neobsahují viditelné cíle.
* Nastavení detekce cílů je příliš přísné.

**Řešení:**

1. Projděte si [Výběr cílových obrázků](choosing-target-images.md)
2. Označte příslušné obrázky ve sloupci Cíl
3. Ověřte, zda jsou cíle viditelné v označených obrázcích
4. V případě potřeby upravte nastavení detekce cílů

***

## Tipy pro úspěšné zpracování

### Před spuštěním

1. **Nejprve proveďte test s malou podskupinou** – zpracujte 10–20 obrázků, abyste ověřili nastavení.
2. **Zkontrolujte dostupný prostor na disku** – zajistěte 2–3násobek volného místa oproti velikosti datového souboru.
3. **Zavřete nepotřebné aplikace** – uvolněte systémové zdroje.
4. **Ověřte cílové obrázky** – zobrazte náhled označených cílů, abyste se ujistili o jejich kvalitě.
5. **Uložte projekt** – projekt se ukládá automaticky, ale je dobré jej uložit ručně.

### Během zpracování

1. **Zabraňte přechodu systému do režimu spánku** – deaktivujte režimy úspory energie.
2. **Nechte Chloros v popředí** – nebo alespoň viditelný na hlavním panelu.
3. **Příležitostně sledujte průběh** – zkontrolujte, zda se nezobrazují varování nebo chyby.
4. **Nenačtěte jiné náročné aplikace** – zejména v paralelním režimu Chloros+.

### Chloros+ GPU akcelerace

Při použití GPU akcelerace NVIDIA:

1. Aktualizujte ovladače NVIDIA na nejnovější verzi.
2. Ujistěte se, že GPU má 4 GB+ VRAM.
3. Zavřete aplikace náročné na GPU (hry, editace videa).
4. Sledujte teplotu GPU (zajistěte dostatečné chlazení).

***

## Další kroky

Po zahájení zpracování:

1. **Sledujte průběh** – viz [Sledování zpracování](monitoring-the-processing.md).
2. **Počkejte na dokončení** – zpracování probíhá automaticky.
3. **Zkontrolujte výsledky** – viz [Dokončení zpracování](finishing-the-processing.md).

Informace o tom, co dělat během zpracování, najdete v části [Sledování zpracování](monitoring-the-processing.md).
