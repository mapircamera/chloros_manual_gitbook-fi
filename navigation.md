# Käyttöliittymä: Navigointi

Kun käynnistät Chloros- ja Chloros-sovellukset (selain) ensimmäisen kerran, niiden taustapalvelu käynnistyy. Kun se on valmis, vasemmassa yläkulmassa oleva päävalikkokuvake tulee näkyviin <img src=".gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

Yläpalkissa on vasemmalta oikealle seuraavat kohdat:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Päävalikko

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

Päävalikosta voit:

* **Uusi projekti** — luoda uuden projektin
* **Avaa projekti** — avata olemassa olevan projektin
* **Avaa projektikansio** — avata projektikansion tiedostoselaimessa
* **Lisää tiedostoja** — lisätä yksittäisiä kuvatiedostoja nykyiseen projektiin _(näkyvissä, kun projekti on avattu)_
* **Lisää kansio** — lisää kuvakansio nykyiseen projektiin _(näkyvissä, kun projekti on avattu)_
* **Aloita käsittely / Lopeta käsittely** — aloita tai lopeta kuvankäsittelyputki _(käytössä, kun tiedostot on lisätty)_

{% hint style="info" %}
**Vain Windows**: Chloros-työpöydän käyttöliittymä on käytettävissä Windows:ssa. Linux-käyttäjien tulisi katsoa [CLI](CLI.md) ja [Python SDK](api-python-sdk.md) -dokumentaatiosta.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1).png" alt="" data-size="line"> Toisto-/Käynnistyspainike

Kun tämä on käytössä, käsittelyn käynnistyspainike käynnistää kuvankäsittelyputken.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> Edistymispalkki <img src=".gitbook/assets/image (5).png" alt="" data-size="line">Ilmaisessa Chloros-tilassa, joka käsittelee kaikki tiedostot peräkkäin, edistymispalkki näyttää kaksi vaihetta: Kohteen tunnistus ja Käsittely.

Maksullisessa Chloros+-lisenssitilassa, joka käsittelee kaikki tiedostot samanaikaisesti, edistymispalkki näyttää neljä vaihetta: Tunnistus, Analysointi, Kalibrointi, Vienti. Jos viet hiiren osoittimen Chloros+-etenemispalkin päälle, avautuu laajennettu nelivaiheinen etenemispalkkipaneeli, josta voit seurata prosessin etenemistä. Yläosan etenemispalkkia napsauttamalla pysäytät avattavan paneelin, ja napsauttamalla uudelleen vapautat sen.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Sivumenu

Vasemmalla sivupalkissa on erilaisia kuvakkeita, joita voit käyttää:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Projektin asetukset](project-settings/project-settings.md)

Projektin asetukset -välilehdessä voit säätää projektin yleisiä ja käsittelyasetuksia. Säädä nämä ennen tiedostojen käsittelyn aloittamista.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Tiedostoselain

Lisää tiedostoja/kansioita ja poista tiedostoja projektista. Kaksoiskappaleet ohitetaan. Valitse kohdekuvien kohdalla kohde-sarakkeen valintaruutu, jolloin käsittely tarkastelee vain valittuja kuvia kohteina, mikä nopeuttaa käsittelyaikaa huomattavasti. Käytä Kuva/Metatiedot-kytkintä vaihtaaksesi valitun kuvan pikkukuvaruudukon ja yksityiskohtaisen metatietotaulukon välillä.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Kuvien katseluohjelma](image-viewer-gui/opening-an-image-full-screen.md)

Kun kuvaa napsautetaan pääkuvakatselijassa, se avautuu koko ruudun kokoisena Kuvien katseluohjelma-välilehdessä.

#### <img src=".gitbook/assets/image (7).png" alt="" data-size="line"> [Kartta](image-viewer-gui/map-markers.md)

Tarkastele kuvia interaktiivisella 2D-kartalla niiden GPS-koordinaattien perusteella. Tukee Google Mapsia ja ESRI-karttapalveluita, valiten automaattisesti sijaintiisi parhaiten sopivan palvelun. Vie hiiri merkkien päälle nähdäksesi kuvien pikkukuvien esikatselut.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Vianmääritysloki

Tarkista lokista vianmääritystulostukset, kun ongelmia ilmenee. Kopioi/lataa loki ja lähetä se [MAPIR-tukeen](https://www.mapir.camera/community/contact) saadaksesi apua.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Käyttäjän kirjautuminen](chloros+-login.md)

Käyttäjän kirjautumissivupalkin avulla voit kirjautua Chloros+ -tiliisi ja avata lisäominaisuudet. Voit myös tarkastella sovelluksen nykyistä versiota sekä muuttaa Chloros-käyttöliittymän ja CLI-sovelluksen tekstin kieltä.
