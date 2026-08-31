# Dynaaminen laskentakapasiteetin mukautus

Chloros 1.2.0 hyödyntää laitteiston tunnistusta ja automaattista käsittelystrategian valintaa. Käsittelymoottori mukautuu laitteistoosi — olipa kyseessä sitten Jetson Orin Nano tai usean GPU:n työasema — ilman manuaalista konfigurointia.

***

## Miten se toimii

Kun Chloros käynnistyy, se luo profiilin järjestelmästäsi:

1. **Tunnistaa käyttöjärjestelmän** — Windows tai Linux
2. **Tunnistaa CPU-ytimet ja RAM-muistin kokonaismäärän**

3.**Tunnistaa GPU:n olemassaolon** — NVIDIA CUDA -yhteensopivuus, VRAM, malli
4. **Tunnistaa Jetson-mallin** (jos sovellettavissa) — `/proc/device-tree/model`:n kautta
5. **Tarkistaa lämpötila-anturit** (Jetson) — lämpötilaa huomioivaa käsittelyä varten
6. **Valitsee laskentastrategian** — kaikkien havaittujen laitteistojen perusteella
7. **Määrittää työntekijöiden lukumäärän, putkityypin ja muistin allokoinnin** automaattisesti

Havaittu profiili tallennetaan välimuistiin istunnon ajaksi sekä muistiin että levylle, jolloin myöhemmät suoritukset käynnistyvät nopeammin:

| Alusta | Välimuistissa oleva profiili |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (kunnioittaa `XDG_CONFIG_HOME`:ää) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Poista kyseinen tiedosto, jotta järjestelmä tunnistaa laitteiston uudelleen — tämä on hyödyllistä esimerkiksi GPU:n tai lisää RAM-muistia lisättyäsi. Chloros tunnistaa laitteiston myös automaattisesti uudelleen, jos välimuisti on kirjoitettu yhteensopimattomalla vanhemmalla versiolla.

***

## Laskentastrategiat

Chloros valitsee yhden kolmesta laskentastrategiasta laitteistosi perusteella:

| Strategia | Valitaan, kun | Työntekijät | Suorittaja | Putki |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| CUDA-GPU, joka ilmoittaa**12 GB+ VRAM**(Jetsonin yhtenäisessä muistissa, vaatii myös vähintään 12 GB jaettua RAM-muistia) | `min(4, VRAM ÷ 4GB)`, vähintään 2 —**rajoitettu arvoon 2 Jetsonissa** | `ProcessPoolExecutor` (spawn) | `fused_gpu` |
| **`GPU_SINGLE`**| CUDA-GPU, jossa on**2–12 GB VRAM**| 3 (I/O-päällekkäisyys; GPU-käyttö sarjoitettu semaforilla).**1 (peräkkäin) Jetson-laitteissa, joissa on alle 12 GB RAM-muistia** | `ProcessPoolExecutor` (spawn); peräkkäin prosessin sisällä vähäisen RAM-muistin Jetson-laitteissa | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Ei CUDA-GPU:ta tai alle 2 Gt VRAM-muistia | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Esimerkkejä `GPU_PARALLEL`-työntekijäkaavan toiminnasta: 12 Gt VRAM → 3 työntekijää, 16 Gt tai enemmän → 4 työntekijää, mikä tahansa Jetson → 2 työntekijää.

Rinnakkaisuus toteutetaan Python:n vakiomuotoisella `concurrent.futures`:llä: GPU-strategiat käyttävät `ProcessPoolExecutor`:ää, jossa on **spawn** -käynnistysmenetelmää (jokainen työntekijä on erillinen prosessi, jolla on oma CUDA-kontekstinsa — `fork` kopioisi jo alustetun CUDA-tilan ja vahingoittaisi aliprosesseja), ja CPU-strategia käyttää `ThreadPoolExecutor`:ää. Chloros ei käytä mitään kolmannen osapuolen hajautettua kehystä (kuten Ray).

### Putkityypit

* **`fused_gpu`** — Täydellinen GPU-käsittelypolku. Debayer-, korjaus- ja indeksointitoiminnot suoritetaan GPU:lla yhdellä yhdistetyllä läpikäynnillä. Suurin läpimenokapasiteetti, vaatii eniten VRAM-muistia.
* **`tiled_gpu`** — Muistitehokas GPU-polku. Käsittelee kuvia ruuduittain, jotta ne mahtuvat rajoitettuun GPU-muistiin. Alhaisempi läpimenokapasiteetti, mutta toimii laitteissa, joissa muistia on rajoitetusti.
* **`cpu_fallback`** — Pelkästään CPU:lla tapahtuva käsittely monisäikeistä rinnakkaisuutta hyödyntäen. Käytetään, kun NVIDIA-GPU:ta ei ole käytettävissä, ja viimeisenä keinona, kun molemmat GPU-käsittelypolut epäonnistuvat.

Suoritusaikainen varajärjestys on aina `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Strategian manuaalinen ohitus

Aseta `CHLOROS_STRATEGY`-ympäristömuuttuja pakottaaksesi tietyn strategian — tämä on asiantuntijoiden varasuunnitelma tilanteisiin, joissa automaattinen tunnistus valitsee tilanteeseesi sopimatonta vaihtoehtoa (esimerkiksi GPU:n pitäminen vapaana muuhun työhön):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

Muuttujan tunnistuksessa ei erotella isoja ja pieniä kirjaimia; kaikki muut kuin nämä kolme nimeä ohitetaan, ja automaattinen tunnistus etenee normaalisti. Ohituksen ollessa käytössä Chloros valitsee edelleen työntekijämäärän puolestasi:

| Ohitus | Käytetty työntekijämäärä |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

On suositeltavaa määrittää asetus komentoittain eikä pysyvästi, jotta normaalit suoritukset sopeutuvat automaattisesti.

***

## Alustakohtainen käyttäytyminen

| Alusta | Strategia | Työntekijät | Putki | Huomautukset |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (peräkkäinen) | Muistitehokas tila, yksi kuva kerrallaan |
| **Jetson Orin NX 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (peräkkäin) | Alle 12 GB:n jaettu RAM pakottaa peräkkäiseen käsittelyyn |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (rinnakkainen) | Suositeltu reunalaite — Jetson rajoitettu 2 työntekijään |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (samanaikainen) | Suurin reuna-laitteen suorituskyky (myös Jetson-rajoitus: enintään 2 työntekijää) |
| **Pöytätietokone, jossa 8 GB:n GPU** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 työprosessia käyttävät I/O-liitäntöjä samanaikaisesti, kun semafori sarjoittaa GPU:n käytön |
| **Työpöytätietokone, jossa on 12 Gt:n tai suurempi GPU** | `GPU_PARALLEL` | 3–4 | `fused_gpu` (samanaikaisesti) | Optimaalinen työpöytätietokoneen suorituskyky: 12 GB → 3 työntekijää, 16 GB+ → 4 |
| **Pelkkä CPU-järjestelmä** | `CPU_PARALLEL` | fyysiset ytimet − 1 (vähintään 2) | `cpu_fallback` | GPU:ta ei tarvita, käyttää säikeistöpoolia |

{% hint style="info" %}
**Jetsonin yhtenäinen muisti**: Jetson-laitteet jakavat GPU:n ja CPU:n muistin. Jetson Orin NX 16 Gt ilmoittaa ~15,3 Gt:n VRAM-muistia, mutta se on samaa fyysistä RAM-muistia, jota käyttöjärjestelmä ja CPU-prosessit käyttävät. Siksi 16 GB:n tai suuremmat Jetson-laitteet täyttävät `GPU_PARALLEL`-vaatimukset kuten 12 GB:n tai suuremmat pöytätietokoneiden GPU:t, mutta niissä on kuitenkin kahden työprosessin rajoitus — GPU, työprosessit ja niiden prosessikohtaiset CUDA-kontekstit käyttävät kaikki samaa jaettua muistipoolia.
{% endhint %}

### GPU-budjetti VRAM:n mukaan (erilliset GPU:t)

x86_64-isäntäkoneissa, joissa on erillinen NVIDIA-GPU, tunnistettu VRAM määrittää myös, kuinka suuren osan kortin prosessointikapasiteetista voidaan varata ja kuinka suuriksi erät voivat kasvaa:

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
* **Säie 2** (Kalibrointi) — Heijastavuuskalibroinnin laskeminen
* **Säie 3** (Käsittely) — GPU-debayer, vinjetoinnin korjaus, indeksin laskeminen
* **Säie 4** (vienti) — tiedostojen tallennus, metatietojen upottaminen

Säikeet 1, 2 ja 4 kuluttavat vain vähän GPU-tehoa; Säie 3 on resurssien suurin kuluttaja. Kun aikaisemmat prosessiketjun säikeet päättyvät, niiden GPU-resurssit **jaetaan uudelleen jäljellä oleville aktiivisille säikeille**, joten säie 3 saa asteittain enemmän muistia prosessin edetessä.

### Allokointivaiheet

| Vaihe | Aktiiviset säikeet | GPU-muistin jakautuminen |
| --- | --- | --- |
| **Alkuvaihe** | 1, 2, 3, 4 | Jaettu kaikkien säikeiden kesken, suurin osa säikeelle 3 |
| **Alku-keskivaihe** | 2, 3, 4 | Säikeen 1 osuus jaetaan uudelleen |
| **Väli-loppuvaihe** | 3, 4 | Säikeiden 1 ja 2 osuudet siirtyvät säikeille 3 ja 4 |
| **Loppuvaihe** | 3 tai 4 | Viimeinen aktiivinen säie saa maksimimääräisen muistivarauksensa |

Lukuja säätelevät kaksi sääntöä:

* **Ainoa** aktiivinen säie saa profiilinsa enimmäismääräisen allokoinnin.
* Kun useampi kuin yksi *raskas* GPU-tehtävä on aktiivinen, kunkin raskaan tehtävän perusallokaatio jaetaan niiden kesken (ei koskaan alle määritetyn vähimmäismäärän).

Suoritusaikana tosiasiallisesti käytetty arvo on alusta-profiilin allokaation ja GPU-muistimonitorin reaaliaikaisen suosituksen **pienempi**, joten kuormitettu kortti voittaa aina optimistisen profiilin.***

## Tekstuuritietoinen käsittely

Tekstuuritietoinen debayer (**Chloros+ vain** — `--debayer texture-aware`) käyttää AI/ML-kohinanpoistomallia, joka tarvitsee noin 1,75 GB VRAM-muistia FP16-muodossa kopiota kohti, joten se käyttää huomattavasti enemmän GPU-muistia kuin Standard-menetelmä:

* Järjestelmät, joissa on **alle 7 GB VRAM-muistia**, käsittelevät tekstuuritietoista käsittelyä**synkronisessa silmukassa, yksi kuva kerrallaan** — useita mallikopioita ei mahdu, ja työryhmä vain lisäisi kilpailua
* Järjestelmissä, joissa on **7 Gt tai enemmän VRAM-muistia**, Texture Aware -toiminto voidaan käsitellä samanaikaisesti, vaikkakin Standard-menetelmään verrattuna pienemmällä työntekijämäärällä
* **Jetson**-laitteilla Texture Aware on aina kiinnitetty yhteen työprosessiin, ja vähävirtaisissa malleissa (Nano, Orin Nano) se asettaa myös automaattisesti GPU:n taajuusrajoituksen — katso [NVIDIA Jetson -opas](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)***

## Lämmönhallinta (Jetson)

Jetson-laitteilla on lämpörajoituksia, erityisesti suljetuissa tai ilmakäytössä olevissa asennuksissa. Chloros valvoo Jetsonin sisäisiä lämpötila-antureita ja skaalaa eräkokoja automaattisesti:

| Lämpötila | Reaktio |
| --- | --- |
| **&lt; 70 °C** | Normaali toiminta — täysi nopeus |
| **70 °C** (Varoitus) | Erän koko pienenee asteittain (100 % → 50 % välillä 70 °C – 80 °C) |
| **80 °C** (Kriittinen) | Voimakas tehonrajoitus (50 % → 0 % välillä 80 °C – 90 °C) |
| **90 °C** (Sammutus) | GPU-käsittely pysäytetään kokonaan |

Riittävän jäähdytyksen omaavissa pöytätietokoneissa lämpösäätö laukeaa harvoin.

***

## Muistipaineen hallinta

Chloros valvoo GPU:n muistia jatkuvasti käsittelyn aikana ja reagoi kolmella tasolla.

**Erän koko.** Erä alkaa 8 kuvasta kerrottuna yllä olevien taulukoiden mukaisella alustakertoimella. Chloros tarkistaa sitten vapaan VRAM-muistin, varaa siitä 20 % PyTorchin omaa hallintakustannusta varten ja olettaa, että 12 MP:n kuva vaatii noin 100 MB GPU-muistia — erän koko on pienempi näistä kahdesta: muistista johdettu raja tai alustan perusarvo. Se ei koskaan laske alle 1:n.**Ennakoiva pienentäminen.**Kun**VRAM-käyttöaste ylittää 85 %**, eräkokoja pienennetään ennen kuin mitään vikaa ilmenee.**Säikeittäisen allokoinnin rajoittaminen.** Käytön kasvaessa kunkin säikeen GPU-budjettia pienennetään: ×0,75 yli 80 %:n käyttöasteella, ×0,5 yli 90 %:n käyttöasteella. Valvontarajat ovat 70 % (varovainen), 85 % (normaali toimintaraja) ja 95 % (OOM-riski).**OOM-peruutus ja palautuminen.** Jos muistin loppumistilanne kuitenkin tapahtuu:

* erän koko **puolitetaan**, ja se puolitetaan uudelleen jokaisen peräkkäisen OOM-tapahtuman yhteydessä — jokainen seuraava onnistunut erä siirtää tätä rangaistusta yhden askeleen taaksepäin
* aktiivisten säikeiden resurssivaraukset leikataan 70 %:iin niiden nykyisestä arvosta ja allokoija siirtyy konservatiiviseen strategiaan, jota lievennetään jälleen onnistuneiden allokointien sarjan jälkeen
* vakavan kuormituksen alla putki siirtyy tilasta `fused_gpu` tilaan `tiled_gpu` ja viimeisenä keinona tilaan `cpu_fallback`

**Isäntäkoneen RAM-muisti (Jetson).** Ennen käsittelyä CLI arvioi isäntäkoneen huippumuistin kuvamäärän ja debayer-tilan perusteella ja varoittaa, jos RAM-muisti ja tiedostopohjainen swap-tila eivät todennäköisesti riitä, tulostaen tarkat komennot swap-tilan lisäämiseksi — katso [NVIDIA Jetson -opas](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).***

## Laskentakapasiteetin mukautumisen seuranta

### Järjestelmän diagnostiikka

`chloros-cli selftest` on nopein tapa tarkistaa, mitä laskentakerros havaitsee:

```bash
chloros-cli selftest
```

Sen seitsemän tarkistusta kattavat version, porttien saatavuuden, taustapalvelun käynnistymisen, `/api/test`:n, järjestelmätiedot, kohinanpoistomallin olemassaolon sekä CUDA:n ja kohinanpoistomallin valmiuden. Tarkistus 5 tulostaa laitteistotiedot suoraan:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

Tarkistus 7 tulostaa `CUDA: <bool>, Denoiser: <bool>` — molempien on oltava totta, jotta Texture Aware -ominaisuutta voidaan ylipäätään käyttää.

### Taustaprosessin lokit

Strategia ja työntekijöiden lukumäärä valitaan taustaprosessin sisällä kunkin ajon alussa — niitä ei ilmoiteta erillisellä CLI-bannerilla. Kun jokin toimii odottamattomasti (GPU-polun varajärjestelmään siirtyminen, muistin loppuminen (OOM), denoiserin latausongelma), se näkyy kyseisen istunnon taustapalvelimen lokissa:

| Alusta | Lokin sijainti |
| --- | --- |
| **Linux / Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (yksi tiedosto kutakin käynnistystä kohti) |
| **Linux, CLI-started backend** | myös `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Reaaliaikainen edistymisnäyttö

Käynnistyksen aikana CLI näyttää reaaliaikaisen edistymisen säikeittäin (tunnistaminen, analysointi, käsittely, vienti), joka välitetään Server-Sent Events -protokollan kautta — tämä on käytännöllinen tapa selvittää, onko säie 3 pullonkaula. Katso [Käsittelyputki](processing-pipeline.md).

***

## Seuraavat vaiheet

* [Käsittelyputki](processing-pipeline.md) — 4-säikeisen putkiarkkitehtuurin ymmärtäminen
* [NVIDIA Jetson -opas](../linux/nvidia-jetson-guide.md) — Jetson-laitteille ominainen käyttöönotto ja optimointi
* [CLI : Komentorivi](../CLI.md) — CLI-opas
* [CLI-viite](../reference/cli-reference.md) — Kattava komentojen luettelo versioon 1.2.0
