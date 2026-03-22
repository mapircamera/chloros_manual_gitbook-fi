# Projektin asetukset

Projektin asetukset <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> -sivupalkissa voit määrittää projektisi kuvankäsittelyn, kalibrointikohteen tunnistuksen, monispektrisen indeksin laskennan ja vientiasetusten kaikki ominaisuudet. Nämä asetukset tallennetaan projektisi mukana, ja ne voidaan tallentaa malleiksi, joita voi käyttää uudelleen useissa projekteissa.

## Projektin asetusten avaaminen

Voit avata projektin asetukset seuraavasti:

1. Avaa projekti Chloros-ohjelmassa
2. Napsauta **Projektin asetukset**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> -välilehteä vasemmassa sivupalkissa
3. Asetuspaneelissa näkyvät kaikki käytettävissä olevat konfigurointivaihtoehdot luokittain järjestettyinä

***

## Kohteen tunnistus

Nämä asetukset määrittävät, miten Chloros tunnistaa ja käsittelee kalibrointikohteita kuvissasi.

### Kalibroinnin vähimmäisnäytteen pinta-ala (px)

* **Tyyppi**: Luku
* **Alue**: 0–10 000 pikseliä
* **Oletusarvo**: 25 pikseliä
* **Kuvaus**: Määrittää vähimmäispinta-alan (pikseleinä), joka tarvitaan, jotta havaittu alue voidaan katsoa kelvolliseksi kalibrointikohteen näytteeksi. Pienemmät arvot havaitsevat pienempiä kohteita, mutta voivat lisätä vääriä positiivisia tuloksia. Suuremmat arvot vaativat suurempia ja selkeämpiä kohdealueita tunnistusta varten.
* **Milloin säätää**:
  * Lisää arvoa, jos saat vääriä tunnistuksia pienistä kuvan artefakteista
  * Vähennä arvoa, jos kalibrointikohteesi näyttävät pieniltä kuvissasi eivätkä tule tunnistetuiksi

### Kohteiden klusteroinnin vähimmäisarvo (0–100)

* **Tyyppi**: Luku
* **Alue**: 0–100
* **Oletusarvo**: 60
* **Kuvaus**: Säätää ryhmittelykynnystä samanväristen alueiden ryhmittelyyn kalibrointikohteita tunnistettaessa. Suuremmat arvot vaativat useampien samanlaisten värien ryhmittelyä yhteen, mikä johtaa konservatiivisempaan kohteen tunnistukseen. Pienemmät arvot sallivat enemmän värivaihtelua kohderyhmän sisällä.
* **Milloin säätää**:
  * Lisää arvoa, jos kalibrointikohteet jakautuvat useisiin havaintoihin
  * Vähennä arvoa, jos värivaihteluita sisältäviä kalibrointikohteita ei tunnisteta kokonaan

***

## Käsittely

Nämä asetukset säätelevät, miten Chloros käsittelee ja kalibroi kuvasi.

### Vignetoinnin korjaus

* **Tyyppi**: Valintaruutu
* **Oletus**: Käytössä (valittuna)
* **Kuvaus**: Soveltaa vignetoinnin korjausta kompensoimaan objektiivin tummentumaa kuvien reunoilla. Vignetoituminen on yleinen optinen ilmiö, jossa kuvan kulmat ja reunat näyttävät tummemmilta kuin keskusta objektiivin ominaisuuksien vuoksi.
* **Milloin poistaa käytöstä**: Poista käytöstä vain, jos kameran ja objektiivin yhdistelmä on jo soveltanut vignettikorjausta tai jos haluat korjata vignetoinnin manuaalisesti jälkikäsittelyssä.

### Heijastavuuden kalibrointi / valkotasapaino

* **Tyyppi**: Valintaruutu
* **Oletus**: Käytössä (valittuna)
* **Kuvaus**: Ottaa käyttöön automaattisen heijastavuuden kalibroinnin käyttämällä kuvista havaittuja kalibrointikohteita. Tämä normalisoi heijastavuusarvot koko aineistossasi ja varmistaa yhdenmukaiset mittaukset valaistusolosuhteista riippumatta.
* **Milloin poistaa käytöstä**: Poista käytöstä vain, jos haluat käsitellä raakoja, kalibroimattomia kuvia tai jos käytät erilaista kalibrointiprosessia.

### Debayer-menetelmä

* **Tyyppi**: Pudotusvalikko
* **Vaihtoehdot**:
  * Vakio (Nopea, Keskilaatuinen)
  * Tekstuuritietoinen (hidas, korkein laatu) \[Chloros+]
* **Oletus**: Vakio (nopea, keskilaatu)
* **Kuvaus**: Valitsee demosaicing-algoritmin, jota käytetään raakojen Bayer-kuvioanturitietojen muuntamiseen täysvärikuviksi. ”Vakio (nopea, keskilaatu)” -menetelmä tarjoaa optimaalisen tasapainon käsittelynopeuden ja kuvanlaadun välillä. &quot;Tekstuuritietoinen (hidas, korkein laatu)&quot; \[Chloros+] käyttää korkealaatuista reuna-tietoista debayering-algoritmia yhdistettynä AI/ML-kohinanpoistomalliin, joka poistaa lähes kaiken debayering-kohinan. Tekstuuritietoinen malli vaatii GPU-muistia (VRAM) toimiakseen. Suosittelemme sen käyttöä, kun käytettävissä on yli 4 Gt VRAM-muistia, jotta käsittely on nopeampaa.
* **Huomautus**: Chloros:n tulevissa versioissa saatetaan lisätä uusia debayer-menetelmiä.

### Kalibroinnin vähimmäisväli

* **Tyyppi**: Luku
* **Alue**: 0–3 600 sekuntia
* **Oletus**: 0 sekuntia
* **Kuvaus**: Asettaa kalibrointikohteiden käytön välisen vähimmäisajan (sekunteina). Kun arvoksi asetetaan 0, Chloros käyttää jokaista havaittua kalibrointikohdetta. Kun arvoksi asetetaan suurempi luku, Chloros käyttää vain kalibrointikohteita, joiden välinen aika on vähintään tämä sekuntimäärä, mikä lyhentää käsittelyaikaa datasarjoissa, joissa kalibrointikohteita havaitaan usein.
* **Milloin säätää**:
  * Aseta arvoksi 0, jotta kalibrointitarkkuus on mahdollisimman suuri, kun valaistusolosuhteet vaihtelevat
  * Lisää arvoa (esim. 60–300 sekuntiin), jotta käsittely on nopeampaa, kun valaistus on tasainen ja kalibrointikohteiden kuvia on usein

### Valosensorin aikavyöhykkeen siirtymä

* **Tyyppi**: Luku
* **Alue**: -12 – +12 tuntia
* **Oletusarvo**: 0 tuntia
* **Kuvaus**: Määrittää valosensorin datan aikaleimojen aikavyöhykkeen siirtymän (tunteina UTC:stä). Tätä käytetään PPK-datatiedostoja (Post-Processed Kinematic) käsiteltäessä, jotta kuvien ottamisen ja GPS-datan välinen aikasynkronointi on oikea.
* **Milloin säätää**: Aseta tämä paikallisen aikavyöhykkeen siirtymäksi, jos PPK-datasi käyttää paikallista aikaa UTC:n sijaan. Esimerkiksi:
  * Tyynenmeren aika: -8 tai -7 (kesäajasta riippuen)
  * Itäinen aika: -5 tai -4 (kesäajasta riippuen)
  * Keski-Euroopan aika: +1 tai +2 (kesäajasta riippuen)

### Käytä PPK-korjauksia

* **Tyyppi**: Valintaruutu
* **Oletus**: Pois käytöstä (valinta ei ole valittuna)
* **Kuvaus**: Ottaa käyttöön jälkikäsitellyt kinemaattiset (PPK) korjaukset MAPIR DAQ -tallentimista, joissa on GPS (GNSS). Kun tämä on käytössä, Chloros käyttää projektikansiossasi olevia .daq-lokitiedostoja, jotka sisältävät valotustietotietoja, ja soveltaa tarkkoja maantieteellisiä korjauksia kuviisi.
* **Vaatimus**: Projektikansiossa on oltava .daq-lokitiedosto, joka sisältää valotustietojen merkintöjä
* **Milloin ottaa käyttöön**: On suositeltavaa ottaa PPK-korjaus aina käyttöön, jos .daq-lokitiedostossa on valotustietojen merkintöjä.

### Valotuspiste 1

* **Tyyppi**: Pudotusvalikko
* **Näkyvyys**: Näkyy vain, kun &quot;Käytä PPK-korjauksia&quot; on käytössä JA valotustietoja on saatavilla pisteelle 1
* **Vaihtoehdot**:
  * Projektissa havaitut kameramallien nimet
  * &quot;Älä käytä&quot; - Ohita tämä valotuspiste
* **Oletus**: Valitaan automaattisesti projektin kokoonpanon perusteella
* **Kuvaus**: Määrittää tietyn kameran valotuspinnille 1 PPK-ajan synkronointia varten. Valotuspinnit tallentavat tarkan ajankohdan, jolloin kameran suljin laukeaa, mikä on kriittistä tarkalle PPK-paikannukselle.
* **Automaattisen valinnan toiminta**:
  * Yksi kamera + yksi pinni: Valitsee kameran automaattisesti
  * Yksi kamera + kaksi pinniä: Pinni 1 määritetään automaattisesti kameralle
  * Useita kameroita: Manuaalinen valinta vaaditaan

### Valotuspistoke 2

* **Tyyppi**: Pudotusvalikko
* **Näkyvyys**: Näkyy vain, kun &quot;Käytä PPK-korjauksia&quot; on käytössä JA valotustietoja on saatavilla pistokkeelle 2
* **Vaihtoehdot**:
  * Projektissa tunnistetut kameramallien nimet
  * &quot;Älä käytä&quot; - Ohita tämä valotuspistoke
* **Oletus**: Valitaan automaattisesti projektin kokoonpanon perusteella
* **Kuvaus**: Määrittää tietyn kameran valotusnastaan 2:lle PPK-ajan synkronointia varten, kun käytetään kahden kameran kokoonpanoa.
* **Automaattisen valinnan toiminta**:
  * Yksi kamera + yksi nasta: Nasta 2 asetetaan automaattisesti tilaan &quot;Älä käytä&quot;
  * Yksi kamera + kaksi nastaa: Nasta 2 asetetaan automaattisesti tilaan &quot;Älä käytä&quot;
  * Useita kameroita: Manuaalinen valinta vaaditaan
* **Huomautus**: Samaa kameraa ei voi määrittää samanaikaisesti sekä nastalle 1 että nastalle 2.***

## Indeksi

Näiden asetusten avulla voit määrittää monispektriset indeksit analysointia ja visualisointia varten.

### Lisää indeksi

* **Tyyppi**: Erityinen indeksien määrittämispaneeli
* **Kuvaus**: Avaa interaktiivisen paneelin, jossa voit valita ja määrittää monispektriset kasvillisuusindeksit (NDVI, NDRE, EVI jne.) laskettavaksi kuvankäsittelyn aikana. Voit lisätä useita indeksejä, joista jokaisella on omat visualisointiasetuksensa.
* **Käytettävissä olevat indeksit**: Järjestelmä sisältää yli 30 ennalta määriteltyä monispektristä indeksiä, mukaan lukien:
  * NDVI (normalisoitu kasvillisuusindeksi)
  * NDRE (normalisoitu ero RedEdge)
  * EVI (parannettu kasvillisuusindeksi)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * Ja monia muita (katso täydellinen luettelo kohdasta [Monispektriset indeksikaavat](multispectral-index-formulas.md))
* **Ominaisuudet**:
  * Valitse valmiista indeksikaavoista
  * Määritä visualisoinnin värigradientit (LUT - hakutaulukot)
  * Aseta analyysin kynnysarvot
  * Luo mukautettuja indeksikaavoja

### Mukautetut kaavat (Chloros+ -ominaisuus)

* **Tyyppi**: Mukautettujen kaavojen määritelmien taulukko
* **Kuvaus**: Mahdollistaa mukautettujen monispektristen indeksikaavojen luomisen ja tallentamisen käyttämällä kaistamatematiikkaa. Mukautetut kaavat tallennetaan projektin asetusten mukana, ja niitä voidaan käyttää aivan kuten sisäänrakennettuja indeksejä.
* **Kuinka luoda**:
  1. Etsi indeksin määrityspaneelista mukautetun kaavan vaihtoehto
  2. Määritä kaava käyttämällä kaistatunnisteita (esim. NIR, Red, Green, Blue)
  3. Tallenna kaava kuvaavalla nimellä
* **Kaavan syntaksi**: Tuetaan tavallisia matemaattisia operaatioita, mukaan lukien:
  * Aritmeettiset: `+`, `-`, `*`, `/`
  * Suluissa laskutoimitusten järjestys
  * Kaistaviittaukset: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Vienti

Nämä asetukset määrittävät vietyjen käsiteltyjen kuvien muodon ja laadun.

### Kalibroitu kuvamuoto

* **Tyyppi**: Pudotusvalikko
* **Vaihtoehdot**:
  * **TIFF (16-bittinen)** - Pakkaamaton 16-bittinen TIFF-muoto
  * **TIFF (32-bittinen, prosentti)** - 32-bittinen liukulukuinen TIFF, jossa heijastusarvot on ilmaistu prosentteina
  * **PNG (8-bittinen)** - Pakattu 8-bittinen PNG-muoto
  * **JPG (8-bittinen)** - Pakattu 8-bittinen JPEG-muoto
* **Oletus**: TIFF (16-bittinen)
* **Kuvaus**: Valitsee tiedostomuodon käsiteltyjen ja kalibroitujen kuvien tallentamista varten.
* **Muoto-suositukset**:
  * **TIFF (16-bittinen)**: Suositellaan tieteelliseen analyysiin ja ammattimaisiin työnkulkuihin. Säilyttää maksimaalisen datan laadun ilman pakkausartefakteja. Paras monispektrianalyysiin ja jatkokäsittelyyn GIS-ohjelmistoissa.
  * **TIFF (32-bittinen, prosentti)**: Paras työnkulkuille, jotka vaativat heijastusarvoja prosentteina (0–100 %). Tarjoaa parhaan tarkkuuden radiometrisille mittauksille.
  * **PNG (8-bittinen)**: Sopii hyvin verkkokatseluun ja yleiseen visualisointiin. Pienemmät tiedostokoot häviöttömällä pakkauksella, mutta pienempi dynaaminen alue.
  * **JPG (8-bittinen)**: Pienimmät tiedostokoot, sopii parhaiten vain esikatseluun ja verkkonäyttöön. Käyttää häviöllistä pakkausta, joka ei sovellu tieteelliseen analyysiin.***

## Tallenna projektimalli

Tämän ominaisuuden avulla voit tallentaa nykyiset projektiasetuksesi uudelleen käytettäväksi malliksi.

* **Tyyppi**: Tekstinsyöttö + Tallenna-painike
* **Kuvaus**: Kirjoita asetusmallillesi kuvaava nimi ja napsauta tallennuskuvaketta. Malli tallentaa kaikki nykyiset projektiasetuksesi (kohteen tunnistus, käsittelyasetukset, indeksit ja vientimuoto) myöhempää käyttöä varten.
* **Käyttötapaukset**:
  * Luo malleja eri kamerajärjestelmille (RGB, monispektrinen, NIR)
  * Tallenna vakioasetukset tietyille viljelykasvilajeille tai analyysityönkuluille
  * Jaa yhdenmukaiset asetukset koko tiimille
* **Käyttöohjeet**:
  1. Määritä kaikki haluamasi projektin asetukset
  2. Kirjoita mallin nimi (esim. &quot;RedEdge Survey3 NDVI Standard&quot;)
  3. Napsauta tallennuskuvaketta
  4. Malli voidaan nyt ladata uusia projekteja luotaessa

***

## Tallenna projektikansio

Tämä asetus määrittää, mihin uudet projektit tallennetaan oletuksena.

* **Tyyppi**: Hakemistopolun näyttö + Muokkaa-painike
* **Oletus (Windows)**: `C:\Users\[Username]\Chloros Projects`
* **Oletus (Linux)**: `~/.local/share/chloros/projects`
* **Kuvaus**: Näyttää nykyisen oletuskansion, johon uudet Chloros-projektit luodaan. Napsauta muokkauskuvaketta valitaksesi toisen kansion.
* **Milloin muuttaa**:
  * Aseta verkkoasemaksi tiimiyhteistyötä varten
  * Vaihda asemaan, jossa on enemmän tallennustilaa suurille tietojoukoille
  * Järjestä projektit vuosittain, asiakkaittain tai projektityypeittäin eri kansioihin
* **Huomautus**: Tämän asetuksen muuttaminen vaikuttaa vain UUSIIN projekteihin. Olemassa olevat projektit pysyvät alkuperäisissä sijainneissaan.***

## Asetusten pysyvyys

Kaikki projektin asetukset tallennetaan automaattisesti projektitiedoston (`.mapir`-projektimuoto) mukana. Kun avaat projektin uudelleen, kaikki asetukset palautuvat täsmälleen sellaisina kuin ne olivat, kun suljit projektin.

### Asetusten hierarkia

Asetukset otetaan käyttöön seuraavassa järjestyksessä:

1. **Järjestelmän oletusasetukset** – Chloros:n määrittämät sisäänrakennetut oletusasetukset
2. **Malliasetukset** – Jos lataat mallin projektia luodessasi
3. **Tallennetut projektin asetukset** – Projektitiedoston mukana tallennetut asetukset
4. **Manuaaliset muutokset** – Kaikki muutokset, jotka teet nykyisen istunnon aikana

### Asetukset ja kuvankäsittely

Useimmat asetusten muutokset (erityisesti Käsittely- ja Vienti-kategorioissa) käynnistävät kuvien uudelleenkäsittelyn, jotta uudet asetukset tulevat voimaan. Jotkut asetukset ovat kuitenkin ”vain vientiä varten” eivätkä vaadi välitöntä uudelleenkäsittelyä:

* Tallenna projektimalli
* Työkansio
* Kalibroitu kuvamuoto (koskee vientiä)

***

## Parhaat käytännöt

1. **Aloita oletusasetuksilla**: Oletusasetukset toimivat hyvin useimmissa MAPIR-kamerajärjestelmissä ja tyypillisissä työnkulkuissa.
2. **Luo malleja**: Kun olet optimoinut asetukset tietylle työnkululle tai kameralle, tallenna ne malliksi varmistaaksesi yhdenmukaisuuden eri projekteissa.
3. **Testaa ennen täysimittaista käsittelyä**: Kun kokeilet uusia asetuksia, testaa niitä pienellä osalla kuvista ennen koko aineiston käsittelyä.
4. **Dokumentoi asetuksesi**: Käytä kuvaavia mallinimiä, jotka ilmaisevat kamerajärjestelmän, käsittelytyypin ja käyttötarkoituksen (esim. &quot;Survey3\_RGB\_NDVI\_Agriculture&quot;).
5. **Vientimuodon valinta**: Valitse vientimuoto lopullisen käyttötarkoituksen mukaan:
   * Tieteellinen analyysi → TIFF (16-bittinen tai 32-bittinen)
   * GIS-käsittely → TIFF (16-bittinen)
   * Nopea visualisointi → PNG (8-bittinen)
   * Jakaminen verkossa → JPG (8-bittinen)

***

Lisätietoja Chloros:n monispektrisistä indekseistä on sivulla [Monispektriset indeksikaavat](multispectral-index-formulas.md).
