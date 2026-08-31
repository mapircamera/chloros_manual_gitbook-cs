# Chloros+ Přihlášení

## Přihlášení přes grafické rozhraní

Boční nabídka uživatele <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> vám umožňuje přihlásit se ke svému účtu Chloros+ a odemknout další funkce.

**Na každém počítači se stačí přihlásit pouze jednou.** Grafické uživatelské rozhraní, CLI a Python SDK sdílejí stejnou relaci v mezipaměti — přihlášení přes grafické uživatelské rozhraní na ploše aktivuje také CLI a SDK na daném počítači (a naopak přes `chloros-cli login`).

Po přihlášení se zobrazí podrobnosti vašeho účtu:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.2.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->
## Úrovně tarifů

| Tarif | `plan_id` | Typ |
| --- | --- | --- |
| Iron | `0` | Zdarma |
| Copper | `1` | Placený (Chloros+) |
| Bronze | `2` | Placený (Chloros+) |
| Stříbro | `3` | Placené (Chloros+) |
| Zlatá | `4` | Placená (Chloros+) |

Co jednotlivé placené úrovně zahrnují, najdete v [tarifech a cenách](https://cloud.mapir.camera/pricing).

### Přístup k CLI / SDK vyžaduje placený tarif

Přístup k CLI a Python SDK vyžaduje **jakýkoli placený tarif Chloros+ (Copper nebo vyšší)**. Toto omezení je vynucováno**na straně serveru** — každý požadavek typu CLI/SDK musí obsahovat jak aktivní relaci, tak placený tarif:

| Stav HTTP | `error_code` | Význam | Řešení |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Není přihlášeno na tomto zařízení | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Jste přihlášeni, ale úroveň tarifu je příliš nízká (bezplatná úroveň Iron) | Přejděte na jakýkoli placený tarif Chloros+ |

`chloros-cli status` zůstává dostupný i v bezplatné úrovni, takže můžete kdykoli zkontrolovat svůj aktuální tarif a důvod, proč byl přístup odepřen.

### Limity připojeného hardwaru podle tarifu

Každý tarif omezuje počet kamer LATTICE a světelných senzorů DAQ, které lze připojit k živému přenosu najednou:

| Tarif | Kamery LATTICE | Světelné senzory DAQ |
| --- | --- | --- |
| Iron (bezplatný / bez přihlášení) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## Přihlášení do CLI

Přihlaste se pomocí svých přihlašovacích údajů pro Chloros+, abyste mohli aktivovat zpracování v CLI. V systému Linux (bez grafického rozhraní) je to jediný způsob, jak aktivovat licenci.

**Syntaxe:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Uživatelé SDK**: Python SDK také poskytuje programovou metodu `logout()` pro vymazání přihlašovacích údajů z mezipaměti. Podrobnosti najdete v [referenční příručce k SDK](reference/sdk-reference.md).
{% endhint %}

**Příklad:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Zvláštní znaky**: Hesla obsahující znaky jako `$`, `!` nebo mezery uveďte v jednoduchých uvozovkách.
{% endhint %}

**Výstup:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.2.0"; capture a successful login with the current output format. -->
### Ukládání přihlašovacích údajů

Uložené přihlašovací údaje a konfigurace se na **všech platformách** ukládají do složky `.chloros` ve vašem domovském adresáři uživatele:

| Platforma | Cesta k mezipaměti přihlašovacích údajů |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

### Vypršení platnosti plánu a offline lhůta

Datum vypršení platnosti plánu v grafickém uživatelském rozhraní udává, kdy vaše licence přestane být platná. U opakujících se měsíčních předplatných je datum vypršení platnosti na konci měsíce; u ročních předplatných je to rok od zahájení předplatného.

Chloros ověřuje vaši licenci online, ale práce v režimu offline je podporována v rámci přechodného období:

* Úspěšná ověření na serveru se ukládají do mezipaměti na **5 minut**, takže při běžném používání dochází k velmi malému počtu volání licence.
* Podepsaná mezipaměť licence vázaná na konkrétní zařízení pokrývá delší období offline: **30 dní u měsíčních plánů**a**až do data vypršení platnosti vašeho předplatného (maximálně 365 dní) u ročních plánů**.
* Po uplynutí přechodného období se tarif přepne na bezplatnou úroveň Iron, dokud se zařízení alespoň jednou nepřipojí k licenčnímu serveru; přístup bude obnoven při příští úspěšné kontrole.

### Limit zařízení

Každý tarif Chloros+ nabízí odlišný počet registrovaných zařízení. Každé zařízení, na kterém se přihlásíte pomocí účtu Chloros+, se započítává do počtu vašich registrovaných zařízení. Na stránce svého účtu MAPIR Cloud můžete zařízení přejmenovat nebo odstranit.

<table><thead><tr><th width="168.5999755859375" align="right">Tarif Chloros+</th><th align="center">MĚĎ</th><th align="center">BRONZE</th><th align="center">STŘÍBRO</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Podporovaná zařízení</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>Přesný počet zařízení, které může váš účet podporovat, je uveden na stránce vašeho účtu MAPIR Cloud. Odhlášení ze zařízení spolehlivě uvolní jeho místo a zařízení, které je již zaregistrováno, se může kdykoli znovu přihlásit, i když účet dosáhl svého limitu počtu zařízení.
