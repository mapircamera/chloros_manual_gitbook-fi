# Heijastavuuden mittausprosessit

DAQ-valosensori muuntaa radiometriset kuvat heijastavuudeksi. Mittausprosesseja on kaksi erilaista:

1. **Yksi sensori** — yksi DAQ-laite mittaa alaspäin suuntautuvaa säteilyvoimakkuutta samalla kun kamera tallentaa kuvaa, ja Chloros
-ohjelmisto jakaa kameran mittaaman säteilyvoimakkuuden tällä vertailuarvolla.
2. **Kaksi anturia** — kaksi DAQ-anturia, joista toinen tarkkailee taivasta ja toinen kohdetta, tuottavat reaaliaikaisen spektrisen heijastavuuskäyrän ilman kameraa.

## Yksi anturi + kamera (alaspäin suuntautuva vertailuarvo)

DAQ toimii alaspäin suuntautuvan valon anturina (DLS): kamera mittaa ylöspäin suuntautuvaa säteilyvoimakkuutta **L**(W/m²/sr/nm), DAQ mittaa alaspäin suuntautuvaa säteilyvoimakkuutta**E** (W/m²/nm) ja Chloros
laskee heijastavuuden kaistakohtaisesti seuraavasti:

> ρ = π · L / E

DAQ:n lukema on aina **aikaleimaltaan sovitettu valotukseen** — tämän vuoksi DAQ:lla ja kameroilla on yhteinen PTP-synkronoitu kello (katso [DAQ-E-verkko ja aikasynkronointi](ethernet-ptp.md)). Sovita Sunshine-kosinuskorkki ulkokäyttöön ja määritä se oikein; korkin määrittely skaalaa suoraan E:n arvoa (katso [Cap Profiles &amp; Calibrated Range](caps-and-range.md)). Määrällisessä työssä muista laitteen ominaispiirteet: määrällinen säteilyvoimakkuus lasketaan vähintään 15 sekunnin lukemien keskiarvona.

### Reaaliaikainen tallennus

Liitä DAQ-laite kameraan Kamerat-välilehdessä: jokaisen kameran asetuspaneelissa on **Valosensori**-pudotusvalikko, jossa luetellaan kaikki liitetyt DAQ-laitteet (DAQ-U/M/E) Valosensorit-välilehdestä; synkronoidussa kameraryhmässä koko ryhmää koskeva valosensorivalinta välittyy jokaiselle jäsenelle (yksittäiset kamerat voivat silti ohittaa tämän asetuksen). Kun liitäntä on tehty, anturin spektrit syötetään kameran DLS-paikkaan ja heijastavuusarvot jaetaan vastaavalla lukemalla. Järjestelmässä jäsen, jolla on oma anturi, käyttää omaa lukemaansa — sen kaistoille reaaliaikaisessa yhdistelmäkuvassa, sen vientiin ja automaattiseen valotukseen — ja muut jäsenet käyttävät järjestelmän lukemaa.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->


Kaksi hyödyllistä tietoa:

* **Ei DAQ-sidontaa → heijastavuus hylätään, sitä ei väärennetä.** Chloros
hylkää heijastavuustuloksen ja tallentaa ohituksen syyn sen sijaan, että palauttaisi hiljaisesti heikomman tuloksen.
* **Käytetty lukema säilytetään.** Jokaisesta heijastuskehyksestä todellisuudessa sovellettu DAQ-lukema tallennetaan `.daq`-tiedostona kaappausten `daq/`-kansioon, jotta kaappaus voidaan käsitellä uudelleen myöhemmin ([Tallennus ja .daq-muoto](recording.md)). Sen vieressä oleva `.daq.cameras.json`-tiedosto nimeää kuvatut kamerat, ja uudelleenkäsittely yhdistää sen vain kyseisten kameroiden kuviin.

### Tallennettujen kuvien käsittely

Lennon jälkeistä käsittelyä varten tallenna istunnon aikana `.daq`-tiedosto ja säilytä se kuvamateriaalin yhteydessä — käsittelyputki määrittää aikaleimalla täsmätyt alaspäin suuntautuvat kuvat automaattisesti ja hakee puuttuvat tehdaskalibroinnit MAPIR
in pilvipalvelusta ensimmäisellä käyttökerralla. GUI-tallenteet lisätään avoimeen projektiin automaattisesti, kun ne päättyvät.

Heijastavuusviite voidaan valita käsittelyvaiheessa – `--reflectance-source` tai `chloros-cli process`, tai GUI:n Projektiasetuksissa oleva heijastavuuslähteen asetus:

| Arvo | Toiminta |
| --- | --- |
| `auto` (oletus) | Laadunvarmistuksen läpäissyt kehyksessä oleva kalibrointikohde on absoluuttinen vertailuarvo; DAQ-alasäteileminen (ρ = π·L/E) on varavaihtoehto. Kohdetta, joka läpäisee laadunvarmistuksen lipun kanssa (paneeli pudonnut tai kaista epäkunnossa), ei automaattisesti aseteta etusijalle: kohde, kohde+DAQ- ja pelkkä DAQ-ketjut kokeillaan niiden mitatun pistemäärän mukaisessa järjestyksessä testipaneeleilla. |
| `daq` | DAQ-päättävä |
| `target` | Tiukka kehyksen sisäinen kohde; ei DAQ-korvausta |

Katso [Kalibrointikohteet](../calibration-targets.md) kohteen työnkulkujen osalta sekä [LATTICE-luku](../lattice/README.md) sekä [CLI
-viite](../reference/cli-reference.md) koko käsittelyputken osalta. Kun luet vietyjä heijastuspikseleitä, käytä merkittyä asteikkoa (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3
: 65535) — katso [Tulostuskuvamuodot](../output-image-formats.md).

### DAQ:n kalibroidun alueen ulkopuolella olevat kaistat

DAQ:n radiometrisesti kalibroitu alue on ~374–974 nm. Chloros
hylkää DAQ-pohjaisen heijastusarvon kaikilta kameran kaistoilta, joiden spektripaino on alle puolet kyseisen alueen sisällä, ja ilmoittaa ohitussyyn `dls-uncalibrated-band-<nm>`. Toimitettavista tuotemalleista tämä koskee ainoastaan F988-mallia: F988:n heijastavuusarvot kalibroidaan käyttämällä kuvauspaikalla olevaa heijastavuuspaneelia; aallonpituusalue ulottuu DAQ:n valosensorin kalibroidun alueen ulkopuolelle, joten Chloros
käyttää viimeisintä paneelista tallennettua arvoa ja säilyttää sen paneelin mittausten välillä. Jos F988-kameraa käytetään pelkästään DAQ-tilassa, Chloros
hylkää kyseisen kaistan DAQ-pohjaisen heijastavuuden ohitussyynä `dls-uncalibrated-band-988` — paneelipohjainen työnkulku on tuettu menetelmä.

## Kaksi anturia (ympäristövalo + kohde)

Kaksi DAQ-anturia — mikä tahansa pari, millä tahansa siirtoyhteydellä — tuottaa reaaliaikaisen heijastusspektrin ilman kameraa: yksi anturi on suunnattu taivaalle (**Ympäristön valonlähde**), toinen kohteeseen (**Kohteen skanneri**), jChloros
laskee aallonpituuskohtaisesti:

> R(λ) = kohde(λ) / ympäristö(λ)

(nolla, kun ympäristövalon arvo on ≤ 0).

### Käyttöliittymässä

Kun molemmat anturit on liitetty Valoanturit-välilehdessä, avaa anturin lisäysikkuna (ruudukkonäkymän kaavion ruudussa oleva ”+”-painike) ja valitse **Yhdistä ympäristön valo + kohde**. Valitse kaksi anturia Ympäristön valonlähde- ja Kohteen skanneri -pudotusvalikoista ja napsauta Luo. Ryhmä näkyy omana kaavionsa sekä sivupalkin rivinä, jossa on vihreä**REF**-merkki.



<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->
Heijastavuuskaavion (luettelonäkymä) alapuolella oleva reaaliaikainen **kasvillisuusindeksitaulukko** laskee indeksit käyrästä käyttäen kaistakeskuksia sininen 450 / vihreä 550 / punainen 670 / NIR
800 nm. Suhdepohjaiset indeksit, jotka kumoavat absoluuttisen asteikon (NDVI
, GNDVI
, ENDVI, WDRVI
, GRVI
, CVI
, GCI
, MSR) näytetään aina; indeksit, jotka vaativat absoluuttista heijastavuutta (EVI
, SAVI
, OSAVI
, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI
, NLI, MNLI, FCI, GEMI
), näkyvät vain, kun molemmat anturit ovat teho-kalibroituja malleja.

### Näennäinen vs. suhteellinen — nimeämissääntö

Chloros
nimeää kahden anturin tuloksen sen mukaan, mitä anturipari voi tosiasiallisesti ilmoittaa:

| Anturipari | Nimike |
| --- | --- |
| Molemmat anturit kalibroitu — tehdaskalibrointipaketti ladattu | **Näennäinen heijastavuus** |
| Jompikumpi anturi kalibroimaton | **Suhteellinen heijastavuus** |

Kaikki kolme mallia ovat radiometrisiä: kun anturin tehtaan kalibrointipaketti on ladattu, sen spektrit ovat absoluuttisia W/m²/nm, joten kalibroitujen anturien parin suhdeluvut viittaavat absoluuttiseen näennäiseen heijastavuuteen — kuljetus ei sitä määrää. Anturi, joka edelleen lähettää raakalukemia (pakettia ei ole saatavilla), alentaa tuloksen suhteelliseksi käyräksi (spektrin muoto on edelleen pätevä). Molemmilla antureilla tulisi olla oikein määritellyt ylärajat ([Ylärajaprofiilit ja kalibroitu alue](caps-and-range.md)).

### Python
sta

Yhdistetyssä SDK
-rajapinnassa ei ole erillistä kahden anturin kutsua: avaa kaksi istuntoa `chloros_sdk.connect_daq_sensor()`:llä ja laske niiden `latest()`-spektrien suhde itse käyttäen samaa nimeämiskäytäntöä. (Kaksoisanturien tallennustyökalu löytyy myös MAPIR
in sisäisestä suorasta laitteistoliittymästä, ja se on lueteltu täydellisyyden vuoksi [CLI
-viitteessä](../reference/cli-reference.md) — se ei kuulu toimitettuun CLI
:hen; yllä kuvattu GUI-työnkulku on tuettu käyttötapa.)
