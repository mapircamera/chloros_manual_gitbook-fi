# Käsittelyn seuranta

Kun käsittely on alkanut, Chloros tarjoaa useita tapoja seurata edistymistä, tarkistaa ongelmia ja ymmärtää, mitä datajoukolle tapahtuu. Tällä sivulla selitetään, miten voit seurata käsittelyä ja tulkita Chloros:n tarjoamia tietoja.

## Edistymispalkin yleiskatsaus

Yläotsikon edistymispalkki näyttää reaaliaikaisen käsittelyn tilan ja valmistumisprosentin.

### Ilmainen tila -edistymispalkki

Käyttäjille, joilla ei ole Chloros+ -lisenssiä:

**2-vaiheinen edistymisen näyttö:**

1. **Kohteen tunnistus** - Kalibrointikohteiden etsiminen kuvista
2. **Käsittely** - Korjausten tekeminen ja vienti

**Edistymispalkki näyttää:**

* Kokonaisvalmistumisprosentin (0–100 %)
* Nykyisen vaiheen nimen
* Yksinkertaisen vaakapalkin visualisoinnin

### Chloros+ -edistymispalkki

Käyttäjille, joilla on Chloros+ -lisenssi:

**4-vaiheinen edistymisen näyttö:**

1. **Tunnistaminen** - Kalibrointikohteiden etsiminen
2. **Analysointi** - Kuvien tarkastelu ja putkilinjan valmistelu
3. **Kalibrointi** - Vignette- ja heijastavuuskorjausten soveltaminen
4. **Vienti** - Käsiteltyjen tiedostojen tallentaminen

**Interaktiiviset ominaisuudet:**

* **Vie hiiri** edistymispalkin päälle nähdäksesi laajennetun 4-vaiheisen paneelin
* **Napsauta** edistymispalkkia jäädyttääksesi/kiinnittääksesi laajennetun paneelin
* **Napsauta uudelleen** vapauttaaksesi jäädytyksen ja piilottaaksesi paneelin automaattisesti, kun hiiri poistuu sen päältä
* Jokainen vaihe näyttää yksilöllisen edistymisen (0–100 %)

***

## Kunkin käsittelyvaiheen ymmärtäminen

### Vaihe 1: Tunnistaminen (kohteen tunnistaminen)

**Mitä tapahtuu:**

* Chloros skannaa kuvat, jotka on merkitty Kohde-valintaruudulla
* Tietokoneen näköalgoritmit tunnistavat 4 kalibrointipaneelia
* Heijastavuusarvot poimitaan kustakin paneelista
* Kohteiden aikaleimat tallennetaan oikean kalibrointiaikataulun laatimista varten

**Kesto:**

* Merkityillä kohteilla: 10–60 sekuntia
* Ilman merkittyjä kohteita: 5–30+ minuuttia (skannaa kaikki kuvat)

**Edistymisilmaisin:**

* Tunnistaminen: 0 % → 100 %
* Skannattujen kuvien määrä
* Löydettyjen kohteiden määrä

**Mitä on syytä tarkkailla:**

* Pitäisi valmistua nopeasti, jos kohteet on merkitty oikein
* Jos kestää liian kauan, kohteita ei ehkä ole merkitty
* Tarkista Debug Log -lokista, onko siellä viestejä &quot;Target found&quot; (Kohde löydetty)

### Vaihe 2: Analysointi

**Mitä tapahtuu:**

* Kuvien EXIF-metatietojen lukeminen (aikaleimat, valotusasetukset)
* Kalibrointistrategian määrittäminen kohteen aikaleimojen perusteella
* Kuvankäsittelyjonon järjestäminen
* Rinnakkaiskäsittelytyöntekijöiden valmistelu (vain Chloros+)

**Kesto:** 5–30 sekuntia

**Edistymisilmaisin:**

* Analysointi: 0 % → 100 %
* Nopea vaihe, valmistuu yleensä nopeasti

**Mitä on syytä tarkkailla:**

* Etenemisen tulisi olla tasaista ilman taukoja
* Varoitukset puuttuvista metatiedoista näkyvät vianmäärityslogissa

### Vaihe 3: Kalibrointi

**Mitä tapahtuu:**

* **Debayering**: RAW-Bayer-kuvion muuntaminen 3 kanavaksi
* **Vignette-korjaus**: Poistaa objektiivin reunan tummenemisen
* **Heijastavuuskalibrointi**: Normalisointi tavoitearvoilla
* **Indeksilaskenta**: Laskee monispektriset indeksit
* Käsittelee jokaisen kuvan koko prosessin läpi

**Kesto:** Suurin osa kokonaiskäsittelyajasta (60–80 %)

**Edistymisen osoitin:**

* Kalibrointi: 0 % → 100 %
* Käsiteltävä kuva
* Valmiit kuvat / Kuvien kokonaismäärä

**Käsittelyn toiminta:**

* **Vapaa tila**: Käsittelee yhden kuvan kerrallaan peräkkäin
* **Chloros+ -tila**: Käsittelee jopa 16 kuvaa samanaikaisesti
* **GPU-kiihdytys**: Nopeuttaa tätä vaihetta merkittävästi

**Mitä on syytä tarkkailla:**

* Tasainen eteneminen kuvamäärän suhteen
* Tarkista Debug Log -lokista kuvakohtaiset valmistumisilmoitukset
* Varoitukset kuvanlaadusta tai kalibrointiongelmista

### Vaihe 4: Vienti

**Mitä tapahtuu:**

* Kalibroitujen kuvien kirjoittaminen levylle valitussa muodossa
* Monispektrisen indeksin kuvien vieminen LUT-väreillä
* Kameramallien alikansioiden luominen
* Alkuperäisten tiedostonimien säilyttäminen sopivilla päätteillä

**Kesto:** 10–20 % kokonaiskäsittelyajasta

**Edistymisen ilmaisin:**

* Vienti: 0 % → 100 %
* Tiedostojen kirjoittaminen
* Vientimuoto ja kohde

**Mitä on syytä tarkkailla:**

* Levytilan varoitukset
* Tiedostojen kirjoitusvirheet
* Kaikkien määritettyjen tulosteiden valmistuminen

***

## Debug-lokivälilehti

Debug-loki tarjoaa yksityiskohtaista tietoa käsittelyn edistymisestä ja mahdollisista ongelmista.

### Debug-lokin avaaminen

1. Napsauta **Debug-loki** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> -kuvaketta vasemmassa sivupalkissa.
2. Lokipaneeli avautuu ja näyttää reaaliaikaiset käsittelyviestit.
3. Vierittää automaattisesti uusimpia viestejä.

### Lokiviestien ymmärtäminen

#### Tiedotukset (valkoinen/harmaa)

Normaalit käsittelypäivitykset:

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Varoitukset (keltainen)

Ei-kriittiset ongelmat, jotka eivät keskeytä käsittelyä:

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Toimenpide:** Tarkista varoitukset käsittelyn jälkeen, mutta älä keskeytä käsittelyä.

#### Virheilmoitukset (Red)

Kriittiset ongelmat, jotka voivat aiheuttaa käsittelyn epäonnistumisen:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Toimenpide:** Keskeytä käsittely, korjaa virhe, käynnistä uudelleen.

### Yleiset lokiviestit

| Viesti                          | Merkitys                                | Tarvittava toimenpide                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| &quot;Kohde havaittu tiedostossa \[tiedostonimi]&quot; | Kalibrointikohde löydetty onnistuneesti  | Ei mitään - normaali                                         |
| &quot;Käsitellään kuvaa X/Y&quot;        | Nykyisen edistymisen päivitys                | Ei mitään - normaali                                         |
| &quot;Kohteita ei löytynyt&quot;               | Kalibrointikohteita ei havaittu        | Merkitse kohdekuvat tai poista heijastavuuden kalibrointi käytöstä |
| &quot;Levytilaa ei riitä&quot;        | Tallennustilaa ei riitä tulostukseen          | Vapauta levytilaa                                    |
| &quot;Ohitetaan vioittunut tiedosto&quot;        | Kuvatiedosto on vioittunut                  | Kopioi tiedosto uudelleen SD-kortilta                             |
| &quot;PPK-tiedot sovellettu&quot;               | .daq-tiedoston GPS-korjaukset sovellettu | Ei mitään - normaali                                         |

### Lokitietojen kopiointi

Lokien kopiointi vianmääritystä tai tukea varten:

1. Avaa Debug Log -paneeli.
2. Napsauta **&quot;Copy Log&quot;**-painiketta (tai napsauta hiiren kakkospainikkeella → Valitse kaikki).
3. Liitä tekstitiedostoon tai sähköpostiin.
4. Lähetä tarvittaessa MAPIR-tukeen.

***

## Järjestelmän resurssien valvonta

### CPU:n käyttö

**Vapaa tila:**

* 1 CPU-ydin ~100 %:ssa
* Muut ytimet käyttämättöminä tai käytettävissä
* Järjestelmä reagoi edelleen

**Chloros+ rinnakkaistila:**

* Useita ytimiä 80–100 %:ssa (enintään 16 ydintä)
* Korkea CPU:n kokonaiskäyttöaste
* Järjestelmä voi tuntua vähemmän reagoivalta

**Seuranta:**

* Windows Tehtävienhallinta (Ctrl+Shift+Esc)
* Suorituskyky-välilehti → CPU-osio
* Etsi prosessit &quot;Chloros&quot; tai &quot;chloros-backend&quot;

### Muistin (RAM) käyttö

**Tyypillinen käyttö:**

* Pienet projektit (&lt; 100 kuvaa): 2–4 Gt
* Keskisuuret projektit (100–500 kuvaa): 4–8 Gt
* Suuret projektit (yli 500 kuvaa): 8–16 Gt
* Chloros+ rinnakkaistila käyttää enemmän RAM-muistia

**Jos muistia on vähän:**

* Käsittele pienempiä eriä
* Sulje muut sovellukset
* Päivitä RAM-muistia, jos käsittelet säännöllisesti suuria tietojoukkoja

### GPU:n käyttö (Chloros+ ja CUDA)

Kun GPU-kiihdytys on käytössä:

* NVIDIA GPU:n käyttöaste on korkea (60–90 %)
* VRAM-muistin käyttö kasvaa (vaatii vähintään 4 Gt VRAM-muistia)
* Kalibrointivaihe on huomattavasti nopeampi

**Seuranta:**

* NVIDIA-kuvaketta järjestelmäpalkissa
* Tehtävienhallinta → Suorituskyky → GPU
* GPU-Z tai vastaava seurantatyökalu

### Levyn I/O

**Mitä odottaa:**

* Korkea levyn lukunopeus analysointivaiheessa
* Korkea levyn kirjoitusnopeus vientivaiheessa
* SSD on huomattavasti nopeampi kuin HDD

**Suorituskykyvinkki:**

* Käytä SSD-levyä projektikansiossa, jos mahdollista
* Vältä verkkoasemia suurille tietojoukoille
* Varmista, että levyn kapasiteetti ei ole lähes täynnä (vaikuttaa kirjoitusnopeuteen)

***

## Ongelmien havaitseminen käsittelyn aikana

### Varoitusmerkit

**Edistyminen pysähtyy (ei muutosta yli 5 minuuttiin):**

* Tarkista virheistä virheloki
* Tarkista levytilan saatavuus
* Tarkista Tehtävienhallinnasta, että Chloros on käynnissä

**Virheilmoitukset näkyvät usein:**

* Keskeytä käsittely ja tarkista virheet
* Yleisiä syitä: levytila, vioittuneet tiedostot, muistiongelmat
* Katso alla oleva Vianmääritys-osio

**Järjestelmä ei vastaa:**

* Chloros+ rinnakkaistila käyttää liikaa resursseja
* Harkitse samanaikaisten tehtävien vähentämistä tai laitteiston päivittämistä
* Vapaa tila vaatii vähemmän resursseja

### Milloin käsittely on keskeytettävä

Keskeytä käsittely, jos näet:

* ❌ &quot;Levy täynnä&quot; tai &quot;Tiedostoa ei voi kirjoittaa&quot; -virheet
* ❌ Toistuvat kuvatiedostojen vioittumisvirheet
* ❌ Järjestelmä on täysin jumissa (ei vastaa)
* ❌ Huomaat, että asetukset on määritetty väärin
* ❌ Väärät kuvat on tuotu

**Kuinka lopettaa:**

1. Napsauta **Lopeta/Peruuta-painiketta** (korvaa Käynnistä-painikkeen)
2. Käsittely keskeytyy, edistys menetetään
3. Korjaa ongelmat ja aloita alusta

***

## Vianmääritys käsittelyn aikana

### Käsittely on erittäin hidasta

**Mahdolliset syyt:**

* Merkitsemättömät kohdekuvat (kaikkien kuvien skannaus)
* HDD SSD-tallennustilan sijaan
* Riittämättömät järjestelmäresurssit
* Määritettyjä hakemistoja on paljon
* Verkkolevyn käyttö

**Ratkaisut:**

1. Jos käsittely on juuri alkanut ja on tunnistamisvaiheessa: Peruuta, merkitse kohteet, aloita alusta
2. Tulevaisuutta varten: Käytä SSD-levyä, vähennä hakemistojen määrää, päivitä laitteisto
3. Harkitse CLI:n käyttöä suurten tietojoukkojen eräprosessointiin

### &quot;Levytila&quot; -varoitukset

**Ratkaisut:**

1. Vapauta levytilaa välittömästi
2. Siirrä projekti levylle, jossa on enemmän tilaa
3. Vähennä vietävien indeksien määrää.
4. Käytä JPG-muotoa TIFF:n sijaan (pienemmät tiedostot).

### Usein toistuvat &quot;Corrupted File&quot; -viestit

**Ratkaisut:**

1. Kopioi kuvat uudelleen SD-kortilta varmistaaksesi niiden eheyden.
2. Testaa SD-kortti virheiden varalta.
3. Poista vioittuneet tiedostot projektista.
4. Jatka jäljellä olevien kuvien käsittelyä.

### Järjestelmän ylikuumeneminen / kuristaminen

**Ratkaisut:**

1. Varmista riittävä ilmanvaihto.
2. Puhdista pöly tietokoneen tuuletusaukoista.
3. Vähennä käsittelykuormitusta (käytä Free-tilaa Chloros+:n sijaan).
4. Käsittele kuvat päivän viileimpään aikaan.

***

## Ilmoitus käsittelyn päättymisestä

Kun käsittely on päättynyt:

* Edistymispalkki saavuttaa 100 %
* **&quot;Käsittely valmis&quot;** -viesti näkyy vianmäärityslogissa
* Käynnistyspainike aktivoituu uudelleen
* Kaikki tulostustiedostot ovat kameramallin alikansiossa

***

## Seuraavat vaiheet

Kun käsittely on valmis:

1. **Tarkista tulokset** - Katso [Käsittelyn lopettaminen](finishing-the-processing.md)
2. **Tarkista tulostuskansio** - Varmista, että kaikki tiedostot on viety oikein
3. **Tarkista virheenkorjausloki** - Tarkista, onko siinä varoituksia tai virheitä
4. **Esikatsele käsiteltyjä kuvia** - Käytä kuvankatseluohjelmaa tai ulkoista ohjelmistoa

Lisätietoja käsiteltyjen tulosten tarkastelusta ja käytöstä on kohdassa [Käsittelyn lopettaminen](finishing-the-processing.md).
