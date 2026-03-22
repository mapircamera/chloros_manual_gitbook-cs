# Výběr obrázků s kalibračními cíli

Označení obrázků obsahujících kalibrační cíle je klíčovým krokem, který výrazně zrychluje zpracovatelský proces Chloros. Předběžným výběrem obrázků s kalibračními cíli eliminujete nutnost, aby Chloros prohledával každý obrázek ve vašem datovém souboru za účelem nalezení kalibračních cílů.

## Proč označovat obrázky s kalibračními cíli?

### Rychlost zpracování

Bez označení cílových obrázků musí Chloros:

* Prohledat každý jednotlivý obrázek ve vašem projektu
* Spustit algoritmy detekce cílů na každém obrázku
* Zbytečně zkontrolovat stovky nebo tisíce obrázků

**Výsledek**: Zpracování může trvat výrazně déle, zejména u velkých datových sad.

### S označenými cílovými obrázky

Když zaškrtnete sloupec Cíl u konkrétních obrázků:

* Chloros prohledává pouze zaškrtnuté obrázky
* Detekce cílů je dokončena mnohem rychleji
* Celková doba zpracování je výrazně zkrácena

{% hint style="success" %}
**Zvýšení rychlosti**: Označení 2–3 cílových obrázků v datovém souboru o 500 obrázcích může zkrátit dobu detekce cílů z více než 30 minut na méně než 1 minutu.
{% endhint %}

***

## Jak označit obrázky cílů

### Krok 1: Identifikujte obrázky cílů

Projděte si importované obrázky v prohlížeči souborů a identifikujte, které obrázky obsahují kalibrační cíle.

**Běžné scénáře:*** **Cíl před snímáním**: Zachycen před zahájením relace
* **Cíl po snímání**: Zachycen po dokončení relace
* **Cíle v terénu**: Cíle umístěné v oblasti snímání
* **Více cílů**: 2–3 cílové snímky na relaci (doporučeno)

### Krok 2: Zkontrolujte sloupec Cíl

U každého snímku obsahujícího kalibrační cíl:

1. Vyhledejte snímek v tabulce prohlížeče souborů
2. Najděte sloupec **Cíl** (sloupec zcela vpravo)
3. Zaškrtněte políčko ve sloupci Cíl u daného snímku
4. Opakujte u všech snímků obsahujících cíle

### Krok 3: Ověřte svůj výběr

Před zpracováním zkontrolujte:

* [ ] Jsou zaškrtnuty všechny snímky s kalibračními cíli
* [ ] Nejsou omylem zaškrtnuty žádné snímky bez cílů
* [ ] Cíle jsou na zaškrtnutých snímcích jasně viditelné

***

## Osvědčené postupy pro snímky cílů

### Pokyny pro snímání cílů

**Načasování:**

* Snímky cílů pořizujte bezprostředně před začátkem a v průběhu celé relace snímání
* Za stejných světelných podmínek jako váš světelný senzor DAQ
* Pro dosažení nejlepších výsledků snímávejte obrázky cílů ideálně co nejčastěji. V opačném případě budou k postupné úpravě kalibrace použita data světelného senzoru.

**Poloha kamery:**

* Držte kameru nad cílem tak, aby byl vycentrován a vyplňoval přibližně 40–60 % středu obrázku.
* Udržujte kameru rovnoběžně/v nadirální poloze vůči povrchu cíle

**Osvětlení:**

* Stejné okolní osvětlení jako u vašeho světelného senzoru DAQ
* Vyhněte se stínům na povrchu cílů
* Nezakrývejte zdroj světla svým tělem, vozidlem nebo vegetací
* Zatažená obloha poskytuje nejkonzistentnější výsledky

**Stav cíle:**

* Udržujte panely cílů čisté a suché
* Všechny 4 panely by měly být jasně viditelné a ničím nezakryté
* Cíle by měly být pokud možno kolmé/v nadirální poloze vůči zdroji světla

### Kolik snímků cílů?

**Minimálně:**1 snímek cíle na jednu relaci.**Doporučeno:** 3–5 snímků cíle na jednu relaci.**Osvědčený postup:**

* 3–5 snímků pořízených krátce po spuštění záznamu světelného senzoru
* Pro dosažení nejlepších výsledků otáčejte kameru mezi jednotlivými snímky
* Volitelně: pravidelně v průběhu relace, pokud se světelné podmínky neustále mění

***

## Práce s více kamerami

### Konfigurace se dvěma kamerami

Při současném použití dvou kamer MAPIR (např. Survey3W RGN + Survey3N OCN):

1. Zachyťte snímky cíle **oběma kamerami** současně
2. Použijte **stejný fyzický cíl** pro obě kamery
3. Označte snímky cíle pro **oba typy kamer** v prohlížeči souborů
4. Chloros použije pro kalibraci každé kamery příslušné cíle

### Sloupec Model kamery

Sloupec **Model kamery** pomáhá identifikovat, které snímky pocházejí z které kamery:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* atd.

Tento sloupec použijte k ověření, zda jste v projektu označili cíle pro každý typ kamery.

***

## Nastavení detekce cílů

### Úprava citlivosti detekce

Pokud Chloros nedetekuje vaše cíle správně, upravte tato nastavení v [Nastavení projektu](adjusting-project-settings.md):**Minimální oblast kalibračního vzorku:*** **Výchozí**: 25 pixelů
* **Zvyšte**, pokud dochází k falešným detekcím u malých artefaktů
* **Snižte**, pokud nejsou cíle detekovány**Minimální seskupení cílů:*** **Výchozí**: 60
* **Zvyšte**, pokud jsou cíle rozděleny do více detekcí
* **Snižte**, pokud nejsou cíle s barevnými odchylkami plně detekovány***

## Časté problémy s obrázky cílů

### Problém: Nebyly detekovány žádné cíle

**Možné příčiny:**

* Obrázky cílů nejsou označeny v prohlížeči souborů
* Cíl je v rámečku příliš malý (&lt; 30 % obrázku)
* Špatné osvětlení (stíny, odlesky)
* Příliš přísná nastavení detekce cílů

**Řešení:**

1. Ověřte, zda je u správných obrázků zaškrtnuto políčko Sloupec cíle
2. Zkontrolujte kvalitu obrázku cíle v náhledu
3. Pokud je kvalita špatná, pořiďte snímky cílů znovu
4. V případě potřeby upravte nastavení detekce cílů

### Problém: Falešné detekce cílů

**Možné příčiny:**

* Bílé budovy, vozidla nebo pokryv půdy zaměněné za cíle
* Světlé skvrny ve vegetaci
* Příliš nízká citlivost detekce

**Řešení:**

1. Označte pouze skutečné snímky cílů, abyste omezili rozsah detekce
2. Zvětšete minimální plochu kalibračního vzorku
3. Zvyšte minimální hodnotu shlukování cílů
4. Zajistěte, aby snímky cílů zobrazovaly pouze cíl (minimální rušivé prvky v pozadí)

***

## Kontrolní seznam pro ověření

Před zahájením zpracování ověřte výběr snímků cílů:

* [ ] Je označen alespoň 1 snímek cíle na jednu relaci
* [ ] Zaškrtávací políčka ve sloupci Cíl jsou zaškrtnuta u všech obrázků cílů
* [ ] Obrázky cílů byly pořízeny ve stejném časovém rámci jako průzkum
* [ ] Cíle jsou při kliknutí jasně viditelné v náhledu
* [ ] Všechny 4 kalibrační panely jsou viditelné na každém obrázku cíle
* [ ] Na cílech nejsou žádné stíny ani překážky
* [ ] U dvoukamerového systému: Cíle jsou označeny pro oba typy kamer

***

## Zpracování bez cílů

### Zpracování bez kalibračních cílů

Ačkoli se to pro vědeckou práci nedoporučuje, můžete provádět zpracování bez cílů:

1. Nechte všechna zaškrtávací políčka ve sloupci Cíl nezaškrtnutá
2. **Deaktivujte** „Kalibraci odrazivosti“ v nastavení projektu
3. Korekce vinětace bude i nadále aplikována
4. Výstup nebude kalibrován pro absolutní odrazivost

{% hint style="warning" %}
**Nedoporučeno**: Bez kalibrace odrazivosti představují hodnoty pixelů pouze relativní jas, nikoli vědecká měření odrazivosti. Pro přesné a opakovatelné výsledky použijte kalibrační cíle.
{% endhint %}

***

## Další kroky

Jakmile jste označili své cílové obrázky:

1. **Zkontrolujte nastavení** – viz [Úprava nastavení projektu](adjusting-project-settings.md)
2. **Spusťte zpracování** – viz [Spuštění zpracování](starting-the-processing.md)
3. **Sledujte průběh** – viz [Sledování zpracování](monitoring-the-processing.md)

Další informace o samotných kalibračních terčích najdete v části [Kalibrační terče](../calibration-targets.md).
