# Přidávání souborů do projektu

Jakmile v aplikaci Chloros vytvoříte nebo otevřete projekt, dalším krokem je přidání multispektrálních snímků, abyste mohli zahájit jejich zpracování. Karta „Prohlížeč souborů“ <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> usnadňuje import snímků a správu datového souboru.

## Otevření prohlížeče souborů

1. Otevřete nebo vytvořte projekt v aplikaci Chloros
2. Klikněte na ikonu **Prohlížeč souborů** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> v levém postranním panelu
3. V panelu Prohlížeč souborů se zobrazí seznam souborů vašeho projektu

{% hint style="info" %}
**Podporované typy souborů**:

* **Survey3W / Survey3N**: páry souborů RAW+JPG a snímky JPG (doporučeno RAW+JPG)
* **LATTICE**: záznamy `.tif` / `.tiff` — pořízené pomocí ovládání kamery Chloros nebo přes rozbočovač LATTICE
* **Data ze světelných senzorů**: záznamy `.daq` (DAQ-U/M/E) a záznamy směřující dolů DAQ-M `.csv` — importované společně se snímky pro kalibraci odrazivosti
{% endhint %}

***

## Přidávání snímků do projektu

Existují dva hlavní způsoby, jak přidat snímky do projektu:

### Metoda 1: Přidat soubory

Tuto možnost použijte k importu jednotlivých obrazových souborů nebo malého výběru souborů.

1. Klikněte na tlačítko **„Přidat soubory“** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> v horní části panelu Prohlížeč souborů
2. Přejděte do složky obsahující vaše obrázky
3. Vyberte jeden nebo více obrazových souborů (podržte klávesu **Ctrl** pro výběr více souborů)
4. Klikněte na **„Otevřít“** pro import vybraných souborů

### Způsob 2: Přidání složky

Tuto možnost použijte k hromadnému importu všech obrázků ze složky. V jednom dialogovém okně můžete vybrat **více složek**.

1. Klikněte na tlačítko **„Přidat složku“** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> v horní části panelu Prohlížeč souborů
2. Přejděte do složky (složek) obsahující obrázky z vaší relace snímání a vyberte ji (je)
3. Klikněte na **„Vybrat složku“** pro import všech podporovaných obrázků

{% hint style="info" %}
**Soubory, které se nepodaří načíst, jsou nahlášeny.** Pokud složka obsahuje soubory, které program Chloros rozpozná, ale nedokáže načíst, zobrazí se varování – obrázky z mřížky nezmizí bez upozornění.
{% endhint %}

***

## Import složek snímků LATTICE

Snímky LATTICE se ukládají do **jedné podsložky pro každou úroveň exportu** — například `raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` — s odpovídajícím souborem `.daq` v kořenovém adresáři:

```
output/
├── raw/           capture_<timestamp>_SN<serial>_raw.tif
├── debayered/     capture_<timestamp>_SN<serial>_debayered.tif
├── preview/       capture_<timestamp>_SN<serial>_display.tif
└── *.daq          the downwelling reading matched to the capture
```

**Nastavte možnost „Přidat složku“ na kořenový adresář snímků** (`output/` výše). Pokud vybraná složka sama o sobě neobsahuje žádné obrázky, ale má podsložky, Chloros do nich automaticky sestoupí — podsložky dané úrovně i kořenová složka `.daq` jsou načteny najednou.

**Jak se snímky importují:**

* Každý snímek se importuje jako **jediný obrázek**, seskupený podle snímku (ne jeden záznam na úroveň). Ostatní úrovně téhož snímku se zobrazují jako režimy zobrazení tohoto jediného obrázku.
* **Zpracování vždy začíná od surového snímku.** Ostatní úrovně jsou viditelné, ale do zpracovatelského řetězce se vždy dostane pouze `raw` — opětovné zpracování již zpracovaného produktu by znamenalo dvojí použití korekcí, což Chloros odmítá. Znovu importovaný export nemůže nikdy obsadit slot pro surový snímek.
* Složka se snímky uložená **bez** surových snímků se importuje a zobrazuje normálně, ale zpracování ji přeskočí a tuto skutečnost zaznamená do protokolu. (Příkazový řádek CLI s příznakem `--input-level` může v tomto případě vynutit vstupní bod — viz [Referenční příručka CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like).)
* Od verze 1.3.0 může složka snímků obsahovat také složku `clamp_masks/` (masky na úrovni jednotlivých pixelů pro úrovně radiance a odrazivosti — viz [Formáty výstupních obrázků](../output-image-formats.md#clamp-masks)). Masky nejsou obrázky scény a Chloros je při importu přeskočí, ať se nacházejí kdekoli.

**Relace LATTICE hub** se importují stejným způsobem: v sekci „Add Folder“ (Přidat složku) vyberte složku relace zkopírovanou z hubu (obsahuje složky `raw/` a `previews/`) spolu s jakýmkoli záznamem o dopadajícím záření DAQ-M `.csv`. Pokud kalibrace kamery nebo DAQ ještě není v mezipaměti vašeho počítače, Chloros ji při importu automaticky načte podle sériového čísla (vyžaduje jednorázové připojení k internetu).

***

## Jak rozumět tabulce v prohlížeči souborů

Jakmile jsou snímky importovány, zobrazí se v tabulce s následujícími sloupci:

### Název souboru

* Původní název souboru z kamery
* Zachovává konvenci pojmenování kamery (např. IMG\_0001.RAW nebo capture\_20260816\_101500\_SN213800234\_raw.tif)

### Časové razítko

* Datum a čas pořízení snímku
* Získané z metadat EXIF snímku
* Používá se pro přiřazování světelných senzorů, synchronizaci PPK a plánování kalibračních cílů

### Model fotoaparátu

* Automaticky detekovaná konfigurace kamery a filtru
* Příklady z Survey3: Survey3W_RGN, Survey3N_OCN, Survey3W_RGB
* Příklady z LATTICE: LATT-M3M-L41-F550, LATT-M3C-L87-FRGN
* Používá se k použití správných profilů zpracování

### Sloupec cílů (zaškrtávací políčko)

* Toto políčko zaškrtněte u snímků, které obsahují kalibrační cíle
* Je-li zaškrtnut alespoň jeden snímek, **jsou pro vyhledání cílů prohledávány pouze zaškrtnuté snímky**
* Podrobnosti najdete v části [Výběr snímků s cíli](choosing-target-images.md)

### Zobrazení metadat snímků

Kliknutím na přepínací tlačítko v pravém horním rohu nad tabulkou zobrazíte metadata vybraného snímku v oblasti mřížky snímků.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Soubory světelných senzorů ve vašem projektu

* Soubory `.daq` a `.csv` se zobrazují v seznamu prohlížeče souborů, ale nejedná se o klikatelné snímky — poskytují intenzitu dopadajícího záření pro kalibraci odrazivosti.
* Každý importovaný soubor typu `.daq`/`.csv` je uveden v části **Nastavení projektu → Světelný senzor DAQ**, kde můžete zkontrolovat korekci difuzního krytu platnou pro každý soubor. Viz [Úprava nastavení projektu](adjusting-project-settings.md).
* Záznamy pořízené na kartě **Světelné senzory** se do otevřeného projektu přidávají automaticky – není třeba je importovat ručně.

***

## Správa souborů v projektu

### Odstranění souborů

Chcete-li z projektu odstranit nepotřebné snímky:

1. Vyberte jeden nebo více snímků v tabulce Prohlížeče souborů
2. Klikněte na tlačítko **„Odstranit vybrané“** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
3. Potvrďte odstranění (soubory nejsou smazány z disku, pouze odstraněny z projektu)

### Třídění a filtrování

* **Třídění podle sloupce**: Kliknutím na libovolný záhlaví sloupce obrázky seřadíte
* **Třídění podle časového razítka**: Užitečné pro uspořádání chronologických sekvencí snímků
* **Filtr podle modelu fotoaparátu**: Seskupte obrázky podle typu fotoaparátu, pokud používáte více fotoaparátů***

## Náhled obrázku

### Zobrazení celého obrázku

Kliknutím na libovolnou miniaturu obrázku v prohlížeči souborů jej zobrazíte v hlavní oblasti náhledu:

1. Obrázek se zobrazí ve středním panelu náhledu
2. Pomocí ovládacích prvků pro přiblížení si prohlédněte detaily obrázku
3. Mezi obrázky se pohybujte pomocí kláves se šipkami

### Rychlá navigace

* **Předchozí snímek**: Klikněte na šipku vlevo nebo stiskněte klávesu ←
* **Další snímek**: Klikněte na šipku vpravo nebo stiskněte klávesu →
* **Přiblížení/oddálení**: Použijte kolečko myši nebo tlačítka pro změnu měřítka
* **Posun**: Při přiblížení klikněte na snímek a táhněte myší***

## Zpracování duplicitních souborů

Program Chloros automaticky detekuje a ignoruje duplicitní soubory:

* Soubory se stejnými názvy jsou přeskočeny
* Zabraňuje náhodnému dvojímu zpracování
* Při detekci duplicit se zobrazí varovná zpráva

{% hint style="warning" %}
**Důležité**: Před importem nepřejmenovávejte ani neupravujte původní obrazové soubory. Chloros se při správném zpracování spoléhá na původní názvy souborů a metadata.
{% endhint %}

***

## Smíšené sady dat z kamer

Pokud váš projekt obsahuje snímky z více kamer MAPIR:

1. Chloros automaticky detekuje každý model kamery — Survey3, LATTICE nebo jejich kombinaci
2. Každý typ kamery je zpracován s příslušným kalibračním profilem
3. Průzkumník souborů zobrazuje model kamery ve sloupci „Model kamery“
4. Každá kamera má po zpracování vlastní strom výstupních složek

**Příkladové scénáře**: konfigurace se dvěma kamerami Survey3W RGN + Survey3N OCN, nebo pole LATTICE s hlavní kamerou RGB a několika úzkopásmovými moduly***

## Osvědčené postupy

### Uspořádejte data před importem

* Uchovávejte snímky kalibračních terčů ve stejné složce jako snímky z průzkumu
* Soubory světelných senzorů `.daq` / `.csv` z každé snímací relace uchovávejte společně se snímky z dané relace
* Zachovejte původní strukturu složek z kamery/SD karty/rozbočovače
* Nemíchejte datové sady z různých snímacích relací v jednom projektu

### Pojmenování souborů

* Zachovejte původní názvy souborů z kamery (IMG\_0001.RAW, capture\_..., atd.)
* Před importem soubory nepřejmenovávejte
* Původní názvy obsahují důležitá metadata

### Snímky kalibračních terčů

* Vždy přiložte 1–2 snímky kalibračních terčů z každé relace (program Survey3; v případě LATTICE je lze nahradit záznamem DAQ — viz [Výběr snímků terčů](choosing-target-images.md))
* Snímky kalibračních terčů pořiďte před a po snímací relaci
* Umístěte terče do stejných světelných podmínek jako snímací oblast
* Označte snímky terčů zaškrtnutím políčka „Target“

***

## Časté problémy a jejich řešení

### Snímky se po importu nezobrazují

**Možné příčiny:**

* Nepodporovaný formát souboru (viz seznam podporovaných typů v horní části této stránky)
* Snímky pocházejí z kamer jiných než MAPIR (viz [Podporované kamery](../supported-cameras.md))
* Poškozený soubor nebo neúplný přenos z SD karty

**Řešení**: Ověřte kompatibilitu formátu souboru a modelu fotoaparátu a zkontrolujte varování při načítání souborů, abyste zjistili, které konkrétní soubory selhaly

### Model fotoaparátu nebyl rozpoznán

**Možné příčiny:**

* Upravená metadata EXIF
* Snímky upravené v externím softwaru
* Neúplný přenos souborů

**Řešení**: Znovu importujte původní, neupravené soubory z fotoaparátu/SD karty

### Chybějící časová razítka

**Možné příčiny:**

* Nesprávně nastavené hodiny fotoaparátu
* Data EXIF odstraněná externím softwarem

**Řešení**: Ověřte, zda bylo nastavení času fotoaparátu během pořizování snímků správné

### Znovu otevřený projekt hlásí chybějící soubory

Pokud byly zdrojové soubory od posledního otevření projektu přesunuty nebo smazány, Chloros vám sdělí, **které**soubory chybí, místo aby se otevřel prázdný rastr. Pokud byla přesunuta samotná složka projektu, Chloros přesměruje cesty ke zdrojovým souborům na složku, ve které se nyní nachází složka `project.json`, a uchová zálohu (`project.json.before-relink-<date>-<time>`). Pokud je ani tam nenajde, okno**Zdrojové soubory nebyly nalezeny**nabídne možnost**Procházet** k novému umístění a poté Vytvořit nový projekt nebo Odstranit projekt. Z příkazového řádku (CLI) použijte `chloros-cli project open PATH --relink FOLDER`; z SDK použijte `open_project(path, relink=...)`.

***

## Další kroky

Jakmile jsou soubory importovány:

1. **Zkontrolujte seznam souborů** – ujistěte se, že se všechny obrázky načtou správně
2. **Zkontrolujte modely fotoaparátů** – ověřte správnou detekci fotoaparátů
3. **Označte cílové snímky** – viz [Výběr cílových snímků](choosing-target-images.md)
4. **Upravte nastavení** – nakonfigurujte možnosti zpracování v [Nastavení projektu](adjusting-project-settings.md)
5. **Spusťte zpracování** – viz [Spuštění zpracování](starting-the-processing.md)

Podrobné informace o konfiguraci projektu najdete v části [Úprava nastavení projektu](adjusting-project-settings.md).
