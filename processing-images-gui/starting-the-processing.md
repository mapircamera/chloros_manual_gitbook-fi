# Käsittelyn aloittaminen

Kun olet tuonut kuvat, merkinnyt kalibrointikohteet ja määrittänyt projektin asetukset, olet valmis aloittamaan käsittelyn. Tällä sivulla opastetaan sinua Chloros-käsittelyputken käynnistämisessä.

## Esikäsittelyn tarkistuslista

Ennen kuin napsautat Käynnistä-painiketta, varmista, että kaikki on valmiina:

* [ ] **Tiedostot tuotu** – Kaikki kuvat näkyvät tiedostoselaimessa
* [ ] **Kohdekuvat merkitty** – Kohde-sarakkeessa on valittu kalibrointikuvat (tai LATTICE-ohjelmistoon on tuotu `.daq`-tallenne)
* [ ] **Kameramallit tunnistettu** – Kameramalli-sarakkeessa näkyvät oikeat kamerat
* [ ] **Asetukset määritetty** – Projektin asetukset tarkistettu ja mukautettu
* [ ] **Indeksit valittu** – Halutut monispektriset indeksit lisätty (tarvittaessa)
* [ ] **Vientimuoto valittu** – Työnkulkuusi sopiva tulostusmuoto

{% hint style="info" %}
**Vinkki**: Selaa muutamia kuvia tiedostoselaimessa varmistaaksesi, että ne ovat latautuneet oikein ennen käsittelyä.
{% endhint %}

***

## Käsittelyn aloittaminen

### Etsi Käynnistä-painike

Käynnistä/Toista-painike sijaitsee Chloros:n yläreunan palkissa:

* Sijainti: Ikkunan yläosassa keskellä
* Kuvake: **Toista/Käynnistä-painike** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Tila: Painike on käytössä (valaistu), kun käsittely on valmis

### Aloita napsauttamalla

1. Napsauta yläpalkissa olevaa **Toisto-/Käynnistyspainiketta**

2. Käsittely alkaa välittömästi
3. Painike muuttuu **Pysäytyspainikkeeksi** käsittelyn aikana
4. Edistymispalkki päivittyy ja näyttää käsittelyn tilan

{% hint style="success" %}
**Käsittely aloitettu**: Kun painiketta on napsautettu, Chloros hoitaa automaattisesti kaikki käsittelyvaiheet – kohteen tunnistuksen, debayering, kalibrointi, indeksin laskeminen ja vienti. Se tunnistaa automaattisesti, onko projektisi Survey3-, LATTICE- vai sekaprojekti, ja soveltaa oikeaa käsittelyputkea kuhunkin kameraan.
{% endhint %}

***

## Käsittelytilojen ymmärtäminen

Chloros toimii kahdessa eri käsittelytilassa lisenssistäsi riippuen:

### Ilmainen tila (peräkkäinen käsittely)

**Kaikkien käyttäjien käytettävissä**

**Kuinka se toimii:**

* Käsittelee kuvia yksi kerrallaan, peräkkäin
* Yksisäikeinen toiminta
* Pienempi muistin käyttö

**Edistymispalkki näyttää 2 vaihetta:**

1.**Kohteen tunnistus** – Kalibrointikohteiden etsiminen
2. **Käsittely** – Kalibroinnin soveltaminen ja kuvien vienti**Käsittelyaika:**

* Paljon hitaampi kuin Chloros+ -rinnakkaistilassa
* Sopii pienille ja keskisuurille aineistoille (&lt; 200 kuvaa)

### Chloros+ -tila (rinnakkaiskäsittely)

**Vaatii Chloros+ -lisenssin**

**Kuinka se toimii:**

* Käsittelee useita kuvia samanaikaisesti käyttäen [4-säikeistä käsittelyputkea](../processing-architecture/processing-pipeline.md)
* [Dynaaminen laskentasovitus](../processing-architecture/dynamic-compute-adaptation.md) valitsee automaattisesti laitteistollesi optimaalisen strategian suorituksen alkaessa
* GPU (CUDA)-kiihdytys NVIDIA-näytönohjaimilla (pöytätietokone ja Jetson)
* **Työntekijöiden lukumäärä mukautuu laitteistoon**: GPU-strategioissa käytetään**1–4 samanaikaista työntekijää** (skaalataan VRAM-muistin mukaan — vähämuistinen Jetson käyttää yhtä, yli 12 GB:n pöytätietokoneen GPU jopa neljää); pelkästään CPU:ta käyttävissä järjestelmissä käytetään yhtä työntekijää fyysistä ydintä kohti, miinus yksi**Edistymispalkki näyttää 4 vaihetta** (vastaavat 4:ää putkistoketjua):

1. **Tunnistaminen** (ketju 1) – Kalibrointikohteiden etsiminen
2. **Analysointi** (ketju 2) – Kuvan metatietojen tarkastelu ja kalibroinnin laskeminen
3. **Kalibrointi** (säie 3) – Debayering, vinjetoinnin korjaus, kalibrointi, indeksin laskeminen
4. **Vienti** (säie 4) – Käsiteltyjen kuvien ja indeksien tallentaminen**Edistymispalkin käyttö:*** **Vie hiiri** palkin päälle nähdäksesi yksityiskohtaisen nelivaiheisen pudotusvalikon
* **Napsauta** etenemispalkkia pysäyttääksesi pudotusvalikon paikoilleen
* **Napsauta uudelleen** vapauttaaksesi ja piilottaaksesi valikon**Käsittelyaika:**

* Huomattavasti nopeampi kuin ilmainen tila
* GPU-kiihdytys parantaa nopeutta entisestään

{% hint style="info" %}
**Chloros+ Nopeus**: Rinnakkaiskäsittely voi olla 5–10 kertaa nopeampaa kuin peräkkäiskäsittely suurilla aineistoilla. 500 kuvan projekti, joka kestää ilmaisversiossa 2 tuntia, voi valmistua 15–20 minuutissa Chloros+:n avulla.
{% endhint %}

***

## Mitä käsittelyn aikana tapahtuu

### Vaihe 1: Kohteiden tunnistus

**Mitä Chloros tekee:**

* Skannaa kuvat, jotka olet valinnut Kohde-sarakkeessa (kaikki kuvat, jos yhtään ei ole valittu)
* Tunnistaa kunkin kohteen kalibrointipaneelit
* Poimii heijastusarvot kohdepaneeleista
* Tallentaa kohteiden aikaleimat kalibroinnin aikatauluttamista varten

**Kesto:** 1–30 sekuntia (merkityt kohteet), 5–30+ minuuttia (merkitsemättömät)

### Vaihe 2: Debayering (RAW-muunnos)

**Chloros:n toiminta:**

* Muuntaa RAW-muotoiset Bayer-kuviotiedot täysimittaisiksi 3-kanavaisiksi kuviksi (LATTICE-mono-moduulit pysyvät yksikaistaisina — niiden osalta debayering ohitetaan ja siitä tehdään merkintä lokiin)
* Soveltaa valittua demosaicing-algoritmia
* Säilyttää kuvan laadun ja yksityiskohdat mahdollisimman hyvin

**Kesto:** Vaihtelee kuvamäärän ja CPU/GPU:n nopeuden mukaan

### Vaihe 3: Kalibrointi

**Chloros:n toiminta:*** **Vignettikorjaus**: Poistaa objektiivin aiheuttaman tummenemisen reunoilta
* **Heijastavuuden kalibrointi**: Normalisoi käyttämällä kohdeheijastavuusarvoja ja/tai DAQ:n alaspäin suuntautuvaa dataa
* Soveltaa korjauksia kaikkiin kaistoihin/kanaviin
* Käyttää kullekin kuvalle sopivaa kalibrointiviitettä aikaleiman perusteella

**Kesto:** Suurin osa käsittelyajasta

### Vaihe 4: Indeksin laskeminen

**Mitä Chloros tekee:**

* Laskee määritetyt monispektriset indeksit (NDVI, NDRE jne.)
* Soveltaa kaistamatematiikkaa kalibroituihin kuviin
* Luo indeksikuvat jokaiselle valitulle indeksille

**Kesto:** Muutama sekunti kuvaa kohti

### Vaihe 5: Vienti

**Mitä Chloros tekee:**

* Tallentaa käsitellyt kuvat valitussa muodossa
* **LATTICE-fan-out**: jokainen raaka LATTICE-kuvakehys viedään yhtenä kerrana kaikkina käytössä olevina tuotteina — debayeroitu, esikatselu, säteilyvoimakkuus (aina float32), heijastavuus
* Kirjoittaa tiedostot projektin tulostuspuuhun: `<project>/<camera>/<format>/<Product>_Images/`
* **Säilyttää lähdetiedoston nimen** — kansio tunnistaa tuotteen, eikä tiedostotunnistetta lisätä**Kesto:** Vaihtelee vientimuodon ja tiedostokoon mukaan***

## Käsittelyn toiminta

### Automaattinen käsittelyputki

Kun käsittely on käynnistetty, koko käsittelyputki toimii automaattisesti:

* Käyttäjän toimia ei tarvita
* Kaikki määritetyt vaiheet suoritetaan järjestyksessä
* Edistymispäivitykset näkyvät reaaliajassa
* Viedyt tiedostot tallennetaan levylle heti, kun ne valmistuvat — voit avata valmiit tulokset jo ennen kuin käsittely on päättynyt

### Tietokoneen käyttö käsittelyn aikana

**Vapaa tila:**

* Suhteellisen alhainen CPU-kuormitus (yksisäikeinen)
* Tietokone pysyy reagoivana muihin tehtäviin
* Chloros-ohjelman voi turvallisesti minimoida ja työskennellä muissa sovelluksissa

**Chloros+ rinnakkaistila:**

* Suuri prosessorin kuormitus strategian työryhmässä
* GPU-kiihdytyksen kanssa: suuri GPU:n kuormitus
* Tietokone saattaa reagoida hitaammin käsittelyn aikana
* Vältä muiden prosessoria kuormittavien tehtävien käynnistämistä

{% hint style="warning" %}
**Suorituskykyvinkki**: Parhaan Chloros+ suorituskyvyn saavuttamiseksi sulje muut sovellukset ja anna Chloros:n käyttää järjestelmän resursseja täysimääräisesti.
{% endhint %}

### Käsittelyä ei voi keskeyttää (mutta pysäyttäminen on siistiä)

* Kun käsittely on alkanut, sitä ei voi keskeyttää ja jatkaa myöhemmin
* **Pysäytä**-painikkeen napsauttaminen pysäyttää ajon siististi jo ensimmäisellä napsautuksella
* Ennen pysäyttämistä jo viedyt tuotteet jäävät levylle
* Pysäytetty käsittely raportoi rehellisesti, mitä se on saanut valmiiksi (katso lokin `[RUN-SUMMARY]`-rivit)
* Uusi käsittely käynnistää prosessin alusta alkaen

**Suunnitteluvinkki:** Erittäin suurissa projekteissa kannattaa harkita käsittelyä erissä tai CLI-komennon käyttöä paremman hallinnan saavuttamiseksi.***

## Käsittelyn seuranta

Käsittelyn ollessa käynnissä voit:

* **Seurata edistymispalkkia** – Näet kokonaisvalmiusprosentin
* **Tarkastella nykyistä vaihetta** – Tunnistus, analysointi, kalibrointi tai vienti
* **Tarkistaa loki-välilehden** – Katso yksityiskohtaiset käsittelyviestit ja varoitukset
* **Esikatsella valmiita kuvia** – Vientitiedostot näkyvät levyllä käsittelyn aikana

Yksityiskohtaisia tietoja seurannasta on kohdassa [Käsittelyn seuranta](monitoring-the-processing.md).

***

## Käsittelyn pysäyttäminen

Jos haluat pysäyttää käsittelyn:

### Pysäyttämisohjeet

1. Etsi **Pysäytä-painike** (korvaa Käynnistä-painikkeen käsittelyn aikana)
2. Napsauta sitä kerran — palkissa näkyy **”Pysäytetään...”**, kunnes keskeneräinen kuva on valmis
3. Suoritus päättyy lopullisesti pysäytettyyn tilaan ja lokiin tulostuu tarkka `[RUN-SUMMARY]`-raportti siitä, mitä on saatu valmiiksi

### Milloin keskeyttää

**Hyväksyttäviä syitä keskeyttämiseen:**

* Huomattiin, että asetukset olivat virheelliset
* Unohdettiin merkitä kohdekuvia
* Tuotiin vääriä kuvia
* Järjestelmä toimii liian hitaasti tai ei vastaa

**Keskeyttämisen jälkeen:**

* Ennen keskeyttämistä viedyt tuotteet säilyvät levyllä
* Tarkista ja korjaa mahdolliset ongelmat, säädä asetuksia tarpeen mukaan
* Käynnistä käsittely uudelleen — ajo alkaa alusta

***

## Arviot käsittelyajasta

Todellinen käsittelyaika vaihtelee suuresti seuraavien tekijöiden mukaan:

* Kuvien lukumäärä
* Kuvan resoluutio
* RAW- vai JPG-tulomuoto
* Käsittelytila (Free vs. Chloros+)
* Prosessorin nopeus ja ydinten lukumäärä
* GPU:n käytettävyys (vain Chloros+)
* Laskettavien indeksien lukumäärä
* Käytössä olevien vientituotteiden lukumäärä (LATTICE)

### Karkeat arviot (Chloros+, 12 MP:n kuvat, nykyaikainen prosessori)

| Kuvien lukumäärä | Ilmainen tila | Chloros+ (prosessori) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 kuvaa   | 15–20 min | 5–8 min        | 3–5 min        |
| 100 kuvaa  | 30–40 min | 10–15 min      | 5–8 min        |
| 200 kuvaa  | 1–1,5 tuntia | 20–30 min      | 10–15 min      |
| 500 kuvaa  | 2–3 tuntia   | 45–60 min      | 20–30 min      |
| 1 000 kuvaa | 4–6 tuntia   | 1,5–2 tuntia      | 40–60 min      |

{% hint style="info" %}
**Ensimmäinen käyttökerta**: Alustava käsittely voi kestää kauemmin, koska Chloros luo välimuisteja ja profiileja. Samanlaisten aineistojen myöhempi käsittely sujuu nopeammin.
{% endhint %}

***

## Yleisiä ongelmia käynnistyksen yhteydessä

### Käynnistyspainike ei ole käytettävissä (harmaana)

**Mahdolliset syyt:**

* Kuvia ei ole tuotu
* Taustapalvelu ei ole käynnistynyt kokonaan
* Edellinen käsittely on vielä käynnissä
* Projektia ei ole ladattu kokonaan

**Ratkaisut:**

1. Odota, kunnes taustapalvelu on alustettu kokonaan (tarkista päävalikon kuvake)
2. Varmista, että kuvat on tuotu tiedostoselaimeen
3. Käynnistä Chloros uudelleen, jos painike on edelleen poissa käytöstä
4. Tarkista virheilmoitukset vianmäärityslokista

### Käsittely käynnistyy, mutta keskeytyy välittömästi

**Mahdolliset syyt:**

* Projektissa ei ole kelvollisia kuvia
* Vioittuneet kuvatiedostot
* Riittämätön levytila
* Riittämätön muisti (RAM)

**Ratkaisut:**

1. Tarkista virheilmoitukset vianmäärityslokista (<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">)
2. Varmista, että levytilaa on riittävästi
3. Yritä käsitellä pienempää osajoukkoa kuvista
4. Varmista, että kuvat eivät ole vioittuneita

### Suoritus päättyy, mutta kuvia ei tallenneta

Suoritus, jossa pyydettiin kuvatuotteita mutta yhtään kuvaa ei tallennettu, käsitellään **epäonnistumisena, ei onnistumisena** — Chloros ilmoittaa siitä selvästi:

* GUI-lokissa näkyy viesti `[RUN-SUMMARY]`, joka vihjaa todennäköiseen syyhyn — kuvia ei ole tuotu, kohdetta ei ole havaittu tai kaikki pyydetyt tuotteet on ohitettu soveltumattomina (esim. säteily- tai heijastusarvojen pyytäminen kameroista, jotka tukevat vain RGB-tilaa)
* CLI:n vastine (`chloros-cli process`) tulostaa `Processing finished but wrote no image products.`:n ja **päättyy nollasta poikkeavalla arvolla**, joten skriptit voivat havaita sen
* Tarkoituksellinen pelkästään metatietoja tuottava suoritus (kaikki vientituotteet pois käytöstä, ei indeksejä) lasketaan silti onnistuneeksi

Katso [CLI-viite](../reference/cli-reference.md#a-run-that-writes-no-images-fails) saadaksesi täydelliset semanttiset tiedot.

### Varoitus ”Kohteita ei havaittu”

**Mahdolliset syyt:**

* Unohdit merkitä kohdekuvat
* Kohdekuvissa ei ole näkyviä kohteita
* Kohteen tunnistuksen asetukset ovat liian tiukat

**Ratkaisut:**

1. Tutustu kohtaan [Kohdekuvien valinta](choosing-target-images.md)
2. Merkitse sopivat kuvat Kohde-sarakkeeseen
3. Varmista, että kohteet ovat näkyvissä merkityissä kuvissa
4. Säädä kohteen tunnistuksen asetuksia tarvittaessa

***

## Vinkkejä onnistuneeseen käsittelyyn

### Ennen aloittamista

1. **Testaa ensin pienellä osajoukolla** – Käsittele 10–20 kuvaa asetusten tarkistamiseksi
2. **Tarkista käytettävissä oleva levytila** – Varmista, että vapaata tilaa on 2–3 kertaa datajoukon koko (enemmän, jos kaikki LATTICE-tuotteet ovat käytössä)
3. **Sulje tarpeettomat sovellukset** – Vapauta järjestelmäresursseja
4. **Tarkista kohdekuvat** – Esikatsele merkittyjä kohteita laadun varmistamiseksi
5. **Tallenna projekti** – Projekti tallentuu automaattisesti, mutta on hyvä tallentaa se myös manuaalisesti

### Käsittelyn aikana

1. **Vältä järjestelmän siirtymistä lepotilaan** – Poista virransäästötilat käytöstä
2. **Pidä Chloros etualalla** – Tai ainakin näkyvissä tehtäväpalkissa
3. **Seuraa edistymistä ajoittain** – Tarkista, onko varoituksia tai virheitä
4. **Älä avaa muita resursseja kuluttavia sovelluksia** – Erityisesti Chloros+:n rinnakkaistilassa

### Chloros+:n GPU-kiihdytys

Jos käytät NVIDIA-GPU-kiihdytystä:

1. Päivitä NVIDIA-ajurit uusimpaan versioon
2. Varmista, että GPU:lla on vähintään 4 Gt VRAM-muistia (vähintään 7 Gt samanaikaista Texture Aware -debayering-käsittelyä varten)
3. Sulje GPU:ta rasittavat sovellukset (pelit, videonmuokkaus)
4. Tarkkaile GPU:n lämpötilaa (varmista riittävä jäähdytys)

***

## Seuraavat vaiheet

Kun käsittely on alkanut:

1. **Seuraa käsittelyn etenemistä** – Katso [Käsittelyn seuranta](monitoring-the-processing.md)
2. **Odota käsittelyn päättymistä** – Käsittely suoritetaan automaattisesti
3. **Tarkista tulokset** – Katso [Käsittelyn päättäminen](finishing-the-processing.md)

Tietoa siitä, mitä käsittelyn aikana tulee tehdä, löytyy kohdasta [Käsittelyn seuranta](monitoring-the-processing.md).
