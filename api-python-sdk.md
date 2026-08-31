# API : Python SDK

{% hint style="info" %}
**Etsitkö täydellistä API-opasta?** Tämä sivu on käytännönläheinen opas. Kaikki julkiset luokat, metodit, tarkat allekirjoitukset ja kopioitavat esimerkit löytyvät [SDK-viitteestä](reference/sdk-reference.md), joka on optimoitu tekoälyavustajille.**Käytätkö tekoälyavustajaa?** Liitä tämä URL chat-keskusteluun, jotta avustajalla on käytössään täydellinen ja ajantasainen Chloros 1.2.0 API:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Tämän käyttöoppaan jokainen sivu on saatavilla raakamuodossa (markdown) sen pienillä kirjaimilla kirjoitetulla slug-tunnuksella + `.md`, ja koko käyttöopas on indeksoitu osoitteessa `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` PyPI:ssä) ohjaa kaikkea, mitä työpöytäsovellus pystyy tekemään alkaen kohdasta Python: kuvien eräkäsittely, LATTICE-kameran ja -matriisin reaaliaikainen ohjaus, DAQ-valosensorisessiot sekä tallennettujen projektien automaatio. Se on ohut kerros saman paikallisen taustakomponentin päällä, jota sekä graafinen käyttöliittymä että CLI käyttävät (HTTP `127.0.0.1:5000`:ssa), joten toiminta on identtistä kaikilla kolmella käyttöliittymällä.

## Asennus

Asennus tapahtuu kahdessa vaiheessa: ensin asennetaan Chloros-työpöytäpaketti (se tarjoaa käsittelytaustan ja laitteiston ajoympäristöt), sitten Python-paketti.

**Vaihe 1 — Asenna Chloros.** Windows: suorita työpöytäasennusohjelma (oletuspolku `C:\Program Files\MAPIR\Chloros\`) [Lataa](download.md)-sivulta. Linux: asenna `.deb`-paketti ([Linux:n asennus](linux/linux-installation.md)).**Vaihe 2 — Asenna SDK** (Python 3.7+):

```bash
pip install chloros-sdk
```

Et ehkä edes tarvitse pip:iä: jokainen asennusohjelma sisältää vastaavan SDK-wheel-tiedoston. Windows-asennusohjelma asentaa sen automaattisesti järjestelmään Python; Linux `.deb` sijoittaa sen `/usr/lib/chloros/sdk/`:ään ja tulostaa tarkan `pip install --user`-komennon. PyPI päivitetään julkaisurakennuksissa, joten `pip install chloros-sdk` vastaa uusinta vakaata julkaisua.

**Vaihe 3 — Kirjaudu sisään kerran kutakin konetta kohti:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Kirjautumistiedot tallennetaan välimuistiin kohdassa `~/.chloros/` (molemmilla alustoilla). Kohdassa Windows voit kirjautua sisään vastaavasti työpöytäsovelluksen Käyttäjä<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">-välilehdeltä. SDK edellyttää maksullista Chloros+-sopimusta — katso alla oleva [Lisenssivaatimus](#license-requirement).

| Vaatimus | Tiedot |
| --- | --- |
| **Chloros asennettuna** | Windows: työpöytäasennusohjelma; Linux: `.deb`-paketti (sisältää taustaprosessin binääritiedoston) |
| **Python** | 3.7 tai uudempi (kehitetty/testattu versiossa 3.10) |
| **Käyttöjärjestelmä** | Windows 10/11 64-bittinen, Ubuntu 22.04 LTS tai uudempi, tai NVIDIA Jetson (JetPack 6) |
| **Lisenssi** | Aktiivinen Chloros+-kirjautuminen, mikä tahansa maksullinen taso (Copper tai uudempi) |

## 60 sekunnin voitto

Yhdellä kutsulla luodaan projekti, tuodaan kansio, määritetään käsittely ja suoritetaan prosessiketju — taustapalvelu käynnistyy automaattisesti, jos sitä ei ole jo käynnissä:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(Linux:ssa käytä Linux-polkuja: `/home/user/drone_images/flight001`. SDK toimii identtisesti molemmilla alustoilla.)

Käsitteletkö LATTICE-tallennuskansiota? Käytä LATTICE-yhteensopivaa kääreohjelmaa — se soveltaa oikeita oletusasetuksia (ei paneelikohteen tunnistusta, vakiomuotoinen debayer):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — täysi prosessiketjun hallinta

Kaikkeen, mikä on pidempää kuin yksirivinen komento, käytä `ChlorosLocal`:ää. Se käynnistää taustaprosessin ensimmäisellä käyttökerralla (`auto_start_backend=True`), luo ja määrittää projektit, valvoo etenemistä ja palauttaa suorituksen jälkeisen yhteenvedon.

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

{% hint style="info" %}
Käytä oletusarvoista `http://127.0.0.1:5000`:ää sen sijaan, että korvaisit sen `localhost`:llä — komennossa Windows `localhost` ratkaistaan ensin `::1`:ksi ja se vie noin 2 sekuntia pyyntöä kohti, kun taustapalvelu tukee vain IPv4:ää.
{% endhint %}

Käytä sitä kontekstinhallinta-objektina, jotta puhdistus tapahtuu varmasti:

```python
import chloros_sdk

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

`configure()` hyväksyy seuraavat avainsanat: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` ja `custom_settings`. Tärkeimmät arvot:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

LATTICE-spesifiset säätimet (`input_level`, `radiometric_output`, `array_alignment*`-perhe) on dokumentoitu täydellisine arvo taulukoineen [SDK-viitteessä](reference/sdk-reference.md#supported-values).

### Edistymisen seuranta

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Suorituksen jälkeisen yhteenvedon lukeminen — ja tyhjien suoritusten havaitseminen

Suorituksen päätyttyä `process()` liittää taustaprosessin käsittelyyhteenvedon tiedostona `result["summary"]`. Jokainen `summary["hints"]`-tiedoston merkintä on kokonainen lause, joka selittää kaikki huomionarvoiset seikat — esimerkiksi miksi ajo ei tuottanut tulosta — ja jokainen vihje lähetetään uudelleen tiedostona Python `UserWarning`, joten tyhjät ajot diagnosoituvat itsestään, vaikka et koskaan tarkastelisi sanakirjaa:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` ei synny, kun suoritus ei tuota kuvia.** Tämä on ainoa kohta, jossa SDK ja CLI eroavat toisistaan tarkoituksellisesti: `chloros-cli process` käsittelee tilannetta ”tuotteita pyydettiin, mutta yhtään ei kirjoitettu” virheenä ja lopettaa nollasta poikkeavalla tuloksella, kun taas SDK palaa normaalisti ja raportoi tilanteen `summary`:n / vihjeiden kautta. Jos putkistosi pitäisi pysähtyä tyhjän ajon yhteydessä, tarkista asia itse — tarkista `summary` (tai laske projektikansiossa olevien tiedostojen määrä) sen sijaan, että luottaisit poikkeukseen.
{% endhint %}

## Smart Connect — reaaliaikainen laitteisto

Kolme apuohjelmaa avaa pysyviä istuntoja taustapalvelimen laitepoolissa — samassa poolissa, jota käyttöliittymä käyttää, joten SDK-skriptit toimivat rinnakkain työpöytäsovelluksen kanssa ilman, että ne kilpailevat sarjaporttien tai verkkokaistanleveyden käytöstä. Kaikki kolme käynnistävät automaattisesti paikallisen taustapalvelimen, jos sellaista ei ole käynnissä.

### Yksi LATTICE-kamera — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### Synkronoitu kamerajärjestelmä — `connect_array`

`connect_array` on suositeltu lähtökohta monikamerajärjestelmille. Se suorittaa saman älykkään valmisteluvirran kuin graafinen käyttöliittymä: verkkoanalyysi, synkronointitason automaattinen valinta, PTP-aikasynkronointi, kamerakohtainen pikselimuodon valinta, AE-alustaminen ja GPIO-laukaisun virittäminen. **Ensimmäinen sarjaliitäntä on päälaite** (se laukaisee laitteiston laukaisupulssin); muut ovat alilaitteita.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Lisää `smart=True` mihin tahansa matriisikuvausasetukseen, jotta automaattinen valotus ehtii vakiintua kaikissa kameroissa ennen laukaisua. Tallennustiloista (Yksittäinen / Jatkuva / Intervalli / Nopein), tallentimista, sarjakuvauksesta videoksi ja matriisin kohdistuksesta katso [SDK-viite](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### DAQ-valosensori — `connect_daq_sensor`

Ilman argumentteja `connect_daq_sensor()` tunnistaa siirtotavan älykkäästi (etuoikeusjärjestys: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Jokainen kehys sisältää 135 pisteen `spectrum`-arvon (W/m²/nm kalibroituna), `is_saturated`-lipun sekä CIE `x`, `y`, `z`. Jos haluat määrittää tietyn anturin tai siirtotavan — mikä on luotettava valinta isäntäkoneissa, joissa on useita verkkoliitäntöjä ja joissa Ethernetin automaattinen tunnistus saattaa jättää toimivan DAQ-E:n huomaamatta ensimmäisellä yrityksellä — välitä yksi nimenomainen vihje:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Huomaa, että kapasitanssikorjausprofiilit (`cap_id`) **eivät** ole SDK-säädin – valitse ne sen sijaan `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Tallennetut projektit — `open_project`

Tallennettu Chloros-projekti säilyttää siihen liitetyt laitteet (`cameras.json` + `sensors.json` sekä `project.json`), ja `chloros_sdk.open_project(path)` voi yhdistää kaiken uudelleen kerralla ja ohjata tallennuksia laitenimien perusteella. Katso viitteestä [Projektiautomaatio](reference/sdk-reference.md#project-automation--chlorosproject).

## Mitä pelkkä pip-asennus tarjoaa

Tarkista moduulitason saatavuusliput ennen laitteistopintojen käyttöä:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Isäntäkoneella, jossa on **vain** `pip install chloros-sdk` eikä Chloros-työpöytäpakettia:

* `ChlorosLocal`, `process_folder` ja `process_lattice_capture` **eivät** toimi — ne tarvitsevat taustaprosessin, joka toimitetaan työpöytäasennusohjelman mukana.
* Smart-connect-apuohjelmat (`connect_camera`, `connect_array`, `connect_daq_sensor`) ovat puhtaita HTTP-asiakasohjelmia, joten ne toimivat toisella koneella sijaitsevan taustapalvelimen kanssa — mutta mukana toimitetut taustapalvelimet sitoutuvat vain loopback-liitäntään, joten sinun on itse ohjattava portti eteenpäin (esim. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) ja välittää `backend_url="http://127.0.0.1:5000"`:n yhdessä `auto_start_backend=False`:n kanssa. Katso [Etä-backend-tila](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Suoraan laitteistoon liittyvät LATTICE-luokat (`LatticeCamera`, `CameraPool`, …) voidaan tuoda, mutta ne vaativat Arena SDK -ajoympäristön työpöytäasennuksesta — ilman sitä `CAMERA_AVAILABLE` on `False`.
* `daq_sdk` (suorat DAQ-luokat) toimitetaan työpöytäasennuksen mukana, ei PyPI-paketin mukana, joten `DAQ_AVAILABLE` on `False` pelkästään pip-ympäristössä — ohjaa DAQ-antureita sen sijaan `connect_daq_sensor()`:n kautta (tunneloidun) taustapalvelimen kautta.

## Lisenssivaatimukset

SDK:n käyttö edellyttää aktiivista Chloros+-kirjautumista millä tahansa maksullisella tasolla — **Copper tai korkeampi**(Copper / Bronze / Silver / Gold); ilmaisella Iron-tasolla ei ole SDK/CLI-käyttöoikeutta. Valvonta tapahtuu**palvelinpuolella**: jokaisessa SDK-pyynnössä on oltava sekä aktiivinen istunto että maksullinen tilaus, muuten taustajärjestelmä palauttaa virheen `403` / `PLAN_UPGRADE_REQUIRED` (joka palautetaan koodina `ChlorosLicenseError` koodin `ChlorosLocal` toimesta ja koodina `ChlorosConnectError` koodin `connect_*` aputoimintojen toimesta). Kirjautunut ulos kutsuja saa sen sijaan virhekoodit `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — `chloros-cli login`:n uudelleenkäynnistäminen korjaa ensimmäisen tapauksen, mutta ei toista.

Offline-käyttö toimii suunnitelman armonaikana: käyttöoikeustaso luetaan palvelimen vahvistusvälimuistista (5 minuuttia) tai allekirjoitetusta, laitteeseen sidotusta lisenssivälimuistista (30 päivää kuukausisuunnitelmissa; vuosittaisissa tilauksissa tilauksen voimassaolon loppuun asti). Kun armonaika päättyy, tilaus muuttuu ilmaiseksi ja SDK-käyttö keskeytyy, kunnes laite muodostaa yhteyden palvelimeen kerran. `chloros-cli status` pysyy käytettävissä ilmaisella tasolla, joten syy on aina näkyvissä. Katso [Chloros+ Kirjautuminen](chloros+-login.md).

## Poikkeukset

Käsittele perusluokkaa, jotta voit käsitellä ”kaikki Chloros-virheet”:

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

Kaikki putkistopoikkeukset (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) johtuvat poikkeuksesta `ChlorosError`. Yksi huomioitava seikka: `ChlorosConnectError` — aiheutuu ainoastaan virheistä `connect_camera` / `connect_array` / `connect_daq_sensor` — johtuu tavallisesta `Exception`:stä, **ei** `ChlorosError`:stä, joten `except ChlorosError` ei tunnista sitä. Koko hierarkia löytyy [SDK-viitteestä](reference/sdk-reference.md#exceptions).

## Katso myös

* [SDK-viite](reference/sdk-reference.md) — täydellinen API-pinta, optimoitu tekoälyavustajille.
* [CLI-viite](reference/cli-reference.md) — jokainen CLI-alikomento vastaa yhtä SDK-kutsua.
* [Lataa](download.md) — asennusohjelmat Windows:lle ja Linux:lle.
