# Projektin asetukset

Chloros-sovelluksen Projektin asetukset <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

-sivupalkissa voit määrittää projektisi kaikki kuvankäsittelyyn, kalibrointikohteiden tunnistukseen, monispektristen indeksien laskentaan ja vientivaihtoehtoihin liittyvät asetukset. Nämä asetukset tallennetaan projektisi mukana, ja ne voidaan tallentaa malleiksi, joita voi käyttää uudelleen useissa projekteissa.

## Projektin asetusten avaaminen

Voit avata projektin asetukset seuraavasti:

1. Avaa projekti Chloros-ohjelmassa
2. Napsauta vasemmassa sivupalkissa **Projektin asetukset** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

-välilehteä
3. Asetuspaneelissa näkyvät kaikki käytettävissä olevat määritysvaihtoehdot luokittain järjestettyinä



<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->

{% hint style="info" %}
**Muista asetuksista riippuvat asetukset näkyvät harmaana.** Kun yläpuolinen valinta estää jonkin asetuksen käytön (esimerkiksi *Heijastavuuskalibrointi / valkotasapaino* -valinnan poistaminen estää *Vie heijastavuus* -toiminnon käytön), riippuva säätö on poissa käytöstä ja sen työkaluvihjeessä mainitaan valinta, jota on muutettava.
{% endhint %}

***

## Näyttö

### Kuvan pikkukuvan resoluutio

* **Tyyppi**: Pudotusvalikko
* **Vaihtoehdot**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Oletus**: Oletus (512 px)
* **Kuvaus**: Resoluutio (pisin sivu, pikseleinä), jolla kuvaruudukon pikkukuvat renderöidään. Suuremmat arvot näyttävät terävämmiltä zoomattaessa, mutta latautuvat hitaammin ja vievät enemmän muistia. Täysi resoluutio vastaa alkuperäisen kuvan kokoa.
* **Huomautus**: Vain näyttöä varten — tämä ei vaikuta milloinkaan käsittelyyn tai vietyihin tiedostoihin.

***

## Kohteen tunnistus

Nämä asetukset määrittävät, miten Chloros tunnistaa ja käsittelee kalibrointikohteita kuvissasi. Molemmat ovat aktiivisia vain, kun **Heijastuskalibrointi / valkotasapaino** on käytössä (muutoin ne ovat harmaana, koska kohteen tunnistus ohitetaan kokonaan).

### Kalibrointikohteen vähimmäispinta-ala (px)

* **Tyyppi**: Luku
* **Alue**: 0–10 000 pikseliä
* **Oletus**: 25 pikseliä
* **Kuvaus**: Määrittää pienimmän alueen (pikseleinä), joka tarvitaan, jotta havaittu alue voidaan katsoa kelvolliseksi kalibrointikohteen näytteeksi. Pienemmät arvot havaitsevat pienempiä kohteita, mutta voivat lisätä vääriä positiivisia tuloksia. Suuremmat arvot vaativat suurempia ja selkeämpiä kohdealueita tunnistamista varten.
* **Milloin säätää**:
  * Suurenna arvoa, jos saat vääriä tunnistuksia pienistä kuvan artefakteista
  * Pienennä arvoa, jos kalibrointikohteesi näkyvät kuvissasi pieninä eivätkä tule tunnistetuiksi

### Kohteiden ryhmittelyn vähimmäisraja (0–100)

* **Tyyppi**: Luku
* **Alue**: 0–100
* **Oletusarvo**: 60
* **Kuvaus**: Säätää ryhmittelykynnystä, jolla samanväriset alueet ryhmitellään kalibrointikohteita tunnistettaessa. Suuremmat arvot edellyttävät, että ryhmiin kootaan enemmän samanlaisia värejä, mikä johtaa konservatiivisempaan kohteiden tunnistukseen. Pienemmät arvot sallivat enemmän värivaihtelua kohderyhmän sisällä.
* **Milloin säätää**:
  * Lisää arvoa, jos kalibrointikohteet jakautuvat useiksi tunnistuksiksi
  * Vähennä arvoa, jos värivaihtelua sisältäviä kalibrointikohteita ei tunnisteta kokonaan

***

## Käsittely

Nämä asetukset säätelevät sitä, miten Chloros-ohjelma käsittelee ja kalibroi kuvasi.

### Vignetoinnin korjaus

* **Tyyppi**: Valintaruutu
* **Oletus**: Käytössä (valittuna)
* **Kuvaus**: Soveltaa vignettikorjausta kompensoimaan objektiivin aiheuttamaa tummenemista kuvien reunoilla. Vignettaus on yleinen optinen ilmiö, jossa kuvan kulmat ja reunat näyttävät keskustaa tummemmilta objektiivin ominaisuuksien vuoksi.
* **Sivuvaikutus**: Tämä valinta määrittää myös, minkä *kalibroimattoman varatuotteen* käsittely tulostaa (katso alla).

### Heijastavuuden kalibrointi / valkotasapaino

* **Tyyppi**: Valintaruutu
* **Oletus**: Käytössä (valittuna)
* **Kuvaus**: Ottaa heijastavuuskalibroinnin käyttöön — joko kuvakehyksessä havaituista kalibrointikohteista ja/tai DAQ-valosensorin alaspäin suuntautuvasta valosta kerätyistä tiedoista, riippuen kamerasta ja käytettävissä olevista tiedoista. Tämä normalisoi heijastavuusarvot koko aineistossasi ja varmistaa yhdenmukaiset mittaukset valaistusolosuhteista riippumatta.
* **Kun pois käytöstä**: Kohteiden tunnistus ohitetaan kokonaan, eikä**mikään kamera voi tuottaa heijastavuustuotetta** — ei Survey3:n kohdepohjainen eikä LATTICE DAQ:n ohjattu. Riippuvat asetukset (*Vie heijastavuus*, *Vähimmäisrekalibrointiväli* ja kohteiden tunnistuksen kynnysarvot) ovat harmaana.

### Kalibroimattomat varatuotteet: Vie anturin vaste / Vie vinjetointikorjattu

* **Tyyppi**: Kaksi valintaruutua
* **Oletusasetukset**: Molemmat käytössä (valittuna)
* **Kuvaus**: Kun kehyksen heijastavuutta ei voida kalibroida (kalibrointikohdetta ei löytynyt tai heijastavuuskalibrointi on pois päältä), se tallennetaan sen sijaan *kalibroimattomana varatuotteena*. **Jokaista kameramallia kohti on täsmälleen yksi näistä kahdesta varatuotteesta kutakin ajokertaa kohti**, ja se valitaan *Vignettikorjaus*-kytkimellä:
  * Vignettikorjaus **päällä**→ `Vignette_Corrected_Images/` (määritetään asetuksella**Vignettikorjattu vienti**)
  * Vignettikorjaus **pois päältä**→ `Sensor_Response_Images/` (määritetään asetuksella**Anturin vasteen vienti**)
* Käytössä oleva varatuote on harmaana. Käytössä olevan valinnan poistaminen estää kyseisen tiedoston tallentamisen kokonaan.

### LATTICE-vientituotteet

LATTICE-kaappauksia sisältävissä projekteissa jokainen tuotu LATTICE-kuva haarautuu jokaiseen käytössä olevaan **ja soveltuvaan**tuotteeseen yhdellä käsittelykierroksella. Neljä valintaruutua ohjaavat jakautumista (kaikki oletuksena**päällä**):

| Asetus | Tulostuskansio | Mitä se vie |
| --- | --- | --- |
| **Vie debayeroitu** | `Debayered_Images/` | Lineaarinen debayeroitu kuva: kalibrointipaketin tumma offset on poistettu, minkä jälkeen suoritetaan reunoja huomioiva demosaic-käsittely ilman lineaarisuuskorjausta, tasokenttää tai värien erottelua — sama tiedosto, jonka tallennus tallentaa debayeroidulla tasollaan. Säilytetään anturin alkuperäisessä suunnassa (tallennettu XMP:hen). Koskee RGB- ja monispektrikameroita. |
| **Vie esikatselu** | `Preview_Images/` | Näytön esikatselu. RGB = valkotasapaino (DAQ-valonlähde, jos saatavilla, muuten harmaa maailma) + gamma; monispektrinen = väärävärinen venytys. |
| **Vie säteilyvoimakkuus** | `Radiance_Images/` | Float32-muotoinen spektrinen säteilyvoimakkuus yksikössä W/m²/sr/nm. Vain monispektriset (M3C/M3M) — ei koske RGB-pääkuvia. Tallennetaan aina 32-bittisenä TIFF-tiedostona *Kalibroitu kuvamuoto* -asetuksesta riippumatta. |
| **Vie heijastavuus**| `Reflectance_Calibrated_Images/` | Uint16-heijastavuus, skaalattu siten, että**32768 = heijastavuus 1,0** (merkitty XMP-tiedostoon `Chloros:PixelScale`). Vain monispektriset kuvat; tallennetaan, kun vastaava `.daq`-alasvirtaustietue (tai laadunvarmistuksen läpäissyt kehyksen sisäinen kohde) peittää kehyksen. |

* RGB-pääkamerat lähettävät debayeroidun + esikatselukuvan; säteilyvoimakkuutta/heijastavuutta ei niille tallenneta, koska ne eivät ole sovellettavissa.
* Debayeroidun/esikatselukuvan bittisyvyys noudattaa *Kalibroitu kuvamuoto* -asetusta; säteilyvoimakkuus on aina float32.
* Nämä neljä kytkintä eivät vaikuta *Survey3 *-käsittelyyn.
* Monispektristen M3C-kameroiden säteily- ja heijastavuuslähdökset voivat myös kirjoittaa pikselikohtaisen **rajoitusmaskin** tuotekansioiden viereiseen `clamp_masks/`-kansioon, tallentaen kohdat, joissa kaistan erottelu on leikanut negatiivisen arvon nollaksi. Katso [Lähtökuvamuodot](../output-image-formats.md#clamp-masks).

Samat neljä kytkintä ovat käytettävissä myös ilman käyttöliittymää nimellä `chloros-cli process --debayered / --preview / --radiance / --reflectance`. Ne korvasivat vanhan CLI-lipun `--radiometric-output`, jota ei enää ole olemassa.

{% hint style="warning" %}
**Kaikkien soveltuvien tuotteiden poistaminen käytöstä aiheuttaa ajon epäonnistumisen.** Versiosta 1.2.0 lähtien käsittelyajo, jolta pyydettiin tuotteita mutta joka ei kirjoittanut yhtään kuvatuotetta, raportoi virheen ja CLI poistuu nollasta poikkeavalla tuloksella sen sijaan, että se ilmoittaisi hiljaisesta onnistumisesta. Lokissa mainitaan tuote, jota ei voitu kirjoittaa, sekä syy siihen. Tarkoituksellisesti vain metatietoja tuottava ajo (mitään ei pyydetty) katsotaan edelleen onnistuneeksi.
{% endhint %}

### Heijastavuuslähde (projektiasetus, määritetään CLI:n kautta)

Projekti tallentaa myös sen, mitä **heijastavuusviitettä** LATTICE-heijastavuustuote käyttää. Asetuspaneelissa ei ole erillistä säätöä; arvo tallennetaan projektin konfiguraatioon nimellä `Processing → "Target reflectance source"` ja se asetetaan komennolla `chloros-cli process --reflectance-source {auto,target,daq}` (SDK:n `configure()`:llä ei ole tätä varten avainsanaa):

* **`auto`** (oletus): laadunvarmistuksen läpäissyt kuvakehyksessä oleva kalibrointikohde toimii absoluuttisena vertailukohtana, ja jos kohdetta ei ole tai laadunvarmistus epäonnistuu, käytetään varajärjestelmänä DAQ:n alaspäin suuntautuvaa jakosuhdetta (ρ = πL/E). Kohdetta, joka läpäisee laadunvarmistuksen varoituksella (paneeli pudonnut tai kaista epäkunnossa), ei automaattisesti aseteta etusijalle: kohde-, kohde+DAQ- ja pelkkä DAQ-ketjut kokeillaan niiden mitatun pistemäärän mukaisessa järjestyksessä testipaneeleilla.
* **`target`**: tiukka kohteeseen perustuva heijastavuus — ei DAQ-korvausta.
* **`daq`**: DAQ-pohjainen heijastavuus; kehyksessä olevia kohteita ei käytetä vertailukohtana.

Tallennettua arvoa verrataan kirjainkokoa huomioimatta, ja muutamia kirjoitusmuotoja hyväksytään aliaksina: `target`, `target_image`, `empirical` ja `empirical_line` tarkoittavat kaikki **kohdetta**; `daq`, `dls`, `light_sensor` ja `sensor` tarkoittavat kaikki**daq**. Kaikki muut – mukaan lukien puuttuva avain – tulkitaan**auto**:ksi.

Yksikkökohtaiset **mitatut** kohdeskannaukset haetaan kohdeyksikön sarjanumeron/QR-koodin perusteella, muodossa `<serial>.csv`, kolmesta paikasta: hakemistosta, joka on määritetty `--target-reflectance-dir`:llä (tallennettu nimellä `Processing → "Target reflectance dir"`), projektin omasta `target_reflectance/`-kansiosta sekä `CHLOROS_TARGET_REFLECTANCE_DIR`-ympäristömuuttujan polusta. Jos kyseiselle yksikölle ei ole olemassa mitattua skannausta, käytetään sen sijaan kohdemallin julkaistua nimelliskäyrää.

### Debayer-menetelmä

* **Tyyppi**: Pudotusvalikko
* **Vaihtoehdot**:
  * Vakio (nopea, keskilaatuinen)
  * Tekstuuritietoinen (hidas, korkein laatu) \[Chloros+]
* **Oletus**: Vakio (nopea, keskilaatuinen)
* **Kuvaus**: Valitsee demosaicing-algoritmin, jota käytetään raakojen Bayer-kuvioisten anturitietojen muuntamiseen täysvärikuviksi. ”Vakio (nopea, keskilaatu)” -menetelmä tarjoaa optimaalisen tasapainon käsittelynopeuden ja kuvanlaadun välillä. ”Tekstuuritietoinen (hidas, korkein laatu)” \[Chloros+] käyttää korkealaatuista, reunoja huomioivaa debayering-menetelmää yhdistettynä tekoäly-/koneoppimismallin avulla toteutettuun kohinanpoistoon, joka poistaa lähes kaiken debayering-kohinan. Texture Aware -mallin suorittaminen vaatii GPU-muistia (VRAM). Suosittelemme sen käyttöä, kun käytettävissä on yli 4 Gt VRAM-muistia, jotta käsittely on nopeampaa.
* **Ainoastaan silloin, kun kyseessä on pudotusvalikko**: kahden vaihtoehdon pudotusvalikko näkyy vain, kun**molemmat** ehdot täyttyvät — olet kirjautunut sisään kelpoisella Chloros+-tilauksella, **ja** projekti ei sisällä LATTICE-kuvauksia. Muussa tapauksessa rivi näkyy pelkkänä tekstinä `Standard (Fast, Medium Quality)` ilman valintamahdollisuutta.
* **LATTICE-huomautus**: LATTICE-koulutettua Texture Aware -mallia ei ole olemassa, ja käsittelyputki pakottaa LATTICE-kehyksille tavallisen demosaicin tallennetusta arvosta riippumatta. Jos lisäät LATTICE-kansion projektiin, jossa Texture Aware oli jo valittuna, Chloros palauttaa asetuksen takaisin Standard-tilaan sen sijaan, että jättäisi vanhentuneen arvon `project.json`:ään.

### Kalibroinnin vähimmäisväli

* **Tyyppi**: Luku
* **Alue**: 0–3 600 sekuntia
* **Oletus**: 0 sekuntia
* **Kuvaus**: Määrittää kalibrointikohteiden käytön välisen vähimmäisajan (sekunteina). Kun arvoksi asetetaan 0, Chloros käyttää jokaista havaittua kalibrointikohdetta. Kun arvoksi asetetaan suurempi luku, Chloros käyttää vain kalibrointikohteita, joiden välinen aika on vähintään tämän sekuntimäärän suuruinen, mikä lyhentää käsittelyaikaa datasarjoissa, joissa kalibrointikohteita tallennetaan tiheästi.
* **Milloin säätää**:
  * Aseta arvoksi 0, jotta kalibrointitarkkuus on mahdollisimman suuri vaihtelevissa valaistusolosuhteissa
  * Suurenna arvoa (esim. 60–300 sekuntiin), jotta käsittely on nopeampaa, kun valaistus on tasainen ja kalibrointikohteiden kuvia on tiheästi

### Valosensorin aikavyöhykkeen siirtymä

* **Tyyppi**: Luku
* **Alue**: -12 – +12 tuntia
* **Oletusarvo**: 0 tuntia
* **Kuvaus**: Määrittää valosensorin tietojen aikaleimojen aikavyöhykkeen siirtymän (tunteina UTC:stä), jota käytetään valosensorilokien sovittamisessa kuvien ottamisajankohtiin. Uudemmat `.daq`-tallenteet sisältävät oman aikavyöhykkeen alkuperätiedon, joten tätä tarvitaan pääasiassa vanhemmille, paikallisaikana tallennetuille lokeille.

### Käytä PPK-korjauksia

* **Tyyppi**: Valintaruutu
* **Oletus**: Pois käytöstä (valinta poistettu)
* **Kuvaus**: Ottaa käyttöön GPS:ää (GNSS) sisältävien MAPIR-DAQ-tallentimien jälkikäsitellyt kinemaattiset (PPK) korjaukset. Kun tämä on käytössä, Chloros käyttää projektikansiossasi olevia .daq-lokitiedostoja, jotka sisältävät valotusaikatietoja, ja soveltaa tarkkoja maantieteellisiä sijaintikorjauksia kuviisi.
* **Vaatimus**: Projektikansiossa on oltava .daq-lokitiedosto, jossa on valotustietueita
* **Milloin otetaan käyttöön**: On suositeltavaa ottaa PPK-korjaus aina käyttöön, jos .daq-lokitiedostossasi on valotustietueita.

### Valotuspiste 1

* **Tyyppi**: Pudotusvalikko
* **Näkyvyys**: Näkyy vain, kun ”Käytä PPK-korjauksia” on käytössä JA valotustietoja on saatavilla nastalle 1
* **Vaihtoehdot**:
  * Projektista tunnistetut kameramallinimet
  * ”Älä käytä” – Ohita tämä valotustappi
* **Oletus**: Valitaan automaattisesti projektin asetusten perusteella
* **Kuvaus**: Määrittää tietyn kameran valotuspinnille 1 PPK-aikasynkronointia varten. Valotuspinnit tallentavat tarkan ajankohdan, jolloin kameran suljin laukeaa, mikä on kriittistä tarkan PPK-paikannuksen kannalta.
* **Automaattisen valinnan toiminta**:
  * Yksi kamera + yksi pin: Valitsee kameran automaattisesti
  * Yksi kamera + kaksi piniä: Pin 1 määritetään automaattisesti kameralle
  * Useita kameroita: Manuaalinen valinta vaaditaan

### Valotuspinn 2

* **Tyyppi**: Pudotusvalikko
* **Näkyvyys**: Näkyy vain, kun ”Käytä PPK-korjauksia” on käytössä JA valotustietoja on saatavilla pinnille 2
* **Vaihtoehdot**:
  * Projektissa tunnistetut kameramallien nimet
  * ”Älä käytä” – Ohita tämä valotuspistoke
* **Oletus**: Valitaan automaattisesti projektin kokoonpanon perusteella
* **Kuvaus**: Määrittää tietyn kameran valotuspistokkeeseen 2 PPK-aikasynkronointia varten, kun käytetään kahden kameran kokoonpanoa.
* **Automaattisen valinnan toiminta**:
  * Yksi kamera + yksi liitin: Liitin 2 asetetaan automaattisesti tilaan ”Älä käytä”
  * Yksi kamera + kaksi liitintä: Liitin 2 asetetaan automaattisesti tilaan ”Älä käytä”
  * Useita kameroita: Manuaalinen valinta vaaditaan
* **Huomautus**: Samaa kameraa ei voi määrittää samanaikaisesti sekä liittimeen 1 että liittimeen 2.

***

## DAQ-valosensori

Tämä osio näkyy Projektin asetuksissa ja luettelee kaikki projektin DAQ-alasuuntaiset tiedostot — `.daq`-tallenteet ja DAQ-M `.csv`-alasvalolokit. Valosensorit-välilehdessä tehdyt tallenteet lisätään avoimeen projektiin automaattisesti.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->



Jokaisella rivillä näkyy tiedosto, anturimalli ja kyseiselle tiedostolle tosiasiallisesti voimassa oleva hajotin-korkin korjaus. Rivien yläpuolella on yksi koko projektia koskeva säätö:

### Korkin ohitus (kaikki tiedostot)

* **Tyyppi**: Pudotusvalikko
* **Vaihtoehdot**: `Auto` sekä projektissa oleville anturityypeille voimassa olevat suojuksen korjausprofiilit
* **Oletus**: Auto
* **Tallennetaan nimellä**: `Processing → "DAQ cap id"` (oletusarvo `auto`)
* **Kuvaus**: `Auto` käyttää kunkin tiedoston tallennettua suojusta (jos mitään ei ole tallennettu, oletuksena käytetään Sunshine-suojusta, mutta DAQ-U-, DAQ-M- ja DAQ-E-laitteissa ei ole suojusta mukana ja Chloros-ohjelmisto ei näe, mikä suojus on asennettu, joten käsittely antaa varoituksen oletetusta korkista: valitse asennettu korkki). Tietyn korkin valinta ohittaa **kaikki** projektin alaspäin suuntautuvat tiedostot paitsi DAQ-E-S:n tiedostot, joiden hajotin on osa sen kalibrointia; projektissa, jossa on vain DAQ-E-S-laitteita, pudotusvalikkoa ei näy: raakatallenteet korjataan sen avulla, ja tallenteet, joissa on jo korkki, viitataan uudelleen (tallennettu korjaus peruutetaan ja valittu korjaus otetaan käyttöön).
* **Tärkeää**: Valitun korkin on vastattava tallennuksen aikana fyysisesti asennettua korkkia. Anturi tai ohjelmisto eivät pysty tunnistamaan fyysistä korkkia — väärä korkkitunnus korjaa spektrit virheellisesti.

Tarkoituksena on, että on **yksi** koko projektia koskeva säätö eikä tiedostokohtaisia pudotusvalikoita: asetus koskee jokaista projektin alaspäin suuntautuvaa lähdettä.

***

## Array-kohdistus

Tämä osio näkyy **vain**, kun vähintään yhdessä projektin kuvassa on moduulien välinen kohdistusmuunnos, jonka LATTICE-matriisit merkitsevät kuvaushetkellä (XMP-tunnisteet `Chloros:Alignment*`). Siinä näkyy, kuinka monessa kuvassa on kohdistustunnisteita, mikä kamera on vertailukamerana (`REF`-tunniste) sekä kamerakohtainen taulukko kuvien lukumääristä.

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
* **Kuvaus**: Rajaa kohdistetut vientitiedostot alueelle, joka on yhteinen kaikille kameramoduuleille, jolloin jokaisella kaistalla on sama peittoalue. Pois-asetus säilyttää koko anturin kuvapinnan (musta täyttö lähteen ulkopuolella).

### Uudelleennäytteenotto

* **Tyyppi**: Pudotusvalikko (aktiivinen vain, kun *Käytä matriisin kohdistusta* on päällä)
* **Vaihtoehdot**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Oletus**: Bilineaarinen
* **Tallennetaan nimellä**: `Processing → "Array alignment interpolation"`
* **Kuvaus**: Kohdistuksen vääristymässä käytettävä interpolointi. Lähin säilyttää tarkat lähdearvot (ei pikselien välistä sekoittumista) tarkkaa radiometristä analyysia varten; Bilineaarinen sopii parhaiten kartoitukseen ja visuaaliseen käyttöön.

Samat kolme vaihtoehtoa ovat käytettävissä ilman etuliitettä nimillä `chloros-cli process --array-alignment`, `--array-alignment-crop` ja `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Indeksi

Näiden asetusten avulla voit määrittää monispektriset indeksit analysointia ja visualisointia varten.

### Lisää indeksi

* **Tyyppi**: Erityinen indeksien määrittämispaneeli
* **Kuvaus**: Avaa interaktiivisen paneelin, jossa voit valita ja määrittää monispektriset kasvillisuusindeksit (NDVI, NDRE, EVI jne.), jotka lasketaan kuvankäsittelyn aikana. Voit lisätä useita indeksejä, joista jokaisella on omat visualisointiasetuksensa.
* **Käytettävissä olevat indeksit**: Käyttöliittymän pudotusvalikossa on**27** ennalta määriteltyä monispektrisen indeksin kaavaa (katso [Monispektrisen indeksin kaavat](multispectral-index-formulas.md) täydellinen luettelo, mukaan lukien nimet, jotka CLI- ja SDK `--indices`-vaihtoehdossa).
* **Ominaisuudet**:
  * Valitse valmiista indeksikaavoista
  * Vedä kamerasi suodatinkanavat kaavan kaistapaikoille
  * Määritä visualisoinnin värigradientit (LUT – hakutaulukot)
  * Aseta kynnysarvot ja leikkausmoodit
  * Luo mukautettuja indeksikaavoja
* **Huomautus**: Indeksejä ei lasketa yksikaistaisille LATTICE M3M -monokameroille — monikaistaiset indeksit ovat määrittelemättömiä yhdellä kaistalla. Tämä ei koske malleja Survey3 ja LATTICE M3C.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->



Jokainen lisäämäsi indeksi näyttää kaavansa matemaattisena, ja jokaisella kaistapaikalla on värillinen ympyrä: punainen = Red, vihreä = Green, sininen = Blue, oranssi = Orange, syaani = Cyan, violetti = NIR, magenta = RE. Vedä ympyrä kaavan yläpuolella olevasta rivistä paikkaan liittääksesi sen; kaksoisnapsauta liitettyä paikkaa tyhjentääksesi sen. Indeksi lasketaan vain kerran, kun jokaisessa kaavan käyttämässä paikassa on kanava.

### Mukautetut kaavat (Chloros+ -ominaisuus)

* **Tyyppi**: Mukautettujen kaavojen määritelmien taulukko
* **Saatavuus**: Vaatii kirjautumisen kelpoisella Chloros+-tilauksella.
* **Kuvaus**: Mahdollistaa mukautettujen monispektristen indeksikaavojen luomisen ja tallentamisen kaistamatematiikkaa käyttäen. Mukautetut kaavat tallennetaan projektiasetuksiisi, ja niitä voidaan käyttää aivan kuten sisäänrakennettuja indeksejä.
* **Luominen**:
  1. Avaa indeksin asetuspaneelissa mukautettujen kaavojen laskin
  2. Kirjoita kaava käyttämällä **kaistapaikkasymboleja**, älä kaistojen nimiä
  3. Tallenna kaava kuvaavalla nimellä — se näkyy sitten kaavojen pudotusvalikon alaosassa, ja voit vetää kamerasi kanavapiirejä sen paikoille täsmälleen kuten sisäänrakennetun esiasetuksen kohdalla
* **Kaavan syntaksi**:
  * Kaistapaikat: `x`, `y`, `z`, `a`, `b`, `c` — kuusi paikkaa, jotka määritetään todellisille kanaville vetämällä
  * Operaattorit: `+`, `-`, `*`, `/`, `^` ja `()` ryhmittelyä varten
  * Toiminnot: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Miksi symboleja, ei bändinimiä**: kaava, joka on kirjoitettu muodossa `(y-x)/(y+x)`, toimii millä tahansa kameralla, koska vedä ja pudota -kartoitus päättää, onko `y` RGN-suodattimen 850 nm:n NIR vai OCN-suodattimen 808 nm:n NIR. sisäänrakennetut esiasetukset tallennetaan samalla tavalla — katso [Monispektriset indeksikaavat](multispectral-index-formulas.md), josta löydät kaikkien 27 kaavan tarkat symbolimuodot.
* **Käyttöalue**: mukautetut kaavat tallennetaan projektin asetusten mukana, ja niitä voidaan käyttää sekä [indeksi-/LUT-hiekkalaatikossa](../image-viewer-gui/index-lut-sandbox.md) että kuvankäsittelyssä. CLI- ja SDK-nimiluettelo (`--indices`) **ei** tue niitä, sillä se laajentaa vain 22 sisäänrakennettua esiasetuksen nimeä.

***

## Vienti

Nämä asetukset määrittävät vietyjen käsiteltyjen kuvien muodon ja laadun.

### Kalibroidun kuvan muoto

* **Tyyppi**: Pudotusvalikko
* **Vaihtoehdot**:
  * **TIFF (16-bittinen)** – Pakkaamaton 16-bittinen TIFF-muoto
  * **TIFF (8-bittinen)** – Pakkaamaton 8-bittinen TIFF-muoto
  * **TIFF (32-bittinen, prosentti)** – 32-bittinen liukulukuinen TIFF, jossa heijastusarvot on ilmaistu prosentteina
  * **PNG (8-bittinen)** – Pakattu 8-bittinen PNG-muoto
  * **JPG (8-bittinen)** – Pakattu 8-bittinen JPEG-muoto
* **Oletus**: TIFF (16-bittinen); kokonaan 8-bittisessä projektissa näkyy TIFF (8-bittinen), ja tarjolla on vain 8-bittisiä muotoja
* **Kuvaus**: Valitsee tiedostomuodon, johon käsitellyt ja kalibroidut kuvat tallennetaan. Viedyt tiedostot tallennetaan kunkin kameran kansion sisällä olevaan, muotoa vastaavaan alikansioon (`tiff16`, `tiff32`, `png8`, `jpg8`), ja jokaiselle tuotteelle on yksi `<Product>_Images/`-kansio. Viedyt tiedostot säilyttävät lähdetiedoston nimen — tuotteen tunnistaa kansio, ei tiedostonimen pääte.
* **Tiedostomuotojen suositukset**:
  * **TIFF (16-bittinen)**: Suositellaan tieteelliseen analyysiin ja ammattimaisiin työnkulkuihin. Säilyttää parhaan mahdollisen datan laadun ilman pakkausartefakteja. Sopii parhaiten monispektrianalyysiin ja jatkokäsittelyyn GIS-ohjelmistoissa.
  * **TIFF (32-bittinen, prosentti)**: Sopii parhaiten työnkulkuihin, joissa heijastusarvot tarvitaan prosentteina (0–100 %). Tarjoaa suurimman tarkkuuden radiometrisille mittauksille.
  * **PNG (8-bittinen)**: Sopii hyvin verkkokatseluun ja yleiseen visualisointiin. Pienemmät tiedostokoot häviöttömällä pakkauksella, mutta pienempi dynaaminen alue.
  * **JPG (8-bittinen)**: Pienimmät tiedostokoot, sopii parhaiten vain esikatseluun ja verkkonäyttöön. Käyttää häviöllistä pakkausta, joka ei sovellu tieteelliseen analyysiin.
* **Huomautus**: LATTICE-säteilyarvot viedään aina 32-bittisenä float-TIFF-tiedostona tästä asetuksesta riippumatta.

***

## Tallenna projektimalli

Tämän ominaisuuden avulla voit tallentaa nykyiset projektiasetuksesi uudelleen käytettäväksi malliksi.

* **Tyyppi**: Tekstikenttä + Tallenna-painike
* **Kuvaus**: Anna asetusmallillesi kuvaava nimi ja napsauta tallennuskuvaketta. Malli tallentaa kaikki nykyiset projektiasetuksesi (kohteen tunnistus, käsittelyasetukset, indeksit, ja vientimuoto), jotta niitä on helppo käyttää uudelleen tulevissa projekteissa. Mallit tallennetaan projektin tallennuskansion sisällä olevaan `Project Templates/`-kansioon, ja ne voidaan myös valita tai viedä päävalikosta (*Valitse malli* / *Tallenna malli* / *Vie malli*).
* **Käyttötapaukset**:
  * Luo malleja eri kamerajärjestelmille (RGB, monispektrinen, NIR)
  * Tallenna vakioasetukset tietyille viljelykasvilajeille tai analyysityönkuluille
  * Jaa yhdenmukaiset asetukset koko tiimille
* **Käyttöohjeet**:
  1. Määritä kaikki haluamasi projektin asetukset
  2. Anna mallille nimi (esim. ”RedEdge Survey3 NDVI Standard”)
  3. Napsauta tallennuskuvaketta
  4. Malli voidaan nyt ladata uusia projekteja luotaessa

***

## Projektikansio

Tämä asetus määrittää, mihin uudet projektit tallennetaan oletuksena.

* **Tyyppi**: Hakemistopolun näyttö + Muokkaa-painike
* **Oletus (Windows)**: `Documents\MAPIR\Chloros Projects` Asiakirjat-kansiossa — yleensä `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`, tai missä tahansa OneDrive tai vaeltava profiili on siirtänyt Asiakirjat-kansion
* **Oletus (Linux)**: `~/Documents/MAPIR/Chloros Projects`
* **Kuvaus**: Näyttää nykyisen oletuskansion, johon uudet Chloros-projektit luodaan. Napsauta muokkauskuvaketta valitaksesi toisen kansion. Muutos tallennetaan yhtenä tekstirivinä tiedostoon `~/.chloros/working_directory.txt` — Windows issa se on `C:\Users\<Username>\.chloros\working_directory.txt`. Jos kyseistä tiedostoa ei ole tai se viittaa polkuun, jota ei enää ole olemassa, Chloros käyttää yllä mainittua oletusarvoa. CLI lukee ja kirjoittaa samaa tiedostoa, joten `chloros-cli` ja graafinen käyttöliittymä ovat aina yhtä mieltä projektien sijainnista.
* **Projektimallit** sijaitsevat tämän hakemiston `Project Templates/`-alikansiossa.
* **Milloin muuttaa**:
  * Aseta verkkoasemalle tiimityöskentelyä varten
  * Vaihda asemaan, jossa on enemmän tallennustilaa suurille tietojoukoille
  * Järjestä projektit vuosittain, asiakkaittain tai projektityypeittäin eri kansioihin
* **Huomautus**: Tämän asetuksen muuttaminen vaikuttaa vain UUSIIN projekteihin. Olemassa olevat projektit pysyvät alkuperäisissä sijainneissaan.
* **Windows-polun pituus**: ellei Windows in pitkiä polkuja ole otettu käyttöön (asennusohjelman **Ota Windows in pitkät polut käyttöön** -vaihtoehto), projektikansio, jonka polun pituus ylittää noin 130 merkkiä, hylätään ennen käsittelyä, koska sen tulokset ylittäisivät Windows in 260 merkin polkurajan. Katso [Käsittelyn aloittaminen](../processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

***

## Asetusten säilyminen

Chloros-projekti on **kansio**. Kaikki projektin asetukset tallennetaan sen sisällä olevaan tiedostoon `project.json`; liitetyt laitteet tallennetaan niiden ohella tiedostoihin `cameras.json` ja `sensors.json`, joten projektin avaaminen uudelleen yhdistää myös sen kamerat ja valosensorit uudelleen. Kun avaat projektin uudelleen, kaikki asetukset palautuvat täsmälleen sellaisina kuin ne olivat, kun suljit projektin. Tallennettuja projekteja voidaan ohjata myös ilman käyttöliittymää `chloros-cli project`-tiedoston tai SDK:n `open_project`-tiedoston avulla.

### Asetusten hierarkia

Asetukset otetaan käyttöön seuraavassa järjestyksessä:

1. **Järjestelmän oletusasetukset** – Chloros in määrittämät sisäänrakennetut oletusasetukset 2. **Malliasetukset** – Jos lataat mallin projektia luodessasi
3. **Tallennetut projektiasetukset** – Projektitiedoston mukana tallennetut asetukset
4. **Manuaaliset muutokset** – Kaikki muutokset, jotka teet nykyisen istunnon aikana

### Asetukset ja kuvankäsittely

Käsittelyasetukset luetaan, kun käsittelykäynnistys alkaa. Asetuksen muuttaminen ei vaikuta takautuvasti levyllä jo oleviin tuotteisiin – suorita käsittely uudelleen, jotta uudet asetukset otetaan käyttöön. Muutamat asetukset eivät vaikuta käsittelyyn lainkaan:

* Kuvan pikkukuvan resoluutio (vain näyttöä varten)
* Tallenna projektimalli
* Tallenna projektikansio

***

## Konfiguraatioavainten viite

Automaatiota varten (CLI: `--config`, SDK: `configure` tai lukemalla `project.json` suoraan) nämä ovat tarkat avaimet `Project Settings`:n alla:

| Avainpolku | Tyyppi | Oletus |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | luku 0–10000 | `25` |
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
| `Processing → Minimum recalibration interval` | luku 0-3600 | `0` |
| `Processing → Light sensor timezone offset` | luku -12..12 | `0` |
| `Processing → Apply PPK corrections` | bool | `false` |
| `Processing → DAQ cap id` | cap-profiilin tunnus tai `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | luettelo indeksimäärityksistä | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (8-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

`Array alignment`-avaimet kirjoitetaan, kun Array Alignment -osio renderöidään ensimmäisen kerran tai kun automaatiokutsu asettaa ne. Niiden puuttuessa putki käyttää samoja arvoja kuin yllä on esitetty (`true`, `true`, bilineaarinen), joten projekti, jossa niitä ei ole, käyttäytyy täsmälleen samalla tavalla kuin projekti, jossa ne ovat.

### `project.json`:ään tallennetut avaimet, joita ei voi hallita asetuspaneelissa

Nämä sijaitsevat samassa `Project Settings`-puussa ja ne luetaan käsittelyn yhteydessä, mutta sivupalkista ei löydy niille omaa widgetiä:

| Avainpolku | Tyyppi | Oletus | Asettaja |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK `input_level=`. Ohittaa LATTICE-syöttötiedostojen (TIFF) tulkinnan; `auto` päättelee kunkin tiedoston `Chloros:ProcessingLevel` XMP-tunnisteen ja kanavien lukumäärän perusteella. Ohitetaan Survey3-tallenteissa `.raw`. Tarkoituksella ei ole GUI-asetusta — automaattinen asetus on oikea kaikissa normaaleissa tapauksissa. |
| `Processing → Target reflectance dir` | polkumerkkijono | `""` | `chloros-cli process --target-reflectance-dir`, tai projektin kohde-API |
| `Processing → Target reflectance config` | kameran sarjanumeron mukaan indeksoitu sanakirja | `{}` | Kehyksessä olevan kohteen rekisteröinti (tila `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | polkumerkkijono | `""` | SDK `process_folder(daq_log_path=…)`. Osoittaa `.daq`-tallenteeseen tai niiden kansioon |
| `Target Detection → Minimum calibration target squares` | numero | `4` | Vanha oletusarvo; ei hallintaa eikä CLI-lippua |
| `UI → Grid thumbnail size` | numero | `160` | Kuvaruudukon oma pikkukuvien zoomausliukusäädin |

Kaksi katseluasetusta on tallennettu **ylätasolle `project.json`**:ään, kokonaan `Project Settings`:n ulkopuolelle, koska ne koskevat näyttötilaa eivätkä käsittelyasetuksia:

| Avainpolku | Tyyppi | Oletusarvo | Asettaja |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | kokonaisluku 1–256 | `1` | Kuvavälilehden GSD (px) -säädin — katso [Kuvan avaaminen koko näytön tilassa](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Parhaat käytännöt

1. **Aloita oletusasetuksilla**: Oletusasetukset toimivat hyvin useimmissa MAPIR-kamerajärjestelmissä ja tyypillisissä työnkuluissa.
2. **Luo malleja**: Kun olet optimoinut asetukset tietylle työnkululle tai kameralle, tallenna ne malliksi, jotta varmistat yhdenmukaisuuden eri projekteissa.
3. **Testaa ennen koko aineiston käsittelyä**: Kun kokeilet uusia asetuksia, testaa niitä pienellä kuvajoukolla ennen koko aineiston käsittelyä.
4. **Dokumentoi asetuksesi**: Käytä kuvaavia mallinimiä, joista käy ilmi kamerajärjestelmä, käsittelytyyppi ja käyttötarkoitus (esim. ”Survey3\_RGB\_NDVI\_Agriculture”).
5. **Vientimuodon valinta**: Valitse vientimuoto lopullisen käyttötarkoituksesi mukaan:
   * Tieteellinen analyysi → TIFF (16-bittinen tai 32-bittinen)
   * GIS-käsittely → TIFF (16-bittinen)
   * Nopea visualisointi → PNG (8-bittinen)
   * Jakaminen verkossa → JPG (8-bittinen)

***

Lisätietoja Chloros-ohjelmiston monispektrisistä indekseistä on sivulla [Monispektristen indeksien kaavat](multispectral-index-formulas.md).
