# Zahájení zpracování

Jakmile jste naimportovali snímky, označili kalibrační cíle a nakonfigurovali nastavení projektu, můžete zahájit zpracování. Tato stránka vás provede spuštěním zpracovatelského řetězce Chloros.

## Kontrolní seznam před zpracováním

Než kliknete na tlačítko „Start“, zkontrolujte, zda je vše připraveno:

* [ ] **Soubory importovány** – Všechny snímky se zobrazují v prohlížeči souborů
* [ ] **Cílové snímky označeny** – Ve sloupci „Target“ jsou zaškrtnuty kalibrační snímky (nebo byl pro LATTICE importován záznam `.daq`)
* [ ] **Modely kamer detekovány** – Sloupec „Model kamery“ zobrazuje správné kamery
* [ ] **Nastavení nakonfigurováno** – Nastavení projektu zkontrolováno a upraveno
* [ ] **Indexy vybrány** – Požadované multispektrální indexy přidány (v případě potřeby)
* [ ] **Zvolen formát exportu** – Výstupní formát vhodný pro váš pracovní postup

{% hint style="info" %}
**Tip**: Projděte si v prohlížeči souborů několik snímků, abyste se před zpracováním ujistili, že se načtly správně.
{% endhint %}

***

## Spuštění zpracování

### Vyhledání tlačítka Start

Tlačítko Start/Přehrát se nachází v horní liště aplikace Chloros:

* Poloha: Horní střed okna
* Ikona: **Tlačítko Přehrát/Start** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Stav: Tlačítko je aktivní (svítí), když je vše připraveno ke zpracování

### Kliknutím spustíte

1. Klikněte na **tlačítko Přehrát/Spustit** v horní liště
2. Zpracování začne okamžitě
3. Během zpracování se tlačítko změní na **tlačítko Zastavit**

4. Ukazatel průběhu se aktualizuje a zobrazuje stav zpracování

{% hint style="success" %}
**Zpracování zahájeno**: Po kliknutí Chloros automaticky provede všechny kroky zpracování – detekci cíle, debayering, kalibraci, výpočet indexu a export. Automaticky rozpozná, zda se jedná o projekt typu Survey3, LATTICE nebo jejich kombinaci, a pro každou kameru použije správný postup zpracování.
{% endhint %}

***

## Vysvětlení režimů zpracování

Chloros pracuje ve dvou různých režimech zpracování v závislosti na vaší licenci:

### Bezplatný režim (sekvenční zpracování)

**Dostupné pro všechny uživatele**

**Jak to funguje:**

* Zpracovává snímky jeden po druhém, postupně
* Jednovláknový provoz
* Nižší využití paměti

**Ukazatel průběhu zobrazuje 2 fáze:**

1.**Detekce cílů** – Vyhledávání kalibračních cílů
2. **Zpracování** – Aplikace kalibrace a export snímků**Doba zpracování:**

* Mnohem pomalejší než paralelní režim Chloros+
* Vhodné pro malé až střední datové sady (< 200 snímků)

### Režim Chloros+ (paralelní zpracování)

**Vyžaduje licenci Chloros+**

**Jak to funguje:**

* Zpracovává více snímků současně pomocí [4-vláknového zpracovatelského potrubí](../processing-architecture/processing-pipeline.md)
* [Dynamická adaptace výpočtů](../processing-architecture/dynamic-compute-adaptation.md) automaticky vybere optimální strategii pro váš hardware při spuštění
* Akcelerace pomocí GPU (CUDA) s grafickými kartami NVIDIA
* **Počet pracovníků se přizpůsobuje hardwaru**: Strategie využívající GPU spouštějí**2–4 souběžné pracovníky** (škálováno podle VRAM — 3 na GPU s 2–12 GB, 2 až 4 na GPU s 12 GB a více); Systémy pouze s CPU spouštějí jednoho pracovníka na každé fyzické jádro, minus jeden**Ukazatel průběhu zobrazuje 4 fáze** (odpovídající 4 vláknům v pipeline):

1. **Detekce** (vlákno 1) – Vyhledávání kalibračních cílů
2. **Analýza** (vlákno 2) – Prozkoumání metadat obrázku a výpočet kalibrace
3. **Kalibrace** (vlákno 3) – Odstranění bayesovského efektu, korekce vinětace, kalibrace, výpočet indexu
4. **Export** (vlákno 4) – Uložení zpracovaných obrázků a indexů**Interakce s ukazatelem průběhu:*** **Přejděte myší** nad ukazatel a zobrazí se podrobný rozbalovací panel se 4 fázemi
* **Kliknutím** na ukazatel průběhu panel zafixujete na místě
* **Dalším kliknutím** panel uvolníte a skryjete**Doba zpracování:**

* Výrazně rychlejší než v bezplatném režimu
* Akcelerace pomocí GPU dále zvyšuje rychlost

{% hint style="info" %}
**Rychlost Chloros+**: Paralelní zpracování může být u velkých datových sad 5–10krát rychlejší než sekvenční režim. Projekt s 500 obrázky, který v bezplatném režimu trvá 2 hodiny, může být s Chloros+ dokončen za 15–20 minut.
{% endhint %}

***

## Co se děje během zpracování

### Fáze 1: Detekce cílů

**Co dělá Chloros:**

* Prohledává snímky, které jste zaškrtli ve sloupci „Cíl“ (pokud není zaškrtnuto nic, prohledá všechny snímky)
* Identifikuje kalibrační panely v každém cíli
* Extrahuje hodnoty odrazivosti z kalibračních panelů
* Zaznamenává časová razítka cílů pro plánování kalibrace

**Doba trvání:** 1–30 sekund (s označenými cíli), 5–30+ minut (neoznačené)

### Fáze 2: Debayering (konverze RAW)

**Co dělá Chloros:**

* Převádí data ve formátu RAW s Bayerovým vzorem na plnohodnotné tříkanálové snímky (mono moduly LATTICE zůstávají jednopásmové – u nich se debayering přeskočí a do protokolu se zapíše poznámka)
* Použije vybraný algoritmus demosaicingu
* Zachovává maximální kvalitu a detaily snímku

**Doba trvání:** Liší se podle počtu snímků a rychlosti CPU/GPU

### Fáze 3: Kalibrace

**Co Chloros dělá:*** **Korekce vinětace**: Odstraňuje ztmavnutí okrajů způsobené objektivem
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

**Co Chloros dělá:**

* Ukládá zpracované snímky ve vybraném formátu
* **LATTICE fan-out**: každý surový snímek LATTICE je exportován jako všechny povolené produkty v jednom průchodu — debayered, náhled, radiance (vždy float32), odrazivost
* Ukládá soubory do výstupní struktury projektu: `<project>/<camera>/<format>/<Product>_Images/`
* **Zachovává název zdrojového souboru** — produkt identifikuje složka, nepřidává se žádná přípona**Doba trvání:** Liší se podle formátu exportu a velikosti souboru***

## Chování zpracování

### Automatický zpracovatelský řetězec

Po spuštění běží celý řetězec automaticky:

* Není nutná žádná interakce uživatele
* Všechny nakonfigurované kroky se provádějí postupně
* Aktualizace průběhu se zobrazují v reálném čase
* Exportované soubory se ukládají na disk ihned po dokončení — hotové výstupy můžete otevírat, zatímco zpracování pokračuje

### Využití počítače během zpracování

**Režim Free:**

* Relativně nízké využití CPU (jednovláknový režim)
* Počítač zůstává odezvu pro jiné úkoly
* Je bezpečné minimalizovat Chloros a pracovat v jiných aplikacích

**Režim Chloros+ Parallel:**

* Vysoké využití procesoru v rámci skupiny pracovníků dané strategie
* S akcelerací GPU: Vysoké využití grafické karty
* Počítač může být během zpracování méně odezvu
* Vyhněte se spouštění dalších úloh náročných na procesor

{% hint style="warning" %}
**Tip pro výkon**: Pro dosažení nejlepšího výkonu Chloros+ zavřete ostatní aplikace a nechte Chloros využívat všechny systémové zdroje.
{% endhint %}

### Zpracování nelze pozastavit (ale zastavení je čisté)

* Jakmile je zpracování spuštěno, nelze jej pozastavit a později obnovit
* Kliknutím na **Zastavit** se běh čistě ukončí již při prvním kliknutí
* Výstupy již exportované před zastavením zůstanou na disku
* Zastavený běh poctivě vykazuje, co dokončil (viz řádky `[RUN-SUMMARY]` v protokolu)
* Nový běh spustí proces od začátku

**Tip pro plánování:** U velmi rozsáhlých projektů zvažte zpracování v dávkách nebo použití příkazového řádku (CLI) pro lepší kontrolu.***

## Sledování zpracování

Během zpracování můžete:

* **Sledovat ukazatel průběhu** – zobrazit celkové procento dokončení
* **Zobrazit aktuální fázi** – detekce, analýza, kalibrace nebo export
* **Zkontrolovat kartu protokolu** – zobrazit podrobné zprávy a varování týkající se zpracování
* **Zobrazit náhled dokončených snímků** – exportované soubory se během zpracování objevují na disku

Podrobné informace o sledování najdete v článku [Sledování zpracování](monitoring-the-processing.md).

***

## Zastavení zpracování

Pokud potřebujete zpracování zastavit:

### Jak zastavit

1. Najděte **tlačítko Zastavit** (během zpracování nahrazuje tlačítko Spustit)
2. Jednou na něj klikněte – na liště se zobrazí **„Zastavuje se...“**, dokud se nedokončí právě zpracovávaný snímek
3. Úloha skončí v definitivním zastaveném stavu a do protokolu se zapíše přesný výpis `[RUN-SUMMARY]` toho, co bylo dokončeno

### Kdy zastavit

**Oprávněné důvody k zastavení:**

* Zjistili jste, že byla použita nesprávná nastavení
* Zapomněli jste označit cílové snímky
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
* Dostupnosti grafické karty (pouze Chloros+)
* Počtu indexů k výpočtu
* Počtu povolených exportovaných produktů (LATTICE)

### Hrubé odhady (Chloros+, snímky 12 MP, moderní procesor)

| Počet snímků | Režim Free | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 obrázků   | 15–20 min | 5–8 min        | 3–5 min        |
| 100 obrázků  | 30–40 min | 10–15 min      | 5–8 min        |
| 200 obrázků  | 1–1,5 hod. | 20–30 min      | 10–15 min      |
| 500 obrázků  | 2–3 hod.   | 45–60 min      | 20–30 min      |
| 1 000 obrázků | 4–6 hodin   | 1,5–2 hodiny      | 40–60 min      |

{% hint style="info" %}
**První spuštění**: Počáteční zpracování může trvat déle, protože Chloros vytváří mezipaměti a profily. Následné zpracování podobných datových sad bude rychlejší.
{% endhint %}

***

## Časté problémy při spuštění

### Tlačítko Spustit je deaktivované (zašedlé)

**Možné příčiny:**

* Nebyly importovány žádné obrázky
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

* V projektu nejsou žádné platné snímky
* Poškozené obrazové soubory
* Nedostatek místa na disku
* Nedostatek paměti (RAM)

**Řešení:**

1. Zkontrolujte ladicí protokol <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">, zda neobsahuje chybové zprávy
2. Ověřte dostupné místo na disku
3. Zkuste zpracovat menší podmnožinu obrázků
4. Ověřte, zda obrázky nejsou poškozené

### Spuštění skončilo, ale nezapsalo žádné obrázky

Spuštění, které požadovalo výstupy v podobě obrázků, ale žádný nezapsalo, je považováno za **selhání, nikoli za úspěch** — Chloros na to hlasitě upozorňuje:

* Protokol grafického rozhraní (GUI) vypisuje hlášení `[RUN-SUMMARY]` s náznaky pravděpodobné příčiny — nebyly importovány žádné snímky, nebyl detekován žádný cíl nebo byl každý požadovaný produkt přeskočen jako nepoužitelný (např. požadavek na radianci/reflektanci z kamer podporujících pouze RGB)
* Ekvivalent v CLI (`chloros-cli process`) vypíše `Processing finished but wrote no image products.` a **ukončí se s nenulovým kódem**, takže skripty to mohou detekovat
* Úmyslné spuštění pouze s metadaty (všechny exportované produkty jsou deaktivovány, žádné indexy) se stále počítá jako úspěch

Úplnou sémantiku najdete v [referenční příručce k CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails).

### Složka projektu je pro tento počítač příliš hluboko

**Pouze pro Windows, novinka ve verzi 1.3.0.** Windows se dodává s vypnutou podporou dlouhých cest, přičemž žádná cesta k souboru nesmí být delší než 260 znaků. Produkty Chloros se nacházejí až asi 130 znaků pod projektovou složkou, takže projektová složka, která je hlouběji než tato hloubka, nemůže přijímat své produkty. Místo toho, aby se spustil a nic nezapsal, Chloros nyní před zahájením zpracování provede kontrolu:

* Kliknutí na tlačítko Start nespustí běh: ukazatel průběhu zobrazí důvod a v ladicím protokolu se objeví kód `[PROCESS] refused: The project folder is too deep for this computer…`. Zpráva uvádí délku projektové složky a nejdelší projektovou složku, která se vejde.
* Vytvoření nebo otevření projektu ve složce, která je příliš hluboko i pro vlastní soubory projektu, je rovněž odmítnuto. V okně se zobrazí důvod, nejdelší povolená délka složky projektu a návod, jak povolit dlouhé cesty; nic se nevytvoří ani neotevře; v ladicím protokolu se to zaznamená jako `[PROJECT] refused: …`. Duplikování projektu pod delším názvem může rovněž překročit limit: kopie se vytvoří, ale neotevře, a stejná zpráva uvádí důvod.

**Řešení:**

1. Ukládejte projekty do složky s kratším názvem — změňte**Pracovní adresář** v [Nastavení projektu](../project-settings/project-settings.md) a projekt vytvořte znovu, nebo
2. Zapněte dlouhé cesty ve Windows a restartujte Chloros: spusťte instalační program Chloros znovu s zaškrtnutou volbou **Povolit dlouhé cesty ve Windows** (ve výchozím nastavení je zaškrtnuta), nebo použijte zásady skupiny: *Konfigurace počítače > Šablony pro správu > Systém > Souborový systém > Povolit dlouhé cesty Win32*.

### Varování „Nebyly detekovány žádné cíle“

**Možné příčiny:**

* Zapomněli jste označit cílové obrázky
* Cílové obrázky neobsahují viditelné cíle
* Nastavení detekce cílů je příliš přísné

**Řešení:**

1. Projděte si [Výběr cílových obrázků](choosing-target-images.md)
2. Označte příslušné obrázky ve sloupci „Cíl“
3. Ověřte, zda jsou cíle na označených obrázcích viditelné
4. V případě potřeby upravte nastavení detekce cílů

***

## Tipy pro úspěšné zpracování

### Před spuštěním

1. **Nejprve proveďte test s malou podskupinou** – Zpracujte 10–20 snímků k ověření nastavení
2. **Zkontrolujte volné místo na disku** – Zajistěte 2–3násobek velikosti datového souboru (více, pokud jsou povoleny všechny produkty LATTICE)
3. **Ukončete nepotřebné aplikace** – uvolněte systémové zdroje
4. **Ověřte snímky s cíli** – zkontrolujte náhledy označených cílů, abyste zajistili kvalitu
5. **Uložte projekt** – projekt se ukládá automaticky, ale je dobré jej uložit i ručně

### Během zpracování

1. **Zabraňte přechodu systému do režimu spánku** – Vypněte režimy úspory energie
2. **Nechte Chloros v popředí** – Nebo alespoň viditelný na hlavním panelu
3. **Příležitostně sledujte průběh** – Zkontrolujte, zda se neobjevují varování nebo chyby
4. **Nespouštějte jiné náročné aplikace** – Zejména v paralelním režimu Chloros+

### Akcelerace Chlorosu+ pomocí GPU

Při použití akcelerace pomocí GPU NVIDIA:

1. Aktualizujte ovladače NVIDIA na nejnovější verzi
2. Ujistěte se, že GPU má 4 GB+ VRAM (7 GB+ pro souběžné odstraňování bayeringu s podporou textur)
3. Ukončete aplikace náročné na GPU (hry, střih videa)
4. Sledujte teplotu GPU (zajistěte dostatečné chlazení)

***

## Další kroky

Jakmile se zpracování spustí:

1. **Sledujte průběh** – viz [Sledování zpracování](monitoring-the-processing.md)
2. **Počkejte na dokončení** – zpracování probíhá automaticky
3. **Zkontrolujte výsledky** – viz [Dokončení zpracování](finishing-the-processing.md)

Informace o tom, co dělat během zpracování, najdete v části [Sledování zpracování](monitoring-the-processing.md).
