# Käsittelyn päättäminen

Kun Chloros on saanut käsittelyn päätökseen, on aika tarkastella tuloksia, varmistaa tuloslaatu ja valmistella käsitellyt kuvat käytettäväksi työnkulussasi. Tällä sivulla opastetaan sinua viimeisten vaiheiden ja seuraavien toimien läpi.

## Käsittelyn päättymisen merkki

Kun käsittely on suoritettu onnistuneesti, näet useita ilmaisimia:

* ✅ **Edistymispalkki**: Saavuttaa 100 %:n valmiusasteen
* ✅ **Vianmääritysloki**: Näyttää `[RUN-SUMMARY]`-ohjelman viimeiset rivit ja niiden lukumäärät (kuvat, kameraryhmät, kohteet, kalibroidut kuvat, tallennetut tiedostot)
* ✅ **Käynnistyspainike**: Aktivoituu uudelleen (valmis seuraavaan käsittelykierrokseen)
* ✅ **Tulostustiedostot**: Kaikki käsitellyt kuvat on tallennettu projektin tulostuskansioon (alla)

{% hint style="warning" %}
**Käsittely, joka ei tuota kuvia, on epäonnistunut.** Jos pyysit kuvatuotteita ja käsittelykierros ei kirjoittanut yhtään, Chloros raportoi sen epäonnistumiseksi — `[RUN-SUMMARY]` vihjaa lokin nimessä todennäköisen syyn (mitään ei tuotu, kohdetta ei havaittu tai jokainen pyydetty tuote ohitettiin soveltumattomana). Vastaava CLI-ajo päättyy nollasta poikkeavalla tuloksella. Tarkoituksellisesti vain metatietoja tuottava ajo (kaikki vientituotteet pois päältä, ei hakemistoja) on silti onnistunut. Katso [CLI-viite](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Käsiteltyjen kuvien paikantaminen

### Tulostuskansion avaaminen

1. Napsauta **Päävalikko**-kuvaketta <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (vasemmassa yläkulmassa)
2. Valitse **”Avaa projektikansio”**

3. Tiedostoselain avautuu projektikansioon
4. Etsi projekti nimen perusteella

### Tulostuspuurakenne

Tuotteet tallennetaan **projektikansion alle, ryhmiteltynä kameran ja sen jälkeen tiedostomuodon mukaan**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per selected index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

* **Kamerakansio**: `LATT-<sensor>-<lens>-F<filter>` LATTICE-kameralle (vastaa kuvan EXIF-tietoja `Model`), `<model>_<filter>` kameralle Survey3 (esim. `Survey3N_RGN`). Kaksi kameraa, joilla on sama kenno ja suodatin mutta eri objektiivi, säilytetään erillisissä hakemistopuissa — vinjetointi, kuvakulma ja vääristymä eroavat toisistaan.
* **Muoto-kansio**: noudattaa vientimuotoasetustasi — `tiff16`, `tiff8`, `png8`, `jpg8` tai `tiff32` tiedostolle TIFF (32-bittinen, prosentti). Radiance on aina float32-tyyppinen ja sijoittuu aina `tiff32`:n alle.
* **Tuotekansiot**:
  * `Reflectance_Calibrated_Images/` — kalibroitu heijastavuus
  * `Debayered_Images/` — lineaarisesti debayeroitu (LATTICE)
  * `Preview_Images/` — näytön esikatselu (LATTICE)
  * `Radiance_Images/` — float32-spektrinen säteilyvoimakkuus, W/m²/sr/nm (LATTICE-monispektrinen)
  * `Vignette_Corrected_Images/` **tai** `Sensor_Response_Images/` — kalibroimaton varavaihtoehto kehyksille, joissa ei ole heijastavuusviitettä; kumpaakin on täsmälleen yksi kutakin ajokertaa kohti, ja valinta tapahtuu Vignette-korjausasetuksen perusteella
  * `<INDEX>_Index_Images/` — yksi kansio kutakin valittua indeksiä kohti (esim. `NDVI_Index_Images`)

{% hint style="info" %}
**Jokainen viety tuote säilyttää LÄHDE-tiedoston nimen.**`capture_..._raw.tif`:n säteilyvoimakkuusvienti on edelleen nimeltään `capture_..._raw.tif` — se vain sijaitsee kansiossa `tiff32/Radiance_Images/`.**Kansio tunnistaa tuotteen, ei tiedostonimi**, joten hakusanalla `*radiance*.tif` ei löydy mitään; hae sen sijaan hakemiston perusteella.
{% endhint %}



<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-… camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images) and tiff32 (Radiance_Images) subfolders visible -->### Kuinka monta tiedostoa pitäisi olla?

Älä laske kaavan mukaan — tulosteiden määrä riippuu siitä, mitkä tuotteet on otettu käyttöön ja mitkä koskevat kutakin kameraa (esim. RGB-kameroille ei tuoteta säteily- tai heijastusarvoja). Luotettava lukumäärä löytyy lokista: viimeinen `[RUN-SUMMARY]`-rivi ilmoittaa tarkalleen, kuinka monta tiedostoa on kirjoitettu, ja viherivit selittävät, mitä on ohitettu.

***

## Käsiteltyjen kuvien tarkastelu

### Pikaesikatselu File Explorerissa

**Windows:n sisäänrakennettu esikatselu:**

1. Siirry tuotekansioon (esim. `tiff16/Reflectance_Calibrated_Images/`)
2. Valitse kuvatiedosto
3. Esikatselu näkyy Windows Explorerin esikatselupaneelissa
4. Selaa kuvia nuolinäppäimillä

### Esikatselu ulkoisissa kuvankatseluohjelmissa

**Suositellut katseluohjelmat:*** **QGIS** – Ilmainen GIS-ohjelmisto (sopii parhaiten georeferoituun monispektrianalyysiin)
* **IrfanView** – Nopea, kevyt kuvankatseluohjelma (tukee TIFF-tiedostomuotoa)
* **Adobe Photoshop** – ammattimainen kuvankäsittely (tukee TIFF-tiedostomuotoa)
* **GIMP** – ilmainen vaihtoehto Photoshopille
* **Windows Photos** – Perustason katselu (ei välttämättä tue 16-bittistä TIFF-tiedostomuotoa)

### Esikatselu Chloros-kuvankatseluohjelmassa

Käytä Chloros:n sisäänrakennettua kuvankatseluohjelmaa edistyneeseen esikatseluun:

1. Napsauta kuvan pikkukuvaa tiedostoselaimessa
2. Kuva avautuu pääesikatselualueelle
3. Napsauta vasemman sivupalkin **Image Viewer**-<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">-välilehteä
4. Käytä [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) interaktiiviseen analysointiin

Katso yksityiskohtaiset ohjeet kohdasta [Kuvankatseluohjelma](../image-viewer-gui/opening-an-image-full-screen.md).

***

## Heijastavuuden pikseliarvojen lukeminen (GIS / Pix4D / skriptit)

Heijastavuus tallennetaan kokonaislukuna (DN), ja **DN-arvo, joka vastaa ρ = 1,0, riippuu lähdekamerasta**:

| Lähde          | ρ = 1,0 on | Miten selvittää                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (varaa ρ 2,0:een asti) | Tiedostoon on merkitty XMP-tunniste `Chloros:PixelScale=32768` |
| Survey3         | **65535** (rajoitettu arvoon ρ 1,0)     | Ei `Chloros:*`-XMP-tunnisteita — tämä puuttuminen on merkki |

**Lue `Chloros:PixelScale`-tunniste ja jaa arvo sillä** sen sijaan, että oletettaisiin yleisesti arvoa 65535 — LATTICE-heijastavuuden jakaminen luvulla 65535 puolittaa hiljaisesti jokaisen arvon. Yksi ääritapaus ei sisällä mittakaavaa suunnittelun vuoksi: 8-bittisenä lähteenä tallennettu ja 8-bittisenä tulosteena tallennettu kuva leikkautuu, sitä ei skaalata uudelleen, eikä sille anneta tarkoituksella skaalausmerkintää — vie se uudelleen 16- tai 32-bittisenä sen sijaan, että jakaisit sen. Katso [Lähtökuvamuodot](../output-image-formats.md) saadaksesi koko tarinan.***

## Vientiin siirrettävät metatiedot

Jokainen tuote säilyttää lähdekuvauksen **GPS-lohkon**ja sen**EXIF-alilohkon**, joten
vienti sisältää `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` ja
`CameraSerialNumber` sekä georeferenssitiedot.

{% hint style="warning" %}
**Jos ortomosaiikin mittakaava on täysin järjetön, tarkista ensin `FocalLength`.**
Pix4D laskee maaperän näytteenottovälin polttovälistä ja korkeudesta. Ilman tätä tunnistetta
se käyttää oletusarvoisesti täysin virheellistä mittakaavaa — yhdessä mitatussa 49 kuvan lennossa 411 m × 160 m:n
kooltaan oleva appelsiinitarha rekonstruoitiin 47,8 km × 13 km:n kokoiseksi, mikä tuotti 455 megapikselin ortokuvan, joka koostui pääosin
tyhjästä tilasta. Hidas ruututus ja odottamattoman suuri tiedostokoko ovat tämän oireita, eivät erillisiä
ongelmia.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Kaikkia tunnisteita ei kopioida. IFD0:n rakenteelliset tunnisteet jätetään tarkoituksella pois (niiden kopioiminen
vääristää LATTICE-tulostetta), ja `ExifImageWidth` / `ExifImageHeight` jätetään pois,
koska ne kuvaavat alkuperäistä kaappausta — muuten koon muutetun viennin
mitat olisivat ristiriidassa sen oman rasterin mittojen kanssa.

***

## Virhelokin tarkastelu

### Tarkista varoitukset tai virheet

1. Avaa **Debug Log**-välilehti <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">
2. Selaa viestejä
3. Etsi keltaisia varoituksia tai punaisia virheitä
4. Lue `[RUN-SUMMARY]`-rivit ja mahdolliset vihjeet
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

* Tuote ei sovellu kyseiseen kameraan (esim. säteily/heijastavuus RGB-kameroille — tämä näkyy lokissa)
* Vaadittu vertailuarvo puuttui (esim. heijastavuus ilman kohdetta ja ilman `.daq`-alasvirtausta)
* Tuotteen vientivalintaruutu oli poistettu käytöstä projektin asetuksissa
* Levytila loppui viennin aikana

**Ratkaisut:**

1. Tarkista virhelokista `[RUN-SUMMARY]`-vihjeet ja `[EXPORT-CHECK]`-rivit — ne selittävät kamerakohtaiset ohitukset
2. Tarkista vientituotteiden valintaruudut kohdassa [Projektin asetukset](adjusting-project-settings.md)
3. Varmista, että levytilaa oli riittävästi
4. Suorita käsittely uudelleen, kun olet korjannut syyn

### Ongelma: Tummat tai kirkkaat reunat (vignettointi edelleen näkyvissä)

**Mahdolliset syyt:**

* Vignetoinnin korjaus pois käytöstä
* Kameraa/objektiivia ei ole Chloros-profiilitietokannassa
* Äärimmäinen vignetointi, jota ei voida korjata

**Ratkaisut:**

1. Varmista, että vignettikorjaus on otettu käyttöön projektin asetuksissa
2. Tarkista, että kameramalli on tunnistettu oikein
3. Ota yhteyttä MAPIR-tukeen, jos vignettia esiintyy edelleen

### Ongelma: Virheelliset värit tai arvot

**Mahdolliset syyt:**

* Kalibrointikohteita ei havaittu
* Valittu väärä kalibrointikohteen malli
* Heijastavuuskalibrointi on pois käytöstä
* Kohdekuvien laatu on huono

**Ratkaisut:**

1. Varmista, että heijastavuuskalibrointi on otettu käyttöön
2. Tarkista ”Kohde löydetty” -viestit virheenkorjauslokista
3. Tarkista kohdekuvien laatu
4. Suorita käsittely uudelleen merkitsemällä oikeat kohteet

### Ongelma: NDVI-arvot vaikuttavat virheellisiltä

**Odotetut NDVI-arvoalueet:*** **Vesi, kivet, maaperä**: -0,1 – 0,2
* **Harva/epäterve kasvillisuus**: 0,2–0,4
* **Kohtalainen kasvillisuus**: 0,4–0,6
* **Terve, tiheä kasvillisuus**: 0,6–0,9**Jos arvot ovat näiden alueiden ulkopuolella:**

1. Varmista, että heijastavuuskalibrointi on suoritettu
2. Varmista, että valosensorin loki on mukana
3. Tarkista, että kalibrointikohteet on tunnistettu
4. Varmista, että oikea kameramalli on tunnistettu
5. Tarkista kohdekuvien ottamisajankohdat ja olosuhteet
6. Jos lasket indeksit itse heijastavuustiedostoista, varmista, että olet jakanut arvon tiedoston `Chloros:PixelScale`-arvolla (katso yllä)

***

## Käsiteltyjen kuvien käyttö

### Fotogrammetriaan / ortomosaiikin luomiseen

**Suositeltu työnkulku:**

1.**Tuo kalibroidut heijastavuuskuvat** fotogrammetriaohjelmistoon:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Säilytä EXIF-metatiedot**: Varmista, että GPS-tiedot säilyvät geotunnisteita varten
3. **Kalibroidut työnkulut**: Käytä heijastuskuvia tieteellisen tarkkuuden varmistamiseksi — LATTICE-heijastuskuvat sisältävät XMP-kalibrointitunnisteet, joita Pix4D lukee
4. **Käsittele indeksimosaiikit**: Luo NDVI-ortomosaiikit yksittäisistä indeksikuvista
5. **Vie georeferoidut GeoTIFF-tiedostot**: Käyttöä varten GIS-sovelluksissa

### GIS-analyysia varten

**Suositeltu työnkulku:**

1.**Lataa QGIS:ään, ArcGIS:ään tai vastaavaan ohjelmaan**

2.**Käytä 16-bittisiä TIFF**-heijastuskuvia monikaistaisessa analyysissä (jaa tiedoston `Chloros:PixelScale`-arvolla)
3. **Käytä indeksikuvia** (NDVI, NDRE) käyttövalmiina kasvillisuuskerroksina
4. **Rasterilaskuri**: Yhdistä kaistat mukautettua analyysia varten
5. **Vienti**: Luo luokittelukarttoja, muutosten havaitsemista ja kasvillisuuden terveyskarttoja

### Suoraan analysointiin / raportointiin

**Suositeltu työnkulku:**

1.**Käytä indeksikuvia LUT-väreillä** visuaalisissa raporteissa
2. **Poimi tilastotiedot**: NDVI-keskiarvo pelto- tai koealakohtaisesti
3. **Aikasarja**: Vertaa indeksejä eri mittauskertojen välillä
4. **Luo raportteja**: Sisällytä karttoja, tilastoja ja visualisointeja***

## Arkistointi ja varmuuskopiointi

### Suositeltu varmuuskopiointistrategia

**Mitä tallentaa:*** ✅ **Alkuperäiset RAW-/JPG-kuvat tai LATTICE-raakakuvat** – Arkistoi erilliselle levylle tai pilvipalveluun; raakakuvat ovat prosessin lähde, ja kaikki muu voidaan luoda uudelleen niiden pohjalta
* ✅ **`.daq` / `.csv` -valosensoritiedostot** – Tarvitaan heijastavuuden uudelleenlaskemiseen myöhemmin
* ✅ **Käsitellyt tulokset** – Säilytä kalibroidut kuvat ja indeksit
* ✅ **Projektikansio** (`project.json` ja siihen liittyvät tiedostot) – Sisältää kaikki asetukset uudelleenprosessointia varten, jos sitä tarvitaan
* ✅ **Vianmääritysloki** – Dokumentoi käsittelyn yksityiskohdat
* ✅ **Kalibrointikohdekuvat** – Tarkistusta ja uudelleenkäsittelyä varten**Tallennussuositukset:*** **Välitön varmuuskopiointi**: Ulkoinen kiintolevy
* **Pitkäaikainen arkistointi**: Pilvitallennustila (Google Drive, Dropbox jne.)
* **Kriittiset tiedot**: Säilytä 2–3 kopiota eri paikoissa***

## Seuraavat käsittelykierrokset

### Projektiasetusten uudelleenkäyttö

Jos käsittelet vastaavia aineistoja tulevaisuudessa:

1. **Tallenna projektimalli** (jos et ole vielä tehnyt niin)
2. **Luo uusi projekti** tallennetun mallin avulla
3. **Tuo uudet kuvat**

4.**Käsittele**samoilla asetuksilla yhdenmukaisuuden varmistamiseksi

### Useiden istuntojen eräkäsittely

Useiden istuntojen/tietojoukkojen tapauksessa:**Vaihtoehto 1: Käyttöliittymä – Useita projekteja**

* Luo erillinen projekti jokaiselle istunnolle
* Käytä yhdenmukaisia malliasetuksia
* Käsittele yksi kerrallaan

**Vaihtoehto 2: Chloros CLI (vain Chloros+)**

* Automatisoi eräkäsittely
* Käsittele useita kansioita skriptien avulla
* Katso [CLI-ohjeet](../CLI.md) ja [CLI-viiteopas](../reference/cli-reference.md)

**Vaihtoehto 3: Python SDK (vain Chloros+)**

* Ohjelmoitava ohjaus
* Integrointi analyysiprosesseihin
* Katso [API-dokumentaatio](../api-python-sdk.md) ja [SDK-viite](../reference/sdk-reference.md)

***

## Jälkikäsittelyn vianmääritys

### Uudelleenkäsittely eri asetuksilla

Jos tulokset eivät ole tyydyttäviä:

1. Säilytä alkuperäiset kuvat (älä koskaan poista niitä)
2. Avaa sama projekti ohjelmassa Chloros
3. Säädä asetuksia Projektin asetukset -paneelissa
4. Suorita käsittely uudelleen — tulokset tallennetaan samoihin tuotekansioihin, joten edellisen ajon samannimiset tiedostot korvataan

### Kuvien osajoukon käsittely

Vain tiettyjen kuvien uudelleenkäsittely:

1. Luo uusi projekti
2. Tuo vain ne kuvat, jotka vaativat uudelleenkäsittelyä
3. Käytä samaa asetusmallia
4. Käsittele pienempää kuvajoukkoa

### Ohjeiden saaminen

Jos kohtaat ongelmia:

* 📧 **Sähköposti**: info@mapir.camera (liitä mukaan vianmääritysloki)
* 🌐 **Tuki**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **UKK**: [Usein kysytyt kysymykset](../faq.md)
* 📖 **Dokumentaatio**: [Chloros-käyttöopas](../)***

## Yhteenveto: Koko työnkulku

Olet nyt suorittanut koko Chloros-käsittelytyönkulun:

1. ✅ **Luotu projekti** – Katso [Projektit](../projects.md)
2. ✅ **Lisätty tiedostoja** – Katso [Tiedostojen lisääminen](adding-files-to-a-project.md)
3. ✅ **Säätänyt asetukset** – Katso [Projektin asetusten säätäminen](adjusting-project-settings.md)
4. ✅ **Kohteet merkitty** – Katso [Kohdekuvien valinta](choosing-target-images.md)
5. ✅ **Käsittely aloitettu** – Katso [Käsittelyn aloittaminen](starting-the-processing.md)
6. ✅ **Seurattu edistymistä** – Katso [Käsittelyn seuranta](monitoring-the-processing.md)
7. ✅ **Tarkasteltu tuloksia** – Tämä sivu**Kalibroidut, heijastavuudeltaan korjatut monispektrikuvasi ovat valmiina analysoitavaksi!**

***

## Lisäresurssit

### Edistyneet ominaisuudet

* [**Kuvankatseluohjelma**](../image-viewer-gui/opening-an-image-full-screen.md) – Interaktiivinen visualisointi ja analysointi
* [**Indeksi-/LUT-kokeilualusta**](../image-viewer-gui/index-lut-sandbox.md) – Mukautettujen indeksien testaaminen
* [**Monispektriset indeksikaavat**](../project-settings/multispectral-index-formulas.md) – Kattava indeksiviite

### Automaatio ja integraatio

* [**CLI-dokumentaatio**](../CLI.md) – Komentorivipohjainen eräkäsittely
* [**Python SDK**](../api-python-sdk.md) – Ohjelmoitu automaatio
* [**Chloros+ Ominaisuudet**](../#chloros) – Edistyneet käsittelyominaisuudet

### Tuki ja oppiminen

* [**Usein kysytyt kysymykset**](../faq.md) - Vastauksia yleisiin kysymyksiin
* [**Kalibrointikohteet**](../calibration-targets.md) - Heijastavuuskalibroinnin ymmärtäminen
* [**Tuetut kamerat**](../supported-cameras.md) - Yhteensopiva laitteisto
