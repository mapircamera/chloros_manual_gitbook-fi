# Dynaaminen laskentakapasiteetin mukautus

Chloros 1.3.0 hyödyntää laitteiston tunnistusta ja automaattista käsittelystrategian valintaa. Käsittelymoottori mukautuu laitteistoosi — Jetson Orin Nanosta monen GPU:n työasemaan — ilman manuaalista konfigurointia.

***

## Miten se toimii

Kun Chloros käynnistyy, se kartoittaa järjestelmän:

1. **Tunnistaa käyttöjärjestelmän** — Windows tai Linux
2. **Tunnistaa CPU-ytimet ja RAM-muistin kokonaismäärän**

3.**Tunnistaa GPU:n läsnäolon** — NVIDIA CUDA -yhteensopivuus, VRAM, malli
4. **Tunnistaa Jetson-mallin** (jos sovellettavissa) — tunnuksen `/proc/device-tree/model` avulla
5. **Tarkistaa lämpötila-anturit** (Jetson) — lämpötilaa huomioivaa käsittelyä varten
6. **Valitsee laskentastrategian** — kaikkien tunnistettujen laitteistokomponenttien perusteella
7. **Määrittää työntekijöiden lukumäärän, putkityypin ja muistin varauksen** automaattisesti

Tunnistettu profiili tallennetaan välimuistiin istunnon ajaksi sekä muistiin että levylle, jolloin myöhemmät suoritukset käynnistyvät nopeammin:

| Alusta | Välimuistissa oleva profiili |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (huomioi `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Poista kyseinen tiedosto, jotta tunnistus suoritetaan uudelleen — tämä on hyödyllistä esimerkiksi GPU:n tai lisää RAM-muistia lisättyäsi. Chloros suorittaa tunnistuksen automaattisesti uudelleen myös silloin, kun välimuisti on tallennettu yhteensopimattomalla vanhemmalla versiolla.

***

## Laskentastrategiat

Chloros valitsee laitteistosi perusteella yhden kolmesta laskentastrategiasta:

| Strategia | Valitaan, kun | Työntekijät | Suorittaja | Putki |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| CUDA-grafiikkakortti, joka ilmoittaa**12 GB tai enemmän VRAM-muistia**(Jetsonin yhtenäisessä muistissa, vaatii myös vähintään 12 GB jaettua RAM-muistia) | `min(4, ⌊(VRAM − 0.5GB) × 0.9 ÷ 3.5GB⌋)`, vähintään 2 —**rajoitettu 2:een Jetsonissa** | Työntekijäprosessit (erilliset CUDA-kontekstit) | `fused_gpu` |
| **`GPU_SINGLE`**| CUDA-GPU, jossa on**2–12 GB VRAM**| 3 (I/O-päällekkäisyys; GPU-käyttö sarjoitettu semaforilla).**1 (peräkkäin) Jetson-laitteissa, joissa on alle 12 GB RAM-muistia** | Työprosessit (erilliset CUDA-kontekstit); peräkkäin samassa prosessissa vähämuistisissa Jetson-laitteissa | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Ei CUDA-GPU:ta tai alle 2 GB VRAM-muistia | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Esimerkkejä `GPU_PARALLEL`-työntekijäkaavasta: 12 Gt VRAM → 2 työntekijää, 16 Gt → 3 työntekijää, 20 Gt+ → 4 työntekijää. Jokaiselle työntekijälle varataan 3,5 GB, mikä on Texture Aware -tehtävän mitattu tarve, 90 %:n sisällä 0,5 GB:n varauksen jälkeen jäljellä olevasta VRAM-muistista.

{% hint style="info" %}
**GPU-strategiat edellyttävät CUDA:ta tukevaa PyTorchia.** Laite, jonka PyTorchissa ei ole käytettävissä olevaa CUDA-laitetta, saa luokituksen `CPU_PARALLEL` riippumatta sen GPU:sta. Jetson-paketti 1.3.0 käyttää JetPack 6:n CUDA-rakennetta PyTorchista, joten Jetson täyttää vaatimukset, kunhan JetPackin CUDA- ja cuDNN-kirjastot on asennettu. Jetson, jonka RAM-muisti on alle 12 GB, käsittelee aina yhden kuvan kerrallaan prosessin aikana (kirjataan virhelokiksi `GPU_SINGLE`).
{% endhint %}

GPU-strategiat käyttävät työprosessien joukkoa, joista kukin on erillinen käyttöjärjestelmäprosessi omalla CUDA-kontekstillaan. Chloros käynnistää ne käynnistämällä uudelleen oman asennetun taustaohjelmansa (ei Python in `multiprocessing`:n kautta, jonka työprosessien käynnistys ei toimi pakatun sovelluksen sisällä). CPU-strategia käyttää säiepoolia (Python in `ThreadPoolExecutor`). Chloros ei käytä mitään kolmannen osapuolen hajautettua kehystä (kuten Rayta).

### Putkityypit

* **`fused_gpu`** — Täysi GPU-käsittelypolku. Debayer-, korjaus- ja indeksointitoiminnot suoritetaan GPU:lla yhdellä yhdistetyllä kierroksella. Suurin läpimenokapasiteetti, vaatii eniten VRAM-muistia.
* **`tiled_gpu`** — Muistitehokas GPU-polku. Käsittelee kuvia ruuduittain, jotta ne mahtuvat rajoitettuun GPU-muistiin. Alhaisempi läpimenokapasiteetti, mutta toimii laitteissa, joissa muistia on vähän.
* **`cpu_fallback`** — Pelkästään CPU:lla tapahtuva käsittely, jossa hyödynnetään monisäikeistä rinnakkaisuutta. Käytetään, kun NVIDIA-GPU:ta ei ole käytettävissä, sekä viimeisenä keinona, jos molemmat GPU-käsittelypolut epäonnistuvat.

Suoritusaikainen varajärjestys on aina `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Strategian manuaalinen ohitus

Aseta `CHLOROS_STRATEGY`-ympäristömuuttuja pakottaaksesi tietyn strategian — tämä on asiantuntijoiden pakotie tilanteisiin, joissa automaattinen tunnistus valitsee tilanteeseesi sopimaton vaihtoehdon (esimerkiksi GPU:n pitäminen vapaana muuhun työhön):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

Muuttujan tunnistuksessa ei oteta huomioon kirjainkokoa; kaikki muut kuin nämä kolme nimeä ohitetaan ja automaattinen tunnistus etenee normaalisti. Ohituksen ollessa käytössä Chloros valitsee edelleen työntekijämäärän puolestasi:

| Ohitus | Käytetty työntekijämäärä |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

`GPU_SINGLE`- tai `GPU_PARALLEL`-ohitus koneella, jolla ei ole käytettävissä olevaa CUDA-laitetta, hylätään, ja lokiin kirjataan rivi `[STRATEGY]`, ja suoritus käyttää `CPU_PARALLEL`-asetusta.

On suositeltavaa asettaa tämä komento-kohtaisesti eikä pysyvästi, jotta normaalit suoritukset sopeutuvat edelleen automaattisesti.

***

## Alustakohtainen käyttäytyminen

| Alusta | Strategia | Työntekijät | Putki | Huomautukset |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (peräkkäinen) | Muistitehokas tila, yksi kuva kerrallaan |
| **Jetson Orin NX 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (peräkkäinen) | Alle 12 GB:n jaettu RAM pakottaa peräkkäiseen käsittelyyn |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (samanaikainen) | Suositeltu reunalaite — Jetson-rajoitus 2 työntekijää |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (samanaikainen) | Suurin reuna-laitteen suorituskyky (myös Jetson-rajoitus: enintään 2 työntekijää) |
| **Pöytätietokone, jossa 8 GB:n GPU** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 työntekijää suorittavat I/O-operaatioita päällekkäin, kun semafori sarjoittaa GPU:n käytön |
| **Pöytätietokone, jossa on vähintään 12 Gt:n GPU** | `GPU_PARALLEL` | 2–4 | `fused_gpu` (samanaikaisesti) | Optimaalinen työpöytätietokoneen suorituskyky: 12 GB → 2 työntekijää, 16 GB → 3, 20 GB+ → 4 |
| **Pelkkä CPU-järjestelmä** | `CPU_PARALLEL` | fyysiset ytimet − 1 (vähintään 2) | `cpu_fallback` | GPU:ta ei tarvita, käyttää säikeistöpoolia |

{% hint style="info" %}
**Jetsonin yhtenäinen muisti**: Jetson-laitteet jakavat GPU:n ja CPU:n muistin. Jetson Orin NX 16 GB ilmoittaa noin 15,3 GB VRAM-muistia, mutta se on samaa fyysistä RAM-muistia, jota käyttöjärjestelmä ja CPU-prosessit käyttävät. Siksi 16 GB:n tai suuremmat Jetson-laitteet täyttävät `GPU_PARALLEL`-vaatimukset kuten 12 GB:n tai suuremmat työpöytä-GPU:t, mutta niissä on kuitenkin 2 työntekijän rajoitus — GPU, työntekijäprosessit ja niiden työntekijäkohtaiset CUDA-kontekstit käyttävät kaikki samaa jaettua muistipoolia.
{% endhint %}

### GPU-budjetti VRAM:n mukaan (erilliset GPU:t)

x86_64-isäntäkoneissa, joissa on erillinen NVIDIA-GPU, havaittu VRAM määrittää myös, kuinka suuren osan kortin prosessointikapasiteetista voidaan varata ja kuinka suuriksi erät voivat kasvaa:

| Tunnistettu VRAM | GPU-budjetin yläraja | Erän koon kerroin |
| --- | --- | --- |
| **8 GB+** | 90 % | ×2,0 |
| **6–8 GB** | 85 % | ×1,75 |
| **3,5–6 Gt** | 80 % | ×1,5 |
| **2–3,5 Gt** | 75 % | ×1,25 |
| **Alle 2 Gt** | 70 % | ×1,0 |

Erilliset GPU:t varaavat järjestelmälle vain 0,5 GB, koska ne eivät jaa järjestelmän RAM-muistia. Jetson-profiilit varaavat huomattavasti enemmän ja asettavat alarajan matalammaksi — katso [NVIDIA Jetson -opas](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Dynaaminen GPU-muistin allokointi

Chloros käyttää [4-säikeistä käsittelyputkea](processing-pipeline.md):

* **Säie 1** (Tunnistus) — Kuvan lataaminen, EXIF-tietojen jäsentäminen, kohteen tunnistus
* **Säie 2** (Kalibrointi) — Heijastavuuskalibroinnin laskenta
* **Säie 3** (Käsittely) — GPU-debayer, vinjetoinnin korjaus, indeksin laskenta
* **Säie 4** (Vienti) — Tiedostojen kirjoittaminen, metatietojen upottaminen

Säikeet 1, 2 ja 4 kuluttavat vain vähän GPU-resursseja; Säie 3 on resurssien suurkuluttaja. Kun aikaisemmat prosessiketjun säikeet päättyvät, niiden GPU-resurssit **jaetaan uudelleen jäljellä oleville aktiivisille säikeille**, joten säie 3 saa asteittain enemmän muistia prosessin edetessä.

### Allokointivaiheet

| Vaihe | Aktiiviset säikeet | GPU-muistin jakautuminen |
| --- | --- | --- |
| **Alkuvaihe** | 1, 2, 3, 4 | Jaettu kaikkien säikeiden kesken, suurin osa säikeelle 3 |
| **Alku-keskivaihe** | 2, 3, 4 | Säikeen 1 osuus jaetaan uudelleen |
| **Myöhäinen vaihe** | 3, 4 | Säikeiden 1 ja 2 osuudet siirretään säikeille 3 ja 4 |
| **Loppuvaihe** | 3 tai 4 | Viimeinen aktiivinen säie saa suurimman mahdollisen muistimäärän |

Lukuja säätelevät kaksi sääntöä:

* **Ainoa** aktiivinen säie saa profiilinsa enimmäismääräisen allokoinnin.
* Kun useampi kuin yksi *raskas* GPU-tehtävä on aktiivinen, kunkin raskaan tehtävän perusallokointi jaetaan niiden kesken (ei koskaan alle määritetyn vähimmäismäärän).

Suoritusaikana tosiasiallisesti käytetty arvo on alusta-profiilin allokoinnin ja GPU-muistimonitorin reaaliaikaisen suosituksen **pienempi**, joten kuormitettu kortti voittaa aina optimistisen profiilin.***

## Tekstuuritietoinen käsittely

Texture Aware -debayer (**vain Chloros+** — `--debayer texture-aware`) käyttää AI/ML-kohinanpoistomallia, joka tarvitsee kopioittain noin 1,75 GB VRAM-muistia FP16-muodossa, joten se käyttää huomattavasti enemmän GPU-muistia kuin Standard-menetelmä:

* Järjestelmät, joissa on **alle 7 Gt VRAM-muistia**, käsittelevät tekstuuritietoista käsittelyä**synkronisessa silmukassa, yksi kuva kerrallaan** — useita mallikopioita ei mahdu kerralla, ja työryhmä vain lisäisi kilpailua
* Järjestelmät, joissa on **7 GB tai enemmän VRAM-muistia**, voivat käsitellä Texture Aware -toimintoa samanaikaisesti, vaikkakin Standard-menetelmään verrattuna pienemmällä työntekijämäärällä
* **Jetson**-laitteilla Texture Aware on aina kiinnitetty yhteen työntekijään
* Jos **käytettävissä ei ole CUDA-GPU:ta**, suoritus käyttää Standard-debayeria, ja taustapalvelu kirjaa lokiin yhden `[DEBAYER] … needs a CUDA GPU …`-rivin, jossa selitetään syy
* Vähävirtaisissa Jetson-laitteissa (Nano, Orin Nano) CLI asettaa GPU:n taajuudelle ylärajan, kun pyydät Texture Aware -toimintoa — katso [NVIDIA Jetson -opas](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)

***

## Lämmönhallinta (Jetson)

Jetson-laitteilla on lämpörajoituksia, erityisesti suljetuissa tai lentokoneissa käytettävissä asennuksissa. Chloros valvoo Jetsonin sisäisiä lämpötila-antureita ja skaalaa eräkokoja automaattisesti:

| Lämpötila | Reaktio |
| --- | --- |
| **< 70 °C** | Normaali toiminta — täysi nopeus |
| **70 °C** (Varoitus) | Erän koko pienenee asteittain (100 % → 50 % välillä 70 °C – 80 °C) |
| **80 °C** (Kriittinen) | Voimakas tehonrajoitus (50 % → 0 % välillä 80 °C – 90 °C) |
| **90 °C** (Sammutus) | GPU-käsittely lopetetaan kokonaan |

Riittävän jäähdytyksen omaavissa pöytätietokoneissa lämpösäätö laukeaa harvoin.

***

## Muistin kuormituksen hallinta

Chloros valvoo GPU:n muistia jatkuvasti käsittelyn aikana ja reagoi kolmella tasolla.

**Erän koon määrittäminen.** Erä alkaa 8 kuvasta kerrottuna yllä olevien taulukoiden mukaisella alustakertoimella. Chloros tarkistaa sitten vapaan VRAM-muistin, varaa siitä 20 % PyTorchin omalle hallinnalle ja olettaa, että 12 MP:n kuva vie noin 100 Mt GPU-muistia — erän koko on pienempi näistä kahdesta: muistista johdettu raja tai alustan perusarvo. Se ei koskaan laske alle 1:n.

**Ennakoiva pienentäminen.**Kun**VRAM-käyttöaste ylittää 85 %**, eräkokoja pienennetään ennen kuin mitään menee vikaan.**Säikeittäinen allokaation leikkaaminen.** Kun reaaliaikainen käyttöaste nousee, kunkin säikeen GPU-budjettia supistetaan: ×0,75 yli 80 %:n käyttöasteella, ×0,5 yli 90 %:n käyttöasteella. Seurannan raja-arvot ovat 70 % (varovainen), 85 % (normaali toimintaraja) ja 95 % (OOM-riski).**OOM-peruutus ja palautuminen.** Jos muistin loppumistilanne kuitenkin tapahtuu:

* erän koko **puolitetaan**, ja se puolitetaan uudelleen jokaisen peräkkäisen OOM-tapahtuman yhteydessä — jokainen seuraava onnistunut erä siirtää tätä rangaistusta yhden askeleen taaksepäin
* aktiivisten säikeiden allokaatiot leikataan 70 %:iin niiden nykyisestä arvosta ja allokoija siirtyy konservatiiviseen strategiaan, jota lievennetään jälleen onnistuneiden allokaatioiden sarjan jälkeen
* vakavan kuormituksen alla putki siirtyy tilasta `fused_gpu` tilaan `tiled_gpu` ja viimeisenä keinona tilaan `cpu_fallback`

**Isäntäkoneen RAM-muisti (Jetson).** Ennen käsittelyä CLI arvioi isäntäkoneen huippumuistin kuvamäärän ja debayer-tilan perusteella ja varoittaa, jos RAM-muisti ja tiedostopohjainen swap-tila eivät todennäköisesti riitä, tulostaen tarkat komennot swap-tilan lisäämiseksi — katso [NVIDIA Jetson -opas](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).

***

## Laskentakapasiteetin mukautumisen seuranta

### Järjestelmän diagnostiikka

`chloros-cli selftest` on nopein tapa tarkistaa, mitä laskentakerros havaitsee:

```bash
chloros-cli selftest
```

Sen 7 tarkistusta kattavat version, porttien saatavuuden, taustapalvelun käynnistymisen, `/api/test`, järjestelmätiedot, kohinanpoistomallin olemassaolon sekä CUDA:n ja kohinanpoistomallin käyttövalmiuden. Tarkistus 5 tulostaa laitteistorivin suoraan:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

Tarkistus 7 tulostaa `CUDA: <bool>, Denoiser: <bool>` — molempien on oltava totta, jotta Texture Awarea voidaan ylipäätään käyttää.

### Taustaprosessin lokit

Strategia ja työntekijöiden lukumäärä valitaan taustaprosessissa kunkin ajon alussa — niitä ei ilmoiteta CLI-bannerissa. Kun jokin toimii odottamattomasti (GPU-polun varasuunnitelma, OOM-virhe, denoiserin latausongelma), se näkyy kyseisen istunnon taustaprosessin lokissa:

| Alusta | Lokin sijainti |
| --- | --- |
| **Linux / Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (yksi tiedosto kutakin käynnistystä kohti) |
| **Linux, CLI:llä käynnistetty taustaprosessi** | myös `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Reaaliaikainen edistymisnäyttö

Suorituksen aikana CLI näyttää reaaliaikaisen säikeittäisen edistymisen (tunnistus, analysointi, käsittely, vienti), joka välitetään Server-Sent Events -protokollan kautta — tämä on käytännöllinen tapa selvittää, onko säie 3 pullonkaula. Katso [Käsittelyputki](processing-pipeline.md).

***

## Seuraavat vaiheet

* [Käsittelyputki](processing-pipeline.md) — 4-säikeisen putkiarkkitehtuurin ymmärtäminen
* [NVIDIA Jetson -opas](../linux/nvidia-jetson-guide.md) — Jetson-laitteille ominaiset käyttöönotto- ja optimointiohjeet
* [CLI: komentorivi](../CLI.md) — CLI-opas
* [CLI-viite](../reference/cli-reference.md) — Kattava komentojen luettelo versioon 1.3.0
