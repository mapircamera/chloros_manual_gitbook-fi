# Tallennusasetukset ja -tilat

Tallennus ”Kamerat”-välilehdellä tapahtuu yhden punaisen **Tallenna kaikki**-painikkeen ja yhden**Tallennusasetukset**-paneelin avulla, jossa määritetään, mitä kyseinen painike tuottaa: mitkä kamerat osallistuvat tallennukseen, mitä vientimuotoja kukin kamera tallentaa ja laukeaako suljin kerran, jatkuvasti vai tietyn aikavälin välein. Tällä sivulla kuvataan koko prosessi – konfigurointi, itse kuvaus, tiedostojen tallennuspaikka levyllä sekä se, miten ne voidaan myöhemmin käsitellä uudelleen kalibroituiksi tuotteiksi. Itse kamera- ja ryhmäasetukset löytyvät kohdasta [Kamera-asetukset](camera-settings.md).

{% hint style="info" %}
**Kuvaus edellyttää avointa projektia.** ”Capture All” (Ota kaikki) ja ”Capture Settings” (Kuvausasetukset) -hammasratas ovat pois käytöstä, kunnes projekti on avoinna (”Luo tai avaa projekti kuvien tallentamiseksi”). Jokainen kuvaus tallennetaan projektikansioon kohdassa `captures/`.
{% endhint %}

## Tallennusasetukset-paneeli

Avaa se sivupalkin kameraluettelossa olevan **”Tallenna kaikki” -painikkeen vieressä olevalla rataspainikkeella**tai minkä tahansa kamerakohtaisen asetuspaneelin alaosassa olevalla**”Avaa tallennusasetukset…”** -painikkeella. Otsikossa lukee ”Tallennusasetukset” ja siinä on ←-paluupainike.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->



Täällä tekemäsi valinnat – mukana olevat kamerat, tyyppikohtaiset valintaruudut ja tallennustila – tallennetaan **projektikohtaisesti** ja palautetaan, kun avaat projektin uudelleen.

### Tallennustilat

Paneelin yläosassa on kolme tilapainiketta:

| Tila | Toiminto | Alavalinnat (oletusarvot) |
| --- | --- | --- |
| **Yksittäinen** *(oletus)* | Yksi tallennus kaikista valituista kameroista. | — |
| **Jatkuva**| Peräkkäisiä tallennuksia, kunnes pysäytysolosuhde täyttyy. | Pysäytys**kuvausmäärän** perusteella (oletus 1) *tai* **kuvauskeston** perusteella (oletus 10 s; yksiköt: sekunnit / minuutit / tunnit / päivät). |
| **Väli**(aikaväli) | Sarjakuvaus ajastimen mukaan. |**Kuvia / väli**(oletus 1) ·**Joka**N yksikköä (oletus 5 s) ·**Kestää** N yksikköä (oletus 1 m). |

Jatkuvassa tai väli-tilassa Kaikki-painike muuttuu **Pysäytä (N)** -painikkeeksi suorituksen aikana, ja se laskee otoksia niiden saapuessa. Täysi tallennuslaite pysäyttää ajon (katso [Kaikki tallennukset -prosessi](#the-capture-all-flow)).**Jatkuva-tila ryhmissä toimii taustapalvelimessa (uusi ominaisuus versiossa 1.3.0).** Kun kaikki luettelossa olevat kamerat kuuluvat ryhmään (erilliset kamerat on piilotettu tai suljettu pois), Jatkuva-tila ei enää lähetä yhtä pyyntöä kutakin kaappausta kohti: Chloros-taustapalvelu suorittaa kaappaussilmukan itse, kirjoittaen yhden synkronoidun ryhmän samalla kun se valitsee seuraavan, ja painike tarkistaa sen lukumäärän. Kun luettelossa on erillisiä kameroita, Chloros kaappaa passikohtaisesti kuten aiemminkin.

{% hint style="warning" %}
**Nopea tallennus Windows issa: pidä Chloros-taustapalvelu pois Microsoft Defenderin tarkistusalueelta.** Jokaisen nopealla tallennuksella kirjoitetun tiedoston reaaliaikainen skannaus kuluttaa noin 30 % tallennusnopeudesta MAPIR in testeissä (satoja tiedostoja sekunnissa pienellä ROI-alueella). Windows-asennusohjelma tarjoaa vaihtoehdon **”Jätä Chloros-taustapalvelu Microsoft Defenderin skannauksen ulkopuolelle (paljon nopeampi kameran tallennus; suositeltava)”**, joka on valittuna oletuksena ja poistaa poissulkemisen asennuksen poistamisen yhteydessä. Jos olet poistanut valinnan, suorita asennusohjelma uudelleen valinnan ollessa valittuna tai lisää Defender-prosessin poissulkeminen kohteelle `chloros-backend.exe` (Chloros-asennuskansion `backend`-kansiossa). Kun Defender käyttää yhtä tai useampaa CPU-ydintä matriisin jatkuvan tallennuksen aikana, Chloros-taustapalvelu merkitsee tämän kerran lokiinsa ja ehdottaa samoja kahta korjaustoimenpidettä.
{% endhint %}

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->




### Kameroiden ja vientityyppien valinta

Paneelin ohjeteksti tiivistää asian: valitse, mitkä kamerat ja vientityypit ”Capture All” tuottaa — oletusarvoisesti kaikki ovat valittuna, ja valinnat tallennetaan tämän projektin yhteydessä.

* **Valitse kaikki / Älä valitse mitään** -painikkeet kytkevät jokaisen kameran valintaruudun päälle tai pois kerralla.
* **Vientityyppien joukkovalinnat**(kaksi riviä painikkeita):**Kaikki Raw / Kaikki Debayered / Kaikki esikatselu / Kaikki säteily / Kaikki heijastavuus / Kaikki indeksi**. Jokaisella on kolmivaiheinen värikoodi: vihreä ✓ = käytössä kaikissa kameroissa, jotka tukevat sitä, keltainen – = käytössä joissakin, harmaa = ei missään. Valintakytkin on pois käytöstä, jos yksikään kytketty kamera ei tue kyseistä tyyppiä. Ne kaikki muuttuvat harmaiksi, kun Fastest Capture -toiminto on päällä.
* **Kamerakohtaiset rivit**: valintaruutu sekä laajennettava (▸/▾) luettelo kyseisen kameran soveltuvista vientityypeistä, joissa on erilliset valintaruudut. Rivillä näkyy käyttöönottojen lukumäärä, kuten ”4/6”.

### Vientityypit ja mitkä kamerat tukevat niitä

Vientityyppejä on kuusi: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Kunkin kameran rivillä näkyvät vain kyseiselle kameralle soveltuvat tyypit:

| Vientityyppi | Sisältö | RGB (FRGB) | Bayer-monispektrinen (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Bayer-mosaiikki (mono: yksi kaista) suoraan anturista | ✓ | ✓ | ✓ |
| **Debayered** | Lineaarinen demosaikki: kalibrointipaketin tumma offset (DSNU) poistettu raakakuvasta, minkä jälkeen suoritetaan reunoja huomioiva demosaikki — ei lineaarisuuskorjausta, tasokenttää tai värien erottelua. Tallennetaan 3-kanavaisena RGB-TIFF-tiedostona (mono: 1-kanavainen harmaasävy) | ✓ | ✓ | ✓ |
| **Esikatselu** | Täydellinen näyttöketju (valkotasapaino + gamma kameran profiilin mukaan sekä RGB-kameran Highlights-asetus; monispektrinen: väärävärinen venytys) | ✓ | ✓ | ✓ |
| **Säteilyvoimakkuus** | float32 W/m²/sr/nm koko radiometrisen ketjun kautta | — (ei tarjolla) | ✓ | ✓ |
| **Heijastavuus** | uint16 ρ (32768 = 1,0) | — (ei tarjolla) | ✓ — näkyy vain, kun kamerassa on DAQ-valosensori (oma tai matriisista peritty) | sama kuin monispektrinen |
| **Indeksi** | Kasvillisuusindeksi (LUT) | — | ✓ — vaatii kamerassa käytössä olevan, ei-tyhjän indeksilausekkeen, eikä sitä tarjota yhdistettyjen matriisien jäsenille (matriisi omistaa yhden jaetun indeksin) | — (indeksi vaatii vähintään 2 kaistaa; katso [Mono-kamerat ja kasvillisuusindeksit](mono-indices.md)) |

Säteilyvoimakkuutta ja heijastavuutta ei koskaan tarjota RGB-kameroille — Bayer-kohtainen säteilyvoimakkuus ei ole merkityksellistä laajakaistaiselle fotometriselle anturille.

### Nopein tallennus

**⚡ Nopein tallennus — vain raakadata**-kytkin (oranssi, kun päällä) ohittaa kaikki vientivalinnat ja asettaa ne**vain raakadataksi** — sekä tarjoaa ilmaisen yhdistettyjen indeksien yhdistelmän matriiseille — jotta kuva tallentuu mahdollisimman nopeasti: säteilyvoimakkuuden/heijastavuuden/näytön laskelmat ohitetaan kokonaan tallennushetkellä.

{% hint style="info" %}
**`.daq` tallennetaan silti.** Kun valosensori on määritetty, Nopein tallennus tallentaa silti DAQ:n alaspäin suuntautuvan lukeman (tallenteiden `daq/`-kansioon) — joten säteilyvoimakkuus-, heijastavuus- ja indeksituotteet voidaan kaikki luoda myöhemmin uudelleenkäsittelyllä (katso [Tallenteiden uudelleenkäsittely](#re-processing-captures-into-calibrated-products)). Fastest Capture ei myöskään vaikuta valintaruutujen valintoihin: kun toiminto kytketään pois päältä, valinnat palautuvat.
{% endhint %}

### Ryhmäkohtaiset asetukset

Jokaisella liitetyllä ryhmällä on oma ryhmäkorttinsa paneelissa:

* **Sisällytä-valintaruutu** (kolmiasentoinen jäsenien välillä) sekä ryhmän nimi ja sen näyttötila: ”(yhdistetty | erillinen)”.
* **Kohdistettu**-valintaruutu (oletus**päällä**): vääristää jäsenien vientiä matriisin kohdistusprofiilin mukaiseksi, jolloin vienti on pikselitasolla rekisteröity kameroiden välillä. Raakadata pysyy vääristämättömänä, mutta sisältää muunnoksen metatiedoissaan. (Profiili itsessään lasketaan [matriisin asetuspaneelissa](camera-settings.md#alignment-co-registration).) Matriisin jäsenetiedostot tallennetaan kunkin kameran alkuperäisessä suunnassa, ja mahdolliset peilaus- tai kiertämisasetukset tallennetaan tiedostoon sen sijaan, että niitä sovellettaisiin.
* Jäsenkameroiden rivit on upotettu kortin sisään.

Matriisikortti sisältää myös kaksi tallenninta. Ajattele niitä **seurannan ja analyysin** vastakohtina:

| Tallennin | Luokka | Mitä se tallentaa |
| --- | --- | --- |
| **● Tallenna indeksivideo / ■ Lopeta tallennus** *(vain yhdistetyt matriisit)* | **Seuranta** | Yhdistetty indeksi-komposiittikuva videona 10 kuvaa sekunnissa — 8-bittinen, esikatseluresoluutio, LUT sisäänrakennettuna. Vaatii avoimen projektin ja suoratoistettavan reaaliaikaisen näkymän. Näyttää kehykset ja kuluneen ajan tallennuksen aikana. |
| **⦿ Tallenna raakasarja / ■ Lopeta raakasarja** *(mikä tahansa kuvasarja)* | **Analyysi**| Raakamuotoiset Bayer-kehykset reaaliaikaisella kaappausnopeudella (ilman käsittelyä) sekä kehyskohtainen manifesti ja `.daq`-lukemat, tallennettuna tiedostoon `captures/bursts/`. Sarjakuvauksen jälkeen näkyviin tulee**Luo video** -painike: se käsittelee sarjakuvauksen offline-tilassa kalibroiduksi videoksi — yhdistetty indeksi ja/tai kamerakohtainen säteilyvoimakkuus / heijastavuus / indeksi — sekä valinnaiset TIFF-tiedostot. Yhdistetyn indeksin luominen käynnistyy automaattisesti, kun lopetat sarjakuvauksen. |

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

## Capture All -työnkulku

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->



Paina **Capture All** -painiketta sivupalkin kameraluettelossa:

1. Jokainen mukana oleva, näkyvä ja keskeyttämätön kamera tallentaa valituilla vientityypeillä. **Ryhmät laukeavat yhtenä synkronoituna laukaisuna** (yksi synkronoitu ryhmä kaikkien jäsenten kesken — katso [Monikameraryhmät](arrays.md)); erilliset kamerat tallentavat yksitellen.
2. Piilotetut (silmä) tai keskeytetyt kamerat ohitetaan. Ryhmä estetään kokonaan vasta, kun *kaikki* sen jäsenet ovat piilotettuja tai keskeytettyjä.
3. Aina kun valosensori on määritetty, vastaava DAQ-alasuuntaisen valon lukema tallennetaan `.daq`-tiedostona tallenteiden `daq/`-kansioon — jopa pelkästään raakatietotallenteiden kohdalla — jotta radiometriset tuotteet voidaan aina johtaa myöhemmin.
4. Painike näyttää reaaliaikaista edistymistä — ”Tallennetaan… valmis/yhteensä” — ja muuttuu Jatkuva-/Väli-tilassa muotoon **Pysäytä (N)**. Jokaisella tallennuskohteella on 300 sekunnin aikakatkaisu.
5. Kun tallennuskerta päättyy, tulosilmoituksessa näkyy **”Tallennettu N tiedostoa”**tai**”Tallennettu N, F epäonnistui”**sekä ”(S piilotettu/keskeytetty ohitettu)”, jos kameroita on ohitettu. Jos levy täyttyy, ajon suoritus keskeytyy ja näyttöön tulee viesti**”Pysäytetty: levy täynnä. Tallennettu N, F epäonnistui”** — jokainen tiedosto, jota ei voitu tallentaa, lasketaan epäonnistuneeksi.

## Tallennusten sijainti

Tallenteet tallennetaan avoimen projektin alle kansioon `<project>/captures/`. Jokainen vientityyppi tallennetaan omaan **alikansioonsa**, joten monitasoisessa tallennuksessa eri tyypit eivät sekoitu keskenään:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_clampmask.tif, reflectance/…, index/… — unmix clamp masks,
│                  one subfolder per product folder, only when a band was clamped
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── daq/           the downwelling reading(s) matched to each capture (.daq), each with a
                   .daq.cameras.json naming the cameras it describes
```

* `<ts>` on tallennuksen aikaleima ja `<serial>` kameran sarjanumero. Yksittäiset tallenteet nimetään muodossa `capture_<ts>_SN<serial>_<level>`; yhden synkronoidun laukaisimen tuottamat sarjatallenteet nimetään muodossa `sync_<ts>_SN<serial>_<level>`, ja **niillä on yhteinen aikaleima ryhmän kaikille kameroille** (tason jälkiliite jätetään pois, kun kamera tallentaa vain yhden tason).
* **Yksi huomioitava poikkeus:** näyttötaso tallennetaan kansioon nimeltä `preview/`, kun taas tiedostojen nimissä säilyy `_display` — kansio ja loppuliite eroavat toisistaan vain kyseisen tason osalta.
* Tuntemattomat tasot tallennetaan omaa nimeään kantavaan kansioon; jos alikansiota ei voida luoda, tiedosto tallennetaan tallenteiden juurikansioon sen sijaan, että se menetettäisiin.
* Tallennetut TIFF-tiedostot pakataan oletusarvoisesti häviöttömästi (DEFLATE), ja niiden täydelliset kalibrointi- ja käsittelymetatiedot sisältyvät **tiedoston XMP-tiedostoon** — tallenteet ovat itsestään selittäviä, eikä niillä ole muita sivutiedostoja kuin `.daq`-tiedosto (ja sen `.daq.cameras.json`).
* **Clamp-maskit (uusi ominaisuus versiossa 1.3.0).**Jos 3×3-spektrinen erottelu joutui rajoittamaan kaistan arvoon 0 vähintään yhdellä pikselillä, säteily-, heijastus- ja indeksituotteet saavat pikselikohtaisen maskin, `clamp_masks/<product folder>/<product name>_clampmask.tif` (bitti 0 = tuotteen R-kanavassa oleva kaista, bitti 1 = G, bitti 2 = B); tuotteen `Chloros:UnmixClampMask` XMP-tunniste viittaa siihen. Naamio-tiedostot sijaitsevat**tuotekansioiden vieressä**, eivät koskaan niiden sisällä, joten tuotekansio, kuten `reflectance/`, voidaan siirtää fotogrammetriaohjelmistolle sellaisenaan. Chloros in oma tuontitoiminto ohittaa nämä naamio-tiedostot.
* **Suunta (1.3.0).** Itsenäisen kameran Preview-, Radiance-, Reflectance- ja Index-tuotteiden suuntaa muutetaan Mirror/Rotation-asetuksella; Raw- ja Debayered-tiedostot pysyvät sellaisina kuin anturi ne luki. Array-ryhmän tiedostot pysyvät kaikki alkuperäisessä muodossaan. Jokainen tiedosto tallentaa asetuksen XMP-tietoihinsa, joten uudelleenkäsittely soveltaa sitä samalla tavalla.
* **Mihin kameraan `.daq` kuuluu (1.3.0).** Kuvauksen `.daq`-tunnus yhdistetään käsittelyvaiheessa vain niiden kameroiden kuviin, jotka sen `.daq.cameras.json`-tunnus nimeää. Itsenäinen kamera, jolla ei ole omaa valosensoria, ei siis enää lainaa toisen kameran lukemaa — liitä siihen sensori tai tallenna sellainen Valosensorit-välilehdessä. Valosensorin tallennus (ja kaikki `.daq`-tiedostot, joissa kyseistä tiedostoa ei ole, mukaan lukien vanhemmat tallenteet) palvelee edelleen jokaista kameraa.

Tämä on sama rakenne, jonka `chloros-cli lattice capture` / `array-capture` kirjoittavat `-o`-hakemistoonsa — dokumentoitu kohdassa [CLI-viite § Miltä tallennuskansio näyttää](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, the daq/ and clamp_masks/ folders, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Kaappausten uudelleenkäsittely kalibroiduiksi tuotteiksi

Kaapatut raakakuvat sekä tallennettu `.daq` ovat kaikki, mitä käsittelyputki tarvitsee — tämän vuoksi Fastest Capture on turvallinen käyttää todellisessa työssä.

* **GUI**: lisää tallennuskansio projektiin ([Tiedostojen lisääminen projektiin](../processing-images-gui/adding-files-to-a-project.md)) ja käsittele kuten tavallisesti.
* **CLI**: osoita `process` **tallennuskansion juurihakemistoon**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` tuo yleensä vain nimeämäsi kansion, mutta jos kyseisessä kansiossa ei ole kuvia ja siinä on alikansioita, se etenee automaattisesti alikansioihin — joten alikansiot ja `daq/`-kansio haetaan kerralla. Jokainen tallennus tuodaan **yhtenä kuvana**, johon muut tasot on liitetty katselutilana, ei yhtenä kuvana per taso.

Tason alikansion nimeäminen suoraan (esim. `…/captures/raw/`) toimii myös, mutta jättää `daq/`-kansion pois — kopioi se rinnalle, kun-johdat radiometristä tuotetta kansiosta `raw/`, muuten aikaleimojen täsmäytys ei löydä vastaavuutta.

{% hint style="warning" %}
**Käsittely alkaa aina kansiosta `raw`.** Kussakin tallennuksessa raakakuva on prosessiketjun lähde; `debayered`, `radiance`, `reflectance`, ja `preview` ovat katseltavissa olevia tiloja, mutta niitä ei koskaan syötetä takaisin prosessointiputkeen — johdetun tuotteen uudelleenkäsittely soveltaisi uudelleen sen pikseleihin jo upotettuja vinjetointia, väriä ja säteilyn laskentaa, joten Chloros kieltäytyy siitä kaksoiskäsittelyn sijaan. `index/`- ja `composite/`-renderöintejä ei käsitellä lainkaan (ne ovat tulosteita, eivät tallenteita). Tallennettu captures-kansio, jossa **ei** ole raakatiedostojen tuontia, näkyy normaalisti, mutta `process` ohittaa sen ja ilmoittaa siitä; `--input-level {raw,debayered,processed}` on tarkoituksellinen pakotie, joka pakottaa aloituskohdan. Katso tarkat ohitusviestit [CLI-viitteestä](../reference/cli-reference.md#what-a-captures-folder-looks-like).
{% endhint %}

Kaksi muuta käyttäytymistä, jotka on syytä tietää käsittelyn uudelleenkäsittelyä skriptoitaessa:

* `chloros-cli process`-ajo, joka pyysi tuotteita mutta ei kirjoittanut **yhtään kuvatuotetta, epäonnistuu näkyvästi ja päättyy nollasta poikkeavalla tuloksella** — et koskaan saa hiljaista tyhjää ajoa. Onnistuneet ajot raportoivat tuotemääränsä. (Tarkoituksellinen, pelkästään metatietoja käsittelevä ajo lasketaan silti onnistuneeksi.)
* Uudelleen tuodut käsitellyt vientitiedostot eivät koskaan vie kaappauksen raakapaikkaa — alkuperäinen raakatiedosto pysyy aina prosessilinjalla lähteenä.

## CLI-vastaavuudet

Kaikki tällä sivulla esitetyt toiminnot voidaan suorittaa ilman graafista käyttöliittymää. Graafisen käyttöliittymän kaappausmoodit vastaavat suoraan komentoa `chloros-cli lattice array-capture`:

| GUI | CLI |
| --- | --- |
| Yksittäinen | `chloros-cli lattice array-capture` |
| Jatkuva | `array-capture --continuous [--count N] [--duration S]` |
| Interval | `array-capture --interval S [--duration S]` |
| Fastest Capture | `array-capture --fastest` |
| Aligned-valintaruutu | `--aligned / --no-aligned` |
| Vientityyppivalintaruudut | `--processing LEVEL` tai `--levels L1,L2,…` (oletus `all`) |
| Tallennusindeksivideo | `chloros-cli lattice array-record` |
| Raakasarjan tallennus / Videon luonti | `chloros-cli lattice array-burst` / `array-build-video` |

Täydelliset lipputaulukot, älykkään automaattisen valotuksen (smart-AE) vakiintunut tallennusvaihtoehto (`--smart`) sekä vakiovauhtimalli on kuvattu kohdassa [CLI-viite § Tallennustilat, tallentimet ja offline-jälkikäsittely](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
