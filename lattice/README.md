# LATTICE-kamerat

LATTICE on MAPIRin modulaarinen monispektrikamerajärjestelmä maatalous- ja tieteelliseen kuvantamiseen. Jokainen LATTICE-kamera perustuu Sony IMX265 -globaalilukkoanturiin (**3,1 MP, 3,45 µm pikselit**) ja liitetään Ethernet-verkkoon**GigE Vision** -laitteena.

Chloros 1.3.0 ohjaa LATTICE-kameroita reaaliaikaisesti — tunnistus, reaaliaikainen esikatselu, kuvaus ja synkronoidut monikamerajärjestelmät — kolmesta käyttöliittymästä:

| Käyttöliittymä    | Missä                                                          | Alustat                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| Graafinen käyttöliittymä | **Kamerat**-välilehti Chloros-sivupalkissa                         | Windows 10/11 x64                                        |
| CLI        | `chloros-cli lattice`-komentoperhe                           | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |
| Python-SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **Etsitkö laitteistoa?**Kameramoduulit, objektiivit, suodattimet ja kaistat, kehykset ja kiinnikkeet, kaapelit, PoE- ja laukaisukaapelointi on kuvattu [**LATTICE-käyttöoppaassa**](https://mapir.gitbook.io/lattice-camera). Tässä luvussa käsitellään kameroiden ohjaamista Chloros-ohjelmistosta.

LATTICE-tallenteet ovat vakiomuotoisia `.tif`/`.tiff`-tiedostoja, ja Chloros käsittelee ne aina raakatallenteesta lähtien. Katso [CLI-viite](../reference/cli-reference.md) ja [SDK-viite](../reference/sdk-reference.md), joissa on täydellinen komento- ja API-rajapinta.

## Kaksi anturikonfiguraatiota

| Konfiguraatio | Anturi       | Suodatin                                | Mitä yksi kamera tuottaa                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Bayer-väri | kolmikaistainen suodatin                |**Kolme kalibroitua kaistaa yhdestä valotuksesta**                 |
| **M3M**| Mustavalkoinen   | yksi kapeakaistainen häiriösuodatin |**Yksi kalibroitu kaista**; yhdistämällä useita M3M-kameroita saadaan indeksejä |

Koska M3M-kamera on mustavalkoinen ja siinä on yksi suodatin, jokaiselle kaistalle tehdään oma valotus. M3C-kamera kattaa kaikki kolme kaistaansa yhdellä anturin valotuksella.

## Mallimerkkijonot ja nimeäminen

Jokainen kamera tallentaa tunnisteensa GenICam-järjestelmään `DeviceUserID` mallimerkkijonona:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros-ohjelmisto näyttää sen etuliitteellä `LATT-` (esimerkiksi `LATT-M3M-L87-F450`). Sama merkkijono `LATT-…` kirjoitetaan jokaisen viennin EXIF-tunnisteeseen `Model` ja sitä käytetään kameran tulostuskansion nimenä käsitellyissä projekteissa.

| Komponentti | Arvot                                                   | Merkitys                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Kenno    | `M3C` / `M3M`                                            | Bayer-väri / mustavalkoinen                                                                          |
| Objektiivi | `L41` / `L87`                                            | Luku on **vaakasuora kuvakulma asteina**: L41 = kapea (41°), L87 = laaja (87°)    |
| Suodatin    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) tai `F<nm>` (M3M) | Katso [Suodattimet ja spektrikaistat](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

Mallimerkkijono ohjaa kaikkea jatkokäsittelyä: Chloros määrittää anturiprofiilin, kaistajärjestelyn ja tehdaskalibroinnin arvoista `DeviceUserID` + `DeviceSerialNumber`. Kamerakohtaisia asetuksia ei tarvitse määrittää — katso [Kameroiden liittäminen](connecting.md).

## Suodattimet ja kaistat

Kaistakeskukset, FWHM-reunat ja koko 23 SKU:n M3M-tuoteluettelo ovat tuotetietoja, joten ne löytyvät laitteistokäsikirjasta: [**Suodattimet ja spektrikaistat**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Ohjelmistopuolella merkitystä on sillä, että mallimerkkijonossa oleva suodatinkoodi määrittää, mitä tuotteita Chloros voi tuottaa. RGB-suodatinkamerat (`FRGB`) tuottavat ainoastaan debayeroituja ja esikatselutuotteita — kaistakohtainen säteilyvoimakkuus ja heijastavuus eivät ole merkityksellisiä laajakaistaiselle anturille, joten Chloros ohittaa ne ja ilmoittaa asiasta. Kaikki muut suodattimet tuottavat täydellisen säteilyvoimakkuus → heijastavuus → indeksi -ketjun.

## Radiometrinen kalibrointi lyhyesti

Jokainen LATTICE-kamera kalibroidaan tehtaalla yksilöllisesti NIST-jäljitettävää ketjua vasten, ja sen mukana toimitetaan kamerakohtainen sertifikaatti. Mitä kalibrointi kattaa, miten se mitataan ja minkä tarkkuuden voit ilmoittaa, löytyy laitteistokäsikirjasta: [**Tehdasradiometrinen kalibrointi**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

Ohjelmistopuolella on tärkeää, että Chloros määrittää oikean kalibroinnin, kun kamera kytketään, ja lukitsee käytetyt kertoimet jokaiseen vientiin — katso [Kameroiden kytkeminen](connecting.md).

## Tässä luvussa

* [Kameroiden liittäminen](connecting.md) — automaattinen tunnistus, käyttöliittymän liitäntävalintaikkuna, CLI- ja SDK-vastaavuudet sekä se, miten tehdaskalibrointi määritetään (kameran mukana toimitettu paketti vs. pilvipalvelu), kun kamera kytketään.
* [Kameran asetukset](camera-settings.md) — kaikki Kamerat-välilehden hallintalaitteet: kameraluettelo, kamerakohtainen asetuspaneeli ja matriisiasetuspaneeli (mukaan lukien ROI-editori ja kohdistus).
* [Tallennusasetukset ja -tilat](capture.md) — Kaikkien kuvien tallennus, yksittäinen / jatkuva / aikavälin tallennus, vientityypit sekä tallennuskansion rakenne.
* [Monikamerajärjestelmät](arrays.md) — järjestelmän liitäntäohje, synkronointikaapelin koetin, kaistanleveys, PTP ja järjestelmän reaaliaikainen näkymä.
* [Yksikamerat ja kasvillisuusindeksit](mono-indices.md) — M3M-kamerat sekä kohdistus → pinoaminen → indeksointi -työnkulku.

Koko komentojoukko löytyy [CLI-ohjeesta](../reference/cli-reference.md) ja [SDK-ohjeesta](../reference/sdk-reference.md).
