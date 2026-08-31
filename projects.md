# Käyttöliittymä: Projektit

Chloros:n avulla voit luoda projekteja, jotka voidaan avata uudelleen myöhemmin. Projekti on tavallinen kansio (projektikansiossasi), joka sisältää:

* `project.json` — projektin asetukset, tiedostoluettelo ja näyttöasetukset
* `cameras.json` — projektin ollessa auki kytketyt kamerat ja anturiryhmät sekä niiden asetukset
* `sensors.json` — projektin ollessa auki kytketyt DAQ-valosensorit sekä kamera↔sensori-yhdistelmät
* tallenteesi, `.daq`-tallenteet ja käsitellyt tulostuskansiot

Erillistä projektitiedostomuotoa ei ole — kansio ja sen JSON-tiedostot muodostavat projektin, minkä ansiosta projekteja on helppo kopioida, arkistoida ja siirtää [CLI](CLI.md) tai [Python SDK](api-python-sdk.md).

## Uusi projekti

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>Valitse päävalikosta ”Uusi projekti” ja anna projektillesi yksilöllinen nimi.

Jos olet tallentanut projektimalleja, **Valitse malli** -pudotusvalikko ilmestyy nimikentän alle — valitsemalla yhden niistä uusi projekti aloitetaan kyseisen mallin asetuksista. Mallit tallennetaan kohdasta [Projektin asetukset](project-settings/project-settings.md): kirjoita nimi kenttään ”Tallenna projektimallin nimi” ja napsauta tallennuskuvaketta.

## Avaa projekti

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>”Avaa projekti” -kohdassa näkyy luettelo projektikansiossasi olevista projekteista, ja alareunassa on <strong>”Avaa projektikansio”</strong></p></figcaption></figure>Valitse ”Avaa projekti”, niin näet luettelon projektikansiossa olevista projekteista. Jos projekteja ei ole, toissijainen sivumenu ei avaudu. Yllä olevassa kuvassa näkyy joitakin GUI:n luomia projekteja (t1, t2, t3). DATE\_TIME-projektit on luotu CLI-ohjelmalla käyttäen oletusarvoista projektien nimeämiskäytäntöä. Klikkaamalla mitä tahansa projektin nimeä avaat sen.

Napsauttamalla ”Avaa projektikansio”-painiketta avaat tietokoneesi tiedostoselaimen projektin polulle. Voit muokata projektin polkua kohdassa [Projektin asetukset](project-settings/project-settings.md).

Jos jotakin projektin lähdekuvatiedostoista on siirretty tai poistettu sen viimeisen avaamisen jälkeen, Chloros näyttää valintaikkunan, jossa luetellaan tarkasti puuttuvat tiedostot, sen sijaan että avaisi tyhjän ruudukon.

## Projektin kopiointi

Toiminto on käytettävissä, kun projekti on avattu. Valitse &quot;Kopioi projekti&quot;, jos haluat kopioida nykyisen projektin uudella nimellä — Chloros ehdottaa seuraavaa vapaata nimeä (esim. &quot;MyProject (2)&quot;) — ja kopio avautuu välittömästi.

## Lisää tiedostoja

Kun projekti on avattu, valitse päävalikosta ”Lisää tiedostoja” lisätäksesi yksittäisiä kuvatiedostoja nykyiseen projektiin. Tämä vastaa tiedostoselaimen lisäystoimintoa, mutta on käytettävissä suoraan päävalikosta mukavuuden vuoksi.

## Lisää kansio

Kun projekti on avattu, valitse päävalikosta ”Lisää kansio”, jotta voit lisätä kuvakansioita nykyiseen projektiin. Voit valita useita kansioita kerralla. Päällekkäiset tiedostot ohitetaan.

## Käynnistä / Pysäytä käsittely

Kun tiedostot on lisätty projektiin, päävalikossa tulee näkyviin vaihtoehto ”Käynnistä käsittely”. Tämä vastaa yläpalkissa olevan Toista/Käynnistä-painikkeen napsauttamista. Käsittelyn aikana valikkokohta muuttuu muotoon ”Pysäytä käsittely”, jolloin voit keskeyttää käsittelyputken.

## Yhdistä kameraan / Yhdistä valosensoriin

Päävalikon alaosassa on kaksi laitteistopikakuvaketta, jotka ovat käytettävissä riippumatta siitä, onko projekti auki vai ei:

* **Yhdistä kameraan** — avaa [Kamerat-välilehden](lattice/), jossa voit yhdistää LATTICE-kameran tai -matriisin.
* **Yhdistä valosensoriin** — avaa [Valosensorit-välilehden](daq/), jossa voit liittää DAQ-valosensorin.

Laitteiston liittäminen projektin ollessa auki tallentaa sen projektiin (katso alla). Ilman projektia liitännät ovat voimassa vain kyseisen istunnon ajan.

{% hint style="info" %}
Valikkokohdat Lisää tiedostoja, Lisää kansio ja Käynnistä/pysäytä käsittely näkyvät tai ovat käytettävissä vain, kun projekti on auki ja tiedostoja on lisätty. Niiden avulla pääsee nopeasti toimintoihin, jotka ovat käytettävissä myös Tiedostoselaimen sivupalkissa ja otsikkopainikkeissa.
{% endhint %}

## Projektit muistavat laitteistosi

Uutta versiossa 1.2.0: projekti säilyttää tiedot liitetyistä laitteista niin kauan kuin se on auki. Kamerat ja kameraryhmät (kamera-kohtaiset asetukset, nimet, värit ja ruudukon asettelu mukaan lukien) tallennetaan tilannekuvana tiedostoon `cameras.json`, ja valosensorit (nimet, värit ja kamerayhdistykset mukaan lukien) tiedostoon `sensors.json` — automaattisesti, kun työskentelet.

Kun **avaat uudelleen** projektin, Chloros ei ota välittömästi yhteyttä mihinkään laitteistoon. Kukin puolisko muodostaa yhteyden uudelleen, kun avaat sen sisältävän välilehden ensimmäisen kerran:

* **Kamerat**-välilehden avaaminen muodostaa uudelleen yhteyden tallennettuihin kameroihin ja matriiseihin ja ottaa niiden tallennetut asetukset uudelleen käyttöön.
* **Valosensorit**-välilehden avaaminen muodostaa uudelleen yhteyden tallennettuihin DAQ-antureihin.

Tällä tavalla projektin avaaminen pelkästään kuvien selaamista tai vientiä varten ei koskaan käynnistä kameroiden suoratoistoa. Jos tallennettua laitetta ei löydy välilehden avaamisen yhteydessä, valintaikkuna ilmoittaa, mitkä laitteet eivät ole käytettävissä, jotta voit muodostaa niihin yhteyden uudelleen tai poistaa ne.

## DAQ-tallenteet ja .daq-tiedostot projektissa

* Projektin ollessa auki tehdyt `.daq`-tallenteet (Valoanturit-välilehdestä tai tallennusten aikana) **lisätään automaattisesti projektiin**.
* Tuodut `.daq`-tiedostot sekä kaikki projektin tallenteet näkyvät [Projektin asetukset](project-settings/project-settings.md) -kohdan **DAQ-valosensori**-osiossa, ja jokaisella on oma valokorjausprofiilinsa.
* Käsittelyn aikana projektin `.daq`-tiedostot toimittavat alaspäin suuntautuvan valaistuksen heijastavuuslukuja varten — katso [Lähtökuvamuodot](output-image-formats.md).

## Tallennetun projektin ajaminen ilman käyttöliittymää

Tallennettu projekti voidaan ajaa ilman graafista käyttöliittymää:

* **CLI**: `chloros-cli project open / connect / capture / sensor / align / run` toimii projektikansion polun perusteella — katso [CLI-viite](reference/cli-reference.md).
* **SDK**: `chloros_sdk.open_project(path)` palauttaa projektikahvan; `connect_all()` ottaa kaikki tallennetut kamerat ja anturit käyttöön tallennetuilla asetuksilla — katso [SDK-viite](reference/sdk-reference.md).
