---
metaLinks: {}
---

# Aloitusopas

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros on [MAPIR](https://www.mapir.camera) -yhtiön kehittämä sovellus, jolla voidaan käsitellä monispektrikuvia, ohjata MAPIR-laitteita reaaliaikaisesti ja tallentaa anturitietoja. Chloros 1.3.0 tukee koko MAPIR-tuoteperhettä:

* **Survey3-kamerat** — käsittelee RAW+JPG-kuvat kalibroiduiksi heijastavuus- ja kasvillisuusindeksikartoiksi. Katso [Tuetut kamerat](supported-cameras.md).
* **LATTICE-kamerat** — yhdistä GigE-monispektrikameramoduulit reaaliaikaisesti, yksittäin tai synkronoituina monikamerajärjestelminä: esikatsele, tallenna ja käsittele kalibroiduiksi säteily- ja heijastavuustuotteiksi. Katso [LATTICE-osio](lattice/README.md).
* **DAQ-valosensorit** — DAQ-U (USB), DAQ-M (Bluetooth) sekä DAQ-E ja DAQ-E-S (Ethernet) -spektrisensorit: reaaliaikaiset kalibroidut spektrit, `.daq`-tallenteet sekä alaspäin suuntautuva valaistus heijastavuuden käsittelyä varten. Katso [DAQ-osio](daq/README.md).

{% hint style="success" %}
**Chloros 1.3.0:n uudet ominaisuudet**: Tekoälyavustajat voivat ohjata Chloros-ohjelmistoa `chloros-mcp`:n kautta, ja CLI, SDK sekä `chloros-mcp` tarjoavat samat ominaisuudet. LATTICE-indeksituotteita on parannettu, täysianturijärjestelmät ovat nopeampia ja toimivat missä tahansa verkossa, ja taustaprosessi jatkaa toimintaansa komentojen välillä. Katso alla oleva Uutuudet-luettelo ja [Lataa](download.md) asennusohjelmat.
{% endhint %}

{% hint style="info" %}
**Käytätkö Chloros ia tekoälyavustajan kanssa?** Tämä käyttöopas on laadittu juuri sitä varten. Ohjaa avustajasi osoitteeseen:

* `https://mapir.gitbook.io/chloros/llms.txt` — koneellisesti luettava hakemisto kaikista sivuista.
* Mikä tahansa sivu raakamuodossa (Markdown) — lisää `.md` sen URL-osoitteeseen (esim. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI-ohje](reference/cli-reference.md) ja [SDK-ohje](reference/sdk-reference.md) — kattavat, tarkkoja arvoja sisältävät ohjesivut, jotka on kirjoitettu suurten kielimallien (LLM) käyttöön.

Esimerkki-komento: *"Lue https://mapir.gitbook.io/chloros/reference/cli-reference.md, ja kirjoita sitten skripti, joka kirjautuu sisään ja käsittelee kansion ~/flights/flight_001 heijastavuus- ja NDVI-arvojen sisältäviksi GeoTIFF-tiedostoiksi."*

Tai anna sen suorittaa Chloros ia suoraan: `chloros-mcp`, SDK:n mukana asennettu MCP-palvelin, toimii Claude Desktopin, Claude Coden, Cursorin ja muiden MCP-asiakasohjelmien kanssa.

Kattava opas: [Chloros in käyttö AI-avustajien kanssa](ai-assistants.md).
{% endhint %}

***

## Uutta Chloros 1.3.0:ssa

* **Tekoälyavustajat voivat ohjata Chloros ia.** `chloros-mcp`, Python-SDK:n mukana asennettu MCP-palvelin, mahdollistaa sen, että Claude Desktop, Claude Code, Cursor ja muut MCP-asiakasohjelmat voivat käsitellä kuvamateriaalia sekä käyttää LATTICE-kameroita, -anturiryhmiä ja DAQ-valosensoreita. Katso [Chloros in käyttö tekoälyavustajien kanssa](ai-assistants.md).
* **CLI, SDK ja `chloros-mcp` tarjoavat samat ominaisuudet.** Turvallisuussyistä vain kirjautuminen, DAQ-E-määritykset ja suorat DAQ-palvelimet pysyvät CLI:n piirissä. Joitakin 1.2.0-skriptejä on muutettava: katso **Muutokset versiossa 1.3.0** [CLI-ohjeen](reference/cli-reference.md) ja [SDK-ohjeen](reference/sdk-reference.md) alussa.
* **Parannetut LATTICE-indeksituotteet.** LATTICE-tallenteista vietyjä NDVI- ja muita indeksituotteita on parannettu kaikissa käsittelytiloissa.
* **Windows-asennusohjelman asetukset.** **Järjestelmäasetukset**-kohdassa asennusohjelma tarjoaa vaihtoehdot**Ota Windows in pitkät polut käyttöön**ja**Jätä Chloros-taustapalvelu pois Microsoft Defenderin tarkistuksesta**, jotka molemmat ovat valittuna oletuksena. Ensimmäinen vaihtoehto sallii projektikansioiden sijainnin yli 260 merkin syvyydessä; toinen estää reaaliaikaista skannausta hidastamasta kameran tallennusta (noin 30 % tallennusnopeudesta suurilla nopeuksilla). Asennusohjelma lisää nyt myös CLI-kansion `PATH`-kansioon vain kerran ja poistaa aiempien asennusohjelmien jättämät päällekkäiset merkinnät. Katso [Lataa](download.md).
* **Liian syvät projektikansiot hylätään heti alussa.** Kun Windows in pitkät polut on kytketty pois päältä (Windows in oletusasetus), projektikansio, joka on liian syvä 260 merkin polkurajalle, hylätään ennen kuin mitään suoritetaan, ja näytetään pisin sopiva projektikansio sekä ohjeet pitkien polkujen kytkemiseksi päälle. Aiemmin tällainen suoritus ei tuottanut tuloksia. Kun pitkät polut ovat käytössä, hyvin syvissä kansioissa olevat tuotteet säilyttävät nyt metatietonsa (GPS- ja XMP-tunnisteet).
* **Naarmuuntumismaskit.** Kun LATTICE-säteily- tai heijastavuustuotteen kaistojen erottelu tuottaa negatiivisen kaistiarvon, kyseinen arvo leikataan nollaan, ja tuotekansioiden vieressä olevassa `clamp_masks/`-kansiossa oleva pikselikohtainen maski tallentaa, mihin pikseleihin ja kaistoihin tämä vaikutti. Tuote nimeää maskinsa `Chloros:UnmixClampMask`-tunnisteessa. Reaaliaikaiset näkymät leikkaavat nyt täsmälleen samalla tavalla kuin vienti. Katso [Tulostuskuvamuodot](output-image-formats.md).
* **Tuotteet kuvaavat itseään.** Indeksituotteet sisältävät omat käsittelytunnisteensa (`Chloros:ProcessingLevel=index`, indeksilauseke sekä tieto siitä, onko ne laskettu säteilyvoimakkuuden vai heijastavuuden perusteella) lähderaw-tiedostojen sijaan. Survey3-tuotteet sisältävät Chloros-käsittelytunnisteet kaikissa käsittelytiloissa. LATTICE-raakakuvat ja debayeroidut tiedostot säilyttävät anturin alkuperäisen suuntauksen ja tallentavat suuntauksensa XMP-tiedostoon; kuvankatseluohjelma kääntää ne vain näyttöä varten.
* **Linux-paketit toimitetaan pakkaamattomina.** Taustaprosessi ja CLI-käyttöliittymä asennetaan kansioina `/usr/lib/chloros/`-kansion alle, joten ne eivät enää pura itseään jokaisen käynnistyksen yhteydessä. x86_64-alustalla taustapalvelimen käynnistyminen ei enää kestä noin 100 sekuntia, ja `.deb`-kansion koko on noin puolet entisestä. Mukana toimitettu ExifTool toimii nyt puhtaassa Ubuntu-asennuksessa. `.deb` asentaa nyt myös Python-SDK:n järjestelmälle `python3` (asenna se komennolla `sudo apt install ./chloros_1.3.0_<arch>.deb`), kasvattaa ytimen socket-vastaanottopuskureita, joita kameravirrat tarvitsevat, ja lisää taustapalvelun, joka toimii käyttäjän nimissä. Jetson-paketti, `chloros_1.3.0_arm64_jp6.deb`, on rakennettu JetPack 6:lle ja se käsittelee dataa Jetsonin GPU:lla JetPackin CUDA-rakenteisella PyTorch-versiolla. Linux issa jokainen viety kuva päätyy nyt levylle kerran (kirjoitetaan ja merkitään muistiin, minkä jälkeen kopioidaan paikalleen): Jetsonissa, jossa levy määrää vauhdin, 288 kuvan LATTICE-ajo kestää 14 minuuttia 24 minuutin sijaan. Katso [Linux-asennus](linux/linux-installation.md).
* **LATTICE-OCN-kamerat lukevat tarkemmin auringonvalossa.** Säteilyvoimakkuus, heijastavuus ja indeksit korjataan nopeasti kaistojen erottelun jälkeen, joten harmaa kohde lukee auringossa lähellä neutraalia NDVI-arvoa. RGN-kamerat pysyvät ennallaan.
* **Sovelluksen sisäiset vikailmoitukset.** Debug Log -välilehden yläosassa oleva vikakuvake lähettää MAPIR ille kuvauksen, enintään 10 kuvaa, lokitiedot ja järjestelmätiedot. Katso [GUI: Navigointi](navigation.md).
* **LATTICE-matriisit.**Matriisit säilyttävät oletusarvoisesti koko anturin tiedot, ja pakattu 12-bittinen siirto (`--packed`, oletusarvo `auto`) nostaa koko anturin kuvataajuutta noin kolmanneksella; matriisi toimii missä tahansa verkossa sen sallimalla kuvataajuudella (hidas yhteys alentaa kuvataajuutta sen sijaan, että hylkäisi matriisin tai pienentäisi sen kuvia); matriisi, jonka kamerat käynnistyvät uudelleen, palautuu itsestään, kun PTP kytkeytyy takaisin päälle; automaattinen valotus ei enää alenna kuvataajuutta hämärässä valossa (se täyttää jakson ja nostaa sitten vahvistusta); Jatkuva tallennus pysyy nopeiden ryhmien vauhdissa;**Highlights**-säädin RGB-kameroille; ja SDK:n `ArraySession` saa lisäominaisuudet `grab()`, `frame_stream()` ja `continuous()`. Katso [LATTICE-osio](lattice/README.md).
* **Vakaampi automaattinen valotus.** Valotus pysyy vakiona valaistuksen ollessa muuttumaton ja vakiintuu muutaman sekunnin kuluessa muutoksesta; pienet kirkkaat kohdat eivät enää tummenna kuvaa; manuaalisessa valotuksessa vahvistus pysyy automaattisena; ja kamera, joka lakkaa reagoimasta kirkkaassa auringonvalossa, pidetään valaistustasolla, jonka valosensori lukee tai, jos sensoria ei ole kytketty, kameran omien kuvien perusteella. Katso [Kameran asetukset](lattice/camera-settings.md).
* **Valosensorit.** **DAQ-E-S-tuki:** sen kiinteä hajotin ei vaadi korkkikorjausta, ja jokainen lukema sisältää laitteen kallistuksen, kiertymisen ja nousukulman. Tallenne ilmoittaa, onko sen korkki valittu vai oletettu, ja käsittely varoittaa oletetusta korkista: DAQ-U-, DAQ-M- ja DAQ-E-malleissa ei ole mukana korkkia, joten nimeä asennettu korkki. Reaaliaikaiset `.daq`-tallenteet säilyvät sähkökatkon jälkeen (säilytä `.daq-journal`-tiedosto yhdessä sen `.daq`-tiedoston kanssa), ja DAQ-E, joka lakkaa lähettämästä, kytketään uudelleen, jolloin tauko merkitään sen tallenteeseen. Projekti, joka sisältää vain valosensorin tallenteita, listaa ne ja käsittelee niitä sovelluksen ja CLI:n kautta, ja `chloros_scripts`:llä tehty tallenne listataan sen tuottaneena DAQ:na, ja sille voi asettaa ylärajan. Jokainen tallennus tallentaa käytetyn DAQ:n lukeman. Katso [DAQ-osio](daq/README.md).
* **Käsittely.**LATTICE-kehykset aloittavat käsittelyn kohteen tunnistuksen aikana. Kohde- ja DAQ-viitteet määritetään kaistakohtaisesti. Siirretty projekti linkittää tiedostonsa uudelleen. 8-bittisen tallennuksen vientitiedostot ovat 8-bittisiä, ja niissä käytetään uutta**TIFF (8-bittinen)** -muotoa. Survey3-ajo ilman kalibrointikohdetta tallennetaan sen varatuotteena (`Vignette_Corrected_Images` tai `Sensor_Response_Images`), ei heijastavuutena. Heijastavuuskuvan kaistakohtaiset DLS-tunnisteet ovat aina sen oman kehyksen tunnisteita. Texture Aware -toimintoa ei voida käyttää LATTICE-kuvissa, joihin sovelletaan aina vakiomuotoista debayerointia.
* **CLI & SDK.** Suhteelliset tulostuspolut tallentuvat nyt siihen kansioon, josta komento tai skripti suoritetaan (aiemmin asennettu taustaprosessi yritti ratkaista ne omassa asennuskansiossaan, mikä johti virheeseen). Taustapalvelu pysyy käynnissä komentojen ja skriptien välillä ja pysähtyy itsestään, jos sitä ei käytetä 30 minuuttiin; `chloros-cli backend status` ja `backend stop` hallitsevat tätä. Tallennuskomennot luettelevat kaikki tiedostot, joita ne eivät pystyneet kirjoittamaan, ja lopettavat toimintansa koodilla 1, ja toistuvat tallennukset pysähtyvät, kun levy on täynnä. Onnistunut `process`-ajo raportoi tuloksen `Image products written: N (plus M clamp masks)`. Windows issa SDK:n `ChlorosLocal` / `process_folder` etsii asennetun taustapalvelimen (ja käyttää jo käynnissä olevaa), ja `read_image_audit_tags` toimii tavallisesta `pip install`-tiedostosta.

***

## Uutta Chloros-versiossa 1.2.0

* **Kameran reaaliaikainen ohjaus — uusi Kamerat-välilehti.** Liitä LATTICE-kamerat yksitellen tai synkronoituina monikamerajärjestelminä (PTP-aikasynkronointi, laitteistokäynnistetty kuvaus), joissa on reaaliaikaiset esikatselupäällekkäisyydet, kaistakohtaiset histogrammit, älykäs automaattinen valotus, reaaliaikainen indeksilaskuri ja sovelluksen sisäiset kameran laiteohjelmistopäivitykset.
* **Valoanturit — uusi Valoanturit-välilehti.** Liitä DAQ-U (USB), DAQ-M (Bluetooth) ja DAQ-E (Ethernet) -anturit; tarkastele kalibroituja reaaliaikaisia spektrejä (W/m²/nm), tallenna `.daq`-tiedostoja projektiisi, valitse kap-korjausprofiileja ja päivitä DAQ-E-laiteohjelmisto verkon kautta.
* **Tallennustilat ja tallentimet.** Yksittäinen / Jatkuva / Intervallitallennus sekä pelkästään raakadataa käyttävä Nopein tallennus -tila; projektikohtainen valinta siitä, mitä kameroita ja vientityyppejä ”Tallenna kaikki” tuottaa; matriisitallentimet seurantatasoiseen indeksivideoon ja analyysitasoisiin raakadatan sarjoihin, joissa on offline-videokoosteet.
* **LATTICE-käsittelyputki.** Tuo LATTICE-tallennuskansiot ja jaa jokainen raakakuva debayeroituihin, esikatselu-, float32-säteilyvoimakkuus- (W/m²/sr/nm) ja heijastavuus-tuotteisiin, joissa on tuotekohtaiset kytkimet. Heijastavuusarvo voi perustua kehyksessä olevaan kalibrointikohteeseen tai DAQ-tietojen alaspäin suuntautuvaan valoon; vientiin sovelletaan matriisin kohdistusta; puuttuva tehdaskalibrointi ladataan automaattisesti kameran sarjanumeron perusteella.
* **Projektit muistavat laitteiston.** Liitetyt kamerat ja valosensorit tallennetaan projektin mukana (`cameras.json` / `sensors.json`) ja ne muodostavat yhteyden uudelleen tallennetuilla asetuksillaan, kun avaat projektin uudelleen. Katso [GUI: Projektit](projects.md).
* **Kuvankatseluohjelman päivitykset.** Kursorin pikseli-/indeksilukema, jossa on oikea tiedostokohtainen heijastuskertoimen skaalaus, kerrosten histogrammit, GSD-binning-liukusäädin, ruudukkomoodit ”Per Trigger” ja ”Per Camera”, LATTICE-tuotteen näkymät sekä indeksi-/LUT-sandbox-vienti levylle.
* **CLI ja SDK on laajennettu huomattavasti.** Uudet komentojoukot `lattice`, `daq pool-*`, `project` ja `time-sync`; uudet `process`-vaihtoehdot (`--input-level`, tuotekohtaiset kytkimet, `--reflectance-source`, taulukon kohdistusliput); SDK:n älykkäät yhteyskäsittelyt (`connect_camera` / `connect_array` / `connect_daq_sensor`), jotka käynnistävät taustapalvelun automaattisesti; `open_project()`-automaatio; SDK-pyörä on mukana asennusohjelmissa ja julkaistu PyPI:ssä nimellä `chloros-sdk`.
* **Rehellinen virhesemantiikka.** `chloros-cli process`-ajo, joka pyysi tuotteita mutta ei kirjoittanut yhtään, epäonnistuu nyt selvästi ja päättyy nollasta poikkeavalla tuloksella; onnistuneet ajot raportoivat, kuinka monta kuvatuotetta ne kirjoittivat.
* **Uusi tulosteiden rakenne.** Tuotteet tallennetaan `<project>/<camera>/<format>/<Product>_Images/`-kansioihin ja säilyttävät lähdetiedoston nimen — kansio, ei tiedostotunnistetta, tunnistaa tuotteen. Katso [Kuvien tulostusformaatit](output-image-formats.md).
* **Lisää syötteitä, suunnitelmia ja kieliä.** `.dng`-syötteen tuki; kaikki 38 käyttöliittymäkieltä täysin käytettävissä; suunnitelmakohtaiset laitteistorajoitukset, joissa ilmaista (kirjautumattomaa) käyttöä varten sallitaan enintään 4 kameraa ja 2 valosensoria.
* **Luotettavuus.** Käsittelyn lopettaminen päättyy siististi ja rehellisellä suoritusyhteenvedolla, monikameraprojektit vievät tiedot jokaisesta kamerasta, eikä asennusohjelman päivitys enää kirjaa sinua ulos.***

Chloros ia on saatavana kolmella käyttöliittymäalustalla:

## Chloros: Työpöydän graafinen käyttöliittymä

Erillinen ikkuna, jossa on kaikki ominaisuudet, mukaan lukien reaaliaikaiset Kamerat- ja Valosensorit-välilehdet. _Vain Windows._

## [Chloros CLI: Komentoriviliittymä](CLI.md)

Komentorivipohjainen eräkäsittely sekä reaaliaikaiset komennot `lattice`, `daq pool-*`, `project` ja `time-sync`. Sopii erinomaisesti automaatioon, skriptien kirjoittamiseen ja päättömään käyttöön. Saatavilla **Windows issa, Linux amd64:ssa ja Linux arm64:ssa (NVIDIA Jetson)**. _CLI
:n käyttö edellyttää maksullista Chloros+-tasoa._

## [Chloros API: Python SDK](api-python-sdk.md)

Ohjelmoitava Python-rajapinta automaatioon ja mukautettuihin työnkulkuihin: koko prosessiketjun käsittely, reaaliaikaiset kamera-/matriisisessiot, DAQ-anturisessiot ja tallennettujen projektien automaatio. Asennetaan työpöytä-/CLI-paketin mukana ja julkaistaan myös nimellä `pip install chloros-sdk`. _API:n käyttö edellyttää maksullista Chloros+-tasoa._

***

## Tuetut alustat

| Alusta | GUI | CLI | Python-SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | Kyllä | Kyllä | Kyllä |
| **Linux amd64 (x86_64)** | Ei | Kyllä | Kyllä |
| **Linux arm64 (NVIDIA Jetson)** | Ei | Kyllä | Kyllä |

Katso Linux-asennusohjeet kohdasta [Linux ja reunalaskenta](linux/linux-overview.md).

***

## Aloita kolmessa vaiheessa

1. **Asenna** — lataa ja suorita alustallesi sopiva asennusohjelma. Katso [Lataa](download.md).
2. **Kirjaudu sisään (valinnainen GUI:n osalta)** — GUI käsittelee kuvia ilmaiseksi ilman tiliä. [Chloros+ -kirjautuminen](chloros+-login.md) avaa rinnakkaiskäsittelyn, GPU-kiihdytyksen, suuremmat laiterajat sekä CLI- ja SDK-käyttöoikeudet.
3. **Luo ensimmäinen projektisi** — avaa Chloros, luo [Uusi projekti](projects.md), [lisää kuvasi](processing-images-gui/adding-files-to-a-project.md) ja [aloita käsittely](processing-images-gui/starting-the-processing.md). Jos haluat sen sijaan ohjata reaaliaikaista laitteistoa, avaa Kamerat- tai Valosensorit-välilehti — katso [GUI: Navigointi](navigation.md).

***

## Chloros+

Vaikka Chloros on ilmainen useimpiin tehtäviin, saatat huomata, että haluat enemmän. Siinä tapauksessa Chloros+:n maksullinen lisenssi voi olla sinulle hyödyksi. Chloros+ -lisenssillä voit avata uusia ominaisuuksia, kuten:

* **Monisäikeinen käsittely**: nopeuta huomattavasti kuvankäsittelyä suurissa projekteissa käsittelemällä kuvia samanaikaisesti käsittelyputken kautta.
* **GPU (CUDA) -kiihdytys**: hyödynnä nykypäivän suurempia GPU-muistivaihtoehtoja kuvankäsittelyputken nopeuttamiseksi entisestään. Suosittelemme vähintään 4 GB:n VRAM-muistia parhaiden tulosten saavuttamiseksi.
* **Chloros+**[**CLI**](CLI.md) **Käyttö**: suorita Chloros+ komentoriviltä automatisoidaksesi ja integroidaksesi sen omaan ohjelmistoosi. Saatavilla kaikilla maksullisilla tasoilla; toteutetaan palvelinpuolella.
* **Chloros+**[**API**](api-python-sdk.md) **Käyttöoikeus:** Käynnistä Chloros+ Python-kielellä ohjelmoitavaa ohjausta varten, mikä mahdollistaa saumattoman integroinnin tutkimusprosesseihisi, data-analyysin työnkulkuihisi ja mukautettuihin sovelluksiisi. Saatavilla kaikilla maksullisilla tasoilla; valvonta tapahtuu palvelinpuolella.
* **Korkeammat laitteistorajat**: liitä useampia kameroita ja valosensoreita kerralla. Ilman kirjautumista käyttöliittymä tukee enintään 4 kameraa ja 2 DAQ-valosensoria; maksulliset tasot nostavat molempia rajoja:

| Taso | Kamerat | DAQ-valosensorit |
| --- | --- | --- |
| Iron (ilmainen, ei kirjautumista) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Useiden laitteiden käyttö**: jokaisella Chloros+-lisenssillä voi rekisteröidä vähintään 2 laitetta. Hallinnoi rekisteröityjä laitteita MAPIR-pilvipalvelutilisi avulla. Lisää laitteiden tukea päivittämällä Chloros+-lisenssiäsi.
* **Edistyksellinen tekstuuritietoinen debayer-menetelmä:** korkealaatuinen, reunoja tunnistava debayer yhdistettynä tekoäly-/koneoppimismalliin, joka poistaa lähes kaiken debayer-kohinan.
* **Mukautetut monispektriset indeksikaavat:** syötä mukautetut monispektriset indeksit Chloros-rasterilaskureihin sekä käsittelyä että kuvien katselusandboxia varten.
* **Linux ja reunalaskenta:** käytä Chloros ia Linux x86_64- ja ARM64-alustoilla, mukaan lukien NVIDIA Jetson, kenttä- ja reunalaskentaan. Katso [Linux-yleiskatsaus](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+: hinnoittelu ja rekisteröityminen</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.3.0 build so the banner prints "Chloros CLI 1.3.0". -->
