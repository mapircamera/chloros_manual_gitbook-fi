# Anturikupujen profiilit ja kalibroitu mittausalue

> Itse suojukset — eli mitkä suojukset toimitetaan minkäkin anturin mukana, miten ne asennetaan ja niiden optiset ominaisuudet — on dokumentoitu **[DAQ-käyttöoppaassa](https://mapir.gitbook.io/daq)**. Tällä sivulla käsitellään asennetun suojuksen *ilmoittamista* Chloros-järjestelmään, mikä varmistaa korjauksen oikeellisuuden.

Jokaisen DAQ-valosensorin tehtaan radiometrinen kalibrointi kuvaa *paljaata* anturia. Diffuusorin päälle asennettu fyysinen suojus muuttaa anturin keräämää valoa, joten Chloros soveltaa tehtaan mittaamaa **suojuksen korjausprofiilia** kalibrointipaketin päälle. Oikean suojuksen ilmoittaminen on osa kalibroitujen tietojen saamista — tällä sivulla kerrotaan, mitä suojuksia on saatavilla mallikohtaisesti, miten ne ilmoitetaan ja mikä on anturin kalibroitu spektrialue.

## Suojusten saatavuus mallikohtaisesti

| Suojaprofiili (`cap_id`) | Fyysinen suojus | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Sunshine-kosinikorjauskorkki (**oletus kaikissa malleissa**) | Kyllä | Kyllä | Kyllä |
| `fov_15` / `fov_45` / `fov_90` | Näkökenttää rajoittavat kartiot (15° / 45° / 90°) | Kyllä | — | Kyllä |
| `fov_30` / `fov_60` | Näkökenttää rajoittavat kartiot (30° / 60°) | Kyllä | — | — |
| `none` | Ei suojakantta | — | — | Kyllä |

Mallikohtaiset huomautukset:

* **DAQ-M:llä on yksi suojakantaprofiili: `sunshine_cosine`.** Tuotemääritelmänä on ”Bare-plus-Sunshine-cap”, ja paljaalle DAQ-M:lle ei tarvita geometriprofiilia.
* **Paljas DAQ-U on täysin paljas** — se ei tarvitse lainkaan geometriprofiilia, minkä vuoksi sille ei ole olemassa `none`-profiilia.
* **`none` DAQ-E-mallissa EI OLE toimimaton.** DAQ-E:n upotetulla, lasilla peitetyllä hajottajalla on oma todellinen geometrian korjaus, joten ”ei suojakantta” on itsessään mitattu profiili tässä mallissa.
* **Paljas DAQ-E ei voi mitata suoraa auringonvaloa missään korkeudessa** — Sunshine-suojus on kenttäasetus. Älä suunnittele ulkotöitä paljaan DAQ-E:n ympärille.

GUI:n anturikohtaisissa asetuksissa (hammasrataskuvake Valoanturit-välilehdessä) **Suojus**-pudotusvalikosta löytyy myös vaihtoehto ”Ei mitään (paljas anturi)” malleissa DAQ-U ja DAQ-M — näissä kahdessa mallissa ”paljas” tarkoittaa yksinkertaisesti sitä, että suojuskorjausta ei sovelleta, kuten yllä mainittiin. Valitse tämä vaihtoehto vain, kun suojus on fyysisesti poistettu.

## Suojuksen ilmoittaminen — ja miksi sillä on merkitystä

**Ilmoitettu `cap_id`-koodin on vastattava anturissa fyysisesti olevaa suojusta.** Anturi tai ohjelmisto eivät pysty tunnistamaan asennettua suojusta. Ilmoitus vaikuttaa kahteen asiaan:

1. Jokaiseen spektriin sovellettu **reaaliaikainen korjaus**.
2. **Jokaiseen `.daq`-tallenteeseen kirjoitettu suojusmerkintä**, johon heijastavuuden jatkokäsittely perustuu.

Sunshine-suojus vaimentaa **suunnittelunsa mukaisesti noin 12-kertaisesti**, joten tallennus, jossa on ilmoitettu väärä suojus, vääristää spektrien mittakaavaa suunnilleen tällä kertoimella. Ilmoita suojuksen muutoksista välittömästi.

### Suojuksen asettaminen

GUI: Light Sensors -välilehti → hammasrataskuvake anturirivillä → **Cap**-pudotusvalikko. Oletusarvo kaikille malleille on `sunshine_cosine`, mikä on oletus (DAQ-U-, DAQ-M- ja DAQ-E-mallit toimitetaan ilman korkkia, ja tallenteessa oletetaan, että korkkia käytettiin, kunnes valitset toisen), ja valinta säilyy projektin mukana.



CLI<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->
(taustapalvelimen on oltava käynnissä):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

CLI hyväksyy syntaktisesti koko `cap_id`-luettelon (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); jokainen profiili tarkistetaan anturin mallin perusteella yhteyden muodostamisen yhteydessä, joten käytettävissä olematon cap-tunnus (esimerkiksi pelkästään E-tunnus DAQ-U:ssa) aiheuttaa selkeän virheilmoituksen sen sijaan, että se korjattaisiin virheellisesti. Taustapalvelimen oletusarvo, kun mitään ei välitetä, on `sunshine_cosine`.

Python-SDK: nimeä cap yhteyden muodostuksen yhteydessä `connect_daq_sensor(cap_id="sunshine_cosine")`:llä tai vaihda se suorituksen aikana `DAQSensorSession.set_cap("fov_45")`:llä; katso [SDK-viite](../reference/sdk-reference.md#daq-sensor-sessions).

Edistynyt: profiilit toimitetaan Chloros-asennuksen mukana kohdassa `daq/cap_profiles/<u|m|e>/<cap_id>.json`, ja ne voidaan ohittaa käyttäjäkohtaisesti kohdassa `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Rajoituksista riippumatta anturit, joita ei ole koskaan kalibroitu uudelleen, saavat automaattisesti pienen, laitteistosta johdetun pimeäoffsetin tarkennuksen — ilman käyttäjän toimia.

## DAQ-E-S: kiinteä hajotin ja kallistus

**DAQ-E-S** on DAQ-E, jossa on kaksi Chloros ille merkittävää eroa. Kaikki muu (tunnistus, liittäminen, PTP, tallennus, laiteasetukset) toimii täsmälleen kuten DAQ-E:ssä.

* **Sen hajotin on kiinteä, joten se ei vaadi cap-korjausta.** Diffuusori oli laitteessa jo silloin, kun sen tehdaskalibrointi mitattiin, joten kalibrointi osoittaa jo todellista alaspäin suuntautuvaa säteilyvoimakkuutta. Chloros ei sovella siihen mitään cap-profiilia riippumatta siitä, mitä pyydetään: sille nimetty cap ohitetaan viestillä, sen tallenteissa mainitaan, että cap on peräisin mallista, eikä oletetun capin varoitus koskaan laukea sen kohdalla. Kohdassa Projektin asetukset → DAQ-valosensori sen rivillä ei näy ylärajaa, eikä projektikohtainen ylärajan ohitus vaikuta siihen. Älä sovella ylärajaa sen tietoihin manuaalisesti: sen päälle tehty auringonpaisteen korjaus antaisi noin 11-kertaisen lukeman.
* **Siinä on 3-akselinen kiihtyvyysanturi.** Diffuusorin kosinivaste on voimassa vain, kun se on vaakasuorassa, joten kallistunut anturi ilmoittaa väärän säteilyvoimakkuuden ilman muita oireita. Jokainen DAQ-E-S-lukema sisältää siksi sen kallistuksen, rullauksen ja pituuskallistuksen, ja yhdestä laitteesta tallennettu `.daq` tallentaa ne jokaisen spektrin viereen (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, …), joten voit erottaa pilven kiinnikkeen kallistumisesta.

Katso jokaisen asennon kentän tiedot ja miten asennon datavirrassa esiintyvä vika raportoidaan kohdasta [DAQ-E-S CLI-ohjeessa](../reference/cli-reference.md#daq-e-s--fixed-diffuser-and-tilt-sensor).

## Sunshine Capin suorituskyky (ulkokokoonpano)

Luvut, joiden perusteella voit laatia menettelytapoja:

| Ominaisuus | Arvo |
| --- | --- |
| Näkökenttä | 180° puolipallomainen |
| Kosinivastevirhe | ≤ ±4 % 60°:n tulokulmaan asti; ≤ ±4,5 % 70°:n tulokulmaan asti |
| Matalan auringon raja | Ei suositella alle ~15°:n auringon korkeuskulmalla |
| Vaimennus | ~12× (suunnittelun mukainen) |
| Suojuksen uudelleenasennuksen toistettavuus | ≈ 1,5 % |
| Määrällinen säteilyvoimakkuus | Keskimääräinen **≥ 15 s**:n lukema (laitteen ominaisuus, ei vika) |

Kaikissa määrällisissä säteilyvoimakkuusarvoissa — heijastavuusviitteet mukaan lukien — käytä vähintään 15 sekunnin lukemien keskiarvoa yhden kuvan sijaan.

## Kalibroitu spektrialue

| Ominaisuus | Arvo |
| --- | --- |
| Spektrinäytteenotto | 340–1010 nm 5 nm:n välein (135 pistettä) |
| Radiometrisesti kalibroitu alue | **~374–974 nm** (ohjelmistossa pakollinen) |

Anturi raportoi koko 340–1010 nm:n ruudukon, mutta NIST-jäljitettävä radiometrinen vahvistus ulottuu alueelle ~374–974 nm. Chloros **hylkää absoluuttisen heijastavuuden jaon** kaikille kameran kaistoille, joiden spektrinen paino on alle puolet kyseisestä alueesta, ja ilmoittaa ohitussyyn `dls-uncalibrated-band-<nm>` sen sijaan, että tuottaisi kalibroimattoman tuloksen. Myynnissä olevista kameramalleista vain F988-suodatin jää tämän alueen ulkopuolelle; siinä käytetään sen sijaan heijastavuuspaneelin työnkulkua — katso [Heijastavuustyönkulut](reflectance.md).

Anturimallit, siirtoalustat ja anturitunnukset löytyvät [DAQ-yleiskatsauksesta](README.md). Tietoa siitä, miten cap-leima kulutetaan käsittelyn aikana, löytyy kohdasta [Tallennus ja .daq-muoto](recording.md).
