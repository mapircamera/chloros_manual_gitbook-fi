# Käyttöliittymä: Navigointi

Kun käynnistät Chloros-ohjelman ensimmäisen kerran, se käynnistää käsittelytaustaprosessinsa. Kun taustaprosessi on valmis, vasemmassa yläkulmassa näkyy päävalikon kuvake <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> ja vasemmassa sivupalkissa avautuvat Kamerat- ja Valosensorit-välilehdet (ne ovat siihen asti harmaana).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Yläpalkissa on vasemmalta oikealle seuraavat kohdat:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">-päävalikko

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Päävalikosta voit:

* **Uusi projekti**— luo uuden projektin. Jos olet tallentanut projektimallipohjia, näkyviin tulee**Valitse malli** -pudotusvalikko, jolloin uusi projekti aloitetaan mallipohjan asetuksista.
* **Avaa projekti**— avaa olemassa oleva projekti. Luettelossa on**Avaa projektikansio** -painike, joka avaa projektikansion tiedostoselaimessasi.
* **Kopioi projekti** — kopioi tällä hetkellä avoinna oleva projekti uudella nimellä (ehdotetaan vapaata nimeä, kuten ”MyProject (2)”) ja avaa kopio. _(näkyy, kun projekti on avattu)_
* **Lisää tiedostoja** — lisää yksittäisiä kuvatiedostoja nykyiseen projektiin _(näkyy, kun projekti on avattu)_
* **Lisää kansio** — lisää yksi tai useampi kuvakansio nykyiseen projektiin _(näkyy, kun projekti on avattu)_
* **Käynnistä käsittely / Lopeta käsittely** — käynnistä tai lopeta kuvankäsittelyputki _(käytettävissä tiedostojen lisäämisen jälkeen)_
* **Yhdistä kameraan** — siirry [Kamerat-välilehdelle](lattice/) yhdistääksesi LATTICE-kameran tai -matriisin. Toimii ilman avointa projektia.
* **Yhdistä valosensoriin** — siirry [Valosensorit-välilehdelle](daq/) yhdistääksesi DAQ-valosensorin. Toimii ilman avointa projektia.

{% hint style="info" %}
**Vain Windows**: Chloros-työpöytäkäyttöliittymä on käytettävissä Windows:ssä. Linux-käyttäjien tulisi tutustua [CLI](CLI.md) ja [Python SDK](api-python-sdk.md) -ohjeet, jotka koskevat päättömää käsittelyä.
{% endhint %}

###<img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">

Toisto-/Käynnistyspainike

Kun tämä toiminto on käytössä, käsittelyn käynnistyspainike käynnistää kuvankäsittelyputken.

###<img src=".gitbook/assets/image (4).png" alt="" data-size="line">

Edistymispalkki<img src=".gitbook/assets/image (5).png" alt="" data-size="line">

Ilmaisessa Chloros-tilassa, jossa kaikki tiedostot käsitellään peräkkäin, edistymispalkki näyttää kaksi vaihetta: Kohteen tunnistus ja Käsittely.

Maksullisessa Chloros+-lisenssimoodissa, joka käsittelee kaikki tiedostot samanaikaisesti, edistymispalkki näyttää neljä vaihetta: tunnistus, analysointi, kalibrointi ja vienti. Jos viet hiiren osoittimen Chloros+-etenemispalkin päälle, avautuu laajennettu nelivaiheinen etenemispalkkipaneeli, jonka avulla voit seurata prosessin etenemistä. Ylimmän etenemispalkin napsauttaminen jäädyttää avattavan paneelin, ja uudelleen napsauttaminen vapauttaa sen.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Sivumenu

Vasemmalla olevassa sivupalkkimenussa on erilaisia toimintokuvakkeita, jotka ovat järjestyksessä ylhäältä alas seuraavat:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Projektin asetukset](project-settings/project-settings.md)

Projektin asetukset -välilehdessä voit säätää projektin yleisiä asetuksia ja käsittelyasetuksia. Määritä nämä asetukset ennen tiedostojen käsittelyn aloittamista.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Tiedostoselain

Lisää tiedostoja tai kansioita projektiin tai poista niitä siitä. Kaksoiskappaleet ohitetaan. Valitse kohdekuvien kohdalla kohdesarakkeen valintaruutu, jolloin käsittely etsii kohteita vain valituista kuvista, mikä nopeuttaa käsittelyä huomattavasti. Käytä Kuva/Metatiedot-kytkintä vaihtaaksesi valitun kuvan pikkukuvaruudukon ja yksityiskohtaisen metatietotaulukon välillä.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Kuvankatseluohjelma](image-viewer-gui/opening-an-image-full-screen.md)

Kun kuvaa napsautetaan pääkuvankatselijassa, se avautuu koko näytön tilassa Kuvan katselu -välilehdessä.

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Karttakatselu](image-viewer-gui/map-markers.md)

Tarkastele kuvia interaktiivisella 2D-kartalla niiden GPS-koordinaattien perusteella. Tukee Google Mapsia ja ESRI-ruutupalveluntarjoajia ja valitsee automaattisesti sijaintiisi parhaiten sopivan palvelun. Vie hiiri merkintöjen päälle nähdäksesi kuvien pikkukuvien esikatselut.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Kamerat](lattice/)

Yhdistä ja hallitse LATTICE-kameroita reaaliaikaisesti — yksi kerrallaan tai synkronoituina monikamerajärjestelminä. Välilehdessä näkyvät reaaliaikaiset esikatseluruudut, joissa on päällekkäisiä kerroksia ja histogrammeja, kamera- ja ryhmäkohtaiset asetukset sekä tallennusasetukset, joilla valitaan, mitkä kamerat ja vientityypit ”Capture All” -toiminto tuottaa. Käytettävissä, kun taustapalvelu on valmis; katso [LATTICE-osio](lattice/) saadaksesi täydellisen ohjeen.

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Valoanturit](daq/)

Liitä DAQ-valoanturit — DAQ-U (USB), DAQ-M (Bluetooth) ja DAQ-E (Ethernet) — ja tarkastele niiden reaaliaikaisia kalibroituja spektrikaavioita yksiköissä W/m²/nm. Täältä voit tallentaa `.daq`-tiedostoja avoimeen projektiin, nimetä antureita uudelleen, valita korkkikorjausprofiileja ja päivittää DAQ-E:n laiteohjelmiston. Käytettävissä, kun taustapalvelu on valmis; katso [DAQ-osio](daq/) saadaksesi täydellisen ohjeen.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line">-virheenkorjausloki

Tarkista lokista virheenkorjausviestit, kun ongelmia ilmenee. Kopioi tai lataa loki ja lähetä se [MAPIR-tukeen](https://www.mapir.camera/community/contact) avun saamiseksi.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Käyttäjän kirjautuminen](chloros+-login.md)

Käyttäjän kirjautumispalkin avulla voit kirjautua Chloros+-tilillesi ja avata lisäominaisuudet. Voit myös tarkastella sovelluksen nykyistä versiota sekä muuttaa Chloros-käyttöliittymän ja CLI:n tekstin kieltä.
