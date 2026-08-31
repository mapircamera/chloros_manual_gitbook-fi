# Kuvan avaaminen koko näytön tilassa

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption><p>Kuva avattuna koko näytön tilassa, kerroksen valitsin oikeassa yläkulmassa</p></figcaption></figure>

Chloros-kuvankatseluohjelma on koko näytön käyttöliittymä kuvien katseluun, tarkasteluun ja mittaamiseen. Siellä voit lukea **todelliset pikseliarvot** — kanavakohtaisen DN-arvon, heijastusprosentin tai säteilyvoimakkuuden yksikössä W/m²/sr/nm — sen sijaan, että katsoisit näytöllä näkyvää venytettyä esikatselukuvaa.

## Image Viewerin avaaminen

### Tiedostoselaimesta

1. Avaa **File Browser**-välileh<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Napsauta mitä tahansa **pikkukuvaa** [kuvaruudukossa](image-grid.md)
3. Kuva avautuu koko ruudun kokoisena **Kuvankatseluohjelma**-välilehdessä

Kuva avautuu siinä tuotteessa, jota ruudukko juuri näytti. Jos ruudukko on asetettu `RAW (Reflectance)`:ään, pääset juuri kyseiselle tasolle.

### Kuvankatseluohjelman sivupalkin avaaminen

Napsauta vasemmassa sivupalkissa olevaa **Kuvankatseluohjelman** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">-kuvaketta, jolloin analyysipaneeli avautuu. Se sisältää ylhäältä alas:

* kuvan nimen ja sen kameramallin
* **Vie/Tallenna kuva(t)** -painikkeen (vain kun indeksi tai LUT on aktiivinen)
* **Indeksi**- ja**LUT**-valintaruudut sekä indeksin asetuspaneeli — katso [Indeksi/LUT-kokeilualusta](index-lut-sandbox.md)
* **Kohdistimen arvot** -paneeli: kanavakohtainen lukema, kerroksen histogrammi ja GSD-säädin***

## Liikkuminen ja zoomaus

### Kuvien selaaminen

* **Seuraava kuva**: →-painike tai**→** (oikea nuoli) -näppäin
* **Edellinen kuva**: ←-painike tai**←** (vasen nuoli) -näppäin
* **Siirry tiettyyn kuvaan**: palaa ruudukkoon ja napsauta sen pikkukuvaa

Zoomaaminen ja panorointi säilyvät, kun siirryt kuvien välillä, joten voit selata kuvasarjaa pysyen samalla kuvan osalla.

### Zoomaaminen

Zoomausta ohjataan **hiiren rullalla** 15 %:n välein, ja se on kiinnitetty kursoriin — osoittimen alla oleva piste pysyy osoittimen alla. Zoomaustarkkuuden alue määräytyy kuvan ja ikkunan koon mukaan: et voi zoomata ulos pidemmälle kuin ikkunaan mahtuvaksi, ja yläraja määräytyy kuvan alkuperäisen resoluution mukaan.

Koko näytön katseluohjelmassa ei ole erillisiä zoomausnäppäimiä. (Ruudukossa **Ctrl + `+` / `−`** muuttaa pikkukuvien kokoa – tämä on eri toiminto.)

### Panorointi zoomattuna

Napsauta ja pidä hiiren vasenta painiketta painettuna kuvan päällä ja vedä. Panorointia on rajoitettu, joten kuvaa ei voi vetää pois näytöltä.

### Pikselikohtainen tarkastelu suuressa zoomauksessa

Kun tehollinen suurennus ylittää **60×**, Chloros piirtää korostusruudun kursorin alla olevan yksittäisen näytetyn pikselin ympärille ja kelluvan arvon sen viereen.

”Tehollinen” suurennus perustuu GSD-lohkon kokoon: kun lohkon koko on 8, korostus ilmestyy jo 7,5×-suurennuksella eikä vasta 60×:lla, koska yksi näytössä näkyvä pikseli vastaa jo 8 × 8 lähdepikseliä. Kun zoomaat takaisin alle kynnysarvon, korostus katoaa.

### Pikanäppäimet

| Näppäin                             | Missä       | Toiminto                              |
| ------------------------------- | ----------- | ----------------------------------- |
| **→**                           | Koko näyttö | Seuraava kuva                          |
| **←**                           | Koko näyttö | Edellinen kuva                      |
| **Ctrl + R**                    | Koko näyttö | Nollaa hakemisto/LUT-hiekkalaatikko         |
| **Ctrl + `+`**/**Ctrl + `=`** | Ruudukko        | Suuremmat pikkukuvat (4 px painallusta kohti)  |
| **Ctrl + `−`**                  | Ruudukko        | Pienemmät pikkukuvat (4 px painallusta kohti) |***

## Kursorin arvot

Vie kursori kuvan päälle, niin **Kursorin arvot** -paneeli näyttää sen alla olevan jokaisen kanavan arvon.

{% hint style="success" %}
**Nämä ovat tiedoston todelliset arvot.** Näytöllä näkyvä kanka on 8-bittinen venytetty esikatselu, joka ei voi näyttää näitä arvoja, joten Chloros ottaa näytteitä varsinaisesta tuotetiedostosta lukemien tuottamiseksi. Siksi 12-bittinen raakakuva näyttää arvoja yli 255 ja float32-radiance-kerros näyttää fysikaalisia yksiköitä.
{% endhint %}

### Sarakkeiden merkitys

Paneeli mukautuu katsomasi kerroksen mukaan:

| Katsomasi kerros              | Näytettävät sarakkeet    | Huomautukset                                                                                           |
| ---------------------------------- | ---------------- | ----------------------------------------------------------------------------------------------- |
| Heijastavuus                        | **DN**ja**%** | Prosenttiosuus lasketaan kyseisen tiedoston omalla asteikolla — katso alla                                      |
| Säteilyvoimakkuus                           | **W/m²/sr/nm**   | Float-muotoiset fysikaaliset arvot; DN-saraketta ei ole, koska DN:llä ei ole tässä yhteydessä merkitystä                           |
| Raaka / Debayeroitu / esikatselu / JPG    | **DN**           | Digitaaliset kokonaisluvut                                                                         |
| 32-bittiset heijastusprosenttivienti | vain **%**       | Tallennettu liukuluku ei ole DN, joten sen pyöristäminen kokonaislukuksi tulostaisi merkityksettömän `0` tai `1` |

Jokainen rivi on merkitty kamerasi suodattimen kanavanimellä — `Red / Green / NIR` vastaa RGN:ää, `Orange / Cyan / NIR` kameralle OCN, `NIR / Green / Blue` kameralle NGB, `Red / Green / Blue` kameralle RGB:lle sekä yksikaistaisille RE-, NIR- ja mono-M3M-kameroille. Jokaisessa tarrassa on värillinen piste, joka vastaa indeksikaavan editorissa käytettyjä kanavapiirejä.

Tallennetut **indeksi- ja LUT-**kuvat ovat erityistapaus: ne sisältävät värikarttakomponentteja spektrikaistojen sijaan, joten niiden rivit on nimetty `Red / Green / Blue` (tai `Index` yksikanavaisen indeksitiedoston tapauksessa) kameran suodatinten nimien sijaan.

Kun indeksi on aktiivinen hiekkalaatikossa, kanavien alle ilmestyy ylimääräinen rivi, joka näyttää **indeksiarvon** kohdistimen kohdalla sekä indeksin nimen ja valkoisen pisteen, joka vastaa sen merkkiä histogrammissa.

### Heijastusprosentti käyttää kunkin tiedoston omaa asteikkoa

{% hint style="warning" %}
**Älä oleta, että 65535 = 100 %.** Chloros tallentaa heijastavuuden eri asteikoilla sen mukaan, mikä kamera sen on tuottanut, ja katseluohjelma määrittää tiedostokohtaisesti oikean asteikon.
{% endhint %}

| Lähde                  | DN, joka vastaa heijastusarvoa 1,0 | Miten se tunnistetaan                                                                                                                               |
| ----------------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **LATTICE**(M3C / M3M) |**32768**                      | XMP-tunniste `Chloros:PixelScale=32768`, joka on kirjoitettu jokaiseen LATTICE-heijastavuusvientiin. 2×:n liikkumavara mahdollistaa ρ-arvon yli 1,0 ilman leikkautumista |
| **Survey3**|**65535**                      | Ei Chloros-XMP-skaalausmerkintää — Survey3-kalibrointi kirjoittaa ρ × dtype-max ja leikkaa arvon 1,0:ssa                                                               |

Katseluohjelma, indeksi-/LUT-hiekkalaatikko ja indeksin vienti määrittävät skaalauksen kaikki saman yksittäisen toteutuksen kautta, joten kursorin kohdalla näkyvä arvo on sama kuin indeksin laskennassa käytetty arvo.

Kaksi huomionarvoista seikkaa:

* **32-bittinen prosenttiarvo**TIFF tallentaa DN/65535:n float-muodossa, ja**8-bittinen** PNG/JPG-vienti tallentaa DN × 255/65535 — katseluohjelma muuntaa molemmat takaisin ennen prosenttiluvun tulostamista.
* Yhtä tapausta ei voida korjata: **8-bittisen lähdekuvan 8-bittinen TIFF-vienti** rajataan arvoihin 0–255 sen sijaan, että sitä skaalattaisiin uudelleen, eikä siinä ole tarkoituksella skaalausmerkintää. Näiden tiedostojen kohdalla paneeli tulostaa vain DN-arvon ilman prosenttisaraketta. Tämä on rehellinen vastaus, ei vika.***

## Kerroksen histogrammi

Kohdistimen rivien alapuolella on katsomasi kerroksen reaaliaikainen histogrammi, **256 luokassa**. Oletuksena se piirtää yhden yhdistelmäkäyrän, painotettuna `(R + 2G + B) / 4` — sama mittaustila, jota LATTICE-kameran histogrammit käyttävät. Kun**RGB** otetaan käyttöön, se korvataan kanavakohtaisilla käyrillä kanavien väreissä, jotka on sekoitettu additiivisesti, jotta päällekkäisyydet pysyvät luettavina. Monokromakerrokset piirtävät aina yhden käyrän.

Vaakasuora akseli on kerroksen omassa yksikössä:

| Kerros       | Akselin yksikkö  | Akselin enimmäisarvo                                               |
| ----------- | ---------- | ---------------------------------------------------------- |
| Heijastavuus | prosenttia    | 125 % — tuotteen liikkumavara sallii ρ:n arvon yli 1,0           |
| Säteilyvoimakkuus | W/m²/sr/nm | Kehyksen oma huippuarvo, pyöristettynä kahteen merkitsevään numeroon |
| 8-bittiset tiedot | DN         | 255                                                        |
| 12-bittiset tiedot | DN         | 4095                                                       |
| 16-bittiset tiedot | DN         | 65535                                                      |

Kun akseli on DN-tilassa ja osuu johonkin näistä kolmesta ylärajasta, Chloros tunnistaa myös tarkasteltavan kuvan bittisyvyyden.

Histogrammin yläpuolella on kolme painiketta:

| Painike     | Oletusarvo | Vaikutus                                                                                                                                                                                                                                                                                   |
| ---------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **KURSSORI** | Päällä      | Piirtää histogrammiin merkkiviivoja juuri niillä arvoilla, jotka näkyvät yllä olevissa riveissä, jotta voit nähdä, missä kohdassa kehyksen jakaumaa kursorin alla oleva pikseli sijaitsee. RGB-tilassa on yksi merkkiviiva kanavaa kohti omalla värillään; muutoin yksi valkoinen merkkiviiva yhdistettyyn arvoon |
| **INDEX**| Päällä      | Näkyy vain, kun indeksi on aktiivinen. Vaihdaa histogrammin lähdekaistoista**indeksiarvon jakaumaan**, jolloin kaksi leikkauskynnystä on merkitty oransseina katkoviivoina ja kursorin indeksiarvo valkoisena viivana                                                          |
| **RGB**| Pois     | Vaihtaa yhdistetyistä käyristä kanavakohtaisiin käyriin. Monosensorissa tämän painikkeen teksti on**MONO** ja se on pois käytöstä — näytettävää kanavaa on vain yksi                                                                                                                                  |

Histogrammi lasketaan **näkyvissä olevien lohkojen** perusteella, ei niiden takana olevien lähdepikselien: kun muutat GSD-lohkon kokoa, jakauma lasketaan uudelleen, jolloin histogrammi, kursorimerkki ja näytettävä kuva ovat aina yhdenmukaisia.***

## GSD-lohkon koko

Paneelin alaosassa on **GSD (px)**-säädin: numerokenttä, liukusäädin**1:stä 256:een**ja**RESET**-painike.

Se karkeistaa _näytettävää_ kuvaa laskemalla N × N-lohkon lähdepikseleistä keskiarvon, joka muodostaa yhden näytettävän pikselin. `1` on alkuperäinen resoluutio.

* Se vaikuttaa **koko näytön näkymään, ruudukon pikkukuviin, kohdistimen lukemaan sekä molempiin histogrammeihin** — kaikki, mikä näyttää kuvaa, käyttää samaa perusresoluutiota.
* Se koskee **vain näyttöä**. Käsittely ja vienti eivät muutu. Ainoa poikkeus on tarkoituksellinen: [Index/LUT Sandbox](index-lut-sandbox.md) -vienti tallentaa sen, mitä parhaillaan katsot, joten se säilyttää nykyisen lohkokoon, ja vientipaneeli varoittaa, kun lohkoko on yli 1.
* Arvo tallennetaan **projektikohtaisesti** nimellä `viewer_display.gsd_bin` tiedostossa `project.json`, joten se säilyy ohjelman sulkemisen ja uudelleen avaamisen jälkeen.
* Kursorin lukema ilmoittaa lohkon arvon, ei lähdepikselin arvoa, aina kun lohkon koko on yli 1 — näytetty arvo on kursorin alla olevan lohkon keskiarvo.

{% hint style="info" %}
**Miksi ”lohkon koko” eikä senttimetrejä pikseliä kohti?** Cm/px-arvo edellyttää korkeutta maanpinnasta. Yksittäisen kuvan EXIF-tiedoissa on GPS-korkeus merenpinnan yläpuolella, ei sen maaston yläpuolella, johon kamera oli suunnattu, joten Chloros ei tulosta maastoväliä, jota se ei voi luotettavasti laskea. Lohkon koko lähdepikseleinä on sama varamenetelmä, jota MAPIR-pilvityökalut käyttävät, kun maastonäytteen etäisyys on tuntematon.
{% endhint %}

***

## Katseltavissa olevat kuvatyypit

Katseluohjelman oikeassa yläkulmassa oleva kerrosten pudotusvalikko listaa kaikki nykyisen kuvan versiot. Mitkä merkinnät näkyvät, riippuu kamerasta ja siitä, mitä on käsitelty — katso [Kuvakerrokset](image-layers.md) saadaksesi täydellisen luettelon ja tietoa pudotusvalikon toiminnasta.

### Survey3

* **JPG** — kameran oma esikatselutiedosto
* **RAW (Original)** — lähdetiedosto `.RAW`, josta on poistettu bayeri-kuvio näyttöä varten, ilman korjauksia
* **RAW (kohde)** — kehys, jonka on tunnistettu sisältävän kalibrointikohteen
* **RAW (heijastavuus)** — kalibroitu heijastavuustuote (65535 = ρ 1,0)
* **Vignette Corrected**/**Sensor Response** — kalibroimaton varatuote
* **Valkotasapainotettu** — valkotasapainotettu tuote
* **RAW (`<INDEX>`-indeksi)**ja**`<INDEX>` LUT** — lasketut indeksikuvat

### LATTICE

LATTICE-kaappauksissa käytetään samaa pudotusvalikkoa, jossa on prosessiketjun tasojen nimet:

| Kerros                 | Mitä se sisältää                                                        |
| --------------------- | -------------------------------------------------------------------- |
| **RAW (alkuperäinen)**    | Tallennettu raakakuva                                     |
| **RAW (debayeroitu)**   | Lineaarinen debayeroitu kuva                                           |
| **RAW (esikatselu)**     | Näytön esikatselu — väärävärinen venytys monispektrikameroille |
| **Valkotasapainotettu**    | Näytön esikatselu RGB-pääkameroille (valkotasapaino + gamma)   |
| **RAW (säteilyvoimakkuus)**    | Float32-muotoinen spektrinen säteilyvoimakkuus yksikössä W/m²/sr/nm                              |
| **RAW (heijastavuus)** | uint16-muotoinen heijastavuus, 32768 = ρ 1,0                                    |

Säteilyvoimakkuus ja heijastavuus ovat käytettävissä vain monispektrikuvissa: RGB-pääkamerassa ei ole kaistakohtaista radiometriaa, joten kyseisiä kerroksia ei tuoteta sille.

***

## Indeksien ja LUT-taulukoiden käyttö

Käytä monispektrisiä indeksejä ja väri-LUT-taulukoita sivupalkista:

1. Avaa **Kuvankatseluohjelma** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">-sivupalkista
2. Valitse **Indeksi**

3. Valitse kamerasi suodatin ja indeksikaava, ja vedä sitten kanavaympyrät kaavan paikoille
4. Lisää LUT ja valitse gradientti, kynnysarvot ja leikkaustila
5. Lue arvot kursorin kohdalta ja tallenna tulos valitsemalla **Vie/Tallenna kuva(s)**Katso [Indeksi/LUT-kokeilualusta](index-lut-sandbox.md) saadaksesi täydellisen ohjeen.***

## Vianmääritys

### Kuva ei avaudu

**Mahdolliset syyt**: tiedosto on siirretty tai poistettu tuonnin jälkeen; tuotetta ei ole koskaan tallennettu; muistia ei ole tarpeeksi erittäin suurelle kuvalle.**Toimenpiteet**:

1. Tarkista, että kerroksen tiedosto on edelleen olemassa projektin tulostuspuussa
2. Avaa tiedosto ulkoisessa katseluohjelmassa varmistaaksesi, että se on ehjä
3. Sulje muut sovellukset muistin vapauttamiseksi

### Kuva on musta, valkoinen tai värit ovat sekavia

**Mahdolliset syyt**: näytön venytyksellä ei ole mitään käsiteltävää (lähes vakioinen kehys); float32-kerros, jossa on epätavallisia arvoja; indeksi, joka ei tuottanut kelvollista dataa.**Mitä tehdä**:

1. Lue kursorin arvot – jos jokainen kanava on nollassa tai lähellä nollaa, ongelma on tiedoissa, ei näytössä
2. Tarkista histogrammi: yksittäinen piikki toisessa päässä kertoo, että kehys on leikattu tai tyhjä
3. Tarkista sen ajon käsittelyloki, joka tuotti kerroksen

### Arvot näyttävät virheellisiltä

**Mahdolliset syyt**: olet eri kerroksessa kuin luulet; vertailet prosenttiarvoa raaka-DN-arvoon; vertailet LATTICE-tiedostoa Survey3-tiedostoon käyttäen samaa jakajaa.**Mitä tehdä**:

1. Varmista valittu kerros pudotusvalikosta — paneelin yksiköt noudattavat kerrosta
2. Heijastavuuden osalta käytä **%**-saraketta sen sijaan, että jakaisit DN-arvon itse; jos jakaminen on välttämätöntä, käytä kyseisen tiedoston `Chloros:PixelScale`-arvoa (32768 LATTICE-tiedostolle, puuttuva arvo tarkoittaa 65535:tä Survey3-tiedostolle)
3. Aseta GSD-lohkon koko takaisin arvoon 1 — arvon 1 yläpuolella luet lohkon keskiarvoa, et pikseliä
4. Tarkista, että heijastavuuden kalibrointi todella suoritettiin kyseiselle kehykselle; kalibroimaton varatuote (Sensor Response / Vignette Corrected) ei ole heijastavuus

***

## Seuraavat vaiheet

* [**Kuvakerrokset**](image-layers.md) — jokaisen kerroksen nimi, mikäli sellainen on, ja sen arvojen merkitys
* [**Indeksi/LUT-hiekkalaatikko**](index-lut-sandbox.md) — indeksin visualisointien luominen, hienosäätö ja vienti
* [**Karttamerkit**](map-markers.md) — sama kuvasarja kartalla
* [**Monispektriset indeksikaavat**](../project-settings/multispectral-index-formulas.md) — indeksiviite

Käsittelytyönkulusta katso [Kuvien käsittely (GUI)](../processing-images-gui/adding-files-to-a-project.md).
