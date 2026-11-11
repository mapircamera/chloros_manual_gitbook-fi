# Chloros-CLI-ohje

**Versio:**

1.3.0**Luotu:**

29.7.2026 klo 19.19 ·**Päivitetty:**

6.11.2026**Kohderyhmä:** Optimoitu LLM-käyttöön; ihmisen luettavissa.**Laajuus:** Kaikki `chloros-cli`-komennon käyttäjälle näkyvät alikomennot, mukaan lukien vaihtoehdot ja kopioitavat esimerkit.

Tämä asiakirja on kattava viiteopas MAPIR Chloros -ohjelmiston mukana toimitettavalle `chloros-cli`-komentorivityökalulle. Se on tarkoituksella laadittu mahdollisimman kattavaksi, jotta LLM (tai ihminen) voi luoda minkä tahansa tuetun työnkulun alla olevien luetteloiden perusteella tarkastelematta lähdekoodia.

Jos tarvitset vain pääkohdat, siirry kohtaan:
- [Viiden minuutin pikaopas](#five-minute-quickstart)
- [LATTICE-kameran ensiliitäntätyönkulku](#lattice-camera-first-connect-workflow)
- [DAQ-anturin ensiliitäntätyönkulku](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Tallennustilat, tallentimet ja offline-uudelleenkäsittely](#capture-modes-recorders--offline-reprocess)

> **Chlorosin ohjaaminen tekoälyavustajasta?** Claude Desktop, Claude Code, Cursor ja muut MCP-asiakasohjelmat voivat ohjata prosessointia, LATTICE-kameroita ja -anturiryhmiä sekä DAQ-valosensoreita `chloros-mcp`:n kautta, joka asennetaan Python-SDK:n mukana (sekä Linux-järjestelmän .deb-paketin mukana, kohdassa `/usr/bin`): `claude mcp add chloros -- chloros-mcp`. Asennusohjeet, asetukset ja työkalut löytyvät SDK-viiteoppaasta kohdasta [MCP-palvelin (tekoälyavustajat)](sdk-reference.md#mcp-server-ai-assistants). Shell-ympäristöllä varustettu avustaja voi myös suorittaa kaikki tämän sivun komennot suoraan.

> **Muutettu versiossa 1.3.0** (mitä olemassa oleva 1.2.0-skripti tai -työnkulku huomaa).
>
> **Muutokset, joita 1.2.0-skriptin saattaa olla tarpeen käsitellä:**
>
> - **`--preset` on voimassa.** `lattice capture`, `continuous`, `calibrate`, `multi-capture`, `multi-stream`, `align-calibrate`, `align-apply` ja `align-stream` poistettiin versiosta 1.2.0 ja toimi kameran omilla asetuksilla. Se määrittää nyt esiasetuksen formaatin, valotuksen, vahvistuksen ja kuvataajuuden: `--preset high_speed` tallentaa 8-bittisenä 5 ms:n välein, 6 dB:n vahvistuksella, 60 fps. Myös `lattice cam-connect --preset` säilyttää esiasetuksen formaatin (versiossa 1.2.0 kamera asetettiin 12-bittiseksi esiasetuksen soveltamisen jälkeen). Katso [Yleiset asetukset](#common-options-shared-by-most-subcommands).
> - **M3M (mono)-kameran Bayer-pikselimuoto on sen bittisyvyys.** Esiasetuksissa mainitaan Bayer-muotoja, joita monosensori ei voi tallentaa: versiossa 1.2.0 kamera hylkäsi kyseisen muodon ja säilytti aiemman muodonsa, joten `--preset high_speed` ei ollut siellä 8-bittinen. Se on nyt saman syvyyden Mono-formaatti: `high_speed` ja `--pixel-format BayerRG8` tuottavat `Mono8`:n, `high_quality`, `triggered` ja `BayerRG12` antavat tulokseksi `Mono12`. Sekoitetussa M3C + M3M-matriisissa kukin kamera käyttää oman perheensä yhteistä formaattia, ja `network-analysis` / `analyze-array --pixel-format` projisoivat kunkin kameran kyseiselle syvyydelle. Sama pätee myös päinvastaisessa tilanteessa: Mono-muotoa pyytänyt M3C ottaa kyseisen syvyyden Bayer-muodon, joten matriisi, jonka pääkamera on M3M, ei enää avaa M3C-kameroitaan Mono-tilassa. Katso [Mono (M3M) -yksikaistakamerat](#mono-m3m-single-band-cameras).
> - **`lattice array-capture`-vienti on oletusarvoisesti kohdistettu, kuten sovelluksen vienti.** Jokaisen jäsenen `debayered`-, `display`-, `radiance`-, `reflectance`- ja hakemistotiedostot sijoitetaan pääkameran kehykseen ja rajataan jäsenten päällekkäisyyden mukaan (joka on pienempi kuin anturi kameroiden siirtymien verran, `XMP-mapir:AlignmentJSON` `"applied": true`), joten CLI-tallennus koostuu samoista tiedostoista kuin Chloros-sovelluksen tallennus samasta kohtauksesta; `raw` säilyy alkuperäisessä muodossaan. Aiemmin tallennus ilman `--aligned` tallensi jokaisen tiedoston alkuperäisessä koossa, muunneltamattomana. `--no-aligned` säilyttää tämän. Sama oletusasetus koskee `--smart`:ää, `--continuous`:ään ja `--interval`:ään sekä SDK:hen ja `chloros-mcp`:ään. Katso [`array-capture`-tilat](#array-capture-modes).
> - **`lattice array-capture --compression`:n oletusasetuksena on `none`.** Tiedostot tallennetaan pakkaamattomina (noin 6,3 Mt täysresoluutioista kuvaa kohti) nopeimman jatkuvan tallennusnopeuden saavuttamiseksi; versio 1.2.0 tallensi `deflate`-muodossa (noin 4,1 Mt). Käytä `--compression deflate`-tilaa, jos haluat pienempiä tiedostoja. Molemmat ovat häviöttömiä.
> - **PNG- ja JPEG-tiedostot tukevat vain `display`- ja `preview`-tiloja sekä indeksituotteita.** `--format png`- tai `jpg`-kaappaus, jonka taso ylittää 8-bittisen tiedoston kapasiteetin, hylätään ennen kaappausta (HTTP-virhe 400, poistuminen `1`), samoin kuin oletusarvoinen `--processing all`, jossa on `--format png`/`jpg`. Katso [Tallennuksen vientitasot](#capture-export-levels-the-all-default).
> - **`lattice dls` vaatii `--mac`:n.**Oletus-MAC-osoitetta ei enää ole (se oli yhden yksikön osoite); `daq pool-discover --only ble` etsii anturin osoitteen.**`lattice dls --camera SERIAL`** on uusi.
> - **Lisää virheitä lopettaa `1`.** `process`, kun ajon vientiä ei voitu tarkistaa tai se menetti kuvan, vaikka muut tuotteet on kirjoitettu; tallennuskomento, jossa on tiedosto, jota se ei voinut kirjoittaa (muut tiedostot tallentuvat silti); `lattice continuous` ja `multi-stream`, kun levy on täynnä; `array-burst` / `array-record`, kun tallennin ilmoittaa virheestä; `array-build-video`, kun pyydetty tuote ohitettiin tai sitä ei luotu pyydetyn mukaisesti; `project burst` / `stream`, kun tallennus epäonnistuu kesken; `project sensor log`, kun tallenteessa ei ole lukemia; `lattice exposure --set`, `gain --set` ja `configure --set`, joiden arvo kamera hylkää. `lattice network-analysis --json` poistuu samalla tavalla kuin yhteenveto (`1` ja `needs_force_slip`). Päinvastoin: `daq pool-disconnect`, jonka anturi ei enää kuulu pooliin, lopettaa `0`X:n. Katso [Lopetuskoodit](#exit-codes).
> - **Hidas `lattice array-connect` ei koskaan epäonnistu, kun taulukko on kytkettynä.** Se odottaa enintään 600 s (180 s versiossa 1.2.0; `project connect` 90 s) ja tulostaa samalla taustapalvelimen vaiheen; jos yhteyden muodostus ei ole siihen mennessä valmis, taustapalvelin — jokainen kamera palautetaan siihen tilaan, jossa se löydettiin — joten `Error:` (lopetus `1`) tarkoittaa, ettei kameraryhmää ole kytketty. Versiossa 1.2.0 heti kameroiden virran kytkemisen jälkeen aloitettu liitäntä saattoi kestää odotusaikaa pidempään ja liittää matriisin sen jälkeen, kun komento oli epäonnistunut. Ctrl+C peruuttaa nyt liitännän. Katso [Pysyvät yhteydet](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-connect` säilyttää koko anturin.** Ilman `--width` / `--height` / `--binning`-asetuksia jokainen kamera asetetaan käyttämään koko anturia, ja kuvataajuus laskee tämän tietokoneen ja verkon tarjoamaan tasoon; yhteyden muodostuksen tulosteessa luetellaan nopeampia vaihtoehtoja (binning tai pienempi ROI) ja niiden ennustetut nopeudet. Versiossa 1.2.0 kuvia pienennettiin tai binnoitiin kuvataajuuden ylläpitämiseksi. Katso [`array-connect`-asetukset](#array-connect-options).
> - **Hidas verkko ei enää estä matriisin toimintaa.** Jokaisen kameran nopeutta sopeutetaan sen osuuden mukaan verkkokaistanleveyteen, ja kuvataajuus laskee sen mukana. Jos kameroiden nopeutta ei voida sopeuttaa niin hitaaksi, matriisi toimii sarjalähetystilassa (`slip-emit-and-capture`): jokainen kehys saapuu, mutta kameroita ei valoteta samanaikaisesti. `--force-tier sim-capture-sim-emit` vaatii samanaikaista valotusta. Katso [Kuvasarjan kuvataajuus ja sarjakuvausmalli](#array-fps--burst-model).
> - **`lattice network-analysis` / `analyze-array` määrittävät, mitä connect-asetusta sovelletaan**: ilman `--pixel-format`:tä, connect-asetukset määrittävät formaatit (Mono12 / BayerRG12, aiemmin BayerRG8) sekä [pakatun 12-bittisen](#packed-12-bit-transport---packed) valinnan. Sama komento voi siis ilmoittaa eri kuvataajuuden kuin versiossa 1.2.0.
> - **Suhteelliset polut viittaavat siihen paikkaan, jossa komento suoritetaan.** `-o` ja muut polut, jotka taustaprosessi avaa (`--input`, `--burst-dir`, `--profile`, `--save`, `--mask`), reseptivaiheen `output`, sekä oletusarvoiset `output/` ja `lattice index --live` määritetään sen kansion perusteella, josta `chloros-cli` suoritetaan. Versiossa 1.2.0 taustapalvelin ratkaisi ne omassa asennuskansiossaan, mikä johti asennetussa versiossa virheeseen ”Pääsy evätty”. Katso [Käytännöt](#conventions).
> - **`lattice index` toimii taustapalvelimella kaikissa muodoissaan**, kuten muutkin `lattice`-komennot, joten käynnistä ensin taustapalvelin. Offline-hakemisto (`--input`) lasketaan taustapalvelimella, joka lukee TIFF-tiedoston ja kirjoittaa tulokset sen viereen tai kohteeseen `-o`, ja `--list-presets` / `--list-gradients` tulostavat taustapalvelimen taulukot. Windowsissa nämä toimivat nyt: aiemmin Windowsin CLI vastasi kaikkiin `lattice index`-muotoihin, mukaan lukien `--live`, komennolla `Error: lattice_sdk not available`. Linuxissa offline-indeksi toimi aiemmin CLI:ssä ilman taustapalvelinta ja kirjautumista; nyt se tarvitsee molemmat, kuten kaikki muutkin CLI-komennot. Katso [Indeksi / Kasvillisuusmatematiikka](#index--vegetation-maths).
> - **Uudet tiedostot tuotteiden rinnalla.** Jokaiseen säteily-, heijastus- ja indeksituotteeseen, jossa kaistan erottelu on rajoittanut pikselin, lisätään maski tuotteen kansion vieressä olevaan `clamp_masks/`-kansioon, ja tulostukseen lisätään rivi `clamp mask:`. Katso [Unmix-rajoitusmaski](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on).
> - **Kameran suunta.** Yhden kameran tallennus tallentaa tiedostot `raw` ja `debayered` sellaisina kuin anturi ne luki ja tallentaa kameran käännön / kiertämisen XMP-tiedostoon; `process` soveltaa sitä tuottamiinsa tuotteisiin. Taulukon jäsen-tiedostot pysyvät alkuperäisessä muodossaan, ellet välitä uutta `--orient` (`array-capture`, `array-burst`, `project capture` / `burst` / `stream`). Katso [Kameran suunta](#miltä-tallennuskansio-näyttää).
> - **Vientitason nimi, joka ei ole yksi, hylätään.** `lattice capture --processing` ja kaikki `--levels`-luettelot hyväksyvät `raw`, `debayered`, `radiance`, `reflectance` tai `display` (tai aliaksena esimerkiksi `preview`; `--processing` sekä `all`). Mikä tahansa muu nimi hylätään ennen kuin kameraa kosketetaan (HTTP 400 `UNKNOWN_LEVEL`, poistuminen `1`); 1.2.0 tallensi sille nimen `raw` ja lopetti toiminnon `0`.
> - **`daq pool-set-cap` tarvitsee `--cap-id`:n.** Ilman sitä komento lopetetaan `2`; 1.2.0 poisti korkin korjauksen, jolloin jäljelle jäi paljas kenno. `--cap-id none` pyytää paljaata anturia.
> - **`lattice resolution --binning-mode` on poistettu.** Binning on aina keskiarvo; lippua hyväksytään edelleen, se tulostaa varoituksen eikä tee mitään. Katso [Anturin säätö](#sensor-tuning).
> - **`process --debayer texture-aware` hylätään LATTICE-tallenteiden yhteydessä.** LATTICE-kuvauksia sisältävän kansion kohdalla `process` lopettaa `1`:n tuonnin jälkeen ja ilmoittaa kuvien lukumäärän ennen minkään käsittelyä. LATTICE-kuviin sovelletaan aina vakiomuotoista demosaicia, joten versio 1.2.0 hyväksyi vaihtoehdon mutta jätti sen huomiotta; myöskään Chloros-sovellus ei tarjoa sitä näille kuville. Katso [`chloros-cli process`](#chloros-cli-process).
> - **Kamerat palautetaan siihen tilaan, jossa ne löydettiin — ellei `--keep-settings`.** `lattice array-disconnect`, jokainen väliaikainen komento (`multi-capture` ilman taulukkoa, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`) sekä `lattice continuous` palauttavat jokaisen kameran siihen tilaan, jossa se oli: laukaisimen ja synkronointilinjan johdotuksen, PTP-asetukset sekä kaikki istunnon aikana muutetut kameran asetukset (valotus ja vahvistus, AE-kohde, rajat ja vaimennus, valkotasapaino, mustataso, pikselimuoto, pikselien yhdistely ja ROI, käännöt, kuvataajuus). `--keep-settings` puolestaan tallentaa kunkin kameranasetukset kameran omaan muistiin, jolloin se käynnistyy niillä; `--clear-settings` nollaa kyseisen muistin tehdasasetuksiin. Kummassakin tapauksessa kamerat jätetään vapaasti toimimaan. Projekti asettaa kameransa omista tallennetuista asetuksistaan, kun se yhdistää ne. Katso [Kameran muisti](#camera-memory----keep-settings----clear-settings).
> - **Jokainen tila tallentaa projektin samalla tavalla.** Kaikki muutokset projektiin liitettyyn kameraan, matriisiin tai valosensoriin – olipa ne tehty sovelluksella, CLI:llä, SDK:llä tai MCP-palvelimella – tallennetaan kyseisen projektin `cameras.json` / `sensors.json` -tiedostoon, ja projektin avaaminen missä tahansa tilassa asettaa laitteet sen mukaisiksi. Aiemmin vain sovellus tallensi projektin, ja sen seuraava tallennus korvasi kaikki muut tallennukset. Katso [Yksi projektitallenne, kaikki tilat](#one-project-record-every-mode).
> - **Useista kameroista luodut tiedostot nimeävät jokaisen kameran.** Monikaistapino (`align-apply`, `align-stream`, `index --live --save-multiband`), useiden kameroiden kaistoista muodostettu reaaliaikainen indeksi sekä taulukkomuotoisen kuvan `composite/` sisältävät kunkin lähdekameran sarjanumeron, mallin ja kalibrointipaketin luetteloina (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), ja EXIF-tiedostossa `SerialNumber` sarjanumerot on lueteltu pilkuilla erotettuina. Aiemmin niissä ei ollut lainkaan kameratietoja tai niissä oli vain ensimmäisen kameran sarjanumero. Katso [Useista kameroista luodut tiedostot](#files-made-from-several-cameras).
> - **Vain `raw`-tiedosto sisältää kameran kalibrointitunnisteet.** MicaSense-yhteensopiva sarja (`BlackLevel`, `RadiometricCalibration`, kanavakohtainen `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`-sarjat) sekä `CalibrationJSON`-hyötykuorma kuvaavat anturin raaka-arvoja, joten ne sisältyvät ainoastaan `raw`-tallennustiedostoon. Kaikki muut tiedostot — tallennuksen `debayered`-, `display`-, `radiance`-, `reflectance` ja hakemistotiedostot, `lattice continuous` ja `multi-stream`-kehykset, sekä jokainen `process`-tuote — sisältää kameran tunnisteen (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) sekä kuvan ottamisen yhteydessä kehyksen oman tietueen (`XMP-mapir:FrameJSON`: valotus, vahvistus, lämpötila, pikselimuoto, kuvausajankohta). Versiossa 1.2.0 ne sisälsivät myös raakadatan, jota soveltava työkalu (Pix4D, Metashape) sovelsi pikseliin, jotka eivät olleet raaka-arvoja. `process` löytää tällaisen tiedoston kalibroinnin sen sarjanumeron ja `CalBundleSha256`:n avulla.
>
> **Muut muutokset:**
>
> - **Taulukon automaattinen kuvataajuus sopeutuu taulukon tuottamaan taajuuteen.** Taajuus, joka menettää yli 1 % laukaisuistaan, laskee, mikä arvioidaan useiden pitkien tarkastelujaksojen perusteella yhden sijaan, ja taajuutta, josta se laski, yritetään uudelleen silloin tällöin. Itse asettamaasi taajuutta ei muuteta.
> - **Automaattinen valotus on vakaampi.** Se pysyy vakiona tasaisessa valaistuksessa, vakiintuu muutoksen jälkeen muutamassa sekunnissa, jättää pienet kirkkaat kohdat huomiotta ja palauttaa toimintakyvyn kameralle, joka lakkaa reagoimasta kirkkaassa auringonpaisteessa. Katso [Automaattisen valotuksen toiminta](#how-auto-exposure-behaves).
> - **`process` laskee vain omat kuvasi ja tallenteesi.** Kun suoritat ohjelman uudelleen kansiossa, jossa on projektin `camera_cal/`, se ei enää laske kyseisiä kalibrointitiedostoja mukaan (”Löytyi 96 kuvaa” 90 kuvan sarjassa), ja projekti, joka on jo ottanut ne huomioon, hylkää ne.
> - **`lattice multi-test` ottaa lähtökohdakseen kameroiden virran kytkemisen jälkeisen tilan.** Se kytkee kameroiden PTP:n päälle ajon ajaksi, odottaa niiden kellojen lukittumista ja vakiintumista ennen ensimmäistä laukaisua — jokaisen kameran kello lukittuu 120 sekunnin kuluttua virran kytkemisestä (`--ptp-settle SECONDS` asettaa sen), sillä juuri lukittunut kello voi poiketa jopa ~200 us noin 2 minuutin ajan — ja palauttaa kunkin kameran PTP:n siihen tilaan, jossa se sen löysi. Kameraa, joka on edelleen lukittunut aiemmasta testistä, ei odoteta uudelleen. Aikaisemmin PTP jätettiin pois päältä yksikytkimisessä verkossa, jossa mikään DAQ-E ei vastannut, ja kunkin kameran vapaasti käyvää kelloa luettiin PTP-kellona, joka ei lukittunut (`PTP is not locked`, jokaisella ajolla). Katso [Monikamera (väliaikaiset istunnot)](#multi-camera-transient-sessions).
> - **`.daq`-tallenne kertoo, valittiinko sen suojus vai oletettiinko se.** DAQ-U-, DAQ-M- ja DAQ-E-laitteet toimitetaan ilman suojakantta, eChloros-ohjelmisto ei voi nähdä, onko sellainen asennettu, joten anturi, joka on kytketty ilman nimettyä suojakantta, saa oletusarvoisesti tunnuksen `sunshine_cosine`, mikä on *oletus*. Jokainen Chloros-tallenne (skeema 1.30) tallentaa nyt `cap_id_source`:n `cap_id`:n rinnalle: `operator`, jos olet nimennyt suojuksen (mukaan lukien `none`), ja `auto_default`, jos se oli oletusarvo. Kun käsitellään tallennetta, jonka suojuksen olemassaolo on oletettu, ja projektin DAQ-suojuksen asetus on jätetty tilaan Auto, tulostaa `[daq-dls] WARNING`-viestin, jossa mainitaan tiedoston nimi: pelkkä DAQ-U auringonvalon korjauksen alla lukee noin 30-kertaisesti liian korkean arvon. **Nimeä asettamasi yläraja** liitettäessä tai aseta projektin DAQ-yläraja, niin varoitus katoaa.
> - **Windowsissa ei enää avaudu pip-konsoli-ikkunoita.** Tietokoneella, johon on asennettu Python, taustaprosessi ja jokainen sen kuvankäsittelytyöntekijä suorittivat `pip`-komennon kerran käynnistyksen yhteydessä, ja kukin avasi konsoli-ikkunan. Näin ei enää tapahdu.
> - **`.daq`-reaaliaikaiset tallenteet säilyvät sähkökatkon jälkeen.**Tallenne kirjoitetaan lokiin ja levyn täyssynkronointi suoritetaan jokaisen tallennuksen yhteydessä anturin taustalla, joten hidas levy viivästyttää tiedostoa, mutta ei koskaan lukemia. Tallennus, joka keskeytyi kesken tallennuksen (sähkökatkos, pakotettu lopetus), voi jättää `<name>.daq-journal`-tiedoston viereensä:**kopioi nämä kaksi yhteen.**Chloros palauttaa lokikirjan ensimmäisellä tiedoston lukukerralla, joten tallenne avautuu sisältäen kaiken, mikä oli vahvistettu ennen keskeytyksen hetkeä (noin viimeiset 3 sekuntia menetetään); versiossa 1.2.0 väärällä hetkellä tapahtunut virrankatkos saattoi jättää koko tiedoston lukukelvottomaksi (&quot;tietokannan levykuva on virheellinen&quot;).
> - **Tallennus ruuhkaiselle levylle ei enää jumita anturia.** `daq pool-record` asettaa lukemat jonoon ja kirjoittaa ne omalla säikeellään; lukema, jota ei voitu kirjoittaa, lasketaan (`daq pool-list` näyttää `N write error(s)`:n viimeisen virheen kanssa) sen sijaan, että se jumiuttaisi anturin (yksi kirjoitus kesti 8,4 sekuntia neljän TIFF-tiedostoja tallentavan kameran rinnalla).
> - **`lattice multi-capture` ilman taulukkoa ei enää menetä tallentamiaan kehyksiä, kun laukaisija aikakatkaistaan.** Kameran laukaisun aikakatkaisu (`SC_ERR_TIMEOUT -1011`, havaittu neljän kameran ollessa kytkettynä yhteen 1 GbE -kytkimeen) lopettaa ajon kyseisessä kehysryhmässä: jo tallennetut ryhmät luetellaan `Saved:`:llä, minkä jälkeen tulee virhe ja exit `1`. Versiossa 1.2.0 se oli HTTP-virhe 500 ilman luetteloa, vaikka tiedostot olivat levyllä.
> - **Ohjelmistokäynnistetyt kamerat eivät enää jätä joka toista laukaisua väliin.** Ohjelmistokäynnistystilassa oleva kamera asettaa nyt jonotukseen laukaisun, joka saapuu sen lukemisen aikana (tai Chloros odottaa, että se on valmis), kuten kaapelisyntetisoidut kamerat jo tekivät.
> - **`daq pool-discover` nimeää DAQ-E-S:n `DAQ-E-S`** `MODEL`-sarakkeessaan (ja `--json`:n `model`). Se ilmoitti jokaiselle Ethernet-anturille nimeksi `DAQ-E`; mutta anturi, joka ei vastaa tunnistustarkistukseen noin 2 sekunnin kuluessa, näkyy edelleen luettelossa nimellä `DAQ-E`.
> - **Parannetut LATTICE-indeksituotteet** (NDVI ja muut indeksit) alkaen versiosta `process` kaikissa käsittelytiloissa.
> - **OCN-kamerat: parannetut säteily-, heijastus- ja indeksituotteet auringonvalossa.** Laivuekorjaus sovelletaan kaistojen erottelun jälkeen; RGN-kameroissa ei ole muutoksia, ja OCN-arvot poikkeavat versiosta 1.2.0. Se koskee myös aikaisemmalla versiolla otettua OCN-kuvaa, kun se käsitellään tietokoneella, jossa on tallennettuna kuvan ottamisen aikainen kalibrointi (Chloros säilyttää kameran kalibroinnin, kun kamera on kytketty).
> - **`lattice continuous` tallentaa raakakuvan.** `tiff`-ajo kirjoitti demosaikoidut 3-tasoiset kuvat, joista `process` ei voi aloittaa. Sen kuvat ovat nyt `raw`-tiedosto, jonka array-tallennus kirjoittaa: yksi mosaiikkitaso anturin omilla arvoilla, pakkaamaton, kalibrointipaketin kera. Skripti, joka lukee kolme tasoa näistä tiedostoista, lukee vain yhden.
>
> - **Jokainen tallennus tallentaa käyttämänsä DAQ-lukeman, kuten käyttöliittymäkin tekee.** `lattice capture`, `lattice continuous` (yksi `.daq` kehystä kohti), `array-capture` ja `multi-capture`, joihin on liitetty taulukko (ja SDK), kirjoittavat tiedostot `.daq`:n ja sen `.match.json`:n aina, kun DAQ sidotaan tai yhdistetään, mukaan lukien pelkästään raakamuotoinen tallennus; versiossa 1.2.0-versiossa raaka- tai debayeroitu tallennus tallennettiin vain `--force-daq`:n avulla. `--no-daq` poistaa sen käytöstä, `--daq-tolerance-ms MS` asettaa kyseisen tallennuksen vastaavuustoleranssin, ja uusi `daq pool-bind` sitoo kameran tai matriisin DAQ:hen CLI:n kautta. `.daq`:n avulla tallennettu tallennus täsmäytetään `process`:ssä vain niihin kameroihin, jotka siinä nimetään. Tallennus odottaa nyt DAQ:n seuraavaa lukemaa, jotta se voi yhdistää valotuksen molemmin puolin olevat lukemat. Katso [Tallennuksen tallentama DAQ-lukema](#the-daq-reading-a-capture-saves) ja [Valotuksesta kaukana oleva DAQ-lukema](#a-daq-reading-far-from-the-exposure).
> - **`process --indices NAME`** lisää indeksin projektiin kunkin kameran kaistoille, mukaan lukien uusi projekti. Versiossa 1.2.0 projekti, jota nimiluettelossa ei vielä ollut, ohitettiin komennolla `Unknown index name(s) ignored`.
> - **`process` reaaliaikaisen matriisin vieressä:**se ei enää käynnistä uudelleen kameraa tai DAQ-laitetta sisältävää taustaprosessia, vaan odottaa enintään 15 sekuntia päättyvää ajoa.**Ctrl+C** pysäyttää ajon taustaprosessissa, ei vain CLI:ssä. Katso [`process`-huomautukset](#notes).
> - **Windowsin 260 merkin polkurajan ylittävä projektikansio** hylätään heti käyttämällä pisintä sopivaa kansiota sen sijaan, että suoritus ei kirjoittaisi mitään. Windows-asennusohjelma tarjoaa nyt vaihtoehdon ”Ota Windowsin pitkät polut käyttöön” sekä Microsoft Defenderin poissulkemisen taustapalvelimelle; molemmat ovat oletusarvoisesti valittuna. Katso [Windows (.exe)](#windows-exe).
> - **`lattice array-capture --continuous`** suorittaa taustapalvelimen oman jatkuvan tallennuksen: edistymispalkki 2 sekunnin välein, yhteenveto tiedostokohtaisen luettelon sijaan sekä nollasta poikkeava poistumiskoodi, jos jokin tiedosto jää kirjoittamatta tai levy täyttyy. Katso [`array-capture`-tilat](#array-capture-modes).
> - **Ensimmäinen `array-capture` `array-connect`:n jälkeen** odottaa enintään 20 sekuntia matriisin ensimmäistä kehysryhmää sen sijaan, että se keskeyttäisi toiminnon HTTP-virheellä 503 ”Array still warming up”.
> - **Matriisin valotus ROI-alueen alla:** pieni ROI-alue ei enää rajoita matriisin jäsenen valotusta anturin vapaakäyntijaksoon (4,9 ms resoluutiolla 768 × 304). Katso [Valotus matriisin jäsenessä](#exposure-on-an-array-member).
> - **Automaattinen valotus ei enää laske matriisin automaattista kuvataajuutta.** Kaapelilla synkronoitu matriisi toimii sillä taajuudella, jonka verkko ja kameroiden lukunopeus sallivat; automaattinen valotus täyttää kyseisen taajuuden antaman jakson ja nostaa sitten vahvistusta. Ajanjaksoa pidempi manuaalinen valotus (`lattice exposure --set`, `configure --set`) hidastaa laukaisua sen mukaiseksi sen sijaan, että se hylättäisiin, ja kuvausnopeus palautuu, kun jäsen palaa automaattitilaan. Katso [Valotus matriisin jäsenessä](#exposure-on-an-array-member).
> - **Matriisit tuottavat vähemmän epätäydellisiä kehysryhmiä** ja vähemmän 503-virheitä (&quot;Ei uutta synkronoitua kehysryhmää&quot;): jokaisen orjakameran laukaisusignaali on nyt puhdistettu häiriöistä.
> - **Pakattu 12-bittinen siirto** (`--packed`, oletusarvo `auto`) nostaa koko anturin taajuutta noin kolmanneksella. `array-connect --packet-size` on nyt käytössä (versio 1.2.0 hyväksyi sen mutta jätti sen huomiotta). Katso [Packed 12-bit transport](#packed-12-bit-transport---packed).
> - **Matriisi, jonka kamerat palautuvat itsestään virrankatkaisun jälkeen.** Kytkimen virrankatkaisun jälkeen `array-connect`:n matriisi-avautuu uudelleen, kun kamerat vastaavat, ja jatkaa lähetystä PTP:n ollessa jälleen päällä; versiossa 1.2.0 se pysähtyi lopullisesti ja vaati uuden `array-connect`:n. `array-status` raportoi `ptp_enabled: False`:n (ja `ptp_off:`:n), kun kamera lukee PTP:n pois päältä, kun se aiemmin ilmoitti asetuksen, jolla järjestelmä oli kytketty. Katso [Pysyvät yhteydet](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-record` tallentaa, vaikka GUI ei ole auki.** Kun GUI:n yhdistetty näkymä on suljettu, taustapalvelu renderöi näkymän itse: matriisinNDVI-arvo tai indeksi, jonka välität uuden `--expr`:n avulla. Tallennus, joka ei kirjoita yhtään kehystä, epäonnistuu nyt (`no-frames: …`, poistu `1`). Aiemmin tallennus tapahtui vain silloin, kun käyttöliittymän yhdistetty näkymä oli auki, ja tallennus, joka ei kirjoittanut mitään, lopetti silti `0`:n. Katso [`array-record`](#array-record--combined-index-videogif-monitoring-grade).
> - **Chlorosin jo hallussa olevaa kameraa ei avata toista kertaa.** Komento, joka yrittäisi avata sen itse, saa vastauksena HTTP-virheen 409 (`CAMERA_HELD`), jossa mainitaan, kuka kameraa hallitsee ja mikä komento vapauttaa sen. Tämä koskee myös kameraa, jota toinen komento käyttää parhaillaan (esimerkiksi `multi-test` tai `multi-capture`, jotka avaavat kameroitaan, tai `lattice capture`): toinen komento hylätään, kunnes ensimmäinen on suoritettu loppuun. Sen avaaminen uudelleen voisi keskeyttää haltijan videovirran tai kaataa taustapalvelimen. Katso [Pysyvät yhteydet](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **Väliaikaiset monikamerakomennot toimittavat jokaisen kameran tai epäonnistuvat.** `lattice multi-capture` ilman liitettyä kameraryhmää, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream` ja `index --live` lopettavat toimintansa, kun `1` nimeää kameran, joka ei toimittanut yhtään kuvaa. Virtat ohittavat kuvaryhmän, josta kamera puuttuu, ja pysähtyvät kolmen peräkkäisen jälkeen; `multi-test` tulostaa yhden rivin kutakin laukaisua kohti, `(no frame)` siinä kohdassa, jossa kamera jäi väliin; `align-stream` lopettaa myös `1` työvian vuoksi. Jokainen suoritus palauttaa sitten jokaisen kameran siihen tilaan, jossa se sen löysi: sen laukaisun ja synkronointilinjan johdotuksen, sen PTP (`PtpEnable`) sekä kaikki ajon aikana muutetut kameran asetukset — valotus ja vahvistus, AE-kohde, rajat ja vaimennus, valkotasapaino, mustataso, pikselimuoto, binning ja ROI, käännöt, kuvataajuus — ellei `--keep-settings` / `--clear-settings` (katso [Kameran muisti](#camera-memory----keep-settings----clear-settings)). Aiemmin nämä komennot tallensivat kaikki saapuneet kehykset ja poistivat `0`-tilasta (`align-stream` tulosti työvian `Note:`:n yhteydessä), ja jättivät kaikki kamerat tilaan `TriggerMode` `On`. Ne asettivat myös kunkin kameran valotuksen edellisestä kohtauksesta ennen kuvan tallentamista (enintään muutama sekunti), kohteessa, jossa liitetty matriisi käynnistää jokaisen kameran asetuksilla: 40 multispektraalista, 60 RGB-kuvaa (`--brightness N` asettaa ne) ja pitää ne koko ajon ajan; samoin toimii yksittäinen `lattice capture`-kamera, jota Chloros ei hallitse. Aiemmin kunkin kameran ensimmäinen kehys otettiin kameran omalla valotusrajalla, noin 100 ms, valaistuksesta riippumatta, ja se leikkautui usein. Katso [Monikamera (väliaikaiset istunnot)](#multi-camera-transient-sessions).
> - **Väliaikaiset monikamerakomennot jakavat verkon samalla tavalla kuin `array-connect`, ja avaavat vain `--serials`:n.** Jokaiselle kameralle varataan sen osuus isäntäyhteydestä, joten neljä kameraa toimii 1 Gbps:n yhteydellä; aiemmin neljä kameraa pyysi yhteensä 125 MB/s, kaikki 1 GbE:n kautta, ja jokaisesta kehyksestä hävisi paketteja. Verkon kantokyvyn ylittävä ryhmä hylätään ennen kuin yksikään kamera alkaa lähettää. `--serials` valitsee nyt kamerat, joista ensimmäinen laukaisee toimintoa; aiemmin jokainen komento avasi kaikki verkossa olevat kamerat riippumatta siitä, mitä `--serials` määräsi. GVSP-pakettihäviönä ilmenevä vika ilmoitetaan nyt sellaisenaan sen sijaan, että syyksi osoitettaisiin M8-synkronointikaapelia. Katso [Monikamera (väliaikaiset istunnot)](#multi-camera-transient-sessions).
> - **`lattice array-disconnect` jättää kamerat toimimaan vapaasti, sellaisina kuin ne löytyivät.** `TriggerMode`, `Off` ja kaikki muu sellaisena kuin yhteys ne löysi — laukaisija- ja synkronointilinjan asetukset (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`), PTP sekä istunnon aikana muutetut kameran asetukset — joten yksikamera-komento entiselle jäsenelle hakee kehykset kerralla, eikä pääkameraa enää jätetä tilaan `TriggerSource` `Software`. `--keep-settings` tallentaa istunnon asetukset kunkin kameran muistiin, ja `--clear-settings` nollaa kyseisen muistin (katso [Kameran muisti](#camera-memory----keep-settings----clear-settings)). `lattice continuous` asettaa myös kameran, jonka se havaitsee aktivoituna, vapaakäyttöön ajon ajaksi (ja palauttaa sen takaisin sen jälkeen), ja ajo, jossa ei ole kehystä, poistuu `1`:stä.
> - **`lattice configure --get` / `lattice exposure`** tulostavat `note:`:n, jossa on kamerantodellisen valotuksen tai vahvistuksen, kun sen automaattitila on käynnissä. Hylätty `lattice exposure --set` / `configure --set` ilmoittaa kameran hyväksymän alueen ja palauttaa automaattitilan sen sijaan, että jättäisi automaattivalotuksen pois päältä. Katso [Anturin säätö](#sensor-tuning).
> - **Uudet kameran säätötoiminnot:**`lattice highlights` (vähentää kirkkaiden valkoisten pintojen kirkkautta RGB-näytön renderoinnissa) ja `lattice exposure --brightness-auto`.**`lattice multi-detect`** etsii kaapeliliitetyt kamerat yhden mittauskierroksen aikana ja asettaa pääkameraksi oletuksena RGB-kameran.
> - **Uudet DAQ-komennot:** `daq pool-bind`; `pool-enable-ptp`; sekä DAQ-E:n omia asetuksia varten `pool-info`, `pool-set-network`, `pool-set-auto-stream`, `pool-reboot` ja `pool-update-firmware`. Katso [DAQ-E-laitteen asetukset](#daq-e-device-settings).
> - **Backend muodostaa uudelleen yhteyden DAQ-E-laitteeseen, joka on lakannut lähettämästä dataa.** Kun poolattuun DAQ-E-laitteeseen liittyvä datavirta katkeaa (esimerkiksi sen virransyöttökytkimen virrankatkaisu tai kamerayhteyden katkeaminen), taustapalvelu muodostaa siihen uudelleen yhteyden aiemmilla asetuksilla ja yrittää uudelleen yhä pidemmillä väliajoilla laitteen ollessa poissa. Tallennus jatkuu samassa `.daq`-tiedostossa, ja tauko merkitään tiedostoon, ja tiedostossa `pool-record --stop` luetellaan jokainen katkos. Versiossa 1.2.0 anturi pysyi hiljaa, kunnes tiedostot `pool-disconnect` ja `pool-connect` luotiin, mikä myös lopetti tallennuksen. Katso [DAQ-E, joka lakkaa toimittamasta](#a-daq-e-that-stops-delivering).
> - **`time-sync status`** ilmoittaa, saavuttavatko PTP-grandmasterin paketit verkon (`health`), ja jos eivät, tulostaa `WARNING:` stderr-virheilmoituksen. Katso [`time-sync`](#chloros-cli-time-sync).
> - **Linuxin .deb-paketti:** se kasvattaa ytimen socket-vastaanottopuskureita, joita kameravirrat tarvitsevat (ytimen oletusasetuksilla menetetään kehyksiä ja kehysnopeutta), lisää taustapalvelun, joka toimii käyttäjän nimissä, ja `lattice network` tarkistaa puskurit. CLI-käyttöliittymä ei koskaan pysäytä palvelun ajamaa taustapalvelua, ja päivitys käynnistää käytössä olevat taustapalvelut uudelleen. Katso [Taustapalvelun ajaminen palveluna](#running-the-backend-as-a-service-headless) ja [Linux-isäntäkoneet — socket-vastaanottopuskuri](#linux-hosts--socket-receive-buffers).
> - **`lattice array-capture` ilmoittaa, kun kohdistamista ei voitu suorittaa:**jokainen syy, jonka vuoksi kaappaus tallennettiin kohdistamattomana (ei kohdistusprofiilia, eri resoluutioilla olevat jäsenet, taulukko, jota ei ole synkronoitu kaapelin kanssa, kehys, johon profiilia ei voida kartoittaa) tulostetaan `Alignment:`-rivinä.**`chloros-cli daq help`** on `daq --help` (se lopetti `2`), ja Linuxissa ohjeessa luetellaan suorat laitteistokomennot `daq` , jotka kyseinen rakennus sisältää.
> - **Jetson:** paketti on `chloros_1.3.0_arm64_jp6.deb`, JetPack 6:lle, ja sen GPU-käsittely toimii Orin-laitteella JetPackin CUDA PyTorch -ympäristössä. Katso [Linux arm64 — Jetson](#linux-arm64--jetson-jetpack-6).

---

## Nimeämiskäytännöt

- Kaikkien komentojen etuliitteenä on `chloros-cli`. Windowsissa binääritiedoston nimi on `chloros-cli.exe`; Linuxissa/Jetsonissa se on `chloros-cli`.
- Valinnaiset argumentit esitetään muodossa `--flag`. Pakolliset sijaintipohjaiset argumentit esitetään ilman sulkeita.
- Jos oletusarvo on määritetty, lippua jättämällä käytetään kyseistä arvoa.
- CLI on kevyt HTTP-asiakasohjelma, joka käyttää Chloros-taustapalvelinta (palvelinta osoitteessa `127.0.0.1:5000`, jota myös työpöytäsovellus käyttää). **Mitkä komennot käynnistävät sen:**
  - Ydinkomennot — `process`, `login`, `logout`, `status`, `export-status`, `time-sync` ja `selftest` — käyttävät käynnissä olevaa taustapalvelinta tai käynnistävät sen, jos sellaista ei ole käynnissä, ja pysäyttävät sen uudelleen, kun ne ovat valmiit.
  - **Komennot `lattice`, `project` ja `daq` eivät koskaan käynnistä taustapalvelua.** Ne edellyttävät jo käynnissä olevaa taustapalvelinta: Windows-työpöytäsovelluksen taustapalvelinta, Linuxin [taustapalvelua](#running-the-backend-as-a-service-headless) tai sellaista, jonka käynnistät itse (katso [Käynnistä taustapalvelin](#start-the-backend)). Ilman taustapalvelua ne tulostavat virheilmoituksen `Error: Chloros backend not reachable at http://127.0.0.1:5000` sekä käskyn, jolla se käynnistetään, ja lopettavat toimintansa virheilmoituksella `1`.
- `CHLOROS_BACKEND_URL=<url>` ohjaa **`lattice`**-,**`project`**- ja**`daq pool-*`** komentoperheisiin toisessa taustapalvelimessa (`lattice network-analysis` puolestaan ohjaa `--backend-url`:ään). Ydinkomennot kiinnittävät tarkoituksella `http://127.0.0.1:<port>`:n ja ohittavat sen (IPv4-literaali välttää Windowsin `localhost`→`::1` -viiveen, joka on noin 2 sekuntia pyyntöä kohti). Katso [Ympäristömuuttujat](#environment-variables).
- **Suhteelliset polut ratkaistaan sen kansion suhteen, josta `chloros-cli` suoritetaan.** Taustaprosessi avaa tulostus- ja syöttökansiot, joten ennen polun lähettämistä CLI muuttaa sen absoluuttiseksi: `-o` / `--output` (mukaan lukien sen oletusarvo, `output`), `--input`, `--burst-dir`, `--profile`, `--save` ja `--mask`, [resepti](#recipe-dsl)-vaiheen `output` (oletusarvo `output`), sekä oletuskansio `output`, joka sijaitsee kansiossa `lattice index --live`. `-o captures/` kirjoittaa siis kansioon `<current folder>/captures/`, ja tulostetut polut ovat absoluuttisia. `~` laajennetaan. Etätaustapalvelinta vastaan (`CHLOROS_BACKEND_URL`, joka viittaa toiseen koneeseen) polut lähetetään sellaisina kuin ne on kirjoitettu, ja ne ratkaistaan kyseisellä koneella.
- Chloros+-tilin kirjautuminen vaaditaan kaikissa SDK- ja CLI-kutsuissa -kutsuihin (suorita `chloros-cli login` kerran kutakin konetta kohti; tallennetaan välimuistiin tiedostoon `~/.chloros/`).
- Esimerkeissä käytetään Linux-polkuja; Windowsissa korvaa `/home/user/...` komennolla `C:/Users/.../...`. DAQ-U:nsarjaportti on Windowsissa muotoa `COM3` ja Linuxissa muotoa `/dev/ttyUSB0` / `/dev/ttyACM0`.
- Yhteenvedoissa toisiaan poissulkevat liput kirjoitetaan muodossa `{--a | --b}` ja valinnaiset liput muodossa `[...]`.

---

## Ylätason synopsis

```
chloros-cli [global options] COMMAND [command options]
```

### Globaalit asetukset

| Lippu | Kuvaus |
| --- | --- |
| `--backend-exe PATH` | Ohittaa automaattisesti tunnistetun taustaprosessin suoritustiedoston. |
| `--port N` | Taustaprosessin HTTP-portti (oletus: `5000`). Vain ydinkomentojen käytössä (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`); komennot `lattice`, `project` ja `daq` käyttävät komentoa `CHLOROS_BACKEND_URL` (oletusarvoisesti `http://127.0.0.1:5000`). |
| `-v, --verbose` | Ota käyttöön yksityiskohtainen tulostus. |
| `--restart` | Pakottaa taustaprosessin uudelleenkäynnistyksen (lopettaa kaikki käynnissä olevat `backend_server.py`-prosessit). Systemd-palvelun (Linux .deb) ei koskaan pysähdy: komento tulostaa sen sijaan `sudo systemctl restart <unit>` ja lopettaa `1`. |
| `--version` | Tulosta versio (`Chloros CLI 1.3.0`). |
| `--help` | Näytä ylätason ohje. |

### Komentojen hakemisto

| Komento | Käyttötarkoitus |
| --- | --- |
| [`process`](#chloros-cli-process) | Käsittele kansio, joka sisältää Survey3- tai LATTICE-tallenteita, alusta loppuun. |
| [`login`](#chloros-cli-login) | Todentaa tämän laitteen Chloros+-tilillä. |
| [`logout`](#chloros-cli-logout) | Tyhjentää välimuistissa olevat tunnistetiedot. |
| [`status`](#chloros-cli-status) | Näytä nykyinen lisenssi- ja todennustila. |
| [`export-status`](#chloros-cli-export-status) | Näytä Thread-4-viennin reaaliaikainen eteneminen `process`-ajon aikana. |
| [`language`](#chloros-cli-language) | Aseta tai näytä CLI:n näyttökieli (38 tuettua kieltä). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Oletusprojektikansio (jaettu GUI:n kanssa). |
| [`update`](#chloros-cli-update) | Tarkista ja asenna Chloros-päivitykset (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Järjestelmän diagnostiikka + savutestit. |
| [`time-sync`](#chloros-cli-time-sync) | PTP-grandmaster-tila / hallinta. |
| [`lattice`](#chloros-cli-lattice) | LATTICE-kameran hallinta ja kuvaus (yli 45 alikomentoa). |
| [`daq`](#chloros-cli-daq) | DAQ-spektrisensorin ohjaus (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Avaa ja ohjaa tallennettua Chloros-projektia (kamerat + DAQ-laitteet). |

---

## Asennus

`chloros-cli` toimitetaan Chloros-työpöytäasennusohjelman mukana kaikilla tuetuilla alustoilla — erillistä CLI-latausta ei ole. Alustapaketin asentaminen lisää `chloros-cli`:n `PATH`:ään työpöytäsovelluksen ja sen ohjaaman taustaprosessin rinnalle.

Uusimmat lataukset: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Asennusohjelma sisältää myös käteviä käynnistysskriptejä (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`), jotka avaavat käyttövalmiin CLI-komentotulkin.

### Windows (.exe)

1. Lataa Windows-asennusohjelma lataussivulta.
2. Suorita `Chloros Installer 1.3.0.exe` ja seuraa ohjatun asennuksen ohjeita. Oletusasennuskansio on `C:\Program Files\Chloros\` (CLI-komentotulkki sijaitsee kansiossa `C:\Program Files\Chloros\cli\`, jonka asennusohjelma lisää PATH-muuttujaan).
3. Avaa uusi terminaali (`cmd.exe`, PowerShell tai Windows-terminaali), jotta päivitetty `PATH` tunnistetaan.

```powershell
chloros-cli --version
```

Asennusohjelma lisää automaattisesti `chloros-cli.exe` järjestelmän `PATH`-kansioon ja liittää mukaan LATTICE-kameran ajurit. Taustapalvelin on `C:\Program Files\Chloros\backend\chloros-backend.exe` (asennuskansion `backend`-kansio), jota ei ole `PATH`:ssä.

**Järjestelmäasetukset.** Asennusohjelman *Valitse lisätehtävät* -sivulla on kaksi vaihtoehtoa kohdassa **Järjestelmäasetukset**, jotka molemmat ovat**oletusarvoisesti valittuna**:

| Vaihtoehto (asennusohjelman sanamuodon mukaan) | Mitä se tekee | Hiljaisen asennuksen poistaminen käytöstä |
| --- | --- | --- |
| **Ota Windowsin pitkät polut käyttöön** (sallii projektikansioiden sijoittamisen yli 260 merkin syvyyteen; suositeltava) | Asettaa Windowsin koko tietokoneen kattavan `LongPathsEnabled`-asetuksen arvoksi 1. Windowsissa tämä asetus on oletusarvoisesti pois päältä, jolloin mikään polku ei saa ylittää 260 merkkiä; Chloros-tuotteet sijaitsevat noin 130 merkkiä projektikansion alapuolella, joten syvemmälle sijoitettua projektikansiota ei hyväksytä (katso [Vianmääritys](#vianmääritysvinkit)). Asetus astuu voimaan asennuksen jälkeen käynnistetyissä ohjelmissa. Jätetään päälle asennuksen poistamisen yhteydessä, koska muut ohjelmat saattavat tarvita sitä. | `/MERGETASKS="!longpaths"` |
| **Chloros-taustapalvelu suljetaan pois Microsoft Defenderin tarkistuksesta**(paljon nopeampi kameran tallennus; suositeltavaa) | Lisää Defenderin**prosessi**-poissulkemisen `chloros-backend.exe`:lle asennuskansioon, joten reaaliaikainen suojaus ei tarkista jokaista tiedostoa, jonka taustapalvelu kirjoittaa, riippumatta siitä, mihin se ne kirjoittaa. Mitään muuta ohjelmaa ei suljeta pois; poissulkeminen poistetaan asennuksen poistamisen yhteydessä. Jos Defenderia ei ole tai se on käytäntöjen vuoksi lukittu, vaihe epäonnistuu ilman ilmoitusta ja asennus jatkuu. | `/MERGETASKS="!defenderexclusion"` |**Miksi Defender-vaihtoehto on tärkeä:** suurilla tallennusnopeuksilla tallennustiedostojen reaaliaikainen tarkistus vie noin 30 % tallennusnopeudesta (mitattu 9 kameran jatkuvassa tallennuksessa: 77,7 tallennusta/s suljettu pois, 52,7 skannattu). `lattice array-capture --continuous` tulostaa ilmoituksen, kun se mittaa Defenderin käyttävän yhtä tai useampaa CPU-ydintä tallennuksen aikana; suorita asennusohjelma uudelleen, kun vaihtoehto on valittuna.

Hiljainen asennus (`/SILENT` tai `/VERYSILENT`) ottaa molemmat käyttöön, ellei niitä ole poistettu käytöstä, esim. `"Chloros Installer 1.3.0.exe" /VERYSILENT /MERGETASKS="!defenderexclusion"`.

**Windowsin palomuuri.** Asennusohjelma lisää aina kaksi saapuvaa sääntöä `chloros-backend.exe`:lle jokaiseen verkkoprofiiliin, ja asennuksen poistaja poistaa ne: `Chloros Backend (UDP-In)` kameravirroille, PTP:lle, mDNS:ää ja DAQ-E-datavirtaa, sekä `Chloros Backend (TCP-In)`, joka on rajoitettu paikalliseen aliverkkoon ja tarkoitettu DAQ-E-laiteohjelmiston päivitystä varten, jolloin anturi muodostaa yhteyden takaisin tähän tietokoneeseen kuvan keräämistä varten (katso [DAQ-E-laiteasetukset](#daq-e-device-settings)). Taustapalvelimen HTTP-API pysyy sidottuna `127.0.0.1`:ään.

### Linux amd64 (.deb)

Ubuntu 22.04 LTS:lle tai uudemmalle / Debian-pohjaisille x86_64-työasemille.

> **Ubuntu 20.04:ää ei tueta.** Paketin riippuvuuslista perustuu siihen,
> mihin taustakoodi tosiasiallisesti linkittyy, ja siihen sisältyy `libc6 (>= 2.35)`;
> focal toimittaa glibc 2.31:n. `apt` estää asennuksen sen sijaan, että antaisi sen epäonnistua
> suorituksen aikana. Luettelossa mainitaan myös `libgl1`, jonka mukana toimitettu OpenCV lataa ja jota
> minimaalisesta palvelinversiosta saattaa puuttua; `apt install` sisältää sen.

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
```

.deb-paketti asentaa:
- `chloros-cli`:n, `/usr/bin/chloros-cli`:n ja `chloros-backend`:n (käynnistää taustaprosessin) versioon `/usr/bin/chloros-backend`
- Käännetty taustaprosessi ja CLI, purettuna, tiedostoihin `/usr/lib/chloros/backend/` ja `/usr/lib/chloros/cli/`
- LATTICE-kameran ajurit
- Kohinanpoistomallit, kalibrointipaketit ja päivityskanavan asetukset
- Järjestelmän Python-SDK: `python3` (`import chloros_sdk`). `apt install` sisältää myös omat vaatimuksensa, `python3-numpy` ja `python3-requests` (suositeltava, ei pakollinen: yllä mainitut ohjelmat ovat itsenäisiä; `dpkg -i` ohittaa ne, ja asennuksen tuloste ilmoittaa puuttuvat osat). Virtualenv-ympäristössä: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`
- `/etc/sysctl.d/60-chloros-gige.conf`, joka nostaa ytimen kameran virtojen socket-vastaanottopuskureita 32 Mt:iin ja ottaa ne käyttöön kerralla (jos arvo on jo asetettu suuremmaksi, sitä ei muuteta). Paketin poistaminen poistaa tiedoston. Katso [Linux-isäntäkoneet — socket-vastaanottopuskureita](#linux-hosts--socket-receive-buffers).
- Kaksi valinnaista systemd-palvelua taustapalvelimelle, joista kumpikaan ei ole käytössä (seuraava osa)

#### Taustapalvelimen ajaminen palveluna (ilman käyttöliittymää)

Ilman palvelua taustapalvelin (ja sen mukana PTP-grandmaster) toimii vain niin kauan kuin jokin on sen käynnistänyt: `chloros-backend` terminaalissa, SDK-skriptissä tai ydin-`chloros-cli`-komennolla sen suorituksen keston ajan. Jotta taustapalvelu olisi aina päällä päättömällä koneella, ota palvelu käyttöön **omalla käyttäjätunnuksellasi**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` suorittaa taustapalvelua kyseisenä käyttäjänä. Sen kirjoittamat tallenteet kuuluvat kyseiselle käyttäjälle, ja sen yksityiskohtainen loki löytyy kyseisen käyttäjän `~/.cache/chloros/logs`-tiedostosta, ja se käyttää kyseisen käyttäjän `chloros-cli login`-tunnusta (`~/.chloros`), joten kirjaudu ensin kerran kyseisenä käyttäjänä. PTP ei vaadi root-oikeuksia. Kuten kaikki asennetut taustapalvelut, se kuuntelee vain porttia `127.0.0.1:5000`.

`chloros-backend.service` suorittaa taustapalvelimen **root**-käyttäjänä. Se on edelleen asennettuna, jotta kone, jolla se on otettu käyttöön, pysyy käynnissä, mutta kaikki, mitä root-taustapalvelu kirjoittaa, kuuluu rootille, joten käyttäjä ei voi poistaa omia tallenteitaan. Vaihtaaksesi omaan käyttäjätunnukseesi:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

Root-palvelun yksityiskohtainen loki on tiedostossa `/var/log/chloros`, jota `adm`-ryhmän jäsenet voivat lukea (palvelu asettaa `CHLOROS_LOG_DIR`; katso [Ympäristömuuttujat](#environment-variables)). Paketin asennus tulostaa komennon `enable`, johon on täytetty käyttäjänimesi, sekä vaihdon, kun root-palvelu on käytössä.

**Ota yksi taustayksikkö käyttöön.** Kerrallaan toimii vain yksi taustayksikkö. Toinen käytössä oleva yksikkö (`chloros-backend.service` `chloros-backend@<user>.service`:n lisäksi tai kahden käyttäjän `@`-yksiköt) ei käynnisty ja pysyy tilassa `failed` poistumistilalla 98; systemd ei käynnistä sitä uudelleen. Sen `systemctl status` kertoo syyn: `Another instance is already running. Exiting.`, sitten mikä pitää taustapalvelinta kiinni (portti 5000 tai lukitustiedosto ja käyttäjä, jolle se kuuluu), sitten `systemd will not restart <unit> (exit status 98)`. Uudelleenkäynnistyksen jälkeen kumpi tahansa yksikkö voi olla se, joka toimii. Vaihtaaksesi, poista käytöstä se, jota et käytä, kuten yllä.**CLI:n kautta ei koskaan pysäytetä palvelun taustaprosessia.** `chloros-cli` käyttää sitä sellaisenaan, kuten myös graafinen käyttöliittymä ja SDK: `process` toimii sen päällä ja lataa oman projektinsa edellisen tilalle. Siinä missä CLI:n pitäisi muuten käynnistää taustapalvelu uudelleen (`--restart` tai `process`, jonka tuonti ei löytänyt mitään), se nimeää yksikön, tulostaa `sudo systemctl restart <unit>` ja lopettaa `1`:n, koska tällä tavalla pysäytetty taustapalvelu pysyisi pois käytöstä. `GET /api/status` nimeää yksikön `service`:ksi (`null`, jos taustapalvelussa ei ole käynnissä yhtään palvelua).**Päivitys ja poisto.** Paketin päivittäminen pysäyttää kaikki taustapalvelut ja käynnistää sitten uudelleen kaikki käytössä olevat palvelut, ensin yksiköt `@<user>` (käsin käynnistetty mutta koskaan käyttöön otettu yksikkö pysyy pysäytettynä, kuten uudelleenkäynnistyksen jälkeen). Jos useampi kuin yksi on käytössä, vain yksi toimii ja muut pysyvät epäonnistuneina kuten edellä; päivitys tulostaa virheilmoitukset `Chloros: N backend units are enabled: …` ja `Keep one enabled: sudo systemctl disable --now <unit>`. Poistaminen pysäyttää ja poistaa ne käytöstä. Sen tyhjentäminen poistaa myös `/var/log/chloros`:n.

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Sama rakenne kuin amd64 .deb-tiedostossa, sisältää vastaanottopuskurit ja [taustapalvelut](#running-the-backend-as-a-service-headless), tarkoitettu Jetson Orinille / Orin NX:lle / Orin Nanolle **JetPack 6**:ssa (L4T R36). Sen PyTorch on JetPack 6:n CUDA-rakennus Orinille, joten GPU-käsittely, mukaan lukien tekstuurit tunnistava debayer, suoritetaan Jetsonin GPU:lla. Paketti ei sisällä itse CUDAa: se käyttää CUDA- ja cuDNN-kirjastoja, jotka JetPack 6 asentaa (`sudo apt install nvidia-jetpack` kuvassa, josta ne puuttuvat). Jos CUDA-GPU:ta ei ole käytettävissä, käsittely tapahtuu CPU:lla ja tekstuuritietoinen käsittely käyttää tavallista debayeria (katso [`process`](#chloros-cli-process)).

### Todennus kerran kutakin laitetta kohti

Jokaisella alustalla vaaditaan kertaluonteinen Chloros+-kirjautuminen, ennen kuin SDK- ja CLI-kutsut toimivat:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Kirjautumistiedot tallennetaan välimuistiin tiedostossa `~/.chloros/user_session.json`.

### Käynnistä taustapalvelu

Komennot `lattice`, `project` ja `daq` kommunikoivat jo käynnissä olevan taustapalvelimen kanssa; ne eivät koskaan käynnistä sitä (katso [Käytännöt](#conventions)). Pidä yksi käynnissä, kun käytät niitä:

| Alusta | Miten |
| --- | --- |
| Windows | Avaa Chloros-työpöytäsovellus (sen taustapalvelu palvelee myös CLI:tä) tai suorita komento `"C:\Program Files\Chloros\backend\chloros-backend.exe"` omassa terminaalissaan (asennuskansion `backend`-kansiossa) ja jätä se auki. |
| Linux / Jetson | Suorita `chloros-backend` omassa terminaalissaan tai pidä yksi aina käynnissä [taustapalvelun](#running-the-backend-as-a-service-headless) avulla. |

Kerrallaan on käynnissä vain yksi taustapalvelu; kaikki asiakasohjelmat (GUI, CLI, SDK-skriptit) jakavat sen. Myös ydinkomennot (`process`, `login`, `status`, …) käyttävät sitä ja käynnistävät sen, jos sitä ei ole käynnissä.

**Backend pysyy käynnissä komentojen välillä**, joten seuraava komento ei käynnistä sitä uudelleen. Se pysähtyy, kun kirjaudut ulos (`chloros-cli logout`, ellei Chloros-sovellusta ole auki tai ajon tai offline-rakennuksen ollessa käynnissä), kun suoritat komennon `chloros-cli backend stop`, tai itsestään 30 minuutin kuluttua, jos mikään ei käytä sitä: ei komentoja, skriptejä tai sovelluksia, ei ajotilannetta tai offline-rakennusta, ei kytkettyjä kameroita tai valosensoreita. `backend stop` kieltäytyy, kun ajotilanne tai offline-rakennus on käynnissä, ja ilmoittaa sen; `--force` pysäyttää sen välittömästi, jolloin kyseinen ajotilanne tai rakennus menetetään (sarja säilytetään, ja `array-build-video` rakentaa sen uudelleen). Aseta `CHLOROS_BACKEND_IDLE_MIN` ennen kuin taustapalvelu alkaa muuttaa 30 minuutin aikarajaa (`0`: ei koskaan); [taustapalvelu](#running-the-backend-as-a-service-headless) ei koskaan pysähdy itsestään. `chloros-cli backend status` näyttää, onko jokin käynnissä, mikä sitä käyttää ja onko offline-rakennus käynnissä. `process` avaa projektin käynnissä olevassa taustapalvelimessa, ellei Chloros-sovellusta ole avattuna projektin kanssa, jonka se avannut: tällöin se kieltäytyy, ja sinun on joko käsiteltävä asia sovelluksesta tai suljettava kyseinen projekti ensin. Projekti, jonka aiempi `process` tai SDK-skripti on jättänyt auki, korvataan yksinkertaisesti.

### Varmista asennus

```bash
chloros-cli --version           # prints "Chloros CLI 1.3.0"
chloros-cli selftest            # 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

`selftest` sulkeutuu (`0`) vasta, kun kaikki seitsemän tarkistusta on läpäisty. Viimeinen tarkistus vaatii CUDA-GPU:n (NVIDIA-GPU:n, tai Jetson Orin) sekä kohinanpoistomallit, joten tietokoneella, jossa sellaista ei ole, se näyttää virheen `Failed: CUDA + Denoiser` ja lopettaa ohjelman `1`; kaikki paitsi pelkästään GPU:ta käyttävä tekstuuritietoinen debayer toimii siellä edelleen.

> **Chloros+-tilaus vaaditaan.**CLI vaatii voimassa olevan Chloros+-tilauksen.**Copper**on Chloros+:n perustaso — jokaisella maksullisella Chloros+-tasolla on CLI/SDK-käyttöoikeus; ainoastaan ilmaisella**Iron**-tasolla ei ole. (Tilaus-ID-vastaavuus: `0`=Iron/free, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Päivitä osoitteessa [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Tätä alarajaa valvoo taustajärjestelmä, ei pelkästään CLI: SDK- tai CLI-lipulla varustettu pyyntö, jolla ei ole maksettua tilausta, hylätään virhekoodilla `403 PLAN_UPGRADE_REQUIRED` riippumatta siitä, tuleeko se `chloros-cli`:sta, Python-SDK:sta vai itse kehitetyltä HTTP-asiakasohjelmalta. Uloskirjautunut kutsuja saa sen sijaan virhekoodin `401 AUTH_REQUIRED`. Käyttö toimii offline-tilassa tilauksen armonaikana (kuukausitilauksessa 30 päivää, vuositilauksessa voimassaolon loppuun asti) ja lakkaa, kun se päättyy; `chloros-cli status` toimii edelleen, jotta syy on näkyvissä (se on ainoa SDK-/CLI-reitti, joka on vapautettu tasorajoituksesta — `GET /api/license-status`).

---

## Viiden minuutin pikaopas

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI GNDVI OSAVI

# 3. The lattice / daq commands need a running backend: on Windows open the
#    Chloros desktop app; on Linux start one (or see "Start the Backend")
chloros-backend &

# 4. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 5. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 6. Read a spectrum from a connected DAQ-U (Windows: --port COM3)
chloros-cli daq pool-connect --port /dev/ttyUSB0
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

`process` (vaihe 2) käynnistää oman ajonsa taustapalvelimen, jos sellaista ei ole käynnissä, ja pysäyttää sen, kun se on valmis, joten käynnistä yksi vaiheesta 4 eteenpäin. Katso [Taustaprosessin käynnistäminen](#start-the-backend).

---

## `chloros-cli process`

Käsittele kuvakansio koko Chloros-prosessin läpi (kohteen tunnistus → kalibrointi → vignetti → heijastavuus → indeksin vienti).

### Yhteenveto

```
chloros-cli process INPUT [OPTIONS]
```

### Sijaintiparametrit

| Parametri | Kuvaus |
| --- | --- |
| `INPUT` | Polku syöttökansioon, joka sisältää `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) tai `.dng`-tiedostoja. |

### Yleiset asetukset

| Asetus | Oletus | Kuvaus |
| --- | --- | --- |
| `-o, --output PATH` | uusi kansio, jonka nimi on `-n`, oletusprojektikansiossa | Projektikansio. Projekti nimetään sen mukaan, ja kansio, jossa on jo tiedosto nimeltä `project.json`, saa tiedoston `_1` / `_2`-sisarkansion sen sijaan, että se korvattaisiin. Oletusprojektikansio on se, jonka `set-project-folder` (tai käyttöliittymä) on valinnut, muuten `Documents/MAPIR/Chloros Projects`. |
| `-n, --project-name NAME` | auto (aikaleima) | Projektin nimi, käytetään vain **ilman** `-o`:ää (kun `-o` on mukana, projekti saa kansion nimen). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` käyttää Chloros+ -neuraalista debayeria Survey3-kuvissa; hitaampi mutta laadukkaampi. Se on tarkoitettu vain Survey3:lle: LATTICE-kuvissa käytetään aina vakiomuotoista demosaicia, joten kansiossa, joka sisältää LATTICE-kuvia, `process` hylkää `texture-aware`:n tuonnin jälkeen virheilmoituksella, jossa mainitaan kuvien lukumäärä, ja lopettaa `1` ennen minkään käsittelyä (Chloros-sovellus ei tarjoa Texture Aware -toimintoa myöskään näille kuville). Se vaatii CUDA-grafiikkaprosessorin (NVIDIA-grafiikkaprosessorin Windowsissa / Linuxissa x64, Orinin grafiikkaprosessorin Jetsonissa): ilman sitä suoritus käyttää `standard`:ää, ja taustaprosessi kirjaa yhden `[DEBAYER] … needs a CUDA GPU …`-rivin, jossa kerrotaan syy (esitetty `--verbose`:llä). |
| `--vignette / --no-vignette` | `--vignette` | Vignettikorjaus. |
| `--reflectance / --no-reflectance` | `--reflectance` | Heijastavuuden kalibrointi (käyttää paneelikohdetta, jos sellainen löytyy, tai LATTICE-laitteille NIST:n sarjanumeroittaista kalibrointia). LATTICE-monispektrikuvissa tämä toimii myös heijastavuuden **tulemukseksi** — katso [Tuotekohtaiset vientikytkimet](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | pois | Sovella PPK-GNSS-korjauksia sivutiedostoista. |
| `--exposure-pin-1 CAMERA_MODEL` | ei mitään | Valotusnastaan 1 kytketty kameramalli, PPK-aikasynkronointia varten. |
| `--exposure-pin-2 CAMERA_MODEL` | none | Valotusnastaan 2 kytketty kameramalli PPK-aikasynkronointia varten. |
| `--recal-interval SECONDS` | `0` | Vähimmäisrekalibrointiväli sekunteina (projektin *Vähimmäisrekalibrointiväli*). |
| `--timezone-offset HOURS` | `0` | Valosensorin aikavyöhykkeen poikkeama UTC:stä, tunteina (projektin *Valosensorin aikavyöhykkeen poikkeama*). |
| `--format FORMAT` | `TIFF (16-bit)` | Yksi seuraavista: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros ei koskaan nosta kuvan bittisyvyyttä: kaikissa 8-bittisissä kuvissa syvempi formaatti tallennetaan nimellä `TIFF (8-bit)`, ja tämä mainitaan kerran suorituksen aikana. Projektissa, jossa sekoitetaan 8-bittisiä ja 12/16-bittisiä kuvia, kukin kuva viedään omalla bittisyvyydellään tai sitä alemmalla. Radiance on aina 32-bittinen liukuluku. |
| `--indices NAME [NAME ...]` | ei mitään | Kasvillisuusindeksit esiasetuksen nimen mukaan (`NDVI`, `GNDVI`, `OSAVI`, `SAVI`, `EVI`, `GCI`, …), kirjainkokoa ei huomioida. Kukin indeksi lisätään projektiin kyseisen kameran kaistojen perusteella, mukaan lukien uusi projekti. Nimi, joka ei ole esiasetus tai jota kameran kaistat eivät pysty muodostamaan (esim. `NDRE` vaatii RE- ja NIR-kaistoja, joita yksikään kamera ei tarjoa), mainitaan varoituksessa eikä sille luoda hakemistokuvaa. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Pakottaa LATTICE TIFF -tiedostojen käsittelyputken aloituskohdan (ei vaikuta Survey3.raw-tiedostoihin). Myös pakotie, jonka avulla **raakakuvattomia** kuvia voidaan käsitellä lainkaan — katso [Miltä kuvauskansio näyttää](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | päällä | Lähettää lineaarisen debayeroidun tuloksen (`Debayered_Images`). Katso [Tuotteittaiset vientikytkimet](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | päällä | Lähettää näytön esikatselun (`Preview_Images`): RGB = valkotasapaino (DAQ-valonlähde, jos saatavilla, muuten harmaa maailma) + gamma; multispek = väärävärinen venytys. |
| `--radiance / --no-radiance` | päällä | Lähettää float32-tyyppisen säteilyvoimakkuuden (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Viite LATTICE-heijastavuustuotteelle: `auto` = laadunvarmistuksen läpäissyt kehyksen sisäinen tavoitearvo on absoluuttinen viite (tavoitearvo, jolla on laadunvarmistuslippu, kilpailee mitatun pistemäärän suhteen DAQ:n kanssa), DAQ-alasvirtauksen (ρ = π·L/E) varavaihtoehto; `target` = tiukka (ei DAQ-korvausta); `daq` = DAQ-määräysvaltainen. Katso [Tuotekohtaiset vientikytkimet](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | none | Yksikkökohtaisten **mitattujen** kohdeheijastavuusskannausten hakemisto (`<serial>.csv`); käyttää nimellisiä T3/T4P-spektrejä, jos mittausta ei ole saatavilla. |
| `--array-alignment / --no-array-alignment` | päällä | LATTICE-matriisit: sovelletaan jokaisen kuvan `Chloros:Alignment*` XMP-tiedostoon merkittyä moduulien välistä kohdistusta kaikkiin käsiteltyihin tuotteisiin (debayering / esikatselu / säteilyvoimakkuus / heijastavuus / indeksi). Ei vaikutusta kuviin, joissa tunnisteita ei ole. |
| `--array-alignment-crop / --no-array-alignment-crop` | rajaus | Rajaa kohdistetut vientitiedostot matriisin yhteiseen päällekkäisalueeseen, jotta kaikki moduulit jakavat yhden peittoalueen; `--no-…` säilyttää koko anturin kuvapinnan (musta täyttö lähteen ulkopuolella). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Uudelleennäytteenotto kohdistuksen vääristymän korjaamiseksi. `nearest` säilyttää tarkat lähteen digitaaliset numerot (DN:t) (ei radiometristen arvojen sekoittumista pikselien välillä). |

### Kohteen tunnistuksen asetukset

| Asetus | Kuvaus |
| --- | --- |
| `--min-target-size PIXELS` | Ilmaisimen kalibrointikohteen vähimmäiskoko pikseleinä. |
| `--target-clustering 0-100` | Kohteiden ryhmittelyprosentti (oletus `60`). |
| `--target / --targets` | Etsi kalibrointikohteita vain syötteen `target`- tai `targets`-alikansiosta (nopeampi). |

### Esimerkkejä

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI GNDVI OSAVI

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

# Save to a project folder of your choice (the project is named after it)
chloros-cli process "C:/input" -o "C:/projects/Field_A_2026-05-26"

# Name the project and let it go to the default project folder
chloros-cli process "C:/input" -n "Field_A_2026-05-26"
```

### Tuotekohtaiset vientikytkimet (LATTICE-monispektrinen)

LATTICE-käsittely tuottaa **kaikki soveltuvat tuotteet yhdellä kertaa**. Neljä tyyppikohtaista kytkintä — `--debayered`, `--preview`, `--radiance`, `--reflectance` — ovat kaikki**oletusarvoisesti päällä**; käytä lomaketta `--no-<type>`, jos haluat poistaa yhden. RGB-pääkamerat lähettävät aina vain debayeroitua kuvaa + esikatselua (ei kaistakohtaista säteilyä/heijastavuutta), joten `--radiance`/`--reflectance` eivät vaikuta niihin. Kytkimet ohitetaan Survey3:n `.raw`:n kohdalla (joka noudattaa standardin mukaista heijastavuus-/kohdepolkua). *(Vanha `--radiometric-output {reflectance,radiance,sensor-response}`-lippua **poistettiin** ja korvattiin näillä kytkimillä; `sensor-response`-tasoa ei enää ole.)*

| Tuote | Lähtö | Tarvitaanko DAQ-alasvirtausta? |
| --- | --- | --- |
| `--debayered` | Lineaarinen demosaic (`Debayered_Images`): sama tiedosto, jonka tallennus tallentaa `debayered`-tasollaan — kalibrointipaketin tumman offsetin (DSNU) poisto raakadatasta, sitten OpenCV:n reuna-herkkä demosaikki; ei lineaarisuus-LUT:ta, tasokenttää tai sekoituksen purkua. | Ei. |
| `--preview` | Esikatselu näytöllä (`Preview_Images`): RGB = valkotasapaino + gamma; monispektrinen = väärävärinen venytys. | Ei. |
| `--radiance` | float32 W/m²/sr/nm koko radiometrisestä ketjusta (`Radiance_Images`). | Nro |
| `--reflectance` | uint16-heijastavuus ρ (`32768` = 1,0), Pix4D-valmis. | **Kyllä**, ellei sitä ankkuroi laadunvarmistuksen läpäissyt kehyksessä oleva kohde (katso alla). |

`--reflectance-source` valitsee heijastavuusviitteen:**`auto`**(oletus) tekee laadunvarmistuksen läpäisseestä kehyksessä olevasta kohteesta**absoluuttisen viitteen**— kohteeseen ankkuroidut empiiriset linjaketjut verrataan testipaneeleihin ja mitattu voittaja sovelletaan kaistakohtaisesti: kaista, jota DAQ ei voi kalibroida tai jonka DAQ-kaltevuussovitus epäonnistuu, ottaa kohteen oman linjan (`ReflectanceElBandMethods` määrittää, mikä) — kohde, joka läpäisee laadunvarmistuksen lipun kanssa (paneeli pudonnut, kaista epäkunnossa), kilpailee myös DAQ:n jakolinjan kanssa kyseisellä mitatulla pisteytyksellä, ja kehys, jolle se häviää, merkitään `ReflectanceChain=daq` — palataan DAQ:n alaspäin suuntautuvaan jakorajaan (ρ = π·L/E), kun kohdetta ei ole läsnä tai laadunvarmistus epäonnistuu;**`target`**on tiukka (ei DAQ-korvausta);**`daq`**siirtyy DAQ:n määräävään käyttäytymiseen. Kohteen geometria (ArUco / kiinteä ROI / kaistale) tulee projektin kohdekonfiguraatiosta; `--target-reflectance-dir DIR` säilyttää yksikkökohtaiset**mitatut** skannaukset (`<serial>.csv`), jotka haetaan kohdeyksikön sarjanumeron/QR-koodin perusteella, ja nimelliset T3/T4P-spektrit toimivat varavaihtoehtona.

DAQ-heijastuspolku määrittää **aikaleimalla täsmätyn alaspäin suuntautuvan säteilyn**automaattisesti tallennetusta**`.daq`**-tiedostosta (DAQ-U/M/E)**tai kuvamateriaalin yhteydessä löytyvästä DAQ-M:n omasta `.csv`-tiedostosta**. Jos kamera- tai DAQ-kalibrointipakettia ei ole tallennettu paikalliseen välimuistiin, putkisto**hakee sen automaattisesti AWS:stä** ensimmäisellä käyttökerralla (vaatii internet-yhteyden kerran; tallennetaan välimuistiin tiedostona `~/.chloros/`).**Kenen lukema se on.**`.daq`, jonka tallennus tallentaa, on sen kuvaaman kameran tai kameroiden lukema — nimetty sen vieressä olevassa `<file>.daq.cameras.json`-sidecar-tiedostossa (`lattice capture`, `multi-capture`, `array-capture` ja käyttöliittymä; raakasarja nimeää jokaisen jäsenen lukeman manifestissaan) — ja `process` yhdistää sen**vain kyseisten kameroiden kuviin**. Itsenäinen kamera, jolla ei ole omaa lukemaa, ei siis enää lainaa toisen kameran DAQ:ta; liitä siihen DAQ tai tallenna sellainen. DAQ:n oma tallenne (`daq pool-record` tai käyttöliittymässä aloitettu tallennus) tai mikä tahansa `.daq` ilman kyseistä sivutiedostoa (mukaan lukien vanhemmat tallenteet) palvelee edelleen kaikkia kameroita.

#### Heijastuspikseleiden lukeminen (Pix4D / Metashape / omat skriptisi)

Heijastusarvo tallennetaan kokonaislukuna DN, ja **se DN-arvo, joka tarkoittaa ρ = 1,0, riippuu lähdekamerasta**:

| Lähde | ρ = 1,0 on | Miten selvittää |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (varaa ρ 2,0:aan asti) | XMP-tiedostoon on merkitty `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (rajoitettu arvoon ρ 1,0) | Ei `Chloros:*` XMP-tunnisteita — niiden puuttuminen *on* merkki. |

**Lue `Chloros:PixelScale` ja jaa sillä** sen sijaan, että olettaisit sen olevan vakio. Tunniste on määritelty uint16-alueella, joten se pysyy samana `32768` kaikissa ulostulomuodoissa, joissa suoritetaan skaalaus — `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)` ja `TIFF (32-bit, Percent)` ovat kaikki itsestään selviä (normaloi tallennettu tietotyyppi ensin takaisin uint16:ksi: ×257 8-bittisestä, ×65535 float-tyypistä).

> **Vanhemmista versioista peräisin olevissa tiedostoissa ei voi olla mittakaavaa.** 8-bittinen heijastusTIFF-tiedosto skaalataan uudelleen ja merkitään nimellä `Chloros:PixelScale` muiden tavoin. Versioissa ennen 1.3.0 *rajasivat* sen sen sijaan arvoalueelle 0..255, joten jokainen arvo, joka oli suurempi kuin ρ≈0,008, tasoitettiin arvoon 255, ja ne jättivät pois sekä `Chloros:PixelScale`- että `MicaSense:RadiometricCalibration`-tupelin. **Jos tunniste puuttuu LATTICE-heijastustiedostosta, älä oleta mittakaavaa — käsittele se uudelleen** sen sijaan, että jakaisit pikseleitä, joita ei ole koskaan voitu jakaa.

#### Indeksipikselien lukeminen (NDVI ja muut kasvillisuusindeksit)

**Kokonaislukuiset indeksirasterit ovat kuvakohtaisia kontrastin venytyksiä, eivät −1…1-karttoja.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` ja `JPG (8-bit)` -indeksituotteet – mukaan lukien 8-bittinen TIFF-tiedosto, johon 8-bittinen tallenne tallennetaan – tallentavat

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

jossa `IndexMin` / `IndexMax` ovat **kyseisen kehyksen omat pienin ja suurin äärelliset indeksiarvot**. DN 0 ei ole −1, eikä täysi asteikko ole +1.

Chloros merkitsee molemmat päätepisteet, jotta kuvaus pysyy käännettävissä:

| Tunniste | Merkitys |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — alla oleva käytäntö |
| `Chloros:IndexMin` | indeksiarvo, joka on tallennettu DN 0:na |
| `Chloros:IndexMax` | indeksiarvo, joka on tallennettu täysasteikolla |

Lue ne ja käännä ne ympäri, normalisoimalla tallennettu DN ensin tallennetun dtype-tyypin maksimiarvoon 0…1, jotta yksi lauseke kattaa kaikki kokonaislukumuodot:

```python
import json, subprocess
import numpy as np, tifffile

tags = json.loads(subprocess.run(
    ["exiftool", "-j", "-XMP:all", path],
    capture_output=True, text=True).stdout)[0]

# float(): exiftool hands back a negative real as a quoted string, so
# IndexMin arrives as str on most real NDVI data while IndexMax is a number.
lo, hi = float(tags["IndexMin"]), float(tags["IndexMax"])

raw = tifffile.imread(path)                    # uint16 for TIFF (16-bit)
ndvi = lo + raw.astype(np.float64) / np.iinfo(raw.dtype).max * (hi - lo)
```

Lukemiseen ei tarvita `-config pix4d.config`:ää — exiftool lukee nämä XMP-ominaisuudet ilman apua. Määritys tarvitaan vain niiden *kirjoittamiseen*.

> **`IndexMin == IndexMax`** tarkoittaa, että kyseisen kehyksen jokaisella pikselillä oli sama indeksiarvo. Rasteri on täytetty keskiasteikolla, eikä ole mitään, millä jakaa — ota päätepiste.

> **`Chloros:Index*`-tunnisteiden puuttuminen tarkoittaa, ettei kartoitusta ole määritelty.** LUT-vienti on värillinen renderointi, jonka pikselit ovat LUT-kynnysikkunaan sidottuja gradienttivärejä, eikä kehyksellä, jossa ei ole äärellistä indeksipikseliä, ole määriteltävää aluetta. Älä oleta kartoitusta kummassakaan tapauksessa — sama sääntö kuin puuttuvan `PixelScale`-tunnisteen kohdalla.**Koska venytys tapahtuu kuva kerrallaan, yhden lennon kahta kuvaa ei voi verrata toisiinsa DN-arvojen perusteella**— kummallakin on omat päätepisteensä, joten sama NDVI-arvo sijoittuu eri kuvissa eri DN-arvoihin. Lennon laajuista kvantitatiivista työtä varten (vyöhykekohtaiset tilastot, kiinteät kynnysarvot, aikasarjat) vie**`TIFF (32-bit, Percent)`**: indeksirasterit tallennetaan sinne float32-muotoisina**indeksiarvoina**, ilman skaalausta tai tarvittavia tunnisteita. Muodosta huolimatta indeksituotteet eivät ole ×100 — kyseinen skaalaus koskee vain kokonaislukutyyppistä heijastavuutta.

#### Unmix-rajoitusmaski (mitkä pikselit on rajoitettu tietyllä kaistalla)

3×3-spektrinen unmix-laskenta voi aiheuttaa kaistan negatiivisen arvon — jos valossa on tuskin kyseistä kaistaa tai jos kohde sijaitsee kameran unmix-matriisin kuvaaman alueen ulkopuolella. Se ei ole todellinen arvo, joten vienti asettaa sen arvoon 0 – ja reaaliaikaisessa näkymässä näkyy sama 0. Kaista voidaan rajoittaa muutamaan pikseliin tai lähes kaikkiin pikseliin; maski tallentaa kummankin tapauksen. `Chloros:Band{R,G,B}UnmixClampFraction` kertoo, kuinka suuri osa kehyksestä oli kyseessä; rajoitusmaski kertoo **mitkä pikselit**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— `clamp_masks/`-kansiossa**tuotteen kansion vieressä**, ei koskaan sen sisällä (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`; `process`-viennin `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`) — jokaiselle säteilyvoimakkuudelle, heijastavuus- ja indeksituotteelle, jossa vähintään yksi pikseli on rajattu: sekä tallennukset että `process`-vienti (sekä `array-build-video --save-tiffs`:n kehyskohtaiset TIFF-tiedostot). Yksikaistaiset `uint8`-tiedostot, tuotteen omalla pikseliristikolla (suunta, ROI, binning ja taulukon kohdistus mukaan lukien). Jokainen maski nimeää tuotteen kameran (`Make` / `Model` / sarjanumero), tallennusajan ja objektiivitunnisteet sekä luettelee kaistat, joita sen bitit edustavat (`XMP-Camera:BandName`, bitti 0 ensin); sen `ImageDescription`- ja `Software`-tunnisteet merkitsevät sen clamp-maskiksi, eikä se sisällä kalibrointia.
- **Bitit**: bitti 0 = tuotteen R-kanavan kaista (`Chloros:BandRName`), bitti 1 = G (`BandGName`), bitti 2 = B (`BandBName`). Asetettu bitti tarkoittaa, että kyseisen kaistan pikselin arvo on rajoitettu arvoon 0 — käsittele sitä ja siitä muodostettuja indeksejä luotettavuudeltaan heikkoina.
- **`Chloros:UnmixClampMask`** on merkitty jokaiseen tuotteeseen, johon on suoritettu unmix-käsittely: maskin polku suhteessa tuotteen omaan kansioon (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, aina `/`-erotettuna — liitä se kuvan kansioon avataksesi maskin), tai `none`, kun yhtään pikseliä ei ole rajoitettu. Jos tunnistetta ei ole, tuotteelle ei ole koskaan suoritettu unmix-käsittelyä (RGB- tai mustavalkokamerat, raakakuvat/debayeroidut tasot).
- `lattice capture` / `array-capture` tulostavat maskin polun tallennetun tiedoston alle (`clamp mask: …`), ja tallennustuloksissa se näkyy kehyksen `clamp_mask`-tiedostona.
- `lattice index --live --save-multiband` kirjoittaa yhden maskin kutakin lähdekameraa kohti monikaistapinoon — katso [`lattice index` — monikaistapinomaskit](#lattice-index--multiband-stack-masks). Videolla ei ole maskia: `array-build-video` raportoi sen sijaan kunkin kehyksen rajoitusosuudet (katso [`array-build-video`](#array-build-video--offline-reprocess-a-saved-burst)).

Koska missään tuotekansiossa ei ole maskia, missään syvyystasolla, tuotekansio (`radiance/`, `reflectance/`, `process`-vientiin kuuluva `*_Images/`-kansio) voidaan siirtää Pix4D:lle, Metashapelle tai muulle fotogrammetriaohjelmistolle sellaisenaan — alikansiot mukaan lukien — ja se hyväksyy vain kuvia. Siirrä sille tuotekansio, älä sen yläpuolella olevaa kansiota (jossa on myös `clamp_masks/` ja kaikki muut vientityypit). Chlorosin oma tuontitoiminto ohittaa naamiot. Naamion lukeminen: `(mask & 1) != 0` tarkoittaa ”R-kanavan arvoa on rajoitettu tässä kohdassa”, `& 2` G, `& 4` B.

#### EXIF-tiedot siirretään vientiin

`process` kopioi lähdekuvan **GPS-lohkon ja sen ExifIFD:n** jokaiseen tuotteeseen, joten
vienti sisältää `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` ja
`CameraSerialNumber` georeferenssin ohella.

**`FocalLength` ei ole valinnainen fotogrammetrian kannalta.** Pix4D laskee maanpinnan näytteenottovälin
polttovälistä ja korkeudesta; jos tunniste puuttuu, se käyttää oletusarvoisesti täysin väärää mittakaavaa. Eräässä
49 kuvan appelsiinitarhan lennossa puuttuva tunniste muutti 411 m × 160 m:n alueen rekonstruoiduksi
47,8 km × 13 km:n alueeksi – 455 MP:n ortokuvaksi, jossa oli pääosin nodata-arvoja, mikä tulkittiin aluksi laatoitusongelmaksi ja
BigTIFF-ongelmaksi, ennen kuin kukaan tarkisti GSD:n. Jos ortokuvasi tulostuu epäuskottavalla
mittakaavalla, aja ensin `exiftool -FocalLength` viedyn tuotteen päälle.

Kopio ei ole tarkoituksella **`-all:all`: IFD0&#x27;**:n

rakenteelliset tunnisteet rikkovat LATTICE-tulostetta, kun
ne kopioidaan, ja `ExifImageWidth` / `ExifImageHeight` on jätetty pois, koska ne kuvaavatrakennetunnisteet rikkovat LATTICE-tulostuksen, kun
ne kopioidaan, ja `ExifImageWidth` / `ExifImageHeight` on jätetty pois, koska ne kuvaavat
*lähde*kuvaa — muuten kokoa muutettu vienti sisältäisi mitat, jotka
, jotka ovat ristiriidassa sen oman rasterin kanssa. XMP-tiedosto kirjoitetaan suoraan sen sijaan, että se kopioitaisiin, koska ExifTool
hävittää samasta kutsusta peräisin olevat XMP-tunnisteet, kun XMP-lohko kopioidaan (mikä poistaisi MAPIR-
kalibrointitunnisteet).

### Mihin tulosteet tallennetaan

Tulostustiedostot tallennetaan **projektikansioon, ryhmiteltynä kameran ja sen jälkeen tiedostomuodon mukaan**:

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

LATTICE-kameran kansio on `LATT-<sensor>-<lens>-F<filter>` (vastaa kuvan EXIF-tunnisteita
`Model`) ja `<model>_<filter>` (Survey3) — kaksi kameraa, joilla on sama kenno ja suodatin mutta erilaiset
objektiivit, säilytetään erillisissä hakemistopuissa, koska vinjetointi, kuvakulma ja vääristymä eroavat toisistaan. Tiedostomuoto
on seuraava: `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` tai `tiff32`, kun taas
`TIFF (32-bit, Percent)`.

> **Jokainen viety tuote säilyttää LÄHDE-tiedoston nimen.** Säteilyvoimakkuuden vienti
> `capture_…_raw.tif`:stä on edelleen nimeltään `capture_…_raw.tif` — se vain sijaitsee
> `tiff32/Radiance_Images/`-kansiossa. **Kansio tunnistaa tuotteen, ei tiedostonimi**, joten
> `*radiance*.tif`-tiedoston etsiminen ei tuota tulosta; käytä sen sijaan hakukriteerinä kansiota.

### Valosensorin tallenteet — kalibroitu `.daq` + `.csv`

`process` käsittelee myös syöttökansiossasi olevat `.daq`-tallenteet, eikä se **tarvitse**
tarvitse mitään kuvamateriaalia: yksinään lennetty DAQ-U / DAQ-M / DAQ-E on täydellinen
tallennus, ja kansio, jossa on vain `.daq`-tiedostoja, on kelvollinen syöte.

DAQ-laite voidaan tallentaa **ilman** sen kalibrointia — juuri näin toimivat julkiset
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) tallentimet
(`record_daq.py`) tekevät oletuksena: ne kirjoittavat raakatiedot anturilukemista ja merkitsevät tiedostoon leiman, jotta
Chloros hakee kyseisen anturin tehdaskalibroinnin **sarjanumeron kautta** (ensin paikallisesta välimuistista,
sitten MAPIR-pilvestä) ja soveltaa sitä. `process` kirjoittaa tuloksen takaisin:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` sisältää yhden rivin kutakin lukemaa kohti: UTC-aikaleima, integraatioaika, kokonaisteho,
fotopinen/skotopinen lux, PPFD (ja sen sininen/vihreä/punainen jakautuma), huippuaallonpituus, sitten
koko spektri anturin omalla aallonpituusruudukolla. Tiedosto `.daq` tuodaan uudelleen ilman, että sitä
kalibroidaan toista kertaa.

`<name>` on tallenteen oma tiedostonimi. Kaksi eri kansioissa olevaa tallennetta voi jakaa
yhden — kahden kameran samana
millisekuntina tallentamat lukemat (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) — ja kumpikin saa silti omat tuotteensa: ensimmäinen säilyttää
`<name>`, seuraava lisää kansiot, jotka erottavat sen muista, **nimen jälkeen**
(`<name>__<capture>_daq_calibrated.daq`), joten tallenteen päivämäärä ja kellonaika ovat edelleen etusijalla.

Onnistuessaan suoritus raportoi `Light-sensor products written: N (calibrated .daq + .csv)`.
Suluissa oleva teksti kuvaa sitä, mitä tosiasiassa kirjoitettiin, joten se kuuluu
`(RAW COUNTS — this sensor has no calibration bundle)`, jos kyseessä on pakettiton anturi, ja
`(N calibrated, M raw counts)`, jos kyseessä on kansio, jossa on molempia. Backendin omat
otsikot `[DAQ-EXPORT]` ja `[RUN-SUMMARY]` muodostuvat samalla tavalla — mikään
näistä kolmesta ei voi kutsua raakaa vientiä kalibroiduksi.

DAQ-U / DAQ-M / DAQ-E-tallenne, jonka kalibrointipakettia ei voida hakea – olet
offline-tilassa tai kyseisellä anturilla ei ole kalibrointitietoja tiedostossa – **ohitetaan syyn kera**
`[DAQ-EXPORT]`-rivillä, eikä sitä koskaan tallenneta ”kalibroituna” tiedostona, joka sisältää raakalukemia.
Yhdistä internetiin ja suorita uudelleen. Syy on se, jonka lukija tosiasiallisesti
määritteli kyseiselle tiedostolle (lukukelvoton skeema, ei kalibrointipakettia, kirjoitusvirhe), ja ajon
yhteenveto luettelee **erilliset** syyt — kaksikymmentä ohitettua tiedostoa yhdestä syystä näkyy yhtenä
syyna, ei kaksikymmentä toistoa siitä. CLI nimeää jokaisen ohitetun tallennuksen ajon lopussa
ja ilmoittaa sen syyn:

```
⚠ Light-sensor recording not exported: <file>.daq: no calibration bundle available for …
```

Kansio, jossa ei ole **yhtään** vietyä tallennetta, epäonnistuu: katso
[Suoritus, joka ei kirjoita kuvia, epäonnistuu](#a-run-that-writes-no-images-fails).

Tallenne, jonka nimi päättyy `_calibrated` tai `_raw` (`field_raw.daq`), **ei
tuotu**: nämä ovat nimiä, joita `process` antaa omille tuotteilleen, ja tuote, joka luetaan takaisin
tallenteena, laskettaisiin kahdesti. CLI-käyttöliittymä ilmoittaa tämän ja nimeää tiedoston —
`Not imported as a light-sensor recording: field_raw.daq … Rename the file to import it.`
— sen sijaan, että ilmoittaisi tyhjästä kansiosta. Nimeä tällainen tallenne uudelleen ennen sen käsittelyä.

#### DAQ-A-tallenteet viedään raakalukuna

**DAQ-A** -tuoteperhe on peräisin ajalta ennen sarjanumeroittain jaettua pakettijärjestelmää, eikä sillä ole kalibrointipakettia,
jota voitaisiin hakea — se kalibroidaan sen sijaan kentällä heijastavuuskohteen avulla, minkä vuoksi
sille ei ole koskaan tarvittu kalibrointipakettia. Näiden tallenteiden hylkääminen olisi estänyt niiden
lukujen saamiseksi ulos lainkaan, joten ne viedään **eri nimellä**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Käytetään erillistä tiedostonimeä tiedoston sisäisen tunnisteen sijaan, koska tiedoston nimen on säilyttävä
sellaisenaan, kun se lähetetään sähköpostitse pelkänä nimenä. `.csv`-otsikossa lukee
`raw spectral sensor counts (NOT irradiance)` ja varoitetaan, että arvot ovat vertailukelpoisia
**tiedoston sisällä** — mikä on juuri se, mihin kohdepohjainen kalibrointi niitä käyttää — eikä
anturien välillä. Teho riippuvaiset fotometriset sarakkeet (kokonaisteho, fotopinen ja
skotopinen lux, PPFD) kirjoitetaan **NULL**:na sen sijaan, että ne integroitaisiin laskentojen perusteella, ja ajon
yhteenvedossa lukee `RAW COUNTS`, joten lokissa ”vietyä” lokissa ei voida tulkita säteilyvoimakkuudeksi.

Vanhat **v1.01 / v1.02**-tallenteet (joita DAQ-A-SD kirjoittaa) eivät sisällä lukukohtaista aikamerkintää,
vaan ainoastaan tiedoston kirjoitusajan. Kuva↔alasuuntaisen säteilyn täsmäytin hylkää ne edelleen —
kehyksen täsmäyttäminen kirjoitusajankohtaan johtaisi huomaamattomaan virheeseen — mutta vientiohjelma lukee ne, ja
CSV-tiedostossa tulostuu `clock=daq_created_on`, joten tuote ilmoittaa, minkä kellon mukaan se toimii.

### Huomautuksia

- `process` tunnistaa automaattisesti, onko kansiosi Survey3-, LATTICE- vai sekakansio.
- Edistymisraportit välitetään Server-Sent Events -protokollan kautta; CLI näyttää reaaliaikaisen edistymisen säikeittäin (tunnistaminen, analysointi, käsittely, vienti).
- Linux/Jetson-ympäristössä CLI tarkistaa swap-tilan ja saattaa antaa varoituksen ennen suurten kansioiden käsittelyä. Tekstuuritunnistava debayer soveltaa myös automaattisesti GPU:n taajuusrajoitusta vähävirtaisissa Jetson-laitteissa (Nano, Orin Nano).
- Onnistuessaan suoritus raportoi, kuinka monta kuvatuotetta se tallensi (`Image products written: N`; jos sekoituksen purkaminen rajoitti jotakin pikseliä, tulos on `(plus M clamp masks)`).
- **`process` jakaa käynnissä olevan taustaprosessin turvallisesti.** Se käynnistää jo käynnissä olevan taustaprosessin uudelleen automaattisesti vain, jos kyseinen taustaprosessi on lepotilassa, edellinen projekti on ladattu ja siinä ei ole kameroita, taulukoita tai DAQ-laitteita. Taustaprosessia, joka käsittelee jotakin näistä (matriisia suoratoistava käyttöliittymä, SDK-skripti, DAQ-tallennus), käytetään sellaisenaan, eikä sitä koskaan käynnistetä uudelleen, joten `process` voi toimia reaaliaikaisen matriisin rinnalla. Jos taustaprosessi ilmoittaa käynnissä olevasta ajosta, `process` odottaa enintään 15 sekuntia sen päättymistä (juuri päättyneen ajon tyhjentäminen kestää noin sekunnin), minkä jälkeen se hylkää pyynnön virheellä `Backend is mid-run (processing). Wait for it to finish, or pass --restart to force a fresh backend (this kills the in-flight run).`. Globaali lippu `--restart` pakottaa edelleen uuden taustapalvelimen käyttöön ja lopettaa kaikki siinä käynnissä olevat kamera-, taulukko- tai DAQ-istunnot. Taustapalvelinta, jota Linuxin systemd-palvelu ajaa, ei koskaan käynnistetä uudelleen: `process` käyttää sitä, ja `--restart` tulostaa sen sijaan `sudo systemctl restart <unit>`:n (katso [Taustapalvelun ajaminen palveluna](#running-the-backend-as-a-service-headless)). Eikä myöskään Chloros-sovelluksen omaa taustapalvelinta: kun sovellus on auki, `process` käyttää sitä ja lataa oman projektinsa sovelluksen projektin tilalle (sen tilalle käynnistetty taustapalvelin lukitsisi avoimen sovelluksen ulos siitä).
- **Käynnissä oleva käsittelykierros säilyttää projektinsa.** Kun käsittelyajo on käynnissä — käynnistettynä sovelluksella, `process`:llä, SDK-skriptillä tai `chloros-mcp`:llä — taustapalvelu kieltäytyy luomasta, avaamasta, tuomasta, lisäämästä tai poistamasta tiedostoja, siirtämästä, poistaa tai muuttaa projektin käsittelyasetuksia, ja palauttaa HTTP-virheen 409 (`PROCESSING_IN_PROGRESS` ja `A processing run is in progress on project '<name>', so ... now is refused`); `process`, joka saa tämän virheen, poistuu `1`. Näyttöasetukset (pikkukuvien koko) tallentuvat edelleen. Käsittelytila ja projektin kohdeheijastavuusasetukset hylätään samalla tavalla (ajokerta säilyttää sen tilan, jossa se aloitettiin). Käynnin aikana projektiin tallennettu kaappaus (sovelluksesta, CLI:stä, SDK:stä tai `chloros-mcp`:stä) tallennetaan tavalliseen tapaan, mutta se liitetään projektin tiedostoluetteloon vasta, kun ajo päättyy; tämä käy ilmi vastauksesta (`project_add`), samoin kuin projektiin keskeytetty valosensorin tallennus. Aiemmin toinen asiakasohjelma saattoi vaihtaa avointa projektia ajon aikana, jolloin käynnissä oleva ajo vietiin toiseen projektiin.
- **Ctrl+C pysäyttää suorituksen taustaprosessissa, ei pelkästään CLI:ssä.** CLI-käyttöliittymä pyytää taustaprosessia lopettamaan käsittelyn, tulostaa `Waiting for the backend to stop (Ctrl+C again to stop waiting)...` ja odottaa enintään 120 sekuntia viestiä `Processing stopped.`, jotta `chloros-cli process … && next` ei käynnistä `next`:ää taustapalvelimella, joka on edelleen varattu ajon kanssa. Toinen Ctrl+C lopettaa odottamisen. `process` lopettaa sitten `130`:n, kuten kaikki keskeytetyt komennot tekevät.

#### Suoritus, joka ei kirjoita kuvia, epäonnistuu

Jos pyysit tuotteita ja ajo kirjoitti **ei mitään** — vain `project.json` ja
`calibration_data.json` — `process` käsittelee tämän epäonnistumisena: se tulostaa
`Processing finished but wrote no image products.` ja **päättyy nollasta poikkeavalla arvolla**, joten skripti voi
havaita sen. Viestissä mainitaan projektikansio ja tavallisimmat syyt:

- syöttökansiota ei tunnistettu tallennukseksi (tarkista asettelu ja `--input-level`), tai
- jokainen pyydetty tuote ohitettiin, koska se ei sovellu kyseisille kameroille (esim.
  säteilyvoimakkuutta/heijastavuutta vain RGB-kameroista).

Suorita komento uudelleen komennolla `--verbose` ja tarkista taustapalvelimen lokista rivit `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
jotka selittävät kamerakohtaiset ohitukset, jotka eivät muuten näy CLI:n tulosteessa.

Tarkoituksellinen, pelkästään metatietoja tuottava ajo — kaikki tuotteet pois päältä ja ilman `--indices`:ää — on silti
**onnistunut**, koska tyhjä kuvatulos on siinä tapauksessa oikea tulos.

#### Ajo, jonka vientiä ei voitu tarkistaa, epäonnistuu

Suorituksen jälkeen `process` tarkistaa vietyjen tiedostojen tunnisteet ExifToolilla. Jos ExifToolia ei voitu suorittaa, se
tulostaa `Viennin tarkistus EPÄONNISTUI: ExifToolia ei voitu suorittaa, joten N:n M:stä viedystä tiedostosta
ei tarkistettu (katso [EXIFTOOL]-rivit taustalokissa)` and `Tulokset on tallennettu ja ne pysyvät
paikallaan; suorita ajo uudelleen niiden tarkistamiseksi.`, does not print `Käsittely valmis!`, ja **päättyy nollasta poikkeavalla arvolla**.
Tulokset säilytetään.

#### Mitkä kuvat sisällytetään ajokertaan

Ennen `process`:n suorittamista valmiiksi tulleet kuvat sisällytetään. Ajon aikana tallennetut kuvat eivät sisälly; `process`
varoittaa, kuinka monta kuvaa jäi pois, ja ohjelman uudelleenkäynnistys sisällyttää ne.

#### Ajo, jossa kuva hävisi, epäonnistuu

Jos kuva epäonnistui ajon aikana, `process` tulostaa `N image(s) failed and were not exported`:n, jossa on
yleisin syy, ei tulosta `Processing complete!`:ää ja **päättyy nollasta poikkeavalla arvolla**. Muut tuotteet
tallennetaan ja jäävät paikoilleen, ja taustalokissa näkyy jokainen epäonnistuminen.

Sama pätee **vain valosensorilla suoritettuun ajoon**: `.daq`-tallenteiden kansiossa ei ole määritelmän mukaan vietyä kuvamateriaalia,
ja ajo arvioidaan sen sijaan sen kirjoittamien kalibroitujen `.daq`- ja `.csv`-tiedostojen perusteella.
Jos se ei kirjoittanut **mitään** — jokainen tallenne ohitettiin — se tulostaa
`Processing finished but wrote no light-sensor products.`, nimeää jokaisen tallenteen sen
ohittamisen syyn mukaan ja **päättää ohjelman nollasta poikkeavalla arvolla**.

---

## `chloros-cli login`

Todentaa tämän laitteen Chloros+ -pilvitilillä. Kirjautumistiedot tallennetaan turvallisesti välimuistiin tiedostoon `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Esimerkkejä

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (poistamalla osia salasanasta tai toistamalla sen osia). Virheen 401 sattuessa CLI-sovellus yrittää automaattisesti uudelleen lisäämällä `$$`-liitteenä, ja sen jälkeen puolet salasanasta ilman päällekkäisyyksiä; jos uudelleenkäyttö onnistuu, se kirjaa sinut sisään ja tulostaa oikean yksinkertaisten lainausmerkkien syntaksin, jota käytetään seuraavalla kerralla.

> **Päättömä/skriptoitu käyttö: välimuistissa ei ole istuntoa, joten näkyviin tulee kirjautumiskehote.** Core-komento (`process`, `status`, `export-status`, `time-sync`, …) ilman välimuistissa olevaa istuntoa pyytää `Email:` / `Password:`:ää stdin:stä ennen jatkamista. Jos stdin on suljettu (cron, systemd-yksikkö), ohjelma epäonnistuu välittömästi virheellä `login cancelled`; jos stdin on avoin pääte tai putki, ohjelma odottaa syötettä. Suorita `chloros-cli login EMAIL PASSWORD` kerran jokaisella koneella ennen päättömän työn ajoittamista.

---

## `chloros-cli logout`

Tyhjentää välimuistissa olevan istunnon ja pakottaa uuden kirjautumisen seuraavalla kutsulla.

Uloskirjautuminen kesken käsittelyjuoksun ei keskeytä sitä: juoksu päättyy siinä käsittelytilassa, jossa se alkoi, ja seuraava juoksu käyttää vakiokäsittelyä. `logout` kertoo tämän (`The processing run in progress finishes in <mode> mode; the next run uses standard`), ja niin kertoo myös sovellus, kun kirjaudut siellä ulos.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Näyttää nykyisen lisenssitason (Iron/Copper/Bronze/Silver/Gold), todennetun käyttäjän ja laitesidosten lukumäärän.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Tarkista Thread-4-viennin reaaliaikainen eteneminen. Voidaan kutsua turvallisesti **kesken** `process`-komennon suorituksen toisesta komentotulkista.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Aseta CLI:n näyttökieli (tukee 38 kieltä, mukaan lukien CJK, RTL ja indialaiset kielet). Siirtyy sujuvasti englanniksi vanhoissa konsoleissa, jotka eivät pysty näyttämään skriptiä.

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

`get-project-folder` tulostaa kansion, johon uudet projektit tallennetaan. Jos määritetty kansio ei enää ole olemassa (se on poistettu, nimetty uudelleen tai sijaitsee levyllä, joka ei ole kytkettynä), CLI-ohjelma siirtyy oletuskansioon, kuten graafinen käyttöliittymäkin: oletuskansio tulostetaan stdout-ulostuloon, puuttuvaa kansiota koskeva varoitus lähetetään stderr-ulostuloon ja poistumiskoodi on `0`.

---

## `chloros-cli update`

Vain Linux / Jetson. Tarkistaa `version_url`:n ja `/etc/chloros/update.conf`:n väliltä ja, jos uudempi versio on olemassa, lataa vastaavan `.deb`-version ja asentaa sen välittömästi `sudo dpkg -i`-komennolla (ei vahvistuspyynnötä; `sudo` saattaa pyytää salasanaasi). Jetson-laitteella päivitystä tarjotaan vain JetPack 6 -järjestelmälle. Windowsissa komento ei tee mitään ja ilmoittaa `up to date`: päivitä uudella asennusohjelmalla.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Linuxissa/Jetsonissa CLI suorittaa myös **automaattisen päivitystarkistuksen jokaisella käynnistyksellä** (ei estä toimintaa, ei koskaan viivästytä komentoa): se lukee `/etc/chloros/update.conf`, tallentaa tuloksen välimuistiin tunniksi `~/.chloros/update_cache.json` ja tulostaa `Update available: vX.Y.Z / Run: chloros-cli update`, kun uudempi versio on saatavilla. Ohitetaan hiljaisesti virhetilanteissa ja Windows-ympäristössä; komento `CHLOROS_DISABLE_UPDATE_CHECK=1` poistaa tämän toiminnon käytöstä. Yhden tunnin välimuistin vuoksi tunnin sisällä julkaistu versio voi silti näkyä ajantasaisena.

---

## `chloros-cli selftest`

Suorittaa 7-vaiheisen pikatestin: versio, portin saatavuus, taustaprosessin käynnistys, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), kohinanpoistomallin olemassaolo, CUDA+kohinanpoistajan valmius. Se käyttää käynnissä olevaa taustapalvelinta tai käynnistää sellaisen testausta varten ja pysäyttää sen jälkeen. Se tulostaa `N/7 checks passed`:n ja epäonnistuneet tarkistukset sekä lopettaa `0`:n vasta, kun kaikki seitsemän tarkistusta on läpäisty; CUDA+kohinanpoistotarkistus epäonnistuu tietokoneella, jossa ei ole CUDA-GPU:ta (katso [Asennuksen tarkistaminen](#verify-the-install)).

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

PTP-grandmasterin tila ja ohjaus. Chloros-isäntäkone toimii PTP-grandmasterina; LATTICE-kamerat ja DAQ-E-yksiköt toimivat sen alaisina laitteiden välisten aikaleimojen varmistamiseksi (noin 1 ms laitteiden välillä). PTP synkronoi **aikaleimat**: kameraryhmän kamerat ottavat kuvan samanaikaisesti, koska M8-laitteiston laukaisija laukaisee ne (alle 100 µs moduulien välillä), ei PTP:n ansiosta. `chloros-cli time-sync` ilman alikomentoa suorittaa `status`:n.

| Alikomento | Kuvaus |
| --- | --- |
| `status` | Näyttää grandmasterin tilan, BMCA-prioriteetit, kellotunnisteen ja sen, pääsevätkö sen paketit verkkoon: `health` on `OK` (`STOPPED`, kun grandmaster ei ole käynnissä) tai `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` ja `health_detail`, ja sen jälkeen stderr:iin tulostetaan rivi `WARNING:` (stdout pysyy JSON-muodossa). `sync_sent` / `announce_sent` laskevat vain ne viestit, jotka lähetettiin; `*_send_errors` ja `last_send_error` laskevat ne, joita ei lähetetty. Kun kamerayhteys katkeaa ja palautuu, grandmaster rakentaa socketit uudelleen itsenäisesti (`rebinds`, `last_rebind`). |
| `peers` | Luettelee Delay_Req:n kautta havaitut orjat (kamerat + DAQ-E-anturit). |
| `cameras` | Kamerakohtainen PTP-kunto (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Käynnistä grandmaster-prosessi uudelleen. |
| `set-priority --priority1 N --priority2 N` | Ohita BMCA-prioriteetit. |

Grandmaster ilmoittaa kameran verkkokorttien kautta (mikä tahansa sovitin, jonka osoitteeseen sisältyy `169.254.x.x`). Jos yhtään ei ole käytössä, se siirtyy oletusreitin sovittimeen vain, jos se on langallinen, ei koskaan Wi-Fi- tai virtuaalisovittimeen (Tailscale, WSL), koska mikään kamera tai DAQ-E-laite ole niiden takana. Jos ilmoitettavaa ei ole, se toimii ilman socket-yhteyksiä (`health` `NO_SOCKETS`), ja seuraavan kameran tai DAQ-laitteen yhteyden muodostamisen yhteydessä se rakennetaan uudelleen siihen verkkokorttiin, joka on sillä hetkellä käytössä; `time-sync restart` tekee tämän kerralla.

Grandmaster toimii omassa prosessissaan, backendin rinnalla (`own_process` / `process_pid` kohdassa `time-sync status`), korkealla prioriteetilla Windowsissa (`process_priority`), joten kiireinen taustapalvelin (käsittelyajo, vienti, useita komentoja kerralla) ei voi viivästyttää sen leimaamia aikoja; aiemmin yksi tällainen viivästys saattoi siirtää kaikkien kameroiden kelloa kerralla. Tämä prosessi päättyy taustaprosessin mukana; jos sitä ei voida käynnistää, grandmaster toimii taustaprosessin sisällä kuten aiemmin (`own_process` false, ja taustaprosessin loki ilmoittaa tämän).

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

LATTICE-kameran ohjaus. Jokainen alikomento reititetään Chloros-taustapalvelimen kautta; taustapalvelin hallinnoi kamerapoolia, joten seuraavat CLI-kutsut käyttävät uudelleen samaa avointa kahvaa.

### Yleiset asetukset (yhteisiä useimmille alikomennoille)

| Lippu | Kuvaus |
| --- | --- |
| `-d, --device N` | Kameran indeksi (oletus: 0). |
| `-s, --serial SN` | Tietty sarjanumero; ohittaa `--device`-komennon. |
| `--serials SN1,SN2,…` | Pilkuilla erotetut sarjanumerot monikamerakäyttöä varten. [Väliaikaiset monikamerakomennot](#multi-camera-transient-sessions) avaavat vain nämä kamerat, joista ensimmäinen laukaisee laukaisimen. |
| `--all` | Toimii jokaisella löydetyllä kameralla. |
| `--exposure US` | Valotusaika mikrosekunteina. |
| `--gain DB` | Vahvistus dB:nä. |
| `--pixel-format FMT` | esim. `BayerRG8`, `BayerRG12`; `Mono8`, `Mono12` M3M-mallissa. Nimi määrittää sen bittisyvyyden kameran omassa tuoteperheessä: M3M-mallissa Bayer-nimi määrittää sen syvyyden Mono-muodon (`BayerRG8` määrittää `Mono8`), M3C-mallissa Mono-nimi määrittää Bayer-muodon (`Mono12` määrittää `BayerRG12`). |
| `--width N` / `--height N` | Kuvan mitat. |
| `--preset {default,high_quality,high_speed,triggered}` | Asetusten esiasetuksen soveltaminen: sen pikselimuoto, valotus, vahvistus ja kuvataajuus (`high_speed`: `BayerRG8`, 5 ms, 6 dB, 60 fps; `high_quality`: `BayerRG12`, 20 ms, 0 dB; `lattice presets` luettelee ne). M3M-laitteella formaatti on samaa syvyyttä oleva Mono-formaatti: `high_speed` tallentaa `Mono8`:n, `high_quality` ja `triggered` `Mono12`. Sen mukana annettu lippu (`--exposure`, `--gain`, `--pixel-format`) ohittaa esiasetuksen, ja asetus, jota esiasetuksessa ei mainita, säilyttää kameran oman asetuksen. Kaikki vapaattoimivat vapaasti, paitsi `triggered`, joka asettaa kameran valmiustilaan linjan 2 laitteistoreunan perusteella — jos linjaa ei ohjata millään, kamera odottaa ikuisesti sen sijaan, että se tallentaisi kuvaa. |
| `-o, --output DIR` | Tulostuskansio (oletus: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | GVSP-paketin koko. `auto` suorittaa ICMP+GVSP-tunnistuksia; `jumbo` = 9000; `standard` = 1500. |

### LATTICE-kameran ensimmäisen yhteyden muodostuksen työnkulku

```bash
# 0. A backend must be running: the Chloros desktop app (Windows),
#    `chloros-backend`, or the Linux service (see "Start the Backend")

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
#      - Full sensor (no binning, no ROI); the frame rate falls to
#        what the host and wire deliver, and the faster options are
#        printed
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

#### Tunnistus ja tiedot

| Alikomento | Tarkoitus |
| --- | --- |
| `lattice info` | Luettelee kytketyt kamerat (valmistaja, malli, sarjanumero, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analysoi isäntäjärjestelmän optimaalisen kamerakonfiguraation löytämiseksi. `--no-discover` ohittaa kameran tunnistuksen (nopeampi, vain verkkokortin analyysi). |
| `lattice network [--fix] [--estimate] [--cameras N] [--pixel-format FMT]` | Tarkista/korjaa verkkokortin asetukset (Linuxissa myös ytimen socket-vastaanottopuskureita — katso [Linux-isäntäkoneet](#linux-hosts--socket-receive-buffers)); arvioi kaistanleveys/FPS. |
| `lattice network-analysis --master SN [--slaves SN1,SN2,…] [--width N] [--height N] [--pixel-format FMT] [--binning {1,2,4}] [--force-tier TIER] [--packed {auto,on,off}] [--backend-url URL] [--json]` | Stable-schema-taustapalvelimen verkko-ominaisuus + taulukkosuositus (palauttaa `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `--width` / `--height` oletuksena koko anturi, `2048` × `1536`. Se poistuu `0`:stä `ok`:n, `auto_capped_fps`:n ja `auto_shrunk`:n kohdalla sekä `1`:n kohdalla `needs_force_slip`:n ja `error`:n kohdalla, riippumatta siitä, onko mukana `--json`. Se saavuttaa taustapalvelimen kohdassa `--backend-url` (oletusarvo `http://127.0.0.1:5000`), ei `CHLOROS_BACKEND_URL`. `auto_capped_fps` säilyttää pyydetyn resoluution, mutta rajoittaa kohdekuvataajuuden — lue `recommended.recommended_target_fps` ja välitä se connect-kohteeksi; käsittele sitä onnistumisena, ei virheenä. `auto_shrunk` ilmestyy vain, kun pyydettyä kuvaa ei voida toimittaa millään kuvataajuudella; connect ei koskaan pienennä kuvia omasta aloitteestaan. |
| `lattice analyze-array [--models M1,M2,…] [--binning {1,2,4}] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--packed {auto,on,off}] [--json]` | Mitä-jos-analyysi avaamatta kameroita (`--width` / `--height` oletusarvo `2048` × `1536`). Sekä tämä että `network-analysis` ennustavat, mitä connect lähettää: ilman `--pixel-format`:ää Connect soveltaa kamerakohtaisia formaatteja (Mono12 / BayerRG12), ja kun käytössä on `--packed auto` (oletus), [pakattu 12-bittinen](#packed-12-bit-transport---packed) -päätös, jonka Connect tekisi tällä tietokoneella — joten ilmoitettu fps on se nopeus, jolla ryhmä toimii. Molemmat tulostavat pakkauspäätöksen. **`--n-active` on verkossa olevien kameroiden kokonaismäärä, ei vain tämän ryhmän**— nosta sitä, kun erilliskamerat lähettävät dataa samanaikaisesti tai kun verkon kapasiteettia laskettaessa niiden määrä on aliarvioitu (oletus: `len(--models)`). Tulostaa aina yhteenlasketut rivit `Wire budget:` (vaadittu MB/s vs. törmäysturvallinen yläraja) ja `Max cameras:` sekä merkitsee `** OVER-SUBSCRIBED**`:n, kun ryhmä ylikuormittaa kaapelia — katso [Matriisin fps- ja pursemalli](#array-fps--burst-model). |
| `lattice gpu` | Näytä GPU:n tila. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Tarkista tai päivitä kameran laiteohjelmisto. Paikallinen `.fwa`-valinta on kiinnitetty: tiedosto, joka sijaitsee `firmware/<MODEL_PREFIX>/`:ssa ja vastaa rakennuksen `MIN_FIRMWARE_VERSION`:ää, flashataan, kun se on läsnä (vain korkein versio varavaihtoehtona), joten levylle tallennettu uudempi valmistajan kuva pysyy passiivisena, kunnes kyseinen kiinnitys poistetaan — tarkoituksellisesti uudemmat julkaisut saapuvat laitteisiin allekirjoitetun AWS-manifestin kautta, jota suositellaan käytettäväksi, kun se on uudempi. |
| `lattice presets [--apply NAME]` | Luettele tai ota käyttöön kameran esiasetukset. `--apply` määrittää esiasetusten nimet (pikselimuoto, valotus, vahvistus, kuvataajuus, laukaisija) ja tulostaa nämä asetukset sellaisina kuin kamera lukee ne: M3M-laitteella esiasetuksen syvyyden Mono-muoto; kuvataajuus, jolla se toimii, joka voi olla alhaisempi kuin esiasetus vaatii (`high_speed` vaatii 60 fps; täyskuvakamera 1 Gbps:n yhteydellä toimii noin 35 fps:llä). Kaikki muut asetukset, jotka sen avoin tiedosto kirjoitti, palautetaan sellaisina kuin ne löydettiin. `--apply default` palauttaa kameran tehdasasetuksiin: sen tehdasasetukset, joissa Line2:lla on vapaasti toimiva automaattinen valotus, linkin läpimenorajoitus ja vikakorjaus pois päältä sekä RGB-kameran kalibroitu valkotasapaino säilytetään (muissa kameroissa yhtenäisyys). Mitään ei tallenneta kameran muistiin, joten seuraavalla käynnistyskerralla se käynnistyy omasta muististaan kuten aiemminkin. Kun kyseessä on tallennus- tai yhteyskomento `--preset`, `default` ei ole esiasetus. |
| `lattice status` | Kameran reaaliaikainen tila. |

#### Tallennus

| Alikomento | Tarkoitus |
| --- | --- |
| `lattice capture [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--no-daq] [--daq-tolerance-ms MS]` | Yksittäinen ruutu. **Tallentaa oletuksena kaikki vientityypit**(`--processing all`); katso [Tallennuksen vientitasot](#capture-export-levels-the-all-default). `--processing` hyväksyy `all`, `raw`, `debayered`, `radiance`, `reflectance` tai `display` (`preview` hyväksytään nimelle `display`:n osalta); kaikki muut nimet hylätään ennen kameran koskettamista (poistu `1`). `--levels` tallentaa eksplisiittisen osajoukon (korvaa `--processing`). Chlorosin hallussa olematon kamera avataan kuvauksen ajaksi, ja sen valotus asetetaan ensin kohtauksen mukaan (korkeintaan muutama sekunti), kuten [väliaikaiset komennot](#multi-camera-transient-sessions) tekevät; kun tallennus päättyy, riippumatta siitä miten se päättyy, kyseinen kamera palautetaan tilaan, jossa se löydettiin (formaatti, valotus, väriprosessointiputki, kaikki tallennuksen muuttamat asetukset — luettelo kohdassa [Monikamera (väliaikaiset istunnot)](#multi-camera-transient-sessions)). Käytetyn tallennuksen lukeva DAQ tallennetaan `.daq`-sidecar-tiedostona, johon sisältyy pelkkä raakadata; `--no-daq` / `--daq-tolerance-ms` säätävät sitä ([Tallennusta lukeva DAQ tallentaa](#the-daq-reading-a-capture-saves)). `--force-daq` hyväksytään edelleen. `--jpeg-quality` = JPEG-laatu 1–100 (oletus 95).**Avoimeen projektiin tallennetut tiedostot liitetään siihen.** Kuten sovelluksessakin, tallennus, jonka tiedostot päätyvät Chlorosissa avoimena olevan projektin kansioon, lisää ne kyseisen projektin tiedostoluetteloon; muualle tallennetut tiedostot jätetään rauhaan. Komento tulostaa yhden rivin, jossa ilmoitetaan tästä (`Project: ...`), tai, jos käsittely on käynnissä, että ne pidetään odottamassa ja liitetään projektiin käsittelyn päätyttyä. Sama pätee myös komentoihin `multi-capture`, `array-capture` (myös `--smart`) ja `project capture`. |
| `lattice continuous [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--queue-depth N] [--no-daq] [--daq-tolerance-ms MS]` | Siirtää tiedostoja levylle, kunnes painetaan Ctrl+C. `tiff`-ajon:n kehykset muodostavat `raw`-tiedoston, jonka kameran kaappaus kirjoittaa: anturin mosaiikki, yksi taso omilla arvoillaan, pakkaamaton, kalibrointipaketin kanssa (2048 x 1536 12-bittinen kehys on 6,0 MiB plus paketti, riippumatta kuvauskohteesta), joten `process` alkaa niistä. `png` ja `jpg` sisältävät 8 bittiä, joten nämä sarjat ovat demosaikoituja kuvia katselua varten, joista `process` ei lähde liikkeelle. DAQ-sidoksella (`daq pool-bind`) tai yhdistettynä kukin kehys saa omat `.daq`- ja `.match.json`-tiedostonsa sarjan `daq/`-kansiossa, koska `lattice capture` tallentaa yhden; `--no-daq` poistaa tämän käytöstä, ja lopetuksen yhteydessä tulostetaan, kuinka monta niitä on. Jokainen kehys tallennetaan kerran, täydellisine metatietoineen; Ctrl+C-näppäinyhdistelmällä jo kaapatut kehykset tallennetaan ennen kuin kamera vapautetaan. Ohjelma pysähtyy itsestään, kun asema täyttyy, tulostaa `Failed:` / `Disk full:` ja lopettaa toimintansa `1`; jokainen kehys, jota se ei pystynyt tallentamaan, lopettaa myös toimintansa `1`. Kamera, joka on jätetty valmiustilaan laitteistolähtöä varten (`TriggerMode` `On`) asetetaan vapaakäynnille kuvausjakson ajaksi. Kun kuvausjakso päättyy, kamera palautetaan tilaan, jossa se löydettiin — sen laukaisuasetukset ja kaikki kuvausjakson aikana muutetut asetukset (valotus, vahvistus, formaatti jne.) – ellei kyseessä ole `--keep-settings`, joka tallentaa kuvaussarjan asetukset kameran omaan muistiin (vapaakäynti), tai `--clear-settings`, joka palauttaa kyseisen muistin tehdasasetuksiin. Kuvausjakso, joka ei tallenna yhtään kuvaa, on epäonnistunut: jos kuvia ei tallenneta 10 sekunnin ajan, se pysähtyy itsestään, tulostaa `No frames:` syyn kera ja lopettaa toimintansa `1`. Kuvat, joiden tallentamiseen kirjoittajat eivät ehtineet, lasketaan hylätyiksi, ja pysäytys tulostaa viestin `Dropped:` syyn kera (kameran kuvataajuus, kirjoittajien kuvataajuus ja jono); alhaisempi kuvataajuus tai 8-bittinen pikselimuoto säilyttää ne kaikki. Komento tarkistaa yhteyden taustapalvelimeen 2 sekunnin välein; jos se kaatuu pysäyttämättä tallennusta (`kill -9`, muisti loppu), taustapalvelin pysäyttää sen 30 sekuntia viimeisen tarkistuksen jälkeen ja palauttaa kameran (`stop_reason` `client_gone`, ja rivi taustapalvelimen lokissa). `multi-stream` ja `align-stream` toimivat samalla tavalla. Käynnistys, jossa kameran avaaminen kestää kauemmin kuin komento odottaa (60 s; 180 s kohdissa `multi-stream` / `align-stream`) — kamera on vielä käynnistymässä — peruutetaan taustapalvelimella: kamera palautetaan ja suljetaan, eikä tallennusta jätetä käynnissä (504-virhe HTTP:ssä `CLIENT_DEADLINE_PASSED`). |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Selainpohjainen MJPEG-reaaliaikainen esikatselu: muodostaa yhteyden kameraan, tulostaa videovirran URL-osoitteen ja avaa sen selaimessasi. `--ae-damping` asettaa automaattisen valotuksen vaimennuksen (0,4–100). |

##### Suorat videovirrat (MJPEG; ei RTSP)

Chloros toimittaa suoraa videokuvaa **MJPEG-muodossa HTTP:n kautta**(`multipart/x-mixed-replace`, raja-arvo `frame`) taustapalvelimelta.**RTSP:tä ei tueta.** Avaa URL-osoite selaimessa tai lue se millä tahansa MJPEG-asiakasohjelmalla (OpenCV:n `cv2.VideoCapture(url)`, ffmpeg, VLC). Kamera on liitettävä ensin taustapalvelimeen (`lattice cam-connect`, `lattice array-connect`, `lattice viewer` tai käyttöliittymän kautta).

| URL (`http://127.0.0.1:5000` + …) | Stream |
| --- | --- |
| `/api/camera/stream/<serial>` | Esikatselukuva, enintään noin 30 kuvaa sekunnissa. Taulukon jäsenenä `?max_h=` (oletus `720`) ja `?jpeg_q=` (oletus `80`) määrittävät korkeuden ja JPEG-laadun. Tämä on URL, jonka `lattice viewer` avaa. |
| `/api/camera/<serial>/stream-annotated` | Sama esikatselu, johon on piirretty käyttöliittymän päällekkäiset elementit: `zebra=1`, `crosshair=1`, `grid=3x3` (tai `9x9`), `histogram=1`, `peaking=1`, `spot=1`; ilman kyselyä, projektin tallennetut peittokuva-asetukset kyseiselle kameralle. |
| `/api/camera/<serial>/stream-reflectance` | 8-bittinen heijastavuus kaistakohtaisesti: ρ = pikseli ÷`X-Chloros-Pixel-Scale`-otsikosta (255, ellei `refl_max` ole asetettu). `fps` (oletusarvo `10`), `jpeg_q` (oletusarvo `95`), `max_h` (oletusarvo `0` = natiivi), `band=<name>`, `align=1`. JPEG, eli häviöllinen. Hylätään HTTP-koodilla 409, kun kameralla ei ole heijastavuutta (RGB-kamera, ei kalibrointipakettia, ei DAQ-lukemaa). |

- **Kuvien täsmäyttäminen kameroiden välillä.**Jokainen**taulukon jäsenen** lähettämä osa sisältää kaikissa kolmessa datavirrassa `X-Chloros-Group-Id` (taulukon synkronoidun ryhmän laskuri), `X-Chloros-Frame-Id` ja `X-Chloros-Timestamp-Ns`-otsikot. Eri kameroista peräisin olevat osat, joilla on sama `X-Chloros-Group-Id`, ovat peräisin samasta laukaisusta. Itsenäisellä kameralla ei ole ryhmää, eikä se lähetä ryhmätunnusta. Selaimet sivuuttavat osakohtaiset otsikot; asiakasohjelma, joka jäsentää moniosaisen virran itse, lukee ne.
- **Keskeytetty, kun ryhmänäkymä on auki.** Ryhmän jäsenen `/api/camera/stream/<serial>` lähettää staattisen ”paused”-kehyksen, kun ryhmän yhdistetty reaaliaikainen virta on auki (esimerkiksi käyttöliittymän ryhmänäkymä), ja jatkaa lähetystä, kun se sulkeutuu.

#### Anturin säätö

| Alikomento | Tarkoitus |
| --- | --- |
| `lattice configure [{--get N1 N2… \| --set N=V N=V… \| --dump}] [--json]` | Lue/kirjoita mikä tahansa GenICam-solmu. `--set ExposureTime=…` (`Gain=…`) ilman sen `ExposureAuto` (`GainAuto`) -kytkimiä, jotka ensin kytkevät automaattitilan pois päältä, kuten `lattice exposure --set` tekee; arvo, jota kamera ei hyväksy, tulostaa `Failed to set <node>: …` stderr:iin ja lopettaa `1`:n. `--get` ja oletusarvoinen ryhmitelty näkymä tulostavat `note:`:n kohdassa `ExposureTime` / `Gain`, kun automaattitila on käynnissä — katso [Valotuksen ja vahvistuksen lukeminen automaattitilassa](#reading-exposure-and-gain-under-auto-mode). |
| `lattice exposure [{--auto \| --auto-once \| --off \| --set US}] [--brightness 0-255] [--brightness-auto] [--damping 0.4-100] [--upper-limit US]` | Valotus ja AE. `--set`-valotus tai `--brightness`-kohde pidetään omasi (Chlorosin automaattinen valotuksen säätö jättää sen rauhaan); vahvistus pysyy ennallaan, joten automaattitilassa se säätää jatkuvasti kohti kohdetta, kunnes asetat `lattice gain`. `--auto` palauttaa valotuksen, `--brightness-auto` palauttaa kohdearvon Chlorosin automaattiselle kohdearvolle. Ilman lippuja se lukee valotussolmut samalla `note:`-arvolla kuin `configure --get`. Jos kamera hylkää `--set`-arvon, se tulostaa `Failed to set ExposureTime: ExposureTime=<value> refused (<camera error>); the camera accepts <min>..<max> now (ExposureAuto put back to <mode>)` stderr:iin — kun kirjoitus on kytketty pois päältä, automaattitila palautetaan sen sijaan, että se jätettäisiin pois päältä — ei tulosta `Exposure: … us (auto OFF)`-riviä ja lopettaa `1`. |
| `lattice gain [{--auto \| --off \| --set DB}]` | Vahvistus ja automaattinen vahvistus. Jos kamera hylkää `--set`-arvon, se tulostaa `Failed to set Gain: …` stderr:iin, ei tulosta `Gain: … dB (auto OFF)`-riviä ja lopettaa ohjelman `1`. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N]` | Anturin ROI ja pikselien yhdistely. Pikselien yhdistely on aina **keskiarvo** — katso alla. |
| `lattice format [--set FMT] [--list]` | Pikselimuoto. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Laitteisto-/ohjelmistolaukaisin. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (ei lippuja = kertaluonteinen valkotasapaino) | Valkotasapainotoiminnot. Vain RGB- ja Bayer-kamerat; ei-toiminto (ohitetaan) mono-M3M-kameroissa. Yksittäinen valkoisen tasapainon määritys säilyttää laskemansa valkoisen tasapainon (`BalanceWhiteAuto` ja vahvistukset); kaikki muut asetukset, joiden avoin kirjoitus on määritetty, palautetaan löydetyssä muodossa, ja kaikki asetukset palautetaan, jos yksittäistä valkoisen tasapainon määritystä ei voida suorittaa. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct 2000-10000] [--get]` | RGB-näytön väriprosessointiputki. `natural` (oletus) on edullinen reaaliaikainen viimeistely; `enhanced` lisää värinreunojen poiston + eloisuuden + CLAHE-paikalliskontrastin täyden hub-pariteetin ulkoasun saavuttamiseksi noin kaksinkertaisella kuvakohtaisella käsittelykustannuksella, mikä johtaa alhaisempaan **reaaliaikaiseen** kuvataajuuteen — tallennetut kuvat saavat joka tapauksessa aina täyden käsittelyn. Vain RGB-/Bayer-kamerat; ohitetaan mono-M3M:llä. |
| `lattice color [--saturation 0-200] [--contrast 0-200] [--reset] [--get]` | Reaaliaikaisen esikatselun kylläisyys/kontrasti (RGB-suodatinkamerat); `100` on neutraali, `--reset` asettaa molemmat arvoon 100. Tallennettuihin kuviin tämä ei vaikuta. Ohitetaan mono-M3M-kamerassa. |
| `lattice highlights [{--set -100..0 \| --reset \| --get}]` (ei lippuja = lue) | Näytä **korostukset** (kameroissa, joissa on RGB-suodatin): vaalennetaan kirkkaimpia, valkoisimpia pikseleitä — esim. kirkas valkoinen pinta, joka näkyy tasaisena valkoisena läiskänä. `0` = pois päältä, `-100` = voimakkain. Reaaliaikainen esikatselu ja näyttötason kuvat noudattavat tätä asetusta; säteily/heijastavuus eivät koskaan, eikä se voi palauttaa leikkautuneita anturitietoja (vähennä valotusta tämän vuoksi). Ohitetaan mono-M3M:llä. |
| `lattice filter [--list]` | Luettele suodatinmallit, jotka Chloros tuntee (`RGN-IMX265`, `OCN`, `NGB`, …). Kameran suodatin luetaan sen mallimerkkijonosta, eikä sitä voi asettaa: `--set NAME` etsii vain mallin ja lopettaa sitten `1`:n `setting the filter model is not supported`:llä. |
| `lattice power [--sleep]` | Tarkistaa virta- ja lämpösolmut; kytkee virransäästötilan päälle tai pois päältä. `--sleep` jättää kameran lepotilaan (sen linkin läpimenorajoitus kytketty päälle (`DeviceLinkThroughputLimitMode` `On`) alimmalle tasolleen, virransäästötila päällä, jos kamerassa on sellainen) kaikki muut asetukset palautetaan sellaisina kuin ne olivat alun perin; epäonnistunut lepotila palautetaan kokonaisuudessaan. Herätys palauttaa linkin tilaan, jossa se oli lepotilaan siirryttäessä (raja-arvo, sen tila ja pakettiviive `GevSCPD`, jonka kamera pitää arvossa 0, kun raja-arvo on päällä; tieto tallennetaan tähän tietokoneeseen, jotta myös herätys backendin uudelleenkäynnistyksen jälkeen löytää ne), tai jos kyseessä on kamera, jota tämä tietokone ei ole asettanut lepotilaan mutta joka näyttää olevan lepotilassa (raja-arvo alarajalla), raja-arvo poistetaan käytöstä ja `GevSCPD` jätetään arvoon 0 tasolla 80 (tehdasasetukset); raja-arvo, joka on asetettu alarajan yläpuolelle, jätetään ennalleen. Anturi ja herätys palauttavat sen, mitä niiden avauskirjoitti. |

> **Binning on aina keskiarvo.** `lattice resolution --binning N` asettaa keskiarvo-binningin eikä
> tarjoa tilanvaihtoa. Sum-binning kertoo jokaisen DN:n N²:lla, mikä vie noin kaksi aukkoa
> kirkkaiden alueiden varaa binning-askelta kohti; Chloros jakaa tämän kertoimen takaisin ennen
> radiometristä ketjua, mutta pikselit, jotka Sum-tilassa ajettiin anturin ylärajaan, leikkautuvat eikä niitä voi
> palauttaa. Se ei tarjoa mitään sellaista, mitä keskiarvo ei tarjoaisi — indeksi on mittakaavariippumaton ja säteilyvoimakkuuden
> skalaarit on kalibroitu keskiarvon mittakaavassa. Jos kamera saapuu jo Sum-asetuksella (joka on
> jätetty sinne toisella työkalulla tai tehdasasetusten palautuksen seurauksena), Chloros tunnistaa tämän solmun lukemasta ja
> kompensoi sen; komennon `lattice resolution --binning N` suorittaminen palauttaa asetuksen takaisin Average-tilaan. Versiossa 1.2.0
> `--binning-mode`-lippu hyväksytään edelleen, joten vanhemmat skriptit toimivat, mutta se tulostaa vain varoituksen:
> pikselien yhdistely pysyy keskiarvona.

> **Kameran binningin, pikselimuodon ja laukaisun muuttaminen ei tuota tulosta.** Kamerassa, jossa
> taustaprosessi pysyy auki ja suoratoistaa (`cam-connect`, käyttöliittymä), komento `lattice resolution --binning`
> ei tule voimaan (se tulostaa, että binning on edelleen voimassa, `Binning: 1x1`), ja
> kamera hylkää komennot `lattice format --set` ja `lattice trigger --mode`
> (`SC_ERR_ERROR -1001`). Vapauta se ensin (`lattice cam-disconnect -s <SN>`) tai aseta
> ne yhteyden muodostamisen yhteydessä (`cam-connect --pixel-format …`, `array-connect --binning …`).

##### Valotuksen ja vahvistuksen lukeminen automaattitilassa

Kun kameran `ExposureAuto` (`GainAuto`) on käynnissä, sen `ExposureTime` (`Gain`) -solmu säilyttää **viimeksi kirjoitetun arvon**, ei sitä, mitä kamera tekee juuri nyt. Kameran osalta taustapalvelu pitää auki (`cam-connect`, `array-connect`, käyttöliittymä), `lattice configure --get`, oletusnäkymä `lattice configure` ja `lattice exposure` (ilman lippuja) tulostavat siksi solmun arvon ja sen alle huomautuksen, jossa on kameran uusin ruutu:

```

ExposureTime = 2000.448
  note: ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448); the camera's newest frame was at 4900 us.
```

Lue kehysarvo huomautuksesta; solmun arvo on vain viimeisin kirjoitus. HTTP:n kautta sama `POST /api/camera/<serial>/read-nodes`-vastaus säilyttää `nodes`:n muuttumattomana ja lisää `frame_values` (`{"ExposureTime": <us>, "Gain": <dB>}`, uusimman kehyksen) ja `notes` (edellinen lause). Molemmat ovat läsnä vain, kun automaattitila ei ole `Off` ja kehys on tiedossa; vain lukemista varten avattu kamera ei sisällä kehystä eikä saa kumpaakaan. `lattice gain` (ilman lippuja) tulostaa vain solmun — käytä `lattice configure --get Gain`:ää huomautusta varten.

##### Valotus taulukon jäsenellä

Kun taulukko lähettää dataa, jokainen jäsen toimii **`AcquisitionFrameRateEnable` päällä**hieman taajuudella, joka on taajuutta hieman suurempi kuin taajuuden laukaisutaajuus: sen jakso on laukaisujakso vähennettynä 15 %:lla (vähintään 3 ms) tai kameran oma enimmäistaajuus, kun laukaisutaajuus on jo lähellä sitä. Siksi jäsenen `configure --get` lukee `AcquisitionFrameRateEnable`:n arvoksi `True` ja `AcquisitionFrameRate` kyseisellä asetuksella; aktivointi kytkeytyy pois päältä, kun matriisi pysähtyy.**Älä kirjoita näitä kahta solmua matriisin jäsenelle** — matriisi pitää ne synkronoituna laukaisunsa kanssa ja kirjoittaa ne uudelleen.

Tämä asetus on myös jäsenen valotuksen yläraja: noin 85 ms 10 fps:n laukaisulla (100 ms:n jakso), **millä tahansa ROI:lla tai binningillä**. Ennen versiota 1.3.0 pieni ROI rajoitti jäsenen valotuksen sen sijaan anturin vapaakäyntijaksoon (4,9 ms resoluutiolla 768 × 304), riippumatta siitä, kuinka hitaasti matriisi laukaisi.**Manuaalinen `ExposureTime`, joka on pidempi kuin yläraja, hidastaa laukaisua sen mukaiseksi**, kun se kirjoitetaan komennolla `lattice exposure --set` tai `configure --set` (tai käyttöliittymän kautta). Järjestelmä avaa ensin kyseisen jäsenen ylärajan kyseiselle arvolle ja antaa hitaamman taajuuden toimia muutaman laukaisujakson ajan, joten kirjoitus voi kestää muutaman sekunnin pidempään; laukaisu tapahtuu sitten enintään valotuksen sallimalla nopeudella. Automaattitaajuusjärjestelmässä aiempi taajuus palautuu, kun jokainen jäsen, jonka manuaalinen valotus hidasti järjestelmää, on palannut automaattivalotukseen. Arvo, jota kamera itse ei voi ottaa, hylätään edelleen, ja virheilmoituksessa ilmoitetaan sen hyväksymä alue; jäsenen automaattinen valotus palautetaan, jos se oli kytketty pois päältä vain kyseisen kirjoituksen ajaksi. Resepti `apply` ei hidasta laukaisua: se pidetään ylärajalla. Chloros ei koskaan lyhennä manuaalista valotusaikaa nopeamman laukaisun mukaiseksi; kiinnitetty tahti (`array-connect --target-fps F`), joka on valotusaikaa nopeampi, jättää kyseiselle jäsenelle riittävän pitkän ylärajan, ja kyseinen jäsen laukaisee vain niin nopeasti kuin sen valotusaika sallii.

#### Kalibrointi ja anturit

| Alikomento | Tarkoitus |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibroi heijastavuuskohteesta. Sitä etsitään enintään `--attempts` kehystä (oletus `30`); epätäydellinen kehys (GVSP-paketin menetys) kuluttaa yhden yrityksen, ei koko ajoa. Komento epäonnistuu, jos arvo on `no target detected in N attempts`, tai `no complete frame in N attempts: <last error>`, jos yhtään kehystä ei saapunut kokonaisuudessaan. |
| `lattice dls {--connect \| --spectrum \| --irradiance} --mac MAC [--filter NAME] [--camera SERIAL] [--json]` | DAQ-M:n spektri ja kaistojen voimakkuudet, luetaan Bluetoothin (BLE) kautta taustapalvelimen välityksellä. `--mac` on anturin oma BLE-MAC-osoite, ja sitä vaaditaan kaikissa kolmessa toiminnossa: oletusarvoa ei ole, koska jokaisella laitteella on oma osoitteensa (`daq pool-discover --only ble` luettelee ne); ilman sitä komento lopetetaan `1`. Jos toimintoa ei valita, se tulostaa ohjeensa. `--camera SERIAL` integroi tulostetut kaistavoimakkuudet kyseisen kameran oman mitatun spektrivasteen avulla suodatinmallin fleet-käyrän sijaan — RGB-kameroissa, kun kameran kalibrointipaketti on ladattu (kytketty) tai välimuistissa tällä tietokoneella. Otsikossa ilmoitetaan, kenen käyrää käytettiin: `Band magnitudes (camera <SERIAL>):` tai `Band magnitudes (model <filter>):`, jos kameralla ei ole omaa (ei RGB-kamera, tai pakettia ei ole ladattu tai välimuistissa) ja sen sijaan käytettiin vakiokäyrää. HTTP-yhteyden kautta `POST /api/dls/cli-read` ottaa vastaan `camera_serial`:n ja palauttaa `band_magnitudes_source`:n. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Sovelletaan vinjetointikorjausta olemassa oleviin kuviin, joista jokaisella on oman objektiivinsa tasokenttä (EXIF-mallitiedon perusteella). `--lens-model` sovelletaan kuviin, joiden mallitiedossa ei mainita kameraa; kuva, jonka mallitiedot ovat ristiriidassa tämän kanssa, ohitetaan. |

#### Monikamera (tilapäiset istunnot)

Tilapäinen komento avaa kamerat yhden ajon ajaksi ja sulkee ne, kun se päättyy: `multi-capture` ilman liitettyä pysyvää taulukkoa, `multi-stream` ja `multi-test` tässä, ja alla olevat `align-calibrate`, `align-apply`, `align-stream` ja `index --live`. Näihin kaikkiin sovelletaan seuraavia sääntöjä:

- **`--serials SN1,SN2,…` valitsee kamerat.** Vain ne avataan, ja ensimmäinen laukaisee laukaisimen (jos määritetty, myös `align-calibrate` ja `--reference`). Sarjanumero, jota ei ole verkossa, aiheuttaa suorituksen epäonnistumisen ennen kuin yhtään kameraa avataan. Ilman `--serials`:ää kaikki verkossa olevat kamerat avataan. Chlorosin jo hallussa oleva kamera (käyttöliittymä, `cam-connect`, `array-connect`) hylkää suorituksen riippumatta siitä, nimeääkö `--serials` sen vai ei: se kuuluu samaan verkkoon, jolle suoritus on suunniteltu. (`index --live` ilman `--serials` avaa sen sijaan profiilinsa käytössä olevat kamerat: katso [Hakemisto / Kasvillisuusmatematiikka](#index--vegetation-maths).)
- **Kamerat jakavat verkon samalla tavalla kuin `array-connect` jakaa sen.** Jokainen kamera on tahdistettu sen osuuden mukaan, jonka isäntäyhteys kuljettaa (pakettien välinen viive, ei kameran läpimenorajaa, jonka 31,25 MB/s:n alaraja neljä kameraa 1 GbE:llä jo ylittävät), joten neljä kameraa toimii 1 Gbps:n linkillä. Kun kamerat eivät voi kaikki lähettää kerralla hitaimmalla saavuttamallaan vauhdilla, ne laukaistaan yksi toisensa jälkeen, kukin omalla linkillään, ja komento tulostaa `Serial emit: the wire cannot carry these cameras at once, so each is triggered <ms> ms after the one before; their frames are not exposed together`. Kun edes yksi kamera kerrallaan ei mahdu mukaan, suoritus hylätään ennen kuin yksikään kamera ehtii lähettää: `array over-subscribes the wire: <n> cameras demand <x> MB/s but the collision-safe ceiling is <y> MB/s. …`, HTTP-koodi 409, lopetus `1`.
- **Jokaisen kameran on lähetettävä dataa, muuten suoritus epäonnistuu.** Kamera, joka ei lähetä yhtään kuvaa, on virhe: komento tulostaa `Error: … no frame from <n> of <total> cameras: SN:<serial> (<why>), …`, jossa `<why>` on kyseisen kameran hakuvirhe, sen jälkeen syyn osoittama viesti, ja lopettaa `1`. Kehys, joka saapui GVSP-pakettihäviöllä (`IncompleteBufferError: GVSP packet loss …`), tarkoittaa, että verkko ei kykene siirtämään kameroiden yhteenlaskettua kaistanleveyttä (`Every missing frame arrived with GVSP packet loss: …`): tarkista isäntäkoneen ja kytkimen uplink-yhteyden nopeus (`lattice network`), käytä jumbo-kehyksiä, tai avaa vähemmän kameroita komennolla `--serials`. Kamera, joka ei lähettänyt mitään, ei yleensä saa isäntäkameran laukaisusignaalia: tarkista sen M8-synkronointikaapeli ja suorita sitten komento `lattice multi-detect`. Kunkin komennon toiminta, kun kehysryhmästä puuttuu kamera, on kuvattu kyseisen rivin kohdalla.
- **Jokainen kamera palautetaan sellaisena kuin se löydettiin.** Kun suoritus päättyy, jokainen kamera saa takaisin sen, mitä sillä oli, kun suoritus avasi sen: `TriggerMode`, `TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, sekä `LineMode` / `LineSource` kohdissa `Line2` ja `Line3` (ja laukaisutulon häiriönpoisto, jonka taulukko antaa orjalinjalle, `LineFilterSelector` / `LineFilterWidth`); sen `PtpEnable` (ja sen kanssa asetetut `PtpSlaveOnly` / `PtpDomainNumber`); `ExposureAuto` / `GainAuto` (sekä `ExposureTime` / `Gain`, joissa se oli manuaalisesti asetettu); `TargetBrightness`, `ExposureAutoLimitAuto` / `ExposureAutoUpperLimit` / `ExposureAutoLowerLimit`, `GainAutoUpperLimit`, `ExposureAutoDamping`; `BalanceWhiteAuto` (ja jokainen `BalanceRatio`, jossa valkotasapaino oli manuaalinen); `BlackLevelAuto` / `BlackLevel`; `PixelFormat`, pikselien yhdistely, `Width` / `Height` / `OffsetX` / `OffsetY`; `GammaEnable` (ja `Gamma`, joissa gamma-korjaus oli päällä), `LUTEnable`, `ColorTransformationEnable`; `ReverseX` / `ReverseY`; `UserSetSelector`; paketin koko (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`); tahdistus, `DeviceLinkThroughputLimitMode` (ja `DeviceLinkThroughputLimit`, kun tila oli `On`),pakettien välinen viive `GevSCPD` ja käynnistysviive `GevSCFTD`, jotka jokainen kamera antaa; jokaisen virran kytkemä palanen data (`ChunkModeActive`, kukin `ChunkEnable` omassa `ChunkSelector`:ssään); `AcquisitionFrameRateEnable` (ja `AcquisitionFrameRate`, jossa aktivointi oli päällä — kun se on pois päältä, nopeus on vain luku -tilassa ja lukee eniten, mitä tahdistus sallii, joten se ei ole asetus). Vain eroavat arvot tallennetaan: kamera pysähtyi ensin, tahdistus ja kuvataajuus ennen valotusta, joka vaatii niiden kehäjakson (kamera rajoittaa `ExposureTime`:n yhteen kehäjakssoon). Asetuksella `--keep-settings` laukaisija ja synkronointilinjan johdotus palautetaan, kamera toimii vapaasti ja loput tiedot tallennetaan kameran omaan muistiin ajon asetusten mukaisesti; asetus `--clear-settings` palauttaa tehdasasetukset.
- **Valotus asetetaan ensimmäisen kehyksen tallentamista edeltävästä kohtauksesta.** Kuvaussarja ottaa synkronoituja kuvaryhmiä ja hylkää ne samalla, kun se asettaa kunkin kameran valotuksen ja vahvistuksen näiden kuvien perusteella. Tavoitteena on kameran kohteen keskimääräinen kirkkaus, jossa enintään 1 % kuvasta on leikkautunut; kohtaus, jossa on kirkkaita kohokohtia, jää hieman alle tavoitteen. Tämä vaatii yleensä 2–6 kuvaryhmää ja kestää enintään 5 s.
  - Kukin kamera kohdistaa suodattimensa avulla kohteeseen, josta liitetty matriisi aloittaa: TargetBrightness **40**monispektrikameralla (RGN, OCN, NGB) ja**60** RGB-kameralla. Tämä on puolet valotustavoitteesta 80 (RGB-kameralla kolme neljäsosaa), joten kirkkaan heijastuspaneelin leikkautuminen on epätodennäköisempää. `--brightness N` (0–255) asettaa sen sijaan jokaisen kameran kohdearvon (`multi-test`:llä ei ole `--brightness`:tä: sen automaattinen valotus on pois päältä). Backendin lokitiedostot nimeävät kunkin kameran valotustavoitteen: `[transient-ae] AE target per camera (filter default): SN:<serial> <target>, …`.
  - Valotus määritetään ensin, kameran oman automaattisen valotuksen rajan puitteissa. Vahvistus kattaa vain jäljelle jäävän osan.
  - Löydetyt valotusarvot ja vahvistusarvot säilytetään koko ajon ajan, joten jokainen `multi-stream`-kameran kuva otetaan samalla valotuksella. Kuvaus, jonka on seurattava muuttuvaa valaistusta, kuuluu liitettyyn taulukkoon (`array-connect`), jonka automaattinen valotus säätyy jatkuvasti.
  - `--exposure` ja `--gain` säilytetään täsmälleen annetuissa asetuksissa. Kun molemmat on annettu, yhtään kuvaa ei hylätä.
  - Kun kuvaussarja päättyy, kunkin kameran automaattinen valotus kytketään takaisin päälle.
  - Taustapalvelimen loki raportoi tuloksen: `[array-ae] exposure solved in <s> s (<n> warm-up groups): SN:<serial> <ms> ms <dB> dB mean <m>/<target>, …` tai `exposure NOT solved after …` syyn kera sekä `(not settled)` jokaisesta kamerasta, jossa näin ei tapahtunut. Tällöinkin kuvat otetaan.
  - `CHLOROS_TRANSIENT_AE_SETTLE_S` muuttaa 5 sekunnin raja-arvoa; `0` poistaa tämän käytöstä (katso [Ympäristömuuttujat](#environment-variables)).

**Kun Chlorosilla on hallussaan mikä tahansa kamera** (käyttöliittymä, `cam-connect` tai liitetty kameraryhmä), `multi-stream`, `multi-test`, `multi-detect`, `align-apply`, `align-stream` ja `index --live` kieltäytyvät suorittamasta, koska ne avaavat kaikki verkon kamerat ja toisen avauksen pidätetyn kameran avaaminen voi kaataa taustapalvelimen. (`index --live` kieltäytyy, kun kamera, jonka se yrittäisi avata, on pidätettynä: kyseessä on `--serials` tai sen profiilin kamerat, sekä mikä tahansa kamera profiilille, jossa ei ole määritelty yhtään kameraa.) Taustapalvelin vastaa HTTP-koodilla 409 `CAMERA_HELD`, luettelemalla jokaisen varatun kameran ja vapautuskomennot (`chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`), minkä jälkeen komento päättyy (`1`). Kun taulukko on liitetty, `array-capture --continuous` tallentaa sen `multi-stream`:n tilalle, ja `array-capture` (oletusarvoisesti kohdistettu) tallentaa kohdistetun joukon `align-apply`:n tilalle; `multi-capture` ja `align-calibrate` käyttävät liitettyä taulukkoa, joka sisältää niiden kamerat, sen sijaan että avaisivat ne.

| Alikomento | Käyttötarkoitus |
| --- | --- |
| `lattice multi-info [--serials SN1,SN2,…]` | Luettelee kaikki kamerat, joilla on synkronointirooli; komennolla `--serials` vain ne, ja ensimmäisenä komennolla `MASTER`. |
| `lattice multi-capture [--serials SN1,SN2,…] [--format FMT] [--jpeg-quality N] [--processing LEVEL] [--brightness N] [--no-daq] [--daq-tolerance-ms MS]` | Yksi synkronoitu kehys kustakin kamerasta. Tallentaa **oletusarvoisesti kaikki vientityypit**, kun pysyvä taulukko on kytketty, sekä sen lukeman, jota DAQ käytti ([Lukema, jonka DAQ tallentaa kaappauksen yhteydessä](#the-daq-reading-a-capture-saves)); tilapäinen, ilman matriisia toimiva varajärjestelmä on**vain debayeroitu** (suorita ensin `array-connect` muiden osalta). `--brightness` koskee vain varajärjestelmää: kytketty matriisi säilyttää oman automaattisen valotuksensa. Varajärjestelmä ei tallenna `.daq`:ää (`--no-daq` / `--daq-tolerance-ms` eivät vaikuta siinä), eikä `multi-capture` ota huomioon `--force-daq`:ää. Varajärjestelmässä kamera, joka ei toimita yhtään kuvaa, epäonnistuu tallennuksessa (`Error: multi-capture failed: no frame from …`, poistu `1`) eikä kyseisen kuvaryhmän tiedostoja tallenneta. |
| `lattice multi-stream [--serials SN1,SN2,…] [--fps F] [--count N] [--format FMT] [--jpeg-quality N] [--brightness N]` | Synkronoidut kehykset (väliaikainen), `--fps` oletusarvoisesti `3`, kunnes painetaan Ctrl+C tai `--count`-ryhmiä. Kehysryhmää, johon kamera ei toimittanut mitään, ei tallenneta, ja kolme peräkkäistä lopettaa streamin. Jos jokin ryhmä ohitettiin, suoritus päättyy tulostamalla `Incomplete: <n> frame group(s) not written, a camera delivered nothing (SN:<serial> x<count>, …)` sekä `Error: the stream stopped: …`, jossa mainitaan kamera, kun se lopetti streamin, ja ohjelma poistuu `1`X. Pysäyttää itsensä, kun asema täyttyy, tulostaa `Failed:` / `Disk full:` ja lopettaa ohjelman `1`; myös tiedostot, joita se ei pystynyt kirjoittamaan, lopettavat `1`. |
| `lattice multi-test [--serials SN1,SN2,…] [--count N] [--ptp-settle SECONDS]` | GPIO-synkronoinnin ajoitustesti (ohimenevä), `--count` oletusarvoiset `10`-laukaisijat. Tulostaa, miten verkko on jaettu (`Wire: each camera paced to <x> MB/s (<n> on a <y> MB/s ceiling, <link> Mbps link)`, ja rivi `Serial emit:`, kun kamerat laukeavat peräkkäin, jolloin aikaleimoihin sisältyy niiden välinen viive), yksi `Sync group: master SN:<serial> -> SN:<serial>, …`-rivi synkronointiryhmää kohti (`(no slaves: not synced)` kameralle, jolla ei ole synkronointikaapelia ja joka laukeaa itsenäisesti), sitten yksi rivi kutakin laukaisua kohti: `[  0] SN:<serial>=<timestamp> \| …`, ja `SN:<serial>=(no frame)` kameralle, joka ei lähettänyt mitään kyseisen laukaisun yhteydessä. Sitten rivi `Offsets from master SN:<serial> (<PTP / device (PTP off)> clock, <n> trigger(s)):` ja rivi `SN:<serial>: mean <±x> us, jitter <y> us` kutakin kameraa kohti, jokainen kamera suhteessa päälaitteeseen yhden kellosykliin sisällä: laukaisut, joiden aikaleimat sekoittavat kameroiden laitteiden kellot ja PTP:n (PTP pois päältä tai vielä lukittumassa) jätetään pois, ja `Note:` laskee ne. Jokainen puuttuva kehys aiheuttaa testin epäonnistumisen (`Error: multi-test failed: <k> of <n> frame groups incomplete (SN:<serial> missed <k>); …`, poistuminen `1`); kolme peräkkäistä epätäydellistä laukaisua lopettaa testin ennenaikaisesti. Testi epäonnistuu myös (exit `1`), jos alle kahdessa laukaisussa kaikkien kameroiden kellot ovat samassa tahdissa (`… the clocks cannot be compared`) tai jos kellot eivät ole koskaan yhteneväisiä: PTP-kellot ovat yli 1 ms:n päässä (plus mahdollinen sarjatasoisen viiveen) pääkellosta (`… PTP is not locked`), tai laitteiden kellojen välinen ero kasvaa laukaisujen välillä enemmän kuin kyseinen arvo. Se käynnistää PTP-grandmasterin, jos se ei ole käynnissä, kytkee jokaisen kameran PTP:n päälle (kuten `array-connect` tekee, riippumatta verkon rakenteesta) ja odottaa ennen ensimmäistä laukaisua, että jokaisen kameran PTP-kello lukittuu grandmasteriin ja vakiintuu: `PtpStatus` Slave on yhdellä grandmasterilla, alle 100 us:n päässä siitä ja vakaana 3 s ajan, ja jokainen kamera lukittuneena tällä tavalla, keskeytyksettä, 120 s ajan — noin 2 minuuttia virran kytkemisestä. Vasta lukittunut kello on jopa ~200 us:n päässä vakaasta kellosta noin 2 minuutin ajan sen jälkeen, kun se lukee Slave-tilaa, eikä kameran oma `PtpOffsetFromMaster` näytä sitä; 120 s:n kuluttua kamerat ovat ~10 us:n päässä vakaasta kellosta. Kamera on edelleen lukittunut aiemmasta testistä lähtien, jossa PTP jätettiin päälle, kuten tämä taustapalvelin sen havaitsi, laskee tuosta lukituksesta lähtien, joten peräkkäiset testit eivät odota uudelleen; kamera, jonka testi käynnistää, tai kamera, jonka PTP on päällä mutta jota tämä taustapalvelin ei ole nähnyt lukittuneen (käynnistetty muualla tai ennen taustapalvelimen uudelleenkäynnistystä), odottaa 120 sekuntia, ja molempien juuri lukittuneiden kameroiden joukossa ne odottavat edelleen omaa aikaansa. Kamera, joka poistuu orja-tilasta, ylittää 100 us:n rajan tai seuraa toista grandmasteria kolmen lukeman ajan peräkkäin (1,5 s) odotuksen aikana, aloittaa laskennan uudelleen; yksi tai kaksi tällaista lukemaa kirjataan (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`), ja laskenta jatkuu; grandmaster-tunnusta, jota ei voida lukea, pidetään tuntemattomana, ei muutoksena. Jos kaikkien kameroiden laskenta alkaa uudelleen yhtä aikaa (sekunnin sisällä), se johtuu grandmasterin ajan kulumisesta, ei kameroiden: taustapalvelimen lokissa lukee `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` listaa sen kohdassa `grandmaster_steps`, ja sen jälkeen umpeutuva odotus aloittaa virheensä kohdassa `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Rajaa ei pidennetä sen vuoksi. `--ptp-settle SECONDS` asettaa vakiintumisaikaksi ajolle (0–600; `0` ilman vakiintumisodotusta). Odotus on rajattu vakiintumisaikaan plus 90 s (oletusarvoisesti 210 s, vähintään 90 s), ja komento odottaa taustapalvelinta 240 s plus vakiintumisaika (oletusarvoisesti 360 s). Se tulostaa `PTP: locked after <s> s (each camera locked <settle> s, already locked: SN:<serial>, …; from the grandmaster: SN:<serial> <±x> us, …)`, ja taustapalvelimen lokissa on 5 sekunnin välein rivi, jossa on kunkin kameran tila ja siirtymä, kuinka kauan se on ollut lukittuna ja kuinka kauan lukitusta on jäljellä: `[array-sync] PTP at <s> s: SN:<serial> Slave <±x> us for <s> s (<s> s left), … -- <s> s left, the wait gives up at <s> s`. Jokainen kamera käynnistyy PTP:n ollessa pois päältä; tämän jälkeen kellot lukittuvat noin 10 sekunnissa. Sen jälkeen kukin kamera palautetaan tilaan, jossa testi sen löysi — sen PTP, automaattinen valotus ja vahvistus, jotka testi pitää pois päältä suorituksen ajan, sekä muut asetukset (ellei `--keep-settings`) — joten kamera, jonka PTP on jätetty päälle (`lattice configure -s <serial> --set PtpEnable=True` tai testiajo, jossa on `--keep-settings`), pysyy synkronoituna, ja kun tämä taustaprosessi on havainnut sen synkronoituneen vakiintumisaikana, myöhempi testi ei odota sitä. Jos kellot eivät vakiinnu odotuksen aikana, testi epäonnistuu ennen laukaisua (`PTP: NOT locked …`, poistuminen `1`) ja kertoo, mitä on tehtävä: kamera, joka ei ole koskaan lukittunut (`PtpStatus Listening`), tarkoittaa, että grandmaster ei saavuta sitä (`time-sync status`); kamera, joka on jatkuvasti pudonnut pois, nimetään sen mukaan, kuinka kauan se on ollut lukittuna (`… had not been locked for the 120 s settle time … (SN:<serial> locked <s> s, <s> s to go)`); kellot yhä synkronoituvat: tarkista `time-sync status`, ja jotta ne lukittuisivat ennen testiä, kytke PTP päälle ja jätä se päälle (`lattice configure -s <serial> --set PtpEnable=True` jokaiselle kameralle tai suorita testi komennolla `--keep-settings`; testi palauttaa löytämänsä asetukset, joten se pysyy päällä), ja suorita sitten testi. |
| `lattice multi-detect [--line LINE] [--json]` | Etsi M8-synkronointilinjalta kamerat ja oletusarvoinen pääkamera: RGB-kamera (mallin loppuosa `-FRGB`), jos se on kytketty kaapelilla, muuten ensimmäinen kaapelilla kytketty kamera sarjanumeron mukaan. M8-kaapelit yhtyvät yhteen liitoskohtaan, joten **mikä tahansa kytketty kamera voi toimia pääkamerana** — painettu `MASTER` on oletusarvo, ei kytkentätieto. Jokainen kamera ohjaa vuorollaan linjaa, ja muiden pulssia tarkistetaan; kameraa, joka on jo löydetty kytketyistä, ei tutkita uudelleen, joten täysin kaapeloidun ryhmän tarkistaminen kestää yhden kierroksen. Jos `--line` puuttuu, se etsii `Line2`:ää, sitten `Line3`:ää. Tulostaa `MASTER: SN:…`:n, yhden `SLAVE: SN:…`:n kutakin muuta kaapeloitua kameraa kohti sekä `(probed on <line>)`:n; `--json` lisää lopullisen tuloksen (`master`, `slaves`, `no_cable` — kamerat, joihin pulssi ei saavuttanut — `line` sekä kamerakohtainen `results`). Kun se on valmis, jokaiselle kameralle palautetaan sen aiemmat laukaisija- ja synkronointilinjan asetukset (samat solmut kuin yllä). |

#### Kohdistus

| Alikomento | Käyttötarkoitus |
| --- | --- |
| `lattice align-calibrate [--serials SN1,SN2,…] [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N] [--brightness N]` — plus-ilmaisin-/sovitinvivut `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, RANSAC-säätimet `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, monikehysyhdistelmä `[--averaging mean\|median\|inlier_weighted]`, geometriset rajoitukset `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, avaruudelliset rajoitukset `[--roi X0,Y0,X1,Y1] [--mask PATH]` jaorja-ohitukset `[--per-cam-override SN:KEY=VALUE]` (toistettavissa) | Laske kohdistusprofiili reaaliaikaisista kameroista. Oletusarvot: `--method orb`, `--model rigid`, `--frames 3`, `--save alignment_profile.json` (siinä kansiossa, josta ohjelma suoritetaan), `--max-features 2000`, `--ratio-threshold 0.75`, `--matcher bf`, `--knn-k 2`, `--ransac-threshold-px 3.0`, `--ransac-iters 2000`, `--ransac-confidence 0.99`, `--averaging mean`. (`project align calibrate`:llä on omat oletusasetuksensa; katso [sen taulukko](#project-align-calibrate-options).) `--prefilter`:n oletusarvona on `gradient` (reunakartta; vastaa GUI-/matriisikohdistinta — reunat säilyvät spektrikaistojen yli). `--matcher flann` on kannattava, kun piirteitä on yli ~5000; `--averaging median` kestää yhden huonon kuvan, `inlier_weighted` painottaa vastausten lukumäärän mukaan; `--lock-scale` projisoi lähimpään kiertoon (ei skaalausta), `--lock-axis` nollaa yhden siirtokomponentin; `--mask` koskee kaikkia kameroita (käytä `--per-cam-override`:ää kamerakohtaisiin asetuksiin, esim. `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` kieltäytyy tallentamasta kalibrointia, jonka uudelleenprojektioinnin RMS ylittää raja-arvon. `--method manual` ottaa pisteensä `--points PATH`:stä: JSON-objekti `{"<serial>": [[x, y], ...], ...}`, joka antaa kunkin kameran pikselikoordinaatit samoista piirteistä, mukaan lukien vertailukamera (vähintään 2 kappaletta kameraa kohti); jokainen `--frames` ratkaistaan näistä pisteistä. Kun kamerat sisältyvät liitettyyn taulukkoon, kalibrointi käyttää kyseisen taulukon reaaliaikaisia kehyksiä, ja profiilista tulee myös taulukon kohdistus sen ottamille kuville. Muussa tapauksessa se avaa [väliaikaisen taulukon](#multi-camera-transient-sessions): kamera, joka ei lähetä kuvaa missään `--frames`-kohdassa, epäonnistuu kalibroinnissa (poistuu `1`-kohdasta nimeämällä sen), eikä profiilia tallenneta. |
| `lattice align-apply --profile PATH [--serials SN1,SN2,…] [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N] [--brightness N]` | Kaappaa yhden kohdistetun monikaistakuvan. `--bit-depth` sovittaa oletuksena kameran; `--no-crop` säilyttää koko kehyksen (täytetään mustalla); `--interpolation` (oletus `linear`) ja `--border-mode`/`--border-value` (oletusarvo `constant`/0) säätelevät CPU-vääristymää — GPU-polku on joka tapauksessa bilineaarinen. Avaa [väliaikaisen taulukon](#multi-camera-transient-sessions): kamera, joka ei lähetä yhtään kehystä, epäonnistuu tallennuksessa (poistuu `1`:stä nimeämällä sen), eikä mitään kirjoiteta. |
| `lattice align-stream --profile PATH [--serials SN1,SN2,…] [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N] [--brightness N]` | Virtaan kohdistetut monikaistaiset kehykset (samat warp-säätimet kuin `align-apply`), `--fps` oletusarvo, `3`. Avaa [transient-taulukon](#multi-camera-transient-sessions): kehysryhmää, johon kamera ei toimittanut mitään, ei tallenneta, ja kolme peräkkäistä kehystä lopettavat streamin. Jos jokin ryhmä ohitettiin, suoritus päättyy tulostamalla `Incomplete: <n> frame group(s) not saved, a camera delivered nothing (SN:<serial> x<count>, …)` ja `Error: …`, ja lopetetaan komennolla `1`. Myös virrassa, joka päättyy mihin tahansa muuhun virheeseen, tulostetaan komennot `Error: …` ja lopetetaan komennolla `1`. |
| `lattice align-info --profile PATH [--json]` | Näytä profiilin tiedot. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Muuta kerrosten järjestystä. |

#### Hakemisto / Kasvillisuuden matematiikka

```bash
# Offline: compute NDVI from an aligned multi-band TIFF (an RGN camera's bands)
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: open the cameras, capture one aligned set with a profile made by
# `lattice align-calibrate`, and compute the index from it
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --save-multiband -o output/
```

**`--channel` liittää esiasetuksen symbolit kaistoihin** (NDVI: `red`, `nir`), yksi `--channel SYMBOL=BAND` kutakin.

- **`--live`: vain symbolille, johon pino ei osaa vastata.** Kuten `--input`:n tapauksessa, esiasetuksen symboli, jonka jätät pois, ottaa sen yhden kaistan pinosta (profiilin käytössä olevien kameroiden kaistat), jonka nimi alkaa kyseisellä symbolilla. Tämä tarkistetaan ennen kuin mikään kamera avautuu: symboli, johon mikään kaista tai useat kaistat eivät vastaa, hylätään tällöin, ja ne nimetään, ja ohjelma poistuu `1`:sta. Mukautettu `--formula` tarvitsee jokaiselle symbolille oman `--channel`:n.
- **`--live`-tuotteet ilmoittavat, kenen ne ovat.** `index.tif` ja `index_LUT.png` sisältävät pinon kameran tiedot (merkki, malli, sarjanumero), sen kuvausajan (vertailukameran valotusaika, UTC) ja objektiivin, ja molemmat nimeävät indeksin omaksi kaistakseen (`XMP-Camera:BandName` `NDVI`; värillinen PNG-tiedosto nimeää indeksin, ei sen värikanavia), kuten `--input`-tuote tekee. Useiden kameroiden kaistoista koostuva indeksi nimeää sen sijaan jokaisen niistä, ja sen `Model` on `LATTICE` — katso [Useista kameroista tehdyt tiedostot](#files-made-from-several-cameras).
- **`--input`: vain symbolille, johon tiedosto ei osaa vastata.** Jos jätät symbolin pois, tiedostosta valitaan se kaista, jonka nimi alkaa kyseisellä symbolilla: `red` ottaa `Red_660`:n tai `Red_660_RGN`:n, `nir` ottaa `NIR_850`:n. Nimet ovat peräisin itse tiedostosta: pinon omasta kaistaluettelosta tai, Chloros-säteily-/heijastuskuvauksessa tai `process`-tuotteessa olevista sen `Chloros:Band{R,G,B}Name`- ja `XMP-Camera:BandName`-tunnisteista. Siten `lattice index --input <reflectance capture> --preset NDVI` ei tarvitse `--channel`:ää.
- **Kaksi kaistaa, jotka vastaavat yhtä symbolia, eivät tule arvattaviksi** (9-kamerapinossa on kaksi NIR-kaistaa): nimeä se. Symboli, johon mikään ei vastaa, hylätään tunnuksella `channel_map missing entries`, poistu `1`.**Pinoon kuuluvien kaistojen nimet** sisältävät kameran suodattimen: M3C-kameran kaistat ovat `<band>_<filter>` — `NIR_850_RGN`, `Green_550_RGN`, `Red_660_RGN` (RGN); `NIR_808_OCN`, `Cyan_490_OCN`, `Orange_615_OCN` (OCN); `Blue_475_RGB`, `Green_550_RGB`, `Red_625_RGB` (RGB); `Blue_475_NGB`, `Green_550_NGB`, `NIR_850_NGB` NGB:llä — ja M3M-kameranyksikaistainen on `Mono_<filter>` (`Mono_660`, `Mono_850`). `lattice align-info --profile PATH` luettelee profiilin kaistat.

Täydellinen lippusarja: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (toistettavissa), `--capture-level raw|debayered|radiance|reflectance|unknown` (ohittaa lähde-TIFF-tiedostossa tallennetun tallennustason; oletus: luetaan TIFF-metatiedoista, ja jos niitä ei löydy, käytetään Chloros-tiedoston -arvoa 001565, joten heijastavuusmittaus ilmoittaa arvon `reflectance`), `--output PATH` / `-o` (offline: polku; kuvatiedostotunnisteen kanssa tiedostot nimetään sen juuresta — `-o ndvi_FRGN.png` kirjoittaa tiedostot nimillä `ndvi_FRGN.tif` ja `ndvi_FRGN_LUT.png` — ja ilman laajennusta se toimii etuliitteenä, kuten `-o ndvi` -&gt; `ndvi.tif`, `ndvi_LUT.png`; oletusarvo syötteen vieressä, kuten `<input name>_<preset>`; kun on `--live` mutta ei `-o`, kansio `output/` sen kansion alla, josta komento suoritetaan), `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Tiedostossa `--live` kohdistuksen vääristymän säätimet koskevat myös seuraavia tiedostoja: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`, `--brightness N` (kameroiden automaattisen valotuksen kohde, katso transienttisäännöt), `--serials SN1,SN2,…`. `--live` avaa [transienttijonon](#multi-camera-transient-sessions): kamera, joka ei toimita yhtään kuvaa, aiheuttaa ajon epäonnistumisen (poistutaan `1`:stä, nimeämällä se), eikä pinoa tai indeksiä tallenneta. Se avaa `--serials`:n (ensimmäinen laukaisee liipaisimen), muuten profiilin kamerat, joiden `layer_order`-merkintä on käytössä, viitekamera edellä — viitekamera jopa silloin, kun sen kaistat on poistettu käytöstä, koska muut on kohdistettu siihen ja se laukaisee liipaisimen. Kameraa, jonka kaistat on poistettu käytöstä, ei koskaan kalibroida, joten profiilissa poistettu RGB-kamera ei enää aiheuta heijastusindeksin epäonnistumista (`no-absolute-radiance-scale`). Profiili, jossa ei nimetä kameroita, avaa kaikki verkon kamerat, kuten aiemminkin.

Jokainen lomake toimii taustapalvelimella, joten sen on oltava käynnissä: käynnistä se Chloros-käyttöliittymästä tai komennolla, jonka CLI tulostaa, kun se ei pääse siihen käsiksi. Offline-tilassa taustapalvelu lukee `--input`:n ja kirjoittaa itse float-TIFF-tiedoston sekä värillisen PNG-tiedoston, joten suhteellinen `--input` tai `-o` viittaa siihen kansioon, josta komento suoritetaan (katso [Käytännöt](#conventions)). Molemmat offline-tuotteet sisältävät syötteen kameratiedot, kuten `process`-indeksivienti: merkki / malli / sarjanumero, GPS, objektiivitunnisteet, kuvausajankohta (syötteen oma; jos sen EXIF-tiedoissa ei ole mainintaa, tallennettu valotushetki tai tiedostonimen aikaleima, UTC-aikaa) sekä MAPIR-kalibrointitunnus — ei koskaan raw-DN-kalibrointia, jota ei sovelleta hakemistoon — sekä `Chloros:ProcessingLevel=index` ja hakemistolauseke; float-tyyppinen TIFF-nimi viittaa yhteen kaistaan, esiasetukseen. `--list-presets` ja `--list-gradients` tulostavat taustaprosessin esiasetukset ja gradienttitaulukot. Kaava, jonka taustakäsittely hylkää, tai `--channel`-kaista, jota TIFF-tiedostossa ei ole, poistuu `1`:stä taustakäsittelyn syyn kanssa.

> **`--channel`-symboleissa on kirjainkokoero.** Symbolin puolen on vastattava esiasetuksen kanavanimiä tarkalleen (esiasetuksissa käytetään pieniä kirjaimia, esim. NDVI = `red`,`nir` — tarkista `--list-presets`), ja kaistapuolen on vastattava kaistan nimeä kohdistetussa pinossa (tai oltava 0-pohjainen kaistaindeksi offline-tilassa). `--channel red=Red_660_RGN --channel nir=NIR_850_RGN` toimii; `--channel RED=660` epäonnistuu ja aiheuttaa `channel_map missing entries` -virheen.

#### `lattice index` — monikaistapinojen peitteet

Kohdissa `--capture-level radiance` tai `reflectance` kunkin kameran 3×3-sekoituksen purku voi rajoittaa kaistan arvoon 0 (katso [Unmix-rajoitusmaski](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on)). Monikaistapino sisältää useiden kameroiden kaistat, ja kukin kamera rajoittaa oman kaistansa, joten `--live --save-multiband` kirjoittaa **yhden maskin kutakin lähdekameraa kohti** pinon viereen:

```
output/
├── index_multiband.tif
└── clamp_masks/
    ├── index_multiband_SN<serial A>-<filter A>_clampmask.tif
    └── index_multiband_SN<serial B>-<filter B>_clampmask.tif
```

- Jokainen maski on yksikaistainen `uint8` **pinoon** kuuluvalla pikseliristikolla (saman kohdistusmuunnoksen ja rajauksen kautta), ja siinä on kyseisen kameran omat bitit: bitti 0 = sen R-kanavan kaista, bitti 1 = G, bitti 2 = B. Kameran kanavasta *c* otettu pino-kaista rajataan siellä, missä kyseisen kameran bitti *c* (R = 0, G = 1, B = 2) on asetettu.
- Kunkin kaistan upotettu alkuperätieto nimeää sen kameran maskin (`unmix_clamp_mask`, suhteessa pino-kansion nimiin, tai `none`) sekä luettavan bitin (`unmix_clamp_mask_bit`).
- Kamera, jonka unmix-käsittely ei rajoittanut mitään, ei saa tiedostoa; mono- tai RGB-kamera ei suorita unmix-käsittelyä eikä se myöskään saa tiedostoa. CLI tulostaa jokaisen maskin pinoon (`clamp mask SN…: …`); vastaus listaa ne muodossa `clamp_masks`.
- Saman ajon indeksi-TIFF- ja PNG-tiedostot lasketaan samasta pinosta, samalla ruudukolla, joten maskit kuvaavat niitäkin. `--capture-level raw` ei suorita unmix-käsittelyä eikä kirjoita maskia.
- Pino ja indeksi sijaitsevat suoraan kansiossa `--output`, joten maskit menevät kansioon `--output`:n omaan kansioon `clamp_masks/` — eivät koskaan nimeämäsi kansion yläpuolelle. Tuo kansio sisältää erilaisia tuotetyyppejä (pino, indeksi, värillinen renderöinti): valitse siitä tarvittavat tiedostot, älä luovuta koko kansiota fotogrammetriaohjelmistolle.

#### Pysyvät yhteydet (Smart-Prep, GUI-vastaava työnkulku)

Nämä komennot pitävät kamerat auki taustapoolissa eri CLI-kutsujen välillä.

| Alikomento | Tarkoitus |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Lisää yksi kamera pooliin (yksi kamera, ei ryhmää). [Yleiset asetukset](#common-options-shared-by-most-subcommands) `--exposure`, `--gain`, `--preset`, `--pixel-format`, `--width` / `--height` ja `--packet-size` (`standard` = 1500, `jumbo` = 9000 tai tavuja; `auto`-anturit) otetaan käyttöön yhteyden muodostamisen yhteydessä (`--exposure`- tai `--gain`-kytkimet, jotka kytkevät automaattitilan pois päältä). Se palaa, kun kameran datavirta on alkanut (odottaa sitä noin 15 sekuntia, sitten 10 sekuntia ensimmäisen täydellisen puskurin saapumista) ja tulostaa `stream: streaming` tai `stream: no_frames -- connected, no frames yet: …`, kun datavirta ei toimittanut mitään — kun kyseessä on kamera, joka käyttää jumbopaketteja, ja ohjeena on muodostaa yhteys uudelleen viestillä `--packet-size standard`. Kun yksi kamera muodostaa yhteyden (ensimmäinen yhteys lataa sen kalibroinnin, noin 70 sekuntia), toinen kameraa tarvitseva komento odottaa sitä enintään 5 sekuntia, sitten epäonnistuu HTTP-virheellä 503 (`CAMERA_POOL_BUSY`), jossa mainitaan muodostava yhteys; suorita komento uudelleen hetken kuluttua. `lattice status` ja kameraluettelo eivät koskaan odota: yhteyden muodostamisen keskivaiheessa oleva kamera näkyy muodostamassa yhteyttä, ja sen vaihe näkyy. |
| `lattice cam-disconnect [--serial SN] [--all] [--keep-settings \| --clear-settings]` | Vapautus. Kamera palautetaan tilaan, jossa se oli yhteyden muodostamisen yhteydessä (laukaisijajohdotus, PTP, asetukset); `--keep-settings` tallentaa sen asetukset sen sijaan omaan muistiinsa, `--clear-settings` nollaa kyseisen muistin tehdasasetuksiin (katso [Kameran muisti](#camera-memory----keep-settings----clear-settings)). Ryhmään kuuluva kamera vapauttaa koko ryhmän. |
| `lattice cam-list` | Luettele poolissa olevat kamerat. |
| **`lattice array-connect`**|**Yhdistä pysyvä synkronoitu ryhmä (SUOSITELTU aloituskohta).** Suorittaa koko GUI-käyttöliittymän älykkään valmisteluvirran. Noin minuutti 9 kameralle; heti kameroiden käynnistymisen jälkeen se voi kestää useita minuutteja. Odotuksen aikana se tulostaa jokaisen vaiheen, johon taustaprosessi pääsee, kuten `    <s> s: <step>` (esimerkiksi `    41 s: open SN:214000535 (4 of 9)`), ja taustapalvelimen lokissa on yksi aikamerkintä kutakin vaihetta kohti (`[array-connect <id>] <step>: <s> s …`) sekä viimeinen rivi, jossa on hitaimmat vaiheet. Se odottaa enintään 600 s. Jos yhteyttä ei ole muodostettu siihen mennessä, taustapalvelin peruuttaa sen: jokainen avattu kamera palautetaan tilaan, jossa se löydettiin, ja suljetaan, komento tulostaa `Error:` saavutetun vaiheen kanssa ja lopettaa `1`, eikä yhtään kameraryhmää ole kytketty (HTTP-virhe 504 `CLIENT_DEADLINE_PASSED`). Ctrl+C keskeyttää yhteyden muodostamisen samalla tavalla (`Interrupted: …`); jos matriisi oli jo muodostanut yhteyden, rivillä ilmoitetaan siitä ja mainitaan komento `array-disconnect`, joka vapauttaa sen. Komento, joka keskeytetään odotuksen aikana (`kill -9`) jättää yhteyden purkamatta, kun taustapalvelin ei ole saanut siitä ilmoitusta 30 sekunnin kuluessa. Järjestelmä, joka on käynnissä mutta jolla ei vielä ole synkronoitua kehysryhmää, ilmoitetaan silti kytketyksi, ja sen virta on `no_frames`. |
| `lattice array-disconnect [--array-id ID] [--all] [--keep-settings \| --clear-settings]` | Vapauta taulukko. Jokainen kamera jätetään vapaasti toimimaan (`TriggerMode` `Off`), ja kaikki muu säilyy sellaisena kuin yhteys sen löysi — sen laukaisija- ja synkronointilinjan asetukset (päälaitteen`TriggerSource` palautetaan entiselleen, sen synkronointilinja palautetaan tuloksi), sen PTP sekä istunnon muuttamat kameran asetukset — joten jäsenelle annettu yksittäisen kameran komento toimii välittömästi. `--keep-settings` tallentaa kunkin kameranasetukset sellaisina kuin istunto ne jätti kameran omaan muistiin; `--clear-settings` palauttaa kyseisen muistin tehdasasetuksiin. Katso [Kameran muisti](#camera-memory----keep-settings----clear-settings). |
| `lattice array-list` | Luettelee liitetyt kameraryhmät. |
| `lattice array-status [--array-id ID]` | Reaaliaikainen kuvataajuus (fps), PTP, viimeisin virhe. `ptp_enabled` on `True` vain, kun jokaisella jäsenellä on PTP käytössä; `ptp_off:` nimeää jäsenen, jolla se ei ole käytössä. `state:`-rivi lukee `restarting (applying settings)` asetusten uudelleenkäynnistyksen aikana, `recovering (…)` kun virran menettänyt jäsen avataan uudelleen (katso huomautus alla), `stalled (…)`, kun tallennussilmukka on käynnissä mutta ei julkaise kehysryhmää, ja `stopped (…)`, kun tallennussilmukka on luopunut (kytke matriisi uudelleen). |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--orient] [--index\|--no-index] [--no-daq] [--daq-tolerance-ms MS] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Yksi synkronoitu tallennus live-ryhmästä — Yksittäinen / Jatkuva / Aikaväli / Nopein. **Oletusarvo on `all`**(yksi tiedosto kutakin soveltuvaa vientityyppiä kohti kameraa kohden),**täsmäytetty** sovelluksen tallennuksen mukaisesti (`--no-aligned` natiivitiedostoille). Ohitetut kamerat (esim. RGB, joka on jätetty pois säteily-/heijastusarvoista) raportoidaan koodilla `Skipped: SN:<serial> (<reason>)`; käytetyn tallennuksen lukema DAQ:sta tallennetaan rinnalla, mukaan lukien pelkkä raakatallenne, ja raportoidaan koodilla `DAQ: <path>` ([DAQ tallentaa tallennuksen lukeman](#the-daq-reading-a-capture-saves)). Katso [Tallennustilat, tallentimet ja offline-uudelleenkäsittely](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only] [--expr EXPR]` | Tallenna reaaliaikainen yhdistetty indeksinäkymä videoksi/GIF-tiedostoksi (valvontataso; ilman avointa GUI-näkymää taustaprosessi renderöi matriisin NDVI-arvon, tai `--expr`). |
| `lattice array-burst [--duration S] [--max-frames N] [--orient] [--build] [--products …]` | Raaka-Bayer-sarjakuvaus korkealla kuvataajuudella (analyysilaatuinen; jälkikäsitellään offline-tilassa). Kuten `array-record` ja `array-capture --continuous`, se ilmoittautuu taustapalvelimelle jokaisen kyselyn yhteydessä: lopetetaan pysäyttämättä (`kill -9`), taustapalvelin pysäyttää sen 30 sekuntia viimeisen yhteydenoton jälkeen (`stop_reason` `client_gone`; sarjakuvauksen manifesti viimeistellään ja yhdistettyjen indeksien luominen aloitetaan, kuten pysäytyksen yhteydessä tapahtuu) ja taulukko pysyy yhteydessä. |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Käsittele tallennettu raakabursti uudelleen kalibroiduksi videoksi (videoiksi). |

> **Kamera, jonka virta katkeaa tai yhteys katkeaa, palaa takaisin itsestään.** Kun jäsen lakkaa vastaamasta (kytkimen virran katkaisu ja uudelleenkäynnistys käynnistää uudelleen kaikki sen virralla toimivat kamerat), ryhmä odottaa sitä, avaa yhteyden uudelleen, kun se vastaa jälleen, ja palauttaa sille `array-connect`:n asettamat arvot: PTP, laukaisija ja synkronointilinja, verkon tahdistus, pikselimuoto ja kuvakoko sekä automaattinen valotus. Sen jälkeen se jatkaa lähetystä ilman uutta `array-connect`-asetusta. Sillä välin `array-status` näyttää `state: recovering (...)`:n ja `last_error` kertoo, mitä kameraa se odottaa; tämän jälkeen se tulostaa `re-opened:`:n. Uudelleenkäynnistynyt kamera palaa PTP-tilan ollessa pois päältä, joten kunnes se avataan uudelleen, `ptp_enabled` on `False` ja `ptp_off:` nimeää sen.

> **Pidätettyä kameraa ei koskaan avata toista kertaa.** Toinen avaus pysäyttää pidättäjän datavirran ja kirjoittaa sen asetukset uudelleen, mikä voi kaataa taustapalvelimen. Joten kamerassa, jota pool pitää hallussaan (GUI tai `cam-connect`), taulukon jäsenessä tai kamerassa, jota taustatehtävä pitää hallussaan (käynnissä oleva `lattice continuous`, `multi-stream` tai `align-stream`, joko oma tai toisen asiakkaan), `lattice continuous`, `lattice capture`, `cam-connect`, `power --sleep`, `calibrate`, yhden-kuvainen `white-balance` ja `presets --apply` hylkäävät pyynnön HTTP-virheellä 409 `CAMERA_HELD`, mainiten haltijan ja vapautuskomennon (`chloros-cli lattice cam-disconnect -s <SN>`, `chloros-cli lattice array-disconnect --array-id <ID>` tai tehtävän osalta sen tunnus ja kuka sen aloitti) sekä lopetus `1`. Sama pätee kameraan, jota toinen komento käyttää parhaillaan (`multi-test` tai `multi-capture`, joka avaa kameransa, toinen kertakäyttöinen), jokaisen tässä mainitun komennon kohdalla: virheilmoitus kehottaa odottamaan kyseisen komennon päättymistä. `lattice power` ilman `--sleep` lukee haltijan oman kahvan kautta, jos sellainen on (pooli, taulukko, työn avoin kamera), ja se hylätään, jos kamera on toisen komennon käytössä. Jokainen `CAMERA_HELD`-vastaus sisältää koodin sekä `code`:ssä että `error_code`:ssä. Komennot, jotka avaavat kaikki kamerat, löytyvät kohdasta [Monikamera (väliaikaiset istunnot)](#multi-camera-transient-sessions).

##### `array-connect`-asetukset

| Lippu | Oletus | Kuvaus |
| --- | --- | --- |
| `--serials SN1,SN2,…` | tunnista kaikki LATTICE-kamerat automaattisesti (vaatii vähintään 2) | Ensimmäinen sarjanumero on MASTER. Jos asetusta ei määritetä, tunnistus suodatetaan LATTICE (M3C / M3M) -malleihin ja kaikki ne kytketään. |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO-synkronointilinja. |
| `--target-fps F` | auto | Master-laukaisun laukaisutaajuus. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Ohita tasovalitsin. |
| `--wire-ceiling-mbps MB_PER_S` | tunnistetaan automaattisesti | **Isäntäkoneen jatkuva kaistanleveys, MB/s — luku, johon koko matriisin allokointi perustuu.** Pienennä sitä, kun array ilmoittaa GVSP-vioittuneista kehyksistä: automaattinen arvo johdetaan verkkokortin ilmoittamasta linkkinopeudesta, joka yliarvioi USB-sovittimet, kapeat PCIe-kaistat ja kuormitetut jaetut verkot. Tallennetaan projektin array-kaappauslohkoon, joten uudelleen avaaminen / CLI-komento / SDK-uudelleenyhteyden muodostaminen palauttaa sen. Katso [Järjestelmän kunto](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | ei mitään (koko kenno) | Laitteistopohjainen binning (aina keskiarvo). |
| `--width N` / `--height N` / `--pixel-format FMT` | koko anturi; kamerakohtainen formaatti | Jokaiselle kameralle määritetty nimenomainen kehyskoko tai pikseliformaatti koko anturin ja kamerakohtaisen formaatin valinnan (Mono12 / BayerRG12) sijaan. |
| `--no-recommend` | pois käytöstä | Ohita verkkoanalyysivaihe (tällöin kukin kamera säilyttää nykyisen kuvakokonsa sen sijaan, että se asetettaisiin koko kennon kokoon). |
| `--no-ptp` | pois käytöstä | Poista PTP käytöstä (kameroiden väliset aikaleimat eivät tällöin ole **vertailukelpoisia**). |
| `--packed {auto,on,off}` | `auto` | Lähetä 12-bittisiä kehyksiä pakattuina ja pura ne tällä tietokoneella — enemmän kehyksiä sekunnissa täysikokoisella anturilla, identtiset tiedot. `auto` pakkaa vain, kun se on hyödyllistä ja tämän tietokoneen kapasiteetti riittää; `off` ei pakkaa koskaan. Katso [Pakattu 12-bittinen siirto](#packed-12-bit-transport---packed). |
| `--packet-size {auto,jumbo,standard,N}` | `auto` | GVSP-paketin koko jokaiselle kameralle ryhmässä. `auto` antaa jokaisen kameran tutkia verkkoa; `jumbo` = 9000, `standard` = 1500 tai tavuina ilmaistu koko rajoittaa sitä. Ennen versiota 1.3.0 `array-connect` hyväksyi tämän lipun ja jätti sen huomiotta. |

> **Oletusarvoisesti täysi anturi.** Ilman `--width` / `--height` / `--binning` /
> `--pixel-format`, `array-connect` asettaa jokaisen kameran käyttämään koko anturia — ei
> binningiä, ei ROI-rajauksia — ja antaa kuvataajuuden laskea siihen, mitä tämä tietokone ja
> verkko pystyvät tarjoamaan, sen sijaan että kuvia pienennettäisiin kuvataajuuden ylläpitämiseksi.
> Verkkoanalyysirivi näyttää ennustetun kuvataajuuden, ja seuraavalla rivillä luetellaan
> nopeammat vaihtoehdot (pikselien yhdistely tai pienempi ROI) sekä niiden ennustetut kuvataajuudet; ohita
> `--binning` tai `--width`/`--height` valitaksesi yhden. Connect ei koskaan suorita pikselien yhdistämistä tai rajausta
> itsestään.

##### Pakattu 12-bittinen siirto (`--packed`)

Täydellä anturikapasiteetilla matriisin kuvataajuus määräytyy sen mukaan, kuinka nopeasti kukin kamera voi lähettää
kuvansa. 12-bittiset formaatit (`BayerRG12`, `Mono12`) siirretään yleensä 16-bittisinä
sanoina; **pakattu** siirto lähettää samat 12 bittiä 1,5 tavuna pikseliä kohti ja
purkaa ne tällä tietokoneella, joten kunkin kehyksen lähettäminen vie 25 % vähemmän aikaa —
noin **33 % enemmän kehyksiä sekunnissa täydellä anturitilalla** (mitattu: 10,7 → 14,0 fps
9-kamerajärjestelmässä). **Tallennetut tiedot ovat identtiset**: samat arvot, samat
16-bittiset tiedostot, samat metatiedot.

- `auto` (oletus) pakkaa vain, kun siitä on hyötyä **ja** tämä tietokone pystyy
  käsittelemään purkamisen. Yhteyden muodostamisen yhteydessä taustapalvelin mittaa oman purkunopeutensa, prosessorien lukumäärän,
  nykyisen prosessorikuormituksen ja verkkokapasiteetin, joten työpöytätietokone, jolla on ylimääräisiä ytimiä, pakkaa
  täydellä anturikapasiteetilla, kun taas kuormitettu tai vähätehoinen tietokone suoratoistaa purkamattomana. Pienet ROI-alueet ja yhdistetyt kuvat tuovat
  vähän.
- `off` ei pakkaa koskaan — käytä sitä pakkauksen poissulkemiseen vianmäärityksen aikana tai
  tietokoneessa, jonka on varattava koko prosessoritehonsa muuhun työhön.
- `on` pakkaa aina 12-bittiset formaatit.

Connect-lähtö näyttää päätöksen ja sen perustelun (`packed 12-bit: on (auto:
ennustettu 11,0 -&gt; 14,4 fps, …)`); `/api/camera/array/status` raportoi
`packed_transport` ja `packed_decision`. Jos haluat asettaa koko koneen kattavan oletusarvon
sen sijaan, että välittäisit lippua joka kerta, aseta `CHLOROS_ARRAY_PACKED12=off` (tai
`on`) taustapalvelimen ympäristössä — nimenomainen `--packed on|off`-asetus on kuitenkin etusijalla.
8- ja 16-bittisiä formaatteja ei koskaan pakata.

### Yksi projektitietue, kaikki tilat

Yksi projektikansio sisältää jokaisen laitteen asetukset: `cameras.json` — kunkin kameran (`cameras[].settings`, yksittäisen kameran ja jokaisen ryhmän jäsenen osalta) sekä kunkin ryhmän koko ryhmää koskevat asetukset (`arrays[].capture`: formaatti, ROI, binning, laukaisutaajuus, kaapelibudjetti) — ja `sensors.json`, jokaisen valosensorin asetukset (integraatioaika, kehyksen keskiarvo, yläraja). Sovellus, CLI, SDK ja MCP-palvelin lukevat ja kirjoittavat kaikki samoja tiedostoja samalla tavalla:

- **Projektiin liitetty laite tallennetaan siihen ja tallentaa kaikki muutokset siihen.** ”Projektin alle” tarkoittaa, että projekti on avoinna taustapuolella (sovelluksessa) tai — SDK:n kautta — `ChlorosProject`:n kautta, joka nimeää oman kansionsa. Kamera, matriisi tai valosensori, jota projektissa ei vielä ole, lisätään sen tiedostoihin, kun se muodostaa yhteyden (`lattice cam-connect`, `array-connect`, `daq pool-connect`, sovellus, SDK, `chloros-mcp`) yhdessä sen asetusten kanssa. Muutos mistä tahansa tilasta (`lattice exposure`, `gain`, `configure --set`, `resolution`, matriisinasetukset ja laukaisutaajuus, valosensorin raja-arvo, sovelluksen säätimet) tallennetaan hetken kuluttua kyseisen projektin tiedostoihin. Kameraan asetettu valotus, vahvistus tai AE-kohde tallennetaan pidettynä, joten projektin uudelleen avaaminen säilyttää asetukset ennallaan.
- **Projektin avaaminen asettaa laitteet sen mukaisiksi** kaikissa tiloissa: kamera, matriisi tai valosensori, joka on kytketty ilman omia asetuksia, saa projektin tallennetut asetukset (`lattice cam-connect` / `array-connect`, kun projekti on auki; `chloros-cli project connect`, `open_project().connect_all()`, sovelluksen uudelleen avaaminen). Uuden tunnisteen alla liitetty matriisi päivittää projektin tallentaman matriisin samoilla kameroilla; matriisi, jossa on muita kameroita, lisätään sen viereen.
- **Sovelluksen oma tallennus ei koskaan korvaa laitteen reaaliaikaisia asetuksia** vanhemmalla kopiollaan, ja se säilyttää jäsenkohtaiset ohitukset, jotka CLI- tai SDK-kutsuja on tallentanut.
- **Määrittämäsi pikselimuoto on se, jonka projekti säilyttää.** Array Settings -asetuksissa, komennolla `--pixel-format` tai `connect_array(pixel_format=)` määritetty muoto, 8-bittinen mukaan lukien, tallennetaan ja otetaan uudelleen käyttöön täsmälleen samana. Jos mitään ei ole määritetty, kukin kamera muodostaa yhteyden 12-bittisellä oletusasetuksellaan (`BayerRG12`, `Mono12` monokromisensorilla). Formaatin nimi viittaa sen bittisyvyyteen; formaattiperhe on anturin mukainen, joten värikameraa ei koskaan aseteta monoformaattiin eikä monokameraa Bayer-formaattiin, ja rivi `[PixelFormat <serial>] ... written` ilmoittaa tämän, kun formaatti on määritetty. 8-bittisen tallennuksen vientitiedostot ovat 8-bittisiä, heijastusarvot mukaan lukien: ne skaalataan uudelleen ja niihin lisätään merkintä `PixelScale`.
- Laite, joka on kytketty ilman projektia, ei tallenna mitään.
- **Tallennus noudattaa sen jälkeen luotua projektia; avaaminen jättää sen projektiinsa.**Kun projekti luodaan — SDK:n `create_project()`, `chloros-mcp`:n `create_project` tai sovelluksen**Uusi projekti**— tai kun se kopioidaan (sovelluksen**Kopioi**-toiminto, joka avaa kopion) samalla kun tallennus kirjoittaa edelleen aiemmin avoimena olleeseen projektiin, tallennus jatkuu ja tallennettu aineisto päätyy uuteen projektiin, samaan paikkaan sen sisällä (alle). Olemassa olevan projektin avaaminen — `chloros-cli process`, `project devices --reconnect`, `ChlorosProject.process()`, sovelluksen**Avaa projekti** — jättää tallennuksen edelleen käynnissä siihen projektiin, johon se tallentuu: se päättyy sinne, ja sen tiedostot liitetään kyseisen projektin tiedostoluetteloon (sovellus pysäyttää jokaisen tallennuksen ennen projektin avaamista).
  - `array-record` ja `array-burst`: video tai jokaisen kehyksen sisältävä sarjakuvauskansio siirretään kokonaisuudessaan sinne, kun tallennus loppuu (sen `--duration`, Ctrl+C). Sovelluksen tulostama `Burst dir` on uusi tiedosto, ja `--build` sekä `array-build-video` toimivat sen pohjalta. Jos siirto epäonnistuu levyn syystä, tallennus jää sinne, mihin se on kirjoitettu; lopunloppuloki kertoo syyn, samoin kuin pysäytyksen HTTP-vastaus (`relocation_error`). Pysäytetyn sarjan raakakehykset ja `.daq` lisätään sen projektin tiedostoluetteloon, johon se päätyy; videota (`array-record`, sarjasta koostettu video) ei lisätä projektin tiedostoluetteloon.
  - `array-capture --continuous`: kaikki muutoksen jälkeen tehdyt tallennukset tallennetaan uuteen projektiin; ennen muutosta tallennetut pysyvät entisellä paikallaan.
  - `daq pool-record` ja `project sensor log` (sekä sovelluksen oma valosensoritallennus): `.daq` siirtyy uuden projektin`light_sensor`-kansioon, kun tallennus loppuu, ja liitetään kyseisen projektin tiedostoluetteloon.

  Tallennus avoimen projektin ulkopuolella olevaan kansioon (jossain muualla oleva `-o`) pysyy paikallaan, ja `lattice continuous` sekä `multi-stream` säilyttävät niille annetun kansion.
- **Sovelluksessa uuden projektin jälkeen tehdyt tallennukset menevät siihen.** Projektin luominen tai kopioiminen pitää kaikki tallennussessiot ja tallennukset käynnissä, ja jokainen sen jälkeen tehty tallennus (Tallenna kaikki, Aloita tallennus) tallennetaan uuteen projektiin ja liitetään sen tiedostoluetteloon. Olemassa olevan projektin avaaminen sovelluksessa pysäyttää ensin ne kaikki.
- **Projektia ei siirretä sen tallennuksen aikana.**Avoimen projektin siirtäminen toiseen työkansioon (sovelluksen**Asetukset → Työkansio**; mikään CLI-komento ei siirrä projektia, mutta HTTP-asiakasohjelma, joka lähettää pyyntöjä osoitteeseen `/api/move-project`, voi tehdä sen) hylätään HTTP-virhekoodilla 409 `PROJECT_RECORDING`, kun taulukon tallennus tai sarjatallennus, taulukon jatkuva kaappaus, `lattice continuous`- tai `multi-stream`-tehtävä tai valosensorin tallennus kirjoittaa siihen: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.` Sovellus hylkää pyynnön samalla tavalla, ennen kuin se pyytää kansiota, kun sen oma Capture All-, Start Capturing- tai valosensorin tallennus on käynnissä.

Tämä on erillinen asia siitä, mihin tilaan kamerat jäävät yhteyden katketessa (katso [Kameran muisti](#camera-memory----keep-settings----clear-settings)): projekti asettaa kameransa itse avautuessaan.

### Kameran muisti — `--keep-settings` / `--clear-settings`

Kun kameraryhmä on kytketty, sen kamerat ottavat käyttöön ryhmän asetukset, ja voit sitten muuttaa niitä. Kun Chloros vapauttaa kamerat — `array-disconnect`, väliaikaisen komennon loppu (`multi-capture` ilman matriisia, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`), `lattice continuous`:n pysäytys, sellaisen kameran `lattice capture`:n loppu, jota se ei pidä hallussaan — se palauttaa jokaisen sellaiseksi kuin se oli ennen yhteyden muodostamista: laukaisimen ja synkronointilinjan johdotuksen, PTP- ja kameran asetukset (täydellinen luettelo on kohdassa [Monikamera (väliaikaiset istunnot)](#multi-camera-transient-sessions)).

Jos kameroita käytetään projektin ulkopuolella, kaksi lippua missä tahansa näistä komennoista tallentaa kameran **omaan muistiin** — sen asetukset, joilla se käynnistyy, missä tahansa ohjelmistossa:

- **`--keep-settings`** tallentaa kunkin kameran asetukset sellaisina kuin ne olivat istunnon päättyessä kameran muistiin (UserSet2, josta muodostuu käynnistysasetus). Kamerat jätetään edelleen vapaasti toimimaan (`TriggerMode` `Off`), synkronointilinjan johdotukset säilyvät ennallaan. Kamera, jolla ei ole UserSet2-asetusta, tallentaa asetukset UserSet1:een; tämä tietokone muistaa ne, joten Chloros-järjestelmän automaattinen korjaus — joka nollaa UserSet1:n, joka on poikennut Chlorosin lähtötasosta — jättää sen ennalleen.
- **`--clear-settings`** nollaa kunkin kameran muistin tehdasasetuksiin: Chlorosin lähtötasoon, jonka sen matriisin itsensä korjausprosessi vahvistaa — tehdasasetukset komentojen `TriggerMode`, `Off`, `TriggerSource` ja `Line2` avulla, automaattinen valotus, RGB-kameran kalibroitu valkotasapaino säilytetään — UserSet1:ssä, josta muodostuu käynnistysasetus.
- Näitä kahta ei voi yhdistää. Jokaiselle kameralle annetaan oma rivi: `SN:…: settings saved to the camera's memory (UserSet2); it powers up with them.` tai `SN:…: camera memory reset to the factory settings.`; vastauksen `camera_memory` on sama jokaiselle kameralle.

Sovellus ei tarjoa näitä lippuja.

**Jokainen poistumistapa palauttaa kamerat alkuperäiseen tilaan.**Sama löydetyssä muodossa oleva palautus suoritetaan jokaisella poistumisreitillä, ei vain yllä olevien komentojen yhteydessä: `cam-disconnect` (ja sovelluksen kamera**X**); sovelluksen**X** taulukossa tai minkä tahansa taulukon jäsenen rivillä (taulukko ei voi jatkaa toimintaansa ilman jäsentä, joten koko taulukko vapautetaan); sovelluksen sulkeminen (se antaa taustapalvelimelle enintään 20 sekuntia aikaa palauttaa jokainen kamera, taulukon ja käynnissä olevan `lattice continuous`:n takaisin ennen kuin se pysäyttää ne; tuolloin vielä käynnissä oleva komento — ohimenevä komento, joka avaa kameroitaan tai odottaa niiden PTP-kelloja, kertaluonteinen `lattice capture`, `power`, `white-balance` tai `presets --apply` — pysähtyy seuraavassa turvallisessa kohdassa, palauttaa ensin kameransa enintään 8 sekunnin ajan, ja vastaa HTTP-koodilla 503 `SHUTTING_DOWN`; edelleen käynnissä oleva yhteysjoukko — `lattice array-connect`, sovelluksen yhteys — pysähtyy ennen seuraavaa kameraansa ja sulkee avaamansa kamerat enintään 10 sekunniksi, ja vastaa HTTP-koodilla 503 `SHUTTING_DOWN`; ja sellainen, joka saapuu sammutuksen aikana, ei avaa mitään: HTTP 503 `SHUTTING_DOWN`); ja CLI-komento, joka lopetetaan tai jonka terminaali sulkeutuu (SIGTERM / SIGHUP lopettavat sen tehtävän — ja palauttaa kamerat — ennen kuin se poistuu `143` / `129`, kuten Ctrl+C). `lattice continuous`-, `multi-stream`- tai `align-stream`-prosessi, joka lopetetaan välittömästi (SIGKILL) – mikä ei voi pysäyttää mitään – sen tehtävä keskeytetään ja sen kamerat palautetaan taustapalvelimella 30 sekuntia viimeisen tilannetarkistuksen jälkeen. Kaatunut tai pakotetusti lopetettu taustapalvelin ei voi palauttaa mitään; virran katkaiseminen ja uudelleenkäynnistäminen palauttaa jokaisen kameran omaan muistiinsa, ja seuraava Chloros-yhteys määrittää uudelleen kameran, jonka se löytää valmiustilassa.**Projekti määrittää kameransa itse.** Kun ryhmä muodostaa yhteyden, kun projekti on auki, jokainen jäsen saa asetukset, jotka projekti on sille tallentanut — sille määritetyn valotuksen, vahvistuksen tai AE-tavoitteen (joka pidetään omana, joten automaattinen valotus jättää sen rauhaan) sekä sen valkotasapainotilan (Chlorosin korjaus; kameran oma valkotasapaino pysyy pois päältä) — riippumatta siitä, mitä kameran muistissa on.

### Smart-AE / Smart-Capture

LATTICE-ryhmät suorittavat jatkuvaa automaattista valotusta taustalla heti kytkeytyessään, mutta vastikään kohdistetun kohtauksen valotuksen vakiintuminen kestää hetken. `array-capture --smart` on **valmis kätevä ratkaisu**: se odottaa, että automaattinen valotus vakiintuu jokaisessa ryhmän kamerassa, ja laukaisee sitten kuvauksen. Käytä sitä, kun vaihdat kohdetta kuvaussession aikana.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Vakautumiskäytäntö on oletusarvoisesti konservatiivinen: 5 sekunnin aikakatkaisu, 1,5 sekunnin vakausikkuna, ±5 %:n valotuksen vaihteluväli. Säädä asetuksia SDK:n (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) kautta, jos tarvitset automaatiolta erilaista käyttäytymistä.

#### Automaattisen valotuksen toiminta

- **Valotus ensin, vahvistus viimeiseksi.** Kun vahvistus on Auto-tilassa, vahvistus pysyy 0 dB:ssä, kunnes valotus on saavuttanut sallitun enimmäisarvon.
- **Manuaalinen valotus jättää vahvistuksen Auto-tilaan**, kunnes muutat vahvistusta tai asetat sen manuaaliseksi. Palatessasi Auto-tilaan vahvistuksen raja on sama kuin aiemmin.
- **Pienet kirkkaat kohdat eivät tummenna kuvaa.** Jopa 1 % kuvasta kattava heijastus jätetään huomiotta, ja tunnetun kalibrointikohteen paneeleissa jopa 0,5 % paneelista.
- **DAQ-valotuksen yläraja on voimassa vain silloin, kun kalibrointikohde on näkymässä.** Se estää valkoisen paneelin leikkautumisen; jos paneelia ei näy, kamera valottaa kuvan kohteen mukaan.
- **Vakio valaistus pysyy vakaana, ja valaistuksen muutokset tasaantuvat muutamassa sekunnissa.** Vakio valaistuksessa valotus ja sen raja pysyvät ennallaan, ja suuren valaistusmuutoksen jälkeen kuva tasaantuu muutamassa sekunnissa. Leikkautuminen ei enää rajoita valotusta pysyvästi, ja ROI-alueen muutos palauttaa valotuksen oikeaksi.
- **Kirkkaustavoite pysyy saavutettavissa.** Se ei nouse, kun kamera on valotuksen rajalla eikä sen vahvistusta voi nostaa, joten palaava valo ei aiheuta valohäiriötä kuvassa.
- **Kamera, jonka automaattinen valotus lakkaa reagoimasta, pysyy sen valaistustason mukaisella tasolla.** Kirkkaassa auringonpaisteessa kamera voi laskea lyhimmälle valotusajalleen ja pysyä siellä tummalla kuvalla, vaikka kaikki sen asetukset näyttäisivät normaaleilta. Chloros tarkkailee kuvaa, pitää kameran sillä valaistustasolla, jonka se on säilyttänyt kyseiselle valolle, ja vapauttaa sen, kun valaistus muuttuu. Tumma tai peitetty valosensori ei häiritse tätä.

### Kuvaus- ja vientitasot (oletusarvo `all`)

Tästä versiosta, `lattice capture`, `lattice multi-capture` ja `lattice array-capture` **oletusarvo on `--processing all`** — yksi tallennettu tiedosto kutakin vientityyppiä kohti, joka koskee jokaista kameraa ja vastaa käyttöliittymän ”Capture All” -toimintoa. Tasot ovat:

| Taso | Lähtö | Koskee |
| --- | --- | --- |
| `raw` | Yksikanavainen Bayer (mustavalkokamerat: yksi kaista) suoraan anturista. | Kaikki kamerat. |
| `debayered` | 3-kanavainen väri, tallennettu **RGB**-kanavajärjestyksessä (TIFF-tiedosto, jossa on RGB-fotometrinen tulkinta; PNG- ja JPEG-tiedostot avautuvat myös RGB-muodossa): kalibrointipaketin tumman osan kompensointi (DSNU) on poistettu raakadatasta, minkä jälkeen suoritetaan OpenCV:n reuna-tunnistava demosaic-menetelmä — ei lineaarisuus-LUT:ta, tasokenttää tai värien erottelua (mustavalkokamerat: 1-kanavainen harmaasävy). Python-SDK:n palauttamat, muistissa demosaikoidut kehykset (`grab()`, `frame_stream()`) ovat**BGR**-muodossa, OpenCV:n järjestyksessä; vain tallennettu tiedosto on RGB-muodossa. | Kaikki kamerat. |
| `radiance` | float32 W/m²/sr/nm täyden radiometrisen ketjun kautta. | Vain monispektriset (M3C/M3M) — **ohitetaan RGB-suodattimella varustetut kamerat**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), Pix4D-valmis. | Vain monispektrinen, ja **vain kun DAQ on sidottu + kamera on kalibroitu**; muuten ohitetaan. |
| `display` | Täydellinen GUI-esikatseluketju (CCM + WB + gamma kameran profiilin mukaan). Tallennetaan `preview/`-kansioon. `lattice capture` hyväksyy sille myös nimen `preview`; `array-capture` / `multi-capture` hyväksyvät vain `display`. | Kaikki kamerat. |

Syötä yksi taso, jos haluat tallentaa vain sen yhden (`--processing debayered`). Kun pyydät `all`:ää, tasot, jotka eivät koske tiettyä kameraa, ohitetaan (ja raportoidaan), mutta virhettä ei ilmoiteta — kytkemätön tai kalibroimaton kamera saa silti arvot `raw` / `debayered` / `preview`.

#### DAQ:n lukema tallennetaan

Aina kun DAQ on liitetty kameraan tai kameraryhmään tai lisätty poolin jäseneksi, tallennuksessa käytetty alaspäin suuntautuva lukema kirjoitetaan **`.daq`**-tiedostoon tallennuskansion `daq/`-alikansioon, ja siihen merkitään `.match.json` (kuinka kaukana lukema oli valotuksesta) ja sen `.cameras.json` (kenen lukema se on), ja se raportoidaan `DAQ:`-rivillä. Tämä pätee jokaiseen tasoon, mukaan lukien raakakuvat ja debayeroidut kuvat, joten mikä tahansa kuva voidaan myöhemmin käsitellä uudelleen säteilyvoimakkuudeksi, heijastuskyvyksi tai indeksiksi. Kyseessä on sama tiedosto, jolla on sama nimi ja sivutiedostot, riippumatta siitä, onko tallenne peräisin graafisesta käyttöliittymästä (GUI), komentoriviltä (CLI) vai sovelluskehityspaketista (SDK). Jos DAQ-lukemaa ei ole lainkaan, `.daq`-tiedostoa ei ole, eikä `multi-capture`-tiedostoa, johon ei ole liitetty matriisia (tilapäinen varajärjestelmä) ei tallenna mitään.

| Lippu | Vaikutus |
| --- | --- |
| `--no-daq` | Älä tallenna lukemaa tallennuksessa, jonka tasot eivät käytä sitä (`raw`, `debayered`). `radiance`-, `reflectance`- tai `display`-taso kirjoittaa sen kuitenkin, koska uudelleen tuonti tarvitsee lukeman, jolla tuote on valmistettu. |
| `--daq-tolerance-ms MS` | Kuinka kaukana mittausarvo saa olla altistuksesta, jotta se vielä katsotaan toleranssin sisällä olevaksi (oletusarvo 500). Vain arvio muuttuu (`.match.json`:n `in_tolerance` ja `tolerance_ms`, heijastavuuden `DlsInTolerance`), ei koskaan käytettyä lukemaa. Katso [DAQ-lukema kaukana valotuksesta](#a-daq-reading-far-from-the-exposure). |
| `--force-daq` | Vanhempi tapa pyytää `.daq`-arvoa pelkästään raakamuodossa olevasta kuvasta. Se on nyt oletusasetus, ja `lattice capture` sekä `array-capture` hyväksyvät edelleen tämän lipun (`multi-capture` ei). `--no-daq` ohittaa sen. |

Mikä DAQ: kameraan tai matriisiin sidottu (käyttöliittymän valoanturivalinta, projektin tallennettu sidonta tai `chloros-cli daq pool-bind`), muuten uusin, joka taustapalvelimella on. Raakasarja (`array-burst`) säilyttää oman `.daq`-tiedostonsa jokaiselle erilliselle lukemalle sarjakuvauskansiossa kuten aiemminkin.

**PNG- ja JPEG-tiedostot sisältävät 8 bittiä kanavaa kohti, joten ne käyttävät vain `display`- ja `preview`-tiedostoja sekä indeksituotteita.** `--format png` tai `jpg`, joiden taso ylittää niiden kapasiteetin, hylätään ennen kuin mitään tallennetaan, ja niissä mainitaan taso sekä TIFF-muoto, joka tukee sitä:

- `radiance` (float32) ja `reflectance` (uint16, `32768` = 1,0) — aina, joten myös `--processing all` (oletus) yhdessä `--format png`:n tai `jpg`:n kanssa hylätään: käytä `--format tiff` tai `--levels display`.
- `raw` ja `debayered` kamerasta, jonka datavirta on yli 8 bittiä (12-bittiset M3C- ja Mono12 M3M -kehykset) — CLI:n ei näe kameran bittisyvyyttä, joten taustapalvelu hylkää nämä (HTTP-vastaus 400). Yksittäinenkehyksen `lattice capture --processing debayered` tallentaa 8-bittisen kehyksen PNG- ja JPEG-tiedostoille eikä sitä hylätä, eikä myöskään `multi-capture`:ää, jossa ei ole pysyvää taulukkoa (8-bittinen debayeroitu).

Jokainen tallennusreitti vastaa samalla tavalla (`error_code` `FORMAT_CANNOT_HOLD_LEVEL`): `lattice capture`, `multi-capture`, `array-capture`, `project capture` / `burst` sekä niiden takana olevat HTTP-reitit.

### Miltä tallennuskansio näyttää

Jokainen vientityyppi tallennetaan **omaan alikansioonsa**`-o`-kansion alle, joten monitasoisessa tallennuksessa eri tyypit eivät sekoitu keskenään. Piirroksessa näkyy**`all`-oletustallennus yhdestä kamerasta, jonka on tehnyt `lattice capture`** — useita tasoja kameraa kohti, joten jokaisessa tiedostossa on `_<level>`-päätte:

```
output/
├── raw/           capture_<ts>_SN<serial>-<filter>_raw.tiff
├── debayered/     capture_<ts>_SN<serial>-<filter>_debayered.tiff
├── radiance/      capture_<ts>_SN<serial>-<filter>_radiance.tiff
├── reflectance/   capture_<ts>_SN<serial>-<filter>_reflectance.tiff
├── preview/       capture_<ts>_SN<serial>-<filter>_display.tiff
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_radiance_clampmask.tif, reflectance/…, index/… — the unmix clamp
│                  masks, one subfolder per product folder, only when a band clamped
└── daq/           daq_data_<sensor id>_<ts>.daq: the downwelling reading matched to the capture
                   (+ .daq.cameras.json: whose)
```

Miten tiedostonimet vaihtelevat:

- **Kamerakenttä — sarjanumero, sitten suodatin:** jokaisessa tallennustiedostossa on sen ottaneen kameran nimi muodossa `SN<serial>-<filter>`, jossa `<filter>` on kameramallin suodatinosa: `SN261902142-F450` kameralle `M3M-L87-F450`, `SN213800234-FRGN` mallille `M3C-L87-FRGN`, `SN214701292-FRGB` mallille RGB-kamera. Suodatin yhdistää sarjanumeron muotoon `-`, ei `_`:n kanssa, joten nimessä on samat `_`:llä erotetut kentät kuin aiemmin (etuliite, päivämäärä, kellonaika, millisekunnit, kamera, taso). Kamera, jonka mallia ei voida lukea (pyyhitty DeviceUserID), nimetään yksinkertaisesti `SN<serial>`. Aikaisempien versioiden tallenteiden nimissä ei ole suodatinta; Chloros lukee molempia.
- **Etuliite:** `lattice capture` nimeää tiedostot `capture_…`; `array-capture`, `multi-capture`, kun niihin liittyy taulukko, ja `project capture` / `burst` / `stream`, jotka kuuluvat sarjaan, nimetään `sync_<ts>_SN<serial>-<filter>…`.
- **Tason jälkiliite vain, kun tasoja on useita.**Kamera, joka tallentaa**useita**tasoja yhdessä tallennuksessa (`all` tai `--levels a,b`), lisää `_<level>`. Kamera, joka tallentaa**yhden** tason, säilyttää pelkän nimen ilman loppuliitettä — `--processing reflectance` tallentaa nimellä `reflectance/sync_<ts>_SN<serial>-<filter>.tiff` — jotta yksitasoisia kuvauksia varten kirjoitetut skriptit toimivat edelleen. Taso ilmaistaan edelleen kansion nimessä.
- **Rajoitusmaskit seuraavat tuotteen nimeä:** `<product name>_clampmask.tif`. Yhden tason heijastuskuvaus kirjoittaa `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`; `all`-kuvaus kirjoittaa `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`.
- **Laajennus** seuraa `--format`: oletusarvoinen `tiff` tuottaa `.tiff`, `--format tif` tuottaa `.tif` (rajoitusmaskit ovat aina `.tif`).
- **Yksi poikkeus:** `lattice capture --processing debayered` (tai `--levels debayered`) ottaa nopean yksittäisen-kehyksen polun ja kirjoittaa `capture_<ts>_SN<serial>-<filter>.tiff` suoraan `-o`:ään — ilman alikansiota.
- **Virrat ja sarjat** sisältävät saman kentän: `lattice continuous` kirjoittaa `frame_<seq>_SN<serial>-<filter>.tiff`, `lattice multi-stream`, `stream_<seq>_SN<serial>-<filter>_<frame id>.tiff` ja raakasarjan `<burst>_f<frame>_SN<serial>-<filter>_raw.tiff`.

`<ts>` on tallennuksen aikaleima ja `<serial>` kameran sarjanumero, joten yksi synkronoitu ryhmä jakaa
aikaleiman kaikkien kameroiden kesken. **Huomaa yksi epäsymmetria:** `display`-taso tallennetaan kansioon
nimeltä `preview/`, kun taas monitasoisissa tiedostoissa `_display` esiintyy nimessä — kansio ja pääte eroavat toisistaan
vain kyseisellä tasolla. Tuntemattomat tasot tallennetaan omalla nimellään nimettyyn kansioon, ja jos alikansiota
ei voida luoda, tiedosto kirjoitetaan tulostuksen juurikansioon sen sijaan, että se menetettäisiin.

**Captures-kansion uudelleenkäsittely:**osoita `chloros-cli process`**captures-juurikansioon**
(`output/`). `process` tuo yleensä vain nimeämäsi kansion, mutta jos kyseisessä kansiossa ei ole
kuvia ja siinä on alikansioita, se etenee automaattisesti alikansioihin — joten tason alikansiot ja
`daq/`-alikansio haetaan kaikki kerralla. Jokainen kaappauksen taso tuodaan yhtenä kuvana, ja
muut tasot ovat käytettävissä tiloina sen sijaan, että jokaisesta tasosta tuodaan yksi kuva.

Myös **tason alikansion** nimeäminen suoraan (esim. `output/raw/`) toimii. Tällöin `daq/`
jää pois, joten kopioi tai osoita DAQ-lukema sen viereen, kunjohdat radiometristä
tuotetta tiedostosta `raw/` — muuten aikaleimojen täsmäytys ei löydä vastaavuutta.

**Käsittely alkaa aina tiedostosta `raw`.** Kunkin tallennuksen sisällä raakakuva toimii prosessilinjalle syötteenä;
`debayered`, `radiance`, `reflectance` ja `preview` ovat katseltavia tiloja, mutta niitä ei koskaan syötetä
takaisin käsittelyputkeen. Johdetun tuotteen uudelleenkäsittely johtaisi vignetin, CCM:n ja
säteilyn laskennan uudelleen soveltamiseen, jotka on jo upotettu sen pikseleihin, joten Chloros kieltäytyy siitä
kaksinkertaisen käsittelyn sijaan. Kaksi huomionarvoista seurausta:

- `index/`- ja `composite/`-renderöintejä ei **koskaan** käsiteltyjä. Ne ovat tulosteita, eivät kaappauksia —
  NDVI-LUT-renderöinnillä ei ole merkityksellistä säteilytulkintaa.
- Kaappauskansio, joka on viety **ilman** `raw`-tiedostoa (esim. `array-capture --processing reflectance`), ei ole
  laillista prosessilähdettä. Nämä tallenteet tuodaan ja näytetään normaalisti, mutta `process` ohittaa
  ne ja ilmoittaa asiasta:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Jos joudut todella lähettämään johdetun tuotteen läpi — esimerkiksi
  `demosaic` päällä, tai vanha kansio — `--input-level {raw,debayered,processed}` pakottaa sisäänmenokohdan
  ja ohittaa ohituksen. Tämä lippu on tarkoituksellinen pakotie; `auto` (oletus)
  ei koskaan käsittele tallennetta, jossa ei ole raakadataa.

**Kalibrointitunnisteet: vain `raw`:ssä.** Tallennuksen `raw`-tiedosto sisältää kameran kalibrointipaketin: `CalibrationJSON`-hyötykuorman ja MicaSense-yhteensopivat tunnisteet (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), jotka kuvaavat anturin raaka-arvoja. Jokaisen muun tason tiedosto (`debayered`, `display`, `radiance`, `reflectance`, `index/`-renderöinti) sekä jokainen `lattice continuous` / `multi-stream`-kehyksessä on kameran tunniste (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) sekä kehyksen oman tietueen, `XMP-mapir:FrameJSON` (valotus, vahvistus, lämpötila, pikselimuoto, tallennusajankohta, DAQ-vastaavuus), sillä jokainen `process`-tuote sisältää tämän tunnisteen. `process` etsii tällaisen tiedoston kalibroinnin sarjanumeron perusteella ja `CalBundleSha256` etsii sen Chlorosin kalibrointivälimuistista, lataamalla paketin kerran, jos tietokoneelta puuttuu.

#### Useista kameroista luodut tiedostot

Tiedosto, joka on luotu useamman kuin yhden kameran kaistoista — monikaistapino (`align-apply`, `align-stream`, `index --live --save-multiband`), useista kameroista koostuva reaaliaikainen indeksi (`index.tif`, `index_LUT.png`), matriisikuvauksen `composite/`-kuva — nimeää jokaisen siihen sisältyvän kameran:

- **Kamerakohtaiset luettelot, kaistajärjestyksessä** (yhdistelmä: taustakerros, sitten etualakerros; indeksikerros, kamerat, joista sen kaistat ovat peräisin): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (kalibrointipaketti, jonka kautta kunkin kameran kaistat on käsitelty, muuten — `align-apply` / `align-stream`-pino, jonka kaistat eivät läpäise mitään — kameran paketti, jonka Chloros on tallentanut välimuistiin, se, jonka mukaan sen tallennustiedostot on nimetty), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — rinnakkaislistoja, joista kunkin kohde *n* kuvaa samaa kameraa; `-`, jossa kameralla ei ole arvoa (ei kalibrointipakettia, tuntematon objektiivi). Jokainen kamera on **yksi** kohde: jos sen paketissa on useita arvoja samalle kentälle — esimerkiksi paketin halogeenilamppusertifikaatit — ne yhdistetään merkillä `;` (`F-1479-1;F-1480-1;F-1481-1`), ei koskaan pilkulla, joten luettelossa on yksi tuote per kamera riippumatta siitä, miten se tulostetaan.
- **EXIF `SerialNumber`** sisältää samat sarjanumerot, pilkulla yhdistettyinä (`214000535,214000551`).
- Se ei sisällä **yhtään** yksittäisen kameran tunnusta (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): ne nimeävät yhden kameran ja sen paketin, ja `process` määrittäisi tiedoston kalibroinnin niiden perusteella. Useiden kameroiden kaistoista laskettu indeksi (`index.tif`, `index_LUT.png`) sisältää EXIF-tiedot `Model` `LATTICE` (laitteisto), ei yhden kameran `LATT-…`-mallia.**Yhden** kameran kaistojen tiedosto — kaistakohtainen TIFF-tiedosto (`--per-band`), `align-apply --per-camera`-kuva, yhden kameran pino tai reaaliaikainen hakemisto — sisältää kyseisen kameran omat tunnistetiedot, kuten kaikki muutkin tallennustiedostot. `align-apply`:n pino-, kaistakohtaiset ja kamerakohtaiset tiedostot sekä jokainen `align-stream`-pino ja sen kaistakohtaiset tiedostot sisältävät ryhmänsä tallennusajan (vertailukameran valotusaika, UTC), ja monokameran `--per-camera`-kehys on yksi taso, kuten `--bit-depth`-pino.**Kameran suunta (peilaus / kierto).** Kameran suunta-asetus (käyttöliittymän Mirror H /
Mirror V / kierto) määrittää, mitä kuvaus tallentaa:

- **`raw` ja `debayered` tallennetaan täsmälleen sellaisina kuin anturi ne luki**, asetuksista riippumatta.
  Käännetty tai peilattu Bayer-mosaiikki muodostaisi erilaisen värisuodatinkuvion, ja kalibrointikartat
  (dark offset, flat field) ovat anturin omalla ruudukolla.
- **`preview`/`display`, `radiance`, `reflectance` ja `index/`-tiedostot on suunnattu** — käännetty
  kerran kaikkien kalibrointilaskelmien jälkeen, joten ne avautuvat samalla tavalla kuin kamera oli suunnattu. Niiden
  `clamp_masks/…_clampmask.tif`-tiedostot sijaitsevat samalla ruudukolla.
- **Jokainen kuvatiedosto tallentaa asetukset** XMP-muodossa: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, myötäpäivään,
  käännösten jälkeen) sekä `ChlorosOrientationApplied` — `false` raakatiedostoissa ja debayeroiduissa (alkuperäiset pikselit),
  `true` suunnatuissa tuotteissa. `ChlorosProcessing` luettelee kyseisen tiedoston
  pikseleihin sovelletut operaatiot (`none`, kun kyseessä on natiivi). Tähän sisältyy jokainen `lattice continuous`-virran kehys (
  kameran oma asetus, soveltamatta) ja jokainen `lattice multi-stream`-virran kehys (identiteetti: sen
  jäsenkehykset ovat natiiveja, kuten taulukkomuotoisessa tallennuksessa). `lattice align-stream` / `align-apply`
  -pinot tallentavat pääkameran asetukset, joita ei ole sovellettu: niiden kaistat on vääristetty pääkameran
  kuvakehykseen, mutta niitä ei ole käännetty.
- **`process` soveltaa tallennettua suuntaa** käsitellessään raakadataa: ensin tumman offsetin, tasoituksen
  , demosaicin ja sekoituksen purkamisen anturikoordinaateissa, seuraavaksi matriisin kohdistuksen vääristymän ja
  suunta viimeiseksi — joten esikatselu-, säteilyvoimakkuus-, heijastavuus- ja indeksituotteet tulevat täsmälleen samankaltaisiksi kuin
  reaaliaikaisessa kuvauksessa, ja tallennetaan `ChlorosOrientationApplied=true`. Kalibroimattomat varatuotteet
  tuotteet (`Sensor_Response_Images`, `Vignette_Corrected_Images`) on suunnattu samalla tavalla,
  tasokentän jälkeen. Debayeroitu tuote pysyy alkuperäisessä muodossaan.
- **Chloros-kuvankatseluohjelma näyttää raakakuvan tai debayeroidun tiedoston käännettynä sen tallennusjärjestyksen mukaisesti** (vain näytöllä —
  tiedostoa ei muuteta), joten se näyttää tuotteiltaan; pikkukuva, täysikokoinen näkymä,
  kohdistimen lukema ja Index/LUT-hiekkalaatikko käyttävät kaikki samaa käännöstä. Tiedosto ilman tallennustietoa ja
  tuote (joka on jo suunnattu) näytetään tallennetussa muodossaan. Sandbox-vienti tallentaa sen
  pikselien sisältämän suunnan (`ChlorosOrientationApplied=true`; tunnus kerrokselle, jossa ei ole käännöstä).
- **Tallenteet, joissa ei ole tietuetta, käsitellään alkuperäisessä muodossaan** (käyttäytyminen ei muutu). Ennen versiota 1.3.0
  yhden kameran tallenne, jossa käännös tai kierto oli asetettu, käänsi itse raaka- ja debayeroidut tiedostonsa;
  Chloros ei osaa tunnistaa tällaista jokäännettyä raakatiedostoa natiivitiedostosta eikä arvaile, joten se
  käsittelee sen tallennetussa muodossa. Kalibrointikartat ovat anturin omalla ruudukolla, joten tällaisessa
  kuvauksessa käytä 1.3.0:lla otettua kuvaa, kun sen kalibroinnilla on merkitystä.
- **Arrays ei käännä oletusarvoisesti yhtään jäsen-tiedostoa** (kohdistusvääristymä ja `Alignment*` XMP ovat
  kameroiden omissa, kääntämättömissä kehyksissä) ja tallentaa identiteetin. `array-capture --orient` kääntää jokaisen
  jäsenkuvan tulokset kohdistusmuunnoksen jälkeen: **pääkuvan** suuntaan, kun ne
  muunnettiin sen kehykseen (jotta jäsenkuvat pysyvät rekisteröityinä toisiinsa nähden), muuten omien kameroidensa suuntaan. Jokainen
  tiedosto tallentaa sen kehyksen suuntauksen, jossa sen omat pikselit sijaitsevat — raakakuvana, oman kameran suuntauksena —
  ja raakakuva, joka sisältää kohdistusmuunnoksen, tallentaa myös `ChlorosAlignedFlipHorizontal` /
  `ChlorosAlignedFlipVertical` / `ChlorosAlignedRotation`: mestarikuvan koordinaatit, kun tuote on taivutettu
  mestarin kehykseen. `process` kääntää jokaisen tuotteen sen pikseliin kuuluvan arvon verran:
  kohdistetun arvon, kun se vääristää raakatiedoston (`--array-alignment`, oletusarvo – reaaliaikaisesti kohdistetun
  kaappauksen tuotteet), kameran oma, kun sitä ei tehdä (`--no-array-alignment` – reaaliaikaisen
  kohdistamattoman kuvauksen tuotteet). Sama vaihtoehto on olemassa myös `array-burst`:lle, `project capture`:lle / `burst`:lle /
  `stream`:lle, resepteille ja SDK:lle.

### Ohitetut kuvaukset sekasuodatinryhmissä

Kun yhdistät RGB- ja monispektrikameroita yhteen ryhmään, `array-capture --processing radiance` (tai `reflectance`) tallentaa monispektrikuvat ja **ohittaa** RGB-kameroiden kuvat — Bayer-kohtainen säteilyvoimakkuus ei ole merkityksellinen laajakaistaiselle anturille. CLI tulostaa jokaisen tallennetun tiedoston (sen vientitason kanssa), jokaisen kirjoitetun `.daq`-merkinnän ja jokaisen ohituksen erikseen, joten tiedostojen lukumäärä ei tule yllätyksenä:

```
  Saved: …/output/reflectance/sync_…_SN213800234-FRGN.tiff [reflectance] (SN:213800234, fid:1)
  Saved: …/output/reflectance/sync_…_SN214000533-FOCN.tiff [reflectance] (SN:214000533, fid:1)
  Saved: …/output/reflectance/sync_…_SN214701288-FNGB.tiff [reflectance] (SN:214701288, fid:1)
  DAQ:   …/output/daq/daq_data_daq-e-54b5e0_….daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Ohitussyyn tunnukset noudattavat mallia `<level>-not-applicable-to-rgb-cam`. Kamera, jonka kalibrointipakettia ei ole ladattu, ohittaa `radiance` / `reflectance`, kun `<level>-needs-calibration (connect the camera so its cal pack loads)`, ja jos käyttökelpoista DAQ-heijastuslukemaa ei ole, ohitetaan koodit `reflectance-skipped-no-dls`. Heijastusarvo voi myös ohittaa koodit `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, ja mallin `dls-uncalibrated-band-<nm>` kanssa, kun kaista sijaitsee pääosin DAQ-valosensorin radiometrisesti kalibroidun alueen (~374–974 nm) ulkopuolella — toimitettavista tuotekoodeista ainoastaan F988, jonka tuettu käyttöpolku on heijastavuuspaneelin työnkulku.

Käytä `--processing debayered`:ää (tai `display`:ää), jos haluat sisällyttää kaikki kamerat suodattimen tyypistä riippumatta, tai oletusarvoista `all`:ää, jos haluat saada kaikki soveltuvat tasot kamerakohtaisesti yhdellä kertaa.

### DAQ-lukema kaukana valotuksesta

Jokainen tallennettu `.daq`-arvo on kyseiseen valotukseen sovitettu DAQ-lukema, ja sen vieressä oleva `.match.json`-arvo tallentaa, kuinka kaukana nämä kaksi olivat toisistaan (`gap_ms`) ja onko ero toleranssin sisällä (`in_tolerance`, 500 ms, ellei kuvaussarjassa ole määritetty toisin `--daq-tolerance-ms`). Myös toleranssin ulkopuolella oleva lukema otetaan käyttöön, joten yhtään kuvaa ei menetetä, ja se merkitään: heijastustiedoston XMP-tiedostossa on merkinnät `DlsInTolerance=False` ja `DlsMatchGapS`. `lattice capture`, `multi-capture` ja `array-capture` tulostavat sen myös sen `.daq`:n alle, jota se koskee, ja DAQ:nvirhevirrasta, kun asiaa selitetään:

**Miten lukema sovitetaan valotukseen.** Tallennus odottaa DAQ:n ensimmäistä lukemaa valotuksen jälkeen, jotta se voi yhdistää sen molemmin puolin olevat lukemat (`method` `interpolated`). DAQ-E lähettää lukeman päivänvalossa 0,4 sekunnin välein ja sisätiloissa 1–2 sekunnin välein, joten odotus lisää komentoon noin puolitoista tällaista aikaväliä, ei valotusaikaan. Lähettämisen lopettanutta DAQ:ta ei odoteta. DAQ-E-lukema on edellisen lukeman jälkeisen aikavälin valon keskiarvo, joten se sijoitetaan kyseisen aikavälin keskelle: `gap_ms` mitataan valotuksesta kyseiseen keskikohtaan, ja `bracket_stamps_ns` antaa lukemien omat aikaleimat. `array-capture --continuous` odottaa hidastamatta kuvausprosessia: sen raakatiedostot kirjoitetaan välittömästi ja niiden `.daq`-arvot, kun lukema saapuu. `--interval`:n avulla jokainen kierros odottaa kuin yksittäinen tallennus.

```
  DAQ:   …/output/daq/daq_data_daq-e-54b774_….daq
         DAQ reading 114.9 s from the exposure (tolerance 500 ms): reflectance uses it, labelled DlsInTolerance=False
         DAQ stream fault: no frame accepted from daq-e-54b774.local in 25 s, after 1932 good frame(s). …
```

Jos kyseisestä lukemasta ei kirjoitettu heijastustiedostoa, ensimmäinen rivi päättyy sen sijaan merkkiin `saved anyway; its .match.json says in_tolerance: false`.

Tallennus päättyy silti merkillä `0`: kaikki tiedostot on kirjoitettu ja niissä ilmoitetaan niiden sisältö. DAQ, jonka datavirta on pysähtynyt, jatkaa viimeisen lukemansa toimittamista, joten tarkista se merkillä `chloros-cli daq pool-list` ennen kuin luotat kyseisen tallennuksen heijastustietoihin. Taustapalvelin yhdistää uudelleen poolatun DAQ-E:n, joka on lakannut toimittamasta (katso [DAQ-E, joka lakkaa toimittamasta](#a-daq-e-that-stops-delivering)). Toleranssin ulkopuolella oleva lukema tulostaa vain `DAQ:`-rivinsä, ellei se ole säilytetty lukema (katso alla).

**Lukema, joka on säilytetty DAQ:n katkeamisen jälkeen.** Kun mikään DAQ ei voi toimittaa lukemaa tietylle altistukselle (sidottu DAQ ei ole käytettävissä tai yhdelläkään poolatulla DAQ:lla ei ole lukemaa, esim. `daq pool-disconnect`:n jälkeen), tallennus käyttää uudelleen viimeistä lukemaa, joka Chlorosilla oli, ja sen `.match.json` kuvaa kyseistä lukemaa suhteessa tähän altistukseen: `method` `retained`, `gap_ms` tästä altistuksesta näytteisiin, joista lukema on otettu, `in_tolerance` kyseisestä välistä sekä `retained_from_epoch_ns`, eli altistuksesta, johon lukema alun perin yhdistettiin. Siksi kaappaus, joka on otettu kauan sen jälkeen, kun DAQ-laite on poistettu käytöstä, tulostaa yllä olevat rivit ja niiden alle niiden alle vielä yksi, joka tulostuu silloinkin, kun lukema on vielä toleranssin rajoissa:

```
         This is the DAQ's last reading: the DAQ is no longer reporting, so each capture reuses it until a new one arrives
```

Sen `.daq` nimeää edelleen lukeman ottaneen DAQ:n sekä kalibroinnin ja suojuksen, joiden alla lukema otettiin, joten uudelleen tuonnissa tiedetään, mikä DAQ, kalibrointi ja suojus sen tuottivat.

### Tiedostot, joita ei voida tallentaa

Kamera, jonka tiedostoa ei voida tallentaa – täysi kiintolevy, käyttöoikeusvirhe – ei estä muiden kameroiden tiedostojen tallentamista. Jokainen tallennuskomento (`lattice capture`, `multi-capture`, `array-capture`; `project capture`, `burst`, `stream`; sekä reseptin komennot `capture`, `burst` ja `stream`) tulostavat jokaisen epäonnistuneen tiedoston stderr-tiedostoon, ja **lopettaa sitten `1`**, vaikka loput tallentuivatkin:

```
  Failed: SN:214701292 [reflectance] OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

Kaikki tallennuksen sisältämät tiedostot on käsitelty, mukaan lukien yksittäinendebayeroitu yksittäiskuva `lattice capture --levels debayered` mukaan lukien. `.daq`-lukema, jonka `.match.json`-alkuperätiedostoa ei voitu kirjoittaa, on luettelossa nimellä `[match]` (itse `.daq` tallennettiin).

Täydellä levyllä toistuva tallennus (`array-capture --continuous` / `--interval`, `project burst` / `stream`, resepti `burst` / `stream`) **pysähtyy kyseisessä kierroksessa** sen sijaan, että samaan asemaan ei kirjoitettaisi mitään ajon loppuun asti. Kaikki muut virheet raportoidaan jokaisella kierroksella, jolla ne tapahtuvat, ja ajo jatkuu loppuun asti, minkä jälkeen ohjelma lopetetaan `1`. `array-capture --continuous` (taustaprosessin jatkuva tallennus) ei luettele tiedostoja, epäonnistuneet mukaan lukien: se laskee ne edistymis- ja yhteenvetoriveillään (`N failed`, `N file(s) failed to write.`) ja lopettaa ohjelman `1` — katso [Jatkuva tila](#continuous-mode---continuous).

Tallentimet — `array-burst` ja `array-record` — ilmoittavat omista virheistään. `array-burst` pysäyttää itsensä, kun asema täyttyy tai juuri ennen sitä. Molemmat jatkavat toimintaansa muiden epäonnistuneiden kirjoitusten jälkeen. Kun jompikumpi ilmoittaa virheestä, se tulostetaan laskurien jälkeen, ja komento **poistuu `1`**:

```
  Recorder error: disk-full: OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

`Disk low:` `Disk full:`:n sijaan tarkoittaa, että sarja keskeytyi ennen kuin asemaa. Ennen pysähtymistä tallennetut kehykset säilytetään. `array-burst --build` ohittaa rakennuksen, kun tila loppui, koska videot menisivät samalle asemalle. Vapauta ensin tilaa ja suorita sitten `array-build-video --burst-dir …`. Muiden virheiden jälkeen rakennus jatkuu silti.

Myös `array-record`, joka ei kirjoittanut yhtään kehystä, on epäonnistunut tallennus (`Recorder error: no-frames: the combined view published no frame to record …`, poistu `1`) — katso [`array-record`](#array-record--combined-index-videogif-monitoring-grade).

---

## Tallennustilat, tallentimet ja offline-uudelleenkäsittely

Nämä kaikki toimivat **pysyvällä taulukolla** (suorita ensin `array-connect`). Ne vastaavat graafisen käyttöliittymän tallennuspaneelia.

### `array-capture`-tilat

`array-capture` on yksi komento, jossa on neljä suljintilaa sekä joukko vientivalintoja:

| Tila | Lippu | Toiminta |
| --- | --- | --- |
| **Yksittäinen** *(oletus)* | (ei mitään) | Yksi synkronoitu kaappausryhmä, jonka jälkeen ohjelma sulkeutuu. |
| **Jatkuva** | `--continuous` | Peräkkäisiä kaappauksia, kunnes `Ctrl+C`, `--count N` kaappaa, `--duration S` tai kun levy on täynnä. Suorittaa taustapalvelimen oman jatkuvan tallennuksen (käyttöliittymän Jatkuva-tila) — katso [Jatkuva-tila](#continuous-mode---continuous) alla. |
| **Väli** | `--interval S` | Yksi kierros joka `S` sekuntia (mitattuna kunkin kierroksen alusta), samat rajat. |
| **Nopein** | `--fastest` | Vain raakadata-vain + määritetty DAQ-lukema + yhdistetty indeksi; ohittaa säteilyvoimakkuuden/heijastavuuden/näytön laskennan, jotta kehys latautuu nopeasti. Edellyttää `--processing raw --force-daq`:ää. Käsittele tallennettu `.daq` myöhemmin uudelleen kalibroiduiksi tuotteiksi. |

Vientivaihtoehdot (yhdistettävissä mihin tahansa tilaan; kaikilla on yhteinen GUI/SDK-päätelaite):

| Lippu | Vaikutus |
| --- | --- |
| `--array-id ID` | Taulukko, josta tiedot kerätään (jätä pois, jos vain yksi on kytkettynä). |
| `-o, --output DIR` | Tulostuskansio (oletusarvo `output`, siinä kansiossa, josta komento suoritetaan). |
| `--format {tiff,tif,png,jpg,jpeg}` | Tiedostomuoto (oletus `tiff`). PNG- ja JPEG-tiedostomuodot tukevat vain `display`-tasoa ja indeksin renderointia; katso [Tallennuksen vientitasot](#capture-export-levels-the-all-default). |
| `--processing LEVEL` | Yksi vientitaso (`raw`, `debayered`, `radiance`, `reflectance`, `display`) tai `all` (oletus). |
| `--levels L1,L2,…` | Vienti-tyyppien eksplisiittinen osajoukko (esim. `raw,radiance,reflectance`); **korvaa `--processing`**. |
| `--aligned` / `--no-aligned` | **Oletus: kohdistettu, kuten Chloros-sovelluksen tallennus samasta taulukosta on**(sen taulukokohtainen**Kohdistettu**-tallennusasetus, päällä ellei sitä kytke pois päältä). Jokaisen jäsenen ei-raaka-vienti (`debayered`, `display`, `radiance`, `reflectance`, indeksin renderointi) vääristetään pääkameran kehykseen ja rajataan jäsenten yhteiseen päällekkäisyyteen, joten tiedostot ovat pikselitasoitetut ja niiden koko, rajaus ja `Alignment*`-tunnisteet ovat samat kuin sovelluksen tiedostoissa — pienemmät kuin anturin (tai ROI:n) jäsenien siirtymien verran; ROI:n alla muunnos perustetaan uudelleen ikkunaan. `XMP-mapir:AlignmentJSON` määrittää `"applied": true`:n ja tuottaa `crop_roi`:n. Raw-tiedosto pysyy muuntamattomana alkuperäisessä koossaan, mutta sisältää muunnoksen metatiedoissa (`"applied": false`). `--no-aligned` tallentaa jokaisen tiedoston alkuperäisessä koossaan, muuntamattomana. `--aligned` on oletusasetus. Riippumatta siitä, sovelletaanko vääristymää vai ei, jokainen jäsen-tiedosto tallentaa kameransa `Alignment*`-muunnoksen, kun taulukolla on kohdistusprofiili — lukuun ottamatta kameraa, jota profiili ei kohdista (jätetään pois tai pakotetaan identiteettikuvaksi epäonnistuneen ratkaisun vuoksi), sekä vertailukameraa, kun mitään muuta kameraa ei ole kohdistettu: näissä ei ole `Alignment*`-tunnisteita. Jos matriisilla ei ole profiilia, tallenne tallennetaan kohdistamattomana, ja CLI tulostaa syyn `Alignment:`-riville, kuten se tekee kaikissa muissakin tapauksissa, joissa vääristymän korjausta ei voitu soveltaa (eri resoluutioilla olevat jäsenet, matriisi, jota ei ole synkronoitu kaapelilla, kehys, johon profiilia ei voida kartoittaa). Matriisin automaattinen kohdistus luo profiilin, kun valotus on vakiintunut; `align-calibrate` luo profiilin pyynnöstä. |
| `--orient` | Käännä kunkin jäsenen **tuotteet**— `display`, `radiance`, `reflectance`, `index/`-renderöinti ja `composite/` — kameran suuntaan kohdistuksen vääristymän jälkeen: tuote, joka on vääristetty pääkuvan kehykseen (oletus)**pääkuvan**suuntaan, jolloin osat pysyvät kohdistettuina toisiinsa; kohdistamaton (`--no-aligned` tai tallenne, johon vääristymää ei voitu soveltaa) oman kameransa suuntaan.**Oletus: pois päältä — jokainen osatiedosto pysyy alkuperäisessä muodossaan**, kuten taulukkomuotoisissa tallenteissa aina. `raw` ja `debayered` pysyvät alkuperäisessä kehyksessään kummassakin tapauksessa. Jokainen tiedosto tallentaa tuotteidensa suuntauksen (katso [Kameran suuntaus](#miltä-kuvauskansio-näyttää)); `Alignment*`-XMP-tiedosto säilyttää alkuperäisen kehyksen muunnoksen, joka on sovellettu ennen suuntaa, joten `process` (vääristys, sitten suunta) toistaa reaaliaikaisen tuotoksen. |
| `--index` / `--no-index` | Tallenna / ohita kamerakohtainen kasvillisuusindeksin peittokuva, jos sellainen on määritetty. Oletus: renderöi se. |
| `--no-daq` | Ohita `.daq`-sidecar-tiedosto, kun mikään valittu taso ei käytä DAQ:ta (vain raakadataa sisältävä kaappaus). Oletusarvoisesti lukema tallennetaan, jotta kehykset voidaan käsitellä uudelleen heijastavuus-/indeksitiedoiksi offline-tilassa. `--force-daq` hyväksytään edelleen (oletus). |
| `--daq-tolerance-ms MS` | Näiden kaappausten DAQ-vastaavuustoleranssi (oletus 500 ms). |
| `--smart` | Odota, että AE vakiintuu kaikissa kameroissa ennen laukaisua (katso [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | TIFF-pikselipakaus. `none` (oletusarvo versiosta 1.3.0 lähtien; versio 1.2.0 kirjoitti `deflate`) = pakkaamaton — anturin pikselit täsmälleen sellaisina kuin ne on luettu, ~6,3 Mt täysresoluutioista kuvaa kohti, ja nopein jatkuva tallennusnopeus; `deflate` = häviötön zlib L1 + vaakasuora ennustaja, ~4,1 Mt kuvaa kohti, hitaampi. Molemmat ovat häviöttömiä ja luetaan identtisesti tuonnin yhteydessä. |

> **Yksikertaisesti kirjoitettu TIFF + jatkuvan tallennusnopeuden malli.**Jokainen tiedosto kirjoitetaan**kerran**: pikselit, XMP-paketti, IFD0:n valmistaja/malli/kameran sarjanumero sekä EXIF-tunnisteet `ExposureTime` / `SerialNumber` kirjoitetaan yhdellä kertaa, eikä niitä kirjoiteta uudelleen myöhemmin. `--continuous` tallentaa peräkkäisiä synkronoituja ryhmiä — jokainen tallennus ottaa edellisen ryhmän jälkeen tulevan ryhmän, mukaan lukien sen, joka julkaistiin vielä tallennuksen ollessa kesken — joten se toimii matriisin nopeudella. DEFLATE pitää Pythonin GIL:ää, joten pakatut kirjoitukset**eivät**suoriteta rinnakkain kamerakohtaisissa kirjoitussäikeissä; jatkuva täysresoluutioinen tallennus anturin nopeudella vaatii pakkaamattomia kirjoituksia (oletus)**ja** NVMe-luokan levyä (~600 MB/s jatkuvasti 9-kamerajärjestelmälle ~10 fps:llä). Sama säätö on käytettävissä nimellä `compression` kohdassa `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (aligned is the default; drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --no-index -o out/

# Each member at its native size, unwarped
chloros-cli lattice array-capture --processing reflectance --no-aligned -o native/

# Continuous raw capture for 60 s, as fast as the array publishes
chloros-cli lattice array-capture --continuous --duration 60 --fastest -o flightline/
```

#### Jatkuva tila (`--continuous`)

`array-capture --continuous` (ilman `--interval`:ää tai `--smart`:ää) siirtää suorituksen **taustapalvelimen omalle jatkuvalle tallennukselle** — istunnolle, jossa käyttöliittymän jatkuva tila toimii. Taustapalvelin tallentaa jokaisen synkronoidun ryhmän itse, jolloin kaksi tallennusta on käynnissä samanaikaisesti eikä CLI:ltä tule pyyntöä ryhmien välillä, joten se pysyy mukana nopeassa järjestelmässä, jossa yksi pyyntö per tallennus jää jälkeen (mitattu 73–77 tallennusta sekunnissa 9 kameran järjestelmässä 512 pikselin ROI:lla). Jokainen yllä oleva vientikytkin koskee jokaista tallennusta.

- **Rajat:** `--count N` pysähtyy N tallennuksen jälkeen (yksi tallennus = yksi synkronoitu ryhmä: jokaisen jäsenen tiedostot); `--duration S` S sekunnin kuluttua. Ilman kumpaakaan se toimii, kunnes `Ctrl+C`, kiintolevy täyttyy, virhe ilmenee tai kamerajärjestelmä sammuu. `Ctrl+C` lopettaa taustapalvelimen istunnon, ja tulostaa sitten yhteenvedon.
- **Edistys** 2 sekunnin välein, ja `, N failed` ja `, N groups skipped` liitetään mukaan, kun arvo on eri kuin nolla (ohitettu ryhmä on sellainen, jonka ryhmä on julkaissut, mutta josta ei ole tehty tallennusta):

  ```
     12.0s  880 captures (73.3/s), 7920 files
  ```

- **Ei tiedostokohtaista luetteloa.** Suoritus päättyy yhteen yhteenvetoriviin — kaappaukset, nopeus, tiedostot ja tulostuskansio — jota seuraa `N file(s) failed to write.` / `N group(s) skipped.`, kun arvo on eri kuin nolla:

  ```
  Done — 880 captures (73.3/s), 7920 files in /home/user/flightline.
  ```

- **Pysähtyy**, kun levy on täynnä (`Continuous capture stopped: the drive is full.`) tai kun tapahtuu virhe tai levyjoukko katoaa (`Continuous capture stopped: <reason>.`) (molemmat stderr:ssä).
- **Lopetustila:** `0`, kun tallennukset on tallennettu ja kaikki tiedostot on kirjoitettu, mukaan lukien `Ctrl+C`; `1`, jos jonkin tiedoston kirjoittaminen epäonnistui, levyn täyttyessä, virheen sattuessa, taulukon poistumisen yhteydessä tai jos tallennusta ei ole lainkaan.
- **Microsoft Defender:** kun Defender käyttää yhtä tai useampaa CPU-ydintä tallennuksen kirjoittamien tiedostojen skannaamiseen, suoritus tulostaa kerran stderr:iin: `Note: Microsoft Defender is using N.N CPU cores scanning the capture files, CPU the capture needs. Re-run the Chloros installer with "Exclude the Chloros backend from Microsoft Defender scanning" checked.` Katso asennusohjelman [Järjestelmäasetukset](#windows-exe).
- Yksi jatkuva tallennus kerrallaan kutakin taulukkoa kohti; toinen hylätään virheellä `a continuous capture is already running for this array`. HTTP:n kautta istunnon koodi on `POST /api/camera/array/continuous/start` (`/api/camera/array/capture`-runko sekä `max_captures` / `duration_s`), `GET /api/camera/array/continuous/status?array_id=ID` ja `POST /api/camera/array/continuous/stop`.

**Interval**(`--interval S`, joko `--continuous`:n kanssa tai ilman) ja**`--continuous --smart`** säilyttävät pyyntökohtaisen silmukan: yksi tallennuspyyntö per kierros — jokaisella aikataulun mukaisella hetkellä Interval-tilassa, kun AE on vakiintunut `--smart`:n kohdalla — jokaisen kierroksen tiedostot luetellaan kuten Single-tilassa, lopussa on `Done — N capture pass(es).`-rivi ja `--count N` laskee kierrokset.

### `array-record` — yhdistetty indeksi video/GIF (valvontataso)

Tallentaa matriisin **reaaliaikaisen yhdistelmäindeksinäkymän** tiedostoon `.avi` (ja valinnaisesti tiedostoon `.gif`). Kun käyttöliittymän yhdistelmänäkymä on auki, se tallentaa näkymän sellaisena kuin se näkyy. Jos mitään näkymää ei ole auki, taustaprosessi renderöi näkymän itse: indeksin, jonka välität `--expr`:n avulla, tai oletuksena matriisin NDVI:n, kun matriisi on kohdistettu. OletusNDVI käyttää punaista ja NIR-kaistoja, jotka ovat lähimpänä 660 nm ja 850 nm, kuten reaaliaikaisessa näkymässä. Tallennuksen aikana avattu GUI-näkymä tallennetaan niin kauan kuin se pysyy auki. Se tarkistaa edistymisen 2 sekunnin välein ja pysähtyy `--duration`:n, `Ctrl+C`, tai kun tallennin lopettaa toimintansa itsestään.

Tallennus, joka ei kirjoita yhtään kehystä, epäonnistuu: tallennin ilmoittaa virhekoodin `no-frames: …` (matriisi saattaa vielä olla kohdistumassa tai valotusta vakiinnuttamassa) ja komento **lopetetaan `1`**.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF (no GUI needed)
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# Record a chosen index instead of the default NDVI
chloros-cli lattice array-record --duration 30 \
  --expr "(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)" -o monitoring/
```

| Lippu | Oletus | Kuvaus |
| --- | --- | --- |
| `--array-id ID` | vain matriisi | Kohdematriisi (jätä pois, jos vain yksi on kytkettynä). |
| `-o, --output DIR` | `output` | Tulostuskansio. Suhteellinen polku on sen kansion alla, josta komennon suoritat. |
| `--fps F` | `10` | Tallennuksen kuvataajuus. |
| `--duration S` | kunnes Ctrl+C | Pysäytä automaattisesti `S` sekunnin kuluttua. |
| `--gif` | pois | Kirjoita myös animoitu GIF-tiedosto. |
| `--gif-only` | pois | Tallenna vain GIF (ei `.avi`). |
| `--expr EXPR` | taulukon NDVI | Indeksilauseke, joka tallennetaan, kun GUI-näkymää ei ole auki, taulukon kaistanimissä (esim. `'(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)'`; katso [kaistanimet](#index--vegetation-maths)). Lyhyt värinimi (`NIR`, `Red`) toimii vain, jos taulukon yhdellä kaistalla on sellainen. Sen sijaan avoin GUI-näkymä tallennetaan sellaisenaan. |

### `array-burst` — raaka-Bayer-sarjakuvaus suurella kuvataajuudella (analyysilaatuinen)

Lukee suoraan kuvausloopin synkronoidun ryhmän puskurin — **ei kalibrointiketjua, ei exiftool-työkalua, ei live-näkymää tarvita** — joten se toimii kameran täydellä kuvausnopeudella. Kirjoittaa raakakuvat + kuvakohtaisen manifestin + yhden `.daq`-tiedoston kutakin erillistä DLS-lukemaa kohti `<output>/bursts/<base>/`-kansiossa. Käsittele uudelleen offline-tilassa (seuraava komento) tai anna `--build`, jotta se tehdään välittömästi pysäytyksen yhteydessä.

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
| `--max-frames N` | rajoittamaton | Automaattinenpysähtyy `N` raakakehyksen jälkeen. |
| `--orient` | pois | Rakennuksen tästä sarjasta tuottamat tulokset käännetään kameran suunnan mukaan — kamerakohtaiset tulokset oman kameransa mukaan, yhdistetty indeksi pääkameran suuntaan — kuten `array-capture --orient` tekee. Raakakuvat pysyvät alkuperäisessä muodossaan; sarjan `manifest.json` tallentaa jokaisen kuvan suuntauksen ja valitun vaihtoehdon. |
| `--build` | pois | Pysäytyksen jälkeen sarja käsitellään välittömästi uudelleen (samoin kuin `array-build-video`). |
| `--products …` | `combined:index` | Kun käytössä on `--build`: mitkä videot luodaan (katso alla). |
| `--fps F` | `10` | Yhdessä `--build`:n kanssa: videon kuvataajuus (fps). |
| `--save-tiffs` | pois | Yhdessä `--build`:n kanssa: tallentaa myös kehyksittäin kalibroidut TIFF-tiedostot (kussakin on oma [unmix-clamp-mask](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on), kun pikseli on rajattu; rakennustuloksessa ne luetellaan nimellä `clamp_masks`). |
| `--gif` | pois | Kun `--build` on käytössä: kirjoita myös animoituja GIF-tiedostoja. |

### `array-build-video` — käsittele tallennettu sarja uudelleen offline-tilassa

Sovittaa kunkin raakakehyksen ajallisesti lähimpään tallennettuun `.daq`-lukemaan ja syöttää sen **saman säteilyvoimakkuus-/heijastavuus-/indeksiketjun läpi kuin tuontiputki**, jolloin tuloksena on yksi tai useampi video.

`--products` on pilkulla erotettu luettelo `kind:level`-kohteista, joissa `kind` ∈ `per_cam` | `combined` ja `level` ∈ `radiance` | `reflectance` | `index`. Pelkkä `level` (ilman `kind:`) on oletusarvoisesti `per_cam`. Oletusarvo on `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Lippu | Oletus | Kuvaus |
| --- | --- | --- |
| `--burst-dir DIR` | (vaaditaan) | Polku sarjakuvakansioon (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | `kind:level`-luettelo, kuten yllä. |
| `--fps F` | `10` | Videon tulostusnopeus (fps). |
| `--save-tiffs` | pois | Tallenna myös kehyksittäin kalibroidut TIFF-tiedostot videon (videoiden) ohella. |
| `--gif` | off | Kirjoita myös animoituja GIF-tiedostoja. |

**Videossa ei ole rajoitusmaskia** (yksi kuvavirta, 8-bittinen). Sen sijaan rakennustuloksessa on mukana `clamp_fractions`: jokaiselle videotuotteelle jokaisen kehyksen osuus pikseleistä, jotka on erotettu sen R/G/B-kanavan sisältämässä kaistassa — `Chloros:Band{R,G,B}UnmixClampFraction`, jolla kyseisen kehyksen säteily-/heijastusvoimakkuustuote leimattaisiin (yhdistettyä indeksiä varten, jokaisen mukana olevan kameran osalta). Suoritus tulostaa kunkin tuotteen huonoimman kehyksen `Clamp:`-riville. `--save-tiffs` lisää pikselikohtaiset naamiot TIFF-tiedostoihin, rivit `clamp_masks/<level>/` TIFF-tiedostojen `<level>/`-kansion viereen.**Keskeytetty koontiprosessi ei jätä rikkoutuneita videoita.** Jokainen video tallennetaan nimellä `<name>.partial.avi` ja saa lopullisen nimensä vasta, kun se on valmis, joten vanhempi video kyseisellä nimellä säilyy, kunnes sen korvaava video on kokonainen. Suorita koontiprosessi uudelleen korvataksesi jäljelle jääneen `.partial.avi`-tiedoston.

Suoritus tulostaa jokaisen tallentamansa videon, minkä jälkeen mahdolliset `Skipped:`-tuotteet (pyydetty kamera/taso – tai yhdistetty indeksi – joka ei tuottanut videota, syyn kera – esim. ei kalibrointipakettia, ei DLS-lukemaa heijastavuudelle tai ei puna-/NIR-kaistaparia yhdistetylle indeksille) sekä kaikki `Note:`-tuotteet. RGB-kameralle, jolta pyydettiin säteilyvoimakkuutta tai heijastavuutta, annetaan sen sijaan näyttövideo, jonka valkotasapaino on säädetty, kun kameran kalibrointipaketti on käytettävissä; ilman sitä renderoitu kamera mainitaan `Note:`-tiedostossa. Se **päättyy nollasta poikkeavalla arvolla**, kun pyydetty tuote ohitettiin tai sitä ei rakennettu pyydetyn mukaisesti (määriteltyä indeksilauseketta ei arvioida offline-tilassa); rakennetut videot kirjoitetaan ja luetellaan silti.

> **Valitse oikea tallennin.** `array-record` on *valvontatason* — se tallentaa renderoidun reaaliaikaisen yhdistelmäkuvan: käyttöliittymän yhdistelmänäkymän tai, jos käyttöliittymää ei ole auki, taustapalvelimen oman NDVI-näkymän (tai `--expr`). `array-burst` → `array-build-video` on *analyysitason* — se tallentaa raakaa anturidataa täydellä nopeudella ja rekonstruoi kalibroidut säteilyvoimakkuus-/heijastavuus-/indeksivideot jälkikäteen, ilman että reaaliaikaista näkymää tarvitaan.

### Mono (M3M) yksikaistaiset kamerat

**M3M**-sarja on Bayer**M3C**:n mono-vastine: yksi kapeakaistainen häiriösuodatin kameraa kohti (`M3M-<lens>-F<wavelength>`, esim. `M3M-L87-F685`), joten kenno tuottaa**yhden harmaasävykaistan** ilman Bayer-mosaiikkia. Demosaiikkia ei tarvitse suorittaa, kanavien välistä ylikuulumista ei tarvitse erottaa eikä valkotasapainoa tarvitse säätää — koko RGB-näytön väriprosessia ei yksinkertaisesti sovelleta.

Mitä tämä tarkoittaa CLI:ssä:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**tunnistavat mustavalkokameran ja**ohittavat sen yhden rivin viestillä** sen sijaan, että ne lähettäisivät merkityksettömiä asetuksia. Ne toimivat edelleen normaalisti RGB- tai Bayer-M3C-kameran kanssa samassa istunnossa.
- **Bayer-pikselimuoto tarkoittaa sen syvyyttä.** M3M tarjoaa vain monokromaattisia formaatteja, joten Bayer-formaatti komennosta `--pixel-format` tai `--preset` asettaa saman syvyyden monokromaattisen formaatin: `BayerRG8` ja `--preset high_speed` tuottavat tuloksen `Mono8`; `BayerRG12`, `high_quality` ja `triggered` tuottavat tuloksen `Mono12`; pakattu nimi pysyy pakattuna (`BayerRG12p` tuottaa `Mono12p`). Tiedostot ja `lattice format` nimeävät Mono-muodon. Taulukko, jossa sekoitetaan M3C- ja M3M-kameroita, käyttää yhtä muotoa samalla tavalla: Bayer-muoto jokaisessa M3C-kamerassa, sen Mono-syvyys jokaisessa M3M-kamerassa. Nimi on sen bittisyvyys: sama pätee myös päinvastoin, joten M3C-kameraa ei koskaan aseteta Mono-muotoon (`--pixel-format Mono12` on `BayerRG12` M3C-kamerassa, riippumatta siitä, kumpi kamera on ryhmän pääkamera), ja `lattice format --set` kartoitetaan samalla tavalla ja tulostaa sen asettaman formaatin.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** toimivat edelleen — säteilyvoimakkuus ja heijastavuus ovat *kaistakohtaisia* radiometrisiä karttoja, ja ne on määritelty täysin tarkasti yhdelle kaistalle. Monokromaattiset kuvat sisältävät **identiteetti**-anturivaste-matriisin (ei 3×3-sekoituksen purkua), joten taso läpäisee kalibrointilaskelmat muuttumattomana.
- **Yksi yksikanavainen kamera ei voi tuottaa kasvillisuusindeksiä.**NDVI, NDRE jne. vaativat vähintään kaksi kaistaa (esim. Red + NIR). Saadaksesi indeksin yksikanavaisella laitteistolla, suuntaa**useita** M3M-kameroita eri aallonpituuksille, yhdistä ne yhdeksi monikaistaiseksi pinoiksi ja laske *sen* indeksi:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Mono_660 --channel nir=Mono_850 \
  --save-multiband -o output/
```

`index --live` avaa kamerat itse (ns. [väliaikainen istunto](#multi-camera-transient-sessions)), joten älä liitä aluksi mitään: `--serials`, muuten profiilin käytössä olevat kamerat (katso [Indeksi / Kasvillisuusmatematiikka](#index--vegetation-maths)). Kun Chlorosilla on hallussaan mikä tahansa kamera (käyttöliittymä, `cam-connect`, `array-connect`) se hylätään HTTP-virheellä 409 `CAMERA_HELD` ja ohjelma sulkeutuu `1`; vapauta ne komennolla `chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`.

`--channel`-tunnisteiden on vastattava esiasetuksen kanavien nimiä **täsmälleen** (kirjainkoko on merkitsevä; NDVI-arvot ovat pienillä kirjaimilla `red`,`nir` — katso `--list-presets`), ja kaistan sivunimen on vastattava kaistaa kohdistetussa pinoissa (offline-tilassa hyväksytään myös 0-pohjaiset kaistaindeksit, esim. `--channel red=0 --channel nir=1`).

Koko pinoon sovellettava erottelija on mallimerkkijonossa oleva `M3M`-tunniste (se ei koskaan esiinny `M3C`-merkkijonossa), joka näkyy käyttöliittymässä (GUI) ja SDK:ssa muodossa `is_mono`.

---

## Isäntäkoneen verkkokortin asetukset ja säätö (LATTICE-järjestelmät)

LATTICE-kamerat lähettävät GVSP-virtaa isäntäkoneen Ethernet-verkkokortin kautta, joten monikamerajärjestelmissä verkkokortin **ajuri**ja**vastaanottorenkaan koko** ovat yhtä tärkeitä kuin linkin nopeus. Väärät asetukset näkyvät `FRAMES WILL DROP` / `Reduce ROI to enable`-virheenä Array Settings -paneelissa (sekä `lattice network-analysis` / SDK:n `analyze_array_network()`-kohdassa), vaikka kamerat itsessään olisivatkin kunnossa.

### USB 10GbE -sovittimet — Realtek RTL8157 (&quot;Realtek USB 10GbE Family Controller&quot;)

Alla olevat asetukset ovat Windows-ohjaimen ominaisuuksia. Linuxissa USB-sovittimelle asetetaan myös yläraja (400 MB/s x86_64-alustalla, 200 MB/s NVIDIA Jetsonissa) (katso [Array fps &amp; burst model](#array-fps--burst-model)), ja ytimen socket-vastaanottopuskureilla on merkitystä jokaiselle sovittimelle — katso [Linux-isäntäkoneet](#linux-hosts--socket-receive-buffers).

| Kohde | Vaadittu arvo | Miksi se on tärkeää |
| --- | --- | --- |
| **Ohjaimen versio**|**≥ v10.67 (tammikuu 2026)**, INF `rtump64x64sta.inf` | Vanha**2016**-ohjain (v10.65, `rtump64x64.inf`) käsittelee virrankatkaisua ja virhetarkistuksia virheellisesti, mikä aiheuttaa**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**-virheen sammutuksen, uudelleenkäynnistyksen tai lepotilan yhteydessä. Siirtymävaihe jumittuu (noin 5 minuutin aikakatkaisu), käyttäjä sammuttaa laitteen väkisin, ja toistuvat epäpuhtaat sammutukset**vääristävät WMI-tietovarastoa**(PowerShell/työkalut alkavat epäonnistua virheen `Invalid class` vuoksi) ja**jumiuttavat USB-pinoon** seuraavalla käynnistyksellä (verkkokortti ei aktivoidu; USB-asemat lakkaavat tunnistamasta laitteita). Päivitä realtek.comista (tai donglen toimittajalta) ennen kuin luotat puhtaisiin uudelleenkäynnistyksiin. |
| **Vastaanottopuskuri**— avainsana `ReceiveBufferLen` |**256**(ajurin enimmäisarvo) | Verkkokortin RX-rengas. Ajurin oletusarvo**32**jättää käytettäväksi vain noin 0,26 MB:n renkaan — aivan liian pienen monikameran sarjakuvaukselle — joten array-paneeli ilmoittaa virheen `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` ja estää yhteyksien muodostamisen. Arvolla**256**rengas on suuri (**~13,5 MB mitattuna laboratorion 10 GbE -isäntäkoneella**), mikä antaa RX-putkelle todellista liikkumavaraa monikameran GVSP-sarjakuvauksille. (Se, *yhdistyykö* tietty kokoonpano todella, ratkaistaan kahdella tarkistuksella — **tyhjennystä huomioivalla**pääsytarkistus ja**aggregoitu ylimerkintä**-tarkistus – ei raakaa purske-vs.-rengas-vertailua; katso [Array fps &amp; burst -malli](#array-fps--burst-model).) |
| **Vastaanotto-URB:t**— avainsana `PendingReceives` |**64** (maks.) | Lähetysvaiheessa olevat USB-pyyntölohkot; lisää yhdessä vastaanottopuskureiden kanssa purskeiden vaimentamiseksi. |
| **Jumbo-kehys** — avainsana `*JumboPacket` | **9014** | Tarvitaan 9000 tavun GVSP-paketeille (6× vähemmän paketteja/kehys kuin 1500). |

> ⚠️ **NIC-ohjaimen päivitys NOLLAA nämä lisäasetukset oletusarvoihin.**Kun olet päivittänyt tai vaihtanut sovittimen ohjaimen,**määritä uudelleen** `ReceiveBufferLen=256` ja `PendingReceives=64`, tai muuten laitepaneeli sulkeutuu uudelleen, vaikka ”laitteistossa ei ole tapahtunut muutoksia”. Tämä on yleisin syy siihen, että aiemmin toiminut laitteisto yhtäkkiä kieltäytyy muodostamasta yhteyttä.

Määritä asetukset **järjestelmänvalvojan oikeuksilla** toimivasta PowerShellistä (korvaa sovittimen nimi, esim. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` kattaa USB 10GbE -sovittimet.** Se tunnistaa nyt sovittimen tyypin ja säätää oikean receive-ring-avainsanan: `*ReceiveBuffers`→2048 PCIe-verkkokortteille (Intel I219, jne.) tai `ReceiveBufferLen`→256 + `PendingReceives`→64 Realtekin **USB** 10GbE-ohjaimelle (joka ei paljasta `*ReceiveBuffers`). Arvot rajoitetaan kunkin ohjaimen ilmoittamaan enimmäisarvoon (`NumericParameterMaxValue`), joten se ei koskaan kirjoita alueen ulkopuolista arvoa. Suorita komento **pääkäyttäjän oikeuksilla** toimivasta terminaalista; kuten kaikki rekisteripohjaiset säätötoimenpiteet, muutos astuu voimaan vasta, kun sovitin käynnistetään uudelleen tai järjestelmä käynnistetään uudelleen. Yllä mainitut manuaaliset `Set-NetAdapterAdvancedProperty`-komennot ovat edelleen hyvä vaihtoehto — ne astuvat voimaan välittömästi (sovitin sidotaan uudelleen) ilman uudelleenkäynnistystä.

### Linux
-isäntäkoneet — socket-vastaanottopuskuri

Linux
-ympäristössä, mukaan lukien NVIDIA Jetson, ytimen socket-vastaanottopuskuri on yhtä tärkeä kuin verkkokortin vastaanottorengas. Ytimen oletusasetuksilla (`net.core.rmem_max` / `net.core.rmem_default` = 212 992 tavua) kameran lähettämä GVSP-pakettien sarja täyttää stream-socketin yli ennen kuin Chloros
ehtii lukea sen. Paketit hylätään ytimessä, vaikka verkkokortti itse ei ilmoita menetyksiä, ja kuvasarja menettää kehysryhmiä ja hidastaa kuvataajuuttaan. Mitattuna nelikamerajärjestelmällä, joka on kytketty USB 10 GbE -sovittimen taakse, molempien arvojen nostaminen 32 MB:iin nosti järjestelmän kuvataajuuden 4,96:sta 7,52 fps:ään yhteyden muodostamisen jälkeen ja jatkuvan raakakuvauksen 4,7:stä 7,6 kuvaan sekunnissa, ilman että ytimen kautta hävisi yhtään pakettia.

- **.deb-paketti asettaa nämä arvot.** Paketin asennus kirjoittaa arvot `/etc/sysctl.d/60-chloros-gige.conf` (molemmat arvoiksi `33554432`) ja ottaa ne käyttöön välittömästi. Jos arvo on jo asetettu korkeammaksi, sitä ei muuteta.
- **`lattice network` tarkistaa ne.** Kun jompikumpi arvo on pienempi, se listaa `socket receive buffers too small for GigE Vision (…)` kohdassa *Löydetyt ongelmat*, ja taustapalvelimen loki sisältää saman varoituksen ensimmäisen kameran tai kameraryhmän kytkemisen yhteydessä.
- **Nostamaan niitä manuaalisesti** (seuraavaan uudelleenkäynnistykseen asti; paketin tiedosto soveltaa niitä jokaisella käynnistyksellä):

  ```bash
  sudo sysctl -w net.core.rmem_max=33554432 net.core.rmem_default=33554432
  ```

  `lattice network --fix` nostaa ja säilyttää ne myös, mutta vain kun taustapalvelu toimii root-käyttäjänä. Taustapalvelu, joka toimii käyttäjänäsi (joka on käynnistetty `chloros-backend`:llä tai [`chloros-backend@<user>`]-komennolla (#running-the-backend-as-a-service-headless)) käynnistetty taustapalvelu ei voi muuttaa ytimen asetuksia, vaan tulostaa sen sijaan rivin `sudo sysctl`.

**Automaattinen sovittimen säätö koskee vain kytkettyjä sovittimia.** Kun kamera kytketään ensimmäisen kerran, root-oikeuksilla toimiva taustapalvelu säätää langallisia sovittimia, joiden vastaanottorengas on pieni. Nyt se säätää vain liitettyjä sovittimia, joten irrotettu sisäinen portti säilyttää MTU-arvonsa, eikä se koske vastaanottopuskureita. `lattice network` ei enää mainitse irrotetun sovittimenMTU:ta ongelmana. `lattice network`, `--fix`, `lattice probe` ja sovittimen säätö ottavat huomioon vain langalliset Ethernet-portit, eivätkä Jetsonin CAN-väylää (`can0`) tai USB-laitemoodin yhteyksiä (`usb0`, `usb1`), joita se tarjoaa isäntätietokoneelle; kameroille tarkoitettu USB-Ethernet-sovitin lasketaan kuitenkin mukaan.

### Verkon perusteet (kaikki LATTICE-yhteydet)

- **Osoitteistus:** linkkikohtainen `169.254.0.0/16` (GigE Vision LLA). Isäntätietokone käyttää staattista osoitetta `169.254.x.x/16`; kamerat ja DAQ-E määrittävät itselleen osoitteet samasta alueesta. DHCP:tä tai yhdyskäytävää ei tarvita.
- **Paketin koko:**suositellaan jumbo-kokoa (9000), mutta anna automaattisen tunnistuksen selvittää se — se ohittaa kameran 1500 tavun ICMP-rajoituksen GVSP-tunnistuksen avulla, joten se päätyy jumbo-kokoon siellä, missä kaapeli todellisuudessa sitä tukee.**Jokainen kamera mitataan kerran jokaisessa backend-istunnossa ja uudelleen, kun sen yhteys muuttuu (toinen sovitin, toinen yhteysnopeus tai katkennut yhteys, kuten kytkimen vaihtaminen aiheuttaa); uudelleenkytkennät käyttävät tulosta uudelleen. Kun muutos on tapahtunut kytkimen takana, joka on pysynyt toiminnassa, käynnistä backend uudelleen mittausta varten.**Ensimmäinen yhteyden muodostus voi tässä tarkistuksessa toisinaan kestää muutaman minuutin, ja loki ilmoittaa siitä, kun näin tapahtuu. Määritä kiinnitys (pin) koodilla `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` vain, jos tiedät paremmin kuin koetin, ja käytä mieluummin komentoa kohti määritettyä kiinnitystä kuin pysyvää: kiinnitys ohittaa koettimen, joten jos reitti ei todellisuudessa kykene siirtämään 9000:ta,**jokainen** kaappaus aikakatkaistaan `SC_ERR_TIMEOUT -1011`:n avulla (katso [Ympäristömuuttujat](#environment-variables)).
- **RX-rengas skaalautuu `ReceiveBufferLen`:n mukaan:**oletusarvolla `32` käytettävissä oleva rengas on ~0,26 MB (liian pieni mille tahansa monikameran sarjakuvaukselle); maksimiarvolla `256` se on suuri (~13,5 Mt mitattuna laboratorion 10 GbE -isäntäkoneella), mikä tarjoaa todellista liikkumavaraa. Se, yhdistyykö konfiguraatio, päätetään sitten virrankulutusta huomioivan pääsytarkistuksen**ja** alla olevan aggregoidun ylimerkintätarkistuksen perusteella — ei raakana sarjakuvauskapasiteetin ja renkaan vertailuna.

### Array-kuvataajuus ja sarjakuvausmalli

Kuinka tulkita Array-asetuspaneelia (ja `lattice analyze-array` / SDK
:n `analyze_array_network`):

- **Burst-arvo lasketaan yhteen kamerakohtaisesti kunkin kameran todellisessa pikselimuodossa.**Mono**M3M**-kamerat lähettävät**Mono12 (2 B/px)**-muodossa tai, kun käytetään `--pixel-format`:ää, syvyyden Mono-muodossa (`BayerRG8` on `Mono8`, 1 bittiä/pikseli);**M3C**-Bayer-kamerat lähettävät 8- tai 12-bittistä dataa (M3C-laiteohjelmisto lähettää hiljaisesti BayerRG12-muotoa, vaikka pyydettäisiin BayerRG8-muotoa). Joten neljän kameran täysresoluutioinen kehys on**~12,6 MB, jos kaikki ovat 8-bittisiä, mutta ~25 MB, jos kolme kameraa on 12-bittisiä mono-kameroita**. Projisointi määrittää kunkin kameran formaatin sen mallin perusteella (identiteettivälimuisti), joten sarjakuvaus vastaa sitä, mitä kaapeli tosiasiassa välittää — ei yhtä kokoa sopivaa BayerRG8-oletusta.
- **USB-Ethernet-sovittimen enimmäisnopeus on 200 MB/s riippumatta sen nimilapusta.** Tehokkuustaulukko, joka muuntaa linkkinopeuden jatkuvaksi arvoksi, on johdettu PCIe:stä; USB-verkkokortti ilmoittaa *Ethernet*-linkkinopeutensa, mutta sitä rajoittavat USB-väylä ja sen ohjain. USB 10GbE -sovitinta käytettiin mittaamaan ~1063 MB/s ”jatkuvana” nopeutena – lukua, jota ei koskaan testattu – ja tuloksena oleva tahdistus vioitti 6–18 % kehyksistä, vaikka se silti ilmoitti terveen tavoite-fps-arvon. USB-liitännällä varustettujen verkkokorttien nopeus on nyt rajoitettu ehdottomasti: **200 MB/s** Windows
issa ja NVIDIA Jetsonissa, **400 MB/s** Linux
x86_64:llä (Linux
issa USB 2.0 -porttiin liitetylle sovittimelle on varattu **30 MB/s**, ja käyttöohjeissa kehotetaan liittämään se USB 3 -porttiin) (rajoitus johtuu isäntäkoneen USB-väylästä ja ajurista, joten se ei skaalaudu nimelliskapasiteetin mukaan; USB 1 GbE -sovitin saavuttaa noin 80 MB/s, eikä rajoitus vaikuta siihen). Linux
issa sovitin tunnistetaan sysfs:n kautta (varajärjestelmänä käytetään `ethtool -i`:n `bus-info`-tunnisteita). Jokainen luku on mitattu: Windows
issa sama neljän kameran ryhmä vioitti 6–10 % kehyksistä yli 200 MB/s:n nopeudella; Linux
x86_64:ssa sama sovitin toimi virheettömästi kaikilla nopeuksilla jopa 480 MB/s:iin asti (273 MB/s toimitettu, kameroiden oma raja). NVIDIA Jetsonilla sama sovitin toimi virheettömästi vain 200 MB/s:n nopeudella ja menetti kehyksiä ajoittain sitä suuremmilla nopeuksilla. `wire_ceiling_source` suorituskykyluettelossa kertoo tämän sanallisesti, ja `nic_is_usb` merkitsee sen. Ohita kumpi tahansa asetus XPR:lläOTX002475:llä.
- **Pääsy on drain-tietoinen, ei koko-burst-vs-ring-pohjainen.** Samanaikaisen burstin on sovittava vain *tilapäiseen ruuhkaan* = `max(0, Σ per-cam arrival − host drain) × emit_window`, ei koko burstiin. Nopean isäntäkoneen ja hitaan kameran verkostossa (**PCIe**10G-isäntä + 4× 1 GbE-cam: saapuminen ≈ 320 MB/s, tyhjennys ≈ 1063 MB/s) isäntä tyhjentää nopeammin kuin kamerat täyttävät, ruuhka ≈ 0, joten täysresoluutioinen sim-emit**päästää läpi**, vaikka 25 MB:n purske ylittääkin 13,5 MB:n renkaan. Kun samat neljä kameraa liitetään**USB** 10GbE-sovittimen taakse Windows
issa, tyhjennysnopeus on 200 MB/s, ei 1063 — saapuvien tietojen määrä ylittää sen, ja menetys näkyy vioittuneina kehyksinä eikä alhaisempana kehysnopeutena. 1 GbE -isäntäkoneella kameroiden 31,25 MB/s DLThr-alaraja saa saapuvan datan ylittämään poistokapasiteetin → se **estää**oikein (tämän tyyppisissä estotilanteissa pienennä ROI:ta tai käytä binningiä ≥ 2). Admittance on yksi**kahdesta** connect-portista — toinen on alla oleva aggregoitu ylimerkintätarkistus.
- **Ennustettu fps on hitain kameran sykli.**Samanaikaisessa lähetyksessä jokainen kamera lähettää kehyksensä samaan aikaan, joten matriisi syklisii hitain jäsenensä anturin lukemisen ja lähetysajan pidemmän ajan mukaan (kehyksen tavut ÷ sen lähetysnopeus: kameran**liityntäyhteys**, 1 GbE ≈ 80 MB/s, tai sen osuus isäntäkoneen kaistanleveyden ylärajasta, jos se on pienempi). Jokainen valotus menee päällekkäin edellisen kehyksen lukemisen kanssa, joten valotus ei tule mukaan sykliin: automaattinen valotus täyttää lähetysnopeuden määrittämän jakson, nostaa sitten vahvistusta, eikä hämärä kohde enää laske lähetysnopeutta (versiossa 1.2.0 se vielä laski). Ainoastaan manuaalinen valotus, joka kestää jaksoa pidempään, hidastaa laukaisua (katso [Valotus matriisin jäsenessä](#exposure-on-an-array-member)). Kameroiden lisääminen ei hidasta matriisia, ennen kuin niiden siirtokaistan leveys pienenee. Sarjaliikenteessä (`slip-emit-and-capture`) kameroiden lähetysajat sen sijaan lasketaan yhteen, ja kunkin jäsenen valotusaika otetaan huomioon porrastuksessa. Mitattu täydellä anturilla, 12-bittisenä, 9 kameran matriisilla, jossa on PCIe 10 GbE -isäntä: 10,7 fps tai 14,0 fps [pakatulla siirrolla](#packed-12-bit-transport---packed).
- **Jatkuvan tallennuksen jälkeen ennuste kertoo, mihin arvoon se vakiintui.** Vähintään 30 sekunnin tallennus automaattisella kuvausnopeudella tallentaa sen nopeuden, johon se vakiintui kyseisellä järjestelmäkokoonpanolla tällä tietokoneella; nopeuden säätimen lyhyet kokeilut nopeammalla nopeudella jätetään huomiotta. Siitä lähtien `array-connect`:n rivi ja Array Settings -paneeli ilmoittavat kyseisen nopeuden, kun se on mallin nopeutta alhaisempi, sekä sen rajoittaneen tekijän: `clock-bound` (matriisi toimi vakiintuneella nopeudella, kaappaussilmukka odotti laukaisukelloaan), `loop-bound` (kaappaussilmukalla ei ollut enää aikaa jäljellä), `capture-bound` (kaappaus jäi jälkeen ja ohitti ryhmiä) tai `measured` (ei kerro, mikä niistä). Mallin arvo on `wire_fps_max`. Taulukko nousee edelleen siihen, mitä verkko kantaa.
- **Taulukko toimii missä tahansa verkossa — kuvataajuus määrää rajan.**Jokainen kamera saa tasaisen osuuden**törmäysturvallisesta kaapelirajasta**(`sustained × sim_emit_factor`), joka tahdistetaan GevSCPD:n kanssa, ja kaikki kamerat laukaisevat edelleen samalla laukaisimella. Hitaalla kaapelilla osuus on pieni ja kuvataajuus laskee sen mukana: 8 täysresoluutioista 12-bittistä kameraa 1 GbE:llä ilman jumbo-kehyksiä saavat kukin 52 / 8 = 6,5 MB/s, eli noin**1 fps**(noin 1,3 kuvaa sekunnissa pakattuna), kun taas 10G-isäntäkoneella ne toimivat 11–14 kuvan sekuntinopeudella. Jos kameroita ei voida tahdistaa niin hitaasti — selvitä tämä lukemalla kunkin kameran GevSCPD-alue — järjestelmä siirtyy**sarjaliikenteiseen**lähetystilaan (`slip-emit-and-capture`): kamerat käyttävät kaapelia vuorotellen täydellä linkkinopeudella, laukaistuna peräkkäin, joten jokainen kehys saapuu edelleen, mutta jäsenet**eivät ole näkyvissä samanaikaisesti**. `analyze-array` ja connect-komennon tuloste nimeävät tilan; aseta `--force-tier sim-capture-sim-emit`-asetukseksi ”refuse” estääksesi varajärjestelmän käytön. Kehyksen koko ei vaikuta tähän jakamiseen, joten**kuvien yhdistely ja pienempi ROI-alue nostavat kuvataajuutta, mutta eivät muuta sitä, mahtuuko matriisi verkkoon**. Vain kaapeli, joka ei kykene siirtämään taulukkoa edes sarjallisesti, on**ylikuormitettu**— esim. kun kameroita on huomattavasti enemmän kuin yksi kaapeli pystyy käsittelemään tai kun monet muut kamerat lähettävät dataa samalla kaapelilla — ja tällöin oireena on GVSP-pakettihäviö, ei alhaisempi siirtonopeus: `analyze-array` nollaa saavutettavissa olevat fps-arvot ja tulostaa `** OVER-SUBSCRIBED**`, yhteys muodostetaan voimakkaan varoituksen kera, ja HTTP
-yhteys muodostetaan `pin_resolution: true`:n (tiukka tila) **hylkää** sen sijaan. Ratkaisuina ovat kameramäärän vähentäminen, jumbo-kehysten käyttö päästä päähän tai nopeampi verkkokortti. `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` alentaa hylkäyksen voimakkaaksi varoitukseksi testikäytössä — katso [Ympäristömuuttujat](#environment-variables).

### Järjestelmän kunto — mikä alijärjestelmä menettää kehyksiä

Yhdistetyn järjestelmän `GET /api/camera/array/<array_id>/capability` sisältää reaaliaikaisen
`health`-lohkon, jota arvioidaan uudelleen **10 sekunnin** pituisella liukuvalla ikkunalla. Se jakaa kehysten menetykset
kahteen syyhyn, jotka vaativat vastakkaisia korjauksia, sen sijaan että raportoisi yhtä ”epätäydellistä”
astetta, joka ei nimeä kumpaakaan:

| Kenttä | Mitä se tarkoittaa | Mikä alijärjestelmä |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (sarjaporttikohtainen) | Kehys **saapui, mutta oli rakenteellisesti viallinen**— GVSP-paketin menetys. |**Verkko**: kaapelibudjetti, tahdistus, NIC RX-rengas, MTU |
| `never_arrived_rate_pct` (sarjanumeroittain) | Kehystä **ei saapunut lainkaan**— kamera ei laukaisi tai siitä ei lähtenyt mitään. |**Laukaisu / synkronointi**: M8-kaapeli, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Kunkin kameran huonoin suorituskyky. | — |
| `per_cam_rate_pct` | Yhdistetty epätäydellisyysaste kameraa kohti (molemmat syyt yhdessä). | — |
| `stable_for_seconds` | Kuinka kauan kukin kamera on pysynyt alle 0,01 %:n. | — |

Yli 5 %:n rajan ylittyessä taustapalvelin kirjaa lokiin `[array-health <id>] WARN`-rivin, jossa mainitaan jakautuma —
ensimmäisen ylityksen yhteydessä, vakavuusluokan muuttuessa, kerran minuutissa niin kauan kuin tilanne jatkuu ja kerran, kun
tilanne korjaantuu. Viallinen puoli kirjoittaa `[gvsp-corrupt <SN>]`-merkinnän backend-lokitiedostoon
ensimmäisen osuman yhteydessä kameraa ja syytä kohden, minkä jälkeen yhteenveto 60 sekunnin välein; sovelluksen lokipaneeli näyttää sen sijaan
`[array-health <id>]`-rivin. Jokainen arviointi tallentuu edelleen taustalokitiedostoon;
laskurit päivittyvät jokaisen puskurin yhteydessä riippumatta siitä, mitä tulostetaan.

Sama tietue ilmoittaa numeron, johon koko allokointi perustuu:

| Kenttä | Mitä se tarkoittaa |
| --- | --- |
| `wire_ceiling_mbps` | Isännän voimassa oleva jatkuva siirtokapasiteetti, MB/s. |
| `wire_ceiling_source` | Mistä kyseinen luku on peräisin, sanoin — esim. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` (Linux
-järjestelmässä x86_64, `USB-capped 400 MB/s (…; sysfs: bus=usb (usb2/2-1/2-1:1.0))`) tai `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, kun `--wire-ceiling-mbps` (tai käyttöliittymän **Wire Budget**-kenttä) on asettanut sen. |
| `nic_is_usb` | `true` USB-Ethernet-sovittimelle — katso yllä oleva USB-rajoitus. |

**Tulkinta:** nollasta poikkeava `gvsp_corrupt_rate_pct`-arvo ja `never_arrived_rate_pct`-arvo 0
tarkoittaa, että laukaisu ja kaapelisynkronointi ovat täydellisiä ja 100 % häviöstä johtuu verkkoreitistä
— pienennä `--wire-ceiling-mbps`-arvoa ja muodosta yhteys uudelleen. Päinvastainen kuvio viittaa sen sijaan
synkronointikaapeliin tai laukaisulinjaan.

> **`--target-fps` ei ole ratkaisu vioittuneiden kehysten ongelmaan.** GevSCPD-taajuus määritetään
> kerran yhteyden muodostamisen yhteydessä, joten laukaisutaajuuden laskeminen muuttaa käyttöjaksoa, ei
> samanaikaisten lähetyspurkausten taajuutta. Mitattu 5-kertainen vaatimusten leikkaus ei tuottanut parannusta;
> kaapelin enimmäisnopeuden laskeminen 240:stä 200 MB/s:iin laski saman laitteiston virheprosentin 10,4 %:sta
> 0,00 %:iin.

> **Käynnissä oleva matriisi ei voi suunnitella itseään uudelleen LATTICE (M3C / M3M) -laiteohjelmistolla.** Se
> ei voi korjata tätä itse; katkaise yhteys ja muodosta se uudelleen, jotta yhteyden muodostusajankohdan valitsin voi
> suunnitella järjestelmän uudelleen uuden ylärajan mukaisesti.

### Oire → korjaus

| Oire (Järjestelyasetukset / yhteys / `analyze_array_network`) | Syy | Korjaus |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` palautuu arvoon 32 (yleensä ajuripäivityksen jälkeen) | Aseta `ReceiveBufferLen`→256, `PendingReceives`→64; avaa paneeli uudelleen (käynnistä taustapalvelu uudelleen, jos se on tallentanut vanhan rengaskokoa välimuistiin) |
| Käynnistys/sammutus jumittuu; myöhemmin `Invalid class` WMI-virheitä, verkkokorttia ei voi ottaa käyttöön, USB-asemat puuttuvat | Vanha vuoden 2016 Realtek USB 10GbE -ajuri → BSOD `0x9F` → pakotetut virrankatkaisut | Päivitä sovittimen ajuri versioon ≥ v10.67 (2026), ja määritä sitten uudelleen yllä olevat vastaanottorengasasetukset |
| Array muodostaa yhteyden täydellä anturilla, mutta kuvanopeus on alhainen | Auto-tila säilyttää täyden anturin ja antaa kuvanopeuden laskea linjan kantokyvyn mukaiseksi | Valitse yksi nopeimmista vaihtoehdoista, jotka näkyvät yhteyden tulostusluettelossa (`--binning 2` / `4` tai pienempi `--width`/`--height`), ota käyttöön jumbo-kehykset tai päivitä yhteys |
| Array ilmoittaa kunnollisen tavoite-fps:n, mutta toimittaa vain murto-osan siitä; `health.gvsp_corrupt_rate_pct` ei ole nolla, `never_arrived_rate_pct` 0 | Isäntäkoneen arvioitu siirtokapasiteetti on suurempi kuin sen todellinen kapasiteetti (tyypillistä USB-Ethernet-sovittimessa, kapealla PCIe-kaistalla tai jaetussa verkossa) | Yhdistä uudelleen pienemmällä `--wire-ceiling-mbps`-arvolla ja tarkista toimintakunto-lohko uudelleen. **Ei** `--target-fps` — GevSCPD-taajuuden säätö on kiinteä yhteyden muodostuksen yhteydessä |
| Linux
: taulukko menettää kehysryhmiä ja hidastaa kehysnopeuttaan, vaikka verkkokortti ei ilmoita pudotuksia; `lattice network` listaa `socket receive buffers too small` | Ytimen socket-vastaanottopuskureiden koko on oletusarvoinen, joten kamerapaketit hylätään ytimessä | Nosta `net.core.rmem_max` / `rmem_default` arvoon 32 Mt — katso [Linux
-isäntäkoneet](#linux-hosts--socket-receive-buffers). .deb-paketti tekee tämän asennuksen yhteydessä |
| Kameroita puuttuu julkaistuista ryhmistä; `health.never_arrived_rate_pct` on eri kuin nolla, `gvsp_corrupt_rate_pct` 0 | Laukaisu-/synkronointireitti — kamerat eivät laukea, ei verkko-ongelma | Tarkista M8-synkronointikaapeli ja `--line`; varmista, että jokainen jäsen on kytketty päälle (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` ylitetty `analyze-array`:ssa tai yhteyden muodostamisen epäonnistuminen tiukassa tilassa (`pin_resolution`; `array over-subscribes the wire`) | Kaapeli ei kykene siirtämään dataa millään kamerien saavuttamalla tahdilla, samanaikaisesti tai vuorotellen — kameroita on huomattavasti enemmän kuin kaapeli pystyy jakamaan, tai muut kamerat lähettävät dataa samanaikaisesti | Vähemmän kameroita, jumbo-kehykset päästä päähän tai nopeampi verkkokortti. **ROI/binning EI auta** (jakaminen on kehyskokosta riippumatonta). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` ohittaa virheen testausympäristössä (hyväksyy pakettihäviön) |
| Kameraryhmä kytkeytyy `slip-emit-and-capture`:ään (sarjaliitäntä) hitaassa verkossa | Kameroiden tahdistusta ei voida hidastaa tarpeeksi, jotta ne voisivat jakaa kaapelin samanaikaisesti, joten ne toimivat vuorotellen; kehyksiä ei tallenneta samanaikaisesti | Jumbo-kehykset tai nopeampi verkkokortti palauttavat samanaikaisen tallennuksen; `--force-tier sim-capture-sim-emit` vaatii sitä (ja menettää sen sijaan paketteja) |

---

## `chloros-cli daq`

Spektrisensorin komennot. `pool-*`-komennot ovat kevyitä HTTP
-asiakasohjelmia, jotka ohjaavat anturia taustapalvelimen pysyvän poolin kautta, joten ne tarvitsevat käynnissä olevan taustapalvelimen (katso [Käynnistä taustapalvelin](#start-the-backend)). Taustapalvelu hallitsee siirtoyhteyttä, joten CLI
- ja SDK
-skriptit jakavat yhden aktiivisen käsittelykäsittimen sen sijaan, että ne kilpailevat sarjaportista. `pool-discover` on ainoa tapa selvittää DAQ-M:n BLE-MAC-osoite.

> **`chloros-cli daq --help`** (tai `-h` tai `daq help`) listaa `pool-*`-alikomennot (Linux
issa yhdessä buildin sisältämien suorien laitteistokomentojen kanssa), ja komento `chloros-cli daq <subcommand> --help` (tai `daq help <subcommand>`) näyttää kunkin alikomennon vaihtoehdot. Ne kattavat yhteyden muodostamisen, stream- ja record-komennot, kalibroitujen `.daq`-tiedostojen tallennuksen, kapasiteettiprofiilien vaihdon ja DAQ:n liittämisen kameraan sekä DAQ-E:n osalta sen verkkoasetukset, PTP:n, uudelleenkäynnistyksen ja laiteohjelmistopäivitykset. DAQ-laitetta voidaan ohjata myöPython
illa komennolla `chloros_sdk.connect_daq_sensor()`, joka käyttää samaa yhdistettyä polkua.
>
> **Käytä komentoja `pool-*` kaikilla alustoilla.** Ne toimivat samalla tavalla Windows
issa, Linux
issa ja Jetsonissa. Windows
issa `daq discover` ja `daq list` ovat `pool-discover`:n aliaksia, ja kaikki muut `daq`-komennot, jotka eivät ole `pool-*`, päättää `1`:n komennolla `DAQ direct-hardware commands require the full daq package`. Linux
issa ja Jetsonissa `daq discover`, `daq list` ja muut kuin `pool-*`- ja `daq`-komennot ovat DAQ-työkalun suoria-laitteistokomentoja: ne avaavat anturin CLI
:n omassa prosessissa, backendin poolin ulkopuolella (eli ei anturissa, jota backend, GUI tai skripti hallitsee), ja siellä ne ottavat vastaan komennot `daq discover`, `--only` ja `--timeout`, mutta eivät komentoa `--json`. Tässä viitteessä käsitellään ainoastaan `pool-*`-komentoja.

### DAQ-anturin ensimmäisen yhteyden muodostamisen työnkulku

```bash
# 1. Scan every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U serial port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq pool-discover
chloros-cli daq pool-discover --only ble           # BLE only
chloros-cli daq pool-discover --json               # machine-readable

# 2. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect                                   # smart-detect
chloros-cli daq pool-connect --port /dev/ttyUSB0               # DAQ-U on a serial port (Windows: COM3)
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 3. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 4. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

#    Bind it to a camera, so that camera's captures use and save its readings
chloros-cli daq pool-bind --camera 213800234 CB-7C-A8-2E-5F

# 5. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 6. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### `pool-*`-viite

| Alikomento | Tarkoitus |
| --- | --- |
| `daq pool-connect` (smart-detect) | Avaa anturi taustapoolissa. |
| `daq pool-connect --port PORT` (`-p`) | DAQ-U tietyllä sarjaportilla. |
| `daq pool-connect --ble` | DAQ-M BLE:n kautta, MAC-osoite skannataan automaattisesti. |
| `daq pool-connect --mac MAC` (`-m`) | DAQ-M BLE:n kautta tunnetulla MAC-osoitteella (edellyttää `--ble`:ää). |
| `daq pool-connect --eth-host HOST` | DAQ-E Ethernetin kautta tunnetulla isäntänimellä tai IP-osoitteella (edellyttää `--eth`:ää). |
| `daq pool-connect --eth` | DAQ-E Ethernetin kautta, isäntä tunnistetaan automaattisesti (mDNS + ARP-varajärjestelmä; toimii tyhjästä ARP-välimuistista Windows
issa ja Linux
issa). |
| `daq pool-connect --integration-time MS (-t) --frame-avg N (-f) --no-ae` | Integraatioikkunan / AE-tilan säätö. |
| `daq pool-connect --no-stream` | Yhdistä, mutta älä aloita vielä datavirtaa (jatka komennolla `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Cap-korjausprofiili. Oletusarvo taustapuolella on `sunshine_cosine`; `none` on paljas anturi. [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor) ohittaa sen (kiinteä hajotin). **Anturit toimitetaan ilman suojakantta, joten nimeä asentamasi suojakansi:** oletusarvona tallennetaan oletettu (`cap_id_source=auto_default`), ja käsittely varoittaa siitä, koska paljas DAQ-U antaa auringonvalokorjauksen alla noin 30-kertaisen lukeman. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Skannaa jokainen kuljetusyksikkö etsimällä antureita, joihin voisit muodostaa yhteyden, ilman että muodostat yhteyttä. **Näin löydät DAQ-M:n BLE-MAC-osoitteen.** `--timeout` on kullekin siirtoyhteydelle (oletus 5 s, 1–20). Poolissa jo avoimina olevia antureita ei luetella — kytketty DAQ-M lopettaa mainostamisen — joten käytä niihin komentoa `pool-list`. `usb` listaa vain USB-sarjaliikennelaitteet: tietokoneen omat sarjaportit (`COM1`, Linux
in `/dev/ttyS*`) eivät voi olla antureita, joten ne jätetään pois. |
| `daq pool-list` | Näyttää kaikki anturit taustapoolissa. |
| `daq pool-bind (--camera SERIAL \| --array ARRAY_ID) {SENSOR \| --clear}` | Liittää kameran tai matriisin DAQ:hen, kuten graafisen käyttöliittymän valonanturin valinta tekee: sen tallennukset käyttävät, sovittavat ja tallentavat kyseisen DAQ:n lukemia. `SENSOR` on anturitunnus, joka on peräisin `pool-list`:stä, tai osoite, josta DAQ:ta tavoitetaan. `--clear` (ei `SENSOR`) poistaa liitännän, ja tallennukset siirtyvät käyttämään tuoreinta DAQ:ta. Sidonta kestää, kunnes taustapalvelu käynnistyy uudelleen tai kunnes kyseinen kamera, matriisi tai DAQ irrotetaan: irrottaminen (tässä tapauksessa SDK
:sta tai sovelluksesta) katkaisee kaikki irrotettavan laitteen sidokset riippumatta siitä, kuka ne on asettanut, ja matriisin jäsenten sidokset matriisin kanssa. Kameran on oltava kytkettynä, avoimessa projektissa tai aiemmin avattuna tällä tietokoneella, ja matriisin on oltava kytkettynä tai avoimessa projektissa; kaikki muut sarjanumerot tai tunnisteet hylätään nimen perusteella eikä mitään sidota. |
| `daq pool-disconnect {--sensor-id ID \| --all}` | Vapautus — riippumatta siitä, kuka avasi anturin, joten se vapauttaa myös anturin, jonka kaatunut skripti jätti poolin (mahdolliset tallennukset sillä saatetaan ensin loppuun). Anturilla, jota ei ole poolissa, ei ole mitään vapautettavaa: se tulostaa `ID: already released (not in the pool)` ja lopettaa `0`; vapautus, jonka taustapalvelin hylkäsi tai jota se ei vahvistanut, lopettaa `1`. SDK
: `DAQSensorSession.force_disconnect()`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Viimeisimmät N spektrikehystä (oletus 1). |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Jatka / keskeytä suoratoisto. |
| `daq pool-record --sensor-id ID [--duration SEC] [-o DIR] [--device-name NAME] [--confirm-after S] [--heartbeat S]` / `--stop` | Käynnistä / lopeta .daq-tallennus. `-o`:n oletusarvona on `~/Documents/DAQ Live View` taustapalvelimen tietokoneella (suhteellinen polku on sen kansion alla, josta komento suoritetaan); `--duration` (`-d`, oletusarvo `0` = kunnes `--stop`). `--confirm-after S` antaa varoituksen, jos lukemaa ei ole saapunut S sekunnin kuluttua käynnistyksestä (oletusarvo 20; 0 poistaa varoituksen käytöstä); komennolla `--duration` `--heartbeat S` tulostaa lukemien määrän S sekunnin välein ja antaa varoituksen, kun määrä lakkaa kasvamasta (oletusarvo 30; arvo 0 poistaa toiminnon käytöstä). DAQ-E:llä, jonka lähetys keskeytyy, jatkuu samassa tiedostossa, kun taustapalvelin on muodostanut yhteyden uudelleen, ja `--stop` listaa aukot ([DAQ-E, joka lakkaa lähettämästä](#a-daq-e-that-stops-delivering)). Ctrl+C lopettaa `--duration`-tallennuksen ennenaikaisesti ja ilmoittaa siitä; ilman `--duration`-asetusta tallennus jatkuu Ctrl+C:n jälkeen (poistu `130`) kunnes `--stop`. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Vaihda kapasitanssikorjausprofiilia ajon aikana. `--cap-id` vaaditaan; `--cap-id none` poistaa kapasitanssikorjauksen (paljas anturi). DAQ-E-S säilyttää `as_recorded`-asetuksen riippumatta siitä, mikä on asetettu. |
| `daq pool-enable-ptp --sensor-id ID` | Kytke PTP takaisin päälle DAQ-E-laitteelle, jonka PTP on pois päältä. Asetus tallennetaan laitteeseen, joten pois päältä jätetty laite pysyy pois päältä — sen aikaleimat eivät ole PTP-synkronoituja — kunnes tämä komento suoritetaan. `pool-list` merkitsee tällaisen laitteen (`!! PTP is switched OFF`) ja tulostaa tämän komennon sille. Chloros
ei koskaan kytke laitetta takaisin päälle itsestään, koska pois päältä -tila voi olla tarkoituksellinen. |
| `daq pool-info --sensor-id ID [--json]` | DAQ-E:n omat asetukset: sen laiteohjelmisto (ja se, sisältääkö tämä Chloros
-versio uudempaa), sen osoite ja miten se on saatu (DHCP / link-local / staattinen), tallennetut verkkoasetukset siihen asti, kunnes anturi suorittaa ne (yksi tallennettu, jossa `--no-reboot` sisältyy), virta-päivitysvirta ja PTP. Anturissa, jonka virta ei saavuta tätä tietokonetta, se lukee anturin uudella yhteydellä ja ilmoittaa siitä. |
| `daq pool-set-network --sensor-id ID {--auto \| --static IP [--mask M] [--gateway G]} [--no-reboot] [--force] [--timeout S]` | Aseta DAQ-E käyttämään DHCP:tä (`--auto`) tai staattiseen osoitteeseen (`--mask` oletus `255.255.255.0`). Anturi käynnistyy uudelleen asetusten voimaansaattamiseksi ja muodostaa uuden yhteyden aiempien asetustensa mukaisesti, odottaen sitä enintään `--timeout` sekuntia (oletusarvo 45); `--no-reboot` tallentaa asetuksen vain. Katso [DAQ-E-laitteen asetukset](#daq-e-device-settings). |
| `daq pool-set-auto-stream --sensor-id ID {--on \| --off}` | Määrittää, aloittaako DAQ-E datan lähettämisen virran kytkemisen yhteydessä, vaikka ohjelmistoa ei ole kytketty (tallennetaan anturiin). Chloros
käynnistää datan lähettämisen joka tapauksessa. |
| `daq pool-reboot --sensor-id ID [--timeout S]` | Käynnistä DAQ-E uudelleen ja liitä se uudelleen aiemmilla asetuksilla (odottaa enintään `--timeout`, oletusarvo 45 s). |
| `daq pool-update-firmware --sensor-id ID [--timeout S]` | Asenna tämän Chloros
-julkaisun mukana toimitettu DAQ-E-laiteohjelmisto anturiin, jossa on vanhempi versio, ja liitä se sitten uudelleen (noin minuutti; älä sammuta laitetta; `--timeout` oletusarvo 45 s, ennen kuin se palaa käyttöön). Jo ajan tasalla olevaa anturia ei käsitellä. |

### DAQ-E, joka lakkaa lähettämästä

Backend kytkee uudelleen uima-altaassa olevan DAQ-E-anturin, joka ei lähetä spektriä 20 sekunnin ajan. Tämä tapahtuu anturia virranlähteenä toimivan kytkimen virran katkaisemisen ja uudelleenkytkemisen avulla: anturi käynnistyy uudelleen kytkimen mukana eikä lähetä dataa uudelleen, ennen kuin sille annetaan käsky (ellei `pool-set-auto-stream --on`).

- **Se tarkistaa ensin, että kyseessä on sama anturi.** Se kysyy anturin aiemmasta osoitteesta, muuttamatta mitään, anturin nimeä, MAC-osoitetta ja sarjanumeroa. Se kysyy myös anturin `daq-e-<id>.local`-nimellä, mutta luottaa vastaukseen vain, jos se tulee anturin omalta verkkosovittimelta (MAC). Mitään muuta vastaajaa ei kytketä uudelleen tai siihen ei kirjoiteta.
- **Se muodostaa yhteyden uudelleen istunnon asetuksilla:** integraatioaika, kuvan keskiarvoistaminen, automaattinen valotus ja valotusprofiili säilytetään ennallaan, ei käytetä anturin käynnistysasetuksia. Anturi, jonka PTP-toiminto oli päällä ja joka palasi pois päältä, kytketään takaisin päälle. Sitten se käynnistää datavirran uudelleen sekä tietokoneen vastaanottimen sille.
- **Se yrittää uudelleen yhä pidemmillä väliajoilla:** heti, sitten 5, 10, 20 ja 40 sekunnin kuluttua, ja sen jälkeen 60 sekunnin välein niin kauan kuin anturi on poissa. Jokaisella yrityksellä kysytään anturilta vain, mikä se on. Anturi, joka vastaa mutta ei vieläkään lähetä mitään kolmen uudelleenkytkennän jälkeen, jätetään rauhaan ja siitä raportoidaan (`auto-reconnect GAVE UP`). `pool-stream --start` yrittää uudelleen.
- **`pool-list` näyttää tilanteen.** Kun anturi on hiljaa, sen `STREAM FAULT`-rivi kertoo, mitä yhteyden palautusyritys tekee. Rivi tyhjenee ensimmäisen spektrin myötä, ja anturin lukema on tämän jälkeen `reconnected automatically N time(s)`.
- **Tallennus jatkuu samassa `.daq`-tiedostossa.** Tiedosto merkitsee jokaisen katkon kahdella rivillä: `event_type` 5 viimeisessä spektrissä ennen katkoa ja 6 ensimmäisessä sen jälkeen, kummassakin vain aikaleima. `pool-record --duration` tulostaa `!! STREAM LOST …`:n, kun anturi on poissa, ja `stream resumed after …`:n, kun se palaa. `pool-record --stop` luettelee jokaisen katkon ja antaa varoituksen, kuten se tekee myös tyhjän tiedoston kohdalla (tallenne, jossa ei ole lukemia, poistuu `1`:stä).
- **Tallenne säilyttää yhden kalibroinnin.** Jos anturi palaa takaisin eri kalibroinnilla tallennuksen ollessa käynnissä, tallenteen loppuosa säilyttää sen kalibroinnin, jolla se alkoi, ja pysäytysvaroitus ilmoittaa tämän. `pool-disconnect` + `pool-connect` ottaa uuden kalibroinnin käyttöön.
- **Käskyjäsi noudatetaan.** Anturia, jota olet keskeyttänyt komennolla `pool-disconnect` tai jota olet pysäyttänyt komennolla `pool-stream --stop`, ei koskaan kytketä uudelleen, vaikka uudelleenkytkentä olisi jo käynnissä.
- **`pool-info` hiljaisella anturilla** lukee anturin uudella yhteydellä ja lisää `!! stream NOT reaching this computer`. Anturi, joka ei vastaa, raportoidaan välittömästi. `pool-list` ei enää näytä PTP-tilaa anturille, joka on lakannut vastaamasta.
- DAQ-U- ja DAQ-M-laitteita ei käsitellä tässä. DAQ-M muodostaa yhteyden uudelleen Bluetoothin kautta itsestään, ja USB-liitännästä irrotettu DAQ-U tarvitsee uudelleen `pool-connect`-komennon.

### DAQ-E-S — kiinteä hajotin ja kallistusanturi

**DAQ-E-S** on DAQ-E, jossa on kaksi ohjelmiston kannalta merkittävää eroa. Kaikki muu — laitteen tunnistus, yhteyden muodostaminen, PTP, tallennus sekä alla mainitut laiteasetukset — toimii täsmälleen samalla tavalla kuin DAQ-E:ssä.**Sen diffuusori on kiinteä, joten se ei vaadi korkkikorjausta.** Sunshine-tyyppinen diffuusori on sisäänrakennettu ja oli laitteessa, kun sen tehdaskalibrointi mitattiin, joten kalibrointi osoittaa jo todellista alaspäin suuntautuvaa säteilyvoimakkuutta. Chloros
asettaa sen korkin arvoksi `as_recorded` (ei profiilia) riippumatta siitä, mitä pyydetään: sille nimetty yläraja (`sunshine_cosine`, `fov_*`, `none`) ohitetaan virheilmoituksella, ja sen tallenteissa käytetään arvoa `cap_id_source=model`, joten oletetun ylärajan varoitus ei koskaan laukea sen kohdalla. Älä myöskään aseta ylärajaa sen dataan manuaalisesti: yläraja `sunshine_cosine` aiheuttaisi noin 11-kertaisen lukeman.

**Siinä on 3-akselinen kiihtyvyysanturi.** Auringonvalon hajottajan kosinivaste on voimassa vain, kun se on vaakasuorassa, joten kallistunut anturi ilmoittaa väärän säteilyvoimakkuuden — ja heijastavuus perii virheen — ilman muita oireita. Jokainen DAQ-E-S-lukema sisältää siksi laitteen asennon:

| Kenttä | Merkitys |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Kulmat asteina. `None`, kun laiteohjelmisto merkitsee kulman määrittelemättömäksi (vapaapudotus, kuollut akseli). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Kiihtyvyys milli-g:inä. Jos `mag_mg` poikkeaa huomattavasti arvosta ~1000, laite oli liikkeessä eikä kulmia voida tulkita kyseisen näytteen osalta. |
| `cal_applied` | Asetettu: akselit ja kulmat ovat diffuusorin koordinaatistossa (laitteen kiinnityskalibrointi on otettu huomioon). Tyhjä: raaka kortin koordinaatisto, joka voi näyttää jopa ~2,3°:n kallistuksen, joka ei ole todellinen. |
| `sample_age_ms` | Kiihtyvyysanturin näytteen ikä, kun spektri lähetettiin. |
| `temp_raw` | Kalibroimaton piisirun lämpötilan **trendi** — vain erot, ei koskaan lämpömittarin lukemaa. |

`daq pool-latest --json` palauttaa ne kunkin kehyksen `imu`-avaimen alle (`null` DAQ-U-, DAQ-M- tai tavallisessa DAQ-E-laitteessa). DAQ-E-S-laitteesta tallennettu `.daq` tallentaa samat arvot `imu_*`-sarakkeisiinsa (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) jokaisen spektrin viereen, jotta lukija voi erottaa pilven kolhiintuneesta kolmijalasta. Kaikissa muissa malleissa nämä sarakkeet ovat tyhjiä.

Asennon datavirta raportoidaan erikseen: `daq pool-list` tulostaa `!! ATTITUDE STREAM FAULT (spectra unaffected)` laitteen ollessa pois käytöstä sekä sen, kuinka monesta viasta se toipui sen jälkeen. Siellä esiintyvä vika ei koskaan aiheuta aukkoa spektritallenteessa. `daq pool-discover` listaa laitteen nimellä `DAQ-E-S` (nimellä `DAQ-E`, jos se ei vastaa tunnistustarkistukseen noin 2 sekunnin kuluessa).

### DAQ-E-laitteen asetukset

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330

# A fixed address on the network this computer is on (192.168.1.x)
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1

# Back to the factory setting: DHCP, falling back to a 169.254.x.x
# link-local address when no DHCP server answers
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto

chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

- **`--static` ilman `--gateway`:ää ei tallenna yhdyskäytävää** (kaikki aiemmin asetetut yhdyskäytävät hylätään), mikä sopii kameraverkkoon, jossa ei ole reititintä.
- **Tällä tietokoneella on oltava staattinen osoite.** Chloros
hylkää sellaisen, jonka aliverkko ei sisällä tämän tietokoneen osoitetta anturin verkossa: uudelleenkäynnistyksen jälkeen mikään ei pääse anturiin muuttamaan asetusta takaisin. Hyväksy `--force` vain, jos tälle tietokoneelle määritetään uusi osoite tai se reititetään uuteen aliverkkoon.
- **Uudelleenkäynnistys hylätään, kun anturi tallentaa.** Lopeta ensin tallennus (`pool-record --stop`). Uudelleenkäynnistyksen alkaessa käynnissä ollut tallennus suljetaan sen seurauksena, ja virheilmoitukset `pool-set-network`, `pool-reboot` ja `pool-update-firmware` päättyvät rivillä `Warning:`, jossa mainitaan tallennuksen tiedostonimi (myös silloin, kun anturi ei tule takaisin).
- **Uudelleenkäynnistyksen jälkeen Chloros
yhdistää uudelleen vain itse anturin.** Se odottaa anturin aiemmasta sijainnista – ja `--auto`:n jälkeen myös sen nimellä `daq-e-<id>.local` – vastausta, jossa on anturin nimi, isäntänimen ja mallin sisältävää vastausta sen aiemmalta verkkosovittimelta (MAC) ja muodostaa yhteyden uudelleen osoitteeseen, josta vastaus saapui. Vastaus kyseisellä nimellä hyväksytään vain kyseisellä MAC-osoitteella, joten tietokoneelta, joka tavoittaa anturin reitittimen kautta, tai WSL2:n oletusarvoisessa NAT-verkossa, anturia, jonka DHCP on siirtänyt uuteen osoitteeseen, ei yhdistetä uudelleen: komento ilmoittaa tämän ja tulostaa komennon `pool-connect --eth-host <its address> …`, joka palauttaa anturin sen aiemmilla valotusrajoitus- ja valotusasetuksilla.
- **`pool-update-firmware` hylkää anturin, jonka laiteohjelmistoversiota se ei pysty lukemaan.** Anturi, jonka paluu salaman jälkeen kestää odotusaikaa kauemmin, ei ole vioittunut: kun se vastaa, se yhdistetään uudelleen uuden laiteohjelmiston asetuksilla, ja komento varoittaa, että se oli hidas.
- **Laiteohjelmiston päivitys vaatii saapuvan TCP-yhteyden.** Anturi muodostaa yhteyden takaisin tähän tietokoneeseen kuvan keräämiseksi. Chloros
kuuntelee anturiin reitittävää sovitinta (kaikilla sovittimilla vain silloin, kun se ei osaa tunnistaa oikeaa) ja palvelee vain anturin omaa osoitetta; kaikki muut yhteydet suljetaan palvelematta. Windows
issa asennusohjelman palomuurisääntö `Chloros Backend (TCP-In)` sallii yhteyden (katso [Windows
(.exe)](#windows-exe)); palomuuri, joka estää yhteyden, aiheuttaa päivityksen epäonnistumisen virhekoodilla `… device never connected back on TCP <port> …`.
- **WSL2:n alla käytä peilattua verkkoa** (`networkingMode=mirrored` ja `%UserProfile%\.wslconfig`). Oletusarvoisessa NAT-tilassa tämän tietokoneen osoite anturin verkossa on WSL:n oma, joten kaikki staattiset osoitteet hylätään (paitsi `--force`), suoratoisto ei saavuta WSL:ää eikä laiteohjelmiston päivitys voi valmistua, koska anturi muodostaa yhteyden takaisin tietokoneeseen kuvan vastaanottamiseksi.
- **Anturin tunnus ja isäntänimi eivät muutu.** Oli sen osoite mikä tahansa, DAQ-E on edelleen `daq-e-<id>` ja `daq-e-<id>.local`, joten `--eth-host daq-e-<id>.local` löytää sen edelleen.
- **Käyttöliittymässä on samat asetukset**: Valosensorit-välilehti, anturin asetukset (hammasratas), Verkko. Käyttöliittymä ylläpitää omaa yhteyttään anturiin, joten jos muutat anturia, johon käyttöliittymä on yhteydessä, käyttöliittymän kautta: CLI
:n kautta suoritettu uudelleenkäynnistys lopettaa käyttöliittymän istunnon kyseisen anturin kanssa.
- Vastaavat SDK
-tunnukset ovat `DAQSensorSession.device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` ja `update_firmware()`.

---

## `chloros-cli project`

Avaa, muodosta yhteys ja ohjaa tallennettua Chloros
-projektia (kansio, jossa on tiedostot `cameras.json` + `sensors.json` + `project.json`). Kaikki reititetään taustapalvelimen kautta, joten graafinen käyttöliittymä (GUI) ja komentoriviliittymä (CLI
) tuottavat identtisen laitteiston tilan.

### Alikomentojen viite

| Alikomento | Tarkoitus |
| --- | --- |
| `project open PATH [--relink FOLDER]` | Tulostaa projektin laitemaniestin (kamerat, matriisit, anturit), lukee sen tiedostoista ja tarkistaa sen lähdetiedostot (katso [Siirretty projekti](#a-moved-project---relink)). Se ei avaa projektia taustapalvelimessa: siellä avoinna oleva projekti sekä sen laitteet ja tallenteet pysyvät ennallaan. Poistuu ohjelmasta `1`, jos yhtään projektin lähdetiedostoa ei löydy. |
| `project devices PATH [--reconnect]` | Luettelee tai suorittaa laitteiden tunnistuksen uudelleen. |
| `project connect PATH [--cameras-only] [--sensors-only] [--packed {auto,on,off}]` | Yhdistää kaikki tallennetut kamerat / matriisit / anturit. Jokainen matriisi yhdistetään samalla tavalla kuin `lattice array-connect`: enintään 600 sekuntia, ja sen vaiheet tulostetaan stderr:iin (`[array <id>]    <s> s: <step>`; stdout on JSON
-raportti), ja ryhmä, jota ei ole kytketty tuossa ajassa, peruutetaan — sen kamerat palautetaan löydettyyn tilaan — ja raportoidaan kohdassa `errors`, eikä niitä jätetä koskaan kytketyiksi. `--packed` asettaa jokaisen ryhmän pakatun 12-bittisen siirron (oletusarvo `auto`; projekti ei tallenna ryhmäkohtaisia asetuksia; katso [Pakattu 12-bittinen siirto](#packed-12-bit-transport---packed)). Jokainen valosensori avataan taustaprosessin DAQ-poolissa sen tallennetussa osoitteessa tallennetulla integraatioajalla, kehyskeskiarvostuksella, automaattisella valotuksella ja ylärajalla: anturille, jolle ei ole valittu ylärajaa, annetaan arvo `sunshine_cosine`, tallennettu paljas anturi (`none`) pysyy paljaana, eikä DAQ-E-S-anturi ota koskaan ylärajaa. GUI:n DAQ-paneeli ei kuulu pooliin: sulje anturi ensin sieltä. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Yksittäinen tallennus nimetystä kamerasta tai matriisista (`--prefix` oletus, `capture`). `--orient` (ryhmät): ryhmä tallentaa raakadataa, joka säilyy alkuperäisessä muodossaan — tämä merkitsee, että tuotteet, joita `process` siitä tuottaa, on suunnattava, kuten `array-capture --orient`. Oletus: alkuperäinen. `project burst` / `stream` ja reseptin `capture` / `burst` / `stream` (`orient: true`) käyttävät samaa vaihtoehtoa. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | N-kuvan sarja nimetystä kamerasta tai ryhmästä (`-n/--count` oletusarvo 5; `-i/--interval` kuvien välinen aika sekunteina, oletusarvo 0). Ryhmäkuvasarjat poistavat toistuvat synkronoidut ryhmät (vanhentuneisuuden valvonta), jotta osittain kiertyvä ryhmä ei voi palauttaa N kopiota yhdestä kehyksestä; tulostaa tulokset jokaisen iteraation jälkeen. Kesken jäänyt tallennus keskeyttää sarjan ja lopettaa `1`:n ilmoittaen, kuinka monta kuvaa ehdittiin tallentaa ennen keskeytyksen. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--orient]` | Tallennus levylle silmukassa: `--fps F` tallentaa sekunnin (oletusarvo 3), yhden kehyksen kerrallaan kamerasta, yhden synkronoidun kehysryhmän taulukosta (samalla vanhentumisvalvonnalla kuin `project burst`), kunnes `-n N` tallentaa (oletusarvo `0` = kunnes painetaan Ctrl+C, jolloin tulostetaan tallennettujen kuvien määrä ja ohjelma lopetetaan `0`, tai `1`, jos yhtään kuvaa ei tallennettu tai tiedoston tallennus epäonnistui; Ctrl+C ennen kuin `-n N` tallentaa lopettaa `130`). Kesken jäänyt tallennus keskeyttää datavirran ja lopettaa ohjelman `1`, ilmoittaen kuinka monta kehystä tallennettiin ennen keskeytyksen tapahtumista. `--poll-interval` on vanhentunut ja ohitetaan: se hyväksytään edelleen, jotta vanhemmat skriptit toimivat, ja siitä ilmoitetaan stderr:ssä. |
| `project sensor read PATH NAME [--json]` | Uusin spektrikehys. |
| `project sensor log PATH NAME [--seconds SEC] [-o DIR] [--device-name NAME]` | Tallentaa .daq-tiedoston `--seconds`:lle (oletusarvo 10). Tallenne, joka ei sisällä lukemia, raportoidaan stderr:ään ja ohjelma lopetetaan `1`. |
| `project run PATH RECIPE.yaml` | Suorita YAML-/JSON
-kaappausresepti. `--dry-run` tarkistaa kelpoisuuden suorittamatta. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Laske taulukon kohdistus — katso [alla oleva lipputaulukko](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Tulosta nykyinen kohdistusprofiili. |
| `project align clear PATH NAME` | Poista välimuistissa oleva profiili. |
| `project align tweak PATH NAME --serial SN [--dx N] [--dy N] [--rotation-deg N] [--scale N]` | Siirrä yhden orjan muunnosta (oletusarvot `0`, `0`, `0`, `1.0`: ei muutosta). |
| `project align export PATH NAME --to FILE` | Tallenna profiili JSON
-muotoon. |
| `project align import PATH NAME --from FILE [--no-validate]` | Lataa tallennettu profiili. |

#### `project align calibrate`-asetukset

| Lippu | Oletus | Kuvaus |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Kohdistusmenetelmä. **Nämä kirjoitusmuodot eroavat `lattice align-calibrate`**:stä, joka hyväksyy lyhenteet `orb` / `akaze` / `phase`; näitä kahta komentoa ei voi käyttää keskenään vaihdellen tässä lippussa. |
| `--model {translation, rigid, affine, homography}` | `affine` | Muunna malli sopivaksi. |
| `--frames N` | `1` | Synkronoi kehyskuvat keskiarvoon. |
| `--reference SN` | pääkameraan | Viitekameran sarjanumero; kaikki muut jäsenet vääristetään sen mukaan. |
| `--max-features N` | `5000` | ORB-piirteiden lukumäärän yläraja. |
| `--ratio-threshold F` | `0.75` | Lowe&#x27;suhdetesti. |
| `--ransac-threshold-px F` | `3.0` | RANSAC-sisäpisteiden kynnysarvo. |
| `--min-matches N` | `15` | **Laatukynnys** — hylkää ratkaisu, jos sisäpisteiden vastaavuuksien määrä on alle tämän arvon. |
| `--max-reproj-err-px F` | `4.0` | **Laatukynnys** — hylkää ratkaisu, jos RMS-uudelleenprojektio-virhe ylittää tämän arvon. |
| `--checkerboard RxC` | — | `--method checkerboard`:n levyn geometria, esim. `9x6`. |
| `--name PROFILE` | tyhjä | Tallennettuun JSON
-tiedostoon upotettu profiilin nimi. **Ei taulukon nimeä** — se on sijaintitieto `NAME`. |

Nämä kaksi laatuporttia ovat syy siihen, että kalibrointi voi onnistua ratkaisussa mutta silti
kieltäytyä tallentamasta: profiili, joka epäonnistuu jommassakummassa, rekisteröisi hiljaisesti väärin jokaisen
myöhemmän kaappauksen, joten se hylätään sen sijaan, että se tallennettaisiin.

### Siirretty projekti (`--relink`)

Projektin `project.json` nimeää jokaisen lähdetiedoston — jokaisen kuvan ja valosensorin
tallennuksen — sen täydellisen polun mukaan. Jos siirrät projektikansion tai koko projektikansion
toiselle asemalle, nämä polut viittaavat edelleen vanhaan sijaintiin.

Projektin avaaminen korjaa sen, minkä se pystyy korjaamaan itse: tiedostoa, joka oli aiemmin projektin
kansiossa, etsitään kansiosta, jossa projekti nyt sijaitsee, ja jokainen siellä löytyvä polku
ohjataan uudelleen ja tallennetaan (`project.json` varmuuskopioidaan ensin nimellä
`project.json.before-relink-<date>-<time>`). Tämä tapahtuu jokaisen avaamisen yhteydessä – sovelluksen,
`chloros-cli project open`, `chloros-cli process` olemassa olevaan projektiin, SDK
:n
ja `chloros-mcp`:n. Polku osoitetaan uudelleen vain tiedostoon, joka on kyseisessä paikassa.

Tiedostoja, jotka olivat projektikansion ulkopuolella – esimerkiksi Hub-istunnon tai ”Lisää kansio” -tuonnin yhteydessä –
ei löydy tällä tavalla. Nimeä kansio, jossa ne nyt ovat:

```bash
chloros-cli project open "N:/Chloros Projects/Field_A" --relink "E:/backup/Hub Projects"
```

`--relink` ottaa kansion, johon tiedostot siirrettiin, tai minkä tahansa sen yläpuolella olevan kansion: jokaisen puuttuvan
tiedoston kansiota etsitään sen alapuolelta tiedoston lopussa olevan kansionimen perusteella
(`Hub Projects/<session>/raw/<serial>`, sitten `<session>/raw/<serial>`, ...), ja kunkin
kansion tiedostot haetaan siitä paikasta, josta löytyy eniten tiedostoja. `project open` tulostaa
mitä uudelleenosoitettiin, kuinka monta lähdetiedostoa löytyi ja missä kansioissa loput olivat
. Joitakin puuttuu edelleen: varoitus, poistu ohjelmasta `0` – projekti avautuu loput tiedostot mukana. Ei
löydy: virhe kyseisten kansioiden nimeämisessä ja `--relink`, poistu ohjelmasta `1`; taustapalvelin
kieltäytyy avaamasta tällaista projektia `process`:n tai `project devices --reconnect`:n osalta (HTTP
409 `PROJECT_SOURCES_MISSING`).

### Esimerkkejä

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Its hub captures moved to another drive: point it at them
chloros-cli project open "/home/user/Chloros Projects/Field_A" --relink "/media/backup/Hub Projects"

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

`project run RECIPE.yaml` hyväksyy YAML- tai JSON-tiedoston, jossa kuvataan toimien sarja:

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
        auto_exposure: "Off"
        gain_db: 6.0
        auto_gain: "Off"
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

Tuetut toiminnot: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. `apply` ottaa avaimet `cameras.json`, `settings` — `exposure_us`, `auto_exposure`, `gain_db`, `auto_gain`, `target_brightness`, … (vanhemmat nimet `exposure_time`, `exposure_auto`, `gain`, `gain_auto` hyväksytään aliaksina). Tällä tavalla asetettu valotus, vahvistus tai kohdekirkkaus säilyy käyttäjän valintana: Chlorosin automaattinen valotuksen säätö (mukaan lukien DAQ-mitattu valotus) ei muuta sitä. Manuaalinen valotus jättää vahvistuksen ennalleen: automaattitilassa kamera säätää vahvistusta (koko alueellaan) kohti kohdekirkkautta, kunnes asetat `gain_db` tai `auto_gain: "Off"`. Myöhempi `auto_exposure: "Continuous"`-komento palauttaa valotuksen automaattitilaan, ja `target_brightness_auto: true`-komento palauttaa lukitun kohteen Chlorosin automaattiseen kohteenvalintaan. `burst`-toiminto vaatii `name` (pakollinen), `count` (oletusarvo 5), `interval` (sekuntia, oletusarvo 0), `output`, `format`, ja `settings` (sama kamerakohtainen muoto kuin `apply`); taulukkosarjat käyttävät samaa juuri synkronoitua ryhmän valvontaa kuin `project burst`.

A `capture`, `burst` tai `stream`-vaiheen `output`-oletusarvo on `output`. Suhteellinen `output` — oletusarvoisesti mukana — määritetään sen kansion perusteella, josta `chloros-cli project run` suoritetaan, ei reseptitiedoston kansiosta tai projektikansiosta (versiossa 1.2.0 se sijoitettiin taustapalvelimen asennuskansioon).

Suorita se:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

**Jokainen suoritus tarkistetaan ensin projektin suhteen.**Jokaisen toiminnon avaimena on oltava tarkalleen yksi yllä mainituista verbeistä — `- capture: {name: FrontLeft}`, ei `- action: capture` tai `- type: capture` — jos avaimena on `name`, projektissa on `cameras.json` / `sensors.json`, kamera tai matriisi `apply` / `capture` / `burst` / `stream` sekä valosensorin mallille `sensor`; `stream` tarvitsee, että `count` &gt; 0, ja `wait` tarvitsee keston. Resepti, jossa on tällainen virhe, ei suorita**mitään** toimintoa (`[recipe] error: …`, `[recipe] not run: …`, poistuminen `1`), edes `stop_on_error: false`:n ollessa käytössä. Avaimet, joita vaihe ei käytä, ovat `[recipe] warning:`, samoin kuin jokainen reseptin ohjaama taulukko tai valosensori, joka ei ole tällä hetkellä kytkettynä. Resepti ei koskaan liitä laitteita: suorita ensin `chloros-cli project connect PATH`. `--dry-run` tulostaa reseptin, sitten samat virheet ja varoitukset sekä `[recipe] valid: …` / `NOT valid: …`, ei suorita mitään ja lopettaa `1`, kun reseptissä on virhe.

---

## Ympäristömuuttujat

| Muuttuja | Vaikutus |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Ohittaa taustapalvelimen URL-osoitteen (oletus `http://127.0.0.1:5000`) — **huomioidaan vain komentojen `lattice`, `project` ja `daq pool-*` perheissä** (`lattice network-analysis` käyttää sen sijaan `--backend-url`-komentoa). Ydinkomennot (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) asettavat `http://127.0.0.1:<port>`:n ja ohittavat tämän muuttujan (IPv4-literaali kiertää Windowsin `localhost`→`::1` ~2 s/pyyntö -viiveen), joten ne kohdistuvat aina paikalliseen koneeseen. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | Määritetään taustapalvelimen ympäristössä. `1` lieventää ylimerkinnän hylkäystä — taulukkomuotoinen yhteys, jossa on `pin_resolution`, tai [väliaikainen monikamerakomento](#multi-camera-transient-sessions) (`error_code` `ARRAY_OVERSUBSCRIBED`) — voimakkaaksi varoitukseksi, jolloin GVSP-pakettihäviöt hyväksytään. Vain testausta varten — katso [Taulukon fps- ja burst-malli](#array-fps--burst-model). |
| `CHLOROS_ARRAY_PACKED12` | Määritetään taustapalvelimen ympäristössä: `on` tai `off` asettaa oletukseksi [pakatun 12-bittisen siirron](#packed-12-bit-transport---packed) jokaiselle taulukolle (oletusarvo `auto`). Nimenomainen `--packed` on edelleen etusijalla. |
| `CHLOROS_DISABLE_UPDATE_CHECK` | `1` (tai `true` / `yes`) poistaa käytöstä Linux-CLI:n [automaattisen päivitystarkistuksen](#chloros-cli-update) käynnistyksen yhteydessä. |
| `CHLOROS_CLI_MODE` | Asetetaan itse CLI:n toimesta; käskee taustapalvelua ottamaan rinnakkaiskäsittelyn käyttöön. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` ohittaa GVSP-varatestauksen (vain ICMP-tulokset). **Tämä poistaa jumbo-paketit käytöstä, se ei pelkästään vaimenna lokia** — kamera vastaa DF-ping-kyselyihin vain 1500:aan asti jokaisella reitillä, joten tämä testi on ainoa keino havaita jumbo-paketit. Säästää noin 1 s kameraa kohti yhteyttä kohti; maksaa noin 1,45-kertaisen kaistanleveyden, jos verkko *olisi voinut* siirtää jumbo-paketteja. SDK antaa varoituksen, kun asetus määritetään. |
| `CHLOROS_GVSP_SESSION_REUSE` | Määritetään taustapalvelimen ympäristössä: `0` mittaa paketin koon jokaisella yhteyden muodostuksella sen sijaan, että se tehtäisiin kerran kameraa kohti taustapalvelimen istuntoa kohti. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Kiinnitä GVSP-paketin koko arvoon N tavua; ohittaa testauksen kokonaan. Suosittele komentoa kohti tehtävää asetusta (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) pysyvän asetuksen sijaan: kiinteä koko ei enää sopeudu edessä olevaan verkkoon, ja arvon 9000 kiinnittäminen reitille, joka ei tue jumbo-paketteja, aiheuttaa **jokaisen** tallennuksen aikakatkaisun `SC_ERR_TIMEOUT -1011`:n kanssa. |
| `CHLOROS_TRANSIENT_AE_SETTLE_S` | Määritetään taustapalvelimen ympäristössä: enimmäismäärä sekunteja, jonka [väliaikainen monikamerakomento](#multi-camera-transient-sessions) tai kameran `lattice capture`, jota mikään ei pidä yllä, käyttää kunkin kameran valotuksen asettamiseen edellisestä kohtauksesta ennen kuin se tallentaa kehyksen (oletusarvo `5`). `0` poistaa tämän käytöstä: kunkin kameran oma automaattinen valotus toimii tällöin ensimmäisestä kehyksestä lähtien, alkaen sen valotusrajasta. |
| `CHLOROS_LOG_DIR` | Määritetään taustapalvelimen ympäristöasetuksissa: kansio, johon sen yksityiskohtainen loki tallennetaan. Oletusarvo `%LOCALAPPDATA%\Chloros\logs` Windowsissa, `~/.cache/chloros/logs` Linuxissa (`$XDG_CACHE_HOME/chloros/logs`, kun se on määritetty). .deb-tiedoston juuripalvelu asettaa `/var/log/chloros`-muuttujan, jota `adm`-ryhmä voi lukea — katso [Taustapalvelun ajaminen palveluna](#running-the-backend-as-a-service-headless). |
| `TMPDIR` (Linux) | Väliaikaisten tiedostojen sijainti. CLI käyttää automaattisesti `/mnt/ssd/tmp`:ää, jos se on määritetty. |

---

## Lopetuskoodit

| Koodi | Merkitys |
| --- | --- |
| `0` | Onnistui. |
| `1` | Yleinen virhe (useimmat alikomentojen virheet). Tallennuskomento lopettaa toimintansa myös koodilla `1`, jos jonkin tiedoston kirjoittaminen epäonnistui, vaikka muut tiedostot tallennettiin, sekä koodilla `array-burst` / `array-record` lopettaa ohjelman `1`, kun tallennin ilmoittaa virheestä — katso [Tiedostot, joiden kirjoittaminen epäonnistui](#files-that-fail-to-write). `process` lopettaa toimintansa myös, kun ajon vientiä ei voitu tarkistaa tai kun se menetti kuvan, vaikka muut tuotteet tallennettiin. |
| `2` | Argumenttivirhe (tuntematon tai puuttuva lippu; myös `time-sync set-priority` ilman kumpaakaan prioriteettia). |
| `130` | Keskeytetty Ctrl+C:llä: mikä tahansa komento, jonka se keskeyttää — `lattice`, `project`, `daq` ja `process` (`process` sen jälkeen, kun taustaprosessi on lopettanut suorituksen, katso [`process` Huomautukset](#notes)). Komento, jonka normaali lopetus on Ctrl+C — `lattice array-capture --continuous` / `--interval`, `lattice viewer`, `lattice continuous`, `multi-stream`, `align-stream`, `array-record`, `array-burst` ja `project stream` sekä `-n 0` — lopettaa sen mukaan, mitä se teki: `0`, kun kaikki oli kirjoitettu, `1`, kun jokin epäonnistui. |
| `143` / `129` | Lopetettu SIGTERM- tai SIGHUP-signaalilla (128 + signaali) sen jälkeen, kun komennon siivoussuoritus oli suoritettu: käynnissä ollut työ pysäytettiin ja sen kamerat palautettiin, ja taustaprosessointi lopetettiin. `lattice continuous` tai muu tallennus, jonka normaali pysäytyskomento on Ctrl+C, poistuu myös `143`:stä, ei `0`:stä. |

`lattice network-analysis` lopettaa `1`:n, kun tuloksena on `needs_force_slip` tai `error`, ja `selftest` lopettaa `1`:n, elleivät kaikki seitsemän tarkistusta läpäise.

---

## Vianmääritysvinkkejä

- **&quot;Kirjautuminen vaaditaan&quot;** → Suorita `chloros-cli login EMAIL PASSWORD` kerran tällä koneella.
- **&quot;Projektikansio on liian syvällä tällä tietokoneella&quot;** → Windowsin pitkät polut on poistettu käytöstä tällä koneella (Windowsin oletusasetus), joten mikään polku ei saa ylittää 260 merkkiä, ja tuotteet sijaitsevat enintään noin 130 merkkiä projektikansion alapuolella. Mitään ei käsitelty. Viesti ilmoittaa pisimmän sopivan projektikansion nimen: käytä lyhyempää (`-o` tai `chloros-cli set-project-folder`) tai ota Windowsin pitkät polut käyttöön ja käynnistä Chloros uudelleen. Windowsin asennusohjelma tarjoaa tämän vaihtoehdon nimellä ”Ota Windowsin pitkät polut käyttöön”, joka on valittuna oletuksena; muussa tapauksessa käytä ryhmäkäytäntöä: *Tietokoneen kokoonpano &gt; Hallintamallit &gt; Järjestelmä &gt; Tiedostojärjestelmä &gt; Ota Win32-pitkät polut käyttöön*.
- **&quot;backend unreachable&quot;** → Käynnistä Chloros-työpöytäsovellus (Windows) tai käynnistä taustapalvelu itse: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` Windowsissa (asennuskansion `backend`-kansio; se ei ole kansiossa `PATH`), `chloros-backend` Linuxissa. Tai tarkista `CHLOROS_BACKEND_URL`, jos kyseessä on etäkäyttö.
- **`lattice`-komennot epäonnistuvat virheilmoituksella ”LATTICE-kameran ajureita ei löydy”** → Taustapalvelin ei pystynyt lataamaan sen mukana asennettua LATTICE-kameran ajoympäristöä. Jokainen `lattice`-komento suoritetaan taustapalvelimella; CLI-käyttöliittymä itsessään ei sisällä kameran ajureita. Asenna Chloros uudelleen: Windows-asennusohjelma ja .deb-paketti asentavat ajoympäristön taustapalvelimen viereen.
- **Array connect / Array Settings näyttää viestin &quot;FRAMES WILL DROP&quot; tai &quot;Reduce ROI to enable&quot;** → Isäntäkoneen verkkokortin vastaanottorengas on liian pieni (palautuu yleensä arvoon 32 verkkokortin ohjaimen päivityksen jälkeen). Katso [Isäntäkortin asetukset ja säätö](#host-nic-setup--tuning-lattice-arrays) — aseta `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Kone jumittuu uudelleenkäynnistyksen tai sammutuksen yhteydessä, minkä jälkeen WMI `Invalid class` / verkkokortti ei aktivoidu / USB-asemat puuttuvat** → Vanhentunut USB 10 GbE -sovittimen ohjain aiheuttaa virheen `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Päivitä sovittimen ajuri — katso [Isäntäverkkokortin asetukset ja säätö](#host-nic-setup--tuning-lattice-arrays).
- **Linux: tallenteet kuuluvat root-käyttäjälle, joten et voi poistaa niitä, eikä backendin loki ole käytettävissäsi `~/.cache`** → Backend toimii root-käyttäjänä `chloros-backend.service`:n kautta. Vaihda `chloros-backend@<user>.service`-kansiin, jossa se toimii sinun käyttäjätunnuksellasi — katso [Backendin ajaminen palveluna](#running-the-backend-as-a-service-headless). Siihen asti root-palvelun loki sijaitsee tiedostossa `/var/log/chloros` (jonka `adm`-ryhmä voi lukea).
- **Linux: taustayksikkö on `failed` ja siinä näkyy viesti ”Another instance is already running” (lopetustila 98)** → Toinen taustayksikkö on jo varannut portin: vain yksi voi olla käynnissä kerrallaan. Jos kaksi taustayksikköä on käytössä, säilytä toinen: `sudo systemctl disable --now <unit>` sille, jota et käytä — katso [Taustayksikön ajaminen palveluna](#running-the-backend-as-a-service-headless). Jos vain tämä yksikkö on käytössä, porttia pitää hallussaan systemd:n ulkopuolella käynnistetty taustapalvelu (komennolla `chloros-backend`, `chloros-cli` tai SDK-skriptillä): pysäytä kyseinen taustapalvelu ja suorita sitten komento `sudo systemctl start <unit>`.
- **Jetson-swap-varoitus** → Lisää tiedostopohjainen swap; CLI tulostaa tarkat `fallocate`- ja `swapon`-komennot.
- **&quot;backend not reachable&quot; -virhe `lattice`-, `project`- tai `daq`-komennosta** → mikään taustaprosessi ei ole käynnissä: nämä komennot eivät koskaan käynnistä taustaprosessia. Suorita komento `chloros-cli backend status` ja katso ohjeet kohdasta [Backendin käynnistäminen](#start-the-backend).
- **&quot;`daq net ...` ei ole käytettävissä Chlorosissa&quot;** (Linux ja Jetson) → Osa DAQ-E:n omista asetuksista on varattu MAPIR-palvelulle. Verkkoasetukset, uudelleenkäynnistykset ja laiteohjelmistopäivitykset hoidetaan komentoilla `pool-set-network`, `pool-reboot` ja `pool-update-firmware`; ota yhteyttä MAPIR-tukeen muissa tapauksissa.
- **&quot;DAQ:n suorat laitteistokomennot edellyttävät täydellistä daq-pakettia&quot;** (Windows) → Käytä `pool-*`:ää, joka ohjaa samaa anturia taustapalvelun kautta, tai `chloros_sdk.connect_daq_sensor()`:ää Pythonista.

---

## Katso myös

- [Python-SDK-viite](sdk-reference.md) — jokaisen CLI-komennon ohjelmointivastaava.
- [DAQ-valosensorit](https://mapir.gitbook.io/chloros/daq-light-sensors) Chloros-käsikirjassa — sensorit, niiden ylärajat ja kalibroitu alue, tallennus, DAQ-E-verkko.
- [LATTICE-kamerat](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) Chloros-käsikirjassa — liittäminen, kameran asetukset, kuvausmoodit, kameraryhmät.
- Verkkodokumentaatio: `https://mapir.gitbook.io/chloros/cli`
