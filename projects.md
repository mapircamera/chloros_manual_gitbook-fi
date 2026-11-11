# Käyttöliittymä: Projektit

Chloros-ohjelmalla voit luoda projekteja, jotka voidaan avata uudelleen myöhemmin. Projekti on tavallinen kansio (projektikansiossasi), joka sisältää seuraavat tiedostot:

* `project.json` — projektin asetukset, tiedostoluettelo ja näyttöasetukset
* `cameras.json` — projektin ollessa auki kytketyt kamerat ja anturiryhmät sekä niiden asetukset
* `sensors.json` — projektin ollessa auki kytketyt DAQ-valosensorit sekä kamera↔sensori-yhdistelmät
* tallenteesi, `.daq`-tallenteet ja käsitellyn tuloksen kansiot

Projektitiedostoille ei ole omaa tiedostomuotoa — kansio ja sen JSON-tiedostot muodostavat projektin, minkä ansiosta projekteja on helppo kopioida, arkistoida ja käyttää [CLI:n](CLI.md) tai [Python SDK:n](api-python-sdk.md) kautta.

Ellet muuta sitä [Projektin asetuksissa](project-settings/project-settings.md), projektikansio on `Documents\MAPIR\Chloros Projects` Dokumentit-kansiossasi (`~/Documents/MAPIR/Chloros Projects` Linuxissa).

{% hint style="warning" %}
**Windows: pidä projektikansiot lyhyinä, ellei pitkiä polkuja ole käytössä.**Windowsissa pitkät polut ovat oletusarvoisesti pois käytöstä, jolloin tiedostopolku ei saa olla pidempi kuin 260 merkkiä. Käsitellyt tuotteet sijaitsevat enintään noin 130 merkkiä projektikansion alapuolella, joten versiosta 1.3.0 lähtien Chloros hylkää liian syvällä sijaitsevan projektikansion – kun luot tai avaat sen tai kun aloitat käsittelyn – sen sijaan, että se suorittaisi ohjelman kirjoittamatta mitään. Viestissä ilmoitetaan pisin sopiva projektikansio. Käytä lyhyempää projektikansiota tai ota käyttöön Windowsin pitkät polut (asennusohjelman**Ota Windowsin pitkät polut käyttöön** -vaihtoehto, valittuna oletuksena) ja käynnistä Chloros uudelleen. Katso [Käsittelyn aloittaminen](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).
{% endhint %}

## Uusi projekti

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>Valitse päävalikosta ”Uusi projekti” ja anna projektillesi yksilöllinen nimi.

Jos olet tallentanut projektimalleja, **Valitse malli** -pudotusvalikko ilmestyy nimikentän alle — mallin valitseminen käynnistää uuden projektin kyseisen mallin asetuksista. Mallit tallennetaan kohdasta [Projektin asetukset](project-settings/project-settings.md): kirjoita nimi kenttään ”Tallenna projektimallin nimi” ja napsauta tallennuskuvaketta.

## Avaa projekti

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>”Avaa projekti” -kohdassa näkyy luettelo kaikista projektikansiossasi olevista projekteista, ja alareunassa on <strong>”Avaa projektikansio”</strong></p></figcaption></figure>

Valitse ”Avaa projekti”, niin näet luettelon projektikansiossa olevista projekteista. Jos projekteja ei ole, sivumenu ei avaudu. Yllä olevassa kuvassa näkyy joitakin GUI:n luomia projekteja (t1, t2, t3). DATE\_TIME-projektit on luotu CLI:n avulla oletusprojektinimityskäytännön mukaisesti. Klikkaamalla mitä tahansa projektin nimeä avaat sen.

Klikkaamalla ”Avaa projektikansio”-painiketta avaat tietokoneesi tiedostoselaimen projektin polulle. Voit muokata projektin polkua kohdassa [Projektin asetukset](project-settings/project-settings.md).

Jos jotakin projektin lähdekuvatiedostoista on siirretty tai poistettu sen viimeisen avaamisen jälkeen, Chloros näyttää valintaikkunan, jossa luetellaan tarkasti puuttuvat tiedostot, sen sijaan että avaisi tyhjän ruudukon.

## Kopioi projekti

Toiminto on käytettävissä, kun projekti on avattu. Valitse ”Kopioi projekti” kopioidaksesi nykyisen projektin uudella nimellä — Chloros ehdottaa seuraavaa vapaata nimeä (esim. ”MyProject (2)”) — ja kopio avautuu välittömästi.

## Lisää tiedostoja

Kun projekti on avattu, valitse päävalikosta ”Lisää tiedostoja” lisätäksesi yksittäisiä kuvatiedostoja nykyiseen projektiin. Tämä vastaa tiedostoselaimen lisäystoimintoa, mutta on käytettävissä suoraan päävalikosta mukavuuden vuoksi.

## Lisää kansio

Kun projekti on avattu, valitse päävalikosta ”Lisää kansio”, jotta voit lisätä kuvakansioita nykyiseen projektiin. Voit valita useita kansioita kerralla. Päällekkäiset tiedostot ohitetaan.

## Käsittelyn aloittaminen / lopettaminen

Kun tiedostot on lisätty projektiin, päävalikossa tulee näkyviin vaihtoehto ”Aloita käsittely”. Tämä vastaa yläpalkissa olevan Toista/Aloita-painikkeen napsauttamista. Käsittelyn aikana valikkokohta muuttuu muotoon ”Lopeta käsittely”, jolloin voit keskeyttää käsittelyputken.

## Yhdistä kameraan / Yhdistä valosensoriin

Päävalikon alaosassa on kaksi laitteistopikakuvaketta, jotka ovat käytettävissä riippumatta siitä, onko projekti auki vai ei:

* **Yhdistä kameraan** — avaa [Kamerat-välilehden](lattice/), jossa voit yhdistää LATTICE-kameran tai -matriisin.
* **Yhdistä valosensoriin** — avaa [Valosensorit-välilehden](daq/), jossa voit liittää DAQ-valosensorin.

Laitteiston liittäminen projektin ollessa auki tallentaa sen projektiin (katso alla). Ilman projektia liitännät ovat voimassa vain kyseisen istunnon ajan.

{% hint style="info" %}
Valikkokohdat Lisää tiedostoja, Lisää kansio ja Käynnistä/pysäytä käsittely näkyvät tai ovat käytettävissä vain, kun projekti on auki ja tiedostoja on lisätty. Niiden avulla pääsee nopeasti toimintoihin, jotka ovat käytettävissä myös Tiedostoselaimen sivupalkin ja otsikkopainikkeiden kautta.
{% endhint %}

## Projektit muistavat laitteistosi

Uutta versiossa 1.2.0: projekti säilyttää tiedot liitetyistä laitteista niin kauan kuin se on auki. Kamerat ja kameraryhmät (kamera-kohtaiset asetukset, nimet, värit ja ruudukon asettelu mukaan lukien) tallennetaan tilannekuvana tiedostoon `cameras.json`, ja valosensorit (nimet, värit ja kamerayhteydet mukaan lukien) tiedostoon `sensors.json` — automaattisesti, kun työskentelet.

Kun **avaat uudelleen** projektin, Chloros ei heti muokkaa laitteistoa. Kukin osa muodostaa yhteyden uudelleen, kun avaat sen sisältävän välilehden ensimmäisen kerran:

* **Kamerat**-välilehden avaaminen yhdistää tallennetut kamerat ja matriisit uudelleen ja ottaa niiden tallennetut asetukset uudelleen käyttöön.
* **Valoanturit**-välilehden avaaminen yhdistää tallennetut DAQ-anturit uudelleen ja ottaa niiden tallennetut asetukset uudelleen käyttöön, suojus mukaan lukien. Anturi, jonka Cap-valikossa on asetettu**None (paljas anturi)**, pysyy paljaana. Aikaisemmalla Chloros-versiolla tallennetussa projektissa paljas anturi tallennettiin vaihtoehtona, joten se palautuu auringonvalon kosinikorjauksella; aseta se uudelleen None-tilaan ja tallenna projekti.

Tällä tavalla projektin avaaminen pelkästään kuvien selaamista tai vientiä varten ei koskaan käynnistä kameroiden suoratoistoa. Jos tallennettua laitetta ei löydy, kun sen välilehti avataan, valintaikkuna ilmoittaa, mitkä laitteet eivät ole käytettävissä, jotta voit yhdistää ne uudelleen tai poistaa ne.

## DAQ-tallenteet ja .daq-tiedostot projektissa

* Projektin ollessa auki tehdyt `.daq`-tallenteet (Valoanturit-välilehdestä tai kuvauksen aikana) **lisätään automaattisesti projektiin**.
* Tuodut `.daq`-tiedostot sekä kaikki projektin tallenteet näkyvät luettelossa [Projektin asetukset](project-settings/project-settings.md) -kohdan **DAQ-valosensori**-osiossa, ja jokaisella on oma valokorjausprofiilinsa.
* Käsittelyn aikana projektin `.daq`-tiedostot toimittavat alaspäin suuntautuvan valaistuksen heijastavuuslukuja varten — katso [Tulostuskuvamuodot](output-image-formats.md).

## Tallennetun projektin ajaminen ilman käyttöliittymää

Tallennettua projektia voidaan ajaa ilman graafista käyttöliittymää:

* **CLI**: `chloros-cli project open / connect / capture / sensor / align / run` toimii projektikansion polun perusteella — katso [CLI-viite](reference/cli-reference.md).
* **SDK**: `chloros_sdk.open_project(path)` palauttaa projektikahvan; `connect_all()` ottaa kaikki tallennetut kamerat, matriisit ja valosensorit käyttöön tallennetuilla asetuksilla (valosensorit käynnissä olevan Chloros-taustapalvelimen kautta) — katso [SDK-ohje](reference/sdk-reference.md).
