# Monikamerajärjestelmät

LATTICE-**järjestelmä**koostuu kahdesta tai useammasta LATTICE-kamerasta, jotka on kytketty yhteen synkronoiduksi kokonaisuudeksi. Yksi kamera toimii**pääkamerana**: se lähettää laitteistopohjaisen GPIO-laukaisupulssin jaetulle synkronointilinjalle (oletuksena**Line2**), jolloin kaikki kamerat tallentavat saman hetken. Chloros lisää PTP-aikasynkronoinnin, reaaliaikaisen esikatselun (kamerakohtaiset ruudut tai yksi kohdistettu monikaistainen yhdistelmä) sekä synkronoidun kuvauksen — jokainen kuvauskerta tuottaa yhden**kuvaryhmän**, jossa kaikilla kameroilla on sama aikaleima ja kuvatunnus (raportoidaan muodossa `fid:N` kuvausulostulossa).

Mono- (M3M) kamerat tuottavat kasvillisuusindeksejä kameraryhmien avulla — yksi kamera tuottaa yhden kaistan, ja ryhmä kohdistaa ne monikaistaiseksi pinoon. Katso [Mono-kamerat ja kasvillisuusindeksit](mono-indices.md).

Matriisin liittämiseen on kolme vastaavaa tapaa, ja niissä kaikissa suoritetaan sama ”smart-prep”-prosessi:

| Pinta | Aloituskohta |
| --- | --- |
| Käyttöliittymä | Kamerat-välilehti → **Yhdistä kameraryhmä** (sininen painike) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (ensimmäinen sarjanumero = päälaite) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (ensimmäinen sarjanumero = päälaite) |

Smart-prep suorittaa järjestyksessä seuraavat toimet: verkkokelpoisuustestin (ICMP DF-ping + GVSP-testi), synkronointitason valinnan, kehyksen koon automaattisen pienentämisen kaapelille sopivaksi, PTP:n käyttöönoton, kamerakohtaisen pikselimuodon automaattisen valinnan, automaattisen valotuksen alustuksen kunkin kameran tallennetusta tilasta sekä GPIO-laukaisun konfiguroinnin linjalla 2.

{% hint style="info" %}
Kameroiden on oltava tavoitettavissa linkillä, ennen kuin mikään näistä toiminnoista toimii — katso [Kameroiden liittäminen](connecting.md) laitteiden tunnistuksen, osoitteistuksen ja ensimmäisen yhteyden kalibrointitiedoston lataamisen osalta. Monikamerajärjestelmissä isäntäkoneen verkkokortin vastaanottorengasasetukset ovat yhtä tärkeitä kuin linkin nopeus; täydellinen oire→korjaus-taulukko löytyy kohdasta [CLI Viite § Isäntäkoneen verkkokortin asetukset ja säätö](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Array Connect -valintaikkuna

Kamerat-välilehti → **Yhdistä ryhmä**avaa kolmivaiheisen ohjatun toiminnon:**Valitse → Näyttötila → Asetukset**.

### Vaihe 1 — Valitse pääkamera ja alikamerat

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered. Table showing Camera / Serial / IP / Master radio / Slave checkbox columns, with the green "GPIO master detected — selections pre-populated" probe banner visible above the table. -->

Valintaikkuna skannaa verkon heti avautuessaan (&quot;Skannataan verkkoa...&quot;), minkä jälkeen se tarkistaa GPIO-laukaisun johdotuksen (&quot;Tarkistetaan GPIO-johdotusta...&quot;). Ryhmän muodostamiseen tarvitaan vähintään **2 kameraa**.

Johdotuksen tarkistus täyttää roolivalinnat automaattisesti, kun se on mahdollista, ja näyttää yhden kolmesta ilmoituksesta:

| Ilmoitus | Merkitys |
| --- | --- |
| &quot;GPIO-päälaite havaittu — valinnat esitäytetty&quot; (vihreä) | Tarkistus löysi laukaisutopologian; päälaitteen ja alilaitteiden valintaruudut on jo täytetty. |
| &quot;Päälaitetta ei havaittu – tarkista GPIO-kaapeli&quot; (oranssi) | Mikään kamera ei havainnut laukaisupulssia; tarkista synkronointikaapelointi. Voit silti valita roolit manuaalisesti. |
| &quot;Ei synkronointikaapelia: {sarjanumerot}&quot; (oranssi) | Luettelossa oleviin kameroihin ei ole kytketty synkronointikaapelia. |

Kamerataulukossa on sarakkeet **Kamera / Sarjanumero / IP / Master (radio) / Slave (valintaruutu)**:

* Valitse tarkalleen **yksi master**ja**yksi tai useampi slave**. Napsauttamalla nykyisen masterin radiota uudelleen poistat valinnan.
* Kameraa, jonka kohdalla on merkintä **&quot;Ei synkronointikaapelia&quot;**, ei voi koskaan valita orjaksi — orja, jolla ei ole laukaisukaapeloitusta, odottaisi synkronointilinjalla ikuisesti ja toimittaisi tyhjän kuvan. Liitä kyseinen kamera sen sijaan erilliskamerana.
* Jo erillisinä kytketyt kamerat *eivät* poistu käytöstä: ryhmäkytkentä vapauttaa erillisen istunnon ja avaa kameran uudelleen ryhmän sisällä.

**Seuraava: Näyttötila →**aktivoituu, kun päälaite ja vähintään yksi alilaite on valittu.**Skannaa uudelleen** suorittaa laitteiden tunnistuksen ja johdotuksen tarkistuksen uudelleen.

{% hint style="warning" %}
**Peruuta**-painike on pois käytöstä, kun skannaus tai johdotustarkistus on käynnissä — tarkistuksen keskeyttäminen kesken voi kaataa kameran SDK LATTICE-kameran laiteohjelmistossa. Odota, kunnes pyörivä kuvake lakkaa pyörimästä.
{% endhint %}

### Vaihe 2 — Näyttötila | Tila

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

| Mitä saat |
| --- | --- |
| **Erilliset kamerat** | Yksi reaaliaikainen ruutu kameraa kohden, kaikki laukeavat yhdessä, jotta kuvat pysyvät synkronoituna. Jokainen kamera säilyttää oman värinsä ja asetuksensa. |
| **Yhdistetyt kamerat** *(oletus)* | Yksi ruutu, joka näyttää kohdistetun monikaistaisen NDVI/index-yhdistelmän. Kamerat jakavat matriisin värin. |

Näyttötila muuttaa vain reaaliaikaisen esikatselun ulkoasua — kuvauskäyttäytyminen on sama molemmissa.

### Vaihe 3 — Ryhmän asetukset ja ennustettu tulos

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Pin resolution / Trigger Rate controls, right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

Kun siirryt tähän näkymään, Chloros pyytää taustapalvelimelta **suositusta**ja soveltaa automaattisesti ROI- ja binning-yhdistelmän, joka sopii NIC:n vastaanottorenkaaseen (se suosii binningiä ROI-rajauksen sijaan, koska binning säilyttää koko näkökentän). Jokainen tekemäsi muutos suorittaa analyysin uudelleen reaaliajassa ja päivittää oikealla olevan**Ennustettu tulos** -paneelin.

Vasen sarake — asetukset:

| Ohjaus | Vaihtoehdot | Oletus | Huomautukset |
| --- | --- | --- | --- |
| **ROI (näköala)** | Täysi (2048×1536) / Puoli (1024×768) / Neljännes (512×384) | Täysi | Anturin rajaus: Puoli/neljännes rajataan pienempään alueeseen alkuperäisellä pikselivälillä. |
| **Binning** | 1× / 2× (summa 2×2) / 4× (summa 4×4) | 1× | Laitteistopohjainen binning: 2×2 = täysi FoV neljänneksellä siirtokaistan kustannuksista; 4×4 = täysi FoV 1/16:lla. Piilotettu, jos kamerat eivät tue binningiä. |
| **Verkon puolella oleva kuva** (lukeminen) | — | — | Binningin jälkeen verkon kautta tosiasiallisesti lähetetty leveys × korkeus, pyöristettynä 16:n kerrannaisiksi (vähintään 64). |
| **Pin-resoluutio**| valintaruutu | pois päältä | Chloros nostaa binning-tasoa yleensä automaattisesti yhteyden muodostamisen yhteydessä, kun ennustettu kuvataajuus laskee alle**1,5 fps**. Pinning säilyttää valitsemasi kuvakokoisen ja hyväksyy alhaisemman siirtonopeuden — ja muuttaa ylikuormitetun kokoonpanon automaattisen siirtonopeuden alennuksen sijaan kovan yhteyden muodostamisen epäämiseksi. |
| **Laukaisunopeus** | 0,5–60 fps, askel 0,1 | tyhjä = auto | Master-laitteen laukaisunopeus. Jätä tyhjäksi, jotta Chloros voi laskea sen. |
| **Kaistanleveys**| 20–2000 MB/s, askel 10 | tyhjä = auto | Kuinka paljon isäntä voi tosiasiallisesti käsitellä, MB/s:na —**se yksi luku, josta koko matriisin allokointi riippuu.** Tunnistetaan automaattisesti verkkosovittimesta. Laske arvoa, jos ryhmä ilmoittaa vioittuneista kehyksistä: havaittu arvo yliarvioi USB-sovittimien ja jaettujen kytkinten suorituskykyä. Arvon muuttaminen suorittaa ennusteen uudelleen reaaliajassa. |

Oikea sarake — **Ennustettu tulos**:

* **Synkronointitaso** — ”Samanaikainen tallennus” (vihreä), ”Samanaikainen tallennus (FTD-porrastettu lähetys)” (vihreä), ”Porrastettu tallennus (100 ms:n viive)” (keltainen) tai ”Määritys liian suuri” (punainen).
* **fps-ennuste** — esitetään vaihteluvälinä (&quot;himmeä → kirkas&quot;), koska synkronoidun matriisin nopeus määräytyy hitain kameran valotuksen mukaan.
* **NIC-rivi** — linkin nopeus ja jatkuva siirtonopeus (&quot;NIC {mbps} Mbps · jatkuva {N} MB/s&quot;).
* **Sim-emit-purkaustarkistus** — pystyykö isäntäkoneen NIC-renkaan vastaanottokapasiteetti käsittelemään yhden samanaikaisen purkauksen kaikilta kameroilta (&quot;Sim-emit-purkaus: X MB · käytettävissä oleva NIC-renkaan kapasiteetti: Y MB ✓/✗&quot;).
* **Kaapelibudjetin tarkistus** — vakaassa tilassa oleva kokonaiskysyntä suhteessa törmäyksiltä suojaavaan kaapelirajaan (&quot;Kaapelibudjetti: {kysyntä} MB/s, jota vaatii {n} kameraa · yläraja {yläraja} MB/s ✓/✗ ylimerkitty&quot;).
* **&quot;Tämän kaistan enimmäiskameramäärä: {n} — määritetään kamerakohtaisen kaistanleveyden alarajan perusteella, joten ryhmittely ei nosta sitä.&quot;** — näkyy, kun olet lähellä (tai yli) kameramäärän ylärajaa.
* **&quot;KUVAT MENETETÄÄN näillä asetuksilla.&quot;**— punainen varoitus, jossa ilmoitetaan taustajärjestelmän syy, sekä estolista ja siniset**korjausehdotukset** (&quot;Jotta tämä ryhmä mahtuu verkkoon&quot; / &quot;Samanaikaisen tallennuksen mahdollistamiseksi&quot;).**Käytä ja yhdistä** -painiketta ei voi käyttää, ennen kuin ennuste on valmis, ja sen teksti kertoo, miksi se hylätään:

| Painikkeen teksti | Merkitys | Mikä todella auttaa |
| --- | --- | --- |
| &quot;Analysoidaan...&quot; | Analyysi vielä käynnissä. | Odota. |
| **&quot;Liian monta kameraa tälle verkolle&quot;**| Järjestelmä ylikuormittaa verkkoyhteyden (kokonaiskapasiteetin tarkistus epäonnistui). | Vähemmän kameroita, jumbo-kehykset päästä päähän tai nopeampi verkkokortti.**Pienempi ROI EI auta** — katso alla. |
| **&quot;Pienennä ROI:ta, jotta toiminto voidaan ottaa käyttöön&quot;** | Kehykset hävisi näillä asetuksilla (burst/ring-tarkistus epäonnistui). | Pienennä ROI:ta, nosta binning-arvoa tai korjaa verkkokortin vastaanottorengas. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->

Yhdistämisen aikana saattaa ilmestyä vihreä **kalibroinnin latauspaneeli**, jossa on sarjanumeroittain etenevä edistymispalkki: kun kamera liitetään koneeseen ensimmäistä kertaa, Chloros lataa kamerasta noin 3,8 MB:n suuruisen tehdaskalibrointipaketin GigE-yhteyden kautta (noin 70 sekuntia kameraa kohti). Välimuistiin tallennetut kamerat eivät koskaan näytä tätä paneelia. Katso [Kameroiden liittäminen](connecting.md).

## Kaistanleveys: kuinka monta kameraa mahtuu

Se, kuinka monta kameraa ryhmä pystyy käsittelemään, riippuu verkon kapasiteetista, ei Chloros:stä, joten suunnitteluluvut löytyvät laitteiston käyttöoppaasta: **[Ryhmän kaistanleveyden suunnittelu](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Mitä Chloros tekee näillä tiedoilla: liitäntävalintaikkuna suorittaa verkkotestin, ennustaa saavutettavissa olevan kuvataajuuden ja valitsee sopivan tason. Jos ryhmä ylikuormittaa kaapelia, se kieltäytyy muodostamasta yhteyttä sen sijaan, että hylkäisi paketteja hiljaisesti — katso edellä kuvattu ennustetun tuloksen paneeli.

## Kun kehyksiä katoaa

Kamera voi puuttua julkaistusta ryhmästä kahdesta täysin eri syystä,
ja ne vaativat vastakkaisia korjaustoimenpiteitä. Chloros laskee ne erikseen sen sijaan, että ilmoittaisi yhden
”epätäydellisen” luvun, joka ei nimeä kumpaakaan:

| Mitä tapahtui | Mitä se tarkoittaa | Mistä etsiä |
| --- | --- | --- |
| **Vioittunut**— kehys saapui, mutta sen rakenne oli virheellinen | GVSP-pakettihäviö verkkoreitillä |**Kaapelibudjetti**, verkkokortin vastaanottorengas, jumbo-kehykset, kytkin |
| **Ei saapunut lainkaan**— kehystä ei tullut lainkaan | Kamera ei laukaisi, tai siitä ei lähtenyt mitään |**M8-synkronointikaapeli**, synkronointilinja, ovatko kaikki jäsenet aktivoituna |

Jakautumaa arvioidaan uudelleen 10 sekunnin välein, kun ryhmä lähettää dataa. Yli 5 %:n ylittyessä se
kirjataan lokiin molempien lukujen nimillä, ja jokaisesta vioittuneesta puskurista raportoidaan ensimmäisen kerran, kun se
tapahtuu kamerakohtaisesti, minkä jälkeen tiedot kootaan yhteen kerran minuutissa, jotta pitkä istunto pysyy luettavissa.

**Vioittuneet kehykset, joiden ”ei koskaan saapunut” -arvo on nolla, tarkoittavat, että laukaisu ja kaapelisynkronointi ovat moitteettomia**ja jokainen kadonnut kehys johtuu verkkoreitistä. Korjaus on pienentää**Wire Budget** -arvoa ja
muodostaa yhteys uudelleen.

{% hint style="warning" %}
**Laukaisutaajuuden alentaminen ei auta vioittuneiden kehysten kanssa.** Kameran pakettien
lähetysvauhti määritetään kertaluontoisesti yhteyden muodostuksen yhteydessä. Laukaisutaajuuden alentaminen muuttaa sarjasiirron
esiintymistiheyttä, ei sitä, kuinka nopeasti sarjasiirto itse siirtyy verkkoon. Mitatussa 4-kamerajärjestelmässä
laukaisutaajuuden pienentäminen viisinkertaisesti ei muuttanut tilannetta, kun taas langan budjetin laskeminen 240:stä
200 MB/s:iin laski saman järjestelmän vioittuneiden kehysten osuuden 10,4 %:sta nollaan.
{% endhint %}

Käynnissä oleva järjestelmä ei voi suunnitella itseään uudelleen — katkaise yhteys ja muodosta se uudelleen, jotta yhteyden muodostusajankohdan
valitsin voi toimia uuden kaistanleveyden puitteissa.

### USB-verkkosovittimien yläraja on 200 MB/s

USB-Ethernet-sovitin ilmoittaa *Ethernet*-yhteysnopeutensa, mutta se, mitä se todellisuudessa
pystyy ylläpitämään, on rajoitettu USB-väylän ja sen ohjaimen mukaan. USB 10GbE -sovittimelle annettiin aiemmin
noin 1000 MB/s:n läpimenonopeus – lukemaa, jota kukaan ei ollut koskaan mitannut – ja neljän kameran
toiminnan säätäminen tämän kuvitteellisen varan mukaan aiheutti 6–18 %:n kehysten vioittumisen, vaikka järjestelmä
ilmoitti edelleen terveen kohdekehysnopeuden. USB-liitännällä varustettujen sovittimien nopeus on nyt rajattu
**200 MB/s**:ään. Raja on absoluuttinen eikä prosentuaalinen, koska rajoitteena on
väylä: USB 1 GbE -sovitin saavuttaa noin 80 MB/s, eikä rajoitus vaikuta siihen.

Jos isäntäkoneesi on todellisuudessa nopeampi kuin yläraja, nosta **Wire Budget**-arvoa vastaavasti.

## PTP-aikasynkronointi

Kuvien *synkronointi* tapahtuu laitteistotriggerin avulla; **PTP** (IEEE 1588 PTPv2) tarjoaa vertailukelpoiset *aikaleimat* kaikille laitteille. Se on oletusarvoisesti käytössä, kun järjestelmä kytketään:

* **Chloros-isäntälaite toimii PTP-grandmasterina**. LATTICE-kamerat ja DAQ-E-valosensorit toimivat sen orjina domainissa 0, joten kuvien aikaleimat ja DAQ-spektrit saavat yhden kellon (~1 ms).
* `--no-ptp` (CLI) poistaa sen käytöstä pöytätutkimuksissa — tällöin kameroiden väliset aikaleimat **eivät** ole vertailukelpoisia.
* Tarkista synkronoinnin kunto CLI:n avulla:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Kamerat-välilehdessä itsessään ei ole PTP-ilmaisinta; kamerakohtaiset synkronointitiedot näkyvät siellä vain luku -tilassa olevina **Rooli**(Master/Slave),**Synkronointilinja** ja matriisin Ominaisuudet-taso. DAQ-E:n PTP-tila näkyy Valosensorit-välilehden anturitiedoissa.

## Järjestelmän reaaliaikainen näkymä

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows, and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

Pääsyötealueella on kaksi asettelua (vaihdettavissa yläpalkista): **ruudukkonäkymä**(jokainen ruutu on solu; järjestystä voi muuttaa vetämällä, kun ruudukon lukko on auki) ja**luettelonäkymä**(järjestelmät täysleveänä yläosassa, yksi aktiivinen kamera alla).**Feed Zoom** -liukusäädin säätää ruutujen kokoa; kun solun leveys on alle 200 px, nimi- ja fps-päällekkäisnäytöt piiloutuvat automaattisesti.**Erillinen tila** näyttää yhden ruudun kameraa kohden. Jokaisessa ruudussa näkyy:

* kameran nimi (vasemmassa yläkulmassa),
* **kuvataajuusluku** (oikeassa yläkulmassa) — tämä on taustapalvelimen ilmoittama kameran *todellinen kuvanottotaajuus*, ei esikatselun päivitystaajuus (reaaliaikainen esikatselu on rajattu 30 kuvaan sekunnissa riippumatta kuvanottotaajuudesta),
* tilapiste — vihreä (suoratoisto) / keltainen (latautumassa) / punainen (virhe),
* **vanhan kehyksen pyörivä kuvake**, kun uutta kehystä ei ole saapunut 2 sekunnin aikana — normaalia noin 5 sekunnin ajan yhteyden muodostamisen tai katkeamisen jälkeen, kun taustapalvelu tasapainottaa kaistanleveyttä kameroiden välillä.**Yhdistetty tila**näyttää yhden yhdistelmäruudun: taustapalvelin suorittaa debayeroinnin, skaalauksen, kohdistuksen, kohinanpoiston, muuntamisen kaistakohtaiseksi säteilyvoimakkuudeksi (sekä DLS-heijastavuudeksi, kun valosensori on liitetty), arvioi matriisin indeksilausekkeen, soveltaa LUT:ta ja suoratoistaa tuloksen MJPEG-muodossa. Kunnes ensimmäinen kohdistettu ruutu renderöidään, ruutu ilmoittaa tilansa: ”Valmistellaan matriisia…”, ”Kalibroidaan kohdistusta…”, ”Odotetaan ensimmäistä kehystä…” tai — jos automaattisen kohdistuksen uudelleenyrittämisraja (~30 s) on käytetty loppuun — ”Kohdistus vaaditaan” sekä**Kalibroi kohdistus** -painike.

Hyödyllisiä tietoja yhdistelmätilasta:

* Yhdistelmä rekisteröidään **pääkamera**n kehykseen. AE-ROI-kohdistus ja pistevalotus yhdistelmäkuvassa ovat tarkkoja pääkameran osalta ja likimääräisiä orjakameroiden osalta; käytä**Split View** -toimintoa (matriisin asetukset → &quot;Näytä jäsenkamerat&quot;) saadaksesi pikselintarkat kamerakohtaiset ruudut avaamatta ylimääräisiä kamerayhteyksiä.
* ****Näytöskerrokset**(ryhmäasetukset; oletusarvoisesti pois päältä) mahdollistavat etu- ja taustakerroksen valinnan — mikä tahansa jäsenkamera tai**Indeksi**. Kun etukerros on Indeksi, LUT:n minimi- ja maksimiarvojen ulkopuolella olevat pikselit näyttävät taustakerrosta.
* **Renderöintitarkkuus** (oletus 720p) määrittää livestriimin korkeuden *ja* tallennetun komposiittikuvan vientikoon. Kamerakohtaiset kuvat viedään aina täydellä tarkkuudella.
* Kohdistus lasketaan istuntokohtaisesti eikä sitä tallenneta pysyvästi — katso ryhmäasetuspaneelin kohdistusosio RMS-jäännösten ja *Recalibrate*-painikkeen osalta.

## Tallennus: seuranta vs. analyysi

Array-tallennuspinnat jakautuvat selkeästi **seurantatasoon**(tallentaa näkyvän kuvan) ja**analyysitasoon** (tallentaa raakadataa, kalibroidaan myöhemmin):

| Työnkulku | Taso | Mitä tallennetaan | Käyttöliittymä | CLI |
| --- | --- | --- | --- | --- |
| **Tallennus**(valokuvat) | Analyysi | Yksi synkronoitu kehysryhmä per läpimeno; kamerakohtaiset tiedostot jokaisella valitulla vientitasolla (raaka/debayeroitu/säteily/heijastavuus/esikatselu/indeksi) + `.daq`-sidecar |**Tallenna kaikki**-painike + tallennusasetukset | `lattice array-capture` |
| **Tallenna indeksivideo** | Valvonta | Näytettävä yhdistetty indeksikomposiitti reaaliajassa — 8-bittinen, esikatselutarkkuus, LUT sisäänrakennettuna; edellyttää, että suoratoisto on auki | ● Tallenna indeksivideo (yhdistetyt matriisit) | `lattice array-record` |
| **Raakasarja → videon luominen**| Analyysi | Raakatunnistinkuvat täydellä tallennusnopeudella + manifesti + `.daq`, minkä jälkeen offline-rekonstruointi kalibroiduksi säteilyvoimakkuus-/heijastavuus-/indeksivideoksi, ajallisesti sovitettuna DAQ-lukemiin | ⦿ Tallenna raakasarja →**Luo video** | `lattice array-burst` → `lattice array-build-video` |

Nyrkkisääntö: jos pikseleistä saadaan *mittaustuloksia*, käytä tallennusta tai sarjakuvaus (analyysilaatu); jos haluat vain *katsella tai esitellä* mitä anturi näki, tallenna indeksivideo (seurantalaatu).

### Tallennusasetukset (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->

**Capture All** -painikkeen vieressä oleva rataskuvake avaa Tallennusasetukset-paneelin (vaatii avoimen projektin — tallenteet tallennetaan siihen):

* **Tallennustila**:**Yksittäinen**(yksi kierros) /**Jatkuva**(peräkkäin; rajoitettu tallennusmäärällä, oletusarvo 1, tai kestolla, oletusarvo 10 s) /**Väli** (aikaväli: N tallennusta joka X väli, yhteensä Y; oletusarvo 1 joka 5 s, 1 minuutin ajan).
* **Vientityypit kameraa kohti**: Raw, Debayered, Radiance, Reflectance, Preview, Index — kaikki soveltuvat vaihtoehdot ovat oletusarvoisesti päällä. Radiance/Reflectance-vaihtoehdot ovat piilotettuja RGB-suodattimella varustetuissa kameroissa;**Reflectance näkyy vain, kun kamerassa on DAQ-valosensori** (oma tai matriisista peritty); Index vaatii määritetyn indeksilausekkeen.
* **Kohdistettu**(matriisia kohti, oletusarvoisesti**päällä**): vääristää jäsenviennit matriisin kohdistusprofiilin mukaisiksi, jotta vienti on pikselikohtaisesti rekisteröity. Raw-muoto pysyy aina vääristämättömänä, mutta sisältää muunnoksen metatiedoissa.
* **Nopein tallennus** (kytkin): vain raakadata + määritetty DAQ-lukema + ilmainen yhdistetty indeksikomposiitti, jolloin kalibrointilaskelmat ohitetaan tallennushetkellä maksiminopeuden saavuttamiseksi — säteilyvoimakkuus/heijastavuus/indeksi rakennetaan myöhemmin uudelleen tallennetusta `.daq`-tiedostosta.
* Valinnat säilyvät projektin mukana. Piilotetut tai keskeytetyt kamerat ohitetaan.

Vastaava CLI (sama taustapäätepiste, sama semantiikka):

```bash
# One synced group, every applicable export level per camera (the default)
chloros-cli lattice array-capture -o output/

# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# 30-second monitoring clip of the combined index view, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# 5-second analysis-grade raw burst, then build the combined index video
chloros-cli lattice array-burst --duration 5 --build --products combined:index --fps 10 -o capture/
```

TIFF-pakkaus tallennuksille on `deflate` (häviötön, oletus) tai `none` — täydelliset lipputaulukot, tallennuskansion rakenne ja uudelleenkäsittelysäännöt löytyvät [CLI-viitteestä](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## DAQ-valosensorin parittaminen

Heijastavuus- ja valaistuskorjatut esikatselukuvat edellyttävät alaspäin suuntautuvan valon tietoja DAQ-anturilta (kytketty **Valoanturit**-välilehdessä):

* Sivupalkin **matriisin rivillä**näkyy**&quot;DAQ · päällä/pois&quot; -painike** — *päällä*, kun matriisitason valosensori on asetettu **tai** jollakin matriisin kameralla on oma sensori; sen työkaluvihjeessä luetellaan tarkasti, mikä sensori syöttää tietoa millekin kameralle.
* Määritä koko ryhmää koskeva asetus ryhmäasetuksissa → **Ympäristön valosensori**→**Valosensori**-pudotusvalikosta. Valinta säilyy projektin mukana, välittyy jokaiselle ryhmään kuuluvalle kameralle, ja yksittäiset kamerat voivat silti ohittaa sen omalla sensorillaan.
* Sen alapuolella oleva tilarivi ilmoittaa reaaliaikaisen tilan: **Pois**→ &quot;Odotetaan ensimmäistä spektriä…&quot; →**&quot;Aktiivinen — kaikkien matriisin kameroiden valaistus on korjattu&quot;** → tai, jos viimeisten 3 sekunnin aikana ei ole saapunut uutta spektriä, vanhentunut ilmoitus — viimeistä lukemaa käytetään edelleen (lukemat eivät vanhene koskaan tallennuspolulla).

Kun anturi on määritetty: heijastavuus-vientityyppi tulee käytettäväksi, reaaliaikaiset esikatselukuvat ovat valaistuksen suhteen korjattuja, ennakoiva automaattinen valotus voi käyttää spektriä, ja jokainen heijastavuuskuvaus tallentaa tosiasiallisesti käytetyn DAQ-lukeman **`.daq`-sidecarina** kuvan viereen, jotta tallennetta voidaan käsitellä uudelleen myöhemmin.

## `array-connect` CLI -asetukset

| Lippu | Oletus | Kuvaus |
| --- | --- | --- |
| `--serials SN1,SN2,…` | tunnista kaikki LATTICE-kamerat automaattisesti (vaatii ≥2) | **Ensimmäinen sarjanumero on MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO-synkronointilinja. |
| `--target-fps F` | automaattinen | Master-laukaisun laukaisutaajuus. |
| `--binning {1,2,4}` | automaattinen | Laitteistopohjainen binning. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Asiantuntijan ohitus synkronointitason valitsimelle. |
| `--wire-ceiling-mbps MB_PER_S` | tunnistetaan automaattisesti | Isäntälaitteen kaistanleveys (MB/s) — **Wire Budget**-kentän CLI-muoto. Pienennä arvoa, jos matriisi ilmoittaa vioittuneista kehyksistä. Tallennetaan projektin mukana, joten myöhempi uudelleenkytkentä palauttaa sen. |
| `--no-recommend` | pois | Ohita verkkoanalyysivaihe. |
| `--no-ptp` | pois | Poista PTP käytöstä (kamerien välisiä aikaleimoja ei tällöin voida verrata). |

`lattice array-list`, `array-status` ja `array-disconnect` hallitsevat pysyvää istuntoa. Täydellinen alikomentojen viite, mukaan lukien kohdistus (`align-calibrate` / `align-apply`) ja verkkotyökalut, löytyy kohdasta [CLI-viite § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); vastaavat komennot (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) löytyvät kohdasta [SDK Viite](../reference/sdk-reference.md). Python:stä alkaen johdon budjetti on `connect_array(..., wire_ceiling_mbps=120)`, ja toimivien, vioittuneiden ja perille saapumattomien jakautuma on [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
