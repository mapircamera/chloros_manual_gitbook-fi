# Käsittelyn loppuun saattaminen

Kun Chloros on suorittanut käsittelyn, on aika tarkistaa tulokset, varmistaa tulostuksen laatu ja valmistella käsitellyt kuvat käytettäväksi työnkulussasi. Tällä sivulla opastetaan sinua viimeisten vaiheiden ja seuraavien toimien läpi.

## Käsittelyn valmistumisen merkki

Kun käsittely on suoritettu onnistuneesti, näet useita merkkejä:

* ✅ **Edistymispalkki**: Saavuttaa 100 %:n valmiuden
* ✅ **Debug-loki**: Näyttää viestin &quot;Käsittely valmis&quot;
* ✅ **Käynnistyspainike**: Aktivoituu uudelleen (valmis seuraavaan käsittelyyn)
* ✅ **Tulostustiedostot**: Kaikki käsitellyt kuvat tallennetaan kameramallin alikansioon***

## Käsiteltyjen kuvien paikantaminen

### Tulostuskansion avaaminen

1. Napsauta **Päävalikko** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (vasemmassa yläkulmassa)
2. Valitse **&quot;Avaa projektikansio&quot;**

3. Tiedostoselain avautuu projektikansioon
4. Etsi projekti nimen perusteella

***

## Käsiteltyjen kuvien tarkastelu

### Nopea esikatselu tiedostoselaimessa

**Windows:n sisäänrakennettu esikatselu:**

1. Siirry kameramallin alikansioon
2. Valitse kuvatiedosto
3. Esikatselu näkyy Windows Explorerin esikatselupaneelissa
4. Selaa kuvia nuolinäppäimillä

### Esikatselu ulkoisissa kuvankatseluohjelmissa

**Suositeltavat katseluohjelmat:*** **QGIS** – ilmainen GIS-ohjelmisto (paras georeferenssoituun monispektriseen analyysiin)
* **IrfanView** – nopea, kevyt kuvankatseluohjelma (tukee TIFF:ää)
* **Adobe Photoshop** - Ammattimainen kuvankäsittely (tukee TIFF)
* **GIMP** - Ilmainen vaihtoehto Photoshopille
* **Windows Photos** - Perustason katselu (ei välttämättä tue 16-bittistä TIFF)

### Esikatselu Chloros-kuvankatseluohjelmassa

Käytä Chloros:n sisäänrakennettua kuvankatseluohjelmaa edistyneeseen visualisointiin:

1. Napsauta kuvan pikkukuvaa tiedostoselaimessa
2. Kuva avautuu pääesikatselualueelle
3. Napsauta **Kuvankatseluohjelma** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> -välilehteä vasemmassa sivupalkissa.
4. Käytä [Index/LUT Sandbox] (../image-viewer-gui/index-lut-sandbox.md) interaktiiviseen analyysiin.

Katso [Kuvankatseluohjelma] (../image-viewer-gui/opening-an-image-full-screen.md) saadaksesi yksityiskohtaiset ohjeet.

***

## Debug-lokin tarkistaminen

### Tarkista varoitukset tai virheet

1. Avaa **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> -välilehti
2. Selaa viestejä
3. Etsi keltaisia varoituksia tai punaisia virheitä
4. Tarkista kaikki merkityt ongelmat
5. Ota yhteyttä MAPIR-tukeen saadaksesi apua

### Lokin tallentaminen

Jotta voit säilyttää käsittelyn tiedot tai lähettää ne MAPIR-tukeen:

1. Napsauta **&quot;Kopioi&quot;**- tai**&quot;Lataa&quot;**-painiketta
2. Tallenna tekstitiedostona projektikansioon
3. Liitä projektin dokumentaatioon
4. Lähetä MAPIR-tukeen, jos ongelmia ilmenee

***

## Yleisiä tulostusongelmia ja ratkaisuja

### Ongelma: Puuttuvat tulostustiedostot

**Mahdolliset syyt:**

* Tiedostot eivät täyttäneet käsittelykriteereitä
* Vain kohdekuvat (poissuljettu vientistä)
* Levytila loppui viennin aikana
* Tiedostojen vioittuminen käsittelyn aikana

**Ratkaisut:**

1. Tarkista vianmääritysloogista ohitus-/virheilmoitukset
2. Varmista, että levytilaa oli riittävästi
3. Laske tiedostot: Pitäisi vastata (alkuperäinen lukumäärä - kohdelukumäärä) × (indeksit + 1)
4. Tuo puuttuvat tiedostot uudelleen ja käsittele ne uudelleen.

### Ongelma: Tummat tai kirkkaat reunat (vignettia näkyy edelleen)

**Mahdolliset syyt:**

* Vignettikorjaus on poistettu käytöstä.
* Kameraa/objektiivia ei ole Chloros-profiilitietokannassa.
* Korjauskyvyn ylittävä äärimmäinen vignettia.

**Ratkaisut:**

1. Varmista, että vinjetoinnin korjaus on käytössä projektin asetuksissa.
2. Tarkista, että kameramalli on tunnistettu oikein.
3. Ota yhteyttä MAPIR-tukeen, jos vinjetointi jatkuu.

### Ongelma: Väärät värit tai arvot

**Mahdolliset syyt:**

* Kalibrointikohteita ei havaittu.
* Valittu väärä kalibrointikohdemalli.
* Heijastavuuden kalibrointi on pois käytöstä.
* Kohdekuvien laatu on heikko.

**Ratkaisut:**

1. Varmista, että heijastavuuden kalibrointi on käytössä.
2. Tarkista Debug Log -lokista, onko kohdassa &quot;Target found&quot; (Kohde löydetty) -viesti.
3. Tarkista kohdekuvien laatu.
4. Käsittele uudelleen merkitsemällä oikeat kohteet.

### Ongelma: NDVI-arvot näyttävät virheellisiltä

**Odotetut NDVI-alueet:*** **Vesi, kivet, maaperä**: -0,1–0,2
* **Harva/epäterve kasvillisuus**: 0,2–0,4
* **Kohtalainen kasvillisuus**: 0,4–0,6
* **Terve, tiheä kasvillisuus**: 0,6–0,9**Jos arvot ovat näiden alueiden ulkopuolella:**

1. Varmista, että heijastavuuden kalibrointi on suoritettu.
2. Varmista, että valosensorin loki on mukana.
3. Tarkista, että kalibrointikohteet on havaittu.
4. Varmista, että oikea kameramalli on havaittu.
5. Tarkista kohteen kuvan ottamisen ajoitus ja olosuhteet.

***

## Käsiteltyjen kuvien käyttö

### Fotogrammetriaan / ortomosaiikin luomiseen

**Suositeltava työnkulku:**

1.**Tuo kalibroidut heijastavuuskuvat** fotogrammetriaohjelmistoon:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Säilytä EXIF-metatiedot**: Varmista, että GPS-tiedot säilyvät geotaggaamista varten
3. **Kalibroidut työnkulut**: Käytä heijastavuuskuvia tieteellisen tarkkuuden saavuttamiseksi
4. **Käsittele indeksimosaiikit**: Luo NDVI-ortomosaiikit yksittäisistä indeksikuvista
5. **Vie georeferenssoidut GeoTIFF**: Käytettäväksi GIS-sovelluksissa

### GIS-analyysiin

**Suositeltava työnkulku:**

1.**Lataa QGIS-, ArcGIS- tai vastaavaan ohjelmaan**

2.**Käytä 16-bittisiä TIFF** heijastavuuskuvia monikaistaisessa analyysissä
3. **Käytä indeksikuvia** (NDVI, NDRE) käyttövalmiina kasvillisuuskerroksina
4. **Rasterilaskuri**: Yhdistä kaistat mukautettuun analyysiin
5. **Vie**: Luo luokittelukarttoja, muutosten havaitsemista, kasvillisuuden terveyskarttoja

### Suoraan analysointia / raportointia varten

**Suositeltu työnkulku:**

1.**Käytä indeksisuhteita LUT-väreillä** visuaalisia raportteja varten
2. **Pura tilastot**: Keskimääräinen NDVI kenttää/tonttia kohti
3. **Aikasarja**: Vertaa indeksejä useiden istuntojen välillä
4. **Luo raportteja**: Sisällytä kartat, tilastot ja visualisoinnit***

## Arkistointi ja varmuuskopiointi

### Suositeltava varmuuskopiointistrategia

**Mitä tallentaa:*** ✅ **Alkuperäiset RAW/JPG-kuvat** – Arkistoi erilliselle levylle/pilvipalveluun
* ✅ **Käsitellyt tulokset** – Säilytä kalibroidut kuvat ja indeksit
* ✅ **Projektitiedosto** – Sisältää kaikki asetukset uudelleen käsittelyä varten tarvittaessa
* ✅ **Virheenkorjausloki** – Dokumentoi käsittelyn yksityiskohdat
* ✅ **Kalibrointikohdekuvat** – Tarkistusta ja uudelleen käsittelyä varten**Tallennussuositukset:*** **Välitön varmuuskopiointi**: Ulkoinen kiintolevy
* **Pitkäaikainen arkistointi**: Pilvitallennus (Google Drive, Dropbox jne.)
* **Kriittiset tiedot**: Säilytä 2–3 kopiota eri paikoissa***

## Seuraavat käsittelyt

### Projektin asetusten uudelleenkäyttö

Jos käsittelet vastaavia tietojoukkoja tulevaisuudessa:

1. **Tallenna projektimalli** (jos et ole vielä tehnyt sitä)
2. **Luo uusi projekti** tallennetun mallin avulla
3. **Tuo uudet kuvat**

4.**Käsittele**samoilla asetuksilla yhdenmukaisuuden varmistamiseksi

### Useiden istuntojen eräprosessointi

Useita istuntoja/tietojoukkoja varten:**Vaihtoehto 1: GUI – Useita projekteja**

* Luo erillinen projekti jokaiselle istunnolle
* Käytä yhdenmukaisia malliasetuksia
* Käsittele yksi kerrallaan

**Vaihtoehto 2: Chloros CLI (vain Chloros+)**

* Automatisoi eräprosessointi
* Käsittele useita kansioita skripteillä
* Katso [CLI-dokumentaatio](../CLI.md)

**Vaihtoehto 3: Python SDK (vain Chloros+)**

* Ohjelmoitu ohjaus
* Integrointi analyysiputkiin
* Katso [API-dokumentaatio](../api-python-sdk.md)

***

## Jälkikäsittelyn vianmääritys

### Uudelleenkäsittely eri asetuksilla

Jos tulokset eivät ole tyydyttäviä:

1. Säilytä alkuperäiset kuvat (älä koskaan poista niitä)
2. Avaa sama projekti Chloros:ssä
3. Säädä asetuksia Projektin asetukset -paneelissa
4. Käsittele uudelleen – tulokset korvaavat aiemmat tulokset

### Kuvien osajoukon käsittely

Vain tiettyjen kuvien uudelleenkäsittely:

1. Luo uusi projekti
2. Tuo vain uudelleenkäsittelyä tarvitsevat kuvat
3. Käytä samaa asetusmallia
4. Käsittele pienempiä tietojoukkoja

### Ohje

Jos kohtaat ongelmia:

* 📧 **Sähköposti**: info@mapir.camera (liitä mukaan vianmääritysprotokolli)
* 🌐 **Tuki**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **UKK**: [Usein kysytyt kysymykset](../faq.md)
* 📖 **Dokumentaatio**: [Chloros-käyttöopas](../)***

## Yhteenveto: Valmis työnkulku

Olet nyt suorittanut Chloros-käsittelyn koko työnkulun:

1. ✅ **Luotu projekti** - Katso [Projektit](../projects.md)
2. ✅ **Lisätty tiedostoja** - Katso [Tiedostojen lisääminen](adding-files-to-a-project.md)
3. ✅ **Säätöasetukset** - Katso [Projektin asetusten säätäminen](adjusting-project-settings.md)
4. ✅ **Merkityt kohteet** - Katso [Kohdekuvien valitseminen](choosing-target-images.md)
5. ✅ **Käsittely käynnistetty** - Katso [Käsittelyn käynnistäminen](starting-the-processing.md)
6. ✅ **Edistymistä seurattu** - Katso [Käsittelyn seuranta](monitoring-the-processing.md)
7. ✅ **Tulosten tarkastelu** - Tämä sivu**Kalibroidut, heijastavuudeltaan korjatut monispektrikuvasi ovat valmiina analysoitavaksi!**

***

## Lisäresurssit

### Lisäominaisuudet

* [**Kuvankatseluohjelma**](../image-viewer-gui/opening-an-image-full-screen.md) - Interaktiivinen visualisointi ja analysointi
* [**Indeksi/LUT-hiekkalaatikko**](../image-viewer-gui/index-lut-sandbox.md) - Mukautettu indeksitestaus
* [**Monispektriset indeksikaavat**](../project-settings/multispectral-index-formulas.md) - Täydellinen indeksiviite

### Automaatio ja integraatio

* [**CLI-dokumentaatio**](../CLI.md) – Komentoriviltä suoritettava eräprosessointi
* [**Python SDK**](../api-python-sdk.md) - Ohjelmoitu automaatio
* [**Chloros+ Ominaisuudet**](../#chloros) - Edistyneet käsittelyominaisuudet

### Tuki ja oppiminen

* [**UKK**](../faq.md) - Vastauksia yleisiin kysymyksiin
* [**Kalibrointikohteet**](../calibration-targets.md) - Heijastavuuden kalibroinnin ymmärtäminen
* [**Tuetut kamerat**](../supported-cameras.md) - Yhteensopiva laitteisto
