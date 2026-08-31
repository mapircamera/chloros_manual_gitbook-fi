# Kameran asetukset

**Kamerat**-välilehti on Chloros:n reaaliaikainen ohjauspaneeli LATTICE-kameroille: pääkuva-alue, jossa jokainen kytketty kamera näkyy reaaliaikaisena ruutuna, sekä sivupalkki, jota voi liu’uttaa kolmen sivun välillä —**kameraluettelo**,**asetusruutu**(kamerakohtaiset, ryhmä- tai tallennusasetukset – yksi kerrallaan) sekä**indeksilaskuri**. Tällä sivulla kuvataan kaikki kameraluettelon, kamerakohtaisen asetusruudun ja ryhmäasetusruudun säätimet. Tallennustilat, vientityypin valinta sekä ”Capture All” -toiminto löytyvät oheissivulta [Tallennusasetukset ja -tilat](capture.md).

”Kamerat”-välilehti ilmestyy sivupalkkiin, kun Chloros-taustapalvelu on valmis. Kaikki alla olevat säätimet kommunikoivat paikallisen taustapalvelun `127.0.0.1:5000`:n kautta; muutokset näkyvät kameran reaaliaikaisessa kuvassa välittömästi, ellei toisin mainita.

## Tällä sivulla käytetyt kameratyypit

Säätimet näkyvät tai piiloutuvat sen mukaan, minkä tyyppinen kamera on valittuna. Käyttöoppaassa käytetään seuraavia termejä:

| Termi | Merkitys | Suodatinkanavat |
| --- | --- | --- |
| **RGB-kamera** | LATTICE M3C, jossa on FRGB-suodatin (malli sisältää `-FRGB`) | Red / Green / Blue |
| **Bayer-monispektrinen** | LATTICE M3C, jossa on FRGN, FOCN tai FNGB | FRGN: Red / Green / NIR · FOCN: Orange / Cyan / NIR · FNGB: NIR / Green / Blue |
| **Mono (M3M)** | LATTICE M3M — yksi kapeakaistainen suodatin, yksi kalibroitu kaista | Yksikaistainen |
| **Ryhmän jäsen** | Kamera, joka on kytketty osaksi synkronoitua ryhmää (yhdistetty tai erillinen näyttö) | Suodattimensa mukaan |

RGB-kameroille suoritetaan fotometrinen käsittely (valkotasapaino, väriprofiilit, gamma); monispektriset ja monokamerat käyvät läpi radiometrisen ketjun ja ohittavat fotometriset säädöt. Ryhmän jäsenet välittävät stream-tason asetukset (pikselimuoto, resoluutio, binning, laukaisija, kuvataajuus) ryhmälle — nämä rivit muuttuvat kamerakohtaisessa paneelissa vain luku -tilaan ja siirtyvät sen sijaan ryhmän asetuspaneeliin.

## Pääsyötealue

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

Kun kameroita ei ole kytketty, syötealueella näkyy **&quot;Kytke kamera aloittaaksesi&quot;**-aloitusnäyttö, jossa on kaksi painiketta:**Liitä kamera**(vihreä, avaa yhden kameran liitäntävalintaikkunan) ja**Liitä kamerajärjestelmä** (sininen, avaa kamerajärjestelmän liitäntävalintaikkunan). Itse liitäntävalintaikkunoista on lisätietoja kohdassa [Kameroiden liittäminen](connecting.md); ryhmäkonseptit (synkronointi, tasot, kaistanleveys) on kuvattu kohdassa [Monikameraryhmät](arrays.md). Kun avaat tallennetun projektin, jossa on kameroita, aloitusnäytössä näkyy sen sijaan pyörivä kuvake ja teksti &quot;Avaan uudelleen N tallennettua kameraa…&quot; kun taas Chloros palauttaa virrat edellisestä istunnosta.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Yläpalkki

| Ohjaus | Toiminto |
| --- | --- |
| **Näkymätilan vaihtaminen**| Vaihtaa**ruudukkonäkymän**(kaikki ruudut soluina) ja**luettelonäkymän** (järjestelmät täysleveinä yläosassa, YKSI aktiivinen kamera alla) välillä. Työkaluvinkit: ”Vaihda ruudukkonäkymään” / ”Vaihda luettelonäkymään”. |
| **Ruudukon lukitus**(lukko) | Oletusarvoisesti**lukittu** — ruudut on jäädytetty paikoilleen. Avaa lukitus, jotta voit vetää ja järjestellä ruutuja mihin tahansa paikkaan (aukot säilyvät). Ruudukko lukittuu automaattisesti uudelleen aina, kun uusi kamera muodostaa yhteyden. Työkaluvinkit: &quot;Avaa ruudukon lukitus (ota ruutujen vetäminen käyttöön)&quot; / &quot;Lukitse ruudukko (jäädyttää ruudut paikoilleen)”. |
| **Syötteen zoomaus**-liukusäädin | Ruudun koko, 60 pikselistä säiliön koko leveydelle asti. Solut säilyttävät 4:3-kuvasuhteen. Alle 200 pikselin soluleveydellä nimi- ja fps-päällekkäisnäytöt piilotetaan ruudun siisteyden säilyttämiseksi. |

### Syötteen ruudut

Jokainen kamera renderöi yhdistelmäruudun; kamera voi lisäksi näyttää kolme harmaasävyistä **kanavakohtaisesti jaettua** ruutua (katso [Kanavajaot](#display-overlays-drawn-over-the-live-feed)), ja ruudukot tuottavat yhdistelmäruudun. Aktiivisessa ruudussa näkyy valintarengas kameran (tai ruudukon) värillä.

Kun viet hiiren ruudun päälle, näkyviin tulee **X**-sulkupainike:

* **Yhdistelmäruudun** sulkeminen, kun sen kanavajakoruudut ovat edelleen näkyvissä, piilottaa vain yhdistelmäruudun.
* **Yksittäisen kameran viimeisen näkyvän ruudun** sulkeminen irrottaa kyseisen kameran.
* **Yhdistettyjen kameraryhmien jaetut ruudut eivät koskaan katkaise** kameran yhteyttä — ne vain piilotetaan.

Kun ruudukko on lukitsematon, vedä mikä tahansa ruutu mihin tahansa paikkaan; asettelu tallennetaan projektin mukana.

## Sivupalkki — kameraluettelo

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

Ensimmäisellä sivupalkkisivulla on luettelo kaikista kytketyistä kameroista ja ryhmistä:

* **Yhdistä kamera**(vihreä) /**Yhdistä ryhmä** (sininen, näyttää ”Havaitaan...” skannauksen aikana). Molemmat ovat pois käytöstä, kun liitäntävalintaikkuna on auki.
* **Tallenna kaikki** (punainen) — tallentaa kaikki luettelossa olevat kamerat tallennusasetuksissa valituilla vientityypeillä. Edellyttää avointa projektia. Täydelliset ohjeet kohdassa [Tallennusasetukset ja -tilat](capture.md).
* **Tallennusasetusten hammasratas** (”Tallenna kaikki” -painikkeen vieressä) — avaa [Tallennusasetukset-paneelin](capture.md#the-capture-settings-pane). Pois käytöstä ilman projektia tai tallennuksen aikana.

### Kamerarivit

Jokaisessa kamerarivissä näkyy värikoodattu reunus (kameran mukautettu väri), ”CAM”-merkintä — jossa on sininen **M**(pääkamera) tai vihreä**S** (alakamera) roolikirjaimella ryhmän jäsenille — sekä näytönimen. Oletusnimi on `LATTICE-MODEL (serial)`; voit nimetä sen uudelleen kamerakohtaisissa asetuksissa. Rivin painikkeet:

| Painike | Vaikutus |
| --- | --- |
| **Silmä**| Näkyvyyden kytkeminen päälle/pois. Piilotetut kamerat poistuvat ruudukosta ja**jätetään pois Capture All -toiminnosta**. |
| **Hammasratas** | Avaa kamerakohtaisen asetuspaneelin (seuraava osa). |
| **Tauko / Toisto**| Jäädyttää reaaliaikaisen esikatselun**vain näytön puolella** — taustalla tapahtuva tallennus jatkuu. Tauko-tilassa olevat kamerat eivät voi tallentaa. |
| **X** | Katkaise yhteys. Käyttöliittymä päivittyy välittömästi (optimaalinen tilanne); yhteyden katkaiseminen taustalla voi kestää 10–30 sekuntia. |

### Ryhmän rivit

Ryhmän rivillä näkyy ”ARRAY”-merkki ryhmän värillä, ryhmän nimi (voidaan nimetä uudelleen ryhmän asetuksissa) sekä **DAQ · päällä/pois**-painike —**päällä**, kun ryhmätason valosensori on asetettu *tai* jollakin jäsenkameralla on kamerakohtainen sensori; sen työkaluvinkissä luetellaan tarkasti, mikä anturi syöttää mitäkin. Jäsenkamerat on lueteltu sisennettyinä alla omilla riveillään. Järjestelmän rivin painikkeet: **silmä**(piilottaa/näyttää KAIKKI jäsenet yhdessä),**hammasratas**(ryhmän asetuspaneeli),**X**(katkaise koko ryhmän yhteys).

Ryhmän riveissä ja ryhmän asetuspaneelissa käytetyn valosensorin (DLS) tilalla on neljä tilaa:**pois**,**odottaa**(ei vielä spektriä),**aktiivinen**(spektri saapui viimeisten 3 sekunnin aikana) ja**vanhentunut** — ei tuoretta spektriä 3 sekunnin aikana, mutta viimeistä lukemaa *käytetään edelleen* (DAQ-lukemat eivät vanhene koskaan tallennuspolulla).

Voit järjestää luettelon uudelleen vetämällä yksittäisiä kameroita ja kokonaisia matriisiryhmiä sivupalkissa toistensa ohi; matriisin jäseniä ei voi vetää erikseen.

## Kamerakohtainen asetuspaneeli

Avaa kamerarivin **hammasratas**-painikkeella. Paneeli liukuu kameraluettelon päälle.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->

**Otsikko**: kameran**värinäyte**(napsauttamalla avaat alkuperäisen värivalitsimen — asettaa sivupalkin reunan ja ruudun valintarenkaan värin),**nimi**, jossa on kynäkuvake**Nimeä uudelleen**(tyhjän nimen tallentaminen palauttaa oletusarvon `MODEL (serial)`) ja**×** sulkemiseen.

### Reaaliaikainen histogrammi

Paneelin yläosassa on reaaliaikainen luma-histogrammi, joka lasketaan JPEG-esikatselukuvasta noin 8 Hz:n taajuudella. Keskiarvo on Bayer-painotettu — (R+2G+B)/4 — vastaamaan kameran omaa AE-mittausta.

* **Orange katkoviiva**= AE-kohde.**Vedä sitä vaakasuoraan kohteen muuttamiseksi** — yksi komento lähetetään, kun irrotat sormesi, ja vetäminen vaihtaa AE-kohdetilan manuaaliseksi.
* **Green yhtenäinen viiva** = todellinen keskimääräinen luma (se, mitä automaattinen valotus tällä hetkellä tuottaa).
* **RGB-painike** (oikeassa yläkulmassa): kytkee päälle kaistakohtaiset päällekkäiset histogrammit, jotka on värjätty kameran suodattimen mukaan (esim. FRGN-tilassa: harmaa NIR, vihreä, punainen). Mono-kameroissa (M3M) painikkeessa lukee &quot;MONO&quot; ja se on pois käytöstä — mono-tilassa näytetään aina yksikaistainen luma-histogrammi.
* X-akselin merkinnät noudattavat nykyisen pikselimuodon anturin bittisyvyyttä: 0..255, 0..1023, 0..4095 tai 0..65535.

### Kameratietorivit

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

| Rivi | Toiminta |
| --- | --- |
| **Malli** | Vain luku (esim. `LATT-M3C-L87-FRGN`). |
| **Radiometrinen kalibrointi**| Green**”Active”**-merkki, jonka tekstissä näkyy kalibrointitaso, hash-arvo, kalibrointipäivämäärä ja kaistaluettelo, jotka on ladattu kameran kalibrointipaketista (katso [Tehdasasetusten radiometrinen kalibrointi](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Piilotettu RGB-kameroissa** — niissä on fotometrinen valkotasapainokalibrointi, ei kaistakohtaista säteilyvoimakkuutta. |
| **Kalibrointiraportti**|**Lataa**-painike — avaa kameran sarjanumeroittain jaetun NIST-kalibrointitodistuksen PDF-tiedostona käyttöjärjestelmän katseluohjelmassa. Jos todistusta ei ole vielä tallennettu välimuistiin, Chloros näyttää sen sijaan vihjeen. |
| **Sarjanumero** | Vain luku -tila. |
| **Laiteohjelmisto**| Näyttää nykyisen version ja hakee sitten tämän mallin saatavilla olevan version (tallennetaan välimuistiin mallikohtaisesti — N-kamerajärjestelmä tarkistaa palvelimen kerran). Tilat: &quot;Tarkistetaan…&quot; →**&quot;Päivitä versioon X&quot;**-painike → &quot;Päivitetään…&quot; → &quot;Päivitetty A → B&quot; / &quot;Epäonnistui: …&quot; / &quot;Ohitettu: …&quot; / vihreä**&quot;Ajantasalla&quot;**. Päivityspainikkeen työkaluvinkki: ”Tehdasasetusten palautus + flashaus + UserSet1:n uudelleenohjelmointi. ~2–3 minuuttia; älä katkaise yhteyttä.” |
| **IP** | Vain luku. |
| **Lämpötila** | Vain luku -tila, päivitetään 3 sekunnin välein. Muuttuu oranssiksi, kun lämpötila on ≥65 °C, ja punaiseksi ⚠-merkillä, kun lämpötila on ≥75 °C. |
| **Kalibrointikohde**-valintaruutu | Ottaa käyttöön ArUco-heijastavuustavoitteen tunnistuksen paneelikohtaisella NDVI-validointitaulukolla reaaliaikaisen syötteen alla (luettelonäkymä). Vain istunnon ajaksi — avautuu aina pois päältä. |
| **Valosensori**-pudotusvalikko | Liittää DAQ-valosensorin (DAQ-E/M/U, Valosensorit-välilehden luettelosta) tähän kameraan alaspäin suuntautuvan valon (DLS) valaistuksen korjausta ja ennakoivaa automaattista valotusta varten. Valinta ”Ei mitään” poistaa liitännän. Jos antureita ei ole kytketty, pudotusvalikossa näkyy ”(antureita ei kytketty — avaa DAQ-välilehti)”. Liitäntä tallennetaan projektin mukana. |

### Valotus ja vahvistus

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Kaikki tämän osion numeeriset syöttökentät toimivat pidä-nopeuta-periaatteella: napautus = ±1, pidä painettuna &gt;1,5 s = ±10, pidä painettuna &gt;3 s = ±100. Arvo lähetetään kameraan, kun vapautat sormesi.

| Säädin | Alue / vaihtoehdot | Oletus | Koskee | Toiminto |
| --- | --- | --- | --- | --- |
| **Valotus (us)**| Kameran reaaliaikaiset minimi-/maksimiarvot | Auto | Kaikki | Valotusaika mikrosekunteina,**Auto/Manuaalinen**-kytkimellä. Auto = jatkuva kameran oma automaattivalotus. |
| **Vahvistus (dB)**| Kameran reaaliaikaiset minimi- ja maksimiarvot (esim. enintään 48 dB) | Manuaalinen (pois päältä) | Kaikki | Analoginen/digitaalinen vahvistus, jossa on oma**Auto/Manuaalinen**-kytkin. |
| **AE-kohdekirkkaus**| 0–255 | 80, tila**Auto**| Kaikki (muokattavissa, kun AE tai automaattinen vahvistus on päällä) | Kirkkaus, johon AE pyrkii.**Auto**-tilassa (oletus) histogrammiin perustuva taustasäädin valitsee tavoitearvon itse ja pitää valotuksen 60–75 %:ssa anturin maksimiarvosta. Arvon kirjoittaminen tai histogrammin oranssin viivan vetäminen vaihtaa tilan**Manuaaliseen**. |
| **AE-tasoitus** | 0,5–40, askel 0,1 | 8,0 | Kaikki | AE-vaimennus. Työkaluvinkki: ”Pienempi arvo = AE reagoi nopeammin (voi pulsoida korkeilla kuvataajuuksilla). Suurempi arvo = tasaisempi / hitaampi.” Oletusarvoa huomattavasti pienemmät arvot voivat aiheuttaa AE:n pulsointia ja horjuttaa suoratoistoa korkeilla kuvataajuuksilla; 8,0 on vakaa oletusarvo. |
| **AE:n kohdealue**| Valintaruutu ”Ota käyttöön” +**Aim**-painike | Pois | Kaikki | Kun tämä on päällä, AE mittaa vain viivalla merkittyä vihreää aluetta koko kehyksen sijaan.**Tähtää**-toiminto aktivoi napsauttamalla sijoittamisen suorassa lähetyksessä: napsautus keskittää alueen 30 %:iin kehyksestä; napsauta ja vedä piirtääksesi mukautetun suorakulmion (vähintään 5 % × 5 %). Tähtää-toiminto poistuu käytöstä yhden sijoituksen jälkeen. Alue kartoitetaan takaisin kameran omille koordinaateille asettamasi kierto- tai peilausasetusten mukaisesti, ja se tallennetaan projektin mukana. |
| **AE-säätönopeus** | 0,1–5, askel 0,1 | 1,0 | Vain Array-jäsenille | Kuinka nopeasti automaattisen AE-kohteen seuranta reagoi kohtauksen kirkkauden muutoksiin; 1,0× tarkistaa tilanteen uudelleen 2,5 sekunnin välein. |
| **Kirkkaiden alueiden suojaus** | Tiukka (1 %) / Normaali (5 %) / Väljä (15 %) | Tiukka | Kamerat, joissa tämä asetus on käytettävissä | Kuinka suuri osa kuvasta saa leikkautua valkoiseksi, ennen kuin automaattinen valotuksen säätö tummentaa kuvaa. |

{% hint style="info" %}
**Valaistusvaatimus Bayer-monispektrikameroille (RGN / OCN / NGB):** kuvauskohteessa on oltava riittävästi valoa kaikissa kolmessa kanavassa, muuten kalibrointi ei toimi oikein — yksi aistimen valotus kattaa kaikki kolme spektriä. Mittaa valoa DAQ-valosensorilla tai siirry kokonaan monokromaattiseen tilaan (M3M) -tilaan, jolloin jokaisella kaistalla on oma valotuksensa. Jos kuvaus rikkoo tätä sääntöä, Chloros havaitsee sen ja varoittaa sinua (unmix-clamp-ilmoitus).
{% endhint %}

### Pikselimuoto ja

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

resoluutio**Array-jäsenet** näyttävät vain luku -tilassa olevat ”Current” (muoto + WxH) ja ”Binning” -rivit, joissa on huomautus ”Set in array settings” — streamin uudelleenkäynnistys yhdessä jäsenessä rikkoisi synkronoinnin, joten näitä hallitaan [array-asetuspaneelissa](#array-settings-pane).**Erilliskameroissa** on seuraavat asetukset:

| Ohjaus | Vaihtoehdot | Toiminto |
| --- | --- | --- |
| **Pikseliformaatti** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Anturin pikseliformaatti (bittisyvyys). |
| **Tarkkuus** | Täysi / Puoli / Neljännes | Suhteessa nykyiseen binningiin: Täysi = 2048/N × 1536/N N×N-binningissä. |
| **Pikselien yhdistely** | 1x1 (ei yhdistelyä) / 2x2 / 4x4 | Laitteistopohjainen N×N-pikselien yhdistely — suuremmat arvot pienentävät resoluutiota, mutta parantavat signaali-kohinasuhdetta (SNR) ja kuvanopeutta. Asetuksen muuttaminen käynnistää videovirran uudelleen ja nollaa kaikki kiinnostusalueet (ROI) uuteen täyteen näkökenttään. |
| **Nykyinen** | vain luku -tila | Voimassa olevat todelliset WxH-arvot ja (x, y)-siirtymä. |

### Live-esikatselu

Kaikki tässä osiossa on **vain näyttöpuolen asetuksia**— se muuttaa sitä, mitä näet suorassa lähetyksessä, kun taas tallennetut kuvat pysyvät lineaarisina ja muuttumattomina — yhdellä poikkeuksella:**Vignette** on radiometrinen ja vaikuttaa myös vientiin (selostettu alla).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->

| Säädin | Alue / vaihtoehdot | Oletus | Koskee | Toiminto |
| --- | --- | --- | --- | --- |
| **Renderöintitarkkuus** | 360p (nopein) / 480p / 720p / 1080p / Alkuperäinen anturitarkkuus (hitain) | 720p | Kaikki | Korkeus, jolla taustaprosessi suorittaa radiometrisen esikatseluketjun. Pienemmällä arvolla saavutetaan parempi kuvataajuus muuttamatta näkökenttää. |
| **Indeksi**| Valintaruutu + ratas | Pois | Vain Bayer-monispektrikuvat,**ei** yhdistettyjen anturijärjestelmien kuvia | Reaaliaikainen kasvillisuusindeksin esikatselu. Ratas avaa jaetun [Indeksilaskurin](#index-calculator-pane), johon on esiladattu kameran suodattimien luonnolliset kaistat (esim.esim. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Mukautettu lauseke ja LUT (päällä/pois, oletustaso 3, oletusminimi 0,2, oletusmaksimi 1) lasketaan jokaisessa esikatselukuvassa. Yhdistettyjen taulukoiden jäsenet piilottavat tämän rivin — taulukolla on yksi jaettu indeksi. |
| **Valkotasapaino** | Pois / Kerta / Jatkuva + uudelleenmittauspainike | Jatkuva | Vain RGB | Reaaliaikainen valkotasapaino. Päivitä-painike mittaa valkotasapainon uudelleen nykyisestä DLS-spektristä (pois käytöstä, kun tila on Pois). |
| **Gamma** | Päällä / Pois | Päällä | Vain RGB | Näyttää gaman (γ = 2,2 LUT) reaaliaikaisessa esikatselussa. Tallennetut kuvat pysyvät lineaarisina. |
| **Kohinanpoisto** | Valintaruutu + voimakkuus 0–100 | Pois / 50 | Kaikki (kamera-kohtaisesti, myös ryhmissä) | Bilateraalinen suodatin reaaliaikaisessa esikatselussa. Suurempi arvo = tasaisempi, mutta pehmeämmät yksityiskohdat. |
| **Terävyys** | Valintaruutu + voimakkuus 0–100 | Pois / 30 | Kaikki | Unsharp-maski reaaliaikaisessa esikatselussa, sovelletaan viimeisenä. Voi vahvistaa kohinaa. Vain esikatselussa. |
| **Vignette**| Valintaruutu + voimakkuus 0–100 | Pois / 0 | Kaikki | Manuaalinen jäännösvignetoinnin poisto (kirkastaa kulmia), kerrostettu ryhmän Smart Vignette -arvion päälle.**Radiometrinen — vaikuttaa reaaliaikaiseen näkymään JA vientiin**, toisin kuin kohinanpoisto/terävyys. |
| **Väriprofiili** | Raw / Lineaarinen / Luonnollinen / Parannettu / Mukautettu lämpötila | Luonnollinen | Vain RGB | Katso alla. |
| **Värilämpötila** | 2000–10000 K, askel 100 | 5500 K | Vain RGB, mukautettu lämpötilaprofiili | Kiinnittää valkotasapainon kiinteään korreloituun värilämpötilaan (DLS-syöttöä ei huomioida). Viimeksi valittu Kelvin-arvo tallentuu profiilinvaihtojen yli. |
| **Kylläisyys** | 0–200 (100 = neutraali) | 100 | Vain RGB | HSV-kylläisyys reaaliaikaisessa esikatselussa. |
| **Kontrasti** | 0–200 (100 = neutraali) | 100 | Vain RGB | Lineaarinen kontrasti keskiharmaan ympärillä reaaliaikaisessa esikatselussa. |
| **Peilaa H / Peilaa V** | Valintaruudut | Pois | Kaikki | Kääntää esikatselua vaakasuoraan / pystysuoraan. |
| **Kierto**| 0° / 90° / 180° / 270° | 0° | Kaikki | Kierrä esikatselua. Suunta sovelletaan taustaprosessiketjun lopussa —**tallennetut kuvat säilyttävät kameran alkuperäisen suunnan**, ja matriisikoostetut näkymät eivät ota sitä huomioon. |**Väriprofiilin semantiikka** (RGB-kamerat):

* **Raw** — ohita käsittelyketju kokonaan.
* **Lineaarinen** — tummasignaali + tasokenttä + valkotasapaino; ei värimatriisia, ei gammaa.
* **Luonnollinen** *(oletus)* — lineaarinen plus mitattu värinkorjausmatriisi ja kohtaukseen sopeutuva sävykäyrä.
* **Parannettu**— Luonnollinen sekä värikylläisyys ja CLAHE-paikalliskontrasti. Lisäkustannus koskee**vain reaaliaikaista esikatselua** — tallennetut kuvat saavat aina täyden viimeistelyn profiilista riippumatta.
* **Mukautettu lämpötila** — Luonnollinen, jossa valkotasapaino on kiinnitetty valitsemaasi Kelvin-arvoon.

{% hint style="warning" %}
Luonnollinen-, Parannettu- ja Mukautettu lämpötila -asetuksissa paneelissa näkyy sävyhuomautus: kuvia kirkastetaan kunkin kohtauksen mukaan, joten tallennettuja *näyttökuvia* ei voi verrata keskenään. **Vie säteilyvoimakkuus tai heijastavuus mittauksia varten.**
{% endhint %}

### Näytön päällekkäiset kerrokset (piirretään reaaliaikaisen syötteen päälle)

Nämä ovat käytettävissä vain käyttöliittymässä — ne piirretään videon päälle, eivätkä ne koskaan vaikuta suoratoistoon tai tallennettuihin kuviin.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Päällekkäinen kerros | Säätimet | Oletus | Toiminto |
| --- | --- | --- | --- |
| **Zebra** | Valintaruutu + kynnysarvo 200–255 | Pois / 250 | Magentanväriset diagonaaliset raidat leikkautuneissa pikseleissä. |
| **Ristikohta** | Valintaruutu | Pois | Kehyksen keskipistemerkki. |
| **Ruudukko** | Pois / 3 × 3 / 9 × 9 | Pois | Sommitteluruudukko. |
| **Histogrammi** | Valintaruutu + leveys 0,10–0,90 kuvan leveydestä | Pois / 0,25 | Kuvavirtaan näkyvä histogrammikaistale. |
| **Tarkennushuippu** | Valintaruutu + kynnysarvo 20–200 + värinäyte | Pois / 80 / `#ff5722` | Sobel-reunakorostus tarkennusta varten. |
| **Kanavajaot** | &quot;Näytä jaot (Red / Green / NIR)&quot; / &quot;Piilota jaot&quot; -painike | Piilotettu | Lisää kolme kanavakohtaista, toisistaan riippumatonta harmaasävyruutua yhdistelmäkuvan viereen (painikkeen teksti noudattaa kameran suodatinkanavia). Kutakin jaettua ruutua voi vetää, ja se käyttää kameran reunusväriä. Ei käytettävissä mustavalkokameroissa. Tallennetaan projektin mukana. |

### Pistemittari

* **Napsauta näytteenottoa varten**-valintaruutu: napsauta reaaliaikaista kuvaa yhden pikselin näytteenottoa varten (ristikohdistin merkitsee sen) tai napsauta ja vedä aluetta pikselien keskiarvon laskemiseksi.**Tyhjennä**poistaa näytteen ja kohdistimen. Ei voi olla käytössä samanaikaisesti AE-ROI**Aim** -tilan kanssa.
* **Näytä**-pudotusvalikko:**Raw (bittisyvyys)**— alkuperäiset digitaaliset arvot anturin bittisyvyydellä (esim. 12-bittinen → 0..4095) — tai**Näyttö (8-bittinen)** (oletus). Kun reaaliaikainen indeksi on aktiivinen, Näyttö-valikossa näkyy sen sijaan laskettu indeksiarvo (esim. NDVI).
* Lukupaneelissa näkyvät pikselikoordinaatit, kuvakoko, pikselimuoto, bittisyvyys sekä kanavataulukko (Chan / Arvo / %) kaistamerkinnöillä ja aallonpituuksilla; Bayer-vihreät parit on keskiarvoistettu; alueiden näytteissä näkyy ”N px avg”.

Pistemittarin tila on voimassa vain kyseisen istunnon ajan.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Ennakoiva automaattinen valotus (DLS-ohjattu)

Tämä osio näkyy vain, kun **vähintään yksi DAQ-valosensori on kytketty** — ratkaisija tarvitsee reaaliaikaista alaspäin suuntautuvaa spektriä toimiakseen.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->

| Ohjaus | Alue | Oletus | Toiminto |
| --- | --- | --- | --- |
| **Ota käyttöön** | Valintaruutu | Päällä (erilliset kamerat) | Suljetun muodon ratkaisija käyttää DLS-spektriä sekä kameran kalibrointipaketin skalaareja saadakseen kirkkaimman kaistan lähelle kyllästymistä ja pitäen samalla himmeimmän kaistan SNR-rajan yläpuolella — yksi valotuksen tallennus ratkaisua kohti, ei vakautuspiiriä. Suunniteltu aurinkovoimalla toimiviin aikavälikuvauksiin, joissa jokaisen kuvan valotuksen on oltava oikea. Taustaprosessi siirtyy huomaamattomasti reaktiiviseen automaattiseen valotukseen aina, kun DLS-lukema on vanhentunut tai puuttuu tai kalibrointipakettia ei ole ladattu. |
| **Tasoitus (α)** | 0,05–1,0, askel 0,05 | 0,3 | Peräkkäisten ennustavien ratkaisujen tasoitus (pienempi arvo = tasaisempi). |
| **Kohteen heijastavuus**|**Kalibroi ρ uudelleen**-painike | — | Arvioi uudelleen ratkaisijan käyttämän kohteen heijastavuuskertoimen. |

{% hint style="info" %}
**Array connect poistaa ennakoivan automaattisen valotuksen käytöstä oletusarvoisesti** — matriisikuvauksessa Chloros:n älykäs automaattinen valotus sekä kameran oma automaattinen valotus hoitavat valotuksen (kylläisyyssuojauksella), eikä ennakoivan automaattisen valotuksen yksittäinen kohtauksen heijastavuusarvio ole turvallinen sekakohtauksissa. Voit ottaa sen uudelleen käyttöön kamerakohtaisesti tässä, jos haluat nimenomaan DLS-ohjattua radiometristä valotusta.
{% endhint %}

**DAQ-ohjattu valotuksen yläraja ja saapuvan valon mukaan kiinnitetty automaattinen valotus.**Edellä mainitusta valintaruudusta riippumatta, kun DAQ-valosensori on määritetty RGB-kameraan, Chloros laskee — mitatusta absoluuttisesta alaspäin suuntautuvasta säteilyvoimakkuudesta — sen suurimman valotuksen×vahvistuksen, jolla 100 %:n heijastavuuden omaava pinta pysyy leikkautumisen alapuolella, ja soveltaa sitä**ylärajana**automaattisessa valotuksessa. Kun yläraja on aktiivinen, kamera toimii**tulevan valon mukaan sidottuna**: se toimii avoimessa piirissä saapuvan valon mukaan mitatulla valotuksella ja vahvistuksella 0 dB — valotus seuraa mitattua valoa, ei kuvauskohteen sisältöä. Koska yläraja voi vain lyhentää valotusaikaa, se ei voi itsessään aiheuttaa leikkautumista. Yläraja kytkeytyy pois päältä automaattisesti — ja normaali kohtauksen automaattinen valotus (AE) jatkuu — aina, kun DAQ-lukema puuttuu, on vanhentunut (&gt;30 s) tai tumma, tai jos ≥15 % kuvasta leikkautuu kiinnitetyllä valotuksella (eli anturi ja kamera havaitsevat eri valaistuksen). Tälle ei ole GUI-kytkintä; tämä on vakiokäyttäytyminen aina, kun RGB-kameralla on DAQ-yhteys.

### Acquisition- ja Trigger

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

Array -jäsenet näyttävät lisäksi vain luku -muotoiset **Rooli**(Master sinisellä / Slave vihreällä),**Synkronointilinja**ja**Vertaiset**-rivit.

| Ohjaus | Vaihtoehdot | Oletus | Huomautukset |
| --- | --- | --- | --- |
| **Laukaisutila** | Pois / Päällä | Päällä | Poissa käytöstä matriisin jäsenille (ryhmä hallitsee laukaisua). |
| **Laukaisulähde** | Ohjelmisto / Linja 0 (M8) / Linja 1 / Linja 2 | Linja 0 | Piilotettu, kun laukaisutila on Pois; poissa käytöstä ryhmän jäsenille. Line0 on M8:n optoeristetty ulkoinen laukaisutulo. |
| **Kuvataajuus**| Auto / Manuaalinen + arvo | Auto |**Auto**: kameran kuvataajuuden rajoitus on pois päältä — valotus määrää kuvataajuuden (fps), ja ruudussa näkyy reaaliaikainen todellinen taajuus.**Manuaalinen**: rajoitat kuvataajuuden liukusäätimellä (1–kaistanleveyden rajoittama enimmäisarvo), joka perustuu nykyiseen todelliseen kuvataajuuteen. Ryhmän jäsenille näkyy vain luku -muotoinen ”N fps (reaaliaikainen)” ja teksti ”Asetettu ryhmäasetuksissa”. |

### Verkko / Siirto

| Rivi | Toiminta |
| --- | --- |
| **Paketin koko**| 1500 (Vakio) / 9000 (Jumbo) — oletusarvo**Jumbo**. |
| **Siirtonopeus** | Vain luku -muotoinen linkin siirtonopeuden raja MB/s. Taustapalvelin tasapainottaa tämän kaikkien kytkettyjen kameroiden kesken jokaisen yhteyden muodostamisen tai katkaisemisen yhteydessä. |
| **Puskurin käsittely** | Vain luku -tilan puskurin käsittelytila. |

### Tallennus

Paneeli päättyy **&quot;Avaa tallennusasetukset…&quot;** -painikkeeseen, joka vie [Tallennusasetukset-paneeliin](capture.md#the-capture-settings-pane) (pois käytöstä, kunnes projekti on avattu — &quot;Luo tai avaa projekti tallennusten tallentamiseksi&quot;). Jos kamera on piilotettu tai keskeytetty, vihje muistuttaa sinua palauttamaan näkyvyyden tai jatkamaan ennen tallennusta.

## Ryhmäasetukset-paneeli

Avaa painamalla **hammasrataskuvaketta**ARRAY-rivillä. Otsikko: ryhmän nimi, jonka vieressä on nimeämiskynä sekä**×** sulkemista varten. Alla *vain yhdistetty* -merkinnällä varustetut osiot näkyvät vain yhdistetyllä näyttötilalla kytketyissä matriiseissa.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->

### Synkronointi

Vain luku -muotoiset **Master**-,**Slaves**- ja**Sync Line** -rivit.

### Ympäristön valosensori

Näkyy sekä yhdistetyissä että erillisissä matriiseissa:

* **Kalibrointikohde** -valintaruutu — ”Tunnista MAPIR ArUco -kohde ja vahvista NDVI paneelin heijastavuus-LUT:n suhteen”; ohjaa yhdistettyjen ruutujen kohdepeittokuvaa ja vahvistustaulukkoa.
* **Valosensori**-pudotusvalikko — liittää yhden DAQ:n koko ryhmään. Valinta astuu voimaan välittömästi, välittyy jokaisen ryhmään kuuluvan kameran omaan Valosensori-pudotusvalikkoon (voit silti ohittaa valinnan kamerakohtaisesti) ja alkaa välittää spektrejä ryhmälle.
* Reaaliaikainen **Tila**-rivi: Pois · &quot;Odotetaan ensimmäistä spektriä…&quot; · ”Aktiivinen — kaikkien ryhmän kameroiden valaistus on korjattu” · ”Ei uutta spektriä viimeisten 3 sekunnin aikana — käytetään edelleen viimeistä lukemaa (ei vanhentunut aikakatkaisua)…”.
* Huomautus ruudussa: ”Ryhmänlaajuinen radiometrinen korjaus. Kamerakohtaiset asetukset ohittavat tämän.”

### Tallennus — yhtenäiset anturiasetukset *(vain yhdistettynä)*

Nämä asetukset koskevat yhtenäisesti jokaista jäsentä (jäsenkohtaiset muutokset rikkoisivat synkronoinnin). Muokkaukset valmistellaan ja otetaan käyttöön yhdessä.

<!-- SCREENSHOT-NEEDED: array settings Capture section — Pixel Format, Binning, Resolution preset, the ROI crop W/H/X/Y fields with the "max WxH" hint and Reset button, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons; ideally with the live orange crop-preview box visible on the array tile. -->

| Ohjaus | Vaihtoehdot / alue | Toiminto |
| --- | --- | --- |
| **Pikselimuoto** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Yhtenäinen anturimuoto kaikille jäsenille. |
| **Pikselien yhdistely** | 1x1 / 2x2 / 4x4 | Laitteistopohjainen pikselien yhdistely — säilyttää koko näkökentän ja parantaa samalla signaali-kohinasuhdetta (SNR) sekä kuvataajuutta. Asetuksen muuttaminen nollaa ROI-kentät uuteen täyteen näkökenttään. |
| **Resoluutio**-esiasetus | Täysi / Puoli / Neljännes | Suhteutettu binningiin; täyttää ROI-kentät keskitetyllä rajauksella. |
| **ROI-rajaus (px)**| W / H / X / Y-numerokentät | Anturin rajaus. Leveys/korkeus kiinnittyvät 16:n kerrannaisiin (vähintään 64); siirtymät kiinnittyvät 4:n kerrannaisiin. ”max WxH”-vihje näyttää ylärajan ja**Nollaa** palauttaa täyden näkökentän. Muokkauksen aikana matriisiruutuun piirretään oranssi reaaliaikainen rajausesikatseluruutu (mukaan lukien koko anturin kaaviokuva, kun rajausta laajennetaan ulospäin). |
| **Laukaisutaajuus**| Auto / Manuaali-vaihtokytkin + fps 0,5–10, askel 0,5 |**Auto**(oletus): taustaohjelma laskee laukaisutaajuuden resoluution ja kaistanleveyden perusteella — syöttökenttä on pois käytöstä ja siinä näkyy laskettu arvo.**Manuaali**: lukitsee arvon valitun arvon kohdalle Apply-painiketta painettaessa. |

Huomautus ruudussa: ”Muoto- tai resoluutiomuutokset käynnistävät kaikki kamerat hetkellisesti uudelleen. Laukaisutaajuus otetaan käyttöön reaaliaikaisesti.” **Käytä / Peruuta**-painikkeet sijaitsevat ruudun alaosassa.

### Kohdistus (yhteisrekisteröinti) *(vain yhdistettynä)*

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* **Jäännös**-ruutu: ”RMS x,xx px” — vihreä alle 1 px, keltainen alle 3 px, punainen muussa tapauksessa tai jos jokin kamera epäonnistui; ”ei profiilia” ennen ensimmäistä ratkaisua.
* Yhteenvetorivi: ”✓ Kaikki kamerat kohdistettu (N)” / ”⚠ p/N kameraa kohdistettu —  <serial (filter)="">epäonnistui” / ”Rajaaminen aktiivinen — Kalibroi uudelleen kohdistusta varten (käyttää koko anturia)” / ”Odotetaan valotuksen vakiintumista…&quot;.
* Kamerakohtainen taulukko: kamera (sarjanumeron 4 viimeistä merkkiä + suodatin), uudelleenprojektio-virhe px:inä ja vastaavuuksien lukumäärä (&quot;ref&quot; pääkameralle) sekä päällekkäisyyden normalisoitu ristikorrelaatiopisteet suhteessa 0,35:n läpäisyrajaan.
* **Kalibroi kohdistus uudelleen** -painike (teksti ”Kalibroi kohdistus” ennen ensimmäistä profiilia) — suorittaa ko-rekisteröinnin uudelleen uusilla kehyksillä.
* **”Automaattinen valotus kameroille kohdistusta varten”** -valintaruutu (oletusarvoisesti valittuna) — kirkastaa väliaikaisesti tummia tai tasaisia kameroita (ensin valotus, sitten vahvistus), jotta niissä on tekstuuria vastaavuuden löytämiseksi, ja palauttaa sitten automaattisen valotuksen.

Yhdistetty esikatselu kohdistuu automaattisesti avattaessa; kalibroi uudelleen, jos tarkennus tai kuvauskohteen syvyys on muuttunut. Kohdistus on **suunniteltu vain istuntokohtaiseksi** — sitä ei koskaan tallenneta profiiliin, koska se riippuu kyseisen hetken kuvausetäisyydestä. Tallenteet voidaan silti viedä pikselikohtaisesti kohdistettuina (katso [Kohdistetut vientitiedostot](capture.md#per-array-controls)).

### Älykäs vignettikorjaus

* **Ota korjaus käyttöön**-valintaruutu — soveltaa kamerakohtaista vignettiarviota radiometriseen ketjuun (reaaliaikaisesti**ja** vienneissä).
* **Kalibroi nykyisestä näkymästä**— suuntaa ensin kameraryhmä tasaiselle kohteelle (litteä paneeli, seinä tai taivas); jokainen kamera tasoitetaan erikseen ja tilanneraportti ilmoittaa tasoituksen parannuksen muodossa ”n/N kameraa · −x,x %”.**Tyhjennä** poistaa arvion.
* Hienosäädä kameraa kohden kamerakohtaisella **Vignette**-liukusäätimellä [Live-esikatselussa](#live-preview).

### Live-esikatselu *(vain yhdistetty)** **Indeksi**: valintaruutu + ratas — avaa jaetun [Indeksilaskurin](#index-calculator-pane), jossa on**kaikkien** jäsenkameroiden tuottamat kaistat. Sen alla oleva lausekkeen esikatselurivi näyttää nykyisen lausekkeen (&quot;Lauseketta ei ole määritetty — avaa laskuri luodaksesi sellaisen&quot;), joka päivitetään sekunnin välein.
* **Renderöintitarkkuus**-pudotusvalikko (samat esiasetukset kuin kamerakohtaisesti, oletus 720p): live-näkymän virran korkeus**ja** tallennetun yhdistelmän vientikoko. Huomaa paneelissa: ”Esikatselu + tallennetun yhdistelmän koko. Kamerakohtaiset kuvat viedään aina täysresoluutiolla.”

### Näyttökerrokset *(vain yhdistettynä)** **Ota käyttöön** -valintaruutu (oletusarvo pois päältä — pääkamera näkyy suoraan; päällä = kerrostettu yhdistelmä).
* **Etuala**/**Tausta**-pudotusvalikot: kukin jäsenkamera (nimen mukaan) tai**Indeksi**. Kun Etualalla on valittuna Indeksi, LUT:n minimi- ja maksimiarvojen ulkopuolella olevat pikselit näyttävät Taustakerroksen.

### Jaettu näkymä *(vain yhdistetty)*

**”Näytä jäsenkamerat”**—**Jaettu / Piilota jäsenkamerat** -painike, joka lisää kunkin jäsenkameran oman reaaliaikaisen syötteen erillisinä ruudukon ruutuina yhdistelmän viereen. Ruudut lukevat ryhmän olemassa olevan kehyspuskurin (ei ylimääräistä kamerayhteyttä). Vain ruudukkonäkymä; tallennetaan ryhmäkohtaisesti projektin mukana.

### Ominaisuudet

Vain luku -paneeli, joka päivitetään 5 sekunnin välein:

* **Tason nimike**: ”Samanaikainen tallennus” (vihreä) · ”Samanaikainen tallennus (FTD-porrastettu lähetys)” (vihreä) · ”Porrastettu tallennus (100 ms:n viive)” (keltainen) · ”Määritys liian suuri” (punainen).
* **Kuvan tila**: ”x,xx % keskeneräinen” — vihreä alle 1 %, keltainen alle 5 %, punainen 5 % tai enemmän.
* **Yhteyslinja**: ”NIC {mbps} Mbps – jatkuva {MB/s} MB/s”.

Tämä on järjestelmän reaaliaikainen kaistanleveysbudjetti. Taustalla olevasta kuvataajuudesta (fps) ja verkkomallista — sekä siitä, mitä muutoksia on tehtävä, kun taso muuttuu keltaiseksi tai punaiseksi — katso [Monikamerajärjestelmät](arrays.md) ja [CLI-viite](../reference/cli-reference.md).



<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->## Indeksilaskuri-paneeli

Kolmas sivupalkkisivu, jota jakavat kamerakohtainen indeksisäädin ja yhdistettyjen kameraryhmien indeksisäädin (yksi kerrallaan – otsikossa lukee ”Indeksilaskuri — <camera name="">” tai ”Indeksilaskuri —<array name="">

&quot;). Se vastaanottaa kaistaluettelon (kameran suodattimen luonnolliset kaistat tai kaikki kaistat koko ryhmän jäseniltä), nykyisen lausekkeen ja LUT-määritykset (päällä/pois, taso — oletus 3, min – oletusarvo 0,2, max – oletusarvo 1) sekä reaaliaikaisen indeksi-histogrammin. **Käytä**-painike vahvistaa lausekkeen; LUT-muutokset näkyvät reaaliaikaisesti esikatselussa.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Kamerakohtaiset vs. ryhmän hallinnoimat asetukset

Pikaopas siitä, mitkä asetukset sijaitsevat missä, kun kamera on ryhmän jäsen:

| Ryhmän hallinnoimat (vain luku -tila kamerapaneelissa) | Edelleen kamerakohtaiset ryhmän sisällä |
| --- | --- |
| Pikselimuoto, resoluutio, binning | Automaattinen valotus (valotus, vahvistus, kohde, tasoitus, ROI) |
| Laukaisutila/lähde, kuvataajuus | Kohinanpoisto, terävyys, vinjetti |
| | Suunta (peilaus/kierto), näytön päällekkäisnäytöt, pistevalotus |
| | Indeksi (erillisnäyttöiset ryhmät), valosensorin sidonta |

Muita kokonaisuutta koskevia toimintoja:

* **Yhdistetty vs. erillinen näyttö** valitaan ryhmän liitännän yhteydessä: yhdistetty = yksi kohdistettu yhdistelmäruutu (jäsenet lähettävät syötteitä vain Split View -näkymän kautta); erillinen = kukin jäsen renderöi oman synkronoidun ruutunsa. Kamera ei koskaan näytä sekä erillistä syötettä että ryhmän ruutua.
* **Automaattinen uudelleenkytkentä**: tallennetun projektin avaaminen palauttaa sen kamerat ja ryhmät ja soveltaa kaikki tallennetut asetukset uudelleen taustapalvelimeen ennen kuin lähetykset jatkuvat.
* **Tallennuksen rajoitus**: piilotetut tai keskeytetyt kamerat jätetään pois Capture All -toiminnosta; matriisi estetään kokonaan vasta, kun KAIKKI jäsenet ovat piilotettuja tai keskeytettyjä. Katso [Tallennusasetukset ja -tilat](capture.md).

## Miten asetukset säilyvät

Kameravälilehden tila tallennetaan **projektin mukana**, ei selaimeen:

* Jokainen reaktiivinen muutos tallentaa kamerat ja ryhmät projektin `cameras.json` (viive 500 ms). Tämä kattaa kameranimet ja värit, valotus-/vahvistus-/AE-asetukset, pikselimuodon/resoluution/binningin, laukaisunopeuden, esikatseluasetukset (renderöintiresoluutio, kohinanpoisto, terävyys, vinjetti, väriprofiili, kylläisyys/kontrasti), suunnan, päällekkäisnäytöt, kanavajaot, indeksin konfiguraation, ennakoivan automaattisen valotuksen (predictive-AE) asetukset, automaattisen valotuksen ROI:n, matriisin nimet, näyttötilan, matriisin tallennusasetukset (mukaan lukien ROI:n rajausasento) sekä ruudukon (syötteen zoomaus, katselutila, ruudukon lukitus, manuaalinen ruutujen järjestys, piilotetut kamerat, suljetut ruudut, aktiivinen kamera).
* Valosensorien sidokset tallennetaan projektin tiedostoon `sensors.json`.
* Projektin avaaminen uudelleen yhdistää laitteiston uudelleen ja ottaa kaikki asetukset uudelleen käyttöön.
* **Ei avointa projektia = vain istunto**: ilman projektia mikään ei säily Chloros-ohjelman sulkemisen jälkeen.
* Vain istunto riippumatta projektista: taukotila, pistevalotusnäytteet, kamerakohtainen Kalibrointikohde-valintaruutu (avautuu aina pois päältä) ja matriisin kohdistusprofiili (lasketaan uudelleen istuntokohtaisesti suunnittelun mukaisesti).
* Yksi poikkeus: **Kuvausasetukset**-vientiä koskevat valinnat ja kuvausmoodi säilyvät projektikohtaisesti sovelluksen paikallisessa tallennustilassa eikä `cameras.json`:ssä — katso [Kuvausasetukset ja -moodit](capture.md).</array></camera></serial>
