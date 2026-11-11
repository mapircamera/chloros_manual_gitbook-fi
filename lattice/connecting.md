# Kameroiden liittäminen

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>Kamerat-välilehti ennen kuin mitään on kytketty</p></figcaption></figure>

Chloros
tunnistaa LATTICE-kamerat verkosta automaattisesti — joko käyttöliittymän Kamerat-välilehdestä, tiedostosta `chloros-cli lattice` tai Python
-SDK
:sta. Kameran mallimerkkijono ohjaa kaikkea jatkokäsittelyä: Chloros
selvittää anturiprofiilin, kaistajärjestelyn ja tehdaskalibroinnin kameran `DeviceUserID` + `DeviceSerialNumber`-tiedoista, joten **kameraa kohden ei tarvitse määrittää mitään**.

Varmista ennen liittämistä, että isäntäverkko on määritetty — link-local-osoitteistus, jumbo-kehykset ja, jos kyseessä on kameraryhmä, verkkokortin vastaanottopuskuri-asetukset. Tämä on laitteistopuolinen asetus, joka löytyy LATTICE-käyttöoppaasta: [**Verkon asetukset**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Yhdistäminen graafisen käyttöliittymän kautta

Avaa **Kamerat**-välilehti Chloros
-sivupalkissa (laitteistovälilehdet tulevat näkyviin, kun taustapalvelu on käynnistynyt) tai käytä päävalikkoa → **Yhdistä kameraan**. Molemmat avaavat**Yhdistä kamera(t)**-valintaikkunan.

### Yhdistä kamera(t)-valintaikkuna

Valintaikkuna skannaa verkon heti avautuessaan (&quot;Skannataan verkkoa...&quot;), ja listaa kaikki löytämänsä kamerat. Jokaisessa rivissä näkyy kameran **malli**(esim. `LATT-M3M-L41-F550`),**sarjanumero**ja**IP-osoite**.

* **Valitse rivi napsauttamalla sitä**(vihreä korostus). Voit valita**useita kameroita** ja liittää ne kerralla — Chloros
liittää ne peräkkäin.
* Rivit, joissa on **&quot;Liitetty&quot;**-merkki, ovat jo liitettyjä, eikä niitä voi valita uudelleen.
* Rivit, joissa on **&quot;Ryhmässä&quot;**-merkki, kuuluvat tällä hetkellä liitettyyn kameraryhmään. Irrota ryhmä ensin, jotta voit käyttää kyseistä kameraa itsenäisesti.
* **Yhdistä** — yhdistää valitut kamerat; painikkeessa näkyy lukumäärä, esim. ”Yhdistä (3)”, kun valittuna on useampi kuin yksi kamera.
* **Skannaa uudelleen** — suorittaa tunnistuksen uudelleen.
* **Sulje** — sulkee valintaikkunan.
* Jos haku päättyy ilman tuloksia, valintaikkunassa näkyy **&quot;Verkosta ei löytynyt kameroita&quot;** — katso [Vianmääritys](connecting.md#troubleshooting) alla.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>Yhdistä kamera(t) -valintaikkuna — kuvassa verkossa ei ole kameroita</p></figcaption></figure>

### Ensimmäinen kytkentä: kalibrointipaketin lataus

Kun tietty kamera kytketään laitteeseen **ensimmäistä kertaa**, Chloros
lataa kameran tehdaskalibrointipaketin (\~3,8 MB) itse kamerasta GigE-yhteyden kautta. Tämän aikana valintaikkunassa näkyy vihreä **&quot;Lataa kalibrointitietoja kamerasta&quot;**-paneeli, jossa on sarjanumeroittain jaettu edistymispalkki — kesto on noin**70 sekuntia** kameraa kohti. Kun kameroita on yli kuusi, edistymisrivit vierivät paneelin sisällä. Paketti tallennetaan välimuistiin isäntäkoneelle, joten saman kameran myöhemmät liitännät ohittavat latauksen kokonaan (eikä paneelia näytetä lainkaan).

### Järjestelmän analysointi

Valintaikkunan **Analyze System** -painike tutkii isäntäkoneen ja verkon (teksti &quot;Analyzing...&quot; näkyy toiminnon aikana) ja tuottaa diagnostiikkaraportin:

* **Host** — CPU-ytimet ja RAM-muisti; GPU:n nimi ja muisti, tai &quot;GPU: None detected&quot;.
* **Verkkoliitännät** — kunkin verkkokortin nimi, linkkinopeus, MTU (”jumbo”-merkinnällä, jos käytössä), ylös-/alas-tila ja tieto siitä, sijaitseeko se USB-väylällä.
* **Kamerat**— sarjanumero, malli, IP-osoite ja**mille verkkokortille kukin kamera on kytketty**.
* **Suorituskyky** — kamerakohtainen nykyinen ja ihanteellinen kuvanopeus (fps) pikselimuodolle, ja vihreä rivi ”Potentiaali: N-kertainen parannus mahdollinen”, kun ihanteellinen arvo ylittää nykyisen.
* **Varoitukset ja numeroidut suositukset** — tai viesti ”Järjestelmä näyttää toimivan hyvin nykyisen kameramäärän osalta”, kun korjattavaa ei ole.

Suorita tämä aina, kun laitteiden tunnistus tai suoratoisto toimii odottamattomasti — se tunnistaa useimmat verkkokorttipuolen ongelmat (väärä MTU, kamera väärällä liitännällä, USB-sovittimen rajoitukset) poistumatta valintaikkunasta.

### Ryhmän liittäminen

Jos haluat liittää kaksi tai useampia kameroita **synkronoiduksi ryhmäksi**, käytä sen sijaan ryhmän liittämisohjetta (**Liitä kameraryhmä**): se opastaa sinut läpi pää-/alaiskameran valinnan (esitäytetty synkronointikaapelin tunnistuksen perusteella — mikä tahansa kaapelilla liitetty kamera voi toimia pääkamerana, ja RGB
-kamera `-FRGB` on valittu oletuksena, kun se on kytketty kaapelilla), näyttötilan valinnan (erilliset vs. yhdistetyt ruudut) sekä ryhmäasetusten näkymän, jossa näkyy reaaliaikainen ennuste saavutettavissa olevasta kuvataajuudesta (fps) ja kaistanleveydestä ennen vahvistamista. Ohjattu toiminto ja kameraryhmän työnkulut käsitellään tämän käyttöoppaan monikameraryhmiä koskevassa osiossa; CLI
-vastaava on ”LATTICE Camera First-Connect Workflow” [CLI
-viitteessä](../reference/cli-reference.md).

## Yhteyden muodostaminen CLI
:n ja SDK
:n kautta

CLI
:n ja SDK
:n käyttö edellyttää maksullista Chloros
+-tasoa ja kirjautumista sisään; tämä vaaditaan palvelinpuolella (`401 AUTH_REQUIRED`, kun et ole kirjautunut sisään, `403 PLAN_UPGRADE_REQUIRED` ilmaisella tasolla).

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

Täydelliset allekirjoitukset, vaihtoehdot ja tallennusprosessit: [CLI
-ohje](../reference/cli-reference.md) § `chloros-cli lattice`, [SDK
-ohje](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Miten kalibrointi hoidetaan yhteyden muodostamisen yhteydessä

Jokaisessa LATTICE-kamerassa on tehtaalla kalibroitu paketti **kameran sisällä**, ja Chloros
tarkistaa myös MAPIR
in pilvipalvelun, kun kamera muodostaa yhteyden:

| Tilanne   | Mitä Chloros
käyttää                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Verkossa**|**Kyseiselle sarjanumerolle julkaistu uusin kalibrointi** — pilvipalvelun kopio syrjäyttää kameran sisäisen kopion. Kamera, joka on kalibroitu uudelleen tai päivitetty MAPIR
in toimesta, päivittää itsensä siten automaattisesti; käyttäjän toimia ei tarvita. |
| **Offline**|**Kameran sisäinen kalibrointipaketti** sellaisenaan. Täysin offline-työnkulut toimivat edelleen; ne eivät yksinkertaisesti ota käyttöön uudempia kalibrointeja, ennen kuin kamera on kerran ollut verkossa (tai sen ohjelmisto on päivitetty tehdasasetuksiin).                                                  |

Kuvaushetkellä tosiasiallisesti käytetyt kertoimet **tallennetaan kunkin kuvan XMP-metatietoihin**. Myöhempi kalibrointipäivitys ei koskaan muuta hiljaisesti jo ottamiasi kuvia — vanhan kuvan uudelleenkäsittelyssä käytetään sen XMP-tiedostoon tallennettuja kertoimia, ei sitä, mikä on tällä hetkellä uusin.

## Vianmääritys

* **&quot;Verkosta ei löytynyt kameroita&quot;**— tarkista linkkikohtaiset asetukset kohdassa [Verkkoasetukset](https://mapir.gitbook.io/lattice-camera/setup/network-setup): isäntäkoneen verkkokortti staattinen `169.254.x.x/16`, kamerat samalla linkillä, DHCP:tä tai yhdyskäytävää ei odoteta. Käytä sitten yhteysvalintaikkunan**Järjestelmän analysointi**-toimintoa tarkistaaksesi, millä verkkokortilla kukin kamera näkyy (tai ei näy). Suorita**uusi haku** kaapelointimuutosten tai verkkokortin vaihdon jälkeen.
* **Aiemmin toiminut järjestelmä ei muodosta yhteyttä** (matriisipaneelin portit, joissa on virheet `FRAMES WILL DROP` / `Reduce ROI to enable`) — verkkokortin ohjaimen päivitys on nollannut vastaanottorengasasetukset ilman erillistä ilmoitusta. Määritä asetukset uudelleen tai suorita komento `chloros-cli lattice network --fix` pääkäyttäjän oikeuksilla; katso [Verkon asetukset](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **Kamerassa näkyy teksti ”In Array”** — se kuuluu kytkettyyn ryhmäistuntoon. Irrota ryhmä, jotta voit käyttää kameraa itsenäisesti.
