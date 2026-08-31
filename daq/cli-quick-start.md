# CLI Pikaopas (pool-*)

Toimitettu `chloros-cli` ohjaa DAQ-antureita **`daq pool-*`** -komentoperheen kautta — ohuita HTTP-asiakasohjelmia, jotka ohjaavat anturia Chloros-taustapalvelimen pysyvän anturipoolin kautta. Taustapalvelu hallitsee siirtoa, joten graafinen käyttöliittymä, CLI sekä SDK-skriptit jakavat kaikki yhden aktiivisen käsittelykäsittimen sen sijaan, että ne kilpailevat portista. Kaikki, mitä asiakas tarvitsee, on saavutettavissa `pool-*`:n kautta: yhteyden muodostaminen, datavirran lähetys, kalibroitujen `.daq`-tiedostojen tallennus ja kap-profiilien vaihtaminen.

`pool-*` on myös **ainoa** DAQ-käyttöliittymä julkaistuissa versioissa. `chloros-cli daq --help` listaa `pool-*`-alikomennot, ja jos suoritat laitteistoon suoraan vaikuttavan DAQ-alikomennon julkaistussa versiossa, ohjelma lopetetaan nimenomaisella virheilmoituksella, jossa mainitaan puuttuva paketti ja ohjataan sinut takaisin `pool-*`:ään — mikään ei epäonnistu hiljaisesti. (Suoraan laitteistoon kohdistuvat komennot toimivat vain MAPIR-lähdekoodin tarkistuksesta; `pip install chloros-sdk` ei myöskään tarjoa niitä.)

***

## Edellytykset

* **Chloros-taustapalvelimen on oltava käynnissä** — `pool-*`-komennot ovat HTTP-asiakasohjelmia, eivät laiteajureita. Windows-laitteessa käynnistä Chloros-työpöytäsovellus (se käynnistää taustapalvelun). Käynnistä palvelu headless-laitteessa Linux/Jetson: `sudo systemctl enable --now chloros-backend.service`.
* **Chloros+ (maksullinen taso) -kirjautuminen**: suorita ensin `chloros-cli login`. Valvonta tapahtuu palvelinpuolella — ilman kirjautumista komennot epäonnistuvat virheellä `401 AUTH_REQUIRED`; ilmaisella (Iron) tasolla ne epäonnistuvat virheellä `403 PLAN_UPGRADE_REQUIRED`.
* Komennot kohdistuvat oletusarvoisesti `http://127.0.0.1:5000`:ään; `daq pool-*`-perhe noudattaa `CHLOROS_BACKEND_URL`-ympäristömuuttujaa, jos taustapalvelimesi toimii muualla.

***

## Viiden minuutin istunto

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)

# 2. List the pool — this shows the sensor_id used by every command below
chloros-cli daq pool-list

# 3. Read the most recent calibrated spectrum frame (add --json for scripting)
chloros-cli daq pool-latest --sensor-id daq-e-def330 --json

# 4. Record a calibrated .daq file for 60 seconds
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 60 \
  --device-name "field-A"

# 5. Release the sensor when done
chloros-cli daq pool-disconnect --sensor-id daq-e-def330
```

***

## `pool-connect` — avaa anturi poolista

| Vaihtoehto | Merkitys |
| --- | --- |
| `daq pool-connect` | Älykäs tunnistus: etsi kaikki tämän koneen DAQ-laitteet. |
| `daq pool-connect --port PORT` | DAQ-U tietyllä sarjaportilla (esim. `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | DAQ-M BLE-yhteydellä, MAC-osoite skannattu automaattisesti. |
| `daq pool-connect --mac MAC` | DAQ-M tunnetulla BLE-MAC-osoitteella (tarkoittaa `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E tunnetulla isäntänimellä tai IP-osoitteella — **luotettava reitti**. |
| `daq pool-connect --eth` | DAQ-E automaattisella tunnistuksella (mDNS, ARP-varajärjestelmällä). Katso alla oleva varoitus. |

Säätöflagit, kaikki valinnaisia:

| Flagi | Merkitys |
| --- | --- |
| `--integration-time MS` / `-t MS` | Manuaalinen integraatioaika millisekunteina. |
| `--frame-avg N` / `-f N` | Raportoidun spektrin keskiarvoistettujen kehysten määrä. |
| `--no-ae` | Poista automaattinen valotus käytöstä (AE on oletusarvoisesti päällä). |
| `--no-stream` | Yhdistä käynnistämättä lähetystä (jatka myöhemmin komennolla `pool-stream --start`). |
| `--cap-id CAP` | Cap-korjausprofiili; taustapalvelimen oletusarvo on `sunshine_cosine`. Katso [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

{% hint style="warning" %}
**`--eth`:n automaattisen tunnistuksen varoitus.** Moniliitännäisessä isäntäkoneessa (useampi kuin yksi aktiivinen verkkoliitäntä) *ensimmäinen* `pool-connect --eth` käynnistyksen jälkeen voi palauttaa tyhjän tuloksen, vaikka anturi olisi kunnossa — tunnistushaku voi ohittaa anturin liitännän, kun ARP-välimuisti on tyhjä. Jos `--eth` ei löydä mitään, yritä uudelleen tai ohita tunnistus kokonaan käyttämällä komentoa `--eth-host <ip-or-hostname>`, joka on luotettava menetelmä moniverkkoyhteyksillä varustetuissa koneissa. DAQ-E:n isäntänimi on `daq-e-<id>.local` (esim. `daq-e-def330.local`); myös sen pelkkä IP-osoite toimii.
{% endhint %}

## `pool-list` — katso, mitä on kytketty

Näyttää kaikki anturit taustapoolissa, mukaan lukien `sensor_id`, jota kaikki muut komennot tarvitsevat:

| Malli | `sensor_id`-muoto | Esimerkki |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-oktettinen, väliviivalla erotettu | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

## `pool-latest` — lue spektrikehyksiä

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Palauttaa viimeisimmän kehyksen tai viimeisimmät `--recent N`-kehykset; `--json` tuottaa koneellisesti luettavan tulosteen skriptejä varten. Kehykset ovat radiometrisesti kalibroituja spektrisiä säteilyvoimakkuuksia (W/m²/nm) 135 pisteen, 340–1010 nm:n ruudukolla, johon on jo sovellettu anturin peiteprofiilia. Saadaksesi kvantitatiivisia säteilyvoimakkuusarvoja, laske keskiarvo vähintään 15 sekunnin ajalta kerätyistä kehyksistä — tämä on laitteen ominaisuus, ei vika.

## `pool-stream` — keskeytä tai jatka suoratoistoa

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` — tallenna `.daq`-tiedosto

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| Lippu | Oletus | Merkitys |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | Tallennuksen kesto sekunteina; `0` tarkoittaa, että tallennus jatkuu, kunnes annat komennon `--stop`. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Tulostuskansio, määritetään **taustapalvelinta ajavalla koneella**. |
| `--device-name NAME` | — | Tallennuksen mukana tallennettu tunniste. |
| `--stop` | — | Pysäytä käynnissä oleva tallennus. |

{% hint style="info" %}
Tallennus tapahtuu backendissä, joten `.daq`-tiedosto tallentuu **taustapalvelimen** tiedostojärjestelmään — oletuksena `~/Documents/DAQ Live View/`-kansioon, ei välttämättä sinne, missä suoritat CLI-komennon. Tiedostonimissä on mukana anturin tunnus ja aikaleima.
{% endhint %}

## `pool-set-cap` — määritä asennettu suojus

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

Kannen tunniste valitsee jokaiselle spektrille sovellettavan tehtaalla mitatun korjausprofiilin, ja sen **on vastattava anturiin fyysisesti asennettua kantta** — anturi tai ohjelmisto eivät pysty tunnistamaan kantta itsenäisesti, ja valinta merkitään jokaiseen `.daq`-tiedostoon. Oletusarvo kaikkialla on `sunshine_cosine` (jokaisessa DAQ-laitteessa on asennettuna Sunshine-kosinikorjauskorkki, jonka vaimennus on suunniteltu noin 12-kertaiseksi — ilmoittamaton korkin vaihto korjaa spektrejä virheellisesti suunnilleen tällä kertoimella).

| `--cap-id` | Saatavilla |
| --- | --- |
| `sunshine_cosine` (oletus) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | Vain DAQ-U |
| `none` | Vain DAQ-E — katso huomautus |

Anturin asetusten ulkopuolella oleva suojuksen tunnus hylätään yhteyden muodostamisen yhteydessä selkeällä virheilmoituksella. `none` (DAQ-E) tarkoittaa, että suojus on fyysisesti irrotettu — laite käyttää edelleen DAQ-E:n upotetun lasidiffuusorin tehdasgeometrian profiilia, joten kyseessä ei ole toimimaton tila, ja paljas DAQ-E on testipenkki-asetelmassa, ei tuetussa kenttäasetelmassa. (Paljas DAQ-U on täysin paljas eikä tarvitse lainkaan korjausprofiilia; DAQ-M:ää käytetään yhdessä sen Sunshine-suojuksen kanssa.)

## `pool-disconnect` — vapauta anturit

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

***

## Komentojen yhteenveto

| Komento | Tarkoitus |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Avaa anturi taustapoolissa. |
| `daq pool-list` | Näytä kaikki pooliin kuuluvat anturit niiden `sensor_id`-tunnisteineen. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Viimeisimmät N kalibroitua spektrikehystä. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Jatka / keskeytä suoratoisto. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--stop]` | Käynnistä / lopeta `.daq`-tallennus (taustapuolella). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Vaihda cap-korjausprofiilia suorituksen aikana. |
| `daq pool-disconnect --sensor-id ID [--all]` | Vapauta yksi anturi tai kaikki anturit. |

***

## Ensimmäisen DAQ-E-yhteyden vianmääritys

1. DAQ-E:ssä ei ole tilavaloa — tarkista virransyöttö kytkimen tai injektoriportin PoE-/link-merkkivalosta ja odota muutaman sekunnin ajan käynnistyksen jälkeen, kunnes laite käynnistyy ja liittyy verkkoon.
2. Taustakoneen on oltava **samassa lähetysalueessa** kuin anturi — mDNS ei läpäise reitittimiä.
3. Windows-laitteessa hyväksy Defender-palomuurin kehote ensimmäisellä käyttökerralla (mDNS UDP 5353, DAQ-E-data UDP 5002, PTP UDP 319/320).
4. Eikö `--eth` vieläkään vastaa? Käytä `--eth-host`-komentoa laitteen isäntänimellä (`daq-e-<id>.local`) tai IP-osoitteella – tämä on luotettavin reitti, etenkin moniyhteysisissä isäntäkoneissa.

***{% hint style="info" %}**Vinkki tekoälyavustajille.** Tämän käyttöohjeen jokainen sivu toimitetaan raakamuodossa Markdown-muodossa — liitä `.md` sivun pienillä kirjaimilla kirjoitettuun URL-slugiin (tämä sivu: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); koneellisesti luettavissa oleva hakemisto on `https://mapir.gitbook.io/chloros/llms.txt`. Saadaksesi täydellisen lipputason dokumentaation `chloros-cli daq`:stä ja kaikista muista komentojoukoista, hae [CLI-viite](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); Python-polku on `chloros_sdk.connect_daq_sensor()` [SDK-viitteessä](../reference/sdk-reference.md).
{% endhint %}
