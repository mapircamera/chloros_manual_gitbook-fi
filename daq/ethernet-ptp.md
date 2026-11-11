# DAQ-E-verkko ja aikasynkronointi

> Anturin fyysinen verkkoasennus – kaapelointi, PoE, IP-osoitteen määrittäminen ja laitteen omat verkkoasetukset – on kuvattu **[DAQ-käyttöoppaassa](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Tällä sivulla käsitellään Chloros-puolta: liitäntää, aikasynkronointia ja toimia, jos laitteen tunnistus ei tuota tulosta.

DAQ-E on DAQ-tuoteperheen Ethernet-malli: se saa virran PoE:n kautta, se löydetään mDNS:n avulla (palvelu `_daq-e._tcp`) ja se on osoitettavissa anturin tunnisteesta johdetulla isäntänimellä — `daq-e-<6 hex>.local`, esim. `daq-e-def330.local`. Tällä sivulla käsitellään, miten laite siirtää dataa verkossa ja miten se osallistuu PTP-aikasynkronointiin.

## Siirtotilat

| Tila | Päätelaite | Käyttäjät | Huomautukset |
| --- | --- | --- | --- |
| **Multicast** (oletus) | UDP `239.10.10.10:5002` | Mikä tahansa laite samassa lähiverkossa vastaanottaa saman datavirran | Jokainen datagrammi on CRC-16/CCITT-validoitu |
| **Raw** | TCP-portti `5000` | Täsmälleen yksi asiakas (yksinomainen) | Täysin yhteensopiva DAQ-U:n kanssa tavutason |

Chloros käyttää oletuksena multicastia, minkä ansiosta GUI, CLI ja SDK voivat kaikki seurata yhtä anturia samanaikaisesti.

## Verkkoedellytykset

* **Sama lähetysalue.** Chloros-ohjelmistoa ajavan koneen on oltava samassa L2-verkko-osassa kuin anturi — mDNS-hakutoiminto ei läpäise reitittimiä.
* **Palomuuri.** Windows issa asennusohjelma lisää jokaiseen verkkoprofiiliin **Chloros Backend (UDP-In)**-palomuurisäännön, joka kattaa DAQ-E-tiedot (UDP 5002), mDNS:n (UDP 5353) ja PTP:n (UDP 319/320), sekä**Chloros Backend (TCP-In)** -säännön, joka on rajoitettu paikalliseen aliverkkoon ja koskee DAQ-E:n yhteyttä tietokoneeseen laiteohjelmiston päivityksen aikana. Linux issa `.deb` avaa UDP-portit `ufw`:ssä tai `firewalld`:ssä, kun kyseinen palomuuri on aktiivinen asennuksen aikana. Kolmannen osapuolen palomuurissa tai myöhemmin käyttöönotetussa palomuurissa samat portit on sallittava.
* **PoE-virta, ei tilan LED-merkkivaloa.** DAQ-E:llä ei ole omaa LED-merkkivaloa — tarkista virransyöttö kytkimen tai injektoriportin linkki-/PoE-merkkivalosta ja odota käynnistyksen jälkeen muutama sekunti, jotta laite ehtii käynnistyä ja liittyä verkkoon.

## Yhdistäminen

**GUI:** Valosensorit-välilehti → Yhdistä anturi → Laitetyyppi "DAQ-E (Ethernet)". Laitteiden etsintä tapahtuu vain niin kauan kuin yhdistämisvalintaikkuna on näytöllä (mDNS-haku sekä ARP-pyyhkäisy Windows issa), ja se toistuu 15 sekunnin välein; Päivitä-painike käynnistää uuden haun välittömästi. Löydetyt anturit näkyvät pudotusvalikossa; ensimmäinen havaittu anturi valitaan automaattisesti.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (taustapalvelu käynnissä):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

Poolissa jo oleva DAQ-E tunnistetaan sekä sen isäntänimestä että IP-osoitteesta: sen uudelleenliittäminen toisella osoitteella palauttaa olemassa olevan poolimerkinnän sen sijaan, että laite avattaisiin toisen kerran.

### Usean verkkokortin isäntäkoneet ja ensimmäinen yhteyden muodostus käynnistyksen jälkeen

Isäntäkoneissa, joissa on useampi kuin yksi aktiivinen verkkoliitäntä, **ensimmäinen** `pool-connect --eth` käynnistyksen jälkeen voi jäädä tyhjäksi, vaikka anturi olisi kunnossa — tunnistushaku voi ohittaa sen liitännän, jossa anturi sijaitsee, kun ARP-välimuisti on vielä kylmä. Luotettava ratkaisu on ohittaa etsintä ja antaa osoite suoraan:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` hyväksyy mDNS-isäntänimen tai IP-osoitteen, kohdistuu aina oikeaan anturiin ja on suositeltava muoto skripteille ja päättömille asennuksille. Käytä graafisessa käyttöliittymässä yhteysvalintaikkunan Päivitä-painiketta ja anna skannauksen toistua.

## Laiteasetukset ja laiteohjelmisto

Anturi itse säilyttää verkkoasetuksensa: DHCP (tehdasasetus, joka siirtyy 169.254.x.x-linkkipaikalliseen osoitteeseen, jos mikään DHCP-palvelin ei vastaa) tai staattinen osoite. Se säilyttää myös tiedon siitä, aloittaako se datan lähettämisen virran kytkemisen yhteydessä. Muuta asetuksia graafisessa käyttöliittymässä (anturin hammasratasasetukset, **Verkko**:**Muuta…**,**Lähetä virran kytkemisen yhteydessä**,**Käynnistä anturi uudelleen**; katso [DAQ-välilehti](gui.md)), tai CLI:n ja SDK:n kautta komentoilla `pool-set-network`, `pool-set-auto-stream` ja `pool-reboot` (`DAQSensorSession.set_network()`, `set_auto_stream()`, `reboot()`; katso [CLI-ohje](../reference/cli-reference.md#daq-e-device-settings)). Anturi käynnistyy uudelleen verkkomuutoksen voimaansaattamiseksi, ja Chloros muodostaa siihen uuden yhteyden. Verkossa on oltava staattinen osoite, joka kuuluu Chloros-ohjelmistoa käyttävän tietokoneen verkkoon: Chloros hylkää osoitteen, johon se ei pääse käsiksi uudelleenkäynnistyksen jälkeen. Anturin tunnus ja sen `daq-e-<id>.local`-isäntänimi eivät koskaan muutu. Uutta versiossa 1.3.0.

**Laiteohjelmistopäivitykset on integroitu käyttöliittymään.** Kun kytketty DAQ-E käyttää vanhempaa laiteohjelmistoversiota kuin Chloros-asennuksesi mukana toimitettu kuva, sen anturirivillä näkyy keltainen **Päivitys saatavilla** -kuvake, ja asetusvalikossa on ”Päivitä versioon <version>” -painike. Päivitys siirtyy verkon kautta noin 30 sekunnissa; anturi käynnistyy uudelleen ja muodostaa yhteyden automaattisesti, ja keskeytynyt siirto jättää nykyisen laiteohjelmiston ennallaan. CLI:n tai SDK:n kautta sekä Linux-ympäristössä, jossa ei ole käyttöliittymää, suorita komento `chloros-cli daq pool-update-firmware --sensor-id <id>` (`DAQSensorSession.update_firmware()`).

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## PTP-aikasynkronointi

DAQ-E-laiteohjelmisto v1.2.0+ toimii IEEE 1588 PTPv2 -protokollassa tavallisena (vain orjana toimivana) kellona. **Chloros-isäntäkoneen taustapalvelu toimii PTP-grandmasterina** — jokainen DAQ-E-laite ja jokainen LATTICE-kamera lähiverkossa toimii sen orjana domainissa 0, jolloin kaikkien laitteiden aikaleimat pysyvät noin 1 ms:n toleranssissa. Juuri tämä jaettu kello mahdollistaa DAQ-lukemien aikaleimojen täsmäämisen kameran valotusaikojen kanssa (katso [Tallennus ja .daq-muoto](recording.md)).

Tarkista synkronointi CLI:n kautta:

| Komento | Näyttää |
| --- | --- |
| `chloros-cli time-sync status` | Isäntäkellon tila, BMCA-prioriteetit, kellotunniste sekä `health`: `OK` tai `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT`, jossa `health_detail` ilmoittaa, miksi sen paketit eivät saavuta verkkoa |
| `chloros-cli time-sync peers` | Kaikki havaitut orjalaitteet (DAQ-E-anturit + LATTICE-kamerat) |
| `chloros-cli time-sync cameras` | Kamerakohtainen PTP-toimintatila (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Käynnistä grandmaster-prosessi uudelleen |

Käyttöliittymässä DAQ-E-asetusten modaalissa näkyy reaaliaikainen **PTP Sync** -rivi, jossa näkyy anturin nykyinen PTP-tila.**DAQ-E, jonka PTP on kytketty pois päältä, pysyy pois päältä.** Laite tallentaa PTP:n päällä/pois-asetuksensa omaan muistiinsa, joten laite, jonka PTP on kytketty pois päältä, pysyy pois päältä uudelleenkytkennän tai virran katkaisun ja uudelleenkytkennän jälkeen, eikä sen aikaleimoja synkronoida PTP:n avulla. `chloros-cli daq pool-list` merkitsee tällaisen laitteen (`!! PTP is switched OFF on this unit …`) ja tulostaa komennon, jolla se korjataan:

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Chloros ei koskaan kytke laitetta takaisin päälle itsestään, koska pois päältä -tila voi olla tarkoituksellinen.

Yksityiskohdat tiukan synkronoinnin vaativille käyttäjille:

* Jokaisessa lähetetyssä datagrammissa on lippukenttä; **bitti 2 on asetettu kehyksissä, joiden aikaleima on PTP-synkronoitu**. Putkistot, jotka vaativat tiukkaa kamera-/DAQ-synkronointia, tulisi rajata kyseisen bitin perusteella.
* Ennen synkronoitua tallennusta varmista, että anturi näkyy `chloros-cli time-sync peers`:ssä ja että `chloros-cli daq pool-list` ei merkitse sen PTP:tä pois päältä olevaksi. (MAPIRin sisäiset suorat laitteistotyökalut voivat myös rajoittaa tallennusta PTP-lukituksen perusteella `--wait-ptp`-lipulla, joka odottaa jopa 15 sekuntia, kunnes anturi saavuttaa SLAVE-tilan; kyseiset työkalut eivät kuulu toimitettuun CLI-käyttöliittymään.)
* Kun PTP toimii aktiivisesti orjana, anturi hylkää manuaaliset kellosyötöt ("PTP toimittaa kellotaajuuden"). Tämä on tarkoituksellista — luota PTP:hen.

## Linux-huomautukset

* **PTP-portit ilman root-oikeuksia.** PTP käyttää UDP-portteja 319/320. `.deb` kirjoittaa `/etc/sysctl.d/60-chloros-ptp.conf` (`net.ipv4.ip_unprivileged_port_start = 319`), joten taustapalvelu sitoo ne käyttäjän nimissä. Versiosta 1.3.0 lähtien paketti ei aseta tiedosto-oikeutta (`setcap`) taustapalvelimelle eikä tarvitse `libcap2-bin`:ää; älä lisää oikeutta itse, koska purkamaton taustapalvelin ei silloin käynnisty. Jos PTP ei pysty sitomaan porttia 319, katso [Linux-asennus](../linux/linux-installation.md#ptp-will-not-start--cannot-bind-port-319).

* **Headless Jetson / Linux-isäntäkone:** paketti asentaa taustapalvelut, mutta ei ota yhtään niistä käyttöön. Jotta PTP (ja DAQ:n saatavuus) olisi aina päällä ilman, että kukaan suorittaa komentoa, ota palvelu käyttöön omalla käyttäjätunnuksellasi:

  ```bash
  sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
  ```

  Ilman tätä PTP toimii vain, kun jokin on käynnistänyt taustapalvelun — joko `chloros-cli`-komento tai SDK-skripti. Katso [Linux-asennus](../linux/linux-installation.md#always-on-ptp-for-headless-hosts) saadaksesi tietoja root-käyttäjästä `chloros-backend.service` ja siitä, miten siirtyä pois siitä.

## Vianmääritys: "No DAQ-E devices found"

| Tarkistus | Yksityiskohdat |
| --- | --- |
| Virta | Anturissa ei pala LED-valoa — tarkista kytkimen/injektorin portin PoE- ja linkkivalot; odota muutama sekunti virran kytkemisen jälkeen |
| Lähetysalue | Isäntä ja anturi samassa L2-segmentissä; mDNS ei reititä |
| Palomuuri | Windows: asennusohjelman **Chloros Backend (UDP-In)** -sääntö sallii DAQ-E-virran (UDP 5002), mDNS:n (5353) ja PTP:n (319/320) pääsyn; kolmannen osapuolen palomuurissa tarvitaan sama asetus. Linux: `.deb` avaa nämä portit `ufw` / `firewalld`:ssä vain, jos palomuuri oli aktiivinen asennuksen aikana — salli ne itse, jos palomuuri otetaan käyttöön myöhemmin |
| Usean verkkokortin isäntäkone | Löytötoiminto etsii mDNS-verkosta ARP-varajärjestelmän avulla; `--eth-host <ip-or-hostname>` ohittaa löytötoiminnon ja muodostaa yhteyden suoraan |
| Uudelleenskannaus käyttöliittymässä | Löytötoiminto toimii vain, kun yhteysvalintaikkuna on auki; käytä Päivitä-painiketta |</version>
