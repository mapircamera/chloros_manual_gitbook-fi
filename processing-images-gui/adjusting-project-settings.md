# Projektiasetusten määrittäminen

Ennen kuvien käsittelyä on tärkeää määrittää projektiasetukset työnkulun vaatimusten mukaisesti. Projektiasetukset-paneeli (<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">) tarjoaa kattavat hallintamahdollisuudet kalibroinnille, käsittelyvaihtoehdoille, monispektrisille indekseille ja vientimuodoille.

## Projektiasetusten avaaminen

1. Avaa projektisi Chloros-ohjelmassa
2. Napsauta vasemmassa sivupalkissa **Projektiasetukset**-kuvaketta <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">
3. Projektiasetukset-paneeli näyttää kaikki määritysvaihtoehdot

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>Projektin asetukset -paneeli — Näyttö, kohteen tunnistus ja käsittely</p></figcaption></figure>{% hint style="info" %}
**Asetukset tallennetaan automaattisesti** projektisi mukana. Kun avaat projektin uudelleen, kaikki asetukset palautuvat.
{% endhint %}

***

## Yleisten työnkulkujen pikasetukset

### Oletusasetukset (suositellaan useimmille käyttäjille)

Oletusasetukset toimivat hyvin tyypillisissä Survey3- ja LATTICE-työnkuluissa:

* ✅ **Vignettikorjaus**: Käytössä
* ✅ **Heijastavuuskalibrointi / valkotasapaino**: Käytössä (käyttää MAPIR-kohteita ja/tai DAQ-valosensorin tietoja)
* ✅ **Debayer-menetelmä**: Vakio (nopea, keskilaatuinen)
* ✅ **Vientimuoto**: TIFF (16-bittinen)
* ✅ **Kaikki vientituotteet**: Käytössä (LATTICE tallentaa automaattisesti fan-out-tiedostot debayeroituna, esikatseluna, säteilyvoimakkuutena ja heijastavuutena)

Tuo vain kuvasi ja aloita käsittely näillä oletusasetuksilla.

***

## Projektin asetusten yleiskatsaus

Projektin asetukset -paneeli on jaoteltu alla oleviin osioihin. Kaksi lisäosiota — **DAQ-valosensori**ja**matriisin kohdistus** — ilmestyy automaattisesti, kun projektisi sisältää kyseisiä tiedostoja. Katso täydellinen dokumentaatio kohdasta [Projektin asetukset](../project-settings/project-settings.md).

### Näyttö

* **Kuvien pikkukuvien resoluutio**: Kuvaruudukon pikkukuvien resoluutio. Vaihtoehdot:**Oletus (512 px)**,**1024 px**,**2048 px**,**Täysi resoluutio**. Vain näyttöä varten — ei vaikuta koskaan käsittelyyn. Suuremmat arvot näyttävät terävämmiltä zoomattuna, mutta latautuvat hitaammin.

### Kohteen tunnistus

Määrittää, miten Chloros tunnistaa kalibrointikohteet kuvissasi.

**Tärkeimmät asetukset:*** **Kalibroinnin vähimmäisnäytteen pinta-ala (px)**: Kohteen tunnistuksen kokoraja (oletus:**25**, alue 0–10000)
* **Kohteiden ryhmittelyn vähimmäisarvo (0–100)**: Samankaltaisuuden kynnysarvo kohdealueiden ryhmittelyä varten (oletus:**60**)**Milloin säätää:**

* Suurenna näytealuetta, jos saat vääriä tunnistuksia
* Pienennä, jos kohteita ei tunnisteta
* Säädä ryhmittelyä, jos kohteet jakautuvat useiksi tunnistuksiksi

{% hint style="info" %}
Nämä asetukset ovat harmaana, kun **heijastuskalibrointi / valkotasapaino** on pois päältä — kun se on pois päältä, kohteen tunnistusta ei suoriteta lainkaan.
{% endhint %}

### Käsittely

Tärkeimmät kuvankäsittely- ja kalibrointiasetukset.

**Tärkeimmät asetukset:*** **Vignettikorjaus**: Kompensoi objektiivin reunoilla esiintyvää tummenemista ✅ Suositeltava
* **Heijastavuuskalibrointi / valkotasapaino**: Kalibroi kuvat tunnistettujen kohteiden (Survey3) ja/tai DAQ-valosensorin tietojen (LATTICE) avulla ✅ Suositeltava
* **Debayer-menetelmä**: Algoritmi RAW-tiedostojen muuntamiseksi 3-kanavaisiksi monispektraalisiksi kuviksi
* **Vähimmäisrekalibrointiväli**: Kalibrointikohteen käytön välinen vähimmäisaika sekunteina (oletus:**0** = käytä kaikkia, alue 0–3600)**Kalibroimattomat varatuotteet:**Kun kuvan heijastuskykyä ei voida kalibroida (kohdetta ei ole käytettävissä tai kalibrointi on pois käytöstä), se viedään yhtenä kahdesta varatuotteesta —**jokaisesta ajosta on olemassa täsmälleen yksi näistä kahdesta**, ja valinta tehdään Vignette-korjauskytkimellä:

* **Vie anturin vaste**: tallentaa `Sensor_Response_Images` — käytetään, kun vignettikorjaus on**pois päältä*** **Vienti vignettikorjattuna**: tallentaa tiedoston `Vignette_Corrected_Images` — käytetään, kun vignettikorjaus on**päällä**Valintaruutu, jota ei ole käytössä, on harmaana. Aktiivisen valintaruudun valinnan poistaminen estää kyseisen tiedoston tallentamisen kokonaan.**LATTICE-vientituotteet** (näkyvät jokaisessa projektissa; ne koskevat LATTICE-kaappauksia):

* **Vie debayeroitu**: lineaarisesti debayeroitu kuva (`Debayered_Images`). Koskee RGB- ja monispektrimoduuleja.
* **Vie esikatselu**: näytön esikatselu (`Preview_Images`). RGB = valkotasapaino (DAQ-valonlähde, jos saatavilla, muuten harmaamaailma) + gamma; monispektrinen = väärävärinen venytys.
* **Säteilyn vienti**: float32-tyyppinen spektrinen säteily (`Radiance_Images`, W/m²/sr/nm). Vain monispektriset moduulit — ei koske RGB-päämoduuleja.
* ****Heijastavuuden vienti**: uint16-muotoinen heijastavuus (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0), kun `.daq`-alaspäin suuntautuva lukema tai kehyksen sisällä oleva kohde peittää kehyksen. Vain monispektriset moduulit.

Kaikki neljä ovat **oletusarvoisesti päällä**— yksi tuotu LATTICE-raakakehys jaetaan kaikkiin käytössä oleviin ja soveltuviin tuotteisiin yhdellä käsittelykerralla.**Vie heijastavuus** -valintaruutu on harmaana, kun heijastavuuden kalibrointi on pois päältä. Asetukset, joiden käyttö on mahdotonta niiden yläpuolella olevan kytkimen vuoksi, ovat aina harmaana, ja niissä näkyy työkaluvinkki, jossa mainitaan muutettava kytkin.**Lisäasetukset:*** **Valosensorin aikavyöhykkeen siirtymä**: Tunnit UTC:stä valosensorin ajan sovittamista varten (oletus: 0, alue −12–+12)
* **Käytä PPK-korjauksia**: Käyttää `.daq`-tiedostojen GPS-/valotuspistetietoja (oletus: pois päältä)
* **Valotuspiste 1/2**: Määrittää kamerat valotuspisteisiin kaksoiskamerajärjestelmissä

{% hint style="info" %}
**LATTICE-syöttötaso on automaattinen.** LATTICE-kaappaukset sisältävät käsittelytasonsa XMP-metatiedoissa, ja käsittely aloitetaan aina raakakuvasta — graafisessa käyttöliittymässä ei ole mitään määritettävää. (CLI-lippu `--input-level` on tarkoitettu edistyneille käyttäjille metatietojen puuttuessa; katso [CLI-viite](../reference/cli-reference.md).)
{% endhint %}

### Debayer-menetelmä

Tarjoamme tällä hetkellä kaksi debayer-menetelmää Chloros:ssa:

#### Standard (Nopea, keskilaatuinen)

Standard-debayeri käsittelee tiedot nopeasti, mutta siinä esiintyy debayer-värikohinaa, mikä johtaa vähemmän tarkkoihin ja kohinaisempiin kuviin.

#### Tekstuuritietoinen (hidas, korkein laatu) \[Vain Chloros+]

Tekstuuritietoinen menetelmä käyttää korkealaatuista, reunoja tunnistavaa debayeria yhdistettynä tekoäly-/koneoppimiseen perustuvaan kohinanpoistomalliin, joka poistaa lähes kaiken debayering-kohinan. Mallin suorittaminen vaatii GPU-muistia (VRAM): **vähintään 7 GB:n VRAM-muistilla** se voi käsitellä useita kuvia samanaikaisesti; alle 7 GB:n muistilla se käsittelee yhden kuvan kerrallaan (huomattavasti hitaammin). Katso [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**LATTICE-kuvissa käytetään aina Standard-demosaicia.** LATTICE-kuville ei ole LATTICE-koulutettua Texture Aware -mallia, joten vaihtoehtoa ei tarjota LATTICE-kuville — saman projektin Survey3-kuvat voivat kuitenkin käyttää sitä.
{% endhint %}

### Indeksi (monispektriset indeksit)

Määritä, mitkä kasvillisuusindeksit lasketaan ja viedään. Käyttöliittymän pudotusvalikosta löytyy **27 ennalta määriteltyä indeksikaavaa**.**Indeksien lisääminen:**

1. Napsauta**”Lisää indeksi”**-painiketta
2. Valitse indeksi pudotusvalikosta (NDVI, NDRE, GNDVI jne.)
3. Määritä visualisointiasetukset (LUT-värit, arvoalueet)
4. Lisää useita indeksejä tarpeen mukaan

**Suosittuja indeksejä:*** **NDVI**: Kasvillisuuden yleinen kunto (yleisin)
* **NDRE**: Stressin varhainen havaitseminen yhdessä RedEdge:n kanssa
* **GNDVI**: Herkkä klorofyllipitoisuudelle
* **OSAVI**: Toimii hyvin näkyvän maaperän kanssa
* **EVI**: Alueet, joilla lehtipinta-ala-indeksi (LAI) on korkea**Mukautetut kaavat:**

* Luo mukautettuja monispektrisiä indeksikaavoja käyttämällä kaistojen matemaattisia operaatioita kaikilla kuvakanavilla
* Tallenna mukautetut kaavat myöhempää käyttöä varten
* Mukautetut kaavat ovat Chloros+-ominaisuus; saatavuus riippuu tilauspaketistasi

Kaikki saatavilla olevat indeksit ja kaavat – mukaan lukien tiedot siitä, mitkä nimet ovat käytettävissä vain graafisessa käyttöliittymässä ja mitkä toimivat myös CLI/SDK-ohjelmissa – löytyvät kohdasta [Monispektriset indeksikaavat](../project-settings/multispectral-index-formulas.md).

### Vienti

Määrittää tulostetiedoston muodon.

**Käytettävissä olevat muodot**(asetus:**Kalibroitu kuvamuoto**, oletus**TIFF (16-bittinen)**):

* **TIFF (16-bittinen)**: Suositellaan GIS-käyttöön ja tieteelliseen analyysiin
* **TIFF (32-bittinen, prosentteina)**: Liukulukuarvot
* **PNG (8-bittinen)**: Häviötön pakkaus visualisointia varten
* **JPG (8-bittinen)**: Pienimmät tiedostot, häviöllinen pakkaus

Tulostetut tiedostot tallennetaan projektikansioon kameran ja tiedostomuodon mukaan ryhmiteltynä: `<project>/<camera>/<format>/<Product>_Images/`. Radiance tallennetaan **aina** float32-muodossa kansioon `tiff32` tästä asetuksesta riippumatta. Viedyt tiedostot säilyttävät lähdetiedoston nimen — kansio identifioi tuotteen. Katso [Käsittelyn viimeistely](finishing-the-processing.md) nähdäksesi täydellisen tulostuspuun.

{% hint style="warning" %}
**Heijastusarvojen lukeminen**: DN-arvo, joka tarkoittaa ρ = 1,0, riippuu lähdekamerasta — LATTICE käyttää arvoa 32768 (merkitty XMP-tiedostoon `Chloros:PixelScale`), Survey3 käyttää arvoa 65535. Lue tunniste sen sijaan, että olettaisit arvon olevan vakio. Katso [Tulostuskuvien formaatit](../output-image-formats.md).
{% endhint %}

### DAQ-valosensori

Tässä osiossa luetellaan kaikki projektisi DAQ-alasvalotiedostot (`.daq` / `.csv`), yksi rivi tiedostoa kohden, ja niissä näkyy anturimalli, tiedostonimi sekä kyseiselle tiedostolle voimassa oleva diffuusorin **kap**-korjaus.

* **Katon ohitus (kaikki tiedostot)**: yksi koko projektia koskeva pudotusvalikko.**Auto** (oletus) käyttää kunkin tiedoston tallennettua kattoarvoa — jos mitään ei ole tallennettu, oletuksena on auringonpaiste, koska kaikkiin MAPIR-DAQ-tiedostoihin sisältyy auringonpaisteen korjaus. Cap-arvon valinta ohittaa jokaisen tiedoston: raakatallenteet korjataan sen avulla, ja tallenteet, joissa on jo cap-arvo, viitataan uudelleen (tallennettu korjaus peruutetaan, valittu cap-arvo otetaan käyttöön).
* Rivit antavat varoituksen, kun tallennettu yläraja oli keskuksen oletusarvo eikä käyttäjän vahvistama, ja kun valitulla ylärajalla ei ole profiilia kyseiselle laitemallille (ohitus hylätään kyseisen tiedoston osalta).

Light Sensors -välilehdessä tehdyt DAQ-tallenteet lisätään avoimeen projektiin automaattisesti, ja tuodut `.daq` / `.csv`-tiedostot näkyvät täällä heti, kun ne on lisätty.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Alemmat projektiasetukset — Hakemisto, vienti-muoto, DAQ-valosensoriosio sekä projektimallin ja -kansion hallinta</p></figcaption></figure>### Matriisin kohdistus

Tämä osio näkyy **vain**, kun vähintään yhdessä projektin kuvassa on moduulien välinen kohdistusmuunnos, jonka LATTICE-matriisit lisäävät tallennuksen yhteydessä (`Chloros:Alignment*` XMP). Se näyttää, kuinka monessa kuvassa on tunnisteet ja mikä kamera on vertailukohteena, ja siinä on seuraavat säätimet:

* **Käytä matriisin kohdistusta** (oletus: päällä): vääristää jokaisen käsitellyn tuotteen (debayeroitu / esikatselu / säteilyvoimakkuus / heijastavuus / indeksi) matriisin yhteiseen vertailugeometriaan. Pois päältä = vienti alkuperäisessä anturigeometriassa.
* **Rajaa yhteiseen päällekkäisyyteen** (oletus: päällä): rajaa kohdistetut vientitiedostot alueelle, joka on yhteinen kaikille moduuleille, jolloin jokaisella kaistalla on sama peittoalue. Pois päältä säilyttää koko anturin kuvapinnan (mustalla täytetty alue lähteen ulkopuolella).
* **Uudelleennäytteistys**:**Bilineaarinen (tasainen, oletus)**,**Lähin (säilyttää tarkat arvot)**— ei pikselien välistä sekoittumista, tiukkaa radiometristä analyysia varten — tai**Kubinen (terävin)**.***

## Asetusten tallentaminen ja lataaminen

### Projektimallin tallentaminen

Luo uudelleen käytettäviä malleja yhdenmukaisten työnkulkujen varmistamiseksi:

1. Määritä kaikki halutut asetukset Projektin asetukset -paneelissa
2. Vieritä alareunassa olevaan **”Tallenna projektimalli”** -osioon
3. Kirjoita kuvaava mallin nimi (esim. &quot;Survey3N\_RGN\_Agriculture&quot;)
4. Napsauta tallennuskuvaketta

**Edut:**

* Sovella samoja asetuksia useisiin projekteihin
* Jaa asetukset tiimin jäsenten kanssa
* Varmista yhdenmukaisuus toistuvissa kyselyissä

### Lataa malli uuteen projektiin

Kun luot uuden projektin:

1. Valitse päävalikosta **”Uusi projekti”**

2. Valitse projektimalli valinnaisesta mallivalitsimesta
3. Kaikki mallin asetukset otetaan käyttöön automaattisesti

### Työkansio

**&quot;Työkansio&quot;**-asetus määrittää, mihin uudet projektit luodaan oletuksena:

* **Oletussijainti**: `C:\Users\[Username]\Chloros Projects`
* **Sijainnin muuttaminen**: Napsauta muokkauskuvaketta ja valitse uusi kansio
* **Jaettu CLI:n kanssa**: `chloros-cli` käyttää samaa oletusprojektikansiota
* **Milloin muuttaa**:
  * Verkkoasema tiimiyhteistyötä varten
  * Erillinen asema, jossa on enemmän tallennustilaa
  * Järjestetty kansiorakenne vuosittain/asiakaskohtaisesti

***

## PPK (Post-Processed Kinematic) -asetukset

Jos käytät MAPIR-DAQ-tallentimia, joissa on GPS tarkkaa maantieteellistä paikannusta varten:

### Edellytykset

* MAPIR DAQ, jossa on GPS (GNSS)-moduuli
* .daq-lokitiedosto, jossa on valotuspinnimerkinnät
* Kamera kytkettynä DAQ:n valotuspinnien kautta tallennuksen aikana

### Määritysvaiheet

1. Sijoita .daq-lokitiedosto projektikansioosi
2. Valitse Projektin asetuksissa **&quot;Käytä PPK-korjauksia&quot;** -valintaruutu
3. Määritä tarvittaessa **&quot;Valosensorin aikavyöhykkeen siirtymä&quot;** (oletus: 0 UTC:lle)
4. Määritä kamerat valotusliittimille:
   * **Yksi kamera**: Määritetään automaattisesti liittimelle 1
   * **Kaksi kameraa**: Määritä kukin kamera manuaalisesti oikealle liittimelle**Valotusliittimien määrittäminen:*** **Valotusliitin 1**: Valitse kameramalli avattavasta valikosta
* **Valotusliitin 2**: Valitse toinen kamera tai ”Älä käytä”
* Samaa kameraa ei voi määrittää molemmille liittimille

{% hint style="warning" %}
**Tärkeää**: Valotuspinnit on määritettävä oikein kunkin kameran kohdalle. Virheellinen määrittäminen johtaa virheellisiin maantieteellisiin sijaintitietoihin.
{% endhint %}

***

## Edistyneet käyttötapaukset

### Monikameraprojektit

Kun käsitellään kuvia useista MAPIR-kameroista yhdessä projektissa:

1. Chloros tunnistaa automaattisesti kunkin kameramallin (sekä Survey3- että LATTICE-mallit)
2. Jokaiselle kameralle määritetään sopivat käsittelyprofiilit, ja jokaiselle kameralle luodaan oma tulostuskansiohierarkia
3. PPK: Määritä jokaiselle Survey3-kameralle manuaalisesti oikea valotusnasta
4. Kaikki kamerat käyttävät samaa vientimuotoa ja indeksejä

**Esimerkkejä**: Survey3W, RGN + Survey3N, OCN -kaksoiskamerajärjestelmä, tai LATTICE-ryhmittely, jossa yhdistetään RGB-pääkamera kapeakaistaisiin moduuleihin

### Aikavälin tai usean päivämäärän mittaukset

Saman alueen toistuvia mittauksia ajan kuluessa varten:

1. Luo malli vakioasetuksillasi
2. Käytä jokaisessa mittaussessiossa yhdenmukaista kalibrointikohteen asetusta
3. Käsittele jokainen päivämäärä erillisenä projektina
4. Käytä identtisiä asetuksia vertailukelpoisten tulosten saamiseksi
5. Vie tiedot samassa muodossa ajallista analyysia varten

### Suuret tietojoukot

Projekteissa, joissa on paljon kuvia (yli 500):

* Harkitse jakamista pienempiin projekteihin päivämäärän tai alueen mukaan
* Käytä Chloros+ -rinnakkaisprosessointia nopeampien tulosten saamiseksi
* Harkitse CLI- tai API-ohjelmia eräajon automatisointiin
* Säädä uudelleenkalibroinnin vähimmäisväliä kohteen tunnistusaikojen lyhentämiseksi

***

## Asetusten tarkistaminen

Ennen käsittelyn aloittamista tarkista nämä keskeiset asetukset:

* [ ] Kameramalli tunnistettu oikein tiedostoselaimessa
* [ ] Vignettikorjaus käytössä
* [ ] Heijastavuuskalibrointi käytössä
* [ ] Survey3: vähintään yksi kalibrointikohdekuva on tuotu ja tarkistettu; LATTICE: kohde ja/tai `.daq`-alas suuntautuva tallenne on olemassa
* [ ] Halutut monispektriset indeksit on lisätty
* [ ] Työnkulkuusi sopiva vientimuoto on valittu
* [ ] PPK-asetukset on määritetty (jos käytetään .daq-tiedostoja, joissa on valotustapahtumia)

***

## Seuraavat vaiheet

Kun asetukset on määritetty:

1. **Merkitse kalibrointikohdekuvat** – Katso [Kohdekuvien valinta](choosing-target-images.md)
2. **Aloita käsittely** – Katso [Käsittelyn aloittaminen](starting-the-processing.md)
3. **Seuraa käsittelyn etenemistä** – Katso [Käsittelyn seuranta](monitoring-the-processing.md)

Tarkemmat tiedot kaikista käytettävissä olevista asetuksista löytyvät [Projektiasetukset](../project-settings/project-settings.md) -ohjeesta.
