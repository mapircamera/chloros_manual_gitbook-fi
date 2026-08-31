# Kuvakerrokset

Kuvankatseluohjelman oikeassa yläkulmassa oleva **kerrosvalikko** mahdollistaa siirtymisen katsomasi kuvan kaikkien versioiden välillä – alkuperäisestä kuvasta jokaisen käsitellyn tuotteen kautta laskettuihin indeksikuvisiin – poistumatta katseluohjelmasta.

## Mitä ovat kuvakerrokset?

Chloros-ohjelmistossa ”kerros” on yksi **tuotetiedosto**, joka on rekisteröity yhtä lähdekuvaa kohden. Tuonti tuottaa lähdetiedostot; käsittely lisää kerroksen jokaiselle ajon tuottamalle tuotteelle. Viedyt tiedostot säilyttävät lähdetiedoston nimen — tuotteen tunnistaa**kansio**, ja kerroksen nimi on Chloros:n kyseiselle kansiolle antama nimike.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## Kerrosten luettelo

### Aina läsnä

| Kerros | Mikä se on |
| --- | --- |
| **JPG**(tai**PNG**/**TIFF**) | Tallennuksen mukana tullut perustiedosto. Survey3 tuo `.JPG`-tiedoston jokaisen `.RAW`-tiedoston viereen; LATTICE-kaappaukset tuovat mukanaan PNG- tai TIFF-näytön esikatselun. Nimetty sen mukaan, mitä tosiasiallisesti tuotiin |
| **RAW (Alkuperäinen)** | Lähteenä oleva raakakuva, josta on poistettu bayeri-koodi näyttöä varten ilman korjauksia. Käytettävissä heti tuonnin jälkeen — ei vaadi käsittelyä |

LATTICE-kaappauksella, jonka perustiedosto **on** sen raakakuva, ei ole erillistä perustietuetta: `RAW (Original)` kattaa sen jo.

### Survey3-käsittelytuotteet

| Kerros | Tallennettu | Olemassa, kun |
| --- | --- | --- |
| **RAW (Kohde)** | — | Kehys tunnistettiin sisältävän kalibrointikohteen |
| **RAW (heijastavuus)** | `Reflectance_Calibrated_Images/` | Heijastavuuskalibrointi suoritettiin onnistuneesti tällä kehyksellä |
| **Vignettikorjattu**| `Vignette_Corrected_Images/` | Kehystä ei voitu kalibroida heijastavuuden suhteen**ja** *vignettikorjaus* oli päällä |
| **Anturin vaste**| `Sensor_Response_Images/` | Kehystä ei voitu kalibroida heijastavuuden perusteella**ja** *vignetoinnin korjaus* oli pois päältä |
| **Valkotasapaino** | `White_Balanced_Images/` | Valkotasapainotettu tulos tallennettiin |

{% hint style="info" %}
**Vignettikorjaus ja anturin vaste ovat vaihtoehtoja, ei koskaan molempia.** Jokaista kameramallia kohti on täsmälleen yksi kalibroimaton varatuote kutakin ajokertaa kohti, ja *Vignettikorjaus*-valitsin valitsee, mikä niistä käytetään. Katso [Projektin asetukset](../project-settings/project-settings.md).
{% endhint %}

### LATTICE-tasot

LATTICE tallentaa fan-out-tiedot näihin tasoihin yhdellä käsittelykierroksella. Mitkä tasot ovat käytettävissä, riippuu Projektin asetuksissa määritetyistä tuotekohtaisista vientivalinnoista sekä siitä, mitä kameralle sovelletaan.

| Kerros | Tallennetaan | Sovelletaan |
| --- | --- | --- |
| **RAW (Debayered)** | `Debayered_Images/` | RGB ja monispektrinen |
| **RAW (esikatselu)** | `Preview_Images/` | Monispektriset (väärävärinen venytys) |
| **Valkotasapainotettu** | `Preview_Images/` | RGB-pääkamerat — RGB-esikatselu on rekisteröity tällä nimellä, jotta se on linjassa samannimisen Survey3-kerroksen kanssa |
| **RAW (säteily)** | `Radiance_Images/` | Vain monispektrinen |
| **RAW (heijastavuus)** | `Reflectance_Calibrated_Images/` | Vain monispektrinen, ja vain silloin, kun vastaava `.daq`-alasheijastustietue tai laadunvarmistuksen läpäissyt kehyksen sisäinen kohde peittää kehyksen |

RGB-pääkameroilla ei ole kaistakohtaista radiometriaa, joten niiden säteilyvoimakkuus- ja heijastavuusarvot ohitetaan merkinnällä **ei sovelleta** — loki ilmoittaa tästä sen sijaan, että virhe jäisi huomaamatta.

### Indeksi-, LUT- ja sandbox-kerrokset

| Kerroksen rakenne | Esimerkki | Mistä se on peräisin |
| --- | --- | --- |
| **RAW (`<INDEX>`-indeksi)** | `RAW (NDVI Index)` | Yksi kutakin projektiasetuksissa määritettyä indeksiä kohti, lasketaan käsittelyn aikana |
| **`<INDEX>` LUT** | `NDVI LUT` | Indeksin värikartoitettu versio |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Yksi kutakin [Indeksi/LUT-hiekkalaatikko](index-lut-sandbox.md) -vientikierrosta kohti |

Jos sama indeksin nimi on määritetty useammin kuin kerran eri asetuksilla, toiseen ja sitä seuraaviin lisätään numero nimeen (`RAW (NDVI2 Index)`), jotta kerrokset pysyvät erottuvina.

***

## Kerroksen valitsimen käyttö

1. Avaa kuva koko ruudun tilassa napsauttamalla ruudukon pikkukuvaa
2. Napsauta katseluohjelman oikeassa yläkulmassa olevaa **kerrosvalikkoa**

3. Valitse kerros — kuva päivittyy välittömästi

Pudotusvalikossa näkyvät ensin **JPG, RAW (Original), RAW (Target), RAW (Reflectance)** tässä järjestyksessä, ja kaikki muut luetellaan niiden jälkeen tuotteiden rekisteröintijärjestyksessä.

### Kerroksen valinta selaamisen aikana

Painamalla **←**/**→** siirryt seuraavaan kuvaan, ja ohjelma yrittää pitää sinut samalla kerroksella:

1. **Tarkka vastaavuus ensin** — jos seuraavassa kuvassa on samanniminen kerros, se valitaan. Tämä pitää sinut kerroksella `RAW (NDVI Index)`, kun selaat koko sarjaa läpi
2. **Sitten tyypin mukainen vastaavuus** — indeksikerros etsii mitä tahansa indeksikerrosta, LUT mitä tahansa LUT:ta, heijastavuuskerros heijastavuuskerrosta, kohdekerros kohdekerroksia, alkuperäiskerros alkuperäiskerroksia, pohjakerros pohjakerroksia
3. **Sitten, vain vientikerroksille** — nimi säilytetään, vaikka kerroksiluettelo ei olisi vielä päivittynyt, koska tiedosto on jo levyllä. Tämän ansiosta voit tarkastella tuotteita jo silloin, kun ajo on vielä kirjoittamassa niitä
4. **Muussa tapauksessa** — ensimmäinen käytettävissä oleva kerros, joka on yleensä peruskuva

Projektin `.daq`- ja `.csv`-sidecar-tiedostot ohitetaan nuolinäppäimillä navigoitaessa, joten kuvia selattaessa ei koskaan päädy valosensorin tallenteeseen.

Zoomaaminen ja panorointi siirtyvät myös kuvien välillä, mikä helpottaa saman kenttäkohdan ennen/jälkeen-vertailua.

***

## Pikseliarvojen ymmärtäminen kerrosten mukaan

[Kursorin arvot -paneeli](opening-an-image-full-screen.md#cursor-values) näyttää kursorin alla olevan kanavakohtaisen arvon siinä yksikössä, jossa kyseinen kerros on tallennettu. Sen sarakkeet vaihtuvat kerroksen mukaan:

| Kerros | Näytetty yksikkö | Huomautukset |
| --- | --- | --- |
| Base (JPG / PNG / TIFF-esikatselu) | DN, 0–255 | Näyttöarvot, gamma-korjattu RGB:ssa. Vain silmämääräiseen tarkasteluun |
| RAW (Alkuperäinen) | DN | Anturin raakadigitaalinumerot. Histogrammin akseli ilmaisee syvyyden: 255 (8-bittinen), 4095 (12-bittinen) tai 65535 (16-bittinen) |
| RAW (Debayered) | DN | Lineaarinen, ilman näytön venytystä |
| RAW (Esikatselu) / Valkotasapainotettu | DN | Näytettävä tulos — venytetty tai gamma-korjattu. Ei mittauskäyttöön |
| RAW (Säteilyvoimakkuus) | **W/m²/sr/nm** | Float32-muotoinen fyysinen säteilyvoimakkuus. Ei DN-saraketta |
| RAW (heijastavuus) | DN **ja %** | Prosenttiosuus laskettu kyseisen tiedoston omalla asteikolla — katso alla |
| Indeksi / LUT / sandbox-vienti | Indeksiarvo tai RGB-komponentit | Yksikanavainen indeksitiedosto ilmoittaa indeksiarvon; värikartoitettu LUT-tiedosto ilmoittaa komponentit Red/Green/Blue |

### Heijastavuus: asteikko on tiedostokohtainen

{% hint style="warning" %}
**”Jaa 65 535:llä” on oikea vain Survey3:n tapauksessa.** LATTICE-heijastavuus on tallennettu eri mittakaavassa, ja näiden kahden jakajan sekoittaminen on yleisin tapa saada heijastavuusarvot, jotka ovat täsmälleen puolet siitä, mitä niiden pitäisi olla.
{% endhint %}

| Lähde | DN, joka vastaa heijastavuutta 1,0 | Tunnistetaan |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | XMP-tunniste `Chloros:PixelScale=32768`, joka on merkitty jokaiseen LATTICE-heijastavuusvientiin. 2×:n liikkumavara tarkoittaa, että ρ-arvo yli 1,0 voidaan esittää eikä sitä leikata |
| **Survey3**|**65535** | Ei Chloros-XMP-skaalausmerkintää — Survey3-kalibrointi kirjoittaa ρ × dtype-max ja leikkaa arvon 1,0:ssa |

GIS- ja skriptikäyttöön: lue tiedostosta `Chloros:PixelScale` ja jaa sillä. Jos tunnistetta ei ole, tiedoston mittakaava on Survey3 (65535). Katseluohjelma, indeksi-/LUT-hiekkalaatikko ja indeksin vienti määrittävät mittakaavan kaikki tällä samalla tavalla, joten kursorin kohdalla näkyvä luku on se luku, jota indeksin laskennassa on käytetty.

Tämän skaalan lisäksi formaattikohtainen tallennus:

* **TIFF (32-bittinen, prosentteina)** tallentaa DN / 65535 liukulukuna
* **PNG (8-bittinen)**ja**JPG (8-bittinen)** tallentavat DN × 255 / 65535
* **8-bittisen lähteen tallennuksen 8-bittinen TIFF-vienti** rajataan arvoihin 0–255 sen sijaan, että sitä skaalattaisiin uudelleen, eikä siinä ole tarkoituksella skaalausmerkintää. Paneeli tulostaa näiden tiedostojen osalta vain DN-arvon ilman prosenttisaraketta

### Indeksiarvojen alueet

| Indeksiperhe | Tyypillinen alue | Lukema |
| --- | --- | --- |
| Normalisoitu ero (NDVI, GNDVI, NDRE, ENDVI…) | −1 – +1 | Terve kasvillisuus yleensä 0,4–0,9; paljas maaperä lähellä 0:aa; vesi negatiivinen |
| Maaperän mukaan korjattu (SAVI, OSAVI, MSAVI2…) | noin −1 – +1,5 | Samanlainen lukema kuin NDVI, mutta maaperän taustavaikutus on vaimennettu |
| Suhdeluku (GRVI, GCI, MSR, CIRE…) | ylärajaa ei ole | Suhdeluvut kasvavat rajattomasti, kun nimittäjäkaista lähestyy nollaa |
| EVI / LAI | 0 – ~1, 0 – ~3,5 | Pilvet ja muut kyllästyneet pikselit työntävät molemmat arvojen ulkopuolelle — peitä ne ensin |

Katso [monispektristen indeksien kaavat](../project-settings/multispectral-index-formulas.md) saadaksesi jokaisen esiasetuksen taustalla olevan tarkan kaavan.

***

## Yleisiä työnkulkuja

### Ennen / jälkeen -vertailu

1. Valitse **RAW (Alkuperäinen)** ja huomioi vinjetointi sekä kalibroimattomat arvot
2. Vaihda **RAW (Heijastavuus)** -tilaan
3. Vertaa — vinjetointi poistettu, arvot kalibroitu. Zoomaa ja panoroi niin, että katsot samaa aluetta

### Tarkista yksi indeksi koko sarjasta

1. Avaa ensimmäinen käsitelty kuva ja valitse indeksikerros
2. Paina **→**-näppäintä toistuvasti — indeksikerros seuraa sinua kuvasta toiseen
3. Tarkkaile sivupalkin histogrammia samalla: kehys, jonka jakauma hyppää, on syytä tarkastella lähemmin

### Varmista kalibrointikohteet

1. Valitse **RAW (Target)** kohdekuvasta
2. Varmista, että kohde on selvästi näkyvissä ja tunnistettu
3. Siirry seuraavaan kohdekuvaan — kohdekerros seuraa mukana

### Tarkista heijastusarvojen tarkkuus

1. Valitse **RAW (Reflectance)**

2. Lue**%**-sarake Cursor Values -paneelista — se on jo skaalattu oikein kyseiselle tiedostolle
3. Tarkista arvot vertaamalla niitä kehyksessä oleviin tunnettuihin materiaaleihin: terve kasvillisuus on korkea NIR-arvossa ja matala punaisessa; kalibrointikohteen arvon tulisi olla lähellä sen julkaistua heijastusarvoa

***

## Vianmääritys

### Odotettua kerrosta ei näy pudotusvalikossa

**Mahdolliset syyt**

* Kuvaa ei ole koskaan käsitelty — vain pohjakerros ja `RAW (Original)` ovat olemassa
* Tuotteen vientivalintaa ei ole valittu Projektiasetuksissa
* Tuotetta ei voida soveltaa kyseiseen kameraan (säteily ja heijastavuus RGB-pääkamerassa; mikä tahansa indeksi yksikaistaisessa M3M-monokamerassa)
* Heijastavuuskalibroinnilla ei ollut mitään, mihin perustua — ei `.daq`-alaspäin suuntautuvaa peittoaluetta eikä laadunvarmistuksen läpäissyttä kohdetta kuvassa — joten kuvan käsittely palasi Vignette Corrected- tai Sensor Response -tilaan

**Toimenpiteet**

1. Tarkista ajon loki: Chloros ilmoittaa, milloin pyydettyä vientituotetta ei voitu tuottaa ja miksi
2. Tarkista tuotekohtaiset vientivalinnat kohdassa [Project Settings](../project-settings/project-settings.md)
3. Varmista, että tuotekansio on olemassa projektin tulostuspuussa
4. Suorita käsittely uudelleen, kun tuote on käytössä

### Kerrosluettelo näyttää vanhentuneelta

Chloros skannaa projektin tuotekansiot uudelleen ajon ollessa käynnissä ja korjaa puuttuvat kerrosten rekisteröinnit levyllä tosiasiallisesti olevien tietojen perusteella, joten normaalisti viety kerros näkyy itsestään kyselyssä. Siirtyminen pois kuvasta ja takaisin pakottaa uuden ratkaisun.

### Heijastusarvot näyttävät olevan puolet siitä, mitä niiden pitäisi olla

Olet lähes varmasti jakanut LATTICE-tiedoston luvulla 65535. Käytä `Chloros:PixelScale` (32768) tai tarkista **%**-saraketta, jossa jakaja on jo sovellettu.

### Indeksikerros on olemassa, mutta kuva on tyhjä

Indeksi vaatii kaistoja, joita kerroksessasi ei ole — esimerkiksi indeksi, joka lukee kolmatta kanavaa, on sovellettu yksi- tai kaksikanavaiseen tiedostoon. Vaihda monikaistakerrokseen (heijastavuus tai debayeroitu) tai valitse indeksi, joka sopii kameran suodattimeen.

***

## Seuraavat vaiheet

* [**Kuvan avaaminen koko näytön tilassa**](opening-an-image-full-screen.md) — kohdistimen lukema, histogrammi ja GSD-säätö
* [**Indeksi/LUT-kokeilualusta**](index-lut-sandbox.md) — interaktiivinen indeksin visualisointi ja vienti
* [**Monispektriset indeksikaavat**](../project-settings/multispectral-index-formulas.md) — indeksin viite
* [**Käsittelyn viimeistely**](../processing-images-gui/finishing-the-processing.md) — tulostuskansiohierarkia, johon nämä tasot viittaavat
