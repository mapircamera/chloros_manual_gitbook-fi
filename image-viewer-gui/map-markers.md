# Karttamerkit

Kartta-välilehti näyttää kuvasi interaktiivisella 2D-kartalla niiden GPS-koordinaattien perusteella. Tämä tarjoaa maantieteellisen yleiskuvan kuvausistunnosta ja auttaa visualisoimaan alueellisen kattavuuden. Se on myös hyödyllinen, kun tuot kuvasi ensimmäisen kerran, jotta voit nopeasti poistaa kuvat, joita et tarvitse käsitellä.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Kartta-välilehden avaaminen

1. Avaa tai luo projekti Chloros:ssä.
2. Tuo kuvat, jotka sisältävät GPS-metatietoja.
3. Napsauta **Kartta** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> -välilehteä vasemmassa sivupalkissa.
4. Kartta näyttää merkit kunkin kuvan GPS-sijainnissa.

{% hint style="info" %}
**GPS vaaditaan**: Kartalla näkyvät vain kuvat, joiden EXIF-metatiedoissa on upotetut GPS-koordinaatit. Varmista, että kamerassasi on GPS käytössä kuvauksen aikana.
{% endhint %}

***

## Kuvien säätäminen Kartta-välilehdeltä

**Kartta** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> -välilehdessä on samat lisäys-  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  ja poisto  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  -painikkeet kuin [**Tiedostoselain**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> -välilehdessä. Se näyttää myös saman projektitiedostotaulukon, mutta eri sarakkeiden otsikoilla:

### Tiedostonimi

* Kameran alkuperäinen tiedostonimi
* Säilyttää kameran nimeämiskäytännön (esim. IMG\_0001.RAW)

### Leveysaste

* Kuvan leveysaste

### Pituusaste

* Kuvan pituusaste

### Korkeus

* Kuvan korkeus

{% hint style="info" %}
Taulukon sarakkeiden otsikoita napsauttamalla voit myös lajitella rivien tiedot
{% endhint %}

***

## Kuvan merkit

Jokainen kuva, jossa on GPS-tietoja, on merkitty kartalle merkillä:

### Merkkien näyttö

* Merkkien avulla näkyy kunkin kuvan tarkat GPS-koordinaatit
* Ryhmitetyt merkit voivat näkyä yhdessä, kun karttaa pienennetään
* Zoomaa karttaa nähdäksesi yksittäisten kuvien sijainnit

{% hint style="success" %}
SUPERZOOM: Kun karttalaattojen tarjoajan suurin zoomausaste on saavutettu, laattaa suurennetaan edelleen, jolloin voit nähdä lähellä toisiaan olevat merkit.
{% endhint %}

### Esikatselu hiiren osoittimella

* **Vie hiiren osoitin** minkä tahansa merkin päälle nähdäksesi pienen esikatselukuvan kyseisestä kuvasta.
* Tämä mahdollistaa nopean visuaalisen tunnistamisen poistumatta karttanäkymästä.
* Hyödyllinen tiettyjen kuvien paikantamiseen suuresta kuvaussessiosta.

***

## Karttatiilipalvelujen tarjoajat

{% hint style="success" %}
**Automaattinen valinta**: Chloros valitsee automaattisesti ruutupalvelun, joka tarjoaa parhaan zoomausasteen nykyiselle kartan sijainnille. Voit vaihtaa palveluntarjoajaa manuaalisesti, jos haluat.
{% endhint %}

Kartta-välilehti tukee kahta ruutupalveluntarjoajaa taustakartan kuville:

### Google Maps

* Googlen tavalliset satelliitti- ja karttakuvat
* Paras yleiseen maailmanlaajuiseen kattavuuteen

### ESRI

* ESRI ArcGIS:n satelliitti- ja ilmakuvat
* Tarjoaa usein korkeamman resoluution kuvia tietyillä alueilla

***

## Karttatiilityypit

Voit valita karttakerroksen tyypin (vasemmalta oikealle):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="line">### Maasto

Näyttää korkeusprofiilit ja karttalaatat yksityiskohdilla (tiet jne.)

### Kartta

Näyttää tavalliset (pienemmän kaistanleveyden) karttalaatat yksityiskohdilla (tiet jne.)

### Satelliitti

Näyttää yksityiskohtaiset (suuremman kaistanleveyden) satelliittikarttalaatat

### Hybridi

Näyttää satelliittikarttalaatat lisäyksillä (tiet jne.)

***

## Kartan navigointi

### Zoomaustoiminnot

* **Lähennä/loitonna**: Käytä hiiren rullaa tai zoomauspainikkeita.
* **Koko näyttö**: Näytä kartta koko näytöllä.

### Panoraamatoiminnot

* **Panoraama**: Napsauta ja vedä hiiren painikkeita liikutellaksesi karttaa.

***

## Käyttötapaukset

### Lentoreitin visualisointi

* Tarkastele drone-kuvausistuntojen peittoaluetta
* Tunnista kuvien peittoalueen aukot
* Tarkista lentoreitin toteutuminen

### Maastokartoituksen tarkastelu

* Tarkastele maastokartoituksen kuvien alueellista jakautumista
* Paikanna kalibrointikohteiden kuvat suhteessa kartoitusalueeseen
* Suunnittele lisäkuvauspaikat

### Laadunvalvonta

* Tunnista nopeasti odottamattomissa paikoissa otetut kuvat.
* Tarkista GPS:n tarkkuus koko tietojoukossa.
* Vertaa kuvien sijainteja kenttämuistiinpanoihin.

***

## Vianmääritys

### Merkkejä ei näy

**Mahdolliset syyt:**

* Kuvat eivät sisällä GPS-metatietoja.
* GPS oli pois käytöstä kamerassa kuvauksen aikana.
* EXIF-tiedot on poistettu ulkoisella ohjelmistolla.

**Ratkaisu**: Varmista, että GPS on käytössä kamerassa, ja tuo alkuperäiset tiedostot uudelleen.

### Merkinnät väärässä paikassa

**Mahdolliset syyt:**

* Kameran GPS:n satelliittisignaalin vastaanotto oli heikko.
* GPS:n signaali heilahti kuvauksen aikana.

**Ratkaisu**: Tämä on tyypillisesti kuvausajankohtaan liittyvä ongelma. Harkitse PPK/RTK GPS:n käyttöä tarkkuutta vaativissa sovelluksissa.
