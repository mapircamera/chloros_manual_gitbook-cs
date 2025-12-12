---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Vzorce multispektrálního indexu

Níže uvedené vzorce indexu používají kombinaci průměrných rozsahů propustnosti filtru Survey3:

<table><thead><tr><th align="center">Survey3 Barva filtru</th><th width="196.199951171875" align="center">Survey3 Název filtru</th><th width="159.800048828125" align="center">Rozsah propustnosti (FWHM)</th><th align="center">Průměrná propustnost</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>

Při použití těchto vzorců může název končit na „\_1“ nebo „\_2“, což odpovídá tomu, který filtr NIR, NIR1 nebo NIR2 byl použit.

***

## EVI – vylepšený index vegetace

Tento index byl původně vyvinut pro použití s daty MODIS jako vylepšení oproti NDVI optimalizací signálu vegetace v oblastech s vysokým indexem listové plochy (LAI). Je nejužitečnější v oblastech s vysokým LAI, kde může dojít k nasycení NDVI. Používá modrou odrazivost k opravě signálů pozadí půdy a ke snížení atmosférických vlivů, včetně rozptylu aerosolů.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Hodnoty EVI by se měly pohybovat v rozmezí 0 až 1 pro pixely vegetace. Světlé prvky, jako jsou mraky a bílé budovy, spolu s tmavými prvky, jako je voda, mohou vést k anomálním hodnotám pixelů v obraze EVI. Před vytvořením obrazu EVI byste měli z obrazu odrazivosti odstranit mraky a jasné prvky a případně nastavit prahovou hodnotu pixelů od 0 do 1.

_Reference: Huete, A., et al. „Přehled radiometrických a biofyzikálních vlastností indexů vegetace MODIS.“ Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 – Index lesního porostu 1

Tento index odlišuje lesní porosty od jiných typů vegetace pomocí multispektrálních odrazivých snímků, které zahrnují červené okrajové pásmo.

$$
FCI1 = Red * RedEdge
$$

Zalesněné oblasti budou mít nižší hodnoty FCI1 kvůli nižší odrazivosti stromů a přítomnosti stínů v korunách stromů.

_Reference: Becker, Sarah J., Craig S.T. Daughtry a Andrew L. Russ. „Robustní indexy lesního porostu pro multispektrální snímky.“ Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 – Index lesního porostu 2

Tento index rozlišuje korunové vrstvy lesů od jiných typů vegetace pomocí multispektrálních odrazivých snímků, které neobsahují červené okrajové pásmo.

$$
FCI2 = Red * NIR
$$

Zalesněné oblasti budou mít nižší hodnoty FCI2 kvůli nižší odrazivosti stromů a přítomnosti stínů v korunách stromů.

_Reference: Becker, Sarah J., Craig S.T. Daughtry a Andrew L. Russ. „Robust forest cover indices for multispectral images.“ Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI – Index globálního monitorování životního prostředí

Tento nelineární index vegetace se používá pro globální monitorování životního prostředí z družicových snímků a snaží se korigovat atmosférické vlivy. Je podobný NDVI, ale je méně citlivý na atmosférické vlivy. Je ovlivněn holou půdou, proto se nedoporučuje používat v oblastech s řídkou nebo středně hustou vegetací.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Kde:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Reference: Pinty, B., a M. Verstraete. GEMI: nelineární index pro monitorování globální vegetace ze satelitů. Vegetation 101 (1992): 15-20._

***

## GARI - Green Index odolný vůči atmosférickým vlivům

Tento index je citlivější na široký rozsah koncentrací chlorofylu a méně citlivý na atmosférické vlivy než NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Konstanta gama je váhová funkce, která závisí na podmínkách aerosolu v atmosféře. ENVI používá hodnotu 1,7, což je doporučená hodnota od Gitelsona, Kaufmana a Merzylaka (1996, strana 296).

_Reference: Gitelson, A., Y. Kaufman a M. Merzylak. „Použití kanálu Green při dálkovém snímání globální vegetace z EOS-MODIS.“ Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI – Green Chlorofylový index

Tento index se používá k odhadu obsahu chlorofylu v listech u široké škály rostlinných druhů.

$$
GCI = {NIR \over Green} - 1
$$

Široké spektrum NIR a zelené vlnové délky umožňují lepší předpověď obsahu chlorofylu a zároveň zajišťují vyšší citlivost a lepší poměr signálu k šumu.

_Reference: Gitelson, A., Y. Gritz a M. Merzlyak. „Vztahy mezi obsahem chlorofylu v listech a spektrální odrazivostí a algoritmy pro nedestruktivní hodnocení chlorofylu v listech vyšších rostlin.“ Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI – Green Listový index

Tento index byl původně navržen pro použití s digitální kamerou RGB k měření pokryvu pšenice, kde červená, zelená a modrá digitální čísla (DN) mají rozsah od 0 do 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Hodnoty GLI se pohybují v rozmezí od -1 do +1. Záporné hodnoty představují půdu a neživé prvky, zatímco kladné hodnoty představují zelené listy a stonky.

_Reference: Louhaichi, M., M. Borman a D. Johnson. „Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat.“ Geocarto International 16, č. 1 (2001): 65-70._

***

## GNDVI - Green Normalizovaný rozdílový vegetační index

Tento index je podobný indexu NDVI, s tím rozdílem, že měří zelené spektrum od 540 do 570 nm namísto červeného spektra. Tento index je citlivější na koncentraci chlorofylu než index NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Reference: Gitelson, A., a M. Merzlyak. „Dálkový průzkum koncentrace chlorofylu v listech vyšších rostlin.“ Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green Optimalizovaný index vegetace upravený podle půdy

Tento index byl původně navržen s použitím barevné infračervené fotografie k předpovědi požadavků kukuřice na dusík. Je podobný indexu OSAVI, ale nahrazuje zelené pásmo červeným.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Reference: Sripada, R., et al. „Stanovení sezónní potřeby dusíku pro kukuřici pomocí letecké barevné infračervené fotografie.“ Doktorská disertační práce, North Carolina State University, 2005._

***

## GRVI – Green Poměrný index vegetace

Tento index je citlivý na fotosyntetickou aktivitu v korunách lesů, protože odrazivost zelené a červené barvy je silně ovlivněna změnami pigmentů v listech.

$$
GRVI = {NIR \over Green }
$$

_Reference: Sripada, R., et al. „Letecká barevná infračervená fotografie pro stanovení požadavků na dusík v kukuřici na začátku vegetačního období.“ Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green Index vegetace upravený podle půdy

Tento index byl původně navržen s použitím barevné infračervené fotografie k předpovědi požadavků na dusík u kukuřice. Je podobný indexu SAVI, ale nahrazuje zelené pásmo červeným.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Reference: Sripada, R., et al. „Stanovení sezónní potřeby dusíku pro kukuřici pomocí letecké barevné infračervené fotografie.“ Doktorská práce, North Carolina State University, 2005._

***

## LAI – Index listové plochy

Tento index se používá k odhadu listového pokryvu a k předpovědi růstu a výnosu plodin. ENVI počítá zelený LAI pomocí následující empirické formule od Boegh et al (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Kde EVI je:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Vysoké hodnoty LAI se obvykle pohybují v rozmezí od přibližně 0 do 3,5. Pokud však scéna obsahuje mraky a jiné jasné prvky, které vytvářejí nasycené pixely, mohou hodnoty LAI překročit 3,5. Ideálně byste měli před vytvořením obrazu LAI zakrýt mraky a jasné prvky ve scéně.

_Reference: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde a A. Thomsen. „Letecké multispektrální údaje pro kvantifikaci indexu listové plochy, koncentrace dusíku a fotosyntetické účinnosti v zemědělství.“ Remote Sensing of Environment 81, č. 2-3 (2002): 179-193._

***

## LCI – Index chlorofylu listů

Tento index se používá k odhadu obsahu chlorofylu ve vyšších rostlinách, které jsou citlivé na změny odrazivosti způsobené absorpcí chlorofylu.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Reference: Datt, B. „Dálkový průzkum obsahu vody v listech eukalyptu.“ Journal of Plant Physiology 154, č. 1 (1999): 30–36._

***

## MNLI – modifikovaný nelineární index

Tento index je vylepšením nelineárního indexu (NLI), který zahrnuje index vegetace upravený podle půdy (SAVI), aby zohlednil půdní pozadí. ENVI používá hodnotu faktoru úpravy pozadí koruny stromů (_L_) 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Reference: Yang, Z., P. Willis a R. Mueller. „Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy.“ Sborník z konference Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 – modifikovaný index vegetace upravený podle půdy 2

Tento index je jednodušší verzí indexu MSAVI navrženého Qi et al (1994), který vylepšuje index vegetace upravený podle půdy (SAVI). Snižuje šum půdy a zvyšuje dynamický rozsah signálu vegetace. MSAVI2 je založen na indukční metodě, která nepoužívá konstantní hodnotu _L_ (jako u SAVI) k zvýraznění zdravé vegetace.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Reference: Qi, J., A. Chehbouni, A. Huete, Y. Kerr a S. Sorooshian. „A Modified Soil Adjusted Vegetation Index.“ Remote Sensing of Environment 48 (1994): 119-126._

***

## NDRE – Normalizovaný rozdíl RedEdge

Tento index je podobný NDVI, ale porovnává kontrast mezi NIR a RedEdge namísto Red, který často detekuje stres vegetace dříve.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – normalizovaný rozdílový index vegetace

Tento index je měřítkem zdravé, zelené vegetace. Kombinace jeho normalizované rozdílové formulace a použití oblastí s nejvyšší absorpcí a odrazivostí chlorofylu ho činí robustním v široké škále podmínek. Může však dosáhnout nasycení v podmínkách husté vegetace, když LAI dosáhne vysoké hodnoty.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Hodnota tohoto indexu se pohybuje v rozmezí od -1 do 1. Běžný rozsah pro zelenou vegetaci je 0,2 až 0,8.

_Reference: Rouse, J., R. Haas, J. Schell a D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. Třetí sympozium ERTS, NASA (1973): 309-317._

***

## NLI – nelineární index

Tento index předpokládá, že vztah mezi mnoha vegetačními indexy a biofyzikálními parametry povrchu je nelineární. Lineárizuje vztahy s povrchovými parametry, které mají tendenci být nelineární.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Reference: Goel, N. a W. Qin. „Vliv architektury korun stromů na vztahy mezi různými vegetačními indexy a LAI a Fpar: počítačová simulace.“ Remote Sensing Reviews 10 (1994): 309–347._

***

## OSAVI – Optimalizovaný index vegetace upravený podle půdy

Tento index je založen na indexu vegetace upraveném podle půdy (SAVI). Používá standardní hodnotu 0,16 pro faktor úpravy pozadí porostu. Rondeaux (1996) stanovil, že tato hodnota poskytuje větší variabilitu půdy než SAVI pro nízký vegetační pokryv, přičemž vykazuje zvýšenou citlivost na vegetační pokryv větší než 50 %. Tento index se nejlépe používá v oblastech s relativně řídkým porostem, kde je půda viditelná přes porost.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Reference: Rondeaux, G., M. Steven a F. Baret. „Optimization of Soil-Adjusted Vegetation Indices.“ Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI – Renormalizovaný rozdílový vegetační index

Tento index využívá rozdíl mezi vlnovými délkami v blízké infračervené a červené oblasti spolu s NDVI k zvýraznění zdravé vegetace. Je necitlivý na vlivy půdy a geometrie slunečního svitu.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Reference: Roujean, J., a F. Breon. „Odhad PAR absorbovaného vegetací z měření bidirekcionální odrazivosti.“ Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI – Index vegetace upravený o půdu

Tento index je podobný indexu NDVI, ale potlačuje vliv pixelů půdy. Používá faktor úpravy pozadí porostu, _L_, který je funkcí hustoty vegetace a často vyžaduje předchozí znalost množství vegetace. Huete (1988) navrhuje optimální hodnotu _L_=0,5, aby se zohlednily variace pozadí půdy prvního řádu. Tento index se nejlépe používá v oblastech s relativně řídkou vegetací, kde je půda viditelná přes porost.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Reference: Huete, A. „A Soil-Adjusted Vegetation Index (SAVI).“ Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI – transformovaný rozdílový vegetační index

Tento index je užitečný pro monitorování vegetačního pokryvu v městském prostředí. Není nasycený jako NDVI a SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Reference: Bannari, A., H. Asalhi a P. Teillet. „Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping“ (Transformovaný index rozdílu vegetace (TDVI) pro mapování vegetačního pokryvu) v Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, svazek 5 (2002)._

***

## VARI – Index odolnosti vůči atmosférickým vlivům ve viditelném spektru

Tento index je založen na indexu ARVI a používá se k odhadu podílu vegetace v oblasti s nízkou citlivostí na atmosférické vlivy.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Reference: Gitelson, A., et al. „Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – Index vegetace s širokým dynamickým rozsahem

Tento index je podobný indexu NDVI, ale používá váhový koeficient (_a_) ke snížení rozdílu mezi příspěvky signálů v blízké infračervené a červené oblasti k indexu NDVI. WDRVI je zvláště účinný v scénách se střední až vysokou hustotou vegetace, když NDVI přesahuje 0,6. NDVI má tendenci se vyrovnávat, když se zvyšuje podíl vegetace a index listové plochy (LAI), zatímco WDRVI je citlivější na širší rozsah podílů vegetace a na změny v LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Vážený koeficient (_a_) se může pohybovat v rozmezí 0,1 až 0,2. Hodnota 0,2 je doporučena Henebrym, Viñou a Gitelsonem (2004).

_Reference_

_Gitelson, A. „Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.“ Journal of Plant Physiology 161, č. 2 (2004): 165–173._

_Henebry, G., A. Viña a A. Gitelson. „Index vegetace s širokým dynamickým rozsahem a jeho potenciální využití pro analýzu mezer.“ Gap Analysis Bulletin 12: 50–56._
