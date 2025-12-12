# Tuetut kielet

Chloros tarjoaa täyden käyttöliittymätuen **38 kielellä ympäri maailmaa**, joten se on käytettävissä käyttäjille kaikkialla maailmassa. Voit vaihtaa kieltä välittömästi kaikissa käyttöliittymissä: työpöytä, selain, CLI ja Python SDK.

Chloros tukee seuraavia kieliä:

| # | Kieli | Alkuperäinen nimi | CLI-koodi |
|---|----------|-------------|----------|
| 1 | 🇺🇸 Englanti | Englanti | `en` |
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
| 19 | 🇹🇭 Thaimaalainen | ไทย | `th` |
| 20 | 🇸🇪 Ruotsalainen | Svenska | `sv` |
| 21 | 🇩🇰 Tanskalainen | Dansk | `da` |
| 22 | 🇳🇴 Norja | Norsk | `no` |
| 23 | 🇫🇮 Suomi | Suomi | `fi` |
| 24 | 🇬🇷 Kreikka | Ελληνικά | `el` |
| 25 | 🇨🇿 Tšekki | Čeština | `cs` |
| 26 | 🇭🇺 Unkari | Magyar | `hu` |
| 27 | 🇷🇴 Romania | Română | `ro` |
| 28 | 🇺🇦 Ukraina | Українська | `uk` |
| 29 | 🇧🇷 Brasilian portugali | Português Brasileiro | `pt-BR` |
| 30 | 🇭🇰 Kantoni | 粵語 | `zh-HK` |
| 31 | 🇲🇾 Malaiji | Bahasa Melayu | `ms` |
| 32 | 🇸🇰 Slovakki | Slovenčina | `sk` |
| 33 | 🇧🇬 Bulgarian | Български | `bg` |
| 34 | 🇭🇷 Croatian | Hrvatski | `hr` |
| 35 | 🇱🇹 Liettua | Lietuvių | `lt` |
| 36 | 🇱🇻 Latvia | Latviešu | `lv` |
| 37 | 🇪🇪 Viron kieli | Eesti | `et` |
| 38 | 🇸🇮 Slovenian kieli | Slovenščina | `sl` |

## Kielen vaihtaminen

### Chloros-työpöydällä/selaimessa

1. Avaa sovelluksen asetukset.
2. Siirry kielivalikkoon.
3. Valitse haluamasi kieli luettelosta.
4. Käyttöliittymä päivittyy välittömästi.

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

Lisätietoja on [CLI-dokumentaatiossa](CLI.md).

### Chloros Python SDK

Aseta kieliparametri SDK:n alustuksen yhteydessä, jotta saat viestit ja tulosteet haluamallasi kielellä.

## Kattavuus

Kaikki 38 kieltä ovat täysin tuettuja seuraavissa:

* **Chloros Desktop** - Täydellinen GUI-käännös
* **Chloros Browser** - Verkkokäyttöliittymä kaikilla kielillä
* **Chloros CLI** - Komentoriviliittymä ja tulostusviestit
* **Chloros Python SDK** - API-viestit ja dokumentaatio

Kielituki varmistaa, että käyttäjät ympäri maailmaa voivat työskennellä tehokkaasti omalla äidinkielellään ilman esteitä.
