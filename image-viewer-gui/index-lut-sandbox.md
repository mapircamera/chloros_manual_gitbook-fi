# Indeksi/LUT-hiekkalaatikko

Index/LUT-hiekkalaatikko on Chloros-kuvankatseluohjelman sivupalkissa oleva interaktiivinen työtila. Valitset kaavan, liität kamerasi kanavat siihen, värität sen liukuvärillä ja säädät arvoaluetta — ja kuva päivittyy reaaliaikaisesti samalla kun teet muutoksia. Versiosta 1.2.0 lähtien voit myös **tallentaa luomuksesi** joko yksittäisen kuvan tai koko projektin osalta ilman uudelleenkäsittelyä.

## Mihin Sandboxia käytetään

| Index/LUT Sandbox (interaktiivinen)        | Projektin käsittely (eräajona)       |
| -------------------------------------- | -------------------------------- |
| Yksi kuva kerrallaan, välitön palaute  | Koko aineisto yhdellä kertaa     |
| Kokeellinen ja iteratiivinen             | Valmiiksi määritetyt asetukset          |
| Renderöi reaaliaikaisesti; tallentaa vain pyydettäessä  | Kirjoittaa aina lopulliset tiedostot      |
| Täydellinen oikeiden asetusten löytämiseen | Paras, kun asetukset ovat lopulliset |

{% hint style="success" %}
**Tavallinen työnkulku**: säädä asetuksia Sandboxissa, kunnes visualisointi vastaa toiveitasi, ja vie sitten joko suoraan Sandboxista tai kopioi samat indeksi- ja LUT-asetukset [Projektin asetuksiin](../project-settings/project-settings.md), jotta seuraava käsittelykierros sisällyttää ne jokaiseen kuvaan.
{% endhint %}

***

## Sandboxin avaaminen

1. Napsauta ruudukossa olevaa kuvaa — se avautuu koko ruudun kokoisena **Kuvankatselijan** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">-välilehdessä
2. Napsauta **Kuvankatselijan** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">-kuvaketta avataksesi vasemman sivupalkin, jos se ei ole jo auki
3. Valitse monikaistakerros oikean yläkulman kerrosvalikosta — **RAW (Reflectance)** on tavallisin valinta, koska kalibroidun heijastavuuden perusteella lasketut indeksiarvot ovat vertailukelpoisia eri kuvien välillä

Sivupalkissa näkyy ylhäältä alas:

* kuvan nimi ja sen kameramalli
* **Vie/Tallenna kuva(t)**-painike — tulee näkyviin, kun**Indeksi**tai**LUT** on valittu
* **Indeksi**- ja**LUT**-valintaruudut
* indeksin asetuspaneeli
* **Kursoriarvot**-paneeli, jossa on lukema, histogrammi ja GSD-säädin

{% hint style="warning" %}
**Ei käytettävissä mustavalkokameroissa.** Yksikaistaisessa LATTICE M3M -kuvassa molemmat valintaruudut ovat pois käytöstä, ja niiden työkaluvihjeessä lukee _”Ei käytettävissä monokromaattisissa (M3M) antureissa”_ — monikaistainen indeksi on määrittelemätön yhdellä kaistalla. Laskeaksesi indeksejä M3M-kameroista, yhdistä kaksi tai useampia kameraa kohdistetuksi monikaistaiseksi kuvapinoksi ja käytä LATTICE-indeksimoottoria.
{% endhint %}

***

## Indeksin soveltaminen

1. Valitse **Indeksi**-valintaruutu sivupalkin yläosassa
2. Valitse kamerasi suodatin vasemmasta pudotusvalikosta (`RGN`, `OCN`, `NGB`, `RGB`, `RE`, `NIR`)
3. Valitse indeksikaava oikeanpuoleisesta pudotusvalikosta — 27 sisäänrakennettua kaavaa sekä mahdolliset tallentamasi mukautetut kaavat
4. Kaava näkyy alla olevana matemaattisena lausekkeena, jossa jokaisessa kaistapaikassa on tyhjä ympyrä. **Vedä värillinen kanavaympyrä paikalleen** sitoaksesi sen
5. Kun kaikki kaavan käyttämät paikat on liitetty, kuva päivittyy ja näyttää indeksiarvot
6. Vie kohdistin kuvan päälle lukeaksesi arvot; **Kohdistimen arvot** -paneeli lisää indeksirivin, jossa näkyy kohdistimen alla oleva arvo

Kaksoisnapsauta sidottua paikkaa tyhjentääksesi sen. Keskeneräinen kaava on normaali tilanne vedon aikana, ei virhe — kuva ei yksinkertaisesti päivitty, ennen kuin kaava on valmis.

Kanavaympyrät on värikoodattu: punainen = Red, vihreä = Green, sininen = Blue, oranssi = Orange, syaani = Cyan, violetti = NIR, magenta = RE. Samoja värejä käytetään kanavapisteissä ja histogrammikäyrissä Kursorin arvot -paneelissa.

### Esimerkki NDVI

```

Formula: (NIR - Red) / (NIR + Red)

For a Survey3W RGN camera:
  NIR = 850 nm band
  Red = 661 nm band

Result range:          -1.0 to +1.0
Typical vegetation:     0.4 to 0.9
Stressed vegetation:    0.2 to 0.4
Bare soil:              0.0 to 0.2
Water:                 -0.1 to 0.1
```

Täydellinen kaavojen viite — kaikki kolme esiasetettua luetteloa ja tiedot siitä, mitkä nimet toimivat missäkin — on kohdassa [Monispektriset indeksikaavat](../project-settings/multispectral-index-formulas.md).

### Kun Indeksi on valittuna, mutta LUT:ta ei ole

Kuva piirretään **harmaasävyisenä**, venytettynä kahden kynnysarvon välille. Tämä on tarkoituksellista: indeksikuvassa on skalaaritietoja, ja harmaasävyinen kuva on sen rehellinen esitys. Lisää LUT, kun haluat väriä.***

## LUT-taulukoiden (Look-Up Tables) käyttö**Hakutaulukko** yhdistää indeksiarvot väreihin: syötetään NDVI 0,65, tuloksena tietty vihreä väri. Se ei muuta dataa — se muuttaa tapaa, jolla dataa tulkitaan.

### LUT:n lisääminen

1. Napsauta kaavan alla olevaa **&quot;+ Lisää LUT&quot;**-painiketta (<img src="../.gitbook/assets/image (1) (1) (1).png" alt="" data-size="line">)
2. Valitse värigradientti
3. Aseta leikkauksen minimi- ja maksimiarvot
4. Valitse leikkausmoodi
5. Valitse sivupalkista **LUT**-valintaruutu, jotta se renderöidään

LUT-valintaruutu pysyy pois käytöstä, kunnes LUT on todella määritetty indeksiin.

### Värigradientin valitseminen

Vie hiiri **gradienttipalkin**päälle avataksesi esiasetusluettelon — Chloros sisältää**seitsemän** gradienttiesiasetusta:

| # | Gradientti                            | Muoto                                                               |
| - | ----------------------------------- | ------------------------------------------------------------------- |
| 1 | Red → Keltainen → Green (**oletus**)  | Hajaantuva — vastaa tavanomaista kasvillisuutta koskevaa käsitystä, vihreä = terve |
| 2 | Violetti → Keltainen → Green             | Hajautuva, selvästi matalalla alareunalla                                  |
| 3 | Ruskea → Valkoinen → Blue                | Poikkeava vaalean keskipisteen ympärillä                                   |
| 4 | Musta → Violetti → Vaaleanpunainen → Vaaleankeltainen | Peräkkäinen, tummasta vaaleaan                                           |
| 5 | Red → Keltainen → Blue                 | Hajaantuvat vaalean keskipisteen ympärillä                                   |
| 6 | Violetti → Blue → Green → Keltainen      | Peräkkäin, tummasta vaaleaan                                           |
| 7 | Orange → Valkoinen → Violetti             | Hajaantuva vaalean keskipisteen ympärillä                                   |

**Hajautuva**väriliukuma sijoittaa neutraalin värin ikkunan keskelle, mikä toimii hyvin, kun keskipisteellä on merkitystä (kynnysarvo, viitepäivämäärä).**Peräkkäinen** väriliukuma kulkee yksitoikkoisesti tummasta vaaleaan, mikä toimii hyvin suureille, joilla on vain &quot;enemmän&quot; ja &quot;vähemmän&quot;.

Jokaisessa esiasetuksessa on seitsemän väripistettä. Napsauta esiasetusta, niin kuva päivittyy välittömästi (kun LUT-ruutu on valittuna).

### Väripisteiden muokkaaminen

Gradienttipalkin alla on rivi värinäytteitä, yksi kutakin väripistettä kohti:

* **Värin muuttaminen**: napsauta värinäytettä avataksesi värivalitsimen (värirengas, RGB/HSV-liukusäätimet tai heksakoodi, kuten `#FF0000`)
* **Lisää väri**: napsauta rivin lopussa olevaa**+**-painiketta — valkoinen väri lisätään
* **Poista väri**:**kaksoisnapsauta** värinäytettä
* **Tallenna muokattu väriliukuma**: napsauta väriliukumapalkin vieressä olevaa tallennuskuvaketta lisätäksesi muokatun väriliukumasi esiasetusluetteloon, jotta voit valita sen uudelleen

Indeksiin määrittämäsi väriliukuma tallennetaan kyseisen indeksin mukana projektin asetuksiin, joten se säilyy, vaikka projekti suljetaan ja avataan uudelleen.

**Vähemmän väriasteita**tuottaa selkeitä vyöhykkeitä, jotka luokitellaan;**enemmän väriasteita** tuottaa pehmeitä, lähes valokuvamaisia siirtymiä. Kolme–viisi väriastetta sopivat esitysdioihin ja luokittelukarttoihin; kuusi–kymmenen sopivat yleiseen analyysiin; viisitoista tai enemmän sopivat yksityiskohtaiseen tarkasteluun ja julkaisukuvioihin.

### Arvoalueen asettaminen

Kynnysarvon säätö on **kaksikahvainen liukusäädin**, jonka arvoalue on −1 – +1. Sen kummassakin päässä on muokattava tekstikenttä tarkkojen arvojen syöttämistä varten sekä**AUTO**-painike.

* Vedä kumpaakin kahvaa tai kirjoita luku ruutuun ja paina Enter
* **AUTO**asettaa alueen kuvan kelvollisten indeksiarvojen**

2. ja 98. persentiiliin** — tämä on hyvä lähtökohta, joka jättää poikkeavat arvot huomiotta. Chloros pyöristää tuloksen joustavasti: 4 desimaalin tarkkuudella hyvin kapealla alueella, 3 desimaalin tarkkuudella kapealla alueella ja muutoin 2 desimaalin tarkkuudella
* Manuaaliset säätöjä on etusija AUTO-asetukseen nähden, kunnes painat AUTO-painiketta uudelleen

Esimerkki NDVI-ikkunoista:

| Tavoite                                    | Min  | Max |
| --------------------------------------- | ---- | --- |
| Näytä kaikki                         | −1,0 | 1,0 |
| Vain kasvillisuus, sulje pois maaperä ja vesi | 0,2  | 0,9 |
| Vain terve kasvillisuus                 | 0,5  | 0,9 |
| Korosta stressiä                        | 0,2  | 0,5 |

Ikkunan kaventaminen lisää kontrastia kiinnostuksen kohteena olevalla alueella ja siirtää kaiken muun alueen ulkopuolelle — missä **leikkausmoodi** päättää, mitä sille tapahtuu.***

## Leikkausmoodit

Kun pikselin indeksiarvo jää minimi-/maksimi-ikkunan ulkopuolelle, leikkausmoodi päättää, miten se piirretään.

| Pudotusvalikon nimi                  | Tallennettu arvo      | Alueen ulkopuolella olevat pikselit piirretään seuraavasti                                                                                                |
| ------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Minimi ja maksimi** (oletus) | `clip`            | Gradientin lähin pääteväri — minimiarvon alapuolella olevat arvot saavat ensimmäisen värin, maksimiarvon yläpuolella olevat arvot saavat viimeisen |
| **Läpinäkyvä tausta**      | `transparent`     | Täysin läpinäkyvä (todellinen alfa)                                                                                                  |
| **Indeksitausta**| `indexColor`      | Harmaasävy, joka ulottuu kuvan**koko** indeksialueelle, joten alueen ulkopuolella oleva rakenne näkyy edelleen harmaana                |
| **Alkuperäinen tausta**         | `backgroundColor` | Itse taustakuvan kuva, joten värikerros sijaitsee todellisen kuvan päällä                                                |

| Tila                       | Sopii parhaiten                               | Ulkonäkö                                      |
| -------------------------- | -------------------------------------- | ----------------------------------------- |
| **Minimi ja maksimi**      | Täydellinen tietojen esitys, tieteellinen analyysi | Jokainen pikseli on värillinen                      |
| **Läpinäkyvä tausta** | GIS-päällekkäisyydet, arvoalueen erottelu   | Väri ikkunan sisällä, ei mitään sen ulkopuolella |
| **Indeksitausta**       | Korostus säilyttäen tietojen kontekstin    | Väri sisällä, harmaa ulkopuolella               |
| **Alkuperäinen tausta**    | Raportit ja esitykset              | Väri sisällä, valokuva ulkopuolella         |

{% hint style="info" %}
**Pikselit, joista ei ole dataa, ovat aina läpinäkyviä kaikissa tiloissa.** Pikseli, jonka indeksi ei ole äärellinen (0/0-jakolasku) tai on täsmälleen −1,0 tai +1,0 (kylläisyysraja-arvot, kun toisessa kaistassa lukema on nolla ja toisessa ei), käsitellään datattomana pikselinä eikä ääriarvona. Tämä pitää ylivalotetut kohokohdat ja tummat varjot pois väriskaalasta sen sijaan, että ne maalattaisiin kehyksen äärimmäisimmiksi lukemiksi. Sama sääntö määrittää, mitkä pikselit syötetään AUTO-kynnyksiin ja indeksihistogrammiin, jotta kaikki kolme ovat yhdenmukaisia.
{% endhint %}

Läpinäkyvyys säilyy, kun vienti tallennetaan muodossa PNG. Sitä ei voida esittää JPG-muodossa.

***

## Arvojen lukeminen säätöjen aikana

Asetuspaneelin alapuolella oleva **Kursoriarvot**-paneeli toimii Sandboxin mittausvälineenä:

* Siirrä kursori kuvan päälle ja lue kanavakohtaiset lähdearvot sekä indeksiarvo omalta riviltään
* Ota käyttöön histogrammin yläpuolella oleva **INDEX**-painike nähdäksesi indeksiarvojen jakauman kehyksessä; kaksi leikkauskynnystäsi näkyvät oransseina katkoviivoina ja kursorin arvo valkoisena viivana — tämä on nopein tapa valita ikkuna, joka todella sisältää haluamasi tiedot
* Ota **CURSOR** käyttöön nähdäksesi merkintäviivat osoittimen alla olevissa arvoissa
* Zoomaa yli 60× (vähemmän, jos GSD-lohkon koko on asetettu) korostaaksesi yksittäiset näytetyt pikselit, joilla on kelluva arvo

Käytännön ohjeet:

1. Merkitse muistiin arvot terveen kasvillisuuden, stressaantuneen kasvillisuuden, paljaan maan ja veden kohdalta
2. Tarkista, missä nämä klusterit sijaitsevat indeksi-histogrammissa
3. Aseta minimi- ja maksimiarvot niin, että ne rajaavat sinua kiinnostavan klusterin
4. Valitse leikkausmoodi — _Original Background_ pitää ympäröivän näkymän näkyvissä

***

## Vienti Sandboxista

Kaikki yllä oleva on reaaliaikaista esikatselua, kunnes tallennat sen. Sivupalkin yläosassa oleva **Vie/Tallenna kuva(t)** -painike avaa paneelin, joka liukuu sivupalkin päälle (eikä peitä kuvaa, joten näet edelleen, mitä olet päättämässä).

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>### Asetukset

| Asetus                          | Vaikutus                                                                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Käytä nykyiseen kuvaan**      | Tallentaa näytetyn kuvan täsmälleen näillä asetuksilla                                                                                                |
| **Käytä kaikkiin projektin kuviin** | Suorittaa saman konfiguraation uudelleen jokaiselle projektin kuvalle. Kuvat, joista puuttuvat tämän indeksin tarvitsemat kaistat, ohitetaan, eikä niitä käsitellä virheinä |
| **Indeksi-/LUT-gradienttipalkki**      | Tallentaa lisäksi jokaisesta viennistä erillisen selitekuvan, jossa arvoalue on merkitty                                                                     |
| **Indeksi-histogrammi**             | Tallentaa lisäksi jokaisesta viennistä erillisen histogrammikuvan, joka näyttää datan minimi- ja maksimiarvot sekä leikkauskynnykset                                               |

Jos kuvavälilehden **GSD-lohkon koko** on yli 1, paneeli ilmoittaa siitä ennen vahvistamista: vienti tallentaa sen, mitä näet, lohkojen keskiarvoistaminen mukaan lukien. Aseta GSD-säädin ensin takaisin arvoon 1, jos haluat täyden resoluution.

### Tiedostojen tallennuspaikka

Jokainen **Vie**-painikkeen napsautus luo**uuden, kertakäyttöisen kansion**:

```
<project folder>/Sandbox_Exports/<IndexName>_<Index|LUT>_<NNN>/
```

Esimerkkejä: `Sandbox_Exports/NDVI_LUT_001/`, sitten `Sandbox_Exports/NDVI_LUT_002/` seuraavalle ajolle. Numerointi perustuu levyllä jo olevien tiedostojen skannaamiseen, joten se säilyy uudelleenkäynnistyksissä ja vaikka poistaisit kansioita manuaalisesti. Mitään ei koskaan korvata — Sandboxin koko idea on verrata yhtä yritystä edelliseen.

Kansion sisällä, kuva kohden:

| Tiedosto                                                   | Sisältö                                                   |
| ------------------------------------------------------ | ---------------------------------------------------------- |
| `<source name>_<IndexName>_<Index\|LUT>.png`           | Renderöity kuva, pikseli pikseliltä sellainen kuin katseluohjelma sen näytti |
| `<source name>_<IndexName>_<Index\|LUT>_legend.png`    | Gradienttipalkin sivutiedosto, jos sitä on pyydetty                     |
| `<source name>_<IndexName>_<Index\|LUT>_histogram.png` | Indeksihistogrammin sivutiedosto, jos sitä on pyydetty                  |

Nämä kaksi sivukuvia tallennetaan aina **täydellä resoluutiolla**, vaikka pääkuva olisi lohkokohtaisesti keskiarvoistettu: lohkon koko vastaa näytön resoluutiota, ja molemmat sivukuvat sisältävät todelliset pikselikohtaiset indeksiarvot. Ne tulostavat myös enemmän tietoa kuin ruudulla näkyvät versiot — molemmat merkitsevät venytysikkunan _ja_ todelliset datan minimi- ja maksimiarvot, joten tallennettu selite on luettavissa vielä kuukausienkin kuluttua ilman, että projektia tarvitsee avata.

### Edistyminen ja tulokset

Koko projektin vienti kestää muutaman minuutin, joten ohjelma raportoi edistymisestä reaaliaikaisen edistymiskanavan kautta sen sijaan, että se jumiutuisi:

* Edistymispalkki näyttää tekstin ”`current / total`” ja tiedoston, jota parhaillaan tallennetaan
* Kun vienti on valmis, ruudussa ilmoitetaan, kuinka monta kuvaa vietiin, kuinka monta ohitettiin ja tulostuskansion polku
* Ohitetut kuvat luetellaan syyn kera (näytetään enintään viisi, minkä jälkeen tulee rivi ”+N lisää”). Yleisin syy on kerros, jossa ei ole tämän indeksin tarvitsemia kanavia
* Jos **yksikään** projektin kuva ei voi käyttää indeksiä, suoritus ilmoittaa epäonnistumisesta sen sijaan, että jättäisi sinulle tyhjän kansion

Vain yksi hiekkalaatikkovienti voi olla käynnissä kerrallaan. Toisen käynnistäminen kesken käynnissä olevan vientiä estetään selkeällä viestillä sen sijaan, että kaksi suoritusta kilpailisi samasta projektitiedostosta.

### Ruudukko poimii ajon

Jokainen valmis ajo näkyy omana painikkeenaan [kuvaruudukossa](image-grid.md) työkalurivillä, nimellä `<IndexName> <Index|LUT> <NNN>`. Näin voit verrata suorituksia: tee kaksi vientiä eri gradientteilla tai kynnysarvoilla ja vaihda sitten ruudukon kahden painikkeen välillä.

***

## Mukautetut indeksikaavat (Chloros+)

{% hint style="info" %}
**Missä ne luodaan**: Sandbox-sivupalkissa tai**Projektin asetuksissa** ennen käsittelyä. Molemmat tallentavat samaan projektitason luetteloon.
{% endhint %}

1. Avaa mukautetun kaavan laskin indeksikaavan pudotusvalikosta (vaatii kirjautumisen kelpoisella Chloros+-tilauksella)
2. Kirjoita kaava käyttämällä **kaistapaikkasymboleja** `x`, `y`, `z`, `a`, `b`, `c` — eivät kaistojen nimiä
3. Käytettävissä olevat operaattorit: `+`, `-`, `*`, `/`, `^` ja `()` ryhmittelyä varten
4. Käytettävissä olevat funktiot: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
5. Nimeä ja tallenna se — se näkyy kaavan pudotusvalikon alaosassa, ja voit määrittää sen paikat vetämällä kanavapiirejä, aivan kuten sisäänrakennetussa esiasetuksessa

```

Modified NDVI with an offset:   (y-x)/(y+x+0.5)
Simple ratio:                   y/x
Three-band difference:          (y-x)/(y+x-z)
Squared ratio:                  (y/x)^2
```

{% hint style="warning" %}
**Mukautetut kaavat ovat käytettävissä vain graafisessa käyttöliittymässä.** CLI/SDK `--indices` -vaihtoehto laajentaa 22 sisäänrakennettua esiasetuksen nimeä ja ohittaa hiljaisesti kaiken muun, mukaan lukien mukautetut kaavasi. Jos haluat käsitellä mukautetun kaavan eränä, määritä se Projektin asetuksissa ja suorita käsittely tai käytä Sandboxin ”Käytä kaikkiin projektin kuviin” -vientiä.
{% endhint %}

***

## Vianmääritys

### ”Tässä kerroksessa ei ole tämän indeksin tarvitsemia kanavia”

Kaava lukee kanavan sijaintia, jota nykyisessä kerroksessa ei ole — esimerkiksi kolmen paikan indeksiä yhden tai kahden kanavan tiedostossa. Vaihda monikaistaiseksi kerrokseksi (heijastavuus tai debayeroitu) tai valitse indeksi, joka sopii kamerasi suodattimeen.

### ”Kuvankäsittelytaustapalvelua ei voitu tavoittaa”

Taustapalvelu ei vastaa. Tarkista Lokit-välilehti; jos taustapalvelu käynnistyy uudelleen, Sandbox palautuu itsestään, kun se on jälleen käytettävissä.

### Kuva ei muuttunut, kun vedin ympyrää

Kaava ei ole vielä valmis. Keskeneräistä kaavaa käsitellään tavallisena vedon keskivaiheen tilana — mitään ei renderöidä eikä mitään raportoida virheeksi. Täytä kaikki kaavan käyttämät kentät.

### Koko kuva on yhtä väriä

Leikeikkunasi on todennäköisesti kaukana datan ulkopuolella. Paina **AUTO**-painiketta kiinnittääksesi sen 2. tai 98. persentiiliin, tai ota**INDEX**-histogrammi käyttöön nähdäksesi, missä data tosiasiassa sijaitsee.

### Viedyt värit eivät vastaa näkemääni

Niiden pitäisi vastata – vientipolku on tarkoituksellisesti live-esikatselun peilikuva, mukaan lukien leikkausmoodin alfa, ja lohkojen keskiarvoistaminen suoritetaan _värityksen jälkeen_ täsmälleen samalla tavalla kuin katseluohjelmassa. Jos ne eroavat toisistaan, tarkista, ettei GSD-lohkon koko ole muuttunut katselun ja viennin välillä.

***

## Seuraavat vaiheet

* [**Kuvakerrokset**](image-layers.md) — mille kerrokselle indeksi suoritetaan ja mitä sen arvot tarkoittavat
* [**Kuvan avaaminen koko näytön tilassa**](opening-an-image-full-screen.md) — kursorin lukema, histogrammi ja GSD-säätö yksityiskohtaisesti
* [**Monispektriset indeksikaavat**](../project-settings/multispectral-index-formulas.md) — kaikki esiasetukset, kaikilla pinnoilla
* [**Projektin asetukset**](../project-settings/project-settings.md) — löytämiesi asetusten tallentaminen käsittelykierrokseen
