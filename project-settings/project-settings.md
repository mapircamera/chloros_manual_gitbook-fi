# Projektin asetukset

Chloros-ohjelmiston sivupalkissa ”Projektin asetukset” (<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

) voit määrittää projektisi kaikki kuvankäsittelyyn, kalibrointikohteiden tunnistukseen, monispektristen indeksien laskemiseen ja vientiasetuksiin liittyvät asetukset. Nämä asetukset tallennetaan projektisi mukana, ja ne voidaan tallentaa malleiksi, joita voi käyttää uudelleen useissa projekteissa.

## Projektiasetusten avaaminen

Voit avata projektiasetukset seuraavasti:

1. Avaa projekti Chloros-ohjelmassa
2. Napsauta vasemmassa sivupalkissa **Project Settings**-<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

-välilehteä
3. Asetuspaneelissa näkyvät kaikki käytettävissä olevat konfigurointivaihtoehdot luokittain järjestettyinä

<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->

{% hint style="info" %}
**Asetukset, jotka riippuvat muista asetuksista, näkyvät harmaana.** Kun yläpuolinen valinta estää jonkin asetuksen käytön (esimerkiksi *Heijastavuuskalibrointi / valkotasapaino*-valinnan poistaminen estää *Vie heijastavuus*-toiminnon käytön), riippuva säätö on poissa käytöstä ja sen työkaluvihjeessa mainitaan valinta, jota on muutettava.
{% endhint %}

***

## Näyttö

### Kuvan pikkukuvan resoluutio

* **Tyyppi**: Pudotusvalikko
* **Vaihtoehdot**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Oletus**: Oletus (512 px)
* **Kuvaus**: Resoluutio (pisin sivu, pikseleinä), jolla kuvaruudukon pikkukuvat renderöidään. Suuremmat arvot näyttävät terävämmiltä zoomattaessa, mutta latautuvat hitaammin ja vievät enemmän muistia. Täysi resoluutio vastaa alkuperäisen kuvan kokoa.
* **Huomautus**: Vain näyttöä varten — tämä ei vaikuta milloinkaan käsittelyyn tai vietyihin tiedostoihin.***

## Kohteen tunnistus

Nämä asetukset määrittävät, miten Chloros tunnistaa ja käsittelee kalibrointikohteita kuvissasi. Molemmat ovat aktiivisia vain, kun **Heijastuskalibrointi / valkotasapaino** on käytössä (muutoin ne ovat harmaana, koska kohteen tunnistus ohitetaan kokonaan).

### Kalibrointikohteen vähimmäispinta-ala (px)

* **Tyyppi**: Luku
* **Alue**: 0–10 000 pikseliä
* **Oletusarvo**: 25 pikseliä
* **Kuvaus**: Määrittää pienimmän alueen (pikseleinä), joka tarvitaan, jotta tunnistettu alue voidaan katsoa kelvolliseksi kalibrointikohteen näytteeksi. Pienemmät arvot tunnistavat pienempiä kohteita, mutta voivat lisätä vääriä positiivisia tuloksia. Suuremmat arvot vaativat tunnistamista varten suurempia ja selkeämpiä kohdealueita.
* **Milloin säätää**:
  * Suurenna arvoa, jos saat vääriä tunnistuksia pienistä kuvan artefakteista
  * Pienennä arvoa, jos kalibrointikohteesi näyttävät pieniltä kuvissasi eivätkä ne tule tunnistetuiksi

### Kohteiden ryhmittelyn vähimmäisarvo (0–100)

* **Tyyppi**: Luku
* **Alue**: 0–100
* **Oletusarvo**: 60
* **Kuvaus**: Säätää ryhmittelykynnystä, jolla samanväriset alueet ryhmitellään kalibrointikohteita tunnistettaessa. Suuremmat arvot edellyttävät, että ryhmiin kootaan enemmän samanlaisia värejä, mikä johtaa konservatiivisempaan kohteiden tunnistukseen. Pienemmät arvot sallivat enemmän värivaihtelua kohderyhmän sisällä.
* **Milloin säätää**:
  * Lisää arvoa, jos kalibrointikohteet jakautuvat useiksi tunnistuksiksi
  * Vähennä arvoa, jos värivaihtelua sisältäviä kalibrointikohteita ei tunnisteta kokonaan

***

## Käsittely

Nämä asetukset säätelevät, miten Chloros käsittelee ja kalibroi kuvasi.

### Vignetoinnin korjaus

* **Tyyppi**: Valintaruutu
* **Oletus**: Käytössä (valittuna)
* **Kuvaus**: Soveltaa vignettikorjausta kompensoimaan objektiivin aiheuttamaa tummenemista kuvien reunoilla. Vignettaus on yleinen optinen ilmiö, jossa kuvan kulmat ja reunat näyttävät keskustaa tummemmilta objektiivin ominaisuuksien vuoksi.
* **Sivuvaikutus**: Tämä valinta määrittää myös, minkä *kalibroimattoman varatuotteen* ajo tallentaa (katso alla).

### Heijastavuuden kalibrointi / valkotasapaino

* **Tyyppi**: Valintaruutu
* **Oletus**: Käytössä (valittuna)
* **Kuvaus**: Ottaa heijastavuuskalibroinnin käyttöön — joko kuvan sisällä havaituista kalibrointikohteista ja/tai DAQ-valosensorin alaspäin suuntautuvista mittaustiedoista, riippuen kamerasta ja käytettävissä olevista tiedoista. Tämä normalisoi heijastavuusarvot koko aineistossa ja varmistaa yhdenmukaiset mittaustulokset valaistusolosuhteista riippumatta.
* **Kun pois käytöstä**: Kohteiden tunnistus ohitetaan kokonaan, eikä**mikään kamera voi tuottaa heijastavuustuotetta** — ei Survey3-kohdeohjattu eikä LATTICE DAQ-ohjattu. Riippuvat asetukset (*Vie heijastavuus*, *Vähimmäisrekalibrointiväli* ja kohteen tunnistuksen kynnysarvot) ovat harmaana.

### Kalibroimattomat varatuotteet: Vie anturin vaste / Vie vignettikorjattu

* **Tyyppi**: Kaksi valintaruutua
* **Oletusasetukset**: Molemmat käytössä (valittuna)
* **Kuvaus**: Kun kuvan heijastavuutta ei voida kalibroida (kalibrointikohdetta ei löytynyt tai heijastavuuskalibrointi on pois päältä), se tallennetaan sen sijaan *kalibroimattomana varatuotteena*. **Jokaista kameramallia kohti on täsmälleen yksi näistä kahdesta varatuotteesta kutakin ajokertaa kohti**, ja se valitaan *Vignettikorjaus*-kytkimellä:
  * Vignettikorjaus **päällä**→ `Vignette_Corrected_Images/` (säädetään asetuksella**Vienti vignettikorjattuna**)
  * Vignettikorjaus **pois**→ `Sensor_Response_Images/` (määritetään asetuksella**Vie anturin vaste**)
* Vara-tuote, jota ei ole käytössä, on harmaana. Käytössä olevan valinnan poistaminen estää kyseisen tiedoston tallentamisen kokonaan.

### LATTICE-vientituotteet

LATTICE-kuvauksia sisältävissä projekteissa jokainen tuotu LATTICE-kuva jaetaan kaikkiin käytössä oleviin **ja soveltuviin**tuotteisiin yhdellä käsittelykierroksella. Neljä valintaruutua ohjaavat jakamista (kaikki oletuksena**päällä**):

| Asetus | Tulostuskansio | Mitä se vie |
| --- | --- | --- |
| **Vie debayeroituna** | `Debayered_Images/` | Lineaarinen debayeroitu kuva. Koskee RGB- ja monispektrikameroita. |
| **Vie esikatselu** | `Preview_Images/` | Näytön esikatselu. RGB = valkotasapaino (DAQ-valonlähde, jos saatavilla, muuten harmaamaailma) + gamma; monispektrinen = väärävärinen venytys. |
| **Vienti: säteilyvoimakkuus** | `Radiance_Images/` | Float32-tyyppinen spektrinen säteilyvoimakkuus yksiköissä W/m²/sr/nm. Vain monispektriset (M3C/M3M) — ei koske RGB-mestareita. Kirjoitetaan aina 32-bittisenä TIFF-muodossa riippumatta *Kalibroitu kuvamuoto* -asetuksesta. |
| **Vientiheijastavuus**| `Reflectance_Calibrated_Images/` | Uint16-heijastavuus, skaalattu siten, että**32768 = heijastavuus 1,0** (merkitty XMP-muodossa `Chloros:PixelScale`). Vain monispektrikuvissa, tallennetaan, kun vastaava `.daq`-alas suuntautuva tietue (tai laadunvarmistuksen läpäissyt kehyksen sisäinen kohde) peittää kehyksen. |

* RGB-pääkamerat lähettävät debayeroitua + esikatselukuvaa; niiden säteilyvoimakkuutta/heijastavuutta ei oteta huomioon, koska ne eivät ole sovellettavissa.
* Debayeroidun/esikatselukuvan bittisyvyys noudattaa *Kalibroitu kuvamuoto* -asetusta; säteilyvoimakkuus on aina float32.
* Nämä neljä kytkintä eivät vaikuta Survey3-käsittelyyn.

Samat neljä kytkintä ovat olemassa ilman nimeä tunnuksina `chloros-cli process --debayered / --preview / --radiance / --reflectance` ja SDK:n vastaavina parametreina. Ne korvasivat vanhan `--radiometric-output`-lipun, jota ei enää ole olemassa.

{% hint style="warning" %}
**Kaikkien soveltuvien tuotteiden poistaminen käytöstä aiheuttaa ajon epäonnistumisen.** Versiosta 1.2.0 lähtien käsittelyajo, jolta pyydettiin tuotteita mutta joka ei kirjoittanut yhtään kuvatuotetta, ilmoittaa epäonnistumisesta, ja CLI lopetetaan arvollanollasta, sen sijaan että se ilmoittaisi hiljaisen onnistumisen. Lokissa mainitaan tuote, jota ei voitu kirjoittaa, ja syy siihen. Tarkoituksellisesti vain metatietoja tuottava ajo (mitään ei pyydetty) on edelleen onnistunut.
{% endhint %}

### Heijastavuuslähde (projektiasetus, määritetään komentojen CLI/SDK avulla)

Projekti tallentaa myös sen, mitä **heijastavuusviitettä** LATTICE-heijastavuustuote käyttää. Asetuspaneelissa ei ole tätä varten erillistä säätöä; arvo tallennetaan projektin konfiguraatioon nimellä `Processing → "Target reflectance source"` ja se asetetaan `chloros-cli process --reflectance-source {auto,target,daq}`:n tai SDK:n `reflectance_source`-parametrin avulla:

* **`auto`** (oletus): laadunvarmistuksen läpäissyt kehyksen sisäinen kalibrointikohde toimii absoluuttisena vertailuarvona; jos kohdetta ei ole tai laadunvarmistus epäonnistuu, käytetään DAQ:n alaspäin suuntautuvan säteilyn jakosuhdetta (ρ = πL/E).
* **`target`**: tiukka kohteeseen perustuva heijastavuus — ei DAQ-korvausta.
* **`daq`**: DAQ-pohjainen heijastavuus; kehyksen sisällä olevia kohteita ei käytetä vertailukohtana.

Tallennettua arvoa verrataan kirjainkokoa huomioimatta, ja muutamia kirjoitusmuotoja hyväksytään aliaksina: `target`, `target_image`, `empirical` ja `empirical_line` tarkoittavat kaikki **target**; `daq`, `dls`, `light_sensor` ja `sensor` tarkoittavat kaikki**daq**:ta. Kaikki muut – mukaan lukien puuttuva avain – tulkitaan**auto**:ksi.

Yksikkökohtaiset **mitatut** kohteen skannaukset haetaan kohteen yksikön sarjanumeron/QR-koodin perusteella, kuten `<serial>.csv`, kolmesta paikasta: hakemistosta, joka on annettu `--target-reflectance-dir`:llä (tallennettu nimellä `Processing → "Target reflectance dir"`), projektin omasta `target_reflectance/`-kansiosta sekä `CHLOROS_TARGET_REFLECTANCE_DIR`-ympäristömuuttujan polusta. Jos kyseiselle yksikölle ei ole olemassa mitattua skannausta, käytetään sen sijaan kohdemallin julkaistua nimelliskäyrää.

### Debayer-menetelmä

* **Tyyppi**: Pudotusvalikko
* **Vaihtoehdot**:
  * Vakio (nopea, keskilaatuinen)
  * Tekstuuritietoinen (hidas, korkein laatu) \[Chloros+]
* **Oletus**: Standard (Nopea, keskilaatuinen)
* **Kuvaus**: Valitsee demosaicing-algoritmin, jota käytetään raakojen Bayer-kuvioisten anturitietojen muuntamiseen täysvärikuviksi. ”Standard (Nopea, keskilaatuinen)” -menetelmä tarjoaa optimaalisen tasapainon käsittelynopeuden ja kuvanlaadun välillä. ”Tekstuuritietoinen (hidas, korkein laatu)” \[Chloros+] käyttää korkealaatuista, reunoja huomioivaa demosaicing-menetelmää yhdistettynä tekoäly-/koneoppimismalliin, joka poistaa lähes kaiken demosaicing-kohinan. Texture Aware -mallin suorittaminen vaatii GPU-muistia (VRAM). Suosittelemme sen käyttöä, kun käytettävissä on yli 4 Gt VRAM-muistia, jotta käsittely on nopeampaa.
* **Ainoastaan silloin, kun rivi on pudotusvalikko**: kahden vaihtoehdon pudotusvalikko näkyy vain, kun**molemmat**ehdot täyttyvät — olet kirjautunut sisään kelpoisella Chloros+-tilauksella,**ja** projekti ei sisällä LATTICE-kaappauksia. Muussa tapauksessa rivi näkyy pelkkänä tekstinä `Standard (Fast, Medium Quality)` ilman valintamahdollisuutta.
* **LATTICE-huomautus**: LATTICE-koulutettua Texture Aware -mallia ei ole, ja käsittelyputki pakottaa LATTICE-kehyksille tavallisen demosaicin tallennetusta arvosta riippumatta. Jos lisäät LATTICE-kansion projektiin, jossa Texture Aware oli jo valittuna, Chloros palauttaa asetuksen takaisin Standard-tilaan sen sijaan, että jättäisi vanhentuneen arvon kohtaan `project.json`.

### Kalibroinnin vähimmäisväli

* **Tyyppi**: Luku
* **Alue**: 0–3 600 sekuntia
* **Oletusarvo**: 0 sekuntia
* **Kuvaus**: Määrittää kalibrointikohteiden käytön välisen vähimmäisajan (sekunteina). Kun arvoksi asetetaan 0, Chloros käyttää jokaista havaittua kalibrointikohdetta. Kun arvoksi asetetaan suurempi luku, Chloros käyttää vain kalibrointikohteita, joiden välinen aika on vähintään tämän sekuntimäärän suuruinen, mikä lyhentää käsittelyaikaa datasarjoissa, joissa kalibrointikohteita havaitaan tiheästi.
* **Milloin säätää**:
  * Aseta arvoksi 0, jotta kalibrointitarkkuus on mahdollisimman suuri vaihtelevissa valaistusolosuhteissa
  * Suurenna arvoa (esim. 60–300 sekuntiin), jotta käsittely on nopeampaa, kun valaistus on tasainen ja kalibrointikohteiden kuvia on tiheästi

### Valosensorin aikavyöhykepoikkeama

* **Tyyppi**: Luku
* **Alue**: -12 – +12 tuntia
* **Oletus**: 0 tuntia
* **Kuvaus**: Määrittää valosensorin tietojen aikaleimojen aikavyöhykkeen siirtymän (tunteina UTC:sta), jota käytetään valosensorilokien ja kuvien ottamisajankohtien yhteensovittamisessa. Uudemmat `.daq`-tallenteet sisältävät oman aikavyöhykkeen alkuperätiedon, joten tätä tarvitaan pääasiassa vanhemmille, paikallisaikana tallennetuille lokeille.

### Käytä PPK-korjauksia

* **Tyyppi**: Valintaruutu
* **Oletus**: Pois käytöstä (valintaruutu tyhjä)
* **Kuvaus**: Mahdollistaa GPS:ää (GNSS) sisältävien MAPIR-DAQ-tallentimien Post-Processed Kinematic (PPK) -korjausten käytön. Kun tämä on käytössä, Chloros käyttää projektikansiossasi olevia .daq-lokitiedostoja, jotka sisältävät valotuspin-tietoja, ja soveltaa tarkkoja maantieteellisiä korjauksia kuviisi.
* **Vaatimus**: Projektikansiossasi on oltava .daq-lokitiedosto, jossa on valotuspin-merkintöjä
* **Milloin otetaan käyttöön**: On suositeltavaa ottaa PPK-korjaus aina käyttöön, jos .daq-lokitiedostossasi on valotustietueita.

### Valotustieto 1

* **Tyyppi**: Pudotusvalikko
* **Näkyvyys**: Näkyy vain, kun ”Käytä PPK-korjauksia” on otettu käyttöön JA valotustietoja on saatavilla pin 1:lle
* **Vaihtoehdot**:
  * Projektista tunnistetut kameramallien nimet
  * ”Älä käytä” – Ohita tämä valotuspinn
* **Oletus**: Valitaan automaattisesti projektin asetusten perusteella
* **Kuvaus**: Määrittää tietyn kameran valotuspinnille 1 PPK-aikasynkronointia varten. Valotuspinnit tallentavat tarkan ajankohdan, jolloin kameran suljin laukeaa, mikä on kriittistä tarkan PPK-paikannuksen kannalta.
* **Automaattisen valinnan toiminta**:
  * Yksi kamera + yksi liitin: Valitsee kameran automaattisesti
  * Yksi kamera + kaksi liitintä: Liitin 1 määritetään automaattisesti kameralle
  * Useita kameroita: Manuaalinen valinta vaaditaan

### Valotusliitin 2

* **Tyyppi**: Pudotusvalikko
* **Näkyvyys**: Näkyy vain, kun ”Käytä PPK-korjauksia” on käytössä JA valotustietoja on saatavilla nastalle 2
* **Vaihtoehdot**:
  * Projektissa tunnistetut kameramallien nimet
  * ”Älä käytä” – Ohita tämä valotustappi
* **Oletus**: Valitaan automaattisesti projektin kokoonpanon perusteella
* **Kuvaus**: Määrittää tietyn kameran valotusnastaan 2:lle PPK-ajan synkronointia varten, kun käytetään kahden kameran kokoonpanoa.
* **Automaattisen valinnan toiminta**:
  * Yksi kamera + yksi nasta: Nasta 2 asetetaan automaattisesti tilaan ”Älä käytä”
  * Yksi kamera + kaksi pinniä: Pinni 2 asetetaan automaattisesti tilaan ”Älä käytä”
  * Useita kameroita: Manuaalinen valinta vaaditaan
* **Huomautus**: Samaa kameraa ei voi määrittää samanaikaisesti sekä pinnille 1 että pinnille 2.***

## DAQ-valosensori

Tämä osio näkyy Projektin asetuksissa ja luettelee kaikki projektin DAQ-alasvalotiedostot — `.daq`-tallenteet ja DAQ-M `.csv`-alasvalolokit. Valosensorit-välilehdessä tehdyt tallenteet lisätään avoimeen projektiin automaattisesti.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->

Jokaisella rivillä näkyy tiedosto, anturimalli ja kyseiselle tiedostolle tosiasiallisesti voimassa oleva hajotin-korkin korjaus. Rivien yläpuolella on yksi koko projektia koskeva säätö:

### Korkin ohitus (kaikki tiedostot)

* **Tyyppi**: Pudotusvalikko
* **Vaihtoehdot**: `Auto` sekä projektissa oleville anturityypeille voimassa olevat suojuksen korjausprofiilit
* **Oletus**: Auto
* **Tallennetaan nimellä**: `Processing → "DAQ cap id"` (oletus `auto`)
* **Kuvaus**: `Auto` käyttää kunkin tiedoston tallennettua cap-arvoa (jos mitään ei ole tallennettu, oletuksena käytetään Sunshine-cap-arvoa — kaikki MAPIR-DAQ-laitteet toimitetaan Sunshine-korjaimella). Tietyn cap-arvon valinta ohittaa**kaikki** projektin alaspäin suuntautuvan tiedoston: raakatallenteet korjataan sen avulla, ja tallenteet, joissa on jo korjausarvo, viitataan uudelleen (tallennettu korjaus peruutetaan ja valittu korjausarvo otetaan käyttöön).
* **Tärkeää**: Valitun suojuksen on vastattava tallennuksen aikana fyysisesti asennettua suojusta. Anturi tai ohjelmisto eivät tunnista fyysistä suojusta — väärän suojuksen tunnisteen käyttö korjaa spektrit virheellisesti.

Tarkoituksella on käytössä **yksi** koko projektia koskeva säätö tiedostokohtaisten pudotusvalikoiden sijaan: asetus koskee kaikkia projektin alaspäin suuntautuvia lähteitä.***

## Array-kohdistus

Tämä osio näkyy **vain**, kun vähintään yhdessä projektin kuvassa on moduulien välinen kohdistusmuunnos, jonka LATTICE-array-järjestelmät merkitsevät kuvaushetkellä (XMP-tunnisteet `Chloros:Alignment*`). Siinä näkyy, kuinka monessa kuvassa on kohdistustunnisteita, mikä kamera on vertailukamerana (`REF`-tunniste) sekä kamerakohtainen taulukko kuvien lukumääristä.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Sovella matriisin kohdistusta

* **Tyyppi**: Valintaruutu
* **Oletus**: Käytössä (valittuna)
* **Tallennetaan nimellä**: `Processing → "Array alignment"`
* **Kuvaus**: Muuntaa jokaisen käsitellyn tuotteen (debayered / esikatselu / säteily / heijastavuus / indeksi) matriisin yhteiseen vertailugeometriaan käyttäen kuvaushetkellä tallennettua muunnosta. Pois käytöstä = vienti alkuperäisessä anturikohtaisessa geometriassa.

### Rajaa yhteiseen päällekkäisyyteen

* **Tyyppi**: Valintaruutu (aktiivinen vain, kun *Käytä matriisin kohdistusta* on päällä)
* **Oletus**: Käytössä (valittuna)
* **Tallennetaan nimellä**: `Processing → "Array alignment crop"`
* **Kuvaus**: Rajaa kohdistetut vientitiedostot alueelle, joka on yhteinen kaikille kameramoduuleille, jolloin jokaisella kaistalla on sama peittoalue. Pois-asetuksella säilytetään koko anturin kuvapinta (lähteen ulkopuolella musta täyttö).

### Uudelleennäytteenotto

* **Tyyppi**: Pudotusvalikko (aktiivinen vain, kun *Käytä matriisin kohdistusta* on päällä)
* **Vaihtoehdot**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Oletus**: Bilineaarinen
* **Tallennetaan nimellä**: `Processing → "Array alignment interpolation"`
* **Kuvaus**: Kohdistuksen vääristymän interpolointimenetelmä. *Lähin* säilyttää tarkat lähdearvot (ei pikselien välistä sekoittumista) tarkkaa radiometrista analyysia varten; *Bilineaarinen* sopii parhaiten kartoitukseen ja visuaaliseen käyttöön.

Samat kolme vaihtoehtoa ovat käytettävissä ilman etuliitettä muodossa `chloros-cli process --array-alignment`, `--array-alignment-crop` ja `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Indeksi

Näiden asetusten avulla voit määrittää monispektriset indeksit analysointia ja visualisointia varten.

### Lisää indeksi

* **Tyyppi**: Erityinen indeksien määrittelypaneeli
* **Kuvaus**: Avaa interaktiivisen paneelin, jossa voit valita ja määrittää monispektriset kasvillisuusindeksit (NDVI, NDRE, EVI jne.), jotka lasketaan kuvankäsittelyn aikana. Voit lisätä useita indeksejä, joista jokaisella on omat visualisointiasetuksensa.
* **Käytettävissä olevat indeksit**: Käyttöliittymän pudotusvalikossa on**27** ennalta määriteltyä monispektristä indeksikaavaa (katso [Monispektriset indeksikaavat](multispectral-index-formulas.md) täydellisen luettelon, mukaan lukien nimet, jotka hyväksytään myös CLI/SDK- ja `--indices`-vaihtoehdoissa).
* **Ominaisuudet**:
  * Valitse valmiista indeksikaavoista
  * Vedä kamerasi suodatinkanavat kaavan kaistapaikoille
  * Määritä visualisoinnin värigradientit (LUT – hakutaulukot)
  * Aseta kynnysarvot ja leikkausmoodit
  * Luo mukautettuja indeksikaavoja
* **Huomautus**: Indeksejä ei lasketa yksikaistaisille LATTICE M3M -monokameroille — monikaistaiset indeksit ovat määrittelemättömiä yhdellä kaistalla. Tämä ei koske malleja Survey3 ja LATTICE M3C.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->

Jokainen lisäämäsi indeksi renderöi kaavansa matemaattisena, ja jokaiselle kaistapaikalle tulee värillinen ympyrä: punainen = Red, vihreä = Green, sininen = Blue, oranssi = Orange, syaani = Cyan, violetti = NIR, magenta = RE. Vedä ympyrä kaavan yläpuolella olevasta rivistä paikkaan sitoaksesi sen; kaksoisnapsauta sidottua paikkaa poistaaksesi sen. Indeksi lasketaan vain kerran, kun jokaisessa kaavan käyttämässä paikassa on kanava.

### Mukautetut kaavat (Chloros+-ominaisuus)

* **Tyyppi**: Mukautettujen kaavojen määritelmien taulukko
* **Saatavuus**: Vaatii kirjautumisen kelpoisella Chloros+-tilauksella.
* **Kuvaus**: Mahdollistaa mukautettujen monispektristen indeksikaavojen luomisen ja tallentamisen kaistamatematiikkaa käyttäen. Mukautetut kaavat tallennetaan projektin asetusten mukana, ja niitä voidaan käyttää aivan kuten sisäänrakennettuja indeksejä.
* **Luominen**:
  1. Avaa indeksin määrityspaneelissa mukautettujen kaavojen laskin
  2. Kirjoita kaava käyttämällä **kaistapaikkasymboleja**, älä kaistojen nimiä
  3. Tallenna kaava kuvaavalla nimellä — se näkyy sitten kaavavalikon alareunassa, ja voit vetää kamerasi kanavaympyrät sen paikoille täsmälleen kuten sisäänrakennetussa esiasetuksessa
* **Kaavan syntaksi**:
  * Kaistapaikat: `x`, `y`, `z`, `a`, `b`, `c` — kuusi paikkaa, jotka määritetään todellisille kanaville vetämällä
  * Operaattorit: `+`, `-`, `*`, `/`, `^` ja `()` ryhmittelyä varten
  * Funktiot: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Miksi symboleja, ei bändinimiä**: kaava, joka on kirjoitettu muodossa `(y-x)/(y+x)`, toimii millä tahansa kameralla, koska vetämällä jamäärittää, onko `y` 850 nm:n NIR-suodattimen RGN vai 808 nm:n NIR-suodattimessa. Sisäänrakennetut esiasetukset tallennetaan samalla tavalla — katso [Multispektriset indeksikaavat](multispectral-index-formulas.md), josta löydät kaikkien 27 kaavan tarkan symbolimuodon.
* **Käyttöalue**: mukautetut kaavat tallennetaan projektin asetusten mukana, ja niitä voidaan käyttää sekä [indeksi-/LUT-hiekkalaatikossa](../image-viewer-gui/index-lut-sandbox.md) että kuvankäsittelyssä. Niitä**ei** hyväksytä CLI/SDK `--indices`-nimiluettelossa, joka laajentaa vain 22 sisäänrakennettua esiasetuksen nimeä.***

## Vienti

Nämä asetukset määrittävät vietyjen, käsiteltyjen kuvien muodon ja laadun.

### Kalibroitu kuvamuoto

* **Tyyppi**: Pudotusvalikko
* **Vaihtoehdot**:
  * **TIFF (16-bittinen)** – Pakkaamaton 16-bittinen TIFF-muoto
  * **TIFF (32-bittinen, prosentti)** – 32-bittinen liukulukuformaatti TIFF, jossa heijastusarvot on ilmaistu prosentteina
  * **PNG (8-bittinen)** – Pakattu 8-bittinen PNG-muoto
  * **JPG (8-bittinen)** - Pakattu 8-bittinen JPEG-muoto
* **Oletus**: TIFF (16-bittinen)
* **Kuvaus**: Valitsee tiedostomuodon käsiteltyjen ja kalibroitujen kuvien tallentamista varten. Viedyt tiedostot tallennetaan kunkin kameran kansion sisällä olevaan formaattikohtaiseen alikansioon (`tiff16`, `tiff32`, `png8`, `jpg8`), ja jokaiselle tuotteelle on oma `<Product>_Images/`-kansio. Viedyt tiedostot säilyttävät alkuperäisen tiedostonimen — tuotteen tunnistaa kansio, ei tiedostonimen pääte.
* **Tiedostomuotosuositukset**:
  * **TIFF (16-bittinen)**: Suositellaan tieteelliseen analyysiin ja ammattimaisiin työnkulkuihin. Säilyttää parhaan mahdollisen datan laadun ilman pakkausartefakteja. Sopii parhaiten monispektrianalyysiin ja jatkokäsittelyyn GIS-ohjelmistoissa.
  * **TIFF (32-bittinen, prosentteina)**: Sopii parhaiten työnkulkuihin, joissa heijastusarvot tarvitaan prosentteina (0–100 %). Tarjoaa parhaan tarkkuuden radiometrisissä mittauksissa.
  * **PNG (8-bittinen)**: Sopii hyvin verkkokatseluun ja yleiseen visualisointiin. Pienemmät tiedostokoot häviöttömällä pakkauksella, mutta pienempi dynaaminen alue.
  * **JPG (8-bittinen)**: Pienimmät tiedostokoot, sopii parhaiten vain esikatseluihin ja verkkonäyttöön. Käyttää häviöllistä pakkausta, joka ei sovellu tieteelliseen analyysiin.
* **Huomautus**: LATTICE-säteilyarvot viedään aina 32-bittisenä liukulukuna (TIFF) tästä asetuksesta riippumatta.***

## Tallenna projektimalli

Tämän ominaisuuden avulla voit tallentaa nykyiset projektiasetuksesi uudelleen käytettäväksi malliksi.

* **Tyyppi**: Tekstikenttä + Tallenna-painike
* **Kuvaus**: Anna asetusmallillesi kuvaava nimi ja napsauta tallennuskuvaketta. Malli tallentaa kaikki nykyiset projektiasetuksesi (kohteen tunnistus, käsittelyasetukset, indeksit ja vientimuoto), jotta niitä on helppo käyttää uudelleen tulevissa projekteissa. Mallit tallennetaan projektin tallennuskansion sisällä olevaan `Project Templates/`-kansioon, ja ne voidaan myös valita tai viedä päävalikosta (*Valitse malli* / *Tallenna malli* / *Vie malli*).
* **Käyttötapauksia**:
  * Luo malleja eri kamerajärjestelmille (RGB, monispektrinen, NIR)
  * Tallenna vakioasetukset tietyille viljelykasvilajeille tai analyysityönkulkuille
  * Jaa yhtenäiset asetukset koko tiimille
* **Käyttöohjeet**:
  1. Määritä kaikki haluamasi projektiasetukset
  2. Anna mallin nimi (esim. &quot;RedEdge Survey3 NDVI Standard&quot;)
  3. Napsauta tallennuskuvaketta
  4. Malli voidaan nyt ladata uusia projekteja luotaessa

***

## Projektikansio

Tämä asetus määrittää, mihin uudet projektit tallennetaan oletuksena.

* **Tyyppi**: Hakemistopolun näyttö + Muokkaa-painike
* **Oletus (Windows)**: `C:\Users\[Username]\Chloros Projects`
* **Oletus (Linux)**: `~/Chloros Projects`
* **Kuvaus**: Näyttää nykyisen oletuskansion, johon uudet Chloros-projektit luodaan. Napsauta muokkauskuvaketta valitaksesi toisen kansion. Muutos tallennetaan yhtenä tekstirivinä tiedostoon `~/.chloros/working_directory.txt` — Windows:ssa se on tiedosto `C:\Users\<Username>\.chloros\working_directory.txt`. Jos kyseistä tiedostoa ei ole, tai siinä on polku, jota ei enää ole olemassa, Chloros käyttää yllä mainittua oletusarvoa. CLI lukee ja kirjoittaa samaa tiedostoa, joten `chloros-cli` ja käyttöliittymä ovat aina yhtä mieltä siitä, missä projektit sijaitsevat.
* **Projektimallit** sijaitsevat tämän hakemiston `Project Templates/`-alikansiossa.
* **Milloin muuttaa**:
  * Aseta verkkoasemaksi tiimiyhteistyötä varten
  * Vaihda asemaan, jossa on enemmän tallennustilaa suurille tietojoukoille
  * Järjestä projektit vuosittain, asiakkaittain tai projektityypeittäin eri kansioihin
* **Huomautus**: Tämän asetuksen muuttaminen vaikuttaa vain UUSIIN projekteihin. Olemassa olevat projektit pysyvät alkuperäisissä sijainneissaan.***

## Asetusten säilyminen

Chloros-projekti on **kansio**. Kaikki projektin asetukset tallennetaan sen sisällä olevaan `project.json`-kansioon; liitetyt laitteet tallennetaan sen ohella kansioihin `cameras.json` ja `sensors.json`, joten projektin avaaminen uudelleen yhdistää myös sen kamerat ja valosensorit uudelleen. Kun avaat projektin uudelleen, kaikki asetukset palautuvat täsmälleen sellaisina kuin ne jätit. Tallennettuja projekteja voidaan ohjata myös ilman käyttöliittymää tiedoston `chloros-cli project` tai tiedoston SDK sisältämän tiedoston `open_project` avulla.

### Asetusten hierarkia

Asetukset otetaan käyttöön seuraavassa järjestyksessä:

1. **Järjestelmän oletusasetukset** – Chloros:n määrittämät sisäänrakennetut oletusasetukset
2. **Malliasetukset** – Jos lataat mallin projektia luodessasi
3. **Tallennetut projektiasetukset** – Projektitiedoston mukana tallennetut asetukset
4. **Manuaaliset muutokset** – Kaikki muutokset, jotka teet nykyisen istunnon aikana

### Asetukset ja kuvankäsittely

Käsittelyasetukset luetaan, kun käsittelykäynnistys alkaa. Asetuksen muuttaminen ei vaikuta takautuvasti levyllä jo oleviin tuotteisiin — suorita käsittely uudelleen, jotta uudet asetukset tulevat voimaan. Jotkin asetukset eivät vaikuta käsittelyyn lainkaan:

* Kuvan pikkukuvan resoluutio (vain näyttöä varten)
* Tallenna projektimalli
* Tallenna projektikansio

***

## Konfiguraatioavainten viite

Automaatiota varten (CLI `--config`, SDK `configure` tai lukemalla suoraan `project.json` suoraan) nämä ovat tarkat avaimet kohdassa `Project Settings`:

| Avaimen polku | Tyyppi | Oletus |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | luku 0-10000 | `25` |
| `Target Detection → Minimum Target Clustering (0-100)` | luku 0–100 | `60` |
| `Processing → Vignette correction` | bool | `true` |
| `Processing → Reflectance calibration / white balance` | bool | `true` |
| `Processing → Export sensor response` | bool | `true` |
| `Processing → Export vignette corrected` | bool | `true` |
| `Processing → Export debayered` | bool | `true` |
| `Processing → Export preview` | bool | `true` |
| `Processing → Export radiance` | bool | `true` |
| `Processing → Export reflectance` | bool | `true` |
| `Processing → Array alignment` | bool | `true` |
| `Processing → Array alignment crop` | bool | `true` |
| `Processing → Array alignment interpolation` | `"Bilinear" \| "Nearest" \| "Cubic"` | `"Bilinear"` |
| `Processing → Debayer method` | `"Standard (Fast, Medium Quality)" \| "Texture Aware (Slow, Highest Quality)"` | Vakio |
| `Processing → Minimum recalibration interval` | luku 0–3600 | `0` |
| `Processing → Light sensor timezone offset` | luku -12..12 | `0` |
| `Processing → Apply PPK corrections` | bool | `false` |
| `Processing → DAQ cap id` | cap-profiilin tunnus tai `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | luettelo indeksin määrityksistä | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

`Array alignment`-avaimet kirjoitetaan ensimmäisen kerran, kun Array Alignment -osio renderöidään tai kun automaatiokutsu asettaa ne. Jos niitä ei ole, prosessiketju käyttää samoja arvoja kuin yllä on esitetty (`true`, `true`, bilineaarinen), joten projekti, jossa niitä ei ole, käyttäytyy täsmälleen samalla tavalla kuin projekti, jossa ne ovat.

### `project.json`:ään tallennetut avaimet, joita ei voi hallita asetuspaneelissa

Nämä sijaitsevat samassa `Project Settings`-puussa ja käsittely lukee ne, mutta sivupalkista ei löydy niille omaa widgetiä:

| Avainpolku | Tyyppi | Oletus | Asettaja |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK `input_level=`. Ohittaa LATTICE-syöttötiedostojen TIFF-tiedostojen tulkinnan; `auto` päättelee kunkin tiedoston `Chloros:ProcessingLevel` XMP-tunnisteen ja kanavamäärän perusteella. Ohitetaan Survey3 `.raw`-kaappauksissa. Ei tarkoituksellisesti GUI-asetus — auto on oikea valinta kaikissa normaaleissa tapauksissa. |
| `Processing → Target reflectance dir` | polkumerkkijono | `""` | `chloros-cli process --target-reflectance-dir` tai projektin kohde API |
| `Processing → Target reflectance config` | kameran sarjanumeron mukaan indeksoitu sanakirja | `{}` | Kehyksessä olevan kohteen rekisteröinti (tila `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | polkumerkkijono | `""` | SDK `process_folder(daq_log_path=…)`. Viittaa `.daq`-tallenteeseen tai niiden sisältävään kansioon |
| `Target Detection → Minimum calibration target squares` | numero | `4` | Vanha oletusarvo; ei hallintaa eikä CLI-lippua |
| `UI → Grid thumbnail size` | numero | `160` | Kuvaruudukon oma pikkukuvien zoomausliukusäädin |

Kaksi katseluasetusta tallennetaan **ylimmälle-tasolla `project.json`**, kokonaan `Project Settings`:n ulkopuolella, koska ne koskevat näyttötilaa eivätkä käsittelyasetuksia:

| Avainpolku | Tyyppi | Oletus | Asettaja |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | kokonaisluku 1–256 | `1` | Kuvavälilehden GSD (px) -säädin — katso [Kuvan avaaminen koko näytön tilassa](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Parhaat käytännöt

1. **Aloita oletusasetuksilla**: Oletusasetukset toimivat hyvin useimmissa MAPIR-kamerajärjestelmissä ja tyypillisissä työnkuluissa.
2. **Luo malleja**: Kun olet optimoinut asetukset tietylle työnkululle tai kameralle, tallenna ne malliksi varmistaaksesi yhdenmukaisuuden eri projekteissa.
3. **Testaa ennen koko aineiston käsittelyä**: Kun kokeilet uusia asetuksia, testaa niitä pienellä osalla kuvia ennen koko aineiston käsittelyä.
4. **Dokumentoi asetuksesi**: Käytä kuvaavia mallinimiä, jotka ilmaisevat kamerajärjestelmän, käsittelytyypin ja käyttötarkoituksen (esim. ”Survey3\_RGB\_NDVI\_Agriculture”).
5. **Vientimuodon valinta**: Valitse vientimuoto lopullisen käyttötarkoituksen mukaan:
   * Tieteellinen analyysi → TIFF (16-bittinen tai 32-bittinen)
   * GIS-käsittely → TIFF (16-bittinen)
   * Nopea visualisointi → PNG (8-bittinen)
   * Jakaminen verkossa → JPG (8-bittinen)

***

Lisätietoja Chloros:n monispektrisistä indekseistä on sivulla [Monispektristen indeksien kaavat](multispectral-index-formulas.md).
