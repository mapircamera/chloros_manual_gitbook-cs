---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/supported-cameras
---

# Podporované kamery

Chloros zpracovává snímky ze dvou řad kamer MAPIR na **všech platformách** (Windows, Linux amd64 a Linux arm64/Jetson):

* **Survey3** — kamery Survey3W (širokoúhlé) a Survey3N (úzké). Vstup: `RAW+JPG`.
* **LATTICE**— multispektrální kamerové moduly M3C a M3M. Vstup: snímky z kamer `.tif`/`.tiff`. Kamery LATTICE lze také**ovládat v reálném čase** z Chloros — prostřednictvím záložky „Kamery“ v grafickém uživatelském rozhraní (Windows) nebo `chloros-cli lattice` / Python SDK (Windows a Linux) — včetně synchronizovaných soustav více kamer. Viz [příručka LATTICE](lattice/).

Zpracovací pipeline také přijímá vstupní soubory formátu `.dng`.

## Survey3

<table data-header-hidden><thead><tr><th width="156">Výrobce</th><th width="250">Model kamery</th><th width="138">Model filtru</th><th width="187">Typ snímku</th></tr></thead><tbody><tr><td><strong>Výrobce</strong></td><td><strong>Model fotoaparátu</strong></td><td><strong>Model filtru</strong></td><td><strong>Typ snímku</strong></td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>OCN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RE</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NIR</td><td>RAW+JPG, JPG</td></tr></tbody></table>## LATTICE

Řada LATTICE je modulární multispektrální kamerový systém postavený na snímači Sony IMX265 s globální závěrkou (3,1 MP, pixely 3,45 µm). Každá kamera ukládá svou identitu jako řetězec modelu:

```
<sensor>-<lens>-F<filter>        e.g.  M3C-L41-FRGN,  M3M-L87-F550
```

Chloros ji zobrazuje s předponou `LATT-` (například `LATT-M3M-L41-F550`) a tento řetězec modelu řídí vše, co následuje – profil snímače, rozložení pásem a kalibrace se řeší automaticky; není třeba nic konfigurovat pro jednotlivé kamery. Číslo objektivu udává **horizontální zorné pole ve stupních**: `L41` = úzký 41°, `L87` = široký 87°.

Existují dvě konfigurace snímače:

| Konfigurace | Snímač      | Typ filtru                           | Pásma na kameru                                                        |
| ------------- | ----------- | ------------------------------------- | ----------------------------------------------------------------------- |
| **M3C**       | Bayerova barevná | Trojité pásmové                       | 3 spektrální pásma z jedné expozice                                 |
| **M3M**       | Monochromatický  | Jeden úzkopásmový interferenční filtr | 1 kalibrované pásmo — pro výpočet vegetačních indexů kombinujte více kamer M3M |

### Možnosti filtrů M3C (Bayer)

| Filtr | Pásma (název @ středová vlnová délka v nm / FWHM v nm)       |
| ------ | ---------------------------------------- |
| `FRGB` | Blue 475/30 · Green 550/30 · Red 625/30  |
| `FRGN` | Red 660/21 · Green 550/30 · NIR 850/30   |
| `FOCN` | Orange 615/21 · Cyan 490/38 · NIR 808/14 |
| `FNGB` | Blue 475/30 · Green 550/30 · NIR 850/30  |

### Katalog filtrů M3M (mono) — 23 položek

Číslo F je označení SKU; naměřené pásmo (vyražené na každém kalibrovaném výstupu) je sken filtru pro danou šarži:

| SKU    | Střed (nm, naměřeno) | FWHM okraje (nm) | Šířka (nm) |
| ------ | --------------------- | --------------- | ---------- |
| F385   | 379,4                 | 367–392         | 25         |
| F405   | 403,9                 | 390–417         | 27         |
| F450   | 443,7                 | 430–458         | 28         |
| F485   | 489,7                 | 478–502         | 24         |
| F520   | 519,9                 | 504–536         | 32         |
| F550   | 548,4                 | 531–566         | 35         |
| F590   | 589,0                 | 570–608         | 38         |
| F615   | 623,8                 | 614–634         | 20         |
| F632   | 633,4                 | 616–651         | 35         |
| F650   | 651,1                 | 636–666         | 30         |
| F685   | 686,2                 | 675–698         | 23         |
| F715   | — (nominální)           | 706–724         | 18         |
| F725   | 725.2                 | 712–738         | 26         |
| F750   | 746.0                 | 729–763         | 34         |
| F780   | 775.1                 | 754–796         | 42         |
| F808   | 810.3                 | 789–832         | 43         |
| F832   | 826,1                 | 810–843         | 33         |
| F850   | 846,5                 | 828–865         | 37         |
| F880   | — (nominální)           | 867–893         | 26         |
| F905   | — (nominální)           | 892–920         | 28         |
| F940   | 940,6                 | 923–958         | 35         |
| F950   | 945,1                 | 929–961         | 32         |
| F988 † | 985,3                 | 968–1003        | 35         |

_„Hranice pásma jsou měřeny jako hodnoty plné šířky v polovině maxima z filtrovaných skenů MAPIR pro každou šarži — stejné hodnoty, které Chloros vkládá do každého kalibrovaného exportu.“_ „— (nominální)“ = zatím není k dispozici sken dané šarže; u těchto SKU je jako střed uvedeno číslo SKU a šířka odpovídá údajům výrobce.

† „Reflexe F988 je kalibrována pomocí reflexního panelu v rámci scény: pásmo leží mimo kalibrovaný rozsah světelného senzoru DAQ, takže Chloros použije váš nejnovější záznam panelu a uchová jej mezi jednotlivými měřeními panelu.“ Viz [Kalibrační terče](calibration-targets.md).

Informace o živém ovládání kamery, maticích, nastavení sítě a radiometrickém zpracovatelském řetězci najdete v [příručce LATTICE](lattice/).
