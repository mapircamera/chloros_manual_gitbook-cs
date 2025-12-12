# Přidání souborů do projektu

Jakmile vytvoříte nebo otevřete projekt v Chloros, dalším krokem je přidání multispektrálních snímků, abyste mohli zahájit zpracování. Karta Prohlížeč souborů<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> usnadňuje import obrázků a správu datových sad.

## Přístup k prohlížeči souborů

1. Otevřete nebo vytvořte projekt v Chloros
2. Klikněte na ikonu **Prohlížeč souborů** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> v levém postranním panelu.
3. Panel Prohlížeč souborů zobrazí seznam souborů vašeho projektu.

{% hint style=&quot;info&quot; %}
**Podporované typy souborů**: Chloros podporuje obrazové soubory RAW+JPG a JPG z fotoaparátů MAPIR Survey3W a Survey3N. Doporučujeme pouze formát RAW+JPG.
{% endhint %}

***

## Přidávání obrázků do projektu

Existují dva základní způsoby, jak přidat obrázky do projektu:

### Metoda 1: Přidání souborů

Tuto možnost použijte k importu jednotlivých obrazových souborů nebo malého výběru souborů.

1. Klikněte na tlačítko **„Přidat soubory“** v horní části panelu Prohlížeč souborů.
2. Přejděte do složky obsahující vaše obrázky.
3. Vyberte jeden nebo více obrazových souborů (podržte klávesu **Ctrl** pro výběr více souborů).
4. Klikněte na **„Otevřít“** pro import vybraných souborů.

### Metoda 2: Přidání složky

Tuto možnost použijte pro import všech obrázků ze složky najednou.

1. Klikněte na tlačítko **„Přidat složku“** v horní části panelu Prohlížeč souborů.
2. Přejděte do složky obsahující snímky z vaší relace snímání a vyberte ji.
3. Kliknutím na **„Vybrat složku“** importujte všechny podporované snímky z této složky.

***

## Popis tabulky prohlížeče souborů

Po importu se snímky zobrazí v tabulce s následujícími sloupci:

### Miniatura

* Malý náhled každého obrázku.
* Kliknutím na miniaturu zobrazíte celý obrázek v hlavní náhledové oblasti.

### Název souboru

* Původní název souboru z fotoaparátu.
* Zachovává konvenci pojmenování fotoaparátu (např. IMG\_0001.RAW).

### Časové razítko

* Datum a čas pořízení obrázku.
* Extrahováno z metadat EXIF obrázku.
* Používá se pro synchronizaci PPK a detekci kalibračního cíle

### Model fotoaparátu

* Automaticky detekovaná konfigurace fotoaparátu a filtru
* Příklady: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Používá se k použití správných profilů zpracování

### Sloupec cíle (zaškrtávací políčko)

* Zaškrtněte toto políčko u obrázků, které obsahují kalibrační cíle
* Výrazně urychluje detekci cíle během zpracování
* Podrobnosti najdete v části [Výběr cílových obrázků](choosing-target-images.md)

***

## Správa souborů v projektu

### Odstranění souborů

Chcete-li z projektu odstranit nepotřebné obrázky:

1. Vyberte jeden nebo více obrázků v tabulce Prohlížeč souborů.
2. Klikněte na tlačítko **„Odebrat vybrané“**.
3. Potvrďte odstranění (soubory nejsou odstraněny z disku, pouze z projektu).

### Třídění a filtrování

* **Třídění podle sloupce**: Kliknutím na záhlaví libovolného sloupce můžete obrázky seřadit.
* **Seřazení podle časového razítka**: užitečné pro uspořádání chronologických sekvencí snímků.
* **Filtr modelu fotoaparátu**: seskupte obrázky podle typu fotoaparátu, pokud používáte více fotoaparátů.

***

## Náhled obrázku

### Zobrazení celého obrázku

Kliknutím na libovolnou miniaturu obrázku v prohlížeči souborů jej zobrazíte v hlavní oblasti náhledu:

1. Obrázek se zobrazí ve středním panelu náhledu.
2. Pomocí ovládacích prvků zoomu můžete prohlížet detaily obrázku.
3. Mezi obrázky můžete přecházet pomocí kláves se šipkami

### Rychlá navigace

* **Předchozí obrázek**: Klikněte na šipku vlevo nebo stiskněte klávesu ←
* **Další obrázek**: Klikněte na šipku vpravo nebo stiskněte klávesu →
* **Přiblížení/oddálení**: Použijte kolečko myši nebo tlačítka pro přiblížení
* **Posun**: Po přiblížení klikněte na obrázek a přetáhněte jej

***

## Zpracování duplicitních souborů

Chloros automaticky detekuje a ignoruje duplicitní soubory:

* Soubory se stejnými názvy jsou přeskočeny.
* Zabraňuje náhodnému dvojímu zpracování.
* Při detekci duplicit se zobrazí varovná zpráva.

{% hint style=&quot;warning&quot; %}
**Důležité**: Před importem nepřejmenovávejte ani neměňte původní obrazové soubory. Chloros se pro správné zpracování spoléhá na původní názvy souborů a metadata.
{% endhint %}

***

## Smíšené soubory dat z kamer

Pokud váš projekt obsahuje obrázky z více kamer MAPIR:

1. Chloros automaticky detekuje každý model kamery.
2. Každý typ kamery je zpracován pomocí příslušného kalibračního profilu.
3. Prohlížeč souborů zobrazuje model kamery ve sloupci Model kamery.
4. Zpracování aplikuje správná nastavení pro každý typ kamery.

**Příklad scénáře**: Survey3W RGN + Survey3N OCN nastavení dvou kamer

***

## Osvědčené postupy

### Uspořádejte před importem

* Uchovávejte kalibrační cílové obrázky ve stejné složce jako snímky z průzkumu.
* Zachovejte původní strukturu složek z fotoaparátu/SD karty.
* Nemíchejte datové soubory z různých relací v jednom projektu.

### Pojmenování souborů

* Zachovejte původní názvy souborů fotoaparátu (IMG\_0001.RAW atd.).
* Před importem soubory nepřejmenovávejte.
* Původní názvy obsahují důležité metadata.

### Kalibrační cílové obrázky

* Vždy zahrňte 1–2 kalibrační cílové obrázky na relaci.
* Zachyťte cíle před a po relaci zachycení.
* Umístěte cíle do stejných světelných podmínek jako oblast zachycení.
* Označte cílové obrázky pomocí zaškrtávacího políčka Cíl, abyste urychlili zpracování.

***

## Časté problémy a řešení

### Obrázky se po importu nezobrazují

**Možné příčiny:**

* Nepodporovaný formát souboru (pouze RAW+JPG a JPG z fotoaparátů MAPIR)
* Obrázky pocházejí z fotoaparátů jiných než MAPIR (viz [Podporované fotoaparáty](../supported-cameras.md))
* Poškození souboru nebo neúplný přenos z karty SD

**Řešení**: Ověřte kompatibilitu formátu souboru a modelu fotoaparátu.

### Model fotoaparátu nebyl detekován

**Možné příčiny:**

* Upravená metadata EXIF
* Obrázky upravené v externím softwaru
* Neúplný přenos souboru

**Řešení**: Znovu importujte původní, neupravené soubory z fotoaparátu/SD karty.

### Chybějící časová razítka

**Možné příčiny:**

* Nesprávně nastavené hodiny fotoaparátu
* EXIF data odstraněna externím softwarem

**Řešení**: Ověřte, zda bylo nastavení času fotoaparátu během pořizování snímků správné.

***

## Další kroky

Po importu souborů:

1. **Zkontrolujte seznam souborů** – ujistěte se, že všechny snímky byly načteny správně.
2. **Zkontrolujte modely fotoaparátů** – ověřte správnou detekci fotoaparátů.
3. **Označte cílové snímky** – viz [Výběr cílových snímků](choosing-target-images.md)
4. **Upravte nastavení** – nakonfigurujte možnosti zpracování v [Nastavení projektu](adjusting-project-settings.md)
5. **Spusťte zpracování** – viz [Spuštění zpracování](starting-the-processing.md)

Podrobné informace o konfiguraci projektu najdete v části [Úprava nastavení projektu](adjusting-project-settings.md).
