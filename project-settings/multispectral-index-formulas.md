---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Vzorce multispektrálních indexů

Níže uvedené vzorce indexů využívají kombinaci rozsahů průměrné propustnosti filtrů Survey3:

<table><thead><tr><th align="center">Barva filtru Survey3</th><th width="196.199951171875" align="center">Název filtru Survey3</th><th width="159.800048828125" align="center">Rozsah propustnosti (FWHM)</th><th align="center">Průměrná propustnost</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re-RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>

Při použití těchto vzorců může název končit na „\_1“ nebo „\_2“, což odpovídá tomu, který NIR filtr (NIR1 nebo NIR2) byl použit.

U kamer LATTICE M3C (Bayerův trojpásmový filtr) používá stejný indexovací modul pásma filtru M3C:

| Filtr M3C | Pásmo 1 (střed/FWHM) | Pásmo 2 (střed/FWHM) | Pásmo 3 (střed/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orangeová 615 nm / 21 nm | Cyanová 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

Kamery LATTICE M3M jsou jednopásmové (jedna úzkopásmová filtrační vložka na kameru), proto se pro samostatný snímek z kamery M3M nepočítají vícepásmové indexy. Chcete-li vypočítat indexy pomocí M3M, zkombinujte dvě nebo více kamer do zarovnaného vícepásmového souboru a použijte indexovací engine LATTICE (`chloros-cli lattice index` nebo živou kalkulačku indexů v grafickém uživatelském rozhraní).

***

## Kde funguje který název indexu

Systém Chloros má **tři** povrchy indexů a jejich přednastavené seznamy nejsou identické. V této části si ověřte, zda daný název bude fungovat tam, kde jej plánujete použít.

| Kde se nacházíte | Který seznam platí | Počet |
| --- | --- | --- |
| Nastavení projektu → Index → Přidat index (GUI) | Povrch 1 | 27 |
| Prohlížeč obrázků [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) (GUI) | Povrch 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | Surface 2 | 22 |
| SDK `process_folder(indices=[...])` | Surface 2 | 22 |
| `chloros-cli lattice index --preset` | Surface 3 | 22 (jiné 22) |
| Záložka „Kamery“ – kalkulátor živého indexu | Surface 3 | 22 (jiné 22) |

Surface 1 a 2 pracují s **jedním snímkem najednou z jedné kamery**, přičemž využívají symbolové sloty `x`/`y`/`z`(/`a`) vázané na filtrační kanály dané kamery. Povrch 3 pracuje s**zarovnaným vícepásmovým stackem** — několika kamerami LATTICE, které jsou společnou registrací sloučeny do jednoho krychlového souboru — a odkazuje na kanály pomocí názvů psaných malými písmeny.

### 1. Nastavení projektu v grafickém uživatelském rozhraní / rozevírací seznam v sandboxu prohlížeče snímků — 27 vzorců

Rozevírací seznam je uvádí v tomto pořadí (jedná se o pořadí vložení, nikoli abecední pořadí):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

V grafickém uživatelském rozhraní přetáhnete filtrační kanály vaší kamery na pozice pásem ve vzorci, takže jakýkoli vzorec lze použít s jakýmkoli přiřazením pásem, které vaše kamera podporuje. Vlastní vzorce, které jste uložili, jsou připojeny pod tímto seznamem.

**Pět vzorců dostupných pouze v grafickém rozhraní** — tedy těch, které seznam CLI/SDK `--indices` nepřijímá — je implementováno takto:

| Předvolba dostupná pouze v grafickém rozhraní | Vzorec (jak je implementován) | Sloty |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a (čtyři sloty) |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

Zamýšlené přiřazení pro každý z nich je uvedeno v samostatné části dále na této stránce (například GARI očekává x=Green, y=NIR, z=Blue, a=Red). GARI je jediný vzorec v Chloros, který používá čtvrtý slot.

### 2. CLI / SDK `--indices` rozbalení názvů — 22 předvoleb

Možnost `chloros-cli process --indices` (a parametr SDK `indices`) přijímá tyto názvy předvoleb:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**Neznámé názvy indexů jsou bez upozornění přeskočeny.** Název mimo tento seznam (včetně pěti vzorců určených pouze pro grafické uživatelské rozhraní `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` a jakékoli vlastní vzorce, které jste uložili v grafickém rozhraní) je vynechán pouze s upozorněním v protokolu – běh pokračuje bez tohoto indexu a samotný běh je i tak vyhodnocen jako úspěšný. Upozornění se zobrazí takto:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

Názvy se porovnávají bez rozlišování velkých a malých písmen po odstranění mezer, takže `ndvi`, `NDVI` a ` NDVI ` představují stejnou předvolbu. Předvolba je také přeskočena, pokud vyžaduje pásmo, které filtr vaší kamery neposkytuje.
{% endhint %}

Přesné vzorce tak, jak jsou implementovány (symboly `x`/`y`/`z` představují pozice pásem; výchozí přiřazení je uvedeno pro každou předvolbu):

| Předvolba | Vzorec (jak je implementován) | Výchozí filtr | Sloty (x, y, z) |
| --- | --- | --- | --- |
| NDVI | `(y-x)/(y+x)` | RGN | Red, NIR |
| GNDVI | `(y-x)/(y+x)` | RGN | Green, NIR |
| NDRE | `(y-x)/(y+x)` | RE | RE, NIR |
| OSAVI | `(y-x)/(y+x+0.16)` | RGN | Red, NIR |
| SAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Red, NIR |
| MSAVI2 | `(2*y+1-sqrt((2*y+1)*(2*y+1)-8*(y-x)))/2` | RGN | Red, NIR |
| EVI | `2.5*(y-x)/(y+6*x-7.5*z+1)` | RGN | Red, NIR, Blue |
| MSR | `((y/x)-1)/(sqrt(y/x)+1)` | RGN | Red, NIR |
| TDVI | `1.5*(y-x)/sqrt(y*y+x+0.5)` | RGN | Red, NIR |
| LAI | `3.618*(2.5*(y-x)/(y+6*x-7.5*z+1))-0.118` | RGN | Red, NIR, Blue |
| GCI | `(y/x)-1` | RGN | Green, NIR |
| GRVI | `y/x` | RGN | Green, NIR |
| GSAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Green, NIR |
| GOSAVI | `(y-x)/(y+x+0.16)` | RGN | Green, NIR |
| NLI | `((y*y)-x)/((y*y)+x)` | RGN | Red, NIR |
| MNLI | `((y*y-x)*(1+0.5))/((y*y)+x+0.5)` | RGN | Red, NIR |
| RDVI | `(y-x)/sqrt(y+x)` | RGN | Red, NIR |
| WDRVI | `(0.2*y-x)/(0.2*y+x)` | RGN | Red, NIR |
| CVI | `(z/y)*(x/y)` | RGN | Red, Green, NIR |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | NGB | NIR, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | 

Když zadáte pouhý název, jako je například `NDVI`, musí program Chloros určit, který kanál z kterého souboru každý symbol čte. Používá k tomu tuto tabulku, která přiřazuje kód filtru k pozici každého kanálu v poli:

| Kód filtru | Kanál → index pole |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red` je akceptován jako alias pro Orange, rovněž 0) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

**Výchozí filtr** předvolby (sloupec „Výchozí filtr“ výše) se použije, pokud projekt obsahuje snímky s tímto filtrem. Pokud tomu tak není, Chloros prohledá filtry skutečně přítomné v projektu v pořadí `RGN, OCN, NGB, RGB, RE, NIR` a vybere první z nich, který dokáže poskytnout všechny kanály, které předvolba potřebuje. Pokud žádný z nich tuto podmínku nesplňuje, předvolba se pro daný běh vynechá. Proto `NDVI` požadovaný u datového souboru obsahujícího pouze OCN stále poskytuje smysluplný výsledek — váže se na pozice Orange a NIR v OCN.

Řetězce modelu LATTICE M3C nesou filtr s předponou `F` (`LATT-M3C-L41-FRGN`), avšak předpona je při čtení kódu filtru z obrazu odstraněna, takže kamera FRGN rozlišuje pomocí řádku `RGN` výše a nevyžaduje žádné zvláštní zacházení.

### 3. Indexovací engine LATTICE (`lattice index --preset`, živý Index Calculator) — 22 předvoleb

Modul LATTICE pracuje se zarovnanými vícepásmovými sériemi snímků (živá pole nebo exportované vícepásmové soubory TIFF) a používá názvy kanálů psané malými písmeny (`red`, `green`, `blue`, `red_edge`, `nir`). Jeho seznam předvoleb se liší od dvou výše uvedených:

| Předvolba | Vzorec | Kanály |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | červená, NIR |
| GNDVI | `(nir - green) / (nir + green)` | zelená, NIR |
| BNDVI | `(nir - blue) / (nir + blue)` | modrá, NIR |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | červená_hranice, NIR |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | modrá, zelená, NIR |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | červená, NIR |
| OSAVI | `(nir - red) / (nir + red + 0.16)` | červená, NIR |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | červená, NIR |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | modrá, červená, NIR |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | červená, NIR |
| CVI | `nir * red / green**2` | červená, zelená, NIR |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | červená, NIR |
| TDVI | `1.5 * (nir - red) / sqrt(nir**2 + red + 0.5)` | červená, NIR |
| LAI | `3.618 * (2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)) - 0.118` | modrá, červená, NIR |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | červená, zelená, modrá |
| NGRDI | `(green - red) / (green + red)` | červená, zelená |
| VARI | `(green - red) / (green + red - blue)` | červená, zelená, modrá |
| TGI | `green - 0.39*red - 0.61*blue` | červená, zelená, modrá |
| EXG | `2*green - red - blue` | červená, zelená, modrá |
| CIRE | `(nir / red_edge) - 1` | červená_okraj, nir |
| CIGREEN | `(nir / green) - 1` | zelená, nir |
| NDWI | `(green - nir) / (green + nir)` | zelená, nir |

Spusťte příkaz `chloros-cli lattice index --list-presets` pro vytištění této tabulky z nainstalované verze, a příkaz `--list-gradients` pro dostupné barevné přechody. U symbolů kanálů se rozlišují velká a malá písmena a musí se shodovat s názvy předvoleb psanými malými písmeny (např. `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI – index vegetace na základě chlorofylu

Tento index je citlivý na obsah chlorofylu v listech v širokém rozsahu hustot korun stromů. Využívá pásma NIR, červené a zelené.

$$
CVI = NIR * {Red \over Green^{2}}
$$

V grafickém uživatelském rozhraní (GUI) a v seznamu předvoleb CLI/SDK se jedná o symbolový vzorec `(z/y)*(x/y)`, kde x = Red, y = Green, z = NIR (pásma kamery typu RGN); předvolba indexového modulu LATTICE je `nir * red / green**2`. Každý povrch vypočítává stejný index.

_Odkaz: Vincini, M., E. Frazzi a P. D&#x27;Alessio. „A Broad-Band Leaf Chlorophyll Vegetation Index at the Canopy Scale.“ Precision Agriculture 9 (2008): 303–319._

***

## ENDVI – Vylepšený normalizovaný rozdílový vegetační index

Tento index využívá kromě pásma NIR a zeleného také modré pásmo a je oblíbený u kamer s filtrem NGB, kde modré pásmo nahrazuje červené.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

Implementace je symbolický vzorec `((x+y)-(2*z))/((x+y)+(2*z))` — přiřaďte kanály NIR a Green vaší kamery do slotů x/y a Blue do z (pro kameru NGB: x=NIR, y=Green, z=Blue).

***

## EVI – vylepšený vegetační index

Tento index byl původně vyvinut pro použití s daty MODIS jako vylepšení indexu NDVI prostřednictvím optimalizace signálu vegetace v oblastech s vysokým indexem listové plochy (LAI). Je nejužitečnější v oblastech s vysokým LAI, kde může docházet k nasycení indexu NDVI. Využívá oblast modré odrazivosti k korekci signálů půdního pozadí a ke snížení vlivů atmosféry, včetně rozptylu aerosolů.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Hodnoty EVI by se u pixelů s vegetací měly pohybovat v rozmezí od 0 do 1. Světlé prvky, jako jsou mraky a bílé budovy, spolu s tmavými prvky, jako je voda, mohou v obraze EVI způsobit anomální hodnoty pixelů. Před vytvořením obrazu EVI je třeba z obrazu odrazivosti odstranit mraky a světlé prvky a případně nastavit prahovou hodnotu pixelových hodnot v rozmezí 0 až 1.

_Odkaz: Huete, A., et al. „Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices.“ Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 – Index lesního porostu 1

_Pouze v grafickém uživatelském rozhraní (GUI) — není k dispozici jako přednastavení CLI/SDK `--indices`._

Tento index rozlišuje lesní porosty od jiných typů vegetace pomocí multispektrálních snímků odrazivosti, které zahrnují pásmo červeného okraje.

$$
FCI1 = Red * RedEdge
$$

Lesnaté oblasti budou mít nižší hodnoty FCI1 kvůli nižší odrazivosti stromů a přítomnosti stínů v korunách stromů.

_Odkaz: Becker, Sarah J., Craig S.T. Daughtry a Andrew L. Russ. „Robustní indexy lesního porostu pro multispektrální snímky.“ Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## FCI2 – Index lesního porostu 2

_Pouze v grafickém uživatelském rozhraní (GUI) — není k dispozici jako přednastavení CLI/SDK `--indices`._

Tento index rozlišuje lesní porosty od jiných typů vegetace pomocí multispektrálních snímků odrazivosti, které neobsahují pásmo červeného okraje.

$$
FCI2 = Red * NIR
$$

Lesnaté oblasti budou mít nižší hodnoty FCI2 kvůli nižší odrazivosti stromů a přítomnosti stínů v korunách stromů.

_Odkaz: Becker, Sarah J., Craig S.T. Daughtry a Andrew L. Russ. „Robustní indexy lesního porostu pro multispektrální snímky.“ Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## GEMI – Globální index monitorování životního prostředí

_Pouze GUI – není k dispozici jako přednastavení CLI/SDK `--indices`._

Tento nelineární vegetační index se používá pro globální monitorování životního prostředí na základě satelitních snímků a snaží se korigovat atmosférické vlivy. Je podobný indexu NDVI, ale je méně citlivý na atmosférické vlivy. Je ovlivňován holou půdou; proto se nedoporučuje jeho použití v oblastech s řídkou nebo středně hustou vegetací.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Kde:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Odkaz: Pinty, B., a M. Verstraete. „GEMI: nelineární index pro monitorování globální vegetace ze satelitů“. Vegetation 101 (1992): 15–20._

***

## GARI – Green Atmospherically Resistant Index

_Pouze GUI – není k dispozici jako přednastavení CLI/SDK `--indices`._

Tento index je citlivější na široký rozsah koncentrací chlorofylu a méně citlivý na atmosférické vlivy než NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Gamma konstanta je váhová funkce, která závisí na podmínkách aerosolů v atmosféře. ENVI používá hodnotu 1,7, což je hodnota doporučená Gitelsonem, Kaufmanem a Merzylakem (1996, strana 296).

_Odkaz: Gitelson, A., Y. Kaufman a M. Merzylak. „Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS.“ Remote Sensing of Environment 58 (1996): 289–298._

***

## GCI – Green Chlorophyll Index (zelený chlorofylový index)

Tento index se používá k odhadu obsahu chlorofylu v listech u široké škály rostlinných druhů.

$$
GCI = {NIR \over Green} - 1
$$

Využití širokého spektra vlnových délek v oblasti NIR a zelené barvy umožňuje lepší odhad obsahu chlorofylu a zároveň zajišťuje vyšší citlivost a lepší poměr signálu k šumu.

_Odkaz: Gitelson, A., Y. Gritz a M. Merzlyak. „Vztahy mezi obsahem chlorofylu v listech a spektrální odrazivostí a algoritmy pro nedestruktivní stanovení obsahu chlorofylu v listech vyšších rostlin.“ Journal of Plant Physiology 160 (2003): 271–282._

***

## GLI – index zelených listů (Green Leaf Index)

Tento index byl původně navržen pro použití s digitální RGB kamerou k měření porostu pšenice, kde se červené, zelené a modré digitální hodnoty (DN) pohybují v rozmezí od 0 do 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Hodnoty GLI se pohybují v rozmezí od -1 do +1. Záporné hodnoty představují půdu a neživé prvky, zatímco kladné hodnoty představují zelené listy a stonky.

_Odkaz: Louhaichi, M., M. Borman a D. Johnson. „Prostorově lokalizovaná platforma a letecké snímky pro dokumentaci dopadů pastvy na pšenici.“ Geocarto International 16, č. 1 (2001): 65–70._

***

## GNDVI – normalizovaný index vegetace v zelené oblasti (Green)

Tento index je podobný indexu NDVI, s tím rozdílem, že namísto červeného spektra měří zelené spektrum v rozmezí 540 až 570 nm. Tento index je citlivější na koncentraci chlorofylu než NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Zdroj: Gitelson, A., a M. Merzlyak. „Dálkový průzkum koncentrace chlorofylu v listech vyšších rostlin.“ Advances in Space Research 22 (1998): 689–692._

***

## GOSAVI – index vegetace optimalizovaný pro zelené spektrum a upravený pro půdu (Green Optimized Soil Adjusted Vegetation Index)

Tento index byl původně navržen pro barevnou infračervenou fotografii s cílem předpovídat potřebu dusíku u kukuřice. Je podobný indexu OSAVI, avšak nahrazuje červené spektrum zeleným.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Odkaz: Sripada, R., et al. „Stanovení sezónní potřeby dusíku pro kukuřici pomocí letecké barevné infračervené fotografie.“ Doktorská disertační práce, North Carolina State University, 2005._

***

## GRVI – Vegetace index zeleného poměru (Green Ratio Vegetation Index)

Tento index citlivě reaguje na rychlost fotosyntézy v lesním porostu, protože odrazivost zelené a červené barvy je silně ovlivněna změnami v listových pigmentech.

$$
GRVI = {NIR \over Green }
$$

_Odkaz: Sripada, R., et al. „Letecká barevná infračervená fotografie pro stanovení raných potřeb dusíku u kukuřice v průběhu vegetačního období.“ Agronomy Journal 98 (2006): 968–977._

***

## GSAVI – index vegetace upravený o zelenou půdu (Green Soil Adjusted Vegetation Index)

Tento index byl původně navržen pro použití s barevnou infračervenou fotografií za účelem předpovídání potřeby dusíku u kukuřice. Je podobný indexu SAVI, avšak nahrazuje červené pásmo zeleným.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Zdroj: Sripada, R., et al. „Stanovení potřeb dusíku u kukuřice v průběhu vegetačního období pomocí letecké barevné infračervené fotografie.“ Doktorská práce, North Carolina State University, 2005._

***

## LAI – index listové plochy

Tento index se používá k odhadu listového pokryvu a k předpovědi růstu a výnosu plodin. ENVI vypočítává zelený LAI pomocí následujícího empirického vzorce od Boegha a kol. (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Kde EVI je:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Vysoké hodnoty LAI se obvykle pohybují v rozmezí přibližně 0 až 3,5. Pokud však scéna obsahuje mraky a jiné jasné prvky, které způsobují nasycené pixely, mohou hodnoty LAI překročit 3,5. Ideálně byste měli před vytvořením snímku LAI z vaší scény odstranit mraky a jasné prvky.

_Odkaz: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde a A. Thomsen. „Letecká multispektrální data pro kvantifikaci indexu listové plochy, koncentrace dusíku a fotosyntetické účinnosti v zemědělství.“ Remote Sensing of Environment 81, č. 2–3 (2002): 179–193._

***

## LCI – index chlorofylu listů

_Pouze v grafickém uživatelském rozhraní (GUI) — není k dispozici jako přednastavení CLI/SDK `--indices`._

Tento index se používá k odhadu obsahu chlorofylu ve vyšších rostlinách a je citlivý na změny odrazivosti způsobené absorpcí chlorofylu.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Odkaz: Datt, B. „Remote Sensing of Water Content in Eucalyptus Leaves.“ Journal of Plant Physiology 154, č. 1 (1999): 30–36._

***

## MNLI – Modifikovaný nelineární index

Tento index je vylepšením nelineárního indexu (NLI), který zahrnuje index vegetace upravený o vliv půdy (SAVI), aby zohlednil půdní pozadí. ENVI používá hodnotu faktoru úpravy pozadí korunového zápoje (_L_) rovnou 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Odkaz: Yang, Z., P. Willis a R. Mueller. „Vliv snímků AWIFS s vylepšeným poměrem pásem na přesnost klasifikace plodin.“ Sborník ze sympozia Pecora 17 o dálkovém průzkumu Země (2008), Denver, CO._

***

## MSAVI2 – Modifikovaný index vegetace upravený o půdu 2

Tento index je zjednodušenou verzí indexu MSAVI navrženého Qi et al. (1994), který představuje vylepšení indexu vegetace upraveného o půdní vliv (SAVI). Snižuje půdní šum a zvyšuje dynamický rozsah signálu vegetace. MSAVI2 je založen na induktivní metodě, která k zvýraznění zdravé vegetace nepoužívá konstantní hodnotu _L_ (jako u SAVI).

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Odkaz: Qi, J., A. Chehbouni, A. Huete, Y. Kerr a S. Sorooshian. „A Modified Soil Adjusted Vegetation Index.“ Remote Sensing of Environment 48 (1994): 119–126._

***

## MSR – Modified Simple Ratio (modifikovaný jednoduchý poměr)

Tento index je modifikací jednoduchého poměru NIR/Red, jehož cílem je linearizovat jeho vztah k biofyzikálním parametrům, a je citlivější než NDVI při vyšší hustotě vegetace.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Zdroj: Chen, J. „Evaluation of Vegetation Indices and a Modified Simple Ratio for Boreal Applications.“ Canadian Journal of Remote Sensing 22 (1996): 229–242._

***

## NDRE – Normalizovaný rozdíl RedEdge

Tento index je podobný indexu NDVI, avšak porovnává kontrast mezi NIR a RedEdge namísto Red, což často umožňuje včasnější detekci stresu vegetace.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – Normalizovaný rozdíl vegetačního indexu

Tento index je měřítkem zdravé, zelené vegetace. Kombinace jeho normalizovaného rozdílu a využití oblastí s nejvyšší absorpcí a odrazivostí chlorofylu zajišťuje jeho spolehlivost v širokém spektru podmínek. Může však v podmínkách husté vegetace, kdy je hodnota LAI vysoká, dojít k jeho nasycení.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Hodnota tohoto indexu se pohybuje v rozmezí od -1 do 1. Běžný rozsah pro zelenou vegetaci je 0,2 až 0,8.

_Zdroj: Rouse, J., R. Haas, J. Schell a D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. Třetí sympozium ERTS, NASA (1973): 309–317._

***

## NLI – nelineární index

Tento index vychází z předpokladu, že vztah mezi mnoha vegetačními indexy a biofyzikálními parametry povrchu je nelineární. Linearizuje vztahy s povrchovými parametry, které mají tendenci být nelineární.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Odkaz: Goel, N., a W. Qin. „Vlivy architektury korunového zápoje na vztahy mezi různými vegetačními indexy a LAI a Fpar: počítačová simulace.“ Remote Sensing Reviews 10 (1994): 309–347._

***

## OSAVI – optimalizovaný index vegetace upravený o půdní podmínky

Tento index vychází z indexu vegetace upraveného o půdní podmínky (SAVI). Pro faktor korekce pozadí korunového zápoje používá standardní hodnotu 0,16. Rondeaux (1996) zjistil, že tato hodnota poskytuje větší variabilitu v závislosti na půdě než SAVI u nízkého vegetačního pokryvu, přičemž vykazuje zvýšenou citlivost u vegetačního pokryvu většího než 50 %. Tento index se nejlépe používá v oblastech s relativně řídkým porostem, kde je půda viditelná skrz korunový zápoje.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Zdroj: Rondeaux, G., M. Steven a F. Baret. „Optimization of Soil-Adjusted Vegetation Indices.“ Remote Sensing of Environment 55 (1996): 95–107._

***

## RDVI – renormalizovaný rozdílový vegetační index

Tento index využívá rozdíl mezi vlnovými délkami v blízké infračervené a červené oblasti spolu s indexem NDVI k zvýraznění zdravé vegetace. Je necitlivý na vlivy půdy a geometrie pozorování Slunce.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Odkaz: Roujean, J. a F. Breon. „Odhad PAR absorbovaného vegetací na základě měření obousměrné odrazivosti.“ Remote Sensing of Environment 51 (1995): 375–384._

***

## SAVI – index vegetace upravený o vliv půdy

Tento index je podobný indexu NDVI, potlačuje však vliv pixelů půdy. Využívá korekční faktor pozadí korunového zápoje, _L_, který je funkcí hustoty vegetace a často vyžaduje předchozí znalost množství vegetace. Huete (1988) navrhuje optimální hodnotu _L_=0,5 pro zohlednění variací půdního pozadí prvního řádu. Tento index se nejlépe používá v oblastech s relativně řídkou vegetací, kde je půda viditelná skrz korunový zápoje.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Odkaz: Huete, A. „A Soil-Adjusted Vegetation Index (SAVI).“ Remote Sensing of Environment 25 (1988): 295–309._

***

## TDVI – transformovaný rozdílový vegetační index

Tento index je užitečný pro monitorování vegetačního pokryvu v městském prostředí. Na rozdíl od indexů NDVI a SAVI nedochází k jeho nasycení.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Odkaz: Bannari, A., H. Asalhi a P. Teillet. „Transformovaný rozdílový vegetační index (TDVI) pro mapování vegetačního pokryvu“ In Sborník z sympozia o geovědách a dálkovém průzkumu Země, IGARSS &#x27;02, IEEE International, svazek 5 (2002)._

***

## VARI – index viditelného spektra odolný vůči atmosférickým vlivům

Tento index vychází z indexu ARVI a používá se k odhadu podílu vegetace ve scéně s nízkou citlivostí na atmosférické vlivy.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Odkaz: Gitelson, A., et al. „Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction.“ International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – index vegetace s širokým dynamickým rozsahem

Tento index je podobný indexu NDVI, avšak využívá váhový koeficient (_a_) ke snížení rozdílu mezi příspěvky signálů z blízké infračervené a červené oblasti do indexu NDVI. Index WDRVI je obzvláště účinný ve scénách se střední až vysokou hustotou vegetace, kdy hodnota NDVI přesahuje 0,6. NDVI má tendenci se stabilizovat s rostoucím podílem vegetace a indexem listové plochy (LAI), zatímco WDRVI je citlivější na širší rozsah podílů vegetace a na změny LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Váhový koeficient (_a_) se může pohybovat v rozmezí od 0,1 do 0,2. Hodnotu 0,2 doporučují Henebry, Viña a Gitelson (2004).

_Literatura_

_Gitelson, A. „Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.“ Journal of Plant Physiology 161, č. 2 (2004): 165–173._

_Henebry, G., A. Viña a A. Gitelson. „Vegetační index s širokým dynamickým rozsahem a jeho potenciální využití pro analýzu mezer.“ Gap Analysis Bulletin 12: 50–56._
