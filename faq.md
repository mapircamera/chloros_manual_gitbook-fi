---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Usein kysyttyjä kysymyksiä

<details>

<summary>Voinko käsitellä Chloros-ohjelmistolla muiden kuin MAPIR-merkkisten kameroiden kuvia?</summary>

Ei, Chloros tukee vain MAPIR-kameroiden kuvien käsittelyä — Survey3- ja LATTICE-tuoteperheitä. Katso lisätietoja [tuettujen kameramallien](supported-cameras.md) luettelosta. Tarjoamme muiden kameroiden kuvien käsittelyä MAPIR-pilvipalvelussa, katso täydellinen luettelo [täältä](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Tukeeko Chloros LATTICE-kameroita?</summary>

Kyllä. Chloros 1.3.0 tukee LATTICE M3C- ja M3M-kameramoduuleja kokonaisuudessaan: **reaaliaikainen hallinta**— tunnista, yhdistä, esikatsele ja tallenna GUI:n Kamerat-välilehdestä, `chloros-cli lattice`, tai Python-SDK:n avulla, mukaan lukien synkronoidut monikamerajärjestelmät PTP-aikasynkronoinnilla — sekä**tallenteiden täydellinen radiometrinen käsittely** (raakadata → debayering → säteilyvoimakkuus → heijastavuus → indeksi). Katso [Tuetut kamerat](supported-cameras.md) ja [LATTICE-opas](lattice/README.md).

</details>

<details>

<summary>Voinko kalibroida kuvani heijastavuuden suhteen ilman kalibrointikohdetta?</summary>

**Survey3:** Ei. Ilman kalibrointikohteen kuvaa, joka on otettu samaan aikaan kuin muut kuvat, et voi yhdistää kuvan pikseliarvoja tunnettuun heijastavuusprosenttiin. Jos et myöskään liitä mukaan MAPIR-valosensorin lokitietoja, ympäristön valospektriä ei mitata, eivätkä heijastavuustulokset ole tarkkoja.

**LATTICE:** Kyllä. Heijastavuus voidaan suhteuttaa paneelin sijaan DAQ-valosensorilla mitattuun alaspäin suuntautuvaan säteilyvoimakkuuteen (ρ = π·L/E). Kun laadunvarmistuksen läpäissyt kohde *on* läsnä kuvakehyksessä, siitä tulee oletusarvoisesti absoluuttinen vertailukohta (`--reflectance-source auto`), ellei se läpäise tarkastusta lippua käyttäen, jolloin kohde ja DAQ-ketjut kilpailevat mitatusta pisteytyksestä. Yksi poikkeus: ”F988-heijastavuus kalibroidaan kuvauskohteessa olevan heijastavuuspaneelin avulla: kaista sijaitsee DAQ-valosensorin kalibroidun alueen ulkopuolella, joten Chloros käyttää viimeisintä paneelista tallennettua arvoa ja säilyttää sen paneelin havaintojen välillä.” Katso [Kalibrointikohteet](calibration-targets.md).

</details>

<details>

<summary>Tarvitsenko DAQ-valosensorin?</summary>

Ei säteilyvoimakkuuden mittaamiseen: LATTICE-säteilyvoimakkuusvienti perustuu kunkin kameran tehtaalla tehtyyn radiometriseen kalibrointiin, eikä se vaadi DAQ-anturia eikä kohdetta. **Heijastavuuden**mittaamiseen tarvitset vertailukohdan ympäristön valolle — joko DAQ-valosensorin alaspäin suuntautuvan valon mittauksen tai kuvan sisällä olevan kalibrointikohteen. DAQ-anturin avulla voit tuottaa kalibroidun heijastavuuden**asettamatta paneeleita kuvauspaikalle**. Tallennetut `.daq`-tiedostot yhdistetään kuvamateriaaliisi automaattisesti aikaleiman perusteella. Katso [Kalibrointikohteet](calibration-targets.md) ja [CLI-ohjeet](reference/cli-reference.md).

</details>

<details>

<summary>Voinko käyttää Chloros-ohjelmistoa tekoälyavustajan (Claude, ChatGPT jne.) kanssa?</summary>

Kyllä — tämä käyttöopas sekä CLI ja SDK on suunniteltu juuri tätä varten:

* Käyttöoppaan täydellinen hakemisto on saatavilla osoitteessa `https://mapir.gitbook.io/chloros/llms.txt`, jotta tekoälyavustajat voivat löytää jokaisen sivun.
* Jokaisen sivun raaka markdown-muotoinen sisältö on saatavilla sivun pienillä kirjaimilla kirjoitetussa URL-osoitteessa, johon on liitetty `.md` (esimerkiksi `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI-ohje](reference/cli-reference.md) ja [SDK-ohje](reference/sdk-reference.md) on kirjoitettu suurten kielimallien (LLM) käyttöön: tarkat liput, oletusarvot, lopetussemantiikka ja kopioitavat komennot.

Katso [AI-avustajat](ai-assistants.md) saadaksesi ohjeet avustajasi ohjaamiseen Chloros-palveluun.

</details>

<details>

<summary>Mihin käsitellyt tulostustiedostot tallennetaan?</summary>

Tulostiedostot tallennetaan projektikansioon, ryhmiteltynä ensin kameran ja sitten tiedostomuodon mukaan:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **kamerakansio** — `LATT-<sensor>-<lens>-F<filter>` (LATTICE), `<model>_<filter>` (esim. `Survey3N_RGN`) (Survey3)
* **tiedostomuoto-kansio** — `tiff16`, `tiff8`, `png8`, `jpg8` tai `tiff32`
* **tuotekansiot** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (aina kansion `tiff32` alla), `<INDEX>_Index_Images/`
* **`clamp_masks/`** — vain LATTICE-monispektrikuvat: pikselikohtaiset naamiot tuotekansioiden vieressä, katso alla**Viedyt tiedostot säilyttävät lähdetiedoston nimen — kansio tunnistaa tuotteen, ei tiedostonimen pääte.**CLI-komennolla projektikansio luodaan oletusprojektikansiosi alle (`Documents/MAPIR/Chloros Projects`, ellet ole muuttanut sitä), ellet anna parametria `-o`; suhteellinen `-o` määritetään suhteessa kansioon, josta komento suoritetaan. Huomaa, että `chloros-cli process`-komennon suoritus, joka pyysi tuotteita mutta ei kirjoittanut yhtään, tulostaa `Processing finished but wrote no image products.` ja**päättyy nollasta poikkeavalla arvolla**, joten skriptit voivat tunnistaa sen. Katso [Tulostuskuvamuodot](output-image-formats.md) ja [CLI-viite](reference/cli-reference.md).

</details>

<details>

<summary>Mitä ovat clamp_masks-kansiot ja *_clampmask.tif-tiedostot?</summary>

Uutta versiossa 1.3.0. Monispektrisen LATTICE M3C -kameran säteily- ja heijastavuustuotteille suoritetaan 3×3-kaistainen erottelu, ja jos erottelussa syntyy negatiivinen arvo jollakin kaistalla, Chloros rajaa kyseisen arvon nollaksi. Tuote, jossa on vähintään yksi leikatun pikseli, saa yksikaistaisen `uint8`-maskin, joka merkitsee, mitkä pikselit ja kaistat on leikattu (bitti 0 = tuotteen R-kanava, bitti 1 = G, bitti 2 = B). Naamioita tallennetaan `clamp_masks/`-kansioon **tuotekansioiden viereen** — esimerkiksi `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif` — ja tuotteen `Chloros:UnmixClampMask` XMP-tunniste nimeää sen maskin (tai siinä lukee `none`).

Ne eivät ole kuvia kohtauksesta: Chlorosin oma tuontitoiminto ohittaa ne, ja koska ne eivät koskaan sijaitse tuotekansiossa, voit siirtää tuotekansion Pix4D:lle tai Metashapelle sellaisenaan. Katso [Kiinnitysmaskit](output-image-formats.md#clamp-masks).

</details>

<details>

<summary>Miksi Chloros ilmoittaa: ”Projektikansio on liian syvällä tässä tietokoneessa”?</summary>

Tämä koskee vain Windowsia. Windowsissa pitkät polut ovat oletusarvoisesti pois päältä, jolloin yksikään tiedostopolku ei saa olla pidempi kuin 260 merkkiä. Chloros-tuotteet sijaitsevat jopa noin 130 merkkiä projektikansion alapuolella, joten Chloros hylkää syvemmän projektikansion ennen käsittelyä — sen sijaan, että se suorittaisi ohjelman ja jättäisi kirjoittamatta mitään, kuten saattoi tapahtua ennen versiota 1.3.0. Viesti ilmoittaa pisimmän sopivan projektikansion.

Korjaa ongelma jommallakummalla tavalla:

* Käytä lyhyempää projektikansiota (**Työkansio** projektiasetuksissa tai `-o` / `chloros-cli set-project-folder` CLI-käytössä).
* Ota Windowsin pitkät polut käyttöön ja käynnistä Chloros uudelleen. Chloros-asennusohjelman **Enable Windows long paths** -vaihtoehto (valittuna oletuksena) hoitaa tämän; suorita asennusohjelma uudelleen, jos olet poistanut valinnan.

Katso [Käsittelyn aloittaminen](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

</details>

<details>

<summary>Voinko muokata kuviani ennen niiden käsittelyä Chloros-ohjelmassa?</summary>

Ei. Chloros olettaa, että syöttötietoja ei ole muokattu. Älä muuta tiedostojen nimiä.

</details>

<details>

<summary>Voinko asettaa MAPIRSurvey3-kamerani automaattiselle valotukselle ja käsitellä kuvat Chloros-ohjelmassa?</summary>

Ei. Survey3-kuvasarjojen valotuksen on oltava kiinteä/lukittu, joten automaattista suljinnopeutta tai automaattista ISO-arvoa ei saa käyttää. Kaikilla saman kameramallin kuvilla on oltava identtinen suljinnopeus ja ISO-arvo (valotus).

LATTICE-kameroilla ei ole tätä rajoitusta: Chloros ohjaa niiden valotusta reaaliaikaisesti (Smart AE), ja jokainen otos tallentaa tosiasiallisesti käytetyn valotuksen ja vahvistuksen, jotka radiometrinen käsittelyputki ottaa huomioon.

</details>

<details>

<summary>Voiko Chloros-ohjelmisto käsitellä tai analysoida ortomosaiikkikuvia?</summary>

Ei. Vain yksittäisiä MAPIR-kamerakuvia tuetaan, ei yhdistettyjä kuvia, kuten ortomosaiikkikarttaa.

</details>

<details>

<summary>Miten voin nopeuttaChloros-ohjelmiston kohteen tunnistamisvaihetta?</summary>

Kun valitset tiedostoselaimen taulukon oikeassa sarakkeessa kohdekuvat etukäteen, Chloros etsii kalibrointikohteita vain näistä kuvista, mikä nopeuttaa käsittelyä huomattavasti.

</details>

<details>

<summary>Jos aion ladata kuvani <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR-pilvipalveluun,</a> pitäisikö minun käsitellä ne Chloros-ohjelmassa ennen lataamista?</summary>

Jos aiot ladata kuvat online-käsittelyalustallemme [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), älä muokkaa kuvia ennen lataamista. Cloud suorittaa kaikki samat käsittelyt ja paljon muuta.

</details>

<details>

<summary>Tuleeko MAPIR koskaan tukemaan X-ominaisuutta? Toivoisin todella, että MAPIR tarjoaisi X:n.</summary>

Olemme aina kiinnostuneita saamaan palautetta tuotteistamme. Jos huomaat ongelman tuotteissamme tai sinulla on ehdotus siitä, miten voimme parantaa tuotteitamme, ota [YHTEYTTÄ](https://www.mapir.camera/community/contact) ja kerro ajatuksesi. Suurin osa tutkimus- ja kehitystyöstämme perustuu asiakkaidemme tärkeimpien tarpeiden kuuntelemiseen.

</details>

<details>

<summary>OnkChlorossaatavilla Linuxille?</summary>

Kyllä! Chloros 1.3.0 tukee Linux-alustoja amd64 (x86_64) ja arm64 (NVIDIA Jetson JetPack 6) `.deb`-pakettien kautta. CLI ja Python-SDK ovat täysin tuettuja Linuxissa, mukaan lukien LATTICE-kameran ja DAQ-anturien reaaliaikainen ohjaus. Linuxille ei ole graafista käyttöliittymää — kaikki vuorovaikutus tapahtuu [CLIn](CLI.md) tai [Python-SDK:n](api-python-sdk.md) kautta. Katso lisätietoja [Linux-yleiskatsauksesta](linux/linux-overview.md).

</details>

<details>

<summary>Voinko käyttää Chlorosia NVIDIA Jetsonilla?</summary>

Kyllä! Chloros tukee NVIDIA Jetsonin **Orin-tuoteperhettä** — Orin Nano, Orin NX ja AGX Orin — joissa on JetPack 6. Vanhemmat mallit (Jetson Nano, TX2, Xavier NX) eivät tue JetPack 6:ta, eikä nykyinen paketti tue niitä. Chloros tunnistaa Jetson-mallisi automaattisesti ja optimoi sen käsittelystrategian. Jetson-paketti käyttää JetPack 6:n CUDA-rakennetta PyTorchista, joten käsittely tapahtuu Jetsonin GPU:lla. Katso [NVIDIA Jetson -opas](linux/nvidia-jetson-guide.md) asennus- ja käyttöönotto-ohjeiden saamiseksi.

</details>

<details>

<summary>Optimoiko Chloros järjestelmän automaattisesti laitteistolleni sopivaksi?</summary>

Kyllä! Chloros sisältää [dynaamisen laskentasovituksen](processing-architecture/dynamic-compute-adaptation.md), joka tunnistaa automaattisesti CPU:n, GPU:n, RAM-muistin ja (Jetsonissa) lämpötila-anturit. Sen jälkeen se valitsee optimaalisen käsittelystrategian — alkaen `GPU_PARALLEL`:stä suurimuistisissa GPU:issa, `GPU_SINGLE`:stä pienemmissä GPU:issa ja päättyen `CPU_PARALLEL`:ään järjestelmissä, joissa ei ole CUDA-GPU:ta, jota PyTorch voisi käyttää. Manuaalista konfigurointia ei tarvita.

</details>

<details>

<summary>Mikä on 4-säikeinen käsittelyputki?</summary>

Chloros käyttää 4-säikeistä putkiarkkitehtuuria Chloros+-käyttäjille: Säie 1 (tunnistus) lataa kuvat ja tunnistaa kalibrointikohteet, säie 2 (kalibrointi) laskee heijastavuuskalibroinnin, säie 3 (käsittely) suorittaa GPU-kiihdytetyn debayeringin ja indeksin laskennan, ja säie 4 (vienti) kirjoittaa tulostustiedostot. Useita kuvia voi olla samanaikaisesti eri säikeissä maksimaalisen suorituskyvyn saavuttamiseksi. Katso lisätietoja kohdasta [Käsittelyputki](processing-architecture/processing-pipeline.md).

</details>

<details>

<summary>Miten suoritan Chloros-asennuksen diagnostiikan?</summary>

Käytä komentoa `selftest` suorittaaksesi 7-vaiheisen savutestin: versio, porttien saatavuus, taustapalvelun käynnistys, API-yhteys (`/api/test`), järjestelmätiedot (`/api/system-info` — GPU/CUDA/PyTorch), kohinanpoistomallin läsnäolo sekä CUDA:n ja kohinanpoistomallin käyttövalmius:

```bash
chloros-cli selftest
```

Tämä on erityisen hyödyllistä Linux-järjestelmissä GPU:n ja CUDA:n asetusten tarkistamiseksi. Jetson-laitteella `CUDA: False` tarkoittaa yleensä, että JetPack 6:n CUDA- ja cuDNN-kirjastot puuttuvat: asenna ne komennolla `sudo apt install nvidia-jetpack`.

</details>
