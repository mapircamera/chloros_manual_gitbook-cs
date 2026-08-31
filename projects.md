# Grafické uživatelské rozhraní: Projekty

Chloros umožňuje vytvářet projekty, které lze v budoucnu znovu otevřít. Projekt je obyčejná složka (uvnitř vaší projektové složky), která obsahuje:

* `project.json` — nastavení projektu, seznam souborů a předvolby zobrazení
* `cameras.json` — kamery a pole připojené v době, kdy byl projekt otevřen, včetně jejich nastavení
* `sensors.json` — světelné senzory DAQ připojené během otevření projektu, včetně přiřazení kamer a senzorů
* vaše snímky, záznamy `.daq` a složky se zpracovanými výstupy

Neexistuje žádný proprietární formát projektového souboru — projekt tvoří složka a její soubory JSON, což také usnadňuje kopírování, archivaci a přenos projektů z [CLI](CLI.md) nebo [Python SDK](api-python-sdk.md).

## Nový projekt

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>V hlavním menu vyberte možnost „Nový projekt“ a zadejte jedinečný název svého projektu.

Pokud jste uložili nějaké šablony projektů, pod polem pro název se zobrazí rozevírací nabídka **Vybrat šablonu** — výběrem jedné z nich spustíte nový projekt s nastavením dané šablony. Šablony se ukládají v [Nastavení projektu](project-settings/project-settings.md): zadejte název do pole „Název šablony projektu“ a klikněte na ikonu uložení.

## Otevřít projekt

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>V okně „Otevřít projekt“ se zobrazí seznam všech projektů ve vaší projektové složce, přičemž v dolní části se nachází tlačítko <strong>„Otevřít projektovou složku“</strong></p></figcaption></figure>Vyberte možnost „Otevřít projekt“, abyste zobrazili seznam existujících projektů ve složce projektů. Pokud žádné projekty neexistují, sekundární boční nabídka se neotevře. Na výše uvedené fotografii můžete vidět některé projekty vytvořené grafickým uživatelským rozhraním (t1, t2, t3). Projekty DATE\_TIME byly vytvořeny programem CLI s použitím výchozího schématu pojmenování projektů. Kliknutím na název libovolného projektu jej otevřete.

Kliknutím na tlačítko „Otevřít složku projektu“ se otevře Průzkumník souborů vašeho počítače v cestě k projektu. Cestu k projektu můžete upravit v [Nastavení projektu](project-settings/project-settings.md).

Pokud byly od posledního otevření projektu přesunuty nebo smazány některé ze zdrojových obrazových souborů projektu, Chloros místo otevření prázdné mřížky zobrazí dialogové okno s přesným seznamem chybějících souborů.

## Duplikovat projekt

Tato možnost je k dispozici po otevření projektu. Vyberte „Duplikovat projekt“ pro zkopírování aktuálního projektu pod novým názvem — Chloros navrhne další volný název (např. „Můj projekt (2)“) — a duplikát se okamžitě otevře.

## Přidat soubory

Po otevření projektu vyberte z hlavního menu možnost „Přidat soubory“, abyste do aktuálního projektu přidali jednotlivé obrazové soubory. Tato funkce odpovídá funkci přidávání v prohlížeči souborů, ale pro větší pohodlí je přístupná přímo z hlavního menu.

## Přidat složku

Po otevření projektu vyberte z hlavního menu položku „Přidat složku“, abyste do aktuálního projektu přidali složky s obrázky. Najednou můžete vybrat více složek. Duplicitní soubory jsou ignorovány.

## Spustit / Zastavit zpracování

Po přidání souborů do projektu se v hlavním menu zpřístupní položka „Spustit zpracování“. Jedná se o stejnou akci jako kliknutí na tlačítko Přehrát/Spustit v horní záhlaví. Během zpracování se položka menu změní na „Zastavit zpracování“, což vám umožní zastavit zpracovatelský řetězec.

## Připojit kameru / Připojit světelný senzor

Ve spodní části hlavního menu se nacházejí dvě hardwarové zkratky, které jsou dostupné bez ohledu na to, zda je projekt otevřený:

* **Připojit kameru** — otevře [kartu Kamery](lattice/) pro připojení kamery nebo pole kamer LATTICE.
* **Připojit k světelnému senzoru** — otevře [kartu Světelné senzory](daq/) pro připojení světelného senzoru DAQ.

Připojení hardwaru při otevřeném projektu uloží toto připojení do projektu (viz níže). Bez projektu jsou připojení platná pouze pro danou relaci.

{% hint style="info" %}
Položky nabídky Přidat soubory, Přidat složku a Spustit/Zastavit zpracování jsou viditelné nebo aktivní pouze tehdy, je-li otevřen projekt a byly-li přidány soubory. Poskytují rychlý přístup k akcím, které jsou k dispozici také prostřednictvím postranního panelu Prohlížeč souborů a tlačítek v záhlaví.
{% endhint %}

## Projekty si pamatují váš hardware

Novinka ve verzi 1.2.0: projekt si uchovává hardware, který připojíte, dokud je otevřený. Kamery a pole (včetně nastavení pro každou kameru, názvů, barev a rozložení mřížky) se automaticky ukládají do souboru `cameras.json` a světelné senzory (včetně názvů, barev a přiřazení ke kamerám) do souboru `sensors.json` — a to automaticky během vaší práce.

Když projekt **znovu otevřete**, soubor Chloros se k žádnému hardwaru okamžitě nepřipojí. Každá polovina se znovu připojí při prvním otevření záložky, ke které patří:

* Otevření záložky **Kamery** znovu připojí uložené kamery a pole a znovu použije jejich uložená nastavení.
* Otevření záložky **Světelné senzory** znovu připojí uložené senzory DAQ.

Tímto způsobem otevření projektu pouze za účelem prohlížení nebo exportu snímků nikdy nespustí streamování z kamer. Pokud při otevření záložky nelze najít uložené zařízení, dialogové okno vás informuje, která zařízení nejsou k dispozici, abyste je mohli znovu připojit nebo odstranit.

## Záznamy DAQ a soubory .daq v projektu

* Záznamy `.daq` pořízené při otevřeném projektu (z karty Světelné senzory nebo během snímání) se **automaticky přidají do projektu**.
* Importované soubory `.daq` a všechny záznamy projektu jsou uvedeny v sekci **Světelný senzor DAQ** v [Nastavení projektu](project-settings/project-settings.md), každý s vlastním profilem korekce jasu.
* Během zpracování poskytují soubory `.daq` projektu osvětlení směřující dolů pro produkty odrazivosti — viz [Formáty výstupních obrázků](output-image-formats.md).

## Spuštění uloženého projektu bez grafického rozhraní

Uložený projekt lze spustit bez grafického uživatelského rozhraní:

* **CLI**: `chloros-cli project open / connect / capture / sensor / align / run` pracuje s cestou ke složce projektu — viz [Referenční příručka k CLI](reference/cli-reference.md).
* **SDK**: `chloros_sdk.open_project(path)` vrací identifikátor projektu; `connect_all()` uvede všechny uložené kamery a senzory do provozního stavu s jejich uloženými nastaveními — viz [Referenční příručka k SDK](reference/sdk-reference.md).
