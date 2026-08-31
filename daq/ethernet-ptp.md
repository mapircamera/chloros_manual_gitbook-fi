# DAQ-E-verkko ja aikasynkronointi

> Anturin fyysinen verkkoasennus – kaapelointi, PoE, IP-osoitteen määrittäminen ja laitteen omat verkkoasetukset – on kuvattu **[DAQ-käyttöoppaassa](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Tällä sivulla käsitellään Chloros-puolta: liitäntä, ajan synkronointi ja mitä tehdä, jos laitteen tunnistus ei tuota tulosta.

DAQ-E on DAQ-tuoteperheen Ethernet-malli: se saa virran PoE:n kautta, se löydetään mDNS:n kautta (palvelu `_daq-e._tcp`) ja se on osoitettavissa anturin tunnisteesta johdetulla isäntänimellä — `daq-e-<6 hex>.local`, esim. `daq-e-def330.local`. Tällä sivulla käsitellään, miten laite siirtää dataa verkossa ja miten se osallistuu PTP-aikasynkronointiin.

## Siirtotilat

| Tila | Päätelaite | Käyttäjät | Huomautukset |
| --- | --- | --- | --- |
| **Multicast** (oletus) | UDP `239.10.10.10:5002` | Mikä tahansa laite samassa lähiverkossa vastaanottaa saman datavirran | Jokainen datagrammi on CRC-16/CCITT-validoitu |
| **Raw** | TCP-portti `5000` | Täsmälleen yksi asiakas (yksinomainen) | Täysin yhteensopiva DAQ-U:n kanssa tavutason |

Chloros käyttää oletuksena multicastia, minkä ansiosta GUI, CLI ja SDK voivat kaikki seurata yhtä anturia samanaikaisesti.

## Verkkoedellytykset

* **Sama lähetysalue.** Chloros:ää ajavan koneen on oltava samassa L2-verkkosegmentissä kuin anturi — mDNS-hakutoiminto ei läpäise reitittimiä.
* **Windows-palomuurikysely: hyväksy se.** Kun Chloros sitoo multicast-socketit ensimmäisen kerran, Windows Defender kysyy lupaa kerran. Sen salliminen kattaa DAQ-E-tiedot (UDP 5002), mDNS:n (UDP 5353) ja PTP:n (UDP 319/320). Linux:ssä tämä tapahtuu hiljaisesti.
* **PoE-virta, ei tilan LED-valoa.** DAQ-E:llä ei ole omaa LED-valoa — tarkista virta kytkimen tai injektoriportin linkki-/PoE-merkkivalosta ja odota käynnistyksen jälkeen muutama sekunti, jotta laite ehtii käynnistyä ja liittyä verkkoon.

## Yhdistäminen

**GUI:** Light Sensors -välilehti → Connect Sensor → Device Type &quot;DAQ-E (Ethernet)&quot;. Haku suoritetaan vain niin kauan kuin yhteysvalintaikkuna on näytöllä (mDNS-haku sekä ARP-pyyhkäisy osoitteessa Windows), ja se toistuu 15 sekunnin välein; Päivitä-painike käynnistää haun välittömästi uudelleen. Löydetyt anturit näkyvät pudotusvalikossa; ensimmäinen havaittu anturi valitaan automaattisesti.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (taustapalvelu käynnissä):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

### Usean verkkokortin isäntäkoneet ja ensimmäinen yhteyden muodostus käynnistyksen jälkeen

Isäntäkoneissa, joissa on useampi kuin yksi aktiivinen verkkoliitäntä, **ensimmäinen** `pool-connect --eth` käynnistyksen jälkeen voi jäädä tyhjäksi, vaikka anturi olisi kunnossa — tunnistushaku voi ohittaa sen liitännän, jossa anturi sijaitsee, kun ARP-välimuisti on vielä kylmä. Luotettava ratkaisu on ohittaa etsintä ja syöttää osoite suoraan:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` hyväksyy mDNS-isäntänimen tai IP-osoitteen, kohdistuu aina oikeaan anturiin ja on suositeltava muoto skripteille ja päättömille asennuksille. Käytä graafisessa käyttöliittymässä yhteysvalintaikkunan Päivitä-painiketta ja anna skannauksen suorittaa uusi kierros.

## Laiteasetukset ja laiteohjelmisto

Anturissa itsessään on verkkoasetukset – staattinen IP vs. DHCP + link-local-osoitteistus, laitteen nimi, automaattinen suoratoisto käynnistyksen yhteydessä, OTA-salasana. Näitä laitteen omia asetuksia ei ole tuotu esiin komentoina toimitetussa CLI-versiossa; niitä hallitaan Chloros-käyttöliittymän kautta, jos ne näkyvät siellä, tai MAPIR-tuen avulla.

**Laiteohjelmistopäivitykset on integroitu graafiseen käyttöliittymään.**Kun kytketty DAQ-E käyttää laiteohjelmistoa, joka on vanhempi kuin Chloros-versiossa mukana toimitettu kuva, sen anturirivillä näkyy keltainen**Päivitys saatavilla** -kuvake, ja hammasratasasetusten ikkunassa on ”Päivitä versioon<version>

” -painike. Päivitys siirretään verkon kautta noin 30 sekunnissa; anturi käynnistyy uudelleen ja muodostaa yhteyden automaattisesti, ja keskeytynyt siirto jättää nykyisen laiteohjelmiston ennalleen.

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## PTP-aikasynkronointi

DAQ-E-laiteohjelmistoversio v1.2.0+ toimii IEEE 1588 PTPv2 -protokollassa tavallisena (vain orjana toimivana) kellona. **Chloros-isäntäkoneen taustalla toimii PTP-grandmaster** — jokainen DAQ-E ja jokainen LATTICE-kamera lähiverkossa toimii sen orjana domainissa 0, pitäen kaikkien laitteiden aikaleimat noin 1 ms:n toleranssin sisällä. Juuri tämä jaettu kello mahdollistaa DAQ-lukemien aikaleimojen täsmäämisen kameran valotusaikojen kanssa (katso [Tallennus ja .daq-tiedostomuoto](recording.md)).

Tarkista synkronointi tiedostosta CLI:

| Komento | Näyttää |
| --- | --- |
| `chloros-cli time-sync status` | Isäntäkellon tila, BMCA-prioriteetit, kellotunniste |
| `chloros-cli time-sync peers` | Kaikki havaitut orjalaitteet (DAQ-E-anturit + LATTICE-kamerat) |
| `chloros-cli time-sync cameras` | Kamerakohtainen PTP-kunto (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Grandmaster-prosessin uudelleenkäynnistys |

GUI:ssa DAQ-E-asetusten modaalissa näkyy reaaliaikainen **PTP Sync** -rivi, joka sisältää anturin nykyisen PTP-tilan.

Tietoja tiukan synkronoinnin vaativille kuluttajille:

* Jokaisessa lähetetyssä datagrammissa on lippukenttä; **bitti 2 on asetettu kehyksissä, joiden aikaleima on PTP-synkronoitu**. Putkistot, jotka vaativat tiukkaa kamera-/DAQ-synkronointia, tulisi rajata kyseisen bitin perusteella.
* Varmista ennen synkronoitua tallennusta, että anturi näkyy luettelossa `chloros-cli time-sync peers`. (MAPIR:n sisäiset suorat laitteistotyökalut voivat myös rajoittaa tallennusta PTP-lukituksen perusteella `--wait-ptp`-lipulla, joka odottaa jopa 15 sekuntia, kunnes anturi saavuttaa SLAVE-tilan; kyseinen työkalu ei kuulu toimitettuun CLI-versioon.)
* Kun PTP toimii aktiivisesti orjana, anturi hylkää manuaaliset kellosignaalien lähetykset (&quot;PTP toimittaa kellosignaalin&quot;). Tämä on tarkoituksellista — luota PTP:hen.

## Linux-huomautukset

* **PTP tarvitsee `libcap2-bin`:n asennuksen yhteydessä.** `.deb`-postinst-komento myöntää `cap_net_bind_service=+ep`-oikeudet `/usr/lib/chloros/chloros-backend`:lle, jotta se voi sitoa PTP-portit 319/320 ilman root-oikeuksia. Jos `libcap2-bin` puuttuu, kyseinen vaihe ohitetaan ja PTP:n käynnistys epäonnistuu. Korjaus:

  ```bash
  sudo apt install libcap2-bin
  sudo apt reinstall chloros
  ```

* **Päätön Jetson / Raspberry Pi:** Ensimmäisen asennuksen yhteydessä systemd-yksikkö `chloros-backend.service` luodaan, mutta sitä ei oteta käyttöön. Jotta PTP (ja DAQ-käytettävyys) olisi aina päällä ilman graafista käyttöliittymää:

  ```bash
  sudo systemctl enable --now chloros-backend.service
  ```

  Ilman tätä PTP toimii vain silloin, kun Chloros-käyttöliittymä on auki.

## Vianmääritys: ”No DAQ-E devices found”

| Tarkistus | Yksityiskohdat |
| --- | --- |
| Virta | Anturissa ei pala LED-valoa — tarkista kytkimen/injektorin portin PoE- ja linkkivalot; odota muutama sekunti virran kytkemisen jälkeen |
| Lähetysalue | Isäntä ja anturi samassa L2-segmentissä; mDNS ei reititä |
| Windows-palomuuri | Hyväksy Defender-kehote ensimmäisellä käyttökerralla (UDP 5002, 5353, 319/320) |
| Usean verkkokortin isäntä | Ensimmäisessä tunnistuksessa käynnistyksen jälkeen anturi saattaa jäädä huomaamatta — muodosta yhteys `--eth-host <ip-or-hostname>`:n avulla |
| Uudelleentarkistus käyttöliittymässä | Tunnistus toimii vain, kun yhteysvalintaikkuna on auki; käytä Päivitä-painiketta |</version>
