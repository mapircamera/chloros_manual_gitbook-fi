# Kuvan avaaminen koko ruudulla

Chloros-kuvankatseluohjelma tarjoaa erillisen koko ruudun käyttöliittymän monispektrikuvien katseluun, analysointiin ja muokkaamiseen. Kuvankatseluohjelma tarjoaa tehokkaat työkalut tarkasteluun ja analysointiin sekä alkuperäisten kuvien että käsiteltyjen tulosteiden katseluun.

## Kuvankatseluohjelman avaaminen

### Tiedostoselaimesta

Yleisin tapa avata kuva kuvankatseluohjelmassa:

1. Varmista, että olet **Tiedostoselain**-välilehdessä <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Napsauta mitä tahansa **kuvien pikkukuvaa** kuvaruudukossa
3. Kuva avautuu **pääesikatselualueelle** (näytön keskelle)
4. Kuva on nyt ladattu ja valmis koko näytön katseluun

### Image Viewer -välilehden avaaminen

Kun kuva on ladattu esikatselualueelle:

1. Napsauta **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> -kuvaketta vasemmassa sivupalkissa.
2. Kuvankatseluohjelman välilehti avautuu ja näyttää valitun kuvan koko ruudulla.
3. Lisäasetukset ja analyysityökalut tulevat saataville vasemmassa sivupalkissa.

***

## Kuvankatseluohjelman käyttöliittymän yleiskatsaus

### Pääruutu

Suurin osa ruudusta näyttää kuvan:

* **Täysi resoluutio**: Kuvat näytetään alkuperäisellä resoluutiolla.
* **Zoomattava**: Zoomaa säätimillä tai hiiren rullalla
* **Panoroitava**: Siirrä kuvaa zoomatussa tilassa napsauttamalla ja vetämällä
* **Kuvasuhde säilyy**: Kuvat skaalautuvat suhteessa

***

## Katseluvaihtoehdot

### Peruskuvanavigointi

#### Selaa kuvia

Selaa kuvasarjaa näppäimistön pikanäppäimillä tai painikkeilla:

* **Seuraava kuva**: Napsauta →-painiketta tai paina **→** (oikea nuoli) -näppäintä
* **Edellinen kuva**: Napsauta ←-painiketta tai paina **←** (vasen nuoli) -näppäintä
* **Siirry tiettyyn kuvaan**: Palaa tiedostoselaimeen ja napsauta haluamaasi pikkukuvaa

#### Zoomaustoiminnot

Säädä suurennusta kuvan yksityiskohtien tarkastelemiseksi:

**Lähennä:**

* Napsauta **+** (plus) -painiketta
* Paina **+** tai **=** -näppäintä
* Vieritä hiiren rullaa **ylöspäin**

**Kavenna:**

* Napsauta **−** (miinus) -painiketta
* Paina **−** (miinus) -näppäintä
* Vieritä hiiren rullaa **alaspäin**

#### Panorointi zoomattuna

Kun olet zoomannut yli näytön koon:

1. Siirrä hiiren osoitin kuvan päälle
2. Napsauta ja **pidä hiiren vasen painike painettuna**
3. **Vedä** siirtääksesi kuvaa
4. Vapauta painike lopettaaksesi panoroinnin

**Vaihtoehto**: Käytä nuolinäppäimiä panoroidaksesi pienin askelin

***

## Pikseliarvojen tarkastelu

### Pikseliarvojen tarkastelu kursorilla

Kun siirrät hiiren osoittimen kuvan päälle, pikseliarvot näkyvät reaaliajassa:

**Arvon näyttöpaikka:**

* **Kelluva luku ja punainen viiva oikeanpuoleisessa indeksi-LUT-gradienttilegendassa**
* **Kun zoomataan edelleen, kelluva arvo osoittimen lähellä ja korostettu pikseli**
* Näyttää pikselin arvot **osoittimen alla tai korostettuna**
* Päivittyy, kun siirrät hiirtä

***

## Kuvat, joita voit tarkastella

### JPG

**Kameran JPG-kuvat:**

* Näytä JPG-tiedot esikatseluna
* Näytä alkuperäiset, korjaamattomat arvot
* Hyödyllinen kuvan laadun tarkistamiseen ennen käsittelyä

### RAW (alkuperäinen)

### RAW (heijastavuus)

**Käsittelyn jälkeen:**

* Vignette korjattu
* Heijastavuus kalibroitu
* Monikaistainen TIFF (Red, Green, NIR jne.)
* Tieteelliset tiedot valmiina analysoitavaksi

### RAW (indeksi)

**NDVI, NDRE, GNDVI jne. (\_NDVI.tif-tiedostot):**

* Yksikaistaiset harmaasävykuvat
* Pikseliarvot edustavat indeksin laskentatuloksia
* Normaalisoiduille indekseille tyypillinen alue on -1–+1
* Visualisointia varten voidaan käyttää väri-LUT-taulukoita

***

## Indeksin ja LUT-taulukon käyttö

Käytä monispektrisiä indeksejä ja väri-LUT-taulukoita:

1. Etsi **Index/LUT Sandbox** **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sivupalkista
2. Valitse kasvillisuusindeksi (NDVI, NDRE jne.)
3. Valitse monispektrinen kaava tai luo oma mukautettu kaava (vain Chloros+)
4. Käytä värilukutaulukon gradienttia visualisointia varten
5. Säädä arvoalueita ja kynnysarvoja

Katso yksityiskohtaiset ohjeet kohdasta [Indeksi/LUT-hiekkalaatikko](index-lut-sandbox.md).

***

## Pikanäppäimet

### Navigointi

* **→** (oikea nuoli): Seuraava kuva
* **←** (vasen nuoli): Edellinen kuva
* **Koti**: Luettelon ensimmäinen kuva
* **End**: Luettelon viimeinen kuva

### Zoomaaminen

* **+** tai **=**: Lähennä
* **−**: Loitonna
* **Hiiren rulla**: Lähennä/loitonna

***

### Indeksilaskennan tarkistaminen

Tarkista, että indeksit on laskettu oikein:

1. Avaa NDVI tai muu indeksikuva
2. Tarkista kasvillisuusalueet:
   * **NDVI**: Terveiden kasvien arvojen tulisi olla 0,4–0,9
   * **NDRE**: Voimakkaan kasvun arvojen tulisi olla korkeammat
   * **GNDVI**: Samanlainen kuin NDVI, mutta klorofylliherkkä
3. Tarkista kasvillisuuden ulkopuoliset alueet:
   * **Maaperä**: Lähes 0 tai hieman negatiivinen
   * **Vesi**: Negatiiviset arvot (-0,5–0)

***

## Näyttöongelmien vianmääritys

### Kuva ei avaudu

**Mahdolliset syyt:**

* Tiedosto on vioittunut käsittelyn aikana
* Tiedostomuoto ei ole tuettu
* Muisti ei riitä suurelle kuvalle

**Ratkaisut:**

1. Yritä avata tiedosto ulkoisella katseluohjelmalla tiedoston eheyden tarkistamiseksi.
2. Tarkista, että tiedostomuoto vastaa odotettua tyyppiä.
3. Sulje muut sovellukset muistin vapauttamiseksi.
4. Kokeile pienempää/eri kuvaa.

### Musta tai valkoinen kuva

**Mahdolliset syyt:**

* Arvoalue ylittää näytön kapasiteetin.
* 32-bittinen kelluva kuva, jossa on epätavallisia arvoja.
* Indeksin laskentavirhe.

**Ratkaisut:**

1. Tarkista pikseliarvot – jos ne ovat kaikki hyvin alhaisia tai korkeita, säädä näyttöaluetta.
2. Yritä avata kuva QGIS:ssä tai vastaavassa ohjelmassa, jossa on automaattinen alueiden säätö.
3. Tarkista käsittelyn virhelokista, onko siinä virheitä.

### Pikseliarvot näyttävät virheellisiltä

**Mahdolliset syyt:**

* Katselet väärää kuvaa (alkuperäinen vs. käsitelty)
* Kalibrointia ei ole sovellettu oikein
* Valosensorin tietoja ei ole sisällytetty syötteeseen
* Prosenttimuoto on kytketty väärin

**Ratkaisut:**

1. Varmista, että katselet käsiteltyä tulosta (tarkista tiedostonimen pääte)
2. Tarkista prosenttimuodon painikkeen tila
3. Vertaa samaan tietojoukkoon kuuluvia tunnettuja hyviä kuvia

***

## Seuraavat vaiheet

Nyt kun voit tarkastella kuvia koko näytön tilassa:

* [**Kuvakerrokset**](image-layers.md) – Lisätietoja monikaistaisesta visualisoinnista
* [**Indeksi/LUT-hiekkalaatikko**](index-lut-sandbox.md) – Mukautettujen indeksien ja värikartoitusten soveltaminen
* [**Monispektriset indeksikaavat**](../project-settings/multispectral-index-formulas.md) – Tutustu käytettävissä oleviin indekseihin

Käsittelyn työnkulusta katso:

* [**Kuvien käsittely (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Kattava käsittelyopas
