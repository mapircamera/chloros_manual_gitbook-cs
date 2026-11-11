# Kamery LATTICE

LATTICE je modulární multispektrální kamerový systém společnosti MAPIR určený pro zemědělské a vědecké snímkování. Každá kamera LATTICE je postavena na snímači Sony IMX265 s globální závěrkou (**3,1 MP, pixely 3,45 µm**) a připojuje se přes Ethernet jako zařízení**GigE Vision**.

Chloros 1.3.0 ovládá kamery LATTICE v reálném čase – vyhledávání, živý náhled, snímání a synchronizované sestavy více kamer – ze tří rozhraní:

| Rozhraní    | Kde                                                          | Platformy                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| Grafické rozhraní | Záložka **Kamery** v postranním panelu Chloros                         | Windows 10/11 x64                                        |
| CLI        | Rodina příkazů `chloros-cli lattice`                           | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **Hledáte hardware?**Kamerové moduly, objektivy, filtry a pásma, rámečky a upevnění, kabely, PoE a spouštěcí zapojení jsou popsány v [**uživatelské příručce LATTICE**](https://mapir.gitbook.io/lattice-camera). Tato kapitola se zabývá ovládáním kamer z Chlorosu.

Záznamy LATTICE jsou standardní soubory typu `.tif`/`.tiff` a Chloros je vždy zpracovává přímo z nezpracovaného záznamu. Kompletní přehled příkazů a rozhraní API najdete v [Referenci CLI](../reference/cli-reference.md) a [Referenci SDK](../reference/sdk-reference.md).

## Dvě konfigurace senzorů

| Konfigurace | Senzor       | Filtr                                | Co poskytuje jedna kamera                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Bayerova barevná | trojitý pásmový filtr                |**Tři kalibrovaná pásma z jedné expozice**                 |
| **M3M**| Monochromatická   | jeden úzkopásmový interferenční filtr |**Jedno kalibrované pásmo**; pro výpočet indexů kombinujte více kamer M3M |

Protože kamera M3M je za jediným filtrem monochromatická, každé pásmo má svou vlastní expozici. Kamera M3C pokrývá všechna tři svá pásma jednou expozicí snímače.

## Řetězce modelů a pojmenování

Každá kamera ukládá svou identitu v GenICam `DeviceUserID` jako řetězec modelu:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros jej zobrazuje s předponou `LATT-` (například `LATT-M3M-L87-F450`). Stejný řetězec `LATT-…` se zapisuje do EXIF tagu `Model` při každém exportu a používá se jako název výstupní složky fotoaparátu ve zpracovaných projektech.

| Komponenta | Hodnoty                                                   | Význam                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Snímač    | `M3C` / `M3M`                                            | Bayerova barevná matice / černobílý                                                                          |
| Objektiv      | `L41` / `L87`                                            | Číslo udává **horizontální zorné pole ve stupních**: L41 = úzký (41°), L87 = široký (87°)    |
| Filtr    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) nebo `F<nm>` (M3M) | Viz [Filtry a spektrální pásma](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

Řetězec modelu určuje vše, co následuje: Chloros určuje profil snímače, rozložení pásem a tovární kalibraci na základě `DeviceUserID` + `DeviceSerialNumber`. U jednotlivých kamer není třeba nic konfigurovat — viz [Připojení kamer](connecting.md).

## Filtry a spektrální pásma

Středové hodnoty pásem, šířky FWHM a kompletní katalog M3M s 23 položkami jsou specifikacemi produktu, a proto jsou uvedeny v příručce k hardwaru: [**Filtry a spektrální pásma**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Co je důležité ze strany softwaru: kód filtru v řetězci modelu určuje, které produkty může Chloros vytvořit. Kamery s RGB filtry (`FRGB`) poskytují pouze produkty po odstranění bayesovského vzoru a náhledy — radiance a odrazivost pro jednotlivá pásma nemají u širokopásmového senzoru smysl, proto je Chloros vynechává a tuto skutečnost uvádí. Každý jiný filtr poskytuje kompletní řetězec radiance → odrazivost → index.

## Radiometrická kalibrace v kostce

Každá kamera LATTICE je individuálně kalibrována ve výrobním závodě podle řetězce zpětně sledovatelného k NIST a je dodávána s certifikátem pro každou kameru. Co tato kalibrace zahrnuje, jak se provádí a jakou přesnost můžete uvádět, najdete v příručce k hardwaru: [**Tovární radiometrická kalibrace**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

Z hlediska softwaru je důležité, že Chloros při připojení kamery určí správnou kalibraci a použité koeficienty zafixuje do každého exportu — viz [Připojení kamer](connecting.md).

## V této kapitole

* [Připojení kamer](connecting.md) — automatické rozpoznání, dialogové okno pro připojení v grafickém uživatelském rozhraní, ekvivalenty v CLI a SDK a způsob určení tovární kalibrace (balíček v kameře vs. cloud) při připojení kamery.
* [Nastavení kamer](camera-settings.md) — všechny ovládací prvky na kartě Kamery: seznam kamer, panel nastavení pro jednotlivé kamery a panel nastavení pole (včetně editoru ROI a zarovnání).
* [Nastavení a režimy snímání](capture.md) — Snímání všeho, jednotlivé / sériové / intervalové snímání, typy exportu a struktura složky se snímky.
* [Sestavy více kamer](arrays.md) — průvodce připojením sestavy, sonda synchronizačního kabelu, šířka pásma, PTP a živý náhled sestavy.
* [Mono kamery a vegetační indexy](mono-indices.md) — kamery M3M a pracovní postup zarovnání → složení → indexování.

Kompletní přehled příkazů najdete v [Referenci CLI](../reference/cli-reference.md) a [Referenci SDK](../reference/sdk-reference.md).
