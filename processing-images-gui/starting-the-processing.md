# Zahájení zpracování

Jakmile jste importovali snímky, označili kalibrační cíle a nakonfigurovali nastavení projektu, můžete zahájit zpracování. Tato stránka vás provede spuštěním zpracovatelského řetězce Chloros.

## Kontrolní seznam před zpracováním

Než kliknete na tlačítko Start, zkontrolujte, zda je vše připraveno:

* [ ] **Soubory importovány** – Všechny snímky se zobrazují v prohlížeči souborů
* [ ] **Cílové snímky označeny** – Ve sloupci „Cíl“ jsou zaškrtnuty kalibrační snímky (nebo je pro LATTICE importován záznam `.daq`)
* [ ] **Modely kamer detekovány** – sloupec „Model kamery“ zobrazuje správné kamery
* [ ] **Nastavení nakonfigurováno** – nastavení projektu zkontrolováno a upraveno
* [ ] **Indexy vybrány** – požadované multispektrální indexy přidány (v případě potřeby)
* [ ] **Zvolen formát exportu** – Výstupní formát vhodný pro váš pracovní postup

{% hint style="info" %}
**Tip**: Projděte si několik snímků v prohlížeči souborů a ověřte, zda se před zpracováním načetly správně.
{% endhint %}

***

## Spuštění zpracování

### Vyhledání tlačítka Start

Tlačítko Spustit/Přehrát se nachází v horní liště aplikace Chloros:

* Poloha: Horní střed okna
* Ikona: **Tlačítko Přehrát/Spustit** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Stav: Tlačítko je aktivní (svítí), když je vše připraveno ke zpracování

### Kliknutím spustíte

1. Klikněte na **tlačítko Přehrát/Spustit** v horní liště
2. Zpracování začne okamžitě
3. Během zpracování se tlačítko změní na tlačítko **Zastavit**

4. Ukazatel průběhu se aktualizuje a zobrazuje stav zpracování

{% hint style="success" %}
**Zpracování zahájeno**: Po kliknutí Chloros automaticky provede všechny kroky zpracování – detekci cíle, debayering, kalibraci, výpočet indexu a export. Automaticky rozpozná, zda se váš projekt týká Survey3, LATTICE nebo jejich kombinace a pro každou kameru použije správný postup zpracování.
{% endhint %}

***

## Vysvětlení režimů zpracování

Chloros pracuje ve dvou různých režimech zpracování v závislosti na vaší licenci:

### Bezplatný režim (sekvenční zpracování)

**Dostupné pro všechny uživatele**

**Jak to funguje:**

* Zpracovává snímky jeden po druhém, sekvenčně
* Jednovláknový provoz
* Nižší využití paměti

**Ukazatel průběhu zobrazuje 2 fáze:**

1.**Detekce cílů** – vyhledávání kalibračních cílů
2. **Zpracování** – provedení kalibrace a export snímků**Doba zpracování:**

* Mnohem pomalejší než paralelní režim Chloros+
* Vhodné pro malé až střední datové sady (&lt; 200 snímků)

### Režim Chloros+ (paralelní zpracování)

**Vyžaduje licenci Chloros+**

**Jak to funguje:**

* Zpracovává více snímků současně pomocí [4-vláknového zpracovatelského potrubí](../processing-architecture/processing-pipeline.md)
* [Dynamická adaptace výpočtů](../processing-architecture/dynamic-compute-adaptation.md) automaticky vybere optimální strategii pro váš hardware při spuštění
* Akcelerace GPU (CUDA) s grafickými kartami NVIDIA (stolní počítače a Jetson)
* **Počet pracovníků se přizpůsobuje hardwaru**: Strategie využívající GPU spouštějí**1–4 souběžné pracovníky** (v závislosti na VRAM — Jetson s malou pamětí spouští 1, stolní GPU s 12 GB a více spouští až 4); systémy pouze s CPU spouštějí jednoho pracovníka na fyzické jádro, minus jeden**Indikátor průběhu zobrazuje 4 fáze** (odpovídající 4 vláknům v pipeline):

1. **Detekce** (vlákno 1) – Vyhledávání kalibračních cílů
2. **Analýza** (vlákno 2) – Zkoumání metadat obrazu a výpočet kalibrace
3. **Kalibrace** (vlákno 3) – odstraňování bayesovského efektu, korekce vinětace, kalibrace, výpočet indexu
4. **Export** (vlákno 4) – ukládání zpracovaných snímků a indexů**Interakce s ukazatelem průběhu:*** **Přejděte myší** nad ukazatel a zobrazí se podrobný rozbalovací panel se 4 fázemi
* **Kliknutím** na ukazatel průběhu panel zafixujete na místě
* **Dalším kliknutím** panel uvolníte a skryjete**Doba zpracování:**

* Výrazně rychlejší než v bezplatném režimu
* Akcelerace GPU dále zvyšuje rychlost

{% hint style="info" %}
**Chloros+ Rychlost**: Paralelní zpracování může být u velkých datových sad 5–10krát rychlejší než sekvenční režim. Projekt s 500 obrázky, který v bezplatném režimu trvá 2 hodiny, může být s Chloros+ dokončen za 15–20 minut.
{% endhint %}

***

## Co se děje během zpracování

### Fáze 1: Detekce cílů

**Co dělá Chloros:**

* Prohledává snímky, které jste zaškrtli ve sloupci „Cíl“ (pokud není zaškrtnuto nic, prohledá všechny snímky)
* Identifikuje kalibrační panely v každém cíli
* Extrahuje hodnoty odrazivosti z panelů cílů
* Zaznamenává časová razítka cílů pro plánování kalibrace

**Doba trvání:** 1–30 sekund (s označenými cíli), 5–30+ minut (neoznačené)

### Fáze 2: Debayering (konverze RAW)

**Co dělá Chloros:**

* Převádí data ve formátu RAW s Bayerovým vzorem na plnohodnotné tříkanálové snímky (mono moduly LATTICE zůstávají jednopásmové – u nich se debayering přeskočí a do protokolu se zapíše poznámka)
* Použije vybraný algoritmus demosaicingu
* Zachovává maximální kvalitu obrazu a detaily

**Doba trvání:** Liší se podle počtu snímků a rychlosti CPU/GPU

### Fáze 3: Kalibrace

**Co dělá Chloros:*** **Korekce vinětace**: Odstraňuje ztmavnutí okrajů způsobené objektivem
* **Kalibrace odrazivosti**: Normalizuje pomocí cílových hodnot odrazivosti a/nebo dat o slunečním záření z DAQ
* Aplikuje korekce napříč všemi pásmy/kanály
* Používá pro každý snímek vhodnou kalibrační referenci na základě časového razítka

**Doba trvání:** Většina času zpracování

### Fáze 4: Výpočet indexů

**Co dělá Chloros:**

* Vypočítává nakonfigurované multispektrální indexy (NDVI, NDRE atd.)
* Aplikuje matematické operace s pásmy na kalibrované snímky
* Generuje indexové snímky pro každý vybraný index

**Doba trvání:** Několik sekund na jeden snímek

### Fáze 5: Export

**Co dělá Chloros:**

* Ukládá zpracované snímky ve vybraném formátu
* **LATTICE fan-out**: každý surový snímek LATTICE je exportován jako všechny povolené produkty v jednom průchodu — debayering, náhled, radiance (vždy float32), odrazivost
* Zapisuje soubory do výstupní struktury projektu: `<project>/<camera>/<format>/<Product>_Images/`
* **Zachovává název zdrojového souboru** — produkt identifikuje složka, nepřidává se žádná přípona**Doba trvání:** Liší se podle formátu exportu a velikosti souboru***

## Chování zpracování

### Automatický zpracovatelský řetězec

Po spuštění běží celý řetězec automaticky:

* Není nutná žádná interakce uživatele
* Všechny nakonfigurované kroky se provádějí postupně
* Aktualizace průběhu se zobrazují v reálném čase
* Exportované soubory se ukládají na disk ihned po dokončení — hotové výstupy můžete otevírat ještě během probíhajícího zpracování

### Využití počítače během zpracování

**Režim Free:**

* Relativně nízké využití CPU (jednovláknový režim)
* Počítač zůstává odezvu pro jiné úkoly
* Je bezpečné minimalizovat okno Chloros a pracovat v jiných aplikacích

**Chloros+ Paralelní režim:**

* Vysoké využití procesoru v rámci skupiny pracovníků dané strategie
* S akcelerací GPU: Vysoké využití GPU
* Počítač může být během zpracování méně odezvěný
* Vyhněte se spouštění dalších úloh náročných na procesor

{% hint style="warning" %}
**Tip pro výkon**: Pro dosažení nejlepšího výkonu Chloros+ zavřete ostatní aplikace a nechte Chloros využívat všechny systémové zdroje.
{% endhint %}

### Zpracování nelze pozastavit (ale lze jej čistě ukončit)

* Jakmile je zpracování spuštěno, nelze jej pozastavit a později obnovit
* Kliknutím na **Stop** se běh čistě ukončí již po prvním kliknutí
* Produkty, které byly exportovány před zastavením, zůstanou na disku
* Zastavený běh poctivě vykazuje, co dokončil (viz řádky `[RUN-SUMMARY]` v protokolu)
* Nový běh spustí proces od začátku

**Tip pro plánování:** U velmi rozsáhlých projektů zvažte zpracování v dávkách nebo použití CLI pro lepší kontrolu.***

## Sledování zpracování

Během probíhajících běhů můžete:

* **Sledovat ukazatel průběhu** – Zobrazit celkové procento dokončení
* **Zobrazit aktuální fázi** – Detekce, analýza, kalibrace nebo export
* **Zkontrolovat kartu protokolu** – zobrazit podrobné zprávy a varování týkající se zpracování
* **Zobrazit náhled hotových snímků** – exportované soubory se během zpracování ukládají na disk

Podrobné informace o monitorování najdete v části [Monitorování zpracování](monitoring-the-processing.md).

***

## Zastavení zpracování

Pokud potřebujete zpracování zastavit:

### Jak zastavit

1. Najděte **tlačítko Zastavit** (během zpracování nahrazuje tlačítko Spustit)
2. Klikněte na něj jednou — na pruhu se zobrazí **„Zastavuje se...“**, zatímco se dokončuje právě zpracovávaný snímek
3. Úloha skončí v definitivním zastaveném stavu a do protokolu se zapíše přesný výpis `[RUN-SUMMARY]` toho, co bylo dokončeno

### Kdy zastavit

**Oprávněné důvody k zastavení:**

* Zjistili jste, že byla použita nesprávná nastavení
* Zapomněli jste označit cílové obrázky
* Byly importovány nesprávné snímky
* Systém běží příliš pomalu nebo nereaguje

**Po zastavení:**

* Výstupy exportované před zastavením zůstanou na disku
* Zkontrolujte a opravte případné problémy, podle potřeby upravte nastavení
* Restartujte zpracování — běh začne od začátku

***

## Odhady doby zpracování

Skutečná doba zpracování se značně liší v závislosti na:

* Počtu snímků
* Rozlišení snímků
* Vstupním formátu (RAW vs. JPG)
* Režimu zpracování (Free vs. Chloros+)
* Rychlosti procesoru a počtu jader
* Dostupnosti GPU (pouze Chloros+)
* Počtu indexů k výpočtu
* Počtu povolených exportovaných produktů (LATTICE)

### Hrubé odhady (Chloros+, snímky 12 MP, moderní procesor)

| Počet obrázků | Režim Free | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 snímků   | 15–20 min | 5–8 min        | 3–5 min        |
| 100 snímků  | 30–40 min | 10–15 min      | 5–8 min        |
| 200 snímků  | 1–1,5 hod | 20–30 min      | 10–15 min      |
| 500 snímků  | 2–3 hodiny   | 45–60 min      | 20–30 min      |
| 1 000 snímků | 4–6 hodin   | 1,5–2 hodiny      | 40–60 min      |

{% hint style="info" %}
**První spuštění**: Počáteční zpracování může trvat déle, protože Chloros vytváří mezipaměti a profily. Následné zpracování podobných datových sad bude rychlejší.
{% endhint %}

***

## Časté problémy při spuštění

### Tlačítko Spustit je deaktivované (šedé)

**Možné příčiny:**

* Nebyly importovány žádné snímky
* Backend není plně spuštěn
* Stále probíhá předchozí zpracování
* Projekt není plně načten

**Řešení:**

1. Počkejte, až se backend plně inicializuje (zkontrolujte ikonu v hlavním menu)
2. Ověřte, zda jsou obrázky importovány v prohlížeči souborů
3. Pokud tlačítko zůstává neaktivní, restartujte Chloros
4. Zkontrolujte ladicí protokol, zda neobsahuje chybové zprávy

### Zpracování se spustí, ale okamžitě selže

**Možné příčiny:**

* V projektu nejsou žádné platné obrázky
* Poškozené obrazové soubory
* Nedostatek místa na disku
* Nedostatek paměti (RAM)

**Řešení:**

1. Zkontrolujte, zda v ladicím protokolu <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> nejsou chybové zprávy
2. Ověřte, zda je k dispozici dostatek místa na disku
3. Zkuste zpracovat menší podmnožinu snímků
4. Ověřte, zda nejsou snímky poškozené

### Spuštění skončilo, ale nezapsalo žádné snímky

Spuštění, které požadovalo výstupy v podobě snímků, ale žádné nezapsalo, je považováno za **selhání, nikoli za úspěch** — Chloros na to hlasitě upozorňuje:

* Protokol grafického uživatelského rozhraní (GUI) zobrazuje hlášení `[RUN-SUMMARY]` s náznaky pravděpodobné příčiny — nebyly importovány žádné snímky, nebyl detekován žádný cíl nebo byly všechny požadované produkty přeskočeny jako nepoužitelné (např. požadavek na odrazivost/odrazivost z kamer podporujících pouze RGB)
* Ekvivalent CLI (`chloros-cli process`) vypíše `Processing finished but wrote no image products.` a **ukončí se s nenulovým stavem**, takže skripty to mohou detekovat
* Úmyslné spuštění pouze s metadaty (všechny exportované produkty deaktivovány, žádné indexy) se stále počítá jako úspěch

Úplnou sémantiku najdete v [referenční příručce k CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails).

### Varování „Nebyly detekovány žádné cíle“

**Možné příčiny:**

* Zapomněli jste označit cílové obrázky
* Cílové obrázky neobsahují viditelné cíle
* Nastavení detekce cílů je příliš přísné

**Řešení:**

1. Přečtěte si [Výběr cílových obrázků](choosing-target-images.md)
2. Označte příslušné obrázky ve sloupci „Cíl“
3. Ověřte, zda jsou cíle na označených obrázcích viditelné
4. V případě potřeby upravte nastavení detekce cílů

***

## Tipy pro úspěšné zpracování

### Před zahájením

1. **Nejprve proveďte test na malé podskupině** – Zpracujte 10–20 obrázků a ověřte nastavení
2. **Zkontrolujte volné místo na disku** – Zajistěte volné místo ve velikosti 2–3násobku velikosti datového souboru (více, pokud jsou povoleny všechny produkty LATTICE)
3. **Ukončete nepotřebné aplikace** – Uvolněte systémové zdroje
4. **Ověřte snímky cílů** – Prohlédněte si náhledy označených cílů, abyste se ujistili o jejich kvalitě
5. **Uložte projekt** – Projekt se ukládá automaticky, ale je dobré jej uložit i ručně

### Během zpracování

1. **Zabraňte přechodu systému do režimu spánku** – Vypněte režimy úspory energie
2. **Nechte Chloros v popředí** – Nebo alespoň viditelný na hlavním panelu
3. **Příležitostně sledujte průběh** – Zkontrolujte, zda se neobjevují varování nebo chyby
4. **Nespouštějte další náročné aplikace** – Zejména při paralelním režimu Chloros+

### Chloros+ Akcelerace GPU

Při použití akcelerace GPU NVIDIA:

1. Aktualizujte ovladače NVIDIA na nejnovější verzi
2. Ujistěte se, že GPU má 4 GB+ VRAM (7 GB+ pro souběžné odstraňování bayeringu s podporou textur)
3. Zavřete aplikace náročné na GPU (hry, střih videa)
4. Sledujte teplotu GPU (zajistěte dostatečné chlazení)

***

## Další kroky

Jakmile se zpracování spustí:

1. **Sledujte průběh** – viz [Sledování zpracování](monitoring-the-processing.md)
2. **Počkejte na dokončení** – zpracování probíhá automaticky
3. **Zkontrolujte výsledky** – viz [Dokončení zpracování](finishing-the-processing.md)

Informace o tom, co dělat během zpracování, najdete v části [Sledování zpracování](monitoring-the-processing.md).
