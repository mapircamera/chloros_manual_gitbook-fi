# Kohdekuvien valinta

Merkitsemällä, mitkä kuvat sisältävät kalibrointikohteita, kerrot Chloros-sivustolle tarkalleen, mistä niitä tulee etsiä. Kun vähintään yksi kuva on valittu Kohde-sarakkeessa, Chloros skannaa **vain valitut kuvat** — kohteiden merkitseminen siis sekä nopeuttaa käsittelyä että estää kartoituskuvien sekoittumisen kohteisiin.

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

## Miksi kohdekuvat kannattaa merkitä?

### Merkitseminen ohjaa skannausta

Kun valitset tiettyjä kuvia Kohde-sarakkeesta:

* Chloros skannaa kohteita vain valituista kuvista
* Kohteiden tunnistus tapahtuu paljon nopeammin
* Kartoituskuvat eivät voi aiheuttaa vääriä kohdetunnistuksia

Jos **yhtään** kuvaa ei ole valittu, Chloros skannaa oletuksena kaikki projektin kuvat:

* Kohteen tunnistusalgoritmit suoritetaan jokaiselle kuvalle
* Satoja tai tuhansia kuvia tarkistetaan tarpeettomasti
* Käsittely kestää huomattavasti kauemmin, etenkin suurissa aineistoissa

{% hint style="success" %}
**Nopeuden parannus**: 2–3 kohdekuvan merkitseminen 500 kuvan aineistossa voi lyhentää kohteen tunnistusaikaa yli 30 minuutista alle 1 minuuttiin.
{% endhint %}

***

## Kohdekuvien merkitseminen

### Vaihe 1: Tunnista kohdekuvasi

Selaa tuotuja kuvia tiedostoselaimessa ja tunnista, mitkä kuvat sisältävät kalibrointikohteita.

**Yleisiä tilanteita:*** **Ennen kuvauksen aloittamista otettu kohde**: Otettu ennen istunnon aloittamista
* **Kuvauksen jälkeinen kohde**: Kuvattu istunnon päätyttyä
* **Kenttäkohteet**: Kohteet, jotka on sijoitettu kuvausalueelle
* **Useita kohteita**: 2–3 kohdekuvaa istuntoa kohti (suositeltava)

### Vaihe 2: Tarkista kohdesarake <img src="../.gitbook/assets/image (33).png" alt="" data-size="original">

Jokaiselle kuvalle, joka sisältää kalibrointikohteen:

1. Etsi kuva tiedostoselaimen taulukosta
2. Etsi **Target**-sarake (oikeanpuoleisin sarake)
3. Napsauta kyseisen kuvan kohdesarakkeen valintaruutua
4. Toista tämä kaikille kuville, joissa on kohteita

### Vaihe 3: Varmista valintasi

Tarkista ennen käsittelyä vielä kerran:

* [ ] Kaikki kuvat, joissa on kalibrointikohteita, on valittu
* [ ] Mitään kuvia, joissa ei ole kohteita, ei ole valittu vahingossa
* [ ] Kohteet näkyvät selvästi valituissa kuvissa

***

## LATTICE: Kohteet ovat valinnaisia, kun DAQ tallentaa

LATTICE-monispektrikameroissa kuvakehyksessä oleva kalibrointikohde on **toinen kahdesta** mahdollisesta heijastavuusviitteestä:

* **Kuvassa oleva kohde**: kun merkitty kohdekuva läpäisee Chloros:n laadunvalvontaportit (QA), kohteesta tulee**absoluuttinen heijastavuusviite** sitä ympäröivälle kuvamateriaalille.
* **DAQ:n alaspäin suuntautuva säteily**: kun kohdetta ei ole (tai laadunvalvonta epäonnistuu), Chloros laskee heijastavuuden sen sijaan DAQ-valosensorin alaspäin suuntautuvan säteilyn voimakkuuden perusteella (ρ = π·L/E). Jos `.daq`- tai DAQ-M `.csv`-tallennus kattaa kaappauksesi, saat kalibroidun heijastavuuden**ilman minkäänlaisia kohdekuvia**.

Tämä automaattinen toiminta on oletusasetus. CLI- ja SDK-tiedostoissa tämä vastaa `--reflectance-source auto`-asetusta; voit myös pakottaa `target`-asetuksen (tiukka — ei DAQ-korvausta) tai `daq`-asetuksen (DAQ-määräävä). Katso [CLI-viite](../reference/cli-reference.md#per-product-export-toggles-lattice-multispectral).

**LATTICE-kohteiden geometriat**: Survey3:ssa käytetyn perinteisen paneelintunnistuksen lisäksi LATTICE-käsittely tukee**ArUco-merkittyjä kohteita**,**kiinteitä ROI-kohteita**ja**nauhakohteita**, jotka määritetään projektikohtaisesti. Yksikkökohtaiset**mitatut** kohteen heijastavuusskannaukset voidaan toimittaa sarjanumeroittain (CLI: `--target-reflectance-dir`, yksi `<serial>.csv` kohdeyksikköä kohti), ja nimelliset T3/T4P-spektrit toimivat varavaihtoehtona.

{% hint style="info" %}
**F988-moduuli**: F988-heijastavuus kalibroidaan käyttämällä kuvauspaikalla olevaa heijastavuuspaneelia: kaista sijaitsee DAQ-valosensorin kalibroidun alueen ulkopuolella, joten Chloros käyttää viimeisintä paneelista tallennettua arvoa ja säilyttää sen paneelin mittausten välillä. Jos F988-moduuli käsitellään pelkästään DAQ:n avulla, Chloros hylkää kyseisen kaistan DAQ-pohjaisen heijastavuuden (ohitusperuste `dls-uncalibrated-band-988`) — paneeliprosessi on tuettu menetelmä.
{% endhint %}

***

## Kohdekuvien parhaat käytännöt

### Kohdekuvien ottamista koskevat ohjeet

**Ajoitus:**

* Ota kohdekuvia välittömästi ennen kuvausistuntoa ja sen aikana
* Samoissa valaistusolosuhteissa kuin DAQ-valosensori
* Parhaiden tulosten saavuttamiseksi kohdekuvia tulisi ihannetapauksessa ottaa mahdollisimman usein. Muussa tapauksessa valosensorin tietoja käytetään kalibroinnin säätämiseen ajan mittaan.

**Kameran sijainti:**

* Pidä kameraa kohteen yläpuolella siten, että kohde on kuvan keskellä ja täyttää noin 40–60 % kuvan keskiosasta.
* Pidä kamera yhdensuuntaisena kohteen pinnan kanssa tai sen suoraan alapuolella

**Valaistus:**

* Sama ympäristön valaistus kuin DAQ-valosensorissasi
* Vältä varjoja kohteen pinnoilla
* Älä peitä valonlähdettä kehollasi, ajoneuvollasi tai kasvillisuudella
* Pilviset olosuhteet tuottavat tasaisimmat tulokset

**Kohteen kunto:**

* Pidä kohdepaneelit puhtaina ja kuivina
* Kaikkien kohteen paneelien (esim. T4:n kaikki 4) tulee olla selvästi näkyvissä ja esteettömiä
* Kohteet mahdollisuuksien mukaan kohtisuorassa/nadir-asennossa valonlähteeseen nähden

### Kuinka monta kohdekuvaa?

**Vähimmäismäärä:**1 kohdekuva per istunto.**Suositus:** 3–5 kohdekuvaa per istunto.**Paras käytäntö aikataulun suhteen:**

* 3–5 kuvaa otetaan pian sen jälkeen, kun valosensori on alkanut tallentaa
* Käännä kameraa kuvien välillä parhaan tuloksen saavuttamiseksi
* Valinnainen: säännöllisesti istunnon aikana, jos valaistusolosuhteet muuttuvat jatkuvasti

***

## Useiden kameroiden käyttö

### Kahden kameran kokoonpanot

Jos käytät kahta MAPIR-kameraa samanaikaisesti (esim. Survey3W RGN + Survey3N OCN):

1. Ota kohdekuvia **molemmilla kameroilla** samanaikaisesti
2. Käytä **samaa fyysistä kohdetta** molemmille kameroille
3. Merkitse kohdekuvat **molemmille kameratyypeille** tiedostoselaimessa
4. Chloros käyttää sopivia kohteita kunkin kameran kalibrointiin

### Kameramalli-sarake

**Kameramalli**-sarakkeen avulla voit tunnistaa, mitkä kuvat ovat peräisin mistäkin kamerasta:

* Survey3W\_RGN
* Survey3N\_OCN
* LATT-M3M-L41-F550
* LATT-M3C-L87-FRGN
* jne.

Käytä tätä saraketta varmistaaksesi, että olet merkinnyt kohteet jokaiselle kameratyypille projektissasi.

***

## Kohteiden tunnistuksen asetukset

### Tunnistuksen herkkyyden säätäminen

Jos Chloros ei tunnista kohteitasi oikein, säädä näitä asetuksia kohdassa [Projektin asetukset](adjusting-project-settings.md):**Kalibroinnin vähimmäisnäytteen pinta-ala (px):*** **Oletus**: 25 pikseliä
* **Suurenna**, jos pienistä esineistä saadaan vääriä tunnistustuloksia
* **Pienennä**, jos kohteita ei tunnisteta**Kohteiden ryhmittelyn vähimmäisarvo (0–100):*** **Oletus**: 60
* **Lisää**, jos kohteet jakautuvat useiksi tunnistuksiksi
* **Vähennä**, jos värivaihteluita sisältäviä kohteita ei tunnisteta kokonaan

{% hint style="info" %}
**Vinkki CLI:lle**: `chloros-cli process` tukee samoja säätimiä (`--min-target-size`, `--target-clustering`), ja sen lippu `--target`/`--targets` merkitsee koko syöttökansion kohdepaneelikäyttöön tarkoitetuksi. Katso [CLI-viite](../reference/cli-reference.md).
{% endhint %}

***

## Yleisiä kohdekuvien ongelmia

### Ongelma: Kohteita ei havaittu

**Mahdolliset syyt:**

* Kohdekuvia ei ole merkitty tiedostoselaimessa
* Kohde on liian pieni kuvassa (&lt; 30 % kuvasta)
* Huono valaistus (varjot, häikäisy)
* Kohteen tunnistuksen asetukset ovat liian tiukat

**Ratkaisut:**

1. Varmista, että Kohde-sarakkeen valintaruutu on valittuna oikeiden kuvien kohdalla
2. Tarkista kohdekuvien laatu esikatselussa
3. Ota kohteet uudelleen kuvaan, jos laatu on huono
4. Säädä kohteen tunnistuksen asetuksia tarvittaessa

### Ongelma: Vääriä kohteen tunnistuksia

**Mahdolliset syyt:**

* Valkoiset rakennukset, ajoneuvot tai maaperän peitteet tulkitaan virheellisesti kohteiksi
* Kasvillisuuden kirkkaat alueet
* Tunnistuksen herkkyys liian alhainen

**Ratkaisut:**

1. Merkitse vain todelliset kohdekuvat — vain valitut kuvat skannataan
2. Suurenna kalibroinnin vähimmäisnäytealuetta
3. Suurenna kohteiden ryhmittelyn vähimmäisarvoa
4. Varmista, että kohdekuvissa näkyy vain kohde (mahdollisimman vähän taustahäiriöitä)

***

## Tarkistuslista

Ennen käsittelyn aloittamista tarkista kohdekuvien valinta:

* [ ] Vähintään yksi kohdekuva merkitty per istunto (tai LATTICE-järjestelmässä istunnon kattava `.daq`/`.csv`-tallenne)
* [ ] Kohdesarakkeen valintaruudut on valittu kaikissa kohdekuvissa
* [ ] Kohdekuvat on otettu saman ajanjakson aikana kuin tutkimus
* [ ] Kohteet näkyvät selvästi esikatselussa, kun niitä napsautetaan
* [ ] Kaikki kalibrointipaneelit näkyvät jokaisessa kohdekuvassa
* [ ] Kohteissa ei ole varjoja tai esteitä
* [ ] Kaksikamerajärjestelmässä: Kohteet on merkitty molemmille kameratyypeille

***

## Kohteettoman käsittelyn

### LATTICE: DAQ-tallenteen avulla

Jos DAQ-valosensori on tallentanut alaspäin suuntautuvaa säteilyn voimakkuutta LATTICE-kuvauksen aikana, kohdetta ei tarvita:

1. Tuo kuvamateriaalin sisältävä tiedosto `.daq` (tai DAQ-M `.csv`)
2. Jätä Kohde-sarakkeen valinta valitsematta
3. Heijastuskyky lasketaan automaattisesti DAQ:n alaspäin suuntautuvan säteilyn viitearvosta
4. Säteilyvoimakkuus ei koskaan vaadi kohdetta tai DAQ:ta — se perustuu yksinomaan kameran tehtaalla tehtyyn radiometriseen kalibrointiin

### Käsittely ilman viitteitä

Voit käsitellä aineistoa myös ilman kohteita ja ilman DAQ:ta:

1. Jätä kaikki Kohde-sarakkeen valintaruudut valitsematta
2. **Poista käytöstä** ”Heijastavuuskalibrointi / valkotasapaino” Projektin asetuksissa – kohteen tunnistus ohitetaan tällöin kokonaan
3. Vignettokorjaus sovelletaan edelleen
4. Tulostetta ei kalibroida absoluuttisen heijastavuuden suhteen (LATTICE multispektraali vie edelleen debayeroidut, esikatselu- ja säteilyvoimakkuustuotteet)

{% hint style="warning" %}
**Ei suositella tieteelliseen työhön (Survey3)**: Ilman heijastavuuskalibrointia Survey3 pikseliarvot edustavat vain suhteellista kirkkautta, eivät tieteellisiä heijastavuusmittauksia. Käytä kalibrointikohteita (tai LATTICE-ohjelmistossa DAQ-valosensoria) tarkkojen ja toistettavien tulosten saamiseksi.
{% endhint %}

***

## Seuraavat vaiheet

Kun olet merkinnyt kohdekuvasi:

1. **Tarkista asetuksesi** – Katso [Projektin asetusten säätäminen](adjusting-project-settings.md)
2. **Aloita käsittely** – Katso [Käsittelyn aloittaminen](starting-the-processing.md)
3. **Seuraa edistymistä** – Katso [Käsittelyn seuranta](monitoring-the-processing.md)

Lisätietoja kalibrointikohteista itsestään on kohdassa [Kalibrointikohteet](../calibration-targets.md).
