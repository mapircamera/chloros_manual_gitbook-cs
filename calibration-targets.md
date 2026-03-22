---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibrační terče

Společnost MAPIR nabízí různé kalibrační terče pro širokou škálu aplikací. Níže zobrazený kompaktní model T4-R50 obsahuje 4 panely, u nichž byla změřena odrazivost světla v rozsahu 250–2 500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Difúzní referenční terče T4 mají následující křivky odrazivosti, [data ke stažení zde](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR Odrazivost T4 :: 250–2 500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR Odrazivost T4 :: 400–1 000 nm</p></figcaption></figure>Difúzní referenční terče T4P mají následující křivky odrazivosti, [data ke stažení zde](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P odrazivost :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR Odrazivost T4P :: 400–1000 nm</p></figcaption></figure>Při pohledu na graf odrazivosti vidíte, že hodnoty představují vlnovou délku (osa x) v závislosti na procentuální odrazivosti (osa y). Když pořídíme snímek kalibračního terče, vytvoříme vztah mezi hodnotou pixelu a procentuální odrazivostí v rámci spektra, na které je citlivé každé z pásem snímače kamery.

To znamená, že u každého snímku pořízeného našimi kamerami můžete použít fotografii našich kalibračních terčů, jako je [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) nebo [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), k kalibraci snímků z hlediska odrazivosti. Po kalibraci odpovídá každý pixel na snímku procentuálnímu odrazivosti.

Pokud kalibrované snímky exportujete v Chloros jako běžný formát JPG nebo TIFF, procentuální odrazivost se vypočítá vydělením hodnoty pixelu bitovou hloubkou formátu snímku. U formátu JPG tedy vydělte 255 a u formátu TIFF vydělte 65 535. Můžete také zvolit výstup ve formátu PERCENT v Chloros, a pak se každý pixel bude pohybovat v rozmezí procentuální hodnoty 0,0 až 1,0 (0 % až 100 % odrazivosti). Mějte však na paměti, že některé obrazové aplikace nemohou přijímat obrázky v procentuálním formátu (s plovoucí desetinnou čárkou) a že jsou z hlediska úložiště velké.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
