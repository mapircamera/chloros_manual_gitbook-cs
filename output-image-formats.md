---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Formáty výstupních obrázků

Chloros exportuje zpracované výstupy ve čtyřech formátech souborů. Formát vyberte v nastavení projektu (GUI) pomocí `--format` (CLI) nebo pomocí `export_format` (SDK). CLI a SDK akceptují přesně níže uvedené řetězce.

| Formátovací řetězec | Přípona | Typ pixelu | Rozsah pixelů | Poznámky |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(výchozí)* | `.tif` | uint16 digitální číslo | 0 – 65535 | Doporučeno pro fotogrammetrii / GIS. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100 % odrazivost. Některé aplikace nedokážou číst soubory TIFF s plovoucí desetinnou čárkou; soubory jsou větší. |
| `PNG (8-bit)` | `.png` | číselná hodnota typu uint8 | 0 – 255 | Bezztrátová komprese, vhodná pro prohlížení na webu a vizualizaci. |
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

Složka kamery je `LATT-<sensor>-<lens>-F<filter>` pro LATTICE a `<model>_<filter>` (např. `Survey3N_RGN`) pro Survey3. **Každý exportovaný produkt si zachovává název zdrojového souboru – produkt identifikuje složka, nikoli přípona souboru.** Úplná pravidla najdete v části [Kam se ukládají výstupy](reference/cli-reference.md) v příručce CLI.

## Produkty LATTICE (úrovně snímání a exportu)

Jeden surový snímek LATTICE se v jediném průchodu rozdělí na všechny požadované produkty. Každý typ produktu má vlastní přepínač (zaškrtávací políčka v grafickém rozhraní nebo CLI `--debayered` / `--preview` / `--radiance` / `--reflectance`, ve výchozím nastavení jsou všechny zapnuty):

| Úroveň | Obsah | Datový typ |
| --- | --- | --- |
| `raw` | Data Bayerova filtru přímo ze snímače (černobílé kamery: jedno pásmo). Zpracování vždy začíná od surových dat. | Tak, jak byla zachycena |
| `debayered` | Lineární demosaikování — 3 kanály pro M3C, 1 kanál ve stupních šedi pro M3M. | Lineární DN |
| `radiance` | Absolutní spektrální radiance z kompletního radiometrického řetězce, v **W/m²/sr/nm**. Vždy zaznamenáno jako 32bitové TIFF (`tiff32/Radiance_Images/`), bez ohledu na zvolený formát exportu. | float32 |
| `reflectance` | Odrazivost ρ, kde **DN 32768 = ρ 1,0 (100 %)** s rezervou až do ρ 2,0. Připraveno pro Pix4D. | uint16 |
| `preview` | Render připravený pro zobrazení: RGB = vyvážení bílé + gama; multispektrální = roztažení falešných barev. | 8bitové zobrazení |

## Čtení hodnot odrazivosti pixelů

Odrazivost je uložena jako celé digitální číslo a **digitální číslo (DN), které odpovídá ρ = 1,0 (100 % odrazivosti), závisí na zdrojové kameře**:

| Zdrojová kamera | ρ = 1,0 je DN | Jak zjistit |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (rozsah až do ρ 2,0) | Do souboru je vložen XMP tag `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (omezeno na ρ 1,0) | Žádné značky XMP `Chloros:*` — tato absence je signálem. |

**Přečtěte si značku XMP `Chloros:PixelScale` a vydělte tím**, místo abyste předpokládali konstantu. Značka je definována v doméně uint16, takže zůstává `32768` napříč výstupními formáty, které mění měřítko — nejprve normalizujte uložený datový typ zpět na uint16 (×257 z 8 bitů, ×65535 z float32).

{% hint style="warning" %}
**Jeden případ záměrně neobsahuje žádné měřítko.** Když se záznam z 8bitového zdroje (BayerRG8) zapíše jako 8bitový TIFF, potrubí ořízne hodnoty na rozsah 0–255 namísto přepočítání, takže soubor není popsán žádným měřítkem — Chloros zde záměrně vynechává `Chloros:PixelScale`. Pokud v souboru odrazivosti LATTICE tento tag chybí, nepředpokládejte žádnou měřítkovou hodnotu; místo toho soubor znovu exportujte v 16bitovém nebo 32bitovém formátu.
{% endhint %}

Úplná pravidla (včetně značek kompatibilních s MicaSense) najdete v části **„Čtení pixelů odrazivosti“** v [CLI Referenci](reference/cli-reference.md).
