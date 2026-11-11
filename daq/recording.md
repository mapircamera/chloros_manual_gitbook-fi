# Tallennus ja .daq-tiedostomuoto

`.daq`-tiedosto on Chloros in valosensorin tallennusmuoto: **SQLite-tietokanta**, joka sisältää yhden DAQ-anturin kalibroituja spektrikuvia. Tallenna yksi tiedosto tallennusistunnon aikana, niin heijastavuusprosessointiputki voi myöhemmin jakaa jokaisen kuvan juuri sillä hetkellä mitatulla alaspäin suuntautuvalla säteilyvoimakkuudella.

## Mitä .daq-tiedosto sisältää

| Ominaisuus | Arvo |
| --- | --- |
| Säilytysmuoto | SQLite-tietokanta, yksi tiedosto anturia ja tallennusta kohti |
| Tiedostonimi | Sisältää **anturin tunnisteen**ja**aikaleiman**, esim. `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spektri kehystä kohti | 135 pistettä, 340–1010 nm 5 nm:n välein, sekä CIE XYZ-kolmivärikuvaus |
| Yksiköt | Kalibroitu spektrinen säteilyvoimakkuus, **W/m²/nm** (tehdaskalibrointipaketti + suojuksen profiili sovellettu) |
| Leimatut metatiedot | Anturin tunnus (avain kyseisen laitteen tehdaskalibroinnin hakemiseen) ja voimassa oleva suojusprofiili — katso [Suojusprofiilit ja kalibroitu alue](caps-and-range.md) |

Muoto on identtinen malleissa DAQ-U, DAQ-M, DAQ-E ja DAQ-E-S, joten jatkokäsittelyssä ei ole merkitystä, mikä siirtolaite tallensi tiedot. DAQ-E-S-tallenteessa tallennetaan myös laitteen kallistus, kierto ja nousukulma jokaisen spektrin yhteyteen.

**Tallenteet säilyvät sähkökatkon jälkeen (1.3.0).**Reaaliaikainen tallenne kirjoitetaan lokitiedostoon ja levyn täyssynkronoinnin yhteydessä jokaisen tallennuksen yhteydessä, joten sähkökatko tai keskeytetty prosessi menettää vain noin viimeiset 3 sekuntia, ei koko tiedostoa. Kirjoituksen kesken pysähtynyt tallenne voi jättää `<name>.daq-journal`-tiedoston `.daq`-tiedoston viereen:**säilytä ja kopioi molemmat yhdessä.** Chloros palauttaa lokin ensimmäisellä kerralla, kun se lukee tiedoston. Kiireinen levy viivästyttää tiedoston tallennusta, mutta ei koskaan anturin lukemia.

**DAQ-E, joka lakkaa lähettämästä dataa, kytketään uudelleen (1.3.0).** Kun DAQ-E:n datavirta katkeaa (kytkimen virta katkaistaan ja kytketään uudelleen, yhteys katkeaa), taustapalvelu yhdistää sen uudelleen aiemmilla asetuksilla, ja tallennus jatkuu samassa `.daq`-tiedostossa, jossa katkos on merkitty.

Kalibroitu tallennus edellyttää anturin tehdaskalibrointipakettia. DAQ-U- ja DAQ-M-laitteiden osalta taustapalvelu hakee paketin MAPIR in pilvestä anturin tunnisteen perusteella (tallennus hylätään, jos tämä ei onnistu); DAQ-E-laitteet ovat poikkeus, koska niiden kalibrointitiedot säilytetään laitteessa.

## Tallennus käyttöliittymästä

Tallennus graafisessa käyttöliittymässä edellyttää **avointa projektia** (muussa tapauksessa Tallennus-painikkeet ovat pois käytöstä):

* **Tallenna kaikki / Lopeta kaikki** — valosensorien sivupalkin yläosassa; käynnistää tai lopettaa `.daq`-tallennuksen kaikissa kytketyissä antureissa kerralla.
* **Tallenna / Lopeta tallennus** — antikohtaisesti, hammasratasasetusten modaalissa. Punainen ”REC”-merkkivalo näkyy anturin reaaliaikaisissa tietoriveissä tallennuksen aikana.

Tiedostot tallennetaan `<project>/light_sensor/`-kansioon, ja kun tallennus lopetetaan — joko Stop-, Stop All -painikkeella tai irrottamalla tallennusanturi — valmis `.daq`-tiedosto **lisätään automaattisesti avoimeen projektiin**. Se näkyy projektin tiedostoluettelossa ilman manuaalista lisäystä, valmiina heijastavuusarvojen käsittelyä varten.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->



## Tallennus CLI:n kautta

CLI tallentaa taustapalvelimen anturipoolin kautta (taustapalvelimen on oltava käynnissä — nämä komennot ovat kevyitä HTTP-asiakasohjelmia):

```bash
# Connect the sensor into the backend pool
chloros-cli daq pool-connect --eth-host daq-e-def330.local

# Record for 150 seconds, with a human-friendly device label
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
    -o ./out --device-name "rooftop-A"

# Or run open-ended and stop explicitly
chloros-cli daq pool-record --sensor-id daq-e-def330            # --duration defaults to 0 = run until --stop
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

Hae arvo `--sensor-id` kohteesta `chloros-cli daq pool-list`. Kaksi hyödyllistä oletusarvoa:

| Optio | Oletusarvo |
| --- | --- |
| `--duration` | `0` — tallenna, kunnes `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` **taustapalvelimen** tiedostojärjestelmässä, ei CLI:n |

Tulostuskansioiden ero on merkityksellinen, kun CLI:n kohde on toisella koneella sijaitseva taustapalvelin: tiedosto tallentuu sinne, missä taustapalvelin toimii. Kun taustapalvelin sijaitsee samalla tietokoneella, suhteellinen `-o` (kuten yllä oleva `./out`) määritetään sen kansion suhteen, jossa komento suoritettiin.

## Tallennus Python

ista `DAQSensorSession` (jonka palauttaa `chloros_sdk.connect_daq_sensor()`) paljastaa saman yhdistettyjen tallenteiden joukon: `record_start(output_dir=None, device_name=None)` palauttaa tiedostopolun, `record_stop()` palauttaa `{path, rows}`:n. Katso [SDK-viite](../reference/sdk-reference.md) saadaksesi täydellisen istunto-API:n. SDK:n suorat laitteistoluokat (vain työpöytäasennukset) kirjoittavat tallenteet oletusarvoisesti `~/Documents/DAQ/`:ään; julkaistuissa versioissa yllä mainittu yhdistetty polku on tuettu reitti.

## .daq-tiedoston käyttö käsittelyvaiheessa

Jotta Chloros voi laskea kuvien heijastavuuden, se tarvitsee jokaiselle valotukselle vastaavan alaspäin suuntautuvan säteilyn voimakkuuden:

* **Säilytä `.daq` kuvien mukana.**Käsittelyvaiheessa prosessiketju määrittää**aikaleimaltaan vastaavan alaspäin suuntautuvan säteilyn** automaattisesti tallennetusta `.daq`-tiedostosta (mikä tahansa DAQ-malli) — tai kuvien vierestä löytyvästä DAQ-M-muotoisesta `.csv`-tiedostosta — joka löytyy kuvien vierestä. GUI-tallenteet täyttävät tämän vaatimuksen automaattisesti, koska ne lisätään projektiin heti, kun ne pysähtyvät.
* **Kalibrointi haetaan tarpeen mukaan.** Jos kamera- tai DAQ-kohtaista tehdaskalibrointipakettia ei ole vielä tallennettu paikalliseen välimuistiin, Chloros hakee sen automaattisesti MAPIR in pilvestä ensimmäisellä käyttökerralla (vaatii internet-yhteyden kerran; tallennetaan välimuistiin tiedostona `~/.chloros/`).
* **Reaaliaikaiset tallenteet kirjoittavat oman sivutiedostonsa.** Jokaisesta reaaliaikaisesti tallennetusta heijastavuuskehyksestä tallennetaan tosiasiallisesti käytetty DAQ-lukema `.daq`-tiedostona tallenteiden `daq/`-kansioon, jotta tallennetta voidaan käsitellä uudelleen myöhemmin ilman alkuperäistä tallennetta.
* **`chloros_scripts`-tallentimista peräisin olevat tallenteet (1.3.0).**Julkisten [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)-tallentimien kirjoittama `.daq`-tiedosto näkyy sen tuottaneena DAQ-mallina, ja sen yläraja voidaan asettaa kohdassa**Projektin asetukset → DAQ-valosensori** kuten minkä tahansa muun tallennuksen kohdalla. Nämä tallentimet tallentavat raakatiedot anturilukemista; Chloros hakee anturin tehdaskalibroinnin sarjaliitännän kautta ja soveltaa sitä.
* **Kenen lukema se on (1.3.0).**Jokaisen `.daq`-tallennuksen vieressä on `.daq.cameras.json`-merkintä, joka nimeää kameran tai kamerat, joiden kuvia kyseinen lukema kuvaa, ja käsittely sovittaa sen**vain kyseisten kameroiden kuviin**. Itsenäinen kamera, jolla ei ole omaa valosensoria, ei siis enää lainaa toisen kameran lukemaa — liitä siihen sensori tai tallenna oma lukema. Itse DAQ:lla tehty tallenne (GUI Record, `daq pool-record`) sekä kaikki `.daq`-tiedostot, joissa kyseistä tiedostoa ei ole (mukaan lukien vanhemmat tallenteet), palvelevat edelleen kaikkia kameroita.

## Säteilyvoimakkuuden palauttaminen

Projektin käsittely vie myös kaikki sen sisältämät valosensoritallenteet
`Light Sensor/`-kansioon kuvatuotteiden viereen. Tämä ei **vaadi** kuvamateriaalia:
yksinään tallennettu valosensori on täydellinen tallenne, ja kansio, joka sisältää vain `.daq`-
tiedostoja, on kelvollinen syöte. Projekti, joka sisältää vain valosensoritallenteita, listaa ne ja
käsittelee ne sekä sovelluksessa että CLI:ssä, ja ne pysyvät luettelossa käsittelyn jälkeen. Käsittely
ilmoittaa, kuinka monta valosensorituotetta se on tallentanut.

| Tuote | Mikä se on |
| --- | --- |
| `<name>_calibrated.daq` | Uudelleenkäsiteltävä arkisto, joka noudattaa samaa kaavaa kuin reaaliaikainen tallenne, mutta jossa ilmoitetaan sen tuottanut kalibrointipaketti. Sen uudelleen tuominen **ei** kalibroi sitä toista kertaa. |
| `<name>_calibrated.csv` | Spektrinen säteilyvoimakkuus yksikössä W/m²/nm anturin omalla aallonpituusruudukolla, yksi rivi lukemaa kohti, sekä fotometriset sarakkeet: kokonaisteho, fotopinen ja skotopinen lux, PPFD sinisen/vihreän/punaisen jakaumalla ja huippuaallonpituus. |

DAQ-U- tai DAQ-M-laite, jonka kalibrointipakettia ei voida hakea — olet offline-tilassa tai
kyseisellä anturilla ei ole kalibrointitietoja tiedostossa — **ohitetaan syyn kera**, eikä sitä koskaan tallenneta
”kalibroituna” tiedostona, joka sisältää raakalukemia. Yhdistä internetiin ja suorita uudelleen. DAQ-E
sisältää oman kalibrointinsa, joten se tarvitsee tätä vain silloin, kun laite ei ole kytkettynä ja
paikalliseen välimuistiin ei ole tallennettu mitään.

### DAQ-A: raakalukemat, ja miksi se on oikea ratkaisu

**DAQ-A** on peräisin ajalta ennen sarjanumeroittaista kalibrointipakettijärjestelmää, eikä sillä ole pakettia, jota
voitaisiin hakea. Tämä ei ole laiminlyönti: DAQ-A kalibroidaan kentällä
heijastavuuskohteen avulla, ja kohdepohjainen kalibrointi vaatii vain anturin *suhteellisen*
vasteen — mikä on juuri sen raakalukujen sisältö. Chloros kalibroi laitteet nykyään niiden avulla.

Joten DAQ-A-tallenne viedään, mutta eri nimellä:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, ei `_calibrated` — eri tiedostonimi eikä tiedoston sisäinen merkki,
koska tiedoston nimi on säilytettävä, kun tiedosto lähetetään sähköpostitse pelkkänä nimenä. `.csv`
-otsikossa lukee `raw spectral sensor counts (NOT irradiance)` ja siinä varoitetaan, että arvoja voidaan
verrata **tiedoston sisällä**, mutta ei eri anturien välillä. Sarakkeet, joilla on merkitystä
vain todelliselle säteilyvoimakkuudelle — kokonaisteho, lux, PPFD — jätetään tyhjiksi sen sijaan, että ne
laskettaisiin laskentojen perusteella.

Vanhemmat DAQ-A-SD-tallenteet (skeema v1.01 / v1.02) tallentavat vain tiedoston kirjoitusajan, eivät
aikaleimaa kutakin lukemaa kohden. Chloros ei täsmää kuvia näiden kanssa — kehyksen yhdistäminen
kirjoitusajankohtaan olisi virheellistä, vaikka se ei näyttäisikään virheelliseltä — mutta vienti lukee ne ongelmitta ja
CSV-tiedostossa ilmoitetaan, mihin kelloon ne perustuvat.

Kattava kuvaus heijastavuuden käsittelystä — yksi anturi kameran kanssa ja kaksi anturia (ympäristö/kohde) — löytyy kohdasta [Heijastavuuden työnkulut](reflectance.md).
