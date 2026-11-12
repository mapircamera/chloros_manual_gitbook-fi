# DAQ-valosensorit

> **Etsitkö laitteistoa?**Itse anturit – mallit, kiinnitys, suojukset, liitännät, virransyöttö ja SCANNER-sovellus – on kuvattu**[DAQ-käyttöoppaassa](https://mapir.gitbook.io/daq)**. Tässä luvussa käsitellään niiden käyttöä Chloros-ohjelmistossa.

MAPIRin **DAQ**-valosensorit mittaavat ympäristön valoa radiometrisesti kalibroituina spektreinä. Chloros-ohjelmistossa niillä on kaksi tehtävää:

* **Itsenäinen spektrimittari** — reaaliaikaiset spektrikaaviot, kolorimetriset tiedot ja `.daq`-tallenteet, kaikki saatavilla [Valosensorit-välilehdestä](gui.md), [CLI:stä](cli-quick-start.md) tai Python-SDK:sta.
* **Alaspäin suuntautuvan säteilyn lähde heijastavuuden laskemiseksi** — käsittelyn aikana Chloros interpoloi `.daq`-lukemasi kunkin kuvan valotusaikaleimaan ja käyttää mitattua alaspäin suuntautuvaa valoa kameran säteilyvoimakkuuden muuntamiseen heijastavuudeksi (`--reflectance-source daq`); kalibroituja kaistoja varten ei tarvita kuvauspaikalla olevaa paneelia.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Kolme mallia, yksi tiedostomuoto

| Malli | Siirto | Tunnistus |
| --- | --- | --- |
| **DAQ-U** | USB (sarjaliitäntä) | sarjaportin skannaus |
| **DAQ-M** | Bluetooth Low Energy | BLE-skannaus nimen perusteella |
| **DAQ-E** | Ethernet (IPv4, PoE-virtalähde) | mDNS `_daq-e._tcp` (isäntänimi `daq-e-<id>.local`) |

**DAQ-E-S (uusi versio 1.3.0).** DAQ-E, jossa on kiinteä hajotin ja kallistusanturi. Chloros-ohjelmisto luettelee sen nimellä `DAQ-E-S`, ja se toimii täsmälleen kuten DAQ-E, paitsi että se ei vaadi korkkikorjausta ja jokainen lukema sisältää laitteen kallistuksen, kiertymän ja nousukulman. Katso [DAQ-E-S: kiinteä hajotin ja kallistus](caps-and-range.md#daq-e-s-fixed-diffuser-and-tilt).

Kaikki kolme käyttävät samaa tiedonsiirtoprotokollaa ja toimittavat identtisiä tietoja:

* **135 pisteen spektri välillä 340–1010 nm 5 nm:n välein** sekä CIE XYZ-kolmiväriarvot jokaisessa kehyksessä.
* **Radiometrisesti kalibroitu spektrinen säteilyvoimakkuus yksikössä W/m²/nm** — kunkin laitteen tehdaskalibrointipaketti (sekä sen aktiivinen korkkikorjausprofiili) otetaan huomioon ennen kuin tiedot saapuvat sinulle.
* Sama **`.daq`-tallennusmuoto** (SQLite-tiedosto). Jatkokäsittely on identtistä riippumatta siitä, mikä siirtoprotokolla tuotti tiedoston.

Siirtoprotokollat (USB-sarjaliitäntä, BLE, mDNS/zeroconf) on integroitu Chloros-taustapalvelimeen — mitään ei tarvitse asentaa, jotta voit kommunikoida minkä tahansa kolmen mallin kanssa graafisen käyttöliittymän tai CLI:n `pool-*`-komentojen kautta.

***

## Kalibroitu alue: raportoitu 340–1010 nm, kalibroitu ~374–974 nm

Anturi raportoi koko 340–1010 nm:n ruudukon, mutta NIST-jäljitettävä radiometrinen vahvistus ulottuu noin **374–974 nm:n** alueelle. Chloros-ohjelmisto hylkää absoluuttisen heijastuskyvyn jakamisen kaikille kameran kaistoille, joiden spektrinen paino on alle puolet kyseisen kalibroidun alueen sisällä; ohitettu kaista ilmoitetaan ohitussyynä `dls-uncalibrated-band-<nm>`.

Toimitettavista LATTICE-suodattimista vain **F988** on tämän vaikutuksen alainen:

F988:n heijastavuus on kalibroitu käyttämällä kuvauspaikalla olevaa heijastavuuspaneelia: kaista sijaitsee DAQ-valosensorin kalibroidun alueen ulkopuolella, joten Chloros käyttää viimeisintä paneelista tallennettua arvoa ja säilyttää sen paneelin mittausten välillä.

Jos F988-mittausta käsitellään siten, että käytettävissä on vain DAQ-tietoja, Chloros hylkää kyseisen kaistan DAQ-pohjaisen heijastavuuden ohitussyynä `dls-uncalibrated-band-988` — [heijastavuuspaneelin työnkulku](../calibration-targets.md) on F988:n tuettu menetelmä.

***

## Anturitunnukset

Jokainen DAQ ilmoittaa vakaan anturitunnuksen. Sen muoto vaihtelee mallin mukaan:

| Malli | Tunnuksen muoto | Esimerkki |
| --- | --- | --- |
| DAQ-U | 5-oktettinen, väliviivalla erotettu | `CB-7C-A8-2E-5F` |
| DAQ-M | 5-oktettinen, väliviivalla erotettu | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

Anturin tunniste on:

* merkitty jokaiseen sen tallentamaan `.daq`-tiedostoon,
* avain, jota Chloros käyttää kyseisen laitteen tehdaskalibrointipaketin hakemiseen,
* arvo, jonka välität `--sensor-id`:lle CLI-komentoissa `pool-*`, ja
* DAQ-E:n osalta myös sen mDNS-isäntänimi (`daq-e-def330.local`) — arvo, jonka `--eth-host` hyväksyy.

***

## Tehdaskalibrointi ja pilvipalvelu

Jokainen DAQ-yksikkö on kalibroitu tehtaalla erikseen NIST-jäljitettävällä radiometrisellä ketjulla, ja Chloros lataa jokaisen yksikön kalibrointipaketin, joka on sidottu sen anturitunnukseen. Yksittäisen laitteen kalibrointiraportti (PDF) on ladattavissa anturin asetuksista [Valosensorit-välilehdeltä](gui.md).

{% hint style="warning" %}
**DAQ-U ja DAQ-M vaativat pilvipalvelun käyttöoikeuden kalibrointia varten.**Kumpikaan malli ei tallenna mitään laitteeseen: niiden tehdaskalibrointipaketit sijaitsevat MAPIRin pilvipalvelussa ja haetaan anturin tunnisteen perusteella (ja tallennetaan sitten paikalliseen välimuistiin). Chloros tarvitsee internetyhteyden toimittaakseen kalibroituja W/m²/nm-tietoja DAQ-U- tai DAQ-M-laitteesta.**DAQ-E on poikkeus** — se tallentaa kalibrointitietonsa laitteeseen.
{% endhint %}
<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

***

## Tallenteiden tallennuspaikat

| Pinta | Oletuskohde `.daq` |
| --- | --- |
| Käyttöliittymä — Valosensorit-välilehti | `<project folder>/light_sensor/` (valmiit tallenteet lisätään projektiin automaattisesti) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` koneella, jolla taustapalvelu on käynnissä (suhteellinen `-o` viittaa siihen kansioon, jossa komento suoritettiin, kun taustapalvelin on samalla tietokoneella) |

Jokainen `.daq`-tiedostonimi sisältää anturin tunnuksen ja aikaleiman.

***

## Tässä luvussa

* [**Chloros-ohjelman DAQ-välilehti**](gui.md) — kattava GUI-opas: kunkin mallin liittäminen, antikohtaiset asetukset, spektrikaaviot, reaaliaikaiset kolorimetriset tiedot, kahden anturin heijastavuus ja tallennus.
* [**CLI-pikaopas (pool-\*)**](cli-quick-start.md) — DAQ-anturien ohjaaminen `chloros-cli daq pool-*`:n kautta, tuettu komentorivipolku.
* [**Kapasiteettiprofiilit ja kalibroitu alue**](caps-and-range.md) — mitkä kapasiteettiprofiilit ovat käytettävissä kussakin mallissa, miten ne määritetään sekä kalibroitu spektrialue yksityiskohtaisesti.
* [**Tallennus ja .daq-muoto**](recording.md) — `.daq` SQLite-muoto ja tallennusprosessit.
* [**DAQ-E-verkko ja aikasynkronointi**](ethernet-ptp.md) — DAQ-E-siirtotilat ja PTP-aikasynkronointi.
* [**Heijastavuustyönkulut**](reflectance.md) — DAQ-alaspäin suuntautuvien tietojen käyttö heijastavuuden laskemiseen.
* Kattava lipputason dokumentaatio löytyy [CLI-viitteestä](../reference/cli-reference.md) (kohta `chloros-cli daq`) ja [SDK-viitteestä](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), jotka on molemmat kirjoitettu siten, että tekoälyavustajat voivat käyttää niitä suoraan.
