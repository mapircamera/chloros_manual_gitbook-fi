---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Lataa

Lataa uusin versio Chloros:stä Windows:lle, jotta voit aloittaa monispektrisen kuvankäsittelyn.

### Järjestelmävaatimukset

| Vaatimus          | Vähimmäisvaatimukset                         | Suositellut vaatimukset                     |
| -------------------- | ------------------------------- | ------------------------------- |
| **Käyttöjärjestelmä** | Windows 10 (64-bittinen)             | Windows 11 (64-bittinen)             |
| **Prosessori**        | Intel Core i5 tai vastaava     | Intel Core i7 tai parempi         |
| **Muisti (RAM)**     | 8 Gt                             | 16 Gt tai enemmän                    |
| **Näytönohjain**    | DirectX 11 -yhteensopiva           | NVIDIA GPU, 4 Gt+ VRAM       |
| **Tallennustila**          | 2 Gt vapaata tilaa                  | SSD, 10 Gt+ vapaata tilaa       |
| **Näyttö**          | 1920x1080                       | 2560x1440 tai parempi             |
| **Internet**         | Vaaditaan lisenssin aktivointiin | Vaaditaan lisenssin aktivointiin |

{% vihje style=&quot;info&quot; %}
**GPU-kiihdytys**: Chloros+ -käyttäjät, joilla on NVIDIA-GPU (4 Gt+ VRAM), voivat käyttää CUDA-kiihdytystä huomattavasti nopeampaan käsittelyyn.
{% endhint %}

***

## Lataa Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Lataa Chloros täältä</a>

### Uusin vakaa versio

**Chloros-asennusohjelma Windows:lle**

* **Versio**: 1.0.3
* **Julkaisupäivä**: Joulukuu ?, 2025
* **Tiedoston koko**: 1,6 Gt
* **Tiedostotyyppi**: .exe (Windows-asennusohjelma)

#### **Asennusohjeet:**

1. Lataa `CHLOROS INSTALLER - CURRENT VERSION.exe`-tiedosto.
2. Kaksoisnapsauta asennusohjelmaa aloittaaksesi asennuksen.
3. Noudata asennusohjeiden ohjeita.
4. Valitse asennushakemisto (oletus: `C:\Program Files\Chloros\`).
5. Viimeistele asennus ja käynnistä Chloros.
6. Kirjaudu sisään MAPIR Cloud Chloros+ -tililläsi (tai jatka ilmaisella versiolla)

{% hint style=&quot;success&quot; %}
Asennusohjelma lisää `chloros-cli` automaattisesti järjestelmän PATH-polkuun komentorivikäyttöä varten.
{% endhint %}

***

## Lisäresurssit

### Python SDK

Kehittäjille ja automaatiotyönkulkuja varten asenna Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentaatio**: [API: Python SDK](api-python-sdk.md)

**Vaatimukset**: Chloros Desktop on asennettava, Chloros+ -lisenssi vaaditaan.

***

## Sisältö

Chloros-asennus sisältää:

* ✅ **Chloros Desktop GUI** - Täydellinen graafinen käyttöliittymä
* ✅ **Chloros (selain)** - Verkkopohjainen käyttöliittymä heikommille järjestelmille
* ✅ **Chloros CLI** - Komentoriviliittymä (vaatii Chloros+ -lisenssin)
* ✅ **Backend Engine** - Kuvan käsittelyputki
* ✅ **Kameraprofiilit** - Esiasetetut MAPIR-kameramallit

***

## Päivitä Chloros+:aan

Avaa lisäominaisuudet Chloros+-tilauksella:

* 🚀 **Monisäikeinen käsittely** - Käsittele kuvia rinnakkain
* ⚡ **GPU (CUDA) -kiihdytys** - Hyödynnä NVIDIA GPU:n tehoa
* 💻 **CLI-käyttö** - Automatisoi komentorivityökaluilla
* 🐍 **Python SDK** - Ohjelmoitava API-käyttö
* 📱 **Useita laitteita** - Käytä 2–10+ laitteella (riippuen paketista)
* 🧮 **Mukautetut kaavat** - Luo mukautettuja monispektrisiä indeksejä

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Näytä Chloros+ -paketit ja hinnat</a></p>***

## Asennuksen ohjeet

### Vianmääritys

**Asennus epäonnistuu ja näyttöön tulee virheilmoitus:**

* Varmista, että sinulla on järjestelmänvalvojan oikeudet.
* Poista virustorjuntaohjelma väliaikaisesti käytöstä.
* Tarkista, että järjestelmäsi täyttää vähimmäisvaatimukset.

**Sovellus ei käynnisty:**

* Kokeile Chloros (selain) -versiota
* Varmista, että Windows 10/11 (64-bittinen) on asennettu
* Päivitä grafiikkakortin ajurit
* Tarkista Windows Tapahtumienvalvonnasta virheen yksityiskohdat
* Ota yhteyttä tukeen ja toimita virhelokit

**Lisenssin aktivointiongelmat:**

* Varmista, että internetyhteys on aktiivinen
* Tarkista tunnistetiedot osoitteessa [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Tarkista, että palomuuri ei estä Chloros:ää
* Katso yksityiskohtaiset ohjeet kohdasta [Chloros+ Kirjautuminen](chloros+-login.md)

### Tukea

Tarvitsetko apua asennuksessa tai asetusten määrittämisessä?

* 📧 **Sähköposti**: info@mapir.camera
* 🌐 **Verkkosivusto**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentaatio**: [Aloittaminen](./)
* ❓ **UKK**: [Usein kysytyt kysymykset](faq.md)

***

## Muutospäiväkirja

<details>

<summary>Versio 1.0.3</summary>

### **Julkaisupäivä**: Joulukuu ?, 2025

#### Uudet ominaisuudet

* Ensimmäinen julkaisu

#### Parannukset

* Ensimmäinen julkaisu

#### Virhekorjaukset

* Ensimmäinen julkaisu

#### Tunnettuja ongelmia

* Ensimmäinen julkaisu

</details>***

## Lisenssisopimus

**Omistusoikeudellinen ohjelmisto** - Copyright (c) 2025 MAPIR Inc.

Luvaton käyttö, jakelu tai muokkaaminen on kielletty.

**Ilmainen versio**: Saatavilla henkilökohtaiseen ja kaupalliseen käyttöön rajoitetuin ominaisuuksin.

**Chloros+**: Tilauspohjainen lisenssi edistyneille ominaisuuksille ja kaupalliseen käyttöön.
