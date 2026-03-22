# Dynaaminen laskentaympäristön mukautus

Chloros 1.1.0 tuo mukanaan älykkään laitteistotunnistuksen ja automaattisen käsittelystrategian valinnan. Käsittelymoottori mukautuu laitteistoosi – Jetson Nanosta monen GPU:n työasemaan – ilman manuaalista konfigurointia.

***

## Toimintaperiaate

Kun Chloros käynnistyy, se luo automaattisesti profiilin järjestelmästäsi:

1. **Tunnistaa käyttöjärjestelmän** — Windows tai Linux
2. **Tunnistaa CPU-ytimet ja kokonaisRAM-muistin**

3.**Tunnistaa GPU:n läsnäolon** — NVIDIA CUDA -ominaisuus, VRAM, malli
4. **Tunnistaa Jetson-mallin** (jos sovellettavissa) — `/proc/device-tree/model`:n kautta
5. **Tarkistaa lämpötila-anturit** (Jetson) — lämpötilaa huomioivaa käsittelyä varten
6. **Valitsee optimaalisen laskentastrategian** — kaikkien havaittujen laitteistojen perusteella
7. **Määrittää työntekijöiden lukumäärän, putkityypin ja muistin allokoinnin** automaattisesti

Tulos tallennetaan välimuistiin, jotta seuraavat suoritukset käynnistyvät nopeammin. Jos laitteisto muuttuu (esim. GPU lisätään), Chloros luo uuden profiilin seuraavalla käynnistyksellä.

***

## Laskentastrategiat

Chloros valitsee yhden kolmesta laskentastrategiasta laitteistosi perusteella:

| Strategia | Vaadittu GPU | Työntekijät | Putki | Sopii parhaiten |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`** | Kyllä (vähintään 12 Gt VRAM-muistia tai vähintään 16 Gt jaettua muistia) | 3–4 | `fused_gpu` | Pöytätietokoneiden GPU:t, joissa on vähintään 12 Gt, Jetson Orin NX 16 Gt, AGX Orin |
| **`GPU_SINGLE`** | Kyllä (&lt; 12 Gt VRAM) | 1–3 | `tiled_gpu` | Perustason GPU:t, Jetson Nano, Orin Nano |
| **`CPU_PARALLEL`** | Ei | ytimet - 1 | `cpu_fallback` | Järjestelmät ilman NVIDIA-grafiikkapiiriä |

### Putkityypit

* **`fused_gpu`** — Täysi grafiikkapiirin käsittelypolku. Kaikki debayer-, korjaus- ja indeksointitoiminnot suoritetaan GPU:lla yhdellä fuusioidulla kierroksella. Suurin läpimenokapasiteetti, mutta vaatii enemmän VRAM-muistia.
* **`tiled_gpu`** — Muistitehokas GPU-polku. Käsittelee kuvia ruuduittain, jotta ne mahtuvat rajoitettuun GPU-muistiin. Pienempi läpimenokapasiteetti, mutta toimii laitteissa, joissa muistia on rajoitetusti.
* **`cpu_fallback`** — Pelkästään CPU:lla suoritettava käsittely, jossa käytetään monisäikeistä rinnakkaisuutta. Käytetään, kun NVIDIA-GPU:ta ei ole käytettävissä.***

## Alustakohtainen käyttäytyminen

| Alusta | Strategia | Työntekijät | Putki | Huomautukset |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sarjoitettu) | Muistitehokas tila, käsittelee yhden kuvan kerrallaan |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 3 | `fused_gpu` (samanaikainen) | Suositeltu reunalaite — todellinen rinnakkainen GPU-käsittely |
| **Jetson AGX Orin 64 GB** | `GPU_PARALLEL` | 4 | `fused_gpu` (samanaikainen) | Maksimaalinen reuna-laitteen suorituskyky |
| **Pöytätietokone, jossa on 8 Gt:n GPU** | `GPU_SINGLE` | 3 | `tiled_gpu` | Hyvä pöytätietokoneen suorituskyky muistitehokkailla ruuduilla |
| **Pöytätietokone, jossa on vähintään 12 Gt:n GPU** | `GPU_PARALLEL` | 3–4 | `fused_gpu` | Optimaalinen pöytätietokoneen suorituskyky |
| **Pelkkä CPU-järjestelmä** | `CPU_PARALLEL` | ytimet - 1 | `cpu_fallback` | Ei vaadi GPU:ta, käyttää ThreadPoolia |

{% hint style="info" %}
**Jetsonin yhtenäinen muisti**: Jetson-laitteet jakavat GPU- ja CPU-muistin. Jetson Orin NX 16GB ilmoittaa ~15,3 GB VRAM-muistia, mutta tämä on sama fyysinen RAM-muisti, jota käyttöjärjestelmä ja CPU-prosessit käyttävät. Chloros ottaa tämän huomioon muistivarausrajoja asetettaessa.
{% endhint %}

***

## Dynaaminen GPU-muistin allokointi

Chloros käyttää [4-säikeistä käsittelyputkea](processing-pipeline.md):

* **Säie 1** (Tunnistus) — Kuvan lataus, EXIF-tietojen jäsentäminen, kohteen tunnistus
* **Säie 2** (Kalibrointi) — Heijastavuuden kalibrointilaskelma
* **Säie 3** (Käsittely) — GPU-debayer, vinjetin korjaus, indeksin laskeminen
* **Säie 4** (Vienti) — Tiedostojen kirjoittaminen, metatietojen upottaminen

Kun aikaisemmat prosessointiputken säikeet saavat työnsä valmiiksi (esim. kaikki kuvat on tunnistettu), niiden GPU-muistin allokointi vapautuu ja **jaetaan uudelleen jäljellä oleville aktiivisille säikeille**. Tämä tarkoittaa, että säie 3 (GPU-intensiivinen vaihe) saa asteittain enemmän muistia prosessin edetessä, mikä parantaa laskentatehoa laskennallisesti vaativimmissa tehtävissä.

### Allokointivaiheet

| Vaihe | Aktiiviset säikeet | GPU-muistin jakautuminen |
| --- | --- | --- |
| **Alkuvaihe** | 1, 2, 3, 4 | Jaettu kaikkien säikeiden kesken |
| **Alku-keskivaihe** | 2, 3, 4 | Säiteen 1 muisti jaetaan uudelleen |
| **Keskivaihe-loppuvaihe** | 3, 4 | Säikeiden 1+2 muisti siirtyy säikeille 3+4 |
| **Loppuvaihe** | 3 tai 4 | Enimmäismuisti jäljellä olevalle säikeelle |***

## Tekstuuritietoinen käsittely

Tekstuuritietoinen debayer-menetelmä (vain Chloros+) käyttää huomattavasti enemmän GPU-muistia kuin Standard-menetelmä AI/ML-kohinanpoistomallin vuoksi:

* Järjestelmät, joissa on **&lt; 7 Gt VRAM**, pakotetaan synkroniseen käsittelysilmukkaan tekstuuritietoisessa tilassa (yksi kuva kerrallaan)
* Järjestelmät, joissa on **7 Gt tai enemmän VRAM-muistia**, voivat käsitellä tekstuuritietoista käsittelyä samanaikaisesti, vaikkakin pienemmällä työntekijämäärällä verrattuna standardiin***

## Lämmönhallinta (Jetson)

Jetson-laitteilla on lämpörajoituksia, erityisesti suljetuissa tai lentokäytössä olevissa asennuksissa. Chloros valvoo GPU:n ja CPU:n lämpötiloja ja säätää käsittelyä automaattisesti:

| Lämpötila | Reaktio |
| --- | --- |
| **&lt; 70 °C** | Normaali toiminta — täysi nopeus |
| **70 °C** (Varoitus) | Pienennä erän kokoa |
| **80 °C** (Kriittinen) | Aggressiivinen kuristaminen — pienennä samanaikaisuutta ja työntekijöiden määrää |
| **90 °C** (Sammutus) | Lopeta GPU-käsittely kokonaan |

Lämpötilan seurannassa käytetään `tegrastats`:ää Jetson-alustoilla. Riittävän jäähdytyksen omaavissa pöytätietokoneissa lämpösäätö laukeaa harvoin.

***

## Muistin kuormituksen hallinta

Chloros valvoo järjestelmän muistipaineita käsittelyn aikana:

* **Muistikynnys**: 85 %:n käyttöaste laukaisee varovaisen käyttäytymisen
* **OOM-vähennys**: Jos muistin loppumistilanne tapahtuu, allokointia vähennetään 25 % (kerroin 0,75)
* **Pipeline-varajärjestelmä**: Vakavassa muistipaineessa putki siirtyy automaattisesti `fused_gpu`:stä `tiled_gpu`:ään
* **Swap-suositukset**: Jetsonissa Chloros varoittaa, jos swap-tilaa ei ole riittävästi datajoukon koon kannalta***

## Laskentasovituksen seuranta

### CLI-tilan tulostus

Kun käsittely alkaa, CLI näyttää havaitun laitteistoprofiilin:

```

Chloros CLI 1.1.0
Platform: Linux aarch64 (Jetson Orin NX 16GB)
Strategy: GPU_PARALLEL | Workers: 3 | Pipeline: fused_gpu
CUDA: Available | GPU Memory: 15.3 GB (shared)
```

### Järjestelmän diagnostiikka

Suorita `chloros-cli selftest` nähdäksesi täydellisen laitteistoprofiilin ja tarkistaaksesi laskentakapasiteetin:

```bash
chloros-cli selftest
```

Tämä tarkistaa CUDA:n saatavuuden, GPU-muistin, kohinanpoistomallit ja taustayhteydet.

***

## Seuraavat vaiheet

* [Käsittelyputki](processing-pipeline.md) — 4-säikeisen putkiarkkitehtuurin ymmärtäminen
* [NVIDIA Jetson -opas](../linux/nvidia-jetson-guide.md) — Jetson-laitteille ominainen käyttöönotto ja optimointi
* [CLI : Komentorivi](../CLI.md) — Täydellinen CLI-viite
