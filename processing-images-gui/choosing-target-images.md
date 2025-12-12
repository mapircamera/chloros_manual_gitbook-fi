# Kohdekuvien valinta

Kalibrointikohteita sisältävien kuvien merkitseminen on tärkeä vaihe, joka nopeuttaa merkittävästi Chloros-käsittelyprosessia. Valitsemalla kohdekuvat etukäteen, Chloros:n ei tarvitse skannata kaikkia datajoukon kuvia kalibrointikohteiden löytämiseksi.

## Miksi kohdekuvat on merkittävä?

### Käsittelynopeus

Ilman kohdekuvien merkitsemistä Chloros:n on:

* Skannattava jokainen projektiisi kuuluva kuva
* Suoritettava kohteen tunnistusalgoritmit jokaiselle kuvalle
* Tarkistettava satoja tai tuhansia kuvia tarpeettomasti

**Tulos**: Käsittely voi kestää huomattavasti kauemmin, etenkin suurten tietojoukkojen kohdalla.

### Merkityt kohdekuvat

Kun valitset kohdekuvat kohdesarakkeesta:

* Chloros skannaa vain valitut kuvat kohteiden löytämiseksi
* Kohteiden tunnistus tapahtuu paljon nopeammin
* Kokonaiskäsittelyaika lyhenee huomattavasti

{% hint style=&quot;success&quot; %}
**Nopeuden parannus**: Merkitsemällä 2–3 kohdekuvaa 500 kuvan tietojoukosta voidaan kohteen tunnistusaika lyhentää yli 30 minuutista alle minuuttiin.
{% endhint %}

***

## Kohdekuvien merkitseminen

### Vaihe 1: Tunnista kohdekuvat

Tarkista tuodut kuvat tiedostoselaimessa ja tunnista, mitkä kuvat sisältävät kalibrointikohteita.

**Yleisiä tilanteita:**

* **Ennen kuvausta otettu kohde**: Otettu ennen istunnon aloittamista.
* **Kuvauksen jälkeen otettu kohde**: Otettu istunnon päätyttyä.
* **Kentällä olevat kohteet**: Kohteet, jotka on sijoitettu kuvausalueelle.
* **Useita kohteita**: 2–3 kohdekuvaa istuntoa kohti (suositeltava määrä).

### Vaihe 2: Tarkista kohdesarake

Jokaiselle kalibrointikohteen sisältävälle kuvalle:

1. Etsi kuva tiedostoselaimen taulukosta.
2. Etsi **Kohde**-sarake (oikeanpuoleisin sarake).
3. Napsauta kyseisen kuvan Kohde-sarakkeen valintaruutua.
4. Toista tämä kaikille kohteita sisältäville kuville.

### Vaihe 3: Vahvista valintasi

Tarkista ennen käsittelyä:

* [ ] Kaikki kalibrointikohteita sisältävät kuvat on valittu.
* [ ] Mitään muita kuin kohdekuvia ei ole valittu vahingossa.
* [ ] Kohteet ovat selvästi näkyvissä valituissa kuvissa.

***

## Kohdekuvien parhaat käytännöt

### Kohteiden kuvaamisen ohjeet

**Ajoitus:**

* Ota kohdekuvat välittömästi ennen kuvaussessiota ja sen aikana.
* Ota kuvat samoissa valaistusolosuhteissa kuin DAQ-valosensori.
* Ihannetapauksessa ota kohdekuvia mahdollisimman usein, jotta saat parhaat tulokset. Muussa tapauksessa valosensorin tietoja käytetään kalibroinnin säätämiseen ajan mittaan.

**Kameran sijainti:**

* Pidä kameraa kohteen yläpuolella siten, että se on keskellä ja täyttää noin 40–60 % kuvan keskikohdasta.
* Pidä kamera yhdensuuntaisena/nadir-asennossa kohteen pintaan nähden

**Valaistus:**

* Sama ympäristön valaistus kuin DAQ-valosensorissa
* Vältä varjoja kohteen pinnalla
* Älä peitä valonlähdettä kehollasi, ajoneuvolla tai kasvillisuudella
* Pilvinen sää tuottaa tasaisimmat tulokset

**Kohteen kunto:**

* Pidä kohdepaneelit puhtaina ja kuivina
* Kaikkien neljän paneelin tulee olla selvästi näkyvissä ja esteettömiä
* Kohteet kohtisuorassa/nadir-asennossa valonlähteeseen nähden, jos mahdollista

### Kuinka monta kohdekuvaa?

**Vähimmäismäärä:** 1 kohdekuva per istunto. **Suositeltava määrä:** 3–5 kohdekuvaa per istunto.

**Paras käytäntö:**

* Ota 3–5 kuvaa pian valosensorin tallennuksen aloittamisen jälkeen.
* Kierrä kameraa kuvien välillä parhaan tuloksen saamiseksi.
* Valinnainen: ota kuvia säännöllisesti istunnon aikana, jos valaistusolosuhteet muuttuvat jatkuvasti.

***

## Useiden kameroiden käyttö

### Kaksi kameraa

Jos käytät kahta MAPIR-kameraa samanaikaisesti (esim. Survey3W RGN + Survey3N OCN):

1. Ota kohdekuvat **molemmilla kameroilla** samanaikaisesti.
2. Käytä **samaa fyysistä kohdetta** molemmille kameroille.
3. Merkitse kohdekuvat **molemmille kameratyypeille** tiedostoselaimessa.
4. Chloros käyttää sopivia kohteita kunkin kameran kalibrointiin.

### Kameramallipylväs

**Kameramalli**-pylväs auttaa tunnistamaan, mitkä kuvat ovat peräisin mistäkin kamerasta:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* jne.

Käytä tätä saraketta tarkistaaksesi, että olet merkinnyt kohteet jokaiselle kameratyypille projektissasi.

***

## Kohteen tunnistuksen asetukset

### Tunnistuksen herkkyyden säätäminen

Jos Chloros ei tunnista kohteitasi oikein, säädä näitä asetuksia [Projektin asetuksissa](adjusting-project-settings.md):

**Vähimmäiskalibrointinäytealue:**

* **Oletus**: 25 pikseliä
* **Lisää**, jos pienet esineet tunnistetaan virheellisesti
* **Vähennä**, jos kohteita ei tunnisteta

**Vähimmäiskohteiden ryhmittely:**

* **Oletus**: 60
* **Lisää**, jos kohteet jaetaan useisiin tunnistuksiin
* **Vähennä**, jos värivaihteluita sisältäviä kohteita ei tunnisteta kokonaan

***

## Yleisiä kohdekuvaongelmia

### Ongelma: Kohteita ei tunnistettu

**Mahdolliset syyt:**

* Kohdekuvia ei ole merkitty tiedostoselaimessa
* Kohde on liian pieni kehyksessä (&lt; 30 % kuvasta)
* Huono valaistus (varjot, häikäisy)
* Kohteen havaitsemisen asetukset liian tiukat

**Ratkaisut:**

1. Varmista, että Kohde-sarakkeessa on valittu oikeat kuvat
2. Tarkista kohteen kuvanlaatu esikatselussa
3. Ota kohteet uudelleen, jos laatu on huono
4. Säädä kohteen havaitsemisen asetuksia tarvittaessa

### Ongelma: Vääriä kohteen havaitsemisia

**Mahdolliset syyt:**

* Valkoiset rakennukset, ajoneuvot tai maaperä peittävät kohteet
* Kirkkaat kohdat kasvillisuudessa
* Tunnistuksen herkkyys liian alhainen

**Ratkaisut:**

1. Merkitse vain todelliset kohdekuvat tunnistuksen rajoittamiseksi
2. Lisää kalibroinnin vähimmäisnäytealuetta
3. Lisää kohteen vähimmäisklusterointiarvoa
4. Varmista, että kohdekuvissa näkyy vain kohde (minimaalinen taustakohina)

***

## Tarkistuslista

Ennen käsittelyn aloittamista tarkista kohdekuvien valinta:

* [ ] Vähintään 1 kohdekuva merkitty per istunto
* [ ] Kohdesarakkeen valintaruudut on valittu kaikille kohdekuville
* [ ] Kohdekuvat on otettu samanaikaisesti kuin mittaus
* [ ] Kohteet näkyvät selvästi esikatselussa, kun niitä napsautetaan
* [ ] Kaikki 4 kalibrointipaneelia näkyvät jokaisessa kohdekuvassa
* [ ] Kohteissa ei ole varjoja tai esteitä
* [ ] Kaksikamerajärjestelmässä: Kohteet on merkitty molemmille kameratyypeille

***

## Kohteettoman käsittelyn

### Käsittely ilman kalibrointikohteita

Vaikka sitä ei suositella tieteelliseen työhön, voit käsitellä kuvia ilman kohteita:

1. Jätä kaikki kohdesarakkeen valintaruudut valitsematta
2. **Poista käytöstä** &quot;Heijastavuuskalibrointi&quot; projektin asetuksissa
3. Vignettikorjaus otetaan edelleen käyttöön.
4. Tulostetta ei kalibroida absoluuttisen heijastavuuden osalta.

{% hint style=&quot;warning&quot; %}
**Ei suositella**: Ilman heijastavuuden kalibrointia pikseliarvot edustavat vain suhteellista kirkkautta, eivät tieteellisiä heijastavuusmittauksia. Käytä kalibrointikohteita tarkkojen, toistettavien tulosten saamiseksi.
{% endhint %}

***

## Seuraavat vaiheet

Kun olet merkinnyt kohdekuvat:

1. **Tarkista asetukset** - Katso [Projektin asetusten säätäminen](adjusting-project-settings.md)
2. **Aloita käsittely** - Katso [Käsittelyn aloittaminen](starting-the-processing.md)
3. **Seuraa edistymistä** - Katso [Käsittelyn seuranta](monitoring-the-processing.md)

Lisätietoja kalibrointikohteista on kohdassa [Kalibrointikohteet](../calibration-targets.md).
