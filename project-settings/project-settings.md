# Projektin asetukset

Projektin asetukset <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> -sivupalkissa voit määrittää kaikki kuvankäsittelyn, kalibrointikohteen tunnistuksen, monispektrisen indeksin laskennan ja projektin vientiasetukset. Nämä asetukset tallennetaan projektin mukana ja ne voidaan tallentaa malleiksi, joita voidaan käyttää uudelleen useissa projekteissa.

## Projektin asetusten avaaminen

Projektin asetukset avataan seuraavasti:

1. Avaa projekti Chloros-ohjelmassa.
2. Napsauta **Projektin asetukset**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> -välilehteä vasemmassa sivupalkissa
3. Asetuspaneeli näyttää kaikki käytettävissä olevat konfigurointivaihtoehdot luokittain järjestettyinä

***

## Kohteen tunnistus

Nämä asetukset ohjaavat, miten Chloros tunnistaa ja käsittelee kalibrointikohteita kuvissasi.

### Kalibroinnin vähimmäisnäytealue (px)

* **Tyyppi**: Luku
* **Alue**: 0–10 000 pikseliä
* **Oletus**: 25 pikseliä
* **Kuvaus**: Asettaa vähimmäisalueen (pikseleinä), joka tarvitaan, jotta havaittu alue voidaan katsoa kelvolliseksi kalibrointikohteen näytteeksi. Pienemmät arvot havaitsevat pienempiä kohteita, mutta voivat lisätä vääriä positiivisia tuloksia. Suuremmat arvot vaativat suurempia, selkeämpiä kohdealueita havaitsemista varten.
* **Milloin säätää**:
  * Lisää arvoa, jos saat vääriä tunnistuksia pienistä kuvan artefakteista.
  * Vähennä arvoa, jos kalibrointikohteet näyttävät pieniltä kuvissasi eivätkä tule tunnistetuiksi.

### Kohteiden vähimmäisklusterointi (0–100)

* **Tyyppi**: Luku
* **Alue**: 0–100
* **Oletus**: 60
* **Kuvaus**: Säätää klusterointikynnystä samanlaisten värialueiden ryhmittämiseksi kalibrointikohteita havaitaessa. Suuremmat arvot edellyttävät, että samanlaiset värit ryhmitellään yhteen, mikä johtaa konservatiivisempaan kohteen havaitsemiseen. Pienemmät arvot sallivat enemmän värieroja kohderyhmän sisällä.
* **Milloin säätää**:
  * Lisää, jos kalibrointikohteet jaetaan useisiin havaitsemisiin.
  * Vähennä, jos värieroja sisältäviä kalibrointikohteita ei havaita kokonaan.

***

## Käsittely

Nämä asetukset säätelevät, miten Chloros käsittelee ja kalibroi kuvasi.

### Vignettikorjaus

* **Tyyppi**: Valintaruutu
* **Oletus**: Käytössä (valittuna)
* **Kuvaus**: Soveltaa vignettikorjausta kompensoimaan objektiivin tummenemista kuvien reunoilla. Vignettikorrektio on yleinen optinen ilmiö, jossa kuvan kulmat ja reunat näyttävät tummemmilta kuin keskusta objektiivin ominaisuuksien vuoksi.
* **Milloin poistaa käytöstä**: Poista käytöstä vain, jos kamerasi/objektiivisi yhdistelmä on jo soveltanut vignettikorrektiota tai jos haluat korjata vignettikorrektion manuaalisesti jälkikäsittelyssä.

### Heijastavuuskalibrointi / valkotasapaino

* **Tyyppi**: Valintaruutu
* **Oletus**: Käytössä (valittuna)
* **Kuvaus**: Ottaa käyttöön automaattisen heijastavuuden kalibroinnin käyttämällä kuvissasi havaittuja kalibrointikohteita. Tämä normalisoi heijastavuusarvot koko tietojoukossa ja varmistaa yhdenmukaiset mittaukset valaistusolosuhteista riippumatta.
* **Milloin poistaa käytöstä**: Poista käytöstä vain, jos haluat käsitellä raakoja, kalibroimattomia kuvia tai jos käytät eri kalibrointityönkulkua.

### Debayer-menetelmä

* **Tyyppi**: Pudotusvalikko
* **Vaihtoehdot**:
  * Korkea laatu (nopeampi) – Tällä hetkellä ainoa käytettävissä oleva vaihtoehto
* **Oletus**: Korkea laatu (nopeampi)
* **Kuvaus**: Valitsee demosaicing-algoritmin, jota käytetään raakojen Bayer-kuvioisten anturitietojen muuntamiseen täysvärikuviksi. Menetelmä ”Korkea laatu (nopeampi)” tarjoaa optimaalisen tasapainon käsittelynopeuden ja kuvanlaadun välillä.
* **Huomautus**: Lisää debayer-menetelmiä saatetaan lisätä Chloros:n tulevissa versioissa.

### Vähimmäiskalibrointiväli

* **Tyyppi**: Luku
* **Alue**: 0–3 600 sekuntia
* **Oletus**: 0 sekuntia
* **Kuvaus**: Asettaa vähimmäisajan (sekunteina) kalibrointikohteiden käytön välillä. Kun asetukseksi valitaan 0, Chloros käyttää kaikkia havaittuja kalibrointikohteita. Kun asetukseksi valitaan suurempi arvo, Chloros käyttää vain kalibrointikohteita, joiden välinen aika on vähintään tämä sekuntimäärä, mikä lyhentää käsittelyaikaa datajoukoissa, joissa kalibrointikohteita otetaan usein.
* **Milloin säätää**:
  * Aseta arvoksi 0, jos haluat maksimaalisen kalibrointitarkkuuden vaihtelevissa valaistusolosuhteissa.
  * Lisää arvoa (esim. 60–300 sekuntiin), jos haluat nopeuttaa käsittelyä, kun valaistusolosuhteet ovat tasaiset ja kalibrointikohteiden kuvia on paljon.

### Valosensorin aikavyöhykkeen poikkeama

* **Tyyppi**: Luku
* **Alue**: -12–+12 tuntia
* **Oletus**: 0 tuntia
* **Kuvaus**: Määrittää valosensorin datan aikaleimojen aikavyöhykkeen poikkeaman (tunteina UTC:stä). Tätä käytetään PPK-datatiedostojen (Post-Processed Kinematic) käsittelyssä, jotta kuvien ottamisen ja GPS-datan välinen aikasynkronointi on oikea.
* **Milloin säätää**: Aseta tämä paikallisen aikavyöhykkeen poikkeamaksi, jos PPK-tiedot käyttävät paikallista aikaa UTC:n sijaan. Esimerkiksi:
  * Tyynenmeren aika: -8 tai -7 (riippuen kesäajasta)
  * Itäinen aika: -5 tai -4 (riippuen kesäajasta)
  * Keski-Euroopan aika: +1 tai +2 (riippuen kesäajasta)

### PPK-korjausten soveltaminen

* **Tyyppi**: Valintaruutu
* **Oletus**: Pois käytöstä (valinta poistettu)
* **Kuvaus**: Mahdollistaa jälkikäsiteltyjen kinemaattisten (PPK) korjausten käytön MAPIR DAQ -tallentimista, jotka sisältävät GPS:n (GNSS). Kun tämä on käytössä, Chloros käyttää kaikkia .daq-lokitiedostoja, jotka sisältävät valotustietoja projektikansiossasi, ja soveltaa tarkkoja maantieteellisiä korjauksia kuviisi.
* **Vaatimus**: .daq-lokitiedosto, joka sisältää valotustietoja, on oltava projektikansiossasi.
* **Käyttöönotto**: On suositeltavaa ottaa PPK-korjaus aina käyttöön, jos .daq-lokitiedostossasi on valotustietoja.

### Valotusnasta 1

* **Tyyppi**: Pudotusvalikko
* **Näkyvyys**: Näkyvissä vain, kun &quot;Käytä PPK-korjauksia&quot; on käytössä JA valotustiedot ovat käytettävissä nastalle 1.
* **Vaihtoehdot**:
  * Projektissa havaitut kameramallinimet.
  * &quot;Älä käytä&quot; – Ohita tämä valotusnasta.
* **Oletus**: Valitaan automaattisesti projektin kokoonpanon perusteella.
* **Kuvaus**: Määrittää tietyn kameran valotustapin 1:lle PPK-ajan synkronointia varten. Valotustappi tallentaa tarkasti kameran sulkimen laukaisun ajankohdan, mikä on tärkeää tarkan PPK-paikannuksen kannalta.
* **Automaattisen valinnan toiminta**:
  * Yksi kamera + yksi tappi: Valitsee kameran automaattisesti.
  * Yksi kamera + kaksi tappia: Tappi 1 määritetään automaattisesti kameralle.
  * Useita kameroita: Manuaalinen valinta vaaditaan

### Valotusnasta 2

* **Tyyppi**: Pudotusvalikko
* **Näkyvyys**: Näkyvissä vain, kun &quot;Käytä PPK-korjauksia&quot; on käytössä JA valotustiedot ovat käytettävissä nastalle 2
* **Vaihtoehdot**:
  * Projektissa havaitut kameramallinimet
  * &quot;Älä käytä&quot; - Ohita tämä valotusnasta
* **Oletus**: Valitaan automaattisesti projektin kokoonpanon perusteella
* **Kuvaus**: Määrittää tietyn kameran valotustappiin 2 PPK-ajan synkronointia varten, kun käytetään kahden kameran kokoonpanoa.
* **Automaattisen valinnan toiminta**:
  * Yksi kamera + yksi tappi: Tappi 2 asetetaan automaattisesti asetukseksi &quot;Älä käytä&quot;
  * Yksi kamera + kaksi nastaa: Nasta 2 asetetaan automaattisesti &quot;Älä käytä&quot; -tilaan
  * Useita kameroita: Manuaalinen valinta vaaditaan
* **Huomautus**: Samaa kameraa ei voi määrittää samanaikaisesti nastalle 1 ja nastalle 2.

***

## Indeksi

Näiden asetusten avulla voit määrittää monispektriset indeksit analysointia ja visualisointia varten.

### Indeksin lisääminen

* **Tyyppi**: Erityinen indeksien konfigurointipaneeli
* **Kuvaus**: Avaa interaktiivisen paneelin, jossa voit valita ja konfiguroida monispektriset kasvillisuusindeksit (NDVI, NDRE, EVI jne.) laskemista varten kuvankäsittelyn aikana. Voit lisätä useita indeksejä, joista jokaisella on omat visualisointiasetuksensa.
* **Käytettävissä olevat indeksit**: Järjestelmä sisältää yli 30 ennalta määriteltyä monispektrista indeksiä, mukaan lukien:
  * NDVI (normalisoitu kasvillisuusindeksi)
  * NDRE (normalisoitu ero RedEdge)
  * EVI (parannettu kasvillisuusindeksi)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * Ja monia muita (katso [Monispektrinen indeksi kaavat](multispectral-index-formulas.md) täydellinen luettelo)
* **Ominaisuudet**:
  * Valitse valmiista indeksi kaavoista
  * Määritä visualisointivärien gradientit (LUT - Look-Up Tables)
  * Aseta kynnysarvot analyysille
  * Luo mukautettuja indeksi kaavoja

### Mukautetut kaavat (Chloros+ -ominaisuus)

* **Tyyppi**: Mukautettujen kaavojen määritelmien joukko
* **Kuvaus**: Voit luoda ja tallentaa mukautettuja monispektrisiä indeksikaavoja käyttämällä kaistamatematiikkaa. Mukautetut kaavat tallennetaan projektin asetusten mukana, ja niitä voidaan käyttää samalla tavalla kuin sisäänrakennettuja indeksejä.
* **Luominen**:
  1. Etsi indeksin konfigurointipaneelista mukautetun kaavan vaihtoehto.
  2. Määritä kaava käyttämällä kaistatunnisteita (esim. NIR, Red, Green, Blue).
  3. Tallenna kaava kuvaavalla nimellä.
* **Kaavan syntaksi**: Tuetaan tavallisia matemaattisia operaatioita, mukaan lukien:
  * Aritmeettiset: `+`, `-`, `*`, `/`
  * Suluissa laskujen järjestys
  * Kaistaviittaukset: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Vienti

Nämä asetukset ohjaavat vietyjen käsiteltyjen kuvien muotoa ja laatua.

### Kalibroitu kuvamuoto

* **Tyyppi**: Pudotusvalikko
* **Vaihtoehdot**:
  * **TIFF (16-bittinen)** - Pakkaamaton 16-bittinen TIFF-muoto
  * **TIFF (32-bittinen, prosentti)** – 32-bittinen liukulukuinen TIFF, jossa heijastavuusarvot on ilmaistu prosentteina
  * **PNG (8-bittinen)** - Pakattu 8-bittinen PNG-muoto
  * **JPG (8-bittinen)** - Pakattu 8-bittinen JPEG-muoto
* **Oletus**: TIFF (16-bittinen)
* **Kuvaus**: Valitsee tiedostomuodon käsiteltyjen ja kalibroitujen kuvien tallentamista varten.
* **Muotoehdotukset**:
  * **TIFF (16-bittinen)**: Suositellaan tieteelliseen analyysiin ja ammattimaisiin työnkulkuihin. Säilyttää maksimaalisen datan laadun ilman pakkausartefakteja. Paras monispektrinen analyysi ja jatkokäsittely GIS-ohjelmistossa.
  * **TIFF (32-bittinen, prosentti)**: Paras työnkulkuille, jotka vaativat heijastavuusarvoja prosentteina (0–100 %). Tarjoaa maksimaalisen tarkkuuden radiometrisille mittauksille.
  * **PNG (8-bittinen)**: Sopii hyvin verkkokatseluun ja yleiseen visualisointiin. Pienemmät tiedostokoot häviöttömällä pakkauksella, mutta pienempi dynaaminen alue.
  * **JPG (8-bittinen)**: Pienimmät tiedostokoot, sopii parhaiten esikatseluun ja vain verkkokäyttöön. Käyttää häviöllistä pakkausta, joka ei sovellu tieteelliseen analyysiin.

***

## Tallenna projektimalli

Tämän ominaisuuden avulla voit tallentaa nykyiset projektiasetuksesi uudelleen käytettävänä mallina.

* **Tyyppi**: Tekstin syöttö + Tallenna-painike
* **Kuvaus**: Anna asetusmallillesi kuvaava nimi ja napsauta tallennuspainiketta. Malli tallentaa kaikki nykyiset projektiasetuksesi (kohteen tunnistus, käsittelyvaihtoehdot, indeksit ja vientimuoto) helpottamaan niiden uudelleenkäyttöä tulevissa projekteissa.
* **Käyttötapaukset**:
  * Luo malleja eri kamerajärjestelmille (RGB, monispektrinen, NIR)
  * Tallenna vakiokokoonpanot tietyille viljelytyypeille tai analyysityönkulkuille
  * Jaa yhdenmukaiset asetukset koko tiimille
* **Käyttöohjeet**:
  1. Määritä kaikki haluamasi projektiasetukset
  2. Anna mallille nimi (esim. &quot;RedEdge Survey3 NDVI Standard&quot;)
  3. Napsauta tallennuspainiketta
  4. Malli voidaan nyt ladata uusia projekteja luotaessa

***

## Tallenna projektikansio

Tämä asetus määrittää, mihin uudet projektit tallennetaan oletuksena.

* **Tyyppi**: Hakemistopolun näyttö + Muokkaa-painike
* **Oletus**: `C:\Users\[Username]\Chloros Projects`
* **Kuvaus**: Näyttää nykyisen oletushakemiston, johon uudet Chloros-projektit luodaan. Napsauta muokkaa-kuvaketta valitaksesi toisen hakemiston.
* **Milloin muuttaa**:
  * Aseta verkkolevyksi tiimiyhteistyötä varten.
  * Vaihda levyksi, jossa on enemmän tallennustilaa suurille tietojoukoille.
  * Järjestä projektit eri kansioihin vuoden, asiakkaan tai projektityypin mukaan.
* **Huomautus**: Tämän asetuksen muuttaminen vaikuttaa vain UUSIIN projekteihin. Olemassa olevat projektit pysyvät alkuperäisissä sijainneissaan.

***

## Asetusten pysyvyys

Kaikki projektin asetukset tallennetaan automaattisesti projektitiedostoon (`.mapir`-projektimuoto). Kun avaat projektin uudelleen, kaikki asetukset palautetaan täsmälleen sellaisina kuin ne olivat.

### Asetusten hierarkia

Asetukset sovelletaan seuraavassa järjestyksessä:

1. **Järjestelmän oletusasetukset** – Chloros:n määrittämät sisäänrakennetut oletusasetukset
2. **Malliasetukset** – Jos lataat mallin projektia luodessasi
3. **Tallennetut projektiasetukset** – Projektitiedoston mukana tallennetut asetukset
4. **Manuaaliset säädöt** – Kaikki muutokset, jotka teet nykyisen istunnon aikana

### Asetukset ja kuvankäsittely

Useimmat asetusten muutokset (erityisesti Käsittely- ja Vienti-kategorioissa) käynnistävät kuvien uudelleenkäsittelyn, jotta uudet asetukset tulevat voimaan. Jotkin asetukset ovat kuitenkin &quot;vain vientiä varten&quot; eivätkä vaadi välitöntä uudelleenprosessointia:

* Tallenna projektimalli
* Työkansio
* Kalibroitu kuvamuoto (koskee vientiä)

***

## Parhaat käytännöt

1. **Aloita oletusasetuksista**: Oletusasetukset toimivat hyvin useimmissa MAPIR-kamerajärjestelmissä ja tyypillisissä työnkuluissa.
2. **Luo mallit**: Kun olet optimoinut asetukset tietylle työnkululle tai kameralle, tallenna ne malliksi, jotta varmistat yhdenmukaisuuden eri projekteissa.
3. **Testaa ennen täyttä käsittelyä**: Kun kokeilet uusia asetuksia, testaa ne pienellä osajoukolla kuvia ennen koko tietojoukon käsittelyä.
4. **Dokumentoi asetuksesi**: Käytä kuvaavia mallinimiä, jotka ilmaisevat kamerajärjestelmän, käsittelytyypin ja käyttötarkoituksen (esim. &quot;Survey3\_RGB\_NDVI\_Agriculture&quot;).
5. **Vienti-muodon valinta**: Valitse vientimuoto lopullisen käyttötarkoituksen perusteella:
   * Tieteellinen analyysi → TIFF (16-bittinen tai 32-bittinen)
   * GIS-käsittely → TIFF (16-bittinen)
   * Nopea visualisointi → PNG (8-bittinen)
   * Verkkopohjainen jakaminen → JPG (8-bittinen)

***

Lisätietoja Chloros:n monispektrisistä indekseistä on sivulla [Monispektriset indeksikaavat](multispectral-index-formulas.md).
