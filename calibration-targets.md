---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibrointikohteet

MAPIR tarjoaa erilaisia kalibrointikohteita monenlaisiin käyttötarkoituksiin. Alla näkyvä kompakti T4-R50-malli sisältää neljä paneelia, joiden valonheijastavuus on mitattu aallonpituusalueella 250–2 500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>T4-diffuusiviitekohteilla on seuraavat heijastuskäyrät, [tiedot ladattavissa täältä](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Heijastavuus :: 250–2 500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4:n heijastavuus :: 400–1 000 nm</p></figcaption></figure>T4P-diffuusiovertailukohteilla on seuraavat heijastuskäyrät, [tiedot ladattavissa täältä](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P:n heijastavuus :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P-heijastavuus :: 400–1000 nm</p></figcaption></figure>Heijastavuuskuviosta näet, että arvot esitetään aallonpituuden (x-akseli) ja heijastavuusprosentin (y-akseli) suhteena. Kun otamme kuvan kalibrointikohteesta, luomme suhteen pikseliarvon ja heijastavuusprosentin välille kunkin kameran anturikaistan herkkyysalueella.

Tämä tarkoittaa, että jokaisen kameroillamme ottamasi kuvan yhteydessä voit käyttää valokuvia heijastavuuskohteistamme, kuten [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) tai [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), kuvien heijastavuuden kalibrointiin. Kalibroinnin jälkeen kuvan jokainen pikseli vastaa heijastavuusprosenttia.

**Survey3**-tulostusten osalta, jos tulostat kalibroidut kuvat Chloros-ohjelmassa tavallisina JPG- tai TIFF-tiedostoina, heijastusprosentti lasketaan jakamalla pikseliarvo kuvamuodon bittisyvyydellä. JPG-tiedostojen kohdalla jaetaan siis 255:llä ja TIFF-tiedostojen kohdalla 65 535:llä. Voit myös valita Chloros-ohjelmassa PERCENT-tiedostomuodon, jolloin kunkin pikselin arvo vaihtelee välillä 0,0–1,0 (0 %–100 % heijastavuus). Muista kuitenkin, että jotkin kuvankäsittelyohjelmat eivät tue prosenttiarvoisia (liukuluku) kuvia, ja ne vievät paljon tallennustilaa.

{% hint style="info" %}
**LATTICE-heijastavuus käyttää erilaista pikseliasteikkoa.** LATTICE-heijastavuus tallennetaan siten, että DN 32768 = 100 % heijastavuus (ei 65 535), ja jokaisessa tiedostossa on XMP-tunniste `Chloros:PixelScale`, joka ilmoittaa sen asteikon. Lue tunniste ja jaa arvo sen mukaan sen sijaan, että olettaisit vakioarvon — katso [Lähtökuvamuodot](output-image-formats.md).
{% endhint %}

## Kalibrointikohteet LATTICE-kameroilla

LATTICE-kameroilla kalibrointikohde on **valinnainen** heijastavuuden osalta: Chloros voi sen sijaan viitata heijastavuuteen DAQ-valosensorin mittaaman alaspäin suuntautuvan säteilyn voimakkuuden perusteella (ρ = π·L/E). Vertailuarvo valitaan projektin heijastavuuslähteen asetuksella, joka määritetään kohdassa `chloros-cli process --reflectance-source` (käyttöliittymässä ei ole tätä koskevaa säätöä, vaan se käyttää projektin tallennettua arvoa, oletusarvoisesti `auto` — katso [Projektin asetukset](project-settings/project-settings.md)):

| Arvo | Toiminta |
| --- | --- |
| `auto` *(oletus)* | Laadunvarmistuksen läpäissyt kehyksessä oleva kohde on **absoluuttinen vertailuarvo**; jos kohdetta ei ole tai laadunvarmistus epäonnistuu, Chloros käyttää varajärjestelmänä DAQ:n alaspäin suuntautuvaa jakolinjaa. Kohde, joka läpäisee laadunvarmistuksen (QA) lippun kanssa (paneeli pudonnut tai kaista epäkunnossa), ei tule automaattisesti ensisijaiseksi: kohde-, kohde+DAQ- ja pelkkä DAQ-ketjut kokeillaan niiden mitatun pistemäärän mukaisessa järjestyksessä testipaneeleilla. |
| `target` | Tiukka kohde-vain — ei DAQ-korvausta. |
| `daq` | DAQ-määräysvaltainen — alaspäin suuntautuva mittaus on aina vertailukohta. |

Lisätoimintoja kohteille LATTICE-ohjelmistossa:

* **Kohteiden geometriat** — ArUco-merkityt paneelit, kiinteän ROI-alueen paneelit ja nauhakohteet ovat kaikki tuettuja; geometria määritetään projektin kohdekonfiguraatiosta.
* **Yksikkökohtaisesti mitatut kohdetiedot** — `--target-reflectance-dir DIR` osoittaa hakemistoon, jossa on yksikkökohtaisesti mitatut kohteen heijastavuusskannaukset (`<serial>.csv`, haetaan kohdeyksikön sarjanumeron tai QR-koodin perusteella). Jos kohdetta ei löydy, Chloros käyttää varamenetelmänä nimellisiä T3/T4P-spektrejä.
* **Ajallinen ankkurointi** — havaittu kohde kalibroi sitä ympäröivät kehykset ja pidetään paikallaan kohteen havaintojen välillä.

Täydelliset lippujen merkitykset ja esimerkit löytyvät [CLI-viitteestä](reference/cli-reference.md) (katso ”Tuotteittaiset vientikytkimet”).

### F988

”F988-heijastavuus kalibroidaan käyttämällä kuvauspaikalla olevaa heijastavuuspaneelia: kaista sijaitsee DAQ-valosensorin kalibroidun alueen ulkopuolella, joten Chloros käyttää viimeisintä paneelista tallennettua arvoa ja pitää sen voimassa paneelin havaintojen välillä.”

Jos F988 suoritetaan pelkästään DAQ-kalibroinnilla, Chloros hylkää kyseisen kaistan DAQ-pohjaisen heijastavuuden ja ilmoittaa syyn (ohitusperuste `dls-uncalibrated-band-988`); paneelipohjainen työnkulku on tuettu menetelmä.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
