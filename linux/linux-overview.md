# Linux-yleiskatsaus

Chloros 1.3.0 tarjoaa natiivin Linux-tuen **CLI:lle**ja**Python SDK:lle** — näytöttömän monispektrikuvankäsittelyn sekä LATTICE-kameran ja DAQ-valosensorin reaaliaikaisen ohjauksen — Linux-työasemilla, palvelimilla ja NVIDIA Jetson -reunalaitteilla.

{% hint style="info" %}
**Linuxissa ei ole työpöytäkäyttöliittymää.**Chloros-työpöytäkäyttöliittymä on saatavilla vain Windowsille. Linux-käyttäjät käyttävät Chlorosia [CLI:n](../CLI.md) ja [Python-SDK:n](../api-python-sdk.md) kautta. `.deb` lisää**Chloros CLI** -kohdan sovellusvalikkoon — se avaa yksinkertaisesti `chloros-cli`:ää ajavan terminaaliemulaattorin.
{% endhint %}

***

## Alustatukitaulukko

| Ominaisuus | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Työpöydän graafinen käyttöliittymä** | Kyllä | Ei saatavilla | Ei | Ei |
| **CLI** (`chloros-cli`) | Kyllä | Kyllä | Kyllä | Kyllä |
| **Python-SDK** (`chloros-sdk`) | Kyllä | Kyllä | Kyllä | Kyllä |
| **Kuvankäsittelyputki** | Kyllä | Kyllä | Kyllä | Kyllä |
| **LATTICE-kameran ohjaus (reaaliaikainen)** | Kyllä (Kamerat-välilehti) | Kyllä (`chloros-cli lattice`, SDK) | Kyllä | Kyllä |
| **DAQ-valosensorit (reaaliaikainen)** | Kyllä (Valosensorit-välilehti) | Kyllä (`chloros-cli daq pool-*`, SDK) | Kyllä | Kyllä |
| **PTP-aikasynkronointi (isäntä on grandmaster)** | Kyllä | Kyllä (`chloros-cli time-sync`) | Kyllä | Kyllä |
| **GPU-kiihdytys (CUDA)** | Kyllä | Kyllä | Kyllä | Kyllä (JetPack 6) |
| **Tekstuuritunnistava debayer** (vaatii CUDA-GPU:n) | Kyllä (Chloros+) | Kyllä (Chloros+) | Kyllä (Chloros+) | Kyllä (Chloros+) |
| **Dynaaminen laskentasovitus** | Kyllä | Kyllä | Kyllä | Kyllä |
| **Taustapalvelu järjestelmäpalveluna** (`chloros-backend@<user>.service`) | Ei | Ei | Kyllä (valinnainen) | Kyllä (valinnainen) |
| **Paikallinen päivitysohjelma** (`chloros-cli update`) | Ei (suorita asennusohjelma) | Ei (suorita asennusohjelma) | Kyllä | Kyllä |

***

## Tuetut arkkitehtuurit

| Arkkitehtuuri | Kuvaus | Paketti |
| --- | --- | --- |
| **amd64 (x86_64)** | Tavalliset työpöytä- ja palvelinprosessorit (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | ARM-prosessorit — NVIDIA Jetson Orin -tuoteperhe (JetPack 6) | `chloros_<version>_arm64_jp6.deb` |

## Tuetut Linux-jakelut

* **Ubuntu 22.04 LTS tai uudempi** (amd64)
* **Debian 12 tai uudempi** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson Orin -alustat)

***

## Mitä Linux-käyttäjät saavat

* **Chloros CLI** — kattava komentoriviliitäntä eräprosessointia, automaatiota ja skriptejä varten
* **Chloros Python SDK** — ohjelmointirajapinta Python-kielellä tutkimusprosesseja ja mukautettuja työkaluja varten; asennetaan `.deb`-komennolla järjestelmään `python3` (versiota vastaava wheel virtualenvs-ympäristöille löytyy tiedostosta `/usr/lib/chloros/sdk/`; PyPI on tarkoitettu koneille, joissa ei ole Chloros-pakettia)
* **LATTICE-kameran ohjaus** — LATTICE-kameroiden ja synkronoitujen monikamerajärjestelmien tunnistaminen, liittäminen, konfigurointi ja kuvaus tapahtuu `chloros-cli lattice`:n ja SDK:n kautta; `.deb` sisältää kameroiden edellyttämän Arena SDK -ajoympäristön
* **DAQ-valosensorien hallinta** — liitä DAQ-U/M/E-sensoreita, lähetä kalibroituja spektrejä ja tallenna `.daq`-tiedostoja `chloros-cli daq pool-*`:n ja SDK:n kautta
* **PTP-aikasynkronointi** — Chloros-taustapalvelu käyttää PTP-grandmasteria, johon LATTICE-kamerat ja DAQ-E-anturit ovat orja-laitteina kytkettyjä; tarkista se `chloros-cli time-sync`:n avulla ja pidä se käynnissä taustalla `chloros-backend@<user>.service`-systemd-yksikön avulla (katso [Linux-asennus](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Projektien automatisointi** — aja tallennettuja projekteja taustalla `chloros-cli project`:n ja SDK:n `open_project`:n avulla
* **GPU-kiihdytys** — CUDA-kiihdytetty käsittely NVIDIA-grafiikkaprosessoreilla (amd64-alustan työpöytägrafiikkaprosessorit sekä Jetsonin grafiikkaprosessori JetPack 6:n kanssa)
* **Dynaaminen laskennan mukautus** — automaattinen laitteiston tunnistus ja käsittelystrategian valinta, jossa `CHLOROS_STRATEGY`-ohitus toimii asiantuntijan pakotienä
* **Kaikki käsittelyominaisuudet** — sama prosessiketju kuin Windowsissa: kalibrointi, vinjetoinnin korjaus, kasvillisuusindeksit ja kaikki vientimuodot
* **Chloros+ -ominaisuudet** — monisäikeinen (putkistettu) käsittely, Texture Aware -debayer (CUDA-grafiikkaprosessorilla) ja mukautetut indeksit, maksullisella Chloros+ -sopimuksella

## Mitä Linux-käyttäjät eivät saa

* **Työpöydän graafinen käyttöliittymä** — ei graafista käyttöliittymää; kaikki vuorovaikutus tapahtuu CLI:n tai Python-SDK:n kautta
* **Kuvankatseluohjelma** — ei interaktiivista kuvankatseluohjelmaa, ruudukkonäkymää tai karttamerkkejä
* **Visuaalinen projektinhallinta** — projektit luodaan ja hallitaan CLI-komentojen ja SDK-kutsuiden avulla (itse laitteisto — kamerat, anturit, tallennus — on edelleen täysin hallittavissa terminaalista)

***

## Lisenssivaatimukset

CLI- ja SDK-käyttöoikeudet edellyttävät **maksullista Chloros+-tasoa — Copper tai korkeampi**(Copper, Bronze, Silver, Gold). Ilmaisella**Iron**-tasolla ei ole CLI/SDK-käyttöoikeuksia. Rajaa valvotaan taustapalvelimella, ei pelkästään CLI:n kautta:

| Tilanne | Taustapalvelimen vastaus |
| --- | --- |
| Ei kirjautunut sisään | `401` ja `error_code: AUTH_REQUIRED` |
| Kirjautunut sisään ilmaisella Iron-tasolla | `403` ja `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` toimii kaikilla tasoilla — se on ainoa reitti, joka on vapautettu portista — joten hylkäyksen syy on aina näkyvissä.

***

## Aloittaminen Linuxissa

1. **Asenna Chloros** — katso [Linux-asennus](linux-installation.md) `.deb`:n asennusta varten
2. **Tarkista** — `chloros-cli --version` tulostaa `Chloros CLI 1.3.0`; `chloros-cli selftest` suorittaa 7-vaiheisen diagnoosin
3. **Tarkista Python-SDK** — `.deb` asentaa sen: `python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"` (virtualenv-ympäristössä, `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`)
4. **Kirjaudu sisään** — `chloros-cli login your@email.com 'your-password'` (kerran kutakin konetta kohti; paketin päivitykset säilyttävät kirjautumistiedot, paketin poistaminen tyhjentää ne)
5. **Käsittele ensimmäinen aineistosi** — `chloros-cli process ~/datasets/flight001`

NVIDIA Jetsonin osalta katso erillinen [NVIDIA Jetson -opas](nvidia-jetson-guide.md), josta löydät alustakohtaiset asennusohjeet, lämpökäyttäytymistä koskevat tiedot ja kenttäkäyttöönotto-ohjeet.

***

## Seuraavat vaiheet

* [Linux-asennus](linux-installation.md) — yksityiskohtaiset asennusohjeet, tiedostojen sijainnit ja vianmääritys amd64- ja arm64-alustoille
* [NVIDIA Jetson -opas](nvidia-jetson-guide.md) — Jetson-kohtaiset asetukset, muistin ja lämmönkäyttäytyminen sekä kenttäkäyttöönotto
* [CLI: komentorivi](../CLI.md) — CLI-opas
* [API: Python SDK](../api-python-sdk.md) — SDK-opas
* [CLI-viite](../reference/cli-reference.md) ja [SDK-viite](../reference/sdk-reference.md) — kattavat komento- ja API-luettelot versioon 1.3.0
* [Dynaaminen laskentakapasiteetin mukautus](../processing-architecture/dynamic-compute-adaptation.md) — miten Chloross mukautuu laitteistoosi

{% hint style="info" %}
**Tämän oppaan lukeminen ohjelmointitavalla.** Jokainen sivu on saatavilla myös raakamuodossa Markdown-muodossa omalla URL-osoitteellaan, johon on lisätty `.md` (esimerkiksi `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), ja koko käyttöoppaan hakemisto on julkaistu osoitteessa [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
