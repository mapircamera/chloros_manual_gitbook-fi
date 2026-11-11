# Yksivärikamerat ja kasvillisuusindeksit

## Yksi kamera = yksi kaista

**M3M**-kamera on Bayer**M3C**:n yksivärinen vastine: siinä on yksi kapeakaistainen häiriösuodatin, jonka takana on yksivärinen IMX265-kenno. Mallinimessä mainitaan kaista — `M3M-<lens>-F<wavelength>`, esim. `M3M-L87-F685` (Chloros

-ohjelmassa näkyy muodossa `LATT-M3M-L87-F685`). Kuva-anturi tuottaa **yhden harmaasävykaistan** ilman Bayer-mosaiikkia: demosaikointia ei tarvita, kanavien välistä ylikuulumista ei tarvitse erottaa eikä valkotasapainoa tarvitse säätää.

Seuraavat seikat on syytä tietää ennen monokromaattisen järjestelmän suunnittelua:

* **Säteilyvoimakkuus ja heijastavuus on määritelty täysin kaistakohtaisesti.**Ne ovat kaistakohtaisia radiometrisiä karttoja, joten yksi M3M-kamera tuottaa kalibroitua float32-säteilyä (W/m²/sr/nm) ja uint16-heijastavuutta (`32768` = ρ 1,0) täsmälleen samalla tavalla kuin M3C-kaista. Mono-kuvissa on**identiteetti**-anturivaste-matriisi — 3×3-sekoituksen purkamista ei tarvita eikä sitä sovelleta.
* **Yksi mono-kamera ei voi tuottaa kasvillisuusindeksiä.** NDVI

, NDRE

ja vastaavat indeksit vaativat vähintään kaksi kaistaa. Indeksien laskemiseksi mono-laitteistolla yhdistät useita M3M-kameroita — katso alla.
* M3M-kamerat lähettävät **Mono12**-dataa (12-bittinen: 2 tavua/pikseli siirrossa tai 1,5 tavua/pikseli, kun matriisi käyttää [pakattua 12-bittistä siirtoa](arrays.md#packed-12-bit-transport)), mikä on merkittävää [matriisin kaistanleveyden budjetoinnissa](arrays.md#bandwidth-how-many-cameras-fit).

## Mitä Chloros

ohittaa monokromisessa tilassa — ja miten se ilmoittaa siitä

Väriprosessointiputken vaiheet eivät yksinkertaisesti koske yksikaistaista anturia. Chloros

**ohittaa ne yhden rivin viestillä** virheilmoituksen sijaan, mutta suorittaa ne silti normaalisti kaikille M3C (Bayer)-kameroille samassa istunnossa:

| Vaihe | Mono (M3M) -käyttäytyminen | M3C-käyttäytyminen |
| --- | --- | --- |
| Demosaic / debayer | Ohitetaan — `debayered`-vientitaso on 1-kanavainen harmaasävykuva. | 3-kanavainen demosaic. |
| Valkotasapaino (`lattice white-balance`) | Ohitetaan yhden rivin viestillä. | Suoritetaan normaalisti. |
| Väriprofiili (`lattice color-profile`) | Ohitetaan yhden rivin viestillä. | Suoritetaan normaalisti. |
| Värikylläisyys/kontrasti (`lattice color`) | Ohitetaan yhden rivin viestillä. | Sujuu normaalisti. |
| Spektrinen ylikuulumisen erottelu | Identiteetti (ei 3×3-matriisia). | Kameraa kohti 3×3-matriisi sovellettu. |
| Säteilyvoimakkuus / heijastavuus | **Toimii** — kaistakohtaisesti, täysin kalibroitu. | Toimii kaistakohtaisesti. |

GUI soveltaa samaa rajausta: monokameran kohdalla kamerakohtaisessa asetuspaneelissa piilotetaan pelkästään RGB

-rivit (valkotasapaino, gamma, väriprofiili, kylläisyys, kontrasti, kanavajaot), ja reaaliaikainen histogrammi on lukittu yhteen **MONO**-käyrään. Koko pinoamisen erottelijana toimii mallimerkkijonossa oleva `M3M`-tunnus, joka näkyy käyttöliittymässä ja SDK

:ssa muodossa `is_mono`.

## Indeksit vaativat vähintään 2 kaistaa: kohdistus → pinoaminen → indeksointi

Mono-indeksin työnkulku koostuu aina samoista kolmesta vaiheesta:

1. **Kohdistus** — suunnataan useita M3M-kameroita eri aallonpituuksille (esim. F650 ”Red

” ja F850 ”NIR

”), liitetään ne [monikamerajärjestelmäksi](arrays.md) ja anna Chloros

in laskea kameroiden välinen ko-rekisteröintivääristymä.
2. **Pinoaminen** — kohdistetut kehykset muodostavat yhden monikaistakuvan (jokainen kamera tuottaa yhden nimetyn kaistan).
3. **Indeksi** — laske indeksikaava pinoon sisältyvien kaistojen perusteella ja renderoi se haluttaessa LUT:n avulla.

Käyttöliittymässä tämä koko ketju on **Yhdistetyt kamerat**-matriisin näyttötila: reaaliaikainen yhdistelmäkuva on jo kohdistettu, ja matriisin indeksilaskuri (alla) määrittelee sen renderointikaavan. Tallennetut vientitiedostot voidaan vääristää samaan kohdistukseen**Aligned**-tallennusvaihtoehdolla.

## Indeksilaskuri

Indeksilaskuri laatii indeksilausekkeen, jota reaaliaikainen näkymä ja kamerakohtaiset indeksiviennit käyttävät. Se on yksi jaettu pinta, joka avautuu kahdesta paikasta Kamerat-välilehden sivupalkissa:

* **Kamera-kohtainen**— Live-esikatselu →**Indeksi**-hammasratas (vain RGN

-, OCN

- ja NGB

-Bayer-kamerat; yksittäisellä monokameralla ei ole indeksin hallintaa, koska yhdellä kaistalla ei voi muodostaa indeksiä).
* **Järjestelmäkohtaisesti**— järjestelmän asetukset → Live-esikatselu →**Indeksi**-hammasratas. Tämä on monokamerareitti: kaistaluettelo kattaa**kaikki järjestelmään kuuluvat kamerat**, joten monokamerapari tuo tähän omat kaksi kaistaansa.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Sen säätimet ylhäältä alas:

* **Kaistapiirit** (&quot;Kaistat — napsauta lisätäksesi lausekkeeseen&quot;) — yksi painike kutakin käytettävissä olevaa kaistaa kohti, merkitty värinimellä + aallonpituudella nm:inä (samannimiset värit erotetaan toisistaan esimerkiksi muodossa &quot;Väri 850&quot;). Napsauttamalla lisätään kaistatunnus kohdistimen kohdalle. Kaistat kameroista, jotka eivät pysty tuottamaan kaistakohtaista säteilyvoimakkuutta (RGB

/FRGB), suodatetaan pois.
* **Operaattori- ja funktiopainikkeet** — `+ - * / ( ) ^ ,` sekä `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Lausekkeen tekstikenttä** — vapaasti kirjoitettava kaava; paikkamerkki näyttää klassisen NDVI

-muodon `(NIR - Red) / (NIR + Red)`. Sen yläpuolella oleva, vain luku -tilassa oleva tokenoitu esikatselu renderöi kaistalohkot, numerot ja liput tuntemattomiksi tunnisteiksi.
* **Kelpoisuusbanneri**— harmaa ”Tyhjä — indeksiä ei sovelleta”; vihreä ”Kelvollinen lauseke”; punainen, jossa näkyy tarkka jäsentämisvirhe (tuntematon kaista, useiden kameroiden paljastama epäselvä kaista, puuttuva sulu, …); tai keltainen, kun lauseke on kelvollinen mutta**vakio** (esim. `X/X` tai NDVI

-nimittäjä, joka on kirjoitettu muodossa `−` sen sijaan, että se olisi `+`) — vakio muuntaa koko kehyksen yhdeksi väriksi.
* Erillinen keltainen varoitus ilmestyy, jos käytetty lauseke on kunnossa, mutta **reaaliaikainen kehys on yhtenäinen** (tasainen tai kylläinen kohtaus) — histogrammin romahtaminen havaitaan puolestasi.
* **Käytä LUT:ta**(oletusarvoisesti päällä; pois päältä = harmaasävyn venytys),**Taso**2/3/5/7-pysähdystä (oletusarvo 7-pysähdystä) sekä**Min / Max**-syöttökentät gradienttipalkin molemmin puolin. Min-arvon oletusarvo on**0,2**— se zoomaa väriskaalan kasvillisuuden kannalta merkitykselliseen alueeseen, kun taas sitä pienemmät arvot näkyvät harmaasävyinä; aseta Min-arvoksi −1, jos haluat koko indeksialueen (**Nollaa**-painike palauttaa arvon −1…+1). Max-arvon oletusarvo on 1.
* **Reaaliaikainen histogrammi** indeksin jakaumasta — neliöjuuriskalattuja palkkeja, keltaiset p2/p98-prosenttipisteviivat, valkoinen mediaaniviiva sekä alueen ulkopuolella olevat häntäpään lukemat (&quot;◀ N% &lt; lo&quot; / &quot;hi &lt; N% ▶&quot;), jotka muuttuvat keltaisiksi yli 1 %:n kohdalla merkkinä Min/Max-ikkunan laajentamisesta.
* **Apply**-painike vahvistaa lausekkeen reaaliaikaiseen virtaan; LUT-säädöt tulevat voimaan reaaliaikaisesti ilman Apply-painikkeen painamista. Kun projekti on auki, lauseke ja LUT-asetukset tallennetaan projektin mukana ja palautetaan, kun avaat sen uudelleen; ilman projektia ne säilyvät vain kyseisen istunnon ajan.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->



## CLI

-polku

Sama align → stack → index -ketju, skriptattavissa päästä päähän:

```bash
chloros-cli lattice align-calibrate --serials SN_RED,SN_NIR --save align.json
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`align-calibrate` käyttää liitettyä taulukkoa, joka sisältää sen kamerat, tai avaa ne itse kyseistä suoritusta varten. `index --live` avaa kamerat aina itse ([väliaikainen taulukko](../reference/cli-reference.md#multi-camera-transient-sessions)) eikä sillä ole `--serials`: se avaa kaikki verkossa olevat kamerat. Vapauta siis kaikki kamerat, joita Chloros

:lla on hallussaan ennen sitä — GUI, `cam-connect`, `array-connect` (`chloros-cli lattice array-disconnect --all`): pidätetyn kameran avaaminen toisen kerran voi pysäyttää sen haltijan datavirran tai kaataa taustapalvelimen. Kamera, joka ei lähetä yhtään kuvaa, aiheuttaa ajon epäonnistumisen, eikä pinoa tai hakemistoa tallenneta.

`--channel` yhdistää esiasetuksen symbolit pinon kaistanimiin. Kaksi sääntöä auttaa välttämään epäonnistuneen suorituksen:

* **Symboleissa huomioidaan kirjainkoko**, ja niiden on vastattava esiasetuksen kanavanimiä tarkalleen — esiasetuksissa käytetään pieniä kirjaimia (esimerkiksi NDVI

-esiasetukset ovat `red`, `nir`; katso `--list-presets`). `--channel red=Red_660` toimii; `--channel RED=660` epäonnistuu ja antaa virheen `channel_map missing entries`.
* Kaistapuolella on nimettävä kaista kohdistetusta pinoista (`lattice align-info --profile align.json` luettelee ne). Offline-tilassa hyväksytään myös 0-pohjaiset kaistaindeksit, esim. `--channel red=0 --channel nir=1`.

`lattice index` toimii myös offline-tilassa tallennettua kohdistettua monikaistaisTIFF

-tiedostoa vastaan. Jokainen lomake toimii taustapalvelimella, joten sen on oltava käynnissä ja kirjautuneena maksulliseen Chloros

+-tasoon; suhteellinen `--input` tai `--output` viittaa kansioon, josta komento suoritetaan:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Indeksi-esiasetukset

`lattice index --preset` (ja Kuva-välilehden [Indeksi/LUT-hiekkalaatikko](../image-viewer-gui/index-lut-sandbox.md), joka käyttää samaa moottoria) sisältää nämä **22 esiasetusta**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Suorita komento `chloros-cli lattice index --list-presets` kunkin esiasetuksen kaavan ja kanavasymbolien tarkastelemiseksi ja komento `--list-gradients` käytettävissä olevien värigradienttien tarkastelemiseksi. Mukautetuissa kaavoissa käytetään komentoa `--formula EXPR` samalla syntaksilla kuin indeksilaskurissa. Huomaa, että tämä esiasetuslista koskee nimenomaan LATTICE-indeksimoottoria — tuoduille kuville tarkoitettu Projektin asetukset -valikosta avautuva käsittelyvalikko sisältää eri luettelon (katso [Monispektriset indeksikaavat](../project-settings/multispectral-index-formulas.md)).

Täydellinen lippusarja (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, `--live`:n kohdistus- ja vääristymäsäätimet sekä muut) on dokumentoitu kohdassa [CLI

-viite § Indeksi / Kasvillisuusmatematiikka](../reference/cli-reference.md#index--vegetation-maths); SDK

-vastaavuudet löytyvät [SDK

-ohjeesta](../reference/sdk-reference.md).

## Indeksituotteiden tallentaminen monokromaattisesta taulukosta

Kun taulukko on kytketty ja indeksilauseke on sovellettu, `array-capture` (tai GUI:n **Capture All**) tallentaa kamerakohtaiset vientitasot *ja* indeksin renderoinnin — `--index`/`--no-index` kytkee tämän päälle CLI

:ssä, ja tallennus sisältää oletuksena kaikki sovellettavat tasot. Monokameran osuus kussakin tallennusryhmässä on sen yksi kaista raaka-/debayeroitu (harmaasävy)/säteily-/heijastustasoilla sekä jaettu yhdistetty indeksikomposiitti, kun matriisi toimii yhdistetyllä tilalla. Katso [Monikamerajärjestelmät § Tallennus](arrays.md#capturing-monitoring-vs-analysis).
