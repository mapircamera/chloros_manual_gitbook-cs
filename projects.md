# Grafické rozhraní: Projekty

Chloros umožňuje vytvářet projekty, které lze v budoucnu znovu otevřít. Projekt je obyčejná složka (uvnitř vaší projektové složky), která obsahuje:

* `project.json` — nastavení projektu, seznam souborů a předvolby zobrazení
* `cameras.json` — kamery a pole připojené během otevření projektu, včetně jejich nastavení
* `sensors.json` — světelné senzory DAQ připojené během otevření projektu, včetně přiřazení kamer a senzorů
* vaše snímky, záznamy `.daq` a složky se zpracovanými výstupy

Neexistuje žádný proprietární formát projektových souborů — projekt tvoří složka a její soubory JSON, což také usnadňuje kopírování, archivaci a spouštění projektů z [CLI](CLI.md) nebo [Python SDK](api-python-sdk.md).

Pokud to nezměníte v [Nastaveních projektu](project-settings/project-settings.md), je složka projektu `Documents\MAPIR\Chloros Projects` ve složce Dokumenty (`~/Documents/MAPIR/Chloros Projects` v systému Linux).

{% hint style="warning" %}
**Windows: používejte krátké názvy složek projektu, pokud nemáte zapnuté dlouhé cesty.**Windows se dodává s vypnutými dlouhými cestami, a proto žádná cesta k souboru nesmí být delší než 260 znaků. Zpracované výstupy se ukládají přibližně 130 znaků pod složkou projektu, takže od verze 1.3.0 Chloros odmítne příliš hlubokou složku projektu – při jejím vytvoření, otevření nebo při zahájení zpracování – místo toho, aby se spustil a nic nezapsal. Zpráva uvádí nejdelší možnou složku projektu, která se vejde. Použijte kratší složku projektu nebo zapněte dlouhé cesty v systému Windows (možnost**Povolit dlouhé cesty v systému Windows** v instalačním programu, která je ve výchozím nastavení zaškrtnutá) a restartujte Chloros. Viz [Spuštění zpracování](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).
{% endhint %}

## Nový projekt

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>V hlavním menu vyberte „Nový projekt“ a zadejte jedinečný název pro svůj projekt.

Pokud jste uložili nějaké šablony projektů, pod polem pro název se zobrazí rozevírací seznam **Vybrat šablonu** — výběrem jedné z nich spustíte nový projekt s nastavením dané šablony. Šablony se ukládají v části [Nastavení projektu](project-settings/project-settings.md): do pole „Název šablony projektu“ zadejte název a klikněte na ikonu uložení.

## Otevřít projekt

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>Sekce Otevřít projekt obsahuje seznam všech projektů ve vaší projektové složce, přičemž v dolní části se nachází položka <strong>Otevřít projektovou složku</strong></p></figcaption></figure>

Vyberte možnost „Otevřít projekt“, abyste zobrazili seznam existujících projektů ve složce s projekty. Pokud žádné projekty neexistují, sekundární boční nabídka se neotevře. Na výše uvedené fotografii můžete vidět některé projekty vytvořené pomocí grafického uživatelského rozhraní (t1, t2, t3). Projekty DATE\_TIME byly vytvořeny pomocí příkazového řádku (CLI) s využitím výchozího schématu pojmenování projektů. Kliknutím na název libovolného projektu jej otevřete.

Kliknutím na tlačítko „Otevřít složku projektu“ se otevře Průzkumník souborů vašeho počítače v cestě k projektu. Cestu k projektu můžete upravit v [Nastavení projektu](project-settings/project-settings.md).

Pokud byly od posledního otevření projektu přesunuty nebo smazány nějaké zdrojové obrazové soubory projektu, Chloros místo otevření prázdné mřížky zobrazí dialogové okno s přesným seznamem chybějících souborů.

## Duplikovat projekt

Tato možnost je k dispozici, jakmile je projekt otevřen. Vyberte „Duplikovat projekt“, abyste zkopírovali aktuální projekt pod novým názvem – Chloros navrhne další volný název (např. „Můj projekt (2)“) – a duplikát se okamžitě otevře.

## Přidat soubory

Po otevření projektu vyberte z hlavního menu možnost „Přidat soubory“, abyste do aktuálního projektu přidali jednotlivé obrazové soubory. Tato funkce odpovídá funkci přidávání v prohlížeči souborů, ale pro větší pohodlí je přístupná přímo z hlavního menu.

## Přidat složku

Po otevření projektu vyberte z hlavního menu položku „Přidat složku“, abyste do aktuálního projektu přidali složky s obrázky. Najednou můžete vybrat více složek. Duplicitní soubory jsou ignorovány.

## Spustit / Zastavit zpracování

Po přidání souborů do projektu se v hlavním menu zobrazí možnost „Spustit zpracování“. Jedná se o stejnou akci jako kliknutí na tlačítko Přehrát/Spustit v horní záhlaví. Během zpracování se položka menu změní na „Zastavit zpracování“, což vám umožní zastavit zpracovatelský řetězec.

## Připojit kameru / Připojit světelný senzor

Ve spodní části hlavního menu se nacházejí dvě hardwarové zkratky, které jsou k dispozici bez ohledu na to, zda je projekt otevřený:

* **Připojit kameru** — otevře [kartu Kamery](lattice/), kde můžete připojit kameru nebo pole kamer LATTICE.
* **Připojit k světelnému senzoru** — otevře [kartu Světelné senzory](daq/) pro připojení světelného senzoru DAQ.

Připojení hardwaru při otevřeném projektu uloží toto připojení do projektu (viz níže). Bez projektu jsou připojení platná pouze pro danou relaci.

{% hint style="info" %}
Položky nabídky Přidat soubory, Přidat složku a Spustit/Zastavit zpracování jsou viditelné nebo aktivní pouze tehdy, je-li otevřen projekt a byly-li přidány soubory. Poskytují rychlý přístup k akcím, které jsou k dispozici také prostřednictvím postranního panelu Prohlížeč souborů a tlačítek v záhlaví.
{% endhint %}

## Projekty si pamatují váš hardware

Novinka ve verzi 1.2.0: projekt si uchovává hardware, který připojíte, dokud je otevřený. Kamery a pole (včetně nastavení jednotlivých kamer, názvů, barev a rozložení mřížky) se automaticky ukládají do souboru `cameras.json` a světelné senzory (včetně názvů, barev a přiřazení ke kamerám) do souboru `sensors.json` — automaticky, během vaší práce.

Když projekt **znovu otevřete**, Chloros se k žádnému hardwaru okamžitě nepřipojí. Každá polovina se znovu připojí při prvním otevření záložky, ke které patří:

* Otevřením záložky **Kamery** se znovu připojí uložené kamery a pole a znovu se použijí jejich uložená nastavení.
* Otevření záložky **Světelné senzory**znovu připojí uložené senzory DAQ a znovu použije jejich uložená nastavení, včetně krytu. Senzor nastavený v nabídce krytu na**None (holý senzor)** zůstane holý. Projekt uložený dřívější verzí Chlorosu zaznamenal holý senzor jako jedinou možnost, takže se obnoví s korekcí kosinus slunečního svitu; nastavte jej znovu na None a projekt uložte.

Tímto způsobem otevření projektu pouze za účelem prohlížení nebo exportu snímků nikdy nespustí streamování z kamer. Pokud při otevření karty nelze najít uložené zařízení, dialogové okno vás informuje, která zařízení nejsou k dispozici, abyste je mohli znovu připojit nebo odstranit.

## Záznamy DAQ a soubory .daq v projektu

* Záznamy `.daq` pořízené při otevřeném projektu (z karty Světelné senzory nebo během snímání) se **automaticky přidávají do projektu**.
* Importované soubory `.daq` a všechny záznamy projektu jsou uvedeny v sekci **Světelný senzor DAQ** v [Nastavení projektu](project-settings/project-settings.md), každý s vlastním profilem korekce stropu.
* Během zpracování poskytují soubory `.daq` projektu osvětlení směrem dolů pro produkty odrazivosti — viz [Formáty výstupních obrázků](output-image-formats.md).

## Spuštění uloženého projektu bez grafického rozhraní

Uložený projekt lze spustit bez grafického rozhraní:

* **CLI**: `chloros-cli project open / connect / capture / sensor / align / run` pracuje s cestou ke složce projektu — viz [Referenční příručka CLI](reference/cli-reference.md).
* **SSDK**: `chloros_sdk.open_project(path)` vrací identifikátor projektu; `connect_all()` uvede všechny uložené kamery, pole a světelné senzory do provozního stavu s jejich uloženými nastaveními (světelné senzory prostřednictvím spuštěného backendu Chloros) — viz [Referenční příručka k SDK](reference/sdk-reference.md).
