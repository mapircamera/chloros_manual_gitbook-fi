---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibrointikohteet

MAPIR tarjoaa erilaisia kalibrointikohteita monenlaisiin sovelluksiin. Alla näkyvä kompakti T4-R50 sisältää 4 paneelia, joiden valonheijastavuus on mitattu aallonpituusalueella 250–2 500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>T4-diffuusiviitekohteilla on seuraavat heijastavuuskäyrät, [tiedot ladattavissa täältä](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4-heijastavuus :: 250–2 500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4-heijastavuus :: 400–1 000 nm</p></figcaption></figure>T4P-diffuusi-vertailukohteilla on seuraavat heijastuskäyrät, [tiedot ladattavissa täältä](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P-heijastavuus :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P-heijastavuus :: 400–1000 nm</p></figcaption></figure>Heijastavuuskaaviosta näet, että arvot esittävät aallonpituuden (x-akseli) suhteessa heijastavuusprosenttiin (y-akseli). Kun otamme kuvan kalibrointikohteesta, luomme suhteen pikseliarvon ja heijastavuusprosentin välille sen spektrin alueella, jolle kunkin kameran anturikaistan herkkyys ulottuu.

Tämä tarkoittaa, että jokaisen kameroillamme ottamasi kuvan yhteydessä voit käyttää valokuvaa heijastavuuskohteistamme, kuten [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) tai [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), kuvien heijastavuuden kalibrointiin. Kalibroinnin jälkeen kuvan jokainen pikseli vastaa heijastavuusprosenttia.

**Survey3** -tulostuksissa, jos tulostat kalibroidut kuvat tiedostomuodossa Chloros tavallisena JPG-kuvana tai tiedostomuodossa TIFF, heijastavuusprosentti lasketaan jakamalla pikseliarvo kuvamuodon bittisyvyydellä. JPG-muodossa jaetaan siis 255:llä ja TIFF-muodossa jaetaan 65 535:llä. Voit myös valita PERCENT-tiedostomuodon tulostuksen tiedostossa Chloros, jolloin kunkin pikselin arvo vaihtelee välillä 0,0–1,0 (heijastavuus 0 % – 100 %). Muista kuitenkin, että jotkut kuvankäsittelyohjelmat eivät tue prosenttiarvoisia (liukuluku) kuvia, ja ne vievät paljon tallennustilaa.

{% hint style="info" %}
**LATTICE-heijastavuus käyttää erilaista pikseliasteikkoa.** LATTICE-heijastavuus tallennetaan siten, että DN 32768 = 100 % heijastavuus (ei 65535), ja jokaisessa tiedostossa on XMP-tunniste `Chloros:PixelScale`, joka ilmoittaa sen asteikon. Lue tunniste ja jaa arvo sen mukaan sen sijaan, että olettaisit vakioarvon — katso [Tulostuskuvamuodot](output-image-formats.md).
{% endhint %}

## Kalibrointikohteet LATTICE-kameroilla

LATTICE-kameroilla kalibrointikohde on **valinnainen** heijastavuuden osalta: Chloros voi sen sijaan viitata heijastavuuteen DAQ-valosensorilla mitattuun alaspäin suuntautuvaan säteilyvoimakkuuteen (ρ = π·L/E). Viite valitaan heijastavuuslähteen asetuksella (Projektin asetukset käyttöliittymässä; `--reflectance-source` kohdassa CLI; `reflectance_source` kohdassa SDK):

| Arvo | Käyttäytyminen |
| --- | --- |
| `auto` *(oletus)* | Laadunvarmistuksen läpäissyt kehyksessä oleva kohde on **absoluuttinen vertailuarvo**; jos kohdetta ei ole tai laadunvarmistus epäonnistuu, Chloros käyttää varajärjestelmänä DAQ:n alaspäin suuntautuvaa jakosuhdetta. |
| `target` | Vain kohde — ei DAQ-korvausta. |
| `daq` | DAQ-määräysvaltainen — alaspäin suuntautuva mittaus on aina vertailukohta. |

Lisäkohdekäyttäytyminen LATTICE:lle:

* **Kohteen geometriat** — ArUco-merkityt paneelit, kiinteän ROI:n paneelit ja nauhakohteet ovat kaikki tuettuja; geometria tulee projektin kohdekonfiguraatiosta.
* **Yksikkökohtaisesti mitatut kohdetiedot** — `--target-reflectance-dir DIR` osoittaa hakemistoon, jossa on yksikkökohtaisesti mitatut kohteen heijastavuusskannaukset (`<serial>.csv`, haetaan kohteen yksikön sarjanumeron tai QR-koodin perusteella). Jos kohdetta ei löydy, Chloros käyttää varamenetelmänä nimellisiä T3/T4P-spektrejä.
* **Ajallinen ankkurointi** — havaittu kohde kalibroi sitä ympäröivät kehykset ja pidetään paikallaan kohteen havaintojen välillä.

Täydelliset lippusemantiikat ja esimerkit löytyvät [CLI-viitteestä](reference/cli-reference.md) (katso ”Tuotteittaiset vientikytkimet”).

### F988

&quot;F988-heijastavuus kalibroidaan käyttämällä kuvauskohteessa olevaa heijastavuuspaneelia: kaista sijaitsee DAQ-valosensorin kalibroidun alueen ulkopuolella, joten Chloros soveltaa viimeisintä paneelista tallennettua arvoa ja pitää sitä voimassa paneelin havaintojen välillä.&quot;

Jos F988 suoritetaan pelkästään DAQ-kalibroinnilla, Chloros hylkää kyseisen kaistan DAQ-pohjaisen heijastavuuden ja ilmoittaa syyn (ohitussyyn `dls-uncalibrated-band-988`); paneeliprosessi on tuettu menetelmä.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
