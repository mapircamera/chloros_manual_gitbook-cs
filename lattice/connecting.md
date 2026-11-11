# Připojení kamer

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>Karta „Kamery“ před připojením jakéhokoli zařízení</p></figcaption></figure>

Chloros
automaticky detekuje kamery LATTICE na síti — z karty „Kamery“ v grafickém uživatelském rozhraní, z `chloros-cli lattice` nebo z Python
SDK
. Řetězec označující model kamery řídí vše, co následuje: Chloros
určí profil snímače, rozložení pásem a tovární kalibraci na základě souborů `DeviceUserID` a `DeviceSerialNumber` kamery, takže **není třeba nic konfigurovat pro jednotlivé kamery**.

Před připojením se ujistěte, že je nastavena hostitelská síť — adresování link-local, jumbo rámce a v případě polí nastavení přijímacího bufferu síťové karty. Jedná se o nastavení na straně hardwaru, které najdete v příručce LATTICE: [**Nastavení sítě**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Připojení z grafického uživatelského rozhraní

Otevřete záložku **Kamery** v postranním panelu Chloros
u (záložky hardwaru se zobrazí, jakmile se dokončí spouštění backendu), nebo použijte hlavní menu → **Připojit ke kameře**. Obojí otevře dialogové okno**Připojit kamery**.

### Dialogové okno **Připojit kameru (kamery)**Dialogové okno ihned po otevření prohledá síť („Prohledávání sítě...“) a zobrazí seznam všech nalezených kamer. Každý řádek zobrazuje**model**kamery (např. `LATT-M3M-L41-F550`),**sériové číslo**a**IP adresu**.

* **Kliknutím na řádek jej vyberete**(zelené zvýraznění). Můžete vybrat**více kamer** a připojit je najednou — Chloros
je připojí postupně.
* Řádky s označením **„Připojeno“** jsou již připojeny a nelze je znovu vybrat.
* Řádky s označením **„V poli“** patří k aktuálně připojenému poli kamer. Chcete-li danou kameru používat samostatně, nejprve odpojte skupinu.
* **Připojit** — připojí vybrané kamery; pokud je vybráno více než jedna kamera, zobrazí se na tlačítku počet, např. „Připojit (3)“.
* **Znovu vyhledat** — spustí vyhledávání znovu.
* **Zavřít** — zavře dialogové okno.
* Pokud vyhledávání skončí bez výsledků, v dialogovém okně se zobrazí **„V síti nebyly nalezeny žádné kamery“** — viz [Řešení problémů](connecting.md#troubleshooting) níže.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>Dialogové okno „Připojit kameru (kamery)“ — zde zobrazeno bez kamer v síti</p></figcaption></figure>

### První připojení: stažení kalibračního balíčku

Při **prvním** připojení dané kamery k počítači si Chloros
stáhne tovární kalibrační balíček kamery (\~3,8 MB) přímo z kamery přes rozhraní GigE. Během tohoto procesu se v dialogovém okně zobrazí zelený panel **„Stahování kalibračních dat z kamery“**s ukazatelem průběhu pro každé sériové číslo — počítejte s přibližně**70 sekundami** na jednu kameru. Při více než šesti kamerách se řádky ukazatele průběhu posouvají uvnitř panelu. Balíček se ukládá do mezipaměti na hostitelském počítači, takže při pozdějším připojení stejné kamery se stahování zcela přeskočí (a panel se nezobrazí).

### Analýza systému

Tlačítko **Analyze System** v dialogovém okně provede test hostitele a sítě (během provádění se zobrazuje nápis „Analyzing...“) a vygeneruje diagnostickou zprávu:

* **Host** — jádra procesoru a RAM; název a paměť grafické karty, případně „GPU: None detected“.
* **Síťová rozhraní** — název každé síťové karty, rychlost připojení, MTU (s označením „jumbo“, je-li aktivní), stav připojení (aktivní/neaktivní) a zda je připojena na sběrnici USB.
* **Kamery**— sériové číslo, model, IP adresa a**na které síťové kartě je která kamera připojena**.
* **Výkon** — aktuální vs. ideální počet snímků za sekundu (fps) na kameru pro daný formát pixelů, se zeleným řádkem „Potenciál: možné zlepšení o N×“, pokud ideální hodnota převyšuje aktuální.
* **Varování a očíslovaná doporučení** — nebo hlášení „Systém vypadá v pořádku pro aktuální počet kamer“, pokud není třeba nic opravovat.

Spusťte tento nástroj vždy, když se vyhledávání nebo streamování chová neočekávaně — identifikuje většinu problémů na straně síťových karet (nesprávná hodnota MTU, kamera připojená k nesprávnému rozhraní, omezení USB adaptéru) bez nutnosti opustit dialogové okno.

### Připojení pole

Chcete-li připojit dvě nebo více kamer jako **synchronizované pole**, použijte místo toho průvodce připojením pole (**Připojit pole kamer**): provede vás výběrem master/slave (předvyplněno pomocí sondy synchronizačního kabelu — jakákoli kabelem připojená kamera může být master, a kamera RGB
`-FRGB` je předvolena, pokud je připojena kabelem), výběrem režimu zobrazení (samostatné vs. kombinované dlaždice) a scénou nastavení pole s živou projekcí dosažitelných snímků za sekundu (fps) a šířky pásma kabelu před potvrzením. Průvodce a pracovní postupy pro pole kamer jsou popsány v části této příručky věnované polím s více kamerami; ekvivalentem v CLI
je „LATTICE Camera First-Connect Workflow“ v [CLI
Reference](../reference/cli-reference.md).

## Připojení z CLI
a SDK
Přístup k

CLI
a SDK
vyžaduje placený tarif Chloros
+ a přihlášení; toto omezení je vynucováno na straně serveru (`401 AUTH_REQUIRED` při odhlášení, `403 PLAN_UPGRADE_REQUIRED` v bezplatném tarifu).

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

Úplné podpisy, možnosti a pracovní postupy pro zachycování: [Referenční příručka CLI
](../reference/cli-reference.md) § `chloros-cli lattice`, [Referenční příručka k SDK
](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Jak se provádí kalibrace při připojení

Každá kamera LATTICE má svůj tovární kalibrační balíček **uložený přímo v kameře** a Chloros
při připojení kamery také kontroluje cloud MAPIR
:

| Situace   | Co Chloros
používá                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Online**|**Nejnovější kalibrace zveřejněná pro dané sériové číslo** — kopie v cloudu má přednost před kopií v kameře. Kamera, která byla překalibrována nebo aktualizována společností MAPIR
, se proto aktualizuje automaticky; není nutný žádný zásah uživatele. |
| **Offline**|**Balíček v kameře**, v původním stavu. Plně offline pracovní postupy fungují i nadále; jednoduše nepřijímají novější kalibrace, dokud se kamera alespoň jednou nepřipojí k internetu (nebo nedojde k obnovení továrního nastavení).                                                  |

V okamžiku pořízení snímku jsou skutečně použité koeficienty **zaznamenány do metadat XMP každého snímku**. Pozdější aktualizace kalibrace nikdy tiše nemění snímky, které jste již pořídili — při opětovném zpracování starého snímku se použijí koeficienty zaznamenané v jeho metadatech XMP, nikoli ty, které jsou dnes nejnovější.

## Řešení problémů

* **„V síti nebyly nalezeny žádné kamery“**— ověřte nastavení link-local v [Nastavení sítě](https://mapir.gitbook.io/lattice-camera/setup/network-setup): statická síťová karta hostitele `169.254.x.x/16`, kamery na stejném spojení, neočekává se použití DHCP ani brány. Poté použijte funkci**Analyze System**v dialogovém okně pro připojení, abyste zkontrolovali, na které síťové kartě je (nebo není) každá kamera viditelná. Po jakékoli změně kabeláže nebo síťové karty proveďte**Rescan**.
* **Dříve funkční sestava se odmítá připojit** (panely pole s chybami `FRAMES WILL DROP` / `Reduce ROI to enable`) — aktualizace ovladače síťové karty bez upozornění resetovala nastavení přijímacího kruhu. Znovu je použijte nebo spusťte příkaz `chloros-cli lattice network --fix` z terminálu s oprávněními správce; viz [Nastavení sítě](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **Kamera zobrazuje „In Array“** — patří k připojené relaci pole. Odpojte pole, chcete-li kameru používat samostatně.
