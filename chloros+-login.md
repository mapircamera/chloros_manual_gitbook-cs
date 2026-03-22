# Chloros+ – přihlášení

## Chloros a Chloros (prohlížeč) – přihlášení

Boční <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> v postranní nabídce se můžete přihlásit ke svému účtu Chloros+ a odemknout další funkce.

Po přihlášení se zobrazí podrobnosti o vašem účtu:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## CLI Přihlášení

Přihlaste se pomocí svých přihlašovacích údajů pro Chloros+, abyste povolili zpracování CLI. V Linux (bez grafického uživatelského rozhraní) je to jediný způsob, jak aktivovat licenci.

**Syntaxe:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Uživatelé SDK**: Python SDK také poskytuje programovou metodu `logout()` pro vymazání přihlašovacích údajů uložených v mezipaměti. Podrobnosti najdete v [dokumentaci k Python SDK](api-python-sdk.md#logout).
{% endhint %}

**Příklad:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Zvláštní znaky**: Hesla obsahující znaky jako `$`, `!` nebo mezery uveďte v jednoduchých uvozovkách.
{% endhint %}

**Výstup:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Ukládání přihlašovacích údajů

Uložené přihlašovací údaje se ukládají do umístění specifického pro danou platformu:

| Platforma | Cesta k mezipaměti přihlašovacích údajů |
| --- | --- |
| **Windows** | `%APPDATA%\Chloros\cache\` |
| **Linux** | `~/.cache/chloros/` |

### Vypršení platnosti plánu

Vypršení platnosti plánu v grafickém uživatelském rozhraní (GUI) ukazuje, kdy vaše licence přestane platit. U opakujících se měsíčních předplatných je vypršení platnosti na konci měsíce. U ročních předplatných je to rok od zahájení předplatného. Kontrola licence vyžaduje měsíční připojení k internetu k ověření, s 30denní lhůtou.

### Limit zařízení

Každý plán Chloros+ nabízí jiný počet registrovaných zařízení. Každé zařízení, na kterém se přihlásíte pomocí účtu Chloros+, se započítá do počtu vašich registrovaných zařízení. Na stránce svého účtu MAPIR Cloud můžete zařízení přejmenovat a odstranit.

<table><thead><tr><th width="168.5999755859375" align="right">Tarif Chloros+</th><th align="center">MĚĎ</th><th align="center">BRONZE</th><th align="center">STŘÍBRO</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Podporovaná zařízení</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
