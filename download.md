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
| **Muisti (RAM)**     | 8 GB                                                  | 16 GB tai enemmän                                         |
| **Näytönohjain**    | DirectX 11 -yhteensopiva                                | NVIDIA-näytönohjain, jossa vähintään 4 GB VRAM-muistia                            |
| **Tallennustila**          | 6 GB vapaata tilaa                                       | SSD, jossa vähintään 10 GB vapaata tilaa                            |
| **Näyttö**          | 1920x1080                                            | 2560x1440 tai suurempi                                  |
| **Internet**         | Vaaditaan \[valinnainen] Chloros+ -lisenssin aktivointiin | Vaaditaan \[valinnainen] Chloros+ -lisenssin aktivointiin |

#### Linux amd64 (x86\_64)

| Vaatimus       | Vähimmäisvaatimus                    | Suositeltu               |
| ----------------- | -------------------------- | ------------------------- |
| **Jakelu**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Prosessori**     | x86\_64 (Intel/AMD)        | Intel Core i7 tai parempi   |
| **Muisti (RAM)**  | 8 GB                        | 16 GB tai enemmän              |
| **Näytönohjain** | Ei tarvita (käsitellään prosessorilla)      | NVIDIA-näytönohjain, jossa vähintään 4 GB VRAM-muistia |
| **Tallennustila**       | 2 GB vapaata tilaa             | SSD, jossa vähintään 10 GB vapaata tilaa       |
| **Python**        | Python 3.7+ (mallille SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Vaatimus      | Vähimmäisvaatimus                      | Suositeltu                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Alusta**     | NVIDIA Jetson ja JetPack 6 | Jetson Orin NX 16 GB tai AGX Orin |
| **Muisti (RAM)** | 8 Gt (jaettu GPU/CPU)         | 16 Gt+ jaettu                    |
| **Tallennustila**      | 2 Gt vapaata tilaa               | NVMe SSD, jossa vähintään 10 Gt vapaata tilaa        |
| **Python**       | Python 3.7+ (mallille SDK)        | Python 3.10+                    |

{% hint style="info" %}
**GPU-kiihdytys**: NVIDIA-grafiikkakortteja käyttävät Chloros+ -käyttäjät voivat hyödyntää CUDA-kiihdytystä huomattavasti nopeampaan käsittelyyn. Tämä toimii sekä Windows:llä (pöytätietokoneiden grafiikkapiirit) että Linux:llä (pöytätietokoneiden grafiikkapiirit ja NVIDIA Jetson). Chloros+-käyttäjät saavat lisäksi käyttöönsä monisäikeisen käsittelyn, joka takaa maksimaalisen nopeuden.
{% endhint %}

***

## Lataa Chloros

### Uusin vakaa versio: Versio 1.2.0

<!-- NOLAN: replace installer links + release date for 1.2.0 — the three download buttons below still point at the 1.1.0 Google Drive files, and the release date needs to be added to the heading above. -->



### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Lataa Chloros mallille Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Lataa Chloros Linux amd64:lle (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Lataa Chloros Linux:lle (arm64 / Jetson, .deb)</a>

#### Windows-asennusohjelma (GUI + CLI + taustapalvelu)

* **Tiedostotyyppi**: .exe (Windows-asennusohjelma)**Asennusohjeet:**

1. Lataa yllä oleva .exe-tiedosto
2. Käynnistä asennus kaksoisnapsauttamalla asennusohjelmaa
3. Noudata asennusohjeiden ohjeita
4. Valitse asennuskansio (oletus: `C:\Program Files\MAPIR\Chloros\`)
5. Viimeistele asennus ja käynnistä Chloros tai Chloros CLI
6. Kirjaudu sisään [MAPIR Cloud Chloros+ -tililläsi](https://cloud.mapir.camera/pricing) (tai jatka ilmaisversiolla)

{% hint style="success" %}
Asennusohjelma lisää automaattisesti `chloros-cli` järjestelmän PATH-muuttujaan komentorivikäyttöä varten.
{% endhint %}

#### Linux amd64 (.deb-paketti — CLI + taustapalvelu)

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

Katso [Linux:n asennusohjeet](linux/linux-installation.md) saadaksesi yksityiskohtaiset asennusohjeet ja [NVIDIA Jetson -opas](linux/nvidia-jetson-guide.md) saadaksesi Jetson-laitteille tarkoitettuja ohjeita.

#### Python SDK (kaikki alustat)

Jokaiseen asennusohjelmaan sisältyy vastaava `chloros_sdk`-wheel, joten SDK-versio vastaa aina asennettua GUI:ta/CLI:ta/taustapalvelinta. Windows:ssa asennusohjelma asentaa sen järjestelmäänne Python automaattisesti; versiossa Linux asennusohjelma sijoittaa wheel-tiedoston polkuun `/usr/lib/chloros/sdk/` ja tulostaa asennuskomennon:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Pelkästään pip-palvelimille (joihin ei ole asennettu Chloros-pakettia) SDK löytyy myös PyPI:stä:

```bash
pip install chloros-sdk
```

Katso [API : Python SDK](api-python-sdk.md) ja [SDK-viite](reference/sdk-reference.md) dokumentaatiota varten.

{% hint style="info" %}
**Linux-käyttäjät**: `.deb`-paketti asentaa CLI:n ja taustapalvelimen. Linux:lle ei ole graafista käyttöliittymää — kaikki vuorovaikutus tapahtuu CLI:n tai SDK:n kautta.
{% endhint %}

***

## Lisäresurssit

### Python SDK

Kehittäjille ja automaatiotyönkulkuja varten asenna Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Ohjeet**: [API: Python SDK](api-python-sdk.md)**Vaatimukset**: Chloros on oltava asennettuna (Windows-asennusohjelma tai Linux `.deb`-paketti), vaaditaan Chloros+ -lisenssin kirjautumistiedot***

## Sisältö

### Windows-asennusohjelma

* ✅ **Chloros-käyttöliittymä** – Täydellinen graafinen käyttöliittymä
* ✅ **Chloros CLI** - Komentoriviliittymä (vaatii Chloros+-lisenssin)
* ✅ **Chloros-taustaprosessi** – Käsittelymoottori
* ✅ **Kameraprofiilit** – Valmiiksi määritetyt MAPIR-kameramallit

### Linux .deb-paketti

* ✅ **Chloros CLI** – Komentoriviliitäntä (vaatii Chloros+ -lisenssin)
* ✅ **Chloros-taustapalvelu** – Käsittelymoottori
* ✅ **Kameraprofiilit** – Esiasetetut MAPIR-kameramallit
* ❌ Ei graafista käyttöliittymää — Linux on pelkästään päättömäinen CLI/SDK

### Python SDK (PIP, kaikki alustat)

* ✅ **Chloros SDK** – Python API (vaatii Chloros+ -lisenssin)***

## Päivitä Chloros+ -versioon

Avaa edistyneet ominaisuudet Chloros+ -tilauksella:

* 🚀 **Monisäikeinen käsittely** – Käsittele kuvia rinnakkain
* ⚡ **GPU (CUDA) -kiihdytys** – Hyödynnä NVIDIA-grafiikkaprosessorin tehoa
* 💻 **CLI-käyttöoikeus** - Automatisoi komentorivityökaluilla
* 🐍 **Python SDK** – Ohjelmoitu API-käyttö
* 📱 **Useita laitteita** – Käytä 2–10+ laitteella (riippuu tilauksesta)
* **🐻 Edistyksellinen tekstuuritietoinen debayer-menetelmä** – korkealaatuinen, reunoja huomioiva debayer yhdistettynä tekoäly-/koneoppimismalliin, joka poistaa lähes kaiken debayer-kohinan.
* 🧮 **Mukautetut kaavat** – Luo mukautettuja monispektrisiä indeksejä

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Katso Chloros+ -paketit ja hinnat</a></p>***

## Asennusohjeet

### Vianmääritys

**Asennus epäonnistuu ja näyttöön tulee seuraava virheilmoitus:**

* Varmista, että sinulla on järjestelmänvalvojan oikeudet
* Poista virustorjuntaohjelma väliaikaisesti käytöstä
* Tarkista, että laitteesi täyttää järjestelmän vähimmäisvaatimukset

**Sovellus ei käynnisty (Windows):**

* Varmista, että Windows 10/11 (64-bittinen) on asennettu
* Päivitä näytönohjaimen ajurit
* Tarkista virheen tiedot Windows-tapahtumalokista
* Ota yhteyttä tukeen ja lähetä virhelokit

**CLI ei käynnisty (Linux):**

* Varmista, että `.deb`-paketti on asennettu oikein: `dpkg -l | grep chloros`
* Tarkista käyttöoikeudet: `sudo chmod +x /usr/bin/chloros-cli`
* Suorita vianmääritys: `chloros-cli selftest`
* Tarkista, puuttuuko kirjastoja: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Lisenssin aktivointiongelmat:**

* Varmista, että internetyhteys on aktiivinen
* Tarkista tunnistetiedot osoitteessa [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Tarkista, ettei palomuuri estä Chloros:ää
* Katso yksityiskohtaiset ohjeet kohdasta [Chloros+ Kirjautuminen](chloros+-login.md)

### Tuen saaminen

Tarvitsetko apua asennuksessa tai käyttöönotossa?

* 📧 **Sähköposti**: info@mapir.camera
* 🌐 **Verkkosivusto**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Ohjeet**: [Aloitusopas](./)
* ❓ **UKK**: [Usein kysytyt kysymykset](faq.md)***

## Ohjelmistopäivitykset

Chloros tarkistaa päivitykset, ilmoittaa, kun uusi versio on saatavilla, ja ohjaa tälle lataussivulle — päivitys suoritetaan ajamalla uusi allekirjoitettu asennusohjelma. Asetuksesi ja projektisi säilyvät päivitysten jälkeen. Linux- ja Jetson-versioissa `chloros-cli update` tarkistaa, onko uudempaa versiota saatavilla, ja tarjoaa mahdollisuuden ladata ja asentaa vastaavan `.deb` (tämä komento on käytettävissä vain versiossa Linux).

***

## Muutosloki**Versio 1.2.0 (uusin)**— katso**Uutta Chloros 1.2.0:ssa** [Aloitusoppaasta](./) sivulta täydellinen ominaisuusluettelo.

<details>

<summary>Versio 1.0.5</summary>

**Julkaisupäivä: 10. helmikuuta 2026**

**Uudet ominaisuudet*** **Tekstuuritietoinen debayer-menetelmä \[vain Chloros+] —** Tekstuuritietoinen menetelmä käyttää korkealaatuista, reunat tunnistavaa debayeria yhdistettynä AI/ML-kohinanpoistomalliin, joka poistaa lähes kaiken debayeroinnista aiheutuvan kohinan.
* **Tuki T4P-kalibrointikohteille*** **Nopeampi Chloros+:n GPU-käsittely, parempi muistinhallinta**

**Virhekorjaukset*** Täysin uusi käyttöliittymä (GUI), jonka pitäisi nyt toimia kaikilla Windows-tietokoneilla.

</details>

<details>

<summary>Versio 1.0.4</summary>

**Julkaisupäivä: 5. tammikuuta 2026**

**Uudet ominaisuudet*** **Kuva/metatiedot-vaihtopainike**: Tiedostoselaimeen on lisätty vaihtopainike, jolla valitun kuvan metatiedot voidaan tarkastella taulukkomuodossa kuvaruudukon sijaan
* **Kuvaruudukon zoomausliukusäädin**: Uusi käyttöliittymän liukusäädin pikkukuvien koon säätämiseen (tukee myös CTRL + hiiren rullaa)
* **Kuvaristikon vientipainikkeet**: Ylärivillä olevat painikkeet, joilla pikkukuvien näkymä voidaan vaihtaa JPG-muodosta käsiteltyihin vientimuotoihin (Targets, Reflectance, Index, LUT)
* **Kartta-välilehti**: Uusi interaktiivinen 2D-kartta, joka näyttää kuvien GPS-sijaintimerkit
  * Tukee Google Mapsia ja ESRI-karttatiilejä (valitsee automaattisesti parhaan karttapalvelun zoomausasteen saatavuuden perusteella)
  * Pikkukuvien esikatselu karttamerkkien päällä hiiren osoittimella

**Virhekorjaukset*** Parannettu tuki Chloros-ohjelman asentamiselle muilla kuin englanninkielisillä tietokoneilla

</details>

<details>

<summary>Versio 1.0.3</summary>

**Julkaisupäivä: 20. joulukuuta 2025**

**Uudet ominaisuudet*** Ensimmäinen julkaisu

**Parannukset*** Ensimmäinen julkaisu

**Virhekorjaukset*** Ensimmäinen julkaisu

**Tunnetut ongelmat*** Ensimmäinen julkaisu

</details>***

## Käyttöoikeussopimus**Omistusoikeudellinen ohjelmisto** – Copyright (c) 2026 MAPIR Inc.

Luvaton käyttö, jakelu tai muokkaaminen on kielletty.

**Ilmainen versio**: Saatavilla henkilökohtaiseen ja kaupalliseen käyttöön, ominaisuuksissa on rajoituksia**Chloros+**: Tilauspohjainen lisenssi edistyneille ominaisuuksille ja kaupalliseen käyttöön
