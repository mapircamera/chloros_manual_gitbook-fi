# Kameroiden liittäminen

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>Kamerat-välilehti ennen kuin mitään on kytketty</p></figcaption></figure>Chloros tunnistaa LATTICE-kamerat linkissä automaattisesti — GUI:n Kamerat-välilehdestä, `chloros-cli lattice`:sta tai Python:stä ja SDK:stä. Kameran mallimerkkijono ohjaa kaikkea seuraavaa: Chloros määrittää anturiprofiilin, kaistajärjestelyn ja tehdaskalibroinnin kameran `DeviceUserID` + `DeviceSerialNumber`-tiedoista, joten **kameraa kohden ei tarvitse määrittää mitään**.

Varmista ennen liittämistä, että isäntäverkko on määritetty — link-local-osoitteistus, jumbo-kehykset ja, jos kyseessä on ryhmä, verkkokortin vastaanottopuskuri-asetukset. Nämä ovat laitteistopuolen asetuksia, ja ne löytyvät LATTICE-käyttöoppaasta: [**Verkon asetukset**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Yhdistäminen käyttöliittymän kautta

Avaa **Kamerat**-välilehti Chloros-sivupalkissa (laitteistoa koskevat välilehdet tulevat näkyviin, kun taustapalvelu on käynnistynyt) tai käytä päävalikkoa →**Yhdistä kameraan**. Molemmat avaavat**Yhdistä kamera(t)**-valintaikkunan.

### Yhdistä kamera(t)-valintaikkuna

Valintaikkuna skannaa verkon heti avautuessaan (&quot;Skannataan verkkoa...&quot;) ja listaa kaikki löytämänsä kamerat. Jokaisessa rivissä näkyy kameran **malli**(esim. `LATT-M3M-L41-F550`),**sarjanumero**ja**IP-osoite**.

* **Napsauta riviä valitaksesi sen**(vihreä korostus). Voit valita**useita kameroita** ja yhdistää ne kerralla — Chloros yhdistää ne peräkkäin.
* Rivit, joissa on **&quot;Yhdistetty&quot;**-merkki, on jo yhdistetty eikä niitä voi valita uudelleen.
* Rivit, joissa on **&quot;Ryhmässä&quot;**-merkki, kuuluvat tällä hetkellä yhdistettyyn kameraryhmään. Irrota ryhmä ensin, jotta voit käyttää kyseistä kameraa itsenäisesti.
* **Yhdistä** — yhdistää valitut kamerat; painikkeessa näkyy lukumäärä, esim. &quot;Yhdistä (3)&quot;, kun valittuna on useampi kuin yksi kamera.
* **Skannaa uudelleen** — suorittaa tunnistuksen uudelleen.
* **Sulje** — sulkee valintaikkunan.
* Jos haku päättyy ilman tuloksia, valintaikkunassa näkyy **&quot;Verkosta ei löytynyt kameroita&quot;** — katso alla oleva [Vianmääritys](connecting.md#troubleshooting).

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>Yhdistä kamera(t)-valintaikkuna — kuvassa verkossa ei ole kameroita</p></figcaption></figure>### Ensimmäinen kytkentä: kalibrointipaketin lataus

Kun tietty kamera kytketään koneeseen **ensimmäistä kertaa**, Chloros lataa kameran tehdaskalibrointipaketin (\~3,8 MB) itse kamerasta GigE-yhteyden kautta. Tämän aikana valintaikkunassa näkyy vihreä**”Lataa kalibrointitietoja kamerasta”**-paneeli, jossa on sarjanumeroittain jaettu edistymispalkki — kesto on noin**70 sekuntia** kameraa kohti. Paketti tallennetaan välimuistiin isäntäkoneelle, joten saman kameran myöhemmät liitännät ohittavat latauksen kokonaan (eikä paneelia näytetä koskaan).

### Analysoi järjestelmä

Valintaikkunan **Analysoi järjestelmä** -painike tutkii isäntäkoneen ja verkon (teksti ”Analysoidaan...” näkyy prosessin aikana) ja tuottaa diagnostiikkaraportin:

* **Isäntäkone** — CPU-ytimet ja RAM-muisti; GPU:n nimi ja muisti, tai ”GPU: Ei havaittu”.
* **Verkkoliitännät** — kunkin verkkokortin nimi, linkkinopeus, MTU (”jumbo”-merkinnällä, jos käytössä), ylös-/alas-tila ja se, sijaitseeko se USB-väylällä.
* **Kamerat**— sarjanumero, malli, IP-osoite ja**mille verkkokortille kukin kamera on kytketty**.
* **Suorituskyky** — kamerakohtainen nykyinen ja ihanteellinen kuvanopeus (fps) pikselimuodolle, ja vihreä rivi ”Potentiaali: N-kertainen parannus mahdollinen”, kun ihanteellinen arvo ylittää nykyisen.
* **Varoitukset ja numeroidut suositukset** — tai viesti ”Järjestelmä näyttää hyvältä nykyisen kameramäärän suhteen”, kun korjattavaa ei ole.

Suorita tämä aina, kun laitteiden tunnistus tai suoratoisto toimii odottamattomasti — se tunnistaa useimmat verkkokorttipuolen ongelmat (väärä MTU, kamera väärällä liitännällä, USB-sovittimen rajoitukset) poistumatta valintaikkunasta.

### Ryhmän liittäminen

Jos haluat liittää kaksi tai useampia kameroita **synkronoiduksi ryhmäksi**, käytä sen sijaan ryhmän liittämisen ohjatun toiminnon (**Liitä kameraryhmä**): se opastaa sinut läpi pää-/alaislaitteen valinnan (joka on esitäytetty GPIO-kytkentätestillä), näyttötilan valinnan (erilliset vs. yhdistetyt ruudut) sekä ryhmän asetusnäytön, jossa näkyy reaaliaikainen ennuste saavutettavissa olevasta kuvataajuudesta (fps) ja kaistanleveydestä ennen kuin vahvistat valinnat. Ohjattu toiminto ja kameraryhmän työnkulut käsitellään tämän käyttöoppaan monikameraryhmät-osiossa; CLI-mallin vastaava menettely on ”LATTICE-kameran ensimmäisen kytkennän työnkulku” [CLI-viiteoppaassa](../reference/cli-reference.md).

## Yhteyden muodostaminen CLI- ja SDK-palveluista

CLI- ja SDK-palveluiden käyttö edellyttää maksullista Chloros+-tasoa ja kirjautumista sisään; tämä valvotaan palvelinpuolella (`401 AUTH_REQUIRED`, kun käyttäjä ei ole kirjautunut sisään, `403 PLAN_UPGRADE_REQUIRED` ilmaisella tasolla).

```bash
# List cameras on the network (vendor, model, serial, IP, MAC)
chloros-cli lattice info

# Single-camera smoke test: capture one frame (saves every applicable export type)
chloros-cli lattice capture -o output/

# Connect a synchronized array — same smart-prep flow as the GUI
chloros-cli lattice array-connect --serials 213800234,214000533
```

```python
import chloros_sdk

# Persistent live-camera session through the backend
with chloros_sdk.connect_camera("213800234") as cam:
    ...

# Array session (smart-prep: network probe, tier auto-pick, PTP, AE seeding, trigger config)
with chloros_sdk.connect_array(["213800234", "214000533"]) as array:
    ...
```

Täydelliset allekirjoitukset, asetukset ja sieppausprosessit: [CLI-viite](../reference/cli-reference.md) § `chloros-cli lattice`, [SDK-viite](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Miten kalibrointi hoidetaan yhteyden muodostamisen yhteydessä

Jokaisessa LATTICE-kamerassa on tehtaalla määritetty kalibrointipaketti **kameran sisällä**, ja Chloros tarkistaa myös MAPIR:n pilvipalvelun, kun kamera muodostaa yhteyden:

| Tilanne   | Mitä Chloros käyttää                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Verkossa**|**Kyseiselle sarjanumerolle julkaistu uusin kalibrointi** — pilvipalvelun kopio ohittaa kameran sisäisen kopion. Kamera, joka on kalibroitu uudelleen tai päivitetty MAPIR:n avulla, päivittyy siten automaattisesti; käyttäjän toimia ei tarvita. |
| **Offline**|**Kameran sisäinen kalibrointipaketti** sellaisenaan. Täysin offline-työnkulut toimivat edelleen; ne eivät yksinkertaisesti ota käyttöön uudempia kalibrointeja, ennen kuin kamera on ollut kerran verkossa (tai sen ohjelmisto on päivitetty tehdasasetuksiin).                                                  |

Kuvaushetkellä tosiasiallisesti käytetyt kertoimet **jäävät kiinteästi kunkin kuvan XMP-metatietoihin**. Myöhempi kalibrointipäivitys ei koskaan muuta hiljaisesti jo ottamiasi kuvia — vanhan kuvan uudelleenkäsittelyssä käytetään sen XMP-tiedostoon tallennettuja kertoimia, ei sitä, mikä on tällä hetkellä uusinta.

## Vianmääritys

* **&quot;Verkosta ei löytynyt kameroita&quot;**— tarkista linkkikohtainen asetus kohdassa [Verkkoasetukset](https://mapir.gitbook.io/lattice-camera/setup/network-setup): isäntäkoneen verkkokortti staattinen `169.254.x.x/16`, kamerat samalla linkillä, DHCP:tä tai yhdyskäytävää ei odoteta. Käytä sitten yhteysvalintaikkunan**Analyze System**-toimintoa tarkistaaksesi, millä verkkokortilla kukin kamera näkyy (tai ei näy). Suorita**Rescan**-toiminto aina kaapelointia tai verkkokortin vaihtoa jälkeen.
* **Aiemmin toimiva järjestelmä ei muodosta yhteyttä** (array-paneelin virheviestit `FRAMES WILL DROP` / `Reduce ROI to enable`) — verkkokortin ohjaimen päivitys on nollannut vastaanottorengasasetukset ilman erillistä ilmoitusta. Määritä asetukset uudelleen tai suorita komento `chloros-cli lattice network --fix` pääkäyttäjän oikeuksilla; katso [Verkon asetukset](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **Kamerassa näkyy teksti ”In Array”** — se kuuluu kytkettyyn ryhmäistuntoon. Irrota ryhmä, jotta voit käyttää kameraa itsenäisesti.
