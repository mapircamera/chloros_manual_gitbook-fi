# Chloros:n käyttö tekoälyavustajien kanssa

Tämä käyttöopas on tarkoitettu kahdelle kohderyhmälle: ihmisille sekä tekoälyavustajille, joiden avulla ihmiset työskentelevät yhä useammin. Jokaisella sivulla on tarkat arvot, oletusasetukset ja kopioitavat komennot, jotta avustaja (Claude, ChatGPT, Copilot, koodausagentti jne.) voi kirjoittaa toimivan Chloros-automaation jo ensimmäisellä yrityksellä.

Chloros-versio: **

1.2.0**. CLI/SDK-alustat: Windows 10/11 x64 ja Linux (x86_64 / Jetson aarch64).

## Mitä antaa avustajallesi

| Resurssi | URL | Mihin sitä käytetään |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Koneellisesti luettava hakemisto tämän käyttöohjeen jokaisesta sivusta. |
| **CLI-viite** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Täydellinen `chloros-cli`-komentopinta: kaikki komennot, liput, oletusarvot, poistumiskoodit ja tulostuskansiota koskevat säännöt. Kirjoitettu LLM-käyttöön. |
| **SDK-viite** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Täydellinen `chloros_sdk` Python API: luokat, allekirjoitukset, poikkeukset ja esimerkit. Kirjoitettu LLM-käyttöön. |
| **Mikä tahansa sivu raakamuodossa Markdownina** | lisää `.md` sivulle URL | esim. `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` palauttaa sivun raakamuotoisena Markdown-muodossa — ihanteellinen liitettäväksi konteksti-ikkunaan tai haettavaksi agentista. |

Käsikirjan sisäiset linkit: [CLI Viite](reference/cli-reference.md) · [SDK-viite](reference/sdk-reference.md).

{% hint style="info" %}
Nämä kaksi viitesivua ovat itsenäisiä: avustaja, joka on lukenut jommankumman niistä, ei tarvitse käyttöohjeen muuta osaa kirjoittaakseen oikean skriptin.
{% endhint %}

## Valmiita komentoja

Kopioi, täytä `<placeholders>` ja liitä avustajaasi.

### 1. Käsittele lentokansio NDVI:ksi

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Tarkkaile kaappaushakemistoa eränä

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Liitä LATTICE-matriisi ja kaappaa

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Tallenna DAQ-valosensorin spektrit

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
DAQ-komentosarjojen suorittaminen komentoriviltä tapahtuu aina `daq pool-*`-perheen kautta (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Muut `daq`-alikomennot, joita avustajasi saattaa keksiä, eivät ole käytettävissä toimitetuissa versioissa ja aiheuttavat virheen.
{% endhint %}

## Miksi tekoälyn kirjoittamat skriptit toimivat hyvin Chloros:n kanssa

Jokainen näistä on todellinen, todennettu käyttäytyminen Chloros-versiossa 1.2.0 — ne poistavat koneellisesti kirjoitetun automaation klassiset vikamoodit:

* **Ei monimutkaista asetusten säätämistä.**SDK:n älykkäät liitäntäapuohjelmat (`connect_camera`, `connect_array`, `connect_daq_sensor`) ja käsittelyn aloituskohdat (`ChlorosLocal`, `process_folder`)**käynnistävät paikallisen taustapalvelimen automaattisesti**. Luotu skripti ei vaadi graafisen käyttöliittymän avaamista tai manuaalisesti käynnistettyä palvelinta — se tarvitsee vain asennetun desktop/CLI-paketin.
* **Koko prosessiketju suoritetaan yhdellä komennolla.** `chloros_sdk.process_folder("path", indices=["NDVI"])` suorittaa tuonnin → kalibroinnin → heijastavuuden → indeksin viennin alusta loppuun. Pienempi pinta-ala, vähemmän kohtia, joissa luotu skripti voi mennä pieleen.
* **Tuloksettomat ajot suorittavat itsediagnoosin.** `process()`:n jälkeen ajon yhteenveto liitetään tulokseen, ja jokainen käsittelyvinkki (esim. *miksi* ajosta ei saatu tulosta) lähetetään uudelleen Python- ja `UserWarning`-komentoina — joten jopa skripti, joka ei koskaan tarkista tulosdiktionaria, tuo diagnoosin esiin.
* **CLI epäonnistuu näkyvästi.**`chloros-cli process`-ajo, joka pyysi tuotoksia mutta ei kirjoittanut yhtään, tulostaa `Processing finished but wrote no image products.` ja**päättyy nollasta poikkeavalla koodilla**, joten komentosarjat ja jatkuva integraatio (CI) havaitsevat sen tavallisella poistumiskoodin tarkistuksella. Onnistuneet suoritukset raportoivat `Image products written: N`.

Yksi epäsymmetria, jonka avustajan tulisi tietää: SDK:n `process()` ei tarkoituksella aiheuta poikkeusta nollatuotoksisessa suorituksessa — se raportoi sen sijaan yhteenvedon/vihjeiden kautta. Jos Python-putkisto joutuu pysähtymään tyhjän ajon yhteydessä, tarkista yhteenveto (resepti 2 tekee näin).

## Huomautuksia

* **Chloros+ -kirjautuminen vaaditaan.**CLI ja SDK edellyttävät**maksullista** Chloros+-tason, joka valvotaan palvelinpuolella: pyynnöt epäonnistuvat koodilla `401 AUTH_REQUIRED`, jos et ole kirjautunut sisään, ja koodilla `403 PLAN_UPGRADE_REQUIRED` ilmaisella tasolla. Suorita `chloros-cli login` kerran kutakin konetta kohti ennen luotujen skriptien suorittamista. Katso [Chloros+ Kirjautuminen](chloros+-login.md).
* **Capture-komennot ohjaavat todellista laitteistoa.** `lattice`-, `daq`- ja `project`-komennot sekä SDK-istunto-objektit muodostavat yhteyden fyysisiin kameroihin ja antureihin, lähettävät niiltä dataa ja laukaisevat niitä. Tarkista luotu skripti ennen sen ensimmäistä suorittamista ja suorita se laitteiston ollessa paikalla.
* **Tarkista tulokset pistokokein.** Varmista tuotekansiot ja muutamat pikseliarvot ennen tulosten julkaisemista. Erityisesti heijastavuustiedostot (TIFF) skaalataan lähteittäin — lue `Chloros:PixelScale`-XMP-tunniste (LATTICE: 32768 = heijastavuus 1,0; Survey3: 65535) sen sijaan, että olettaisit jakajan. Molemmissa ohjesivuissa tämä on dokumentoitu kohdassa ”Reflectance-pikselien lukeminen”.
* **Pieniä sudenkuoppia, jotka saattavat aiheuttaa ongelmia generoidussa koodissa:**`pool-record` kirjoittaa**taustapalvelimen** tiedostojärjestelmään (oletusarvoisesti `~/Documents/DAQ Live View/`); koneissa, joissa on useita verkkoliitäntöjä, käytä mieluummin `daq pool-connect --eth-host <ip-or-hostname>`:ää kuin automaattista tunnistusta; ja käytä `http://127.0.0.1:5000`:ää (ei koskaan `localhost`:ää) kaikkialla, missä taustapalvelimen URL esiintyy.
