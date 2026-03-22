# Linux:n asennus

Chloros toimitetaan Linux:lle `.deb`-paketteina, jotka asentavat CLI:n ja taustapalvelimen. Python SDK asennetaan erikseen pip:n kautta.

***

## Linux amd64 (x86_64)

### Järjestelmävaatimukset

| Vaatimus | Vähimmäisvaatimus | Suositeltu |
| --- | --- | --- |
| **Jakelu** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **Prosessori** | x86_64 (Intel/AMD) | Intel Core i7 tai parempi |
| **Muisti (RAM)** | 8 Gt | 16 Gt tai enemmän |
| **Näytönohjain** | Ei tarvita (CPU-käsittely) | NVIDIA-näytönohjain, jossa vähintään 4 Gt VRAM-muistia |
| **Tallennustila** | 2 Gt vapaata tilaa | SSD, jossa vähintään 10 Gt vapaata tilaa |
| **Python** | Python 3.7+ (SDK:lle) | Python 3.10+ |

### Asennus

Lataa `.deb`-paketti ja asenna:

```bash
sudo dpkg -i chloros-amd64.deb
```

Varmista asennus:

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### Järjestelmävaatimukset

| Vaatimus | Vähimmäisvaatimus | Suositeltu |
| --- | --- | --- |
| **Alusta** | NVIDIA Jetson ja JetPack 6 | Jetson Orin NX 16 Gt tai AGX Orin |
| **JetPack** | JetPack 6.x | Uusin JetPack 6 |
| **Muisti (RAM)** | 8 Gt (jaettu GPU/CPU) | 16 Gt+ jaettu |
| **Tallennustila** | 2 Gt vapaata tilaa | NVMe SSD, jossa 10 Gt+ vapaata tilaa |
| **Python** | Python 3.7+ (SDK:lle) | Python 3.10+ |

### Asennus

Lataa JetPack 6 `.deb` -paketti ja asenna:

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

Varmista asennus:

```bash
chloros-cli --version
```

Yksityiskohtaiset ohjeet Jetsonin asennuksesta, mukaan lukien lämmönhallinta ja kenttäkäyttöönotto, löydät [NVIDIA Jetson -oppaasta](nvidia-jetson-guide.md).

***

## Python SDK-asennus (kaikki Linux)

Python SDK asennetaan erikseen pip-komennolla ja se toimii sekä amd64- että arm64-alustoilla:

```bash
pip install chloros-sdk
```

Valinnaisen etenemisen suoratoistotuen lisääminen:

```bash
pip install chloros-sdk[progress]
```

Varmista SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
`.deb`-paketti asentaa Chloros CLI:n ja taustapalvelimen. Python SDK on erillinen pip-paketti, joka kommunikoi taustapalvelimen kanssa paikallisen HTTP API:n kautta.
{% endhint %}

***

## Määrityskansiot

Chloros Linux:ssä noudattaa [XDG-peruskansiospesifikaatiota](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html):

| Tarkoitus | Linux Polku | Windows Vastaava |
| --- | --- | --- |
| **Määritykset** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **Tiedot / Projektit** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **Välimuisti / tunnistetiedot** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## Taustaprosessin suoritustiedostojen sijainnit

`.deb`-paketti asentaa taustaprosessin vakiosijaintiin. CLI ja SDK tunnistavat taustapalvelimen polun automaattisesti:

| Asennustapa | Taustapalvelimen polku |
| --- | --- |
| `.deb`-paketti | `/usr/lib/chloros/chloros-backend` |
| Manuaalinen / mukautettu | `/opt/mapir/chloros/backend/chloros-backend` |

Voit ohittaa taustapalvelimen polun `--backend-exe`- tai CLI-lipulla tai `backend_exe`- tai SDK-konstruktoriparametrilla.

***

## Ensimmäinen asennus

### 1. Aktivoi lisenssisi

Chloros+ -lisenssi vaaditaan CLI- ja SDK-käyttöoikeuksiin:

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. Tarkista lisenssin tila

```bash
chloros-cli status
```

### 3. Käsittele ensimmäinen tietojoukko

```bash
chloros-cli process ~/datasets/flight001
```

### 4. Suorita järjestelmän diagnostiikka

Varmista, että järjestelmäsi on määritetty oikein:

```bash
chloros-cli selftest
```

Tämä suorittaa 7 diagnostiikkatarkistusta, mukaan lukien versio, taustapalvelimen käynnistys, API-yhteys ja CUDA/GPU:n saatavuus.

***

## Esimerkkejä Bash-skripteistä

### Käsittele useita tietojoukkoja

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### Käsittele mukautetuilla asetuksilla

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### Automaattinen käsittely Cronilla

Lisää crontab-tiedostoon (`crontab -e`) käsitelläksesi uusia tietojoukkoja automaattisesti:

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

Jos `chloros-cli` ei löydy `.deb`-paketin asennuksen jälkeen:

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### Käyttöoikeus evätty

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
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

### CUDA:ta ei havaittu

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### Puuttuvia jaettuja kirjastoja

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## Chloros:n päivittäminen Linux:ssa

Käytä sisäänrakennettua päivityskomentoa päivitysten tarkistamiseen ja asentamiseen:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## Seuraavat vaiheet

* [NVIDIA Jetson -opas](nvidia-jetson-guide.md) — Jetson-laitteille ominaiset optimoinnit ja käyttöönotto
* [CLI : Komentorivi](../CLI.md) — Täydellinen CLI-komento-opas
* [API : Python SDK](../api-python-sdk.md) — Täydellinen SDK-ohje
* [Dynaaminen laskentakapasiteetin mukautus](../processing-architecture/dynamic-compute-adaptation.md) — Miten Chloros mukautuu laitteistoosi
