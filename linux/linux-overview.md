# Linux – yleiskatsaus

Chloros 1.2.0 tarjoaa natiivin Linux-tuen **CLI**:lle ja**Python SDK** — päätön monispektrinen kuvankäsittely sekä LATTICE-kameran ja DAQ-valosensorin reaaliaikainen ohjaus — Linux-työasemilla, palvelimilla ja NVIDIA Jetson -reunalaitteilla.

{% hint style="info" %}
**Linux-laitteessa ei ole työpöytäkäyttöliittymää.**Chloros-työpöytäkäyttöliittymä on käytettävissä vain Windows-laitteessa. Linux-käyttäjät käyttävät Chloros-järjestelmää [CLI](../CLI.md)- ja [Python SDK](../api-python-sdk.md)-sovellusten kautta. `.deb` lisää**Chloros CLI** -kohdan sovellusvalikkoosi — se avaa yksinkertaisesti `chloros-cli`-ohjelmaa ajavan terminaaliemulaattorin.
{% endhint %}

***

## Alustatukitaulukko

| Ominaisuus | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Työpöydän käyttöliittymä** | Kyllä | Ei sovellu | Ei | Ei |
| **CLI** (`chloros-cli`) | Kyllä | Kyllä | Kyllä | Kyllä |
| **Python SDK** (`chloros-sdk`) | Kyllä | Kyllä | Kyllä | Kyllä |
| **Kuvankäsittelyputki** | Kyllä | Kyllä | Kyllä | Kyllä |
| **LATTICE-kameran ohjaus (reaaliaikainen)** | Kyllä (Kamerat-välilehti) | Kyllä (`chloros-cli lattice`, SDK) | Kyllä | Kyllä |
| **DAQ-valosensorit (reaaliaikainen)** | Kyllä (Valosensorit-välilehti) | Kyllä (`chloros-cli daq pool-*`, SDK) | Kyllä | Kyllä |
| **PTP-aikasynkronointi (isäntä on grandmaster)** | Kyllä | Kyllä (`chloros-cli time-sync`) | Kyllä | Kyllä |
| **GPU-kiihdytys (CUDA)** | Kyllä | Kyllä | Kyllä | Kyllä (JetPack 6) |
| **Tekstuuritietoinen debayer** | Kyllä (Chloros+) | Kyllä (Chloros+) | Kyllä (Chloros+) | Kyllä (Chloros+) |
| **Dynaaminen laskentasovitus** | Kyllä | Kyllä | Kyllä | Kyllä |
| **Taustapalvelu järjestelmäpalveluna** (`chloros-backend.service`) | Ei | Ei | Kyllä (valinnainen) | Kyllä (valinnainen) |
| **Paikallinen päivitysohjelma** (`chloros-cli update`) | Ei (suorita asennusohjelma) | Ei (suorita asennusohjelma) | Kyllä | Kyllä |***

## Tuetut arkkitehtuurit

| Arkkitehtuuri | Kuvaus | Paketti |
| --- | --- | --- |
| **amd64 (x86_64)** | Tavalliset työpöytä-/palvelinprosessorit (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | ARM-prosessorit — NVIDIA Jetson Orin -tuoteperhe | `chloros_<version>_arm64_jp6.deb` (JetPack 6 -versio) |

## Tuetut Linux-jakelut

* **Ubuntu 22.04 LTS tai uudempi** (amd64)
* **Debian 12 tai uudempi** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson Orin -alustat)***

## Mitä Linux-käyttäjät saavat

* **Chloros CLI** — kattava komentoriviliittymä eräprosessointia, automaatiota ja skriptien kirjoittamista varten
* **Chloros Python SDK** — ohjelmoitava Python-rajapinta tutkimusprosesseille ja mukautetuille työkaluille (asennettavissa PyPI:stä, ja mukana myös `.deb`:n sisällä versioon sopivana wheel-tiedostona)
* **LATTICE-kameroiden hallinta** — LATTICE-kameroiden ja synkronoitujen monikamerajärjestelmien tunnistaminen, liittäminen, konfigurointi ja kuvaus `chloros-cli lattice`:n ja SDK:n kautta; `.deb` sisältää kameroiden edellyttämän Arena SDK-ajoympäristön
* **DAQ-valosensorien hallinta** — liitä DAQ-U/M/E-anturit, lähetä kalibroituja spektrejä ja tallenna `.daq`-tiedostoja `chloros-cli daq pool-*`:n ja SDK:n kautta
* **PTP-aikasynkronointi** — Chloros-taustapalvelu ajaa PTP-grandmasteria, johon LATTICE-kamerat ja DAQ-E-anturit ovat orja-laitteina kytkettyjä; tarkista se `chloros-cli time-sync`:n avulla, ja pidä se käynnissä headless-tilassa `chloros-backend.service`-systemd-yksikön avulla (katso [Linux-asennus](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Projektien automatisointi** — aja tallennettuja projekteja taustalla komennolla `chloros-cli project` ja SDK:n `open_project`:llä
* **GPU-kiihdytys** — CUDA-kiihdytetty käsittely NVIDIA-grafiikkaprosessoreilla (pöytätietokoneet ja Jetson)
* **Dynaaminen laskentasovitus** — automaattinen laitteiston tunnistus ja käsittelystrategian valinta, jossa `CHLOROS_STRATEGY`-ohitus toimii asiantuntijan varasuunnitelmana
* **Kaikki käsittelyominaisuudet** — sama prosessiputki kuin Windows: kalibrointi, vinjetin korjaus, kasvillisuusindeksit ja kaikki vientimuodot
* **Chloros+ -ominaisuudet** — monisäikeinen (putkistettu) käsittely, Texture Aware -debayer ja mukautetut indeksit, maksullisella Chloros+ -sopimuksella

## Mitä Linux-käyttäjät eivät saa

* **Työpöytäkäyttöliittymä** — ei graafista käyttöliittymää; kaikki vuorovaikutus tapahtuu CLI:n tai Python:n kautta SDK
* **Kuvankatseluohjelma** — ei interaktiivista kuvankatseluohjelmaa, ruudukkonäkymää tai karttamerkkejä
* **Visuaalinen projektinhallinta** — projektit luodaan ja hallitaan CLI-komentojen ja SDK-kutsujen avulla (itse laitteisto — kamerat, anturit, tallennus — on edelleen täysin ohjattavissa terminaalista)***

## Lisenssivaatimukset

CLI- ja SDK-käyttöoikeudet edellyttävät **maksullista Chloros+-tasoa — Copper tai korkeampi**(Copper, Bronze, Silver, Gold). Ilmaisella**Iron**-tasolla ei ole pääsyä CLI- ja SDK-toimintoihin. Rajaa valvoo taustajärjestelmä, ei pelkästään CLI:

| Tilanne | Palvelimen vastaus |
| --- | --- |
| Ei kirjautunut sisään | `401` ja `error_code: AUTH_REQUIRED` |
| Kirjautunut sisään ilmaisella Iron-tasolla | `403` ja `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` toimii kaikilla tasoilla — se on ainoa reitti, joka on vapautettu portista — joten hylkäyksen syy on aina näkyvissä.

***

## Aloittaminen Linux:llä

1. **Asenna Chloros** — katso [Linux:n asennusohjeet](linux-installation.md) `.deb`:n asennusta varten
2. **Tarkista** — `chloros-cli --version` tulostaa `Chloros CLI 1.2.0`; `chloros-cli selftest` suorittaa 7-vaiheisen vianmäärityksen
3. **Asenna Python ja SDK** (valinnainen) — `pip install chloros-sdk`
4. **Kirjaudu sisään** — `chloros-cli login your@email.com 'your-password'` (kerran kutakin laitetta kohti ja uudelleen jokaisen pakettipäivityksen jälkeen)
5. **Käsittele ensimmäinen aineistosi** — `chloros-cli process ~/datasets/flight001`

NVIDIA Jetsonin osalta katso erillinen [NVIDIA Jetson -opas](nvidia-jetson-guide.md), jossa on alustakohtaiset asennusohjeet, lämpökäyttäytyminen ja kenttäkäyttöönotto.

***

## Seuraavat vaiheet

* [Linux Asennus](linux-installation.md) — yksityiskohtaiset asennusohjeet, tiedostojen sijainnit ja vianmääritys amd64- ja arm64-alustoille
* [NVIDIA Jetson -opas](nvidia-jetson-guide.md) — Jetson-alustakohtaiset asetukset, muistin ja lämmönkäyttäytyminen sekä kenttäkäyttöönotto
* [CLI : Komentorivi](../CLI.md) — CLI-opas
* [API : Python SDK](../api-python-sdk.md) — SDK-opas
* [CLI-viite](../reference/cli-reference.md) ja [SDK-viite](../reference/sdk-reference.md) — kattavat komento- ja API-luettelot versioon 1.2.0
* [Dynaaminen laskentakapasiteetin mukautus](../processing-architecture/dynamic-compute-adaptation.md) — miten Chloros mukautuu laitteistoosi

{% hint style="info" %}
**Tämän käyttöoppaan lukeminen ohjelmoidusti.** Jokainen sivu on saatavilla myös raakamuodossa Markdown-muodossa omalla sivullaan URL sekä `.md` (esimerkiksi `https://mapir.gitbook.io/chloros/linux/linux-installation.md`) -osoitteessa, ja koko käyttöohjeen hakemisto on julkaistu osoitteessa [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
