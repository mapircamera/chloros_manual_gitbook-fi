# Käsittelyn aloittaminen

Kun olet tuonut kuvat, merkinnyt kalibrointikohteet ja määrittänyt projektin asetukset, olet valmis aloittamaan käsittelyn. Tällä sivulla opastetaan Chloros-käsittelyprosessin aloittaminen.

## Esikäsittelyn tarkistuslista

Ennen kuin napsautat Käynnistä-painiketta, varmista, että kaikki on valmista:

* [ ] **Tiedostot tuotu** - Kaikki kuvat näkyvät tiedostoselaimessa
* [ ] **Kohdekuvat merkitty** - Kohdesarakkeessa on tarkistettu kalibrointikuvat
* [ ] **Kameramallit tunnistettu** - Kameramalli-sarakkeessa näkyvät oikeat kamerat
* [ ] **Asetukset määritetty** - Projektin asetukset tarkistettu ja säädetty
* [ ] **Indeksit valittu** - Halutut monispektriset indeksit lisätty (tarvittaessa)
* [ ] **Vienti-muoto valittu** - Työnkulullesi sopiva tulostusmuoto

{% vihje style=&quot;info&quot; %}
**Vinkki**: Napsauta muutamaa kuvaa tiedostoselaimessa varmistaaksesi, että ne on ladattu oikein ennen käsittelyä.
{% endhint %}

***

## Käsittelyn aloittaminen

### Etsi Käynnistä-painike

Käynnistä/Toista-painike sijaitsee Chloros:n yläreunan palkissa:

* Sijainti: Ikkunan yläosassa keskellä
* Kuvake: **Toista/Käynnistä-painike** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
* Tila: Painike on käytössä (kirkas), kun se on valmis käsittelyyn

### Aloita napsauttamalla

1. Napsauta **Toisto/Käynnistä-painiketta** yläreunassa
2. Käsittely alkaa välittömästi
3. Painike poistuu käytöstä (harmaana) käsittelyn aikana
4. Edistymispalkki päivittyy ja näyttää käsittelyn tilan

{% hint style=&quot;success&quot; %}
**Käsittely aloitettu**: Kun painiketta on napsautettu, Chloros käsittelee automaattisesti kaikki käsittelyvaiheet – kohteen tunnistuksen, debayeroinnin, kalibroinnin, indeksin laskennan ja viennin.
{% endhint %}

***

## Käsittelytilojen ymmärtäminen

Chloros toimii kahdessa eri käsittelytilassa lisenssistäsi riippuen:

### Ilmainen tila (peräkkäinen käsittely)

**Saatavilla kaikille käyttäjille**

**Kuinka se toimii:**

* Käsittelee kuvat yksi kerrallaan, peräkkäin
* Yksisäikeinen toiminta
* Pienempi muistin käyttö

**Edistymispalkki näyttää 2 vaihetta:**

1.**Kohteen tunnistus** - Kalibrointikohteiden skannaus
2. **Käsittely** - Kalibroinnin soveltaminen ja kuvien vienti**Käsittelyaika:**

* Paljon hitaampi kuin Chloros+ rinnakkaistila
* Sopii pienille ja keskisuurille tietojoukoille (&lt; 200 kuvaa)

### Chloros+ -tila (rinnakkaiskäsittely)

**Vaatii Chloros+ -lisenssin**

**Kuinka se toimii:**

* Käsittelee useita kuvia samanaikaisesti
* Monisäikeinen toiminta (jopa 16 rinnakkaista työntekijää)
* Hyödyntää useita CPU-ytimiä
* Valinnainen GPU (CUDA) -kiihdytys NVIDIA-näytönohjaimilla

**Edistymispalkki näyttää 4 vaihetta:**

1.**Tunnistaminen** - Kalibrointikohteiden etsiminen
2. **Analysointi** - Kuvan metatietojen tarkastelu ja putkilinjan valmistelu
3. **Kalibrointi** – Korjausten ja kalibrointien soveltaminen
4. **Vienti** – Käsiteltyjen kuvien ja indeksien tallentaminen**Edistymispalkin käyttö:*** **Vie hiiri** palkin päälle nähdäksesi yksityiskohtaisen 4-vaiheisen pudotusvalikon
* **Napsauta** edistymispalkkia jäädyttääksesi pudotusvalikon paikalleen
* **Napsauta uudelleen** vapauttaaksesi ja piilottaaksesi valikon**Käsittelyaika:**

* Huomattavasti nopeampi kuin vapaa tila
* Skaalautuu CPU-ytimien lukumäärän mukaan
* GPU-kiihdytys parantaa nopeutta entisestään

{% hint style=&quot;info&quot; %}
**Chloros+ Nopeus**: Rinnakkaiskäsittely voi olla 5–10 kertaa nopeampi kuin peräkkäinen tila suurille tietojoukoille. 500 kuvan projekti, joka kestää 2 tuntia ilmaisessa tilassa, voi valmistua 15–20 minuutissa Chloros+:lla.
{% endhint %}

***

## Mitä tapahtuu käsittelyn aikana

### Vaihe 1: Kohteen tunnistus

**Mitä Chloros tekee:**

* Skannaa merkityt kohdekuvat (tai kaikki kuvat, jos merkittyjä ei ole)
* Tunnistaa 4 kalibrointipaneelia jokaisessa kohteessa
* Poimii heijastavuusarvot kohdepaneeleista
* Tallentaa kohteen aikaleimat kalibroinnin aikatauluttamista varten

**Kesto:** 1–30 sekuntia (merkityillä kohteilla), 5–30+ minuuttia (merkitsemättömillä)

### Vaihe 2: Debayering (RAW-muunnos)

**Chloros:n toiminnot:**

* Muuntaa RAW-Bayer-kuvion tiedot täysiksi RGB-kuviksi
* Soveltaa korkealaatuista demosaicing-algoritmia
* Säilyttää kuvan laadun ja yksityiskohdat mahdollisimman hyvin

**Kesto:** Vaihtelee kuvien määrän ja prosessorin nopeuden mukaan

### Vaihe 3: Kalibrointi

**Mitä Chloros tekee:*** **Vignette-korjaus**: Poistaa objektiivin tummenemisen reunoilla
* **Heijastavuuskalibrointi**: Normalisoi käyttämällä kohteen heijastavuusarvoja
* Soveltaa korjauksia kaikkiin kaistoihin/kanaviin
* Käyttää sopivaa kalibrointikohdetta jokaiselle kuvalle aikaleiman perusteella

**Kesto:** Suurin osa käsittelyajasta

### Vaihe 4: Indeksin laskeminen

**Mitä Chloros tekee:**

* Laskee konfiguroidut monispektriset indeksit (NDVI, NDRE jne.)
* Soveltaa kaistamatematiikkaa kalibroituihin kuviin
* Luo indeksikuvat jokaiselle valitulle indeksille

**Kesto:** Muutama sekunti kuvaa kohti

### Vaihe 5: Vienti

**Mitä Chloros tekee:**

* Tallentaa kalibroidut kuvat valitussa muodossa
* Vie indeksikuvat konfiguroiduilla LUT-väreillä
* Kirjoittaa tiedostot kameramallin alikansioihin
* Säilyttää alkuperäiset tiedostonimet ja päätteet

**Kesto:** Vaihtelee vientimuodon ja tiedostokoon mukaan***

## Käsittelykäyttäytyminen

### Automaattinen käsittelyputki

Kun käsittely on aloitettu, koko putki toimii automaattisesti:

* Käyttäjän toimia ei tarvita
* Kaikki määritetyt vaiheet suoritetaan peräkkäin
* Edistyspäivitykset näkyvät reaaliajassa

### Tietokoneen käyttö käsittelyn aikana

**Vapaa tila:**

* Suhteellisen alhainen CPU-käyttö (yksisäikeinen)
* Tietokone pysyy reagoivana muihin tehtäviin
* Chloros-ohjelman voi turvallisesti minimoida ja työskennellä muissa sovelluksissa

**Chloros+ rinnakkaistila:**

* Korkea CPU-käyttö (monisäikeinen, jopa 16 ydintä)
* GPU-kiihdytyksellä: Korkea GPU-käyttö
* Tietokone voi olla vähemmän reagoiva käsittelyn aikana
* Vältä muiden CPU-intensiivisten tehtävien käynnistämistä

{% hint style=&quot;warning&quot; %}
**Suorituskykyvinkki**: Parhaan suorituskyvyn saavuttamiseksi sulje muut sovellukset ja anna Chloros:n käyttää kaikkia järjestelmän resursseja.
{% endhint %}

### Käsittelyä ei voi keskeyttää

**Tärkeitä rajoituksia:**

* Kun käsittely on aloitettu, sitä ei voi keskeyttää.
* Voit peruuttaa käsittelyn, mutta edistys menetetään.
* Osittaisia tuloksia ei tallenneta.
* Peruutuksen jälkeen on aloitettava alusta.

**Suunnitteluvinkki:** Erittäin suurissa projekteissa kannattaa harkita käsittelyä erissä tai CLI:n käyttöä paremman hallinnan saavuttamiseksi.***

## Käsittelyn seuranta

Käsittelyn aikana voit:

* **Tarkkailla edistymispalkkia** – Katso kokonaisvaltainen valmistumisprosentti
* **Tarkastella nykyistä vaihetta** – Tunnista, analysoi, kalibroi tai vie
* **Tarkastella lokivälilehteä** – Katso yksityiskohtaiset käsittelyviestit ja varoitukset
* **Esikatsella valmiita kuvia** – Jotkin vientitiedostot voivat näkyä käsittelyn aikana

Yksityiskohtaiset tiedot seurannasta ovat kohdassa [Käsittelyn seuranta](monitoring-the-processing.md).

***

## Käsittelyn peruuttaminen

Jos haluat lopettaa käsittelyn:

### Peruuttaminen

1. Etsi **Lopeta/Peruuta-painike** (korvaa Käynnistä-painikkeen käsittelyn aikana)
2. Napsauta Stop-painiketta
3. Käsittely keskeytyy välittömästi
4. Osittaiset tulokset hylätään

### Milloin peruuttaa

**Peruuttamisen pätevät syyt:**

* Huomasit, että asetukset olivat virheelliset
* Unohdit merkitä kohdekuvat
* Tuotiin vääriä kuvia
* Järjestelmä toimii liian hitaasti tai ei vastaa

**Peruuttamisen jälkeen:**

* Tarkista ja korjaa mahdolliset ongelmat
* Säädä asetuksia tarpeen mukaan
* Käynnistä käsittely alusta
* Parhaan käyttökokemuksen saamiseksi sulje Chloros kokonaan ja käynnistä se uudelleen

{% hint style=&quot;warning&quot; %}
**Ei osittaisia tuloksia**: Peruuttaminen hylkää kaiken edistymisen. Chloros ei tallenna osittain käsiteltyjä kuvia.
{% endhint %}

***

## Käsittelyajan arviointi

Todellinen käsittelyaika vaihtelee suuresti seuraavien tekijöiden mukaan:

* Kuvien määrä
* Kuvan resoluutio
* RAW- tai JPG-tiedostomuoto
* Käsittelytila (ilmainen vs. Chloros+)
* CPU:n nopeus ja ytimien lukumäärä
* GPU:n saatavuus (vain Chloros+)
* Laskettavien indeksien lukumäärä
* Vientimuodon monimutkaisuus

### Karkeat arviot (Chloros+, 12 MP:n kuvat, moderni CPU)

| Kuvien lukumäärä | Ilmainen tila | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 kuvaa   | 15–20 min | 5–8 min        | 3–5 min        |
| 100 kuvaa  | 30–40 min | 10–15 min      | 5–8 min        |
| 200 kuvaa  | 1–1,5 tuntia | 20–30 min      | 10–15 min      |
| 500 kuvaa  | 2–3 tuntia   | 45–60 min      | 20–30 min      |
| 1000 kuvaa | 4–6 tuntia   | 1,5–2 tuntia      | 40–60 min      |

{% hint style=&quot;info&quot; %}
**Ensimmäinen käyttökerta**: Alustava käsittely voi kestää kauemmin, koska Chloros luo välimuistit ja profiilit. Samanlaisten tietojoukkojen myöhempi käsittely on nopeampaa.
{% endhint %}

***

## Yleisiä ongelmia käynnistyksen yhteydessä

### Käynnistyspainike ei ole käytettävissä (harmaana)

**Mahdolliset syyt:**

* Kuvia ei ole tuotu
* Taustaprosessi ei ole käynnistynyt kokonaan
* Edellinen käsittely on vielä käynnissä
* Projekti ei ole ladattu kokonaan

**Ratkaisut:**

1. Odota, että taustaohjelma käynnistyy kokonaan (tarkista päävalikon kuvake)
2. Varmista, että kuvat on tuotu tiedostoselaimessa
3. Käynnistä Chloros uudelleen, jos painike on edelleen pois käytöstä
4. Tarkista virheilmoitukset virhelokista

### Käsittely alkaa, mutta epäonnistuu välittömästi

**Mahdolliset syyt:**

* Projektissa ei ole kelvollisia kuvia
* Vioittuneet kuvatiedostot
* Riittämätön levytila
* Riittämätön muisti (RAM)

**Ratkaisut:**

1. Tarkista virheloki <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> virheilmoitusten varalta
2. Tarkista käytettävissä oleva levytila
3. Yritä käsitellä pienempiä kuvajoukkoja
4. Tarkista, että kuvat eivät ole vioittuneet

### Varoitus &quot;Kohteita ei havaittu&quot;

**Mahdolliset syyt:**

* Kohdekuvien merkitseminen on unohdettu
* Kohdekuvat eivät sisällä näkyviä kohteita
* Kohteiden tunnistuksen asetukset ovat liian tiukat

**Ratkaisut:**

1. Tarkista [Kohdekuvien valinta](choosing-target-images.md)
2. Merkitse sopivat kuvat Kohde-sarakkeeseen
3. Varmista, että kohteet ovat näkyvissä merkityissä kuvissa
4. Säädä kohteen tunnistuksen asetuksia tarvittaessa

***

## Vinkkejä onnistuneeseen käsittelyyn

### Ennen aloittamista

1. **Testaa ensin pienellä osajoukolla** - Käsittele 10–20 kuvaa asetusten tarkistamiseksi
2. **Tarkista käytettävissä oleva levytila** - Varmista, että vapaata tilaa on 2–3 kertaa datajoukon koko
3. **Sulje tarpeettomat sovellukset** - Vapauta järjestelmän resursseja
4. **Tarkista kohdekuvat** - Esikatsele merkityt kohteet laadun varmistamiseksi
5. **Tallenna projekti** - Projekti tallentuu automaattisesti, mutta on hyvä tallentaa se manuaalisesti

### Käsittelyn aikana

1. **Vältä järjestelmän lepotilaa** - Poista virransäästötilat käytöstä
2. **Pidä Chloros etualalla** - Tai ainakin näkyvissä tehtäväpalkissa
3. **Seuraa edistymistä ajoittain** - Tarkista varoitukset tai virheet
4. **Älä lataa muita raskaita sovelluksia** - Erityisesti Chloros+ rinnakkaistilassa

### Chloros+ GPU-kiihdytys

Jos käytät NVIDIA GPU-kiihdytystä:

1. Päivitä NVIDIA-ajurit uusimpaan versioon
2. Varmista, että GPU:ssa on vähintään 4 Gt VRAM-muistia
3. Sulje GPU:ta paljon kuormittavat sovellukset (pelit, videonmuokkaus)
4. Tarkkaile GPU:n lämpötilaa (varmista riittävä jäähdytys)

***

## Seuraavat vaiheet

Kun käsittely on alkanut:

1. **Seuraa edistymistä** - Katso [Käsittelyn seuraaminen](monitoring-the-processing.md)
2. **Odota käsittelyn päättymistä** – Käsittely suoritetaan automaattisesti.
3. **Tarkista tulokset** – Katso [Käsittelyn lopettaminen](finishing-the-processing.md).

Lisätietoja käsittelyn aikana suoritettavista toimista on kohdassa [Käsittelyn seuranta](monitoring-the-processing.md).
