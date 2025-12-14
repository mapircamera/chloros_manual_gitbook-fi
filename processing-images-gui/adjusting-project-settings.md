# Projektin asetusten säätäminen

Ennen kuvien käsittelyä on tärkeää määrittää projektin asetukset vastaamaan työnkulun vaatimuksia. Projektin asetukset <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> -paneeli tarjoaa kattavan hallinnan kalibroinnista, käsittelyvaihtoehdoista, monispektrisistä indekseistä ja vientimuodoista.

## Projektin asetusten avaaminen

1. Avaa projekti Chloros
2. Napsauta **Projektin asetukset** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> -kuvaketta vasemmassa sivupalkissa
3. Projektin asetukset -paneeli näyttää kaikki määritysvaihtoehdot

{% hint style=&quot;info&quot; %}
**Asetukset tallennetaan automaattisesti** projektin mukana. Kun avaat projektin uudelleen, kaikki asetukset palautetaan.
{% endhint %}

***

## Nopea asetusten määrittäminen yleisiä työnkulkuja varten

### Oletusasetukset (suositellaan useimmille käyttäjille)

Tyypillisiin MAPIR Survey3 kameratyönkulkuihin oletusasetukset sopivat hyvin:

* ✅ **Vignettikorjaus**: Käytössä
* ✅ **Heijastavuuskalibrointi**: Käytössä (vaatii kuvia MAPIR kohteista)
* ✅ **Debayer-menetelmä**: Korkea laatu (nopeampi)
* ✅ **Vienti-muoto**: TIFF (16-bittinen)

Tuo kuvasi ja aloita käsittely näillä oletusasetuksilla.

***

## Projektin asetusten yleiskatsaus

Projektin asetukset -paneeli on jaettu useisiin luokkiin. Alla on yhteenveto kustakin osiosta. Täydelliset ohjeet ovat kohdassa [Projektin asetukset](../project-settings/project-settings.md).

### Kohteen tunnistus

Ohjaa sitä, miten Chloros tunnistaa kalibrointikohteet kuvissasi.

**Tärkeimmät asetukset:**

* **Vähimmäiskalibrointinäytealue**: Kohteen tunnistuksen kokokynnys (oletus: 25 pikseliä)
* **Vähimmäiskohteiden klusterointi**: Kohdealueiden ryhmittelyn samankaltaisuuskynnys (oletus: 60)

**Milloin säätää:**

* Lisää näytealuetta, jos saat vääriä tunnistuksia.
* Vähennä, jos kohteita ei tunnisteta.
* Säädä klusterointia, jos kohteet jaetaan useisiin tunnistuksiin.

### Käsittely

Tärkeimmät kuvankäsittely- ja kalibrointivaihtoehdot.

**Tärkeimmät asetukset:**

* **Vignette-korjaus**: Kompensoi objektiivin tummenemista reunoilla ✅ Suositeltava
* **Heijastavuuskalibrointi**: Normalisoi arvot kalibrointikohteiden avulla ✅ Suositeltava
* **Debayer-menetelmä**: Algoritmi RAW-tiedostojen muuntamiseksi 3-kanavaisiksi monispektrisiksi tiedostoiksi
* **Minimi kalibrointiväli**: Aika kalibrointikohteiden käytön välillä (0 = käytä kaikkia)

**Lisäasetukset:**

* **Valosensorin aikavyöhykkeen poikkeama**: PPK-ajan synkronointia varten (oletus: 0)
* **Käytä PPK-korjauksia**: Käyttää GPS-/valotuspin-tietoja .daq-tiedostoista
* **Valotusnasta 1/2**: Määrittää kamerat valotusnastoihin kaksoiskamerajärjestelyissä

### Indeksi (monispektriset indeksit)

Määritä, mitkä kasvillisuusindeksit lasketaan ja viedään.

**Indeksien lisääminen:**

1. Napsauta **&quot;Lisää indeksi&quot;**-painiketta
2. Valitse indeksi avattavasta valikosta (NDVI, NDRE, GNDVI jne.)
3. Määritä visualisointiasetukset (LUT-värit, arvoalueet)
4. Lisää useita indeksejä tarpeen mukaan

**Suosittuja indeksejä:**

* **NDVI**: Kasvillisuuden yleinen terveys (yleisin)
* **NDRE**: Stressin varhainen havaitseminen RedEdge:n avulla
* **GNDVI**: Herkkä klorofyllipitoisuudelle
* **OSAVI**: Toimii hyvin näkyvän maaperän kanssa
* **EVI**: Korkea lehtialaindeksi (LAI) alueet

**Mukautetut kaavat (vain Chloros+):**

* Luo mukautettuja monispektrisiä indeksikaavoja
* Käytä kaistamatematiikkaa kaikilla kuvakanavilla
* Tallenna mukautetut kaavat uudelleenkäyttöä varten

Kaikki saatavilla olevat indeksit ja kaavat ovat kohdassa [Monispektriset indeksikaavat](../project-settings/multispectral-index-formulas.md).

### Vienti

Hallitse tulostustiedoston muotoa ja laatua.

**Saatavilla olevat muodot:**

* **TIFF (16-bittinen)**: Suositellaan GIS- ja tieteelliseen analyysiin (alue 0–65 535)
* **TIFF (32-bittinen, prosentti)**: Liukulukuisten heijastavuusarvojen (alue 0,0–1,0)
* **PNG (8-bittinen)**: Häviötön pakkaus visualisointia varten (alue 0–255)
* **JPG (8-bittinen)**: Pienimmät tiedostot, häviöllinen pakkaus (alue 0–255)

***

## Asetusten tallentaminen ja lataaminen

### Projektimallin tallentaminen

Luo uudelleenkäytettäviä malleja yhdenmukaisten työnkulkujen varmistamiseksi:

1. Määritä kaikki halutut asetukset Projektin asetukset -paneelissa.
2. Vieritä alareunassa olevaan **&quot;Tallenna projektimalli&quot;** -osioon.
3. Anna mallille kuvaava nimi (esim. &quot;Survey3N\_RGN\_Agriculture&quot;).
4. Napsauta tallennuspainiketta.

**Edut:**

* Käytä samoja asetuksia useissa projekteissa.
* Jaa asetukset tiimin jäsenten kanssa.
* Säilytä johdonmukaisuus toistuvissa tutkimuksissa.

### Mallin lataaminen uuteen projektiin

Kun luot uuden projektin:

1. Valitse päävalikosta **&quot;Uusi projekti&quot;**.
2. Valitse vaihtoehto **&quot;Lataa mallista&quot;**.
3. Valitse tallentamasi malli.
4. Kaikki asetukset otetaan käyttöön automaattisesti.

### Työkansio

**&quot;Tallenna projektikansio&quot;** -asetus määrittää, mihin uudet projektit luodaan oletuksena:

* **Oletussijainti**: `C:\Users\[Username]\Chloros Projects`
* **Muuta sijaintia**: Napsauta muokkauskuvaketta ja valitse uusi kansio
* **Milloin muuttaa**:
  * Verkkolevy tiimiyhteistyötä varten
  * Erilainen levy, jossa on enemmän tallennustilaa
  * Järjestetty kansiorakenne vuoden/asiakkaan mukaan

***

## PPK (jälkikäsitelty kinemaattinen) -asetukset

Jos käytät MAPIR DAQ -tallentimia GPS:n kanssa tarkkaan maantieteelliseen paikannukseen:

### Edellytykset

* MAPIR DAQ GPS (GNSS) -moduulilla
* .daq-lokitiedosto, jossa on valotustulppamerkinnät
* Kamera kytketty DAQ-altistustappiin kuvausistunnon aikana

### Asetusten vaiheet

1. Siirrä .daq-lokitiedosto projektikansioon.
2. Valitse Projektin asetukset -kohdassa **&quot;Käytä PPK-korjauksia&quot;** -valintaruutu.
3. Aseta tarvittaessa **&quot;Valosensorin aikavyöhykkeen poikkeama&quot;** (oletus: 0 UTC:lle).
4. Määritä kamerat altistustappeihin:
   * **Yksi kamera**: Määritetään automaattisesti nastalle 1
   * **Kaksi kameraa**: Määritä kukin kamera manuaalisesti oikealle nastalle

**Valotusnastojen määrittäminen:**

* **Valotusnasta 1**: Valitse kameramalli avattavasta valikosta
* **Valotusnasta 2**: Valitse toinen kamera tai &quot;Älä käytä&quot;
* Samaa kameraa ei voi määrittää molemmille nastoille

{% hint style=&quot;warning&quot; %}
**Tärkeää**: Valotusnastat on määritettävä oikein vastaaville kameroille. Virheellinen määrittäminen johtaa virheellisiin maantieteellisiin sijaintitietoihin.
{% endhint %}

***

## Edistyneet skenaariot

### Monikameraprojektit

Kun käsitellään useiden MAPIR-kameroiden kuvia yhdessä projektissa:

1. Chloros tunnistaa automaattisesti kunkin kameramallin
2. Jokaiselle kameralle annetaan sopiva käsittelyprofiili
3. PPK: Määritä manuaalisesti jokaiselle kameralle oikea valotusnasta
4. Kaikki kamerat käyttävät samaa vientimuotoa ja indeksejä

**Esimerkki**: Survey3W RGN + Survey3N OCN kaksoiskamerajärjestelmä

### Aikaväli- tai monipäiväiset tutkimukset

Saman alueen toistuville tutkimuksille ajan mittaan:

1. Luo malli vakioasetuksillasi
2. Käytä jokaisessa istunnossa yhdenmukaista kalibrointikohteen asetusta
3. Käsittele jokainen päivämäärä erillisenä projektina
4. Käytä identtisiä asetuksia vertailukelpoisten tulosten saamiseksi
5. Vie samassa muodossa ajallista analyysia varten

### Suuret tietojoukot

Projekteille, joissa on paljon kuvia (500+):

* Harkitse jakamista pienempiin projekteihin päivämäärän tai alueen mukaan.
* Käytä Chloros+ rinnakkaisprosessointia nopeampien tulosten saamiseksi.
* Harkitse CLI tai API eräautomaatiota varten.
* Säädä vähimmäiskalibrointiväliä kohteen havaitsemisajan lyhentämiseksi.

***

## Asetusten tarkistaminen

Ennen käsittelyn aloittamista tarkista nämä tärkeät asetukset:

* [ ] Kameramalli on tunnistettu oikein tiedostoselaimessa
* [ ] Vignettikorjaus on käytössä
* [ ] Heijastavuuskalibrointi on käytössä
* [ ] Vähintään yksi kalibrointikohdekuva on tuotu
* [ ] Halutut monispektriset indeksit on lisätty
* [ ] Vienti-muoto sopii työnkulullesi
* [ ] PPK-asetukset on määritetty (jos käytät .daq-tiedostoa valotustapahtumien kanssa)

***

## Seuraavat vaiheet

Kun asetukset on määritetty:

1. **Merkitse kalibrointikohdekuvat** - Katso [Kohdekuvien valinta](choosing-target-images.md)
2. **Aloita käsittely** - Katso [Käsittelyn aloittaminen](starting-the-processing.md)
3. **Seuraa edistymistä** – Katso [Käsittelyn seuraaminen](monitoring-the-processing.md)

Täydelliset tiedot kaikista käytettävissä olevista asetuksista ovat [Projektin asetukset](../project-settings/project-settings.md) -viitedokumentaatiossa.
