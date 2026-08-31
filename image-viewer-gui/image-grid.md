# Kuvaruudukko

Kun olet tuonut kuvat projektiin, ne näkyvät pääalueella ruudukossa. Ruudukosta voit valita, **mitä versiota kustakin kuvasta katsot** — sen yläpuolella olevilla painikkeilla voit vaihtaa kaikkien pikkukuvien välillä kerralla lähdetiedostojen ja kunkin käsitellyn tuotteen välillä.

## Pikkukuvien koko

Säädä pikkukuvien kokoa oikeassa yläkulmassa olevalla zoomausliukusäätimellä. Liukusäätimen alue on **64 px – 1200 px**.

* **Ctrl + hiiren rulla** skaalaa myös pikkukuvia.
* **Ctrl + `+`**/**Ctrl + `=`**ja**Ctrl + `−`** muuttavat kokoa 4 px kerrallaan painallusta kohti. Näppäinyhdistelmän avulla kokoa voi pienentää 64 px:iin ja suurentaa niin suureksi, että nykyiseen ikkunaan mahtuu tarkalleen kaksi pikkukuvaa riviä kohden.
* Valitsemasi koko tallennetaan projektin mukana (`UI → Grid thumbnail size` kohdassa `project.json`, oletusarvo `160`), joten projektin avaaminen palauttaa sen.

<figure><img src="../.gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>Pikkukuvan *resoluutio* on erillinen asetus pikkukuvan *koosta*: katso **Näyttö → Kuvan pikkukuvan resoluutio** kohdassa [Projektin asetukset](../project-settings/project-settings.md) (oletusarvo 512 pikseliä pitkältä sivulta). Koko määrittää, kuinka suureksi ruutu piirretään; resoluutio määrittää, kuinka paljon yksityiskohtia haetaan sen täyttämiseksi.***

## Ruudukon työkalurivi

Ruudukon yläpuolella olevassa painikerivissä on enintään kolme ryhmää, vasemmalta oikealle:

1. **Per Trigger / Per Camera** — ryhmittelytila. Näkyy vain projekteissa, jotka sisältävät LATTICE-tallenteita.
2. **Kamerasuodatinpainikkeet** — yksi kutakin LATTICE-kameraa kohti. Näkyy vain Per Camera -tilassa.
3. **Vienti-/katselutilapainikkeet** — mikä tuote kussakin pikkukuvassa näkyy.

Kun ikkuna on liian kapea kaikkien painikkeiden näyttämiseen, ryhmät piilotetaan oikealta vasemmalle hiiren osoittimen päälle tulemalla avautuviksi pudotusvalikoiksi: ensin piilotetaan vienti-/katselupainikkeet, sitten kamerapainikkeet. Piilotetusta ryhmästä jää näkyviin yksi painike, jossa on merkitty kulloinkin aktiivinen valinta, ja kun osoitat sen hiiren osoittimella, koko sarja avautuu näkyviin. **”Per Trigger” / ”Per Camera” -painikkeet eivät koskaan taitu.

<!-- SCREENSHOT-NEEDED: Image grid toolbar of a LATTICE array project at full width, showing all three button groups inline: Per Trigger / Per Camera, three camera filter buttons labelled "LATT-M3M (serial)", and the export/view buttons including TIFF, RAW (Original), RAW (Radiance), RAW (Reflectance). -->

*****

## Vienti- ja katselupainikkeet

Nämä painikkeet vaihtavat ruudukon pikkukuvien kuvatyyppiä. **Painike ilmestyy heti, kun sen nimessä mainittu tuote on olemassa** — mikä lähdetiedostojen osalta tarkoittaa välittömästi tuonnin yhteydessä, ei vasta käsittelyn jälkeen. Chloros skannaa projektin tuotteet uudelleen käsittelyn aikana, joten painikkeet ilmestyvät käsittelyn aikana, kun kukin tuote alkaa tallentua levylle.

### Peruspainike

Vasemmassa reunassa oleva vientipainike on nimetty **sen mukaan, mitä olet tosiasiallisesti tuonut**:

| Mitä olet tuonut | Painikkeen nimi |
| --- | --- |
| Survey3 RAW+JPG | `JPG` |
| LATTICE-kuvat, joiden vieressä on esikatselukuva RAW-kuvan rinnalla | `PNG` tai `TIFF`, riippuen siitä, mitkä esikatselukuvat ovat |
| LATTICE-kaappaukset, joissa perustiedosto **on** raakakuva | *ei painiketta* — `RAW (Original)` näyttää jo kyseisen tiedoston |

Sekaprojektissa teksti noudattaa sitä tiedostotunnistetta, jota useimmissa kuvissa käytetään.

### Tuotepainikkeet

| Painike | Näyttää | Milloin se näkyy |
| --- | --- | --- |
| **Kohteet** | Kuvat, joissa on havaittu kalibrointikohde | Suorituksen jälkeen, jossa kohteet on havaittu |
| **Heijastavuus** | Kalibroidut heijastavuuskuvat | Vain Survey3-projekteissa — LATTICE-projekteissa käytetään sen sijaan `RAW (Reflectance)`:ää, joten ruudukossa ei koskaan näy kahta heijastavuuspainiketta |
| **Valkotasapainotettu** | Valkotasapainotettu tulos (RGB-kamerat) | Käsittelyn jälkeen |
| **Vignettikorjattu** | Kalibroimaton vignettikorjattu varavaihtoehto | Suorituksen jälkeen, jossa heijastavuuskalibrointia ei voitu soveltaa ja *Vignettikorjaus* oli päällä |
| **Anturin vaste** | Kalibroimaton anturin vasteen varavaihtoehto | Sama, mutta *Vignettikorjaus* pois päältä |
| **`RAW (<INDEX> Index)`** | Yksi painike kutakin laskettua indeksiä kohti | Indeksien määrittelyn jälkeen suoritetun ajon jälkeen |
| **`<INDEX> LUT`** | Yksi painike kutakin värikartoitettua indeksiä kohti | LUT-taulukon määrittämisen jälkeen suoritetun ajon jälkeen |
| **`<Index> <Index\|LUT> <NNN>`** | Yksi painike kutakin [Indeksi/LUT-sandbox](index-lut-sandbox.md)-vientia kohti | Heti kun sandbox-vienti on valmis |

### LATTICE-tason painikkeet

LATTICE-kaappauksia sisältäviin projekteihin lisätään nämä painikkeet, jotka on nimetty tason nimen mukaan tuotteen nimen sijaan:

| Painike | Taso |
| --- | --- |
| **RAW (Alkuperäinen)** | Tuotu raakakuva |
| **RAW (Radiance)** | Float32-muotoinen spektrinen säteilyvoimakkuus, W/m²/sr/nm |
| **RAW (Reflectance)** | uint16-muotoinen heijastavuus, 32768 = ρ 1,0 |

`RAW (Original)` on käytettävissä heti tuonnin jälkeen — se ei vaadi käsittelyä. Kun LATTICE-tuonnissa ei ole lainkaan perustason painiketta (jokaisen kaappauksen perustiedosto on sen raakakuva), ruudukko siirtyy itsestään ensimmäiseen käytettävissä olevaan tasopainikkeeseen, jotta työkalurivin korostus vastaa näkymääsi.

Kaksitasoiset Chloros-vientiä eivät saa **omaa ruudukkopainiketta**:

* **Debayered** — `RAW (Original)`-näkymä renderöi jo debayered-muodossa, joten toinen painike visuaalisesti identtiselle kuvalle olisi turhaa. `RAW (Debayered)`-tuote tallennetaan edelleen levylle, ja se on edelleen valittavissa koko näytön kerroksen pudotusvalikosta.
* **Esikatselu** — RGB-kameroissa esikatselu rekisteröidään `White Balanced`-kerrokseksi, jolla on painike. Monispektrikameroissa se on rekisteröity nimellä `RAW (Preview)`, ja se on valittavissa koko näytön kerrosten pudotusvalikosta.

{% hint style="info" %}
Nämä tasopainikkeet näkyvät vain projekteissa, jotka todella sisältävät LATTICE-kehyksiä. Survey3-projekteissa rekisteröidään joitakin samoja sisäisiä tasonimiä, ja painikkeet suodatetaan pois niistä, joten Survey3-ruudukko säilyttää tutun `JPG / Targets / Reflectance`-joukkonsa.
{% endhint %}

Ruudukon pikkukuvan napsauttaminen avaa koko näytön [Kuvankatseluohjelman](opening-an-image-full-screen.md) **samassa tuotteessa, jota ruudukko esittää** — jos ruudukko on asetettu tilaan `Targets`, pikkukuva avaa viedyn kohdekuvan.

<figure><img src="../.gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: This GIF predates the LATTICE level buttons and the toolbar group separators. Reshoot on a LATTICE project cycling base -> RAW (Original) -> RAW (Radiance) -> RAW (Reflectance) -> an index button, so the new button set and the level names are visible. -->

***

## LATTICE-projektin ryhmittely: Trigger-kohtaisesti vs. kamera-kohtaisesti

Array-kaappaukset tuottavat useita kuvia samasta hetkestä eri kameramoduuleista. Ryhmittely määrittää, miten ruudukko järjestää ne päällekkäin. Molemmissa tiloissa näkyy koko leveydeltä ulottuvat, kokoontaitettavat otsikkopalkit; **jokainen ryhmä avautuu oletuksena**, ja Chloros muistaa sulkemasi ryhmät. Kokoontaitetun tilan seuranta tapahtuu erikseen kummassakin tilassa, joten ryhmän sulkeminen Per Camera -tilassa ei sulje mitään Per Trigger -tilassa.

### Per Camera (oletus)

Yksi ryhmä kutakin kameramoduulia kohti. Otsikossa näkyy kameramalli ja sarjanumero (`LATT-M3M — <serial>`) sekä valokuvien lukumäärä. Ryhmän sisällä olevat ruudut on järjestetty kronologisesti kuvaushetken mukaan.

Tässä tilassa työkaluriville ilmestyy myös yksi **kamerasuodatinpainike kutakin kameraa kohti**, jonka nimeksi on merkitty `MODEL (SERIAL)`. Kaikki kamerat ovat aluksi valittuna; painiketta napsauttamalla kyseisen kameran valinta poistetaan ja sen ryhmä poistetaan ruudukosta. Tämä on nopea tapa tarkastella yhtä kaistaa koko lennon ajalta.

### Laukaisukohtaisesti

Yksi ryhmä kutakin kuvauskertaa kohti — joukko kehyksiä, jotka kaikki moduulit ovat ottaneet samalla laukaisulla. Otsikossa näkyy kuvausajankohta, kuvauksiin osallistuneiden kameroiden lukumäärä sekä merkki jokaisesta ryhmän kameramallista. Ryhmän sisällä ruudut on järjestetty kameran sarjanumeron mukaan, joten sama kaista sijaitsee samassa sarakkeessa jokaisella laukaisukerralla.

<!-- SCREENSHOT-NEEDED: Image grid in Per Trigger mode for a 3-camera LATTICE array, showing two consecutive trigger groups with their header bars (chevron, capture timestamp, "3 cameras", and the three model badges) and one group collapsed to show the closed state. -->
Sekaprojektissa olevia muita kuin LATTICE-kuvia ei ryhmitellä – ne näkyvät tavallisina ruuduina ryhmien jälkeen.

***

## Ruudukon pikkukuvat noudattavat GSD-lohkon kokoa

Jos olet asettanut **GSD (px)** -lohkon koon kuvavälilehden sivupalkissa, ruudukon pikkukuvat näytetään samalla maanpinnan resoluutiolla — ei vain koko näytön näkymässä. Lohkokoko 8 tarkoittaa, että jokainen näytetty pikseli on 8 × 8-lohkon lähdepikselien keskiarvo kaikkialla sovelluksessa, jossa kuvaa näytetään.

Koska ruutu on alun perin vain muutaman sadan pikselin levyinen, karkeat lohkokoot lakkaavat vaikuttamasta näkyvästi ruudukkoon jo hyvissä ajoin ennen kuin ne vaikuttavat koko näytön näkymässä: 160 pikselin ruutuun piirretty 4000 pikselin kehys on jo noin 25 lähdepikseliä näytettyä pikseliä kohti. Katso [Kuvan avaaminen koko näytön tilassa](opening-an-image-full-screen.md#gsd-block-size) itse säätimestä.

***

## Aiheeseen liittyvät sivut

* [**Kuvan avaaminen koko näytön tilassa**](opening-an-image-full-screen.md) — koko näytön katseluohjelma, kursoriarvot ja histogrammi
* [**Kuvakerrokset**](image-layers.md) — kerrosten pudotusvalikko täysnäyttöisen katseluohjelman sisällä
* [**Indeksi/LUT-hiekkalaatikko**](index-lut-sandbox.md) — indeksin visualisointien luominen ja vienti
* [**Projektin asetukset**](../project-settings/project-settings.md) — vientivalinnat, jotka määrittävät, mitkä tuotteet ovat ylipäätään käytettävissä
