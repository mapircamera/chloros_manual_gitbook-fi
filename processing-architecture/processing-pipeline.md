# Käsittelyputki

Chloros1.2.0 käyttää nelisäikeistä käsittelyputkea, joka toimii vaiheittain etenevän kokoonpanolinjan tavoin. Kukin säie hoitaa työnkulun erillisen vaiheen, joten useita kuvia voi olla samanaikaisesti käsiteltävänä eri vaiheissa.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Prosessointiputken arkkitehtuuri

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Jokainen kuva kulkee kaikkien neljän säikeen läpi järjestyksessä. Chloros+:n monisäikeisessä käsittelyssä useat kuvat ovat samanaikaisesti eri säikeissä — kun säie 3 käsittelee yhtä kuvaa, säie 1 voi tunnistaa seuraavaa, säie 2 kalibroida toista ja säie 4 tallentaa valmiin kuvan levylle.

Edistymistä raportoidaan säiettä kohden, ja tiedot välitetään Server-Sent Events -protokollan kautta (taustapalvelu julkaisee ne `/api/events`:ssä). CLI-sovelluksen reaaliaikaisessa edistymisnäytössä neljä vaihetta on nimetty **Havaitseminen, Analysointi, Käsittely, Vienti**.***

## Säikeiden tiedot

### Säie 1: Tunnistus

**Tarkoitus**: Ladata kuvat ja tunnistaa kalibrointikohteet.

* Lukee kuvatiedostoja levyltä — Survey3 `.raw`+`.jpg`-pareja, LATTICE `.tif`/`.tiff` -kuvaukset sekä `.dng`
* Puraa EXIF-metatiedot (GPS, kameramalli, aikaleimat, valotus)
* Tunnistaa kalibrointikohteet: ArUco-merkityt kohteiden geometriat LATTICE-kuvissa sekä klassisen paneelidetektorSurvey3-kalibrointikohteiden valokuvissa
* Tulokset: kuvatiedot + metatiedot + kohteiden tunnistustulokset

Pääasiassa I/O- ja CPU-riippuvainen säie.

### Säie 2: Kalibrointi

**Tarkoitus**: Laskee kalibrointiparametrit tunnistetuista kohteista.

* Laskee heijastavuuden kalibrointikertoimet kohdekuvista
* Laskee vinjetoinnin korjausparametrit
* Määrittää kaistakohtaiset kalibrointikäyrät
* Tulokset: kalibrointiparametrit jokaiselle kuvalle

CPU-riippuvainen laskentasäie. Säie 3 odottaa tätä, kun heijastavuuden kalibrointi on käytössä, jotta sen kertoimet ovat valmiina ennen kuin yhtään kuvaa käsitellään.

### Säie 3: Käsittely (GPU)

**Tarkoitus**: Soveltaa korjauksia ja laskea kasvillisuusindeksejä.**Tämä on laskennallisesti vaativin säie.*** **Debayering**: muuntaa RAW-Bayer-datan monikanavaisiksi kuviksi
  * Standard (nopea, keskilaatu) — oletusasetus, `--debayer standard`
  * Texture Aware (hidas, korkein laatu) — vain Chloros+ -versiossa, `--debayer texture-aware`, käyttää AI/ML-kohinanpoistomallia
  * LATTICE mono (M3M) -kuvat ovat yksikaistaisia: niiden kohdalla demosaic- ja valkotasapainovaiheet ohitetaan (yhden rivin lokiviestin kera), kun taas samassa ajossa olevat M3C/Bayer-kuvat käyvät ne läpi
* **Vignettikorjaus**: soveltaa objektiivin vignettikorjausta koko kuvaan
* **Heijastuskyvyn kalibrointi**: soveltaa kalibrointikertoimia heijastusarvojen muuntamiseksi
* **Indeksien laskenta**: laskee kasvillisuusindeksit (NDVI, NDRE, GNDVI, …)
* Tulokset: vientiin valmiit käsitellyt kuvatiedot

Tämä säie hyötyy eniten GPU-kiihdytyksestä, ja se on säie, jota [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) säätää.

### Säie 4: Vienti

**Tarkoitus**: Kirjoittaa käsitellyt kuvat levylle.

* Kirjoittaa tulostustiedostot valitussa muodossa — `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Upottaa metatiedot tulostustiedostoihin (GPS, aikaleimat, käsittelyparametrit)
* Järjestää tulostustiedostot projektikansioon nimellä `<camera>/<format>/<Product>_Images/` – esimerkiksi `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Viedyt tiedostot säilyttävät lähdetiedoston nimen; kansio identifioi tuotteen.**
* LATTICE-kaappausten osalta yksi lähdekuva voi haarautua useiksi tuotteiksi (Debayered, Preview, Radiance, Reflectance, Index), joista jokaisella on oma tuotekansionsa
* Tulostukset: lopulliset tiedostot levyllä

Pääasiassa I/O-rajoitteinen säie — SSD-tallennustila parantaa suorituskykyä huomattavasti.

***

## Tekniset yksityiskohdat: Suorittajat

Säikeessä 3 kuvakohtainen käsittely on rinnakkaistettu käyttämällä Python:n vakiomuotoista `concurrent.futures`-tiedostomuotoa:

* **GPU-strategiat**(`GPU_SINGLE`, `GPU_PARALLEL`) käyttävät `ProcessPoolExecutor`-menetelmää, jossa on**spawn** -aloitusmenetelmää — jokainen työntekijä on erillinen prosessi, jolla on oma CUDA-kontekstinsa (`fork` perisi vanhemman alustetun CUDA-tilan ja vahingoittaisi lapsia)
* **`CPU_PARALLEL`** käyttää `ThreadPoolExecutor`:ää — NumPy ja OpenCV vapauttavat GIL:n, joten säikeet riittävät
* Jetson-laitteet, joissa on 8 GB tai vähemmän jaettua RAM-muistia, ohittavat suorittajan kokonaan ja käsittelevät prosessin sisällä peräkkäin
* Texture Aware toimii myös peräkkäin GPU:lla, jossa on alle 7 GB VRAM-muistia — kohinanpoistomalli ei mahdu mukaan useammin kuin kerran

Chlorosei käytä mitään kolmannen osapuolen hajautettua kehystä (kuten Ray). Katso [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) selvittääksesi, miten strategia ja työntekijöiden lukumäärä valitaan.

***

## Peräkkäinen vs. putkistettu käsittely

### Vapaa tila (peräkkäinen)

Chlorosin ilmaisversiossa kuvat käsitellään **yksi kerrallaan** peräkkäin kaikkien neljän vaiheen läpi:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

GUI näyttää ilmaisversiossa yksinkertaistetun edistymispalkin; sen peräkkäiset vaiheet ilmoitetaan nimillä **Target Detection**ja sitten**Processing**.

### Chloros+ -tila (putkikäsittely)

Chloros+ -lisenssillä kaikki neljä säiettä toimivat **samanaikaisesti** eri kuvien parissa:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

GUI:n edistymispalkki näyttää neljä vaihetta; vie hiiri sen päälle nähdäksesi kunkin säikeen edistymisen. CLI-palvelussa samat neljä vaihetta näkyvät reaaliaikaisesti nimillä **Tunnistus, Analysointi, Käsittely, Vienti**.

{% hint style="info" %}
**Yksi nimike, kaksi nimeä.** CLI kutsuu vaihetta 3 nimellä _Processing_. Backendin premium-tilan etenemissyöte — se, jota käyttöliittymän etenemispalkki näyttää — nimeää saman vaiheen _Calibrating_. Kyseessä on sama säie, joka suorittaa samaa työtä (Säie 3: debayer, korjaukset, indeksit).
{% endhint %}

{% hint style="success" %}
**Chloros:n** avulla toteutettu putkikäsittely voi olla 3–5 kertaa nopeampaa kuin peräkkäinen käsittely, riippuen laitteistostasi ja aineistosi koosta. Nopeusetu on suurin järjestelmissä, joissa on nopeat GPU:t ja SSD-levyt.
{% endhint %}

***

## Säie 4: Viennin eteneminen

Viennisäiellä on oma etenemisen seuranta, jota voit tarkkailla erikseen:

**CLI:**

```bash
chloros-cli export-status
```

**SDK:**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Käsittely on valmis, kun säie 4 saavuttaa 100 %.

{% hint style="info" %}
**Suoritus, joka ei kirjoita yhtään kuvaa, on epäonnistunut.**Onnistuessaan `chloros-cli process` ilmoittaa, kuinka monta kuvatuotetta se kirjoitti (`Image products written: N`). Jos tuotteita pyydettiin, mutta**yhtään**ei kirjoitettu — vain `project.json` ja `calibration_data.json` — CLI tulostaa `Processing finished but wrote no image products.` ja**lopettaa nollasta poikkeavalla arvolla**, mainiten projektikansion nimen ja tavanomaiset syyt (syöttökansiota ei tunnistettu tallennukseksi – tarkista asettelu ja `--input-level` – tai kaikki pyydetyt tuotteet olivat soveltumattomia kyseisille kameroille). Skriptit voivat luottaa poistumiskoodiin.
{% endhint %}

***

## Suhde dynaamiseen laskentasovittamiseen

[Dynaaminen laskentasovittaminen](dynamic-compute-adaptation.md) vaikuttaa ensisijaisesti **säikeeseen 3 (käsittely)**:

* **`GPU_PARALLEL`**: Säie 3 käsittelee useita kuvia samanaikaisesti GPU:n kautta käyttäen `fused_gpu`-putkea
* **`GPU_SINGLE`**: Säie 3 sarjoittaa GPU:n käytön semaforilla, kun taas työprosessit suorittavat I/O-operaatioita päällekkäin käyttäen `fused_gpu`-putkea tai muistitehokasta `tiled_gpu`-putkea
* **`CPU_PARALLEL`**: Säie 3 käyttää CPU-pohjaista käsittelyä monisäikeisellä rinnakkaisuudella

Säikeen 3 GPU-muistin allokointi kasvaa myös, kun säikeet 1 ja 2 päättyvät — katso [Dynaaminen GPU-muistin allokointi](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Seuraavat vaiheet

* [Dynaaminen laskentasovitus](dynamic-compute-adaptation.md) — Kuinka Chloros valitsee laitteistollesi optimaalisen strategian
* [NVIDIA Jetson -opas](../linux/nvidia-jetson-guide.md) — Alustakohtainen prosessiketjun käyttäytyminen Jetsonissa
* [Käsittelyn seuranta](../processing-images-gui/monitoring-the-processing.md) — GUI-käyttöliittymän avulla tapahtuva edistymisen seuranta
* [CLI-viite](../reference/cli-reference.md) — `process`, `export-status`, poistumiskoodit ja tulosteiden asettelu
