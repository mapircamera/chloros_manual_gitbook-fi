# Chloros CLI Viite

**Versio:**

1.2.0**Luotu:**

29.7.2026 klo 19.19 ·**Päivitetty:**

30.8.2026**Kohderyhmä:** Optimoitu suurten kielimallien (LLM) käyttöön; ihmisen luettavissa.**Soveltamisala:** Kaikkikäyttöön tarkoitettu `chloros-cli`-alikomento, sisältäen vaihtoehdot ja kopioitavat esimerkit.

Tämä asiakirja on kattava viiteopas `chloros-cli`-komentorivityökalulle, joka toimitetaan MAPIR Chloros -sivuston mukana. Se on tarkoituksella kattava, jotta LLM (tai ihminen) voi koota minkä tahansa tuetun työnkulun alla olevista luetteloista tarkastelematta lähdekoodia.

Jos tarvitset vain tärkeimmät kohdat, siirry kohtaan:
- [Viiden minuutin pikaopas](#five-minute-quickstart)
- [LATTICE-kameran ensimmäisen kytkennän työnkulku](#lattice-camera-first-connect-workflow)
- [DAQ-anturin ensiliitäntätyönkulku](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Tallennustilat, tallentimet ja offline-uudelleenkäsittely](#capture-modes-recorders--offline-reprocess)

---

## Nimeämiskäytännöt

- Kaikkien komentojen etuliitteenä on `chloros-cli`. Windows -sivustolla binääritiedoston nimi on `chloros-cli.exe`; Linux /Jetson -sivustolla se on `chloros-cli`.
- Valinnaiset argumentit esitetään muodossa `--flag`. Pakolliset sijaintiparametrit esitetään ilman sulkeita.
- Jos oletusarvo on määritetty, lippua jättämällä käytetään kyseistä arvoa.
- CLI on kevyt HTTP-asiakasohjelma, joka käyttää Chloros-taustapalvelinta (Flask-palvelin osoitteessa `127.0.0.1:5000`). Useimmat komennot käynnistävät taustapalvelimen automaattisesti. `CHLOROS_BACKEND_URL=<url>` ohjaa **`lattice`**-,**`project`**ja**`daq pool-*`**-komentoryhmät etätaustapalvelimeen — ydinkomennot (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) kiinnitä `http://127.0.0.1:<port>` tarkoituksellisesti ja ohita se (IPv4-literaali välttää Windows:n `localhost`→`::1` -noin 2 sekunnin viiveen pyyntöä kohti). Katso [Ympäristömuuttujat](#environment-variables).
- Kaikissa SDK / CLI -kutsuissa vaaditaan Chloros+ -tilin kirjautuminen (suorita `chloros-cli login` kerran kutakin konetta kohti; tallennetaan välimuistiin `~/.chloros/`).
- Esimerkeissä käytetään polkuja Linux; osoitteessa Windows korvaa `/home/user/...` komennolla `C:/Users/.../...`.

---

## Ylätason yhteenveto

```
chloros-cli [global options] COMMAND [command options]
```

### Yleiset asetukset

| Lippu | Kuvaus |
| --- | --- |
| `--backend-exe PATH` | Ohita automaattisesti tunnistettu taustaprosessin suoritustiedosto. |
| `--port N` | Taustaprosessin HTTP-portti (oletus: `5000`). |
| `-v, --verbose` | Ota käyttöön yksityiskohtainen tuloste. |
| `--restart` | Pakota taustaprosessin uudelleenkäynnistys (lopettaa kaikki käynnissä olevat `backend_server.py`-prosessit). |
| `--version` | Tulosta versio (`Chloros CLI 1.2.0`). |
| `--help` | Näytä ylätason ohje. |

### Komentojen hakemisto

| Komento | Tarkoitus |
| --- | --- |
| [`process`](#chloros-cli-process) | Käsittele kansio, joka sisältää Survey3- tai LATTICE-kaappauksia päästä päähän. |
| [`login`](#chloros-cli-login) | Todentaa tämän laitteen Chloros+ -tilillä. |
| [`logout`](#chloros-cli-logout) | Tyhjennä välimuistissa olevat tunnistetiedot. |
| [`status`](#chloros-cli-status) | Näytä nykyinen lisenssi- ja todennustila. |
| [`export-status`](#chloros-cli-export-status) | Näytä Thread-4-viennin eteneminen reaaliaikaisesti `process`-suorituksen aikana. |
| [`language`](#chloros-cli-language) | Aseta tai näytä luettelo CLI näyttökielistä (38 tuettua kieltä). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Oletusprojektikansio (jaettu graafisen käyttöliittymän kanssa). |
| [`update`](#chloros-cli-update) | Tarkista ja asenna CLI-päivitykset (Linux /Jetson). |
| [`selftest`](#chloros-cli-selftest) | Järjestelmän diagnostiikka + savutestit. |
| [`time-sync`](#chloros-cli-time-sync) | PTP-grandmaster-tila / -ohjaus. |
| [`lattice`](#chloros-cli-lattice) | LATTICE-kameran ohjaus ja kuvaus (yli 45 alikomentoa). |
| [`daq`](#chloros-cli-daq) | DAQ-spektrisensorin ohjaus (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Avaa ja ohjaa tallennettua Chloros-projektia (kamerat + DAQ-laitteet). |

---

## Asennus

`chloros-cli` toimitetaan osana Chloros työpöytäasennusohjelmaa kaikilla tuetuilla alustoilla — erillistä CLI-latausta ei ole. Alustapaketin asentaminen lisää `chloros-cli`:n `PATH`:iin työpöytäsovelluksen ja sen ohjaamantaustaprosessiin, jota se ohjaa.

Uusimmat lataukset: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Asennusohjelma sisältää myös käteviä käynnistysskriptejä (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`), jotka avaavat käyttövalmiin CLI-komentotulkin; niitä käsitellään [CLI-käyttöoppaassa](../CLI.md), eikä niitä toisteta tässä.

### Windows (.exe)

1. Lataa Windows-asennusohjelma lataussivulta.
2. Suorita `Chloros-Setup-x.y.z.exe` ja seuraa ohjatun asennuksen ohjeita. Oletusasennuskansio on `C:\Program Files\Chloros\` (CLI-tiedosto tallentuu kansioon `C:\Program Files\Chloros\cli\`, jonka asennusohjelma lisää PATH-muuttujaan).
3. Avaa uusi terminaali (`cmd.exe`, PowerShell tai Windows-terminaali), jotta päivitetty `PATH` valitaan käyttöön.

```powershell
chloros-cli --version
```

Asennusohjelma lisää automaattisesti `chloros-cli.exe` järjestelmään `PATH` ja sisällyttää siihen LATTICE-kameroiden käyttöön tarvittavan Arena-SDK-ajoympäristön.

### Linux amd64 (.deb)

Ubuntu 22.04 LTS:lle tai uudemmalle / Debian-pohjaisille x86_64-työasemille.

> **Ubuntu 20.04:ää ei tueta.** Paketin riippuvuuslista perustuu siihen,
> mihin taustaprosessi tosiasiallisesti linkittyy, ja siihen sisältyy `libc6 (>= 2.34)`;
> Focal toimittaa glibc 2.31:n. `apt` estää asennuksen sen sijaan, että antaisi sen epäonnistua
> suorituksen aikana.

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

.deb-paketti asentaa:
- `chloros-cli`:n versioon `/usr/bin/chloros-cli`
- Käännetyn taustakomponentin versioon `/usr/lib/chloros/chloros-backend`
- Arena-SDK-ajoympäristön (LATTICE-kameroille)
- Kohinanpoistomallit, kalibrointipaketit ja päivityskanavan asetukset

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Sama rakenne kuin amd64 .deb-tiedostossa, mutta CUDA-rakennus on optimoitu Jetson Orin / Orin NX / Orin Nano -laitteille.

### Todennus kerran kutakin laitetta kohti

Jokaisella alustalla vaaditaan kertaluonteinen kirjautuminen osoitteeseen Chloros+ ennen kuin SDK / CLI -kutsut toimivat:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Kirjautumistiedot tallennetaan välimuistiin tiedostossa `~/.chloros/user_session.json`.

### Varmista asennus

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **Vaaditaan Chloros+ -tilaus.**CLI vaatii voimassa olevan Chloros+ -sopimuksen.**Copper**on lähtötaso Chloros+ — jokaisella maksullisella Chloros+ -tasolla on pääsy CLI / SDK; vain ilmaisella**Iron**-tasolla ei ole. (Sopimus-ID-vastaavuus: `0`=Iron/free, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Päivitä osoitteessa [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Tätä alarajaa valvotaan taustapuolella, ei pelkästään CLI-sivustolla: SDK / CLI -merkitty pyyntö ilman maksullista tilausta hylätään virhekoodilla `403 PLAN_UPGRADE_REQUIRED` riippumatta siitä, tuleeko se `chloros-cli`:stä, Python SDK:stä vai itse kehitetyltä HTTP-asiakasohjelmalta. Uloskirjautunut käyttäjä saa sen sijaan virhekoodin `401 AUTH_REQUIRED` . Käyttö toimii offline-tilassa palvelun armonaikana (kuukausitilauksessa 30 päivää, vuositilauksessa voimassaolon loppuun asti) ja lakkaa, kun tämä aika umpeutuu; `chloros-cli status` jatkaa toimintaansa, jotta syy on näkyvissä (se on ainoa SDK / CLI-reitti, joka on vapautettu tasorajoituksesta — `GET /api/license-status`).

---

## Viiden minuutin pikaopas

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

Käsittele kuvakansio koko Chloros-prosessin läpi (kohteen tunnistus → kalibrointi → vinjetti → heijastavuus → indeksin vienti).

### Yhteenveto

```
chloros-cli process INPUT [OPTIONS]
```

### Sijaintiparametrit

| Parametri | Kuvaus |
| --- | --- |
| `INPUT` | Polku syöttökansioon, joka sisältää tiedostot `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) tai `.dng`-tiedostoja. |

### Yleiset asetukset

| Asetus | Oletus | Kuvaus |
| --- | --- | --- |
| `-o, --output PATH` | uusi aikaleimattu kansio oletusprojektipolun alla (`~/Chloros Projects`, ellei toisin määritetä) | Luotava tai uudelleen käytettävä projektikansio. Jos kansiossa on jo `project.json`, luodaan sen sijaan `_1`/`_2`-sisarkansio sen sijaan, että se korvattaisiin. |
| `-n, --project-name NAME` | automaattinen (aikaleima) | Projektin nimi. |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` käyttää Chloros+ neuroverkko-debayeria; hitaampi mutta laadukkaampi. |
| `--vignette / --no-vignette` | `--vignette` | Vignettikorjaus. |
| `--reflectance / --no-reflectance` | `--reflectance` | Heijastavuuskalibrointi (käyttää paneelikohdetta, jos sellainen löytyy, NIST-sarjakohtainen kalibrointi LATTICE-laitteille). LATTICE-monispektrikuvauksessa tämä toimii myös heijastavuuden **tuote**-kytkimenä — katso [Tuotekohtaiset vientikytkimet](#per-product-export-toggles-lattice-multispektraali). |
| `--ppk` | pois | Sovella PPK GNSS-korjauksia sivutiedostoista. |
| `--exposure-pin-1 MODEL` | pois | Kiinnitä Survey3 kaksikamerajärjestelmän &quot;pin-1&quot;-mallin. |
| `--exposure-pin-2 MODEL` | pois | Kiinnitä &quot;pin-2&quot;-malli. |
| `--recal-interval SECONDS` | 0 | Pakota kalibrointilaskelmien uudelleenkäynnistys N sekunnin välein tallennusajan aikana. |
| `--timezone-offset HOURS` | local | Ohita tulostusmetatietoihin sisällytetty aikavyöhykkeen siirtymä. |
| `--format FORMAT` | `TIFF (16-bit)` | Yksi seuraavista: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | ei mitään | Kasvillisuusindeksit (`NDVI`, `NDRE`, `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …). |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Pakottaa LATTICE TIFF -tiedostojen käsittelyputken aloituskohdan (tämä ei vaikuta Survey3 .raw -tiedostoihin). Lisäksi pakotie, jonka avulla **raakaa dataa sisältämätön** tallenne voidaan käsitellä kokonaan — katso [Miltä tallennuskansio näyttää ](#miltä-kaappauskansio-näyttää). |
| `--debayered / --no-debayered` | päällä | Tuottaa lineaarisen debayeroidun tuloksen (`Debayered_Images`). Katso [Tuotekohtaiset vientikytkimet](#tuote-kohtaiset-vienti-kytkimet-toggles-lattice-multispectral). |
| `--preview / --no-preview` | päällä | Lähettää näytön esikatselun (`Preview_Images`): RGB = valkotasapaino (DAQ-valonlähde, jos saatavilla, muuten harmaa maailma) + gamma; multispec = väärävärinen venytys. |
| `--radiance / --no-radiance` | päällä | Lähettää float32-tyyppisen säteilyvoimakkuuden (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Viite LATTICE-heijastavuustuotteelle: `auto` = QA-tarkastuksen läpäissyt kehyksen sisäinen kohde on absoluuttinen viite, DAQ-alas(ρ = π·L/E) varavaihtoehto; `target` = tiukka (ei DAQ-korvausta); `daq` = DAQ-määräysvaltainen. Katso [Tuotteittaiset vientikytkimet](#tuote-kohtaiset-vientikytkimet-lattice-multispektraalinen). |
| `--target-reflectance-dir DIR` | ei mitään | Yksikkökohtaisten **mitattujen** kohteen heijastavuusskannausten hakemisto (`<serial>.csv`); käyttää nimellisiä T3/T4P-spektrejä, jos skannausta ei löydy. |
| `--array-alignment / --no-array-alignment` | päällä | LATTICE-matriisit: sovelletaan jokaisen kuvan `Chloros:Alignment*` XMP-tiedostoon merkittyä moduulien välistä kohdistusta kaikkiin käsiteltyihin tuotteisiin (debayering / esikatselu / säteilyvoimakkuus / heijastavuus / indeksi). Ei vaikutusta kuviin, joissa tunnisteita ei ole. |
| `--array-alignment-crop / --no-array-alignment-crop` | rajaus | Rajaa kohdistetut vientitiedostot matriisin yhteiseen-päällekkäisyysalueelle, jotta kaikki moduulit jakavat yhden peittoalueen; `--no-…` säilyttää koko anturin kuvapinnan (musta täyttö lähteen ulkopuolella). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Uudelleennäytteenotto kohdistusmuunnokselle. `nearest` säilyttää tarkat lähteen digitaaliset numerot (ei pikselien välistä radiometristen arvojen sekoittumista). |

### Kohteen tunnistuksen asetukset

| Lippu | Kuvaus |
| --- | --- |
| `--min-target-size PIXELS` | Ilmaisimen paneelikohteen vähimmäiskoko (px). |
| `--target-clustering 0-100` | Klusteroinnin herkkyys. |
| `--target / --targets` | Käsittele syöttökansiota pelkästään kohdepaneelina (ohita kartoitustunnistus). |

### Esimerkkejä

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### Tuotekohtaiset vientikytkimet (LATTICE-monispektrinen)

LATTICE-käsittely haarautuu **kaikkiin soveltuviin tuotteisiin yhdellä kerralla**. Neljätyyppistä kytkintä — `--debayered`, `--preview`, `--radiance`, `--reflectance` — ovat kaikki**oletusarvoisesti päällä**; käytä muotoa `--no-<type>`, jos haluat poistaa yhden. RGB-pääkamerat tuottavat aina vain debayeroitua kuvaa + esikatselua (ei kaistakohtaista säteilyä/heijastavuus), joten `--radiance`/`--reflectance` eivät vaikuta niihin. Kytkimet ohitetaan Survey3 `.raw`:n kohdalla (joka noudattaa standardia heijastavuus-/kohdepolkua). *(Vanha `--radiometric-output {reflectance,radiance,sensor-response}`-lippu **poistettiin** ja korvattiin näillä kytkimillä; `sensor-response`-tasoa ei enää ole.)*

| Tuote | Lähtö | Tarvitaanko DAQ-alasvirtausta? |
| --- | --- | --- |
| `--debayered` | Lineaarinen demosaikki (`Debayered_Images`). | Ei. |
| `--preview` | Esikatselu (`Preview_Images`): RGB = WB + gamma; multispec = väärävärinen venytys. | Ei. |
| `--radiance` | float32 W/m²/sr/nm täydellisestä radiometrisestä ketjusta (`Radiance_Images`). | Nro |
| `--reflectance` | uint16 heijastavuus ρ (`32768` = 1,0), Pix4D-valmis. | **Kyllä**, ellei sitä ankkuroi laadunvarmistuksen läpäissyt kehyksen sisäinen kohde (katso alla). |

`--reflectance-source` valitsee heijastavuusviitteen:**`auto`**(oletus) tekee laadunvarmistuksen läpäisseestä kuvan sisällä olevasta kohteesta**absoluuttisen viitteen**— kohteeseen ankkuroidut empiiriset linjaketjut pisteytetään-arvioidaan pidätetyillä paneeleilla ja mitattu voittaja otetaan käyttöön — palataan DAQ:n alaspäin suuntautuvaan jakolinjaan (ρ = π·L/E), kun kohdetta ei ole läsnä tai laadunvarmistus epäonnistuu;**`target`**on tiukka (ei DAQ-korvausta);**`daq`**valitsee DAQ:n määräävän käyttäytymisen. Kohteen geometria (ArUco / kiinteä ROI / kaistale) tulee projektin kohdekonfiguraatiosta; `--target-reflectance-dir DIR` säilyttää yksikkökohtaiset**mitatut** skannaukset (`<serial>.csv`), jotka haetaan kohdeyksikön sarjanumeron/QR:n avulla, ja nimelliset T3/T4P-spektrit toimivat varavaihtoehtona.

DAQ-heijastuspolku määrittää **aikaleimalla täsmäytetyn alaspäin suuntautuvan säteilyn**automaattisesti tallennetusta**`.daq`**(DAQ-U/M/E)**tai DAQ-M:n omasta `.csv`**-tiedostosta, joka löytyy kuvien yhteydessä. Jos kamera- tai DAQ-kalibrointipakettia ei ole tallennettu paikalliseen välimuistiin, prosessiketju**hakee sen automaattisesti AWS:stä** ensimmäisellä käyttökerralla (vaatii internet-yhteyden kerran; tallennetaan välimuistiin nimellä `~/.chloros/`).

#### Heijastuspikseleiden lukeminen (Pix4D / Metashape / omat skriptisi)

Heijastavuus tallennetaan kokonaislukuna (DN), ja **se DN-arvo, joka tarkoittaa ρ = 1,0, riippuu lähdekamerasta**:

| Lähde | ρ = 1,0 on | Miten tunnistaa |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (varaa ρ 2,0:aan asti) | Tiedostoon on merkitty XMP-tunniste `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (rajoitettu arvoon ρ 1,0) | Ei `Chloros:*` XMP-tunnisteita — tämä puuttuminen *on* signaali. |

**Lue `Chloros:PixelScale` ja jaa sillä** sen sijaan, että olettaisit vakioarvon. Tunniste on määritelty uint16-alueella, joten se säilyy `32768`:na myös skaalausmuutoksia tekevissä tulostusmuodoissa — `TIFF (16-bit)`, `PNG (8-bit)`, `JPG (8-bit)` ja `TIFF (32-bit, Percent)` ovat kaikki itsestään selviä (normalisoi tallennettu tietotyyppi ensin takaisin uint16:ksi: ×257 8-bittisestä, ×65535 float-tyypistä).

> **Yhdessä tapauksessa skaalaa ei ole, mikä on tarkoituksellista.** Kun 8-bittinen lähdekuva (BayerRG8) kirjoitetaan 8-bittisenä TIFF, prosessointiputki *rajaa* arvot välille 0..255 skaalaamisen sijaan, joten kaikki arvot, jotka ovat suurempia kuin ρ≈0,008 tasoittuu arvoon 255, eikä tiedostoa kuvata minkään skaalauksen avulla. Chloros jättää tarkoituksella pois sekä `Chloros:PixelScale`- että `MicaSense:RadiometricCalibration`-tupelin kyseisessä kohdassa ja kirjaa syyn lokiin. **Jos tunniste puuttuu LATTICE-heijastustiedostosta puuttuu, älä oleta mittakaavaa — vie tiedosto uudelleen 16- tai 32-bittisenä** sen sijaan, että jakaisit pikseleitä, joita ei ole koskaan voitu jakaa.

#### EXIF-tiedot siirretään vientiin

`process` kopioi lähdekuvan **GPS-lohkon ja sen ExifIFD:n** jokaiseen tuotteeseen, joten
vienti sisältää `FocalLength`-, `FNumber`-, `ExposureTime`-, `ISO`- ja `DateTimeOriginal` ja
`CameraSerialNumber` georeferenssin ohella.

**`FocalLength` ei ole valinnainen fotogrammetrian kannalta.** Pix4D laskee maanpinnan näytteenottovälin
polttovälistä ja korkeudesta; jos tunniste puuttuu, se käyttää oletusarvoisesti täysin virheellistä mittakaavaa. Yhdessä
49-kuvauslennolla appelsiinitarhalla puuttuva tunniste muutti 411 m × 160 m:n alueen rekonstruoiduksi
47,8 km × 13 km:n alueeksi – 455 MP:n ortokuvaksi, jossa oli pääosin nodata-arvoja, mikä tulkittiin aluksi laatoitusongelmaksi ja
BigTIFF-ongelmaksi, ennen kuin kukaan tarkisti GSD:n. Jos ortokuvasi tulostuu epäuskottavassa
mittakaavassa, aja ensin `exiftool -FocalLength` vietyyn tuotteeseen.

Kopio ei ole tarkoituksella **`-all:all`**: IFD0:n rakenteelliset tunnisteet rikkovat LATTICE-tulostuksen, kun
kopioidaan, ja `ExifImageWidth` / `ExifImageHeight` on suljettu pois, koska ne kuvaavat
*lähdettä* kaappausta — muuten vienti, jonka kokoa on joskus muutettu, sisältäisi mitat,
jotka ovat ristiriidassa sen oman rasterin kanssa. XMP kirjoitetaan suoraan sen sijaan, että se kopioitaisiin, koska ExifTool
hävittää samasta kutsusta peräisin olevat XMP-tunnisteet, kun XMP-lohko kopioidaan (mikä poistaisi MAPIR
kalibrointitunnisteet).

### Mihin tulosteet tallennetaan

Tulostiedostot tallennetaan **projektikansioon, ryhmiteltynä kameran ja sen jälkeen tiedostomuodon mukaan**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

LATTICE-kameran kansio on `LATT-<sensor>-<lens>-F<filter>` (vastaa kuvan EXIF-tietoja
`Model`) ja `<model>_<filter>` (Survey3) — kaksi kameraa, joilla on sama kenno ja suodatin, mutta jotka eroavat
objektiivin osalta, säilytetään erillisissä hakemistopuissa, koska vinjetointi, kuvakulma ja vääristymä eroavat toisistaan. Tiedostomuoto
-kansio on nimeltään `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` tai `tiff32` kunnes
`TIFF (32-bit, Percent)`.

> **Jokainen viety tuote säilyttää LÄHDE-tiedoston nimen.** Radiance-vienti tiedostosta
> `capture_…_raw.tif` on edelleen nimeltään `capture_…_raw.tif` — se vain sijaitsee kansiossa
> `tiff32/Radiance_Images/`. **Kansio tunnistaa tuotteen, ei tiedostonimi**, joten
> `*radiance*.tif`:n avulla ei löydy mitään; käytä sen sijaan hakemistovastaavuutta.

### Valosensorin tallenteet — kalibroitu `.daq` + `.csv`

`process` käsittelee myös syöttökansiossasi olevat `.daq`-tallenteet, eikä se **tarvitse**
siihen mitään kuvamateriaalia: itsenäisesti käytetty DAQ-U / DAQ-M / DAQ-E, jota lennetään yksinään, riittää täydelliseen
tallennukseen, ja kansio, joka sisältää vain `.daq`-tiedostoja, on kelvollinen syöte.

DAQ-laite voidaan tallentaa **ilman** kalibrointia — juuri näin julkiset
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) tallentimet
(`record_daq.py`) toimivat oletuksena: ne kirjoittavat raakatiedot anturilukemista ja merkitsevät tiedostoon, jotta
Chloros hakeekyseisen anturin tehdaskalibroinnin **sarjanumeron perusteella** (ensin paikallisesta välimuistista,
sitten MAPIR-pilvestä) ja soveltaa sitä. `process` tallentaa tuloksen takaisin:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` sisältää yhden rivin kutakin lukemaa kohti: UTC-aikaleima, integraatioaika, kokonaisteho,
fotopinen/skotopinen lux, PPFD (ja sen sininen/vihreä/punainen jakautuma), huippuaallonpituus, sitten
koko spektri anturin omalla aallonpituusruudukolla. `.daq` tuodaan uudelleen ilman, että sitä
kalibroidaan toista kertaa.

Onnistuessaan ajo raportoi `Light-sensor products written: N (calibrated .daq + .csv)`.
Suluissa oleva teksti kuvaa sitä, mitä tosiasiallisesti kirjoitettiin, joten se on
`(RAW COUNTS — this sensor has no calibration bundle)` niputtamattomalle anturille ja
`(N calibrated, M raw counts)`, jos kansiossa on molempia. Taustapalvelimen omat
otsikot `[DAQ-EXPORT]` ja `[RUN-SUMMARY]` muodostuvat samalla tavalla — mikään
kolmesta ei voi kutsua raakaa vientiä kalibroiduksi.

DAQ-U / DAQ-M / DAQ-E-tallenne, jonka kalibrointipakettia ei voida hakea — olet
offline-tilassa tai kyseisellä anturilla ei ole kalibrointitietoja tiedostossa — **ohitetaan syyn kera** rivillä
`[DAQ-EXPORT]`, eikä sitä koskaan tallenneta ”kalibroituna” tiedostona, joka sisältää raakalukemia.
Yhdistä internetiin ja suorita uusi ajo. Syy on se, jonka lukija on tosiasiallisesti
määrittänyt kyseiselle tiedostolle (lukukelvoton skeema, ei kalibrointipakettia, kirjoitusvirhe), ja ajon
yhteenvedossa luetellaan **erillisiä** syitä — kaksikymmentä ohitettua tiedostoa yhden syyn vuoksi näkyy yhtenä
syyna, ei kaksikymmentä toistoa siitä.

#### DAQ-A-tallenteet viedään raakalukuna

**DAQ-A**-tuoteperhe on vanhempi kuin sarjanumeroittain toimiva niputusjärjestelmä, eikä sillä ole kalibrointinippua
haettavaksi — se kalibroidaan sen sijaan kentällä heijastavuuskohteen avulla, minkä
vuoksi sitä ei ole koskaan tarvittu. Näiden tallenteiden hylkääminen jätti käyttäjille
mahdottoman saada lukujaan ulos lainkaan, joten ne viedään **eri nimellä**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Erillinen tiedostonimi tiedoston sisällä olevan tunnisteen sijaan, koska tiedoston nimi on säilytettävä
sellaisenaan, kun se lähetetään sähköpostitse. `.csv`-otsikossa lukee
`raw spectral sensor counts (NOT irradiance)` ja varoitetaan, että arvot ovat vertailukelpoisia
**tiedoston sisällä** – mikä on juuri se, mihin kohdepohjainen kalibrointi niitä käyttää – eikä
anturien välillä. Teho riippuvaiset fotometriset sarakkeet (kokonaisteho, fotopinen ja
skotopinen lux, PPFD) kirjoitetaan **NULL**:ksi sen sijaan, että ne integroitaisiin laskentatuloksista, ja ajon
yhteenvedossa lukee `RAW COUNTS`, joten lokiin ”vietyä” tietoa ei voida tulkita säteilyvoimakkuudeksi.

Vanhat **v1.01 / v1.02**-tallenteet (joita DAQ-A-SD kirjoittaa) eivät sisällä lukukohtaista aikamerkintää,
vaan ainoastaan tiedoston kirjoitusajan. Kuva↔alasuuntaisen säteilyn täsmäytin hylkää ne edelleen —
kehyksen täsmäyttäminen kirjoitusajankohtaan johtaisi huomaamattomaan virheeseen — mutta vientiohjelma lukee ne, ja
CSV-tiedostossa tulostetaan `clock=daq_created_on`, joten tuote ilmoittaa, millä kellolla se toimii.

### Huomautuksia

- `process` tunnistaa automaattisesti, onko kansiosi tyyppiä Survey3, LATTICE vai sekatyyppinen.
- Edistymisvirta välitetään Server-Sent Events -tapahtumien kautta; CLI näyttää reaaliaikaisen edistymisen säikeittäin (Havaitseminen, Analysointi, Käsittely, Vienti).
- Linux- ja Jetson-laitteissa CLI tarkistaa swap-muistin ja saattaa antaa varoituksen ennen suurten kansioiden käsittelyä. Tekstuuritunnistava debayer asettaa myös automaattisesti GPU:n taajuusrajoituksen vähävirtaisille Jetson-laitteille (Nano, Orin Nano).
- Onnistuessaan suoritus ilmoittaa, kuinka monta kuvatuotetta se on tallentanut (`Image products written: N`).

#### Suoritus, joka ei tallenna kuvia, epäonnistuu

Jos pyysit tuotteita ja suoritus ei tallentanut **yhtään** — vain `project.json` ja
`calibration_data.json` — `process` tulkitsee tämän epäonnistumiseksi: se tulostaa
`Processing finished but wrote no image products.` ja **päättyy nollasta poikkeavalla arvolla**, joten skripti voi
havaita sen. Viestissä mainitaan projektikansio ja tavallisimmat syyt:

- syöttökansiota ei tunnistettu tallennukseksi (tarkista asettelu ja `--input-level`), tai
- jokainen pyydetty tuote ohitettiin, koska se ei sovellu kyseisille kameroille (esim. pyydettäessä
  säteilyvoimakkuutta/heijastavuutta kameroista, jotka tukevat vain RGB -kameroista).

Suorita komento uudelleen `--verbose`:llä ja tarkista taustaprosessin lokista `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`-rivit,
joissa selitetään kamerakohtaiset ohitukset, jotka eivät muuten pääse CLI:n tulostukseen.

Tarkoituksellinen pelkästään metatietoja käyttävä ajo — kaikki tuotteet pois päältä ja ilman `--indices`:ää — on silti
**onnistunut**, koska tyhjä kuvatulostus on siinä tapauksessa oikea tulos.

Samoin on **vain valosensorilla suoritettu ajokerta**: `.daq`-tallenteiden kansiossa ei ole määritelmän mukaan vietyä kuvamateriaalia
, ja suoritus arvioidaan sen sijaan sen kirjoittamien kalibroitujen `.daq`- ja `.csv`-tiedostojen perusteella.

---

## `chloros-cli login`

Todentaa tämä laite Chloros+ -pilvitilillä. Kirjautumistiedot tallennetaan turvallisesti välimuistiin tiedostoon `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Esimerkkejä

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (poistamalla osia salasanasta tai toistamalla osia siitä). Virhekoodin 401 sattuessa CLI yrittää automaattisesti uudelleen liittämällä `$$` takaisin ja sitten toistamatta jättämällä puolet salasanasta; jos uudelleenkäyttö onnistuu, se kirjaa sinut sisään ja tulostaa oikean yksinkertaisen lainausmerkin syntaksin, jota on käytettävä seuraavalla kerralla.

> **Päättömän laitteen tai skriptin käyttö: välimuistissa olematon istunto tarkoittaa interaktiivista kehotetta, ei nopeaa epäonnistumista.** Mikä tahansa taustaprosessia käynnistävä komento (`process`, `status`, `export-status`, `time-sync`, …) ilman välimuistissa olevaa lisenssiä tai istuntoa siirtyy interaktiiviseen `Email:` / `Password:`-kehotteeseen stdin-syötteen kautta ennen jatkamista. Vartioimaton työ, jolla ei ole välimuistissa olevaa istuntoa, jää siksi odottamaan syötettä — suorita komento `chloros-cli login EMAIL PASSWORD` kerran kutakin konetta kohti ennen päättömän työn ajoittamista.

---

## `chloros-cli logout`

Tyhjentää välimuistissa olevan istunnon ja pakottaa uuden kirjautumisen seuraavalla kutsulla.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Näyttää nykyisen lisenssitaso (Iron/Copper/Bronze/Silver/Gold), todennetun käyttäjän ja laitesidosten lukumäärän.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Tarkista Thread-4-viennin reaaliaikainen eteneminen. Voidaan kutsua turvallisesti **`process`:n**suorituksen**aikana** toisesta komentotulkista.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

CLIin näyttökielen asettaminen (tukee 38 kieltä, mukaan lukien CJK, RTL ja indialaiset kielet). Siirtyy sujuvasti englanniksi vanhoissa konsoleissa, jotka eivät pysty renderoimaan kyseistä kirjoitusta.

```
chloros-cli language [LANG_CODE] [--list]
```

### Esimerkkejä

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Projektikansiokomennot

Näillä hallitaan projektikansion oletussijaintia (jaettu GUI:n kanssa).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update`

Linux/ Vain Jetson. Tarkistaa `version_url`:n tiedostosta `/etc/chloros/update.conf` ja tarjoaa ladattavaksi ja asennettavaksi vastaavan `.deb`-tiedoston.X:n.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Linux:llä / Jetsonilla CLI suorittaa myös **automaattisen päivitystarkistuksen jokaisella käynnistyksellä** (ei estä toimintaa, ei koskaan viivästytä komentoa): se lukee `/etc/chloros/update.conf`, tallentaa tuloksen välimuistiin tunniksi tiedostoon `~/.chloros/update_cache.json` ja tulostaa `Update available: vX.Y.Z / Run: chloros-cli update`, jos uudempi versio on saatavilla. Virhetilanteissa ja osoitteessa Windowsohitetaan virhetilanteissa ja kun tulos on .

---

## `chloros-cli selftest`

Suorittaa 7-vaiheisen savutestin: versio, portin saatavuus, taustapalvelimen käynnistys, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), kohinanpoistomallin olemassaolo, CUDA+kohinanpoistimen valmius.

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

PTP-grandmasterin tila ja hallinta. Chloros-isäntä suorittaa PTP-grandmasteria; LATTICE-kamerat ja DAQ-E-yksiköt toimivat sen alaisina laitteiden välisten aikaleimojen tuottamiseksi.

| Alikomento | Kuvaus |
| --- | --- |
| `status` | Näytä grandmasterin tila, BMCA-prioriteetit ja kellotunniste. |
| `peers` | Luettele Delay_Req-pyynnön kautta havaitut orjalaitteet (kamerat + DAQ-E-anturit). |
| `cameras` | Kamerakohtainen PTP-toimintatila (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Käynnistä grandmaster-prosessi uudelleen. |
| `set-priority --priority1 N --priority2 N` | Ohita BMCA-prioriteetit. |

### Esimerkkejä

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

LATTICE-kameran ohjaus. Jokainen alikomento reititetään Chloros-taustaprosessin kautta; taustaprosessi hallinnoi kamerapoolia, joten seuraavat CLI-kutsut käyttävät uudelleen samaa avointa kahvaa.

### Yleiset vaihtoehdot (käytetään useimmissa alikomennoissa)

| Lippu | Kuvaus |
| --- | --- |
| `-d, --device N` | Kameran indeksi (oletus: 0). |
| `-s, --serial SN` | Tietty sarjanumero; ohittaa `--device`-asetuksen. |
| `--serials SN1,SN2,…` | Pilkuilla erotetut sarjanumerot monikamerakäyttöä varten. |
| `--all` | Toimii kaikilla löydetyillä kameroilla. |
| `--exposure US` | Valotusaika mikrosekunteina. |
| `--gain DB` | Vahvistus dB:nä. |
| `--pixel-format FMT` | esim. `BayerRG8`, `BayerRG12`. |
| `--width N` / `--height N` | Kuvan mitat. |
| `--preset {default,high_quality,high_speed,triggered}` | Käytä valmiita asetuksia. Kaikki toimivat vapaasti lukuun ottamatta `triggered`:ää, joka asettaa kameran valmiustilaan rivin 2 laitteistoreunan perusteella — jos mikään ei ohjaa kyseistä riviä, kamera odottaa ikuisesti sen sijaan, että tekisi tallennuksen. |
| `-o, --output DIR` | Tulostuskansio (oletus: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | GVSP-paketin koko. `auto` suorittaa ICMP+GVSP-tunnistuksia; `jumbo` = 9000; `standard` = 1500. |

### LATTICE-kameran ensimmäisen yhteyden muodostuksen työnkulku

```bash
# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Auto-shrink frame size to fit the wire
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Alikomentojen viite

#### Löytö ja tiedot

| Alikomento | Tarkoitus |
| --- | --- |
| `lattice info` | Luettele kytketyt kamerat (valmistaja, malli, sarjanumero, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analysoi isäntäjärjestelmä optimaalisen kamerakonfiguraation löytämiseksi. `--no-discover` ohittaa kameran tunnistuksen (nopeampi, vain verkkokortin analyysi). |
| `lattice network [--fix] [--estimate] [--cameras N]` | Tarkista/korjaa verkkokortin asetukset; arvioi kaistanleveys/FPS. |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | Vakaa-skeemainen taustajärjestelmän verkkokapasiteetti + taulukkosuositus (palauttaa `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `auto_capped_fps` säilyttää pyydetyn resoluution, mutta rajoittaa kohdekuvataajuuden — lue `recommended.recommended_target_fps` ja välitä se yhteyden kohteeksi; käsittele sitä onnistumisena, ei virheenä. |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | Mitä-jos-analyysi avaamatta kameroita. **`--n-active` on verkossa olevien kameroiden kokonaismäärä, ei vain tämän taulukon kameroiden**— nosta sitä, kun erilliset kamerat lähettävät dataa samanaikaisesti tai kun verkon kapasiteettia laskettaessa niiden määrä on aliarvioitu (oletus: `len(--models)`). Tulostaa aina yhteenlasketun `Wire budget:` (vaadittu MB/s vs. törmäysturvallinen yläraja) ja `Max cameras:`-rivit sekä merkitsee `** OVER-SUBSCRIBED**`:n, kun ryhmä ylikuormittaa kaistan — katso [Ryhmän fps ja pursemalli](#array-fps--burst-model). |
| `lattice gpu` | Näytä GPU:n tila. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Tarkista tai päivitä kameran laiteohjelmisto. Paikallinen `.fwa`-valinta on kiinnitetty: tiedosto kohdassa `firmware/<MODEL_PREFIX>/`, joka vastaa rakennuksen `MIN_FIRMWARE_VERSION`-versiota vastaava tiedosto flashataan, jos se on olemassa (vain korkein versio varavaihtoehtona), joten levylle tallennettu uudempi valmistajan kuva pysyy passiivisena, kunnes kyseinen kiinnitys poistetaan — tarkoituksellisesti uudemmat julkaisut saapuvat laitteisiin allekirjoitetun AWS-manifestin kautta, jota suositellaan käytettäväksi, kun se on uudempi. |
| `lattice presets [--apply NAME]` | Luettele tai ota käyttöön kameran esiasetukset. |
| `lattice status` | Kameran reaaliaikainen tila. |

#### Tallennus

| Alikomento | Tarkoitus |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | Yksittäinen ruutu. **Tallentaa oletusarvoisesti kaikki vientityypit** (`--processing all`); katso [Tallennuksen vientitasot](#capture-export-levels-the-all-default). `--levels` tallentaa nimenomaisen osajoukon (ohittaa `--processing`); `--force-daq` kirjoittaa määritetyn DAQ-lukeman `.daq`-sidecar-tiedostoksi jopa pelkän raakadatan kaappauksessa. `--jpeg-quality` = JPEG laatu 1–100 (oletus 95). |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | Lähetä virtaa levylle, kunnes painetaan Ctrl+C. |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Selainpohjainen reaaliaikainen MJPEG-esikatselu. `--ae-damping` asettaa automaattisen valotuksen vaimennuksen (0,4–100). |

#### Anturin säätö

| Alikomento | Tarkoitus |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | Lue/kirjoita mikä tahansa GenICam-solmu. |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | Valotus ja automaattinen valotus. |
| `lattice gain [--auto] [--off] [--set DB]` | Vahvistus ja automaattinen vahvistus. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | Anturin ROI ja binning. |
| `lattice format [--set FMT] [--list]` | Pikselimuoto. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Laitteisto-/ohjelmistolaukaisin. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (ei lippuja = kertaluonteinen valkotasapainotus) | Valkotasapainotoiminnot. Vain RGB- ja Bayer-kamerat; ei-toiminto (ohitetaan) mono-M3M-kameroissa. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | RGB-näytön väriprosessointiputki. `natural` (oletus) on edullinen reaaliaikainen viimeistely; `enhanced` lisää värisävyjen poiston + eloisuuden + CLAHE-paikalliskontrastin täyden hub-parity-ilmeen saavuttamiseksi noin kaksinkertaisella kuvakohtaisella viimeistelykustannuksella, joten **reaaliaikainen** kuvataajuus on alhaisempi — tallennetut kuvat saavat joka tapauksessa aina täyden viimeistelyn. RGB /Vain Bayer-kamerat; ohitetaan mono-M3M:ssä. |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | Näytä kylläisyys/kontrasti (RGB-suodatinkamerat). Ohitetaan mono-M3M:ssä. |
| `lattice filter [--set NAME] [--list]` | Aseta kameran suodatinmalli (`RGN-IMX265`, `OCN`, `NGB`, …). |
| `lattice power [--sleep]` | Anturien virta/lämpösolmut; kytke virransäästö. |

#### Kalibrointi ja anturit

| Alikomento | Tarkoitus |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibroi heijastavuuskohteesta. |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | Sisäänrakennetun alaspäin suuntautuvan valon anturin komennot. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Sovella vinjetointikorjausta olemassa oleviin kuviin. |

#### Monikamera (tilapäiset istunnot)

| Alikomento | Tarkoitus |
| --- | --- |
| `lattice multi-info` | Luettele kaikki kamerat, joilla on synkronointirooli. |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | Yksi synkronoitu ruutu kustakin kamerasta. Tallentaa **oletuksena kaikki vientityypit**, kun pysyvä matriisi on kytketty; tilapäinen taulukoton varajärjestelmä on**vain debayeroitu** (suorita ensin `array-connect` lopuille). |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | Lähetä synkronoituja kehyksiä (tilapäinen). |
| `lattice multi-test [--count N]` | GPIO-synkronoinnin ajoitustesti. |
| `lattice multi-detect [--line LINE] [--json]` | GPIO-isäntä-/orjakytkentöjen automaattinen tunnistus. |

#### Kohdistus

| Alikomento | Tarkoitus |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — sekä detektori-/vastaavuussäätimet `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, RANSAC-säätimet `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, monikehysyhdistelmä `[--averaging mean\|median\|inlier_weighted]`, geometriset rajoitukset `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, avaruudelliset rajoitukset `[--roi X0,Y0,X1,Y1] [--mask PATH]` ja orja-kohtaiset ohitukset `[--per-cam-override SN:KEY=VALUE]` (toistettavissa) | Laske kohdistusprofiili reaaliaikaisista kameroista. `--prefilter`:n oletusarvo on `gradient` (reunakartta; vastaa GUI-/matriisikohdistinta — reunat säilyvät spektrikaistojen yli). `--matcher flann` on kannattava yli ~5000 piirteessä; `--averaging median` on vakaa yhden virheellisen kuvan suhteen, `inlier_weighted` painottaa vastaavuuksien lukumäärän mukaan; `--lock-scale` projisoi lähimpään kiertoon (ei skaalausta), `--lock-axis` nollaa yhden siirtokomponentin; `--mask` soveltuu kaikkiin kameroihin (käytä `--per-cam-override` kamera-kohtaisiin asetuksiin, esim. `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` kieltäytyy tallentamasta kalibrointia, jonka uudelleenprojisoinnin RMS ylittää rajan. |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | Kaappaa yhden kohdistetun monikaistaisen kehyksen. `--bit-depth` sovittaa oletuksena kuvan kameraan; `--no-crop` säilyttää koko kehyksen (täyttää mustalla); `--interpolation` (oletus `linear`) ja `--border-mode`/`--border-value` (oletusarvo `constant`/0) ohjaavat CPU-vääristymää — GPU-polku on joka tapauksessa bilineaarinen. |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | Lähetä kohdistetut monikaistaiset kehykset (samat warp-säätimet kuin `align-apply`). |
| `lattice align-info --profile PATH [--json]` | Näytä profiilin tiedot. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Muuta kerrosten järjestystä. |

#### Indeksi / Kasvillisuusmatematiikka

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

Täydellinen lippujoukko: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI / NDRE / EVI / SAVI / GNDVI /…), `--formula EXPR`, `--channel SYM=BAND` (toistettavissa), `--capture-level raw|debayered|radiance|reflectance|unknown` (ohittaa lähdeTIFFissa tallennetun tallennustason; oletus: luetaan TIFF-metatiedoista), `--output PATH`, `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Kun käytetään `--live`:ää, kohdistuksen vääristymän säätimet koskevat myös seuraavia: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`.

> **`--channel`-symbolit ovat kirjainkokoherkkiä.** Symbolipuolen on vastattava tarkalleen esiasetuksen kanavanimiä (esiasetuksissa käytetään pieniä kirjaimia, esim. NDVI = `red`,`nir` — tarkista `--list-presets`), ja taajuusaluepuolen on vastattava taajuusalueen nimeä kohdistetussa pinossa (tai oltava 0-pohjainen taajuusalueindeksi offline-tilassa). `--channel red=Red_660 --channel nir=NIR_850` toimii; `--channel RED=660` epäonnistuu ja aiheuttaa virheen `channel_map missing entries`.

#### Pysyvät yhteydet (Smart-Prep, GUI-vastaava virtaus)

Nämä komennot pitävät kamerat auki taustapoolissCLI-kutsujen välillä.

| Alikomento | Tarkoitus |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Lisää yksi kamera pooliin (yksi kamera, ei ryhmää). |
| `lattice cam-disconnect [--serial SN] [--all]` | Vapauta. |
| `lattice cam-list` | Luettelee pooliin kuuluvat kamerat. |
| **`lattice array-connect`**|**Yhdistää pysyvän synkronoidun ryhmän (suositeltu aloituskohta).** Suorittaa koko GUI-käyttöliittymän älykkään valmisteluvirran. |
| `lattice array-disconnect [--array-id ID] [--all]` | Vapauta matriisi. |
| `lattice array-list` | Luettele kytketyt matriisit. |
| `lattice array-status [--array-id ID]` | Live-kuvataajuus (fps), PTP, viimeisin virhe. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Yksi synkronoitu tallennus live-matriisista — Yksittäinen / Jatkuva / Välillä / Nopein. **Oletusarvo on `all`** (yksi tiedosto kutakin soveltuvaa vientityyppiä kohti kameraa kohti). Ohitetut kamerat (esim. RGB, jotka on suljettu pois säteilyvoimakkuuden/heijastavuuden mittauksesta) raportoidaan tunnuksella `Skipped: SN:<serial> (<reason>)`; heijastavuuden mittauksessa käytetty DAQ-lukema tallennetaan rinnalla ja raportoidaan tunnuksella `DAQ: <path>`. Katso [Tallennustilat, tallentimet ja offline-jälkikäsittely](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | Tallenna yhdistetty indeksin reaaliaikainen näkymä videoksi/GIF-tiedostoksi (seurantataso; vaatii yhdistettyä virtaa auki). |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | Raaka-Bayer-sarjakuvaus korkealla kuvataajuudella (analyysilaatuinen; jälkikäsitellään offline-tilassa). |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Käsittele tallennettu raaka-sarjakuvaus uudelleen kalibroiduksi videoksi (videoiksi). |

##### `array-connect`-asetukset

| Merkki | Oletus | Kuvaus |
| --- | --- | --- |
| `--serials SN1,SN2,…` | Tunnista automaattisesti kaikki LATTICE-kamerat (vaatii vähintään 2) | Ensimmäinen sarjanumero on MASTER. Jos tämä jätetään pois, tunnistus suodattaa tulokset LATTICE (`TRI032*`) -malleihin ja yhdistää ne kaikki. |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO-synkronointilinja. |
| `--target-fps F` | auto | Master-laukaisun laukaisutaajuus. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Ohita tasovalitsin. |
| `--wire-ceiling-mbps MB_PER_S` | tunnistetaan automaattisesti | **Isäntäkoneen jatkuva kaistanleveys, MB/s — luku, johon koko matriisin allokointi perustuu.** Laske arvoa, kun matriisi ilmoittaa GVSP-vioittuneista kehyksistä: automaattinen arvo johdetaan verkkokortin ilmoittamasta linkkinopeudesta, joka yliarvioi USB-sovittimet, kapeat PCIe-kaistat ja kuormitetut jaetut verkot. Tallennetaan projektinjoten sen voi palauttaa avaamalla tiedoston uudelleen (reopen) tai suorittamalla komennot `CLI` tai `SDK`. Katso [Matriisin kunto](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | auto | Laitteistopohjainen binning. |
| `--no-recommend` | off | Ohita verkko-analyysivaihetta. |
| `--no-ptp` | off | PTP:n poistaminen käytöstä (kameroiden väliset aikaleimat eivät tällöin **ole** vertailukelpoisia). |

### Smart-AE / Smart-Capture

LATTICE-matriisit suorittavat jatkuvaa automaattista valotuksen säätöä (AE) taustalla heti, kun ne on kytketty, mutta vastikään kohdistetun kohtauksen konvergoituminen kestää hetken. `array-capture --smart` on **valmiiksi pakattu kätevä toiminto**: se odottaa, että AE vakiintuu jokaisessa matriisin kamerassa, ja käynnistää sitten kuvauksen. Käytä sitä, kun vaihdat kohdetta istunnon aikana.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Vakautumiskäytäntö on oletusarvoisesti konservatiivinen: 5 sekunnin aikakatkaisu, 1,5 sekunnin vakausikkuna, ±5 %:n valotuksen vaihtelutoleranssi. Säädä asetuksia SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) -sovelluksella, jos tarvitset automaatiolta erilaista käyttäytymistä.

### Kuvausvientitasot (oletusarvo `all`)

Tästä versiosta lähtien, `lattice capture`, `lattice multi-capture` ja `lattice array-capture` **oletusarvo on `--processing all`** — yksi tallennettu tiedosto kutakin vientityyppiä kohti, joka koskee jokaista kameraa ja vastaa käyttöliittymän ”Capture All” -toimintoa. Tasot ovat:

| Taso | Lähtö | Koskee |
| --- | --- | --- |
| `raw` | Yksikanavainen Bayer (mustavalkokamerat: yksi kaista) suoraan anturista. | Kaikki kamerat. |
| `debayered` | 3-kanavainen BGR-demosaikki (mustavalkokamerat: 1-kanavainen harmaasävy). | Kaikki kamerat. |
| `radiance` | float32 W/m²/sr/nm koko radiometrisen ketjun kautta. | Vain monispektriset (M3C/M3M) — **ohitetaan RGB-suodattimilla varustetuissa kameroissa**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), Pix4D-valmis. | Vain monispektrinen, ja **vain kun DAQ on sidottu + kamera on kalibroitu**; muuten ohitetaan. |
| `preview` / `display` | Täydellinen GUI-esikatseluketju (CCM + WB + gamma kameran profiilin mukaan). `lattice capture` nimeää tämän `preview`:ksi; `array-capture`/`multi-capture` käyttävät `display`:ää. | Kaikki kamerat. |

Syötä yksi taso, jos haluat tallentaa vain sen yhden (`--processing debayered`). Kun pyydät `all`:ää, tasot, jotka eivät koske tiettyä kameraa, ohitetaan (ja raportoidaan), mutta virhettä ei ilmoiteta — kytkemätön tai kalibroimaton kamera saa silti `raw` / `debayered` / `preview`.

Kaikissa heijastavuuskuvissa todellisuudessa käytetty DAQ:n alaspäin suuntautuva lukema kirjoitetaan **`.daq`**-sivutiedostoon kuvan viereen (jotta tallenne voidaan käsitellä uudelleen myöhemmin) ja raportoidaan `DAQ:`-rivillä.

### Miltä tallennuskansio näyttää

Jokainen vientityyppi sijoitetaan **omaan alikansioonsa** `-o`:n alle, joten monitasoinentasoinen tallennus ei koskaan sekoita eri tyyppejä:

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>` on tallennuksen aikaleima ja `<serial>` kameran sarjanumero, joten yksi synkronoitu ryhmä jakaa
aikaleimankaikissa kameroissa. **Huomaa yksi epäsymmetria:** `display`-taso tallennetaan kansioon,
jonka nimi on `preview/`, kun taas tiedostojen nimissä säilyy `_display` — kansio ja tiedostotunniste eroavat toisistaan
vain vain kyseisellä tasolla. Tuntemattomat tasot tallennetaan omaa nimeään kantavaan kansioon, ja jos alikansiota
ei voida luoda, tiedosto kirjoitetaan tulostuksen juurikansioon sen sijaan, että se menetettäisiin.

**Captures-kansion uudelleenkäsittely:**osoita `chloros-cli process`**captures-juurikansioon**
(`output/`). `process` tuo yleensä vain nimeämäsi kansion, mutta jos kyseisessä kansiossa ei ole
kuvia ja siinä on alikansioita, se etenee automaattisesti alikansioihin – joten juurikansion tasoiset alikansiot ja
juurikansio `.daq` haetaan kaikki kerralla. Jokainen kuvauskansion taso tuodaan yhtenä kuvana, ja
muut tasot ovat käytettävissä tiloina sen sijaan, että jokaisesta tasosta tuodaan yksi kuva.

**Tason alikansion** suoraan (esim. `output/raw/`) toimii myös. Tällöin juurikansio
`.daq` jäljelle, joten kopioi tai osoita DAQ-lukema sen viereen, kun johdat uudelleen radiometrisen
tuotteen `raw/`:sta — muuten aikaleiman täsmäystä ei voida ratkaista mihinkään.

**Käsittely alkaa aina tiedostosta `raw`.** Kussakin tallennuksessa raakakehys on käsittelyputken lähde;
`debayered`, `radiance`, `reflectance` ja `preview` ovat katselukuvia, mutta niitä ei koskaan syötetä
takaisin prosessointiputkeen. Johdetun tuotteen uudelleenkäsittely johtaisi vignettin, CCM:n ja
säteilyn laskentaa, jotka on jo upotettu sen pikseleihin, joten Chloros kieltäytyy tästä sen sijaan, että
suorittaisi käsittelyn kahdesti. Kaksi huomionarvoista seurausta:

- `index/`- ja `composite/`-renderöintejä ei **koskaan** käsitellä. Ne ovat tulosteita, eivät tallenteita —
  NDVI-LUT-renderöinnillä ei ole merkityksellistä säteilytulkintaa.
- Tallennuskansio, joka on viety **ilman** `raw`-tiedostoa (esim. `array-capture --processing reflectance`), ei sisällä
  kelvollista prosessilähdettä. Nämä tallenteet tuodaan ja näytetään normaalisti, mutta `process` ohittaa
  ne ja ilmoittaa asiasta seuraavasti:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Jos sinun on ehdottomasti välitettävä johdettu tuote eteenpäin — esimerkiksi hub-istunto, joka on tallennettu
  `demosaic`-lippua käytettäessä, tai vanha kansio — `--input-level {raw,debayered,processed}` pakottaa sisäänmenokohdan
  ja ohittaa ohituksen. Tämä lippu on tarkoituksellinen pakotie; `auto` (oletus)
  ei koskaan käsittele tallennetta, jolla ei ole raakadataa.

### Ohitetut tallenteet sekasuodatinryhmissä

Kun yhdistät RGB- ja monispektrikameroita samassa ryhmässä, `array-capture --processing radiance` (tai `reflectance`) tallentaa monispektrikuvat ja **ohittaa** RGB-kamerat — Bayer-kohtainen säteilyvoimakkuus ei ole merkityksellistä laajakaistaiselle anturille. Komento CLI tulostaa jokaisen tallennetun tiedoston (ja sen vientitason), jokaisen kirjoitetun `.daq`-tiedoston sekä jokaisen ohituksen erikseen, joten tiedostojen lukumäärä ei tule yllätyksenä:

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Ohitussyyn tunnisteet noudattavat mallia `<level>-not-applicable-to-rgb-cam`. Heijastusance voi myös ohittaa tiedostoja merkinnöillä `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)` sekä merkinnällä `dls-uncalibrated-band-<nm>`, kun kaista sijaitsee pääosin DAQ-valosensorin radiometrisesti kalibroidun alueen ulkopuolella (~374–974 nm) – toimitettavista tuotetunnuksista ainoastaan F988, jonka tuettu polku on heijastavuuspaneelin työnkulku.

Käytä `--processing debayered` (tai `display`) sisällyttääksesi kaikki kamerat suodatintyypistä riippumatta, tai oletusarvoista `all` saadaksesi kaikki soveltuvat tasot kamerakohtaisesti yhdellä kertaa.

---

## Tallennustilat, tallentimet ja offline-uudelleenkäsittely

Nämä kaikki toimivat **pysyvällä taulukolla** (suorita ensin `array-connect`). Ne vastaavat käyttöliittymän tallennuspaneelia.

### `array-capture`-tilat

`array-capture` on yksittäinen komento, joka sisältää neljä suljintilaa sekä joukon vientiasetuksia:

| Tila | Lippu | Toiminta |
| --- | --- | --- |
| **Yksittäinen** *(oletus)* | (ei mitään) | Yksi synkronoitu kaappausryhmä, jonka jälkeen ohjelma sulkeutuu. |
| **Jatkuva** | `--continuous` | Peräkkäisiä kierroksia, kunnes `Ctrl+C`, `--count N` tai `--duration S`. |
| **Väli** | `--interval S` | Yksi kierros joka `S` sekuntia (mitattuna kunkin kierroksen alusta), samat rajat. |
| **Nopein** | `--fastest` | Vain raakadata + määritetty DAQ-lukema + yhdistetty indeksi; ohittaa säteilyvoimakkuuden/heijastavuuden/näytön laskennan, jotta kehys latautuu nopeasti. Edellyttää `--processing raw --force-daq`:ää. Käsittele tallennettu `.daq` myöhemmin kalibroiduiksi tuotteiksi. |

Vientiasetukset (yhdistettävissä mihin tahansa tilaan; kaikilla on yhteinen GUI/SDK-päätetunnus):

| Lippu | Vaikutus |
| --- | --- |
| `--processing LEVEL` | Yksi vientitaso tai `all` (oletus). |
| `--levels L1,L2,…` | Vientityyppien nimenomainen osajoukko (esim.esim. `raw,radiance,reflectance`); **ohittaa `--processing`**. |
| `--aligned` / `--no-aligned` | Muunna jokaisen jäsenenei-raakamuotoinen vienti taulukon [kohdistusprofiiliin](#alignment) (yhteisrekisteröity). Raakamuotoinen vienti jää muuntamatta, mutta sisältää muunnoksen metatiedoissa. Käytetään oletuksena kohdistamatonta (varoituksella), jos taulukolla ei ole profiilia. |
| `--index` / `--no-index` | Tallenna / ohita kamerakohtainen kasvillisuusindeksin peittokuva, jos sellainen on määritetty. Oletus: renderöi se. |
| `--force-daq` | Tallenna määritetty DAQ/DLS-lukema `.daq`-sidecar-tiedostona, vaikka mikään valittu taso ei sitä tarvitsisikaan (esim. pelkkä raakadata-kaappaus), jotta kehykset voidaan käsitellä uudelleen heijastavuus-/indeksitiedoiksi offline-tilassa. |
| `--smart` | Odota, että AE vakiintuu kaikissa kameroissa ennen laukaisua (katso [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | TIFF pikselipakkaus. `deflate` (oletus) = häviötön zlib L1 + vaakasuuntainen ennustaja, ~4,1 MB täysresoluutioista kuvaa kohti; `none` = pakkaamaton, ~5× nopeampi kirjoitusnopeus, ~6,3 MB per ruutu — käytä maksimaalisen jatkuvan tallennusnopeuden saavuttamiseksi, kun levitila sen sallii. Molemmat ovat häviöttömiä ja luetaan identtisesti tuonnin yhteydessä. |

> **Yksikertaisen kirjoituksen TIFF + jatkuvan siirtonopeuden malli.**Tallenteet kirjoitetaan**yhdellä**TIFF-tiedostokierroksella, joka sisältää pikselit + XMP + IFD0 Valmistaja/Malli (mitattu täysresoluutioisella Mono12:llä: 36 ms pakattuna / 6,5 ms pakkaamattomana, verrattuna ~148 ms:iin vanhassa ”kirjoita ensin, kirjoita sitten uudelleen ExifToolilla” -menetelmässä); ainoa jäljellä oleva ExifTool-tehtävä (EXIF-ali-IFD:n viimeistely) suoritetaan asynkronisessa taustatyöprosessissa, ja kehys on valmis ja tuontivalmis, vaikka kyseistä tehtävää ei koskaan suoritettaisikaan. Huomaa , että DEFLATE-pakkaus pitää hallussaan Python GIL:ää, joten pakattuja kirjoituksia**ei**suoriteta rinnakkain kamerakohtaisissa kirjoitussäikeissä — jatkuva 8-kameran täysresoluutioinen tallennus anturin nopeudella (~10,4 fps) vaatii `--compression none`**ja** NVMe-luokan levyä (~500 MB/s jatkuvaa kirjoitusta). Sama säätö on käytettävissä nimellä `compression` kohdassa `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — yhdistetty indeksi video/GIF (valvontataso)

Tallentaa kaiken, mitä **yhdistetty reaaliaikainen indeksinäkymä** näyttää `.avi`:lle (ja valinnaisesti `.gif`:lle). Koska se poimii reaaliaikaista yhdistelmäkuvaa, yhdistelmävirran on oltava auki (esim. taulukkoa esikatsellaan käyttöliittymässä), jotta kehykset tallentuvat. Se tarkistaa edistymisen joka 2 sekunnin välein ja pysähtyy laitteissa `--duration`, `Ctrl+C` tai kun tallennin lopettaa toimintansa itsestään.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| Lippu | Oletus | Kuvaus |
| --- | --- | --- |
| `--array-id ID` | vain kuvamassiivi | Kohdekuvamassiivi (jätä pois, jos vain yksi on kytketty). |
| `-o, --output DIR` | `output` | Tulostuskansio (taustaprosessin paikallinen). |
| `--fps F` | `10` | Tallennus kuvataajuus. |
| `--duration S` | kunnes Ctrl+C | Pysäytetään automaattisesti `S` sekunnin kuluttua. |
| `--gif` | pois | Kirjoita myös animoitu GIF-tiedosto. |
| `--gif-only` | pois | Tallenna vain GIF (ei `.avi`). |

### `array-burst` — raaka-Bayer-sarjakuvaus korkealla kuvataajuudella (analyysilaatuinen)

Lukee kuvausloopin synkronoidun ryhmän puskurin suoraan — **ei kalibrointiketjua, ei exiftool-työkalua, ei live-näkymää tarvita** — joten se toimii kameran täydellä kuvausnopeudella. Tallentaa raakakuvat + kuvakohtaisen manifestin + yhden `.daq`-tiedoston jokaista erillistä DLS-lukemaa kohti `<output>/bursts/<base>/`. Käsittele uudelleen offline-tilassa (seuraava komento) tai välitä `--build`, jotta se tehdään välittömästi pysäytyksen yhteydessä.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Lippu | Oletus | Kuvaus |
| --- | --- | --- |
| `--array-id ID` | vain taulukko | Kohdetaulukko. |
| `-o, --output DIR` | `output` | Tulostuskansio (burst tallentuu `<DIR>/bursts/<base>/`:ään). |
| `--duration S` | kunnes Ctrl+C | Automaattinen pysäytys `S` sekunnin kuluttua. |
| `--max-frames N` | rajoittamaton | Automaattinen pysäytys `N` raakakuvaa jälkeen. |
| `--build` | pois | Pysäytyksen jälkeen sarja käsitellään välittömästi uudelleen (sama kuin `array-build-video`). |
| `--products …` | `combined:index` | Kun `--build`: mitkä videot luodaan (katso alla). |
| `--fps F` | `10` | Yhdessä `--build`:n kanssa: videon kuvataajuus (fps). |
| `--save-tiffs` | pois | Yhdessä `--build`:n kanssa: tallentaa myös kehyksittäin kalibroidut TIFF-tiedostot. |
| `--gif` | pois | Yhdessä `--build`:n kanssa: kirjoittaa myös animoituja GIF-tiedostoja. |

### `array-build-video` — tallennetun sarjakuvauksen jälkikäsittely offline-tilassa

Sovittaa kunkin raakakehyksen ajallisesti lähimpään tallennettuun `.daq`-lukemaan ja käsittelee sen **samassa säteilyvoimakkuuden / heijastavuuden / indeksin ketjussa kuin tuontiprosessi**, jolloin tuloksena syntyy yksi tai useampi video.

`--products` on pilkulla erotettu luettelo `kind:level`-kohteista, joissa `kind` ∈ `per_cam` | `combined` ja `level` ∈ `radiance` | `reflectance` | `index`. Pelkkä `level` (ilman `kind:`:ää) on oletusarvoisesti `per_cam`. Oletusarvo on `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Lippu | Oletusarvo | Kuvaus |
| --- | --- | --- |
| `--burst-dir DIR` | (pakollinen) | Polku burst-kansioon (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | `kind:level`-luettelo, kuten yllä. |
| `--fps F` | `10` | Videon kuvataajuus (fps). |
| `--save-tiffs` | pois | Tallenna myös kuvakohtaisesti kalibroidut TIFF-tiedostot videon (videoiden) ohella. |
| `--gif` | off | Kirjoita myös animoituja GIF-tiedostoja. |

> **Valitse oikea tallennin.** `array-record` on *valvontatason* — se tallentaa reaaliaikaisen komposiittikuvan sellaisena kuinnäytetään ja vaatii, että videovirta on auki. `array-burst` → `array-build-video` on *analyysitason* — se tallentaa raakaa anturidataa täydellä nopeudella ja rekonstruoi kalibroidut säteily-/heijastus-/indeksivideot jälkikäteen ilman, että reaaliaikaista kuvaa tarvitaan.

### Mono (M3M) yksikaistaiset kamerat

**M3M**-sarja on Bayer**M3C**:n monokrominen vastine: yksi kapeakaistainen häiriösuodatin kameraa kohti (`M3M-<lens>-F<wavelength>`, esim. `M3M-L87-F685`), joten anturi tuottaa**yhden harmaasävykaistan** ilman Bayer-mosaiikkia. Demosaiikkia ei tarvitse tehdä, kanavien välistä ylikuulumista ei tarvitse erottaa, eikä valkotasapainoa tarvitse säätää — koko RGB -näytön väriprosessi ei yksinkertaisesti tule kyseeseen.

Mitä tämä tarkoittaa CLI -laitteessa:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**tunnistavat monokromaattisen kameran ja**ohittavat sen yhden rivin viestillä** sen sijaan, että ne soveltaisivat merkityksettömiä asetuksia. Ne toimivat edelleen normaalisti RGB /Bayer M3C -kameran kanssa samassa istunnossa.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** toimivat edelleen — säteily ja heijastavuus ovat *kaistakohtaisia* radiometrisiä karttoja, ja ne on määritelty täysin tarkasti yhdelle kaistalle. Mono-kuvissa on **identiteetti**-anturivaste-matriisi (ei 3×3-sekoituksen purkua), joten kalibrointilaskelmat eivät vaikuta niihin.
- **Yksittäinen monokamera ei voi tuottaa kasvillisuusindeksiä.**NDVI / NDRE / jne. vaativat vähintään kaksi kaistaa (esim. Red + NIR). Jos haluat saada indeksin monokameralla, suuntaa**useita** M3M-kameroita eri aallonpituuksille, yhdistä ne yhdeksi monikaistapinoksi, ja luo *sille* indeksi:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel`-symbolien on vastattava esiasetuksen kanavien nimiä **täsmälleen** (kirjainkoko; NDVI-tiedostot ovat pienillä kirjaimilla `red`,`nir` — katso `--list-presets`), ja kaistan nimi viittaa kaistalle kohdistetussa pinoissa (offline-tilassa hyväksytään myös 0-pohjaiset kaistaindeksejä, esim. `--channel red=0 --channel nir=1`).

Pinoon kuuluvien osien erottimena toimii mallimerkkijonossa oleva `M3M`-tunniste (sitä ei esiinny koskaan `M3C`-merkkijonossa), joka näkyy käyttöliittymässä/SDKissa muodossa `is_mono`.

---

## Isäntäkoneen verkkokortin asetukset ja säätö (LATTICE-ryhmät)

LATTICE-kamerat lähettävät GVSP:tä isäntäkoneen Ethernet-sovittimen kautta, joten monikameraryhmissä sovittimen **ohjaimella**ja**vastaanottorenkaan koolla** ovat yhtä tärkeitä kuin linkin nopeus. Väärät asetukset näkyvät `FRAMES WILL DROP` / `Reduce ROI to enable`-porttina Array Settings -paneelissa (ja `lattice network-analysis` / SDK:n `analyze_array_network()`), vaikka kamerat itsessään toimisivatkin moitteettomasti.

### USB 10GbE -sovittimet — Realtek RTL8157 (&quot;Realtek USB 10GbE Family Controller&quot;)

| Kohde | Vaadittu arvo | Miksi se on tärkeää |
| --- | --- | --- |
| **Ohjaimen versio**|**≥ v10.67 (tammikuu 2026)**, INF `rtump64x64sta.inf` | Vanha**2016**-ohjain (v10.65, `rtump64x64.inf`) käsittelee virrankatkaisua ja virhetarkistuksia virheellisesti, mikä aiheuttaa**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**-virheen sammutuksen, uudelleenkäynnistyksen tai lepotilan aikana. Siirtyminen jumittuu (~5 minuutin aikakatkaisu), käyttäjä sammuttaa laitteen väkisin, ja toistuvat epäasialliset sammutukset**vääristävät WMI-tietovaraston**(PowerShell/työkalut alkavat epäonnistua virheen `Invalid class` vuoksi) ja**jumiuttavat USB-pino** seuraavalla käynnistyksellä (verkkokortti ei aktivoidu; USB-asemien tunnistus lakkaa toimimasta). Päivitä realtek.comista (tai donglen valmistajalta) ennen kuin luotat puhtaisiin uudelleenkäynnistyksiin. |
| **Vastaanottopuskuri**— avainsana `ReceiveBufferLen` |**256**(ajurin enimmäisarvo) | Verkkokortin RX-rengas. Ajurin oletusarvo**32**jättää vain ~0,26 Mt käytettävää tilaa — aivan liian vähän monikameran sarjakuvaukseen — joten array-paneeli ilmoittaa virheen `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` ja estää yhteyden muodostamisen. Arvolla**256**rengas on suuri (**~13,5 Mt mitattuna laboratorion 10 GbE -isäntäkoneella**), mikä antaa RX-putkelle todellista liikkumavaraa monikameran GVSP-sarjakuvauksille. (Se, *muodostaako* tietty kokoonpano todella yhteyden, ratkaistaan kahdella tarkistuksella — **drain-aware**pääsytarkistus ja**aggregoitu ylimerkintä** -tarkistus — ei raakaa sarjakuvaus-vs.-rengas-vertailua; katso [Array fps &amp; burst model](#array-fps--burst-model).) |
| **Vastaanotto-URB:t**— avainsana `PendingReceives` |**64** (maks.) | Lähetysvaiheessa olevat USB-pyyntölohkot; lisää yhdessä vastaanottopuskureiden kanssa purskeiden vaimentamiseksi. |
| **Jumbo-kehys** — avainsana `*JumboPacket` | **9014** | Tarvitaan 9000 tavun GVSP-paketeille (6× vähemmän paketteja/kehys kuin 1500). |

> ⚠️ **NIC-ohjaimen päivitys NOLLAA nämä lisäasetukset oletusarvoihin.**Kun olet päivittänyt tai vaihtanut sovittimen ohjaimen,**määritä uudelleen** `ReceiveBufferLen=256` ja `PendingReceives=64`, tai muuten array-paneeli lukittuu uudelleen, vaikka ”laitteistossa ei ole tapahtunut muutoksia”. Tämä on yleisin syy siihen, että aiemmin toiminut laitteisto yhtäkkiä kieltäytyy muodostamasta yhteyttä.

Määritä asetukset **järjestelmänvalvojan oikeuksilla** toimivasta PowerShellistä (korvaa sovittimen nimesi, esim. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` kattaa USB 10GbE -sovittimet.** Se tunnistaa nyt sovittimen tyypin ja säätää oikean receive-ring-avainsanan: `*ReceiveBuffers`→2048 PCIe-verkkokorteille (Intel I219, jne.) tai `ReceiveBufferLen`→256 + `PendingReceives`→64 Realtekin **USB** 10GbE-ohjaimelle (joka ei paljasta `*ReceiveBuffers`-arvoa). Kohteet rajoitetaan kunkin ohjaimen ilmoittamaan enimmäisarvoon (`NumericParameterMaxValue`), joten se ei koskaan kirjoita alueen ulkopuolista arvoa. Suorita se **korotetuista**oikeuksista varustetusta** terminaalista; kuten minkä tahansa rekisteripohjaisen säätämisen kohdalla, muutos astuu voimaan vasta, kun sovitin käynnistetään uudelleen tai järjestelmä käynnistetään uudelleen. Yllä mainitut manuaaliset `Set-NetAdapterAdvancedProperty`-komennot ovat edelleen hyvä vaihtoehto — ne otetaan käyttöön reaaliaikaisesti (sovitin sidotaan uudelleen) ilman uudelleenkäynnistystä.

### Verkon perusteet (kaikki LATTICE-yhteydet)

- **Osoitteistus:** link-local `169.254.0.0/16` (GigE Vision LLA). Isäntäkone käyttää staattista `169.254.x.x/16`-osoitetta; kamerat ja DAQ-E määrittävät itselleen osoitteet samasta alueesta. DHCP:tä tai yhdyskäytävää ei tarvita.
- **Paketin koko:**suositellaan jumbo-pakettia (9000), mutta anna automaattisen mittauksen selvittää se – se mittaa uudelleen jokaisen yhteyden muodostuksen yhteydessä ja ohittaa jo kameran 1500-tavun ICMP-rajoituksen GVSP-tunnistuksen avulla, joten se päätyy jumbo-kokoon siellä, missä kaapeli sitä todella tukee. Määritä kiinteä arvo (pin) `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`:llä vain, jos tiedät paremmin kuin tunnistin, ja käytä mieluummin komento-kohtaista asetusta kuin pysyvää: kiinteä arvo ohittaa tunnistuksen, joten jos reitti ei todellisuudessa tue 9000:ta,**jokainen** kaappaus aikakatkaistaan `SC_ERR_TIMEOUT -1011`:n avulla (katso [Ympäristömuuttujat](#environment-variables)).
- **RX-rengas skaalautuu `ReceiveBufferLen`:n mukaan:**oletusarvolla `32` käytettävissä oleva rengas on ~0,26 MB (liian pieni mille tahansa monikamerapurskeelle); suurimmalla `256`-arvolla se on suuri (~13,5 MB mitattuna laboratorion 10GbE-isäntäkoneella), mikä tarjoaa todellista liikkumavaraa. Se, muodostuuko yhteys tietyn konfiguraation kanssa, ratkaistaan sitten tyhjennystä huomioivalla pääsytarkistuksella**ja** alla kuvattuun kokonaisylivarausarviointiin — ei pelkkään raakaverstaukseen ja renkaan vertailuun.

### Array-kuvataajuus (fps) ja sarjakuvausmalli

Kuinka tulkita Array-asetuspaneelia (sekä `lattice analyze-array` / SDK:n `analyze_array_network`):

- **Burst-arvo lasketaan yhteen kamerakohtaisesti kunkin kameran todellisessa pikselimuodossa.**Mono**M3M**-kamerat lähettävät**Mono12 (2 bittiä/pikseli)**;**M3C**Bayer-kamerat lähettävät 8- tai 12-bittistä dataa (TRI032S lähettää hiljaisesti BayerRG12:ta, vaikka pyydettäisiin BayerRG8:aa). Näin ollen neljän kameran täysresoluutioinen kehys on**~12,6 MB, jos kaikki ovat 8-bittisiä, mutta ~25 MB, jos kolme kameraa on 12-bittisiä mono-kameroita**. Projisointi määrittää kunkin kameran formaatin sen mallin perusteella (identiteettivälimuisti), joten lähetys vastaa sitä, mitä kaapeli tosiasiassa välittää — ei yhtä kokoa sopivaa BayerRG8-oletusta.
- **USB-Ethernet-sovittimen enimmäisnopeus on 200 MB/s riippumatta sen nimilapusta.** Tehokkuustaulukko, joka muuntaa linkkinopeuden jatkuvaksi arvoksi, on johdettu PCIe:stä; USB-verkkokortti ilmoittaa *Ethernet*-linkkinopeutensa, mutta sen toimintaa rajoittavat USB-väylä ja sen ohjain. Eräs USB 10GbE -sovitin saavutti aiemmin noin 1063 MB/s:n ”jatkuvan” nopeuden — lukua, jota ei koskaan testattu — ja tuloksena oleva rytmityshäiriö vioitti 6–18 % kehyksistä, vaikka se ilmoitti edelleen terveen tavoite-fps-arvon. USB-liitännällä varustettujen verkkokorttien enimmäisnopeus on nyt **200 MB/s** absoluuttisena arvona (rajoitus johtuu väylästä, joten se ei skaalaudu nimellistehon mukaan; USB 1 GbE -sovitin saavuttaa noin 80 MB/s, eikä rajoitus vaikuta siihen). `wire_ceiling_source`-merkintä suorituskykytietueessa ilmaisee tämän sanallisesti, ja `nic_is_usb` merkitsee sen. Kumpikin rajoitus voidaan ohittaa `--wire-ceiling-mbps`:llä.
- **Pääsy on drain-tietoinen, ei koko-burst-vs-ring-tyyppinen.** Samanaikaisen burstin on sovittava vain *tilapäiseen ruuhkaan* = `max(0, Σ per-cam arrival − host drain) × emit_window`, ei koko burstiin. Nopean isäntäkoneen ja hitaiden kameroiden verkostossa (**PCIe**10G-isäntäkone + 4× 1 GbE-kameraa: saapuminen ≈ 320 MB/s, tyhjennys ≈ 1063 MB/s) isäntäkone tyhjenee nopeammin kuin kamerat täyttyvät, joten ruuhka ≈ 0, joten täysresoluutioinen sim-emit**päästää**läpi, vaikka 25 MB:n purske ylittää 13,5 MB:n renkaan. Kun samat neljä kameraa liitetään**USB**10GbE -sovittimen taakse, tyhjennysnopeus on 200 MB/s, ei 1063 — saapuva data ehtii ohi, ja menetys näkyy vioittuneina kehyksinä eikä alhaisempana kehysnopeutena. 1 GbE -isäntäkoneella kameroiden 31,25 MB/s:n DLThr-alaraja saa saapuvan datan ylittämään poistokapasiteetin → se**estää**tilanteen oikein (tämän tyyppisissä estotilanteissa pienennä ROI:ta tai käytä binningiä ≥ 2). Admittanssi on yksi**kahdesta** yhteysportista — toinen on alla oleva aggregoitu ylimerkintätarkistus.
- **Ennustettu fps on konservatiivinen sarjallinen-hakun yläraja.**Isäntäkoneen keräyssilmukka hakee tällä hetkellä kunkin kameran puskurin**sarjallisesti**(kunkin kameran kohdalla noin yksi lähetysikkuna), joten sykliä rajoittaa `max(readout+emit, N × emit)`, jossa kamerakohtainen lähetys on rajoitettu kameran**liityntäyhteyden**(1 GbE ≈ 80 MB/s), ei isäntäkoneen uplinkiin. Neljän kameran täysresoluutioisessa 12-bittisessä järjestelmässä tämä on**~2,8 fps**, mikä vastaa mitattua ~2,7–3,0 fps:ää. Arvo on tarkoituksellisesti**valotuksesta riippumaton**, joten hämärissä kohtauksissa todellinen kuvataajuus voi laskea hieman alle ylärajan, kun valotusaika pitenee. Sarjaliikenne on todellinen kuvataajuuden rajoittaja; sen rinnakkaistaminen nostaisi ylärajan kohti yksittäisen lähetysnopeuden tasoa.
- **Kokonaismääräinen ylimerkintä estää yhteyden muodostamisen kokonaan.**Kamerakohtaisen kaistanleveyden allokaatio on rajoitettu vähintään**8 MB/s**(`ARRAY_PER_CAM_FLOOR_BPS`), joten kun alaraja saavutetaan, kokonaiskysyntä (`per_cam × N`) voi ylittää**törmäyksiltä suojatun verkon ylärajan**(`sustained × sim_emit_factor`). Käytännön täysresoluutioiset ylärajat 1 GbE:llä:**6 kameraa 1500 MTU:lla, 9 jumbo-kehyksillä**. Tämä yläraja riippuu yksinomaan kaapelista ja alarajasta — se on**riippumaton kehyskoolta**, joten**binning ja pienemmät ROI-alueet EIVÄT auta** (ne vähentävät tavuja *kehyksessä*, eivät GevSCPD:n tahdittamia tavuja *sekunnissa*); ainoat ratkaisut ovat kameramäärän vähentäminen, jumbo-kehykset päästä päähän tai nopeampi verkkokortti. Oireena olisi GVSP-pakettihäviö, ei asteittainen kuvanopeuden lasku, joten `analyze-array` nollaa saavutettavissa olevat fps-luvut ja tulostaa `**OVER-SUBSCRIBED**`, ja `array-connect` kiinteällä resoluutiolla **kieltäytyy muodostamasta yhteyttä** (muutoin walk-down-menetelmä ryhmittelee kehykset pienempään luokkaan, mikä ei myöskään poista tämän luokan estettä). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` alentaa kieltäytymisen äänekkäksi varoitukseksi testityötä varten — katso [Ympäristömuuttujat](#environment-variables).

### Järjestelmän kunto — mikä alijärjestelmä menettää kehyksiä

Yhdistetyn järjestelmän `GET /api/camera/array/<array_id>/capability` sisältää reaaliaikaisen
`health`-lohkon, jota arvioidaan uudelleen **10 sekunnin** liukuvassa ikkunassa. Se jakaa kehysten menetykset
kahteen syyhyn, jotka vaativat vastakkaisia korjauksia, sen sijaan että raportoitaisiin yksi ”epätäydellinen”
aste, joka ei nimeä kumpaakaan:

| Kenttä | Mitä se tarkoittaa | Mikä alijärjestelmä |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (sarjaliikennettä kohti) | Kehys **saapui ja oli rakenteellisesti viallinen**— GVSP-paketin menetys. |**Verkko**: kaapelibudjetti, tahdistus, NIC RX-rengas, MTU |
| `never_arrived_rate_pct` (sarjanumeroittain) | Kehystä **ei tullut lainkaan**— kamera ei laukaisi tai siitä ei lähtenyt mitään. |**Laukaisu / synkronointi**: M8-kaapeli, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Kunkin kameran huonoin lähetysnopeus. | — |
| `per_cam_rate_pct` | Yhdistetty epätäydellisyysaste kameraa kohti (molemmat syyt yhteensä). | — |
| `stable_for_seconds` | Kuinka kauan kukin kamera on pysynyt alle 0,01 %:n. | — |

Yli 5 %:n rajan ylittyessä taustajärjestelmä kirjaa `[array-health <id>] WARN`-rivin, jossa mainitaan jakautuma — ensimmäisen
rikkomuksen yhteydessä, vakavuusluokan muuttuessa, kerran minuutissa niin kauan kuin tilanne jatkuu, ja kerran, kun
tilanne korjaantuu. Viallinen puoli tulostaa `[gvsp-corrupt <SN>]`-rivin ensimmäisen osuman yhteydessä kameraa ja
syytä kohden, minkä jälkeen se tekee yhteenvedon 60 sekunnin välein. Jokainen arviointi tallentuu silti taustapalvelimen lokitiedostoon;
laskurit etenevät jokaisen puskurin kohdalla riippumatta siitä, mitä tulostetaan.

Sama tietue ilmoittaa koko allokoinnin kokonaismäärän:

| Kenttä | Merkitys |
| --- | --- |
| `wire_ceiling_mbps` | Isäntäkoneen voimassa oleva jatkuva siirtokapasiteetti, MB/s. |
| `wire_ceiling_source` | Mistä luku on peräisin, sanoin — esim. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` tai `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, kun `--wire-ceiling-mbps` (tai GUI:n **Wire Budget**-kenttä) on asettanut sen. |
| `nic_is_usb` | `true` USB-Ethernet-sovittimelle — katso yllä oleva 200 MB/s:n yläraja. |

**Lukeminen:** nollasta poikkeava `gvsp_corrupt_rate_pct` ja `never_arrived_rate_pct` arvona 0
tarkoittaa, että laukaisu ja kaapelisynkronointi ovat täydellisiä ja 100 % häviöstä johtuu verkkoreitistä
— pienennä `--wire-ceiling-mbps`-arvoa ja muodosta yhteys uudelleen. Päinvastainen kuvio viittaa sen sijaan
synkronointikaapeliin tai laukaisulinjaan.

> **`--target-fps` ei ole ratkaisu vioittuneisiin kehysjonoihin.** GevSCPD-taajuus määritetään
> kerran yhteyden muodostuksen yhteydessä, joten laukaisunopeuden laskeminen muuttaa käyttöjaksoa, ei
> samanaikaisten lähetysten purskeiden nopeutta. Mitattu 5-kertainen vaatimusten leikkaus ei tuottanut parannusta;
> kaapelirajan laskeminen 240:stä 200 MB/s:iin laski saman laitteiston vioittuneiden kehysten osuuden 10,4 %:sta
> 0:aan.00 %:iin.

> **TRI032S-laiteohjelmistossa ei ole käytettävissä lähetyksen keskivaiheilla tapahtuvaa automaattista supistusta.** Käynnissä oleva ryhmä
> ei voi korjata tätä itse; katkaise yhteys ja muodosta se uudelleen, jotta yhteyden muodostushetken valitsin voi
> suunnitella uudelleen uuden ylärajan mukaisesti.

### Oire → korjaus

| Oire (Järjestelmän asetukset / yhteys / `analyze_array_network`) | Syy | Korjaus |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` palautuu arvoon 32 (tyypillisesti ohjainpäivityksen jälkeen) | Aseta `ReceiveBufferLen`→256, `PendingReceives`→64; avaa paneeli uudelleen (käynnistä taustapalvelu uudelleen, jos se on tallentanut vanhan rengaskokoa välimuistiin) |
| Käynnistys/sammutus jumittuu; myöhemmin `Invalid class` WMI-virheitä, verkkokorttia ei voi ottaa käyttöön, USB-asemat puuttuvat | Vanha Realtekin USB 10GbE -ajuri vuodelta 2016 → BSOD `0x9F` → pakotetut virrankatkaisut | Päivitä sovittimen ajuri versioon ≥ v10.67 (2026) ja aseta sitten edellä mainitut vastaanottorenkaan asetukset uudelleen |
| Yhteyden muodostaminen onnistuu, mutta palauttaa natiivia pienemmän resoluution | Smart-prep pienensi kehyksen automaattisesti kaapelin mukaiseksi | Päivitä linkki / hyväksy pienennys / `--force-tier slip-emit-and-capture` |
| Array ilmoittaa terveen tavoite-fps:n, mutta toimittaa vain murto-osan siitä; `health.gvsp_corrupt_rate_pct` ei nolla, `never_arrived_rate_pct` 0 | Isäntälaitteen päätelty kaapelibudjetti yliarvioi sen, mitä se tosiasiallisesti kestää (tyypillistä USB-Ethernet-sovittimessa, kapealla PCIe-kaistalla tai jaetussa verkossa) | Yhdistä uudelleen alhaisemmalla `--wire-ceiling-mbps`-arvolla ja tarkista toimintakunto-lohko uudelleen. **Ei** `--target-fps` — GevSCPD-taajuuden säätö on kiinteä yhteyden muodostuksen yhteydessä |
| Kameroita puuttuu julkaistuista ryhmistä; `health.never_arrived_rate_pct` einolla, `gvsp_corrupt_rate_pct` 0 | Laukaisu-/synkronointireitti — kamerat eivät laukea, ei verkko-ongelma | Tarkista M8-synkronointikaapeli ja `--line`; varmista, että jokainen jäsen on valmiustilassa (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` ylitetty `analyze-array`:ssa, tai yhteyden muodostaminen epäonnistui kiinteällä resoluutiolla (`array over-subscribes the wire`) | Kameroiden yhteenlaskettu kysyntä (vähintään 8 MB/s × N kameraa) ylittää törmäyksiltä suojatun kaistanleveyden ylärajan — 6 kameraa täysresoluutiolla 1 GbE:llä @1500 MTU, 9 jumbo-kehyksillä | Vähemmän kameroita, jumbo-kehykset päästä päähän tai nopeampi verkkokortti. **ROI/binning EI auta** (raja on kehyskokosta riippumaton). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` ohittaa testialustalla (hyväksyy pakettihäviön) |

---

## `chloros-cli daq`

Spectral-anturikomennot. Kaksi luokkaa:
- **`pool-*`**— ohutHTTP-asiakasohjelmat, jotka ohjaavat anturia backendin pysyvän poolin kautta.**Tämä on tuettu reitti, ja ainoa, joka on mukana toimitetussa CLI-ohjelmistossa.** Taustapalvelu hallinnoi siirtoa, joten graafinen käyttöliittymä sekä skriptit CLI ja SDK jakavat kaikki yhden aktiivisen käsittelykäsittimen sen sijaan, että ne kilpailevat sarjaportista.
- **Kaikki muut**(`test`, `record`, `live`, `stream`, `connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`, `serve`, `ws`, `udp`, `mqtt`, `reflectance`, `login`, `logout`, `status`) — suora laitteistoyhteys, joka on dokumentoitu alla täydellisyyden vuoksi. Nämä vaativat `daq`-Python-paketin, joka**ei sisälly mihinkään toimitettuun artefaktiin**: käännetty CLI jättää sen pois (`scripts/Build-CLI.ps1` asettaa `--nofollow-import-to=daq`:n, ja siirto-ohjelmat `pyserial` / `bleak` / `zeroconf` sisältävät sen), eikä PyPI-SDK-paketti sisällä sitä myöskään. Ne toimivat vain lähdekoodin checkoutista, joten pidä niitä pikemminkin MAPIR-sisäisenä kehityspoluna kuin yleisesti saatavilla olevana ratkaisuna.
- **`discover` / `list`** sijoittuvat näiden kahden välimaastoon: ne ovat suoria laitteistokomentoja lähdekoodin tarkistuksesta, mutta valmiissa rakennuksessa ne siirtyvät käyttämään `pool-discover`:ää ja taustapalvelin suorittaa skannauksen. Skannaus toimii siis kaikkialla — mikä on tärkeää, koska se on ainoa tapa selvittää DAQ-M:n BLE MAC -osoite.

> **`chloros-cli daq --help`** (sekä `-h` / `help`) luettelee `pool-*`-alikomennot — ohjeet ohjataan tarkoituksella pool-asiakasohjelmaan, jotta ne heijastavat komentoja, jotka todella suoritetaan. Jos kutsut suoraa laitteistokomentoa toimitetussa rakennuksessa, se päättyy nimenomaisella virheilmoituksella, jossa mainitaan puuttuva paketti ja ohjataan sinut takaisin komentoon `pool-*`; mikään ei epäonnistu hiljaisesti. (`discover` / `list` ovat poikkeus — ne ohjaavat uudelleen komentoon `pool-discover` ja toimivat ilman ongelmia.)
>
> **Kaikki, mitä asiakas tarvitsee, on saatavilla `pool-*`:n kautta** — yhteyden muodostaminen, datavirran lähetys, kalibroitujen `.daq`-tiedostojen tallennus ja kapseliprofiilien vaihto. DAQ-laitetta voidaan ohjata myöPython-palvelimelta käyttämällä `chloros_sdk.connect_daq_sensor()`:ääX:n avulla, joka käyttää samaa yhdistettyä polkua.

### DAQ-anturin ensimmäisen yhteyden muodostamisen työnkulku

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### `pool-*`-viite

| Alikomento | Tarkoitus |
| --- | --- |
| `daq pool-connect` (smart-detect) | Avaa anturi taustapoolissa. |
| `daq pool-connect --port PORT` | DAQ-U tietyllä sarjaportilla. |
| `daq pool-connect --ble` | DAQ-M BLE:n kautta, MAC-osoite skannataan automaattisesti. |
| `daq pool-connect --mac MAC` | DAQ-M BLE-yhteydellä tunnetulla MAC-osoitteella (tarkoittaa `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E Ethernet-yhteyden kautta tunnetulla isäntäkoneella. |
| `daq pool-connect --eth` | DAQ-E Ethernet-yhteydellä, isäntä löydetty automaattisesti (mDNS + ARP-varajärjestelmä; toimii tyhjästä ARP-välimuistista osoitteissa Windows ja Linux). |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | Säädä integraatioikkunaa / AE-tilaa. |
| `daq pool-connect --no-stream` | Yhdistä, mutta älä aloita vielä datavirtaa (jatka komennolla `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Kapasiteetin korjausprofiili. Oletusarvo taustapuolella on `sunshine_cosine`. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Skannaa jokainen siirtoyhteys etsimällä antureita, joihin voisit muodostaa yhteyden, ilman että muodostat yhteyttä. **Näin löydät DAQ-M:n BLE-MAC-osoitteen.** `daq discover` / `daq list` reititetään tänne automaattisesti toimitettavissa versioissa. Poolissa jo avoimina olevia antureita ei näytetä luettelossa – kytketty DAQ-M lopettaa mainostamisen – joten käytä niihin komentoa `pool-list`. |
| `daq pool-list` | Näytä kaikki anturit taustapoolissa. |
| `daq pool-disconnect --sensor-id ID [--all]` | Vapauta. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Viimeisimmät N spektrikehykset. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Jatka / keskeytä suoratoisto. |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | .daq-tallennuksen aloittaminen / lopettaminen. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Cap-korjausprofiilin vaihtaminen suorituksen aikana. |

### Suorat laitteistokäskyt (vain lähdekoodista ladattavissa — eivät sisälly toimitettuihin versioihin)

> Lueteltu kattavuuden vuoksi. Nämä edellyttävät `daq`-Python-pakettia sekä `pyserial` / `bleak` / `zeroconf`-komentoja, joista yksikään sisällytetään käännettyyn CLI tai PyPI:n SDK versioon — ne toimivat vain MAPIR lähdekoodin latauksesta. **Jos käytät julkaistua Chloros rakennetta, käytä sen sijaan yllä olevia `pool-*`-komentoja**; ne kattavat yhteyden muodostamisen, streamauksen, tallennuksen ja kap-valinnan.

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

Avaa, muodosta yhteys ja ohjaa tallennettua Chloros-projektia (kansio, jossa on `cameras.json` + `sensors.json` + `project.json`). Kaikki reititetään taustapalvelimen kautta, joten käyttöliittymä ja CLI tuottavat identtisen laitteiston tilan.

### Alikomentojen viite

| Alikomento | Tarkoitus |
| --- | --- |
| `project open PATH` | Tulostaa projektin laitemanifestin (kamerat, ryhmät, anturit). |
| `project devices PATH [--reconnect]` | Luettelee tai suorittaa laitteiden tunnistuksen uudelleen. |
| `project connect PATH [--cameras-only] [--sensors-only]` | Yhdistää kaikki tallennetut kamerat / matriisin / anturin. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Yksittäinen kuvaus nimetystä kamerasta tai matriisista. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | N-kuvan sarja nimetystä kamerasta tai matriisista (`-n/--count` oletus 5; `-i/--interval` kuvien välinen aika sekunteina, oletusarvo 0). Ryhmäkuvasarjat poistavat toistuvat synkronoidut ryhmät (vanhentuneisuuden valvonta), jotta osittain kiertävä ryhmä voivoi palauttaa N kopiota yhdestä kehyksestä; tulostaa tulokset jokaisen iteraation jälkeen. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | Virtaus levylle taustatehtävän kautta. `--poll-interval` = sekuntia `/stats`-kyselyjen välillä (oletus 2,0). |
| `project sensor read PATH NAME [--json]` | Uusin spektrikehys. |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | Tallenna .daq-tiedosto. |
| `project run PATH RECIPE.yaml` | Suorita YAML/JSON-tallennusresepti. `--dry-run` tarkistaa ilman suorittamista. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Laske taulukon kohdistus — katso [alla oleva lipputaulukko](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Tulosta nykyinen kohdistusprofiili. |
| `project align clear PATH NAME` | Poista välimuistissa oleva profiili. |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | Siirtää yhden orjan muunnosta. |
| `project align export PATH NAME --to FILE` | Tallentaa profiilin tiedostoon JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Lataa tallennettu profiili. |

#### `project align calibrate`-asetukset

| Lippu | Oletus | Kuvaus |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Kohdistusmenetelmä. **Nämä kirjoitusmuodot eroavat kohdasta `lattice align-calibrate`**, jossa käytetään lyhenteitä `orb` / `akaze` / `phase`; nämä kaksi komentoa eivät ole keskenään vaihdettavissa tässä lippukentässä. |
| `--model {translation, rigid, affine, homography}` | `affine` | Muunna malli sopivaksi. |
| `--frames N` | `1` | Synkronoi kehyskuvat keskiarvoon. |
| `--reference SN` | pääkamera | Vertailukameran sarjanumero; kaikki muut jäsenet vääristetään sen mukaan. |
| `--max-features N` | `5000` | ORB-piirteiden lukumäärän yläraja. |
| `--ratio-threshold F` | `0.75` | Lowen suhdetesti. |
| `--ransac-threshold-px F` | `3.0` | RANSAC:n sisäpisteiden kynnysarvo. |
| `--min-matches N` | `15` | **Laatukriteeri** — hylkää ratkaisu, jos inlier-vastaavuuksien määrä on alle tämän rajan. |
| `--max-reproj-err-px F` | `4.0` | **Laatukynnys** — hylkää ratkaisu, jos RMS-uudelleenprojektio-virhe ylittää tämän rajan. |
| `--checkerboard RxC` | — | `--method checkerboard`:n piirilevyn geometria, esim. `9x6`. |
| `--name PROFILE` | tyhjä | Tallennettuun JSON-tiedostoon upotettu profiilin nimi. **Ei taulukon nimeä** — se on sijaintitunnus `NAME`. |

Nämä kaksi laatuporttia ovat syy siihen, että kalibrointi voi onnistua ratkaisussa mutta silti
kieltäytyä tallentamasta: profiili, joka epäonnistuu jommassakummassa, rekisteröisi hiljaisesti väärin jokaisen
myöhemmän tallennuksen, joten se hylätään sen sijaan, että se tallennettaisiin.

### Esimerkkejä

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### Resepti-DSL

`project run RECIPE.yaml` hyväksyy YAML- tai JSON-tiedoston, joka kuvaa toimintojen sarjaa:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Tuetut toiminnot: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. Toiminto `burst` vaatii parametrit `name` (pakollinen), `count` (oletusarvo 5), `interval` (sekuntia, oletusarvo 0), `output`, `format` ja `settings` (sama kamerakohtainen asetusmuoto kuin `apply`); array-sarjakuvaukset käyttävät samaa juuri synkronoitua ryhmän valvontaa kuin `project burst`.

Suorita se:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## Ympäristömuuttujat

| Muuttuja | Vaikutus |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Ohittaa taustaprosessin URL (oletus `http://127.0.0.1:5000`) — **huomioidaan vain komentojen `lattice`-, `project`- ja `daq pool-*`-komentoperheet.** Ydinkomennot (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) ohittavat `http://127.0.0.1:<port>`:n ja jättävät tämän muuttujan huomiotta (IPv4-literaali ohittaa Windows `localhost`→`::1` ~2 s:n viiveen pyyntöä kohti), joten ne kohdistuvat aina paikalliseen koneeseen. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1` alentaa taulukon ylimerkinnän aiheuttaman yhteyden hylkäyksen (kameraa kohti laskettu kokonaiskysyntä &gt; törmäysturvallinen siirtokapasiteetin yläraja, kun käytössä on `pin_resolution`) voimakkaaksi varoituksella ja jatkamisella, hyväksyen GVSP-pakettihäviöt. Vain testikäyttöön — katso [Matriisin fps- ja pursemalli](#array-fps--burst-model). |
| `CHLOROS_CLI_MODE` | Asetetaan itse CLI:n toimesta; käskee taustapalvelua ottamaan rinnakkaiskäsittelyn käyttöön. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` ohittaa GVSP-varatestauksen (vain ICMP-tulokset). **Tämä poistaa jumbo-paketit käytöstä, se ei pelkästään vaimenna lokia** — kamera vastaa DF-ping-kyselyihin vain 1500:aan asti jokaisella reitillä, joten tämä testi on ainoa keino havaita jumbo-paketit. Säästää ~1 s kameraa kohti yhteyttä kohti; maksaa ~1,45× kaapelin enimmäiskapasiteetin, jos verkko *olisi voinut* siirtää jumbo-paketteja. SDK antaa varoituksen, kun asetat tämän. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Kiinnittää GVSP-paketin koon arvoon N tavua; ohittaa testauksen kokonaan. Suosittelemme komentoa kohti (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) pysyvän asetuksen sijaan: kiinnitetty koko estää sopeutumisen edessä olevaan verkkoon, ja arvon 9000 kiinnittäminen reitille, joka ei kykene siirtämään jumbo-paketteja, aiheuttaa **jokaisen** kaappauksen aikakatkaisuun `SC_ERR_TIMEOUT -1011`:n kanssa. |
| `TMPDIR` (Linux) | Ohita Nuitkan onefile-purkukansio. CLI käyttää automaattisesti `/mnt/ssd/tmp`:ää, jos se on määritetty. |

---

## Lopetuskoodit

| Koodi | Merkitys |
| --- | --- |
| `0` | Onnistui. |
| `1` | Yleinen virhe (useimmat alikomentojen virheet). |
| `2` | Argumenttivirhe. |
| `130` | Keskeytetty Ctrl+C-näppäimellä. |

---

## Vianmääritysohjeita

- **&quot;Kirjautuminen vaaditaan&quot;** → Suorita `chloros-cli login EMAIL PASSWORD` kerran tällä koneella.
- **&quot;backend-palvelinta ei tavoiteta&quot;** → Käynnistä Chloros-työpöytäsovellus, tai suorita backend-binaari suoraan (`chloros-backend`), tai tarkista `CHLOROS_BACKEND_URL`, jos kyseessä on etäkäyttö.
- **`lattice`-komennot epäonnistuvat virheilmoituksella &quot;LATTICE-kameran ajureita ei löydy&quot;** → Arena-SDK -ajoympäristöä ei ole asennettu; CLI toimitetaan yhdessä `win32api`:n kanssa osoitteessa Windows, mutta C-ajoympäristö on osa graafisen käyttöliittymän asennusohjelmaa.
- **Array connect / Array Settings -kohdassa näkyy &quot;FRAMES WILL DROP&quot; tai &quot;Reduce ROI to enable&quot;** → Isäntäkoneen verkkokortin vastaanottorengas on liian pieni (palautuu yleensä arvoon 32 verkkokortin ohjaimen päivityksen jälkeen). Katso [Isäntäverkkokortin asetukset ja säätö](#host-nic-setup--tuning-lattice-arrays) — aseta `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Kone jumittuu uudelleenkäynnistyksen/sammutuksen yhteydessä, minkä jälkeen WMI `Invalid class` / verkkokorttia ei voi ottaa käyttöön / USB-asemat puuttuvat** → Vanhentunut USB 10 GbE -sovittimen ohjain aiheuttaa virheen `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Päivitä sovittimen ajuri — katso [Isäntäverkkokortin asetukset ja säätö](#host-nic-setup--tuning-lattice-arrays).
- **Jetsonin swap-varoitus** → Lisää tiedostopohjainen swap-tila; CLI-komento tulostaa tarkat `fallocate` / `swapon`-komennot.
- **DAQ-suorakomennot puuttuvat** → Odotettu: mukana toimitettu `chloros-cli` jättää tarkoituksella `daq`-paketin pois, joten vain `pool-*` on läsnä (PyPI-SDK-sivustollakaan sitä ei ole saatavilla). Käytä `pool-*`-pakettia, joka ohjaa samaa anturia taustapalvelimen kautta, tai `chloros_sdk.connect_daq_sensor()`-pakettia osoitteesta Python.

---

## Katso myös

- [Python SDK-viite](sdk-reference.md) — ohjelmointikielinen vastine jokaiselle CLI-komennolle.
- [DAQ-anturiohje](../daq/README.md) — anturikohtaiset kytkentäohjeet ja kalibrointi.
- Verkkodokumentaatio: `https://mapir.gitbook.io/chloros/cli`
