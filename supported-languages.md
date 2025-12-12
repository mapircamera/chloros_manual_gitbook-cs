# Podporované jazyky

Chloros poskytuje plnou podporu rozhraní v **38 jazycích po celém světě**, díky čemuž je přístupný uživatelům z celého světa. Jazyky můžete okamžitě přepínat ve všech rozhraních: Desktop, Browser, CLI a Python SDK.

Chloros podporuje následující jazyky:

| # | Jazyk | Rodný název | Kód CLI |
|---|----------|-------------|----------|
| 1 | 🇺🇸 Angličtina | Angličtina | `en` |
| 2 | 🇪🇸 Španělština | Español | `es` |
| 3 | 🇵🇹 Portugalština | Português | `pt` |
| 4 | 🇫🇷 Francouzština | Français | `fr` |
| 5 | 🇩🇪 Němčina | Deutsch | `de` |
| 6 | 🇮🇹 Italština | Italiano | `it` |
| 7 | 🇯🇵 Japonština | 日本語 | `ja` |
| 8 | 🇰🇷 Korejština | 한국어 | `ko` |
| 9 | 🇨🇳 Čínština (zjednodušená) | 简体中文 | `zh` |
| 10 | 🇹🇼 Čínština (tradiční) | 繁體中文 | `zh-TW` |
| 11 | 🇷🇺 Ruština | Русский | `ru` |
| 12 | 🇳🇱 Holandština | Nederlands | `nl` |
| 13 | 🇸🇦 Arabština | العربية | `ar` |
| 14 | 🇵🇱 Polština | Polski | `pl` |
| 15 | 🇹🇷 turečtina | Türkçe | `tr` |
| 16 | 🇮🇳 hindština | हिंदी | `hi` |
| 17 | 🇮🇩 Indonéština | Bahasa Indonesia | `id` |
| 18 | 🇻🇳 Vietnamština | Tiếng Việt | `vi` |
| 19 | 🇹🇭 Thajština | ไทย | `th` |
| 20 | 🇸🇪 Švédština | Svenska | `sv` |
| 21 | 🇩🇰 Dánština | Dansk | `da` |
| 22 | 🇳🇴 Norsky | Norsk | `no` |
| 23 | 🇫🇮 Finsky | Suomi | `fi` |
| 24 | 🇬🇷 Řečtina | Ελληνικά | `el` |
| 25 | 🇨🇿 Čeština | Čeština | `cs` |
| 26 | 🇭🇺 Maďarština | Magyar | `hu` |
| 27 | 🇷🇴 Rumunština | Română | `ro` |
| 28 | 🇺🇦 Ukrajinština | Українська | `uk` |
| 29 | 🇧🇷 Brazilská portugalština | Português Brasileiro | `pt-BR` |
| 30 | 🇭🇰 Kantonská čínština | 粵語 | `zh-HK` |
| 31 | 🇲🇾 Malajština | Bahasa Melayu | `ms` |
| 32 | 🇸🇰 Slovenština | Slovenčina | `sk` |
| 33 | 🇧🇬 Bulharština | Български | `bg` |
| 34 | 🇭🇷 Chorvatština | Hrvatski | `hr` |
| 35 | 🇱🇹 Litevština | Lietuvių | `lt` |
| 36 | 🇱🇻 Lotyština | Latviešu | `lv` |
| 37 | 🇪🇪 Estonština | Eesti | `et` |
| 38 | 🇸🇮 Slovinština | Slovenščina | `sl` |

## Jak změnit jazyk

### V Chloros Desktop/Browser

1. Otevřete nastavení aplikace.
2. Přejděte do nabídky pro výběr jazyka.
3. Ze seznamu vyberte preferovaný jazyk.
4. Rozhraní se okamžitě aktualizuje.

### V Chloros CLI

Pomocí příkazu `language` můžete zobrazit nebo změnit jazyk rozhraní CLI:

```bash
# View current language
chloros-cli language

# Change to Spanish
chloros-cli language es

# Change to Chinese (Simplified)
chloros-cli language zh

# Change to Brazilian Portuguese
chloros-cli language pt-BR

# List all available languages
chloros-cli language --list
```

Další podrobnosti najdete v [dokumentaci CLI](CLI.md).

### V Chloros Python SDK

Nastavte parametr jazyka při inicializaci SDK, abyste dostávali zprávy a výstupy ve svém preferovaném jazyce.

## Pokrytí

Všech 38 jazyků je plně podporováno v:

* **Chloros Desktop** – kompletní překlad grafického uživatelského rozhraní
* **Chloros Browser** – webové rozhraní ve všech jazycích
* **Chloros CLI** – rozhraní příkazového řádku a výstupní zprávy
* **Chloros Python SDK** – zprávy a dokumentace API

Jazyková podpora zajišťuje, že uživatelé po celém světě mohou efektivně pracovat ve svém rodném jazyce bez překážek.
