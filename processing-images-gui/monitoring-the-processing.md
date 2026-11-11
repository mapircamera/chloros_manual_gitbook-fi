# Käsittelyn seuranta

Kun käsittely on alkanut, Chloros

tarjoaa useita tapoja seurata edistymistä, tarkistaa mahdolliset ongelmat ja ymmärtää, mitä aineistollesi tapahtuu. Tällä sivulla kerrotaan, miten voit seurata käsittelyä ja tulkita Chloros

in tarjoamia tietoja.

## Edistymispalkin yleiskatsaus

Yläreunan etupalkissa oleva edistymispalkki näyttää käsittelyn tilan ja valmistumisprosentin reaaliajassa. Edistymistiedot lähetetään suorana taustapalvelimelta Server-Sent Events (SSE) -protokollan kautta, joten palkki heijastaa sitä, mitä käsittelyputki kulloinkin tekee.

### Ilmaistilan edistymispalkki

Käyttäjille, joilla ei ole Chloros

+ -lisenssiä:

**2-vaiheinen edistymisnäkymä:**

1.**Kohteiden tunnistus** – Kalibrointikohteiden etsiminen kuvista
2. **Käsittely** – Korjausten soveltaminen ja vienti**Edistymispalkki näyttää:**

* Kokonaisvalmiusprosentti (0–100 %)
* Nykyisen vaiheen nimi
* Yksinkertainen vaakasuora palkkikuvaus

### Chloros

+:n edistymispalkki

Chloros

+-lisenssin omaaville käyttäjille:

**4-vaiheinen edistymisnäkymä:**

1.**Tunnistus** – Kalibrointikohteiden etsiminen
2. **Analysointi** – Kuvien tarkastelu ja prosessiketjun valmistelu
3. **Kalibrointi** – Vignetoinnin ja heijastavuuden korjausten soveltaminen
4. **Vienti** – Käsiteltyjen tiedostojen tallentaminen**Interaktiiviset ominaisuudet:*** **Vie hiiri** edistymispalkin päälle nähdäksesi laajennetun 4-vaiheisen paneelin
* **Napsauta** edistymispalkkia jäädyttääksesi/kiinnittääksesi laajennetun paneelin
* **Napsauta uudelleen**, jotta paneeli vapautuu ja piiloutuu automaattisesti, kun hiiri viedään pois
* Jokaisessa vaiheessa näkyy sen oma edistymisaste (0–100 %)

{% hint style="info" %}
**Pariteetti CLI

:n kanssa**: `chloros-cli process`-ajon aikana samat neljä säiettä raportoivat tiloina Detecting, Analyzing, Processing ja Exporting, ja `chloros-cli export-status` näyttää reaaliaikaisesti säikeen 4 vientin etenemisen toisesta päätelaitteesta. Katso [CLI

-ohje](../reference/cli-reference.md).
{% endhint %}

***

## Kunkin käsittelyvaiheen ymmärtäminen

{% hint style="info" %}
**Pipeline-arkkitehtuuri**: Nämä neljä GUI-vaihetta vastaavat [4-säikeistä käsittelyputkea](../processing-architecture/processing-pipeline.md). GPU-kiihdytyksellä varustetuissa järjestelmissä säie 3 (Kalibrointi) hyötyy [dynaamisesta laskentasovituksesta](../processing-architecture/dynamic-compute-adaptation.md), joka optimoi käsittelyn juuri sinun laitteistollesi.
{% endhint %}

### Vaihe 1: Tunnistaminen (kohteen tunnistus)

**Mitä tapahtuu:**

* Chloros

skannaa kuvat, jotka olet valinnut Kohde-valintaruudulla (kaikki kuvat, jos yhtään ei ole valittu)
* Tietokoneen näköalgoritmit tunnistavat kalibrointipaneelit
* Jokaisesta paneelista poimitaan heijastusarvot
* Kohteiden aikaleimat tallennetaan oikean kalibrointiaikataulun suunnittelua varten

**Kesto:**

* Merkityt kohteet: 10–60 sekuntia
* Merkitsemättömät kohteet: 5–30+ minuuttia (skannaa kaikki kuvat)

**Edistymisilmaisin:**

* Tunnistaminen: 0 % → 100 %
* Skannattujen kuvien määrä (lasketaan vain ne kuvat, jotka tosiasiallisesti skannataan)
* Löydettyjen kohteiden lukumäärä

**Mitä on syytä huomioida:**

* Pitäisi valmistua nopeasti, jos kohteet on merkitty oikein
* Jos kestää liian kauan, kohteita ei ehkä ole merkitty
* Tarkista vianmäärityslokista ”Target found” -viestit

### Vaihe 2: Analysointi

**Mitä tapahtuu:**

* Kuvien EXIF-metatietojen lukeminen (aikaleimat, valotusasetukset)
* Kalibrointistrategian määrittäminen kohteiden aikaleimojen ja käytettävissä olevien DAQ-alasuuntaisten tietojen perusteella
* Kuvankäsittelyjonon järjestäminen
* Rinnakkaiskäsittelytyöntekijöiden valmistelu (vain Chloros

+)

**Kesto:** 5–30 sekuntia**Edistymisilmaisin:**

* Analysoidaan: 0 % → 100 %
* Nopea vaihe, valmistuu yleensä nopeasti

**Mitä kannattaa seurata:**

* Edistymisen tulisi olla tasaista ilman taukoja
* Varoitukset puuttuvista metatiedoista näkyvät vianmäärityslokissa

### Vaihe 3: Kalibrointi

**Mitä tapahtuu:*** **Debayering**: RAW-Bayer-kuvion muuntaminen 3 kanavaksi (ohitetaan LATTICE-mono-moduuleissa, mistä on huomautus)
* **Vignettikorjaus**: Objektiivin reunoilla esiintyvän tummenemisen poistaminen
* **Heijastavuuden kalibrointi**: Normalisointi tavoitearvojen ja/tai DAQ-tiedon perusteella
* **Indeksin laskeminen**: Monispektristen indeksien laskeminen
* Kunkin kuvan käsittely koko prosessiketjun läpi

**Kesto:** Suurin osa kokonaiskäsittelyajasta (60–80 %)**Edistymisilmaisin:**

* Kalibrointi: 0 % → 100 %
* Käsiteltävänä oleva kuva
* Valmiit kuvat / Kuvien kokonaismäärä

**Käsittelytapa:*** **Vapaa tila**: Käsittelee kuvia yksi kerrallaan peräkkäin
* **Chloros

+ -tila**: Käyttää laitteistoon sopeutuvaa työryhmää — 1–4 samanaikaista työryhmää GPU-järjestelmissä (VRAM:n mukaan), yksi työryhmä fyysistä ydintä kohti (miinus yksi) pelkästään CPU-pohjaisissa järjestelmissä. Katso [Dynaaminen laskentatehon sopeutus](../processing-architecture/dynamic-compute-adaptation.md)
* **GPU-kiihdytys**: Nopeuttaa tätä vaihetta merkittävästi**Mitä kannattaa seurata:**

* Tasainen eteneminen kuvamäärän mukaan
* Tarkista virhelokista kuvakohtaiset valmistumisilmoitukset
* Varoitukset kuvanlaatuun tai kalibrointiongelmiin liittyen

### Vaihe 4: Vienti

**Mitä tapahtuu:**

* Käsitellyt kuvat tallennetaan levylle valitussa muodossa sitä mukaa kuin ne valmistuvat
* **LATTICE**: jokainen kehys jaetaan kaikkiin käytössä oleviin tuotteisiin (debayered / esikatselu / säteily / heijastavuus)
* Monispektristen indeksikuvien vienti LUT-väreillä
* Tulostuspuun `<project>/<camera>/<format>/<Product>_Images/` luominen — viedyt tiedostot säilyttävät lähdetiedoston nimen; kansio tunnistaa tuotteen

**Kesto:** 10–20 % kokonaiskäsittelyajasta**Edistymisilmaisin:**

* Vienti: 0 % → 100 %
* Tiedostoja kirjoitetaan
* Vientimuoto ja kohde

**Mitä on syytä seurata:**

* Varoitukset levytilan loppumisesta
* Tiedostojen kirjoitusvirheet
* Kaikkien määritettyjen tulosteiden valmistuminen

***

## Vianmääritysloki-välilehti

Vianmääritysloki tarjoaa yksityiskohtaista tietoa käsittelyn etenemisestä ja mahdollisista ongelmista. Myös taustaprosessin käynnistysviestit tallennetaan lokikonsoliin, joten loki kertoo koko tarinan, vaikka avaisit sen myöhässä.

### Debug Log -välilehden avaaminen

1. Napsauta vasemmassa sivupalkissa olevaa **Debug Log**-kuvaketta <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">


2. Lokipaneeli avautuu ja näyttää reaaliaikaisia käsittelyviestejä
3. Paneeli vierii automaattisesti näyttääkseen uusimmat viestit

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written), with the 1.3.0 bug icon (Report a bug) visible at the top of the tab -->

### Lokiviestien ymmärtäminen

Chloros

-lokirivien alussa on suluissa oleva tunniste, joka nimeää alijärjestelmän — esimerkiksi `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Tärkein tiedettävä asia on **ajon yhteenveto**, joka tulostetaan jokaisen ajon lopussa (myös keskeytetyt ajot):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Lisäksi `[RUN-SUMMARY]`-vihjeitä seuraa aina, kun jotain on selitettävä — esimerkiksi ajo, joka ei tuottanut mitään, tai kamera, jonka pyydetty tuote ohitettiin soveltumattomana. `[EXPORT-CHECK]`-rivit selittävät kamerakohtaisia ohituksia (esim. miksi RGB

-kamera ei saanut säteilyvoimakkuustuotetta).

Yleiset viestien vakavuustasot (alla olevat esimerkit ovat havainnollistavia, eivät sananmukaisia):

#### Tiedotusviestit (valkoinen/harmaa)

Normaalit käsittelypäivitykset: käsittely aloitettu, kohteet havaittu (paneelien lukumäärineen), kuvakohtaisen kalibroinnin eteneminen, tiedostot viety, käsittely valmis.

#### Varoitusviestit (keltainen)

Ei-kriittisiä ongelmia, jotka eivät keskeytä käsittelyä — esim. puuttuvat GPS-tiedot kehyksessä, suuri aikaleimaväli kohdekuvien välillä tai kalibrointipaneelin heikko kontrasti.

**Toimenpide:** Tarkista varoitukset käsittelyn jälkeen, mutta älä keskeytä sitä

#### Virheilmoitukset (vRed

)

Kriittisiä ongelmia, jotka voivat aiheuttaa käsittelyn epäonnistumisen – esim. levy täynnä, vioittunut kuvatiedosto tai kohteita ei havaittu, kun heijastuskalibrointia pyydettiin.

**Toimenpide:** Keskeytä käsittely, korjaa virhe, käynnistä uudelleen

### Yleisiä lokitilanteita

| Tilanne                             | Merkitys                                       | Tarvittava toimenpide                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Kohde havaittu tiedostossa \[tiedostonimi]        | Kalibrointikohde löydetty onnistuneesti         | Ei toimenpiteitä – normaali                                         |
| Kuvan edistymisviivat              | Nykyisen edistymisen päivitys                       | Ei toimenpiteitä – normaali                                         |
| Kohteita ei löydy                      | Kalibrointikohteita ei havaittu               | Merkitse kohdekuvat tai poista heijastuskalibrointi käytöstä |
| Levytilaa ei riitä               | Tulostustiedostoille ei ole tarpeeksi tallennustilaa                 | Vapauta levytilaa                                    |
| Ohitetaan vioittunut tiedosto               | Kuvatiedosto on vioittunut                         | Kopioi tiedosto uudelleen SD-kortilta                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | Kuvaus ilman raakakuvaa ei voi käsitellä | Ota kuva uudelleen raakakuvalla, tai käytä CLI

:tä `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | Suoritus ei tuottanut kuvatuotteita — raportoitu epäonnistumisena vihjeiden kera | Lue vihjelinjat; tarkista, mitä ohitettiin ja miksi |
| `[PROCESS] refused: The project folder is too deep for this computer…` | Windows

in pitkät polut ovat pois päältä ja projektikansio on liian syvällä 260 merkin polkurajan vuoksi; mitään ei käsitelty | Käytä lyhyempää projektikansiota (viestissä ilmoitetaan pisin sallittu polku) tai ota Windows

in pitkät polut käyttöön ja käynnistä Chloros

uudelleen — katso [Käsittelyn aloittaminen](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer) |
| `[unmix-clamp-mask] …` | LATTICE-tuotteen clamp-maskia ei voitu kirjoittaa; tuote itsessään on kunnossa ja ilmoittaa `Chloros:UnmixClampMask=unavailable` | Tarkista levytila ja kansioiden käyttöoikeudet, ja käsittele sitten uudelleen, jos tarvitset maskia |

### Lokitietojen kopioiminen

Kopioi loki vianmääritystä tai tukea varten seuraavasti:

1. Avaa Debug Log -paneeli
2. Napsauta **Copy**-painiketta (kopioi lokin leikepöydälle) tai**Download**-painiketta tallentaaksesi sen tekstitiedostona
3. Liitä tekstitiedostoon tai sähköpostiin
4. Lähetä tarvittaessa MAPIR

-tukeen

### Virheen ilmoittaminen lokivälilehdeltä

Uutta versiossa 1.3.0: Debug Log -välilehden yläosassa oleva virhekuvake (**Ilmoita virhe**) lähettää raportin suoraan MAPIR

ille.

1. Napsauta Debug Log -välilehden yläosassa olevaa virhekuvaketta
2. Kuvaile ongelmaa: mitä olit tekemässä, mitä odotit tapahtuvan ja mitä sen sijaan tapahtui (pakollinen)
3. Valitse halutessasi **Lisää muutama kuva (enintään 10)**ja valitse kuvat avoimesta projektista tai käytä**Selaa...**-painiketta muiden kuvatiedostojen valitsemiseen
4. Napsauta **Lähetä**

Raportti sisältää kuvauksesi, liittämät kuvat, Chloros

-lokit sekä järjestelmätiedot (Chloros

-versio, käyttöjärjestelmä, GPU ja projektin asetukset). Sinun on oltava kirjautuneena Chloros

iin (Käyttäjä-välilehti), jotta voit lähettää raportin. Voit sulkea ikkunan lähetyksen aikana; kun lähetys on valmis, lokiin kirjoitetaan viitenumero, kuten `Bug report sent. Reference: BR-…` — mainitse se, kun otat yhteyttä MAPIR

:iin. Jos raporttia ei voida ladata (esimerkiksi internetyhteyden puuttuessa), Chloros

tallentaa sen tietokoneellesi ja näyttää polun, jotta voit lähettää sen sähköpostitse osoitteeseen support@mapir.camera.

***

## Järjestelmäresurssien seuranta

### CPU:n käyttö

**Vapaa tila:**

* 1 CPU-ydin käyttöasteella ~100 %
* Muut ytimet lepotilassa tai käytettävissä
* Järjestelmä toimii edelleen sujuvasti

**Chloros

+:n rinnakkaistila:**

* Useat ytimet korkealla käyttöasteella – niiden määrä riippuu [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) -toiminnon valitsemasta strategiasta
* Järjestelmä saattaa tuntua hitaammalta

**Seuranta:**

* Windows

in Tehtävienhallinta (Ctrl+Shift+Esc)
* Suorituskyky-välilehti → CPU-osio
* Etsi ”Chloros

”- tai ”chloros-backend”-prosesseja

### Muistin (RAM) käyttö

**Tyypillinen käyttö:**

* Pienet projektit (&lt; 100 kuvaa): 2–4 GB
* Keskisuuret projektit (100–500 kuvaa): 4–8 GB
* Suuret projektit (yli 500 kuvaa): 8–16 GB
* Chloros

+:n rinnakkaistila kuluttaa enemmän RAM-muistia

**Jos muistia on vähän:**

* Käsittele pienempiä eräkoot
* Sulje muut sovellukset
* Päivitä RAM-muistia, jos käsittelet säännöllisesti suuria aineistoja

### GPU:n käyttö (Chloros

+ ja CUDA)

Kun GPU-kiihdytys on käytössä:

* NVIDIA-GPU:n käyttöaste on korkea (60–90 %)
* VRAM-muistin käyttö kasvaa (vaatii vähintään 4 GB VRAM-muistia; vähintään 7 GB samanaikaista Texture Aware -debayering-käsittelyä varten)
* Kalibrointivaihe on huomattavasti nopeampi

**Seurattavat kohteet:**

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
* Varmista, että levytila ei ole lähes täynnä (vaikuttaa kirjoitusnopeuteen)

***

## Ongelmien havaitseminen käsittelyn aikana

### Varoitusmerkit

**Käsittely pysähtyy (ei muutosta yli 5 minuuttiin):**

* Tarkista virheet vianmäärityslokista
* Tarkista käytettävissä oleva levytila
* Tarkista Tehtävienhallinnasta, että Chloros

-ohjelma on käynnissä

**Virheilmoituksia ilmenee usein:**

* Keskeytä käsittely ja tarkista virheet
* Yleisiä syitä: levytilan puute, vioittuneet tiedostot, muistiongelmat
* Katso alla oleva Vianmääritys-osio

**Järjestelmä lakkaa vastaamasta:**

* Chloros

in rinnakkaistila käyttää liikaa resursseja
* Harkitse samanaikaisten tehtävien vähentämistä tai laitteiston päivittämistä
* Vapaa tila kuluttaa vähemmän resursseja

### Milloin käsittely on keskeytettävä

Keskeytä käsittely, jos huomaat:

* ❌ ”Levy täynnä” tai ”Tiedostoa ei voi kirjoittaa” -virheet
* ❌ Toistuvat kuvatiedostojen vioittumisvirheet
* ❌ Järjestelmä on täysin jumiutunut (ei vastaa)
* ❌ Huomaat, että asetukset on määritetty väärin
* ❌ Väärät kuvat on tuotu

**Kuinka keskeyttää:**

1. Napsauta**Stop-painiketta** (korvaa Start-painikkeen) — kerran riittää
2. Palkissa näkyy teksti ”Stopping...” (Keskeytetään...), kunnes keskeneräinen kuva on valmis, minkä jälkeen käsittely päättyy keskeytettyyn tilaan
3. Jo viedyt tuotteet säilyvät levyllä; lokiin tulostuu rehellinen `[RUN-SUMMARY]`-raportti siitä, mitä on saatu valmiiksi
4. Korjaa ongelmat ja käynnistä uudelleen — käsittely alkaa alusta

***

## Vianmääritys käsittelyn aikana

### Käsittely on erittäin hidasta

**Mahdollisia syitä:**

* Merkitsemättömät kohdekuvat (kaikkien kuvien skannaus)
* HDD-tallennustila SSD:n sijaan
* Riittämättömät järjestelmäresurssit
* Määritettyjä indeksejä on liikaa
* Verkkoaseman käyttö

**Ratkaisut:**

1. Jos käsittely on juuri alkanut ja on tunnistusvaiheessa: Pysäytä, merkitse kohteet ja käynnistä uudelleen
2. Tulevaisuutta varten: Käytä SSD-asemaa, vähennä indeksejä, päivitä laitteistoa
3. Harkitse CLI

:n käyttöä suurten tietojoukkojen eräkäsittelyssä

### ”Levytila”-varoitukset

**Ratkaisut:**

1. Vapauta levytilaa välittömästi
2. Siirrä projekti levylle, jossa on enemmän tilaa
3. Vähennä vietävien indeksien määrää
4. Poista käytöstä tarpeettomat LATTICE-vientituotteet (Projektin asetukset → Käsittely)
5. Käytä JPG-muotoa TIFF

-muodon sijaan (pienemmät tiedostot)

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
3. Vähennä käsittelykuormitusta (käytä Free-tilaa Chloros

+:n sijaan)
4. Käsittele kuvia päivän viileämpinä aikoina

***

## Ilmoitus käsittelyn päättymisestä

Kun käsittely päättyy:

* Edistymispalkki saavuttaa 100 %
* `[RUN-SUMMARY]`-rivit näkyvät virhelokissa lopullisine lukumäärineen
* Käynnistä-painike aktivoituu uudelleen
* Kaikki tulostustiedostot ovat projektin kamerakohtaisessa tulostushakemistossa: `<project>/<camera>/<format>/<Product>_Images/`

***

## Seuraavat vaiheet

Kun käsittely on valmis:

1. **Tarkista tulokset** – Katso [Käsittelyn viimeistely](finishing-the-processing.md)
2. **Tarkista tulostuskansio** – Varmista, että kaikki tiedostot on viety oikein
3. **Tarkista vianmääritysloki** – Tarkista, onko siinä varoituksia tai virheitä
4. **Esikatsele käsiteltyjä kuvia** – Käytä kuvankatseluohjelmaa tai ulkoista ohjelmistoa

Lisätietoja käsiteltyjen tulosten tarkastelusta ja käytöstä on kohdassa [Käsittelyn loppuun saattaminen](finishing-the-processing.md).
