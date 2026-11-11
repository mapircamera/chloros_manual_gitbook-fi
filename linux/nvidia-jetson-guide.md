# NVIDIA Jetson -opas NVIDIA Jetsonilla toimiva

Chloros mahdollistaa monispektrisen kuvankäsittelyn reuna-laitteissa — kentällä, lennokeissa ja etäasennuksissa. Chloros 1.3.0 tunnistaa Jetson-mallisi käynnistyksen yhteydessä ja optimoi käsittelystrategiansa löydetyn laitteiston mukaan. **Manuaalista säätämistä ei tarvita.**
{% hint style="info" %}
**Jetson-paketti 1.3.0 suorittaa käsittelyn Jetsonin GPU:lla.** `chloros_1.3.0_arm64_jp6.deb` on kehitetty JetPack 6:lle ja käyttää JetPack 6:n CUDA-rakennetta PyTorchista, joten käsittely, mukaan lukien Texture Aware -debayer, suoritetaan Orinin GPU:lla. Paketti ei sisällä itse CUDAa: se käyttää JetPack 6:n asentamia CUDA- ja cuDNN-kirjastoja (`sudo apt install nvidia-jetpack` kuvassa, josta ne puuttuvat). Jos CUDA-grafiikkaprosessoria ei ole käytettävissä, käsittely tapahtuu CPU:lla ja Texture Aware käyttää Standard-debayeria.
{% endhint %}

***

## Tuetut Jetson-mallit

| Malli                | RAM            | Käsittelystrategia                                     | Suositeltu käyttö                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 Gt jaettua | `GPU_PARALLEL` (2 työntekijää)                              | Suurin suorituskyky, suuret tietojoukot                      |
| **Jetson Orin NX**   | 8–16 GB jaettua | `GPU_PARALLEL` (2 työprosessia, 16 GB) / `GPU_SINGLE` (8 GB)   | Ensisijainen suositus kenttäkäyttöön              |
| **Jetson Orin Nano** | 8 Gt jaettua     | `GPU_SINGLE` (1 työntekijä, peräkkäinen)                     | Perustason reunalaskenta                                 |

{% hint style="info" %}
Linux-arm64-paketti vaatii **JetPack 6**:n, joka on saatavilla Jetson Orin -tuoteperheessä. Vanhemmat mallit (Nano, TX2, Xavier NX) eivät tue JetPack 6:ta, eikä nykyinen paketti tue niitä.
{% endhint %}

***

## Vaatimukset

* **JetPack 6.x** (suositellaan uusinta versiota)
* **NVIDIA CUDA ja cuDNN** JetPack 6:sta (`sudo apt install nvidia-jetpack`, jos ne puuttuvat kuvastasi)
* **Maksullinen Chloros+ -sopimus** — Copper-taso tai korkeampi (vaaditaan kaikkeen CLI- ja SDK-käyttöön; valvotaan palvelinpuolella)

## Asennus

```bash
# Install the arm64 .deb package (JetPack 6) -- apt, not dpkg -i, so the
# Python SDK's requirements (python3-numpy, python3-requests) come too
sudo apt install ./chloros_1.3.0_arm64_jp6.deb

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.3.0"
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # the .deb installs the SDK

# Run system diagnostics
chloros-cli selftest
```

`.deb` asentaa tämän rakennuksen kanssa yhteensopivan Python-SDK:n järjestelmään `python3`. Vain virtualenv-ympäristöön on asennettava mukana toimitettu wheel-tiedosto: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` (täysin offline-tilassa). Yleisiä Linux-asennuksen yksityiskohtia, tiedostojen sijainteja ja vianmääritystä koskevat ohjeet löytyvät kohdasta [Linux-asennus](linux-installation.md).

{% hint style="info" %}
**Ei enää purkamista käynnistyksen yhteydessä.** Versiosta 1.3.0 lähtien taustapalvelu ja CLI-sovellus asennetaan purkamattomina kansioon `/usr/lib/chloros/`, joten ne eivät enää pura itseään väliaikaiseen kansioon joka kerta käynnistyessään. Väliaikaiset tiedostot tallennetaan edelleen automaattisesti kansioon `/mnt/ssd/tmp`, jos kyseinen kansio on olemassa; muussa tapauksessa voit määrittää `TMPDIR`:n poluksi NVMe-asemallasi (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Dynaaminen laskentakapasiteetin mukautus Jetsonissa

### Miten se toimii

Käynnistyksen yhteydessä Chloros luo järjestelmästäsi profiilin:

1. **Tunnistaa Jetson-mallin** `/proc/device-tree/model`:n kautta
2. **Lukee käytettävissä olevan jaetun GPU-/CPU-muistin** (Jetson käyttää yhtenäistettyä muistia)
3. **Kysyy PyTorchilta, voiko se käyttää CUDA:ta** — kun JetPack 6:n CUDA ja cuDNN on asennettu, se voi
4. **Valitsee käsittelystrategian** (`GPU_PARALLEL`, `GPU_SINGLE` tai `CPU_PARALLEL`) ja työntekijöiden lukumäärän automaattisesti

Päätös perustuu **jaetun RAM-muistin kokonaismäärään**, ei mallin nimeen:

* **Alle 12 GB RAM-muistia yhteensä**(kaikki 8 GB:n Jetson-laitteet):**1 työntekijä — tarkoituksellinen peräkkäinen käsittely**. Muistia ei riitä samanaikaisille työntekijöille, joten kuvat käsitellään yksi kerrallaan. Jetson-laitteissa, joissa on**8 Gt tai vähemmän**, Thread 3 ohittaa työprosessipoolin kokonaan ja suorittaa kuvakohtaisen työnsä prosessin sisällä. Taustapalvelun lokissa tätä strategiaa kutsutaan nimellä `GPU_SINGLE`.
* **12 GB tai enemmän**(Orin NX 16 GB, AGX Orin): yhtenäinen muisti täyttää `GPU_PARALLEL`-strategian vaatimukset, mutta työprosessien lukumäärä on**rajoitettu kahteen Jetson-laitteissa** — GPU, työprosessien RAM-muisti ja kunkin työprosessin CUDA-kontekstit käyttävät kaikki samaa jaettua muistipoolia, joten useammat työprosessit lisäävät muistin loppumisen riskiä.
* **Ei käyttökelpoista CUDA:ta** (JetPackin CUDA-kirjastot puuttuvat): `CPU_PARALLEL` 12 GB:n tai suuremmalla muistimäärällä, alla yksi kuva kerrallaan.

Voit ohittaa automaattisen valinnan `CHLOROS_STRATEGY`-ympäristömuuttujalla — katso [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Mallikohtainen käyttäytyminen

| Jetson-malli                | Strategia       | Työntekijät | Suoritus                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Peräkkäinen prosessin sisäinen silmukka (`tiled_gpu` muistipaineen alla) |
| **Jetson Orin NX 8GB**      | `GPU_SINGLE`   | 1       | Peräkkäinen prosessin sisäinen silmukka                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Rinnakkaiset työprosessit, `fused_gpu`-polku  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2       | Samanaikaiset työprosessit, `fused_gpu`-polku  |

Alustojen keskeisin ero on **muisti**. 8 GB:n Jetson joutuu kuormituksen ollessa suurta käsittelemään kuvia yksi kerrallaan muistitehokkaalla tiloitetulla menetelmällä, kun taas 16 GB:n tai suurempi Orin pystyy käsittelemään GPU:n kautta kahta kuvaa samanaikaisesti käyttämällä suuremman läpimenokapasiteetin omaavaa yhdistettyä prosessointiputkea.

### Mallikohtainen GPU-budjetti

Jokaisella Jetson-mallilla on myös laitteistoprofiili, joka rajoittaa, kuinka suuren osan jaetusta muistipoolista prosessointi voi varata, ja skaalaa eräkokoja:

| Malli | GPU-budjetin yläraja | Eräkoko-kerroin | Varattu järjestelmälle/näytölle |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70 % | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75 % | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80 % | ×1,5 | 4,0 GB |

Havaittu RAM-muistin määrä säätää profiilia: jos Jetson ilmoittaa **16 GB tai enemmän**, sen erän kerroin nostetaan arvoon ×1,2. Peruserän koko ennen kertoimia on 8 kuvaa.

Katso täydellinen laskentatehon mukautusohje [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) -sivulta.

***

## GPU:n taajuusrajoitus Texture Aware -toiminnolle Nano- ja Orin Nano -malleissa

Texture Aware -debayer suorittaa GPU:n neuroverkkojen päättelyä, mikä voi laukaista **ylivirta-varoituksia**matalatehoisissa Jetson-malleissa (10–15 W:n luokka) GPU:n täydellä kellotaajuudella. Kun välität `--debayer texture-aware`:n**Jetson Nano- tai Orin Nano -laitteessa**, CLI tarkistaa GPU:n enimmäistaajuuden ja rajoittaa sen arvoon **510 MHz** (510000000), jos se on tällä hetkellä korkeampi:

* Jos CLI:n on mahdollista kirjoittaa GPU:n taajuuden sysfs-solmuun, rajoitus **otetaan käyttöön automaattisesti** ja näyttöön tulostetaan vahvistus.
* Jos ei (vaatii root-oikeudet), CLI tulostaa tarkan `sudo`-komennon rajoituksen asettamiseksi manuaalisesti, odottaa hetken, jotta voit lukea sen, ja jatkaa sitten — käsittely jatkuu, mutta saattaa näyttää ylivirta-varoituksia.

Voit asettaa rajoituksen itse ennen käsittelyä seuraavasti:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Suuremman tehon mallit (Orin NX 25W, AGX Orin 60W) toimivat täydellä GPU-nopeudella; rajoitusta ei sovelleta. Standard-debayer ei koskaan laukaise rajoitusta missään mallissa.

{% hint style="info" %}
**Texture Aware -tila Jetsonissa käsittelee aina yhden kuvan kerrallaan.** Jokainen työntekijä tarvitsisi oman CUDA-kontekstin (~1 GB) sekä oman kopionsa kohinanpoistomallista, mihin yhtenäinen muisti ei riitä — joten Jetsonissa Texture Aware -polku on kiinnitetty yhteen työntekijään, jonka GPU-käyttö on sarjoitettu. Texture Aware on odotettavasti huomattavasti hitaampi kuin Standard kaikilla Jetson-laitteilla. Texture Aware -tila koskee Survey3-kuvia: LATTICE-kaappaukset käsitellään aina Standard-debayerin avulla. Ilman käyttökelpoista CUDA-GPU:ta suoritus käyttää Standard-tilaa, ja taustaprosessi kirjaa yhden `[DEBAYER] … needs a CUDA GPU …`-rivin, jossa selitetään syy (esitetty kuvassa `chloros-cli --verbose`).
{% endhint %}

***

## Lämmönhallinta

Jetson-laitteiden lämmönvaranto on rajallinen, etenkin suljetuissa tiloissa tai ilmakäytössä. Chloros valvoo SoC:n lämpötilaa ja rajoittaa eräkokoja automaattisesti:

| Lämpötila         | Toimenpide                                            |
| ------------------- | ------------------------------------------------- |
| **< 70 °C**          | Normaali toiminta — täysi prosessointinopeus          |
| **70 °C** (Varoitus)  | Erän koko pienenee asteittain (100 % → 50 % välillä 70 °C – 80 °C) |
| **80 °C** (Kriittinen) | Voimakas rajoitus (50 % → 0 % välillä 80 °C – 90 °C) |
| **90 °C** (Sammutus) | GPU-käsittely pysäytetään kokonaan — jäähdytys vaaditaan |

{% hint style="warning" %}
**Varmista riittävä ilmanvaihto ja lämmönpoisto** jatkuvaa käsittelyä varten, erityisesti suljetuissa kenttäkoteloissa tai lentokoneisiin asennetuissa järjestelmissä. Lämpösäätö vähentää prosessointikapasiteettia laitteiston suojaamiseksi.
{% endhint %}

***

## Muistin hallinta

Jetson-laitteet käyttävät **yhtenäistä muistia** — GPU ja CPU jakavat saman fyysisen RAM-muistin. Ilmoitettu VRAM (esim. ~15,3 GB Orin NX 16 GB:ssa) ei ole erillistä GPU-muistia; se on samaa RAM-muistia, jota käyttöjärjestelmä ja kaikki muut prosessit käyttävät.

### Swap-varoitus ja suositukset

Ennen käsittelyn aloittamista Jetsonissa komentorivi laskee syöttökansiossasi olevien RAW-kuvien määrän (`.tif`, `.tiff`, `.raw`, `.dng` — JPG-esikatselukuvia ei lasketa mukaan), arvioi suorituksen vaatiman muistin enimmäismäärän ja **varoittaa ennen aloittamista**, jos RAM-muistin ja swap-tilan yhteismäärä ei todennäköisesti riitä. Varoituksen otsikkona on `LOW MEMORY WARNING - Jetson Detected`, ja siinä ilmoitetaan kuvien lukumäärä, RAM-muistin, nykyisen swap-tilan ja arvioidun huipputarpeen, minkä jälkeen se antaa tarkat `fallocate` / `chmod` / `mkswap` / `swapon`-komennot, jotka on mitoitettu (ei koskaan alle 8 GB). Ohjelma pysähtyy muutamaksi sekunniksi, jotta viesti ei häviä vierityshistoriaan, minkä jälkeen käsittely jatkuu.**Varoituksessa käytetyt muistiarviot:**

| Debayer-tila | Perus | Kuva kohti |
| --- | --- | --- |
| Vakio | ~1,5 GB | ~10 MB |
| Tekstuuritietoinen | ~2,5 GB (malli + Python-ajoympäristö) | ~15 MB |

Varoitus laukeaa, kun arvioitu huippuarvo ylittää RAM-muistin ja swap-tilan summan vähennettynä 1 GB:n turvamarginaalilla, ja se laskee mukaan vain **tiedostopohjaisen** swap-tilan — pelkästään zram-asetuksetkin aiheuttavat varoituksen.

Voit lisätä swap-tilaa manuaalisesti (esimerkki: 8 GB):

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
```

<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->


### OOM (Out of Memory) -tilanteiden käsittely

Käsittelyn aikana Chloros valvoo GPU:n muistia ja hidastaa toimintaa sujuvasti kaatumisen sijaan:

1. Kun GPU:n muistin käyttöaste ylittää **85 %**, eräkokoja pienennetään ennaltaehkäisevästi
2. Jos muistin loppumistilanne silti tapahtuu, eräkoko **puolitetaan** ja puolitetaan uudelleen jokaisen peräkkäisen OOM-tilanteen yhteydessä; jokainen sen jälkeen onnistunut erä siirtää tämän rajoituksen yhden askeleen taaksepäin
3. Jatkuvan kuormituksen alla prosessiputki siirtyy `fused_gpu`-polusta muistitehokkaampaan `tiled_gpu`-polkuun ja viimeisenä keinona CPU-käsittelyyn

Koska GPU jakaa Jetsonin RAM-muistin, RAM-muisti ja swap-tila rajoittavat edelleen suorituksen kestoa — yllä oleva swap-varoitus on tarkistettava asia.

***

## Kenttäkäyttö

### Virrankulutusta koskevat huomioitavat seikat

| Jetson-malli     | Tyypillinen virrankulutus | Huomautukset                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7–15 W              | DC-tynnyriliitin          |
| Jetson Orin NX   | 10–25 W             | DC-tynnyriliitin          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD tai tynnyriliitin |

Suunnittele virrankulutus jatkuvaa prosessointia varten — suurin virrankulutus tapahtuu GPU:ta intensiivisesti käyttävän Thread 3:n (Processing) aikana.

### Tallennustilan suositukset

* **NVMe SSD** suositellaan vahvasti arm64-käyttöönottoihin
* SD-kortit ovat liian hitaita prosessointiin — käytä niitä vain käynnistysvälineenä
* Varaa prosessoidulle tulokselle 2–3-kertainen tila raakakuvadatan kokoon verrattuna

### Päätön käyttö SSH:n kautta

Chloros-CLI on ihanteellinen päätön Jetson-käyttöönottoihin:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Jatkuvasti päällä oleva taustapalvelu LATTICE- ja DAQ-E-aikasynkronointia varten

Jos Jetson ohjaa LATTICE-kameroita tai DAQ-E-valosensoreita ilman näyttöä, ota backend-systemd-palvelu käyttöön **omalla käyttäjätunnuksellasi**, jotta PTP-grandmaster toimii jatkuvasti ja SDK-skriptit löytävät käynnissä olevan backendin (yksikkö on asennettu, mutta sitä ei ole oletusarvoisesti otettu käyttöön):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
chloros-cli time-sync status
```

Tällöin taustapalvelu toimii sinun nimissäsi: sen tallentamat tiedot kuuluvat sinulle, ja se käyttää sinun `chloros-cli login`-tunnustasi. Katso [Linux-asennus](linux-installation.md#always-on-ptp-for-headless-hosts) saadaksesi lisätietoja, mukaan lukien root-käyttäjän `chloros-backend.service` ja siitä, miten paketti tekee PTP-porteista 319/320 sidottavissa ilman root-oikeuksia.

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

`chloros-cli process` palauttaa nollasta poikkeavan arvon, kun tuotteita pyytänyt suoritus ei kirjoita kuvia, joten systemd:n virhetila on merkityksellinen valvonnan kannalta.

Yhdistä systemd-ajastimeen aikataulutettua käsittelyä varten:

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
    --indices NDVI GNDVI OSAVI
```

### Python-SDK Jetsonilla

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Useiden lentojen eräkäsittely

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI OSAVI
    echo "Completed $name"
done
```

***

## Suositellut Jetson-järjestelmät kenttäkäyttöön

Kenttä- ja ilmakäyttöön kannattaa harkita seuraavia Jetson Orin NX 16 GB -kantolevyvaihtoehtoja:

* **Ilmakäyttö/drone**: Tärinänkestävät (MIL-STD) järjestelmät, kevyet (alle 300 g), passiivijäähdytys
* **Kestävä kenttäkäyttö**: IP67/IP69K-luokan vedenpitävät kotelot, joissa on PoE-GigE-kameraliitäntä
* **Minimaalinen/edullinen**: Kehittäjäsarjat lisäkoteloilla

Ota yhteyttä [MAPIR-tukeen](https://www.mapir.camera/community/contact), jos tarvitset laitteistosuosituksia juuri sinun käyttötilanteeseesi.

***

## Seuraavat vaiheet

* [Linux-asennus](linux-installation.md) — Yleisiä Linux-asennustietoja
* [Dynaaminen laskentakapasiteetin mukautus](../processing-architecture/dynamic-compute-adaptation.md) — Täydellinen laskentastrategiaopas
* [Käsittelyputki](../processing-architecture/processing-pipeline.md) — 4-säikeisen käsittelyputken ymmärtäminen
* [CLI: komentorivi](../CLI.md) — CLI-opas
* [API: Python-SDK](../api-python-sdk.md) — SDK-opas
* [CLI-viite](../reference/cli-reference.md) ja [SDK-viite](../reference/sdk-reference.md) — Kattavat komento- ja API-luettelot versioon 1.3.0
