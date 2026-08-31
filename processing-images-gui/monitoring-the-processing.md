# Käsittelyn seuranta

Kun käsittely on alkanut, Chloros tarjoaa useita tapoja seurata edistymistä, tarkistaa mahdolliset ongelmat ja ymmärtää, mitä aineistollesi tapahtuu. Tällä sivulla selitetään, miten voit seurata käsittelyäsi ja tulkita Chloros:n tarjoamia tietoja.

## Edistymispalkin yleiskatsaus

Yläreunan otsikossa oleva edistymispalkki näyttää käsittelyn tilan reaaliajassa sekä valmistumisprosentin. Edistymistiedot lähetetään reaaliaikaisesti taustajärjestelmästä Server-Sent Events (SSE) -protokollan kautta, joten palkki kuvaa tarkasti, mitä käsittelyputki parhaillaan tekee.

### Ilmaisen version edistymispalkki

Käyttäjille, joilla ei ole Chloros+-lisenssiä:

**2-vaiheinen etenemisen näyttö:**

1.**Kohteen tunnistus** – Kalibrointikohteiden etsiminen kuvista
2. **Käsittely** – Korjausten soveltaminen ja vienti**Edistymispalkki näyttää:**

* Kokonaisvalmiusprosentti (0–100 %)
* Nykyisen vaiheen nimi
* Yksinkertainen vaakasuora palkkikuvaus

### Chloros+ -edistymispalkki

Käyttäjille, joilla on Chloros+-lisenssi:

**4-vaiheinen edistymisnäkymä:**

1.**Tunnistus** – Kalibrointikohteiden etsiminen
2. **Analysointi** – Kuvien tarkastelu ja käsittelyputken valmistelu
3. **Kalibrointi** – Vignetoinnin ja heijastavuuden korjausten soveltaminen
4. **Vienti** – Käsiteltyjen tiedostojen tallentaminen**Interaktiiviset ominaisuudet:*** **Vie hiiri** edistymispalkin päälle nähdäksesi laajennetun 4-vaiheisen paneelin
* **Napsauta** etenemispalkkia jäädytääksesi/kiinnittääksesi laajennetun paneelin
* **Napsauta uudelleen** vapauttaaksesi kiinnityksen ja piilottaaksesi paneelin automaattisesti, kun hiiri viedään pois
* Jokaisessa vaiheessa näkyy sen oma eteneminen (0–100 %)

{% hint style="info" %}
**CLI-pariteetti**: `chloros-cli process`-ajon aikana samat neljä säiettä raportoivat olevansa tunnistamassa, analysoimassa, Processing, Exporting, ja `chloros-cli export-status` näyttää reaaliaikaisen Thread-4-vientin etenemisen toiselta päätelaitteelta. Katso [CLI-viite](../reference/cli-reference.md).
{% endhint %}

***

## Kunkin käsittelyvaiheen ymmärtäminen

{% hint style="info" %}
**Pipeline-arkkitehtuuri**: Nämä neljä GUI-vaihetta vastaavat [4-säikeistä käsittelyputkea](../processing-architecture/processing-pipeline.md). GPU-kiihdytyksellä varustetuissa järjestelmissä säie 3 (Kalibrointi) hyötyy [dynaamisesta laskentasovituksesta](../processing-architecture/dynamic-compute-adaptation.md), joka optimoi käsittelyn käyttämäsi laitteiston mukaan.
{% endhint %}

### Vaihe 1: Tunnistus (kohteen tunnistus)

**Mitä tapahtuu:**

* Chloros skannaa kuvat, jotka olet valinnut Kohde-valintaruudulla (kaikki kuvat, jos yhtään ei ole valittu)
* Tietokonenäköalgoritmit tunnistavat kalibrointipaneelit
* Kustakin paneelista poimitaan heijastusarvot
* Kohteiden aikaleimat tallennetaan kalibroinnin oikean ajoituksen varmistamiseksi

**Kesto:**

* Merkittyjen kohteiden kanssa: 10–60 sekuntia
* Ilman merkittyjä kohteita: 5–30+ minuuttia (skannaa kaikki kuvat)

**Edistymisilmaisin:**

* Tunnistaminen: 0 % → 100 %
* Skannattujen kuvien lukumäärä (lasketaan vain ne kuvat, jotka tosiasiallisesti skannataan)
* Löydettyjen kohteiden lukumäärä

**Mitä kannattaa tarkkailla:**

* Pitäisi valmistua nopeasti, jos kohteet on merkitty oikein
* Jos kestää liian kauan, kohteita ei ehkä ole merkitty
* Tarkista vianmäärityslokista ”Target found” -viestit

### Vaihe 2: Analysointi

**Mitä tapahtuu:**

* Kuvien EXIF-metatietojen lukeminen (aikaleimat, valotusasetukset)
* Kalibrointistrategian määrittäminen kohteiden aikaleimojen ja käytettävissä olevien DAQ-alasvirtaustietojen perusteella
* Kuvankäsittelyjonon järjestäminen
* Rinnakkaiskäsittelytyöntekijöiden valmistelu (vain Chloros+)

**Kesto:** 5–30 sekuntia**Edistymisilmaisin:**

* Analysoidaan: 0 % → 100 %
* Nopea vaihe, valmistuu yleensä nopeasti

**Mitä on syytä seurata:**

* Edistymisen tulisi olla tasaista ilman taukoja
* Varoitukset puuttuvista metatiedoista näkyvät virhelokissa

### Vaihe 3: Kalibrointi

**Mitä tapahtuu:*** **Debayering**: RAW-Bayer-kuvion muuntaminen 3 kanavaksi (ohitetaan LATTICE-mono-moduuleissa, mistä on huomautus)
* **Vignettikorjaus**: Objektiivin reunoilla esiintyvän tummenemisen poistaminen
* **Heijastavuuden kalibrointi**: Normalisointi kohdearvojen ja/tai DAQ-alasvirtauksen avulla
* **Indeksin laskeminen**: Monispektristen indeksien laskeminen
* Kunkin kuvan käsittely koko prosessiketjun läpi

**Kesto:** Suurin osa kokonaiskäsittelyajasta (60–80 %)**Edistymisilmaisin:**

* Kalibrointi: 0 % → 100 %
* Käsiteltävä kuva parhaillaan
* Valmiit kuvat / Kuvien kokonaismäärä

**Käsittelytapa:*** **Vapaa tila**: Käsittelee kuvia yksi kerrallaan peräkkäin
* **Chloros+-tila**: Käyttää laitteistoon sopeutuvaa työryhmäpoolia — 1–4 samanaikaista työryhmää GPU-järjestelmissä (VRAM:n mukaan), yksi työryhmä fyysistä ydintä kohti (miinus yksi) pelkästään CPU-pohjaisissa järjestelmissä. Katso [Dynaaminen laskentakapasiteetin mukautus](../processing-architecture/dynamic-compute-adaptation.md)
* **GPU-kiihdytys**: Nopeuttaa tätä vaihetta merkittävästi**Mitä kannattaa seurata:**

* Tasainen eteneminen kuvamäärän mukaan
* Tarkista virhelokista kuvakohtaiset valmistumisilmoitukset
* Varoitukset kuvanlaadusta tai kalibrointiongelmista

### Vaihe 4: Vienti

**Mitä tapahtuu:**

* Käsiteltyjen kuvien kirjoittaminen levylle valitussa muodossa niiden valmistuessa
* **LATTICE**: jokainen kehys jaetaan kaikkiin käytössä oleviin tuotteisiin (debayered / esikatselu / säteily / heijastavuus)
* Monispektristen indeksikuvien vienti LUT-väreillä
* Tulostuspuun `<project>/<camera>/<format>/<Product>_Images/` luominen — viedyt tiedostot säilyttävät lähdetiedoston nimen; kansio tunnistaa tuotteen

**Kesto:** 10–20 % kokonaiskäsittelyajasta**Edistymisilmaisin:**

* Vienti: 0 % → 100 %
* Tiedostoja tallennetaan
* Vientimuoto ja kohde

**Mitä on syytä seurata:**

* Levytilan varoitukset
* Tiedostojen kirjoitusvirheet
* Kaikkien määritettyjen tulosteiden valmistuminen

***

## Vianmääritysloki-välilehti

Vianmääritysloki tarjoaa yksityiskohtaista tietoa käsittelyn etenemisestä ja mahdollisista ongelmista. Myös taustapalvelimen käynnistysviestit tallennetaan lokikonsoliin, joten loki kertoo koko tarinan, vaikka avaisit sen myöhässä.

### Debug Log -välilehden avaaminen

1. Napsauta vasemmassa sivupalkissa olevaa **Debug Log** -kuvaketta<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">


2. Lokipaneeli avautuu ja näyttää reaaliaikaisia käsittelyviestejä
3. Paneeli vierii automaattisesti näyttämään uusimmat viestit

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written) -->

### Lokiviestien ymmärtäminen

Chloros-lokirivien alussa on suluissa oleva tunniste, joka nimeää alijärjestelmän — esimerkiksi `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Tärkein tiedettävä asia on **ajon yhteenveto**, joka tulostetaan jokaisen ajon lopussa (myös keskeytetyt ajot):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Lisävihjeitä sisältäviä `[RUN-SUMMARY]`-rivejä seuraa aina, kun jotain on selitettävä — esimerkiksi suoritus, joka ei tuottanut tulosta, tai kamera, jonka pyydetty tuote ohitettiin soveltumattomana. `[EXPORT-CHECK]`-rivit selittävät kamerakohtaisia ohituksia (esim. miksi RGB-kamera ei saanut säteilyvoimakkuustuotetta).

Viestien yleiset vakavuusluokat (alla olevat esimerkit ovat havainnollistavia, eivät sanatarkkoja):

#### Tiedotusviestit (valkoinen/harmaa)

Normaalit käsittelypäivitykset: käsittely aloitettu, kohteet havaittu (paneelien lukumäärineen), kuvakohtaisen kalibroinnin eteneminen, tiedostot viety, käsittely valmis.

#### Varoitusviestit (keltainen)

Ei-kriittisiä ongelmia, jotka eivät keskeytä käsittelyä — esim. puuttuvat GPS-tiedot kehyksessä, suuri aikaleimaväli kohdekuvien välillä tai kalibrointipaneelin heikko kontrasti.

**Toimenpide:** Tarkista varoitukset käsittelyn jälkeen, mutta älä keskeytä käsittelyä

#### Virheilmoitukset (Red)

Kriittisiä ongelmia, jotka voivat aiheuttaa käsittelyn epäonnistumisen – esim. levy täynnä, vioittunut kuvatiedosto tai kohteita ei havaittu, kun heijastavuuskalibrointia pyydettiin.

**Toimenpide:** Keskeytä käsittely, korjaa virhe ja käynnistä uudelleen

### Yleisiä lokitilanteita

| Tilanne                             | Merkitys                                       | Tarvittava toimenpide                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Kohde havaittu tiedostossa \[tiedostonimi]        | Kalibrointikohde löydetty onnistuneesti         | Ei toimenpiteitä – normaali                                         |
| Kuvan edistymisviivat              | Nykyisen edistymisen päivitys                       | Ei toimenpiteitä – normaali                                         |
| Kohteita ei löydetty                      | Kalibrointikohteita ei havaittu               | Merkitse kohdekuvat tai poista heijastuskalibrointi käytöstä |
| Levytilaa ei riitä               | Tulosteille ei ole tarpeeksi tallennustilaa                 | Vapauta levytilaa                                    |
| Ohitetaan vioittunut tiedosto               | Kuvatiedosto on vioittunut                         | Kopioi tiedosto uudelleen SD-kortilta                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | Raakakuvaa sisältämätöntä tallennusta ei voi käsitellä | Ota kuva uudelleen raakakuva mukana tai käytä CLI `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | Suoritus ei tuottanut kuvatuotteita — raportoitu epäonnistumisena vihjeiden kera | Lue vihjelinjat; tarkista, mitä ohitettiin ja miksi |

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

* 1 CPU-ydin käyttää ~100 %
* Muut ytimet ovat lepotilassa tai käytettävissä
* Järjestelmä reagoi edelleen

**Chloros+ Rinnakkaistila:**

* Useat ytimet korkealla käyttöasteella — lukumäärä riippuu [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) -toiminnon valitsemasta strategiasta
* Järjestelmä saattaa tuntua hitaammalta

**Seuranta:**

* Windows Tehtävienhallinta (Ctrl+Shift+Esc)
* Suorituskyky-välilehti → CPU-osio
* Etsi prosesseja nimeltä &quot;Chloros&quot; tai &quot;chloros-backend&quot;

### Muistin (RAM) käyttö

**Tyypillinen käyttö:**

* Pienet projektit (&lt; 100 kuvaa): 2–4 GB
* Keskisuuret projektit (100–500 kuvaa): 4–8 GB
* Suuret projektit (yli 500 kuvaa): 8–16 GB
* Chloros+ -rinnakkaistilassa käytetään enemmän RAM-muistia

**Jos muistia on vähän:**

* Käsittele pienempiä eräkoot
* Sulje muut sovellukset
* Päivitä RAM-muistia, jos käsittelet säännöllisesti suuria aineistoja

### GPU:n käyttö (Chloros+ ja CUDA)

Kun GPU-kiihdytys on käytössä:

* NVIDIA-grafiikkaprosessorin käyttöaste on korkea (60–90 %)
* VRAM-muistin käyttö kasvaa (vaatii vähintään 4 GB VRAM-muistia; vähintään 7 GB samanaikaista Texture Aware -debayering-käsittelyä varten)
* Kalibrointivaihe on huomattavasti nopeampi

**Seurattavaa:**

* NVIDIA-järjestelmäpalkin kuvake
* Tehtävienhallinta → Suorituskyky → GPU
* GPU-Z tai vastaava seurantatyökalu

### Levyn I/O

**Mitä odottaa:**

* Korkea levyn lukunopeus analysointivaiheessa
* Korkea levyn kirjoitusnopeus vientivaiheessa
* SSD on huomattavasti nopeampi kuin HDD

**Suorituskykyvinkki:**

* Käytä projektikansioon SSD-levyä, jos mahdollista
* Vältä verkkoasemia suurille tietojoukoille
* Varmista, että levyn tila ei ole lähes täynnä (vaikuttaa kirjoitusnopeuteen)

***

## Ongelmien havaitseminen käsittelyn aikana

### Varoitusmerkit

**Käsittely pysähtyy (ei muutosta yli 5 minuuttiin):**

* Tarkista virheet vianmäärityslokista
* Varmista, että levytilaa on käytettävissä
* Tarkista Tehtävienhallinnasta, että Chloros on käynnissä

**Virheilmoituksia ilmestyy usein:**

* Keskeytä käsittely ja tarkista virheet
* Yleisiä syitä: levytila, vioittuneet tiedostot, muistiongelmat
* Katso alla oleva Vianmääritys-osio

**Järjestelmä lakkaa vastaamasta:**

* Chloros+ rinnakkaistila käyttää liikaa resursseja
* Harkitse samanaikaisten tehtävien vähentämistä tai laitteiston päivittämistä
* Vapaa tila kuluttaa vähemmän resursseja

### Milloin käsittely on keskeytettävä

Keskeytä käsittely, jos havaitset:

* ❌ ”Levy täynnä” tai ”Tiedostoa ei voi kirjoittaa” -virheet
* ❌ Toistuvat kuvatiedostojen vioittumisvirheet
* ❌ Järjestelmä on täysin jumiutunut (ei vastaa)
* ❌ Huomaat, että asetukset on määritetty väärin
* ❌ Väärät kuvat on tuotu

**Kuinka keskeyttää:**

1. Napsauta**Stop-painiketta** (korvaa Start-painikkeen) — kerran riittää
2. Palkissa näkyy ”Stopping...” (Keskeytetään...), kunnes keskeneräinen kuva valmistuu, minkä jälkeen käsittely päättyy keskeytettyyn tilaan
3. Jo viedyt tuotteet jäävät levylle; lokiin tulostuu rehellinen `[RUN-SUMMARY]`-raportti siitä, mitä on saatu valmiiksi
4. Korjaa ongelmat ja käynnistä uudelleen — käsittely alkaa alusta

***

## Vianmääritys käsittelyn aikana

### Käsittely on erittäin hidasta

**Mahdollisia syitä:**

* Merkitsemättömät kohdekuvat (kaikkien kuvien skannaus)
* HDD-tallennustila SSD:n sijaan
* Riittämättömät järjestelmäresurssit
* Määritettyjä hakemistoja on liikaa
* Verkkoaseman käyttö

**Ratkaisut:**

1. Jos käsittely on juuri alkanut ja on tunnistusvaiheessa: Pysäytä, merkitse kohteet, käynnistä uudelleen
2. Tulevaisuutta varten: Käytä SSD-levyä, vähennä hakemistojen määrää, päivitä laitteisto
3. Harkitse CLI-ohjelmaa suurten tietojoukkojen eräkäsittelyyn

### ”Levytila”-varoitukset

**Ratkaisut:**

1. Vapauta levytilaa välittömästi
2. Siirrä projekti levylle, jossa on enemmän tilaa
3. Vähennä vietävien indeksien määrää
4. Poista käytöstä tarpeettomat LATTICE-vientituotteet (Projektin asetukset → Käsittely)
5. Käytä JPG-muotoa TIFF:n sijaan (pienemmät tiedostot)

### Toistuvat ”Tiedosto vioittunut” -viestit

**Ratkaisut:**

1. Kopioi kuvat uudelleen SD-kortilta varmistaaksesi niiden eheyden
2. Tarkista SD-kortti virheiden varalta
3. Poista vioittuneet tiedostot projektista
4. Jatka jäljellä olevien kuvien käsittelyä

### Järjestelmän ylikuumeneminen / suorituskyvyn rajoittuminen

**Ratkaisut:**

1. Varmista riittävä ilmanvaihto
2. Puhdista pöly tietokoneen tuuletusaukoista
3. Vähennä prosessointikuormitusta (käytä Free-tilaa Chloros+:n sijaan)
4. Suorita käsittely päivän viileämpinä aikoina

***

## Ilmoitus käsittelyn päättymisestä

Kun käsittely päättyy:

* Edistymispalkki saavuttaa 100 %
* `[RUN-SUMMARY]`-rivit näkyvät virheenkorjauslokissa lopullisine lukumäärineen
* Käynnistä-painike aktivoituu jälleen
* Kaikki tulostustiedostot ovat projektin kamerakohtaisessa tulostuspuussa: `<project>/<camera>/<format>/<Product>_Images/`

***

## Seuraavat vaiheet

Kun käsittely on valmis:

1. **Tarkista tulokset** – Katso [Käsittelyn viimeistely](finishing-the-processing.md)
2. **Tarkista tulostuskansio** – Varmista, että kaikki tiedostot on viety oikein
3. **Tarkista virheloki** – Tarkista, onko siinä varoituksia tai virheitä
4. **Esikatsele käsiteltyjä kuvia** – Käytä kuvankatseluohjelmaa tai ulkoista ohjelmistoa

Lisätietoja käsiteltyjen tulosten tarkastelusta ja käytöstä on kohdassa [Käsittelyn loppuun saattaminen](finishing-the-processing.md).
