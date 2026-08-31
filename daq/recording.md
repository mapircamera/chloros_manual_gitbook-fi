# Tallennus ja .daq-tiedostomuoto

`.daq`-tiedosto on Chloros:n valosensorin tallennusmuoto: **SQLite-tietokanta**, joka sisältää yhden DAQ-anturin kalibroituja spektrikuvia. Tallentamalla yhden tällaisen tiedoston mittaussession aikana heijastavuuslaskentaprosessi voi myöhemmin jakaa jokaisen kuvan juuri sillä hetkellä mitatulla alaspäin suuntautuvalla säteilyvoimakkuudella.

## Mitä .daq-tiedosto sisältää

| Ominaisuus | Arvo |
| --- | --- |
| Säilytysmuoto | SQLite-tietokanta, yksi tiedosto anturia ja tallennusta kohti |
| Tiedostonimi | Sisältää **anturitunnuksen**ja**aikaleiman**, esim. `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spektri kehystä kohti | 135 pistettä, 340–1010 nm 5 nm:n välein, sekä CIE XYZ-kolmivärikuvaus |
| Yksiköt | Kalibroitu spektrinen säteilyvoimakkuus, **W/m²/nm** (tehdaskalibrointipaketti + suojuksen profiili sovellettu) |
| Tallennetut metatiedot | Anturin tunniste (avain kyseisen laitteen tehdaskalibroinnin hakemiseen) ja voimassa oleva cap-profiili — katso [Cap-profiilit ja kalibroitu alue](caps-and-range.md) |

Muoto on identtinen DAQ-U-, DAQ-M- ja DAQ-E-laitteissa, joten jatkokäsittelyssä ei ole merkitystä, mikä siirtolaitteisto on tallentanut tiedot.

Kalibroitu tallennus edellyttää anturin tehdaskalibrointipakettia. DAQ-U- ja DAQ-M-laitteiden osalta taustapalvelu hakee paketin MAPIR:n pilvestä anturitunnuksen perusteella (tallennus hylätään, jos tämä ei onnistu); DAQ-E-laitteet ovat poikkeus, koska niiden kalibrointitiedot ovat tallennettuina laitteeseen.

## Tallennus käyttöliittymästä

Tallennus graafisessa käyttöliittymässä edellyttää **avointa projektia** (muussa tapauksessa Tallennus-painikkeet ovat poissa käytöstä):

* **Tallenna kaikki / Lopeta kaikki** — valosensorien sivupalkin yläosassa; käynnistää tai lopettaa `.daq`-tallennuksen kaikissa kytketyissä antureissa kerralla.
* **Tallenna / Lopeta tallennus** — antinkohtaisesti, hammasratasasetusten modaalissa. Punainen ”REC”-merkkivalo näkyy anturin reaaliaikaisissa tietoriveissä tallennuksen aikana.

Tiedostot tallennetaan `<project>/light_sensor/`-kansioon, ja kun tallennus lopetetaan — joko Lopeta-, Lopeta kaikki -painikkeella tai irrottamalla tallennusanturi — valmis `.daq`-tiedosto **lisätään automaattisesti avoimeen projektiin**. Se näkyy projektin tiedostoluettelossa ilman manuaalista lisäysvaihetta, valmiina heijastuskyvyn käsittelyä varten.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->

## Tallennus CLI:stä

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

Hae `--sensor-id`-arvo `chloros-cli daq pool-list`:sta. Kaksi hyödyllistä oletusarvoa:

| Vaihtoehto | Oletusarvo |
| --- | --- |
| `--duration` | `0` — tallenna, kunnes `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` **taustajärjestelmän** tiedostojärjestelmässä, ei CLI:n |

Tiedostokansion ero on merkityksellinen, kun CLI kohdistuu toisella koneella olevaan taustapalvelimeen: tiedosto tallentuu sinne, missä taustapalvelin toimii.

## Tallennus Python:stä

`DAQSensorSession` (palautettu `chloros_sdk.connect_daq_sensor()`:n toimesta) paljastaa saman yhdistettyjen tallenteiden joukon: `record_start(output_dir=None, device_name=None)` palauttaa tiedostopolun, `record_stop()` palauttaa `{path, rows}`:n. Katso [SDK-viite](../reference/sdk-reference.md) koko istunnon API osalta. SDK:n suorat laitteistoluokat (vain työpöytäasennukset) tallentavat tallenteet oletuksena `~/Documents/DAQ/`:ään; julkaistuissa versioissa yllä mainittu yhdistetty polku on tuettu reitti.

## .daq-tiedoston käyttö käsittelyvaiheessa

Heijastavuuden laskemiseksi kuvamateriaalista Chloros tarvitsee kuhunkin valotukseen sovitetun alaspäin suuntautuvan säteilyn voimakkuuden:

* **Säilytä `.daq` kuvamateriaalin mukana.**Käsittelyvaiheessa prosessiketju määrittää**aikaleimalla sovitetun alaspäin suuntautuvan säteilyn** automaattisesti tallennetusta `.daq`-tiedostosta (mikä tahansa DAQ-malli) — tai DAQ-M:n omasta `.csv`-tiedostosta — joka löytyy kuvien vierestä. GUI-tallenteet täyttävät tämän vaatimuksen automaattisesti, koska ne lisätään projektiin heti, kun ne päättyvät.
* **Kalibrointi haetaan tarpeen mukaan.** Jos kamera- tai DAQ-kohtaista tehdaskalibrointipakettia ei ole vielä tallennettu paikalliseen välimuistiin, Chloros hakee sen automaattisesti MAPIR:n pilvipalvelusta ensimmäisellä käyttökerralla (vaatii internet-yhteyden kerran; tallennetaan välimuistiin `~/.chloros/`:n alle).
* **Reaaliaikaiset tallenteet kirjoittavat oman sivutiedostonsa.** Jokaisesta reaaliaikaisesti tallennetusta heijastavuuskehyksestä tallennetaan kuvamateriaalin viereen `.daq`-sivutiedostona se DAQ-lukema, jota tosiasiallisesti käytettiin, jotta tallennetta voidaan käsitellä uudelleen myöhemmin ilman alkuperäistä tallennetta.

## Säteilyvoimakkuuden palauttaminen

Projektin käsittely vie myös kaikki sen sisältämät valosensoritallenteet
`Light Sensor/`-kansioon kuvatuotteiden viereen. Tämä ei **vaadi** kuvamateriaalia:
yksinään lennetty valosensori on itsessään täydellinen tallenne, ja kansio, joka sisältää vain `.daq`-
tiedostoja, on kelvollinen syöte. Suoritus raportoi, kuinka monta valosensorituotetta se on tallentanut.

| Tuote | Mikä se on |
| --- | --- |
| `<name>_calibrated.daq` | Uudelleenkäsiteltävä arkisto, joka noudattaa samaa rakennekaavaa kuin reaaliaikainen tallenne, mutta jossa ilmoitetaan sen tuottanut kalibrointipaketti. Sen uudelleen tuominen **ei** kalibroi sitä toista kertaa. |
| `<name>_calibrated.csv` | Spektrinen säteilyvoimakkuus yksikössä W/m²/nm anturin omalla aallonpituusruudukolla, yksi rivi lukemaa kohti, sekä fotometriset sarakkeet: kokonaisteho, fotopinen ja skotopinen lux, PPFD sinisen/vihreän/punaisen jakaumalla sekä huippuaallonpituus. |

DAQ-U- tai DAQ-M-laite, jonka kalibrointipakettia ei voida hakea — olet offline-tilassa tai
kyseisellä anturilla ei ole kalibrointitietoja tiedostossa — **ohitetaan syyn kera**, eikä sitä koskaan tallenneta
”kalibroituna” tiedostona, joka sisältää raakalukemia. Yhdistä internetiin ja suorita käsittely uudelleen. DAQ-E
sisältää oman kalibrointinsa, joten se tarvitsee tätä vain silloin, kun laite ei ole kytkettynä ja
paikalliseen välimuistiin ei ole tallennettu mitään.

### DAQ-A: raakalukemat, ja miksi se on oikea ratkaisu

**DAQ-A** on peräisin ajalta ennen sarjanumeroittain määriteltävää kalibrointipakettijärjestelmää, eikä sillä ole pakettia, jota
voitaisiin hakea. Tämä ei ole laiminlyönti: DAQ-A kalibroidaan kentällä
heijastavuuskohdetta vasten, ja kohdepohjainen kalibrointi vaatii vain anturin *suhteellisen*
vasteen — mikä on juuri sen raakalukemia. Chloros kalibroi niiden avulla nykyäänkin.

Joten DAQ-A-tallenne viedään, mutta eri nimellä:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, ei `_calibrated` — eri tiedostonimi eikä tiedoston sisäinen merkki,
koska tiedoston nimi on säilytettävä, kun tiedosto lähetetään sähköpostitse pelkkänä nimenä. `.csv`
-otsikossa lukee `raw spectral sensor counts (NOT irradiance)` ja varoitetaan, että arvoja voidaan
verrata **tiedoston sisällä**, mutta ei eri anturien välillä. Sarakkeet, joilla on merkitystä
vain todellisen säteilyvoimakkuuden osalta — kokonaisteho, lux, PPFD — jätetään tyhjiksi sen sijaan, että ne
laskettaisiin laskentojen perusteella.

Vanhemmat DAQ-A-SD-tallenteet (skeema v1.01 / v1.02) tallentavat vain tiedoston kirjoitusajan, eivät
aikaleimaa kutakin lukemaa kohden. Chloros ei täsmää kuvia näiden kanssa — kehyksen yhdistäminen
kirjoitusajankohtaan olisi virheellistä, vaikka se ei näyttäisikään virheelliseltä — mutta vienti lukee ne oikein ja
CSV ilmoittaa, minkä kellon mukaan tiedot on tallennettu.

Kattava kuvaus heijastavuudesta – yksi anturi kameran kanssa ja kaksi anturia (ympäristö/kohde) – löytyy kohdasta [Heijastavuuden työnkulut](reflectance.md).
