# CLI : Komentorivi

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** tarjoaa tehokkaan komentorivipääsyn Chloros-kuvankäsittelymoottoriin, mikä mahdollistaa automaation, skriptien käytön ja headless-toiminnon kuvankäsittelytyönkulkuissasi.

### Tärkeimmät ominaisuudet

* 🚀 **Automaatio** – Useiden tietojoukkojen skriptien eräprosessointi
* 🔗 **Integraatio** – Upotus olemassa oleviin työnkulkuihin ja putkistoihin
* 💻 **Headless-käyttö** – Käyttö ilman graafista käyttöliittymää
* 🌍 **Monikielisyys** – Tuki 38 kielelle
* ⚡ **Rinnakkaiskäsittely** – Skaalautuu dynaamisesti CPU:hun (jopa 16 rinnakkaista työntekijää)

### Vaatimukset

| Vaatimus          | Tiedot                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Käyttöjärjestelmä** | Windows 10/11 (64-bittinen)                                              |
| **Lisenssi**          | Chloros+ ([maksullinen paketti vaaditaan](https://cloud.mapir.camera/pricing)) |
| **Muisti**           | Vähintään 8 Gt RAM-muistia (suositellaan 16 Gt)                                  |
| **Internet**         | Vaaditaan lisenssin aktivoimiseksi                                     |
| **Levytila**       | Vaihtelee projektin koon mukaan                                              |

{% hint style=&quot;warning&quot; %}
**Lisenssivaatimus**: CLI edellyttää maksullista Chloros+ -tilausta. Standard (ilmainen) -paketeissa ei ole CLI-käyttöoikeutta. Käy osoitteessa [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) päivittääksesi.
{% endhint %}

## Pikaopas

### Asennus

CLI sisältyy automaattisesti Chloros-asennusohjelmaan:

1. Lataa ja suorita **Chloros Installer.exe**
2. Suorita asennusohjeet loppuun
3. CLI asennettu: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style=&quot;success&quot; %}
Asennusohjelma lisää `chloros-cli` automaattisesti järjestelmän PATH-polkuun. Käynnistä päätelaite uudelleen asennuksen jälkeen.
{% endhint %}

### Ensimmäinen asennus

Ennen kuin käytät CLI:ää, aktivoi Chloros+ -lisenssisi:

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### Peruskäyttö

Käsittele kansio oletusasetuksilla:

```powershell
chloros-cli process "C:\Images\Dataset001"
```

***

## Komentojen viite

### Yleinen syntaksi

```
chloros-cli [global-options] <command> [command-options]
```

***

## Komennot

### `process` - Käsittele kuvia

Käsittele kansion kuvat kalibroimalla.

**Syntaksi:**

```bash
chloros-cli process <input-folder> [options]
```

**Esimerkki:**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### Käsittelykomennon vaihtoehdot

| Vaihtoehto                | Tyyppi    | Oletus        | Kuvaus                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Polku    | _Vaaditaan_     | Kansio, joka sisältää RAW/JPG-monispektrikuvia                                         |
| `-o, --output`        | Polku    | Sama kuin syöte  | Käsiteltyjen kuvien tulostuskansio                                                     |
| `-n, --project-name`  | Merkkijono  | Automaattisesti luotu | Mukautettu projektin nimi                                                                    |
| `--vignette`          | Lippu    | Käytössä        | Ota vignettikorjaus käyttöön                                                             |
| `--no-vignette`       | Lippu    | -              | Poista vignettikorjaus käytöstä                                                            |
| `--reflectance`       | Lippu    | Käytössä        | Ota heijastavuuden kalibrointi käyttöön                                                         |
| `--no-reflectance`    | Lippu    | -              | Poista heijastavuuden kalibrointi käytöstä                                                        |
| `--ppk`               | Lippu    | Pois käytöstä       | Käytä PPK-korjauksia .daq-valosensorin tiedoista                                      |
| `--format`            | Valinta  | TIFF (16-bittinen)  | Tulostusmuoto: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Kokonaisluku | Auto           | Kalibrointipaneelin tunnistuksen vähimmäiskohdekoko pikseleinä                          |
| `--target-clustering` | Kokonaisluku | Auto           | Kohteiden ryhmittelyn kynnysarvo (0–100)                                                    |
| `--exposure-pin-1`    | Merkkijono  | Ei mitään           | Lukitse kameramallin valotus (nasta 1)                                                 |
| `--exposure-pin-2`    | Merkkijono  | Ei mitään           | Lukitse kameramallin valotus (nasta 2)                                                 |
| `--recal-interval`    | Kokonaisluku | Auto           | Uudelleenkalibrointiväli sekunteina                                                      |
| `--timezone-offset`   | Kokonaisluku | 0              | Aikavyöhykkeen ero tunteina                                                               |

***

### `login` - Todennetaan tili

Kirjaudu sisään Chloros+ -tunnuksillasi, jotta CLI-käsittely voidaan ottaa käyttöön.

**Syntaksi:**

```bash
chloros-cli login <email> <password>
```

**Esimerkki:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Erikoismerkit**: Käytä yksittäisiä lainausmerkkejä salasanojen ympärillä, jotka sisältävät merkkejä kuten `$`, `!` tai välilyöntejä.
{% endhint %}

**Tulos:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Tyhjennä tunnistetiedot

Tyhjennä tallennetut tunnistetiedot ja kirjaudu ulos tililtäsi.

**Syntaksi:**

```bash
chloros-cli logout
```

**Esimerkki:**

```powershell
chloros-cli logout
```

**Tulos:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

***

### `status` - Tarkista lisenssin tila

Näytä nykyinen lisenssi ja todennustila.

**Syntaksi:**

```bash
chloros-cli status
```

**Esimerkki:**

```powershell
chloros-cli status
```

**Tulos:**

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

### `export-status` - Tarkista vientiprosessin eteneminen

Seuraa Thread 4 -vientiprosessin etenemistä käsittelyn aikana tai sen jälkeen.

**Syntaksi:**

```bash
chloros-cli export-status
```

**Esimerkki:**

```powershell
chloros-cli export-status
```

**Käyttötapaus:** Kutsu tämä komento käsittelyn aikana tarkistaaksesi viennin etenemisen.

***

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

**Esimerkit:**

```powershell
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
| `en`    | Englanti               | Englanti          |
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
| `id`    | Indonesia            | Bahasa Indonesia |
| `vi`    | Vietnam            | Tiếng Việt       |
| `th`    | Thai                  | ไทย              |
| `sv`    | Ruotsin kieli               | Svenska          |
| `da`    | Tanskan kieli                | Dansk            |
| `no`    | Norjan kieli             | Norsk            |
| `fi`    | Suomi               | Suomi            |
| `el`    | Kreikka                 | Ελληνικά         |
| `cs`    | Tšekki                 | Čeština          |
| `hu`    | Unkari             | Magyar           |
| `ro`    | Romania              | Română           |
| `uk`    | Ukraina             | Українська       |
| `pt-BR` | Brasilian portugali  | Português Brasileiro |
| `zh-HK` | Kantoni             | 粵語             |
| `ms`    | Malaiji                 | Bahasa Melayu    |
| `sk`    | Slovakki                | Slovenčina       |
| `bg`    | Bulgarian             | Български        |
| `hr`    | Croatian              | Hrvatski         |
| `lt`    | Lithuanian            | Lietuvių         |
| `lv`    | Latvian               | Latviešu         |
| `et`    | Estonian              | Eesti            |
| `sl`    | Slovenian             | Slovenščina      |

{% hint style=&quot;success&quot; %}
**Automaattinen pysyvyys**: Kielivalintasi tallennetaan `~/.chloros/cli_language.json`:ään ja se säilyy kaikissa istunnoissa.
{% endhint %}

***

### `set-project-folder` - Aseta oletusprojektikansio

Muuta oletusprojektikansion sijaintia (jaettu GUI:n kanssa).

**Syntaksi:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Esimerkki:**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder` - Näytä projektikansio

Näytä nykyisen oletusprojektikansion sijainti.

**Syntaksi:**

```bash
chloros-cli get-project-folder
```

**Esimerkki:**

```powershell
chloros-cli get-project-folder
```

**Tulos:**

```
ℹ Current project folder: C:\Projects\2025
```

***

### `reset-project-folder` - Palauta oletusasetukset

Palauta projektikansio oletuspaikkaan.

**Syntaksi:**

```bash
chloros-cli reset-project-folder
```

***

## Yleiset asetukset

Nämä asetukset koskevat kaikkia komentoja:

| Asetus          | Tyyppi    | Oletus       | Kuvaus                                      |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | Polku    | Automaattisesti tunnistettu | Polku taustaprosessin suoritustiedostoon                       |
| `--port`        | Kokonaisluku | 5000          | Taustaprosessin API-portin numero                          |
| `--restart`     | Lippu    | -             | Pakota taustaprosessin uudelleenkäynnistys (lopettaa olemassa olevat prosessit) |
| `--version`     | Lippu    | -             | Näytä versiotiedot ja poistu                |
| `--help`        | Lippu    | -             | Näytä ohjetiedot ja poistu                   |

**Esimerkki globaaleista asetuksista:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## Käsittelyasetusten opas

### Rinnakkaiskäsittely

Chloros+ CLI **skaalaa automaattisesti** rinnakkaiskäsittelyn tietokoneesi suorituskyvyn mukaan:

**Kuinka se toimii:**

* Tunnistaa CPU-ytimet ja RAM-muistin
* Allokoi työntekijät: **2× CPU-ytimet** (käyttää hyperthreading-tekniikkaa)
* **Enintään: 16 rinnakkaista työntekijää** (vakauden takaamiseksi)

**Järjestelmätasot:**

| Järjestelmätyyppi   | CPU        | RAM      | Työntekijät  | Suorituskyky     |
| ------------- | ---------- | -------- | -------- | --------------- |
| **Huippuluokka**  | 16+ ydintä  | 32+ Gt   | Jopa 16 | Maksiminopeus   |
| **Keskitaso** | 8–15 ydintä | 16–31 Gt | 8–16     | Erinomainen nopeus |
| **Alataso**   | 4–7 ydintä  | 8–15 Gt  | 4–8      | Hyvä nopeus      |

{% hint style=&quot;success&quot; %}
**Automaattinen optimointi**: CLI tunnistaa automaattisesti järjestelmän tekniset tiedot ja määrittää optimaalisen rinnakkaisprosessoinnin. Manuaalista määritystä ei tarvita!
{% endhint %}

### Debayer-menetelmät

CLI käyttää oletusarvoisesti ja suositeltuna debayer-algoritmina **High Quality (Faster)** -menetelmää:

| Menetelmä                      | Laatu | Nopeus | Kuvaus                                 |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **Korkea laatu (nopeampi)** ⭐ | ⭐⭐⭐⭐    | ⚡⚡⚡   | Reunat tunnistava algoritmi (oletus, suositeltu) |

### Vignette-korjaus

**Toiminto:** Korjaa valon heikkenemisen kuvan reunoilla (kamerakuvissa yleiset tummemmat kulmat).

* **Oletuksena käytössä** - Useimpien käyttäjien kannattaa pitää tämä käytössä
* Poista käytöstä `--no-vignette`:llä

{% hint style=&quot;success&quot; %}
**Suositus**: Ota vignettikorjaus aina käyttöön, jotta kuvan kirkkauden tasaisuus varmistetaan.
{% endhint %}

### Heijastavuuden kalibrointi

Muuntaa raakakuvausanturin arvot standardoiduiksi heijastavuusprosenttiosuuksiksi kalibrointipaneelien avulla.

* **Oletusarvoisesti käytössä** – välttämätön kasvillisuuden analysoinnissa.
* Vaatii kalibrointikohdepaneelit kuvissa.
* Poista käytöstä `--no-reflectance`:llä.

{% hint style=&quot;info&quot; %}
**Vaatimukset**: Varmista, että kalibrointipaneelit ovat oikein valaistuja ja näkyvissä kuvissasi, jotta heijastavuuden muunnos on tarkka.
{% endhint %}

### PPK-korjaukset

**Toiminto:** Soveltaa jälkikäsiteltyjä kinemaattisia korjauksia DAQ-A-SD-lokitietojen avulla GPS-tarkkuuden parantamiseksi.

* **Oletusarvoisesti pois käytöstä**
* Käytä `--ppk`:ää ottaaksesi käyttöön
* Vaatii .daq-tiedostot projektikansiossa MAPIR DAQ-A-SD -valosensorista.

### Tulostusmuodot

<table><thead><tr><th width="197">Muoto</th><th width="130.20001220703125">Bittisyvyys</th><th width="116.5999755859375">Tiedostokoko</th><th>Sopii parhaiten</th></tr></thead><tbody><tr><td><strong>TIFF (16-bittinen)</strong> ⭐</td><td>16-bittinen kokonaisluku</td><td>Suuri</td><td>GIS-analyysi, fotogrammetria (suositeltava)</td></tr><tr><td><strong>TIFF (32-bittinen, prosentti)</strong></td><td>32-bittinen liukuluku</td><td>Erittäin suuri</td><td>Tieteellinen analyysi, tutkimus</td></tr><tr><td><strong>PNG (8-bittinen)</strong></td><td>8-bittinen kokonaisluku</td><td>Keskikokoinen</td><td>Silmämääräinen tarkastus, verkkopohjainen jakaminen</td></tr><tr><td><strong>JPG (8-bittinen)</strong></td><td>8-bittinen kokonaisluku</td><td>Pieni</td><td>Nopea esikatselu, pakattu tulostus</td></tr></tbody></table>***

## Automaatio ja skriptit

### PowerShell-eräprosessointi

Useiden tietojoukkojen kansioiden automaattinen käsittely:

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

### Windows-eräskripti

Yksinkertainen silmukka eräprosessointia varten:

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

### Python Automaatioskripti

Edistynyt automaatio virheiden käsittelyllä:

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

1. **Syöte**: Kansio, joka sisältää RAW/JPG-kuvapareja
2. **Haku**: CLI etsii automaattisesti tuetut kuvatiedostot
3. **Käsittely**: Rinnakkaistila skaalautuu CPU-ytimien mukaan (Chloros+)
4. **Tulos**: Luo kameramallin alikansiot käsitellyillä kuvilla

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

### Arvioitu käsittelyaika

Tyypillinen käsittelyaika 100 kuvalle (kukin 12 MP):

| Tila              | Aika      | Laitteisto                                     |
| ----------------- | --------- | -------------------------------------------- |
| **Rinnakkaistila** | 5–10 min  | i7/Ryzen 7, 16 Gt RAM-muistia, SSD (enintään 16 työntekijää) |
| **Rinnakkaistila** | 10–15 min | i5/Ryzen 5, 8 Gt RAM-muistia, HDD (enintään 8 työntekijää)   |

{% hint style=&quot;info&quot; %}
**Suorituskykyvinkki**: Käsittelyaika vaihtelee kuvien määrän, tarkkuuden ja tietokoneen ominaisuuksien mukaan.
{% endhint %}

***

## Vianmääritys

### CLI ei löydy

**Virhe:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Ratkaisut:**

1. Tarkista asennuksen sijainti:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Käytä koko polkua, jos se ei ole PATH-polussa:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Lisää PATH-polkuun manuaalisesti:
   * Avaa Järjestelmän ominaisuudet → Ympäristömuuttujat
   * Muokkaa PATH-muuttujaa
   * Lisää: `C:\Program Files\Chloros\resources\cli`
   * Käynnistä terminaali uudelleen.

***

### Backend-palvelun käynnistys epäonnistui.

**Virhe:**

```
Backend failed to start within 30 seconds
```

**Ratkaisut:**

1. Tarkista, onko backend-palvelu jo käynnissä (sulje se ensin).
2. Tarkista, ettei palomuuri estä Windows-palvelua.
3. Kokeile eri porttia:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. Pakota backendin uudelleenkäynnistys:

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### Lisenssi-/todennusongelmat

**Virhe:**

```
Chloros+ license required for CLI access
```

**Ratkaisut:**

1. Varmista, että sinulla on voimassa oleva Chloros+ -tilaus.
2. Kirjaudu sisään tunnuksillasi:

```powershell
chloros-cli login user@example.com 'password'
```

3. Tarkista lisenssin tila:

```powershell
chloros-cli status
```

4. Ota yhteyttä tukeen: info@mapir.camera

***

### Kuvia ei löytynyt

**Virhe:**

```
No images found in the specified folder
```

**Ratkaisut:**

1. Varmista, että kansio sisältää tuettuja tiedostomuotoja (.RAW, .TIF, .JPG).
2. Tarkista, että kansion polku on oikea (käytä lainausmerkkejä poluissa, joissa on välilyöntejä).
3. Varmista, että sinulla on kansion lukuoikeudet.
4. Tarkista, että tiedostotunnisteet ovat oikeat.

***

### Käsittely pysähtyy tai jumittuu

**Ratkaisut:**

1. Tarkista käytettävissä oleva levytila (varmista, että sitä on riittävästi tulostusta varten).
2. Sulje muut sovellukset vapauttaaksesi muistia.
3. Vähennä kuvien määrää (käsittele ne erissä).

***

### Portti jo käytössä

**Virhe:**

```
Port 5000 is already in use
```

**Ratkaisu:**

Määritä eri portti:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## UKK

### K: Tarvitsenko lisenssin CLI:lle?

**V:** Kyllä! CLI vaatii maksullisen **Chloros+ -lisenssin**.

* ❌ Standard (ilmainen) -paketti: CLI pois käytöstä
* ✅ Chloros+ (maksulliset) paketit: CLI täysin käytössä

Tilaa osoitteesta: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### K: Voinko käyttää CLI:ää palvelimella, jossa ei ole graafista käyttöliittymää?

**V:** Kyllä! CLI toimii täysin ilman käyttöliittymää. Vaatimukset:

* Windows Server 2016 tai uudempi
* Visual C++ Redistributable asennettuna
* Riittävä RAM-muisti (vähintään 8 Gt, suositellaan 16 Gt)
* Kertaluonteinen GUI-lisenssin aktivointi millä tahansa koneella

***

### K: Mihin käsitellyt kuvat tallennetaan?

**V:** Oletuksena käsitellyt kuvat tallennetaan **samaan kansioon kuin syötöt** kameramallin alikansioihin (esim. `Survey3N_RGN/`).

Käytä `-o`-vaihtoehtoa määrittääksesi toisen tulostuskansion:

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### K: Voinko käsitellä useita kansioita kerralla?

**V:** Ei suoraan yhdellä komennolla, mutta voit käyttää skriptejä kansioiden käsittelyyn peräkkäin. Katso kohta [Automaatio ja skriptit](CLI.md#automation--scripting).

***

### K: Kuinka tallennan CLI-tuloksen lokitiedostoon?

**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

***

### K: Mitä tapahtuu, jos painan Ctrl+C käsittelyn aikana?

**V:** CLI:

1. Lopettavat käsittelyn siististi
2. Sammuttaa taustaprosessin
3. Sulkeutuu koodilla 130

Osittain käsitellyt kuvat voivat jäädä tulostuskansioon.

***

### K: Voinko automatisoida CLI-käsittelyn?

**V:** Totta kai! CLI on suunniteltu automatisointia varten. Katso [Automaatio ja skriptit](CLI.md#automation--scripting) PowerShell-, Batch- ja Python-esimerkkejä varten.

***

### K: Miten tarkistan CLI-version?

**V:**

```powershell
chloros-cli --version
```

**Tulos:**

```
Chloros CLI 1.0.2
```

***

## Ohjeen saaminen

### Komentorivin ohje

Tarkastele ohjeita suoraan CLI:ssa:

```powershell
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
* **Hinnat**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

## Täydelliset esimerkit

### Esimerkki 1: Peruskäsittely

Käsittely oletusasetuksilla (vignette, heijastavuus):

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### Esimerkki 2: Korkealaatuinen tieteellinen tulos

32-bittinen float TIFF:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### Esimerkki 3: Nopea esikatselukäsittely

8-bittinen PNG ilman kalibrointia nopeaa tarkastelua varten:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### Esimerkki 4: PPK-korjattu käsittely

PPK-korjausten soveltaminen heijastavuudella:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### Esimerkki 5: Mukautettu tulostuspaikka

Käsittely eri asemalle tietyssä muodossa:

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### Esimerkki 6: Todentamisen työnkulku

Suorita todentamisen työnkulku loppuun:

```powershell
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
chloros-cli process "C:\Datasets\Field_A"

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Esimerkki 7: Monikielinen käyttö

Vaihda käyttöliittymän kieli:

```powershell
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
chloros-cli process "C:\Vuelos\Campo_A"

# Change back to English
chloros-cli language en
```
