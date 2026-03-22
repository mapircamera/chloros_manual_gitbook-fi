# API : Python SDK

**Chloros Python SDK** tarjoaa ohjelmointipohjaisen pääsyn Chloros-kuvankäsittelymoottoriin, mikä mahdollistaa automaation, mukautetut työnkulut ja saumattoman integroinnin Python-sovelluksiisi ja tutkimusprosesseihisi.

### Tärkeimmät ominaisuudet

* 🐍 **Natiivi Python** - Siisti, Python-tyylinen API kuvankäsittelyyn
* 🔧 **Täysi API-käyttöoikeus** - Täydellinen hallinta Chloros-käsittelystä
* 🚀 **Automaatio** – Rakenna mukautettuja eräprosessointityönkulkuja
* 🔗 **Integrointi** – Upota Chloros olemassa oleviin Python-sovelluksiin
* 📊 **Tutkimusvalmis** – Täydellinen tieteellisiin analyysiprosesseihin
* ⚡ **Rinnakkaiskäsittely** - Skaalautuu CPU-ytimiesi mukaan (Chloros+)

### Vaatimukset

| Vaatimus          | Tiedot                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros asennettu** | Windows: Työpöytäasennusohjelma; Linux: `.deb`-paketti                  |
| **Lisenssi**          | Chloros+ ([vaatii maksullisen tilauksen](https://cloud.mapir.camera/pricing)) |
| **Käyttöjärjestelmä** | Windows 10/11 (64-bittinen), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Python**           | Python 3.7 tai uudempi                                                |
| **Muisti**           | Vähintään 8 Gt RAM-muistia (suositus 16 Gt)                                  |
| **Internet**         | Vaaditaan lisenssin aktivointia varten                                     |

{% hint style="warning" %}
**Lisenssivaatimus**: Python SDK vaatii maksullisen Chloros+ -tilauksen API-käyttöoikeuden saamiseksi. Vakiopaketeissa (ilmaiset) ei ole pääsyä API/SDK-palveluun. Vieraile [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) päivittääksesi tilauksesi.
{% endhint %}

## Pikaopas

### Asennus

Asenna pip:n kautta:

```bash
pip install chloros-sdk
```

{% hint style="info" %}
**Ensimmäinen asennus**: Ennen kuin käytät SDK:ää, aktivoi Chloros+ -lisenssisi avaamalla Chloros, Chloros (Selain) tai Chloros CLI ja kirjautumalla sisään tunnuksillasi. Tämä tarvitsee tehdä vain kerran. Linux:ssä (ei graafista käyttöliittymää) käytä: `chloros-cli login user@example.com 'password'`
{% endhint %}

### Peruskäyttö

Käsittele kansio vain muutamalla rivillä:

```python
from chloros_sdk import process_folder

# One-line processing (Windows)
results = process_folder("C:\\DroneImages\\Flight001")

# One-line processing (Linux)
results = process_folder("/home/user/drone_images/flight001")
```

{% hint style="info" %}
**Alustojen väliset polut**: Tämän sivun koodiesimerkit käyttävät Windows-tyylisiä polkuja (esim. `C:\\DroneImages\\Flight001`). Käytä Linux:ssä sen sijaan Linux-polkuja (esim. `/home/user/drone_images/flight001` tai `~/drone_images/flight001`). SDK toimii identtisesti molemmilla alustoilla.
{% endhint %}

### Täysi hallinta

Edistyneille työnkulkuille:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")  # Windows
# chloros.import_images("/home/user/drone_images/flight001")  # Linux

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## Asennusopas

### Edellytykset

Ennen kuin asennat SDK:n, varmista, että sinulla on:

1. **Chloros asennettuna** — Windows: Työpöytäasennusohjelma ([lataa](download.md)); Linux: `.deb`-paketti ([Linux-asennus](linux/linux-installation.md))
2. **Python 3.7+** asennettu ([python.org](https://www.python.org))
3. **Voimassa oleva Chloros+ lisenssi** ([päivitys](https://cloud.mapir.camera/pricing))

### Asenna pip:n kautta

**Vakiomuotoinen asennus:**

```bash
pip install chloros-sdk
```

**Edistymisen seurannan tuella:**

```bash
pip install chloros-sdk[progress]
```

**Kehitysasennus:**

```bash
pip install chloros-sdk[dev]
```

### Varmista asennus

Testaa, että SDK on asennettu oikein:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Ensimmäinen asennus

### Lisenssin aktivointi

SDK käyttää samaa lisenssiä kuin Chloros, Chloros (selain) ja Chloros CLI. Aktivoi kerran käyttöliittymän kautta tai CLI:**Windows:**Avaa**Chloros tai Chloros (selain)** ja kirjaudu sisään <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> -välilehdellä tai käytä CLI:ää.**Linux:** Käytä CLI:ää (käyttöliittymää ei ole saatavilla):

```bash
chloros-cli login user@example.com 'your_password'
```

Lisenssi tallennetaan paikallisesti välimuistiin ja säilyy uudelleenkäynnistyksien jälkeen.

{% hint style="success" %}
**Kertaluonteinen asennus**: Kun olet kirjautunut sisään käyttöliittymän tai CLI:n kautta, SDK käyttää automaattisesti välimuistissa olevaa lisenssiä. Lisätodentamista ei tarvita!
{% endhint %}

{% hint style="info" %}
**Uloskirjautuminen**: SDK-käyttäjät voivat tyhjentää välimuistissa olevat tunnistetiedot ohjelmoimalla käyttämällä `logout()`-menetelmää. Katso [logout()-menetelmä](#logout) API-viitteessä.
{% endhint %}

### Yhteyden testaaminen

Varmista, että SDK pystyy muodostamaan yhteyden Chloros:iin:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## API-viite

### ChlorosLocal-luokka

Pääluokka paikalliseen Chloros-kuvankäsittelyyn.

#### Konstruktori

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**Parametrit:**

| Parametri                 | Tyyppi | Oletusarvo                   | Kuvaus                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | Paikallisen Chloros-taustaprosessin URL          |
| `auto_start_backend`      | bool | `True`                    | Käynnistä taustapalvelu automaattisesti tarvittaessa |
| `backend_exe`             | str  | `None` (auto-tunnistus)      | Polku taustapalvelimen suoritustiedostoon            |
| `timeout`                 | int  | `30`                      | Pyynnön aikakatkaisu sekunteina            |
| `backend_startup_timeout` | int  | `60`                      | Taustapalvelimen käynnistymisen aikakatkaisu (sekuntia) |

**Esimerkkejä:**

```python
# Default (auto-start backend, auto-detect path on Windows and Linux)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path (Windows)
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom backend path (Linux)
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")

# Custom timeout with longer startup (e.g., for Jetson)
chloros = ChlorosLocal(timeout=60, backend_startup_timeout=120)
```

{% hint style="info" %}
**Alustojen välinen automaattinen tunnistus**: SDK kokeilee automaattisesti alustallesi sopivaa taustaprosessin polkua:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (manuaalinen)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

***

### Menetelmät

#### `create_project(project_name, camera=None)`

Luo uusi Chloros-projekti.

**Parametrit:**

| Parametri      | Tyyppi | Pakollinen | Kuvaus                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Kyllä      | Projektin nimi                                     |
| `camera`       | str  | Ei       | Kameramalli (esim. &quot;Survey3N\_RGN&quot;, &quot;Survey3W\_OCN&quot;) |

**Palauttaa:** `dict` - Projektin luomisen vastaus**Esimerkki:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Tuo kuvia kansiosta.

**Parametrit:**

| Parametri     | Tyyppi     | Pakollinen | Kuvaus                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Kyllä      | Polku kuvia sisältävään kansioon         |
| `recursive`   | bool     | Ei       | Etsi alikansioista (oletus: False) |

**Palauttaa:** `dict` - Tuontitulokset ja tiedostojen lukumäärä**Esimerkki:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Määritä käsittelyasetukset.

**Parametrit:**

| Parametri                 | Tyyppi | Oletus                 | Kuvaus                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | &quot;Standard (Fast, Medium Quality)&quot; | Debayer-menetelmä            |
| `vignette_correction`     | bool | `True`                  | Ota vignettikorjaus käyttöön      |
| `reflectance_calibration` | bool | `True`                  | Ota heijastavuuskalibrointi käyttöön  |
| `indices`                 | list | `None`                  | Laskettavat kasvillisuusindeksit |
| `export_format`           | str  | &quot;TIFF (16-bittinen)&quot;         | Tulostusmuoto                   |
| `ppk`                     | bool | `False`                 | Ota PPK-korjaukset käyttöön          |
| `custom_settings`         | dict | `None`                  | Edistyneet mukautetut asetukset        |

**Vientimuodot:**

* `"TIFF (16-bit)"` - Suositellaan GIS/fotogrammetriaan
* `"TIFF (32-bit, Percent)"` - Tieteellinen analyysi
* `"PNG (8-bit)"` - Silmämääräinen tarkastus
* `"JPG (8-bit)"` - Pakattu tulos

**Käytettävissä olevat indeksit:**NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 ja muut.**Esimerkki:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

Käsittele projektikuvia.

**Parametrit:**

| Parametri           | Tyyppi     | Oletusarvo      | Kuvaus                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Käsittelytila: &quot;parallel&quot; tai &quot;serial&quot;   |
| `wait`              | bool     | `True`       | Odota valmistumista                       |
| `progress_callback` | kutsuttava | `None`       | Edistymisen takaisinsoittofunktio (progress, msg) |
| `poll_interval`     | float    | `2.0`        | Edistymisen kyselyväli (sekuntia)   |

**Palauttaa:** `dict` - Käsittelyn tulokset

{% hint style="warning" %}
**Rinnakkaistila**: Vaatii Chloros+ -lisenssin. Skaalautuu automaattisesti CPU-ytimiesi mukaan (enintään 16 työntekijää).
{% endhint %}

**Esimerkki:**

```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```

***

#### `get_config()`

Hae nykyinen projektin kokoonpano.

**Palauttaa:** `dict` - Nykyinen projektin kokoonpano**Esimerkki:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Hakee taustapalvelimen tilatiedot, mukaan lukien kunkin säikeen käsittelyn etenemisen.

**Palauttaa:** `dict` - Backendin tila seuraavalla rakenteella:

```python
{
    "running": True,
    "url": "http://localhost:5000",
    "processing": {
        "percent": 75.0,
        "phase": "processing"
    },
    "export": {
        "percent": 50.0,
        "phase": "exporting",
        "active": True
    }
}
```

**Esimerkki:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
print(f"Processing: {status['processing']['percent']}%")
print(f"Export: {status['export']['percent']}% - Active: {status['export']['active']}")
```

***

#### `shutdown_backend()`

Sammuta backend (jos käynnistetty komennolla SDK).

**Esimerkki:**

```python
chloros.shutdown_backend()
```

***

#### `logout()`

Tyhjentää välimuistissa olevat tunnistetiedot paikallisesta järjestelmästä.

**Kuvaus:**

Kirjautuu ulos ohjelmoimalla poistamalla välimuistissa olevat tunnistetiedot. Tämä on hyödyllistä seuraavissa tilanteissa:
* Vaihtaminen eri Chloros+-tilien välillä
* Tunnistetietojen tyhjentäminen automatisoiduissa ympäristöissä
* Turvallisuussyistä (esim. tunnistetietojen poistaminen ennen asennuksen poistamista)

**Palauttaa:** `dict` - Uloskirjautumisen tulos**Esimerkki:**

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Clear cached credentials
result = chloros.logout()
print(f"Logout successful: {result}")

# After logout, login required via GUI/CLI/Browser before next SDK use
```

{% hint style="info" %}
**Uudelleentodennus vaaditaan**: Kun olet kutsunut `logout()`-funktiota, sinun on kirjauduttava uudelleen sisään Chloros-, Chloros (selain) tai Chloros CLI ennen kuin voit käyttää SDK-komentoa.
{% endhint %}

***

### Aputoiminnot

#### `process_folder(folder_path, **options)`

Yhden rivin aputoiminto kansion käsittelyyn.

**Parametrit:**

| Parametri                 | Tyyppi     | Oletusarvo         | Kuvaus                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Pakollinen        | Polku kuvia sisältävään kansioon     |
| `project_name`            | str      | Luodaan automaattisesti  | Projektin nimi                   |
| `camera`                  | str      | `None`          | Kameramalli                |
| `indices`                 | list     | `["NDVI"]`      | Laskettavat indeksit           |
| `vignette_correction`     | bool     | `True`          | Ota vignettikorjaus käyttöön     |
| `reflectance_calibration` | bool     | `True`          | Ota heijastavuuskalibrointi käyttöön |
| `export_format`           | str      | &quot;TIFF (16-bittinen)&quot; | Tulostusmuoto                  |
| `mode`                    | str      | `"parallel"`    | Käsittelytila                |
| `progress_callback`       | kutsuttava | `None`          | Edistymiskutsun palautus              |

**Palauttaa:** `dict` - Käsittelyn tulokset**Esimerkki:**

```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```

***

## Kontekstinhallinnan tuki

SDK tukee kontekstinhallintaa automaattista puhdistusta varten:

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***

## Täydelliset esimerkit

{% hint style="info" %}
**Linux-käyttäjät**: Kaikissa alla olevissa esimerkeissä käytetään Windows-polkuja. Korvaa `C:\\...`-polut omilla Linux-poluillasi (esim. `/home/user/...` tai `~/...`). Kaikki SDK-toiminnot ovat identtisiä kaikilla alustoilla.
{% endhint %}

### Esimerkki 1: Peruskäsittely

Käsittele kansio oletusasetuksilla:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Esimerkki 2: Mukautettu työnkulku

Täysi hallinta käsittelyputkesta:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### Esimerkki 3: Useiden kansioiden eräkäsittely

Käsittele useita lentodata-aineistoja:

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***

### Esimerkki 4: Tutkimusprosessin integrointi

Integroi Chloros data-analyysiin:

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```

***

### Esimerkki 5: Mukautettu edistymisen seuranta

Edistynyt edistymisen seuranta lokitiedostojen avulla:

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***

### Esimerkki 6: Virheiden käsittely

Vankka virheiden käsittely tuotantokäyttöön:

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros is installed (Windows installer or Linux .deb package)."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```

***

### Esimerkki 7: Tilinhallinta ja uloskirjautuminen

Hallitse tunnistetietoja ohjelmoimalla:

```python
from chloros_sdk import ChlorosLocal

def switch_account():
    """Clear credentials to switch to a different account"""
    try:
        chloros = ChlorosLocal()
        
        # Clear current credentials
        result = chloros.logout()
        print("✓ Credentials cleared successfully")
        print("Please log in with new account via Chloros, Chloros (Browser), or CLI")
        
        return True
    
    except Exception as e:
        print(f"✗ Logout failed: {e}")
        return False

def secure_cleanup():
    """Remove credentials for security purposes"""
    try:
        chloros = ChlorosLocal()
        chloros.logout()
        print("✓ Credentials removed for security")
        
    except Exception as e:
        print(f"Warning: Cleanup error: {e}")

# Switch accounts
if switch_account():
    print("\nRe-authenticate via Chloros GUI/CLI/Browser before next SDK use")

# Or perform secure cleanup
# secure_cleanup()
```

***

### Esimerkki 8: Komentorivityökalu

Rakenna mukautettu CLI-työkalu SDK:n avulla:

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    parser.add_argument('--logout', action='store_true',
                       help='Clear cached credentials before processing')
    
    args = parser.parse_args()
    
    # Handle logout if requested
    if args.logout:
        from chloros_sdk import ChlorosLocal
        chloros = ChlorosLocal()
        chloros.logout()
        print("Credentials cleared. Please re-login via Chloros GUI/CLI/Browser.")
        return 0
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**Käyttö:**

```bash
# Process multiple folders
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI

# Clear cached credentials
python my_processor.py --logout
```

***

## Poikkeusten käsittely

SDK tarjoaa erityisiä poikkeusluokkia eri virhetyypeille:

### Poikkeushierarkia

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### Poikkeusesimerkkejä

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros is installed (Windows installer or Linux .deb package).")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Edistyneitä aiheita

### Mukautettu taustapalvelimen konfigurointi

Käytä mukautettua taustapalvelimen sijaintia tai konfiguraatiota:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Estämätön käsittely

Aloita käsittely ja jatka muiden tehtävien parissa:

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### Muistin hallinta

Suurten tietojoukkojen kohdalla käsittele erissä:

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***

## Vianmääritys

### Taustapalvelu ei käynnisty

**Ongelma:** SDK ei onnistu käynnistämään taustapalvelua**Ratkaisut:**

1. Varmista, että Chloros on asennettu:

```python
import os
import platform

# Auto-detect backend path
if platform.system() == "Windows":
    backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
else:
    backend_path = "/usr/lib/chloros/chloros-backend"

print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Tarkista palomuuri (Windows) tai portin saatavuus (Linux: `lsof -i :5000`)
3. Kokeile manuaalista taustapalvelimen polkua:

```python
# Windows
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")

# Linux
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")
```

***

### Lisenssiä ei tunnistettu**Ongelma:** SDK varoittaa puuttuvasta lisenssistä**Ratkaisut:**

1. Avaa Chloros, Chloros (selain) tai Chloros CLI ja kirjaudu sisään.
2. Varmista, että lisenssi on tallennettu välimuistiin:

```python
from pathlib import Path
import os
import platform

# Check cache location
if platform.system() == "Windows":
    cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
else:
    cache_path = Path.home() / '.cache' / 'chloros'

print(f"Cache exists: {cache_path.exists()}")
```

3. Jos sinulla on ongelmia tunnistetietojen kanssa, tyhjennä välimuistissa olevat tunnistetiedot ja kirjaudu uudelleen sisään:

```python
from chloros_sdk import ChlorosLocal

# Clear cached credentials
chloros = ChlorosLocal()
chloros.logout()

# Then login again via Chloros, Chloros (Browser), or Chloros CLI
```

4. Ota yhteyttä tukeen: info@mapir.camera

***

### Tuontivirheet**Ongelma:** `ModuleNotFoundError: No module named 'chloros_sdk'`**Ratkaisut:**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### Käsittelyn aikakatkaisu**Ongelma:** Käsittelyn aikakatkaisu**Ratkaisut:**

1. Pidentä aikakatkaisua:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Käsittele pienempiä erä
3. Tarkista käytettävissä oleva levytila
4. Seuraa järjestelmän resursseja

***

### Portti jo käytössä**Ongelma:** Backend-portti 5000 varattu**Ratkaisut:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Tai etsi ja sulje ristiriitainen prosessi:

```powershell
# Windows PowerShell
Get-NetTCPConnection -LocalPort 5000
```

```bash
# Linux
lsof -i :5000
kill $(lsof -t -i :5000)
```

***

## Suorituskykyvinkkejä

### Optimoi käsittelynopeus

1. **Käytä rinnakkaistilaa** (vaatii Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Pienennä tulostusresoluutiota** (jos se on hyväksyttävää)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Poista tarpeettomat indeksit käytöstä**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Käsittele SSD-levyllä** (ei HDD-levyllä)***

### Muistin optimointi

Suurille tietojoukoille:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Taustakäsittely

Vapauta Python muille tehtäville:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## Integrointiesimerkkejä

### Django-integraatio

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyter Notebook

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***

## UKK

### K: Tarvitseeko SDK internetyhteyden?

**V:** Vain lisenssin ensimmäisen aktivoinnin yhteydessä. Kun olet kirjautunut sisään Chloros:n, Chloros:n (selain) tai Chloros:n kautta, lisenssi tallennetaan paikallisesti välimuistiin ja toimii offline-tilassa 30 päivän ajan.***

### K: Voinko käyttää SDK:ää palvelimella, jossa ei ole graafista käyttöliittymää?**V:** Kyllä! SDK toimii ilman näyttöä sekä Windows- että Linux-palvelimilla.**Linux (suositellaan headless-käyttöön):**
* Asenna `.deb`-paketin kautta
* Aktivoi lisenssi: `chloros-cli login user@example.com 'password'`

**Windows-palvelin:**
* Windows-palvelin 2016 tai uudempi
* Chloros asennettu (kertaluonteisesti)
* Lisenssi aktivoitu CLI:n kautta tai millä tahansa koneella

***

### K: Mitä eroa on Desktop-, CLI- ja SDK-versioilla?

| Ominaisuus         | Desktop-käyttöliittymä | CLI-komentorivi | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Käyttöliittymä**   | Piste-klikkaus | Komento          | Python API  |
| **Sopii parhaiten**    | Visuaaliseen työhön | Skriptaukseen        | Integrointiin |
| **Automaatio**  | Rajoitettu     | Hyvä             | Erinomainen   |
| **Joustavuus** | Perustaso       | Hyvä             | Maksimi     |
| **Lisenssi**     | Chloros+    | Chloros+         | Chloros+    |***

### K: Voinko jakaa SDK:llä rakennettuja sovelluksia?**V:** SDK-koodi voidaan integroida sovelluksiisi, mutta:

* Loppukäyttäjillä on oltava asennettuna Chloros
* Loppukäyttäjillä on oltava voimassa olevat Chloros+-lisenssit
* Kaupallinen jakelu edellyttää OEM-lisenssiä

Ota yhteyttä info@mapir.camera:ään OEM-kyselyissä.

***

### K: Miten päivitän SDK:n?

```bash
pip install --upgrade chloros-sdk
```

***

### K: Mihin käsitellyt kuvat tallennetaan?

Oletusarvoisesti projektipolkuun:

```

Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### K: Voinko käsitellä kuvia aikataulun mukaisesti suoritettavista Python-skripteistä?**V:** Kyllä! Käytä käyttöjärjestelmän ajastinta Python-skripteihin:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("/data/flights/today")  # Linux
# results = process_folder("C:\\Flights\\Today")  # Windows
```

**Windows:** Ajasta Tehtävien ajastimen kautta suoritettavaksi päivittäin.**Linux:** Ajasta cronin kautta:

```cron
# Run at 2 AM daily
0 2 * ** /usr/bin/python3 /home/user/scheduled_processing.py >> /var/log/chloros.log 2>&1
```

***

### K: Tukee SDK async/await-toimintoa?**V:** Nykyinen versio on synkroninen. Asynkronista toimintaa varten käytä `wait=False`-skriptiä tai suorita se erillisessä säikeessä:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

### K: Kuinka vaihdan eri Chloros+-tilien välillä?**V:** Käytä `logout()`-menetelmää tyhjentääksesi välimuistissa olevat tunnistetiedot ja kirjaudu sitten uudelleen uudella tilillä:

```python
from chloros_sdk import ChlorosLocal

# Clear current credentials
chloros = ChlorosLocal()
chloros.logout()

# Re-login via Chloros, Chloros (Browser), or Chloros CLI with new account
```

Kirjaudu ulos ja todenna uusi tili käyttöliittymän, selaimen tai CLI:n kautta ennen kuin käytät SDK:ää uudelleen.

***

## Ohjeiden saaminen

### Dokumentaatio

* **API-viite**: Tämä sivu

### Tukikanavat

* **Sähköposti**: info@mapir.camera
* **Verkkosivusto**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Hinnasto**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Esimerkkikoodi

Kaikki tässä luetellut esimerkit on testattu ja ne ovat tuotantokäyttöön valmiita. Kopioi ja muokkaa niitä omaan käyttötarkoitukseesi sopiviksi.

***

## Lisenssi**Omistusoikeudellinen ohjelmisto** - Copyright (c) 2025 MAPIR Inc.

SDK edellyttää voimassa olevaa Chloros+-tilausta. Luvaton käyttö, jakelu tai muokkaaminen on kielletty.
