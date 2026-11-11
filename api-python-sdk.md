# API: Python-SDK

{% hint style="info" %}
**Etsitkö täydellistä API:ta?** Tämä sivu on käytännönläheinen opas. Kaikki julkiset luokat, metodit, tarkat allekirjoitukset ja kopioitavat esimerkit löytyvät [SDK-viitteestä](reference/sdk-reference.md), joka on optimoitu tekoälyavustajille.**Käytätkö tekoälyavustajaa?** Liitä tämä URL-osoite chattiin, jotta saat käyttöösi täydellisen ja ajantasaisen Chloros 1.3.0 -API:n:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Tämän oppaan jokainen sivu on saatavilla raakamuodossa (markdown) sen pienillä kirjaimilla kirjoitetulla slug-tunnuksella + `.md`, ja koko opas on indeksoitu osoitteessa `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` PyPI:ssä) ohjaa kaikkea, mitä työpöytäsovellus voi tehdä Python-kielellä: kuvien eräkäsittelyä, LATTICE-kameran ja -anturijärjestelmän reaaliaikaista ohjausta, DAQ-valosensorisessioita sekä tallennettujen projektien automatisointia. Se on ohut kerros saman paikallisen taustapalvelimen päällä, jota sekä graafinen käyttöliittymä että komentoriviliittymä (HTTP) käyttävät (`127.0.0.1:5000`), joten toiminta on identtistä kaikilla kolmella käyttöliittymällä.

## Asennus

SDK:n mukana toimitetaan Chloros-työpöytäpaketti, joka sisältää myös käsittelytaustapalvelimen ja laitteiston ajoympäristöt, ja jonka versio vastaa aina SDK:n versiota.

**Vaihe 1 — Asenna Chloros.** Windows: suorita työpöytäasennusohjelma (oletuspolku `C:\Program Files\Chloros\`) [Lataa](download.md)-sivulta. Linux: asenna `.deb`-paketti ohjeiden `sudo apt install ./chloros_1.3.0_<arch>.deb` mukaisesti ([Linux-asennus](linux/linux-installation.md)).**Vaihe 2 — Tarkista SDK** (Python 3.7+). Yleensä mitään ei tarvitse asentaa:

* **Windows:** asennusohjelma asentaa mukana toimitetun SDK-wheel-tiedoston järjestelmän Pythoniin (jos Pythonia ei ole asennettu, tämä vaihe ohitetaan).
* **Linux:** `.deb` asentaa järjestelmän `python3` SDK:n `/usr/lib/python3/dist-packages`:ään. Paketti suosittelee seuraavia vaatimuksia: `python3-numpy` ja `python3-requests`. `apt install` sisältää ne, mutta `dpkg -i` ei.
* **Virtualenv** ei näe järjestelmän paketteja; asenna mukana toimitettu wheel-tiedosto siihen: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` Linuxissa tai `chloros_sdk-*.whl` Windowsin asennuskansion `sdk`-kansiossa (oletuksena `C:\Program Files\Chloros\sdk\`).

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

Koneella, jolla ei ole Chloros-pakettia ja jonka SDK-sovellus ohjaa toisen koneen taustapalvelinta (katso [Mitä pelkkä pip-asennus tuottaa](#what-a-pip-only-install-gets)), asenna se PyPI:stä: `pip install chloros-sdk`. PyPI päivitetään julkaisujen yhteydessä, joten se vastaa uusinta vakaata versiota.

**Vaihe 3 — Kirjaudu sisään kerran kutakin konetta kohti:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Kirjautumistiedot tallennetaan välimuistiin tiedostossa `~/.chloros/` (molemmilla alustoilla). Windowsissa voit kirjautua sisään vastaavasti työpöytäsovelluksen Käyttäjä-<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">-välilehden kautta. SDK vaatii maksullisen Chloros+-tilauksen — katso [Lisenssivaatimus](#license-requirement) alla.

| Vaatimus | Tiedot |
| --- | --- |
| **Chloros asennettuna** | Windows: työpöytäasennusohjelma; Linux: `.deb`-paketti (sisältää taustapalvelimen binääritiedoston ja SDK:n) |
| **Python** | 3.7 tai uudempi (kehitetty/testattu versiolla 3.10) |
| **Käyttöjärjestelmä** | Windows 10/11 64-bittinen, Ubuntu 22.04 LTS tai uudempi tai NVIDIA Jetson (JetPack 6) |
| **Lisenssi** | Voimassa oleva Chloros+ -kirjautumistunnus, mikä tahansa maksullinen taso (Copper tai korkeampi) |

## 60 sekunnin voitto

Yhdellä kutsulla luodaan projekti, tuodaan kansio, määritetään käsittely ja suoritetaan prosessiketju — taustapalvelu käynnistyy automaattisesti, ellei sitä ole jo käynnissä:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

(Linuxissa käytä Linux-polkuja: `/home/user/drone_images/flight001`. SDK toimii identtisesti molemmilla alustoilla.)

Käsitteletkö LATTICE-tallennuskansiota? Käytä LATTICE-yhteensopivaa wrapperia – se soveltaa oikeita oletusasetuksia (ei paneelikohteen tunnistusta, vakiomuotoinen debayer):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` – täysi putkilinjan hallinta

Jos tarvitset enemmän kuin yhden rivin komennon, käytä `ChlorosLocal`. Se käyttää jo käynnissä olevaa taustapalvelinta (työpöytäsovelluksen) tai käynnistää asennetun ensimmäisellä käyttökerralla (`auto_start_backend=True`; Windowsissa Chloros-asennuskansion `backend`-kansiosta, oletuksena `C:\Program Files\Chloros\backend\`), luo ja määrittää projektit, valvoo etenemistä ja palauttaa suorituksen jälkeisen yhteenvedon.

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
Käytä oletusarvoista `http://127.0.0.1:5000`:ää sen sijaan, että korvaisit sen `localhost`:llä — Windows-ympäristössä `localhost` tulkitaan ensin `::1`:ksi, ja se vie noin 2 sekuntia pyyntöä kohti, kun käytetään pelkästään IPv4-taustapalvelinta.
{% endhint %}

Käytä sitä kontekstinhallintana. Lohkon poistuminen lopettaa tämän instanssin taustapalvelimen käytön, ei itse taustapalvelinta: se pysyy käynnissä seuraavaa skriptiä varten ja pysähtyy itsestään, kun sitä ei ole käytetty 30 minuuttiin. `shutdown_backend()` lopettaa tämän instanssin käynnistyksen.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` hyväksyy seuraavat avainsanat: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` ja `custom_settings`. Tärkeimmät arvot:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

LATTICE-spesifiset säätimet (`input_level`, `radiometric_output` sekä `array_alignment*`-perhe) on dokumentoitu täydellisine arvo-taulukoineen [SDK-viitteessä](reference/sdk-reference.md#supported-values).

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

Suorituksen päätyttyä `process()` liittää taustaprosessin käsittelyyhteenvedon nimellä `result["summary"]`. Jokainen `summary["hints"]`:n merkintä on täydellinen lause, joka selittää kaikki huomionarvoiset seikat — esimerkiksi miksi ajo ei tuottanut lainkaan tulosta — ja jokainen vihje lähetetään uudelleen myös Python-muodossa `UserWarning`, joten tyhjät ajot diagnosoituvat itsestään, vaikka et koskaan tarkastelisikaan sanakirjaa:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` ei aiheuta poikkeusta, kun suoritus ei tuota kuvia.** Tämä on ainoa kohta, jossa SDK ja CLI eroavat toisistaan tarkoituksellisesti: `chloros-cli process` käsittelee tilannetta ”tuotteita pyydettiin, mutta yhtään ei kirjoitettu” virheenä ja lopettaa nollasta poikkeavalla tuloksella, kun taas SDK palaa normaalisti ja raportoi tilanteen `summary`:n / vihjeiden kautta. Jos prosessiketjusi pitäisi pysähtyä tyhjän ajon yhteydessä, tarkista asia itse — tarkista `summary` (tai laske projektikansiossa olevien tiedostojen määrä) sen sijaan, että luottaisit poikkeukseen.
{% endhint %}

## Smart Connect — reaaliaikainen laitteisto

Kolme apuohjelmaa avaa pysyviä istuntoja taustapalvelimen laitteistopoolissa — samassa poolissa, jota CLI, muut skriptit ja käyttöliittymän kameravälilehdet käyttävät, joten SDK-skriptit toimivat rinnakkain niiden kanssa ilman, että ne kilpailevat sarjaporttien tai verkkokaistanleveyden käytöstä. Kaikki kolme käynnistävät paikallisen taustapalvelimen automaattisesti, jos sellaista ei ole käynnissä. GUI:n DAQ-paneeli ei kuulu pooliin: sulje valosensori GUI:ssa ennen kuin avaat sen skriptistä.

### Yksi LATTICE-kamera — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**Suhteelliset polut viittaavat skriptin suorituspaikkaan (muutettu versiossa 1.3.0).** Paikallisella taustapalvelimella suhteellinen `output_dir` (mukaan lukien oletusarvoinen `"output"`) — ja muut polut, jotka taustapalvelin avaa (`burst_dir`, `profile_path`, `save_path`, `mask_path`) — määritetään skriptisi työkansion suhteen, joten `cam.capture("output/")` kirjoittaa tiedostoon `<current folder>/output/`. Versiossa 1.2.0 asennettu taustapalvelin tulkitsi ne omassa asennuskansiossaan, mikä johti virheeseen ”Pääsy evätty”. Etätaustapalvelimen tapauksessa polut lähetetään sellaisina kuin ne on kirjoitettu.
{% endhint %}

### Synkronoitu taulukko — `connect_array`

`connect_array` on suositeltu lähtökohta monikamerajärjestelmille. Se suorittaa saman älykkään valmisteluvirran kuin graafinen käyttöliittymä: verkkoanalyysi, synkronointitason automaattinen valinta, PTP-aikasynkronointi, kamerakohtainen pikselimuodon valinta, AE-alustaminen ja GPIO-laukaisun aktivointi. **Ensimmäinen sarjanumero on päälaite** (se laukaisee laitteiston laukaisupulssin); muut ovat alilaitteita.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Lisää `smart=True` mihin tahansa taulukon tallennukseen, jotta automaattinen valotus ehtii vakiintua kaikissa kameroissa ennen laukaisua.

Uutuutena versiossa 1.3.0 `ArraySession` toimittaa sinulle myös synkronoidut kehykset suoraan ilman tiedostojen tallennusta:

* `arr.grab()` palauttaa yhden kehyksen kameraa kohti, kaikki yhdestä laukaisusta, numpy-taulukoina (`{serial: array}`); `.group_id` on jaettu laukaisu ja `.errors` nimeää kaikki pois jääneet kamerat.
* `arr.frame_stream(fps=5.0)` tuottaa yhden tällaisen joukon kutakin tikkiä kohti, reaaliaikaista käsittelyä varten omassa prosessiketjussasi.
* `arr.continuous("output/")` suorittaa taustapalvelimen oman jatkuvan tallennuksen (GUI:n jatkuva tila) ja palauttaa `ContinuousHandle`:n yhdessä `status()`:n kanssa, `wait()` ja `stop()` — käytä tätä tallennussilmukan sijaan, kun jokainen laukaisu on tallennettava suurella taajuudella.

Tietoja tallennustiloista (Yksittäinen / Jatkuva / Intervalli / Nopein), tallentimista, burst-to-video-toiminnosta ja matriisin kohdistuksesta löytyy [SDK-viitteestä](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### DAQ-valosensori — `connect_daq_sensor`

Ilman argumentteja `connect_daq_sensor()` tunnistaa siirtotavan automaattisesti (etuoikeusjärjestys: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Jokainen kehys sisältää 135 pisteen `spectrum`-arvon (W/m²/nm kalibroituna), `is_saturated`-lipun sekä CIE `x`, `y`, `z`. Jos haluat määrittää tietyn anturin tai siirtotavan, anna yksi nimenomainen vihje:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Huomaa, että cap-korjausprofiilit (`cap_id`) **eivät** ole SDK-säätimiä — valitse ne sen sijaan `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`:n kautta.

### Tallennetut projektit — `open_project`

Tallennettu Chloros-projekti säilyttää siihen liitetyt laitteet (`cameras.json` + `sensors.json` sekä `project.json`), ja `chloros_sdk.open_project(path)` voi yhdistää kameransa, matriisinsa ja valosensorinsa uudelleen kerralla sekä ohjata tallennuksia laitenimien perusteella. Valoanturit muodostavat yhteyden uudelleen käynnissä olevan Chloros-taustapalvelimen kautta tallennettujen asetustensa ja tallennuskapasiteettinsa kanssa. Katso viitteestä [Projektin automatisointi](reference/sdk-reference.md#project-automation--chlorosproject).

## Mitä pelkkä pip-asennus tarjoaa

Tarkista moduulitason saatavuusliput ennen laitteistopintojen käyttöä:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Isäntäkoneella, jossa on **vain** `pip install chloros-sdk` eikä Chloros-työpöytäpakettia:

* `ChlorosLocal`, `process_folder` ja `process_lattice_capture` **eivät** toimi — ne tarvitsevat työpöytäasennusohjelman mukana toimitettavan taustaprosessin.
* Smart-connect-apuohjelmat (`connect_camera`, `connect_array`, `connect_daq_sensor`) ovat puhtaita HTTP-asiakasohjelmia, joten ne toimivat toisella koneella sijaitsevan taustapalvelimen kanssa — mutta mukana toimitetut taustapalvelimet sitoutuvat vain loopback-liitäntään, joten sinun on itse ohjattava portti eteenpäin (esim. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) ja välitettävä `backend_url="http://127.0.0.1:5000"` yhdessä `auto_start_backend=False`:n kanssa. Katso [Etätaustapalvelintila](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Suoraan laitteistoon liittyvät LATTICE-luokat (`LatticeCamera`, `CameraPool`, …) tuodaan, mutta ne tarvitsevat Arena-SDK:n ajoympäristön työpöytäpaketista — ilman sitä `CAMERA_AVAILABLE` on `False`.
* DAQ-antureita ohjataan ainoastaan (tunneloidun) taustapalvelimen kautta — `connect_daq_sensor()` tai projektin `proj.sensors`; taustapalvelin avaa anturin omalla koneellaan.

## Tekoälyavustajat — `chloros-mcp`

SDK:hen sisältyy `chloros-mcp`, MCP-palvelin, jonka avulla Claude Desktop, Claude Code, Cursor ja muut MCP-asiakasohjelmat voivat ohjata Chlorosia: käsitellä kuvamateriaalia sekä käyttää LATTICE-kameroita, -anturiryhmiä ja DAQ-valosensoreita. Sen työkalut kattavat tämän SDK:n toiminnot. Se asennetaan SDK:n mukana, joten Claude Code -sovelluksen asennus tapahtuu yhdellä rivillä:

```bash
claude mcp add chloros -- chloros-mcp
```

Muut asiakasohjelmat: [Chlorosin käyttö AI-avustajien kanssa](ai-assistants.md). Asetukset ja kaikki työkalut: [MCP-palvelin (AI-avustajat)](reference/sdk-reference.md#mcp-server-ai-assistants).

## Lisenssivaatimukset

SDK:n käyttö edellyttää aktiivista Chloros+-kirjautumista millä tahansa maksullisella tasolla — **Copper tai korkeampi**(Copper / Bronze / Silver / Gold); ilmaisella Iron-tasolla ei ole SDK- tai CLI-käyttöoikeutta. Valvonta tapahtuu**palvelinpuolella**: jokaisessa SDK-pyynnössä on oltava sekä aktiivinen istunto että maksullinen tilaus, muuten taustapalvelu palauttaa virhekoodit `403` / `PLAN_UPGRADE_REQUIRED` (jotka `ChlorosLicenseError`-koodina tuotetaan `ChlorosLocal`-koodin avulla ja `ChlorosConnectError`-koodina `connect_*`-aputoimintojen avulla). Kirjautunut ulos kutsuja saa virheen `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — `chloros-cli login`:n uudelleenkäynnistäminen korjaa ensimmäisen tapauksen, mutta ei toista.

Offline-käyttö toimii palvelupaketin armonaikana: käyttöoikeustaso luetaan palvelimen vahvistusvälimuistista (5 minuuttia) tai allekirjoitetusta, laitteeseen sidotusta lisenssivälimuistista (30 päivää kuukausipaketeissa; vuosipaketeissa tilauksen voimassaolon loppuun asti). Kun armonaika päättyy, tilaus muuttuu ilmaiseksi ja SDK-käyttö keskeytyy, kunnes laite muodostaa yhteyden palvelimeen kerran. `chloros-cli status` pysyy saatavilla ilmaisella tasolla, joten syy on aina näkyvissä. Katso [Chloros+-kirjautuminen](chloros+-login.md).

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

Kaikki putkistopoikkeukset (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) ovat peräisin `ChlorosError`:stä. Yksi huomioitava seikka: `ChlorosConnectError` — aiheutuu ainoastaan virheistä `connect_camera` / `connect_array` / `connect_daq_sensor` — johtuu tavallisesta `Exception`:stä, **ei** `ChlorosError`:stä, joten `except ChlorosError` ei tunnista sitä. Koko hierarkia löytyy [SDK-viitteestä](reference/sdk-reference.md#exceptions).

## Katso myös

* [SDK-viite](reference/sdk-reference.md) — täydellinen API-rajapinta, optimoitu tekoälyavustajille.
* [CLI-viite](reference/cli-reference.md) — jokainen CLI-alikomento vastaa SDK-kutsua.
* [Chlorosin käyttö tekoälyavustajien kanssa](ai-assistants.md) — `chloros-mcp`-asennus- ja kehoteohjeet.
* [Lataa](download.md) — asennusohjelmat Windowsille ja Linuxille.
