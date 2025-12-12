# Indeksi/LUT-hiekkalaatikko

Indeksi/LUT-hiekkalaatikko on interaktiivinen työtila Chloros-kuvankatseluohjelmassa, jonka avulla voit kokeilla monispektrisen indeksin laskemista ja värien visualisointia reaaliajassa. Tämän tehokkaan työkalun avulla voit testata erilaisia indeksejä, tarkentaa arvoalueita ja luoda julkaisukelpoisia visualisointeja ilman, että sinun tarvitsee käsitellä koko tietojoukkoasi uudelleen.

## Mikä on indeksi/LUT-hiekkalaatikko?

### Tarkoitus

Hiekkalaatikko tarjoaa:

* **Reaaliaikaisen indeksilaskennan** – sovella mitä tahansa kasvillisuusindeksiä välittömästi
* **Interaktiivisen LUT-säätämisen** – hienosäädä värigradientteja ja -alueita
* **Työnkulun optimoinnin** – määritä parhaat asetukset ennen eräprosessointia

### Sandbox vs. projektin käsittely

**Index/LUT Sandbox (interaktiivinen):**

* Yksi kuva kerrallaan
* Välitön palaute
* Kokeellinen ja iteratiivinen
* Ei pysyviä muutoksia tiedostoihin
* Täydellinen tutkimiseen ja testaamiseen

**Projektin käsittely (erä):**

* Koko tietojoukko kerralla
* Esiasetetut asetukset
* Pysyvät tulostustiedostot
* Aikaa vievä
* Paras, kun asetukset on vahvistettu

{% hint style=&quot;success&quot; %}
**Paras työnkulku**: Käytä Sandboxia kokeilemiseen ja optimaalisten indeksi- ja LUT-asetusten löytämiseen, ja käytä sitten näitä asetuksia projektin käsittelyn aikana koko tietojoukkoon.
{% endhint %}

***

## Indeksi/LUT Sandboxin käyttö

### Ennalta laskettujen indeksien ymmärtäminen

Chloros:ssä indeksejä voidaan soveltaa projektin käsittelyn aikana. Helpoin tapa määrittää, mitä indeksi- ja LUT-asetuksia haluat soveltaa vientiin, on käyttää kuvankatseluohjelman sandboxia.

Sandboxin avulla voit:

* **Sovelletaan uusia indeksejä ja värigradientteja (LUT)** tietojen visualisointiin.
* **Säädetään visualisointiasetuksia** interaktiivisesti.
* **Tarkastellaan** jo laskettuja indeksikuvia.
* **Tarkastellaan** pikseliarvoja kaikilla zoomausasteilla.

### Sandboxin avaaminen

Indeksi/LUT-sandbox avataan **kuvankatseluohjelman** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sivupalkin välilehdestä:

1. Napsauta kuvaa tiedostoselaimen kuvaruudukossa, jolloin se avautuu **kuvankatseluohjelman** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> -välilehdessä
2. Napsauta **Kuvankatseluohjelman** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> -välilehteä avataksesi vasemman sivupalkin, jos se ei ole vielä auki

### Valitse kuva, johon haluat soveltaa indeksiä/LUT:ia

Voit työskennellä indeksin kanssa Image Viewer <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sandboxissa:

1. **Avaa kuva** pääkuvaristikosta napsauttamalla sitä
2. **Kuvankatseluohjelman** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> -välilehti avautuu
3. Napsauta **Kerros-pudotusvalikkoa** (katselijan oikeassa yläkulmassa)
4. Valitse kerros pudotusvalikosta:
   * RAW (heijastavuus)

### Indeksin soveltaminen kuvaan

Kun kuva on koko ruudulla ja **Kuvankatseluohjelman** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> -välilehden sivupalkki on auki:

1. Valitse sivupalkin yläosassa oleva Indeksi-ruutu.
2. Valitse kamerasi suodatin vasemmasta pudotusvalikosta.
3. Valitse haluamasi indeksikaava oikeasta pudotusvalikosta.
4. Vedä suodatinkanavan väripiirit alla olevan indeksikaavan paikoille.
5. Kun kaava on kelvollinen, kuva päivittyy ja näyttää indeksiarvot.
6. Siirrä hiiren osoitinta nähdäksesi osoittimen sijainnin arvot.
7. Zoomaa nähdäksesi yksittäiset pikselit ja niihin liittyvät arvot.

Jokaisella indeksillä on tietty arvoalue ja merkitys:

#### NDVI Esimerkki

```
Formula: (NIR - Red) / (NIR + Red)

For Survey3W RGN camera:
NIR = 850nm band
Red = 661nm band

Result range: -1.0 to +1.0
Typical vegetation: 0.4 to 0.9
Stressed vegetation: 0.2 to 0.4
Bare soil: 0.0 to 0.2
Water: -0.1 to 0.1
```

Täydelliset indeksikaavojen dokumentit ovat kohdassa [Multispektriset indeksikaavat](../project-settings/multispectral-index-formulas.md).

***

## LUT-taulukoiden (Look-Up Tables) käyttö

### Mikä on LUT?

**Look-Up Table (LUT)** -taulukko yhdistää numeeriset indeksiarvot väreihin visualisointia varten:

* **Syöte**: Indeksin pikseliarvo (esim. NDVI 0,65)
* **Lähtö**: RGB väri (esim. kirkkaanvihreä)
* **Tarkoitus**: Helpottaa kuvioiden näkemistä ja tulkintaa

**Harmaasävy vs. väri-LUT:**

* Harmaasävy: Tieteellinen ja neutraali, näyttää raakadatan
* Väri-LUT: Intuitiivinen ja vaikuttava, korostaa kuvioita ja eroja

{% hint style=&quot;success&quot; %}
**Visualisointivoima**: Värillisen LUT:n soveltaminen harmaasävyiseen indeksikuvaan helpottaa huomattavasti kuvioiden, poikkeamien ja kiinnostavien alueiden tunnistamista yhdellä silmäyksellä.
{% endhint %}

### LUT:n soveltaminen indeksikuvaan

Kun sinulla on indeksikuva, joka näyttää

1. Napsauta <img src="../.gitbook/assets/image.png" alt="" data-size="line"> &quot;+Lisää LUT&quot; -painiketta
2. Valitse värigradientti
3. Säädä leikkauksen minimi-/maksimipisteitä
4. Säädä leikkausmoodia
5. Valitse **Kuvankatseluohjelman** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> -välilehden sivupalkissa LUT:n soveltamiseksi.

### Värigradientin valitseminen

**Gradientin valitseminen:**

1. Etsi LUT-paneelista **värillinen gradienttipalkki**
2. Vie hiiri sen päälle nähdäksesi käytettävissä olevat gradienttivalinnat
3. Valitse haluamasi gradientti
4. Kuva **päivittyy välittömästi** uusilla väreillä, kun Indeksi-ruutu on valittuna

{% vihje style=&quot;success&quot; %}
**Paras käytäntö**: Kasvillisuusindekseissä, kuten NDVI, Red-Yellow-Green-gradientti on intuitiivisin, koska se vastaa luonnollisia väriyhdistelmiä (vihreä = terve, keltainen = kohtalainen, punainen = stressaantunut).
{% endhint %}

### Väriluokkien säätäminen

**Luokkien säätö** määrittää, kuinka monta erillistä värivaihetta gradientissa näkyy:

**Luokkien lukumäärän vaihtoehdot:**

* **2–5 luokkaa**: Erittäin laajat kategoriat, erilliset vyöhykkeet
* **6–10 luokkaa**: Tasapainoinen, sopii luokitteluun
* **11–20 luokkaa**: Tasaiset gradientit, jatkuva ulkonäkö
* **20+ luokkaa**: Lähes jatkuva, maksimaalinen tasaisuus

**Säätöohjeet:**

1. Etsi LUT-paneelista **värinäytteen ruudut gradienttipalkin alapuolelta**
2. Säädä luokkien määrää lisäämällä niitä +-painikkeella
3. Poista luokkien lukumäärä kaksoisnapsauttamalla värinäytettä
4. Gradientti päivittyy **reaaliajassa** kuvassa

**Vaikutus visualisointiin:**

* **Vähemmän luokkia** (3–5): Luo erilliset vyöhykkeet, yksinkertaistettu luokittelu, helpompi erottaa kategoriat
* **Keskimääräinen luokkien määrä** (6–10): Tasapainoinen lähestymistapa, sopii useimpiin sovelluksiin
* **Enemmän luokkia** (15–20): Sujuvat siirtymät, yksityiskohtaiset vaihtelut, valokuvamainen ulkonäkö

**Käyttötarkoitukset:**

* **Vähän luokkia (3–5)**: Esitysdia, luokittelukartat, yksinkertaiset raportit
* **Keskimääräinen määrä luokkia (6–10)**: Yleinen analyysi, tasapainoinen yksityiskohtaisuus, vakiomuotoiset raportit
* **Monet luokat (15–20)**: Tieteellinen analyysi, yksityiskohtainen tarkastus, julkaisukelpoiset tulokset

### Arvoalueiden hienosäätö

**Arvoalueiden säätimet** määrittävät, mitkä indeksiarvot vastaavat gradientin värejä:

**Alueiden säätimet LUT-paneelissa:**

* **Minimiarvo**: Väriskaalan alaraja
* **Maksimiarvo**: Väriskaalan yläraja
* **Välivärit**: Jaetaan automaattisesti minimin ja maksimiarvon välillä (luokkien lukumäärän perusteella)

#### Minimi-/maksimiarvojen säätäminen

**Arvoalueiden säätäminen:**

1. Etsi LUT-paneelista **Minimiarvo**- ja **Maksimiarvo**-syöttökentät
2. Napsauta **Minimiarvo**-kenttää
3. Kirjoita haluttu minimiarvo (esim. `0.2`)
4. Paina **Enter** tai napsauta kentän ulkopuolella
5. Toista sama **Maksimiarvo**-kentässä (esim. `0.9`)
6. Visualisointi **päivittyy välittömästi**

{% vihje style=&quot;info&quot; %}
**Automaattinen skaalaus**: Kun käytät LUT:ia ensimmäisen kerran, Chloros asettaa automaattisesti minimi-/maksimiarvon kuvan todelliseen data-alueeseen. Voit sitten kaventaa tätä aluetta keskittyäksesi tiettyihin kiinnostaviin arvoalueisiin.
{% endhint %}

**Esimerkki NDVI-alueen säätämisestä:**

* **Koko alue**: `-1.0` – `1.0` (näytä kaikki mahdolliset arvot)
* **Kasvillisuuteen keskittynyt**: `0.2` – `0.9` (poista paljas maaperä ja vesi)
* **Vain terve kasvillisuus**: `0.5` – `0.9` (korosta vain voimakkaasti kasvavat kasvit)
* **Stressin havaitseminen**: `0.2` – `0.5` (korosta ongelma-alueet)
* **Mukautettu alue**: Säädä havaittujen pikseliarvojen perusteella

**Miksi alueita tulisi säätää?**

* **Kontrastin lisääminen** kiinnostavalla alueella
* **Poista merkityksettömät arvot** (esim. vesistöt, paljas maaperä)
* **Yhtenäistä visualisointi** useiden kuvien tai päivämäärien välillä
* **Korosta hienovaraiset erot** kapealla arvoalueella

### Leikkaa alueelle kuulumattomat arvot

Kun pikseliarvot eivät kuulu määrittelemääsi minimi-/maksimialueeseen, voit hallita niiden näyttötapaa **leikkaustiloilla**.

#### **Käytettävissä olevat leikkaustilan vaihtoehdot:**

#### 1. Minimi ja maksimi

* Pikselit **minimin alapuolella** → näytetään gradientin **ensimmäisellä värillä** (esim. punaisella)
* Pikselit **maksimin yläpuolella** → näytetään gradientin **viimeisellä värillä** (esim. vihreällä)
* **Käyttötapaus**: Korosta ääripäät, näytä koko data-alue saturoiduilla väreillä rajoissa
* **Esimerkki**: NDVI-arvot alle 0,2 näkyvät punaisina, arvot yli 0,9 näkyvät vihreinä

#### 2. Läpinäkyvä tausta

* **Alueen ulkopuolella** olevat pikselit muuttuvat **täysin läpinäkyviksi**
* Vain **alueen sisällä** olevat pikselit näyttävät värigradientin
* **Käyttötapaus**: GIS-peittokuva, tiettyjen arvoalueiden eristäminen, vain kiinnostavien alueiden korostaminen
* **Esimerkki**: Näytä vain NDVI 0,4–0,7 värillisenä, kaikki muu läpinäkyvänä

{% hint style=&quot;warning&quot; %}
**Läpinäkyvyyden rajoitus**: Läpinäkyvät pikselit näkyvät katseluohjelmassa taustavärinä. Kun ne viedään käsittelyn aikana, läpinäkyvyys säilyy PNG-muodossa, mutta ei JPG-muodossa.
{% endhint %}

#### 3. Indeksin tausta

* **Alueen ulkopuolella** olevat pikselit näkyvät **harmaasävyisinä** (näyttävät raakaindeksiarvot)
* **Alueella** olevat pikselit näyttävät **värigradientin**
* **Käyttötapaus**: Hienovarainen korostus, kontekstin säilyttäminen ja kiinnostavien alueiden korostaminen
* **Esimerkki**: Korosta stressaantunut kasvillisuus värillä (NDVI 0,3–0,5) ja näytä terveet alueet harmaana

#### 4. Alkuperäinen tausta

* **Alueen ulkopuolella** olevat pikselit näyttävät **alkuperäisen monispektrikuvan**
* **Alueella** olevat pikselit näyttävät **värigradientin**
* **Käyttötapaus**: Intuitiivisin – yhdistää luonnollisen kuvakontekstin analyyttiseen värikerrokseen
* **Esimerkki**: Katso todellinen pellon/sadon ulkonäkö värikoodatuilla stressialueilla

### Oikean leikkaustilan valinta

| Leikkaustila              | Paras käyttötarkoitus                                   | Visualisointityyli          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimi ja maksimi**    | Täydellinen tietojen näyttö, tieteellinen analyysi     | Kaikki pikselit värillisiä           |
| **Läpinäkyvä tausta** | GIS-peittokuvat, tiettyjen alueiden eristäminen    | Väri alueella, tyhjä sen ulkopuolella |
| **Indeksitausta**       | Hienovarainen korostus, tietojen kontekstin säilyttäminen  | Väri alueella, harmaa sen ulkopuolella  |
| **Alkuperäinen tausta**    | Raportit, esitykset, intuitiivinen analyysi | Väri alueella, valokuva sen ulkopuolella |

### Mukautettujen LUT-värien luominen

Voit hallita visualisointia täysin luomalla **mukautettuja värigradientteja** muokkaamalla yksittäisiä väripysähdyksiä.

**Mukautetun gradientin luominen:**

1. Etsi LUT-paneelista **gradientin esikatselupalkki**
2. Etsi **värinäytteen neliöt** gradientin alapuolelta
3. **Napsauta väripysähdystä** valitaksesi sen
4. **Värivalitsin** avautuu.
5. Valitse uusi väri käyttämällä:
   * **Värirengasta**: Visuaalinen värivalinta.
   * **RGB/HSV-liukusäätimiä**: Tarkka värinhallinta.
   * **Hex-koodin syöttöä**: Tarkka värimäärittely (esim. `#FF0000` punaiselle).
6. Napsauta värivalitsimen ulkopuolella **sovellaksesi uuden värin**
7. Gradientti **päivittyy välittömästi** kuvaan

**Värin pysäytysten lisääminen tai poistaminen:**

* **Lisää pysäytys**: Napsauta +-kuvaketta lisätäksesi uuden värinäytteen loppuun
* **Poista pysäytys**: Kaksoisnapsauta väriruutua poistaaksesi värinäytteen

**Mukautusstrategiat:**

* **Käännä gradientti**: Käännä värien järjestys päinvastaiseksi (esim. vihreä = matala, punainen = korkea)
* **Brändivärit**: Sovita raporttien väripaletti organisaatiosi väripalettiin
* **Värisokeille sopiva**: Käytä oranssi-sinisiä tai violetti-keltaisia yhdistelmiä
* **Tulostuksen optimointi**: Valitse värit, jotka toimivat sekä väri- että harmaasävyisessä tulostuksessa
* **Monikynnys**: Käytä erillisiä värejä tiettyjen arvojen kynnyksillä luokittelua varten

{% vihje style=&quot;info&quot; %}
**Mukautettujen gradienttien tallentaminen**: Mukautetut gradientit voidaan tallentaa ja käyttää uudelleen. Napsauta LUT-paneelin tallennuskuvaketta, jotta mukautetut värimaailmat säilyvät tulevaa käyttöä varten.
{% endhint %}

***

## Interaktiivinen työnkulku

### Reaaliaikaiset päivitykset

Kaikki sandboxin LUT-säädöt päivittävät kuvan **välittömästi ja interaktiivisesti**:

* **Vaihda taso** → Kuva muuttuu välittömästi
* **Valitse gradientti** → Värit päivittyvät välittömästi
* **Säädä arvoaluetta** → Kontrasti muuttuu reaaliajassa
* **Vaihda luokkia** → Gradientin tasaisuus päivittyy välittömästi
* **Muokkaa leikkausta** → Taustan näyttö muuttuu välittömästi
* **Muokkaa värejä** → Mukautettu gradientti otetaan käyttöön välittömästi

**Ei tarvitse &quot;Käytä&quot;-painiketta** – kaikki muutokset ovat reaaliaikaisia ja interaktiivisia!

{% hint style=&quot;success&quot; %}
**Reaaliaikainen palaute**: Välittömän visuaalisen palautteen ansiosta voit kokeilla nopeasti erilaisia asetuksia, kunnes löydät analyysitarpeisiisi parhaiten sopivan visualisoinnin.
{% endhint %}

### Iteratiivinen hienosäätötyönkulku

**Tyypillinen LUT-optimoinnin työnkulku:**

1. **Valitse indeksikerros** (esim. RAW (heijastavuus))
2. **Käytä indeksiä** – Valitse kamerasuodatin ja indeksikaava, vedä värilliset ympyrät sopivaan kohtaan indeksikaavassa
3. **Käytä LUT-gradienttia** – Aloita Red-Yellow-Green-esiasetuksella
4. **Tarkista pikseliarvot** - Siirrä kursoria ja merkitse muistiin arvoalueet
5. **Säädä min/max** - Rajaa keskittymään kasvillisuuteen (esim. 0,2–0,9)
6. **Valitse leikkaus** - Kokeile &quot;Original Background&quot; (Alkuperäinen tausta) kontekstia varten
7. **Hio värejä** - Mukauta gradienttia tarvittaessa tiettyjen korostusten tekemiseksi
8. **Viimeistele asetukset** - Dokumentoi asetukset ja kopioi ne projektin asetuksiin vientiä varten

### Pikseliarvojen tarkastus

Todellisten pikseliarvojen ymmärtäminen on tärkeää tehokkaiden LUT-alueiden asettamiseksi:

**Arvojen tarkastaminen:**

1. Pikseliarvot näkyvät, kun kuvassa on joko indeksi tai sekä indeksi että LUT **valittuna**.
2. **Siirrä kursori** kuvan eri alueiden päälle
3. **Tarkkaile pikseliarvoja**, jotka näkyvät selitteessä, kun viet kursorin niiden päälle
4. Zoomaa sisään nähdäksesi yksittäiset pikselit, jotka on korostettu kelluvalla arvolla
5. **Tee muistiinpanoja** eri ominaisuuksien arvoalueista:
   * **Terve kasvillisuus**: esim. NDVI 0,55–0,85
   * **Stressaantunut kasvillisuus**: esim. NDVI 0,30–0,50
   * **Paljas maaperä**: esim. NDVI 0,05–0,25
   * **Vesi** (jos läsnä): esim. NDVI -0,05–0,10

**Pikseliarvojen käyttäminen LUT-alueiden asettamiseen:**

Tarkastettuasi pikseliarvot, säädä LUT:n minimi- ja maksimiarvot vastaavasti:

**Esimerkkitilanne:**

* **Havainto**: Maaperän arvot = 0,05–0,25, stressaantunut = 0,25–0,50, terve = 0,50–0,85
* **Tavoite**: Visualisoi vain kasvien terveys (poista maaperä)
* **LUT-asetukset**: Min = `0.25`, Max = `0.85`
* **Leikkaus**: &quot;Alkuperäinen tausta&quot; maaperän näkemiseksi luonnollisessa värissä
* **Tulos**: Värigradientti koskee vain kasvillisuutta, maaperä näkyy alkuperäisenä kuvana

{% hint style=&quot;info&quot; %}
**Dynaaminen alue**: Eri viljelykasveilla, vuodenaikoilla ja kasvuvaiheilla on erilaiset arvoalueet. Tarkista aina pikseliarvot omassa tietojoukossasi ennen LUT-alueiden asettamista.
{% endhint %}

***

## Mukautetut indeksit (Chloros+)

### Mukautettujen indeksikaavojen luominen

{% hint style=&quot;info&quot; %}
**Luominen**: Mukautetut indeksit voidaan määrittää **Projektin asetuksissa** ennen käsittelyä sekä Image Viewer -selaimen sivupalkissa.
{% endhint %}

**Mukautetun indeksin luominen:**

1. **Avaa Projektin asetukset** (ennen käsittelyä) tai Image Viewer -selaimen sivupalkki
2. Siirry **Indeksikaavan pudotusvalikkoon**
3. Etsi **&quot;Mukautettu&quot;**-vaihtoehto (sinun on oltava kirjautuneena sisään Chloros+-lisenssillä)
4. **Määritä kaava** käyttämällä kaistamuuttujia:
   * Kaistojen nimet: `NIR`, `Red`, `Green`, `Blue`, `RedEdge` jne.
   * Operaattorit: `+`, `-`, `*`, `/`, `^` (eksponentti)
   * Funktiot: `sqrt()`, `abs()` jne. (jos tuettu)
   * Suluissa: `()` laskujärjestyksen määrittämiseksi
5. **Nimeä indeksi** (esim. &quot;MyIndex&quot; tai &quot;CustomNDVI&quot;)
6. **Tallenna kokoonpano**

**Esimerkkejä mukautetuista kaavoista:**

```
Modified NDVI with offset:
(NIR - Red) / (NIR + Red + 0.5)

Simple ratio:
NIR / Red

Complex multi-band:
(NIR - Red) / (NIR + Red - Blue)

Exponential index:
(NIR / Red) ^ 2
```

{% hint style=&quot;warning&quot; %}
**Kaavan vahvistus**: Varmista, että kaavassasi käytetään kamerassasi käytettävissä olevia kaistoja. Esimerkiksi RedEdge on käytettävissä vain kameroissa, joissa on RedEdge-suodatin.
{% endhint %}

***

## Seuraavat vaiheet

Nyt kun ymmärrät indeksi/LUT-hiekkalaatikon:

* **Sovella käsittelyyn**: Käytä löydettyjä asetuksia [Projektin asetuksissa](../project-settings/project-settings.md)
* **Erän käsittely**: Käytä optimoituja indeksejä koko tietojoukkoon
* **Lisätietoja**: Lue [Monispektriset indeksikaavat](../project-settings/multispectral-index-formulas.md)

Aiheeseen liittyvä dokumentaatio:

* [**Kuvakerrokset**](image-layers.md) - Kerrosten hallinta ja visualisointi
* [**Kuvan avaaminen koko ruudulla**](opening-an-image-full-screen.md) – Kuvankatseluohjelman perusteet
* [**Kuvien käsittely (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Koko käsittelyprosessi
