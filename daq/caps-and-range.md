# Kannen profiilit ja kalibroitu mittausalue

> Itse kannet – eli se, minkä anturin kanssa kukin kansi toimitetaan, miten ne kiinnitetään ja niiden optiset ominaisuudet – on kuvattu **[DAQ-käyttöoppaassa](https://mapir.gitbook.io/daq)**. Tällä sivulla käsitellään asennetun suojuksen *ilmoittamista* Chloros:lle, mikä varmistaa korjauksen oikeellisuuden.

Jokaisen DAQ-valosensorin tehtaalla suoritettu radiometrinen kalibrointi kuvaa *paljaata* anturia. Diffuusorin päälle asennettu fyysinen suojus muuttaa anturin keräämää valoa, joten Chloros soveltaa tehtaalla mitattua **suojuksen korjausprofiilia** kalibrointipaketin päälle. Oikean suojuksen määrittäminen on osa kalibroitujen tietojen saamista — tällä sivulla kerrotaan, mitä suojuksia on saatavilla mallikohtaisesti, miten ne määritetään ja mikä on anturin kalibroitu spektrialue.

## Suojusten saatavuus mallikohtaisesti

| Suojaprofiili (`cap_id`) | Fyysinen suojus | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Sunshine-kosinikorjauskorkki (**oletus kaikissa malleissa**) | Kyllä | Kyllä | Kyllä |
| `fov_15` / `fov_45` / `fov_90` | Näkökenttää rajoittavat kartiot (15° / 45° / 90°) | Kyllä | — | Kyllä |
| `fov_30` / `fov_60` | Näkökenttää rajoittavat kartiot (30° / 60°) | Kyllä | — | — |
| `none` | Ei suojakorkkia | — | — | Kyllä |

Mallikohtaiset huomautukset:

* **DAQ-M:llä on yksi suojakorkkiprofiili: `sunshine_cosine`.** Tuotemääritelmänä on ”Bare-plus-Sunshine-cap”, ja paljaalle DAQ-M:lle ei tarvita geometriprofiilia.
* **Paljas DAQ-U on täysin paljas** — se ei tarvitse lainkaan geometriprofiilia, minkä vuoksi sille ei ole olemassa `none`-profiilia.
* **`none` DAQ-E-mallissa EI OLE toimimaton.** DAQ-E:n upotetulla, lasilla peitetyllä hajottajalla on oma todellinen geometrian korjaus, joten ”ilman suojakantta” on tässä mallissa itsessään mitattu profiili.
* **Paljas DAQ-E ei voi mitata suoraa auringonvaloa missään korkeuskulmassa** — Sunshine-suojus on kenttäasetus. Älä suunnittele ulkotöitä paljaan DAQ-E:n ympärille.

GUI:n antinkohtaisissa asetuksissa (hammasrataskuvake Valosensorit-välilehdessä) **Suojus**-pudotusvalikosta löytyy myös vaihtoehto ”Ei mitään (paljas anturi)” malleissa DAQ-U ja DAQ-M — näissä kahdessa mallissa ”paljas” tarkoittaa yksinkertaisesti sitä, että suojuskorjausta ei sovelleta, kuten yllä mainittiin. Valitse tämä vaihtoehto vain, kun suojus on fyysisesti poistettu.

## Suojuksen ilmoittaminen — ja miksi se on tärkeää

**Ilmoitettu `cap_id`-koodin on vastattava anturissa fyysisesti olevaa suojusta.** Anturi tai ohjelmisto eivät pysty tunnistamaan asennettua suojusta. Ilmoitus vaikuttaa kahteen asiaan:

1. Jokaiseen spektriin sovellettu **reaaliaikainen korjaus**.
2. **Jokaiseen `.daq`-tallenteeseen kirjoitettava suojusmerkintä**, johon heijastavuuden jatkokäsittely luottaa.

Sunshine-korkki vaimentaa valoa **suunnittelunsa mukaan noin 12-kertaisesti**, joten tallennus, jossa on ilmoitettu väärä korkki, vääristää spektrien mittakaavaa suunnilleen tällä kertoimella. Ilmoita korkin vaihdoista välittömästi.

### Korkin asettaminen

GUI: Light Sensors -välilehti → anturirivin rataskuvake → **Cap**-pudotusvalikko. Kaikkien mallien oletusasetus on `sunshine_cosine` (kaikkiin DAQ-antureihin on asennettu kosinikorjain), ja valinta säilyy projektin ajan.

<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->

CLI (taustapalvelimen on oltava käynnissä):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

Malli CLI hyväksyy syntaktisesti koko `cap_id`-luettelon (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); jokainen profiili tarkistetaan anturin mallin perusteella yhteyden muodostamisen yhteydessä, joten käytettävissä olematon kapselitunnus (esimerkiksi E-only-tunnus DAQ-U:ssa) aiheuttaa selkeän virheilmoituksen sen sijaan, että se korjattaisiin virheellisesti. Taustapalvelimen oletusarvo, kun mitään ei välitetä, on `sunshine_cosine`.

Python SDK huomautus: `cap_id` ei **ole** SDK-säädin — `connect_daq_sensor()` / `DAQSensorSession` eivät paljasta cap-parametria. Valitse yläraja yllä olevien CLI-komentojen tai käyttöliittymän pudotusvalikon avulla; katso [SDK-viite](../reference/sdk-reference.md).

Edistynyt: profiilit toimitetaan Chloros-asennuksen mukana kohdassa `daq/cap_profiles/<u|m|e>/<cap_id>.json`, ja ne voidaan ohittaa käyttäjäkohtaisesti kohdassa `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Rajoituksista riippumatta antureille, joita ei ole koskaan kalibroitu uudelleen, tehdään automaattisesti pieni laivastosta johdettu pimeäoffsetin tarkistus – käyttäjän toimia ei tarvita.

## Auringonvalon rajoituksen suorituskyky (ulkokokoonpano)

Luvut, joiden perusteella voit laatia menettelytapoja:

| Ominaisuus | Arvo |
| --- | --- |
| Näkökenttä | 180° puolipallomainen |
| Kosinivastevirhe | ≤ ±4 % 60°:n tulokulmaan asti; ≤ ±4,5 % 70°:n tulokulmaan asti |
| Matalan auringon raja | Ei suositella alle ~15°:n aurinkokorkeudella |
| Vaimennus | ~12× (suunnittelun mukainen) |
| Suojan uudelleenasennuksen toistettavuus | ≈ 1,5 % |
| Määrällinen säteilyvoimakkuus | Keskimääräinen **≥ 15 s**:n lukemien keskiarvo (laitteen ominaisuus, ei vika) |

Kaikissa määrällisissä säteilyvoimakkuusarvoissa — heijastavuusviitteet mukaan lukien — on käytettävä vähintään 15 sekunnin lukemien keskiarvoa yksittäisen kuvan sijaan.

## Kalibroitu spektrialue

| Ominaisuus | Arvo |
| --- | --- |
| Spektrinäytteenotto | 340–1010 nm 5 nm:n välein (135 pistettä) |
| Radiometrisesti kalibroitu alue | **~374–974 nm** (ohjelmistossa pakollinen) |

Anturi raportoi koko 340–1010 nm:n ruudukon, mutta NIST-jäljitettävä radiometrinen vahvistus kattaa alueen ~374–974 nm. Chloros **hylkää absoluuttisen heijastavuuden jakamisen** kaikille kamerakaistoille, joiden spektripainosta alle puolet sijaitsee kyseisellä alueella, ja ilmoittaa ohitussyyn `dls-uncalibrated-band-<nm>` sen sijaan, että tuottaisi kalibroimattoman tuloksen. Toimitettavista kameramalleista ainoastaan F988-suodatin jää tämän alueen ulkopuolelle; siinä käytetään sen sijaan heijastavuuspaneelin työnkulkua — katso [Heijastavuustyönkulut](reflectance.md).

Anturimallit, siirtoformaatit ja anturitunnukset löytyvät [DAQ-yleiskatsauksesta](README.md). Tietoja siitä, miten cap-leima kulutetaan käsittelyn aikana, löytyy kohdasta [Tallennus ja .daq-muoto](recording.md).
