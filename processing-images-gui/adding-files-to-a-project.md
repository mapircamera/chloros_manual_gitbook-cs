# Přidávání souborů do projektu

Jakmile v programu Chloros vytvoříte nebo otevřete projekt, je dalším krokem přidání multispektrálních snímků, abyste mohli zahájit jejich zpracování. Záložka Prohlížeč souborů<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> usnadňuje import snímků a správu datového souboru.

## Otevření prohlížeče souborů

1. Otevřete nebo vytvořte projekt v Chloros
2. Klikněte na ikonu **Prohlížeč souborů** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> v levém postranním panelu
3. Panel Prohlížeč souborů zobrazí seznam souborů vašeho projektu

{% hint style="info" %}
**Podporované typy souborů**: Chloros podporuje obrazové soubory RAW+JPG a JPG z kamer MAPIR, Survey3W a Survey3N. Doporučujeme používat pouze formát RAW+JPG.
{% endhint %}

***

## Přidávání obrázků do projektu

Existují dva hlavní způsoby, jak přidat obrázky do projektu:

### Metoda 1: Přidat soubory

Tuto možnost použijte k importu jednotlivých obrazových souborů nebo malého výběru souborů.

1. Klikněte na tlačítko **„Přidat soubory“** <img src="../.gitbook/assets/image.png" alt="" data-size="line"> v horní části panelu Prohlížeč souborů
2. Přejděte do složky obsahující vaše obrázky
3. Vyberte jeden nebo více obrazových souborů (podržte klávesu **Ctrl** pro výběr více souborů)
4. Klikněte na **„Otevřít“** pro import vybraných souborů

### Metoda 2: Přidat složku

Tuto možnost použijte pro import všech obrázků ze složky najednou.

1. Klikněte na tlačítko **„Přidat složku“** <img src="../.gitbook/assets/image (1).png" alt="" data-size="line"> v horní části panelu Prohlížeč souborů
2. Přejděte do složky obsahující snímky z dané relace a vyberte ji
3. Klikněte na **„Vybrat složku“** pro import všech podporovaných snímků z této složky***

## Popis tabulky v prohlížeči souborů

Po importu se snímky zobrazí v tabulce s následujícími sloupci:

### Název souboru

* Původní název souboru z fotoaparátu
* Zachovává konvenci pojmenování fotoaparátu (např. IMG\_0001.RAW)

### Časové razítko

* Datum a čas pořízení snímku
* Extrahováno z metadat EXIF snímku
* Používá se pro synchronizaci PPK a detekci kalibračních cílů

### Model fotoaparátu

* Automaticky detekovaná konfigurace fotoaparátu a filtru
* Příklady: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Používá se k použití správných profilů zpracování

### Sloupec cíle (zaškrtávací políčko)

* Zaškrtněte toto políčko u obrázků, které obsahují kalibrační cíle
* Výrazně urychluje detekci cílů během zpracování
* Podrobnosti najdete v části [Výběr obrázků cílů](choosing-target-images.md)

### Zobrazení metadat obrázku

Kliknutím na přepínací tlačítko v pravém horním rohu nad tabulkou se v oblasti mřížky obrázků zobrazí metadata vybraného obrázku.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Správa souborů v projektu

### Odstranění souborů

Chcete-li z projektu odstranit nepotřebné obrázky:

1. Vyberte jeden nebo více obrázků v tabulce Prohlížeče souborů
2. Klikněte na tlačítko **„Odstranit vybrané“** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line"> .
3. Potvrďte odstranění (soubory nejsou smazány z disku, pouze odstraněny z projektu).

### Třídění a filtrování

* **Třídění podle sloupce**: Kliknutím na libovolný záhlaví sloupce můžete obrázky seřadit.
* **Třídění podle časového razítka**: Užitečné pro uspořádání chronologických sekvencí snímků.
* **Filtr podle modelu fotoaparátu**: Pokud používáte více fotoaparátů, seskupte obrázky podle typu fotoaparátu.***

## Náhled obrázku

### Zobrazení celého obrázku

Klikněte na libovolnou miniaturu obrázku v prohlížeči souborů, aby se zobrazil v hlavní oblasti náhledu:

1. Obrázek se zobrazí ve středním panelu náhledu
2. Pomocí ovládacích prvků pro přiblížení si prohlédněte detaily obrázku
3. Mezi obrázky se pohybujte pomocí kláves se šipkami

### Rychlá navigace

* **Předchozí obrázek**: Klikněte na šipku vlevo nebo stiskněte klávesu ←
* **Další obrázek**: Klikněte na šipku vpravo nebo stiskněte klávesu →
* **Přiblížení/oddálení**: Použijte kolečko myši nebo tlačítka pro přiblížení
* **Posun**: Klikněte a táhněte na obrázku při přiblížení***

## Zpracování duplicitních souborů

Chloros automaticky detekuje a ignoruje duplicitní soubory:

* Soubory se stejnými názvy jsou přeskočeny
* Zabraňuje náhodnému dvojímu zpracování
* Při detekci duplicit se zobrazí varovná zpráva

{% hint style="warning" %}
**Důležité**: Před importem nepřejmenovávejte ani neupravujte původní obrazové soubory. Chloros se pro správné zpracování spoléhá na původní názvy souborů a metadata.
{% endhint %}

***

## Smíšené sady dat z kamer

Pokud váš projekt obsahuje snímky z více kamer MAPIR:

1. Chloros automaticky detekuje každý model kamery
2. Každý typ kamery je zpracován s příslušným kalibračním profilem
3. Prohlížeč souborů zobrazuje model kamery ve sloupci Model kamery
4. Zpracování použije správná nastavení pro každý typ kamery

**Příklad scénáře**: Survey3W RGN + Survey3N OCN konfigurace se dvěma kamerami***

## Osvědčené postupy

### Uspořádejte data před importem

* Uchovávejte kalibrační cílové snímky ve stejné složce jako snímky z průzkumu
* Zachovejte původní strukturu složek z fotoaparátu/SD karty
* Nemíchejte datové sady z různých relací v jednom projektu

### Pojmenování souborů

* Zachovejte původní názvy souborů z fotoaparátu (IMG\_0001.RAW atd.)
* Nepřejmenovávejte soubory před importem
* Původní názvy obsahují důležitá metadata

### Kalibrační snímky

* Vždy zahrňte 1–2 kalibrační snímky na jednu relaci
* Zachyťte cíle před a po snímací relaci
* Umístěte cíle do stejných světelných podmínek jako snímací oblast
* Označte kalibrační snímky zaškrtnutím políčka Target, abyste urychlili zpracování

***

## Časté problémy a řešení

### Snímky se po importu nezobrazují

**Možné příčiny:**

* Nepodporovaný formát souboru (pouze RAW+JPG a JPG z fotoaparátů MAPIR)
* Snímky pocházejí z fotoaparátů jiných než MAPIR (viz [Podporované fotoaparáty](../supported-cameras.md))
* Poškozený soubor nebo neúplný přenos z SD karty

**Řešení**: Ověřte kompatibilitu formátu souboru a modelu fotoaparátu

### Model fotoaparátu nebyl rozpoznán

**Možné příčiny:**

* Upravená metadata EXIF
* Obrázky upravené v externím softwaru
* Neúplný přenos souborů

**Řešení**: Znovu importujte původní, neupravené soubory z fotoaparátu/SD karty

### Chybějící časová razítka

**Možné příčiny:**

* Nesprávně nastavené hodiny fotoaparátu
* Data EXIF odstraněná externím softwarem

**Řešení**: Ověřte, zda bylo nastavení času fotoaparátu během pořizování snímků správné***

## Další kroky

Jakmile jsou soubory importovány:

1. **Zkontrolujte seznam souborů** – Ujistěte se, že se všechny snímky načtaly správně
2. **Zkontrolujte modely fotoaparátů** – ověřte správnou detekci fotoaparátu
3. **Označte cílové snímky** – viz [Výběr cílových snímků](choosing-target-images.md)
4. **Upravte nastavení** – nakonfigurujte možnosti zpracování v [Nastavení projektu](adjusting-project-settings.md)
5. **Spusťte zpracování** – viz [Spuštění zpracování](starting-the-processing.md)

Podrobné informace o konfiguraci projektu najdete v části [Úprava nastavení projektu](adjusting-project-settings.md).
