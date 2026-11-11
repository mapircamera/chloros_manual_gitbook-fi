# CLI: Komentorivi

> **Täydellinen ohje:**[CLI-ohje](reference/cli-reference.md) sisältää**kaikkien alikomentojen kaikki lipput** ja on optimoitu tekoälyavustajille — liitä sen URL-osoite avustajaasi ja pyydä toimiva komento: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Vinkki tekoälytyökaluille:** mikä tahansa tämän käyttöoppaan sivu on saatavilla raakamuotoisena Markdown-tiedostona lisäämällä sen URL-osoitteeseen `.md` (esim. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), ja `https://mapir.gitbook.io/chloros/llms.txt` indeksoi koko käyttöoppaan LLM-käyttöä varten.

> **Haluatko, että avustajasi suorittaa Chlorosin itse?** `chloros-mcp`, MCP-palvelin, joka asennetaan Python-SDK:n mukana, toimii Claude Desktopin, Claude Coden, Cursorin ja muiden MCP-asiakasohjelmien kanssa: katso [Chlorosin käyttö tekoälyavustajien kanssa](ai-assistants.md).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.3.0 build so the version line reads "Chloros CLI 1.3.0" -->


## Mikä on CLI


`chloros-cli` on komentoriviliittymä samalle käsittelymoottorille, jota Chloros
-työpöytäsovellus käyttää. Se on kevyt HTTP
-asiakasohjelma, joka käyttää Chloros
-taustapalvelinta (paikallinen palvelin `127.0.0.1:5000`:llä) — useimmat komennot käynnistävät taustapalvelimen automaattisesti, joten skripti tarvitsee vain yhden `chloros-cli process …`-kutsun. Taustapalvelin pysyy tämän jälkeen käynnissä seuraavaa komentoa varten: se pysähtyy komennolla `chloros-cli logout` (ellei sovellus ole auki tai suoritus tai koontiprosessi ole käynnissä), komennolla `chloros-cli backend stop` tai itsestään 30 minuutin kuluttua, jos asiakasta, suoritusta tai kytkettyä laitetta ei ole. `chloros-cli backend status` näyttää tämän.

Se toimii **Windows
10/11 (x64)**- ja**Linux
(x86_64 sekä NVIDIA Jetson arm64 JetPack 6:ssa)**-ympäristöissä missä tahansa terminaalissa ilman graafista käyttöliittymää. Varmista asennus seuraavalla komennolla:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

Komentoryhmät yhdellä silmäyksellä:

* **Käsittely ja tili** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 kieltä — katso [Tuetut kielet](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (vain Linux
/Jetson)
* **Käytössä oleva laitteisto** — `lattice` (LATTICE-kameran ohjaus, yli 45 alikomentoa), `daq pool-*` (DAQ-valosensorit), `time-sync` (PTP)
* **Automaatio** — `project` (tallennetun Chloros
-projektin ajaminen ilman käyttöliittymää, mukaan lukien YAML-tallennusohjeet)

Hyödyllisiä yleisiä asetuksia: `--port N` (taustapalvelimen portti, oletus `5000`), `-v/--verbose`, `--restart` (pakota taustapalvelimen uudelleenkäynnistys), `--backend-exe PATH`. Katso täydellinen luettelo [CLI
-ohjeesta](reference/cli-reference.md).

***

## Asennus

CLI
**sisältyy Chloros
-asennusohjelmaan** kaikilla alustoilla — erillistä CLI
-latausta ei ole. Lataa asennusohjelma [Lataa](download.md)-sivulta.

### Windows


Asennusohjelma sijoittaa CLI
:n seuraavaan kansioon:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

ja lisää kyseisen kansion järjestelmään `PATH` kerran (päivitys poistaa myös aiempien asennusohjelmien jättämät päällekkäiset merkinnät) — **avaa uusi terminaali**asennuksen jälkeen, jotta päivitetty `PATH` tunnistetaan. Asennusohjelma tallentaa myös käynnistyskomentosarjat (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) asennuksen juurikansioon sekä**Chloros
CLI
** pikakuvakkeen Käynnistä-valikkoon, joista kumpikin avaa terminaalin, jossa `chloros-cli` on käyttövalmis.

### Linux


Asenna arkkitehtuurillesi sopiva `.deb` komennolla `apt` (ei `dpkg -i`, joka ohittaa Python
-SDK
:n suositellut vaatimukset):

```bash
# Linux x86_64
sudo apt install ./chloros_1.3.0_amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
```

Tämä asentaa `chloros-cli`:n versioon `/usr/bin/chloros-cli` (jo asennettuna versioon `PATH`), taustapalvelin ja CLI
-ohjelmat puretaan versioihin `/usr/lib/chloros/backend/` ja `/usr/lib/chloros/cli/` (`/usr/lib/chloros/chloros-backend` on käynnistysohjelma, joka käynnistää taustaprosessin), Arena SDK
-ajoympäristö, jota taustaprosessi tarvitsee LATTICE-kameroille, sekä järjestelmän Python
-SDK
`python3`. Katso lisätietoja kohdasta [Linux
-asennus](linux/linux-installation.md).

### Varmista

```bash
chloros-cli --version    # "Chloros CLI 1.3.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Kirjautuminen ja lisensointi

CLI
:n (ja Python
-SDK
:n) käyttö edellyttää **maksullista Chloros
+-pakettia**— se sisältyy kaikkiin maksullisiin tasoihin, mutta ei ilmaiseen tasoon. Raja valvotaan**palvelinpuolella** taustapalvelimen toimesta, ei CLI
-binaarin toimesta: kirjautumattoman käyttäjän kutsu hylätään virheellä `401 AUTH_REQUIRED`, ja kirjautuneen käyttäjän pyyntö ilmaisella tasolla virheellä `403 PLAN_UPGRADE_REQUIRED`, riippumatta siitä, tuleeko se `chloros-cli`:sta, SDK
:sta vai itse kehitetyltä HTTP
-asiakasohjelmalta. Päivitä osoitteessa [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Kirjaudu sisään **kerran kutakin laitetta kohti**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.3.0 build showing the license tier line -->


{% hint style="warning" %}
**Salasanat, joissa on erikoismerkkejä**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` sekoittuu komentotulkissa (CLI
havaitsee tämän 401-virheen yhteydessä ja yrittää uudelleen automaattisesti, mutta yksinkertaiset lainausmerkit estävät ongelman kokonaan).
{% endhint %}

Istunto tallennetaan välimuistiin tunnuksella `~/.chloros/user_session.json` ja toimii edelleen offline-tilassa tilauksen armonaikana (30 päivää kuukausitilauksissa, vuositilauksissa voimassaolon päättymiseen asti). `chloros-cli status` toimii myös ilman maksullista tilausta, joten hylkäyksen syy on aina näkyvissä.

{% hint style="danger" %}
**Aiotko ajoittaa headless-tehtäviä? Kirjaudu ensin sisään.**Taustaprosessia käynnistävä komento (`process`, `status`, `export-status`, …) suoritettuna**ilman välimuistissa olevaa istuntoa**ei epäonnistu nopeasti — se siirtyy interaktiiviseen `Email:` / `Password:`-kehotteeseen stdin:ssä. Tämän vuoksi automaattinen cron-tehtävä tai CI-vaihe**jää odottamaan syötettä**. Suorita `chloros-cli login EMAIL 'PASSWORD'` kerran koneella ennen kuin ajoitat mitään.
{% endhint %}

***

## Ensimmäinen käsittelykierros

Suuntaa `process` tallennettujen tiedostojen kansioon — se tunnistaa automaattisesti Survey3
:n (`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` tai niiden yhdistelmän:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Edistymisvirrat näkyvät reaaliaikaisesti kunkin putkistoketjun säikeen mukaan (tunnistus, analysointi, käsittely, vienti), ja onnistunut suoritus päättyy ilmoittamalla, kuinka monta kuvatuotetta se on tallentanut: `Image products written: N`, jota seuraa `(plus M clamp masks)`, jos suoritus tallensi myös [clamp-maskeja](output-image-formats.md#clamp-masks) (niitä ei lasketa tuotteiksi).

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N (plus M clamp masks)" summary line -->


Jos Chloros
-taustapalvelu on jo käynnissä (esimerkiksi työpöytäsovelluksen), `process` käyttää sitä ja avaa projektinsa siinä. Se kieltäytyy, kun sovellus on auki projektissa, jonka sovellus on avannut: suorita prosessi sovelluksesta tai sulje kyseinen projekti. Jos edellinen suoritus on vielä päättymässä, se odottaa sitä enintään 15 sekuntia ennen kieltäytymistä. `--restart` pakottaa uuden taustaprosessin käynnistymisen. Linux
issa systemd-palveluna käynnissä olevaa taustapalvelua ei koskaan käynnistetä uudelleen: `process` käyttää sitä sellaisenaan, ja tilanteissa, joissa se muuten käynnistäisi sen uudelleen (`--restart` tai tuonti, joka ei löytänyt mitään), se tulostaa virheen `sudo systemctl restart <unit>` ja lopettaa toimintansa `1`.

### Mihin tulosteet tallennetaan

`process` tallentaa tiedostot **projektikansioon**, ei syöttökansioosi:

* Jos `-o`-asetusta ei ole määritetty: projekti luodaan oletusprojektikansioosi (joka on yhteinen graafisen käyttöliittymän kanssa; hallitse sitä komentojen `get-project-folder` / `set-project-folder` avulla; jos mitään ei ole määritetty, `Documents/MAPIR/Chloros Projects` käyttäjäkansiossasi), ja sen nimeksi tulee `-n/--project-name` tai aikaleima (`YYYYMMDD_HHMMSS`), jos nimeä ei ole annettu.
* Kun käytetään `-o PATH`: kyseinen kansio **on** projektikansio. Jos se sisältää jo `project.json`-tiedoston, luodaan sen sijaan `_1`/`_2`… -päätteinen sisarkansio sen sijaan, että se korvattaisiin. Suhteellinen `PATH` määritetään suhteessa kansioon, josta `chloros-cli` suoritetaan.

{% hint style="info" %}
**Suhteelliset polut (muutettu versiossa 1.3.0).** Jokainen polku, jonka taustaprosessi avaa — `-o`/`--output` millä tahansa komennolla (mukaan lukien tallennuskomentojen oletusarvoinen `output`), `--input`, `--burst-dir`, `--profile`, `--save` ja `--mask` — määritetään sen kansion perusteella, josta `chloros-cli`-komento suoritetaan, joten `-o captures/` kirjoittaa tiedostoon `<current folder>/captures/`. Versiossa 1.2.0 taustapalvelin ratkaisi ne omassa asennuskansiossaan, mikä epäonnistui virheilmoituksella ”Pääsy evätty”. Etätaustapalvelinta (`CHLOROS_BACKEND_URL`) vastaan polut lähetetään sellaisina kuin ne on kirjoitettu, ja ne ratkaistaan kyseisellä koneella.
{% endhint %}

{% hint style="warning" %}
**Windows
: pidä projektikansio lyhyenä, ellei pitkiä polkuja ole käytössä.** Kun Windows
in pitkät polut on kytketty pois päältä (Windows
in oletusasetus), mikään tiedostopolku ei saa olla pidempi kuin 260 merkkiä, ja tuotteet sijaitsevat enintään noin 130 merkkiä projektikansion alapuolella. `process` tarkistaa tämän ennen käynnistymistä: liian syvä projektikansio hylätään viestillä **”Projektikansio on liian syvä tälle tietokoneelle”**, ilmoitetaan pisin sopiva projektikansio ja annetaan ohjeet pitkien polkujen ottamiseksi käyttöön — mitään ei käsitellä. Käytä lyhyempää `-o`-tiedostoa (tai `set-project-folder`) tai ota Windows
in pitkät polut käyttöön (asennusohjelman **Ota Windows
in pitkät polut käyttöön** -vaihtoehto, valittuna oletuksena) ja käynnistä Chloros
uudelleen.
{% endhint %}

Projektin sisällä tuotteet on ryhmitelty **kameran mukaan ja sen jälkeen tiedostomuodon mukaan**:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

LATTICE-kameran kansio on `LATT-<sensor>-<lens>-F<filter>` (vastaa kuvan EXIF-tietoja `Model`) ja `<model>_<filter>` (esim. `Survey3N_RGN`) on Survey3
:lle. Tiedostomuotokansio noudattaa järjestystä `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` tai `tiff32`, kun kyseessä on `TIFF (32-bit, Percent)`.

Monispektrisen M3C-kameran tapauksessa formaattikansioon voi tuotteiden kansioiden lisäksi ilmestyä myös `clamp_masks/`-kansio (esimerkiksi `tiff16/clamp_masks/Reflectance_Calibrated_Images/` ja `tiff32/clamp_masks/Radiance_Images/`). Se sisältää säteily- ja heijastavuustuotteiden pikselikohtaiset naamiot, joissa kaistan erottelussa negatiivinen arvo on rajoitettu nollaan — katso [Rajoitusnaamiot](output-image-formats.md#clamp-masks). Mikään tuotekansio ei koskaan sisällä naamiota.

{% hint style="info" %}
**Jokainen viety tuote säilyttää LÄHDE-tiedoston nimen.**`capture_..._raw.tif`:n säteilyvoimakkuusvienti on edelleen nimeltään `capture_..._raw.tif` — se vain sijaitsee kansiossa `tiff32/Radiance_Images/`.**Kansio tunnistaa tuotteen, ei tiedostonimi**, joten käytä yleismerkintää hakemistolle, älä `*radiance*`-päätteelle.
{% endhint %}

### Vaihtoehdot, joita tulet tosiasiallisesti käyttämään

| Lippu | Oletus | Toiminto |
| --- | --- | --- |
| `-o, --output PATH` | oletusprojektikansio | Projektikansion sijainti (katso yllä). |
| `-n, --project-name NAME` | aikaleima | Projektin nimi. |
| `--format FMT` | `TIFF (16-bit)` | Jokin seuraavista: `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros
ei skaalaa kuvaa suuremmaksi: kaikissa 8-bittisissä kuvissa syvempi formaatti tallennetaan nimellä `TIFF (8-bit)`, ja tämä mainitaan kerran. |
| `--indices NAME [NAME ...]` | ei mitään | Viedyt kasvillisuusindeksit (katso [Kasvillisuusindeksit](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = neuroverkko-debayer, hitaampi, korkein laatu (Chloros
+, vaatii CUDA-GPU:n: ilman sitä suoritus käyttää `standard`; hylätään LATTICE-kuvauksissa, joissa käytetään aina `standard`). |
| `--vignette / --no-vignette` | päällä | Vignettikorjaus. |
| `--reflectance / --no-reflectance` | päällä | Heijastavuuskalibrointi; LATTICE-kuvissa tämä toimii myös heijastavuustuotteen kytkimenä. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Pakota putkilinjan aloituskohta LATTICE-TIFF-tiedostoille. |

Kaikki muu — kohteen tunnistuksen säätö, PPK, valotuspisteet, matriisin kohdistusliput — katso [CLI
-viitteen `process`-osio](reference/cli-reference.md).

***

## Vienti kohteiden valinta (LATTICE-tuotteet)

LATTICE-käsittely haarautuu **kaikkiin soveltuviin tuotteisiin yhdellä kierroksella**. Neljä tuotekohtaista kytkintä ovat kaikki**oletusarvoisesti päällä**; käytä lomaketta `--no-` poistaaksesi yhden:

| Kytkin | Tuote |
| --- | --- |
| `--debayered` | Lineaarinen demosaic → `Debayered_Images/`: kalibrointipaketin tumman offsetin poisto, minkä jälkeen reuna-herkkä demosaic ilman lineaarisuuskorjausta, tasoitusta tai sekoituksen purkua — sama tiedosto, jonka kaappaus tallentaa `debayered`-tasollaan. Säilytetään anturin alkuperäisessä suunnassa (tallennettu XMP:hen). |
| `--preview` | Esikatselu (valkotasapaino + gamma; väärävärinen venytys monispektrikuville) → `Preview_Images/` |
| `--radiance` | float32-tyyppinen säteilyvoimakkuus, W/m²/sr/nm → `Radiance_Images/` (aina `tiff32/`) |
| `--reflectance` | uint16 heijastavuus, Pix4D-yhteensopiva → `Reflectance_Calibrated_Images/` |

RGB
-pääkamerat lähettävät aina vain debayeroitua kuvaa + esikatselua — kaistakohtainen säteilyvoimakkuus/heijastavuus ei ole merkityksellistä laajakaistaiselle anturille, joten nämä kytkimet eivät vaikuta niihin. Survey3
`.raw` ohittaa kytkimet ja noudattaa standardin mukaista heijastavuus-/kohdepolkua.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (oletusarvo `auto`) valitsee heijastavuusviitteen: `auto` asettaa QA-testin läpäisseen kehyksessä olevan [kalibrointikohteen](calibration-targets.md) absoluuttiseksi vertailukohteeksi ja käyttää varajärjestelmänä DAQ-valosensorin alaspäin suuntautuvaa jakosuhdetta (ρ = π·L/E), kun kohdetta ei ole läsnä (laadunvarmistuksen läpäissyt kohde, jolla on lippu, kilpailee DAQ:n kanssa mitatun pistemäärän suhteen); `target` on tiukka (ei DAQ-korvausta); `daq` on DAQ-määräysvaltainen. Yksikkökohtaiset mitatut kohteen skannaukset voidaan toimittaa `--target-reflectance-dir`:n avulla.

{% hint style="info" %}
**Heijastuspikseleiden lukeminen:**DN-arvo, joka tarkoittaa ρ = 1,0, on**lähdekohtainen** — LATTICE-tiedostot merkitsevät XMP:hen tunnisteen `Chloros:PixelScale=32768`; Survey3
-tiedostot käyttävät arvoa 65535 eivätkä sisällä `Chloros:PixelScale`-tunnistetta (ne sisältävät kuitenkin muita `Chloros:*`-käsittelytunnisteita). Lue tunniste ja jaa sillä sen sijaan, että olettaisit arvon olevan vakio. Yksityiskohdat ja yksi tarkoituksellinen mittakaavaton ääritapaus löytyvät [CLI
-viitteestä](reference/cli-reference.md).
{% endhint %}

**Käsittely alkaa aina tunnisteesta `raw`.** Johdettuja tuotteita (debayeroidut/säteily-/heijastavuusvienti) ei koskaan syötetä takaisin käsittelyputkeen — niiden uudelleen tuominen ja käsittely johtaisi kalibrointilaskennan kaksinkertaiseen soveltamiseen, joten Chloros
ohittaa ne ja ilmoittaa siitä. `--input-level` on tarkoituksellisesti jätetty pakotie tilanteisiin, joissa lähtöpistettä on aidosti pakko pakottaa.

***

## Kun ajon suoritus epäonnistuu

Versiosta 1.2.0 lähtien `process` epäonnistuu selvästi sen sijaan, että se ”onnistuisi” ilman näkyviä tuloksia:

* Suoritus, joka **pyysi tuotteita mutta ei kirjoittanut yhtään**— vain `project.json` ja `calibration_data.json` — tulostaa `Processing finished but wrote no image products.` ja**päättyy nollasta poikkeavalla arvolla**, joten skriptit pystyvät havaitsemaan sen. Yleisimmät syyt: syöttökansiota ei tunnistettu kaappauksena (tarkista rakenne ja `--input-level`) tai jokainen pyydetty tuote oli soveltumaton kyseisille kameroille (esim. säteilyvoimakkuuden/heijastavuuden pyytäminen pelkästään RGB
-kameroilta).
* **Tarkoituksellinen pelkästään metatietoja tuottava suoritus** (kaikki tuotteet pois päältä, ei `--indices`) on silti onnistunut — tyhjä kuvatulos on siinä tapauksessa oikea tulos.
* Suorita ajo uudelleen käyttämällä `--verbose` ja tarkista taustalokista rivit `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, jotka selittävät kamerakohtaiset ohitukset.
* Windows
issa, jossa pitkät polut on poistettu käytöstä, projektikansio, joka on liian syvällä 260 merkin polkurajan vuoksi, hylätään **ennen** käsittelyn alkamista, ja käytetään pisintä sopivaa projektikansiota (katso [Mihin tulosteet tallennetaan](#where-the-outputs-land)).
* Tallennuskomennot (`lattice capture`, `lattice array-capture` ja muut tallennuskomentojen ryhmät) luettelevat kaikki tiedostot, joita ne eivät pystyneet kirjoittamaan, ja lopettavat toiminnon koodilla `1`; toistuvat tallennukset pysähtyvät, kun asema on täynnä.

Lopetuskoodit: `0` onnistui · `1` yleinen virhe · `2` argumenttivirhe · `130` keskeytettiin Ctrl+C:llä.

***

## Kasvillisuusindeksit

Suorita komento `--indices` yhdellä tai useammalla ennalta määritetyllä nimellä; kukin indeksi tallennetaan omaan `<INDEX>_Index_Images/`-kansioonsa:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI GNDVI OSAVI
```

22 esiasetettua nimeä, jotka `process --indices` hyväksyy:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

Projektiin lisätään nimi, jota siinä ei vielä ole. Nimi, joka ei ole esiasetettu tai jota kameroiden taajuusalueet eivät pysty tuottamaan, ohitetaan ja siitä annetaan varoitus, jossa nimi mainitaan. Jos CLI
-käyttöliittymä ei pysty lukemaan projektin nykyistä hakemistoluetteloa, se jättää luettelon ennalleen ja antaa varoituksen sen sijaan, että korvaisi sen.

{% hint style="warning" %}
**Hakemistoluetteloita on kolme — älä sekoita niitä keskenään.**GUI:n Projektin asetukset -pudotusvalikossa on 27 kaavaa (lisää `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — nämä viisi ovat käytettävissä vain käyttöliittymässä eivätkä**kelpaa** `--indices`:lle). Live-/offline-komento `lattice index --preset` käyttää omaa erillistä 22 esiasetuksen luetteloaan. Kaavat ja kaistamatematiikka on dokumentoitu kohdassa [Monispektriset indeksikaavat](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## DAQ-valosensorit: Pikaesittely

`daq pool-*`-tuoteperhe ohjaa MAPIR
-DAQ-spektrisensoreita (DAQ-U USB:n kautta, DAQ-M BLE:n kautta, DAQ-E Ethernetin kautta) taustapalvelun pysyvän poolin kautta — GUI, CLI
ja SDK
jakavat kaikki yhden live-käsittelykohteen. **`pool-*` on toimitetussa CLI
:ssä tuettu DAQ-polku**; muut `daq`-alikomennot, joihin saatat nähdä viittauksia, ovat MAPIR
:n sisäisiä, vain lähdekoodina käytettäviä rajapintoja, ja ne päättyvät eksplisiittiseen virheeseen, joka ohjaa sinut `pool-*`:ään.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record` ilman `--duration`-komentoa suoritetaan, kunnes `pool-record --stop`; oletustulostuskansio on `~/Documents/DAQ Live View/` **taustakoneella**. Kapasitanssikorjausprofiili valitaan yhteyden muodostuksen yhteydessä (`--cap-id`, taustapalvelimen oletusarvo `sunshine_cosine`) ja sitä voidaan vaihtaa reaaliaikaisesti `pool-set-cap`:n avulla — cap-profiileja ja anturin kalibroitua mitta-aluetta käsitellään tämän käyttöohjeen DAQ-luvuissa.

{% hint style="warning" %}
**DAQ-E monen verkkokortin isäntäkoneella:** ensimmäinen `pool-connect --eth`-automaattinen tunnistus käynnistyksen jälkeen voi epäonnistua, vaikka anturi olisi kunnossa. `--eth-host <ip-or-hostname>` on luotettava vaihtoehto — käytä sitä aina, kun tunnistus ei tuota tulosta.
{% endhint %}

***

## LATTICE-kamerat, PTP ja projektiautomaatio

`lattice`-perhe (yli 45 alikomentoa) kattaa LATTICE-kameroiden käytön alusta loppuun: tunnistuksen, yksittäiset kuvaukset, pysyvät synkronoidut kuvasarjat GUI:n älykkään valmisteluyhteysprosessin avulla, reaaliaikaisen selaimen esikatselun, kohdistuksen, indeksilaskennan ja isäntäkoneen verkkokortin vianmäärityksen. Esimerkki:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Sen rinnalla: `chloros-cli time-sync` raportoi PTP-grandmasterista, jota Chloros
-isäntäkone käyttää (LATTICE-kamerat ja DAQ-E-anturit toimivat sen alaisina laitteiden välisten aikaleimojen muodostamiseksi), ja `chloros-cli project` avaa tallennetun Chloros
-projektin ja ohjaa sen kameroita, anturiryhmiä ja antureita ilman käyttöliittymää — mukaan lukien skriptatut YAML-tallennusohjeet.

Nämä kolme tuoteperhettä (`lattice`, `project`, `daq pool-*`) ovat myös ainoat, jotka tukevat `CHLOROS_BACKEND_URL`-komentoa **etä**-taustapalvelimen ohjaamiseen; ydinkomennot kohdistuvat aina paikalliseen koneeseen.

Kattavat ohjeet löytyvät tämän käyttöoppaan LATTICE-luvuista; kaikki liput on lueteltu [CLI
-viitteessä](reference/cli-reference.md).

***

## Vianmääritys: 5 yleisintä ongelmaa

| Oire | Korjaus |
| --- | --- |
| `Login required` tai ajoitettu tehtävä jumittuu `Email:`-kehotteeseen | Suorita `chloros-cli login EMAIL 'PASSWORD'` kerran tällä koneella — komennot, joilla ei ole välimuistissa olevaa istuntoa, toimivat vuorovaikutteisesti sen sijaan, että ne epäonnistuisivat välittömästi. |
| `backend unreachable` | Käynnistä Chloros
-työpöytäsovellus tai käynnistä taustapalvelu itse: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` Windows
issa (asennuskansion `backend`-kansiossa; sitä ei ole `PATH`:ssa), `chloros-backend` Linux
issa. `chloros-cli backend status` näyttää, onko jokin käynnissä; `lattice`/`project`/`daq pool-*` eivät koskaan käynnistä sitä. Jos osoitat ne etätaustapalvelimeen, tarkista `CHLOROS_BACKEND_URL`. |
| Array-yhteyden muodostaminen estetty: `FRAMES WILL DROP` / `Reduce ROI to enable` | Isäntäkoneen verkkokortin vastaanottorengas on palautettu oletusasetuksiin — tämä on yleisin syy siihen, että aiemmin toiminut laitteisto kieltäytyy muodostamasta yhteyttä, tyypillisesti verkkokortin ohjaimen päivityksen jälkeen. Suorita komento `chloros-cli lattice network --fix` **korotetuista** oikeuksista käsin (tai aseta `ReceiveBufferLen=256`, `PendingReceives=64`); katso ohjeen kohta *Isäntäkoneen verkkokortin asetukset ja säätö*. |
| `daq`-alikomento keskeytyy: ”vaatii täydellisen DAQ-paketin…” | Käytä komentoja `daq pool-*`, jotka ohjaavat samaa anturia taustapalvelimen kautta: yhteyden muodostus, datavirta, tallennus, kap-valinta sekä DAQ-E:n osalta sen verkkoasetukset, uudelleenkäynnistys ja laiteohjelmistopäivitykset (tai `chloros_sdk.connect_daq_sensor()` Python
ista). |
| Jetson tulostaa swap-varoituksen ennen suurten kansioiden käsittelyä | Lisää tiedostopohjainen swap — CLI
-käyttöliittymä tulostaa tarkat `fallocate`/`swapon`-komennot, jotka on suoritettava. |

***

## Ohjeiden hakeminen

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Kaikki lipput, kaikki alikomennot:** [CLI
-viite](reference/cli-reference.md)
* **Python
-vastaavuus:** [Python
SDK
](api-python-sdk.md) ja [SDK Reference](reference/sdk-reference.md)
* **Tuki:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
