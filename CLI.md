# CLI : Komentorivi

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** tarjoaa tehokkaan komentorivipääsyn Chloros-kuvankäsittelymoottoriin, mikä mahdollistaa kuvankäsittelytyönkulkujen automatisoinnin, skriptien käytön ja päättömän käytön.

### Tärkeimmät ominaisuudet

* 🚀 **Automaatio** – Useiden tietojoukkojen skriptipohjainen eräkäsittely
* 🔗 **Integrointi** – Upotettavissa olemassa oleviin työnkulkuihin ja prosessiketjuihin
* 💻 **Päättömät toiminto** – Käytettävissä ilman graafista käyttöliittymää
* 🌍 **Monikielisyys** – Tuki 38 kielelle
* ⚡ **Rinnakkaisprosessointi** – [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) optimoi automaattisesti laitteistosi mukaan

### Vaatimukset

| Vaatimus          | Tiedot                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Käyttöjärjestelmä** | Windows 10/11 (64-bittinen), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Lisenssi**          | Chloros+ ([vaatii maksullisen tilauksen](https://cloud.mapir.camera/pricing)) |
| **Muisti**           | Vähintään 8 Gt RAM-muistia (suositellaan 16 Gt)                                  |
| **Internet**         | Vaaditaan lisenssin aktivointiin                                     |
| **Levytila**       | Vaihtelee projektin koon mukaan                                              |

{% hint style="warning" %}
**Lisenssivaatimus**: CLI vaatii maksullisen Chloros+-tilauksen. Standard- (ilmaiset) paketit eivät sisällä CLI-käyttöoikeutta. Siirry [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) päivittääksesi.
{% endhint %}

## Pikaopas

### Asennus

#### Windows

CLI sisältyy automaattisesti Chloros-asennusohjelmaan:

1. Lataa ja suorita **Chloros Installer.exe**

2. Suorita asennusohjeet loppuun
3. CLI asennettu: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
Asennusohjelma lisää automaattisesti `chloros-cli` järjestelmän PATH-polkuun. Käynnistä terminaali uudelleen asennuksen jälkeen.
{% endhint %}

#### Linux

Asenna arkkitehtuurillesi sopiva `.deb`-paketti:

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Yksityiskohtaiset ohjeet Linux:n asennuksesta löydät kohdasta [Linux:n asennus](linux/linux-installation.md).

### Ensimmäinen asennus

Ennen kuin käytät CLI-ohjelmaa, aktivoi Chloros+-lisenssisi:

**Windows:**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### Peruskäyttö

Käsittele kansio oletusasetuksilla:

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## Komento-opas

### Yleinen syntaksi

```
chloros-cli [global-options] <command> [command-options]
```

***

## Komennot

### `process` - Käsittele kuvia

Käsittele kansion kuvat kalibroinnin avulla.

**Syntaksi:**

```bash
chloros-cli process <input-folder> [options]
```

**Esimerkkejä:**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### Komennon käsittelyvaihtoehdot

| Vaihtoehto                | Tyyppi    | Oletus        | Kuvaus                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Polku    | _Pakollinen_     | Kansio, joka sisältää RAW/JPG-monispektrikuvia                                         |
| `-o, --output`        | Polku    | Sama kuin syöte  | Käsiteltyjen kuvien tulostuskansio                                                     |
| `-n, --project-name`  | Merkkijono  | Luodaan automaattisesti | Mukautettu projektin nimi                                                                    |
| `--vignette`          | Lippu    | Käytössä        | Ota vignettikorjaus käyttöön                                                             |
| `--no-vignette`       | Lippu    | -              | Poista vignettikorjaus käytöstä                                                            |
| `--reflectance`       | Lippu    | Käytössä        | Ota heijastavuuskalibrointi käyttöön                                                         |
| `--no-reflectance`    | Lippu    | -              | Poista heijastavuuskalibrointi käytöstä                                                        |
| `--ppk`               | Lippu    | Poissa käytöstä       | Käytä PPK-korjauksia .daq-valosensorin tiedoista                                      |
| `--format`            | Valinta  | TIFF (16-bittinen)  | Tulostusmuoto: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Kokonaisluku | Auto           | Kalibrointipaneelin tunnistuksen vähimmäiskohdekoko pikseleinä                          |
| `--target-clustering` | Kokonaisluku | Auto           | Kohteiden ryhmittelykynnys (0–100)                                                    |
| `--debayer`           | Valinta  | `standard`     | Debayer-menetelmä: `standard` tai `texture-aware` (vain Chloros+)                          |
| `--target`, `--targets` | Lippu  | Pois käytöstä       | Etsi kalibrointikohteita vain &quot;target&quot; tai &quot;targets&quot; -alikansiosta (nopeuttaa käsittelyä) |
| `--indices`           | Lista    | Ei mitään           | Laskettavat kasvillisuusindeksit (esim. `--indices NDVI NDRE GNDVI`)                    |
| `--exposure-pin-1`    | Merkkijono  | Ei mitään           | Lukitse valotus kameramallille (Pin 1)                                                 |
| `--exposure-pin-2`    | Merkkijono  | Ei mitään           | Kameramallin valotuksen lukitus (nasta 2)                                                 |
| `--recal-interval`    | Kokonaisluku | Automaattinen           | Uudelleenkalibrointiväli sekunteina                                                      |
| `--timezone-offset`   | Kokonaisluku | 0              | Aikavyöhykkeen siirtymä tunteina                                                               |

***

### `login` - Tilin todennus

Kirjaudu sisään Chloros+ -tunnuksillasi, jotta CLI-käsittely voidaan ottaa käyttöön.

**Syntaksi:**

```bash
chloros-cli login <email> <password>
```

**Esimerkki:**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Erikoismerkit**: Käytä yksinkertaisia lainausmerkkejä salasanojen ympärillä, jotka sisältävät merkkejä kuten `$`, `!` tai välilyöntejä.
{% endhint %}

**Tulostus:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Tyhjennä tunnistetiedot

Tyhjennä tallennetut tunnistetiedot ja kirjaudu ulos tililtäsi.

**Syntaksi:**

```bash
chloros-cli logout
```

**Esimerkki:**

```bash
chloros-cli logout
```

**Tulostus:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**SDK-käyttäjät**: Python SDK tarjoaa myös ohjelmointimenetelmän `logout()` tunnistetietojen tyhjentämiseen Python-skripteissä. Katso lisätietoja [Python SDK-dokumentaatiosta](api-python-sdk.md#logout).
{% endhint %}

***

### `status` - Tarkista lisenssin tila

Näyttää nykyisen lisenssin ja todennuksen tilan.

**Syntaksi:**

```bash
chloros-cli status
```

**Esimerkki:**

```bash
chloros-cli status
```

**Tulostus:**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` - Tarkista viennin eteneminen

Seuraa Thread 4 -viennin etenemistä käsittelyn aikana tai sen jälkeen.

**Syntaksi:**

```bash
chloros-cli export-status
```

**Esimerkki:**

```bash
chloros-cli export-status
```

**Käyttötapaus:** Käytä tätä komentoa käsittelyn ollessa käynnissä tarkistaaksesi viennin etenemisen.***

### `language` - Hallitse käyttöliittymän kieltä

Tarkastele tai muuta CLI-käyttöliittymän kieltä.

**Syntaksi:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Esimerkkejä:**

```bash
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Tuetut kielet (yhteensä 38)

| Koodi    | Kieli              | Alkuperäinen nimi      |
| ------- | --------------------- | ---------------- |
| `en`    | Englanti               | English          |
| `es`    | Espanja               | Español          |
| `pt`    | Portugali            | Português        |
| `fr`    | Ranska                | Français         |
| `de`    | Saksa                | Deutsch          |
| `it`    | Italia               | Italiano         |
| `ja`    | Japani              | 日本語              |
| `ko`    | Korea                | 한국어              |
| `zh`    | Kiina (yksinkertaistettu)  | 简体中文             |
| `zh-TW` | Kiina (perinteinen) | 繁體中文             |
| `ru`    | Venäjä               | Русский          |
| `nl`    | Hollanti                 | Nederlands       |
| `ar`    | Arabia                | العربية          |
| `pl`    | Puola                | Polski           |
| `tr`    | Turkki               | Türkçe           |
| `hi`    | Hindi                 | हिंदी            |
| `id`    | Indonesia                | Bahasa Indonesia |
| `vi`    | Vietnam                | Tiếng Việt       |
| `th`    | Thai                  | ไทย              |
| `sv`    | Ruotsi               | Svenska          |
| `da`    | Tanska                | Dansk            |
| `no`    | Norja             | Norsk            |
| `fi`    | Suomi               | Suomi            |
| `el`    | Kreikka                 | Ελληνικά         |
| `cs`    | Tšekki                 | Čeština          |
| `hu`    | Unkari             | Magyar           |
| `ro`    | Romania              | Română           |
| `uk`    | Ukraina             | Українська       |
| `pt-BR` | Brasilian portugali  | Português Brasileiro |
| `zh-HK` | Kantoninkiina             | 粵語             |
| `ms`    | Malaiji                 | Bahasa Melayu    |
| `sk`    | Slovakki                | Slovenčina       |
| `bg`    | bulgaria             | Български        |
| `hr`    | kroatia              | Hrvatski         |
| `lt`    | liettua            | Lietuvių         |
| `lv`    | Latvian               | Latviešu         |
| `et`    | Estonian              | Eesti            |
| `sl`    | Slovenian             | Slovenščina      |

{% hint style="success" %}
**Automaattinen tallennus**: Kieliasetuksesi tallennetaan tiedostoon `~/.chloros/cli_language.json` ja säilyy kaikissa istunnoissa.
{% endhint %}

***

### `set-project-folder` - Aseta oletusprojektikansio

Muuta oletusprojektikansion sijaintia (jaettu GUI:n kanssa tiedostossa Windows).

**Syntaksi:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Esimerkit:**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` - Näytä projektikansio

Näyttää nykyisen oletusprojektikansion sijainnin.

**Syntaksi:**

```bash
chloros-cli get-project-folder
```

**Esimerkki:**

```bash
chloros-cli get-project-folder
```

**Tulostus:**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` - Palauta oletusasetukset

Palauttaa projektikansion oletussijaintiin.

**Syntaksi:**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` - Suorita järjestelmän diagnostiikka

Suorita 7 diagnostiikkatarkistusta järjestelmän kokoonpanon varmistamiseksi.

**Syntaksi:**

```bash
chloros-cli selftest
```

**Suoritetut diagnostiikkatarkistukset:**

1. Versiotarkistus
2. Portin saatavuus (5000)
3. Backend-käynnistys
4. API-yhteystesti
5. Järjestelmätiedot ja GPU-tunnistus
6. Denoiser-mallien tarkistus
7. CUDA-saatavuustarkistus

{% hint style="info" %}
**Hyödyllistä vianmäärityksessä**: Suorita `selftest` asennuksen jälkeen varmistaaksesi, että järjestelmäsi on määritetty oikein, erityisesti Linux/Jetson-laitteissa, joissa GPU- ja CUDA-asetukset saattavat vaatia tarkistusta.
{% endhint %}

***

### `update` - Tarkista päivitykset (vain Linux)

Tarkista ja asenna CLI-päivitykset Linux-järjestelmissä.

**Syntaksi:**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| Vaihtoehto    | Kuvaus                        |
| --------- | ---------------------------------- |
| `--check` | Etsi vain päivityksiä, älä asenna |

{% hint style="info" %}
Tämä komento on käytettävissä vain Linux-järjestelmissä. Windows-järjestelmissä päivitykset toimitetaan asennusohjelman kautta.
{% endhint %}

***

## Yleiset asetukset

Nämä asetukset koskevat kaikkia komentoja:

| Asetus            | Tyyppi    | Oletus       | Kuvaus                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | Polku    | Tunnistetaan automaattisesti | Polku taustaprosessin suoritustiedostoon                       |
| `--port`          | Kokonaisluku | 5000          | Taustaprosessin API portin numero                          |
| `--restart`       | Lippu    | -             | Pakota taustaprosessin uudelleenkäynnistys (lopettaa olemassa olevat prosessit) |
| `--version`       | Lippu    | -             | Näytä versiotiedot ja poistu                |
| `--help`          | Lippu    | -             | Näytä ohjetiedot ja poistu                   |

{% hint style="info" %}
**Taustapalvelimen automaattinen tunnistus**: `--backend-exe`-polku tunnistetaan automaattisesti alustakohtaisesti:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (manuaalinen)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**Esimerkki globaaleilla asetuksilla:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux:**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## Käsittelyasetusten opas

### Rinnakkaiskäsittely ja dynaaminen laskentatehon sopeutus

Chloros 1.1.0 sisältää [dynaamisen laskentatehon sopeutuksen](processing-architecture/dynamic-compute-adaptation.md) — käsittelymoottori **tunnistaa laitteistosi automaattisesti** ja valitsee optimaalisen strategian:

| Alusta | Strategia | Työntekijät | Putki | Huomautukset |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` | Muistitehokas, sarjoitettu |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 3 | `fused_gpu` | Samanaikainen GPU-käsittely |
| **Pöytätietokone, jossa 8 Gt:n GPU** | `GPU_SINGLE` | 3 | `tiled_gpu` | Hyvä pöytätietokoneen suorituskyky |
| **Pöytätietokone, jossa on vähintään 12 Gt:n GPU** | `GPU_PARALLEL` | 3–4 | `fused_gpu` | Optimaalinen pöytätietokoneen suorituskyky |
| **Pelkkä CPU-järjestelmä** | `CPU_PARALLEL` | ytimet - 1 | `cpu_fallback` | Ei vaadi GPU:ta |

{% hint style="success" %}
**Ei manuaalista konfigurointia!** Chloros tunnistaa automaattisesti CPU:n, GPU:n, RAM-muistin ja (Jetsonissa) lämpötila-anturit ja määrittää sitten optimaalisen prosessointiputken automaattisesti.
{% endhint %}

### Debayer-menetelmät

| Menetelmä | CLI-lippu | Laatu | Nopeus | Lisenssi |
| --- | --- | --- | --- | --- |
| **Vakio (nopea, keskitasoinen laatu)** | `--debayer standard` | Hyvä | Nopea | Ilmainen / Chloros+ |
| **Tekstuuritietoinen (hidas, korkein laatu)** | `--debayer texture-aware` | Korkein | Hidas | Vain Chloros+ |

Oletusarvoinen debayer-menetelmä on **Standard**.**Texture Aware** -menetelmä käyttää AI/ML-kohinanpoistomallia korkealaatuisimman tuloksen saavuttamiseksi, mutta vaatii Chloros+ -lisenssin ja NVIDIA-näytönohjaimen.

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### Vignettikorjaus

**Toiminto:** Korjaa valon heikkenemistä kuvan reunoilla (kamerakuvissa yleiset tummemmat kulmat).

* **Oletusarvoisesti käytössä** – Useimpien käyttäjien tulisi pitää tämä käytössä
* Poista käytöstä `--no-vignette`:llä

{% hint style="success" %}
**Suositus**: Ota vignettikorjaus aina käyttöön, jotta kuvan kirkkaus on tasainen koko kehyksessä.
{% endhint %}

### Heijastavuuden kalibrointi

Muuntaa raakakuvasensorin arvot standardoiduiksi heijastavuusprosenteiksi kalibrointipaneelien avulla.

* **Oletusarvoisesti käytössä** – välttämätön kasvillisuuden analysoinnille
* Vaatii kalibrointikohdepaneeleita kuvissa
* Poista käytöstä `--no-reflectance`

{% hint style="info" %}
**Vaatimukset**: Varmista, että kalibrointipaneelit ovat kuvissa oikein valotettuja ja näkyvissä, jotta heijastavuusmuunnos on tarkka.
{% endhint %}

### PPK-korjaukset

**Toiminto:** Sovelletaan jälkikäsiteltyjä kinemaattisia korjauksia käyttämällä DAQ-A-SD-lokitietoja GPS-tarkkuuden parantamiseksi.

* **Oletusarvoisesti pois käytöstä**
* Käytä `--ppk`:ää ottaaksesi käyttöön
* Vaatii .daq-tiedostoja projektikansiossa MAPIR DAQ-A-SD-valosensorista.

### Tulostusmuodot

<table><thead><tr><th width="197">Muoto</th><th width="130.20001220703125">Bittisyvyys</th><th width="116.5999755859375">Tiedostokoko</th><th>Sopii parhaiten</th></tr></thead><tbody><tr><td><strong>TIFF (16-bittinen)</strong> ⭐</td><td>16-bittinen kokonaisluku</td><td>Suuri</td><td>GIS-analyysi, fotogrammetria (suositeltava)</td></tr><tr><td><strong>TIFF (32-bittinen, prosentti)</strong></td><td>32-bittinen liukuluku</td><td>Erittäin suuri</td><td>Tieteellinen analyysi, tutkimus</td></tr><tr><td><strong>PNG (8-bittinen)</strong></td><td>8-bittinen kokonaisluku</td><td>Keskikokoinen</td><td>Silmämääräinen tarkastus, jakaminen verkossa</td></tr><tr><td><strong>JPG (8-bittinen)</strong></td><td>8-bittinen kokonaisluku</td><td>Pieni</td><td>Pikakatselu, pakattu tulos</td></tr></tbody></table>***

## Automaatio ja skriptit

### PowerShell-eräkäsittely (Windows)

Käsittele useita tietojoukkojen kansioita automaattisesti Windows:

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Windows-eräskripti (Windows)

Yksinkertainen silmukka eräprosessointia varten Windows:

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Bash-eräprosessointi (Linux)

Useiden tietojoukkojen kansioiden käsittely Linux:

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### Python-automaatioskripti (alustariippumaton)

Edistynyt automaatio virheiden käsittelyllä (toimii Windows:ssä ja Linux:ssä):

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## Käsittelytyönkulku

### Vakiotyönkulku

1. **Syöte**: Kansio, joka sisältää RAW/JPG-kuvaparit
2. **Tunnistus**: CLI etsii automaattisesti tuetut kuvatiedostot
3. **Käsittely**: Rinnakkaistila skaalautuu prosessorin ytimien määrän mukaan (Chloros+)
4. **Tulos**: Luo kameramallikohtaiset alikansiot, joihin tallennetaan käsitellyt kuvat

### Esimerkki tulosrakenteesta

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### Arvioidut käsittelyajat

Tyypilliset käsittelyajat 100 kuvalle (kukin 12 MP):

| Alusta | Tila | Arvioitu aika | Huomautukset |
| --- | --- | --- | --- |
| **Pöytätietokone 12 GB+ GPU** | `GPU_PARALLEL` | 5–10 min | Nopein vaihtoehto |
| **Pöytätietokone 8 Gt:n GPU** | `GPU_SINGLE` | 10–15 min | Hyvä suorituskyky |
| **Jetson Orin NX 16 Gt** | `GPU_PARALLEL` | 15–25 min | Reunalaskenta |
| **Jetson Nano 8 Gt** | `GPU_SINGLE` | 30–60 min | Muistin rajoittama |
| **Vain CPU** | `CPU_PARALLEL` | 20–40 min | Ei vaadi GPU:ta |

{% hint style="info" %}
**Suorituskykyvinkki**: Käsittelyaika vaihtelee kuvien lukumäärän, tarkkuuden, debayer-menetelmän ja laitteiston mukaan. Texture Aware -debayer kestää huomattavasti kauemmin kuin Standard. Katso lisätietoja kohdasta [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md).
{% endhint %}

***

## Vianmääritys

### CLI ei löydy

**Windows Virhe:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Windows Ratkaisut:**

1. Tarkista asennuspaikka:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Käytä täydellistä polkua, jos se ei ole PATH-muuttujassa:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Lisää PATH-muuttujaan manuaalisesti:
   * Avaa Järjestelmän ominaisuudet → Ympäristömuuttujat
   * Muokkaa PATH-muuttujaa
   * Lisää: `C:\Program Files\Chloros\resources\cli`
   * Käynnistä terminaali uudelleen

**Linux Virhe:**

```
chloros-cli: command not found
```

**Linux Ratkaisut:**

1. Tarkista asennus:

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. Lataa shell uudelleen:

```bash
source ~/.bashrc
```

3. Tarkista käyttöoikeudet:

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### Backend-palvelimen käynnistys epäonnistui**Virhe:**

```

Backend failed to start within 30 seconds
```

**Ratkaisut:**

1. Tarkista, onko taustapalvelu jo käynnissä (sulje se ensin)
2. Tarkista, ettei palomuuri estä yhteyttä (Windows) tai tarkista portin saatavuus (Linux: `lsof -i :5000`)
3. Kokeile eri porttia:

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. Pakota taustapalvelimen uudelleenkäynnistys:

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. Tarkista kohdassa Linux, onko taustapalvelimen suoritustiedosto olemassa:

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### Lisenssi- ja todennusongelmat**Virhe:**

```

Chloros+ license required for CLI access
```

**Ratkaisut:**

1. Varmista, että sinulla on voimassa oleva Chloros+ -tilaus
2. Kirjaudu sisään tunnuksillasi:

```bash
chloros-cli login user@example.com 'password'
```

3. Tarkista lisenssin tila:

```bash
chloros-cli status
```

4. Ota yhteyttä tukeen: info@mapir.camera

***

### Kuvia ei löytynyt**Virhe:**

```

No images found in the specified folder
```

**Ratkaisut:**

1. Varmista, että kansio sisältää tuettuja tiedostomuotoja (.RAW, .TIF, .JPG)
2. Tarkista, että kansion polku on oikea (käytä lainausmerkkejä poluissa, joissa on välilyöntejä)
3. Varmista, että sinulla on lukuoikeudet kansioon
4. Tarkista, että tiedostotunnisteet ovat oikeat

***

### Käsittely pysähtyy tai jumittuu**Ratkaisut:**

1. Tarkista käytettävissä oleva levytila (varmista, että sitä on tarpeeksi tulostusta varten)
2. Sulje muut sovellukset muistin vapauttamiseksi
3. Vähennä kuvien määrää (käsittele erissä)

***

### Portti jo käytössä**Virhe:**

```

Port 5000 is already in use
```

**Ratkaisut:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux:**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## UKK

### K: Tarvitsenko lisenssin CLI:lle?

**V:**Kyllä! CLI vaatii maksullisen**Chloros+ -lisenssin**.

* ❌ Standard (ilmainen) -paketti: CLI pois käytöstä
* ✅ Chloros+ (maksullinen) -paketit: CLI täysin käytössä

Tilaa osoitteesta: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### K: Voinko käyttää CLI:ää palvelimella, jossa ei ole graafista käyttöliittymää?**V:** Kyllä! CLI toimii täysin ilman käyttöliittymää. Tämä on Linux:n pääasiallinen käyttötapa.**Windows-palvelin:**
* Windows Server 2016 tai uudempi
* Visual C++ Redistributable asennettuna

**Linux-palvelin:**
* Ubuntu 20.04+ / Debian 11+ (amd64) tai JetPack 6 (arm64)
* Asenna `.deb`-paketin kautta

**Molemmat alustat:**
* Vähintään 8 Gt RAM-muistia (suositus 16 Gt)
* Kertaluonteinen lisenssin aktivointi: `chloros-cli login user@example.com 'password'`

***

### K: Mihin käsitellyt kuvat tallennetaan?**V:**Oletusarvoisesti käsitellyt kuvat tallennetaan**samaan kansioon kuin syötteen** kameramallin alikansioihin (esim. `Survey3N_RGN/`).

Käytä `-o`-vaihtoehtoa määrittääksesi toisen tulostuskansion:

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### K: Voinko käsitellä useita kansioita kerralla?**V:** Ei suoraan yhdellä komennolla, mutta voit käyttää skriptejä kansioiden käsittelemiseen peräkkäin. Katso kohta [Automaatio ja skriptit](CLI.md#automation--scripting).***

### K: Miten tallennan CLI-tuloksen lokitiedostoon?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash:**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### K: Mitä tapahtuu, jos painan Ctrl+C käsittelyn aikana?**V:** CLI:

1. Lopettaa käsittelyn hallitusti
2. Sammuttaa taustapalvelimen
3. Poistuu koodilla 130

Osittain käsitellyt kuvat voivat jäädä tulostuskansioon.

***

### K: Voinko automatisoida CLI-käsittelyn?**V:** Totta kai! CLI on suunniteltu automatisointia varten. Katso [Automatisointi ja skriptit](CLI.md#automation--scripting) PowerShellille (Windows), Batchille (Windows), Bash (Linux) ja Python (alustojen välinen) esimerkkejä.***

### K: Miten tarkistan CLI-version?**V:**

```bash
chloros-cli --version
```

**Tulos:**

```

Chloros CLI 1.1.0
```

***

## Ohjeen saaminen

### Komentorivin ohje

Tarkastele ohjetietoja suoraan CLI:ssa:

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Tukikanavat

* **Sähköposti**: info@mapir.camera
* **Verkkosivusto**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Hinnat**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Täydelliset esimerkit

### Esimerkki 1: Peruskäsittely

Käsittele oletusasetuksilla (vignette, heijastavuus):

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### Esimerkki 2: Laadukas tieteellinen tulos

32-bittinen liukuluku TIFF:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### Esimerkki 3: Nopea esikatselukäsittely

8-bittinen PNG ilman kalibrointia nopeaa tarkastelua varten:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### Esimerkki 4: PPK-korjattu käsittely

Käytä PPK-korjauksia heijastavuuden avulla:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### Esimerkki 5: Mukautettu tulostuskohde

Käsittele eri kohteeseen tietyllä formaatilla:

**Windows:**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux:**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### Esimerkki 6: Todentamisprosessi

Täydellinen todentamisprosessi (sama kaikilla alustoilla):

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Esimerkki 7: Monikielinen käyttö

Käyttöliittymän kielen vaihtaminen (sama kaikilla alustoilla):

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```
