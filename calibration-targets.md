---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibrační terče

MAPIR nabízí různé kalibrační terče pro širokou škálu aplikací. Kompaktní model T4-R50, který vidíte níže, obsahuje 4 panely, u nichž byla změřena odrazivost světla v rozsahu 250–2 500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Difúzní referenční terče T4 mají následující křivky odrazivosti, [data ke stažení zde](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR Odrazivost T4 :: 250–2 500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR Odrazivost T4 :: 400–1 000 nm</p></figcaption></figure>Difúzní referenční terče T4P mají následující křivky odrazivosti, [data ke stažení zde](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P odrazivost :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P Odrazivost :: 400–1000 nm</p></figcaption></figure>Z grafu odrazivosti je patrné, že hodnoty představují vlnovou délku (osa x) v závislosti na procentuální odrazivosti (osa y). Když pořídíme snímek kalibračního terče, vytvoříme vztah mezi hodnotou pixelu a procentuální odrazivostí v rámci spektra, na které je citlivé každé z pásem snímače kamery.

To znamená, že u každého snímku pořízeného našimi kamerami můžete použít fotografii našich kalibračních terčů, jako je například [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) nebo [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), k kalibraci snímků z hlediska odrazivosti. Po kalibraci se hodnota každého pixelu na snímku rovná procentuální odrazivosti.

U výstupů **Survey3** platí, že pokud kalibrované snímky v Chloros exportujete jako běžné soubory JPG nebo TIFF, procentuální odrazivost se vypočítá vydělením hodnoty pixelu bitovou hloubkou formátu snímku. U formátu JPG tedy vydělte hodnotu pixelu číslem 255 a u formátu TIFF číslem 65 535. Můžete také zvolit výstup ve formátu PERCENT v Chloros; v takovém případě bude každý pixel mít hodnotu v rozmezí 0,0 až 1,0 (0 % až 100 % odrazivosti). Mějte však na paměti, že některé aplikace pro práci s obrázky nepodporují obrázky v procentním formátu (s plovoucí desetinnou čárkou) a z hlediska úložiště mají tyto soubory velkou velikost.

{% hint style="info" %}
**Odrazivost LATTICE používá odlišné měřítko pixelů.** Odrazivost LATTICE se ukládá s hodnotou DN 32768 = 100 % odrazivosti (nikoli 65535) a každý soubor obsahuje značku XMP `Chloros:PixelScale`, která udává jeho měřítko. Přečtěte si tento tag a vydělte jím hodnotu, místo abyste předpokládali konstantu — viz [Formáty výstupních snímků](output-image-formats.md).
{% endhint %}

## Kalibrační terče s kamerami LATTICE

U kamer LATTICE je kalibrační terč pro odrazivost **volitelný**: Chloros může místo toho vázat odrazivost na dopadající ozáření měřené světelným senzorem DAQ (ρ = π·L/E). Referenční hodnota se volí nastavením zdroje odrazivosti (Nastavení projektu v grafickém uživatelském rozhraní; `--reflectance-source` v souboru CLI; `reflectance_source` v souboru SDK):

| Hodnota | Chování |
| --- | --- |
| `auto` *(výchozí)* | Cíl v rámci snímku, který prošel kontrolou kvality (QA), je **absolutní referencí**; pokud není k dispozici žádný cíl nebo pokud kontrola kvality selže, Chloros se vrátí k dělení sestupného toku DAQ. |
| `target` | Pouze striktní cíl — žádná náhrada ze strany DAQ. |
| `daq` | Autoritativní DAQ — měření směrem dolů je vždy referencí. |

Další chování cílů pro LATTICE:

* **Geometrie cílů** — podporovány jsou panely označené ArUco, panely s pevnou oblastí zájmu (ROI) a pásové cíle; geometrie pochází z konfigurace cílů projektu.
* **Data o měřených cílech pro jednotlivé jednotky** — `--target-reflectance-dir DIR` odkazuje na adresář skenů odrazivosti měřených cílů pro jednotlivé jednotky (`<serial>.csv`, vyhledávané podle sériového čísla/QR kódu jednotky cíle). V případě neúspěchu se Chloros vrátí k nominálním spektrům T3/T4P.
* **Časové ukotvení** — detekovaný cíl kalibruje snímky v jeho okolí a je udržován mezi jednotlivými pozorováními cíle.

Úplná sémantika příznaků a příklady jsou uvedeny v [Referenci CLI](reference/cli-reference.md) (viz „Přepínače exportu pro jednotlivé produkty“).

### F988

„Reflexe F988 je kalibrována pomocí reflexního panelu v rámci scény: pásmo leží mimo kalibrovaný rozsah světelného senzoru DAQ, takže Chloros použije váš nejnovější záznam panelu a zachová jej mezi jednotlivými pozorováními panelu.“

Pokud je F988 spuštěn s kalibrací pouze pomocí DAQ, Chloros odmítne odrazivost založenou na DAQ pro dané pásmo a uvede důvod (důvod přeskočení `dls-uncalibrated-band-988`); podporovaným postupem je práce s panelem.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
