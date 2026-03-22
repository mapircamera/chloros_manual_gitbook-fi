# Käsittelyn päättäminen

Kun Chloros on saanut käsittelyn päätökseen, on aika tarkistaa tulokset, varmistaa tuloslaatu ja valmistella käsitellyt kuvat käytettäväksi työnkulussasi. Tällä sivulla opastetaan sinut viimeisten vaiheiden ja seuraavien toimien läpi.

## Käsittelyn päättymisen merkki

Kun käsittely on onnistuneesti päättynyt, näet useita merkkejä:

* ✅ **Edistymispalkki**: Saavuttaa 100 %:n valmiusasteen
* ✅ **Vianmääritysloki**: Näyttää viestin &quot;Käsittely valmis&quot;
* ✅ **Käynnistyspainike**: Aktivoituu uudelleen (valmis seuraavaan käsittelykierrokseen)
* ✅ **Tulostustiedostot**: Kaikki käsitellyt kuvat on tallennettu kameramallin alikansioon***

## Käsiteltyjen kuvien paikantaminen

### Tulostuskansion avaaminen

1. Napsauta **Päävalikko** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> -kuvaketta (vasemmassa yläkulmassa)
2. Valitse **&quot;Avaa projektikansio&quot;**

3. Tiedostoselain avautuu projektikansioon
4. Etsi projekti nimen perusteella

***

## Käsiteltyjen kuvien tarkastelu

### Pikaesikatselu tiedostoselaimessa

**Windows:n sisäänrakennettu esikatselu:**

1. Siirry kameramallin alikansioon
2. Valitse kuvatiedosto
3. Esikatselu näkyy Windows Explorerin esikatselupaneelissa
4. Selaa kuvia nuolinäppäimillä

### Esikatselu ulkoisissa kuvankatseluohjelmissa

**Suositellut katseluohjelmat:*** **QGIS** – Ilmainen GIS-ohjelmisto (paras georeferoituun monispektrianalyysiin)
* **IrfanView** – Nopea, kevyt kuvankatseluohjelma (tukee TIFF)
* **Adobe Photoshop** – Ammattimainen kuvankäsittely (tukee TIFF)
* **GIMP** – Ilmainen vaihtoehto Photoshopille
* **Windows Photos** – Perustason katselu (ei välttämättä tue 16-bittistä TIFF)

### Esikatselu Chloros-kuvankatseluohjelmassa

Käytä Chloros:n sisäänrakennettua kuvankatseluohjelmaa edistyneeseen visualisointiin:

1. Napsauta kuvan pikkukuvaa tiedostoselaimessa
2. Kuva avautuu pääesikatselualueelle
3. Napsauta **Kuvankatseluohjelma** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> -välilehteä vasemmassa sivupalkissa
4. Käytä [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) interaktiiviseen analysointiin

Katso [Kuvankatseluohjelma](../image-viewer-gui/opening-an-image-full-screen.md) saadaksesi yksityiskohtaiset ohjeet.

***

## Virhelokin tarkastelu

### Tarkista varoitukset tai virheet

1. Avaa **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> -välilehden
2. Selaa viestejä
3. Etsi keltaisia varoituksia tai punaisia virheitä
4. Tarkista havaitut ongelmat
5. Ota yhteyttä MAPIR-tukeen saadaksesi apua

### Lokin tallentaminen

Käsittelyn kirjaamiseksi tai lähettämiseksi MAPIR-tukeen:

1. Napsauta **&quot;Kopioi&quot;**- tai**&quot;Lataa&quot;**-painiketta
2. Tallenna tekstitiedostona projektikansioon
3. Liitä projektidokumentaatioon
4. Lähetä MAPIR-tukeen, jos ongelmia ilmenee

***

## Yleisiä tulostusongelmia ja ratkaisuja

### Ongelma: Puuttuvat tulostustiedostot

**Mahdolliset syyt:**

* Tiedostot eivät täyttäneet käsittelykriteereitä
* Vain kohdekuvat (jätetty pois viennistä)
* Levytila loppui viennin aikana
* Tiedosto vioittui käsittelyn aikana

**Ratkaisut:**

1. Tarkista virhelokista ohitus-/virheilmoitukset
2. Varmista, että levytilaa oli riittävästi
3. Laske tiedostojen määrä: Sen pitäisi vastata (alkuperäinen määrä – kohdemäärä) × (indeksit + 1)
4. Tuo puuttuvat tiedostot uudelleen ja käsittele ne uudelleen

### Ongelma: Tummat tai kirkkaat reunat (vignetoituminen edelleen näkyvissä)

**Mahdolliset syyt:**

* Vignetoinnin korjaus pois käytöstä
* Kamera/objektiivi ei ole Chloros-profiilitietokannassa
* Äärimmäinen vignetoituminen, jota ei voida korjata

**Ratkaisut:**

1. Varmista, että vignetoinnin korjaus on käytössä Projektin asetuksissa
2. Tarkista, että kameramalli on tunnistettu oikein
3. Ota yhteyttä MAPIR-tukeen, jos vignetoituminen jatkuu

### Ongelma: Virheelliset värit tai arvot

**Mahdolliset syyt:**

* Kalibrointikohteita ei havaittu
* Valittu väärä kalibrointikohdemalli
* Heijastavuuskalibrointi pois käytöstä
* Huonolaatuiset kohdekuvat

**Ratkaisut:**

1. Varmista, että heijastavuuskalibrointi on käytössä
2. Tarkista &quot;Kohde löydetty&quot; -viestit vianmäärityslokista
3. Tarkista kohdekuvien laatu
4. Suorita käsittely uudelleen merkitsemällä oikeat kohteet

### Ongelma: NDVI-arvot näyttävät virheellisiltä

**Odotetut NDVI-arvoalueet:*** **Vesi, kivet, maaperä**: -0,1 – 0,2
* **Harva/epäterve kasvillisuus**: 0,2 – 0,4
* **Kohtalainen kasvillisuus**: 0,4 – 0,6
* **Terve, tiheä kasvillisuus**: 0,6 – 0,9**Jos arvot ovat näiden alueiden ulkopuolella:**

1. Varmista, että heijastavuuden kalibrointi on suoritettu
2. Varmista, että valosensoriloki on mukana
3. Tarkista, että kalibrointikohteet on tunnistettu
4. Varmista, että oikea kameramalli on tunnistettu
5. Tarkista kohdekuvien ottamisen ajankohta ja olosuhteet

***

## Käsiteltyjen kuvien käyttö

### Fotogrammetriaan / ortomosaiikin luomiseen

**Suositeltu työnkulku:**

1.**Tuo kalibroidut heijastavuuskuvat** fotogrammetriaohjelmistoon:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Säilytä EXIF-metatiedot**: Varmista, että GPS-tiedot säilyvät geotunnistamista varten
3. **Kalibroidut työnkulut**: Käytä heijastavuuskuvia tieteellisen tarkkuuden varmistamiseksi
4. **Käsittele indeksimosaiikit**: Luo NDVI-ortomosaiikit yksittäisistä indeksikuvista
5. **Vie georeferoidut GeoTIFF-tiedostot**: Käyttöä varten GIS-sovelluksissa

### GIS-analyysia varten

**Suositeltu työnkulku:**

1.**Lataa QGIS-, ArcGIS- tai vastaavaan ohjelmaan**

2.**Käytä 16-bittisiä TIFF**-heijastavuuskuvia monikaistanalyysiin
3. **Käytä indeksikuvia** (NDVI, NDRE) käyttövalmiina kasvillisuuskerroksina
4. **Rasterilaskuri**: Yhdistä kaistat mukautettua analyysia varten
5. **Vie**: Luo luokittelukarttoja, muutosten havaitsemista ja kasvillisuuden terveyskarttoja

### Suoraan analysointiin / raportointiin

**Suositeltu työnkulku:**

1.**Käytä indeksikuvia LUT-väreillä** visuaalisiin raportteihin
2. **Pura tilastotiedot**: NDVI-keskiarvo kenttä-/koeala-kohtaisesti
3. **Aikasarja**: Vertaa indeksejä useiden istuntojen välillä
4. **Luo raportteja**: Sisällytä kartat, tilastot ja visualisoinnit***

## Arkistointi ja varmuuskopiointi

### Suositeltu varmuuskopiointistrategia

**Mitä tallentaa:*** ✅ **Alkuperäiset RAW/JPG-kuvat** – Arkistoi erilliselle levylle/pilvipalveluun
* ✅ **Käsitellyt tulokset** – Säilytä kalibroidut kuvat ja indeksit
* ✅ **Projektitiedosto** – Sisältää kaikki asetukset uudelleenkäsittelyä varten, jos tarvitaan
* ✅ **Virheloki** – Dokumentoi käsittelyn yksityiskohdat
* ✅ **Kalibrointikohdekuvat** – Tarkistusta ja uudelleenkäsittelyä varten**Tallennussuositukset:*** **Välitön varmuuskopio**: Ulkoinen kiintolevy
* **Pitkäaikainen arkistointi**: Pilvitallennustila (Google Drive, Dropbox jne.)
* **Kriittiset tiedot**: Säilytä 2–3 kopiota eri paikoissa***

## Seuraavat käsittelykierrokset

### Projektiasetusten uudelleenkäyttö

Jos käsittelet vastaavia aineistoja tulevaisuudessa:

1. **Tallenna projektimalli** (jos et ole vielä tehnyt niin)
2. **Luo uusi projekti** käyttämällä tallennettua mallia
3. **Tuo uudet kuvat**

4.**Käsittele**samoilla asetuksilla yhdenmukaisuuden varmistamiseksi

### Useiden istuntojen eräkäsittely

Useille istunnoille/tietojoukoille:**Vaihtoehto 1: Käyttöliittymä – Useita projekteja**

* Luo erillinen projekti jokaiselle istunnolle
* Käytä yhdenmukaisia malliasetuksia
* Käsittele yksi kerrallaan

**Vaihtoehto 2: Chloros CLI (vain Chloros+)**

* Automatisoi eräkäsittely
* Käsittele useita kansioita skripteillä
* Katso [CLI-ohjeet](../CLI.md)

**Vaihtoehto 3: Python SDK (vain Chloros+)**

* Ohjelmoitu ohjaus
* Integrointi analyysiputkiin
* Katso [API-dokumentaatio](../api-python-sdk.md)

***

## Jälkikäsittelyn vianmääritys

### Uudelleenkäsittely eri asetuksilla

Jos tulokset eivät ole tyydyttäviä:

1. Säilytä alkuperäiset kuvat (älä koskaan poista niitä)
2. Avaa sama projekti Chloros:ssa
3. Säädä asetuksia Projektin asetukset -paneelissa
4. Käsittele uudelleen – tulokset korvaavat aiemmat tulokset

### Kuvien osajoukon käsittely

Vain tiettyjen kuvien uudelleenkäsittely:

1. Luo uusi projekti
2. Tuo vain uudelleenkäsittelyä vaativat kuvat
3. Käytä samaa asetusmallia
4. Käsittele pienempää tietojoukkoa

### Ohjeen saaminen

Jos kohtaat ongelmia:

* 📧 **Sähköposti**: info@mapir.camera (liitä mukaan vianmääritysloki)
* 🌐 **Tuki**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **UKK**: [Usein kysytyt kysymykset](../faq.md)
* 📖 **Ohjeet**: [Chloros-käyttöopas](../)***

## Yhteenveto: Koko työnkulku

Olet nyt suorittanut koko Chloros-käsittelytyönkulun:

1. ✅ **Luotu projekti** - Katso [Projektit](../projects.md)
2. ✅ **Lisätty tiedostoja** - Katso [Tiedostojen lisääminen](adding-files-to-a-project.md)
3. ✅ **Säätänyt asetuksia** – Katso [Projektin asetusten säätäminen](adjusting-project-settings.md)
4. ✅ **Merkannut kohteet** – Katso [Kohdekuvien valinta](choosing-target-images.md)
5. ✅ **Käsittely aloitettu** - Katso [Käsittelyn aloittaminen](starting-the-processing.md)
6. ✅ **Edistymistä seurattu** - Katso [Käsittelyn seuranta](monitoring-the-processing.md)
7. ✅ **Tarkistettu tulokset** - Tämä sivu**Kalibroidut, heijastavuuskorjatut monispektrikuvasi ovat valmiina analysoitavaksi!**

***

## Lisäresurssit

### Lisäominaisuudet

* [**Kuvankatseluohjelma**](../image-viewer-gui/opening-an-image-full-screen.md) - Interaktiivinen visualisointi ja analysointi
* [**Indeksi/LUT-hiekkalaatikko**](../image-viewer-gui/index-lut-sandbox.md) – Mukautettujen indeksien testaus
* [**Monispektriset indeksikaavat**](../project-settings/multispectral-index-formulas.md) – Täydellinen indeksiviite

### Automatisointi ja integrointi

* [**CLI-dokumentaatio**](../CLI.md) - Komentoriviltä suoritettava eräajokäsittely
* [**Python SDK**](../api-python-sdk.md) - Ohjelmoitu automaatio
* [**Chloros+ Ominaisuudet**](../#chloros) - Kehittyneet käsittelyominaisuudet

### Tuki ja oppiminen

* [**UKK**](../faq.md) - Vastauksia yleisiin kysymyksiin
* [**Kalibrointikohteet**](../calibration-targets.md) - Heijastavuuskalibroinnin ymmärtäminen
* [**Tuetut kamerat**](../supported-cameras.md) - Yhteensopiva laitteisto
