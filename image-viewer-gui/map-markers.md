# Karttamerkit

Kartta-välilehti sijoittaa kuvasi interaktiiviseen 2D-karttaan niiden GPS-koordinaattien perusteella. Se tarjoaa maantieteellisen yleiskuvan kuvausistunnosta ja on tuonnin jälkeen nopein tapa poistaa kuvat, joita et halua käsitellä.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Kartta-välilehden avaaminen

1. Avaa tai luo projekti Chloros-ohjelmassa
2. Tuo kuvat, jotka sisältävät GPS-metatietoja
3. Napsauta vasemmassa sivupalkissa **Kartta**-<img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line">-välilehteä
4. Kartalla näkyy merkki kunkin kuvan GPS-sijainnissa

{% hint style="info" %}
**GPS vaaditaan**: kartalle näkyvät vain kuvat, joiden EXIF-metatiedoissa on GPS-koordinaatit. Kuva, jolla ei ole koordinaatteja, on edelleen projektissa ja käsitellään normaalisti – sillä ei vain ole merkkiä.
{% endhint %}

***

## Kuvien muokkaaminen Kartta-välilehdellä**Kartta**-<img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line">-välilehdessä on samat lisää-<img src="../.gitbook/assets/image (3).png" alt="" data-size="line">-<img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line">- ja poista-<img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">-painikkeet kuin [**Tiedostoselaimessa**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">-välilehdessä. Se näyttää saman projektitiedostoluettelon, jossa on maantieteelliset sarakkeet:

| Sarake        | Sisältö                                                           |
| ------------- | ------------------------------------------------------------------ |
| **Nimi**      | Tiedoston nimi sellaisena kuin se tallennettiin kamerasta                             |
| **Leveysaste**  | Desimaaliasteina, kuusi desimaalia                                |
| **Pituusaste** | Desimaaliasteet, kuusi desimaalia                                |
| **Korkeus**  | Metrejä, yksi desimaali — `-`, jos kuvassa ei ole korkeustietoja |

{% hint style="info" %}
Napsauta mitä tahansa sarakkeen otsikkoa lajitellaksesi sen mukaan; napsauta uudelleen kääntääksesi järjestyksen.
{% endhint %}

{% hint style="warning" %}
**Korkeus on korkeus merenpinnan yläpuolella, ei korkeus maanpinnan yläpuolella.** Arvo on peräisin kuvan EXIF-tunnisteesta `GPSAltitude`, joka viittaa merenpinnan tasoon. Se ei ole lentokorkeus maaston yläpuolella, eikä Chloros laske siitä maanpinnan näytteenottoväliä — 300 m merenpinnan yläpuolella sijaitsevan pellon yläpuolella 100 m AGL:n korkeudella lentävä drone tallentaa tähän arvoon noin 400 m. Käytä saraketta poikkeavien arvojen havaitsemiseen ja johdonmukaisen lentokorkeuden varmistamiseen, älä AGL-mittauksena.
{% endhint %}

***

## Kuvan merkit

Jokaiseen GPS-tietoja sisältävään kuvaan lisätään merkki sen koordinaatteihin.

### Merkintöjen näyttö

* Merkinnät sijaitsevat kunkin kuvan tallennetuissa tarkkoissa koordinaateissa
* Lähekkäin olevat merkinnät voivat näyttää päällekkäisiltä, kun kuvaa pienennetään – zoomaa kuvaa, jotta ne erottuvat toisistaan
* Valitut ja korostetut merkinnät näkyvät muiden yläpuolella

### Esikatselu hiiren osoittimella

* **Vie hiiren osoitin** minkä tahansa merkin päälle, niin kyseisen kuvan pikkukuva ja tiedostonimi avautuvat ponnahdusikkunaan
* **Napsauta**merkkiä valitaksesi kuvan ja**kiinnitä** ponnahdusikkuna auki — se pysyy näkyvissä, kunnes napsautat muualle. Kun ponnahdusikkuna on kiinnitetty, hiiren vieminen muiden merkkien päälle ei vie sitä pois
* Tämä on nopea tapa löytää tietty kuva suuresta kuvasarjasta poistumatta kartalta

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption><p>Kartta-välilehti piirtää kaikki projektin geotunnisteiset kuvat</p></figcaption></figure>### Superzoom

{% hint style="success" %}
**SUPERZOOM**: kun saavutat suurimman zoomausasteen, jolle ruutujen tarjoajalla on kuvamateriaalia, lisäzoomaus suurentaa ruutuja sen sijaan, että pysähtyisi, joten voit erottaa toisistaan merkkejä, jotka sijaitsevat lähes päällekkäin.
{% endhint %}

* Superzoom aktivoituu vain, kun olet **juuri** palveluntarjoajan kyseiselle sijainnille määrittämässä suurimmassa zoomausasteessa ja ruudut ovat latautuneet kokonaan. Alle tämän tason zoomaus toimii normaalisti
* Alue on **1×–32×** palveluntarjoajan oman suurimman zoomausasteen lisäksi
* Kulmassa oleva ilmaisin näyttää nykyisen superzoomin prosentteina, ja sen vieressä oleva **×**-painike palauttaa sinut normaalizomaukseen yhdellä napsautuksella
* Loitontaminen välittyy aina itse karttaan, joten et voi koskaan jäädä jumiin superzoomiin
* Zoomaaminen ja panoroiminen superzoom-tilassa siirtää tuloksena olevan siirtymän takaisin karttaan, joten keskipisteestä poispäin siirtynyt alue jatkaa ruutujen lataamista sen sijaan, että se muuttuisi tyhjäksi
* Merkit piirretään vektorielementeinä eikä rasteroituina, joten ne pysyvät terävinä kaikilla superzoom-tasoilla

***

## Karttalaattojen tarjoajat

{% hint style="success" %}
**Automaattinen valinta**: Chloros valitsee laattapalvelun, joka tarjoaa parhaan zoomausasteen kuviesi sijainnin mukaan. Voit vaihtaa palvelua manuaalisesti milloin tahansa.
{% endhint %}

| Palveluntarjoaja        | Huomautukset                                                                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Google Maps** | Laaja maailmanlaajuinen kattavuus; tukee kaikkia neljää karttalaattatyyppiä                                                                                                            |
| **Esri ArcGIS**| Usein korkeamman resoluution ilmakuvia tietyillä alueilla.**Terrain**-karttalaattatyyppiä ei tarjota Esrille, ja sen painike on poissa käytöstä, kun Esri on valittuna |***

## Karttalaattatyypit

Valitse karttakerroksen tyyppi painikkeilla (vasemmalta oikealle):

![](&lt;../.gitbook/assets/image (14).png&gt;)

| Tyyppi                 | Näyttää                                                                |
| -------------------- | -------------------------------------------------------------------- |
| **Maasto**          | Korkeusvarjostus ja karttayksityiskohdat (tiet, merkinnät). Vain Google       |
| **Kartta**              | Tavalliset katukarttalaatat — vaihtoehto, joka vaatii vähiten kaistanleveyttä              |
| **Satelliitti**        | Yksityiskohtaiset satelliittikuvat, ilman merkintöjä — vaihtoehto, joka vaatii eniten kaistanleveyttä |
| **Hybridi** (oletus) | Satelliittikuvat, joiden päälle on piirretty tiet ja merkinnät                |

Kartta-välilehti avautuu **Hybridi**-asetuksella. Valintasi vaikuttaa palveluntarjoajan vaihtoon, jos palveluntarjoaja tukee sitä.***

## Kartan navigointi

* **Lähentäminen**: hiiren vierityspyörä tai kartan zoomauspainikkeet
* **Panorointi**: napsauta ja vedä
* **Koko näyttö**: koko näytön painike laajentaa kartan koko ikkunaan***

## Käyttötapaukset

### Lentoreitin tarkastelu

* Näe yhdellä silmäyksellä droonilennon kattama alue
* Havaitset aukot, joissa yksi kierros on jäänyt väliin
* Varmistat, että lento seurasi suunniteltua reittiä

### Maastotutkimuksen tarkastelu

* Näet, miten maasta otetut kuvat ovat jakautuneet
* Paikannat kalibrointikohteiden kehykset suhteessa tutkimusalueeseen
* Päätät, missä tarvitaan lisäkuvia

### Laadunvalvonta

* Etsi odottamattomista paikoista otetut kuvat ja poista ne ennen käsittelyä
* Lajittele korkeuden mukaan, jotta voit havaita väärällä korkeudella otetut kuvat tai kuvat, joissa GPS-paikannus oli heikko
* Vertaa kuvien sijainteja kenttämuistiinpanoihin

***

## Vianmääritys

### Merkkejä ei näy

**Mahdolliset syyt**

* Kuvissa ei ole GPS-metatietoja
* GPS oli pois päältä kamerassa kuvauksen aikana
* EXIF-tiedot on poistettu toisella ohjelmistolla ennen tuontia

**Toimenpiteet**: varmista, että GPS on käytössä kamerassa, ja tuo alkuperäiset tiedostot uudelleen. Voit tarkistaa, onko tietyllä tiedostolla koordinaatteja, etsimällä sitä Kartta-välilehden tiedostotaulukosta — kuvalla, jolla ei ole koordinaatteja, ei ole siellä riviä.

### Merkit ovat väärässä paikassa

**Mahdolliset syyt**: heikko satelliittisignaali kuvaushetkellä tai GPS-poikkeama istunnon aikana.**Mitä tehdä**: tämä on kuvaushetkeen liittyvä ongelma, jota Chloros ei voi korjata jälkikäteen. Tarkkuutta vaativaan työhön kannattaa käyttää PPK/RTK-GPS-työnkulkua – katso asetus**Käytä PPK-korjauksia** kohdassa [Projektin asetukset](../project-settings/project-settings.md).

### Kartta on tyhjä tai ruutujen latautuminen keskeytyy

Ruutujen tarjoajat ovat verkkopalveluita. Jos ruutujen latautuminen lakkaa, tarkista laitteen verkkoyhteys ja yritä sitten vaihtaa palveluntarjoajaa. Jos olet zoomannut karttaa erittäin paljon, paina **×**-nollauspainiketta palataksesi normaaliin zoomausasteeseen ja anna kartan pyytää ruudut uudelleen.***

## Aiheeseen liittyvät sivut

* [**Kuvaruudukko**](image-grid.md) — sama kuvasarja kuin pikkukuvissa
* [**Kuvan avaaminen koko näytön tilassa**](opening-an-image-full-screen.md) — yhden kuvan tarkastelu yksityiskohtaisesti
* [**Tiedostojen lisääminen projektiin**](../processing-images-gui/adding-files-to-a-project.md) — tämän välilehden tiedostojen lisäys- ja poistopainikkeet
