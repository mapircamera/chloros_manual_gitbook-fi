---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibrointikohteet

MAPIR tarjoaa erilaisia kalibrointikohteita monenlaisiin sovelluksiin. Alla näkyvä kompakti T4-R50 sisältää 4 paneelia, joiden valonheijastavuus on mitattu välillä 250–2 500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

T4-diffuusi vertailukohteilla on seuraavat heijastavuuskäyrät, [lataa tiedot täältä](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4-heijastavuus :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Heijastavuus :: 400–1000 nm</p></figcaption></figure>

Heijastavuusgraafista näet, että arvot ovat aallonpituus (x-akseli) suhteessa heijastavuusprosenttiin (y-akseli). Kun otamme kuvan kalibrointikohteesta, luomme suhteen pikseliarvon ja heijastavuusprosentin välille kameran kunkin anturikaistan herkkyysalueella.

Tämä tarkoittaa, että jokaisessa kamerallamme ottamassasi kuvassa voit käyttää valokuvaa heijastavuuskohteistamme, kuten [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) tai [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), kuvien heijastavuuden kalibroimiseksi. Kalibroinnin jälkeen kuvan jokainen pikseli vastaa heijastavuusprosenttia.

Jos tulostat kalibroidut kuvat Chloros-muodossa tavallisena JPG- tai TIFF-tiedostona, heijastavuusprosentti lasketaan jakamalla pikseliarvo kuvamuodon bittisyvyydellä. JPG-tiedostojen kohdalla jaetaan siis 255:llä ja TIFF-tiedostojen kohdalla 65 535:llä. Voit myös valita PERCENT-muodon tulostuksen Chloros-muodossa, jolloin jokainen pikseli vaihtelee prosenttiarvosta 0,0 arvoon 1,0 (0 % – 100 % heijastavuus). Muista kuitenkin, että jotkin kuvankäsittelyohjelmat eivät tue prosenttiarvoisia (kelluvia desimaaleja) kuvia, ja ne ovat tallennustilaltaan suuria.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
