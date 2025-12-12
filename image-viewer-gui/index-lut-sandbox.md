# Index/LUT Sandbox

Index/LUT Sandbox je interaktivní pracovní prostor v rámci prohlížeče obrázků Chloros, který vám umožňuje experimentovat s výpočty multispektrálních indexů a vizualizacemi barev v reálném čase. Tento výkonný nástroj vám pomůže testovat různé indexy, upřesňovat rozsahy hodnot a vytvářet vizualizace připravené k publikování, aniž byste museli znovu zpracovávat celý datový soubor.

## Co je Index/LUT Sandbox?

### Účel

Sandbox poskytuje:

* **Výpočet indexu v reálném čase** – okamžité použití libovolného vegetačního indexu
* **Interaktivní úprava LUT** – jemné doladění barevných přechodů a rozsahů
* **Optimalizace pracovního postupu** – určení nejvhodnějších nastavení před hromadným zpracováním

### Sandbox vs. zpracování projektu

**Index/LUT Sandbox (interaktivní):**

* Jeden obrázek najednou
* Okamžitá zpětná vazba
* Experimentální a iterativní
* Žádné trvalé změny souborů
* Ideální pro zkoumání a testování

**Zpracování projektu (hromadné):**

* Celý datový soubor najednou
* Předem nakonfigurovaná nastavení
* Trvalé výstupní soubory
* Časově náročné
* Nejlepší, když jsou nastavení finální

{% hint style=&quot;success&quot; %}
**Nejlepší pracovní postup**: Použijte Sandbox k experimentování a nalezení optimálních nastavení indexu a LUT, poté tato nastavení použijte během zpracování projektu pro celý datový soubor.
{% endhint %}

***

## Práce s Index/LUT Sandbox

### Porozumění předem vypočítaným indexům

V Chloros lze indexy použít během zpracování projektu. Chcete-li určit, která nastavení indexu a LUT chcete použít pro export, nejjednodušší je použít sandbox prohlížeče obrázků.

Sandbox vám umožňuje:

* **Použít nové indexy a barevné přechody (LUT)** k vizualizaci dat.
* **Interaktivně upravit nastavení vizualizace**.
* **Zobrazit** již vypočítané indexové obrázky.
* **Zkontrolovat** hodnoty pixelů ve všech úrovních přiblížení.

### Otevření sandboxu

Sandbox Index/LUT je přístupný v **prohlížeči obrázků** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Klikněte na obrázek v mřížce prohlížeče souborů, otevře se v záložce **Prohlížeč obrázků** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> .
2. Klikněte na kartu **Prohlížeč obrázků** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> , aby se otevřel levý vyskakovací postranní panel, pokud ještě není otevřený

### Výběr obrázku, na který se má aplikovat index/LUT

Chcete-li pracovat s indexem v prohlížeči obrázků <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. **Otevřete obrázek** z hlavní mřížky obrázků kliknutím na něj.
2. Otevře se karta **Prohlížeč obrázků** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> .
3. Klikněte na **rozevírací nabídku Vrstva** (vpravo nahoře v prohlížeči).
4. Z rozevírací nabídky vyberte vrstvu:
   * RAW (odrazivost)

### Použití indexu na obrázek

Jakmile je obrázek zobrazen na celé obrazovce a je otevřen postranní panel **Prohlížeč obrázků** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Zaškrtněte políčko Index v horní části postranní lišty.
2. Z levého rozevíracího seznamu vyberte filtr fotoaparátu.
3. Z pravého rozevíracího seznamu vyberte požadovaný indexový vzorec.
4. Přetáhněte barevné kruhy kanálu filtru na místa v indexovém vzorci níže.
5. Jakmile je vzorec platný, obrázek se aktualizuje a zobrazí hodnoty indexu.
6. Pohybem kurzoru myši můžete zobrazit hodnoty v místě kurzoru.
7. Přiblížením si můžete prohlédnout jednotlivé pixely a jejich přidružené hodnoty.

Každý index má specifický rozsah hodnot a význam:

#### NDVI Příklad

```
Formula: (NIR - Red) / (NIR + Red)

For Survey3W RGN camera:
NIR = 850nm band
Red = 661nm band

Result range: -1.0 to +1.0
Typical vegetation: 0.4 to 0.9
Stressed vegetation: 0.2 to 0.4
Bare soil: 0.0 to 0.2
Water: -0.1 to 0.1
```

Kompletní dokumentaci vzorců indexů najdete v části [Vzorce multispektrálních indexů](../project-settings/multispectral-index-formulas.md).

***

## Práce s LUT (vyhledávacími tabulkami)

### Co je LUT?

**Vyhledávací tabulka (LUT)** mapuje číselné hodnoty indexu na barvy pro vizualizaci:

* **Vstup**: Hodnota indexu pixelu (např. NDVI 0,65)
* **Výstup**: barva RGB (např. jasně zelená)
* **Účel**: Usnadnění viditelnosti a interpretace vzorů

**Šedá stupnice vs. barevná LUT:**

* Šedá stupnice: Vědecká a neutrální, zobrazuje surová data
* Barevná LUT: Intuitivní a působivá, zdůrazňuje vzory a rozdíly

{% hint style=&quot;success&quot; %}
**Vizuální síla**: Použití barevné LUT na indexový obrázek ve stupních šedi výrazně usnadňuje identifikaci vzorů, anomálií a oblastí zájmu na první pohled.
{% endhint %}

### Použití LUT na indexový obrázek

Jakmile máte indexový obrázek zobrazující

1. Klikněte na <img src="../.gitbook/assets/image.png" alt="" data-size="line"> „+Přidat LUT“
2. Vyberte barevný přechod
3. Upravte minimální/maximální koncové body oříznutí
4. Upravte režim oříznutí
5. Zaškrtněte políčko Index v postranní liště **Prohlížeče obrázků** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> v postranní liště, aby se LUT použila.

### Výběr barevného přechodu

**Výběr přechodu:**

1. V panelu LUT najděte **barevnou lištu přechodu**.
2. Najeďte na ni myší, aby se zobrazily dostupné předvolby přechodu.
3. Vyberte požadovaný přechod.
4. Po zaškrtnutí políčka Index se obrázek **okamžitě aktualizuje** novými barvami

{% hint style=&quot;success&quot; %}
**Osvědčená praxe**: U vegetačních indexů, jako je NDVI, je nejintuitivnější přechod Red-Yellow-Green, protože odpovídá přirozeným asociacím barev (zelená = zdravá, žlutá = střední, červená = stresovaná).
{% endhint %}

### Úprava barevných tříd

**Ovládací prvek Třídy** určuje, kolik samostatných barevných kroků se objeví ve vašem přechodu:

**Možnosti počtu tříd:**

* **2–5 tříd**: Velmi široké kategorie, odlišné zóny
* **6–10 tříd**: Vyvážené, vhodné pro klasifikaci
* **11–20 tříd**: Plynulé přechody, souvislý vzhled
* **20+ tříd**: Téměř plynulý, maximální hladkost

**Jak upravit:**

1. V panelu LUT najděte **barevné vzorky pod pruhem přechodu**.
2. Upravte počet tříd přidáním pomocí tlačítka +.
3. Počet tříd odstraníte dvojitým kliknutím na barevný vzorek.
4. Přechod se **v reálném čase** aktualizuje na obrázku.

**Vliv na vizualizaci:**

* **Méně tříd** (3–5): Vytváří odlišné zóny, zjednodušenou klasifikaci, snazší rozlišení kategorií.
* **Střední počet tříd** (6–10): Vyvážený přístup, vhodný pro většinu aplikací.
* **Více tříd** (15–20): Plynulé přechody, detailní variace, fotografický vzhled.

**Kdy použít:**

* **Málo tříd (3–5)**: Prezentační snímky, klasifikační mapy, jednoduché zprávy
* **Střední počet tříd (6–10)**: Obecná analýza, vyvážené detaily, standardní zprávy
* **Mnoho tříd (15–20)**: Vědecká analýza, podrobná kontrola, výstupy v publikační kvalitě

### Jemné doladění hodnotových rozsahů

**Ovládací prvky hodnotového rozsahu** určují, které hodnoty indexu se přiřadí k kterým barvám ve vašem přechodu:

**Ovládací prvky rozsahu v panelu LUT:**

* **Minimální hodnota**: Dolní hranice barevné škály
* **Maximální hodnota**: Horní hranice barevné škály
* **Střední hodnoty**: Automaticky rozdělené mezi minimální a maximální hodnotu (na základě počtu tříd)

#### Úprava minimálních/maximálních hodnot

**Úprava rozsahů hodnot:**

1. V panelu LUT vyhledejte vstupní pole **Minimální hodnota** a **Maximální hodnota**.
2. Klikněte na pole **Minimální hodnota**.
3. Zadejte požadovanou minimální hodnotu (např. `0.2`).
4. Stiskněte klávesu **Enter** nebo klikněte mimo pole.
5. Opakujte postup pro pole **Maximální hodnota** (např. `0.9`).
6. Vizualizace se **okamžitě aktualizuje**.

{% hint style=&quot;info&quot; %}
**Automatické škálování**: Při prvním použití LUT nastaví Chloros automaticky minimální/maximální hodnotu na skutečný rozsah dat v obrázku. Tento rozsah pak můžete zúžit, abyste se zaměřili na konkrétní rozsahy hodnot, které vás zajímají.
{% endhint %}

**Příklad úprav rozsahu NDVI:**

* **Celý rozsah**: `-1.0` až `1.0` (zobrazit všechny možné hodnoty)
* **Zaměřeno na vegetaci**: `0.2` až `0.9` (vyloučit holou půdu a vodu)
* **Pouze zdravá vegetace**: `0.5` až `0.9` (zvýraznit pouze vitální rostliny)
* **Detekce stresu**: `0.2` až `0.5` (zdůraznění problémových oblastí)
* **Vlastní rozsah**: Upravte na základě pozorovaných hodnot pixelů

**Proč upravovat rozsahy?**

* **Zvýšení kontrastu** ve vaší oblasti zájmu
* **Vyloučení irelevantních hodnot** (např. vodní plochy, holá půda)
* **Standardizace vizualizace** napříč více obrázky nebo daty
* **Zdůraznění jemných rozdílů** v úzkém rozsahu hodnot

### Oříznutí hodnot mimo rozsah

Pokud hodnoty pixelů přesahují vámi definovaný minimální/maximální rozsah, můžete pomocí **režimů oříznutí** ovládat způsob jejich zobrazení.

#### **Dostupné možnosti režimu ořezávání:**

#### 1. Minimum a maximum

* Pixely **pod minimem** → zobrazení pomocí **první barvy** v přechodu (např. červená)
* Pixely **nad maximem** → zobrazení pomocí **poslední barvy** v přechodu (např. zelená)
* **Případ použití**: Zdůraznění extrémů, zobrazení celého rozsahu dat s nasycenými barvami na mezích
* **Příklad**: Hodnoty NDVI pod 0,2 se zobrazí červeně, hodnoty nad 0,9 se zobrazí zeleně

#### 2. Průhledné pozadí

* Pixely **mimo rozsah** se stanou **zcela průhlednými**
* Pouze pixely **v rozsahu** zobrazují barevný přechod
* **Případ použití**: překrytí GIS, izolace konkrétních rozsahů hodnot, zvýraznění pouze oblastí zájmu
* **Příklad**: Zobrazit pouze NDVI 0,4–0,7 v barvě, vše ostatní průhledné

{% hint style=&quot;warning&quot; %}
**Omezení průhlednosti**: Průhledné pixely se v prohlížeči zobrazí jako barva pozadí. Při exportu během zpracování je průhlednost zachována ve formátu PNG, ale ne ve formátu JPG.
{% endhint %}

#### 3. Pozadí indexu

* Pixely **mimo rozsah** se zobrazují v **odstínech šedé** (zobrazují surové hodnoty indexu)
* Pixely **v rozsahu** zobrazují **barevný přechod**
* **Případ použití**: Jemné zvýraznění, zachování kontextu při zdůraznění oblastí zájmu
* **Příklad**: Barevně zvýrazněná stresovaná vegetace (NDVI 0,3–0,5) při zobrazení zdravých oblastí šedou barvou

#### 4. Původní pozadí

* Pixely **mimo rozsah** zobrazují **původní multispektrální obraz**
* Pixely **v rozsahu** zobrazují **barevný přechod**
* **Případ použití**: Nejintenzivnější – kombinuje přirozený kontext obrazu s analytickým barevným překryvem
* **Příklad**: Zobrazení skutečného vzhledu pole/plodiny s barevně označenými stresovými oblastmi

### Výběr správného režimu ořezávání

| Režim ořezávání              | Nejvhodnější pro                                   | Styl vizualizace          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimum a maximum**    | Zobrazení všech dat, vědecká analýza     | Všechny pixely barevné           |
| **Průhledné pozadí** | Překryvy GIS, izolace konkrétních rozsahů    | Barva v rozsahu, mimo rozsah prázdné |
| **Indexové pozadí**       | Jemné zdůraznění, zachování kontextu dat  | Barva v rozsahu, mimo rozsah šedé  |
| **Původní pozadí**    | Zprávy, prezentace, intuitivní analýza | Barva v rozsahu, mimo rozsah fotografie |

### Vytváření vlastních barev LUT

Pro úplnou kontrolu nad vizualizací můžete vytvořit **vlastní barevné přechody** úpravou jednotlivých barevných zastavení.

**Vytvoření vlastního přechodu:**

1. V panelu LUT vyhledejte **lištu náhledu přechodu**.
2. Pod přechodem vyhledejte **barevné vzorníky**.
3. **Kliknutím na barevné zastavení** jej vyberte.
4. Otevře se **výběr barev**.
5. Vyberte novou barvu pomocí:
   * **Barevného kolečka**: Vizuální výběr barvy.
   * **Posuvníků RGB/HSV**: Přesné ovládání barev.
   * **Zadání hexadecimálního kódu**: Přesná specifikace barvy (např. `#FF0000` pro červenou).
6. Kliknutím mimo vzorník barev **aplikujte novou barvu**.
7. Přechod se **okamžitě aktualizuje** na obrázku.

**Přidávání nebo odebírání barevných zastavení:**

* **Přidání přechodu**: Kliknutím na ikonu + přidáte nový vzorek na konec.
* **Odebrání přechodu**: Dvojitým kliknutím na barevný čtverec odeberete vzorek.

**Strategie přizpůsobení:**

* **Inverzní přechod**: Obrátíte pořadí barev, abyste změnili význam (např. zelená = nízká, červená = vysoká).
* **Barvy značky**: Přizpůsobíte paletu barev vaší organizace pro zprávy.
* **Vhodné pro barvoslepé**: Použijte kombinace oranžová-modrá nebo fialová-žlutá
* **Optimalizace tisku**: Vyberte barvy, které fungují jak při barevném, tak při černobílém tisku
* **Více prahových hodnot**: Použijte odlišné barvy pro konkrétní prahové hodnoty klasifikace

{% hint style=&quot;info&quot; %}
**Ukládání vlastních přechodů**: Vlastní přechody lze uložit a znovu použít. Kliknutím na ikonu uložení v panelu LUT můžete uložit vlastní barevná schémata pro budoucí použití.
{% endhint %}

***

## Interaktivní pracovní postup

### Aktualizace v reálném čase

Všechny úpravy LUT v sandboxu aktualizují obrázek **okamžitě a interaktivně**:

* **Přepnutí vrstvy** → Obrázek se okamžitě změní
* **Výběr přechodu** → Barvy se okamžitě aktualizují
* **Úprava rozsahu hodnot** → Kontrast se mění v reálném čase
* **Změna tříd** → Hladkost přechodu se okamžitě aktualizuje
* **Úprava ořezu** → Zobrazení pozadí se okamžitě změní
* **Upravit barvy** → Vlastní přechod se použije okamžitě

**Není potřeba tlačítko „Použít“** – všechny změny jsou živé a interaktivní!

{% hint style=&quot;success&quot; %}
**Živá zpětná vazba**: Okamžitá vizuální zpětná vazba vám umožňuje rychle experimentovat s různými nastaveními, dokud nenajdete optimální vizualizaci pro vaše analytické potřeby.
{% endhint %}

### Iterativní vylepšovací pracovní postup

**Typický pracovní postup optimalizace LUT:**

1. **Vyberte indexovou vrstvu** (např. RAW (odrazivost))
2. **Použijte index** – vyberte filtr fotoaparátu a vzorec indexu, přetáhněte barevné kruhy na příslušné místo ve vzorci indexu
3. **Použijte gradient LUT** – začněte s předvolbou Red-Yellow-Green
4. **Zkontrolujte hodnoty pixelů** – pohybujte kurzorem a zaznamenejte rozsahy hodnot
5. **Upravte min/max** – zúžte rozsah, abyste se zaměřili na vegetaci (např. 0,2 až 0,9)
6. **Vyberte oříznutí** – zkuste „Původní pozadí“ pro kontext
7. **Vylepšete barvy** – v případě potřeby přizpůsobte přechod pro konkrétní zdůraznění
8. **Dokončete nastavení** – zaznamenejte nastavení a zkopírujte je do nastavení projektu pro export

### Kontrola hodnot pixelů

Porozumění skutečným hodnotám pixelů je zásadní pro nastavení efektivních rozsahů LUT:

**Jak zkontrolovat hodnoty:**

1. Hodnoty pixelů se zobrazí, když je u obrázku zaškrtnuto políčko Index nebo políčka Index a LUT.
2. **Přesuňte kurzor** na různé oblasti obrázku.
3. **Sledujte hodnoty pixelů** zobrazené v legendě při najetí kurzorem.
4. Přibližte si obrázek, abyste viděli jednotlivé pixely zvýrazněné plovoucí hodnotou.
5. **Poznamenejte si** rozsahy hodnot pro různé prvky:
   * **Zdravá vegetace**: např. NDVI 0,55–0,85
   * **Stresovaná vegetace**: např. NDVI 0,30–0,50
   * **Holá půda**: např. NDVI 0,05–0,25
   * **Voda** (pokud je přítomna): např. NDVI -0,05 až 0,10

**Použití hodnot pixelů k nastavení rozsahů LUT:**

Po zkontrolování hodnot pixelů upravte odpovídajícím způsobem minimální/maximální hodnoty LUT:

**Příklad scénáře:**

* **Pozorování**: Hodnoty půdy = 0,05–0,25, stresované = 0,25–0,50, zdravé = 0,50–0,85
* **Cíl**: Vizualizovat pouze zdraví rostlin (vynechat půdu)
* **Nastavení LUT**: Min = `0.25`, Max = `0.85`
* **Ořez**: „Původní pozadí“ pro zobrazení půdy v přirozené barvě
* **Výsledek**: Barevný přechod se vztahuje pouze na vegetaci, půda se zobrazuje jako původní obraz

{% hint style=&quot;info&quot; %}
**Dynamický rozsah**: Různé plodiny, roční období a fáze růstu budou mít různé rozsahy hodnot. Před nastavením rozsahů LUT vždy zkontrolujte hodnoty pixelů ve vašem konkrétním datovém souboru.
{% endhint %}

***

## Vlastní indexy (Chloros+)

### Vytváření vlastních vzorců indexů

{% hint style=&quot;info&quot; %}
**Kde vytvořit**: Vlastní indexy lze nakonfigurovat v **Nastavení projektu** před zpracováním, stejně jako v postranním panelu sandboxu prohlížeče obrázků.
{% endhint %}

**Vytvoření vlastního indexu:**

1. **Otevřete nastavení projektu** (před zpracováním) nebo postranní panel sandboxu prohlížeče obrázků.
2. Přejděte do **rozevíracího seznamu vzorců indexu**.
3. Vyhledejte možnost **„Vlastní“** (musíte být přihlášeni s licencí Chloros+).
4. **Definujte svůj vzorec** pomocí proměnných pásma:
   * Názvy pásem: `NIR`, `Red`, `Green`, `Blue`, `RedEdge` atd.
   * Operátory: `+`, `-`, `*`, `/`, `^` (exponent)
   * Funkce: `sqrt()`, `abs()` atd. (pokud jsou podporovány)
   * Závorky: `()` pro pořadí operací
5. **Pojmenujte svůj index** (např. „MyIndex“ nebo „CustomNDVI“)
6. **Uložte konfiguraci**

**Příklady vlastních vzorců:**

```
Modified NDVI with offset:
(NIR - Red) / (NIR + Red + 0.5)

Simple ratio:
NIR / Red

Complex multi-band:
(NIR - Red) / (NIR + Red - Blue)

Exponential index:
(NIR / Red) ^ 2
```

{% hint style=&quot;warning&quot; %}
**Ověření vzorce**: Ujistěte se, že váš vzorec používá pásma dostupná ve vaší kameře. Například RedEdge je k dispozici pouze u kamer s filtrem RedEdge.
{% endhint %}

***

## Další kroky

Nyní, když rozumíte Index/LUT Sandbox:

* **Použít při zpracování**: Použijte nalezená nastavení v [Nastavení projektu](../project-settings/project-settings.md)
* **Hromadné zpracování**: Použijte optimalizované indexy na celé datové sady
* **Další informace**: Přečtěte si [Vzorec multispektrálního indexu](../project-settings/multispectral-index-formulas.md)

Související dokumentace:

* [**Vrstvy obrazu**](image-layers.md) – Správa vrstev a vizualizace
* [**Otevření obrázku na celou obrazovku**](opening-an-image-full-screen.md) – Základy prohlížeče obrázků
* [**Zpracování obrázků (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Kompletní pracovní postup zpracování
