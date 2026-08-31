# Linux:n asennus

Chloros toimitetaan Linux:lle `.deb`-paketteina, jotka asentavat CLI:n ja taustapalvelimen. Python SDK on erillinen pip-paketti (joka sisältyy myös `.deb`:ään version mukaisena wheel-tiedostona).

Pakettitiedostojen nimissä on versio ja arkkitehtuuri: `chloros_1.2.0_amd64.deb` x86_64:lle ja `chloros_1.2.0_arm64_jp6.deb` JetPack 6 Jetson -rakennuksille. Korvaa alla olevissa komennoissa tiedosto, jonka olet itse ladannut.

***

## Linux amd64 (x86_64)

### Järjestelmävaatimukset

| Vaatimus | Vähimmäisvaatimus | Suositeltu |
| --- | --- | --- |
| **Käyttöjärjestelmä** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Prosessori** | x86_64 (Intel/AMD) | Intel Core i7 tai parempi |
| **Muisti (RAM)** | 8 Gt | 16 Gt tai enemmän |
| **Näytönohjain** | Ei tarvita (käsitellään prosessorilla) | NVIDIA-näytönohjain, jossa vähintään 4 Gt VRAM-muistia (vähintään 12 Gt avaa `GPU_PARALLEL`-ominaisuuden, vähintään 7 Gt pitää Texture Aware -ominaisuuden pois käytöstä yksittäiskuvapolulla) |
| **Tallennustila** | 2 Gt vapaata tilaa | SSD, jossa vähintään 10 Gt vapaata tilaa |
| **Python** | Python 3.7+ (SDK:lle) | Python 3.10+ |

> **Ubuntu 20.04:ää ja Debian 11:tä ei tueta.** `.deb`:n riippuvuuslista on
> johdettu siitä, mihin Chloros-taustakomponentti tosiasiallisesti linkittyy, ja siihen sisältyy
> `libc6 (>= 2.34)`. Sekä Focal- että bullseye-jakeluissa toimitetaan glibc 2.31, joten `apt` hylkää
> asennuksen suoraan sen sijaan, että antaisi sen epäonnistua myöhemmin suorituksen aikana.

### Asennus

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
`dpkg -i` ei ratkaise riippuvuuksia. Jos se ilmoittaa puuttuvista paketeista, `sudo apt-get install -f` (tai `sudo apt --fix-broken install`) viimeistelee asennuksen — tämä on normaali kulku, ei virhe.
{% endhint %}

Varmista asennus:



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```***

## Linux arm64 (NVIDIA Jetson)

### Järjestelmävaatimukset

| Vaatimus | Vähimmäisvaatimus | Suositeltu |
| --- | --- | --- |
| **Alusta** | NVIDIA Jetson ja JetPack 6 | Jetson Orin NX 16 GB tai AGX Orin |
| **JetPack** | JetPack 6.x | Uusin JetPack 6 |
| **Muisti (RAM)** | 8 GB (jaettu GPU/CPU) | 16 GB+ jaettu (12 GB+ on kynnysarvo rinnakkaisille GPU-työntekijöille) |
| **Tallennustila** | 2 GB vapaata tilaa | NVMe SSD, jossa vähintään 10 GB vapaata tilaa |
| **Python** | Python 3.7+ (mallille SDK) | Python 3.10+ |

### Asennus

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Sama rakenne kuin amd64-versiossa `.deb`, mutta CUDA-rakennus on optimoitu Jetson Orin / Orin NX / Orin Nano -laitteille. Jetson-laitteiden muisti-, lämpö- ja kenttäkäyttöön liittyvistä ominaisuuksista on tietoa [NVIDIA Jetson -oppaassa](nvidia-jetson-guide.md).

***

## Python SDK:n asennus (kaikki Linux)

SDK on puhdas Python HTTP-asiakasohjelma taustapalvelimelle, joten sama paketti toimii sekä amd64- että arm64-alustoilla. Kaksi lähdettä:

**PyPI:stä** — julkaistu vakaa versio:

```bash
pip install chloros-sdk
```

**Mukana tulevasta wheel-tiedostosta** — sopii taatusti juuri asentamaasi CLI/backend-versioon (käytä tätä, jos `.deb`-versiosi on uudempi kuin PyPI:n):

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**PEP 668 -jakelut** (Ubuntu 23.10+, Debian 12+) eivät salli järjestelmänlaajuisia pip-asennuksia. Käytä `pip install --user …`:ää, virtuaaliympäristöä tai `sudo pip install --break-system-packages …`:ää. Pakettien asennusohjelma ei koskaan asenna SDK:ää automaattisesti järjestelmän Python:ään — valinta jää sinulle.
{% endhint %}

Valinnaiset lisäosat:

| Lisäosa | Komento | Lisää |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` reaaliaikaista edistymisen suoratoistoa varten |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` BLE (DAQ-M) -siirtoa varten |

Tarkista SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
`.deb` asentaa Chloros:n, CLI:n ja taustapalvelimen. Python, SDK kommunikoi kyseisen taustapalvelimen kanssa paikallisen HTTP API (`http://127.0.0.1:5000`) kautta ja käynnistää sen automaattisesti tarvittaessa. Käytä aina kirjaimellista IPv4-osoitetta `localhost`:n sijaan — `localhost` voi ratkaista `::1`:ksi ja kestää noin kaksi sekuntia pyyntöä kohti.
{% endhint %}

***

## Ensimmäinen asennus

### 1. Kirjaudu sisään

CLI- ja SDK-käyttöoikeudet edellyttävät maksullista Chloros+-tasoa (**Copper** tai korkeampi), jota valvotaan palvelinpuolella: uloskirjautunut käyttäjä saa tunnuksen `401 AUTH_REQUIRED` ja ilmaistason (Iron) käyttäjä tunnuksen `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Käyttöoikeustiedot tallennetaan välimuistiin tiedostossa `~/.chloros/user_session.json`.

{% hint style="warning" %}
**Sinun on kirjauduttava uudelleen sisään jokaisen asennuksen tai päivityksen jälkeen.** Paketin `prerm`-skripti tyhjentää tarkoituksella `~/.chloros/user_session.json`-tiedoston ja välimuistissa olevan lisenssin jokaiselta koneen käyttäjältä, jotta uusi versio tarkistaa lisenssin aina uudelleen sen sijaan, että se luottaisi vanhentuneeseen välimuistiin.
{% endhint %}

### 2. Tarkista lisenssin tila

```bash
chloros-cli status
```

`chloros-cli status` toimii kaikilla tasoilla (myös ilmaisella), joten voit aina nähdä, miksi pääsy on tai ei ole käytettävissä.

### 3. Suorita järjestelmän diagnostiikka

```bash
chloros-cli selftest
```

Seitsemän tarkistusta suoritetaan järjestyksessä, ja komento päättyy nollasta poikkeavalla tuloksella, jos jokin niistä epäonnistuu:

| # | Tarkistus | Mitä se todistaa |
| --- | --- | --- |
| 1 | **Versio** | CLI ilmoittaa versionsa (`v1.2.0`). |
| 2 | **Portti käytettävissä** | Portti 5000 on vapaa *tai* siihen on jo vastannut toimiva Chloros-taustapalvelin (mikä lasketaan hyväksytyksi). |
| 3 | **Taustapalvelimen käynnistys** | Taustapalvelimen binääritiedosto käynnistyy. |
| 4 | **API-testi (`/api/test`)** | Taustapalvelin vastaa `status: ok`. |
| 5 | **Järjestelmätiedot** | Tulostaa `GPU: <name>, CUDA: <bool>, PyTorch: <version>` tiedostosta `/api/system-info`. |
| 6 | **Denoiser-mallit** | Löytää `*.pth.enc`-mallit (Linux:ssa: `/usr/lib/chloros/models`). |
| 7 | **CUDA + kohinanpoistaja**| Texture Aware on todellakin käyttökelpoinen — vaatii CUDA:n**ja** vähintään yhden mallitiedoston. |

Suoritus päättyy `N/7 checks passed`:ään, ja siinä luetellaan mahdolliset virheet nimeltä.

### 4. Käsittele ensimmäinen tietojoukko

```bash
chloros-cli process ~/datasets/flight001
```

***

## Tiedostot ja hakemistot

### Käyttäjäkohtaiset

Chloros säilyttää tunnistetietonsa ja CLI-määrityksensä yhdessä alustojen välisessä hakemistossa, **`~/.chloros/`** (Windows:ssa, `%USERPROFILE%\.chloros\`:ssa). Kaksi Linux-kohtaista välimuistia noudattaa sen sijaan XDG-käytäntöjä — nämä noudattavat asetuksia `XDG_CONFIG_HOME` / `XDG_CACHE_HOME`, kun ne on määritetty.

| Polku | Käyttötarkoitus |
| --- | --- |
| `~/.chloros/user_session.json` | `chloros-cli login`:n kirjoittama kirjautumissession välimuisti (tyhjennetään jokaisen paketin asennuksen tai päivityksen yhteydessä) |
| `~/.chloros/working_directory.txt` | Oletusprojektikansion ohitus (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | CLI-kieliasetus (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Kieliasetus jaetaan Windows-käyttöliittymän kanssa — tässä `language`-asetus on etusijalla `cli_language.json`-asetukseen nähden |
| `~/.chloros/update_cache.json` | Yhden tunnin välimuisti Linux/Jetsonin käynnistyspäivitysten tarkistusta varten |
| `~/.chloros/backend.log` | Taustapalvelimen loki, kun taustapalvelin käynnistettiin CLI:n toimesta |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Kamerakohtaiset LATTICE-kalibrointipaketit välimuistissa, indeksoituna sarjanumeron ja nipun hajautusarvon perusteella |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Valinnaiset käyttäjän tekemät muutokset DAQ-kapasiteetin korjausprofiileihin |
| `~/.config/chloros/system_config.json` | Dynamic Compute Adaptation -toiminnosta välimuistiin tallennettu laitteistoprofiili — poista se, jos haluat pakottaa uuden laitteistotunnistuksen |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Taustapalvelimen lokitiedostot, yksi tiedosto kutakin käynnistystä kohti |
| `~/Chloros Projects/` | Oletusprojektikansio, kun ylikirjoitusta ei ole määritetty |

### Koko järjestelmän laajuinen

| Polku | Käyttötarkoitus |
| --- | --- |
| `/usr/bin/chloros-cli` | Käärekomentosarja — asettaa `LD_LIBRARY_PATH`:n mukana toimitettujen natiivikirjastojen osalta ja suorittaa sitten varsinaisen binääritiedoston |
| `/usr/bin/chloros-backend` | Kääre-skripti — sama kuin edellä, mutta lisäksi `CHLOROS_PRODUCTION=1`, jotta taustapalvelun todennusportti ei voi koskaan poistaa itseään käytöstä ilman ilmoitusta |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Käännetyt binäärit |
| `/usr/lib/chloros/arena_runtime/` | LATTICE-kameroiden edellyttämä Arena SDK-ajoympäristö |
| `/usr/lib/chloros/models/*.pth.enc` | Texture Aware -debayerin käyttämät salatut kohinanpoistomallit |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK -pyörä, joka vastaa tarkalleen tätä rakennetta |
| `/usr/lib/chloros/exiftool` | Mukana toimitettu exiftool (symlinkitetty `/usr/local/bin/exiftool`:ään vain, jos järjestelmässä ei ole exiftool-ohjelmaa) |
| `/etc/chloros/update.conf` | `chloros-cli update` lukee päivityskanavan asetukset |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Asettaa `net.ipv4.ip_unprivileged_port_start = 319`:n siten, että taustaprosessi voi sitoa PTP-portit ilman root-oikeuksia |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Ohjaa dynaamisen lataajan kohteeksi `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Myöntää kirjautuneelle käyttäjälle pääsyn DAQ-U USB-sarjaliikennesillalle (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend.service` | Ottaa käyttöön aina päällä olevan taustapalvelun (asennettu, **ei käytössä**) |
| `/usr/share/applications/chloros-cli.desktop` | Sovellusvalikon kohta ”Chloros CLI”, joka avaa terminaalin |

## Taustapalvelun suoritustiedoston sijainti

CLI ja SDK tunnistavat taustapalvelun automaattisesti:

| Komponentti | Polku |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Taustaprosessi | `/usr/lib/chloros/chloros-backend` |

Ohita taustapalvelimen polku lipulla `--backend-exe` CLI tai konstruktoriparametrilla `backend_exe` SDK, ja portin `--port`:llä (oletusarvo `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` osoittaa **`lattice`**-,**`project`**- ja**`daq pool-*`** komentoryhmät etätaustapalvelimella. Ydinkomennot (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) jättävät sen tarkoituksella huomiotta ja kohdistuvat aina komentoon `http://127.0.0.1:<port>`.
{% endhint %}

***

## LATTICE-kamerat ja DAQ-valosensorit laitteessa Linux

Kaikki live-hardware-komentoryhmät toimivat laitteella Linux (amd64 ja Jetson):

* **`chloros-cli lattice`** — tunnistaa, muodostaa yhteyden, määrittää ja tallentaa LATTICE-kameroista ja synkronoiduista matriiseista. `.deb` sisältää tarvittavan Arena SDK-ajoympäristön ja rekisteröi sen dynaamiseen lataajaan.
* **`chloros-cli daq pool-*`** — DAQ-U/M/E-valosensorien liittäminen taustapoolin kautta, kalibroitujen spektrien suoratoisto ja `.daq`-tiedostojen tallennus. Käännetty CLI sisältää ainoastaan `pool-*`-tuoteperheen: `pool-connect`, `pool-disconnect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`.
* **`chloros-cli project`** — aja tallennettua projektia (sen kameroita, antureita ja käsittelyasetuksia) ilman käyttöliittymää.
* **`chloros-cli time-sync`** — tarkasta PTP-grandmaster, jota Chloros-taustapalvelu käyttää LATTICE-kameroiden ja DAQ-E-anturien kanssa.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` on pakollinen `pool-latest`-, `pool-stream`-, `pool-record` ja `pool-set-cap`; `pool-list` näyttää tällä hetkellä poolissa olevat tunnukset.

{% hint style="info" %}
**Käytä mieluiten `--eth-host`:ää ensimmäisessä DAQ-E-yhteydessä moniverkkoyhteydellä varustetussa koneessa.** Automaattinen tunnistus etsii mDNS-verkosta, mutta voi jättää anturin rajapinnan huomiotta tyhjän ARP-välimuistin vuoksi, joten ensimmäinen `pool-connect --eth`-yhteys käynnistyksen jälkeen saattaa epäonnistua, vaikka anturi olisi täysin kunnossa. Anturin IP-osoitteen tai isäntänimen antaminen ohittaa tunnistuksen kokonaan.
{% endhint %}

**DAQ-U-sarjaliikenteen käyttöoikeudet** hallitaan asennetulla udev-säännöllä (`uaccess` + ryhmä `dialout`). Jos jo liitetty anturi ei ole käytettävissä, lataa säännöt uudelleen tai liitä se uudelleen:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Katso [CLI-ohje](../CLI.md) täydellisen komentojoukon osalta.

### Aina päällä oleva PTP päättömille isäntäkoneille

Ensimmäisellä asennuksella systemd-yksikkö `chloros-backend.service` luodaan, mutta sitä **ei oteta käyttöön**. Päätä vailla olevassa Jetson-laitteessa tai palvelimessa, jossa PTP-aikasynkronoinnin tulisi olla jatkuvasti käynnissä DAQ-E-antureiden ja LATTICE-kameroiden vuoksi, ota se käyttöön:

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

Ilman tätä PTP toimii vain silloin, kun Chloros-taustapalvelu on käynnissä — eli aktiivisen CLI/SDK-istunnon aikana.

Laite sitoo taustapalvelimen laitteen sisällä oleviin `127.0.0.1:5000` (laitteen sisäiset ympäristöasetukset `CHLOROS_HOST` / `CHLOROS_PORT`; ohita asetukset `sudo systemctl edit chloros-backend.service`:llä) ja käynnistää sen uudelleen 5 sekunnin kuluttua vian sattuessa.

**Miten PTP saa porttinsa.** PTP käyttää UDP-portteja 319/320, jotka molemmat ovat alle normaalin 1024:n etuoikeutettujen porttien rajan. Paketin `postinst` kirjoittaa `/etc/sysctl.d/60-chloros-ptp.conf`:n ja `net.ipv4.ip_unprivileged_port_start = 319`:n, mikä antaa taustapalvelimelle mahdollisuuden sitoa ne käynnissä ollessaan käyttäjänäsi. Se soveltaa myös `setcap cap_net_bind_service,cap_net_raw=+ep`-tiedostoa taustaprosessin binääritiedostoon varmuuden vuoksi — siksi `libcap2-bin` on määritelty paketin riippuvuudeksi.***

## Bash-skriptien esimerkkejä

{% hint style="info" %}
**Skriptausystävälliset poistumiskoodit.**`chloros-cli process` lopettaa `0`:n onnistuessaan ja**antaa nollasta poikkeavan arvon epäonnistuessaan — mukaan lukien suoritus, joka pyysi kuvatuotteita mutta ei kirjoittanut yhtään** (se tulostaa `Processing finished but wrote no image products.` ja mainitsee projektikansion sekä tavallisimmat syyt). Onnistuneet suoritukset ilmoittavat, kuinka monta kuvatuotetta tallennettiin (`Image products written: N`). Lopetuskoodit: `0` onnistuminen, `1` epäonnistuminen, `2` argumenttivirhe, `130` keskeytys.
{% endhint %}

### Useiden aineistojen käsittely

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Käsittely mukautetuilla asetuksilla

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

Kelvollisia `--format`-arvoja on tarkalleen neljä, ja ne sisältävät välilyöntejä — laita ne aina lainausmerkkien sisään:

| `--format`-arvo | Tulostuskansio |
| --- | --- |
| `TIFF (16-bit)` *(oletus)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` hyväksyy `standard` (oletus) tai `texture-aware` (Chloros+).

### Automaattinen käsittely Cronin avulla

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK Esimerkki

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Vianmääritys

### CLI ei löydy asennuksen jälkeen

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Käyttöoikeus evätty

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### ”setcap failed” asennuksen aikana

`.deb` soveltaa `cap_net_bind_service`:ää `/usr/lib/chloros/chloros-backend`:ään, jotta se voi sitoa PTP-portit 319/320 ilman pääkäyttäjän oikeuksia. Jos `libcap2-bin` puuttui asennuksen aikana, kutsu ohitetaan. Asenna se ja asenna paketti uudelleen:

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### PTP ei käynnisty / porttia 319 ei voi sitoa

Varmista, että rajoittamattomien porttien alaraja on laskettu, ja sovella se uudelleen nykyiseen käynnistykseen, jos sitä ei ole tehty:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Tarkista sitten grandmaster:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### &quot;LATTICE-kameran ajureita ei löydy&quot;

Arena SDK -ajoympäristöä ei tunnisteta. Varmista, että paketin kirjoittama latausohjelman konfiguraatio on olemassa ja päivitetty:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
ls /usr/lib/chloros/arena_runtime | head
```

### Taustapalvelimen käynnistys epäonnistui

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

Epäonnistuneen käynnistyksen taustaprosessin lokit ovat tiedostossa `~/.cache/chloros/logs/`.

### CUDA:ta ei havaittu

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

`chloros-cli selftest` ilmoittaa samasta asiasta yhdellä rivillä: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Puuttuvat jaetut kirjastot

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### Hidas käynnistys SD-korttijärjestelmissä

Kompiloidut binäärit purkautuvat väliaikaiseen hakemistoon jokaisen käynnistyksen yhteydessä. Jos tiedosto `/mnt/ssd/tmp` on olemassa, Chloros käyttää sitä automaattisesti; muussa tapauksessa aseta `TMPDIR` nopealle tiedostojärjestelmälle:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Chloros:n päivittäminen Linux:ssä

`update`-komento on käytettävissä vain Linux/Jetson-laitteissa. Se tarkistaa `/etc/chloros/update.conf`:ssä määritetyssä päivityskanavassa julkaistun version ja tarjoaa vastaavan `.deb`:n lataamista ja asentamista:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Linux/Jetson-laitteessa CLI suorittaa myös jokaisella käynnistyksellä ei-estävän päivitystarkistuksen (tulos tallennetaan välimuistiin tunniksi `~/.chloros/update_cache.json`:ssa) ja tulostaa viestin `Update available: vX.Y.Z`, jos uudempi versio on saatavilla. Asetuksesi ja projektisi säilyvät päivityksen jälkeen; sinun on kirjauduttava sisään uudelleen päivityksen jälkeen.

## Poistaminen

```bash
sudo apt remove chloros
```

Poisto pysäyttää `chloros-backend.service`:n, palauttaa oletusarvoisen rajoittamattomien porttien alarajan (1024), poistaa mukana toimitetun exiftool-symbolisen linkin ja Arena-lataajan asetukset sekä tyhjentää välimuistissa olevat tunnistetiedot. Projektisi ja `~/.chloros/`-datatiedostosi säilyvät ennallaan.

***

## Seuraavat vaiheet

* [NVIDIA Jetson -opas](nvidia-jetson-guide.md) — Jetson-laitteille ominaiset optimoinnit ja käyttöönotto
* [CLI : Komentorivi](../CLI.md) — CLI-opas
* [API : Python SDK](../api-python-sdk.md) — SDK-opas
* [CLI-viite](../reference/cli-reference.md) ja [SDK-viite](../reference/sdk-reference.md) — kattavat komento- ja API-luettelot versiolle 1.2.0
* [Dynaaminen laskentatehon mukautus](../processing-architecture/dynamic-compute-adaptation.md) — miten Chloros mukautuu laitteistoosi
