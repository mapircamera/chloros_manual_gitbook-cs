# Výběr cílových obrázků

Označení obrázků, které obsahují kalibrační cíle, je klíčovým krokem, který výrazně urychluje zpracování Chloros. Předběžným výběrem cílových obrázků eliminujete potřebu, aby Chloros skenoval každý obrázek ve vašem datovém souboru za účelem nalezení kalibračních cílů.

## Proč označovat cílové obrázky?

### Rychlost zpracování

Bez označení cílových obrázků musí Chloros:

* Prohledat každý jednotlivý obrázek ve vašem projektu.
* Spustit algoritmy detekce cílů na každém obrázku.
* Zbytečně zkontrolovat stovky nebo tisíce obrázků.

**Výsledek**: Zpracování může trvat výrazně déle, zejména u velkých datových sad.

### S označenými cílovými obrázky

Když zaškrtnete sloupec Cíl u konkrétních obrázků:

* Chloros prohledá pouze zaškrtnuté obrázky pro cíle
* Detekce cílů je mnohem rychlejší
* Celková doba zpracování je výrazně zkrácena

{% hint style=&quot;success&quot; %}
**Zrychlení**: Označení 2–3 cílových obrázků v datovém souboru o 500 obrázcích může zkrátit dobu detekce cílů z více než 30 minut na méně než 1 minutu.
{% endhint %}

***

## Jak označit cílový obrázek

### Krok 1: Identifikujte své cílové obrázky

Prohlédněte si importované obrázky v prohlížeči souborů a identifikujte, které obrázky obsahují kalibrační cíle.

**Běžné scénáře:**

* **Cíl před pořízením**: Pořízený před zahájením relace
* **Cíl po pořízení**: Pořízený po dokončení relace
* **Cíle v terénu**: Cíle umístěné v oblasti pořízení
* **Více cílů**: 2–3 cílové obrázky na relaci (doporučeno)

### Krok 2: Zkontrolujte sloupec Cíl

Pro každý obrázek obsahující kalibrační cíl:

1. Najděte obrázek v tabulce prohlížeče souborů.
2. Najděte sloupec **Cíl** (sloupec zcela vpravo).
3. Zaškrtněte políčko ve sloupci Cíl pro daný obrázek.
4. Opakujte pro všechny obrázky obsahující cíle.

### Krok 3: Ověřte svůj výběr

Před zpracováním zkontrolujte:

* [ ] Jsou zaškrtnuty všechny snímky s kalibračními cíli.
* [ ] Nejsou omylem zaškrtnuty žádné snímky bez cíle.
* [ ] Cíle jsou na zaškrtnutých snímcích jasně viditelné.

***

## Osvědčené postupy pro snímky s cíli

### Pokyny pro pořizování snímků s cíli

**Načasování:**

* Pořizujte snímky s cíli bezprostředně před a během pořizování snímků.
* Za stejných světelných podmínek, jaké má světelný senzor DAQ.
* Pro dosažení nejlepších výsledků pořizujte snímky cílů ideálně co nejčastěji. V opačném případě budou k úpravě kalibrace v průběhu času použita data světelného senzoru.

**Poloha kamery:**

* Držte kameru nad cílem tak, aby byl vycentrován a vyplňoval přibližně 40–60 % středu snímku.
* Udržujte kameru rovnoběžně/nad cílovou plochou.

**Osvětlení:**

* Stejné okolní osvětlení jako u vašeho světelného senzoru DAQ.
* Vyhněte se stínům na povrchu cíle.
* Nezakrývejte zdroj světla svým tělem, vozidlem nebo vegetací.
* Nejkonzistentnější výsledky poskytují zatažené podmínky.

**Stav cíle:**

* Udržujte panely cíle čisté a suché.
* Všechny 4 panely by měly být jasně viditelné a ničím nezakryté.
* Cíle by měly být pokud možno kolmé/nadir ke zdroji světla.

### Kolik snímků cíle?

**Minimálně:** 1 snímek cíle na relaci. **Doporučeno:** 3–5 snímků cíle na relaci.

**Osvědčený postup:**

* 3–5 snímků pořízených krátce po zahájení záznamu světelného senzoru
* Pro dosažení nejlepších výsledků otáčejte kamerou mezi jednotlivými snímky
* Volitelně: pravidelně v polovině relace, pokud se podmínky osvětlení neustále mění

***

## Práce s více kamerami

### Nastavení dvou kamer

Při současném použití dvou kamer MAPIR (např. Survey3W RGN + Survey3N OCN):

1. Zachyťte cílové snímky **oběma kamerami** současně.
2. Použijte **stejný fyzický cíl** pro obě kamery.
3. Označte cílové snímky pro **oba typy kamer** v prohlížeči souborů.
4. Chloros použije pro kalibraci každé kamery vhodné cíle.

### Sloupec Model kamery

Sloupec **Model kamery** pomáhá identifikovat, které snímky pocházejí z které kamery:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* atd.

Tento sloupec použijte k ověření, zda jste v projektu označili cíle pro každý typ kamery.

***

## Nastavení detekce cílů

### Nastavení citlivosti detekce

Pokud Chloros nedetekuje vaše cíle správně, upravte tato nastavení v [Nastavení projektu](adjusting-project-settings.md):

**Minimální kalibrační vzorek:**

* **Výchozí**: 25 pixelů
* **Zvyšte**, pokud dochází k falešným detekcím u malých artefaktů.
* **Snižte**, pokud cíle nejsou detekovány.

**Minimální seskupení cílů:**

* **Výchozí**: 60
* **Zvyšte**, pokud jsou cíle rozděleny do více detekcí
* **Snižte**, pokud cíle s barevnými odchylkami nejsou plně detekovány

***

## Časté problémy s obrázky cílů

### Problém: Nejsou detekovány žádné cíle

**Možné příčiny:**

* Obrázky cílů nejsou označeny v prohlížeči souborů
* Cíl je v rámečku příliš malý (&lt; 30 % obrázku)
* Špatné osvětlení (stíny, odlesky)
* Příliš přísná nastavení detekce cílů

**Řešení:**

1. Ověřte, zda je ve sloupci Cíl zaškrtnuto správné obrázky
2. Zkontrolujte kvalitu obrázku cíle v náhledu
3. Pokud je kvalita špatná, znovu zachyťte cíle
4. V případě potřeby upravte nastavení detekce cílů

### Problém: Falešné detekce cílů

**Možné příčiny:**

* Bílé budovy, vozidla nebo porost zaměněné za cíle
* Světlé skvrny ve vegetaci
* Příliš nízká citlivost detekce

**Řešení:**

1. Označte pouze skutečné snímky cílů, abyste omezili rozsah detekce.
2. Zvětšete minimální kalibrační vzorkovací plochu.
3. Zvětšete minimální hodnotu shlukování cílů.
4. Ujistěte se, že snímky cílů zobrazují pouze cíl (minimální rušení pozadí).

***

## Kontrolní seznam pro ověření

Před zahájením zpracování ověřte výběr obrázků cílů:

* [ ] Nejméně 1 obrázek cíle označený za relaci
* [ ] Zaškrtávací políčka ve sloupci Cíl jsou zaškrtnuta pro všechny obrázky cílů
* [ ] Obrázky cílů pořízené ve stejném časovém rámci jako průzkum
* [ ] Cíle jsou při kliknutí jasně viditelné v náhledu
* [ ] Všechny 4 kalibrační panely jsou viditelné na každém obrázku cíle
* [ ] Na cílech nejsou žádné stíny ani překážky.
* [ ] U duálních kamer: Cíle jsou označeny pro oba typy kamer.

***

## Zpracování bez cílů

### Zpracování bez kalibračních cílů

Ačkoli se to pro vědeckou práci nedoporučuje, můžete provádět zpracování bez cílů:

1. Nechte všechna zaškrtávací políčka ve sloupci Cíl nezaškrtnutá.
2. **Deaktivujte** „Kalibraci odrazivosti“ v nastavení projektu.
3. Korekce vinětace bude i nadále aplikována.
4. Výstup nebude kalibrován pro absolutní odrazivost.

{% hint style=&quot;warning&quot; %}
**Nedoporučuje se**: Bez kalibrace odrazivosti představují hodnoty pixelů pouze relativní jas, nikoli vědecká měření odrazivosti. Pro přesné a opakovatelné výsledky použijte kalibrační cíle.
{% endhint %}

***

## Další kroky

Jakmile označíte cílové obrázky:

1. **Zkontrolujte nastavení** – viz [Úprava nastavení projektu](adjusting-project-settings.md)
2. **Spusťte zpracování** – viz [Spuštění zpracování](starting-the-processing.md)
3. **Sledujte průběh** – viz [Sledování zpracování](monitoring-the-processing.md)

Další informace o kalibračních terčích naleznete v části [Kalibrační terče](../calibration-targets.md).
