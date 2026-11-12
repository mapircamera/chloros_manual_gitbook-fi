# Käsittelyn aloittaminen

Kun olet tuonut kuvasi, merkinnyt kalibrointikohteet ja määrittänyt projektin asetukset, olet valmis aloittamaan käsittelyn. Tällä sivulla opastetaan sinua Chloros-käsittelyprosessin käynnistämisessä.

## Esikäsittelyn tarkistuslista

Ennen kuin napsautat Käynnistä-painiketta, varmista, että kaikki on valmiina:

* [ ] **Tiedostot tuotu** – Kaikki kuvat näkyvät tiedostoselaimessa
* [ ] **Kohdekuvat merkitty** – Kohde-sarakkeessa on valittu kalibrointikuvat (tai LATTICE-ohjelmistolle on tuotu `.daq`-tallenne)
* [ ] **Kameramallit tunnistettu** – Kameramalli-sarakkeessa näkyvät oikeat kamerat
* [ ] **Asetukset määritetty** – Projektin asetukset tarkistettu ja säädetty
* [ ] **Indeksit valittu** – Halutut monispektriset indeksit lisätty (tarvittaessa)
* [ ] **Vientimuoto valittu** – Työnkulkuusi sopiva tulostusmuoto

{% hint style="info" %}
**Vinkki**: Selaa muutamia kuvia tiedostoselaimessa varmistaaksesi, että ne ovat latautuneet oikein ennen käsittelyä.
{% endhint %}

***

## Käsittelyn aloittaminen

### Etsi Käynnistä-painike

Käynnistä/Toista-painike sijaitsee Chloros-ohjelman yläpalkissa:

* Sijainti: Ikkunan yläosassa keskellä
* Kuvake: **Toista/Käynnistä-painike** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Tila: Painike on käytössä (valaistu), kun käsittely on valmis

### Aloita napsauttamalla

1. Napsauta yläpalkissa olevaa **Toisto-/Käynnistyspainiketta**

2. Käsittely alkaa välittömästi
3. Painike muuttuu **Pysäytyspainikkeeksi** käsittelyn aikana
4. Edistymispalkki päivittyy ja näyttää käsittelyn tilan

{% hint style="success" %}
**Käsittely aloitettu**: Kun painiketta on napsautettu, Chloros hoitaa automaattisesti kaikki käsittelyvaiheet – kohteen tunnistuksen, debayeringin, kalibroinnin, indeksin laskennan ja viennin. Se tunnistaa automaattisesti, onko projektisi Survey3-, LATTICE- vai sekamuotoinen, ja soveltaa kuhunkin kameraan oikeaa käsittelyputkea.
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

* Paljon hitaampi kuin Chloros+:n rinnakkaistilassa
* Sopii pienille ja keskisuurille aineistoille (< 200 kuvaa)

### Chloros+-tila (rinnakkaiskäsittely)

**Vaatii Chloros+-lisenssin**

**Kuinka se toimii:**

* Käsittelee useita kuvia samanaikaisesti käyttämällä [4-säikeistä käsittelyputkea](../processing-architecture/processing-pipeline.md)
* [Dynaaminen laskentasovitus](../processing-architecture/dynamic-compute-adaptation.md) valitsee automaattisesti laitteistollesi optimaalisen strategian suorituksen alkaessa
* GPU (CUDA) -kiihdytys NVIDIA-näytönohjaimilla
* **Työntekijöiden lukumäärä mukautuu laitteistoon**: GPU-strategioissa ajetaan**2–4 samanaikaista työntekijää** (skaalataan VRAM-muistin mukaan — 3 kpl 2–12 Gt:n GPU:lla, 2–4 kpl 12 Gt:n tai suuremmalla GPU:lla); Pelkästään CPU:ta käyttävissä järjestelmissä ajetaan yksi työprosessi kutakin fyysistä ydintä kohti, miinus yksi**Edistymispalkki näyttää 4 vaihetta** (vastaavat 4:ää putkistoketjua):

1. **Tunnistaminen** (ketju 1) – Kalibrointikohteiden etsiminen
2. **Analysointi** (säie 2) – Kuvan metatietojen tarkastelu ja kalibroinnin laskeminen
3. **Kalibrointi** (säie 3) – Debayering, vinjetoinnin korjaus, kalibrointi, indeksin laskeminen
4. **Vienti** (säie 4) – Käsiteltyjen kuvien ja indeksien tallentaminen**Edistymispalkin käyttö:*** **Vie hiiri** palkin päälle nähdäksesi yksityiskohtaisen nelivaiheisen pudotusvalikon
* **Napsauta** edistymispalkkia pysäyttääksesi pudotusvalikon paikalleen
* **Napsauta uudelleen** vapauttaaksesi valikon ja piilottaaksesi sen**Käsittelyaika:**

* Huomattavasti nopeampi kuin ilmainen tila
* GPU-kiihdytys parantaa nopeutta entisestään

{% hint style="info" %}
**Chloros+:n nopeus**: Suurten tietojoukkojen kohdalla rinnakkaiskäsittely voi olla 5–10 kertaa nopeampaa kuin peräkkäinen käsittely. 500 kuvan projekti, joka kestää ilmaisversiossa 2 tuntia, voi valmistua Chloros+:lla 15–20 minuutissa.
{% endhint %}

***

## Mitä tapahtuu käsittelyn aikana

### Vaihe 1: Kohteiden tunnistus

**Mitä Chloros tekee:**

* Skannaa kuvat, jotka olet valinnut Kohde-sarakkeessa (kaikki kuvat, jos yhtään ei ole valittu)
* Tunnistaa kalibrointipaneelit kussakin kohteessa
* Poimii heijastusarvot kohdepaneeleista
* Tallentaa kohteiden aikaleimat kalibroinnin aikataulutusta varten

**Kesto:** 1–30 sekuntia (merkittyjen kohteiden kanssa), 5–30+ minuuttia (merkitsemättömien kohteiden kanssa)

### Vaihe 2: Debayering (RAW-muunnos)

**Mitä Chloros tekee:**

* Muuntaa RAW-muotoiset Bayer-kuviotiedot täysimittaisiksi 3-kanavaisiksi kuviksi (LATTICE-mono-moduulit pysyvät yksikaistaisina — niiden osalta debayering ohitetaan ja siitä tehdään merkintä lokiin)
* Soveltaa valittua demosaicing-algoritmia
* Säilyttää kuvan laadun ja yksityiskohdat mahdollisimman hyvin

**Kesto:** Vaihtelee kuvamäärän ja CPU/GPU:n nopeuden mukaan

### Vaihe 3: Kalibrointi

**Mitä Chloros tekee:*** **Vignettikorjaus**: Poistaa objektiivin aiheuttaman tummenemisen reunoilta
* **Heijastavuuden kalibrointi**: Normalisoi käyttämällä kohteen heijastavuusarvoja ja/tai DAQ:n alaspäin suuntautuvaa dataa
* Soveltaa korjauksia kaikkiin kaistoihin/kanaviin
* Käyttää kullekin kuvalle sopivaa kalibrointiviitettä aikaleiman perusteella

**Kesto:** Suurin osa käsittelyajasta

### Vaihe 4: Indeksin laskeminen

**Mitä Chloros tekee:**

* Laskee määritetyt monispektriset indeksit (NDVI, NDRE jne.)
* Soveltaa kaistamatematiikkaa kalibroituihin kuviin
* Luo indeksikuvia jokaiselle valitulle indeksille

**Kesto:** Muutama sekunti kuvaa kohti

### Vaihe 5: Vienti

**Mitä Chloros tekee:**

* Tallentaa käsitellyt kuvat valitussa muodossa
* **LATTICE-fan-out**: jokainen raaka LATTICE-kehys viedään yhtenä kerralla kaikkina käytössä olevina tuotteina — debayeroitu, esikatselu, säteilyvoimakkuus (aina float32), heijastavuus
* Kirjoittaa tiedostot projektin tulostuskansioon: `<project>/<camera>/<format>/<Product>_Images/`
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
* Tietokone reagoi edelleen muihin tehtäviin
* Voit turvallisesti pienentää Chloros-ikkunan ja työskennellä muissa sovelluksissa

**Chloros+ rinnakkaistila:**

* Suuri CPU-kuormitus strategian työryhmässä
* GPU-kiihdytyksellä: Suuri GPU-kuormitus
* Tietokoneen reagointikyky voi heikentyä käsittelyn aikana
* Vältä muiden CPU-resursseja vaativien tehtävien käynnistämistä

{% hint style="warning" %}
**Suorituskykyvinkki**: Parhaan Chloros+-suorituskyvyn saavuttamiseksi sulje muut sovellukset ja anna Chlorosin käyttää järjestelmän resursseja täysimääräisesti.
{% endhint %}

### Käsittelyä ei voi keskeyttää (mutta lopettaminen on siistiä)

* Kun käsittely on alkanut, sitä ei voi keskeyttää ja jatkaa myöhemmin
* **Stop**-painikkeen napsauttaminen lopettaa ajon siististi jo ensimmäisellä napsautuksella
* Ennen lopettamista jo viedyt tuotteet säilyvät levyllä
* Lopetettu ajo raportoi rehellisesti, mitä se on saanut valmiiksi (katso lokin `[RUN-SUMMARY]`-rivit)
* Uusi ajo käynnistää prosessiketjun alusta

**Suunnitteluvinkki:** Erittäin suurissa projekteissa kannattaa harkita käsittelyä erissä tai komentorivin (CLI) käyttöä paremman hallinnan saavuttamiseksi.

***

## Käsittelyn seuranta

Käsittelyn ollessa käynnissä voit:

* **Seurata edistymispalkkia** – Näet kokonaisvalmiusprosentin
* **Tarkastella nykyistä vaihetta** – Tunnistus, analysointi, kalibrointi tai vienti
* **Tarkistaa loki-välilehden** – Näet yksityiskohtaiset käsittelyviestit ja varoitukset
* **Esikatsella valmiita kuvia** – Vientitiedostot näkyvät levyllä käsittelyn aikana

Yksityiskohtaisia tietoja seurannasta on kohdassa [Käsittelyn seuranta](monitoring-the-processing.md).

***

## Käsittelyn pysäyttäminen

Jos haluat pysäyttää käsittelyn:

### Pysäyttäminen

1. Etsi **Pysäytä-painike** (korvaa Käynnistä-painikkeen käsittelyn aikana)
2. Napsauta sitä kerran — palkissa näkyy **”Pysäytetään...”**, kunnes keskeneräinen kuva valmistuu
3. Suoritus päättyy lopullisesti pysäytettyyn tilaan ja lokiin tulostuu tarkka `[RUN-SUMMARY]`-raportti siitä, mitä on saatu valmiiksi

### Milloin käsittely kannattaa keskeyttää

**Hyväksyttäviä syitä keskeyttämiseen:**

* Huomattiin, että asetukset olivat virheelliset
* Unohdettiin merkitä kohdekuvat
* Tuotiin vääriä kuvia
* Järjestelmä toimii liian hitaasti tai ei vastaa

**Keskeyttämisen jälkeen:**

* Ennen keskeyttämistä viedyt tuotteet säilyvät levyllä
* Tarkista ja korjaa mahdolliset ongelmat, säädä asetuksia tarpeen mukaan
* Käynnistä käsittely uudelleen — ajo alkaa alusta

***

## Käsittelyajan arviot

Todellinen käsittelyaika vaihtelee suuresti seuraavien tekijöiden mukaan:

* Kuvien lukumäärä
* Kuvan resoluutio
* RAW- vai JPG-tulomuoto
* Käsittelytila (Free vs. Chloros+)
* Prosessorin nopeus ja ydinten lukumäärä
* GPU:n saatavuus (vain Chloros+)
* Laskettavien indeksien määrä
* Käytössä olevien vientituotteiden määrä (LATTICE)

### Karkeat arviot (Chloros+, 12 MP:n kuvat, nykyaikainen prosessori)

| Kuvien lukumäärä | Ilmainen tila | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 kuvaa   | 15–20 min | 5–8 min        | 3–5 min        |
| 100 kuvaa  | 30–40 min | 10–15 min      | 5–8 min        |
| 200 kuvaa  | 1–1,5 tuntia | 20–30 min      | 10–15 min      |
| 500 kuvaa  | 2–3 tuntia   | 45–60 min      | 20–30 min      |
| 1000 kuvaa | 4–6 tuntia   | 1,5–2 tuntia      | 40–60 min      |

{% hint style="info" %}
**Ensimmäinen ajo**: Alustava käsittely voi kestää kauemmin, kun Chloros luo välimuisteja ja profiileja. Samankaltaisten aineistojen myöhempi käsittely sujuu nopeammin.
{% endhint %}

***

## Yleisiä ongelmia käynnistyksen yhteydessä

### Käynnistyspainike ei ole käytettävissä (harmaana)

**Mahdolliset syyt:**

* Kuvia ei ole tuotu
* Taustapalvelu ei ole täysin käynnistynyt
* Edellinen käsittely on vielä käynnissä
* Projektia ei ole ladattu kokonaan

**Ratkaisut:**

1. Odota, kunnes taustapalvelu on alustettu kokonaan (tarkista päävalikon kuvake)
2. Varmista, että kuvat on tuotu tiedostoselaimeen
3. Käynnistä Chloros uudelleen, jos painike on edelleen poissa käytöstä
4. Tarkista virheilmoitukset virhelokista

### Käsittely käynnistyy, mutta epäonnistuu välittömästi

**Mahdolliset syyt:**

* Projektissa ei ole kelvollisia kuvia
* Vioittuneet kuvatiedostot
* Levytilaa ei riitä
* Muistia (RAM) ei riitä

**Ratkaisut:**

1. Tarkista virheilmoitukset vianmäärityslokista <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">
2. Tarkista käytettävissä oleva levytila
3. Yritä käsitellä pienempää osajoukkoa kuvia
4. Varmista, että kuvat eivät ole vioittuneita

### Suoritus päättyy, mutta kuvia ei tallenneta

Suoritus, jossa pyydettiin kuvatuotteita mutta yhtään ei tallennettu, käsitellään **epäonnistumisena, ei onnistumisena** — Chloros ilmoittaa siitä selvästi:

* GUI-lokissa näkyy viesti `[RUN-SUMMARY]`, joka vihjaa todennäköiseen syyhyn — kuvia ei tuotu, kohdetta ei havaittu tai kaikki pyydetyt tuotteet ohitettiin soveltumattomina (esim. säteilyvoimakkuuden/heijastavuuden pyytäminen pelkästään RGB-kameroista)
* Vastaava CLI-viesti (`chloros-cli process`) tulostaa `Processing finished but wrote no image products.` ja **päättyy nollasta poikkeavalla arvolla**, joten skriptit voivat havaita sen
* Tarkoituksellisesti vain metatietoja tuottava ajo (kaikki vientituotteet pois käytöstä, ei indeksejä) lasketaan silti onnistuneeksi

Katso [CLI-viite](../reference/cli-reference.md#a-run-that-writes-no-images-fails) saadaksesi täydelliset semanttiset tiedot.

### Projektikansio on liian syvällä tässä tietokoneessa

**Vain Windows, uusi ominaisuus versiossa 1.3.0.** Windowsissa pitkät polut ovat oletusarvoisesti pois käytöstä, jolloin tiedostopolku ei saa olla pidempi kuin 260 merkkiä. Chloros-tuotteet sijaitsevat noin 130 merkkiä projektikansion alapuolella, joten tätä syvemmällä sijaitseva projektikansio ei voi vastaanottaa tuotteitaan. Sen sijaan, että ohjelma suoritettaisiin ilman tuloksia, Chloros tarkistaa nyt ennen käsittelyn aloittamista:

* Käynnistä-painikkeen napsauttaminen ei käynnistä suoritusta: etenemispalkki näyttää syyn, ja virhelokissa näkyy `[PROCESS] refused: The project folder is too deep for this computer…`. Viesti ilmoittaa projektikansion pituuden ja pisimmän sopivan projektikansion.
* Projektin luominen tai avaaminen kansiossa, joka on liian syvällä jopa projektin omille tiedostoille, hylätään myös. Viestiruudussa kerrotaan syy, mikä on pisin mahtuva projektikansio ja miten pitkät polut otetaan käyttöön, eikä mitään luoda tai avata; virheloki tallentaa tapahtuman koodina `[PROJECT] refused: …`. Projektin kopioiminen pidemmällä nimellä voi myös ylittää rajan: kopio luodaan, mutta sitä ei avata, ja sama viesti kertoo syyn.

**Ratkaisut:**

1. Säilytä projektit lyhyemmässä kansiossa — muuta**Työkansio** kohdassa [Projektin asetukset](../project-settings/project-settings.md) ja luo projekti uudelleen, tai
2. Ota Windowsin pitkät polut käyttöön ja käynnistä Chloros uudelleen: suorita Chloros-asennusohjelma uudelleen, kun **Ota Windowsin pitkät polut käyttöön** -valinta on valittuna (se on valittuna oletuksena), tai käytä ryhmäkäytäntöä: *Tietokoneen kokoonpano > Hallintamallit > Järjestelmä > Tiedostojärjestelmä > Ota Win32-pitkät polut käyttöön*.

### Varoitus ”Kohteita ei havaittu”

**Mahdolliset syyt:**

* Unohdit merkitä kohdekuvat
* Kohdekuvat eivät sisällä näkyviä kohteita
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
5. **Tallenna projekti** – Projekti tallentuu automaattisesti, mutta on hyvä käytäntö tallentaa se manuaalisesti

### Käsittelyn aikana

1. **Vältä järjestelmän lepotilaa** – Poista virransäästötilat käytöstä
2. **Pidä Chloros etualalla** – Tai ainakin näkyvissä tehtäväpalkissa
3. **Seuraa edistymistä ajoittain** – Tarkista, onko varoituksia tai virheitä
4. **Älä lataa muita raskaita sovelluksia** – Erityisesti Chloros+:n rinnakkaistilassa

### Chloros+:n GPU-kiihdytys

Jos käytät NVIDIA:n GPU-kiihdytystä:

1. Päivitä NVIDIA-ajurit uusimpaan versioon
2. Varmista, että GPU:ssa on vähintään 4 Gt VRAM-muistia (vähintään 7 Gt samanaikaista Texture Aware -debayeringia varten)
3. Sulje GPU:ta paljon kuormittavia sovelluksia (pelit, videonmuokkaus)
4. Tarkkaile GPU:n lämpötilaa (varmista riittävä jäähdytys)

***

## Seuraavat vaiheet

Kun käsittely on alkanut:

1. **Seuraa edistymistä** – Katso [Käsittelyn seuranta](monitoring-the-processing.md)
2. **Odota käsittelyn päättymistä** – Käsittely sujuu automaattisesti
3. **Tarkista tulokset** – Katso [Käsittelyn päättäminen](finishing-the-processing.md)

Lisätietoja siitä, mitä käsittelyn aikana tulee tehdä, on kohdassa [Käsittelyn seuranta](monitoring-the-processing.md).
