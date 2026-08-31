# DAQ-välilehti ohjelmassa Chloros

DAQ-välilehti — jonka nimi on **Valosensorit** Chloros-sivupalkissa — on [DAQ-U-, DAQ-M- ja DAQ-E-valosensorien](README.md) reaaliaikainen ohjauspaneeli: liitä anturit minkä tahansa siirtoprotokollan kautta, seuraa kalibroituja spektrejä reaaliajassa, laske reaaliaikaista heijastavuutta anturiparista ja tallenna `.daq`-tiedostoja suoraan projektiisi.

Välilehti tulee käytettäväksi, kun Chloros-taustapalvelu on käynnistynyt. Välilehden kaaviot saavat tietonsa Chloros:n DAQ-palvelusta reaaliaikaisen yhteyden kautta, joka muodostuu uudelleen automaattisesti (2–10 sekunnin viiveellä), jos yhteys katkeaa; kun palvelua ei tavoiteta, anturin Tila-rivillä lukee **Ei palvelinta**.

Asettelu koostuu **anturien sivupalkista**(yksi rivi kutakin kytkettyä anturia kohti) sekä**kaavioalueesta** (yksi kaaviolaatta kutakin anturia tai ryhmää kohti).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Anturin kytkeminen

Napsauta sivupalkin yläosassa olevaa **Yhdistä anturi** -painiketta. Yhdistämisvalintaikkuna avautuu pääalueelle (tai päällekkäisenä ikkunana, kun lisätään toista anturia – tällöin näkyviin tulee Peruuta-painike).

| Ohjaus | Toiminta |
| --- | --- |
| **Laitetyyppi** | `DAQ-U (USB)` (oletus), `DAQ-M (Bluetooth)` tai `DAQ-E (Ethernet)`. Vaihtaminen käynnistää skannauksen uudelleen valitulle siirtotavalle. |
| **Portti / BLE-laite / Isäntänimi / IP** | Luettelee löydetyt laitteet muodossa `device - description`; ensimmäinen anturiksi tunnistettu merkintä valitaan automaattisesti. Skannauksen aikana näytössä näkyy `Scanning...` (USB), `Scanning (N)...` 8 sekunnin lähtölaskennalla (BLE) tai `Discovering ethernet sensors (N)...` 5 sekunnin lähtölaskennalla (Ethernet). Tyhjät tulokset näkyvät muodossa `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Päivitä** | Skannaa valitun siirtotavan uudelleen välittömästi (pois käytöstä BLE-/Ethernet-skannauksen aikana). |
| **Yhdistä** | Aktivoituu, kun laite on valittu; nimeää uudelleen `Connecting...`:ksi yhteyden muodostamisen aikana. |

Laitteiden etsintä toimii vain **silloin, kun yhteysvalintaikkuna on näytöllä**, ja toistuu 15 sekunnin välein vain valitun siirtotavan osalta — pelkkä välilehden avaaminen ei käynnistä skannausta. Epäonnistumisen yhteydessä valintaikkunassa näkyy: *&quot;Yhteyden muodostaminen epäonnistui. Kokeile irrottaa ja kytkeä anturi uudelleen, ja napsauta sitten Yhdistä uudelleen.&quot;*

Sivupalkki avautuu automaattisesti, kun ensimmäinen anturi muodostaa yhteyden.

{% hint style="info" %}
**Eikö DAQ-E näy?** DAQ-E:ssä ei ole tilan LED-valoa — tarkista PoE-/linkkivalo kytkimessä tai injektoriportissa, johon se on kytketty, ja odota muutama sekunti käynnistyksen jälkeen, jotta laite ehtii käynnistyä. Chloros-laitteen on oltava samassa lähetysalueessa (mDNS ei läpäise reitittimiä). Kun Windows on käynnissä, hyväksy Defender-palomuurin kehote ensimmäisellä kerralla, kun Chloros muodostaa yhteyden multicast-liitäntöihinsä (mDNS UDP 5353, DAQ-E-data UDP 5002, PTP UDP 319/320). Kaksi samassa lähiverkossa olevaa DAQ-E-yksikköä tunnistetaan erikseen, kumpikin omalla `daq-e-<id>.local`-isäntänimellään.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Laitetyyppi-valikosta löytyy DAQ-U (USB), DAQ-M (Bluetooth) ja DAQ-E (Ethernet)</figcaption></figure>***

## Anturien sivupalkki

Jokaiselle liitetylle anturille varataan oma rivi (sekä yksi rivi kutakin Ambient+Object-ryhmää kohti). Rivien järjestystä voi muuttaa vetämällä, ja niiden järjestys vaikuttaa myös kaavion ruutujen järjestykseen. Napsauttamalla riviä voit asettaa kyseisen anturin tai ryhmän aktiiviseksi kaavioksi luettelonäkymässä.

| Elementti | Merkitys |
| --- | --- |
| Värillinen vasen reuna | Anturin kaavion väri. |
| Siirto-merkki | `DAQ-U` / `DAQ-M` / `DAQ-E` tai vihreä `REF`-tunnus Ambient+Object-heijastusryhmälle. |
| Laitteen nimi | Oletusarvo on anturin sarjanumero (sen vakiintunut tunniste kalibrointia, `.daq`-tiedostonimiä ja tuonnin täsmäytystä varten); mukautetut nimet säilyvät projektikohtaisesti. |
| **Kalibroitu**-pilleri (vihreä) | Näkyy, kun anturin tehdaskalibrointipaketti on ladattu, eli spektrit ovat todellisia W/m²/nm. |
| **Päivitys saatavilla** -kuvake (keltainen, vain DAQ-E) | Käytössä oleva laiteohjelmisto on vanhempi kuin tämän Chloros-rakennuksen mukana toimitettu kuva. Päivityksen aikana se näyttää reaaliaikaista edistymistä (`Flashing… N%`, `Restarting sensor…`, sitten `Updated X → Y` tai `Failed`). |
| Silmä | Kytkee tämän anturin näkyvyyden päälle tai pois sen kaaviossa. |
| Hammasratas | Avaa anturikohtaisen asetusikkunan (alla). |
| ✕ (punainen) | Irrottaa anturin tai poistaa Ambient+Object-ryhmän. |

Rivien yläpuolella on kaksi painiketta:

* **Yhdistä anturi** — avaa yhdistämisvalintaikkunan (muuttuu nimeksi `Connecting...`, kun toiminto on käynnissä).
* **Tallenna kaikki / Lopeta kaikki**— käynnistää tai lopettaa `.daq`-tallennuksen**kaikilla**kytketyillä antureilla. Vaatii vähintään yhden anturin**ja avoimen projektin** (työkaluvinkki: ”Avaa projekti tallennusta varten”); painike muuttuu punaiseksi, kun tallennus on käynnissä.

Tyhjässä tilassa näkyy teksti &quot;Ei kytkettyjä antureita&quot;.

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Anturikohtaiset asetukset (hammasratas-modaali)

Avaa anturirivin hammasrataskuvakkeesta. Sisältö järjestyksessä:

* **Tietorivit** — Laitetyyppi (DAQ-U/M/E), Liitäntä (`Serial (USB)` / `Bluetooth` / `Ethernet`), portti (COM-portti, BLE-osoite tai isäntä) ja sarjanumero.
* **Kalibrointiraportti: Lataa** — hakee tämän laitteen NIST-jäljitettävän kalibrointitodistuksen (PDF) ja avaa sen PDF-katseluohjelmassasi. Käytettävissä, kun sarjanumero on tiedossa; todistus tallennetaan välimuistiin ensimmäisen yhteyden muodostuksen yhteydessä.
* **Laitteen nimi** — napsauta kynää nimen muuttamiseksi; säilyy projektikohtaisesti.
* **Käyrän viivan väri** — värinäyte; säilyy projektikohtaisesti.
* **Integrointiaika (ms)**— liukusäädin + luku,**1–500 ms**, oletusarvo**32 ms**. Pois käytöstä, kun AE on päällä.
* **Kehysten keskiarvo**— liukusäädin + luku,**1–50 kehystä**, oletusarvo**20**.
* **AE: PÄÄLLÄ/POIS**— automaattisen valotuksen kytkin;**oletusarvo PÄÄLLÄ** yhteyden muodostamisen yhteydessä. Kytke se pois päältä, jos haluat asettaa integrointiajan manuaalisesti.
* **Lopeta suoratoisto / Aloita suoratoisto** — keskeytä tai jatka suoratoistoa.
* **Tallenna / Lopeta tallennus** — anturikohtainen `.daq`-tallennus (vaatii avoimen projektin).
* **Cap** — cap-korjausprofiili (seuraava osa).
* **Reaaliaikaiset tietorivit** — integraatioaika (ms), FPS, näytteet, tallennus (punainen `REC` tai `Off`) ja tila (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Vain DAQ-E: verkko-, laiteohjelmisto- ja PTP-rivit

* **Isäntänimi / IP** — laitteen nykyinen osoite.
* **Laiteohjelmisto**— reaaliaikainen laiteohjelmistoversio sekä toimintosolu:**Päivitä versioon \<version\>

** -painike tulee näkyviin, kun tämä Chloros-versio sisältää uudemman DAQ-E-laiteohjelmistokuvan. Päivitys asennetaan verkon kautta noin 30 sekunnissa; anturi käynnistyy uudelleen ja muodostaa yhteyden automaattisesti, ja keskeytynyt siirto jättää nykyisen laiteohjelmiston ennallaan. Päivityksen eteneminen näkyy reaaliaikaisesti (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`), ja kentässä lukee `Up to date`, kun päivitys on ajantasainen.
* **PTP-synkronointi** — reaaliaikainen PTP-tila (palaa tilaan `unknown`). DAQ-E-laiteohjelmisto versio 1.2.0 tai uudempi toimii IEEE 1588 PTPv2 -standardissa pelkästään orjakellona; Chloros-isäntäkoneen taustapalvelin on PTP-grandmaster, ja jokainen LAN-verkossa oleva DAQ-E- ja LATTICE-kamera toimii sen orjana domainissa 0, jolloin aikaleimat pysyvät noin 1 ms:n tarkkuudella.

Ambient+Object-ryhmän kohdalla laitemodaali näyttää vain ryhmän lähdesensorit, laitteen nimen ja graafilinjan värin.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->

### Suojuksen valinta

**Cap**-pudotusvalikosta määritetään Chloros:lle, mikä fyysinen suojus on asennettu anturin hajottimen päälle, ja sovelletaan kyseisen suojuksen tehtaalla mitattua korjausprofiilia jokaiseen spektriin. Valinnat riippuvat mallista:

| Malli | Suojusvaihtoehdot |
| --- | --- |
| DAQ-U | Ei mitään (paljas anturi), FOV 15°, FOV 30°, FOV 45°, FOV 60°, FOV 90°, Sunshine (kosinikorjain) |
| DAQ-M | Ei mitään (paljas anturi), Sunshine (kosinikorjain) |
| DAQ-E | Ei mitään (paljas anturi), FOV 15°, FOV 45°, FOV 90°, Sunshine (kosinikorjain) |

**Kaikkien mallien oletusasetuksena on Sunshine (kosinikorjain)** — MAPIR toimittaa jokaisen DAQ-laitteen Sunshine-suojuksella asennettuna, ja se on vakiomainen ulkokäyttöön tarkoitettu kokoonpano: 180°:n puolipallonäkymä, jossa kosinivirhe on ≤ ±4 % 60°:n asti ja ≤ ±4,5 % 70°:n asti (ei suositella auringon korkeuden ollessa alle ~15°), suunnittelun mukainen vaimennus (~12×). Valintasi säilyy projektissa.

{% hint style="warning" %}
**Kannen valinnan on vastattava fyysistä kantta.**Anturi tai ohjelmisto eivät pysty tunnistamaan, mikä kansi on asennettuna. Valinta ohjaa sekä reaaliaikaista korjausta että jokaiseen `.daq`-tiedostoon kirjoitettavaa leimaa — Sunshine-suojuksen ~12× vaimennuksen vuoksi ilmoittamaton suojuksen vaihto korjaa spektrejä väärin suunnilleen kyseisellä kertoimella. (Saman suojuksen irrottaminen ja takaisin asettaminen toistuu noin 1,5 %:n tarkkuudella.) Valitse**None (paljas anturi)** vain silloin, kun suojus on fyysisesti irrotettu; DAQ-E-laitteessa ”None” soveltaa silti tehtaan geometriaprofiilia sen upotetulle lasidiffuusorille – se ei ole tyhjä toiminto – ja paljas DAQ-E on laboratoriokokoonpano, ei tuettu kenttäasetus.
{% endhint %}

{% hint style="info" %}
Päivitys aiemmasta käyttöoppaasta: selaimen puolella oleva 1.1.0-versiosta peräisin oleva ”Sunshine Diffuser Installed” -kytkin on poistettu. Suojuksen käsittely tapahtuu nyt tämän anturikohtaisen suojusprofiilin avulla, jota sovelletaan palvelinpuolella.
{% endhint %}

***

## Kaavioalue

Kiinnitetty yläpalkki sisältää **luettelo- ⇄ ruudukkonäkymän kytkimen**ja**kaavion zoomausliukusäätimen** (ruudun koko 200–2000 px). Näkymä vaihtuu automaattisesti ruudukonäkymään, kun kaavioita on useampi kuin yksi, ja takaisin luettelonäkymään, kun kaavioita on yksi tai vähemmän. Näkymätila ja kaavion koko tallentuvat projektikohtaisesti.

Kunkin anturin **spektrikaavio** näyttää:

* **X-akseli** — Aallonpituus (nm). Anturiruudukon alue on 340–1010 nm 5 nm:n välein (135 pistettä), ja se interpoloidaan näytössä 1 nm:n tarkkuudella.
* **Y-akseli** — Teho (W/m²), jossa huippuarvosta valitaan automaattisesti SI-etuliite (m/µ/n). Spektrit ovat radiometrisesti kalibroituja spektrisiä säteilyvoimakkuuksia (W/m²/nm) kaikissa kolmessa siirtomuodossa.
* Yksittäisen käyrän alla on sateenkaarenvärinen spektrinen täyttö; useat anturit samassa kaaviossa näkyvät päällekkäisinä värillisinä viivoina, joiden täyttö on himmennetty.
* **Hover**— pystysuora osoitin, jossa näkyy aallonpituus ja anturikohtainen arvo;**vedä** zoomataksesi (zoomattuna näkyy loitontamispainike).
* **+**-painike (vain ruudukkonäkymässä) anturin lisäämiseksi tähän kaavioon tai ryhmän luomiseksi (alla).
* Laitteen nimi keskitettynä yläreunaan ja pyörivä kuvake, kunnes ensimmäinen kehys saapuu.

**Kylläisyys** ei näy itse kaaviossa: kylläinen anturi näkyy punaisena `SATURATED`-tilatekstinä ja punaisena `Saturated: Yes`-rivinä reaaliaikaisessa datataulukossa. Vähennä integrointiaikaa tai ota automaattinen valotus (AE) uudelleen käyttöön, jotta tilanne korjaantuu.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->

***

## Reaaliaikainen datataulukko (luettelonäkymä)

Kaavion alapuolella luettelonäkymässä, päivitetään 500 ms välein:

* **Kaikki mallit**: Valon värinäyte (sRGB CIE XYZ:stä), Kylläinen (Kyllä/Ei), CIE 1931 X/Y/Z, Kromaattisuus x/y, CIE u′/v′, CCT (K), CRI (Ra), hallitseva aallonpituus (nm), huippuaallonpituus (nm), virityspuhtaus, Duv, CIE L\*/a\*/b\* ja Munsell H/V/C.
* **Vain kalibroidut anturit**(mikä tahansa DAQ-U / DAQ-M / DAQ-E -anturi, kun sen tehdaskalibrointipaketti on ladattu — tunnusmerkkinä on vihreä**Kalibroitu**-merkki anturirivillä): Kokonaisteho (W/m²), fotopinen lux (lx), skotopinen lux (lx), S/P-suhde, PPFD sekä PPFD Red/Green/Blue (µmol/m²/s) sekä opiset säteilyvoimakkuudet — S-kartio, melanopinen, rodopinen, M-kartio, L-kartio (kaikki W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Heijastusryhmät (ympäristö + kohde)

Kaksi toisiinsa liitettyä anturia voidaan yhdistää reaaliaikaiseksi heijastusnäytöksi — ilman kameraa:

1. Napsauta ruudukkonäkymässä kaavion ruudun **+**-painiketta ja valitse**Yhdistä ympäristö + kohde**.
2. Valitse **Ympäristön valonlähde**-anturi ja**Kohteen skanneri**-anturi (kaksi erillistä anturia) ja valitse sitten**Luo**.

Chloros laskee aallonpituuskohtaisesti R(λ) = kohde(λ) / ympäristö(λ) näiden kahden reaaliaikaisen datavirran perusteella (arvo 0, kun ympäristövalon arvo on ≤ 0). Ryhmän nimi määräytyy anturien kalibrointiluokan mukaan:

* Molemmat anturit kalibroitu (paketti ladattu) → **&quot;Näennäinen heijastavuus&quot;**.
* Jompikumpi anturi on kalibroimaton → **&quot;Suhteellinen heijastavuus&quot;**.

Ryhmä näkyy vihreänä `REF`-rivinä sivupalkissa ja omassa kaaviossaan (sateenkaari-täyttö, arvot näkyvät hiiren osoittimen ollessa päällä neljän desimaalin tarkkuudella, zoomaus vetämällä).

**+**-valikosta löytyy myös vaihtoehto**Lisää uusi anturi**, jossa on kolme sijoitusvaihtoehtoa: *Yhdistä uusi anturi* (liitä tähän kaavioon), *Siirrä olemassa oleva anturi tähän* tai *Näytä uusi anturi* (oma kaavio).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->

### Kasvillisuusindeksitaulukko

Luettelonäkymässä kasvillisuusindeksitaulukko sijaitsee heijastusryhmän kaavion alapuolella, ja se lasketaan reaaliaikaisesta heijastuksesta kaistakeskuksissa **sininen 450 / vihreä 550 / punainen 670 / NIR 800 nm** (arvot neljään desimaaliin asti, `---`, jos arvoa ei voida laskea; vie hiiri indeksin nimen päälle nähdäksesi sen täydellisen nimen):

* **Näkyy aina** (asteikkoriippumaton, mikä tahansa anturiyhdistelmä): NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR.
* **Vain kun molemmat anturit on kalibroitu tehon mukaan** (molemmat paketit ladattuina): EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI.

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

***

## `.daq`-tiedostojen tallennus

* Tallennus edellyttää **avointa projektia** — muutoin sekä ”Tallenna kaikki” (sivupalkki) että antikohtaiset tallennuspainikkeet ovat pois käytöstä.
* Tiedostot tallennetaan nimellä **`<project folder>/light_sensor/`**; tiedostonimissä on anturin tunnus ja aikaleima, ja laitteen nimi tallennetaan tallennuksen mukana.
* Kun tallennus lopetetaan (Lopeta, Lopeta kaikki tai yhteyden katkeaminen tallennuksen aikana), valmis `.daq` **lisätään avoimeen projektiin automaattisesti** — se näkyy projektin tiedostoluettelossa ilman manuaalista lisäystä, valmiina toimimaan alaspäin suuntautuvana datana [heijastavuuden käsittelyä](README.md) varten.
* Punainen `REC`-ilmaisin näkyy asetusmodaalin reaaliaikaisissa riveissä tallennuksen aikana.

Saadaksesi kvantitatiivisia säteilyvoimakkuuslukuja, laske keskiarvo vähintään 15 sekunnin ajalta kerätyistä tiedoista — tämä on laitteen ominaisuus, ei vika.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->

***

## Monisensorijärjestelyt ja projektien pysyvyys

* Yhdistä useita antureita yhteen kaavioon (jaetut akselit), pidä kaaviot erillisinä (automaattinen ruudukkoasettelu), siirrä antureita kaavioiden välillä, järjestä rivejä tai ruutuja uudelleen vetämällä ja piilota yksittäisiä antureita silmäkuvakkeen avulla.
* Projektikohtaisesti Chloros säilyttää seuraavat tiedot: laitenimet, kaavion värit, kaavion koko, katselutila sekä kunkin anturin asetukset (integraatioaika, kehysten keskiarvo, AE-tila, rajausvalinta).
* **Projektin avaaminen uudelleen yhdistää sen anturit automaattisesti** osoitteen perusteella — DAQ-U:n COM-portti, DAQ-M:n BLE-laite, mDNS-isäntänimen avulla DAQ-E:n osalta (tunnistetaan, vaikka laitteen IP-osoite olisi muuttunut) — ja soveltaa uudelleen kunkin anturin tallennetun cap-profiilin, kehysten keskiarvon laskennan, AE-tilan ja manuaalisen integraatioajan.***

## Kameran pariliitos (DLS)

Pariliitosta ei tarvitse tehdä. Toisin kuin drone-DLS-työnkulut, joissa valosensori liitetään kameraan etukäteen, Chloros yhdistää DAQ-tiedot kuviin jälkikäteen: tuonti- ja käsittelyvaiheessa `.daq`-lukemat interpoloidaan kunkin kuvan valotusaikaleimaan. Tallenna millä tahansa liitetyllä anturilla (`.daq` lisätään projektiin automaattisesti), ja heijastuskyvyn käsittely löytää oikeat lukemat ajan perusteella — katso [DAQ-valosensorit](README.md) saadaksesi tietoa siitä, miten alaspäin suuntautuvaa dataa käytetään.</version\>
