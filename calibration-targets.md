---
description: Laboratoriomitatut paneelit, joita käytetään kaapattujen tietojen kalibrointiin jälkikäsittelyssä
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibrointitavoitteet

MAPIR tarjoaa erilaisia ​​kalibrointikohteita monenlaisiin sovelluksiin. Alla näkyvä kompakti T4-R50 sisältää 4 paneelia, joiden valonheijastuskyky on mitattu välillä 250 - 2500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

T4-hajareferenssikohteissa on seuraavat heijastuskäyrät, [lataa tiedot tästä](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Heijastus :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Heijastus :: 400-1000 nm</p></figcaption></figure>

Tarkasteltaessa heijastuskäyrää voit nähdä, että arvot ovat aallonpituus (x-akseli) vs. heijastusprosentti (y-akseli). Kun otamme kuvan kalibrointikohteesta, luomme suhteen pikseliarvon ja heijastusprosenttien välille siinä spektrissä, jolle jokainen kameran anturikaista on herkkä.

Tämä tarkoittaa, että jokaisessa kamerallamme ottamasi kuvassa voit kalibroida kuvien heijastavuuden käyttämällä valokuvaa heijastuskohteistamme, kuten [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) tai [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125). Kalibroinnin jälkeen kuvan jokainen pikseli vastaa heijastusprosenttia.

Jos tulostat kalibroidut kuvat Chlorosissa tyypillisenä JPG- tai TIFF-muodossa, heijastusprosentti lasketaan jakamalla pikseliarvo kuvamuodon bittisyvyydellä. Joten jaa JPG:llä 255:llä ja TIFF:llä jaa 65 535:llä. Voit myös valita PERCENT-muotoisen lähdön Chlorosissa, jolloin kunkin pikselin prosenttiarvo vaihtelee välillä 0,0–1,0 (heijastuskyky 0–100 %). Muista vain, että jotkin kuvasovellukset eivät voi hyväksyä prosentuaalisia (liukulukuja) kuvia, ja ne ovat tallennustilan kannalta suuria.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
