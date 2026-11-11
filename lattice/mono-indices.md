# Monochromatické kamery a vegetační indexy

## Jedna kamera = jedno pásmo

Kamera **M3M**je monochromatickou variantou modelu Bayer**M3C**: jedná se o monochromatický snímač IMX265 s jedním úzkopásmovým interferenčním filtrem. Název pásma je uveden v řetězci modelu — `M3M-<lens>-F<wavelength>`, např. `M3M-L87-F685` (v aplikaci Chloros

se zobrazuje jako `LATT-M3M-L87-F685`). Snímač poskytuje **jediné pásmo v odstínech šedé** bez Bayerovy mozaiky: není co demosaikovat, není třeba oddělovat přeslechy mezi kanály a není třeba nastavovat vyvážení bílé.

Důsledky, které je dobré znát před plánováním monochromatického systému:

* **Zářivost a odrazivost jsou plně definovány pro každé pásmo.**Jedná se o radiometrické mapy pro jednotlivá pásma, takže jedna kamera M3M produkuje kalibrovanou radianci typu float32 (W/m²/sr/nm) a odrazivost typu uint16 (`32768` = ρ 1,0) přesně stejně jako pásmo M3C. Mono snímky obsahují**identickou** matici odezvy snímače — není potřeba ani se nepoužívá žádná 3×3 separace.
* **Jedna mono kamera nemůže vypočítat vegetační index.** NDVI

, NDRE

a podobné indexy vyžadují alespoň dvě pásma. Pro výpočet indexů z mono hardwaru je třeba kombinovat několik kamer M3M — viz níže.
* Kamery M3M vysílají **Mono12** (12 bitů: 2 bajty/pixel při přenosu, nebo 1,5 bajtu/pixel, pokud pole používá [zhuštěný 12bitový přenos](arrays.md#packed-12-bit-transport)), což má význam pro [plánování šířky pásma pole](arrays.md#bandwidth-how-many-cameras-fit).

## Co Chloros

u mono přeskočí — a jak vás o tom informuje

Fáze barevného zpracování se na jednopásmový snímač prostě nevztahují. Chloros

**je přeskočí s jednovětou zprávou**, místo aby vyhodil chybu, a přesto je v téže relaci normálně spustí pro jakoukoli kameru M3C (Bayer):

| Fáze | Chování v režimu Mono (M3M) | Chování v režimu M3C |
| --- | --- | --- |
| Demosaic / debayer | Přeskočeno — výstupní úroveň `debayered` je jednokanálový obrázek ve stupních šedi. | 3kanálový demosaic. |
| Vyvážení bílé (`lattice white-balance`) | Přeskočeno s jednořádkovou zprávou. | Probíhá normálně. |
| Barevný profil (`lattice color-profile`) | Přeskočeno s jednořádkovou zprávou. | Probíhá normálně. |
| Sytost/kontrast (`lattice color`) | Přeskočeno s jednořádkovou zprávou. | Probíhá normálně. |
| Odstranění spektrálního přeslechu | Identita (bez matice 3×3). | Použita matice 3×3 pro každou kameru. |
| Radiance / odrazivost | **Probíhá** — pro každý pásmo, plně kalibrováno. | Probíhá pro každý pásmo. |

GUI používá stejné omezení: u monochromatické kamery panel nastavení pro jednotlivé kamery skryje řádky týkající se pouze RGB

(vyvážení bílé, gama, barevný profil, sytost, kontrast, rozdělení kanálů) a živý histogram je uzamčen na jedinou stopu **MONO**. Rozlišovacím znakem v celém stohu je token `M3M` v řetězci modelu, který se v grafickém uživatelském rozhraní (GUI) a SDK

u zobrazuje jako `is_mono`.

## Indexy vyžadují ≥ 2 pásma: zarovnání → stohování → indexování

Pracovní postup pro monoindex se vždy skládá ze tří stejných kroků:

1. **Zarovnání** — nasměrujte několik kamer M3M na různé vlnové délky (např. F650 „Red

“ a F850 „NIR

“), propojte je jako [multikamerové pole](arrays.md) a nechte program Chloros

vypočítat transformaci pro společnou registraci mezi kamerami.
2. **Složení** — zarovnané snímky se spojí do jednoho vícepásmového obrazu (každá kamera přispívá jedním pojmenovaným pásmem).
3. **Index** — vyhodnotí indexový vzorec pro pásma stacku, volitelně jej vykreslí pomocí LUT.

V grafickém uživatelském rozhraní představuje celý tento řetězec režim zobrazení pole **Combined Cameras**: živý kompozit je již zarovnán a kalkulátor indexu pole (níže) definuje vzorec, podle kterého se vykresluje. Zachycené exporty lze deformovat do stejného zarovnání pomocí možnosti zachycení**Aligned**.

## Kalkulátor indexu

Kalkulátor indexu vytváří indexový výraz používaný živým náhledem a exporty indexu pro jednotlivé kamery. Jedná se o jednu sdílenou plochu, kterou lze otevřít ze dvou míst v postranním panelu záložky „Kamery“:

* **Pro jednotlivé kamery**— Živý náhled → ozubené kolečko**Index** (pouze kamery RGN

/OCN

/NGB

typu Bayer; samostatná monochromatická kamera nemá ovládání indexu, protože jedno pásmo nemůže vytvořit index).
* **Pro pole**— nastavení pole → Živý náhled → ozubené kolečko**Index**. Toto je cesta pro monochromatické kamery: seznam pásem zahrnuje**všechny kamery v poli**, takže monochromatický pár zde přispívá svými dvěma pásmy.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Ovládací prvky, shora dolů:

* **Čipy pásem** („Pásma — kliknutím přidáte do výrazu“) — jedno tlačítko pro každé dostupné pásmo, označené názvem barvy + vlnovou délkou v nm (duplicitní názvy barev jsou odlišeny, např. „Barva 850“). Kliknutím vložíte token pásma na místo kurzoru. Pásma z kamer, které nedokážou generovat radianci pro jednotlivá pásma (RGB

/FRGB), jsou odfiltrována.
* **Tlačítka operátorů a funkcí** — `+ - * / ( ) ^ ,` a `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Textové pole pro výraz** — volně zadávaný vzorec; zástupný symbol zobrazuje klasickou formu NDVI

`(NIR - Red) / (NIR + Red)`. Nade ním se nachází náhled v režimu jen pro čtení, který tokenizuje pásmové čipy, čísla a označuje neznámé tokeny.
* **Banner platnosti**— šedý „Prázdné — nebude použit žádný index“; zelený „Platný výraz“; červený s konkrétní chybou analýzy (neznámé pásmo, nejednoznačné pásmo zachycené více kamerami, chybějící závorka, …); nebo oranžový, když je výraz platný, ale**konstanta** (např. `X/X` nebo jmenovatel NDVI

zadaný jako `−` namísto `+`) — konstanta namapuje celý snímek na jednu barvu.
* Samostatné žluté varování se zobrazí, pokud je použitý výraz v pořádku, ale **živý snímek je jednotný** (plochá nebo nasycená scéna) — detekce zhroucení histogramu proběhne automaticky.
* **Použít LUT**(výchozí nastavení zapnuto; vypnuto = roztažení šedé škály),**Úroveň**2/3/5/7-stop (výchozí 7-stop) a vstupy**Min / Max**po stranách lišty přechodu. Min je ve výchozím nastavení**0,2**— přiblíží barevnou rampu do rozsahu relevantního pro vegetaci, zatímco hodnoty pod touto hodnotou procházejí jako šedá stupnice; nastavte Min na −1 pro plný rozsah indexu (tlačítko**Reset** obnoví rozsah −1…+1). Max je ve výchozím nastavení 1.
* **Živý histogram** rozložení indexu — pruhy škálované podle odmocniny, jantarové čáry 2. a 98. percentilu, bílá mediánová čára a údaje o hodnotách mimo rozsah („◀ N% &lt; lo“ / „hi &lt; N% ▶“), které se nad 1 % zbarví jantarově jako signál k rozšíření okna Min/Max.
* Tlačítko **Apply** potvrdí výraz pro živý datový proud; úpravy LUT se aplikují v reálném čase bez nutnosti stisknout Apply. Je-li otevřen projekt, nastavení výrazu a LUT se uloží spolu s projektem a obnoví se při jeho opětovném otevření; bez projektu trvají pouze po dobu dané relace.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->



## Cesta v CLI



Stejný řetězec zarovnání → skládání → indexování, skriptovatelný od začátku do konce:

```bash
chloros-cli lattice align-calibrate --serials SN_RED,SN_NIR --save align.json
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`align-calibrate` používá připojené pole, které obsahuje jeho kamery, nebo je sám otevírá pro jeden běh. `index --live` vždy otevírá kamery sám (pomocí [dočasného pole](../reference/cli-reference.md#multi-camera-transient-sessions)) a nemá `--serials`: otevírá všechny kamery v síti. Uvolněte tedy všechny kamery, které Chloros

drží před ním — GUI, `cam-connect`, `array-connect` (`chloros-cli lattice array-disconnect --all`): druhé otevření zadržené kamery může zastavit stream jejího držitele nebo způsobit pád backendu. Kamera, která nedodává žádné snímky, způsobí selhání běhu a nezapíše se žádný zásobník ani index.

`--channel` mapuje symboly předvolby na názvy pásem v zásobníku. Dvě pravidla vám pomohou zabránit neúspěšnému běhu:

* **U symbolů se rozlišují velká a malá písmena** a musí se přesně shodovat s názvy kanálů předvolby — předvolby používají malá písmena (předvolby NDVI

jsou `red`, `nir`; viz `--list-presets`). `--channel red=Red_660` funguje; `--channel RED=660` selže s chybou `channel_map missing entries`.
* Strana pásma musí pojmenovat pásmo ve zarovnaném stohu (`lattice align-info --profile align.json` je uvádí). Režim offline také přijímá indexy pásem počínající od 0, např. `--channel red=0 --channel nir=1`.

`lattice index` také běží offline na uloženém zarovnaném vícepásmovém souboru TIFF

. Každý formulář běží na backendu, takže ten musí být spuštěn a přihlášen k placenému tarifu Chloros

+; relativní `--input` nebo `--output` se vyhodnocuje podle složky, ze které příkaz spouštíte:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Předvolby indexu

`lattice index --preset` (a [Index/LUT sandbox](../image-viewer-gui/index-lut-sandbox.md) na záložce Obrázek, který využívá stejný engine) obsahuje těchto **22 předvoleb**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Pro vzorce a symboly kanálů jednotlivých předvoleb spusťte `chloros-cli lattice index --list-presets` a pro dostupné barevné přechody `--list-gradients`. Vlastní vzorce používají `--formula EXPR` se stejnou syntaxí jako Kalkulátor indexů. Upozorňujeme, že tento seznam předvoleb je specifický pro indexovací engine LATTICE — rozevírací nabídka „Zpracování“ v nastavení projektu pro importované snímky obsahuje jiný seznam (viz [Vzorce multispektrálních indexů](../project-settings/multispectral-index-formulas.md)).

Kompletní sada příznaků (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, ovládací prvky pro zarovnání a deformaci u `--live` a další) je popsána v [Referenci CLI

§ Index / Vegetation Maths](../reference/cli-reference.md#index--vegetation-maths); ekvivalenty v SDK

jsou uvedeny v [SDK

Reference](../reference/sdk-reference.md).

## Zachycení indexových produktů z monokromatického pole

Po připojení pole a použití indexového výrazu funkce `array-capture` (nebo volba v grafickém rozhraní **Zachytit vše**) uloží exportní úrovně pro každou kameru *a* indexové vykreslení — `--index`/`--no-index` tuto funkci přepíná v CLI

a zachycování ve výchozím nastavení zahrnuje všechny použitelné úrovně. Příspěvek monokamery do každé skupiny záznamů tvoří její jedno pásmo na úrovních raw/debayered (stupně šedi)/radiance/reflectance, plus sdílený kompozit kombinovaného indexu, pokud pole běží v kombinovaném režimu. Viz [Systémy s více kamerami § Snímání](arrays.md#capturing-monitoring-vs-analysis).
