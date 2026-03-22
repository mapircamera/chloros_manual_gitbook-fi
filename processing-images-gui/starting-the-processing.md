# Käsittelyn aloittaminen

Kun olet tuonut kuvat, merkinnyt kalibrointikohteet ja määrittänyt projektin asetukset, olet valmis aloittamaan käsittelyn. Tällä sivulla opastetaan sinua Chloros-käsittelyputken käynnistämisessä.

## Käsittelyä edeltävä tarkistuslista

Ennen kuin napsautat Käynnistä-painiketta, varmista, että kaikki on valmiina:

* [ ] **Tiedostot tuotu** – Kaikki kuvat näkyvät tiedostoselaimessa
* [ ] **Kohdekuvat merkitty** – Kohdesarakkeessa on valittu kalibrointikuvat
* [ ] **Kameramallit tunnistettu** – Kameramalli-sarakkeessa näkyvät oikeat kamerat
* [ ] **Asetukset määritetty** – Projektin asetukset tarkistettu ja säädetty
* [ ] **Indeksit valittu** – Halutut monispektriset indeksit lisätty (tarvittaessa)
* [ ] **Vientimuoto valittu** – Työnkulkuusi sopiva tulostusmuoto

{% hint style="info" %}
**Vinkki**: Selaa muutamia kuvia tiedostoselaimessa varmistaaksesi, että ne on ladattu oikein ennen käsittelyä.
{% endhint %}

***

## Käsittelyn aloittaminen

### Etsi Käynnistä-painike

Käynnistä/Toista-painike sijaitsee Chloros:n yläpalkissa:

* Sijainti: Ikkunan yläosassa keskellä
* Kuvake: **Toista/Käynnistä-painike** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
* Tila: Painike on käytössä (valaistu), kun käsittely on valmis

### Aloita napsauttamalla

1. Napsauta **Toisto-/Käynnistyspainiketta** yläpalkissa
2. Käsittely alkaa välittömästi
3. Painike poistuu käytöstä (harmaana) käsittelyn aikana
4. Edistymispalkki päivittyy ja näyttää käsittelyn tilan

{% hint style="success" %}
**Käsittely aloitettu**: Kun painiketta on napsautettu, Chloros hoitaa automaattisesti kaikki käsittelyvaiheet – kohteen tunnistuksen, debayeringin, kalibroinnin, indeksin laskennan ja viennin.
{% endhint %}

***

## Käsittelytilojen ymmärtäminen

Chloros toimii kahdessa eri käsittelytilassa lisenssistäsi riippuen:

### Ilmainen tila (peräkkäinen käsittely)

**Saatavilla kaikille käyttäjille**

**Kuinka se toimii:**

* Käsittelee kuvia yksi kerrallaan, peräkkäin
* Yksisäikeinen toiminta
* Pienempi muistin käyttö

**Edistymispalkki näyttää 2 vaihetta:**

1.**Kohteen tunnistus** – Kalibrointikohteiden etsiminen
2. **Käsittely** – Kalibroinnin soveltaminen ja kuvien vieminen**Käsittelyaika:**

* Paljon hitaampi kuin Chloros+:n rinnakkaistila
* Sopii pienille ja keskisuurille tietojoukoille (&lt; 200 kuvaa)

### Chloros+ -tila (rinnakkaiskäsittely)

**Vaatii Chloros+ -lisenssin**

**Kuinka se toimii:**

* Käsittelee useita kuvia samanaikaisesti käyttämällä [4-säikeistä käsittelyputkea](../processing-architecture/processing-pipeline.md)
* [Dynaaminen laskentasovitus](../processing-architecture/dynamic-compute-adaptation.md) valitsee automaattisesti laitteistollesi optimaalisen strategian
* GPU (CUDA) -kiihdytys NVIDIA-näytönohjaimilla (pöytätietokone ja Jetson)
* Skaalautuu Jetson Nanosta (1 työntekijä) pöytätietokoneeseen, jossa on vähintään 12 Gt:n GPU (3–4 työntekijää)

**Edistymispalkki näyttää 4 vaihetta** (vastaavat 4 putkistoketjua):

1. **Havaitseminen** (Ketju 1) – Kalibrointikohteiden etsiminen
2. **Analysointi** (Ketju 2) – Kuvan metatietojen tarkastelu ja kalibroinnin laskeminen
3. **Kalibrointi** (Ketju 3) – GPU-debayering, vinjetin korjaus, indeksin laskeminen
4. **Vienti** (Säie 4) – Käsiteltyjen kuvien ja indeksien tallentaminen**Edistymispalkin käyttö:*** **Vie hiiri** palkin päälle nähdäksesi yksityiskohtaisen 4-vaiheisen pudotusvalikon
* **Napsauta** edistymispalkkia pysäyttääksesi pudotusvalikon paikalleen
* **Napsauta uudelleen** vapauttaaksesi ja piilottaaksesi valikon**Käsittelyaika:**

* Huomattavasti nopeampi kuin ilmainen tila
* Skaalautuu CPU-ytimien määrän mukaan
* GPU-kiihdytys parantaa nopeutta entisestään

{% hint style="info" %}
**Chloros+ Nopeus**: Rinnakkaisprosessointi voi olla 5–10 kertaa nopeampaa kuin peräkkäistila suurille tietojoukoille. 500 kuvan projekti, joka kestää 2 tuntia ilmaisessa tilassa, voi valmistua 15–20 minuutissa Chloros+:lla.
{% endhint %}

***

## Mitä tapahtuu käsittelyn aikana

### Vaihe 1: Kohteen tunnistus

**Mitä Chloros tekee:**

* Skannaa merkityt kohdekuvat (tai kaikki kuvat, jos mitään ei ole merkitty)
* Tunnistaa kunkin kohteen 4 kalibrointipaneelia
* Poimii heijastusarvot kohdepaneeleista
* Tallentaa kohteen aikaleimat kalibroinnin aikataulutusta varten

**Kesto:** 1–30 sekuntia (merkityt kohteet), 5–30+ minuuttia (merkitsemättömät)

### Vaihe 2: Debayering (RAW-muunnos)

**Mitä Chloros tekee:**

* Muuntaa RAW-Bayer-kuviotiedot täysikokoisiksi RGB-kuviksi
* Käyttää korkealaatuista demosaicing-algoritmia
* Säilyttää kuvan laadun ja yksityiskohdat mahdollisimman hyvin

**Kesto:** Vaihtelee kuvien määrän ja prosessorin nopeuden mukaan

### Vaihe 3: Kalibrointi

**Mitä Chloros tekee:*** **Vignettikorjaus**: Poistaa objektiivin tummentaman reunan
* **Heijastavuuden kalibrointi**: Normalisoi käyttämällä kohdeheijastavuusarvoja
* Soveltaa korjauksia kaikkiin kaistoihin/kanaviin
* Käyttää kullekin kuvalle sopivaa kalibrointikohdetta aikaleiman perusteella

**Kesto:** Suurin osa käsittelyajasta

### Vaihe 4: Indeksin laskeminen

**Mitä Chloros tekee:**

* Laskee määritetyt monispektriset indeksit (NDVI, NDRE jne.)
* Soveltaa kaistamatematiikkaa kalibroituihin kuviin
* Luo indeksikuvia jokaiselle valitulle indeksille

**Kesto:** Muutama sekunti kuvaa kohti

### Vaihe 5: Vienti

**Mitä Chloros tekee:**

* Tallentaa kalibroidut kuvat valitussa muodossa
* Vie indeksikuvat määritetyillä LUT-väreillä
* Kirjoittaa tiedostot kameramallien alikansioihin
* Säilyttää alkuperäiset tiedostonimet ja liitteet

**Kesto:** Vaihtelee vientimuodon ja tiedostokoon mukaan***

## Käsittelyn toiminta

### Automaattinen käsittelyputki

Kun käsittely on käynnistetty, koko putki toimii automaattisesti:

* Käyttäjän toimia ei tarvita
* Kaikki määritetyt vaiheet suoritetaan järjestyksessä
* Edistymispäivitykset näkyvät reaaliajassa

### Tietokoneen käyttö käsittelyn aikana

**Vapaa tila:**

* Suhteellisen alhainen CPU-käyttö (yksisäikeinen)
* Tietokone reagoi edelleen muihin tehtäviin
* Voit turvallisesti minimoida Chloros-ikkunan ja työskennellä muissa sovelluksissa

**Chloros+ Rinnakkaistila:**

* Korkea CPU-käyttö (monisäikeinen, jopa 16 ydintä)
* GPU-kiihdytyksellä: Korkea GPU-käyttö
* Tietokoneen reagointikyky voi heikentyä käsittelyn aikana
* Vältä muiden CPU-intensiivisten tehtävien käynnistämistä

{% hint style="warning" %}
**Suorituskykyvinkki**: Parhaan Chloros+ suorituskyvyn saavuttamiseksi sulje muut sovellukset ja anna Chloros:n käyttää järjestelmän resursseja täysimääräisesti.
{% endhint %}

### Käsittelyä ei voi keskeyttää

**Tärkeitä rajoituksia:**

* Kun käsittely on aloitettu, sitä ei voi keskeyttää
* Voit peruuttaa käsittelyn, mutta edistys menetetään
* Osittaisia tuloksia ei tallenneta
* Peruutuksen jälkeen on aloitettava alusta

**Suunnitteluvinkki:** Erittäin suurissa projekteissa kannattaa harkita käsittelyä erissä tai CLI:n käyttöä paremman hallinnan saavuttamiseksi.***

## Käsittelyn seuranta

Käsittelyn ollessa käynnissä voit:

* **Seurata edistymispalkkia** – Katsoa kokonaisvalmiusprosenttia
* **Tarkastella nykyistä vaihetta** – Tunnista, Analysoi, Kalibroi tai Vie
* **Tarkistaa loki-välilehden** – Katsoa yksityiskohtaisia käsittelyviestejä ja varoituksia
* **Esikatsella valmiita kuvia** – Jotkin vientitiedostot saattavat näkyä käsittelyn aikana

Yksityiskohtaisia tietoja seurannasta on kohdassa [Käsittelyn seuranta](monitoring-the-processing.md).

***

## Käsittelyn peruuttaminen

Jos haluat keskeyttää käsittelyn:

### Peruuttaminen

1. Etsi **Stop/Cancel-painike** (korvaa Start-painikkeen käsittelyn aikana)
2. Napsauta Stop-painiketta
3. Käsittely keskeytyy välittömästi
4. Osittaiset tulokset hylätään

### Milloin keskeyttää

**Hyväksyttävät syyt keskeyttämiseen:**

* Huomattiin, että asetukset olivat virheelliset
* Unohdettiin merkitä kohdekuvat
* Tuotiin vääriä kuvia
* Järjestelmä toimii liian hitaasti tai ei vastaa

**Keskeyttämisen jälkeen:**

* Tarkista ja korjaa mahdolliset ongelmat
* Säädä asetuksia tarpeen mukaan
* Aloita käsittely alusta
* Parhaan käyttökokemuksen saamiseksi sulje Chloros kokonaan ja käynnistä

{% hint style="warning" %}
uudelleen
**Ei osittaisia tuloksia**: Peruuttaminen hylkää kaiken edistymisen. Chloros ei tallenna osittain käsiteltyjä kuvia.
{% endhint %}

***

## Käsittelyajan arviot

Todellinen käsittelyaika vaihtelee suuresti seuraavien tekijöiden mukaan:

* Kuvien määrä
* Kuvan resoluutio
* RAW- tai JPG-tulomuoto
* Käsittelytila (Free tai Chloros+)
* Prosessorin nopeus ja ydinten lukumäärä
* GPU:n saatavuus (vain Chloros+)
* Laskettavien hakemistojen määrä
* Vientimuodon monimutkaisuus

### Karkeat arviot (Chloros+, 12 MP:n kuvat, nykyaikainen CPU)

| Kuvien määrä | Ilmainen tila | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 kuvaa   | 15–20 min | 5–8 min        | 3–5 min        |
| 100 kuvaa  | 30–40 min | 10–15 min      | 5–8 min        |
| 200 kuvaa  | 1–1,5 tuntia | 20–30 min      | 10–15 min      |
| 500 kuvaa  | 2–3 tuntia   | 45–60 min      | 20–30 min      |
| 1000 kuvaa | 4–6 tuntia   | 1,5–2 tuntia      | 40–60 min      |

{% hint style="info" %}
**Ensimmäinen käyttökerta**: Alustava käsittely voi kestää kauemmin, kun Chloros luo välimuistia ja profiileja. Samanlaisten tietojoukkojen myöhempi käsittely on nopeampaa.
{% endhint %}

***

## Yleisiä ongelmia käynnistyksen yhteydessä

### Käynnistyspainike ei toimi (harmaana)

**Mahdolliset syyt:**

* Kuvia ei ole tuotu
* Taustapalvelu ei ole käynnistynyt kokonaan
* Edellinen käsittely on vielä käynnissä
* Projektia ei ole ladattu kokonaan

**Ratkaisut:**

1. Odota, että taustapalvelu käynnistyy kokonaan (tarkista päävalikon kuvake)
2. Varmista, että kuvat on tuotu tiedostoselaimeen
3. Käynnistä Chloros uudelleen, jos painike on edelleen pois käytöstä
4. Tarkista virheilmoitukset virhelokista

### Käsittely käynnistyy, mutta epäonnistuu välittömästi

**Mahdolliset syyt:**

* Projektissa ei ole kelvollisia kuvia
* Vioittuneet kuvatiedostot
* Riittämätön levytila
* Riittämätön muisti (RAM)

**Ratkaisut:**

1. Tarkista virheloki <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> virheilmoitusten varalta
2. Tarkista käytettävissä oleva levytila
3. Yritä käsitellä pienempiä kuvajoukkoja
4. Varmista, että kuvat eivät ole vioittuneita

### Varoitus &quot;Kohteita ei havaittu&quot;

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
2. **Tarkista käytettävissä oleva levytila** – Varmista, että vapaata tilaa on 2–3 kertaa datajoukon koko
3. **Sulje tarpeettomat sovellukset** – Vapauta järjestelmän resursseja
4. **Tarkista kohdekuvat** – Esikatsele merkittyjä kohteita laadun varmistamiseksi
5. **Tallenna projekti** – Projekti tallentuu automaattisesti, mutta on hyvä tallentaa se myös manuaalisesti

### Käsittelyn aikana

1. **Vältä järjestelmän lepotilaa** – Poista virransäästötilat käytöstä
2. **Pidä Chloros etualalla** – Tai ainakin näkyvissä tehtäväpalkissa
3. **Seuraa edistymistä ajoittain** – Tarkista, onko varoituksia tai virheitä
4. **Älä lataa muita raskaita sovelluksia** – Erityisesti Chloros+ rinnakkaistilassa

### Chloros+ GPU-kiihdytys

Jos käytät NVIDIA-GPU-kiihdytystä:

1. Päivitä NVIDIA-ajurit uusimpaan versioon
2. Varmista, että GPU:lla on vähintään 4 Gt VRAM-muistia
3. Sulje GPU:ta paljon kuormittavat sovellukset (pelit, videonmuokkaus)
4. Tarkkaile GPU:n lämpötilaa (varmista riittävä jäähdytys)

***

## Seuraavat vaiheet

Kun käsittely on alkanut:

1. **Seuraa edistymistä** – Katso [Käsittelyn seuranta](monitoring-the-processing.md)
2. **Odota käsittelyn päättymistä** – Käsittely suoritetaan automaattisesti
3. **Tarkista tulokset** – Katso [Käsittelyn päättäminen](finishing-the-processing.md)

Lisätietoja käsittelyn aikana suoritettavista toimista on kohdassa [Käsittelyn seuranta](monitoring-the-processing.md).
