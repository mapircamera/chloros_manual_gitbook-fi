---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Monispektriset indeksikaavat

Seuraavissa indeksikaavoissa käytetään yhdistelmää suodattimien keskimääräisistä läpäisyalueista (Survey3):

<table><thead><tr><th align="center">Survey3 Suodattimen väri</th><th width="196.199951171875" align="center">Survey3 Suodattimen nimi</th><th width="159.800048828125" align="center">Läpäisyalue (FWHM)</th><th align="center">Keskimääräinen läpäisykyky</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re-RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>

Kun näitä kaavoja käytetään, nimi voi päättyä merkintään &quot;\_1&quot; tai &quot;\_2&quot;, mikä vastaa sitä, kumpaNIR-suodatinta (NIR1 tai NIR2) on käytetty.

LATTICE M3C (Bayerin kolmikaistainen) -kameroissa sama indeksointimoottori käyttää M3C-suodattimien kaistoja:

| M3C-suodatin | Kaista 1 (keskus/ FWHM) | Kaista 2 (keskus/ FWHM) | Kaista 3 (keskus/ FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

LATTICE M3M -kamerat ovat yksikaistaisia (yksi kapeakaistainen suodatin kameraa kohti), joten monikaistaisia indeksejä ei lasketa yksittäisestä M3M-kuvasta. Laskeaksesi indeksejä M3M:llä, yhdistä kaksi tai useampia kameroita kohdistetuksi monikaistapinoksi ja käytä LATTICE-indeksimoottoria (`chloros-cli lattice index` tai käyttöliittymän reaaliaikaista indeksilaskuria).

***

## Missä kukin indeksinimi toimii

Chloros sisältää **kolme** indeksipintaa, joiden esiasetellut luettelot eivät ole identtisiä. Tarkista tästä osiosta, toimiiko nimi siinä yhteydessä, jossa aiot sitä käyttää.

| Missä olet | Mikä luettelo pätee | Lukumäärä |
| --- | --- | --- |
| Projektiasetukset → Indeksi → Lisää indeksi (käyttöliittymä) | Pinta 1 | 27 |
| Kuvan katseluohjelma [Indeksi/LUT-hiekkalaatikko](../image-viewer-gui/index-lut-sandbox.md) (käyttöliittymä) | Pinta 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | Pinta 2 | 22 |
| SDK `process_folder(indices=[...])` | Pinta 2 | 22 |
| `chloros-cli lattice index --preset` | Pinta 3 | 22 (eri 22) |
| Kamerat-välilehden reaaliaikainen indeksilaskuri | Pinta 3 | 22 (eri 22) |

Surface 1 ja 2 käsittelevät **yhtä kuvaa kerrallaan yhdestä kamerasta**käyttäen symbolipaikkoja `x`/`y`/`z`(/`a`), jotka on sidottu kyseisen kameran suodatinkanaviin. Surface 3 käsittelee**kohdistettua monikaistapinoa** — useita LATTICE-kameroita, jotka on rekisteröity yhteen kuutioon — ja viittaa kanaviin pienillä kirjaimilla.

### 1. GUI-projektin asetukset / Image Viewer -sandbox-pudotusvalikko — 27 kaavaa

Pudotusvalikko listaa ne tässä järjestyksessä (järjestys on lisäysjärjestys, ei aakkosjärjestys):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

GUI:ssa vedät kamerasi suodatinkanavat kaavan kaistapaikkoihin, joten mitä tahansa kaavaa voidaan käyttää minkä tahansa kamerasi tukeman kaistamäärityksen kanssa. Tallentamasi mukautetut kaavat lisätään tämän luettelon alle.

**Viisi GUI-käyttöliittymän omaa** kaavaa — joita CLI / SDK `--indices` -luettelo ei hyväksy — on toteutettu seuraavasti:

| Vain GUI-käyttöliittymän esiasetus | Kaava (toteutettuna) | Paikat |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a (neljä paikkaa) |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

Kunkin kaavan tarkoitettu arvomääritys on esitetty omassa osiossaan tämän sivun alempana (esimerkiksi GARI edellyttää, että x = Green, y = NIR, z = Blue, a = Red). GARI on ainoa kaava sivustolla Chloros, jossa käytetään neljättä paikkaa.

### 2. CLI / SDK `--indices`-nimien laajennus — 22 esiasetusta

`chloros-cli process --indices`-vaihtoehto (ja SDK `indices`-parametri) hyväksyy seuraavat esiasetusten nimet:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**Tuntemattomat hakemistonimet ohitetaan ilman ilmoitusta.** Tämän luettelon ulkopuolella oleva nimi (mukaan lukien viisi pelkästään käyttöliittymässä käytettävää kaavaa `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` sekä kaikki GUI:ssa tallentamasi mukautetut kaavat) hylätään vain loki-ilmoituksella — suoritus jatkuu ilman kyseistä indeksiä, ja suoritus itsessään raportoidaan edelleen onnistuneeksi. Ilmoitus tulostetaan seuraavasti:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

Nimien vertailussa ei oteta huomioon kirjainkokoa, ja välilyönnit poistetaan ennen vertailua, joten `ndvi`, `NDVI` ja ` NDVI ` ovat sama esiasetus. Esiasetus ohitetaan myös, jos se vaatii kaistan, jota kamerasi suodatin ei tarjoa.
{% endhint %}

Tarkat kaavat sellaisina kuin ne on toteutettu (merkit `x`/`y`/`z` ovat kaistapaikkoja; oletusmääritykset on esitetty esiasetusta kohden):

| Esiasetus | Kaava (toteutettuna) | Oletussuodatin | Paikat (x, y, z) |
| --- | --- | --- | --- |
| NDVI | `(y-x)/(y+x)` | RGN | Red, NIR |
| GNDVI | `(y-x)/(y+x)` | RGN | Green, NIR |
| NDRE | `(y-x)/(y+x)` | RE | RE, NIR |
| OSAVI | `(y-x)/(y+x+0.16)` | RGN | Red, NIR |
| SAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Red, NIR |
| MSAVI2 | `(2*y+1-sqrt((2*y+1)*(2*y+1)-8*(y-x)))/2` | RGN | Red, NIR |
| EVI | `2.5*(y-x)/(y+6*x-7.5*z+1)` | RGN | Red, NIR, Blue |
| MSR | `((y/x)-1)/(sqrt(y/x)+1)` | RGN | Red, NIR |
| TDVI | `1.5*(y-x)/sqrt(y*y+x+0.5)` | RGN | Red, NIR |
| LAI | `3.618*(2.5*(y-x)/(y+6*x-7.5*z+1))-0.118` | RGN | Red, NIR, Blue |
| GCI | `(y/x)-1` | RGN | Green, NIR |
| GRVI | `y/x` | RGN | Green, NIR |
| GSAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Green, NIR |
| GOSAVI | `(y-x)/(y+x+0.16)` | RGN | Green, NIR |
| NLI | `((y*y)-x)/((y*y)+x)` | RGN | Red, NIR |
| MNLI | `((y*y-x)*(1+0.5))/((y*y)+x+0.5)` | RGN | Red, NIR |
| RDVI | `(y-x)/sqrt(y+x)` | RGN | Red, NIR |
| WDRVI | `(0.2*y-x)/(0.2*y+x)` | RGN | Red, NIR |
| CVI | `(z/y)/(x/y)` | RGB | Red, Green, Blue |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | RGB | Red, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### Miten esiasetuksen nimi muuttuu kaistapaikoiksi

Kun syötät pelkän nimen, kuten `NDVI`, Chloros:n on päätettävä, minkä tiedoston mistäkin kanavasta kukin symboli lukee. Se käyttää tätä taulukkoa, joka yhdistää suodatinkoodin kunkin kanavan taulukon sijaintiin:

| Suodatinkoodi | Kanava → taulukon indeksi |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red` hyväksytään Orange:n aliasena, myös 0) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

Esiasetuksen **oletussuodatinta** (yllä oleva sarake ”Oletussuodatin”) käytetään, kun projekti sisältää kyseisellä suodattimella käsiteltyjä kuvia. Jos näin ei ole, Chloros skannaa projektissa tosiasiallisesti olevat suodattimet järjestyksessä `RGN, OCN, NGB, RGB, RE, NIR` ja valitsee ensimmäisen, joka pystyy tarjoamaan jokaisen kanavan, jota esiasetus tarvitsee. Jos mikään ei pysty, esiasetusta ei käytetä kyseisessä ajossa. Tämän vuoksi `NDVI`, jota pyydetään pelkästään OCN -tietojoukosta, tuottaa silti järkevän tuloksen — se sitoutuu OCN-sivuston kohtiin Orange ja NIR.

LATTICE M3C -mallimerkkijonoissa suodatin on merkitty etuliitteellä `F` (`LATT-M3C-L41-FRGN`), mutta etuliite poistuu, kun suodatinkoodi luetaan kuvasta, joten FRGN-kamera tunnistaa sen yllä olevasta `RGN`-rivistä eikä vaadi erityistä käsittelyä.

### 3. LATTICE-indeksimoottori (`lattice index --preset`, reaaliaikainen indeksilaskuri) — 22 esiasetusta

LATTICE-moottori toimii kohdistetuilla monikaistapinoilla (reaaliaikaisilla matriiseilla tai vietyillä monikaistaisilla TIFF-tiedostoilla) ja käyttää kanavien nimiä pienillä kirjaimilla (`red`, `green`, `blue`, `red_edge`, `nir`). Sen esiasetusluettelo eroaa kahdesta edellä mainitusta:

| Esiasetus | Kaava | Kanavat |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | punainen, nir |
| GNDVI | `(nir - green) / (nir + green)` | vihreä, nir |
| BNDVI | `(nir - blue) / (nir + blue)` | sininen, nir |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | punainen_reuna, nir |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | sininen, vihreä, nir |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | punainen, nir |
| OSAVI | `1.5 * (nir - red) / (nir + red + 0.16)` | punainen, nir |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | punainen, infrapuna |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | sininen, punainen, infrapuna |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | punainen, infrapuna |
| CVI | `(nir / green) - (red / green)` | punainen, vihreä, lähi-infrapuna |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | punainen, lähi-infrapuna |
| TDVI | `sqrt((nir - red) / (nir + red) + 0.5)` | punainen, lähi-infrapuna |
| LAI | `3.618 * ((nir - red) / (nir + 6*red - 7.5*green + 1)) - 0.118` | punainen, vihreä, NIR |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | punainen, vihreä, sininen |
| NGRDI | `(green - red) / (green + red)` | punainen, vihreä |
| VARI | `(green - red) / (green + red - blue)` | punainen, vihreä, sininen |
| TGI | `green - 0.39*red - 0.61*blue` | punainen, vihreä, sininen |
| EXG | `2*green - red - blue` | punainen, vihreä, sininen |
| CIRE | `(nir / red_edge) - 1` | punainen\_reuna, nir |
| CIGREEN | `(nir / green) - 1` | vihreä, nir |
| NDWI | `(green - nir) / (green + nir)` | vihreä, nir |

Suorita komento `chloros-cli lattice index --list-presets` tulostaaksesi tämän taulukon asennetusta versiosta ja komento `--list-gradients` saatavilla olevien värigradienttien tulostamiseksi. Kanavasymbolit ovat kirjainkokoherkkiä, ja niiden on vastattava esiasetuksen pienillä kirjaimilla kirjoitettuja nimiä (esim. `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI

Kuten GUI:ssa ja esiasetusluettelossa CLI / SDK on toteutettu, CVI on suhdelukujen suhde -kaava:

$$
CVI = {(z / y) \over (x / y)}
$$

jossa oletusarvoinen RGB-kanavamääritys on x= Red, y= Green, z= Blue. Käyttöliittymässä voit vetää minkä tahansa kamerasi kanavan x/y/z-paikoille. Huomaa, että LATTICE-indeksimoottorin `CVI`-esiasetus käyttää erilaista kaavaa, `(NIR / Green) - (Red / Green)` — tarkista yllä olevista taulukoista käyttämäsi pinnan tiedot.

***

## ENDVI – Parannettu normalisoitu kasvillisuusindeksi (Enhanced Normalized Difference Vegetation Index)

Tämä indeksi käyttää sinistä kanavaa punaisen (NIR) ja vihreän lisäksi, ja se on suosittu NGB-suodatetuissa kameroissa, joissa sininen kaista korvaa punaisen.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

Toteutus tapahtuu symbolikaavalla `((x+y)-(2*z))/((x+y)+(2*z))` — määritä kamerasi kanavat NIR ja Green x/y-paikoille ja Blue z-paikalle (NGB-kameran tapauksessa: x= NIR, y= Green, z= Blue).

***

## EVI – Parannettu kasvillisuusindeksi

Tämä indeksi kehitettiin alun perin käytettäväksi MODIS-tietojen kanssa parannuksena NDVI-indeksiin optimoimalla kasvillisuussignaalia alueilla, joilla lehtipinta-indeksi on korkea (LAI). Se on erityisen hyödyllinen alueilla, joilla LAI-arvo on korkea ja joissa NDVI-arvo saattaa kyllästyä. Se käyttää sinisen heijastavuusalueen korjaamaan maaperän taustasignaaleja ja vähentämään ilmakehän vaikutuksia, mukaan lukien aerosolien sironta.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

EVI -arvojen tulisi olla välillä 0–1 kasvillisuuspikseleissä. Kirkkaat kohteet, kuten pilvet ja valkoiset rakennukset, sekä tummat kohteet, kuten vesi, voivat aiheuttaa poikkeavia pikseliarvoja EVI-kuvassa. Ennen EVI-kuvan luomista sinun tulisi peittää pilvet ja kirkkaat kohteet heijastuskuvasta ja valinnaisesti asettaa pikseliarvojen kynnysarvoksi 0–1.

_Lähde: Huete, A., et al. ”Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices.” Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 - Metsäpeiteindeksi 1

_Vain GUI — ei saatavilla CLI / SDK -sivuston `--indices`-esiasetuksena._

Tämä indeksi erottaa metsäkatoksen muista kasvillisuustyypeistä käyttämällä monispektrisiä heijastavuuskuvia, jotka sisältävät punaisen reunan kaistan.

$$
FCI1 = Red * RedEdge
$$

Metsäalueilla on alhaisemmat FCI1-arvot, mikä johtuu puiden alhaisemmasta heijastavuudesta ja latvuston sisällä olevista varjoista.

_Lähde: Becker, Sarah J., Craig S.T. Daughtry ja Andrew L. Russ. ”Robust forest cover indices for multispectral images.” Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## FCI2 - Metsäpeittoindeksi 2

_Vain käyttöliittymässä — ei saatavilla CLI / SDK `--indices`-esiasetuksena._

Tämä indeksi erottaa metsäkatokset muista kasvillisuustyypeistä käyttämällä monispektrisiä heijastavuuskuvia, jotka eivät sisällä punaista reuna-alueita.

$$
FCI2 = Red * NIR
$$

Metsäalueilla on alhaisemmat FCI2-arvot, mikä johtuu puiden alhaisemmasta heijastavuudesta ja latvuston sisällä olevista varjoista.

_Lähde: Becker, Sarah J., Craig S.T. Daughtry ja Andrew L. Russ. ”Robust forest cover indices for multispectral images.” Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## GEMI – Globaali ympäristöseuranta-indeksi

_Vain GUI – ei saatavillaCLI- tai SDK-esiasetuksena `--indices`._

Tätä epälineaarista kasvillisuusindeksiä käytetään maailmanlaajuiseen ympäristöseurantaan satelliittikuvien perusteella, ja sen avulla pyritään korjaamaan ilmakehän vaikutuksia. Se on samankaltainen kuin NDVI, mutta vähemmän herkkä ilmakehän vaikutuksille. Paljas maaperä vaikuttaa siihen, joten sen käyttöä ei suositella alueilla, joilla kasvillisuus on harvaa tai kohtalaisen tiheää.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Missä:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Lähde: Pinty, B. ja M. Verstraete. ”GEMI: a Non-Linear Index to Monitor Global Vegetation From Satellites”. Vegetation 101 (1992): 15–20._

***

## GARI - Green Ilmakehän vaikutuksille vastustuskykyinen indeksi

_Vain graafinen käyttöliittymä (GUI) — ei saatavilla CLI / SDK `--indices`-esiasetuksena._

Tämä indeksi on herkempi laajalle klorofyllipitoisuusalueelle ja vähemmän herkkä ilmakehän vaikutuksille kuNDVI-indeksi.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Gamma-vakio on painotusfunktio, joka riippuu ilmakehän aerosoliolosuhteista. ENVI käyttää arvoa 1,7, joka on Gitelsonin, Kaufmanin ja Merzylakin (1996, sivu 296) suosittelema arvo.

_Lähde: Gitelson, A., Y. Kaufman ja M. Merzylak. ”Green-kanavan käyttö maailmanlaajuisen kasvillisuuden kaukokartoituksessa EOS-MODIS -satelliitin avulla.” Remote Sensing of Environment 58 (1996): 289–298._

***

## GCI - Green Klorofylli-indeksi

Tätä indeksiä käytetään arvioimaan lehtien klorofyllipitoisuutta laajalla kasvilajivalikoimalla.

$$
GCI = {NIR \over Green} - 1
$$

Laaja-alainen aallonpituusalue NIR ja vihreät aallonpituudet mahdollistavat paremman klorofyllipitoisuuden ennustamisen sekä tarjoavat suuremman herkkyyden ja paremman signaali-kohinasuhteen.

_Lähde: Gitelson, A., Y. Gritz ja M. Merzlyak. ”Lehtien klorofyllipitoisuuden ja spektrisen heijastavuuden väliset suhteet sekä algoritmit korkeampien kasvien lehtien klorofyllipitoisuuden tuhoamattomaan arviointiin.” Journal of Plant Physiology 160 (2003): 271–282._

***

## GLI – Green-lehtindeksi

Tämä indeksi suunniteltiin alun perin käytettäväksi digitaalisen RGB-kameran kanssa vehnän peittävyyden mittaamiseen, jossa punaisen, vihreän ja sinisen digitaaliset arvot (DN) vaihtelevat välillä 0–255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI -arvot vaihtelevat välillä -1 – +1. Negatiiviset arvot edustavat maaperää ja elottomia kohteita, kun taas positiiviset arvot edustavat vihreitä lehtiä ja varret.

_Lähde: Louhaichi, M., M. Borman ja D. Johnson. ”Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat.” Geocarto International 16, nro 1 (2001): 65–70._

***

## GNDVI - Green Normalisoitu kasvillisuusindeksi (NDVI)

Tämä indeksi on samanlainen kuin kasvien vihreä indeksi (NDVI), paitsi että se mittaa vihreää spektriä aallonpituusalueella 540–570 nm punaisen spektrin sijaan. Tämä indeksi on herkempi klorofyllipitoisuudelle kuin kasvien punainen indeksi (NDVI).

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Lähde: Gitelson, A. ja M. Merzlyak. ”Remote Sensing of Chlorophyll Concentration in Higher Plant Leaves.” Advances in Space Research 22 (1998): 689–692._

***

## GOSAVI – Green Optimized Soil Adjusted Vegetation Index

Tämä indeksi kehitettiin alun perin väri-infrapunakuvauksen avulla maissin typpitarpeen ennustamiseksi. Se on samanlainen kuin OSAVI, mutta siinä vihreä kaista korvaa punaisen.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Lähde: Sripada, R., ym. ”Determining In-Season Nitrogen Requirements for Corn Using Aerial Color-Infrared Photography.” Väitöskirja, North Carolina State University, 2005._

***

## GRVI - Green-suhteen kasvillisuusindeksi

Tämä indeksi reagoi herkästi metsän latvuston fotosynteesinopeuteen, sillä lehtien pigmenttien muutokset vaikuttavat voimakkaasti vihreän ja punaisen heijastavuuteen.

$$
GRVI = {NIR \over Green }
$$

_Lähde: Sripada, R. ym. ”Aerial Color Infrared Photography for Determining Early In-season Nitrogen Requirements in Corn.” Agronomy Journal 98 (2006): 968–977._

***

## GSAVI – maaperän mukaan korjattu kasvillisuusindeksi (Green)

Tämä indeksi kehitettiin alun perin väri-infrapunavalokuvauksen avulla maissin typpitarpeen ennustamiseksi. Se on samanlainen kuin GSAVI (SAVI), mutta siinä vihreä kaista korvaa punaisen.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Lähde: Sripada, R., et al. ”Determining In-Season Nitrogen Requirements for Corn Using Aerial Color-Infrared Photography.” Väitöskirja, North Carolina State University, 2005._

***

## Lehtipinta-ala-indeksi (LAI)

Tätä indeksiä käytetään lehtipeitteen arviointiin sekä viljelykasvien kasvun ja sadon ennustamiseen. ENVI laskee vihreän lehtipinta-ala-indeksin (LAI) käyttäen seuraavaa Boegh et al. (2002) esittämää empiiristä kaavaa:

$$
LAI = 3.618 * EVI - 0.118
$$

Jossa EVI on:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Korkeat LAI-arvot vaihtelevat tyypillisesti noin 0:sta 3,5:een. Kuitenkin, kun kuvassa on pilviä ja muita kirkkaita kohteita, jotka tuottavat kylläisiä pikseleitä, LAI-arvot voivat ylittää 3,5:n. Ihannetapauksessa pilvet ja kirkkaat kohteet tulisi poistaa kuvasta ennen LAI-kuvan luomista.

_Lähde: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde ja A. Thomsen. ”Airborne Multi-spectral Data for Quantifying Leaf Area Index, Nitrogen Concentration and Photosynthetic Efficiency in Agriculture.” Remote Sensing of Environment 81, nro 2–3 (2002): 179–193._

***

## LCI – Lehtien klorofylli-indeksi

_Vain graafisessa käyttöliittymässä (GUI) — ei saatavilla esiasetuksena CLI / SDK `--indices`._

Tätä indeksiä käytetään korkeampien kasvien klorofyllipitoisuuden arviointiin, ja se reagoi herkästi klorofyllin absorptioista johtuviin heijastavuuden vaihteluihin.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Lähde: Datt, B. ”Remote Sensing of Water Content in Eucalyptus Leaves.” Journal of Plant Physiology 154, nro 1 (1999): 30–36._

***

## MNLI – Modifioitu epälineaarinen indeksi

Tämä indeksi on epälineaarisen indeksin (NLI) parannettu versio, johon on sisällytetty maaperän mukaan korjattu kasvillisuusindeksi (SAVI) maaperän taustan huomioon ottamiseksi. ENVI käyttää latvuston taustakorjauskerrointa (_L_) arvolla 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Lähde: Yang, Z., P. Willis ja R. Mueller. ”Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy.” Pecora 17 -kaukokartoitussymposiumin julkaisu (2008), Denver, CO._

***

## MSAVI2 – Modified Soil Adjusted Vegetation Index 2

Tämä indeksi on yksinkertaistettu versio Qi:n ym. (1994) ehdottamasta ”MSAVI” -indeksistä, joka on parannettu versio Soil Adjusted Vegetation Index -indeksistä (SAVI). Se vähentää maaperän aiheuttamaa kohinaa ja laajentaa kasvillisuussignaalin dynaamista aluetta. MSAVI2 perustuu induktiiviseen menetelmään, jossa ei käytetä vakioarvoa _L_ (kuten SAVI:ssa) terveen kasvillisuuden korostamiseen.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Lähde: Qi, J., A. Chehbouni, A. Huete, Y. Kerr ja S. Sorooshian. ”A Modified Soil Adjusted Vegetation Index.” Remote Sensing of Environment 48 (1994): 119–126._

***

## MSR – Modified Simple Ratio

Tämä indeksi on muunnos yksinkertaisesta suhteesta NIR / Red, joka on suunniteltu linearisoimaan sen suhde biofysikaalisiin parametreihin, ja se on herkempi kuin NDVI suuremmilla kasvillisuustiheyksillä.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Lähde: Chen, J. ”Evaluation of Vegetation Indices and a Modified Simple Ratio for Boreal Applications.” Canadian Journal of Remote Sensing 22 (1996): 229–242._

***

## NDRE - Normalisoitu eroRedEdge

Tämä indeksi on samanlainen kuin NDVI, mutta siinä verrataan NIR ja RedEdge välistä kontrastia sen sijaan, että verrataan Red:ään, mikä usein havaitsee kasvillisuuden stressin aikaisemmin.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - Normalisoitu ero-kasvillisuusindeksi

Tämä indeksi mittaa terveen, vihreän kasvillisuuden määrää. Sen normalisoidun eron laskentakaavan ja klorofyllin suurimman absorptio- ja heijastusalueiden yhdistelmä tekee siitä luotettavan monenlaisissa olosuhteissa. Se voi kuitenkin ylikuormittua tiheässä kasvillisuudessa, kun LAI nousee korkeaksi.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Tämän indeksin arvo vaihtelee välillä -1–1. Vihreän kasvillisuuden tyypillinen arvoalue on 0,2–0,8.

_Lähde: Rouse, J., R. Haas, J. Schell ja D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. Kolmas ERTS-symposiumi, NASA (1973): 309–317._

***

## NLI – epälineaarinen indeksi

Tämä indeksi perustuu oletukseen, että monien kasvillisuusindeksien ja pinnan biofysikaalisten parametrien välinen suhde on epälineaarinen. Se linearisoi suhteet pinnan parametrien kanssa, jotka ovat yleensä epälineaarisia.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Lähde: Goel, N., ja W. Qin. ”Puuston rakenteen vaikutukset eri kasvillisuusindeksien sekäLAI- ja Fpar-arvojen välisiin suhteisiin: tietokonesimulaatio.” Remote Sensing Reviews 10 (1994): 309–347._

***

## OSAVI – Optimoitu maaperän mukaan korjattu kasvillisuusindeksi

Tämä indeksi perustuu maaperän mukaan korjattuun kasvillisuusindeksiin (SAVI). Siinä käytetään latvuston taustakorjauskertoimena vakioarvoa 0,16. Rondeaux (1996) totesi, että tämä arvo kuvaa maaperän vaihtelua paremmin kuin SAVI, kun kasvillisuuspeite on vähäistä, ja osoittaa samalla suurempaa herkkyyttä kasvillisuuspeitteelle, joka on yli 50 %. Tätä indeksiä kannattaa käyttää alueilla, joilla kasvillisuus on suhteellisen harvaa ja maaperä näkyy latvuston läpi.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Lähde: Rondeaux, G., M. Steven ja F. Baret. ”Optimization of Soil-Adjusted Vegetation Indices.” Remote Sensing of Environment 55 (1996): 95–107._

***

## RDVI – Renormalisoitu kasvillisuusindeksi (Renormalized Difference Vegetation Index)

Tämä indeksi hyödyntää lähi-infrapuna- ja puna-aallonpituuksien välistä eroa sekä kasvillisuuden optimaalisen kasvuvalon (NDVI) korostamaan terveellistä kasvillisuutta. Se ei ole herkkä maaperän tai auringon katselukulman vaikutuksille.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Lähde: Roujean, J. ja F. Breon. ”Estimating PAR Absorbed by Vegetation from Bidirectional Reflectance Measurements.” Remote Sensing of Environment 51 (1995): 375–384._

***

## SAVI – Maaperän vaikutukset korjattu kasvillisuusindeksi

Tämä indeksi on samanlainen kuin NDVI, mutta se poistaa maaperän pikselien vaikutukset. Se käyttää latvuston taustakorjauskerrointa, _L_, joka on kasvillisuuden tiheyden funktio ja vaatii usein ennakkotietoa kasvillisuuden määrästä. Huete (1988) ehdottaa optimaalista arvoa _L_=0,5 ensimmäisen asteen maaperän taustavaihteluiden huomioon ottamiseksi. Tätä indeksiä käytetään parhaiten alueilla, joilla kasvillisuus on suhteellisen harvaa ja maaperä näkyy latvuston läpi.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Lähde: Huete, A. ”A Soil-Adjusted Vegetation Index (SAVI).” Remote Sensing of Environment 25 (1988): 295–309._

***

## TDVI – Transformed Difference Vegetation Index

Tämä indeksi on hyödyllinen kasvillisuuspeitteen seurannassa kaupunkiympäristöissä. Se ei kyllästy kuteNDVI- ja SAVI-indeksit.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Lähde: Bannari, A., H. Asalhi ja P. Teillet. ”Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping” teoksessa Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, Volume 5 (2002)._

***

## Kasvillisuuden ja maaperän raja-arvot näkyvässä spektrissä (VARI) – näkyvän spektrin ilmakehän vaikutuksille vastustuskykyinen indeksi

Tämä indeksi perustuu kasvillisuuden ja maaperän raja-arvoihin näkyvässä spektrissä (ARVI) ja sitä käytetään arvioimaan kasvillisuuden osuutta kuvassa, jossa herkkyys ilmakehän vaikutuksille on vähäinen.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Lähde: Gitelson, A., et al. &quot;Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction.&quot; International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – Laaja-alainen kasvillisuusindeksi (Wide Dynamic Range Vegetation Index)

Tämä indeksi on samanlainen kuin NDVI, mutta siinä käytetään painotuskerrointa (_a_) vähentämään lähi-infrapuna- ja punasignaalien välistä eroa kasvillisuusindeksissä (NDVI). Laaja-alainen kasvillisuusindeksi (WDRVI) on erityisen tehokas kuvissa, joissa kasvillisuuden tiheys on kohtalainen tai korkea, kun kasvillisuusindeksi (NDVI) ylittää arvon 0.6. NDVI:n arvo tasoittuu yleensä, kun kasvillisuusosuus ja lehtipinta-ala-indeksi (LAI) kasvavat, kun taas WDRVI reagoi herkästi laajempaan kasvillisuusosuuksien alueeseen ja LAI:n muutoksiin.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Painotuskerroin (_a_) voi vaihdella välillä 0,1–0,2. Henebry, Viña ja Gitelson (2004) suosittelevat arvoa 0,2.

_Lähteet_

_Gitelson, A. ”Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.” Journal of Plant Physiology 161, nro 2 (2004): 165–173._

_Henebry, G., A. Viña ja A. Gitelson. ”Laaja-alainen kasvillisuusindeksi ja sen potentiaalinen käyttökelpoisuus aukkoanalyysissä.” Gap Analysis Bulletin 12: 50–56._
