---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Usein kysyttyjä kysymyksiä

<details>

<summary>Voinko käsitellä MAPIR-merkkisten kameroiden kuvia Chloros-ohjelmistolla?</summary>

Ei, Chloros tukee vain MAPIR-kamerakuvien käsittelyä. Katso lisätietoja [tuettujen kameramallien](supported-cameras.md) luettelosta. Tarjoamme muiden kameroiden kuvien käsittelyä MAPIR Cloud -palvelussa, katso täydellinen luettelo [täältä](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Voinko kalibroida kuvieni heijastavuuden ilman kalibrointikohdetta?</summary>

Ei. Ilman kalibrointikohteen kuvaa, joka on otettu samaan aikaan kuin muut kuvat, et voi yhdistää kuvan pikseliarvoja tunnettuun heijastavuusprosenttiin. Jos et myöskään liitä mukaan MAPIR-valosensorin lokia, ympäristön valospektriä ei mitata, eivätkä heijastavuustulokset ole tarkkoja.

</details>

<details>

<summary>Voinko muokata kuviani ennen niiden käsittelyä Chloros-ohjelmassa?</summary>

Ei. Chloros olettaa, että syöttötietoja ei ole muokattu. Älä muuta tiedostojen nimiä.

</details>

<details>

<summary>Voinko asettaa MAPIR- ja Survey3-kamerani automaattiselle valotukselle ja käsitellä kuvat Chloros-ohjelmassa?</summary>

Ei. Survey3-kuvasarjojen valotuksen on oltava kiinteä/lukittu, joten automaattista suljinnopeutta tai automaattista ISO-arvoa ei voi käyttää. Kaikilla saman kameramallin kuvilla on oltava identtinen suljinnopeus ja ISO-arvo (valotus).

</details>

<details>

<summary>Voiko Chloros käsitellä tai analysoida ortomosaiikkikuvia?</summary>

Ei. Ohjelma tukee vain yksittäisiä MAPIR-kamerakuvia, ei ortomosaiikkikartan kaltaisia yhdistettyjä kuvia.

</details>

<details>

<summary>Kuinka voin nopeuttaa Chloros:n kohteen tunnistamisvaihetta?</summary>

Tiedostoselaimen taulukossa oikean sarakkeen kohdekuvien esivalinta kertoo Chloros:lle, että se etsii kalibrointikohteita vain kyseisistä kuvista, mikä nopeuttaa käsittelyä huomattavasti.

</details>

<details>

<summary>Jos aion ladata kuvani <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud</a> -palveluun, pitäisikö minun käsitellä ne Chloros:ssä ennen lataamista?</summary>

Jos aiot ladata kuvat online-käsittelyalustallemme [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), älä muokkaa kuvia ennen lataamista. Cloud suorittaa kaikki samat käsittelyt ja paljon muuta.

</details>

<details>

<summary>Tuleeko MAPIR koskaan tukemaan X-ominaisuutta? Toivon todella, että MAPIR tarjoaisi X:n.</summary>

Olemme aina kiinnostuneita saamaan palautetta tuotteistamme. Jos löydät ongelman tuotteissamme tai sinulla on ehdotus siitä, miten voimme parantaa tuotteitamme, ota meihin yhteyttä [CONTACT US](https://www.mapir.camera/community/contact) ja jaa ajatuksesi. Suurin osa tutkimus- ja kehitystyöstämme perustuu asiakkaidemme tärkeimpien tarpeiden kuuntelemiseen.

</details>

<details>

<summary>Onko Chloros saatavilla Linux:lle?</summary>

Kyllä! Chloros 1.1.0 tukee Linux amd64 (x86_64) ja arm64 (NVIDIA Jetson JetPack 6) -alustoja `.deb`-pakettien kautta. CLI ja Python SDK ovat täysin tuettuja Linux:ssä. Linux:lle ei ole graafista käyttöliittymää — kaikki vuorovaikutus tapahtuu [CLI](CLI.md):n tai [Python SDK](api-python-sdk.md). Katso lisätietoja kohdasta [Linux Yleiskatsaus](linux/linux-overview.md).

</details>

<details>

<summary>Voinko käyttää Chloros:ää NVIDIA Jetsonissa?</summary>

Kyllä! Chloros 1.1.0 tukee NVIDIA Jetson -alustoja, mukaan lukien Jetson Nano, Orin Nano, Orin NX ja AGX Orin, joissa on JetPack 6. Chloros tunnistaa Jetson-mallisi automaattisesti ja optimoi sen käsittelystrategian. Katso [NVIDIA Jetson -opas](linux/nvidia-jetson-guide.md) asennus- ja käyttöönotto-ohjeiden saamiseksi.

</details>

<details>

<summary>Optimoiko Chloros automaattisesti laitteistolleni?</summary>

Kyllä! Chloros 1.1.0 sisältää [dynaamisen laskentasovituksen](processing-architecture/dynamic-compute-adaptation.md), joka tunnistaa automaattisesti CPU:n, GPU:n, RAM-muistin ja (Jetsonissa) lämpötila-anturit. Sen jälkeen se valitsee optimaalisen käsittelystrategian — `GPU_PARALLEL`:stä suurimuistisissa järjestelmissä, `GPU_SINGLE`:stä rajoitetuissa laitteissa ja `CPU_PARALLEL`:stä järjestelmissä, joissa ei ole NVIDIA-GPU:ta. Manuaalista konfigurointia ei tarvita.

</details>

<details>

<summary>Mikä on 4-säikeinen prosessointiputki?</summary>

Chloros 1.1.0 käyttää 4-säikeistä putkitetun arkkitehtuuria Chloros+ -käyttäjille: Säie 1 (tunnistus) lataa kuvat ja tunnistaa kalibrointikohteet, säie 2 (kalibrointi) laskee heijastavuuskalibroinnin, säie 3 (käsittely) suorittaa GPU-kiihdytetyn debayeringin ja indeksin laskennan ja säie 4 (vienti) kirjoittaa tulostustiedostot. Useita kuvia voi olla samanaikaisesti eri säikeissä maksimaalisen läpimenon saavuttamiseksi. Katso lisätietoja kohdasta [Käsittelyputki](processing-architecture/processing-pipeline.md).

</details>

<details>

<summary>Kuinka suoritan diagnostiikan Chloros-asennuksellani?</summary>

Käytä komentoa `selftest` suorittaaksesi 7 järjestelmän diagnostiikkaa, mukaan lukien version tarkistus, porttien saatavuus, taustapalvelimen käynnistys, API-yhteys, järjestelmätiedot, kohinanpoistomallit ja CUDA:n saatavuus:

```bash
chloros-cli selftest
```

Tämä on erityisen hyödyllistä Linux/Jetson-järjestelmissä GPU:n ja CUDA:n asetusten tarkistamiseksi.

</details>
