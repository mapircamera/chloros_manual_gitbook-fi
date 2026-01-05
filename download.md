---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Lataa

Lataa uusin versio Chloros:stä, jotta voit aloittaa monispektrisen kuvankäsittelyn.

### Järjestelmävaatimukset

| Vaatimus          | Vähimmäisvaatimukset                         | Suositellut vaatimukset                     |
| -------------------- | ------------------------------- | ------------------------------- |
| **Käyttöjärjestelmä** | Windows 10 (64-bittinen)             | Windows 11 (64-bittinen)             |
| **Prosessori**        | Intel Core i5 tai vastaava     | Intel Core i7 tai parempi         |
| **Muisti (RAM)**     | 8 Gt                             | 16 Gt tai enemmän                    |
| **Näytönohjain**    | DirectX 11 -yhteensopiva           | NVIDIA GPU, 4 Gt+ VRAM       |
| **Tallennustila**          | 6 Gt vapaata tilaa                  | SSD, 10 Gt+ vapaata tilaa       |
| **Näyttö**          | 1920x1080                       | 2560x1440 tai parempi             |
| **Internet**         | Vaaditaan lisenssin aktivointiin | Vaaditaan lisenssin aktivointiin |

{% hint style=&quot;info&quot; %}
**GPU-kiihdytys**: Chloros+ -käyttäjät, joilla on NVIDIA-GPU (4 Gt+ VRAM), voivat käyttää CUDA-kiihdytystä huomattavasti nopeampaan käsittelyyn. Chloros+ -käyttäjät saavat myös monisäikeisen käsittelyn maksimaalisen nopeuden saavuttamiseksi.
{% endhint %}

***

## Lataa Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Lataa Chloros täältä</a>

### Uusin vakaa versio

**Chloros-asennusohjelma Windows:lle*** **Versio**: 1.0.4
* **Julkaisupäivä**: 5. tammikuuta 2026
* **Tiedoston koko (lataus)**: 1,8 Gt
* **Tiedoston koko (asennettuna)**: 5,7 Gt
* **Tiedostotyyppi**: .exe (Windows-asennusohjelma)

#### **Asennusohjeet:**

1. Lataa tiedosto `CHLOROS INSTALLER - CURRENT VERSION.exe`
2. Kaksoisnapsauta asennusohjelmaa aloittaaksesi asennuksen
3. Noudata asennusohjeiden ohjeita
4. Valitse asennuskansio (oletus: `C:\Program Files\[USER]\Chloros\`)
5. Suorita asennus loppuun ja käynnistä Chloros, Chloros (selain) tai Chloros CLI
6. Kirjaudu sisään [MAPIR Cloud Chloros+ -tililläsi](https://cloud.mapir.camera/pricing) (tai jatka ilmaisella versiolla)

{% vihje style=&quot;success&quot; %}
Asennusohjelma lisää automaattisesti `chloros-cli` järjestelmän PATH-polkuun komentorivikäyttöä varten.
{% endhint %}

***

## Lisäresurssit

### Python SDK

Kehittäjille ja automaatiotyönkulkuja varten asenna Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentaatio**: [API: Python SDK](api-python-sdk.md)**Vaatimukset**: Chloros Desktop on oltava asennettuna, Chloros+ -lisenssin kirjautuminen vaaditaan.***

## Sisältö

Chloros-asennus sisältää:

* ✅ **Chloros** - Täydellinen graafinen käyttöliittymä
* ✅ **Chloros (selain)** - Verkkopohjainen käyttöliittymä heikompitehoisille järjestelmille
* ✅ **Chloros CLI** - Komentoriviliittymä (vaatii Chloros+ -lisenssin)
* ✅ **Chloros SDK** - Python API (vaatii Chloros+ lisenssin)
* ✅ **Kameraprofiilit** - Esiasetetut MAPIR kameramallit***

## Päivitä Chloros+:aan

Avaa lisäominaisuudet Chloros+ -tilauksella:

* 🚀 **Monisäikeinen käsittely** - Käsittele kuvia rinnakkain
* ⚡ **GPU (CUDA) -kiihdytys** - Hyödynnä NVIDIA GPU:n tehoa
* 💻 **CLI-käyttö** – Automatisoi komentorivityökaluilla
* 🐍 **Python SDK** – Ohjelmoitava API-käyttö
* 📱 **Useita laitteita** - Käytä 2–10+ laitteessa (riippuen paketista)
* 🧮 **Mukautetut kaavat** - Luo mukautettuja monispektrisiä indeksejä

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Katso Chloros+ -paketit ja hinnat</a></p>***

## Asennuksen ohjeet

### Vianmääritys

**Asennus epäonnistuu ja näyttöön tulee virheilmoitus:**

* Varmista, että sinulla on järjestelmänvalvojan oikeudet
* Poista virustorjuntaohjelma väliaikaisesti käytöstä
* Tarkista, että järjestelmäsi täyttää vähimmäisvaatimukset

**Sovellus ei käynnisty:**

* Kokeile Chloros (selain) -versiota
* Varmista, että Windows 10/11 (64-bittinen) on asennettu
* Päivitä grafiikkakortin ajurit
* Tarkista Windows Event Viewer -tapahtumalokista virheen yksityiskohdat
* Ota yhteyttä tukeen ja toimita virhelokit

**Lisenssin aktivointiongelmat:**

* Varmista, että internetyhteys on aktiivinen
* Tarkista tunnistetiedot osoitteessa [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Tarkista, että palomuuri ei estä Chloros:ää
* Katso yksityiskohtaiset ohjeet osoitteesta [Chloros+ Kirjautuminen](chloros+-login.md)

### Tukipalvelut

Tarvitsetko apua asennuksessa tai asetusten määrittämisessä?

* 📧 **Sähköposti**: info@mapir.camera
* 🌐 **Verkkosivusto**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentaatio**: [Aloittaminen](./)
* ❓ **UKK**: [Usein kysytyt kysymykset](faq.md)***

## Muutospäiväkirja

<details>

<summary>Versio 1.0.4</summary>

#### **Julkaisupäivä**: 5. tammikuuta 2026**Uudet ominaisuudet*** **Kuva/metatieto-kytkin**: Lisätty kytkin tiedostoselaimeen, jotta valitun kuvan metatiedot voidaan tarkastella taulukossa kuvaruudukon sijaan
* **Kuvaruudukon zoomausliukusäädin**: Uusi käyttöliittymän liukusäädin pikkukuvien koon säätämiseen (tukee myös CTRL + hiiren rullaa)
* **Kuvaruudukon vientipainikkeet**: Painikkeet ylärivillä pikkukuvien vaihtamiseksi JPG-muodosta käsiteltyihin vientimuotoihin (kohteet, heijastavuus, indeksi, LUT)
* **Kartta-välilehti**: Uusi interaktiivinen 2D-kartta, joka näyttää kuvien GPS-sijaintimerkit.
  * Tukee Google Maps- ja ESRI-karttaruutuja (valitsee automaattisesti parhaan ruutupalvelun zoomausasteen saatavuuden perusteella).
  * Pienoiskuvien esikatselu karttamerkkeillä hiiren osoittimen ollessa niiden päällä.

**Virhekorjaukset*** Parannettu tuki Chloros:n asentamiselle muunkielisille tietokoneille.

</details>

<details>

<summary>Versio 1.0.3</summary>

#### **Julkaisupäivä**: 20. joulukuuta 2025**Uudet ominaisuudet*** Ensimmäinen julkaisu

**Parannukset*** Ensimmäinen julkaisu

**Virhekorjaukset*** Ensimmäinen julkaisu

**Tunnetut ongelmat*** Ensimmäinen julkaisu

</details>***

## Lisenssisopimus**Omistusoikeudellinen ohjelmisto** - Copyright (c) 2025 MAPIR Inc.

Luvaton käyttö, jakelu tai muokkaaminen on kielletty.

**Ilmainen versio**: Saatavilla henkilökohtaiseen ja kaupalliseen käyttöön rajoitetuin ominaisuuksin.**Chloros+**: Tilauspohjainen lisenssi edistyneille ominaisuuksille ja kaupalliseen käyttöön.
