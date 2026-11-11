---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Monispektriset indeksikaavat

Seuraavissa indeksikaavoissa käytetään yhdistelmää Survey3-suodattimien keskimääräisistä läpäisyalueista:

<table><thead><tr><th align="center">Survey3-suodattimen väri</th><th width="196.199951171875" align="center">Survey3-suodattimen nimi</th><th width="159.800048828125" align="center">Läpäisyalue (FWHM)</th><th align="center">Keskimääräinen läpäisykyky</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re-RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>

Kun näitä kaavoja käytetään, nimi voi päättyä merkintään &quot;\_1&quot; tai &quot;\_2&quot;, mikä vastaa sitä, kumpaa NIR-suodatinta, joko NIR1:tä tai NIR2:tä, on käytetty.

LATTICE M3C (Bayerin kolmikaistainen) -kameroissa sama indeksimoottori käyttää M3C-suodatinkaistoja:

| M3C-suodatin | Kaista 1 (keskitaajuus/FWHM) | Kaista 2 (keskitaajuus/FWHM) | Kaista 3 (keskitaajuus/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

LATTICE M3M -kamerat ovat yksikaistaisia (yksi kapeakaistainen suodatin kameraa kohti), joten monikaistaisia indeksejä ei lasketa yksittäiselle M3M-kuvalle. Laskeaksesi indeksejä M3M:llä, yhdistä kaksi tai useampia kameroita kohdistetuksi monikaistapinoksi ja käytä LATTICE-indeksimoottoria (`chloros-cli lattice index` tai käyttöliittymän reaaliaikaista indeksilaskuria).

***

## Missä kukin indeksinimi toimii

KChlorosilla on **kolme** indeksipintaa, ja niiden esiasetellut luettelot eivät ole identtisiä. Tarkista tästä osiosta, toimiiko nimi siinä kohdassa, jossa aiot sitä käyttää.

| Missä olet | Mikä luettelo pätee | Lukumäärä |
| --- | --- | --- |
| Projektiasetukset → Indeksi → Lisää indeksi (käyttöliittymä) | Pinta 1 | 27 |
| Kuvan katseluohjelma [Indeksi/LUT-hiekkalaatikko](../image-viewer-gui/index-lut-sandbox.md) (käyttöliittymä) | Pinta 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | Pinta 2 | 22 |
| SDK `process_folder(indices=[...])` | Pinta 2 | 22 |
| `chloros-cli lattice index --preset` | Pinta 3 | 22 (eri 22) |
| Kamerat-välilehden reaaliaikainen indeksilaskuri | Pinta 3 | 22 (eri 22) |

Surface 1 ja 2 käsittelevät **yhtä kuvaa kerrallaan yhdestä kamerasta**, käyttäen symbolipaikkoja `x`/`y`/`z`(/`a`), jotka on sidottu kyseisen kameran suodatinkanaviin. Pinta 3 käsittelee**kohdistettua monikaistapinoa** — useita LATTICE-kameroita, jotka on rekisteröity yhteen kuutioon — ja viittaa kanaviin pienillä kirjaimilla.

### 1. GUI-projektin asetukset / Image Viewer -hiekkalaatikon pudotusvalikko — 27 kaavaa

Pudotusvalikko listaa ne tässä järjestyksessä (järjestys on lisäysjärjestys, ei aakkosjärjestys):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

GUI:ssa vedät kamerasi suodatinkanavat kaavan kaistapaikkoihin, joten mitä tahansa kaavaa voidaan käyttää minkä tahansa kamerasi tukeman kaistamäärityksen kanssa. Tallentamasi mukautetut kaavat lisätään tämän luettelon alle.

**Viisi GUI-käyttöliittymän yksinomaista** kaavaa — joita CLI- ja SDK-luettelo `--indices` ei hyväksy — on toteutettu seuraavasti:

| Vain GUI-käyttöliittymän esiasetus | Kaava (toteutettuna) | Paikat |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a (neljä paikkaa) |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

Kunkin kaavan tarkoitettu kartoitus on esitetty omassa osiossaan tämän sivun alempana (esimerkiksi GARI odottaa x=Green, y=NIR, z=Blue, a=Red). GARI on ainoa kaava Chloros-ohjelmistossa, joka käyttää neljättä paikkaa.

### 2. CLI / SDK `--indices`-nimien laajennus — 22 esiasetusta

`chloros-cli process --indices`-vaihtoehto (ja SDK-parametri `indices`) hyväksyy seuraavat esiasetusten nimet:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**Tuntemattomat hakemistonimet ohitetaan ilman ilmoitusta.** Tämän luettelon ulkopuolella oleva nimi (mukaan lukien viisi pelkästään käyttöliittymässä käytettävää kaavaa `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` sekä kaikki käyttöliittymässä tallentamasi mukautetut kaavat) hylätään vain loki-ilmoituksella — suoritus jatkuu ilman kyseistä indeksiä, ja suoritus itsessään raportoidaan edelleen onnistuneeksi. Ilmoitus tulostetaan seuraavasti:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

Nimiä verrataan välilyönnit poistettuina ilman kirjainkokoa, joten `ndvi`, `NDVI` ja ` NDVI ` ovat sama esiasetus. Esiasetus ohitetaan myös, jos se vaatii kaistan, jota kameran suodatin ei tarjoa.
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
| CVI | `(z/y)*(x/y)` | RGN | Red, Green, NIR |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | NGB | NIR, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### Miten esiasetuksen nimi muunnetaan kaistapaikoiksi

Kun syötät pelkän nimen, kuten `NDVI`, Chlorosin on päätettävä, minkä tiedoston mistäkin kanavasta kukin symboli lukee. Se käyttää tätä taulukkoa, joka yhdistää suodatinkoodin kunkin kanavan taulukon sijaintiin:

| Suodatinkoodi | Kanava → taulukon indeksi |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red` hyväksytään Orange-aliasiksi, myös 0) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

Esiasetuksen **oletussuodatinta** (yllä oleva sarake ”Default filter”) käytetään, kun projekti sisältää kuvia, joissa on kyseinen suodatin. Jos näin ei ole, Chloros skannaa projektissa tosiasiallisesti läsnä olevat suodattimet järjestyksessä `RGN, OCN, NGB, RGB, RE, NIR` ja valitsee ensimmäisen, joka pystyy tarjoamaan kaikki esiasetuksen tarvitsemat kanavat. Jos mikään niistä ei pysty tähän, esiasetusta ei käytetä kyseisessä ajossa. Tämän vuoksi `NDVI`, jota pyydetään pelkästään OCN-aineistossa, tuottaa silti järkevän tuloksen — se sitoutuu OCN:n Orange- ja NIR-asemiin.

LATTICE M3C -mallimerkkijonot sisältävät suodattimen, jonka etuliitteenä on `F` (`LATT-M3C-L41-FRGN`), mutta etuliite poistetaan, kun suodatinkoodi luetaan kuvasta, joten FRGN-kamera tunnistaa sen yllä olevan `RGN`-rivin kautta eikä vaadi erityiskäsittelyä.

### 3. LATTICE-indeksimoottori (`lattice index --preset`, reaaliaikainen indeksilaskuri) — 22 esiasetusta

LATTICE-moottori toimii kohdistetuilla monikaistapinoilla (reaaliaikaiset matriisit tai viedyt monikaistaiset TIFF-tiedostot) ja käyttää kanavien nimiä pienillä kirjaimilla (`red`, `green`, `blue`, `red_edge`, `nir`). Sen esiasetuslista eroaa kahdesta edellä mainitusta:

| Esiasetus | Kaava | Kanavat |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | punainen, lähi-infrapuna |
| GNDVI | `(nir - green) / (nir + green)` | vihreä, lähi-infrapuna |
| BNDVI | `(nir - blue) / (nir + blue)` | sininen, lähi-infrapuna |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | punaisen reuna, lähi-infrapuna |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | sininen, vihreä, lähi-infrapuna |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | punainen, lähi-infrapuna |
| OSAVI | `(nir - red) / (nir + red + 0.16)` | punainen, nir |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | punainen, nir |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | sininen, punainen, nir |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | punainen, NIR |
| CVI | `nir * red / green**2` | punainen, vihreä, NIR |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | punainen, NIR |
| TDVI | `1.5 * (nir - red) / sqrt(nir**2 + red + 0.5)` | punainen, nir |
| LAI | `3.618 * (2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)) - 0.118` | sininen, punainen, NIR |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | punainen, vihreä, sininen |
| NGRDI | `(green - red) / (green + red)` | punainen, vihreä |
| VARI | `(green - red) / (green + red - blue)` | punainen, vihreä, sininen |
| TGI | `green - 0.39*red - 0.61*blue` | punainen, vihreä, sininen |
| EXG | `2*green - red - blue` | punainen, vihreä, sininen |
| CIRE | `(nir / red_edge) - 1` | punainen\_reuna, nir |
| CIGREEN | `(nir / green) - 1` | vihreä, nir |
| NDWI | `(green - nir) / (green + nir)` | vihreä, nir |

Suorita komento `chloros-cli lattice index --list-presets` tulostaaksesi tämän taulukon asennetusta versiosta, ja `--list-gradients` saatavilla olevien värigradienttien tulostamiseksi. Kanavasymbolit ovat kirjainkokoherkkiä ja niiden on vastattava esiasetuksen pienillä kirjaimilla kirjoitettuja nimiä (esim. `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI – Klorofylli-kasvillisuusindeksi

Tämä indeksi on herkkä lehtien klorofyllipitoisuudelle laajalla lehtikaton tiheyden alueella. Se käyttää NIR-, puna- ja vihreäkaistoja.

$$
CVI = NIR * {Red \over Green^{2}}
$$

GUI:ssa sekä CLI- ja SDK-esiasetusluettelossa sen symbolikaava on `(z/y)*(x/y)`, jossa x=Red, y=Green, z=NIR (RGN-kameran kaistat); LATTICE-indeksimoottorin esiasetus on `nir * red / green**2`. Jokainen pinta laskee saman indeksin.

_Lähde: Vincini, M., E. Frazzi ja P. D&#x27;Alessio. &quot;A Broad-Band Leaf Chlorophyll Vegetation Index at the Canopy Scale.&quot; Precision Agriculture 9 (2008): 303–319._

***

## ENDVI – Parannettu normalisoitu kasvillisuusindeksi (Enhanced Normalized Difference Vegetation Index)

Tämä indeksi käyttää sinistä kanavaa NIR- ja vihreän kanavan lisäksi, ja se on suosittu NGB-suodatetuissa kameroissa, joissa sininen kaista korvaa punaisen.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

Toteutus on symbolikaava `((x+y)-(2*z))/((x+y)+(2*z))` — määritä kamerasi NIR- ja Green-kanavat x/y-paikoille ja Blue z-paikalle (NGB-kameran tapauksessa: x=NIR, y=Green, z=Blue).

***

## EVI – parannettu kasvillisuusindeksi

Tämä indeksi kehitettiin alun perin käytettäväksi MODIS-tietojen kanssa NDVI:n parannuksena optimoimalla kasvillisuussignaalia alueilla, joilla lehtipinta-ala-indeksi (LAI) on korkea. Se on hyödyllisin alueilla, joilla LAI on korkea ja NDVI saattaa kyllästyä. Se käyttää sinisen heijastavuusalueen korjaamaan maaperän taustasignaaleja ja vähentämään ilmakehän vaikutuksia, mukaan lukien aerosolien sironta.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

EVI-arvojen tulisi olla välillä 0–1 kasvillisuuspikseleissä. Kirkkaat kohteet, kuten pilvet ja valkoiset rakennukset, sekä tummat kohteet, kuten vesi, voivat aiheuttaa poikkeavia pikseliarvoja EVI-kuvassa. Ennen EVI-kuvan luomista pilvet ja kirkkaat kohteet tulisi poistaa heijastavuuskuvasta ja tarvittaessa asettaa pikseliarvoille kynnysarvo välille 0–1.

_Lähde: Huete, A., et al. ”Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices.” Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 – Metsäpeittoindeksi 1

_Vain GUI — ei saatavilla CLI- tai SDK-esiasetuksena `--indices`._

Tämä indeksi erottaa metsäkatoksen muista kasvillisuustyypeistä käyttämällä monispektrisiä heijastavuuskuvia, jotka sisältävät punaisen reunan kaistan.

$$
FCI1 = Red * RedEdge
$$

Metsäalueilla on alhaisemmat FCI1-arvot, mikä johtuu puiden alhaisemmasta heijastavuudesta ja latvuston sisällä olevista varjoista.

_Lähde: Becker, Sarah J., Craig S.T. Daughtry ja Andrew L. Russ. ”Robust forest cover indices for multispectral images.” Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## FCI2 – Metsäpeittoindeksi 2

_Vain graafinen käyttöliittymä (GUI) — ei saatavilla CLI- tai SDK-esiasetuksena `--indices`._

Tämä indeksi erottaa metsäkatoksen muista kasvillisuustyypeistä käyttämällä monispektrisiä heijastavuuskuvia, joissa ei ole punaista reuna-alueita.

$$
FCI2 = Red * NIR
$$

Metsäalueilla on alhaisemmat FCI2-arvot, mikä johtuu puiden alhaisemmasta heijastavuudesta ja latvuston sisällä olevista varjoista.

_Lähde: Becker, Sarah J., Craig S.T. Daughtry ja Andrew L. Russ. ”Robust forest cover indices for multispectral images.” Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## GEMI – maailmanlaajuinen ympäristöseuranta-indeksi

_Vain GUI — ei saatavilla CLI- tai SDK-muodossa `--indices`-esiasetuksena._

Tätä epälineaarista kasvillisuusindeksiä käytetään maailmanlaajuiseen ympäristöseurantaan satelliittikuvien avulla, ja sen tarkoituksena on korjata ilmakehän vaikutuksia. Se on samanlainen kuin NDVI, mutta vähemmän herkkä ilmakehän vaikutuksille. Paljas maaperä vaikuttaa siihen; siksi sitä ei suositella käytettäväksi alueilla, joilla kasvillisuus on harvaa tai kohtalaisen tiheää.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Missä:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Lähde: Pinty, B., ja M. Verstraete. ”GEMI: a Non-Linear Index to Monitor Global Vegetation From Satellites”. Vegetation 101 (1992): 15–20._

***

## GARI – Green Atmospherically Resistant Index

_Vain GUI – ei saatavilla CLI- tai SDK-esiasetuksena `--indices`._

Tämä indeksi reagoi herkämmin laajalle klorofyllipitoisuusalueelle ja on vähemmän herkkä ilmakehän vaikutuksille kuin NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Gammavakio on painotusfunktio, joka riippuu ilmakehän aerosoliolosuhteista. ENVI käyttää arvoa 1,7, joka on Gitelsonin, Kaufmanin ja Merzylakin suosittelema arvo (1996, sivu 296).

_Lähde: Gitelson, A., Y. Kaufman ja M. Merzylak. ”Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS.” Remote Sensing of Environment 58 (1996): 289–298._

***

## GCI – vihreä klorofylli-indeksi (Green Chlorophyll Index)

Tätä indeksiä käytetään arvioimaan lehtien klorofyllipitoisuutta laajalla kasvilajivalikoimalla.

$$
GCI = {NIR \over Green} - 1
$$

Laaja NIR- ja vihreä aallonpituusalue mahdollistaa paremman klorofyllipitoisuuden ennustamisen ja tarjoaa samalla suuremman herkkyyden sekä paremman signaali-kohinasuhteen.

_Lähde: Gitelson, A., Y. Gritz ja M. Merzlyak. &quot;Lehtien klorofyllipitoisuuden ja spektrisen heijastavuuden väliset suhteet sekä algoritmit korkeampien kasvien lehtien klorofyllipitoisuuden tuhoamattomaan arviointiin.&quot; Journal of Plant Physiology 160 (2003): 271–282._

***

## GLI – Green Leaf Index (vihreiden lehtien indeksi)

Tämä indeksi suunniteltiin alun perin käytettäväksi digitaalisen RGB-kameran kanssa vehnän peittävyyden mittaamiseen, jolloin punaisen, vihreän ja sinisen digitaaliset arvot (DN) vaihtelevat välillä 0–255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI-arvot vaihtelevat välillä -1–+1. Negatiiviset arvot edustavat maaperää ja elottomia kohteita, kun taas positiiviset arvot edustavat vihreitä lehtiä ja varret.

_Lähde: Louhaichi, M., M. Borman ja D. Johnson. ”Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat.” Geocarto International 16, nro 1 (2001): 65–70._

***

## GNDVI – Green Normalized Difference Vegetation Index

Tämä indeksi on samanlainen kuin NDVI, paitsi että se mittaa vihreää spektriä välillä 540–570 nm punaisen spektrin sijaan. Tämä indeksi on herkempi klorofyllipitoisuudelle kuin NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Lähde: Gitelson, A., ja M. Merzlyak. ”Remote Sensing of Chlorophyll Concentration in Higher Plant Leaves.” Advances in Space Research 22 (1998): 689–692._

***

## GOSAVI – vihreälle optimoitu, maaperän mukaan korjattu kasvillisuusindeksi (Green Optimized Soil Adjusted Vegetation Index)

Tämä indeksi suunniteltiin alun perin värillisen infrapunakuvauksen avulla maissin typpitarpeen ennustamiseksi. Se on samanlainen kuin OSAVI, mutta siinä punainen kaista on korvattu vihreällä.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Lähde: Sripada, R., ym. ”Determining In-Season Nitrogen Requirements for Corn Using Aerial Color-Infrared Photography.” Väitöskirja, North Carolina State University, 2005._

***

## GRVI – vihreän osuuden kasvillisuusindeksi (Green Ratio Vegetation Index)

Tämä indeksi reagoi herkästi metsän latvuston fotosynteesinopeuteen, sillä lehtien pigmenttien muutokset vaikuttavat voimakkaasti vihreän ja punaisen heijastavuuteen.

$$
GRVI = {NIR \over Green }
$$

_Lähde: Sripada, R., ym. ”Ilmakuvaus värillisinä infrapunakuvina maissin kasvukauden alkuvaiheen typpitarpeen määrittämiseksi.” Agronomy Journal 98 (2006): 968–977._

***

## GSAVI – Green Soil Adjusted Vegetation Index (vihreän maaperän mukautettu kasvillisuusindeksi)

Tämä indeksi kehitettiin alun perin värillisen infrapunavalokuvauksen avulla maissin typpitarpeen ennustamiseksi. Se on samanlainen kuin SAVI-indeksi, mutta siinä vihreä kaista korvaa punaisen.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Lähde: Sripada, R. ym. ”Maissin kasvukauden aikaisen typpitarpeen määrittäminen ilmakuvausmenetelmällä käyttäen värillistä infrapunakuvausta.” Väitöskirja, North Carolina State University, 2005._

***

## LAI – lehtipinta-ala-indeksi

Tätä indeksiä käytetään lehtipeitteen arviointiin sekä viljelykasvien kasvun ja sadon ennustamiseen. ENVI laskee vihreän LAI:n käyttäen seuraavaa Boegh et al. (2002) esittämää empiiristä kaavaa:

$$
LAI = 3.618 * EVI - 0.118
$$

Jossa EVI:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Korkeat LAI-arvot vaihtelevat tyypillisesti noin 0:sta 3,5:een. Kun kuvassa on pilviä tai muita kirkkaita kohteita, jotka tuottavat ylikylläisiä pikseleitä, LAI-arvot voivat kuitenkin ylittää 3,5:n. Ihannetapauksessa pilvet ja kirkkaat kohteet tulisi poistaa kuvasta ennen LAI-kuvan luomista.

_Lähde: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde ja A. Thomsen. ”Ilmasta kerätyt monispektriset tiedot lehtipinta-ala-indeksin, typpipitoisuuden ja fotosynteettisen tehokkuuden kvantifioimiseksi maataloudessa.” Remote Sensing of Environment 81, nro 2–3 (2002): 179–193._

***

## LCI – lehtien klorofylli-indeksi

_Vain GUI — ei saatavilla CLI- tai SDK-esiasetuksena `--indices`._

Tätä indeksiä käytetään korkeampien kasvien klorofyllipitoisuuden arviointiin; se on herkkä klorofyllin absorptioista johtuvalle heijastavuuden vaihtelulle.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Lähde: Datt, B. ”Remote Sensing of Water Content in Eucalyptus Leaves.” Journal of Plant Physiology 154, nro 1 (1999): 30–36._

***

## MNLI – Modifioitu epälineaarinen indeksi

Tämä indeksi on parannettu versio epälineaarisesta indeksistä (NLI), johon on sisällytetty maaperän mukaan korjattu kasvillisuusindeksi (SAVI) maaperän taustan huomioon ottamiseksi. ENVI käyttää latvuston taustakorjauskerrointa (_L_) arvolla 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Lähde: Yang, Z., P. Willis ja R. Mueller. ”Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy.” Pecora 17 -kaukokartoitussymposiumin julkaisu (2008), Denver, CO._

***

## MSAVI2 – Modified Soil Adjusted Vegetation Index 2

Tämä indeksi on yksinkertaistettu versio Qi:n ym. (1994) ehdottamaa MSAVI-indeksiä, joka on parannettu versio maaperän mukaan korjatusta kasvillisuusindeksistä (SAVI). Se vähentää maaperän aiheuttamaa kohinaa ja lisää kasvillisuussignaalin dynaamista aluetta. MSAVI2 perustuu induktiiviseen menetelmään, jossa ei käytetä vakioarvoa _L_ (kuten SAVI:ssa) terveen kasvillisuuden korostamiseen.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Lähde: Qi, J., A. Chehbouni, A. Huete, Y. Kerr ja S. Sorooshian. ”A Modified Soil Adjusted Vegetation Index.” Remote Sensing of Environment 48 (1994): 119–126._

***

## MSR – Modified Simple Ratio

Tämä indeksi on muunnos yksinkertaisesta NIR/Red-suhteesta, joka on suunniteltu linearisoimaan sen suhde biofysikaalisiin parametreihin, ja se on herkempi kuin NDVI korkeammilla kasvillisuustiheyksillä.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Lähde: Chen, J. ”Evaluation of Vegetation Indices and a Modified Simple Ratio for Boreal Applications.” Canadian Journal of Remote Sensing 22 (1996): 229–242._

***

## NDRE – Normalisoitu ero RedEdge

Tämä indeksi on samanlainen kuin NDVI, mutta siinä verrataan NIR-alueen ja RedEdge-alueen välistä kontrastia Red-alueen sijaan, mikä usein havaitsee kasvillisuuden stressin aikaisemmin.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – Normalisoitu ero kasvillisuusindeksi

Tämä indeksi mittaa terveen, vihreän kasvillisuuden tilaa. Sen normalisoidun eron laskentakaavan ja klorofyllin suurimman absorptio- ja heijastusalueiden yhdistelmä tekee siitä luotettavan monenlaisissa olosuhteissa. Se voi kuitenkin tulla kyllästyneeksi tiheässä kasvillisuudessa, kun LAI-arvo nousee korkeaksi.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Tämän indeksin arvo vaihtelee välillä -1–1. Vihreän kasvillisuuden tyypillinen arvoalue on 0,2–0,8.

_Lähde: Rouse, J., R. Haas, J. Schell ja D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. Kolmas ERTS-symposiumi, NASA (1973): 309–317._

***

## NLI – Epälineaarinen indeksi

Tämä indeksi perustuu oletukseen, että monien kasvillisuusindeksien ja pinnan biofysikaalisten parametrien välinen suhde on epälineaarinen. Se linearisoi suhteet pinnan parametrien kanssa, jotka ovat yleensä epälineaarisia.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Lähde: Goel, N., ja W. Qin. ”Lehtikaton rakenteen vaikutukset erilaisten kasvillisuusindeksien sekä LAI:n ja Fpar:n välisiin suhteisiin: tietokonesimulaatio.” Remote Sensing Reviews 10 (1994): 309–347._

***

## OSAVI – optimoitu maaperän mukaan korjattu kasvillisuusindeksi

Tämä indeksi perustuu maaperän mukaan korjattuun kasvillisuusindeksiin (SAVI). Siinä käytetään latvuston taustakorjauskertoimena vakioarvoa 0,16. Rondeaux (1996) totesi, että tämä arvo tarjoaa suuremman maaperän vaihtelun kuin SAVI matalalla kasvillisuuspeitteellä, samalla kun se osoittaa suurempaa herkkyyttä yli 50 %:n kasvillisuuspeitteelle. Tätä indeksiä käytetään parhaiten alueilla, joilla kasvillisuus on suhteellisen harvaa ja maaperä näkyy latvuston läpi.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Lähde: Rondeaux, G., M. Steven ja F. Baret. ”Optimization of Soil-Adjusted Vegetation Indices.” Remote Sensing of Environment 55 (1996): 95–107._

***

## RDVI – Renormalisoitu kasvillisuusindeksi (Renormalized Difference Vegetation Index)

Tämä indeksi hyödyntää lähi-infrapuna- ja puna-aallonpituuksien välistä eroa yhdessä NDVI-indeksin kanssa terveen kasvillisuuden korostamiseksi. Se ei ole herkkä maaperän vaikutuksille eikä auringon katselukulmalle.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Lähde: Roujean, J., ja F. Breon. ”Estimating PAR Absorbed by Vegetation from Bidirectional Reflectance Measurements.” Remote Sensing of Environment 51 (1995): 375–384._

***

## SAVI – maaperän vaikutuksella korjattu kasvillisuusindeksi

Tämä indeksi on samanlainen kuin NDVI, mutta se vaimentaa maaperän pikselien vaikutuksia. Se käyttää latvuston taustakorjauskerrointa _L_, joka on kasvillisuuden tiheyden funktio ja vaatii usein ennakkotietoa kasvillisuuden määrästä. Huete (1988) ehdottaa optimaalista arvoa _L_=0,5 ensimmäisen asteen maaperän taustavaihteluiden huomioon ottamiseksi. Tätä indeksiä käytetään parhaiten alueilla, joilla kasvillisuus on suhteellisen harvaa ja maaperä näkyy latvuston läpi.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Lähde: Huete, A. ”A Soil-Adjusted Vegetation Index (SAVI).” Remote Sensing of Environment 25 (1988): 295–309._

***

## TDVI – Transformed Difference Vegetation Index

Tämä indeksi on hyödyllinen kasvillisuuspeitteen seurannassa kaupunkiympäristöissä. Se ei kyllästy kuteNDVI ja SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Lähde: Bannari, A., H. Asalhi ja P. Teillet. ”Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping” Teoksessa Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, Volume 5 (2002)._

***

## VARI – näkyvän valon ilmakehän vaikutuksille vastustuskykyinen indeksi

Tämä indeksi perustuu ARVI-indeksiin, ja sitä käytetään arvioimaan kasvillisuuden osuutta kuvassa, jossa herkkyys ilmakehän vaikutuksille on vähäinen.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Lähde: Gitelson, A., ym. ”Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction.” International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – laajadynamiikkainen kasvillisuusindeksi

Tämä indeksi on samanlainen kuin NDVI, mutta siinä käytetään painotuskerrointa (_a_) vähentämään lähi-infrapuna- ja punasignaalien osuuksien välistä eroa NDVI:ssä. WDRVI on erityisen tehokas kuvissa, joissa kasvillisuuden tiheys on kohtalainen tai korkea ja NDVI ylittää arvon 0,6. NDVI:n arvoilla on taipumus tasaantua kasvillisuusosuuden ja lehtipinta-ala-indeksin (LAI) kasvaessa, kun taas WDRVI reagoi herkästi laajempaan kasvillisuusosuuksien alueeseen ja LAI:n muutoksiin.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Painotuskerroin (_a_) voi vaihdella välillä 0,1–0,2. Henebry, Viña ja Gitelson (2004) suosittelevat arvoa 0,2.

_Lähteet_

_Gitelson, A. ”Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.” Journal of Plant Physiology 161, nro 2 (2004): 165–173._

_Henebry, G., A. Viña ja A. Gitelson. ”Laaja-alainen kasvillisuusindeksi ja sen potentiaalinen käyttökelpoisuus aukkoanalyysissä.” Gap Analysis Bulletin 12: 50–56._
