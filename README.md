---
metaLinks: {}
---

# Aloitusopas

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros on ohjelmistosovellus, jonka on kehittänyt [MAPIR](https://www.mapir.camera), jonka avulla voidaan käsitellä monispektrikuvia, ohjata MAPIR-laitteistoa reaaliaikaisesti ja tallentaa anturitietoja. Chloros 1.2.0 tukee koko MAPIR-tuoteperhettä:

* **Survey3-kamerat** — käsittele RAW+JPG-kuvaukset kalibroiduiksi heijastavuus- ja kasvillisuusindeksikartoiksi. Katso [Tuetut kamerat](supported-cameras.md).
* **LATTICE-kamerat** — yhdistä GigE-monispektrikameramoduulit reaaliaikaisesti, yksittäin tai synkronoituina monikamerajärjestelminä: esikatsele, tallenna ja käsittele kalibroiduiksi säteily- ja heijastavuustuotteiksi. Katso [LATTICE-osio](lattice/README.md).
* **DAQ-valosensorit** — DAQ-U (USB), DAQ-M (Bluetooth) ja DAQ-E (Ethernet) spektrisensorit: reaaliaikaiset kalibroidut spektrit, `.daq`-tallenteet ja alaspäin suuntautuva valaistus heijastavuuden käsittelyä varten. Katso [DAQ-osio](daq/README.md).

{% hint style="success" %}
**Uutta versiossa Chloros 1.2.0**: LATTICE-kameran ja -anturiryhmän reaaliaikainen ohjaus, DAQ-valosensorien integrointi, tallennustilat ja tallentimet, täydellinen LATTICE-radiometrinen käsittelyputki, projektien automatisointi CLI/SDK:sta ja paljon muuta. Katso alla oleva Uutta-luettelo ja [Lataa](download.md) muutospäiväkirja.
{% endhint %}

{% hint style="info" %}
**Käytätkö Chloros:ää tekoälyavustajan kanssa?** Tämä käyttöopas on suunniteltu juuri sitä varten. Ohjaa avustajasi seuraaviin:

* `https://mapir.gitbook.io/chloros/llms.txt` — koneellisesti luettava hakemisto jokaisesta sivusta.
* Mikä tahansa sivu raakamuodossa (Markdown) — liitä `.md` sen URL-tunnisteen perään (esim. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI-viite](reference/cli-reference.md) ja [SDK-viite](reference/sdk-reference.md) — täydelliset, tarkkoja arvoja sisältävät viitesivut, jotka on kirjoitettu LLM:n käyttöön.

Esimerkki kehotteesta: *&quot;Lue https://mapir.gitbook.io/chloros/reference/cli-reference.md, ja kirjoita sitten skripti, joka kirjautuu sisään ja käsittelee kansion ~/flights/flight_001 heijastavuusarvoiksi + NDVI GeoTIFF-tiedostoiksi.&quot;*

Kattava opas: [Chloros:n käyttö tekoälyavustajien kanssa](ai-assistants.md).
{% endhint %}

***

## Uutta versiossa Chloros 1.2.0

* **Kameran reaaliaikainen hallinta — uusi Kamerat-välilehti.** Liitä LATTICE-kamerat yksitellen tai synkronoituina monikamerajärjestelminä (PTP-aikasynkronointi, laitteistokäynnistetty kuvaus), joissa on reaaliaikaiset esikatselupäällysteet, kaistakohtaiset histogrammit, älykäs automaattinen valotus, reaaliaikainen indeksilaskuri ja sovelluksen sisäiset kameran laiteohjelmistopäivitykset.
* **Valoanturit — uusi Valoanturit-välilehti.** Liitä DAQ-U (USB), DAQ-M (Bluetooth) ja DAQ-E (Ethernet) -anturit; tarkastele kalibroituja reaaliaikaisia spektrejä (W/m²/nm), tallenna `.daq`-tiedostoja projektiisi, valitse kap-korjausprofiileja ja päivitä DAQ-E-laiteohjelmisto verkon kautta.
* **Tallennustilat ja tallentimet.** Yksittäinen / Jatkuva / Intervallitallennus sekä pelkästään raakadataa käyttävä Nopein tallennustila; projektikohtainen valinta siitä, mitä kameroita ja vientityyppejä ”Tallenna kaikki” tuottaa; matriisitallentimet seurantatasoiseen indeksivideoon ja analyysitasoisiin raakadatan sarjoihin, joissa on offline-videokoosteet.
* **LATTICE-käsittelyputki.** Tuo LATTICE-tallennuskansiot ja jaa jokainen raakakuva debayeroituihin, esikatselu-, float32-säteilyvoimakkuus- (W/m²/sr/nm) ja heijastavuus-tuotteisiin, joissa on tuotekohdaiset kytkimet. Heijastavuusarvo voi perustua kehyksessä olevaan kalibrointikohteeseen tai DAQ-mittaukseen; vientiin sovelletaan matriisin kohdistusta; puuttuva tehdaskalibrointi ladataan automaattisesti kameran sarjanumeron perusteella.
* **Projektit muistavat laitteiston.** Liitetyt kamerat ja valosensorit tallennetaan projektin mukana (`cameras.json` / `sensors.json`), ja ne muodostavat yhteyden uudelleen tallennetuilla asetuksillaan, kun avaat projektin uudelleen. Katso [GUI: Projektit](projects.md).
* **Kuvankatseluohjelman parannukset.** Kursorin pikseli-/indeksilukema, jossa on tiedostokohtainen oikea heijastavuusskaalaus, kerrosten histogrammit, GSD-binning-liukusäädin, Per Trigger / Per Camera -ruudukkomoodit, LATTICE-tuotenäkymät sekä indeksi-/LUT-sandbox-vienti levylle.
* **CLI ja SDK, huomattavasti laajennettu.** Uudet `lattice`-, `daq pool-*`-, `project`- ja `time-sync`-komentoryhmät; uudet `process`-vaihtoehdot (`--input-level`, tuotekohtaiset kytkimet, `--reflectance-source`, taulukon kohdistusliput); SDK-älyliitäntäkäsittelyt (`connect_camera` / `connect_array` / `connect_daq_sensor`), jotka käynnistävät taustapalvelimen automaattisesti; `open_project()`-automaatio; SDK-pyörä on mukana asennusohjelmissa ja julkaistu PyPI:ssä nimellä `chloros-sdk`.
* **Rehellinen virhesemantiikka.** `chloros-cli process`-ajo, joka pyysi tuotteita mutta ei kirjoittanut yhtään, epäonnistuu nyt selvästi ja päättyy nollasta poikkeavalla tuloksella; onnistuneet ajot raportoivat, kuinka monta kuvatuotetta ne kirjoittivat.
* **Uusi tulosteiden rakenne.** Tuotteet tallennetaan `<project>/<camera>/<format>/<Product>_Images/`-kansioihin, ja ne säilyttävät lähdetiedoston nimen — kansio, ei tiedostonimen pääte, tunnistaa tuotteen. Katso [Tulostuskuvamuodot](output-image-formats.md).
* **Lisää syötteitä, suunnitelmia ja kieliä.** `.dng`-syötteen tuki; kaikki 38 käyttöliittymäkieltä täysin käytettävissä; suunnitelmakohtaiset laitteistorajoitukset, joissa ilmaista (kirjautumista vaativaa) käyttöä varten on käytettävissä enintään 4 kameraa ja 2 valosensoria.
* **Luotettavuus.** Käsittelyn lopettaminen päättyy siististi ja rehellisellä suoritusyhteenvedolla, monikameraprojektit vievät tiedot jokaisesta kamerasta, eikä asennusohjelman päivitys enää kirjaa sinua ulos.***

Chloros on saatavilla kolmella käyttöliittymäalustalla:

## Chloros: Työpöydän graafinen käyttöliittymä

Erillinen ikkuna, jossa on kaikki ominaisuudet, mukaan lukien reaaliaikaiset Kamerat- ja Valosensorit-välilehdet. _Vain Windows._

## [Chloros CLI: Komentoriviliitäntä](CLI.md)

Komentoriviltä suoritettava eräkäsittely sekä reaaliaikaiset komennot `lattice`, `daq pool-*`, `project` ja `time-sync`. Sopii erinomaisesti automaatioon, skriptien luomiseen ja päättömään käyttöön. Saatavilla **Windows-, Linux amd64- ja Linux arm64 (NVIDIA Jetson)** -versioissa. _CLI:n käyttö edellyttää maksullista Chloros+-tasoa._

## [Chloros API: Python SDK](api-python-sdk.md)

Ohjelmoitava Python-rajapinta automaatioon ja mukautettuihin työnkulkuihin: koko prosessiputken käsittely, reaaliaikaiset kamera-/matriisisessiot, DAQ-anturisessiot ja tallennettujen projektien automaatio. Asennetaan työpöytä-/CLI-paketin mukana ja julkaistaan myös nimellä `pip install chloros-sdk`. _API:n käyttö edellyttää maksullista Chloros+-tasoa._

***

## Tuetut alustat

| Alusta | Käyttöliittymä | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | Kyllä | Kyllä | Kyllä |
| **Linux amd64 (x86_64)** | Ei | Kyllä | Kyllä |
| **Linux arm64 (NVIDIA Jetson)** | Ei | Kyllä | Kyllä |

Linux:n asennusohjeet löytyvät kohdasta [Linux ja reunalaskenta](linux/linux-overview.md).

***

## Aloita kolmessa vaiheessa

1. **Asenna** — lataa ja suorita alustallesi tarkoitettu asennusohjelma. Katso [Lataa](download.md).
2. **Kirjaudu sisään (valinnainen GUI:n osalta)** — GUI käsittelee kuvia ilmaiseksi ilman tiliä. [Chloros+ -kirjautuminen](chloros+-login.md) avaa rinnakkaisprosessoinnin, GPU-kiihdytyksen, suuremmat laiterajat sekä CLI/SDK-käyttöoikeudet.
3. **Luo ensimmäinen projektisi** — avaa Chloros, luo [Uusi projekti](projects.md), [lisää kuvasi](processing-images-gui/adding-files-to-a-project.md) ja [aloita käsittely](processing-images-gui/starting-the-processing.md). Jos haluat sen sijaan ohjata reaaliaikaista laitteistoa, avaa Kamerat- tai Valosensorit-välilehti — katso [GUI: Navigointi](navigation.md).***

## Chloros+

Vaikka Chloros on ilmainen useimpiin tehtäviin, saatat huomata, että tarvitset enemmän. Siinä tapauksessa Chloros+:n maksullinen lisenssi voi olla sinulle hyödyksi. Chloros+-lisenssillä voit avata uusia ominaisuuksia, kuten:

* **Monisäikeinen käsittely**: nopeuta huomattavasti kuvankäsittelyä suurissa projekteissa käsittelemällä kuvia samanaikaisesti käsittelyputkessa.
* **GPU (CUDA) -kiihdytys**: hyödynnä nykypäivän suurempia GPU-muistivaihtoehtoja kuvankäsittelyputken nopeuttamiseksi entisestään. Suosittelemme vähintään 4 GB:n VRAM-muistia parhaiden tulosten saavuttamiseksi.
* **Chloros+**[**CLI**](CLI.md)**Pääsy**: suorita Chloros+ komentoriviltä automatisoidaksesi ja integroidaksesi sen omaan ohjelmistoosi. Saatavilla kaikilla maksullisilla tasoilla; valvoo palvelinpuoli.
* **Chloros+**[**API**](api-python-sdk.md)**Käyttö:** suorita komento Chloros+ komennosta Python ohjelmoitua ohjausta varten, mikä mahdollistaa saumattoman integroinnin tutkimusprosesseihisi, data-analyysin työnkulkuihisi ja mukautettuihin sovelluksiisi. Saatavilla kaikilla maksullisilla tasoilla; valvonta tapahtuu palvelinpuolella.
* **Korkeammat laitteistorajat**: liitä useampia kameroita ja valosensoreita kerralla. Ilman kirjautumista käyttöliittymä tukee enintään 4 kameraa ja 2 DAQ-valosensoria; maksulliset tasot nostavat molempia rajoja:

| Taso | Kamerat | DAQ-valosensorit |
| --- | --- | --- |
| Iron (ilmainen, ei kirjautumista) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Usean laitteen käyttö**: jokaisella Chloros+-lisenssillä voi rekisteröidä vähintään 2 laitetta. Hallitse rekisteröityjä laitteita MAPIR Cloud -tilisi avulla. Lisää laitetukea päivittämällä Chloros+-lisenssisi.
* **Edistyksellinen tekstuuritietoinen debayer-menetelmä:** korkealaatuinen, reunoja tunnistava debayer yhdistettynä AI/ML-kohinanpoistomalliin, joka poistaa lähes kaiken debayer-kohinan.
* **Mukautetut monispektriset indeksikaavat:** syötä mukautetut monispektriset indeksit Chloros-rasterilaskureihin sekä käsittelyä että kuvien katselusandboxia varten.
* **Linux ja reunalaskenta:** aja Chloros:ää Linux x86_64- ja ARM64-alustoilla, mukaan lukien NVIDIA Jetson, kenttä- ja reunalaskentaa varten. Katso [Linux:n yleiskatsaus](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+: hinnoittelu ja rekisteröityminen</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.2.0 build so the banner prints "Chloros CLI 1.2.0". -->
