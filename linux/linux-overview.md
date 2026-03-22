# Linux:n yleiskatsaus

Chloros 1.1.0 tarjoaa natiivin Linux-tuen **CLI**- ja**Python SDK**, mikä mahdollistaa päättömän monispektrisen kuvankäsittelyn Linux-työasemilla, palvelimilla ja NVIDIA Jetson -reunalaitteilla.

{% hint style="info" %}
**Linux:ssä ei ole graafista käyttöliittymää.** Chloros-työpöydän graafinen käyttöliittymä on käytettävissä vain Windows:ssä. Linux-käyttäjät ovat vuorovaikutuksessa Chloros:n kanssa [CLI](../CLI.md) ja [Python SDK](../api-python-sdk.md).
{% endhint %}

***

## Alustatukimatriisi

| Ominaisuus | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Työpöydän käyttöliittymä** | Kyllä | Ei saatavilla | Ei | Ei |
| **CLI** | Kyllä | Kyllä | Kyllä | Kyllä |
| **Python SDK** | Kyllä | Kyllä | Kyllä | Kyllä |
| **GPU-kiihdytys (CUDA)** | Kyllä | Kyllä | Kyllä | Kyllä (JetPack 6) |
| **Tekstuuritunnistava debayer** | Kyllä (Chloros+) | Kyllä (Chloros+) | Kyllä (Chloros+) | Kyllä (Chloros+) |
| **Dynaaminen laskentasovitus** | Kyllä | Kyllä | Kyllä | Kyllä |***

## Tuetut arkkitehtuurit

| Arkkitehtuuri | Kuvaus | Asennustapa |
| --- | --- | --- |
| **amd64 (x86_64)** | Tavalliset työpöytä-/palvelinprosessorit (Intel, AMD) | `.deb`-paketti |
| **arm64 (aarch64)** | ARM-pohjaiset prosessorit, pääasiassa NVIDIA Jetson | `.deb`-paketti (JetPack 6) |

## Tuetut Linux-jakelut

* **Ubuntu 20.04+** (amd64)
* **Debian 11+** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson-alustat)***

## Mitä Linux-käyttäjät saavat

* **Chloros CLI** — Täydellinen komentoriviliittymä eräprosessointiin, automaatioon ja skriptaukseen
* **Chloros Python SDK** — Ohjelmoitava Python-rajapinta (`pip install chloros-sdk`) integrointia varten tutkimusprosesseihin ja mukautettuihin työkaluihin
* **GPU-kiihdytys** — CUDA-kiihdytetty käsittely NVIDIA-GPU:illa (pöytätietokoneet ja Jetson)
* **Dynaaminen laskentasovitus** — Automaattinen laitteiston tunnistus ja käsittelystrategian optimointi
* **Kaikki käsittelyominaisuudet** — Sama monispektrinen käsittelyputki kuin Windows (kalibrointi, vinjetin korjaus, kasvillisuusindeksit, kaikki vientimuodot)
* **Chloros+ -ominaisuudet** — Monisäikeinen käsittely, tekstuuritunnistava debayer, mukautetut indeksit (Chloros+ -lisenssillä)

## Mitä Linux-käyttäjät eivät saa

* **Työpöydän käyttöliittymä** — Ei graafista käyttöliittymää; kaikki vuorovaikutus tapahtuu CLI:n tai Python:n kautta
* **Kuvankatseluohjelma** — Ei interaktiivista kuvankatseluohjelmaa, ruudukkonäkymää tai karttamerkkejä
* **Visuaalinen projektinhallinta** — Projekteja hallitaan CLI-komentojen ja SDK-kutsujen kautta***

## Aloittaminen Linux:llä

1. **Asenna Chloros** — Katso [Linux Asennus](linux-installation.md) `.deb`-paketin asennusta varten
2. **Asenna Python SDK** (valinnainen) — `pip install chloros-sdk`
3. **Aktivoi lisenssisi** — `chloros-cli login your@email.com 'password'`
4. **Käsittele ensimmäinen tietojoukko** — `chloros-cli process ~/datasets/flight001`

NVIDIA Jetson -käyttäjät voivat tutustua alustakohtaisiin asennus- ja optimointiohjeisiin erillisessä [NVIDIA Jetson -oppaassa](nvidia-jetson-guide.md).

***

## Seuraavat vaiheet

* [Linux Asennus](linux-installation.md) — Yksityiskohtaiset asennusohjeet amd64- ja arm64-alustoille
* [NVIDIA Jetson -opas](nvidia-jetson-guide.md) — Jetson-kohtaiset asetukset, lämmönhallinta ja kenttäkäyttöönotto
* [CLI : Komentorivi](../CLI.md) — Täydellinen CLI-viite
* [API : Python SDK](../api-python-sdk.md) — Täydellinen SDK-viite
* [Dynaaminen laskentakapasiteetin mukautus](../processing-architecture/dynamic-compute-adaptation.md) — Miten Chloros mukautuu laitteistoosi
