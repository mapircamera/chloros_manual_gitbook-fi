# NVIDIA Jetson -opas

Chloros NVIDIA Jetsonilla mahdollistaa monispektrisen kuvankäsittelyn reuna-laitteissa — kentällä, UAV-laitteissa ja etäasennuksissa. Chloros 1.2.0 tunnistaa Jetson-mallisi käynnistyksen yhteydessä ja optimoi käsittelystrategiansa löydetyn laitteiston mukaan. **Manuaalista säätämistä ei tarvita.**

***

## Tuetut Jetson-mallit

| Malli                | RAM            | Käsittelystrategia                                     | Suositeltu käyttö                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 Gt jaettua | `GPU_PARALLEL` (2 työntekijää)                              | Maksimi suorituskyky, suuret tietojoukot                      |
| **Jetson Orin NX**   | 8–16 Gt jaettua | `GPU_PARALLEL` (2 työprosessia, 16 Gt) / `GPU_SINGLE` (8 GB)   | Ensisijainen suositus ilma- ja kenttäkäyttöön |
| **Jetson Orin Nano** | 8 GB jaettua     | `GPU_SINGLE` (1 työntekijä, peräkkäinen)                     | Perustason reunalaskenta                                 |

{% hint style="info" %}
Linux arm64 -paketti vaatii **JetPack 6**:n, joka on saatavilla Jetson Orin -tuoteperheelle. Vanhemmat mallit (Nano, TX2, Xavier NX) eivät tue JetPack 6:ta, eikä nykyinen paketti tue niitä.
{% endhint %}

***

## Vaatimukset

* **JetPack 6.x** (suositellaan uusinta versiota)
* **NVIDIA CUDA** (sisältyy JetPackiin)
* **Maksullinen Chloros+-paketti** — Copper-taso tai korkeampi (vaaditaan kaikkeen CLI/SDK-käyttöön; valvotaan palvelinpuolella)

## Asennus

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.2.0"

# Install Python SDK (optional) — the bundled wheel always matches this build
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl

# Run system diagnostics
chloros-cli selftest
```

Yleisiä Linux-asennustietoja, tiedostojen sijainteja ja vianmääritysohjeita on kohdassa [Linux-asennus](linux-installation.md).

{% hint style="info" %}
**Sijoita purkukansio nopeaan tallennuspaikkaan.** Käännetyt binäärit purkautuvat itsestään väliaikaiseen kansioon jokaisen käynnistyksen yhteydessä — mikä on erittäin hidasta SD-kortilta. Chloros käyttää automaattisesti `/mnt/ssd/tmp`-tiedostoa, jos se on olemassa; muussa tapauksessa aseta `TMPDIR`-asetukseksi polku NVMe-asemallasi (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Dynaaminen laskentatehon sopeutus Jetsonissa

### Miten se toimii

Käynnistyksen yhteydessä Chloros luo profiilin järjestelmästäsi:

1. **Tunnistaa Jetson-mallin** `/proc/device-tree/model`:n avulla
2. **Lukee käytettävissä olevan jaetun GPU-/CPU-muistin** (Jetson käyttää yhtenäistettyä muistia)
3. **Valitsee käsittelystrategian** (`GPU_PARALLEL`, `GPU_SINGLE` tai `CPU_PARALLEL`)
4. **Asettaa työntekijöiden lukumäärän, putkityypin ja muistin allokoinnin** automaattisesti

Päätös perustuu **jaetun RAM-muistin kokonaismäärään**, ei mallin nimeen:

* **Alle 12 GB kokonaisRAM-muistia**(kaikki 8 GB:n Jetson-laitteet): `GPU_SINGLE`, jossa on**1 työntekijä — tarkoituksellinen peräkkäinen käsittely**. Muistia on liian vähän samanaikaisille työntekijöille, joten kuvat käsitellään yksi kerrallaan. Jetson-laitteissa, joissa on**8 GB tai vähemmän**, Thread 3 ohittaa työprosessipoolin kokonaan ja suorittaa kuvakohtaisen työnsä prosessin sisällä.
* **12 Gt tai enemmän**(Orin NX 16 Gt, AGX Orin): yhtenäinen muisti täyttää `GPU_PARALLEL`:n vaatimukset, mutta työprosessien lukumäärä on**rajoitettu kahteen Jetsonissa** — GPU, työprosessien RAM-muisti ja kunkin työprosessin CUDA-kontekstit käyttävät kaikki samaa jaettua muistipoolia, joten useammat työprosessit lisäävät muistin loppumisen riskiä.

Voit ohittaa automaattisen valinnan `CHLOROS_STRATEGY`-ympäristömuuttujalla — katso [Dynaaminen laskentakapasiteetin sopeutus](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Mallikohtainen käyttäytyminen

| Jetson-malli                | Strategia       | Työntekijät | Suoritus                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8GB**    | `GPU_SINGLE`   | 1       | Peräkkäinen prosessin sisäinen silmukka (`tiled_gpu` muistipaineen alaisena) |
| **Jetson Orin NX 8GB**      | `GPU_SINGLE`   | 1       | Peräkkäinen prosessin sisäinen silmukka                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Samanaikaiset työprosessit, `fused_gpu`-polku  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2       | Samanaikaiset työprosessit, `fused_gpu`-polku  |

Alustojen keskeisin ero on **muisti**. 8 GB:n Jetson joutuu käsittelemään kuvia yksi kerrallaan muistitehokkaalla tiled-menetelmällä, kun kuormitus on suuri, kun taas 16 GB:n tai suurempi Orin voi käsitellä kaksi kuvaa samanaikaisesti GPU:n kautta käyttämällä suuremman läpimenokapasiteetin omaavaa fused-putkea.

### GPU-resurssit mallikohtaisesti

Jokaisella Jetson-mallilla on myös laitteistoprofiili, joka rajoittaa, kuinka suuren osan jaetusta muistipoolista prosessointi voi käyttää, sekä skaalaa eräkokoja:

| Malli | GPU-resurssien yläraja | Eräkoko-kerroin | Varattu järjestelmälle/näytölle |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70 % | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75 % | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80 % | ×1,5 | 4,0 GB |

Havaittu RAM-muistin määrä säätää profiilia: jos Jetson ilmoittaa **16 GB tai enemmän**, sen erän kerroin nostetaan arvoon ×1,2. Peruserän koko ennen kertoimia on 8 kuvaa.

Katso täydellinen laskentakapasiteetin mukautusohje [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md).

***

## GPU:n taajuusrajoitus Texture Aware -toiminnolle Nano- ja Orin Nano -malleissa

Texture Aware -debayer suorittaa GPU:n neuroverkkojen päättelyä, mikä voi laukaista **ylivirta-varoituksia**matalatehoisissa Jetson-malleissa (10–15 W:n luokka) GPU:n käydessään täydellä kellotaajuudella. Ennen Texture Aware -käsittelyn aloittamista**Jetson Nano- tai Orin Nano -laitteella**Chloros tarkistaa GPU:n maksimitaajuuden ja rajoittaa sen arvoon**510 MHz** (510000000), jos se on tällä hetkellä suurempi:

* Jos CLI pystyy kirjoittamaan GPU:n taajuuden sysfs-solmuun, rajoitus **otetaan käyttöön automaattisesti** ja näyttöön tulostetaan vahvistus.
* Jos ei (vaatii root-oikeudet), CLI tulostaa tarkan `sudo`-komennon rajoituksen asettamiseksi manuaalisesti, odottaa hetken, jotta ehdit lukea sen, ja jatkaa sitten — käsittely jatkuu, mutta saattaa näyttää ylivirta-varoituksia.

Jos haluat asettaa rajoituksen itse ennen käsittelyä:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Suuremman tehon mallit (Orin NX 25W, AGX Orin 60W) toimivat täydellä GPU-nopeudella; rajoitusta ei sovelleta. Standard-debayer ei koskaan laukaise rajoitusta missään mallissa.

{% hint style="info" %}
**Texture Aware -tila Jetsonissa käsittelee aina yhden kuvan kerrallaan.** Jokainen työntekijä tarvitsisi oman CUDA-kontekstinsa (~1 GB) sekä oman kopionsa kohinanpoistomallista, mihin yhtenäinen muisti ei riitä — joten Jetsonissa Texture Aware -polku on kiinnitetty yhteen työntekijään, jonka GPU-käyttö on sarjoitettu. Texture Aware on odotettavasti huomattavasti hitaampi kuin Standard-menetelmä kaikissa Jetson-laitteissa.
{% endhint %}

***

## Lämmönhallinta

Jetson-laitteilla on rajallinen lämpövara, etenkin suljetuissa tai lentokäyttöön tarkoitetuissa asennuksissa. Chloros valvoo SoC:n lämpötilaa ja rajoittaa eräkokoja automaattisesti:

| Lämpötila         | Toimenpide                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normaali toiminta — täysi prosessointinopeus          |
| **70 °C** (Varoitus)  | Erän koko pienenee asteittain (100 % → 50 % välillä 70 °C – 80 °C) |
| **80 °C** (Kriittinen) | Voimakas rajoitus (50 % → 0 % välillä 80 °C – 90 °C) |
| **90 °C** (Sammutus) | GPU-käsittely pysäytetään kokonaan — jäähdytys vaaditaan |

{% hint style="warning" %}
**Varmista riittävä ilmanvaihto ja lämmönpoisto** jatkuvaa käsittelyä varten, erityisesti suljetuissa kenttäkoteloissa tai lentokoneissa. Lämpösäätö vähentää käsittelyn suorituskykyä laitteiston suojaamiseksi.
{% endhint %}

***

## Muistin hallinta

Jetson-laitteet käyttävät **yhdistettyä muistia** — GPU ja CPU jakavat saman fyysisen RAM-muistin. Ilmoitettu VRAM (esim. ~15,3 GB Orin NX 16 GB:ssa) ei ole erillistä GPU-muistia; se on samaa RAM-muistia, jota käyttöjärjestelmä ja kaikki muut prosessit käyttävät.

### Swap-varoitus ja suositukset

Ennen käsittelyn aloittamista Jetsonissa CLI laskee syöttökansiossasi olevien RAW-kuvien määrän (`.tif`, `.tiff`, `.raw`, `.dng` — JPG-esikatselukuvia ei lasketa mukaan), arvioi suorituksen tarvitseman muistin huippumäärän ja **varoittaa ennen aloittamista**, jos RAM-muistin ja swap-muistin yhteismäärä ei todennäköisesti riitä. Varoituksen otsikko on `LOW MEMORY WARNING - Jetson Detected`, ja siinä ilmoitetaan kuvien lukumäärä, RAM-muistin määrän, nykyisen swap-tilan ja arvioidun huipputarpeen, minkä jälkeen se antaa tarkat `fallocate` / `chmod` / `mkswap` / `swapon` -komennot, jotka on mitoitettu projektisi mukaan (ei koskaan alle 8 GB). Ohjelma keskeyttää toiminnan muutamaksi sekunniksi, jotta viesti ei häviä vierityshistoriassa, minkä jälkeen käsittely jatkuu.**Varoituksessa käytetyt muistiarviot:**

| Debayer-tila | Perus | Kuva kohti |
| --- | --- | --- |
| Vakio | ~1,5 GB | ~10 MB |
| Tekstuuritunnistava | ~2,5 GB (malli + Python-ajonaikainen muisti) | ~15 MB |

Varoitus laukeaa, kun arvioitu huippuarvo ylittää RAM-muistin ja swap-tilan yhteismäärän vähennettynä 1 GB:n varmuusmarginaalilla, ja se laskee mukaan vain **tiedostopohjaisen** swap-tilan — pelkästään zramia käyttävä kokoonpano merkitään silti.

Vaihtomuistin lisääminen manuaalisesti (esimerkki: 8 GB):



<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->

```bash
# Check current memory and swap
free -h

# Create a swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```### OOM (Out of Memory) -käsittely

Käsittelyn aikana Chloros valvoo GPU:n muistia ja hidastaa toimintaa sujuvasti kaatumisen sijaan:

1. Kun GPU:n muistin käyttöaste ylittää **85 %**, eräkokoja pienennetään ennakoivasti
2. Jos muistin loppumistilanne silti tapahtuu, eräkoko **puolitetaan** ja puolitetaan uudelleen jokaisen peräkkäisen OOM-tilanteen yhteydessä; jokainen seuraava onnistunut erä siirtää tämän rangaistuksen yhden askeleen taaksepäin
3. Jatkuvan kuormituksen alla prosessiputki siirtyy `fused_gpu`-tilasta muistitehokkaaseen `tiled_gpu`-polkuun ja viimeisenä keinona CPU-käsittelyyn

***

## Käyttöönotto kenttäolosuhteissa

### Virrankulutusta koskevat huomioitavat seikat

| Jetson-malli     | Tyypillinen virrankulutus | Huomautukset                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7–15 W              | DC-tynnyriliitin          |
| Jetson Orin NX   | 10–25 W             | DC-tynnyriliitin          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD tai tynnyriliitin |

Suunnittele virrankulutus jatkuvaa prosessointia varten — suurin virrankulutus tapahtuu GPU:ta paljon kuormittavassa vaiheessa 3 (käsittely).

### Tallennustilaa koskevat suositukset

* **NVMe SSD** suositellaan vahvasti arm64-käyttöönottoihin
* SD-kortit ovat liian hitaita prosessointiin — käytä niitä vain käynnistysvälineenä
* Varaa 2–3-kertainen määrä raakakuvadatan kokoa prosessoidulle tulokselle

### Päätön käyttö SSH:n kautta

Chloros ja CLI sopivat erinomaisesti päätömiin Jetson-käyttöönottoihin:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Jatkuvasti päällä oleva taustapalvelu LATTICE- ja DAQ-E-aikasyntronisointia varten

Jos Jetson ohjaa LATTICE-kameroita tai DAQ-E-valosensoreita ilman näyttöä, ota backend-systemd-palvelu käyttöön, jotta PTP-grandmaster toimii jatkuvasti (yksikkö on asennettu, mutta se ei ole oletusarvoisesti käytössä):

```bash
sudo systemctl enable --now chloros-backend.service
chloros-cli time-sync status
```

Katso [Linux-asennusohjeet](linux-installation.md#always-on-ptp-for-headless-hosts) saadaksesi lisätietoja, mukaan lukien siitä, miten paketti mahdollistaa PTP-porttien 319/320 sitomisen ilman root-oikeuksia.

### Automaattinen käsittely systemd:n avulla

Luo systemd-palvelu automatisoitua käsittelyä varten:

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

`chloros-cli process` palauttaa nollasta poikkeavan arvon, jos tuotteita pyytänyt ajo ei kirjoita kuvia, joten systemd:n virhetila on merkityksellinen seurannan kannalta.

Yhdistä systemd-ajastimeen ajoitettua käsittelyä varten:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## Esimerkkityönkulut

### Jetsonin peruskäsittely

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI
```

### Python ja SDK Jetsonissa

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Useiden lentojen eräprosessointi

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Suositellut Jetson-järjestelmät kenttäkäyttöön

Kenttä- ja ilmakäyttöön kannattaa harkita seuraavia Jetson Orin NX 16 GB -kantolevyvaihtoehtoja:

* **Ilmassa/drooni**: Järjestelmät, joilla on tärinänkestävyysluokitus (MIL-STD), kevyt rakenne (alle 300 g) ja passiivinen jäähdytys
* **Kestävä kenttäkäyttö**: IP67/IP69K-luokan vedenpitävät kotelot, joissa on PoE GigE -kameraliitäntä
* **Minimi/edullinen**: Kehittäjäsarjat, joihin voi liittää lisäkoteloita

Ota yhteyttä [MAPIR-tukeen](https://www.mapir.camera/community/contact), jos tarvitset laitteistosuosituksia juuri sinun käyttötarkoitukseesi.

***

## Seuraavat vaiheet

* [Linux-asennus](linux-installation.md) — Yleisiä Linux-asennustietoja
* [Dynaaminen laskentakapasiteetin mukautus](../processing-architecture/dynamic-compute-adaptation.md) — Täydellinen laskentastrategiaopas
* [Käsittelyputki](../processing-architecture/processing-pipeline.md) — 4-säikeisen käsittelyputken ymmärtäminen
* [CLI : Komentorivi](../CLI.md) — CLI-opas
* [API : Python SDK](../api-python-sdk.md) — SDK-opas
* [CLI-viite](../reference/cli-reference.md) ja [SDK-viite](../reference/sdk-reference.md) — Kattavat komento-/API-luettelot versioon 1.2.0
