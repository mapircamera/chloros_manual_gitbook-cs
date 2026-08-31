# Mono kamery a vegetační indexy

## Jedna kamera = jedno pásmo

Kamera **M3M**je monochromatickou variantou modelu Bayer**M3C**: jedná se o monochromatický snímač IMX265 s jedním úzkopásmovým interferenčním filtrem. Název pásma je uveden v řetězci modelu — `M3M-<lens>-F<wavelength>`, např. `M3M-L87-F685` (zobrazeno v Chloros jako `LATT-M3M-L87-F685`). Snímač poskytuje**jediný pásmo ve stupních šedi** bez Bayerovy mozaiky: není co demosaikovat, není třeba oddělovat přeslechy mezi kanály a není třeba nastavovat vyvážení bílé.

Důsledky, které je třeba znát před plánováním monochromatického systému:

* **Zářivost a odrazivost jsou plně definovány pro každý pásmo.**Jedná se o radiometrické mapy pro jednotlivá pásma, takže jedna kamera M3M produkuje kalibrovanou radianci typu float32 (W/m²/sr/nm) a odrazivost typu uint16 (`32768` = ρ 1,0) přesně stejně jako pásmo M3C. Mono snímky obsahují**identickou** matici odezvy snímače — není třeba provádět ani aplikovat žádné 3×3 rozložení.
* **Jedna monochromatická kamera nemůže vypočítat vegetační index.** NDVI, NDRE a podobné indexy vyžadují alespoň dvě pásma. Pro výpočet indexů z monochromatického hardwaru je třeba kombinovat několik kamer M3M — viz níže.
* Kamery M3M streamují **Mono12** (12 bitů, 2 bajty/pixel v přenosu), což je důležité pro [plánování šířky pásma pole](arrays.md#bandwidth-the-rules-of-thumb).

## Co Chloros u mono přeskočí — a jak vám to oznámí

Fáze barevného zpracování se na jednopásmový snímač prostě nevztahují. Chloros **je přeskočí s jednovětou zprávou** namísto hlášení chyby a stále je normálně provádí pro jakoukoli kameru M3C (Bayer) ve stejné relaci:

| Fáze | Chování v režimu Mono (M3M) | Chování v režimu M3C |
| --- | --- | --- |
| Demosaik / debayer | Přeskočeno — výstupní formát `debayered` je jednokanálový obrázek ve stupních šedi. | 3kanálový demosaik. |
| Vyvážení bílé (`lattice white-balance`) | Přeskočeno s jednořádkovou zprávou. | Probíhá normálně. |
| Barevný profil (`lattice color-profile`) | Přeskočeno s jednořádkovou zprávou. | Probíhá normálně. |
| Sytost/kontrast (`lattice color`) | Přeskočeno s jednořádkovou zprávou. | Probíhá normálně. |
| Odstranění spektrálního přeslechu | Identita (bez matice 3×3). | Použita matice 3×3 pro každou kameru. |
| Radiance / odrazivost | **Probíhá** — pro každý pásmo, plně kalibrováno. | Probíhá pro každý pásmo. |

GUI používá stejné omezení: u monochromatické kamery panel nastavení pro jednotlivé kamery skryje řádky určené pouze pro RGB (vyvážení bílé, gama, barevný profil, sytost, kontrast, rozdělení kanálů), a živý histogram je uzamčen na jedinou stopu **MONO**. Rozlišovacím znakem v celém stohu je token `M3M` v řetězci modelu, který se v grafickém uživatelském rozhraní zobrazuje jako SDK.

## Indexy vyžadují ≥ 2 pásma: zarovnání → snímky → index

Pracovní postup pro monoindexování se vždy skládá ze tří kroků:

1. **Zarovnání** — nasměrujte několik kamer M3M na různé vlnové délky (např. kameru F650 „Red“ a kameru F850 „NIR“), propojte je do [multikamerového pole](arrays.md) a nechte Chloros vypočítat deformaci pro společnou registraci mezi kamerami.
2. **Stack** — zarovnané snímky se spojí do jednoho vícepásmového obrazu (každá kamera přispívá jedním pojmenovaným pásmem).
3. **Index** — vyhodnoťte indexový vzorec nad pásmy stacku, případně jej vykreslete pomocí LUT.

V grafickém uživatelském rozhraní představuje celý tento řetězec režim zobrazení pole **Combined Cameras**: živý kompozitní obraz je již zarovnán a kalkulátor indexu pole (níže) definuje vzorec, podle kterého se vykresluje. Exportované snímky lze pomocí možnosti**Aligned** při pořizování snímků deformovat do stejného zarovnání.

## Kalkulátor indexu

Kalkulátor indexu vytváří indexový výraz používaný živým náhledem a exporty indexu pro jednotlivé kamery. Jedná se o jednu sdílenou plochu, kterou lze otevřít ze dvou míst v postranním panelu záložky „Kamery“:

* **Pro jednotlivé kamery**— Živý náhled → ozubené kolečko**Index** (pouze kamery RGN/OCN/NGB typu Bayer; samostatná monochromatická kamera nemá ovládání indexu, protože jedno pásmo nemůže vytvořit index).
* **Pro pole**— nastavení pole → Živý náhled → ozubené kolečko**Index**. Toto je cesta pro monochromatické kamery: seznam pásem zahrnuje**všechny kamery v poli**, takže monochromatický pár zde přispívá svými dvěma pásmy.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Ovládací prvky, shora dolů:

* **Čipy pásem** („Pásma — kliknutím přidáte do výrazu“) — jedno tlačítko pro každé dostupné pásmo, označené názvem barvy + vlnovou délkou v nm (duplicitní názvy barev se rozlišují, např. „Barva 850“). Kliknutím vložíte token pásma na místo kurzoru. Pásma z kamer, které nemohou produkovat radianci pro jednotlivá pásma (RGB/FRGB), jsou odfiltrována.
* **Tlačítka operátorů a funkcí** — `+ - * / ( ) ^ ,` a `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Textové pole pro výraz** — volně zadávaný vzorec; zástupný symbol zobrazuje klasický tvar NDVI ve formátu `(NIR - Red) / (NIR + Red)`. Nade ním se nachází tokenizovaný náhled pouze pro čtení, který vykresluje pásmové čipy, čísla a příznaky jako neznámé tokeny.
* **Banner platnosti**— šedý „Prázdné — nebude použit žádný index“; zelený „Platný výraz“; červený s konkrétní chybou analýzy (neznámý pás, nejednoznačný pás zachycený více kamerami, chybějící závorka, …); nebo oranžový, pokud je výraz platný, ale**konstantní** (např. `X/X`, nebo jmenovatel NDVI zadaný jako `−` místo `+`) — konstanta namapuje celý snímek na jednu barvu.
* Samostatné žluté varování se objeví, pokud je použitý výraz v pořádku, ale **živý snímek je jednolitý** (plochá nebo nasycená scéna) — kolaps histogramu je detekován automaticky.
* **Použít LUT**(výchozí nastavení zapnuto; vypnuto = roztažení šedé škály),**Úroveň**2/3/5/7-stop (výchozí 7-stop) a vstupy**Min / Max**po stranách lišty přechodu. Min je ve výchozím nastavení**0,2**— přibližuje barevnou rampu do rozsahu relevantního pro vegetaci, zatímco hodnoty pod touto hodnotou procházejí jako šedá stupnice; nastavte Min na −1 pro plný rozsah indexu (tlačítko**Reset** obnoví rozsah −1…+1). Max je ve výchozím nastavení 1.
* **Živý histogram** rozložení indexu — sloupce v kvadratické škále, jantarové čáry 2. a 98. percentilu, bílá mediánová čára a údaje o hodnotách mimo rozsah („◀ N% &lt; lo“ / „hi &lt; N% ▶“), které se nad 1 % zbarví jantarově jako signál k rozšíření okna Min/Max.
* Tlačítko **Apply**aplikuje výraz na živý proud dat; úpravy LUT se provádějí v reálném čase bez nutnosti stisknout Apply. Výrazy jsou záměrně**určeny pouze pro danou relaci** — mezi relacemi se neuchovávají.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->

## Cesta CLI

Stejný řetězec zarovnání → zásobník → index, skriptovatelný od začátku do konce:

```bash
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` mapuje symboly předvolby na názvy pásem v zásobníku. Dvě pravidla vám ušetří neúspěšný běh:

* **U symbolů se rozlišují velká a malá písmena** a musí přesně odpovídat názvům kanálů v předvolbě — předvolby používají malá písmena (NDVI jsou `red`,`nir`; zkontrolujte `--list-presets`). `--channel red=Red_660` funguje; `--channel RED=660` selže s chybou `channel_map missing entries`.
* Strana pásma musí pojmenovat pásmo ve zarovnaném zásobníku (`lattice align-info --profile align.json` je uvádí). Režim offline také přijímá indexy pásem počínaje od 0, např. `--channel red=0 --channel nir=1`.

`lattice index` také běží zcela offline s uloženým zarovnaným vícepásmovým TIFF:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Předvolby indexů

`lattice index --preset` (a [Index/LUT sandbox](../image-viewer-gui/index-lut-sandbox.md) na záložce Obrázek, který používá stejný engine) obsahuje těchto **22 předvoleb**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Pro vzorce a symboly kanálů jednotlivých předvoleb použijte `chloros-cli lattice index --list-presets` a pro dostupné barevné přechody použijte `--list-gradients`. Vlastní vzorce používají `--formula EXPR` se stejnou syntaxí jako Kalkulátor indexů. Upozorňujeme, že tento seznam předvoleb je specifický pro indexovací modul LATTICE — rozevírací nabídka „Zpracování“ v nastavení projektu pro importované snímky obsahuje jiný seznam (viz [Vzorce multispektrálních indexů](../project-settings/multispectral-index-formulas.md)).

Kompletní sada příznaků (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, ovládací prvky pro zarovnání a deformaci u `--live`, a další) je popsána v [Referenci k CLI § Index / Matematika vegetace](../reference/cli-reference.md#index--vegetation-maths); ekvivalenty k SDK jsou uvedeny v [SDK Reference](../reference/sdk-reference.md).

## Zachycení indexových produktů z mono pole

Po připojení pole a použití indexového výrazu uloží `array-capture` (nebo funkce **Zachytit vše** v grafickém rozhraní) úrovně exportu pro každou kameru *a* indexové vykreslení — `--index`/`--no-index` tuto funkci přepíná na CLI a zachycování ve výchozím nastavení zahrnuje všechny použitelné úrovně. Příspěvek monokamery do každé skupiny snímání tvoří její jedno pásmo na úrovních raw/debayered (šedá stupnice)/radiance/reflectance, plus sdílený kompozit kombinovaného indexu, pokud pole běží v kombinovaném režimu. Viz [Systémy s více kamerami § Snímání](arrays.md#capturing-monitoring-vs-analysis).
