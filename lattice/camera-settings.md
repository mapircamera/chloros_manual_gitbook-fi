# Kameran asetukset

**Kamerat**-välilehti on Chloros

in LATTICE-kameroiden reaaliaikainen ohjauspaneeli: pääkuva-alue, jossa jokainen kytketty kamera näkyy reaaliaikaisena ruuduna, sekä sivupalkki, jota liu’uttamalla voi vaihtaa kolmen sivun välillä — **kameraluettelo**,**asetusruutu**(kamerakohtaiset, ryhmä- tai tallennusasetukset – yksi kerrallaan) sekä**indeksilaskuri**. Tällä sivulla kuvataan kaikki kameraluettelon, kamerakohtaisen asetuspaneelin ja ryhmäasetuspaneelin säätimet. Tallennustilat, vientityypin valinta ja ”Capture All” -toiminto löytyvät oheissivulta [Tallennusasetukset ja -tilat](capture.md).

”Kamerat”-välilehti näkyy sivupalkissa, kun Chloros

-taustapalvelu on valmis. Kaikki alla olevat säätimet kommunikoivat paikallisen taustapalvelun kanssa tunnuksen `127.0.0.1:5000`:n kautta; muutokset näkyvät kameran reaaliaikaisessa kuvassa välittömästi, ellei toisin mainita.

## Tällä sivulla käytetyt kameratyypit

Säätimet näkyvät tai piiloutuvat sen mukaan, minkä tyyppinen kamera on valittuna. Käyttöoppaassa käytetään seuraavia termejä:

| Termi | Merkitys | Suodatinkanavat |
| --- | --- | --- |
| **RGB

-kamera** | LATTICE M3C, jossa on FRGB-suodatin (mallissa on `-FRGB`) | Red

/ Green

/ Blue

|
| **Bayer-monispektrinen** | LATTICE M3C, jossa on FRGN-, FOCN- tai FNGB-suodatin | FRGN: Red

/ Green

/ NIR

· FOCN: Orange

/ Cyan

/ NIR

· FNGB: NIR

/ Green

/ Blue

|
| **Mono (M3M)** | LATTICE M3M — yksi kapeakaistainen suodatin, yksi kalibroitu kaista | Yksi kaista |
| **Ryhmän jäsen** | Kamera, joka on kytketty osaksi synkronoitua ryhmää (yhdistetty tai erillinen näyttö) | Suodattimensa mukaan |

 RGB

-kameroille suoritetaan fotometrinen käsittely (valkotasapaino, väriprofiilit, gamma); monispektrisille ja mono-kameroille suoritetaan radiometrinen käsittelyketju, ja fotometriset säädöt ohitetaan. Ryhmän jäsenet välittävät stream-tason asetukset (pikselimuoto, resoluutio, binning, laukaisija, kuvataajuus) ryhmälle — nämä rivit muuttuvat kamerakohtaisessa paneelissa vain luku -tilaan ja siirtyvät sen sijaan ryhmän asetuspaneeliin.

## Pääsyötealue

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->



Jos kameroita ei ole kytketty, syötealueella näkyy **&quot;Kytke kamera aloittaaksesi&quot;**-aloitusnäyttö, jossa on kaksi painiketta:**Kytke kamera**(vihreä, avaa yhden kameran liitäntävalintaikkunan) ja**Liitä kameraryhmä** (sininen, avaa kameraryhmän liitäntävalintaikkunan). Itse liitäntävalintaikkunoista on tietoa kohdassa [Kameroiden liittäminen](connecting.md); kameraryhmän käsitteistä (synkronointi, tasot, kaistanleveys) kohdassa [Monikamerajärjestelmissä](arrays.md). Kun avaat tallennetun projektin, jossa on kameroita, aloitusnäytössä näkyy sen sijaan pyörivä kuvake ja teksti ”Avataan uudelleen N tallennettua kameraa…”, kun Chloros

palauttaa viimeisen istunnon videovirrat.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Yläpalkki

| Ohjaus | Toiminto |
| --- | --- |
| **Näkymätilan vaihto**| Vaihtaa**ruudukkonäkymän**(kaikki ruudut soluina) ja**luettelonäkymän**(täysleveät rivit: jokainen yhdistetty ryhmä yhtenä yhdistettynä rivinä, jokainen**erillisen** ryhmän jäsen omalla rivillään omalla syötteellään ja kalibrointikohdetaulukollaan, ja sen alapuolella YKSI erillinen kamera — aktiivinen kamera) välillä. Vihjeet: ”Vaihda ruudukkonäkymään” / ”Vaihda luettelonäkymään”. |
| **Ruudukon lukitus**(lukko) | Oletusarvoisesti**lukittu** — ruudut on jäädytetty paikoilleen. Avaa lukitus, jotta voit vetää ja järjestellä ruutuja mihin tahansa paikkaan (aukot säilyvät). Ruudukko lukittuu automaattisesti uudelleen aina, kun uusi kamera muodostaa yhteyden. Työkaluvinkit: ”Avaa ruudukon lukitus (ota ruutujen vetäminen käyttöön)” / ”Lukitse ruudukko (jäädyytä ruudut paikoilleen)”. |
| **Syötteen zoomaus** -liukusäädin | Ruudun koko, 60 pikselistä koko säiliön leveyteen asti. Solut säilyttävät 4:3-kuvasuhteen. Kun ruudun leveys on alle 200 px, nimi ja fps-päällekkäisnäytöt piilotetaan, jotta ruutu pysyy siistinä. |

### Syötteen ruudut

Jokainen kamera renderöi yhdistelmäruudun; kamera voi lisäksi näyttää kolme harmaasävyistä **kanavakohtaisesti jaettua** ruutua (katso [Kanavajaot](#display-overlays-drawn-over-the-live-feed)), ja kameraryhmät tuottavat yhdistelmäruudun. Aktiivisessa ruudussa näkyy valintarengas kameran (tai kameraryhmän) värillä.

Kun viet hiiren ruudun päälle, näkyviin tulee **X**-sulkupainike:

* **Yhdistelmäruudun** sulkeminen, kun sen kanavakohtaiset ruudut ovat edelleen näkyvissä, piilottaa vain yhdistelmäruudun.
* **Yksittäisen kameran viimeisen näkyvän ruudun** sulkeminen katkaisee kyseisen kameran yhteyden.
* **Yhdistelmäruudukon jäsenten jaetut ruudut eivät koskaan katkaise** kameran yhteyttä — ne vain piilottavat ruudun.

Kun ruudukko on lukitsematon, vedä mikä tahansa ruutu mihin tahansa paikkaan; asettelu tallennetaan projektin mukana.

## Sivupalkki — kameraluettelo

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

Ensimmäisellä sivupalkkisivulla on luettelo kaikista kytketyistä kameroista ja ryhmistä:

* **Yhdistä kamera**(vihreä) /**Yhdistä ryhmä** (sininen, näyttää tekstin ”Tunnistetaan...” skannauksen aikana). Molemmat ovat pois käytöstä, kun yhdistämisvalintaikkuna on auki.
* **Tallenna kaikki**(punainen) — tallentaa kaikki luettelossa olevat kamerat**Tallennusasetuksissa** valituilla vientityypeillä. Edellyttää avointa projektia. Kattavat ohjeet löytyvät kohdasta [Tallennusasetukset ja -tilat](capture.md).
* **Tallennusasetusten ratas** (”Tallenna kaikki” -painikkeen vieressä) — avaa [Tallennusasetukset-paneelin](capture.md#the-capture-settings-pane). Ei käytettävissä ilman projektia tai tallennuksen aikana.

### Kamerarivit

Jokaisessa kamerarivissä näkyy värikoodattu reunus (kameran mukautettu väri), ”CAM”-merkintä — jossa on sininen **M**(master) tai vihreä**S** (slave) roolikirjaimella ryhmän jäsenille — sekä näyttönimen. Oletusnimi on `LATTICE-MODEL (serial)`; nimeä se uudelleen kamerakohtaisista asetuksista. Rivin painikkeet:

| Painike | Vaikutus |
| --- | --- |
| **Silmä**| Vaihtaa näkyvyyttä. Piilotetut kamerat poistuvat ruudukosta ja**jätetään pois Capture All -toiminnosta**. |
| **Hammasratas** | Avaa kamerakohtaisen asetuspaneelin (seuraava osa). |
| **Tauko / Toisto**| Jäädyttää reaaliaikaisen esikatselun**vain näytön puolella** — taustapuolella tallennus jatkuu. Tauko-tilassa olevat kamerat eivät voi tallentaa. |
| **X** | Katkaise yhteys. Käyttöliittymä päivittyy välittömästi (optimistinen arvio); taustatoimintojen yhteyden katkaiseminen voi kestää 10–30 sekuntia. |

### Ryhmän rivit

Ryhmän rivillä näkyy ”ARRAY”-merkki ryhmän värillä, ryhmän nimi (joka voidaan nimetä uudelleen ryhmän asetuksissa) sekä **DAQ · päällä/pois**-painike —**päällä**, kun ryhmätason valosensori on asetettu *tai* jollakin jäsenkameralla on kamerakohtainen sensori; sen työkaluvihjeessä näkyy tarkasti, mikä anturi syöttää mitäkin. Jäsenten kamerat on lueteltu sisennettyinä alla omilla riveillään. Ryhmärivin painikkeet: **silmä**(piilottaa/näyttää KAIKKI jäsenet yhdessä),**hammasratas**(ryhmän asetuspaneeli),**X**(katkaise koko ryhmän yhteys).

Ryhmäriveissä ja ryhmän asetuspaneelissa käytetyn valosensorin (DLS) tilalla on neljä tilaa:**pois**,**odottaa**(ei vielä spektriä),**aktiivinen**(spektri saapui viimeisten 3 sekunnin aikana) ja**vanhentunut** — ei tuoretta spektriä 3 sekunnin aikana, mutta viimeistä lukemaa *käytetään edelleen* (DAQ-lukemat eivät vanhene koskaan tallennuspolulla).

Kunnes järjestät sen uudelleen, luettelossa näkyvät ensin matriisit, sitten erilliset kamerat, ja kamerat (sekä kunkin matriisin jäsenet) **kamerajärjestyksessä**: M3C (Bayer) ennen M3M (mono); M3C:n joukossa FRGB, FRGN, FOCN ja sitten FNGB; M3M suodattimen aallonpituuden mukaan, pienimmästä alkaen. Ruudukon oletusasettelu noudattaa samaa järjestystä. Voit vetää erillisiä kameroita ja kokonaisia matriisiryhmiä sivupalkissa toistensa ohi luettelon järjestyksen muuttamiseksi; matriisin jäseniä ei voi vetää erikseen.

## Kamerakohtainen asetuspaneeli

Avaa kamerarivin **hammasrataskuvakkeella**. Paneeli liukuu kameraluettelon päälle.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->



**Otsikko**: kameran**värinäyte**(napsauta avataksesi laitteen oman värivalitsimen — asettaa sivupalkin reunan ja ruudun valintarenkaan värin),**nimi**, jossa on kynäkuvake**Nimeä uudelleen**(tyhjän nimen tallentaminen palauttaa oletusarvon `MODEL (serial)`), ja**×** sulkemiseen.

### Reaaliaikainen histogrammi

Paneelin yläosassa on reaaliaikainen luma-histogrammi, joka lasketaan JPEG

-esikatselukuvasta noin 8 Hz:n taajuudella. Keskiarvo on Bayer-painotettu — (R+2G+B)/4 — vastaamaan kameran omaa AE-mittausta.

* **Oranssi (Orange

) katkoviiva**= AE-kohde.**Vedä sitä vaakasuoraan kohteen muuttamiseksi** — yksi komento lähetetään, kun irrotat sormesi, ja vetäminen vaihtaa AE-kohdetilan manuaaliseksi.
* **Vihreä (Green

) yhtenäinen viiva** = todellinen keskimääräinen luma (se, mitä AE tuottaa tällä hetkellä).
* **RGB

-painike** (oikeassa yläkulmassa): vaihtaa kaistakohtaisten päällekkäisten histogrammien värejä kameran suodattimen mukaan (esim. FRGN-tilassa: harmaa NIR

, vihreä, punainen). Mono-kameroissa (M3M) painikkeessa lukee ”MONO” ja se on pois käytöstä — mono-tilassa näytetään aina yksikaistainen luma-histogrammi.
* X-akselin merkinnät noudattavat nykyisen pikselimuodon anturin bittisyvyyttä: 0..255, 0..1023, 0..4095 tai 0..65535.

### Kameratietorivit

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->



| Rivi | Toiminta |
| --- | --- |
| **Malli** | Vain luku -tila (esim. `LATT-M3C-L87-FRGN`). |
| **Radiometrinen kalibrointi** | Green**&quot;Active&quot;**-merkki, jonka tekstissä näkyy kalibrointitaso, hash-arvo, kalibrointipäivämäärä ja kaistaluettelo, jotka on ladattu kameran kalibrointipaketista (katso [Tehdasradiometrinen kalibrointi](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Piilotettu RGB

-kameroissa** — niissä on fotometrinen valkotasapainokalibrointi, ei kaistakohtaista säteilyvoimakkuutta. |
| **Kalibrointiraportti**|**Lataa**-painike — avaa kameran sarjanumeroittain jaetun NIST-kalibrointitodistuksen PDF-tiedostona käyttöjärjestelmän katseluohjelmassa. Jos todistusta ei ole vielä tallennettu välimuistiin, Chloros

näyttää sen sijaan vihjeen. |
| **Sarjanumero** | Vain luku -tila. |
| **Laiteohjelmisto**| Näyttää nykyisen version ja hakee sitten tämän mallin saatavilla olevan version (tallennetaan välimuistiin mallikohtaisesti — N-kamerajärjestelmä tarkistaa palvelimen kerran). Tilat: ”Tarkistetaan…” →**”Päivitä versioon X”**-painike → ”Päivitetään…” → ”Päivitetty A → B” / ”Epäonnistui: …” / ”Ohitettu: …” / vihreä**”Ajantasalla”**. Päivityspainikkeen työkaluvinkki: ”Tehdasasetusten palautus + flashaus + UserSet1:n uudelleenohjelmointi. ~2–3 minuuttia; älä katkaise yhteyttä.” |
| **IP** | Vain luku. |
| **Lämpötila** | Vain luku -tila, päivitetään 3 sekunnin välein. Muuttuu oranssiksi, kun lämpötila on ≥65 °C, ja punaiseksi ⚠-merkillä, kun lämpötila on ≥75 °C. |
| **Kalibrointikohde** valintaruutu | Ottaa käyttöön ArUco-heijastavuustavoitteen tunnistuksen paneelikohtaisella NDVI

-validointitaulukolla reaaliaikaisen syötteen alapuolella (luettelonäkymä) — katso [Kalibrointikohdetaulukko](#the-calibration-target-table). Vain istunnon ajan — avautuu aina pois päältä. |
| **Valosensori**-pudotusvalikko | Liittää DAQ-valosensorin (DAQ-E/M/U, Valosensorit-välilehden luettelosta) tähän kameraan alaspäin suuntautuvan valon (DLS) valaistuksen korjausta ja ennakoivaa automaattista valotusta varten. &quot;”Ei mitään” poistaa liitännän. Jos antureita ei ole kytketty, pudotusvalikossa näkyy ”(antureita ei kytketty — avaa DAQ-välilehti)”. Liitos tallennetaan projektin mukana. |

#### Kalibrointikohdetaulukko

Taulukko on **vain validointitarkistusta varten** (&quot;vain validointia varten — ei muuta kalibrointia&quot;). Sen otsikossa mainitaan kameran sarjanumero, suodatin, kohteen numero ja merkin koko pikseleinä, ja siinä on yksi rivi kutakin paneelia kohti:

* **Paneeli**, jonka jälkeen on yksi sarake kutakin kaistaa kohti, jossa on paneelin arvo — Bayer-kamera näyttää kolme kaistaansa nimen ja aallonpituuden mukaan (esim. FRGN: Red

660 nm / Green

550 nm / NIR

850 nm); **mono (M3M)-kamera näyttää yhden kaistansa** (esim. `F750`) eikä siinä ole NDVI

-sarakkeita.
* **Sat** — paneelin kyllästyneiden pikselien osuus (keltainen alkaen 1 %, punainen alkaen 5 %).
* **NDVI

Meas**,**NDVI

Theo** (paneelin vertailuheijastavuus-NDVI

) ja **Δ**— vain Bayer-monispektrikameroille. Pieni merkki ”NDVI

Meas” -merkinnän jälkeen kertoo, mistä mitattu NDVI

on laskettu:**ρ**(heijastavuus — valosensorin lukema on käytettävissä),**L**(säteily — valosensorin lukemaa ei ole) tai**DN**(kameran raaka-arvot). Vain**ρ**-NDVI

on vertailukelpoinen NDVI

Theo:n kanssa: L:n ja DN:n kohdalla Δ-sarake on harmaana ja huomautus kertoo tämän (&quot;Ei valosensorin lukemaa: NDVI

on peräisin säteilyvoimakkuudesta, joten Δ ei ole kalibrointitarkistus&quot; / &quot;NDVI

on peräisin kameran raaka-arvoista, joten Δ ei ole kalibrointitarkistus&quot;). Arvon ρ ollessa kyseessä Δ on vihreä alle 0,02:n, keltainen alle 0,05:n ja muutoin punainen.

<!-- SCREENSHOT-NEEDED: a 1.3.0 Calibration Target table under a live feed in list view — a Bayer camera's table showing the band columns, Sat, and "NDVI Meas ρ" / NDVI Theo / Δ; ideally next to a mono camera's single-band table with no NDVI columns. -->

### Valotus ja vahvistus

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Kaikki tämän osion numeeriset syöttökentät käyttävät pidä painettuna nopeuttaaksesi -tyyppisiä säätimiä: napautus = ±1, pidä painettuna &gt;1,5 s = ±10, pidä painettuna &gt;3 s = ±100. Arvo lähetetään kameraan, kun vapautat painikkeen.

| Säädin | Alue / vaihtoehdot | Oletus | Koskee | Toiminto |
| --- | --- | --- | --- | --- |
| **Valotus (us)**| Kameran reaaliaikaiset minimi-/maksimiarvot | Auto | Kaikki | Valotusaika mikrosekunteina, jossa on**Auto/Manuaali**vaihtoehdon. Auto = jatkuva kameran oma automaattinen valotus. Manuaalisen valotuksen asettaminen jättää**vahvistuksen** Auto-tilaan, jos se oli siinä, jolloin kameran koko vahvistusalue on käytettävissä, kunnes muokkaat vahvistusta tai asetat sen manuaaliseksi. |
| **Vahvistus (dB)**| Kameran reaaliaikaiset minimi- ja maksimiarvot (esim. enintään 48 dB) | Manuaalinen (pois) | Kaikki | Analoginen/digitaalinen vahvistus, jossa on oma**Auto/Manuaali**-kytkin. |
| **AE-kohdekirkkaus**| 0–255 | 80, tila**Auto**| Kaikki (muokattavissa, kun AE tai automaattinen vahvistus on päällä) | Kirkkaus, johon AE tähtää.**Auto**-tilassa (oletus) histogrammipohjainen taustasäädin valitsee tavoitearvon itse ja pitää valotuksen 60–75 %:ssa anturin maksimiarvosta. Arvon kirjoittaminen tai histogrammin oranssin viivan vetäminen vaihtaa tilan**Manuaaliseksi**. |
| **AE-tasoitus** | 0,5–40, askel 0,1 | 8,0 | Kaikki | AE-vaimennus. Työkaluvinkki: &quot;Pienempi arvo = AE reagoi nopeammin (voi pulsoida korkeilla kuvataajuuksilla). Suurempi arvo = tasaisempi / hitaampi.” Oletusarvoa huomattavasti pienemmät arvot voivat aiheuttaa AE:n pulsointia ja horjuttaa suoratoistoa korkeilla kuvataajuuksilla; 8,0 on vakaa oletusarvo. |
| **AE:n kohdealue**| Valintaruutu +**Aim**-painike | Pois | Kaikki | Kun tämä on päällä, AE mittaa vain viivalla merkittyä vihreää aluetta koko kehyksen sijaan.**Tähtäys** aktivoi napsauta-ja-sijoita-toiminnon suorassa lähetyksessä: napsautus keskittää alueen 30 %:iin kehyksestä; napsauta ja vedä piirtää mukautetun suorakulmion (vähintään 5 % × 5 %). Tavoite-toiminto poistuu käytöstä yhden sijoituksen jälkeen. Alue kartoitetaan takaisin kameran omille koordinaateille asettamasi kierto- tai peilausasetusten mukaisesti ja tallennetaan projektin mukana. |
| **AE-säätönopeus** | 0,1–5, askel 0,1 | 1,0 | Vain Array-jäsenille | Kuinka nopeasti automaattisen AE-kohteen seuranta reagoi kohtauksen kirkkauden muutoksiin; 1,0× tarkistaa tilanteen uudelleen 2,5 sekunnin välein. |
| **Kirkkaiden alueiden suojaus** | Tiukka (1 %) / Normaali (5 %) / Löyhä (15 %) | Tiukka | Kamerat, joissa tämä asetus on käytettävissä | Kuinka suuri osa kuvasta saa leikkautua valkoiseksi, ennen kuin AE tummentaa kuvaa. |

{% hint style="info" %}
**Valaistusvaatimukset Bayer-monispektrikameroille (RGN

/ OCN

/ NGB

):** kuvauskohteessa on oltava riittävästi valoa kaikissa kolmessa kanavassa, muuten kalibrointi ei toimi oikein — yksi aistimen valotus kattaa kaikki kolme spektriä. Mittaa valaistusta DAQ-valosensorilla tai käytä kokonaan monokromista tilaa (M3M), jolloin jokainen kaista saa oman valotuksensa. Jos kuvaus rikkoo tätä sääntöä, Chloros

havaitsee sen ja varoittaa sinua (unmix-clamp-ilmoitus). Reaaliaikaisessa näkymässä kanavajakokuvaruudussa näkyy **”Low-confidence band data · N%”**, kun unmix on rajoittanut kyseisen kaistan 25 %:ssa tai useammassa sen pikseleistä.
{% endhint %}

{% hint style="info" %}
**Kuinka automaattinen valotus käyttää valotusaikaa ja vahvistusta (1.3.0).** Kun vahvistus on asetettu automaattiseksi (Gain on Auto), Chloros

pitää vahvistuksen 0 dB:ssä, kunnes valotus on saavuttanut sallitun enimmäisarvon (AE:n yläraja, matriisin laukaisuväli tai DAQ:n valotuksen yläraja); vasta sitten vahvistus nousee. Matriisissa laukaisutaajuus on etusijalla: automaattinen valotus täyttää taajuuden määrittämän jaksonja nostaa sitten vahvistusta, joten pimenemässä oleva kohde ei enää laske automaattista laukaisutaajuutta (versiossa 1.2.0 se vielä laski). **Manuaalinen**valotus matriisin jäsenellä, joka kestää kauemmin kuin laukaisujakso, hidastaa matriisin laukaisutaajuutta sen mukaiseksi; automaattinen laukaisutaajuus palautuu, kun jokainen sen hidastanut jäsen on palannut automaattiseen valotukseen. Kun**Valotus**ja**AE-kohteen kirkkaus** ovat molemmat automaattisella ja DAQ-valosensori antaa lukeman, kunkin kameran valotusta pidetään vakiona, kun automaattinen valotus on vakiintunut, niin kauan kuin mitattu valo pysyy noin ±25 %:n rajoissa; suurempi, pitkäkestoinen valonmuutos, asetusten uudelleenkäynnistys (esimerkiksi ROI-alueen muutos) tai käyttäjän tekemä valotuksen muutos käynnistää sen uudelleen.
{% endhint %}

{% hint style="info" %}
**Automaattisen valotuksen toiminta (1.3.0).**

* Manuaalinen valotus jättää vahvistuksen automaattitilaan, kunnes muutat vahvistusta.
* Pienet kirkkaat kohdat eivät tummenna kuvaa: enintään 1 % kuvasta muodostava heijastus jätetään huomiotta, ja tunnetun kalibrointikohteen paneeleissa enintään 0,5 % paneelista.
* DAQ-valotuksen yläraja on voimassa vain silloin, kun kalibrointikohde on näkyvissä; jos paneelia ei näy, kamera valottaa kuvan kohteen mukaan.
* Valotus pysyy vakaana tasaisessa valossa ja vakiintuu muutaman sekunnin kuluessa suuresta muutoksesta. Leikkaus ei enää rajoita valotusta pysyvästi, ja ROI-alueen muutos palauttaa valotuksen oikeaksi.
* Kirkkaustavoite ei nouse kameranulottuvuuden ulkopuolelle, joten palaava valo ei aiheuta häikäisyä kuvassa.
* Kamera, jonka automaattinen valotus lakkaa reagoimasta kirkkaassa auringonpaisteessa, pidetään valotustasolla, joka vastaa sen ympäristön valaistusta ja joka luetaan valosensorista tai, jos sellaista ei ole kytketty, kameran omista kuvista, ja vapautetaan, kun valo muuttuu. Pimeä tai peitetty valosensori ei häiritse tätä.
{% endhint %}

### Pikselimuoto ja resoluutio

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

**Ryhmän jäsenet** näyttävät vain luku -tilassa olevat ”Current” (muoto + WxH) ja ”Binning” -rivit, joissa on huomautus ”Set in array settings” — yhden jäsenen virran uudelleenkäynnistys katkaisisi synkronoinnin, joten näitä hallitaan [ryhmän asetuspaneelissa](#array-settings-pane).**Erilliskameroissa** on seuraavat asetukset:

| Ohjaus | Vaihtoehdot | Toiminto |
| --- | --- | --- |
| **Pikselimuoto** | 8-bittinen / 10-bittinen / 12-bittinen / 16-bittinen | Anturin bittisyvyys; formaattiperhe on anturin oma. Oletusarvo on 12-bittinen, ja projekti tallentaa asettamasi arvon, mukaan lukien 8-bittisen, ja muodostaa yhteyden uudelleen sen mukaan. |
| **Resoluutio** | Täysi / Puoli / Neljännes | Suhteessa nykyiseen binningiin: Täysi = 2048/N × 1536/N N×N-binningissä. |
| **Binning** | 1x1 (ei mitään) / 2x2 / 4x4 | Laitteistopohjainen N×N-binning — suuremmat arvot pienentävät resoluutiota, mutta parantavat signaali-kohinasuhdetta (SNR) ja kuvataajuutta. Asetuksen muuttaminen käynnistää datavirran uudelleen ja nollaa kaikki ROI-alueet uuteen täyteen näkökenttään. |
| **Nykyinen** | vain luku -tila | Voimassa olevat todelliset WxH-arvot ja (x, y)-siirtymä. |

### Live-esikatselu

Kaikki tässä osiossa on **vain näyttöpuolen asetuksia**— ne muuttavat sitä, mitä näet suorassa lähetyksessä, kun taas tallennetut kuvat pysyvät lineaarisina ja muuttumattomina — yhtä poikkeusta lukuun ottamatta:**Vignette** on radiometrinen ja vaikuttaa myös vientiin (kuten alla mainitaan).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->



| Säädin | Alue / vaihtoehdot | Oletus | Koskee | Toiminto |
| --- | --- | --- | --- | --- |
| **Renderöintitarkkuus** | 360p (nopein) / 480p / 720p / 1080p / Alkuperäinen anturitarkkuus (hitain) | 720p | Kaikki | Korkeus, jolla taustaprosessi suorittaa radiometrisen esikatseluketjun. Pienempi arvo parantaa kuvataajuutta muuttamatta näkökenttää. |
| **Indeksi**| Valintaruutu ”Ota käyttöön” + ratas | Pois | Vain Bayer-monispektriset,**ei** yhdistettyjen matriisien jäsenet | Kasvillisuusindeksin reaaliaikainen esikatselu. Hammasratas avaa jaetun [Indeksilaskurin](#index-calculator-pane), johon on esiladattu kameran suodattimen luonnolliset kaistat (esim. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Mukautettu lauseke sekä LUT (päällä/pois, oletustaso 7-stop, oletusminimi 0,2, oletusmaksimi 1) lasketaan jokaisessa esikatselukuvassa. Yhdistettyjen matriisien jäsenet piilottavat tämän rivin — matriisi omistaa yhden jaetun indeksin. |
| **Valkotasapaino** | Pois / Kerran / Jatkuva + uudelleenkuvauspainike | Jatkuva | Vain RGB

| Reaaliaikainen valkotasapaino. Päivitä-painike tallentaa valkotasapainon uudelleen nykyisestä DLS-spektristä (pois käytöstä, kun tila on Pois). |
| **Gamma** | Päällä / Pois | Päällä | Vain RGB

| Näytä gamma (γ = 2,2 LUT) reaaliaikaisessa esikatselussa. Tallennetut kuvat pysyvät lineaarisina. |
| **Kohinanpoisto** | Valintaruutu + voimakkuus 0–100 | Pois / 50 | Kaikki (kamera-kohtaisesti, myös ryhmien sisällä) | Bilateraalinen suodatin reaaliaikaisessa esikatselussa. Suurempi arvo = tasaisempi, mutta pehmeämmät yksityiskohdat. |
| **Terävyys** | Valintaruutu + voimakkuus 0–100 | Pois / 30 | Kaikki | Terävöintimaski reaaliaikaisessa esikatselussa, sovelletaan viimeisenä. Voi vahvistaa kohinaa. Vain esikatselussa. |
| **Vignette**| Valintaruutu + voimakkuus 0–100 | Pois / 0 | Kaikki | Manuaalinen jäännösvignetoinnin poisto (kirkastaa kulmia), joka lisätään ryhmän Smart Vignette -arvion päälle.**Radiometrinen — vaikuttaa reaaliaikaiseen näkymään JA vientiin**, toisin kuin kohinanpoisto ja terävyys. |
| **Väriprofiili** | Raw / Lineaarinen / Luonnollinen / Parannettu / Mukautettu lämpötila | Luonnollinen | Vain RGB

| Katso alla. |
| **Värilämpötila** | 2000–10000 K, askel 100 | 5500 K | Vain RGB

, mukautettu lämpötilaprofiili | Kiinnittää valkotasapainon kiinteään korreloituun värilämpötilaan (DLS-syöttöä ei huomioida). Viimeksi valittu Kelvin-arvo muistetaan profiilinvaihtojen yli. |
| **Kirkkaat kohdat** | −100 – 0, askel 1 | 0 (pois päältä) | Vain RGB

| Uutta versiossa 1.3.0. Tummentaa kuvan kirkkaimpia, valkoisimpia alueita (0 = ei muutosta, −100 = voimakkain). Koskee reaaliaikaista esikatselua **sekä näytötasoisia (Preview) kaappauksia**, jotka tallentavat asetuksen (`Chloros:DisplayHighlights`); säteily- ja heijastavuusvientiä tämä ei koskaan vaikuta. Se ei voi palauttaa leikkautuneita anturitietoja — vähennä valotusta sen vuoksi. Zebra-peittokuva seuraa tummennettua kuvaa. Poissa käytöstä Raw-väriprofiililla (”Ei sovelleta Raw-väriprofiilin kanssa.”). |
| **Kylläisyys** | 0–200 (100 = neutraali) | 100 | Vain RGB

| HSV-kylläisyys reaaliaikaisessa esikatselussa. |
| **Kontrasti** | 0–200 (100 = neutraali) | 100 | Vain RGB

| Lineaarinen kontrasti keskiharmaan ympärillä reaaliaikaisessa esikatselussa. |
| **Peilaa vaakasuoraan / Peilaa pystysuoraan** | Valintaruudut | Pois | Kaikki | Käännä esikatselua vaakasuoraan / pystysuoraan. |
| **Kierto**| 0° / 90° / 180° / 270° | 0° | Kaikki | Kierrä esikatselua. Suunta sovelletaan esikatseluketjun lopussa.**Tallennetut kuvat (1.3.0):**erillisen kameran esikatselu-, säteily-, heijastus- ja indeksituotteet käännetään peilaus-/kiertotoiminnoilla, kun taas**Raw- ja Debayered-tuotteet säilyttävät kameran alkuperäisen suunnan**; jokainen kuvatiedosto tallentaa asetuksen XMP-tiedostoonsa, joten raakakuvan myöhempi käsittely soveltaa sitä samalla tavalla.**Array-jäsenetiedostot säilyttävät alkuperäisen suunnan** (asetus tallennetaan, mutta sitä ei sovelleta), ja array-yhdistelmänäkymät eivät ota sitä huomioon. |**Väriprofiilien semantiikka** (RGB

-kamerat):

* **Raw** — ohittaa käsittelyketjun kokonaan.
* **Lineaarinen** — tummasignaali + tasokenttä + valkotasapaino; ei värimatriisia, ei gammaa.
* **Natural** *(oletus)* — lineaarinen plus mitattu värinkorjausmatriisi ja kohtaukseen sopeutuva sävykäyrä.
* **Enhanced**— Natural plus värikylläisyys ja CLAHE-paikalliskontrasti. Lisäkustannus koskee**vain reaaliaikaista esikatselua** — tallennetut kuvat saavat aina täyden viimeistelyn profiilista riippumatta.
* **Mukautettu lämpötila** — Luonnollinen, jossa valkotasapaino on kiinnitetty valitsemaasi Kelvin-arvoon.

{% hint style="warning" %}
Luonnollinen-, Parannettu- ja Mukautettu lämpötila -asetuksissa paneelissa näkyy sävyhuomautus: kuvia kirkastetaan kunkin kohtauksen mukaan, joten tallennettuja *näyttökuvia* ei voi verrata keskenään. **Vie säteilyvoimakkuus- tai heijastavuusarvot mittauksia varten.**
{% endhint %}

### Näytön päällekkäiset kerrokset (piirretään reaaliaikaisen videovirran päälle)

Nämä ovat vain käyttöliittymän ominaisuuksia — ne piirretään videon päälle, eivätkä ne koskaan vaikuta videovirtaan tai tallennuksiin.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Päällekkäinen kerros | Säätimet | Oletus | Toiminto |
| --- | --- | --- | --- |
| **Zebra** | Valintaruutu + kynnysarvo 200–255 | Pois / 250 | Magentanväriset diagonaaliset raidat leikkautuneissa pikseleissä. |
| **Ristiko | Valintaruutu | Pois | Kehyksen keskipistemerkki. |
| **Ruudukko** | Pois / 3 × 3 / 9 × 9 | Pois | Sommitteluruudukko. |
| **Histogrammi** | Valintaruutu + leveys 0,10–0,90 kuvan leveydestä | Pois / 0,25 | Lähetyksessä näkyvä histogrammikaistale. |
| **Focus Peak** | Valintaruutu + kynnysarvo 20–200 + värinäyte | Pois / 80 / `#ff5722` | Sobel-reunakorostus tarkennusta varten. |
| **Kanavajaot** | &quot;Näytä jaot (Red

/ Green

/ NIR

)&quot; / &quot;Piilota jaot&quot; -painike | Piilotettu | Lisää kolme itsenäistä harmaasävyistä kanavakohtaista ruutua yhdistelmäkuvan viereen (painikkeen teksti noudattaa kameran suodatinkanavia). Jokainen jaettu ruutu on siirrettävissä ja sillä on kameran reunusväri. Ei käytettävissä mustavalkokameroissa. Tallennetaan projektin mukana. |

### Pistemittari

* **Napsauta näytteenottoa varten**-valintaruutu: napsauta reaaliaikaista kuvaa yhden pikselin näytteen ottamiseksi (ristikohdistin merkitsee sen) tai napsauta ja vedä aluetta pikselien keskiarvon laskemiseksi.**Tyhjennä**poistaa näytteen ja kohdistimen. Ei voi olla käytössä samanaikaisesti AE-ROI**Aim** -tilan kanssa.
* **Näytä**-pudotusvalikko:**Raw (bittisyvyys)**— alkuperäiset digitaaliset arvot anturin bittisyvyydellä (esim. 12-bittinen → 0..4095) — tai**Näyttö (8-bittinen)** (oletus). Kun reaaliaikainen indeksi on aktiivinen, Näyttö-valinta näyttää sen sijaan lasketun indeksiarvon (esim. NDVI

).
* Lukupaneelissa näkyvät pikselikoordinaatit, kehyksen koko, pikselimuoto, bittisyvyys sekä kanavataulukko (Chan / Value / %), jossa on kaistamerkinnät ja aallonpituudet; Bayerin vihreät parit on keskiarvoistettu; alueiden näytteissä näkyy ”N px avg”.

Pistemittarin tila on voimassa vain kyseisen istunnon ajan.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Ennakoiva automaattinen valotus (DLS-ohjattu)

Tämä osio näkyy vain, kun **vähintään yksi DAQ-valosensori on kytketty** — ratkaisija tarvitsee reaaliaikaisen alaspäin suuntautuvan spektrin toimiakseen.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->



| Ohjaus | Alue | Oletus | Toiminto |
| --- | --- | --- | --- |
| **Ota käyttöön** | Valintaruutu | Päällä (erilliset kamerat) | Suljetun muodon ratkaisija käyttää DLS-spektriä sekä kameran kalibrointipaketin skalaareja saadakseen kirkkaimman kaistan lähelle kyllästymistä ja pitäen samalla himmeimmän kaistan SNR-rajan yläpuolella — yksi valotuskerta ratkaisua kohti, ei vakautussilmukkaa. Suunniteltu aurinkovoimaisiin aikavälikuvauksiin, joissa jokaisen kuvan valotuksen on oltava oikea. Taustaprosessi siirtyy huomaamattomasti reaktiiviseen automaattiseen valotukseen aina, kun DLS-lukema on vanhentunut tai puuttuu tai kalibrointipakettia ei ole ladattu. |
| **Tasoitus (α)** | 0,05–1,0, askel 0,05 | 0,3 | Peräkkäisten ennustettujen ratkaisujen tasoitus (pienempi arvo = tasaisempi). |
| **Kohteen heijastavuus**|**Kalibroi ρ uudelleen** -painike | — | Arvioi uudelleen ratkaisijan käyttämän kohteen heijastavuuskertoimen. |

{% hint style="info" %}
**Array Connect poistaa ennakoivan automaattisen valotuksen käytöstä oletuksena** — matriisikameroissa Chloros

in älykäs automaattinen valotus sekä kameran oma automaattinen valotus hoitavat valotuksen (kylläisyyssuojauksella), eikä ennakoivan automaattisen valotuksen yksittäisen kohtauksenheijastavuusarvio ei ole turvallinen sekakohtauksissa. Voit ottaa sen uudelleen käyttöön kamerakohtaisesti täällä, jos haluat nimenomaan DLS-ohjattua radiometristä valotusta.
{% endhint %}

**DAQ-ohjattu valotuksen yläraja ja saapuvan valon mukaan lukittu AE.** Edellä mainitusta valintaruudusta riippumatta, kun DAQ-valosensori on määritetty RGB

-kameraan, Chloros

laskee — mitatun absoluuttisen alaspäin suuntautuvan säteilyvoimakkuuden perusteella — suurimman valotuksen×vahvistuksen, jolla 100 %:n heijastavuuden pinta pysyy leikkautumisen alapuolella, ja soveltaa sitä **rajoitukseksi**automaattisessa valotuksessa. Versiosta 1.3.0 lähtien rajoitus on voimassa vain silloin, kun kalibrointikohde on kyseisen kameran näkökentässä (näkyy sen kuvakehyksessä viimeisten 10 sekunnin aikana); kun kohdetta ei ole näkymässä, kameran tavallinen automaattinen valotus, jossa on oma kirkkaiden alueiden suojaus, toimii sen sijaan, jotta muuta kuvaa ei pidetä tarpeettoman tummana. Kun yläraja on aktiivinen, kamera on**kiinnitetty tulevan valon mukaan**: se toimii avoimessa säätöpiirissä valonmittaukseen perustuvalla valotuksella, kun vahvistus on 0 dB — valotus seuraa mitattua valoa, ei kuvauskohteen sisältöä. Koska yläraja voi vain lyhentää valotusaikaa, se ei voi itsessään aiheuttaa leikkautumista. Versiosta 1.3.0 lähtien yläraja lasketaan kameran omalla valotusvakiolla, joka on peräisin sen kalibrointipaketista (laivaston keskiarvo käytetään vain, jos paketissa ei ole omaa arvoa). Yläraja kytkeytyy pois päältä automaattisesti — ja normaali kuvan automaattinen valotus jatkuu — aina, kun kalibrointikohdetta ei ole näkymässä, DAQ-lukema puuttuu, on vanhentunut (&gt;30 s), pimeä tai jos ≥15 % kuvasta leikkautuu kiinnitetyllä valotuksella (eli anturi ja kamera havaitsevat erilaista valaistusta). Tälle ei ole GUI-kytkintä; tämä on vakiokäyttäytyminen aina, kun RGB

-kameralla on DAQ-yhteys.

### Tiedonkeruu ja laukaisu

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

Ryhmän jäsenissä näkyvät lisäksi vain luku -muotoiset **Rooli**(Master sinisellä / Slave vihreällä),**Synkronointilinja**ja**Vertaislaitteet** -rivit.

| Ohjaus | Vaihtoehdot | Oletus | Huomautukset |
| --- | --- | --- | --- |
| **Laukaisutila** | Pois / Päällä | Päällä | Poissa käytöstä taulukon jäsenille (taulukko hallitsee laukaisua). |
| **Laukaisulähde** | Ohjelmisto / Linja 0 (M8) / Linja 1 / Linja 2 | Linja 0 | Piilotettu, kun laukaisutila on Pois; poissa käytöstä ryhmän jäsenille. Linja 0 on M8:n optisesti eristetty ulkoinen laukaisutulo. |
| **Kuvataajuus**| Auto / Manuaalinen + arvo | Auto |**Auto**: kameran kuvataajuuden rajoitusta ei ole käytössä — valotusaika määrää kuvataajuuden, ja ruudussa näkyy reaaliaikainen todellinen kuvataajuus.**Manuaalinen**: rajoitat kuvataajuuden liukusäätimellä (1–kaistanleveyden rajoittama enimmäisarvo), joka perustuu nykyiseen todelliseen taajuuteen. Ryhmän jäsenille näkyy vain luku -muotoinen ”N fps (reaaliaikainen)” ja teksti ”Määritetty ryhmäasetuksissa”. |

### Verkko / Siirto

| Rivi | Toiminta |
| --- | --- |
| **Paketin koko**| 1500 (Vakio) / 9000 (Jumbo) — oletus**Jumbo**. |
| **Siirtonopeus** | Vain luku -muotoinen linkin siirtonopeuden raja megabitteinä sekunnissa (MB/s). Taustapalvelu tasapainottaa tämän uudelleen kaikkien kytkettyjen kameroiden kesken jokaisen yhteyden muodostamisen tai katkaisemisen yhteydessä. |
| **Puskurin käsittely** | Vain luku -muotoinen puskurin käsittelytila. |

### Tallennus

Paneelin lopussa on **&quot;Avaa tallennusasetukset…&quot;** -painike, joka vie [Tallennusasetukset-paneeliin](capture.md#the-capture-settings-pane) (pois käytöstä, kunnes projekti on avoinna — ”Luo tai avaa projekti tallentaaksesi tallenteet”). Jos kamera on piilotettu tai keskeytetty, vihje muistuttaa sinua paljastamaan/jatkamaan ennen tallennusta.

## Ryhmäasetukset-paneeli

Avaa painamalla **hammasratasta**ARRAY-rivillä. Otsikko: matriisin nimi, jonka vieressä on nimeämiskynä ja**×** sulkemista varten. Alla olevat *vain yhdistetty* -merkityt osiot näkyvät vain matriiseissa, jotka on kytketty yhdistettyyn näyttötilaan.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->



### Synkronointi

Vain luku -tilassa olevat **Master**-,**Slaves**- ja**Sync Line**-rivit.

### Ympäristön valosensori

Näkyy sekä yhdistetyissä että erillisissä matriiseissa:

* **Valosensori**-pudotusvalikko — sitoo yhden DAQ:n koko matriisiin. Valinta astuu voimaan välittömästi, ja se välittyy jokaisen jäsenkameran omaan**Valoanturi**-pudotusvalikkoon (voit silti ohittaa valinnan kamerakohtaisesti) sekä alkaa välittää spektrejä matriisiin.
* Reaaliaikainen **Tila**-rivi: Pois · ”Odotetaan ensimmäistä spektriä…” · ”Aktiivinen — kaikkien ryhmän kameroiden valaistus on korjattu” · ”Ei uutta spektriä viimeisten 3 sekunnin aikana — käytetään edelleen viimeistä lukemaa (ei vanhentunut aikakatkaisua)…”.
* Huomautus ruudussa: &quot;Koko ryhmän kattava radiometrinen korjaus. Kamerakohtaiset asetukset ohittavat tämän.&quot;

### Kalibrointikohde

Oma ryhmänsä heti ympäristön valosensorin jälkeen, sekä yhdistetyille että erillisille ryhmille:

* **Ota käyttöön** -valintaruutu — &quot;Tunnista MAPIR

ArUco -kohde ja validoi NDVI

:n suhteen paneelin heijastavuus-LUT&quot;; ohjaa yhdistettyjen ruutujen kohdepeittoa ja validointitaulukoita. **Erillisessä** matriisissa se aktivoi jokaisen jäsenen oman [kalibrointikohdetaulukon](#the-calibration-target-table) (yksi kutakin kamerariviä kohti luettelonäkymässä).

### Tallennus — yhtenäiset anturiasetukset

Näkyy sekä yhdistetyille että erillisille ryhmille (ryhmän otsikossa lukee ”Tallennus (yhtenäinen kaikille kameroille)”). Nämä asetukset koskevat yhtenäisesti jokaista jäsentä (jäsenkohtaiset muutokset rikkoisivat synkronoinnin). Muokkaukset tallennetaan väliaikaisesti ja otetaan käyttöön yhdessä **Apply**-painikkeen kanssa — lukuun ottamatta**ROI on / ROI off**-kytkintä, joka otetaan käyttöön välittömästi.

<!-- SCREENSHOT-NEEDED: array settings Capture section (1.3.0) — Pixel Format, Binning, Resolution preset, the ROI crop row with the "ROI on" toggle, the "Select" button, the "max WxH" hint and Reset button, the W/H/X/Y fields, the full-sensor ROI map below them with the orange crop box and its "W×H @ X,Y" label above the box, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons. -->

| Ohjaus | Vaihtoehdot / alue | Toiminto |
| --- | --- | --- |
| **Pikselimuoto** | 8-bittinen / 10-bittinen / 12-bittinen / 16-bittinen | Yhtenäinen bittisyvyys kaikille jäsenille; kukin kamera käyttää kyseistä bittisyvyyttä omalle anturilleen. |
| **Binning** | 1x1 / 2x2 / 4x4 | Laitteistopohjainen binning — säilyttää koko näkökentän ja parantaa samalla signaali-kohinasuhdetta (SNR) ja kuvataajuutta. Asetuksen muuttaminen nollaa ROI-kentät uuteen täyteen näkökenttään. |
| **Resoluutio**-esiasetus | Täysi / Puoli / Neljännes | Suhteessa binningiin; täyttää ROI-kentät keskitetyllä rajauksella. |
| **ROI-rajaus (px)**|**ROI päällä / ROI pois**-kytkin,**Valitse**-painike, W / H / X / Y-kentät, koko anturin kartta | Anturin rajaus — katso [ROI-editori](#the-roi-editor). Leveys/korkeus kiinnittyvät 16:n kerrannaisiin (vähintään 64); siirtymät kiinnittyvät 4:n kerrannaisiin. ”max WxH”-vihje näyttää ylärajan ja**Nollaa** palauttaa kentät koko näkökenttään. Muokkauksen aikana oranssi rajausruutu seuraa jokaista kentän muutosta kartalla ja anturiruudulla. |
| **Laukaisutaajuus**| Auto / Manuaali-kytkin + fps 0,5–10, askel 0,5 |**Auto**(oletus): taustajärjestelmä laskee laukaisutaajuuden resoluution ja kaistanleveyden perusteella — syöttökenttä on pois käytöstä ja näyttää lasketun arvon.**Manuaalinen**: lukitsee arvon, kun valitset ”Käytä”. Lukittu taajuus, jota uusi kehyskoko ei kykene tukemaan, pidetään suurimpana sallittuna taajuutena kyseiselle koolle, ja se palautuu, kun kehyskoko sallii sen jälleen. |

Huomautus paneelissa: ”Muoto- tai resoluutiomuutokset käynnistävät kaikki kamerat uudelleen hetkeksi. Laukaisutaajuus otetaan käyttöön reaaliaikaisesti.” **Käytä / Peruuta** -painikkeet sijaitsevat paneelin alaosassa.

#### ROI-editori

Uutta versiossa 1.3.0. ROI-rajausrivi ja sen alla oleva kartta toimivat yhdessä:

* **ROI päällä / ROI pois päältä**— kytkee anturin rajauksen**välittömästi**päälle tai pois päältä (kamerat käynnistyvät uudelleen hetkeksi; ruudukon ruudussa näkyy ”Asetuksia sovelletaan…”).**Pois**lukee koko anturin tiedot ja**muistaa rajauksesi**— kentät näyttävät sen edelleen;**Päällä** ottaa sen uudelleen käyttöön. Jos ROI kytketään päälle ilman määritettyä rajausta, se soveltaa keskitettyä puolikokoista rajausta (&quot;Rajausta ei määritetty, joten sovellettiin keskitettyä puolikokoista rajausta. Säädä sitä kenttien avulla.&quot;).
* **Valitse**— piirtää rajauksen kartalle: vedä sen yli tai napsauta kahta vastakkaista kulmaa (Esc peruuttaa). Suorakulmio tallentuu W/H/X/Y-kenttiin, kiinnittyen edellä mainittuihin vaiheisiin; mitään ei sovelleta, ennen kuin painat**Käytä**, joten voit ensin hienosäätää sitä kenttien nuolilla. Yksi suorakulmio jokaista Valitse-painallusta kohti.
* **Kartta**näyttää rajauksen koko kennolla, joten voit laajentaa sitä takaisin: ”Koko kenno, reaaliaikainen”, kun kamerat lukevat koko kennon, tai ”Koko kennon tilannekuva ajankohdasta {aika}. Kamerat lukevat vain rajattua aluetta.” kun ne on rajattu. Jos kameraryhmä on lähettänyt virtaa aina vain rajattuna, kartassa lukee ”Ei vielä koko kennon näkymää: tämä kameraryhmä on lähettänyt virtaa vain rajattuna.” ja**Näytä koko kuva** -painikkeen, joka poistaa ROI:n käytöstä. Rajauksen `W×H @ X,Y`-merkintä sijaitsee ruudun ulkopuolella (sen yläpuolella tai alapuolella, kun ruutu ulottuu yläreunaan asti) sekä kartalla että reaaliaikaisessa ruudussa.
* **Käytä**vahvistaa kenttien asetukset: rajaus kytkee ROI:n päälle; täyskuva (**Nollaa**-toiminnon jälkeen) jättää sen pois päältä, mutta edellinen rajaus muistetaan; pois päältä oleva ROI, jonka kentissä näkyy edelleen muistettu rajaus, pysyy pois päältä, kun teet uuden muutoksen (formaatti, binning, kuvataajuus).
* **Kohdistetussa kameraryhmässä kunkin jäsenen rajausikkuna sijoitetaan kohdistuksen perusteella**, joten jokainen kamera kehystää saman osan kohtauksesta. Ennen kohdistusta rajattu kameraryhmä kohdistetaan ensin koko kennon alueelle (rajaus poistetaan, kohdistus ratkaistaan ja rajaus palautetaan kamerakohtaisilla ikkunoilla).
* Rajauksen muutoksen jälkeen automaattinen valotus sopeutuu uudelleen uuteen rajausalueeseen. Rajauksen aikana jäsenen valotus voi edelleen jatkua laukaisujakson loppuun asti.
* ROI:n päällä/pois-tila ja tallennettu rajaus tallennetaan projektin mukana.

### Kohdistus (yhteisrekisteröinti)

Näytetään **sekä yhdistetyille että erillisille matriiseille** (versiossa 1.2.0 se näytettiin vain yhdistetyille matriiseille); ryhmän otsikossa lukee ”Alignment”.

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* **Jäännös**-pilleri: ”RMS x,xx px” — vihreä alle 1 px, keltainen alle 3 px, punainen muussa tapauksessa tai jos jokin kamera epäonnistui; ”ei profiilia” ennen ensimmäistä ratkaisua.
* Yhteenvetorivi: ”✓ Kaikki kamerat kohdistettu (N)” / ”⚠ p/N kameraa kohdistettu — <serial (filter)="">epäonnistui” / ”Rajaaminen aktiivinen — Kalibroi uudelleen kohdistusta varten (käyttää koko kennoa)” / ”Odotetaan valotuksen vakiintumista…” / ”Asetuksia sovelletaan…” (kamerat käynnistyvät uudelleen) / ”Järjestelmä lakkasi toimittamasta kuvia — liitä järjestelmä uudelleen”.
* Kamerakohtainen taulukko: kamera (sarjanumeron 4 viimeistä merkkiä + suodatin), uudelleenprojektio-virhe px:inä ja vastaavuuksien lukumäärä (&quot;ref&quot; pääkameralle) sekä päällekkäisyyden normalisoitu ristikorrelaatiopisteet suhteessa 0,35:n läpäisyrajaan.
* **Kalibroi kohdistus uudelleen** -painike (teksti ”Kalibroi kohdistus” ennen ensimmäistä profiilia) — suorittaa ko-rekisteröinnin uudelleen uusilla kuvilla.
* **&quot;Automaattinen valotus kameroille kohdistusta varten&quot;** -valintaruutu (oletusarvoisesti valittuna) — kirkastaa väliaikaisesti tummia tai tasaisia kameroita (ensin valotus, sitten vahvistus), jotta niillä on tekstuuria, jota voidaan verrata, ja palauttaa sitten automaattisen valotuksen.

Chloros kohdistaa jokaisen matriisin automaattisesti yhteyden muodostamisen jälkeen molemmissa näyttötiloissa, kun matriisin jäsenten automaattinen valotus on vakiintunut (ohjelma yrittää tätä enintään 5 minuuttia); yhdistetty esikatselu kohdistaa myös avattaessa. Erillisellä ryhmällä ei ole yhdistettyä ruutua, joten tämä osa tarkistaa kohdistustilan ja suorittaa Recalibrate-toiminnon itse. Suorita Recalibrate, jos tarkennus tai kuvausetäisyys on muuttunut. Kohdistus on **suunniteltu vain istuntokohtaiseksi** — sitä ei koskaan tallenneta profiiliin, koska se riippuu kulloisestakin kuvausetäisyydestä. Tallenteet voidaan silti viedä pikselikohtaisesti kohdistettuina (katso [Kohdistetut vientitiedostot](capture.md#per-array-controls)).

### Älykäs vinjetointi

* **Ota korjaus käyttöön**-valintaruutu — soveltaa kamerakohtaista vinjetointiarviota radiometriseen ketjuun (reaaliaikaisesti**ja** vientitiedostoissa).
* **Kalibroi nykyisestä näkymästä**— suuntaa matriisi ensin tasaiselle kohteelle (litteä paneeli, seinä tai taivas); jokainen kamera tasoitetaan erikseen ja tilailmoitus kertoo tasaisuuden parannuksen muodossa &quot;n/N kameraa · −x,x %&quot;.**Tyhjennä** poistaa arvion.
* Tila: ”Kalibroitu”, ”Ei kalibroitu” tai ”Osittain kalibroitu — suorita uudelleen”, kun vain joidenkin kameroiden arviot pätevät edelleen niiden nykyisessä kuvauskehyksessä (esimerkiksi kun ”Kalibroi uudelleen” -toiminto on siirtänyt kameran rajausikkunaa) — suorita kalibrointi uudelleen.
* Hienosäädä kameraa kohden kamerakohtaisella **Vignette**-liukusäätimellä kohdassa [Live Preview](#live-preview).

### Live-esikatselu

* **Histogrammi**-valintaruutu + leveysliukusäädin (0,10–0,90 kuvan leveydestä) — syötteen päällä näkyvä histogrammikaistale jokaisen jäsenen ruudulla kerralla (mukaan lukien jaetut kanavaruudut), koko ryhmälle asetettu [histogrammipäällys](#display-overlays-drawn-over-the-live-feed) asetetaan koko ryhmälle. Se näkyy päällä, kun jokaisella jäsenellä on se jo.**Erillisessä** ryhmässä tämä on ryhmän ainoa rivi.
* *Vain yhdistettynä:* **Indeksi**: valintaruutu + ratas — avaa jaetun [Indeksilaskurin](#index-calculator-pane), jossa on**kaikkien** jäsenkameroiden kuvista piirretyt kaistat. Alla oleva lausekkeen esikatselurivi näyttää nykyisen lausekkeen (&quot;Lauseketta ei ole asetettu — avaa laskin luodaksesi sellaisen&quot;), joka päivitetään sekunnin välein.
* *Vain yhdistettynä:* **Renderöintitarkkuus**-pudotusvalikko (samat esiasetukset kuin kamerakohtaisesti, oletus 720p): live-näkymän virran korkeus**ja** tallennetun yhdistelmän vientikoko. Huomaa paneelissa: ”Esikatselu + tallennetun yhdistelmän koko. Kamerakohtaiset kuvat viedään aina täysresoluutiolla.”

### Näyttökerrokset *(vain yhdistetty)** **Ota käyttöön** -valintaruutu (oletusarvoisesti pois päältä — pääkamera näkyy suoraan; päällä = kerrostettu yhdistelmä).
* ****Etualalla**/**Taustalla**-pudotusvalikot: kukin jäsenkamera (nimen mukaan) tai**Indeksi**. Kun Etualalla on Indeksi, LUT:n minimi- ja maksimiarvojen ulkopuolella olevat pikselit näyttävät Taustakerroksen.

### Jaettu näkymä *(vain yhdistetty)*

**”Näytä jäsenkamerat”**—**Jaa / Piilota jäsenkamerat** -painike, joka lisää kunkin jäsenkameran oman reaaliaikaisen kuvan erillisinä ruudukon ruutuina yhdistelmäkuvan viereen. Ruudut lukevat ryhmän olemassa olevan kehyspuskurin (ei erillistä kamerayhteyttä). Vain ruudukkonäkymä; tallennetaan ryhmäkohtaisesti projektin mukana.

### Ominaisuudet

Vain luku -paneeli, joka päivitetään 5 sekunnin välein:

* **Tason nimike**: &quot;Samanaikainen tallennus&quot; (vihreä) · &quot;Samanaikainen tallennus (FTD-porrastettu lähetys)&quot; (vihreä) · &quot;Porrastettu tallennus (100 ms viive)&quot; (keltainen) · &quot;Määritys liian suuri&quot; (punainen).
* **Kehysten kunto**: &quot;x,xx % keskeneräistä&quot; — vihreä alle 1 %, keltainen alle 5 %, punainen 5 % tai enemmän.
* **Linkkilinja**: &quot;NIC {mbps} Mbps – jatkuva {MB/s} MB/s&quot;.

Tämä on kameraryhmän reaaliaikainen kaistanleveysbudjetti. Taustalla olevat kuvataajuudet (fps) ja verkkomalli — sekä mitä muutoksia on tehtävä, kun taso muuttuu keltaiseksi tai punaiseksi — löytyvät kohdasta [Monikamerajärjestelmät](arrays.md) ja [CLI-viiteoppaasta](../reference/cli-reference.md).

<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->

## Index Calculator -paneeli

Kolmas sivupalkin sivu, jota jakavat kamerakohtainen Index-asetus ja yhdistettyä järjestelmää koskeva Index-asetus (yksi kerrallaan – otsikossa lukee ”Index Calculator — <camera name="">” tai ”Index Calculator — <array name="">”). Se vastaanottaa kaistaluettelon (kameran suodattimenluonnolliset kaistat tai kaikki ryhmän jäsenten kaistat), nykyisen lausekkeen ja LUT-määritykset (päällä/pois, taso — oletus 7-stop, min — oletus 0,2, max — oletus 1) sekä reaaliaikaisen indeksi-histogrammin. **Käytä**-painike vahvistaa lausekkeen; LUT-muutokset näkyvät reaaliaikaisesti esikatselussa.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Kamerakohtaiset vs. matriisin hallinnoimat asetukset

Pikaopas siitä, mitkä asetukset sijaitsevat missä, kun kamera on matriisin jäsen:

| Matriisin hallinnoimat (vain luku -tila kamerapaneelissa) | Edelleen kamerakohtaiset matriisin sisällä |
| --- | --- |
| Pikselimuoto, resoluutio, binning | Automaattinen valotus (valotus, vahvistus, kohde, tasoitus, ROI) |
| Laukaisutila/lähde, kuvataajuus | Kohinanpoisto, terävyys, vignettaus |
| | Suunta (peilaus/kierto), näytön päällekkäisnäytöt (ryhmäasetukset → Live Preview asettaa histogrammin kerralla jokaiselle jäsenelle), pistevalotus |
| | Indeksi (erillisenä näytettävät ryhmät), valosensorin sidonta |

Muita kokonaisuutta koskevia toimintoja:

* **Yhdistetty vs. erillinen näyttö** valitaan matriisin liitännän yhteydessä: yhdistetty = yksi kohdistettu yhdistelmäruutu (jäsenet lähettävät kuvaa vain jaetun näkymän kautta); erillinen = kukin jäsen renderöi oman synkronoidun ruutunsa. Kamera ei koskaan näytä sekä erillistä kuvaa että matriisin ruutua samanaikaisesti.
* ****Automaattinen uudelleenkytkentä**: tallennetun projektin avaaminen palauttaa sen kamerat ja matriisit sekä soveltaa kaikki tallennetut asetukset taustapalvelimeen ennen kuin lähetykset jatkuvat.
* **Tallennuksen rajoitus**: piilotetut tai keskeytetyt kamerat jätetään pois Capture All -toiminnosta; matriisi estetään kokonaan vasta, kun KAIKKI jäsenet ovat piilotettuja/keskeytettyjä. Katso [Tallennusasetukset ja -tilat](capture.md).

## Asetusten säilyminen

Kameravälilehden tila tallennetaan **projektin mukana**, ei selaimeen:

* Jokainen reaktiivinen muutos tallentaa kamerat ja ryhmät projektin `cameras.json`-tiedostoon (viive 500 ms). Tämä kattaa kameranimet ja värit, valotus-/vahvistus-/AE-asetukset, pikselimuodon/resoluution/binningin, laukaisutaajuuden, esikatseluasetukset (renderöintiresoluutio, kohinanpoisto, terävyys, vinjetti, väriprofiili, korostukset, kylläisyys/kontrasti), suunnan, päällekkäisyydet, kanavajaot, indeksin konfiguraation, ennakoivan AE:n asetukset, AE-ROI:n, matriisin nimet, näyttötila, matriisin tallennusasetukset (mukaan lukien ROI-rajaus, onko ROI päällä ja muistettu rajaus, kun se on pois päältä), matriisin Wire Budget sekä ruudukkolohko (syötteen zoomaus, katselutila, ruudukon lukitus, manuaalinen ruutujärjestys, piilotetut kamerat, suljetut ruudut, aktiivinen kamera).
* Valosensorien sidokset tallennetaan projektin `sensors.json`-tiedostoon.
* Projektin uudelleen avaaminen yhdistää laitteiston uudelleen ja ottaa kaikki asetukset uudelleen käyttöön.
* **Ei avointa projektia = vain istunto**: ilman projektia mikään ei säily, kun Chloros suljetaan.
* Vain istuntoon liittyvät asetukset projektista riippumatta: taukotila, pistevalotusnäytteet, kamerakohtainen Kalibrointikohde-valintaruutu (avautuu aina pois päältä) sekä matriisin kohdistusprofiili (lasketaan uudelleen jokaisessa istunnossa suunnittelun mukaisesti).
* Yksi poikkeus: **Kuvausasetukset**-vientiä koskevat valinnat ja kuvausmoodi säilyvät projektikohtaisesti sovelluksen paikallisessa tallennustilassa eikä `cameras.json`:ssa — katso [Kuvausasetukset ja -moodit](capture.md).</array></camera></serial>
