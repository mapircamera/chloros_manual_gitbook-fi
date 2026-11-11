# Monikamerajärjestelmät

LATTICE-**järjestelmä**koostuu kahdesta tai useammasta LATTICE-kamerasta, jotka on kytketty yhteen synkronoiduksi yksiköksi. Yksi kamera toimii**pääkamerana**: se lähettää laitteistopohjaisen GPIO-laukaisupulssin jaetulle synkronointilinjalle (oletuksena**Line2**), jotta jokainen jäsen ottaa kuvan samana hetkenä. Chloros lisää PTP-aikasynkronoinnin, reaaliaikaisen esikatselun (kamerakohtaiset ruudut tai yksi kohdistettu monikaistainen yhdistelmä) sekä synkronoidun kuvauksen — jokainen kuvauskerta tuottaa yhden **kuvaryhmän**, jossa kaikilla kameroilla on sama aikaleima ja kuvatunnus (ilmoitetaan tallennustuloksessa muodossa `fid:N`).

Mono (M3M)-kamerat tuottavat kasvillisuusindeksejä matriisien avulla – yksi kamera tuottaa yhden kaistan, ja matriisi kohdistaa ne monikaistaiseksi pinoiksi. Katso [Mono-kamerat ja kasvillisuusindeksit](mono-indices.md).

Järjestelmän liittämiseen on kolme vastaavaa tapaa, ja niissä kaikissa suoritetaan sama ”smart-prep”-prosessi:

| Pinta | Aloituskohta |
| --- | --- |
| GUI | Kamerat-välilehti → **Liitä järjestelmä** (sininen painike) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (ensimmäinen sarjanumero = päälaite) |
| Python-SDK | `connect_array(serials=[…])` → `ArraySession` (ensimmäinen sarjanumero = päälaite) |

Smart-prep suorittaa seuraavat toimet: verkkokapasiteetin testaus (ICMP DF-ping + GVSP-testi), synkronointitason valinta, kuvataajuuden ennuste pyydetyllä kuvakoolla, pakatun 12-bittisen siirron valinta (katso [Pakattu 12-bittinen siirto](#packed-12-bit-transport)), PTP:n käyttöönotto, kamerakohtainen pikselimuodon automaattinen valinta, automaattisen valotuksen aloitusarvojen asettaminen kunkin kameran tallennetun tilan perusteella sekä GPIO-laukaisun konfigurointi linjalla 2.

**Oletusarvoisesti koko anturi (uusi ominaisuus versiossa 1.3.0).** Connect säilyttää pyytämäsi kuvakokoon — koko 2048×1536-anturin, ilman pikselien yhdistämistä tai rajausta, ellet valitse toisin — ja antaa kuvataajuuden laskea siihen, mitä isäntä ja verkko pystyvät tarjoamaan. Se ei koskaan suorita pikselien yhdistämistä tai rajausta omasta aloitteestaan (versiossa 1.2.0 pikselien yhdistäminen käynnistyi automaattisesti alle 1,5 fps:n kuvataajuudella). Binning tai pienempi ROI on sinun päätettävissäsi: valitse yksi vaihtoehto Connect-valintaikkunassa, ja Projected Outcome -kenttä näyttää sen tuottaman kuvataajuuden; `array-connect` tulostaa nopeammat vaihtoehdot ja niiden ennustetut kuvataajuudet.

{% hint style="info" %}
Kameroiden on oltava tavoitettavissa linkin kautta, ennen kuin mikään tästä toimii — katso [Kameroiden liittäminen](connecting.md) löytämisen, osoitteistuksen ja ensimmäisen yhteyden kalibrointilatauksen osalta. Monikamerajärjestelmissä isäntä-NIC:n vastaanottorengasasetukset ovat yhtä tärkeitä kuin linkin nopeus; täydellinen oire→korjaus-taulukko löytyy kohdasta [CLI-viite § Isäntä-NIC:n asetukset ja säätö](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Array Connect -valintaikkuna

Kamerat-välilehti → **Yhdistä ryhmä**avaa kolmivaiheisen ohjatun toiminnon:**Valitse → Näyttötila → Asetukset**.

### Vaihe 1 — Valitse pääkamera ja alikamerat

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered (an FRGB among them). Table showing Camera / Serial / IP / Master radio / Slave checkbox columns in camera order (M3C first, FRGB at the top), with the FRGB pre-selected as master and the green "Sync cable found on N cameras. Any of them can be master; the RGB camera (else the first) is pre-selected." probe banner visible above the table. -->



Valintaikkuna skannaa verkon heti avautuessaan ("Scanning network..."), minkä jälkeen se tunnistaa synkronointikaapelit ("Probing sync cables...") selvittääkseen, mitkä kamerat ovat M8-synkronointilinjalla. Array-ryhmän muodostamiseen tarvitaan vähintään **2 kameraa**.

M8-synkronointikaapelit yhdistyvät yhteen liitäntään, joten **mikä tahansa kaapeloitu kamera voi toimia pääkamerana**. Tunnistin täyttää roolit valmiiksi — RGB-kamera (`-FRGB`) pääkameraksi, kun se on kytketty, muuten ensimmäinen kytketty kamera, ja muut kytketyt kamerat alikameroiksi — ja ilmoittaa yhden kolmesta bannerista:

| Banneri | Merkitys |
| --- | --- |
| ”Synkronointikaapeli löydetty {n} kamerasta. Mikä tahansa niistä voi toimia pääkamerana; RGB-kamera (muussa tapauksessa ensimmäinen) on valittu oletuksena.” (vihreä) | Anturi löysi kamerat synkronointilinjalta; pääkameran ja orjakameran valintaruudut on jo täytetty. |
| ”Pääkameraa ei havaittu – tarkista GPIO-kaapeli” (oranssi) | Mikään kamera ei havainnut laukaisupulssia; tarkista synkronointikaapelointi. Roolit on edelleen valittu valmiiksi (RGB-kamera, muuten luettelon ensimmäinen kamera, pääkameraksi), ja voit muuttaa niitä. |
| "Ei synkronointikaapelia: {sarjanumerot}" (oranssi) | Luettelossa oleviin kameroihin ei ole kytketty synkronointikaapelia. |

Kamerataulukossa on sarakkeet **Kamera / Sarjanumero / IP / Master (radio) / Slave (valintaruutu)**. Rivit on lueteltu**kameroiden järjestyksessä** — M3C (Bayer)-kamerat ennen M3M (mono)-kameroita; M3C-kameroiden joukossa FRGB, FRGN, FOCN ja sitten FNGB; M3M-kamerat suodattimen aallonpituuden mukaan, pienimmästä alkaen (F450 ennen F808). Sivupalkin kameraluettelo ja ruudukon oletusasettelu noudattavat samaa järjestystä.

* Valitse tarkalleen **yksi pääkamera**ja**yksi tai useampi orjakamera**. Napsauttamalla nykyisen pääkameran valintaruutua uudelleen poistat valinnan. Uuden pääkameran valitseminen tapahtuu yhdellä napsautuksella: uusi pääkamera irrottaa orjakamerat, ja edellinen pääkamera muuttuu orjakameraksi (ellei sillä ole synkronointikaapelia).
* Kameraa, jonka kohdalla on merkintä **”Ei synkronointikaapelia”**, ei voi koskaan valita orjaksi — orja, jolla ei ole laukaisukaapelia, odottaisi synkronointilinjaa ikuisesti ja toimittaisi pilaantunutta kuvaa. Liitä kyseinen kamera sen sijaan erilliskamerana.
* Jo itsenäisesti kytketyt kamerat *eivät* poistu käytöstä: ryhmäkytkentä vapauttaa itsenäisen istunnon ja avaa kameran uudelleen ryhmän sisällä.

**Seuraava: Näyttötila →**aktivoituu, kun pääkamera ja vähintään yksi orjakamera on valittu.**Skannaa uudelleen** suorittaa laitteiden tunnistuksen ja johdotuksen tarkistuksen uudelleen.

{% hint style="warning" %}
**Peruuta** on poissa käytöstä, kun skannaus tai johdotustesti on käynnissä — keskeyttäminen kesken testin voi kaataa kameran SDK:n LATTICE-kameran laiteohjelmistossa. Odota, kunnes pyörivä kuvake lakkaa pyörimästä.
{% endhint %}

### Vaihe 2 — Näyttötila

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->



| Tila | Mitä saat |
| --- | --- |
| **Erilliset kamerat** | Yksi reaaliaikainen ruutu kameraa kohti, kaikki laukeavat yhdessä, jotta kuvat pysyvät synkronoituna. Jokainen kamera säilyttää oman värinsä ja asetuksensa. |
| **Yhdistetyt kamerat** *(oletus)* | Yksi ruutu, joka näyttää kohdistetun monikaistaisen NDVI-/indeksi-yhdistelmän. Kamerat jakavat matriisin värin. |

Näyttötila muuttaa vain reaaliaikaisen esikatselun ulkoasua — kuvauskäyttäytyminen on sama molemmissa.

### Vaihe 3 — Matriisin asetukset ja ennustettu tulos

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Trigger Rate / Wire Budget controls (1.3.0 has no "Pin resolution" checkbox), right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->



Tähän näkymään siirryttäessä Chloros pyytää taustapalvelimelta **suositusta**ja soveltaa sitä:**täysi anturi — ei binningiä, ei ROI-rajaus — aina kun kuvataajuus sen sallii**, riippumatta siitä, millä nopeudella isäntä ja yhteys toimittavat dataa. Vasta kun mikään kuvataajuus ei riitä täyden anturin käyttöön, suositus pienentää kuvaa, ja tällöin se suosii pikselien yhdistämistä (binning) ROI-rajauksen sijaan, koska pikselien yhdistäminen säilyttää koko näkökentän. Siitä lähtien vaihdat resoluutiota kuvataajuuden hyväksi. Jokainen tekemäsi muutos suorittaa analyysin uudelleen reaaliaikaisesti ja päivittää oikealla olevan**Ennustettu tulos** -paneelin.

Vasen sarake — asetukset:

| Ohjaus | Vaihtoehdot | Oletus | Huomautukset |
| --- | --- | --- | --- |
| **ROI (näköala)** | Täysi (2048×1536) / Puoli (1024×768) / Neljännes (512×384) | Täysi | Anturin rajaus: Puoli- tai neljännesrajaus pienempään alueeseen alkuperäisellä pikselivälillä. |
| **Binning** | 1× / 2× (summa 2×2) / 4× (summa 4×4) | 1× | Laitteistopohjainen binning: 2×2 = täysi näkökenttä neljänneksellä siirtokustannuksista; 4×4 = täysi näkökenttä 1/16:lla. Piilotettu, jos kamerat eivät tue binningiä. |
| **Verkon puolella oleva kuva** (lukema) | — | — | Binningin jälkeinen leveys × korkeus, joka tosiasiallisesti lähetetään verkon kautta, pyöristettynä 16:n kerrannaisiksi (vähintään 64). |
| **Laukaisutaajuus** | 0,5–60 fps, askel 0,1 | tyhjä = automaattinen | Isäntälaitteen laukaisutaajuus. Jätä tyhjäksi, jotta Chloros voi laskea sen. |
| **Verkkokaistanleveys**| 20–2000 MB/s, askel 10 | tyhjä = auto | Kuinka paljon isäntä voi tosiasiallisesti käsitellä, MB/s:nä —**se yksi luku, josta koko matriisin allokointi riippuu.** Tunnistetaan automaattisesti verkkosovittimesta. Laske arvoa, jos matriisi ilmoittaa vioittuneista kehyksistä: tunnistettu arvo yliarvioi USB-sovittimien ja jaettujen kytkinten kapasiteettia. Arvon muuttaminen käynnistää ennusteen uudelleen reaaliajassa. Kenttä avautuu viimeisimmän liitetyn matriisin kaistanleveydellä (tallennettu projektin mukana), joten kerran korjaamasi kaistanleveys säilyy. |

Versiosta 1.2.0 lähtien **Pin-resoluutio**-valintaruutu on poistettu: yhteys säilyttää aina valitsemasi kehyskokoa ja laskee sen sijaan kehysnopeutta.

Oikea sarake — **Ennustettu tulos**:

* **Synkronointitaso** — ”Samanaikainen tallennus” (vihreä), ”Samanaikainen tallennus (FTD-porrastettu lähetys)” (vihreä), ”Porrastettu tallennus (100 ms:n viive)” (keltainen) tai ”Määritys liian suuri” (punainen). Chloros valitsee tason: samanaikainen tallennus aina, kun kaapelia voidaan jakaa nopeudella, johon kamerat voidaan sovittaa (kuvataajuus laskee, kun kunkin kameran osuus pienenee), ja porrastettu tallennus vain silloin, kun se ei ole mahdollista — joten kameraryhmä muodostaa yhteyden mihin tahansa verkkoon sillä kuvataajuudella, jonka verkko sallii.
* **fps-ennuste** — kuvataajuus, jolla kameraryhmän odotetaan toimivan. Samanaikaisessa tallennuksessa se on yksi luku: kuvataajuus määritetään ensin, ja automaattinen valotus täyttää sen antaman ajanjakson ja nostaa sitten vahvistusta, jotta hämärä kohde ei laske sitä (katso [Kuinka automaattinen valotus jakaa valotuksen ja vahvistuksen](camera-settings.md)). Porrastetussa tallennuksessa se esitetään alueena ("hämärä → kirkas"), koska jokaisen kameran valotusaika pidentää porrastusta hämärässä kohtauksessa.
* **NIC-linja** — linkin nopeus ja jatkuva siirtonopeus ("NIC {mbps} Mbps · jatkuva {N} MB/s").
* **Sim-emit-burst-tarkistus** — pystyykö isäntäkoneen NIC-renkaan vastaanottokapasiteetti käsittelemään yhden samanaikaisen burstin kaikilta kameroilta ("Sim-emit-burst: X MB · käytettävissä oleva NIC-renkaan kapasiteetti: Y MB ✓/✗").
* **Kaapelibudjetin tarkistus** — vakaassa tilassa oleva kokonaiskysyntä verrattuna törmäyksiltä suojaavaan kaapelirajaan ("Kaapelibudjetti: {kysyntä} MB/s, jota vaatii {n} kameraa · yläraja {yläraja} MB/s ✓/✗ ylimerkitty").
* **"Tämän kaistan enimmäiskameramäärä: {n} — määräytyy kamerakohtaisen kaistanleveyden alarajan perusteella, joten ryhmittely ei nosta sitä."** — näkyy, kun olet lähellä (tai yli) kameramäärän ylärajaa.
* **"KUVAT HÄVIÄVÄT näillä asetuksilla."**— punainen varoitus, jossa on taustajärjestelmän syy, sekä estolista ja siniset**korjausehdotukset** ("Jotta tämä ryhmä mahtuu verkkoon" / "Samanaikaisen tallennuksen avaamiseksi").**Käytä & Yhdistä** -painiketta ei voi käyttää, ennen kuin ennuste on olemassa, ja sen teksti kertoo, miksi se hylkää pyynnön:

| Painikkeen teksti | Merkitys | Mikä todella auttaa |
| --- | --- | --- |
| "Analysoidaan..." | Analyysi vielä käynnissä. | Odota. |
| **"Liian monta kameraa tälle verkolle"**| Verkko ei kykene käsittelemään kameraryhmää edes yhtä kameraa kerrallaan (kokonaistarkistus epäonnistui): kameroita on huomattavasti enemmän kuin verkko pystyy käsittelemään, tai verkossa on jo muita kameroita lähettämässä. | Vähemmän kameroita, jumbo-kehykset päästä päähän tai nopeampi verkkokortti.**Pienempi ROI EI auta** — katso alla. |
| **"Pienennä ROI:ta, jotta toiminto voidaan ottaa käyttöön"** | Kehykset putoaisivat näillä asetuksilla (burst/ring-tarkistus epäonnistui). | Pienennä ROI:ta, lisää binningiä tai korjaa verkkokortin vastaanottorengas. |



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->

Yhteyden muodostamisen aikana saattaa ilmestyä vihreä **kalibroinnin latauspaneeli**, jossa on sarjanumeroittain etenevä edistymispalkki: kun kamera liitetään laitteeseen ensimmäistä kertaa, Chloros lataa kamerasta noin 3,8 MB:n suuruisen tehdaskalibrointipaketin GigE-yhteyden kautta (noin 70 sekuntia kameraa kohti). Välimuistissa olevat kamerat eivät koskaan näytä tätä paneelia; kun kameroita on yli kuusi, rivit vierivät. Katso [Kameroiden liittäminen](connecting.md).

### Pakattu 12-bittinen siirto

12-bittiset formaatit (`BayerRG12`, `Mono12`) siirretään yleensä 16-bittisinä sanoina. **Pakattu**siirto lähettää samat 12 bittiä 1,5 tavuna pikseliä kohti ja purkaa ne isäntäkoneella, joten kunkin kehyksen lähettäminen vie 25 % vähemmän aikaa — täydellä anturikapasiteetilla tämä tarkoittaa noin kolmanneksen enemmän kehyksiä sekunnissa.**Tallennetut tiedot ovat identtiset.** Array Connect päättää tämän automaattisesti (`auto`): se pakkaa vain silloin, kun siitä on hyötyä ja isäntäkoneella on prosessoritehoa purkamiseen. Käyttöliittymässä ei ole tätä varten kytkintä; CLI:ssä käytä `array-connect --packed {auto,on,off}` (oletusarvo `auto`). 8- ja 16-bittisiä formaatteja ei pakata koskaan.

## Kaistanleveys: kuinka monta kameraa mahtuu

Se, kuinka monta kameraa array-yhteys pystyy siirtämään, riippuu verkon kapasiteetista, ei Chloros-ohjelmistosta, joten suunnitteluluvut löytyvät laitteistokäsikirjasta: **[Array-kaistanleveyden suunnittelu](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Chloros in toiminta: yhteysvalintaikkuna suorittaa verkkotestin, ennustaa saavutettavissa olevan kuvataajuuden ja valitsee sopivan tason. Ryhmä muodostaa yhteyden mihin tahansa verkkoon: hitaalla yhteydellä jokaisen kameran siirtonopeutta sopeutetaan sen osuuden mukaan kaistanleveydestä ja kuvataajuus laskee, sen sijaan että ryhmä hylättäisiin tai sen kehysten kokoa pienennettäisiin. Vain sellainen ryhmä, jota yhteys ei kykene siirtämään edes yhtä kameraa kerrallaan, hylätään (**Liian monta kameraa tälle verkolle**) sen sijaan, että paketteja pudotettaisiin hiljaisesti — katso edellä kuvattu ennustetun tuloksen paneeli.

## Kun kuvia katoaa

Kamera voi puuttua julkaistusta ryhmästä kahdesta täysin eri syystä,
ja ne vaativat vastakkaisia korjaustoimenpiteitä. Chloros laskee ne erikseen sen sijaan, että ilmoittaisi yhden
”epätäydellisen” luvun, joka ei nimeä kumpaakaan:

| Mitä tapahtui | Mitä se tarkoittaa | Mistä etsiä |
| --- | --- | --- |
| **Vioittunut**— kehys saapui, mutta oli rakenteellisesti viallinen | GVSP-pakettihäviö verkkoreitillä |**Kaapelibudjetti**, verkkokortin vastaanottorengas, jumbo-kehykset, kytkin |
| **Ei saapunut koskaan**— kehystä ei tullut lainkaan | Kamera ei laukaisi, tai siitä ei lähtenyt mitään |**M8-synkronointikaapeli**, synkronointilinja, ovatko kaikki jäsenet valmiustilassa |

Jakautumaa arvioidaan uudelleen 10 sekunnin välein, kun kameraryhmä lähettää dataa. Yli 5 %:n ylittyessä se
kirjataan lokiin molempien lukujen kanssa, ja jokaisesta vioittuneesta puskurista raportoidaan ensimmäisen kerran, kun se
tapahtuu kamerakohtaisesti, minkä jälkeen tiedot kootaan yhteen kerran minuutissa, jotta pitkä istunto pysyy luettavissa.

**Vioittuneet kehykset, joiden ”ei saapunut” -arvo on nolla, tarkoittavat, että laukaisu ja kaapelisynkronointi toimivat moitteettomasti**ja jokainen kadonnut kehys johtuu verkkoreitistä. Korjaus on pienentää**Wire Budget** -arvoa ja
muodostaa yhteys uudelleen.

{% hint style="warning" %}
**Laukaisutaajuuden alentaminen ei auta vioittuneiden kehysten kanssa.** Kameran pakettien
lähetystahti määritetään kerran, yhteyden muodostamisen yhteydessä. Laukaisutaajuuden alentaminen muuttaa sitä, kuinka usein purske
tapahtuu, ei sitä, kuinka nopeasti purske itse siirtyy kaapeliin. Mitatussa 4-kamerajärjestelmässä
laukaisutaajuuden pienentäminen viisinkertaisesti ei muuttanut tilannetta, kun taas langan budjetin laskeminen 240:stä
200 MB/s:iin laski saman järjestelmän vioittuneiden kehysten osuuden 10,4 %:sta nollaan.
{% endhint %}

Käynnissä oleva järjestelmä ei voi suunnitella itseään uudelleen — katkaise yhteys ja muodosta se uudelleen, jotta yhteyden muodostushetken
valitsin voi toimia uuden kaistanleveyden puitteissa.

### USB-verkkosovittimille on asetettu yläraja

USB-Ethernet-sovitin ilmoittaa *Ethernet*-yhteysnopeutensa, mutta se, mitä se todellisuudessa
pystyy ylläpitämään, on rajoitettu USB-väylän ja sen ohjaimen mukaan. USB 10GbE -sovittimelle annettiin aiemmin
noin 1000 MB/s:n läpimenonopeus — lukua, jota kukaan ei ollut koskaan mitannut — ja neljän kameran
sovittaminen tähän kuvitteelliseen kapasiteettiin vioitti 6–18 % kehyksistä, vaikka järjestelmä
ilmoitti edelleen terveen tavoitekehysnopeuden. USB-liitännällä varustettujen sovittimien yläraja on nyt
**200 MB/s**:ään Windows issa ja NVIDIA Jetsonissa sekä **400 MB/s**:ään Linux x86_64:ssä. Raja on absoluuttinen eikä prosentuaalinen, koska rajoitteena on
väylä: USB 1 GbE -sovitin saavuttaa noin 80 MB/s, eikä rajoitus vaikuta siihen.

Jos isäntäkoneesi on todellisuudessa nopeampi kuin yläraja, nosta **Wire Budget** -arvoa sen mukaisesti.

## PTP-aikasynkronointi

Kuvien *synkronointi* tapahtuu laitteistotriggerin avulla; **PTP** (IEEE 1588 PTPv2) tarjoaa vertailukelpoiset *aikaleimat* kaikille laitteille. Se on oletusarvoisesti käytössä, kun järjestelmä kytketään:

* **Chloros-isäntäpalvelin toimii PTP-päälaitteena**. LATTICE-kamerat ja DAQ-E-valosensorit toimivat sen alaisina domainissa 0, joten kuvien aikaleimat ja DAQ-spektrit ovat samassa kellossa (~1 ms).
* `--no-ptp` (CLI) poistaa sen käytöstä pöytätutkimuksissa — tällöin kameroiden väliset aikaleimat **eivät** ole vertailukelpoisia.
* Tarkista synkronoinnin tila CLI:n avulla:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity, health (OK, or why its packets are not reaching the network)
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Kamerat-välilehdessä itsessään ei ole PTP-ilmaisinta; kamerakohtaiset synkronointitiedot näkyvät siellä vain luku -muotoisina **Rooli**(Master/Slave),**Synkronointilinja** ja matriisin Ominaisuudet-taso. DAQ-E:n PTP-tila näkyy Valosensorit-välilehden anturitiedoissa.

## Ryhmän reaaliaikainen näkymä

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows in camera order (M3C before M3M, monos by wavelength), and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

Pääsyötealueella on kaksi asettelua (vaihdettavissa yläpalkissa): **ruudukkonäkymä**(jokainen ruutu on solu; järjestystä voi muuttaa vetämällä, kun ruudukon lukko on auki) ja**luettelonäkymä**(täysleveät rivit:**yhdistetty**ryhmä on yksi rivi – sen yhdistetty ruutu – kun taas**erillinen**ryhmä antaa**jokaiselle jäsenelle oman rivinsä**, omalla reaaliaikaisella syötöllä ja omalla kalibrointikohdetaulukollaan; ruudukkojen alapuolella yksi erillinen kamera — aktiivinen kamera).**Syötteen zoomaus** -liukusäädin säätää ruutujen kokoa; kun solun leveys on alle 200 px, nimi- ja fps-päällekkäisnäytöt piiloutuvat automaattisesti. Jäsenkamerat näkyvät sivupalkissa ja ruudukon oletusasettelussa kamerajärjestyksessä (M3C ennen M3M, FRGB ensin, monokromikamerat aallonpituuden mukaan); ruudukon lukituksen avaaminen ja vetäminen ohittaa kuitenkin edelleen ruudukon järjestyksen.

<!-- SCREENSHOT-NEEDED: list view of a connected SEPARATE array (1.3.0): one full-width row per member camera, each with its live feed and its own Calibration Target table below it. -->



**Erillinen tila** näyttää yhden ruudun kameraa kohden. Jokaisessa ruudussa näkyy:

* kameran nimi (vasemmassa yläkulmassa),
* **kuvataajuuslukema** (oikeassa yläkulmassa) — tämä on taustapalvelimen ilmoittama kameran *todellinen kuvanottotaajuus*, ei esikatselun kyselytaajuus (reaaliaikaisen esikatselun yläraja on 30 fps kuvanottotaajuudesta riippumatta),
* tilapiste — vihreä (suoratoisto) / keltainen (latautumassa) / punainen (virhe),
* **vanhentuneen kehyksen pyörivä kuvake**, kun uutta kehystä ei ole saapunut 2 sekunnin aikana — normaalia noin 5 sekunnin ajan minkä tahansa yhteyden muodostamisen tai katkaisemisen jälkeen, kun taustapalvelu tasapainottaa kaistanleveyttä kameroiden välillä.**Yhdistetty tila**näyttää yhden yhdistelmäruudun: taustapalvelu suorittaa debayeroinnin, skaalauksen, kohdistuksen, kohinanpoiston, muuntamisen kaistakohtaiseksi säteilyvoimakkuudeksi (sekä DLS-heijastavuudeksi, kun valosensori on liitetty), arvioi matriisin indeksilausekkeen, soveltaa LUT:ta ja lähettää tuloksen MJPEG-muodossa. Kunnes ensimmäinen kohdistettu kehys on renderoitu, ruutu ilmoittaa tilansa: ”Valmistellaan matriisia…”, ”Kalibroidaan kohdistusta…”, ”Odotetaan ensimmäistä kehystä…” tai — jos ruudun automaattisen kohdistuksen uudelleenkokeilut (noin minuutti) loppuvat — ”Kohdistus vaaditaan” sekä**Kalibroi kohdistus**-painike.

Matriisiruudun fps-näyttö ilmoittaa myös streamin tilan:**”Sovelletaan asetuksia…”**, kun asetusten muutos (formaatti, binning, ROI) käynnistää kamerat uudelleen,**”Jumissa”**, kun matriisi on toiminnassa mutta on lopettanut kehysryhmien toimittamisen, ja**”Pysäytetty"**, kun se on lopettanut datan lähettämisen — liitä matriisi tällöin uudelleen. Kun spektrinen erottelu joutui rajoittamaan komposiitissa näkyvää kaistaa suuressa osassa sen pikseleistä (25 % tai enemmän), ruudussa näkyy**"Vähäisen luotettavuuden kaistatiedot · N %"**: kyseisen kaistan perusteella lasketut indeksiarvot ovat epäluotettavia (yleensä yksi kaista on vastaanottanut liian vähän valoa).

Hyödyllisiä tietoja yhdistelmätilasta:

* Yhdistelmäkuva on rekisteröity **pääkameran**kehykseen. AE-ROI-kohdistus ja pistevalotus yhdistelmäkuvassa ovat tarkkoja pääkameran osalta ja likimääräisiä orjakameroiden osalta; käytä**Split View** -toimintoa (anturi-asetukset → ”Näytä jäsenkamerat”) saadaksesi pikselintarkat kamerakohtaiset ruudut avaamatta ylimääräisiä kamerayhteyksiä.
* **Näytökerrokset**(ryhmäasetukset; oletusarvoisesti pois päältä) mahdollistavat etu- ja taustakerroksen valinnan — mikä tahansa jäsenkamera tai**Indeksi**. Kun etukerros = Indeksi, LUT:n minimi- ja maksimiarvojen ulkopuolella olevat pikselit näyttävät taustakerroksen.
* **Renderöintitarkkuus** (oletus 720p) määrittää livestriimin korkeuden *ja* tallennetun komposiittikuvan vientikoon. Kamerakohtaiset kuvat viedään aina täydessä resoluutiossa.
* Kohdistus lasketaan istuntokohtaisesti, eikä sitä tallenneta. Chloros kohdistaa **jokaisen ryhmän automaattisesti yhteyden muodostamisen jälkeen, molemmissa näyttötiloissa**, kun jäsenkameroiden automaattinen valotus on vakiintunut (ohjelma yrittää enintään 5 minuuttia). Ryhmäasetuspaneelin**Kohdistus**-osiossa näkyvät RMS-jäännökset ja**Kohdistuksen uudelleenkalibrointi**-painike — myös erillisille ryhmille, joilla ei ole yhdistettyä ruutua.

## Tallennus: seuranta vs. analyysi

Kuvauspinnat jakautuvat selkeästi **seurantatasoon**(tallentaa näkyvän kuvan) ja**analyysitasoon** (tallentaa raakadataa, kalibroidaan myöhemmin):

| Työnkulku | Taso | Mitä tallennetaan | GUI | CLI |
| --- | --- | --- | --- | --- |
| **Tallennus**(valokuvat) | Analyysi | Yksi synkronoitu kehysryhmä per läpikäynti; kamerakohtaiset tiedostot jokaisella valitulla vientitasolla (raaka/debayeroitu/säteily/heijastavuus/esikatselu/indeksi) + `.daq`-sidecar |**Tallenna kaikki**-painike + Tallennusasetukset | `lattice array-capture` |
| **Tallenna indeksivideo** | Valvonta | Näytettävä yhdistetty indeksikomposiitti reaaliajassa — 8-bittinen, esikatseluresoluutio, LUT sisäänrakennettuna. CLI:stä ilman avointa GUI-näkymää taustaprosessi renderöi matriisin NDVI:n (tai `--expr`:n), kun matriisi on kohdistettu; tallennus epäonnistuu, jos yhtään kehystä ei kirjoiteta | ● Tallenna indeksivideo (yhdistetyt matriisit) | `lattice array-record` |
| **Raakasarja → videon luominen**| Analyysi | Raakakuvat täydellä tallennusnopeudella + manifesti + `.daq`, minkä jälkeen offline-rekonstruointi kalibroiduksi säteily-/heijastus-/indeksivideoksi, ajallisesti sovitettuna DAQ-lukemiin | ⦿ Tallenna raakasarja →**Luo video** | `lattice array-burst` → `lattice array-build-video` |

Nyrkkisääntö: jos pikseleistä saadaan *mittaustuloksia*, käytä tallennusta tai sarjakuvaus (analyysitarkkuus); jos haluat vain *katsoa tai esitellä* mitä anturi näki, tallenna indeksivideo (seurantatarkkuus).

### Tallennusasetukset (GUI)



<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->

**Capture All** -painikkeen vieressä oleva ratas avaa Tallennusasetukset-paneelin (vaatii avoimen projektin — tallenteet tallennetaan siihen):

* **Tallennustila**:**Yksittäinen**(yksi kierros) /**Jatkuva**(peräkkäin; rajoitettu tallennusmäärällä, oletusarvo 1, tai kestolla, oletusarvo 10 s) /**Väli** (aikaväli: N tallennusta joka X väli, yhteensä Y; oletusarvo 1 joka 5 s, 1 minuutin ajan).
* **Vientityypit kameraa kohti**: Raw, Debayered, Radiance, Reflectance, Preview, Index — kaikki soveltuvat ovat oletusarvoisesti päällä. Radiance/Reflectance ovat piilotettuja RGB-suodattimella varustetuissa kameroissa; **Reflectance näkyy vain, kun kamerassa on DAQ-valosensori** (oma tai matriisista peritty); Index vaatii määritetyn indeksilausekkeen.
* **Aligned**(matriisikohtaisesti, oletus**päällä**): muokkaa jäsenviennit matriisin kohdistusprofiilin mukaisiksi, jolloin vienti on pikselikohtaisesti rekisteröity. Raw-tiedostot pysyvät aina muuntamattomina, mutta sisältävät muunnoksen metatiedoissa. Matriisin jäsenetiedostot tallennetaan kunkin kameran alkuperäisessä suunnassa (mahdolliset peilaus- tai kiertämisasetukset tallennetaan tiedostoon, mutta niitä ei sovelleta).
* **Nopein tallennus** (kytkin): vain raakadata + määritetty DAQ-lukema + ilmainen yhdistetty indeksikomposiitti; kalibrointilaskelmat ohitetaan tallennushetkellä maksiminopeuden saavuttamiseksi — säteilyvoimakkuus/heijastavuus/indeksi voidaan rakentaa myöhemmin tallennetusta `.daq`-tiedostosta.
* Valinnat säilyvät projektin mukana. Piilotetut tai keskeytetyt kamerat ohitetaan.
* **Jatkuva**-tila ryhmissä toimii Chloros-taustapalvelimessa (uusi ominaisuus versiossa 1.3.0): kun jokainen luettelossa oleva kamera kuuluu ryhmään, taustapalvelin suorittaa tallennussilmukan itse, kirjoittaen yhden ryhmän samalla kun se valitsee seuraavan, ilman erillistä pyyntöä jokaiselle tallennukselle. Täysi tallennuslaite keskeyttää ajon ("Pysäytetty: tallennuslaite täynnä. Tallennettu N, F epäonnistui"). Katso [Tallennusasetukset ja -tilat](capture.md#capture-modes), jossa on Microsoft Defenderin huomautus suurista tallennusnopeuksista.

Vastaava CLI-komento (sama taustapalvelimen päätepiste, sama semantiikka):

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

TIFF-pakkaus tallennuksille on `deflate` (häviötön, oletus) tai `none` — täydelliset lipputaulukot, tallennuskansion rakenne ja uudelleenkäsittelysäännöt löytyvät [CLI-viitteessä](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## DAQ-valosensorin parittaminen

Heijastavuus- ja valaistuskorjatut esikatselukuvat vaativat DAQ-sensorilta (joka on kytketty **Valosensorit**-välilehdessä) tulevaa alaspäin suuntautuvaa valotietoa:

* Sivupalkin **matriisin rivillä**näkyy**"DAQ · päällä/pois" -painike** — *päällä*, kun matriisitason valosensori on asetettu **tai** jollakin matriisin kameralla on oma sensori; sen työkaluvihje kertoo tarkalleen, mikä sensori syöttää tietoja millekin kameralle.
* Määritä koko ryhmää koskeva asetus ryhmän asetuksissa → **Ympäristön valosensori**→**Valosensori**-pudotusvalikosta. Valinta säilyy projektin mukana, leviää jokaiseen ryhmään kuuluvaan kameraan, ja yksittäiset kamerat voivat silti ohittaa sen omalla sensorillaan.
* Sen alapuolella oleva tilarivi ilmoittaa reaaliaikaisen tilan: **Pois**→ ”Odotetaan ensimmäistä spektriä…” →**”Aktiivinen — kaikkien ryhmän kameroiden valaistus on korjattu”** → tai, jos uutta spektriä ei ole saapunut viimeisten 3 sekunnin aikana, vanhentunut ilmoitus — viimeistä lukemaa käytetään edelleen (lukemat eivät vanhene koskaan tallennuspolulla).

Kun anturi on määritetty: Reflectance-vientityyppi tulee käytettäväksi, reaaliaikaiset esikatselukuvat ovat valaistuksen suhteen korjattuja, ennakoiva automaattinen valotus voi käyttää spektriä, ja jokainen heijastavuuskuvaus tallentaa tosiasiallisesti käytetyn DAQ-lukeman **`.daq`-tiedostona** kuvien `daq/`-kansioon, jotta tallennetta voidaan käsitellä uudelleen myöhemmin. Sen vieressä oleva `.daq.cameras.json`-tiedosto nimeää kamerat, joiden kuvia kyseinen lukema kuvaa; uudelleenkäsittelyn yhteydessä Chloros yhdistää lukeman vain kyseisten kameroiden kuviin.

## `array-connect`-CLI-vaihtoehdot

| Lippu | Oletus | Kuvaus |
| --- | --- | --- |
| `--serials SN1,SN2,…` | tunnista kaikki LATTICE-kamerat automaattisesti (vaatii vähintään 2) | **Ensimmäinen sarjanumero on MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO-synkronointilinja. |
| `--target-fps F` | auto | Master-laukaisun laukaisutaajuus. |
| `--binning {1,2,4}` | auto | Laitteistopohjainen binning. Auto-asetus tarkoittaa koko anturia (ei binningiä), kun tahansa kuvataajuus tukee sitä; connect-asetus ei koskaan käytä binningiä, jotta kuvataajuus pysyy korkeana. |
| `--packed {auto,on,off}` | `auto` | [Pakattu 12-bittinen siirto](#packed-12-bit-transport): `auto` pakkaa vain silloin, kun siitä on hyötyä ja isäntä pystyy purkamaan pakkauksen. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Synkronointitason valitsimen asiantuntijan ohitus. |
| `--wire-ceiling-mbps MB_PER_S` | tunnistetaan automaattisesti | Isäntäkoneen kaistanleveysraja megabitteinä sekunnissa (MB/s) — **Wire Budget**-kentän CLI-muoto. Pienennä arvoa, jos järjestelmä ilmoittaa vioittuneista kehyksistä. Tallennetaan projektin mukana, joten myöhempi uudelleenkytkentä palauttaa sen. |
| `--no-recommend` | pois | Ohita verkkoanalyysivaihe. |
| `--no-ptp` | pois | Poista PTP käytöstä (kameroiden välisiä aikaleimoja ei tällöin voida verrata). |

`lattice array-list`, `array-status` ja `array-disconnect` hallitsevat pysyvää istuntoa. Täydellinen alikomento-opas, mukaan lukien kohdistus (`align-calibrate` / `align-apply`) ja verkkotyökalut, löytyy kohdasta [CLI-opas § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); vastaavat SDK-komennot (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) löytyvät [SDK-viitteessä](../reference/sdk-reference.md). Python-kielellä tiedonsiirtokapasiteetti on `connect_array(..., wire_ceiling_mbps=120)`, ja reaaliaikainen jaottelu vioittuneisiin ja perille saapumattomiin viesteihin löytyy kohdasta [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
