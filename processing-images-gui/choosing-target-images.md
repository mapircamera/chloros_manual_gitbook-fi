# Kohdekuvien valinta

Niiden kuvien merkitseminen, jotka sisältävät kalibrointikohteita, on ratkaiseva vaihe, joka nopeuttaa merkittävästi Chloros-käsittelyprosessia. Valitsemalla kohdekuvat etukäteen vältät sen, että Chloros joutuu skannaamaan jokaisen kuvan datajoukossasi kalibrointikohteiden löytämiseksi.

## Miksi kohdekuvat kannattaa merkitä?

### Käsittelynopeus

Ilman kohdekuvien merkitsemistä Chloros:n on:

* Skannattava jokainen projektiisi kuuluva kuva
* Suoritettava kohteen tunnistusalgoritmeja jokaiselle kuvalle
* Tarkistettava satoja tai tuhansia kuvia tarpeettomasti

**Tulos**: Käsittely voi kestää huomattavasti kauemmin, etenkin suurissa datajoukoissa.

### Merkittyjen kohdekuvien kanssa

Kun valitset tiettyjen kuvien Kohde-sarakkeen:

* Chloros skannaa kohteita vain valituista kuvista
* Kohteen tunnistus valmistuu paljon nopeammin
* Kokonaiskäsittelyaika lyhenee huomattavasti

{% hint style="success" %}
**Nopeuden parannus**: 2–3 kohdekuvan merkitseminen 500 kuvan aineistossa voi lyhentää kohteen tunnistusaikaa yli 30 minuutista alle minuuttiin.
{% endhint %}

***

## Kohdekuvien merkitseminen

### Vaihe 1: Tunnista kohdekuvasi

Selaa tuotuja kuvia tiedostoselaimessa ja tunnista, mitkä kuvat sisältävät kalibrointikohteita.

**Yleisiä tilanteita:*** **Ennen kuvauksen kohde**: Kuvattu ennen istunnon aloittamista
* **Kuvauksen jälkeinen kohde**: Kuvattu istunnon päätyttyä
* **Kenttäkohteet**: Kohteet, jotka on sijoitettu kuvausalueelle
* **Useita kohteita**: 2–3 kohdekuvaa istuntoa kohti (suositeltava)

### Vaihe 2: Tarkista kohdesarake

Jokaiselle kalibrointikohteen sisältävälle kuvalle:

1. Etsi kuva tiedostoselaimen taulukosta
2. Etsi **Kohde**-sarake (oikeanpuoleisin sarake)
3. Napsauta kyseisen kuvan Kohde-sarakkeen valintaruutua
4. Toista tämä kaikille kohteita sisältäville kuville

### Vaihe 3: Varmista valintasi

Tarkista ennen käsittelyä:

* [ ] Kaikki kuvat, joissa on kalibrointikohteita, on valittu
* [ ] Mitään kuvia, joissa ei ole kohteita, ei ole valittu vahingossa
* [ ] Kohteet näkyvät selvästi valituissa kuvissa

***

## Kohdekuvien parhaat käytännöt

### Kohteen kaappaamisen ohjeet

**Ajoitus:**

* Kaappaa kohdekuvat välittömästi ennen kuvaussessiota ja sen aikana
* Samoissa valaistusolosuhteissa kuin DAQ-valosensori
* Parhaiden tulosten saavuttamiseksi ota kohdekuvia mieluiten mahdollisimman usein. Muussa tapauksessa valosensorin tietoja käytetään kalibroinnin säätämiseen ajan mittaan.

**Kameran sijainti:**

* Pidä kameraa kohteen yläpuolella siten, että se on keskitetty ja täyttää noin 40–60 % kuvan keskiosasta.
* Pidä kamera yhdensuuntaisena/nadirina kohteen pintaan nähden

**Valaistus:**

* Sama ympäristön valaistus kuin DAQ-valosensorillasi
* Vältä varjoja kohdepinnoilla
* Älä peitä valonlähdettä kehollasi, ajoneuvollasi tai kasvillisuudella
* Pilviset olosuhteet antavat tasaisimmat tulokset

**Kohteen kunto:**

* Pidä kohdepaneelit puhtaina ja kuivina
* Kaikkien neljän paneelin tulee olla selvästi näkyvissä ja esteettömiä
* Kohteet kohtisuorassa/nadir-asennossa valonlähteeseen nähden, jos mahdollista

### Kuinka monta kohdekuvaa?

**Vähimmäismäärä:**1 kohdekuva per istunto.**Suositus:** 3–5 kohdekuvaa per istunto.**Paras käytäntö:**

* 3–5 kuvaa otetaan pian sen jälkeen, kun valosensori on alkanut tallentaa
* Käännä kameraa kuvien välillä parhaan tuloksen saamiseksi
* Valinnainen: säännöllisesti istunnon aikana, jos valaistusolosuhteet muuttuvat jatkuvasti

***

## Useiden kameroiden käyttö

### Kahden kameran kokoonpanot

Jos käytät kahta MAPIR-kameraa samanaikaisesti (esim. Survey3W RGN + Survey3N OCN):

1. Ota kohdekuvia **molemmilla kameroilla** samanaikaisesti
2. Käytä **samaa fyysistä kohdetta** molemmille kameroille
3. Merkitse kohdekuvat **molemmille kameratyypeille** tiedostoselaimessa
4. Chloros käyttää sopivia kohteita kunkin kameran kalibrointiin

### Kameramalli-sarake

**Kameramalli**-sarake auttaa tunnistamaan, mitkä kuvat ovat peräisin mistäkin kamerasta:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* jne.

Käytä tätä saraketta tarkistaaksesi, että olet merkinnyt kohteet jokaiselle kameratyypille projektissasi.

***

## Kohteen tunnistuksen asetukset

### Tunnistuksen herkkyyden säätäminen

Jos Chloros ei tunnista kohteitasi oikein, säädä näitä asetuksia [Projektin asetuksissa](adjusting-project-settings.md):**Vähimmäiskalibrointinäytteen pinta-ala:*** **Oletus**: 25 pikseliä
* **Lisää**, jos saat vääriä tunnistuksia pienistä esineistä
* **Vähennä**, jos kohteita ei tunnisteta**Vähimmäiskohteiden ryhmittely:*** **Oletus**: 60
* **Lisää**, jos kohteet jakautuvat useiksi tunnistuksiksi
* **Pienennä**, jos värivaihteluita sisältäviä kohteita ei tunnisteta kokonaan***

## Yleisiä kohdekuvien ongelmia

### Ongelma: Kohteita ei tunnistettu

**Mahdolliset syyt:**

* Kohdekuvia ei ole merkitty tiedostoselaimessa
* Kohde on liian pieni kuvassa (&lt; 30 % kuvasta)
* Huono valaistus (varjot, häikäisy)
* Kohteen tunnistuksen asetukset ovat liian tiukat

**Ratkaisut:**

1. Varmista, että Kohde-sarakkeen valinta on valittuna oikeille kuville
2. Tarkista kohdekuvan laatu esikatselussa
3. Ota kohteet uudelleen, jos laatu on huono
4. Säädä kohteen tunnistuksen asetuksia tarvittaessa

### Ongelma: Vääriä kohteen tunnistuksia

**Mahdolliset syyt:**

* Valkoiset rakennukset, ajoneuvot tai maaperän peitteet, joita luullaan kohteiksi
* Kirkkaat alueet kasvillisuudessa
* Tunnistuksen herkkyys liian alhainen

**Ratkaisut:**

1. Merkitse vain todelliset kohdekuvat tunnistusalueen rajoittamiseksi
2. Suurenna kalibroinnin vähimmäisnäytteen pinta-alaa
3. Suurenna kohteiden ryhmittelyn vähimmäisarvoa
4. Varmista, että kohdekuvissa näkyy vain kohde (mahdollisimman vähän taustahäiriöitä)

***

## Tarkistuslista

Tarkista kohdekuvien valinta ennen käsittelyn aloittamista:

* [ ] Vähintään 1 kohdekuva merkitty per istunto
* [ ] Kohdesarakkeen valintaruudut on valittu kaikille kohdekuville
* [ ] Kohdekuvat on otettu samanaikaisesti tutkimuksen kanssa
* [ ] Kohteet näkyvät selvästi esikatselussa, kun niitä napsautetaan
* [ ] Kaikki 4 kalibrointipaneelia näkyvät jokaisessa kohdekuvassa
* [ ] Kohteissa ei ole varjoja tai esteitä
* [ ] Kaksikamerajärjestelmässä: Kohteet on merkitty molemmille kameratyypeille

***

## Kohteettomat käsittelyt

### Käsittely ilman kalibrointikohteita

Vaikka sitä ei suositella tieteelliseen työhön, voit käsitellä kuvia ilman kohteita:

1. Jätä kaikki kohde-sarakkeen valintaruudut valitsematta
2. **Poista käytöstä** &quot;Heijastavuuden kalibrointi&quot; Projektin asetuksissa
3. Vignettikorjaus sovelletaan edelleen
4. Tulosta ei kalibroida absoluuttisen heijastavuuden suhteen

{% hint style="warning" %}
**Ei suositella**: Ilman heijastavuuskalibrointia pikseliarvot edustavat vain suhteellista kirkkautta, eivät tieteellisiä heijastavuusmittauksia. Käytä kalibrointikohteita tarkkojen ja toistettavien tulosten saamiseksi.
{% endhint %}

***

## Seuraavat vaiheet

Kun olet merkinnyt kohdekuvasi:

1. **Tarkista asetuksesi** – Katso [Projektin asetusten säätäminen](adjusting-project-settings.md)
2. **Aloita käsittely** – Katso [Käsittelyn aloittaminen](starting-the-processing.md)
3. **Seuraa edistymistä** – Katso [Käsittelyn seuranta](monitoring-the-processing.md)

Lisätietoja kalibrointikohteista on kohdassa [Kalibrointikohteet](../calibration-targets.md).
