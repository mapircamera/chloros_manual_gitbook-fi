# Käsittelyn seuranta

Kun käsittely on alkanut, Chloros tarjoaa useita tapoja seurata edistymistä, tarkistaa mahdolliset ongelmat ja selvittää, mitä tietojoukolle tapahtuu. Tällä sivulla kerrotaan, miten voit seurata käsittelyä ja tulkita Chloros:n tarjoamia tietoja.

## Edistymispalkin yleiskatsaus

Yläreunan otsikossa oleva edistymispalkki näyttää käsittelyn tilan ja valmistumisprosentin reaaliajassa.

### Vapaamoodin edistymispalkki

Käyttäjille, joilla ei ole Chloros+-lisenssiä:

**2-vaiheinen edistymisnäyttö:**

1.**Kohteen tunnistus** – Kalibrointikohteiden etsiminen kuvista
2. **Käsittely** – Korjausten soveltaminen ja vienti**Edistymispalkki näyttää:**

* Kokonaisvalmiusprosentti (0–100 %)
* Nykyisen vaiheen nimi
* Yksinkertainen vaakasuora palkkikuvaus

### Chloros+:n edistymispalkki

Käyttäjille, joilla on Chloros+-lisenssi:

**4-vaiheinen edistymisnäyttö:**

1.**Tunnistus** – Kalibrointikohteiden etsiminen
2. **Analysointi** – Kuvien tarkastelu ja prosessiketjun valmistelu
3. **Kalibrointi** – Vignetoinnin ja heijastavuuden korjausten soveltaminen
4. **Vienti** – Käsiteltyjen tiedostojen tallentaminen**Interaktiiviset ominaisuudet:*** **Vie hiiri** etenemispalkin päälle nähdäksesi laajennetun 4-vaiheisen paneelin
* **Napsauta** etenemispalkkia jäädyttääksesi/kiinnittääksesi laajennetun paneelin
* **Napsauta uudelleen** vapauttaaksesi paneelin ja piilottaaksesi sen automaattisesti, kun hiiri viedään pois
* Jokainen vaihe näyttää yksilöllisen etenemisen (0–100 %)

***

## Kunkin käsittelyvaiheen ymmärtäminen

{% hint style="info" %}
**Pipeline-arkkitehtuuri**: Nämä 4 GUI-vaihetta vastaavat [4-säikeistä käsittelyputkea](../processing-architecture/processing-pipeline.md). GPU-kiihdytyksellä varustetuissa järjestelmissä säie 3 (Kalibrointi) hyötyy [dynaamisesta laskentasovituksesta](../processing-architecture/dynamic-compute-adaptation.md), joka optimoi käsittelyn laitteistollesi.
{% endhint %}

### Vaihe 1: Tunnistaminen (kohteen tunnistus)

**Mitä tapahtuu:**

* Chloros skannaa kuvat, jotka on merkitty Kohde-valintaruudulla
* Tietokoneen näköalgoritmit tunnistavat neljä kalibrointipaneelia
* Jokaisesta paneelista poimitaan heijastavuusarvot
* Kohteen aikaleimat tallennetaan oikean kalibrointiaikataulun varmistamiseksi

**Kesto:**

* Merkittyjen kohteiden kanssa: 10–60 sekuntia
* Ilman merkittyjä kohteita: 5–30+ minuuttia (skannaa kaikki kuvat)

**Edistymisilmaisin:**

* Tunnistaminen: 0 % → 100 %
* Skannattujen kuvien määrä
* Löydettyjen kohteiden lukumäärä

**Mitä on syytä tarkkailla:**

* Pitäisi valmistua nopeasti, jos kohteet on merkitty oikein
* Jos kestää liian kauan, kohteita ei ehkä ole merkitty
* Tarkista vianmäärityslokista ”Kohde löydetty” -viestit

### Vaihe 2: Analysointi

**Mitä tapahtuu:**

* Kuvan EXIF-metatietojen lukeminen (aikaleimat, valotusasetukset)
* Kalibrointistrategian määrittäminen kohteiden aikaleimojen perusteella
* Kuvankäsittelyjonon järjestäminen
* Rinnakkaiskäsittelytyöntekijöiden valmistelu (vain Chloros+)

**Kesto:** 5–30 sekuntia**Edistymisilmaisin:**

* Analysointi: 0 % → 100 %
* Nopea vaihe, valmistuu yleensä nopeasti

**Mitä tarkkailla:**

* Edistymisen tulisi olla tasaista ilman taukoja
* Varoitukset puuttuvista metatiedoista näkyvät vianmäärityslokissa

### Vaihe 3: Kalibrointi

**Mitä tapahtuu:*** **Debayering**: RAW-Bayer-kuvion muuntaminen 3 kanavaksi
* **Vignettikorjaus**: Objektiivin reunan tummumisen poistaminen
* **Heijastavuuden kalibrointi**: Normalisointi tavoitearvojen avulla
* **Indeksin laskeminen**: Monispektristen indeksien laskeminen
* Kunkin kuvan käsittely koko prosessin läpi

**Kesto:** Suurin osa kokonaiskäsittelyajasta (60–80 %)**Edistymisilmaisin:**

* Kalibrointi: 0 % → 100 %
* Käsiteltävänä oleva kuva
* Valmiit kuvat / Kuvien kokonaismäärä

**Käsittelykäyttäytyminen:*** **Vapaa tila**: Käsittelee kuvia yksi kerrallaan peräkkäin
* **Chloros+ -tila**: Käsittelee jopa 16 kuvaa samanaikaisesti
* **GPU-kiihdytys**: Nopeuttaa tätä vaihetta merkittävästi**Mitä on syytä seurata:**

* Tasainen eteneminen kuvamäärän suhteen
* Tarkista virhelokista kuvakohtaiset valmistumisilmoitukset
* Varoitukset kuvanlaadusta tai kalibrointiongelmista

### Vaihe 4: Vienti

**Mitä tapahtuu:**

* Kalibroitujen kuvien kirjoittaminen levylle valitussa muodossa
* Monispektristen indeksikuvien vieminen LUT-väreillä
* Kameramallien alikansioiden luominen
* Alkuperäisten tiedostonimien säilyttäminen sopivilla päätteillä

**Kesto:** 10–20 % kokonaiskäsittelyajasta**Edistymisilmaisin:**

* Vienti: 0 % → 100 %
* Tiedostoja kirjoitetaan
* Vientimuoto ja kohde

**Mitä kannattaa seurata:**

* Levytilan varoitukset
* Tiedostojen kirjoitusvirheet
* Kaikkien määritettyjen tulosteiden valmistuminen

***

## Debug Log -välilehti

Debug Log tarjoaa yksityiskohtaista tietoa käsittelyn etenemisestä ja mahdollisista ongelmista.

### Debug Login avaaminen

1. Napsauta **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> -kuvaketta vasemmassa sivupalkissa
2. Lokipaneeli avautuu ja näyttää reaaliaikaisia käsittelyviestejä
3. Vierittää automaattisesti näyttääkseen uusimmat viestit

### Lokiviestien ymmärtäminen

#### Tiedotusviestit (valkoinen/harmaa)

Normaalit käsittelypäivitykset:

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Varoitusviestit (keltainen)

Ei-kriittiset ongelmat, jotka eivät keskeytä käsittelyä:

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Toimenpide:** Tarkista varoitukset käsittelyn jälkeen, mutta älä keskeytä sitä

#### Virheilmoitukset (Red)

Kriittiset ongelmat, jotka voivat aiheuttaa käsittelyn epäonnistumisen:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Toimenpide:** Keskeytä käsittely, korjaa virhe ja käynnistä uudelleen

### Yleisiä lokiviestejä

| Viesti                          | Merkitys                                | Tarvittava toimenpide                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| &quot;Kohde havaittu tiedostossa \[tiedostonimi]&quot; | Kalibrointikohde löydetty onnistuneesti  | Ei mitään - normaali                                         |
| &quot;Käsitellään kuvaa X/Y&quot;        | Nykyisen edistymisen päivitys                | Ei mitään - normaali                                         |
| &quot;Kohteita ei löytynyt&quot;               | Kalibrointikohteita ei havaittu        | Merkitse kohdekuvat tai poista heijastuskalibrointi käytöstä |
| &quot;Levytilaa ei riitä&quot;        | Tallennustilaa ei riitä tulosteille          | Vapauta levytilaa                                    |
| &quot;Ohitetaan vioittunut tiedosto&quot;        | Kuvatiedosto on vioittunut                  | Kopioi tiedosto uudelleen SD-kortilta                             |
| &quot;PPK-tiedot sovellettu&quot;               | .daq-tiedoston GPS-korjaukset sovellettu | Ei mitään - normaali                                         |

### Lokitietojen kopioiminen

Lokin kopioiminen vianmääritystä tai tukea varten:

1. Avaa Debug Log -paneeli
2. Napsauta **&quot;Copy Log&quot;** -painiketta (tai napsauta hiiren oikealla painikkeella → Valitse kaikki)
3. Liitä tekstitiedostoon tai sähköpostiin
4. Lähetä tarvittaessa MAPIR-tukeen

***

## Järjestelmän resurssien seuranta

### CPU:n käyttö

**Vapaa tila:**

* 1 CPU-ydin ~100 %:ssa
* Muut ytimet lepotilassa tai käytettävissä
* Järjestelmä reagoi edelleen

**Chloros+ Rinnakkaistila:**

* Useita ytimiä 80–100 %:ssa (jopa 16 ydintä)
* Korkea CPU:n kokonaiskäyttöaste
* Järjestelmä saattaa tuntua vähemmän reagoivalta

**Seuranta:**

* Windows Tehtävienhallinta (Ctrl+Shift+Esc)
* Suorituskyky-välilehti → CPU-osio
* Etsi &quot;Chloros&quot; tai &quot;chloros-backend&quot; -prosessit

### Muistin (RAM) käyttö

**Tyypillinen käyttö:**

* Pienet projektit (&lt; 100 kuvaa): 2–4 Gt
* Keskisuuret projektit (100–500 kuvaa): 4–8 Gt
* Suuret projektit (yli 500 kuvaa): 8–16 Gt
* Chloros+ rinnakkaistilassa käytetään enemmän RAM-muistia

**Jos muistia on vähän:**

* Käsittele pienempiä erä
* Sulje muut sovellukset
* Päivitä RAM-muistia, jos käsittelet säännöllisesti suuria tietojoukkoja

### GPU:n käyttö (Chloros+ ja CUDA)

Kun GPU-kiihdytys on käytössä:

* NVIDIA-GPU:n käyttöaste on korkea (60–90 %)
* VRAM-käyttö kasvaa (vaatii vähintään 4 Gt VRAM-muistia)
* Kalibrointivaihe on huomattavasti nopeampi

**Seuranta:**

* NVIDIA-järjestelmäpalkin kuvake
* Tehtävienhallinta → Suorituskyky → GPU
* GPU-Z tai vastaava seurantatyökalu

### Levyn I/O

**Mitä odottaa:**

* Korkea levyn lukunopeus analysointivaiheessa
* Korkea levyn kirjoitusnopeus vientivaiheessa
* SSD on huomattavasti nopeampi kuin HDD

**Suorituskykyvinkki:**

* Käytä projektikansioon SSD:tä, jos mahdollista
* Vältä verkkoasemia suurille tietojoukoille
* Varmista, että levyn kapasiteetti ei ole täynnä (vaikuttaa kirjoitusnopeuteen)

***

## Ongelmien havaitseminen käsittelyn aikana

### Varoitusmerkit

**Käsittely pysähtyy (ei muutosta yli 5 minuuttiin):**

* Tarkista virheet vianmäärityslokista
* Tarkista käytettävissä oleva levytila
* Tarkista Tehtävienhallinnasta, että Chloros on käynnissä

**Virheilmoituksia ilmestyy usein:**

* Keskeytä käsittely ja tarkista virheet
* Yleisiä syitä: levytila, vioittuneet tiedostot, muistiongelmat
* Katso alla oleva Vianmääritys-osio

**Järjestelmä ei vastaa:**

* Chloros+ rinnakkaistila käyttää liikaa resursseja
* Harkitse samanaikaisten tehtävien vähentämistä tai laitteiston päivittämistä
* Vapaa tila kuluttaa vähemmän resursseja

### Milloin käsittely on keskeytettävä

Keskeytä käsittely, jos näet:

* ❌ Virheet &quot;Levy täynnä&quot; tai &quot;Tiedostoa ei voi kirjoittaa&quot;
* ❌ Toistuvat kuvatiedostojen vioittumisvirheet
* ❌ Järjestelmä on täysin jumiutunut (ei vastaa)
* ❌ Huomasit, että asetukset on määritetty väärin
* ❌ Tuotiin vääriä kuvia

**Kuinka lopettaa:**

1. Napsauta**Lopeta/Peruuta-painiketta** (korvaa Käynnistä-painikkeen)
2. Käsittely keskeytyy, edistys menetetään
3. Korjaa ongelmat ja aloita alusta

***

## Vianmääritys käsittelyn aikana

### Käsittely on erittäin hidasta

**Mahdolliset syyt:**

* Merkitsemättömät kohdekuvat (kaikkien kuvien skannaus)
* HDD-tallennustila SSD:n sijaan
* Riittämättömät järjestelmäresurssit
* Määritettyjä hakemistoja on liikaa
* Verkkoaseman käyttö

**Ratkaisut:**

1. Jos käsittely on juuri alkanut ja on tunnistusvaiheessa: Peruuta, merkitse kohteet, aloita alusta
2. Tulevaisuutta varten: Käytä SSD-asemaa, vähennä hakemistoja, päivitä laitteisto
3. Harkitse CLI:ää suurten tietojoukkojen eräkäsittelyyn

### &quot;Levytila&quot;-varoitukset

**Ratkaisut:**

1. Vapauta levytilaa välittömästi
2. Siirrä projekti asemalle, jossa on enemmän tilaa
3. Vähennä vietävien indeksien määrää
4. Käytä JPG-muotoa TIFF:n sijaan (pienemmät tiedostot)

### Toistuvat &quot;Tiedosto vioittunut&quot; -viestit

**Ratkaisut:**

1. Kopioi kuvat uudelleen SD-kortilta varmistaaksesi niiden eheyden
2. Testaa SD-kortti virheiden varalta
3. Poista vioittuneet tiedostot projektista
4. Jatka jäljellä olevien kuvien käsittelyä

### Järjestelmän ylikuumeneminen / tehonrajoitus

**Ratkaisut:**

1. Varmista riittävä ilmanvaihto
2. Puhdista pöly tietokoneen tuuletusaukoista
3. Vähennä käsittelykuormitusta (käytä Free-tilaa Chloros+:n sijaan)
4. Käsittele viileämpään aikaan päivästä

***

## Käsittelyn päättymisilmoitus

Kun käsittely päättyy:

* Edistymispalkki saavuttaa 100 %
* **&quot;Käsittely valmis&quot;** -viesti näkyy virhelokissa
* Käynnistä-painike aktivoituu uudelleen
* Kaikki tulostetiedostot ovat kameramallin alikansiossa

***

## Seuraavat vaiheet

Kun käsittely on valmis:

1. **Tarkista tulokset** - Katso [Käsittelyn viimeistely](finishing-the-processing.md)
2. **Tarkista tulostuskansio** – Varmista, että kaikki tiedostot on viety oikein
3. **Tarkista vianmääritysloki** – Tarkista, onko siinä varoituksia tai virheitä
4. **Esikatsele käsiteltyjä kuvia** – Käytä kuvankatseluohjelmaa tai ulkoista ohjelmistoa

Lisätietoja käsiteltyjen tulosten tarkastelusta ja käytöstä on kohdassa [Käsittelyn loppuun saattaminen](finishing-the-processing.md).
