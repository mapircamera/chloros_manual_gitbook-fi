# Kuvan avaaminen koko näytön tilassa

Chloros-kuvankatseluohjelma tarjoaa erillisen koko näytön käyttöliittymän monispektrikuvien katseluun, analysointiin ja muokkaamiseen. Olipa kyseessä alkuperäiset kuvat tai käsitellyt tulokset, kuvankatseluohjelma tarjoaa tehokkaat työkalut tarkasteluun ja analysointiin.

## Image Viewerin avaaminen

### Tiedostoselaimesta

Yleisin tapa avata kuva Image Viewerissa:

1. Varmista, että olet **Tiedostoselain**-välilehdellä <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Napsauta mitä tahansa **kuvan pikkukuvaa** kuvaruudukossa
3. Kuva avautuu **pääesikatselualueelle** (näytön keskelle)
4. Kuva on nyt ladattu ja valmis katsottavaksi koko näytön tilassa

### Image Viewer -välilehden avaaminen

Kun kuva on ladattu esikatselualueelle:

1. Napsauta **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> -kuvaketta vasemmassa sivupalkissa
2. Kuvankatselu-välilehti avautuu ja näyttää valitun kuvan koko näytön tilassa
3. Lisäasetukset katseluun ja analysointiin tulevat näkyviin vasemmassa sivupalkissa

***

## Kuvankatselu-käyttöliittymän yleiskatsaus

### Päänäyttöalue

Suurin osa näytöstä näyttää kuvasi:

* **Täysi resoluutio**: Kuvat näytetään alkuperäisellä resoluutiolla
* **Zoomaaminen**: Zoomaa säätimillä tai hiiren rullalla
* **Panorointi**: Napsauta ja vedä liikkua zoomatussa tilassa
* **Kuvasuhde säilyy**: Kuvat skaalautuvat suhteessa***

## Katseluvaihtoehdot

### Peruskuvanavigointi

#### Selaa kuvia

Selaa kuvasarjaasi pikanäppäimillä tai painikkeilla:

* **Seuraava kuva**: Napsauta →-painiketta tai paina**→** (oikea nuoli) -näppäintä
* **Edellinen kuva**: Napsauta ←-painiketta tai paina**←** (vasen nuoli) -näppäintä
* **Siirry tiettyyn kuvaan**: Palaa tiedostoselaimeen ja napsauta haluamaasi pikkukuvaa

#### Zoomaustyökalut

Säädä suurennusta kuvan yksityiskohtien tarkastelemiseksi:

**Lähentäminen:*** Napsauta **+** (plus) -painiketta
* Paina **+**- tai**=**-näppäintä
* Vieritä hiiren rullaa **ylöspäin**

**Loitontaminen:*** Napsauta **−** (miinus) -painiketta
* Paina **−** (miinus) -näppäintä
* Vieritä hiiren rullaa **alaspäin**

#### Panorointi zoomattuna

Kun olet zoomannut kuvan näytön koon ulkopuolelle:

1. Siirrä hiiren osoitin kuvan päälle
2. Napsauta ja **pidä hiiren vasenta painiketta painettuna**

3.**Vedä** siirtääksesi kuvaa
4. Vapauta lopettaaksesi panoroinnin

**Vaihtoehto**: Käytä nuolinäppäimiä panoroidaksesi pienin askelin***

## Pikseliarvojen tarkastelu

### Pikseliarvojen tarkastelu kohdistimen kohdalla

Kun siirrät hiiren kohdistinta kuvan päälle, pikseliarvot näkyvät reaaliajassa:**Arvon näyttöpaikka:*** **Kelluva luku ja punainen viiva oikeanpuoleisen indeksi-LUT-gradientin selitteessä*** **Kun zoomataan vielä pidemmälle, kelluva arvo kohdistimen ja korostetun pikselin lähellä*** Näyttää arvot **kohdistimen alla tai korostetulle** pikselille
* Päivittyy hiiren liikkuessa

***

## Katsottavat kuvatyypit

### JPG

**Kameran JPG-kuvat:**

* Näyttää JPG-tiedot esikatseluna
* Näyttää alkuperäiset, korjaamattomat arvot
* Hyödyllinen kuvan laadun tarkistamiseen ennen käsittelyä

### RAW (Alkuperäinen)

### RAW (Heijastavuus)

**Käsittelyn jälkeen:**

* Vignetoinnin korjaus
* Heijastavuuden kalibrointi
* Monikaistainen TIFF (Red, Green, NIR jne.)
* Tieteelliset tiedot valmiina analysoitavaksi

### RAW (indeksi)

**NDVI, NDRE, GNDVI jne. (\_NDVI.tif-tiedostot):**

* Yksikaistaiset harmaasävykuvat
* Pikseliarvot edustavat indeksin laskentatuloksia
* Alue tyypillisesti -1 – +1 normalisoiduille indekseille
* Värillisiä LUT-taulukoita voidaan käyttää visualisointiin

***

## Indeksin ja LUT-taulukon käyttö

Käytä monispektrisiä indeksejä ja värillisiä Look-Up-taulukoita:

1. Etsi **Index/LUT Sandbox** **Image Viewer** -sovelluksesta <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sivupalkissa
2. Valitse kasvillisuusindeksi (NDVI, NDRE jne.)
3. Valitse monispektrinen kaava tai luo oma mukautettu kaava (vain Chloros+)
4. Käytä värillistä LUT-gradienttia visualisointiin
5. Säädä arvoalueita ja kynnysarvoja

Katso [Indeksi/LUT-hiekkalaatikko](index-lut-sandbox.md) saadaksesi yksityiskohtaiset ohjeet.

***

## Pikanäppäimet

### Navigointi

* **→** (Oikea nuoli): Seuraava kuva
* **←** (Vasen nuoli): Edellinen kuva
* **Home**: Luettelon ensimmäinen kuva
* **End**: Luettelon viimeinen kuva

### Zoomaaminen

* **+**tai**=**: Lähentää
* **−**: Loitontaa
* **Hiiren rulla**: Lähentää/loitontaa***

### Indeksilaskentojen tarkistaminen

Tarkista, että indeksit on laskettu oikein:

1. Avaa NDVI tai muu indeksikuva
2. Tarkista kasvillisuusalueet:
   * **NDVI**: Terveiden kasvien arvon tulisi olla 0,4–0,9
   * **NDRE**: Voimakkaan kasvun arvo on suurempi
   * **GNDVI**: Samanlainen kuin NDVI, mutta klorofylliherkkä
3. Tarkista muut kuin kasvillisuusalueet:
   * **Maaperä**: Lähellä 0:aa tai hieman negatiivinen
   * **Vesi**: Negatiiviset arvot (-0,5–0)***

## Katseluongelmien vianmääritys

### Kuva ei avaudu

**Mahdolliset syyt:**

* Tiedosto vioittunut käsittelyn aikana
* Tiedostomuotoa ei tueta
* Muistia ei riitä suurelle kuvalle

**Ratkaisut:**

1. Yritä avata tiedosto ulkoisessa katseluohjelmassa tiedoston eheyden varmistamiseksi
2. Tarkista, että tiedostomuoto vastaa odotettua tyyppiä
3. Sulje muut sovellukset muistin vapauttamiseksi
4. Kokeile pienempää/eri kuvaa

### Kuva näkyy mustana tai valkoisena

**Mahdolliset syyt:**

* Arvoalue ylittää näytön kapasiteetin
* 32-bittinen float-kuva, jossa on epätavallisia arvoja
* Indeksin laskentavirhe

**Ratkaisut:**

1. Tarkista pikseliarvot – jos ne ovat kaikki hyvin alhaisia tai hyvin korkeita, säädä näyttöaluetta
2. Yritä avata tiedosto QGIS:ssä tai vastaavassa ohjelmassa, jossa on automaattinen alueen säätö
3. Tarkista käsittelyn virheloki virheiden varalta

### Pikseliarvot näyttävät vääriltä

**Mahdolliset syyt:**

* Väärän kuvan katselu (alkuperäinen vs. käsitelty)
* Kalibrointia ei sovellettu oikein
* Valosensorin tietoja ei sisällytetty syötteeseen
* Prosenttitila on kytketty väärin

**Ratkaisut:**

1. Varmista, että katsot käsiteltyä tulosta (tarkista tiedostonimen pääte)
2. Tarkista prosenttitilan painikkeen tila
3. Vertaa samoista aineistoista peräisin oleviin, tiedossa oleviin hyviin kuviin

***

## Seuraavat vaiheet

Nyt kun voit katsella kuvia koko näytön tilassa:

* [**Kuvakerrokset**](image-layers.md) – Tutustu monikaistaisiin visualisointiin
* [**Indeksi/LUT-hiekkalaatikko**](index-lut-sandbox.md) – Käytä mukautettuja indeksejä ja värikartoitusta
* [**Monispektriset indeksikaavat**](../project-settings/multispectral-index-formulas.md) – Tutustu käytettävissä oleviin indekseihin

Käsittelytyönkulusta katso:

* [**Kuvien käsittely (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Kattava käsittelyopas
