---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Tulostuskuvamuodot

Chloros vie käsitellyt tuotteet neljässä tiedostomuodossa. Valitse muoto Projektin asetuksissa (käyttöliittymä), käyttämällä `--format` (CLI) tai `export_format` (SDK). CLI ja SDK hyväksyvät tarkalleen alla olevat merkkijonot.

| Muotoilumerkkijono | Laajennus | Pikselityyppi | Pikselialue | Huomautukset |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(oletus)* | `.tif` | uint16-digitaaliluku | 0 – 65535 | Suositellaan fotogrammetriaan / GIS:ään. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100 % heijastavuus. Jotkin sovellukset eivät pysty lukemaan liukuluvullisia TIFF-tiedostoja; tiedostot ovat suurempia. |
| `PNG (8-bit)` | `.png` | uint8-digitaaliluku | 0 – 255 | Häviötön pakkaus, sopii verkkokatseluun ja visualisointiin. |
| `JPG (8-bit)` | `.jpg` | uint8-numero | 0 – 255 | Häviöllinen pakkaus, pienimmät tiedostot. |

## Tulosteiden sijainti

Tuotteet tallennetaan projektikansioon, ryhmiteltynä kameran ja sen jälkeen tiedostomuodon mukaan:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera (model+lens+filter)
    ├── tiff16/                          # follows --format: tiff16, tiff8, png8, jpg8, or tiff32
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one <INDEX>_Index_Images/ folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Kamerakansio on `LATT-<sensor>-<lens>-F<filter>` LATTICE-kameralle ja `<model>_<filter>` (esim. `Survey3N_RGN`) Survey3-kameralle. **Jokainen viety tuote säilyttää lähdetiedoston nimen — kansio tunnistaa tuotteen, ei tiedostonimen pääte.** Katso täydelliset säännöt kohdasta [Tulostustiedostojen sijainti](reference/cli-reference.md) CLI-viitteessä.

## LATTICE-tuotteet (kaappaus- ja vientitasot)

Yksi LATTICE-raakakehys haarautuu jokaiseksi pyydetyksi tuotteeksi yhdellä kierroksella. Jokaisella tuotetyypillä on oma kytkin (GUI-valintaruudut tai CLI `--debayered` / `--preview` / `--radiance` / `--reflectance`, oletusarvoisesti kaikki PÄÄLLÄ):

| Taso | Sisältö | Tietotyyppi |
| --- | --- | --- |
| `raw` | Suoraan anturista tuleva Bayer-data (mustavalkokamerat: yksi kaista). Käsittely alkaa aina raakadatasta. | Kuten tallennettu |
| `debayered` | Lineaarinen demosaikki — 3-kanavainen M3C:lle, 1-kanavainen harmaasävyinen M3M:lle. | Lineaarinen DN |
| `radiance` | Absoluuttinen spektrinen säteilyvoimakkuus koko radiometrisestä ketjusta, yksikössä **W/m²/sr/nm**. Kirjoitetaan aina 32-bittisenä TIFF (`tiff32/Radiance_Images/`), valitusta vientimuodosta riippumatta. | float32 |
| `reflectance` | Heijastavuus ρ, jossa **DN 32768 = ρ 1,0 (100 %)** ja liikkumavaraa ρ 2,0:aan asti. Pix4D-valmis. | uint16 |
| `preview` | Näyttövalmis renderointi: RGB = valkotasapaino + gamma; monispektrinen = väärävärinen venytys. | 8-bittinen näyttö |

## Heijastavuuden pikseliarvojen lukeminen

Heijastavuus tallennetaan kokonaislukuna, ja **DN-arvo, joka vastaa ρ = 1,0 (100 % heijastavuus), riippuu lähdekamerasta**:

| Lähdekamera | ρ = 1,0 on DN | Miten selvittää |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (varaa ρ 2,0:een asti) | XMP-tunniste `Chloros:PixelScale=32768` on merkitty tiedostoon. |
| Survey3 | `65535` (rajoitettu arvoon ρ 1,0) | Ei `Chloros:*`-XMP-tunnisteita — niiden puuttuminen on merkki. |

**Lue `Chloros:PixelScale` XMP-tunniste ja jaa sillä** sen sijaan, että olettaisit vakion. Tunniste on määritelty uint16-alueella, joten se pysyy samana `32768` myös skaalausmuunnoksissa — normalisoi tallennettu tietotyyppi ensin takaisin uint16:ksi (×257 8-bittisestä, ×65535 float32:sta).

{% hint style="warning" %}
**Yhdessä tapauksessa skaalaa ei ole suunniteltu käytettäväksi.** Kun 8-bittinen lähdekuva (BayerRG8) kirjoitetaan 8-bittisenä TIFF:na, putkisto rajaa arvot välille 0–255 skaalaamisen sijaan, joten tiedostossa ei ole skaalaa — Chloros jättää siinä tarkoituksella `Chloros:PixelScale`:n pois. Jos tunniste puuttuu LATTICE-heijastustiedostosta, älä oleta mittakaavaa; vie tiedosto sen sijaan uudelleen 16-bittisenä tai 32-bittisenä.
{% endhint %}

Katso täydelliset säännöt (mukaan lukien MicaSense-yhteensopivat tunnisteet) kohdasta **”Heijastuspikselien lukeminen”** [CLI-viitteessä](reference/cli-reference.md).
