# Podporované jazyky

Chloros nabízí plnou podporu rozhraní v **38 jazycích po celém světě**, díky čemuž je přístupný uživatelům na celém světě. Jazyk můžete okamžitě přepínat jak v grafickém uživatelském rozhraní pro stolní počítače, tak v CLI.

Chloros podporuje následující jazyky:

| # | Jazyk | Nativní název | Kód CLI |
|---|----------|-------------|----------|
| 1 | 🇺🇸 Angličtina | English | `en` |
| 2 | 🇪🇸 Španělština | Español | `es` |
| 3 | 🇵🇹 Portugalština | Português | `pt` |
| 4 | 🇫🇷 Francouzština | Français | `fr` |
| 5 | 🇩🇪 Němčina | Deutsch | `de` |
| 6 | 🇮🇹 Italština | Italiano | `it` |
| 7 | 🇯🇵 japonština | 日本語 | `ja` |
| 8 | 🇰🇷 korejština | 한국어 | `ko` |
| 9 | 🇨🇳 Čínština (zjednodušená) | 简体中文 | `zh` |
| 10 | 🇹🇼 Čínština (tradiční) | 繁體中文 | `zh-TW` |
| 11 | 🇷🇺 Ruština | Русский | `ru` |
| 12 | 🇳🇱 Nizozemština | Nederlands | `nl` |
| 13 | 🇸🇦 Arabština | العربية | `ar` |
| 14 | 🇵🇱 Polština | Polski | `pl` |
| 15 | 🇹🇷 Turečtina | Türkçe | `tr` |
| 16 | 🇮🇳 hindština | हिंदी | `hi` |
| 17 | 🇮🇩 Indonéština | Bahasa Indonesia | `id` |
| 18 | 🇻🇳 Vietnamština | Tiếng Việt | `vi` |
| 19 | 🇹🇭 Thajština | ไทย | `th` |
| 20 | 🇸🇪 Švédština | Svenska | `sv` |
| 21 | 🇩🇰 Dánština | Dansk | `da` |
| 22 | 🇳🇴 Norština | Norsk | `no` |
| 23 | 🇫🇮 finština | Suomi | `fi` |
| 24 | 🇬🇷 řečtina | Ελληνικά | `el` |
| 25 | 🇨🇿 Čeština | Čeština | `cs` |
| 26 | 🇭🇺 Maďarština | Magyar | `hu` |
| 27 | 🇷🇴 Rumunština | Română | `ro` |
| 28 | 🇺🇦 Ukrajinština | Українська | `uk` |
| 29 | 🇧🇷 brazilská portugalština | Português Brasileiro | `pt-BR` |
| 30 | 🇭🇰 kantonština | 粵語 | `zh-HK` |
| 31 | 🇲🇾 Malajština | Bahasa Melayu | `ms` |
| 32 | 🇸🇰 Slovenská | Slovenčina | `sk` |
| 33 | 🇧🇬 Bulharština | Български | `bg` |
| 34 | 🇭🇷 Chorvatština | Hrvatski | `hr` |
| 35 | 🇱🇹 litevština | Lietuvių | `lt` |
| 36 | 🇱🇻 lotyština | Latviešu | `lv` |
| 37 | 🇪🇪 Estonština | Eesti | `et` |
| 38 | 🇸🇮 Slovinština | Slovenščina | `sl` |

## Jak změnit jazyk

### V aplikaci Chloros pro stolní počítače

1. Otevřete nastavení aplikace.
2. Přejděte do nabídky pro výběr jazyka.
3. Ze seznamu vyberte požadovaný jazyk.
4. Rozhraní se okamžitě aktualizuje.

### V Chloros CLI

K zobrazení nebo změně jazyka rozhraní CLI použijte příkaz `language`:

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

Další podrobnosti najdete v [dokumentaci k CLI](CLI.md).

## Pokrytí

Všech 38 jazyků je plně podporováno v:

* **Chloros Desktop** – Kompletní překlad grafického uživatelského rozhraní
* **Chloros CLI** – Rozhraní příkazového řádku a výstupní zprávy

Python SDK API a jeho [referenční dokumentace](reference/sdk-reference.md) jsou k dispozici v angličtině.

Díky jazykové podpoře mohou uživatelé po celém světě efektivně pracovat ve svém rodném jazyce bez jakýchkoli překážek.
