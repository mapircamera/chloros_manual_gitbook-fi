# Heijastavuuden mittausprosessit

DAQ-valosensori muuntaa radiometriset kuvat heijastavuusarvoiksi. Mittausprosesseja on kaksi erilaista:

1. **Yksi anturi** — yksi DAQ-anturi mittaa alaspäin suuntautuvaa säteilyvoimakkuutta samalla kun kamera tallentaa kuvaa, ja Chloros jakaa kameran mittaaman säteilyvoimakkuuden tällä vertailuarvolla.
2. **Kaksoisanturi** — kaksi DAQ-anturia, joista toinen seuraa taivasta ja toinen kohdetta, tuottavat reaaliaikaisen spektrisen heijastavuuskäyrän ilman kameraa.

## Yksi anturi + kamera (alaspäin suuntautuva vertailuarvo)

DAQ toimii alaspäin suuntautuvana valoanturina (DLS): kamera mittaa ylöspäin suuntautuvaa säteilyvoimakkuutta **L**(W/m²/sr/nm), DAQ mittaa alaspäin suuntautuvaa säteilyn voimakkuutta**E** (W/m²/nm) ja Chloros laskee heijastavuuden kaistakohtaisesti seuraavasti:

> ρ = π · L / E

DAQ:n lukema on aina **aikaleimaltaan sovitettu valotukseen** — tämän vuoksi DAQ:lla ja kameroilla on yhteinen PTP-säännellyt kello (katso [DAQ-E-verkko ja aikasynkronointi](ethernet-ptp.md)). Aseta Sunshine-kosinuskorkki ulkokäyttöön ja määritä se oikein; korkin määrittely skaalaa suoraan E:n arvoa (katso [Cap Profiles &amp; Calibrated Range](caps-and-range.md)). Määrällisessä työssä muista laitteen ominaispiirteet: määrällinen säteilyvoimakkuus lasketaan vähintään 15 sekunnin lukemien keskiarvona.

### Reaaliaikainen tallennus

Liitä DAQ kameraan Kamerat-välilehdessä: jokaisen kameran asetuspaneelissa on **Valosensori**-pudotusvalikko, jossa luetellaan kaikki liitetyt DAQ-laitteet (DAQ-U/M/E) Valosensorit-välilehdestä; synkronoidussa kameraryhmässä koko ryhmää koskeva valosensorivalinta välittyy jokaiseen jäsenkameraan (yksittäiset kamerat voivat silti ohittaa tämän asetuksen). Kun liitäntä on tehty, anturin spektrit syötetään kameran DLS-paikkaan ja heijastavuusarvot jaetaan vastaavalla lukemalla.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->

Kaksi huomionarvoista seikkaa:

* **Ei sidottua DAQ-laitetta → heijastavuusarvo hylätään, sitä ei väärennetä.** Chloros hylkää heijastavuustuotteen ja tallentaa ohituksen syyn sen sijaan, että palauttaisi hiljaisesti heikomman tuotteen.
* **Käytetty lukema säilytetään.** Jokaisesta heijastavuuskehyksestä todellisuudessa käytetty DAQ-lukema kirjoitetaan `.daq`-sidecarina kuvamateriaalin viereen, jotta tallennetta voidaan käsitellä uudelleen myöhemmin ([Tallennus ja .daq-muoto](recording.md)).

### Tallennettujen kuvien käsittely

Lennon jälkeistä käsittelyä varten tallenna istunnon aikana `.daq` ja säilytä se kuvamateriaalin yhteydessä — käsittelyputki ratkaisee aikaleimalla täsmäävän alaspäin suuntautuvan säteen automaattisesti ja hakee puuttuvat tehdaskalibroinnit MAPIR:n pilvestä ensimmäisellä käyttökerralla. GUI-tallenteet lisätään avoimeen projektiin automaattisesti, kun ne päättyvät.

Heijastavuusviite voidaan valita käsittelyvaiheessa — `--reflectance-source` tai `chloros-cli process`, tai GUI:n Projektiasetukset-kohdassa oleva heijastavuuslähteen asetus:

| Arvo | Toiminta |
| --- | --- |
| `auto` (oletus) | Laadunvarmistuksen läpäissyt kehyksen sisäinen kalibrointikohde on absoluuttinen vertailuarvo; DAQ:n alaspäin suuntautuva säteily (ρ = π·L/E) on varavaihtoehto |
| `daq` | DAQ-määräysvalta |
| `target` | Tiukka kehyssisäinen kohde; ei DAQ-korvausta |

Katso [Kalibrointikohteet](../calibration-targets.md) kohteen työnkulkujen osalta ja [LATTICE-luku](../lattice/README.md) sekä [CLI-viite](../reference/cli-reference.md), josta löytyy koko käsittelyputki. Kun luet vietyjä heijastuspikseleitä, käytä merkittyä asteikkoa (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3: 65535) — katso [Tulostuskuvamuodot](../output-image-formats.md).

### DAQ:n kalibroidun alueen ulkopuolella olevat kaistat

DAQ:n radiometrisesti kalibroitu alue on ~374–974 nm. Chloros hylkää DAQ-pohjaisen heijastavuuden kaikilta kameran aallonpituusalueilta, joiden spektripaino on alle puolet kyseisen alueen sisällä, ja ilmoittaa ohituksen syyksi `dls-uncalibrated-band-<nm>`. Toimitettavista tuotetunnuksista tämä koskee vain F988-mallia: F988:n heijastavuus on kalibroitu käyttämällä kuvauspaikalla olevaa heijastavuuspaneelia; aallonpituusalue ulottuu DAQ:n valosensorin kalibroidun alueen ulkopuolelle, joten Chloros käyttää viimeisintä paneelista tallennettua arvoa ja säilyttää sen paneelin mittausten välillä. Jos F988-kameraa käytetään pelkästään DAQ-tilassa, Chloros hylkää kyseisen kaistan DAQ-pohjaisen heijastavuuden ohitussyynä `dls-uncalibrated-band-988` — paneeliprosessi on tuettu menetelmä.

## Kaksi anturia (ympäristövalo + kohde)

Kaksi DAQ-anturia — mikä tahansa pari, millä tahansa kuljetusvälineellä — tuottaa reaaliaikaisen heijastusspektrin ilman kameraa: toinen anturi on suunnattu taivaalle (**ympäristön valonlähde**), toinen kohteeseen (**kohteen skanneri**), ja Chloros laskee aallonpituuskohtaisesti:

> R(λ) = kohde(λ) / ympäristö(λ)

(nolla, kun ympäristövalon arvo on ≤ 0).

### Käyttöliittymässä

Kun molemmat anturit on liitetty Valoanturit-välilehdessä, avaa anturin lisäyspaneeli (ruudukkonäkymän kaavion ruudussa oleva ”+”-painike) ja valitse **Yhdistä ympäristön valo + kohde**. Valitse kaksi anturia valinta-ruuduista ”Ambient Light Source” ja ”Object Scanner” ja napsauta ”Create”. Ryhmä näkyy omana kaavionsa sekä sivupalkin rivinä, jossa on vihreä**REF**-merkki.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->

Heijastavuuskaavion (luettelonäkymä) alapuolella oleva reaaliaikainen **kasvillisuusindeksitaulukko** laskee indeksit käyrästä käyttäen kaistakeskuksia sininen 450 / vihreä 550 / punainen 670 / NIR 800 nm. Suhteisiin perustuvat indeksit, jotka kumoavat absoluuttisen asteikon (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR) näytetään aina; indeksit, jotka vaativat absoluuttista heijastavuutta (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) näkyvät vain, kun molemmat anturit ovat teho-kalibroituja malleja.

### Näennäinen vs. suhteellinen — nimityssääntö

Chloros nimeää kaksoisanturin tuloksen sen mukaan, mitä anturipari voi tosiasiallisesti ilmoittaa:

| Anturipari | Nimike |
| --- | --- |
| Molemmat anturit kalibroitu — tehdaskalibrointipaketti ladattu | **Näennäinen heijastavuus** |
| Jompikumpi anturi kalibroimaton | **Suhteellinen heijastavuus** |

Kaikki kolme mallia ovat radiometrisiä: kun anturin tehtaan kalibrointipaketti on ladattu, sen spektrit ovat absoluuttisia W/m²/nm, joten kalibroitujen anturien parin suhde määrittää absoluuttisen näennäisen heijastavuuden — siirto ei vaikuta siihen. Anturi, joka edelleen lähettää raakalukemia (pakettia ei ole saatavilla), alentaa tuloksen suhteelliseksi käyräksi (spektrin muoto on edelleen pätevä). Molemmilla antureilla tulisi olla oikein määritellyt ylärajat ([Ylärajaprofiilit ja kalibroitu alue](caps-and-range.md)).

### Lähde: Python

Yhdistetyssä SDK-pinnassa ei ole erillistä kaksoisanturikutsua: avaa kaksi istuntoa `chloros_sdk.connect_daq_sensor()`:llä ja laske itse niiden `latest()`-spektrien suhde käyttäen samaa nimeämiskäytäntöä. (Kaksisensorinen tallennustyökalu löytyy myös MAPIR:n sisäiseltä suorahardware-pinnalta, joka on lueteltu [CLI-viitteessä](../reference/cli-reference.md) täydellisyyden vuoksi — se ei kuulu toimitettuun CLI-laitteeseen; yllä esitetty graafisen käyttöliittymän työnkulku on tuettu reaaliaikainen menetelmä.)
