---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Lataa

Lataa Chloros:n uusin versio, jotta pääset alkuun monispektrikuvien käsittelyssä.

### Järjestelmävaatimukset

#### Windows

| Vaatimus          | Vähimmäisvaatimukset                                              | Suositellut vaatimukset                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Käyttöjärjestelmä** | Windows 10 (64-bittinen)                                  | Windows 11 (64-bittinen)                                  |
| **Prosessori**        | Intel Core i5 tai vastaava                          | Intel Core i7 tai parempi                              |
| **Muisti (RAM)**     | 8 Gt                                                  | 16 Gt tai enemmän                                         |
| **Näytönohjain**    | DirectX 11 -yhteensopiva                                | NVIDIA-näytönohjain, jossa vähintään 4 Gt VRAM-muistia                            |
| **Tallennustila**          | 6 Gt vapaata tilaa                                       | SSD, jossa vähintään 10 Gt vapaata tilaa                            |
| **Näyttö**          | 1920x1080                                            | 2560x1440 tai suurempi                                  |
| **Internet**         | Vaaditaan \[valinnainen] Chloros+ -lisenssin aktivointiin | Vaaditaan \[valinnainen] Chloros+ -lisenssin aktivointiin |

#### Linux amd64 (x86\_64)

| Vaatimus       | Vähimmäisvaatimus                    | Suositeltu               |
| ----------------- | -------------------------- | ------------------------- |
| **Jakelu**  | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+             |
| **Prosessori**     | x86\_64 (Intel/AMD)        | Intel Core i7 tai parempi   |
| **Muisti (RAM)**  | 8 Gt                        | 16 Gt tai enemmän              |
| **Näytönohjain** | Ei tarvita (CPU-käsittely)      | NVIDIA-näytönohjain, jossa vähintään 4 Gt VRAM-muistia |
| **Tallennustila**       | 2 Gt vapaata tilaa             | SSD, jossa vähintään 10 Gt vapaata tilaa       |
| **Python**        | Python 3.7+ (SDK:lle)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Vaatimus      | Vähimmäisvaatimus                      | Suositeltu                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Alusta**     | NVIDIA Jetson ja JetPack 6 | Jetson Orin NX 16 Gt tai AGX Orin |
| **Muisti (RAM)** | 8 Gt (jaettu GPU/CPU)         | 16 Gt+ jaettu                    |
| **Tallennustila**      | 2 Gt vapaata tilaa               | NVMe SSD, jossa 10 Gt+ vapaata        |
| **Python**       | Python 3.7+ (SDK:lle)        | Python 3.10+                    |

{% hint style="info" %}
**GPU-kiihdytys**: NVIDIA-näytönohjaimia käyttävät Chloros+ -käyttäjät voivat käyttää CUDA-kiihdytystä huomattavasti nopeampaan käsittelyyn. Tämä toimii sekä Windows:ssä (pöytätietokoneiden näytönohjaimet) että Linux:ssä (pöytätietokoneiden näytönohjaimet ja NVIDIA Jetson). Chloros+ -käyttäjät saavat myös monisäikeisen käsittelyn maksimaalisen nopeuden saavuttamiseksi.
{% endhint %}

***

## Lataa Chloros

### Uusin vakaa julkaisu (23. maaliskuuta 2026): Versio 1.1.0

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Lataa Chloros Windows:lle (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Lataa Chloros Linux amd64:lle (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Lataa Chloros Linux arm64 / Jetsonille (.deb)</a>

#### Windows-asennusohjelma (GUI + CLI + taustaohjelma)

* **Tiedostotyyppi**: .exe (Windows-asennusohjelma)**Asennusohjeet:**

1. Lataa yllä oleva .exe-tiedosto
2. Kaksoisnapsauta asennusohjelmaa aloittaaksesi asennuksen
3. Noudata asennusohjeiden ohjeita
4. Valitse asennuskansio (oletus: `C:\Program Files\[USER]\Chloros\`)
5. Viimeistele asennus ja käynnistä Chloros tai Chloros CLI
6. Kirjaudu sisään [MAPIR Cloud Chloros+ -tililläsi](https://cloud.mapir.camera/pricing) (tai jatka ilmaisversiolla)

{% hint style="success" %}
Asennusohjelma lisää automaattisesti `chloros-cli` järjestelmän PATH-polkuun komentorivikäyttöä varten.
{% endhint %}

#### Linux amd64 (.deb-paketti — CLI + Backend)

* **Tiedostotyyppi**: .deb (Debian/Ubuntu-paketti)
* **Arkkitehtuuri**: x86\_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (.deb-paketti — CLI + Backend)

* **Tiedostotyyppi**: .deb (JetPack 6)
* **Arkkitehtuuri**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Katso [Linux-asennus](linux/linux-installation.md) saadaksesi yksityiskohtaiset asennusohjeet ja [NVIDIA Jetson -opas](linux/nvidia-jetson-guide.md) saadaksesi Jetson-kohtaisia ohjeita.

#### Python SDK (kaikki alustat)

```bash
pip install chloros-sdk
```

Katso dokumentaatio kohdasta [API : Python SDK](api-python-sdk.md).

{% hint style="info" %}
**Linux-käyttäjät**: `.deb`-paketti asentaa CLI:n ja taustapalvelimen. Python SDK asennetaan erikseen pip:n kautta. Linux:lle ei ole graafista käyttöliittymää — kaikki vuorovaikutus tapahtuu CLI:n tai SDK:n kautta.
{% endhint %}

***

## Lisäresurssit

### Python SDK

Kehittäjille ja automaatiotyönkulkuja varten asenna Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentaatio**: [API: Python SDK](api-python-sdk.md)**Vaatimukset**: Chloros on asennettava (Windows-asennusohjelma tai Linux `.deb`-paketti), Chloros+ -lisenssin kirjautuminen vaaditaan***

## Sisältö

### Windows-asennusohjelma

* ✅ **Chloros GUI** - Täysimittainen graafinen käyttöliittymä
* ✅ **Chloros CLI** - Komentoriviliittymä (vaatii Chloros+ lisenssin)
* ✅ **Chloros Backend** - Käsittelymoottori
* ✅ **Kameraprofiilit** - Esiasetetut MAPIR-kameramallit

### Linux .deb-paketti

* ✅ **Chloros CLI** - Komentoriviliittymä (vaatii Chloros+ -lisenssin)
* ✅ **Chloros Backend** - Käsittelymoottori
* ✅ **Kameraprofiilit** - Esiasetetut MAPIR-kameramallit
* ❌ Ei graafista käyttöliittymää — Linux on vain päättömäinen CLI/SDK

### Python SDK (pip, kaikki alustat)

* ✅ **Chloros SDK** - Python API (vaatii Chloros+ -lisenssin)***

## Päivitä Chloros+:aan

Avaa lisäominaisuudet Chloros+-tilauksella:

* 🚀 **Monisäikeinen käsittely** - Käsittele kuvia rinnakkain
* ⚡ **GPU (CUDA) -kiihdytys** - Hyödynnä NVIDIA-GPU:n tehoa
* 💻 **CLI-käyttö** – Automatisoi komentorivityökaluilla
* 🐍 **Python SDK** – Ohjelmoitu API-käyttö
* 📱 **Useita laitteita** – Käytä 2–10+ laitteella (riippuu paketista)
* **🐻 Kehittynyt tekstuuritietoinen debayer-menetelmä** – korkealaatuinen reuna-tietoinen debayer yhdistettynä AI/ML-kohinanpoistomalliin, joka poistaa lähes kaiken debayer-kohinan.
* 🧮 **Mukautetut kaavat** – Luo mukautettuja monispektrisiä indeksejä

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Katso Chloros+ -paketit ja hinnat</a></p>***

## Asennusohjeet

### Vianmääritys

**Asennus epäonnistuu ja näyttöön tulee virheilmoitus:**

* Varmista, että sinulla on järjestelmänvalvojan oikeudet
* Poista virustorjuntaohjelma väliaikaisesti käytöstä
* Tarkista, että järjestelmäsi täyttää vähimmäisvaatimukset

**Sovellus ei käynnisty (Windows):**

* Varmista, että Windows 10/11 (64-bittinen) on asennettu
* Päivitä näytönohjaimen ajurit
* Tarkista Windows Tapahtumienvalvonta virheen yksityiskohdista
* Ota yhteyttä tukeen virhelokien kanssa

**CLI ei käynnisty (Linux):**

* Varmista, että `.deb`-paketti on asennettu oikein: `dpkg -l | grep chloros`
* Tarkista käyttöoikeudet: `sudo chmod +x /usr/bin/chloros-cli`
* Suorita diagnostiikka: `chloros-cli selftest`
* Tarkista puuttuvat kirjastot: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Lisenssin aktivointiongelmat:**

* Varmista, että internetyhteys on aktiivinen
* Tarkista tunnistetiedot osoitteessa [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Tarkista, ettei palomuuri estä Chloros
* Katso yksityiskohtaiset ohjeet kohdasta [Chloros+ Kirjautuminen](chloros+-login.md)

### Tukea

Tarvitsetko apua asennuksessa tai käyttöönotossa?

* 📧 **Sähköposti**: info@mapir.camera
* 🌐 **Verkkosivusto**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Ohjeet**: [Aloitusopas](./)
* ❓ **UKK**: [Usein kysyttyjä kysymyksiä](faq.md)***

## Muutosloki

<details>

<summary>Versio 1.1.0 (Uusin)</summary>

**Julkaisupäivä: maaliskuu 2026**

**Uudet ominaisuudet*** **Linux-tuki** — Natiivi CLI ja SDK Linux amd64 (x86\_64) ja arm64 (NVIDIA Jetson JetPack 6) -alustoille. Asenna `.deb`-pakettien kautta.
* **NVIDIA Jetson -tuki** — Optimoitu käsittely Jetson Nano-, Orin Nano-, Orin NX- ja AGX Orin -reunalaitteille.
* **Dynaaminen laskennan mukautus** — Automaattinen laitteiston tunnistus ja käsittelystrategian optimointi. Chloros mukautuu laitteistoosi Jetson Nanosta monen GPU:n työasemaan.
* **4-säikeinen käsittelyputki** — Samanaikaiset tunnistus-, kalibrointi-, käsittely- ja vientisäikeet dynaamisella GPU-muistin allokoinnilla.
* **Uudet CLI-komennot** — `selftest` (järjestelmän diagnostiikka) ja `update` (Linux-päivitysten hallinta).
* **Uudet CLI-prosessilippuja** — `--debayer` (vakio/tekstuuritunnistava), `--indices` (määritä indeksit), `--target` (etsi ensin kohdekansiosta nopeampaa tunnistusta varten).
* **Uudet GUI-valikkokohdat** — Lisää tiedostoja, Lisää kansio ja Käynnistä/Lopeta käsittely ovat nyt käytettävissä päävalikon pudotusvalikosta.**Parannukset**

* Alustojen välinen taustapalvelimen automaattinen tunnistus (polut Windows ja Linux)
* Parannettu SDK `get_status()` säikeittäisellä edistymisen seurannalla
* Uudet SDK-poikkeukset: `ChlorosConfigurationError`, `ChlorosAuthenticationError`
* Lämmönhallinta ja adaptiivinen kuristaminen NVIDIA Jetsonille
* Automaattinen muistinhallinta, jossa OOM-tilanteessa siirrytään laattamaiseen GPU-käsittelyyn

</details>

<details>

<summary>Versio 1.0.5</summary>

**Julkaisupäivä: 10. helmikuuta 2026**

**Uudet ominaisuudet*** **Texture Aware -debayer-menetelmä \[Vain Chloros+] -** Texture Aware käyttää korkealaatuista reuna-tietoista debayer-menetelmää yhdistettynä AI/ML-kohinanpoistomalliin, joka poistaa lähes kaiken debayer-kohinan.
* **Tuki T4P-kalibrointikohteille*** **Nopeampi Chloros+ GPU-käsittely, parempi muistinhallinta**

**Virhekorjaukset*** Täysin uusi käyttöliittymä (GUI), jonka pitäisi nyt toimia kaikilla Windows-tietokoneilla.

</details>

<details>

<summary>Versio 1.0.4</summary>

**Julkaisupäivä: 5. tammikuuta 2026**

**Uudet ominaisuudet*** **Kuva/metatiedot-kytkin**: Tiedostoselaimeen on lisätty kytkin, jolla valitun kuvan metatiedot voidaan tarkastella taulukkomuodossa kuvaruudukon sijaan
* **Kuvaruudukon zoomausliukusäädin**: Uusi käyttöliittymän liukusäädin pikkukuvien koon säätämiseen (tukee myös CTRL + hiiren rullaa)
* **Kuvaruudukon vientipainikkeet**: Ylärivillä olevat painikkeet, joilla voi vaihtaa pikkukuvien muotoa JPG:stä käsiteltyihin vientimuotoihin (Targets, Reflectance, Index, LUT)
* **Kartta-välilehti**: Uusi interaktiivinen 2D-kartta, joka näyttää kuvien GPS-sijaintimerkit
  * Tukee Google Mapsia ja ESRI-karttalaattoja (valitsee automaattisesti parhaan karttapalvelun zoomausasteen saatavuuden perusteella)
  * Pikkukuvien esikatselu karttamerkkeihin osoittamalla hiirellä

**Virhekorjaukset*** Parannettu tuki Chloros:n asentamiselle muille kuin englanninkielisille tietokoneille

</details>

<details>

<summary>Versio 1.0.3</summary>

**Julkaisupäivä: 20. joulukuuta 2025**

**Uudet ominaisuudet*** Ensimmäinen julkaisu

**Parannukset*** Ensimmäinen julkaisu

**Virhekorjaukset*** Ensimmäinen julkaisu

**Tiedossa olevat ongelmat*** Ensimmäinen julkaisu

</details>***

## Käyttöoikeussopimus**Omistusoikeudellinen ohjelmisto** - Copyright (c) 2026 MAPIR Inc.

Luvaton käyttö, jakelu tai muokkaaminen on kielletty.

**Ilmainen versio**: Saatavilla henkilökohtaiseen ja kaupalliseen käyttöön rajoitetuin ominaisuuksin**Chloros+**: Tilauspohjainen lisenssi edistyneille ominaisuuksille ja kaupalliseen käyttöön
