---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Monispektriset indeksikaavat

Seuraavissa indeksikaavoissa käytetään yhdistelmää Survey3-suodattimen keskimääräisistä läpäisyalueista:

<table><thead><tr><th align="center">Survey3-suodattimen väri</th><th width="196.199951171875" align="center">Survey3-suodattimen nimi</th><th width="159.800048828125" align="center">Läpäisyalue (FWHM)</th><th align="center">Keskimääräinen läpäisy</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB – Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN– Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>Kun näitä kaavoja käytetään, nimi voi päättyä merkkeihin &quot;\_1&quot; tai &quot;\_2&quot;, mikä vastaa sitä, kumpaa NIR-suodatinta, joko NIR1:ää tai NIR2:ää, on käytetty.

***

## EVI - Parannettu kasvillisuusindeksi

Tämä indeksi kehitettiin alun perin käytettäväksi MODIS-tietojen kanssa parannuksena NDVI-indeksiin optimoimalla kasvillisuussignaalia alueilla, joilla lehtipinta-indeksi (LAI) on korkea. Se on hyödyllisin alueilla, joilla LAI-arvo on korkea ja joissa NDVI-arvo saattaa kyllästyä. Se käyttää sinisen heijastavuusalueen korjaamaan maaperän taustasignaaleja ja vähentämään ilmakehän vaikutuksia, mukaan lukien aerosolien sironta.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

EVI-arvojen tulisi olla välillä 0–1 kasvillisuuspikseleille. Kirkkaat kohteet, kuten pilvet ja valkoiset rakennukset, sekä tummat kohteet, kuten vesi, voivat aiheuttaa poikkeavia pikseliarvoja EVI-kuvassa. Ennen EVI-kuvan luomista sinun tulisi peittää pilvet ja kirkkaat kohteet heijastavuuskuvasta ja valinnaisesti asettaa pikseliarvojen kynnysarvoksi 0–1.

_Viite: Huete, A., et al. &quot;Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices.&quot; Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 - Metsäpeittoindeksi 1

Tämä indeksi erottaa metsän latvuston muista kasvillisuustyypeistä käyttämällä monispektrisiä heijastavuuskuvia, jotka sisältävät punaisen reunan kaistan.

$$
FCI1 = Red * RedEdge
$$

Metsäalueilla on alhaisemmat FCI1-arvot, mikä johtuu puiden alhaisemmasta heijastavuudesta ja latvuston sisällä olevista varjoista.

_Lähde: Becker, Sarah J., Craig S.T. Daughtry ja Andrew L. Russ. &quot;Robust forest cover indices for multispectral images.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 - Metsäpeittoindeksi 2

Tämä indeksi erottaa metsän latvuston muista kasvillisuustyypeistä käyttämällä monispektrisiä heijastavuuskuvia, jotka eivät sisällä punaista reuna-alueita.

$$
FCI2 = Red * NIR
$$

Metsäalueilla on alhaisemmat FCI2-arvot puiden alhaisemman heijastavuuden ja latvuston varjojen vuoksi.

_Lähde: Becker, Sarah J., Craig S.T. Daughtry ja Andrew L. Russ. &quot;Robust forest cover indices for multispectral images.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI - Globaali ympäristöseurantaindeksi

Tätä epälineaarista kasvillisuusindeksiä käytetään satelliittikuvien pohjalta tapahtuvaan globaaliin ympäristöseurantaan, ja sen tarkoituksena on korjata ilmakehän vaikutukset. Se on samanlainen kuin NDVI, mutta vähemmän herkkä ilmakehän vaikutuksille. Paljas maaperä vaikuttaa siihen, joten sen käyttöä ei suositella alueilla, joilla kasvillisuus on harvaa tai kohtalaisen tiheää.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Missä:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Viite: Pinty, B., ja M. Verstraete. GEMI: epälineaarinen indeksi maailmanlaajuisen kasvillisuuden seurantaan satelliittien avulla. Vegetation 101 (1992): 15–20._

***

## GARI – Green Ilmakehän vaikutuksille vastustuskykyinen indeksi

Tämä indeksi on herkempi laajalle klorofyllipitoisuuksien alueelle ja vähemmän herkkä ilmakehän vaikutuksille kuin NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Gammavakiota käytetään painotusfunktiona, joka riippuu ilmakehän aerosoliolosuhteista. ENVI käyttää arvoa 1,7, joka on Gitelsonin, Kaufmanin ja Merzylakin (1996, sivu 296) suosittelema arvo.

_Viite: Gitelson, A., Y. Kaufman ja M. Merzylak. &quot;Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS.&quot; Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI – Green Klorofylli-indeksi

Tätä indeksiä käytetään arvioimaan lehtien klorofyllipitoisuutta laajalla kasvilajivalikoimalla.

$$
GCI = {NIR \over Green} - 1
$$

Laaja NIR- ja vihreä aallonpituusalue mahdollistaa paremman klorofyllipitoisuuden ennustamisen ja tarjoaa samalla suuremman herkkyyden ja paremman signaali-kohinasuhteen.

_Lähde: Gitelson, A., Y. Gritz ja M. Merzlyak. &quot;Relationships Between Leaf Chlorophyll Content and Spectral Reflectance and Algorithms for Non-Destructive Chlorophyll Assessment in Higher Plant Leaves.&quot; Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI – Green-lehtindeksi

Tämä indeksi on alun perin suunniteltu käytettäväksi digitaalisen RGB-kameran kanssa vehnän peittävyyden mittaamiseen, jossa punaisen, vihreän ja sinisen digitaaliset arvot (DN) vaihtelevat välillä 0–255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI-arvot vaihtelevat välillä -1–+1. Negatiiviset arvot edustavat maaperää ja elottomia kohteita, kun taas positiiviset arvot edustavat vihreitä lehtiä ja varret.

_Lähde: Louhaichi, M., M. Borman ja D. Johnson. &quot;Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat.&quot; Geocarto International 16, nro 1 (2001): 65–70._

***

## GNDVI – Green Normalisoitu kasvillisuusindeksi

Tämä indeksi on samanlainen kuin NDVI, paitsi että se mittaa vihreää spektriä 540–570 nm:n alueella punaisen spektrin sijaan. Tämä indeksi on herkempi klorofyllipitoisuudelle kuin NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Viite: Gitelson, A., ja M. Merzlyak. &quot;Remote Sensing of Chlorophyll Concentration in Higher Plant Leaves.&quot; Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green Optimoitu maaperän mukautettu kasvillisuusindeksi

Tämä indeksi suunniteltiin alun perin värillisen infrapunavalokuvauksen avulla maissin typpitarpeen ennustamiseksi. Se on samanlainen kuin OSAVI, mutta siinä vihreä kaista korvaa punaisen.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Lähde: Sripada, R., et al. &quot;Determining In-Season Nitrogen Requirements for Corn Using Aerial Color-Infrared Photography.&quot; Väitöskirja, North Carolina State University, 2005._

***

## GRVI - Green-suhde kasvillisuusindeksi

Tämä indeksi reagoi herkästi metsän latvuston fotosynteesinopeuteen, sillä lehtien pigmenttien muutokset vaikuttavat voimakkaasti vihreän ja punaisen heijastavuuteen.

$$
GRVI = {NIR \over Green }
$$

_Viite: Sripada, R., et al. &quot;Ilmakuvaus värillisinä infrapunakuvina maissin varhaisen kasvukauden typpitarpeen määrittämiseksi.&quot; Agronomy Journal 98 (2006): 968–977._

***

## GSAVI – Green Maaperän mukaan korjattu kasvillisuusindeksi

Tämä indeksi suunniteltiin alun perin väri-infrapunavalokuvauksen avulla maissin typpitarpeen ennustamiseksi. Se on samanlainen kuin SAVI, mutta siinä vihreä kaista korvaa punaisen.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Lähde: Sripada, R., et al. &quot;Determining In-Season Nitrogen Requirements for Corn Using Aerial Color-Infrared Photography.&quot; Väitöskirja, North Carolina State University, 2005._

***

## LAI – Lehtipinta-indeksi

Tätä indeksiä käytetään lehtipeitteen arviointiin sekä sadon kasvun ja tuoton ennustamiseen. ENVI laskee vihreän LAI:n käyttäen seuraavaa Boegh et al. (2002) esittämää empiiristä kaavaa:

$$
LAI = 3.618 * EVI - 0.118
$$

Jossa EVI on:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Korkeat LAI-arvot vaihtelevat tyypillisesti noin 0:sta 3,5:een. Jos kuvassa on kuitenkin pilviä tai muita kirkkaita kohteita, jotka tuottavat kylläisiä pikseleitä, LAI-arvot voivat ylittää 3,5:n. Ihannetapauksessa pilvet ja kirkkaat kohteet tulisi peittää kuvasta ennen LAI-kuvan luomista.

_Viite: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde ja A. Thomsen. &quot;Airborne Multi-spectral Data for Quantifying Leaf Area Index, Nitrogen Concentration and Photosynthetic Efficiency in Agriculture.&quot; Remote Sensing of Environment 81, nro 2-3 (2002): 179-193._

***

## LCI - Lehtien klorofylli-indeksi

Tätä indeksiä käytetään korkeampien kasvien klorofyllipitoisuuden arviointiin, ja se on herkkä klorofyllin absorptioista johtuvalle heijastavuuden vaihtelulle.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Viite: Datt, B. &quot;Eukalyptuslehtien vesipitoisuuden kaukokartoitus.&quot; Journal of Plant Physiology 154, nro 1 (1999): 30–36._

***

## MNLI - Modifioitu epälineaarinen indeksi

Tämä indeksi on parannus epälineaariseen indeksiin (NLI), joka sisällyttää maaperän taustan huomioon ottavan maaperän mukautetun kasvillisuusindeksin (SAVI). ENVI käyttää latvuston taustan korjauskerrointa (_L_) arvolla 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Viite: Yang, Z., P. Willis ja R. Mueller. &quot;Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy.&quot; Proceedings of the Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 – Modified Soil Adjusted Vegetation Index 2

Tämä indeksi on yksinkertaisempi versio Qi et al. (1994) ehdottamasta MSAVI-indeksistä, joka parantaa Soil Adjusted Vegetation Index -indeksiä (SAVI). Se vähentää maaperän kohinaa ja lisää kasvillisuussignaalin dynaamista aluetta. MSAVI2 perustuu induktiiviseen menetelmään, joka ei käytä vakioarvoa _L_ (kuten SAVI) terveen kasvillisuuden korostamiseen.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Viite: Qi, J., A. Chehbouni, A. Huete, Y. Kerr ja S. Sorooshian. &quot;A Modified Soil Adjusted Vegetation Index.&quot; Remote Sensing of Environment 48 (1994): 119-126._

***

## NDRE – Normalisoitu ero RedEdge

Tämä indeksi on samanlainen kuin NDVI, mutta siinä verrataan NIR:n ja RedEdge:n välistä kontrastia Red:n sijaan, mikä usein havaitsee kasvillisuuden stressin aikaisemmin.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – Normalisoitu kasvillisuusindeksi

Tämä indeksi mittaa terveen, vihreän kasvillisuuden määrää. Sen normalisoidun eron kaavan ja klorofyllin suurimman absorptio- ja heijastusalueiden yhdistelmä tekee siitä luotettavan monenlaisissa olosuhteissa. Se voi kuitenkin kyllästyä tiheässä kasvillisuudessa, kun LAI-arvo nousee korkeaksi.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Tämän indeksin arvo vaihtelee välillä -1–1. Vihreän kasvillisuuden yleinen arvoalue on 0,2–0,8.

_Lähde: Rouse, J., R. Haas, J. Schell ja D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. Third ERTS Symposium, NASA (1973): 309–317._

***

## NLI – Epälineaarinen indeksi

Tämä indeksi olettaa, että monien kasvillisuusindeksien ja pinnan biofysikaalisten parametrien välinen suhde on epälineaarinen. Se linearisoi suhteet pinnan parametrien kanssa, jotka ovat yleensä epälineaarisia.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Viite: Goel, N. ja W. Qin. &quot;Influences of Canopy Architecture on Relationships Between Various Vegetation Indices and LAI and Fpar: A Computer Simulation.&quot; Remote Sensing Reviews 10 (1994): 309–347._

***

## OSAVI – Optimoitu maaperän mukaan korjattu kasvillisuusindeksi

Tämä indeksi perustuu maaperän mukaan korjattuun kasvillisuusindeksiin (SAVI). Se käyttää latvuston taustakorjauskertoimena vakioarvoa 0,16. Rondeaux (1996) totesi, että tämä arvo tarjoaa suuremman maaperän vaihtelun kuin SAVI vähäisen kasvillisuuden peitossa, samalla kun se osoittaa suurempaa herkkyyttä yli 50 %:n kasvillisuuspeitolle. Tätä indeksiä käytetään parhaiten alueilla, joilla kasvillisuus on suhteellisen harvaa ja maaperä näkyy latvuston läpi.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Viite: Rondeaux, G., M. Steven ja F. Baret. &quot;Optimization of Soil-Adjusted Vegetation Indices.&quot; Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI – Renormalisoitu kasvillisuusindeksi

Tämä indeksi käyttää lähi-infrapuna- ja punaisen aallonpituuksien välistä eroa yhdessä NDVI:n kanssa terveen kasvillisuuden korostamiseen. Se on herkkä maaperän ja auringon katselukulman vaikutuksille.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Viite: Roujean, J., ja F. Breon. &quot;Estimating PAR Absorbed by Vegetation from Bidirectional Reflectance Measurements.&quot; Remote Sensing of Environment 51 (1995): 375–384._

***

## SAVI – Maaperän mukaan korjattu kasvillisuusindeksi

Tämä indeksi on samanlainen kuin NDVI, mutta se vaimentaa maaperän pikselien vaikutuksia. Se käyttää latvuston taustan korjauskerrointa _L_, joka on kasvillisuuden tiheyden funktio ja vaatii usein ennakkotietoa kasvillisuuden määrästä. Huete (1988) ehdottaa optimaalista arvoa _L_=0,5 ensimmäisen asteen maaperän taustavaihteluiden huomioon ottamiseksi. Tätä indeksiä käytetään parhaiten alueilla, joilla kasvillisuus on suhteellisen harvaa ja maaperä näkyy latvuston läpi.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Viite: Huete, A. &quot;A Soil-Adjusted Vegetation Index (SAVI).&quot; Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI – Transformed Difference Vegetation Index

Tämä indeksi on hyödyllinen kasvillisuuspeitteen seurannassa kaupunkiympäristöissä. Se ei kyllästy kuten NDVI ja SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Viite: Bannari, A., H. Asalhi ja P. Teillet. &quot;Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping&quot; Teoksessa Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, Volume 5 (2002)._

***

## VARI – Näkyvä ilmakehän vaikutuksille vastustuskykyinen indeksi

Tämä indeksi perustuu ARVI-indeksiin, ja sitä käytetään arvioimaan kasvillisuuden osuutta kuvassa, jossa herkkyys ilmakehän vaikutuksille on vähäinen.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Viite: Gitelson, A., et al. &quot;Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – Laaja dynaaminen alue -kasvillisuusindeksi

Tämä indeksi on samanlainen kuin NDVI, mutta siinä käytetään painotuskerrointa (_a_) vähentämään lähi-infrapuna- ja punasignaalien osuuksien välistä eroa NDVI:ssä. WDRVI on erityisen tehokas kohtauksissa, joissa kasvillisuuden tiheys on kohtalainen tai korkea, kun NDVI ylittää arvon 0.6. NDVI:n arvo tasoittuu yleensä, kun kasvillisuusosuus ja lehtipinta-indeksi (LAI) kasvavat, kun taas WDRVI on herkempi laajemmalle kasvillisuusosuuksien alueelle ja muutoksille LAI:ssä.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Painotuskerroin (_a_) voi vaihdella välillä 0,1–0,2. Henebry, Viña ja Gitelson (2004) suosittelevat arvoa 0,2.

_Viitteet_

_Gitelson, A. &quot;Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.&quot; Journal of Plant Physiology 161, nro 2 (2004): 165–173._

_Henebry, G., A. Viña ja A. Gitelson. &quot;The Wide Dynamic Range Vegetation Index and its Potential Utility for Gap Analysis.&quot; Gap Analysis Bulletin 12: 50–56._
