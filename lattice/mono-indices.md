# Yksivärikamerat ja kasvillisuusindeksit

## Yksi kamera = yksi kaista

**M3M**-kamera on Bayer**M3C**:n yksivärinen vastine: siinä on yksi kapeakaistainen häiriösuodatin, jonka takana on IMX265-monokromaattinen kenno. Mallinimessä mainitaan kaista — `M3M-<lens>-F<wavelength>`, esim. `M3M-L87-F685` (näkyy mallissa Chloros nimellä `LATT-M3M-L87-F685`). Anturi tuottaa**yhden harmaasävykaistan** ilman Bayer-mosaiikkia: demosaikointia ei tarvita, kanavien välistä ylikuulumista ei tarvitse erottaa eikä valkotasapainoa tarvitse säätää.

Seuraavat seikat on syytä tietää ennen monokromaattisen järjestelmän suunnittelua:

* **Säteilyvoimakkuus ja heijastavuus on määritelty täysin kaistakohtaisesti.**Ne ovat kaistakohtaisia radiometrisiä karttoja, joten yksi M3M-kamera tuottaa kalibroitua float32-säteilyä (W/m²/sr/nm) ja uint16-heijastavuutta (`32768` = ρ 1,0) täsmälleen samalla tavalla kuin M3C-kaista. Mono-kuvissa on**identiteetti**-anturivaste-matriisi — 3×3-sekoituksen purkamista ei tarvita eikä sitä sovelleta.
* **Yksi mono-kamera ei voi tuottaa kasvillisuusindeksiä.** NDVI, NDRE ja vastaavat tarvitsevat vähintään kaksi kaistaa. Indeksien laskemiseksi mono-laitteistolla yhdistetään useita M3M-kameroita — katso alla.
* M3M-kamerat lähettävät **Mono12**-dataa (12-bittistä, 2 tavua/pikseli siirrossa), mikä on merkittävää [matriisin kaistanleveyden budjetoinnin](arrays.md#bandwidth-the-rules-of-thumb) kannalta.

## Mitä Chloros ohittaa monokameran kohdalla — ja miten se ilmoittaa siitä

Väriprosessointivaiheet eivät yksinkertaisesti koske yksikaistaista anturia. Chloros **ohittaa ne yhden rivin viestillä** virheilmoituksen sijaan, mutta suorittaa ne silti normaalisti kaikille samassa istunnossa käytetyille M3C (Bayer)-kameroille:

| Vaihe | Mono (M3M) -käyttäytyminen | M3C-käyttäytyminen |
| --- | --- | --- |
| Demosaic / debayer | Ohitettu — `debayered`-vientitaso on 1-kanavainen harmaasävykuva. | 3-kanavainen demosaic. |
| Valkotasapaino (`lattice white-balance`) | Ohitetaan yhden rivin viestillä. | Suoritetaan normaalisti. |
| Väriprofiili (`lattice color-profile`) | Ohitetaan yhden rivin viestillä. | Suoritetaan normaalisti. |
| Värikylläisyys/kontrasti (`lattice color`) | Ohitetaan yhden rivin viestillä. | Suoritetaan normaalisti. |
| Spektrinen ylikuulumisen erottelu | Identiteetti (ei 3×3-matriisia). | Kameraa kohti 3×3-matriisi sovellettu. |
| Säteilyvoimakkuus / heijastavuus | **Suoritetaan** — kaistakohtaisesti, täysin kalibroitu. | Suoritetaan kaistakohtaisesti. |

Käyttöliittymä soveltaa samaa suodatusta: yksikanavaisen kameran kohdalla kamerakohtaisessa asetuspaneelissa piilotetaan vain RGB-rivit (valkotasapaino, gamma, väriprofiili, kylläisyys, kontrasti, kanavajaot), ja reaaliaikainen histogrammi on lukittu yhteen **MONO**-käyrään. Koko pinoamisprosessin erottelukriteerinä toimii mallimerkkijonossa oleva `M3M`-tunnus, joka näkyy käyttöliittymässä/SDK nimellä `is_mono`.

## Indeksit vaativat vähintään 2 kaistaa: kohdistus → pinoaminen → indeksointi

Mono-indeksoinnin työnkulku koostuu aina samoista kolmesta vaiheesta:

1. **Kohdistus** — suunnataan useita M3M-kameroita eri aallonpituuksille (esim. F650 ”Red” ja F850 &quot;NIR&quot;), yhdistämällä ne [monikamerajärjestelmäksi](arrays.md) ja antamalla Chloros:n laskea kameroiden välisen ko-rekisteröinnin vääristymän.
2. **Pino** — kohdistetut kehykset yhdistyvät yhdeksi monikaistaiseksi kuvaksi (jokainen kamera tuottaa yhden nimetyn kaistan).
3. **Indeksi** — laske indeksikaava pinon kaistojen perusteella, tarvittaessa renderoimalla se LUT:n kautta.

Käyttöliittymässä tämä koko ketju on **Yhdistetyt kamerat**-ryhmän näyttötila: reaaliaikainen yhdistelmäkuva on jo kohdistettu, ja ryhmän indeksilaskuri (alla) määrittää sen renderointikaavan. Tallennetut vientiä voidaan vääristää samaan kohdistukseen**Kohdistettu**-tallennusvaihtoehdolla.

## Indeksilaskuri

Indeksilaskuri luo indeksilausekkeen, jota reaaliaikainen näkymä ja kamerakohtaiset indeksivienti-tiedostot käyttävät. Se on yksi jaettu pinta, joka avautuu kahdesta paikasta Kamerat-välilehden sivupalkissa:

* **Kamera-kohtainen**— Live-esikatselu →**Indeksi**-hammasratas (vain RGN/OCN/NGB-Bayer-kamerat; yksittäisellä monokameralla ei ole indeksisäätöä, koska yhdellä kaistalla ei voi muodostaa indeksiä).
* **Järjestelmäkohtainen**— järjestelmän asetukset → Live-esikatselu →**Indeksi**-hammasratas. Tämä on monokamerareitti: kaistaluettelo kattaa**kaikki järjestelmään kuuluvat kamerat**, joten monokamerapari tuo tähän omat kaksi kaistaansa.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Sen säätimet ylhäältä alas:

* **Kaistapainikkeet** (&quot;Kaistat — napsauta lisätäksesi lausekkeeseen&quot;) — yksi painike kutakin käytettävissä olevaa kaistaa kohti, merkitty värinimellä + aallonpituudella nm:inä (samannimiset värit erotetaan toisistaan, esim. &quot;Väri 850&quot;). Napsauttamalla lisätään kaistatunnus kohdistimen kohdalle. Kaistat kameroista, jotka eivät pysty tuottamaan kaistakohtaista säteilyvoimakkuutta (RGB/FRGB), suodatetaan pois.
* **Operaattori- ja funktiopainikkeet** — `+ - * / ( ) ^ ,` sekä `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Lausekkeen tekstikenttä** — vapaasti kirjoitettava kaava; paikkamerkki näyttää klassisen NDVI-muodon `(NIR - Red) / (NIR + Red)`. Sen yläpuolella oleva vain luku -tilaan asetettu tokenoitu esikatselu renderöi kaistalohkot, numerot ja liput tuntemattomina tunnisteina.
* **Voimassaolobanneri**— harmaa ”Tyhjä — indeksiä ei sovelleta”; vihreä &quot;Kelvollinen lauseke&quot;; punainen, jossa on tarkka jäsennysvirhe (tuntematon kaista, useiden kameroiden paljastama epäselvä kaista, puuttuva sulu, …); tai keltainen, kun lauseke on kelvollinen mutta**vakio** (esim. `X/X`, tai nimittäjä NDVI, joka on kirjoitettu muodossa `−` sen sijaan, että se olisi `+`) — vakio muuttaa koko kehyksen yhdeksi väriksi.
* Erillinen keltainen varoitus ilmestyy, jos käytetty lauseke on kunnossa, mutta **reaaliaikainen kehys on yhtenäinen** (tasainen tai kylläinen kohtaus) — histogrammin romahtaminen havaitaan puolestasi.
* **Käytä LUT:ta**(oletusarvoisesti päällä; pois päältä = harmaasävyn venytys),**Taso**2/3/5/7-stop (oletusarvo 7-stop) ja**Min / Max**-syötteet gradienttipalkin molemmin puolin. Min-arvon oletusarvo on**0,2**— se zoomaa väriskaalan kasvillisuuden kannalta merkitykselliseen alueeseen, kun taas sitä pienemmät arvot näkyvät harmaasävyinä; aseta Min-arvoksi −1, jos haluat koko indeksi-alueen (**Reset**-painike palauttaa arvon −1…+1). Max-arvon oletusarvo on 1.
* **Reaaliaikainen histogrammi** indeksin jakaumasta — neliöjuuriskalattuja palkkeja, keltaiset p2/p98-persentiiliviivat, valkoinen mediaaniviiva sekä alueen ulkopuolella olevat lukemat (&quot;◀ N% &lt; lo&quot; / &quot;hi &lt; N% ▶&quot;), jotka muuttuvat keltaisiksi yli 1 %:n ollessa merkkinä Min/Max-ikkunan laajentamisesta.
* **Käytä**-painike vahvistaa lausekkeen reaaliaikaiseen virtaan; LUT-säädöt tulevat voimaan reaaliaikaisesti ilman, että Käytä-painiketta painetaan. Lausekkeet ovat tarkoituksellisesti**vain istuntokohtaisia** — niitä ei tallenneta istuntojen välillä.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->

## CLI-polku

Sama align → stack → index -ketju, skriptattavissa päästä päähän:

```bash
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` yhdistää esiasetuksen symbolit pinon kaistanimiin. Kaksi sääntöä säästää sinut epäonnistuneelta ajolta:

* **Symboleissa otetaan huomioon kirjainkoko**, ja niiden on vastattava täsmälleen esiasetuksen kanavanimiä — esiasetuksissa käytetään pieniä kirjaimia (NDVI:n vastineet ovat `red`,`nir`; tarkista `--list-presets`). `--channel red=Red_660` toimii; `--channel RED=660` epäonnistuu ja antaa virheen `channel_map missing entries`.
* Kaistapuolella on nimettävä kaista kohdistetussa pinossa (`lattice align-info --profile align.json` luettelee ne). Offline-tila hyväksyy myös 0-pohjaiset kaistaindeksit, esim. `--channel red=0 --channel nir=1`.

`lattice index` toimii myös täysin offline-tilassa tallennetun kohdistetun monikaistaisen TIFF:n kanssa:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Indeksi-esiasetukset

`lattice index --preset` (ja Image-välilehden [Index/LUT sandbox](../image-viewer-gui/index-lut-sandbox.md), joka käyttää samaa moottoria) sisältää nämä **22 esiasetusta**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Suorita `chloros-cli lattice index --list-presets` kunkin esiasetuksen kaavan ja kanavasymbolien tarkastelemiseksi ja `--list-gradients` käytettävissä olevien värigradienttien tarkastelemiseksi. Mukautetuissa kaavoissa käytetään `--formula EXPR`:ää samalla syntaksilla kuin indeksilaskimessa. Huomaa, että tämä esiasetuslista koskee nimenomaan LATTICE-indeksimoottoria — tuoduille kuville tarkoitettu Projektiasetukset-valikko on eri lista (katso [Monispektriset indeksikaavat](../project-settings/multispectral-index-formulas.md)).

Koko lippujoukko (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, `--live`:n kohdistuksen vääristymän säätimet, ja muita) on dokumentoitu kohdassa [CLI-viite § Indeksi / Kasvillisuusmatematiikka](../reference/cli-reference.md#index--vegetation-maths); SDK:n vastaavat löytyvät kohdasta [SDK-viite](../reference/sdk-reference.md).

## Indeksituotteiden tallentaminen mono-taulukosta

Kun taulukko on liitetty ja indeksilauseke on sovellettu, `array-capture` (tai käyttöliittymän **Capture All**) tallentaa kamerakohtaiset vientitasot *ja* indeksin renderoinnin — `--index`/`--no-index` kytkee tämän toiminnon päälle CLI:ssä, ja tallennus sisältää oletuksena kaikki soveltuvat tasot. Monokameran osuus kussakin tallennusryhmässä on sen yksi kaista raaka-/debayeroitu (harmaasävy)/säteily-/heijastustasoilla sekä jaettu yhdistetty indeksikomposiitti, kun ryhmä toimii yhdistetyllä tilalla. Katso [Monikamerajärjestelmät § Tallennus](arrays.md#capturing-monitoring-vs-analysis).
