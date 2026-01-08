# Tiedostojen lisääminen projektiin

Kun olet luonut tai avannut projektin Chloros:ssä, seuraava vaihe on lisätä monispektrikuvat käsittelyä varten. Tiedostoselaimen<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> -välilehti helpottaa kuvien tuomista ja tietojoukkojen hallintaa.

## Tiedostoselaimen avaaminen

1. Avaa tai luo projekti Chloros-ohjelmassa
2. Napsauta **Tiedostoselain** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> -kuvaketta vasemmassa sivupalkissa
3. Tiedostoselaimen paneeli näyttää projektisi tiedostoluettelon

{% hint style="info" %}
**Tuetut tiedostotyypit**: Chloros tukee RAW+JPG- ja JPG-kuvatiedostoja MAPIR Survey3W ja Survey3N kameroista. Suosittelemme vain RAW+JPG-tiedostoja.
{% endhint %}

***

## Kuvien lisääminen projektiin

Kuvia voi lisätä projektiin kahdella tavalla:

### Tapa 1: Lisää tiedostot

Käytä tätä vaihtoehtoa, jos haluat tuoda yksittäisiä kuvatiedostoja tai pienen valikoiman tiedostoja.

1. Napsauta **&quot;Lisää tiedostot&quot;** <img src="../.gitbook/assets/image.png" alt="" data-size="line"> -painiketta tiedostoselaimen paneelin yläosassa
2. Siirry kansioon, joka sisältää kuvasi
3. Valitse yksi tai useampi kuvatiedosto (pidä **Ctrl**-näppäintä painettuna valitaksesi useita tiedostoja)
4. Napsauta **&quot;Avaa&quot;** tuodaksesi valitut tiedostot

### Tapa 2: Lisää kansio

Käytä tätä vaihtoehtoa tuodaksesi kaikki kuvat kansiosta kerralla.

1. Napsauta **&quot;Lisää kansio&quot;** <img src="../.gitbook/assets/image (1).png" alt="" data-size="line"> -painiketta tiedostoselaimen paneelin yläosassa.
2. Siirry kansioon, joka sisältää kuvausistunnon kuvat, ja valitse se.
3. Napsauta **&quot;Valitse kansio&quot;** tuodaksesi kaikki tuetut kuvat kyseisestä kansiosta.

***

## Tiedostoselaimen taulukon ymmärtäminen

Kun kuvat on tuotu, ne näkyvät taulukossa, jossa on seuraavat sarakkeet:

### Tiedostonimi

* Kameran alkuperäinen tiedostonimi
* Säilyttää kameran nimeämiskäytännön (esim. IMG\_0001.RAW)

### Aikaleima

* Kuvan ottamisen päivämäärä ja kellonaika
* Otettu kuvan EXIF-metatiedoista
* Käytetään PPK-synkronointiin ja kalibrointikohteen tunnistamiseen

### Kameramalli

* Automaattisesti tunnistettu kamera ja suodatinkonfiguraatio
* Esimerkkejä: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Käytetään oikeiden käsittelyprofiilien soveltamiseen

### Kohdesarake (valintaruutu)

* Valitse tämä valintaruutu, jos kuvat sisältävät kalibrointikohteita
* Nopeuttaa huomattavasti kohteen tunnistusta käsittelyn aikana
* Katso lisätietoja kohdasta [Kohdekuvien valinta](choosing-target-images.md)

### Kuvan metatietojen tarkastelu

Napsauttamalla taulukon oikeassa yläkulmassa olevaa vaihtopainiketta voit näyttää valitun kuvan metatiedot kuvaruudukossa.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Projektin tiedostojen hallinta

### Tiedostojen poistaminen

Poista projektista tarpeettomat kuvat seuraavasti:

1. Valitse yksi tai useampi kuva tiedostoselaimen taulukosta
2. Napsauta **&quot;Poista valitut&quot;** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line"> -painiketta.
3. Vahvista poisto (tiedostot eivät poistu levyltä, vaan vain projektista).

### Lajittelu ja suodatus

* **Lajittele sarakkeen mukaan**: Napsauta mitä tahansa sarakkeen otsikkoa lajitellaksesi kuvat.
* **Aikaleimasuodatus**: Hyödyllinen kronologisten kuvaussekvenssien järjestämisessä.
* **Kameramallisuodatin**: Ryhmittele kuvat kameratyypin mukaan, jos käytät useita kameroita.

***

## Kuvan esikatselu

### Kuvan katseleminen kokonaan

Napsauta mitä tahansa kuvan pikkukuvaa tiedostoselaimessa, jotta se näkyy pääesikatselualueella:

1. Kuva näkyy keskimmäisessä esikatselupaneelissa.
2. Tarkista kuvan yksityiskohdat zoomauspainikkeilla.
3. Siirry kuvien välillä nuolinäppäimillä.

### Pikanavigointi

* **Edellinen kuva**: Napsauta vasenta nuolta tai paina ←-näppäintä.
* **Seuraava kuva**: Napsauta oikeaa nuolta tai paina →-näppäintä
* **Lähennä/loitonna**: Käytä hiiren rullaa tai zoomauspainikkeita
* **Panoroida**: Napsauta ja vedä kuvaa, kun se on lähennetty

***

## Duplikaattitiedostojen käsittely

Chloros tunnistaa ja ohittaa automaattisesti duplikaattitiedostot:

* Tiedostot, joilla on identtiset tiedostonimet, ohitetaan
* Estää vahingossa tapahtuvan kaksinkertaisen käsittelyn
* Varoitusviesti näkyy, kun kaksoiskappaleita havaitaan

{% hint style="warning" %}
**Tärkeää**: Älä nimeä uudelleen tai muokkaa alkuperäisiä kuvatiedostoja ennen tuontia. Chloros käyttää alkuperäisiä tiedostonimiä ja metatietoja oikean käsittelyn varmistamiseksi.
{% endhint %}

***

## Sekalaiset kameratiedot

Jos projektisi sisältää kuvia useista MAPIR-kameroista:

1. Chloros tunnistaa automaattisesti kunkin kameramallin
2. Kukin kameratyyppi käsitellään sen mukaisella kalibrointiprofiililla
3. Tiedostoselain näyttää kameramallin Kameramalli-sarakkeessa
4. Käsittely soveltaa oikeita asetuksia kullekin kameratyypille

**Esimerkkiskenaario**: Survey3W RGN + Survey3N OCN kaksoiskamerajärjestelmä

***

## Parhaat käytännöt

### Järjestä ennen tuontia

* Säilytä kalibrointikohdekuvat samassa kansiossa kuin mittauskuvat
* Säilytä kameran/SD-kortin alkuperäinen kansiorakenne
* Älä sekoita eri istuntojen tietojoukkoja yhteen projektiin

### Tiedostojen nimeäminen

* Säilytä kameran alkuperäiset tiedostonimet (IMG\_0001.RAW jne.)
* Älä nimeä tiedostoja uudelleen ennen tuontia
* Alkuperäiset nimet sisältävät tärkeitä metatietoja.

### Kalibrointikohdekuvat

* Lisää aina 1–2 kalibrointikohdekuvaa istuntoa kohti.
* Ota kuvat ennen ja jälkeen kuvausistunnon.
* Aseta kohteet samoihin valaistusolosuhteisiin kuin kuvausalue.
* Merkitse kohdekuvat Target-valintaruudulla käsittelyn nopeuttamiseksi.

***

## Yleisiä ongelmia ja ratkaisuja

### Kuvat eivät näy tuonnin jälkeen

**Mahdolliset syyt:**

* Tiedostomuotoa ei tueta (vain RAW+JPG ja JPG MAPIR-kameroista)
* Kuvat ovat peräisin muista kuin MAPIR-kameroista (katso [Tuetut kamerat](../supported-cameras.md))
* Tiedosto on vioittunut tai siirto SD-kortilta on jäänyt kesken

**Ratkaisu**: Tarkista tiedostomuoto ja kameramallin yhteensopivuus

### Kameramallia ei tunnistettu

**Mahdolliset syyt:**

* Muokatut EXIF-metatiedot
* Ulkopuolisella ohjelmistolla muokatut kuvat
* Keskeneräinen tiedostonsiirto

**Ratkaisu**: Tuo alkuperäiset, muokkaamattomat tiedostot uudelleen kamerasta/SD-kortilta

### Puuttuvat aikaleimat

**Mahdolliset syyt:**

* Kameran kello ei ole asetettu oikein
* EXIF-tiedot poistettu ulkoisella ohjelmistolla

**Ratkaisu**: Tarkista, että kameran aika-asetukset olivat oikeat kuvan ottamisen aikana

***

## Seuraavat vaiheet

Kun tiedostot on tuotu:

1. **Tarkista tiedostoluettelo** - Varmista, että kaikki kuvat on ladattu oikein
2. **Tarkista kameramallit** – Varmista, että kamera on tunnistettu oikein
3. **Merkitse kohdekuvat** – Katso [Kohdekuvien valinta](choosing-target-images.md)
4. **Säädä asetuksia** – Määritä käsittelyasetukset kohdassa [Projektin asetukset](adjusting-project-settings.md)
5. **Aloita käsittely** – Katso [Käsittelyn aloittaminen](starting-the-processing.md)

Yksityiskohtaiset tiedot projektin konfiguroinnista ovat kohdassa [Projektin asetusten säätäminen](adjusting-project-settings.md).
