# API : Python SDK

**Chloros Python SDK** tarjoaa ohjelmoitavan pääsyn Chloros-kuvankäsittelymoottoriin, mikä mahdollistaa automaation, mukautetut työnkulut ja saumattoman integroinnin Python-sovelluksiin ja tutkimusputkiin.

### Tärkeimmät ominaisuudet

* 🐍 **Natiivi Python** - Puhdas, Pythonic API kuvankäsittelyyn
* 🔧 **Täysi API-pääsy** - Täydellinen hallinta Chloros-käsittelyyn
* 🚀 **Automaatio** - Rakenna mukautettuja eräkäsittelytyönkulkuja
* 🔗 **Integrointi** - Upota Chloros olemassa oleviin Python-sovelluksiin
* 📊 **Tutkimuskäyttöön valmis** - Täydellinen tieteellisiin analyysiputkiin
* ⚡ **Rinnakkaiskäsittely** - Skaalautuu CPU-ytimien mukaan (Chloros+)

### Vaatimukset

| Vaatimus          | Tiedot                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros Desktop**  | Täytyy olla asennettuna paikallisesti                                           |
| **Lisenssi**          | Chloros+ ([maksullinen tilaus vaaditaan](https://cloud.mapir.camera/pricing)) |
| **Käyttöjärjestelmä** | Windows 10/11 (64-bittinen)                                              |
| **Python**           | Python 3.7 tai uudempi                                                |
| **Muisti**           | Vähintään 8 Gt RAM-muistia (suositus 16 Gt)                                  |
| **Internet**         | Vaaditaan lisenssin aktivointiin                                     |

{% hint style=&quot;warning&quot; %}
**Lisenssivaatimukset**: Python SDK edellyttää maksullista Chloros+ -tilausta API-käyttöoikeuden saamiseksi. Vakiomuotoisissa (ilmaisissa) paketeissa ei ole API/SDK-käyttöoikeutta. Käy osoitteessa [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) päivittääksesi pakettisi.
{% endhint %}

## Pikaopas

### Asennus

Asenna pip:n avulla:

```bash
pip install chloros-sdk
```

{% hint style=&quot;info&quot; %}
**Ensimmäinen asennus**: Ennen kuin käytät SDK, aktivoi Chloros+ -lisenssisi avaamalla Chloros, Chloros (selain) tai Chloros CLI ja kirjautumalla sisään tunnuksillasi. Tämä tarvitsee tehdä vain kerran.
{% endhint %}

### Peruskäyttö

Käsittele kansio muutamalla rivillä:

```python
from chloros_sdk import process_folder

# One-line processing
results = process_folder("C:\\DroneImages\\Flight001")
```

### Täysi hallinta

Edistyneitä työnkulkuja varten:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")

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

Ennen kuin asennat SDK, varmista, että sinulla on:

1. **Chloros Desktop** asennettuna ([lataa](download.md))
2. **Python 3.7+** asennettuna ([python.org](https://www.python.org))
3. **Voimassa oleva Chloros+ -lisenssi** ([päivitys](https://cloud.mapir.camera/pricing))

### Asenna pip:n avulla

**Vakioasennus:**

```bash
pip install chloros-sdk
```

**Edistymisen seurannan tuella:**

```bash
pip install chloros-sdk[progress]
```

**Kehitystarkistuksen asennus:**

```bash
pip install chloros-sdk[dev]
```

### Tarkista asennus

Testaa, että SDK on asennettu oikein:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Ensimmäinen asennus

### Lisenssin aktivointi

SDK käyttää samaa lisenssiä kuin Chloros, Chloros (selain) ja Chloros CLI. Aktivoi kerran GUI:n tai CLI:n kautta:

1. Avaa **Chloros tai Chloros (selain)** ja kirjaudu sisään käyttäjä <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> -välilehdellä. Tai avaa **CLI**.
2. Syötä Chloros+ -tunnistetietosi ja kirjaudu sisään
3. Lisenssi tallennetaan paikallisesti välimuistiin (säilyy uudelleenkäynnistyksen jälkeen)

{% vihje style=&quot;success&quot; %}
**Kertaluonteinen asennus**: Kun olet kirjautunut sisään GUI:n tai CLI:n kautta, SDK käyttää automaattisesti välimuistissa olevaa lisenssiä. Lisäautentikointia ei tarvita!
{% endhint %}

### Testaa yhteys

Varmista, että SDK voi muodostaa yhteyden Chloros:ään:

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

Pääluokka paikalliselle Chloros-kuvankäsittelylle.

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
| `api_url`                 | str  | `"http://localhost:5000"` | URL paikallisesta Chloros-taustaprosessista          |
| `auto_start_backend`      | bool | `True`                    | Käynnistä backend automaattisesti tarvittaessa |
| `backend_exe`             | str  | `None` (automaattinen tunnistus)      | Polku backend-suoritustiedostoon            |
| `timeout`                 | int  | `30`                      | Pyynnön aikakatkaisu sekunteina            |
| `backend_startup_timeout` | int  | `60`                      | Backendin käynnistymisen aikakatkaisu (sekunteina) |

**Esimerkkejä:**

```python
# Default (auto-start backend)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom timeout
chloros = ChlorosLocal(timeout=60)
```

***

### Menetelmät

#### `create_project(project_name, camera=None)`

Luo uusi Chloros-projekti.

**Parametrit:**

| Parametri      | Tyyppi | Vaadittu | Kuvaus                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Kyllä      | Projektin nimi                                     |
| `camera`       | str  | Ei       | Kameramalli (esim. &quot;Survey3N\_RGN&quot;, &quot;Survey3W\_OCN&quot;) |

**Palauttaa:** `dict` - Projektin luomisen vastaus

**Esimerkki:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Tuo kuvat kansiosta.

**Parametrit:**

| Parametri     | Tyyppi     | Vaadittu | Kuvaus                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Kyllä      | Polku kuvien sisältävään kansioon         |
| `recursive`   | bool     | Ei       | Etsi alikansiot (oletus: False) |

**Palauttaa:** `dict` - Tuontitulokset tiedostojen lukumäärän kanssa

**Esimerkki:**

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
| `debayer`                 | str  | &quot;Korkea laatu (nopeampi)&quot; | Debayer-menetelmä                  |
| `vignette_correction`     | bool | `True`                  | Ota vignettikorjaus käyttöön      |
| `reflectance_calibration` | bool | `True`                  | Ota heijastavuuskalibrointi käyttöön  |
| `indices`                 | lista | `None`                  | Laskettavat kasvillisuusindeksit |
| `export_format`           | str  | &quot;TIFF (16-bittinen)&quot;         | Tulostusmuoto                   |
| `ppk`                     | bool | `False`                 | Ota PPK-korjaukset käyttöön          |
| `custom_settings`         | dict | `None`                  | Lisäasetukset        |

**Vientimuodot:**

* `"TIFF (16-bit)"` - Suositellaan GIS/fotogrammetriaan
* `"TIFF (32-bit, Percent)"` - Tieteellinen analyysi
* `"PNG (8-bit)"` - Silmämääräinen tarkastus
* `"JPG (8-bit)"` - Pakattu tulostus

**Käytettävissä olevat indeksit:**

NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 ja muut.

**Esimerkki:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

Käsittele projektikuvat.

**Parametrit:**

| Parametri           | Tyyppi     | Oletusarvo      | Kuvaus                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Käsittelytila: &quot;parallel&quot; tai &quot;serial&quot;   |
| `wait`              | bool     | `True`       | Odota valmistumista                       |
| `progress_callback` | callable | `None`       | Edistymisen palautusfunktio (progress, msg) |
| `poll_interval`     | float    | `2.0`        | Edistymisen kyselyväli (sekuntia)   |

**Palauttaa:** `dict` - Käsittelytulokset

{% hint style=&quot;warning&quot; %}
**Rinnakkaistila**: Vaatii Chloros+ -lisenssin. Skaalautuu automaattisesti CPU-ytimien mukaan (enintään 16 työntekijää).
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

**Palauttaa:** `dict` - Nykyinen projektin kokoonpano

**Esimerkki:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Hae taustaprosessin tilatiedot.

**Palauttaa:** `dict` - Taustaprosessin tila

**Esimerkki:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
```

***

#### `shutdown_backend()`

Sammuta taustaprosessi (jos se on käynnistetty SDK:llä).

**Esimerkki:**

```python
chloros.shutdown_backend()
```

***

### Käteviä toimintoja

#### `process_folder(folder_path, **options)`

Yhden rivin kätevä toiminto kansion käsittelyyn.

**Parametrit:**

| Parametri                 | Tyyppi     | Oletusarvo         | Kuvaus                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Pakollinen        | Polku kansioon, jossa kuvat ovat     |
| `project_name`            | str      | Automaattisesti luotu  | Projektin nimi                   |
| `camera`                  | str      | `None`          | Kameramalli                |
| `indices`                 | list     | `["NDVI"]`      | Laskettavat indeksit           |
| `vignette_correction`     | bool     | `True`          | Vignettikorjauksen käyttöönotto     |
| `reflectance_calibration` | bool     | `True`          | Heijastavuuden kalibroinnin käyttöönotto |
| `export_format`           | str      | &quot;TIFF (16-bittinen)&quot; | Tulostusmuoto                  |
| `mode`                    | str      | `"parallel"`    | Käsittelytila                |
| `progress_callback`       | kutsuttava | `None`          | Edistymisen palautus |

**Palauttaa:** `dict` - Käsittelyn tulokset

**Esimerkki:**

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
    debayer="High Quality (Faster)",
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

Käsittele useita lentotietojoukkoja:

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

Edistynyt edistymisen seuranta lokitiedostoilla:

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
        return False, f"Backend error: {e}. Ensure Chloros Desktop is installed."
    
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

### Esimerkki 7: Komentorivityökalu

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
    
    args = parser.parse_args()
    
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
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI
```

***

## Poikkeusten käsittely

SDK tarjoaa erityisiä poikkeusluokkia eri virhetyypeille:

### Poikkeusten hierarkia

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### Poikkeusten esimerkkejä

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros Desktop is installed.")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Edistyneet aiheet

### Mukautettu taustakonfiguraatio

Käytä mukautettua taustasovelluksen sijaintia tai konfiguraatiota:

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

Käsittele suuria tietojoukkoja erissä:

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

### Taustaprosessi ei käynnisty

**Ongelma:** SDK ei käynnistä taustaprosessia.

**Ratkaisut:**

1. Varmista, että Chloros Desktop on asennettu:

```python
import os
backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Tarkista, että Windows-palomuuri ei estä yhteyttä
3. Kokeile manuaalista backend-polkua:

```python
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")
```

***

### Lisenssiä ei tunnistettu

**Ongelma:** SDK varoittaa puuttuvasta lisenssistä

**Ratkaisut:**

1. Avaa Chloros, Chloros (selain) tai Chloros CLI ja kirjaudu sisään.
2. Varmista, että lisenssi on tallennettu välimuistiin:

```python
from pathlib import Path
import os

# Check cache location (Windows)
cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
print(f"Cache exists: {cache_path.exists()}")
```

3. Ota yhteyttä tukeen: info@mapir.camera

***

### Tuontivirheet

**Ongelma:** `ModuleNotFoundError: No module named 'chloros_sdk'`

**Ratkaisut:**

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

### Käsittelyn aikakatkaisu

**Ongelma:** Käsittelyn aikakatkaisu

**Ratkaisut:**

1. Lisää aikakatkaisua:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Käsittele pienempiä eriä
3. Tarkista käytettävissä oleva levytila
4. Valvo järjestelmän resursseja

***

### Portti jo käytössä

**Ongelma:** Backend-portti 5000 on varattu

**Ratkaisut:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Tai etsi ja sulje ristiriitainen prosessi:

```powershell
# PowerShell
Get-NetTCPConnection -LocalPort 5000
```

***

## Suorituskykyvinkkejä

### Optimoi käsittelynopeus

1. **Käytä rinnakkaistilaa** (vaatii Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Pienennä tulostustarkkuutta** (jos se on hyväksyttävää)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Poista tarpeettomat indeksit käytöstä**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Käsittele SSD-levyllä** (ei HDD-levyllä)

***

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

## Integraatioesimerkkejä

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

**V:** Vain ensimmäisen lisenssin aktivoinnin yhteydessä. Kun olet kirjautunut sisään Chloros:n, Chloros:n (selain) tai Chloros:n CLI:n kautta, lisenssi tallennetaan paikallisesti välimuistiin ja toimii offline-tilassa 30 päivän ajan.

***

### K: Voinko käyttää SDK:ää palvelimella, jossa ei ole graafista käyttöliittymää?

**V:** Kyllä! Vaatimukset:

* Windows Server 2016 tai uudempi
* Chloros asennettuna (kertaluonteinen)
* Lisenssi aktivoitu millä tahansa koneella (välimuistissa oleva lisenssi kopioitu palvelimelle)

***

### K: Mikä ero on Desktopin, CLI:n ja SDK:n välillä?

| Ominaisuus         | Desktop GUI | CLI Komentorivi | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Käyttöliittymä**   | Piste-klikkaus | Komento          | Python API  |
| **Sopii parhaiten**    | Visuaalinen työ | Skriptit        | Integraatio |
| **Automaatio**  | Rajoitettu     | Hyvä             | Erinomainen   |
| **Joustavuus** | Perustaso       | Hyvä             | Maksimi     |
| **Lisenssi**     | Chloros+    | Chloros+         | Chloros+    |

***

### K: Voinko jakaa SDK:llä rakennettuja sovelluksia?

**V:** SDK-koodi voidaan integroida sovelluksiisi, mutta:

* Loppukäyttäjillä on oltava asennettuna Chloros
* Loppukäyttäjillä on oltava aktiiviset Chloros+-lisenssit
* Kaupallinen jakelu edellyttää OEM-lisensointia

Ota yhteyttä info@mapir.camera:ään OEM-kyselyjen osalta.

***

### K: Kuinka päivitän SDK:n?

```bash
pip install --upgrade chloros-sdk
```

***

### K: Mihin käsitellyt kuvat tallennetaan?

Oletuksena projektipolkuun:

```
Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### K: Voinko käsitellä kuvia Python-skripteillä, jotka suoritetaan aikataulun mukaisesti?

**V:** Kyllä! Käytä Windows-tehtävien ajoitinta Python-skripteillä:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("C:\\Flights\\Today")
```

Ajoita tehtävien ajoittimella päivittäinen suoritus.

***

### K: Tukeeeko SDK async/await-toimintoa?

**V:** Nykyinen versio on synkroninen. Asynkronista toimintaa varten käytä `wait=False`-skriptiä tai suorita se erillisessä säikeessä:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

## Ohjeen saaminen

### Dokumentaatio

* **API-viite**: Tämä sivu

### Tukikanavat

* **Sähköposti**: info@mapir.camera
* **Verkkosivusto**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Hinnat**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Esimerkkikoodi

Kaikki tässä luetellut esimerkit on testattu ja ne ovat tuotantokäyttöön valmiita. Kopioi ja muokkaa niitä omaan käyttötarkoitukseesi sopiviksi.

***

## Lisenssi

**Omistusoikeudellinen ohjelmisto** - Copyright (c) 2025 MAPIR Inc.

SDK edellyttää voimassa olevaa Chloros+ -tilausta. Luvaton käyttö, jakelu tai muokkaaminen on kielletty.
