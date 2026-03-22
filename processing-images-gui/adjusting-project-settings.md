# Projektin asetusten määrittäminen

Ennen kuvien käsittelyä on tärkeää määrittää projektin asetukset työnkulun vaatimusten mukaisiksi. Projektin asetukset <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> -paneeli tarjoaa kattavat hallintamahdollisuudet kalibroinnille, käsittelyvaihtoehdoille, monispektrisille indekseille ja vientimuodoille.

## Projektin asetusten avaaminen

1. Avaa projektisi Chloros-ohjelmassa
2. Napsauta **Projektin asetukset** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> -kuvaketta vasemmassa sivupalkissa
3. Projektin asetukset -paneeli näyttää kaikki määritysvaihtoehdot

{% hint style="info" %}
**Asetukset tallennetaan automaattisesti** projektin mukana. Kun avaat projektin uudelleen, kaikki asetukset palautuvat.
{% endhint %}

***

## Yleisten työnkulkujen pikasetukset

### Oletusasetukset (suositellaan useimmille käyttäjille)

Tyypillisissä MAPIR Survey3-kameratyönkuluissa oletusasetukset toimivat hyvin:

* ✅ **Vignettikorjaus**: Käytössä
* ✅ **Heijastavuuskalibrointi**: Käytössä (vaatii kuvia MAPIR-kohteista)
* ✅ **Debayer-menetelmä**: Vakio (Nopea, Keskilaatuinen)
* ✅ **Vientimuoto**: TIFF (16-bittinen)

Tuo kuvasi ja aloita käsittely näillä oletusasetuksilla.

***

## Projektin asetusten yleiskatsaus

Projektin asetukset -paneeli on jaoteltu useisiin luokkiin. Alla on yhteenveto kustakin osiosta. Katso täydelliset ohjeet kohdasta [Projektin asetukset](../project-settings/project-settings.md).

### Kohteen tunnistus

Määrittää, miten Chloros tunnistaa kalibrointikohteet kuvissasi.

**Tärkeimmät asetukset:*** **Vähimmäiskalibrointinäytteen pinta-ala**: Kohteen tunnistuksen kokoraja (oletus: 25 pikseliä)
* **Vähimmäiskohteen klusterointi**: Kohdealueiden ryhmittelyn samankaltaisuusraja (oletus: 60)**Milloin säätää:**

* Suurenna näytteen pinta-alaa, jos saat vääriä tunnistuksia
* Pienennä, jos kohteita ei tunnisteta
* Säädä ryhmittelyä, jos kohteet jakautuvat useiksi tunnistuksiksi

### Käsittely

Tärkeimmät kuvankäsittely- ja kalibrointiasetukset.

**Tärkeimmät asetukset:*** **Vignettikorjaus**: Kompensoi objektiivin reunoilla esiintyvää tummenemista ✅ Suositeltava
* **Heijastavuuskalibrointi**: Normalisoi arvot kalibrointikohteiden avulla ✅ Suositeltava
* **Debayer-menetelmä**: Algoritmi RAW-tiedostojen muuntamiseksi 3-kanavaisiksi monispektrisiksi
* **Vähimmäiskalibrointiväli**: Aika kalibrointikohteiden käytön välillä (0 = käytä kaikkia)**Lisäasetukset:*** **Valosensorin aikavyöhykkeen siirtymä**: PPK-ajan synkronointia varten (oletus: 0)
* **Käytä PPK-korjauksia**: Käyttää .daq-tiedostojen GPS-/valotuspin-tietoja
* **Valotuspin 1/2**: Määrittää kamerat valotuspineille kaksoiskamerajärjestelmissä

### Debayer-menetelmä

Tarjoamme tällä hetkellä 2 debayering-menetelmää Chloros:ssa:

#### Vakio (nopea, keskilaatuinen)

Vakio-debayer-menetelmä käsittelee tiedot nopeasti, mutta siinä näkyy debayering-värikohinaa, mikä johtaa vähemmän tarkkoihin ja kohinaisempiin kuviin.

#### Texture Aware (hidas, korkein laatu) \[Vain Chloros+]

Texture Aware käyttää korkealaatuista reuna-tietoista debayeria yhdistettynä AI/ML-kohinanpoistomalliin, joka poistaa lähes kaiken debayering-kohinan. Texture Aware -mallin käyttö vaatii GPU-muistia (VRAM). Suosittelemme sen käyttöä, kun käytettävissä on yli 4 Gt VRAM-muistia, jotta käsittely on nopeampaa.

### Indeksi (monispektriset indeksit)

Määritä, mitkä kasvillisuusindeksit lasketaan ja viedään.

**Kuinka lisätä indeksejä:**

1. Napsauta**&quot;Lisää indeksi&quot;** -painiketta
2. Valitse indeksi avattavasta valikosta (NDVI, NDRE, GNDVI jne.)
3. Määritä visualisointiasetukset (LUT-värit, arvoalueet)
4. Lisää useita indeksejä tarpeen mukaan

**Suosittuja indeksejä:*** **NDVI**: Kasvillisuuden yleinen terveys (yleisin)
* **NDRE**: Varhainen stressin havaitseminen yhdessä RedEdge:n kanssa
* **GNDVI**: Herkkä klorofyllipitoisuudelle
* **OSAVI**: Toimii hyvin näkyvän maaperän kanssa
* **EVI**: Alueet, joilla lehtipinta-indeksi (LAI) on korkea**Mukautetut kaavat (vain Chloros+):**

* Luo mukautettuja monispektrisiä indeksikaavoja
* Käytä kaistamatematiikkaa kaikilla kuvakanavilla
* Tallenna mukautetut kaavat uudelleen käytettäväksi

Katso kaikki saatavilla olevat indeksit ja kaavat kohdasta [Monispektriset indeksikaavat](../project-settings/multispectral-index-formulas.md).

### Vie

Hallitsee tulostetiedoston muotoa ja laatua.

**Saatavilla olevat muodot:*** **TIFF (16-bittinen)**: Suositellaan GIS- ja tieteellisiin analyyseihin (alue 0–65 535)
* **TIFF (32-bittinen, prosentteina)**: Liukulukuarvoiset heijastusarvot (alue 0,0–1,0)
* **PNG (8-bittinen)**: Häviötön pakkaus visualisointia varten (alue 0–255)
* **JPG (8-bittinen)**: Pienimmät tiedostot, häviöllinen pakkaus (alue 0–255)***

## Asetusten tallentaminen ja lataaminen

### Projektimallin tallentaminen

Luo uudelleen käytettäviä malleja yhdenmukaisten työnkulkujen varmistamiseksi:

1. Määritä kaikki halutut asetukset Projektin asetukset -paneelissa
2. Vieritä alareunassa olevaan **&quot;Tallenna projektimalli&quot;** -osioon
3. Kirjoita kuvaava mallin nimi (esim. &quot;Survey3N\_RGN\_Agriculture&quot;)
4. Napsauta tallennuskuvaketta

**Edut:**

* Käytä samoja asetuksia useissa projekteissa
* Jaa kokoonpanot tiimin jäsenten kanssa
* Säilytä yhdenmukaisuus toistuvissa kyselyissä

### Lataa malli uuteen projektiin

Kun luot uuden projektin:

1. Valitse päävalikosta **&quot;Uusi projekti&quot;**

2. Valitse vaihtoehto**&quot;Lataa mallista&quot;**

3. Valitse tallennettu mallisi
4. Kaikki asetukset otetaan käyttöön automaattisesti

### Työkansio

**&quot;Tallenna projektikansio&quot;** -asetus määrittää, mihin uudet projektit luodaan oletuksena:

* **Oletussijainti**: `C:\Users\[Username]\Chloros Projects`
* **Vaihda sijainti**: Napsauta muokkauskuvaketta ja valitse uusi kansio
* **Milloin muuttaa**:
  * Verkkoasema tiimiyhteistyötä varten
  * Erillinen asema, jossa on enemmän tallennustilaa
  * Järjestetty kansiorakenne vuosi-/asiakaskohtaisesti

***

## PPK (Post-Processed Kinematic) -asetukset

Jos käytät MAPIR DAQ-tallentimia GPS:n kanssa tarkkaa maantieteellistä paikannusta varten:

### Edellytykset

* MAPIR DAQ, jossa on GPS (GNSS) -moduuli
* .daq-lokitiedosto, jossa on valotuspinnitiedot
* Kamera kytkettynä DAQ:n valotuspinnit tallennusistunnon aikana

### Asennusvaiheet

1. Sijoita .daq-lokitiedosto projektikansioosi
2. Ota Projektin asetuksissa käyttöön **&quot;Käytä PPK-korjauksia&quot;** -valintaruutu
3. Aseta tarvittaessa **&quot;Valosensorin aikavyöhykkeen siirtymä&quot;** (oletus: 0 UTC:lle)
4. Määritä kamerat valotuspinneille:
   * **Yksi kamera**: Määritetään automaattisesti pinniin 1
   * **Kaksi kameraa**: Määritä kukin kamera manuaalisesti oikealle pinnille**Valotuspinnien määrittäminen:*** **Valotusnasta 1**: Valitse kameramalli pudotusvalikosta
* **Valotusnasta 2**: Valitse toinen kamera tai &quot;Älä käytä&quot;
* Samaa kameraa ei voi määrittää molemmille nastoille

{% hint style="warning" %}
**Tärkeää**: Valotusnastat on määritettävä oikein vastaaville kameroille. Virheellinen määritys johtaa virheellisiin maantieteellisiin sijaintitietoihin.
{% endhint %}

***

## Edistyneet skenaariot

### Monikameraprojektit

Kun käsitellään kuvia useista MAPIR-kameroista yhdessä projektissa:

1. Chloros tunnistaa automaattisesti kunkin kameramallin
2. Jokainen kamera saa sopivan käsittelyprofiilin
3. PPK: Määritä jokaiselle kameralle manuaalisesti oikea valotuspiste
4. Kaikki kamerat käyttävät samaa vientimuotoa ja indeksejä

**Esimerkki**: Survey3W RGN + Survey3N OCN kahden kameran laitteisto

### Aikavälin tai usean päivämäärän mittaukset

Saman alueen toistuville mittauksille ajan kuluessa:

1. Luo malli vakioasetuksillasi
2. Käytä johdonmukaista kalibrointikohteen asetusta jokaisessa istunnossa
3. Käsittele jokainen päivämäärä erillisenä projektina
4. Käytä identtisiä asetuksia vertailukelpoisten tulosten saamiseksi
5. Vie samassa formaatissa ajallista analyysia varten

### Suuret tietojoukot

Projekteille, joissa on paljon kuvia (yli 500):

* Harkitse jakamista pienempiin projekteihin päivämäärän tai alueen mukaan
* Käytä Chloros+ rinnakkaisprosessointia nopeampien tulosten saamiseksi
* Harkitse CLI- tai API-versiota eräajon automatisointiin
* Säädä vähimmäiskalibrointiväliä kohteen tunnistusaikojen lyhentämiseksi

***

## Asetusten tarkistaminen

Ennen käsittelyn aloittamista tarkista nämä keskeiset asetukset:

* [ ] Kameramalli tunnistettu oikein tiedostoselaimessa
* [ ] Vignettikorjaus käytössä
* [ ] Heijastavuuskalibrointi käytössä
* [ ] Vähintään yksi kalibrointikohdekuva tuotu
* [ ] Halutut monispektriset indeksit lisätty
* [ ] Työnkulkuusi sopiva vientimuoto
* [ ] PPK-asetukset määritetty (jos käytät .daq-tiedostoja valotustapahtumien kanssa)

***

## Seuraavat vaiheet

Kun asetukset on määritetty:

1. **Merkitse kalibrointikohdekuvat** – Katso [Kohdekuvien valinta](choosing-target-images.md)
2. **Aloita käsittely** – Katso [Käsittelyn aloittaminen](starting-the-processing.md)
3. **Seuraa edistymistä** – Katso [Käsittelyn seuranta](monitoring-the-processing.md)

Katso kaikki saatavilla olevat asetukset [Projektin asetukset](../project-settings/project-settings.md) -ohjeista.
