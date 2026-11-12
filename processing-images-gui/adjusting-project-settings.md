# Projektiasetusten määrittäminen

Ennen kuvien käsittelyä on tärkeää määrittää projektiasetukset työnkulun vaatimusten mukaisiksi. Projektiasetukset-paneeli <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> tarjoaa kattavat hallintamahdollisuudet kalibrointiin, käsittelyvaihtoehtoihin, monispektrisiin indekseihin ja vientimuotoihin.

## Projektiasetusten avaaminen

1. Avaa projektiChloros-ohjelmassa
2. Napsauta vasemmassa sivupalkissa olevaa **Projektiasetukset**-kuvaketta <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">
3. Projektiasetukset-paneelissa näkyvät kaikki määritysvaihtoehdot

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>Projektin asetukset -paneeli — Näyttö, kohteen tunnistus ja käsittely</p></figcaption></figure>

{% hint style="info" %}
**Asetukset tallennetaan automaattisesti** projektin mukana. Kun avaat projektin uudelleen, kaikki asetukset palautuvat.
{% endhint %}

***

## Yleisten työnkulkujen pikasetukset

### Oletusasetukset (suositellaan useimmille käyttäjille)

Oletusasetukset toimivat hyvin tyypillisissä Survey3- ja LATTICE-työnkuluissa:

* ✅ **Vignettikorjaus**: Käytössä
* ✅ **Heijastavuuden kalibrointi / valkotasapaino**: Käytössä (käyttää MAPIR-kohteita ja/tai DAQ-valosensorin tietoja)
* ✅ **Debayer-menetelmä**: Vakio (nopea, keskilaatuinen)
* ✅ **Vientimuoto**: TIFF (16-bittinen) (TIFF (8-bittinen) kokonaan 8-bittisessä projektissa)
* ✅ **Kaikki vientituotteet**: Käytössä (LATTICE tallentaa automaattisesti debayeroidut, esikatselu-, säteily- ja heijastavuuskuvat)

Tuo kuvasi ja aloita käsittely näillä oletusasetuksilla.

***

## Projektiasetusten yleiskatsaus

Projektiasetukset-paneeli on jaoteltu alla oleviin osioihin. Kaksi lisäosiota — **DAQ-valosensori**ja**Matriisin kohdistus** — tulevat näkyviin automaattisesti, kun projektisi sisältää kyseisiä tiedostoja. Katso täydellinen dokumentaatio kohdasta [Projektin asetukset](../project-settings/project-settings.md).

### Näyttö

* **Kuvien pikkukuvien resoluutio**: Kuvaruudukon pikkukuvien resoluutio. Vaihtoehdot:**Oletus (512 px)**,**1024 px**,**2048 px**,**Täysi resoluutio**. Vaikuttaa vain näyttöön – ei vaikuta koskaan käsittelyyn. Suuremmat arvot näyttävät terävämmiltä zoomattaessa, mutta latautuvat hitaammin.

### Kohteen tunnistus

Määrittää, miten Chloros tunnistaa kalibrointikohteet kuvissasi.

**Tärkeimmät asetukset:*** **Kalibroinnin vähimmäisnäytteen pinta-ala (px)**: Kohteen tunnistuksen kokoraja (oletus:**25**, alue 0–10000)
* **Kohteiden ryhmittelyn vähimmäisarvo (0–100)**: Kohdealueiden ryhmittelyn samankaltaisuuskynnys (oletus:**60**)**Milloin säätää:**

* Suurenna näytealuetta, jos ohjelma tunnistaa vääriä kohteita
* Pienennä, jos kohteita ei tunnisteta
* Säädä ryhmittelyä, jos kohteet jakautuvat useiksi tunnistuksiksi

{% hint style="info" %}
Nämä asetukset ovat harmaana, kun **heijastavuuskalibrointi / valkotasapaino** on pois päältä — kun se on pois päältä, kohteen tunnistusta ei suoriteta lainkaan.
{% endhint %}

### Käsittely

Tärkeimmät kuvankäsittely- ja kalibrointiasetukset.

**Tärkeimmät asetukset:*** **Vignettikorjaus**: Kompensoi objektiivin reunoilla esiintyvää tummenemista ✅ Suositeltava
* **Heijastavuuskalibrointi / valkotasapaino**: Kalibroi kuvat tunnistettujen kohteiden (Survey3) ja/tai DAQ-valosensorin tietojen (LATTICE) avulla ✅ Suositeltava
* **Debayer-menetelmä**: Algoritmi RAW-tiedostojen muuntamiseksi 3-kanavaisiksi monispektrisiksi
* **Vähimmäiskalibrointiväli**: Kalibrointikohteen käytön välinen vähimmäisaika sekunteina (oletus:**0** = käytä kaikkia, alue 0–3600)**Kalibroimattomat varatuotteet:**Kun kuvakehyksen heijastuskykyä ei voida kalibroida (kohdetta ei ole saatavilla tai kalibrointi on pois käytöstä), se viedään yhtenä kahdesta varatuotteesta —**jokaisella ajolla on täsmälleen yksi näistä kahdesta**, ja valinta tehdään Vignette-korjauskytkimellä:

* **Vie anturin vaste**: tallentaa tiedoston `Sensor_Response_Images` — käytetään, kun vignettikorjaus on**pois päältä*** **Vienti vignettikorjattuna**: tallentaa tiedoston `Vignette_Corrected_Images` — käytetään, kun vignettikorjaus on**päällä**Valintaruutu, jota ei ole käytössä, on harmaana. Aktiivisen valintaruudun valinnan poistaminen estää kyseisen tiedoston tallentamisen kokonaan.**LATTICE-vientituotteet** (näkyvät jokaisessa projektissa; ne koskevat LATTICE-kaappauksia):

* **Vie debayeroitu**: lineaarinen debayeroitu kuva (`Debayered_Images`): kalibrointipaketin tumma offset on poistettu, minkä jälkeen kuva on demosaikoitu ilman tasokenttää tai sekoituksen purkua — sama tiedosto, jonka tallennus tallentaa debayeroidulla tasollaan — anturin alkuperäisessä suunnassa. Koskee RGB- ja monispektrisiä moduuleja.
* **Vie esikatselu**: näytön esikatselu (`Preview_Images`). RGB = valkotasapaino (DAQ-valonlähde, jos saatavilla, muuten harmaa maailma) + gamma; monispektrinen = väärävärinen venytys.
* **Vie säteilyvoimakkuus**: float32-muotoinen spektrinen säteilyvoimakkuus (`Radiance_Images`, W/m²/sr/nm). Vain monispektriset moduulit — ei koske RGB-pääkuvia.
* **Vie heijastavuus**: uint16-heijastavuus (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0), kun `.daq`-alas suuntautuva lukema tai kehyksen sisällä oleva kohde peittää kehyksen. Vain monispektriset moduulit.

Kaikki neljä ovat **oletusarvoisesti päällä**— yksi tuotu LATTICE-raakakehys jaetaan kaikkiin käytössä oleviin ja soveltuviin tuotteisiin yhdellä käsittelykierroksella.**Vie heijastavuus** -valintaruutu on harmaana, kun heijastavuuden kalibrointi on pois päältä. Asetukset, joiden käyttö on mahdotonta niiden yläpuolella olevan kytkimen vuoksi, ovat aina harmaana, ja niissä näkyy työkaluvinkki, jossa mainitaan muutettava kytkin.**Lisäasetukset:*** **Valosensorin aikavyöhykkeen siirtymä**: Tunnit UTC:sta valosensorin ajan sovittamista varten (oletus: 0, alue −12 – +12)
* **Käytä PPK-korjauksia**: Käyttää `.daq`-tiedostojen GPS-/valotuspin-tietoja (oletus: pois päältä)
* **Valotuspin 1/2**: Määrittää kamerat valotuspineille kaksoiskamerajärjestelmissä

{% hint style="info" %}
**LATTICE-syöttötaso on automaattinen.** LATTICE-kaappaukset sisältävät käsittelytasonsa XMP-metatiedoissa, ja käsittely aloitetaan aina raakakuvasta lähtien — graafisessa käyttöliittymässä ei ole mitään määritettävää. (CLI-lippu `--input-level` on tarkoitettu edistyneille käyttäjille metatietojen menettäneiden tallenteiden ohittamiseen; katso [CLI-viite](../reference/cli-reference.md).)
{% endhint %}

### Debayer-menetelmä

Chloros tarjoaa tällä hetkellä kaksi debayer-menetelmää:

#### Standard (nopea, keskitasoinen laatu)

Standard-debayer käsittelee nopeasti, mutta siinä esiintyy debayer-värikohinaa, mikä johtaa vähemmän tarkkoihin ja kohinaisempiin kuviin.

#### Tekstuuritietoinen (hidas, korkein laatu) \[Vain Chloros+]

Tekstuuritietoinen menetelmä käyttää korkealaatuista, reunoja tunnistavaa debayeria yhdistettynä tekoäly-/koneoppimismallin avulla toimivaan kohinanpoistoon, joka poistaa lähes kaiken debayering-kohinan. Mallin suorittaminen vaatii GPU-muistia (VRAM): **vähintään 7 GB VRAM-muistia** käytettäessä se voi käsitellä useita kuvia samanaikaisesti; alle 7 GB:n muistimäärällä se käsittelee yhden kuvan kerrallaan (huomattavasti hitaammin). Katso [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**LATTICE-kuvissa käytetään aina Standard-demosaicia.** LATTICE-kuville ei ole olemassa LATTICE-koulutettua Texture Aware -mallia, joten vaihtoehtoa ei tarjota LATTICE-kuville — saman projektin Survey3-kuvissa sitä voidaan kuitenkin edelleen käyttää.
{% endhint %}

### Indeksi (monispektriset indeksit)

Määritä, mitkä kasvillisuusindeksit lasketaan ja viedään. Käyttöliittymän pudotusvalikosta löytyy **27 valmiiksi määriteltyä indeksikaavaa**.**Indeksien lisääminen:**

1. Napsauta**”Lisää indeksi”**-painiketta
2. Valitse indeksi pudotusvalikosta (NDVI, NDRE, GNDVI jne.)
3. Määritä visualisointiasetukset (LUT-värit, arvoalueet)
4. Lisää useita indeksejä tarpeen mukaan

**Suosittuja indeksejä:*** **NDVI**: Kasvillisuuden yleinen terveydentila (yleisin)
* **NDRE**: RedEdge-tekniikalla tapahtuva varhainen stressin havaitseminen* **GNDVI**: Herkkä klorofyllipitoisuudelle
* **OSAVI**: Toimii hyvin näkyvän maaperän kanssa
* **EVI**: Alueet, joilla lehtipinta-ala-indeksi (LAI) on korkea

**Mukautetut kaavat:**

* Luo mukautettuja monispektrisiä indeksikaavoja käyttämällä kaistamatematiikkaa kaikilla kuvakanavilla
* Tallenna mukautetut kaavat myöhempää käyttöä varten
* Mukautetut kaavat ovat Chloros+-ominaisuus; saatavuus riippuu tilauspaketistasi

Kaikki saatavilla olevat indeksit ja kaavat — mukaan lukien tiedot siitä, mitkä nimet ovat käytettävissä vain graafisessa käyttöliittymässä (GUI) ja mitkä toimivat myös komentoriviliittymässä (CLI) tai sovelluskehityspaketissa (SDK) — löytyvät kohdasta [Monispektriset indeksikaavat](../project-settings/multispectral-index-formulas.md).

### Vienti

Määrittää tulostetiedoston muodon.

**Käytettävissä olevat muodot**(asetus:**Kalibroitu kuvamuoto**, oletus**TIFF (16-bittinen)**):

* **TIFF (16-bittinen)**: Suositellaan GIS-käyttöön ja tieteelliseen analyysiin
* **TIFF (8-bittinen)**: 8-bittinen TIFF
* **TIFF (32-bittinen, prosentteina)**: Liukulukuarvot
* **PNG (8-bittinen)**: Häviötön pakkaus visualisointia varten
* **JPG (8-bittinen)**: Pienimmät tiedostot, häviöllinen pakkaus

Luettelo osoittaa, mitä projektin kuvat voivat sisältää: Chloros ei korota bittisyvyyttä. Kaikki 8-bittiset kuvat tarjoavat vain TIFF- (8-bittinen), PNG- (8-bittinen) ja JPG- (8-bittinen) tiedostomuotoja, ja oletusarvo on TIFF (8-bittinen). Sekaprojektissa jokainen tallennus viedään omalla bittisyvyydellään tai sitä alemmalla. Radiance pysyy 32-bittisenä liukulukuna.

Tulostetut tiedostot tallennetaan projektikansioon, ryhmiteltynä kameran ja tiedostomuodon mukaan: `<project>/<camera>/<format>/<Product>_Images/`. Radiance tallennetaan **aina** float32-muodossa kansioon `tiff32` tästä asetuksesta riippumatta. Viedyt tiedostot säilyttävät lähdetiedoston nimen — kansio tunnistaa tuotteen. LATTICE-monispektriset vientitiedostot voivat myös tallentaa pikselikohtaisia rajoitusmaskeja `clamp_masks/`-kansioon tuotekansioiden viereen. Katso [Käsittelyn viimeistely](finishing-the-processing.md) nähdäksesi koko tulostuspuun.

{% hint style="warning" %}
**Heijastusarvojen lukeminen**: DN-arvo, joka tarkoittaa ρ = 1,0, riippuu lähdekamerasta — LATTICE käyttää arvoa 32768 (merkitty XMP-tiedostoon `Chloros:PixelScale`), kun taas Survey3 käyttää arvoa 65535. Lue tunniste sen sijaan, että olettaisit arvon olevan vakio. Katso [Tulostuskuvamuodot](../output-image-formats.md).
{% endhint %}

### DAQ-valosensori

Tässä osiossa luetellaan kaikki projektisi DAQ-alasvalotiedostot (`.daq` / `.csv`), yksi rivi tiedostoa kohti, ja niissä näkyy anturimalli, tiedostonimi sekä kyseiselle tiedostolle voimassa oleva hajottajan **korkin** korjaus, joka on voimassa kyseisessä tiedostossa.

* **Kannen ohitus (kaikki tiedostot)**: yksi koko projektia koskeva pudotusvalikko.**Auto** (oletus) käyttää kunkin tiedoston tallennettua cap-arvoa — jos mitään ei ole tallennettu, oletuksena käytetään auringonpaistetta, mutta DAQ-U-, DAQ-M- ja DAQ-E-laitteissa ei ole cap-arvoa, eikä Chloros-ohjelmisto näe, mitä on sovitettu, joten käsittely antaa varoituksen oletetusta cap-arvosta: valitse sovitettu cap-arvo. Kannen valinta ohittaa kaikki tiedostot paitsi DAQ-E-S:n tiedostot, joiden diffuusori on osa sen kalibrointia: raakatallenteet korjataan sen avulla, ja tallenteet, joissa on jo kansi, viitataan uudelleen (tallennettu korjaus peruutetaan, valittu kansi otetaan käyttöön).
* Rivit antavat varoituksen, jos tallennettu korkki oli jakelijan oletusarvo eikä käyttäjän vahvistama, ja jos valitulla korkilla ei ole profiilia kyseiselle laitemallille (ohitus hylätään kyseisen tiedoston osalta).

Light Sensors -välilehdessä tehdyt DAQ-tallenteet lisätään avoimeen projektiin automaattisesti, ja tuodut `.daq` / `.csv`-tiedostot näkyvät tässä heti, kun ne on lisätty.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Alemmat projektiasetukset — Hakemisto, vientimuoto, DAQ-valosensoriosio sekä projektimallin ja -kansion hallinta</p></figcaption></figure>### Matriisin kohdistus

Tämä osio näkyy **vain**, kun vähintään yksi projektin kuva sisältää moduulien välisen kohdistusmuunnoksen, jonka LATTICE-matriisit merkitsevät tallennuksen yhteydessä (`Chloros:Alignment*` XMP). Se näyttää, kuinka monessa kuvassa on tunnisteet ja mikä kamera on vertailukamerana, ja sisältää seuraavat säätimet:

* **Käytä matriisin kohdistusta** (oletus: päällä): vääristää jokaisen käsitellyn tuotteen (debayeroitu / esikatselu / säteilyvoimakkuus / heijastavuus / indeksi) matriisin yhteiseen vertailugeometriaan. Pois päältä = vienti alkuperäisessä anturigeometriassa.
* **Rajaa yhteiseen päällekkäisyyteen** (oletus: päällä): rajaa kohdistetut vientitiedostot alueelle, joka on yhteinen kaikille moduuleille, jolloin jokaisella kaistalla on sama peittoalue. Pois päältä säilyttää koko anturin kuvan (musta täyttö lähteen ulkopuolella).
* **Uudelleennäytteenotto**:**Bilineaarinen (tasainen, oletus)**,**Lähin (säilyttää tarkat arvot)**— ei pikselien välistä sekoittumista, tiukkaa radiometristä analyysia varten — tai**Kubinen (terävin)**.

***

## Asetusten tallentaminen ja lataaminen

### Projektimallin tallentaminen

Luo uudelleenkäytettäviä malleja yhdenmukaisten työnkulkujen varmistamiseksi:

1. Määritä kaikki halutut asetukset Projektin asetukset -paneelissa
2. Vieritä alareunassa olevaan **”Tallenna projektimalli”** -osioon
3. Kirjoita kuvaava mallinimi (esim. ”Survey3N\_RGN\_Agriculture”)
4. Napsauta tallennuskuvaketta

**Edut:**

* Käytä samoja asetuksia useissa projekteissa
* Jaa asetukset tiimin jäsenten kanssa
* Varmista yhdenmukaisuus toistuvissa mittauksissa

### Mallin lataaminen uuteen projektiin

Kun luot uuden projektin:

1. Valitse päävalikosta **"Uusi projekti"**

2. Valitse projektimalli valinnaisesta mallivalitsimesta
3. Kaikki mallin asetukset otetaan automaattisesti käyttöön

### Työkansio

**"Työkansio"**-asetus määrittää, mihin uudet projektit luodaan oletuksena:

* **Oletussijainti**: `Documents\MAPIR\Chloros Projects` Windows in Asiakirjat-kansiossa (yleensä `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`; se seuraa Asiakirjat-kansiota, jos OneDrive tai vaeltava profiili siirtää sitä)
* **Vaihda sijainti**: Napsauta muokkauskuvaketta ja valitse uusi kansio
* **Jaettu CLI:n kanssa**: `chloros-cli` käyttää samaa oletusprojektikansioasetusta
* **Pidä nimi lyhyenä Windows issa**: ellei Windows in pitkiä polkuja ole otettu käyttöön, projektikansio, jonka nimi on yli noin 130 merkkiä, hylätään ennen käsittelyä — katso [Käsittelyn aloittaminen](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer)
* **Milloin muuttaa**:
  * Verkkoasema tiimiyhteistyötä varten
  * Erillinen asema, jossa on enemmän tallennustilaa
  * Järjestetty kansiorakenne vuosittain/asiakaskohtaisesti

***

## PPK (Post-Processed Kinematic) -asetukset

Jos käytät GPS-toiminnolla varustettuja MAPIR-DAQ-tallentimia tarkkaan maantieteelliseen paikannukseen:

### Edellytykset

* MAPIR-DAQ, jossa on GPS (GNSS)-moduuli
* .daq-lokitiedosto, jossa on valotusaikapistojen tiedot
* Kamera kytkettynä DAQ:n valotusaikapisteisiin tallennuksen aikana

### Määritysvaiheet

1. Sijoita .daq-lokitiedosto projektikansioosi
2. Ota Projektin asetuksissa käyttöön **"Käytä PPK-korjauksia"** -valintaruutu
3. Määritä tarvittaessa **"Valosensorin aikavyöhykkeen siirtymä"** (oletus: 0, eli UTC)
4. Määritä kamerat valotusliittimille:
   * **Yksi kamera**: Määritetään automaattisesti liittimeen 1
   * **Kaksi kameraa**: Määritä kukin kamera manuaalisesti oikealle liittimelle**Valotusliittimien määrittäminen:*** **Valotusliitin 1**: Valitse kameramalli avattavasta valikosta
* **Valotusliitin 2**: Valitse toinen kamera tai ”Älä käytä”
* Samaa kameraa ei voi määrittää molemmille liittimille

{% hint style="warning" %}
**Tärkeää**: Valotusliittimet on määritettävä oikein vastaaville kameroille. Virheellinen määritys johtaa virheellisiin maantieteellisiin sijaintitietoihin.
{% endhint %}

***

## Edistyneet skenaariot

### Monikameraprojektit

Kun käsitellään useiden MAPIR-kameroiden kuvia yhdessä projektissa:

1. Chloros tunnistaa automaattisesti kunkin kameramallin (sekä Survey3:n että LATTICE:n)
2. Jokaiselle kameralle määritetään sopivat käsittelyprofiilit, ja jokaiselle kameralle luodaan oma tulostuskansiohierarkia
3. PPK: Määritä jokainen Survey3-kamera manuaalisesti oikeaan valotusnastaan
4. Kaikki kamerat käyttävät samaa vientimuotoa ja indeksejä

**Esimerkkejä**: Survey3W RGN + Survey3N OCN-kaksoiskamerajärjestelmä tai LATTICE-järjestelmä, jossa yhdistyvät RGB-pääkamera ja kapeakaistamoduulit

### Aikavälin tai usean päivämäärän mittaukset

Kun samaa aluetta mitataan toistuvasti eri aikoina:

1. Luo malli, jossa on vakioasetuksesi
2. Käytä jokaisessa istunnossa yhdenmukaista kalibrointikohteen asetusta
3. Käsittele jokainen päivämäärä erillisenä projektina
4. Käytä identtisiä asetuksia vertailukelpoisten tulosten saamiseksi
5. Vie samassa muodossa ajallista analyysia varten

### Suuret aineistot

Projekteille, joissa on paljon kuvia (yli 500):

* Harkitse jakamista pienempiin projekteihin päivämäärän tai alueen mukaan
* Käytä Chloros+:n rinnakkaisprosessointia nopeampien tulosten saamiseksi
* Harkitse CLI:n tai API:n käyttöä eräajon automatisointiin
* Säädä uudelleenkalibroinnin vähimmäisväliä kohteen tunnistusaikojen lyhentämiseksi

***

## Asetusten tarkistaminen

Ennen käsittelyn aloittamista tarkista nämä keskeiset asetukset:

* [ ] Kameramalli on tunnistettu oikein tiedostoselaimessa
* [ ] Vignettikorjaus käytössä
* [ ] Heijastuskalibrointi käytössä
* [ ] Survey3: vähintään yksi kalibrointikohdekuva tuotu ja tarkistettu; LATTICE: kohde ja/tai `.daq`-alas suuntautuva tallenne olemassa
* [ ] Halutut monispektriset indeksit on lisätty
* [ ] Työnkulkuusi sopiva vientimuoto on valittu
* [ ] PPK-asetukset on määritetty (jos käytät .daq-tiedostoja, joissa on valotustapahtumia)

***

## Seuraavat vaiheet

Kun asetukset on määritetty:

1. **Merkitse kalibrointikohdekuvat** – Katso [Kohdekuvien valinta](choosing-target-images.md)
2. **Aloita käsittely** – Katso [Käsittelyn aloittaminen](starting-the-processing.md)
3. **Seuraa käsittelyn etenemistä** – Katso [Käsittelyn seuranta](monitoring-the-processing.md)

Tarkemmat tiedot kaikista käytettävissä olevista asetuksista löytyvät [Projektiasetukset](../project-settings/project-settings.md) -ohjeista.
