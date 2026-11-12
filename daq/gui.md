# Chloros-ohjelman DAQ-välilehti

DAQ-välilehti — joka on nimetty **Valosensorit** Chloros-sivupalkissa — on [DAQ-U-, DAQ-M- ja DAQ-E-valosensorien](README.md) reaaliaikainen ohjauspaneeli: liitä anturit minkä tahansa siirtoprotokollan kautta, seuraa kalibroituja spektrejä reaaliajassa, laske anturiparin reaaliaikaista heijastavuutta ja tallenna `.daq`-tiedostoja suoraan projektiisi.

Välilehti tulee käyttöön, kun Chloros-taustapalvelu on käynnistynyt. Välilehden kaaviot saavat tietonsa Chlorosin DAQ-palvelusta reaaliaikaisen yhteyden kautta, joka muodostuu uudelleen automaattisesti (2–10 sekunnin viiveellä), jos yhteys katkeaa; kun palveluun ei saada yhteyttä, anturin Tila-rivillä lukee **No Server**.

Asettelu koostuu **anturien sivupalkista**(yksi rivi kutakin kytkettyä anturia kohti) sekä**kaavioalueesta** (yksi kaaviolaatta kutakin anturia tai ryhmää kohti).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Anturin liittäminen

Napsauta sivupalkin yläosassa olevaa **Liitä anturi** -painiketta. Liitäntävalintaikkuna avautuu pääalueelle (tai päällekkäisenä ikkunana, kun lisätään toista anturia – tällöin näkyviin tulee Peruuta-painike).

| Ohjaus | Toiminto |
| --- | --- |
| **Laitetyyppi** | `DAQ-U (USB)` (oletus), `DAQ-M (Bluetooth)` tai `DAQ-E (Ethernet)`. Vaihtaminen käynnistää skannauksen uudelleen valitulle siirtotavalle. |
| **Portti / BLE-laite / Isäntänimi / IP** | Luettelee löydetyt laitteet muodossa `device - description`; ensimmäinen anturiksi tunnistettu merkintä valitaan automaattisesti. Skannauksen aikana näytössä näkyy `Scanning...` (USB), `Scanning (N)...` 8 sekunnin lähtölaskennalla (BLE) tai `Discovering ethernet sensors (N)...` 5 sekunnin lähtölaskennalla (Ethernet). Tyhjät tulokset näkyvät muodossa `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Päivitä** | Skannaa valitun siirtotavan uudelleen välittömästi (pois käytöstä BLE-/Ethernet-skannauksen aikana). |
| **Yhdistä anturi** | Aktivoituu, kun laite on valittu; nimeää uudelleen `Connecting...`:ksi yhteyden muodostamisen aikana. |

Haku toimii vain **silloin, kun yhteysvalintaikkuna on näytöllä**, ja se toistuu 15 sekunnin välein vain valitun siirtotavan osalta — pelkkä välilehden avaaminen ei käynnistä skannausta. Epäonnistumisen yhteydessä valintaikkunassa näkyy: *"Yhteyden muodostaminen epäonnistui. Kokeile irrottaa ja kytkeä anturi uudelleen, ja napsauta sitten Yhdistä uudelleen."*

Sivupalkki avautuu automaattisesti, kun ensimmäinen anturi muodostaa yhteyden.

{% hint style="info" %}
**DAQ-E ei näy?**DAQ-E:ssä ei ole tilavaloa — tarkista PoE-/linkkivalo kytkimessä tai injektoriportissa, johon se on kytketty, ja odota muutama sekunti käynnistyksen jälkeen, jotta laite ehtii käynnistyä. Chloros-laitteen on oltava samassa lähetysalueessa (mDNS ei läpäise reitittimiä). Asennusohjelman**Chloros-taustapalvelimen (UDP-In)** palomuurisääntö sallii mDNS:n (UDP 5353), DAQ-E-datan (UDP 5002) ja PTP:n (UDP 319/320) läpikulun; kolmannen osapuolen palomuurissa on sallittava samat portit. Kaksi samassa lähiverkossa olevaa DAQ-E-yksikköä tunnistetaan erikseen, kumpikin omalla `daq-e-<id>.local`-isäntänimellään.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Laitetyyppi-valikossa on vaihtoehdot DAQ-U (USB), DAQ-M (Bluetooth) ja DAQ-E (Ethernet)</figcaption></figure>

***

## Anturien sivupalkki

Jokaiselle kytketylle anturille varataan oma rivi (sekä yksi rivi kutakin Ambient+Object-ryhmää kohti). Rivien järjestystä voi muuttaa vetämällä, ja niiden järjestys vaikuttaa myös kaavion ruutujen järjestykseen. Napsauttamalla riviä voit asettaa kyseisen anturin tai ryhmän aktiiviseksi kaavioksi luettelonäkymässä.

| Elementti | Merkitys |
| --- | --- |
| Värillinen vasen reuna | Anturin kaavion väri. |
| Siirto-merkki | `DAQ-U` / `DAQ-M` / `DAQ-E` tai vihreä `REF`-merkki Ambient+Object-heijastusryhmälle. |
| Laitteen nimi | Oletusarvo on anturin sarjanumero (sen vakiintunut tunniste kalibrointia, `.daq`-tiedostonimiä ja tuonnin täsmäytystä varten); mukautetut nimet säilyvät projektikohtaisesti. |
| **Kalibroitu**-pilleri (vihreä) | Näkyy, kun anturin tehdaskalibrointipaketti on ladattu, eli spektrit ovat todellisia W/m²/nm. |
| **Päivitys saatavilla** -kuvake (keltainen, vain DAQ-E) | Käytössä oleva laiteohjelmisto on vanhempi kuin tämän Chloros-rakennuksen mukana toimitettu kuva. Päivityksen aikana se näyttää reaaliaikaista edistymistä (`Flashing… N%`, `Restarting sensor…`, sitten `Updated X → Y` tai `Failed`). |
| Silmä | Vaihtaa tämän anturin näkyvyyttä kaaviossa. |
| Hammasratas | Avaa antikohtaisen asetusikkunan (alla). |
| ✕ (punainen) | Irrottaa anturin tai poistaa Ambient+Object-ryhmän. |

Rivien yläpuolella on kaksi painiketta:

* **Yhdistä anturi** — avaa yhdistämisvalintaikkunan (muuttuu nimeksi `Connecting...`, kun toiminto on käynnissä).
* **Tallenna kaikki / Lopeta kaikki**— käynnistää tai lopettaa `.daq`-tallennuksen**kaikilla**kytketyillä antureilla. Vaatii vähintään yhden anturin**ja avoimen projektin** (työkaluvinkki: ”Avaa projekti tallennusta varten”); painike muuttuu punaiseksi, kun tallennus on käynnissä.

Tyhjänä painikkeessa lukee ”Ei kytkettyjä antureita”.

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Anturikohtaiset asetukset (hammasratas-modaali)

Avaa hammasrataskuvakkeella anturirivillä. Sisältö järjestyksessä:

* **Tietorivit** — Laitetyyppi (DAQ-U/M/E), Yhteys (`Serial (USB)` / `Bluetooth` / `Ethernet`), portti (COM-portti, BLE-osoite tai isäntä) ja sarjanumero.
* **Kalibrointiraportti: Lataa** — hakee tämän laitteen NIST-jäljitettävän kalibrointitodistuksen (PDF) ja avaa sen PDF-katseluohjelmassasi. Käytettävissä, kun sarjanumero on tiedossa; sertifikaatti tallennetaan välimuistiin ensimmäisen yhteyden muodostuksen yhteydessä.
* **Laitteen nimi** — napsauta kynää nimen muuttamiseksi; säilyy projektikohtaisesti.
* **Kaavion viivan väri** — värinäyte; säilyy projektikohtaisesti.
* **Integraatioaika (ms)**— liukusäädin + luku,**1–500 ms**, oletusarvo**32 ms**. Poissa käytöstä, kun AE on päällä.
* **Kehysten keskiarvo**— liukusäädin + luku,**1–50 kehystä**, oletusarvo**20**.
* **AE: PÄÄLLÄ/POIS**— automaattisen valotuksen kytkin;**oletusarvoisesti PÄÄLLÄ** yhteyden muodostamisen yhteydessä. Kytke se pois päältä, jos haluat asettaa integrointiajan manuaalisesti.
* **Lopeta suoratoisto / Aloita suoratoisto** — keskeytä tai jatka suoratoistoa.
* **Tallenna / Lopeta tallennus** — `.daq`-tallennus anturikohtaisesti (vaatii avoimen projektin).
* **Cap** — cap-korjausprofiili (seuraava osio).
* **Reaaliaikaiset tietorivit** — Integraatioaika (ms), FPS, Näytteet, Tallennus (punainen `REC` tai `Off`) ja Tila (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Vain DAQ-E: verkko-, laiteohjelmisto- ja PTP-rivit

* **Isäntänimi / IP** — laitteen nykyinen osoite.
* **Laiteohjelmisto**— reaaliaikainen laiteohjelmistoversio sekä toimintosolu:**Päivitä versioon \<version\>

** -painike tulee näkyviin, kun tämä Chloros-versio sisältää uudemman DAQ-E-laiteohjelmistokuvan. Päivitys asennetaan verkon kautta noin 30 sekunnissa; anturi käynnistyy uudelleen ja muodostaa yhteyden automaattisesti, ja keskeytynyt siirto jättää nykyisen laiteohjelmiston ennallaan. Edistymistiedot näkyvät reaaliaikaisesti (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`), ja kentässä lukee `Up to date`, kun versio on ajantasainen.
* **PTP-synkronointi** — reaaliaikainen PTP-tila (palaa tilaan `unknown`). DAQ-E-laiteohjelmistoversio v1.2.0+ toimii IEEE 1588 PTPv2 -standardissa pelkästään orjakellona; Chloros-isäntäkoneen taustapalvelin toimii PTP-grandmasterina, ja jokainen LAN-verkossa oleva DAQ-E- ja LATTICE-kamera toimii sen orjana domainissa 0, jolloin aikaleimat pysyvät noin 1 ms:n tarkkuudella.
* **Verkko**— laitteen käyttämä osoite ja sen hankintatapa (DHCP, link-local tai staattinen) sekä tallennettu asetus, joka otetaan käyttöön uudelleenkäynnistyksen jälkeen.**Muuta…**avaa lomakkeen:**Automaattinen (DHCP)**tai**Staattinen IP**, jossa on IP-osoite, aliverkon peite ja valinnainen yhdyskäytävä.**Käytä ja käynnistä uudelleen** tallentaa asetukset, käynnistää anturin uudelleen ja muodostaa yhteyden uudelleen. Tämän tietokoneen verkon ulkopuolella oleva staattinen osoite hylätään, koska Chloros ei päässyt anturiin käynnistyksen jälkeen; tällöin avautuu valintaruutu, jolla asetus voidaan sallia kuitenkin, jos tietokoneelle annetaan uusi osoite tai se reititetään kyseiseen verkkoon. **Lähetys käynnistyksen yhteydessä**määrittää, aloittaako anturi lähetyksen heti virran kytkemisen jälkeen, ennen kuin mikään ohjelmisto on muodostanut yhteyttä.**Käynnistä anturi uudelleen** käynnistää sen uudelleen ja muodostaa yhteyden uudelleen nykyisillä asetuksilla. Uudelleenkäynnistys hylätään, kun anturi tallentaa. Uutta versiossa 1.3.0.

Ambient+Object-ryhmän kohdalla hammasratasvalintaikkuna näyttää vain ryhmän lähdeanturit, laitteen nimen ja kaavion viivan värin.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->



### Suojuksen valinta

**Suojus**-pudotusvalikosta kerroChloros

ille, mikä fyysinen suojus on asennettu anturin hajottajan päälle, ja sovelletaan kyseisen suojuksen tehtaalla mitattua korjausprofiilia jokaiseen spektriin. Valinnat riippuvat mallista:

| Malli | Suojusvaihtoehdot |
| --- | --- |
| DAQ-U | Ei mitään (paljas anturi), FOV 15°, FOV 30°, FOV 45°, FOV 60°, FOV 90°, Sunshine (kosinikorjain) |
| DAQ-M | Ei mitään (paljas anturi), Sunshine (kosinikorjain) |
| DAQ-E | Ei mitään (paljas anturi), FOV 15°, FOV 45°, FOV 90°, Sunshine (kosinikorjain) |

**Kaikkien mallien oletusasetuksena on Sunshine (kosinikorjain)** — MAPIR toimittaa jokaisen DAQ-laitteen Sunshine-suojuksella asennettuna, ja se on vakiomainen ulkokäyttöön tarkoitettu kokoonpano: 180°:n puolipallonäkymä, jossa kosinivirhe on ≤ ±4 % 60°:n asti ja ≤ ±4,5 % 70°:n asti (ei suositella auringon korkeuden ollessa alle ~15°), suunnittelun mukainen vaimennus (~12×). Valintasi säilyy projektissa.

{% hint style="warning" %}
**Suojuksen valinnan on vastattava fyysistä suojusta.**Anturi tai ohjelmisto eivät pysty tunnistamaan, mikä suojus on asennettuna. Valinta ohjaa sekä reaaliaikaista korjausta että jokaiseen `.daq`-tiedostoon kirjoitettavaa leimaa — Sunshine-suojuksen ~12-kertaisen vaimennuksen vuoksi ilmoittamaton suojuksen vaihto korjaa spektrejä väärin suunnilleen tällä kertoimella. (Saman suojuksen irrottaminen ja uudelleenkiinnittäminen toistaa virheen noin 1,5 %:iin.) Valitse**None (paljas anturi)** vain, kun suojus on fyysisesti irrotettu; DAQ-E-laitteessa ”None” soveltaa edelleen tehtaan geometriprofiilia sen upotetulle lasidiffuusorille – se ei ole tyhjä toiminto – ja paljas DAQ-E on laboratoriokokoonpano, ei tuettu kenttäasetus.
{% endhint %}

{% hint style="info" %}
Päivitys aiemmasta käyttöoppaasta: selaimessa oleva 1.1.0-versiosta tuttu ”Sunshine Diffuser Installed” -kytkin on poistettu. Suojuksen käsittely tapahtuu nyt anturikohtaisen suojusprofiilin avulla, jota sovelletaan palvelinpuolella.
{% endhint %}

***

## Kaavioalue

Kiinnitetty yläpalkki sisältää **luettelo- ⇄ ruudukkonäkymän kytkimen**ja**kaavion zoomausliukusäätimen** (ruudun koko 200–2000 px). Näkymä vaihtuu automaattisesti ruudukonäkymään, kun kaavioryhmiä on useampi kuin yksi, ja takaisin luettelonäkymään, kun niitä on yksi tai vähemmän. Näkymätila ja kaavion koko säilyvät projektikohtaisesti.

Kunkin anturin **spektrikaavio** näyttää:

* **X-akseli** — Aallonpituus (nm). Anturin ruudukko on 340–1010 nm 5 nm:n välein (135 pistettä), joka interpoloidaan 1 nm:n tarkkuudeksi näyttöä varten.
* **Y-akseli**— kalibroidulle anturille (vihreä**Kalibroitu**-pilleri),**Teho (W/m²/nm)**— radiometrisesti kalibroitu spektrinen säteilyvoimakkuus kaikilla kolmella siirtokertoimella — automaattisella SI-etuliitteellä (m/µ/n), joka valitaan huippuarvosta. Anturi, jonka kalibrointipakettia ei ole ladattu, piirtää raakalukemansa**Intensiteettinä (laskenta)**, skaalaamattomina. Kun kaaviossa on päällekkäin molempia tyyppejä, teho on vasemmalla akselilla ja lukumäärä oikealla akselilla.
* Yksittäisen käyrän alla on sateenkaarenvärinen spektrinen täyttö; useat anturit samassa kaaviossa näkyvät päällekkäin värillisinä viivoina, joiden täyttö on himmennetty.
* **Vie hiiri**— pystysuora osoitin, jossa näkyy aallonpituus ja anturikohtainen arvo (W/m²/nm kalibroidulle anturille, muutoin kokonaislaskentamäärä);**vedä** zoomataksesi (zoomattuna näkyy loitontamispainike).
* **+**-painike (vain ruudukkonäkymässä) anturin lisäämiseksi tähän kaavioon tai ryhmän luomiseksi (alla).
* Laitteen nimi keskitettynä yläreunassa ja pyörivä kuvake, kunnes ensimmäinen kehys saapuu.

**Ylikuormitus** ei näy itse kaaviossa: ylikuormittunut anturi näkyy punaisena `SATURATED`-tilatekstinä ja punaisena `Saturated: Yes`-rivinä reaaliaikaisessa datataulukossa. Vähennä integrointiaikaa tai ota AE uudelleen käyttöön sen poistamiseksi.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->



***

## Reaaliaikainen datataulukko (luettelonäkymä)

Kaavion alapuolella luettelonäkymässä, päivitetään 500 ms välein:

* **Kaikki mallit**: Valon värinäyte (sRGB CIE XYZ:stä), Kylläinen (Kyllä/Ei), CIE 1931 X/Y/Z, Kromaattisuus x/y, CIE u′/v′, CCT (K), CRI (Ra), hallitseva aallonpituus (nm), huippuaallonpituus (nm), virityspuhtaus, Duv, CIE L\*/a\*/b\* ja Munsell H/V/C.
* **Vain kalibroidut anturit**(mikä tahansa DAQ-U / DAQ-M / DAQ-E, kun sen tehdaskalibrointipaketti on ladattu — tunnisteena on vihreä**Kalibroitu**-merkki anturirivillä): Kokonaisteho (W/m²), fotopinen lux (lx), skotopinen lux (lx), S/P-suhde, PPFD sekä PPFD Red /Green /Blue (µmol/m²/s) ja opiset säteilyvoimakkuudet — S-kartio, Melanopic, Rhodopic, M-kartio, L-kartio (kaikki W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Heijastusryhmät (Ympäristö + Kohde)

Kaksi toisiinsa liitettyä anturia voidaan yhdistää reaaliaikaiseksi heijastavuusnäytöksi — ilman kameraa:

1. Napsauta ruudukkonäkymässä kaavion ruudun **+**-merkkiä ja valitse**Yhdistä ympäristö + kohde**.
2. Valitse **Ympäristön valonlähde**-anturi ja**Kohteen skanneri**-anturi (kaksi erillistä anturia) ja valitse sitten**Luo**.

Chloros laskee R(λ) = kohde(λ) / ympäristö(λ) aallonpituuskohtaisesti kahdesta reaaliaikaisesta datavirrasta (arvo 0, kun ympäristön arvo ≤ 0). Ryhmän nimike määräytyy anturien kalibrointiluokan mukaan:

* Molemmat anturit kalibroitu (paketti ladattu) → **”Näennäinen heijastavuus”**.
* Jompikumpi anturi kalibroimaton → **"Suhteellinen heijastavuus"**.

Ryhmä näkyy sivupalkissa vihreänä `REF`-rivinä ja omassa kaaviossaan (sateenkaarenvärinen täyttö, arvot näkyvät neljän desimaalin tarkkuudella hiiren osoittimen ollessa päällä, zoomaus vetämällä).

**+**-valikosta löytyy myös vaihtoehto**Lisää uusi anturi**, jossa on kolme sijoitusvaihtoehtoa: *Yhdistä uusi anturi* (liitä tähän kaavioon), *Siirrä olemassa oleva anturi tähän* tai *Näytä uusi anturi* (oma kaavio).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->



### Kasvillisuusindeksitaulukko

Luettelonäkymässä kasvillisuusindeksitaulukko sijaitsee heijastusryhmän kaavion alapuolella, ja se lasketaan reaaliaikaisesta heijastuksesta kaistakeskuksissa **sininen 450 / vihreä 550 / punainen 670 / NIR 800 nm** (arvot neljään desimaaliin asti, `---`, jos laskeminen ei ole mahdollista; vie hiiri indeksin nimen päälle nähdäksesi sen täydellisen nimen):

* **Näkyy aina** (asteikkoriippumaton, mikä tahansa anturiyhdistelmä): NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR.
* **Vain kun molemmat anturit on kalibroitu tehon suhteen** (molemmat paketit ladattuina): EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI.

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

***

## `.daq`-tiedostojen tallennus

* Tallennus edellyttää **avointa projektia** — muuten sekä ”Tallenna kaikki” (sivupalkki) että anturikohtainen ”Tallenna”-painike ovat pois käytöstä.
* Tiedostot tallennetaan kansioon **`<project folder>/light_sensor/`**; tiedostonimissä on anturin tunnus ja aikaleima, ja laitteen nimi tallennetaan tallennuksen mukana.
* Kun tallennus lopetetaan (Lopeta, Lopeta kaikki tai yhteyden katkeaminen tallennuksen aikana), valmis `.daq` **lisätään avoimeen projektiin automaattisesti** — se näkyy projektin tiedostoluettelossa ilman manuaalista lisäystä, valmiina toimimaan alaspäin suuntautuvina tietoina [heijastavuusprosessointia](README.md) varten.
* Punainen `REC`-ilmaisin näkyy asetusmodaalin reaaliaikaisissa riveissä tallennuksen aikana.

Kvantitatiivisten säteilyvoimakkuuslukujen saamiseksi laske keskiarvo vähintään 15 sekunnin ajalta kerätyistä tiedoista — tämä on laitteen ominaisuus, ei vika.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->



***

## Monisensorijärjestelyt ja projektien pysyvyys

* Yhdistä useita antureita yhteen kaavioon (jaetut akselit), pidä kaaviot erillisinä (automaattinen ruudukkojärjestely), siirrä antureita kaavioiden välillä, järjestä rivejä/ruutuja uudelleen vetämällä ja piilota yksittäisiä antureita silmäkuvakkeen avulla.
* Chloros tallentaa projektikohtaisesti seuraavat tiedot: laitenimet, kaavion värit, kaavion koko, katselutila sekä kunkin anturin asetukset (integraatioaika, kehysten keskiarvo, AE-tila, suodattimen valinta).
* **Projektin avaaminen uudelleen yhdistää sen anturit automaattisesti** osoitteen perusteella — DAQ-U:n COM-portti, DAQ-M:n BLE-laite, mDNS-isäntänimi DAQ-E:n osalta (tunnistetaan, vaikka laitteen IP-osoite olisi muuttunut) — ja soveltaa uudelleen kunkin anturin tallennetun cap-profiilin, kehysten keskiarvon, AE-tilan ja manuaalisen integraatioajan.

***

## Kameran pariliitos (DLS)

Parittamista ei tarvita. Toisin kuin drone-DLS-työnkulut, joissa valosensori liitetään kameraan etukäteen, Chloros yhdistää DAQ-tiedot kuviin jälkikäteen: tuonti- ja käsittelyvaiheessa `.daq`-lukemat interpoloidaan kunkin kuvan valotusaikaleimaan. Tallenna millä tahansa liitetyllä anturilla (`.daq` lisätään projektiin automaattisesti), ja heijastuskyvyn käsittely löytää oikeat lukemat ajan perusteella — katso [DAQ-valosensorit](README.md) saadaksesi tietoa siitä, miten alaspäin suuntautuvaa dataa käytetään. **Reaaliaikaisissa**LATTICE-kaappauksissa valitse sen sijaan anturi Kamerat-välilehden**Valosensori**-pudotusvalikosta; jolloin jokainen tallennus tallentaa käyttämänsä lukeman, joka on sovitettu vain kyseisen tallennuksen kameroihin (katso [Heijastavuusprosessit](reflectance.md)).
