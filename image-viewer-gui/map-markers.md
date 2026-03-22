# Karttamerkit

Kartta-välilehdessä kuvasi näkyvät interaktiivisella 2D-kartalla niiden GPS-koordinaattien perusteella. Tämä tarjoaa maantieteellisen yleiskuvan kuvausistunnosta ja auttaa visualisoimaan kuvausalueen. Se on hyödyllinen myös kuvia ensimmäistä kertaa tuotaessa, jotta voit nopeasti poistaa kuvat, joita ei tarvitse käsitellä.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Kartta-välilehden avaaminen

1. Avaa tai luo projekti Chloros-ohjelmassa
2. Tuo kuvat, jotka sisältävät GPS-metatietoja
3. Napsauta **Kartta** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> -välilehteä vasemmassa sivupalkissa
4. Kartta näyttää merkit kunkin kuvan GPS-sijainnissa

{% hint style="info" %}
**GPS vaaditaan**: Kartalle näkyvät vain kuvat, joiden EXIF-metatietoihin on upotettu GPS-koordinaatit. Varmista, että kamerasi GPS-toiminto on käytössä kuvauksen aikana.
{% endhint %}

***

## Kuvien säätäminen Kartta-välilehdellä**Kartta**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> -välilehdessä on sama lisää  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  ja poista  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  -painikkeet kuin [**Tiedostoselaimessa**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> -välilehdessä. Siinä näkyy myös sama projektitiedostojen taulukkoluettelo, mutta eri sarakkeiden otsikoilla:

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
Taulukon sarakkeiden otsikoita klikkaamalla voit myös lajitella rivitiedot
{% endhint %}

***

## Kuvamerkit

Jokainen kuva, jossa on GPS-tietoja, näkyy kartalla merkkinä:

### Merkkien näyttö

* Merkit osoittavat kunkin kuvan tarkat GPS-koordinaatit
* Ryhmitetyt merkit voivat ryhmittyä yhteen, kun karttaa pienennetään
* Suurenna karttaa nähdäksesi yksittäisten kuvien sijainnit

{% hint style="success" %}
SUPER-ZOOM: Kun saavutat karttalaattojen tarjoajan suurimman zoomausasteen, laatta suurennetaan edelleen, jolloin voit nähdä lähekkäin olevat merkit.
{% endhint %}

### Esikatselu hiiren osoittamalla

* **Vie hiiri** minkä tahansa merkin päälle nähdäksesi pienen esikatselukuvan kyseisestä kuvasta
* Tämä mahdollistaa nopean visuaalisen tunnistamisen poistumatta karttanäkymästä
* Hyödyllinen tiettyjen kuvien paikantamiseen suuressa kuvaussessiossa

***

## Karttalaattojen tarjoajat

{% hint style="success" %}
**Automaattinen valinta**: Chloros valitsee automaattisesti ruutupalvelun, joka tarjoaa parhaan zoomausasteen nykyiselle karttapaikallesi. Voit halutessasi vaihtaa palveluntarjoajaa manuaalisesti.
{% endhint %}

Kartta-välilehti tukee kahta ruutupalveluntarjoajaa taustakarttakuville:

### Google Maps

* Googlen vakiomuotoiset satelliitti- ja karttakuvat
* Paras yleinen maailmanlaajuinen kattavuus

### ESRI

* ESRI ArcGISin satelliitti- ja ilmakuvat
* Tarjoaa usein korkeamman resoluution kuvia tietyillä alueilla

***

## Karttalaattatyypit

Voit valita karttakerroksen tyypin (vasemmalta oikealle):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="original">### Maasto

Näyttää korkeusprofiilit ja karttalaatat yksityiskohdilla (tiet jne.)

### Kartta

Näyttää tavalliset (pienemmällä kaistanleveydellä toimivat) karttalaatat yksityiskohdilla (tiet jne.)

### Satelliitti

Näyttää yksityiskohtaiset (suuremmalla kaistanleveydellä toimivat) satelliittikarttalaatat

### Hybridi

Näyttää satelliittikarttalaatat, joihin on lisätty yksityiskohtia (tiet jne.)

***

## Kartan navigointi

### Zoomaustyökalut

* **Lähentäminen/loitontaminen**: Käytä hiiren rullapyörää tai zoomauspainikkeita
* **Koko näyttö**: Näytä kartta koko näytöllä

### Panorointityökalut

* **Panorointi**: Napsauta ja vedä hiirtä liikkuaksesi kartalla***

## Käyttötapaukset

### Lentoreitin visualisointi

* Tarkastele drone-kuvausistuntojen kattamaa aluetta
* Tunnista aukkoja kuvien kattavuudessa
* Tarkista lentoreitin toteutuminen

### Maastotutkimuksen tarkastelu

* Katso maastossa otettujen kuvien alueellinen jakautuminen
* Paikanna kalibrointikohdekuvat suhteessa tutkimusalueeseen
* Suunnittele lisäkuvauspaikkoja

### Laadunvalvonta

* Tunnista nopeasti odottamattomissa paikoissa otetut kuvat
* Tarkista GPS-tarkkuus koko aineistossa
* Vertaa kuvien sijainteja kenttämuistiinpanoihin

***

## Vianmääritys

### Merkkejä ei näy

**Mahdolliset syyt:**

* Kuvissa ei ole GPS-metatietoja
* GPS oli pois päältä kamerassa kuvauksen aikana
* EXIF-tiedot on poistettu ulkoisella ohjelmistolla

**Ratkaisu**: Varmista, että GPS on käytössä kamerassasi, ja tuo alkuperäiset tiedostot uudelleen

### Merkit väärässä paikassa

**Mahdolliset syyt:**

* Kameran GPS:n satelliittisignaali oli heikko
* GPS-signaali ajautui kuvauksen aikana

**Ratkaisu**: Tämä on tyypillisesti kuvaushetkeen liittyvä ongelma; harkitse PPK/RTK-GPS:n käyttöä tarkkuutta vaativissa sovelluksissa
