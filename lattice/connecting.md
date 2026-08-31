# Připojení kamer

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>Karta „Kamery“ před připojením jakéhokoli zařízení</p></figcaption></figure>Chloros automaticky detekuje kamery LATTICE na síti — z karty „Kamery“ v grafickém uživatelském rozhraní, z `chloros-cli lattice` nebo z Python SDK. Řetězec označující model kamery určuje vše, co následuje: Chloros určí profil snímače, rozložení pásem a tovární kalibraci na základě parametrů kamery `DeviceUserID` + `DeviceSerialNumber`, takže **není třeba nic konfigurovat pro jednotlivé kamery**.

Před připojením se ujistěte, že je nastavena hostitelská síť — adresování link-local, jumbo rámce a v případě polí nastavení přijímacího bufferu síťové karty. Jedná se o nastavení na straně hardwaru, které je popsáno v příručce LATTICE: [**Nastavení sítě**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Připojení z grafického uživatelského rozhraní

Otevřete záložku **Kamery**v postranním panelu Chloros (záložky hardwaru se zobrazí, jakmile se dokončí spouštění backendu), nebo použijte hlavní menu →**Připojit se ke kameře**. Oba způsoby otevřou dialogové okno**Připojit kameru(y)**.

### Dialogové okno **Připojit kamery**Dialogové okno ihned po otevření prohledá síť („Prohledávání sítě...“) a zobrazí seznam všech nalezených kamer. Každý řádek zobrazuje**model**kamery (např. `LATT-M3M-L41-F550`),**sériové číslo**a**IP adresu**.

* **Kliknutím na řádek jej vyberete**(zelené zvýraznění). Můžete vybrat**více kamer** a připojit je najednou — Chloros je připojí postupně.
* Řádky s označením **„Připojeno“** jsou již připojeny a nelze je znovu vybrat.
* Řádky s označením **„V poli“** patří k aktuálně připojenému poli kamer. Chcete-li danou kameru použít samostatně, nejprve odpojte celé pole.
* **Připojit** — připojí vybrané kamery; pokud je vybráno více než jedna kamera, zobrazí se na tlačítku jejich počet, např. „Připojit (3)“.
* **Znovu prohledat** — spustí vyhledávání znovu.
* **Zavřít** — zavře dialogové okno.
* Pokud vyhledávání skončí bez výsledků, v dialogovém okně se zobrazí **„V síti nebyly nalezeny žádné kamery“** — viz [Řešení problémů](connecting.md#troubleshooting) níže.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>Dialogové okno „Připojit kameru (kamery)“ — zde zobrazeno bez kamer v síti</p></figcaption></figure>### První připojení: stažení kalibračního balíčku

**Při prvním**připojení dané kamery k počítači si Chloros stáhne tovární kalibrační balíček kamery (\~3,8 MB) přímo z kamery přes rozhraní GigE. Během tohoto procesu se v dialogovém okně zobrazí zelený panel**„Stahování kalibračních dat z kamery“**s ukazatelem průběhu pro každé sériové číslo – počítejte s přibližně**70 sekundami** na jednu kameru. Balíček se ukládá do mezipaměti hostitelského počítače, takže při pozdějších připojeních téže kamery se stahování zcela přeskočí (a panel se nikdy nezobrazí).

### Analyzovat systém

Tlačítko **Analyzovat systém** v dialogovém okně provede test hostitele a sítě (během provádění se zobrazuje nápis „Analyzuji...“) a vygeneruje diagnostickou zprávu:

* **Hostitel** — jádra procesoru a RAM; název a paměť GPU, případně „GPU: Žádná detekována“.
* **Síťová rozhraní** — název každé síťové karty, rychlost připojení, MTU (s označením „jumbo“, je-li aktivní), stav připojení a informace o tom, zda je připojena k sběrnici USB.
* **Kamery**— sériové číslo, model, IP adresa a**na které síťové kartě je která kamera připojena**.
* **Výkon** — aktuální vs. ideální počet snímků za sekundu (fps) na kameru pro daný formát pixelů, se zeleným řádkem „Potenciál: možné N× zlepšení“, pokud ideální hodnota převyšuje aktuální.
* **Varování a očíslovaná doporučení** — nebo hlášení „Systém vypadá v pořádku pro aktuální počet kamer“, pokud není třeba nic opravovat.

Spusťte tento nástroj vždy, když se vyhledávání nebo streamování chová neočekávaně — identifikuje většinu problémů na straně síťových karet (nesprávná hodnota MTU, kamera na nesprávném rozhraní, omezení USB adaptéru), aniž by bylo nutné opustit dialogové okno.

### Připojení pole

Chcete-li připojit dvě nebo více kamer jako **synchronizované pole**, použijte místo toho průvodce připojením pole (**Připojit pole kamer**): provede vás výběrem master/slave (předvyplněno sondou pro zapojení GPIO), volbou režimu zobrazení (Samostatné vs. Kombinované dlaždice) a scénou nastavení pole s živou projekcí dosažitelného počtu snímků za sekundu (fps) a šířky pásma kabelu, než provedete potvrzení. Průvodce a pracovní postupy pro pole jsou popsány v části této příručky věnované polím s více kamerami; ekvivalentem pro model CLI je „Pracovní postup prvního připojení kamery LATTICE“ v [Referenci k modelu CLI](../reference/cli-reference.md).

## Připojení z CLI a SDK

Přístup k CLI a SDK vyžaduje placený tarif Chloros+ a přihlášení; toto omezení je vynucováno na straně serveru (`401 AUTH_REQUIRED` při odhlášení, `403 PLAN_UPGRADE_REQUIRED` v bezplatném tarifu).

```bash
# List cameras on the network (vendor, model, serial, IP, MAC)
chloros-cli lattice info

# Single-camera smoke test: capture one frame (saves every applicable export type)
chloros-cli lattice capture -o output/

# Connect a synchronized array — same smart-prep flow as the GUI
chloros-cli lattice array-connect --serials 213800234,214000533
```

```python
import chloros_sdk

# Persistent live-camera session through the backend
with chloros_sdk.connect_camera("213800234") as cam:
    ...

# Array session (smart-prep: network probe, tier auto-pick, PTP, AE seeding, trigger config)
with chloros_sdk.connect_array(["213800234", "214000533"]) as array:
    ...
```

Úplné podpisy, možnosti a pracovní postupy zachycování: [CLI Reference](../reference/cli-reference.md) § `chloros-cli lattice`, [SDK Odkaz](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Jak se provádí kalibrace při připojení

Každá kamera LATTICE má svůj tovární kalibrační balíček **uložený přímo v kameře** a model Chloros při připojení kamery také kontroluje cloud modelu MAPIR:

| Situace   | Co používá model Chloros                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Online**|**Nejnovější kalibrace zveřejněná pro dané sériové číslo** — kopie v cloudu má přednost před kopií v kameře. Kamera, která byla překalibrována nebo aktualizována pomocí MAPIR, se proto aktualizuje automaticky; není nutný žádný zásah uživatele. |
| **Offline**|**Balíček v kameře**, v původním stavu. Plně offline pracovní postupy fungují i nadále; jednoduše nepřijímají novější kalibrace, dokud se kamera alespoň jednou nepřipojí k internetu (nebo nedojde k obnovení továrního nastavení).                                                  |

V okamžiku pořízení snímku jsou skutečně použité koeficienty **uloženy do XMP metadat každého snímku**. Pozdější aktualizace kalibrace nikdy tiše nezmění snímky, které jste již pořídili — při opětovném zpracování starého snímku se použijí koeficienty uložené v jeho XMP, nikoli ty nejnovější z daného dne.

## Řešení problémů

* **„V síti nebyly nalezeny žádné kamery“**— ověřte nastavení link-local v [Network Setup](https://mapir.gitbook.io/lattice-camera/setup/network-setup): statická síťová karta hostitele `169.254.x.x/16`, kamery na stejném spojení, neočekává se použití DHCP ani brány. Poté použijte funkci**Analyze System**v dialogovém okně pro připojení, abyste zkontrolovali, na které síťové kartě je (nebo není) každá kamera viditelná. Po jakékoli změně kabeláže nebo síťové karty proveďte**Rescan**.
* **Dříve funkční sestava se odmítá připojit** (panely pole s chybami `FRAMES WILL DROP` / `Reduce ROI to enable`) — aktualizace ovladače síťové karty bez upozornění resetovala nastavení přijímacího kruhu. Znovu je použijte nebo spusťte příkaz `chloros-cli lattice network --fix` z terminálu s oprávněními správce; viz [Nastavení sítě](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **Kamera zobrazuje „In Array“** — patří k připojené relaci pole. Odpojte pole, abyste mohli kameru používat samostatně.
