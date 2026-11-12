# CLI:n pikaopas (pool-*)

Toimitettu `chloros-cli` ohjaa DAQ-antureita **`daq pool-*`**-komentoperheen avulla — nämä ovat kevyitä HTTP-asiakasohjelmia, jotka ohjaavat anturia Chloros-taustapalvelimen pysyvän anturipoolin kautta. Taustapalvelu hallitsee siirtoa, joten graafinen käyttöliittymä (GUI), komentoriviliittymä (CLI) ja SDK-skriptit jakavat kaikki yhden aktiivisen käsittelytunnuksen sen sijaan, että ne kilpailevat portista. Kaikki, mitä asiakas tarvitsee, on käytettävissä `pool-*`:n kautta: yhteyden muodostaminen, datavirran lähetys, kalibroitujen `.daq`-tiedostojen tallennus ja cap-profiilien vaihtaminen.

`pool-*` on tuettu DAQ-käyttöliittymä julkaistuissa versioissa. `chloros-cli daq --help` listaa `pool-*`-alikomennot, ja `daq discover` / `daq list` suorittavat saman skannauksen kuin `pool-discover`. Jos suoraa laitteistokohtaista DAQ-alikomentoa ei ole käytettävissä, ohjelma lopetetaan nimenomaisella virheilmoituksella, jossa mainitaan puuttuva paketti ja ohjataan käyttäjä takaisin komentoon `pool-*` — mikään ei epäonnistu hiljaisesti. (Myöskään `pip install chloros-sdk` ei tarjoa suoraan laitteistoon liittyviä komentoja.)

***

## Edellytykset

* **Chloros-taustapalvelun on oltava käynnissä** — `pool-*`-komennot ovat HTTP-asiakasohjelmia, eivät laiteajureita. Käynnistä Windowsissa Chloros-työpöytäsovellus (se käynnistää taustapalvelun). Headless-Linux/Jetson-ympäristössä ota palvelu käyttöön omalla käyttäjätunnuksellasi: `sudo systemctl enable --now chloros-backend@$USER.service` (katso [Linux-asennus](../linux/linux-installation.md#always-on-ptp-for-headless-hosts)).
* **Chloros+ (maksullinen taso) -kirjautuminen**: suorita ensin komento `chloros-cli login`. Valvonta tapahtuu palvelinpuolella — ilman kirjautumista komennot epäonnistuvat `401 AUTH_REQUIRED`:n kanssa; ilmaisella (Iron) tasolla ne epäonnistuvat `403 PLAN_UPGRADE_REQUIRED`:n kanssa.
* Komennot kohdistuvat oletuksena `http://127.0.0.1:5000`:ään; `daq pool-*`-perhe noudattaa `CHLOROS_BACKEND_URL`-ympäristömuuttujaa, jos taustapalvelimesi toimii muualla.

***

## Viiden minuutin istunto

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)
#    (a DAQ-M's BLE MAC is not printed anywhere: find it with `chloros-cli daq pool-discover --only ble`)

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

| Variantti | Merkitys |
| --- | --- |
| `daq pool-connect` | Älykäs tunnistus: etsi kaikki tämän koneen DAQ-laitteet. |
| `daq pool-connect --port PORT` | DAQ-U tietyllä sarjaportilla (esim. `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | DAQ-M BLE-yhteyden kautta, MAC-osoite skannattu automaattisesti. |
| `daq pool-connect --mac MAC` | DAQ-M tunnetulla BLE-MAC-osoitteella (tarkoittaa `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E tunnetulla isäntänimellä tai IP-osoitteella — **luotettava reitti**. |
| `daq pool-connect --eth` | DAQ-E automaattisella tunnistuksella (mDNS, ARP-varajärjestelmällä). Katso alla oleva varoitus. |

Säätöflagit, kaikki valinnaisia:

| Flagi | Merkitys |
| --- | --- |
| `--integration-time MS` / `-t MS` | Manuaalinen integraatioaika millisekunteina. |
| `--frame-avg N` / `-f N` | Raportoidun spektrin keskiarvona käytettävät kehykset. |
| `--no-ae` | Poista automaattinen valotus käytöstä (AE on oletusarvoisesti päällä). |
| `--no-stream` | Yhdistä käynnistämättä lähetystä (jatka myöhemmin komennolla `pool-stream --start`). |
| `--cap-id CAP` | Cap-korjausprofiili; taustapalvelimen oletusarvo on `sunshine_cosine`. Katso [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

Jos yhdistät DAQ-E-laitteen, joka on jo poolissa toisen osoitteensa alla — sen `daq-e-<id>.local`-isäntänimen tai IP-osoitteen kautta —, järjestelmä palauttaa olemassa olevan poolimerkinnän sen sijaan, että avaisi laitteen toisen kerran.

{% hint style="warning" %}
**`--eth`:n automaattisen tunnistuksen varoitus.** Moniverkkoyhteydellisessä isäntäkoneessa (useampi kuin yksi aktiivinen verkkoliitäntä) *ensimmäinen* `pool-connect --eth`-kysely käynnistyksen jälkeen voi palauttaa tyhjän tuloksen, vaikka anturi olisi kunnossa — tunnistushaku voi ohittaa anturin liitännän, kun ARP-välimuisti on tyhjä. Jos `--eth` ei löydä mitään, yritä uudelleen tai ohita tunnistus kokonaan käyttämällä komentoa `--eth-host <ip-or-hostname>`, joka on luotettava menetelmä moniverkkoyhteyksillä varustetuissa koneissa. DAQ-E:n isäntänimi on `daq-e-<id>.local` (esim. `daq-e-def330.local`); myös sen pelkkä IP-osoite toimii.
{% endhint %}

## `pool-discover` — skannaa muodostamatta yhteyttä

```bash
chloros-cli daq pool-discover                  # USB, BLE and Ethernet
chloros-cli daq pool-discover --only ble       # BLE only: this is how you find a DAQ-M's MAC
chloros-cli daq pool-discover --json           # machine-readable
```

Etsii antureita, joihin voisit muodostaa yhteyden, ja näyttää yhteyden muodostamiseen tarvittavan osoitteen ilman, että yhteyttä muodostetaan. `--only` ottaa vastaan pilkuilla erotetun osajoukon `usb,ble,eth`:sta; `--timeout SEC` asettaa kuljetuskohtaisen skannausajan (oletus 5 s, rajoitettu välille 1–20 s). Poolissa jo avoimina olevia antureita ei luetella (kytketty DAQ-M lopettaa ilmoittamisen) — käytä niihin komentoa `pool-list`. Toimitetussa versiossa komennot `chloros-cli daq discover` ja `daq list` suorittavat tämän saman skannauksen.

## `pool-list` — näytä, mitä on kytketty

Näyttää kaikki backend-poolissa olevat anturit, mukaan lukien `sensor_id`, jota kaikki muut komennot tarvitsevat:

| Malli | `sensor_id`-muoto | Esimerkki |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-oktettinen, väliviivalla erotettu | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

DAQ-E, jonka PTP on kytketty pois päältä (asetus tallennetaan laitteeseen), merkitään `pool-list`:ssä tunnisteella `!! PTP is switched OFF on this unit (saved in its flash): its timestamps are not PTP-synced.` sekä komennolla, joka kytkee sen takaisin päälle — katso [`pool-enable-ptp`](#pool-enable-ptp-switch-a-daq-es-ptp-back-on).

## `pool-latest` — lue spektrikehykset

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Palauttaa viimeisimmän kehyksen tai viimeisimmät `--recent N`-kehykset; `--json` tuottaa koneellisesti luettavaa tulostetta skriptejä varten. Kehykset ovat radiometrisesti kalibroituja spektrisiä säteilyvoimakkuuksia (W/m²/nm) 135 pisteen, 340–1010 nm:n ruudukolla, johon on jo sovellettu anturin peiteprofiilia. Saadaksesi kvantitatiivisia säteilyvoimakkuusarvoja, laske keskiarvo vähintään 15 sekunnin ajalta kerätyistä kehyksistä — tämä on laitteen ominaisuus, ei vika.

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
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Tulostuskansio. Kun taustapalvelin sijaitsee tällä tietokoneella, suhteellinen polku määritetään sen kansion suhteen, jossa komento suoritettiin; etätaustapalvelimen tapauksessa polku määritetään **taustapalvelinta ajavalla koneella**. |
| `--device-name NAME` | — | Tallennuksen mukana tallennettu tunniste. |
| `--stop` | — | Keskeytä käynnissä oleva tallennus. |
| `--confirm-after SEC` | `20` | Käynnistyksen jälkeen odota enintään tämän ajan, että ensimmäinen rivi kirjoitetaan, ja anna äänekäs varoitus (`!! WARNING: no rows written after …s. This recording is producing an EMPTY file.`), jos riviä ei tule. `0` poistaa tarkistuksen käytöstä. |
| `--heartbeat SEC` | `30` | Komennolla `--duration` tulostetaan rivien lukumäärä joka SEC sekunti ja annetaan varoitus, jos lukumäärä lakkaa kasvamasta. `0` poistaa sen käytöstä. |

{% hint style="info" %}
Tallennus tapahtuu taustapalvelimella, joten `.daq`-tiedosto tallentuu **taustapalvelimen** tiedostojärjestelmään — oletuksena `~/Documents/DAQ Live View/` sinne, ei välttämättä sinne, missä suoritat CLI-komentoa. Tiedostonimissä on mukana anturin tunnus ja aikaleima.
{% endhint %}

## `pool-set-cap` — ilmoita asennettu suojus

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

Kannen tunnus valitsee tehtaalla mitatun korjausprofiilin, jota sovelletaan jokaiseen spektriin, ja sen **on vastattava anturiin fyysisesti asennettua kantta** — anturi tai ohjelmisto eivät pysty tunnistamaan kantta itsenäisesti, ja valinta merkitään jokaiseen `.daq`-tiedostoon. Oletusarvo kaikkialla on `sunshine_cosine` (jokaisessa DAQ-laitteessa toimitetaan asennettuna Sunshine-kosinikorjauskorkki, jonka vaimennus on suunniteltu noin 12-kertaiseksi — ilmoittamaton korkin vaihto korjaa spektrejä virheellisesti suunnilleen tällä kertoimella).

| `--cap-id` | Saatavana |
| --- | --- |
| `sunshine_cosine` (oletus) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | Vain DAQ-U |
| `none` | Vain DAQ-E — katso huomautus |

Anturin asetusten ulkopuolella oleva suojuksen tunniste hylätään yhteyden muodostamisen yhteydessä selkeällä virheilmoituksella. `none` (DAQ-E) tarkoittaa, että suojus on fyysisesti irrotettu — se soveltaa edelleen DAQ-E:n upotetun lasidiffuusorin tehdasgeometrian profiilia, joten se ei ole toimimaton, ja paljas DAQ-E on testipenkki-asetelmassa, ei tuettu kenttäasetelmassa. (Paljas DAQ-U on täysin paljas eikä tarvitse lainkaan korjausprofiilia; DAQ-M:ää käytetään sen Sunshine-kannen kanssa.)

## `pool-disconnect` — anturien vapauttaminen

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

`pool-disconnect` vapauttaa anturin riippumatta siitä, kuka sen avasi, joten se vapauttaa myös anturin, jonka kaatunut skripti on jättänyt pooliin (anturin tallennus saatetaan ensin loppuun). Anturia, jota ei ole poolissa, ei ole vapautettavaa: komento tulostaa `<sensor-id>: already released (not in the pool)` ja lopettaa `0`. Vapautus, jonka taustapalvelu hylkäsi tai jota se ei vahvistanut, lopettaa `1`.

## `pool-enable-ptp` — kytke DAQ-E:n PTP takaisin päälle

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

DAQ-E säilyttää PTP:n päällä/pois-asetuksen omassa muistissaan, joten laite, jonka PTP on kytketty pois päältä, pysyy pois päältä — eikä sen aikaleimoja synkronoida PTP:n kanssa — kunnes se kytketään takaisin päälle. `pool-list` merkitsee tällaisen laitteen ja tulostaa tämän komennon sille. Chloros ei koskaan kytke laitetta takaisin päälle itsestään, koska pois päältä -tila voi olla tarkoituksellinen. Uusi ominaisuus versiossa 1.3.0.

## DAQ-E-laitteen asetukset

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto
chloros-cli daq pool-set-auto-stream --sensor-id daq-e-def330 --on
chloros-cli daq pool-reboot --sensor-id daq-e-def330
chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

`pool-info` näyttää anturin laiteohjelmiston (ja sen, sisältääkö tämä Chloros-versio uudempaa), sen osoitteen ja miten se on saatu, tallennetut verkkoasetukset, jos ne eroavat käytössä olevista, virran kytkemisen yhteydessä tapahtuvan datan lähetyksen sekä PTP:n. `pool-set-network` käynnistää anturin uudelleen muutoksen voimaansaattamiseksi ja yhdistää sen uudelleen aiemmilla asetuksilla; `--no-reboot` vain tallentaa asetukset. Staattisen osoitteen on oltava tämän tietokoneen verkossa, muuten Chloros hylkää sen; `--force` ohittaa tämän rajoituksen, mutta vain sellaisen tietokoneen osalta, jolle osoite määritetään uudelleen tai joka reititetään siihen. Uudelleenkäynnistykset hylätään, kun anturi tallentaa tietoja. `pool-update-firmware` vastaa käyttöliittymän **Päivitä kohtaan \<version\>** -toimintoa komentorivillä ja on tapa päivittää DAQ-E-laite Linuxissa, jossa ei ole käyttöliittymää. Uusi ominaisuus versiossa 1.3.0.

***

## Komentojen yhteenveto

| Komento | Tarkoitus |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Avaa anturi taustapoolissa. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Etsii antureita muodostamatta yhteyttä (löytää DAQ-M:n BLE-MAC-osoitteen). |
| `daq pool-list` | Näyttää kaikki poolissa olevat anturit niiden `sensor_id`-tunnisteineen. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Viimeisimmät N kalibroitua spektrikehystä. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Jatka / keskeytä suoratoisto. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--confirm-after SEC] [--heartbeat SEC] [--stop]` | Käynnistä / lopeta `.daq`-tallennus (taustapuolella). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Vaihda cap-korjausprofiilia suorituksen aikana. |
| `daq pool-disconnect --sensor-id ID [--all]` | Vapauta yksi anturi tai kaikki anturit. |
| `daq pool-enable-ptp --sensor-id ID` | Kytke PTP takaisin päälle DAQ-E-laitteelle, jonka PTP on pois päältä. |
| `daq pool-info --sensor-id ID [--json]` | DAQ-E-laitteen laiteohjelmisto, osoite, tallennetut verkkoasetukset, käynnistyshetkellä alkava suoratoisto ja PTP. |
| `daq pool-set-network --sensor-id ID (--auto \| --static IP [--mask M] [--gateway G]) [--no-reboot] [--force]` | DHCP tai staattinen osoite; käynnistää anturin uudelleen ja muodostaa yhteyden uudelleen. |
| `daq pool-set-auto-stream --sensor-id ID (--on \| --off)` | Lähetys virran kytkemisen yhteydessä. |
| `daq pool-reboot --sensor-id ID` | Käynnistä DAQ-E uudelleen ja muodosta yhteys uudelleen. |
| `daq pool-update-firmware --sensor-id ID` | Asenna tämän version DAQ-E-laiteohjelmisto vanhempaan anturiin. |

***

## Ensimmäisen DAQ-E-yhteyden vianmääritys

1. DAQ-E:ssä ei ole tilavaloa — varmista virransyöttö kytkimen tai injektoriportin PoE/link-merkkivalosta ja odota käynnistyksen jälkeen muutama sekunti, jotta laite ehtii käynnistyä ja liittyä verkkoon.
2. Taustakoneen on oltava **samassa lähetysalueessa** kuin anturi — mDNS ei läpäise reitittimiä.
3. Palomuurin on sallittava mDNS (UDP 5353), DAQ-E-tiedot (UDP 5002) ja PTP (UDP 319/320). Windows-asennusohjelma lisää tämän varten säännön, ja Linux-versiossa `.deb` avaa ne `ufw`- tai `firewalld`-palomuurissa, joka oli aktiivinen asennuksen aikana; kolmannen osapuolen palomuurissa tai myöhemmin käyttöönotetussa palomuurissa on sallittava nämä yhteydet.
4. Eikö `--eth` vieläkään toimi? Käytä `--eth-host`-komentoa laitteen isäntänimellä (`daq-e-<id>.local`) tai IP-osoitteella, jolloin tunnistus ohitetaan.

***
{% hint style="info" %}
**Vinkki tekoälyavustajille.** Tämän käyttöohjeen jokainen sivu toimitetaan raakamuodossa Markdown-muodossa — liitä `.md` sivun pienillä kirjaimilla kirjoitettuun URL-slugiin (tämä sivu: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); koneellisesti luettavissa oleva hakemisto on `https://mapir.gitbook.io/chloros/llms.txt`. Täydelliset lipputason ohjeet `chloros-cli daq`:stä ja kaikista muista komentojoukoista löydät [CLI-viitteestä](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); Python-polku on `chloros_sdk.connect_daq_sensor()` [SDK-viitteessä](../reference/sdk-reference.md).
{% endhint %}
