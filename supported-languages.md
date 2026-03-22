# Tuetut kielet

Chloros tarjoaa täyden käyttöliittymätuen **38 kielellä ympäri maailmaa**, minkä ansiosta se on käytettävissä käyttäjille kaikkialla maailmassa. Voit vaihtaa kieltä välittömästi kaikissa käyttöliittymissä: Desktop, Browser, CLI ja Python SDK.

Chloros tukee seuraavia kieliä:

| # | Kieli | Alkuperäinen nimi | CLI-koodi |
|---|----------|-------------|----------|
| 1 | 🇺🇸 Englanti | English | `en` |
| 2 | 🇪🇸 Espanja | Español | `es` |
| 3 | 🇵🇹 Portugali | Português | `pt` |
| 4 | 🇫🇷 Ranska | Français | `fr` |
| 5 | 🇩🇪 Saksa | Deutsch | `de` |
| 6 | 🇮🇹 Italia | Italiano | `it` |
| 7 | 🇯🇵 Japani | 日本語 | `ja` |
| 8 | 🇰🇷 Korea | 한국어 | `ko` |
| 9 | 🇨🇳 Kiina (yksinkertaistettu) | 简体中文 | `zh` |
| 10 | 🇹🇼 Kiina (perinteinen) | 繁體中文 | `zh-TW` |
| 11 | 🇷🇺 Venäjä | Русский | `ru` |
| 12 | 🇳🇱 Hollanti | Nederlands | `nl` |
| 13 | 🇸🇦 Arabia | العربية | `ar` |
| 14 | 🇵🇱 Puola | Polski | `pl` |
| 15 | 🇹🇷 Turkki | Türkçe | `tr` |
| 16 | 🇮🇳 Hindi | हिंदी | `hi` |
| 17 | 🇮🇩 Indonesia | Bahasa Indonesia | `id` |
| 18 | 🇻🇳 Vietnam | Tiếng Việt | `vi` |
| 19 | 🇹🇭 Thai | ไทย | `th` |
| 20 | 🇸🇪 Ruotsi | Svenska | `sv` |
| 21 | 🇩🇰 Tanska | Dansk | `da` |
| 22 | 🇳🇴 Norja | Norsk | `no` |
| 23 | 🇫🇮 Suomi | Suomi | `fi` |
| 24 | 🇬🇷 Kreikka | Ελληνικά | `el` |
| 25 | 🇨🇿 Tšekki | Čeština | `cs` |
| 26 | 🇭🇺 Unkari | Magyar | `hu` |
| 27 | 🇷🇴 Romania | Română | `ro` |
| 28 | 🇺🇦 Ukraina | Українська | `uk` |
| 29 | 🇧🇷 Brasilian portugali | Português Brasileiro | `pt-BR` |
| 30 | 🇭🇰 Kantoninkiina | 粵語 | `zh-HK` |
| 31 | 🇲🇾 Malaiji | Bahasa Melayu | `ms` |
| 32 | 🇸🇰 Slovakki | Slovenčina | `sk` |
| 33 | 🇧🇬 Bulgaria | Български | `bg` |
| 34 | 🇭🇷 Kroatia | Hrvatski | `hr` |
| 35 | 🇱🇹 Liettua | Lietuvių | `lt` |
| 36 | 🇱🇻 Latvia | Latviešu | `lv` |
| 37 | 🇪🇪 Viro | Eesti | `et` |
| 38 | 🇸🇮 Slovenia | Slovenščina | `sl` |

## Kielen vaihtaminen

### Chloros-työpöydällä/selaimessa

1. Avaa sovelluksen asetukset
2. Siirry kielivalintavalikkoon
3. Valitse haluamasi kieli luettelosta
4. Käyttöliittymä päivittyy välittömästi

### Chloros CLI

Käytä `language`-komentoa CLI-käyttöliittymän kielen tarkastelemiseen tai muuttamiseen:

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

Lisätietoja on [CLI-ohjeissa](CLI.md).

### Chloros Python SDK

Määritä kieliparametri alustaessasi SDK:n, jotta saat viestit ja tulosteet haluamallasi kielellä.

## Kattavuus

Kaikki 38 kieltä tuetaan täysin seuraavissa:

* **Chloros Desktop** - Täydellinen käyttöliittymän käännös
* **Chloros Browser** - Verkkokäyttöliittymä kaikilla kielillä
* **Chloros CLI** - Komentoriviliittymä ja tulosteviestit
* **Chloros Python SDK** - API-viestit ja dokumentaatio

Kielituki varmistaa, että käyttäjät ympäri maailmaa voivat työskennellä tehokkaasti omalla äidinkielellään ilman esteitä.
