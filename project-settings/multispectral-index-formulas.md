---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Vzorce pro multispektrální indexy

Níže uvedené vzorce pro indexy využívají kombinaci průměrných rozsahů propustnosti filtru Survey3:

<table><thead><tr><th align="center">Barva filtru Survey3</th><th width="196.199951171875" align="center">Survey3 Název filtru</th><th width="159.800048828125" align="center">Rozsah propustnosti (FWHM)</th><th align="center">Průměrná propustnost</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB – Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN – Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN – Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN – Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN – NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR – NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>Při použití těchto vzorců může název končit znakem „\_1“ nebo „\_2“, což odpovídá tomu, který filtr – buď NIR, NIR1, nebo NIR2 – byl použit.

U kamer LATTICE M3C (Bayerův trojpásmový filtr) používá stejný indexovací modul pásma filtrů M3C:

| Filtr M3C | Pásmo 1 (střed/FWHM) | Pásmo 2 (střed/FWHM) | Pásmo 3 (střed/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

Kamery LATTICE M3M jsou jednopásmové (jeden úzkopásmový filtr na kameru), proto se pro samostatný snímek M3M nepočítají vícepásmové indexy. Chcete-li vypočítat indexy pomocí M3M, zkombinujte dvě nebo více kamer do zarovnaného vícepásmového stacku a použijte indexovací engine LATTICE (`chloros-cli lattice index` nebo živou kalkulačku indexů v grafickém rozhraní).

***

## Kde funguje který název indexu

Chloros má **tři** indexové plochy a jejich přednastavené seznamy nejsou identické. V této části si ověřte, zda daný název bude fungovat tam, kde jej plánujete použít.

| Kde se nacházíte | Který seznam platí | Počet |
| --- | --- | --- |
| Nastavení projektu → Index → Přidat index (GUI) | Povrch 1 | 27 |
| Prohlížeč obrázků [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) (GUI) | Povrch 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | Povrch 2 | 22 |
| SDK `process_folder(indices=[...])` | Povrch 2 | 22 |
| `chloros-cli lattice index --preset` | Povrch 3 | 22 (jiných 22) |
| Karta „Kamery“ – kalkulátor živého indexu | Surface 3 | 22 (jiné číslo 22) |

Surface 1 a 2 pracují s **jedním snímkem najednou z jedné kamery**, přičemž používají symbolové sloty `x`/`y`/`z`(/`a`), které jsou vázány na filtrační kanály dané kamery. Surface 3 pracuje s**zarovnaným vícepásmovým stackem** — několika kamerami LATTICE, které jsou společně zaregistrovány do jednoho krychlového souboru — a odkazuje na kanály pomocí malých písmen.

### 1. Nastavení projektu v grafickém uživatelském rozhraní / rozevírací seznam v sandboxu prohlížeče obrázků — 27 vzorců

Rozevírací seznam je uvádí v tomto pořadí (jedná se o pořadí vložení, nikoli abecední):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

V grafickém uživatelském rozhraní přetáhnete filtrační kanály vaší kamery do slotů pro pásma vzorce, takže jakýkoli vzorec lze použít s jakýmkoli přiřazením pásem, které vaše kamera podporuje. Vlastní vzorce, které jste uložili, jsou připojeny pod tímto seznamem.

**Pět vzorců dostupných pouze v grafickém uživatelském rozhraní** — ty, které seznam CLI/SDK `--indices` nepřijímá — jsou implementovány takto:

| Předvolba dostupná pouze v grafickém uživatelském rozhraní | Vzorec (jak je implementován) | Sloty |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a (čtyři sloty) |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

Zamýšlené přiřazení pro každý z nich je uvedeno v samostatné části dále na této stránce (například GARI očekává x=Green, y=NIR, z=Blue, a=Red). GARI je jediný vzorec v Chloros, který používá čtvrtý slot.

### 2. Rozšíření názvu CLI / SDK `--indices` — 22 předvoleb

Možnost `chloros-cli process --indices` (a parametr SDK `indices`) podporuje následující názvy předvoleb:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**Neznámé názvy indexů jsou bez upozornění přeskočeny.** Názvy mimo tento seznam (včetně pěti vzorců určených pouze pro grafické rozhraní `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` a jakékoli vlastní vzorce, které jste uložili v grafickém rozhraní) je vynechán pouze s poznámkou v protokolu — běh pokračuje bez tohoto indexu a samotný běh je i tak vyhodnocen jako úspěšný. Upozornění se vypíše takto:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

Názvy se porovnávají bez rozlišování velkých a malých písmen po odstranění mezer, takže `ndvi`, `NDVI` a ` NDVI ` představují stejné přednastavení. Předvolba je také přeskočena, pokud vyžaduje pásmo, které filtr vaší kamery neposkytuje.
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
| CVI | `(z/y)/(x/y)` | RGB | Red, Green, Blue |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | RGB | Red, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### Jak se z názvu předvolby stanou pozice pásem

Když zadáte pouhý název, jako je například `NDVI`, musí Chloros rozhodnout, který kanál kterého souboru každý symbol čte. Používá k tomu tuto tabulku, která přiřazuje kód filtru k pozici každého kanálu v poli:

| Kód filtru | Kanál → index pole |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red` je akceptován jako alias pro Orange, rovněž 0) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

**Výchozí filtr** předvolby (sloupec „Výchozí filtr“ výše) se použije, pokud projekt obsahuje obrázky s tímto filtrem. Pokud tomu tak není, Chloros prohledá filtry skutečně přítomné v projektu v pořadí `RGN, OCN, NGB, RGB, RE, NIR` a vybere první z nich, který dokáže poskytnout všechny kanály, které předvolba potřebuje. Pokud žádný z nich nemůže, předvolba je pro daný běh vynechána. Proto `NDVI` vyžádaný na datovém souboru obsahujícím pouze OCNstále poskytuje smysluplný výsledek — váže se na pozice Orange a NIR v OCN.

Řetězce modelu LATTICE M3C obsahují filtr s předponou `F` (`LATT-M3C-L41-FRGN`), ale při čtení kódu filtru z obrazu se tato předpona vynechá, takže kamera FRGN rozlišuje pomocí řádku `RGN` výše a nevyžaduje žádné zvláštní zpracování.

### 3. Indexovací engine LATTICE (`lattice index --preset`, živý Index Calculator) — 22 předvoleb

Modul LATTICE pracuje se zarovnanými vícepásmovými sériemi snímků (živá pole nebo exportované vícepásmové soubory TIFF) a používá názvy kanálů psané malými písmeny (`red`, `green`, `blue`, `red_edge`, `nir`). Jeho seznam předvoleb se liší od obou výše uvedených:

| Předvolba | Vzorec | Kanály |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | červená, NIR |
| GNDVI | `(nir - green) / (nir + green)` | zelená, NIR |
| BNDVI | `(nir - blue) / (nir + blue)` | modrá, NIR |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | červená\_okraj, nir |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | modrá, zelená, nir |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | červená, nir |
| OSAVI | `1.5 * (nir - red) / (nir + red + 0.16)` | červená, nir |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | červená, NIR |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | modrá, červená, NIR |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | červená, NIR |
| CVI | `(nir / green) - (red / green)` | červená, zelená, NIR |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | červená, NIR |
| TDVI | `sqrt((nir - red) / (nir + red) + 0.5)` | červená, NIR |
| LAI | `3.618 * ((nir - red) / (nir + 6*red - 7.5*green + 1)) - 0.118` | červená, zelená, NIR |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | červená, zelená, modrá |
| NGRDI | `(green - red) / (green + red)` | červená, zelená |
| VARI | `(green - red) / (green + red - blue)` | červená, zelená, modrá |
| TGI | `green - 0.39*red - 0.61*blue` | červená, zelená, modrá |
| EXG | `2*green - red - blue` | červená, zelená, modrá |
| CIRE | `(nir / red_edge) - 1` | červená\_okraj, nir |
| CIGREEN | `(nir / green) - 1` | zelená, nir |
| NDWI | `(green - nir) / (green + nir)` | zelená, nir |

Spusťte příkaz `chloros-cli lattice index --list-presets` pro vytištění této tabulky z nainstalované verze a příkaz `--list-gradients` pro dostupné barevné přechody. U symbolů kanálů se rozlišují velká a malá písmena a musí se shodovat s názvy předvoleb psanými malými písmeny (např. `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI

Jak je implementováno v grafickém uživatelském rozhraní a v seznamu předvoleb CLI/SDK, CVI je vzorec poměru poměrů:

$$
CVI = {(z / y) \over (x / y)}
$$

s výchozím přiřazením kanálů RGB, kde x = Red, y=Green, z=Blue. V grafickém uživatelském rozhraní (GUI) můžete přetáhnout libovolný kanál vaší kamery do slotů x/y/z. Upozorňujeme, že přednastavení `CVI` indexovacího modulu LATTICE používá odlišný vzorec, `(NIR / Green) - (Red / Green)` — pro povrch, který používáte, zkontrolujte tabulky výše.

***

## ENDVI – Vylepšený normalizovaný rozdíl vegetačního indexu

Tento index kromě NIR a zeleného kanálu využívá také modrý kanál a je oblíbený u kamer s filtrem NGB, kde modré pásmo nahrazuje červené.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

Implementace je symbolický vzorec `((x+y)-(2*z))/((x+y)+(2*z))` — přiřaďte kanály NIR a Green vaší kamery do slotů x/slotům a Blue do z (pro kameru NGB: x=NIR, y=Green, z=Blue).

***

## EVI - Vylepšený vegetační index

Tento index byl původně vyvinut pro použití s daty MODIS jako vylepšení indexu NDVI prostřednictvím optimalizace vegetačního signálu v oblastech s vysokým indexem listové plochy (LAI). Je nejužitečnější v oblastech s vysokými hodnotami LAI, kde může dojít k nasycení indexu NDVI. Využívá oblast modré odrazivosti k korekci signálů půdního pozadí a ke snížení atmosférických vlivů, včetně rozptylu aerosolů.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Hodnoty EVI by se u pixelů s vegetací měly pohybovat v rozmezí od 0 do 1. Světlé prvky, jako jsou mraky a bílé budovy, spolu s tmavými prvky, jako je voda, mohou vést k anomálním hodnotám pixelů v obraze EVI. Před vytvořením obrazu EVI je třeba z obrazu odrazivosti odstranit mraky a jasné prvky a případně nastavit prahovou hodnotu pixelů v rozmezí 0 až 1.

_Odkaz: Huete, A., et al. „Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices.“ Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 – Index lesního porostu 1

_Pouze v grafickém uživatelském rozhraní (GUI) — není k dispozici jako předvolba CLI/SDK `--indices`._

Tento index rozlišuje lesní porosty od jiných typů vegetace pomocí multispektrálních snímků odrazivosti, které zahrnují pásmo červeného okraje.

$$
FCI1 = Red * RedEdge
$$

Lesnaté oblasti budou mít nižší hodnoty FCI1 kvůli nižší odrazivosti stromů a přítomnosti stínů v korunách stromů.

_Odkaz: Becker, Sarah J., Craig S.T. Daughtry a Andrew L. Russ. „Robustní indexy lesního porostu pro multispektrální snímky.“ Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## FCI2 - Index lesního porostu 2

_Pouze v grafickém uživatelském rozhraní (GUI) — není k dispozici jako přednastavení CLI/SDK `--indices`._

Tento index rozlišuje lesní porosty od jiných typů vegetace pomocí multispektrálních snímků odrazivosti, které neobsahují pásmo červeného okraje.

$$
FCI2 = Red * NIR
$$

Lesnaté oblasti budou mít nižší hodnoty FCI2 kvůli nižší odrazivosti stromů a přítomnosti stínů v korunách stromů.

_Odkaz: Becker, Sarah J., Craig S.T. Daughtry a Andrew L. Russ. „Robustní indexy lesního porostu pro multispektrální snímky.“ Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## GEMI – Globální index monitorování životního prostředí

_Pouze v grafickém uživatelském rozhraní (GUI) — není k dispozici jako předvolba CLI/SDK `--indices`._

Tento nelineární vegetační index se používá pro globální monitorování životního prostředí na základě satelitních snímků a snaží se korigovat vlivy atmosféry. Je podobný indexu NDVI, ale je méně citlivý na vlivy atmosféry. Je ovlivněn holou půdou; proto se nedoporučuje používat v oblastech s řídkou nebo středně hustou vegetací.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Kde:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Odkaz: Pinty, B., a M. Verstraete. GEMI: nelineární index pro globální monitorování vegetace pomocí satelitů. Vegetation 101 (1992): 15–20._

***

## GARI – Green Index odolný vůči atmosférickým vlivům

_Pouze v grafickém uživatelském rozhraní (GUI) — není k dispozici jako předvolba CLI/SDK `--indices`._

Tento index je citlivější na široký rozsah koncentrací chlorofylu a méně citlivý na atmosférické vlivy než NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Konstanta gama je váhová funkce, která závisí na podmínkách aerosolů v atmosféře. ENVI používá hodnotu 1,7, což je hodnota doporučená Gitelsonem, Kaufmanem a Merzylakem (1996, strana 296).

_Odkaz: Gitelson, A., Y. Kaufman a M. Merzylak. „Využití kanálu Green při dálkovém průzkumu globální vegetace z EOS-MODIS.“ Remote Sensing of Environment 58 (1996): 289–298._

***

## GCI – Green – index chlorofylu

Tento index se používá k odhadu obsahu chlorofylu v listech u široké škály rostlinných druhů.

$$
GCI = {NIR \over Green} - 1
$$

Využití širokého spektra vlnových délek NIR a zelených vlnových délek umožňuje lepší odhad obsahu chlorofylu a zároveň zajišťuje vyšší citlivost a lepší poměr signálu k šumu.

_Odkaz: Gitelson, A., Y. Gritz a M. Merzlyak. „Vztahy mezi obsahem chlorofylu v listech a spektrální odrazivostí a algoritmy pro nedestruktivní stanovení obsahu chlorofylu v listech vyšších rostlin.“ Journal of Plant Physiology 160 (2003): 271–282._

***

## GLI – Green Listový index

Tento index byl původně navržen pro použití s digitální kamerou RGB k měření pokryvu pšenice, přičemž červené, zelené a modré digitální hodnoty (DN) se pohybují v rozmezí od 0 do 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Hodnoty GLI se pohybují v rozmezí od -1 do +1. Záporné hodnoty představují půdu a neživé prvky, zatímco kladné hodnoty představují zelené listy a stonky.

_Odkaz: Louhaichi, M., M. Borman a D. Johnson. „Prostorově lokalizovaná platforma a letecké snímky pro dokumentaci dopadů pastvy na pšenici.“ Geocarto International 16, č. 1 (2001): 65–70._

***

## GNDVI – Green Normalizovaný rozdíl vegetačního indexu

Tento index je podobný indexu NDVI s tím rozdílem, že namísto červeného spektra měří zelené spektrum v rozmezí 540 až 570 nm. Tento index je citlivější na koncentraci chlorofylu než index NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Odkaz: Gitelson, A., a M. Merzlyak. „Dálkový průzkum koncentrace chlorofylu v listech vyšších rostlin.“ Advances in Space Research 22 (1998): 689–692._

***

## GOSAVI – Green Optimalizovaný index vegetace upravený o půdní podmínky

Tento index byl původně navržen pro použití s barevnou infračervenou fotografií za účelem předpovědi potřeby dusíku u kukuřice. Je podobný indexu OSAVI, avšak nahrazuje zelené spektrální pásmo červeným.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Odkaz: Sripada, R., et al. „Stanovení sezónní potřeby dusíku pro kukuřici pomocí letecké barevné infračervené fotografie.“ Doktorská disertační práce, North Carolina State University, 2005._

***

## Poměr vegetace GRVI – Green

Tento index je citlivý na míru fotosyntézy v lesním porostu, protože odrazivost zeleného a červeného spektra je silně ovlivněna změnami listových pigmentů.

$$
GRVI = {NIR \over Green }
$$

_Odkaz: Sripada, R., et al. „Letecká barevná infračervená fotografie pro stanovení raných sezónních potřeb dusíku u kukuřice.“ Agronomy Journal 98 (2006): 968–977._

***

## GSAVI – Green Vegetace index upravený o půdní podmínky

Tento index byl původně navržen s využitím barevné infračervené fotografie k předpovědi potřeb dusíku u kukuřice. Je podobný indexu SAVI, avšak nahrazuje zelené spektrální pásmo červeným.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Odkaz: Sripada, R., et al. „Stanovení sezónní potřeby dusíku pro kukuřici pomocí letecké barevné infračervené fotografie.“ Doktorská disertační práce, North Carolina State University, 2005._

***

## LAI - Index listové plochy

Tento index se používá k odhadu listového pokryvu a k předpovědi růstu a výnosu plodin. ENVI vypočítává zelený LAI pomocí následujícího empirického vzorce od Boegha a kol. (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Kde EVI je:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Vysoké hodnoty LAI se obvykle pohybují v rozmezí přibližně 0 až 3,5. Pokud však scéna obsahuje mraky a jiné jasné prvky, které způsobují nasycené pixely, mohou hodnoty LAI překročit 3,5. Ideálně byste měli před vytvořením snímku LAI z obrazu odstranit mraky a jasné prvky.

_Zdroj: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde a A. Thomsen. „Letecká multispektrální data pro kvantifikaci indexu listové plochy, koncentrace dusíku a fotosyntetické účinnosti v zemědělství.“ Remote Sensing of Environment 81, č. 2–3 (2002): 179–193._

***

## LCI – Index chlorofylu v listech

_Pouze v grafickém uživatelském rozhraní (GUI) — není k dispozici jako předvolba CLI/SDK `--indices`._

Tento index se používá k odhadu obsahu chlorofylu ve vyšších rostlinách, které jsou citlivé na změny odrazivosti způsobené absorpcí chlorofylu.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Odkaz: Datt, B. „Remote Sensing of Water Content in Eucalyptus Leaves.“ Journal of Plant Physiology 154, č. 1 (1999): 30–36._

***

## MNLI – Modifikovaný nelineární index

Tento index představuje vylepšení nelineárního indexu (NLI), které zahrnuje index vegetace upravený o půdu (SAVI) s cílem zohlednit půdní pozadí. ENVI používá hodnotu faktoru korekce pozadí korunového zápoje (_L_) rovnou 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Zdroj: Yang, Z., P. Willis a R. Mueller. „Vliv snímku AWIFS s vylepšeným poměrem pásem na přesnost klasifikace plodin.“ Sborník z sympozia Pecora 17 o dálkovém průzkumu Země (2008), Denver, CO._

***

## MSAVI2 – Modifikovaný index vegetace upravený o půdní podmínky 2

Tento index je zjednodušenou verzí indexu MSAVI navrženého Qi et al. (1994), který vylepšuje index vegetace upravený o půdu (SAVI). Snižuje půdní šum a zvyšuje dynamický rozsah signálu vegetace. MSAVI2 je založen na induktivní metodě, která k zvýraznění zdravé vegetace nepoužívá konstantní hodnotu _L_ (jako v případě SAVI).

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Odkaz: Qi, J., A. Chehbouni, A. Huete, Y. Kerr a S. Sorooshian. „A Modified Soil Adjusted Vegetation Index.“ Remote Sensing of Environment 48 (1994): 119–126._

***

## MSR – modifikovaný jednoduchý poměr

Tento index je modifikací jednoduchého poměru NIR/Red, jehož cílem je linearizovat jeho vztah k biofyzikálním parametrům, a je citlivější než NDVI při vyšší hustotě vegetace.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Zdroj: Chen, J. „Evaluation of Vegetation Indices and a Modified Simple Ratio for Boreal Applications.“ Canadian Journal of Remote Sensing 22 (1996): 229–242._

***

## NDRE – Normalizovaný rozdíl RedEdge

Tento index je podobný indexu NDVI, avšak porovnává kontrast mezi NIR a RedEdge namísto Red, který často detekuje stres vegetace dříve.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – Normalizovaný rozdíl vegetace

Tento index je měřítkem zdravé, zelené vegetace. Kombinace jeho normalizovaného rozdílu a využití oblastí s nejvyšší absorpcí a odrazivostí chlorofylu zajišťuje jeho robustnost v širokém spektru podmínek. Může však dojít k jeho nasycení v podmínkách husté vegetace, když hodnota LAI stoupne.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Hodnota tohoto indexu se pohybuje v rozmezí od -1 do 1. Obvyklý rozsah pro zelenou vegetaci je 0,2 až 0,8.

_Zdroj: Rouse, J., R. Haas, J. Schell a D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. Třetí sympozium ERTS, NASA (1973): 309–317._

***

## NLI – nelineární index

Tento index vychází z předpokladu, že vztah mezi mnoha vegetačními indexy a biofyzikálními parametry povrchu je nelineární. Linearizuje vztahy s povrchovými parametry, které mají tendenci být nelineární.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Odkaz: Goel, N., a W. Qin. „Vlivy architektury korun stromů na vztahy mezi různými vegetačními indexy a LAI a Fpar: počítačová simulace.“ Remote Sensing Reviews 10 (1994): 309–347._

***

## OSAVI – Optimalizovaný vegetační index upravený o půdní vliv

Tento index vychází z vegetačního indexu upraveného o půdní vliv (SAVI). Používá standardní hodnotu 0,16 pro korekční faktor pozadí korunového zápoje. Rondeaux (1996) zjistil, že tato hodnota poskytuje větší variabilitu půdy než SAVI v případě nízkého vegetačního pokryvu, přičemž vykazuje zvýšenou citlivost na vegetační pokryv větší než 50 %. Tento index se nejlépe používá v oblastech s relativně řídkým porostem, kde je půda viditelná skrz korunový zápoje.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Odkaz: Rondeaux, G., M. Steven a F. Baret. „Optimization of Soil-Adjusted Vegetation Indices.“ Remote Sensing of Environment 55 (1996): 95–107._

***

## RDVI – Renormalizovaný rozdílový vegetační index

Tento index využívá rozdíl mezi vlnovými délkami v blízké infračervené a červené oblasti spolu s indexem NDVI k zvýraznění zdravé vegetace. Je necitlivý na vlivy půdy a geometrie pozorování Slunce.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Odkaz: Roujean, J., a F. Breon. „Odhad PAR absorbovaného vegetací na základě měření obousměrné odrazivosti.“ Remote Sensing of Environment 51 (1995): 375–384._

***

## SAVI – Index vegetace s korekcí na půdu

Tento index je podobný indexu NDVI, potlačuje však vliv pixelů půdy. Využívá korekční faktor pozadí korunového zápoje, _L_, který je funkcí hustoty vegetace a často vyžaduje předchozí znalost množství vegetace. Huete (1988) navrhuje optimální hodnotu _L_=0,5 pro zohlednění variací půdního pozadí prvního řádu. Tento index se nejlépe používá v oblastech s relativně řídkým porostem, kde je půda viditelná skrz porost.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Odkaz: Huete, A. „A Soil-Adjusted Vegetation Index (SAVI).“ Remote Sensing of Environment 25 (1988): 295–309._

***

## TDVI – transformovaný rozdílový vegetační index

Tento index je užitečný pro monitorování vegetačního pokryvu v městském prostředí. Nedochází u něj k nasycení jako u indexů NDVI a SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Odkaz: Bannari, A., H. Asalhi a P. Teillet. „Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping“ In Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, svazek 5 (2002)._

***

## VARI - Index viditelného spektra odolný vůči atmosférickým vlivům

Tento index vychází z indexu ARVI a slouží k odhadu podílu vegetace ve snímku s nízkou citlivostí na atmosférické vlivy.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Odkaz: Gitelson, A., et al. „Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction.“ International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – Index vegetace s širokým dynamickým rozsahem

Tento index je podobný indexu NDVI, ale využívá váhový koeficient (_a_) ke snížení rozdílu mezi příspěvky signálů v blízké infračervené a červené oblasti spektra do indexu NDVI. Index WDRVI je obzvláště účinný v oblastech se střední ažvysokou hustotou vegetace, pokud hodnota NDVI přesáhne 0,6. Hodnota NDVI má tendenci se stabilizovat při nárůstu podílu vegetace a indexu listové plochy (LAI) rostou, zatímco WDRVI je citlivější na širší rozsah podílů vegetace a na změny v LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Váhový koeficient (_a_) se může pohybovat v rozmezí od 0,1 do 0,2. Hodnotu 0,2 doporučují Henebry, Viña a Gitelson (2004).

_Literatura_

_Gitelson, A. „Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.“ Journal of Plant Physiology 161, č. 2 (2004): 165–173._

_Henebry, G., A. Viña a A. Gitelson. „Vegetační index s širokým dynamickým rozsahem a jeho potenciální využití pro analýzu mezer.“ Gap Analysis Bulletin 12: 50–56._
