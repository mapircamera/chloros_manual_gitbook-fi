# Kuvakerrokset

Chloros-kuvankatseluohjelman Kuvakerrokset-pudotusvalikosta voit nopeasti vaihtaa saman kuvan eri versioiden välillä – alkuperäisistä kuvista käsiteltyihin heijastavuuslähtöihin ja laskettuihin indeksikuviin.

## Mitä ovat kuvakerrokset?

Chloros-ohjelmassa **kerrokset** viittaavat eri kuvatulostuksiin, jotka ovat käytettävissä yhdelle lähdekuvalle. Kun käsittelet kuvia, Chloros luo useita versioita:

* **Alkuperäiset kuvat** (kameran JPG- ja RAW-tiedostot)
* **Heijastavuuskalibroidut** tulosteet (jos heijastavuuskalibrointi oli käytössä)
* **Kohdekuvat** (jos kuva sisältää kalibrointikohteita)
* **Indeksikuvat** (NDVI, NDRE, GNDVI jne., jos indeksit oli määritetty)

Kuvankatseluohjelman oikeassa yläkulmassa olevan **Layer Selector -pudotusvalikon** avulla voit vaihtaa näiden versioiden välillä välittömästi poistumatta katseluohjelmasta.

***

## Käytettävissä olevat kerrostyypit

### JPG

* Kameran alkuperäinen JPG-esikatselukuva
* Aina käytettävissä kaikille kuville
* Käsittelemätön, kameran ottama
* Nopein ladata ja näyttää

**Käyttötarkoitus:**

* Nopea esikatselu alkuperäisestä kuvasta
* Kuvan sommittelun ja rajauksen tarkistaminen
* Kuvan laadun tarkistaminen ennen käsittelyä

### RAW (alkuperäinen)

* Kameran alkuperäinen RAW-anturidata
* Debayeroitu ilman jälkikäsittelyä
* Suurempi bittisyvyys kuin JPG (tyypillisesti 12- tai 14-bittinen anturidata)

**Milloin katsella:**

* Alkuperäisen anturidatan laadun tarkastaminen
* Anturin ongelmien tai artefaktien tarkistaminen
* Käsittelyn ennen ja jälkeen -tulosten vertailu

### RAW (kohde)

* Näkyy vain kuvissa, joissa on kalibrointikohteita
* Näyttää alkuperäisen RAW-kuvan, jossa kohde on havaittu
* Käytetään kohteen havaitsemisen onnistumisen tarkistamiseen

**Milloin tarkastella:**

* Kalibrointikohteiden oikean havaitsemisen vahvistaminen
* Kohteen kuvanlaadun tarkistaminen
* Kalibrointiongelmien vianmääritys

{% hint style=&quot;info&quot; %}
**Kohdekerros**: Tämä kerros näkyy vain kalibrointikohteita sisältävien kuvien pudotusvalikossa. Tavallisissa kuvissa tätä vaihtoehtoa ei ole.
{% endhint %}

### RAW (heijastavuus)

* Kalibroitu heijastavuuslähtökuva
* Vignette-korjattu (jos käytössä käsittelyssä)
* Heijastavuus kalibroitu kohdetietojen avulla (jos käytössä)
* Monikaistainen TIFF kaikilla kamerakanavilla
* Pikseliarvot edustavat heijastavuuden prosenttiosuutta (kun käytetään prosenttimuotoa)
* Valmis muokattavaksi [Index/LUT Sandbox] -työkalulla(index-lut-sandbox.md)

**Katsomisen ajankohta:**

* Kalibroitujen tulosten tarkastelu
* Kalibroinnin laadun tarkistaminen
* Pikseliarvojen tarkistaminen tieteellisen tarkkuuden varmistamiseksi
* Vertailu alkuperäiseen kalibroinnin vaikutusten tarkastelemiseksi

{% hint style=&quot;success&quot; %}
**Suositeltava**: Käytä RAW (Reflectance) -kerrosta, kun tarkistat pikseliarvoja tieteellisiä mittauksia ja analyysejä varten.
{% endhint %}

### RAW (NDVI Index)... ja vastaavat

* Laskettu kasvillisuusindeksikuva (tässä esimerkissä NDVI)
* Indeksin nimi muuttuu sen mukaan, mikä indeksi on määritetty käsittelyn aikana.
* Esimerkkejä: RAW (NDVI-indeksi), RAW (NDRE-indeksi), RAW (GNDVI-indeksi) jne.
* Yksikaistainen harmaasävykuva, joka näyttää indeksin laskentatulokset
* Jokaiselle projektin asetuksissa määritetylle indeksille näkyy yksi kerros

**Mahdolliset indeksien nimet:**

* RAW (NDVI-indeksi)
* RAW (NDRE-indeksi)
* RAW (GNDVI-indeksi)
* RAW (OSAVI-indeksi)
* RAW (EVI-indeksi)
* RAW (SAVI-indeksi)
* Ja paljon muuta... (katso [Monispektrinen indeksi kaavat](../project-settings/multispectral-index-formulas.md))

**Milloin tarkastella:**

* Indeksin laskentatulosten tarkastelu
* Indeksin arvojen vaihteluvälien tarkistaminen
* Kiinnostavien alueiden tunnistaminen
* Indeksikuvien tarkistaminen ennen käyttöä GIS:ssä tai analyysissä

***

## Kerroksen valitsimen käyttö

### Pudotusvalikon avaaminen

1. Avaa kuva koko näytön tilassa (napsauta mitä tahansa pikkukuvaa kuvankatselijassa)
2. Etsi **kerrosten pudotusvalikko** katselijan oikeasta yläkulmasta
3. Pudotusvalikossa näkyy valittuna oleva kerros (esim. &quot;JPG&quot;)
4. Napsauta pudotusvalikkoa nähdäksesi kaikki käytettävissä olevat kerrokset

### Kerrosten vaihtaminen

1. Napsauta kerrosten avattavaa valikkoa avataksesi luettelon.
2. Kaikki nykyisen kuvan käytettävissä olevat kerrokset näytetään.
3. Napsauta mitä tahansa kerroksen nimeä vaihtaaksesi kyseiseen versioon.
4. Kuva päivittyy välittömästi näyttämään valitun kerroksen.

**Nopea vaihtaminen:**

* Avattava valikko muistaa viimeisimmän valintasi.
* Kun siirryt seuraavaan kuvaan, Chloros yrittää näyttää saman kerroksen tyypin.
* Jos kyseistä kerrosta ei ole seuraavassa kuvassa, oletusarvona on JPG.

### Kerrosten saatavuus

Kaikki kerrokset eivät ole käytettävissä jokaisessa kuvassa:

**Aina käytettävissä:**

* ✅ JPG (jokaisella kuvalla on JPG-esikatselu)

**Ehdollisesti käytettävissä:**

* ⚠️ RAW (alkuperäinen) – Vain jos kuva on otettu RAW- tai RAW+JPG-tilassa
* ⚠️ RAW (kohde) – Vain jos kuva sisältää havaittuja kalibrointikohteita
* ⚠️ RAW (heijastavuus) – Vain käsittelyn jälkeen, kun heijastavuuden kalibrointi on käytössä
* ⚠️ RAW (\[indeksi] indeksi) – Vain käsittelyn jälkeen, kun indeksit on määritetty

***

## Kerrosten pysyvyys

### Kuvien välillä liikkuminen

Kun siirryt toiseen kuvaan (nuolinäppäimillä tai napsauttamalla pikkukuvia):

**Kerroksen asetukset säilyvät:**

* Jos katselet &quot;RAW (heijastavuus)&quot;, seuraava kuva on &quot;RAW (heijastavuus)&quot; (jos saatavilla)
* Jos katselet &quot;RAW (NDVI-indeksi)&quot;, seuraava kuva on &quot;RAW (NDVI Index)&quot; (jos saatavilla)
* Jos samaa kerrosta ei ole, oletusarvo on JPG

**Esimerkki työnkulusta:**

1. Avaa kuva 1, vaihda RAW (NDVI Index)
2. Paina → nähdäksesi kuvan 2
3. Kuva 2 näyttää automaattisesti RAW (NDVI-indeksi) -kerroksen
4. Jatka selaamista – kaikki kuvat näyttävät NDVI-kerroksen
5. Erittäin tehokas tapa tarkastella indeksin tuloksia useissa kuvissa

***

## Yleiset työnkulut

### Työnkulku 1: Ennen/jälkeen -vertailu

**Tavoite**: Vertaa alkuperäistä kuvaa kalibroituun kuvaan

1. Avaa käsitelty kuva kuvankatseluohjelmassa
2. Valitse **RAW (Alkuperäinen)** avattavasta valikosta
3. Huomaa vinjetointi ja kalibroimattomat arvot
4. Vaihda **RAW (Heijastavuus)** avattavasta valikosta
5. Vertaa – vinjetointi on poistettu, arvot on kalibroitu

### Työnkulku 2: Indeksin tarkastelu

**Tavoite**: Tarkista nopeasti NDVI-tulokset koko tietojoukosta.

1. Avaa ensimmäinen käsitelty kuva.
2. Valitse avattavasta valikosta **RAW (NDVI-indeksi)**.
3. Siirry seuraavaan kuvaan →-nuolinäppäimellä
4. NDVI-kerros säilyy automaattisesti
5. Jatka kaikkien kuvien läpi ja tarkista NDVI-kuviot
6. Vaihda **RAW (NDRE-indeksi)** -tilaan vertailua varten

### Työnkulku 3: Kohteen tarkistus

**Tavoite**: Tarkista, että kaikki kohdekuvat on tunnistettu oikein.

1. Siirry kohdekuvaan.
2. Valitse **RAW (kohde)** avattavasta valikosta.
3. Tarkista, että kalibrointikohteet ovat selvästi näkyvissä ja tunnistettu.
4. Siirry seuraavaan kohdekuvaan.
5. Toista tarkistus kaikille kohteille.

### Työnkulku 4: Pikseliarvon tarkastus

**Tavoite**: Tarkista heijastavuusarvojen tieteellinen tarkkuus.

1. Avaa käsitelty kuva.
2. Valitse **RAW (Heijastavuus)** -kerros.
3. Ota käyttöön **Pikseliprosentti** -tila (painike oikeassa yläkulmassa olevassa työkalurivissä).
4. Siirrä kursori kasvillisuusalueiden päälle.
5. Varmista, että pikseliarvot ovat odotetulla alueella (30–70 % NIR:lle, 5–15 % Red:lle).
6. Tarkista, että maaperän ja veden alueiden arvot ovat asianmukaiset.

***

## Pikseliarvojen ymmärtäminen kerroksittain

Eri kerrokset näyttävät erilaisia pikseliarvoalueita:

### JPG-kerros

* **Alue**: 0–255 (8-bittinen)
* **Merkitys**: Näytetään arvot, gamma-korjattu
* **Käyttö**: Vain silmämääräinen tarkastelu, ei tieteellisiin mittauksiin

### RAW (alkuperäinen)

* **Alue**: 0–65535 (16-bittinen)
* **Merkitys**: Raakadigitaaliset anturiluvut
* **Käyttö**: Anturin suorituskyvyn tarkistaminen, ei kalibroitu

### RAW (heijastavuus)

* **Alue**: 0–65 535 (16-bittinen TIFF) tai 0,0–1,0 (32-bittinen prosentti)
* **Merkitys**: Kalibroitu heijastavuusprosentti
* **Käyttö**: Tieteelliset mittaukset ja analyysit

**16-bittinen TIFF:** Jaa 65 535:llä saadaksesi heijastavuusprosentin **32-bittinen prosentti:** Arvot edustavat suoraan prosenttia (0,5 = 50 % heijastavuus)

### RAW (indeksikuvat)

* **Alue**: Vaihtelee indeksin mukaan (tyypillisesti -1,0–+1,0 normalisoiduille indekseille)
* **Merkitys**: Indeksin laskentatulokset
* **Esimerkkejä**:
  * NDVI: -1–+1 (kasvillisuus tyypillisesti 0,4–0,9)
  * NDRE: -1–+1 (stressin havaitseminen)
  * EVI: 0–1 (parannettu kasvillisuus)

***

## Vinkkejä ja parhaita käytäntöjä

### Tehokas kerrosten vaihtaminen

* **Pikanäppäinten tuntemus**: Vaikka kerroksille ei ole pikanäppäimiä, navigointinuolet (←/→) toimivat kaikissa kerroksissa
* **Johdonmukaiset työnkulut**: Valitse yksi kerros (esim. NDVI) ja tarkista koko tietojoukko ennen siirtymistä toiseen
* **Nopeat vertailut**: Vaihda alkuperäisen ja heijastavuuden välillä prosessoinnin laadun tarkistamiseksi

### Suorituskykyyn liittyvät seikat

* **JPG latautuu nopeimmin**: Käytä nopeaan selaamiseen useiden kuvien välillä.
* **RAW-kerrokset latautuvat hitaammin**: Korkeampi resoluutio ja bittisyvyys.
* **Indeksi-kerrokset**: Samanlainen nopeus kuin heijastavuuskerrokset.
* **Ensimmäinen lataus on hitain**: Saman kerroksen myöhemmät näkymät tallennetaan välimuistiin ja ovat nopeampia.

### Laadun tarkistus

* **Tarkista aina RAW (alkuperäinen)**: Tarkista lähdetietojen laatu ennen kuin luotat käsiteltyihin tuloksiin
* **Vertaa kerroksia**: Käytä kerrosten vaihtoa varmistaaksesi, että käsittely on toiminut oikein
* **Tarkista indeksin alueet**: Käytä pikseliprosenttitilaa indeksisluokkien kanssa varmistaaksesi, että arvot ovat kohtuullisia

***

## Vianmääritys

### Kerros ei ole käytettävissä

**Ongelma**: Odotettu kerros ei näy pudotusvalikossa

**Mahdolliset syyt:**

* Kuvaa ei ole käsitelty (vain JPG ja RAW (alkuperäinen) käytettävissä)
* Heijastavuuden kalibrointi oli pois käytöstä käsittelyn aikana
* Tiettyä indeksiä ei ole määritetty projektin asetuksissa
* Kuva on vain kohdekuva (kohteille ei ole luotu indeksejä)

**Ratkaisut:**

1. Varmista, että kuva on käsitelty (tarkista tulostuskansiosta, onko käsiteltyjä tiedostoja)
2. Tarkista projektin asetukset ja varmista, että indeksit on määritetty
3. Käsittele kuva uudelleen haluttujen indeksien ollessa käytössä

### Väärä kerros näkyy

**Ongelma**: Kuva avautuu odottamattomassa kerroksessa

**Syy**: Edellisen kuvan kerroksen asetukset on siirretty, mutta kyseistä kerrosta ei ole nykyisessä kuvassa

**Ratkaisu**: Chloros siirtyy automaattisesti JPG-muotoon, kun haluttu kerros ei ole käytettävissä – tämä on normaalia

### Kalibrointikohteita ei näy

**Ongelma**: RAW (kohde) -kerros ei näytä kohteen tunnistusta.

**Mahdolliset syyt:**

* Kohteita ei tunnistettu käsittelyn aikana.
* Kuva ei sisällä kohteita.
* Kohteen tunnistuksen asetukset ovat liian tiukat.

**Ratkaisut:**

1. Tarkista virheenkorjauslokista, onko siinä viestejä &quot;Kohde löydetty&quot;
2. Varmista, että kuva sisältää näkyviä kalibrointikohteita
3. Säädä kohteen tunnistuksen asetuksia projektin asetuksissa
4. Katso [Kohdekuvien valinta](../processing-images-gui/choosing-target-images.md)

***

## Liittyvät ominaisuudet

### Kuvan katseluohjelman työkalut

Kun katselet mitä tahansa kerrosta, voit käyttää:

* **Zoomauspainikkeita**: Suurenna tarkastellaksesi yksityiskohtia.
* **Panoraamaa**: Napsauta ja vedä siirtääksesi zoomattua kuvaa.
* **Pikseliarvon tarkastelua**: Katso arvot kursorin sijainnissa.
* **Navigointinuolia**: Siirry kuvien välillä säilyttäen kerroksen.
* **Pikseliprosenttitilaa**: Vaihda DN- ja prosenttimäärän näyttämisen välillä.

Katso [Kuvan avaaminen koko ruudulla](opening-an-image-full-screen.md) saadaksesi täydellisen kuvankatseluohjelman dokumentaation.

### Indeksi/LUT-hiekkalaatikko

Interaktiiviseen indeksin testaukseen ja visualisointiin:

* **Reaaliaikainen indeksin laskeminen**: Testaa erilaisia indeksin kaavoja
* **LUT-värikartoitus**: Käytä värigradientteja harmaasävyindekseihin
* **Vie visualisoinnit**: Tallenna värilliset indeksikuvat

Katso [Indeksi/LUT-hiekkalaatikko](index-lut-sandbox.md) lisätietoja varten.

***

## Seuraavat vaiheet

Nyt kun ymmärrät kuvakerrokset:

* [**Kuvan avaaminen koko näytöllä**](opening-an-image-full-screen.md) - Täydellinen Image Viewer -opas
* [**Indeksi/LUT-hiekkalaatikko**](index-lut-sandbox.md) – Interaktiivinen indeksin visualisointi
* [**Monispektriset indeksikaavat**](../project-settings/multispectral-index-formulas.md) – Käytettävissä olevat indeksit
* [**Käsittelyn viimeistely**](../processing-images-gui/finishing-the-processing.md) – Käsiteltyjen tulosten ymmärtäminen
