# Käsittelyputki

Chloros 1.3.0 käyttää nelisäikeistä käsittelyputkea, joka toimii vaiheittain etenevän kokoonpanolinjan tavoin. Jokainen säie hoitaa työnkulun erillisen vaiheen, joten useita kuvia voi olla samanaikaisesti käsiteltävänä eri vaiheissa.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Prosessointiputken arkkitehtuuri

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Jokainen kuva kulkee järjestyksessä kaikkien neljän säikeen läpi. Chloros+:n monisäikeisessä käsittelyssä useat kuvat ovat samanaikaisesti eri säikeissä — kun säie 3 käsittelee yhtä kuvaa, säie 1 voi tunnistaa seuraavaa, säie 2 kalibroida toista ja säie 4 tallentaa valmiin kuvan levylle.

Edistymistä raportoidaan säiettä kohden, ja tiedot välitetään Server-Sent Events -protokollan kautta (taustapalvelu julkaisee ne osoitteessa `/api/events`). CLI:n reaaliaikaisessa edistymisnäytössä neljä vaihetta on nimetty **Tunnistus, Analysointi, Käsittely, Vienti**.

***

## Säikeiden tiedot

### Säie 1: Tunnistus

**Tarkoitus**: Ladata kuvia ja tunnistaa kalibrointikohteet.

* Lukee kuvatiedostoja levyltä — Survey3-parit `.raw`+`.jpg`, LATTICE `.tif`/`.tiff` -kuvaukset sekä `.dng`
* Puraa EXIF-metatiedot (GPS, kameramalli, aikaleimat, valotusaika)
* Tunnistaa kalibrointikohteet: ArUco-merkityt kohteiden geometriat LATTICE-kuvissa sekä klassisen paneelidetektorin Survey3-kalibrointikohteiden valokuvissa
* Tulokset: kuvatiedot + metatiedot + kohteiden tunnistustulokset

Pääasiassa I/O- ja CPU-riippuvainen säie.

### Säie 2: Kalibrointi

**Tarkoitus**: Laskee kalibrointiparametrit tunnistetuista kohteista.

* Laskee heijastavuuskalibrointikertoimet kohdekuvista
* Laskee vinjetointikorjausparametrit
* Määrittää kaistakohtaiset kalibrointikäyrät
* Tulostukset: kalibrointiparametrit jokaiselle kuvalle

CPU-riippuvainen laskentasäie. Survey3- tai sekakäytössä säie 3 odottaa tätä, kun heijastavuuskalibrointi on käytössä, jotta sen kertoimet ovat valmiina ennen kuin yhtään kuvaa käsitellään. Pelkästään LATTICE-käytössä säie 3 toimii tunnistuksen aikana: jokainen kehys käynnistyy, kun sen kohteen ankkuri on lopullinen. Kuvakehys ottaa huomioon ajallisesti lähimmän kohteen kummaltakin puolelta, joten se odottaa, kunnes kaikki sen kameran kuvakehykset, jotka ovat ajallisesti lähempänä kuin sen nykyinen lähin kohde, on skannattu. Tulokset ovat samat, mutta saatavilla aikaisemmin.

### Säie 3: Käsittely (GPU)

**Tarkoitus**: Korjausten soveltaminen ja kasvillisuusindeksien laskeminen.

**Tämä on laskennallisesti vaativin säie.**

* **Debayering**: muuntaa RAW-Bayer-tiedot monikanavaisiksi kuviksi
  * Standard (nopea, keskilaatu) — oletusasetus, `--debayer standard`
  * Texture Aware (hidas, korkein laatu) — vain Chloros+, `--debayer texture-aware`, käyttää AI/ML-kohinanpoistomallia
  * LATTICE mono (M3M) -kuvat ovat yksikaistaisia: niiden kohdalla demosaic- ja valkotasapainovaiheet ohitetaan (yhden rivin lokiviestillä), kun taas samassa ajossa olevat M3C/Bayer-kuvat käyvät ne läpi
* **Vignettikorjaus**: soveltaa objektiivin vignettikorjausta koko kuvaan
* **Heijastuskyvyn kalibrointi**: soveltaa kalibrointikertoimia heijastusarvojen muuntamiseksi
* **Indeksin laskeminen**: laskee kasvillisuusindeksit (NDVI, NDRE, GNDVI, …)
* Tulokset: vientiin valmiit käsitellyt kuvatiedot

Tämä säie hyötyy eniten GPU-kiihdytyksestä, ja se on säie, jota [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) säätää.

### Säie 4: Vienti

**Tarkoitus**: Kirjoittaa käsitellyt kuvat levylle.

* Kirjoittaa tulostustiedostot valitussa muodossa — `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Upottaa metatiedot tulostustiedostoihin (GPS, aikaleimat, käsittelyparametrit) — mukaan lukien `Chloros:*`-käsittelytunnisteet, jotka versiosta 1.3.0 lähtien kirjoitetaan jokaiseen käsittelytilaan, sekä Survey3:ssa että LATTICE:ssa
* Järjestää tulosteet projektikansion alle nimellä `<camera>/<format>/<Product>_Images/` — esimerkiksi `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Viedyt tiedostot säilyttävät lähdetiedoston nimen; kansio identifioi tuotteen.**
* LATTICE-kuvauksissa yksi lähdekuva voi tuottaa useita tuotteita (Debayered, Preview, Radiance, Reflectance, Index), joista jokaisella on oma tuotekansionsa. Indeksituotteet lasketaan ajon tuottamasta heijastavuustuotteesta (jos sellaista ei ole, jostakin muusta ajon tuottamasta tuotteesta, kuten säteilyvoimakkuudesta). Monispektristen M3C-kameroiden säteily- ja heijastavuustuotteet voivat myös saada pikselikohtaisen rajoitusmaskin tiedostossa `<format>/clamp_masks/` (katso [Lähtökuvamuodot](../output-image-formats.md#clamp-masks))
* Tulokset: lopulliset tiedostot levyllä

Pääasiassa I/O-rajoitteinen säie — SSD-tallennustila parantaa suorituskykyä huomattavasti.

***

## Tekniset yksityiskohdat: Suorittajat

Säikeessä 3 kuvakohtainen työ rinnakkaistetaan seuraavasti:

* **GPU-strategiat**(`GPU_SINGLE`, `GPU_PARALLEL`) käyttävät**työprosessien** joukkoa, joista kukin on erillinen käyttöjärjestelmäprosessi omalla CUDA-kontekstillaan. Chloros käynnistää jokaisen työprosessin käynnistämällä sen oman asennetun taustaohjelman uudelleen — ei Pythonin `multiprocessing`:n kautta, jonka työprosessien käynnistys ei toimi pakatun sovelluksen sisällä
* **`CPU_PARALLEL`** käyttää säikeiden poolia (Pythonin `ThreadPoolExecutor`) — NumPy ja OpenCV vapauttavat GIL:n, joten säikeet riittävät
* Jetson-laitteet, joissa on 8 GB tai vähemmän jaettua RAM-muistia, ohittavat suorittajan kokonaan ja käsittelevät prosessin sisällä peräkkäin
* Texture Aware toimii myös peräkkäin GPU:lla, jossa on alle 7 GB VRAM-muistia — kohinanpoistomalli ei mahdu siihen useammin kuin kerran

Chloros ei käytä mitään kolmannen osapuolen hajautettua kehystä (kuten Ray). Katso [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) selvittääksesi, miten strategia ja työntekijöiden lukumäärä valitaan.

***

## Peräkkäinen vs. putkikäsittely

### Vapaa tila (peräkkäinen)

Chlorosin ilmaisversiossa kuvat käsitellään **yksi kerrallaan** peräkkäin kaikkien neljän vaiheen läpi:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

Käyttöliittymä näyttää vapaatilassa yksinkertaistetun edistymispalkin; sen peräkkäiset vaiheet ilmoitetaan nimillä **Kohteen tunnistus**ja sitten**Käsittely**.

### Chloros+-tila (putkikäsittely)

Chloros+-lisenssillä kaikki neljä säiettä toimivat **samanaikaisesti** eri kuvien parissa:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

GUI:n edistymispalkki näyttää neljä vaihetta; vie hiiri sen päälle nähdäksesi kunkin säikeen edistymisen. CLI:ssä samat neljä vaihetta näkyvät reaaliaikaisesti nimillä **Tunnistus, Analysointi, Käsittely, Vienti**.

{% hint style="info" %}
**Yksi nimike, kaksi nimeä.** CLI-käyttöliittymä kutsuu vaihetta 3 nimellä _Processing_. Taustapalvelimen premium-tilan etenemissyöte — se, jonka GUI-etenemispalkki näyttää — nimeää saman vaiheen _Calibrating_. Ne ovat sama säie, joka suorittaa samaa työtä (Säie 3: debayer, korjaukset, indeksit).
{% endhint %}

{% hint style="success" %}
**Chloros+:n avulla toteutettu putkikäsittely** voi olla 3–5 kertaa nopeampaa kuin peräkkäinen käsittely, riippuen laitteistostasi ja aineistosi koosta. Nopeusetu on suurin järjestelmissä, joissa on nopeat GPU:t ja SSD-asemat.
{% endhint %}

***

## Säie 4: Viennin eteneminen

Viennin säiellä on oma etenemisen seuranta, jota voit tarkkailla erikseen:

**Käyttöliittymä (CLI):**

```bash
chloros-cli export-status
```

**Kehityspaketti (SDK):**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Käsittely on valmis, kun säie 4 saavuttaa 100 %.

{% hint style="info" %}
**Suoritus, jossa ei kirjoiteta kuvia, on epäonnistunut.**Onnistuessaan `chloros-cli process` ilmoittaa, kuinka monta kuvatuotetta se on kirjoittanut (`Image products written: N`). Jos tuotteita pyydettiin, mutta**yhtään**ei tallennettu — vain `project.json` ja `calibration_data.json` — CLI tulostaa `Processing finished but wrote no image products.` ja**lopettaa nollasta poikkeavalla arvolla**, mainiten projektikansion nimen ja tavanomaiset syyt (syöttökansiota ei tunnistettu tallennukseksi — tarkista asettelu ja `--input-level` — tai kaikki pyydetyt tuotteet olivat soveltumattomia kyseisille kameroille). Skriptit voivat luottaa poistumiskoodiin.
{% endhint %}

***

## Suhde dynaamiseen laskentasovittamiseen

[Dynaaminen laskentasovittaminen](dynamic-compute-adaptation.md) vaikuttaa ensisijaisesti **säikeeseen 3 (käsittely)**:

* **`GPU_PARALLEL`**: Säie 3 käsittelee useita kuvia samanaikaisesti GPU:n kautta käyttäen `fused_gpu`-putkea
* **`GPU_SINGLE`**: Säie 3 sarjoittaa GPU:n käytön semaforin avulla, kun taas työprosessit suorittavat I/O-operaatioita päällekkäin käyttäen `fused_gpu`-putkea tai muistitehokasta `tiled_gpu`-putkea
* **`CPU_PARALLEL`**: Säie 3 käyttää CPU-pohjaista käsittelyä monisäikeisellä rinnakkaisuudella

Säikeen 3 GPU-muistin varaus kasvaa myös, kun säikeet 1 ja 2 päättyvät — katso [Dynaaminen GPU-muistin varaus](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Seuraavat vaiheet

* [Dynaaminen laskentasovitus](dynamic-compute-adaptation.md) — Kuinka Chloros valitsee laitteistollesi optimaalisen strategian
* [NVIDIA Jetson -opas](../linux/nvidia-jetson-guide.md) — Alustakohtainen prosessiketjun käyttäytyminen Jetsonissa
* [Käsittelyn seuranta](../processing-images-gui/monitoring-the-processing.md) — GUI-käyttöliittymän etenemisen seuranta
* [Käyttöliittymän (CLI) viite](../reference/cli-reference.md) — `process`, `export-status`, poistumiskoodit ja tulosteiden rakenne
