# LATTICE-kamerat

LATTICE on MAPIR:n modulaarinen monispektrikamerajärjestelmä maatalous- ja tieteelliseen kuvantamiseen. Jokainen LATTICE-kamera perustuu Sony IMX265 -globaalilukitusanturiin (**3,1 MP, 3,45 µm pikselit**) ja kytkeytyy Ethernet-verkkoon**GigE Vision** -laitteena.

Chloros 1.2.0 ohjaa LATTICE-kameroita reaaliaikaisesti – tunnistus, reaaliaikainen esikatselu, kuvaus ja synkronoidut monikamerajärjestelmät – kolmella käyttöliittymällä:

| Käyttöliittymä    | Missä                                                          | Alustat                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| Käyttöliittymä        | **Kamerat**-välilehti Chloros-sivupalkissa                         | Windows 10/11 x64                                        |
| CLI        | `chloros-cli lattice`-komentoperhe                           | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **Etsitkö laitteistoa?**Kameramoduulit, objektiivit, suodattimet ja kaistat, kehykset ja kiinnikkeet, kaapelit, PoE- ja laukaisukaapelointi on kuvattu [**LATTICE-käyttöoppaassa**](https://mapir.gitbook.io/lattice-camera). Tässä luvussa käsitellään kameroiden ohjausta Chloros:n avulla.

LATTICE-tallenteet ovat tavallisia `.tif`/`.tiff`-tiedostoja, ja Chloros käsittelee ne aina raakatallenteesta lähtien. Katso [CLI-viite](../reference/cli-reference.md) ja [SDK-viite](../reference/sdk-reference.md) sisältävät täydelliset komennot ja API-pinnan.

## Kaksi anturikokoonpanoa

| Kokoonpano | Anturi       | Suodatin                                | Yhden kameran tuottama kuva                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Bayer-väri | kolmikaistainen suodatin                |**Kolme kalibroitua kaistaa yhdellä valotuksella**                 |
| **M3M**| Mustavalkoinen   | yksi kapeakaistainen häiriösuodatin |**Yksi kalibroitu kaista**; yhdistämällä useita M3M-kameroita saadaan indeksejä |

Koska M3M-kamera on yksisuodattimisen monokromaattinen, jokaiselle kaistalle tehdään oma valotus. M3C-kamera kattaa kaikki kolme kaistaansa yhdellä anturivalotuksella.

## Mallimerkkijonot ja nimeäminen

Jokainen kamera tallentaa tunnisteensa GenICam-järjestelmään `DeviceUserID` mallimerkkijonona:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros näyttää sen etuliitteellä `LATT-` (esimerkiksi `LATT-M3M-L87-F450`). Sama `LATT-…`-merkkijono kirjoitetaan jokaisen viennin EXIF-tagiin `Model` ja sitä käytetään kameran tulostuskansion nimenä käsitellyissä projekteissa.

| Komponentti | Arvot                                                   | Merkitys                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Kenno    | `M3C` / `M3M`                                            | Bayer-väri / mustavalkoinen                                                                          |
| Objektiivi      | `L41` / `L87`                                            | Luku on **vaakasuora kuvakulma asteina**: L41 = kapea (41°), L87 = laaja (87°)    |
| Suodatin    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) tai `F<nm>` (M3M) | Katso [Suodattimet ja spektrikaistat](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

Mallimerkkijono ohjaa kaikkea jälkipäässä: Chloros määrittää anturiprofiilin, kaistajärjestyksen ja tehdaskalibroinnin arvoista `DeviceUserID` + `DeviceSerialNumber`. Kamerakohtaisia asetuksia ei tarvitse määrittää – katso [Kamerien liittäminen](connecting.md).

## Suodattimet ja kaistat

Kaistakeskukset, FWHM-reunat ja koko 23 SKU:n M3M-tuoteluettelo ovat tuotetietoja, joten ne löytyvät laitteistokäsikirjasta: [**Suodattimet ja spektrikaistat**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Ohjelmistopuolella merkityksellistä on se, että mallimerkkijonossa oleva suodatinkoodi määrittää, mitä tuotteita Chloros voi tuottaa. RGB-suodattimella varustetut kamerat (`FRGB`) tuottavat ainoastaan debayeroituja ja esikatselutuotteita — kaistakohtainen säteilyvoimakkuus ja heijastavuus eivät ole merkityksellisiä laajakaistaiselle anturille, joten Chloros ohittaa ne ja ilmoittaa asiasta. Kaikki muut suodattimet tuottavat täydellisen säteilyvoimakkuus → heijastavuus → indeksi-ketjun.

## Radiometrinen kalibrointi lyhyesti

Jokainen LATTICE-kamera kalibroidaan tehtaalla yksilöllisesti NIST-jäljitettävää ketjua vasten, ja sen mukana toimitetaan kamerakohtainen sertifikaatti. Mitä kalibrointi kattaa, miten se mitataan ja minkä tarkkuuden voit ilmoittaa, löytyy laitteiston käyttöoppaasta: [**Tehtaan radiometrinen kalibrointi**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

Ohjelmistopuolella on tärkeää, että Chloros määrittää oikean kalibroinnin, kun kamera kytketään, ja lukitsee sovelletut kertoimet jokaiseen vientiin — katso [Kameroiden kytkeminen](connecting.md).

## Tässä luvussa

* [Kameroiden liittäminen](connecting.md) — automaattinen tunnistus, käyttöliittymän liitäntävalintaikkuna, CLI/SDK-vastaavat sekä se, miten tehdaskalibrointi määritetään (kameran sisäinen paketti vs. pilvi) kameran kytkeytyessä.

Muut LATTICE-aiheet — kameran asetukset ja reaaliaikainen ohjaus, tallennustilat, monikamerajärjestelmät sekä mono (M3M)-käsittely ja indeksit — käsitellään tämän käyttöoppaan omissa osioissaan, ja täydellinen komentojoukko löytyy [CLI-viitteestä](../reference/cli-reference.md) ja [SDK-viite](../reference/sdk-reference.md).
