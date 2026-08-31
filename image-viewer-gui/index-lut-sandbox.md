# Index/LUT Sandbox

Index/LUT Sandbox je interaktivní pracovní prostor v postranním panelu prohlížeče obrázků Chloros. Vyberete vzorec, přiřadíte k němu kanály vaší kamery, obarvíte jej přechodem a vyladíte rozsah hodnot — a obrázek se přitom v reálném čase aktualizuje. Od verze 1.2.0 můžete také **uložit to, co jste vytvořili**, a to buď pro jeden obrázek, nebo pro celý projekt, a to bez nutnosti opětovného zpracování.

## K čemu slouží Sandbox

| Index/LUT Sandbox (interaktivní)        | Zpracování projektu (dávkové)       |
| -------------------------------------- | -------------------------------- |
| Jeden snímek po druhém, okamžitá zpětná vazba  | Celý datový soubor v jednom průchodu     |
| Experimentální a iterativní             | Předkonfigurovaná nastavení          |
| Rendruje v reálném čase; ukládá pouze na požádání  | Vždy zapisuje výsledné soubory      |
| Ideální pro nalezení správných nastavení | Nejlepší volba, jakmile jsou nastavení finální |

{% hint style="success" %}
**Obvyklý pracovní postup**: laděte v Sandboxu, dokud vizualizace neodpovídá vašim představám, poté buď exportujte přímo ze Sandboxu, nebo zkopírujte stejná nastavení indexu a LUT do [Nastavení projektu](../project-settings/project-settings.md), aby je další běh zpracování zapracoval do každého obrázku.
{% endhint %}

***

## Otevření Sandboxu

1. Klikněte na snímek v mřížce — otevře se na celou obrazovku v záložce **Prohlížeč snímků** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Klikněte na ikonu **Prohlížeče obrázků** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">, aby se vysunul levý postranní panel, pokud ještě není otevřený
3. Z rozevíracího seznamu vrstev v pravém horním rohu vyberte vícepásmovou vrstvu — obvyklou volbou je **RAW (Reflectance)**, protože indexové hodnoty vypočítané na základě kalibrované odrazivosti jsou mezi obrázky srovnatelné

Postranní panel zobrazuje shora dolů:

* název snímku a model kamery
* tlačítko **Exportovat/Uložit snímky** — zobrazí se, jakmile zaškrtnete políčko Index nebo LUT
* zaškrtávací políčka **Index**a**LUT**
* panel konfigurace indexu
* panel **Hodnoty kurzoru** s odečtem, histogramem a ovládacím prvkem GSD

{% hint style="warning" %}
**Není k dispozici pro monochromatické kamery.** U jednopásmového snímku LATTICE M3M jsou obě zaškrtávací políčka deaktivována a zobrazuje se popisek _„Není k dispozici pro monochromatické (M3M) snímače“_ — vícepásmový index není pro jedno pásmo definován. Chcete-li vypočítat indexy z kamer M3M, zkombinujte dvě nebo více snímků do zarovnaného vícepásmového stohu a použijte indexovací engine LATTICE.
{% endhint %}

***

## Použití indexu

1. Zaškrtněte políčko **Index** v horní části postranního panelu
2. Z levého rozevíracího seznamu vyberte filtr vaší kamery (`RGN`, `OCN`, `NGB`, `RGB`, `RE`, `NIR`)
3. Z pravého rozevíracího seznamu vyberte indexový vzorec — k dispozici je 27 vestavěných vzorců a navíc jakékoli vlastní vzorce, které jste uložili
4. Vzorec se zobrazí jako matematický výraz níže, s prázdným kruhem v každém slotu pro pásmo. **Přetáhněte barevný kruh kanálu na pozici**, abyste jej přiřadili
5. Jakmile jsou přiřazeny všechny pozice, které vzorec používá, obrázek se aktualizuje a zobrazí hodnoty indexu
6. Přesuňte kurzor nad obrázek, abyste si přečetli hodnoty; panel **Hodnoty kurzoru** přidá řádek indexu s hodnotou pod kurzorem

Dvojitým kliknutím na přiřazený slot jej vymažete. Neúplný vzorec je normální stav během přetahování, nejedná se o chybu — obrázek se jednoduše neaktualizuje, dokud není vzorec kompletní.

Kruhy kanálů jsou barevně odlišeny: červená = Red, zelená = Green, modrá = Blue, oranžová = Orange, azurová = Cyan, fialová = NIR, purpurová = RE. Stejné barvy se používají pro tečky kanálů a křivky histogramu v panelu Hodnoty kurzoru.

### Příklad NDVI

```

Formula: (NIR - Red) / (NIR + Red)

For a Survey3W RGN camera:
  NIR = 850 nm band
  Red = 661 nm band

Result range:          -1.0 to +1.0
Typical vegetation:     0.4 to 0.9
Stressed vegetation:    0.2 to 0.4
Bare soil:              0.0 to 0.2
Water:                 -0.1 to 0.1
```

Kompletní přehled vzorců – všechny tři seznamy předvoleb a informace o tom, které názvy kde fungují – najdete v článku [Vzorce multispektrálních indexů](../project-settings/multispectral-index-formulas.md).

### S zaškrtnutou volbou Index, ale bez LUT

Obrázek je vykreslen v **stupních šedi**, roztažený mezi dvěma prahovými hodnotami. Je to záměrné: indexový obrázek představuje skalární data a stupně šedi jsou jejich věrným zobrazením. Chcete-li barvy, přidejte LUT.***

## Práce s LUT (vyhledávacími tabulkami)**Vyhledávací tabulka** přiřazuje indexové hodnoty k barvám: vstup NDVI 0,65, výstup konkrétní odstín zelené. Nemění data – mění způsob, jakým je interpretujete.

### Přidání LUT

1. Klikněte na tlačítko **„+ Přidat LUT“** v sekci „<img src="../.gitbook/assets/image (1) (1) (1).png" alt="" data-size="line">“ pod vzorcem
2. Vyberte barevný přechod
3. Nastavte minimální a maximální ořez
4. Vyberte režim oříznutí
5. Zaškrtněte políčko **LUT** v postranním panelu, aby se tabulka vykreslila

Zaškrtávací políčko LUT zůstává neaktivní, dokud není tabulka LUT skutečně nakonfigurována v indexu.

### Výběr barevného přechodu

Přejděte kurzorem myši nad **pruh přechodu**a otevřete seznam předvoleb — Chloros obsahuje**sedm** předvoleb přechodů:

| # | Přechod                            | Tvar                                                               |
| - | ----------------------------------- | ------------------------------------------------------------------- |
| 1 | Red → Žlutá → Green (**výchozí**)  | Rozcházející se — odpovídá běžné představě o vegetaci, zelená = zdravá |
| 2 | Fialová → Žlutá → Green             | Divergující, s výrazným spodním koncem                                  |
| 3 | Hnědá → Bílá → Blue                | Divergující kolem světlého středu                                   |
| 4 | Černá → Fialová → Růžová → Světle žlutá | Sekvenční, od tmavé k světlé                                           |
| 5 | Red → Žlutá → Blue                 | Odchylné kolem světlého středu                                   |
| 6 | Fialová → Blue → Green → Žlutá      | Postupné, od tmavého k světlému                                           |
| 7 | Orange → Bílá → Fialová             | Rozcházející se kolem světlého středu                                   |

**Rozcházející se**přechod umístí neutrální barvu do středu okna, což je vhodné, když středový bod má určitý význam (prahová hodnota, základní datum).**Sekvenční** přechod se mění monotónně od tmavého k světlému, což je vhodné pro veličiny, u nichž existuje pouze „více“ a „méně“.

Každá předvolba má sedm barevných zastávek. Klikněte na předvolbu a obrázek se okamžitě aktualizuje (pokud je zaškrtnuto políčko LUT).

### Úprava barevných zastávek

Pod pruhem přechodu se nachází řada barevných vzorků, jeden pro každou zastávku:

* **Změna barvy**: kliknutím na vzorek otevřete výběr barvy (barevný kruh, posuvníky RGB/HSV nebo hexadecimální kód, např. `#FF0000`)
* **Přidání přechodu**: klikněte na tlačítko**+** na konci řádku — přidá se bílý přechod
* **Odstranění přechodu**:**dvojité kliknutí** na vzorek
* **Uložení upraveného přechodu**: klikněte na ikonu uložení vedle lišty přechodu, čímž přidáte upravený přechod do seznamu předvoleb, abyste jej mohli znovu vybrat

Přechod, který jste nakonfigurovali pro index, se ukládá společně s tímto indexem v nastavení projektu, takže zůstane zachován i po zavření a opětovném otevření projektu.

**Menší počet zastávek**vytváří zřetelné zóny, které lze vnímat jako klasifikaci;**větší počet zastávek** vytváří plynulé, téměř fotografické přechody. Tři až pět zastávek se hodí pro prezentační snímky a klasifikační mapy; šest až deset pro obecnou analýzu; patnáct a více pro podrobnou inspekci a publikační grafy.

### Nastavení rozsahu hodnot

Ovládací prvek prahové hodnoty je **posuvník se dvěma úchyty**v rozsahu od −1 do +1, s editovatelným textovým polem na každém konci pro zadání přesných hodnot a tlačítkem**AUTO**.

* Přetáhněte libovolný úchyt nebo zadejte číslo do příslušného pole a stiskněte klávesu Enter
* **AUTO**nastaví rozsah na**

2. a 98. percentil** platných indexových hodnot obrázku — dobrý výchozí bod, který ignoruje odlehlé hodnoty. Chloros výsledek adaptivně zaokrouhluje na 4 desetinná místa pro velmi úzký rozsah, na 3 pro úzký rozsah a v ostatních případech na 2
* Jakékoli ruční nastavení má přednost před funkcí AUTO, dokud znovu nestisknete tlačítko AUTO

Příklad oken NDVI:

| Cíl                                    | Min  | Max |
| --------------------------------------- | ---- | --- |
| Zobrazit vše                         | −1,0 | 1,0 |
| Pouze vegetace, vyloučit půdu a vodu | 0,2  | 0,9 |
| Pouze zdravá vegetace                 | 0,5  | 0,9 |
| Zdůraznit stres                        | 0,2  | 0,5 |

Zúžením okna zvýšíte kontrast uvnitř oblasti zájmu a vše ostatní se dostane mimo rozsah — kde **režim oříznutí** rozhoduje o tom, co se s tím stane.***

## Režimy oříznutí

Pokud hodnota indexu pixelu leží mimo okno min/max, režim ořezu rozhoduje o tom, jak bude vykreslen.

| Název v rozevíracím seznamu                  | Uložená hodnota      | Pixely mimo rozsah jsou vykresleny jako                                                                                                |
| ------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Minimum a maximum** (výchozí) | `clip`            | Nejbližší koncová barva přechodu — hodnoty pod minimem přebírají první barvu, hodnoty nad maximem přebírají poslední |
| **Průhledné pozadí**      | `transparent`     | Plně průhledné (skutečná alfa)                                                                                                  |
| **Indexové pozadí**| `indexColor`      | Stupně šedi, roztažené přes**celý** indexový rozsah obrázku, takže struktura mimo rozsah je stále viditelná v šedé                |
| **Původní pozadí**         | `backgroundColor` | Samotný podkladový obrázek, takže barevná vrstva leží nad skutečnou scénou                                                |

| Režim                       | Nejvhodnější pro                               | Vzhled                                      |
| -------------------------- | -------------------------------------- | ----------------------------------------- |
| **Minimum a maximum**      | Zobrazení úplných dat, vědecká analýza | Každý pixel je zbarven                      |
| **Průhledné pozadí** | GIS překryvy, izolace hodnotového pásma   | Barva uvnitř okna, mimo okno nic |
| **Indexové pozadí**       | Zdůraznění při zachování kontextu dat    | Barva uvnitř, šedá vně               |
| **Původní pozadí**    | Zprávy a prezentace              | Barva uvnitř, fotografie vně         |

{% hint style="info" %}
**Pixely bez dat jsou vždy průhledné, v každém režimu.** Pixel, jehož index není konečný (dělení 0/0) nebo je přesně −1,0 či +1,0 (saturační sentinely, kdy jedno pásmo vykazuje hodnotu nula, zatímco druhé nikoli), je považován za pixel bez dat, nikoli za extrémní hodnotu. Tím se zabrání tomu, aby se ve vaší barevné škále objevily přeexponované světlé oblasti a zcela tmavé stíny, místo aby byly vykresleny jako nejextrémnější hodnoty v snímku. Stejné pravidlo určuje, které pixely se používají pro prahové hodnoty AUTO a indexový histogram, takže všechny tři hodnoty se shodují.
{% endhint %}

Průhlednost zůstává zachována, pokud je export uložen ve formátu PNG. Ve formátu JPG ji nelze zachovat.

***

## Čtení hodnot během ladění

Panel **Hodnoty kurzoru** pod konfiguračním panelem slouží jako měřicí nástroj pro Sandbox:

* Přesuňte kurzor nad obrázek a odečtěte zdrojové hodnoty jednotlivých kanálů spolu s hodnotou indexu v samostatném řádku
* Zapněte tlačítko **INDEX** nad histogramem, abyste viděli rozložení indexových hodnot v snímku, přičemž vaše dvě prahové hodnoty oříznutí jsou znázorněny jako oranžové přerušované čáry a hodnota kurzoru jako bílá čára — toto je nejrychlejší způsob, jak vybrat okno, které skutečně obsahuje vaše data
* Zapněte **CURSOR**, abyste viděli značkovací čáry u hodnot pod ukazatelem
* Přibližte obraz více než 60× (méně, pokud je nastavena velikost bloku GSD), abyste zvýraznili jednotlivé zobrazené pixely s plovoucí hodnotou

Praktický postup:

1. Poznamenejte si hodnoty nad zdravou vegetací, stresovanou vegetací, holou půdou a vodou
2. Podívejte se, kde se tyto shluky nacházejí na histogramu indexu
3. Nastavte min/max tak, aby ohraničovaly shluk, který vás zajímá
4. Vyberte režim oříznutí — _Original Background_ zachová viditelnost scény kolem něj

***

## Export z Sandboxu

Vše výše uvedené je živý náhled, dokud to neuložíte. Tlačítko **Export/Save Image(s)** v horní části postranního panelu otevře panel, který se vysune nad postranní panel (namísto zakrytí obrázku, takže stále vidíte, o čem rozhodujete).

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>### Možnosti

| Možnost                          | Účinek                                                                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Použít na aktuální obrázek**      | Uloží přesně zobrazený obrázek s těmito nastaveními                                                                                                |
| **Použít na všechny obrázky v projektu** | Znovu spustí identickou konfiguraci na každý obrázek v projektu. Obrázky bez pásem, která tento index potřebuje, jsou přeskočeny a nejsou považovány za chyby |
| **Gradientní lišta indexu/LUT**      | Při každém exportu se také uloží samostatný obrázek s legendou, na kterém je označen rozsah hodnot                                                                     |
| **Histogram indexu**             | Při každém exportu se také uloží samostatný obrázek histogramu, který zobrazuje minimální a maximální hodnoty dat a prahové hodnoty oříznutí                                               |

Pokud je hodnota **velikosti bloku GSD** na kartě snímku vyšší než 1, panel na to upozorní před potvrzením: export uloží to, co právě vidíte, včetně průměrování bloků. Pokud chcete plné rozlišení, nejprve nastavte ovládací prvek GSD zpět na 1.

### Kam se soubory ukládají

Každé kliknutí na **Export**přidělí**novou, nikdy znovu nepoužitou složku**:

```
<project folder>/Sandbox_Exports/<IndexName>_<Index|LUT>_<NNN>/
```

Příklady: `Sandbox_Exports/NDVI_LUT_001/`, poté `Sandbox_Exports/NDVI_LUT_002/` pro další běh. Číslování se odvozuje na základě prohledání obsahu disku, takže zůstává zachováno i po restartu a i v případě, že složky ručně smažete. Nikdy nedochází k přepsání — smyslem Sandboxu je právě porovnání aktuálního pokusu s tím předchozím.

Uvnitř složky, pro každý obrázek:

| Soubor                                                   | Obsah                                                   |
| ------------------------------------------------------ | ---------------------------------------------------------- |
| `<source name>_<IndexName>_<Index\|LUT>.png`           | Vykreslený obrázek, pixel po pixelu přesně to, co zobrazil prohlížeč |
| `<source name>_<IndexName>_<Index\|LUT>_legend.png`    | Doprovodný soubor s gradientovou lištou, pokud je požadován                     |
| `<source name>_<IndexName>_<Index\|LUT>_histogram.png` | Doprovodný soubor s histogramem indexů, pokud je požadován                  |

Oba doplňkové soubory jsou vždy zapsány v **plném rozlišení**, i když je hlavní obrázek průměrován po blocích: velikost bloku odpovídá rozlišení displeje a oba doplňkové soubory obsahují skutečné hodnoty indexu pro každý pixel. Zobrazují také více informací než verze na obrazovce — oba zaznamenávají okno roztažení _i_ skutečné minimální a maximální hodnoty dat, takže uložená legenda je čitelná i po několika měsících, aniž by bylo nutné projekt otevřít.

### Průběh a výsledky

Export celého projektu trvá několik minut, takže proces poskytuje zpětnou vazbu prostřednictvím živého kanálu s průběhem, místo aby blokoval systém:

* Ukazatel průběhu zobrazuje `current / total` a soubor, který se právě zapisuje
* Po dokončení panel uvádí, kolik obrázků bylo exportováno, kolik bylo přeskočeno a cestu k výstupní složce
* Přeskočené obrázky jsou uvedeny s důvodem (zobrazí se až pět, poté řádek „+N dalších“). Obvyklým důvodem je vrstva, která neobsahuje kanály potřebné pro tento index
* Pokud by **žádný** obrázek v projektu nemohl index využít, proces hlásí selhání, místo aby vám zanechal prázdnou složku

Současně běží pouze jeden export do sandboxu. Spuštění druhého exportu, zatímco první ještě probíhá, je odmítnuto jasnou zprávou, aby se zabránilo tomu, že by se dva procesy přetahovaly o stejný projektový soubor.

### Mřížka vybere daný běh

Každý dokončený běh se zobrazí jako samostatné tlačítko na panelu nástrojů [mřížky obrázků](image-grid.md) s označením `<IndexName> <Index|LUT> <NNN>`. Takto můžete porovnávat běhy: proveďte export dvakrát s různými gradienty nebo prahovými hodnotami a poté přepínejte mezi oběma tlačítky v mřížce.

***

## Vlastní vzorce indexů (Chloros+)

{% hint style="info" %}
**Kde je vytvořit**: v postranním panelu Sandboxu nebo v**Nastaveních projektu** před zpracováním. Oba se zapisují do stejného seznamu na úrovni projektu.
{% endhint %}

1. Otevřete kalkulátor vlastních vzorců z rozevíracího seznamu vzorců indexu (vyžaduje přihlášení s oprávněným předplatným Chloros+)
2. Zadejte vzorec pomocí **symbolů pásem a slotů** `x`, `y`, `z`, `a`, `b`, `c` — nejedná se o názvy pásem
3. Dostupné operátory: `+`, `-`, `*`, `/`, `^` a `()` pro seskupování
4. Dostupné funkce: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
5. Pojmenujte a uložte jej — objeví se v dolní části rozevíracího seznamu vzorců a jeho sloty přiřadíte přetažením kruhů kanálů, přesně jako u vestavěné předvolby

```

Modified NDVI with an offset:   (y-x)/(y+x+0.5)
Simple ratio:                   y/x
Three-band difference:          (y-x)/(y+x-z)
Squared ratio:                  (y/x)^2
```

{% hint style="warning" %}
**Vlastní vzorce jsou dostupné pouze v grafickém uživatelském rozhraní.** Možnost CLI/SDK `--indices` rozšiřuje 22 názvů vestavěných předvoleb a bez upozornění přeskočí vše ostatní, včetně vašich vlastních vzorců. Chcete-li vlastní vzorec použít hromadně, nakonfigurujte jej v nastavení projektu a spusťte zpracování, nebo použijte export „Použít na všechny obrázky projektu“ v Sandboxu.
{% endhint %}

***

## Řešení problémů

### „Tato vrstva nemá kanály, které tento index vyžaduje“

Vzorec čte pozici kanálu, kterou aktuální vrstva nemá — například index se třemi sloty u jedno- nebo dvoukanálového souboru. Přepněte na vícepásmovou vrstvu (reflektance nebo debayering) nebo vyberte index, který odpovídá filtru vaší kamery.

### „Nelze se spojit s backendem pro zpracování snímků“

Backend neodpovídá. Zkontrolujte záložku „Log“; pokud se backend restartuje, Sandbox se sám zotaví, jakmile bude opět k dispozici.

### Obrázek se nezměnil, když jsem přetáhl kruh

Vzorec ještě není kompletní. Neúplný vzorec je považován za normální stav během přetahování — nic se nerenderuje a nic není hlášeno jako chyba. Vyplňte všechna pole, která vzorec používá.

### Celý obrázek má jednu barvu

Vaše okno klipu je pravděpodobně daleko mimo rozsah dat. Stiskněte **AUTO**pro přichycení k 2. nebo 98. percentilu, nebo zapněte histogram**INDEX**, abyste viděli, kde se data skutečně nacházejí.

### Exportované barvy neodpovídají tomu, co jsem viděl

Měly by – exportní výstup záměrně zrcadlí živý náhled, včetně alfa kanálu v režimu oříznutí, a průměrování bloků se aplikuje _po_ zabarvení přesně tak, jak to dělá prohlížeč. Pokud se liší, zkontrolujte, zda se velikost bloku GSD mezi prohlížením a exportem nezměnila.

***

## Další kroky

* [**Obrázkové vrstvy**](image-layers.md) — na které vrstvě spustit index a co znamenají její hodnoty
* [**Otevření snímku na celou obrazovku**](opening-an-image-full-screen.md) — podrobnosti o zobrazení kurzoru, histogramu a ovládání GSD
* [**Vzorce multispektrálních indexů**](../project-settings/multispectral-index-formulas.md) — všechny předvolby na každém povrchu
* [**Nastavení projektu**](../project-settings/project-settings.md) — uložení nalezených nastavení do zpracovatelského cyklu
