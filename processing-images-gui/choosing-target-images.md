# Výběr snímků s kalibračními cíli

Označením snímků, které obsahují kalibrační cíle, sdělíte nástroji Chloros, kde přesně je má hledat. Pokud je ve sloupci „Target“ zaškrtnut alespoň jeden snímek, nástroj Chloros prohledá **pouze zaškrtnuté snímky** — označení cílů tedy slouží jak k urychlení zpracování, tak k tomu, aby nedocházelo k omylu a snímky z průzkumu nebyly zaměněny za cíle.

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

## Proč označovat cílové snímky?

### Označení řídí skenování

Když zaškrtnete sloupec „Cíl“ u konkrétních snímků:

* Chloros prohledává pouze zaškrtnuté snímky kvůli cílům
* Detekce cílů proběhne mnohem rychleji
* Snímky z průzkumu nemohou způsobit falešné detekce cílů

Pokud nejsou zaškrtnuty **žádné** snímky, Chloros se vrátí ke skenování všech snímků v projektu:

* Algoritmy detekce cílů se spustí na každém snímku
* Zbytečně se kontrolují stovky či tisíce snímků
* Zpracování trvá výrazně déle, zejména u velkých datových sad

{% hint style="success" %}
**Zrychlení**: Označení 2–3 snímků s cíli v datovém souboru o 500 snímcích může zkrátit dobu detekce cílů z více než 30 minut na méně než 1 minutu.
{% endhint %}

***

## Jak označit snímky cílů

### Krok 1: Identifikujte snímky cílů

Prohlédněte si importované snímky v prohlížeči souborů a určete, které snímky obsahují kalibrační cíle.

**Běžné scénáře:*** **Cíl před pořízením**: Pořízený před zahájením relace
* **Cíl po snímání**: Zachycen po dokončení relace
* **Cíle v terénu**: Cíle umístěné v oblasti snímání
* **Více cílů**: 2–3 snímky cílů na relaci (doporučeno)

### Krok 2: Zkontrolujte sloupec **Target** <img src="../.gitbook/assets/image (33).png" alt="" data-size="original">

U každého snímku obsahujícího kalibrační cíl:

1. Vyhledejte snímek v tabulce prohlížeče souborů
2. Najděte sloupec **Target** (sloupec zcela vpravo)
3. Zaškrtněte políčko ve sloupci **Target** u daného snímku
4. Opakujte tento postup u všech snímků obsahujících kalibrační cíle

### Krok 3: Ověřte svůj výběr

Před zpracováním ještě jednou zkontrolujte:

* [ ] Jsou zaškrtnuty všechny snímky s kalibračními cíli
* [ ] Nejsou omylem zaškrtnuty žádné snímky bez kalibračních cílů
* [ ] Kalibrační cíle jsou na zaškrtnutých snímcích jasně viditelné

***

## LATTICE: Kalibrační cíle jsou volitelné při záznamu pomocí DAQ

U multispektrálních kamer LATTICE je kalibrační cíl v rámci snímku **jedním ze dvou** možných referenčních bodů odrazivosti:

* **Cíl v rámci snímku**: pokud označený snímek s cílem projde kontrolou kvality (QA) vChloros, stane se tento cíl**absolutní referencí odrazivosti** pro snímky v jeho okolí.
* **Sestupné záření DAQ**: pokud není k dispozici žádný cíl (nebo selže kontrola kvality), systém „Chloros“ vypočítá odrazivost namísto toho na základě sestupného ozáření světelného senzoru DAQ (ρ = π·L/E). Pokud vaše snímky pokrývá záznam z `.daq` nebo DAQ-M `.csv`, získáte kalibrovanou odrazivost**zcela bez jakýchkoli snímků cílů**.

Toto automatické chování je výchozí. V souboru CLI / SDK tomu odpovídá `--reflectance-source auto`; můžete také vynutit režim `target` (přísný — bez nahrazení dat z DAQ) nebo `daq` (s předností dat z DAQ). Viz [CLI Referenční příručka](../reference/cli-reference.md#per-product-export-toggles-lattice-multispectral).

**Geometrie cílů LATTICE**: kromě klasické detekce panelů používané pro Survey3 podporuje zpracování LATTICE**cíle označené ArUco**,**cíle s pevnou oblastí zájmu (ROI)**a**pásové cíle**, konfigurované pro každý projekt zvlášť. Skeny**naměřené** odrazivosti cíle pro jednotlivé jednotky lze poskytnout podle sériového čísla (CLI: `--target-reflectance-dir`, jeden `<serial>.csv` na každou cílovou jednotku), přičemž jako záložní řešení slouží nominální spektra T3/T4P.

{% hint style="info" %}
**Modul F988**: Odrazivost F988 se kalibruje pomocí panelu odrazivosti v dané scéně: pásmo leží mimo kalibrovaný rozsah světelného senzoru DAQ, proto Chloros použije váš nejnovější záznam panelu a zachová jej mezi jednotlivými měřeními panelu. Pokud je modul F988 zpracováván pouze pomocí DAQ, Chloros odmítne odrazivost založenou na DAQ pro dané pásmo (důvod přeskočení `dls-uncalibrated-band-988`) — podporovaným postupem je práce s panelem.
{% endhint %}

***

## Osvědčené postupy pro snímky cíle

### Pokyny pro snímání cíle

**Načasování:**

* Snímky cíle pořizujte bezprostředně před začátkem a během celé snímací relace
* Za stejných světelných podmínek jako u světelného senzoru DAQ
* Pro dosažení nejlepších výsledků snímávejte snímky cíle ideálně co nejčastěji. V opačném případě budou k postupné úpravě kalibrace použita data ze světelného senzoru.

**Poloha kamery:**

* Držte kameru nad cílem tak, aby byl vycentrován a zabíral přibližně 40–60 % středu snímku.
* Udržujte fotoaparát rovnoběžně s povrchem cíle nebo v nadirální poloze vůči němu

**Osvětlení:**

* Stejné okolní osvětlení jako u světelného senzoru DAQ
* Vyhněte se stínům na povrchu cíle
* Nezakrývejte zdroj světla svým tělem, vozidlem ani vegetací
* Zatažená obloha poskytuje nejkonzistentnější výsledky

**Stav cíle:**

* Udržujte panely cíle čisté a suché.
* Všechny panely cíle (např. všechny 4 u modelu T4) by měly být jasně viditelné a nic by je nemělo zakrývat.
* Cíle by měly být pokud možno v kolmém směru nebo přímo nad zdrojem světla.

### Kolik snímků cíle?

**Minimálně:**1 snímek cíle na jednu relaci.**Doporučeno:** 3–5 snímků cíle na jednu relaci.**Osvědčený postup:**

* Pořiďte 3–5 snímků krátce po zahájení záznamu světelným senzorem
* Pro dosažení nejlepších výsledků otáčejte kameru mezi jednotlivými snímky
* Volitelně: pravidelně v průběhu relace, pokud se světelné podmínky neustále mění

***

## Práce s více kamerami

### Konfigurace se dvěma kamerami

Při současném použití dvou kamer typu „MAPIR“ (např. Survey3W RGN + Survey3N OCN):

1. Pořizujte snímky cílů **oběma kamerami** současně
2. Pro obě kamery použijte **stejný fyzický cíl**

3. V prohlížeči souborů označte snímky cílů pro**oba typy kamer**

4. Nástroj Chloros použije pro kalibraci každé kamery příslušné cíle

### Sloupec „Model kamery“

Sloupec **„Model kamery“** pomáhá identifikovat, které snímky pocházejí z které kamery:

* Survey3W\_RGN
* Survey3N\_OCN
* LATT-M3M-L41-F550
* LATT-M3C-L87-FRGN
* atd.

Tento sloupec použijte k ověření, zda jste v projektu označili cíle pro každý typ kamery.

***

## Nastavení detekce cílů

### Úprava citlivosti detekce

Pokud funkce „Chloros“ nedetekuje vaše cíle správně, upravte tato nastavení v [Nastavení projektu](adjusting-project-settings.md):**Minimální plocha kalibračního vzorku (px):*** **Výchozí hodnota**: 25 pixelů
* **Zvyšte**, pokud dochází k falešným detekcím u malých artefaktů
* **Snižte**, pokud se cíle nedetekují**Minimální shlukování cílů (0–100):*** **Výchozí**: 60
* **Zvyšte**, pokud se cíle rozdělují na více detekcí
* **Snižte**, pokud nejsou cíle s barevnými odchylkami plně detekovány

{% hint style="info" %}
**Tip k CLI**: `chloros-cli process` podporuje stejné parametry (`--min-target-size`, `--target-clustering`), a jeho příznak `--target`/`--targets` označuje celou vstupní složku jako určenou pouze pro panel cílů. Viz [CLI Referenční příručka](../reference/cli-reference.md).
{% endhint %}

***

## Časté problémy s cílovými obrázky

### Problém: Nebyly detekovány žádné cíle

**Možné příčiny:**

* Cílové obrázky nejsou označeny v prohlížeči souborů
* Cíl je v záběru příliš malý (&lt; 30 % obrázku)
* Špatné osvětlení (stíny, odlesky)
* Příliš přísná nastavení detekce cílů

**Řešení:**

1. Ověřte, zda je u správných obrázků zaškrtnutý sloupec „Cíl“
2. Zkontrolujte kvalitu snímků cílů v náhledu
3. Pokud je kvalita špatná, pořiďte snímky cílů znovu
4. V případě potřeby upravte nastavení detekce cílů

### Problém: Falešná detekce cílů

**Možné příčiny:**

* Bílé budovy, vozidla nebo povrch terénu jsou mylně považovány za cíle
* Světlé skvrny ve vegetaci
* Příliš nízká citlivost detekce

**Řešení:**

1. Označte pouze skutečné snímky cílů — skenovány budou pouze zaškrtnuté snímky
2. Zvětšete minimální plochu kalibračního vzorku
3. Zvětšete minimální hodnotu shlukování cílů
4. Zajistěte, aby snímky cílů zobrazovaly pouze cíl (minimální rušivé prvky v pozadí)

***

## Kontrolní seznam pro ověření

Před zahájením zpracování ověřte výběr snímků cílů:

* [ ] Je označen alespoň 1 snímek cíle na jednu relaci (nebo, v případě LATTICE, záznam `.daq`/`.csv` pokrývající danou relaci)
* [ ] Zaškrtnuta políčka ve sloupci „Cíl“ u všech snímků cílů
* [ ] Snímky cílů pořízené ve stejném časovém rámci jako průzkum
* [ ] Cíle jsou po kliknutí jasně viditelné v náhledu
* [ ] Všechny kalibrační panely jsou viditelné na každém snímku cíle
* [ ] Na cílech nejsou žádné stíny ani překážky
* [ ] U dvoukamerového systému: Cíle jsou označeny pro oba typy kamer

***

## Zpracování bez cílů

### LATTICE: S DAQ záznamem

Pokud světelný senzor DAQ zaznamenal během snímání pomocí LATTICE intenzitu slunečního záření dopadajícího na zem, není třeba žádný cíl:

1. Načtěte soubor `.daq` (nebo DAQ-M `.csv`) se snímky
2. Nechte sloupec „Target“ nezaškrtnutý
3. Odrazivost se automaticky vypočítá z referenční hodnoty slunečního záření směřujícího dolů z DAQ
4. Radiance nikdy nevyžaduje cíl ani DAQ — vychází výhradně z tovární radiometrické kalibrace kamery

### Zpracování bez jakékoli reference

Zpracování můžete provést i bez cílů a bez DAQ:

1. Nechte všechna zaškrtávací políčka ve sloupci „Cíl“ nezaškrtnutá
2. **Deaktivujte** v nastavení projektu možnost „Kalibrace odrazivosti / vyvážení bílé“ – detekce cílů se pak zcela přeskočí
3. Korekce vinětace bude i nadále aplikována
4. Výstup nebude kalibrován na absolutní odrazivost (multispektrální nástroj LATTICE i nadále exportuje produkty po odstranění bayerského filtru, náhledy a produkty radiance)

{% hint style="warning" %}
**Nedoporučuje se pro vědeckou práci s technologií „Survey3“**: Bez kalibrace odrazivosti představují hodnoty pixelů v režimu „Survey3“ pouze relativní jas, nikoli vědecká měření odrazivosti. Pro přesné a opakovatelné výsledky použijte kalibrační cíle (nebo v případě LATTICE světelný senzor DAQ).
{% endhint %}

***

## Další kroky

Jakmile označíte cílové snímky:

1. **Zkontrolujte nastavení** – viz [Úprava nastavení projektu](adjusting-project-settings.md)
2. **Spusťte zpracování** – viz [Spuštění zpracování](starting-the-processing.md)
3. **Sledujte průběh** – viz [Sledování zpracování](monitoring-the-processing.md)

Další informace o samotných kalibračních cílech najdete v části [Kalibrační cíle](../calibration-targets.md).
