# Kuvakerrokset

Chloros-kuvankatseluohjelman Kuvakerrokset-pudotusvalikosta voit vaihtaa nopeasti saman kuvan eri versioiden välillä – alkuperäisistä kuvista käsiteltyihin heijastavuustuloksiin ja laskettuihin indeksikuviksi.

## Mitä kuvakerrokset ovat?

Chloros-ohjelmassa **kerrokset** viittaavat eri kuvatuloksiin, jotka ovat saatavilla yhdestä lähdekuvasta. Kun käsittelet kuvia, Chloros luo useita versioita:

* **Alkuperäiset kuvat** (kameran JPG- ja RAW-tiedostot)
* **Heijastavuuskalibroidut** tulosteet (jos heijastavuuskalibrointi oli käytössä)
* **Kohdekuvat** (jos kuva sisältää kalibrointikohteita)
* **Indeksikuvat** (NDVI, NDRE, GNDVI jne., jos indeksit on määritetty)

Kuvankatseluohjelman oikeassa yläkulmassa olevan **Layer Selector -pudotusvalikon** avulla voit vaihtaa näiden versioiden välillä välittömästi poistumatta katseluohjelmasta.***

## Käytettävissä olevat kerrostyypit

### JPG

* Kameran alkuperäinen JPG-esikatselukuva
* Aina käytettävissä kaikille kuville
* Käsittelemätön, sellaisena kuin kamera on sen tallentanut
* Nopein ladata ja näyttää

**Milloin katsella:**

* Alkuperäisen kuvan nopea esikatselu
* Kuvan sommittelun ja rajauksen tarkistaminen
* Kuvan laadun tarkistaminen ennen käsittelyä

### RAW (Alkuperäinen)

* Kameran alkuperäiset RAW-anturitiedot
* Debayeroitu ilman jälkikäsittelyä
* Suurempi bittisyvyys kuin JPG:llä (tyypillisesti 12- tai 14-bittiset anturitiedot)

**Milloin katsella:**

* Alkuperäisen anturidatan laadun tarkastelu
* Anturiongelmien tai artefaktien tarkistaminen
* Käsittelyn ennen ja jälkeen -tulosten vertailu

### RAW (Kohde)

* Näkyy vain kuvissa, joiden on tunnistettu sisältävän kalibrointikohteita
* Näyttää alkuperäisen RAW-kuvan, josta kohde on tunnistettu
* Käytetään varmistamaan, että kohteen tunnistus onnistui

**Milloin katsella:**

* Kalibrointikohteiden oikean tunnistuksen vahvistaminen
* Kohteen kuvan laadun tarkistaminen
* Kalibrointiongelmien vianmääritys

{% hint style="info" %}
**Kohdekerros**: Tämä kerros näkyy pudotusvalikossa vain kuvissa, jotka sisältävät kalibrointikohteita. Tavallisissa kuvissa tätä vaihtoehtoa ei ole.
{% endhint %}

### RAW (heijastavuus)

* Kalibroitu heijastavuuskuva
* Vignetoinnin korjaus (jos käytössä käsittelyssä)
* Heijastavuus kalibroitu kohdetietojen avulla (jos käytössä)
* Monikaistainen TIFF kaikilla kamerakanavilla
* Pikseliarvot edustavat heijastavuuden prosenttiosuutta (kun käytetään prosenttitilaa)
* Valmis muokattavaksi [Index/LUT Sandbox](index-lut-sandbox.md) -työkalulla

**Milloin tarkastella:**

* Kalibroitujen tulosten tarkastelu
* Kalibroinnin laadun varmistaminen
* Pikseliarvojen tieteellisen tarkkuuden tarkistaminen
* Vertailu alkuperäiseen kuvaan kalibroinnin vaikutusten tarkastelemiseksi

{% hint style="success" %}
**Suositeltava**: Käytä RAW (heijastavuus) -kerrosta, kun tarkistat pikseliarvoja tieteellisiä mittauksia ja analyyseja varten.
{% endhint %}

### RAW (NDVI-indeksi)... ja vastaavat

* Laskettu kasvillisuusindeksikuva (tässä esimerkissä NDVI)
* Indeksin nimi muuttuu sen mukaan, mikä indeksi on määritetty käsittelyn aikana
* Esimerkkejä: RAW (NDVI-indeksi), RAW (NDRE-indeksi), RAW (GNDVI-indeksi) jne.
* Yksikaistainen harmaasävykuva, joka näyttää indeksin laskentatulokset
* Jokaisesta Projektin asetuksissa määritetystä indeksistä näkyy yksi kerros

**Mahdolliset indeksinimet:**

* RAW (NDVI-indeksi)
* RAW (NDRE-indeksi)
* RAW (GNDVI-indeksi)
* RAW (OSAVI-indeksi)
* RAW (EVI-indeksi)
* RAW (SAVI-indeksi)
* Ja monia muita... (katso [Monispektriset indeksikaavat](../project-settings/multispectral-index-formulas.md))

**Milloin tarkastella:**

* Indeksin laskentatulosten tarkastelu
* Indeksin arvoalueiden tarkistaminen
* Kiinnostavien alueiden tunnistaminen
* Indeksikuvien tarkistaminen ennen käyttöä GIS-järjestelmässä tai analyysissä

***

## Kerroksen valitsimen käyttö

### Pudotusvalikon avaaminen

1. Avaa kuva koko näytön tilassa (napsauta mitä tahansa pikkukuvaa kuvankatselijassa)
2. Etsi **kerrosvalikko** katselijan oikeasta yläkulmasta
3. Valikossa näkyy tällä hetkellä valittu kerros (esim. &quot;JPG&quot;)
4. Napsauta valikkoa nähdäksesi kaikki käytettävissä olevat kerrokset

### Kerrosten vaihtaminen

1. Napsauta kerrosvalikkoa avataksesi luettelon
2. Näkyviin tulevat kaikki nykyisen kuvan käytettävissä olevat kerrokset
3. Napsauta mitä tahansa kerroksen nimeä vaihtaaksesi kyseiseen versioon
4. Kuva päivittyy välittömästi näyttämään valitun kerroksen

**Nopea vaihtaminen:**

* Pudotusvalikko muistaa viimeisimmän valintasi
* Kun siirryt seuraavaan kuvaan, Chloros yrittää näyttää saman kerroksen tyypin
* Jos kyseistä kerrosta ei ole seuraavassa kuvassa, oletusarvona on JPG

### Kerrosten saatavuus

Kaikki kerrokset eivät ole käytettävissä jokaisessa kuvassa:

**Aina käytettävissä:*** ✅ JPG (jokaisella kuvalla on JPG-esikatselu)

**Ehdollisesti käytettävissä:**

* ⚠️ RAW (Alkuperäinen) – Vain jos kuva on otettu RAW- tai RAW+JPG-tilassa
* ⚠️ RAW (Kohde) – Vain jos kuva sisältää tunnistettuja kalibrointikohteita
* ⚠️ RAW (heijastavuus) – Vain, jos käsittely on suoritettu heijastavuuskalibrointi käytössä
* ⚠️ RAW (\[Indeksi] Indeksi) – Vain, jos käsittely on suoritettu indeksit määritettynä

***

## Kerrosten pysyvyys

### Kuvien välillä liikkuminen

Kun siirryt toiseen kuvaan (nuolinäppäimillä tai napsauttamalla pikkukuvia):**Kerroksen asetukset säilyvät:**

* Jos katsot &quot;RAW (heijastavuus)&quot;, seuraava kuva näyttää &quot;RAW (heijastavuus)&quot; (jos saatavilla)
* Jos katsot &quot;RAW (NDVI-indeksi)&quot;, seuraava kuva näyttää &quot;RAW (NDVI-indeksi)&quot; (jos saatavilla)
* Jos samaa kerrosta ei ole, oletuksena on JPG

**Esimerkki työnkulusta:**

1. Avaa kuva 1, vaihda RAW (NDVI Index) -tilaan
2. Paina → nähdäksesi kuvan 2
3. Kuva 2 näyttää automaattisesti RAW (NDVI Index) -kerroksen
4. Jatka selaamista – kaikki kuvat näyttävät NDVI-kerroksen
5. Erittäin tehokas tapa tarkastella indeksituloksia useissa kuvissa

***

## Yleisiä työnkulkuja

### Työnkulku 1: Ennen/jälkeen -vertailu

**Tavoite**: Vertaa alkuperäistä ja kalibroitua kuvaa

1. Avaa käsitelty kuva kuvankatseluohjelmassa
2. Valitse pudotusvalikosta **RAW (Original)**

3. Huomaa vinjetointi ja kalibroimattomat arvot
4. Vaihda pudotusvalikosta **RAW (Reflectance)**

5. Vertaa – vinjetointi poistettu, arvot kalibroitu

### Työnkulku 2: Indeksin tarkastelu

**Tavoite**: Tarkastele nopeasti NDVI-tuloksia koko aineistossa

1. Avaa ensimmäinen käsitelty kuva
2. Valitse pudotusvalikosta **RAW (NDVI-indeksi)**

3. Siirry seuraavaan kuvaan nuolinäppäimillä
4. NDVI-kerros säilyy automaattisesti
5. Jatka kaikkien kuvien läpi ja tarkista NDVI-kuviot
6. Vaihda **RAW (NDRE Index)** -tilaan vertailua varten

### Työnkulku 3: Kohteen tarkistus

**Tavoite**: Varmista, että kaikki kohdekuvat on tunnistettu oikein

1. Siirry kohdekuvaan
2. Valitse pudotusvalikosta **RAW (Target)**

3. Varmista, että kalibrointikohteet ovat selvästi näkyvissä ja tunnistettuja
4. Siirry seuraavaan kohdekuvaan
5. Toista tarkistus kaikille kohteille

### Työnkulku 4: Pikseliarvojen tarkastus

**Tavoite**: Tarkista heijastusarvojen tieteellinen tarkkuus

1. Avaa käsitelty kuva
2. Valitse **RAW (Heijastus)**-kerros
3. Ota käyttöön **Pikseliprosentti**-tila (painike oikean yläkulman työkalurivillä)
4. Siirrä kursori kasvillisuusalueiden päälle
5. Varmista, että pikseliarvot ovat odotetuissa alueissa (30–70 % NIR:lle, 5–15 % Red:lle)
6. Tarkista maaperän ja vesialueiden arvot

***

## Pikseliarvojen ymmärtäminen kerroksittain

Eri kerroksilla on erilaiset pikseliarvoalueet:

### JPG-kerros

* **Alue**: 0–255 (8-bittinen)
* **Merkitys**: Näyttöarvot, gamma-korjattu
* **Käyttö**: Vain silmämääräinen tarkastelu, ei tieteellisiin mittauksiin

### RAW (alkuperäinen)

* **Alue**: 0–65535 (16-bittinen)
* **Merkitys**: Raakakuvasensorin digitaaliset luvut
* **Käyttö**: Sensorin suorituskyvyn tarkistamiseen, ei kalibroitu

### RAW (heijastavuus)

* **Alue**: 0–65 535 (16-bittinen TIFF) tai 0,0–1,0 (32-bittinen prosentti)
* **Merkitys**: Kalibroitu heijastusprosentti
* **Käyttö**: Tieteelliset mittaukset ja analyysit**16-bittiselle TIFF:lle:**Jaa 65 535:llä saadaksesi heijastusprosentin**32-bittiselle prosentille:** Arvot edustavat suoraan prosentteja (0,5 = 50 % heijastus)

### RAW (indeksikuvat)

* **Alue**: Vaihtelee indeksin mukaan (tyypillisesti -1,0 – +1,0 normalisoiduille indekseille)
* **Merkitys**: Indeksin laskentatulos
* **Esimerkkejä**:
  * NDVI: -1 – +1 (kasvillisuus tyypillisesti 0,4 – 0,9)
  * NDRE: -1 – +1 (stressin havaitseminen)
  * EVI: 0 – 1 (parannettu kasvillisuus)

***

## Vinkkejä ja parhaita käytäntöjä

### Tehokas kerrosten vaihtaminen

* **Pikanäppäinten käyttö**: Vaikka tasoille ei ole pikanäppäimiä, navigointinuolet (←/→) toimivat kaikilla tasoilla
* **Johdonmukaiset työnkulut**: Valitse yksi taso (esim. NDVI) ja tarkista koko aineisto ennen siirtymistä toiseen
* **Nopeat vertailut**: Vaihda Original- ja Reflectance-tasojen välillä käsittelyn laadun tarkistamiseksi

### Suorituskykyyn liittyvät seikat

* **JPG latautuu nopeimmin**: Käytä nopeaan selaamiseen useiden kuvien välillä
* **RAW-kerrokset latautuvat hitaammin**: Korkeampi resoluutio ja bittisyvyys
* **Indeksikerrokset**: Samanlainen nopeus kuin heijastuskerroksilla
* **Ensimmäinen lataus on hitain**: Saman kerroksen seuraavat näkymät tallennetaan välimuistiin ja ovat nopeampia

### Laadun tarkistaminen

* **Tarkista aina RAW (Alkuperäinen)**: Varmista lähdetietojen laatu ennen kuin luotat käsiteltyihin tuloksiin
* **Vertaa kerroksia**: Käytä kerrosten vaihtoa varmistaaksesi, että käsittely on toiminut oikein
* **Tarkista indeksialueet**: Käytä Pikseliprosentti-tilaa indeksikerrosten kanssa varmistaaksesi, että arvot ovat järkeviä***

## Vianmääritys

### Kerros ei ole käytettävissä

**Ongelma**: Odotettua kerrosta ei näy pudotusvalikossa**Mahdolliset syyt:**

* Kuvaa ei ole käsitelty (vain JPG ja RAW (alkuperäinen) käytettävissä)
* Heijastavuuden kalibrointi oli pois käytöstä käsittelyn aikana
* Tiettyä indeksiä ei ole määritetty projektin asetuksissa
* Kuva on pelkästään kohdekuva (kohteille ei luoda indeksejä)

**Ratkaisut:**

1. Varmista, että kuva on käsitelty (tarkista tulostuskansiosta, onko käsiteltyjä tiedostoja)
2. Tarkista projektin asetuksista, että indeksit on määritetty
3. Käsittele uudelleen halutut indeksit käytössä

### Väärä kerros näkyvissä

**Ongelma**: Kuva avautuu odottamattomalla kerroksella**Syy**: Edellisen kuvan kerrosasetus on siirtynyt eteenpäin, mutta kyseistä kerrosta ei ole nykyisessä kuvassa**Ratkaisu**: Chloros siirtyy automaattisesti JPG-muotoon, kun ensisijaista kerrosta ei ole käytettävissä – tämä on normaalia käyttäytymistä

### Kalibrointikohteita ei näy

**Ongelma**: RAW (Kohde) -kerros ei näytä kohteen tunnistusta**Mahdolliset syyt:**

* Kohteita ei havaittu käsittelyn aikana
* Kuva ei sisällä kohteita
* Kohteen tunnistuksen asetukset ovat liian tiukat

**Ratkaisut:**

1. Tarkista virhelokista, onko siellä &quot;Kohde löydetty&quot; -viestejä
2. Varmista, että kuva sisältää näkyviä kalibrointikohteita
3. Säädä kohteen tunnistuksen asetuksia Projektin asetuksissa
4. Katso [Kohdekuvien valinta](../processing-images-gui/choosing-target-images.md)

***

## Liittyvät ominaisuudet

### Kuvankatselutyökalut

Kun katselet mitä tahansa tasoa, voit käyttää:

* **Zoomauspainikkeita**: Suurenna tarkastellaksesi yksityiskohtia
* **Panorointia**: Napsauta ja vedä liikkuaaksesi zoomattua kuvaa
* **Pikseliarvojen tarkastelua**: Katso arvot kursorin sijainnissa
* **Navigointinuolet**: Siirry kuvien välillä säilyttäen kerroksen
* **Pikseliprosenttitila**: Vaihda DN- ja prosenttinäytön välillä

Katso [Kuvan avaaminen koko ruudulle](opening-an-image-full-screen.md) saadaksesi täydellisen kuvankatseluohjelman dokumentaation.

### Indeksi/LUT-hiekkalaatikko

Interaktiiviseen indeksin testaamiseen ja visualisointiin:

* **Reaaliaikainen indeksin laskeminen**: Testaa erilaisia indeksin kaavoja
* **LUT-värikartoitus**: Sovella värigradientteja harmaasävyindekseihin
* **Visualisointien vienti**: Tallenna värilliset indeksikuvat

Katso [Indeksi/LUT-hiekkalaatikko](index-lut-sandbox.md) saadaksesi lisätietoja.

***

## Seuraavat vaiheet

Nyt kun ymmärrät kuvakerrokset:

* [**Kuvan avaaminen koko ruudulle**](opening-an-image-full-screen.md) – Täydellinen Image Viewer -opas
* [**Indeksi/LUT-hiekkalaatikko**](index-lut-sandbox.md) – Interaktiivinen indeksin visualisointi
* [**Monispektriset indeksikaavat**](../project-settings/multispectral-index-formulas.md) – Käytettävissä olevien indeksien viite
* [**Käsittelyn viimeistely**](../processing-images-gui/finishing-the-processing.md) – Käsiteltyjen tulosten ymmärtäminen
