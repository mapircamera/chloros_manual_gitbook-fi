# DAQ-valosensorit

> **Etsitkö laitteistoa koskevia tietoja?**Itse anturit – mallit, kiinnitys, suojukset, liitännät, virransyöttö ja SCANNER-sovellus – on kuvattu**[DAQ-käyttöoppaassa](https://mapir.gitbook.io/daq)**. Tässä luvussa käsitellään niiden käyttöä alkaen mallista Chloros.

MAPIR:n **DAQ**-valosensorit mittaavat ympäristön valoa radiometrisesti kalibroituina spektreinä. Chloros:ssa niillä on kaksi tehtävää:

* **Itsenäinen spektrilaite** — reaaliaikaiset spektrikaaviot, kolorimetriset tiedot ja `.daq`-tallenteet, jotka kaikki löytyvät [Valosensorit-välilehdeltä](gui.md), [CLI](cli-quick-start.md) tai Python SDK.
* **Alaspäin suuntautuvan säteilyn lähde heijastavuuden laskemiseksi** — käsittelyn aikana Chloros interpoloi `.daq`-lukemasi kunkin kuvanvalotusaikaleimaan ja käyttää mitattua alaspäin suuntautuvaa valoa kameran säteilyvoimakkuuden muuntamiseen heijastavuudeksi (`--reflectance-source daq`); kalibroituja kaistoja varten ei tarvita kuvauspaikalla olevaa paneelia.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Kolme mallia, yksi tiedostomuoto

| Malli | Siirto | Tunnistus |
| --- | --- | --- |
| **DAQ-U** | USB (sarjaliitäntä) | sarjaportin skannaus |
| **DAQ-M** | Bluetooth Low Energy | BLE-skannaus nimen perusteella |
| **DAQ-E** | Ethernet (IPv4, PoE-virtalähde) | mDNS `_daq-e._tcp` (isäntänimi `daq-e-<id>.local`) |

Kaikki kolme käyttävät samaa tiedonsiirtoprotokollaa ja toimittavat identtistä dataa:

* **135 pisteen spektri välillä 340–1010 nm 5 nm:n välein** sekä CIE XYZ-kolmiväriarvot jokaisessa kehyksessä.
* **Radiometrisesti kalibroitu spektrinen säteilyvoimakkuus yksikössä W/m²/nm** — kunkin laitteen tehdaskalibrointipaketti (sekä sen aktiivinen korkkikorjausprofiili) otetaan käyttöön ennen kuin tiedot saapuvat sinulle.
* Sama **`.daq`-tallennusmuoto** (SQLite-tiedosto). Jatkokäsittely on identtinen riippumatta siitä, mikä siirtoprotokolla tiedoston tuotti.

Siirtoprotokollapinot (USB-sarjaliitäntä, BLE, mDNS/zeroconf) on koottu Chloros-taustapalveluun — mitään ei tarvitse asentaa, jotta voit kommunikoida minkä tahansa näistä kolmesta mallista graafisen käyttöliittymän tai CLI:n `pool-*`-komentojen kautta.

***

## Kalibroitu alue: raportoitu 340–1010 nm, kalibroitu ~374–974 nm

Anturi raportoi koko 340–1010 nm:n alueen, mutta NIST-jäljitettävä radiometrinen vahvistus ulottuu noin **374–974 nm**:n alueelle. Chloros hylkää absoluuttisen heijastuskyvyn jakamisen kaikille kameran kaistoille, joiden spektrinen paino on alle puolet kyseisen kalibroidun alueen sisällä; ohitettu kaista ilmoitetaan ohitusperusteella `dls-uncalibrated-band-<nm>`.

Myynnissä olevista LATTICE-suodattimista tämä koskee vain **F988**-mallia:

F988:n heijastavuus on kalibroitu käyttämällä kuvauspaikalla olevaa heijastavuuspaneelia: kaista sijaitsee DAQ-valosensorin kalibroidun alueen ulkopuolella, joten Chloros käyttää viimeisintä paneelin tallennustasi ja säilyttää sen paneelin mittausten välillä.

Jos F988-lukemaa käsitellään tilanteessa, jossa käytettävissä on vain DAQ-tietoja, Chloros hylkää kyseisen kaistan DAQ-pohjaisen heijastavuuden ohitussyynä `dls-uncalibrated-band-988` — [heijastavuuspaneelin työnkulku](../calibration-targets.md) on F988:n tuettu reitti.

***

## Anturitunnukset

Jokainen DAQ ilmoittaa vakaan anturitunnuksen. Sen muoto vaihtelee mallin mukaan:

| Malli | Tunnusmuoto | Esimerkki |
| --- | --- | --- |
| DAQ-U | 5-oktettinen, väliviivoilla erotettu | `CB-7C-A8-2E-5F` |
| DAQ-M | 5-oktettinen, väliviivoilla erotettu | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

Anturin tunniste on:

* merkitty jokaiseen sen tallentamaan `.daq`-tiedostoon,
* avain, jota Chloros käyttää kyseisen laitteen tehdaskalibrointipaketin hakemiseen,
* arvo, jonka välität `--sensor-id`:lle komentoissa CLI ja `pool-*`, sekä
* DAQ-E:n osalta myös sen mDNS-isäntänimen (`daq-e-def330.local`) — arvon, jonka `--eth-host` hyväksyy.

***

## Tehdaskalibrointi ja pilvipalvelu

Jokainen DAQ-yksikkö on kalibroitu tehtaalla erikseen NIST-jäljitettävällä radiometrisellä ketjulla, ja Chloros lataa kunkin yksikön kalibrointipaketin, joka on avainnut sen anturitunnuksen mukaan. Yksikkökohtainen kalibrointiraportti (PDF) on ladattavissa anturin asetuksista [Valoanturit-välilehdeltä](gui.md).

{% hint style="warning" %}
**DAQ-U ja DAQ-M vaativat pilvipalvelun käyttöoikeuden kalibrointia varten.**Kumpikaan malli ei tallenna mitään laitteeseen: niiden tehdaskalibrointipaketit sijaitsevat MAPIR:n pilvipalvelussa ja ne haetaan anturin tunnisteen perusteella (ja tallennetaan sitten paikalliseen välimuistiin). Chloros tarvitsee internetyhteyden toimittaakseen kalibroituja W/m²/nm-tietoja DAQ-U:sta tai DAQ-M:stä.**DAQ-E on poikkeus** — se tallentaa kalibrointitietonsa laitteeseen.

<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

{% endhint %}***

## Tallenteiden tallennuspaikat

| Pinta | `.daq`:n oletuskohde |
| --- | --- |
| Käyttöliittymä — Valosensorit-välilehti | `<project folder>/light_sensor/` (valmiit tallenteet lisätään projektiin automaattisesti) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` taustapalvelinta ajavalla koneella |

Jokaisen `.daq`-tiedostonimessä on anturin tunniste ja aikaleima.

***

## Tässä luvussa

* [**DAQ-välilehti Chloros**](gui.md) — kattava käyttöliittymän esittely: kunkin mallin liittäminen, antinkohtaiset asetukset, spektrikaaviot, reaaliaikaiset kolorimetriset tiedot, kahden anturin heijastavuus ja tallennus.
* [**CLI Pikaopas (pool-\*)**](cli-quick-start.md) — DAQ-anturien ohjaaminen `chloros-cli daq pool-*`-ohjelmasta, tuettu komentorivipolku.
* [**Kattoarvoprofiilit ja kalibroitu alue**](caps-and-range.md) — mitkä kattoarvot ovat käytettävissä kussakin mallissa, miten ne määritellään sekä kalibroitu spektrialue yksityiskohtaisesti.
* [**Tallennus ja .daq-muoto**](recording.md) — `.daq` SQLite-muoto ja tallennusprosessit.
* [**DAQ-E-verkko ja aikasynkronointi**](ethernet-ptp.md) — DAQ-E-siirtotilat ja PTP-aikasynkronointi.
* [**Heijastavuusprosessit**](reflectance.md) — DAQ-alasvalodatan käyttö heijastavuuden laskemisessa.
* Kattava lipputason dokumentaatio löytyy [CLI-viitteestä](../reference/cli-reference.md) (kohta `chloros-cli daq`) ja [SDK-viiteoppaasta](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), jotka molemmat on kirjoitettu siten, että tekoälyavustajat voivat käyttää niitä suoraan.
