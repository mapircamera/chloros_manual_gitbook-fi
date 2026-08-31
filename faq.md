---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Usein kysytyt kysymykset

<details>

<summary>Voinko käsitellä MAPIR-merkkisten kameroiden kuvia Chloros-ohjelmistolla?</summary>

Ei, Chloros tukee vain MAPIR-kameroiden kuvien käsittelyä — Survey3- ja LATTICE-tuoteperheitä. Katso lisätietoja [tuettujen kameramallien](supported-cameras.md) luettelosta. Tarjoamme muiden kameroiden kuvien käsittelyä MAPIR Cloud -palvelussa; katso täydellinen luettelo [täältä](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Tukeeko Chloros LATTICE-kameroita?</summary>

Kyllä. Chloros 1.2.0 tukee LATTICE M3C- ja M3M-kameramoduuleja kokonaisuudessaan: **reaaliaikainen hallinta**— tunnistaminen, yhdistäminen, esikatselu ja tallennus GUI:n Kamerat-välilehdeltä, `chloros-cli lattice`:n tai Python:n SDK:n kautta, mukaan lukien synkronoidut monikamerajärjestelmät PTP-aikasynkronoinnilla — sekä**tallenteiden täydellinen radiometrinen käsittely** (raakadata → debayering → säteilyvoimakkuus → heijastavuus → indeksi). Katso [Tuetut kamerat](supported-cameras.md) ja [LATTICE-opas](lattice/README.md).

</details>

<details>

<summary>Voinko kalibroida kuviani heijastavuuden suhteen ilman kalibrointikohdetta?</summary>

**Survey3:** Ei. Ilman kalibrointikohteen kuvaa, joka on otettu samaan aikaan kuin muut kuvat, et voi yhdistää kuvan pikseliarvoja tunnettuun heijastavuusprosenttiin. Jos et myöskään liitä mukaan MAPIR-valosensorin lokitietoja, ympäristön valospektriä ei mitata, eivätkä heijastavuustulokset ole tarkkoja.**LATTICE:** Kyllä. Heijastavuutta voidaan verrata paneelin sijaan DAQ-valosensorilla mitattuun alaspäin suuntautuvaan säteilyvoimakkuuteen (ρ = π·L/E). Kun kuvakehyksessä *on* läsnä laadunvarmistuksen läpäissyt kohde, siitä tulee oletusarvoisesti absoluuttinen vertailukohta (`--reflectance-source auto`). Yksi poikkeus: ”F988-heijastavuus kalibroidaan kuvauskohteessa olevan heijastavuuspaneelin avulla: kaista sijaitsee DAQ-valosensorin kalibroidun alueen ulkopuolella, joten Chloros käyttää viimeisintä paneelista tallennettua arvoa ja säilyttää sen paneelin havaintojen välillä.” Katso [Kalibrointikohteet](calibration-targets.md).

</details>

<details>

<summary>Tarvitsenko DAQ-valosensorin?</summary>

Ei säteilyvoimakkuuden mittaamiseen: LATTICE-säteilyvoimakkuustiedot perustuvat kunkin kameran tehtaalla suoritettuun radiometriseen kalibrointiin, eivätkä ne vaadi DAQ-anturia tai kalibrointikohdetta. **Heijastavuuden**mittaamiseen tarvitset vertailukohdan ympäristön valolle — joko DAQ-valosensorin alaspäin suuntautuvan valon mittauksen tai kuvan sisällä olevan kalibrointikohteen. DAQ-anturin avulla voit tuottaa kalibroidun heijastavuuden**ilman, että kuvauspaikalle tarvitsee sijoittaa paneeleita**. Tallennetut `.daq`-tiedostot yhdistetään kuvamateriaaliisi automaattisesti aikaleiman perusteella. Katso [Kalibrointikohteet](calibration-targets.md) ja [CLI-viite](reference/cli-reference.md).

</details>

<details>

<summary>Voinko käyttää Chloros-ohjelmaa tekoälyavustajan (Claude, ChatGPT jne.) kanssa?</summary>

Kyllä — tämä käyttöopas sekä CLI/SDK on suunniteltu juuri sitä varten:

* Käyttöoppaan täydellinen hakemisto on saatavilla osoitteessa `https://mapir.gitbook.io/chloros/llms.txt`, jotta tekoälyavustajat voivat löytää jokaisen sivun.
* Jokaisen sivun raakamuotoinen Markdown-koodi on saatavilla sivun pienillä kirjaimilla kirjoitetulla sivulla URL, johon on liitetty `.md` (esimerkiksi `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI-viite](reference/cli-reference.md) ja [SDK-viite](reference/sdk-reference.md) on kirjoitettu suurten kielimallien (LLM) käyttöön: tarkat liput, oletusarvot, lopetussemantiikka ja kopioitavat komennot.

Katso [AI-avustajat](ai-assistants.md) saadaksesi ohjeet avustajasi ohjaamiseen Chloros:ään.

</details>

<details>

<summary>Mihin käsitellyt tulostustiedostoni tallennetaan?</summary>

Tulostiedostot tallennetaan projektikansioon, ryhmiteltynä kameran ja sitten tiedostomuodon mukaan:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **kamerakansio** — `LATT-<sensor>-<lens>-F<filter>` LATTICE-tiedostoille, `<model>_<filter>` (esim. `Survey3N_RGN`) Survey3-tiedostoille
* **tiedostomuoto-kansio** — `tiff16`, `tiff8`, `png8`, `jpg8` tai `tiff32`
* **tuotekansiot** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (aina `tiff32`:n alla), `<INDEX>_Index_Images/`**Viedyt tiedostot säilyttävät lähdetiedoston nimen — kansio määrittää tuotteen, ei tiedostonimen pääte.**CLI:n tapauksessa projektikansio luodaan syöttökansion viereen, ellet anna parametria `-o`. Huomaa, että `chloros-cli process`-ajo, joka pyysi tuotteita mutta ei kirjoittanut yhtään, tulostaa `Processing finished but wrote no image products.` ja**päättyy nollasta poikkeavalla arvolla**, joten skriptit voivat tunnistaa sen. Katso [Tulostuskuvamuodot](output-image-formats.md) ja [CLI-viite](reference/cli-reference.md).

</details>

<details>

<summary>Voinko muokata kuviani ennen niiden käsittelyä Chloros:ssä?</summary>

Ei. Chloros olettaa, että syöttötietoja ei ole muokattu. Älä muuta tiedostojen nimiä.

</details>

<details>

<summary>Voinko asettaa MAPIR- ja Survey3-kamerani automaattiselle valotukselle ja käsitellä kuvat ohjelmassa Chloros?</summary>

Ei. Survey3-kuvasarjoissa valotuksen on oltava kiinteä/lukittu, joten automaattista suljinnopeutta tai automaattista ISO-arvoa ei saa käyttää. Kaikilla saman kameramallin kuvilla on oltava identtinen suljinnopeus ja ISO-arvo (valotus).

LATTICE-kameroilla ei ole tätä rajoitusta: Chloros säätää valotusta reaaliaikaisesti (Smart AE), ja jokaisessa kuvassa tallennetaan todellisuudessa käytetty valotus ja vahvistus, jotka radiometrinen käsittelyputki ottaa huomioon.

</details>

<details>

<summary>Voiko Chloros käsitellä tai analysoida ortomosaiikkikuvia?</summary>

Ei. Ohjelma tukee vain yksittäisiä MAPIR-kamerakuvia, ei ortomosaiikkikartan kaltaisia yhdistettyjä kuvia.

</details>

<details>

<summary>Miten voin nopeuttaa Chloros:n kohteen tunnistusvaihetta?</summary>

Kun valitset tiedostoselaimen taulukon oikeassa sarakkeessa kohdekuvat etukäteen, Chloros etsii kalibrointikohteita vain noista kuvista, mikä nopeuttaa käsittelyä huomattavasti.

</details>

<details>

<summary>Jos aion ladata kuvani <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud</a> -palveluun, pitäisikö minun käsitellä ne Chloros:ssa ennen lataamista?</summary>

Jos aiot ladata kuvat online-käsittelyalustallemme [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), älä muokkaa kuvia ennen lataamista. Cloud suorittaa kaikki samat käsittelyt ja paljon muuta.

</details>

<details>

<summary>Tuleeko MAPIR koskaan tukemaan X-ominaisuutta? Toivoisin todella, että MAPIR tarjoaisi X:n.</summary>

Olemme aina kiinnostuneita saamaan palautetta tuotteistamme. Jos huomaat ongelman tuotteissamme tai sinulla on ehdotuksia siitä, miten voimme parantaa tuotteitamme, ota meihin yhteyttä [OTTAA YHTEYTTÄ](https://www.mapir.camera/community/contact) ja kerro ajatuksesi. Suurin osa tutkimus- ja kehitystyöstämme perustuu asiakkaidemme tärkeimpien tarpeiden kuuntelemiseen.

</details>

<details>

<summary>Onko Chloros saatavilla Linux:lle?</summary>

Kyllä! Chloros 1.2.0 tukee Linux amd64 (x86_64) ja arm64 (NVIDIA Jetson JetPack 6) -alustoja `.deb`-pakettien kautta. CLI ja Python SDK ovat täysin tuettuja Linux:ssa, mukaan lukien reaaliaikainen LATTICE-kameran ja DAQ-anturin ohjaus. Linux:lle ei ole graafista käyttöliittymää — kaikki vuorovaikutus tapahtuu [CLI](CLI.md) tai [Python SDK](api-python-sdk.md) kautta. Katso lisätietoja [Linux-yleiskatsauksesta](linux/linux-overview.md).

</details>

<details>

<summary>Voinko käyttää Chloros:ää NVIDIA Jetsonilla?</summary>

Kyllä! Chloros tukee NVIDIA Jetson -alustoja, mukaan lukien Jetson Nano, Orin Nano, Orin NX ja AGX Orin, joissa on käytössä JetPack 6. Chloros tunnistaa automaattisesti Jetson-mallisi ja optimoi sen käsittelystrategian. Katso asennus- ja käyttöönotto-ohjeet [NVIDIA Jetson -oppaasta](linux/nvidia-jetson-guide.md).

</details>

<details>

<summary>Optimoiko Chloros automaattisesti laitteistolleni sopivaksi?</summary>

Kyllä! Chloros sisältää [dynaamisen laskentasovituksen](processing-architecture/dynamic-compute-adaptation.md), joka tunnistaa automaattisesti CPU:n, GPU:n, RAM-muistin ja (Jetsonissa) lämpötila-anturit. Sen jälkeen se valitsee optimaalisen prosessointistrategian – alkaen `GPU_PARALLEL`:stä suurimuistisissa järjestelmissä, `GPU_SINGLE`:stä rajoitetuissa laitteissa ja päättyen `CPU_PARALLEL`:ään järjestelmissä, joissa ei ole NVIDIA-grafiikkaprosessoria. Manuaalista konfigurointia ei tarvita.

</details>

<details>

<summary>Mikä on 4-säikeinen prosessointiputki?</summary>

Chloros käyttää 4-säikeistä putkitetun arkkitehtuuria Chloros+ -käyttäjille: Säie 1 (tunnistus) lataa kuvat ja tunnistaa kalibrointikohteet, säie 2 (kalibrointi) laskee heijastavuuskalibroinnin, säie 3 (käsittely) suorittaa GPU-kiihdytetyn debayeringin ja indeksin laskennan ja säie 4 (vienti) kirjoittaa tulostustiedostot. Useita kuvia voi olla samanaikaisesti eri säikeissä, mikä takaa maksimaalisen suorituskyvyn. Katso lisätietoja kohdasta [Käsittelyputki](processing-architecture/processing-pipeline.md).

</details>

<details>

<summary>Miten suoritan diagnostiikan Chloros-asennuksellani?</summary>

Käytä komentoa `selftest` suorittaaksesi 7-vaiheisen smoke-testin: versio, porttien saatavuus, taustapalvelun käynnistys, API-yhteys (`/api/test`), järjestelmätiedot (`/api/system-info` — GPU/CUDA/PyTorch), kohinanpoistomallin olemassaolo sekä CUDA:n ja kohinanpoistomallin käyttövalmius:

```bash
chloros-cli selftest
```

Tämä on erityisen hyödyllistä Linux-/Jetson-järjestelmissä GPU:n ja CUDA:n asetusten tarkistamiseksi.

</details>
