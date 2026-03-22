---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Vzorce pro multispektrální indexy

Níže uvedené vzorce pro indexy využívají kombinaci průměrných rozsahů propustnosti filtru Survey3:

<table><thead><tr><th align="center">Barva filtru Survey3</th><th width="196.199951171875" align="center">Survey3 Název filtru</th><th width="159.800048828125" align="center">Rozsah propustnosti (FWHM)</th><th align="center">Průměrná propustnost</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB – Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN– Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN – Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN – Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN – NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR – NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>Při použití těchto vzorců může název končit na „\_1“ nebo „\_2“, což odpovídá tomu, který filtr NIR, buď NIR1, nebo NIR2, byl použit.

***

## EVI – Vylepšený index vegetace

Tento index byl původně vyvinut pro použití s daty MODIS jako vylepšení indexu NDVI optimalizací signálu vegetace v oblastech s vysokým indexem listové plochy (LAI). Je nejužitečnější v oblastech s vysokými hodnotami LAI, kde může dojít k nasycení NDVI. Využívá oblast modré odrazivosti k korekci signálů půdního pozadí a ke snížení atmosférických vlivů, včetně rozptylu aerosolů.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Hodnoty EVI by se měly u pixelů vegetace pohybovat v rozmezí 0 až 1. Světlé prvky, jako jsou mraky a bílé budovy, spolu s tmavými prvky, jako je voda, mohou vést k anomálním hodnotám pixelů v obraze EVI. Před vytvořením obrazu EVI byste měli z obrazu odrazivosti odstranit mraky a světlé prvky a volitelně nastavit prahovou hodnotu pixelů od 0 do 1.

_Reference: Huete, A., et al. „Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices.“ Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 – Index lesního porostu 1

Tento index rozlišuje lesní porosty od jiných typů vegetace pomocí multispektrálních snímků odrazivosti, které zahrnují pásmo červeného okraje.

$$
FCI1 = Red * RedEdge
$$

Lesnaté oblasti budou mít nižší hodnoty FCI1 kvůli nižší odrazivosti stromů a přítomnosti stínů v korunách stromů.

_Odkaz: Becker, Sarah J., Craig S.T. Daughtry a Andrew L. Russ. „Robust forest cover indices for multispectral images.“ Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## FCI2 – Index lesního porostu 2

Tento index rozlišuje lesní porosty od jiných typů vegetace pomocí multispektrálních snímků odrazivosti, které neobsahují pásmo červeného okraje.

$$
FCI2 = Red * NIR
$$

Lesnaté oblasti budou mít nižší hodnoty FCI2 kvůli nižší odrazivosti stromů a přítomnosti stínů v korunách stromů.

_Odkaz: Becker, Sarah J., Craig S.T. Daughtry a Andrew L. Russ. „Robust forest cover indices for multispectral images.“ Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## GEMI – Index globálního monitorování životního prostředí

Tento nelineární index vegetace se používá pro globální monitorování životního prostředí na základě satelitních snímků a snaží se korigovat atmosférické vlivy. Je podobný indexu NDVI, ale je méně citlivý na atmosférické vlivy. Je ovlivněn holou půdou; proto se nedoporučuje používat v oblastech s řídkou nebo středně hustou vegetací.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Kde:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Odkaz: Pinty, B., a M. Verstraete. GEMI: nelineární index pro monitorování globální vegetace ze satelitů. Vegetation 101 (1992): 15-20._

***

## GARI – Green Index odolný vůči atmosférickým vlivům

Tento index je citlivější na široký rozsah koncentrací chlorofylu a méně citlivý na atmosférické vlivy než NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Konstanta gama je váhová funkce, která závisí na podmínkách aerosolů v atmosféře. ENVI používá hodnotu 1,7, což je doporučená hodnota od Gitelsona, Kaufmana a Merzylaka (1996, strana 296).

_Odkaz: Gitelson, A., Y. Kaufman a M. Merzylak. „Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS.“ Remote Sensing of Environment 58 (1996): 289–298._

***

## GCI – Green Index chlorofylu

Tento index se používá k odhadu obsahu chlorofylu v listech u široké škály rostlinných druhů.

$$
GCI = {NIR \over Green} - 1
$$

Použití širokých NIR a zelených vlnových délek umožňuje lepší odhad obsahu chlorofylu a zároveň zajišťuje vyšší citlivost a lepší poměr signálu k šumu.

_Odkaz: Gitelson, A., Y. Gritz a M. Merzlyak. „Vztahy mezi obsahem chlorofylu v listech a spektrální odrazivostí a algoritmy pro nedestruktivní hodnocení chlorofylu v listech vyšších rostlin.“ Journal of Plant Physiology 160 (2003): 271–282._

***

## GLI – Green Listový index

Tento index byl původně navržen pro použití s digitální kamerou RGB k měření pokryvu pšenice, kde se červené, zelené a modré digitální hodnoty (DN) pohybují v rozmezí 0 až 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Hodnoty GLI se pohybují v rozmezí od -1 do +1. Záporné hodnoty představují půdu a neživé prvky, zatímco kladné hodnoty představují zelené listy a stonky.

_Odkaz: Louhaichi, M., M. Borman a D. Johnson. „Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat.“ Geocarto International 16, č. 1 (2001): 65–70._

***

## GNDVI – Green Normalizovaný index vegetace

Tento index je podobný indexu NDVI s tím rozdílem, že namísto červeného spektra měří zelené spektrum v rozmezí 540 až 570 nm. Tento index je citlivější na koncentraci chlorofylu než index NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Odkaz: Gitelson, A., a M. Merzlyak. „Dálkový průzkum koncentrace chlorofylu v listech vyšších rostlin.“ Advances in Space Research 22 (1998): 689–692._

***

## GOSAVI - Green Optimalizovaný index vegetace upravený o půdní podmínky

Tento index byl původně navržen s využitím barevné infračervené fotografie k předpovědi potřeb dusíku u kukuřice. Je podobný indexu OSAVI, ale nahrazuje zelené pásmo červeným.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Odkaz: Sripada, R., et al. „Stanovení sezónní potřeby dusíku pro kukuřici pomocí letecké barevné infračervené fotografie.“ Doktorská disertační práce, North Carolina State University, 2005._

***

## GRVI – Green Poměrný vegetační index

Tento index je citlivý na fotosyntetickou aktivitu v lesním porostu, protože odrazivost zelené a červené barvy je silně ovlivněna změnami v listových pigmentech.

$$
GRVI = {NIR \over Green }
$$

_Odkaz: Sripada, R., et al. „Letecká barevná infračervená fotografie pro stanovení raných požadavků na dusík u kukuřice během vegetačního období.“ Agronomy Journal 98 (2006): 968–977._

***

## GSAVI – Green Index vegetace upravený o půdu

Tento index byl původně navržen s využitím barevné infračervené fotografie k předpovědi požadavků na dusík u kukuřice. Je podobný indexu SAVI, ale nahrazuje zelené pásmo červeným.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Odkaz: Sripada, R., et al. „Stanovení sezónní potřeby dusíku pro kukuřici pomocí letecké barevné infračervené fotografie.“ Doktorská disertační práce, North Carolina State University, 2005._

***

## LAI – Index listové plochy

Tento index se používá k odhadu listového pokryvu a k předpovědi růstu a výnosu plodin. ENVI vypočítává zelený LAI pomocí následujícího empirického vzorce od Boegha et al (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Kde EVI je:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Vysoké hodnoty LAI se obvykle pohybují v rozmezí přibližně 0 až 3,5. Pokud však scéna obsahuje mraky a jiné jasné prvky, které vytvářejí nasycené pixely, mohou hodnoty LAI překročit 3,5. Ideálně byste měli před vytvořením snímku LAI z obrazu odstranit mraky a jasné prvky.

_Reference: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde a A. Thomsen. „Letecká multispektrální data pro kvantifikaci indexu listové plochy, koncentrace dusíku a fotosyntetické účinnosti v zemědělství.“ Remote Sensing of Environment 81, č. 2–3 (2002): 179–193._

***

## LCI – Index chlorofylu v listech

Tento index se používá k odhadu obsahu chlorofylu ve vyšších rostlinách a je citlivý na změny odrazivosti způsobené absorpcí chlorofylu.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Odkaz: Datt, B. „Dálkový průzkum obsahu vody v listech eukalyptu.“ Journal of Plant Physiology 154, č. 1 (1999): 30–36._

***

## MNLI – Modifikovaný nelineární index

Tento index je vylepšením nelineárního indexu (NLI), který zahrnuje index vegetace upravený o půdu (SAVI) s cílem zohlednit půdní pozadí. ENVI používá hodnotu faktoru úpravy pozadí korunového zápoje (_L_) rovnou 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Odkaz: Yang, Z., P. Willis a R. Mueller. „Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy.“ Sborník z sympozia Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 – Modifikovaný index vegetace upravený o půdu 2

Tento index je zjednodušenou verzí indexu MSAVI navrženého Qi et al. (1994), který vylepšuje index vegetace upravený o půdu (SAVI). Snižuje půdní šum a zvyšuje dynamický rozsah signálu vegetace. MSAVI2 je založen na induktivní metodě, která nepoužívá konstantní hodnotu _L_ (jako u SAVI) k zvýraznění zdravé vegetace.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Odkaz: Qi, J., A. Chehbouni, A. Huete, Y. Kerr a S. Sorooshian. „A Modified Soil Adjusted Vegetation Index.“ Remote Sensing of Environment 48 (1994): 119–126._

***

## NDRE – Normalizovaný rozdíl RedEdge

Tento index je podobný indexu NDVI, ale porovnává kontrast mezi NIR a RedEdge namísto Red, což často umožňuje detekovat stres vegetace dříve.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – Normalizovaný rozdíl vegetačního indexu

Tento index je měřítkem zdravé, zelené vegetace. Kombinace jeho normalizovaného rozdílu a využití oblastí s nejvyšší absorpcí a odrazivostí chlorofylu zajišťuje jeho robustnost v širokém spektru podmínek. Může však dojít k jeho nasycení v podmínkách husté vegetace, když hodnota LAI stoupne.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Hodnota tohoto indexu se pohybuje v rozmezí od -1 do 1. Běžný rozsah pro zelenou vegetaci je 0,2 až 0,8.

_Odkaz: Rouse, J., R. Haas, J. Schell a D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. Third ERTS Symposium, NASA (1973): 309-317._

***

## NLI – nelineární index

Tento index předpokládá, že vztah mezi mnoha vegetačními indexy a povrchovými biofyzikálními parametry je nelineární. Linearizuje vztahy s povrchovými parametry, které mají tendenci být nelineární.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Odkaz: Goel, N. a W. Qin. „Vlivy architektury korunového zápoje na vztahy mezi různými vegetačními indexy a LAI a Fpar: počítačová simulace.“ Remote Sensing Reviews 10 (1994): 309–347._

***

## OSAVI – Optimalizovaný index vegetace upravený o půdu

Tento index vychází z indexu vegetace upraveného o půdu (SAVI). Používá standardní hodnotu 0,16 pro faktor úpravy pozadí korunového zápoje. Rondeaux (1996) zjistil, že tato hodnota poskytuje větší variabilitu půdy než SAVI u nízkého vegetačního pokryvu, přičemž vykazuje zvýšenou citlivost na vegetační pokryv větší než 50 %. Tento index se nejlépe používá v oblastech s relativně řídkým porostem, kde je půda viditelná skrz korunový zápoje.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Odkaz: Rondeaux, G., M. Steven a F. Baret. „Optimization of Soil-Adjusted Vegetation Indices.“ Remote Sensing of Environment 55 (1996): 95–107._

***

## RDVI – Renormalizovaný rozdílový vegetační index

Tento index využívá rozdíl mezi vlnovými délkami v blízké infračervené a červené oblasti spolu s indexem NDVI k zvýraznění zdravé vegetace. Je necitlivý na vlivy půdy a geometrie pozorování slunce.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Odkaz: Roujean, J., a F. Breon. „Odhad PAR absorbovaného vegetací na základě měření obousměrné odrazivosti.“ Remote Sensing of Environment 51 (1995): 375–384._

***

## SAVI – Index vegetace upravený o půdu

Tento index je podobný indexu NDVI, ale potlačuje vliv pixelů půdy. Používá korekční faktor pozadí korunového zápoje, _L_, který je funkcí hustoty vegetace a často vyžaduje předchozí znalost množství vegetace. Huete (1988) navrhuje optimální hodnotu _L_=0,5 pro zohlednění variací pozadí půdy prvního řádu. Tento index se nejlépe používá v oblastech s relativně řídkou vegetací, kde je půda viditelná skrz porost.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Reference: Huete, A. „A Soil-Adjusted Vegetation Index (SAVI).“ Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI – transformovaný rozdílový vegetační index

Tento index je užitečný pro monitorování vegetačního pokryvu v městském prostředí. Nedochází u něj k nasycení jako u indexů NDVI a SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Odkaz: Bannari, A., H. Asalhi a P. Teillet. „Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping“ In Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, svazek 5 (2002)._

***

## VARI – Index viditelného spektra odolný vůči atmosférickým vlivům

Tento index vychází z indexu ARVI a používá se k odhadu podílu vegetace ve scéně s nízkou citlivostí na atmosférické vlivy.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Odkaz: Gitelson, A., et al. „Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction.“ International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – Index vegetace s širokým dynamickým rozsahem

Tento index je podobný indexu NDVI, ale používá váhový koeficient (_a_) ke snížení rozdílu mezi příspěvky signálů v blízké infračervené a červené oblasti do indexu NDVI. WDRVI je zvláště účinný ve scénách se střední až vysokou hustotou vegetace, kdy NDVI přesahuje hodnotu 0,6. NDVI má tendenci se stabilizovat při zvyšování podílu vegetace a indexu listové plochy (LAI), zatímco WDRVI je citlivější na širší rozsah podílů vegetace a na změny v LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Váha koeficientu (_a_) se může pohybovat v rozmezí 0,1 až 0,2. Hodnotu 0,2 doporučují Henebry, Viña a Gitelson (2004).

_Odkazy_

_Gitelson, A. „Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.“ Journal of Plant Physiology 161, č. 2 (2004): 165–173._

_Henebry, G., A. Viña a A. Gitelson. „Vegetační index s širokým dynamickým rozsahem a jeho potenciální využití pro analýzu mezer.“ Gap Analysis Bulletin 12: 50–56._
