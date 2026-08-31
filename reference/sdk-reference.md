# Chloros Python SDK Viite

**Versio:**

1.2.0**Luotu:**

29.7.2026 klo 19.19 ·**Päivitetty:**

30.8.2026**Paketti:** `chloros-sdk` (PyPI)**Kohderyhmä:** Optimoitu suurten kielimallien (LLM) käyttöön; ihmisen luettavissa.**Laajuus:** Kaikki `import chloros_sdk`:n tarjoamat julkiset luokat, funktiot ja apufunktiot, sekä kopioitavat esimerkit, jotka kattavat kuvankäsittelyn, yhden kameran ohjauksen, synkronoidut taulukot, DAQ-anturit ja projektin automatisoinnin.

Jos haluat tutustua vain tärkeimpiin kohtiin, siirry kohtaan:
- [Asennus ja pikaopas](#installation)
- [Smart-Connect LATTICE-kameroille](#smart-connect-for-lattice-cameras)
- [DAQ-anturisessiot](#daq-sensor-sessions)
- [Projektiautomaatio](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## Arkkitehtuuri 60 sekunnissa

SDK on ohut Python-kerros Chloros-taustapalvelimen päällä (sama Flask-palvelin, jota työpöytäkäyttöliittymä ja CLI käyttävät). Automaatiota varten tuodaan `chloros_sdk` ja kutsutaan korkean tason metodeja; taustalla jokaisesta kutsusta tulee HTTP-pyyntö paikalliselle taustapalvelimelle portissa 5000 — `http://127.0.0.1:5000/api/...` (tarkoituksella ei `localhost`, joka ohjautuu ensin `::1`:ään osoitteessa Windows ja joka vie noin 2 sekuntia pyyntöä kohti, kun taustapalvelin käyttää vain IPv4-osoitteita). Taustapalvelin hallinnoi laitteistopoolia — kameroita, DAQ-antureita, kohdistusprofiileja, kehyspuskuria — joten SDK-skriptit voivat toimia rinnakkain graafisen käyttöliittymän kanssa ilman, että ne kilpailevat sarjaporttien tai verkkokortin kaistanleveyden käytöstä.

Käytössäsi on kolme käyttöliittymää:

1. **`ChlorosLocal` + ilmaiset funktiot** (`process_folder`, `process_lattice_capture`) — Kuvan käsittelyputki. Käsittele koko kansio kalibroinnin, debayer-muunnoksen ja indeksin viennin kautta yhdellä Python-kutsulla.
2. **Smart-connect-käsittelyt** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Avaa pysyvä taustasessio reaaliaikaista laitteistoa varten. Sama &quot;smart-prep&quot;-prosessi kuin graafisessa käyttöliittymässä: verkkotunnistus, tason automaattinen valinta, PTP, AE-alustus, GPIO-laukaisijan konfigurointi.
3. **`ChlorosProject` / `open_project`** — Lataa tallennettu projekti (kansio, jossa on tiedostot `cameras.json` + `sensors.json` + `project.json`), kytke kaikki kerralla ja aja tallennuksia nimetyillä käsitteillä.

Pinnat 1 ja 2 **käynnistävät paikallisen taustapalvelimen automaattisesti** , ellei sellainen ole jo kuuntelutilassa (sama mukana toimitettu binääri, jonka GUI/CLI käynnistää) — joten pelkkä skripti toimii uudesta komentotulkista ilman, että sinun tarvitsee käynnistää taustapalvelinta ensin. Ohita tämä antamalla `auto_start_backend=False` (esim. kun osoitetaan etätaustapalvelimeen, jota ei koskaan käynnistetä). Katso [Taustapalvelimen automaattinen käynnistys](#backend-auto-start). Surface 3 toimii eri tavalla: `open_project()` ei ota vastaan `auto_start_backend`-parametria, eikä `connect_all()` koskaan käynnistä taustaprosessia — se tunnustaa `http://127.0.0.1:5000`:ää kerran ja, jos mikään ei vastaa, siirtyy hiljaisesti suoraan (taustapalvelimettomaan) `lattice_sdk`-laitteen ohjaukseen. Vain `proj.process()` ja `stream(..., overlays=True)` muodostavat viiveellä `ChlorosLocal()`:n (joka käynnistyy automaattisesti).

Kaikki kolme vaativat todennuksen: suorita `chloros-cli login` kerran koneella tai kirjaudu sisään työpöydän graafisen käyttöliittymän kautta. SDK-kutsut ilman kelvollista istuntoa aiheuttavat virheen `ChlorosAuthenticationError`.

Vaatimukset:
- Python 3.7+ (paketin ilmoituksen mukaan; kehitetty/testattu versiossa 3.10)
- Chloros Desktop asennettuna paikallisesti (taustaprosessi sisältyy asennusohjelmaan)
- Aktiivinen Chloros+ -kirjautumistunnus. SDK / CLI -palvelun vähimmäistaso on **Copper**-taso tai korkeampi (Copper / Bronze / Silver / Gold); ilmaisella**Iron**-tasolla ei ole pääsyä SDK / CLI -palveluihin. Tätä valvotaan**palvelinpuolella**: jokaisessa SDK / CLI -merkityssä pyynnössä on oltava sekä aktiivinen istunto että maksettu tilaus, tai taustapalvelu palauttaa virheen `403` yhdessä virheen `error_code: PLAN_UPGRADE_REQUIRED` kanssa (näkyy muodossa `ChlorosLicenseError` virheen `ChlorosLocal` ja `ChlorosConnectError`-muodossa `connect_*`-aputoimintojen toimesta). Uloskirjautunut kutsuja saa sen sijaan virheen `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — nämä kaksi ovat erillisiä, koska `chloros-cli login`:n uudelleenkäynnistäminen korjaa ensimmäisen, mutta ei toista.
- Offline-käyttöä tuetaan paketinarmonajan puitteissa: käyttöoikeustaso luetaan palvelimen vahvistusvälimuistista (5 min) tai allekirjoitetusta, laitteeseen sidotusta lisenssivälimuistista (30 päivää kuukausitilauksissa, vuositilauksissa tilauksen voimassaolon loppuun asti). Kun tämä armonaika päättyy, tilaus muuttuu ilmaiseksi ja SDK / CLI -käyttö keskeytyy, kunnes laite pääsee kerran yhteyteen palvelimen kanssa. `chloros-cli status` (`GET /api/license-status`) pysyy käytettävissä ilmaisella tasolla, joten syy on näkyvissä — se on ainoa SDK / CLI reitti, joka on vapautettu tasorajoituksesta.
- Windows 10/11 64-bittinen, **Ubuntu 22.04 LTS tai uudempi**, tai Jetson (JetPack 6). Ubuntu 20.04:ää**ei** tueta: `.deb`:n riippuvuudet perustuvat siihen, mihin taustapalvelu linkittyy, mukaan lukien `libc6 (>= 2.34)`, ja Focal toimittaa glibc 2.31:n.

---

## Asennus

Python SDK on ohut Python-kerros Chloros-taustaprosessin päällä. Jos haluat tehdä muutakin kuin muutamia pelkästään DAQ:ta käyttäviä työnkulkuja, tarvitset **paikallisesti asennetun Chloros-työpöytäpaketin** (Windows asennusohjelma tai Linux `.deb`) — se tarjoaa taustapalvelimen binääritiedoston, Arena-SDK-ajoympäristön LATTICE-kameroille sekä kalibrointipaketit.

Uusimmat lataukset: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Vaihe 1 — Asenna Chloros-alustapaketti

#### Windows (.exe)

1. Lataa `Chloros-Setup-x.y.z.exe` lataussivulta.
2. Suorita asennusohjelma ja seuraa ohjatun toiminnon ohjeita. Oletusasennuskansio on `C:\Program Files\MAPIR\Chloros\`.
3. Käynnistä Chloros vähintään kerran ja kirjaudu sisään Chloros+ -tililläsi.

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### Vaihe 2 — Asenna Python SDK

**Chloros-asennusohjelma toimittaa mukana sopivan SDK-wheel-tiedoston.** Jokainen Windows-asennusohjelma ja Linux .deb-tiedosto asentaa levylle `chloros_sdk-X.Y.Z-py3-none-any.whl`-tiedoston, joka vastaa tarkalleen GUI- / CLI / backend-versiota. Sinun ei tarvitse seurata PyPI:tä pysyäksesi synkronoituna.

#### Windows

Asennusohjelma suorittaa automaattisesti `pip install`-komennon mukana toimitetulle wheel-tiedostolle käyttämällä järjestelmänPython-komentoa (`py.exe`-käynnistysohjelma on ensisijainen, mutta se siirtyy varasuunnitelmana `python -m pip`-komentoon). Mitään toimia ei tarvita — `import chloros_sdk` toimii Python-ympäristössäsi asennuksen onnistuttua. Jos koneessa ei ole Python-tiedostoa, asennusohjelma ohittaa tämän vaiheen hiljaisesti, ja graafinen käyttöliittymä sekä CLI jatkavat toimintaansa.

#### Linux (.deb)

.deb-tiedosto sijoittaa wheel-tiedoston kansioon `/usr/lib/chloros/sdk/`. Komento `postinst` tulostaa tarkan komennon — PEP 668 -jakelut estävät oletuksena globaalit pip-kirjoitukset, joten emme asenna ohjelmaa automaattisesti:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Air-gapped-Jetson-asennuksissa tämä tapahtuu täysin offline-tilassa — wheel-tiedosto on jo levyllä.

#### Julkinen PyPI

Pelkästään pip:iä käyttäville isäntäkoneille (Chloros-työpöytäpakettia ei ole asennettu; etätausta- tai pelkästään DAQ-työnkulut):

```bash
pip install chloros-sdk
```

PyPI päivitetään julkaisuversioiden asennusohjelmistojen rakennuksissa, joten julkaistu wheel-tiedosto vastaa uusinta vakaata julkaisua. Kehitysversiot (esim. `1.1.4.dev1`) toimitetaan vain mukana tulevan asennusohjelman wheel-tiedoston kautta.

#### Varmista

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Vaaditaan Chloros+ -tilaus.** Kaikki SDK-kutsut edellyttävät aktiivista Chloros+ -kirjautumista. Suorita `chloros-cli login user@example.com 'YourPassword'` kerran kutakin konetta kohti; kirjautumistiedot tallennetaan välimuistiin `~/.chloros/`:ssä.

### Tarvitsenko työpöytäpaketin?

Pelkkä pip-paketti **ei** riitä useimpiin työnkulkuihin. Tässä on luettelo siitä, mitä kukin SDK-pinta tarvitsee:

| SDK-pinta | Tarvitaanko työpöytäpakettia? | Miksi |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Kyllä** | Käynnistää taustaprosessin automaattisesti `/usr/lib/chloros/chloros-backend`:ssä (Linux) tai `C:\Program Files\MAPIR\Chloros\…`:ssä (Windows). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Kyllä**(paikallinen)**/ Ei**(etä) | Pelkät HTTP-asiakasohjelmat taustapalvelimen kautta. Paikallinen taustapalvelin → vaatii työpöytäpaketin. Etätaustapalvelin → `backend_url=`**tunnelin kautta** (katso Etätaustapalvelintila — mukana toimitetut taustapalvelimet sitovat vain loopback-liitännän). |
| `ChlorosProject` / `open_project` | **Kyllä** | Ajaa tallennettuja projekteja taustapalvelimen kautta. |
| Suorat LATTICE-luokat (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Kyllä** | Tarvitsee Arena-SDK-natiiviruntimea, joka toimitetaan työpöytäpaketin mukana. Muussa tapauksessa `CAMERA_AVAILABLE` on tuonnin yhteydessä `False`. |
| Suorat DAQ-luokat (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **Ei** | Puhdas Python pyserial/bleak/zeroconf:n päälle. Pelkästään pip-ympäristö voi ohjata DAQ-laitteita päästä päähän. |

### Etätaustapalvelintila (vain pip-isäntä, tunnelin kautta)

> **Toimitettua taustapalvelinta ei voi saavuttaa lähiverkon kautta.** Tuotantoversio
> -rakennukset sitovat vain loopback-liitännän (molemmat loopback-perheet) ja hylkäävät jyrkästi
> ainoan ei-loopback-tilan (`CHLOROS_CLOUD_MODE`), joten
> `backend_url="http://<lan-ip>:5000"` **ei voi toimia asennettua
> Chloros:n kanssa** — kyseinen malli on toiminut ainoastaan source/dev-
> taustapalvelimen kanssa. Jos haluat käyttää toisella koneella sijaitsevaa taustapalvelinta, ohjaa sen loopback-
> portti itse ja osoita SDK tunneliin:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

Headless- / CI- / robotiikkakoneet voivat pitää yhden koneen, johon on asennettu täysi työpöytäympäristö, ”Chloros-palvelimena” ja käyttää `pip install chloros-sdk`:ää kaikkialla muualla — mutta niiden välinen siirto tapahtuu yllä mainitun käyttäjän järjestämän tunnelin kautta, ei suoran LAN-URLin kautta.

> **Tunnettu rajoitus — `ChlorosLocal` ei tue pelkästään pip-käyttöä.** `ChlorosLocal(backend_url=BACKEND)` etsii tällä hetkellä paikallisen taustapalvelun binääritiedoston konstruktorissaan *ennen* kuin se tarkistaa URL-osoitteen, ja aiheuttaa virheen `ChlorosBackendError` (&quot;Chloros-taustapalvelinta ei löydy…&quot;), jos työpöytäpakettia ei ole asennettu — vaikka etätaustapalvelin olisi tavoitettavissa. Vain yllä oleva smart-connect-käyttöliittymä (`connect_camera` / `connect_array` / `connect_daq_sensor` sekä `analyze_array_network` sekä apuohjelmat `list_*` / `discover_*`) toimivat pelkän pip-isäntäkoneen kautta.

### Pelkkä DAQ-työnkulku (vain pip-isäntä)

Jos tarvitset vain DAQ-antureita etkä käytä LATTICE-kameroita tai kuvankäsittelyä, pip-paketti on itsenäinen:

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

Ei taustapalvelinta, ei .deb-pakettia, ei Chloros+ -kirjautumista vaadita suorassa laitteistopohjaisessa DAQ-työssä.

---

## Pikaopas

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## API-hakemiston ylätaso

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## Kuvan käsittely — `ChlorosLocal`

Pääputkiluokka. Käynnistää taustaprosessin ensimmäisellä käyttökerralla, luo ja määrittää projektit, valvoo edistymistä ja palauttaa suorituksen jälkeiset yhteenvedot.

### Konstruktori

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Menetelmät

| Menetelmä | Kuvaus |
| --- | --- |
| `create_project(project_name, camera=None)` | Luo uuden projektin (valinnaisesti kameramallin, kuten `"Survey3N_RGN"`, avulla). |
| `import_images(folder_path, recursive=False)` | Tuo RAW/TIF/JPG/DNG-kuvia **ja `.daq`-valosensoritallenteita**. Palauttaa `count` (kuvat) ja `scan_count` (tallenteet). Antaa varoituksen vain, jos kansiossa ei ole kumpaakaan. |
| `export_light_sensor(daq=True, csv=True)` | Kirjoittaa kalibroidut `.daq` + `.csv` jokaisesta projektin valosensoritallenteesta tiedostoon `<project>/Light Sensor/`. Katso [Valosensoritallenteet](#light-sensor-tallenteet--kalibroitu-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Aseta käsittelyn säätimet. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Suorita käsittelyputki. Palauttaa `{"status": "complete", "async": False}`:n, sekä `summary`-avaimen, jos taustapalvelu tarjoaa sellaisen — katso [Suorituksen jälkeinen yhteenveto ja vinkit](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | Tarkista taustapalvelimen tila. |
| `logout()` | Tyhjennä välimuistissa olevat tunnistetiedot. |
| `shutdown_backend()` | Lopeta taustapalvelu (jos SDK -started). |
| `discover_cameras()` | Etsi LATTICE-kamerat **tämän instanssin taustapalvelimen kautta** (`/api/camera/discover`). Palauttaa sanakirjojen luettelon (`serial`, `model`, `ip`, …) — sama rakenne kuin GUI:ssa/ CLI. Tyhjä luettelo, jos kameroita ei löydy tai taustapalvelimeen ei saada yhteyttä. |
| `camera_capture(output_dir, format="tiff", **settings)` | Tallenna yksi kuva**taustapalvelimen kautta**(käynnistyy automaattisesti tämän käsittimen avulla), jolloin se saa saman esikäsittelyn kuin GUI/ CLI (oletusarvo 12 bittiä, poolin uudelleenkäyttö, upotetut kalibrointimetatiedot). Määritä kohde komennolla `serial=` tai `device_index=`; välitä `exposure`/`gain`/`pixel_format`/`preset` arvona `**settings`. Palauttaa vanhan metatietosanakirjan (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Tuottaa päällekkäisistä kuvista koostettuja esikatselukuvia yhdistetyistä kameroista — kevyt MJPEG-asiakasohjelma taustapalvelimen `/api/camera/<serial>/stream-annotated`-reitin kautta (zebra / ruudukko / ristikohdistin / histogrammi / peaking / piste, jotka on piirretty palvelinpuolella). `decode=True` tuottaa BGR-taulukoita; `False` tuottaa raakamuotoisia JPEG-tavuja. Saatavilla myös projektikohtaisesti nimellä `ChlorosProject.stream(overlays=True)`. |

Käytä kontekstinhallintana taatun puhdistuksen varmistamiseksi:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Valosensorin tallenteet — kalibroitu `.daq` + `.csv`

DAQ-U / DAQ-M / DAQ-E voidaan tallentaa **ilman** sen kalibrointipakettia. Juuri
tämän julkiset [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
tallentimet (`record_daq.py`) tekevät oletuksena: ne tallentavat anturien raakalukemat ja merkitsevät
tiedostoon leiman, jotta Chloros hakee kyseisen anturin tehdaskalibroinnin **sarjanumeron perusteella** — ensin paikallisesta välimuistista
, sitten MAPIR-pilvestä — ja soveltaa sen tuonnin yhteydessä.

Chloros tallentaa tuloksen takaisin kahdeksi tuotteeksi tallennusta kohti, nimikkeellä
`<project>/Light Sensor/`:

| Tuote | Mikä se on |
| --- | --- |
| `<name>_calibrated.daq` | Uudelleenkäsiteltävä arkisto — sama rakenne kuin reaaliaikaisessa tallenteessa, mutta nyt ilmoitetaan sen tuottanut paketti. Sen uudelleen tuominen **ei** kalibroi sitä toista kertaa. |
| `<name>_calibrated.csv` | Spektrinen säteilyvoimakkuus yksiköissä W/m²/nm anturin omalla aallonpituusruudukolla, yksi rivi lukemaa kohti, sekä fotometriset sarakkeet (kokonaisteho, fotopinen/skotopinen lux, PPFD ja sen sininen/vihreä/punainen jakautuma, huippuaallonpituus). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Vain niputtamattomat anturit (DAQ-A).** Anturin raakaspektrilukemat — *ei* säteilyvoimakkuutta. Katso alla. |

`process()` suorittaa tämän viennin yhtenä vaiheistaan. Se **ei** vaadi kuvamateriaalia:
yksin lennetty valosensori on ensiluokkainen työnkulku, ja tällaisessa projektissa ei ole lainkaan
kuvia rakenteensa vuoksi.

**DAQ-A-tallenteet viedään raakalukemina.** DAQ-A-tuoteperhe on peräisin ajalta ennen sarjanumeroittain
toimivaa niputusjärjestelmää, eikä sille ole noutettavaa niputusta — se kalibroidaan kentällä
heijastavuuskohteeseen, minkä vuoksi sille ei ole koskaan tarvittu nimeä. Nämä tallenteet viedään
tiedostonimellä `_raw` eikä `_calibrated`: tiedostonimellä eikä tunnisteella
tiedoston sisällä, koska tiedoston nimi on säilytettävä sellaisenaan, kun se lähetetään sähköpostitse.
`.csv`-otsikossa lukee `raw spectral sensor counts (NOT irradiance)` ja varoitetaan, että
arvot ovat vertailukelpoisia **tiedoston sisällä** tiedoston sisällä — juuri siihen tarkoitukseen, johon kohdepohjainen kalibrointi
niitä käyttää — eivätkä anturien välillä. Tehoa riippuvat fotometriset sarakkeet (kokonaisteho,
fotopinen/skotopinen lux, PPFD) palauttavat arvon **NULL** sen sijaan, että ne integroitaisiin laskentojen perusteella.

DAQ-U / DAQ-M / DAQ-E, jonka pakettia ei yksinkertaisesti voitu hakea, **ohitetaan** edelleen,
eikä sitä kirjoiteta raakamuodossa: siinä tapauksessa paketti on olemassa ja ”yhdistä uudelleen ja käsittele uudelleen” on oikea neuvo.

Vanhat **v1.01 / v1.02**-tallenteet (DAQ-A-SD kirjoittaa näitä) eivät sisällä lukukohtaista aikakohdetta,
vain tiedoston kirjoitusajan. Image↔downwelling-vastaavuustarkistaja hylkää ne edelleen —
kehyksen vertaaminen kirjoitusajankohtaan olisi huomaamattomasti virheellistä — mutta viejä lukee ne, ja
CSV tulostaa `clock=daq_created_on`, joten tuote ilmoittaa, millä kellolla se toimii.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Tallenne, jonka kalibrointipakettia ei voida hakea (offline-tilassa tai anturi, jolla ei ole
kalibrointitietoja tiedostossa), raportoidaan koodilla `skipped` **syyn kera**. Sitä ei koskaan
tallenneta ”kalibroituna” tiedostona, joka sisältää raakalukemia — muodosta yhteys internetiin ja
suorita uudelleen, jolloin vienti valmistuu.

### Edistymiskutsut

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Suorituksen jälkeinen yhteenveto ja vinkit

Valmistuessaan `process()` hakee `GET /api/processing-summary`:n ja liittää rungon tiedostona `result["summary"]`. Hakeminen on parasta-ponnistusta eikä se koskaan estä onnistunutta paluuta — jos yhteenvetoa ei ole saatavilla, `process()` siirtyy takaisin tavalliseen `{"status": "complete", "async": False}`-muotoon. Jokainen merkintä `summary["hints"]`:ssä — kokonaiset lauseet, joissa on ehdotettu korjaus, esim. miksi suoritus tuotti nollatuloksen — lähetetään uudelleen myös muodossa Python `UserWarning`, joten nollatulokselliset suoritukset ovat itsediagnostikoivia, vaikka sanakirjaa ei koskaan tarkasteltaisi:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` on koneellisesti luettava puoli:

| Avain | Mitä se laskee |
| --- | --- |
| `models` | Suorituksen kameraryhmät. |
| `images_in_groups` | Näiden ryhmien lähdekuvat. |
| `targets_found` | Havaittuja heijastavuuskohteita. |
| `images_calibrated` | Ajojen kalibroimat kuvat. |
| `exported_files` | **Ajojen luomat kuvatuotetiedostot.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | Valosensorin tallenteet, jotka on tarkoituksella laskettu erikseen — ne ovat peräisin eri vaiheesta ja niitä on myös ajoissa, joissa ei ole lainkaan kuvamateriaalia, joten niiden sisällyttäminen saisi pelkän DAQ-ajon näyttämään siltä, kuin se olisi vienyt kuvia. |

Niiden lisäksi: `summary["output_dirs"]` (jokainen hakemisto, johon on kirjoitettu),
`summary["light_sensor_export"]`, `summary["stopped"]` (tosi, kun käyttäjä keskeytti
ajon, jotta osittaiset laskelmat eivät tulkitu suoritetuksi ajoksi, jossa tuotanto jäi vajaaksi) ja
`summary["groups"]` (ryhmäkohtainen erittely).

`exported_files` tallennetaan prosessiketjussa **kirjoituksen yhteydessä**, eikä sitä skannata
projektin kuvaobjekteista jälkikäteen. Rinnakkais- ja GPU-strategiat rakentavat omat kuvaobjektinsa
(GPU-polkujen tapauksessa työntekijöiden aliprosesseissa), joten vanha skannaus raportoi
`0 file(s) written` jokaisesta tällaisesta ajosta ja lähetti sitten nollaviennin vihjeen — ajoissa,
joissa kaikki oli toiminut. Jos kirjoitat skriptin tämän numeron perusteella, toimiva rinnakkaiskäyttö
ilmoittaa nyt nollasta poikkeavan lukeman.

Ohitusten raportit ilmoittavat syyn, jonka lukija tosiasiallisesti määritteli jokaiselle tiedostolle –
lukukelvoton skeema, puuttuva nippu, kirjoitusvirhe – **päällekkäisyydet poistettuina**, joten kaksikymmentä tiedostoa,
jotka ohitettiin yhden syyn vuoksi, näkyvät yhtenä syynä sen sijaan, että syy toistettaisiin kaksikymmentä kertaa.

> **`process()` ei laukea, kun ajo ei tuota kuvia.** Tämä on ainoa kohta, jossa SDK ja
> CLI eroavat toisistaan tarkoituksellisesti: `chloros-cli process` käsittelee &quot;tuotteita pyydettiin, mutta yhtään ei
> kirjoitettu” -tilannetta virheenä ja lopettaa nollasta poikkeavalla tuloksella, kun taas SDK palaa normaalisti ja ilmoittaa
> tilanteesta `summary` / hints -viestien kautta. Jos putkistosi pitäisi pysähtyä tyhjän ajon yhteydessä, tarkista se
> itse — tarkista `summary` (tai laske projektikansiossa olevien tiedostojen määrä) sen sijaan, että luottaisit
> poikkeuksen puuttumiseen. Yleisiä syitä ovat syöttökansio, jota ei tunnistettu
> tallennukseksi, sekä tuotteet, jotka ohitettiin, koska niitä ei voitu soveltaa käytettävissä oleviin kameroihin (esim. radiance RGB-
> kameroista).

### Aputoiminnot

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### Tuetut arvot

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Radiometrinen tulos (LATTICE-monispektrinen prosessiketju)

`process`-prosessiketjun LATTICE-monispektrinen (M3C/M3M) vientitaso — `reflectance` (oletus), `radiance`, `sensor-response` tai `all` (kaikki kuvakohtaiset soveltuvat tilat) — vastaa projektin **&quot;Radiometrinen tuloste&quot;** -käsittelyasetukseen. `configure()`:llä on oma avainsanansa:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Edistynyt kiertotapa — projektin `"Radiometric output"`-avaimen kirjoittaminen `custom_settings`:n kautta — toimii edelleen, mutta muista, että se korvaa koko asetuslohkon (katso alla oleva varoitus):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (oletus) jakaa kameran säteilyvoimakkuuden **aikaleimalla täsmätyllä DAQ-alasvirtauksella**, joka määritetään automaattisesti tallennetusta `.daq` (DAQ-U/M/E)**tai kuvamateriaalin yhteydessä löytyvästä DAQ-M:n omasta `.csv`**; paikallisesti puuttuvat kamera- tai DAQ-kalibrointipaketit**ladataan automaattisesti AWS:stä** ensimmäisellä käyttökerralla. CLI esittää tämän tuotetyyppikohtaisina kytkiminä tiedostossa `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> `custom_settings` **korvaa** koko lasketun asetuslohkon (se ohittaa suunnittelunsa mukaisesti `configure()`:n muut avainsanat ja validoinnin). Kun käytät sitä, sisällytä kaikki sinulle tärkeät `Project Settings`-avaimet, kuten yllä olevassa esimerkissä.

---

## Smart-Connect LATTICE-kameroille

Pysyvät taustaprosessisessiot reaaliaikaiselle laitteistolle. Samat päätepisteet kuin GUI:ssa, joten toiminta on identtinen osoitteissa SDK / CLI / GUI.

### Yksi kamera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

#### `connect_camera()`-allekirjoitus

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

#### `CameraSession`-menetelmät

| Menetelmä | Kuvaus |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Lue GenICam-solmut; palauttaa `{nodes, errors, enums, device}`. |
| `set_settings(**kwargs)` | Kirjoita solmut ystävällisellä nimellä (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | Tallentaa **yhden** kehyksen. Palauttaa yhden elementin sisältävän luettelon, jossa on kehyksen metatietosanakirjoja. (Sarjakuvaus/usean kehyksen tallennus on poistettu — kutsu `capture()` silmukassa, jos tarvitset sarjaa.) |
| `disconnect()` | Vapauttaa resurssin poolista. Ei tee mitään, jos olemme liittäneet jo avoimeen istuntoon. |

`capture()`-vientiasetukset (sama malli kuin taulukko + käyttöliittymä):

- `processing` / `levels` — `processing="all"` tallentaa kaikki soveltuvat vientityypit; `levels=["raw","radiance"]` tallentaa vain ne (ohittaa `processing`:n). Jätä molemmat pois, jos haluat käyttää taustapalvelimen oletusasetusta.
- `force_daq=True` — tallentaa määritetyn DAQ/DLS-lukemaa `.daq`-sidecarina jopa pelkän raakadatan tallennuksessa, jotta kehys voidaan myöhemmin käsitellä uudelleen heijastavuudeksi/indeksiksi. Ei toimi, jos DAQ:ta ei ole liitetty.

### Synkronoitu matriisi — `ArraySession` (Smart-Prep)

`connect_array` on **suositeltu lähtökohta** monikamerajärjestelmille. Se suorittaa taustalla koko GUI-pohjaisen Smart-Prep-prosessin:

1. **Verkkoanalyysi** (`/api/camera/array/recommend`) — etsii suurimman kehyskoko, joka mahtuu sim-emit-tasoon ilman kehysten menetystä.
2. **Tason automaattinen valinta** — `sim-capture-sim-emit`, jos kaapeliyhteys kestää sen; muussa tapauksessa `sim-capture-ftd-stagger` tai `slip-emit-and-capture`.
3. **Automaattinen pienentäminen**— pienentää kehyksen kokoa / lisää binningiä huomaamatta, kun kaapeli ei kykene ylläpitämään pyydettyä resoluutiota.**Tämä turvaverkko ei kata aggregoitua ylimerkintää**: kaapelille liian monia kameroita ei voida korjata pienentämällä kehyksiä — katso [Ylimerkintä](#over-subscription-the-per-cam-floor).
4. **PTP käytössä** oletuksena — kameroiden väliset aikaleimat ovat vertailukelpoisia mikrosekunnin tarkkuudella.
5. **Kamerakohtainen pikselimuodon automaattinen valinta** — RGB-kamerat → `BayerRG8`, multispec-kamerat → `BayerRG12`.
6. **AE-alustaminen** — tallentaa kunkin kameran nykyisen AE-tilan, jotta yhteyden muodostaminen ei nollaa valotusta kesken toiminnan.
7. **GPIO-laukaisun konfigurointi** — `connect_array` aktivoi kaikki kamerat (`TriggerMode=On`, `TriggerSource=Line2`), jolloin isäntälaitteen pulssi ohjaa orjakameroita M8-kaapelin kautta. Tämä on vain taulukoille tarkoitettu vaihe: yksittäinen kamera, joka avataan komennolla `LatticeCamera`, toimii sen sijaan itsenäisesti.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### `connect_array()`-allekirjoitus

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`force_tier`-arvot:
- `"sim-capture-sim-emit"` — todellinen samanaikaisuus (kaikki kamerat laukeavat samalla kelloreunalla).
- `"sim-capture-ftd-stagger"` — joustava aikatasoinen porrastus (kammiot lähettävät hieman viiveellä, jolloin paketit sarjoituvat siirtoyhteydellä).
- `"slip-emit-and-capture"` — peräkkäinen kaappaus kammiota kohti (ei ajallista synkronointia; ainoa vaihtoehto, kun mikään kehyskoko ei sovi simulaatioon).

`wire_ceiling_mbps` ohittaa **isännän jatkuvan siirtokapasiteetin** megabitteinä sekunnissa (MB/s) — se on se yksi
luku, josta koko matriisin allokointi riippuu. Jätä se `None`-asetukseksi, jotta käytetään automaattisesti tunnistettua
arvoa. Pienennä sitä, kun matriisi ilmoittaa GVSP-vioittuneista kehyksistä: automaattinen arvo johdetaan
NIC:n ilmoittamasta linkkinopeudesta, joka yliarvioi USB-sovittimet, kapeat PCIe-kaistat ja
ruuhkautuneita jaettuja verkkoarkkitehtuureja — ja tämä yliarviointi ilmenee vioittuneina kehyksinä sen sijaan, että se näkyisi
näkyvästi hitaana yhteytenä. Arvo tallennetaan projektin matriisin tallennuslohkoon, joten
uudelleen avaaminen tai myöhempi `connect_array` palauttaa sen kuten minkä tahansa muun matriisin asetuksen.
Katso [Array Health](#array-health--which-subsystem-is-losing-frames).

#### Ylimerkintä (kameraa kohti laskettu alaraja)

Sim-emit-tahtaus jakaa jokaiselle kameralle osuuden törmäysturvallisesta kaistanleveysbudjetista, jonka alaraja on **8 MB/s kameraa kohti**(`per_cam_floor_bps`). Kun `N × floor` ylittää törmäysturvallisen ylärajan, matriisi**ylimerkitsee kaistan**— vikamoodina on GVSP-pakettihäviö, ei alhaisempi kuvataajuus — eikä keino korjata kehyskoon ongelmaa:**binning ja ROI pienentävät tavuja kehystä kohti, eivätkä ne vaikuta tasoitetun tiedonsiirron tavumäärään sekunnissa**, jota kokonaisvalvonta vertaa. Käytännön täysresoluutioiset ylärajat 1 GbE -isäntäkoneella:**6 kameraa @ 1500 MTU, 9 jumbo-kehyksillä** (`max_cams_collision_safe` analyysivastauksessa ilmoittaa kaapelisi ylärajan). Korjaustoimenpiteet: vähemmän kameroita, jumbo-kehykset päästä päähän tai nopeampi verkkokortti.

- Vastaukset `analyze_array_network()` ja `/api/camera/array/connect` sisältävät `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` ja `per_cam_floor_bps`. Kun `oversubscribed` on totta, projektio **nollaa fps-kentät** (`achievable_fps_max` / `fps_bright` / `fps_dark`) sen sijaan, että se ilmoittaisi harhaanjohtavan, hitaan mutta toimivan nopeuden.
- `POST /api/camera/array/connect` hyväksyy `pin_resolution`-runkoparametrin (**vain HTTP — ei SDK-kwargia**; `connect_array` ei paljasta sitä). Kiinnittäminen poistaa binning-laskukierroksen turvaverkon, joten ylimerkitty yhteys, jossa `pin_resolution` on asetettu,**hyläetään ehdottomasti** ja virheilmoituksessa mainitaan kaikki korjaustoimenpiteet. Ilman kiinnitystä yhteyden muodostaminen etenee binning-menetelmän mukaisesti, mutta järjestelmä varoittaa, että pienentäminen ei riitä kokonaismäärän tyhjentämiseen.
- Testausympäristön kiertotapa: aseta `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` taustapalvelimen ympäristöasetuksiin, jolloin hylkäys muuttuu voimakkaaksi varoitukseksi — yhteys muodostetaan silti ja pakettihäviö hyväksytään.

#### Taulukon kunto — mikä alijärjestelmä menettää kehyksiä

`GET /api/camera/array/<array_id>/capability` sisältää aktiivisen `health`-lohkon
kytketyssä massassa, joka arvioidaan uudelleen **10 sekunnin** pituisessa liukuvassa ikkunassa. Se jakaa kehysten menetyksen
kahteen syyhyn, jotka vaativat vastakkaisia korjauksia, sen sijaan että käytettäisiin yhtä ”epätäydellisyysastetta”, joka
ei nimeä kumpaakaan:

| Kenttä | Mitä se tarkoittaa | Mikä alijärjestelmä |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (sarjaliitännän mukaan) | Kehys **saapui, mutta oli rakenteellisesti viallinen**— GVSP-paketin menetys. |**Verkko**: kaapelibudjetti, tahdistus, NIC RX-rengas, MTU |
| `never_arrived_rate_pct` (sarjanumeroittain) | Kehystä **ei saapunut lainkaan**— kamera ei lauennut tai siitä ei lähetetty mitään. |**Laukaisin / synkronointi**: M8-kaapeli, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Kunkin kameran huonoin lähetysnopeus. | — |
| `per_cam_rate_pct` | Yhdistetty epätäydellisyysaste kameraa kohti (molemmat syyt yhdessä). | — |
| `stable_for_seconds` | Kuinka kauan kukin kamera on pysynyt alle 0,01 %:n. | — |

`health`:n ohella sama tietue ilmoittaa koko allokoinnin viiveen:

| Kenttä | Merkitys |
| --- | --- |
| `wire_ceiling_mbps` | Isäntäkoneen voimassa oleva jatkuva kaistanleveysbudjetti, MB/s. |
| `wire_ceiling_source` | Mistä kyseinen luku on peräisin, sanoin — esim. `USB-capped 200 MB/s (was theoretical 1062; …)` tai `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, kun `wire_ceiling_mbps=` on asettanut sen. |
| `nic_is_usb` | `true` USB-Ethernet-sovittimelle. |

Tälle päätepisteelle ei ole SDK-käärettä — lue se suoraan:

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Lukeminen:** nollasta poikkeava `gvsp_corrupt_rate_pct`, jossa `never_arrived_rate_pct` on 0, tarkoittaa, että
liipaisu ja kaapelisynkronointi ovat moitteettomia ja 100 % häviöstä johtuu verkkoreitistä — pienempi
`wire_ceiling_mbps`X ja muodosta yhteys uudelleen. Päinvastainen kuvio viittaa sen sijaan synkronointikaapeliin tai
laukaisulinjaan.

> **`target_fps` ei ole syy vioittuneisiin kehyksiin.** GevSCPD-taajuus määritetään kerran
> yhteyden muodostuksen yhteydessä, joten laukaisutaajuuden laskeminen muuttaa käyttöjaksoa, ei
> samanaikaisten lähetysten pursetaajuutta. Mitattu 5-kertainen kysynnän leikkaus ei tuottanut parannusta, kun taas
> kaapelirajan laskeminen 240:stä 200 MB/s:iin laski saman laitteiston virheiden määrän 10,4 %:sta
> 0,00 %:iin.

> **TRI032S-laiteohjelmistossa ei ole käytettävissä lähetysvirran automaattista supistusta.** Käynnissä oleva ryhmä ei voi
> korjata tätä itse; katkaise yhteys ja muodosta se uudelleen, jotta yhteyden muodostusajankohdan valitsija suunnittelee uudelleen
> uuden ylärajan perusteella.

**USB-Ethernet-sovitin on rajoitettu 200 MB/s:iin** mittauslaitteen toimesta riippumatta sen
tyyppikilvestä: tehokkuustaulukko, joka muuntaa linkkinopeuden jatkuvaksi arvoksi, on
johdettu PCIe:stä, ja USB-verkkokortti ilmoittaa Ethernet-linkkinopeutensa, mutta sitä rajoittaa
USB-väylä ja sen ohjain. Yläraja on absoluuttinen, ei suhteellinen — USB 1 GbE -sovitin
saavuttaa ~80 MB/s, eikä se ole tämän vaikutuksen alainen.

#### `ArraySession`-menetelmät

| Menetelmä | Kuvaus |
| --- | --- |
| `status(timeout=10.0)` | Live `{fps, ptp, frame_count, last_error, …}`. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | Yksi synkronoitu kaappausryhmä. Palauttaa `CaptureResult` (luettelo kehyssanakirjoista + `.skipped`). Vientiä koskevat ohjeet alla. |
| `capture(..., smart=True)` | **Älykäs tallennus** — odottaa, että AE vakiintuu kaikissa kameroissa, ja laukaisee sitten. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Nopein tallennus: vain raakadata + määritetty DAQ-lukema (+ vapaa yhdistetty indeksi). Vastaa käyttöliittymän &quot;Nopein tallennus&quot; -painiketta. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Yksittäinen / Jatkuva / Intervalli yhdessä rajatussa silmukassa. Palauttaa `list[CaptureResult]`.**Vaatii `count`:n ja/tai `duration_s`:n**, jotta se päättyy (SDKissa ei ole Ctrl+C-näppäintä). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Aloita yhdistettyjen indeksien reaaliaikaisen näkymän tallennus videoksi/GIF-tiedostoksi → `RecorderHandle`. Yksi yhdistelmätallennin kutakin taulukkoa kohti. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Aloita korkean-fps:n raaka-Bayer-sarjakuvauksen → `RecorderHandle`. Käsittele uudelleen offline-tilassa komennolla `build_video()`. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Käsittele tallennettu raakasarjakuva uudelleen offline-tilassa kalibroiduksi videoksi(s). Estää jatkotoiminnot, kunnes käsittely on valmis (`wait=True`), ja palauttaa `{outputs, errors, combined}`. |
| `build_video_status(job_id, timeout=15.0)` | Tarkista offline-rakennustyön tila: `{running, result, error, burst_dir}`. |
| `disconnect()` | Vapauta koko taulukko. |

`capture()`-vienti hallitaan (sama päätepiste kuin GUI:ssa / CLI):

- `processing` / `levels` — `processing="all"` (tai `levels=["raw","radiance",…]`) tallentaa jokaisen soveltuvan vientityypin kameraa kohden; yksi `processing`-arvo tallentaa vain kyseisen tason.
- `aligned=True` — muuntaa jokaisen jäsenen ei-raakamuotoisen viennin taulukon [kohdistusprofiilin](#array-alignment) (yhteisrekisteröity); raakamuotoiset tiedot jäävät muuntamattomiksi, mutta niiden metatiedoissa on mukana muunnos. Jos taulukolla ei ole profiilia, palataan kohdistamattomaan tilaan (ja tuloksessa näkyy varoitus`alignment`), jos matriisilla ei ole profiilia.
- `render_index=False` — ohita kamerakohtainen kasvillisuusindeksin peittokuva; oletusarvoisesti se renderöidään, jos se on määritetty.
- `force_daq=True` — tallentaa määritetyn DAQ/DLS-lukeman `.daq`-sidecarina, vaikka mikään valittu taso ei sitä tarvitsisi.

**TIFF-pakkaus (vain HTTP-säädin):**`ArraySession.capture()` ei lähetä `compression`-avainta, joten taustapalvelimen oletusasetus on voimassa — `POST /api/camera/array/capture` lukee `compression`-runko-parametrin, oletuksena `"deflate"` (häviötön zlib L1 + vaakasuuntainen ennustaja, ~4,1 MB per täysresoluutioinen kehys). `"none"` kirjoittaa pakkaamattomana (~6,3 MB/kehys)**~5× nopeammalla kirjoitusnopeudella** — molemmat ovat häviöttömiä ja luetaan identtisesti tuonnin yhteydessä. SDK ei tarjoa sille kwarg-parametria; kiertotie on `chloros-cli lattice array-capture --compression none` tai raaka HTTP. DEFLATE pitää myös Python GIL:iä, joten pakattuja kirjoituksia ei voida rinnakkaistaa perkameraa kohti — jatkuva 8-kameran täysresoluutioinen tallennus anturin taajuudella vaatii `compression: "none"`:ää. Lisätietoja: [CLI Viite → array-capture](cli-reference.md).**Jokaisenjäsenkohtaiset vientiohitukset (vain HTTP):**sama päätepiste hyväksyy myös `exclude_serials` (lista — poista jäseniä tallennetusta joukosta; taulukko laukeaa edelleen yhtenä synkronoituna ryhmänä ja pois jätetyt jäsenet palautetaan `excluded`:ssä), `serial_levels` (`{serial: [level tokens]}` kamera-kohtaiset ohitukset) sekä `serial_index` (`{serial: bool}` kamerakohtaiset indeksi-päällysteen ohitukset). Nämä ovat GUI-pariteettisia runkoparametreja, ja**eivät ole vielä SDK-avainarvoja**; karttoista puuttuvat jäsenet käyttävät oletuksena koko taulukon kattavia `levels`- ja `render_index`-arvoja.

##### Ohitettujen kameroiden tarkastelu — `CaptureResult.skipped`

`ArraySession.capture()` palauttaa `CaptureResult`:n, joka on `list`:n alaluokka: kierrä sitä, indeksoi sitä, `len()` sitä — kaikki olemassa olevat mallit toimivat edelleen. Uusi koodi voi tarkistaa `.skipped`-attribuutin nähdäkseen, mitkä kamerat on jätetty pois ja miksi. Yleisin tapaus on RGB kamerat sekasuodatinmatriisissa, kun pyydät `processing="radiance"`- tai `"reflectance"` — Bayer-kohtainen säteilyvoimakkuus on merkityksetön laajakaistaiselle anturille, joten taustaprosessi ohittaa kyseiset kamerat sen sijaan, että tuottaisi järjetöntä tietoa.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Syytunnisteet noudattavat mallia `<level>-not-applicable-to-rgb-cam` (yksi merkintä ohitettua tasoa kohti, joista jokainen sisältää `level`). Heijastavuuteen liittyvät ohitukset ovat `reflectance-skipped-no-fresh-dls` (uutta alaspäin suuntautuvaa lukemaa ei saatavilla), `reflectance-skipped-bound-daq-unavailable (…)` (sidottua DAQ-laitetta ei voitu tavoittaa) ja `dls-uncalibrated-band-<nm>` — kaista sijaitsee pääosin DAQ-valosensorin radiometrisesti kalibroidun alueen ulkopuolella (~374–974 nm), joten absoluuttinen DAQ-pohjainen heijastusjakaja hylätään ja kehys siirtyy selvästi anturin vasteeseen. Toimitettavista tuotetunnuksista vain F988 laukaisee tämän; kyseisen kameran tuettu polku on heijastuspaneelin työnkulku.

`processing`-tasot:

| Taso | Lähtö |
| --- | --- |
| `"raw"` | Yksikanavainen Bayer (mustavalkokamerat: yksikaistainen) suoraan anturista. |
| `"debayered"` *(SDK oletus)* | 3-kanavainen BGR bilineaarisen demosaikin kautta (mustavalkokamerat: 1-kanavainen harmaasävy). |
| `"radiance"` | float32 W/m²/sr/nm koko radiometrisen ketjun kautta. Ohitetaan vain monispektrisissä — RGB-kamerat ohitetaan. |
| `"reflectance"` | uint16 0..32768 (Pix4D-yhteensopiva); vaatii reaaliaikaisen DAQ-pariliitoksen absoluuttista vertailukohtaa varten. Vain monispektrinen. |
| `"display"` | Täydellinen ketju, joka vastaa käyttöliittymän esikatselua (CCM + WB + gamma kameran profiilin mukaan). |
| `"all"` | **Yksi tiedosto kutakin soveltuvaa tasoa kohti** jokaiselle kameralle (vastaa GUI:n &quot;Capture All&quot; /CLI-oletusasetusta). Palautettu `CaptureResult` sisältää sitten yhden kehyssanakirjan kutakin `(cam, level)`:ää kohti, ja jokaisessa sanakirjassa on taso; soveltumattomat tasot näkyvät tiedostossa `.skipped`. Minkä tahansa heijastavuuskehyksessä tallennetaan `.daq`-sidecarina. |

> **Huomautus — oletusarvo poikkeaa CLI-tiedostosta.** `ArraySession.capture()`:n oletusarvona on `processing="debayered"`; `chloros-cli lattice array-capture`-komennon oletusarvona on `processing="all"`. Syötä `processing="all"` nimenomaisestiSDK-komennolla, jotta se vastaa CLI /GUI:n monitasoista tallennusta.

### Tallennustilat ja tallentimet

Matriisin pinta vastaa GUI:n tallennuspaneelia: Yksittäinen / Jatkuva / Väli / Nopein suljintila sekä kaksi tallenninta (live-komposiittivideo ja raaka sarjatallennus → offline-jälkikäsittely).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**on SDK:n jatkuva/väli-silmukka. Koska ei ole olemassa `Ctrl+C`:ää, jolla sen voisi keskeyttää skriptistä,**sinun on** välitettävä `count` ja/tai `duration_s` (se pysähtyy, kun jompikumpi saavutetaan). `interval_s` mitataan kunkin kierroksen alusta (GUI:n mukaisesti). Jäljellä olevat kwarg-argumentit välitetään suoraan `capture()`:ään.
- **`record`** on *valvontatasoinen*: se tallentaa reaaliaikaisen yhdistelmäindeksin komposiitin sellaisena kuin se näytetään, joten yhdistelmävirran on oltava auki, jotta kehykset pääsevät perille. Yksi komposiittitallennin kutakin taulukkoa kohti (heittää poikkeuksen, jos sellainen on jo käynnissä).
- **`burst` → `build_video`** on *analyysitason*: `burst` tallentaa raakakehykset + kehyskohtaisen manifestin + yhden `.daq`:n kutakin erillistä DLS-lukemaa kohti `<output>/bursts/<base>/`:n alla tallennussilmukan täydellä nopeudella (ei ketjua, ei exiftool-työkalua, ei reaaliaikaista esikatselua). `build_video` sovittaa kunkin kehyksen ajallisesti lähimpään `.daq`-kehykseen ja suorittaa tuontiputken säteily-/heijastavuus-/indeksiketjun uudelleen. `products` on luettelo `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}`-arvoista (oletus: yhdistetty indeksi). `burst().stop()` käynnistää myös automaattisesti parhaansa mukaan tehdyn yhdistettyjen indeksien rakennuksen, joka palautetaan `build_job`:na lopetustuloksessa.

#### `RecorderHandle`

Palautetaan `ArraySession.record()`:n ja `ArraySession.burst()`:n toimesta. Käytä sitä kontekstinhallintana, jotta se pysähtyy automaattisesti laajuuden päättyessä, tai ohjaa sitä manuaalisesti.

| Jäsen | Kuvaus |
| --- | --- |
| `job_id` | Taustatehtävän tunniste (merkkijono). |
| `kind` | `"composite"` (peräisin `record`:sta) tai `"raw"` (peräisin `burst`:sta). |
| `start_stats` | `start`-kutsun palauttama sanakirja. |
| `result` | `None` suorituksen aikana; lopullinen pysäytystulos-sanakirja, kun toiminta on lopetettu. |
| `stats(timeout=10.0)` | Reaaliaikaiset työn tilastot (kirjoitetut kehykset, toteutunut kuvataajuus, kulunut aika). |
| `stop(timeout=60.0)` | Pysäyttää tallentimen; palauttaa ja tallentaa välimuistiin lopullisen tuloksen. Idempotentti (toinen kutsu palauttaa välimuistissa olevan tuloksen). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### Liittäminen jo kytkettyyn taulukkoon — `attach_array`

Jos taulukko on jo käynnissä (GUI on avannut sen tai edellinen SDK-istunto on kutsunut `connect_array`:ää), käytä `attach_array`:ää saadaksesi siihen viittauksen uudelleenkytkemisen sijaan. `connect_array` antaa <sn><id>kyseisessä tilanteessa</id></sn> aina virheilmoituksen ”Kamera <sn>on jo ryhmässä<id>”, koska jäsenelle lähetetty POST-pyyntöei ole idempotenttia; `attach_array` lukee `/api/camera/array/list`:n ja vertaa joko array_id:n tai sarjanumeroiden perusteella.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Malli: SDK skriptit, jotka toimivat rinnakkain työpöydän graafisen käyttöliittymän kanssa, tulisi yrittää ensin `attach_array`:ää ja siirtyä `connect_array`:ään, jos poolissa ei vielä ole yhtään taulukkoa.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Tärkeää — context-managerin lopettaminen KATKAISEE yhteyden.**`ArraySession.disconnect()` lähettää aina POST-pyynnön `/array/disconnect`:lle; siihen ei ole liitetty-not-owned-suojaus, kuten on `CameraSession`:n / `DAQSensorSession`:n kohdalla. Jos käytät samaa vuokra-aluetta GUI:n kanssa etkähalua purkaa taulukkoa laajuuden lopetuksen yhteydessä,**älä käytä `with`-lohkoa** — säilytä kahva tavallisessa muuttujassa ja ohita eksplisiittinen `disconnect()`:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Verkkoanalyysin aputyökalu

Hyödyllinen ennen taulukon avaamista — arvioi, sopivatko ehdotetut asetukset:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

`status` on yksi seuraavista: `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (muussa tapauksessa `error`). `auto_capped_fps` tarkoittaa, että pyydetty resoluutio sopii RX-renkaaseen vain rajoitetulla laukaisutaajuudella — säilytä resoluutio ja siirrä `target_fps=result["recommended"]["recommended_target_fps"]` arvoon `connect_array` (katso [Esimerkki 6](#6-capability-probe-before-connecting-a-4-cam-array)).

**Kuinka tulkita projektio** (sama malli kuin GUI:n Array Settings -paneelissa):

- **Burst (`frame_bytes_total`) lasketaan yhteen kamerakohtaisesti kunkin kameran todellisen pikselimuodon mukaan.**Mono**M3M**-kamerat lähettävät Mono12-virtaa (2 B/px) riippumatta siitä, minkä `pixel_format`-arvon syötät, joten nelikamerainen täysresoluutioinen kehys on**~25 MB** kolmella monokameralla, ei ~12,6 MB:n kokoa, kuten oletettaisiin, jos kaikki olisivat 8-bittisiä. Taustaprosessi määrittää kunkin kameran formaatin sen mallin perusteella.
- **Admittance (`burst_fits_nic_ring`) on drain-tietoinen**, ei koko-burst-vs-ring-tyyppinen: sim-emit sopii tilanteeseen, jossa isäntä tyhjentää RX-renkaan nopeammin kuin kamerat täyttävät sitä. 10G-isäntä + 1 GbE-kamerat**sallivat** täyden resoluution, vaikka purske ylittäisi renkaan; 1 GbE:n isäntä estää (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` on konservatiivinen sarjahaun yläraja** — `max(readout+emit, N×emit)`, jossa kamerakohtainen lähetys on rajoitettu 1 GbE:n kamerayhteyteen, valotuksesta riippumatta. Esim. ~2,8 fps nelikameraiselle täysresoluutioiselle 12-bittiselle matriisille (vastaa ajoympäristössä mitattua ~2,7–3,0). Täydellinen malli: [CLI Viite → Matriisin fps- ja sarjakuvausmalli](cli-reference.md#array-fps--burst-model).
- **Ylimerkintä (`oversubscribed: true`) tarkoittaa, että N × kamerakohtainen alaraja ylittää törmäysturvallisen ylärajan** — fps-kentät (`achievable_fps_max` / `fps_bright` / `fps_dark`) lukevat arvoa 0, eikä automaattinen pienentäminen/binning korjaa tilannetta (ne vähentävät tavuja kehystä kohti, eivät tahdistettuja tavuja sekunnissa). Ratkaisuina ovat kameramäärän vähentäminen, jumbo-kehykset tai nopeampi verkkokortti; `max_cams_collision_safe` ilmoittaa ylärajan (6 täysresoluutioista kameraa 1 GbE:llä @ 1500 MTU, 9 jumbo-kehyksillä). Vastaus sisältää myös virhekoodit `aggregate_demand_bps`, `collision_safe_ceiling_bps` ja `per_cam_floor_bps` (8 MB/s). Katso [Ylimerkintä](#over-subscription-the-per-cam-floor).

### Tunnistus ja luettelointi

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

LATTICE-järjestelmät suorittavat jatkuvaa automaattista valotusta (AE) taustalla heti, kun ne on kytketty, mutta vastikään kohdistetun kuvauskohteen valotuksen vakiintuminen vie hetken. **Smart-Capture** on kätevä toiminto: se tarkistaa kunkin kameran valotuksen, odottaa, kunnes järjestelmä on vakaa koko ikkunassa, ja laukaisee sitten kuvauksen. Se vastaa graafisen käyttöliittymän toimintaa: työpöytäsovelluksen ”älykäs” tallennuspainike kutsuu samaa taustapalvelimen päätepistettä.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Kun käytät `ChlorosProject`-ohjausta (seuraava osa), saat käyttöösi lisää säätimiä:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Älykäs AE-käytäntö on oletusarvoisesti konservatiivinen. Kiristä `exposure_tolerance_pct`-asetusta tarkkoihin radiometrisiin töihin; laajenna sitä nopeasti muuttuvissa tilanteissa, joissa riittää, että tulos on ”riittävän lähellä”.

---

## DAQ-anturisessiot

Pysyvä taustapooli spektrisensoreille (DAQ-U USB:n kautta, DAQ-M BLE:n kautta, DAQ-E Ethernetin kautta). Vastaa kameran toimintaa: älykäs tunnistus, poolin uudelleenkäyttö, idempotentti liittäminen.

### Älykäs tunnistus (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Prioriteetti: Ethernet → BLE → USB. Määritä mikä tahansa eksplisiittinen vihje kiinnittääksesi siirtoyhteyden.

### Kiinnitetty siirtotapa

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### `DAQSensorSession`-metodit

| Metodi | Kuvaus |
| --- | --- |
| `status(timeout=10.0)` | Pool-merkinnän yhteenveto (suoratoisto-/tallennustila, aallonpituusalue, kalibrointisha, integraatioaika, frame_avg, AE-tila). |
| `latest(n=1, timeout=10.0)` | Palauttaa enintään N viimeisintä spektrikehystä. |
| `stream_start()` / `stream_stop()` | Jatka / keskeytä suoratoisto (käsittely pysyy auki). |
| `record_start(output_dir=None, device_name=None)` | Aloita .daq-tiedoston tallennus. Palauttaa tiedostopolun. Hylkää DAQ-U/M:n ilman AWS-kalibrointipakettia (DAQ-E on poikkeus). |
| `record_stop()` | Lopeta tallennus. Palauttaa `{path, rows}`. |
| `disconnect()` | Vapauttaa poolista. Ei-toimiva liitetyille, mutta ei omistetuille kahvoille. |

> **Kapasiteetin korjausprofiilit (`cap_id`) eivät ole SDK-säätimiä.** `connect_daq_sensor()` / `DAQSensorSession` eivät paljasta mitään `cap_id`-parametria tai `set_cap`-menetelmää. Valitse laivaston kapasiteetin korjausprofiili CLI -sivuston kautta (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) tai taustapalvelimen `/api/daq`-HTTP-reittien kautta (`/api/daq/connect` ja `/api/daq/<id>/cap-id` hyväksyvät `cap_id`).

### Löytö — yhteyden muodostamiseen tarvittavan osoitteen etsiminen

`discover_daq_sensors()` skannaa USB- / BLE- ja ETH-liitäntöjä etsiäkseen antureita, jotka *voisit* avata. Se on DAQ-vastaava `discover_lattice_cameras()`:lle, ja ainoa tapa saada selville **DAQ-M:n BLE-MAC-osoite** — DAQ-E:llä on isäntänimi ja DAQ-U:lla COM-portti, mutta MAC-osoitetta ei ole painettu laitteeseen eikä se näy käyttöjärjestelmän luettelossa.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Kenttä | Kuvaus |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM-portti / BLE MAC / isäntänimi — välitetään `connect_daq_sensor`:lle muodossa `port=` / `mac=` / `eth_host=`. |
| `display` | Ihmisen luettavissa oleva nimike. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` tai `None`, jos skannaus ei tunnista porttia (USB-sarjaliikenneadaptereita ei voida erottaa toisistaan ilman mittausanturia, joten tuntemattomat portit näytetään eikä piiloteta). |
| `extra` | Kuljetuskohtaiset tiedot (BLE:n ilmoitettu nimi, USB-valmistaja, DAQ-E:n IP/fw/…). Tyhjät arvot jätetään pois. |

| Parametri | Oletus | Kuvaus |
| --- | --- | --- |
| `transports` | kaikki kolme | Skannausta rajoittava sekvenssi (tai CSV-merkkijono). Kannattaa antaa, kun tiedät mitä haluat — BLE on hidas osa. |
| `scan_timeout` | 5 |-kuljetuskohtainen skannausikkuna sekunteina; taustapalvelin rajoittaa arvon välille 1–20. |
| `timeout` | 60,0 | Koko kutsun HTTP-yläraja (kuten muuallakin SDK-tiedostossa). |
| `auto_start_backend` | `True` | Luo paikallinen taustaprosessi, jos sellaista ei ole käynnissä. Ei luo koskaan etäistä `backend_url`-prosessia. |

> **Poolissa jo avoimina olevat anturit eivät näy.** Yhdistetty BLE-oheislaite lopettaa mainostamisen, eikä avointa COM-porttia voi tunnistaa, joten hakutoiminto listaa ne, jotka ovat *yhdistettävissä*. On odotettavissa, että tulos on tyhjä heti sen jälkeen, kun olet yhdistänyt jonkin laitteen — käytä `list_daq_sensors()`:ää niille, jotka sinulla jo on. Siirrot , joiden skannaus ei voi suorittaa (bleak / zeroconf ei ole asennettu), ohitetaan sen sijaan, että ne aiheuttaisivat virheilmoituksen, joten kone, jossa ei ole Bluetoothia, saa silti vastaukset USB:lle ja ETH:lle.

### Luettelo

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Yhteiskäyttö GUI:n kanssa / CLI

Jos GUI:ssa on jo auki oleva anturi, Python-komennolla kutsuttu `connect_daq_sensor(port="COM3")` palauttaa käsittelytunnuksen, joka on merkitty `already_connected=True`. Istunnon `disconnect()` on tällöin tyhjä operaatio, joten SDK-skriptisi ei irrota anturia GUI:n alta, kun poistumisen yhteydessä.

### Suorat laitteistoluokat (ilman taustaprosessia)

`daq_sdk` onvienti `chloros_sdk`:n kautta, joten voit myös ohjata antureita päästä päähän prosessin sisällä ilman taustapalvelinta:

> **Saatavuus:**`daq_sdk` toimitetaan Chloros-työpöytäasennuksen mukana,**ei** PyPI-paketin mukana — `pip install chloros-sdk` tarjoaa sinulle `lattice_sdk`:n, mutta jättää `chloros_sdk.DAQ_AVAILABLE == False`:n pois. Tarkista tämä asetus ennen näiden luokkien käyttöä; pip-vain-isäntäkoneella ohjaa anturia sen sijaan [`connect_daq_sensor()`](#daq-sensor-sessions) kautta, joka ei vaadi paikallisia siirtokirjastoja.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

Käytä mieluummin smart-connect-polkua (`connect_daq_sensor`), kun haluat jakaa omistajuuden graafisen käyttöliittymän kanssa; käytä suoria luokkia päättömille skripteille, jotka omistavat anturin yksin.

---

## Projektiautomaatio — `ChlorosProject`

Tallennettu Chloros-projekti on kansio, joka sisältää `cameras.json` + `sensors.json` + `project.json`. `open_project` lataa manifestin, ja `connect_all` kytkee kaikki tallennetut laitteet verkkoon niiden tallennetuilla asetuksilla — samassa laitteistotilassa kuin mitä graafinen käyttöliittymä tuottaisi.

### Yksinkertainen esimerkki

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

Tai kontekstinhallintana:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### `ChlorosProject`-metodit

| Metodi | Kuvaus |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Etsi ja yhdistä kaikki tallennetut laitteet. Palauttaa luokkakohtaisen yhteysraportin. Käyttää käynnissä olevaa taustapalvelinta, jos sellainen kuuntelee osoitteessa `127.0.0.1:5000`; muussa tapauksessa siirtyy hiljaisesti suoraan (taustapalvelin-vapaaseen) `lattice_sdk`-laiteohjaukseen — se ei koskaan luo taustapalvelinta. |
| `disconnect_all()` | Katkaise kaikki yhteydet. |
| `capture_all(output_dir=".")` | Yksi kehys jokaisesta kamerasta + matriisi + spektri jokaisesta anturista. |
| `stream(camera, overlays=False, fps=10.0)` | Generaattori, joka tuottaa BGR-kehyksiä nimetystä kamerasta (tai matriisista). `overlays=False` on suora `lattice_sdk`-sieppaussilmukka (matriisit tuottavat `{serial: frame}`-sanakirjoja). `overlays=True` reititetään `ChlorosLocal.camera_stream()` → taustapalvelimen `/api/camera/<serial>/stream-annotated` MJPEG-syötteen kautta, jolloin kameran tallennettu `ui.overlay`-lohko välitetään kyselyparametreina. Vaatii taustapalvelintilan ja **itsenäisen kameran**: suoramoodikamera aiheuttaa virheen `RuntimeError` (taustapalvelu ei voi napata tämän prosessin omistamaa kameraa) ja taulukko aiheuttaa virheen `NotImplementedError` (yhdistää overlayt kamerakohtaisesti — lähettää jäsenen nimellä). Vastaava kertakäyttöinen toiminto: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Suorita kohdistus jokaiselle tällä hetkellä kytketylle taulukolle. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Suorita kalibrointi-/indeksointiprosessi projektinkuviin (käärii `ChlorosLocal.process`:n; nämä neljä ovat **ainoat** hyväksytyt avainsanat — `indices=` jne. aiheuttavat virheen `TypeError`; aseta indeksit komennolla `ChlorosLocal.configure()`). Rakentaa viiveellä `ChlorosLocal()`, joka käynnistää taustaprosessin automaattisesti. |

Attribuutit:
- `proj.cameras` — `Dict[str, CameraHandle]`, jonka avain on nimi JA sarjanumero.
- `proj.arrays` — `Dict[str, ArrayHandle]`, jonka avain on nimi JA array_id.
- `proj.sensors` — `Dict[str, SensorHandle]`, jonka avain on nimi JA slot_id.
- `proj.config` — `project.json["config"]`-sanakirja.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Käsittelytasot.** `capture()`, `grab()` ja `frame_stream()` ottavat kaikki vastaan saman `processing`
-tunnusta, ja ketju on kumulatiivinen — jokainen taso suorittaa kaikki sitä ylemmät tasot:

| Taso | Lähtö | Huomautukset |
| --- | --- | --- |
| `raw` | 1-kanavainen Bayer, anturin oma | Ei demosaicia. Päällekkäiskuvia ei ole käytettävissä tällä tasolla. |
| `debayered` | 3-kanavainen BGR (**oletus**) | Bilineaarinen demosaikointi. Ainoa taso, joka toimii ilman backend-tilaa. |
| `radiance` | float32, W/m²/sr/nm | Täydellinen radiometrinen ketju: demosaikki + 3×3-sekoituksen purku (multispec) + DSNU + tasokenttäkorjaus + NIST-asteikko, jossa valotus × vahvistus on jaettu pois, jotta arvot ovat absoluuttisia. |
| `reflectance` | uint16, 32768 = 1,0 | Säteilysuhde jaettuna alaspäin suuntautuvalla säteilyvoimakkuudella (ρ = π·L/E). Vaatii DLS/DAQ-lukeman — katso huomautus alla. |
| `display` | 8-bittinen sRGB-tyyppinen | GUI-vastaava renderointi: CCM + valkotasapaino + gamma kameran aktiivisen väriprofiilin kautta. |

Mikä tahansa muu kuin `debayered` vaatii taustatoimintatilaa; suoratoimintatilan kamera nostaa
`NotImplementedError`:n. `reflectance` vaatii käyttökelpoisen alaspäin suuntautuvan lukeman — kehyksen päätepiste vetää
kootun DAQ:n automaattisesti kameran DLS-paikkaan, mutta ilman sidottua DAQ:ta ketju hylkää
heijastavuuslähtöä ja merkitsee rehellisesti alennuksen palautettaviin metatietoihin sen sijaan, että
palauttaa heikompaa tuotetta.

> **Heijastavuuden DN-asteikko — älä koodaa sitä kiinteästi.** LATTICE-heijastavuus käyttää `32768` = ρ 1,0 ja merkitsee
> XMP `Chloros:PixelScale=32768`; Survey3 heijastavuus käyttää `65535` = ρ 1,0 eikä sisällä
> `Chloros:*`-tunnisteita. Lue tunniste ja jaa se sillä. Se on määritelty uint16-alueella, joten se pysyy
> `32768`:na kaikissa formaateissa, joissa mittakaava muuttuu (16-bittinen TIFF, 8-bittinen PNG /JPG, 32-bittinen prosentti) — normalisoi
> tallennettu dtype ensin takaisin uint16:ksi (×257 8-bittisestä, ×65535 float-tyypistä). Ainoa poikkeus:
> 8-bittisestä lähteestä peräisin oleva tallenne, joka on kirjoitettu muodossa 8-bittisenä TIFF *rajataan*, ei skaalata uudelleen, joten sitä ei kuvata millään skaalalla
> — Chloros jättää siinä tapauksessa `PixelScale`:n ja MicaSense-tupelin kokonaan pois. Käsittele puuttuvaa
> tunnistetta LATTICE-heijastustiedostossa ”ei kelvollista skaalaa” -tilanteena, ei oletusarvona.

> **EXIF-tiedot siirretään vientiin.** `process()` kopioi lähdekuvan GPS-lohkon
> **ja sen ExifIFD:n** jokaiseen tuotteeseen, joten vienti sisältää `FocalLength`:n, `FNumber`:n,
> `ExposureTime`:n, `ISO`:n, `DateTimeOriginal` ja `CameraSerialNumber` sekä
> georeferenssitiedot. `FocalLength` on se, mistä Pix4D laskee maanpinnan näytteenottovälin — ilman sitä
> rekonstruktio perustuu täysin väärään mittakaavaan (eräässä mitatussa tapauksessa 411 metrin kokoinen alue
> muuttui 47,8 kilometrin alueeksi). Kopio ei ole tarkoituksella `-all:all`: IFD0:n rakenteelliset tunnisteet häiritsevät
> LATTICE-tulostetta, ja `ExifImageWidth`/`Height` on jätetty pois, koska ne kuvaavat lähteen
> tallennusta eikä vietyä rasteria.

Kuvausvaiheen alilippuja (koskevat radiometrisiä tasoja — `radiance`, `reflectance`, `display`):

| Lippu | Oletus | Merkitys |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + tasauskuva + 3x3-sekoituksen erottelu + NIST-radiometrinen asteikko. |
| `apply_white_balance` | `True` | WB LUT. DLS-tuki, kun DAQ on liitetty kameraan. |
| `apply_index` | `False` | Kasvillisuusindeksin arviointi. |
| `index_expression` | `None` | Kaavan ohitus. Tyhjätyhjä → indeksi otetaan käyttöön automaattisesti. |
| `annotated` | `False` | GUI-koristeiden (zebra/ruudukko/peaking) päällekkäisyys. Ei käytettävissä `raw`:lle. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Palautustyyppi on `CapturePathMap`, ei `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` on `Dict[str, Union[str, List[str]]]`: yksitasoinen
> `processing` antaa jokaiselle sarjanumerolle yhden polun, kun taas monitasoinen (`"all"`, tai
> eksplisiittinen `levels`-luettelo) antaa sille **järjestetyn luettelon** kaikista kyseiselle
> kameralle tallennetuista tuotteista. Yhdistetty reaaliaikainen komposiitti, jos sellainen olisi suoratoistossa, saapuu erillisen
> `"combined"`-avaimen alle eikä sarjanimikkeen alle. Koodi, joka olettaa `str`:n, kaatuu
> luettelomuodossa ilman, että tyyppitarkistaja ilmoittaisi virheestä — merkinnän mukaan `Dict[str, str]`
> vielä jonkin aikaa luettelo-muodon julkaisun jälkeen, minkä vuoksi alias on olemassa. Normalisoi
> kun haluat tasaisen muodon:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Taulukon kohdistus

`ArrayHandle` paljastaa koko kohdistuspinnan. Profiilit ovat oletusarvoisesti vain istuntokohtaisia — kutsu `export_alignment()` nimenomaisesti, jos haluat tallentaa ne pysyvästi.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Yhteyden muodostuksen yhteydessä tapahtuva kohdistus

`connect_all(align=...)` voi kohdistaa jokaisen taulukon automaattisesti yhteyden muodostuksen yhteydessä:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Käytetään oletuksena `project.json["config"]["auto_align_on_connect"]`:ää, jos asetusta ei ole määritetty.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## Suora laitteisto (ilman taustajärjestelmää)

Kun haluat täysin riippumattoman ratkaisun taustajärjestelmästä (CI, päättömät robotit, sulautetut järjestelmät), tuo `lattice_sdk` ja `daq_sdk` suoraan — molemmat viedään uudelleen `chloros_sdk`:n kautta. Suojaa `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`: `lattice_sdk` on PyPI-paketissa (mutta vaatii Arena-SDK-ajoympäristön), kun taas `daq_sdk` toimitetaan vain työpöytäasennuksen mukana.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Esiasetukset ja laukaisija

Kolme neljästä esiasetuksesta **free-run**: kamera valottaa jatkuvasti ja
`capture()` palauttaa seuraavan kehyksen. `triggered` on poikkeus — se virittää
kameran odottamaan laitteistoreunaa rivillä 2, joten se ei tallenna mitään ennen kuin sellainen saapuu.

| Esiasetus | Laukaisija | Käytä, kun |
| --- | --- | --- |
| `default` | vapaakäynti | yleiskäyttö |
| `high_speed` | vapaakäynti | 8-bittinen, 60 fps:n rajoitus, lyhyt valotusaika |
| `high_quality` | vapaakäyttö | 12-bittinen, ei fps-rajoitusta — tavallinen valinta still-kuville |
| `triggered` | **valmiustila, linja 2** | kamera on kytketty M8-synkronointikaapeliin ja jokin muu laukaisee sen |

Jos valitset `triggered` (tai asetat itse `trigger_mode="On"`) ilman, että mikään
ohjaa linjaa 2, jokainen `capture()` aikakatkaistaan — oikein, koska pyysit
kameraa odottamaan. SDK selittää tämän, kun se tapahtuu; katso
[SC_ERR_TIMEOUT tallennuksen aikana](#direct-hardware-backend-free).

> **Huomautus — &quot;GVSP probe&quot; / `SC_ERR_TIMEOUT -1011`-viestit yhteyden muodostamisen yhteydessä eivät ole virheitä.**&gt; Yhteyden muodostamisen yhteydessä SDK yrittää neuvotella**jumbo-kehyksistä** (9000 tavun GVSP-paketit) suuremman siirtonopeuden saavuttamiseksi. Suoralla pisteestä pisteeseen -verkkokorttiliitännällä (esim.-paikallisessa `169.254.x.x`-osoitteessa) verkko ei yleensä kykene siirtämään jumbo-kehyksiä, joten tämä koe aikakatkaistaan ja lokiin kirjataan esimerkiksi seuraavanlaisia viestejä:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Tämä on **suunniteltu varajärjestelmä**: SDK palaa automaattisesti tavallisiin 1500-tavuisia paketteja, ja kamera jatkaa yhteyden muodostamista normaalisti (seuraavat `[chunk-enable …]`-rivit ovat osa normaalia yhteydenmuodostussekvenssiä). Kaappaus toimii edelleen.
>
> Voit ohittaa tämän testin, mutta **se ei ole pelkkä lokitiedostojen vaimentaja — se poistaa jumbo-kehykset käytöstä.** Kamera vastaa Don&#x27;t-Fragment-ping-kyselyihin vain enintään 1500 tavun paketeilla riippumatta siitä, kuinka hyvä verkko on, joten pelkällä ping-testillä ei voi koskaan havaita jumbo-kehyksiä; vain tämä testi pystyy siihen. Jos poistat sen käytöstä, kamera käyttää vakiokokoisia 1500-tavuisia paketteja ikuisesti, missä tahansa verkossa:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Kannattaa tehdä vain verkossa, josta *tiedät*, ettei se kestä jumbo-paketteja, missä se säästää noin sekunnin yhteyden muodostusaikaa kameraa kohti. Koska kyseessä on todellinen kompromissi eikä pelkkä kosmeettinen muutos, SDK ilmoittaa nyt asiasta, kun käytät sitä:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Jätä se rauhaan, ellet ole syytä muuttaa sitä.** Jos ominaisuus jätetään käyttöön, jokainen yhteyden muodostus mittaa uudelleen käytössä olevan verkon: kytke kamera jumbo-paketteja tukevaan kytkimeen, ja seuraava yhteyden muodostus tunnistaa jumbo-paketit itsestään, ilman mitään konfigurointia tai uudelleenkäynnistystä.
>
> Jos *haluat* jumbo-läpimenon, ota jumbo käyttöön päästä päähän (NIC MTU 9000 + kytkin, joka välittää ne), tai kiinnitä se asetuksella `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`, kun tiedät, että linkki tukee sitä — vaikka suosittelen komentoa kohti määritettävää `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …`-asetusta pysyvän asetuksen sijaan, sillä kiinteästi määritetty koko ohittaa testauksen ja estää sopeutumisen edessä olevaan verkkoon. **Jokaisen** reitillä olevan laitteen on välitettävä jumbo-paketteja — mukaan lukien kaikki PoE-jakajat tai -injektorit, jotka ovat tavallisin syy siihen, että muuten jumbo-paketteja tukeva kokoonpano ei pysty välittämään niitä.

> **`SC_ERR_TIMEOUT -1011` `capture()`- tai `grab*()`-tarkistuksen aikana on eri ongelma — se on todellinen virhe.**&gt; Yllä oleva huomautus koskee vain**connect-time-probe**-toiminnon kirjaamaa `-1011`-virhettä. Sama virhe, joka ilmenee**capture**-toiminnossa, tarkoittaa, että kamera on kytkeytynyt kunnolla, mutta ei lähetä kuvia:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Paljastava tekijä on kamera, jonka *ohjaus*kanava on kunnossa — tunnistus toimii, asetukset ja `[chunk-enable …]`-kirjoitukset onnistuvat kaikki — mutta *jokainen* kehys ylittää aikarajan.
>
> **Tavallisin syy on, että kamera on asetettu laitteistolaukaisulle.** Kun virheet `trigger_mode="On"` ja `trigger_source="Line2"` ilmenevät, kamera ei lähetä mitään, ennen kuin M8-synkronointikaapeliin saapuu sähköinen reuna. Jos kyseistä linjaa ohjaavaa kaapelia ei ole, jokainen kuvanotto odottaa ikuisesti. Kamera ei ole rikki ja verkko toimii kunnolla — se toimii täsmälleen niin kuin sille on määrätty.
>
> `CameraSettings()` ja `default` / `high_speed` / `high_quality`-esiasetukset sallivat vapaan toiminnan, ja kuvausyritys, jonka aikakatkaisu tapahtuu laitteen ollessa valmiustilassa, selittää tilanteen sen sijaan, että tulostettaisiin pelkkä `-1011`. `PRESETS["triggered"]` aktivoi Line2:n, kuten on suunniteltu.
>
> Kameran pakottaminen vapaakäyntiin:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Jos aikakatkaisu tapahtuu edelleen komennolla `trigger_mode="Off"`, kamera ei todellakaan lähetä dataa — lähetä meille loki ja komento `ip link show`.

#### Väriprofiilit (RGB-reaaliaikainen esikatselu) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` valitsee näytön väriprofiilin **reaaliaikaiselle esikatselulle** RGB-kameroissa (multispec-kamerat eivät ota asetusta huomioon):

| Profiili | Merkitys |
| --- | --- |
| `raw` | Ohita radiometrinen ketju kokonaan. |
| `linear` | DSNU + tasoitus + valkotasapaino, ei CCM:ää, ei gammaa. |
| `natural` | Lineaarinen + mitattu CCM + sRGB-gamma, vain edullisella viimeistelyllä (kromaattinen tasoitus + korostusten desaturaatio) — realistinen oletusasetus. |
| `enhanced` | `natural` sekä täysi hub-parity-viimeistely (reunojen tasoitus, värikylläisyys, CLAHE-paikalliskontrasti). Rikkaampi ulkoasu, jonka **viimeistelykustannus kehystä kohti on noin kaksinkertainen**, joten LIVE-kuvataajuus on alhaisempi. |
| `custom_temp` | `natural`, mutta valkotasapaino kiinnitetty `custom_cct_k` Kelviniin (DLS ohitetaan; rajoitettu 2000–10000 K:n välille backend-puolella). |

Profiili on **vain reaaliaikaisessa esikatselussa** nopeus-/ulkoasukytkin: tallennetut kuvat saavat aina täyden ja rikkaan viimeistelyn valitusta profiilista riippumatta, joten `natural`:n valitseminen kehysajan säästämiseksi ei heikennä levylle tallennettavan materiaalin laatua. Tuntematon profiili nostaa `ValueError`:n arvoa; kun chloros-taustapalvelin on käytettävissä, muutos lähetetään myös sinne POST-pyynnöllä, jotta seuraava esikatselukehys heijastaa sitä (direct-SDK-käyttäjät, joilla ei ole taustapalvelinta, saavat silti asetusten muutoksen).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Mono (M3M)-kamerat ja `Calibration`

Mono **M3M**-kamera (`M3M-<lens>-F<wavelength>`) on yksikaistainen: yksi harmaasävyinen taso, ei Bayer-mosaiikkia, ei 3×3-spektrikrosstalk-matriisia. `Calibration` tunnistaa sen ja paljastaa `is_mono`-lipun. Heijastavuus pätee edelleen kaistakohtaisena radiometrisena karttana (sekoituksen purkaminen tapahtuu identiteettimatriisin avulla), mutta yksittäisen kameran monikaistamatematiikka tuottaa mielekkäitä tuloksia sen sijaan, että se palauttaisi hölynpölyä:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Vegetaatioindeksin rakentamiseksi monokromaattisella laitteistolla yhdistetään useita eri aallonpituuksilla toimivia M3M-kameroita kohdistetuksi monikaistapinoksi (katso [Array Alignment](#array-alignment)) ja laske indeksi kyseisen pinoon perustuen yhden kameran sijaan.

DAQ-suoratila:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **`apply_sensor_settings` hyväksytyt avaimet**— tarkalleen `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E; käytöstä poistettu, korvattu `cap_id`:llä), `filter_model` (DAQ-M), ja `cap_id` (kaikki DAQ-tyypit; `None`/`""`/`"none"` = pelkkä anturi, ilman kondensaattorikorjausta). Tuntemattomat avaimet**ohitetaan huomaamatta** — esim.esim. `{"integration_time": 64}` ei tee mitään (sen on oltava `integration_time_ms`). Palauttaa `{"applied": [...], "errors": {...}}` eikä koskaan aiheuta poikkeusta.

`chloros_sdk` vievain yllä käytetyn ytimen pinnan. Koko `daq_sdk`-julkinen API (22 nimeä) lisää seuraavat — tuo ne suoraan `daq_sdk`:stä:

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## Poikkeukset

Ota kiinni perusluokka käsittelemään ”kaikki, mikä meni pieleen Chloros”:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` ja `ChlorosConfigurationError` viedään ylätasolle muiden rinnalle; ne voidaan myös tuoda tiedostosta `chloros_sdk.exceptions`:stä, kuten kuvassa on esitetty.

Hierarkia:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Kokonaisvaltaiset esimerkit

### 1. Kansion käsittely mukautetulla edistymispalkilla

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Live-LATTICE-matriisi → heijastavuus + DAQ-viite

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Projektilähtöinen tallennuskampanja

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Monikameran kehysvirta → NumPy-putki

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Headless-suora laitteistopohjainen (ilman taustapalvelinta) tallennusskripti

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Toimintakyvyn tarkistus ennen 4-kamerajärjestelmän kytkemistä

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Tallennusohjeen vastine (puhdas Python)

CLI -ohjelman resepti-DSL:llä on suora Python-vastine:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Taustaprosessin automaattinen käynnistys

Smart-connect-liityntäpisteet — `connect_camera`, `connect_array`, `connect_daq_sensor` ja `discover_lattice_cameras` — ovat ohuita HTTP-asiakkaita, jotka olettavat, että taustaohjelma kuuntelee porttia `127.0.0.1:5000` (Smart-Connect-rajapinnan oletusarvoinen URL). Kun graafinen käyttöliittymä (GUI) tai CLI on jo käynnissä, yksi niistä on käynnissä. Pelkästä skriptistä sitä ei välttämättä ole — joten nämä toiminnot **käynnistävät automaattisesti mukana toimitetun taustaprosessin** (ikkunattomana, samalla tavalla kuin `ChlorosLocal`) ennen ensimmäistä kutsua ja odottavat sitten enintään `backend_startup_timeout`, kunnes se käynnistyy.

Säännöt:

- **Ainoastaan paikallinen URL käynnistetään.** `backend_url`, joka osoittaa `localhost`:ään / `127.0.0.1`:ään / `[::1]` on sallittu; kaikkia muita isäntiä pidetään jonkun muun koneina, eikä niitä koskaan käynnistetä.
- **Taustapalvelu jätetään käynnissä uudelleenkäyttöä varten** (samoin kuin CLI) — skriptin päättyessä ei tapahdu automaattista sammutusta. Skriptin uudelleenkäynnistyksessä käytetään jo käynnissä olevaa taustapalvelua.
- **Voit kieltäytyä tästä käyttämällä `auto_start_backend=False`** missä tahansa näistä kutsuista (esim. kun olet määrittänyt etäisen taustaprosessin tai hallitset taustaprosessin elinkaarta itse).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Jos mukana toimitettua binääritiedostoa ei löydy tai sitä ei voida käynnistää, seuraava HTTP-kutsu aiheuttaa toimitettavan, **alustakohtaisen** `ChlorosConnectError`-virheen sen sijaan, että se tuottaisi pelkän yhteyden hylkäämistä koskevan jäljityksen — Windows-sivustolla se ohjaa sinut työpöytäsovellukseen tai `chloros-cli`-komentoon; Linux-sivustolla (ei graafista käyttöliittymää) se ohjaa sinut `chloros-cli`-komentoon tai `.deb`-komentoon.

---

## Ympäristö ja otsikot

SDK-tiedosto merkitsee jokaisen taustapalvelimen HTTP-kutsun tunnisteella `X-Chloros-Client: sdk`. Taustapalvelin soveltaa SDK / CLI lisenssisääntöjä (kirjautuminen **ja** vaaditaan maksullinen Chloros+ -paketti) GUI:n ilmaisen tason sijaan. Tämä asetetaan automaattisesti tuonnin yhteydessä — sinun ei tarvitse tehdä mitään.

`http://localhost` ja `http://127.0.0.1` tunnistetaan paikalliseksi taustapalvelimeksi. Kutsut muihin isäntäkoneisiin (esim. omaan analytiikkapalveluusi) jätetään ennalleen.

Ohita taustapalvelu URL välittämällä `backend_url=` (tai `api_url=`, jos käytössä on `ChlorosLocal`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Muut kuin loopback-yhteydet `backend_url` saavuttavat vain lähde-/dev-taustapalvelimeen — mukana toimitetut taustapalvelimet sitovat vain loopbackia; katso tunnelimallia kohdasta Remote-Backend Mode.)

---

## Versiointi ja yhteensopivuus

- SDK-versio näkyy nimellä `chloros_sdk.__version__`.
- SDK sitoo käyttäytymisen mukana toimitettuun taustapalvelinversioon. Vanhemman SDK-version yhdistäminen uudemman taustapalvelimen kanssa toimii yleensä (eteenpäin yhteensopivat päätepisteet), mutta uudemman SDK-version yhdistäminen vanhemman taustapalvelimen kanssa saattaa aiheuttaa `404`-virheitä uusissa päätepisteissä — päivitä työpöytäsovellus vastaavaksi.
- Smart-Connect-käyttöliittymä (`connect_camera` / `connect_array` / `connect_daq_sensor`) ja verkkoanalyysin päätepiste palauttavat vakaat JSON-skeemat; uudet kentät ovat lisäyksiä.

---

## Vianmääritysvinkkejä

- **`ChlorosAuthenticationError: Login required`** → Suorita komento `chloros-cli login EMAIL PASSWORD` kerran tällä koneella tai kirjaudu sisään Chloros-työpöytäsovelluksen kautta.
- **`ChlorosConnectError: No Chloros backend is running …`** → Smart-Connect-kutsut käynnistävät paikallisen taustapalvelimen automaattisesti, joten tämä viesti näkyy vain, jos mukana toimitettua binääritiedostoa ei löydy tai sitä ei voida käynnistää (esim. pelkästään pip-ympäristössä toimiva isäntäkone, jolla ei ole työpöytäpakettia). Viesti on alustakohtainen: Windows-käyttöjärjestelmässä avaa työpöytäsovellus tai suorita mikä tahansa `chloros-cli`-komento; Linux-käyttöjärjestelmässä suorita `chloros-cli`-komento (GUI-käyttöliittymää ei ole) tai asenna `.deb`. Etätaustapalvelimelle välitä `backend_url=` (ja `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** tuonnin yhteydessä → `lattice_sdk`:n lataaminen epäonnistui (yleensä Arena-SDK-ajonaikaiset DLL-tiedostot eivät ole asennettuina). Muut kuin kamerapinnat toimivat edelleen.
- **Array connect palauttaa natiiviresoluutiota alhaisemman resoluution**→ Backendin smart-prep-toiminto-pienentää kehyksen kokoa, jotta se mahtuu johtoon. Käytä `analyze_array_network()`:ää selvittääksesi syyn, ja päivitä sitten linkki, hyväksy pienentäminen tai käytä `force_tier="slip-emit-and-capture"`:ää peräkkäiseen tallennukseen. Pienentämisen turvaverkko**ei** kata yhteenlaskettua ylimääräistä(`oversubscribed: true`, fps-kentät 0): liian monia kameroita yhteydelle ei voida korjata binningillä/ROI:lla — vähennä kameroiden määrää, ota käyttöön jumbo-kehykset tai siirry nopeampaan verkkokorttiin (katso [Ylimerkintä](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` ilmoittaa verkkokortin vastaanottorengasta pieneksi (~0,26 Mt) / yhdistä portit, joissa on viesti ”FRAMES WILL DROP”** → Isäntäkortin vastaanottorengas on oletusarvossaan (nollautuu usein arvoon 32 verkkokortin ohjaimen päivityksen jälkeen). Realtek USB 10GbE -sovittimella aseta `ReceiveBufferLen=256` ja `PendingReceives=64` (korotettu), ja käynnistä sitten taustapalvelu uudelleen, jotta se lukee renkaan uudelleen. Koko menettely: [CLI Viite → Isäntäverkkokortin asetukset ja säätö](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Isäntäkone jumittuu uudelleenkäynnistyksen tai sammutuksen yhteydessä, myöhemmin WMI-virheitä `Invalid class` / verkkokortti ei aktivoidu** → Vanhentunut USB 10GbE -ohjain aiheuttaa virheen `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Päivitä sovittimen ohjain nykyiseen versioon (≥ 2026) ja määritä vastaanottorengasasetukset uudelleen. Katso [CLI-ohje → Isäntätietokoneen verkkokortin asetukset ja säätö](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Heijastavuus hylätty** → Absoluuttisen mittakaavan heijastavuuden mittaamiseksi kameraan (tai anturiryhmään) on liitettävä aktiivinen DAQ-laite. Liitä joko käyttöliittymän kautta tai käytä `processing="radiance"` (W/m²/sr/nm) -asetusta, joka ei vaadi pariksi liitettyä anturia.
- **`smart=True`-tallennus kestää odotettua kauemmin** → AE-konvergenssi riippuu kohteen dynamiikasta; kiristä `exposure_tolerance_pct`-arvoa tai lyhennä `stability_window_s`-arvoa, jos haluat nopeamman (vähemmän vakaan) laukaisun.

---

## Katso myös

- [CLI-viite](cli-reference.md) — jokainen CLI-alikomento vastaa SDK-kutsua.
- [DAQ-anturiohje](../daq/README.md) — anturikohtaiset kytkentä-, kalibrointi- ja tallennussäännöt.
- Verkkodokumentaatio: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>
