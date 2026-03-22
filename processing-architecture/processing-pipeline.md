# Käsittelyputki

Chloros 1.1.0 käyttää nelisäikeistä käsittelyputkea, joka toimii vaiheittain etenevänä kokoonpanolinjana. Kukin säie hoitaa käsittelytyönkulun erillisen vaiheen, minkä ansiosta useita kuvia voidaan käsitellä samanaikaisesti eri vaiheissa.

***

## Putken arkkitehtuuri

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Jokainen kuva kulkee kaikkien neljän säikeen läpi järjestyksessä. Chloros+:n monisäikeisen käsittelyn ansiosta useita kuvia voi olla eri säikeissä samanaikaisesti — kun säie 3 käsittelee yhtä kuvaa, säie 1 voi tunnistaa seuraavaa, säie 2 voi kalibroida toista ja säie 4 voi kirjoittaa aiemmin käsitellyn kuvan levylle.

***

## Säikeiden tiedot

### Säie 1: Tunnistus

**Tarkoitus**: Lataa kuvat ja tunnistaa kalibrointikohteet.

* Lukee kuvatiedostot levyltä (RAW, JPG)
* Puraa EXIF-metatiedot (GPS, kameramalli, aikaleimat, valotus)
* Tunnistaa ArUco-kalibrointikohteet merkityissä kohdekuvissa
* Tulostukset: kuvatiedot + metatiedot + kohteen tunnistustulokset

Tämä on pääasiassa I/O- ja CPU-riippuvainen säie.

### Säie 2: Kalibrointi

**Tarkoitus**: Laskea kalibrointiparametrit tunnistetuista kohteista.

* Laskee heijastavuuden kalibrointikertoimet kohdekuvista
* Laskee vinjetointikorjausparametrit
* Määrittää kaistakohtaiset kalibrointikäyrät
* Tulostukset: kalibrointiparametrit jokaiselle kuvalle

Tämä on CPU-intensiivinen laskentasäie.

### Säie 3: Käsittely (GPU)

**Tarkoitus**: Soveltaa korjauksia ja laskea kasvillisuusindeksejä.**Tämä on laskentaintensiivisin säie.*** **Debayering**: Muuntaa RAW-Bayer-kuviotiedot monikanavaisiksi kuviksi
  * Standard (Nopea, Keskilaatu) — oletus
  * Texture Aware (Hidas, Korkein laatu) — vain Chloros+, käyttää AI/ML-kohinanpoistoa
* **Vignettikorjaus**: Soveltaa objektiivin vignettikorjausta koko kuvaan
* **Heijastavuuden kalibrointi**: Soveltaa kalibrointikertoimia heijastavuusarvojen muuntamiseksi
* **Indeksin laskeminen**: Laskee kasvillisuusindeksit (NDVI, NDRE, GNDVI jne.)
* Tulokset: vientiin valmiit käsitellyt kuvatiedot

Tämä säie hyötyy eniten GPU-kiihdytyksestä. [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) -järjestelmä optimoi ensisijaisesti tämän säikeen toimintaa.

### Säie 4: Vienti

**Tarkoitus**: Kirjoittaa käsitellyt kuvat levylle.

* Kirjoittaa tulostustiedostot valitussa muodossa (TIFF 16-bittinen, TIFF 32-bittinen %, PNG, JPG)
* Upottaa EXIF-metatiedot tulostustiedostoihin (GPS, aikaleimat, käsittelyparametrit)
* Järjestää tulostuksen kameramallikohtaisiin alikansioihin
* Tulostukset: lopulliset tiedostot levylle

Tämä on ensisijaisesti I/O-riippuvainen säie. SSD-tallennustila parantaa merkittävästi säikeen 4 suorituskykyä.

***

## Peräkkäinen vs. putkikäsittely

### Vapaa tila (peräkkäinen)

Chloros:n ilmaisversiossa kuvat käsitellään **yksi kerrallaan**, peräkkäin kaikkien neljän vaiheen läpi:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

GUI:n edistymispalkki näyttää 2 vaihetta: Kohteen tunnistus ja Käsittely.

### Chloros+ -tila (putkikäsittely)

Chloros+ -lisenssillä kaikki neljä säiettä toimivat **samanaikaisesti** eri kuvien parissa:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

GUI-etenemispalkki näyttää 4 vaihetta: Tunnistus, Analysointi, Kalibrointi, Vienti. Vie hiiri etenemispalkin päälle nähdäksesi säikeittäisen etenemisen.

{% hint style="success" %}
**Pipelinetyyppinen käsittely Chloros+:lla** voi olla 3–5 kertaa nopeampaa kuin peräkkäinen käsittely, riippuen laitteistostasi ja datajoukon koosta. Nopeusetu on suurin järjestelmissä, joissa on nopeat GPU:t ja SSD-asemat.
{% endhint %}

***

## Säikeen 4 vientien eteneminen

Chloros 1.1.0:ssa vientisäikeellä (säie 4) on oma erillinen etenemisen seuranta. Voit seurata viennin etenemistä erikseen:

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

***

## Suhde dynaamiseen laskentasovittamiseen

[Dynaaminen laskentasovitus](dynamic-compute-adaptation.md) -järjestelmä vaikuttaa ensisijaisesti **säikeeseen 3 (käsittely)**:

* **`GPU_PARALLEL`** -strategia: Säie 3 käsittelee useita kuvia samanaikaisesti GPU:n kautta käyttäen `fused_gpu`-putkea
* **`GPU_SINGLE`**-strategia: Säie 3 käsittelee yhden kuvan kerrallaan muistitehokasta `tiled_gpu`-putkea käyttäen
* **`CPU_PARALLEL`**-strategia: Säie 3 käyttää CPU-pohjaista käsittelyä monisäikeisellä rinnakkaisuudella

Säikeen 3 GPU-muistin allokointi muuttuu myös dynaamisesti, kun säikeet 1 ja 2 valmistuvat — katso [Dynaaminen GPU-muistin allokointi](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Seuraavat vaiheet

* [Dynaaminen laskentatehon mukautus](dynamic-compute-adaptation.md) — Kuinka Chloros valitsee laitteistollesi optimaalisen strategian
* [NVIDIA Jetson -opas](../linux/nvidia-jetson-guide.md) — Alustakohtainen putkistokäyttäytyminen Jetsonissa
* [Käsittelyn seuranta](../processing-images-gui/monitoring-the-processing.md) — GUI-etenemisen seuranta
