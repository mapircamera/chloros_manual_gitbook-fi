# Tiedostojen lisääminen projektiin

Kun olet luonut tai avannut projektin Chloros-ohjelmassa, seuraava vaihe on lisätä monispektrikuvat käsittelyn aloittamiseksi. Tiedostoselaimen <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">-välilehden avulla kuvien tuominen ja aineiston hallinta on helppoa.

## Tiedostoselaimen avaaminen

1. Avaa tai luo projekti Chloros-ohjelmassa
2. Napsauta vasemmassa sivupalkissa olevaa **Tiedostoselain**-kuvaketta <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
3. Tiedostoselain-paneeliin avautuu projektisi tiedostoluettelo

{% hint style="info" %}
**Tuetut tiedostotyypit**:

* **Survey3W / Survey3N**: RAW+JPG-parit ja JPG-kuvat (suositus: RAW+JPG)
* **LATTICE**: `.tif` / `.tiff`-tallenteet — tallennettu Chloros-kameranohjauksella tai LATTICE-keskittimellä
* **Valosensoritiedot**: `.daq`-tallenteet (DAQ-U/M/E) ja DAQ-M `.csv`-alasuuntaiset lokit — tuodaan kuvamateriaalin mukana heijastavuuden kalibrointia varten
{% endhint %}

***

## Kuvien lisääminen projektiin

Kuvia voi lisätä projektiin pääasiassa kahdella tavalla:

### Tapa 1: Lisää tiedostoja

Käytä tätä vaihtoehtoa yksittäisten kuvatiedostojen tai pienen tiedostovalikoiman tuomiseen.

1. Napsauta **&quot;Lisää tiedostoja&quot;** -painiketta (<img src="../.gitbook/assets/image (3).png" alt="" data-size="line">) tiedostoselaimen paneelin yläosassa
2. Siirry kansioon, jossa kuvasi sijaitsevat
3. Valitse yksi tai useampi kuvatiedosto (pidä **Ctrl**-näppäintä painettuna valitaksesi useita tiedostoja)
4. Napsauta **&quot;Avaa&quot;** tuodaksesi valitut tiedostot

### Tapa 2: Lisää kansio

Käytä tätä vaihtoehtoa, kun haluat tuoda kaikki kuvat kansiosta kerralla. Voit valita **useita kansioita** yhdessä valintaikkunassa.

1. Napsauta **&quot;Lisää kansio&quot;**-painiketta (<img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line">) tiedostoselaimen paneelin yläosassa
2. Siirry kansioon tai kansioihin, joissa kuvaussession kuvat sijaitsevat, ja valitse ne
3. Napsauta **&quot;Valitse kansio&quot;** tuodaksesi kaikki tuetut kuvat

{% hint style="info" %}
**Tiedostot, joita ei onnistuta lataamaan, raportoidaan.** Jos kansio sisältää tiedostoja, jotka Chloros tunnistaa mutta joita se ei voi ladata, saat siitä varoituksen — kuvat eivät katoa ruudukosta huomaamatta.
{% endhint %}

***

## LATTICE-kaappauskansioiden tuominen

LATTICE-kaappaukset tallennetaan **yhdellä alikansiolla kutakin vientitasoa kohti** — esimerkiksi `raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` — ja vastaava `.daq`-alasvirtaustiedosto juurikansiossa:

```
output/
├── raw/           capture_<timestamp>_SN<serial>_raw.tif
├── debayered/     capture_<timestamp>_SN<serial>_debayered.tif
├── preview/       capture_<timestamp>_SN<serial>_display.tif
└── *.daq          the downwelling reading matched to the capture
```

**Valitse Lisää kansio -toiminto kuvien juurikansiossa** (edellä mainittu `output/`). Kun valitussa kansiossa ei itsessään ole kuvia, mutta siinä on alikansioita, Chloros siirtyy niihin automaattisesti — kyseisen tason alikansiot ja juurikansio `.daq` haetaan kaikki kerralla.**Kuinka kuvat tuodaan:*** Jokainen kuvasarja tuodaan **yhtenä kuvana**, ryhmiteltynä kuvasarjan mukaan (ei yhtä merkintää tasoa kohti). Saman kuvasarjan muut tasot näkyvät kyseisen kuvan katselutilana.
* **Käsittely alkaa aina raakakuvasta.** Muut tasot ovat näkyvissä, mutta vain `raw` syötetään prosessointiputkeen — jo käsitellyn tuotteen uudelleenkäsittely johtaisi korjausten kaksinkertaiseen soveltamiseen, joten Chloros hylätään. Uudelleen tuotu vienti ei voi koskaan ottaa kaappauksen raakakuvapaikkaa.
* Kaappauskansio, joka on tallennettu **ilman** raakakuvien tuontia, näkyy normaalisti, mutta käsittely ohittaa sen ja ilmoittaa siitä lokissa. (CLI-lippu `--input-level` voi pakottaa aloituskohdan tässä tapauksessa — katso [CLI-viite](../reference/cli-reference.md#what-a-captures-folder-looks-like).)**LATTICE-keskittimen istunnot** tuodaan samalla tavalla: valitse Lisää kansio -kohdasta hubista kopioitu istuntokansio (se sisältää `raw/` ja `previews/`) sekä mahdolliset DAQ-M `.csv` -laskeutumislokit. Jos kameran tai DAQ:n kalibrointitietoja ei ole vielä tallennettu koneesi välimuistiin, Chloros hakee ne automaattisesti sarjanumeron perusteella tuonnin yhteydessä (vaatii internet-yhteyden kerran).***

## Tiedostoselaimen taulukon ymmärtäminen

Kun kuvat on tuotu, ne näkyvät taulukossa, jossa on seuraavat sarakkeet:

### Tiedostonimi

* Kameran alkuperäinen tiedostonimi
* Säilyttää kameran nimeämiskäytännön (esim. IMG\_0001.RAW tai capture\_20260816\_101500\_SN213800234\_raw.tif)

### Aikaleima

* Kuvan ottamispäivä ja -aika
* Poimittu kuvan EXIF-metatiedoista
* Käytetään valosensorin täsmäyttämiseen, PPK-synkronointiin ja kalibrointikohteiden aikataulutukseen

### Kameramalli

* Automaattisesti tunnistettu kamera- ja suodatinkonfiguraatio
* Survey3-esimerkkejä: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* LATTICE-esimerkit: LATT-M3M-L41-F550, LATT-M3C-L87-FRGN
* Käytetään oikeiden käsittelyprofiilien soveltamiseen

### Kohde-sarake (valintaruutu)

* Valitse tämä valintaruutu kuville, jotka sisältävät kalibrointikohteita
* Kun vähintään yksi kuva on valittuna, **vain valitut kuvat skannataan** kohteiden löytämiseksi
* Katso lisätietoja kohdasta [Kohdekuvien valinta](choosing-target-images.md)

### Kuvan metatietojen tarkastelu

Napsauttamalla taulukon yläkulmassa oikealla olevaa kytkinpainiketta saat valitun kuvan metatiedot näkyviin kuvaruudukkoalueelle.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Projektisi valosensoritiedostot

* `.daq`- ja `.csv`-tiedostot näkyvät tiedostoselaimen luettelossa, mutta niitä ei voi klikata kuvina — ne sisältävät alaspäin suuntautuvan säteilyn voimakkuuden heijastuskalibrointia varten.
* Jokainen tuotu `.daq`/`.csv`-tiedosto näkyy kohdassa **Projektin asetukset → DAQ-valosensori**, jossa voit tarkistaa kunkin tiedoston voimassa olevan hajotinläpän korjauksen. Katso [Projektiasetusten säätäminen](adjusting-project-settings.md).
* **Valosensorit**-välilehdessä tekemäsi tallenteet lisätään avoimeen projektiin automaattisesti – manuaalista tuontia ei tarvita.***

## Tiedostojen hallinta projektissasi

### Tiedostojen poistaminen

Poistaaksesi tarpeettomat kuvat projektistasi:

1. Valitse yksi tai useampi kuva Tiedostoselaimen taulukosta
2. Napsauta **&quot;Poista valitut&quot;**-painiketta <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
3. Vahvista poisto (tiedostoja ei poisteta levyltä, vaan ne poistetaan vain projektista)

### Lajittelu ja suodatus

* **Lajittele sarakkeen mukaan**: Napsauta mitä tahansa sarakkeen otsikkoa lajitellaksesi kuvat
* **Lajittelu aikaleiman mukaan**: Hyödyllinen kronologisten kuvausjaksojen järjestämiseen
* **Kameramallisuodatin**: Ryhmittele kuvat kameratyypin mukaan, jos käytät useita kameroita***

## Kuvan esikatselu

### Kuvan katselu täysikokoisena

Napsauta mitä tahansa kuvan pikkukuvaa tiedostoselaimessa, jolloin se avautuu pääesikatselualueelle:

1. Kuva näkyy keskellä olevassa esikatselupaneelissa
2. Tarkastele kuvan yksityiskohtia zoomauspainikkeiden avulla
3. Siirry kuvasta toiseen nuolinäppäimillä

### Pikanavigointi

* **Edellinen kuva**: Napsauta vasenta nuolta tai paina ←-näppäintä
* **Seuraava kuva**: Napsauta oikeaa nuolta tai paina →-näppäintä
* **Lähentäminen/loitontaminen**: Käytä hiiren rullaa tai zoomauspainikkeita
* **Panorointi**: Napsauta ja vedä kuvaa, kun se on suurennettuna***

## Kaksoiskappaleiden käsittely

Chloros tunnistaa ja ohittaa kaksoiskappaleet automaattisesti:

* Tiedostot, joiden nimet ovat identtiset, ohitetaan
* Estää tahattoman kaksinkertaisen käsittelyn
* Varoitusviesti näkyy, kun kaksoiskappaleita havaitaan

{% hint style="warning" %}
**Tärkeää**: Älä nimeä uudelleen tai muokkaa alkuperäisiä kuvatiedostoja ennen tuontia. Chloros käyttää alkuperäisiä tiedostonimiä ja metatietoja oikean käsittelyn varmistamiseksi.
{% endhint %}

***

## Sekalaiset kameratietojoukot

Jos projektisi sisältää kuvia useista MAPIR-kameroista:

1. Chloros tunnistaa automaattisesti kunkin kameramallin — Survey3, LATTICE tai niiden yhdistelmä
2. Kukin kameratyyppi käsitellään sille sopivalla kalibrointiprofiililla
3. Tiedostoselaimessa näkyy kameramalli Kameramalli-sarakkeessa
4. Jokaiselle kameralle luodaan oma tulostuskansiohierarkia käsittelyn yhteydessä

**Esimerkkitilanteita**: Survey3W RGN + Survey3N OCN -kaksoiskamerajärjestelmä, tai LATTICE-matriisi, jossa on RGB-pääkamera ja useita kapeakaistamoduuleja***

## Suositukset

### Järjestä tiedostot ennen tuontia

* Säilytä kalibrointikohdekuvat samassa kansiossa kuin kartoituskuvat
* Säilytä kunkin kuvauskerran `.daq` / `.csv`-valosensoritiedostot kyseisen kuvauskerran kuvamateriaalin kanssa
* Säilytä kameran/SD-kortin/keskittimen alkuperäinen kansiorakenne
* Älä sekoita eri kuvaussessioiden aineistoja yhteen projektiin

### Tiedostojen nimeäminen

* Säilytä alkuperäiset kameran tiedostonimet (IMG\_0001.RAW, capture\_... jne.)
* Älä nimeä tiedostoja uudelleen ennen tuontia
* Alkuperäiset nimet sisältävät tärkeitä metatietoja

### Kalibrointikohdekuvat

* Liitä mukaan aina 1–2 kalibrointikohdekuvaa per istunto (Survey3; LATTICE-ohjelmistossa DAQ-tallenne voi korvata ne — katso [Kohdekuvien valinta](choosing-target-images.md))
* Ota kohdekuvia ennen ja jälkeen kuvaussession
* Sijoita kohteet samoihin valaistusolosuhteisiin kuin kuvausalue
* Merkitse kohdekuvat käyttämällä Target-valintaruutua

***

## Yleisiä ongelmia ja ratkaisuja

### Kuvia ei näy tuonnin jälkeen

**Mahdollisia syitä:**

* Tiedostomuotoa ei tueta (katso tuettujen tiedostotyyppien luettelo tämän sivun yläosasta)
* Kuvat ovat peräisin muista kuin MAPIR-kameroista (katso [Tuetut kamerat](../supported-cameras.md))
* Tiedosto on vioittunut tai siirto SD-kortilta on jäänyt kesken

**Ratkaisu**: Varmista tiedostomuodon ja kameramallin yhteensopivuus ja tarkista tiedostojen latausvaroituksesta, mitkä tiedostot tarkalleen ottaen eivät latautuneet

### Kameramallia ei tunnistettu

**Mahdolliset syyt:**

* Muokatut EXIF-metatiedot
* Kuvia on muokattu ulkoisessa ohjelmistossa
* Puutteellinen tiedostonsiirto

**Ratkaisu**: Tuo alkuperäiset, muokkaamattomat tiedostot uudelleen kamerasta tai SD-kortilta

### Puuttuvat aikaleimat

**Mahdolliset syyt:**

* Kameran kelloa ei ole asetettu oikein
* Ulkoinen ohjelmisto on poistanut EXIF-tiedot

**Ratkaisu**: Varmista, että kameran aika-asetukset olivat oikein kuvauksen aikana

### Uudelleen avattu projekti ilmoittaa puuttuvista tiedostoista

Jos lähdetiedostoja on siirretty tai poistettu sen jälkeen, kun projekti avattiin viimeksi, Chloros kertoo **mitkä** tiedostot puuttuvat sen sijaan, että avaisi tyhjän ruudukon. Palauta tiedostot alkuperäisille poluilleen tai poista puuttuvat merkinnät ja tuo tiedostot uudelleen.***

## Seuraavat vaiheet

Kun tiedostot on tuotu:

1. **Tarkista tiedostoluettelo** – Varmista, että kaikki kuvat on ladattu oikein
2. **Tarkista kameramallit** – Varmista, että kamerat on tunnistettu oikein
3. **Merkitse kohdekuvat** – Katso [Kohdekuvien valinta](choosing-target-images.md)
4. **Säädä asetuksia** – Määritä käsittelyasetukset kohdassa [Projektin asetukset](adjusting-project-settings.md)
5. **Käynnistä käsittely** – Katso [Käsittelyn käynnistäminen](starting-the-processing.md)

Yksityiskohtaisia tietoja projektin konfiguroinnista löytyy kohdasta [Projektin asetusten säätäminen](adjusting-project-settings.md).
