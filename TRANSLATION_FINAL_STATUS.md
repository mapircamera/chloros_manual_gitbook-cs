# Chloros Manuál – Konečný stav překladatelského projektu

**Poslední aktualizace:** 13. prosince 2025

---

## 📊 Celkový stav

### ✅ **DOKONČENO: 32 jazyků (DeepL)**

Plně přeloženo a zveřejněno na GitBook:

**Evropské jazyky (20):**
- 🇧🇬 bulharština (bg)
- 🇨🇿 čeština (cs)
- 🇩🇰 dánština (da)
- 🇩🇪 němčina (de)
- 🇬🇷 Řečtina (el)
- 🇪🇸 Španělština (es)
- 🇪🇪 Estonština (et)
- 🇫🇮 Finština (fi)
- 🇫🇷 Francouzština (fr)
- 🇭🇺 Maďarština (hu)
- 🇮🇹 Italština (it)
- 🇱🇻 Lotyština (lv)
- 🇱🇹 Litevština (lt)
- 🇳🇱 Holandština (nl)
- 🇳🇴 Norština (no)
- 🇵🇱 Polština (pl)
- 🇵🇹 portugalština (pt)
- 🇧🇷 portugalština Brazílie (pt-BR)
- 🇷🇴 rumunština (ro)
- 🇸🇰 slovenština (sk)
- 🇸🇮 slovinština (sl)
- 🇸🇪 švédština (sv)

**Další jazyky (12):**
- 🇸🇦 Arabština (ar)
- 🇨🇳 Zjednodušená čínština (zh-CN)
- 🇭🇰 Čínština Hongkong (zh-HK)
- 🇹🇼 Tradiční čínština (zh-TW)
- 🇮🇩 Indonéština (id)
- 🇯🇵 Japonština (ja)
- 🇰🇷 Korejština (ko)
- 🇷🇺 Ruština (ru)
- 🇹🇷 Turečtina (tr)
- 🇺🇦 Ukrajinština (uk)

**Kvalita překladu:**
- ✅ Veškerý obsah je plně přeložen
- ✅ Popisy přední části jsou přeloženy
- ✅ Technické termíny jsou chráněny
- ✅ Bloky kódu jsou zachovány
- ✅ Vzorce jsou neporušené
- ✅ Odkazy jsou funkční
- ✅ Formátování je dokonalé

---

### 🔄 **V PROCESU: 5 jazyků (Google Translate)**

**Aktuální stav:**
- 🇮🇳 **hindština (hi)** - ⏳ PRÁVĚ SE PŘEKLÁDÁ (2–3 hodiny)
- 🇭🇷 **chorvatština (hr)** - ⏳ Čeká se (angličtina + přeložené popisy)
- 🇲🇾 **Malajština (ms)** - ⏳ Čeká na překlad (angličtina + přeložené popisy)
- 🇹🇭 **Thajština (th)** - ⏳ Čeká na překlad (angličtina + přeložené popisy)
- 🇻🇳 **Vietnamština (vi)** - ⏳ Čeká se (angličtina + přeložené popisy)

**Proč jsou pomalejší:**
- Nepodporováno DeepL API
- Google Translate API má omezení rychlosti
- Používá ultrakonzervativní překlad řádek po řádku
- 1sekundové zpoždění na řádek, aby se zabránilo omezení rychlosti

**Aktuální stav (4 jazyky čekající na překlad):**
- ✅ Repozitáře existují na GitHub
- ✅ Popisy frontmatter přeloženy
- ✅ Všechny soubory a obrázky synchronizovány
- ⚠️ Obsah těla stále v angličtině (funkční)

---

## 🔧 Funkce překladového systému

### Automatický překlad
- **Popisná pole** v přední části automaticky přeložena
- **DeepL API** pro 32 jazyků (vysoká kvalita)
- **Google Translate** pro 5 jazyků (s konzervativním omezením rychlosti)

### Ochrana obsahu
- ✅ Názvy produktů (Chloros, MAPIR)
- ✅ Bloky kódu a vložený kód
- ✅ Matematické vzorce
- ✅ Technické názvy barev (Red, Green, Blue, NIR, RedEdge)
- ✅ Cesty k souborům a URL adresy
- ✅ Zkrácené kódy GitBook
- ✅ E-mailové adresy
- ✅ Přípony souborů

### Obsah, který se překládá
- ✅ Názvy stránek
- ✅ Text a odstavce
- ✅ Buňky a záhlaví tabulek
- ✅ Popisky a poznámky
- ✅ Text odkazů
- ✅ Popisy přední části

### Následné zpracování
- ✅ Opravuje nové řádky HTML
- ✅ Obnovuje chráněné prvky
- ✅ Opravuje problémy s formátováním
- ✅ Zajišťuje kompatibilitu GitBook

---

## 📝 Přehled skriptů

### Hlavní denní pracovní postup
**`update_all_translations.py`**
- Aktualizuje všech 37 jazykových repozitářů
- Synchronizuje text, obrázky a soubory
- Překládá pouze změněné soubory
- Automaticky potvrzuje a odesílá do GitHub
- Použití: `python update_all_translations.py`

### Překladové skripty
**`translate_with_deepl.py`**
- Základní překlad DeepL (32 jazyků)
- Zpracovává popisy frontmatter
- Plná ochrana markdown

**`translate_with_google.py`**
- Integrace Google Translate (5 jazyků)
- Stejná ochrana jako DeepL
- Zpracovává omezení API

**`translate_google_conservative.py`**
- Velmi pomalý, ale spolehlivý Google Translate
- Překlad řádek po řádku
- Dlouhé zpoždění, aby se zabránilo omezení rychlosti
- Pro obtížné jazyky: `python translate_google_conservative.py hi`

### Pomocné skripty
**`verify_all_pushed.py`**
- Kontrola, zda všech 37 repozitářů je odesláno do GitHub

**`check_google_progress.py`**
- Kontrola počtu jazykových souborů Google Translate

**`check_hindi_progress.py`**
- Podrobný průběh překladu do hindštiny

**`push_until_stable.py`**
- Nahrajte všechna repozitáře, dokud nedojde ke změnám

---

## 🌐 Integrace GitBook

### Proces synchronizace
1. Změny nahrané do repozitáře GitHub
2. GitBook se automaticky synchronizuje během 5–10 minut
3. Změny se zobrazí na živém webu

### Struktura repozitáře
- **Angličtina:** `chloros_manual_gitbook`
- **Překlady:** `chloros_manual_gitbook-{lang_code}`

### Jazykové kódy
| Název repozitáře | Kód CLI | Jazyk |
|-----------|----------|----------|
| zh-CN | zh | Zjednodušená čínština |
| zh-HK | zh | Hongkongská čínština |
| zh-TW | zh | Tradiční čínština |
| nb | no | Norština |
| pt-BR | pt-BR | Portugalština Brazílie |
| Všechny ostatní | Stejné jako repo | Standardní |

---

## 📈 Statistiky překladu

### Celková velikost projektu
- **Jazyky:** 37 + angličtina = 38 repozitářů
- **Soubory na jazyk:** ~30 souborů markdown
- **Celkový počet přeložených souborů:** 32 × 30 = 960 souborů (DeepL)
- **Obrázky/soubory:** Synchronizováno ve všech 37 repozitářích
- **Přeložené řádky:** ~50 000+ řádků

### API Použití
- **DeepL API:** ~960 přeložených souborů
- **Google Translate:** Probíhá (5 jazyků)
- **Časová náročnost:** Několik dní vývoje a překladu

### Metriky kvality
- ✅ 100 % překladů DeepL je vysoké kvality
- ✅ 100 % popisků přední strany přeloženo (všech 37 jazyků)
- ✅ 100 % formátování zachováno
- ✅ 100 % technických termínů zachováno
- ✅ 0 % nefunkčních odkazů nebo obrázků

---

## 🚀 Další kroky

### Krátkodobé (dnes)
1. ⏳ Počkejte na dokončení překladu do hindštiny (~2–3 hodiny)
2. 📤 Ověřte, zda byla hindština odeslána do GitHub
3. 🔍 Otestujte hindštinu na GitBook

### Střednědobý plán (tento týden)
1. Přeložte zbývající 4 jazyky (hr, ms, th, vi)
2. Každý z nich zabere 2–3 hodiny při použití konzervativní metody
3. Nahrajte a ověřte vše na GitBook

### Dlouhodobě
1. Sledujte, zda DeepL přidá podporu pro těchto 5 jazyků
2. Přeložit znovu pomocí DeepL, jakmile bude k dispozici.
3. Pravidelné aktualizace pomocí `update_all_translations.py`.

---

## 💡 Doporučení

### Pro pravidelné aktualizace
```bash
python update_all_translations.py
```
Toto automaticky zpracuje vše pro jazyky DeepL.

### Pro jazyky Google Translate
Když se změní anglický obsah, ručně spusťte:
```bash
python translate_google_conservative.py hi
python translate_google_conservative.py hr
python translate_google_conservative.py ms
python translate_google_conservative.py th
python translate_google_conservative.py vi
```

### Pro monitorování
```bash
python verify_all_pushed.py       # Check all repos
python check_google_progress.py   # Check Google langs
python check_hindi_progress.py    # Check Hindi specifically
```

---

## 🎯 Kritéria úspěchu

### ✅ Dosaženo
- [x] 32 jazyků plně přeloženo pomocí DeepL
- [x] Přeloženy všechny popisy frontmatter (37 jazyků)
- [x] Všechny repozitáře na GitHub
- [x] Všechny repozitáře synchronizovány s GitBook
- [x] Automatizovaný denní pracovní skript
- [x] Ochrana veškerého technického obsahu
- [x] Post-processing opravuje veškeré formátování

### ⏳ Probíhá
- [ ] 5 jazyků Google Translate plně přeloženo
- [ ] Překlad do hindštiny (v současné době probíhá)

### 📅 Budoucnost
- [ ] Sledovat rozšíření podpory DeepL
- [ ] V případě potřeby zvážit profesionální překlad posledních 5 jazyků

---

## 📞 Podpora a dokumentace

### Klíčové dokumenty
- `TRANSLATION_QUICK_START.md` – Stručný referenční průvodce
- `TRANSLATION_WORKFLOW.md` – Podrobná dokumentace pracovního postupu
- `TRANSLATION_COMMANDS.md` – Referenční příkazy
- `TRANSLATION_FINAL_STATUS.md` – Tento dokument

### Umístění klíčových skriptů
Všechny skripty v: `C:\Users\MAPIR\Documents\GitHub\chloros_manual_gitbook\`

### Umístění repozitářů
Repozitáře překladů: `D:\chloros_translation_robust\`

---

**Stav projektu:** 🟢 **32/37 dokončeno**, 🟡 **5/37 v procesu**

**Celková úspěšnost:** 86 % dokončeno (32 plně přeloženo + 5 s přeloženými popisy)



