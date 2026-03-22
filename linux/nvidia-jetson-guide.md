# NVIDIA Jetson -opas

Chloros mahdollistaa NVIDIA Jetson -alustalla monispektrisen kuvankäsittelyn reuna-laitteissa – kenttäolosuhteissa, lennokeissa ja etäasennuksissa. Chloros tunnistaa Jetson-mallisi automaattisesti ja optimoi käsittelystrategian laitteistollesi sopivaksi.

***

## Tuetut Jetson-mallit

| Malli                | RAM            | Käsittelystrategia                                   | Suositeltu käyttö                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 Gt jaettu | `GPU_PARALLEL` (4 työntekijää)                            | Maksimi suorituskyky, suuret tietojoukot                      |
| **Jetson Orin NX**   | 8–16 Gt jaettu  | `GPU_PARALLEL` (3 työntekijää, 16 Gt) / `GPU_SINGLE` (8 Gt) | Ensisijainen suositus ilmailu- ja kenttäkäyttöön |
| **Jetson Orin Nano** | 8 Gt jaettua     | `GPU_SINGLE` (1 työntekijä)                               | Perustason reunalaskenta                                 |
| **Jetson Nano**      | 4–8 Gt jaettua   | `GPU_SINGLE` (1 työntekijä)                               | Perustaso, muistirajoitteinen                          |

{% hint style="info" %}
**Vanhoja Jetson-malleja** (TX2, TX1, Xavier NX) ei välttämättä tueta. Suorituskyky vaihtelee käytettävissä olevan GPU-muistin ja CUDA-ominaisuuksien mukaan.
{% endhint %}

***

## Vaatimukset

* **JetPack 6.x** (suositellaan uusinta versiota)
* **NVIDIA CUDA** (sisältyy JetPackiin)
* **Chloros+ -lisenssi** (vaaditaan CLI/SDK-käyttöön)

## Asennus

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

Yleisiä Linux-asennustietoja on kohdassa [Linux-asennus](linux-installation.md).

***

## Dynaaminen laskentasovitus Jetsonissa

Chloros tunnistaa Jetson-mallisi automaattisesti ja valitsee optimaalisen käsittelystrategian. **Manuaalista säätämistä ei tarvita.**

### Toimintaperiaate

Käynnistyksen yhteydessä Chloros luo profiilin järjestelmästäsi:

1. **Tunnistaa Jetson-mallin** `/proc/device-tree/model`:n avulla
2. **Lukee käytettävissä olevan GPU:n/jaetun muistin**

3.**Valitsee käsittelystrategian** (`GPU_PARALLEL`, `GPU_SINGLE` tai `CPU_PARALLEL`)
4. **Asettaa työntekijöiden lukumäärän, putkityypin ja muistin allokoinnin** automaattisesti

### Mallikohtainen käyttäytyminen

| Jetson-malli                | Strategia       | Työntekijät | Putki                       | Samanaikaisuus |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8GB**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (muistitehokas) | Sarjoitettu  |
| **Jetson Orin Nano 8GB**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | Sarjoitettu  |
| **Jetson Orin NX 8GB**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | Sarjoitettu  |
| **Jetson Orin NX 16 Gt**     | `GPU_PARALLEL` | 3       | `fused_gpu` (täysi GPU-polku)    | Samanaikainen  |
| **Jetson AGX Orin 32–64 Gt** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | Samanaikainen  |

{% hint style="success" %}
**Jetson Orin NX 16GB** on ihanteellinen ratkaisu reuna-asennuksiin — se käyttää `GPU_PARALLEL`-strategiaa, jossa on 3 samanaikaista työntekijää, ja tarjoaa todellista rinnakkaista GPU-käsittelyä kompaktissa muodossa.
{% endhint %}

Alustojen välinen keskeinen ero on **muisti**. Jetson Nano, jossa on 8 Gt jaettua muistia, joutuu käsittelemään kuvia yksi kerrallaan muistitehokkaalla tiled-lähestymistavalla, kun taas 16 Gt:n muistilla varustettu Orin NX voi käsitellä kolmea kuvaa samanaikaisesti GPU:n kautta käyttämällä suuremman läpimenon fuusioitua putkea.

Katso täydellinen laskentasovituksen viite kohdasta [Dynaaminen laskentasovitus](../processing-architecture/dynamic-compute-adaptation.md).

***

## Lämmönhallinta

Jetson-laitteilla on rajoitettu lämpövaranto, erityisesti suljetuissa tai lentokoneissa käytettävissä asennuksissa. Chloros sisältää automaattisen lämpötilan seurannan ja rajoituksen:

| Lämpötila         | Toimenpide                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normaali toiminta — täysi prosessointinopeus          |
| **70 °C** (Varoitus)  | Pienennä erän kokoa automaattisesti                   |
| **80 °C** (Kriittinen) | Aggressiivinen tehonrajoitus — pienennä samanaikaisuutta         |
| **90 °C** (Sammutus) | Lopeta GPU-käsittely kokonaan — jäähdytys vaaditaan |

{% hint style="warning" %}
**Varmista riittävä ilmanvaihto ja lämmönpoisto** jatkuvaa käsittelyä varten, erityisesti suljetuissa kenttäkoteloissa tai lentokoneissa. Lämpösäätö vähentää käsittelyn läpimenoa laitteiston suojaamiseksi.
{% endhint %}

***

## Muistin hallinta

Jetson-laitteet käyttävät **yhtenäistä muistia** — GPU ja CPU jakavat saman fyysisen RAM-muistin. Tämä tarkoittaa, että ilmoitettu VRAM (esim. 15,3 Gt Orin NX 16 Gt:ssa) ei ole erillistä GPU-muistia, vaan se jaetaan käyttöjärjestelmän ja muiden prosessien kanssa.

### Vaihtomuistisuositukset

Suurille tietojoukoille tai tekstuuritietoiselle debayer-käsittelylle Chloros saattaa suositella vaihtomuistitilan luomista:

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Muistiarviot kuvaa kohti:**

* Tavallinen debayer: ~10 Mt kuvaa kohti
* Texture Aware -debayer: ~15 Mt kuvaa kohti

Chloros laskee tarvittavan muistin automaattisesti tietojoukon koon perusteella ja varoittaa, jos swap-tilaa suositellaan.

### OOM (Out of Memory) -varajärjestelmä

Jos muistin loppuminen havaitaan käsittelyn aikana:

1. Chloros vähentää automaattisesti GPU-työntekijöiden määrää
2. Siirtyy `fused_gpu`-putkesta `tiled_gpu`-putkeen (muistitehokkaampi)
3. Jatkaa käsittelyä pienemmällä läpimenolla kaatumisen sijaan

***

## Kenttäkäyttö

### Virrankulutusta koskevat huomioitavat seikat

| Jetson-malli     | Tyypillinen virrankulutus | Huomautukset                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5–10 W              | USB-C tai tynnyriliitin    |
| Jetson Orin Nano | 7–15 W              | DC-tynnyriliitin          |
| Jetson Orin NX   | 10–25 W             | DC-tynnyriliitin          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD tai tynnyriliitin |

Suunnittele virrankulutus jatkuvaa käsittelyä varten — suurin virrankulutus tapahtuu GPU-intensiivisessä Thread 3:ssa (käsittely).

### Tallennustilan suositukset

* **NVMe SSD** suositellaan vahvasti arm64-käyttöönottoihin
* SD-kortit ovat liian hitaita prosessointiin — käytä niitä vain käynnistysvälineenä
* Varaa prosessoidulle tuotokselle 2–3-kertainen määrä raakakuvadatan kokoa

### Päättömät käyttö SSH:n kautta

Chloros CLI on ihanteellinen headless-Jetson-käyttöönottoihin:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

### Automaattinen käsittely systemd:n avulla

Luo systemd-palvelu automaattista käsittelyä varten:

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

## Esimerkki työnkulkuja

### Perus Jetson-käsittely

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI
```

### Python SDK Jetsonissa

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

### Useiden lentojen eräkäsittely

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Suositellut Jetson-järjestelmät kenttäkäyttöön

Kenttä- ja ilmakäyttöön kannattaa harkita seuraavia Jetson Orin NX 16 GB -kantolevyvaihtoehtoja:

* **Ilmakäyttö/drone**: Järjestelmät, joilla on tärinäkestävyysluokitus (MIL-STD), kevyt paino (alle 300 g) ja passiivinen jäähdytys
* **Kestävä kenttäkäyttö**: IP67/IP69K-luokan vedenpitävät kotelot, joissa on PoE GigE -kamerayhteys
* **Minimaalinen/edullinen**: Kehittäjäsarjat lisäkoteloilla

Ota yhteyttä [MAPIR-tukeen](https://www.mapir.camera/community/contact) saadaksesi laitteistokohtaisia suosituksia käyttöympäristöösi.

***

## Seuraavat vaiheet

* [Linux-asennus](linux-installation.md) — Yleisiä Linux-asennustietoja
* [Dynaaminen laskentakapasiteetin mukautus](../processing-architecture/dynamic-compute-adaptation.md) — Täydellinen laskentastrategiaopas
* [Käsittelyputki](../processing-architecture/processing-pipeline.md) — 4-säikeisen putken ymmärtäminen
* [CLI : Komentorivi](../CLI.md) — Täydellinen CLI-viite
* [API : Python SDK](../api-python-sdk.md) — Täydellinen SDK-viite
