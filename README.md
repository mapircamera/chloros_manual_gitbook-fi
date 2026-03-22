---
metaLinks: {}
---

# Aloitus

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros on [MAPIR](https://www.mapir.camera) -yrityksen kehittämä sovellus kuvien ja muiden anturitietojen käsittelyyn.

***{% hint style="success" %}**Uutta Chloros 1.1.0:ssa**: Natiivi Linux-tuki (amd64 ja arm64), NVIDIA Jetson -reunalaskenta, dynaaminen laskentasovitus, 4-säikeinen käsittelyputki, uudet CLI-komennot ja -vaihtoehdot. Katso [Lataa](download.md) saadaksesi täydellisen muutoslokin.
{% endhint %}

Chloros on saatavilla 3 sovellustilassa:

## Chloros: Työpöydän GUI-sovellus

Erillinen ikkuna, jossa on kaikki ominaisuudet. _Vain Windows._

## [Chloros CLI: Komentoriviliittymä](CLI.md)

Komentorivipohjainen eräajokäsittely. Sopii erinomaisesti automaatioon, skriptaukseen ja päättömään käyttöön. Saatavilla **Windows-, Linux amd64- ja Linux arm64 (NVIDIA Jetson)** -versioissa. _CLI:n käyttö edellyttää Chloros+ -lisenssiä._

## [Chloros API: Python SDK](api-python-sdk.md)

Ohjelmoitava Python-rajapinta automaatioon ja mukautettuihin työnkulkuihin. Sopii erinomaisesti tutkimusprosesseihin, integrointiin olemassa oleviin Python-sovelluksiin sekä mukautettujen työkalujen rakentamiseen. Saatavilla **kaikilla alustoilla** `pip install chloros-sdk`:n kautta. _API:n käyttö edellyttää Chloros+ -lisenssiä._***

## Tuetut alustat

| Alusta | Käyttöliittymä | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11** | Kyllä | Kyllä | Kyllä |
| **Linux amd64 (x86_64)** | Ei | Kyllä | Kyllä |
| **Linux arm64 (NVIDIA Jetson)** | Ei | Kyllä | Kyllä |

Linux:n asennusohjeet löytyvät kohdasta [Linux &amp; Edge Computing](linux/linux-overview.md).

***

## Chloros+

Vaikka Chloros on ilmainen useimpiin tehtäviin, saatat huomata, että haluat enemmän. Siinä tapauksessa Chloros+:n maksullinen lisenssi voi olla sinulle hyödyllinen. Chloros+ -lisenssillä voit avata uusia ominaisuuksia, kuten:

* **Monisäikeinen käsittely**: nopeuta huomattavasti kuvankäsittelyä suurissa projekteissa käsittelemällä kuvia samanaikaisesti prosessiputken kautta.
* **GPU (CUDA) -kiihdytys**: hyödynnä nykypäivän suurempia GPU-muistivaihtoehtoja kuvankäsittelyputken nopeuttamiseksi entisestään. Suosittelemme vähintään 4 Gt:n VRAM-muistia parhaiden tulosten saavuttamiseksi.
* **Chloros+**[**CLI**](CLI.md)**Pääsy**: suorita Chloros+ komentoriviltä automatisoidaksesi ja integroidaksesi sen omaan ohjelmistoosi.
* **Chloros+**[**API**](api-python-sdk.md)**Käyttö:** suorita Chloros+ komennolla Python ohjelmoitavaa ohjausta varten, mikä mahdollistaa saumattoman integroinnin tutkimusprosesseihisi, data-analyysin työnkulkuihisi ja mukautettuihin sovelluksiisi.
* **Usean laitteen käyttö**: jokaisella Chloros+ -lisenssillä voi rekisteröidä vähintään 2 laitetta. Hallitse rekisteröityjä laitteita MAPIR Cloud -tilisi avulla. Lisää tukea useammille laitteille päivittämällä Chloros+ -lisenssisi.
* **Edistyksellinen tekstuuritietoinen debayer-menetelmä:** korkealaatuinen reuna-tietoinen debayer yhdistettynä AI/ML-kohinanpoistomalliin, joka poistaa lähes kaiken debayering-kohinan. 
* **Mukautetut monispektriset indeksikaavat:** syötä mukautetut monispektriset indeksit Chloros-rasterilaskureihin sekä käsittelyä että kuvien katselusandboxia varten.
* **Linux &amp; reuna-laskenta:** suorita Chloros Linux x86\_64- ja ARM64-alustoilla, mukaan lukien NVIDIA Jetson, kenttä- ja reuna-käsittelyä varten. Katso [Linux-yleiskatsaus](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Hinnat ja rekisteröityminen</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
