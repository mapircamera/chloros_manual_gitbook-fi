# Chlorosin käyttö tekoälyavustajien kanssa

Tämä opas on tarkoitettu kahdelle kohderyhmälle: ihmisille ja tekoälyavustajille, joiden avulla ihmiset työskentelevät yhä useammin. Jokaisella sivulla on tarkat arvot, oletusasetukset ja kopioitavat komennot, jotta avustaja (Claude, ChatGPT, Copilot, koodausagentti jne.) voi kirjoittaa toimivan Chloros-automaation jo ensimmäisellä yrityksellä.

Chloros-versio: **

1.3.0**. CLI-/SDK-alustat: Windows 10/11 x64 ja Linux (x86_64 / Jetson aarch64).

## Mitä antaa avustajallesi

| Resurssi | URL | Mihin sitä käytetään |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Koneellisesti luettava hakemisto tämän käyttöoppaan jokaisesta sivusta. |
| **Käyttöliittymän (CLI) viite** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Täydellinen `chloros-cli`-komentorakenne: kaikki komennot, lipput, oletusarvot, poistumiskoodit ja tulostuskansiosäännöt. Kirjoitettu LLM:n käyttöön. |
| **SDK-viite** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Täydellinen `chloros_sdk` Python-API: luokat, allekirjoitukset, poikkeukset ja esimerkit. Kirjoitettu LLM:n käyttöön. |
| **Mikä tahansa sivu raakamuotoisena Markdownina** | lisää `.md` sivun URL-osoitteeseen | esim. `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` palauttaa sivun raakamuotoisena Markdownina — ihanteellinen liitettäväksi konteksti-ikkunaan tai haettavaksi agentista. |

Käsikirjan sisäiset linkit: [CLI-viite](reference/cli-reference.md) · [SDK-viite](reference/sdk-reference.md).

{% hint style="info" %}
Nämä kaksi ohjesivua ovat itsenäisiä: avustaja, joka on lukenut jommankumman niistä, ei tarvitse käsikirjan muuta osaa kirjoittaakseen oikean skriptin.
{% endhint %}

## Anna avustajasi suorittaa Chlorosia suoraan: `chloros-mcp`

Kaikissa edellä mainituissa tapauksissa avustajasi *kirjoittaa* skriptin tai komennon. `chloros-mcp`:n avulla avustaja, joka käyttää [Model Context Protocol](https://modelcontextprotocol.io) (MCP) -protokollaa, kuten Claude Desktop, Claude Code tai Cursor, ohjaa itse Chloros-ohjelmaa. Kysyt tavallisella kielellä, ja se kutsuu palvelimen työkaluja:

* **Kuvien käsittely:** luo projekti, tuo kuvakansio, valitse asetukset, käsittele ja raportoi, mitä suoritus vienti.
* **Laitteiston käyttö:** liitä LATTICE-kamerat, anturiryhmät ja DAQ-valosensorit, esikatsele, aseta valotus ja värit, ota kuvia, tallenna ja päivitä laiteohjelmisto.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

Palvelimen työkalut kattavat [Python-SDK:n](api-python-sdk.md) toiminnot. Ne asennetaan SDK:n mukana, joten yleensä mitään muuta ei tarvitse asentaa: Windowsissa Chloros-asennusohjelma sijoittaa `chloros-mcp.exe`:n Pythonin `Scripts`-kansioon, ja Linuxin `.deb` asentaa tiedoston `chloros-mcp` kansioon `/usr/bin`.

**Claude-koodi:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**lisää tämä tiedostoon `claude_desktop_config.json` (**Asetukset > Kehittäjä > Muokkaa asetuksia**avaa sen) ja käynnistä sitten Claude Desktop uudelleen.**Cursor** käyttää samaa merkintää tiedostossa `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

Jos asiakasohjelma ilmoittaa, että palvelinta ei voitu käynnistää, `chloros-mcp` ei todennäköisesti ole siinä `PATH`-kohdassa, jonka se näkee: anna sen sijaan täydellinen polku (`where chloros-mcp` Windowsissa, `which chloros-mcp` Linuxissa).

Chloros on oltava asennettuna samalle tietokoneelle, ja sinun on kirjauduttava sisään siellä, sovelluksessa tai `chloros-cli login`:n kautta millä tahansa maksullisella Chloros+ -tasolla (Copper tai korkeampi). Palvelin ei voi kirjautua sisään puolestasi.

{% hint style="info" %}
Kunkin sovelluksen, asetusten, työkalujen ja suorituksen käyttäytymisen asiakasasetukset: [MCP-palvelin (AI-avustajat)](reference/sdk-reference.md#mcp-server-ai-assistants) SDK-viitteessä.
{% endhint %}

## Komentorivireseptejä

Kopioi, täytä `<placeholders>` ja liitä avustajaasi.

### 1. Muunna lentokansio NDVI-muotoon

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Tarkkaile kuvauskansiota eräajona

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Liitä LATTICE-matriisi ja ota kuvia

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. DAQ-valosensorin spektrien tallentaminen

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
DAQ-komentosarjojen suorittaminen komentoriviltä tapahtuu aina `daq pool-*`-perheen komentojen kautta (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Muut `daq`-alikomennot, joita avustajasi saattaa keksiä, eivät ole käytettävissä toimitetuissa versioissa ja aiheuttavat virheen.
{% endhint %}

## Miksi tekoälyn kirjoittamat skriptit toimivat hyvin Chlorosin kanssa

Jokainen näistä on Chloros 1.3.0:n todellinen, todennettu käyttäytyminen — ne poistavat koneellisesti kirjoitetun automaation klassiset vikamoodit:

* **Ei monimutkaisia asetuksia.**SDK:n älykkäät liitäntäaputoiminnot (`connect_camera`, `connect_array`, `connect_daq_sensor`) ja käsittelyn aloituspisteet (`ChlorosLocal`, `process_folder`)**käynnistävät paikallisen taustapalvelimen automaattisesti**. Luotu skripti ei vaadi avointa käyttöliittymää tai manuaalisesti käynnistettyä palvelinta — se tarvitsee vain työpöytä- tai CLI-paketin asennettuna.
* **Koko prosessiketju suoritetaan yhdellä kutsulla.** `chloros_sdk.process_folder("path", indices=["NDVI"])` suorittaa tuonnin → kalibroinnin → heijastavuuden → indeksin viennin alusta loppuun. Pienempi pinta-ala tarkoittaa vähemmän kohtia, joissa luotu skripti voi mennä pieleen.
* **Tuloksettomat ajot suorittavat itsediagnoosin.** `process()`:n jälkeen ajon yhteenveto liitetään tulokseen, ja jokainen käsittelyvinkki (esim. *miksi* ajosta ei saatu tulosta) lähetetään uudelleen Python-muodossa `UserWarning` — joten jopa skripti, joka ei koskaan tarkista tulosdiktioa, tuo diagnoosin esiin.
* **CLI-käyttöliittymä epäonnistuu selvästi.**`chloros-cli process`-ajo, joka pyysi tuotoksia mutta ei kirjoittanut yhtään, tulostaa `Processing finished but wrote no image products.` ja**päättyy nollasta poikkeavalla koodilla**, joten shell-skriptit ja CI havaitsevat sen tavallisella poistumiskooditarkistuksella. Onnistuneet suoritukset raportoivat virhekoodin `Image products written: N` sekä virhekoodin `(plus M clamp masks)`, kun maskeja on kirjoitettu. Capture-komennot luettelevat vastaavasti kaikki tiedostot, joita ne eivät pystyneet kirjoittamaan, ja lopettavat virhekoodilla `1`.
* **Suhteelliset polut tarkoittavat sitä, mitä skripti odottaa.** Paikallisella taustapalvelimella suhteellinen tulostuspolku (`-o out/` CLI:ssä, `output_dir="out/"` SDK:ssä) määritetään sen kansion suhteen, josta komento tai skripti suoritetaan.

Yksi epäsymmetria, jonka avustajan tulisi tietää: SDK:n `process()` ei tarkoituksella aiheuta poikkeusta nollatuotoksella päättyneessä ajossa — se raportoi asiasta sen sijaan yhteenvedon/vihjeiden kautta. Jos Python-putkisto on keskeytettävä tyhjän ajon yhteydessä, tarkista yhteenveto (kuten resepti 2 tekee).

## Huomautuksia

* **Vaatii Chloros+-kirjautumisen.**CLI ja SDK edellyttävät**maksullista** Chloros+-tasoa, jota valvotaan palvelinpuolella: pyynnöt epäonnistuvat virheellä `401 AUTH_REQUIRED`, jos käyttäjä ei ole kirjautunut sisään, ja virheellä `403 PLAN_UPGRADE_REQUIRED` ilmaisella tasolla. Suorita komento `chloros-cli login` kerran jokaisella koneella ennen luotujen skriptien suorittamista. Katso [Chloros+-kirjautuminen](chloros+-login.md).
* **Capture-komennot ohjaavat todellista laitteistoa.** `lattice` / `daq` / `project`-komennot, SDK-istunto-objektit ja `chloros-mcp`:n työkalut muodostavat yhteyden fyysisiin kameroihin ja antureihin, lähettävät niistä dataa ja laukaisevat niitä. Tarkista luotu skripti ennen sen ensimmäistä suorittamista ja suorita se laitteiston ollessa käytössä.
* **Tarkista tulokset pistokokein.** Varmista tuotekansiot ja muutama pikseliarvo ennen tulosten julkaisemista. Erityisesti heijastavuus-TIFF-tiedostot skaalataan lähteen mukaan — lue `Chloros:PixelScale`-XMP-tunniste (LATTICE-tiedostot sisältävät sen: 32768 = 1,0 heijastavuus; Survey3-tiedostoissa sitä ei ole, ja niissä käytetään arvoa 65535 = 1,0) sen sijaan, että oletettaisiin jakajaa. Molemmat ohjesivut kuvaavat tämän kohdassa ”Reflektiivisyyspikselien lukeminen”.
* **Pieniä sudenkuoppia, jotka saattavat aiheuttaa ongelmia luodussa koodissa:**`pool-record` kirjoittaa**taustapalvelimen** tiedostojärjestelmään (oletusarvoisesti `~/Documents/DAQ Live View/`); koneissa, joissa on useita verkkoliitäntöjä, käytä mieluummin `daq pool-connect --eth-host <ip-or-hostname>`:ää kuin automaattista tunnistusta; käytä `http://127.0.0.1:5000`:ää (ei koskaan `localhost`:ää) kaikkialla, missä taustapalvelimen URL esiintyy; skriptin, joka käy läpi projektin tulospuun, tulisi ohittaa `clamp_masks/`-kansiot (niiden `*_clampmask.tif`-tiedostot ovat maskeja, eivät kuvia — katso [Tulostuskuvien tiedostomuodot](output-image-formats.md#clamp-masks)); ja Windowsissa, jossa pitkien polkujen käyttö on poistettu käytöstä, pidä projektikansiot lyhyinä — projektikansio, jonka nimi on yli noin 130 merkkiä pitkä, hylätään ennen käsittelyn aloittamista.
