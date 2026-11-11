---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Formáty výstupních obrázků

Chloros exportuje zpracované produkty ve čtyřech formátech souborů. Formát vyberte v nastavení projektu (GUI), pomocí `--format` (CLI) nebo pomocí `export_format` (SDK). CLI a SDK akceptují přesně níže uvedené řetězce.

| Formátovací řetězec | Přípona | Typ pixelu | Rozsah pixelů | Poznámky |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(výchozí)* | `.tif` | číselná hodnota typu uint16 | 0 – 65535 | Doporučeno pro fotogrammetrii / GIS. |
| `TIFF (8-bit)` | `.tif` | uint8 – číselná hodnota | 0 – 255 | Pro všechny 8bitové snímky, které nabízejí pouze 8bitové formáty: Chloros neprovádí upscaling bitové hloubky. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100 % odrazivost. Některé aplikace nedokážou číst soubory TIFF s plovoucí desetinnou čárkou; soubory jsou větší. |
| `PNG (8-bit)` | `.png` | uint8 číselná hodnota | 0 – 255 | Bezztrátová komprese, vhodná pro prohlížení na webu a vizualizaci. |
| `JPG (8-bit)` | `.jpg` | číselná hodnota typu uint8 | 0 – 255 | Komprese se ztrátou, nejmenší soubory. |

## Umístění výstupních souborů

Výstupy se ukládají do projektové složky, seskupené podle kamery a následně podle formátu souboru:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera (model+lens+filter)
    ├── tiff16/                          # follows --format: tiff16, tiff8, png8, jpg8, or tiff32
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one <INDEX>_Index_Images/ folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Složka pro kameru LATTICE je `LATT-<sensor>-<lens>-F<filter>` a pro kameru Survey3 je to `<model>_<filter>` (např. `Survey3N_RGN`). **Každý exportovaný produkt si zachovává název zdrojového souboru — produkt identifikuje složka, nikoli přípona souboru.** Úplná pravidla najdete v části [Umístění výstupů](reference/cli-reference.md) v referenční příručce k CLI.

## Produkty LATTICE (úrovně snímání a exportu)

Jeden surový snímek LATTICE se v jediném průchodu rozdělí na všechny požadované produkty. Každý typ produktu má vlastní přepínač (zaškrtávací políčka v grafickém rozhraní nebo příkazy CLI `--debayered` / `--preview` / `--radiance` / `--reflectance`, ve výchozím nastavení jsou všechny zapnuté):

| Úroveň | Obsah | Datový typ |
| --- | --- | --- |
| `raw` | Data Bayer přímo ze snímače (černobílé kamery: jedno pásmo). Zpracování vždy začíná od surových dat. | Tak, jak byla zachycena |
| `debayered` | Lineární demosaikování — 3 kanály pro M3C, 1 kanál ve stupních šedi pro M3M: odstraněn tmavý offset z kalibračního balíčku, následně demosaikování s ohledem na hrany, bez korekce linearity, vyrovnání pole ani oddělení barev. Export `process` zapíše stejný soubor, jaký se na této úrovni ukládá při pořízení snímku, a to v každém režimu zpracování. | Lineární DN |
| `radiance` | Absolutní spektrální radiance z úplného radiometrického řetězce, v **W/m²/sr/nm**. Vždy se ukládá jako 32bitový soubor TIFF (`tiff32/Radiance_Images/`), bez ohledu na zvolený formát exportu. | float32 |
| `reflectance` | Odrazivost ρ, kde **DN 32768 = ρ 1,0 (100 %)** s rezervou až do ρ 2,0. Připraveno pro Pix4D. | uint16 |
| `preview` | Render připravený k zobrazení: RGB = vyvážení bílé + gama; multispektrální = roztažení falešných barev. | 8bitové zobrazení |

### Orientace

Soubory `raw` a `debayered` vždy zachovávají **nativní** orientaci snímače; nastavení převrácení/otočení fotoaparátu se aplikuje pouze na ostatní produkty (náhled, radiance, odrazivost, index). Při snímání pole zůstávají soubory všech členů v nativní orientaci, pokud nepožádáte o změnu orientace (`--orient` v CLI, `orient=True` v SDK). Každý soubor zaznamenává orientaci příslušnou pro své pixely v XMP — `ChlorosRotation`, `ChlorosFlipHorizontal`, `ChlorosFlipVertical` — a `ChlorosOrientationApplied` udává, zda ji nesou jeho vlastní pixely (`false` u souborů ve formátu RAW a po odstranění bayerského filtru). Prohlížeč obrázků Chlorosu upravuje soubory ve formátu RAW a soubory po odstranění bayesovského kódu podle tohoto záznamu pouze pro účely zobrazení; samotné soubory nejsou nikdy přepsány. Soubor bez tohoto záznamu (vytvořený před verzí 1.3.0 nebo jiným nástrojem) se zobrazí tak, jak byl uložen.

## Omezovací masky

Produkty radiance a odrazivosti z multispektrální kamery M3C procházejí demixováním v pásmech 3×3. Pokud demixování v daném pásmu vygeneruje zápornou hodnotu, je tato hodnota oříznuta na nulu (živé náhledy se ořezávají přesně stejným způsobem). Novinkou ve verzi 1.3.0 je, že produkt, ve kterém byl oříznut alespoň jeden pixel, získává doprovodnou **masku omezování**, která přesně udává, o které pixely a pásma se jedná:

* **Kde:**ve složce `clamp_masks/`**vedle** složek s produkty, nikdy uvnitř žádné z nich: `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif` a `tiff32/Radiance_Images/x.tif` → `tiff32/clamp_masks/Radiance_Images/x_clampmask.tif`.
* **Co:** jednopásmový soubor TIFF `uint8` na vlastní pixelové mřížce produktu (včetně orientace, oblasti zájmu (ROI), binningu a zarovnání pole). Bit 0 (`mask & 1`) je nastaven tam, kde bylo oříznuto pásmo v kanálu R produktu, bit 1 (`& 2`) kanál G, bit 2 (`& 4`) kanál B. Podíl oříznutých pixelů v jednotlivých pásmech je rovněž zaznamenán v produktu jako `Chloros:BandRUnmixClampFraction`, `Chloros:BandGUnmixClampFraction` a `Chloros:BandBUnmixClampFraction`.
* **Jak je to uvedeno v produktu:** XMP tag `Chloros:UnmixClampMask` obsahuje cestu k masce relativně ke složce produktu (`../clamp_masks/<product folder>/<name>_clampmask.tif`, vždy oddělené znakem `/`), `none`, pokud nedošlo k oříznutí žádného pixelu (maska se nezapíše), nebo `unavailable`, pokud nebylo možné masku zapsat. U produktu bez značky se nikdy neprovedlo oddělení barev (kamery RGB a monochromatické, úrovně RAW a po odstranění Bayerova filtru).

Protože žádná složka produktu neobsahuje masku, můžete složku produktu, jako je například `Reflectance_Calibrated_Images/`, předat do Pix4D, Metashape nebo jiného fotogrammetrického softwaru tak, jak je, včetně podsložek — předávejte složku produktu, nikoli nadřazenou složku formátu. Vlastní import Chloros masky omezující rozsah vynechává. CLI je počítá samostatně: `Image products written: N (plus M clamp masks)`.

## Indexové produkty

Každý index se ukládá do vlastní složky `<INDEX>_Index_Images/` (např. `NDVI_Index_Images/`).

* **Celé formáty**(`TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`) ukládají**lineární roztažení pro každý snímek**, nikoli mapu −1…1: DN 0 je nejnižší indexová hodnota daného snímku a plný rozsah je jeho nejvyšší hodnota. Soubor obsahuje `Chloros:IndexScaling=linear_minmax`, `Chloros:IndexMin` a `Chloros:IndexMax`, tedy i `value = IndexMin + DN / full_scale × (IndexMax − IndexMin)`. Protože každý snímek má své vlastní koncové body, porovnávejte snímky podle hodnot, nikoli podle DN.
* **`TIFF (32-bit, Percent)`** ukládá samotné indexové hodnoty jako float32 bez přepočítání měřítka — nejlepší volba pro kvantitativní analýzu v rámci celého letu.

Novinkou ve verzi 1.3.0 je, že indexový produkt popisuje **sám sebe**, místo aby opakoval tagy zdrojového surového datového souboru. Jeho GPS a kamerové tagy se kopírují z produktu, ze kterého byl vypočítán (z produktu odrazivosti, pokud existuje), a poté:

| Tag | Hodnota |
| --- | --- |
| `Chloros:ProcessingLevel` | `index` (soubor tedy není nikdy znovu importován jako záznam) |
| `Chloros:IndexApplied` / `Chloros:IndexExpression` | `True` / použitý vzorec |
| `Chloros:IndexLevel` | `reflectance` nebo `radiance` — z čeho byl index vypočítán (chybí, pokud neplatí ani jedno) |
| `Chloros:IlluminationInvariant` | `True` pro index vypočítaný z odrazivosti, `False` z radiance nebo hodnot snímače |
| `Chloros:ChlorosPipelineVersion` | verze programu Chloros, která index vytvořila |

`Chloros:PixelScale` je z indexových produktů odstraněn — použijte výše uvedené `IndexMin` / `IndexMax`. Kompletní postup zpětného načtení najdete v části „Celé číslo indexového rastru představuje kontrastní roztažení pro každý snímek“ v [Referenci CLI](reference/cli-reference.md).

## Čtení hodnot odrazivosti pixelů

Odrazivost je uložena jako celočíselná digitální hodnota a **DN, která odpovídá ρ = 1,0 (100 % odrazivost), závisí na zdrojové kameře**:

| Zdrojová kamera | ρ = 1,0 je DN | Jak zjistit |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (rozsah až do ρ 2,0) | Soubor je opatřen značkou XMP `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (omezeno na ρ 1,0) | Žádný tag `Chloros:PixelScale` — tato absence je signálem. (Produkty Survey3 obsahují další značky zpracování `Chloros:*`, jako je například `Chloros:CalibrationApplied`, a to ve všech režimech zpracování od verze 1.3.0.) |

**Načtěte značku XMP `Chloros:PixelScale` a vydělte jí**, místo abyste předpokládali konstantu. Značka je definována v doméně uint16, takže zůstává `32768` napříč výstupními formáty, které provádějí přepočítání — nejprve normalizujte uložený datový typ zpět na uint16 (×257 z 8-bit, ×65535 z float32).

{% hint style="warning" %}
**8bitové snímky.** Výstupy z 8bitového snímku jsou 8bitové. Jeho odrazivost je přepočítána a označena jako `Chloros:PixelScale=32768`, takže měřítko zůstává zachováno; 8bitové produkty DN a soubory odrazivosti vytvořené před verzí 1.3.0 jsou oříznuty na rozsah 0–255 a neobsahují žádný tag měřítka. Pokud v souboru odrazivosti LATTICE tento tag chybí, nepředpokládejte žádné měřítko; místo toho proveďte reexport v 16bitovém nebo 32bitovém formátu.
{% endhint %}

Úplná pravidla (včetně značek kompatibilních s MicaSense) najdete v části **„Čtení pixelů odrazivosti“** v [Referenci CLI](reference/cli-reference.md).
