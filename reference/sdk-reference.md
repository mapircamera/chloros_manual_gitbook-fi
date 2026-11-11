# Chloros-Python-SDK-viiteopas

**Versio:**

1.3.0**Luotu:**

29.7.2026 klo 19.19 ·**Päivitetty:**

6.11.2026**Paketti:** `chloros-sdk` (PyPI)**Kohderyhmä:** Optimoitu suurten kielimallien (LLM) käyttöön; ihmisen luettavissa.**Laajuus:** Kaikki `import chloros_sdk`:n tarjoamat julkiset luokat, funktiot ja apufunktiot, sekä kopioitavat esimerkit, jotka kattavat kuvankäsittelyn, yhden kameran ohjauksen, synkronoidut taulukot, DAQ-antureita ja projektiautomaatiota, sekä `chloros-mcp`-palvelin tekoälyavustajille.

Jos haluat tutustua vain tärkeimpiin kohtiin, siirry kohtaan:
- [Asennus ja pikaopas](#asennus)
- [Smart-Connect LATTICE-kameroille](#smart-connect-for-lattice-cameras)
- [DAQ-anturisessiot](#daq-sensor-sessions)
- [Kamera-apuohjelmat](#camera-utilities)
- [Transient-matriisit, kohdistusprofiilit ja indeksit](#transient-arrays-alignment-profiles--indices)
- [Projektikansio ja reseptit](#project-folder--recipes)
- [Projektin automatisointi](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Reaaliaikaiset kehykset omassa prosessiketjussasi](#live-frames-in-your-own-pipeline)
- [MCP-palvelin (AI-avustajat)](#mcp-server-ai-assistants)

> **Muutokset versiossa 1.3.0** (mitä olemassa oleva 1.2.0-skripti huomaa).
>
> **Muutokset, joita 1.2.0-skriptin on mahdollisesti käsiteltävä:**
>
> - **Vapautus, jota taustapalvelu ei tällä hetkellä vahvista, aiheuttaa virheen.** `CameraSession.disconnect()`, `ArraySession.disconnect()` / `force_disconnect()` istunnon avaamassa taulukossa, `DAQSensorSession.disconnect()` / `force_disconnect()` sen avaamassa anturissa sekä `ChlorosProject.disconnect_all()` aiheuttavat virheen `ChlorosConnectError`, kun taustapalvelin hylkää vapautuksen tai ei vahvista sitä ajoissa, koska laite saattaa olla edelleen varattuna. Versiossa 1.2.0 ne palautettiin hiljaisesti. `with`-lohkon poistuminen ei vieläkään aiheuta virhettä.
> - **Lainattua laitetta ei koskaan pureta.** `with attach_array(...)` ja nimenomainen `disconnect()` sen suhteen jättävät taulukon käynnissä (versiossa 1.2.0 lohkon poistaminen purki sen, vaikka GUI omistaisi sen); kutsu `force_disconnect()`, kun haluat lopettaa sen. Samoin `connect_daq_sensor()`, joka löysi anturin jo poolista (mukaan lukien nollalippuinen älykäs tunnistus), jättää sen auki `disconnect()`:n aikana ja kun se poistuu `with`-lohkostaan.
> - **PNG- ja JPEG-tiedostot hyväksyvät vain `display`- ja indeksituotteet.** `capture()` yhdessä `format="png"`:n / `"jpg"`:n (`ext=".png"` / `".jpg"`) sekä `radiance`- tai `reflectance`-taso, tai `processing="all"`, nostaa `ValueError`-tason ennen mitä tahansa pyyntöä; 12-bittinen `raw` / `debayered` hylätään taustapalvelimessa. Katso [PNG / JPEG](#png--jpeg-take-the-display-and-index-products-only).
> - **`connect_daq_sensor(sensor_id=...)` avaa kyseisen anturin tai laukaisee** `ChlorosConnectError`-virheen, jossa luetellaan löydetyt tiedot; se ei koskaan siirry toiseen anturiin. Katso [Kiinnitetty siirto](#pinned-transport).
> - **`frame_stream()` lähettää virheilmoituksen sen sijaan, että menisi hiljaiseksi:** `FrameFetchError` toistaa hylkäyksen jälkeen seuraavalla tikillä (heijastuslukema, johon ei sisälly lainkaan DAQ-tietoja), kun 60 sekuntia on kulunut ilman kehystä ja kun 60 sekuntia on kulunut ilman uutta laukaisua. Katso [Yksi laukaisu lukemalta](#one-trigger-per-read--group_id).
> - **Lukemisen aikakatkaisu tai katkennut yhteys aiheuttaa virheen `ChlorosConnectError`** jokaisesta smart-connect-kutsusta sekä virheen `analyze_array_network()`, kun versio 1.2.0 aiheutti taustalla olevan aikakatkaisun tai yhteysvirheen.
> - **`connect_array()` odottaa oletusarvoisesti 600 sekuntia (`timeout`, 180 sekuntia versiossa 1.2.0), ja yhteys, jonka muodostamisesta se luopuu, peruutetaan.** Kutsu lähettää `timeout`-viestin taustapalvelimelle, joka palauttaa kaikki kamerat alkuperäiseen tilaansa eikä muodosta yhteyttä, jos yhteyden muodostaminen ei onnistu ajoissa; kutsu aiheuttaa virheen `ChlorosConnectError` (HTTP-virhe 504 `CLIENT_DEADLINE_PASSED`). Versiossa 1.2.0 kamerien virran kytkemisen jälkeen heti tehty yhteydenmuodostus saattoi kestää kauemmin kuin kutsu ja muodostaa yhteyden kameraryhmään sen jälkeen, kun virhe oli jo syntynyt. `connect_all()` odottaa jokaista tallennettua kameraryhmää samalla tavalla (120 s versiossa 1.2.0). Katso [`connect_array()`-allekirjoitus](#connect_array-signature).
> - **`connect_array()` säilyttää koko anturin.** Ilman `width` / `height` / `binning`-asetuksia jokainen kamera on asetettu käyttämään koko anturia, ja kuvataajuus laskee isäntäkoneen ja verkon tarjoamaan tasoon; versiossa 1.2.0 pienensi tai yhdisteli kehyksiä kuvataajuuden ylläpitämiseksi. Katso [Synkronoitu matriisi](#synchronized-array--arraysession-smart-prep).
> - **M3M (mono)-kameran Bayer-pikselimuoto on sen syvyys.** M3M tarjoaa vain monomuotoja, ja esiasetukset nimeävät ne Bayer-muodoiksi: versiossa 1.2.0 kamera hylkäsi formaatin ja säilytti oman formaattinsa, kun taas `settings.pixel_format` nimitti sen Bayer-formaatiksi. `LatticeCamera(preset=...)`, `apply_preset()`, `set_format()`, `CameraSettings(pixel_format=...)`, `connect_camera(preset=...)` jajaetun muodon avulla määritetään nyt saman syvyyden Mono-muoto M3M:ssä: `"BayerRG8"` ja `high_speed` tuottavat `Mono8`:n; `"BayerRG12"`, `high_quality` ja `triggered` tuottavat tulokseksi `Mono12`; `cam.settings.pixel_format` nimeää sen. Sama pätee myös päinvastaiseen suuntaan: M3C-tiedostoa ei koskaan tallenneta Mono-muotoon, joten `"Mono12"` – mistä tahansa näistä, `connect_array(pixel_format=...)`, `set_settings(pixel_format=...)` tai taulukon tallennettu muoto – tuottaa M3C:llä tuloksen `BayerRG12` riippumatta siitä, mikä kamera on taulukon pääkamera. Välittämäsi `CameraSettings` säilyttää nimensä, joten sama objekti kysyy edelleen jokaiselta kameralta kyseistä syvyyttä omassa perheessään. `connect_camera(preset=...)` säilyttää esiasetuksen formaatin (versiossa 1.2.0 kamera asetetaan 12-bittiseksi esiasetuksen soveltamisen jälkeen), eikä esiasetuksella avattu kamera enää kirjoita ottamaansa tietoa `PRESETS`:ään. Mikä tahansa muu muoto, jota kamera kieltäytyy (jota sen `PixelFormat`-luettelosta puuttuu, kuten `RGB8` M3M-laitteessa tai `BayerRG16` laiteohjelmistossa, jossa sitä ei ole), ei myöskään nimetä omaksi: kun `LatticeCamera`-yhteyden jälkeen `apply_settings()` tai `set_format()`, `cam.settings.pixel_format` ja `packed_transport` nimeävät kameran lähettämän formaatin, ja yksi tulostettu rivi nimeää formaatin, jota se hylkäsi.
> - **`analyze_array_network()` ilmaisee, mitä yhteys koskee:** `pixel_format=None` (aiemmin `"BayerRG8"`) tarkoittaa yhteyden asettamia formaatteja (Mono12 / BayerRG12), ja `packed="auto"` ilmaisee pakatun siirron valinnan, joten sen ilmoittama kuvataajuus (fps) voi poiketa versiosta 1.2.0&#x27;. Jos välität `pixel_format`-parametrin, se tuottaa M3M-tiedoston kyseisellä syvyydellä. Katso [Verkkoanalyysin aputyökalu](#network-analysis-helper).
> - **`CameraSession.capture()` palauttaa `CaptureResult`:n** (edelleen luettelo kehyssanakirjoista), jossa on `.skipped`, `.failed`, `.daq_files` ja `.daq_matches`. `ArraySession.capture()`-tulos sisältää myös `.alignment`:n (sovellettiinko kohdistuksen vääristymää ja miksi ei) sekä `.oriented`:n.
> - **Taulukkomuotoiset tallenteet kohdistetaan oletusarvoisesti, kuten sovelluksen tallenteetkin.** `ArraySession.capture()`, `continuous()`, `capture_repeated()` ja `ArrayHandle.capture()` / `capture_smart()` vääristävät jokaisen jäsenen `debayered`-, `display`, `radiance`, `reflectance` ja hakemistotiedostot pääkameran kehykseen ja rajaa ne jäsenten päällekkäisyyden (joka on pienempi kuin anturi kameroiden siirtymien verran, `XMP-mapir:AlignmentJSON` `"applied": true`), joten ne ovat samat tiedostot kuin Chloros-sovelluksen ottamat kuvat samasta kohtauksesta; `raw` säilyy alkuperäisessä muodossaan. Aiemmin tallennus ilman `aligned=True` tallensi jokaisen tiedoston alkuperäisessä koossa, ilman vääristymien korjausta. `aligned=False` säilyttää tämän. `chloros-mcp`:n`capture_array` ja `start_capture_run` käyttävät samaa oletusasetusta. Katso [`ArraySession`-menetelmät](#arraysession-methods).
> - **`connect_all()` nimeää taustapalvelimen, joka hylkää SDK:n.** Käynnissä oleva taustapalvelin, josta on kirjauduttu ulos tai joka on Copper-tasoa alemmalla tilauspaketilla, raportoidaan `errors`:n alla omalla syyllään (HTTP 401 / 403), ei muodossa ”yhtään Chloros-taustapalvelinta ei ole käynnissä”, ja jokainen `connect_all()`-kutsu tarkistaa taustapalvelimen uudelleen, joten sen kutsuminen kirjautumisen jälkeen toimii samalla projekti-objektilla.
> - **Suhteellinen `output_dir` toimii paikallisen taustapalvelimen kanssa.** Suhteelliset polut muutetaan absoluuttisiksi skriptin työkansioon nähden ennen niiden lähettämistä. Versiossa 1.2.0 taustapalvelin tulkitsi ne oman asennuskansionsa suhteen, mikä johti virheeseen ”Pääsy evätty”.
> - **`numpy` on nyt SDK:n vaatimus** (live-frame-kutsut palauttavat numpy-taulukoita).
> - **Vienti-tason nimi, joka ei ole yksi, hylätään.** `processing=` ja `levels=` hyväksyvät `raw`, `debayered`, `radiance`, `reflectance` tai `display` (tai aliaksen, kuten `preview`; `processing=` sekä `"all"`). Mikä tahansa muu nimi hylätään ennen kameran koskettamista (HTTP 400 `UNKNOWN_LEVEL`: `capture()` aiheuttaa virheen `ChlorosConnectError`, `grab()` / `frame_stream()` `FrameFetchError`); versio 1.2.0 tallensi tai palautti sille `raw`:n.
> - **Texture Aware -ominaisuutta ei sallita LATTICE-kaappauksissa.** Projektissa, joka sisältää LATTICE-kaappauksia, `configure(debayer="Texture Aware (Slow, Highest Quality)")` aiheuttaa virheen `ChlorosConfigurationError`, samoin kuin `process()` sen jälkeen, kun `configure()` on pyytänyt sitä ennen tuontia. Projektin aiemmin tallentama Texture Aware -arvo asetetaan Standard-tilaan, jolloin syntyy virhe `UserWarning`. LATTICE-kaappaukset saavat aina standardin mukaisen demosaicin, joten versio 1.2.0 hyväksyi asetuksen ja ohitti sen; Chloros-sovellus ei myöskään tarjoa sitä niille. Katso [Tuetut arvot](#supported-values).
> - **Kamerat palautetaan sellaisiksi kuin ne löydettiin — ellei `keep_settings=True`.** `ArraySession.disconnect()` / `force_disconnect()`, jokaisen väliaikaisen toiminnon lopussa (`capture_cameras_transient()`, `start_multi_camera_stream()`, `test_array_sync()`, `calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index()`) sekä `start_camera_continuous()`:n pysäyttäminen palauttavat jokaisen kameran siihen tilaan, jossa se oli löydettäessä: sen laukaisun ja synkronointilinjan johdotuksen, sen PTP:n sekä kaikki istunnon aikana muutetut kameran asetukset (valotus ja vahvistus, AE-kohde, rajat ja vaimennus, valkotasapaino, mustataso, pikselimuoto, pikselien yhdistely ja ROI, peilaus, kuvataajuus). `keep_settings=True` puolestaan tallentaa kunkin kameran asetukset kameran omaan muistiin, jolloin kamera käynnistyy niillä; `clear_settings=True` nollaa kyseisen muistin tehdasasetuksiin. Kummassakin tapauksessa kamerat jäävät toimimaan vapaasti. Projekti asettaa kameransa omien tallennettujen asetustensa mukaan, kun se yhdistää ne. Katso [Kameran muisti](#camera-memory--keep_settings--clear_settings).
> - **Jokainen tila tallentaa projektin samalla tavalla.** Kaikki muutokset projektiin liitettyyn kameraan, matriisiin tai valosensoriin — olipa ne tehty SDK:n, sovelluksen, CLI:n tai MCP-palvelimen kautta — tallennetaan kyseisen projektin `cameras.json` / `sensors.json` -kansioon, ja projektin avaaminen missä tahansa tilassa asettaa laitteet sen mukaisiksi; `ChlorosProject` nimeää oman kansionsa taustajärjestelmään. Aiemmin vain sovellus tallensi projektin. Katso [Yksi projektitietue, kaikki tilat](#one-project-record-every-mode).
> - **Useista kameroista luodut tiedostot nimeävät jokaisen kameran.** Monikaistapino, useiden kameroiden kaistoista muodostettu reaaliaikainen indeksi sekä taulukkomuotoisen kuvan tallennuksen `composite/` sisältävät kunkin lähdekameran sarjanumeron, mallin ja kalibrointipaketin luetteloina (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), ja EXIF-tiedostossa `SerialNumber` sarjanumerot on lueteltu pilkuilla erotettuina. Aiemmin niissä ei ollut kameratietoja lainkaan tai niissä oli vain ensimmäisen kameran sarjanumero. Katso [Useista kameroista luodut tiedostot](#files-made-from-several-cameras).
> - **Vain `raw`-tiedosto sisältää kameran kalibrointitunnisteet.** MicaSense-yhteensopiva sarja (`BlackLevel`, `RadiometricCalibration`,kanavakohtainen `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`-sarjat) sekä `CalibrationJSON`-payload kuvaavat anturin raaka-arvoja, joten ne sisältyvät ainoastaan `raw`-tallennustiedostossa. Kaikki muut tiedostot — tallennustiedostot`debayered`-, `display`-, `radiance`-, `reflectance`- ja hakemistotiedostot, jatkuvan tallennuksen ja monikameran-virran kehykset sekä kaikki `process()`-tuotteet — sisältää kameran tunnisteen (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) sekä kuvauksen yhteydessä kuvan omat tallennustiedot (`XMP-mapir:FrameJSON`: valotus, vahvistus, lämpötila, pikselimuoto, kuvausajankohta). Versiossa 1.2.0 tiedostot sisälsivät myös raakadatan, jota sitä käyttävä työkalu (Pix4D, Metashape) sovelsi pikseliin, jotka eivät olleet raaka-arvoja. Käsittely löytää tällaisen tiedoston kalibroinnin sen sarjanumeron ja `CalBundleSha256`:n perusteella.
>
> **Muut muutokset:**
>
> - **Taulukon automaattinen kuvataajuus mukautuu taulukon tuottamaan taajuuteen.** Taajuus, joka menettää yli 1 % laukaisuistaan, laskee tasoa; tämä arvioidaan useiden pitkien tarkastelujaksojen perusteella yhden sijaan, ja taajuutta, josta se laski, yritetään uudelleen silloin tällöin. Itse asettamaasi taajuutta ei muuteta.
> - **Automaattinen valotus on vakaampi.** Se pysyy vakiona tasaisessa valaistuksessa, vakiintuu muutoksen jälkeen muutamassa sekunnissa, ohittaa pienet kirkkaat kohdat ja palauttaa toimintakuntoon kameran, joka lakkaa reagoimasta kirkkaassa auringonpaisteessa. Katso [Automaattisen valotuksen toiminta](#how-auto-exposure-behaves).
> - **`import_images(recursive=True)` laskee vain omat kuvasi ja tallenteesi.** Kansiossa, joka sisältää projektin `camera_cal/`-tiedoston, olevia kalibrointitiedostoja ei enää lasketa kuviksi ja tallenteiksi, ja projekti, joka on jo ottanut ne mukaan, poistaa ne.
> - **`configure(radiometric_output=...)` on voimassa.** Se määrittää nyt projektin `Export radiance` / `Export reflectance`-tuotteet; aiemmin kunkin projektin omat tuoteasetukset olivat etusijalla, joten `radiance`, `sensor-response` ja `all` veivät kaikki säteilyvoimakkuuden ja heijastavuuden.
> - **`test_array_sync()` käynnistyy kameroiden virran kytkemisen jälkeen.** Se kytkee kameroiden PTP:n päälle ajon ajaksi, odottaa niiden kellojen lukittumista ja vakiintumista ennen ensimmäistä laukaisua (`ptp_lock`) — jokainen kamera lukittuu 120 sekunniksi, noin 2 minuutin kuluttua virran kytkemisestä (`ptp_settle_s=` asettaa sen), sillä juuri synkronoitu kello voi poiketa jopa ~200 us noin 2 minuutin ajan — ja palauttaa kunkin kameran PTP:n siihen tilaan, jossa se sen löysi. Kameraa, joka on edelleen lukittunut aiemmasta testistä, ei odoteta uudelleen. Aiemmin se jätti PTP:n pois päältä yksikytkimisessä verkossa, jossa mikään DAQ-E ei vastannut, ja luki kunkin kameran vapaasti käyvän kellon PTP-kellona, joka ei lukittunut (”PTP ei ole lukittu”, jokaisella ajolla).
> - **`.daq`-tallenne kertoo, valittiinko sen korkki vai oletettiinko se.** DAQ-U, DAQ-M ja DAQ-E toimitetaan ilman suojakantta, eikä Chloros pysty havaitsemaan, onko sellainen asennettu, joten ilman nimettyä suojakantta kytketty anturi saa oletusarvoisesti tunnuksen `sunshine_cosine`, mikä on *oletus*. Jokainen Chloros-tallenne (kaavio 1.30) tallentaa nyt `cap_id_source`:n `cap_id`:n viereen: `operator`, kun olet nimennyt korkin (mukaan lukien `none`), `auto_default`, kun se oli oletusarvo. Kun käsitellään tallennetta, jonka kondensaattori oletettiin, ja projektin DAQ-kondensaattori jätetään asetukseksi Auto, tulostetaan tiedoston nimeksi `[daq-dls] WARNING`: pelkkä DAQ-U auringonvalokorjauksen alla näyttää noin 30-kertaisen lukeman. **Nimeä asentamasi kondensaattori** liitettäessä (`connect_daq_sensor(cap_id=...)` tai `DAQSensorSession.set_cap()`, `chloros-cli daq pool-connect --cap-id` / `daq pool-set-cap` tai GUI:n kondensaattorivalikosta), tai aseta projektin DAQ-kondensaattori, jolloin varoitus katoaa.
> - **Windowsissa ei enää avaudu pip-konsoli-ikkunoita.** Tietokoneella, johon on asennettu Python, taustapalvelin ja jokainen sen kuvankäsittelytyöntekijä suorittivat käynnistyksen yhteydessä kerran komennon `pip`, ja kukin avasi konsoli-ikkunan. Näin ei enää tapahdu.
> - **`.daq`-reaaliaikaiset tallenteet säilyvät sähkökatkon jälkeen.**Tallenne kirjoitetaan lokiin ja levyn täyssynkronointi suoritetaan jokaisen tallennuksen yhteydessä anturin taustalla, joten hidas levy viivästyttää tiedoston muodostumista, mutta ei koskaan lukemia. Tallennus, joka keskeytyi kesken tallennuksen (virrankatkos, pakkopysäytys), voi jättää `<name>.daq-journal`-tiedoston viereensä:**kopioi nämä kaksi yhteen.**Chloros palauttaa lokikirjan ensimmäisellä tiedoston lukukerralla, joten tallenne avautuu sisältäen kaiken ennen keskeytyksen tapahtuneen tallennuksen (noin viimeiset 3 sekuntia menetetään); versiossa 1.2.0 väärällä hetkellä tapahtunut virrankatkos saattoi jättää koko tiedoston lukukelvottomaksi (&quot;tietokannan levykuva on virheellinen&quot;).
> - **`DAQSensorSession.record_start()` ruuhkaisella levyllä ei enää jumita anturia**: lukemat jonotetaan ja kirjoitetaan omalla säikeellään, ja se, jota ei voitu kirjoittaa, lasketaan mukaan `status()`:n `record_errors`:ään (yhdessä `record_last_error`:n kanssa).
> - **Suora `LatticeCamera`:**kehys, jossa ei ole palan valotusta/vahvistusta, ilmoittaa nyt kameran**todellisen** valotuksen ja vahvistuksen (varajärjestelmä luki välimuistissa olevan arvon, joka saattoi jäädä muuttumattomaksi koko automaattisen valotuksen ajaksi), ja `apply_settings(trigger_source='LineN')` / `set_trigger_source('LineN')` soveltavat samaa 5 µs:n tulon häiriönpoistoa, jonka Chlorosin kaapeliohjatut kamerat saavat (suodattamattomana orjakamera laukaisi kehyksiä ilman lähetettyä laukaisusignaalia).
> - **Ohjelmistolaukaisulla toimivat kamerat eivät enää jätä joka toista laukaisua väliin.** Ohjelmistolaukaisutilassa oleva kamera asettaa nyt jonotukseen laukaisun, joka saapuu sen lukemisen aikana (tai Chloros odottaa, kunnes se on valmis), kuten kaapelisynkronoidut kamerat jo tekivät.
> - **`discover_daq_sensors()` nimeää DAQ-E-S:n `DAQ-E-S`:ksi.** Sen `model` oli `DAQ-E` jokaiselle Ethernet-anturille; anturi, joka ei vastaa tunnistustarkistukseen noin 2 sekunnin kuluessa, näkyy edelleen luettelossa nimellä `DAQ-E`.
> - **Uusi: `chloros-mcp`, MCP-palvelin tekoälyavustajille.** Claude Desktop, Claude Code, Cursor ja muut MCP-asiakasohjelmat voivat käsitellä kuvamateriaalia (luoda projektin, tuoda kuvia, valita asetukset, käsitellä, raportoida ajon tulokset) sekä käyttää LATTICE-kameroita, -anturiryhmiä ja DAQ-valosensoreita: muodostaa yhteyden, esikatsella, asettaa valotuksen ja värit, ottaa kuvia, tallentaa. Sen työkalut kattavat tämän SDK:n toiminnot. Se asennetaan SDK:n mukana, ja Linux-järjestelmän .deb-paketti sijoittaa sen kansioon `/usr/bin`. Katso [MCP-palvelin (AI-avustajat)](#mcp-server-ai-assistants).
> - **Uutta: kameran apuohjelmat**, `chloros-cli lattice`-komennot, joita SDK-sovellus kutsuu: väriprofiili, kylläisyys / kontrasti, kirkkaat alueet, valkotasapaino (kertakuvaus, automaattinen, kiinteät suhteet), esiasetukset, suodatinmallit, laiteohjelmiston tila ja päivitys, virta (tunnistus, lepotila, herätys), kohteen kalibrointi, verkkosovittimet (tarkistus, arviointi, korjaus), tiedostojen vinjetointikorjaus, DLS-lukema ja isäntälaitteen tunnistus. Katso [Kamera-apuohjelmat](#camera-utilities).
> - **Uutta: loput CLI:stä SDK-kutsuina**, joten CLI, SDK ja `chloros-mcp` tarjoavat samat ominaisuudet (CLI säilyttää turvallisuussyistä vain kirjautumisen, DAQ-E:n konfiguroinnin ja suorat DAQ-palvelimet): kamerat avataan yhdellä komennolla (`start_camera_continuous()`, `start_multi_camera_stream()`, `capture_cameras_transient()`, `test_array_sync()`, `detect_array_master()`, `analyze_array_models()`), kohdistusprofiilitiedostot (`calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `alignment_profile_info()`, `reorder_alignment_profile()`), hakemistot (`index_catalog()`, `compute_index()`, `capture_live_index()`), PTP-grandmaster (`time_sync_status()` ja muut), projektikansio (`project_folder()`, `set_project_folder()`, `reset_project_folder()`) sekä reseptejä (`run_project_recipe()`, CLI:n oma suoritin). Valosensoreille: `connect_daq_sensor(cap_id=...)`, `DAQSensorSession.set_cap()`, `daq_bindings()` / `set_daq_bindings()`, `daq_band_presets()` ja `light_sensor_reflectance()` (kahden anturin ympäristön ja kohteen heijastavuusarvot sekä indeksit); kameraa varten: `CameraSession.status()` ja `capture_repeated()`; matriisin osalta: `ArraySession.capability()` ja `set_member_settings()` (yksittäisen elementin valotus, vahvistus jne.). Katso [Väliaikaiset matriisit, kohdistusprofiilit ja indeksit](#transient-arrays-alignment-profiles--indices), [Projektikansio ja reseptit](#project-folder--recipes) sekä [DAQ-anturisessiot](#daq-sensor-sessions).
> - **Parannetut LATTICE-indeksituotteet** (kuten NDVI ja muut indeksit) sarjasta `process()` kaikissa käsittelytiloissa.
> - **LATTICE-OCN-kamerat: parannetut säteily-, heijastus- ja indeksituotteet auringonvalossa.** Laivuekorjaus sovelletaan kaistojen erottelun jälkeen — reaaliaikaisesti, tallennuksissa ja `process()`:ssä; RGN-kameroissa ei ole muutoksia. Katso [Radiometrinen tulos](#radiometric-output-lattice-multispectral-pipeline).
> - **Jokainen tallennus tallentaa käyttämänsä DAQ-lukeman, kuten käyttöliittymäkin tekee.** `CameraSession.capture()` ja `ArraySession.capture()` / `continuous()` kirjoittavat `.daq`:n ja sen `.match.json`:n aina, kun DAQ sidotaan tai yhdistetään, mukaan lukien pelkästään raakamuotoiset tallennukset; versiossa 1.2.0-versiossa raaka- tai debayeroitu tallennus tallennettiin vain yhdessä `force_daq=True`:n kanssa. `daq=False` poistaa sen käytöstä ja `daq_tolerance_ms=` asettaa vastaavuustoleranssin; molemmat tulokset kertovat, mitä tallennettiin `.daq_files` ja `.daq_matches`. Tallennus odottaa nyt DAQ:n seuraavaa lukemaa, jotta se voi yhdistää valotuksen molemmin puolin olevat lukemat. Katso [DAQ-lukema, jonka tallennus tallentaa](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
> - **`ArraySession` sisältää reaaliaikaisia kehyksiä ja palvelinpuolen jatkuvan tallennuksen.** `grab()` ja `frame_stream()` palauttavat numpy-kehyksiä yhdestä laukaisusta (sama lukema kuin projektin API:n `ArrayHandle.grab()` / `frame_stream()`), ja `continuous()` tallentaa jokaisen synkronoidun ryhmän taustapalvelimelle ja palauttaa [`ContinuousHandle`](#continuoushandle). Katso [Live-kehykset omassa putkistossasi](#live-frames-in-your-own-pipeline).
> - **Taulukko `grab()` on kiinnitetty yhteen laukaisimeen** (`ArrayHandle.grab()` ja uusi `ArraySession.grab()`). Versiossa 1.2.0 12 neljästä-kameran `ArrayHandle.grab()`-lukemista ulottui kahden laukaisimen yli. Jokaisessa kehyksessä on nyt `group_id`, ja taulukko `frame_stream()` tuottaa kunkin laukaisimen kerran. Katso [Yksi laukaisija per lukema](#one-trigger-per-read--group_id).
> - **Kaistakohtaiset ja kohdistetut kehykset:** `grab()` / `frame_stream()` ottavat `band=`:n, `align=`, `bit_depth=` ja `refl_max=` kameran ja matriisin kahvoista. Katso [Yhden kaistan vieminen](#exporting-a-single-band--band).
> - **Uudet `ArraySession`-metodit ja -argumentit:** `set_target_fps()`, `force_disconnect()`, `capture(after_seq=..., orient=...)` (yhdessä `CaptureResult.group_seq`:n ja `.failed`:n kanssa), `continuous()`, `connect_array(packed=...)`; `capture_repeated()` ketjuttaa peräkkäisiä ryhmiä ja pysähtyy, kun asema on täynnä.
> - **`ArraySession.record()` tallentaa, kun käyttöliittymää ei ole auki.** Kun käyttöliittymän yhdistetty näkymä on suljettu, taustaprosessi renderöi näkymän itse: taulukonNDVI-arvo tai indeksi, jonka välität parametrina `expr=`. `RecorderHandle.stop()` aiheuttaa virheen `ChlorosConnectError`, jos tallennus epäonnistui eli jos se ei kirjoittanut yhtään kehystä, ja säilyttää pysäytystuloksen `.result`:ssa (`with`-lohkon jättäminen ei aiheuta poikkeusta: tarkista `rec.result`). Katso [`RecorderHandle`](#recorderhandle).
> - **Uudet `DAQSensorSession`-menetelmät:** `enable_ptp()`, `force_disconnect()` sekä DAQ-E:n omia asetuksia varten `device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` ja `update_firmware()`. `record_stop()` palauttaa arvot `warning` ja `gaps`. Uudelleenkäynnistys, joka sulki tallennuksen, nimeää sen vastauksessaan tai, jos uudelleenkäynnistys epäonnistuu, `ChlorosConnectError`:ssä (`recording_closed`; katso [DAQ-E-laitteen asetukset](#daq-e-device-settings)). Katso [`DAQSensorSession`-menetelmät](#daqsensorsession-methods).
> - **Poolattu DAQ-E, joka lopettaa tiedonsiirron, kytketään uudelleen taustapalvelimen toimesta** sen aiemmilla asetuksilla, ja sen tallennus jatkuu samassa `.daq`-tiedostossa, jossa aukko on merkitty; `record_stop()` palauttaa katkokset muodossa `gaps`. Versiossa 1.2.0 anturi pysyi hiljaa, kunnes se irrotettiin ja liitettiin uudelleen, mikä lopetti tallennuksen. Katso CLI-ohjeen [DAQ-E, joka lakkaa toimittamasta dataa](cli-reference.md#a-daq-e-that-stops-delivering).
> - **`open_project()` toimii asennetun SDK:n avulla.** Wheel (asennusohjelmanja PyPI:t) sisältää nyt tarvittavat projektin lataajat; aiemmin `open_project()` aiheutti virheen `ImportError`, jossa mainittiin `project_cameras` / `project_sensors`:n virheen missä tahansa pip:llä asennetussa SDK:ssa. `connect_all()` yhdistää tallennetut kamerat, matriisit ja valosensorit käynnissä olevan taustapalvelimen (valosensorit sen DAQ-poolin kautta), ja tallennettu laite, jota se ei voi mitenkään etsiä, näkyy luettelossa kohdassa `errors` syyn kera sen sijaan, että se ohitettaisiin sanomatta sanaakaan. Tallennettu taulukko, joka on jo käynnissä taustapalvelimessa (GUI:n, tai `chloros-cli project connect`:n) liitetään ja jätetään käynnissä, kun aiemmin siitä ilmoitettiin virheenä (&quot;Kamera  on<sn> jo matriisissa <id>&quot;). Katso [`ChlorosProject`-menetelmät](#chlorosproject-methods).
> - **`ChlorosLocal()`, `process_folder()` ja `process_lattice_capture()` etsivät Chlorosin Windowsissa.** Ne etsivät paikkaa, johon asennusohjelma sijoittaa taustapalvelimen (oletuksena `<install folder>\backend\` ja `C:\Program Files\Chloros\backend\`, asennusohjelman omasta tietueesta määritetty mukautettu kansio) ja muodostavat yhteyden jo käynnissä olevaan taustapalvelimeen etsimättä sitä lainkaan. Versiossa 1.2.0 ne antoivat virheilmoituksen ”Chloros-taustapalvelinta ei löydy” asennetulla koneella, ellet antanut `backend_exe=`-parametria.
> - **Käynnistyksen yhteydessä kaatuva taustapalvelu raportoidaan välittömästi.** Kun `ChlorosLocal()` tai smart-connect-kutsu käynnistää taustapalvelun, joka sulkeutuu ennen kuin se ehtii vastata, kutsu heittää virheen muutamassa sekunnissa: `ChlorosBackendError` `ChlorosLocal`:sta ja `ChlorosConnectError` smart--connect-kutsuista, sisältäen lopetuskoodin, suoritetun komennon ja backendin tulostamat viimeiset rivit. Versio 1.2.0 odotti 60 sekunnin käynnistysaikakatkaisun umpeutumista eikä ilmoittanut syytä. Katso [Backendin automaattinen käynnistys](#backend-auto-start).
> - **Taustapalvelin jatkaa toimintaansa skriptien välillä.** Poistuessa `with ChlorosLocal(...)`-lohkosta `process_folder()` / `process_lattice_capture()` palaa ja `chloros-mcp` poistuu, eivät enää pysäytä SDK:n käynnistämää taustapalvelinta; seuraava skripti käyttää sitä uudelleen. Se pysähtyy kohdassa `logout()` (ellei Chloros-sovellusta ole auki tai suoritus ole käynnissä), kohdassa `shutdown_backend()` tai itsestään 30 minuutin kuluttua, jos mikään ei käytä sitä (`CHLOROS_BACKEND_IDLE_MIN` taustaprosessin ympäristössä muuttaa tämän; `0`: ei koskaan).
> - **SDK:n käynnistämää taustaprosessia pyydetään sammumaan, sitä ei lopeteta väkisin.** `shutdown_backend()` antaa taustapalvelimelle mahdollisuuden vapauttaa kameransa ja valosensorinsa sekä saattaa mahdolliset tallennukset loppuun; se lopetetaan vasta, jos se on edelleen käynnissä 10 sekuntia myöhemmin. Versio 1.2.0 lopetti sen välittömästi (Windowsissa pakotettu lopetus), mikä saattoi aiheuttaa sen, että kyseiseen taustapalvelimeen kytketty DAQ-valosensori ei vastannut. SDK:n havaitsema käynnissä oleva taustapalvelin jätetään edelleen rauhaan.
> - **Linuxissa automaattinen käynnistys ei koskaan suorita `backend_server.py`-komentoa nykyisestä kansiosta.** Jos taustapalvelinta ei ollut asennettu, kansiossa, jossa sellainen oli, käynnistetty skripti suoritti sen; nyt käynnistetään vain asennettu taustapalvelin (tai, `ChlorosLocal`:n tapauksessa, se, jonka `backend_exe=` nimeää).
> - **`configure(indices=...)`**lisää jokaisen esiasetuksen projektin kameran kaistoille, mukaan lukien uuden projektin, ja antaa varoituksen (`UserWarning`), jos nimeä ei voida lisätä.**`import_images()`** raportoi `.daq`-tallenteet nimellä `scan_count` eikä enää varoita LATTICE-tallennuskansiosta.
> - **`read_image_audit_tags()` toimii pip-asennuksesta.** Se lukee kuvan metatiedot itse; versiossa 1.2.0 se palautti `{}`:n ilman työpöytäasennuksen sisäisiä osia Python-polulla.
> - **`capture()` heti `connect_array()`:n jälkeen** odottaa jopa 20 sekuntia taulukon ensimmäistä kehysryhmää sen sijaan, että se epäonnistuisi HTTP-virheellä 503 ”Array still warming up”.
> - **`status()` ilmoittaa jumittumisesta.** Uusi `stalled`-kenttä on True, kun kaappaussilmukka on käynnissä, mutta se ei julkaise mitään, ja `actual_fps` on tällöin 0. Versiossa 1.2.0 `actual_fps` säilytti viimeisen arvonsa jumiutumisen ajan, joten jumissa oleva taulukko näytti arvoa `actual_fps > 0`, kun taas kaappaukset epäonnistuivat virheellä 503 ”No fresh synced frame group”.
> - **Vähemmän epätäydellisiä kehysjoukkoja ja tallennuksen 503-virheitä:** jokaisen orjakameran laukaisutulo on nyt puhdistettu häiriöistä.
> - **Pakattu 12-bittinen siirto** (`connect_array(packed="auto")`) nostaa koko anturijärjestelmän kuvataajuutta noin kolmanneksella. Katso [Pakattu 12-bittinen siirto](#packed-12-bit-transport-packed).
> - **Matriisin valotus ROI-alueen alla:** pieni ROI-alue ei enää rajoita jäsenen valotusta anturin vapaakäyntijaksoon (4,9 ms resoluutiolla 768 × 304). Katso [Valotus matriisin jäsenillä](#exposure-on-array-members).
> - **Automaattinen valotus ei enää alenna matriisin automaattista kuvataajuutta.** Kaapelilla synkronoitu matriisi toimii sillä taajuudella, jonka verkko ja kameroiden lukunopeus sallivat; automaattinen valotus täyttää kyseisen taajuuden antaman jakson ja nostaa sitten vahvistusta. Manuaalinen valotus, joka on pidempi kuin kyseinen jakso, hidastaa laukaisua sen mukaiseksi sen sijaan, että se hylättäisiin. Katso [Valotus matriisin jäsenissä](#exposure-on-array-members).
> - **`read_nodes()`**lisää `frame_values` ja `notes` automaattisen valotuksen ollessa käynnissä (katso [`read_nodes()` automaattisen valotuksen aikana](#read_nodes-under-auto-exposure--frame_values-and-notes)). Kun kamera hylkää valotuksen tai vahvistuksen,**`set_settings()`** ilmoittaa hyväksymänsä alueen ja palauttaa automaattitilan, jonka se oli kytkenyt pois päältä kyseisen arvon osalta; `set_settings(target_brightness_auto=True)` on uusi.
> - **`orient=`** kohdissa `ArraySession.capture()` / `capture_fastest()` / `continuous()` / `burst()` ja `ArrayHandle.capture()` / `capture_smart()` kääntää tuotteita kameran suunnan mukaan. Katso [`ArraySession`-menetelmät](#arraysession-methods).
> - **`DLS()` vaatii anturin oman `mac_address`-arvon.** Oletus-MAC-osoitetta ei enää ole (se oli yhden yksikön osoite); `DLS()` ilman sitä aiheuttaa virheen `DLSError`. Katso [DAQ-M-kaistojen suuruusluokat](#daq-m-band-magnitudes--dlsband_magnitudes).
> - **Windows-asennusohjelma** tarjoaa vaihtoehdot ”Ota Windowsin pitkät polut käyttöön” ja ”Jätä Chloros-taustapalvelu pois Microsoft Defenderin tarkistuksesta”, jotka molemmat ovat valittuna oletuksena. Katso [Vaihe 1](#step-1--install-the-chloros-platform-package).
> - **Linuxin .deb-paketti** asentaa järjestelmän Python-SDK:n `python3` (aiemmin se vain asensi wheel-paketin levylle), kasvattaa ytimen socket-vastaanottopuskureita, joita kameravirrat tarvitsevat, ja lisää taustapalvelun, joka toimii käyttäjänäsi. Jetson-paketti on `chloros_1.3.0_arm64_jp6.deb`, JetPack 6:lle, jossa on mukana JetPackin CUDA PyTorch. Katso [Vaihe 1](#step-1--install-the-chloros-platform-package) ja [Vaihe 2](#linux-deb).

---

## Arkkitehtuuri 60 sekunnissa

SDK on ohut Python-kerros Chloros-taustapalvelun päällä (sama Flask-palvelin, jota työpöydän graafinen käyttöliittymä ja CLI käyttävät). Automaatiota varten tuodaan `chloros_sdk` ja kutsutaan korkean tason metodeja; taustalla jokaisesta kutsusta tulee HTTP-pyyntö paikalliselle taustapalvelimelle portissa 5000 — `http://127.0.0.1:5000/api/...` (tarkoituksella ei `localhost`, joka Windowsissa ohjautuu ensin `::1`:ään ja vie noin 2 sekuntia pyyntöä kohti, kun kohteena on pelkästään IPv4-taustapalvelin). Taustapalvelu hallinnoi laitteistopoolia — kameroita, DAQ-antureita, kohdistusprofiileja, kehyspuskuria — joten SDK-skriptit voivat toimia rinnakkain graafisen käyttöliittymän kanssa ilman, että ne kilpailevat sarjaporttien tai verkkokortin kaistanleveyden käytöstä.

Käytössäsi on kolme käyttöliittymää:

1. **`ChlorosLocal` + vapaat funktiot** (`process_folder`, `process_lattice_capture`) — Kuvan käsittelyputki. Suorita koko kansion kalibrointi, debayer-käsittely ja indeksin vienti yhdellä Python-kutsulla.
2. **Smart-connect-käsittelyt** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Avaa pysyvä taustasessio reaaliaikaiselle laitteistolle. Sama ”smart-prep”-prosessi kuin graafisessa käyttöliittymässä: verkkotunnistus, tason automaattinen valinta, PTP, AE-alustaminen, GPIO-laukaisun konfigurointi.
3. **`ChlorosProject` / `open_project`** — Lataa tallennettu projekti (kansio, jossa on tiedostot `cameras.json` + `sensors.json` + `project.json`), yhdistä sen kamerat, matriisit ja valosensorit kerralla ja ohjaa tallennuksia nimetyillä kahvoilla.

Pinnat 1 ja 2 **käynnistävät paikallisen taustaprosessin automaattisesti**, jos sellainen ei ole jo kuuntelutilassa (sama mukana toimitettu binääritiedosto, jonka GUI/CLI käynnistää) — joten pelkkä skripti toimii uudesta komentotulkista ilman, että sinun tarvitsee käynnistää taustaprosessia ensin: `ChlorosLocal` (ja `process_folder()` / `process_lattice_capture()`), `connect_camera()`, `connect_array()`, `connect_daq_sensor()`, `discover_lattice_cameras()` ja `discover_daq_sensors()`. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` ja `analyze_array_network()` eivät koskaan käynnisty itsestään: ne tarvitsevat käynnissä olevan taustapalvelimen. Määritä `auto_start_backend=False`, jos haluat kieltäytyä tästä (esim. kun osoitetaan etätaustapalvelimeen, jota ei koskaan käynnistetä). Katso [Taustapalvelimen automaattinen käynnistys](#backend-auto-start). Surface 3 toimii eri tavalla: `open_project()` ei ota vastaan `auto_start_backend`-parametria, eikä `connect_all()` koskaan käynnistä taustapalvelinta — se tunnustelee `http://127.0.0.1:5000`:ää kerran ja, jos mikään ei vastaa, siirtyy suoraan (taustapalvelimettomaan) `lattice_sdk`-laitteen ohjaukseen. Tämä vaatii Arena-SDK-ajoympäristön ja OpenCV (`cv2`) -kirjaston, joka on tuotava Python-kielessä, mutta jota SDK-wheel ei asenna; ilman niitä jokainen tallennettu kamera ja taulukko raportoidaan virheeksi, syy mukaan lukien. Projektin valosensoreilla ei ole taustapalvelimettomaa reittiä: ilman käynnissä olevaa taustapalvelinta jokainen tallennettu sensori raportoidaan virheeksi, syy mukaan lukien (katso [`SensorHandle`](#sensorhandle)). Vain `proj.process()` ja `stream(..., overlays=True)` muodostavat viiveellä `ChlorosLocal()`:n (joka käynnistyy automaattisesti).

Kaikki kolme vaativat todennusta-valvottuja: suorita `chloros-cli login` kerran koneella tai kirjaudu sisään työpöydän käyttöliittymän kautta. Se, mitä virheellinen kutsu ilman voimassa olevaa istuntoa aiheuttaa, riippuu käyttöliittymästä (katso [Poikkeukset](#exceptions)): `ChlorosAuthenticationError` syntyy `ChlorosLocal`:stä (ja `process_folder()` / `process_lattice_capture()`), `ChlorosConnectError` älykkäistä-connect-kutsuista ja niiden istunnoista, `status: "error"`, jossa on `error_code`, joka johtuu `analyze_array_network()`:stä, sekä merkintä kohdassa `errors` `connect_all()`-raportissa.

Vaatimukset:
- Python 3.7+ (paketin ilmoituksen mukaan; kehitetty/testattu versiolla 3.10)
- Chloros Desktop asennettuna paikallisesti (taustaprosessi sisältyy asennusohjelmaan)
- Aktiivinen Chloros+-kirjautuminen **millä tahansa maksullisella Chloros+-tasolla (Copper tai korkeampi)**; ilmaisella**Iron**-tasolla ei ole SDK- tai CLI-käyttöoikeutta. Tämä vaaditaan**palvelin-puolella**: jokaisessa SDK- tai CLI-merkityssä pyynnössä on oltava sekä aktiivinen istunto että maksullinen tilaus. Copper-tasoa alemmalla tilauksella saadaan virhekoodi `403` yhdessä virhekoodin `error_code: PLAN_UPGRADE_REQUIRED` kanssa (virhekoodi `ChlorosLicenseError` virhekoodista `ChlorosLocal`, `ChlorosConnectError` smart-connect-kutsuista). Uloskirjautunut soittaja saa sen sijaan koodin `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError` koodista `ChlorosLocal`, `ChlorosConnectError` smart-connect-puheluista) — nämä kaksi ovat erillisiä, koska `chloros-cli login`:n uudelleenkäynnistys korjaa virheen `401`, mutta ei voi korjata virhettä `403`, joka vaatii suunnitelman päivityksen.
- Offline-käyttö on tuettu paketin armonaikana: käyttöoikeustaso luetaan palvelimen vahvistusvälimuistista (5 min) tai allekirjoitetusta, laitteeseen sidotusta lisenssivälimuistista (30 päivää kuukausipaketeissa, vuosipaketeissa tilauksen voimassaolon loppuun asti). Kun tämä armonaika päättyy, tilaus muuttuu ilmaiseksi ja SDK- ja CLI-käyttö keskeytyy, kunnes laite pääsee kerran yhteyteen palvelimen kanssa. Virhe `chloros-cli status` (`GET /api/license-status`) pysyy näkyvissä ilmaisella tasolla, jotta syy on selvillä.
- Windows 10/11 64-bittinen, **Ubuntu 22.04 LTS tai uudempi**tai Jetson (JetPack 6). Ubuntu 20.04:ää**ei** tueta: `.deb`:n riippuvuudet johdetaan siitä, mihin taustapalvelin linkittyy, mukaan lukien `libc6 (>= 2.35)`, ja Focal toimittaa glibc 2.31:n.

---

## Asennus

Python-SDK on ohut Python-kerros Chloros-taustaprosessin päällä. Ellet käytä taustaprosessia toisella koneella (katso [Etätaustaprosessitila](#remote-backend-mode-pip-only-host-via-tunnel)), tarvitset **Chloros-työpöytäpaketin asennettuna paikallisesti** (Windows-asennusohjelma tai Linux `.deb`), joka sisältää DAQ-valosensorit. Paketti sisältää taustapalvelimen binääritiedoston, Arena SDK -ajoympäristön LATTICE-kameroille sekä kalibrointipaketit.

Uusimmat lataukset: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Vaihe 1 — Asenna Chloros-alustapaketti

#### Windows (.exe)

1. Lataa `Chloros Installer 1.3.0.exe` lataussivulta.
2. Suorita asennusohjelma ja seuraa ohjatun asennuksen ohjeita. Oletusasennuskansio on `C:\Program Files\Chloros\`.
3. Käynnistä Chloros vähintään kerran ja kirjaudu sisään Chloros+-tililläsi.

Asennusohjelma sallii taustapalvelun (`chloros-backend.exe`, ei mitään muuta) läpi Windowsin palomuurin jokaisessa verkkoprofiilissa: UDP-tuloliikenne kameravirtoja, PTP:tä, mDNS:ää ja DAQ-E-virtaa varten sekä TCP-tuloliikenne vain paikallisesta aliverkosta DAQ-E:n muodostamaa yhteyttä varten tietokoneeseen laiteohjelmiston päivityksen aikana ([`update_firmware()`](#daq-e-device-settings)). Poisto-ohjelma poistaa molemmat säännöt. Taustapalvelimen HTTP-API pysyy joka tapauksessa osoitteessa `127.0.0.1`.

Ohjatun toiminnon **Järjestelmäasetukset**-ryhmässä on kaksi vaihtoehtoa, jotka molemmat ovat valittuna oletuksena:

| Vaihtoehto | Toiminto | Hiljaisen asennuksen poisto |
| --- | --- | --- |
| **Ota Windowsin pitkät polut käyttöön** | Sallii projektikansioiden sijoittamisen syvemmälle kuin Windowsin 260 merkin polkuraja sallii. Kun pitkät polut ovat pois käytöstä (Windowsin oletusasetus), tuotteilleen liian syvälle sijoitettu projektikansio hylätään ennen kuin mitään käsitellään — katso [Vianmääritys](#vianmääritysvinkit). | `/MERGETASKS="!longpaths"` |
| **Chloros-taustapalvelu pois Microsoft Defenderin tarkistuksesta** | Estää Defenderin reaaliaikaisen-aikaisen tarkistuksen jokaisesta tiedostosta, jonka kameran tallennus kirjoittaa. Suurilla tallennusnopeuksilla tarkistus vie noin 30 % nopeudesta: mitattuna ROI 512:lla 9-kamerajärjestelmässä jatkuva raakatallennus sujui 52,7 tallennusta/s tarkistuksen kanssa ja 77,7 tallennusta/s, kun taustapalvelu oli suljettu pois. | `/MERGETASKS="!defenderexclusion"` |

Jatkuva tallennus ilmoittaa Defenderin kuormituksen nimellä `defender_cores` (katso [`ContinuousHandle`](#continuoushandle)).

#### Linux amd64 (.deb)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Asenna `apt install ./…`:n avulla, älä `dpkg -i`:n: apt asentaa myös Python-SDK:n vaatimukset, joita paketti suosittelee mutta ei vaadi (katso [Vaihe 2](#linux-deb)).

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Jetson Orin / Orin NX / Orin Nano -laitteille JetPack 6:ssa (L4T R36). Sen PyTorch on JetPack 6:n CUDA-rakennus, joten GPU-käsittely tapahtuu Orinin GPU:lla; se käyttää JetPack 6:n asentamia CUDA- ja cuDNN-kirjastoja (`sudo apt install nvidia-jetpack` kuvassa, josta ne puuttuvat).

#### Linux: vastaanottopuskurit ja taustapalvelu

Molemmat .deb-paketit nostavat ytimen socket-vastaanottopuskurien koon, jota LATTICE-kameravirrat tarvitsevat, 32 MB:iin (`/etc/sysctl.d/60-chloros-gige.conf`). Ytimen oletusasetuksilla kamerapaketit hylätään ytimessä ja array menettää kehyksiä ja kuvataajuutta; `chloros-cli lattice network` ilmoittaa puskureiden tilasta, kun ne ovat vähissä. Katso CLI-ohjeen kohta [Linux-isäntäkoneet — socket-vastaanottopuskureita](cli-reference.md#linux-hosts--socket-receive-buffers).

Käynnistä backend palveluna **omalla käyttäjätunnuksellasi** päättömällä koneella, jotta skripti löytää käynnissä olevan instanssin (`connect_all()` ei koskaan käynnistä backendia):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
```

Tällöin taustapalvelu toimii sinun nimissäsi: sen kirjoittamat tallenteet ovat sinun, ja se käyttää sinun `chloros-cli login`-tunnustasi. Root-tunnus `chloros-backend.service` on edelleen asennettuna koneille, joissa se on otettu käyttöön, mutta jokainen tiedosto, jonka root-taustapalvelu kirjoittaa, kuuluu rootille. Katso CLI-ohjeen kohta [Taustapalvelun ajaminen palveluna](cli-reference.md#running-the-backend-as-a-service-headless).

Palvelun ajama taustapalvelu on jaettu, eikä asiakas voi koskaan pysäyttää sitä: SDK-skriptit ja `chloros-cli` käyttävät sitä sellaisenaan (`ChlorosLocal.shutdown_backend()` pysäyttää vain sen taustapalvelun, jonka `ChlorosLocal` on itse käynnistänyt), ja `GET /api/status` nimeää yksikön nimellä `service`. Käynnistä se uudelleen komennolla `sudo systemctl restart chloros-backend@$USER.service`. Paketin päivittäminen pysäyttää taustapalvelut asennuksen ajaksi, joten käynnissä oleva skripti menettää kameransa ja anturinsa ja käynnistää sitten kaikki käytössä olevat uudelleen; paketin poistaminen pysäyttää ne ja poistaa käytöstä ne, jotka olivat käytössä. Vain yksi taustapalvelu on käynnissä kerrallaan, joten ota käyttöön yksi yksikkö: toinen kieltäytyy käynnistymästä ja pysyy tilassa `failed` -tilassa (lopetustila 98, ei käynnisty uudelleen), ja sen `systemctl status` kertoo syyn. Jos haluat vaihtaa yksikköä, poista käytöstä se, jota et käytä.</id></sn>

### Vaihe 2 — Asenna Python-SDK

**Chloros-asennusohjelma toimittaa mukana sopivan SDK-wheel-tiedoston.** Jokainen Windows-asennusohjelma ja Linux-.deb-paketti tallentaa levylle `chloros_sdk-X.Y.Z-py3-none-any.whl`-tiedoston, joka vastaa tarkalleen GUI-, CLI- ja taustapalvelinversiota. Sinun ei tarvitse seurata PyPI:tä pysyäksesi ajan tasalla.

#### Windows

Asennusohjelma suorittaa automaattisesti `pip install`-tiedoston mukana toimitetulla wheel-tiedostolla käyttäen järjestelmän Python-tulkkiasi (suositellaan `py.exe`-käynnistysohjelmaa, varasuunnitelmana `python -m pip`). Mitään toimenpiteitä ei tarvita — `import chloros_sdk` toimii Python-ympäristössäsi asennuksen onnistuttua. Jos koneella ei ole Pythonia, asennusohjelma ohittaa tämän vaiheen hiljaisesti, ja GUI + CLI jatkavat toimintaansa.

#### Linux (.deb)

Molemmat .deb-paketit (x86_64 ja Jetson) asentavat SDK:n järjestelmälle `python3`, joten `import chloros_sdk` toimii asennuksen jälkeen ilman pip:iä ja verkkoyhteyttä. Paketit (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` ja projektilataimet) siirretään `/usr/lib/python3/dist-packages`:ään, ja SDK:n `chloros-mcp`-komento ([MCP-palvelin](#mcp-server-ai-assistants)) sijoitetaan `/usr/bin`:ään; kaikki nämä kuuluvat kyseisen paketin omistukseen: päivitys korvaa ne ja paketin poistaminen poistaa ne.

SDK:n omat vaatimukset, `python3-numpy` ja `python3-requests`, ovat **suositeltavia, eivät pakollisia** (käyttöliittymä, CLI ja taustapalvelu ovat itsenäisiä): `sudo apt install ./chloros_<version>_amd64.deb` (Jetsonissa `…_arm64_jp6.deb`) asentaa ne, mutta `dpkg -i` ei. Asennuksen tuloste ilmoittaa SDK-version, jonka `python3` tuo mukanaan, mahdolliset puuttuvat riippuvuudet sekä `chloros_sdk`-paketin, joka sijaitsee muualla `python3`:n polulla ja peittää paketin kopion (kuten aiemmin `pip install --user`:llä asennettu versio).

Virtualenv-ympäristössä, joka ei näe järjestelmän paketteja, asenna mukana toimitettu wheel-tiedosto siihen:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Air-gapped-Jetson-asennuksissa tämä tapahtuu täysin offline-tilassa — SDK ja wheel-tiedosto ovat jo levyllä.

#### Julkinen PyPI

Pelkästään pip-isäntäkoneille (joihin ei ole asennettu Chloros-työpöytäpakettia; tällöin SDK-ohjelmisto ohjaa taustapalvelinta toisella koneella, katso [Etätaustapalvelintila](#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

PyPI sisältää uusimman vakaan julkaisun. Pidä SDK:n versio samana kuin sen ohjaaman Chloros-taustapalvelimen versio (katso [Versiointi ja yhteensopivuus](#versioning--compatibility)). DAQ-M-apuohjelma `DLS` tarvitsee myös `bleak`: `pip install "chloros-sdk[camera]"` (tai `pip install bleak`).

#### Varmista

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Vaatii Chloros+-tilauksen.** Kaikki SDK-kutsut edellyttävät aktiivista Chloros+-kirjautumista. Suorita `chloros-cli login user@example.com 'YourPassword'` kerran kutakin konetta kohti; kirjautumistiedot tallennetaan välimuistiin `~/.chloros/`:ssä.

### Tarvitsenko työpöytäpaketin?

Pelkkä pip-paketti **ei** riitä useimpiin työnkulkuihin. Tässä on luettelo siitä, mitä kukin SDK-käyttöliittymä vaatii:

| SDK-käyttöliittymä | Tarvitaanko työpöytäpakettia? | Miksi |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Kyllä**(käynnistääksesi yhden)**/ Ei** (jos taustapalvelu on jo käytettävissä) | Käytä käynnissä olevaa taustapalvelua (työpöytäsovelluksen omaa tai jonkin toisen käynnistetyn skriptin); muussa tapauksessa käynnistä asennettu taustapalvelu automaattisesti: `/usr/lib/chloros/chloros-backend` (Linux) tai `<install folder>\backend\chloros-backend.exe` (Windows; oletuksena `C:\Program Files\Chloros\backend\`, mukautettu asennuskansio luetaan asennusohjelman tietueesta). Anna `backend_exe=`, jos haluat nimetä sen itse. |
| `chloros-mcp` ([MCP-palvelin](#mcp-server-ai-assistants)) | **Kyllä**(käynnistääksesi yhden)**/ Ei** (taustapalvelin jo käytettävissä) | Perustuu `ChlorosLocal`:ään, joten sama kuin edellisessä rivissä. |
| `read_image_audit_tags` | **Ei** | Lukee käsitellyn kuvan omat metatiedot (vain Python-vakiokirjasto). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Kyllä**(paikallinen)**/ Ei**(etä) | Puhtaat HTTP-asiakasohjelmat taustapalvelimen kautta. Paikallinen taustapalvelin → vaatii työpöytäpaketin. Etätaustapalvelin → `backend_url=`**tunnelin kautta** (katso Etätaustapalvelintila — mukana toimitetut taustapalvelimet sitovat vain loopback-liitännän). |
| `ChlorosProject` / `open_project` | **Kyllä** | Ajaa tallennettuja projekteja taustapalvelimen kautta. |
| Suorat LATTICE-luokat (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Kyllä** | Tarvitaan Arena SDK -ajoympäristö, joka toimitetaan työpöytäpaketin mukana, sekä OpenCV (`cv2`) Python-ympäristössäsi, jota työpöytäpaketti tai SDK eivät asenna (`pip install opencv-python`). Muussa tapauksessa `CAMERA_AVAILABLE` on tuonnin yhteydessä `False`. `DLS` tarvitsee myös `bleak`:n. |

### Etä-backend-tila (vain pip-isäntä, tunnelin kautta)

> **Asennettuun backendiin ei pääse LAN-verkon kautta.** Se kuuntelee
> vain loopback-porttia (`127.0.0.1` / `::1`), joten `backend_url="http://<lan-ip>:5000"`
> **ei pääse käsiksi asennettuun Chloros-palvelimeen**. Jos haluat käyttää toisella
> koneella olevaa taustapalvelinta, ohjaa sen loopback-portti itse ja osoita SDK
> tunneliin:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"     # the tunnel endpoint
serials = ["213800234", "214000533"]   # master first

# auto_start_backend=False: the tunnel URL looks local, so without it a tunnel
# that is down would start this machine's own backend, if one is installed.
arr = chloros_sdk.connect_array(serials, backend_url=BACKEND,
                                auto_start_backend=False)
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                     backend_url=BACKEND,
                                     auto_start_backend=False)

# The backend opens every path on ITS machine: pass absolute paths valid there.
# (A relative path would be made absolute against this machine's working folder.)
arr.capture("/home/chloros/captures/run1", processing="raw")
```

Headless-/CI-/robotiikkakoneissa voidaan pitää yksi kone, johon on asennettu täysi työpöytäympäristö, ”Chloros-palvelimena” ja `pip install chloros-sdk` kaikkialla muualla — mutta niiden välinen siirto tapahtuu yllä mainitun käyttäjän järjestämän tunnelin kautta, ei suoran LAN-URL-osoitteen kautta.

> **`ChlorosLocal` pip-only-isäntäkoneella.** `ChlorosLocal(backend_url=BACKEND)` muodostaa yhteyden taustapalvelimeen, johon se pääsee ilman työpöytäasennusta: se etsii paikallista taustapalvelimen binääritiedostoa vain silloin, kun sen on käynnistettävä sellainen (mikään ei vastaa ja `auto_start_backend=True`). Versioon 1.2.0 asti se etsi ensin binääritiedostoa ja aiheutti virheen `ChlorosBackendError` (”Chloros-taustapalvelinta ei löydy…”) pelkästään pip-isäntäkoneella, vaikka taustapalvelin olikin tavoitettavissa. Kansiot, jotka välität (`process_folder`, `import_images`) tarkistetaan tällä koneella ja taustapalvelin lukee ne, joten etätaustapalvelimen tapauksessa niiden on viitattava samaan kansioon molemmissa, kuten jaettuun asemaan.

### Pelkästään DAQ:ta käyttävä työnkulku (pelkästään pip-isäntä)

`pip install chloros-sdk` ei yksinään voi ohjata DAQ-anturia: DAQ-anturia ohjataan aina Chloros-taustapalvelimen kautta `connect_daq_sensor()`:n avulla:

- **Anturi tällä koneella:** asenna Chloros-työpöytäpaketti tähän. `connect_daq_sensor()` käynnistää backendinsä, jos sitä ei ole käynnissä.
- **Anturi toisella koneella:** kyseiselle koneelle tarvitaan työpöytäpaketti; pääset sen taustapalvelimeen [Etätaustapalvelintilan](#remote-backend-mode-pip-only-host-via-tunnel) tunnelin kautta. Taustapalvelin avaa anturin omalla koneellaan, joten DAQ-U on kytketty kyseiseen koneeseen, DAQ-M on sen Bluetooth-kantaman sisällä ja DAQ-E on verkossa, johon se pääsee käsiksi.

```python
import chloros_sdk

# http://127.0.0.1:5000 is the local desktop install's backend, or the tunnel.
with chloros_sdk.connect_daq_sensor() as daq:   # smart-detect USB / BLE / ETH
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Kummassakin tapauksessa taustapalvelin noudattaa Chloros+-suunnitelmaa: kirjaudu sisään koneelle, jolla taustapalvelin toimii (`chloros-cli login` tai työpöytäsovellus) millä tahansa maksullisella tasolla (Copper tai korkeampi). Uloskirjautunut taustapalvelin ja Copper-tasoa alempi tilaus aiheuttavat molemmat `ChlorosConnectError`-virheen, joka sisältää taustapalvelimen syyn (`AUTH_REQUIRED` / `PLAN_UPGRADE_REQUIRED`).

---

## Pikaopas

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Ylätason API-hakemisto

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by CameraSession.capture / ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job
chloros_sdk.ContinuousHandle                      # handle for an array continuous() capture
chloros_sdk.GrabResult                            # dict returned by grab() / yielded by frame_stream()
chloros_sdk.FrameFetchError                       # raised by grab()/frame_stream() on a refusal

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()
chloros_sdk.daq_bindings() / set_daq_bindings(cameras=, arrays=)   # which DAQ feeds which camera
chloros_sdk.daq_band_presets(filter_code=None)    # band sets for Ambient + Object reflectance
chloros_sdk.light_sensor_reflectance(ambient_id, object_id, ...)   # two DAQs -> reflectance + indices
chloros_sdk.time_sync_status() / time_sync_peers() / time_sync_cameras()   # PTP grandmaster
chloros_sdk.restart_time_sync() / set_time_sync_priority(priority1=, priority2=)

# === Camera utilities (the CLI's lattice commands; by serial) ===
chloros_sdk.camera_color_profile(serial) / set_camera_color_profile(serial, ...)
chloros_sdk.camera_color_adjust(serial) / set_camera_color_adjust(serial, ...)
chloros_sdk.camera_highlights(serial) / set_camera_highlights(serial, value)
chloros_sdk.camera_white_balance_once(serial) / set_camera_white_balance(serial, ...)
chloros_sdk.camera_presets() / apply_camera_preset(serial, preset)
chloros_sdk.camera_filter_models() / camera_filter_model_info(name)
chloros_sdk.camera_firmware_status() / update_camera_firmware(serial)
chloros_sdk.camera_power_probe(serial) / sleep_camera(serial) / wake_camera(serial)
chloros_sdk.calibrate_camera_with_target(serial, ...)
chloros_sdk.network_adapters(...) / fix_network_settings()
chloros_sdk.apply_vignette_correction(input_dir, output_dir, ...)
chloros_sdk.read_dls(mac, ...) / probe_system(...) / system_info()

# === Transient cameras, alignment profiles, indices (cameras opened for one command) ===
chloros_sdk.start_camera_continuous(serial, ..., daq=None, daq_tolerance_ms=None)  # → CliJob; a DAQ bound to the camera records one .daq per frame (daq=False: none)
chloros_sdk.start_multi_camera_stream(serials, ...)   # → CliJob
chloros_sdk.capture_cameras_transient(serials, ...)
chloros_sdk.test_array_sync(serials, count, ptp_settle_s=None) / detect_array_master(line) / analyze_array_models(models)
chloros_sdk.calibrate_alignment_profile(serials, save_path, ...)
chloros_sdk.capture_aligned_multiband(profile_path, ...) / start_aligned_stream(profile_path, ...)  # → dict / CliJob
chloros_sdk.alignment_profile_info(path) / reorder_alignment_profile(path, ...)
chloros_sdk.index_catalog() / compute_index(input_path, ...) / capture_live_index(profile_path, ...)
chloros_sdk.CliJob                                # stats() / stop() / wait()

# === Project lifecycle ===
chloros_sdk.open_project(path, relink=None)       # → ChlorosProject
chloros_sdk.project_folder() / set_project_folder(path) / reset_project_folder()
chloros_sdk.run_project_recipe(recipe, project_path)  # `chloros-cli project run`
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics               # DLS: DAQ-M over Bluetooth; needs bleak

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosProjectFilesMissingError      # a ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True when chloros_sdk.project imported; open_project()
                                 # still raises ImportError without the project loaders
```

---

## Kuvan käsittely — `ChlorosLocal`

Pääputkiluokka. Käynnistää taustaprosessin, kun se luodaan (jos mikään ei vastaa ja `auto_start_backend=True`), luo ja määrittää projekteja, valvoo etenemistä ja palauttaa suorituksen jälkeiset yhteenvedot.

### Konstruktori

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

### Menetelmät

| Menetelmä | Kuvaus |
| --- | --- |
| `create_project(project_name, camera=None)` | Luo uuden projektin (valinnaisesti kameramallin avulla, kuten `"Survey3N_RGN"`). |
| `open_project(project_path, relink=None)` | Avaa tallennetun projektin (sen kansion tai sen nimen projektikansiossa) ja asettaa sen avoimeksi projektiksi. Siirretyn projektin polut osoitetaan uudelleen sen nykyiseen kansioon; `relink` nimeää kansion, jossa muut puuttuvat tiedostot ovat nyt. Palauttaa taustapalvelimen vastauksen `source_files`:llä; heittää poikkeuksen `ChlorosProjectFilesMissingError`, jos mitään ei löydy. Katso **Siirretty projekti** alla. |
| `import_images(folder_path, recursive=False)` | Tuo RAW/TIF/JPG/DNG-kuvat **ja `.daq` valosensorin tallenteet**. Palauttaa `count` (kuvat) ja `scan_count` (tallenteet). Antaa varoituksen vain, jos kansiossa ei ole kumpaakaan. |
| `export_light_sensor(daq=True, csv=True)` | Kirjoita kalibroidut `.daq` + `.csv` jokaisesta projektin valosensoritallenteesta kansioon `<project>/Light Sensor/`. Katso [Valosensoritallenteet](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Aseta käsittelyn säätimet. `indices` ovat esiasetusten nimiä (`NDVI`, `GNDVI`, `OSAVI`, …), joista kukin on lisätty projektiin kyseisen kameran kaistojen perusteella; nimi, joka ei ole esiasetus tai jota kaistat eivät voi muodostaa (`NDRE` vaatii RE- ja NIR-kaistoja, joita yksikään kamera ei tarjoa), saa nimekseen `UserWarning` eikä sille luoda indeksikuvaa. `daq_log_path` nimittää valosensorin tallennuksen (`.daq`- tai `.csv`-tiedosto tai niiden sisältävä kansio), johon jokainen kehys täsmäytetään sekä LATTICE- että Survey3-ohjelmissa kuvien mukana tuodun tallenteen sijaan; jos polku puuttuu tai ei sisällä tallennetta, käytetään tuotuja tallenteita. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Suorita prosessiketju. Palauttaa `{"status": "complete", "async": False}`-tiedoston, sekä `summary`-avaimen, jos taustapalvelu tarjoaa sellaisen — katso [Suorituksen jälkeinen yhteenveto ja vinkit](#post-run-summary--hints) — `export_check_failure`, kun ajon vientitietoja ei voitu tarkistaa, ja `image_failure`, kun kuva katosi. `wait=False` palauttaa `{"status": "started", "async": True}`:n kerralla. |
| `get_config()` / `get_status()` / `status()` | Tarkista taustapalvelimen tila. |
| `logout()` | Kirjaudu ulos: tyhjentää välimuistissa olevat tunnistetiedot ja vapauttaa tämän koneen laitepaikan (tuloksena `device_deregistration`). Käynnissä olevaa käsittelykierrosta ei keskeytetä: se päättyy siinä tilassa, jossa se aloitettiin (tuloksessa `run_in_progress: true` ja `run_mode`), ja seuraava kierros käyttää vakiokäsittelyä. |
| `shutdown_backend()` | Pysäytä taustaprosessi, jos tämä instanssi on käynnistänyt sen; jo käynnissä oleva taustaprosessi jätetään rauhaan. `with`-lohkon jättäminen tai instanssin siirtyminen roskienkeräykseen ei pysäytä sitä: taustaprosessi jatkaa toimintaansa seuraavaa skriptiä varten ja pysäyttää itsensä 30 minuutin kuluttua, jos mikään ei käytä sitä. Taustaprosessia pyydetään sammumaan, joten se vapauttaa kameransa ja valosensorinsa ja lopettaa ensin mahdolliset tallennukset; se lopetetaan vasta, jos se on edelleen käynnissä 10 sekuntia myöhemmin. |
| `discover_cameras()` | Etsi LATTICE-kamerat **tämän instanssin taustaprosessin kautta** (`/api/camera/discover`). Palauttaa sanakirjojen luettelon (`serial`, `model`, `ip`, …) — sama rakenne kuin GUI:ssa ja CLI:ssä. Tyhjä luettelo, jos kameroita ei löydy tai taustapalvelimeen ei saada yhteyttä. |
| `camera_capture(output_dir, format="tiff", **settings)` | Kaappaa yhden kehyksen**taustapalvelimen kautta**(tämän kahvan avulla automaattisesti käynnistettynä), jolloin se saa saman esikäsittelyn kuin GUI:ssa tai CLI:ssä (oletusarvoisesti 12-bittinen, poolin uudelleenkäyttö, upotetut kalibrointimetatiedot). Määritä kohde komennolla `serial=` tai `device_index=`; välitä `exposure`/`gain`/`pixel_format`/`preset` nimellä `**settings`. Palauttaa vanhan metatietosanakirjan (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Tuottaa päällekkäin yhdistettyjä esikatselukuvia yhdistetyltä kameralta — kevyt MJPEG-asiakasohjelma taustapalvelimen `/api/camera/<serial>/stream-annotated`-reitityksen kautta (zebra / ruudukko / hiusristikko / histogrammi / peaking / piste piirretään palvelinpuolella). `decode=True` tuottaa BGR-taulukoita ja vaatii OpenCV:n (`cv2`), jota SDK ei asenna; `False` tuottaa raakoja JPEG-tavuja. Saatavilla myös projektikohtaisesti nimellä `ChlorosProject.stream(overlays=True)`. |

**Käynnissä oleva suoritus säilyttää projektinsa.** Kun käsittelyajo on käynnissä — olipa kyseessä sitten tämän skriptin, sovelluksen, `chloros-cli process`:n tai `chloros-mcp`:n ajo — taustapalvelu kieltäytyy luomasta, avaamasta, tuomasta tai muuttamasta projektin käsittelyasetuksia (HTTP-virhe 409 `PROCESSING_IN_PROGRESS`): `create_project()` ja `import_images()` aiheuttavat virheen `ChlorosProcessingError`, ja `configure()` aiheuttaa virheen `ChlorosConfigurationError`, ja `process()`, jonka tila eroaa käynnissä olevasta, aiheuttaa virheen `ChlorosProcessingError` (ajo säilyttää sen tilan, jossa se alkoi), kukin backendin viestin kera. Projektiin tallennettu kaappaus liitetään siihen, kun ajo päättyy (sen vastauksen `project_add` kertoo niin). Odota, kunnes suoritus päättyy (`get_status()`), ja jatka sitten.**Siirretty projekti.** Projektin `project.json` nimeää jokaisen lähdetiedoston (jokaisen kuvan ja valosensorin tallenteen) sen täydellisen polun mukaan, joten projektikansion – tai koko projektikansion – siirtäminen toiselle asemalle jättää tiedostojen nimet viittaamaan vanhaan sijaintiin. Jokainen avaus korjaa sen, minkä pystyy itse: projektikansiossa aiemmin ollutta tiedostoa etsitään kansiosta, jossa projekti nyt sijaitsee, ja jokainen sieltä löytyvä polku osoitetaan uudelleen ja tallennetaan (`project.json` varmuuskopioidaan ensin nimellä `project.json.before-relink-<date>-<time>`). Polku osoitetaan uudelleen vain tiedostoon, joka sijaitsee kyseisessä paikassa. Tiedostot, jotka olivat projektikansion ulkopuolella – esimerkiksi Hub-istunto tai Add Folder -tuonti – löytyvät vain kansiosta `relink=<the folder they are in now>` (tai sen yläpuolella olevista kansioista). `open_project()` palauttaa `source_files`: `total`, `found`, `missing`, `missing_roots` (kansiot, joissa puuttuvat tiedostot olivat), `missing_outside_project` ja `relink` (mitä osoitettiin uudelleen, mistä minne). Projekti, josta puuttuu joitakin tiedostoja, avautuu muiden tiedostojen kanssa; projektia, josta ei löydy yhtään tiedostoa, ei avata, ja `open_project()` aiheuttaa virheen `ChlorosProjectFilesMissingError` (`.missing_roots`, `.audit`; HTTP 409 `PROJECT_SOURCES_MISSING`). CLI:n virheet `project open --relink` ja `chloros-mcp``open_project` (`relink_folder`) toimivat samalla tavalla.

Käytä kontekstinhallintana taatun puhdistuksen varmistamiseksi:

```python
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

### Valosensorin tallenteet — kalibroitu `.daq` + `.csv`

DAQ-U / DAQ-M / DAQ-E voidaan tallentaa **ilman** sen kalibrointipakettia. Juuri
niin julkiset [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
-tallentimet (`record_daq.py`) tekevät oletuksena: ne tallentavat anturien raakalukemat ja merkitsevät
tiedostoon leiman, jotta Chloros hakee kyseisen anturin tehdaskalibroinnin **sarjanumeron perusteella** — ensin paikallisesta välimuistista
ensin, sitten MAPIR-pilvestä — ja soveltaa sitä tuonnin yhteydessä.

Chloros tallentaa tuloksen takaisin kahdeksi tuotteeksi kutakin tallennusta kohti, nimellä
`<project>/Light Sensor/`:

| Tuote | Mikä se on |
| --- | --- |
| `<name>_calibrated.daq` | Uudelleenkäsiteltävä arkisto — sama rakenne kuin reaaliaikaisessa tallenteessa, mutta nyt siinä ilmoitetaan sen tuottanut paketti. Sen uudelleen tuominen **ei** kalibroi sitä toista kertaa. |
| `<name>_calibrated.csv` | Spektrinen säteilyvoimakkuus yksikössä W/m²/nm anturin omalla aallonpituusruudukolla, yksi rivi lukemaa kohti, sekä fotometriset sarakkeet (kokonaisteho, fotopinen/skotopinen lux, PPFD ja sen sininen/vihreä/punainen jakautuma, huippuaallonpituus). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Vain ilman pakettia toimivat anturit (DAQ-A).** Anturin raakaspektrilukemat — *ei* säteilyvoimakkuutta. Katso alla. |

`<name>` on tallennuksen oma tiedostonimi. Kaksi eri kansioissa olevaa tallennusta voi jakaa
yhden — tallennuskohtaisen lukeman (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`), joka on tallennettu
samana millisekuntina kahdella kameralla — ja kumpikin saa silti omat tuotteensa: ensimmäinen säilyttää
`<name>`, seuraava lisää kansiot, jotka erottavat sen muista, **nimen jälkeen**
(`<name>__<capture>_daq_calibrated.daq`), joten tallenteen päivämäärä ja kellonaika ovat edelleen etusijalla.

`process()` suorittaa tämän viennin yhtenä vaiheistaan. Se **ei** vaadi kuvamateriaalia:
yksinään lennetty valosensori on ensiluokkainen työnkulku, ja tällaisessa projektissa ei ole
rakenteellisesti yhtään kuvaa.

**DAQ-A-tallenteet viedään raakalukuna.** DAQ-A-tuoteperhe on peräisin ajalta ennen sarjanumeroittain
toimivaa niputusjärjestelmää, eikä sille ole nippua haettavaksi — se kalibroidaan kentällä
heijastavuuskohteen avulla, minkä vuoksi sille ei ole koskaan tarvittu nippua. Nämä tallenteet viedään
tiedostotunnisteella `_raw` eikä `_calibrated`: kyseessä on erilainen tiedostonimi eikä tiedoston sisällä oleva merkki
, koska tiedostonimen on säilyttävä sellaisenaan, kun se lähetetään sähköpostitse pelkänä nimenä.
`.csv`-otsikossa lukee `raw spectral sensor counts (NOT irradiance)` ja varoitetaan, että
arvot ovat vertailukelpoisia **tiedoston sisällä** — juuri sitä varten kohdepohjainen kalibrointi niitä käyttää
niitä varten — eikä anturien välillä. Teho-riippuvaiset fotometriset sarakkeet (kokonaisteho,
fotopinen/skotopinen lux, PPFD) palauttavat arvon **NULL** sen sijaan, että ne integroitaisiin laskentojen perusteella.

DAQ-U / DAQ-M / DAQ-E, jonka pakettia ei yksinkertaisesti voitu hakea, **ohitetaan** edelleen,
eikä sitä kirjoiteta raakamuodossa: siinä tapauksessa paketti on olemassa ja ”yhdistä uudelleen ja käsittele uudelleen” on todellinen neuvo.

Vanhat **v1.01 / v1.02**-tallenteet (joita DAQ-A-SD kirjoittaa) eivät sisällä lukukohtaista aikamerkkiä,
vaan ainoastaan tiedoston kirjoitusajan. Kuva↔alasuuntaisen valon vertailija hylkää ne edelleen —
kehyksen täsmäyttäminen kirjoitusajankohtaan olisi huomaamattomasti virheellistä — mutta vientiohjelma lukee ne, ja
tiedostossa CSV tulostetaan `clock=daq_created_on`, joten tuote ilmoittaa, minkä kellon mukaan se toimii.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Tallenne, jonka kalibrointipakettia ei voida hakea (offline-tilassa tai anturi, jolle ei ole
tiedostossa kalibrointia), raportoidaan kohdassa `skipped` **syyn kera**. Sitä ei koskaan
tallenneta ”kalibroituna” tiedostona, joka sisältää raakalukemia — muodosta yhteys internetiin ja
suorita komento uudelleen, jolloin vienti valmistuu.

### Edistymisen takaisinsoitot

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Suorituksen jälkeinen yhteenveto ja vinkit

Kun vienti on valmis, `process()` hakee `GET /api/processing-summary`:n ja liittää sen sisällön tiedostona `result["summary"]`. Hakua yritetään parhaansa mukaan, eikä se koskaan estä onnistunutta palautusta — jos yhteenvetoa ei ole saatavilla, `process()` siirtyy takaisin tavalliseen `{"status": "complete", "async": False}`-muotoon. Jokainen `summary["hints"]`:n merkintä — kokonaiset lauseet, joissa on ehdotettu korjaus, esim. miksi suoritus tuotti nollatuloksen — lähetetään uudelleen myös Python-muodossa `UserWarning`, joten nollatulokselliset suoritukset ovat itsediagnosoivia, vaikka et koskaan tarkastelisi sanakirjaa:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` on koneellisesti luettava puoli:

| Avain | Mitä se laskee |
| --- | --- |
| `models` | Ajon kameraryhmät. |
| `images_in_groups` | Näiden ryhmien lähdekuvat. |
| `targets_found` | Havaittuja heijastavuuskohteita. |
| `images_calibrated` | Kuvat, joista ajossa luotiin heijastavuustuote. |
| `images_failed` | Kuvat, jotka epäonnistuivat eikä niitä viety ulos. |
| `exported_files` | **Kuvatiedostot , jotka ajon tuloksena syntyivät**: kaikki tuotteet (debayeroitu, esikatselu, säteilyvoimakkuus, heijastavuus, indeksi, kohderenderöinnit) kaikissa muodoissa. |
| `reflectance_files` | Niiden joukossa olevat heijastavuustuotteet (`Reflectance_Calibrated_Images`), riippumatta siitä, onko ne kalibroitu kohteen vai valosensorin (DAQ) säteilyvoimakkuuden perusteella. Jos ajossa ei ole kohdetta, mutta heijastavuustiedot on silti tallennettu, tästä ilmoitetaan vihjeessä; vain ajosta, jossa heijastavuustuloksia ei ole tallennettu, ilmoitetaan, että ne puuttuvat. |
| `clamp_masks_written` | [Unmix-rajoitusmaskit](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) on merkitty niiden viereen — ne ovat liitännäisiä, eivät tuotteita, joten ne eivät sisälly `exported_files`:ään (CLI-käyttöliittymä laskee ne erikseen samalla tavalla). |
| `captures_left_out` | Projektin `captures`-kansiossa olevat raakakuvat, joita tämä ajo ei ottanut. Suoritus odottaa käynnistyessään sitä, mitä on tallennettu ennen sitä, joten tämä on 0, ellei tallennusta ole tallennettu suorituksen aikana tai odotusaika ole päättynyt. `summary["captures_left_out_note"]` kertoo, mitä niille tehdään. |
| `daq_recordings_exported` / `daq_recordings_skipped` | Valosensorin tallenteet, jotka on laskettu tarkoituksella erikseen — ne ovat peräisin eri vaiheesta ja esiintyvät ajoissa, joissa ei ole lainkaan kuvamateriaalia, joten niiden sisällyttäminen saisi pelkän DAQ-ajon näyttämään siltä, kuin se olisi vienyt kuvia. |

Niiden lisäksi: `summary["output_dirs"]` (jokainen hakemisto, johon on kirjoitettu),
`summary["light_sensor_export"]`, `summary["stopped"]` (tosi, kun käyttäjä keskeytti
ajon, jotta osittaiset laskelmat eivät näyttäisi suoritukselta, joka on jäänyt vajaaksi), ja
`summary["groups"]` (ryhmäkohtainen erittely).

`exported_files` tallennetaan putkistossa **kirjoituksen yhteydessä**, eikä sitä skannata
projektin kuvaobjekteista jälkikäteen. Rinnakkais- ja GPU-strategiat rakentavat omat kuvaobjektinsa
(GPU-polkujen osalta työntekijöiden aliprosesseissa), joten vanha skannaus raportoi
`0 file(s) written` jokaisesta tällaisesta ajosta ja lähetti sitten nollaviennin vihjeen — ajoissa,
joissa kaikki oli toiminut. Jos kirjoitat skriptiä tämän numeron perusteella, toimiva rinnakkaiskäyttö
ilmoittaa nyt nollasta poikkeavan lukeman.

Ohitusraportit ilmoittavat syyn, jonka lukija tosiasiallisesti määritteli jokaiselle tiedostolle —
lukukelvoton skeema, puuttuva nippu, kirjoitusvirhe — **duplikaatioita poistettuna**, joten kaksikymmentä tiedostoa,
jotka ohitettiin yhden syyn vuoksi, näkyvät yhtenä syynä sen sijaan, että sama syy toistuu kaksikymmentä kertaa.

> **`process()` ei aiheuta poikkeusta, kun suoritus ei tuota kuvia.** Tämä on yksi kahdesta kohdasta, joissa SDK ja
> CLI eroavat toisistaan tarkoituksellisesti: `chloros-cli process` käsittelee tilannetta ”tuotteita pyydettiin, mutta yhtään ei
> kirjoitettu” virheenä ja lopettaa nollasta poikkeavalla tuloksella, kun taas SDK palaa normaalisti ja raportoi
> tilan `summary`:n / vihjeiden kautta. Jos prosessiketjusi pitäisi pysähtyä tyhjän ajon yhteydessä, tarkista se
> itse — tarkista `summary` (tai laske projektikansiossa olevien tiedostojen määrä) sen sijaan, että luottaisit
> poikkeuksen puuttumiseen. Yleisiä syitä ovat syöttökansio, jota ei tunnistettu
> tallennukseksi, sekä tuotteet, jotka ohitettiin, koska niitä ei voitu soveltaa käytössä oleviin kameroihin (esim. säteilyarvot pelkästään RGB-
> kameroista).
>
> `process()` ei myöskään laukea, kun ajon vientiä ei voitu tarkistaa: se antaa varoituksen ja asettaa
> `result["export_check_failure"]`. Tuotteet tallennetaan. Jos haluat pysäyttää prosessin tässä vaiheessa, tarkista kyseinen avain. Sama pätee ajokertaan, jossa
> kuva on kadonnut: se antaa varoituksen ja asettaa `result["image_failure"]`-asetuksen, jonka ollessa nollasta poikkeava
> `chloros-cli process` lopettaa ajon.

### Apufunktiot

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "GNDVI", "OSAVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# process_folder() / process_lattice_capture() also take input_level=,
# array_alignment=, array_alignment_crop=, array_alignment_interpolation=,
# progress_callback=, poll_interval=, timeout=, processing_stuck_timeout=,
# backend_exe= and api_url= (NOT backend_url=). indices defaults to ["NDVI"].

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

Jokainen suoritus toimii omassa `with ChlorosLocal(...)`-funktiossaan; sen käynnistämä taustaprosessi jatkaa toimintaansa, kun se palaa, seuraavaa kutsua varten, ja pysähtyy itsestään 30 minuutin kuluttua, jos mikään ei käytä sitä. Kumpikaan ei ota vastaan `auto_start_backend`-, `radiometric_output`- tai `daq_log_path`-parametreja; käytä niiden sijaan `ChlorosLocal` ja `configure()` näiden tilanteiden varalta. `process_lattice_capture()` ei hyväksy `camera`-, `debayer`- tai `ppk`-arvoja: se ohittaa paneelin tunnistuksen ja käyttää vakiomuunnosta.

### Tuetut arvot

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"TIFF (8-bit)"            # all-8-bit images are written as this; never raised
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer for Survey3 imagery, Chloros+ only.
                                         # Survey3 only: on a project holding LATTICE captures
                                         # (which always use the standard demosaic) configure()
                                         # or process() raises ChlorosConfigurationError; a
                                         # Texture Aware value the project saved earlier is set
                                         # to Standard with a UserWarning, as the app does. Needs a
                                         # CUDA GPU the backend's PyTorch can use (an NVIDIA
                                         # GPU, or a Jetson Orin's). Without one, process()
                                         # uses the standard debayer, and the backend prints
                                         # one "[DEBAYER] ... needs a CUDA GPU" line saying why.
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as Chloros's saved debayered (3-channel) product
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Radiometrinen lähtö (LATTICE-monispektrinen prosessiketju)

`process`-putkiston LATTICE-monispektrinen (M3C/M3M) vientitaso — `reflectance` (oletus), `radiance`, `sensor-response` tai `all` (kaikki kuvakohtaiset soveltuvat tilat) — määrittää projektin **”Vie säteilyvoimakkuus”**- ja**”Vie heijastavuus”**** tuotteet: `radiance` vie vain säteilyvoimakkuuden, `reflectance` vain heijastavuuden, `sensor-response` ei kumpaakaan (vain debayeroidut ja esikatselutuotteet), `all` molemmat. Debayeroituja ja esikatselutuotteita ei voi muuttaa tällä tasolla. `configure()`:llä on oma avainsanansa:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Edistynyt kiertotapa — tuotteiden kirjoittaminen `custom_settings`:n kautta — toimii edelleen, mutta muista, että se korvaa koko asetuslohkon (katso alla oleva varoitus). Kirjoita avaimet `"Export <product>"`: projektin omat asetukset `"Export radiance"` / `"Export reflectance"`-asetukset ohittavat vanhemman `"Radiometric output"`-avaimen, joten kyseinen avain yksinään ei muuta mitään.

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Export radiance": True, "Export reflectance": False},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (oletus) jakaa kameran säteilyvoimakkuuden **aikaleimalla täsmätyllä DAQ-alasvirtauksella**, joka määritetään automaattisesti tallennetusta `.daq` (DAQ-U/M/E)**tai kuvamateriaalin yhteydessä löytyvästä DAQ-M-natiivisesta `.csv`**-tuotteesta; mahdolliset paikallisesti puuttuvat kamera- tai DAQ-kalibrointipaketit**haetaan automaattisesti AWS:stä** ensimmäisellä käyttökerralla. CLI-rajapinta tarjoaa tämäntyyppisinä tuotevalintoina `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.**Kenen lukema se on.**`.daq`-tallennus sisältää sen kuvaaman kameran (kameroiden) lukeman — joka on nimetty sen vieressä olevassa `<file>.daq.cameras.json`-sidecar-tiedostossa (yksittäiset, synkronoidut ja taulukkomuotoiset tallenteet; raakasarjatallenteessa kunkin jäsenen lukema on nimetty sen manifestissa) — ja se täsmää**vain kyseisten kameroiden kuviin**. Itsenäinen kamera, jolla ei ole omaa lukemaa, ei siis enää lainaa toisen kameran DAQ:ta; liitä sille DAQ tai tallenna sellainen. DAQ:n oma tallenne (`chloros-cli daq pool-record` tai GUI:ssa aloitettu tallennus) tai mikä tahansa `.daq` ilman kyseistä sivutiedostoa (mukaan lukien vanhemmat tallenteet) palvelee edelleen jokaista kameraa.**Auringonvalossa olevat OCN-kamerat.** Jokainen OCN-kalibrointipaketti on rakennettu kapeakaistaisessa valossa, yksi LED kaistalta, kun taas auringonvalo sekoittaa kaistat. Chloros 1.3.0 soveltaa laivueen auringonvaloon perustuvaa korjausta kaistojen erottelun jälkeen, mikä parantaa OCN-säteilyvoimakkuus-, heijastavuus- ja indeksituotteita auringonvalossa. Se toimii kaikkialla, missä kaistojen erottelu tapahtuu — `process()`, kuvat, `grab()` / `frame_stream()` ja reaaliaikainen näkymä — niissä OCN-paketeissa, jotka tukevat sitä; RGN-kameroissa ei tapahdu muutoksia. Aikaisemmalla versiolla tallennetut OCN-kuvat saavat tämän korjauksen myös, kun niitä käsitellään tietokoneella, jossa on tallennettuna niiden ottamishetkellä käytetty kalibrointi (Chloros säilyttää kameran kalibroinnin, kun kamera on kytketty); muussa tapauksessa ne käsitellään kuten aiemmin. OCN-säteily-, heijastus- ja indeksiarvot eroavat siten versiosta 1.2.0.

> `custom_settings` **korvaa** koko lasketun asetuslohkon (se ohittaa suunnittelunsa mukaisesti `configure()`:n muut avainsanat ja validoinnin). Kun käytät sitä, sisällytä kaikki sinulle tärkeät `Project Settings`-avaimet, kuten yllä olevassa esimerkissä.

#### Indeksipikselien lukeminen (NDVI ja muut kasvillisuusindeksit)

**Kokonaislukuiset indeksirasterit ovat kuvakohtaisia kontrastin venytyksiä, eivät −1…1-karttoja.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` ja `JPG (8-bit)` -indeksituotteet — mukaan lukien 8-bittinen TIFF-tiedosto, johon 8-bittinen kuvaus tallennetaan — tallentavat

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

jossa `IndexMin` / `IndexMax` ovat **kyseisen kehyksen omat pienin ja suurin äärelliset indeksiarvot**. DN 0 ei ole −1, eikä täysi asteikko ole +1.

Chloros merkitsee molemmat päätepisteet, jotta kuvaus pysyy käännettävissä:

| Tunniste | Merkitys |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — alla oleva käytäntö |
| `Chloros:IndexMin` | DN 0:na tallennettu indeksiarvo |
| `Chloros:IndexMax` | indeksiarvo, joka on tallennettu täysimittaisena |

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

> **`IndexMin == IndexMax`** tarkoittaa, että kyseisen kehyksen jokaisella pikselillä oli sama indeksiarvo. Rasteri on täytetty keskitasolla, eikä sitä ole mitään, millä jakaa — ota päätepiste.

> **Jos `Chloros:Index*`-tunnisteita ei ole, ei ole määriteltyä kartoitusta.** LUT-vienti on värillinen renderointi, jonka pikselit ovat LUT-kynnysikkunaan sidottuja gradienttivärejä, ja kehyksellä, jossa ei ole yhtään äärellisen indeksin pikseliä, ei ole määriteltävää aluetta. Älä oleta kartoitusta kummassakaan tapauksessa — sama sääntö kuin puuttuvan `PixelScale`-tunnisteen kohdalla.**Koska venytys on kuvakohtainen, yhden lennon kahta kuvaa ei voi verrata toisiinsa DN-arvojen perusteella**— kummallakin on omat päätepisteensä, joten sama NDVI-arvo vastaa eri DN-arvoja eri kuvissa. Lennon laajuista kvantitatiivista työtä varten (vyöhykekohtaiset tilastot, kiinteät kynnysarvot, aikasarjat) vie**`TIFF (32-bit, Percent)`**: indeksirasterit tallennetaan sinne float32-muotoisina**indeksiarvoina**, ilman skaalausta tai tarvittavia tunnisteita. Muodosta huolimatta indeksituotteet eivät ole ×100 – kyseinen skaalaus koskee vain kokonaislukutyyppistä heijastavuutta.

#### Unmix-rajoitusmaski — mihin pikseleihin kaista on rajoitettu

3×3-spektrinen unmix-laskenta voi muuttaa kaistan negatiiviseksi — jos valossa on tuskin kyseistä kaistaa tai jos kohde ei kuulu kameran unmix-matriisin kuvaaman alueen sisälle. Se ei ole todellinen arvo, joten vienti asettaa sen arvoon 0 — ja reaaliaikaisessa näkymässä näkyy sama 0. Kaista voidaan rajoittaa muutamalle pikselille tai lähes kaikille; maski tallentaa kummankin tapauksen. `Chloros:Band{R,G,B}UnmixClampFraction` kertoo, kuinka suuri osa kehyksestä oli kyseessä; rajoitusmaski kertoo **mitkä pikselit**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— tuotteen kansion**vieressä** olevassa `clamp_masks/`-kansiossa, ei koskaan sen sisällä (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`) — jokaiselle säteily-, heijastus- ja indeksituotteelle, jossa vähintään yksi pikseli on rajattu. Yksikaistainen `uint8`, tuotteen omalla pikseliristikolla (suunta, ROI, binning ja matriisin kohdistuksen vääristymä mukaan lukien). Jokainen maski nimeää tuotteensa kameran (`Make` / `Model` / sarjanumero), tallennusajan ja objektiivitunnisteet sekä luettelee kaistat, joita sen bitit edustavat (`XMP-Camera:BandName`, bitti 0 ensin); sen `ImageDescription` ja `Software` merkitsevät sen rajoitusmaskiksi, eikä se sisällä kalibrointia.
- **Bitit**: bitti 0 = tuotteen R-kanavan kaista (`Chloros:BandRName`), bitti 1 = G (`BandGName`), bitti 2 = B (`BandBName`). Asetettu bitti tarkoittaa, että kyseisen kaistan pikselin arvo on rajoitettu arvoon 0 — käsittele sitä ja siitä muodostettuja indeksejä luotettavuudeltaan heikkoina.
- **`Chloros:UnmixClampMask`** on merkitty jokaiseen tuotteeseen, johon on suoritettu erottelu: maskin polku suhteessa tuotteen omaan kansioon (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, aina `/`-erotettuna — liitä se kuvan kansioon avataksesi maskin), tai `none`, kun yhtään pikseliä ei ole rajoitettu. Tunnisteen puuttuminen tarkoittaa, että tuotteelle ei ole koskaan suoritettu unmix-käsittelyä (RGB- tai monokamerat, raaka-/debayeroidut tasot).
- Tallennustuloksissa se näkyy kehyksen **`clamp_mask`**-kentässä (maskin polku, tai `None`, kun mitään ei ole rajoitettu) — `CameraSession.capture()`-tuloksissa, `ArraySession.capture()`&#x27;`CaptureResult`-kehyksissä ja HTTP-kaappausvastauksissa. Offline-rakennus listaa sen maskit nimellä `clamp_masks`.
- Live-indeksin monikaistapino (`/api/camera/cli-array-index` ja `save_multiband`) saa yhden maskin kutakin lähdekameraa kohti, tulostuskansion omassa `clamp_masks/`-kansiossa — katso CLI-ohjeen`lattice index`. Videolla ei ole maskia: offline-rakennuksen tulos raportoi sen sijaan kunkin kehyksen rajoitusosuuden.

Koska missään syvyydessä ei ole tuotekansiota, jossa olisi maskia, tuotekansio voidaan luovuttaa fotogrammetriaohjelmistolle sellaisenaan — alikansiot mukaan lukien — ja se ottaa vastaan vain kuvia. Luovuta sille tuotekansio, ei sen yläpuolella olevaa kansiota. Chlorosin oma tuontitoiminto ohittaa maskit.

```python
import os, numpy as np, tifffile

mask = tifffile.imread(
    "clamp_masks/radiance/capture_<ts>_SN<serial>-<filter>_radiance_clampmask.tif")
# or, from the image: os.path.join(os.path.dirname(image_path), tag), with
# tag = the image's Chloros:UnmixClampMask
r_clamped = (mask & 1) != 0      # the band Chloros:BandRName names
g_clamped = (mask & 2) != 0
b_clamped = (mask & 4) != 0
```

---

## Smart-Connect LATTICE-kameroille

Pysyvät taustaprosessisessiot reaaliaikaiselle laitteistolle. Samat päätepisteet kuin GUI:ssa, joten toiminta on identtinen SDK:n, CLI:n ja GUI:n välillä.

### Yksi kamera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

Tässä asetetut valotus, vahvistus ja kohde säilytetään käyttäjän asetuksina: Chlorosin automaattinen valotuksen säätö (mukaan lukien DAQ:n mittaama valotus) ei muuta niitä. `cam.set_settings(exposure_auto="Continuous")` (tai `gain_auto=`) palauttaa valotuksen (vahvistuksen), ja `cam.set_settings(target_brightness_auto=True)` palauttaa kohteen Chlorosin automaattiselle kohteenvalinnalle.

#### `connect_camera()` Allekirjoitus

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

- **Se palaa, kun lähetys on alkanut.** Taustapalvelu odottaa, rajoitetusti (noin 15 sekuntia, kunnes virta alkaa, ja sitten 10 sekuntia, kunnes ensimmäinen täydellinen puskuri on valmis — ei ensimmäistä esikatselukehystä, joka seuraa noin 1,5 sekuntia myöhemmin), joten `status()` tulee heti sen jälkeen, kun se ilmoittaa `streaming`. Istunnon `connect_stream` on vastauksen `stream`-lohko: `state` `streaming`, `no_frames` (virta alkoi, mutta ei toimittanut mitään — `note` kertoo tämän ja, kun kyseessä on jumbo-paketteja käyttävä kamera, kehottaa muodostamaan yhteyden uudelleen 1500 tavun pakettikoolla), `starting` tai `start_failed`, jossa `waited_s`. `None` kameralle, joka oli jo yhteydessä. Esikatselu (`get_camera_preview`, `/api/camera/jpeg/<serial>`), jota pyydetään heti odotuksen jälkeen, odottaa rajoitetun ajan (4 s) kameran ensimmäistä esikatselukehystä sen sijaan, että vastaisi ”No frame available yet”; kamera, jonka virta ei ole toimittanut mitään (`no_frames` tai laitteistolähtöinen), vastaa edelleen välittömästi.
- **Paketin koko on sinun määritettävissäsi.** `settings={"gev_packet_size": 1500, "gev_packet_size_auto": False}` avaa kameran 1500 tavun paketeilla, joita ei testata (`chloros-cli lattice cam-connect --packet-size standard` lähettää saman).
- **Kamerapoolia voi käyttää vain yksi yhteys kerrallaan**: avaus, pakettikoon testaus, ensimmäinen kalibrointitiedoston lataus (noin 70 s, jos kamera on uusi tällä tietokoneella) sekä videovirran aloitus. Pyyntö, joka tarvitsee kameran sillä välin (`read_nodes`, `set_settings`, tallennus, toinen yhteys) odottaa sitä enintään 5 sekuntia, minkä jälkeen se epäonnistuu HTTP-virheellä 503 (`CAMERA_POOL_BUSY`), mainiten yhdistävän kameran nimen (`ChlorosConnectError`); kysy hetken kuluttua uudelleen. Luettelot eivät koskaan odota: `list_cameras()` (`GET /api/camera/pool`), `list_arrays()` ja `CameraSession.status()` vastaavat välittömästi. Yhteyden muodostamisen keskellä oleva kamera näkyy luettelossa poolin vastauksen alla: `connecting` (`serial`, `phase` — `opening the camera`, `rebalancing the pool bandwidth`, `loading its calibration`, `starting its stream`, `waiting for its stream` — ja `elapsed_s`), ja sen `status()` ilmoittaa `connecting: True`:n, jonka `phase`; muut kamerat luetaan reaaliaikaisesti, kun resurssipooli on vapaa, muussa tapauksessa vastaus perustuu niiden viimeisimpään lukemaan (`live: False`).

#### `CameraSession`-menetelmät

| Menetelmä | Kuvaus |
| --- | --- |
| `status(timeout=15.0)` | Kameran reaaliaikainen tila, sellaisena kuin `chloros-cli lattice status` sen lukee: `streaming`, `stream_state` (`starting`, `no_frames`, `streaming`, `start_failed` tai `stopped`), `device` (malli, sarjanumero, laiteohjelmisto, lämpötila, `is_mono`) ja sen nykyiset asetukset. |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Lukee GenICam-solmuja; palauttaa `{nodes, errors, enums, device, was_in_pool, in_array_id}`, sekä `frame_values` ja `notes` automaattisen valotuksen ollessa käynnissä (katso alla). |
| `set_settings(**kwargs)` | Kirjoittaa solmuja ystävällisellä nimellä (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `target_brightness_auto`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). `exposure_time` (`gain`) on manuaalinen valotus (vahvistus): kirjoitettaessa ilman `exposure_auto` (`gain_auto`) ilman, että automaattitila on ensin kytketty pois päältä, jolloin käynnissä oleva automaattitila ei hylkää arvoa. Tuloksena oleva `applied` sisältää kameran ottaman arvon; kirjoitus, jonka kamera hylkää, on tallennettu `errors`:ään eikä sitä säilytetä. Virhe määrittää alueen, jonka kamera nyt hyväksyy (`ExposureTime`:n enimmäisarvo noudattaa kameran kuvataajuutta), ja automaattitila, joka kytkettiin pois päältä vain hylätyn arvon vuoksi, palautetaan; `restored` luettelee ne (esim. `{"ExposureAuto": "Continuous"}`). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, daq=None, daq_tolerance_ms=None, settings=None, timeout=None)` | Tallenna **yksi** kuvan. Palauttaa `CaptureResult`: luettelon kuvien metatietosanakirjoista (yksi tallennettua tasoa kohti), joissa on `.skipped`, `.failed`, `.daq_files` ja `.daq_matches`. (Sarja- ja monikehyksinen tallennus on poistettu — kutsu `capture()` silmukassa, jos tarvitset sarjaa.) |
| `capture_repeated(output_dir="output", *, count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Peräkkäiset (`interval_s=0`) tai aikavälein tapahtuvat kuvaukset tällä kameralla, kun `count`-kaappauksia ja/tai `duration_s`-kaappauksia on vähintään yksi (jotta se päättyy), kuten `ArraySession.capture_repeated()` tekee taulukon kohdalla. Palauttaa luettelon `CaptureResult`-tiedostoista; se pysähtyy, kun tallennus ilmoittaa levyn olevan täynnä. `on_capture(i, result)` suoritetaan jokaisen jälkeen; muut avainsanat menevät `capture()`:ään. |
| `disconnect(keep_settings=False, clear_settings=False)` | Vapautus poolista, kamera palautetaan tilaan, jossa se oli yhteyden muodostamisen yhteydessä (laukaisijajohtimet, PTP, asetukset); `keep_settings=True` tallentaa asetuksensa sen sijaan omaan muistiinsa, `clear_settings=True` nollaa kyseisen muistin (katso [Kameran muisti](#camera-memory--keep_settings--clear_settings)); palauttaa `camera_memory`:n, kun jompaakumpaa pyydetään. Kameran, joka on taulukon jäsen, vapauttaa koko taulukon. Ei tee mitään, jos liityimme jo avoimeen istuntoon. Palauttaa tuloksen, kun taustapalvelu vahvistaa vapautuksen tai kun vapautettavaa ei ole (kamera on jo poistettu tai taustapalvelua ei ole käynnissä); aikakatkaistunut vapautus tai taustapalvelimen hylkäämä vapautus aiheuttaa virheen `ChlorosConnectError`, koska kamera saattaa olla edelleen varattuna ja seuraava yhteysyritys joutuisi ristiriitaan sen kanssa. `with`-lohkon poistuminen ei koskaan aiheuta virhettä. |

Attribuutit: `serial`, `device` (kameran löytötietue) ja `backend_url`.

`capture()`-vientivalvonta (sama malli kuin taulukko + käyttöliittymä):

- `processing` / `levels` — `processing="all"` tallentaa kaikki soveltuvat vientityypit; `levels=["raw","radiance"]` tallentaa vain ne (ohittaa `processing`:n). Jätä molemmat pois, jos haluat käyttää taustapalvelimen oletusasetusta.
- `daq` — käytettyä kaappausta lukeva DAQ tallennetaan oletuksena `.daq`-sidecarina, johon sisältyy pelkkä raakadata, jotta kehys voidaan myöhemmin käsitellä uudelleen heijastavuudeksi/indeksiksi. `daq=False` ohittaa sen, jos mikään taso ei käytä sitä. `force_daq=True` on oletusasetuksen vanhempi kirjoitusmuoto. Ei DAQ:ta, ei `.daq`.
- `daq_tolerance_ms` — kuinka kaukana valotuksesta lukema voi olla ja silti olla toleranssin sisällä (oletus 500). Vain tulos muuttuu.

#### `read_nodes()` automaattisessa valotuksessa — `frame_values` ja `notes`

Kun `ExposureAuto` (`GainAuto`) on käynnissä, `ExposureTime` (`Gain`) -solmu lukee takaisin **viimeksi kirjoitetun arvon**, ei sitä, mitä kamera parhaillaan tekee. `nodes` säilyttää edelleen kyseisen solmun arvon. Kun kysyt `ExposureTime`- tai `Gain`-arvoa liitetyltä kameralta, jonka automaattitila ei ole `Off`, vastaukseen lisätään:

- `frame_values` — kameran uusimman kehyksen sisältämä arvo solmun nimen alla (`ExposureTime` mikrosekunteina, `Gain` desibeleinä);
- `notes` — yksi lause kutakin tällaista solmua kohti, jossa mainitaan molemmat arvot.

```python
r = cam.read_nodes(["ExposureTime", "Gain"])
print(r["nodes"]["ExposureTime"])                  # last value written, e.g. 2000.448
print(r.get("frame_values", {}).get("ExposureTime"))  # what the newest frame used, e.g. 19600.0
for name, note in r.get("notes", {}).items():
    print(note)
# ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448);
# the camera's newest frame was at 19600 us.
```

Molemmat avaimet puuttuvat, kun automaattitila on `Off` (solmu on tällöin kamerantodellinen arvo) ja kameralle, jota ei ole kytketty taustapalvelimeen. Taulukon tapauksessa `ArraySession.status()["member_ae"]` antaa jokaisen jäsenen kehysarvot kerralla.

#### DAQ-mitattu valotuksen yläraja (HTTP)

Kun DAQ-valosensori on kytketty, Chloros rajoittaa kameran valotusta ja vahvistusta mitatun päivänvalon perusteella, jotta valkoinen vertailupaneeli ei leikkaudu. Yläraja on voimassa vain niin kauan kuin kalibrointikohde on kyseisen kameran näkökentässä (näkyy sen kuvassa viimeisten 10 sekunnin aikana); jos kohdetta ei ole näkökentässä, kameran tavallinen automaattinen valotus, jossa on oma kirkkaiden alueiden suojaus, toimii sen sijaan, jotta muuta kuvaa ei pidetä tarpeettoman tummana. SDK-kääreohjelmaa ei ole; lue tai vaihda asetusta kamerakohtaisesti komentoilla `GET` / `POST /api/camera/<serial>/daq-exposure-cap` (POST-viestin runko `{"enabled": false}` poistaa sen käytöstä kyseisessä kamerassa). Vastauksessa on mukana `enabled`, `cap_us` (reaaliaikainen yläraja, `None` kun toiminto on pois päältä, myös silloin kun kalibrointikohdetta ei ole näkymässä), `k_exp_g` (vakio, jolla yläraja lasketaan) sekä:

| Kenttä | Merkitys |
| --- | --- |
| `k_exp_g_source` | Kenen vakio on käytössä: `camera` — tämän kameran oma, laskettu sen kalibrointipaketin laboratoriopisteestä (RGB-kamerat); `fleet` — laivueen vakio, jota käytetään, kun paketissa ei ole laboratoriopistettä; `override` — taustapuolella asetettu kentän ohitusarvo. |
| `k_exp_g_fleet` | Laivueen vakio, vertailua varten. |

### Synkronoitu laitteisto — `ArraySession` (Smart-Prep)

`connect_array` on **suositeltu lähtökohta** monikamerajärjestelmille. Se suorittaa taustalla koko GUI-pohjaisen Smart-Prep-prosessin:

1. **Verkkoanalyysi**(`/api/camera/array/recommend`) — asettaa**koko anturin** (ei pikselien yhdistämistä, ei ROI-rajaus) ja antaa kuvataajuuden laskea siihen tasoon, jonka tämä isäntä ja verkko pystyvät tarjoamaan, ottaen huomioon pikseliformaatit ja pakkausmuodon, joita Connect tulee käyttämään. Kuvia ei koskaan pienennetä kuvataajuuden ylläpitämiseksi: välitä pienempi `width` / `height`- tai `binning=2` / `4`-arvoa, jolloin tarkkuutta voidaan vaihtaa kuvataajuuteen (päätepisteen`alternatives`-luettelossa on nämä vaihtoehdot ja niiden ennustetut fps-arvot).
2. **Tason automaattinen valinta** — `sim-capture-sim-emit` (kaikki kamerat valottavat samalla laukaisulla, lähetystahti on sovitettu kaistan jakamiseksi, kuvataajuus laskee kaistan jakamisen pienentyessä) missä tahansa verkossa, jossa kameroiden lähetystahtia voidaan sovittaa kaistan jakamiseksi; `slip-emit-and-capture` (kamerat käyttävät kaistalla vuorotellen, eivät valoteta samanaikaisesti) vain silloin, kun tämä ei ole mahdollista.
3. **PTP käytössä**oletuksena — kameroiden väliset aikaleimat ovat yhden jaetun kellon mukaisia**~1 ms**:n tarkkuudella, taustapalvelimen [PTP-grandmaster](#ptp-grandmaster--time_sync_status). Samanaikainen valotus perustuu M8-laitteiston laukaisimeen (**&lt; 100 µs** moduulien välillä), ei PTP:hen: PTP synkronoi *aikaleimoja*, ei valotuksia.
4. **Kamerakohtainen pikselimuodon automaattinen valinta** — mustavalkoinen → `Mono12`, väri → `BayerRG12`.
5. **AE-alustaminen** — tallentaa kunkin kameran nykyisen AE-tilan, jotta yhteyden muodostaminen ei nollaa valotusta kesken kuvauksen.
6. **GPIO-laukaisun konfigurointi** — `connect_array` asettaa jokaisen kameran valmiustilaan (`TriggerMode=On`, `TriggerSource=Line2`), jolloin päälaitteenpulssi ohjaa orjakameroita M8-kaapelin kautta. Tämä vaihe koskee vain kameraryhmiä: yksittäinen kamera, joka avataan komennolla `LatticeCamera`, toimii sen sijaan vapaasti. Kameraryhmän vapauttaminen (`disconnect()` / `force_disconnect()`, `chloros-cli lattice array-disconnect`) palauttaa kaikki kamerat vapaaseen toimintaan (`TriggerMode=Off`), ja kaikki muut asetukset säilyvät sellaisina kuin ne olivat yhteyden muodostuksen yhteydessä: laukaisun ja synkronointilinjan asetukset (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`; pääkameran synkronointilinja takaisin tuloksi), PTP sekä istunnon aikana muutetut kameran asetukset — ellei `keep_settings=True` / `clear_settings=True` (katso [Kameran muisti](#camera-memory--keep_settings--clear_settings)).

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

Ominaisuudet: `array_id`, `master_serial`, `slave_serials`, `ptp_enabled`, `sync_mode`, `packed_transport` ja `packed_reason` (neljä viimeistä ovat `None`, joka on `attach_array`:n kahvassa), `connect_stream` ja `backend_url`. `connect_stream` on yhteysvastauksen `stream`-lohko, yksittäisen yhteyden muoto sekä `members` (`{serial: {state, why?}}`): `streaming`, kun taulukon ensimmäinen synkronoitu ryhmä on saapunut (taustapalvelin odottaa sitä, rajoitettuna noin 10 sekuntiin), tai `no_frames`, jossa kunkin jäsenen viimeisin hakuvirhe on sen `why` ja, jumbopakettien osalta, 1500 tavun varajärjestelmä `note`:ssä. `None` `attach_array`-käsittelytunnuksessa.

#### `connect_array()`-allekirjoitus

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # pin the master trigger rate (auto if None); see set_target_fps
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    packed="auto",                        # packed 12-bit transport: "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=600.0,                        # s; also the backend's deadline (see below)
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`timeout` on aika, jonka kutsu odottaa yhteyden muodostumista, ja se lähetetään
taustapalvelimelle määräaikana. Jos yhteyttä ei ole muodostettu siihen mennessä, se peruutetaan: jokainen kamera, jonka se
on avannut, palautetaan tilaan, jossa se löydettiin, ja suljetaan, mikään taulukko ei ole kytkettynä, ja
kutsu aiheuttaa virheen `ChlorosConnectError` -virheen (HTTP 504 `CLIENT_DEADLINE_PASSED`, jossa mainitaan
vaihe, johon yhteyden muodostaminen oli edennyt). Näin ollen virheen aiheuttanut kutsu ei koskaan jätä taustalleen
jälkeensä kytkettyä ryhmää. 9 kameran ryhmä kytkeytyy noin minuutissa; heti
kameroiden käynnistymisen jälkeen se voi kestää useita minuutteja. Ryhmä, joka on käynnissä mutta jolla ei ole
vielä synkronoitua kehysryhmää, palautetaan virheellä `connect_stream["state"]`
`no_frames`. Taustapalvelimen lokissa on yksi aikamerkintä kutakin yhteysvaihetta kohti
(`[array-connect <id>] <step>: <s> s …`) ja viimeinen rivi, jossa mainitaan hitain vaihe.

`force_tier`-arvot:
- `"sim-capture-sim-emit"` — todellinen samanaikaisuus (kaikki kamerat laukeavat samalla kelloreunalla).
- `"sim-capture-ftd-stagger"` — joustava aikatasoinen porrastus (kamerat lähettävät hieman toisistaan poikkeavina aikoina, jolloin paketit sarjoituvat siirtokanavalla).
- `"slip-emit-and-capture"` — peräkkäinen kaappaus kameraa kohti (ei ajallista synkronointia). Valitaan automaattisesti vain silloin, kun kameroiden tahtia ei voida hidastaa tarpeeksi, jotta ne voisivat jakaa siirtoyhteyden samanaikaisesti.

`wire_ceiling_mbps` ohittaa **isännän jatkuvan siirtoyhteyskapasiteetin** megabitteinä sekunnissa (MB/s) — tämän yhden
luvun perusteella koko kameraryhmän resurssien allokointi määräytyy. Jätä se `None`-asetukseksi, jotta käytetään automaattisesti tunnistettua
arvoa. Pienennä sitä, kun järjestelmä ilmoittaa GVSP-vioittuneista kehyksistä: automaattinen arvo johdetaan
verkkokortin ilmoittamasta linkkinopeudesta, joka yliarvioi USB-sovittimet, kapeat PCIe-väylät ja
ruuhkaiset jaetut verkot — ja tämä yliarviointi näkyy vioittuneina kehyksinä eikä
näkyvästi hitaana linkkinä. Arvo tallennetaan projektin matriisin tallennuslohkoon, joten
uudelleen avaaminen tai myöhempi `connect_array`-asetus palauttaa sen kuten minkä tahansa muun matriisin asetuksen.
Katso [Matriisin kunto](#array-health--which-subsystem-is-losing-frames).

#### Pakattu 12-bittinen siirto (`packed=`)

Täydellä anturikapasiteetilla matriisin kuvataajuus määräytyy sen mukaan, kuinka nopeasti kukin kamera voi lähettää
kuvansa. 12-bittiset formaatit (`BayerRG12`, `Mono12`) siirretään yleensä 16-bittisinä
sanoina; pakattu siirto lähettää samat 12 bittiä 1,5 tavuna pikseliä kohti ja
purkaa ne isäntäkoneella, joten kunkin kehyksen lähettäminen vie 25 % vähemmän aikaa — noin
**33 % enemmän kuvia sekunnissa täydellä anturikapasiteetilla** (mitattu: 10,7 → 14,0 fps
9 kameran ryhmässä). **Tallennetut tiedot ovat identtiset**: samat arvot, samat 16-bittiset
tiedostot, samat metatiedot.

- `packed="auto"` (oletus) pakkaa vain, kun siitä on hyötyä **ja** isäntälaite
  pystyy purkamaan pakkauksen: yhteyden muodostamisen yhteydessä taustapalvelin mittaa oman purkunopeutensa,
  prosessorien lukumäärän, nykyisen kuormituksen ja verkkokapasiteetin. Työpöytätietokone, jolla on ylimääräisiä ytimiä,
  pakkaa täydellä anturikapasiteetilla; kuormitettu tai vähätehoinen isäntä (yksilevyiset tietokoneet,
  sulautetut ohjaimet) lähettää dataa purkamatta. Pienet ROI-alueet ja binoidut kehykset hyötyvät tästä vain vähän.
- `packed="off"` (tai `False`) ei pakkaa koskaan — tämän voi sulkea pois vianmäärityksen aikana
  tai isäntäkoneessa, jonka on varattava prosessoritehoa muihin tehtäviin.
- `packed="on"` (tai `True`) pakkaa aina 12-bittiset formaatit.
- Mikä tahansa muu arvo aiheuttaa virheen `ValueError`. 8-bittisiä ja 16-bittisiä formaatteja ei koskaan
  pakata.

`analyze_array_network(..., packed="auto")` toimii samalla periaatteella,
joten sen fps on se nopeus, jolla taulukko toimii (vastaus sisältää
`packed_transport` ja `packed_reason`); sen `pixel_format`-oletusarvo on se, mitä
connect soveltaa (Mono12 / BayerRG12), eikä 8-bittinen.

Päätös ja sen perustelu ovat istunnossa:

```python
arr = chloros_sdk.connect_array(serials)          # packed="auto"
print(arr.packed_transport, arr.packed_reason)
# True  auto: projected 11.0 -> 14.4 fps, unpack 0.16 of 1.50 budgeted cores

arr = chloros_sdk.connect_array(serials, packed="off")
```

`ChlorosProject.connect_all(packed=...)` soveltaa samaa ohitusta jokaiseen
projektin taulukkoon (oletusarvo `None`, mikä tarkoittaa `"auto"`: projekti ei tallenna
pakattua asetusta taulukokohtaisesti).
Koneenlaajuiseksi oletusasetukseksi määritetään `CHLOROS_ARRAY_PACKED12=off` (tai `on`)
taustaprosessin ympäristössä; mutta nimenomainen `packed="on"` / `"off"` on silti etusijalla.
`/api/camera/array/status` raportoi `packed_transport` ja `packed_decision`:n.

#### Odotetut kuvataajuudet

Mitatut matriisin kuvataajuudet (laitteistossa synkronoitu, 12-bittinen; koko anturi = 2048 × 1536). Käytä niitä `status()["actual_fps"]`:n järkevyystarkistukseen; `analyze_array_network()` ennustaa kuvataajuuden omalle isäntäkoneellesi ja verkollesi.

| Asetukset | Matriisin kuvataajuus |
| --- | --- |
| Täysi anturi, PCIe 10 GbE -isäntäverkkokortti, pakattu siirto (`packed="auto"` pöytätietokoneella) | ~14 fps |
| Täysi anturi, PCIe 10 GbE -isäntäverkkokortti, pakkaamaton (`packed="off"`) | ~10,5 kuvaa sekunnissa |
| Täysi anturi, 8 kameraa USB 10 GbE -sovittimella | ~3,8 kuvaa sekunnissa |
| Täysi anturi, 1 GbE -isäntäyhteys | 1,2–1,7 kuvaa sekunnissa |
| 2 × 2 -binning (`binning=2`), 10 GbE -isäntäverkkokortti | ~30–33 kuvaa sekunnissa |

- PCIe 10 GbE -yhteydellä **kamerat** asettavat rajan täysikokoisella anturilla, ei verkko: 4 ja 8 kameraa toimivat samalla nopeudella.
- **`target_fps` on laukaisunopeus, eikä järjestelmä koskaan toimi tätä nopeammin.** `target_fps=1` käyttää järjestelmää 1 fps:n nopeudella; `None` (oletus) käyttää sitä isäntäkoneen ja verkon sallimalla suurimmalla nopeudella. Tätä nopeutta ylittävä arvo lasketaan kyseiselle tasolle (katso [`set_target_fps`](#interval-capture-for-drone-surveys)).
- **`link_speed_mbps`:n arvo `4294`:ssa `status()`:ssa ei ole linkkinopeus.** Tämä tarkoittaa, että Windows ei pystynyt lukemaan sovittimen nopeutta (ohjain ilmoitti suurimman 32-bittisen arvon), mikä on tyypillistä USB-Ethernet-sovittimille. USB-sovittimen käyttökelpoinen nopeus Windowsissa on joka tapauksessa rajoitettu 200 MB/s:iin (katso [Array Health](#array-health--which-subsystem-is-losing-frames)); sovittimen säätämisestä katso CLI-ohjeen kohta ”USB 10GbE -sovittimet” osiossa [Host NIC Setup &amp; Tuning](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **USB-rajoitus koskee myös Linuxia: 400 MB/s x86_64-alustalla, 200 MB/s NVIDIA Jetsonilla.** Sovitin tunnistetaan sysfs:n perusteella (varajärjestelmänä käytetään `ethtool -i`), joten USB 5 GbE- tai 10 GbE -sovittimen nopeus rajoitetaan enimmäisrajaan sen sijaan, että käytettäisiin sen nimellistä nopeutta 500 / ~1060 MB/s. x86_64-arvo mitattiin Linuxissa: sama sovitin, joka Windowsissa vioittaa kehyksiä yli 200 MB/s:n nopeudella, toimi virheettömästi neljän kameran järjestelmässä kaikilla enimmäisnopeuksilla aina 480 MB/s:iin asti. NVIDIA Jetsonilla sama sovitin toimi virheettömästi vain 200 MB/s:n nopeudella ja menetti kehyksiä ajoittain sitä suuremmilla nopeuksilla. Jos isäntäkoneesi käsittelee enemmän kehyksiä ilman `gvsp_corrupt`-kehyksiä, korota sitä `wire_ceiling_mbps`:llä. Linuxissa USB 2.0 -porttiin liitetylle sovittimelle on varattu 30 MB/s; liitä se USB 3 -porttiin.

#### Ylimerkintä (kameraa kohti määritetty alaraja)

Ryhmä toimii missä tahansa verkossa; kuvataajuus määräytyy sen mukaan. Sim-emit-taajuudenhallinta jakaa jokaiselle kameralle tasaisen osuuden törmäyksiltä suojatusta kaistanleveydestä, ja ne kaikki laukaisevat edelleen samalla laukaisimella, joten hitaalla yhteydellä osuus — ja kuvataajuus — on pieni: 8 täysresoluution 12-bittistä kameraa 1 GbE:llä ilman jumbo-kehyksiä saavat kukin 6,5 MB/s, noin **1 fps**(noin 1,3 fps pakattuna). Osuus on rajattu alimmalle nopeudelle, jolla kameroita voidaan tahdistaa (`per_cam_floor_bps`: luetaan kunkin kameran GevSCPD-alueelta yhteyden muodostuksen yhteydessä, eikä se ole koskaan alle 1 MB/s). Jos yhteys vaatii hitaampaa siirtonopeutta, yhteys siirtyy**sarjaliikenteeseen**(`slip-emit-and-capture`): kamerat vuorottelevat täydellä linkkinopeudella, joten jokainen kehys saapuu, mutta jäsenet**eivät ole näkyvissä samanaikaisesti**— vastauksen taso ja estäjät (`SERIAL: ...`) kertovat tämän; välitä `force_tier="sim-capture-sim-emit"` kieltääksesi varajärjestelmän käytön. Kehyksen koko ei vaikuta jakamiseen, joten**kuvien yhdistely ja ROI nostavat kehysnopeutta, mutta eivät muuta sitä, mahtuuko kuvasarja**. Vain linja, joka ei pysty siirtämään matriisia millään tahdilla, samanaikaisesti tai vuorotellen,**ylikuormittuu** — vikatila on GVSP-pakettihäviö, ei alhaisempi kuvataajuus. Korjaustoimenpiteet: vähemmän kameroita, jumbo-kehykset päästä päähän tai nopeampi verkkokortti.

- Vastaukset `analyze_array_network()` ja `/api/camera/array/connect` sisältävät `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` ja `per_cam_floor_bps`. Kun `oversubscribed` on totta, projektio **nollaa fps-kentät** (`achievable_fps_max` / `fps_bright` / `fps_dark`) sen sijaan, että se ilmoittaisi harhaanjohtavan, hitaan mutta toimivan nopeuden.
- `POST /api/camera/array/connect` hyväksyy `pin_resolution`-runoparametrin (**vain HTTP — ei SDK-kwarg**; `connect_array` ei paljasta sitä). Se on tiukka tila: ylimerkitty yhteys, jossa `pin_resolution` on asetettu,**hyläetään ehdottomasti** virheilmoituksella, jossa mainitaan kaikki korjaustoimenpiteet. Ilman sitä yhteyden muodostaminen jatkuu varoituksella, että taulukko menettää paketteja. Kummassakin tapauksessa yhteys säilyttää pyydetyn kehyskokoa — se ei koskaan ryhmittele tai rajaa paketteja omasta aloitteestaan.
- Vain testausta varten: aseta `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` taustapalvelimen ympäristössä, jolloin hylkäys muuttuu voimakkaaksi varoituksiksi — yhteys muodostetaan joka tapauksessa ja pakettihäviö hyväksytään.

#### Taulukon kunto — mikä alijärjestelmä menettää kehyksiä

`ArraySession.capability()` (`GET /api/camera/array/<array_id>/capability`) sisältää reaaliaikaisen `health`-lohkon
kytketyssä taulukossa, jota arvioidaan uudelleen **10 sekunnin** ajanjakson kuluessa. Se jakaa kehysten menetykset
kahteen syyhyn, jotka vaativat vastakkaisia korjauksia, sen sijaan että käytettäisiin yhtä ”epätäydellisyysastetta”, joka
ei nimeä kumpaakaan:

| Kenttä | Mitä se tarkoittaa | Mikä alijärjestelmä |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (sarjanumeroittain) | Kehys **saapui, mutta oli rakenteellisesti viallinen**— GVSP-pakettihäviö. |**Verkko**: kaapelibudjetti, tahdistus, NIC RX-rengas, MTU |
| `never_arrived_rate_pct` (sarjanumeroittain) | Kehystä **ei saapunut lainkaan**— kamera ei laukaisi tai siitä ei lähtenyt mitään. |**Laukaisu / synkronointi**: M8-kaapeli, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Huonoin kameran lähetysnopeus kunkin kameran osalta. | — |
| `per_cam_rate_pct` | Yhdistetty epätäydellisyysaste kameraa kohti (molemmat syyt yhdessä). | — |
| `stable_for_seconds` | Kuinka kauan kukin kamera on pysynyt alle 0,01 %:n. | — |

`health`:n ohella samassa tietueessa ilmoitetaan koko allokoinnin viivästymismäärä:

| Kenttä | Merkitys |
| --- | --- |
| `wire_ceiling_mbps` | Isännän voimassa oleva jatkuva kaistanleveys, MB/s. |
| `wire_ceiling_source` | Mistä kyseinen luku on peräisin, sanoin ilmaistuna — esim. `USB-capped 200 MB/s (was theoretical 1062; …)` tai `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, kun `wire_ceiling_mbps=` asetti sen. |
| `nic_is_usb` | `true` USB-Ethernet-sovittimelle. |

```python
import chloros_sdk

serials = ["213800234", "214000533"]           # master first
arr = chloros_sdk.attach_array(serials)
h = arr.capability()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    # force_disconnect(), not disconnect(): this handle came from attach_array,
    # and disconnect() never tears down an array the session only attached to.
    # It returns only once the backend confirms the release; a teardown that
    # timed out or was refused raises ChlorosConnectError instead, so the
    # reconnect below never races a half-released array.
    arr.force_disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Tulkinta:** nollasta poikkeava `gvsp_corrupt_rate_pct`-arvo ja `never_arrived_rate_pct`-arvo 0 tarkoittaa, että
liipaisu ja kaapelisynkronointi ovat moitteettomia ja 100 % häviöstä johtuu verkkoreitistä — pienennä
`wire_ceiling_mbps` ja muodosta yhteys uudelleen. Päinvastainen kuvio viittaa sen sijaan synkronointikaapeliin tai
liipaisulinjaan.

> **`target_fps` ei ole ratkaisu vioittuneiden kehysten ongelmaan.** GevSCPD-taajuus määritetään kerran
> yhteyden muodostuksen yhteydessä, joten laukaisutaajuuden laskeminen muuttaa käyttöjaksoa, ei
> samanaikaisten lähetysten pursetaajuutta. Mitattu 5-kertainen kysynnän leikkaus ei tuottanut parannusta, kun taas
> kaapelin enimmäisnopeuden laskeminen 240:stä 200 MB/s:iin laski saman laitteiston virheiden määrän 10,4 %:sta
> 0,00 %:iin.

> **Käynnissä oleva matriisi ei voi suunnitella itseään uudelleen LATTICE (M3C / M3M) -laiteohjelmistossa.** Se ei voi
> korjata tätä itse; irrota ja liitä uudelleen, jotta yhteyden muodostusajankohdan valitsin suunnittelee uudelleen
> uuden ylärajan perusteella.

**USB-Ethernet-sovitin on rajoitettu 200 MB/s:iin** mittauslaitteen toimesta riippumatta sen
tyyppikilvestä: tehokkuustaulukko, joka muuntaa linkkinopeuden jatkuvaksi arvoksi, on
PCIe-pohjainen, ja USB-verkkokortti ilmoittaa Ethernet-linkkinopeutensa, mutta sitä rajoittaa
USB-väylä ja sen ohjain. Rajoitus on absoluuttinen, ei suhteellinen — USB 1 GbE -sovitin
tuottaa ~80 MB/s, eikä rajoitus vaikuta siihen. Se koskee Windowsia ja Linuxia (mukaan lukien Jetson);
katso [Odotetut kehysnopeudet](#expected-frame-rates) saadaksesi tietää, missä se on mitattu.

#### `ArraySession`-menetelmät

| Menetelmä | Kuvaus |
| --- | --- |
| `status(timeout=10.0)` | Käynnissä olevan taulukon reaaliaikainen tila: `target_fps` / `target_fps_auto` (laukaisutaajuus ja se, hallitseeko taajuussäädin sitä), `target_fps_clamped_reason` (miksi kiinnitetty taajuus pidetään alhaisempana, muuten `None`), `actual_fps` (mitattu; 0, kun tartuntasilmukka on luopunut tai jumittunut), `recommended_fps`, `streaming` (False, kun tartuntasilmukka on luopunut — tarkista tämä, älä pelkästään `last_error`; pysyy totena asetusten uudelleenkäynnistyksen ajan, minkä `restarting` merkitsee), `stalled` (tosi, kun tartuntasilmukka on käynnissä, mutta ei ole julkaissut kehysryhmää noin kymmenen kehysjakson ajan, 2–30 s; `last_error` ilmoittaa tästä vasta 60 sekunnin kuluttua), `last_frame_age_s`, `last_error`, `serials`, `ptp_enabled` (`True` vain silloin, kun taulukko oli kytketty PTP:hen **ja** jokainen jäsen luki viimeksi `PtpEnable`-tilaan; ei sama kuin lukittu, jota `ptp_epoch` mittaa), `ptp_configured` (mitä yhteys pyysi), `ptp_off_members` / `ptp_unread_members` (jäsenet, jotka lukivat viimeksi `PtpEnable`-tilan pois päältä / eivät vastanneet), `member_recovery` (`None` tai jäsen, joka menetti virran tai yhteytensä: `state`, `waiting`, vaikka se ei vastaa, `reopening`, sitten `recovered`, yhdessä `members`:n ja `why`:n kanssa — ryhmä avaa sen uudelleen, kun se vastaa jälleen, ja palauttaa sille sen yhteyshetken PTP-arvon, liipaisimen ja synkronointilinjan kytkennät, tahdistuksen ja kameran asetukset, minkä jälkeen se lähettää jälleen dataa ilman uusia `connect_array`) ja `ptp_epoch` (mitattu), `sync_mode` / `sync_tier`, `packed_transport` / `packed_decision`, `per_cam_emit_mbps`, `link_speed_mbps` (`4294` tarkoittaa, että Windows ei pystynyt lukemaan sovittimen nopeutta — katso [Odotetut kuvataajuudet](#expected-frame-rates)), `trigger_pipeline` / `pipeline_tune`, `radiometric_chain` (onko kaistakohtainen säteilyketju käyttökelpoinen ja mikä sen jäsen rikkoi sen) sekä `band_clamp` — samalla kun reaaliaikainen näkymä renderöi matriisin: valaistujen pikselien osuus, jonka erottelu on rajoitettu arvoon 0, jäsenittäin (`members`: `{serial: [band0, band1, band2]}` kameran R-, G-, B-kaistajärjestyksessä) ja kaistakohtaisesti yhdistelmä näyttää (`composite`: `{band name: share}`), ja `warn`, osuus, jonka käyttöliittymä merkitsee ”värikaistatiedot, joiden luotettavuus on heikko” (mitataan noin kerran sekunnissa; `None`, kun reaaliaikaista näkymää ei ole käynnissä), ja `member_ae` — jäsenittäin valotus (`exposure_us`) ja vahvistus (`gain_db`) sen viimeisimmässä julkaistussa kehyksessä, otettuna itse kehyksestä (`read_nodes` tai `ExposureTime` / `Gain` palauttaa viimeisimmän arvon, joka on kirjoitettu automaattisen valotuksen ollessa käynnissä, ei senhetkistä tilaa). |
| `set_member_settings(serial, **kwargs)` | Kirjoita yhden jäsenkameran asetukset, kuten `CameraSession.set_settings()` ne ottaa (valotus, vahvistus, kohdekirkkaus jne.); jäsen pysyy taulukossa. Heittää `ChlorosConnectError`-virheen, jos sarjanumero ei kuulu jäseniin. Katso [Valotus taulukon jäsenille](#exposure-on-array-members). |
| `capability(timeout=10.0)` | Mitä smart-prep on määrittänyt (tallennustaso, kuvakoko, `pixel_format`), live-lohko `health` ja isäntäkoneen siirtokapasiteetti. Katso [Array Health](#array-health--which-subsystem-is-losing-frames). |
| `set_target_fps(fps, timeout=10.0)` | Muuta laukaisutaajuutta arrayn ollessa käynnissä ilman uudelleenkytkentää. Luku lukitsee sen (taajuussäädin jättää sen ennalleen); `None` palauttaa sen automaattitilaan. Palauttaa `{requested, target_fps, target_fps_auto, clamped}` — `clamped` on True, kun taajuus on laskettu tasolle, jonka isäntä ja verkko pystyvät käsittelemään. Katso [Väliaikainen kuvaus drone-kartoituksissa](#interval-capture-for-drone-surveys). |
| `grab(processing="debayered", with_metadata=False, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, bit_depth=None, band=None, align=False, refl_max=None)` | Yksi kehys jokaisesta kamerasta, **kaikki yhdellä laukaisulla**, numpy-taulukoina (tiedostoja ei tallenneta). Palauttaa `GrabResult` (tai `dict`): `{serial: array}` tai `{serial: (array, metadata)}` yhdessä `with_metadata=True`:n kanssa. `.group_id` on laukaisija, jota kuvat jakavat; `.errors` kohdistaa jokaisen pois jätetyn kameran syyyn (kameraa, jota ei voida palvella kyseisellä laukaisijalla, jätetään pois, eikä sitä koskaan täytetä toisesta kamerasta). `processing` on `raw` / `debayered` / `radiance` / `reflectance` / `display`. Sama luku kuin projektin API:n [`ArrayHandle.grab()`](#one-trigger-per-read--group_id) — yksi yhteinen toteutus — joten kaikki siellä olevat avainsanat (`band`, `align`, `bit_depth`, `refl_max`) toimii tässä. Tarvitaan `numpy`. |
| `frame_stream(*, processing="debayered", fps=5.0, duration=None, count=None, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, with_metadata=False, bit_depth=None, band=None, align=False, refl_max=None)` | Iteraattori, joka tuottaa yhden `grab()`-tuloksen kutakin tikkiä kohti (vain avainsanaparametrit), tahdistettuna `fps`:ään, rajoitettuna arvoon `duration` (sekuntia) ja/tai `count` (`None` = rajoittamaton). **Jokainen tuotettu joukko on uusi laukaisija:** tikki, jonka uusin laukaisija on jo tuotettu joukko, odottaa seuraavaa laukaisijaa sen sijaan, että toistaisi sen, joten virta kulkee hitaammalla nopeudella, joka on joko `fps` tai taulukon oma kuvataajuus. Nostaa `FrameFetchError`:n sen sijaan, että menisi hiljaiseksi: heti, kun jokainen kamera kieltäytyy jostain syystä toistamasta seuraavaa tikkiä, 60 sekunnin kuluttua, jos tikkejä on ollut ilman kehystä, ja 60 sekunnin kuluttua, jos uutta laukaisijaa ei ole tullut (&quot;ei uutta laukaisijaa 60 sekunnin aikana&quot;). Sama toteutus kuin `ArrayHandle.frame_stream()`. Katso [Live-kehykset omassa putkistossasi](#live-frames-in-your-own-pipeline). |
| `continuous(output_dir="output", *, format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, max_captures=None, duration_s=None, daq=None, daq_tolerance_ms=None, timeout=30.0)` | Käynnistä taustapalvelimen oma jatkuva tallennus (käyttöliittymän Jatkuva-tila) → [`ContinuousHandle`](#continuoushandle). Tallentaa **kaikki** synkronoidut ryhmät, jotka taulukko julkaisee, samalla tavalla kuin `capture()` tallentaisi ne (samat vientirajoitukset), kunnes se pysäytetään tai kunnes `max_captures` / `duration_s` (`None` = kunnes pysäytetään). Heittää `ChlorosConnectError`-virheen, jos tälle taulukolle on jo käynnissä yksi. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, smart=False, after_seq=None, daq=None, daq_tolerance_ms=None, timeout=300.0)` | Yksi synkronoitu kaappausryhmä. Palauttaa `CaptureResult` (luettelo kehyssanakirjoista + `.skipped` + `.failed` + `.group_seq` + `.daq_files` + `.daq_matches` + `.alignment` + `.oriented`). Syötä edellinen tulos`group_seq`:n `after_seq`:ksi, jotta saat sen jälkeisen seuraavan synkronoidun ryhmän sen sijaan, että odottaisit uutta. Vientiä koskevat asetukset alla. |
| `capture(..., smart=True)` | **Älykäs tallennus** — odottaa, että AE vakiintuu kaikissa kameroissa, ja laukaisee sitten. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, orient=None, timeout=120.0)` | Nopein tallennus: vain raakadata + määritetty DAQ-lukema (+ vapaa yhdistetty indeksi). Vastaa käyttöliittymän ”Nopein tallennus” -painiketta. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Yksittäinen / Jatkuva / Väli yhdessä rajatussa silmukassa. Palauttaa `list[CaptureResult]`.**Vaatii `count`:n ja/tai `duration_s`:n**, jotta se päättyy (SDK:ssa ei ole Ctrl+C-komentoa). Lopetetaan myös ennenaikaisesti, kierroksen jälkeen, kun kierros ilmoittaa levyn täyttyneen (`.failed`-merkintä, jossa on `disk_full`); kyseinen kierros on viimeinen tulos. |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, expr=None, timeout=30.0)` | Aloita yhdistettyjen indeksien reaaliaikaisen näkymän tallennus videoksi/GIF-tiedostoksi → `RecorderHandle`. Video on Motion-JPEG-muotoinen `.avi` (ei vaadi ffmpeg:tä). Jos graafista käyttöliittymää ei ole auki, taustaprosessi renderöi sen: `expr` (indeksilauseke taulukon kaistojen nimissä, esim. `"(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)"`) tai oletuksena taulukon NDVI-arvo. Yksi yhdistelmätallennin kutakin taulukkoa kohti. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, orient=None, timeout=30.0)` | Käynnistä korkean kuvataajuuden raaka-Bayer-sarjakuvaus → `RecorderHandle`. Käsittele uudelleen offline-tilassa `build_video()`:llä. `orient=True`: `build_video()`:n tuottamat tuotteet käännetään kameran suuntauksen mukaan (kamerakohtaiset tuotteet oman kameran suuntauksen mukaan, yhdistetty indeksi pääkameran suuntauksen mukaan); raakakuvat pysyvät alkuperäisessä muodossaan ja sarjakuvausmanifesti tallentaa jokaisen kuvan suuntauksen. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Käsittele tallennettu raakasarja offline-tilassa kalibroiduksi videoksi (videoiksi). Estää muita toimintoja, kunnes tehtävä on valmis (`wait=True`), ja palauttaa käsittelytuloksen: `{success, outputs, errors, skipped, substitutions, notes, degraded, combined}` — katso [käsittelytulos](#the-build_video-result) alla. |
| `build_video_status(job_id, timeout=15.0)` | Kysele offline-rakennustyön tilaa: `{running, result, error, burst_dir}`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Vapauta koko matriisi — **jos tämä istunto avasi sen** (`connect_array`): se on tällöin `force_disconnect()`, ja aiheuttaa virheen `ChlorosConnectError`, jos vapautusta ei vahvisteta (`with`-lohkon poistuminen ei koskaan aiheuta poikkeusta). Kun käsitellään `attach_array`:n antamaa kahvaa, se jättää ryhmän käynnissä ja tulostaa stderr:iin yhden rivin, jossa ilmoitetaan tästä: ryhmä voi kuulua graafiseen käyttöliittymään tai toiseen skriptiin. Jokainen kamera jätetään vapaasti toimimaan ja muutoin sellaisena kuin se oli yhteyden muodostuksen yhteydessä; `keep_settings=True` tallentaa sen asetukset kameran omaan muistiin, `clear_settings=True` nollaa kyseisen muistin tehdasasetuksiin (katso [Kameran muisti](#camera-memory--keep_settings--clear_settings)). |
| `force_disconnect(timeout=60.0, keep_settings=False, clear_settings=False)` | Vapauttaa koko ryhmän riippumatta siitä, kuka sen avasi. Käytä tätä liitettyyn kahvaan, kun tarkoituksena on lopettaa ryhmä (esim. liittääksesi sen uudelleen eri asetuksilla). Palauttaa, kun taustapalvelu vahvistaa vapautuksen tai kun vapautettavaa ei ole (taulukko on jo poistettu tai taustapalvelua ei ole käynnissä); aikakatkaistunut tai taustapalvelun hylkäämä purku aiheuttaa virheen `ChlorosConnectError`. Kummallakin lipulla se palauttaa sen, mitä kunkin kameran muisti sai (`camera_memory`), muussa tapauksessa `None`. |

`capture()`-vientiohjaimet (sama päätepiste kuin GUI:ssa ja CLI:ssä):

- `processing` / `levels` — `processing="all"` (tai `levels=["raw","radiance",…]`) tallentaa jokaisen soveltuvan vientityypin kameraa kohden; yksi `processing`-arvo tallentaa vain kyseisen tason.
- `aligned` — **oletusarvoisesti päällä (`None`), koska Chloros-sovelluksen tallennus samasta matriisista on**(sen taulukokohtainen**Aligned**-tallennusasetus). Jokaisen jäsenen ei-raakamuotoinen vienti kohdistetaan pääkameran kehykseen taulukon [kohdistusprofiilin](#array-alignment) avulla jäsenien yhteiseen päällekkäisyyteen, joten tiedostot on pikselitasattu ja niiden koko, rajaus ja `Alignment*`-tunnisteet ovat samat kuin sovelluksen (ROI-alueen alla muunnos perustuu uudelleen ikkunaan); `XMP-mapir:AlignmentJSON` määrittelee `"applied": true`:n sen `crop_roi`:n avulla. Raw-tiedostot pysyvät vääristämättöminä alkuperäisessä koossaan, mutta sisältävät muunnoksen metatiedoissa. `aligned=False` tallentaa jokaisen tiedoston alkuperäisessä koossa, vääristämättömänä; `aligned=True` on oletusasetus. `continuous()`, `capture(smart=True)`, `capture_repeated()` sekä projektitason `ArrayHandle.capture()` / `capture_smart()` käyttävät samaa oletusasetusta. Riippumatta siitä, sovelletaanko vääristymää vai ei, jokainen jäsen-tiedosto tallentaa kameransa `Alignment*`-muunnoksen, kun taulukolla on kohdistusprofiili — lukuun ottamatta kameraa, jota profiili ei kohdista (jätetty pois tai pakotettu identiteettimatriisiksi ratkaisun epäonnistumisen vuoksi) sekä vertailukamera, kun mitään muuta kameraa ei ole kohdistettu: näissä ei ole `Alignment*`-tunnisteita. Jos matriisilla ei ole profiilia, tallennus tallennetaan kohdistamattomana, ja tulos ilmaisee tämän: `result.alignment` on `{"requested": True, "applied": False, "warnings": [...]}`, yksi varoitus kutakin syytä kohden, miksi vääristymää ei voitu soveltaa (ei profiilia, jäsenet eri resoluutioilla, matriisi ei ole-synkronoitu, kehys, johon profiilia ei voida kartoittaa). `applied` on `True` vain silloin, kun jokainen vääntöä pyytänyt vienti sai sen. Matriisin automaattinen kohdistus luo profiilin, kun sen valotus vakiintuu; `ArrayHandle.calibrate_alignment()` / CLI:n `align-calibrate` luovat profiilin pyynnöstä. `result.oriented` kertoo, onko `orient=True` kääntänyt tuotteet.
- `render_index=False` — ohita kamerakohtainen kasvillisuusindeksin peittokuva; oletusasetuksena se renderöidään määritettyyn kohtaan.
- `daq` / `daq_tolerance_ms` — DAQ-lukema tallennetaan oletusarvoisesti `.daq`-sidecarina, vaikka mikään valittu taso ei sitä tarvitsisikaan; `daq=False` ohittaa sen tällöin, ja `daq_tolerance_ms` asettaa vastaavuustoleranssin (oletusarvo 500). `force_daq=True` on oletusarvon vanhempi kirjoitusmuoto. Katso [Tallennuksen tallentamat DAQ-lukemat](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
- `orient=True` — kääntää kunkin jäsenen **tuotteet**(`display`, `radiance`, `reflectance`, kamerakohtainen indeksi ja yhdistelmä) kameran suuntauksen mukaan kohdistusmuutoksen jälkeen:**pääkameran**suuntaan, kun ne on siirretty sen koordinaatistoon (oletusarvo `aligned` — jäsenet pysyvät rekisteröityinä toisiinsa), muuten omien kameroidensa suuntaan.**Oletus: jokainen jäsen-tiedosto alkuperäisessä muodossaan**, kuten array-kaappauksissa aina on; `raw` ja `debayered` pysyvät alkuperäisessä muodossaan kummassakin tapauksessa. Jokainen tiedosto tallentaa sen kehyksen suuntauksen, jossa sen pikselit sijaitsevat (`XMP-Camera:ChlorosRotation` / `ChlorosFlip*` / `ChlorosOrientationApplied` — raakatiedosto, oman kameran), ja `Alignment*` XMP-tiedosto säilyttää alkuperäisen kehyksen muunnoksen, joka on sovellettu ennen suuntaa; RAW-tiedosto, joka sisältää kyseisen muunnoksen, tallentaa myös `ChlorosAligned*`:n (pääkuvan suuntauksen), eli tuotteen suuntauksen, joka on muunnettu pääkuvan kehykseen. Käsittely kääntää jokaisen tuotteen sen pikselien mukaisen suuntauksen mukaan: kohdistetun suuntauksen, kun se muokkaa RAW-tiedostoa (Array alignment on), tai kameran oman suuntauksen, kun se ei kohdista. `capture_repeated()` ja `capture_fastest()` välittävät sen eteenpäin; projektitason `ArrayHandle.capture()` / `capture_smart()` ottavat sen myös huomioon.

**TIFF-pakkaus (vain HTTP-säädin):** `ArraySession.capture()` ei lähetä `compression`-avainta, joten taustapalvelimen oletusasetus on voimassa — `POST /api/camera/array/capture` lukee `compression`-runkoparametrin, oletuksena: pakkaamaton, anturin pikselit täsmälleen sellaisina kuin ne on luettu (~6,3 MB per täysresoluutioinen kehys). `"deflate"` (häviötön zlib L1 + vaakasuuntainen ennustaja, ~4,1 MB/kuva) kirjoittaa pienempiä tiedostoja hitaammin — molemmat ovat häviöttömiä ja luetaan identtisesti tuonnin yhteydessä. SDK ei tarjoa sille kwarg-parametria; kiertotie on `chloros-cli lattice array-capture --compression deflate` tai raaka HTTP. DEFLATE pitää Pythonin GIL:n, joten pakattuja kirjoituksia ei voida suorittaa rinnakkain kamerakohtaisissa kirjoitussäikeissä — jatkuva täysresoluutioinen tallennus anturin taajuudella vaatii pakkaamatonta oletusasetusta. Lisätietoja: [CLI-ohje → array-capture](cli-reference.md).**Peräkkäiset ryhmät (HTTP):** päätepiste palauttaa `group_seq`; silmukka, joka lähettää sen takaisin muodossa `after_seq` seuraavassa pyynnössään, saa seuraavan synkronoidun ryhmän — mukaan lukien sen, joka julkaistiin edellisen tallennuksen tallennuksen aikana — sen sijaan, että odottaisi pyynnön jälkeen julkaistua ryhmää, ja `skipped_groups` ilmoittaa kaikki silmukan ohittamat ryhmät. `capture_repeated`, CLI:n `--continuous` ja GUI:n jatkuva tila toimivat kaikki tällä tavalla; aikavälin läpimeno pysyy ajantasaisena jokaisella aikataulun mukaisella hetkellä.**Jäsenkohtaiset vientiohitukset (vain HTTP):**sama päätepiste hyväksyy myös `exclude_serials` (lista — poista jäseniä tallennetusta joukosta; taulukko laukeaa edelleen yhtenä synkronoituna ryhmänä ja pois jätetyt jäsenet palautetaan `excluded`:ssä), `serial_levels` (`{serial: [level tokens]}` -ohitukset kamerakohtaisella tasolla) sekä `serial_index` (`{serial: bool}` -ohitukset kamerakohtaisessa indeksin päällekkäisyydessä). Nämä ovat GUI-pariteettisia runkoparametreja, eivätkä**SDK-avainsanoja vielä**; karttoista puuttuvat jäsenet käyttävät oletusarvoisesti koko taulukon kattavia `levels` / `render_index`-arvoja.

##### Ohitettujen kameroiden tarkastelu — `CaptureResult.skipped`

`ArraySession.capture()` palauttaa `CaptureResult`:n, joka on `list`:n alaluokka: kierrä sitä, indeksoi se, `len()` — kaikki olemassa olevat mallit toimivat edelleen. Uusi koodi voi tarkistaa `.skipped`-attribuutin nähdäkseen, mitkä kamerat on jätetty pois ja miksi. Yleisin tapaus on RGB-kamerat sekasuodatinmatriisissa, kun pyydät `processing="radiance"`- tai `"reflectance"`-arvoja — Bayer-kohtainen säteilyvoimakkuus on merkityksetön laajakaistaiselle anturille, joten taustakäsittely ohittaa kyseiset kamerat sen sijaan, että tuottaisi järjetöntä tulosta.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Syytunnisteet noudattavat mallia `<level>-not-applicable-to-rgb-cam` (yksi merkintä ohitettua tasoa kohti, joista jokainen sisältää `level`). Kamera, jonka kalibrointipakettia ei ole ladattu, ohittaa `radiance` / `reflectance` merkinnällä `<level>-needs-calibration (connect the camera so its cal pack loads)`, ja jos käyttökelpoista DAQ-heijastuslukemaa ei ole, ohitetaan `reflectance-skipped-no-dls`. Muut heijastukseen liittyvät ohitukset ovat `reflectance-skipped-no-fresh-dls` (uutta alaspäin suuntautuvaa lukemaa ei saatavilla), `reflectance-skipped-bound-daq-unavailable (…)` (sidottua DAQ-laitetta ei voitu tavoittaa) sekä `dls-uncalibrated-band-<nm>` — kaista sijaitsee pääosin DAQ-valosensorin radiometrisesti kalibroidun alueen ulkopuolella (~374–974 nm), joten DAQ:hen perustuvaa absoluuttista heijastuskyvyn jakoa ei hyväksytä ja kehys siirtyy selvästi anturin vasteeseen. Myynnissä olevista tuotteista vain F988 laukaisee tämän; kyseisen kameran tuettu polku on heijastuskykypaneelin työnkulku.

##### Tiedostot, joita ei voitu kirjoittaa — `CaptureResult.failed`

Kuvaus, jossa vain osa tiedostoista voitiin kirjoittaa, palauttaa silti seuraavan: luettelossa on kaikki tiedostot, jotka **kirjoitettiin**, ja `.failed` luettelee loput, yksi sanakirja tiedostoa kohti, jossa `serial` (`None` yhdistetylle komposiitille ja taulukon `.daq`-sidecar-tiedostolle), `level` (`raw`, `radiance`, …, `index`, `composite`, `daq`, tai `match` — `.daq`:n `.match.json`-provenanssi, kun `.daq` kirjoitettiin, mutta sen sivukonttia ei) ja `error`. Täydellä levyllä merkintä sisältää myös `disk_full: True`:n; jokainen myöhempi tallennus kyseiselle levylle epäonnistuisi samalla tavalla, minkä vuoksi `capture_repeated()` päättyy siihen. Osittaisesta tallennuksesta ei synny poikkeusta, eikä tiedostoa `.failed` jäänyt levylle puoliksi kirjoitettuna. `.failed` on tyhjä puhtaassa tallennuksessa.

```python
result = arr.capture("output/", processing="all")
for f in result.failed:
    print(f"SN:{f['serial']} {f['level']} not written: {f['error']}")
if any(f.get("disk_full") for f in result.failed):
    raise SystemExit("drive full -- free space and capture again")
```

##### Tallennuksen lukeva DAQ tallentaa — `.daq_files` ja `.daq_matches`

Aina kun DAQ on sidottu kameraan tai matriisiin tai yhdistetty, jokainen tallennus tallentaa käyttämänsä alaspäin suuntautuvan lukeman: `.daq` tallennuskansion `daq/`-alikansioon, yhdessä sen `.match.json`- ja `.cameras.json`-tiedostojen kanssa (joiden lukema se on). Tämä pätee myös raaka- tai debayeroituihin tallennuksiin, ja tiedosto on sama riippumatta siitä, onko tallennus tehty graafisen käyttöliittymän (GUI), komentorivin (CLI) tai sovelluskehityspaketin (SDK) kautta. `daq=False` ohittaa tämän tallennuksen, jonka tasot eivät käytä DAQ:ta (`radiance`-, `reflectance`- tai `display`-taso kirjoittaa sen edelleen); `daq_tolerance_ms` siirtää kyseisen kaappauksen toleranssirajaa, ei koskaan käytettyä lukemaa. Mikä DAQ: kameraan tai matriisiin sidottu (GUI:n valosensorin valinta, projektin sidonta tai `chloros-cli daq pool-bind`), muuten se, joka on tuorein backendissä. CLI määrittelee komennot `--no-daq` ja `--daq-tolerance-ms` ([CLI-viite](cli-reference.md#the-daq-reading-a-capture-saves)).

```python
r = cam.capture("out", levels=["raw"])      # a CameraSession; ArraySession results match
for m in r.daq_matches:                      # one per .daq in r.daq_files
    print(m["daq_file"], m["gap_ms"], m["in_tolerance"])
```

Tallennus tallentaa käytetyn valosensorin lukeman muodossa `.daq`, ja sen vieressä on `<file>.daq.match.json`, joka kertoo, miten lukema liitettiin valotukseen: `method`, `gap_ms` (valotuksesta lähimpään DAQ-lukemaan) ja `in_tolerance` (500 ms:n sisällä tai tallennuksen `daq_tolerance_ms`). Heijastustiedoston XMP-tiedostossa on sama tulos kuin `Chloros:DlsMatchMethod`, `Chloros:DlsMatchGapS` ja `Chloros:DlsInTolerance`.

Tallennus odottaa DAQ:n ensimmäistä lukemaa valotuksen jälkeen, jotta se voi yhdistää sen molemmin puolin olevat lukemat (`method` `interpolated`). DAQ-E lähettää lukeman päivänvalossa 0,4 sekunnin välein ja sisätiloissa 1–2 sekunnin välein, joten `capture()` palautetaan noin puolitoista kertaa näiden aikavälien jälkeen; altistusta itseään ei viivästetä. Lähettämisen lopettanutta DAQ-laitetta ei odoteta. DAQ-E-lukema on valon keskiarvo edellisen lukeman jälkeiseltä aikaväliltä, joten se sijoitetaan kyseisen aikavälin keskelle: `gap_ms` mitataan kyseiseen keskikohtaan, ja `.match.json`:n`bracket_stamps_ns` antaa lukemien omat aikaleimat. `ArraySession.continuous()` odottaa hidastamatta tallennusta. `capture_repeated()`:n-takaisin-läpikäynnit eivät odota, joten ne säilyttävät tahtinsa ja sovitetaan lähimpään lukemaan ennen valotusta.

Lukema ei vanhene koskaan, joten yhtään kehystä ei menetetä myöhästyneen lukeman takia — sen sijaan se merkitään:

- **DAQ:n virta pysähtyi** — sen uusinta lukemaa käytetään, ja `gap_ms` ilmoittaa, kuinka vanha se on (`in_tolerance: false` yli 500 ms).
- **Ei luettavaa DAQ:ta**— yhteys katkennut (`disconnect()`, `chloros-cli daq pool-disconnect`) tai sidottua DAQ:ta ei tavoiteta — kameran viimeistä lukemaa käytetään uudelleen ja se kuvataan**tämän** valotukseen: `method`, `retained` ja `gap_ms` tästä valotuksesta, `in_tolerance` lasketaan uudelleen, ja `retained_from_epoch_ns` nimeää sen valotuksen, johon se alun perin sovitettiin. Kameralle, jolta ei ole koskaan saatu lukemaa, ei vieläkään anneta heijastavuusarvoa, ja tulos kertoo syyn (`reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, tai kuvan hylkääminen).

Tiedostot, jotka kaappaus tallentaa Chlorosissa avoimena olevan projektin kansioon, liitetään kyseisen projektin tiedostoluetteloon, kuten sovelluksessa tehty kaappauskin (muualle tallennetut tiedostot jätetään rauhaan): `CameraSession.capture()`, `ArraySession.capture()` (myös `smart=True`) ja `capture_cameras_transient()`. Tulos näkyy tiedostossa `.project_add` (`capture_cameras_transient()`: sanakirjan `project_add`) -- `held`, `files` ja taustajärjestelmän `message`; `held` on `True` käsittelyn ollessa käynnissä, ja tiedostot yhdistyvät, kun se päättyy. `None`, kun projektiin ei ole tuotu yhtään tiedostoa.

`CameraSession.capture()` ja `ArraySession.capture()` palauttavat ne tuloksena `.daq_files` ja `.daq_matches`, HTTP-sieppausvastaukset (`/api/camera/<serial>/cli-capture`, `/api/camera/capture`, `/api/camera/capture-all-synced`, `/api/camera/array/capture`) sisältävät `daq_files` ja `daq_matches`, yksi kutakin `.daq`: `daq_file`, `cameras`, `method`, `gap_ms`, `in_tolerance`, `tolerance_ms` ja `stream_fault` — DAQ:n oma virtausvirhe, jota tarkastellaan vain sallitun toleranssin ulkopuolella olevien lukemien yhteydessä. Ennen kuin luotat tallennuksesta saatua heijastavuutta, tarkista `in_tolerance` sen `.match.json`- tai XMP-tiedostosta tai DAQ:n [`status()`](#daqsensorsession-methods).

`processing`-tasot:

| Taso | Lähtö |
| --- | --- |
| `"raw"` | Yksikanavainen Bayer (mustavalkokamerat: yksi kaista) suoraan anturista. |
| `"debayered"` *(SDK:n oletus)* | 3-kanavainen väri: kalibrointipaketin tumma offset (DSNU) poistettu raakadatasta, minkä jälkeen OpenCV:n reuna-herkkä demosaic — ei lineaarisuutta LUT, tasokenttä tai sekoituksen purku (mustavalkokamerat: 1-kanavainen harmaasävy). Tallennettu TIFF-tiedosto on **RGB**-kanavajärjestyksessä (fotometrinen RGB, joten mikä tahansa kuvankatseluohjelma näyttää todelliset värit); muistissa olevat taulukot, jotka `grab()` / `frame_stream()` palauttavat, ovat OpenCV:n**BGR** järjestyksessä — käännä kanavat päinvastaiseksi (`frame[..., ::-1]`), jotta voit verrata tulosta tallennettuun tiedostoon. `chloros-cli process` kirjoittaa saman tuloksen kuin `Debayered_Images`. |
| `"radiance"` | float32 W/m²/sr/nm koko radiometrisen ketjun kautta. Vain monispektrinen — RGB-kamerat ohitetaan. |
| `"reflectance"` | uint16 0..32768 (Pix4D-yhteensopiva); vaatii reaaliaikaisen DAQ-pariliitoksen absoluuttista vertailukohtaa varten. Vain monispektrinen. |
| `"display"` | Täydellinen ketju, joka vastaa käyttöliittymän esikatselua (CCM + WB + gamma kameran profiilin mukaisesti). |
| `"all"` | **Yksi tiedosto kutakin sovellettavaa tasoa kohti** jokaiselle kameralle (vastaa GUI:n ”Capture All” -toimintoa / CLI-oletusasetusta). Palautettu `CaptureResult` sisältää sitten yhden kehysdiktin kutakin `(cam, level)`:ää kohti, ja jokaisessa diktissä on taso; soveltumattomat tasot näkyvät tiedostossa `.skipped`. Käytettyä tallennusta lukeva DAQ tallennetaan sivutiedostona nimellä `.daq`. |

> **Huomautus — oletusarvo eroaa CLI:stä.** `ArraySession.capture()`:n oletusarvona on `processing="debayered"`; komennon `chloros-cli lattice array-capture` oletusarvona on `processing="all"`. Syötä `processing="all"` nimenomaisesti SDK:sta, jotta se vastaa CLI:n/GUI:n monitasoista tallennusta.

##### Taulukon tallennustiedostojen nimet

Jokainen taso tallennetaan omaan alikansioonsa kansiossa `output_dir` (`raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/`, kun kyseessä on `display`, `index/`, `composite/`). Jokaisen tiedoston nimessä on kameran sarjanumero muodossa `SN<serial>-<filter>`: sarjanumero, jonka jälkeen kameran mallin suodatinosa — `SN214000533-FOCN` kameralle `M3C-L87-FOCN`, `SN261902142-F450` mallille `M3M-L87-F450` — yhdistettynä malliin `-`, joten nimessä on samat `_`-erotetut kentät kuin aiemmin. Kamera, jonka mallia ei voida lukea, on yksinään `SN<serial>`, ja aikaisemmista versioista peräisin olevissa kuvissa ei ole suodatinta nimessä. Tiedostonimi riippuu siitä, kuinka monta tasoa jäseneltä pyydettiin:

| Jäseneltä pyydetty | Tiedostonimi | Esimerkki (`reflectance`, oletus `format="tiff"`) |
| --- | --- | --- |
| **Yksi taso** (`processing="reflectance"` tai yhden kohteen `levels`) | `sync_<timestamp>_SN<serial>-<filter>.<format>` — ei tasolisää, kuten aiemmissa versioissa, joten olemassa olevat skriptit toimivat edelleen | `reflectance/sync_<ts>_SN214000533-FOCN.tiff` |
| **Useita tasoja** (`processing="all"` tai `levels`, jossa on useampi kuin yksi kohde) | `sync_<timestamp>_SN<serial>-<filter>_<level>.<format>` | `reflectance/sync_<ts>_SN214000533-FOCN_reflectance.tiff` |

[Clamp-maski](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) seuraa tuotteen nimen perässä, `clamp_masks/<product folder>/<product name>_clampmask.tif`: yhdentason heijastavuusmittaus tuottaa nimen `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`, kun taas `processing="all"` tuottaa nimen `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`. Jälkiliitteen sääntö määräytyy kunkin jäsenen osalta sen mukaan, mitä tasoja sille on pyydetty, joten ohitettu taso (esim. heijastavuus RGB-kamerassa) ei muuta sitä. Lue polut palautetuista kehyksistä sen sijaan, että rakentaisit niitä itse. Yhden kameran tallennus (`CameraSession.capture()`) lisää aina tason: `capture_<ts>_SN<serial>-<filter>_<level>.tiff`.

##### PNG- ja JPEG-tiedostomuodot tukevat vain näyttö- ja hakemistotuotteita

PNG- tai JPEG-tiedostossa on 8 bittiä kanavaa kohti. Jos pyydetään tiedostoa, jonka bittisyvyys ylittää tiedostomuodon salliman rajan, pyyntö hylätään ennen kuvan ottamista, ja näytölle tulee viesti, jossa mainitaan kyseinen bittisyvyys ja TIFF-tiedostomuoto, joka tukee sitä:

- **`radiance`**(float32) ja**`reflectance`**(uint16, `32768` = 1,0) — sekä `processing="all"`, joka sisältää ne. SDK tarkistaa tämän argumenttien perusteella: `capture()` aiheuttaa**`ValueError`** ilman pyyntöä (`CameraSession`, `ArraySession`, `CameraHandle`, `ArrayHandle`, `capture_smart`).
- **`raw` / `debayered` kamerasta, joka lähettää yli 8 bittiä** (12-bittinen M3C, Mono12 M3M) — taustapalvelu hylkää nämä, koska se tietää kunkin kameran bittisyvyyden: HTTP-virhe 400, jossa on `error_code` ja `FORMAT_CANNOT_HOLD_LEVEL`, joka muuttuu virheeksi `ChlorosConnectError` (`CameraSession` / `ArraySession`) tai `RuntimeError` (`CameraHandle` / `ArrayHandle`). Yhden kameran yksitasoinen `debayered`-tallennus kaappaa 8-bittisen kehyksen PNG- tai JPEG-muodossa, eikä sitä hylätä.

Tallenna radiometriset tiedot muodossa `format="tiff"` (`ext=".tiff"`); käytä PNG- tai JPEG-tiedostomuotoa `display`-tiedostoille ja indeksirenderöinneille.

#### Automaattisen valotuksen toiminta

- **Ensin valotus, sitten vahvistus.** Kun vahvistus on Auto-tilassa, vahvistus pysyy 0 dB:ssä, kunnes valotus on saavuttanut sallitun enimmäisarvon.
- **Manuaalinen valotus jättää vahvistuksen automaattitilaan**, kunnes muutat vahvistusta tai asetat sen manuaaliseksi. Kun palaat automaattitilaan, vahvistuksen raja on sama kuin aiemmin.
- **Pienet kirkkaat kohdat eivät tummenna kuvaa.** Kehyksestä enintään 1 %:n suuruinen heijastava alue jätetään huomiotta, ja tunnetun kalibrointikohteen paneeleissa enintään 0,5 % paneelista.
- **DAQ-valotuksen yläraja on voimassa vain silloin, kun kalibrointikohde on näkökentässä.** Se estää valkoisen paneelin leikkautumisen; kun paneelia ei näy, kamera valottaa kuvan kohteen mukaan.
- **Vakio valaistus pysyy vakaana, ja muuttunut valaistus vakiintuu muutamassa sekunnissa.** Vakio valaistuksessa valotus ja sen raja pysyvät ennallaan, ja suuren valaistusmuutoksen jälkeen kuva vakiintuu muutamassa sekunnissa. Ylivalotus ei enää rajoita valotusta pysyvästi, ja kuvausalueen (ROI) muuttuessa valotus säätyy uudelleen.
- **Kirkkaustavoite pysyy saavutettavissa.** Se ei nouse, kun kamera on valotusrajallaan eikä sen vahvistusta voi nostaa, joten palaava valo ei aiheuta valohäiriötä kuvassa.
- **Kamera, jonka automaattinen valotus lakkaa reagoimasta, pidetään senhetkisen valaistustason mukaisena.** Kirkkaassa auringonpaisteessa kamera voi laskea lyhimmälle valotusajalleen ja pysyä siellä tummalla kuvalla, vaikka sen asetukset näyttäisivät normaalilta. Chloros tarkkailee kuvaa, pitää kameran kyseisen valaistustason mukaisena ja vapauttaa sen, kun valaistus muuttuu. Tumma tai peitetty valosensori ei häiritse tätä toimintaa.

#### Valotus matriisin jäsenissä

LATTICE-kamera rajoittaa `ExposureTime`:n yhden kuvataajuusjakson mittaiseksi. Kun matriisi lähettää dataa, Chloros ajaa jokaista jäsentä `AcquisitionFrameRateEnable`-tilassa nopeudella, joka on hieman laukaisunopeutta suurempi, joten jäsenen valotuksen yläraja on hieman laukaisujaksoa lyhyempi **missä tahansa ROI:ssa**:

- Elementin kehysjakso on laukaisujakso vähennettynä 15 %:n liikkumavaraa jaksosta (vähintään 3 ms, enintään puolet jaksosta), ja sen valotuksen yläraja on kyseinen kehysjakso vähennettynä noin 0,1 ms:llä. Esimerkki: 10 fps:n laukaisunopeus (100 ms) antaa jäsenelle nopeuden noin 11,8 fps ja altistumisen ylärajan noin 85 ms.
- **Manuaalinen `ExposureTime`, joka on pidempi kuin kyseinen yläraja, hidastaa laukaisua, jotta se mahtuu siihen.** Chloros avaa ensin kyseisen jäsenen ylärajan kyseiselle arvolle ja antaa hitaamman taajuuden toimia muutaman laukaisujakson ajan (kirjoitus voi kestää muutaman sekunnin kauemmin), kirjoittaa sen sitten, ja laukaisu tapahtuu enintään valotuksen sallimalla nopeudella. Automaattisen taajuuden taulukossa aiempi taajuus palautuu, kun jokainen jäsen, jonka manuaalinen valotus hidasti sitä, on palannut automaattiseen valotukseen. Arvo, jota kamera itse ei voi ottaa, hylätään edelleen, ja sen hyväksymä alue näkyy nyt virheilmoituksessa, ja jäsenen automaattinen valotus palautetaan, jos se oli kytketty pois päältä vain kyseistä kirjoitusta varten. Ennen versiota 1.3.0 pieni ROI rajoitti jokaisen jäsenen anturin omaan vapaakäyntijaksoon, riippumatta laukaisunopeudesta: 4,9 ms resoluutiolla 768 × 304, joten yhtään jäsentä ei voitu kirkastaa.
- Chloros ei koskaan lyhennä manuaalista valotusta nopeamman laukaisun mukaiseksi. Automaattinen taajuus pysyy manuaalisten valotusten sallimissa rajoissa; kiinnitetty taajuus, joka on valotusta nopeampi, jättää kyseiselle jäsenelle riittävän pitkän ylärajan, eikä se ota vastaan laukaisuja, jotka ovat nopeampia kuin valotus sallii. Laske valotusta tai kiinnitettyä taajuutta, kun jokaisen jäsenen on vastattava jokaiseen laukaisuun.
- **Takaisinlukeminen:** jäsenen solmujen lukeminen näyttää `AcquisitionFrameRateEnable` True ja `AcquisitionFrameRate` jäsenen taajuudella; aktivointi kytkeytyy pois päältä, kun matriisi pysähtyy.
- **Järjestelmä omistaa nämä kaksi solmua. Älä kirjoita `AcquisitionFrameRateEnable`- tai `AcquisitionFrameRate`-arvoja järjestelmän jäsenelle** — aseta sen sijaan järjestelmännopeus sen sijaan komennolla `target_fps` / `set_target_fps()`.

Kirjoita jäsenen altistuminen komennolla `arr.set_member_settings(serial, exposure_time=20000)` (avainsanat `CameraSession.set_settings()`; `chloros-cli lattice exposure --serial` toimii samalla tavalla). Jäsen pysyy taulukossa. Älä avaa taulukon jäsentä komennolla `connect_camera()`: connect hylkää kameran, joka on jo taulukossa, ja komento `CameraSession`, jonka se palauttaa, poistaa kameran taulukosta, kun yhteys katkaistaan. Komento `ArraySession.status()["member_ae"]` ilmoittaa jokaisen jäsenenvalotuksen ja vahvistuksen uusimmasta kehyksestä.

### Yksi projektitietue, kaikki tilat

Projektikansio sisältää jokaisen laitteen asetukset: `cameras.json` — kunkin kameran asetukset (`cameras[].settings`, yksittäiselle kameralle ja jokaiselle matriisin jäsenelle) sekä kunkin matriisin koko matriisia koskevat asetukset (`arrays[].capture`: formaatti, ROI, binning, laukaisutaajuus, kaapelibudjetti) — sekä `sensors.json`, joka sisältää kunkin valosensorin asetukset (integraatioaika, kehyksen keskiarvo, yläraja). Sovellus, CLI, SDK ja MCP-palvelin lukevat ja kirjoittavat kaikki samoja tiedostoja samalla tavalla:

- **Projektiin liitetty laite tallennetaan siihen ja tallentaa kaikki muutokset siihen.** ”Projektin alle” tarkoittaa, että projekti on avoinna taustapuolella (sovelluksessa) tai — `ChlorosProject`:n (`open_project()`) kautta, joka määrittää oman kansionsa taustapuolelle (`project_path`). Kamera, anturiryhmä tai valosensori, jota projektissa ei vielä ole, lisätään sen tiedostoihin, kun se kytkeytyy (`connect_camera()`, `connect_array()`, valosensorin kytkentä, CLI, sovellus, `chloros-mcp`) asetuksineen. Muutos mistä tahansa tilasta (`set_settings()`, `ArraySession.set_member_settings()`, `set_target_fps()`, matriisin asetukset, `set_cap()`, CLI, sovelluksen säätimistä) tallennetaan hetken kuluttua kyseisen projektin tiedostoihin. Kameralle asetettu valotus, vahvistus tai AE-kohde tallennetaan pidettynä, joten projektin avaaminen uudelleen säilyttää asetukset.
- **Projektin avaaminen asettaa sen laitteet projektin asetusten mukaisiksi** kaikissa tiloissa: kamera, matriisi tai valosensori, joka on kytketty ilman omia asetuksia, saa projektintallennetut asetukset (`lattice cam-connect` / `array-connect`, kun projekti on auki; `chloros-cli project connect`, `open_project().connect_all()`, sovelluksen uudelleen avaaminen). Uuden tunnisteen alla liitetty matriisi päivittää projektin tallentaman matriisin samoilla kameroilla; matriisi, jossa on muita kameroita, lisätään sen viereen.
- **Sovelluksen oma tallennus ei koskaan korvaa laitteen reaaliaikaisia asetuksia** vanhemmalla kopiollaan, ja se säilyttää jäsenkohtaiset ohitukset, jotka CLI- tai SDK-kutsuja on tallentanut.
- **Asettamasi pikselimuoto on se, jonka projekti säilyttää.** Array Settings -asetuksissa, `--pixel-format`:llä tai `connect_array(pixel_format=)`:llä asetettu muoto, 8-bittinen mukaan lukien, tallennetaan ja kytketään uudelleen täsmälleen samana. Jos mitään ei ole määritetty, kukin kamera muodostaa yhteyden 12-bittisellä oletusasetuksellaan (`BayerRG12`, `Mono12` monokromisensorilla). Muodon nimi tarkoittaa sen bittisyvyyttä; tuoteperhe viittaa anturiin, joten värikameraa ei koskaan aseteta monomuotoon eikä monokameraa Bayerin muotoon, ja rivi `[PixelFormat <serial>] ... written` ilmaisee tämän, kun formaatti on määritetty. 8-bittisen tallennuksen vientitiedostot ovat 8-bittisiä, heijastusarvot mukaan lukien: ne skaalataan uudelleen ja niihin lisätään merkintä `PixelScale`.
- Laite, joka on kytketty ilman projektia, ei tallenna mitään.
- **Tallennus liittyy sen jälkeen luotuun projektiin; avaaminen jättää sen alkuperäiseen projektiinsa.**Kun projekti luodaan — `create_project()`, `chloros-mcp`:n `create_project` tai sovelluksen**Uusi projekti**— tai kopioidaan (sovelluksen**Kopioi**-toiminto, joka avaa kopion) samalla kun tallennus kirjoittaa edelleen aiemmin avoimena olleeseen projektiin, tallennus jatkuu ja tallennettu sisältö päätyy uuteen projektiin, samaan kohtaan sen sisällä (alla). Olemassa olevan projektin avaaminen — `ChlorosProject.process()` (joka avaa projektin taustalla), `chloros-cli process`, sovelluksen**Avaa projekti** — jättää tallennuksen edelleen käynnissä siihen projektiin, johon se tallentaa: se päättyy sinne, ja sen tiedostot liitetään kyseisen projektin tiedostoluetteloon (sovellus pysäyttää kaikki tallennukset ennen projektin avaamista).
  - `ArraySession.record()` ja `ArraySession.burst()`: video tai jokaisen kehyksen sisältävä sarjakuvauskansio siirretään sinne kokonaisuudessaan, kun tallennus lopetetaan. Lopetuksen tulos ilmoittaa uuden polun (`out_dir`, `video_path`), ja sarjakuvien koontiversio luodaan sen pohjalta. Jos siirto epäonnistuu, tallenne jää alkuperäiselle sijainnilleen, ja tulos ilmoittaa tämän (`relocation_error`). Pysäytetyn sarjakuvauksen raakakehykset ja `.daq` lisätään sen projektin tiedostoluetteloon, johon ne päätyvät; videota ei lisätä projektin tiedostoluetteloon.
  - `ArraySession.continuous()`: jokainen muutoksen jälkeen tehty tallennus tallennetaan uuteen projektiin; ennen muutosta tallennetut pysyvät paikallaan.
  - `DAQSensorSession.record_start()` (ja sovelluksen oma valosensoritallenne): `.daq` siirtyy uuden projektin `light_sensor`-kansioon, kun tallennus loppuu; `record_stop()` palauttaa uuden polun, ja tiedosto liitetään kyseisen projektin tiedostoluetteloon.

  Avoimen projektin ulkopuolelle kansioon tallennettu tallenne pysyy paikallaan, ja `start_camera_continuous()` / monivirtaustyöt säilyttävät niille annetun kansion.
- **Sovelluksessa uuden projektin jälkeen tehdyt tallennukset siirtyvät siihen.** Projektin luominen tai kopioiminen pitää kaikki käynnissä olevat tallennusajot ja tallennukset käynnissä, ja jokainen sen jälkeen tehty tallennus (Capture All, Start Capturing) tallennetaan uuteen projektiin ja liitetään sen tiedostoluetteloon. Olemassa olevan projektin avaaminen sovelluksessa pysäyttää ensin kaikki tallennukset.
- **Projektia ei siirretä sen tallennuksen aikana.**Avoimen projektin siirtäminen toiseen työkansioon (sovelluksen**Asetukset → Työkansio**; mikään SDK-kutsu ei siirrä projektia, HTTP-asiakasohjelma, joka lähettää tietoja osoitteeseen `/api/move-project`, siirtää) hylätään HTTP-virheellä 409 `PROJECT_RECORDING`, kun matriisitallennus tai sarjatallennus, matriisin jatkuva tallennus, kameran jatkuva / monivirta-tehtävä tai valosensorin tallennus kirjoittaa siihen: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.`

Tämä on erillinen asia siitä, mihin tilaan kamerat jäävät yhteyden katketessa (katso [Kameran muisti](#camera-memory--keep_settings--clear_settings)): projekti asettaa kameransa itse avautuessaan.

### Kameran muisti — `keep_settings` / `clear_settings`

Kun matriisi kytketään, sen kamerat ottavat matriisin asetukset käyttöön, ja niitä voi sitten muuttaa. Kun Chloros irrottaa kamerat — `ArraySession.disconnect()` / `force_disconnect()`, väliaikaisen toiminnon päättyessä tai `start_camera_continuous()`:n pysähtyessä — se palauttaa jokaisen kameran tilaan, jossa se oli ennen liittämistä: sen laukaisun ja synkronointilinjan johdotuksen, PTP- ja kameran asetukset — `ExposureAuto` / `GainAuto` (sekä manuaalinen `ExposureTime` / `Gain`), `TargetBrightness` sekä AE-rajat ja vaimennus, valkotasapaino, mustataso, `PixelFormat`, pikselien yhdistämistä ja ROI:ta, väriprosessia (`GammaEnable` / `Gamma`, `LUTEnable`, `ColorTransformationEnable`), `ReverseX` / `ReverseY`, `UserSetSelector`, PTP:n `PtpSlaveOnly` / `PtpDomainNumber`, paketin koko (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`), tahdistus (`DeviceLinkThroughputLimitMode` / `DeviceLinkThroughputLimit`, `GevSCPD`, `GevSCFTD`), paladatan (`ChunkModeActive`, kukin `ChunkEnable` omassa `ChunkSelector`:ssään), liipaisusignaalin häiriöiden suodatus, ja taulukko antaa orjalaitteen synkronointilinjan (`LineFilterSelector` / `LineFilterWidth`) sekä kuvausnopeuden (`AcquisitionFrameRate` vain silloin, kun sen käyttöönotto oli päällä: kun se on pois päältä, taajuus lukee suurimman arvon, jonka tahdistus sallii, ei asetusta). Vain eroavat tiedot kirjoitetaan: kamera pysähtyi ensin, tahdistus ja kuvataajuus ennen valotusta, joka vaatii niiden kuvajakson.

Kameran käyttämiseksi projektin ulkopuolella kaksi argumenttia kirjoittavat kameran **omaan muistiin** — sen, millä se käynnistyy, missä tahansa ohjelmistossa (`disconnect()` / `force_disconnect()`, jokaisessa väliaikaisessa toiminnossa, `start_camera_continuous()` ja `ChlorosProject.disconnect_all()`; MCP: samat argumentit kuin `disconnect_device`:ssä ja väliaikaisissa työkaluissa; CLI: `--keep-settings` / `--clear-settings`):

- **`keep_settings=True`** tallentaa kunkin kameran asetukset sellaisina kuin ne olivat istunnon päättyessä kameran muistiin (UserSet2, josta muodostuu käynnistysasetus). Kamerat jätetään edelleen vapaasti toimimaan (`TriggerMode` `Off`), synkronointilinjan johdotus säilyy sellaisenaan. Kamera, jolla ei ole UserSet2-asetuksia, tallentaa asetukset UserSet1:een; tämä tietokone muistaa tämän, joten Chloros-järjestelmän itsensä korjausmekanismi — joka nollaa Chloros-perusarvosta poikkeavan UserSet1-asetuksen — jättää sen rauhaan.
- **`clear_settings=True`** nollaa kunkin kameran muistin tehdasasetuksiin: Chlorosin viitearvoon, johon sen matriisin itsensä korjaus toiminto perustuu — tehdasasetukset komentojen `TriggerMode`, `Off`, `TriggerSource` ja `Line2` avulla, automaattinen valotus sekä RGB-kameran kalibroitu valkotasapaino säilytetään — UserSet1:ssä, josta muodostetaan käynnistysjoukko.
- Ei molempia (`ValueError`; taustapalvelu vastaa 400). Vastaus sisältää `camera_memory`, kamerakohtaisesti `{"saved_to": "UserSet2" | "UserSet1" | None, "errors": [...]}` tai `{"cleared": bool, "errors": [...]}` — `force_disconnect()` palauttaa sen; työn yhteydessä se toimitetaan koodilla `stop()`.

Sovellus ei tarjoa näitä.

**Jokainen poistumistapa palauttaa kamerat.**Löydetyssä muodossa tapahtuva palautus suoritetaan jokaisella poistumisreitillä: `CameraSession.disconnect()` (sovelluksen kamera**X**), sovelluksen**X** taulukossa tai missä tahansa taulukon jäsenessä (koko taulukko vapautetaan), sovelluksen sulkeminen (taustapalvelimella on enintään 20 sekuntia aikaa palauttaa kaikki kamerat, taulukot ja käynnissä olevat jatkuvat tallennukset; vielä käynnissä oleva väliaikainen funktio tai kertakäyttöinen kutsu pysähtyy seuraavaan turvalliseen pisteeseen, palauttaa ensin kameransa, enintään 8 sekunnin ajan, ja vastaa HTTP-koodilla 503 `SHUTTING_DOWN`; edelleen käynnissä oleva `connect_array()` pysähtyy ennen seuraavaa kameraansa ja palauttaa avaamansa kamerat enintään 10 sekunnin ajan ja nostaa `ChlorosConnectError` -virheen (HTTP 503 `SHUTTING_DOWN`); ja sellainen, joka saapuu sammutuksen aikana, ei avaa mitään: HTTP 503 `SHUTTING_DOWN`), sekä `chloros-cli`-komento, joka lopetetaan tai jonka pääte sulkeutuu (sen tehtävä pysäytetään ja sen kamerat palautetaan ensin). Kaatuva tai pakotetusti lopetettu taustapalvelu ei voi palauttaa mitään; virran katkaiseminen ja uudelleenkäynnistäminen palauttaa jokaisen kameran muistiinsa.**Projekti asettaa kameransa itse.** Kun kameraryhmä muodostaa yhteyden projektin kautta — sovelluksen avoimen projektin tai `ChlorosProject`:n kautta, joka ilmoittaa nimensä taustapalvelimelle (`project_path`) — jokainen jäsen saa asetukset, jotka projekti on sille tallentanut: sille määritetyn valotuksen, vahvistuksen tai AE-tavoitteen (pidetään omana, joten automaattinen valotus jättää sen rauhaan) sekä sen valkotasapainotilan (Chlorosin korjaus; kameran oma valkotasapaino pysyy pois päältä), riippumatta siitä, mitä kameran muistissa on.

### Kuvausmoodit ja tallentimet

Taulukon pinta vastaa käyttöliittymän kuvauspaneelia: Yksittäinen / Jatkuva / Intervalli / Nopein suljintila, sekä kaksi tallenninta (live-komposiittivideo ja raakakuva-sarja → jälkikäsittely offline-tilassa).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Continuous — every synced group for 60 s, saved by the backend itself
    with arr.continuous("run1/", processing="raw", duration_s=60) as run:
        run.wait()
    print(run.result["captures"], run.result["captures_per_s"],
          run.result["stop_reason"])

    # Combined-index video/GIF recorder: the GUI's combined view, or with no
    # GUI open the array's NDVI (pass expr="..." for another index)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    if rec.result and rec.result.get("error"):   # leaving the block does not raise
        print("recording failed:", rec.result["error"])
    else:
        print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**on SDK:n jatkuva/väli-silmukka. Koska skriptistä ei voi katkaista sitä `Ctrl+C`:llä, sinun**täytyy** välittää `count` ja/tai `duration_s` (se pysähtyy, kun jompikumpi saavutetaan). `interval_s` mitataan kunkin kierroksen alusta (samoin kuin graafisessa käyttöliittymässä). Jatkuvat kierrokset (`interval_s=0`) ketjuttavat `group_seq` → `after_seq`, joten ne ovat peräkkäisiä synkronoituja ryhmiä taulukonnopeudella. Jäljellä olevat kwargit välitetään suoraan `capture()`:ään.
- **`continuous`**on taustapalvelimen oma jatkuva tallennus — käyttöliittymän Jatkuva-tila.**Käytä sitä `capture_repeated()`:n sijaan, kun haluat tallentaa jokaisen laukaisun levylle korkealla kuvataajuudella:** `capture_repeated()` tekee skriptistäsi yhden HTTP-pyynnön ryhmää kohti, kun taas `continuous()` pitää kaksi tallennusta käynnissä taustapalvelimessa ilman edestakaista viestintää ryhmien välillä. Mitattu ROI 512:lla 9-kamerajärjestelmässä: 73–77 tallennusta/s, kun järjestelmä julkaisee noin 78. Säilytä `capture_repeated()` intervallitallennusta varten (`interval_s`) ja niille läpimenoille, joita tarkastelet yksi kerrallaan. Windows-ympäristössä sulje taustapalvelu Microsoft Defenderin ulkopuolelle, jotta saavutetaan korkeat kuvausnopeudet (katso [Vaihe 1](#step-1--install-the-chloros-platform-package)).
- **`record`** on *valvontatasoinen*: se tallentaa renderoidun reaaliaikaisen yhdistelmäindeksin. Kun GUI:n yhdistetty näkymä on auki, se tallentaa kyseisen näkymän sellaisena kuin se näkyy. Jos mitään näkymää ei ole auki, taustapalvelu renderöi näkymän itse: `expr`, jos annat sellaisen, muuten matriisin NDVI-arvon (punainen ja NIR-kaistat, jotka ovat lähimpänä 660 nm ja 850 nm, kuten reaaliaikainen näkymä ne poimii), kun matriisi on kohdistettu. Nimeä kaistat tiedostossa `expr` matriisin antamien nimien mukaisesti: `<band>_<filter>` M3C-kameralle (`NIR_850_RGN`, `Red_660_RGN`, `Orange_615_OCN`), `Mono_<filter>` M3M-kameralle (`Mono_850`); lyhyt värinimi (`NIR`, `Red`) toimii vain, jos yksi taulukon kaistasta sisältää sen. Tallennuksen aikana avattu GUI-näkymä tallennetaan niin kauan kuin se on auki. Tallennus, joka ei kirjoita yhtään kehystä, epäonnistuu virheillä `error`, `"no-frames: …"` (taulukko saattaa vielä olla kohdistumassa tai vakiinnuttamassa valotustaan), ja `stop()` aiheuttaa virheen (katso [`RecorderHandle`](#recorderhandle)). Yksi yhdistelmätallennin matriisia kohti (aiheuttaa virheen, jos sellainen on jo käynnissä).
- **`burst` → `build_video`** on *analyysitason*: `burst` kirjoittaa raakakuvia + kuvakohtaisen manifestin + yhden `.daq`:n kutakin erillistä DLS-lukemaa kohti `<output>/bursts/<base>/`:n alla tallennussilmukan täydellä nopeudella (ei ketjua, ei exiftool-työkalua, ei reaaliaikaista esikatselua). `build_video` sovittaa kunkin kehyksen ajallisesti lähimpään `.daq`-tiedostoon ja suorittaa tuontiprosessin säteilyvoimakkuus-/heijastavuus-/indeksi-ketjun uudelleen. `products` on luettelo `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}`:sta (oletus: yhdistetty indeksi). `burst().stop()` käynnistää myös automaattisesti parhaansa mukaan laaditun yhdistelmä--indeksin, joka palautetaan lopetustuloksessa nimellä `build_job`.

##### `build_video`-tulos

| Avain | Merkitys |
| --- | --- |
| `success` | `True` vain, kun jokainen pyydetty tuote on rakennettu pyydetyn mukaisesti: vähintään yksi tulos, ei mitään `skipped`:ssä, eikä `degraded`. |
| `outputs` | Tallennettujen videoiden (ja GIF-tiedostojen) polut. |
| `errors` | Ongelmat tekstimuodossa. |
| `skipped` | Yksi `{"serial", "level", "reason"}` kutakin pyydettyä tuotetta kohti, josta **ei** tuotettu videota — kameran (kyseiselle kameralle ei ole kalibrointipakettia, heijastavuudelle ei ole DLS-lukemaa, käsittelyketju epäonnistui) tai yhdistettyjen indeksien, jotka sisältävät `"kind": "combined"`:n ja `serial`:n `None` (kameroiden joukossa ei ole punaisen ja NIR-kaistan paria, ei ryhmää, jossa olisi säteilyarvo molemmille kaistoille). Mikä tahansa merkintä tuottaa `success` `False`; muut videot tallennetaan ja luetellaan edelleen. |
| `substitutions` | Kamerat on renderoitu pyydetystä poikkeavalla tasolla, ja syy tähän — RGB-kameralle, jolle on pyydetty säteilyvoimakkuutta/heijastavuutta, annetaan sen näyttövideo, jonka valkotasapaino on korjattu, kun kameran kalibrointipaketti on käytettävissä. Ei virhe. |
| `notes` | Informatiivinen teksti, ei vikoja — esim. kameralla, jolla ei ole kameraa koskevaa indeksilauseketta, ei ole indeksivideota, tai RGB-kameran näyttövideossa ei ole valkotasapainoa (ei kalibrointipakettia tai paketti, jossa ei ole tehdasasetusten mukaista valkotasapainoa), kameran nimi mainitaan. Ei virhe. |
| `degraded` | `True`, kun tuote on määritelty, mutta se ei ole pyydetty tuote: offline-yhdistetty indeksi on sisäänrakennettu punaisen ja NIR:n normalisoitu ero, joten määritettyä indeksilauseketta ei arvioida. Aiheuttaa virheet `success` ja `False`. |
| `combined` | Yhdistetyn indeksin yhteenveto: käytetyt kaistat, LUT-alue, rakennetut kehykset. |
| `clamp_masks` | Polut [sekoituksenrajoitusmaskeille](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) kunkinkalibroitujen TIFF-tiedostojen [unmix-clamp-maskit](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on), tiedostossa `clamp_masks/<level>/` TIFF-tiedostojen `<level>/`-kansion vieressä, yksi kutakin TIFF-tiedostoa kohti, jossa pikseli on rajoitettu. Videossa ei ole yhtään. |
| `clamp_fractions` | Mitä video kertoo maskin sijaan: yksi `{"kind", "serial", "level", "frames"}` kutakin videotuotetta kohti (`kind` `"combined"` yhdistettyyn hakemistoon kuuluvien kameroiden osalta), `frames` luettelo `[frame, R, G, B]`:istä — kyseisen kehyksen osuus pikseleistä, jotka unmix on rajoittanut sen R/G/B-kanavan kaistalle (`Chloros:Band{R,G,B}UnmixClampFraction`). |

#### `RecorderHandle`

Palautetaan `ArraySession.record()`:n ja `ArraySession.burst()`:n toimesta. Käytä sitä kontekstinhallintana, jotta se pysähtyy automaattisesti, kun skopi sulkeutuu, tai ohjaa sitä manuaalisesti.

| Jäsen | Kuvaus |
| --- | --- |
| `job_id` | Taustatehtävän tunnus (merkkijono). |
| `kind` | `"composite"` (peräisin `record`:stä) tai `"raw"` (peräisin `burst`:stä). |
| `start_stats` | `start`-kutsun palauttama sanakirja. |
| `result` | `None` suorituksen aikana; lopullinen pysäytystulos-sanakirja, kun tallennus on pysäytetty. |
| `stats(timeout=10.0)` | Reaaliaikaiset työn tilastot (kirjoitetut kehykset, toteutunut kuvataajuus, kulunut). |
| `stop(timeout=60.0)` | Pysäytä tallennin; palauttaa ja tallentaa välimuistiin lopullisen tuloksen (`.result` pysyy `None`:nä, jos pysäytyspyyntö itsessään epäonnistui). Jos `record()`-tallennus epäonnistui — eli jos se ei kirjoittanut yhtään kehystä (`error` `"no-frames: …"`) — tulos tallennetaan välimuistiin `.result`:ään, minkä jälkeen syntyy poikkeus `ChlorosConnectError`. Idempotentti (toinen kutsu palauttaa välimuistissa olevan tuloksen ilman poikkeuksen syntymistä). `with`-lohkon ohittaminen pysäyttää tallentimen ilman poikkeuksen syntymistä, joten tarkista `rec.result.get("error")` sen jälkeen. |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

#### `ContinuousHandle`

Palautetaan `ArraySession.continuous()`:n toimesta. Käytä sitä kontekstinhallintana — `with`-lohkon jättäminen pysäyttää tallennuksen, jos se on vielä käynnissä — tai ohjaa sitä komentoilla `status()` / `wait()` / `stop()`. Kerrallaan suoritetaan yksi jatkuva tallennus kutakin taulukkoa kohti.

| Jäsen | Kuvaus |
| --- | --- |
| `start_status` | Käynnistyskutsun palauttama tilasanakirja. |
| `result` | `None` tallennuksen aikana; lopullinen tilasanakirja, kun tallennus on päättynyt (asetettu `stop()`:llä tai sen päättymisen havainneen `status()` / `wait()`:n toimesta). |
| `status(timeout=10.0)` | Reaaliaikainen tilasanakirja (kentät alla). |
| `wait(timeout=None, poll_s=0.5)` | Estää toiminnan, kunnes tallennus päättyy itsestään — `max_captures` tai `duration_s` saavutettu, taulukko tyhjentynyt, levy täynnä, virhe — ja palauttaa lopullisen tilan. Käytettäessä `timeout`-komentoa palautetaan viimeisin tila kyseisen sekuntimäärän kuluttua, vaikka tallennus olisi vielä käynnissä: tarkista `running`. |
| `stop(timeout=30.0)` | Lopeta tallennus ja palauta sen lopullinen tila. Idempotentti (toinen kutsu palauttaa välimuistissa olevan tuloksen). |

Tilakentät:

| Kenttä | Merkitys |
| --- | --- |
| `running` | Tosi, kun tallennus on käynnissä. |
| `captures` | Synkronoidut ryhmät tallennettu. |
| `files` | Tiedostot kirjoitettu. |
| `failed_files` | Tiedostot, joita ei voitu kirjoittaa (kuten kohdassa [`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed)). |
| `failed_captures` | Kokonaan epäonnistuneet tallennusyritykset, mukaan lukien ne, joita yritettiin uudelleen (HTTP-virhe 503, kun taulukossa ei ollut uutta ryhmää ajoissa). |
| `skipped_groups` | Ryhmät, jotka taulukko julkaisi mutta joita ei tallennettu (tallennus jäi jälkeen). |
| `captures_per_s` | Keskimääräinen tallennusaste alusta lähtien. |
| `elapsed_s` / `started_at` | Tähän mennessä kuluneet sekunnit; alkamisaika (Unix-epokki sekunteina). |
| `last_group_seq` | Uusin tallennettu synkronoitu ryhmä. |
| `last_error` | Viimeisin virhe tai `None`. |
| `stop_reason` | Kun päättyy: `count` (`max_captures` saavutettu), `duration` (`duration_s` saavutettu), `stopped` (`stop()` kutsuttu), `array_gone` (taulukko irrotettiin), `error` (20 peräkkäistä virhettä tai taustapalvelimen hylkäämä pyyntö), `disk_full` (tiedosto, jolle levyllä ei ollut tilaa). `None` suorituksen aikana. |
| `max_captures` / `duration_s` | Alkuperäiset rajat (`0` = ei mitään). |
| `defender_cores` | Microsoft Defenderin käyttämät CPU-ytimet viimeisen 10 sekunnin aikana; `None`, kunnes yksi on mitattu, ja aina `None` isäntäkoneella, jossa ei ole Defenderia. |
| `defender_warned` | Arvo on True, kun tallennus on rekisteröinyt, että Defender hidastaa sitä (yhdellä tai useammalla ytimellä). Sulje taustapalvelu pois Defenderin skannauksesta — katso [Vaihe 1](#step-1--install-the-chloros-platform-package). |

```python
run = arr.continuous("captures/run1", processing="raw", max_captures=5000)
try:
    while True:
        st = run.wait(timeout=5)              # returns every 5 s while running
        print(f"{st['captures']} captures, {st['captures_per_s']:.1f}/s, "
              f"{st['skipped_groups']} skipped")
        if not st["running"]:
            break
finally:
    final = run.stop()                        # no-op once it has ended
if final["failed_files"] or final["stop_reason"] in ("disk_full", "error"):
    raise SystemExit(f"continuous capture: {final['stop_reason']}: "
                     f"{final['last_error']}")
```

### Reaaliaikaiset kehykset omassa prosessiketjussasi

Jos haluat syöttää synkronoituja kehyksiä omaan koodiisi — koneoppimiseen, mukautettuun katseluohjelmaan tai omaan tallentimeesi — käytä `ArraySession.frame_stream()`:ää. Jokainen tikki tuottaa `GrabResult`:n: `{serial: numpy array}` jokaiselle kameralle, **kaikki yhdestä laukaisusta**, ja kyseisen laukaisun tunnus on `.group_id`:ssa. Se on sama toteutus kuin projektin API:n `ArrayHandle.frame_stream()`, joten kaikki kohdissa [Yksi laukaisija lukukertaa kohti](#one-trigger-per-read--group_id), [`band`](#exporting-a-single-band--band), [`align`](#aligning-the-streams--align) ja [`bit_depth`](#bit_depth--trading-precision-for-bandwidth) pätee molempiin.

```python
import chloros_sdk
from chloros_sdk import FrameFetchError

serials = ["213800234", "214000533", "214701288", "214701292"]   # master first

with chloros_sdk.connect_array(serials) as arr:
    try:
        for frames in arr.frame_stream(processing="raw", fps=10, duration=300,
                                       with_metadata=True):
            if not frames:              # no camera answered this tick
                continue
            if frames.errors:           # cameras missing from this trigger
                print("left out:", {s: str(e) for s, e in frames.errors.items()})
            for serial, (img, hdr) in frames.items():
                # img: numpy array (raw: one Bayer or mono plane)
                # hdr: group_id, frame_id, timestamp_ns, exposure_us, gain_db, ...
                my_pipeline(frames.group_id, serial, img, hdr["timestamp_ns"])
    except FrameFetchError as exc:      # the stream cannot continue
        print("stream ended:", exc.status, exc)
```

- **Jokainen tuotettu joukko on uusi laukaisija.** `frames.group_id` ei koskaan toistu saman datavirran sisällä; kun pyydät enemmän `fps`:iä kuin taulukko käsittelee kerrallaan, datavirta odottaa seuraavaa laukaisijaa, joten se toimii hitaamman taajuuden mukaan.
- **Puuttuvat kamerat nimetään, niitä ei täytetä jälkikäteen.** Kamera, jota ei voitu palvella kyseisellä laukaisulla, puuttuu sanakirjasta ja luetellaan `frames.errors`:ssä.
- **Se päättyy äkillisesti.** `FrameFetchError` toistuu hylkäyksen yhteydessä seuraavalla tikillä, 60 sekunnin kuluttua ilman kehystä tai 60 sekunnin kuluttua ilman uutta laukaisua.**Mitä se tuottaa.**`frame_stream()` siirtää kokonaisia numpy-kehyksiä paikallisen HTTP-yhteyden kautta ja saavuttaa huippunopeuden lähellä**170 MB/s**. Tämä riittää raakakehysten, binnattujen tai ROI-taulukoiden reaaliaikaiseen käsittelyyn, mutta ei jokaiseen laukaisuun täydellä resoluutiolla —**jokaisen** laukaisun säilyttämiseksi tallenna levylle komennolla [`continuous()`](#continuoushandle). Mitattu nelikamerajärjestelmällä (FOCN, FRGN, FRGB, F450) pakatulla siirrolla; jokainen tikki oli täydellinen yhden laukaisun sarja:

| | Koko kenno (matriisi 14,2–14,4 fps) | 2 × 2-binning (matriisi ~33 fps) |
| --- | --- | --- |
| `frame_stream(processing="raw")` | 6,7 sarjaa/s | 22,2 sarjaa/s |
| `frame_stream(processing="debayered")` | 2,3 sarjaa/s | 9,3 sarjaa/s |
| `frame_stream(processing="display")` | 0,8 sarjaa/s | 3,0 sarjaa/s |
| Kameraa kohti MJPEG, 4 videovirtaa 720p-tarkkuudella | 13,5 kuvaa sekunnissa kukin; jokainen ruutu liitetään laukaisijaan | ~25 kuvaa sekunnissa kukin; ~45 % laukaisijoista läsnä kaikissa 4:ssä |

Käytä `processing="raw"`:ää läpimenon ja demosaikin suorittamiseen omalla puolellasi; `debayered`-taso poistaa myös kalibrointipaketin tumman offsetin, mitä `raw`-demosaikki ei tee.

#### MJPEG-virrat kameraa kohti (ei RTSP)

**RTSP:tä ei tueta.** Suoratoistoliitäntä on MJPEG HTTP:n kautta (`multipart/x-mixed-replace`, yksi JPEG per osa, jokaisessa osassa `Content-Length`), yksi URL per kamera:

| URL | Mitä se tarjoaa |
| --- | --- |
| `GET /api/camera/stream/<serial>` | 8-bittinen esikatselu, enintään ~30 kuvaa sekunnissa. Ryhmän jäsenen kohdalla `max_h` (oletus 720, 64–2160) ja `jpeg_q` (oletus 80) määrittävät koon ja laadun. |
| `GET /api/camera/<serial>/stream-annotated` | Sama esikatselu, johon on piirretty käyttöliittymän päällekkäiskuvat (zebra, ruudukko, hiusristikko, histogrammi, peaking, pistevalotusmittari). `ChlorosLocal.camera_stream()` lukee sen puolestasi. |
| `GET /api/camera/<serial>/stream-reflectance` | 8-bittinen heijastavuus kaistakohtaisesti — katso [Reaaliaikainen heijastavuusvirta](#live-reflectance-stream-mjpeg). |

- **Laukaisun otsikot.** Taulukon jäsenen osalta jokainen osa kaikista kolmesta datavirrasta sisältää `X-Chloros-Group-Id`, `X-Chloros-Frame-Id` ja `X-Chloros-Timestamp-Ns`. Eri kameroista peräisin olevat osat, joilla on sama `X-Chloros-Group-Id`, kuuluvat samaan laukaisijaan: kerää osat ja ryhmittele ne kyseisen otsikon perusteella, jotta sarja voidaan koota uudelleen. Itsenäisellä kameralla ei ole ryhmää: sen esikatseluosat eivät sisällä mitään näistä otsikoista (sen heijastavuusosissa on tyhjä `X-Chloros-Group-Id`).
- **Esikatseluvirta pudottaa kehyksiä kuormituksen alla.** Se toimittaa uusimman kehyksen ja ohittaa ne, joita se ei ehtinyt lähettää ajoissa, joten suurilla matriisinopeuksilla kaikki laukaisijat eivät saavuta jokaisen kameran virtaa (katso yllä oleva taulukko). Käytä jokaiselle laukaisijalle `frame_stream()` tai `continuous()`.
- **`/api/camera/stream/<serial>` keskeytyy, kun matriisin yhdistetty livestriimi on auki** (`/api/camera/array/stream/<array_id>` — käyttöliittymän matriisinäkymä avaa sen). Kunkin jäsenen lähetys lähettää tällöin yhden ”Paused (array stream active)” -paikkamerkki-kehyksen ilman `X-Chloros-*`-otsikoita, eikä mitään muuta, kunnes yhdistetty lähetys sulkeutuu; sen jälkeen se jatkuu itsestään. `stream-annotated` ja `stream-reflectance` eivät keskeydy. Sulje käyttöliittymän ryhmänäkymä, kun skripti lukee kamerakohtaisia esikatselulähetyksiä.
- **Käytä numeroiden, ei kuvien, kanssa `frame_stream()`:ää.**JPEG-tiedostot ovat häviöllisiä ja 8-bittisiä; `frame_stream()` palauttaa samat tulokset ilman koodekkia polussa.

### Intervallikuvaus drone-kartoituksissa

Kartoituslennolla tarvitaan yksi synkronoitu kuvaryhmä muutaman metrin välein, ei jokaista kuvaa, jonka matriisi pystyy tuottamaan. Tätä ohjaavat kaksi asetusta:**`target_fps`**määrittää, kuinka usein kamerat ottavat kuvan, ja**`capture_repeated(interval_s=…)`** määrittää, kuinka usein kuva tallennetaan.**Kamerat jatkavat kuvien ottamista tallennusten välillä.** Jokaisella aikavälillä tallennetaan uusin synkronoitu ryhmä; välissä olevia kuvia ei tallenneta. Automaattinen valotus säätyy jokaisella kuvalla, joten valotus on jo vakiintunut tallennuksen tapahtuessa. Ei ole olemassa tilaa, jossa kamerat pysähtyisivät kuvien välillä ja heräisivät jokaisen kuvan ottamista varten: jokaisen kuvan ottamisen yhteydessä jouduttaisiin silloin odottamaan, että kuvavirta alkaa ja valotus vakiintuu.**Aseta kuvataajuus aikavälin vaatimusten mukaiseksi.**Automaattisella asetuksella järjestelmä toimii isäntäkoneen ja verkon sallimalla suurimmalla taajuudella. Taajuuden laskeminen vähentää kameran lämpenemistä, verkon kuormitusta ja isäntäkoneen prosessorin kuormitusta muuttamatta tallennettavia tietoja. Valitse taajuus, joka tuottaa**2–4 kuvaa aikaväliä kohti**:

- Jokainen tallennus odottaa seuraavaa kuvaa sen pyytämisen jälkeen, joten tallennettu ryhmä saapuu jopa yhden kuvan jakson verran myöhässä aikataulustaan. Korkeampi taajuus pitää tämän viiveen pienenä.
- Automaattinen valotus säätyy kerran kuvaa kohti, joten hyvin matala taajuus hidastaa valotuksen reagoimista ohikulkevaan pilveen.

**Laske intervalli lentosuunnitelman perusteella:**

```
spacing    = along-track footprint × (1 − forward overlap)
interval_s = spacing / ground speed
```

Esimerkiksi 60 metrin peittoalue 80 %:n eteenpäin suuntautuvalla päällekkäisyydellä tarkoittaa 12 metrin väliä; nopeudella 10 m/s se on `interval_s=1.2`, ja kiinteä `target_fps=3` antaa 3–4 kuvaa intervallia kohti. Kuvaus on aikaperusteista: SDK:ssa ei ole etäisyys- tai GPS-laukaisutilaa, joten pidä tasainen maanopeus mittauslinjoilla.

**Tallenna raakadata lentokoneeseen ja kalibroi myöhemmin.** `interval_s` mitataan kunkin tallennuksen alusta, joten jos tallennus kestää pidempään kuin aikaväli, seuraava tallennus alkaa myöhässä ja etäisyys vaihtelee. Raakadatan tallentaminen yhdessä valosensorin lukeman kanssa (`processing="raw"`; lukema tallennetaan oletuksena) pitää jokaisen tallennuksen lyhyenä ja varmistaa, että jokainen kehys voidaan myöhemmin muuntaa heijastusarvoksi.

```python
import chloros_sdk

INTERVAL_S = 1.2      # from the flight plan
FLIGHT_S = 20 * 60    # bounds the loop; it also stops at count= if given

with chloros_sdk.connect_array(serials, target_fps=3.0) as arr:
    groups = arr.capture_repeated(
        "survey/", interval_s=INTERVAL_S, duration_s=FLIGHT_S,
        processing="raw", render_index=False,
        on_capture=lambda i, r: print(f"group {i}: {len(r)} frames"))
```

- **`connect_array(target_fps=…)`**lukitsee taajuuden alusta alkaen;**`arr.set_target_fps(fps)`**muuttaa sitä lennon aikana ilman uudelleenyhteyden muodostamista, ja**`arr.set_target_fps(None)`** palauttaa sen automaattitilaan. Nopeus, joka ylittää isäntäkoneen ja verkon siirtokapasiteetin, lasketaan kyseiseen ylärajaan — tarkista tulos kohdasta `clamped` — ja pienin nopeus on 0,1 fps.
- **Älä käytä `burst()`:ää kehysten harventamiseen.** Se kirjoittaa jokaisen kehyksen, jonka taulukko tuottaa; se noudattaa kiinnitettyä `target_fps`:ää, joten se on oikea työkalu vain silloin, kun jokainen kehys halutaan tallentaa.
- **`target_fps` ei korjaa vioittuneita kehyksiä.** Arvon pienentäminen vähentää tuotettujen kehysten määrää, ei yksittäisten kehysten lähetysnopeutta; GVSP-virheellisten kehysten tapauksessa pienennä sen sijaan `wire_ceiling_mbps`-arvoa (katso [Array Health](#array-health--which-subsystem-is-losing-frames)).

```python
r = arr.set_target_fps(5.0)
if r["clamped"]:
    print(f"asked for {r['requested']} fps, running at {r['target_fps']}")
```

### Liittyminen jo kytkettyyn taulukkoon — `attach_array`

Jos taulukko on jo käynnissä (GUI on avannut sen tai edellinen SDK-istunto on kutsunut `connect_array`:ää), käytä `attach_array`:ää saadaksesi siihen käsittelykäsitteen uudelleenkytkemisen sijaan. `connect_array` antaa tuossa tilanteessa aina virheilmoituksen ”Camera  <sn>on jo matriisissa<id>”, koska `/array/connect`-komennon lähettäminen matriisin jäsenelle ei ole idempotentti; `attach_array` lukee `/api/camera/array/list`:n ja tekee vertailun joko array_id:n tai sarjanumeroiden perusteella.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Malli: SDK-skriptit, jotka toimivat rinnakkain työpöytäkäyttöliittymän kanssa, tulisi kokeilla ensin `attach_array`:ää ja siirtyä varasuunnitelmana `connect_array`:ään, jos poolissa ei vielä ole yhtään taulukkoa.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Liitetty array on lainattu, ei omistettu.** `attach_array`:n käsittelykäsite ei koskaan pura taulukkoa: sen vapauttamista ei tapahdu `with`-lohkon päättyessä eikä eksplisiittisellä `disconnect()`-kutsulla (eksplisiittinen kutsu tulostaa yhden rivin stderr:iin, jossa ilmoitetaan, että jättänyt taulukon käynnissä). Sama sääntö kuin `CameraSession` / `DAQSensorSession`. Joten yhteiskäyttö GUI:n kanssa on turvallista riippumatta siitä, käytetäänkö `with`:ää vai ei:
>
> ```python
> with chloros_sdk.attach_array(serials) as arr:
>     arr.capture("output/", processing="reflectance")
> # … the array stays up for the GUI
> ```
>
> Kun haluat lopettaa sen – esimerkiksi muodostaaksesi uuden yhteyden eri `wire_ceiling_mbps`:n kanssa – kutsu `arr.force_disconnect()`. Uudelleenyhteyden muodostaminen liitetyn kahvan tavallisen `disconnect()`-kutsun jälkeen epäonnistuu virheilmoituksella &quot;Kamera on<sn> jo taulukossa<id>&quot;, koska mitään ei ole vapautettu.

### Verkkoanalyysin apuohjelma

Hyödyllinen ennen taulukon avaamista — arvioi, sopivatko ehdotetut asetuksesi:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    # width / height are sensor pixels BEFORE binning (what connect_array
    # takes with binning=); out_width / out_height is what is streamed.
    print(f"Connect with width={r['width']} height={r['height']} "
          f"binning={r['binning']} ({r['out_width']}x{r['out_height']} streamed)")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

```python
analyze_array_network(
    master_serial,
    slave_serials=None,
    width=2048, height=1536,              # sensor pixels before binning
    pixel_format=None,                    # None = what connect applies (Mono12 / BayerRG12)
    binning=1,                            # 1 | 2 | 4
    force_tier=None,
    packed="auto",                        # "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",
    timeout=30.0,
) -> dict
```

Se ei koskaan käynnistä taustapalvelinta: palvelin, joka ei vastaa, aiheuttaa virheen `ChlorosConnectError`, kun taas hylätty pyyntö palautetaan tuloksena. Virheellinen `packed` aiheuttaa virheen `ValueError`.

`status` on yksi seuraavista: `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (muussa tapauksessa `error` — mukaan lukien hylätty pyyntö: ei kirjautunut sisään, `error_code: AUTH_REQUIRED` tai SDK-vähimmäisrajan alittava paketti, `PLAN_UPGRADE_REQUIRED`). `auto_capped_fps` tarkoittaa, että pyydetty resoluutio sopii RX-renkaaseen vain rajoitetulla laukaisutaajuudella — säilytä resoluutio ja siirry `target_fps=result["recommended"]["recommended_target_fps"]`:stä `connect_array`:ään (katso [Esimerkki 6](#6-capability-probe-before-connecting-a-4-cam-array)). `auto_shrunk` ilmestyy vain, kun pyydettyä kehystä ei voida toimittaa millään kehysnopeudella; connect ei koskaan pienennä kehyksiä itsestään, joten sovella `recommended` itse.

**Kuinka tulkita projektio** (sama malli kuin GUI:n Array Settings -paneelissa):

- **Burst (`frame_bytes_total`) lasketaan yhteen kamerakohtaisesti kunkin kameran todellisessa pikselimuodossa.**Mono**M3M**-kamerat lähettävät Mono12-muotoista dataa (2 bittiä/pikseli) tai, kun `pixel_format` on käytössä, syvyystietojen Mono-muotoa (`"BayerRG8"` on `Mono8`, 1 bitti/pikseli), joten ilman sitä nelikamerainen täysresoluutioinen kehys on**~25 MB** kolmella monokameralla, ei ~12,6 MB, kuten oletettaisiin, jos kaikki olisivat 8-bittisiä. Taustaprosessi määrittää kunkin kameran formaatin sen mallin perusteella.
- **Admittance (`burst_fits_nic_ring`) ottaa huomioon tyhjennysnopeuden**, ei koko-burst-vs-ring: sim-emit sopii tilanteeseen, jossa isäntä tyhjentää RX-renkaan nopeammin kuin kamerat täyttävät sitä. 10G-isäntä + 1 GbE-kamerat**sallivat** täyden resoluution jopa silloin, kun burst ylittää renkaan kapasiteetin; 1 GbE:n isäntä estää sen (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` on konservatiivinen sarjaluvun hakuraja** — `max(readout+emit, N×emit)`, jossa kamerakohtainen lähetys on rajoitettu 1 GbE:n kamerayhteyteen, valotuksesta riippumatta. Esim. ~2,8 fps nelikameraiselle täysresoluutioiselle 12-bittiselle matriisille (vastaa ajon aikana mitattua ~2,7–3,0). Täydellinen malli: [CLI-viite → Järjestelmän kuvanopeus- ja sarjakuvausmalli](cli-reference.md#array-fps--burst-model).
- **Jatkuvan tallennuksen jälkeen ennuste kertoo, mihin arvoon se vakiintui.** Vähintään 30 sekunnin tallennus automaattisella nopeudella tallentaa sen nopeuden, johon se vakiintui kyseisellä matriisikoolla tällä tietokoneella; nopeuden säätimen lyhyet kokeilut nopeammalla nopeudella jätetään pois. Siitä lähtien `array-connect`:nrivi, `connect_array()`:n analyysi ja Array Settings -paneeli mainitsevat kyseisen nopeuden, kun se on mallin nopeutta alhaisempi, sekä sen, mikä rajoitti sitä kohdassa `cycle_limiter`: `"clock"` (matriisi toimi vakiintuneella nopeudella, ja tallennussilmukka odotti laukaisukelloaan), `"loop"` (tallennussilmukalla ei ollut enää aikaa), `"capture"` (kaappaus jäi jälkeen ja ohitti ryhmiä) tai `"measured"` (ei mainita kumpi). Mallin arvo on kohdassa `wire_fps_max`. Taulukko nousee edelleen siihen, mitä kaapeli kantaa.
- **Ylimerkintä (`oversubscribed: true`) tarkoittaa, että kaapeli ei kykene siirtämään taulukkoa millään kamerien saavuttamalla tahdilla** — fps-kentät (`achievable_fps_max` / `fps_bright` / `fps_dark`) lukevat arvoa 0, eikä binningillä voida korjata tilannetta (se vähentää kehyksen sisältämiä tavuja, ei tahdistettujen tavujen määrää sekunnissa). Pelkkä hidas kaapeliyhteys ei aiheuta tätä: sen sijaan kuvataajuus laskee, ja jos kameroiden tahdistusta ei voida hidastaa tarpeeksi, tasoksi muuttuu `slip-emit-and-capture` (sarjaliikenne). Korjaustoimenpiteitä ovat kameroiden määrän vähentäminen, jumbo-kehykset tai nopeampi verkkokortti; `max_cams_collision_safe` ilmoittaa kaapelisi ja kameroidesi enimmäiskapasiteetin. Vastaus sisältää myös koodit `aggregate_demand_bps`, `collision_safe_ceiling_bps` ja `per_cam_floor_bps`. Katso [Ylimerkintä](#over-subscription-the-per-cam-floor).

### Löytäminen ja luettelointi

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

Kumpikin ottaa vastaan vain avainsanoja sisältäviä `backend_url=`- ja `timeout=30.0`-pyyntöjä ja palauttaa sanakirjojen luettelon. `discover_lattice_cameras()` (`serial`, `ip`, `mac`, `model`, `lattice_model`, `vendor`, `connected`, `array_id`, …) ottaa myös vastaan `auto_start_backend=True`; `list_*`-kutsut eivät koskaan käynnistä taustaprosessia.

---

## Smart-AE / Smart-Capture

LATTICE-matriisit suorittavat jatkuvaa automaattista valotusta (AE) taustalla heti, kun ne on kytketty, mutta vastikään kohdistetun kuvauskohteen konvergoituminen kestää hetken. **Smart-capture** on kätevä valmisratkaisu: se tarkistaa kunkin kameran valotuksen, odottaa, kunnes matriisi on vakaa koko ikkunassa, ja käynnistää sitten kuvauksen. Se vastaa graafisen käyttöliittymän toimintoa: työpöytäsovelluksen ”smart”-kuvauspainike kutsuu samaa taustaprosessin päätepistettä.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Kun käytät `ChlorosProject`-komentoa (seuraava osa), saat käyttöösi lisää säätimiä:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Älykäs automaattinen valotus (smart-AE) on oletusarvoisesti konservatiivinen. Kiristä asetusta `exposure_tolerance_pct` tarkkoihin radiometrisiin töihin; löysää sitä nopeasti muuttuvissa tilanteissa, joissa riittää, että tulos on ”riittävän lähellä”.

---

## DAQ-anturisessiot

Pysyvä taustapooli spektrisensoreille (DAQ-U USB:n kautta, DAQ-M BLE:n kautta, DAQ-E Ethernetin kautta). Vastaa kameran pintaa: älykäs tunnistus, poolin uudelleenkäyttö, idempotentti liittäminen.

### Älykäs tunnistus (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Etusija: Ethernet → BLE → USB. Määritä mikä tahansa eksplisiittinen vihje siirtotavan kiinnittämiseksi.

### Kiinnitetty siirtotapa

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# A specific sensor by id (as list_daq_sensors() shows it; ':' or '-' alike)
daq = chloros_sdk.connect_daq_sensor(sensor_id="CB-7C-A7-5A-73")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

`sensor_id=` nimeää yhden anturin, ja vain kyseinen anturi avataan koskaan. Sitä verrataan jo avoimiin antureihin, minkä jälkeen DAQ-E:n isäntänimeltä tai IP-osoitteelta (verkko-skannaus havaitsee DAQ-E:n vain IP-osoitteen perusteella, ja siltä kysytään, vain luku -tilassa, sen nimeä, MAC-osoitetta ja sarjanumeroa, joten mikä tahansa näistä löytää sen myös), DAQ-M:n BLE-MAC-osoitetta sekä tunnistetta, jonka DAQ-U tai DAQ-M ilmoittaa avautuessaan (kaikki muut avatut laitteet suljetaan uudelleen tarkistuksen jälkeen). Jos laitetta ei löydy, `connect_daq_sensor` nostaa `ChlorosConnectError`-ilmoituksen, jossa luetellaan löydetyt kohteet; mukaan lukien osoite (`port=` / `mac=` / `eth_host=`), joka sisältää eri anturin, se laukaisee myös poikkeuksen. Se ei koskaan palaa siihen anturiin, joka on näkyvissä — sitä varten on nollalippukutsu.</id></sn></id></sn>

### `connect_daq_sensor()`-tunniste

```python
connect_daq_sensor(
    *,                                    # every argument is keyword-only
    sensor_id=None,                       # one named sensor (see above)
    transport=None,                       # "usb" | "ble" | "eth"; None = smart-detect
    port=None,                            # DAQ-U serial port ("COM3", "/dev/ttyUSB0")
    mac=None,                             # DAQ-M BLE MAC (implies transport="ble")
    eth_host=None,                        # DAQ-E hostname or IP (implies transport="eth")
    integration_time=None,                # ms; None = the sensor's own
    frame_avg=None,
    enable_ae=None,
    cap_id=None,                          # cap correction: "sunshine_cosine", "fov_30", ..., "none" (bare)
    start_streaming=True,
    backend_url="http://127.0.0.1:5000",
    timeout=60.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> DAQSensorSession
```

Uima-altaassa jo oleva anturi palautetaan sellaisenaan (lainattuna): `integration_time`, `frame_avg` ja `enable_ae` eivät koske sitä. Irrota se ensin, jotta voit avata sen omilla asetuksillasi. `cap_id` on poikkeus: lainatussa anturissa se asetetaan `set_cap()`:llä. Jos se jätetään pois, käytetään taustapalvelimen oletusarvoa (katso huomautus menetelmätaulukon alla).

### `DAQSensorSession`-menetelmät

| Menetelmä | Kuvaus |
| --- | --- |
| `status(timeout=10.0)` | Pool-merkinnän yhteenveto (suoratoisto-/tallennustila, aallonpituusalue, kalibrointitunnus, integraatioaika, frame_avg, AE-tila). DAQ-E:lle, jonka virta on katkennut: `reconnect` (jos taustapalvelimen automaattinen uudelleenkytkentä on käytössä: `state`, `attempts`, `next_attempt_in_s`, `last_error`; `None`, kun se lähettää dataa), `stream_lost_s`, `reconnects` ja `last_reconnect` sekä `record_gaps` käynnissä olevan tallennuksen osalta; `rx_stream_fault` ilmaisee uudelleenkytkentätilan, kunnes ensimmäinen spektri saapuu. |
| `latest(*, n=1, timeout=10.0)` | Palauttaa enintään N viimeisintä spektrikehystä (kukin `ts_ns`, `spectrum`, `is_saturated`, `integration_time`, `x`, `y`, `z`). |
| `stream_start()` / `stream_stop()` | Jatka / keskeytä suoratoisto (käsittelytunnus pysyy auki). |
| `record_start(*, output_dir=None, device_name=None, timeout=15.0)` | Aloittaa .daq-tiedoston tallennuksen. Palauttaa tiedostopolun. Ei toimi DAQ-U/M-laitteissa, joissa ei ole AWS-kalibrointipakettia (DAQ-E on poikkeus). |
| `record_stop()` | Lopeta tallennus. Palauttaa `{path, rows, warning, gaps}`; `warning` on `None`, jos tallennus on onnistunut, muussa tapauksessa taustapalvelimen diagnoosi (esimerkiksi tyhjä tiedosto tai aukko datavirrassa), joka tulostetaan myös stderr:ään. `gaps` luettelee jokaisen jakson, jolloin virrasta ei saatu mitään: `start_utc`, `end_utc` (`None`: edelleen kadoksissa lopetuksen yhteydessä), `duration_s`, `reconnected`, `marked_in_file`. Jatkuvassa tallennuksessa tämä on tyhjä. |
| `set_cap(cap_id, timeout=15.0)` | Aseta ylärajan korjausprofiili, jota sovelletaan seuraavasta lukemasta alkaen: `"sunshine_cosine"`, `"fov_15"` ... `"fov_90"` tai `"none"` (tai `None`) paljaan anturin tapauksessa. Palauttaa anturinyhteenvedon, jonka `cap_id` lukee takaisin. DAQ-E-S pitää `as_recorded`:n riippumatta siitä, mitä kysytään (sen diffuusori on sisäänrakennettu). Nostaa `ChlorosConnectError`:n profiilille, joka ei lataudu. CLI-komento: `chloros-cli daq pool-set-cap`. |
| `enable_ptp(timeout=15.0)` | Vain DAQ-E: kytkee PTP:n takaisin päälle laitteessa, jonka PTP on pois päältä (`status()["rx_ptp_enabled"]` on `False`; asetus tallennetaan laitteeseen, joten se pysyy pois päältä, kunnes se kytketään takaisin päälle). Palauttaa päivitetyn tilan. Nostaa `ChlorosConnectError`-virheen DAQ-U- tai DAQ-M-laitteessa tai kun laite ei vahvista. Chloros-ohjelmisto ei koskaan kytke laitetta takaisin päälle itsestään, koska pois päältä -tila voi olla tarkoituksellinen. CLI-komento: `chloros-cli daq pool-enable-ptp`. |
| `device_info(timeout=15.0)` | Vain DAQ-E: laitteen omat asetukset. `firmware` (`running`, `bundled`, `update_available`), `ip` ja `ip_mode` (`dhcp` / `linklocal` / `static`: mitä se nyt käyttää), `network` (tallennettu asetus: `mode` sekä `ip` / `mask` / `gateway`, kun laite on pysähdyksissä), `network_change_pending` (tosi, kunnes laite käyttää tallennettua asetusta, johon sisältyy `set_network(..., reboot=False)`), `auto_stream`, `ptp`. `firmware["update_available"]` on `None`, kun laitteenversiota ei voitu lukea. Kun laitteen datavirta ei saavuta tätä tietokonetta, se luetaan uuden yhteyden kautta ja sanakirja lisää `stream_silent` (`True`), `stream_fault` ja `reconnect`; laite, joka ei vastaa, aiheuttaa virheen. CLI: `pool-info`. |
| `set_network(mode="auto", *, ip=None, mask=None, gateway=None, reboot=True, force=False, comeback_timeout=45.0)` | DAQ-E vain: DHCP (`"auto"`) tai staattinen osoite. Käynnistää laitteen uudelleen asetusten voimaansaattamiseksi ja muodostaa yhteyden uudelleen; tämä istunto jatkaa toimintaansa ja `address` seuraa laitetta. `reboot=False` tallentaa asetukset vain. Katso [DAQ-E-laitteen asetukset](#daq-e-device-settings). CLI-komento: `pool-set-network`. |
| `set_auto_stream(enabled, timeout=15.0)` | Vain DAQ-E: lähetys käynnistyksen yhteydessä ilman kytkettyä ohjelmistoa (tallennetaan laitteeseen). Palauttaa vahvistetun asetuksen. CLI: `pool-set-auto-stream`. |
| `reboot(*, comeback_timeout=45.0)` | Vain DAQ-E: käynnistä laite uudelleen ja liitä se uudelleen aiemmilla asetuksilla. CLI: `pool-reboot`. |
| `update_firmware(*, comeback_timeout=45.0)` | Vain DAQ-E: päivitä tämän Chloros-julkaisun mukana toimitettu laiteohjelmisto laitteeseen, jossa on vanhempi versio, ja liitä laite sitten uudelleen (kestää noin minuutin; älä sammuta laitetta). Palauttaa virhekoodit `updated`, `fw_before`, `fw_after`, `bundled` ja `warning`, jos laite palautui hitaasti tai palautui eri versiolla; `updated` on `False` laitteelle, jonka versio on jo ajan tasalla. Lähettää virheen `ChlorosConnectError` laitteelle, jonka versiota ei voida lukea. CLI-komento: `pool-update-firmware`. |
| `disconnect()` | Vapautus poolista. Eiliitettyjen, mutta omistamattomien kahvojen kohdalla. Tämän istunnon avaamassa anturissa se on `force_disconnect()`, ja se aiheuttaa virheen `ChlorosConnectError`, kun vapautusta ei vahvisteta (`with`-lohkon poistuminen ei koskaan aiheuta poikkeusta). |
| `force_disconnect(timeout=30.0)` | Vapauta anturi riippumatta siitä, kuka sen avasi — esimerkiksi anturi, jonka kaatunut skripti on jättänyt pooliin ja johon uusi skripti pääsee vain liitettynä kahvana, jonka `disconnect()` ei tee mitään. Toisen asiakkaan anturin lopettaminen päättää sen tekemän tallennuksen. Palaa, kun taustapalvelu vahvistaa vapautuksen tai kun vapautettavaa ei ole (anturia ei ole poolissa tai taustapalvelinta ei ole käynnissä); aikakatkaisuun päättynyt vapautus tai taustapalvelimen hylkäämä vapautus aiheuttaa virheen `ChlorosConnectError`. CLI: `chloros-cli daq pool-disconnect --sensor-id ID`. |

Ominaisuudet: `sensor_id`, `model`, `transport`, `address` (seuraa DAQ-E:tä, jonka `set_network()` osoitti uudelleen), `info` ja `backend_url`.

> **Nimeä suojus.** DAQ-U-, DAQ-M- ja DAQ-E-laitteet toimitetaan ilman kondensaattoria, eikä Chloros voi nähdä, onko sellainen asennettu: ilman `cap_id`:ää kytketty anturi saa arvoksi `sunshine_cosine`, mikä on oletus, jonka sen tallenteiden mukaan se on tehnyt (`cap_id_source=auto_default`). Ilman korjauskerrointa paljas DAQ-U näyttää noin 30-kertaisen lukeman, joten syötä `cap_id="none"` paljaalle anturille (tai `set_cap("none")`), ja muussa tapauksessa asennetun suojuksen profiilin.

### DAQ-E-S — kiinteä hajotin ja kallistusanturi

**DAQ-E-S** on DAQ-E, jossa on kaksi ohjelmiston kannalta merkittävää eroa. Kaikki muu — laitteen tunnistus, liittäminen, PTP, tallennus sekä alla olevat laiteasetukset — toimii täsmälleen samalla tavalla kuin DAQ-E:ssä.**Sen hajotin on kiinteä, joten se ei vaadi suojuksen korjausta.** Sunshine-tyyppinen diffuusori on sisäänrakennettu ja oli laitteessa jo silloin, kun sen tehdaskalibrointi mitattiin, joten kalibrointi osoittaa jo todellista alaspäin suuntautuvaa säteilyvoimakkuutta. Chloros asettaa cap-arvokseen `as_recorded` (ei profiilia) riippumatta siitä, mitä pyydetään: cap, joka on nimetty sen mukaan (`sunshine_cosine`, `fov_*`, `none`) ohitetaan virheilmoituksella, ja sen tallenteisiin tallentuu `cap_id_source=model`, joten oletetun rajoituksen varoitus ei koskaan laukea sen kohdalla. Älä myöskään aseta ylärajaa sen tietoihin manuaalisesti: yläraja `sunshine_cosine` antaisi noin 11-kertaisen lukeman.

**Siinä on 3-akselinen kiihtyvyysanturi.** Auringonvalon hajottajan kosinivaste on voimassa vain, kun se on vaakasuorassa, joten kallistunut anturi ilmoittaa väärän säteilyvoimakkuuden – ja heijastavuus perii virheen – ilman muita oireita. Jokainen DAQ-E-S-lukema sisältää siksi laitteen asennon:

| Kenttä | Merkitys |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Kulmat asteina. `None`, kun laiteohjelmisto merkitsee kulman määrittelemättömäksi (vapaa pudotus, toimimaton akseli). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Kiihtyvyys millig-yksiköinä. `mag_mg`, joka poikkeaa huomattavasti arvosta ~1000, tarkoittaa, että laite oli liikkeessä ja kulmat eivät ole merkityksellisiä kyseisen näytteen osalta. |
| `cal_applied` | Asetettu: akselit ja kulmat ovat diffuusorin koordinaatistossa (laitteen kiinnityskalibrointi on otettu huomioon). Tyhjä: raaka kortin koordinaatisto, joka voi näyttää jopa ~2,3°:n kallistuksen, joka ei ole todellinen. |
| `sample_age_ms` | Kiihtyvyysanturin näytteen ikä, kun spektri lähetettiin. |
| `temp_raw` | Kalibroimaton piisirun lämpötilan **trendi** — vain eroja, ei koskaan lämpömittarin lukemaa. |

`DAQSensorSession.latest()` palauttaa ne kunkin kehyksen `imu`-avaimen alle (`None` DAQ-U-, DAQ-M- tai tavallisessa DAQ-E-laitteessa). DAQ-E-S:stä tallennettu `.daq` tallentaa samat arvot sarakkeisiinsa `imu_*` (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) jokaisen spektrin viereen, jotta lukija voi erottaa pilven kolhiintuneesta kolmijalasta. Kaikissa muissa malleissa nämä sarakkeet ovat tyhjiä.

Asennon datavirta raportoi omista kentistään sarakkeissa `status()` (`rx_imu_stream_fault`, `rx_imu_stream_faults`, `rx_last_imu_stream_fault`), jotka on pidetty erillään spektrikentistä, jotta kiihtyvyysanturin toimintahäiriötä ei koskaan tulkita tallennuksen aukoksi. `discover_daq_sensors()` ja istunnon `model` ilmoittavat `DAQ-E-S`.

### DAQ-E-laitteen asetukset

```python
with chloros_sdk.connect_daq_sensor(eth_host="daq-e-def330.local") as daq:
    info = daq.device_info()
    print(info["ip"], info["ip_mode"], info["firmware"])

    # A fixed address on the network this computer is on
    daq.set_network("static", ip="192.168.1.50", gateway="192.168.1.1")
    print(daq.address)              # 192.168.1.50: the session follows it

    if info["firmware"]["update_available"]:
        daq.update_firmware()
```

- **Tämän tietokoneen verkossa on oltava staattinen osoite.** `set_network` aiheuttaa virheen `ChlorosConnectError`, jos laitteen aliverkko ei sisällä tämän tietokoneenanturin verkossa: uudelleenkäynnistyksen jälkeen mikään ei pääse anturiin muuttamaan asetusta takaisin. Ohita `force=True` vain, kun tälle tietokoneelle määritetään uusi osoite tai se reititetään uuteen aliverkkoon.
- **`set_network`, `reboot` ja `update_firmware` käynnistävät anturin uudelleen** ja laukaisevat `ChlorosConnectError`:n, kun se tallentaa, ja jos se ei palaa takaisin `comeback_timeout` sekunnin kuluessa. Anturin tunnus ja `daq-e-<id>.local`-isäntänimi eivät koskaan muutu.
- **Juuri ennen uudelleenkäynnistystä alkanut tallennus suljetaan uudelleenkäynnistyksen yhteydessä ja sille annetaan nimi.** Kun anturi palaa takaisin, palautetussa sanakirjassa on `recording_closed` (tiedoston polku) ja `warning`, joka ilmoittaa asiasta. Jos anturi ei palaa takaisin tai laiteohjelmiston siirto epäonnistuu, `ChlorosConnectError` ilmoittaa asiasta: sen viesti päättyy varoitukseen, ja `e.recording_closed` / `e.warning` asetetaan (molemmat `None`, jos tallennusta ei ole suljettu). Käynnistä tallennus uudelleen, kun anturi on palannut.
- **Vain anturi itse kytketään uudelleen.** Uudelleenkäynnistyksen jälkeen laitteen on vastattava sieltä, missä se oli -- ja `set_network("auto")`:n jälkeen myös nimellään `.local` -- ilmoittamalla nimensä, isäntänimensä ja mallinsa sekä aiemman verkkosovittimensa (MAC) osoitteen; `address`:stä tulee se IP-osoite, joka vastasi. Nimen alla annettu vastaus lasketaan vain kyseisellä MAC-osoitteella, joten tietokoneelta, joka tavoittaa anturin reitittimen kautta, tai WSL2:n oletusarvoisesta NAT-verkosta, DHCP:n siirtämää anturia ei kytketä uudelleen (`ChlorosConnectError`; muodosta yhteys uudelleen sen uuteen osoitteeseen).
- **Laiteohjelmiston päivitys edellyttää, että anturi muodostaa yhteyden uudelleen.** `update_firmware()` kutsuu laitetta, ja laite avaa TCP-yhteyden takaisin tähän tietokoneeseen kuvan lataamista varten käyttöjärjestelmän valitsemalla portilla. Taustapalvelin kuuntelee sitä sovitinta, joka tavoittaa laitteen (kaikilla sovittimilla vain silloin, kun se ei osaa erottaa, mikä niistä on kyseessä), ja palvelee vain kyseistä laitetta: kaikki muut yhteydet hylätään (`[DAQ-FW] … refused a firmware connection from <ip>` taustapalvelimen tulosteessa), kunnes laite on valmis. Palomuuri, joka estää yhteyden, keskeyttää päivityksen virheilmoituksella `device never connected back on TCP <port>`. Windows-asennusohjelma lisää sille säännön (katso [Vaihe 1](#windows-exe)); toisessa palomuurissa sallitaan saapuva TCP-liikenne backendille anturin aliverkosta.
- **WSL2:n alla käytä peilattua verkkoa** (`networkingMode=mirrored` ja `.wslconfig`): oletusarvoisessa NAT-tilassa jokainen staattinen osoite hylätään (WSL:n oma osoite ei ole anturin verkossa), datavirta ei pääse WSL:ään, eikä laiteohjelmiston päivitys voi valmistua.
- Käyttöliittymä ylläpitää omaa yhteyttään anturiin: skriptin avulla suoritettu uudelleenkäynnistys katkaisee käyttöliittymän istunnon anturin kanssa. Käyttöliittymässä on samat asetukset (Valoanturit-välilehti, anturin asetukset, Verkko).

### Löytö — yhteyden muodostamiseen tarvittavan osoitteen etsiminen

`discover_daq_sensors()` etsii USB-, BLE- ja ETH-liitännöistä antureita, jotka *voisit* avata. Se on DAQ-vastine `discover_lattice_cameras()`:lle ja ainoa tapa saada selville **DAQ-M:n BLE-MAC-osoite** — DAQ-E:llä on isäntänimi ja DAQ-U:lla COM-portti, mutta MAC-osoitetta ei ole painettu laitteeseen eikä se näy käyttöjärjestelmän luettelossa.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               DAQ-U  {'vid': '0403', 'pid': '6001', 'manufacturer': 'FTDI'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Kenttä | Kuvaus |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM-portti / BLE MAC / isäntänimi — välitetään `connect_daq_sensor`:lle nimellä `port=` / `mac=` / `eth_host=`. |
| `display` | Ihmisen luettavissa oleva nimike. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` \| `DAQ-E-S`, tai `None`. Ethernet-anturi on `DAQ-E-S` tai `DAQ-E`, sillä se ilmoittaa itsensä pyydettäessä, vain luku -tilassa, skannauksen aikana (`DAQ-E`, jos se ei vastaa ajoissa). USB-sarjaportti näkyy lukuna `DAQ-U`, kun sen siltapiiri on sellainen, jota DAQ-U käyttää (FTDI, CH340, CP210x, Prolific) — vihje portin kuvauksesta, ei koetin — ja `None` kaikille muille USB-laitteille, jotka ovat edelleen luettelossa, joten voit välittää ne nimellä `port=`. Sarjaporttia, jolla ei ole USB-valmistajatunnusta eikä tällaista silta-piiriä (emolevyn UART, kuten Linuxin `/dev/ttyS*`, Bluetooth-sarjaliitäntä, Intel AMT), ei voida pitää DAQ-U:na, eikä sitä ole luettelossa. |
| `extra` | Kuljetuskohtaiset tiedot (BLE:n ilmoitettu nimi; USB `vid`, `pid`, `manufacturer`, `serial_number`; DAQ-E ip/fw/…). Tyhjät arvot on jätetty pois. |

| Parametri | Oletusarvo | Kuvaus |
| --- | --- | --- |
| `transports` | kaikki kolme | Skannausta rajoittava sekvenssi (tai CSV-merkkijono). Kannattaa määrittää, kun tiedät mitä haluat — BLE on hidas lenkki. |
| `scan_timeout` | 5 | Skannausikkuna kuljetuskohtaisesti sekunteina; taustapalvelu rajoittaa arvon välille 1–20. |
| `timeout` | 60,0 | Koko kutsun HTTP-yläraja (kuten muuallakin SDK:ssa). |
| `auto_start_backend` | `True` | Käynnistää paikallisen taustapalvelimen, jos sellaista ei ole käynnissä. Ei koskaan käynnisty etäisen `backend_url`:n tapauksessa. |

> **Poolissa jo avoimina olevat anturit eivät näy.** Yhdistetty BLE-oheislaite lopettaa mainostamisen, eikä avointa COM-porttia voida tunnistaa, joten hakutoiminto listaa ne, jotka ovat *yhdistettävissä*. Heti laitteen liittämisen jälkeen on odotettavissa tyhjä tulos — käytä komentoa `list_daq_sensors()` niille laitteille, jotka sinulla jo on. Kuljetusprotokollat, joiden skannaus ei voi suorittaa (bleak / zeroconf ei ole asennettu), ohitetaan virheilmoituksen sijaan, joten kone, jossa ei ole Bluetoothia, saa silti vastaukset USB- ja ETH-laitteista.

### Luettelo

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Mikä DAQ syöttää mitä kameraa — `daq_bindings()`

Kuvauksen heijastavuusarvo ja sen tallentama `.daq` käyttävät kyseiseen kameraan tai matriisiin sidottua DAQ:ta, muuten backendin uusinta DAQ:ta. GUI:n valosensorin valinta ja `chloros-cli daq pool-bind` määrittävät sen; samoin kuin:

```python
chloros_sdk.set_daq_bindings(cameras={"214701281": "COM7"},          # a sensor id or an address
                             arrays={"array-1779862544497": "daq-e-def330"})
chloros_sdk.set_daq_bindings(cameras={"214701281": None})            # None clears it
for serial, b in chloros_sdk.daq_bindings()["cameras"].items():
    print(serial, b["binding"], b["source"], b["resolved"]["in_pool"])
```

`daq_bindings()` palauttaa `{"cameras": {serial: b}, "arrays": {array_id: b}}`: jokainen `b` sisältää `binding` (asetettu arvo), `source` (`project`, projektin `sensors.json`:sta tai `runtime`:sta), `known` / `known_as` (riippumatta siitä, nimeääkö kamera- vai taulukkoavain jonkin Chlorosin tunteman kohteen ja miten: `connected`, `project` — avoimen projektin `cameras.json` — tai, kameran osalta, `seen`: avattu aiemmin tällä tietokoneella; `known: False` avaimelle, joka ei nimeä mitään) ja `resolved` (pool-anturi, johon se nyt viittaa: `in_pool`, `sensor_id`, `address`, `model`, `streaming`). Suoritusaikainen sidonta ohittaa projektinistunnon ajaksi; projektitiedostoa ei muuteta. Kameran, taulukon tai DAQ:n irrottaminen poistaa kaikki kyseisen laitteen sidokset riippumatta siitä, kuka ne on asettanut (taulukon jäsenten sidokset taulukon kanssa).

`set_daq_bindings()` sitoo vain kameran, joka on kytkettynä, avoimessa projektissa tai aiemmin avattu tällä tietokoneella, sekä matriisin, joka on kytkettynä tai avoimessa projektissa (tai avaimen, joka on jo sidottu). Mikä tahansa muu avain aiheuttaa virheen `ChlorosConnectError`, jossa mainitaan (`400`, `code`, `UNKNOWN_BINDING_TARGET` sekä `unknown`-näppäimillä), eikä **mitään** kyseisessä kutsussa sidota. `None` tyhjentää kaikki avaimet.

### Ympäristön + kohteen heijastavuus — `light_sensor_reflectance()`

Kaksi yhdistettyä DAQ-laitetta, joista toinen on suunnattu taivaalle (ympäristö) ja toinen kohteeseen (kohde), antavat kohteen heijastavuus- ja kasvillisuusindeksit viimeisimmistä lukemistaan: käyttöliittymän Ympäristö + Kohde -pari sekä `chloros-cli daq reflectance` ilman, että antureita tarvitsee avata toista kertaa. Molempien on oltava kytkettyinä ja lähettämässä dataa.

```python
r = chloros_sdk.light_sensor_reflectance("daq-e-def330", "daq-e-54b774")
print(r["reflectance_type"])                       # "Apparent Reflectance" when both are calibrated
for name, band in r["bands"].items():
    print(name, band["rho"] if band["valid"] else band["reason"])
for idx in r["indices"]:
    print(idx["name"], idx["value"])
```

```python
light_sensor_reflectance(
    ambient_sensor_id, object_sensor_id, *,
    bands=None,          # [{"name", "center_nm", "fwhm_nm", "role"}]; role blue / green / red / nir
    preset=None,         # a preset id from daq_band_presets(); default: the suggested one, else "GENERIC"
    expressions=None,    # custom indices: [{"name": "MyNDVI", "expr": "(NIR_850 - Red_660) / (NIR_850 + Red_660)"}]
    builtins=True,       # include the built-in indices
    backend_url="http://127.0.0.1:5000", timeout=30.0,
) -> dict
```

- **Kaistat.** Kunkin kaistan heijastavuus on kohteen kaistalle-integroitu säteilyvoimakkuus ympäristön säteilyvoimakkuuteen nähden. `daq_band_presets(filter_code=None)` luettelee kaistajoukot: yksi kutakin LATTICE M3C -suodatinta kohti (`RGN`, `OCN`, `NGB`, kameran oma kaistamalli, joten DAQ-NDVI ja kameran NDVI ovat sama suure) sekä `GENERIC`-laajakaista. Sen `suggested` on ennalta määritetty vastaavuus `filter_code`:lle, muuten ensimmäisen kytketyn kameran suodatin (`suggested_from` kertoo, mikä).
- **Palauttaa** `bands` (kaistakohtaisesti `rho`, `valid`, `reason`, `e_ambient`, `e_object`), `indices` (sisäänrakennetutlisäosat), `custom` (kussakin yhdessä `value`:n tai `valid: False`:n ja `error`:n kanssa), `invalid_bands`, `reflectance_type`, `preset` sekä näiden kahden lukeman `timestamps` (`ts_ns`). Alue, jota ympäristön anturi ei pystynyt mittaamaan (mittausalueen ulkopuolella tai kohinatasolla), on virheellinen, ja syy ilmoitetaan; kaikki indeksit, jotka tarvitsevat kyseistä arvoa, jätetään pois, eikä niitä lasketa koskaan nollan suhteen.
- **Näennäinen tai suhteellinen.** Mittakaavasta riippuvat indeksit (SAVI, OSAVI, EVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) vaativat absoluuttisen suhteen, joten ne lasketaan vain, kun molemmat anturit on kalibroitu tehon suhteen (kummallakin on `calibration_bundle_sha`): `"Apparent Reflectance"`. Muussa tapauksessa tulos on `"Relative Reflectance"`, jossa on vain mittakaavariippumattomat indeksit.
- **Kahden eri aallonpituusruudukon** päällekkäisyyttä verrataan: ympäristöanturin pisteet sijaitsevat kohteen mittausalueella, ja kohteen spektri interpoloidaan niihin.
- **Kaksi anturia, ei koskaan yhtä.** Yhden anturin nimeäminen molemmiksi (sama tunnus, millä tahansa kirjoitusasulla, jonka pool hyväksyy) aiheuttaa virheen `ChlorosConnectError`: anturin lukema itsestään on 1,0 jokaisella kaistalla, mikä ei mittaa mitään. Myös taustajärjestelmä hylkää sen (`POST /api/daq/indices` vastaa `400`, `code` ja `SAME_SENSOR`, kun taas SDK:n lähettämät `ambient_sensor_id` ja `object_sensor_id` ovat yksi yhdistetty anturi, esim. sen tunnus ja sen portti).

### Yhteiskäyttö CLI:n ja muiden skriptien kanssa

Backendin DAQ-anturipoolia jakavat kaikki koneella olevat CLI- ja SDK-asiakasohjelmat. Jos anturi on jo poolissa — avattuna komennolla `chloros-cli daq pool-connect` tai jollain toisella skriptillä — komennon `connect_daq_sensor(port="COM3")` kutsuminen Pythonista palauttaa lainatun käsittelytunnuksen: sen `disconnect()` on tyhjä operaatio, joten skriptisi ei vie anturia siltä asiakkaalta, joka avasi sen, kun scope poistuu. Sama pätee `sensor_id=`:ään (kirjoitetaan `:` tai `-`, kumpi tahansa sopii), kun kyseessä on DAQ-E:n osalta, joka on nimetty IP-osoitteensa perusteella, kun se avattiin isäntänimellä (tai päinvastoin), sekä nollalippuisen älykkään tunnistuksen tapauksessa, kun taustapalvelin vastaa anturilla, joka oli jo sen hallussa — `with`-lohko vapauttaa aina vain anturin, jonka se on itse avannut.

Chloros-käyttöliittymä **ei** kuulu tähän pooliin: sen DAQ-paneeli ylläpitää omaa yhteyttään anturiin, joten `connect_daq_sensor` ei koskaan lainaa anturia, jonka käyttöliittymä on avannut. Älä käytä samaa anturia samanaikaisesti sekä käyttöliittymässä että skriptissä. DAQ-U- tai DAQ-M-laite, jota käyttöliittymä pitää hallussaan, ei yleensä avaudu, mutta DAQ-E sallii toisen istunnon: skriptin yhteyden muodostus muuttaa laitteen keräysasetuksia GUI:n alla, ja sen `disconnect()` pysäyttää laitteen datavirran, mikä lopettaa GUI:n reaaliaikaisen näkymän ja sen tekemän tallennuksen. Sulje anturi ensin GUI:ssa.

### PTP-pääkello — `time_sync_status()`

Taustapalvelu ajaa PTP-pääkelloa, josta LATTICE-kamerat ja DAQ-E-yksiköt saavat aikaleimansa. Se käynnistyy pyynnöstä: `connect_camera()`, `connect_array()`, `connect_daq_sensor()` ja `connect_all()` käynnistävät sen, jos se ei ole käynnissä — mukaan lukien yhteyden muodostusyritys, joka epäonnistuu, kuten `connect_all()`:n yritys muodostaa yhteys puuttuvaan DAQ-E-yksikköön — samoin kuin `test_array_sync()` ja jokainen kutsu, joka avaa kameroita yhtä komentoa varten (`capture_cameras_transient()`, `start_multi_camera_stream()`, kohdistuskutsut, `capture_live_index()`).

- **Missä se ilmoittaa.** Niissä langallisissa verkkosovittimissa, jotka ovat yhteydessä kameroihin (mukaan lukien `169.254.x.x`-linkkipaikallinen osoite), muutoin oletusreitin sovittimessa, jos se on langallinen. Ei koskaan Wi-Fi-verkossa tai virtuaalisessa sovittimessa (Tailscale, WSL): niiden takana ei ole kameraa tai DAQ-E-laitetta. Jos tällaista sovitinta ei ole käytössä, se toimii ilmoittaen tyhjää (`health` `NO_SOCKETS`) ja sitoo sovittimen seuraavaan kameraan tai DAQ:hen, kun yhteys muodostuu.
- **Yhteyden katkeamisen jälkeen.** Kun kameran sovitin menettää yhteytensä ja saa sen takaisin (kaapelin irrottaminen, kytkimen vaihto, USB-sovittimen uudelleenluettelointi), grandmaster rakentaa socketit uudelleen itsestään ilman uudelleenkäynnistystä ja tarkistaa uudelleen kaikki kytketyt DAQ-E-laitteet, aktivoiden uudelleen ne, joiden yhteys on katkennut. Se rakentaa yhteydet uudelleen myös silloin, kun kaikki sen palvelemat kamerat ja DAQ-E-laitteet ovat olleet hiljaa 15 sekunnin ajan.
- **Onko yhteys katkennut?** Lue `time_sync_status()` (`GET /api/time-sync/status`). `state` kertoo vain, että grandmaster-prosessi on käynnissä; `health` kertoo, lähtevätkö sen paketit: `OK` (virheitä ei havaittu), `STOPPED`, `NO_SOCKETS`, `SENDS_FAILING`, `DEGRADED` (jotkut sovittimet eivät toimi) tai `PEERS_SILENT` (kamerat ja DAQ-E-laitteet, joita se palveli, lakkasivat vastaamasta), ja lause `health_detail`:ssä. `announce_sent` / `sync_sent` / `delay_resps_sent` laskevat vain lähetetyt viestit; `announce_send_errors`, `sync_send_errors`, `follow_up_send_errors`, `delay_resp_send_errors`, `last_send_error`, `failing_interfaces` ja `last_delay_req_age_s` kuvaavat niitä, jotka eivät lähteneet, ja `rebinds` / `last_rebind` uudelleenrakennuksia. `sync_sends_slow` laskee ne Syncs-viestit, joiden lähetys kesti yli 200 us, ja `sync_send_max_us` on pisin; kunkin Follow_Up-viestin lähetyshetki lasketaan enintään 200 us:n pituiseksi. Grandmaster toimii omassa prosessissaan (`own_process`, `process_pid`), korkealla prioriteetilla Windowsissa (`process_priority`), joten kiireinen taustaprosessi ei voi viivästyttää leimausajankohtia eikä siirtää kaikkien kameroiden kelloja kerralla; jos kyseistä prosessia ei voida käynnistää, se toimii taustaprosessin sisällä kuten aiemmin (`own_process` false).

```python
st = chloros_sdk.time_sync_status()
if st.get("health") not in (None, "OK", "STOPPED"):
    print("PTP:", st["health"], "-", st.get("health_detail"))
    chloros_sdk.restart_time_sync()
```

| Toiminto | Mitä se tekee |
| --- | --- |
| `time_sync_status()` | Grandmasterin tila, kunto, kellotunniste, prioriteetit ja viestilaskurit (yllä). `chloros-cli time-sync status`. |
| `time_sync_peers()` | Sen havaitsemat PTP-orjat (kamerat, DAQ-E-laitteet): `peers`, kukin `port_identity`, `last_seen_ip`, `last_delay_req_age_ms`, `delay_req_count`. `time-sync peers`. |
| `time_sync_cameras()` | Jokaisen kytketyn matriisikameran `PtpStatus`, `PtpOffsetFromMaster` ja `PtpMeanPathDelay` (kamerassa, jonka laiteohjelmistossa ei ole PTP-solmuja, on vain sarjanumero). `time-sync cameras`. |
| `restart_time_sync()` | Pysäytä ja käynnistä grandmaster uudelleen; jokaisen yhdistettyjen DAQ-E-laitteiden PTP tarkistetaan uudelleen tämän jälkeen (`daq_ptp_rechecked`). Lähettää `ChlorosConnectError`-viestin, kun se ei palaa takaisin. `time-sync restart`. |
| `set_time_sync_priority(priority1=None, priority2=None)` | Sen PTP-prioriteetit (0–255, pienempi voittaa best-master-vaalin) verkossa, jossa on toinen PTP-master. Edellyttää, että se on käynnissä. `time-sync set-priority`. |

Kukin ottaa `backend_url=`:n ja `timeout=`:n ja käynnistää paikallisen taustapalvelimen, joka ei ole käynnissä.

---

## Kameran apuohjelmat

Mitä `chloros-cli lattice` tekee yhteyden muodostamisen ja kuvan kaappaamisen lisäksi, kuten tavalliset toiminnot `chloros_sdk`:ssä (moduuli `chloros_sdk.camera_utils`). Kumpikin kutsuu taustapalvelun reitin, jota CLI-komento käyttää, CLI:n sääntöjen mukaisesti, joten ne toimivat samalla tavalla. Ne ottavat vastaan kameran **sarjanumeron**, eivät istuntoa, ja samat `backend_url=` / `timeout=`-avainsanat kuin smart-connect-kutsut; jos taustapalvelu ei ole käynnissä, se käynnistetään, kuten `connect_camera()` tekee. Hylkäys aiheuttaa `ChlorosConnectError`-virheen, jossa ilmoitetaan taustapalvelun hylkäysperuste.

```python
import chloros_sdk as c

c.set_camera_color_profile("214701281", profile="natural")
c.set_camera_highlights("214701281", -40)
print(c.camera_firmware_status()["cameras"])
st = c.network_adapters(estimate=True, cameras=4, pixel_format="BayerRG12")
```

| Toiminto | Mitä se tekee |
| --- | --- |
| `camera_color_profile(serial)` / `set_camera_color_profile(serial, profile=, custom_cct_k=)` | Näytön väriprofiili: `raw`, `linear`, `natural`, `enhanced` tai `custom_temp` sekä sen värilämpötila (rajoitettu välille 2000–10000 K; luetaan, kun profiili on `custom_temp`). |
| `camera_color_adjust(serial)` / `set_camera_color_adjust(serial, saturation=, contrast=, reset=False)` | Näytön kylläisyys ja kontrasti, kumpikin 0–200 % (100 = ei muutosta); `reset=True` palauttaa molemmat arvoiksi 100. |
| `camera_highlights(serial)` / `set_camera_highlights(serial, value)` | Vähentää näytön renderöinnin kirkkaimpia, valkoisimpia pikseleitä: -100..0 (katkaistaan nollan suuntaan, sitten rajoitetaan; 0 = pois päältä). |
| `camera_white_balance_once(serial, settle_s=None)` | Kertaluonteinen valkotasapaino: kamera lähettää lyhyen datavirran, konvergoi ja palauttaa `red_ratio` / `blue_ratio` (`ratios_read` False, jos niitä ei voida ilmoittaa). Se säilyttää ratkaisemansa valkotasapainon; kaikki muut asetukset, jotka sen open-funktio kirjoitti, palautetaan sellaisina kuin ne löydettiin. |
| `set_camera_white_balance(serial, mode=None, red=None, blue=None)` | `mode="auto"` (jatkuva) tai `"off"`; tai kiinteät `red` / `blue` tasapainosuhteet, jotka kytkevät automaattisen toiminnon pois päältä ensimmäisenä. Suhde, jota kamera ei hyväksy, on nimetty kohdassa `errors`. |
| `camera_presets()` / `apply_camera_preset(serial, preset)` | Kameran esiasetukset ja yhden soveltaminen: kamera säilyttää asetukset, joita esiasetusten nimet (vastauksen `settings`, kuten kamera lukee ne myöhemmin, esim. sen käyttämä kuvataajuus; `None` kentälle, jota esiasetus ei nimeä), ja kaikki muut asetukset, jotka sen avoin tiedosto kirjoitti, palautuvat löydetyssä muodossa. `"default"` palauttaa kameran tehdasasetuksiin (tehdasasetukset, joissa automaattinen valotus toimii vapaasti, linkin läpimenorajoitus ja virhekorjaus pois päältä, RGB-kameran kalibroitu valkotasapaino säilyy), joita ei tallenneta kameran muistiin: vastauksessa on `factory_defaults: True` ja kaikki `settings`:n kentät. Kohdassa `camera_presets()`, `default` on `{factory_defaults: True, description}`. |
| `camera_filter_models()` / `camera_filter_model_info(name)` | Suodatinmallit ja yhden mallin tiedot (kaistat, spektrivasteen ehdollinen luku). Kameran suodatinmalli luetaan sen mallimerkkijonosta; sitä ei aseteta manuaalisesti. |
| `camera_firmware_status(serial=None)` | Kunkin kameranlaiteohjelmisto: `current_version`, `available_version`, `update_offered`, `needs_minimum_update` ja `minimum_firmware`. |
| `update_camera_firmware(serial, force=False)` | Asenna tämän Chloros-version laiteohjelmisto yhteen kameraan: tehdasasetusten palautus, asennus, uudelleenkäynnistys, Chloros-perusasetukset kirjoitetaan uudelleen; kameran kalibrointi säilyy. Kestää useita minuutteja; kameran on oltava kytkettynä virtaan. `force=True` päivittää nykyisen kameran laiteohjelmiston uudelleen. |
| `camera_power_probe(serial)` | Kameran virta- ja lämpösensorit sekä sen lämpötila. |
| `sleep_camera(serial)` / `wake_camera(serial)` | Kameran virransäästötila (se pysyy lepotilassa, kunnes herätetään: sen linkin läpimenorajoitus on kytketty päälle (`DeviceLinkThroughputLimitMode` `On`) alarajallaan, virransäästötila päällä, jos kameralla on sellainen), sekä lämpötila ennen ja jälkeen; herätys palauttaa yhteyden tilaan, jossa se oli lepotilassa (raja-arvo, tila ja pakettiviive `GevSCPD`, jonka kamera pitää arvossa 0, kun raja-arvo on päällä; tieto tallennetaan tähän tietokoneeseen, joten myöhempi taustapalvelin löytää sen herätettäessä), tai, jos kyseessä on kamera, jota tämä tietokone ei ole asettanut lepotilaan mutta joka näyttää olevan lepotilassa (raja-arvo on alarajalla), raja-arvo poistetaan käytöstä ja `GevSCPD` jätetään arvoon 0 kohdassa 80 (tehdasasetukset); raja-arvo, joka on asetettu minimirajan yläpuolelle, jätetään ennalleen. Vain virrankäyttötila tallennetaan; kaikki muut asetukset, joihin on tehty muutoksia, palautetaan alkuperäisiksi. |
| `calibrate_camera_with_target(serial, filter=None, attempts=30, settings=None)` | Kaappaa kehyksiä, kunnes MAPIR-kalibrointikohde on näkymässä, ja sovita kanavakohtaiset kertoimet sen paneeleista: `target_id`, `panel_dns`, `coefficients`. |
| `network_adapters(estimate=False, cameras=1, pixel_format="BayerRG8")` | Tämän tietokoneen GigE Vision -Ethernet-sovittimet (MTU, linkkinopeus, jumbo-kehykset) ja mahdolliset `issues`; `estimate=True` lisää kuvataajuuden ja kaistanleveyden, jotka `cameras`-kamerat saisivat. |
| `fix_network_settings()` | Sovella GigE Vision -sovittimen asetukset tämän tietokoneenkamerasovittimeen. Se muuttaa sovittimen asetuksia ja toimii vain, kun Chloros-ohjelma on käynnissä järjestelmänvalvojana; muussa tapauksessa se näyttää lokin löydöksistään. |
| `apply_vignette_correction(input_dir, output_dir, lens_model=None)` | Korjaa kansiossa olevien kuvien vinjetointi, jokaiselle omalla objektiivin tasokenttäkartalla: `processed`, `lens_models` (objektiivi, jolla kukin tiedosto on korjattu), `skipped` (korjausperusteet). Ilman `lens_model`-asetusta kunkin tiedoston objektiivi määritetään sen EXIF-mallitiedon perusteella; `lens_model` koskee tiedostoja, joiden mallitiedossa ei mainita kameraa, ja tiedosto, jonka mallitiedot ovat ristiriidassa tämän kanssa, ohitetaan. Vain tiedostot. |
| `read_dls(mac, filter=None, camera_serial=None)` | Yksi lukema DLS:stä (Bluetooth-alaspäin suuntautuva valosensori): spektri, kaistakohtainen säteilyvoimakkuus ja kaistojen suuruudet kameran oman vasteen kautta, kun `camera_serial` on määritetty. `discover_daq_sensors(transports=["ble"])` etsii MAC-osoitteen. |
| `probe_system(pixel_format="BayerRG12", discover=True)` / `system_info()` | Isäntäkoneen kameralaitteisto ja sen suosittelema kokoonpano (`report_text`) sekä järjestelmän CPU / GPU / PyTorch-ajoympäristö. |

- **Jotkut tarvitsevat kameran yksin käyttöönsä.** `camera_white_balance_once()`, `apply_camera_preset()`, `sleep_camera()` / `wake_camera()`, `calibrate_camera_with_target()` ja `update_camera_firmware()` avaavat kameran. Kun Chloros pitää sitä hallussaan (kytkettynä sovelluksessa, `connect_camera()`:n kanssa, ryhmän jäsenenä tai toisen komennon käytössä juuri tällä hetkellä, kuten taulukotesti tai kameran avaava tallennus), taustapalvelu hylkää pyynnön HTTP-virheellä 409 `CAMERA_HELD` (sekä `code`:ssä että `error_code`:ssä), jossa kerrotaan, miten se vapautetaan. Väriasetukset, korostukset, `camera_power_probe()` ja lukemat toimivat kytketyssä kamerassa.
- **Väri, korostukset ja valkotasapaino koskevat RGB-kameroita.** Yksikaistaisessa mustavalkokamerassa (M3M) asetukset eivät muuta mitään ja palauttavat arvon `{"skipped": True, "reason": ...}`, kuten CLIkin tekee. Ne ovat näyttöasetuksia: reaaliaikainen näkymä ja näyttötason kuvat, eivät koskaan säteilyä tai heijastavuutta.

---

## Transienttitaulukot, kohdistusprofiilit ja indeksit

Mitä `chloros-cli lattice` tekee ilman pysyvää yhteyttä, kuten toiminnot `chloros_sdk`:ssä (moduuli `chloros_sdk.array_utils`): `continuous`, `multi-capture`, `multi-stream`, `multi-test`, `multi-detect`, `analyze-array`, `align-calibrate`, `align-apply`, `align-stream`, `align-info`, `align-reorder` ja `index`. Kukin niistä lähettää CLI-komennon lähettämän pyynnön CLI:n oletusarvoilla.

```python
import chloros_sdk as c

prof = c.calibrate_alignment_profile(["214000535", "214000551", "214000556"],
                                     "D:/rig/alignment_profile.json", method="orb", model="rigid")
print({sn: t["reprojection_error"] for sn, t in prof["transforms"].items()})

c.capture_aligned_multiband("D:/rig/alignment_profile.json", "D:/rig/out")
idx = c.capture_live_index("D:/rig/alignment_profile.json", preset="NDVI",
                           output_dir="D:/rig/ndvi", capture_level="reflectance")

with c.start_multi_camera_stream(["214000535", "214000551"], "D:/rig/stream", fps=5, count=50) as job:
    print(job.wait()["final_stats"])          # waits for the 50 groups, then closes the cameras
```

- **Ne avaavat kamerat itse.** Jokainen komento avaa omat kameransa, suorittaa tehtävänsä ja sulkee ne, joten se tarvitsee kameroita, joita Chlorosilla ei ole hallussaan (kytkettyinä sovelluksessa, `connect_camera()`:n kautta tai taulukon jäsenenä); taustapalvelu hylkää pyynnön HTTP-virheellä 409 `CAMERA_HELD` ja kertoo, miten kamera vapautetaan. Väliaikainen avaaminen kestää jonkin aikaa (noin 40 sekuntia muutamalle kameralle). Toistuvien kuvien ottamisen osalta `connect_array()` kerran on nopein tapa, ja se tallentaa kaikki vientityypit, kun taas väliaikainen tallennus tallentaa debayeroidun kuvan. `calibrate_alignment_profile()` on poikkeus: kun kyseiset kamerat on liitetty ryhmäksi, se ratkaisee ongelman ryhmän omista kehyksistä (`used_live_array`).
- **`serials`** ovat avattavat kamerat, joista ensimmäinen laukaisee laukaisimen; `None` avaa kaikki verkossa olevat kamerat.
- **`settings`** on sanakirja, joka sisältää kameran asetusten ohitukset, eli kentät, jotka CLI:n liput asettavat: `exposure_time` (µs), `gain` (dB), `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `frame_rate`, `gev_packet_size` ja `preset` (nimeltään kameran esiasetus). Annettu valotusarvo tai vahvistusarvo kytkee automaattisen säädön pois päältä, samoin kuin liput.
- **`gpu`**: `True` pakottaa GPU-warpin, `False` CPU:n (`--gpu` / `--no-gpu`); `None` antaa backendin valita.
- **`keep_settings`**/**`clear_settings`** (kaikki kameran avaavat funktiot, oletusarvo `False`): lopuksi jokainen kamera palautetaan sellaiseksi kuin se löydettiin; `keep_settings=True` tallentaa suorituksen asetukset sen sijaan kameran omaan muistiin (vapaa-ajo), `clear_settings=True` palauttaa kyseisen muistin tehdasasetuksiin — katso [Kameran muisti](#camera-memory--keep_settings--clear_settings). Vastaus (tehtävälle, `stop()`:n) sisältää `camera_memory`:n.
- **Polut ovat taustapalvelimen**: suhteelliset polut muutetaan absoluuttisiksi työkansioosi nähden, kun taustapalvelin sijaitsee tällä koneella. Kohdistusprofiilitiedoston toiminnot (`alignment_profile_info()`, `reorder_alignment_profile()`) lukevat tiedoston tältä tietokoneelta, kuten CLIkin.

| Toiminto | Mitä se tekee |
| --- | --- |
| `start_camera_continuous(serial, output_dir="output", *, format="tiff", jpeg_quality=95, queue_depth=64, settings=None)` | Tallentaa yhden kameran kuvaa jatkuvasti, kunnes se pysäytetään. Palauttaa `CliJob`-arvon. `tiff`-ajon kehykset muodostavat `raw`-tiedoston, johon taulukkomuotoinen tallennus kirjoittaa kyseisen kameran tiedot (yksi mosaiikkitaso anturinomien arvojen mukaan, pakkaamattomina); `png` ja `jpg` sisältävät 8 bittiä, joten kyseiset sarjat ovat demosaikoituja kehyksiä katselua varten. Sidottu tai yhdistetty DAQ kirjoittaa yhden `.daq`-tiedoston kutakin kehystä kohti sarjan`daq/`-kansioon (`daq_files` tilastoissa). Jokainen kehys tallennetaan kerran täydellisine metatietoineen (malli, tallennusajankohta, objektiivi, kaistat, raakakehyksen kalibrointi); `stop()` palauttaa tuloksen, kun jokainen jo kaapattu kehys on kirjoitettu (enintään 25 s), ja vasta sitten kamera suljetaan. Kamera, joka on jätetty valmiustilaan laitteistolähtöä varten (`TriggerMode` `On`) asetetaan vapaakäynnille ajon ajaksi (aloitusvastauksen `trigger` kertoo, mitä löydettiin). Pysäytyksen yhteydessä kamera palautetaan tilaan, jossa se löydettiin — sen laukaisuasetukset ja kaikki ajon aikana muutetut asetukset — ellei `keep_settings=True`:ää käytetä, joka tallentaa ajon asetukset kameran omaan muistiin (vapaakäynti), tai `clear_settings=True`, joka nollaa kyseisen muistin. Ajo, joka ei tallenna yhtään kuvaa, päättyy itsestään 10 sekunnin kuluttua komentojen `stop_reason` ja `no_frames` avulla, ja syy ilmoitetaan komennossa `last_error`, ja sen `stop()` palauttaa kyseisen syyn `error`:nä. Kehykset, jotka hylättiin, koska kirjoittimet eivät pysyneet vauhdissa mukana, lasketaan, ja `drop_reason` kertoo syyn. Jos Chlorosin hallussa olevan kameran osalta käytä `CameraSession.capture_repeated()`. `lattice continuous`. |
| `start_multi_camera_stream(serials=None, output_dir="output", *, fps=3.0, count=None, format="tiff", jpeg_quality=95, settings=None)` | Synkronoi kehysryhmiä `fps`, `count`-ryhmiä tai kunnes pysäytetään. Palauttaa `CliJob` (sen `start_reply` sisältää `n_cams` ja siirron `pacing`); sen lopullisissa tilastoissa lasketaan haetut, tallennetut ja hylätyt ryhmät, tiedostot, joita kirjoittaja ei pystynyt tallentamaan, sekä ryhmät, joista kamera ei toimittanut mitään. `lattice multi-stream`. |
| `capture_cameras_transient(serials=None, output_dir="output", *, format="tiff", jpeg_quality=95, settings=None)` | Yksi synkronoitu tallennus, yksi debayeroitu kuva kameraa kohti: `results` (`filepath`, `serial`, `frame_id`) ja `pacing`. Ryhmä, josta puuttuu kamera, nousee esiin, ja siinä ilmoitetaan ensin, kuinka monta tiedostoa tallennettiin. `lattice multi-capture` ilman liitettyä matriisia. |
| `test_array_sync(serials=None, count=10, ptp_settle_s=None, timeout=None)` | `count` synkronoidut kuvaukset kiinteällä valotuksella: `sync_groups`, `master`, yksi `group_timestamps`-rivi kutakin laukaisua kohti (kamera, joka jätti yhden laukaisun väliin, puuttuu riviltään) ja `sync`: jokaisen toisen kameran siirtymä pääkamerasta, laukaisua kohti (`offsets_us`: kameraa kohti `mean_us`, `min_us`, `max_us`, `jitter_us`, `groups`), YHDELLÄ kellolla (`clock_domain`: `ptp` tai `device` — PTP pois päältä, kunkin kameran oma käyntiaikakello, joten sen poikkeama on käynnistyshetkestä lähtien ja vain jitter mittaa synkronointia). Laukaisija, jonka aikaleimat sekoittavat laitteen kellon ja PTP:n (PTP pois päältä tai vielä lukittumassa), jätetään pois ja lasketaan mukaan (`groups_mixed_domain`), samoin kuin sellainen, jossa PTP-kello on edelleen yli `agree_within_us` pääkellosta (`groups_clocks_disagree`); `notes` osoittavat tämän. `clock_offset_mean_us` / `clock_offset_jitter_us` ovat suurimmat kamerakohtaiset keskiarvot ja vaihtelut. `success` on epätosi, samoin kuin `error`, kun laukaisua on jäänyt väliin, kun alle kahdella laukaisulla on jokaisella kameralla sama kellotaajuus tai kun kellotaajuudet eivät ole koskaan yhteneväisiä (`sync.problems`); rivit palautetaan joka tapauksessa. Se käynnistää PTP-grandmasterin, jos se ei ole käynnissä (`time_sync` ilmoittaa `grandmaster_running` ja `grandmaster_started_by_this_test`), kytkee jokaisen kameran PTP:n päälle (kuten `connect_array()` tekee) ja odottaa ennen ensimmäistä laukaisua, että jokaisen kameran PTP-kello lukittuu grandmasteriin ja vakiintuu: `PtpStatus` Slave on yhdellä grandmasterilla, alle 100 us:n päässä siitä ja vakaana 3 s ajan, ja jokainen kamera lukittuu tällä tavalla ilman taukoa, 120 sekunnin ajan — noin 2 minuuttia virran kytkemisestä. Vasta lukittunut kello on jopa ~200 us:n päässä oikeasta ajasta noin 2 minuutin ajan sen jälkeen, kun se lukee Slave-tilaa, eikä kameran oma `PtpOffsetFromMaster` näytä sitä; 120 sekunnin kuluttua kamerat ovat ~10 us:n tarkkuudella vakiintuneesta kellosta. Kamera, joka on edelleen lukittuneena aiemmasta testistä, jossa PTP jätettiin päälle, kuten tämä taustapalvelin sen näki, laskee tuosta lukituksesta lähtien, joten peräkkäiset testit eivät odota uudelleen; kamera, jonka testi käynnistää, tai kamera, jonka PTP on päällä mutta jota tämä taustapalvelin ei ole nähnyt lukittuneen (käynnistetty muualla tai ennen taustapalvelimen uudelleenkäynnistystä), odottaa 120 sekuntia, ja molempien juuri lukittuneiden kameroiden joukossa ne odottavat edelleen omaa aikaansa. Kamera, joka poistuu orja-tilasta, ylittää 100 us:n rajan tai seuraa toista grandmasteria kolmen lukeman ajan peräkkäin (1,5 s) odotuksen aikana, aloittaa laskennan uudelleen; yksi tai kaksi tällaista lukemaa kirjataan lokiin (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) ja laskenta jatkuu, ja grandmaster-tunnus, jota ei voida lukea, tulkitaan tuntemattomaksi, ei muutokseksi. Jos kaikkien kameroiden laskenta alkaa uudelleen yhtä aikaa (sekunnin sisällä), se johtuu grandmasterin ajan muutoksesta, ei kameroiden ajasta: backendinlokissa lukee `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` luettelee sen `grandmaster_steps`:ssa, ja sen jälkeen umpeutuva odotus aloittaa virheilmoituksensa `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`:lla. Sen osalta rajaa ei laajenneta. `ptp_settle_s` (0–600; `0`, jos vakiintumisodotusta ei ole) asettaa ajon vakiintumisajan; muuta arvoa ei hyväksytä (400-virhe HTTP:ssä). Odotus on rajattu vakiintumisaikaan lisättynä 90 sekunnilla (oletusarvoisesti 210 sekuntia, vähintään 90 sekuntia), ja kutsu odottaa taustapalvelinta `timeout` sekuntia: oletusarvoisesti 240 sekuntia plus vakiintumisaika (360 sekuntia), kuten `chloros-mcp`:n `test_array_sync` tekee. `ptp_lock`:llä on `converged`, `waited_s`, `timeout_s`, `settle_s`, `locked_s` (kuinka kauan jokainen kamera on ollut lukittuna ja vakaana yhdessä), `found_settled` (kamerat, jotka olivat jo lukittuneina vakiintumisaikaan löydettäessä) sekä kamerakohtaisesti `status`, `offsets_ns`, `slave_s` (kuinka kauan se on ollut lukittuna; Ei mitään: ei lukittuna) ja `remaining_s` (Ei mitään, kun minkään kameran PTP ei käynnisty). Taustapalvelimen lokissa on 5 sekunnin välein rivi, jossa on kunkin kameran tila, kuinka kauan se on ollut lukittuna ja kuinka kauan lukitusaikaa on jäljellä. Jokainen kamera käynnistyy PTP:n ollessa pois päältä; tästä lähtien kellot lukittuvat noin 10 sekunnissa. Tämän jälkeen jokainen kamera palautetaan tilaan, jossa testi sen löysi — sen PTP, automaattinen valotus ja vahvistus, jotka testi pitää pois päältä testin ajaksi (`ExposureAuto` / `GainAuto` sekä manuaaliset `ExposureTime` / `Gain`), ja loput, ellei `keep_settings=True` — eli kamera, jonka PTP on jätetty päälle (`PtpEnable=True`, tai ajo, jossa `keep_settings=True`) pysyy lukittuna, ja kun tämä taustaprosessi on havainnut sen lukittuneen vakiintumisaikana, myöhempi testi ei odota sitä. Jos kellot eivät vakiinnu, `success` on epätosi ennen mitään laukaisua, ja `error` kertoo, mitä tehdä: kamera, joka ei ole koskaan lukittunut (`not_slave`), tarkoittaa, että grandmaster ei saavuta sitä (`time_sync_status()`); kamera, joka on jatkuvasti pudonnut pois, nimetään sen mukaan, kuinka kauan se on ollut lukittuneena (`remaining_s`); kellot, jotka yhä lähentyvät toisiaan (`far`): tarkista `time_sync_status()`, ja jotta ne lukittuisivat ennen testiä, kytke PTP päälle ja jätä se päälle (`PtpEnable=True` jokaisessa kamerassa tai `keep_settings=True`; testi palauttaa havaitut asetukset, joten se pysyy päällä), ja suorita sitten testi. `lattice multi-test`. |
| `detect_array_master(line=None)` | Synkronointilinjan kamerat ja oletusarvoinen päälaite: `master_serial`, `slave_serials`, `line`, jotka on testattu. `lattice multi-detect`. |
| `analyze_array_models(camera_models=None, *, width=2048, height=1536, n_active=None, force_tier=None, binning=1, packed="auto", pixel_format=None)` | Mikä valikoima näitä kameramalleja (tuotenimet, `"LATT-M3C-L87-FOCN"`) päätyisi tämän tietokoneen verkkoon: `tier`, `achievable_fps_max`, kaapelibudjetti (`max_cams_collision_safe`, `oversubscribed`), `cam_caps` ja `fix_suggestions`. Oletusarvoisesti löydettyjen kameroiden mallit; yhtään kameraa ei ole avattuna. Jos sinulla on kameroita, joiden asetukset sopivat yhteyden muodostamiseen, katso [`analyze_array_network()`](#network-analysis-helper). `lattice analyze-array`. |
| `calibrate_alignment_profile(serials=None, save_path="alignment_profile.json", *, frames=3, method="orb", model="rigid", reference_serial=None, checkerboard=None, control_points=None, preview=False, vignette=False, roi=None, mask_path=None, rms_threshold_px=None, per_cam_overrides=None, settings=None, ...)` | Laske kohdistusprofiili reaaliaikaisista kuvista ja tallenna se. `method`: `orb`, `akaze`, `phase`, `checkerboard` (yhdessä `checkerboard="9x6"`:n kanssa, sisäkulmat) tai `manual` (yhdessä `control_points={serial: [[x, y], ...]}`:n kanssa, myös vertailukameran kuvat). `model`: `translation`, `rigid`, `affine` tai `homography`. Tarkemmat asetukset ovat CLI:n lippuja: `max_features`, `ratio_threshold`, `matcher` (`bf` / `flann`), `knn_k`, `prefilter` (`none` / `gradient` / `clahe` / `blur` / `hist_match`), `ransac_threshold_px`, `ransac_max_iters`, `ransac_confidence`, `averaging`, `lock_rotation`, `lock_scale`, `lock_axis`; `per_cam_overrides` on `{serial: {option: value}}`. `rms_threshold_px` kieltäytyy tallentamasta huonompaa ratkaisua. Palauttaa `save_path`, `output_width` / `output_height`, `crop_roi`, `reference_serial`, `transforms` (kameraa kohti `reprojection_error`, `num_matches`) ja `preview_path`. `lattice align-calibrate`. |
| `capture_aligned_multiband(profile_path, output_dir="output", *, serials=None, format="tiff", bit_depth=None, gpu=None, no_crop=False, bands=None, order=None, vignette=False, per_band=False, per_camera=False, interpolation="linear", border_mode="constant", border_value=0, settings=None)` | Yksi kohdistettu monikaistakuva profiilin kautta: `files` (monikaistainen TIFF-tiedosto, jonka koko on `bands`, lisäksi yksi kuva kutakin kaistaa kohti, jonka tunnus on `per_band`, ja yksi kuva kutakin kameraa kohti, jonka tunnus on `per_camera`) sekä `vignette_corrected`. `bands` valitsee ja `order` järjestää kaistat nimen mukaan (luettelo tai pilkulla erotettu merkkijono). `lattice align-apply`. |
| `start_aligned_stream(profile_path, output_dir="output", *, serials=None, fps=3.0, count=None, ...)` | Virtauksen mukaan kohdistetut monikaistakuvat (samat vääristysasetukset kuin `capture_aligned_multiband()`), `count`:lle tai kunnes pysäytetään. Palauttaa `CliJob`; sen lopulliset tilastotiedot `frames_written` ja mahdolliset `frame_groups_incomplete`. `lattice align-stream`. |
| `alignment_profile_info(path)` | Profiilitiedosto, joka on tiivistetty muotoon `align-info`, tulostaa sen: `name`, `created`, `method`, `reference_serial`, `sensor_resolution`, `output_width` / `output_height`, `crop_roi`, `camera_serials`, `transforms` (kameraa kohti `model`, `reprojection_error`, `num_matches`, `reference`), `camera_models`, `layer_order` (kameraa kohti `band_names`, `wavelengths_nm`, `enabled`); `profile` on koko tiedosto. |
| `reorder_alignment_profile(path, order=None, enable=None, disable=None)` | Muuta profiilitiedostoa paikan päällä: `order` on kaistojen nimiä (kukin siirtää kameransa kerroksen ylöspäin kyseisessä järjestyksessä), `enable` / `disable` ovat sarjanumeroita. Palauttaa uuden `layer_order`. `lattice align-reorder`. |
| `index_catalog()` | Indeksi-esiasetukset (`presets`: kukin `name`, `channels`, `formula`) sekä väri `gradients`. `lattice index --list-presets` / `--list-gradients`. |
| `compute_index(input_path, *, preset=None, formula=None, channel_map=None, output=None, capture_level="unknown", output_format="all", colorize=False, gradient="RdYlGn", vmin=None, vmax=None, percentile=None, bg_mode="clip")` | Indeksi kohdistetusta monikaistaisesta TIFF-tiedostosta: `preset` (`NDVI`, `NDRE`, ...) tai `formula`, joka kartoittaa symbolit `channel_map` kaistoihin (`{"NIR": "NIR_850", "Red": 0}`, nimen tai indeksin perusteella). Esiasetettusymboli `channel_map` jättää pois sen tiedoston yhden kaistan, jonka nimi alkaa sillä (`red` -&gt; `Red_660`, `nir` → `NIR_850_RGN`), jonka nimi määritetään tiedoston itsensä perusteella (sen kaistaluettelon tai Chloros-kuvauksen/tuotteen `Chloros:Band{R,G,B}Name`), joten NDVI:lle Chloros-heijastuskuvassa ei tarvita `channel_map`:ää; kahta yhtä symbolia vastaavaa kaistaa ei hyväksytä, niitä ei arvata. `capture_level` kertoo, mitä kaistat sisältävät: `reflectance` tai `radiance` tekevät arvoista kvantitatiivisia; kalibroimaton pino (`align-apply`:n) antaa laadullisia arvoja; `"unknown"` lukee tiedot tiedostosta, mukaan lukien Chloros-tiedoston `Chloros:ProcessingLevel`. Palauttaa `stats` (muoto, 1–99-prosenttialue, LUT-alue, käytetty `capture_level`) sekä syötteen viereen kirjoitetun `files`:n (`<input stem>_<preset>.tif`, `..._LUT.png`) viereen tai kohdassa `output`, polku: kuvatiedostot nimetään tiedostotunnuksen perusteella (`output="D:/out/ndvi_FRGN.png"` kirjoittaa `ndvi_FRGN.tif` ja `ndvi_FRGN_LUT.png`), ilman sitä se on etuliite (`D:/out/ndvi` → `ndvi.tif`, `ndvi_LUT.png`); lue polut tiedostosta `files`. Ne sisältävät syötteen kameratiedot (merkki / malli / sarjanumero, GPS, objektiivi, kuvausajankohta UTC-aikaa, kalibrointitunnus, mutta eivät koskaan sen raw-DN-kalibrointia), `Chloros:ProcessingLevel=index` sekä indeksi ainoana kaistanaan (`XMP-Camera:BandName`, TIFF- ja LUT-PNG-tiedostot samoin). Vain tiedostot; ei kameraa. `lattice index --input`. |
| `capture_live_index(profile_path, *, preset=None, formula=None, channel_map=None, serials=None, output_dir="output", output_prefix="index", capture_level="reflectance", save_multiband=False, ...)` | Ota kuva, kalibroi (`reflectance` tai `radiance`; `raw` ohittaa tämän vaiheen), kohdista ja laske indeksi yhdellä kertaa; `save_multiband=True` säilyttää kalibroidun pino, jonka `compute_index()` voi ottaa myöhemmin. Palauttaa `stats`:n (yhdessä sen käyttämien kaistojen sisältävän `serials`:n kanssa) ja `files`:n. Se avaa `serials`:n (ensimmäinen laukaisee liipaisimen), muuten profiilin kamerat, joiden `layer_order`-merkintä on käytössä, ensin vertailukamera (avautuu, vaikka sen kaistat olisivatkin pois käytöstä); kameraa, jonka kaistat ovat pois käytöstä, ei koskaan kalibroida. `channel_map` tarvitaan vain symbolille, johon pino ei voi vastata, kuten kohdassa `compute_index()`: esiasetuksen symboli ottaa sen yhden käytössä olevan kaistan, jonka nimi alkaa kyseisellä symbolilla, ja tämä tarkistetaan ennen kuin mikään kamera avautuu (jos vastaavia ei ole tai niitä on useita, ne hylätään ja ne nimetään); jokainen symboli tarvitsee mukautetun `formula`-merkinnän. `index.tif` ja LUT sisältävät pinon kameran, kuvausajan (vertailukameran valotusaika, UTC) ja objektiivin, ja molemmat nimeävät indeksin ainoaksi kaistakseen (`XMP-Camera:BandName`, esim. `NDVI`: värillinen PNG-tiedosto nimeää indeksin, ei sen värikanavia). `lattice index --live`. |

### `CliJob`

Kolmen datavirran palauttama käsittelytunnus. Tehtävä suoritetaan taustapalvelimella, joka hallitsee kameroitaan, kunnes tehtävä pysäytetään.

| Jäsen | Kuvaus |
| --- | --- |
| `stats(timeout=10.0)` | `running`, `state`, `stats` (tallennetut / kirjoitettu, hylätty, `fps`, `elapsed`, kirjoitusvirheet, `stop_reason`; jatkuvassa työssä myös `frames_grabbed` ja `frames_queued` – jokainen kaapattu kehys kirjoitetaan, asetetaan jonoon, hylätään tai epäonnistuu – sekä `drop_reason`, `writers` ja `write_ms`, yhden kehyksen kirjoittamiseen kulunut keskimääräinen aika) sekä `output_dir`. `state` on `running`, `ended` (se on pysähtynyt itsestään eikä sitä ole vielä pysäytetty), `stopping` (pysäytys on kirjoittamassa jononsa: reaaliaikaiset tilastot sekä `stopped_by`) tai `finished` (sen lopulliset tilastot, pysäytyksen `error` ja `stopped_by`). Pysäytetyn työn tiedot säilytetään 10 minuuttia (enintään viimeiset 32), jokaiselle asiakkaalle: toinen skripti tai `chloros-mcp` lukee saman tilan ja lopulliset tilastot. Työ, jonka `chloros-cli` aloitti (`lattice continuous`, `multi-stream`, `align-stream`) pysäytetään taustapalvelimella, kun CLI:n yhteydenpito lakkaa 30 sekunniksi — lopetettu, kone lepotilassa — taustapalvelimella olevien `stop_reason`, `client_gone` ja `stopped_by`:n avulla; tilastojen lukeminen täältä ei pidä sitä käynnissä. SDK:sta käynnistetty työ jatkuu, kunnes se pysäytetään. Jokainen käynnistys lähettää `timeout`:n taustapalvelimen määräaikana (`client_deadline_s`): sen jälkeen avautuva kamera (joka on vielä käynnistymässä) palautetaan ja suljetaan, eikä mitään tehtävää käynnistetä, ja kutsu palauttaa `ChlorosConnectError` (`CLIENT_DEADLINE_PASSED`). Tehtävän hallussa oleva kamera hylätään kaikilta muilta avauksilta (`connect_camera()`, toinen käynnistys, tilapäinen tallennus) HTTP-virheellä 409 (`CAMERA_HELD`), jossa mainitaan tehtävä ja sen aloittaja. |
| `stop(timeout=30.0)` | Pysäytä se ja sulje sen kamerat: `final_stats`, `output_dir` ja `error`, jos pysäytys ei sujunut ongelmitta. Toinen kutsu palauttaa saman vastauksen, samoin kuin pysäytys toiselta asiakkaalta: kun ensimmäinen on vielä kirjoittamassa jonostaan, toinen odottaa sitä (enintään 25 s; sen jälkeen se nostaa virheen, ja lopulliset tilastot seuraavat kohdassa `stats()`). |
| `wait(timeout=None, poll_s=0.5)` | Odota, kunnes se päättyy itsestään (kun sen `count` on saavutettu, levy on täynnä tai virhe), ja suorita sitten `stop()` lopullisten lukujen keräämiseksi. `timeout` aiheuttaa `TimeoutError`:n, kun se on vielä käynnissä; työ, jolla ei ole `count`:ää, päättyy vasta, kun se pysäytetään. |
| `job_id`, `kind`, `output_dir`, `start_reply`, `stopped` | Sen tunniste ja tyyppi (`continuous`, `multi-stream`, `align-stream`), kirjoituskohde, aloitusreitin vastaus sekä tieto siitä, onko `stop()` suoritettu. |

Käytä sitä kontekstinhallintana, jotta se pysähtyy, kun laajuus päättyy.

---

## Projektikansio ja reseptit

**Projektikansio** on kansio, johon uudet projektit luodaan, ja jota Chloros-sovellus, CLI ja SDK (moduuli `chloros_sdk.system_utils`; `chloros-cli project-folder`) käyttävät yhteisesti:

```python
print(chloros_sdk.project_folder())                # the configured folder, else the default
chloros_sdk.set_project_folder("D:/Flights")       # created if missing
chloros_sdk.reset_project_folder()                 # -> {"path": <default>, "was_set": True}
```

Jos määritetty kansio ei enää ole olemassa, käytetään oletuskansiota (`Documents/MAPIR/Chloros Projects`), kuten sovelluksessakin. `set_project_folder()`:lle annettu suhteellinen polku otetaan työkansiosta, kun taustapalvelu sijaitsee tällä koneella.

**Resepti** on YAML- tai JSON-muotoinen toimintoluettelo, jonka `chloros-cli project run PATH RECIPE` suorittaa tallennetulle projektille. `run_project_recipe()` suorittaa sen CLI:nomalla suorittimellaan (asennettu SDK:n mukana), joten yksi resepti toimii samalla tavalla kummassakin:

```python
r = chloros_sdk.run_project_recipe("D:/campaign/morning.yaml", "D:/Projects/Field_A")
print(r["ok"], r["actions"])
print(r["log"])                                    # what the CLI would have printed

chloros_sdk.run_project_recipe({"actions": [{"apply": {"name": "FrontLeft", "settings": {"exposure_us": 5000}}},
                                            {"wait": "2s"},
                                            {"capture": {"name": "FrontLeft", "output": "pose_a/", "format": "tiff"}}]},
                               "D:/Projects/Field_A")
```

```python
run_project_recipe(recipe, project_path, *, dry_run=False, echo=False,
                   backend_url="http://127.0.0.1:5000") -> dict
```

- **`recipe`** on `.yaml` / `.yml` / `.json`-tiedosto tai itse resepti: sanakirja, jossa on `actions` (`apply`, `capture`, `burst`, `stream`, `sensor`, `wait`) sekä valinnaisesti `overrides` ja `stop_on_error` (oletusarvo true: suoritus pysähtyy ensimmäiseen epäonnistuneeseen toimintoon). Toiminnot ovat CLI-viitteen [Recipe DSL](cli-reference.md#recipe-dsl); suhteellinen `output` määritetään skriptisi työkansioon nähden.
- **Palauttaa** `ok` (kaikki toimet onnistuivat), `actions` (kuinka monta), `log` (suorittajan tuloste; myös `echo=True` tulostaa sen suorituksen aikana) ja `validation`. Puuttuva tai jäsentämätön reseptitiedosto aiheuttaa virheen `ChlorosConnectError`.
- **Tarkistetaan ennen suorittamista.**`validation` on `{"ok", "errors", "warnings", "steps"}`: jokaisen toiminnon avaimena on oltava täsmälleen yksi tunnettu verbi (`{"capture": {"name": ...}}`, ei `{"action": "capture"}` tai `{"type": "capture"}`), joka nimeää projektissa olevan oikean tyyppisen kameran, taulukon tai anturin. Virheellinen resepti ei suorita**mitään** toimintoa (`ok` false, virheet kohdassa `log`), riippumatta siitä, mitä `stop_on_error` sanoo. Käyttämättömät avaimet, sekä kaikki reseptin ohjaamat matriisit tai valosensorit, jotka eivät ole tällä hetkellä kytkettyinä, ovat `warnings`. Resepti ei kytke laitteita: kytke ne projektin ensimmäisenä (`ChlorosProject.connect_all()`, `chloros-cli project connect`, tai MCP-työkalu `connect_project_devices`). `dry_run=True` ei suorita mitään ja palauttaa `{"recipe": ..., "validation": ...}`.
- Prosessissa suoritetaan yksi resepti kerrallaan. YAML-resepti vaatii PyYAML:n.

---

## Projektin automatisointi — `ChlorosProject`

Tallennettu Chloros-projekti on kansio, joka sisältää `cameras.json` + `sensors.json` + `project.json`. `open_project(path, relink=None)` lataa manifestin (`relink`: kansio, jossa siirretyn projektin puuttuvat lähdetiedostot ovat nyt, ja jota käytetään, kun taustapalvelu lataa sen — `process()` — katso **Siirretty projekti** kohdassa `ChlorosLocal`), ja `connect_all` tuo kaikki tallennetut kamerat, matriisit ja valosensorit verkkoon niiden tallennetuilla asetuksilla — sama laitteistotila, jonka käyttöliittymä tuottaisi. Valosensorit muodostavat yhteyden taustapalvelimen [DAQ-anturipoolin](#daq-sensor-sessions) kautta, joten ne tarvitsevat käynnissä olevan taustapalvelimen (katso [`SensorHandle`](#sensorhandle)).

### Yksinkertainen esimerkki

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Light sensors are addressable by name OR slot_id
reading = proj.sensors["Sky"].read()   # (spectrum, is_saturated, ...) or None

# One frame from every camera and array, one spectrum from every sensor
proj.capture_all("./out")

proj.disconnect_all()
```

Tai kontekstinhallintana:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### `ChlorosProject`-metodit

| Metodi | Kuvaus |
| --- | --- |
| `connect_all(*, cameras=True, arrays=True, sensors=True, verbose=False, align=None, packed=None)` | Etsi ja yhdistä kaikki tallennetut kamerat, matriisit ja valosensorit. Palauttaa luokkakohtaisen yhdistämisraportin (`{"cameras": {...}, "arrays": {...}, "sensors": {...}}`, joista jokaisella on `connected`, `missing` ja `errors`; `arrays` sisältää myös `alignment`). Kun taustapalvelin kuuntelee porttia `127.0.0.1:5000`, tallennetut kamerat ja matriisit löydetään ja liitetään sen kautta. Jos tällaista palvelinta ei ole, tämä prosessi etsii ja ohjaa niitä itse portin `lattice_sdk`:n kautta, mikä edellyttää Arena SDK:n ajoympäristöä ja OpenCV:tä (`cv2`) — SDK-wheel ei asenna kumpaakaan; kun se ei onnistu, jokainen tallennettu kamera ja matriisi luetellaan kohdassa `errors` syyn kera, ja `missing` pysyy tyhjänä. Taustapalvelu, joka on käynnissä mutta hylkää SDK:n (kirjautunut ulos: HTTP 401; Copperin alapuolella: 403), ei tarkoita ”ei taustapalvelua”: jokainen tallennettu kamera, matriisi ja valosensori luetellaan kohdassa `errors` taustapalvelun syyn kera, eikä tässä prosessissa ole mitään avoinna. Jokainen kutsu tarkistaa taustapalvelimen uudelleen, joten `connect_all()` kirjautumisen jälkeen tai Chlorosin käynnistämisen jälkeen muodostaa yhteyden samaan projektiolion. Tallennettu matriisi, joka on jo käynnissä taustapalvelimessa samoilla kameroilla (käyttöliittymän tai `chloros-cli project connect`:n), liitetään sellaisenaan ilman, että sen tallennettuja asetuksia sovelletaan uudelleen, ja `disconnect_all()` jättää sen käynnissä. Valosensorit kulkevat aina taustapalvelimen kautta (katso [`SensorHandle`](#sensorhandle)). Se ei koskaan luo taustapalvelinta. |
| `disconnect_all()` | Purkaa kaiken. Jokainen laite vapautetaan ensin; sen jälkeen, jos taustaprosessi ei ole vahvistanut kameran, matriisin tai valosensorin vapautusta (purkamisen aikakatkaisu umpeutui tai se hylättiin), kukin nimetään `ChlorosConnectError`:llä. Laite, joka on jo poistettu, tai jonka taustapalvelinta ei ole käynnissä, ei lähetä mitään. Matriisi tai valosensori, jonka toinen asiakas oli avannut, jää käyntiin. Projektin `with`-lopetus ei koskaan laukea. |
| `capture_all(output_dir=".")` | Yksi kehys jokaisesta kytketystä kamerasta + matriisista, ja yksi spektri (`SensorHandle.read()`) jokaiselta kytketyltä valosensorilta, otettuna vuorotellen (kamerat, sitten matriisit, sitten sensorit), ei samanaikaisesti. |
| `stream(camera, overlays=False, fps=10.0)` | Generaattori, joka tuottaa BGR-muotoisia `numpy`-kehyksiä nimetystä kamerasta (tai matriisista). `overlays=False` lukee kehyksiä, joiden kahva on`grab()`:llä `debayered`-tasolla: taustapalvelimen kautta, kun projekti on kytketty sen kautta, muuten suora `lattice_sdk`-sieppaussilmukka (taulukot tuottavat `{serial: frame}`-sanakirjoja). `overlays=True` reitittää `ChlorosLocal.camera_stream()` → taustapalvelimen `/api/camera/<serial>/stream-annotated` MJPEG-syötteen kautta, jolloin kameran tallennettu `ui.overlay`-lohko välitetään kyselyparametreina. Vaatii taustapalvelintilaa, OpenCV:tä (`cv2`) ja **itsenäisen kameran**: suoramoodissa toimiva kamera aiheuttaa virheen `RuntimeError` (taustaprosessi ei voi kaapata tämän prosessin omistamaa kameraa) ja taulukko aiheuttaa virheen `NotImplementedError` (yhdistää päällekkäiset kuvat kamerakohtaisesti — lähettää jäsenen nimellä). Kertakäyttöinen vastine: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Suorita kohdistus jokaiselle tällä hetkellä kytketylle taulukolle. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Suorita kalibrointi- ja indeksointiprosessi projektin kuville (käärii `ChlorosLocal.process`; nämä neljä ovat **ainoat** hyväksytyt avainargumentit — `indices=` jne. aiheuttaa virheen `TypeError`; aseta indeksit komennolla `ChlorosLocal.configure()`). Muodostaa viiveellä `ChlorosLocal()`-objektin, joka käynnistää taustaprosessin automaattisesti. |

Ominaisuudet:
- `proj.cameras` — `Dict[str, CameraHandle]`, jonka avain on nimi JA sarjanumero.
- `proj.arrays` — `Dict[str, ArrayHandle]`, jonka avaimina ovat nimi JA array_id.
- `proj.sensors` — `Dict[str, SensorHandle]`, jonka avaimina ovat nimi JA slot_id.
- `proj.config` — `project.json["config"]` sanakirja.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Käsittelytasot.** `capture()`, `grab()` ja `frame_stream()` käyttävät kaikki samaa `processing`
-tunnuksen, ja ketju on kumulatiivinen — jokainen taso suorittaa kaikki sitä ylempänä olevat toiminnot:

| Taso | Lähtö | Huomautukset |
| --- | --- | --- |
| `raw` | 1-kanavainen Bayer, anturin natiivi | Ei demosaicia. Päällekkäisyyksiä ei ole käytettävissä tällä tasolla. |
| `debayered` | 3-kanavainen väri (**oletus** `CameraHandle`:lle ja kaikille `grab()` / `frame_stream()`; `ArrayHandle.capture()`, `capture_smart()` ja `stream()` oletusarvo on `raw`): BGR muistissa (`grab()` / `frame_stream()`), tallennetaan RGB-muodossa `capture()`:n avulla | Taustatilan tilassa, kaappauksena: kalibrointipaketin pimeäsiirtymä (DSNU) raakakuvasta, sitten OpenCV:n reuna-herkkä demosaikki — ei lineaarisuus-LUT:ta, tasokenttää tai värien erottelua. Ainoa taso, joka toimii ilman taustakäsittelytilaa, on kamerakirjaston tavallinen bilineaarinen demosaikki (ei pimeäoffsetia). Tallennettu TIFF-tiedosto on fotometrinen RGB, joten käännä muistissa olevan kehyksen kanavat (`frame[..., ::-1]`), jotta sitä voidaan verrata tiedostoon. |
| `radiance` | float32, W/m²/sr/nm, **yksi taso kaistalta** | Täydellinen radiometrinen ketju: demosaikki + 3×3-sekoituksen purku (multispek) + DSNU + tasokenttä + NIST-asteikko, jossa valotus × vahvistus on jaettu pois, jotta arvot ovat absoluuttisia. |
| `reflectance` | uint16, 32768 = 1,0 (tai uint8, 255 = 1,0 — ks. `bit_depth`), **yksi taso kaistalta** | Säteilyvoimakkuus jaettuna alaspäin suuntautuvalla säteilyvoimakkuudella (ρ = π·L/E). Vaatii DLS/DAQ-lukeman — katso alla oleva huomautus. |
| `display` | 8-bittinen sRGB-tyyppinen | GUI-vastaava renderointi: CCM + valkotasapaino + gamma kameran aktiivisen väriprofiilin kautta. |

Vain suorassa (backend-vapaassa) tilassa on käytettävissä `debayered`: `grab()` toisella tasolla ohittaa
`NotImplementedError`, `capture()` ohittaa `processing`:n, ja `frame_stream()` nostaa
`RuntimeError`:n jokaisella tasolla. `reflectance` tarvitsee käyttökelpoisen alaspäin suuntautuvan lukeman — kehyksen päätepiste vetää
yhdistetyn DAQ:n automaattisesti kameran DLS-paikkaan, mutta ilman sidottua DAQ:ta ketju hylkää
heijastavuuden ulostulon ja merkitsee alennuksen rehellisesti palautettaviin metatietoihin sen sijaan, että
palauttamalla alemman tason tuotetta.

> **Heijastavuuden DN-asteikko — älä koodaa sitä kiinteästi.** LATTICE-heijastavuus käyttää `32768` = ρ 1,0 ja merkitsee
> XMP `Chloros:PixelScale=32768`; Survey3-heijastavuus käyttää `65535` = ρ 1,0 eikä sisällä
> `Chloros:*`-tunnisteita. Lue tunniste ja jaa arvo sillä. Se on määritelty uint16-alueella, joten se pysyy
> `32768`:nä kaikissa formaateissa, joissa suoritetaan uudelleenskaalaus (16-bittinen TIFF, 8-bittinen TIFF, 8-bittinen PNG/JPG, 32-bittinen prosentti)
> — normalisoi tallennettu tietotyyppi ensin takaisin uint16:ksi (×257 8-bittisestä, ×65535 float-tyypistä). Ainoa
> poikkeus on tiedosto, joka on peräisin versiosta 1.3.0 edeltävästä rakennuksesta: 8-bittinen TIFF-tiedosto *rajattiin* tuolloin, eikä sitä skaalattu uudelleen, joten
> sitä ei kuvata millään mittakaavalla, eikä se sisällä `PixelScale`- tai MicaSense-tupelia. Käsittele puuttuvaa tunnistetta
> LATTICE-heijastustiedostossa ”ei kelvollista mittakaavaa” -tilanteena, ei oletusarvona.

**Kalibrointitunnisteet: vain `raw`-tiedostossa.** Tallennuksen `raw`-tiedosto sisältää kameran kalibrointipaketin: `CalibrationJSON`-hyötykuorman ja MicaSense-yhteensopivat tunnisteet (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), jotka kuvaavat anturin raaka-arvoja. Jokainen muu tason tiedosto (`debayered`, `display`, `radiance`, `reflectance`, `index/`-renderöinti) sekä jokainen jatkuvasti tallennettu / monikameravirran kehys sisältää kameran tunnisteen (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) sekä kehyksen omat tallennustiedot, `XMP-mapir:FrameJSON` (valotusaika, vahvistus, lämpötila, pikselimuoto, tallennusaika, DAQ-vastaavuus), sillä jokaisessa `process()`-tuotteessa on tunniste. Käsittely etsii tällaisen tiedoston kalibroinnin sen sarjanumeron ja `CalBundleSha256`-tunnuksen perusteella Chlorosin kalibrointivälimuistista ja lataa paketin kerran, jos sitä ei ole tällä tietokoneella.

#### Useista kameroista luodut tiedostot

Useamman kuin yhden kameran kaistoista luotu tiedosto — monikaistapino (`capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index(save_multiband=True)`, `Registration.save_multiband()`), useista kameroista koostuva reaaliaikainen indeksi (`index.tif`, `index_LUT.png`) sekä array-kaappauksen `composite/`-kuva — nimeää kaikki siihen sisältyvät kamerat:

- **Kunkin-kamerakohtaiset luettelot, kaistajärjestyksessä** (yhdistelmä: taustakerros, sitten etualakerros; hakemistokerros, kamerat, joista sen kaistat ovat peräisin): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (kalibrointipaketti, jonka kunkin kameran kaistat ovat läpikäyneet, muuten — `capture_aligned_multiband()` / `start_aligned_stream()`-pino, jonka kaistat eivät ole läpikäyneet yhtään — kameran paketti, jonka Chloros on tallentanut välimuistiin, se, jonka mukaan sen tallennustiedostot on nimetty), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — rinnakkaiset luettelot, joista jokaisen kohde *n* kuvaa samaa kameraa; `-`, jossa kameralla ei ole arvoa (ei kalibrointipakettia, tuntematon objektiivi). Jokainen kamera on **yksi** kohde: jos sen paketissa on useita arvoja jollekin kentälle — paketin halogeenilamppusertifikaatit — ne yhdistetään merkinnällä `;` (`F-1479-1;F-1480-1;F-1481-1`), ei koskaan pilkulla, joten luettelossa on yksi kohde kameraa kohti riippumatta siitä, miten se tulostetaan.
- **EXIF `SerialNumber`** sisältää samat sarjanumerot, pilkuilla yhdistettyinä (`214000535,214000551`).
- Se ei sisällä **yhtään** yksittäisen kameran tunnusta (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): nämä nimeävät yhden kameran ja sen paketin, ja käsittely määrittäisi tiedoston kalibroinnin niiden perusteella. Useiden kameroiden kaistoista laskettu indeksi (`index.tif`, `index_LUT.png`) sisältää EXIF-tiedot `Model` `LATTICE` (laitteisto), ei yhden kameran `LATT-…`-mallia.**Yhden** kameran kaistojen tiedosto — kaistakohtainen TIFF-tiedosto, `capture_aligned_multiband(per_camera=True)`-kehys, yhden kameran pino tai reaaliaikainen indeksi — sisältää kyseisen kameran omat tunnistetiedot, kuten kaikki muutkin tallennustiedostot. `capture_aligned_multiband()`:n pino, kaistakohtaiset ja kamerakohtaiset tiedostot sekä jokainen `start_aligned_stream()`-pino ja sen kaistakohtaiset tiedostot sisältävät ryhmänsä tallennusajan (vertailukameran valotusaika, UTC), ja monokameran `per_camera`-kehys on yksi taso, kuten `bit_depth`-pino.**Kameran suunta (peilaus / kierto).** `grab()`- ja `frame_stream()`-tiedostojen kehykset ovat aina
kameran alkuperäisessä suunnassa kaikilla tasoilla. Tallennetut kuvat noudattavat kameran suunta-
asetusta (käyttöliittymän Mirror H / Mirror V / kierto):

- **`raw`- ja `debayered`-tiedostot ovat täsmälleen sitä, mitä anturi lukee**, asetuksista riippumatta — käännetty tai peilattu Bayer-
  mosaiikki olisi erilainen värisuodatinkuvio, ja kalibrointikartat ovat
  anturin omalla ruudukolla.
- **`display`-, `radiance`-, `reflectance`- ja indeksitiedostot on suunnattu** — käännetty kerran, kaikkien
  kalibrointilaskelmat. Niiden rajoitusmaskit ovat samalla ruudukolla.
- **Jokainen tallennustiedosto tallentaa asetuksen** XMP-tiedostoon: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, myötäpäivään,
  käännösten jälkeen) sekä `ChlorosOrientationApplied` — `false` raakakuvissa ja debayeroiduissa kuvissa, `true` suuntautuneissa
  tuotteissa. `ChlorosProcessing` luettelee kyseisen tiedoston pikseleihin sovelletut operaatiot (`none`, kun
  kyseessä on natiivitiedosto). Tämä kattaa jokaisen kehyksen jatkuvasta tallenteesta (kameran oma asetus,
  soveltamatta) sekä monikameravirrasta (identiteetti: sen kehykset ovat alkuperäisiä, kuten
  matriisikuvauksessa).
- **Projektin käsittely soveltaa tallennettua suuntausta** tuotteisiin, jotka se tuottaa raakakuvasta
  (ensin kalibrointi anturikoordinaateissa, seuraavaksi matriisin kohdistuksen vääristymän korjaus, viimeiseksi suunta),
  jotta ne vastaavat reaaliaikaisen tallennuksen tuotteita — mukaan lukien kalibroimattomat anturivaste- ja vinjetointikorjatut
  varatuotteet; sen debayeroitu tuote pysyy natiivina.
- **Chloros-kuvankatseluohjelma näyttää raaka- tai debayeroidun tiedoston, jonka suunta on käännetty tallennuksen mukaisesti** — vain näytöllä,
  tiedostoa ei muuteta; tiedosto, josta puuttuu tallennus, näytetään tallennetussa muodossaan.
- **Tallennukset, joista puuttuu tallennus, käsitellään natiiveina.** Ennen versiota 1.3.0 yhden kameran tallennus
  käänsi raaka- ja debayeroidut tiedostonsa itse; Chloros ei osaa erottaa tällaista raakatiedostoa alkuperäisestä eikä
  yritä arvata.
- **Ryhmätallenteet säilyttävät oletusarvoisesti jokaisen jäsen-tiedoston alkuperäisessä muodossa** ja tallentavat tunnisteen;
  `ArraySession.capture(orient=True)` muuntaa niiden tulokset (katso matriisikuvauksen säätimet).

#### Kaistakohtainen tulostus ja sekoitetut matriisit

Tasoilla `radiance` ja `reflectance` kamera palauttaa **yhden tason kutakin kyseisen
kameran kaistaa kohti** — kolme M3C:lle (sen kolme sekoittamatonta kaistaa, BGR-kanavajärjestys), yksi M3M:lle (sen
yksi kapeakaistainen suodatin). Monokameran replikoidut tasot on yhdistetty, joten **sekoitettu
M3C/M3M-matriisi tuottaa eri matriisimuotoja sarjanumerokohtaisesti**. Lue `bands`-avain kehyksen
metatiedoista selvittääksesi, mikä kukin taso on; älä päätä sitä kanavien lukumäärän perusteella:

```python
for serial, (arr, hdr) in arr_handle.grab(processing="reflectance",
                                          with_metadata=True).items():
    print(serial, arr.shape, hdr["bands"])   # ('…', (1536, 2048, 3), ['nir','green','red'])
                                             # ('…', (1536, 2048, 1), ['F685'])
    rho = arr.astype("float32") / hdr["pixel_scale"]
```

Tarkista aina `hdr["processing_level"] == "reflectance"` ennen arvojen käyttöä. Alennus
(ei DAQ-lukemaa, ei kalibrointipakettia, tuntematon kaistajärjestys) raportoidaan kentissä `processing_level` ja
`radiometric_refused` sen sijaan, että se korotettaisiin — saamasi matriisi on edelleen kelvollinen kehys, mutta
ei ole pyytämäsi tuote. Alennettu tallennettu kaappaus sisältää XMP-tiedostossaan saman koodin
kuin `Chloros:RadiometricRefused` (esim. `no-band-layout`, `no-dls-snapshot`).

#### Yksi laukaisu lukemista kohti — `group_id`

**Ryhmän jäsenet toimitetaan ryhmän synkronoidusta kehysryhmästä, ei kunkin kameran omasta
kaappauspuskurista.** Laitteistollisesti synkronoidun ryhmän kamerat altistuvat yhdessä, mutta N kameraa kohti tehtävää lukemista
ei *näytetä* yhdessä — kunkin kameran viimeisimmän puskuroidun kehyksen lukeminen ylittää laukaisurajat
rajojen yli, ja yksikin pudonnut kehys synkronoi ryhmän ilman, että kukaan huomaa sitä.

Ryhmästä tarjoaminen on välttämätöntä, mutta se ei yksinään riitä: jokainen pyyntö ottaa silti
ottaa tilannekuvan *viimeisimmästä* ryhmästä omalla hetkellään, ja tallennussilmukka voi julkaista välissä.
Elävällä 4-kamerajärjestelmällä mitattuna **12 rinnakkaisesta 20 lukemasta ulottui kahden ryhmän yli** — aina
täsmälleen vierekkäin. Joten lukeminen on **kiinnitetty**:

1. Jokaisessa kehyksessä on `group_id`, järjestelmän monotoninen julkaisulaskuri.
2. `grab()` lukee pääkameran (tai, jos pääkameraa ei voida palvella, seuraavan kameran, joka pystyy siihen),
   ja kiinnittää sitten loput kamerat siihen ryhmätunnukseen, johon se päätyi (`?group_id=N` HTTP-
   päätepisteessä).
3. Taustapalvelu säilyttää viimeisimmät ryhmät **vähintään puolen sekunnin ajan**, joten sisarryhmä, joka on jo
   ohitettu myöhemmillä julkaisuilla, palvelee edelleen kyseistä laukaisinta — riittävän kauan, jotta kamerakohtaisen
   heijastavuusdatan vienti kestää edes 9 kameran täyssensorijärjestelmässä. Kiinnitys, jota taustapalvelin ei
   enää pidä yllä, hylätään koodeilla `409` ja `group_aged_out: true` sen sijaan, että siihen vastattaisiin hiljaisesti
   eri laukaisulla — `grab()` lukee joukon uudelleen.

Uudelleenmitattu kiinnityksellä: **20 lukemasta 20 palautti yhden laukaisun.** `grab()` hoitaa tämän puolestasi;
tarvitset `group_id=`:ää vain, kun ohjaat HTTP-päätettä suoraan.

- Kameraa, jota ei voida palvella kiinnitetyssä ryhmässä — se puuttuu ryhmästä tai hylkää pyynnön
  (sillä ei ole `band`:ää, tasoa, jota se ei voi tuottaa) — **jätetään pois sen sijaan, että se täytettäisiin takautuvasti**
  toisen laukaisijan avulla, ja muut kamerat palautetaan edelleen.
- `grab()`:n palauttama sanakirja on `GrabResult` (tavallinen `dict`-alaluokka;
  `from chloros_sdk import GrabResult`): `.errors` yhdistää jokaisen
  jätetyn sarjan sen poikkeukseen, jonka lukeminen aiheutti, ja `.group_id` on jaettu ryhmä. Tyhjä
  `.errors` tarkoittaa, että jokainen kamera kuuluu tähän ryhmään.
- Itsenäisellä kameralla ei ole ryhmää, se raportoi `group_id: None`:n ja **hylkää** `group_id=`-
  pin-kutsun sen sijaan, että sivuuttaisi sen.

```python
frames = arr_handle.grab(processing="reflectance")
for serial, err in frames.errors.items():
    print(f"SN:{serial} omitted: {err}")       # e.g. status=400 ... band 'F685' not in [...]

frames = arr_handle.grab(processing="reflectance", with_metadata=True)
ids = {hdr["group_id"] for _a, hdr in frames.values()}
assert len(ids) == 1, f"straddled triggers: {ids}"     # belt and braces
```

Sama laskuri julkaistaan **jokaisessa kamerakohtaisessa MJPEG-virrassa**, joka kuuluu ryhmän jäseneen —
esikatselussa (`/api/camera/stream/<serial>`), merkityssä esikatselussa (`/stream-annotated`) ja
heijastusvirta (`/stream-reflectance`) — nimellä `X-Chloros-Group-Id` kussakin osassa, lisäksi
`X-Chloros-Frame-Id` ja `X-Chloros-Timestamp-Ns`. Virrat ovat jatkuvia eivätkä
tiettyyn ajankohtaan sidottuja, joten ne yhdistetään keräämällä osia ja ryhmittelemällä kyseisen otsikon perusteella — mitään
asetettavaa nastaa ei ole. Itsenäisellä kameralla ei ole ryhmää. Katso
[Kamerakohtaiset MJPEG-virrat](#per-camera-mjpeg-streams-no-rtsp).

**`frame_stream()` ei koskaan vaikenna.** Tikki, joka epäonnistuu syystä, joka voi korjaantua itsestään,
ohitetaan — kameravirta ei tuota sille mitään tulosta, taulukkovirta palauttaa ne kamerat, jotka
vastasivat (mahdollisesti ei yhtään). Nämä syyt ovat: ei vielä kehystä; kameran tallennussäie keskeytyi
sekunniksi tai kahdeksi (`404 not streaming`, johon taustapalvelu vastaa aina, kun toinen kamera muodostaa
tai katkaisee yhteyden, tai asetusten muutos tasapainottaa poolia uudelleen); taustapalvelimen uudelleenkäynnistys tai `5xx`; kohdistus
vielä vakiintumassa (`409 … retry shortly`); ja heijastavuuden hylkäys, koska DAQ:lla ei ole vielä lukemaa
(`no-dls-snapshot`, vanhentunut tai tyhjä lukema). Hylkääminen seuraavalla tikillä vain toistuisi —
tuntematon `band`, `bit_depth=8` säteilyvoimakkuudessa, kalibrointipaketti ilman säteilyvoimakkuusasteikkoa, heijastavuuden
hylkäys, jossa **ei ole lainkaan DAQ:ta** (yhtään sidottu kameraan tai sen matriisiin, yhtään kytketty
taustajärjestelmään: hylkäyksen `dls_source` on `"none"`), mikä tahansa muu `4xx` —
aiheuttaa `chloros_sdk.FrameFetchError` (`RuntimeError`, joka sisältää `.status`:n ja `.body`:n)
kerralla. Samoin tekee **60 sekuntia** ohitettuja tikkejä ilman välissä olevaa kehystä, riippumatta siitä, mikä on `fps` — riittävän
pitkä aika, jotta kohdistuksen vakiintuminen ja taustaprosessin uudelleenkäynnistys ehditään odottaa, mutta riittävän lyhyt, jotta kamera, joka on
todella poissa, lopettaa lähetyksen. Tämä luovutusvirhe on myös `FrameFetchError`, niin kameran kuin
ryhmänkin osalta, ja viimeisen lukemisen oma poikkeus on `.errors` (ja kameran osalta `__cause__`).
Ryhmän osalta ”refused” tarkoittaa, että jokainen kamera kieltäytyi; poikkeus `.errors` sisältää kunkin kameran
syyn.

**Taulukko `frame_stream()` tuottaa jokaisen laukaisun kerran** (samoin kuin `ArrayHandle` ja `ArraySession`).
Tick, jonka uusin sarja on jo tuotettu laukaisija, odottaa seuraavaa laukaisijaa sen sijaan, että
tuottaisi saman sarjan uudelleen, joten virta kulkee hitaammalla nopeudella kuin `fps` tai taulukon oma kehysnopeus
, eivätkä peräkkäiset sarjat koskaan jaa samaa `group_id`-arvoa. Julkaisemisen lopettava taulukko tuottaa
`FrameFetchError`:n (&quot;ei uutta laukaisua 60 sekunnin ajan&quot;) sen sijaan, että se toistaisi viimeisimmän sarjansa. Ilman tätä
virta, jolta pyydettiin enemmän kuin taulukko julkaisi — tai joka käynnistyi heti yhteyden muodostamisen jälkeen — jakoi
toistoja: yhdessä mitatussa ajossa 20 sarjassa oli 11 erillistä ryhmätunnusta.

```python
from chloros_sdk import FrameFetchError

try:
    for arr in cam.frame_stream(processing="radiance", bit_depth=8):
        my_analysis(arr)
except FrameFetchError as exc:
    print(exc.status, exc.body)        # the HTTP status and the backend's reason
```

#### Yhden kaistan vieminen — `band`

`grab()` / `frame_stream()` ja `band="<name>"` viedään vain yksi kameran kaistoista
yksikanavaisena `(H, W, 1)`-taulukkona — sama muoto kuin monokameran tuottama, joten yhden kaistan ulostuloa
käsitellään samalla tavalla riippumatta siitä, onko se peräisin M3M-kamerasta vai M3C-kameran jakamisesta. FRGN:n punainen kaista:

```python
red = cam.grab(processing="reflectance", band="red")        # (1536, 2048, 1)
```

Hyväksytyt kirjoitusmuodot ovat täydellinen asettelunimi (`Red_660`, `NIR_850`, `Green_550`) tai sen lyhyt
otsikko (`red`, `nir`, `green`), kirjainkokoa ei huomioida. Tuntematon nimi hylätään ja saatavilla olevat
nimet luetellaan.

> **Valinta tapahtuu nimen perusteella, ei kanavanumeron perusteella — tarkoituksella.** ”Kanava 1” on `Green_550`
> matriisin BGR-kanavajärjestyksessä, mutta *punainen* suodatinkoodinkirjainjärjestyksessä (`RGN`), ja nämä kaksi
> eivät täsmää. Väärän valitseminen tuottaa täysin uskottavan kehyksen väärältä kaistalta, jota
> mikään alavirran laite ei pysty havaitsemaan. Tästä syystä numeerista indeksiä ei ole.

#### Virtojen kohdistaminen — `align`

`align=True` muokkaa kehyksen taulukon yhteiseen vertailukehykseen. Jokainen kamera muokataan
**saman** profiilin resoluution ja rajausalueen (ROI) kautta, joten erikseen avatut kamerakohtaiset virrat
tulevat ulos **keskenään rekisteröityinä** — niitä ei tarvitse pyytää yhdessä:

```python
frames = arr_handle.grab(processing="reflectance", align=True)   # {serial: array}
# or one MJPEG stream per camera, each co-registered on its own:
#   GET /api/camera/<serial>/stream-reflectance?align=1
```

Vaatimukset ja toiminta:

- Kameran on oltava **laitteistollisesti synkronoidussa** ryhmässä (M8-synkronointikaapeli), jolla on kohdistusprofiili
  — yhteisrekisteröinti edellyttää, että jäsenet altistuvat samanaikaisesti, mitä ohjelmisto- tai
  vapaasti toimiva ryhmä ei tarjoa. Muussa tapauksessa hylätään virheellä `409`.
- **Hylätään, ei koskaan hiljaisesti kohdistamatta.** Tallennuspolku voi tallentaa kohdistamattomana varoituksella,
  koska tiedosto säilyttää alkuperätunnisteensa ja voidaan rekisteröidä uudelleen myöhemmin; suorassa lähetyksessä ei ole
  tällaista toista mahdollisuutta, joten puuttuva profiili on virhe eikä varajärjestelmä.
- **Kohdistuksen pyytäminen käynnistää sen.** `align`:n pyytäminen profiilittomalta matriisilta käynnistää
  kalibroinnin ja palauttaa `409`:n sen kanssa, mitä se odottaa (esim. *&quot;odotetaan valotuksen
  vakiintumista arvoon 214000551 (1/3)&quot;*) — yritä uudelleen, niin se onnistuu, yleensä muutaman kymmenen sekunnin kuluessa.
  Tämä on tärkeää päättömässä tilassa: yhteyden muodostuksen yhteydessä tapahtuva automaattinen kohdistus luopuu 30 sekunnin kuluttua, ja jokainen yritys
  tuon ajanjakson sisällä ohitetaan, kun älykäs automaattinen valotus (smart-AE) on vielä vakiintumassa, joten matriisi, jonka valotus
  vakiintuu hitaasti, jäi aiemmin ilman profiilia koko istunnon ajaksi eikä sitä ollut mahdollista saada
  ilman työpöydän yhdistettyä näkymää avaamatta. `alignment_status` raportoi myös
  `ae_align_holdouts`, jossa mainitaan vielä vakiintumassa olevat kamerat ja niiden edistys.
- Anna jokaiselle matriisin virralle **sama `max_h`**, tai niiden rajaukset päätyvät eri pikseliruudukkoihin
 .
- Kohdistus suoritetaan *ennen* bittisyvyyden koodausta, joten vääristymäinterpoloi täyden tarkkuuden
  arvot.

#### `bit_depth` — tarkkuuden vaihtaminen kaistanleveyteen

`grab()` ja `frame_stream()` hyväksyvät `bit_depth=16` (oletus) tai `bit_depth=8`:

| `bit_depth` | Heijastavuuskoodaus | Kustannus |
| --- | --- | --- |
| `16` | uint16, 32768 = ρ 1,0, varaa ρ 2,0:een | Täysi tarkkuus. |
| `8` | uint8, joka kattaa `0..refl_max` (oletusarvo ρ 1,0) | Puolet tavuista; ~0,004 ρ:n kvantisointi oletuskatossa. Katon ylittävät arvot leikkautuvat — katso alla. |

##### 8-bittinen yläraja — `refl_max`

`refl_max` määrittää ρ:n, jonka 8-bittinen alue kattaa, joten `pixel_scale = 255 / refl_max` ja ρ on edelleen
`pixel / pixel_scale`. `grab()` ja `frame_stream()` käsittelevät sitä avainsanana sekä kameran että
taulukon käsittelijöissä (`?refl_max=` HTTP-päätelaitteessa). Nosta arvoa, jotta saat lisää liikkumavaraa heijastaville
korostuksille, suhteellisesti karkeammilla askelilla:

```python
frame = cam.grab(processing="reflectance", bit_depth=8, refl_max=1.5)   # 0..1.5 in 0..255
```

| `refl_max` | `pixel_scale` | askel |
| --- | --- | --- |
| `1.0` (oletus) | 255 | 0,0039 ρ |
| `1.5` | 170 | 0,0059 ρ |
| `2.0` | 127,5 | 0,0078 ρ |

> **Automaattinen valotus ei pysty korjaamaan 8-bittistä leikkausta, ja sen käyttäminen tähän tarkoitukseen vain pahentaa tilannetta.**
> ρ = π·L/E, kun valotus ja vahvistus on jaettu pois, joten ρ on kuvauskohteen ominaisuus, ei
> kameran asetus — mitattuna reaaliaikaisella matriisilla valotuksen puolittaminen siirsi keskimääräistä ρ-arvoa 1,3 %. Sen sijaan
> valotusta alueen ”sovittamiseksi” ajaa pikselit sen sijaan **anturin** saturaatioon, mikä
> aliarvioi L:n ja vääristää ρ:n **alaspäin** ilman, että virhettä ilmoitetaan (mitattu: maks. ρ 1,203 → 0,985
> 2× valotuksella, keskiarvo −23 %). Valitse alue `refl_max`:llä; anturin pitäminen
> kyllästymättömänä on jo automaattisen valotuksen (AE) tehtävä, ja se suojaa pikselin ρ:n *pätevyyttä* sen sijaan, että se suojaisi sen arvoaluetta.

**Lue `reflectance_clipped_fraction`, ei lippua.** `reflectance_clipped_at_1_0` laukeaa
yhdestä pikselistä — todellisessa tilanteessa se ilmoitti `True`, vaikka osuus oli `0.000001`, yksi pikseli
miljoonasta. Murto-osa kertoo, pitäisikö ylärajaa nostaa.

**`bit_depth=8` koskee vain heijastavuutta.** Sen pyytäminen `radiance`:lle hylätään sen sijaan, että
se toimitettaisiin: heijastavuudella on kiinteä alue 0–1, joka säilyy 8-bittisessä koodauksessa, kun taas säteilyllä ei ole
kiinteää aluetta, joten sen sovittaminen 8 bittiin vaatii kehyskohtaisen prosenttipisteiden venytyksen, jonka kuvaus
muuttuu jokaisella kehyksellä — kyseessä on kuva, ei mittaus.

#### Reaaliaikainen heijastavuusvirta (MJPEG)

`GET /api/camera/<serial>/stream-reflectance` tarjoaa 8-bittistä kaistakohtaista heijastavuutta nimellä
`multipart/x-mixed-replace` — M3C-muodossa 3-kanavaisena JPEG-kuvana, M3M-muodossa harmaasävyisenä. Kyselyparametrit:
`fps` (oletus 10), `jpeg_q` (oletus 95), `max_h` (0 = natiivi), `band` (yksi nimetty kaista, kuten
edellä), `align` (`1` rekisteröitäväksi yhdessä — samat säännöt kuin SDK:n `align`), `refl_max` (8-bittisen
koodauksen yläraja, kuten yllä). Jokainen osa sisältää
`X-Chloros-Group-Id`, `X-Chloros-Frame-Id`, `X-Chloros-Timestamp-Ns`, `X-Chloros-Bands`,
`X-Chloros-Pixel-Scale`, `X-Chloros-Aligned` ja `X-Chloros-Reflectance-Clipped`.
**Eri kameroista peräisin olevat osat, joissa on sama `X-Chloros-Group-Id`, ovat sama laukaisija** —
näin yhden matriisin useat kamerakohtaiset virrat kootaan uudelleen, ja tämä taataan
palvelinpuolella, koska matriisin jäsenet toimitetaan synkronoidusta kehysryhmästä (katso
[Yksi laukaisija lukukertaa kohti](#one-trigger-per-read--group_id)). Se palauttaa arvon `409`, kun heijastavuusarvoa ei ole
saatavilla yhteyden muodostushetkellä, ja lopettaa virran, jos se katoaa kesken ajon, joten pysähtynyt
virta tarkoittaa ”heijastavuusarvoa ei enää ole” eikä ”vielä kunnossa, vain staattinen”.

> **JPEG on häviöllinen — nämä pikselit ovat mittaustuloksia.** Pakkausvirhe näkyy suoraan
> heijastusarvoissa ja on pahimmillaan reunoilla, joissa materiaalien rajat sijaitsevat. Laatu
> on tästä syystä oletusarvoisesti 95. Jos tarvitset luvut muuttumattomina, käytä
> `grab(processing="reflectance")` / `frame_stream(...)`, jotka tuottavat saman tuloksen ilman
> koodekkia polussa (ja `bit_depth=8`, jos rajoitteena on kaapelikustannus, ei koodekki).

> **EXIF-tiedot siirretään vientiin.** `process()` kopioi lähdekuvauksen GPS-lohkon
> **ja sen ExifIFD:n** jokaiseen tuotteeseen, joten vienti sisältää `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` ja `CameraSerialNumber` sekä
> georeferenssi. `FocalLength` on se, jonka perusteella Pix4D laskee maastonäytteen etäisyyden — ilman sitä
> rekonstruktio käyttää täysin väärää mittakaavaa (eräässä mitatussa tapauksessa 411 metrin alue
> muuttui 47,8 kilometrin alueeksi). Kopio ei ole tarkoituksella `-all:all`: IFD0:n rakenteelliset tunnisteet rikkovat
> LATTICE-tulostetta, ja `ExifImageWidth`/`Height` on jätetty pois, koska ne kuvaavat lähteen
> kuvausvaihetta eikä vietyä rasteria.

Kaappausvaiheen alilippuja (koskevat radiometrisiä tasoja — `radiance`, `reflectance`, `display`):

| Lippu | Oletus | Merkitys |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + tasokenttä + 3x3-sekoituksen erottelu + NIST-radiometrinen asteikko. |
| `apply_white_balance` | `True` | WB LUT. Tukee DLS:ää, kun DAQ on liitetty kameraan. |
| `apply_index` | `False` | Kasvillisuusindeksin arviointi. |
| `index_expression` | `None` | Kaavan ohitus. Tyhjätyhjä → indeksi otetaan automaattisesti käyttöön. |
| `annotated` | `False` | GUI-koristeiden (zebra/ruudukko/peaking) päällekkäisyys. Ei käytettävissä `raw`:lle. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Palautusarvo on `chloros_sdk.CapturePaths`, tyyppiä `CapturePathMap`, ei `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` on `Dict[str, Union[str, List[str]]]`: yksitasoinen
> `processing` antaa jokaiselle sarjanumerolle yhden polun, kun taas `processing="all"` antaa sille
> **järjestetyn luettelon** kaikista kyseiselle kameralle tallennetuista tuotteista. `ArrayHandle.capture()` ei ota vastaan
> mitään `levels=`, ja se ohittaa avainsanaparametrit, joita se ei tunne (`levels`, `daq`,
> `render_index`, `force_daq`), joten välitä vain tässä esitetyt (`aligned=False`
> tallentaa natiivitiedostoja, kuten `ArraySession.capture()`); käytä
> `ArraySession.capture()`:ää kyseisille ohjauselementeille. Suora yhdistetty komposiitti, jos sellainen lähetetään suoratoistona, saapuu ylimääräisen
> `"combined"`-avaimen alle eikä sarjanumeron alle. Koodi, joka olettaa `str`:n, kaatuu
> listamuodossa ilman, että tyyppitarkistaja ilmoittaa virheestä — merkinnässä sanottiin, että `Dict[str, str]`
> jonkin aikaa luettelo-muodon julkaisun jälkeen, minkä vuoksi alias on olemassa. Normalisoi
> kun haluat tasomaisen muodon:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

> **Mitä ei voitu kirjoittaa: `.failed`.** Kartta, jonka `capture()` palauttaa, on
> `chloros_sdk.CapturePaths` — tavallinen `dict`-alaluokka, joten kaikki yllä oleva toimii
> muuttumattomana — joka sisältää myös `.failed`: tiedostot, joita taustaprosessi ei pystynyt kirjoittamaan,
> samat merkinnät kuin [`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed)
> (`serial`, `level`, `error` ja `disk_full: True` täyteen täyttyneellä levyllä). Osittainen
> tallennus palauttaa silti kaikki polut, joihin on kirjoitettu; täyttyneellä levyllä kartta on
> tyhjä ja tiedosto `.failed` kertoo syyn. `.skipped` luettelee kamerakohtaiset ohitukset ja
> `.group_seq` synkronoidun ryhmän, ja `.alignment` / `.oriented` ovat
> taustapalvelimen tiedot, kuten `CaptureResult`:ssä. `capture_smart()` palauttaa ne muodossa
> `result["frames"].failed` ja `result["failed"]`.
>
> ```python
> paths = arr.capture("./out", processing="all")
> if any(f.get("disk_full") for f in paths.failed):
>     raise SystemExit("drive full -- free space and capture again")
> ```

### Taulukon kohdistus

`ArrayHandle` paljastaa koko kohdistuspinnan. Profiilit ovat oletusarvoisesti vain istuntokohtaisia — kutsu `export_alignment()` nimenomaisesti, jotta ne tallennetaan pysyvästi.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Manual control points: each camera's pixel coordinates of the same
# features, the reference camera's included (>= 2 per camera). Every one
# of num_frames is solved from these points.
arr.calibrate_alignment(method="manual", model="rigid", control_points={
    "214701173": [[412, 300], [1630, 310], [1020, 1180]],   # reference (master)
    "214701292": [[418, 297], [1637, 306], [1027, 1177]],
})

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Kohdistus yhteyden muodostuksen yhteydessä

`connect_all(align=...)` voi kohdistaa jokaisen matriisin automaattisesti yhteyden muodostuksen yhteydessä:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Jos asetusta ei ole määritetty, käytetään oletusarvoisesti `project.json["config"]["auto_align_on_connect"]`-komentoa.

### `SensorHandle`

Tallennettu valosensori (DAQ-U, DAQ-M tai DAQ-E), jonka `connect_all()` avaa taustapalvelimen [DAQ-anturipoolissa](#daq-sensor-sessions) — samassa poolissa, jota `connect_daq_sensor()` ja `chloros-cli daq pool-connect` käyttävät.

```python
reading = proj.sensors["Sky"].read()
if reading is not None:
    spectrum, is_saturated, integration_time, x, y, z = reading
```

- `read()` odottaa anturin **seuraavaa** spektriä — spektri, joka on jo puskuroitu kutsun hetkellä, ei lasketa — ja palauttaa arvon `(spectrum, is_saturated, integration_time, x, y, z)` tai `None`, jos mitään ei saavu 5 sekunnin kuluessa. Keskeytetty virta jatketaan. `spectrum` on yksikössä W/m²/nm, kun anturi on kalibroitu.
- `.sensor` on anturin [`DAQSensorSession`](#daq-sensor-sessions), kaikissa muissa tapauksissa: `latest(n=...)`, `status()`, `record_start()` / `record_stop()`.
- `.slot_id`, `.model`, `.serial`, `.name` ja `.settings` ovat tallennettuja merkintöjä tiedostosta `sensors.json`.

**Mitä `connect_all` palauttaa.** Jokainen anturi avataan sen tallennetussa osoitteessa — sen COM-portissa, Bluetooth-osoitteessa tai DAQ-E-isäntäkoneessa — ja jos projekti on tallentanut anturin tunnisteen, kyseessä on oltava juuri kyseinen anturi: eri laite kyseisessä osoitteessa hylätään, koska se ei ole kytketty. Sen tallennettu integraatioaika, kehysten keskiarvoistaminen ja automaattinen valotus otetaan käyttöön, samoin kuin sen korkkikorjaus:

| Tallennettu korkkiprofiili | Yhdistetty |
| --- | --- |
| Korkkiprofiili (esim. `fov_30`) | Kyseinen profiili. |
| Ei valittu | `sunshine_cosine`, yhteyden oletusprofiili, tallennettu oletetusti (`cap_id_source=auto_default`), joten käsittely antaa varoituksen. Anturit toimitetaan ilman suojakantta: valitse asennettu suojakansi. |
| `none` — valittu käyttöliittymässä **Ei mitään (paljas anturi)** | Ei suojakannen korjausta. |

DAQ-E-S ei ota huomioon suojuksen profiilia riippumatta siitä, mikä on tallennettu: sen hajotin on osa sen kalibrointia (katso [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor)). Aikaisemmalla Chloros-ohjelmalla tallennetussa projektissa paljaa anturia ei ollut valittuna, joten se muodostaa yhteyden uudelleen `sunshine_cosine`-laitteeseen; valitse käyttöliittymässä jälleen **Ei mitään (paljas anturi)** ja tallenna projekti.

Anturi, jonka toinen asiakas on jo avannut poolissa (CLI:ssä tai toisessa skriptissä), liitetään sellaisenaan ilman asetusten uudelleenasettamista, ja `disconnect_all()` jättää sen käynnissä; `connect_all()`:n avaama anturi vapautetaan. Kuten `connect_daq_sensor()`:n tapauksessa, Chloros-käyttöliittymä **ei** kuulu pooliin: sulje anturi käyttöliittymässä ennen projektin liittämistä skriptistä.**Vaatimukset ja raportti.** Valoanturit edellyttävät käynnissä olevaa Chloros-taustapalvelinta, joka on kirjautunut maksulliselle Chloros+-tasolle (Copper tai korkeampi), eikä `connect_all()` koskaan käynnistä sellaista. Jos taustapalvelinta ei ole käynnissä, kaikki tallennetut anturit luetellaan kohdassa `report["sensors"]["errors"]`, jossa tämä mainitaan. Myös anturi, jota taustapalvelu ei voi avata, näkyy siinä luettelossa taustapalvelun ilmoittaman syyn kera; `report["sensors"]["missing"]` pysyy tyhjänä valoantureiden osalta.

---

## Suora laitteisto (ilman taustapalvelua)

Kun haluat, ettei LATTICE-kameroiden (CI, päättömät robotit, sulautetut järjestelmät) toiminta ole lainkaan riippuvainen taustapalvelimesta, tuo `lattice_sdk` suoraan — se viedään uudelleen `chloros_sdk`:n kautta. Huomautus `CAMERA_AVAILABLE`:sta: `lattice_sdk` on SDK-wheelissä, mutta se vaatii Arena SDK -ajoympäristön ja OpenCV:n (`cv2`) läsnäolon, eikä wheel asenna kumpaakaan. Valosensorit kulkevat backendin kautta — [`connect_daq_sensor()`](#daq-sensor-sessions) tai projektin `proj.sensors`:n kautta — yhtä poikkeusta lukuun ottamatta, joka on DAQ-M-kaistan suuruusarvojen apuohjelma [`DLS`](#daq-m-band-magnitudes--dlsband_magnitudes), joka vaatii myös `bleak`:n.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Esiasetukset ja laukaisija

Neljästä esiasetuksesta kolme toimii **vapaakäynnillä**: kamera valottaa jatkuvasti ja
`capture()` palauttaa seuraavan kehyksen. `triggered` on poikkeus — se asettaa
kameran valmiustilaan linjan 2 laitteistoreunan varalle, joten se ei tallenna mitään ennen kuin sellainen saapuu.

| Esiasetus | Laukaisija | Käytä, kun |
| --- | --- | --- |
| `default` | vapaakäynti | yleiskäyttö |
| `high_speed` | vapaakäynti | 8-bittinen, 60 fps:n rajoitus, lyhyt valotusaika |
| `high_quality` | vapaakäynti | 12-bittinen, ei kuvataajuusrajoitusta — tavallinen valinta valokuville |
| `triggered` | **valmiustila, linja 2** | kamera on kytketty M8-synkronointikaapeliin ja jokin muu laukaisee sen |

Jos valitset `triggered` (tai asetat itse `trigger_mode="On"`) ilman, että
mitään ohjaa linjaa 2, jokainen `capture()` menee aikakatkaisuun — oikein, koska pyysit
kameraa odottamaan. SDK selittää tämän, kun se tapahtuu; katso
[SC_ERR_TIMEOUT tallennuksen aikana](#direct-hardware-backend-free).

> **Huomautus — ”GVSP-tunnistus” / `SC_ERR_TIMEOUT -1011`-viestit yhteyden muodostamisen yhteydessä eivät ole virheitä.**
> Yhteyden muodostamisen yhteydessä SDK yrittää neuvotella**jumbo-kehyksistä** (9000 tavun GVSP-paketit) suuremman läpimenokapasiteetin saavuttamiseksi. Suorassa pisteestä pisteeseen -verkkokorttiliitännässä (esim. link-local-osoite `169.254.x.x`) verkko ei yleensä kykene siirtämään jumbo-kehyksiä, joten tämä koe päättyy aikakatkaisuun ja lokiin kirjataan esimerkiksi seuraavanlaisia rivejä:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Tämä on **suunniteltu varajärjestelmä**: SDK palaa automaattisesti tavallisiin 1500 tavun paketteihin, ja kamera jatkaa yhteyden muodostamista normaalisti (seuraavat `[chunk-enable …]`-rivit ovat osa normaalia yhteydenmuodostussekvenssiä). Kuvaus toimii edelleen.
>
> Voit ohittaa tämän testin, mutta **se ei ole pelkkä lokitiedostojen vaimentaja — se poistaa jumbo-kehykset käytöstä.** Kamera vastaa Don&#x27;t-Fragment-ping-kyselyihin vain enintään 1500 tavun paketeilla riippumatta siitä, kuinka hyvä verkko on, joten pelkällä ping-testillä ei voi koskaan havaita jumbo-kehyksiä; vain tämä koe pystyy siihen. Jos poistat sen käytöstä, kamera käyttää ikuisesti tavallisia 1500 tavun paketteja, missä tahansa verkossa:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Se kannattaa vain verkossa, josta *tiedät*, ettei se tue jumbo-kehyksiä, jolloin se säästää noin sekunnin yhteyden muodostusaikaa kameraa kohti. Koska kyseessä on todellinen kompromissi eikä pelkkä kosmeettinen muutos, SDK ilmoittaa siitä nyt, kun käytät sitä:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Jätä se rauhaan, ellet ole siihen syytä.**Jos asetus jätetään käyttöön, jokainen kamera mitataan kerran jokaisessa backend-istunnossa ja uudelleen, kun sen yhteys muuttuu (toinen sovitin, toinen yhteysnopeus tai katkennut yhteys, kuten kytkimen vaihtaminen aiheuttaa); uudelleenkytkennät käyttävät tulosta uudelleen.**Kun muutos on tapahtunut kytkimen takana, joka on pysynyt toiminnassa, käynnistä backend uudelleen mittausta varten.** Ensimmäinen yhteyden muodostus voi tässä tarkistuksessa kestää toisinaan muutaman minuutin, ja loki ilmoittaa siitä, kun näin tapahtuu.
>
> Jos *haluat* jumbo-läpimenon, ota käyttöön jumbo päästä päähän (NIC MTU 9000 + kytkin, joka läpäisee ne) tai kiinnitä se `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`-asetuksella, kun tiedät, että linkki tukee sitä — vaikka komento-kohtainen `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` on suositeltavampi kuin pysyvä asetus, koska kiinteästi asetettu koko ohittaa testauksen ja estää sopeutumisen edessä olevaan verkkoon. **Jokaisen** reitillä olevan laitteen on välitettävä jumbo-paketteja — mukaan lukien kaikki PoE-jakajat tai -syöttölaitteet, jotka ovat tavallisin syy siihen, että muuten jumbo-paketteja tukeva kokoonpano ei pysty välittämään niitä.

> **`SC_ERR_TIMEOUT -1011` `capture()`- tai `grab*()`-tilan aikana on eri ongelma — kyseessä on todellinen virhe.**
> Yllä oleva huomautus koskee vain**connect-time probe**-toiminnon kirjaamaa `-1011`-virhettä. Sama virhe, joka ilmenee**kaappauksesta**, tarkoittaa, että kamera on kytkeytynyt kunnolla, mutta ei lähetä kuvia:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Selvä merkki on kamera, jonka *ohjaus*kanava on kunnossa — tunnistus toimii, asetukset ja `[chunk-enable …]`-kirjoitukset onnistuvat — mutta *jokainen* kehys ylittää aikarajan.
>
> **Yleisin syy on, että kamera on asetettu laitteistolähtöön.** Virheiden `trigger_mode="On"` ja `trigger_source="Line2"` tapauksessa kamera ei lähetä mitään, ennen kuin M8-synkronointikaapelissa ilmenee sähköinen reuna. Jos kyseistä linjaa ohjaavaa kaapelia ei ole, jokainen kuvanotto odottaa ikuisuuden. Kamera ei ole rikki eikä verkossa ole vikaa — se toimii täsmälleen niin kuin sille on käsketty.
>
> `CameraSettings()` sekä mallit `default` / `high_speed` / `high_quality`-esiasetukset toimivat vapaakäynnillä, ja valinta, jonka aikakatkaisu tapahtuu valmiustilassa, selittää tilanteen sen sijaan, että tulostaisi pelkän `-1011`-koodin. `PRESETS["triggered"]` asettaa Line2:n valmiustilaan, kuten on suunniteltu.
>
> Kameran pakottaminen vapaakäyntiin:
>
> ```python
> import copy
> settings = copy.deepcopy(PRESETS["high_quality"])   # PRESETS entries are shared
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Jos aikakatkaisu tapahtuu edelleen komennolla `trigger_mode="Off"`, kamera ei todellakaan lähetä dataa — lähetä meille loki ja komento `ip link show`.

#### Väriprofiilit (RGB-reaaliaikainen esikatselu) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` valitsee näytön väriprofiilin RGB-kameroiden **reaaliaikaiselle esikatselulle** (monispektrikamerat eivät ota tätä asetusta huomioon):

| Profiili | Merkitys |
| --- | --- |
| `raw` | Ohita radiometrinen ketju kokonaan. |
| `linear` | DSNU + tasoitus + valkotasapaino, ei CCM:ää, ei gammaa. |
| `natural` | Lineaarinen + mitattu CCM + sRGB-gamma, vain edullisella viimeistelyllä (kromaattinen tasoitus + kirkkaiden alueiden desaturaatio) — realistinen oletusasetus. |
| `enhanced` | `natural` sekä täysi hub-parity-viimeistely (reunojen poisto, värikylläisyys, CLAHE-paikalliskontrasti). Rikkaampi ulkoasu, mutta **noin kaksinkertainen viimeistelykustannus kehystä kohti**, joten LIVE-kuvataajuus on alhaisempi. |
| `custom_temp` | `natural`, mutta valkotasapaino kiinnitetty `custom_cct_k` Kelviniin (DLS ohitetaan; rajoitettu 2000–10000 K:iin backend-puolella). |

Profiili on **vain reaaliaikaiseen esikatseluun tarkoitettu** nopeuden ja ulkoasun säätö: tallennetut kuvat saavat aina täyden ja rikkaan viimeistelyn valitusta profiilista riippumatta, joten `natural`:n valitseminen kehysajan säästämiseksi ei heikennä levylle tallennettavan materiaalin laatua. Tuntematon profiili nostaa `ValueError`-arvoa; kun chloros-backend on käytettävissä, muutos lähetetään myös sille POST-pyynnöllä, jotta seuraava esikatselukehys heijastaa sitä (suoraan SDK:tä käyttävät käyttäjät, joilla ei ole backendia, saavat silti asetusten muutoksen).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Kohokohdat (RGB-näyttö) — `set_highlights`

`LatticeCamera.set_highlights(value)` poistaa **näytön** kirkkaimmat ja valkoisimmat pikselit RGB-kameroissa — korjaus kirkkaalle valkoiselle pinnalle, joka renderöityy tasaisena valkoisena läiskänä. `value` on `-100`..`0` (rajoitettu); `0` on pois päältä, jolloin tulos on täsmälleen sama kuin ilman asetusta. Voimakkaammat asetukset laskevat näytön valkoista (noin 60 %:iin asetuksella `-100`) ja laajentavat sävykäyrää, jolloin pinta, jonka esikatselun automaattinen kirkkausasetus oli työntänyt puhtaaksi valkoiseksi, saa yksityiskohdat takaisin.

Kyseessä on näytön säätö, mutta toisin kuin kylläisyys ja kontrasti (jotka muuttavat vain reaaliaikaista esikatselua), se muuttaa myös tallennettavaa sisältöä: reaaliaikainen esikatselu ja **näytötasoiset** kuvat noudattavat tätä asetusta, joten myös nämä tiedostot tulevat tummemmiksi (niissä on asetus `Chloros:DisplayHighlights`); säteily- ja heijastavuusvientiä tämä ei koskaan vaikuta, eikä se voi palauttaa leikkaantuneita anturitietoja — pienennä valotusta sen vuoksi. Multispec- ja monokamerat eivät ota sitä huomioon. Arvo tallennetaan `settings.highlights`:ään ja lähetetään POST-pyynnöllä saavutettavaan chloros-taustapalvelimeen, aivan kuten `set_color_profile`; CLI-vastaava on `lattice highlights --set N`.

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_highlights(-60)                      # 0 = off, -100 = strongest
```

#### Mono (M3M)-kamerat ja `Calibration`

Mono **M3M**-kamera (`M3M-<lens>-F<wavelength>`) on yksikaistainen: yksi harmaasävyinen taso, ei Bayer-mosaiikkia, ei 3×3-spektrikrosstalk-matriisia. `Calibration` tunnistaa sen ja paljastaa `is_mono`-lipun. Heijastavuus pätee edelleen kaistakohtaisena radiometrisena karttana (sekoituksen purkaminen tapahtuu identiteettimatriisin avulla), mutta yksittäisen kameran monikaistamatematiikka tuottaa mielekkäitä tuloksia sen sijaan, että se palauttaisi hölynpölyä:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Voit luoda kasvillisuusindeksin monokromaattisella laitteistolla yhdistämällä useita eri aallonpituuksilla toimivia M3M-kameroita kohdistetuksi monikaistapinoksi (katso [Array Alignment](#array-alignment)) ja laskemalla indeksin kyseisen pinon perusteella yhden kameran sijaan.

#### DAQ-M-kaistojen amplitudit — `DLS.band_magnitudes`

`DLS(mac_address, filter_model="RGN-IMX265")` muodostaa yhteyden DAQ-M-laitteeseen Bluetoothin kautta omalla BLE MAC -osoitteellaan. Se on ainoa valosensoriluokka, joka kommunikoi suoraan anturin kanssa eikä taustapalvelimen kautta, joten se tarvitsee samat komponentit kuin suorat kameraluokat (`CAMERA_AVAILABLE`: Arena-SDK:n ajoympäristö ja OpenCV) sekä `bleak`:n Bluetooth-yhteyttä varten (`pip install "chloros-sdk[camera]"` tai `pip install bleak`; ilman sitä `DLS` aiheuttaa virheen `DLSError`). Jos muualla kytketty DAQ-M (taustapalvelimen poolissa, projektissa, käyttöliittymässä) lopettaa mainostamisen, joten vapauta se sieltä ensin. **`mac_address` vaaditaan**: oletusarvoa ei ole, koska jokaisella yksiköllä on oma arvonsa, ja `DLS()` ilman sitä aiheuttaa `DLSError`:n ennen kuin yhteyden muodostamista yritetään. Etsi MAC-osoite XPR:lläOTX003351:n avulla (katso [Löytö](#discovery--finding-an-address-to-connect-with)) tai `chloros-cli daq pool-discover --only ble`:n avulla.

`DLS.band_magnitudes(spectrum=None, *, spectral_response=None)` integroi DAQ-M-spektrin (viimeisin lukema, kun `spectrum` on `None`; `DLSError`, jos sellaista ei vielä ole) kameran kolmen kanavan vasteiden kautta ja palauttaa `{"red": …, "green": …, "nir": …}` — avaimet nimeävät kanavat tässä järjestyksessä; RGB-kameran käyrissä kolmas (`nir`) on sen sininen kanava.

- Ilman `spectral_response`-parametria se käyttää suodatinmallin vakiokäyrää (jota varten `filter_model` ja `DLS` on rakennettu), joka on mitattu yhdestä laitteesta.
- `spectral_response` käyttää sen sijaan kameran **omia** mitattuja käyriä: sana-muotoinen `{"wavelengths": [...], "R": [...], "G": [...], "B": [...]}`, sellaisena kuin se on tallennettu kyseisen kameran kalibrointipakettiin (`color_correction.spectral_response_smoothed` — RGB-kamerat; monispektriset paketit eivät sisällä tällaista käyrää). Jos sana-muotoista tietoa ei voida käyttää, käytetään laivueen käyrää.

```python
from chloros_sdk import DLS

# camera_curves: {"wavelengths": [...], "R": [...], "G": [...], "B": [...]}
with DLS(mac_address="AA:BB:CC:DD:EE:FF") as dls:
    reading = dls.acquire()
    fleet = dls.band_magnitudes(reading)                         # fleet curve
    own = dls.band_magnitudes(reading,
                              spectral_response=camera_curves)   # the camera's own
```

CLI:n `chloros-cli lattice dls --camera SERIAL` toimii samalla tavalla taustaprosessin kautta: se etsii kameran käyrän ladatusta tai välimuistissa olevasta kalibrointipaketista ja ilmoittaa, kenen käyrää se käytti. HTTP:n kautta tämä tapahtuu komennolla `POST /api/dls/cli-read` yhdessä komennon `mac` (pakollinen: ilman sitä vastaus on HTTP-virhe 400, `code` `MAC_REQUIRED`) ja `camera_serial` vastauksen tekstiosassa; vastauksen `band_magnitudes_source` on `camera <serial>`, kun kyseisen kameran omaa käyrää käytettiin, tai `model <filter>` laivaston käyrän tapauksessa (kamera, jolla ei ole käyrää: ei RGB:tä tai pakettia ei ole ladattu tai välimuistissa).

---

## Poikkeukset

Käsittele perusluokkaa, jotta voidaan käsitellä ”kaikki Chlorosin virheet”:

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

> `ChlorosAuthenticationError` ja `ChlorosConfigurationError` ovat `chloros_sdk`:n (ja `chloros_sdk.exceptions`:n) attribuutteja, mutta eivät kuulu `__all__`:ään, joten `from chloros_sdk import *` ei tuo niitä mukaan: tuo ne nimellä. `ChlorosError`-perhe perustuu `ChlorosLocal`:ään ja siihen rakennettuihin funktioihin (`process_folder()`, `process_lattice_capture()`); Smart-Connect-pinta perii `ChlorosConnectError`:n.

Hierarkia:

```

ChlorosError
├── ChlorosBackendError           (backend binary not found / failed to start)
├── ChlorosConnectionError        (HTTP transport failure; no backend running
│                                  with auto_start_backend=False)
├── ChlorosLicenseError           (403 PLAN_UPGRADE_REQUIRED: plan below Copper)
├── ChlorosAuthenticationError    (401 AUTH_REQUIRED: not logged in)
├── ChlorosConfigurationError     (bad configure() inputs)
└── ChlorosProcessingError        (pipeline failed; create/import refused)
    └── ChlorosProjectFilesMissingError  (open_project(): none of the
                                  project's source files found, even after
                                  the relink; .missing_roots, .audit)

ChlorosConnectError                (the smart-connect surface: connect_*, attach_array,
                                    list_*, discover_*, every CameraSession /
                                    ArraySession / DAQSensorSession method,
                                    RecorderHandle, ContinuousHandle,
                                    disconnect_all(), and analyze_array_network()
                                    when no backend answers; 401 and 403 included.
                                    Derives from plain Exception, NOT from
                                    ChlorosError, so `except ChlorosError` will
                                    not catch it. .warning: a warning the
                                    backend sent beside its error (also at the
                                    end of the message); .recording_closed: the
                                    recording a failed DAQ-E restart closed;
                                    both None otherwise)

FrameFetchError (RuntimeError)     (grab() / frame_stream() refusals; .status,
                                    .body, .errors)
ValueError                         (capture() arguments: PNG/JPEG with a level they
                                    cannot hold; a bad packed=)
FileNotFoundError / ImportError    (open_project(): no such folder / no project
                                    loaders)
RuntimeError                       (CameraHandle / ArrayHandle: a backend refusal)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## End-to-End-esimerkit

### 1. Kansion käsittely mukautetulla edistymispalkilla

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Reaaliaikainen LATTICE-matriisi → heijastavuus + DAQ-vertailuarvo

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Projektilähtöinen tallennuskampanja

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Monikamerainen kehysvirta → NumPy-putki

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Päätön suora laitteistopohjainen (ilman taustapalvelinta) tallennusskripti

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Toimintakyvyn tarkistus ennen 4-kamerajärjestelmän kytkemistä

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["width"], height=r["height"],     # before binning
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Tallennusohjeen vastine (puhdas Python)

`run_project_recipe()` suorittaa CLI-ohjeen sellaisenaan (katso [Projektikansio ja ohjeet](#project-folder--recipes)). Samat vaiheet kirjoitettuna Python-kielellä ovat:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply: connect_all() applied each camera's saved settings. CameraHandle
    # has no settings method; a CameraSession on the same pooled camera does
    # (it is borrowed, so it never releases the camera)
    chloros_sdk.connect_camera(cam.serial).set_settings(
        exposure_time=5000, gain=6.0)

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## MCP-palvelin (tekoälyavustajat)

`chloros-mcp` antaa [Model Context Protocol](https://modelcontextprotocol.io) (MCP) -protokollaa käyttävälle tekoälyavustajalle, kuten Claude Desktopille, Claude Codelle tai Cursorille, mahdollisuuden ohjata Chloros-ohjelmistoa: käsitellä kuvamateriaalia (luoda projekti, tuoda kuvakansio, valita asetukset, käsitellä kuvia ja raportoida, mitä käsittely tuotti) sekä käyttää LATTICE-kameroita, -antureita ja DAQ-valosensoreita (kytkeä, esikatsella, asettaa valotuksen ja värin, ottaa kuvia, tallentaa ja päivittää laiteohjelmiston). Sen työkalut kattavat tämän SDK:n toiminnot. Annat käskyn tavallisella kielellä, ja avustaja kutsuu palvelimen työkaluja:

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

The assistant calls:
  create_project          {"project_name": "Flight_2026-10-17"}
  import_images           {"folder_path": "D:\\Flights\\2026-10-17"}
  configure_processing    {"indices": ["NDVI"]}
  start_processing        {}
  get_processing_status   {}      (every 30-60 s until its state is "finished")

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.

The assistant calls:
  list_devices            {}
  get_camera_preview      {"array_id": "array-1779862544497"}
  start_capture_run       {"array_id": "array-1779862544497", "mode": "interval",
                           "interval_s": 10, "duration_s": 300,
                           "processing": "reflectance"}
  get_job_status          {"job_id": "..."}   (until its state is "finished")
```

Palvelin perustuu tähän SDK:iin: kuvankäsittelytyökalut kutsuvat [`ChlorosLocal`](#image-processing--chloroslocal) ja laitteistotyökalut [smart-connect-kutsut](#smart-connect-for-lattice-cameras), [DAQ-istuntoja](#daq-sensor-sessions), `ChlorosProject` ja [kameran apuohjelmia](#camera-utilities), jotka kaikki kommunikoivat saman taustapalvelimen kanssa, jota Chloros-sovellus käyttää. Sen pyynnöt sisältävät `X-Chloros-Client: sdk`:n, joten taustapalvelu soveltaa SDK:n kirjautumis- ja tilausvalintoja, ja SDK:n säännöt tulevat mukana (esimerkiksi Texture Aware hylätään LATTICE-kaappauksissa).

### Asennus

Palvelin toimitetaan `chloros-sdk`-paketissa; muuta ei tarvitse asentaa. SDK asentaa `chloros-mcp`-komennon, joka on sama kuin `python -m chloros_sdk.mcp_server`:

```bash
pip install chloros-sdk        # not needed where the Chloros installer set up the SDK (Step 2)
chloros-mcp --help
```

- **Windows:** Chloros-asennusohjelma asentaa SDK:n pip-komennolla SDK-ympäristöön, jolloin `chloros-mcp.exe` sijoitetaan kyseisen Python-ympäristön `Scripts`-kansioon.
- **Linux (.deb):** paketti asentaa SDK:n järjestelmän `python3`-kansioon ja lisää `chloros-mcp`-komennon `/usr/bin`-kansioon, joten se löytyy jokaisen käyttäjän `PATH`-kansiossa. Virtualenv-ympäristössä `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` lisää `chloros-mcp`:n virtualenv-ympäristön `bin`-kansioon.

Palvelin käyttää ainoastaan SDK:n (Python 3.7+) päälle rakennettua standardikirjastoa. MCP-asiakasohjelma käynnistää sen ja kommunikoi sen kanssa stdin/stdout-kanavien kautta; et aja sitä itse, paitsi `--help`:n avulla.

### Asiakassovelluksen asetukset

**Claude Desktop:**lisää palvelin `claude_desktop_config.json`:ään (Windowsissa `%APPDATA%\Claude\claude_desktop_config.json`; avaat sen valitsemalla**Asetukset &gt; Kehittäjä &gt; Muokkaa asetuksia**), ja käynnistä sitten Claude Desktop uudelleen:

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

**Claude-koodi:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Cursor:** sama `mcpServers`-merkintä kuin Claude Desktopissa, tiedostossa `~/.cursor/mcp.json` (kaikissa projekteissa; `%USERPROFILE%\.cursor\mcp.json` Windowsissa) tai `.cursor/mcp.json`:ssä yhdessä projektissa.

Asetukset syötetään `args`:ään tai Claude Code -komennon jälkeen (`claude mcp add chloros -- chloros-mcp --no-auto-start`). Jos haluat käyttää tiettyä Python-SDK:ta `chloros-mcp`:n sijaan `PATH`:ssa, suorita moduuli kyseisellä Python-versiolla (sen täydellinen polku muodossa `command`, jos se ei ole ensimmäinen `python3` `PATH`-projektissa):

```json
{
  "mcpServers": {
    "chloros": {"command": "python3", "args": ["-m", "chloros_sdk.mcp_server"]}
  }
}
```

Asiakasohjelma käynnistää palvelimen ilman komentotulkkia, joten jos se ilmoittaa, että palvelimen käynnistys epäonnistui, `chloros-mcp` ei todennäköisesti ole siinä `PATH`-kansiossa, jonka asiakasohjelma näkee. Anna sen sijaan täydellinen polku (`where chloros-mcp` Windowsissa, `which chloros-mcp` Linuxissa, jossa .deb-tiedostojen sijainti on `/usr/bin/chloros-mcp`) tai suorita moduuli Python-versiolla, jossa on SDK.

### Asetukset

| Asetus | Oletus | Toiminto |
| --- | --- | --- |
| `--backend-url URL` | `http://127.0.0.1:5000` | Käytettävä Chloros-taustapalvelu. |
| `--no-auto-start` | off | Älä koskaan käynnistä taustapalvelinta: käytä jo käynnissä olevaa, kuten Chloros-sovelluksen, Linux-taustapalvelinta tai tunnelia toisen koneen (katso [Etä-backend-tila](#remote-backend-mode-pip-only-host-via-tunnel); kansiopolkujen on tällöin viitattava samaan kansioon molemmilla koneilla). |
| `--backend-exe PATH` | sinne, minne Chloros asentaa sen | Käynnistettävä taustapalvelu, kun mikään ei ole käynnissä, jos Chloros on asennettu paikkaan, jota SDK-paketti ei etsi. |
| `--log-level LEVEL` | `WARNING` | `DEBUG`, `INFO`, `WARNING` tai `ERROR`. Lokit ohjataan stderr:ään, jonka useimmat asiakkaat näyttävät MCP-lokissaan; stdout välittää vain protokollaviestejä. |

### Työkalut

Jokaisen työkalun argumentit ovat niiden SDK-kutsujen argumentteja, joita työkalu käärii; asiakasohjelma luettelee jokaisen niistä täydellisen skeemansa kanssa. Polut ovat absoluuttisia.

**Käsittely**

| Työkalu | SDK-kutsu | Mitä se tekee |
| --- | --- | --- |
| `create_project` | `ChlorosLocal.create_project` | Luo projektin ja asettaa sen avoimeksi projektiksi, jota käsittelytyökalut käsittelevät. Pelkkä nimi luo kansion oletusprojektikansioon; absoluuttinen polku luo sen sinne. |
| `open_project` | `ChlorosLocal.open_project` | Avaa tallennetun projektin ja asettaa sen avoimeksi projektiksi. Siirretyn projektin polut osoitetaan uudelleen sen nykyiseen kansioon; `relink_folder` (absoluuttinen) nimeää kansion, jossa muut puuttuvat tiedostot ovat nyt. Palauttaa `source_files` ja `image_count`. Ei löytynyt: virheilmoitus, jossa luetellaan niiden sijainnit ja nimetään `relink_folder`; mitään ei ole avattuna. |
| `import_images` | `import_images` | Tuo Survey3-parit `.RAW`+`.JPG`, LATTICE `.tif`-kaappaukset ja DAQ `.daq`-tallenteet; palauttaa `count` (kuvat) ja `scan_count` (tallenteet). |
| `configure_processing` | `configure` | Muuttaa vain välitetyt asetukset, ei muita (arvot: [Tuetut arvot](#supported-values)); `custom_settings` lähettää koko konfiguraation sellaisenaan. |
| `get_project_settings` | `get_config` | Lukee avoimen projektin tallennetut käsittely-, vienti- ja hakemistoasetukset. |
| `start_processing` | `process(wait=False)` | Aloittaa avoimen projektin käsittelyn ja palaa heti. |
| `get_processing_status` | `get_status` | Ilmoittaa `state` (`idle`, `starting`, `processing` tai `finished`) ja edistymisen; kun suoritus on valmis, kirjoitetaan suorituksen `summary`, `export_check_failure`, jos vientiä ei voitu tarkistaa, ja `image_failure`, jos kuvat katosivat: muut tuotteet kirjoitetaan; ilmoita suoritus epäonnistuneeksi. |
| `process_folder` | `process_folder` / `process_lattice_capture` | Luo projektin kansiolle, tuo sen, määrittää asetukset ja käynnistää ajon yhdellä kutsulla (`lattice: true` LATTICE-kaappauksille). Kun ajo on käynnissä, se keskeytyy ennen kuin se ehtii luoda mitään, ja myös taustajärjestelmä hylkää tuolloin projektin muutoksen mistä tahansa työkalusta. |
| `export_light_sensor` | `export_light_sensor` | Kirjoittaa kalibroidut `.daq`- ja `.csv`-tiedostot projektin valosensorin ilman kuvien käsittelyä, hakee tehdaskalibroinnin MAPIR-pilvestä, jos sitä ei ole välimuistissa (katso [Valoanturin tallenteet](#light-sensor-recordings--calibrated-daq--csv)). |
| `read_image_audit_tags` | `read_image_audit_tags` | Lukee käsitellyn kuvan Chloros-tarkastusmetatiedot (lähde, kalibrointi, sovelletut vaiheet). |

**Kamerat ja matriisit**

| Työkalu | SDK-kutsu | Toiminto |
| --- | --- | --- |
| `list_devices` | `list_cameras`, `list_arrays`, `list_daq_sensors` | Mitä on tällä hetkellä kytkettynä missä tahansa asiakassovelluksessa, kukin yhdessä `connected_here`:n kanssa; sekä palvelimen käynnissä olevat työt. |
| `discover_devices` | `discover_lattice_cameras`, `discover_daq_sensors` | Kamerat ja valosensorit, jotka voitaisiin liittää. |
| `get_device_status` | `CameraSession.status`, `ArraySession.status` / `capability`, `DAQSensorSession.status` | Yhden kameran, kameraryhmän (sen suorituskyky ja kunto mukaan lukien) tai valosensorin reaaliaikainen tila. |
| `analyze_array_network` | `analyze_array_network` | Mitä tämä isäntä ja verkko pystyvät siirtämään ehdotetulle ryhmälle. |
| `connect_camera` / `connect_array` | `connect_camera` / `connect_array` | Yhdistä kamera tai synkronoitu ryhmä (päälaite ensin). |
| `connect_project_devices` | `open_project(...).connect_all` | Liitä kaikki laitteet, jotka tallennettu projekti muistaa, niiden asetuksineen. |
| `disconnect_device` | `disconnect` / `force_disconnect` | Vapauta kamera, ryhmä tai valosensori (katso Käyttäytyminen). `keep_settings: true` matriisissa tallentaa kunkin kameran asetukset omaan muistiinsa, `clear_settings: true` nollaa kyseisen muistin (sen tulos on `camera_memory`). |
| `disconnect_project_devices`, `align_project_arrays`, `capture_project_devices` | `disconnect_all`, `align_arrays`, `capture_all` | Projektin laitteet yhdessä. |
| `read_camera_settings` / `set_camera_settings` | `CameraSession.read_nodes` / `set_settings` | Lue tai kirjoita kameran asetuksia käyttämällä ystävällistä nimeä tai GenIC-solmun nimen perusteella. |
| `set_array_frame_rate` | `ArraySession.set_target_fps` | Määritä taulukon kuvausnopeus tai palauta se automaattiseen nopeuteen. |
| `capture_camera` / `capture_array` | `CameraSession.capture` / `ArraySession.capture` | Yksi kuvaus (`fastest` ja `wait_for_exposure` ovat `capture_fastest` ja vakiintuneella valotuksella otettu kuva). Jos `output_dir` puuttuu, se tallennetaan avoimen projektin`captures`-kansioon, ja tiedostot liitetään projektiin (pidätetään, kunnes käsittelykierros päättyy); vastauksen `project_add` kertoo tämän. |
| `start_capture_run` | `ArraySession.continuous`, `capture_repeated` | Jatkuva tai aikavälein tapahtuva kuvaus, työtehtävänä. |
| `start_array_recording` | `ArraySession.record` / `burst` | Yhdistettyindeksinen video tai raaka sarjakuvaus, työtehtävänä. |
| `build_burst_video` | `ArraySession.build_video` | Sarjakuvauksen uudelleenkäsittely kalibroiduksi videoksi, työtehtävänä. |
| `get_job_status` / `stop_job` | | Tehtävän seuranta tai pysäyttäminen. |
| `get_camera_preview` | | Kunkin kameran reaaliaikainen esikatselu, palautetaan kuvana. |
| `grab_frames` | `ArraySession.grab`, `CameraHandle.grab` | Yksi ruutu (tai yksi synkronoitu sarja) käsittelytasolla, tallentamatta: tasokohtaiset tilastot ja kustakin ruudusta PNG-tiedosto. |
| `calibrate_array_alignment`, `get_array_alignment`, `clear_array_alignment`, `nudge_array_alignment`, `export_array_alignment`, `import_array_alignment` | `ArrayHandle.calibrate_alignment`, `alignment_status`, `clear_alignment`, `tweak_alignment`, `export_alignment`, `import_alignment` | Taulukon kohdistus. |

**Valoanturit**

| Työkalu | SDK-kutsu | Toiminto |
| --- | --- | --- |
| `connect_light_sensor` | `connect_daq_sensor` | Liitä DAQ-U-, DAQ-M- tai DAQ-E-laite (`cap_id` asettaa sen korkkikorjauksen). |
| `get_light_sensor_reading` | `DAQSensorSession.latest` | Uusimmat lukemat: aikaleima, saturaatio, spektrialue ja huippu sekä aallonpituusluokkiin keskiarvostettu spektri (`full_spectrum` jokaiselle pisteelle). |
| `set_light_sensor_streaming` | `stream_start` / `stream_stop` | Lähetyksen aloittaminen tai lopettaminen. |
| `start_light_sensor_recording` / `stop_light_sensor_recording` | `record_start` / `record_stop` | Tallentaa `.daq`-tiedoston työtehtävänä; `duration_s` pysäyttää sen itsestään. |
| `set_light_sensor_cap` | `DAQSensorSession.set_cap` | Kannen korjaus (`none` paljaalle anturille). |
| `get_light_sensor_bindings` / `set_light_sensor_bindings` | `daq_bindings` / `set_daq_bindings` | Mikä anturi syöttää kunkin kameran ja matriisin heijastavuusarvot. |
| `get_light_sensor_band_presets` / `get_light_sensor_reflectance` | `daq_band_presets` / `light_sensor_reflectance` | Kaistajoukot sekä kahden suoratoistoanturin ympäristön ja kohteen heijastavuusarvot ja indeksit. |
| `get_light_sensor_device_info`, `set_light_sensor_network`, `set_light_sensor_auto_stream`, `enable_light_sensor_ptp`, `reboot_light_sensor`, `update_light_sensor_firmware` | `device_info`, `set_network`, `set_auto_stream`, `enable_ptp`, `reboot`, `update_firmware` | DAQ-E-laitteen omat asetukset, jotka on tallennettu laitteeseen. |

**Kameran apuohjelmat** ([Kameran apuohjelmat](#camera-utilities))

| Työkalu | SDK-kutsu | Toiminto |
| --- | --- | --- |
| `get_camera_color_settings` / `set_camera_color_settings` | `camera_color_profile`, `camera_color_adjust`, `camera_highlights` ja niiden asetustoiminnot | RGB-kameran näytön värit: profiili ja värilämpötila, kylläisyys / kontrasti, kirkkaat kohdat. |
| `run_camera_white_balance` / `set_camera_white_balance` | `camera_white_balance_once` / `set_camera_white_balance` | Kertavalkaisun valkotasapaino tai automaattinen / pois päältä / kiinteät suhteet. |
| `list_camera_presets` / `apply_camera_preset` | `camera_presets` / `apply_camera_preset` | Kameran esiasetukset. |
| `list_filter_models` | `camera_filter_models`, `camera_filter_model_info` | Suodatinmallit tai yksi nimellä. |
| `get_camera_firmware_status` / `update_camera_firmware` | `camera_firmware_status` / `update_camera_firmware` | Laiteohjelmistoversiot ja salama (oletuksena yksi tehtävä). |
| `get_camera_power_info` / `set_camera_power` | `camera_power_probe`, `sleep_camera`, `wake_camera` | Virta- ja lämpötilatila; lepotila tai herätystila. |
| `calibrate_camera_with_target` | `calibrate_camera_with_target` | Näkymässä olevan kalibrointikohteen kertoimet. |
| `get_network_adapters` / `fix_network_settings` | `network_adapters` / `fix_network_settings` | Kameran verkkosovittimet, arvio ja korjaus (vain järjestelmänvalvojalle). |
| `apply_vignette_correction` | `apply_vignette_correction` | Korjaa kuvakansion vinjetointi. |
| `read_dls` | `read_dls` | Yksi DLS-lukema. |
| `probe_system` / `get_system_info` | `probe_system` / `system_info` | Isäntäkoneen laitteisto ja suoritusaika. |

**Transient-kamerat, kohdistusprofiilit ja indeksit** ([Transient-taulukot, kohdistusprofiilit ja indeksit](#transient-arrays-alignment-profiles--indices)). Jokainen tässä luettelossa oleva työkalu, joka avaa kameroita, käyttää `keep_settings` / `clear_settings` (oletusarvo false): katso [Kameran muisti](#camera-memory--keep_settings--clear_settings).

| Työkalu | SDK-kutsu | Toiminto |
| --- | --- | --- |
| `start_camera_continuous` / `start_multi_camera_stream` | `start_camera_continuous` / `start_multi_camera_stream` | Tallentaa yhden kameran tai suoratoistaa useita kameroita synkronoidusti tehtävänä (`duration_s` lopettaa sen itsestään). |
| `capture_cameras_transient` | `capture_cameras_transient` | Yksi synkronoitu tallennus ilman kameraryhmän liittämistä. |
| `test_array_sync` / `detect_array_master` | `test_array_sync` / `detect_array_master` | Laukaisusynkronoinnin tarkistus ja mitkä kamerat ovat synkronointilinjalla. |
| `analyze_array_models` | `analyze_array_models` | Mitä tällaisista malleista koostuva ryhmä saisi aikaan tässä verkossa. |
| `calibrate_alignment_profile` | `calibrate_alignment_profile` | Ratkaise ja tallenna kohdistusprofiilitiedosto (`options` sisältää tarkemmat ratkaisuvaihtoehdot). |
| `capture_aligned_multiband` / `start_aligned_stream` | `capture_aligned_multiband` / `start_aligned_stream` | Kohdistettu monikaistakuva tai niiden sarja työnä. |
| `read_alignment_profile` / `reorder_alignment_profile` | `alignment_profile_info` / `reorder_alignment_profile` | Profiilitiedoston sisältö; sen kaistajärjestys ja käytössä olevat kamerat. |
| `list_index_presets`, `compute_index`, `capture_live_index` | `index_catalog`, `compute_index`, `capture_live_index` | Indeksi-esiasetukset ja gradientit; monikaistaisen TIFF-tiedoston indeksi; tallennus indeksiin yhdellä kertaa. |

**Aikasynkronointi, projektikansio ja reseptit** ([PTP Grandmaster](#ptp-grandmaster--time_sync_status), [Projektikansio ja reseptit](#project-folder--recipes))

| Työkalu | SDK-kutsu | Toiminto |
| --- | --- | --- |
| `get_time_sync_status` | `time_sync_status`, `time_sync_peers`, `time_sync_cameras` | PTP-päälaitteen tila, sen vertaislaitteet ja kunkin kameraryhmän PTP-tila. |
| `restart_time_sync` / `set_time_sync_priority` | `restart_time_sync` / `set_time_sync_priority` | Käynnistä se uudelleen; aseta sen prioriteetit. |
| `get_project_folder` / `set_project_folder` / `reset_project_folder` | `project_folder` / `set_project_folder` / `reset_project_folder` | Kansio, johon uudet projektit luodaan. |
| `run_project_recipe` | `run_project_recipe` | Suorita tallennetun projektin resepti työtehtävänä sen lokin kanssa. Tarkistetaan ensin: virheellinen resepti hylätään eikä mitään suoriteta; irrotetuista antureista / anturiryhmistä annetaan varoitus (niitä ei kytketä: suoritetaan ensin `connect_project_devices`). `dry_run` lopettaa tarkistuksen eikä suorita mitään. |

Tallennettua projektia ei voi avata käsittelyä varten: jokainen käsittelytehtävä alkaa komennolla `create_project` (tai `process_folder`).

### Vaatimukset

- **Chloros on asennettu samalle tietokoneelle.** Chloros-taustapalvelu suorittaa käsittelyn. Palvelin voi sen sijaan käyttää taustapalvelua, johon pääsee tunnelin kautta (`--backend-url` yhdessä `--no-auto-start`:n kanssa, katso [Asetukset](#options)).
- **Kirjautunut sisään kyseisellä tietokoneella**, joko Chloros-sovelluksessa tai `chloros-cli login`:n avulla. Palvelin ei voi kirjautua sisään puolestasi: uloskirjautuneena käytetty työkalu vastaa ”Ei kirjautunut sisään Chlorosiin” ja ilmoittaa taustapalvelimen syyn, minkä avustaja kertoo sinulle.
- **Mikä tahansa maksullinen Chloros+-taso (Copper tai korkeampi).** Palvelimella on SDK-käyttöoikeus, ja ilmaisella Iron-tasolla sitä ei ole: ilmaisella tasolla kutsuttu työkalu vastaa ”Ei saatavilla tällä Chloros-sopimuksella” ja ilmoittaa taustapalvelimen syyn.
- Python 3.7 tai uudempi SDK:n kanssa sekä MCP-asiakasohjelma, joka käynnistää paikallisia (stdio) palvelimia.

### Toiminta

- **Taustapalvelu.** Palvelimen käynnistäminen tai sen työkalujen luettelointi ei käynnistä mitään; ensimmäinen työkalukutsu muodostaa yhteyden. Jos Chloros-sovellus (tai Linux-taustapalvelu) on käynnissä, palvelin käyttää sen taustapalvelua ja jättää sen käyntiin. Jos taustapalvelua ei ole käynnissä, palvelin käynnistää asennetun taustapalvelun ilman ikkunaa. Kun palvelin sulkeutuu (kun suljet asiakassovelluksen tai se käynnistää palvelimen uudelleen), se pysäyttää aloittamansa tallennustyöt ja valosensorin tallenteet sekä vapauttaa liitetyt laitteet; taustapalvelu jatkaa toimintaansa, joten keskeneräinen käsittelykierros jatkuu loppuun asti, ja se pysähtyy itsestään 30 minuutin kuluttua, jos sitä ei käytetä. `--no-auto-start`:n kanssa palvelin ei koskaan käynnistä taustapalvelua: jos työkalu kutsutaan ilman käynnissä olevaa taustapalvelua, sinua pyydetään avaamaan Chloros-sovellus, ja seuraava kutsu sen jälkeen muodostaa yhteyden.
- **Yksi avoin projekti.** Taustapalvelimella on yksi avoin projekti, joka jaetaan Chloros-sovelluksen kanssa, kun sovellus on auki, joten älä työskentele sovelluksessa toisen projektin parissa, kun avustaja on toiminnassa.
- **Vain välitetyt asetukset muuttuvat.** `configure_processing` jättää kaikki asetukset, joita sille ei ole annettu, tallennettuihin arvoihinsa (SDK:n `configure()` palauttaa debayerin, vinjetin korjauksen, heijastavuuden kalibroinnin, PPK:n ja vientimuodon oletusarvoihinsa, kun niitä ei ole annettu; palvelin täyttää ne projektista). Jos se ei pysty lukemaan projektin tallennettuja asetuksia, se ei muuta mitään ja ilmoittaa siitä, ellei kaikkia viittä ole annettu. `indices` korvaa projektin indeksilistan. Tuloksessa luetellaan se, mitä lähetettiin `settings_sent`:nä, ja nimeää `warnings`:n alle kaikki indeksit, joita kameran kaistat eivät pysty muodostamaan.
- **Käsittely tapahtuu start-then-poll-periaatteella.** `start_processing` palaa heti, kun suoritus alkaa, minkä jälkeen se kestää minuutteja tai tunteja, ja avustaja jatkaa sen jälkeen komennolla `get_processing_status`. `state` on joko `idle` (ei käsittelyä), `starting` (taustapalvelin ei ole vielä ilmoittanut ajoa, enintään 20 sekuntia), `processing` tai `finished`, ja mukana on ajon [yhteenveto](#post-run-summary--hints): mitä se on vienyt, sekä vihjeitä, jos vietyä materiaalia on odotettua vähemmän (ei kuvia, kalibrointikohdetta ei löydy). Tuotteet tallennetaan projektikansioon, kuten sovelluksessakin. `start_processing` hylätään, kun ajo on käynnissä, ja taustapalvelu, joka lakkaa vastaamasta, raportoidaan virheenä, ei valmiina ajona.
- **Mitä avustaja näkee.** SDK:n varoitukset näkyvät työkalun tuloksissa kohdassa `warnings`, ja epäonnistumiset (kirjautuminen ulos, suunnitelma, virheellinen polku) näkyvät virheilmoituksina, jotka avustaja voi lukea ja välittää sinulle. Yli 20 kohdan luettelo lyhennetään 20 ensimmäiseen kohtaan, ja sen koko pituus näkyy koodina `<key>_total`. Polkujen on oltava absoluuttisia.
- **Laitteet ovat jaettuja.** Kamerat, matriisit ja valosensorit sijaitsevat taustapalvelimella, joten avustaja voi ohjata yhtä niistä Chloros-sovelluksen kautta, ja muutokset näkyvät sovelluksessa. `disconnect_device` ilmoittaa, mihin palvelimeen laite on kytketty; laite, johon toinen asiakas on kytketty (sovellus, `chloros-cli`, skripti tai toinen `chloros-mcp`) vapautetaan vain `force: true`:n avulla, joka on SDK:n `force_disconnect`, ja hylkäys ilmoittaa, kuka laitteen on kytkenyt, kun taustapalvelu tietää sen; laite, jota taustapalvelimella ei ole lainkaan hallussaan, raportoidaan kytkemättömäksi (`released: false`, `connected: false`, ei mitään vapautettavaa). Backendin suorittama tallennus tai suoratoisto johtaa sen `get_job_status`-näkymään, jossa on `counts` — `frames_written`, `frames_dropped`, `frames_failed`, `drop_reason` (monikamerastriimin kehysryhmät) — sekä yhden rivin pituinen `summary` (&quot;220 kehystä kirjoitettu, 55 hylätty, 0 epäonnistunut: ...&quot;), kun se on käynnissä, ja lopuksi sen lopulliset luvut; toisen asiakkaan luettelo siitä (`other_clients_jobs`) sisältää samat tiedot (`list_cameras()`, `list_arrays()` ja `list_daq_sensors()` sisältävät kunkin laitteen`connected_by`: `client`, `label`, `name`, `at`). Jokainen `chloros-mcp` ilmoittaa itsensä taustapalvelimelle nimellä `chloros-mcp (pid N)` (`chloros_sdk.connect.CLIENT_LABEL`, lähetetään nimellä `X-Chloros-Client-Label`; mikä tahansa SDK-skripti voi asettaa sen). Toisen asiakkaan valosensorin tallennuksen pysäyttäminen vaatii myös `force`:n.
- **Pitkä työ on tehtävä.** `start_capture_run`, `start_array_recording`, `build_burst_video`, `start_light_sensor_recording`, `start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`, `run_project_recipe`, ja mikä tahansa pyydetty kaappaus, ratkaisu tai indeksi `background`, palauttaa välittömästi `job_id`; `get_job_status` seuraa sitä ja `stop_job` lopettaa sen. Suoritus jatkuu, kunnes se pysäytetään, ellet anna sille `max_captures`-, `duration_s`-komentoa (tai sarjan osalta `max_frames`-komentoa); `min_free_gb` kieltäytyy käynnistymästä, jos levyllä on liian vähän vapaata tilaa. Rajoitusta ei sovelleta, ellet aseta sellaista. Tallennus tai suoratoisto, jota taustapalvelu suorittaa (`start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`) säilyttää taustapalvelimen työtunnuksen, joten toinen `chloros-mcp` — myöhemmin käynnistetty — löytää sen kyseisen tunnuksen avulla: `get_job_status` seuraa sitä (ja listaa tällaiset työt kuten `other_clients_jobs`, alkaen `GET /api/camera/cli-jobs`, joista jokaisella on `started_by`), ja `stop_job` lopettaa sen vasta `force: true`:n avulla. Mikä tahansa muu työ (kaappausajo, taulukon tallennus tai sarjatallennus, taustakaappaus) suoritetaan sitä käynnistäneessä `chloros-mcp`-palvelimessa, ja vain kyseinen palvelin tunnistaa sen: toinen palvelin, joka on pyytänyt sitä, ilmoittaa asiasta. Pysäytetty tehtävä lukee `stopping`:ää kirjoittaessaan jononsa tyhjäksi, sitten `finished`:ää lopullisine lukemineen, 10 minuutin ajan ja mistä tahansa `chloros-mcp`:stä: aloittanut palvelin havaitsee toisen palvelimen pakotetun pysäytyksen samalla tavalla. `get_job_status` ei koskaan odota pysäytystä: pysäytyksen aikana (oman `stop_job`:n, toisen asiakkaantai pysäytyksen, joka vapauttaa itsestään päättyneen työn kamerat) se vastaa välittömästi `stopping`:llä ja reaaliaikaisilla laskureilla, ja jo pysähtymässä olevan työn `stop_job` ilmoittaa siitä. Päättynyt työ kertoo syyn `stop_reason`: omat syynsä (`no_frames`, `disk_full`, `member_missing`, tai jatkuvan ajon tapauksessa `count`, `duration`, `array_gone`), `limit reached` vain silloin, kun sen lukemat osoittavat sen `count` (tai sarjan `max_frames`, tallennuksen`duration_s`) saavutettiin, `error`, `stopped` (arvolla `stopped_by`), kun asiakas lopetti sen, muuten `finished`.
- **Mihin tallenteet tallennetaan.** Avoimen projektin `captures`-kansioon, johon sovellus tallentaa ne, ellei `output_dir`-arvoa ole annettu.
- **Kuvat.** `get_camera_preview` ja `grab_frames` palauttavat kuvia, joita avustaja voi tarkastella, niiden JSON-tietojen lisäksi. `grab_frames`-kuvien kontrastia on laajennettu katselua varten; niiden tilastotiedot ovat todellisia arvoja.
- **Kun palvelin sulkeutuu**, sen käynnistämät tehtävät pysähtyvät ja sen aloittama valosensorin tallennus lopetetaan. Jos palvelin on käynnistänyt taustapalvelun, se vapauttaa myös liittämänsä laitteet ennen taustapalvelun pysäyttämistä; sovelluksen taustapalvelussa ne pysyvät yhteydessä.
- **Protokolla.** Vain stdio (ei HTTP-päätettä), vain työkalut (ei resursseja tai kehotteita), teksti- ja kuvatulokset, MCP-protokollan versiot 2024-11-05 – 2025-11-25.**Mitä ei sisälly.** Sisään- tai uloskirjautuminen (tili on sinun; palvelin ei voi kirjautua sisään) sekä sellaisen taustapalvelun pysäyttäminen, jota palvelin ei ole käynnistänyt. Kaikkeen muuhun, mitä tämä SDK tekee käsittelyn ja laitteiston suhteen, on olemassa työkalu. (DAQ-E:n käyttöönotto – sen nimi, todennustunnus, OTA-salasana, allekirjoitusvaatimus, tehdasasetusten palautus – sekä suorat DAQ-palvelimet kuuluvat yksinomaan `chloros-cli`:lle, myös SDK:ssa: väärä nimi tai tunnus lukitsee jokaisen Chloros-asennuksen laitteen ulkopuolelle.)

---

## Taustapalvelun automaattinen käynnistys

Smart-Connect-liittymäpisteet — `connect_camera`, `connect_array`, `connect_daq_sensor`, `discover_lattice_cameras` ja `discover_daq_sensors` — ovat kevyitä HTTP-asiakasohjelmia, jotka olettavat, että taustapalvelu kuuntelee osoitteessa `127.0.0.1:5000` (smart-connect-rajapinnan oletus-URL). Kun graafinen käyttöliittymä tai CLI on jo käynnissä, yksi niistä on jo käynnissä. Pelkästä skriptistä käsin sitä ei välttämättä ole — joten nämä toiminnot **käynnistävät automaattisesti mukana toimitetun taustapalvelimen** (ikkunattomana, samalla tavalla kuin `ChlorosLocal`) ennen ensimmäistä kutsua ja odottavat sitten enintään 60 sekuntia, kunnes se käynnistyy. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` ja `analyze_array_network()` eivät koskaan käynnistä sitä.

Säännöt:

- **Ainoastaan paikallinen URL-osoite luodaan.** `backend_url`, joka osoittaa `localhost`:ään / `127.0.0.1` / `[::1]` on kelvollinen; minkä tahansa muun isännän oletetaan olevan jonkun toisen kone ja sitä ei koskaan käynnistetä.
- **Backend, joka lopettaa toimintansa ennen vastaamista, lopettaa odotuksen.** Kutsu palauttaa välittömästi poistumiskoodin, suoritetun komennon ja backendin tulostamat viimeiset rivit sen sijaan, että odottaisi 60 sekuntia. Poistumiskoodi 98 on poikkeus: toinen taustaprosessi hallitsee jo porttia tai lukitustiedostoa (Chloros-sovelluksen, taustaprosessipalvelun tai jonkin toisen, vielä käynnistymässä olevan skriptin), joten kutsu jatkaa odottamista, kunnes kyseinen taustaprosessi vastaa. `ChlorosLocal` toimii samalla tavalla.
- **Taustaprosessi jätetään käynnissä uudelleenkäyttöä varten** (kuten CLI:ssä) — skriptin päättyessä ei tapahdu implisiittistä sammutusta. Skriptin uudelleenkäynnistys käyttää uudelleen käynnissä olevaa taustapalvelinta.
- **Voit kieltäytyä tästä käyttämällä `auto_start_backend=False`** missä tahansa näistä kutsuista (esim. kun olet määrittänyt etäisen taustapalvelimen tai hallitset taustapalvelimen elinkaarta itse).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Jos mukana toimitettua binääriä ei löydy tai sitä ei voida käynnistää, seuraava HTTP-kutsu aiheuttaa toimenpiteitä vaativan, **alustakohtaisen** `ChlorosConnectError`-virheen pelkkän yhteyden hylkäysvirheen sijaan — Windowsissa se ohjaa sinut työpöytäsovellukseen tai `chloros-cli`-komentoon; Linuxissa (ei graafista käyttöliittymää) se ohjaa sinut `chloros-cli`-komentoon tai `.deb`-komentoon.

---

## Ympäristö ja otsikot

SDK merkitsee jokaisen taustapalvelimen HTTP-kutsun tunnisteella `X-Chloros-Client: sdk`. Taustapalvelin soveltaa SDK/CLI-lisenssisääntöjä (vaaditaan kirjautuminen **ja** maksullinen Chloros+-paketti) eikä graafisen käyttöliittymän ilmaiskäyttötason sääntöjä. Tämä asetetaan automaattisesti tuonnin yhteydessä — sinun ei tarvitse tehdä mitään.

Otsikko lisätään jokaiseen `requests`- tai `urllib`-kutsuun, jonka URL-osoite sisältää `localhost` tai `127.0.0.1`, millä tahansa portilla – mukaan lukien omat paikalliset palvelusi — ja se korvaa itse asettamasi arvon. Muille isännille osoitetut kutsut jäävät ennalleen.

Ohita taustapalvelimen URL-osoite välittämällä `backend_url=` (`ChlorosLocal` hyväksyy vain `backend_url=` tai `api_url=`; `process_folder()` ja `process_lattice_capture()` hyväksyvät vain `api_url=`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Asennettu taustapalvelin kuuntelee vain loopback-liitäntää, joten LAN-verkossa oleva `backend_url` ei pääse siihen käsiksi; katso [Etätaustapalvelintila](#remote-backend-mode-pip-only-host-via-tunnel) tunnelimallin osalta.)

---

## Versiointi ja yhteensopivuus

- SDK-versio näkyy nimellä `chloros_sdk.__version__`.
- SDK sitoo toiminnot mukana toimitettuun taustapalvelinversioon. Vanhemman SDK:n ja uudemman taustapalvelimen yhdistäminen toimii yleensä (eteenpäin yhteensopivat päätepisteet), mutta uudemman SDK:n ja vanhemman taustapalvelimen yhdistäminen voi aiheuttaa `404`-virheitä uusissa päätepisteissä — päivitä työpöytäsovellus vastaavaksi.
- Smart-connect-rajapinta (`connect_camera` / `connect_array` / `connect_daq_sensor`) ja verkkoanalyysin päätepiste palauttavat vakaat JSON-skeemat; uudet kentät ovat lisäyksiä.

---

## Vianmääritysvinkkejä

- **`ChlorosAuthenticationError: Login required`** (lähde: `ChlorosLocal`; Smart-Connect-kutsut aiheuttavat virheen `ChlorosConnectError: Login required. …` samasta syystä) → Suorita `chloros-cli login EMAIL PASSWORD` kerran tällä koneella tai kirjaudu sisään Chloros-työpöytäsovelluksen kautta.
- **`ChlorosProcessingError: Failed to start processing: … The project folder is too deep for this computer …`** (tai `Failed to create project: …`, joka johtuu virheestä `create_project()`) → Windowsin pitkät polut on poistettu käytöstä tällä koneella (Windowsin oletusasetus), joten mikään polku ei saa ylittää 260 merkkiä, ja tuotteet sijaitsevat noin 130 merkkiä projektikansion alapuolella. Mitään ei käsitelty. Viesti ilmoittaa pisimmän sopivan projektikansion: käytä lyhyempää tai ota Windowsin pitkät polut käyttöön ja käynnistä Chloros uudelleen. Windowsin asennusohjelma tarjoaa tämän vaihtoehdon nimellä ”Ota Windowsin pitkät polut käyttöön”, joka on valittuna oletuksena; muussa tapauksessa käytä ryhmäkäytäntöä: *Tietokoneen kokoonpano &gt; Hallintamallit &gt; Järjestelmä &gt; Tiedostojärjestelmä &gt; Ota Win32-pitkät polut käyttöön*.
- **`ChlorosConnectError: No Chloros backend is running …`** → Smart-connect-toiminto käynnistää paikallisen taustapalvelimen automaattisesti, joten tämä virhe ilmenee vain, kun mukana toimitettua binääritiedostoa ei löydy tai sitä ei voida käynnistää (esim. pelkkä pip-vain-isäntäkone, jossa ei ole työpöytäpakettia). Viesti on alustakohtainen: Windowsissa avaa työpöytäsovellus tai suorita mikä tahansa `chloros-cli`-komento; Linuxissa suorita `chloros-cli`-komento (GUI:ta ei ole) tai asenna `.deb`. Etätaustapalvelimelle välitä `backend_url=` (ja `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** tuonnin yhteydessä → `lattice_sdk`:n lataaminen epäonnistui: Arena SDK:n ajoympäristöä tai OpenCV:tä (`cv2`), ei ole tuotavissa tässä Python-versiossa. Taustaprosessilaitteistovetoinen pinta (`connect_*`, projisoidaan taustaprosessilaitteiston kautta, `ChlorosLocal`) toimii edelleen.
- **Array kytkeytyy täysikokoisella anturilla, mutta alhaisella kuvataajuudella**→ Suunnittelun mukainen toiminta: automaattitila säilyttää täysikokoisen anturin ja antaa taajuuden laskea siirtoyhteyden sallimaan tasoon. Ohita `binning=2` / `4` tai pienemmän `width` / `height`, jos haluat enemmän kuvia sekunnissa, tai ota käyttöön jumbo-kehykset / päivitä yhteys. Binning ja ROI**eivät** kata kokonaisylimerkintää (`oversubscribed: true`, fps-kentät 0): liian monta kameraa kaapelille ei voida korjata binningillä/ROI:lla — vähennä kameroiden määrää, ota käyttöön jumbo-kehykset tai siirry nopeampaan verkkokorttiin (katso [Ylimerkintä](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` ilmoittaa verkkokortin vastaanottorengasta pieneksi (~0,26 MB) / yhdistä portit, joissa näkyy &quot;FRAMES WILL DROP&quot;** → Isäntäkoneen verkkokortin vastaanottorengas on oletusasetuksessaan (nollautuu usein arvoon 32 verkkokortin ohjaimen päivityksen jälkeen). Realtek USB 10GbE -sovittimella aseta `ReceiveBufferLen=256` ja `PendingReceives=64` (korotetut), ja käynnistä sitten taustapalvelu uudelleen, jotta se lukee renkaan uudelleen. Koko menettely: [CLI-ohje → Isäntä-NIC:n asetukset ja säätö](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Isäntäkone jumittuu uudelleenkäynnistyksen tai sammutuksen yhteydessä, myöhemmin WMI-virheet `Invalid class` / NIC ei aktivoidu** → Vanhentunut USB 10GbE -ajuri aiheuttaa virheen `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Päivitä sovittimen ajuri uusimpaan versioon (≥ 2026) ja määritä vastaanottorenkaan asetukset uudelleen. Katso [CLI-viite → Isäntäkoneen verkkokortin asetukset ja säätö](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Array-kaappaus epäonnistuu HTTP-virheen 503 ”No fresh synced frame group” vuoksi** → Array ei julkaissut uutta kehysryhmää odotusaikana. Tarkista `arr.status()`: `stalled: True` (tai `streaming: False`) yhdessä `actual_fps`:n kanssa, jossa 0 tarkoittaa, että tallennussilmukka on lopettanut julkaisemisen — katkaise yhteys ja muodosta se uudelleen. Jos `stalled` on False ja `actual_fps` on yli 0, yritä uudelleen. Heti `connect_array()`:n jälkeen tallennus on jo odottanut ensimmäistä ryhmää jopa 20 sekuntia.
- **`status()` näyttää `link_speed_mbps: 4294`** → Windows ei pystynyt lukemaan sovittimen nopeutta; tämä on tyypillistä USB-Ethernet-sovittimille eikä se ole vika. Katso [Odotetut kehysnopeudet](#expected-frame-rates).
- **`update_firmware()` epäonnistuu `device never connected back on TCP <port>`:n kanssa** → Palomuuri esti DAQ-E:n yhteyden takaisin tähän tietokoneeseen. Windows-asennusohjelma lisää säännön (katso [DAQ-E-laitteen asetukset](#daq-e-device-settings)); jos käytät muuta palomuuria, salli saapuva TCP-liikenne taustapalvelimelle anturin aliverkosta.
- **Heijastavuus hylätty** → Absoluuttisen asteikon heijastavuuden mittaamiseksi kameran (tai anturiryhmän) on oltava sidottu aktiiviseen DAQ-laitteeseen. Tee sidonta joko käyttöliittymän kautta tai käytä `processing="radiance"`-komentoa (W/m²/sr/nm), joka ei vaadi pariksi liitettyä anturia.
- **`smart=True`-tallennus kestää odotettua kauemmin** → AE-konvergenssi riippuu kohteen dynamiikasta. Käytä `ArrayHandle.capture_smart()`-asetusta, laajenna `exposure_tolerance_pct`-asetusta, lyhennä `stability_window_s`-arvoa tai pienennä `settle_timeout_s`-arvoa, jotta laukaisu tapahtuu nopeammin ja on vähemmän vakiintunut.

---

## Katso myös

- [CLI-ohje](cli-reference.md) — jokainen CLI-alikomento vastaa yhtä SDK-kutsua.
- [DAQ-valosensorit](https://mapir.gitbook.io/chloros/daq-light-sensors) Chloros-käsikirjassa — sensorit, niiden ylärajat ja kalibroitu mittausalue, tallennus, DAQ-E-verkko.
- [LATTICE-kamerat](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) Chloros-käyttöoppaassa — liittäminen, kameran asetukset, tallennustilat, kameraryhmät.
- Verkkodokumentaatio: `https://mapir.gitbook.io/chloros/api-python-sdk`
