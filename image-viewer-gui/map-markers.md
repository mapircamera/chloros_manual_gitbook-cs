# Značky na mapě

Karta „Mapa“ zobrazuje vaše snímky na interaktivní 2D mapě podle jejich GPS souřadnic. Poskytuje vám geografický přehled o snímkovací relaci a představuje nejrychlejší způsob, jak ihned po importu vyřadit snímky, které nechcete zpracovávat.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Otevření záložky „Mapa“

1. Otevřete nebo vytvořte projekt v programu Chloros
2. Načtěte snímky obsahující metadata GPS
3. V levém postranním panelu klikněte na záložku **Mapa** <img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line">
4. Na mapě se u GPS polohy každého snímku zobrazí značka

{% hint style="info" %}
**GPS je nutné**: na mapě se zobrazují pouze snímky, které mají ve svých EXIF metadatech GPS souřadnice. Snímek bez souřadnic zůstává v projektu a zpracovává se normálně – pouze nemá značku.
{% endhint %}

***

## Úpravy obrázků na záložce Mapa

Záložka **Mapa**<img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> obsahuje stejná tlačítka pro přidání <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> a odebrání <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line"> souboru jako záložka [**Průzkumník souborů**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">. Zobrazuje stejný seznam projektových souborů s geografickými sloupci:

| Sloupec        | Obsah                                                           |
| ------------- | ------------------------------------------------------------------ |
| **Název**      | Název souboru tak, jak byl pořízen fotoaparátem                             |
| **Šířka**  | Desetinné stupně, šest desetinných míst                                |
| **Délka** | Desetinné stupně, šest desetinných míst                                |
| **Nadmořská výška** | Metry, jedno desetinné místo — `-`, pokud snímek neobsahuje údaje o nadmořské výšce |

{% hint style="info" %}
Kliknutím na záhlaví libovolného sloupce jej seřadíte; dalším kliknutím pořadí obrátíte.
{% endhint %}

{% hint style="warning" %}
**Nadmořská výška je výška nad hladinou moře, nikoli výška nad zemí.** Hodnota pochází ze značky EXIF snímku `GPSAltitude`, která se vztahuje k průměrné hladině moře. Nejedná se o letovou výšku nad terénem a Chloros z ní neodvozuje vzdálenost vzorku od země – nad polem ve výšce 300 m nad hladinou moře zaznamená dron ve výšce 100 m AGL zde přibližně 400 m. Tento sloupec použijte k odhalení výjimečných hodnot a k ověření konzistentní letové výšky, nikoli jako měření AGL.
{% endhint %}

***

## Značky snímků

Každý snímek s GPS údaji je označen značkou na svých souřadnicích.

### Zobrazení značek

* Značky se nacházejí na přesných souřadnicích zaznamenaných pro každý snímek
* Značky, které jsou blízko u sebe, se při oddálení mohou vizuálně překrývat — přibližte si zobrazení, abyste je od sebe oddělili
* Vybrané a zvýrazněné značky se zobrazují nad ostatními

### Náhled při najetí kurzorem

* **Najděte kurzorem** na libovolný značkovač a zobrazí se miniatura daného snímku s názvem souboru
* **Kliknutím**na značku vyberete snímek a**připnete** vyskakovací okno – zůstane otevřené, dokud nekliknete jinde. Pokud je vyskakovací okno připnuté, přejíždění myší nad jinými značkami jej nezavře
* Jedná se o rychlý způsob, jak najít konkrétní snímek ve velké relaci, aniž byste museli opustit mapu

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption><p>Karta „Mapa“ zobrazuje všechny snímky s geotagováním v projektu</p></figcaption></figure>### Superzoom

{% hint style="success" %}
**SUPERZOOM**: když dosáhnete maximálního přiblížení, pro které má poskytovatel dlaždic k dispozici snímky, další přiblížení dlaždice zvětší, místo aby se zastavilo, takže můžete oddělit značky, které leží téměř jedna na druhé.
{% endhint %}

* Super-zoom se aktivuje pouze tehdy, když jste **na** maximálním přiblížení poskytovatele pro danou lokalitu a dlaždice se již načtou. Pod touto úrovní se přiblížení chová normálně
* Rozsah je **1× až 32×** nad maximálním přiblížením samotného poskytovatele
* Indikátor v rohu zobrazuje aktuální úroveň superzoomu v procentech a tlačítko **×** vedle něj vás jedním kliknutím vrátí k normálnímu přiblížení
* Odzoomování vždy prochází přímo na mapu samotnou, takže se nikdy nemůžete zaseknout v superzoomu
* Při superzoomu se při přibližování a posouvání výsledný posun promítá zpět na mapu, takže oblast mimo střed, do které jste se přesunuli, pokračuje v načítání dlaždic, místo aby se vyprázdnila
* Značky jsou vykreslovány jako vektorové prvky, nikoli jako rastrové, takže zůstávají ostré na každé úrovni superzoomu

***

## Poskytovatelé mapových dlaždic

{% hint style="success" %}
**Automatický výběr**: Chloros vybere službu dlaždic, která nabízí nejlepší úroveň přiblížení pro místo, kde se nacházejí vaše obrázky. Kdykoli můžete přepnout ručně.
{% endhint %}

| Poskytovatel        | Poznámky                                                                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Google Maps** | Široké celosvětové pokrytí; podporuje všechny čtyři typy dlaždic                                                                                                            |
| **Esri ArcGIS**| Často letecké snímky s vyšším rozlišením v konkrétních regionech. Typ dlaždic**Terén** není pro Esri nabízen a jeho tlačítko je deaktivováno, pokud je vybrán Esri |***

## Typy mapových dlaždic

Vyberte typ mapové vrstvy pomocí tlačítek (zleva doprava):

![](&lt;../.gitbook/assets/image (14).png&gt;)

| Typ                 | Zobrazuje                                                                |
| -------------------- | -------------------------------------------------------------------- |
| **Terén**          | Stínování výškových rozdílů s mapovými detaily (silnice, popisky). Pouze Google       |
| **Mapa**              | Standardní dlaždice uliční mapy — varianta s nejnižšími nároky na šířku pásma              |
| **Satelit**        | Podrobné satelitní snímky, bez popisků — možnost s nejvyšší šířkou pásma |
| **Hybrid** (výchozí) | Satelitní snímky s nakreslenými silnicemi a popisky                |

Karta Mapa se otevře v režimu **Hybrid**. Vaše volba se promítne do změny poskytovatele, pokud to daný poskytovatel podporuje.***

## Navigace po mapě

* **Přiblížení**: kolečko myši nebo tlačítka pro přiblížení na mapě
* **Posun**: klikněte a táhněte
* **Celá obrazovka**: ovládací prvek pro celou obrazovku zvětší mapu na celou plochu okna***

## Případy použití

### Kontrola letové trasy

* Na první pohled si prohlédněte oblast pokrytí letu dronu
* Odhalte mezery, kde došlo k vynechání průletu
* Ověřte, zda let proběhl podle plánované trasy

### Kontrola pozemního snímkování

* Podívejte se, jak jsou rozmístěny pozemní snímky
* Vyhledejte rámečky kalibračních terčů vzhledem k oblasti snímkování
* Rozhodněte, kde jsou potřeba další snímky

### Kontrola kvality

* Najděte snímky pořízené na neočekávaných místech a odstraňte je před zpracováním
* Seřaďte podle nadmořské výšky, abyste odhalili snímek pořízený v nesprávné výšce nebo snímek, u kterého byla poloha GPS špatně zachycena
* Porovnejte polohy snímků s terénními poznámkami

***

## Řešení problémů

### Nezobrazují se žádné značky

**Možné příčiny**

* Snímky neobsahují metadata GPS
* Během pořizování snímků byla na fotoaparátu vypnutá funkce GPS
* Data EXIF byla před importem odstraněna jiným softwarem

**Co dělat**: ověřte, zda je na fotoaparátu zapnutá GPS, a znovu importujte původní soubory. Můžete zkontrolovat, zda konkrétní soubor obsahuje souřadnice, a to tak, že jej vyhledáte v tabulce souborů na záložce Mapa — snímek bez souřadnic tam nemá žádný řádek.

### Značky jsou na nesprávném místě

**Možné příčiny**: špatné zachycení signálu satelitů v okamžiku pořízení snímku nebo odchylka GPS během relace.**Co dělat**: jedná se o problém související s okamžikem pořízení snímku, který Chloros nemůže dodatečně opravit. Pro přesnou práci použijte pracovní postup s PPK/RTK GPS – viz nastavení**Použít PPK korekce** v [Nastavení projektu](../project-settings/project-settings.md).

### Mapa je prázdná nebo se přestaly načítat dlaždice

Poskytovatelé dlaždic jsou online služby. Pokud se dlaždice přestanou načítat, zkontrolujte síťové připojení zařízení a zkuste změnit poskytovatele. Pokud jste měli nastavené extrémní přiblížení, stiskněte tlačítko **×** pro resetování, abyste se vrátili na normální úroveň přiblížení, a nechte mapu znovu vyžádat dlaždice.***

## Související stránky

* [**Mřížka obrázků**](image-grid.md) — stejná sada obrázků jako miniatury
* [**Otevření obrázku na celou obrazovku**](opening-an-image-full-screen.md) — podrobné prohlížení jednoho obrázku
* [**Přidávání souborů do projektu**](../processing-images-gui/adding-files-to-a-project.md) — tlačítka pro přidání/odebrání souborů, která se na této záložce nacházejí
