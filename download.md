---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Lataa

Lataa Chloros-ohjelmiston uusin versio, jotta pääset alkuun monispektrikuvien käsittelyssä.

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
| **Internet**         | Vaaditaan \[valinnaisen] Chloros+ -lisenssin aktivointiin | Vaaditaan \[valinnaisen] Chloros+ -lisenssin aktivointiin |

#### Linux amd64 (x86\_64)

| Vaatimukset       | Vähimmäisvaatimukset                    | Suositellut vaatimukset               |
| ----------------- | -------------------------- | ------------------------- |
| **Jakelu**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Prosessori**     | x86\_64 (Intel/AMD)        | Intel Core i7 tai parempi   |
| **Muisti (RAM)**  | 8 Gt                        | 16 Gt tai enemmän              |
| **Näytönohjain** | Ei tarvita (käsitellään prosessorilla)      | NVIDIA-näytönohjain, jossa vähintään 4 Gt VRAM-muistia |
| **Tallennustila**       | 2 GB vapaata tilaa             | SSD, jossa vähintään 10 GB vapaata tilaa       |
| **Python**        | Python 3.7+ (SDK:lle)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Vaatimukset      | Vähimmäisvaatimukset                      | Suositellut vaatimukset                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Alusta**     | NVIDIA Jetson ja JetPack 6 | Jetson Orin NX 16 Gt tai AGX Orin |
| **Muisti (RAM)** | 8 Gt (jaettu GPU/CPU)         | 16 Gt tai enemmän jaettua                    |
| **Tallennustila**      | 2 GB vapaata tilaa               | NVMe SSD, jossa vähintään 10 GB vapaata        |
| **Python**       | Python 3.7+ (SDK:lle)        | Python 3.10+                    |

{% hint style="info" %}
**GPU-kiihdytys**: NVIDIA-grafiikkapiirejä käyttävät Chloros+-käyttäjät voivat hyödyntää CUDA-kiihdytystä huomattavasti nopeampaan käsittelyyn. Tämä toimii NVIDIA-työpöytägrafiikkapiireillä Windows- ja Linux amd64 -alustoilla sekä NVIDIA Jetsonilla, jossa on JetPack 6. Chloros+-käyttäjät saavat myös monisäikeisen käsittelyn maksimaalisen nopeuden saavuttamiseksi.
{% endhint %}

***

## Lataa Chloros


### Uusin vakaa versio: Versio 1.3.0 (11. marraskuuta 2026)

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Lataa Chloros Windowsille (.exe)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Lataa Chloros Linux amd64:lle (.deb)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Lataa Chloros Linux-arm64 / Jetson -versio (.deb)</a>


#### Windows-asennusohjelma (GUI + CLI + taustapalvelu)

* **Tiedostotyyppi**: .exe (Windows-asennusohjelma)

**Asennusohjeet:**

1. Lataa yllä oleva .exe-tiedosto
2. Kaksoisnapsauta asennusohjelmaa aloittaaksesi asennuksen
3. Noudata asennusohjeiden ohjeita
4. Valitse asennuskansio (oletus: `C:\Program Files\Chloros\`)
5. Pidä **Valitse lisätehtävät**-sivulla molemmat**Järjestelmäasetukset** -valinnat valittuina (suositeltavaa — katso alla)
6. Viimeistele asennus ja käynnistä Chloros tai Chloros CLI
7. Kirjaudu sisään [MAPIR Cloud Chloros+ -tililläsi](https://cloud.mapir.camera/pricing) (tai jatka ilmaisversiolla)

**Järjestelmäasetukset-vaihtoehdot** (molemmat valittuna oletuksena):

| Vaihtoehto, asennusohjelman sanamuodossa | Mitä se tekee | Hiljaisen asennuksen poisto |
| --- | --- | --- |
| **Ota käyttöön Windows in pitkät polut** (sallii projektikansioiden sijainnin syvemmällä kuin 260 merkkiä; suositeltava) | Asettaa Windows in koko tietokoneen kattavan `LongPathsEnabled`-asetuksen arvoksi 1. Windows issa tämä asetus on oletusarvoisesti pois päältä, jolloin tiedostopolku ei saa olla pidempi kuin 260 merkkiä; Chloros-tuotteet sijaitsevat noin 130 merkkiä projektikansion alapuolella, joten kun pitkät polut eivät ole käytössä, syvemmälle sijoitettua projektikansiota ei hyväksytä. Asetus astuu voimaan asennuksen jälkeen käynnistetyissä ohjelmissa. Se pysyy päällä myös asennuksen poistamisen jälkeen, koska muut ohjelmat saattavat tarvita sitä. | `/MERGETASKS="!longpaths"` |
| **Chloros-taustapalvelimen poissulkeminen Microsoft Defenderin tarkistuksesta** (paljon nopeampi kameran tallennus; suositeltava) | Lisää Microsoft Defenderin prosessipoissulkemisen asennuskansiossa olevalle `chloros-backend.exe`-prosessille, jolloin reaaliaikainen suojaus ei tarkista jokaista tiedostoa, jonka taustapalvelin kirjoittaa, riippumatta siitä, mihin se ne kirjoittaa. Suurilla tallennusnopeuksilla skannaus vie noin 30 % tallennusnopeudesta (77,7 vs. 52,7 tallennusta sekunnissa 9 kameran jatkuvassa tallennuksessa). Mitään muuta ohjelmaa ei suljeta pois, ja poissulkeminen poistetaan asennuksen poistamisen yhteydessä. Jos Defenderia ei ole tai se on lukittu käytäntöjen vuoksi, tämä vaihe epäonnistuu ilman ilmoitusta ja asennus jatkuu. | `/MERGETASKS="!defenderexclusion"` |

Hiljainen asennus (`/SILENT` tai `/VERYSILENT`) soveltaa molempia vaihtoehtoja, ellet kieltäydy siitä näytetyllä `/MERGETASKS`-arvolla.

{% hint style="success" %}
Asennusohjelma lisää CLI-kansion (`C:\Program Files\Chloros\cli\`) järjestelmääsi `PATH` kerran — uudelleenasennus tai päivitys ei enää lisää päällekkäistä merkintää, ja se poistaa aiempien asennusohjelmien lisäämät päällekkäisyydet. Avaa uusi terminaali asennuksen jälkeen, jotta voit käyttää `chloros-cli`:ää. Asennusohjelma lisää myös kaksi Windows in palomuurisääntöä taustapalvelimelle jokaiseen verkkoprofiiliin: **Chloros-taustapalvelu (UDP-In)**, kameravirtoja, PTP:tä, mDNS:ää ja DAQ-E-datavirtaa varten, sekä**Chloros-taustapalvelu (TCP-In)**, joka on rajoitettu paikalliseen aliverkkoon ja tarkoitettu DAQ-E-laiteohjelmiston päivitykseen, jolloin anturi muodostaa yhteyden takaisin tähän tietokoneeseen kuvan keräämistä varten. Molemmat poistetaan asennuksen poistamisen yhteydessä.
{% endhint %}

#### Linux amd64 (.deb-paketti — CLI + taustapalvelin)

* **Tiedostotyyppi**: .deb (Debian/Ubuntu-paketti)
* **Arkkitehtuuri**: x86\_64 (amd64)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

#### Linux arm64 — NVIDIA Jetson (.deb-paketti — CLI + taustapalvelu)

* **Tiedostotyyppi**: .deb (JetPack 6)
* **Arkkitehtuuri**: aarch64 (arm64)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

Käytä todellisuudessa lataamaasi tiedostonimeä ja säilytä `./`, jotta `apt` asentaa kyseisen tiedoston sen sijaan, että se etsisi pakettia nimen perusteella. Asenna komennolla `apt install`, ei `dpkg -i`: apt asentaa myös Python-SDK:n vaatimukset, joita paketti suosittelee mutta ei vaadi, ja `dpkg -i` ohittaa ne. Versiosta 1.3.0 lähtien taustakomponentti ja CLI-käyttöliittymä asennetaan purkamatonina `/usr/lib/chloros/`:n alle, joten ne käynnistyvät ilman, että niitä tarvitsee ensin purkaa.

Jetson-paketti käyttää JetPack 6:n CUDA-rakennetta PyTorchista, joten käsittely, mukaan lukien Texture Aware -debayer, suoritetaan Jetsonin GPU:lla. Se käyttää CUDA- ja cuDNN-kirjastoja, jotka JetPack 6 asentaa (`sudo apt install nvidia-jetpack` kuvassa, josta ne puuttuvat).

Katso [Linux-asennus](linux/linux-installation.md) saadaksesi yksityiskohtaiset asennusohjeet ja [NVIDIA Jetson -opas](linux/nvidia-jetson-guide.md) saadaksesi Jetson-laitteille tarkoitettuja ohjeita.

#### Python-SDK (kaikki alustat)

Jokainen asennusohjelma sisältää vastaavan `chloros_sdk`-wheel-paketin, joten SDK-versio vastaa aina asennettua GUI:ta, CLI:ta tai taustapalvelinta. Windows issa asennusohjelma asentaa sen automaattisesti järjestelmän Python iin. Linux issa `.deb` asentaa SDK:n itse järjestelmän `python3`:lle (kohtaan `/usr/lib/python3/dist-packages`), joten `import chloros_sdk` toimii heti asennuksen jälkeen ilman pip:iä tai verkkoyhteyttä. Ainoastaan virtualenv, joka ei näe järjestelmän paketteja, tarvitsee mukana toimitetun wheel-tiedoston asennettuna siihen:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Pelkästään pip-pohjaisissa isäntäkoneissa (joissa ei ole asennettuna Chloros-pakettia; tällöin SDK ohjaa taustapalvelinta toisella koneella) SDK on saatavilla myös PyPI:stä:

```bash
pip install chloros-sdk
```

Katso dokumentaatiota kohdasta [API: Python SDK](api-python-sdk.md) ja [SDK-viiteoppaasta](reference/sdk-reference.md).

{% hint style="info" %}
**Linux-käyttäjät**: `.deb`-paketti asentaa komentoriviliittymän (CLI), taustapalvelimen ja Python-SDK:n. Linux ille ei ole graafista käyttöliittymää — kaikki vuorovaikutus tapahtuu komentoriviliittymän tai SDK:n kautta.
{% endhint %}

***

## Lisäresurssit

### Python-SDK


Kehittäjille ja automaatioprosesseille tarkoitettu Chloros-Python-SDK sisältyy jokaiseen asennusohjelmaan: Windows-asennusohjelma asentaa sen Python-ympäristöön, ja Linux-asennusohjelma `.deb` asentaa sen järjestelmän `python3` käyttöön. Koneella, jolla ei ole Chloros-pakettia (joka ohjaa toisen koneen taustapalvelinta), asenna se PyPI:stä:

```bash
pip install chloros-sdk
```

**Dokumentaatio**: [API: Python SDK](api-python-sdk.md)

**Vaatimukset**: ohjattava Chloros-taustapalvelu (Windows-asennusohjelma tai Linux-paketti `.deb` asentaa sen) sekä Chloros+-kirjautumistunnus maksullisella tasolla

***

## Sisältö

### Windows-asennusohjelma

* ✅ **Chloros-käyttöliittymä** – Täysimittainen graafinen käyttöliittymä
* ✅ **Chloros-CLI** – Komentoriviliittymä (vaatii Chloros+-lisenssin)
* ✅ **Chloros-taustapalvelu** – Käsittelymoottori
* ✅ **Python SDK** – asennetaan Python-ympäristöön, jos sellainen on käytettävissä (vaatii Chloros+-lisenssin)
* ✅ **Kameraprofiilit** – valmiiksi määritetyt MAPIR-kameramallit

### Linux-.deb-paketti

* ✅ **Chloros CLI** – komentoriviliittymä (vaatii Chloros+ -lisenssin)
* ✅ **Chloros Backend** – käsittelymoottori
* ✅ **Python SDK** – asennetaan järjestelmään `python3` (vaatii Chloros+ -lisenssin)
* ✅ **Kameraprofiilit** – Esiasetetut MAPIR-kameramallit
* ❌ Ei graafista käyttöliittymää — Linux-versio on pelkkä komentorivipohjainen CLI/SDK-versio

### Python-SDK (PyPI, koneille, joissa ei ole Chloros-pakettia)

* ✅ **Chloros-SDK** – Python-API (vaatii Chloros+-lisenssin)

***

## Päivitä Chloros+:aan

Avaa edistyneitä ominaisuuksia Chloros+-tilauksella:

* 🚀 **Monisäikeinen käsittely** – Käsittele kuvia rinnakkain
* ⚡ **GPU (CUDA)-kiihdytys** - Hyödynnä NVIDIA-GPU:n tehoa
* 💻 **Pääsy CLI:hen** - Automatisoi komentorivityökaluilla
* 🐍 **Python-SDK** - Ohjelmoitava API-käyttö
* 📱 **Useita laitteita** - Käytä 2–10+ laitteella (riippuu tilauksesta)
* **🐻 Edistyksellinen tekstuuritietoinen debayer-menetelmä** – korkealaatuinen, reunoja tunnistava debayer-menetelmä yhdistettynä AI/ML-kohinanpoistomalliin, joka poistaa lähes kaiken debayer-kohinan.
* 🧮 **Mukautetut kaavat** – Luo mukautettuja monispektrisiä indeksejä

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Katso Chloros+ -paketit ja hinnat</a></p>

***

## Asennusohjeet

### Vianmääritys

**Asennus epäonnistuu ja näyttää seuraavan virheilmoituksen:**

* Varmista, että sinulla on järjestelmänvalvojan oikeudet
* Poista virustorjuntaohjelma väliaikaisesti käytöstä
* Tarkista, että järjestelmäsi täyttää vähimmäisvaatimukset

**Sovellus ei käynnisty (Windows):**

* Varmista, että Windows 10/11 (64-bittinen) on asennettu
* Päivitä näytönohjaimen ajurit
* Tarkista virheen yksityiskohdat Windows in tapahtumakatselijasta
* Ota yhteyttä tukeen ja lähetä virhelokit

**CLI ei käynnisty (Linux):**

* Varmista, että `.deb`-paketti on asennettu oikein: `dpkg -l | grep chloros`
* Tarkista käyttöoikeudet: `sudo chmod +x /usr/bin/chloros-cli`
* Suorita vianmääritys: `chloros-cli selftest`
* Tarkista, puuttuuko kirjastoja: `ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"` (versiosta 1.3.0 lähtien `/usr/lib/chloros/chloros-backend` on pieni käynnistysskripti; itse ohjelma sijaitsee tiedostossa `/usr/lib/chloros/backend/`)

**Lisenssin aktivointiongelmat:**

* Varmista, että internetyhteys on aktiivinen
* Tarkista tunnistetiedot osoitteessa [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Varmista, ettei palomuuri estä Chloros ia* Katso yksityiskohtaiset ohjeet kohdasta [Chloros+ Kirjautuminen](chloros+-login.md)

### Tuen saaminen

Tarvitsetko apua asennuksessa tai käyttöönotossa?

* 📧 **Sähköposti**: info@mapir.camera
* 🌐 **Verkkosivusto**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Ohjeet**: [Aloitusopas](./)
* ❓ **UKK**: [Usein kysytyt kysymykset](faq.md)

***

## Ohjelmistopäivitykset

Chloros tarkistaa päivitykset, ilmoittaa uuden version saatavuudesta ja tarjoaa linkin tälle lataussivulle — päivitys suoritetaan ajamalla uusi allekirjoitettu asennusohjelma. Asetuksesi, projektisi ja kirjautumistietosi säilyvät päivitysten jälkeen. Linux issa ja Jetsonissa `chloros-cli update` tarkistaa, onko uudempaa versiota saatavilla, ja tarjoaa mahdollisuuden ladata ja asentaa vastaavan `.deb`-version (tämä komento toimii vain Linux issa).

***

## Muutosloki**Versio 1.3.0 (Uusin, 11. marraskuuta 2026)**— katso täydellinen luettelo kohdasta**Chloros

in 1.3.0 uudet ominaisuudet** [Aloitusopas](./)-sivulla.**Versio 1.2.0**— katso**Chloros in 1.2.0 uutuudet** [Aloitus](./)-sivulta.

<details>

<summary>Versio 1.0.5</summary>

**Julkaisupäivä: 10. helmikuuta 2026**

**Uudet ominaisuudet**

* **Tekstuuritietoinen debayer-menetelmä \[Vain Chloros+] -** Tekstuuritietoinen menetelmä käyttää korkealaatuista, reunoja tunnistavaa debayer-menetelmää yhdistettynä AI/ML-kohinanpoistomalliin, joka poistaa lähes kaiken debayer-kohinan.
* **Tuki T4P-kalibrointikohteille**
* **Nopeampi Chloros+ GPU-käsittely, parempi muistinhallinta**

**Virhekorjaukset**

* Täysin uusi käyttöliittymä (GUI), jonka pitäisi nyt toimia kaikilla Windows-tietokoneilla.

</details>

<details>

<summary>Versio 1.0.4</summary>

**Julkaisupäivä: 5. tammikuuta 2026**

**Uudet ominaisuudet**

* **Kuva/metatiedot-kytkin**: Tiedostoselaimeen on lisätty kytkin, jolla valitun kuvan metatiedot voidaan tarkastella taulukkomuodossa kuvaruudukon sijaan
* **Kuvaruudukon zoomausliukusäädin**: Uusi käyttöliittymän liukusäädin pikkukuvien koon säätämiseen (tukee myös CTRL + hiiren rullaa)
* **Kuvaruudukon vientipainikkeet**: Ylärivillä olevat painikkeet, joilla voi vaihtaa pikkukuvien tiedostomuodon JPG:stä käsiteltyihin vientimuotoihin (Targets, Reflectance, Index, LUT)
* **Kartta-välilehti**: Uusi interaktiivinen 2D-kartta, joka näyttää kuvien GPS-sijaintimerkit
  * Tukee Google Mapsia ja ESRI-karttalaattoja (valitsee automaattisesti parhaan karttapalvelun zoomausasteen saatavuuden perusteella)
  * Pikkukuvien esikatselu karttamerkkien päällä hiiren osoittimen ollessa niiden päällä

**Virhekorjaukset**

* Parannettu tuki Chloros-ohjelman asentamiselle muilla kuin englanninkielisillä tietokoneilla

</details>

<details>

<summary>Versio 1.0.3</summary>

**Julkaisupäivä: 20. joulukuuta 2025**

**Uudet ominaisuudet**

* Ensimmäinen julkaisu

**Parannukset**

* Ensimmäinen julkaisu

**Virhekorjaukset**

* Ensimmäinen julkaisu

**Tunnetut ongelmat**

* Ensimmäinen julkaisu

</details>

***

## Käyttöoikeussopimus**Omistusoikeudellinen ohjelmisto** – Copyright (c) 2026 MAPIR Inc.

Luvaton käyttö, jakelu tai muokkaaminen on kielletty.

**Ilmainen versio**: Saatavilla henkilökohtaiseen ja kaupalliseen käyttöön, ominaisuuksissa on rajoituksia**Chloros+**: Tilauspohjainen lisenssi edistyneille ominaisuuksille ja kaupalliseen käyttöön
