---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Tulostuskuvamuodot

Chloros vie käsitellyt tuotteet neljässä tiedostomuodossa. Valitse muoto Projektin asetuksissa (GUI), komennolla `--format` (CLI) tai komennolla `export_format` (SDK). CLI ja SDK hyväksyvät tarkalleen alla olevat merkkijonot.

| Muotoilumerkkijono | Tiedostotunniste | Pikselityyppi | Pikselialue | Huomautukset |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(oletus)* | `.tif` | uint16-numero | 0 – 65535 | Suositellaan fotogrammetriaan / GIS:ään. |
| `TIFF (8-bit)` | `.tif` | uint8-numero | 0 – 255 | Kaikille 8-bittisille tallenteille, jotka tarjoavat vain 8-bittisiä formaatteja: Chloros ei korota bittisyvyyttä. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100 % heijastavuus. Jotkin sovellukset eivät pysty lukemaan liukuluvullisia TIFF-tiedostoja; tiedostot ovat suurempia. |
| `PNG (8-bit)` | `.png` | uint8-numerot | 0 – 255 | Häviötön pakkaus, sopii verkkokatseluun ja visualisointiin. |
| `JPG (8-bit)` | `.jpg` | uint8-numero | 0 – 255 | Häviöllinen pakkaus, pienimmät tiedostot. |

## Tulosteiden tallennuspaikka

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

Kamerakansio on `LATT-<sensor>-<lens>-F<filter>` LATTICE-kameralle ja `<model>_<filter>` (esim. `Survey3N_RGN`) Survey3-kameralle. **Jokainen viety tuote säilyttää lähdetiedoston nimen — tuotteen tunnistaa kansio, ei tiedostonimen pääte.** Katso täydelliset säännöt kohdasta [Tulostustiedostojen sijainti](reference/cli-reference.md) CLI-viitteessä.

## LATTICE-tuotteet (tallennus- ja vientitasot)

Yksi LATTICE-raakakehys jaetaan jokaiseksi pyydetyksi tuotteeksi yhdellä kerralla. Jokaisella tuotetyypillä on oma kytkin (GUI-valintaruudut tai CLI-komennot `--debayered` / `--preview` / `--radiance` / `--reflectance`, kaikki oletusarvoisesti päällä):

| Taso | Sisältö | Tietotyyppi |
| --- | --- | --- |
| `raw` | Suoraan anturista peräisin olevat Bayer-tiedot (mustavalkokamerat: yksi kaista). Käsittely alkaa aina raakadatasta. | Kuten tallennettu |
| `debayered` | Lineaarinen demosaikki — 3-kanavainen M3C:lle, 1-kanavainen harmaasävyinen M3M:lle: kalibrointipaketin tumma offset on poistettu, minkä jälkeen suoritetaan reunoja huomioiva demosaikki ilman lineaarisuuskorjausta, tasokenttää tai värien erottelua. `process`-vienti tallentaa tällä tasolla saman tiedoston kuin kuvaus tallentaa, kaikissa käsittelytiloissa. | Lineaarinen DN |
| `radiance` | Absoluuttinen spektrinen säteilyvoimakkuus koko radiometrisestä ketjusta, yksikkönä **W/m²/sr/nm**. Tallennetaan aina 32-bittisenä TIFF-tiedostona (`tiff32/Radiance_Images/`) valitusta vientimuodosta riippumatta. | float32 |
| `reflectance` | Heijastavuus ρ, jossa **DN 32768 = ρ 1,0 (100 %)** ja liikkumavaraa ρ 2,0:aan asti. Pix4D-yhteensopiva. | uint16 |
| `preview` | Näyttövalmis renderointi: RGB = valkotasapaino + gamma; monispektrinen = väärävärinen venytys. | 8-bittinen näyttö |

### Suunta

`raw`- ja `debayered`-tiedostot säilyttävät aina anturin **alkuperäisen** suunnan; kameran käännös-/kierron asetus koskee vain muita tuotteita (esikatselu, säteily, heijastavuus, indeksi). Array-kaappauksissa jokaisen jäsenen tiedostot säilyvät alkuperäisessä suunnassaan, ellet pyydä suunnan muuttamista (`--orient` CLI:ssä, `orient=True` SDK:ssä). Jokainen tiedosto tallentaa pikseliinsä kuuluvan suunnan XMP-muodossa — `ChlorosRotation`, `ChlorosFlipHorizontal`, `ChlorosFlipVertical` — ja `ChlorosOrientationApplied` kertoo, sisältävätkö sen omat pikselit tämän tiedon (`false` raakatiedostoissa ja debayeroiduissa tiedostoissa). Chloros-kuvankatseluohjelma muokkaa raaka- ja debayeroituja tiedostoja kyseisen tietueen mukaisesti vain näyttöä varten; itse tiedostoja ei koskaan kirjoiteta uudelleen. Tiedosto, jossa ei ole tietuetta (kirjoitettu ennen versiota 1.3.0 tai toisella työkalulla), näytetään tallennetussa muodossaan.

## Leikkausmaskit

Monispektrisen M3C-kameran säteily- ja heijastavuusarvot käsitellään 3×3-kaistaisella erottelulla. Jos erottelu tuottaa jossakin kaistassa negatiivisen arvon, kyseinen arvo leikataan nollaan (reaaliaikaisissa näkymissä leikkaus tapahtuu täsmälleen samalla tavalla). Uutta versiossa 1.3.0 on, että tuote, jossa vähintään yksi pikseli on rajattu, saa mukanaan **rajoitusmaskin**, joka kertoo tarkalleen, mitkä pikselit ja kaistat ovat kyseessä:

* **Missä:**`clamp_masks/`-kansiossa**tuotekansioiden vieressä**, ei koskaan niiden sisällä: `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif` ja `tiff32/Radiance_Images/x.tif` → `tiff32/clamp_masks/Radiance_Images/x_clampmask.tif`.
* **Mitä:** yksikaistainen `uint8`-TIFF-tiedosto tuotteen omalla pikseliristikolla (suunta, ROI, binning ja matriisin kohdistus mukaan lukien). Bitti 0 (`mask & 1`) asetetaan silloin, kun tuotteen R-kanavan kaista on leikattu, bitti 1 (`& 2`) G-kanavassa, bitti 2 (`& 4`) B-kanavassa. Leikattujen pikselien osuus kaistakohtaisesti merkitään tuotteeseen myös tunnisteilla `Chloros:BandRUnmixClampFraction`, `Chloros:BandGUnmixClampFraction` ja `Chloros:BandBUnmixClampFraction`.
* **Miten tuote ilmaisee tämän:** XMP-tunniste `Chloros:UnmixClampMask` sisältää maskin polun suhteessa tuotteen kansioon (`../clamp_masks/<product folder>/<name>_clampmask.tif`, aina `/`-erotettuna), `none`, kun yhtään pikseliä ei leikattu (maskia ei kirjoitettu), tai `unavailable`, kun maskia ei voitu kirjoittaa. Tuotteella, jolla ei ole tunnistetta, ei ole koskaan suoritettu värien erottelua (RGB- ja monokamerat, raaka- ja debayeroidut tasot).

Koska missään tuotekansiossa ei ole maskia, voit siirtää tuotekansion, kuten `Reflectance_Calibrated_Images/`, Pix4D:lle, Metashapelle tai muulle fotogrammetriaohjelmistolle sellaisenaan, alikansiot mukaan lukien — siirrä tuotekansio, älä sen yläpuolella olevaa formaattikansiota. Chlorosin oma tuontitoiminto ohittaa rajoitusmaskit. CLI-komento laskee ne erikseen: `Image products written: N (plus M clamp masks)`.

## Indeksituotteet

Jokainen indeksi tallennetaan omaan `<INDEX>_Index_Images/`-kansioonsa (esim. `NDVI_Index_Images/`).

* **Integer-formaatit**(`TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`) tallentavat**kuvakohtaisen lineaarisen venytyksen**, eivät −1…1-kuvausta: DN 0 on kyseisen kehyksen pienin indeksiarvo ja täysasteikko sen suurin. Tiedostossa on `Chloros:IndexScaling=linear_minmax`, `Chloros:IndexMin` ja `Chloros:IndexMax`, joten myös `value = IndexMin + DN / full_scale × (IndexMax − IndexMin)`. Koska jokaisella kehyksellä on omat päätepisteensä, vertaa kehyksiä arvoina, ei DN-arvoina.
* **`TIFF (32-bit, Percent)`** tallentaa itse indeksiarvot float32-muodossa ilman uudelleenskaalausta — paras valinta kvantitatiiviseen työhön koko lennon osalta.

Uutta versiossa 1.3.0 on, että indeksituote kuvaa **itseään** sen sijaan, että se toistaisi lähdemateriaalin tunnisteita. Sen GPS- ja kameratunnisteet kopioidaan tuotteesta, josta se on laskettu (heijastavuustuote, jos sellainen on), ja sitten:

| Tunniste | Arvo |
| --- | --- |
| `Chloros:ProcessingLevel` | `index` (joten tiedostoa ei koskaan tuoda uudelleen kuvana) |
| `Chloros:IndexApplied` / `Chloros:IndexExpression` | `True` / käytetty kaava |
| `Chloros:IndexLevel` | `reflectance` tai `radiance` — mistä indeksi laskettiin (puuttuu, jos kumpikaan ei ole) |
| `Chloros:IlluminationInvariant` | `True`, kun indeksi on laskettu heijastavuudesta, `False`, kun se on laskettu säteilyvoimakkuudesta tai anturiarvoista |
| `Chloros:ChlorosPipelineVersion` | indeksin luonut Chloros-versio |

`Chloros:PixelScale` on poistettu indeksituotteista — käytä yllä olevia `IndexMin` / `IndexMax`. Täydellinen lukemisohje löytyy kohdasta ”An integer index raster is a per-image contrast stretch” [CLI-viitteessä](reference/cli-reference.md).

## Heijastavuuspikseliarvojen lukeminen

Heijastavuus tallennetaan kokonaislukuisena digitaalisena lukuna, ja **DN-arvo, joka tarkoittaa ρ = 1,0 (100 %:n heijastavuus), riippuu lähdekamerasta**:

| Lähdekamera | ρ = 1,0 on DN | Miten selvittää |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (varaa ρ 2,0:een asti) | XMP-tunniste `Chloros:PixelScale=32768` on merkitty tiedostoon. |
| Survey3 | `65535` (rajoitettu arvoon ρ 1,0) | Ei `Chloros:PixelScale`-tunnistetta — sen puuttuminen on merkki. (Survey3-tuotteissa on muita `Chloros:*`-käsittelytunnisteita, kuten `Chloros:CalibrationApplied`, kaikissa käsittelytiloissa versiosta 1.3.0 lähtien.) |

**Lue `Chloros:PixelScale` XMP-tunniste ja jaa sillä** sen sijaan, että olettaisit vakioarvon. Tunniste on määritelty uint16-alueella, joten se pysyy arvona `32768` myös skaalausmuutoksia tekevissä tulostusmuodoissa — normalisoi tallennettu tietotyyppi ensin takaisin uint16:ksi (×257 8-bittisestä, ×65535 float32:sta).

{% hint style="warning" %}
**8-bittiset tallenteet.** 8-bittisen tallenteen vientitiedot ovat 8-bittisiä. Sen heijastavuus skaalataan uudelleen ja merkitään tunnisteella `Chloros:PixelScale=32768`, joten mittakaava säilyy; 8-bittiset DN-tuotteet ja ennen versiota 1.3.0 tallennetut heijastustiedostot on rajattu arvoihin 0–255, eikä niissä ole mittakaavatunnistetta. Jos LATTICE-heijastustiedostosta puuttuu tunniste, älä oleta mittakaavaa; vie tiedosto sen sijaan uudelleen 16- tai 32-bittisenä.
{% endhint %}

Katso täydelliset säännöt (mukaan lukien MicaSense-yhteensopivat tunnisteet) kohdasta **”Heijastuspikseleiden lukeminen”** [CLI-ohjeessa](reference/cli-reference.md).
