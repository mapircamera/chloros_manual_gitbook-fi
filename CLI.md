# CLI : Komentorivi

> **Täydellinen viite:**[CLI-viite](reference/cli-reference.md) dokumentoi**jokaisen alikomennon jokaisen lipun** ja on optimoitu tekoälyavustajille — liitä sen URL avustajaasi ja pyydä toimiva komento: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Vinkki tekoälytyökaluille:** mikä tahansa tämän käyttöoppaan sivu on saatavilla raakamuodossa Markdown-muodossa lisäämällä `.md` sen URL-tunnisteen perään (esim. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), ja `https://mapir.gitbook.io/chloros/llms.txt` indeksoi koko käyttöohjeen LLM-käyttöä varten.

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->
## Mikä on CLI

`chloros-cli` on komentorivipohjainen käyttöliittymä samalle käsittelymoottorille, jota Chloros-työpöytäsovellus käyttää. Se on kevyt HTTP-asiakasohjelma, joka toimii Chloros-taustapalvelimen (paikallinen palvelin `127.0.0.1:5000`:llä) päällä — useimmat komennot käynnistävät taustapalvelimen automaattisesti, joten skripti tarvitsee vain yhden `chloros-cli process …`-kutsun.

Se toimii **Windows 10/11 (x64)**- ja**Linux (x86_64 sekä NVIDIA Jetson arm64 JetPack 6:ssa)**-alustoilla, missä tahansa terminaalissa ilman graafista käyttöliittymää. Tarkista asennus seuraavalla komennolla:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

Komentoryhmät lyhyesti:

* **Käsittely ja tili** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 kieltä — katso [Tuetut kielet](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (vain Linux/Jetson)
* **Käytössä oleva laitteisto** — `lattice` (LATTICE-kameran ohjaus, yli 45 alikomentoa), `daq pool-*` (DAQ-valosensorit), `time-sync` (PTP)
* **Automaatio** — `project` (tallennetun Chloros-projektin ajaminen ilman käyttöliittymää, mukaan lukien YAML-tallennusohjeet)

Hyödyllisiä yleisiä asetuksia: `--port N` (taustapalvelimen portti, oletus `5000`), `-v/--verbose`, `--restart` (pakota taustapalvelimen uudelleenkäynnistys), `--backend-exe PATH`. Katso täydellinen luettelo [CLI-viitteestä](reference/cli-reference.md).

***

## Asennus

CLI **sisältyy Chloros-asennusohjelmaan** kaikilla alustoilla — erillistä CLI-latausta ei ole. Lataa asennusohjelma [Lataa](download.md)-sivulta.

### Windows

Asennusohjelma sijoittaa CLI-tiedoston kansioon:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

ja lisää kyseisen kansion järjestelmän `PATH`-kansioon — **avaa uusi terminaali**asennuksen jälkeen, jotta päivitetty `PATH` tunnistetaan. Asennusohjelma sijoittaa myös käynnistyskomentosarjat (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) asennuksen juurikansioon sekä**Chloros CLI** pikakuvakkeen, joista kukin avaa terminaalin, jossa `chloros-cli` on käyttövalmis.

### Linux

Asenna arkkitehtuurillesi sopiva `.deb`:

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Tämä asentaa `chloros-cli`:n `/usr/bin/chloros-cli`:iin (jo asennettuna versioon `PATH`) ja taustapalvelimen versioon `/usr/lib/chloros/chloros-backend` sekä LATTICE-kameroille tarvittavan Arena SDK -ajoympäristön. Katso lisätietoja kohdasta [Linux:n asennus](linux/linux-installation.md).

### Varmista

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Kirjautuminen ja lisensointi

CLI (ja Python sekä SDK) -palvelun käyttö edellyttää **maksullista Chloros+-pakettia**— se sisältyy kaikkiin maksullisiin tasoihin, mutta ei ilmaiseen tasoon. Raja valvotaan**palvelinpuolella** taustajärjestelmän toimesta, ei CLI-binaaritiedoston toimesta: kirjautumattoman käyttäjän pyyntö hylätään virhekoodilla `401 AUTH_REQUIRED`, ja ilmaistason kirjautuneen käyttäjän pyyntö virhekoodilla `403 PLAN_UPGRADE_REQUIRED`, riippumatta siitä, tuleeko se `chloros-cli`:sta, SDK:stä tai itse kehitetyltä HTTP-asiakasohjelmalta. Päivitä osoitteessa [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Kirjaudu sisään **kerran kutakin konetta kohti**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->
{% hint style="warning" %}
**Salasanat, joissa on erikoismerkkejä**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` sekoittuu komentotulkissa (CLI havaitsee tämän 401-virheen yhteydessä ja yrittää uudelleen automaattisesti, mutta yksinkertaiset lainausmerkit estävät ongelman kokonaan).
{% endhint %}

Istunto tallennetaan välimuistiin tunnukseen `~/.chloros/user_session.json` ja toimii edelleen offline-tilassa tilauksen armonaikana (30 päivää kuukausitilauksissa, vuositilauksissa voimassaolon päättymiseen asti). `chloros-cli status` toimii jopa ilman maksullista tilausta, joten hylkäyksen syy on aina näkyvissä.

{% hint style="danger" %}
**Aikataulutatko headless-tehtäviä? Kirjaudu ensin sisään.**Taustaprosessia käynnistävät komennot (`process`, `status`, `export-status`, …) suoritettuna**ilman välimuistissa olevaa istuntoa**ei epäonnistu nopeasti — se siirtyy interaktiiviseen `Email:` / `Password:`-kehotteeseen stdin-syötteen kautta. Tämän vuoksi automaattinen cron-tehtävä tai CI-vaihe**jää odottamaan syötettä**. Suorita `chloros-cli login EMAIL 'PASSWORD'` kerran koneella ennen kuin ajoitat mitään.
{% endhint %}

***

## Ensimmäinen käsittelykierros

Ohjaa `process` tallennuskansioon — se tunnistaa automaattisesti Survey3 (`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` tai niiden yhdistelmän:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Edistymisvirrat näkyvät reaaliaikaisesti kunkin prosessisäikeen mukaan (tunnistus, analysointi, käsittely, vienti), ja onnistunut suoritus päättyy ilmoitukseen siitä, kuinka monta kuvatuotetta on tallennettu (`Image products written: N`).

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### Tulostusten sijainti

`process` tallentaa tiedostot **projektikansioon**, ei syöttökansioon:

* Ilman `-o`: projekti luodaan oletusprojektikansioosi (joka on yhteinen graafisen käyttöliittymän kanssa; hallitse sitä komennolla `get-project-folder` / `set-project-folder`, varavaihtoehto `~/Chloros Projects`), ja sen nimenä on `-n/--project-name` tai aikaleima (`YYYYMMDD_HHMMSS`), jos nimeä ei ole määritetty.
* Kun käytetään `-o PATH`:ta, kyseinen kansio **on** projektikansio. Jos kansio sisältää jo tiedoston `project.json`, sen sijaan, että se korvattaisiin, luodaan sen sijaan sisarkansio, jonka nimessä on pääte `_1`/`_2`…

Projektin sisällä tuotteet ryhmitellään **kameran mukaan ja sen jälkeen tiedostomuodon mukaan**:

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

LATTICE-kameran kansio on `LATT-<sensor>-<lens>-F<filter>` (vastaa kuvan EXIF-tietojen `Model`) ja `<model>_<filter>` (esim. `Survey3N_RGN`) on Survey3. Muoto-kansion nimet noudattavat mallia `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` tai `tiff32` tiedostolle `TIFF (32-bit, Percent)`.

{% hint style="info" %}
**Jokainen viety tuote säilyttää LÄHDE-tiedoston nimen.**`capture_..._raw.tif`:n radiance-vienti nimetään edelleen `capture_..._raw.tif`:ksi — se vain sijaitsee kansiossa `tiff32/Radiance_Images/`.**Tuotteen tunnistaa kansio, ei tiedostonimi**, joten käytä yleismerkintää hakemistoa varten, älä `*radiance*`-päätettä varten.
{% endhint %}

### Vaihtoehdot, joita todella tulet käyttämään

| Lippu | Oletus | Toiminto |
| --- | --- | --- |
| `-o, --output PATH` | oletusprojektikansio | Projektikansion sijainti (katso yllä). |
| `-n, --project-name NAME` | aikaleima | Projektin nimi. |
| `--format FMT` | `TIFF (16-bit)` | Jokin seuraavista: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | ei mitään | Vientiä varten valitut kasvillisuusindeksit (katso [Kasvillisuusindeksit](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = neuroverkkoinen debayer, hitaampi, korkein laatu (Chloros+, NVIDIA GPU). |
| `--vignette / --no-vignette` | päällä | Vignettikorjaus. |
| `--reflectance / --no-reflectance` | päällä | Heijastavuuden kalibrointi; LATTICE-tiedostojen kohdalla tämä toimii myös heijastavuustuotteen kytkimenä. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Pakota prosessiketjun aloituskohta LATTICE-TIFF-tiedostoille. |

Kaikki muut asetukset — kohteen tunnistuksen säätö, PPK, valotuspisteet, matriisin kohdistusliput — katso [`process`-osio CLI-viitteestä](reference/cli-reference.md).

***

## Vientikohteiden valinta (LATTICE-tuotteet)

LATTICE-käsittely haarautuu **kaikkiin soveltuviin tuotteisiin yhdellä kertaa**. Neljä tuotekohtaista kytkintä ovat kaikki**oletusarvoisesti päällä**; käytä lomaketta `--no-`, jos haluat poistaa yhden:

| Kytkin | Tuote |
| --- | --- |
| `--debayered` | Lineaarinen demosaikki → `Debayered_Images/` |
| `--preview` | Esikatselun näyttö (valkotasapaino + gamma; vääräväriväriavaruus monispektrikuville) → `Preview_Images/` |
| `--radiance` | float32-säteilyvoimakkuus, W/m²/sr/nm → `Radiance_Images/` (aina `tiff32/`) |
| `--reflectance` | uint16 heijastavuus, Pix4D-yhteensopiva → `Reflectance_Calibrated_Images/` |

RGB-pääkamerat lähettävät aina vain debayeroitua + esikatselua — kaistakohtainen säteilyvoimakkuus/heijastavuus ei ole merkityksellistä laajakaistaiselle anturille, joten nämä kytkimet eivät vaikuta niihin. Survey3 `.raw` ohittaa kytkimet ja noudattaa standardia heijastavuus-/kohdepolkua.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (oletusarvo `auto`) valitsee heijastavuusvertailukohteen: `auto` luo laadunvarmistuksen läpäisseen [kalibrointikohteen](calibration-targets.md) absoluuttiseksi vertailukohteeksi ja siirtyy takaisin DAQ-valosensorin alaspäin suuntautuvan säteilyn jakosuhteeseen (ρ = π·L/E), kun kohdetta ei ole läsnä; `target` on tiukka (ei DAQ-korvausta); `daq` käyttää DAQ:ta pääasiallisena lähteenä. Yksikkökohtaiset mitatut kohdekuvaukset voidaan toimittaa `--target-reflectance-dir`:n avulla.

{% hint style="info" %}
**Heijastuspikselien lukeminen:**DN, joka tarkoittaa ρ = 1,0, on**lähdekohtainen** — LATTICE-tiedostot merkitsevät XMP:hen tunnisteen `Chloros:PixelScale=32768`; Survey3-tiedostot käyttävät arvoa 65535 (eivätkä sisällä `Chloros:*`-tunnisteita). Lue tunniste ja jaa arvo sen mukaan sen sijaan, että olettaisit arvon olevan vakio. Yksityiskohdat ja yksi tarkoituksellinen mittakaavaton reunatapaus löytyvät [CLI-viitteestä](reference/cli-reference.md).
{% endhint %}

**Käsittely alkaa aina tiedostosta `raw`.** Johdannais tuotteet (debayered/radiance/reflectance-vienti) eivät koskaan palaa prosessiputkeen – niiden uudelleen tuominen ja käsittely johtaisi kalibrointilaskennan kaksinkertaiseen soveltamiseen, joten Chloros ohittaa ne ja ilmoittaa asiasta. `--input-level` on tarkoituksellisesti lisätty pakotie tilanteisiin, joissa lähtöpistettä on aidosti pakko pakottaa.***

## Kun suoritus epäonnistuu

Versiosta 1.2.0 lähtien `process` ilmoittaa selvästi epäonnistumisesta sen sijaan, että se ”onnistuisi” ilman näkyviä tuloksia:

* Suoritus, joka **pyysi tuotteita mutta ei kirjoittanut yhtään**— vain `project.json` ja `calibration_data.json` — tulostaa virheen `Processing finished but wrote no image products.` ja**päättyy nollasta poikkeavalla arvolla**, joten skriptit voivat havaita sen. Yleisimmät syyt: syöttökansiota ei tunnistettu kuvauskohteeksi (tarkista asettelu ja `--input-level`) tai jokainen pyydetty tuote oli soveltumaton kyseisille kameroille (esim. pyydetään säteilyvoimakkuutta/heijastavuutta vain RGB-kameroista).
* **Tarkoituksellinen pelkästään metatietoja tuottava ajo** (kaikki tuotteet pois päältä, ei `--indices`) on silti onnistunut — tyhjä kuvatulos on siinä tapauksessa oikea tulos.
* Suorita käsittely uudelleen `--verbose`:llä ja tarkista taustaprosessin lokista `[LATTICE-EXPORT]`- ja `[EXPORT-CHECK]`-rivit, jotka selittävät kamerakohtaiset ohitukset.

Lopetuskoodit: `0` onnistui · `1` yleinen virhe · `2` argumenttivirhe · `130` keskeytettiin Ctrl+C:llä.

***

## Kasvillisuusindeksit

Suorita `--indices` yhdellä tai useammalla esiasetetulla nimellä; kukin indeksi tallentuu omaan `<INDEX>_Index_Images/`-kansioonsa:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

22 esiasetettua nimeä, jotka `process --indices` hyväksyy:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**Indeksiluetteloita on kolme — älä sekoita niitä keskenään.**GUI:n Projektin asetukset -pudotusvalikossa on 27 kaavaa (lisää `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — nämä viisi ovat käytettävissä vain käyttöliittymässä eivätkä ne**kelpaa** `--indices`:lle). Live-/offline-komento `lattice index --preset` käyttää omaa erillistä 22 esiasetuksen luetteloaan. Kaavat ja kaistamatematiikka on dokumentoitu kohdassa [Monispektriset indeksikaavat](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## DAQ-valosensorit: Pikaesittely

`daq pool-*`-tuoteperhe ohjaa MAPIR DAQ-spektrisensoreita (DAQ-U USB:n kautta, DAQ-M BLE:n kautta, DAQ-E Ethernetin kautta) taustapalvelimen pysyvän poolin kautta — käyttöliittymä, CLI ja SDK jakavat kaikki yhden reaaliaikaisen käsittelykäsitteen. **`pool-*` on tuettu DAQ-polku toimitetussa CLI-ohjelmistossa**; muut `daq`-alikäskyt, joihin saatat nähdä viittauksia, ovat MAPIR:n sisäisiä, vain lähdekoodina käytettäviä pintoja, ja ne päättyvät eksplisiittiseen virheeseen, joka ohjaa sinut `pool-*`:ään.

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

`pool-record` ilman `--duration`-komentoa toimii `pool-record --stop`-komentoon asti; oletustulostuskansio on `~/Documents/DAQ Live View/` **taustakoneella**. Kapasitanssikorjausprofiili valitaan yhteyden muodostuksen yhteydessä (`--cap-id`, taustapalvelimen oletusarvo `sunshine_cosine`) ja se voidaan vaihtaa reaaliaikaisesti `pool-set-cap`:n avulla — kap-profiileja ja anturin kalibroitua mitta-aluetta käsitellään tämän käyttöohjeen DAQ-luvuissa.

{% hint style="warning" %}
**DAQ-E monen verkkokortin isäntäkoneella:** ensimmäinen `pool-connect --eth`-automaattinen tunnistus käynnistyksen jälkeen voi epäonnistua, vaikka anturi olisi kunnossa. `--eth-host <ip-or-hostname>` on luotettava vaihtoehto — käytä sitä aina, kun tunnistus ei tuota tulosta.
{% endhint %}

***

## LATTICE-kamerat, PTP ja projektiautomaatio

`lattice`-tuoteperhe (yli 45 alikomentoa) kattaa LATTICE-kameroiden käytön alusta loppuun: tunnistuksen, yksittäiset kuvaukset, pysyvät synkronoidut kuvasarjat GUI:n älykkään valmisteluyhteysprosessin avulla, reaaliaikaisen selainesikatselun, kohdistuksen, indeksilaskelmat sekä isäntäkoneen verkkokortin vianmäärityksen. Esimerkki:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Sen rinnalla: `chloros-cli time-sync` raportoi PTP-grandmasterista, jota Chloros-isäntä käyttää (LATTICE-kamerat ja DAQ-E-anturit toimivat sen orjina laitteiden välisten aikaleimojen muodostamiseksi), ja `chloros-cli project` avaa tallennetun Chloros-projektin ja ohjaa sen kameroita, matriiseja ja antureita ilman käyttöliittymää — mukaan lukien skriptatut YAML-tallennusohjeet.

Nämä kolme tuoteperhettä (`lattice`, `project`, `daq pool-*`) ovat myös ainoat, jotka tukevat `CHLOROS_BACKEND_URL`:ää **etä**-taustapalvelimen ohjaamiseen; ydinkomennot kohdistuvat aina paikalliseen koneeseen.

Kattavat ohjeet löytyvät tämän käsikirjan LATTICE-luvuista; kaikki liput on lueteltu [CLI-viitteessä](reference/cli-reference.md).

***

## Vianmääritys: 5 yleisintä ongelmaa

| Oire | Korjaus |
| --- | --- |
| `Login required` tai ajoitettu tehtävä jumittuu `Email:`-kehotteeseen | Suorita `chloros-cli login EMAIL 'PASSWORD'` kerran tällä koneella — komennot, joilla ei ole välimuistissa olevaa istuntoa, toimivat vuorovaikutteisesti sen sijaan, että ne epäonnistuisivat välittömästi. |
| `backend unreachable` | Käynnistä Chloros-työpöytäsovellus tai suorita taustapalvelinohjelma suoraan (`chloros-backend`). Jos osoitat `lattice`/`project`/`daq pool-*` etätaustapalvelimeen, tarkista `CHLOROS_BACKEND_URL`. |
| Array-yhteyden muodostaminen estetty: `FRAMES WILL DROP` / `Reduce ROI to enable` | Isäntäkoneen verkkokortin vastaanottorengas on palautettu oletusasetuksiin — tämä on yleisin syy siihen, että aiemmin toimiva laitteisto kieltäytyy muodostamasta yhteyttä, tyypillisesti verkkokortin ohjaimen päivityksen jälkeen. Suorita `chloros-cli lattice network --fix` **korotetuista** oikeuksista toimivasta terminaalista (tai aseta `ReceiveBufferLen=256`, `PendingReceives=64`); katso viitteen kohta *Isäntäkoneen verkkokortin asetukset ja hienosäätö*. |
| `daq`-alikomento keskeytyy: ”vaatii täydellisen DAQ-paketin…” | Odotettavissa toimitettavissa versioissa — käännetty CLI sisältää vain `daq pool-*`-perheen, joka kattaa yhteyden muodostuksen, datavirran, tallennuksen ja kap-valinnan. Käytä `pool-*`:ää (tai `chloros_sdk.connect_daq_sensor()`:ää versiosta Python). |
| Jetson tulostaa swap-varoituksen ennen suuria kansioita | Lisää tiedostopohjainen swap — CLI tulostaa tarkat `fallocate`/`swapon`-komennot, jotka on suoritettava. |

***

## Ohjeiden hakeminen

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Jokainen lippu, jokainen alikomento:** [CLI-viite](reference/cli-reference.md)
* **Python-vastaava:** [Python SDK](api-python-sdk.md) ja [SDK-viite](reference/sdk-reference.md)
* **Tuki:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
