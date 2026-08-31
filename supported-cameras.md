---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/supported-cameras
---

# Tuetut kamerat

Chloros käsittelee kuvamateriaalia kahdesta MAPIR-kameraperheestä **kaikilla alustoilla** (Windows, Linux amd64 ja Linux arm64/Jetson):

* **Survey3** — Survey3W (laajakulma) ja Survey3N (kapeakulma) -kamerat. Syöte: `RAW+JPG`.
* **LATTICE**— M3C- ja M3M-monispektrikameramoduulit. Syöte: `.tif`/`.tiff`-tallenteet. LATTICE-kameroita voidaan myös**ohjata reaaliaikaisesti** Chloros:n kautta — GUI:n Kamerat-välilehden (Windows) tai `chloros-cli lattice` / Python SDK (Windows ja Linux) kautta — mukaan lukien synkronoidut monikamerajärjestelmät. Katso [LATTICE-opas](lattice/).

Käsittelyputki hyväksyy myös `.dng`-tulotiedostoja.

## Survey3

<table data-header-hidden><thead><tr><th width="156">Valmistaja</th><th width="250">Kameramalli</th><th width="138">Suodattimen malli</th><th width="187">Kuvatyyppi</th></tr></thead><tbody><tr><td><strong>Valmistaja</strong></td><td><strong>Kameramalli</strong></td><td><strong>Suodattimen malli</strong></td><td><strong>Kuvatyyppi</strong></td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>OCN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RE</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NIR</td><td>RAW+JPG, JPG</td></tr></tbody></table>## LATTICE

LATTICE-sarja on modulaarinen monispektrikamerajärjestelmä, joka perustuu Sony IMX265 -globaalilukitusanturiin (3,1 MP, 3,45 µm pikselit). Jokainen kamera tallentaa tunnisteensa mallimerkkijonona:

```
<sensor>-<lens>-F<filter>        e.g.  M3C-L41-FRGN,  M3M-L87-F550
```

Chloros näyttää sen etuliitteellä `LATT-` (esimerkiksi `LATT-M3M-L41-F550`), ja mallimerkkijono ohjaa kaikkea jatkokäsittelyä — anturiprofiili, kaistajärjestys ja kalibrointi määritetään automaattisesti; kameraa kohden ei tarvitse määrittää mitään. Objektiivin numero on **vaakasuuntainen näkökenttä asteina**: `L41` = kapea 41°, `L87` = laaja 87°.

Anturimäärityksiä on kaksi:

| Määritys | Anturi      | Suodatintyyppi                           | Kaistat kameraa kohti                                                        |
| ------------- | ----------- | ------------------------------------- | ----------------------------------------------------------------------- |
| **M3C**       | Bayer-väri | Kolmikaistainen                       | 3 spektrikaistaa yhdellä valotuksella                                 |
| **M3M**       | Yksivärinen  | Yksi kapeakaistainen häiriösuodatin | 1 kalibroitu kaista — yhdistämällä useita M3M-kameroita kasvillisuusindeksien laskemiseksi |

### M3C (Bayer) -suodatinvaihtoehdot

| Suodatin | Kaistat (nimi @ keskitaajuus nm / FWHM nm)       |
| ------ | ---------------------------------------- |
| `FRGB` | Blue 475/30 · Green 550/30 · Red 625/30  |
| `FRGN` | Red 660/21 · Green 550/30 · NIR 850/30   |
| `FOCN` | Orange 615/21 · Cyan 490/38 · NIR 808/14 |
| `FNGB` | Blue 475/30 · Green 550/30 · NIR 850/30  |

### M3M (mono) -suodattimien tuoteluettelo — 23 tuotenumeroa

F-luku on tuotenumero; mitattu kaista (joka on leimattu jokaiseen kalibroituun tuotteeseen) on eräkohtainen suodattimen skannaus:

| Tuotenumero    | Keskitaajuus (nm, mitattu) | FWHM reunat (nm) | Leveys (nm) |
| ------ | --------------------- | --------------- | ---------- |
| F385   | 379,4                 | 367–392         | 25         |
| F405   | 403,9                 | 390–417         | 27         |
| F450   | 443,7                 | 430–458         | 28         |
| F485   | 489,7                 | 478–502         | 24         |
| F520   | 519,9                 | 504–536         | 32         |
| F550   | 548,4                 | 531–566         | 35         |
| F590   | 589,0                 | 570–608         | 38         |
| F615   | 623,8                 | 614–634         | 20         |
| F632   | 633,4                 | 616–651         | 35         |
| F650   | 651,1                 | 636–666         | 30         |
| F685   | 686,2                 | 675–698         | 23         |
| F715   | — (nimellinen)           | 706–724         | 18         |
| F725   | 725.2                 | 712–738         | 26         |
| F750   | 746.0                 | 729–763         | 34         |
| F780   | 775.1                 | 754–796         | 42         |
| F808   | 810.3                 | 789–832         | 43         |
| F832   | 826,1                 | 810–843         | 33         |
| F850   | 846,5                 | 828–865         | 37         |
| F880   | — (nimellinen)           | 867–893         | 26         |
| F905   | — (nimellinen)           | 892–920         | 28         |
| F940   | 940,6                 | 923–958         | 35         |
| F950   | 945,1                 | 929–961         | 32         |
| F988 † | 985,3                 | 968–1003        | 35         |

_&quot;Kaistojen reunat on mitattu puolimaksimiarvon mukaisina kokonaisleveyksinä MAPIR:n eräkohtaisista suodatusskannauksista — nämä ovat samat arvot, jotka Chloros merkitsee jokaiseen kalibroituun vientiin.&quot;_ &quot;— (nimellinen)&quot; = eräkohtaista skannausta ei ole vielä tehty; näiden SKU-numeroiden kohdalla ilmoitettu keskipiste on SKU-numero ja leveys on valmistajan ilmoittama arvo.

† &quot;F988-heijastavuus kalibroidaan käyttämällä kuvauspaikalla olevaa heijastavuuspaneelia: kaista sijaitsee DAQ-valosensorin kalibroidun alueen ulkopuolella, joten Chloros käyttää viimeisintä paneelista tallennettua arvoa ja säilyttää sen paneelin tarkistusten välillä.&quot; Katso [Kalibrointikohteet](calibration-targets.md).

Kameran reaaliaikaisesta ohjauksesta, matriiseista, verkkoasetuksista ja radiometrisestä käsittelyketjusta on tietoa [LATTICE-oppaassa](lattice/).
