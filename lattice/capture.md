# Tallennusasetukset ja -tilat

Tallennus ”Kamerat”-välilehdessä tapahtuu yhden punaisen **Tallenna kaikki**-painikkeen ja yhden**Tallennusasetukset**-paneelin avulla, jossa määritetään, mitä kyseinen painike tuottaa: mitkä kamerat osallistuvat tallennukseen, mitä vientityyppejä kukin kamera tallentaa ja laukeeko suljin kerran, jatkuvasti vai tietyn aikavälin välein. Tällä sivulla kuvataan koko prosessi – konfigurointi, itse kuvaus, tiedostojen tallennuspaikka levyllä sekä se, miten ne voidaan myöhemmin käsitellä uudelleen kalibroiduiksi tuotteiksi. Itse kamera- ja ryhmäasetukset löytyvät [Kamera-asetuksista](camera-settings.md).

{% hint style="info" %}
**Kuvaus vaatii avoimen projektin.** Capture All -painike ja Capture Settings -hammasratas ovat pois käytöstä, kunnes projekti on avoinna (&quot;Luo tai avaa projekti tallentaaksesi kuvaukset&quot;). Jokainen kuvaus tallennetaan projektikansioon kohdassa `captures/`.
{% endhint %}

## Tallennusasetukset-paneeli

Avaa se sivupalkin kameraluettelossa **”Tallenna kaikki”**-painikkeen vieressä olevalla**rataspainikkeella**tai minkä tahansa kamerakohtaisen asetuspaneelin alaosassa olevalla**”Avaa tallennusasetukset…”** -painikkeella. Otsikossa lukee ”Tallennusasetukset” ja siinä on ←-paluupainike.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->

Täällä tekemäsi valinnat – mukana olevat kamerat, tyyppikohtaiset valintaruudut ja tallennustila – tallennetaan **projektikohtaisesti** ja palautetaan, kun avaat projektin uudelleen.

### Tallennustilat

Paneelin yläosassa on kolme tilapainiketta:

| Tila | Toiminto | Aliasetukset (oletusarvot) |
| --- | --- | --- |
| **Yksittäinen** *(oletus)* | Yksi tallennus kaikista valituista kameroista. | — |
| **Jatkuva**| Peräkkäisiä kuvauksia, kunnes pysäytysolosuhde täyttyy. | Pysäytys**kuvausmäärän** perusteella (oletus 1) *tai* **kuvauskeston** perusteella (oletus 10 s; yksiköt: sekunnit / minuutit / tunnit / päivät). |
| **Väli**(aikaväli) | Sarjakuvaus ajastimella. |**Kuvia / väli**(oletus 1) ·**Joka**N yksikköä (oletus 5 s) ·**Kestää** N yksikköä (oletus 1 m). |

Jatkuvassa tai aikaväli-tilassa Kaikki-painike muuttuu **Pysäytä (N)** -painikkeeksi toiminnan aikana, ja se laskee otetut kuvat niiden saapuessa.

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->

### Kameroiden ja vientityyppien valinta

Paneelin ohjeteksti tiivistää asian: valitse, mitkä kamerat ja vientityypit ”Capture All” tuottaa — oletusarvoisesti kaikki ovat päällä, ja valinnat tallennetaan tämän projektin mukana.

* **Valitse kaikki / Poista kaikki** -painikkeet kääntävät kaikkien kameroiden valintaruudut kerralla.
* **Massiiviviennin tyyppien valintakytkimet**(kaksi riviä painikkeita):**Kaikki raakakuvat / Kaikki debayeroidut / Kaikki esikatselukuvat / Kaikki säteilykuvat / Kaikki heijastuskuvat / Kaikki indeksikuvat**. Jokaisella on kolmivaiheinen värikoodi: vihreä ✓ = käytössä kaikissa sitä tukevissa kameroissa, keltainen – = käytössä joissakin, harmaa = ei missään. Valintakytkin on pois käytöstä, jos yksikään liitetty kamera ei tue kyseistä tyyppiä. Ne kaikki muuttuvat harmaiksi, kun Fastest Capture -toiminto on päällä.
* **Kamerakohtaiset rivit**: valintaruutu ”Sisällytä” sekä laajennettava (▸/▾) luettelo kyseisen kameran soveltuvista vientityypeistä, joissa on erilliset valintaruudut. Rivillä näkyy käyttöönotettujen määrä, kuten ”4/6”.

### Vientityypit ja ne tukevat kamerat

Vientityyppejä on kuusi: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Kunkin kameran rivillä näkyvät vain kyseiselle kameralle soveltuvat tyypit:

| Vientityyppi | Sisältö | RGB (FRGB) | Bayer-monispektrinen (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Bayer-mosaiikki (mono: yksi kaista) suoraan anturista | ✓ | ✓ | ✓ |
| **Debayered** | Lineaarinen demosaikki (mono: 1-kanavainen harmaasävy) | ✓ | ✓ | ✓ |
| **Esikatselu** | Täydellinen näyttöketju (valkotasapaino + gamma kameran profiilin mukaan; monispektrinen: väärävärinen venytys) | ✓ | ✓ | ✓ |
| **Säteilyvoimakkuus** | float32 W/m²/sr/nm koko radiometrisen ketjun kautta | — (ei tarjolla) | ✓ | ✓ |
| **Heijastavuus** | uint16 ρ (32768 = 1,0) | — (ei tarjolla) | ✓ — näkyy vain, kun kamerassa on DAQ-valosensori (oma tai matriisista peritty) | sama kuin monispektrinen |
| **Indeksi** | Kasvillisuusindeksi (LUT) | — | ✓ — vaatii kamerassa käytössä olevan, ei-tyhjän indeksilausekkeen, eikä sitä tarjota yhdistettyjen matriisien jäsenille (matriisi omistaa yhden jaetun indeksin) | — (indeksi vaatii vähintään 2 kaistaa; katso [Mono-kamerat ja kasvillisuusindeksit](mono-indices.md)) |

Säteilyvoimakkuutta ja heijastavuutta ei koskaan tarjota RGB-kameroille — Bayer-kohtainen säteilyvoimakkuus ei ole merkityksellinen laajakaistaiselle fotometriselle anturille.

### Nopein tallennus

**⚡ Nopein tallennus — vain raakadata**-kytkin (oranssi, kun päällä) ohittaa kaikki vientivalinnat ja asettaa ne**vain raakadataksi** — sekä tarjoaa ilmaisen yhdistettyjen indeksien yhdistelmän matriiseille — jotta kehys tallentuu mahdollisimman nopeasti: säteilyvoimakkuuden/heijastavuuden/näytön laskelmat ohitetaan kokonaan tallennushetkellä.

{% hint style="info" %}
**`.daq` tallennetaan silti.** Kun valosensori on määritetty, Nopein tallennus kirjoittaa edelleen DAQ:n alaspäin suuntautuvan lukeman raakakuvien viereen — joten säteilyvoimakkuus-, heijastavuus- ja indeksituotteet voidaan kaikki luoda myöhemmin uudelleenkäsittelyllä (katso [Tallenteiden uudelleenkäsittely](#re-processing-captures-into-calibrated-products)). Fastest Capture ei myöskään vaikuta valintaruutujen valintoihin: kun se kytketään pois päältä, valinnat palautuvat.
{% endhint %}

### Ryhmäkohtaiset säätimet

Jokaisella liitetyllä ryhmällä on oma ryhmäkorttinsa paneelissa:

* **Sisällytä-valintaruutu** (kolmiasentoinen jäsenien välillä) ja ryhmän nimi sekä sen näyttötila: &quot;(yhdistetty | erillinen)&quot;.
* **Kohdistettu**-valintaruutu (oletusarvoisesti**päällä**): muokkaa jäsenien vientiä matriisin kohdistusprofiilin mukaiseksi, jolloin vienti on pikselitasolla rekisteröity kameroiden välillä. Raakadata pysyy vääristämättömänä, mutta sisältää muunnoksen metatiedoissaan. (Profiili itsessään lasketaan [matriisin asetuspaneelissa](camera-settings.md#alignment-co-registration-combined-only).)
* Jäsenten kamerarivit on upotettu kortin sisään.

Matriisikortti sisältää myös kaksi tallenninta. Ajattele niitä **seurannan ja analyysin** vastakohtina:

| Tallennin | Luokka | Mitä se tallentaa |
| --- | --- | --- |
| **● Tallenna indeksivideo / ■ Lopeta tallennus** *(vain yhdistetyt matriisit)* | **Seuranta** | Yhdistetty indeksi-komposiitti videona 10 fps:llä — 8-bittinen, esikatseluresoluutio, LUT sisäänrakennettuna. Vaatii avoimen projektin ja suoratoistettavan reaaliaikaisen näkymän. Näyttää kehykset ja kuluneen ajan tallennuksen aikana. |
| **⦿ Raakasarjan tallennus / ■ Lopeta raakakuva-sarja** *(mikä tahansa kuvasarja)* | **Analyysi**| Raakamuotoiset Bayer-kehykset reaaliaikaisella tallennusnopeudella (ilman käsittelyä) sekä kehyskohtainen manifestitiedosto ja `.daq`-lukemat, tallennettuna tiedostoon `captures/bursts/`. Sarjakuvauksen jälkeen näkyviin tulee**Luo video** -painike: se käsittelee sarjakuvauksen offline-tilassa kalibroiduksi videoksi — yhdistetty indeksi ja/tai kamerakohtainen säteilyvoimakkuus / heijastavuus / indeksi — sekä valinnaiset TIFF-tiedostot. Yhdistetyn indeksin luominen käynnistyy automaattisesti, kun lopetat sarjakuvauksen. |##

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

Capture All -prosessi

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

Paina **Capture All** sivupalkin kameraluettelossa:

1. Jokainen mukana oleva, näkyvä ja keskeyttämätön kamera tallentaa valituilla vientityypeillä. **Ryhmät laukeavat yhtenä synkronoituna laukaisuna** (yksi synkronoitu ryhmä kaikkien jäsenten kesken — katso [Monikameraryhmät](arrays.md)); erilliset kamerat tallentavat yksitellen.
2. Piilotetut (silmä) tai keskeytetyt kamerat ohitetaan. Ryhmä on täysin estetty vasta, kun *kaikki* sen jäsenet ovat piilotettuja tai keskeytettyjä.
3. Aina kun valosensori on määritetty, vastaava DAQ-alasuuntaisen säteilyn lukema tallennetaan `.daq`-tiedostona kuvamateriaalin ohella – jopa pelkästään raakamuodossa olevien tallenteiden kohdalla – jotta radiometriset tuotteet voidaan aina johtaa myöhemmin.
4. Painike näyttää reaaliaikaisen edistymisen — ”Tallennetaan… valmis/yhteensä” — ja jatkuvassa/väli-tilassa muuttuu **Stop (N)**:ksi. Jokaisella tallennuskohteella on 300 sekunnin aikakatkaisu.
5. Kun kuvauskierros päättyy, tulosilmoituksessa näkyy **”Tallennettu N tiedostoa”**tai**”Tallennettu N, F epäonnistui”** sekä ”(S piilotettu/keskeytetty ohitettu)”, jos kameroita on ohitettu.

## Tallennusten sijainti

Tallenteet tallennetaan avoimen projektin alle nimellä `<project>/captures/`. Kukin vientityyppi tallennetaan **omaan alikansioonsa**, joten monitasoisessa tallennuksessa tyypit eivät sekoitu keskenään:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── *.daq          the downwelling reading matched to the capture
```

* `<ts>` on tallennuksen aikaleima ja `<serial>` kameran sarjanumero. Yksittäiset tallenteet nimetään muodossa `capture_<ts>_SN<serial>_<level>`; yhden synkronoidun laukaisijan tuottamat sarjatallenteet nimetään muodossa `sync_<ts>_SN<serial>_<level>`, ja **niillä on yhteinen aikaleima ryhmän kaikille kameroille** (tason jälkiliite poistetaan, kun kamera tallentaa vain yhden tason).
* **Yksi huomioitava epäsymmetria:** näyttötaso tallennetaan kansioon nimeltä `preview/`, kun taas tiedostojen nimissä säilyy `_display` — kansio ja pääte eroavat toisistaan vain kyseisen tason osalta.
* Tuntemattomat tasot tallennetaan omaa nimeään kantavaan kansioon; jos alikansiota ei voida luoda, tiedosto tallennetaan kuvauskansion juurihakemistoon sen sijaan, että se menetettäisiin.
* Capture-TIFF-tiedostot pakataan oletuksena häviöttömästi (DEFLATE), ja niiden täydelliset kalibrointi- ja käsittelymetatiedot sisältyvät **tiedoston XMP-metatietoihin** — tallenteet ovat itsestään selittäviä, eikä niillä ole muita sivutiedostoja kuin `.daq`-tiedosto.

Tämä on sama rakenne, jonka `chloros-cli lattice capture` / `array-capture` tallentavat `-o`-hakemistoonsa — dokumentoitu kohdassa [CLI-viite § Miltä tallennuskansio näyttää](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, a .daq file at the root, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Tallenteiden uudelleenkäsittely kalibroituiksi tuotteiksi

Kaapatut raakakehykset sekä tallennettu `.daq` ovat kaikki, mitä käsittelyputki tarvitsee — tämän vuoksi Fastest Capture on turvallinen käyttää todellisessa työssä.

* **GUI**: lisää tallennuskansio projektiin ([Tiedostojen lisääminen projektiin](../processing-images-gui/adding-files-to-a-project.md)) ja käsittele kuten tavallisesti.
* **CLI**: osoita `process`**kaappausten juurikansioon**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` tuo yleensä vain nimeämäsi kansion, mutta jos kyseisessä kansiossa ei ole kuvia ja siinä on alikansioita, se etenee automaattisesti alikansioihin — joten tason alikansiot ja juurikansio `.daq`:n tiedostot haetaan yhdellä kertaa. Jokainen tallennus tuodaan **yhtenä kuvana**, johon muut tasot on liitetty katselutilana, ei yhtenä kuvana per taso.

Tason alikansion nimeäminen suoraan (esim. `…/captures/raw/`) toimii myös, mutta jättää juuritason `.daq`-tiedostot pois — kopioi ne samanaikaisesti, kun johdat radiometrisen tuotteen uudelleen tiedostosta `raw/`, muuten aikaleimavastaavuudella ei ole mitään, mihin se voisi viitata.

{% hint style="warning" %}
**Käsittely alkaa aina tiedostosta `raw`.**Kussakin tallennuksessa raakakuva on prosessin lähde; `debayered`, `radiance`, `reflectance` ja `preview` ovat katseltavissa olevia tiloja, mutta niitä ei koskaan syötetä takaisin prosessiketjuun — johdetun tuotteen uudelleenkäsittely johtaisi siihen jo pikseliin upotettujen vinjetointi-, väri- ja säteilylaskelmien uudelleensoveltamiseen, joten Chloros hylätään kaksinkertaisen käsittelyn sijaan. `index/`- ja `composite/`-renderöintejä ei käsitellä lainkaan (ne ovat tulosteita, eivät tallenteita). Tallennettu captures-kansio, jossa**ei** ole raakatuontitiedostoja, näkyy normaalisti, mutta `process` ohittaa sen ja ilmoittaa siitä; `--input-level {raw,debayered,processed}` on tarkoituksellinen pakotie, joka pakottaa aloituskohdan. Katso [CLI-viite](../reference/cli-reference.md#what-a-captures-folder-looks-like) tarkkojen ohitusviestien osalta.
{% endhint %}

Kaksi muuta käyttäytymistä, jotka on syytä tietää käsiteltäessä uudelleenkäsittelyä skriptien avulla:

* `chloros-cli process`-ajo, joka pyysi tuotteita mutta ei kirjoittanut **yhtään kuvatuotetta, epäonnistuu selvästi ja päättyy nollasta poikkeavalla tuloksella** — et koskaan saa hiljaista tyhjää ajoa. Onnistuneet ajot raportoivat tuotemääränsä. (Tarkoituksellinen, pelkästään metatietoja käsittelevä ajo lasketaan silti onnistuneeksi.)
* Uudelleen tuodut käsitellyt vientitiedostot eivät koskaan vie kaappauksen raakadata-paikkaa — alkuperäinen raakadata pysyy aina prosessilinjalla lähteenä.

## CLI-vastaavuudet

Kaikki tällä sivulla esitetyt toiminnot voidaan suorittaa ilman käyttöliittymää. GUI-kaappausmoodit vastaavat suoraan `chloros-cli lattice array-capture`-komentoja:

| GUI | CLI |
| --- | --- |
| Yksittäinen | `chloros-cli lattice array-capture` |
| Jatkuva | `array-capture --continuous [--count N] [--duration S]` |
| Intervallitallennus | `array-capture --interval S [--duration S]` |
| Nopein tallennus | `array-capture --fastest` |
| Kohdistettu valintaruutu | `--aligned / --no-aligned` |
| Vientityypin valintaruudut | `--processing LEVEL` tai `--levels L1,L2,…` (oletus `all`) |
| Tallenna indeksivideo | `chloros-cli lattice array-record` |
| Tallenna raakakuva-sarja / Luo video | `chloros-cli lattice array-burst` / `array-build-video` |

Täydelliset lipputaulukot, älykkään automaattisen valotuksen (smart-AE) vakiintunut tallennusvaihtoehto (`--smart`) sekä vakiovauhtimalli löytyvät kohdasta [CLI Viite § Tallennustilat, tallentimet ja offline-jälkikäsittely](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
